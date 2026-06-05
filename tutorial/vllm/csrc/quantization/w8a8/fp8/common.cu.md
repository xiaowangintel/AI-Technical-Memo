# common.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/w8a8/fp8/common.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements FP8 quantization kernels for static scale, dynamic global scale, and dynamic per-token scale, plus the PyTorch-facing launchers that infer scale layouts. / [CN] 实现静态 scale、动态全局 scale、动态逐 token scale 的 FP8 量化内核，以及面向 PyTorch 的启动器（负责推导 scale 布局）。

## Line-by-Line Analysis / 逐行分析
### Strided/grouped static quantization kernel / 支持 stride 与分组的静态量化内核
```cpp
template <typename scalar_t, typename fp8_type, bool STRIDE_I_ZERO,
          bool STRIDE_J_ZERO>
__global__ void scaled_fp8_quant_kernel_strided_group_shape(
    fp8_type* __restrict__ out, const scalar_t* __restrict__ input,
    const float* __restrict__ scale, int hidden_size, int64_t in_row_stride,
    int64_t out_row_stride, int group_m, int group_n, int64_t scale_stride_i,
    int64_t scale_stride_j) {
```
**EN:** This is the main static-scale kernel. It supports per-tensor, per-token, per-channel, and 2D grouped scaling using a single indexing scheme driven by `group_m`, `group_n`, and the scale strides.
**CN:** 这是静态 scale 路径的核心内核。它通过统一的索引方案以及 `group_m`、`group_n`、scale stride，支持 per-tensor、per-token、per-channel 和二维分组量化。

### Scale caching and vectorized fast paths / scale 缓存与向量化快路径
```cpp
const int64_t scale_row_base =
    STRIDE_I_ZERO ? 0
                  : static_cast<int>(token_idx) / group_m * scale_stride_i;
...
constexpr int VEC_SIZE = 16;  // FP8 so vectorize to 128 bits
...
if (STRIDE_J_ZERO && hidden_size % VEC_SIZE == 0) {
  scaled_fp8_conversion_vectorized(token_in, token_out, hidden_size,
                                   get_inv_scale(0));
} else if (group_n % VEC_SIZE == 0) {
  ...
} else {
  ...
}
```
**EN:** The template booleans let the compiler remove row or column stride work when a dimension is broadcast. The kernel then chooses between whole-row vectorization, per-group vectorization, or a scalar fallback for small `group_n`.
**CN:** 模板布尔参数允许编译器在某个维度被广播时直接消除对应的 stride 计算。之后内核会在整行向量化、按组向量化、以及 `group_n` 较小时的标量回退路径之间进行选择。

### Global absmax reduction for dynamic scale / 动态全局 scale 的全局绝对值最大归约
```cpp
template <typename scalar_t, typename fp8_type>
__global__ void segmented_max_reduction_strided(
    float* __restrict__ scale, const scalar_t* __restrict__ input,
    int hidden_size, int64_t in_row_stride, int64_t num_tokens) {
  __shared__ float cache[256];
  ...
  if (tid == 0) {
    atomicMaxFloat(scale, cache[0] / quant_type_max_v<fp8_type>);
  }
}
```
**EN:** One block scans one row, reduces its absolute maximum in shared memory, and atomically updates a single global scale tensor. Dividing by `quant_type_max_v` converts an absmax into the scale required for FP8 saturation.
**CN:** 每个 block 扫描一行，在共享内存中归约出该行绝对值最大值，然后原子更新单个全局 scale 张量。通过除以 `quant_type_max_v`，绝对值最大值会被转换成 FP8 饱和所需的 scale。

### Second pass for dynamic global quantization / 动态全局量化的第二阶段
```cpp
template <typename scalar_t, typename fp8_type>
__global__ void scaled_fp8_quant_kernel_strided_dynamic(
    fp8_type* __restrict__ out, const scalar_t* __restrict__ input,
    const float* __restrict__ scale, int hidden_size, int64_t in_row_stride,
    int64_t out_row_stride) {
  const float reciprocal_scale = 1.0f / (*scale);
  vectorize_with_alignment<16>(...);
}
```
**EN:** After the reduction computes a single scale, this kernel performs the actual quantization for every row using that shared value.
**CN:** 在归约阶段求得单个全局 scale 之后，这个内核再用该共享值对所有行执行真正的量化。

### Per-token dynamic quantization in one kernel / 单内核逐 token 动态量化
```cpp
template <typename scalar_t, typename fp8_type>
__global__ void dynamic_per_token_scaled_fp8_quant_kernel_strided(
    fp8_type* __restrict__ out, float* __restrict__ scale,
    const scalar_t* __restrict__ input, const float* __restrict__ scale_ub,
    int hidden_size, int64_t in_row_stride, int64_t out_row_stride) {
  ...
  const float block_max =
      BlockReduce(tmp).Reduce(absmax_val, CubMaxOp{}, blockDim.x);
  ...
  scale[token_idx] = token_scale;
  ...
}
```
**EN:** This kernel fuses per-row absmax discovery and quantization. It optionally clamps the discovered scale by `scale_ub`, applies a minimum scaling floor, stores one scale per token, and then quantizes the same row.
**CN:** 这个内核把逐行 absmax 发现与量化融合到一起。它可以用 `scale_ub` 对 scale 做上限裁剪，再施加最小 scale 下界，为每个 token 存一份 scale，然后立刻量化同一行。

### Host-side inference of scale layout / 主机侧推导 scale 布局
```cpp
void static_scaled_fp8_quant(
    torch::Tensor& out,
    torch::Tensor const& input,
    torch::Tensor const& scale,
    std::optional<std::tuple<int64_t, int64_t>> opt_group_shape) {
  ...
  if (scale.dim() == 0 || scale.numel() == 1) {
    group_m = num_tokens;
    group_n = hidden_size;
    scale_stride_i = 0;
    scale_stride_j = 0;
  } else if (scale.dim() == 1) {
    ...
  } else if (scale.dim() == 2) {
    ...
  }
}
```
**EN:** The host API accepts 0D, 1D, or 2D scale tensors and normalizes them into a common `(group_m, group_n, stride_i, stride_j)` description. This is the key step that lets one device kernel serve multiple quantization modes.
**CN:** 主机 API 接受 0D、1D 或 2D 的 scale 张量，并将其统一归一化为 `(group_m, group_n, stride_i, stride_j)` 描述。这是单个设备内核能够覆盖多种量化模式的关键步骤。

### Dispatch specialization and launcher paths / 特化分发与启动路径
```cpp
VLLM_DISPATCH_FLOATING_TYPES(
    input.scalar_type(), "scaled_fp8_quant_kernel_scalar_type", [&] {
      VLLM_DISPATCH_FP8_TYPES(
          out.scalar_type(), "scaled_fp8_quant_kernel_fp8_type", [&] {
            VLLM_DISPATCH_BOOL(scale_stride_i == 0, S0_ZERO, [&] {
              VLLM_DISPATCH_BOOL(scale_stride_j == 0, S1_ZERO, [&] {
                vllm::scaled_fp8_quant_kernel_strided_group_shape<
                    scalar_t, fp8_t, S0_ZERO, S1_ZERO>
```
**EN:** Runtime tensor dtypes become compile-time template instantiations through nested dispatch macros. The boolean dispatch is especially important because it removes scale-broadcast branches inside the hot kernel.
**CN:** 通过嵌套的 dispatch 宏，运行时张量 dtype 会被映射成编译期模板实例。布尔分发尤其重要，因为它能把 scale 广播相关的分支从热点内核中消除掉。

### Dynamic launchers / 动态量化启动器
```cpp
AT_CUDA_CHECK(
    cudaMemsetAsync(scale.data_ptr<float>(), 0, sizeof(float), stream));
...
vllm::segmented_max_reduction_strided<scalar_t, fp8_t><<<grid, block, 0, stream>>>(...);
vllm::scaled_fp8_quant_kernel_strided_dynamic<scalar_t, fp8_t><<<grid, block, 0, stream>>>(...);
```
**EN:** The global-dynamic path is explicitly two-pass: clear the output scale tensor, reduce absmax into it, then quantize. The per-token path instead launches a single fused kernel.
**CN:** 全局动态路径明确采用两阶段：先清空输出 scale 张量，再归约 absmax，最后执行量化。逐 token 路径则启动一个融合内核完成全部工作。

## Key Concepts / 关键概念
- **Unified scale layout model**: per-tensor, per-token, per-channel, and 2D grouped scaling all map to the same internal kernel parameters. / **统一的 scale 布局模型**：per-tensor、per-token、per-channel 与二维分组量化最终都映射到同一套内核参数。
- **Vectorization-first kernel design**: the kernels prefer 128-bit vectorized paths and fall back to scalar logic only when grouping prevents it. / **向量化优先的内核设计**：内核优先使用 128-bit 向量化路径，只有在分组方式不允许时才退回标量逻辑。
- **Two dynamic modes**: global dynamic quantization uses a separate reduction pass, while per-token dynamic quantization fuses reduction and encoding. / **两种动态模式**：全局动态量化使用独立归约阶段，而逐 token 动态量化将归约和编码融合。

## Dependencies / 依赖关系
- `common.cuh` provides `scaled_fp8_conversion`, `atomicMaxFloat`, and backend FP8 conversion support. / `common.cuh` 提供 `scaled_fp8_conversion`、`atomicMaxFloat` 以及后端 FP8 转换支持。
- `dispatch_utils.h` and `cub_helpers.h` provide dtype dispatch macros and CUB reductions. / `dispatch_utils.h` 与 `cub_helpers.h` 提供 dtype 分发宏和 CUB 归约支持。
- `libtorch_stable/quantization/vectorization_utils.cuh` supplies aligned vectorized load/store helpers. / `libtorch_stable/quantization/vectorization_utils.cuh` 提供对齐向量化读写辅助函数。
