# vec_pack.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/vec_pack.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-9)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

// To use the transpose functions
#include <ATen/native/cpu/utils.h>

#include "vec.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: pack_vnni_Nx32 (lines 22-49)
```cpp
template <typename scalar_t, typename index_t>
inline void pack_vnni_Nx32(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int ld_src,
    int ld_dst) {
  __m512i vinputs[16];

  int n = 0;
  for (; n < N; ++n) {
    index_t index = get_index(ind, n);
    vinputs[n] = _mm512_loadu_si512(src + index * ld_src);
// ...
    _mm512_mask_storeu_epi32(dst + k * ld_dst * 2, vmask, vinputs[k]);
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: pack_vnni_N_remainder (lines 51-82)
```cpp
template <typename scalar_t, typename index_t>
inline void pack_vnni_N_remainder(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int K,
    int ld_src,
    int ld_dst) {
  __m512i vinputs[16];

  int K2 = K >> 1;
  const __mmask16 vmask = (1 << K2) - 1;

// ...
    _mm512_mask_storeu_epi32(dst + k * ld_dst * 2, vmask2, vinputs[k]);
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: pack_vnni_Kx32 (lines 85-110)
```cpp
template <typename scalar_t, typename index_t>
inline void pack_vnni_Kx32(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int K,
    int ld_src,
    int ld_dst) {
  __m512i vinputs[2];

  int k = 0;
  for (; k < K; ++k) {
    index_t index = get_index(ind, k);
    vinputs[k] = _mm512_loadu_si512(src + index * ld_src);
// ...
  _mm512_storeu_si512(dst + 0 * ld_dst * 2, d0);
  _mm512_storeu_si512(dst + 0 * ld_dst * 2 + 32, d1);
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: pack_vnni_K_remainder (lines 112-149)
```cpp
template <typename scalar_t, typename index_t>
inline void pack_vnni_K_remainder(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int K,
    int N,
    int ld_src,
    int ld_dst) {
  __m512i vinputs[2];

  const __mmask32 vmask = (1 << N) - 1;

  int k = 0;
// ...
    _mm512_mask_storeu_epi32(dst + 0 * ld_dst * 2 + 32, vmask2, d1);
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: pack_vnni (lines 154-201)
```cpp
template <typename scalar_t, typename index_t, bool is_indexed>
void pack_vnni(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int K,
    int ld_src,
    int ld_dst) {
#if defined(CPU_CAPABILITY_AVX512)
  const int NB = div_up(N, 16);
  const int KB = K / 32;
  const int K_remainder = K - KB * 32;

// ...
  }
#endif
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: pack_vnni2 (lines 223-279)
```cpp
template <typename scalar_t, typename index_t, bool is_indexed>
void pack_vnni2(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int K,
    int N,
    int ld_src,
    int ld_dst) {
#if defined(CPU_CAPABILITY_AVX512)
  const int KB = div_up(K, 2);
  const int NB = N / 32;
  const int N_remainder = N - NB * 32;

// ...
  }
#endif
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

## Key Concepts / 关键概念
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / ATen, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
