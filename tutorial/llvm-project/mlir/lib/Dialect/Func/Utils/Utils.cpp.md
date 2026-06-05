# Utils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Func/Utils/Utils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements utilities for the Func dialect.
  - **CN**: 实现 Func 方言与类函数 IR 支持 使用的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Utils.cpp - Utilities to support the Func dialect ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-20
```cpp
//
// This file implements utilities for the Func dialect.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Func/Utils/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/PatternMatch.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/DebugLog.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Func/Utils/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Func/Utils/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`。

### Lines 21-27
```cpp
#define DEBUG_TYPE "func-utils"

using namespace mlir;

/// This method creates an inverse mapping of the provided map `oldToNew`.
/// Given an array where `oldIdxToNewIdx[i] = j` means old index `i` maps
/// to new index `j`,
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or generated-code switches.
- **CN**: 定义受预处理器控制的结构、特性开关或生成代码开关。

### Lines 28-34
```cpp
/// This method returns a vector where `result[j]` contains all old indices
/// that map to new index `j`.
///
/// Example:
/// ```
/// oldIdxToNewIdx = [0, 1, 2, 2, 3]
/// getInverseMapping(oldIdxToNewIdx) = [[0], [1], [2, 3], [4]]
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 35-47
```cpp
/// ```
///
static llvm::SmallVector<llvm::SmallVector<int>>
getInverseMapping(ArrayRef<int> oldIdxToNewIdx) {
  int numOfNewIdxs = 0;
  if (!oldIdxToNewIdx.empty())
    numOfNewIdxs = 1 + *llvm::max_element(oldIdxToNewIdx);
  llvm::SmallVector<llvm::SmallVector<int>> newToOldIdxs(numOfNewIdxs);
  for (auto [oldIdx, newIdx] : llvm::enumerate(oldIdxToNewIdx))
    newToOldIdxs[newIdx].push_back(oldIdx);
  return newToOldIdxs;
}

```
- **EN**: Implements logic around `getInverseMapping`, `empty`, `max_element`, `newToOldIdxs`, and 2 more symbols.
- **CN**: 围绕 `getInverseMapping`, `empty`, `max_element`, `newToOldIdxs`, and 2 more symbols 实现具体逻辑。

### Lines 48-61
```cpp
/// This method returns a new vector of elements that are mapped from the
/// `origElements` based on the `newIdxToOldIdxs` mapping. This function assumes
/// that the `newIdxToOldIdxs` mapping is valid, i.e. for each new index, there
/// is at least one old index that maps to it. Also, It assumes that mapping to
/// the same old index has the same element in the `origElements` vector.
template <typename Element>
static SmallVector<Element> getMappedElements(
    ArrayRef<Element> origElements,
    const llvm::SmallVector<llvm::SmallVector<int>> &newIdxToOldIdxs) {
  SmallVector<Element> newElements;
  for (const auto &oldIdxs : newIdxToOldIdxs) {
    assert(llvm::all_of(oldIdxs,
                        [&origElements](int idx) -> bool {
                          return idx >= 0 &&
```
- **EN**: Implements logic around `getMappedElements`, `assert`.
- **CN**: 围绕 `getMappedElements`, `assert` 实现具体逻辑。

### Lines 62-75
```cpp
                                 static_cast<size_t>(idx) < origElements.size();
                        }) &&
           "idx must be less than the number of elements in the original "
           "elements");
    assert(!oldIdxs.empty() && "oldIdx must not be empty");
    Element origTypeToCheck = origElements[oldIdxs.front()];
    assert(llvm::all_of(oldIdxs,
                        [&](int idx) -> bool {
                          return origElements[idx] == origTypeToCheck;
                        }) &&
           "all oldIdxs must be equal");
    newElements.push_back(origTypeToCheck);
  }
  return newElements;
```
- **EN**: Implements logic around `static_cast`, `assert`, `front`, `push_back`.
- **CN**: 围绕 `static_cast`, `assert`, `front`, `push_back` 实现具体逻辑。

### Lines 76-89
```cpp
}

FailureOr<func::FuncOp>
func::replaceFuncWithNewMapping(RewriterBase &rewriter, func::FuncOp funcOp,
                                ArrayRef<int> oldArgIdxToNewArgIdx,
                                ArrayRef<int> oldResIdxToNewResIdx) {
  // Generate an empty new function operation with the same name as the
  // original.
  assert(funcOp.getNumArguments() == oldArgIdxToNewArgIdx.size() &&
         "oldArgIdxToNewArgIdx must match the number of arguments in the "
         "function");
  assert(
      funcOp.getNumResults() == oldResIdxToNewResIdx.size() &&
      "oldResIdxToNewResIdx must match the number of results in the function");
```
- **EN**: Implements logic around `replaceFuncWithNewMapping`, `assert`, `getNumResults`; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `replaceFuncWithNewMapping`, `assert`, `getNumResults` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 90-102
```cpp

  if (!funcOp.getBody().hasOneBlock())
    return rewriter.notifyMatchFailure(
        funcOp, "expected function to have exactly one block");

  // We may have some duplicate arguments in the old function, i.e.
  // in the mapping `newArgIdxToOldArgIdxs` for some new argument index
  // there may be multiple old argument indices.
  llvm::SmallVector<llvm::SmallVector<int>> newArgIdxToOldArgIdxs =
      getInverseMapping(oldArgIdxToNewArgIdx);
  SmallVector<Type> newInputTypes = getMappedElements(
      funcOp.getFunctionType().getInputs(), newArgIdxToOldArgIdxs);

```
- **EN**: Implements logic around `getBody`, `notifyMatchFailure`, `getInverseMapping`, `getMappedElements`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `getBody`, `notifyMatchFailure`, `getInverseMapping`, `getMappedElements`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 103-111
```cpp
  SmallVector<Location> locs;
  for (const auto &oldArgIdxs : newArgIdxToOldArgIdxs)
    locs.push_back(funcOp.getArgument(oldArgIdxs.front()).getLoc());

  llvm::SmallVector<llvm::SmallVector<int>> newResToOldResIdxs =
      getInverseMapping(oldResIdxToNewResIdx);
  SmallVector<Type> newOutputTypes = getMappedElements(
      funcOp.getFunctionType().getResults(), newResToOldResIdxs);

```
- **EN**: Implements logic around `push_back`, `getInverseMapping`, `getMappedElements`, `getFunctionType`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `push_back`, `getInverseMapping`, `getMappedElements`, `getFunctionType` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 112-120
```cpp
  rewriter.setInsertionPoint(funcOp);
  auto newFuncOp = func::FuncOp::create(
      rewriter, funcOp.getLoc(), funcOp.getName(),
      rewriter.getFunctionType(newInputTypes, newOutputTypes));

  Region &newRegion = newFuncOp.getBody();
  rewriter.createBlock(&newRegion, newRegion.begin(), newInputTypes, locs);
  newFuncOp.setVisibility(funcOp.getVisibility());

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `getLoc`, `getFunctionType`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `setInsertionPoint`, `create`, `getLoc`, `getFunctionType`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 121-131
```cpp
  // Map the arguments of the original function to the new function in
  // the new order and adjust the attributes accordingly.
  IRMapping operandMapper;
  SmallVector<DictionaryAttr> argAttrs, resultAttrs;
  funcOp.getAllArgAttrs(argAttrs);
  for (auto [oldArgIdx, newArgIdx] : llvm::enumerate(oldArgIdxToNewArgIdx))
    operandMapper.map(funcOp.getArgument(oldArgIdx),
                      newFuncOp.getArgument(newArgIdx));
  for (auto [newArgIdx, oldArgIdx] : llvm::enumerate(newArgIdxToOldArgIdxs))
    newFuncOp.setArgAttrs(newArgIdx, argAttrs[oldArgIdx.front()]);

```
- **EN**: Implements logic around `getAllArgAttrs`, `enumerate`, `map`, `getArgument`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getAllArgAttrs`, `enumerate`, `map`, `getArgument`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 132-140
```cpp
  funcOp.getAllResultAttrs(resultAttrs);
  for (auto [newResIdx, oldResIdx] : llvm::enumerate(newResToOldResIdxs))
    newFuncOp.setResultAttrs(newResIdx, resultAttrs[oldResIdx.front()]);

  // Clone the operations from the original function to the new function.
  rewriter.setInsertionPointToStart(&newFuncOp.getBody().front());
  for (Operation &op : funcOp.getOps())
    rewriter.clone(op, operandMapper);

```
- **EN**: Implements logic around `getAllResultAttrs`, `enumerate`, `setResultAttrs`, `setInsertionPointToStart`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `getAllResultAttrs`, `enumerate`, `setResultAttrs`, `setInsertionPointToStart`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 141-147
```cpp
  // Handle the return operation.
  auto returnOp = cast<func::ReturnOp>(
      newFuncOp.getFunctionBody().begin()->getTerminator());
  SmallVector<Value> newReturnValues;
  for (const auto &oldResIdxs : newResToOldResIdxs)
    newReturnValues.push_back(returnOp.getOperand(oldResIdxs.front()));

```
- **EN**: Implements logic around `ReturnOp>`, `getFunctionBody`, `push_back`.
- **CN**: 围绕 `ReturnOp>`, `getFunctionBody`, `push_back` 实现具体逻辑。

### Lines 148-156
```cpp
  rewriter.setInsertionPoint(returnOp);
  func::ReturnOp::create(rewriter, newFuncOp.getLoc(), newReturnValues);
  rewriter.eraseOp(returnOp);

  rewriter.eraseOp(funcOp);

  return newFuncOp;
}

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `eraseOp`; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `setInsertionPoint`, `create`, `eraseOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 157-167
```cpp
func::CallOp
func::replaceCallOpWithNewMapping(RewriterBase &rewriter, func::CallOp callOp,
                                  ArrayRef<int> oldArgIdxToNewArgIdx,
                                  ArrayRef<int> oldResIdxToNewResIdx) {
  assert(callOp.getNumOperands() == oldArgIdxToNewArgIdx.size() &&
         "oldArgIdxToNewArgIdx must match the number of operands in the call "
         "operation");
  assert(callOp.getNumResults() == oldResIdxToNewResIdx.size() &&
         "oldResIdxToNewResIdx must match the number of results in the call "
         "operation");

```
- **EN**: Implements logic around `replaceCallOpWithNewMapping`, `assert`; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `replaceCallOpWithNewMapping`, `assert` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 168-178
```cpp
  SmallVector<Value> origOperands = callOp.getOperands();
  SmallVector<llvm::SmallVector<int>> newArgIdxToOldArgIdxs =
      getInverseMapping(oldArgIdxToNewArgIdx);
  SmallVector<Value> newOperandsValues =
      getMappedElements<Value>(origOperands, newArgIdxToOldArgIdxs);
  SmallVector<llvm::SmallVector<int>> newResToOldResIdxs =
      getInverseMapping(oldResIdxToNewResIdx);
  SmallVector<Type> origResultTypes = llvm::to_vector(callOp.getResultTypes());
  SmallVector<Type> newResultTypes =
      getMappedElements<Type>(origResultTypes, newResToOldResIdxs);

```
- **EN**: Implements logic around `getOperands`, `getInverseMapping`, `getMappedElements`, `to_vector`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `getOperands`, `getInverseMapping`, `getMappedElements`, `to_vector` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 179-190
```cpp
  // Replace the kernel call operation with a new one that has the
  // mapped arguments.
  rewriter.setInsertionPoint(callOp);
  auto newCallOp =
      func::CallOp::create(rewriter, callOp.getLoc(), callOp.getCallee(),
                           newResultTypes, newOperandsValues);
  newCallOp.setNoInlineAttr(callOp.getNoInlineAttr());
  for (auto &&[oldResIdx, newResIdx] : llvm::enumerate(oldResIdxToNewResIdx))
    rewriter.replaceAllUsesWith(callOp.getResult(oldResIdx),
                                newCallOp.getResult(newResIdx));
  rewriter.eraseOp(callOp);

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `setNoInlineAttr`, `enumerate`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `setInsertionPoint`, `create`, `setNoInlineAttr`, `enumerate`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 191-204
```cpp
  return newCallOp;
}

FailureOr<std::pair<func::FuncOp, func::CallOp>>
func::deduplicateArgsOfFuncOp(RewriterBase &rewriter, func::FuncOp funcOp,
                              ModuleOp moduleOp) {
  SmallVector<func::CallOp> callOps;
  auto traversalResult = moduleOp.walk([&](func::CallOp callOp) {
    if (callOp.getCallee() == funcOp.getSymName()) {
      if (!callOps.empty())
        // Only support one callOp for now
        return WalkResult::interrupt();
      callOps.push_back(callOp);
    }
```
- **EN**: Implements logic around `deduplicateArgsOfFuncOp`, `walk`, `getCallee`, `empty`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `deduplicateArgsOfFuncOp`, `walk`, `getCallee`, `empty`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 205-212
```cpp
    return WalkResult::advance();
  });

  if (traversalResult.wasInterrupted()) {
    LDBG() << "function " << funcOp.getName() << " has more than one callOp";
    return failure();
  }

```
- **EN**: Implements logic around `advance`, `wasInterrupted`, `LDBG`, `failure`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `advance`, `wasInterrupted`, `LDBG`, `failure` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 213-219
```cpp
  if (callOps.empty()) {
    LDBG() << "function " << funcOp.getName() << " does not have any callOp";
    return failure();
  }

  func::CallOp callOp = callOps.front();

```
- **EN**: Implements logic around `empty`, `LDBG`, `failure`, `front`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `empty`, `LDBG`, `failure`, `front` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 220-229
```cpp
  // Create mapping for arguments (deduplicate operands)
  SmallVector<int> oldArgIdxToNewArgIdx(callOp.getNumOperands());
  llvm::DenseMap<Value, int> valueToNewArgIdx;
  for (auto [operandIdx, operand] : llvm::enumerate(callOp.getOperands())) {
    auto [iterator, inserted] = valueToNewArgIdx.insert(
        {operand, static_cast<int>(valueToNewArgIdx.size())});
    // Reduce the duplicate operands and maintain the original order.
    oldArgIdxToNewArgIdx[operandIdx] = iterator->second;
  }

```
- **EN**: Implements logic around `oldArgIdxToNewArgIdx`, `enumerate`, `insert`, `static_cast`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `oldArgIdxToNewArgIdx`, `enumerate`, `insert`, `static_cast` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 230-237
```cpp
  bool hasDuplicateOperands =
      valueToNewArgIdx.size() != callOp.getNumOperands();
  if (!hasDuplicateOperands) {
    LDBG() << "function " << funcOp.getName()
           << " does not have duplicate operands";
    return failure();
  }

```
- **EN**: Implements logic around `size`, `LDBG`, `failure`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `size`, `LDBG`, `failure` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 238-251
```cpp
  // Create identity mapping for results (no deduplication needed)
  SmallVector<int> oldResIdxToNewResIdx(callOp.getNumResults());
  for (int resultIdx : llvm::seq<int>(0, callOp.getNumResults()))
    oldResIdxToNewResIdx[resultIdx] = resultIdx;

  // Apply the transformation to create new function and call operations
  FailureOr<func::FuncOp> newFuncOpOrFailure = replaceFuncWithNewMapping(
      rewriter, funcOp, oldArgIdxToNewArgIdx, oldResIdxToNewResIdx);
  if (failed(newFuncOpOrFailure)) {
    LDBG() << "failed to replace function signature with name "
           << funcOp.getName() << " with new order";
    return failure();
  }

```
- **EN**: Implements logic around `oldResIdxToNewResIdx`, `seq`, `replaceFuncWithNewMapping`, `failed`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `oldResIdxToNewResIdx`, `seq`, `replaceFuncWithNewMapping`, `failed`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 252-265
```cpp
  func::CallOp newCallOp = replaceCallOpWithNewMapping(
      rewriter, callOp, oldArgIdxToNewArgIdx, oldResIdxToNewResIdx);

  return std::make_pair(*newFuncOpOrFailure, newCallOp);
}

FailureOr<func::FuncOp>
func::lookupFnDecl(SymbolOpInterface symTable, StringRef name,
                   FunctionType funcT, SymbolTableCollection *symbolTables) {
  FuncOp func;
  if (symbolTables) {
    func = symbolTables->lookupSymbolIn<FuncOp>(
        symTable, StringAttr::get(symTable->getContext(), name));
  } else {
```
- **EN**: Implements logic around `replaceCallOpWithNewMapping`, `make_pair`, `lookupFnDecl`, `lookupSymbolIn`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `replaceCallOpWithNewMapping`, `make_pair`, `lookupFnDecl`, `lookupSymbolIn`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 266-272
```cpp
    func = llvm::dyn_cast_or_null<FuncOp>(
        SymbolTable::lookupSymbolIn(symTable, name));
  }

  if (!func)
    return func;

```
- **EN**: Implements logic around `dyn_cast_or_null`, `lookupSymbolIn`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `dyn_cast_or_null`, `lookupSymbolIn` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 273-282
```cpp
  mlir::FunctionType foundFuncT = func.getFunctionType();
  // Assert the signature of the found function is same as expected
  if (funcT != foundFuncT) {
    return func.emitError("matched function '")
           << name << "' but with different type: " << foundFuncT
           << " (expected " << funcT << ")";
  }
  return func;
}

```
- **EN**: Implements logic around `getFunctionType`, `emitError`.
- **CN**: 围绕 `getFunctionType`, `emitError` 实现具体逻辑。

### Lines 283-296
```cpp
func::FuncOp func::createFnDecl(OpBuilder &b, SymbolOpInterface symTable,
                                StringRef name, FunctionType funcT,
                                bool setPrivate,
                                SymbolTableCollection *symbolTables) {
  OpBuilder::InsertionGuard g(b);
  assert(!symTable->getRegion(0).empty() && "expected non-empty region");
  b.setInsertionPointToStart(&symTable->getRegion(0).front());
  func::FuncOp funcOp =
      func::FuncOp::create(b, symTable->getLoc(), name, funcT);
  if (setPrivate)
    funcOp.setPrivate();
  if (symbolTables) {
    SymbolTable &symbolTable = symbolTables->getSymbolTable(symTable);
    symbolTable.insert(funcOp, symTable->getRegion(0).front().begin());
```
- **EN**: Implements logic around `createFnDecl`, `g`, `assert`, `setInsertionPointToStart`, and 4 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `createFnDecl`, `g`, `assert`, `setInsertionPointToStart`, and 4 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 297-310
```cpp
  }
  return funcOp;
}

FailureOr<func::FuncOp>
func::lookupOrCreateFnDecl(OpBuilder &b, SymbolOpInterface symTable,
                           StringRef funcName, TypeRange paramTypes,
                           SymbolTableCollection *symbolTables,
                           Type resultType) {
  if (!resultType)
    resultType = IntegerType::get(symTable->getContext(), 64);
  auto funcT = FunctionType::get(b.getContext(), paramTypes, {resultType});
  FailureOr<func::FuncOp> func =
      lookupFnDecl(symTable, funcName, funcT, symbolTables);
```
- **EN**: Implements logic around `lookupOrCreateFnDecl`, `get`, `lookupFnDecl`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `lookupOrCreateFnDecl`, `get`, `lookupFnDecl` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 311-317
```cpp
  // Failed due to type mismatch.
  if (failed(func))
    return func;
  // Successfully matched existing decl.
  if (*func)
    return *func;

```
- **EN**: Implements logic around `failed`.
- **CN**: 围绕 `failed` 实现具体逻辑。

### Lines 318-320
```cpp
  return createFnDecl(b, symTable, funcName, funcT,
                      /*setPrivate=*/true, symbolTables);
}
```
- **EN**: Implements logic around `createFnDecl`.
- **CN**: 围绕 `createFnDecl` 实现具体逻辑。

## Key Concepts / 关键概念

- **Utility helpers / 辅助工具**:
  - **EN**: Provides reusable helpers that keep dialect implementations and passes smaller.
  - **CN**: 提供可复用的辅助函数，使方言实现和 pass 保持精简。
- **Function-like regions / 类函数 Region**:
  - **EN**: Provides function operations, callable interfaces, and symbol-aware bodies.
  - **CN**: 提供函数操作、可调用接口以及符号感知的函数体。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Func/Utils/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/Support/DebugLog.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (2), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
