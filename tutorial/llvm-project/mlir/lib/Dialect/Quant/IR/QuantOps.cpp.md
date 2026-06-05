# QuantOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Quant/IR/QuantOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Quant dialect support for dialect IR definitions such as operations, attributes, types, and parsers/printers, centered on `QuantOps`.
  - **CN**: 实现 Quant 方言中围绕 `QuantOps` 的方言 IR 定义，如操作、属性、类型以及解析/打印逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- QuantOps.cpp - Quantization Type and Ops Implementation --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "QuantDialectBytecode.h"
#include "TypeDetail.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `QuantDialectBytecode.h`, `TypeDetail.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `QuantDialectBytecode.h`, `TypeDetail.h`。

### Lines 12-21
```cpp
#include "mlir/Dialect/Quant/IR/Quant.h"
#include "mlir/Dialect/Quant/IR/QuantTypes.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/TypeUtilities.h"
#include "mlir/Transforms/InliningUtils.h"

#include "mlir/Dialect/Quant/IR/QuantOpsDialect.cpp.inc"

namespace mlir {
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/PatternMatch.h`。

### Lines 22-41
```cpp
namespace quant {

namespace {

// Verify the integrity of per-axis quantization information, if present.
//
// - uniformQuantizedPerAxisType
//   A quantized type with per-axis quantization.
//
// - containerType
//   Original input or result type of the operation using the provided quantized
//   type. Used to ensure that the quantized type appears within a tensor and
//   that the tensor is compatible with per-axis quantization information.
//
LogicalResult verifyPerAxisQuantization(
    Operation *op, UniformQuantizedPerAxisType uniformQuantizedPerAxisType,
    Type containerType) {
  auto tensorType = dyn_cast<TensorType>(containerType);
  if (!tensorType)
    return op->emitError("scalar types may not use per-axis quantization");
```
- **EN**: Introduces declarations for `quant`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `quant` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 42-57
```cpp

  if (!tensorType.hasRank())
    return success();

  int32_t quantizedDimension =
      uniformQuantizedPerAxisType.getQuantizedDimension();
  if ((int64_t)quantizedDimension >= tensorType.getRank())
    return op->emitError("quantized dimension must be less than tensor rank");

  int64_t quantizedDimensionSize = tensorType.getDimSize(quantizedDimension);
  if (quantizedDimensionSize != ShapedType::kDynamic &&
      quantizedDimensionSize !=
          (int64_t)uniformQuantizedPerAxisType.getScales().size())
    return op->emitError(
        "quantized dimension size does not match number of scales");

```
- **EN**: Implements logic around `hasRank`, `success`, `getQuantizedDimension`, `getRank`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasRank`, `success`, `getQuantizedDimension`, `getRank`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 58-77
```cpp
  return success();
}

// Verifies that the sub-channel quantization parameters are consistent with
// the given container type. The function checks the following:
//
// - The container type must be a ranked tensor type.
// - Each quantized dimension must be less than the rank of the tensor.
// - The size of each dimension at the quantized dimension must be divisible
//    by the corresponding block size.
// - The scale dimension size at each axis index should match the tensor
//    dimension at the index divided by the corresponding block size.
//
// The `uniformQuantizedSubChannelType` argument provides the sub-channel
// quantization parameters, and the `containerType` argument specifies the
// type of the container holding the quantized data.
//
LogicalResult verifySubChannelQuantization(
    Operation *op,
    UniformQuantizedSubChannelType uniformQuantizedSubChannelType,
```
- **EN**: Implements logic around `success`, `verifySubChannelQuantization`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `success`, `verifySubChannelQuantization` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 78-90
```cpp
    Type containerType) {
  auto tensorType = dyn_cast<TensorType>(containerType);
  if (!tensorType)
    return op->emitError("scalar types may not use sub-channel quantization");

  if (!tensorType.hasRank())
    return op->emitError(
        "tensor containing the sub-channel quantized type must be ranked");

  const SmallVector<std::pair<int32_t, int64_t>> &blockSizeInfo =
      uniformQuantizedSubChannelType.getBlockSizeInfo();
  auto shape = tensorType.getShape();

```
- **EN**: Implements logic around `emitError`, `hasRank`, `getBlockSizeInfo`, `getShape`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `emitError`, `hasRank`, `getBlockSizeInfo`, `getShape` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 91-110
```cpp
  // The dimension size of scale for an axis which is not specified as quantized
  // dimension should be 1.
  SmallVector<int64_t> expectedScaleShape(tensorType.getShape().size(), 1);
  for (auto [quantizedDimension, blockSize] : blockSizeInfo) {
    if (quantizedDimension >= tensorType.getRank())
      return op->emitError()
             << "quantized dimension " << quantizedDimension
             << " must be less than tensor rank " << tensorType.getRank();
    if (!tensorType.isDynamicDim(quantizedDimension) &&
        tensorType.getDimSize(quantizedDimension) % blockSize != 0)
      return op->emitError()
             << "tensor dimension size "
             << tensorType.getDimSize(quantizedDimension) << " at axis "
             << quantizedDimension
             << " must be divisible by the corresponding block size "
             << blockSize;
    if (tensorType.isDynamicDim(quantizedDimension))
      expectedScaleShape[quantizedDimension] = ShapedType::kDynamic;
    else
      expectedScaleShape[quantizedDimension] =
```
- **EN**: Implements logic around `expectedScaleShape`, `getRank`, `emitError`, `isDynamicDim`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `expectedScaleShape`, `getRank`, `emitError`, `isDynamicDim`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 111-130
```cpp
          tensorType.getDimSize(quantizedDimension) / blockSize;
  }

  // Block sizes must be greater than 0 and divide the corresponding dimension
  // size. While a block size b must be less than or equal to the corresponding
  // dimension size d, this constraint is implicitly enforced by requiring that
  // d % b == 0 when d != 0.
  //
  // However, a problem arises when d = 0.  The divisibility constraint allows b
  // to be any value, potentially violating the requirement that b <= d.
  // Furthermore, if b is unspecified (implicitly equal to d), it violates the
  // constraint that b > 0.
  //
  // Therefore, we explicitly disallow the case where d = 0 to maintain
  // consistency and avoid these issues.
  if (llvm::is_contained(tensorType.getShape(), 0)) {
    return op->emitError() << "tensor dimension size of zero is not allowed "
                              "with sub-channel quantization";
  }

```
- **EN**: Implements logic around `getDimSize`, `is_contained`, `emitError`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDimSize`, `is_contained`, `emitError` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 131-148
```cpp
  auto scaleShape =
      uniformQuantizedSubChannelType.getScales().getType().getShape();
  if (scaleShape.size() != shape.size()) {
    return op->emitError() << "Rank of scales " << scaleShape.size()
                           << " must match "
                           << "the rank of the tensor " << shape.size();
  }

  for (auto [index, scaleDim] : llvm::enumerate(expectedScaleShape)) {
    if (expectedScaleShape[index] != ShapedType::kDynamic &&
        expectedScaleShape[index] != scaleShape[index])
      return op->emitError() << "dimension size " << scaleDim
                             << " of scales tensor at axis " << index
                             << " should match (tensor dimension at axis / "
                                "block sizes at axis) = "
                             << expectedScaleShape[index];
  }

```
- **EN**: Implements logic around `getScales`, `size`, `emitError`, `enumerate`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getScales`, `size`, `emitError`, `enumerate`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 149-168
```cpp
  return success();
}

// Common verification logic for 'quant.dcast' and 'quant.qcast' ops.
//
// - quantizedType
//   Quantized type used in the input ('quant.dcast') or result ('quant.qcast'),
//   whether as a primitive type or in a tensor.
//
// - floatType
//   Float type used in the input ('quant.qcast') or result ('quant.dcast'),
//   whether as a primitive type or in a tensor.
//
// - containerType
//   Type of original input or result.
//
LogicalResult verifyQuantizationOp(Operation *op, QuantizedType quantizedType,
                                   FloatType floatType, Type containerType) {
  if (quantizedType.getExpressedType() != floatType)
    return op->emitError(
```
- **EN**: Implements logic around `success`, `verifyQuantizationOp`, `getExpressedType`, `emitError`; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `success`, `verifyQuantizationOp`, `getExpressedType`, `emitError` 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 169-182
```cpp
        "expressed type in quantized type expected to match float type");

  // Verify integrity of per-axis quantization information, if present.
  if (auto quantizedPerAxisType =
          dyn_cast<UniformQuantizedPerAxisType>(quantizedType)) {
    return verifyPerAxisQuantization(op, quantizedPerAxisType, containerType);
  }

  if (auto quantizedSubChannelType =
          dyn_cast<UniformQuantizedSubChannelType>(quantizedType)) {
    return verifySubChannelQuantization(op, quantizedSubChannelType,
                                        containerType);
  }

```
- **EN**: Implements logic around `verifyPerAxisQuantization`, `verifySubChannelQuantization`; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `verifyPerAxisQuantization`, `verifySubChannelQuantization` 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性。

### Lines 183-194
```cpp
  // At this point the type is UniformQuantizedType
  return success();
}

struct QuantInlinerInterface : public DialectInlinerInterface {
  using DialectInlinerInterface::DialectInlinerInterface;
  /// All quant dialect ops can be inlined.
  bool isLegalToInline(Operation *, Region *, bool, IRMapping &) const final {
    return true;
  }
};

```
- **EN**: Introduces declarations for `QuantInlinerInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `QuantInlinerInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 195-211
```cpp
} // namespace

//===----------------------------------------------------------------------===//
// Dialect
//===----------------------------------------------------------------------===//

void QuantDialect::initialize() {
  addTypes<AnyQuantizedType, CalibratedQuantizedType, UniformQuantizedType,
           UniformQuantizedPerAxisType, UniformQuantizedSubChannelType>();
  addOperations<
#define GET_OP_LIST
#include "mlir/Dialect/Quant/IR/QuantOps.cpp.inc"
      >();
  detail::addBytecodeInterface(this);
  addInterfaces<QuantInlinerInterface>();
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Quant/IR/QuantOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Quant/IR/QuantOps.cpp.inc`。

### Lines 212-231
```cpp
//===----------------------------------------------------------------------===//
// DequantizeCastOp
//===----------------------------------------------------------------------===//

LogicalResult DequantizeCastOp::verify() {
  return verifyQuantizationOp(*this, getQuantizedType(), getFloatType(),
                              getInput().getType());
}

OpFoldResult DequantizeCastOp::fold(FoldAdaptor adaptor) {
  // Matches x -> quant.qcast -> quant.dcast -> y, replacing the quant.dcast op
  // with the value of x. Values x and y are guaranteed to be of the same type
  // in this pattern.
  auto srcQcastOp = getInput().getDefiningOp<QuantizeCastOp>();
  if (!srcQcastOp)
    return {};
  assert(srcQcastOp.getInput().getType() == getType());
  return srcQcastOp.getInput();
}

```
- **EN**: Implements logic around `verify`, `verifyQuantizationOp`, `getInput`, `fold`, and 1 more symbols; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `verify`, `verifyQuantizationOp`, `getInput`, `fold`, and 1 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 232-242
```cpp
FloatType DequantizeCastOp::getFloatType() {
  return cast<FloatType>(getElementTypeOrSelf(getResult().getType()));
}

QuantizedType DequantizeCastOp::getQuantizedType() {
  return cast<QuantizedType>(getElementTypeOrSelf(getInput().getType()));
}

//===----------------------------------------------------------------------===//
// QuantizeCastOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getFloatType`, `getElementTypeOrSelf`, `getQuantizedType`.
- **CN**: 围绕 `getFloatType`, `getElementTypeOrSelf`, `getQuantizedType` 实现具体逻辑。

### Lines 243-260
```cpp

LogicalResult QuantizeCastOp::verify() {
  return verifyQuantizationOp(*this, getQuantizedType(), getFloatType(),
                              getInput().getType());
}

OpFoldResult QuantizeCastOp::fold(FoldAdaptor adaptor) {
  // Matches x -> quant.dcast -> quant.qcast -> y, replacing the quant.qcast op
  // with the value of x if the casts invert each other. Contrary to the folding
  // pattern in quant.dcast (i.e., x -> quant.qcast -> quant.dcast -> y), values
  // x and y are not guaranteed to be of the same type here, as they may use
  // different quantization parameters.
  auto srcDcastOp = getInput().getDefiningOp<DequantizeCastOp>();
  if (!srcDcastOp || srcDcastOp.getInput().getType() != getType())
    return {};
  return srcDcastOp.getInput();
}

```
- **EN**: Implements logic around `verify`, `verifyQuantizationOp`, `getInput`, `fold`; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `verify`, `verifyQuantizationOp`, `getInput`, `fold` 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 261-271
```cpp
FloatType QuantizeCastOp::getFloatType() {
  return cast<FloatType>(getElementTypeOrSelf(getInput().getType()));
}

QuantizedType QuantizeCastOp::getQuantizedType() {
  return cast<QuantizedType>(getElementTypeOrSelf(getResult().getType()));
}

//===----------------------------------------------------------------------===//
// StorageCastOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getFloatType`, `getElementTypeOrSelf`, `getQuantizedType`.
- **CN**: 围绕 `getFloatType`, `getElementTypeOrSelf`, `getQuantizedType` 实现具体逻辑。

### Lines 272-288
```cpp

LogicalResult StorageCastOp::verify() {
  auto quantizedType = getQuantizedType();
  auto integerType = getIntegerType();
  if (quantizedType.getStorageType() != integerType)
    return emitError(
        "storage type in quantized type expected to match integer type");

  // Verify integrity of per-axis quantization information, if available. While
  // the quantization type may appear in the input or the result, their tensor
  // shapes are guaranteed to be identical at this point.
  if (auto quantizedPerAxisType =
          dyn_cast<UniformQuantizedPerAxisType>(quantizedType)) {
    return verifyPerAxisQuantization(*this, quantizedPerAxisType,
                                     getInput().getType());
  }

```
- **EN**: Implements logic around `verify`, `getQuantizedType`, `getIntegerType`, `getStorageType`, and 3 more symbols; this block checks operation invariants or verification rules; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `verify`, `getQuantizedType`, `getIntegerType`, `getStorageType`, and 3 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 289-298
```cpp
  if (auto quantizedSunChannelType =
          dyn_cast<UniformQuantizedSubChannelType>(quantizedType)) {
    return verifySubChannelQuantization(*this, quantizedSunChannelType,
                                        getInput().getType());
  }

  // At this point the type is UniformQuantizedType
  return success();
}

```
- **EN**: Implements logic around `verifySubChannelQuantization`, `getInput`, `success`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `verifySubChannelQuantization`, `getInput`, `success` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 299-312
```cpp
OpFoldResult StorageCastOp::fold(FoldAdaptor adaptor) {
  // Matches x -> quant.scast -> quant.scast -> y, replacing the second
  // quant.scast with the value of x if the casts invert each other.
  auto srcScastOp = getInput().getDefiningOp<StorageCastOp>();
  if (!srcScastOp || srcScastOp.getInput().getType() != getType())
    return {};
  return srcScastOp.getInput();
}

IntegerType StorageCastOp::getIntegerType() {
  auto inputScalarType = getElementTypeOrSelf(getInput().getType());
  if (auto integerType = dyn_cast<IntegerType>(inputScalarType))
    return integerType;

```
- **EN**: Implements logic around `fold`, `getInput`, `getIntegerType`, `getElementTypeOrSelf`; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `getInput`, `getIntegerType`, `getElementTypeOrSelf` 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 313-325
```cpp
  auto resultScalarType = getElementTypeOrSelf(getResult().getType());
  return cast<IntegerType>(resultScalarType);
}

QuantizedType StorageCastOp::getQuantizedType() {
  auto inputScalarType = getElementTypeOrSelf(getInput().getType());
  if (auto quantizedType = dyn_cast<QuantizedType>(inputScalarType))
    return quantizedType;

  auto resultScalarType = getElementTypeOrSelf(getResult().getType());
  return cast<QuantizedType>(resultScalarType);
}

```
- **EN**: Implements logic around `getElementTypeOrSelf`, `getQuantizedType`.
- **CN**: 围绕 `getElementTypeOrSelf`, `getQuantizedType` 实现具体逻辑。

### Lines 326-330
```cpp
} // namespace quant
} // namespace mlir

#define GET_OP_CLASSES
#include "mlir/Dialect/Quant/IR/QuantOps.cpp.inc"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Quant/IR/QuantOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Quant/IR/QuantOps.cpp.inc`。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Operation verification / 操作验证**:
  - **EN**: Checks structural and semantic invariants so malformed IR is rejected early.
  - **CN**: 检查结构与语义不变式，以便尽早拒绝非法 IR。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `QuantDialectBytecode.h`, `TypeDetail.h`, `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/PatternMatch.h`, `mlir/IR/TypeUtilities.h`, `mlir/Transforms/InliningUtils.h`, `mlir/Dialect/Quant/IR/QuantOpsDialect.cpp.inc`, `mlir/Dialect/Quant/IR/QuantOps.cpp.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`, `GET_OP_CLASSES`
