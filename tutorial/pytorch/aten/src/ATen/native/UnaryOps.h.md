# UnaryOps.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/UnaryOps.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on unary ops; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 unary ops；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once

#include <ATen/native/DispatchStub.h>
#include <ATen/Generator.h>
#include <c10/core/Scalar.h>

namespace at {
class Tensor;
class TensorBase;
struct TensorIteratorBase;
}

namespace at::native {

using unary_fn = void(*)(TensorIteratorBase&);
using unary_fn_with_scalar = void(*)(TensorIteratorBase&, const Scalar& a);

inline namespace CPU_CAPABILITY {
void conj_kernel(TensorIteratorBase &iter);
void neg_kernel(TensorIteratorBase &iter);
void reciprocal_kernel(TensorIteratorBase &iter);
void rsqrt_kernel(TensorIteratorBase& iter);
void sqrt_kernel(TensorIteratorBase& iter);
} // namespace CPU_CAPABILITY

DECLARE_DISPATCH(unary_fn, abs_stub)
DECLARE_DISPATCH(unary_fn, angle_stub)
DECLARE_DISPATCH(unary_fn, conj_physical_stub)
DECLARE_DISPATCH(unary_fn, acos_stub)
DECLARE_DISPATCH(unary_fn, acosh_stub)
```
- EN: Lines 1-30 pull in 3 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 3 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
DECLARE_DISPATCH(unary_fn, asinh_stub)
DECLARE_DISPATCH(unary_fn, atanh_stub)
DECLARE_DISPATCH(unary_fn, asin_stub)
DECLARE_DISPATCH(unary_fn, atan_stub)
DECLARE_DISPATCH(unary_fn, bitwise_not_stub)
DECLARE_DISPATCH(unary_fn, logical_not_stub)
DECLARE_DISPATCH(unary_fn, ceil_stub)
DECLARE_DISPATCH(unary_fn, cos_stub)
DECLARE_DISPATCH(unary_fn, cosh_stub)
DECLARE_DISPATCH(unary_fn, digamma_stub)
DECLARE_DISPATCH(unary_fn, special_entr_stub)
DECLARE_DISPATCH(unary_fn, special_erfcx_stub)
DECLARE_DISPATCH(unary_fn, erf_stub)
DECLARE_DISPATCH(unary_fn, erfc_stub)
DECLARE_DISPATCH(unary_fn, erfinv_stub)
DECLARE_DISPATCH(unary_fn, exp_stub)
DECLARE_DISPATCH(unary_fn, exp2_stub)
DECLARE_DISPATCH(unary_fn, expm1_stub)
DECLARE_DISPATCH(unary_fn, floor_stub)
DECLARE_DISPATCH(unary_fn, frac_stub)
DECLARE_DISPATCH(unary_fn, frexp_stub)
DECLARE_DISPATCH(unary_fn, i0_stub)
DECLARE_DISPATCH(unary_fn, special_i0e_stub)
DECLARE_DISPATCH(unary_fn, special_i1_stub)
DECLARE_DISPATCH(unary_fn, special_i1e_stub)
DECLARE_DISPATCH(unary_fn, log_stub)
DECLARE_DISPATCH(unary_fn, log10_stub)
DECLARE_DISPATCH(unary_fn, log1p_stub)
DECLARE_DISPATCH(unary_fn, log2_stub)
DECLARE_DISPATCH(unary_fn, special_ndtri_stub)
```
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-90
```cpp
DECLARE_DISPATCH(unary_fn, special_log_ndtr_stub)
DECLARE_DISPATCH(unary_fn, neg_stub)

DECLARE_DISPATCH(unary_fn, reciprocal_stub)
DECLARE_DISPATCH(unary_fn, round_stub)
DECLARE_DISPATCH(unary_fn, rsqrt_stub)
DECLARE_DISPATCH(unary_fn, sigmoid_stub)
DECLARE_DISPATCH(unary_fn_with_scalar, logit_stub)
DECLARE_DISPATCH(unary_fn, sign_stub)
DECLARE_DISPATCH(unary_fn, signbit_stub)
DECLARE_DISPATCH(unary_fn, sgn_stub)
DECLARE_DISPATCH(unary_fn, sin_stub)
DECLARE_DISPATCH(unary_fn, sinc_stub)
DECLARE_DISPATCH(unary_fn, sinh_stub)
DECLARE_DISPATCH(unary_fn, sqrt_stub)
DECLARE_DISPATCH(unary_fn, tan_stub)
DECLARE_DISPATCH(unary_fn, tanh_stub)
DECLARE_DISPATCH(unary_fn, trigamma_stub)
DECLARE_DISPATCH(unary_fn, trunc_stub)
DECLARE_DISPATCH(unary_fn, lgamma_stub)
DECLARE_DISPATCH(unary_fn, special_airy_ai_stub)
DECLARE_DISPATCH(unary_fn, special_bessel_j0_stub)
DECLARE_DISPATCH(unary_fn, special_bessel_j1_stub)
DECLARE_DISPATCH(unary_fn, special_bessel_y0_stub)
DECLARE_DISPATCH(unary_fn, special_bessel_y1_stub)
DECLARE_DISPATCH(unary_fn, special_modified_bessel_i0_stub)
DECLARE_DISPATCH(unary_fn, special_modified_bessel_i1_stub)
DECLARE_DISPATCH(unary_fn, special_modified_bessel_k0_stub)
DECLARE_DISPATCH(unary_fn, special_modified_bessel_k1_stub)
DECLARE_DISPATCH(unary_fn, special_scaled_modified_bessel_k0_stub)
```
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。

### Lines 91-120
```cpp
DECLARE_DISPATCH(unary_fn, special_scaled_modified_bessel_k1_stub)
DECLARE_DISPATCH(unary_fn, special_spherical_bessel_j0_stub)

// NB: these are actually defined in Distribution
DECLARE_DISPATCH(void(*)(const TensorBase&, const TensorBase&, std::optional<Generator>), bernoulli_tensor_stub)
DECLARE_DISPATCH(void(*)(const TensorBase&, const double, std::optional<Generator>), bernoulli_scalar_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const double, const double, std::optional<Generator>), cauchy_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const double, std::optional<Generator>), exponential_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const double, std::optional<Generator>), geometric_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const double, const double, std::optional<Generator>), log_normal_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const double, const double, std::optional<Generator>), uniform_stub)
DECLARE_DISPATCH(void(*)(const TensorBase&, const double, const double, std::optional<Generator>), normal_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const uint64_t, const int64_t, std::optional<Generator>), random_from_to_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, std::optional<Generator>), random_full_64_bits_range_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, std::optional<Generator>), random_stub)

DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const int64_t, const double), kaiser_window_stub)
DECLARE_DISPATCH(void(*)(TensorIteratorBase&, const int64_t), polygamma_stub)
DECLARE_DISPATCH(
    void (*)(Tensor&, const Tensor&, int64_t, std::optional<Generator>),
    multinomial_with_replacement_stub)
DECLARE_DISPATCH(
    void (*)(
        TensorIteratorBase&,
        std::optional<double>,
        std::optional<double>,
        std::optional<double>),
    nan_to_num_stub)
DECLARE_DISPATCH(void (*)(TensorIteratorBase&, int64_t), round_decimals_stub)
```
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-128
```cpp
// Missing unary functions
// digamma
// lgamma
// erfinv
// clone
// contiguous
// zero
} // namespace at::native
```
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

## Key Concepts / 关键概念

- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Notable symbols: DECLARE_DISPATCH.
- CN: 重要符号：DECLARE_DISPATCH。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/DispatchStub.h, ATen/Generator.h, c10/core/Scalar.h`.
- CN: 主要内部头文件：`ATen/native/DispatchStub.h, ATen/Generator.h, c10/core/Scalar.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `DECLARE_DISPATCH`.
- CN: 实现围绕 `DECLARE_DISPATCH` 等符号展开。
