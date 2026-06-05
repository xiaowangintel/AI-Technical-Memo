# fused_layernorm_dynamic_per_token_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/fused_kernels/fused_layernorm_dynamic_per_token_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Builds complete RMSNorm-plus-quantization CUDA kernels on top of `layernorm_utils.cuh`, supporting both per-token dynamic scales and per-block/grouped scales. / [CN] 基于 `layernorm_utils.cuh` 构建完整的 RMSNorm+量化 CUDA 内核，同时支持按 token 的动态 scale 与按块/分组的 scale。

## Line-by-Line Analysis / 逐行分析

### Vectorized RMSNorm helper
```cpp
template <typename scalar_t, typename scalar_out_t, bool has_residual = false>
__device__ void rms_norm_dynamic_per_token_quant_vec(
    scalar_out_t* __restrict__ out,       // [..., hidden_size]
    float* __restrict__ scales,           // [num_tokens]
    scalar_t const* __restrict__ input,   // [..., hidden_size]
    scalar_t const* __restrict__ weight,  // [hidden_size]
    float const* scale_ub, float const var_epsilon, int32_t const hidden_size,
    int32_t const input_stride, scalar_t* __restrict__ residual = nullptr) {
  float rms = 0.0f;
  float token_scale = 0.0f;

  // Compute rms
  vllm::vectorized::compute_rms<scalar_t, has_residual>(
      &rms, input, hidden_size, input_stride, var_epsilon, residual);

  // Compute scale
  vllm::vectorized::compute_dynamic_per_token_scales<scalar_t, scalar_out_t,
                                                     has_residual>(
      &token_scale, scales, input, weight, rms, scale_ub, hidden_size,
      input_stride, residual);

  // RMS Norm + Quant
  if constexpr (std::is_same_v<scalar_out_t, int8_t>) {
    token_scale = 1.0f / token_scale;
    vllm::vectorized::norm_and_quant<scalar_t, scalar_out_t, true,
                                     has_residual>(out, input, weight, rms,
                                                   &token_scale, hidden_size,
                                                   input_stride, residual);
  } else {
    // FP8 - Do not invert token_scale for exact match with FBGemm
    vllm::vectorized::norm_and_quant<scalar_t, scalar_out_t, false,
                                     has_residual>(out, input, weight, rms,
                                                   &token_scale, hidden_size,
                                                   input_stride, residual);
  }
```
**EN:** `rms_norm_dynamic_per_token_quant_vec` is the fast path for contiguous hidden sizes divisible by 4. It chains the vectorized RMS calculation, vectorized dynamic scale computation, and vectorized quantized writeback. For int8 it explicitly inverts the token scale before quantization; for FP8 it preserves the original scale to match FBGemm semantics.
**CN:** `rms_norm_dynamic_per_token_quant_vec` 是面向“隐藏维可被 4 整除且布局连续”场景的快速路径。它把向量化的 RMS 计算、向量化动态 scale 计算以及向量化量化写回串联起来。对于 int8，它会在量化前显式把 token scale 取倒数；对于 FP8，则保持原始 scale 以匹配 FBGemm 语义。

### Fallback-aware dynamic kernel
```cpp
// RMS norm + quant kernel
template <typename scalar_t, typename scalar_out_t, bool has_residual = false>
__global__ void rms_norm_dynamic_per_token_quant_kernel(
    scalar_out_t* __restrict__ out,       // [..., hidden_size]
    float* __restrict__ scales,           // [num_tokens]
    scalar_t const* __restrict__ input,   // [..., hidden_size]
    scalar_t const* __restrict__ weight,  // [hidden_size]
    float const* scale_ub, float const var_epsilon, int32_t const hidden_size,
    int32_t const input_stride, scalar_t* __restrict__ residual = nullptr) {
  // For vectorization, token_input and token_output pointers need to be
  // aligned at 8-byte and 4-byte addresses respectively.
  bool const can_vectorize = hidden_size % 4 == 0 and input_stride % 4 == 0;

  if (can_vectorize) {
    return rms_norm_dynamic_per_token_quant_vec<scalar_t, scalar_out_t,
                                                has_residual>(
        out, scales, input, weight, scale_ub, var_epsilon, hidden_size,
        input_stride, residual);
  }

  float rms = 0.0f;
  float token_scale = 0.0f;

  // Compute RMS
  vllm::compute_rms<scalar_t, has_residual>(
      &rms, input, hidden_size, input_stride, var_epsilon, residual);
  // Compute Scale
  vllm::compute_dynamic_per_token_scales<scalar_t, scalar_out_t, has_residual>(
      &token_scale, scales, input, weight, rms, scale_ub, hidden_size,
      input_stride, residual);

  // RMS Norm + Quant
  if constexpr (std::is_same_v<scalar_out_t, int8_t>) {
    token_scale = 1.0f / token_scale;
    vllm::norm_and_quant<scalar_t, scalar_out_t, true, has_residual>(
        out, input, weight, rms, &token_scale, hidden_size, input_stride,
        residual);
  } else {
    // FP8 - Do not invert s_token_scale for exact match with FBGemm
    vllm::norm_and_quant<scalar_t, scalar_out_t, false, has_residual>(
        out, input, weight, rms, &token_scale, hidden_size, input_stride,
        residual);
  }
```
**EN:** The global kernel checks whether vectorization is legal; if yes, it immediately tail-calls the vectorized helper. Otherwise it falls back to the scalar helpers from `layernorm_utils.cuh` while keeping the same three-stage sequence: RMS, scale, quantize.
**CN:** 该全局内核先检查是否满足向量化条件；如果满足，就直接转入向量化辅助函数。否则退回到 `layernorm_utils.cuh` 中的标量辅助逻辑，但仍保持同样的三阶段流程：RMS、scale、量化。

### Per-block quant kernel
```cpp
// RMS norm + quant kernel
template <typename scalar_t, typename scalar_out_t, bool has_residual = false,
          bool is_scale_transposed = false, int32_t group_size = 0>
__global__ void rms_norm_per_block_quant_kernel(
    scalar_out_t* __restrict__ out,  // [..., hidden_size]
    float* __restrict__ scales,      // [num_tokens, hidden_size / group_size]
                                     // or
                                     // [hidden_size / group_size, num_tokens]
    scalar_t const* __restrict__ input,   // [..., hidden_size]
    scalar_t const* __restrict__ weight,  // [hidden_size]
    float const* scale_ub, float const var_epsilon, int32_t const hidden_size,
    int32_t const input_stride, scalar_t* __restrict__ residual = nullptr,
    int64_t outer_scale_stride = 1) {
  float rms;
  // Compute RMS
  // Always able to vectorize due to constraints on hidden_size
  vllm::vectorized::compute_rms<scalar_t, has_residual>(
      &rms, input, hidden_size, input_stride, var_epsilon, residual);

  // Compute Scale
  // Always able to vectorize due to constraints on hidden_size and group_size
  vllm::vectorized::compute_dynamic_per_token_scales<
      scalar_t, scalar_out_t, has_residual, is_scale_transposed, group_size>(
      nullptr, scales, input, weight, rms, scale_ub, hidden_size, input_stride,
      residual, outer_scale_stride);

  // RMS Norm + Quant
  // Always able to vectorize due to constraints on hidden_size
  // For int8, don't invert token_scale here: do it inside the norm_and_quant
  // kernel. We do it because particular elements of token_scale can be shared
  // between multiple threads, so this way, we avoid extra synchronization
  // overhead.
  vllm::vectorized::norm_and_quant<
      scalar_t, scalar_out_t, std::is_same_v<scalar_out_t, int8_t>,
      has_residual, is_scale_transposed, group_size>(
      out, input, weight, rms, scales, hidden_size, input_stride, residual,
      outer_scale_stride);
}
```
**EN:** `rms_norm_per_block_quant_kernel` is the grouped-scale variant. It always uses the vectorized helpers, computes one scale per hidden-state block, and then quantizes with the grouped scale layout. For int8, scale inversion is deferred into the final quantization loop to avoid extra synchronization on shared scale values.
**CN:** `rms_norm_per_block_quant_kernel` 是分组 scale 版本。它始终使用向量化辅助函数，为每个隐藏维块计算一个 scale，然后按分组布局完成量化。对 int8 而言，scale 的倒数会推迟到最后的量化循环中处理，以避免共享 scale 值上的额外同步。

### Dynamic per-token dispatch helper
```cpp
template <typename scalar_in_t>
void rms_norm_dynamic_per_token_quant_dispatch(
    torch::Tensor& out,           // [..., hidden_size]
    torch::Tensor const& input,   // [..., hidden_size]
    torch::Tensor const& weight,  // [hidden_size]
    torch::Tensor& scales,        // [num_tokens]
    double const var_epsilon,     // Variance epsilon used in norm calculation
    std::optional<at::Tensor> const& scale_ub,
    std::optional<at::Tensor>& residual) {
  int32_t hidden_size = input.size(-1);
  int32_t input_stride = input.view({-1, hidden_size}).stride(0);
  auto num_tokens = input.numel() / hidden_size;

  dim3 grid(num_tokens);
  dim3 block(std::min(hidden_size, 1024));
  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  VLLM_DISPATCH_BOOL(residual.has_value(), has_residual, [&] {
    VLLM_DISPATCH_QUANT_TYPES(
        out.scalar_type(), "rms_norm_dynamic_per_token_quant_kernel", [&] {
          vllm::rms_norm_dynamic_per_token_quant_kernel<scalar_in_t, scalar_t,
                                                        has_residual>
              <<<grid, block, 0, stream>>>(
                  out.data_ptr<scalar_t>(), scales.data_ptr<float>(),
                  input.data_ptr<scalar_in_t>(), weight.data_ptr<scalar_in_t>(),
                  scale_ub.has_value() ? scale_ub->data_ptr<float>() : nullptr,
                  var_epsilon, hidden_size, input_stride,
                  has_residual ? residual->data_ptr<scalar_in_t>() : nullptr);
        });
  });
}
```
**EN:** The dispatch helper derives `hidden_size`, `input_stride`, `num_tokens`, picks a 1D launch, and instantiates the dynamic kernel over input type, output quant type, and residual/no-residual variants.
**CN:** 该 dispatch 辅助函数会推导 `hidden_size`、`input_stride`、`num_tokens`，选择一维 launch，并在输入类型、输出量化类型以及有无 residual 之间实例化动态量化内核。

### Public dynamic API checks
```cpp
void rms_norm_dynamic_per_token_quant(
    torch::Tensor& out,           // [..., hidden_size]
    torch::Tensor const& input,   // [..., hidden_size]
    torch::Tensor const& weight,  // [hidden_size]
    torch::Tensor& scales,        // [num_tokens]
    double const var_epsilon,     // Variance epsilon used in norm calculation
    std::optional<at::Tensor> scale_ub, std::optional<at::Tensor> residual) {
  static c10::ScalarType kFp8Type = is_fp8_ocp()
                                        ? c10::ScalarType::Float8_e4m3fn
                                        : c10::ScalarType::Float8_e4m3fnuz;
  TORCH_CHECK(out.dtype() == kFp8Type || out.dtype() == torch::kInt8);
  TORCH_CHECK(out.is_contiguous());
  TORCH_CHECK(input.stride(-1) == 1,
              "Input must be contiguous in the last dimension");

  if (scale_ub.has_value()) {
    TORCH_CHECK(out.dtype() == kFp8Type);
  }
  TORCH_CHECK(weight.dtype() == input.dtype());
  TORCH_CHECK(scales.dtype() == torch::kFloat32);
  if (residual) {
    TORCH_CHECK(residual->scalar_type() == input.scalar_type());
    TORCH_CHECK(residual->is_contiguous());
  }

  VLLM_DISPATCH_FLOATING_TYPES(
      input.scalar_type(), "rms_norm_dynamic_per_token_quant_dispatch", [&] {
        rms_norm_dynamic_per_token_quant_dispatch<scalar_t>(
            out, input, weight, scales, var_epsilon, scale_ub, residual);
      });
}
```
**EN:** The public `rms_norm_dynamic_per_token_quant` entry point validates output dtype, contiguity, weight/scales compatibility, and optional residual constraints before calling the typed dispatcher.
**CN:** 公开接口 `rms_norm_dynamic_per_token_quant` 会先校验输出 dtype、连续性、weight/scales 的兼容性以及可选 residual 的约束，然后再调用带类型的分发函数。

### Per-block dispatch helper
```cpp
// Residual add + RMS norm + dynamic per token
void rms_norm_per_block_quant_dispatch(
    torch::Tensor& out,           // [..., hidden_size]
    torch::Tensor const& input,   // [..., hidden_size]
    torch::Tensor const& weight,  // [hidden_size]
    torch::Tensor& scales,        // [num_tokens, hidden_size / group_size] or
                                  // [hidden_size / group_size, num_tokens]
    int32_t group_size,
    double const var_epsilon,  // Variance epsilon used in norm calculation
    std::optional<at::Tensor> const& scale_ub,
    std::optional<at::Tensor>& residual, bool is_scale_transposed) {
  int32_t hidden_size = input.size(-1);
  int32_t input_stride = input.view({-1, hidden_size}).stride(0);

  TORCH_CHECK(hidden_size % 4 == 0,
              "Hidden size must be divisible by 4 for vectorized access");
  TORCH_CHECK(input_stride % 4 == 0,
              "Input stride must be divisible by 4 for vectorized access");
  TORCH_CHECK(group_size % 4 == 0,
              "Group size must be divisible by 4 for vectorized access");

  auto num_tokens = input.numel() / hidden_size;

  dim3 grid(num_tokens);
  const int max_block_size = (num_tokens <= 256) ? 512 : 256;
  dim3 block(std::min(hidden_size, max_block_size));
  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  VLLM_DISPATCH_FLOATING_TYPES(
      input.scalar_type(), "rms_norm_per_block_quant_fp_dispatch", [&] {
        using scalar_in_t = scalar_t;
        VLLM_DISPATCH_GROUP_SIZE(group_size, gs, [&] {
          VLLM_DISPATCH_BOOL(residual.has_value(), has_residual, [&] {
            VLLM_DISPATCH_BOOL(is_scale_transposed, transpose_scale, [&] {
              VLLM_DISPATCH_QUANT_TYPES(
                  out.scalar_type(), "rms_norm_per_block_quant_kernel", [&] {
                    vllm::rms_norm_per_block_quant_kernel<scalar_in_t, scalar_t,
                                                          has_residual,
                                                          transpose_scale, gs>
                        <<<grid, block, 0, stream>>>(
                            out.data_ptr<scalar_t>(), scales.data_ptr<float>(),
                            input.data_ptr<scalar_in_t>(),
                            weight.data_ptr<scalar_in_t>(),
                            scale_ub.has_value() ? scale_ub->data_ptr<float>()
                                                 : nullptr,
                            var_epsilon, hidden_size, input_stride,
                            has_residual ? residual->data_ptr<scalar_in_t>()
                                         : nullptr,
                            scales.stride(1));
                  });
            });
          });
        });
      });
}
```
**EN:** The grouped dispatcher enforces vectorization-friendly divisibility, chooses the block size based on token count, and dispatches over input type, group size, residual flag, transposed-scale flag, and output quant type.
**CN:** 分组 dispatcher 会强制检查适合向量化的整除条件，根据 token 数选择 block 大小，并在输入类型、group size、residual 标志、scale 是否转置以及输出量化类型之间做分发。

### Public per-block API checks
```cpp
void rms_norm_per_block_quant(torch::Tensor& out, torch::Tensor const& input,
                              torch::Tensor const& weight,
                              torch::Tensor& scales, double const var_epsilon,
                              std::optional<torch::Tensor> scale_ub,
                              std::optional<torch::Tensor> residual,
                              int64_t group_size, bool is_scale_transposed) {
  static c10::ScalarType kFp8Type = is_fp8_ocp()
                                        ? c10::ScalarType::Float8_e4m3fn
                                        : c10::ScalarType::Float8_e4m3fnuz;
  TORCH_CHECK(out.dtype() == kFp8Type || out.dtype() == torch::kInt8);
  TORCH_CHECK(out.is_contiguous());
  TORCH_CHECK(input.stride(-1) == 1,
              "Input must be contiguous in the last dimension");

  if (scale_ub.has_value()) {
    TORCH_CHECK(out.dtype() == kFp8Type);
  }
  TORCH_CHECK(weight.dtype() == input.dtype());
  TORCH_CHECK(scales.dtype() == torch::kFloat32);
  if (residual) {
    TORCH_CHECK(residual->scalar_type() == input.scalar_type());
    TORCH_CHECK(residual->is_contiguous());
  }

  TORCH_CHECK(group_size == 128 || group_size == 64,
              "Unsupported group size: ", group_size);

  if (scales.stride(1) > 1) {
    TORCH_CHECK(is_scale_transposed,
                "Outer scale stride must be 1 when scales are not transposed");
  }

  int64_t hidden_size = input.size(-1);
  TORCH_CHECK(hidden_size > 0 && hidden_size % group_size == 0,
              "hidden_size must be a positive multiple of group_size");
  int64_t num_tokens = input.numel() / hidden_size;
  int64_t num_groups = hidden_size / group_size;
  TORCH_CHECK(scales.numel() >= num_tokens * num_groups,
              "scales buffer too small: need ", num_tokens * num_groups,
              " elements, got ", scales.numel());

  rms_norm_per_block_quant_dispatch(out, input, weight, scales, group_size,
                                    var_epsilon, scale_ub, residual,
                                    is_scale_transposed);
```
**EN:** The public `rms_norm_per_block_quant` wrapper validates supported group sizes, hidden-size divisibility, scale buffer capacity, and stride rules for transposed layouts before launching the grouped kernel.
**CN:** 公开接口 `rms_norm_per_block_quant` 会在启动分组内核之前校验支持的 group size、隐藏维整除关系、scale 缓冲区容量，以及转置布局下的 stride 规则。

## Key Concepts / 关键概念
- EN: This file is mostly orchestration: the actual math lives in `layernorm_utils.cuh`.
  CN: 该文件主要负责流程编排，实际数学计算主要在 `layernorm_utils.cuh` 中完成。
- EN: Per-token and per-block quantization share the same RMSNorm core but differ in how scales are reduced and indexed.
  CN: 按 token 与按块量化共享同一套 RMSNorm 核心，但 scale 的规约方式和索引方式不同。
- EN: Int8 and FP8 deliberately use different scale-inversion conventions.
  CN: int8 与 FP8 在 scale 是否取倒数这件事上采用了刻意不同的约定。

## Dependencies / 依赖关系
- EN: Directly depends on `layernorm_utils.cuh` and `quant_conversions.cuh`.
  CN: 直接依赖 `layernorm_utils.cuh` 与 `quant_conversions.cuh`。
- EN: Uses ATen CUDA stream/device helpers and vLLM dispatch macros for runtime specialization.
  CN: 使用 ATen CUDA stream/device 辅助函数与 vLLM dispatch 宏做运行时特化。
- EN: Exposes PyTorch-callable wrappers around the CUDA kernels.
  CN: 对外暴露可被 PyTorch 调用的 CUDA 包装接口。
