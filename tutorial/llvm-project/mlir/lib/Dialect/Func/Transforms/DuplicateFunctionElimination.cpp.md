# DuplicateFunctionElimination.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Func/Transforms/DuplicateFunctionElimination.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the Func dialect and function-like IR support.
  - **CN**: 实现 Func 方言与类函数 IR 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- DuplicateFunctionElimination.cpp - Duplicate function elimination --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-12
```cpp

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Func/Transforms/Passes.h"

namespace mlir {
```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 13-17
```cpp
namespace func {
#define GEN_PASS_DEF_DUPLICATEFUNCTIONELIMINATIONPASS
#include "mlir/Dialect/Func/Transforms/Passes.h.inc"
} // namespace func

```
- **EN**: Introduces declarations for `func`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `func` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 18-22
```cpp
namespace {

// Define a notion of function equivalence that allows for reuse. Ignore the
// symbol name for this purpose.
struct DuplicateFuncOpEquivalenceInfo
```
- **EN**: Introduces declarations for `DuplicateFuncOpEquivalenceInfo`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DuplicateFuncOpEquivalenceInfo` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 23-29
```cpp
    : public llvm::DenseMapInfo<func::FuncOp> {

  static unsigned getHashValue(const func::FuncOp cFunc) {
    if (!cFunc) {
      return DenseMapInfo<func::FuncOp>::getHashValue(cFunc);
    }

```
- **EN**: Implements logic around `getHashValue`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getHashValue` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 30-39
```cpp
    // Aggregate attributes, ignoring the symbol name.
    llvm::hash_code hash = {};
    func::FuncOp func = const_cast<func::FuncOp &>(cFunc);
    StringAttr symNameAttrName = func.getSymNameAttrName();
    for (NamedAttribute namedAttr : cFunc->getAttrs()) {
      StringAttr attrName = namedAttr.getName();
      if (attrName == symNameAttrName)
        continue;
      hash = llvm::hash_combine(hash, namedAttr);
    }
```
- **EN**: Implements logic around `getSymNameAttrName`, `getAttrs`, `getName`, `hash_combine`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getSymNameAttrName`, `getAttrs`, `getName`, `hash_combine` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 40-49
```cpp

    // Also hash the func body.
    func.getBody().walk([&](Operation *op) {
      hash = llvm::hash_combine(
          hash, OperationEquivalence::computeHash(
                    op, /*hashOperands=*/OperationEquivalence::ignoreHashValue,
                    /*hashResults=*/OperationEquivalence::ignoreHashValue,
                    OperationEquivalence::IgnoreLocations));
    });

```
- **EN**: Implements logic around `getBody`, `hash_combine`, `computeHash`.
- **CN**: 围绕 `getBody`, `hash_combine`, `computeHash` 实现具体逻辑。

### Lines 50-59
```cpp
    return hash;
  }

  static bool isEqual(func::FuncOp lhs, func::FuncOp rhs) {
    if (lhs == rhs)
      return true;
    if (lhs == getTombstoneKey() || lhs == getEmptyKey() ||
        rhs == getTombstoneKey() || rhs == getEmptyKey())
      return false;

```
- **EN**: Implements logic around `isEqual`, `getTombstoneKey`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `isEqual`, `getTombstoneKey` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 60-67
```cpp
    if (lhs.isDeclaration() || rhs.isDeclaration())
      return false;

    // Check discardable attributes equivalence
    if (lhs->getDiscardableAttrDictionary() !=
        rhs->getDiscardableAttrDictionary())
      return false;

```
- **EN**: Implements logic around `isDeclaration`, `getDiscardableAttrDictionary`.
- **CN**: 围绕 `isDeclaration`, `getDiscardableAttrDictionary` 实现具体逻辑。

### Lines 68-77
```cpp
    // Check properties equivalence, ignoring the symbol name.
    // Make a copy, so that we can erase the symbol name and perform the
    // comparison.
    auto pLhs = lhs.getProperties();
    auto pRhs = rhs.getProperties();
    pLhs.sym_name = nullptr;
    pRhs.sym_name = nullptr;
    if (pLhs != pRhs)
      return false;

```
- **EN**: Implements logic around `getProperties`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getProperties` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 78-83
```cpp
    // Compare inner workings.
    return OperationEquivalence::isRegionEquivalentTo(
        &lhs.getBody(), &rhs.getBody(), OperationEquivalence::IgnoreLocations);
  }
};

```
- **EN**: Implements logic around `isRegionEquivalentTo`, `getBody`.
- **CN**: 围绕 `isRegionEquivalentTo`, `getBody` 实现具体逻辑。

### Lines 84-90
```cpp
struct DuplicateFunctionEliminationPass
    : public func::impl::DuplicateFunctionEliminationPassBase<
          DuplicateFunctionEliminationPass> {

  using DuplicateFunctionEliminationPassBase<
      DuplicateFunctionEliminationPass>::DuplicateFunctionEliminationPassBase;

```
- **EN**: Introduces declarations for `DuplicateFunctionEliminationPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DuplicateFunctionEliminationPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 91-100
```cpp
  void runOnOperation() override {
    auto module = getOperation();

    // Find unique representant per equivalent func ops.
    DenseSet<func::FuncOp, DuplicateFuncOpEquivalenceInfo> uniqueFuncOps;
    DenseMap<StringAttr, func::FuncOp> getRepresentant;
    DenseSet<func::FuncOp> toBeErased;
    module.walk([&](func::FuncOp f) {
      auto [repr, inserted] = uniqueFuncOps.insert(f);
      getRepresentant[f.getSymNameAttr()] = *repr;
```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `walk`, `insert`, and 1 more symbols; this block participates in pass execution or pass construction; works with symbol tables or function-like operations.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `walk`, `insert`, and 1 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并处理符号表或类函数操作。

### Lines 101-105
```cpp
      if (!inserted) {
        toBeErased.insert(f);
      }
    });

```
- **EN**: Implements logic around `insert`.
- **CN**: 围绕 `insert` 实现具体逻辑。

### Lines 106-115
```cpp
    // Update all symbol uses to reference unique func op
    // representants and erase redundant func ops.
    SymbolTableCollection symbolTable;
    SymbolUserMap userMap(symbolTable, module);
    for (auto it : toBeErased) {
      StringAttr oldSymbol = it.getSymNameAttr();
      StringAttr newSymbol = getRepresentant[oldSymbol].getSymNameAttr();
      userMap.replaceAllUsesWith(it, newSymbol);
      it.erase();
    }
```
- **EN**: Implements logic around `userMap`, `getSymNameAttr`, `replaceAllUsesWith`, `erase`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `userMap`, `getSymNameAttr`, `replaceAllUsesWith`, `erase` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 116-120
```cpp
  }
};

} // namespace
} // namespace mlir
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Function-like regions / 类函数 Region**:
  - **EN**: Provides function operations, callable interfaces, and symbol-aware bodies.
  - **CN**: 提供函数操作、可调用接口以及符号感知的函数体。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Func/Transforms/Passes.h`, `mlir/Dialect/Func/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (3)
