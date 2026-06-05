# ScaledBlasUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/ScaledBlasUtils.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on scaled blas utils; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 scaled blas utils；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <cstdint>
#include <c10/util/typeid.h>
#include <c10/util/Exception.h>
#include <c10/util/SmallVector.h>
#include <c10/core/Scalar.h>
#include <c10/core/ScalarType.h>
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS

#include <ATen/BlasBackend.h>
#include <ATen/core/Tensor.h>
#include <ATen/core/NamedTensor.h>
#include <ATen/Dispatch.h>
#include <ATen/ExpandUtils.h>
#include <ATen/OpMathType.h>
#include <ATen/TensorUtils.h>
#include <ATen/native/Resize.h>
#include <c10/util/MaybeOwned.h>
#include <ATen/native/GroupedMMUtils.h>
#include <ATen/native/ScaledBlasUtils.h>
#include <ATen/ceil_div.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_addmm_activation_native.h>
#include <ATen/ops/_efficientzerotensor.h>
#include <ATen/ops/_scaled_mm_native.h>
#include <ATen/ops/_unsafe_view_native.h>
#include <ATen/ops/abs.h>
```
- EN: Lines 1-30 pull in 25 header dependencies, defining the compilation surface for this segment.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 25 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
#include <ATen/ops/addmm_native.h>
#include <ATen/ops/addmv_native.h>
#include <ATen/ops/baddbmm_native.h>
#include <ATen/ops/bmm_native.h>
#include <ATen/ops/copy_native.h>
#include <ATen/ops/dot_native.h>
#include <ATen/ops/empty.h>
#include <ATen/ops/empty_strided.h>
#include <ATen/ops/gelu.h>
#include <ATen/ops/max.h>
#include <ATen/ops/mm_native.h>
#include <ATen/ops/mul.h>
#include <ATen/ops/relu.h>
#include <ATen/ops/ones.h>
#include <ATen/ops/scalar_tensor_native.h>
#include <ATen/ops/vdot_native.h>
#endif

using at::blas::ScalingType;
using at::blas::SwizzleType;

namespace at::native::scaled {

/**
 * Both inputs must be fp8,
 * Each needs a single scale, {Tensorwise (float)}
 */
bool check_tensorwise_recipe(
    c10::ScalarType type_a,
    std::vector<ScalingType>& recipe_a,
```
- EN: Lines 31-60 pull in 16 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 31-60 行引入了 16 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
    c10::ArrayRef<Tensor>& scales_a,
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    c10::ArrayRef<Tensor>& scales_b) {
  // both types must be fp8
  if (!isFloat8Type(type_a) || !isFloat8Type(type_b)) {
    return false;
  }

  // 1 scale each, {Tensorwise, float}
  if (scales_a.size() != 1 || recipe_a.size() != 1 || scales_b.size() != 1 || recipe_b.size() != 1) {
    return false;
  }
  // Need {Blockwise_1x32, e8m0} for A & B
  if (recipe_a[0] != ScalingType::TensorWise) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float) return false;
  if (recipe_b[0] != ScalingType::TensorWise) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float) return false;

  return true;
}

/**
 * Both inputs must be fp8,
 * Each needs scales, {Rowwise (float)}
 */
bool check_rowwise_recipe(
    c10::ScalarType type_a,
    std::vector<ScalingType>& recipe_a,
    ArrayRef<Tensor>& scales_a,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    ArrayRef<Tensor>& scales_b) {
  // both types must be fp8
  if (!isFloat8Type(type_a) || !isFloat8Type(type_b)) {
    return false;
  }

  // 1 scale each, {Tensorwise, float}
  if (scales_a.size() != 1 || recipe_a.size() != 1 || scales_b.size() != 1 || recipe_b.size() != 1) {
    return false;
  }

  // Need {RowWise, dp32} for A & B
  if (recipe_a[0] != ScalingType::RowWise) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float) return false;
  if (recipe_b[0] != ScalingType::RowWise) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float) return false;

  return true;
}

/**
 * Two-level scaling, canonical NVFP4
 * Both inputs must be fp4
 * A, B need 2 scales, {Blockwise_1x16 (e4m3), Tensorwise (fp32)}
 */
bool check_nvfp4_recipe(
    c10::ScalarType type_a,
    std::vector<ScalingType>& recipe_a,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
    ArrayRef<Tensor>& scales_a,
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    ArrayRef<Tensor>& scales_b) {
  // both types must be fp4
  if (type_a != ScalarType::Float4_e2m1fn_x2 || type_b != ScalarType::Float4_e2m1fn_x2) {
    return false;
  }

  // 2 scales, 2 recipes for each input
  if (scales_a.size() != 2 || recipe_a.size() != 2 || scales_b.size() != 2 || recipe_b.size() != 2) {
    return false;
  }

  // Need {Blockwise_1x16, e4m3 for scale[0], Tensorwise, fp32 for scale[1]}
  if (recipe_a[0] != ScalingType::BlockWise1x16 || recipe_a[1] != ScalingType::TensorWise) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float8_e4m3fn || scales_a[1].scalar_type() != ScalarType::Float) return false;
  if (recipe_b[0] != ScalingType::BlockWise1x16 || recipe_b[1] != ScalingType::TensorWise) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float8_e4m3fn || scales_b[1].scalar_type() != ScalarType::Float) return false;

  return true;
}

/**
 * Single-level scaling, what PyT currently understands
 * Both inputs must be fp4
 * A, B need 1 scale, {Blockwise_1x16 (e4m3)}
 */
bool check_nvfp4_recipe_single_scale(
    c10::ScalarType type_a,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-180
```cpp
    std::vector<ScalingType>& recipe_a,
    ArrayRef<Tensor>& scales_a,
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    ArrayRef<Tensor>& scales_b) {
  // both types must be fp4
  if (type_a != ScalarType::Float4_e2m1fn_x2 || type_b != ScalarType::Float4_e2m1fn_x2) {
    return false;
  }

  // 2 scales, 2 recipes for each input
  if (scales_a.size() != 1 || recipe_a.size() != 1 || scales_b.size() != 1 || recipe_b.size() != 1) {
    return false;
  }

  // Need {Blockwise_1x16, e4m3 for scale[0], Tensorwise, fp32 for scale[1]}
  if (recipe_a[0] != ScalingType::BlockWise1x16) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float8_e4m3fn) return false;
  if (recipe_b[0] != ScalingType::BlockWise1x16) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float8_e4m3fn) return false;

  return true;
}

/**
 * Both inputs must be fp8
 * A, B must only have 1 scale each, A: {Blockwise_1x128 (float), B:
 * {Blockwise_128x128 (float)
 */
bool check_deepseek_recipe(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are Blockwise_128x128, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 Blockwise_128x128，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-210
```cpp
    ScalingType expected_recipe_a,
    ScalingType expected_recipe_b,
    c10::ScalarType type_a,
    std::vector<ScalingType>& recipe_a,
    ArrayRef<Tensor>& scales_a,
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    ArrayRef<Tensor>& scales_b) {
  // both types must be fp8
  if (type_a != ScalarType::Float8_e4m3fn || type_b != ScalarType::Float8_e4m3fn) {
    return false;
  }

  // 1 scales, 1 recipes for each input
  if (scales_a.size() != 1 || recipe_a.size() != 1 || scales_b.size() != 1 || recipe_b.size() != 1) {
    return false;
  }

  // Need {Blockwise_1x128, float} for A, {Blockwise_128x128, float} for B
  if (recipe_a[0] != expected_recipe_a) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float) return false;
  if (recipe_b[0] != expected_recipe_b) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float) return false;

  return true;
}

/**
 * Both inputs must be fp8
 * A, B must have 1 scale each, {Blockwise_1x32, e8m0}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 211-240
```cpp
 */
bool check_mxfp8_recipe(
    c10::ScalarType type_a,
    std::vector<ScalingType>& recipe_a,
    ArrayRef<Tensor>& scales_a,
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    ArrayRef<Tensor>& scales_b) {
  // both types must be fp8
  if (type_a != ScalarType::Float8_e4m3fn || type_b != ScalarType::Float8_e4m3fn) {
    return false;
  }

  // 1 scales, 1 recipes for each input
  if (scales_a.size() != 1 || recipe_a.size() != 1 || scales_b.size() != 1 || recipe_b.size() != 1) {
    return false;
  }

  // Need {Blockwise_1x32, e8m0} for A & B
  if (recipe_a[0] != ScalingType::BlockWise1x32) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float8_e8m0fnu) return false;
  if (recipe_b[0] != ScalingType::BlockWise1x32) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float8_e8m0fnu) return false;

  return true;
}

/**
 * Both inputs must be fp4
 * A, B must have 1 scale each, {Blockwise_1x32, e8m0}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-268
```cpp
 */
bool check_mxfp4_recipe(
    c10::ScalarType type_a,
    std::vector<ScalingType>& recipe_a,
    ArrayRef<Tensor>& scales_a,
    c10::ScalarType type_b,
    std::vector<ScalingType>& recipe_b,
    ArrayRef<Tensor>& scales_b) {
  // both types must be fp4
  if (type_a != ScalarType::Float4_e2m1fn_x2 || type_b != ScalarType::Float4_e2m1fn_x2) {
    return false;
  }

  // 1 scales, 1 recipes for each input
  if (scales_a.size() != 1 || recipe_a.size() != 1 || scales_b.size() != 1 || recipe_b.size() != 1) {
    return false;
  }

  // Need {Blockwise_1x32, e8m0} for A & B
  if (recipe_a[0] != ScalingType::BlockWise1x32) return false;
  if (scales_a[0].scalar_type() != ScalarType::Float8_e8m0fnu) return false;
  if (recipe_b[0] != ScalingType::BlockWise1x32) return false;
  if (scales_b[0].scalar_type() != ScalarType::Float8_e8m0fnu) return false;

  return true;
}

}  // at::scaled
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Notable symbols: check_tensorwise_recipe, check_rowwise_recipe, check_nvfp4_recipe, check_nvfp4_recipe_single_scale, check_deepseek_recipe, check_mxfp8_recipe, check_mxfp4_recipe.
- CN: 重要符号：check_tensorwise_recipe, check_rowwise_recipe, check_nvfp4_recipe, check_nvfp4_recipe_single_scale, check_deepseek_recipe, check_mxfp8_recipe, check_mxfp4_recipe。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/util/typeid.h, c10/util/Exception.h, c10/util/SmallVector.h, c10/core/Scalar.h, c10/core/ScalarType.h, ATen/BlasBackend.h, ATen/core/Tensor.h, ATen/core/NamedTensor.h, ATen/Dispatch.h, ATen/ExpandUtils.h, ATen/OpMathType.h, ATen/TensorUtils.h`.
- CN: 主要内部头文件：`c10/util/typeid.h, c10/util/Exception.h, c10/util/SmallVector.h, c10/core/Scalar.h, c10/core/ScalarType.h, ATen/BlasBackend.h, ATen/core/Tensor.h, ATen/core/NamedTensor.h, ATen/Dispatch.h, ATen/ExpandUtils.h, ATen/OpMathType.h, ATen/TensorUtils.h`。
- EN: External/system headers: `cstdint`.
- CN: 外部/系统头文件：`cstdint`。
- EN: The implementation revolves around symbols such as `check_tensorwise_recipe, check_rowwise_recipe, check_nvfp4_recipe, check_nvfp4_recipe_single_scale, check_deepseek_recipe, check_mxfp8_recipe, check_mxfp4_recipe`.
- CN: 实现围绕 `check_tensorwise_recipe, check_rowwise_recipe, check_nvfp4_recipe, check_nvfp4_recipe_single_scale, check_deepseek_recipe, check_mxfp8_recipe, check_mxfp4_recipe` 等符号展开。
