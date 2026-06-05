# DistributionsHelper.h — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/DistributionsHelper.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file declares core ATen/C10 abstractions, with primary focus on `uniform_int_from_to_distribution`, `uniform_int_full_range_distribution`, `uniform_int_distribution`.
- 用途（中文）: 该文件声明ATen/C10 核心抽象，核心关注对象是 `uniform_int_from_to_distribution`, `uniform_int_full_range_distribution`, `uniform_int_distribution`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
#pragma once

#include <ATen/core/TransformationHelper.h>
#include <c10/util/Half.h>
#include <c10/util/BFloat16.h>
#include <c10/util/MathConstants.h>
#include <c10/macros/Macros.h>

#include <cmath>
#include <limits>
#include <optional>
#include <type_traits>

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 14-23
```cpp
/**
 * Distributions kernel adapted from THRandom.cpp
 * The kernels try to follow std::random distributions signature
 * For instance: in ATen
 *      auto gen = at::detail::createCPUGenerator();
 *      at::uniform_real_distribution<double> uniform(0, 1);
 *      auto sample = uniform(gen.get());
 *
 *      vs std::random
 *
```
- EN: Focus symbols: `createCPUGenerator`, `uniform`, `get`. This block documents the design intent, assumptions, or usage notes. The comments give maintainers the context needed to interpret the following code correctly.
- CN: 关注符号：`createCPUGenerator`, `uniform`, `get`。该代码块说明设计意图、前提假设或使用说明。这些注释为维护者提供理解后续代码所需的上下文。

### Lines 24-33
```cpp
 *      std::mt19937 gen;
 *      std::uniform_real_distribution uniform(0, 1);
 *      auto sample = uniform(gen);
 */


namespace at {
namespace {

/**
```
- EN: Focus symbols: `at`, `uniform`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`at`, `uniform`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 34-43
```cpp
 * Samples a discrete uniform distribution in the range [base, base+range) of type T
 */
template <typename T>
struct uniform_int_from_to_distribution {

  C10_HOST_DEVICE inline uniform_int_from_to_distribution(uint64_t range, int64_t base) : range_(range), base_(base) {}

  template <typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
#ifdef FBCODE_CAFFE2
```
- EN: Focus symbols: `uniform_int_from_to_distribution`, `range_`, `base_`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`uniform_int_from_to_distribution`, `range_`, `base_`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 44-53
```cpp
    if ((
      std::is_same_v<T, int64_t> ||
      std::is_same_v<T, double> ||
      std::is_same_v<T, float> ||
      std::is_same_v<T, at::BFloat16>) && range_ >= 1ULL << 32)
#else
    if (range_ >= 1ULL << 28) // allow approx 5% skew in uniform int generation using %
#endif
    {
      return transformation::uniform_int_from_to<T>(generator->random64(), range_, base_);
```
- EN: Focus symbols: `random64`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`random64`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 54-63
```cpp
    } else {
      return transformation::uniform_int_from_to<T>(generator->random(), range_, base_);
    }
  }

  private:
    uint64_t range_;
    int64_t base_;
};

```
- EN: Focus symbols: `random`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`random`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 64-74
```cpp
/**
 * Samples a discrete uniform distribution in the range [min_value(int64_t), max_value(int64_t)]
 */
template <typename T>
struct uniform_int_full_range_distribution {

  template <typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    return transformation::uniform_int_full_range<T>(generator->random64());
  }

```
- EN: Focus symbols: `uniform_int_full_range_distribution`, `min_value`, `max_value`, `operator`, `random64`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`uniform_int_full_range_distribution`, `min_value`, `max_value`, `operator`, `random64`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 75-84
```cpp
};

/**
 * Samples a discrete uniform distribution in the range [0, max_value(T)] for integral types
 * and [0, 2^mantissa] for floating-point types.
 */
template <typename T>
struct uniform_int_distribution {

  template <typename RNG>
```
- EN: Focus symbols: `uniform_int_distribution`, `max_value`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`uniform_int_distribution`, `max_value`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 85-94
```cpp
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    if constexpr (std::is_same_v<T, double> || std::is_same_v<T, int64_t>) {
      return transformation::uniform_int<T>(generator->random64());
    } else {
      return transformation::uniform_int<T>(generator->random());
    }
  }

};

```
- EN: Focus symbols: `operator`, `constexpr`, `random64`, `random`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`operator`, `constexpr`, `random64`, `random`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 95-105
```cpp
/**
 * Samples a uniform distribution in the range [from, to) of type T
 */
template <typename T>
struct uniform_real_distribution {

  C10_HOST_DEVICE inline uniform_real_distribution(T from, T to) : from_(from), to_(to) {
    TORCH_CHECK_IF_NOT_ON_CUDA(from <= to);
    TORCH_CHECK_IF_NOT_ON_CUDA(to - from <= std::numeric_limits<T>::max());
  }

```
- EN: Focus symbols: `uniform_real_distribution`, `from_`, `to_`, `TORCH_CHECK_IF_NOT_ON_CUDA`, `max`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`uniform_real_distribution`, `from_`, `to_`, `TORCH_CHECK_IF_NOT_ON_CUDA`, `max`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 106-119
```cpp
  template <typename RNG>
  C10_HOST_DEVICE inline dist_acctype<T> operator()(RNG* generator) const {
    if constexpr (std::is_same_v<T, double>) {
      return transformation::uniform_real<T>(generator->random64(), from_, to_);
    } else {
      return transformation::uniform_real<T>(generator->random(), from_, to_);
    }
  }

  private:
    T from_;
    T to_;
};

```
- EN: Focus symbols: `operator`, `constexpr`, `random64`, `random`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `constexpr`, `random64`, `random`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 120-131
```cpp
template <typename RNG,
          typename = decltype(&RNG::next_double_normal_sample),
          typename = decltype(&RNG::set_next_double_normal_sample)>
C10_HOST_DEVICE bool maybe_get_next_normal_sample(RNG* generator, double* ret) {
  const auto sample = generator->next_double_normal_sample();
  if (!sample.has_value())
    return false;
  *ret = sample.value();
  generator->set_next_double_normal_sample(std::nullopt);
  return true;
}

```
- EN: Focus symbols: `maybe_get_next_normal_sample`, `next_double_normal_sample`, `has_value`, `value`, `set_next_double_normal_sample`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`maybe_get_next_normal_sample`, `next_double_normal_sample`, `has_value`, `value`, `set_next_double_normal_sample`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 132-143
```cpp
template <typename RNG,
          typename = decltype(&RNG::next_float_normal_sample),
          typename = decltype(&RNG::set_next_float_normal_sample)>
C10_HOST_DEVICE bool maybe_get_next_normal_sample(RNG* generator, float* ret) {
  const auto sample = generator->next_float_normal_sample();
  if (!sample.has_value())
    return false;
  *ret = sample.value();
  generator->set_next_float_normal_sample(std::nullopt);
  return true;
}

```
- EN: Focus symbols: `maybe_get_next_normal_sample`, `next_float_normal_sample`, `has_value`, `value`, `set_next_float_normal_sample`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`maybe_get_next_normal_sample`, `next_float_normal_sample`, `has_value`, `value`, `set_next_float_normal_sample`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 144-154
```cpp
template <typename RNG>
C10_HOST_DEVICE bool maybe_get_next_normal_sample(RNG* /* generator */, void* /* ret */) {
  return false;
}

template <typename RNG,
          typename = decltype(&RNG::set_next_double_normal_sample)>
C10_HOST_DEVICE void maybe_set_next_normal_sample(RNG* generator, const double* cache) {
  generator->set_next_double_normal_sample(*cache);
}

```
- EN: Focus symbols: `maybe_get_next_normal_sample`, `maybe_set_next_normal_sample`, `set_next_double_normal_sample`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`maybe_get_next_normal_sample`, `maybe_set_next_normal_sample`, `set_next_double_normal_sample`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 155-164
```cpp
template <typename RNG,
          typename = decltype(&RNG::set_next_float_normal_sample)>
C10_HOST_DEVICE void maybe_set_next_normal_sample(RNG* generator, const float* cache) {
  generator->set_next_float_normal_sample(*cache);
}

template <typename RNG>
C10_HOST_DEVICE void maybe_set_next_normal_sample(RNG* /* generator */, const void* /* cache */) {
}

```
- EN: Focus symbols: `maybe_set_next_normal_sample`, `set_next_float_normal_sample`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`maybe_set_next_normal_sample`, `set_next_float_normal_sample`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 165-177
```cpp
/**
 * Samples a normal distribution using the Box-Muller method
 * Takes mean and standard deviation as inputs
 * Note that Box-muller method returns two samples at a time.
 * Hence, we cache the "next" sample in the CPUGeneratorImpl class.
 */
template <typename T>
struct normal_distribution {

  C10_HOST_DEVICE inline normal_distribution(T mean_in, T stdv_in) : mean(mean_in), stdv(stdv_in) {
    TORCH_CHECK_IF_NOT_ON_CUDA(stdv_in >= 0, "stdv_in must be positive: ", stdv_in);
  }

```
- EN: Focus symbols: `normal_distribution`, `mean`, `stdv`, `TORCH_CHECK_IF_NOT_ON_CUDA`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`normal_distribution`, `mean`, `stdv`, `TORCH_CHECK_IF_NOT_ON_CUDA`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 178-187
```cpp
  template <typename RNG>
  C10_HOST_DEVICE inline dist_acctype<T> operator()(RNG* generator) const {
    dist_acctype<T> ret;
    // return cached values if available
    if (maybe_get_next_normal_sample(generator, &ret)) {
      return transformation::normal(ret, mean, stdv);
    }

    // otherwise generate new normal values
    uniform_real_distribution<T> uniform(0.0, 1.0);
```
- EN: Focus symbols: `operator`, `maybe_get_next_normal_sample`, `normal`, `uniform`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `maybe_get_next_normal_sample`, `normal`, `uniform`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 188-198
```cpp
    const dist_acctype<T> u1 = uniform(generator);
    const dist_acctype<T> u2 = uniform(generator);
    const dist_acctype<T> r = ::sqrt(static_cast<T>(-2.0) * ::log1p(-u2));
    const dist_acctype<T> theta = static_cast<T>(2.0) * c10::pi<T> * u1;
    const dist_acctype<T> sample = r * ::sin(theta);
    maybe_set_next_normal_sample(generator, &sample);

    ret = r * ::cos(theta);
    return transformation::normal(ret, mean, stdv);
  }

```
- EN: Focus symbols: `uniform`, `sqrt`, `log1p`, `sin`, `maybe_set_next_normal_sample`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`uniform`, `sqrt`, `log1p`, `sin`, `maybe_set_next_normal_sample`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 199-208
```cpp
  private:
    T mean;
    T stdv;
};

template <typename T>
struct DiscreteDistributionType { using type = float; };

template <> struct DiscreteDistributionType<double> { using type = double; };

```
- EN: Focus symbols: `DiscreteDistributionType`, `type`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`DiscreteDistributionType`, `type`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 209-218
```cpp
/**
 * Samples a bernoulli distribution given a probability input
 */
template <typename T>
struct bernoulli_distribution {

  C10_HOST_DEVICE inline bernoulli_distribution(T p_in) : p(p_in) {
    TORCH_CHECK_IF_NOT_ON_CUDA(p_in >= 0 && p_in <= 1);
  }

```
- EN: Focus symbols: `bernoulli_distribution`, `p`, `TORCH_CHECK_IF_NOT_ON_CUDA`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`bernoulli_distribution`, `p`, `TORCH_CHECK_IF_NOT_ON_CUDA`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 219-228
```cpp
  template <typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    uniform_real_distribution<T> uniform(0.0, 1.0);
    return transformation::bernoulli<T>(uniform(generator), p);
  }

  private:
    T p;
};

```
- EN: Focus symbols: `operator`, `uniform`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `uniform`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 229-238
```cpp
/**
 * Samples a geometric distribution given a probability input
 */
template <typename T>
struct geometric_distribution {

  C10_HOST_DEVICE inline geometric_distribution(T p_in) : p(p_in) {
    TORCH_CHECK_IF_NOT_ON_CUDA(p_in > 0 && p_in < 1);
  }

```
- EN: Focus symbols: `geometric_distribution`, `p`, `TORCH_CHECK_IF_NOT_ON_CUDA`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`geometric_distribution`, `p`, `TORCH_CHECK_IF_NOT_ON_CUDA`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 239-248
```cpp
  template <typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    uniform_real_distribution<T> uniform(0.0, 1.0);
    return transformation::geometric<T>(uniform(generator), p);
  }

  private:
    T p;
};

```
- EN: Focus symbols: `operator`, `uniform`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `uniform`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 249-262
```cpp
/**
 * Samples an exponential distribution given a lambda input
 */
template <typename T>
struct exponential_distribution {

  C10_HOST_DEVICE inline exponential_distribution(T lambda_in) : lambda(lambda_in) {}

  template <typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    uniform_real_distribution<T> uniform(0.0, 1.0);
    return transformation::exponential<T>(uniform(generator), lambda);
  }

```
- EN: Focus symbols: `exponential_distribution`, `lambda`, `operator`, `uniform`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`exponential_distribution`, `lambda`, `operator`, `uniform`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 263-274
```cpp
  private:
    T lambda;
};

/**
 * Samples a cauchy distribution given median and sigma as inputs
 */
template <typename T>
struct cauchy_distribution {

  C10_HOST_DEVICE inline cauchy_distribution(T median_in, T sigma_in) : median(median_in), sigma(sigma_in) {}

```
- EN: Focus symbols: `cauchy_distribution`, `median`, `sigma`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`cauchy_distribution`, `median`, `sigma`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 275-285
```cpp
  template <typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    uniform_real_distribution<T> uniform(0.0, 1.0);
    return transformation::cauchy<T>(uniform(generator), median, sigma);
  }

  private:
    T median;
    T sigma;
};

```
- EN: Focus symbols: `operator`, `uniform`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `uniform`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 286-297
```cpp
/**
 * Samples a lognormal distribution
 * Takes mean and standard deviation as inputs
 * Outputs two samples at a time
 */
template <typename T>
struct lognormal_distribution {

  C10_HOST_DEVICE inline lognormal_distribution(T mean_in, T stdv_in) : mean(mean_in), stdv(stdv_in) {
    TORCH_CHECK_IF_NOT_ON_CUDA(stdv_in > 0);
  }

```
- EN: Focus symbols: `lognormal_distribution`, `mean`, `stdv`, `TORCH_CHECK_IF_NOT_ON_CUDA`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`lognormal_distribution`, `mean`, `stdv`, `TORCH_CHECK_IF_NOT_ON_CUDA`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 298-307
```cpp
  template<typename RNG>
  C10_HOST_DEVICE inline T operator()(RNG* generator) const {
    normal_distribution<T> normal(mean, stdv);
    return transformation::log_normal<T>(normal(generator));
  }

  private:
    T mean;
    T stdv;
};
```
- EN: Focus symbols: `operator`, `normal`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `normal`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 308-309
```cpp
}
} // namespace at
```
- EN: Focus symbols: `at`. This block opens or closes namespaces to keep symbols in the intended API scope. This keeps the implementation aligned with the broader ATen/C10 module hierarchy.
- CN: 关注符号：`at`。该代码块打开或关闭命名空间以维持符号的预期作用域。这使实现与更大的 ATen/C10 模块层次保持一致。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- C++ templates and specialization / C++ 模板与特化
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/TransformationHelper.h`, `c10/util/Half.h`, `c10/util/BFloat16.h`, `c10/util/MathConstants.h`, `c10/macros/Macros.h`
- External/system includes / 外部或系统头: `cmath`, `limits`, `optional`, `type_traits`
- Inferred semantic dependencies / 推断出的语义依赖: C++ templates / C++ 模板; runtime validation / 运行时校验; namespace scoping / 命名空间作用域
