# TypeUtilities.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/IR/TypeUtilities.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines generic type utilities.
  - **CN**: 实现 MLIR 核心 IR 对象、打印/解析支持以及内建类型或属性行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- TypeUtilities.cpp - Helper function for type queries ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-20
```cpp
//
// This file defines generic type utilities.
//
//===----------------------------------------------------------------------===//

#include "mlir/IR/TypeUtilities.h"
#include "mlir/IR/Attributes.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Types.h"
#include "mlir/IR/Value.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include <numeric>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/IR/TypeUtilities.h`, `mlir/IR/Attributes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Types.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/IR/TypeUtilities.h`, `mlir/IR/Attributes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Types.h`。

### Lines 21-28
```cpp
using namespace mlir;

Type mlir::getElementTypeOrSelf(Type type) {
  if (auto st = llvm::dyn_cast<ShapedType>(type))
    return st.getElementType();
  return type;
}

```
- **EN**: Implements logic around `getElementTypeOrSelf`, `dyn_cast`, `getElementType`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getElementTypeOrSelf`、`dyn_cast`、`getElementType` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 29-38
```cpp
Type mlir::getElementTypeOrSelf(Value val) {
  return getElementTypeOrSelf(val.getType());
}

Type mlir::getElementTypeOrSelf(Attribute attr) {
  if (auto typedAttr = llvm::dyn_cast<TypedAttr>(attr))
    return getElementTypeOrSelf(typedAttr.getType());
  return {};
}

```
- **EN**: Implements logic around `getElementTypeOrSelf`, `dyn_cast`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getElementTypeOrSelf`、`dyn_cast` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 39-45
```cpp
SmallVector<Type, 10> mlir::getFlattenedTypes(TupleType t) {
  SmallVector<Type, 10> fTypes;
  t.getFlattenedTypes(fTypes);
  return fTypes;
}

/// Return true if the specified type is an opaque type with the specified
```
- **EN**: Implements logic around `getFlattenedTypes`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getFlattenedTypes` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 46-54
```cpp
/// dialect and typeData.
bool mlir::isOpaqueTypeWithName(Type type, StringRef dialect,
                                StringRef typeData) {
  if (auto opaque = llvm::dyn_cast<mlir::OpaqueType>(type))
    return opaque.getDialectNamespace() == dialect &&
           opaque.getTypeData() == typeData;
  return false;
}

```
- **EN**: Implements logic around `isOpaqueTypeWithName`, `OpaqueType>`, `getDialectNamespace`, `getTypeData`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `isOpaqueTypeWithName`、`OpaqueType>`、`getDialectNamespace`、`getTypeData` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 55-68
```cpp
/// Returns success if the given two shapes are compatible. That is, they have
/// the same size and each pair of the elements are equal or one of them is
/// dynamic.
LogicalResult mlir::verifyCompatibleShape(ArrayRef<int64_t> shape1,
                                          ArrayRef<int64_t> shape2) {
  if (shape1.size() != shape2.size())
    return failure();
  for (auto dims : llvm::zip(shape1, shape2)) {
    int64_t dim1 = std::get<0>(dims);
    int64_t dim2 = std::get<1>(dims);
    if (ShapedType::isStatic(dim1) && ShapedType::isStatic(dim2) &&
        dim1 != dim2)
      return failure();
  }
```
- **EN**: Implements logic around `verifyCompatibleShape`, `size`, `failure`, `zip`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `verifyCompatibleShape`、`size`、`failure`、`zip` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 69-75
```cpp
  return success();
}

/// Returns success if the given two types have compatible shape. That is,
/// they are both scalars (not shaped), or they are both shaped types and at
/// least one is unranked or they have compatible dimensions. Dimensions are
/// compatible if at least one is dynamic or both are equal. The element type
```
- **EN**: Implements logic around `success`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `success` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 76-86
```cpp
/// does not matter.
LogicalResult mlir::verifyCompatibleShape(Type type1, Type type2) {
  auto sType1 = llvm::dyn_cast<ShapedType>(type1);
  auto sType2 = llvm::dyn_cast<ShapedType>(type2);

  // Either both or neither type should be shaped.
  if (!sType1)
    return success(!sType2);
  if (!sType2)
    return failure();

```
- **EN**: Implements logic around `verifyCompatibleShape`, `dyn_cast`, `success`, `failure`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `verifyCompatibleShape`、`dyn_cast`、`success`、`failure` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 87-93
```cpp
  if (!sType1.hasRank() || !sType2.hasRank())
    return success();

  return verifyCompatibleShape(sType1.getShape(), sType2.getShape());
}

/// Returns success if the given two arrays have the same number of elements and
```
- **EN**: Implements logic around `hasRank`, `success`, `verifyCompatibleShape`.
- **CN**: 围绕 `hasRank`、`success`、`verifyCompatibleShape` 实现具体逻辑。

### Lines 94-103
```cpp
/// each pair wise entries have compatible shape.
LogicalResult mlir::verifyCompatibleShapes(TypeRange types1, TypeRange types2) {
  if (types1.size() != types2.size())
    return failure();
  for (auto it : llvm::zip_first(types1, types2))
    if (failed(verifyCompatibleShape(std::get<0>(it), std::get<1>(it))))
      return failure();
  return success();
}

```
- **EN**: Implements logic around `verifyCompatibleShapes`, `size`, `failure`, `zip_first`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `verifyCompatibleShapes`、`size`、`failure`、`zip_first` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 104-115
```cpp
LogicalResult mlir::verifyCompatibleDims(ArrayRef<int64_t> dims) {
  if (dims.empty())
    return success();
  auto staticDim =
      llvm::accumulate(dims, dims.front(), [](auto fold, auto dim) {
        return ShapedType::isDynamic(dim) ? fold : dim;
      });
  return success(llvm::all_of(dims, [&](auto dim) {
    return ShapedType::isDynamic(dim) || dim == staticDim;
  }));
}

```
- **EN**: Implements logic around `verifyCompatibleDims`, `empty`, `success`, `accumulate`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; implements transformation or simplification logic.
- **CN**: 围绕 `verifyCompatibleDims`、`empty`、`success`、`accumulate` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并实现变换或简化逻辑。

### Lines 116-128
```cpp
/// Returns success if all given types have compatible shapes. That is, they are
/// all scalars (not shaped), or they are all shaped types and any ranked shapes
/// have compatible dimensions. Dimensions are compatible if all non-dynamic
/// dims are equal. The element type does not matter.
LogicalResult mlir::verifyCompatibleShapes(TypeRange types) {
  auto shapedTypes = llvm::map_to_vector<8>(types, llvm::DynCastTo<ShapedType>);
  // Return failure if some, but not all are not shaped. Return early if none
  // are shaped also.
  if (llvm::none_of(shapedTypes, [](auto t) { return t; }))
    return success();
  if (!llvm::all_of(shapedTypes, [](auto t) { return t; }))
    return failure();

```
- **EN**: Implements logic around `verifyCompatibleShapes`, `map_to_vector`, `none_of`, `success`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `verifyCompatibleShapes`、`map_to_vector`、`none_of`、`success` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 129-141
```cpp
  // Return failure if some, but not all, are scalable vectors.
  bool hasScalableVecTypes = false;
  bool hasNonScalableVecTypes = false;
  for (Type t : types) {
    auto vType = llvm::dyn_cast<VectorType>(t);
    if (vType && vType.isScalable())
      hasScalableVecTypes = true;
    else
      hasNonScalableVecTypes = true;
    if (hasScalableVecTypes && hasNonScalableVecTypes)
      return failure();
  }

```
- **EN**: Implements logic around `dyn_cast`, `isScalable`, `failure`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `dyn_cast`、`isScalable`、`failure` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 142-153
```cpp
  // Remove all unranked shapes
  auto shapes = llvm::filter_to_vector<8>(
      shapedTypes, [](auto shapedType) { return shapedType.hasRank(); });
  if (shapes.empty())
    return success();

  // All ranks should be equal
  auto firstRank = shapes.front().getRank();
  if (llvm::any_of(shapes,
                   [&](auto shape) { return firstRank != shape.getRank(); }))
    return failure();

```
- **EN**: Implements logic around `filter_to_vector`, `hasRank`, `empty`, `success`, and 4 more symbols.
- **CN**: 围绕 `filter_to_vector`、`hasRank`、`empty`、`success` 等另外 4 个符号 实现具体逻辑。

### Lines 154-163
```cpp
  for (unsigned i = 0; i < firstRank; ++i) {
    // Retrieve all ranked dimensions
    auto dims = llvm::map_to_vector<8>(
        llvm::make_filter_range(
            shapes, [&](auto shape) { return shape.getRank() >= i; }),
        [&](auto shape) { return shape.getDimSize(i); });
    if (verifyCompatibleDims(dims).failed())
      return failure();
  }

```
- **EN**: Implements logic around `map_to_vector`, `make_filter_range`, `getRank`, `getDimSize`, and 2 more symbols.
- **CN**: 围绕 `map_to_vector`、`make_filter_range`、`getRank`、`getDimSize` 等另外 2 个符号 实现具体逻辑。

### Lines 164-170
```cpp
  return success();
}

Type OperandElementTypeIterator::mapElement(Value value) const {
  return llvm::cast<ShapedType>(value.getType()).getElementType();
}

```
- **EN**: Implements logic around `success`, `mapElement`, `cast`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `success`、`mapElement`、`cast` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 171-182
```cpp
Type ResultElementTypeIterator::mapElement(Value value) const {
  return llvm::cast<ShapedType>(value.getType()).getElementType();
}

TypeRange mlir::insertTypesInto(TypeRange oldTypes, ArrayRef<unsigned> indices,
                                TypeRange newTypes,
                                SmallVectorImpl<Type> &storage) {
  assert(indices.size() == newTypes.size() &&
         "mismatch between indice and type count");
  if (indices.empty())
    return oldTypes;

```
- **EN**: Implements logic around `mapElement`, `cast`, `insertTypesInto`, `assert`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `mapElement`、`cast`、`insertTypesInto`、`assert` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 183-193
```cpp
  auto fromIt = oldTypes.begin();
  for (auto it : llvm::zip(indices, newTypes)) {
    const auto toIt = oldTypes.begin() + std::get<0>(it);
    storage.append(fromIt, toIt);
    storage.push_back(std::get<1>(it));
    fromIt = toIt;
  }
  storage.append(fromIt, oldTypes.end());
  return storage;
}

```
- **EN**: Implements logic around `begin`, `zip`, `append`, `push_back`.
- **CN**: 围绕 `begin`、`zip`、`append`、`push_back` 实现具体逻辑。

### Lines 194-203
```cpp
TypeRange mlir::filterTypesOut(TypeRange types, const BitVector &indices,
                               SmallVectorImpl<Type> &storage) {
  if (indices.none())
    return types;

  for (unsigned i = 0, e = types.size(); i < e; ++i)
    if (!indices[i])
      storage.emplace_back(types[i]);
  return storage;
}
```
- **EN**: Implements logic around `filterTypesOut`, `none`, `size`, `emplace_back`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `filterTypesOut`、`none`、`size`、`emplace_back` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

## Key Concepts / 关键概念

- **Core IR modeling / 核心 IR 建模**:
  - **EN**: Defines operations, regions, blocks, attributes, types, and the assembly form that ties them together.
  - **CN**: 定义操作、区域、块、属性、类型，以及将它们串联起来的汇编表示。
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

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/IR/TypeUtilities.h`, `mlir/IR/Attributes.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Types.h`, `mlir/IR/Value.h`, `llvm/ADT/SmallVectorExtras.h`
- **Standard-library headers / 标准库头文件**: `<numeric>`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (5), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1)
