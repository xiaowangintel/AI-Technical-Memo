# SharedReduceOps.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/SharedReduceOps.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on shared reduce ops; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 shared reduce ops；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#pragma once
// Please note that this file is
// used across both CPU and GPU.

#include <type_traits>
#include <complex>
#include <c10/macros/Macros.h>
#include <ATen/detail/FunctionTraits.h>
#include <ATen/NumericUtils.h>
#include <ATen/OpMathType.h>
#if defined(__CUDACC__)
#include <ATen/cuda/DeviceUtils.cuh>
#include <ATen/native/cuda/DeviceSqrt.cuh>
#elif defined(__HIPCC__)
#include <ATen/hip/DeviceUtils.cuh>
#include <ATen/native/hip/DeviceSqrt.cuh>
#endif
#if defined(__CUDACC__) || defined(__HIPCC__)
#include <thrust/pair.h>
#else
#include <cmath>
#define device_sqrt std::sqrt
#endif
#if defined(__CUDACC__) || defined(__HIPCC__)
template <typename scalar_t>
inline C10_DEVICE scalar_t max_propagate_nan(scalar_t a, scalar_t b) {
  scalar_t max = at::_isnan(b) ? b : std::max(a, b);
  return max;
}
template <typename scalar_t>
inline C10_DEVICE scalar_t min_propagate_nan(scalar_t a, scalar_t b) {
  scalar_t min = at::_isnan(b) ? b : std::min(a, b);
  return min;
}
#define MAX(X, Y) max_propagate_nan(X,Y)
#define MIN(X, Y) min_propagate_nan(X,Y)
#else
#include <ATen/native/cpu/zmath.h>
#define MAX(X, Y) max_impl(X,Y)
#define MIN(X, Y) min_impl(X,Y)
#endif

// ROCm hip compiler doesn't work well with using std:: in kernel functions
#if defined(__CUDA_ARCH__) || defined(__HIPCC__)
#if defined(__CUDA_ARCH__)
#include <c10/cuda/CUDAMathCompat.h>
#elif defined(__HIPCC__)
#include <c10/hip/HIPMathCompat.h>
#endif
#define compat_pow c10::cuda::compat::pow
#else
#define compat_pow std::pow
#endif

namespace at::native {

namespace detail {

#if defined(__CUDACC__) || defined(__HIPCC__)
template <typename T1, typename T2> using pair = thrust::pair<T1, T2>;
```
- EN: Lines 1-60 pull in 15 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are defined, max_propagate_nan, min_propagate_nan, MAX, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 15 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 defined, max_propagate_nan, min_propagate_nan, MAX，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
#else
template <typename T1, typename T2> using pair = std::pair<T1, T2>;
#endif

} // namespace detail

template <typename scalar_t, typename index_t>
struct WelfordData {
  scalar_t mean;
  scalar_t m2;
  index_t n;
  scalar_t nf;

  C10_HOST_DEVICE WelfordData() : mean(0), m2(0), n(0), nf(0) {}

  C10_HOST_DEVICE WelfordData(
      scalar_t mean,
      scalar_t m2,
      index_t n,
      scalar_t nf)
      : mean(mean), m2(m2), n(n), nf(nf) {}
};


template <typename scalar_t, typename acc_scalar_t, typename index_t, typename res_t>
struct WelfordOps {
  acc_scalar_t correction;
  bool take_sqrt;
 public:
  using acc_t = WelfordData<acc_scalar_t, index_t>;
  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, index_t /*idx*/) const {
    // We accumulate n in index_t to avoid cumulative rounding error, but still
    // need nf for use in combine where int32 may overflow.
    index_t new_n = acc.n + 1;
    acc_scalar_t new_nf = static_cast<acc_scalar_t>(new_n);
    acc_scalar_t delta = data - acc.mean;
    acc_scalar_t new_mean = acc.mean + delta / new_nf;
    acc_scalar_t new_delta = data - new_mean;
    return {
      new_mean,
      acc.m2 + delta * new_delta,
      new_n,
      new_nf,
    };
  }
  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    if (a.nf == 0) {
      return b;
    }
    if (b.nf == 0) {
      return a;
    }
    acc_scalar_t delta = b.mean - a.mean;
    acc_scalar_t new_count = a.nf + b.nf;
    acc_scalar_t nb_over_n = b.nf / new_count;
    return {
      a.mean + delta * nb_over_n,
      a.m2 + b.m2 + delta * delta * a.nf * nb_over_n,
      // setting acc.n as -1 since acc.n might not be able to represent the count
      // correctly within its range, setting it to -1 to avoid confusion
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are reduce, combine, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 reduce, combine，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-180
```cpp
      -1,
      new_count
    };
  }
  inline C10_DEVICE res_t project(acc_t acc) const __ubsan_ignore_float_divide_by_zero__ {
    const auto mean = static_cast<scalar_t>(acc.mean);
    const auto divisor = acc.nf > correction ? acc.nf - correction : 0;
    const auto var = acc.m2 / divisor;
    res_t results(take_sqrt ? device_sqrt(var) : var, mean);
    return results;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline __device__ acc_t warp_shfl_down(acc_t acc, int offset) const {
    return {
      WARP_SHFL_DOWN(acc.mean, offset)
      , WARP_SHFL_DOWN(acc.m2, offset)
      , WARP_SHFL_DOWN(acc.n, offset)
      , WARP_SHFL_DOWN(acc.nf, offset)
    };
  }
#endif
  C10_HOST_DEVICE WelfordOps(acc_scalar_t correction, bool take_sqrt)
      : correction(correction), take_sqrt(take_sqrt) {}
};

template <typename scalar_t, typename acc_t=scalar_t, typename factor_t=acc_t, typename out_t = acc_t>
struct MeanOps {
  factor_t factor;

  inline C10_DEVICE acc_t reduce(acc_t a, scalar_t b, int64_t /*idx*/) const {
    return combine(a, static_cast<acc_t>(b));
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return a + b;
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    return a * factor;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t data, int offset) const {
    return WARP_SHFL_DOWN(data, offset);
  }
#endif

  MeanOps(factor_t factor): factor(factor) {
  }
};
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are translate_idx, defined, warp_shfl_down, WARP_SHFL_DOWN, concentrating a specific part of the operator behavior.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 translate_idx, defined, warp_shfl_down, WARP_SHFL_DOWN，它们承载了某一部分算子行为的核心逻辑。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

### Lines 181-240
```cpp
// This accumulator template is used to calculate the minimum absolute value of
// a set of numbers.
// `scalar_t` is the type of the input and `acc_t` is the type of the accumulated
// value. These types differ for complex number input support.
template <typename scalar_t, typename acc_t = scalar_t, typename out_t = acc_t>
struct AbsMinOps {

  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, int64_t /*idx*/) const {
    return MIN(acc, static_cast<acc_t>(std::abs(at::opmath_type<scalar_t>(data))));
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return MIN(a, b);
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    return a;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return WARP_SHFL_DOWN(acc, offset);
  }
#endif
};

// This accumulator template is used to calculate the maximum absolute value of
// a set of numbers.
// `scalar_t` is the type of the input and `acc_t` is the type of the accumulated
// value. These types differ for complex number input support.
template <typename scalar_t, typename acc_t = scalar_t, typename out_t = acc_t>
struct AbsMaxOps {
  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, int64_t /*idx*/) const {
    return MAX(acc, static_cast<acc_t>(std::abs(at::opmath_type<scalar_t>(data))));
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return MAX(a, b);
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    return a;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return WARP_SHFL_DOWN(acc, offset);
  }
#endif
};

// This accumulator template is used to calculate the norm of the absolute value
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are reduce, combine, project, translate_idx, concentrating a specific part of the operator behavior.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 reduce, combine, project, translate_idx，它们承载了某一部分算子行为的核心逻辑。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 241-300
```cpp
// of a set of numbers.
// `scalar_t` is the type of the input and `acc_t` is the type of the accumulated
// value. These types differ for complex number input support.
// `apply_root` controls whether to apply the final root: if true, returns
// (sum(|x|^p))^(1/p); if false, returns sum(|x|^p) (used by linalg._powsum).
template <typename scalar_t, typename acc_t = scalar_t, typename out_t = acc_t, bool apply_root = true>
struct NormOps {
  acc_t norm_;

  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, int64_t /*idx*/) const {
    return acc + compat_pow(static_cast<acc_t>(std::abs(at::opmath_type<scalar_t>(data))), norm_);
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return a + b;
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    if constexpr (apply_root) {
      return compat_pow(a, static_cast<acc_t>(1.0) / norm_);
    } else {
      return a;
    }
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return WARP_SHFL_DOWN(acc, offset);
  }
#endif

  NormOps(acc_t norm_): norm_(norm_) {
  }
};

// This accumulator template is used to calculate the order zero norm of the
// absolute value of a set of numbers.
// `scalar_t` is the type of the input and `acc_t` is the type of the accumulated
// value. These types differ for complex number input support.
template <typename scalar_t, typename acc_t = scalar_t, typename out_t = acc_t>
struct NormZeroOps {
  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, int64_t /*idx*/) const {
    return acc + (data == static_cast<scalar_t>(0) ? static_cast<acc_t>(0) : static_cast<acc_t>(1));
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return a + b;
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    return a;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are reduce, combine, project, translate_idx, concentrating a specific part of the operator behavior.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 reduce, combine, project, translate_idx，它们承载了某一部分算子行为的核心逻辑。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 301-360
```cpp


#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return WARP_SHFL_DOWN(acc, offset);
  }
#endif
};

// This accumulator template is used to calculate the order one norm of the
// absolute value of a set of numbers.
// `scalar_t` is the type of the input and `acc_t` is the type of the accumulated
// value. These types differ for complex number input support.
template <typename scalar_t, typename acc_t = scalar_t, typename out_t = acc_t>
struct NormOneOps {
  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, int64_t /*idx*/) const {
    return acc + static_cast<acc_t>(std::abs(at::opmath_type<scalar_t>(data)));
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return a + b;
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    return a;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return WARP_SHFL_DOWN(acc, offset);
  }
#endif
};


template<typename acc_t>
struct AbsSwitch {};

template<typename scalar_t, typename acc_t>
inline C10_DEVICE acc_t abs_if_complex(scalar_t data, AbsSwitch<acc_t> /*unused*/) {
  return static_cast<acc_t>(data);
}

template<typename scalar_t, typename acc_t>
inline C10_DEVICE acc_t abs_if_complex(std::complex<scalar_t> data, AbsSwitch<acc_t> /*unused*/) {
  return static_cast<acc_t>(std::abs(data));
}

template<typename scalar_t, typename acc_t>
inline C10_DEVICE acc_t abs_if_complex(c10::complex<scalar_t> data, AbsSwitch<acc_t> /*unused*/) {
  return static_cast<acc_t>(std::abs(at::opmath_type<c10::complex<scalar_t>>(data)));
}

// This accumulator template is used to calculate the order two norm of the
// absolute value of a set of numbers.
// `scalar_t` is the type of the input and `acc_t` is the type of the accumulated
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are defined, warp_shfl_down, reduce, combine, concentrating a specific part of the operator behavior.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 defined, warp_shfl_down, reduce, combine，它们承载了某一部分算子行为的核心逻辑。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 361-420
```cpp
// value. These types differ for complex number input support.
// `apply_root` controls whether to apply the final sqrt: if true, returns
// sqrt(sum(|x|^2)); if false, returns sum(|x|^2) (used by linalg._powsum).
template <typename scalar_t, typename acc_t = scalar_t, typename out_t = acc_t, bool apply_root = true>
struct NormTwoOps {
  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, int64_t /*idx*/) const {
    acc_t data_ = abs_if_complex(data, AbsSwitch<acc_t>());
    return acc + data_ * data_;
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return a + b;
  }

  inline C10_DEVICE out_t project(acc_t a) const {
    if constexpr (apply_root) {
      return device_sqrt(a);
    } else {
      return a;
    }
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return WARP_SHFL_DOWN(acc, offset);
  }
#endif
};

template <typename acc_t, typename data_t>
struct NanSumOps {
  inline C10_DEVICE acc_t reduce(acc_t a, data_t b, int64_t /*idx*/) const {
    return a + (at::_isnan(b) ? acc_t{0.} : acc_t{b});
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    return  a + b;
  }

  inline C10_DEVICE data_t project(acc_t a) const {
    return data_t{a};
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t data, int offset) const {
    return WARP_SHFL_DOWN(data, offset);
  }
#endif
};

namespace detail {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are reduce, combine, project, translate_idx, concentrating a specific part of the operator behavior.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 reduce, combine, project, translate_idx，它们承载了某一部分算子行为的核心逻辑。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 421-480
```cpp
template <typename scalar_t>
struct LessOrNan {
  C10_DEVICE bool operator () (scalar_t a, scalar_t b, int64_t idx_a, int64_t idx_b) const {
    // If (a == b), then choose the one with lower idx, else min(a, b)
    if (at::_isnan(a)) {
      if (at::_isnan(b)) {
        return idx_a < idx_b;
      }
      return true;
    }
    return (a == b) ? idx_a < idx_b : (a < b);
  }
};

template <typename scalar_t>
struct GreaterOrNan {
  C10_DEVICE bool operator () (scalar_t a, scalar_t b, int64_t idx_a, int64_t idx_b) const {
    // If (a == b), then choose the one with lower idx, else max(a, b)
    if (at::_isnan(a)) {
      if (at::_isnan(b)) {
        return idx_a < idx_b;
      }
      return true;
    }
    return (a == b) ? idx_a < idx_b : (a > b);
  }
};

template <typename comp_t>
struct MinMaxReductionOps {
  using scalar_t = typename binary_function_traits<comp_t>::arg1_t;
  using index_t = int64_t;
  using arg_t = detail::pair<scalar_t, index_t>;

  static C10_DEVICE arg_t project(arg_t arg) {
    return arg;
  }

  static C10_DEVICE arg_t reduce(arg_t arg, scalar_t val, int64_t idx) {
    return comp_t{}(arg.first, val, arg.second, idx) ? arg : arg_t(val, idx);
  }

  static C10_DEVICE arg_t combine(arg_t a, arg_t b) {
    return comp_t{}(a.first, b.first, a.second, b.second) ? a : b;
  }

  static C10_DEVICE arg_t translate_idx(arg_t a, int64_t base_idx) {
    return {a.first, a.second + base_idx};
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  static C10_DEVICE arg_t warp_shfl_down(arg_t arg, int offset) {
    return arg_t(WARP_SHFL_DOWN(arg.first, offset),
                 WARP_SHFL_DOWN(arg.second, offset));
  }
#endif
};

template <typename comp_t>
struct ArgReductionOps : public MinMaxReductionOps<comp_t> {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are operator, If, project, reduce, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 operator, If, project, reduce，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 481-540
```cpp
  using typename MinMaxReductionOps<comp_t>::scalar_t;
  using typename MinMaxReductionOps<comp_t>::index_t;
  using typename MinMaxReductionOps<comp_t>::arg_t;

  static C10_DEVICE index_t project(arg_t arg) {
    return arg.second;
  }
};

} // namespace detail

template <typename scalar_t>
struct ArgMaxOps :
  public detail::ArgReductionOps<detail::GreaterOrNan<scalar_t>> {
};

template <typename scalar_t>
struct ArgMinOps :
  public detail::ArgReductionOps<detail::LessOrNan<scalar_t>> {
};

template <typename scalar_t>
struct MinOps :
  public detail::MinMaxReductionOps<detail::LessOrNan<scalar_t>> {
};

template <typename scalar_t>
struct MaxOps :
  public detail::MinMaxReductionOps<detail::GreaterOrNan<scalar_t>> {
};

template <typename scalar_t, typename acc_scalar_t, typename index_t>
struct MinMaxOps {
  using acc_t = detail::pair<acc_scalar_t, acc_scalar_t>;
  inline C10_DEVICE acc_t reduce(acc_t acc, scalar_t data, index_t /*idx*/) const {
    return combine(acc, {data, data});
  }

  inline C10_DEVICE acc_t combine(acc_t a, acc_t b) const {
    auto min_val = (at::_isnan(a.first) || a.first < b.first) ? a.first : b.first;
    auto max_val = (at::_isnan(a.second) || a.second > b.second) ? a.second : b.second;

    return {min_val, max_val};
  }

  inline C10_DEVICE acc_t project(acc_t acc) const {
    return acc;
  }

  static C10_DEVICE acc_t translate_idx(acc_t acc, int64_t /*base_idx*/) {
    return acc;
  }

#if defined(__CUDACC__) || defined(__HIPCC__)
  inline C10_DEVICE acc_t warp_shfl_down(acc_t acc, int offset) const {
    return {
      WARP_SHFL_DOWN(acc.first, offset), WARP_SHFL_DOWN(acc.second, offset)
    };
  }
#endif
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are project, reduce, combine, translate_idx, concentrating a specific part of the operator behavior.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 project, reduce, combine, translate_idx，它们承载了某一部分算子行为的核心逻辑。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

### Lines 541-546
```cpp
};

} // namespace at::native

#undef MAX
#undef MIN
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。

## Key Concepts / 关键概念

- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: max_propagate_nan, min_propagate_nan, WelfordData, reduce, combine, translate_idx, warp_shfl_down, WARP_SHFL_DOWN.
- CN: 重要符号：max_propagate_nan, min_propagate_nan, WelfordData, reduce, combine, translate_idx, warp_shfl_down, WARP_SHFL_DOWN。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/macros/Macros.h, ATen/detail/FunctionTraits.h, ATen/NumericUtils.h, ATen/OpMathType.h, ATen/cuda/DeviceUtils.cuh, ATen/native/cuda/DeviceSqrt.cuh, ATen/hip/DeviceUtils.cuh, ATen/native/hip/DeviceSqrt.cuh, ATen/native/cpu/zmath.h, c10/cuda/CUDAMathCompat.h, c10/hip/HIPMathCompat.h`.
- CN: 主要内部头文件：`c10/macros/Macros.h, ATen/detail/FunctionTraits.h, ATen/NumericUtils.h, ATen/OpMathType.h, ATen/cuda/DeviceUtils.cuh, ATen/native/cuda/DeviceSqrt.cuh, ATen/hip/DeviceUtils.cuh, ATen/native/hip/DeviceSqrt.cuh, ATen/native/cpu/zmath.h, c10/cuda/CUDAMathCompat.h, c10/hip/HIPMathCompat.h`。
- EN: External/system headers: `type_traits, complex, thrust/pair.h, cmath`.
- CN: 外部/系统头文件：`type_traits, complex, thrust/pair.h, cmath`。
- EN: The implementation revolves around symbols such as `max_propagate_nan, min_propagate_nan, WelfordData, reduce, combine, translate_idx, warp_shfl_down, WARP_SHFL_DOWN, WelfordOps, project`.
- CN: 实现围绕 `max_propagate_nan, min_propagate_nan, WelfordData, reduce, combine, translate_idx, warp_shfl_down, WARP_SHFL_DOWN, WelfordOps, project` 等符号展开。
