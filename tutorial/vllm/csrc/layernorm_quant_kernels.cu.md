# layernorm_quant_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/layernorm_quant_kernels.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Implements RMSNorm and fused residual-add RMSNorm kernels that directly emit statically scaled FP8 output instead of an intermediate floating-point tensor. / [CN] 实现直接输出静态缩放 FP8 结果的 RMSNorm 与 fused residual-add RMSNorm 内核，而不是先生成中间浮点张量。

## Line-by-Line Analysis / 逐行分析

### File scope and FP8 support / 文件范围与 FP8 支持
```cpp
/*
 * This file contains the CUDA kernels for the fused quantized layernorm.
 * The kernels correspond to the kernels in layernorm_kernels.cu, except they
 * also produce quantized output directly.
 * Currently, only static fp8 quantization is supported.
 */

#include "type_convert.cuh"
#include "quantization/w8a8/fp8/common.cuh"
#include "dispatch_utils.h"
#include "cub_helpers.h"
#include "core/batch_invariant.hpp"
```
**EN:** This file mirrors `layernorm_kernels.cu`, but adds direct FP8 conversion. The extra FP8 header provides the conversion primitive used to quantize normalized values according to a precomputed scale tensor.
**CN:** 该文件与 `layernorm_kernels.cu` 在结构上基本对应，但增加了直接 FP8 转换。额外引入的 FP8 头文件提供量化原语，用于按预先给定的缩放张量把归一化结果写成 FP8。

### RMSNorm + static FP8 quantization / RMSNorm + 静态 FP8 量化
```cpp
template <typename scalar_t, typename fp8_type, int VEC_SIZE>
__global__ void rms_norm_static_fp8_quant_kernel(
    fp8_type* __restrict__ out,
    const scalar_t* __restrict__ input,
    const int input_stride,
    const scalar_t* __restrict__ weight,
    const float* __restrict__ scale,
    const float epsilon, const int num_tokens, const int hidden_size) {
  __shared__ float s_variance;
  float variance = 0.0f;
  const scalar_t* input_row = input + blockIdx.x * input_stride;
  ...
  if (threadIdx.x == 0) {
    s_variance = rsqrtf(variance / hidden_size + epsilon);
  }
  __syncthreads();

  float const scale_inv = 1.0f / *scale;
```
**EN:** The normalization phase is the same as standard RMSNorm: compute the row sum of squares, reduce it, and derive an RMS reciprocal. The key difference is the additional `scale` parameter, which controls the final FP8 quantization range.
**CN:** 归一化阶段与普通 RMSNorm 相同：先计算每一行的平方和，再归约得到 RMS 倒数。关键差异在于额外的 `scale` 参数，它决定最终 FP8 量化时所使用的数值范围。

### Precision-matching before FP8 conversion / FP8 转换前的精度匹配
```cpp
for (int idx = threadIdx.x; idx < hidden_size / VEC_SIZE; idx += blockDim.x) {
  vec_n_t<scalar_t, VEC_SIZE> src1 = v_in[idx];
  vec_n_t<scalar_t, VEC_SIZE> src2 = v_w[idx];
#pragma unroll
  for (int j = 0; j < VEC_SIZE; j++) {
    float x = static_cast<float>(src1.val[j]);
    float w = static_cast<float>(src2.val[j]);
    scalar_t out_norm = static_cast<scalar_t>(x * s_variance * w);
    out[blockIdx.x * hidden_size + idx * VEC_SIZE + j] =
        scaled_fp8_conversion<true, fp8_type>(static_cast<float>(out_norm),
                                              scale_inv);
  }
}
```
**EN:** The code intentionally rounds the normalized result through `scalar_t` before calling `scaled_fp8_conversion`. That preserves bitwise agreement with the unfused two-step path (`rms_norm` then `static_scaled_fp8_quant`), especially around exact FP8 tie boundaries.
**CN:** 这里特意先把归一化结果回写到 `scalar_t` 精度，再调用 `scaled_fp8_conversion`。这样可以让 fused 路径与拆分路径（`rms_norm` 再 `static_scaled_fp8_quant`）在 FP8 边界值附近保持一致，尤其是精确舍入临界点处。

### Vectorized fused residual path / 向量化 fused residual 路径
```cpp
template <typename scalar_t, int width, typename fp8_type>
__global__ std::enable_if_t<(width > 0) && _typeConvert<scalar_t>::exists>
fused_add_rms_norm_static_fp8_quant_kernel(
    fp8_type* __restrict__ out,
    scalar_t* __restrict__ input,
    const int input_stride,
    scalar_t* __restrict__ residual,
    const scalar_t* __restrict__ weight,
    const float* __restrict__ scale,
    const float epsilon, const int num_tokens, const int hidden_size) {
  ...
  _f16Vec<scalar_t, width> temp = input_v[stride_id];
  temp += residual_v[id];
  variance += temp.sum_squares();
  residual_v[id] = temp;
```
**EN:** This specialization performs three fused actions: residual accumulation, RMS statistics collection, and FP8 emission. It uses packed half/bfloat vectors to keep the residual path bandwidth-efficient.
**CN:** 这个特化一次完成三件事：residual 累加、RMS 统计收集以及 FP8 输出。它借助打包的 half/bfloat 向量，尽量让 residual 路径维持高带宽效率。

### HIP/CUDA portability detail / HIP/CUDA 可移植性细节
```cpp
using Converter = _typeConvert<scalar_t>;
using HipT = typename Converter::hip_type;
...
HipT out_norm_h = Converter::convert(x * s_variance * wf);
out[id * width + i] = scaled_fp8_conversion<true, fp8_type>(
    Converter::convert(out_norm_h), scale_inv);
```
**EN:** Instead of directly using `c10::Half`/`BFloat16` as the intermediate, the kernel converts through the backend-specific `hip_type`. This avoids ambiguous CUDA conversions and ROCm incompatibilities while still reproducing the intended rounding behavior.
**CN:** 内核没有直接把 `c10::Half`/`BFloat16` 作为中间类型，而是通过后端相关的 `hip_type` 进行转换。这样既避免了 CUDA 下的歧义转换，也兼容 ROCm，同时仍保持预期的舍入语义。

### Generic scalar fallback / 通用标量回退
```cpp
template <typename scalar_t, int width, typename fp8_type>
__global__ std::enable_if_t<(width == 0) || !_typeConvert<scalar_t>::exists>
fused_add_rms_norm_static_fp8_quant_kernel(
    fp8_type* __restrict__ out,
    scalar_t* __restrict__ input,
    const int input_stride,
    scalar_t* __restrict__ residual,
    const scalar_t* __restrict__ weight,
    const float* __restrict__ scale,
    const float epsilon, const int num_tokens, const int hidden_size) {
  ...
  scalar_t out_norm = static_cast<scalar_t>(x * s_variance * w);
  out[blockIdx.x * hidden_size + idx] = scaled_fp8_conversion<true, fp8_type>(
      static_cast<float>(out_norm), scale_inv);
}
```
**EN:** The fallback preserves exactly the same numerical ordering: add residual, compute RMS scale, round back through `scalar_t`, and finally quantize to FP8. Only the memory access strategy changes.
**CN:** 回退路径保持完全相同的数值顺序：先加 residual，再计算 RMS 缩放，再回落到 `scalar_t` 精度，最后量化为 FP8。变化的只有内存访问策略。

### Host dispatch and fast-path gating / 主机分发与快速路径门控
```cpp
VLLM_DISPATCH_FLOATING_TYPES(
    input.scalar_type(), "rms_norm_kernel_scalar_type", [&] {
      VLLM_DISPATCH_FP8_TYPES(
          out.scalar_type(), "rms_norm_kernel_fp8_type", [&] {
            const int calculated_vec_size =
                std::gcd(16 / sizeof(scalar_t), hidden_size);
```
**EN:** Host code dispatches over both input floating type and output FP8 format, because the kernel template depends on both. As in the non-quantized file, vector width is derived from hidden-size divisibility and 128-bit alignment assumptions.
**CN:** 主机侧同时对输入浮点类型和输出 FP8 格式进行分发，因为内核模板同时依赖这两者。与非量化版本相同，向量宽度由 hidden size 的可整除性和 128-bit 对齐前提共同决定。

```cpp
bool ptrs_are_aligned =
    inp_ptr % 16 == 0 && res_ptr % 16 == 0 && wt_ptr % 16 == 0;
bool batch_invariant_launch = vllm::vllm_is_batch_invariant();
if (ptrs_are_aligned && hidden_size % 8 == 0 && input_stride % 8 == 0 &&
    !batch_invariant_launch) {
  LAUNCH_FUSED_ADD_RMS_NORM(8);
} else {
  LAUNCH_FUSED_ADD_RMS_NORM(0);
}
```
**EN:** The fused FP8 path only uses width-8 packed kernels when all pointers are 16-byte aligned and tensor extents match that packing. Otherwise it falls back to width 0, which means scalarized logic.
**CN:** fused FP8 路径只有在所有指针都满足 16 字节对齐、并且张量尺寸也适合 width-8 打包时才使用向量化内核；否则回退到 `width=0` 的标量逻辑。

## Key Concepts / 关键概念
- **EN:** This file fuses normalization and static FP8 quantization to avoid materializing an intermediate normalized tensor.  
  **CN:** 该文件把归一化和静态 FP8 量化融合起来，避免中间归一化张量的显式落地。
- **EN:** Numerical compatibility matters: rounding through `scalar_t` is deliberate, not redundant.  
  **CN:** 数值兼容性非常重要：先回到 `scalar_t` 再量化是有意设计，并非多余步骤。
- **EN:** The same fast/slow split exists as in the plain RMSNorm kernels: packed FP16/BF16 path versus generic scalar fallback.  
  **CN:** 与普通 RMSNorm 内核一样，这里也分为快速和回退两条路径：打包的 FP16/BF16 路径与通用标量回退路径。

## Dependencies / 依赖关系
- **EN:** Reuses `type_convert.cuh`, `dispatch_utils.h`, and `cub_helpers.h` from the non-quantized RMSNorm implementation.  
  **CN:** 复用了非量化 RMSNorm 实现中的 `type_convert.cuh`、`dispatch_utils.h` 和 `cub_helpers.h`。
- **EN:** Depends on `quantization/w8a8/fp8/common.cuh` for `scaled_fp8_conversion` and FP8 type dispatch.  
  **CN:** 依赖 `quantization/w8a8/fp8/common.cuh` 提供 `scaled_fp8_conversion` 与 FP8 类型支持。
- **EN:** Uses PyTorch CUDA guards/streams for correct device placement and stream selection.  
  **CN:** 使用 PyTorch CUDA 的设备守卫与流接口来保证设备上下文和流选择正确。
