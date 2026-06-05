# vec_pack.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/vec_pack.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Namespace and shared declarations
```cpp
// To use the transpose functions
#include <ATen/native/cpu/utils.h>

#include "vec.h"

namespace {

using namespace at::vec;

template <typename index_t>
inline index_t get_index(index_t* ind, int i) {
  return (ind == nullptr) ? (index_t)i : ind[i];
}

#if defined(CPU_CAPABILITY_AVX512)
// key: from [N, 32] to [32/2, N, 2]
template <typename scalar_t, typename index_t>
inline void pack_vnni_Nx32(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int ld_src,
    int ld_dst) {
  __m512i vinputs[16];
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 27-45: Control flow and branching
```cpp
  int n = 0;
  for (; n < N; ++n) {
    index_t index = get_index(ind, n);
    vinputs[n] = _mm512_loadu_si512(src + index * ld_src);
  }
  // padding with zero to avoid uninitialized vectors
  for (; n < 16; ++n) {
    vinputs[n] = _mm512_set1_epi32(0);
  }

  // pack key
  transpose_16x16_32bit(vinputs);

  const __mmask16 vmask = (1 << N) - 1;
  for (int k = 0; k < 16; ++k) {
    _mm512_mask_storeu_epi32(dst + k * ld_dst * 2, vmask, vinputs[k]);
  }
}
```
**EN:** This section drives `get_index`, `_mm512_loadu_si512`, `_mm512_set1_epi32` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`get_index`、`_mm512_loadu_si512`、`_mm512_set1_epi32`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 46-68: Templates, aliases, and constants
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

  int n = 0;
  for (; n < N; ++n) {
    index_t index = get_index(ind, n);
    vinputs[n] = _mm512_maskz_loadu_epi32(vmask, src + index * ld_src);
  }
  // padding with zero to avoid uninitialized vectors
  for (; n < 16; ++n) {
    vinputs[n] = _mm512_set1_epi32(0);
  }
```
**EN:** This section defines `pack_vnni_N_remainder`, `get_index`, `_mm512_maskz_loadu_epi32`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni_N_remainder`、`get_index`、`_mm512_maskz_loadu_epi32`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 69-89: Templates, aliases, and constants
```cpp

  // pack key
  transpose_16x16_32bit(vinputs);

  const __mmask16 vmask2 = (1 << N) - 1;
  for (int k = 0; k < K2; ++k) {
    _mm512_mask_storeu_epi32(dst + k * ld_dst * 2, vmask2, vinputs[k]);
  }
}

// value: from [K, 32] to [K/2, 32, 2]
template <typename scalar_t, typename index_t>
inline void pack_vnni_Kx32(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int K,
    int ld_src,
    int ld_dst) {
  __m512i vinputs[2];
```
**EN:** This section defines `pack_vnni_Kx32`, `transpose_16x16_32bit`, `_mm512_mask_storeu_epi32`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni_Kx32`、`transpose_16x16_32bit`、`_mm512_mask_storeu_epi32`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 90-106: Control flow and branching
```cpp
  int k = 0;
  for (; k < K; ++k) {
    index_t index = get_index(ind, k);
    vinputs[k] = _mm512_loadu_si512(src + index * ld_src);
  }
  // padding with zero to avoid uninitialized vectors
  for (; k < 2; ++k) {
    vinputs[k] = _mm512_set1_epi32(0);
  }

  // pack value
  __m512i d0, d1;
  std::tie(d0, d1) = transpose_2x32_16bit(vinputs[0], vinputs[1]);
  _mm512_storeu_si512(dst + 0 * ld_dst * 2, d0);
  _mm512_storeu_si512(dst + 0 * ld_dst * 2 + 32, d1);
}
```
**EN:** This section drives `get_index`, `_mm512_loadu_si512`, `_mm512_set1_epi32` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`get_index`、`_mm512_loadu_si512`、`_mm512_set1_epi32`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 107-128: Templates, aliases, and constants
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
  for (; k < K; ++k) {
    index_t index = get_index(ind, k);
    vinputs[k] = _mm512_maskz_loadu_epi16(vmask, src + index * ld_src);
  }
  // padding with zero to avoid uninitialized vectors
  for (; k < 2; ++k) {
    vinputs[k] = _mm512_set1_epi32(0);
  }
```
**EN:** This section defines `pack_vnni_K_remainder`, `get_index`, `_mm512_maskz_loadu_epi16`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni_K_remainder`、`get_index`、`_mm512_maskz_loadu_epi16`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 129-146: Control flow and branching
```cpp

  // pack value
  __m512i d0, d1;
  std::tie(d0, d1) = transpose_2x32_16bit(vinputs[0], vinputs[1]);

  if (N <= 16) {
    // 2N * 16bits: N * 32bits
    const __mmask16 vmask2 = (1 << N) - 1;
    _mm512_mask_storeu_epi32(dst + 0 * ld_dst * 2, vmask2, d0);
  } else {
    // 2(N-16) * 16bits: (N-16) * 32bits
    const __mmask16 vmask2 = (1 << (N - 16)) - 1;
    _mm512_storeu_epi32(dst + 0 * ld_dst * 2, d0);
    _mm512_mask_storeu_epi32(dst + 0 * ld_dst * 2 + 32, vmask2, d1);
  }
}
#endif
```
**EN:** This section drives `tie`, `_mm512_mask_storeu_epi32`, `_mm512_storeu_epi32` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`tie`、`_mm512_mask_storeu_epi32`、`_mm512_storeu_epi32`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 147-174: Templates, aliases, and constants
```cpp
// convert to vnni format
// from [N, K/2, 2] to [K/2, N, 2] for bfloat16 and float16
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

  for (int nb = 0; nb < NB; ++nb) {
    int nb_size = std::min(N - nb * 16, 16);
    for (int kb = 0; kb < KB; ++kb) {
      // handle 16x512bits each block
      pack_vnni_Nx32<scalar_t, index_t>(
          /*    dst */ dst + ((kb * 32) >> 1) * ld_dst * 2 + nb * 16 * 2,
          /*    src */ src + kb * 32 + (is_indexed ? 0 : nb * 16 * ld_src),
          /*    ind */ is_indexed ? ind + nb * 16 : nullptr,
          /*      N */ nb_size,
          /* ld_src */ ld_src,
          /* ld_dst */ ld_dst);
    }
```
**EN:** This section defines `pack_vnni`, `defined`, `min`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni`、`defined`、`min`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 175-196: Control flow and branching
```cpp
    if (K_remainder > 0) {
      pack_vnni_N_remainder<scalar_t, index_t>(
          /*    dst */ dst + ((KB * 32) >> 1) * ld_dst * 2 + nb * 16 * 2,
          /*    src */ src + KB * 32 + (is_indexed ? 0 : nb * 16 * ld_src),
          /*    ind */ is_indexed ? ind + nb * 16 : nullptr,
          /*      N */ nb_size,
          /*      K */ K_remainder,
          /* ld_src */ ld_src,
          /* ld_dst */ ld_dst);
    }
  }
#else
  for (int n = 0; n < N; ++n) {
    index_t index = get_index(ind, n);
    for (int k = 0; k < K / 2; ++k) {
      for (int d = 0; d < 2; ++d) {
        dst[k * ld_dst * 2 + n * 2 + d] = src[index * ld_src + k * 2 + d];
      }
    }
  }
#endif
}
```
**EN:** This section drives `get_index` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`get_index`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 197-215: Templates, aliases, and constants
```cpp

template <typename scalar_t>
void pack_vnni(scalar_t* __restrict__ dst, const scalar_t* __restrict__ src, int N, int K, int ld_src, int ld_dst) {
  pack_vnni<scalar_t, int32_t, false>(dst, src, nullptr, N, K, ld_src, ld_dst);
}

template <typename scalar_t, typename index_t>
void pack_vnni(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int N,
    int K,
    int ld_src,
    int ld_dst) {
  assert(ind != nullptr);
  pack_vnni<scalar_t, index_t, true>(dst, src, ind, N, K, ld_src, ld_dst);
}
```
**EN:** This section defines `pack_vnni`, `assert`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni`、`assert`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 216-243: Templates, aliases, and constants
```cpp
// convert to vnni format
// from [K/2, 2, N] to [K/2, N, 2] for bfloat16 and float16
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

  for (int kb = 0; kb < KB; ++kb) {
    int kb_size = std::min(K - kb * 2, 2);
    for (int nb = 0; nb < NB; ++nb) {
      // handle 2x512bits each block
      pack_vnni_Kx32<scalar_t, index_t>(
          /*    dst */ dst + ((kb * 2) >> 1) * ld_dst * 2 + nb * 32 * 2,
          /*    src */ src + (is_indexed ? 0 : kb * 2 * ld_src) + nb * 32,
          /*    ind */ is_indexed ? ind + kb * 2 : nullptr,
          /*      K */ kb_size,
          /* ld_src */ ld_src,
          /* ld_dst */ ld_dst);
    }
```
**EN:** This section defines `pack_vnni2`, `defined`, `min`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni2`、`defined`、`min`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 244-264: Control flow and branching
```cpp
    if (N_remainder > 0) {
      pack_vnni_K_remainder(
          /*    dst */ dst + ((kb * 2) >> 1) * ld_dst * 2 + NB * 32 * 2,
          /*    src */ src + (is_indexed ? 0 : kb * 2 * ld_src) + NB * 32,
          /*    ind */ is_indexed ? ind + kb * 2 : nullptr,
          /*      K */ kb_size,
          /*      N */ N_remainder,
          /* ld_src */ ld_src,
          /* ld_dst */ ld_dst);
    }
  }
#else
  int k = 0;
  for (; k < (K >> 1) * 2; k += 2) {
    index_t index0 = get_index(ind, k + 0);
    index_t index1 = get_index(ind, k + 1);
    for (int n = 0; n < N; ++n) {
      dst[(k >> 1) * ld_dst * 2 + n * 2 + 0] = src[index0 * ld_src + n];
      dst[(k >> 1) * ld_dst * 2 + n * 2 + 1] = src[index1 * ld_src + n];
    }
  }
```
**EN:** This section drives `pack_vnni_K_remainder`, `get_index` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`pack_vnni_K_remainder`、`get_index`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 265-292: Templates, aliases, and constants
```cpp
  if (K % 2 != 0) {
    index_t index = get_index(ind, K - 1);
    for (int n = 0; n < N; ++n) {
      dst[(K >> 1) * ld_dst * 2 + n * 2 + 0] = src[index * ld_src + n];
      dst[(K >> 1) * ld_dst * 2 + n * 2 + 1] = 0;
    }
    k += 2;
  }
#endif
}

template <typename scalar_t>
void pack_vnni2(scalar_t* __restrict__ dst, const scalar_t* __restrict__ src, int K, int N, int ld_src, int ld_dst) {
  pack_vnni2<scalar_t, int32_t, false>(dst, src, nullptr, K, N, ld_src, ld_dst);
}

template <typename scalar_t, typename index_t>
void pack_vnni2(
    scalar_t* __restrict__ dst,
    const scalar_t* __restrict__ src,
    const index_t* __restrict__ ind,
    int K,
    int N,
    int ld_src,
    int ld_dst) {
  assert(ind != nullptr);
  pack_vnni2<scalar_t, index_t, true>(dst, src, ind, K, N, ld_src, ld_dst);
}
```
**EN:** This section defines `pack_vnni2`, `get_index`, `assert`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`pack_vnni2`、`get_index`、`assert`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 293-293: Local implementation details
```cpp

```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 294-294: Local implementation details
```cpp
}  // anonymous namespace
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `vec.h`
- **External headers / 外部头文件**: `ATen/native/cpu/utils.h`
- **Path context / 路径上下文**: cpu / vec_pack.h
