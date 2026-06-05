# vec.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/vec.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-14)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

#pragma once

#if defined(__AVX512F__) && defined(__AVX512BF16__) && defined(__AMX_BF16__)
#define CPU_CAPABILITY_AVX512
#endif

#include <ATen/cpu/vec/functional.h>
#include <ATen/cpu/vec/vec.h>
#include <immintrin.h>
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: cvt_e4m3_bf16_intrinsic_with_denorm (lines 108-133)
```cpp
inline __m512bh cvt_e4m3_bf16_intrinsic_with_denorm(__m256i fp8_vec) {
  __m512i x = _mm512_cvtepu8_epi16(fp8_vec);
  __m512i lg2mant = _mm512_mask_mov_epi16(
      _mm512_mask_mov_epi16(
          _mm512_setzero_si512(), _mm512_test_epi16_mask(x, _mm512_set1_epi16(2)), _mm512_set1_epi16(1)),
      _mm512_test_epi16_mask(x, _mm512_set1_epi16(4)),
      _mm512_set1_epi16(2));
  return (__m512bh)(_mm512_or_si512(
      _mm512_maskz_mov_epi16(
          _mm512_cmpneq_epi16_mask(_mm512_and_si512(x, _mm512_set1_epi16(127)), _mm512_setzero_si512()),
          _mm512_mask_blend_epi16(
              _mm512_test_epi16_mask(x, _mm512_set1_epi16(120)),
              _mm512_or_si512(
                  _mm512_and_si512(
// ...
                      7)))),
      _mm512_slli_epi16(_mm512_and_si512(x, _mm512_set1_epi16(128)), 8)));
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: cvt_mxfp4_e2m1_bf16_intrinsic_lut (lines 174-203)
```cpp
inline std::tuple<__m512bh, __m512bh> cvt_mxfp4_e2m1_bf16_intrinsic_lut(__m256i a, __m512i s0, __m512i s1) {
  // LUT
  const __m512 values = _mm512_set_ps(MXFP4_VALUES);
  const __m512i lut = (__m512i)(_mm512_cvtne2ps_pbh(values, values));

  const __m512i abs_mask = _mm512_set1_epi16(0x7FFF);
  const __m512i zero = _mm512_setzero_si512();

  // expand values to 16-bit integers
  __m512i x0 = _mm512_cvtepu8_epi16(a);
  __m512i x1 = _mm512_srli_epi32(x0, 4);

  // LUT to convert mxfp4 values to bf16
  x0 = _mm512_permutexvar_epi16(x0, lut);
// ...

  return std::make_tuple(__m512bh(x0), __m512bh(x1));
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: quantize_row_int8 (lines 231-249)
```cpp
template <typename scalar_t>
inline void
quantize_row_int8(uint8_t* __restrict__ Aq, float& As, const scalar_t* __restrict__ A, int64_t K, float eps = 1e-7) {
  float amax = 0.f;  // absolute max
  for (int64_t k = 0; k < K; ++k) {
    const float val = static_cast<float>(A[k]);
    amax = std::max(amax, std::abs(val));
  }

  amax = std::max(amax, eps);
  const float scale = amax / 127;
  const float inv_scale = 127 / amax;

  for (int64_t k = 0; k < K; ++k) {
// ...
  }
  As = scale;
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: template <> inline void quantize_row_int8<at::BFloat16>( uin (lines 252-288)
```cpp
template <>
inline void quantize_row_int8<at::BFloat16>(
    uint8_t* __restrict__ Aq, float& As, const at::BFloat16* __restrict__ A, int64_t K, float eps) {
  const __m512 signBit = _mm512_set1_ps(-0.0f);
  const __m512i off = _mm512_set1_epi32(128);

  // K is 32x, no remainder
  float amax = 0.f;
  __m512 vamax0 = _mm512_set1_ps(0.f);
  __m512 vamax1 = _mm512_set1_ps(0.f);
  for (int64_t k = 0; k < K; k += 32) {
    __m512i va = _mm512_loadu_si512((void*)(A + k));
    __m512 va0 = CVT_BF16_TO_FP32(_mm512_extracti32x8_epi32(va, 0));
    __m512 va1 = CVT_BF16_TO_FP32(_mm512_extracti32x8_epi32(va, 1));
// ...
  }
  As = scale;
}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: transpose_16x16_32bit (lines 294-363)
```cpp
inline void transpose_16x16_32bit(__m512i* v) {
  __m512i v1[16];
  v1[0] = _mm512_unpacklo_epi32(v[0], v[1]);
  v1[1] = _mm512_unpackhi_epi32(v[0], v[1]);
  v1[2] = _mm512_unpacklo_epi32(v[2], v[3]);
  v1[3] = _mm512_unpackhi_epi32(v[2], v[3]);
  v1[4] = _mm512_unpacklo_epi32(v[4], v[5]);
  v1[5] = _mm512_unpackhi_epi32(v[4], v[5]);
  v1[6] = _mm512_unpacklo_epi32(v[6], v[7]);
  v1[7] = _mm512_unpackhi_epi32(v[6], v[7]);
  v1[8] = _mm512_unpacklo_epi32(v[8], v[9]);
  v1[9] = _mm512_unpackhi_epi32(v[8], v[9]);
  v1[10] = _mm512_unpacklo_epi32(v[10], v[11]);
  v1[11] = _mm512_unpackhi_epi32(v[10], v[11]);
// ...
  v[14] = _mm512_shuffle_i32x4(v1[6], v1[14], 0xdd);
  v[15] = _mm512_shuffle_i32x4(v1[7], v1[15], 0xdd);
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: _mm512_fexp_u20_ps (lines 387-436)
```cpp
inline __attribute__((always_inline)) __m512 _mm512_fexp_u20_ps(const __m512 values) {
  const __m512 vec_c0 = _mm512_set1_ps(0.00010703434948458272f);
  const __m512 vec_c1 = _mm512_set1_ps(0.30354260500649682f);
  const __m512 vec_c2 = _mm512_set1_ps(-0.22433836478672356);
  const __m512 vec_c3 = _mm512_set1_ps(-0.079204240219773236);

  const __m512 vec_exp_log2ef = _mm512_castsi512_ps(_mm512_set1_epi32(0x3fb8aa3b));  // log2(e)

  const __m512 vec_a = _mm512_set1_ps(std::pow(2, 23) / std::log2(2));
  const __m512 vec_b = _mm512_set1_ps(std::pow(2, 23) * 127.f);

  const __m512 vec_ln_flt_min = _mm512_castsi512_ps(_mm512_set1_epi32(0xc2aeac50));
  const __m512 vec_ln_flt_max = _mm512_castsi512_ps(_mm512_set1_epi32(0x42b17218));
  __m512i vec_infinity = _mm512_set1_epi32(0x7F800000);
// ...
  // final interpretation to float
  return _mm512_castsi512_ps(casted_integer);
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

## Key Concepts / 关键概念
- Token permutation / Token 置换
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Integer kernel specialization / 整数内核专用化
- Intel AMX tiling / Intel AMX 分块
- SIMD vectorization / SIMD 向量化
- CUDA programming model / CUDA 编程模型
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / ATen, x86 SIMD intrinsics
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
