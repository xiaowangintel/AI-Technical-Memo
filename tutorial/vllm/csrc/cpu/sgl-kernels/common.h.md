# common.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/sgl-kernels/common.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements SGLang-style CPU kernels for GEMM, MoE, vector ops, and related utilities. / 实现 SGLang 风格的 CPU 内核，涵盖 GEMM、MoE、向量运算及相关工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-13)
```cpp
// Adapted from
// https://github.com/sgl-project/sglang/tree/main/sgl-kernel/csrc/cpu

// clang-format off

#pragma once

#include <ATen/ATen.h>
#include <ATen/Parallel.h>

#if defined(_OPENMP)
#include <omp.h>
#endif
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Compile-time setup: CPU_DISPATCH_PACKED_TYPES (lines 53-79)
```cpp
#define CPU_DISPATCH_PACKED_TYPES(TYPE, ...)                     \
  [&] {                                                          \
    switch (TYPE) {                                              \
      case at::ScalarType::BFloat16: {                           \
        using packed_t = at::BFloat16;                           \
        return __VA_ARGS__();                                    \
      }                                                          \
      case at::ScalarType::Half: {                               \
        using packed_t = at::Half;                               \
        return __VA_ARGS__();                                    \
      }                                                          \
      case at::ScalarType::Char: {                               \
        using packed_t = int8_t;                                 \
        return __VA_ARGS__();                                    \
// ...
        TORCH_CHECK(false, "Unsupported floating data type.\n"); \
    }                                                            \
  }()
```
**EN:** These macros encode compile-time dispatch rules, selecting the right specialization based on ISA, dtype, or backend capabilities.
**CN:** 这些宏封装了编译期分派规则，用于根据 ISA、数据类型或后端能力选择合适的专用实现。

### Function / Kernel: CHECK_INPUT_SHAPE_DTYPE (lines 192-201)
```cpp
template <bool is_only_lastdim_contiguous>
static inline void CHECK_INPUT_SHAPE_DTYPE(const at::Tensor& tensor, const at::IntArrayRef sizes, at::ScalarType st) {
  TORCH_CHECK(tensor.sizes() == sizes, "Input tensor shape mismatch: expected ", sizes, ", got ", tensor.sizes());
  TORCH_CHECK(tensor.scalar_type() == st, "Input tensor dtype mismatch");
  if constexpr (is_only_lastdim_contiguous) {
    CHECK_LAST_DIM_CONTIGUOUS_INPUT(tensor);
  } else {
    CHECK_INPUT(tensor);
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: balance211 (lines 234-256)
```cpp
template <typename T>
inline void balance211(T n, T nth, T ith, T& n_start, T& n_end) {
#if 0
    // onednn partition pattern
    T& n_my = n_end;
    if (nth <= 1 || n == 0) {
        n_start = 0;
        n_my = n;
    } else {
        T n1 = div_up(n, nth);
        T n2 = n1 - 1;
        T T1 = n - n2 * nth;
        n_my = ith < T1 ? n1 : n2;
        n_start = ith <= T1 ? ith*n1 : T1 * n1 + (ith - T1) * n2;
// ...
  n_end = std::min(n_start + n_my, n);
#endif
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: parallel_for (lines 258-272)
```cpp
template <typename func_t>
inline void parallel_for(int n, const func_t& f) {
#if defined(_OPENMP)
#pragma omp parallel
  {
    int nth = omp_get_num_threads();
    int ith = omp_get_thread_num();
    int tbegin, tend;
    balance211(n, nth, ith, tbegin, tend);
    f(tbegin, tend);
  }
#else
  f(0, n);
#endif
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: parallel_2d (lines 284-331)
```cpp
template <typename func_t>
inline void parallel_2d(int m, int n, const func_t& f) {
  // make sure we have even num_threads
  int nth = adjust_num_threads(m);

  // [NOTE] thread blocking:
  //
  //   1) prefer square block per thread
  //   2) use even number of CPU cores
  //   3) use all `num_threads` cores
  //
  //   we have:
  //     TM * TN = T
  //     BM / TM = BN / TN
// ...
  f(0, m, 0, n);
#endif
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: loop_2d (lines 359-373)
```cpp
template <typename T, typename func_t>
inline void loop_2d(int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1, int64_t chunk_size, const func_t& f) {
  // get number of blocks for L2 in most inner loop
  int64_t cache_blocks_nb = get_cache_blocks<T>(chunk_size);

  // loop order: [NB / cache_blocks_nb, MB, cache_blocks_nb]
  // TODO: implement reverse order of [MB / cache_blocks_mb, NB, cache_blocks_mb]
  for (int64_t nbb = nb0; nbb < nb1; nbb += cache_blocks_nb) {
    for (int64_t mb = mb0; mb < mb1; ++mb) {
      for (int64_t nb = nbb; nb < std::min(nbb + cache_blocks_nb, nb1); ++nb) {
        f(mb, nb, nb - nbb);
      }
    }
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

## Key Concepts / 关键概念
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / ATen, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
