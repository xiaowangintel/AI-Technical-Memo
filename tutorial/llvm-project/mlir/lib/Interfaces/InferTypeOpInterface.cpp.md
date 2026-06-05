# InferTypeOpInterface.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Interfaces/InferTypeOpInterface.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains the definitions of the infer op interfaces defined in `InferTypeOpInterface.td`.
  - **CN**: 实现 MLIR 接口或生成的接口适配胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- InferTypeOpInterface.cpp - Infer Type Interfaces ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-19
```cpp
//
// This file contains the definitions of the infer op interfaces defined in
// `InferTypeOpInterface.td`.
//
//===----------------------------------------------------------------------===//

#include "mlir/Interfaces/InferTypeOpInterface.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Matchers.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/InterleavedRange.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Interfaces/InferTypeOpInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Matchers.h`, `llvm/Support/FormatVariadic.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Interfaces/InferTypeOpInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Matchers.h`, `llvm/Support/FormatVariadic.h`。

### Lines 20-33
```cpp
using namespace mlir;

namespace mlir {
#include "mlir/Interfaces/InferTypeOpInterface.cpp.inc"
} // namespace mlir

LogicalResult
mlir::reifyResultShapes(OpBuilder &b, Operation *op,
                        ReifiedRankedShapedTypeDims &reifiedReturnShapes) {
  auto reifiableOp = dyn_cast<ReifyRankedShapedTypeOpInterface>(op);
  if (!reifiableOp)
    return failure();
  LogicalResult status = reifiableOp.reifyResultShapes(b, reifiedReturnShapes);
#ifndef NDEBUG
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Interfaces/InferTypeOpInterface.cpp.inc`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Interfaces/InferTypeOpInterface.cpp.inc`。

### Lines 34-47
```cpp
  if (failed(status))
    return failure();
  // Assert that ReifyRankedShapedTypeOpInterface::reifyResultShapes produced
  // a correct result.
  int64_t resultIdx = 0;
  for (OpResult result : op->getResults()) {
    auto shapedType = dyn_cast<ShapedType>(result.getType());
    if (!shapedType)
      continue;
    if (!shapedType.hasRank()) {
      // Nothing to check for unranked shaped values.
      ++resultIdx;
      continue;
    }
```
- **EN**: Implements logic around `failed`, `failure`, `getResults`, `dyn_cast`, and 1 more symbols.
- **CN**: 围绕 `failed`、`failure`、`getResults`、`dyn_cast` 等另外 1 个符号 实现具体逻辑。

### Lines 48-60
```cpp
    // Assert one OpFoldResult per dimension.
    assert(shapedType.getRank() ==
               static_cast<int64_t>(reifiedReturnShapes[resultIdx].size()) &&
           "incorrect implementation of ReifyRankedShapedTypeOpInterface");
    ++resultIdx;
  }
  // Assert that every shaped value result was reified.
  assert(resultIdx == static_cast<int64_t>(reifiedReturnShapes.size()) &&
         "incorrect implementation of ReifyRankedShapedTypeOpInterface");
#endif // NDEBUG
  return status;
}

```
- **EN**: Implements logic around `assert`, `static_cast`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `assert`、`static_cast` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 61-68
```cpp
FailureOr<SmallVector<OpFoldResult>>
mlir::reifyShapeOfResult(OpBuilder &b, Operation *op, int resultIndex) {
  auto reifiableOp = dyn_cast<ReifyRankedShapedTypeOpInterface>(op);
  if (!reifiableOp)
    return failure();
  return reifiableOp.reifyShapeOfResult(b, resultIndex);
}

```
- **EN**: Implements logic around `reifyShapeOfResult`, `dyn_cast`, `failure`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `reifyShapeOfResult`、`dyn_cast`、`failure` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 69-76
```cpp
FailureOr<OpFoldResult> mlir::reifyDimOfResult(OpBuilder &b, Operation *op,
                                               int resultIndex, int dim) {
  auto reifiableOp = dyn_cast<ReifyRankedShapedTypeOpInterface>(op);
  if (!reifiableOp)
    return failure();
  return reifiableOp.reifyDimOfResult(b, resultIndex, dim);
}

```
- **EN**: Implements logic around `reifyDimOfResult`, `dyn_cast`, `failure`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `reifyDimOfResult`、`dyn_cast`、`failure` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 77-86
```cpp
bool ShapeAdaptor::hasRank() const {
  if (val.isNull())
    return false;
  if (auto t = llvm::dyn_cast_if_present<Type>(val))
    return cast<ShapedType>(t).hasRank();
  if (isa<Attribute>(val))
    return true;
  return cast<ShapedTypeComponents *>(val)->hasRank();
}

```
- **EN**: Implements logic around `hasRank`, `isNull`, `dyn_cast_if_present`, `cast`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `hasRank`、`isNull`、`dyn_cast_if_present`、`cast` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 87-96
```cpp
Type ShapeAdaptor::getElementType() const {
  if (val.isNull())
    return nullptr;
  if (auto t = llvm::dyn_cast_if_present<Type>(val))
    return cast<ShapedType>(t).getElementType();
  if (isa<Attribute>(val))
    return nullptr;
  return cast<ShapedTypeComponents *>(val)->getElementType();
}

```
- **EN**: Implements logic around `getElementType`, `isNull`, `dyn_cast_if_present`, `cast`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getElementType`、`isNull`、`dyn_cast_if_present`、`cast` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 97-110
```cpp
void ShapeAdaptor::getDims(SmallVectorImpl<int64_t> &res) const {
  assert(hasRank());
  if (auto t = llvm::dyn_cast_if_present<Type>(val)) {
    ArrayRef<int64_t> vals = cast<ShapedType>(t).getShape();
    res.assign(vals.begin(), vals.end());
  } else if (auto attr = llvm::dyn_cast_if_present<Attribute>(val)) {
    auto dattr = cast<DenseIntElementsAttr>(attr);
    res.clear();
    res.reserve(dattr.size());
    for (auto it : dattr.getValues<APInt>())
      res.push_back(it.getSExtValue());
  } else {
    auto vals = cast<ShapedTypeComponents *>(val)->getDims();
    res.assign(vals.begin(), vals.end());
```
- **EN**: Implements logic around `getDims`, `assert`, `dyn_cast_if_present`, `cast`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDims`、`assert`、`dyn_cast_if_present`、`cast` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 111-119
```cpp
  }
}

void ShapeAdaptor::getDims(ShapedTypeComponents &res) const {
  assert(hasRank());
  res.ranked = true;
  getDims(res.dims);
}

```
- **EN**: Implements logic around `getDims`, `assert`.
- **CN**: 围绕 `getDims`、`assert` 实现具体逻辑。

### Lines 120-131
```cpp
int64_t ShapeAdaptor::getDimSize(int index) const {
  assert(hasRank());
  if (auto t = llvm::dyn_cast_if_present<Type>(val))
    return cast<ShapedType>(t).getDimSize(index);
  if (auto attr = llvm::dyn_cast_if_present<Attribute>(val))
    return cast<DenseIntElementsAttr>(attr)
        .getValues<APInt>()[index]
        .getSExtValue();
  auto *stc = cast<ShapedTypeComponents *>(val);
  return stc->getDims()[index];
}

```
- **EN**: Implements logic around `getDimSize`, `assert`, `dyn_cast_if_present`, `cast`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDimSize`、`assert`、`dyn_cast_if_present`、`cast` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 132-140
```cpp
int64_t ShapeAdaptor::getRank() const {
  assert(hasRank());
  if (auto t = llvm::dyn_cast_if_present<Type>(val))
    return cast<ShapedType>(t).getRank();
  if (auto attr = llvm::dyn_cast_if_present<Attribute>(val))
    return cast<DenseIntElementsAttr>(attr).size();
  return cast<ShapedTypeComponents *>(val)->getDims().size();
}

```
- **EN**: Implements logic around `getRank`, `assert`, `dyn_cast_if_present`, `cast`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getRank`、`assert`、`dyn_cast_if_present`、`cast` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 141-154
```cpp
bool ShapeAdaptor::hasStaticShape() const {
  if (!hasRank())
    return false;

  if (auto t = llvm::dyn_cast_if_present<Type>(val))
    return cast<ShapedType>(t).hasStaticShape();
  if (auto attr = llvm::dyn_cast_if_present<Attribute>(val)) {
    auto dattr = cast<DenseIntElementsAttr>(attr);
    for (auto index : dattr.getValues<APInt>())
      if (ShapedType::isDynamic(index.getSExtValue()))
        return false;
    return true;
  }
  auto *stc = cast<ShapedTypeComponents *>(val);
```
- **EN**: Implements logic around `hasStaticShape`, `hasRank`, `dyn_cast_if_present`, `cast`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `hasStaticShape`、`hasRank`、`dyn_cast_if_present`、`cast` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 155-163
```cpp
  return llvm::none_of(stc->getDims(), ShapedType::isDynamic);
}

int64_t ShapeAdaptor::getNumElements() const {
  assert(hasStaticShape() && "cannot get element count of dynamic shaped type");

  if (auto t = llvm::dyn_cast_if_present<Type>(val))
    return cast<ShapedType>(t).getNumElements();

```
- **EN**: Implements logic around `none_of`, `getNumElements`, `assert`, `dyn_cast_if_present`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `none_of`、`getNumElements`、`assert`、`dyn_cast_if_present` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 164-173
```cpp
  if (auto attr = llvm::dyn_cast_if_present<Attribute>(val)) {
    auto dattr = cast<DenseIntElementsAttr>(attr);
    int64_t num = 1;
    for (auto index : dattr.getValues<APInt>()) {
      num *= index.getZExtValue();
      assert(num >= 0 && "integer overflow in element count computation");
    }
    return num;
  }

```
- **EN**: Implements logic around `dyn_cast_if_present`, `cast`, `getValues`, `getZExtValue`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `dyn_cast_if_present`、`cast`、`getValues`、`getZExtValue` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 174-182
```cpp
  auto *stc = cast<ShapedTypeComponents *>(val);
  int64_t num = 1;
  for (int64_t dim : stc->getDims()) {
    num *= dim;
    assert(num >= 0 && "integer overflow in element count computation");
  }
  return num;
}

```
- **EN**: Implements logic around `getDims`, `assert`.
- **CN**: 围绕 `getDims`、`assert` 实现具体逻辑。

### Lines 183-196
```cpp
void ShapeAdaptor::dump() const {
  if (!hasRank()) {
    llvm::errs() << "<<unranked>>\n";
    return;
  }

  SmallVector<int64_t> dims;
  getDims(dims);
  auto mapped = llvm::map_range(dims, [](int64_t dim) -> std::string {
    if (ShapedType::isDynamic(dim))
      return "?";
    return llvm::formatv("{0}", dim).str();
  });
  llvm::errs() << "rank = " << getRank()
```
- **EN**: Implements logic around `dump`, `hasRank`, `errs`, `getDims`, and 3 more symbols.
- **CN**: 围绕 `dump`、`hasRank`、`errs`、`getDims` 等另外 3 个符号 实现具体逻辑。

### Lines 197-205
```cpp
               << " dims = " << llvm::interleaved_array(mapped, "x") << "\n";
}

ShapeAdaptor ValueShapeRange::getValueAsShape(int index) {
  Value val = operator[](index);
  if (valueToShape)
    if (ShapeAdaptor ret = valueToShape(val))
      return ret;

```
- **EN**: Implements logic around `interleaved_array`, `getValueAsShape`, `valueToShape`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `interleaved_array`、`getValueAsShape`、`valueToShape` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 206-213
```cpp
  DenseIntElementsAttr attr;
  if (!matchPattern(val, m_Constant(&attr)))
    return nullptr;
  if (attr.getType().getRank() != 1)
    return nullptr;
  return attr;
}

```
- **EN**: Implements logic around `matchPattern`, `getType`.
- **CN**: 围绕 `matchPattern`、`getType` 实现具体逻辑。

### Lines 214-220
```cpp
ShapeAdaptor ValueShapeRange::getShape(Value val) const {
  if (operandShape)
    if (ShapeAdaptor ret = operandShape(val))
      return ret;
  return val.getType();
}

```
- **EN**: Implements logic around `getShape`, `operandShape`, `getType`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getShape`、`operandShape`、`getType` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 221-233
```cpp
ShapeAdaptor ValueShapeRange::getShape(int index) const {
  if (index < 0 || static_cast<size_t>(index) >= size())
    return nullptr;
  return getShape(operator[](index));
}

LogicalResult mlir::detail::inferReturnTensorTypes(
    ArrayRef<ShapedTypeComponents> retComponents,
    SmallVectorImpl<Type> &inferredReturnTypes) {
  for (const auto &shapeAndType : retComponents) {
    Type elementTy = shapeAndType.getElementType();
    assert(elementTy && "element type required to construct tensor");

```
- **EN**: Implements logic around `getShape`, `static_cast`, `inferReturnTensorTypes`, `getElementType`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `getShape`、`static_cast`、`inferReturnTensorTypes`、`getElementType` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 234-245
```cpp
    Attribute attr = shapeAndType.getAttribute();
    if (shapeAndType.hasRank()) {
      inferredReturnTypes.push_back(
          RankedTensorType::get(shapeAndType.getDims(), elementTy, attr));
    } else {
      assert(attr == nullptr && "attribute not supported");
      inferredReturnTypes.push_back(UnrankedTensorType::get(elementTy));
    }
  }
  return success();
}

```
- **EN**: Implements logic around `getAttribute`, `hasRank`, `push_back`, `get`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getAttribute`、`hasRank`、`push_back`、`get` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 246-255
```cpp
LogicalResult mlir::detail::verifyInferredResultTypes(Operation *op) {
  SmallVector<Type, 4> inferredReturnTypes(op->getResultTypes());
  auto retTypeFn = cast<InferTypeOpInterface>(op);
  auto result = retTypeFn.refineReturnTypes(
      op->getContext(), op->getLoc(), op->getOperands(),
      op->getRawDictionaryAttrs(), op->getPropertiesStorage(), op->getRegions(),
      inferredReturnTypes);
  if (failed(result))
    op->emitOpError() << "failed to infer returned types";

```
- **EN**: Implements logic around `verifyInferredResultTypes`, `inferredReturnTypes`, `cast`, `refineReturnTypes`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `verifyInferredResultTypes`、`inferredReturnTypes`、`cast`、`refineReturnTypes` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 256-269
```cpp
  return result;
}

void mlir::detail::reportFatalInferReturnTypesError(OperationState &state) {
  std::string buffer;
  llvm::raw_string_ostream os(buffer);
  os << "Failed to infer result type(s):\n"
     << "\"" << state.name << "\"(...) "
     << state.attributes.getDictionary(state.location.getContext()) << " : ("
     << llvm::interleaved(llvm::map_range(
            state.operands, [](Value val) { return val.getType(); }))
     << ") -> ( ??? )";
  emitRemark(state.location, "location of op");
  llvm::report_fatal_error(llvm::StringRef(buffer));
```
- **EN**: Implements logic around `reportFatalInferReturnTypesError`, `os`, `type`, `getDictionary`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `reportFatalInferReturnTypesError`、`os`、`type`、`getDictionary` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 270-270
```cpp
}
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Uses MLIR interfaces to describe cross-cutting behavior independently from concrete operation classes.
  - **CN**: 使用 MLIR 接口在不依赖具体操作类的前提下描述横切行为。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Interfaces/InferTypeOpInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Matchers.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/InterleavedRange.h`, `mlir/Interfaces/InferTypeOpInterface.cpp.inc`
- **Subsystem categories / 子系统类别**: MLIR interface declarations / MLIR 接口声明 (2), core MLIR IR abstractions / 核心 MLIR IR 抽象 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (2)
