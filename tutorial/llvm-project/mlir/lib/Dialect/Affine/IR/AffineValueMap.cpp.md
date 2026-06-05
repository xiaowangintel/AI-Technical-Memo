# AffineValueMap.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/IR/AffineValueMap.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the core IR, operations, types, attributes, or interfaces for the Affine dialect and affine-loop reasoning.
  - **CN**: 实现 Affine 方言与仿射循环推理 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineValueMap.cpp - MLIR Affine Value Map Class -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp

#include "mlir/Dialect/Affine/IR/AffineValueMap.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"

using namespace mlir;
using namespace mlir::affine;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineValueMap.h`, `mlir/Dialect/Affine/IR/AffineOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineValueMap.h`, `mlir/Dialect/Affine/IR/AffineOps.h`。

### Lines 15-19
```cpp
AffineValueMap::AffineValueMap(AffineMap map, ValueRange operands,
                               ValueRange results)
    : map(map), operands(operands.begin(), operands.end()),
      results(results.begin(), results.end()) {}

```
- **EN**: Implements logic around `AffineValueMap`, `map`, `results`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `AffineValueMap`, `map`, `results` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 20-26
```cpp
void AffineValueMap::reset(AffineMap map, ValueRange operands,
                           ValueRange results) {
  this->map.reset(map);
  this->operands.assign(operands.begin(), operands.end());
  this->results.assign(results.begin(), results.end());
}

```
- **EN**: Implements logic around `reset`, `assign`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `reset`, `assign` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 27-35
```cpp
void AffineValueMap::composeSimplifyAndCanonicalize() {
  AffineMap sMap = getAffineMap();
  fullyComposeAffineMapAndOperands(&sMap, &operands);
  // Full composition also canonicalizes and simplifies before returning. We
  // need to canonicalize once more to drop unused operands.
  canonicalizeMapAndOperands(&sMap, &operands);
  this->map.reset(sMap);
}

```
- **EN**: Implements logic around `composeSimplifyAndCanonicalize`, `getAffineMap`, `fullyComposeAffineMapAndOperands`, `canonicalizeMapAndOperands`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `composeSimplifyAndCanonicalize`, `getAffineMap`, `fullyComposeAffineMapAndOperands`, `canonicalizeMapAndOperands`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 36-45
```cpp
void AffineValueMap::difference(const AffineValueMap &a,
                                const AffineValueMap &b, AffineValueMap *res) {
  assert(a.getNumResults() == b.getNumResults() && "invalid inputs");

  SmallVector<Value, 4> allOperands;
  allOperands.reserve(a.getNumOperands() + b.getNumOperands());
  auto aDims = a.getOperands().take_front(a.getNumDims());
  auto bDims = b.getOperands().take_front(b.getNumDims());
  auto aSyms = a.getOperands().take_back(a.getNumSymbols());
  auto bSyms = b.getOperands().take_back(b.getNumSymbols());
```
- **EN**: Implements logic around `difference`, `assert`, `reserve`, `getOperands`.
- **CN**: 围绕 `difference`, `assert`, `reserve`, `getOperands` 实现具体逻辑。

### Lines 46-50
```cpp
  allOperands.append(aDims.begin(), aDims.end());
  allOperands.append(bDims.begin(), bDims.end());
  allOperands.append(aSyms.begin(), aSyms.end());
  allOperands.append(bSyms.begin(), bSyms.end());

```
- **EN**: Implements logic around `append`.
- **CN**: 围绕 `append` 实现具体逻辑。

### Lines 51-55
```cpp
  // Shift dims and symbols of b's map.
  auto bMap = b.getAffineMap()
                  .shiftDims(a.getNumDims())
                  .shiftSymbols(a.getNumSymbols());

```
- **EN**: Implements logic around `getAffineMap`, `shiftDims`, `shiftSymbols`.
- **CN**: 围绕 `getAffineMap`, `shiftDims`, `shiftSymbols` 实现具体逻辑。

### Lines 56-62
```cpp
  // Construct the difference expressions.
  auto aMap = a.getAffineMap();
  SmallVector<AffineExpr, 4> diffExprs;
  diffExprs.reserve(a.getNumResults());
  for (unsigned i = 0, e = bMap.getNumResults(); i < e; ++i)
    diffExprs.push_back(aMap.getResult(i) - bMap.getResult(i));

```
- **EN**: Implements logic around `getAffineMap`, `reserve`, `getNumResults`, `push_back`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getAffineMap`, `reserve`, `getNumResults`, `push_back` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 63-70
```cpp
  auto diffMap = AffineMap::get(bMap.getNumDims(), bMap.getNumSymbols(),
                                diffExprs, bMap.getContext());
  fullyComposeAffineMapAndOperands(&diffMap, &allOperands);
  canonicalizeMapAndOperands(&diffMap, &allOperands);
  diffMap = simplifyAffineMap(diffMap);
  res->reset(diffMap, allOperands);
}

```
- **EN**: Implements logic around `get`, `getContext`, `fullyComposeAffineMapAndOperands`, `canonicalizeMapAndOperands`, and 2 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `get`, `getContext`, `fullyComposeAffineMapAndOperands`, `canonicalizeMapAndOperands`, and 2 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 71-80
```cpp
// Returns true and sets 'indexOfMatch' if 'valueToMatch' is found in
// 'valuesToSearch' beginning at 'indexStart'. Returns false otherwise.
static bool findIndex(Value valueToMatch, ArrayRef<Value> valuesToSearch,
                      unsigned indexStart, unsigned *indexOfMatch) {
  unsigned size = valuesToSearch.size();
  for (unsigned i = indexStart; i < size; ++i) {
    if (valueToMatch == valuesToSearch[i]) {
      *indexOfMatch = i;
      return true;
    }
```
- **EN**: Implements logic around `findIndex`, `size`.
- **CN**: 围绕 `findIndex`, `size` 实现具体逻辑。

### Lines 81-88
```cpp
  }
  return false;
}

bool AffineValueMap::isMultipleOf(unsigned idx, int64_t factor) const {
  return map.isMultipleOf(idx, factor);
}

```
- **EN**: Implements logic around `isMultipleOf`.
- **CN**: 围绕 `isMultipleOf` 实现具体逻辑。

### Lines 89-98
```cpp
/// This method uses the invariant that operands are always positionally aligned
/// with the AffineDimExpr in the underlying AffineMap.
bool AffineValueMap::isFunctionOf(unsigned idx, Value value) const {
  unsigned index;
  if (!findIndex(value, operands, /*indexStart=*/0, &index)) {
    return false;
  }
  auto expr = const_cast<AffineValueMap *>(this)->getAffineMap().getResult(idx);
  // TODO: this is better implemented on a flattened representation.
  // At least for now it is conservative.
```
- **EN**: Implements logic around `isFunctionOf`, `findIndex`, `getAffineMap`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `isFunctionOf`, `findIndex`, `getAffineMap` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 99-105
```cpp
  return expr.isFunctionOfDim(index);
}

Value AffineValueMap::getOperand(unsigned i) const {
  return static_cast<Value>(operands[i]);
}

```
- **EN**: Implements logic around `isFunctionOfDim`, `getOperand`, `static_cast`.
- **CN**: 围绕 `isFunctionOfDim`, `getOperand`, `static_cast` 实现具体逻辑。

### Lines 106-111
```cpp
ArrayRef<Value> AffineValueMap::getOperands() const {
  return ArrayRef<Value>(operands);
}

AffineMap AffineValueMap::getAffineMap() const { return map.getAffineMap(); }

```
- **EN**: Implements logic around `getOperands`, `ArrayRef`, `getAffineMap`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getOperands`, `ArrayRef`, `getAffineMap` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 112-119
```cpp
bool AffineValueMap::operator==(const AffineValueMap &other) const {
  AffineValueMap diff;
  AffineValueMap::difference(*this, other, &diff);
  return llvm::all_of(diff.getAffineMap().getResults(), [](AffineExpr e) {
    return e == getAffineConstantExpr(0, e.getContext());
  });
}

```
- **EN**: Implements logic around `difference`, `all_of`, `getAffineConstantExpr`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `difference`, `all_of`, `getAffineConstantExpr` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 120-120
```cpp
AffineValueMap::~AffineValueMap() = default;
```
- **EN**: Implements logic around `~AffineValueMap`.
- **CN**: 围绕 `~AffineValueMap` 实现具体逻辑。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineValueMap.h`, `mlir/Dialect/Affine/IR/AffineOps.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (2)
