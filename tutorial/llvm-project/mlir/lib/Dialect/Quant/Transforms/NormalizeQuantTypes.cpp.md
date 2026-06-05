# NormalizeQuantTypes.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Quant/Transforms/NormalizeQuantTypes.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Quant dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `NormalizeQuantTypes`.
  - **CN**: 实现 Quant 方言中围绕 `NormalizeQuantTypes` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```cpp
//===- NormalizeQuantTypes.cpp - Normalize quantized types
//----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 9-21
```cpp
//
// Normalize generic quantized types to specific quantized types
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Func/Transforms/FuncConversions.h"
#include "mlir/Dialect/Quant/IR/Quant.h"
#include "mlir/Dialect/Quant/IR/QuantTypes.h"
#include "mlir/Dialect/Quant/Transforms/Passes.h"
#include "mlir/Transforms/DialectConversion.h"
#include "llvm/ADT/SmallVectorExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`。

### Lines 22-28
```cpp
namespace mlir {
namespace quant {

#define GEN_PASS_DEF_NORMALIZEQUANTTYPES
#include "mlir/Dialect/Quant/Transforms/Passes.h.inc"

namespace {
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Quant/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Quant/Transforms/Passes.h.inc`。

### Lines 29-35
```cpp

/// Returns true if the given sub-channel quantized type is convertible to a
/// per-tensor quantized type. This is true if the sub-channel type has only
/// one scale and one zero point.
///
/// Assumes that `tensorType` is a tensor with element type
/// `quant::UniformQuantizedSubChannelType`.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-42
```cpp
static bool isConvertibleToPerTensor(TensorType tensorType) {
  return cast<UniformQuantizedSubChannelType>(tensorType.getElementType())
             .getScales()
             .getType()
             .getNumElements() == 1;
}

```
- **EN**: Implements logic around `isConvertibleToPerTensor`, `getElementType`, `getScales`, `getType`, and 1 more symbols.
- **CN**: 围绕 `isConvertibleToPerTensor`, `getElementType`, `getScales`, `getType`, and 1 more symbols 实现具体逻辑。

### Lines 43-56
```cpp
/// Returns true if the given sub-channel quantized type is convertible to a
/// per-axis quantized type. This is true if the shape of the scales tensor has
/// all but one non-one value.
///
/// Assumes that `tensorType` is a tensor with element type
/// `quant::UniformQuantizedSubChannelType`.
static bool isConvertibleToPerAxis(TensorType tensorType) {
  auto shape = cast<UniformQuantizedSubChannelType>(tensorType.getElementType())
                   .getScales()
                   .getType()
                   .getShape();
  return llvm::count_if(shape, [](int64_t dim) { return dim != 1; }) == 1;
}

```
- **EN**: Implements logic around `isConvertibleToPerAxis`, `getElementType`, `getScales`, `getType`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isConvertibleToPerAxis`, `getElementType`, `getScales`, `getType`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 57-66
```cpp
/// This class defines a type converter that converts sub-channel quantized
/// types to per-tensor or per-axis quantized types whenever possible.
class NormalizedQuantTypesConverter : public TypeConverter {

  static Type convertType(Type type) {
    auto tensorType = dyn_cast<TensorType>(type);
    if (!tensorType) {
      return type;
    }

```
- **EN**: Introduces declarations for `defines`, `NormalizedQuantTypesConverter`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `defines`, `NormalizedQuantTypesConverter` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 67-80
```cpp
    auto subChannelType =
        dyn_cast<UniformQuantizedSubChannelType>(tensorType.getElementType());
    if (!subChannelType) {
      return type;
    }

    if (isConvertibleToPerTensor(tensorType)) {
      double scale =
          subChannelType.getScales().getValues<APFloat>()[0].convertToDouble();
      int64_t zeroPoint =
          subChannelType.getZeroPoints().getValues<APInt>()[0].getSExtValue();
      auto perTensorType = UniformQuantizedType::get(
          subChannelType.getFlags(), subChannelType.getStorageType(),
          subChannelType.getExpressedType(), scale, zeroPoint,
```
- **EN**: Implements logic around `getElementType`, `isConvertibleToPerTensor`, `getScales`, `getZeroPoints`, and 3 more symbols.
- **CN**: 围绕 `getElementType`, `isConvertibleToPerTensor`, `getScales`, `getZeroPoints`, and 3 more symbols 实现具体逻辑。

### Lines 81-94
```cpp
          subChannelType.getStorageTypeMin(),
          subChannelType.getStorageTypeMax());
      return tensorType.clone(perTensorType);
    }

    if (isConvertibleToPerAxis(tensorType)) {
      auto shape = subChannelType.getScales().getType().getShape();
      const auto *quantizedDimItr =
          llvm::find_if(shape, [](int64_t dim) { return dim != 1; });
      auto scales = llvm::map_to_vector(
          subChannelType.getScales().getValues<APFloat>(),
          [](const APFloat &scale) { return scale.convertToDouble(); });
      auto zeroPoints = llvm::map_to_vector(
          subChannelType.getZeroPoints().getValues<APInt>(),
```
- **EN**: Implements logic around `getStorageTypeMin`, `getStorageTypeMax`, `clone`, `isConvertibleToPerAxis`, and 5 more symbols.
- **CN**: 围绕 `getStorageTypeMin`, `getStorageTypeMax`, `clone`, `isConvertibleToPerAxis`, and 5 more symbols 实现具体逻辑。

### Lines 95-105
```cpp
          [](const APInt &zeroPoint) { return zeroPoint.getSExtValue(); });
      auto perAxisType = UniformQuantizedPerAxisType::get(
          subChannelType.getFlags(), subChannelType.getStorageType(),
          subChannelType.getExpressedType(), scales, zeroPoints,
          quantizedDimItr - shape.begin(), subChannelType.getStorageTypeMin(),
          subChannelType.getStorageTypeMax());
      return tensorType.clone(perAxisType);
    }
    return type;
  }

```
- **EN**: Implements logic around `getSExtValue`, `get`, `getFlags`, `getExpressedType`, and 3 more symbols.
- **CN**: 围绕 `getSExtValue`, `get`, `getFlags`, `getExpressedType`, and 3 more symbols 实现具体逻辑。

### Lines 106-112
```cpp
public:
  explicit NormalizedQuantTypesConverter() { addConversion(convertType); }
};

/// This class implements a conversion pattern that converts any generic
/// operation with sub-channel quantized types to an equivalent operation with
/// per-tensor or per-axis quantized types.
```
- **EN**: Introduces declarations for `implements`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `implements` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 113-125
```cpp
class ConvertGenericOpwithSubChannelType : public ConversionPattern {
public:
  ConvertGenericOpwithSubChannelType(TypeConverter &typeConverter,
                                     MLIRContext *context)
      : ConversionPattern(typeConverter, MatchAnyOpTypeTag{}, 0, context) {}

  LogicalResult
  matchAndRewrite(Operation *op, ArrayRef<Value> operands,
                  ConversionPatternRewriter &rewriter) const final {
    SmallVector<Type> resultTypes;
    if (failed(typeConverter->convertTypes(op->getResultTypes(), resultTypes)))
      return failure();

```
- **EN**: Introduces declarations for `ConvertGenericOpwithSubChannelType`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertGenericOpwithSubChannelType` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 126-139
```cpp
    auto *newOp = Operation::create(
        op->getLoc(), op->getName(), resultTypes, operands, op->getAttrs(),
        op->getPropertiesStorage(), op->getSuccessors(), op->getNumRegions());
    for (auto regions : llvm::zip(op->getRegions(), newOp->getRegions())) {
      Region &before = std::get<0>(regions);
      Region &parent = std::get<1>(regions);
      rewriter.inlineRegionBefore(before, parent, parent.end());
      if (failed(rewriter.convertRegionTypes(&parent, *typeConverter)))
        return failure();
    }
    rewriter.insert(newOp);
    rewriter.replaceOp(op, newOp->getResults());
    return success();
  }
```
- **EN**: Implements logic around `create`, `getLoc`, `getPropertiesStorage`, `zip`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `create`, `getLoc`, `getPropertiesStorage`, `zip`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策。

### Lines 140-147
```cpp
};

// Conversion pass
class NormalizeQuantTypes
    : public impl::NormalizeQuantTypesBase<NormalizeQuantTypes> {
public:
  void runOnOperation() override {

```
- **EN**: Introduces declarations for `NormalizeQuantTypes`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `NormalizeQuantTypes` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 148-161
```cpp
    auto *context = &getContext();

    NormalizedQuantTypesConverter typeConverter;
    ConversionTarget target(*context);

    // Determine legal operations.
    target.addDynamicallyLegalOp<func::FuncOp>([&](func::FuncOp op) {
      return typeConverter.isSignatureLegal(op.getFunctionType()) &&
             typeConverter.isLegal(&op.getBody());
    });
    target.markUnknownOpDynamicallyLegal([&](Operation *op) {
      return typeConverter.isLegal(op->getOperandTypes()) &&
             typeConverter.isLegal(op->getResultTypes());
    });
```
- **EN**: Implements logic around `getContext`, `target`, `FuncOp>`, `isSignatureLegal`, and 2 more symbols; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `getContext`, `target`, `FuncOp>`, `isSignatureLegal`, and 2 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 162-168
```cpp

    // Register conversion patterns
    RewritePatternSet patterns(context);
    populateFunctionOpInterfaceTypeConversionPattern<func::FuncOp>(
        patterns, typeConverter);
    patterns.add<ConvertGenericOpwithSubChannelType>(typeConverter, context);

```
- **EN**: Implements logic around `patterns`, `FuncOp>`, `add`; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `patterns`, `FuncOp>`, `add` 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 169-175
```cpp
    // Apply conversion
    if (failed(
            applyFullConversion(getOperation(), target, std::move(patterns))))
      signalPassFailure();
  }
};

```
- **EN**: Implements logic around `failed`, `applyFullConversion`, `signalPassFailure`.
- **CN**: 围绕 `failed`, `applyFullConversion`, `signalPassFailure` 实现具体逻辑。

### Lines 176-179
```cpp
} // namespace

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
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Quant/IR/Quant.h`, `mlir/Dialect/Quant/IR/QuantTypes.h`, `mlir/Dialect/Quant/Transforms/Passes.h`, `mlir/Transforms/DialectConversion.h`, `llvm/ADT/SmallVectorExtras.h`, `mlir/Dialect/Quant/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
