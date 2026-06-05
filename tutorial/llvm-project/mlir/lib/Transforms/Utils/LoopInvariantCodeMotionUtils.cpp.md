# LoopInvariantCodeMotionUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Transforms/Utils/LoopInvariantCodeMotionUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains the implementation of the core LICM algorithm.
  - **CN**: 实现核心 MLIR 变换 Pass 与变换辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- LoopInvariantCodeMotionUtils.cpp - LICM Utils ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the implementation of the core LICM algorithm.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 12-24
```cpp

#include "mlir/Transforms/LoopInvariantCodeMotionUtils.h"

#include "mlir/IR/Operation.h"
#include "mlir/IR/OperationSupport.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Interfaces/LoopLikeInterface.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Interfaces/SubsetOpInterface.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
#include <queue>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Transforms/LoopInvariantCodeMotionUtils.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Transforms/LoopInvariantCodeMotionUtils.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `mlir/IR/PatternMatch.h`。

### Lines 25-38
```cpp
#define DEBUG_TYPE "licm"

using namespace mlir;

/// Checks whether the given op can be hoisted by checking that
/// - the op and none of its contained operations depend on values inside of the
///   loop (by means of calling definedOutside).
/// - the op has no side-effects.
static bool canBeHoisted(Operation *op,
                         function_ref<bool(OpOperand &)> condition) {
  // Do not move terminators.
  if (op->hasTrait<OpTrait::IsTerminator>())
    return false;

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 39-54
```cpp
  // Walk the nested operations and check that all used values are either
  // defined outside of the loop or in a nested region, but not at the level of
  // the loop body.
  auto walkFn = [&](Operation *child) {
    for (OpOperand &operand : child->getOpOperands()) {
      // Ignore values defined in a nested region.
      if (op->isAncestor(operand.get().getParentRegion()->getParentOp()))
        continue;
      if (!condition(operand))
        return WalkResult::interrupt();
    }
    return WalkResult::advance();
  };
  return !op->walk(walkFn).wasInterrupted();
}

```
- **EN**: Implements logic around `getOpOperands`, `isAncestor`, `condition`, `interrupt`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getOpOperands`、`isAncestor`、`condition`、`interrupt` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 55-67
```cpp
static bool canBeHoisted(Operation *op,
                         function_ref<bool(Value)> definedOutside) {
  return canBeHoisted(
      op, [&](OpOperand &operand) { return definedOutside(operand.get()); });
}

size_t mlir::moveLoopInvariantCode(
    ArrayRef<Region *> regions,
    function_ref<bool(Value, Region *)> isDefinedOutsideRegion,
    function_ref<bool(Operation *, Region *)> shouldMoveOutOfRegion,
    function_ref<void(Operation *, Region *)> moveOutOfRegion) {
  size_t numMoved = 0;

```
- **EN**: Implements logic around `canBeHoisted`, `function_ref`, `definedOutside`, `moveLoopInvariantCode`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `canBeHoisted`、`function_ref`、`definedOutside`、`moveLoopInvariantCode` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 68-77
```cpp
  for (Region *region : regions) {
    LDBG() << "Original loop:\n"
           << OpWithFlags(region->getParentOp(),
                          OpPrintingFlags().skipRegions());

    std::queue<Operation *> worklist;
    // Add top-level operations in the loop body to the worklist.
    for (Operation &op : region->getOps())
      worklist.push(&op);

```
- **EN**: Implements logic around `LDBG`, `OpWithFlags`, `OpPrintingFlags`, `getOps`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `LDBG`、`OpWithFlags`、`OpPrintingFlags`、`getOps` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 78-88
```cpp
    auto definedOutside = [&](Value value) {
      return isDefinedOutsideRegion(value, region);
    };

    while (!worklist.empty()) {
      Operation *op = worklist.front();
      worklist.pop();
      // Skip ops that have already been moved. Check if the op can be hoisted.
      if (op->getParentRegion() != region)
        continue;

```
- **EN**: Implements logic around `isDefinedOutsideRegion`, `empty`, `front`, `pop`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `isDefinedOutsideRegion`、`empty`、`front`、`pop` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 89-99
```cpp
      LDBG() << "Checking op: "
             << OpWithFlags(op, OpPrintingFlags().skipRegions());
      if (!shouldMoveOutOfRegion(op, region) ||
          !canBeHoisted(op, definedOutside))
        continue;

      LDBG() << "Moving loop-invariant op: "
             << OpWithFlags(op, OpPrintingFlags().skipRegions());
      moveOutOfRegion(op, region);
      ++numMoved;

```
- **EN**: Implements logic around `LDBG`, `OpWithFlags`, `shouldMoveOutOfRegion`, `canBeHoisted`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `LDBG`、`OpWithFlags`、`shouldMoveOutOfRegion`、`canBeHoisted` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 100-110
```cpp
      // Since the op has been moved, we need to check its users within the
      // top-level of the loop body.
      for (Operation *user : op->getUsers())
        if (user->getParentRegion() == region)
          worklist.push(user);
    }
  }

  return numMoved;
}

```
- **EN**: Implements logic around `getUsers`, `getParentRegion`, `push`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getUsers`、`getParentRegion`、`push` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 111-120
```cpp
size_t mlir::moveLoopInvariantCode(LoopLikeOpInterface loopLike) {
  return moveLoopInvariantCode(
      loopLike.getLoopRegions(),
      [&](Value value, Region *) {
        return loopLike.isDefinedOutsideOfLoop(value);
      },
      [&](Operation *op, Region *) { return isPure(op); },
      [&](Operation *op, Region *) { loopLike.moveOutOfLoop(op); });
}

```
- **EN**: Implements logic around `moveLoopInvariantCode`, `getLoopRegions`, `isDefinedOutsideOfLoop`, `isPure`, and 1 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `moveLoopInvariantCode`、`getLoopRegions`、`isDefinedOutsideOfLoop`、`isPure` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 121-137
```cpp
namespace {
/// Helper data structure that keeps track of equivalent/disjoint subset ops.
class MatchingSubsets {
public:
  /// Insert a subset op.
  void insert(SubsetOpInterface op, bool collectHoistableOps = true) {
    allSubsetOps.push_back(op);
    if (!collectHoistableOps)
      return;
    if (auto extractionOp =
            dyn_cast<SubsetExtractionOpInterface>(op.getOperation()))
      insertExtractionOp(extractionOp);
    if (auto insertionOp =
            dyn_cast<SubsetInsertionOpInterface>(op.getOperation()))
      insertInsertionOp(insertionOp);
  }

```
- **EN**: Introduces declarations for `MatchingSubsets`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MatchingSubsets` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 138-155
```cpp
  /// Return a range of matching extraction-insertion subset ops. If there is no
  /// matching extraction/insertion op, the respective value is empty. Ops are
  /// skipped if there are other subset ops that are not guaranteed to operate
  /// on disjoint subsets.
  auto getHoistableSubsetOps() {
    return llvm::make_filter_range(
        llvm::zip(extractions, insertions), [&](auto pair) {
          auto [extractionOp, insertionOp] = pair;
          // Hoist only if the extracted and inserted values have the same type.
          if (extractionOp && insertionOp &&
              extractionOp->getResult(0).getType() !=
                  insertionOp.getSourceOperand().get().getType())
            return false;
          // Hoist only if there are no conflicting subset ops.
          return allDisjoint(extractionOp, insertionOp);
        });
  }

```
- **EN**: Implements logic around `getHoistableSubsetOps`, `make_filter_range`, `zip`, `getResult`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getHoistableSubsetOps`、`make_filter_range`、`zip`、`getResult` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 156-165
```cpp
  /// Populate subset ops starting from the given region iter_arg. Return
  /// "failure" if non-subset ops are found along the path to the loop yielding
  /// op or if there is no single path to the tied yielded operand. If
  /// `collectHoistableOps` is set to "false", subset ops are gathered
  /// throughout the traversal, but not enumerated by `getHoistableSubsetOps`.
  LogicalResult populateSubsetOpsAtIterArg(LoopLikeOpInterface loopLike,
                                           BlockArgument iterArg,
                                           bool collectHoistableOps = true);

private:
```
- **EN**: Implements logic around `populateSubsetOpsAtIterArg`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `populateSubsetOpsAtIterArg` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 166-185
```cpp
  /// Helper function for equivalence of tensor values. Since only insertion
  /// subset ops (that are also destination style ops) are followed when
  /// traversing the SSA use-def chain, all tensor values are equivalent.
  static bool isEquivalent(Value v1, Value v2) { return true; }

  /// Return "true" if the subsets of the given extraction and insertion ops
  /// are operating disjoint from the subsets that all other known subset ops
  /// are operating on.
  bool allDisjoint(SubsetExtractionOpInterface extractionOp,
                   SubsetInsertionOpInterface insertionOp) const {
    for (SubsetOpInterface other : allSubsetOps) {
      if (other == extractionOp || other == insertionOp)
        continue;
      if (extractionOp &&
          !other.operatesOnDisjointSubset(extractionOp, isEquivalent))
        return false;
      if (insertionOp &&
          !other.operatesOnDisjointSubset(insertionOp, isEquivalent))
        return false;
    }
```
- **EN**: Implements logic around `isEquivalent`, `allDisjoint`, `operatesOnDisjointSubset`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `isEquivalent`、`allDisjoint`、`operatesOnDisjointSubset` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 186-205
```cpp
    return true;
  }

  /// Insert a subset extraction op. If the subset is equivalent to an existing
  /// subset insertion op, pair them up. (If there is already a paired up subset
  /// extraction op, overwrite the subset extraction op.)
  void insertExtractionOp(SubsetExtractionOpInterface extractionOp) {
    for (auto it : llvm::enumerate(insertions)) {
      if (!it.value())
        continue;
      auto other = cast<SubsetOpInterface>(it.value().getOperation());
      if (other.operatesOnEquivalentSubset(extractionOp, isEquivalent)) {
        extractions[it.index()] = extractionOp;
        return;
      }
    }
    // There is no known equivalent insertion op. Create a new entry.
    extractions.push_back(extractionOp);
    insertions.push_back({});
  }
```
- **EN**: Implements logic around `insertExtractionOp`, `enumerate`, `value`, `cast`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `insertExtractionOp`、`enumerate`、`value`、`cast` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 206-224
```cpp

  /// Insert a subset insertion op. If the subset is equivalent to an existing
  /// subset extraction op, pair them up. (If there is already a paired up
  /// subset insertion op, overwrite the subset insertion op.)
  void insertInsertionOp(SubsetInsertionOpInterface insertionOp) {
    for (auto it : llvm::enumerate(extractions)) {
      if (!it.value())
        continue;
      auto other = cast<SubsetOpInterface>(it.value().getOperation());
      if (other.operatesOnEquivalentSubset(insertionOp, isEquivalent)) {
        insertions[it.index()] = insertionOp;
        return;
      }
    }
    // There is no known equivalent extraction op. Create a new entry.
    extractions.push_back({});
    insertions.push_back(insertionOp);
  }

```
- **EN**: Implements logic around `insertInsertionOp`, `enumerate`, `value`, `cast`, and 3 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `insertInsertionOp`、`enumerate`、`value`、`cast` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 225-241
```cpp
  SmallVector<SubsetExtractionOpInterface> extractions;
  SmallVector<SubsetInsertionOpInterface> insertions;
  SmallVector<SubsetOpInterface> allSubsetOps;
};
} // namespace

/// If the given value has a single use by an op that is a terminator, return
/// that use. Otherwise, return nullptr.
static OpOperand *getSingleTerminatorUse(Value value) {
  if (!value.hasOneUse())
    return nullptr;
  OpOperand &use = *value.getUses().begin();
  if (use.getOwner()->hasTrait<OpTrait::IsTerminator>())
    return &use;
  return nullptr;
}

```
- **EN**: Implements logic around `getSingleTerminatorUse`, `hasOneUse`, `getUses`, `getOwner`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getSingleTerminatorUse`、`hasOneUse`、`getUses`、`getOwner` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 242-258
```cpp
LogicalResult
MatchingSubsets::populateSubsetOpsAtIterArg(LoopLikeOpInterface loopLike,
                                            BlockArgument iterArg,
                                            bool collectHoistableOps) {
  assert(iterArg.getOwner()->getParentOp() == loopLike && "invalid iter_arg");
  Value value = iterArg;

  // Traverse use-def chain. Subset ops can be hoisted only if all ops along the
  // use-def chain starting from the region iter_arg are subset extraction or
  // subset insertion ops. The chain must terminate at the corresponding yield
  // operand (e.g., no swapping of iter_args).
  OpOperand *yieldedOperand = nullptr;
  // Iterate until the single use of the current SSA value is a terminator,
  // which is expected to be the yielding operation of the loop.
  while (!(yieldedOperand = getSingleTerminatorUse(value))) {
    Value nextValue = {};

```
- **EN**: Implements logic around `populateSubsetOpsAtIterArg`, `assert`, `getSingleTerminatorUse`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `populateSubsetOpsAtIterArg`、`assert`、`getSingleTerminatorUse` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 259-278
```cpp
    for (OpOperand &use : value.getUses()) {
      if (auto nestedLoop = dyn_cast<LoopLikeOpInterface>(use.getOwner())) {
        // Subset ops in nested loops are collected to check if there are only
        // disjoint subset ops, but such subset ops are not subject to hoisting.
        // To hoist subset ops from nested loops, the hoisting transformation
        // should be run on the nested loop.
        auto nestedIterArg = nestedLoop.getTiedLoopRegionIterArg(&use);
        if (!nestedIterArg)
          return failure();
        // Note: `populateSubsetOpsAtIterArg` fails if there is no single SSA
        // use-def chain starting at `nestedIterArg` and terminating in the
        // tied, yielding operand.
        if (failed(populateSubsetOpsAtIterArg(nestedLoop, nestedIterArg,
                                              /*collectHoistableOps=*/false)))
          return failure();
        // There must be a single use-def chain. Bail if there are multiple
        // nested loops using this value.
        if (nextValue)
          return failure();
        nextValue = nestedLoop.getTiedLoopResult(&use);
```
- **EN**: Implements logic around `getUses`, `dyn_cast`, `getTiedLoopRegionIterArg`, `failure`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getUses`、`dyn_cast`、`getTiedLoopRegionIterArg`、`failure` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 279-295
```cpp
        continue;
      }

      auto subsetOp = dyn_cast<SubsetOpInterface>(use.getOwner());
      if (!subsetOp)
        return failure();
      insert(subsetOp, collectHoistableOps);

      if (auto insertionOp =
              dyn_cast<SubsetInsertionOpInterface>(use.getOwner())) {
        // Current implementation expects that the insertionOp implement
        // the DestinationStyleOpInterface and with pure tensor semantics
        // as well. Abort if that is not the case.
        auto dstOp = dyn_cast<DestinationStyleOpInterface>(use.getOwner());
        if (!dstOp || !dstOp.hasPureTensorSemantics())
          return failure();

```
- **EN**: Implements logic around `dyn_cast`, `failure`, `insert`, `hasPureTensorSemantics`; this block implements transformation or simplification logic.
- **CN**: 围绕 `dyn_cast`、`failure`、`insert`、`hasPureTensorSemantics` 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 296-307
```cpp
        // The value must be used as a destination. (In case of a source, the
        // entire tensor would be read, which would prevent any hoisting.)
        if (&use != &insertionOp.getDestinationOperand())
          return failure();
        // There must be a single use-def chain from the region iter_arg to the
        // terminator. I.e., only one insertion op. Branches are not supported.
        if (nextValue)
          return failure();
        nextValue = insertionOp.getUpdatedDestination();
      }
    }

```
- **EN**: Implements logic around `getDestinationOperand`, `failure`, `getUpdatedDestination`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getDestinationOperand`、`failure`、`getUpdatedDestination` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 308-320
```cpp
    // Nothing can be hoisted if the chain does not continue with loop yielding
    // op or a subset insertion op.
    if (!nextValue)
      return failure();
    value = nextValue;
  }

  // Hoist only if the SSA use-def chain ends in the yielding terminator of the
  // loop and the yielded value is the `idx`-th operand. (I.e., there is no
  // swapping yield.)
  if (loopLike.getTiedLoopYieldedValue(iterArg) != yieldedOperand)
    return failure();

```
- **EN**: Implements logic around `failure`, `getTiedLoopYieldedValue`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `failure`、`getTiedLoopYieldedValue` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 321-337
```cpp
  return success();
}

/// Hoist all subset ops that operate on the idx-th region iter_arg of the given
/// loop-like op and index into loop-invariant subset locations. Return the
/// newly created loop op (that has extra iter_args) or the original loop op if
/// nothing was hoisted.
static LoopLikeOpInterface hoistSubsetAtIterArg(RewriterBase &rewriter,
                                                LoopLikeOpInterface loopLike,
                                                BlockArgument iterArg) {
  assert(iterArg.getOwner()->getParentOp() == loopLike && "invalid iter_arg");
  BlockArgument *it = llvm::find(loopLike.getRegionIterArgs(), iterArg);
  int64_t iterArgIdx = std::distance(loopLike.getRegionIterArgs().begin(), it);
  MatchingSubsets subsets;
  if (failed(subsets.populateSubsetOpsAtIterArg(loopLike, iterArg)))
    return loopLike;

```
- **EN**: Implements logic around `success`, `hoistSubsetAtIterArg`, `assert`, `find`, and 2 more symbols; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `success`、`hoistSubsetAtIterArg`、`assert`、`find` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 338-357
```cpp
  // Hoist all matching extraction-insertion pairs one-by-one.
  for (auto it : subsets.getHoistableSubsetOps()) {
    auto extractionOp = std::get<0>(it);
    auto insertionOp = std::get<1>(it);

    // Ops cannot be hoisted if they depend on loop-variant values.
    if (extractionOp) {
      if (!canBeHoisted(extractionOp, [&](OpOperand &operand) {
            return loopLike.isDefinedOutsideOfLoop(operand.get()) ||
                   &operand == &extractionOp.getSourceOperand();
          }))
        extractionOp = {};
    }
    if (insertionOp) {
      if (!canBeHoisted(insertionOp, [&](OpOperand &operand) {
            return loopLike.isDefinedOutsideOfLoop(operand.get()) ||
                   &operand == &insertionOp.getSourceOperand() ||
                   &operand == &insertionOp.getDestinationOperand();
          }))
        insertionOp = {};
```
- **EN**: Implements logic around `getHoistableSubsetOps`, `get`, `canBeHoisted`, `isDefinedOutsideOfLoop`, and 2 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `getHoistableSubsetOps`、`get`、`canBeHoisted`、`isDefinedOutsideOfLoop` 等另外 2 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 358-377
```cpp
    }

    // Only hoist extraction-insertion pairs for now. Standalone extractions/
    // insertions that are loop-invariant could be hoisted, but there may be
    // easier ways to canonicalize the IR.
    if (extractionOp && insertionOp) {
      // Create a new loop with an additional iter_arg.
      NewYieldValuesFn newYieldValuesFn =
          [&](OpBuilder &b, Location loc,
              ArrayRef<BlockArgument> innerNewBBArgs) -> SmallVector<Value> {
        return {insertionOp.getSourceOperand().get()};
      };
      FailureOr<LoopLikeOpInterface> newLoop =
          loopLike.replaceWithAdditionalYields(
              rewriter, extractionOp.getResult(),
              /*replaceInitOperandUsesInLoop=*/true, newYieldValuesFn);
      if (failed(newLoop))
        return loopLike;
      loopLike = *newLoop;

```
- **EN**: Implements logic around `getSourceOperand`, `replaceWithAdditionalYields`, `getResult`, `failed`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `getSourceOperand`、`replaceWithAdditionalYields`、`getResult`、`failed` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

### Lines 378-394
```cpp
      // Hoist the extraction/insertion ops.
      iterArg = loopLike.getRegionIterArgs()[iterArgIdx];
      OpResult loopResult = loopLike.getTiedLoopResult(iterArg);
      OpResult newLoopResult = loopLike.getLoopResults()->back();
      rewriter.moveOpBefore(extractionOp, loopLike);
      rewriter.moveOpAfter(insertionOp, loopLike);
      rewriter.replaceAllUsesWith(insertionOp.getUpdatedDestination(),
                                  insertionOp.getDestinationOperand().get());
      extractionOp.getSourceOperand().set(
          loopLike.getTiedLoopInit(iterArg)->get());
      rewriter.replaceAllUsesWith(loopResult,
                                  insertionOp.getUpdatedDestination());
      insertionOp.getSourceOperand().set(newLoopResult);
      insertionOp.getDestinationOperand().set(loopResult);
    }
  }

```
- **EN**: Implements logic around `getRegionIterArgs`, `getTiedLoopResult`, `getLoopResults`, `moveOpBefore`, and 6 more symbols; this block implements transformation or simplification logic.
- **CN**: 围绕 `getRegionIterArgs`、`getTiedLoopResult`、`getLoopResults`、`moveOpBefore` 等另外 6 个符号 实现具体逻辑；该代码块实现变换或简化逻辑。

### Lines 395-410
```cpp
  return loopLike;
}

LoopLikeOpInterface
mlir::hoistLoopInvariantSubsets(RewriterBase &rewriter,
                                LoopLikeOpInterface loopLike) {
  // Note: As subset ops are getting hoisted, the number of region iter_args
  // increases. This can enable further hoisting opportunities on the new
  // iter_args.
  for (int64_t i = 0;
       i < static_cast<int64_t>(loopLike.getRegionIterArgs().size()); ++i) {
    loopLike = hoistSubsetAtIterArg(rewriter, loopLike,
                                    loopLike.getRegionIterArgs()[i]);
  }
  return loopLike;
}
```
- **EN**: Implements logic around `hoistLoopInvariantSubsets`, `static_cast`, `hoistSubsetAtIterArg`, `getRegionIterArgs`; this block manipulates core MLIR IR objects; implements transformation or simplification logic.
- **CN**: 围绕 `hoistLoopInvariantSubsets`、`static_cast`、`hoistSubsetAtIterArg`、`getRegionIterArgs` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并实现变换或简化逻辑。

## Key Concepts / 关键概念

- **IR transformation passes / IR 变换 Pass**:
  - **EN**: Applies canonicalization, dataflow-aware simplification, or structural rewrites to MLIR.
  - **CN**: 对 MLIR 应用规范化、数据流感知简化或结构性重写。
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

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Transforms/LoopInvariantCodeMotionUtils.h`, `mlir/IR/Operation.h`, `mlir/IR/OperationSupport.h`, `mlir/IR/PatternMatch.h`, `mlir/Interfaces/LoopLikeInterface.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `mlir/Interfaces/SubsetOpInterface.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugLog.h`
- **Standard-library headers / 标准库头文件**: `<queue>`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (3), MLIR interface declarations / MLIR 接口声明 (3), LLVM support-library helpers / LLVM Support 库辅助工具 (2), core transformation utilities / 核心变换工具 (1)
