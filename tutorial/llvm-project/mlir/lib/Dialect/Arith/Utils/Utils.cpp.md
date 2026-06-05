# Utils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Arith/Utils/Utils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements utilities for the Linalg dialect.
  - **CN**: 实现 Arith 方言与标量/向量算术语义 使用的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Utils.cpp - Utilities to support the Linalg dialect ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements utilities for the Linalg dialect.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-22
```cpp

#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/AsmParser/AsmParser.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/Diagnostics.h"
#include "llvm/ADT/SmallBitVector.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include <numeric>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`。

### Lines 23-38
```cpp
using namespace mlir;

std::optional<SmallVector<OpFoldResult>>
mlir::inferExpandShapeOutputShape(OpBuilder &b, Location loc,
                                  ShapedType expandedType,
                                  ArrayRef<ReassociationIndices> reassociation,
                                  ArrayRef<OpFoldResult> inputShape) {

  SmallVector<Value> outputShapeValues;
  SmallVector<int64_t> outputShapeInts;
  // For zero-rank inputs, all dims in result shape are unit extent.
  if (inputShape.empty()) {
    outputShapeInts.resize(expandedType.getRank(), 1);
    return getMixedValues(outputShapeInts, outputShapeValues, b);
  }

```
- **EN**: Implements logic around `inferExpandShapeOutputShape`, `empty`, `resize`, `getMixedValues`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `inferExpandShapeOutputShape`, `empty`, `resize`, `getMixedValues` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 39-49
```cpp
  // Check for all static shapes.
  if (expandedType.hasStaticShape()) {
    ArrayRef<int64_t> staticShape = expandedType.getShape();
    outputShapeInts.assign(staticShape.begin(), staticShape.end());
    return getMixedValues(outputShapeInts, outputShapeValues, b);
  }

  outputShapeInts.resize(expandedType.getRank(), ShapedType::kDynamic);
  for (const auto &it : llvm::enumerate(reassociation)) {
    ReassociationIndices indexGroup = it.value();

```
- **EN**: Implements logic around `hasStaticShape`, `getShape`, `assign`, `getMixedValues`, and 3 more symbols.
- **CN**: 围绕 `hasStaticShape`, `getShape`, `assign`, `getMixedValues`, and 3 more symbols 实现具体逻辑。

### Lines 50-67
```cpp
    int64_t indexGroupStaticSizesProductInt = 1;
    bool foundDynamicShape = false;
    for (int64_t index : indexGroup) {
      int64_t outputDimSize = expandedType.getDimSize(index);
      // Cannot infer expanded shape with multiple dynamic dims in the
      // same reassociation group!
      if (ShapedType::isDynamic(outputDimSize)) {
        if (foundDynamicShape)
          return std::nullopt;
        foundDynamicShape = true;
      } else {
        outputShapeInts[index] = outputDimSize;
        indexGroupStaticSizesProductInt *= outputDimSize;
      }
    }
    if (!foundDynamicShape)
      continue;

```
- **EN**: Implements logic around `getDimSize`, `isDynamic`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getDimSize`, `isDynamic` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 68-78
```cpp
    int64_t inputIndex = it.index();
    // Call get<Value>() under the assumption that we're not casting
    // dynamism.
    Value indexGroupSize = cast<Value>(inputShape[inputIndex]);
    Value indexGroupStaticSizesProduct =
        arith::ConstantIndexOp::create(b, loc, indexGroupStaticSizesProductInt);
    Value dynamicDimSize = b.createOrFold<arith::DivSIOp>(
        loc, indexGroupSize, indexGroupStaticSizesProduct);
    outputShapeValues.push_back(dynamicDimSize);
  }

```
- **EN**: Implements logic around `index`, `cast`, `create`, `DivSIOp>`, and 1 more symbols.
- **CN**: 围绕 `index`, `cast`, `create`, `DivSIOp>`, and 1 more symbols 实现具体逻辑。

### Lines 79-88
```cpp
  if ((int64_t)outputShapeValues.size() !=
      llvm::count(outputShapeInts, ShapedType::kDynamic))
    return std::nullopt;

  return getMixedValues(outputShapeInts, outputShapeValues, b);
}

/// Matches a ConstantIndexOp.
/// TODO: This should probably just be a general matcher that uses matchConstant
/// and checks the operation for an index type.
```
- **EN**: Implements logic around `size`, `count`, `getMixedValues`.
- **CN**: 围绕 `size`, `count`, `getMixedValues` 实现具体逻辑。

### Lines 89-104
```cpp
detail::op_matcher<arith::ConstantIndexOp> mlir::matchConstantIndex() {
  return detail::op_matcher<arith::ConstantIndexOp>();
}

llvm::SmallBitVector mlir::getPositionsOfShapeOne(unsigned rank,
                                                  ArrayRef<int64_t> shape) {
  llvm::SmallBitVector dimsToProject(shape.size());
  for (unsigned pos = 0, e = shape.size(); pos < e && rank > 0; ++pos) {
    if (shape[pos] == 1) {
      dimsToProject.set(pos);
      --rank;
    }
  }
  return dimsToProject;
}

```
- **EN**: Implements logic around `matchConstantIndex`, `ConstantIndexOp>`, `getPositionsOfShapeOne`, `dimsToProject`, and 2 more symbols.
- **CN**: 围绕 `matchConstantIndex`, `ConstantIndexOp>`, `getPositionsOfShapeOne`, `dimsToProject`, and 2 more symbols 实现具体逻辑。

### Lines 105-121
```cpp
Value mlir::getValueOrCreateConstantIntOp(OpBuilder &b, Location loc,
                                          OpFoldResult ofr) {
  if (auto value = dyn_cast_if_present<Value>(ofr))
    return value;
  auto attr = cast<IntegerAttr>(cast<Attribute>(ofr));
  return arith::ConstantOp::create(
      b, loc, b.getIntegerAttr(attr.getType(), attr.getValue().getSExtValue()));
}

Value mlir::getValueOrCreateConstantIndexOp(OpBuilder &b, Location loc,
                                            OpFoldResult ofr) {
  if (auto value = dyn_cast_if_present<Value>(ofr))
    return value;
  auto attr = cast<IntegerAttr>(cast<Attribute>(ofr));
  return arith::ConstantIndexOp::create(b, loc, attr.getValue().getSExtValue());
}

```
- **EN**: Implements logic around `getValueOrCreateConstantIntOp`, `dyn_cast_if_present`, `cast`, `create`, and 2 more symbols.
- **CN**: 围绕 `getValueOrCreateConstantIntOp`, `dyn_cast_if_present`, `cast`, `create`, and 2 more symbols 实现具体逻辑。

### Lines 122-131
```cpp
Value mlir::getValueOrCreateCastToIndexLike(OpBuilder &b, Location loc,
                                            Type targetType, Value value) {
  if (targetType == value.getType())
    return value;

  bool targetIsIndex = targetType.isIndex();
  bool valueIsIndex = value.getType().isIndex();
  if (targetIsIndex ^ valueIsIndex)
    return arith::IndexCastOp::create(b, loc, targetType, value);

```
- **EN**: Implements logic around `getValueOrCreateCastToIndexLike`, `getType`, `isIndex`, `create`.
- **CN**: 围绕 `getValueOrCreateCastToIndexLike`, `getType`, `isIndex`, `create` 实现具体逻辑。

### Lines 132-142
```cpp
  auto targetIntegerType = dyn_cast<IntegerType>(targetType);
  auto valueIntegerType = dyn_cast<IntegerType>(value.getType());
  assert(targetIntegerType && valueIntegerType &&
         "unexpected cast between types other than integers and index");
  assert(targetIntegerType.getSignedness() == valueIntegerType.getSignedness());

  if (targetIntegerType.getWidth() > valueIntegerType.getWidth())
    return arith::ExtSIOp::create(b, loc, targetIntegerType, value);
  return arith::TruncIOp::create(b, loc, targetIntegerType, value);
}

```
- **EN**: Implements logic around `dyn_cast`, `assert`, `getWidth`, `create`.
- **CN**: 围绕 `dyn_cast`, `assert`, `getWidth`, `create` 实现具体逻辑。

### Lines 143-162
```cpp
static Value convertScalarToIntDtype(ImplicitLocOpBuilder &b, Value operand,
                                     IntegerType toType, bool isUnsigned) {
  // If operand is floating point, cast directly to the int type.
  if (isa<FloatType>(operand.getType())) {
    if (isUnsigned)
      return arith::FPToUIOp::create(b, toType, operand);
    return arith::FPToSIOp::create(b, toType, operand);
  }
  // Cast index operands directly to the int type.
  if (operand.getType().isIndex())
    return arith::IndexCastOp::create(b, toType, operand);
  if (auto fromIntType = dyn_cast<IntegerType>(operand.getType())) {
    // Either extend or truncate.
    if (toType.getWidth() > fromIntType.getWidth()) {
      if (isUnsigned)
        return arith::ExtUIOp::create(b, toType, operand);
      return arith::ExtSIOp::create(b, toType, operand);
    }
    if (toType.getWidth() < fromIntType.getWidth())
      return arith::TruncIOp::create(b, toType, operand);
```
- **EN**: Implements logic around `convertScalarToIntDtype`, `isa`, `create`, `getType`, and 2 more symbols.
- **CN**: 围绕 `convertScalarToIntDtype`, `isa`, `create`, `getType`, and 2 more symbols 实现具体逻辑。

### Lines 163-182
```cpp
    return operand;
  }

  return {};
}

static Value convertScalarToFpDtype(ImplicitLocOpBuilder &b, Value operand,
                                    FloatType toType, bool isUnsigned) {
  // If operand is integer, cast directly to the float type.
  // Note that it is unclear how to cast from BF16<->FP16.
  if (isa<IntegerType>(operand.getType())) {
    if (isUnsigned)
      return arith::UIToFPOp::create(b, toType, operand);
    return arith::SIToFPOp::create(b, toType, operand);
  }
  if (auto fromFpTy = dyn_cast<FloatType>(operand.getType())) {
    if (toType.getWidth() > fromFpTy.getWidth())
      return arith::ExtFOp::create(b, toType, operand);
    if (toType.getWidth() < fromFpTy.getWidth())
      return arith::TruncFOp::create(b, toType, operand);
```
- **EN**: Implements logic around `convertScalarToFpDtype`, `isa`, `create`, `dyn_cast`, and 1 more symbols.
- **CN**: 围绕 `convertScalarToFpDtype`, `isa`, `create`, `dyn_cast`, and 1 more symbols 实现具体逻辑。

### Lines 183-202
```cpp
    return operand;
  }

  return {};
}

static Value convertScalarToComplexDtype(ImplicitLocOpBuilder &b, Value operand,
                                         ComplexType targetType,
                                         bool isUnsigned) {
  if (auto fromComplexType = dyn_cast<ComplexType>(operand.getType())) {
    if (isa<FloatType>(targetType.getElementType()) &&
        isa<FloatType>(fromComplexType.getElementType())) {
      Value real = complex::ReOp::create(b, operand);
      Value imag = complex::ImOp::create(b, operand);
      Type targetETy = targetType.getElementType();
      if (targetType.getElementType().getIntOrFloatBitWidth() <
          fromComplexType.getElementType().getIntOrFloatBitWidth()) {
        real = arith::TruncFOp::create(b, targetETy, real);
        imag = arith::TruncFOp::create(b, targetETy, imag);
      } else {
```
- **EN**: Implements logic around `convertScalarToComplexDtype`, `dyn_cast`, `isa`, `create`, and 1 more symbols.
- **CN**: 围绕 `convertScalarToComplexDtype`, `dyn_cast`, `isa`, `create`, and 1 more symbols 实现具体逻辑。

### Lines 203-222
```cpp
        real = arith::ExtFOp::create(b, targetETy, real);
        imag = arith::ExtFOp::create(b, targetETy, imag);
      }
      return complex::CreateOp::create(b, targetType, real, imag);
    }
  }

  if (isa<FloatType>(operand.getType())) {
    FloatType toFpTy = cast<FloatType>(targetType.getElementType());
    auto toBitwidth = toFpTy.getIntOrFloatBitWidth();
    Value from = operand;
    if (from.getType().getIntOrFloatBitWidth() < toBitwidth) {
      from = arith::ExtFOp::create(b, toFpTy, from);
    }
    if (from.getType().getIntOrFloatBitWidth() > toBitwidth) {
      from = arith::TruncFOp::create(b, toFpTy, from);
    }
    Value zero = mlir::arith::ConstantFloatOp::create(
        b, toFpTy, mlir::APFloat(toFpTy.getFloatSemantics(), 0));
    return complex::CreateOp::create(b, targetType, from, zero);
```
- **EN**: Implements logic around `create`, `isa`, `cast`, `getIntOrFloatBitWidth`, and 2 more symbols.
- **CN**: 围绕 `create`, `isa`, `cast`, `getIntOrFloatBitWidth`, and 2 more symbols 实现具体逻辑。

### Lines 223-237
```cpp
  }

  if (isa<IntegerType>(operand.getType())) {
    FloatType toFpTy = cast<FloatType>(targetType.getElementType());
    Value from = operand;
    if (isUnsigned) {
      from = arith::UIToFPOp::create(b, toFpTy, from);
    } else {
      from = arith::SIToFPOp::create(b, toFpTy, from);
    }
    Value zero = mlir::arith::ConstantFloatOp::create(
        b, toFpTy, mlir::APFloat(toFpTy.getFloatSemantics(), 0));
    return complex::CreateOp::create(b, targetType, from, zero);
  }

```
- **EN**: Implements logic around `isa`, `cast`, `create`, `APFloat`.
- **CN**: 围绕 `isa`, `cast`, `create`, `APFloat` 实现具体逻辑。

### Lines 238-255
```cpp
  return {};
}

Value mlir::convertScalarToDtype(OpBuilder &b, Location loc, Value operand,
                                 Type toType, bool isUnsignedCast) {
  if (operand.getType() == toType)
    return operand;
  ImplicitLocOpBuilder ib(loc, b);
  Value result;
  if (auto intTy = dyn_cast<IntegerType>(toType)) {
    result = convertScalarToIntDtype(ib, operand, intTy, isUnsignedCast);
  } else if (auto floatTy = dyn_cast<FloatType>(toType)) {
    result = convertScalarToFpDtype(ib, operand, floatTy, isUnsignedCast);
  } else if (auto complexTy = dyn_cast<ComplexType>(toType)) {
    result =
        convertScalarToComplexDtype(ib, operand, complexTy, isUnsignedCast);
  }

```
- **EN**: Implements logic around `convertScalarToDtype`, `getType`, `ib`, `dyn_cast`, and 3 more symbols.
- **CN**: 围绕 `convertScalarToDtype`, `getType`, `ib`, `dyn_cast`, and 3 more symbols 实现具体逻辑。

### Lines 256-272
```cpp
  if (result)
    return result;

  emitWarning(loc) << "could not cast operand of type " << operand.getType()
                   << " to " << toType;
  return operand;
}

SmallVector<Value>
mlir::getValueOrCreateConstantIndexOp(OpBuilder &b, Location loc,
                                      ArrayRef<OpFoldResult> valueOrAttrVec) {
  return llvm::map_to_vector<4>(
      valueOrAttrVec, [&](OpFoldResult value) -> Value {
        return getValueOrCreateConstantIndexOp(b, loc, value);
      });
}

```
- **EN**: Implements logic around `emitWarning`, `getValueOrCreateConstantIndexOp`, `map_to_vector`.
- **CN**: 围绕 `emitWarning`, `getValueOrCreateConstantIndexOp`, `map_to_vector` 实现具体逻辑。

### Lines 273-282
```cpp
Value mlir::createScalarOrSplatConstant(OpBuilder &builder, Location loc,
                                        Type type, const APInt &value) {
  TypedAttr attr;
  if (isa<IntegerType>(type)) {
    attr = builder.getIntegerAttr(type, value);
  } else {
    auto vecTy = cast<ShapedType>(type);
    attr = SplatElementsAttr::get(vecTy, value);
  }

```
- **EN**: Implements logic around `createScalarOrSplatConstant`, `isa`, `getIntegerAttr`, `cast`, and 1 more symbols.
- **CN**: 围绕 `createScalarOrSplatConstant`, `isa`, `getIntegerAttr`, `cast`, and 1 more symbols 实现具体逻辑。

### Lines 283-293
```cpp
  return arith::ConstantOp::create(builder, loc, attr);
}

Value mlir::createScalarOrSplatConstant(OpBuilder &builder, Location loc,
                                        Type type, int64_t value) {
  unsigned elementBitWidth = 0;
  if (auto intTy = dyn_cast<IntegerType>(type))
    elementBitWidth = intTy.getWidth();
  else
    elementBitWidth = cast<ShapedType>(type).getElementTypeBitWidth();

```
- **EN**: Implements logic around `create`, `createScalarOrSplatConstant`, `dyn_cast`, `getWidth`, and 1 more symbols.
- **CN**: 围绕 `create`, `createScalarOrSplatConstant`, `dyn_cast`, `getWidth`, and 1 more symbols 实现具体逻辑。

### Lines 294-306
```cpp
  return createScalarOrSplatConstant(builder, loc, type,
                                     APInt(elementBitWidth, value));
}

Value mlir::createScalarOrSplatConstant(OpBuilder &builder, Location loc,
                                        Type type, const APFloat &value) {
  if (isa<FloatType>(type))
    return builder.createOrFold<arith::ConstantOp>(
        loc, type, builder.getFloatAttr(type, value));
  TypedAttr splat = SplatElementsAttr::get(cast<ShapedType>(type), value);
  return builder.createOrFold<arith::ConstantOp>(loc, type, splat);
}

```
- **EN**: Implements logic around `createScalarOrSplatConstant`, `APInt`, `isa`, `ConstantOp>`, and 2 more symbols.
- **CN**: 围绕 `createScalarOrSplatConstant`, `APInt`, `isa`, `ConstantOp>`, and 2 more symbols 实现具体逻辑。

### Lines 307-326
```cpp
Type mlir::getType(OpFoldResult ofr) {
  if (auto value = dyn_cast_if_present<Value>(ofr))
    return value.getType();
  auto attr = cast<IntegerAttr>(cast<Attribute>(ofr));
  return attr.getType();
}

Value ArithBuilder::_and(Value lhs, Value rhs) {
  return arith::AndIOp::create(b, loc, lhs, rhs);
}
Value ArithBuilder::add(Value lhs, Value rhs) {
  if (isa<FloatType>(lhs.getType()))
    return arith::AddFOp::create(b, loc, lhs, rhs);
  return arith::AddIOp::create(b, loc, lhs, rhs, ovf);
}
Value ArithBuilder::sub(Value lhs, Value rhs) {
  if (isa<FloatType>(lhs.getType()))
    return arith::SubFOp::create(b, loc, lhs, rhs);
  return arith::SubIOp::create(b, loc, lhs, rhs, ovf);
}
```
- **EN**: Implements logic around `getType`, `dyn_cast_if_present`, `cast`, `_and`, and 4 more symbols.
- **CN**: 围绕 `getType`, `dyn_cast_if_present`, `cast`, `_and`, and 4 more symbols 实现具体逻辑。

### Lines 327-345
```cpp
Value ArithBuilder::mul(Value lhs, Value rhs) {
  if (isa<FloatType>(lhs.getType()))
    return arith::MulFOp::create(b, loc, lhs, rhs);
  return arith::MulIOp::create(b, loc, lhs, rhs, ovf);
}
Value ArithBuilder::sgt(Value lhs, Value rhs) {
  if (isa<FloatType>(lhs.getType()))
    return arith::CmpFOp::create(b, loc, arith::CmpFPredicate::OGT, lhs, rhs);
  return arith::CmpIOp::create(b, loc, arith::CmpIPredicate::sgt, lhs, rhs);
}
Value ArithBuilder::slt(Value lhs, Value rhs) {
  if (isa<FloatType>(lhs.getType()))
    return arith::CmpFOp::create(b, loc, arith::CmpFPredicate::OLT, lhs, rhs);
  return arith::CmpIOp::create(b, loc, arith::CmpIPredicate::slt, lhs, rhs);
}
Value ArithBuilder::select(Value cmp, Value lhs, Value rhs) {
  return arith::SelectOp::create(b, loc, cmp, lhs, rhs);
}

```
- **EN**: Implements logic around `mul`, `isa`, `create`, `sgt`, and 2 more symbols.
- **CN**: 围绕 `mul`, `isa`, `create`, `sgt`, and 2 more symbols 实现具体逻辑。

### Lines 346-361
```cpp
namespace mlir::arith {

Value createProduct(OpBuilder &builder, Location loc, ArrayRef<Value> values) {
  return createProduct(builder, loc, values, values.front().getType());
}

Value createProduct(OpBuilder &builder, Location loc, ArrayRef<Value> values,
                    Type resultType) {
  Value one = ConstantOp::create(builder, loc, resultType,
                                 builder.getOneAttr(resultType));
  ArithBuilder arithBuilder(builder, loc);
  return llvm::accumulate(values, one, [&arithBuilder](Value acc, Value v) {
    return arithBuilder.mul(acc, v);
  });
}

```
- **EN**: Introduces declarations for `mlir::arith`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arith` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 362-372
```cpp
FloatType parseFloatType(MLIRContext *ctx, StringRef name) {
  // Parsing non-builtin types is unsafe because the respective dialect may not
  // have been loaded.
  if (!name.empty() && name.front() == '!')
    return FloatType();

  // Suppress diagnostics: callers handle invalid type strings themselves.
  ScopedDiagnosticHandler handler(ctx, [](Diagnostic &) {});
  return dyn_cast_or_null<FloatType>(mlir::parseType(name, ctx));
}

```
- **EN**: Implements logic around `parseFloatType`, `empty`, `FloatType`, `handler`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `parseFloatType`, `empty`, `FloatType`, `handler`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 373-373
```cpp
} // namespace mlir::arith
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Utility helpers / 辅助工具**:
  - **EN**: Provides reusable helpers that keep dialect implementations and passes smaller.
  - **CN**: 提供可复用的辅助函数，使方言实现和 pass 保持精简。
- **Arithmetic semantics / 算术语义**:
  - **EN**: Defines foldable scalar/vector arithmetic operations and constant semantics.
  - **CN**: 定义可折叠的标量/向量算术操作与常量语义。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/Diagnostics.h`, `llvm/ADT/SmallBitVector.h`, `llvm/ADT/SmallVectorExtras.h`
- **Standard-library headers / 标准库头文件**: `<numeric>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (4), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (1)
