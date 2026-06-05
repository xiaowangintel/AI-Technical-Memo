# merge_attn_states.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/attention/merge_attn_states.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Merge prefix and suffix attention outputs with numerically stable log-sum-exp math, while optionally converting the merged result to FP8. / [CN] 使用数值稳定的 log-sum-exp 数学把 prefix 与 suffix 注意力输出合并，并且可选地把结果转换为 FP8。

## Line-by-Line Analysis / 逐行分析
### Kernel template and pack layout / 内核模板与打包布局
```cpp
// Implements section 2.2 of https://www.arxiv.org/pdf/2501.01005
// can be used to combine partial attention results (in the split-KV case)
template <typename scalar_t, typename output_t, const uint NUM_THREADS,
          bool USE_FP8_OUTPUT>
__global__ void merge_attn_states_kernel(
    output_t* output, float* output_lse, const scalar_t* prefix_output,
    const float* prefix_lse, const scalar_t* suffix_output,
    const float* suffix_lse, const uint num_tokens, const uint num_heads,
    const uint head_size, const uint prefix_head_stride,
    const uint output_head_stride, const uint prefix_num_tokens,
    const float* output_scale) {
  using input_pack_t = uint4;
  using output_pack_t =
      std::conditional_t<USE_FP8_OUTPUT,
                         std::conditional_t<sizeof(scalar_t) == 4, uint, uint2>,
                         uint4>;
  const uint pack_size = 16 / sizeof(scalar_t);
  const uint threads_per_head = head_size / pack_size;
```
**EN:** The kernel is parameterized by input type, output type, thread count, and whether the destination is FP8. It processes data in 128-bit packs (`uint4`) so each thread handles a vector chunk instead of a scalar element. `threads_per_head` tells the launcher how many packed threads are needed to cover one attention head.
**CN:** 这个内核按输入类型、输出类型、线程数以及目标是否为 FP8 进行模板化。它以 128-bit 打包（`uint4`）处理数据，因此每个线程负责一个向量块而不是单个标量。`threads_per_head` 表示覆盖一个 attention head 需要多少个“打包线程”。

### Global index mapping and suffix-only fast path / 全局索引映射与仅 suffix 快路径
```cpp
const uint global_idx = blockIdx.x * NUM_THREADS + threadIdx.x;
const uint token_head_threads = num_tokens * num_heads * threads_per_head;

if (global_idx >= token_head_threads) return;

const uint token_head_idx = global_idx / threads_per_head;
const uint pack_idx = global_idx % threads_per_head;

const uint token_idx = token_head_idx / num_heads;
const uint head_idx = token_head_idx % num_heads;

const uint pack_offset = pack_idx * pack_size;
const uint src_head_offset = token_idx * num_heads * prefix_head_stride +
                             head_idx * prefix_head_stride;
const uint dst_head_offset = token_idx * num_heads * output_head_stride +
                             head_idx * output_head_stride;
```
```cpp
if (token_idx >= prefix_num_tokens) {
  if (pack_offset < head_size) {
    input_pack_t s_out_pack = reinterpret_cast<const input_pack_t*>(
        suffix_head_ptr)[pack_offset / pack_size];
    ...
  }
  if (output_lse != nullptr && pack_idx == 0) {
    float s_lse = suffix_lse[head_idx * num_tokens + token_idx];
    output_lse[head_idx * num_tokens + token_idx] = s_lse;
  }
  return;
}
```
**EN:** A flat thread index is decoded into `(token_idx, head_idx, pack_idx)`, then into byte-addressable offsets for source and destination tensors. Tokens past `prefix_num_tokens` do not require merging, so the kernel copies the suffix state directly and writes the suffix LSE once per head.
**CN:** 内核先把扁平线程索引解码成 `(token_idx, head_idx, pack_idx)`，再换算成源张量和目标张量的偏移。对于 `prefix_num_tokens` 之后的 token，不需要执行合并，因此直接复制 suffix 状态，并且每个 head 只写一次 suffix 的 LSE。

### Stable LSE merge and all-`-inf` guard / 稳定 LSE 合并与全 `-inf` 保护
```cpp
float p_lse = prefix_lse[head_idx * num_tokens + token_idx];
float s_lse = suffix_lse[head_idx * num_tokens + token_idx];
p_lse = std::isinf(p_lse) ? -std::numeric_limits<float>::infinity() : p_lse;
s_lse = std::isinf(s_lse) ? -std::numeric_limits<float>::infinity() : s_lse;

const float max_lse = fmaxf(p_lse, s_lse);
...
if (std::isinf(max_lse)) {
  if (pack_offset < head_size) {
    input_pack_t p_out_pack = reinterpret_cast<const input_pack_t*>(
        prefix_head_ptr)[pack_offset / pack_size];
    ...
  }
  if (output_lse != nullptr && pack_idx == 0) {
    output_lse[head_idx * num_tokens + token_idx] = max_lse;
  }
  return;
}
```
**EN:** The merge is driven by prefix/suffix log-sum-exp values. The kernel normalizes any infinite inputs, computes `max_lse` for the standard stable-softmax trick, and explicitly handles the degenerate `p_lse = s_lse = -inf` case by emitting the prefix path. That avoids NaNs later in the pipeline for empty-prefix MLA edge cases.
**CN:** 合并逻辑由 prefix/suffix 的 log-sum-exp 值驱动。内核先规范化无穷值，再计算 `max_lse` 来应用稳定 softmax 的常见技巧，并专门处理 `p_lse = s_lse = -inf` 的退化情况：直接输出 prefix 路径。这可以避免 MLA 的空 prefix 边界场景在后续流水线中产生 NaN。

### Weighted value merge and optional FP8 store / 加权值合并与可选 FP8 存储
```cpp
p_lse = p_lse - max_lse;
s_lse = s_lse - max_lse;
const float p_se = expf(p_lse);
const float s_se = expf(s_lse);
const float out_se = p_se + s_se;
const float p_scale = p_se / out_se;
const float s_scale = s_se / out_se;
...
for (uint i = 0; i < pack_size; ++i) {
  const float p_out_f =
      vllm::to_float(reinterpret_cast<const scalar_t*>(&p_out_pack)[i]);
  const float s_out_f =
      vllm::to_float(reinterpret_cast<const scalar_t*>(&s_out_pack)[i]);
  o_out_f[i] = p_out_f * p_scale + (s_out_f * s_scale);
}
...
float out_lse = logf(out_se) + max_lse;
output_lse[head_idx * num_tokens + token_idx] = out_lse;
```
**EN:** After shifting by `max_lse`, the kernel reconstructs the relative softmax weights in float32 and applies them elementwise to prefix and suffix outputs. The merged vector is then stored either in the original scalar type or converted to FP8 using `scaled_fp8_conversion`, and the merged LSE is reconstructed as `log(out_se) + max_lse`.
**CN:** 在减去 `max_lse` 之后，内核用 float32 重建相对 softmax 权重，并逐元素对 prefix 和 suffix 输出做加权。合并后的向量要么按原始标量类型写回，要么通过 `scaled_fp8_conversion` 转成 FP8；对应的合并后 LSE 则重建为 `log(out_se) + max_lse`。

### Host launcher and FP8 dispatch / Host 启动器与 FP8 分发
```cpp
template <typename scalar_t>
void merge_attn_states_launcher(
    torch::Tensor& output, std::optional<torch::Tensor> output_lse,
    const torch::Tensor& prefix_output, const torch::Tensor& prefix_lse,
    const torch::Tensor& suffix_output, const torch::Tensor& suffix_lse,
    const std::optional<int64_t> prefill_tokens_with_context,
    const std::optional<torch::Tensor>& output_scale) {
  constexpr uint NUM_THREADS = 128;
  const uint num_tokens = output.size(0);
  const uint num_heads = output.size(1);
  const uint head_size = output.size(2);
  ...
  dim3 block(NUM_THREADS);
  dim3 grid((total_threads + NUM_THREADS - 1) / NUM_THREADS);
  ...
  if (output_scale.has_value()) {
    VLLM_DISPATCH_FP8_TYPES(output.scalar_type(), "merge_attn_states_fp8", [&] {
      LAUNCH_MERGE_ATTN_STATES(scalar_t, fp8_t, NUM_THREADS, true);
    });
  } else {
    LAUNCH_MERGE_ATTN_STATES(scalar_t, scalar_t, NUM_THREADS, false);
  }
}
```
**EN:** The host-side launcher derives tensor sizes/strides, checks that `head_size` is pack-aligned, computes the 1D launch grid, guards the CUDA device, and dispatches either the original output type path or an FP8-specialized path. The decision point is `output_scale`: if present, output storage must be FP8.
**CN:** Host 侧启动器负责提取张量的尺寸与 stride，检查 `head_size` 是否与打包粒度对齐，计算一维 launch grid，设置 CUDA 设备保护，并在原始输出类型路径与 FP8 专用路径之间做分发。分发的关键开关是 `output_scale`：只要提供它，输出存储就必须是 FP8。

### Public API validation / 对外 API 校验
```cpp
void merge_attn_states(torch::Tensor& output,
                       std::optional<torch::Tensor> output_lse,
                       const torch::Tensor& prefix_output,
                       const torch::Tensor& prefix_lse,
                       const torch::Tensor& suffix_output,
                       const torch::Tensor& suffix_lse,
                       std::optional<int64_t> prefill_tokens_with_context,
                       const std::optional<torch::Tensor>& output_scale) {
  if (output_scale.has_value()) {
    TORCH_CHECK(output.scalar_type() == at::ScalarType::Float8_e4m3fn ||
                    output.scalar_type() == at::ScalarType::Float8_e4m3fnuz,
                "output must be FP8 when output_scale is provided, got: ",
                output.scalar_type());
  } else {
    TORCH_CHECK(output.scalar_type() == prefix_output.scalar_type(),
                "output dtype (", output.scalar_type(),
                ") must match prefix_output dtype (",
                prefix_output.scalar_type(), ") when output_scale is not set");
  }
  DISPATCH_BY_SCALAR_DTYPE(prefix_output.dtype(),
                           CALL_MERGE_ATTN_STATES_LAUNCHER);
}
```
**EN:** The exported function enforces the output dtype contract before launching CUDA code. In FP8 mode, only the supported FP8 scalar types are accepted; otherwise, `output` must match `prefix_output`. Dispatch is intentionally keyed off the input tensor dtype because the computation reads prefix/suffix states in that type.
**CN:** 这个导出函数会在启动 CUDA 代码之前先校验输出类型约束。若启用 FP8 模式，只接受受支持的 FP8 标量类型；否则 `output` 必须与 `prefix_output` 的类型一致。分发时特意以输入张量类型为准，因为实际计算是按该类型读取 prefix/suffix 状态的。

## Key Concepts / 关键概念
- **EN:** Split-KV attention can produce partial outputs that must be recombined using LSE metadata rather than naïve averaging.  
  **CN:** Split-KV attention 会生成多个局部结果，重新组合时必须依赖 LSE 元数据，而不能简单做平均。
- **EN:** The kernel uses the standard `max + exp` stabilization pattern to avoid overflow/underflow during merge.  
  **CN:** 内核使用标准的 `max + exp` 数值稳定模式，避免合并时出现上溢或下溢。
- **EN:** Memory traffic is vectorized through 128-bit loads/stores, which is why `head_size` must be divisible by the pack size.  
  **CN:** 内存访问通过 128-bit 向量化读写完成，因此 `head_size` 必须能被打包大小整除。
- **EN:** FP8 output is purely a storage/output path here; the merge math itself still happens in float32.  
  **CN:** 这里的 FP8 仅用于输出存储路径；实际合并计算仍然在 float32 中完成。

## Dependencies / 依赖关系
- **EN:** `attention_dtypes.h` and `attention_utils.cuh` provide dtype helpers such as `to_float` / `from_float`.  
  **CN:** `attention_dtypes.h` 与 `attention_utils.cuh` 提供了 `to_float` / `from_float` 之类的类型转换辅助函数。
- **EN:** `../quantization/w8a8/fp8/common.cuh` supplies `scaled_fp8_conversion` for FP8 quantized stores.  
  **CN:** `../quantization/w8a8/fp8/common.cuh` 提供 `scaled_fp8_conversion`，用于 FP8 量化写回。
- **EN:** `../dispatch_utils.h` contributes `VLLM_DISPATCH_FP8_TYPES`, which selects the concrete FP8 output type at compile time.  
  **CN:** `../dispatch_utils.h` 提供 `VLLM_DISPATCH_FP8_TYPES`，用于在编译期选择具体的 FP8 输出类型。
- **EN:** `torch/all.h`, `ATen/cuda/CUDAContext.h`, and `c10/cuda/CUDAGuard.h` are used for tensor access, current-stream lookup, and device guarding.  
  **CN:** `torch/all.h`、`ATen/cuda/CUDAContext.h` 与 `c10/cuda/CUDAGuard.h` 用于张量访问、当前 stream 查询以及设备保护。
