# LowerQuantOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Quant/Transforms/LowerQuantOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Transforms `quant.dcast` and `quant.qcast` into lower-level ops.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Quant/Transforms`，围绕 Quant 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- LowerQuantOps.cpp - Lower 'quant' dialect ops ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Transforms `quant.dcast` and `quant.qcast` into lower-level ops.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Quant/IR/Quant.h"
#include "mlir/Dialect/Quant/IR/QuantTypes.h"
#include "mlir/Dialect/Quant/Transforms/Passes.h"
#include "mlir/Dialect/Shape/IR/Shape.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/DialectConversion.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Quant/IR/Quant.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Quant/IR/Quant.h`。

### Lines 25-40
```cpp
namespace mlir {
namespace quant {

#define GEN_PASS_DEF_LOWERQUANTOPS
#include "mlir/Dialect/Quant/Transforms/Passes.h.inc"

namespace {

// If 'inputType' is a tensor, return its element type. If it is a scalar,
// return it as is.
Type getScalarType(Type inputType) {
  if (auto tensorType = dyn_cast<TensorType>(inputType))
    return tensorType.getElementType();
  return inputType;
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Quant/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Quant/Transforms/Passes.h.inc`。

### Lines 41-59
```cpp
// Return the shape of an input value as a list of attributes (static
// dimensions) and values (dynamic dimensions). If 'input' is a scalar, an empty
// list is returned. If 'input' is a tensor, its shape is returned.
SmallVector<OpFoldResult> getScalarOrTensorShape(OpBuilder &builder,
                                                 Location loc, Value input) {
  if (isa<TensorType>(input.getType()))
    return tensor::getMixedSizes(builder, loc, input);
  return {};
}

// If 'referenceType' is a scalar, return 'elementType' as is. If
// 'referenceType' is a tensor, return another tensor with the same shape and
// elements of type 'elementType'.
Type getScalarOrTensorType(Type elementType, Type referenceType) {
  if (auto tensorType = dyn_cast<TensorType>(referenceType))
    return tensorType.clone(elementType);
  return elementType;
}

```
- **EN**: Implements logic around `getScalarOrTensorShape`, `getType`, `getMixedSizes`, `getScalarOrTensorType`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getScalarOrTensorShape`, `getType`, `getMixedSizes`, `getScalarOrTensorType`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 60-78
```cpp
// Return a constant with the given value. If 'referenceType' is a tensor, a
// tensor splat of shape 'referenceShape' is returned. If 'referenceType' is a
// scalar, 'referenceShape' is ignored and a scalar constant is returned.
Value getScalarOrTensorConstant(OpBuilder &builder, Location loc, Value scalar,
                                Type referenceType,
                                ArrayRef<OpFoldResult> referenceShape) {
  // If the result type is a scalar, return the unmodified scalar constant.
  auto tensorType = dyn_cast<TensorType>(referenceType);
  if (!tensorType) {
    assert(referenceShape.empty());
    return scalar;
  }

  // Create tensor splat
  auto tensorConstant =
      tensor::SplatOp::create(builder, loc, scalar, referenceShape);
  return tensorConstant;
}

```
- **EN**: Implements logic around `getScalarOrTensorConstant`, `assert`, `create`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getScalarOrTensorConstant`, `assert`, `create` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 79-100
```cpp
// Reshape an unranked tensor into a 1D ranked tensor.
//
// - input
//   Unranked tensor.
//
// Return values:
//
// - flatInput
//   1D ranked, dynamically shaped tensor.
//
// - inputShape
//   1D extent tensor containing the shape of the original unranked input.
//
std::pair<Value, Value> flattenUnrankedTensor(OpBuilder &builder, Location loc,
                                              Value input) {
  // Get unranked input shape and total size
  auto *context = builder.getContext();
  auto shapeType = shape::getExtentTensorType(context);
  auto inputShape = shape::ShapeOfOp::create(builder, loc, shapeType, input);
  Value inputSize = shape::NumElementsOp::create(
      builder, loc, builder.getIndexType(), inputShape);

```
- **EN**: Implements logic around `flattenUnrankedTensor`, `getContext`, `getExtentTensorType`, `create`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `flattenUnrankedTensor`, `getContext`, `getExtentTensorType`, `create`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 101-115
```cpp
  // Turn input size into 1D tensor
  auto flatShapeType = shape::getExtentTensorType(context, 1);
  auto flatInputShape =
      tensor::FromElementsOp::create(builder, loc, flatShapeType, inputSize);

  // Reshape input tensor into 1D
  auto inputType = cast<UnrankedTensorType>(input.getType());
  auto elementType = inputType.getElementType();
  auto flatInputType =
      RankedTensorType::get({ShapedType::kDynamic}, elementType);
  auto flatInput = tensor::ReshapeOp::create(builder, loc, flatInputType, input,
                                             flatInputShape);
  return std::make_pair(flatInput, inputShape);
}

```
- **EN**: Implements logic around `getExtentTensorType`, `create`, `getType`, `getElementType`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getExtentTensorType`, `create`, `getType`, `getElementType`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 116-143
```cpp
// Reshape an unranked tensor into a 3D ranked tensor where the central
// dimension of the result tensor corresponds to dimension 'axis' of the input
// tensor.
//
// - input
//   Unranked tensor.
//
// - axis
//   Index of the input dimension around which other input dimiensions will be
//   collapsed.
//
// - axisSize
//   Size of input dimension 'axis'.
//
// Return values:
//
// - flatInput
//   3D ranked tensor of shape [?, axisSize, ?].
//
// - inputShape
//   1D extent tensor containing the shape of the original unranked input.
//
std::pair<Value, Value>
flattenUnrankedTensorAroundAxis(OpBuilder &builder, Location loc, Value input,
                                int64_t axis, int64_t axisSize) {
  // Get full tensor shape
  auto *context = builder.getContext();
  auto indexType = builder.getIndexType();
```
- **EN**: Implements logic around `flattenUnrankedTensorAroundAxis`, `getContext`, `getIndexType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `flattenUnrankedTensorAroundAxis`, `getContext`, `getIndexType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 144-162
```cpp
  auto shapeType = shape::getExtentTensorType(context);
  auto inputShape = shape::ShapeOfOp::create(builder, loc, shapeType, input);

  // Get shape and sizes on left and right of axis
  auto axisValue = arith::ConstantIndexOp::create(builder, loc, axis);
  auto axisNextValue = arith::ConstantIndexOp::create(builder, loc, axis + 1);
  auto shapeLeft =
      shape::SplitAtOp::create(builder, loc, TypeRange{shapeType, shapeType},
                               inputShape, axisValue)
          .getResult(0);
  auto sizeLeft =
      shape::NumElementsOp::create(builder, loc, indexType, shapeLeft);
  auto shapeRight =
      shape::SplitAtOp::create(builder, loc, TypeRange{shapeType, shapeType},
                               inputShape, axisNextValue)
          .getResult(1);
  auto sizeRight =
      shape::NumElementsOp::create(builder, loc, indexType, shapeRight);

```
- **EN**: Implements logic around `getExtentTensorType`, `create`, `getResult`.
- **CN**: 围绕 `getExtentTensorType`, `create`, `getResult` 实现具体逻辑。

### Lines 163-177
```cpp
  // Compute flat input shape as a 3-element 1D tensor
  auto axisSizeValue = arith::ConstantIndexOp::create(builder, loc, axisSize);
  auto flatShapeType = shape::getExtentTensorType(context, 3);
  auto flatInputShape = tensor::FromElementsOp::create(
      builder, loc, flatShapeType,
      ValueRange{sizeLeft, axisSizeValue, sizeRight});

  // Reshape input to 3D tensor
  auto inputType = cast<UnrankedTensorType>(input.getType());
  auto elementType = inputType.getElementType();
  auto flatInputType = RankedTensorType::get(
      {ShapedType::kDynamic, axisSize, ShapedType::kDynamic}, elementType);
  auto flatInput = tensor::ReshapeOp::create(builder, loc, flatInputType, input,
                                             flatInputShape);

```
- **EN**: Implements logic around `create`, `getExtentTensorType`, `getType`, `getElementType`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getExtentTensorType`, `getType`, `getElementType`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 178-197
```cpp
  return std::make_pair(flatInput, inputShape);
}

// Reshape an input tensor into its original unranked shape.
//
// - input
//   Ranked tensor.
//
// - inputShape
//   1D extent tensor.
//
Value restoreUnrankedTensorShape(OpBuilder &builder, Location loc, Value input,
                                 Value inputShape) {
  auto inputType = cast<RankedTensorType>(input.getType());
  auto elementType = inputType.getElementType();
  auto unrankedType = UnrankedTensorType::get(elementType);
  return tensor::ReshapeOp::create(builder, loc, unrankedType, input,
                                   inputShape);
}

```
- **EN**: Implements logic around `make_pair`, `restoreUnrankedTensorShape`, `getType`, `getElementType`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `make_pair`, `restoreUnrankedTensorShape`, `getType`, `getElementType`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 198-219
```cpp
// Create a tensor constant containing all scales in a per-channel quantized
// type. Example:
//
//   !quant.uniform<i8:f32:1, {2.0:10, 3.0:20}>
//
// produces
//
//   %cst = arith.constant dense<[2.0, 3.0]> : tensor<2xf32>
//
Value materializePerChannelScales(OpBuilder &builder, Location loc,
                                  UniformQuantizedPerAxisType quantizedType) {
  auto scales = quantizedType.getScales();
  auto expressedType = quantizedType.getExpressedType();
  auto scaleAttrs = llvm::map_to_vector(scales, [&](double scale) -> Attribute {
    return builder.getFloatAttr(expressedType, scale);
  });
  auto tensorType =
      RankedTensorType::get({(int64_t)scales.size()}, expressedType);
  auto scalesAttr = DenseElementsAttr::get(tensorType, scaleAttrs);
  return arith::ConstantOp::create(builder, loc, tensorType, scalesAttr);
}

```
- **EN**: Implements logic around `materializePerChannelScales`, `getScales`, `getExpressedType`, `map_to_vector`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `materializePerChannelScales`, `getScales`, `getExpressedType`, `map_to_vector`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 220-243
```cpp
// Create a tensor constant containing all zero points in a per-channel
// quantized type. Example:
//
//   !quant.uniform<i8:f32:1, {2.0:10, 3.0:20}>
//
// produces
//
//   %cst = arith.constant dense<[10, 20]> : tensor<2xi8>
//
Value materializePerChannelZeroPoints(
    OpBuilder &builder, Location loc,
    UniformQuantizedPerAxisType quantizedType) {
  auto zeroPoints = quantizedType.getZeroPoints();
  auto storageType = quantizedType.getStorageType();
  auto zeroPointAttrs =
      llvm::map_to_vector(zeroPoints, [&](int64_t zeroPoint) -> Attribute {
        return builder.getIntegerAttr(storageType, zeroPoint);
      });
  auto tensorType =
      RankedTensorType::get({(int64_t)zeroPoints.size()}, storageType);
  auto zeroPointsAttr = DenseElementsAttr::get(tensorType, zeroPointAttrs);
  return arith::ConstantOp::create(builder, loc, tensorType, zeroPointsAttr);
}

```
- **EN**: Implements logic around `materializePerChannelZeroPoints`, `getZeroPoints`, `getStorageType`, `map_to_vector`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `materializePerChannelZeroPoints`, `getZeroPoints`, `getStorageType`, `map_to_vector`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 244-267
```cpp
// Create a tensor constant containing all scales in a sub-channel quantized
// type. Example:
//
//   !quant.uniform<i8:f32:{0:1,1:2}, {{2.0:10, 3.0:20}, {4.0:30, 5.0:40}}>
//
// produces
//
//   %cst = arith.constant dense<[[2.0, 3.0], [4.0, 5.0]]> : tensor<2x2xf32>
//
Value materializeSubChannelScales(
    OpBuilder &builder, Location loc,
    UniformQuantizedSubChannelType quantizedType) {
  auto scales = quantizedType.getScales();
  auto expressedType = quantizedType.getExpressedType();
  auto scaleAttrs = llvm::map_to_vector(
      scales.getValues<APFloat>(), [&](APFloat scale) -> Attribute {
        return builder.getFloatAttr(expressedType, scale);
      });
  auto tensorType =
      RankedTensorType::get(scales.getType().getShape(), expressedType);
  auto scalesAttr = DenseElementsAttr::get(tensorType, scaleAttrs);
  return arith::ConstantOp::create(builder, loc, tensorType, scalesAttr);
}

```
- **EN**: Implements logic around `materializeSubChannelScales`, `getScales`, `getExpressedType`, `map_to_vector`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `materializeSubChannelScales`, `getScales`, `getExpressedType`, `map_to_vector`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 268-291
```cpp
// Create a tensor constant containing all zero points in a sub-channel
// quantized type. Example:
//
//   !quant.uniform<i8:f32:{0:1,1:2}, {{2.0:10, 3.0:20}, {4.0:30, 5.0:40}}>
//
// produces
//
//   %cst = arith.constant dense<[[10, 20], [30, 40]]> : tensor<2x2xi8>
//
Value materializeSubChannelZeroPoints(
    OpBuilder &builder, Location loc,
    UniformQuantizedSubChannelType quantizedType) {
  auto zeroPoints = quantizedType.getZeroPoints();
  auto storageType = quantizedType.getStorageType();
  auto zeroPointAttrs = llvm::map_to_vector(
      zeroPoints.getValues<APInt>(), [&](APInt zeroPoint) -> Attribute {
        return builder.getIntegerAttr(storageType, zeroPoint);
      });
  auto tensorType =
      RankedTensorType::get(zeroPoints.getType().getShape(), storageType);
  auto zeroPointsAttr = DenseElementsAttr::get(tensorType, zeroPointAttrs);
  return arith::ConstantOp::create(builder, loc, tensorType, zeroPointsAttr);
}

```
- **EN**: Implements logic around `materializeSubChannelZeroPoints`, `getZeroPoints`, `getStorageType`, `map_to_vector`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `materializeSubChannelZeroPoints`, `getZeroPoints`, `getStorageType`, `map_to_vector`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 292-312
```cpp
// Clamp the given scalar or tensor input using the storage bounds encoded in
// the given quantized type, if present.
//
// - input
//   Scalar or ranked tensor input. The element type must match the storage type
//   of 'quantizedType'.
//
// - inputShape
//   If 'input' is a tensor, combination of attributes/values representing its
//   static/dynamic dimensions. If 'input' is a scalar, empty list.
//
// - quantizedType
//   Per-axis or per-channel quantized type.
Value clampScalarOrTensor(OpBuilder &builder, Location loc, Value input,
                          ArrayRef<OpFoldResult> inputShape,
                          QuantizedType quantizedType) {
  // If quantized type does not narrow down the storage type range, there is
  // nothing to do.
  if (!quantizedType.hasStorageTypeBounds())
    return input;

```
- **EN**: Implements logic around `clampScalarOrTensor`, `hasStorageTypeBounds`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `clampScalarOrTensor`, `hasStorageTypeBounds` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 313-335
```cpp
  // Materialize bounds
  auto inputType = input.getType();
  auto storageType = quantizedType.getStorageType();
  auto storageMinScalar = arith::ConstantIntOp::create(
      builder, loc, storageType, quantizedType.getStorageTypeMin());
  auto storageMaxScalar = arith::ConstantIntOp::create(
      builder, loc, storageType, quantizedType.getStorageTypeMax());
  auto storageMin = getScalarOrTensorConstant(builder, loc, storageMinScalar,
                                              inputType, inputShape);
  auto storageMax = getScalarOrTensorConstant(builder, loc, storageMaxScalar,
                                              inputType, inputShape);

  // Clamp
  if (quantizedType.isSigned()) {
    input = arith::MaxSIOp::create(builder, loc, input, storageMin);
    input = arith::MinSIOp::create(builder, loc, input, storageMax);
  } else {
    input = arith::MaxUIOp::create(builder, loc, input, storageMin);
    input = arith::MinUIOp::create(builder, loc, input, storageMax);
  }
  return input;
}

```
- **EN**: Implements logic around `getType`, `getStorageType`, `create`, `getStorageTypeMin`, and 3 more symbols.
- **CN**: 围绕 `getType`, `getStorageType`, `create`, `getStorageTypeMin`, and 3 more symbols 实现具体逻辑。

### Lines 336-351
```cpp
// Emit op 'arith.fptosi' or 'arith.fptoui'.
Value convertFloatToInteger(OpBuilder &builder, Location loc, Value input,
                            Type resultType, bool isSigned) {
  if (isSigned)
    return arith::FPToSIOp::create(builder, loc, resultType, input);
  return arith::FPToUIOp::create(builder, loc, resultType, input);
}

// Emit op 'arith.sitofp' or 'arith.uitofp'.
Value convertIntegerToFloat(OpBuilder &builder, Location loc, Value input,
                            Type resultType, bool isSigned) {
  if (isSigned)
    return arith::SIToFPOp::create(builder, loc, resultType, input);
  return arith::UIToFPOp::create(builder, loc, resultType, input);
}

```
- **EN**: Implements logic around `convertFloatToInteger`, `create`, `convertIntegerToFloat`.
- **CN**: 围绕 `convertFloatToInteger`, `create`, `convertIntegerToFloat` 实现具体逻辑。

### Lines 352-365
```cpp
// Quantize a scalar or ranked tensor value. The stored value is clamped using
// the storage bounds encoded in the given quantized type.
//
// See function 'convertRanked()' below for a description of the arguments.
Value quantizeValue(OpBuilder &builder, Location loc, Value input,
                    ArrayRef<OpFoldResult> inputShape, Value scale,
                    Value zeroPoint, QuantizedType quantizedType) {
  // Convert scale to tensor if necessary
  auto inputType = input.getType();
  scale = getScalarOrTensorConstant(builder, loc, scale, inputType, inputShape);

  // Scale input
  auto scaledValue = arith::DivFOp::create(builder, loc, input, scale);

```
- **EN**: Implements logic around `quantizeValue`, `getType`, `getScalarOrTensorConstant`, `create`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `quantizeValue`, `getType`, `getScalarOrTensorConstant`, `create` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 366-381
```cpp
  // Skip unnecessary computations if no zero point is given
  Value storedValueFloat = scaledValue;
  if (!matchPattern(zeroPoint, m_Zero())) {
    // Convert zero point to tensor if necessary
    zeroPoint = getScalarOrTensorConstant(builder, loc, zeroPoint, inputType,
                                          inputShape);

    // Convert zero point from storage to expressed type
    zeroPoint = convertIntegerToFloat(builder, loc, zeroPoint, scale.getType(),
                                      quantizedType.isSigned());

    // Add zero point to stored value
    storedValueFloat =
        arith::AddFOp::create(builder, loc, scaledValue, zeroPoint);
  }

```
- **EN**: Implements logic around `matchPattern`, `getScalarOrTensorConstant`, `convertIntegerToFloat`, `isSigned`, and 1 more symbols; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchPattern`, `getScalarOrTensorConstant`, `convertIntegerToFloat`, `isSigned`, and 1 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 382-404
```cpp
  // Convert stored value to storage type
  auto storageScalarOrTensorType =
      getScalarOrTensorType(quantizedType.getStorageType(), inputType);
  auto storedValueInt = convertFloatToInteger(builder, loc, storedValueFloat,
                                              storageScalarOrTensorType,
                                              quantizedType.isSigned());

  // Clamp stored value it if the storage type is bound
  auto storedValueClamped = clampScalarOrTensor(builder, loc, storedValueInt,
                                                inputShape, quantizedType);
  return storedValueClamped;
}

// Dequantize a scalar or ranked tensor input.
//
// See function 'convertRanked()' below for a description of the arguments.
Value dequantizeValue(OpBuilder &builder, Location loc, Value input,
                      ArrayRef<OpFoldResult> inputShape, Value scale,
                      Value zeroPoint, QuantizedType quantizedType) {
  // Convert scale to tensor if necessary
  auto inputType = input.getType();
  scale = getScalarOrTensorConstant(builder, loc, scale, inputType, inputShape);

```
- **EN**: Implements logic around `getScalarOrTensorType`, `convertFloatToInteger`, `isSigned`, `clampScalarOrTensor`, and 3 more symbols; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getScalarOrTensorType`, `convertFloatToInteger`, `isSigned`, `clampScalarOrTensor`, and 3 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 405-418
```cpp
  // Convert stored value to float
  auto result = convertIntegerToFloat(builder, loc, input, scale.getType(),
                                      quantizedType.isSigned());

  // Skip unnecessary computations if no zero point is given
  if (!matchPattern(zeroPoint, m_Zero())) {
    // Convert zero point to tensor if necessary
    zeroPoint = getScalarOrTensorConstant(builder, loc, zeroPoint, inputType,
                                          inputShape);

    // Convert zero point from storage to expressed type
    zeroPoint = convertIntegerToFloat(builder, loc, zeroPoint, scale.getType(),
                                      quantizedType.isSigned());

```
- **EN**: Implements logic around `convertIntegerToFloat`, `isSigned`, `matchPattern`, `getScalarOrTensorConstant`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertIntegerToFloat`, `isSigned`, `matchPattern`, `getScalarOrTensorConstant` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 419-446
```cpp
    // Subtract zero point to stored value
    result = arith::SubFOp::create(builder, loc, result, zeroPoint);
  }

  // Multiply by scale
  result = arith::MulFOp::create(builder, loc, result, scale);
  return result;
}

// Convert a scalar or ranked tensor input with the given scale and zero point
// values.
//
// - input
//   Scalar or ranked tensor value.
//
// - inputShape
//   If 'input' is a tensor, combination or attributes/values representing its
//   static/dynamic dimensions. If 'input' is a scalar, empty list.
//
// - scale
//   Scale as a floating-point scalar value.
//
// - zeroPoint
//   Zero point as an integer scalar value.
//
// - quantizedType
//   Scalar quantized type of the result ('quant.qcast') or of the input
//   ('quant.dcast').
```
- **EN**: Implements logic around `create`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 447-474
```cpp
//
Value convertRanked(OpBuilder &builder, Location loc, Operation *op,
                    Value input, ArrayRef<OpFoldResult> inputShape, Value scale,
                    Value zeroPoint, QuantizedType quantizedType) {
  if (isa<QuantizeCastOp>(op))
    return quantizeValue(builder, loc, input, inputShape, scale, zeroPoint,
                         quantizedType);
  if (isa<DequantizeCastOp>(op))
    return dequantizeValue(builder, loc, input, inputShape, scale, zeroPoint,
                           quantizedType);
  llvm_unreachable("unexpected quant op");
}

// Convert an operation using per-layer quantization with a scalar or ranked
// tensor input.
//
// - op
//   'quant.dcast' or 'quant.qcast' op.
//
// - input
//   Scalar or ranked tensor.
//
// - quantizedType
//   Per-layer quantized type.
//
Value convertPerLayerRanked(OpBuilder &builder, Location loc, Operation *op,
                            Value input, UniformQuantizedType quantizedType) {
  // Create scale and zero point constants
```
- **EN**: Implements logic around `convertRanked`, `quantizeValue`, `dequantizeValue`, `convertPerLayerRanked`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertRanked`, `quantizeValue`, `dequantizeValue`, `convertPerLayerRanked` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 475-490
```cpp
  auto expressedType = quantizedType.getExpressedType();
  auto storageType = quantizedType.getStorageType();
  auto scaleAttr =
      builder.getFloatAttr(expressedType, quantizedType.getScale());
  auto scale =
      arith::ConstantOp::create(builder, loc, expressedType, scaleAttr);
  auto zeroPointAttr =
      builder.getIntegerAttr(storageType, quantizedType.getZeroPoint());
  auto zeroPoint =
      arith::ConstantOp::create(builder, loc, storageType, zeroPointAttr);

  auto inputShape = getScalarOrTensorShape(builder, loc, input);
  return convertRanked(builder, loc, op, input, inputShape, scale, zeroPoint,
                       quantizedType);
}

```
- **EN**: Implements logic around `getExpressedType`, `getStorageType`, `getFloatAttr`, `create`, and 3 more symbols.
- **CN**: 围绕 `getExpressedType`, `getStorageType`, `getFloatAttr`, `create`, and 3 more symbols 实现具体逻辑。

### Lines 491-509
```cpp
// Convert an operation using per-layer quantization.
//
// - op
//   'quant.dcast' or 'quant.qcast' op.
//
// - input
//   Scalar, ranked tensor, or unranked tensor.
//
// - quantizedType
//   Per-layer quantized type.
//
Value convertPerLayer(OpBuilder &builder, Location loc, Operation *op,
                      Value input, UniformQuantizedType quantizedType) {
  // Flatten input if unranked
  bool isUnranked = isa<UnrankedTensorType>(input.getType());
  Value inputShape;
  if (isUnranked)
    std::tie(input, inputShape) = flattenUnrankedTensor(builder, loc, input);

```
- **EN**: Implements logic around `convertPerLayer`, `getType`, `tie`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertPerLayer`, `getType`, `tie` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 510-537
```cpp
  // Process ranked tensor
  auto result = convertPerLayerRanked(builder, loc, op, input, quantizedType);

  // Restore original shape if unranked
  if (isUnranked)
    result = restoreUnrankedTensorShape(builder, loc, result, inputShape);

  return result;
}

// Convert an operation using per-channel quantization and a scalar or ranked
// tensor as an input.
//
// - op
//   'quant.dcast' or 'quant.qcast' op.
//
// - input
//   Scalar or ranked tensor.
//
// - quantizedType
//   Per-channel quantized type.
//
Value convertPerChannelRanked(OpBuilder &builder, Location loc, Operation *op,
                              Value input,
                              UniformQuantizedPerAxisType quantizedType,
                              int64_t channelAxis) {
  auto *context = builder.getContext();

```
- **EN**: Implements logic around `convertPerLayerRanked`, `restoreUnrankedTensorShape`, `convertPerChannelRanked`, `getContext`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertPerLayerRanked`, `restoreUnrankedTensorShape`, `convertPerChannelRanked`, `getContext` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 538-565
```cpp
  auto inputType = cast<RankedTensorType>(input.getType());
  auto inputRank = inputType.getRank();

  auto scales = materializePerChannelScales(builder, loc, quantizedType);
  auto zeroPoints =
      materializePerChannelZeroPoints(builder, loc, quantizedType);

  auto elementType = isa<FloatType>(inputType.getElementType())
                         ? quantizedType.getStorageType()
                         : quantizedType.getExpressedType();
  auto initShape = tensor::getMixedSizes(builder, loc, input);
  Value init = tensor::EmptyOp::create(builder, loc, initShape, elementType);

  SmallVector<utils::IteratorType> iteratorTypes(inputRank,
                                                 utils::IteratorType::parallel);
  auto channelAxisAffineMap = AffineMap::get(
      inputRank, 0, builder.getAffineDimExpr(channelAxis), context);
  SmallVector<AffineMap> indexingMaps{
      builder.getMultiDimIdentityMap(inputRank), channelAxisAffineMap,
      channelAxisAffineMap, builder.getMultiDimIdentityMap(inputRank)};
  auto result = linalg::GenericOp::create(
                    builder, loc,
                    init.getType(),                        // resultType
                    ValueRange{input, scales, zeroPoints}, // inputs
                    ValueRange{init},                      // outputs
                    indexingMaps, iteratorTypes,
                    [&](OpBuilder &builder, Location loc, ValueRange args) {
                      assert(args.size() == 4);
```
- **EN**: Implements logic around `getType`, `getRank`, `materializePerChannelScales`, `materializePerChannelZeroPoints`, and 10 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `getRank`, `materializePerChannelScales`, `materializePerChannelZeroPoints`, and 10 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 566-580
```cpp
                      auto input = args[0];
                      auto scale = args[1];
                      auto zeroPoint = args[2];

                      auto result =
                          convertRanked(builder, loc, op, input, {}, scale,
                                        zeroPoint, quantizedType);

                      linalg::YieldOp::create(builder, loc, result);
                    })
                    .getResult(0);

  return result;
}

```
- **EN**: Implements logic around `convertRanked`, `create`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertRanked`, `create`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 581-605
```cpp
// Convert an operation using per-channel quantization.
//
// - op
//   'quant.dcast' or 'quant.qcast' op.
//
// - input
//   Scalar, ranked tensor, or unranked tensor.
//
// - quantizedType
//   Per-channel quantized type.
//
Value convertPerChannel(OpBuilder &builder, Location loc, Operation *op,
                        Value input,
                        UniformQuantizedPerAxisType quantizedType) {
  // Flatten unranked tensor into a 3D ranked tensor if necessary
  bool isUnranked = isa<UnrankedTensorType>(input.getType());
  int64_t channelAxis = quantizedType.getQuantizedDimension();
  int64_t channelAxisSize = (int64_t)quantizedType.getScales().size();
  Value inputShape;
  if (isUnranked) {
    std::tie(input, inputShape) = flattenUnrankedTensorAroundAxis(
        builder, loc, input, channelAxis, channelAxisSize);
    channelAxis = 1;
  }

```
- **EN**: Implements logic around `convertPerChannel`, `getType`, `getQuantizedDimension`, `getScales`, and 1 more symbols; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertPerChannel`, `getType`, `getQuantizedDimension`, `getScales`, and 1 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 606-632
```cpp
  // Work on a ranked tensor
  auto result = convertPerChannelRanked(builder, loc, op, input, quantizedType,
                                        channelAxis);

  // Restore original tensor shape if unranked
  if (isUnranked)
    result = restoreUnrankedTensorShape(builder, loc, result, inputShape);

  return result;
}

// Convert an operation using sub-channel quantization.
//
// - op
//   'quant.dcast' or 'quant.qcast' op.
//
// - input
//   Scalar, ranked tensor.
//
// - quantizedType
//   Sub-channel quantized type.
//
Value convertSubChannel(OpBuilder &builder, Location loc, Operation *op,
                        Value input,
                        UniformQuantizedSubChannelType quantizedType) {
  auto *context = builder.getContext();

```
- **EN**: Implements logic around `convertPerChannelRanked`, `restoreUnrankedTensorShape`, `convertSubChannel`, `getContext`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertPerChannelRanked`, `restoreUnrankedTensorShape`, `convertSubChannel`, `getContext` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 633-660
```cpp
  auto inputType = cast<RankedTensorType>(input.getType());
  auto inputRank = inputType.getRank();

  auto scales = materializeSubChannelScales(builder, loc, quantizedType);
  auto zeroPoints =
      materializeSubChannelZeroPoints(builder, loc, quantizedType);

  auto elementType = isa<FloatType>(inputType.getElementType())
                         ? quantizedType.getStorageType()
                         : quantizedType.getExpressedType();
  auto initShape = tensor::getMixedSizes(builder, loc, input);
  Value init = tensor::EmptyOp::create(builder, loc, initShape, elementType);

  SmallVector<utils::IteratorType> iteratorTypes(inputRank,
                                                 utils::IteratorType::parallel);
  const SmallVector<std::pair<int32_t, int64_t>> &blockSizeInfo =
      quantizedType.getBlockSizeInfo();
  SmallVector<AffineExpr> affineExprs(inputRank,
                                      builder.getAffineConstantExpr(0));
  for (auto [quantizedDimension, blockSize] : blockSizeInfo) {
    affineExprs[quantizedDimension] =
        builder.getAffineDimExpr(quantizedDimension).floorDiv(blockSize);
  }
  auto affineMap = AffineMap::get(inputRank, 0, affineExprs, context);
  SmallVector<AffineMap> indexingMaps{
      builder.getMultiDimIdentityMap(inputRank), affineMap, affineMap,
      builder.getMultiDimIdentityMap(inputRank)};
  auto result = linalg::GenericOp::create(
```
- **EN**: Implements logic around `getType`, `getRank`, `materializeSubChannelScales`, `materializeSubChannelZeroPoints`, and 12 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `getRank`, `materializeSubChannelScales`, `materializeSubChannelZeroPoints`, and 12 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 661-675
```cpp
                    builder, loc,
                    init.getType(),                        // resultType
                    ValueRange{input, scales, zeroPoints}, // inputs
                    ValueRange{init},                      // outputs
                    indexingMaps, iteratorTypes,
                    [&](OpBuilder &builder, Location loc, ValueRange args) {
                      assert(args.size() == 4);
                      auto input = args[0];
                      auto scale = args[1];
                      auto zeroPoint = args[2];

                      auto result =
                          convertRanked(builder, loc, op, input, {}, scale,
                                        zeroPoint, quantizedType);

```
- **EN**: Implements logic around `getType`, `assert`, `convertRanked`.
- **CN**: 围绕 `getType`, `assert`, `convertRanked` 实现具体逻辑。

### Lines 676-700
```cpp
                      linalg::YieldOp::create(builder, loc, result);
                    })
                    .getResult(0);

  return result;
}

// Convert a quantization operation.
//
// - op
//   'quant.dcast' or 'quant.qcast' op.
//
// - input
//   Scalar, ranked tensor, or unranked tensor. The element type matches
//   the storage type (quant.dcast) or expressed type (quant.qcast) of
//   'quantizedType'.
//
// - quantizedType
//   Per-layer or per-channel quantized type.
//
Value convertQuantized(OpBuilder &builder, Location loc, Operation *op,
                       Value input, Type quantizedType) {
  if (auto uniformQuantizedType = dyn_cast<UniformQuantizedType>(quantizedType))
    return convertPerLayer(builder, loc, op, input, uniformQuantizedType);

```
- **EN**: Implements logic around `create`, `getResult`, `convertQuantized`, `convertPerLayer`; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResult`, `convertQuantized`, `convertPerLayer` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 701-715
```cpp
  if (auto uniformQuantizedPerAxisType =
          dyn_cast<UniformQuantizedPerAxisType>(quantizedType))
    return convertPerChannel(builder, loc, op, input,
                             uniformQuantizedPerAxisType);

  if (auto uniformQuantizedSubChannelType =
          dyn_cast<UniformQuantizedSubChannelType>(quantizedType))
    return convertSubChannel(builder, loc, op, input,
                             uniformQuantizedSubChannelType);

  llvm_unreachable("unexpected quantized type");
}

// Lowering pattern for 'quant.dcast'
struct DequantizeCastOpConversion
```
- **EN**: Introduces declarations for `DequantizeCastOpConversion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DequantizeCastOpConversion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 716-732
```cpp
    : public OpConversionPattern<quant::DequantizeCastOp> {
  using OpConversionPattern<quant::DequantizeCastOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(quant::DequantizeCastOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto loc = op.getLoc();
    auto input = op.getInput();
    auto quantizedType =
        cast<QuantizedType>(getScalarType(op.getInput().getType()));

    // Convert quantized input to storage type
    auto storageScalarOrTensorType =
        getScalarOrTensorType(quantizedType.getStorageType(), input.getType());
    input = quant::StorageCastOp::create(rewriter, loc,
                                         storageScalarOrTensorType, input);

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getInput`, `getScalarType`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getInput`, `getScalarType`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策。

### Lines 733-751
```cpp
    auto result = convertQuantized(rewriter, loc, op, input, quantizedType);

    rewriter.replaceOp(op, result);
    return success();
  }
};

// Lowering pattern for 'quant.qcast'
struct QuantizeCastOpConversion
    : public OpConversionPattern<quant::QuantizeCastOp> {
  using OpConversionPattern<quant::QuantizeCastOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(quant::QuantizeCastOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto loc = op.getLoc();
    auto input = op.getInput();
    auto quantizedType = getScalarType(op.getResult().getType());

```
- **EN**: Introduces declarations for `QuantizeCastOpConversion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `QuantizeCastOpConversion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 752-766
```cpp
    // Flatten unranked tensor input
    auto result = convertQuantized(rewriter, loc, op, input, quantizedType);

    // Cast stored value to result quantized value
    rewriter.replaceOpWithNewOp<quant::StorageCastOp>(
        op, op.getResult().getType(), result);
    return success();
  }
};

struct LowerQuantOps : public impl::LowerQuantOpsBase<LowerQuantOps> {
  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());
    populateLowerQuantOpsPatterns(patterns);

```
- **EN**: Introduces declarations for `LowerQuantOps`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LowerQuantOps` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 767-780
```cpp
    ConversionTarget target(getContext());
    target.addLegalOp<quant::StorageCastOp>();
    target.addIllegalDialect<quant::QuantDialect>();
    target.addLegalDialect<arith::ArithDialect, linalg::LinalgDialect,
                           shape::ShapeDialect, tensor::TensorDialect>();

    if (failed(applyPartialConversion(getOperation(), target,
                                      std::move(patterns))))
      signalPassFailure();
  }
};

} // namespace

```
- **EN**: Implements logic around `target`, `StorageCastOp>`, `QuantDialect>`, `TensorDialect>`, and 3 more symbols; this block coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `target`, `StorageCastOp>`, `QuantDialect>`, `TensorDialect>`, and 3 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 781-787
```cpp
void populateLowerQuantOpsPatterns(RewritePatternSet &patterns) {
  patterns.add<DequantizeCastOpConversion, QuantizeCastOpConversion>(
      patterns.getContext());
}

} // namespace quant
} // namespace mlir
```
- **EN**: Introduces declarations for `quant`, `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `quant`, `mlir` 等声明，建立后续使用的方言级类型或模式记录。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`, `mlir/Dialect/Quant/Transforms/Passes.h`, `mlir/Dialect/Shape/IR/Shape.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/IR/Matchers.h`, `mlir/IR/PatternMatch.h` ... (+2 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (9), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
