# common.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/common.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Templates, aliases, and constants
```cpp
#pragma once

#include <ATen/ATen.h>
#include <ATen/Parallel.h>

#if defined(_OPENMP)
#include <omp.h>
#endif

namespace {

// dispatch bool
#define AT_DISPATCH_BOOL(BOOL_V, BOOL_NAME, ...) \
  [&] {                                          \
    if (BOOL_V) {                                \
      constexpr bool BOOL_NAME = true;           \
      return __VA_ARGS__();                      \
    } else {                                     \
      constexpr bool BOOL_NAME = false;          \
      return __VA_ARGS__();                      \
    }                                            \
  }()
```
**EN:** This section defines `__VA_ARGS__`, `BOOL_NAME`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`__VA_ARGS__`、`BOOL_NAME`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 24-46: Templates, aliases, and constants
```cpp
#define AT_DISPATCH_BOOL2(BOOL_V1, BOOL_NAME1, BOOL_V2, BOOL_NAME2, ...) \
  [&] {                                                                  \
    if (BOOL_V1) {                                                       \
      constexpr bool BOOL_NAME1 = true;                                  \
      if (BOOL_V2) {                                                     \
        constexpr bool BOOL_NAME2 = true;                                \
        return __VA_ARGS__();                                            \
      } else {                                                           \
        constexpr bool BOOL_NAME2 = false;                               \
        return __VA_ARGS__();                                            \
      }                                                                  \
    } else {                                                             \
      constexpr bool BOOL_NAME1 = false;                                 \
      if (BOOL_V2) {                                                     \
        constexpr bool BOOL_NAME2 = true;                                \
        return __VA_ARGS__();                                            \
      } else {                                                           \
        constexpr bool BOOL_NAME2 = false;                               \
        return __VA_ARGS__();                                            \
      }                                                                  \
    }                                                                    \
  }()
```
**EN:** This section defines `__VA_ARGS__`, `BOOL_NAME1`, `BOOL_NAME2`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`__VA_ARGS__`、`BOOL_NAME1`、`BOOL_NAME2`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 47-75: Runtime integration and dispatch
```cpp
// dispatch: bfloat16, float16, int8_t, fp8_e4m3, uint8_t(mxfp4/int4)
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
      }                                                          \
      case at::ScalarType::Float8_e4m3fn: {                      \
        using packed_t = at::Float8_e4m3fn;                      \
        return __VA_ARGS__();                                    \
      }                                                          \
      case at::ScalarType::Byte: {                               \
        using packed_t = uint8_t;                                \
        return __VA_ARGS__();                                    \
      }                                                          \
      default:                                                   \
        TORCH_CHECK(false, "Unsupported floating data type.\n"); \
    }                                                            \
  }()
```
**EN:** This section uses `__VA_ARGS__`, `TORCH_CHECK`, `packed_t` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`__VA_ARGS__`、`TORCH_CHECK`、`packed_t`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 76-99: Runtime integration and dispatch
```cpp
// Helper MICRO for CPU_DISPATCH_FLOATING_TYPES_EXT:
//   TYPE1: the primary dtype (input, output, weight);
//   TYPE2: defined as PARAM_T input
#define CPU_DISPATCH_TYPE1_WITH_PARAM(TYPE1, PARAM_T, ...)   \
  switch (TYPE1) {                                           \
    case at::ScalarType::BFloat16: {                         \
      using scalar_t = at::BFloat16;                         \
      using param_t = PARAM_T;                               \
      return __VA_ARGS__();                                  \
    }                                                        \
    case at::ScalarType::Half: {                             \
      using scalar_t = at::Half;                             \
      using param_t = PARAM_T;                               \
      return __VA_ARGS__();                                  \
    }                                                        \
    case at::ScalarType::Float: {                            \
      using scalar_t = float;                                \
      using param_t = PARAM_T;                               \
      return __VA_ARGS__();                                  \
    }                                                        \
    default:                                                 \
      TORCH_CHECK(false, "Unsupported floating data type."); \
  }
```
**EN:** This section uses `CPU_DISPATCH_TYPE1_WITH_PARAM`, `__VA_ARGS__`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CPU_DISPATCH_TYPE1_WITH_PARAM`、`__VA_ARGS__`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 100-118: Runtime integration and dispatch
```cpp
// Helper MICRO for CPU_DISPATCH_REDUCED_FLOATING_TYPES_EXT:
//   TYPE1: the primary dtype (input, output, weight);
//   TYPE2: defined as PARAM_T input
#define CPU_DISPATCH_TYPE1_WITH_PARAM_REDUCED(TYPE1, PARAM_T, ...) \
  switch (TYPE1) {                                                 \
    case at::ScalarType::BFloat16: {                               \
      using scalar_t = at::BFloat16;                               \
      using param_t = PARAM_T;                                     \
      return __VA_ARGS__();                                        \
    }                                                              \
    case at::ScalarType::Half: {                                   \
      using scalar_t = at::Half;                                   \
      using param_t = PARAM_T;                                     \
      return __VA_ARGS__();                                        \
    }                                                              \
    default:                                                       \
      TORCH_CHECK(false, "Unsupported floating data type.");       \
  }
```
**EN:** This section uses `CPU_DISPATCH_TYPE1_WITH_PARAM_REDUCED`, `__VA_ARGS__`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CPU_DISPATCH_TYPE1_WITH_PARAM_REDUCED`、`__VA_ARGS__`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 119-136: Runtime integration and dispatch
```cpp
// Helper MICRO for CPU_DISPATCH_REDUCED_FLOATING_TYPES_EXT:
//   TYPE1: the dtype both for scalar_t and param_t
#define CPU_DISPATCH_TYPE1_WITH_SAME_PARAM_REDUCED(TYPE1, ...)       \
  switch (TYPE1) {                                                   \
    case at::ScalarType::BFloat16: {                                 \
      using scalar_t = at::BFloat16;                                 \
      using param_t = at::BFloat16;                                  \
      return __VA_ARGS__();                                          \
    }                                                                \
    case at::ScalarType::Half: {                                     \
      using scalar_t = at::Half;                                     \
      using param_t = at::Half;                                      \
      return __VA_ARGS__();                                          \
    }                                                                \
    default:                                                         \
      TORCH_CHECK(false, "Unsupported reduced floating data type."); \
  }
```
**EN:** This section uses `CPU_DISPATCH_TYPE1_WITH_SAME_PARAM_REDUCED`, `__VA_ARGS__`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CPU_DISPATCH_TYPE1_WITH_SAME_PARAM_REDUCED`、`__VA_ARGS__`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 137-165: Runtime integration and dispatch
```cpp
// dispatch with mixed dtypes (TYPE1, TYPE2):
//   TYPE1: the primary dtype (input, output, weight);
//   TYPE2: the secondary dtype (bias, etc.).
#define CPU_DISPATCH_FLOATING_TYPES_EXT(TYPE1, TYPE2, ...)            \
  [&] {                                                               \
    if (TYPE2 == at::kFloat) {                                        \
      CPU_DISPATCH_TYPE1_WITH_PARAM(TYPE1, float, __VA_ARGS__)        \
    } else if (TYPE2 == at::ScalarType::BFloat16) {                   \
      CPU_DISPATCH_TYPE1_WITH_PARAM(TYPE1, at::BFloat16, __VA_ARGS__) \
    } else if (TYPE2 == at::ScalarType::Half) {                       \
      CPU_DISPATCH_TYPE1_WITH_PARAM(TYPE1, at::Half, __VA_ARGS__)     \
    } else {                                                          \
      TORCH_CHECK(false, "Unsupported floating data type.");          \
    }                                                                 \
  }()

// dispatch with mixed dtypes (reduced one, no float for TYPE1) (TYPE1, TYPE2):
//   TYPE1: the primary dtype (input, output, weight);
//   TYPE2: the secondary dtype (bias, etc.).
#define CPU_DISPATCH_REDUCED_FLOATING_TYPES_EXT(TYPE1, TYPE2, ...)     \
  [&] {                                                                \
    if (TYPE2 == at::kFloat) {                                         \
      CPU_DISPATCH_TYPE1_WITH_PARAM_REDUCED(TYPE1, float, __VA_ARGS__) \
    } else {                                                           \
      TORCH_CHECK(TYPE1 == TYPE2);                                     \
      CPU_DISPATCH_TYPE1_WITH_SAME_PARAM_REDUCED(TYPE1, __VA_ARGS__)   \
    }                                                                  \
  }()
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 166-186: Runtime integration and dispatch
```cpp
#define UNUSED(x) (void)(x)

#define CHECK_CPU(x) TORCH_CHECK(x.device().type() == at::kCPU, #x " must be a CPU tensor")

#define CHECK_CONTIGUOUS(x) TORCH_CHECK(x.is_contiguous(), #x " must be contiguous")
#define CHECK_LAST_DIM_CONTIGUOUS(x) \
  TORCH_CHECK(x.strides()[x.strides().size() - 1] == 1, #x "must be contiguous at last dimension")

#define CHECK_INPUT(x) \
  CHECK_CPU(x);        \
  CHECK_CONTIGUOUS(x)
#define CHECK_LAST_DIM_CONTIGUOUS_INPUT(x) \
  CHECK_CPU(x);                            \
  CHECK_LAST_DIM_CONTIGUOUS(x)

#define CHECK_DIM(d, x) TORCH_CHECK(x.dim() == d, #x " must be a " #d "D tensor")

#define CHECK_EQ(a, b) TORCH_CHECK((a) == (b), "CHECK_EQ(" #a ", " #b ") failed. ", a, " vs ", b)
#define CHECK_GT(a, b) TORCH_CHECK((a) > (b), "CHECK_GT(" #a ", " #b ") failed. ", a, " vs ", b)
#define CHECK_GE(a, b) TORCH_CHECK((a) >= (b), "CHECK_GE(" #a ", " #b ") failed. ", a, " vs ", b)
```
**EN:** This section uses `UNUSED`, `CHECK_CONTIGUOUS` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`UNUSED`、`CHECK_CONTIGUOUS`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 187-209: Runtime integration and dispatch
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

// [NB] Parallel Routines
//
//  * at::parallel_for - applies for most of generic use cases, this will be compiled
//                       against openmp in default torch release.
//
//  * parallel_for     - same function as above, can choose payload partition scheme in
//                       balance211.
//
//  * parallel_2d      - parallel for 2 dimensions, used in GEMM, etc.
//                       this one will do payload balance across 2 dimensions.
//
```
**EN:** This section uses `CHECK_INPUT_SHAPE_DTYPE`, `TORCH_CHECK`, `CHECK_LAST_DIM_CONTIGUOUS_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_INPUT_SHAPE_DTYPE`、`TORCH_CHECK`、`CHECK_LAST_DIM_CONTIGUOUS_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 210-227: Templates, aliases, and constants
```cpp
// grain size for each thread
constexpr int GRAIN_SIZE = 1024;

template <typename T, typename std::enable_if<std::is_integral<T>::value, int>::type = 0>
inline T div_up(T x, T y) {
  return (x + y - 1) / y;
}

// you can only use at::get_thread_num() with at::parallel_for()
// as it is lazy initialized, otherwise it will always return 0.
inline int get_thread_num() {
#if defined(_OPENMP)
  return omp_get_thread_num();
#else
  return 0;
#endif
}
```
**EN:** This section defines `div_up`, `get_thread_num`, `defined`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`div_up`、`get_thread_num`、`defined`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 228-251: Templates, aliases, and constants
```cpp
// balance payload across each thread
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
    }
    n_end += n_start;
#else
  // pytorch aten partition pattern
  T n_my = div_up(n, nth);
  n_start = ith * n_my;
  n_end = std::min(n_start + n_my, n);
#endif
}
```
**EN:** This section defines `balance211`, `div_up`, `min`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`balance211`、`div_up`、`min`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 252-275: Runtime integration and dispatch
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

// for 1d parallel, use `actual_nth`
// for 2d parallel, use even nths, e.g. 43->42
int inline adjust_num_threads(int m) {
  int actual_nth = at::get_num_threads();
  if (m == 1) {
    return actual_nth;
  }
```
**EN:** This section uses `parallel_for`, `adjust_num_threads`, `omp_get_num_threads` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`parallel_for`、`adjust_num_threads`、`omp_get_num_threads`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 276-303: Templates, aliases, and constants
```cpp
  return std::max(1, (actual_nth >> 1) * 2);
}

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
  //   then:
  //     TM = ((BM / BN) * T) ^ 0.5
  //
  float r = float(m) / n;
  int nth_m = std::ceil(std::sqrt(r * nth));
  int nth_n = 1;
  for (; nth_m > 0; --nth_m) {
    nth_n = nth / nth_m;
    if (nth_m * nth_n == nth) {
      break;
    }
```
**EN:** This section defines `parallel_2d`, `max`, `adjust_num_threads`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`parallel_2d`、`max`、`adjust_num_threads`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 304-327: Local implementation details
```cpp
  }

#if defined(_OPENMP)
#pragma omp parallel num_threads(nth)
  {
    int ith = omp_get_thread_num();
    int ith_m = ith / nth_n;
    int ith_n = ith % nth_n;

    int thread_block_m = div_up(m, nth_m);
    int thread_block_n = div_up(n, nth_n);

    int begin_m = ith_m * thread_block_m;
    int end_m = std::min(m, begin_m + thread_block_m);
    int begin_n = ith_n * thread_block_n;
    int end_n = std::min(n, begin_n + thread_block_n);

    f(begin_m, end_m, begin_n, end_n);
  }
#else
  f(0, m, 0, n);
#endif
}
```
**EN:** This section fills in the local implementation details around `defined`, `omp_get_thread_num`, `div_up`, completing the behavior required by the file.
**CN:** 本段补充了`defined`、`omp_get_thread_num`、`div_up`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 328-351: Runtime integration and dispatch
```cpp
// limit max cache blocks
// when we need to do pre-unpack for weights, e.g. fp8
#define MAX_CACHE_BLOCK_SIZE 4

template <typename T>
inline int get_cache_blocks(int chunk_size) {
  // L2 2MB and ratio of 50%
  const int L2_size = 2048 * 1024 >> 1;
  return std::max(1, int(L2_size / (chunk_size * sizeof(T))));
}

template <>
inline int get_cache_blocks<at::Float8_e4m3fn>(int chunk_size) {
  // fp8 uses bf16 as accumulate type
  int cache_block_size = get_cache_blocks<at::BFloat16>(chunk_size);
  return std::min(MAX_CACHE_BLOCK_SIZE, cache_block_size);
}

// 2d sequential loop in range : [mb0, mb1), [nb0, nb1)
template <typename T, typename func_t>
inline void loop_2d(int64_t mb0, int64_t mb1, int64_t nb0, int64_t nb1, int64_t chunk_size, const func_t& f) {
  // get number of blocks for L2 in most inner loop
  int64_t cache_blocks_nb = get_cache_blocks<T>(chunk_size);
```
**EN:** This section uses `get_cache_blocks`, `loop_2d`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`get_cache_blocks`、`loop_2d`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 352-375: Templates, aliases, and constants
```cpp
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

// data indexing for dimension collapse
template <typename T>
inline T data_index_init(T offset) {
  return offset;
}

template <typename T, typename... Args>
inline T data_index_init(T offset, T& x, const T& X, Args&&... args) {
  offset = data_index_init(offset, std::forward<Args>(args)...);
  x = offset % X;
  return offset / X;
}
```
**EN:** This section defines `data_index_init`, `min`, `f`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_index_init`、`min`、`f`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 376-396: Templates, aliases, and constants
```cpp
inline bool data_index_step() {
  return true;
}

template <typename T, typename... Args>
inline bool data_index_step(T& x, const T& X, Args&&... args) {
  if (data_index_step(std::forward<Args>(args)...)) {
    x = ((x + 1) == X) ? 0 : (x + 1);
    return x == 0;
  }
  return false;
}

// forced unroll for perf critical path

#if __has_attribute(always_inline)
#define ALWAYS_INLINE __attribute__((__always_inline__)) inline
#else
#define ALWAYS_INLINE inline
#endif
```
**EN:** This section defines `data_index_step`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_index_step`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 397-419: Types and data layout
```cpp
template <int n>
struct Unroll {
  template <typename Func, typename... Args>
  ALWAYS_INLINE void operator()(const Func& f, Args... args) const {
    Unroll<n - 1>{}(f, args...);
    f(std::integral_constant<int, n - 1>{}, args...);
  }
};

template <>
struct Unroll<1> {
  template <typename Func, typename... Args>
  ALWAYS_INLINE void operator()(const Func& f, Args... args) const {
    f(std::integral_constant<int, 0>{}, args...);
  }
};

// conditional data ptr for optional tensor
template <typename T>
inline T* conditional_data_ptr(const std::optional<at::Tensor>& opt) {
  return opt.has_value() ? opt.value().data_ptr<T>() : nullptr;
}
```
**EN:** This section defines `Unroll`, `conditional_data_ptr`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Unroll`、`conditional_data_ptr`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 420-420: Local implementation details
```cpp
}  // anonymous namespace
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/Parallel.h`, `omp.h`
- **Path context / 路径上下文**: cpu / common.h
