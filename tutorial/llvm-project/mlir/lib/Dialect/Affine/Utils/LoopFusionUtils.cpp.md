# LoopFusionUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Utils/LoopFusionUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements loop fusion transformation utility functions.
  - **CN**: 实现 Affine 方言与仿射循环推理 使用的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- LoopFusionUtils.cpp ---- Utilities for loop fusion ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements loop fusion transformation utility functions.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-28
```cpp

#include "mlir/Dialect/Affine/LoopFusionUtils.h"
#include "mlir/Analysis/SliceAnalysis.h"
#include "mlir/Analysis/TopologicalSortUtils.h"
#include "mlir/Dialect/Affine/Analysis/AffineAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/LoopAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/Utils.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/LoopUtils.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/PatternMatch.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/raw_ostream.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/LoopFusionUtils.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/LoopFusionUtils.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`。

### Lines 29-47
```cpp
#define DEBUG_TYPE "affine-fusion-utils"

using namespace mlir;
using namespace mlir::affine;

// Gathers all load and store memref accesses in 'opA' into 'values', where
// 'values[memref] == true' for each store operation.
static void getLoadAndStoreMemRefAccesses(Operation *opA,
                                          DenseMap<Value, bool> &values) {
  opA->walk([&](Operation *op) {
    if (auto loadOp = dyn_cast<AffineReadOpInterface>(op)) {
      if (values.count(loadOp.getMemRef()) == 0)
        values[loadOp.getMemRef()] = false;
    } else if (auto storeOp = dyn_cast<AffineWriteOpInterface>(op)) {
      values[storeOp.getMemRef()] = true;
    }
  });
}

```
- **EN**: Implements logic around `getLoadAndStoreMemRefAccesses`, `walk`, `dyn_cast`, `count`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getLoadAndStoreMemRefAccesses`, `walk`, `dyn_cast`, `count`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 48-59
```cpp
/// Returns true if 'op' is a load or store operation which access a memref
/// accessed 'values' and at least one of the access is a store operation.
/// Returns false otherwise.
static bool isDependentLoadOrStoreOp(Operation *op,
                                     DenseMap<Value, bool> &values) {
  if (auto loadOp = dyn_cast<AffineReadOpInterface>(op))
    return values.count(loadOp.getMemRef()) > 0 && values[loadOp.getMemRef()];
  if (auto storeOp = dyn_cast<AffineWriteOpInterface>(op))
    return values.count(storeOp.getMemRef()) > 0;
  return false;
}

```
- **EN**: Implements logic around `isDependentLoadOrStoreOp`, `dyn_cast`, `count`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `isDependentLoadOrStoreOp`, `dyn_cast`, `count` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 60-79
```cpp
// Returns the first operation in range ('opA', 'opB') which has a data
// dependence on 'opA'. Returns 'nullptr' of no dependence exists.
static Operation *getFirstDependentOpInRange(Operation *opA, Operation *opB) {
  // Record memref values from all loads/store in loop nest rooted at 'opA'.
  // Map from memref value to bool which is true if store, false otherwise.
  DenseMap<Value, bool> values;
  getLoadAndStoreMemRefAccesses(opA, values);

  // For each 'opX' in block in range ('opA', 'opB'), check if there is a data
  // dependence from 'opA' to 'opX' ('opA' and 'opX' access the same memref
  // and at least one of the accesses is a store).
  Operation *firstDepOp = nullptr;
  for (Block::iterator it = std::next(Block::iterator(opA));
       it != Block::iterator(opB); ++it) {
    Operation *opX = &(*it);
    opX->walk([&](Operation *op) {
      if (!firstDepOp && isDependentLoadOrStoreOp(op, values))
        firstDepOp = opX;
    });
    if (firstDepOp)
```
- **EN**: Implements logic around `getFirstDependentOpInRange`, `getLoadAndStoreMemRefAccesses`, `next`, `iterator`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getFirstDependentOpInRange`, `getLoadAndStoreMemRefAccesses`, `next`, `iterator`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理 MLIR region、block 或控制流边。

### Lines 80-93
```cpp
      break;
  }
  return firstDepOp;
}

// Returns the last operation 'opX' in range ('opA', 'opB'), for which there
// exists a data dependence from 'opX' to 'opB'.
// Returns 'nullptr' of no dependence exists.
static Operation *getLastDependentOpInRange(Operation *opA, Operation *opB) {
  // Record memref values from all loads/store in loop nest rooted at 'opB'.
  // Map from memref value to bool which is true if store, false otherwise.
  DenseMap<Value, bool> values;
  getLoadAndStoreMemRefAccesses(opB, values);

```
- **EN**: Implements logic around `getLastDependentOpInRange`, `getLoadAndStoreMemRefAccesses`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getLastDependentOpInRange`, `getLoadAndStoreMemRefAccesses` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 94-113
```cpp
  // For each 'opX' in block in range ('opA', 'opB') in reverse order,
  // check if there is a data dependence from 'opX' to 'opB':
  // *) 'opX' and 'opB' access the same memref and at least one of the accesses
  //    is a store.
  // *) 'opX' produces an SSA Value which is used by 'opB'.
  Operation *lastDepOp = nullptr;
  for (Block::reverse_iterator it = std::next(Block::reverse_iterator(opB));
       it != Block::reverse_iterator(opA); ++it) {
    Operation *opX = &(*it);
    opX->walk([&](Operation *op) {
      if (isa<AffineReadOpInterface, AffineWriteOpInterface>(op)) {
        if (isDependentLoadOrStoreOp(op, values)) {
          lastDepOp = opX;
          return WalkResult::interrupt();
        }
        return WalkResult::advance();
      }
      for (Value value : op->getResults()) {
        for (Operation *user : value.getUsers()) {
          SmallVector<AffineForOp, 4> loops;
```
- **EN**: Implements logic around `next`, `reverse_iterator`, `walk`, `AffineWriteOpInterface>`, and 5 more symbols; this block moves data between tensor-style values and explicit buffers; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `next`, `reverse_iterator`, `walk`, `AffineWriteOpInterface>`, and 5 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理 MLIR region、block 或控制流边。

### Lines 114-129
```cpp
          // Check if any loop in loop nest surrounding 'user' is 'opB'.
          getAffineForIVs(*user, &loops);
          if (llvm::is_contained(loops, cast<AffineForOp>(opB))) {
            lastDepOp = opX;
            return WalkResult::interrupt();
          }
        }
      }
      return WalkResult::advance();
    });
    if (lastDepOp)
      break;
  }
  return lastDepOp;
}

```
- **EN**: Implements logic around `getAffineForIVs`, `is_contained`, `interrupt`, `advance`.
- **CN**: 围绕 `getAffineForIVs`, `is_contained`, `interrupt`, `advance` 实现具体逻辑。

### Lines 130-149
```cpp
// Computes and returns an insertion point operation, before which the
// the fused <srcForOp, dstForOp> loop nest can be inserted while preserving
// dependences. Returns nullptr if no such insertion point is found.
static Operation *getFusedLoopNestInsertionPoint(AffineForOp srcForOp,
                                                 AffineForOp dstForOp) {
  bool isSrcForOpBeforeDstForOp = srcForOp->isBeforeInBlock(dstForOp);
  auto forOpA = isSrcForOpBeforeDstForOp ? srcForOp : dstForOp;
  auto forOpB = isSrcForOpBeforeDstForOp ? dstForOp : srcForOp;

  Operation *firstDepOpA = getFirstDependentOpInRange(forOpA, forOpB);
  Operation *lastDepOpB = getLastDependentOpInRange(forOpA, forOpB);
  // Block:
  //      ...
  //  |-- opA
  //  |   ...
  //  |   lastDepOpB --|
  //  |   ...          |
  //  |-> firstDepOpA  |
  //      ...          |
  //      opB <---------
```
- **EN**: Implements logic around `getFusedLoopNestInsertionPoint`, `isBeforeInBlock`, `getFirstDependentOpInRange`, `getLastDependentOpInRange`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getFusedLoopNestInsertionPoint`, `isBeforeInBlock`, `getFirstDependentOpInRange`, `getLastDependentOpInRange` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 150-167
```cpp
  //
  // Valid insertion point range: (lastDepOpB, firstDepOpA)
  //
  if (firstDepOpA) {
    if (lastDepOpB) {
      if (firstDepOpA->isBeforeInBlock(lastDepOpB) || firstDepOpA == lastDepOpB)
        // No valid insertion point exists which preserves dependences.
        return nullptr;
    }
    // Return insertion point in valid range closest to 'opB'.
    // TODO: Consider other insertion points in valid range.
    return firstDepOpA;
  }
  // No dependences from 'opA' to operation in range ('opA', 'opB'), return
  // 'opB' insertion point.
  return forOpB;
}

```
- **EN**: Implements logic around `isBeforeInBlock`.
- **CN**: 围绕 `isBeforeInBlock` 实现具体逻辑。

### Lines 168-182
```cpp
// Gathers all load and store ops in loop nest rooted at 'forOp' into
// 'loadAndStoreOps'.
static bool
gatherLoadsAndStores(AffineForOp forOp,
                     SmallVectorImpl<Operation *> &loadAndStoreOps) {
  bool hasIfOp = false;
  forOp.walk([&](Operation *op) {
    if (isa<AffineReadOpInterface, AffineWriteOpInterface>(op))
      loadAndStoreOps.push_back(op);
    else if (isa<AffineIfOp>(op))
      hasIfOp = true;
  });
  return !hasIfOp;
}

```
- **EN**: Implements logic around `gatherLoadsAndStores`, `walk`, `AffineWriteOpInterface>`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `gatherLoadsAndStores`, `walk`, `AffineWriteOpInterface>`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 183-193
```cpp
/// Returns the maximum loop depth at which we could fuse producer loop
/// 'srcForOp' into consumer loop 'dstForOp' without violating data dependences.
// TODO: Generalize this check for sibling and more generic fusion scenarios.
// TODO: Support forward slice fusion.
static unsigned getMaxLoopDepth(ArrayRef<Operation *> srcOps,
                                ArrayRef<Operation *> dstOps) {
  if (dstOps.empty())
    // Expected at least one memory operation.
    // TODO: Revisit this case with a specific example.
    return 0;

```
- **EN**: Implements logic around `getMaxLoopDepth`, `empty`.
- **CN**: 围绕 `getMaxLoopDepth`, `empty` 实现具体逻辑。

### Lines 194-206
```cpp
  // Filter out ops in 'dstOps' that do not use the producer-consumer memref so
  // that they are not considered for analysis.
  DenseSet<Value> producerConsumerMemrefs;
  gatherProducerConsumerMemrefs(srcOps, dstOps, producerConsumerMemrefs);
  SmallVector<Operation *, 4> targetDstOps;
  for (Operation *dstOp : dstOps) {
    auto loadOp = dyn_cast<AffineReadOpInterface>(dstOp);
    Value memref = loadOp ? loadOp.getMemRef()
                          : cast<AffineWriteOpInterface>(dstOp).getMemRef();
    if (producerConsumerMemrefs.count(memref) > 0)
      targetDstOps.push_back(dstOp);
  }

```
- **EN**: Implements logic around `gatherProducerConsumerMemrefs`, `dyn_cast`, `getMemRef`, `cast`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `gatherProducerConsumerMemrefs`, `dyn_cast`, `getMemRef`, `cast`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 207-216
```cpp
  assert(!targetDstOps.empty() &&
         "No dependences between 'srcForOp' and 'dstForOp'?");

  // Compute the innermost common loop depth for loads and stores.
  unsigned loopDepth = getInnermostCommonLoopDepth(targetDstOps);

  // Return common loop depth for loads if there are no store ops.
  if (all_of(targetDstOps, llvm::IsaPred<AffineReadOpInterface>))
    return loopDepth;

```
- **EN**: Implements logic around `assert`, `getInnermostCommonLoopDepth`, `all_of`.
- **CN**: 围绕 `assert`, `getInnermostCommonLoopDepth`, `all_of` 实现具体逻辑。

### Lines 217-236
```cpp
  // Check dependences on all pairs of ops in 'targetDstOps' and store the
  // minimum loop depth at which a dependence is satisfied.
  for (unsigned i = 0, e = targetDstOps.size(); i < e; ++i) {
    Operation *srcOpInst = targetDstOps[i];
    MemRefAccess srcAccess(srcOpInst);
    for (unsigned j = 0; j < e; ++j) {
      auto *dstOpInst = targetDstOps[j];
      MemRefAccess dstAccess(dstOpInst);

      unsigned numCommonLoops =
          getNumCommonSurroundingLoops(*srcOpInst, *dstOpInst);
      for (unsigned d = 1; d <= numCommonLoops + 1; ++d) {
        // TODO: Cache dependence analysis results, check cache here.
        DependenceResult result =
            checkMemrefAccessDependence(srcAccess, dstAccess, d);
        if (hasDependence(result)) {
          // Store minimum loop depth and break because we want the min 'd' at
          // which there is a dependence.
          loopDepth = std::min(loopDepth, d - 1);
          break;
```
- **EN**: Implements logic around `size`, `srcAccess`, `dstAccess`, `getNumCommonSurroundingLoops`, and 3 more symbols.
- **CN**: 围绕 `size`, `srcAccess`, `dstAccess`, `getNumCommonSurroundingLoops`, and 3 more symbols 实现具体逻辑。

### Lines 237-256
```cpp
        }
      }
    }
  }

  return loopDepth;
}

// TODO: This pass performs some computation that is the same for all the depths
// (e.g., getMaxLoopDepth). Implement a version of this utility that processes
// all the depths at once or only the legal maximal depth for maximal fusion.
FusionResult mlir::affine::canFuseLoops(AffineForOp srcForOp,
                                        AffineForOp dstForOp,
                                        unsigned dstLoopDepth,
                                        ComputationSliceState *srcSlice,
                                        FusionStrategy fusionStrategy) {
  // Return 'failure' if 'dstLoopDepth == 0'.
  if (dstLoopDepth == 0) {
    LDBG() << "Cannot fuse loop nests at depth 0";
    return FusionResult::FailPrecondition;
```
- **EN**: Implements logic around `canFuseLoops`, `LDBG`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `canFuseLoops`, `LDBG` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 257-271
```cpp
  }
  // Return 'failure' if 'srcForOp' and 'dstForOp' are not in the same block.
  auto *block = srcForOp->getBlock();
  if (block != dstForOp->getBlock()) {
    LDBG() << "Cannot fuse loop nests in different blocks";
    return FusionResult::FailPrecondition;
  }

  // Return 'failure' if no valid insertion point for fused loop nest in 'block'
  // exists which would preserve dependences.
  if (!getFusedLoopNestInsertionPoint(srcForOp, dstForOp)) {
    LDBG() << "Fusion would violate dependences in block";
    return FusionResult::FailBlockDependence;
  }

```
- **EN**: Implements logic around `getBlock`, `LDBG`, `getFusedLoopNestInsertionPoint`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBlock`, `LDBG`, `getFusedLoopNestInsertionPoint` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 272-284
```cpp
  // Check if 'srcForOp' precedes 'dstForOp' in 'block'.
  bool isSrcForOpBeforeDstForOp = srcForOp->isBeforeInBlock(dstForOp);
  // 'forOpA' executes before 'forOpB' in 'block'.
  auto forOpA = isSrcForOpBeforeDstForOp ? srcForOp : dstForOp;
  auto forOpB = isSrcForOpBeforeDstForOp ? dstForOp : srcForOp;

  // Gather all load and store from 'forOpA' which precedes 'forOpB' in 'block'.
  SmallVector<Operation *, 4> opsA;
  if (!gatherLoadsAndStores(forOpA, opsA)) {
    LDBG() << "Fusing loops with affine.if unsupported";
    return FusionResult::FailPrecondition;
  }

```
- **EN**: Implements logic around `isBeforeInBlock`, `gatherLoadsAndStores`, `LDBG`; this block performs affine reasoning or shape/bounds manipulation; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `isBeforeInBlock`, `gatherLoadsAndStores`, `LDBG` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理 MLIR region、block 或控制流边。

### Lines 285-304
```cpp
  // Gather all load and store from 'forOpB' which succeeds 'forOpA' in 'block'.
  SmallVector<Operation *, 4> opsB;
  if (!gatherLoadsAndStores(forOpB, opsB)) {
    LDBG() << "Fusing loops with affine.if unsupported";
    return FusionResult::FailPrecondition;
  }

  // Return 'failure' if fusing loops at depth 'dstLoopDepth' wouldn't preserve
  // loop dependences.
  // TODO: Enable this check for sibling and more generic loop fusion
  // strategies.
  if (fusionStrategy.getStrategy() == FusionStrategy::ProducerConsumer) {
    // TODO: 'getMaxLoopDepth' does not support forward slice fusion.
    assert(isSrcForOpBeforeDstForOp && "Unexpected forward slice fusion");
    if (getMaxLoopDepth(opsA, opsB) < dstLoopDepth) {
      LDBG() << "Fusion would violate loop dependences";
      return FusionResult::FailFusionDependence;
    }
  }

```
- **EN**: Implements logic around `gatherLoadsAndStores`, `LDBG`, `getStrategy`, `assert`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `gatherLoadsAndStores`, `LDBG`, `getStrategy`, `assert`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理 MLIR region、block 或控制流边。

### Lines 305-324
```cpp
  // Calculate the number of common loops surrounding 'srcForOp' and 'dstForOp'.
  unsigned numCommonLoops =
      affine::getNumCommonSurroundingLoops(*srcForOp, *dstForOp);

  // Filter out ops in 'opsA' to compute the slice union based on the
  // assumptions made by the fusion strategy.
  SmallVector<Operation *, 4> strategyOpsA;
  switch (fusionStrategy.getStrategy()) {
  case FusionStrategy::Generic:
    // Generic fusion. Take into account all the memory operations to compute
    // the slice union.
    strategyOpsA.append(opsA.begin(), opsA.end());
    break;
  case FusionStrategy::ProducerConsumer:
    // Producer-consumer fusion (AffineLoopFusion pass) only takes into
    // account stores in 'srcForOp' to compute the slice union.
    for (Operation *op : opsA) {
      if (isa<AffineWriteOpInterface>(op))
        strategyOpsA.push_back(op);
    }
```
- **EN**: Implements logic around `getNumCommonSurroundingLoops`, `getStrategy`, `append`, `isa`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getNumCommonSurroundingLoops`, `getStrategy`, `append`, `isa`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 325-336
```cpp
    break;
  case FusionStrategy::Sibling:
    // Sibling fusion (AffineLoopFusion pass) only takes into account the loads
    // to 'memref' in 'srcForOp' to compute the slice union.
    for (Operation *op : opsA) {
      auto load = dyn_cast<AffineReadOpInterface>(op);
      if (load && load.getMemRef() == fusionStrategy.getSiblingFusionMemRef())
        strategyOpsA.push_back(op);
    }
    break;
  }

```
- **EN**: Implements logic around `dyn_cast`, `getMemRef`, `push_back`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `dyn_cast`, `getMemRef`, `push_back` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 337-351
```cpp
  // Compute union of computation slices computed between all pairs of ops
  // from 'forOpA' and 'forOpB'.
  SliceComputationResult sliceComputationResult = affine::computeSliceUnion(
      strategyOpsA, opsB, dstLoopDepth, numCommonLoops,
      isSrcForOpBeforeDstForOp, srcSlice);
  if (sliceComputationResult.value == SliceComputationResult::GenericFailure) {
    LDBG() << "computeSliceUnion failed";
    return FusionResult::FailPrecondition;
  }
  if (sliceComputationResult.value ==
      SliceComputationResult::IncorrectSliceFailure) {
    LDBG() << "Incorrect slice computation";
    return FusionResult::FailIncorrectSlice;
  }

```
- **EN**: Implements logic around `computeSliceUnion`, `LDBG`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `computeSliceUnion`, `LDBG` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 352-371
```cpp
  return FusionResult::Success;
}

/// Patch the loop body of a forOp that is a single iteration reduction loop
/// into its containing block.
static LogicalResult promoteSingleIterReductionLoop(AffineForOp forOp,
                                                    bool siblingFusionUser) {
  // Check if the reduction loop is a single iteration loop.
  std::optional<uint64_t> tripCount = getConstantTripCount(forOp);
  if (!tripCount || *tripCount != 1)
    return failure();
  auto *parentOp = forOp->getParentOp();
  if (!isa<AffineForOp>(parentOp))
    return failure();
  SmallVector<Value> newOperands;
  llvm::append_range(newOperands,
                     forOp.getBody()->getTerminator()->getOperands());
  IRRewriter rewriter(parentOp->getContext());
  int64_t parentOpNumResults = parentOp->getNumResults();
  // Replace the parent loop and add iteroperands and results from the `forOp`.
```
- **EN**: Implements logic around `promoteSingleIterReductionLoop`, `getConstantTripCount`, `failure`, `getParentOp`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `promoteSingleIterReductionLoop`, `getConstantTripCount`, `failure`, `getParentOp`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 372-391
```cpp
  AffineForOp parentForOp = forOp->getParentOfType<AffineForOp>();
  AffineForOp newLoop =
      cast<AffineForOp>(*parentForOp.replaceWithAdditionalYields(
          rewriter, forOp.getInits(), /*replaceInitOperandUsesInLoop=*/false,
          [&](OpBuilder &b, Location loc, ArrayRef<BlockArgument> newBbArgs) {
            return newOperands;
          }));

  // For sibling-fusion users, collect operations that use the results of the
  // `forOp` outside the new parent loop that has absorbed all its iter args
  // and operands. These operations will be moved later after the results
  // have been replaced.
  SetVector<Operation *> forwardSlice;
  if (siblingFusionUser) {
    for (unsigned i = 0, e = forOp.getNumResults(); i != e; ++i) {
      SetVector<Operation *> tmpForwardSlice;
      getForwardSlice(forOp.getResult(i), &tmpForwardSlice);
      forwardSlice.set_union(tmpForwardSlice);
    }
  }
```
- **EN**: Implements logic around `getParentOfType`, `cast`, `getInits`, `getNumResults`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getParentOfType`, `cast`, `getInits`, `getNumResults`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 392-411
```cpp
  // Update the results of the `forOp` in the new loop.
  for (unsigned i = 0, e = forOp.getNumResults(); i != e; ++i) {
    forOp.getResult(i).replaceAllUsesWith(
        newLoop.getResult(i + parentOpNumResults));
  }
  // For sibling-fusion users, move operations that use the results of the
  // `forOp` outside the new parent loop
  if (siblingFusionUser) {
    topologicalSort(forwardSlice);
    for (Operation *op : llvm::reverse(forwardSlice))
      op->moveAfter(newLoop);
  }
  // Replace the induction variable.
  auto iv = forOp.getInductionVar();
  iv.replaceAllUsesWith(newLoop.getInductionVar());
  // Replace the iter args.
  auto forOpIterArgs = forOp.getRegionIterArgs();
  for (auto it : llvm::zip(forOpIterArgs, newLoop.getRegionIterArgs().take_back(
                                              forOpIterArgs.size()))) {
    std::get<0>(it).replaceAllUsesWith(std::get<1>(it));
```
- **EN**: Implements logic around `getNumResults`, `getResult`, `topologicalSort`, `reverse`, and 7 more symbols.
- **CN**: 围绕 `getNumResults`, `getResult`, `topologicalSort`, `reverse`, and 7 more symbols 实现具体逻辑。

### Lines 412-422
```cpp
  }
  // Move the loop body operations, except for its terminator, to the loop's
  // containing block.
  forOp.getBody()->back().erase();
  auto *parentBlock = forOp->getBlock();
  parentBlock->getOperations().splice(Block::iterator(forOp),
                                      forOp.getBody()->getOperations());
  forOp.erase();
  return success();
}

```
- **EN**: Implements logic around `getBody`, `getBlock`, `getOperations`, `erase`, and 1 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBody`, `getBlock`, `getOperations`, `erase`, and 1 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 423-432
```cpp
/// Fuses 'srcForOp' into 'dstForOp' with destination loop block insertion point
/// and source slice loop bounds specified in 'srcSlice'.
void mlir::affine::fuseLoops(AffineForOp srcForOp, AffineForOp dstForOp,
                             const ComputationSliceState &srcSlice,
                             bool isInnermostSiblingInsertion) {
  // Clone 'srcForOp' into 'dstForOp' at 'srcSlice->insertPoint'.
  OpBuilder b(srcSlice.insertPoint->getBlock(), srcSlice.insertPoint);
  IRMapping mapper;
  b.clone(*srcForOp, mapper);

```
- **EN**: Implements logic around `fuseLoops`, `b`, `clone`; this block performs affine reasoning or shape/bounds manipulation; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `fuseLoops`, `b`, `clone` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理 MLIR region、block 或控制流边。

### Lines 433-452
```cpp
  // Update 'sliceLoopNest' upper and lower bounds from computed 'srcSlice'.
  SmallVector<AffineForOp, 4> sliceLoops;
  for (unsigned i = 0, e = srcSlice.ivs.size(); i < e; ++i) {
    auto loopIV = mapper.lookupOrNull(srcSlice.ivs[i]);
    if (!loopIV)
      continue;
    auto forOp = getForInductionVarOwner(loopIV);
    sliceLoops.push_back(forOp);
    if (AffineMap lbMap = srcSlice.lbs[i]) {
      auto lbOperands = srcSlice.lbOperands[i];
      canonicalizeMapAndOperands(&lbMap, &lbOperands);
      forOp.setLowerBound(lbOperands, lbMap);
    }
    if (AffineMap ubMap = srcSlice.ubs[i]) {
      auto ubOperands = srcSlice.ubOperands[i];
      canonicalizeMapAndOperands(&ubMap, &ubOperands);
      forOp.setUpperBound(ubOperands, ubMap);
    }
  }

```
- **EN**: Implements logic around `size`, `lookupOrNull`, `getForInductionVarOwner`, `push_back`, and 3 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `size`, `lookupOrNull`, `getForInductionVarOwner`, `push_back`, and 3 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 453-470
```cpp
  llvm::SmallDenseMap<Operation *, uint64_t, 8> sliceTripCountMap;
  auto srcIsUnitSlice = [&]() {
    return (buildSliceTripCountMap(srcSlice, &sliceTripCountMap) &&
            (getSliceIterationCount(sliceTripCountMap) == 1));
  };
  // Fix up and if possible, eliminate single iteration loops.
  for (AffineForOp forOp : sliceLoops) {
    if (isLoopParallelAndContainsReduction(forOp) &&
        isInnermostSiblingInsertion && srcIsUnitSlice())
      // Patch reduction loop - only ones that are sibling-fused with the
      // destination loop - into the parent loop.
      (void)promoteSingleIterReductionLoop(forOp, true);
    else
      // Promote any single iteration slice loops.
      (void)promoteIfSingleIteration(forOp);
  }
}

```
- **EN**: Implements logic around `buildSliceTripCountMap`, `getSliceIterationCount`, `isLoopParallelAndContainsReduction`, `srcIsUnitSlice`, and 2 more symbols.
- **CN**: 围绕 `buildSliceTripCountMap`, `getSliceIterationCount`, `isLoopParallelAndContainsReduction`, `srcIsUnitSlice`, and 2 more symbols 实现具体逻辑。

### Lines 471-487
```cpp
/// Collect loop nest statistics (eg. loop trip count and operation count)
/// in 'stats' for loop nest rooted at 'forOp'. Returns true on success,
/// returns false otherwise.
bool mlir::affine::getLoopNestStats(AffineForOp forOpRoot,
                                    LoopNestStats *stats) {
  auto walkResult = forOpRoot.walk([&](AffineForOp forOp) {
    auto *childForOp = forOp.getOperation();
    auto *parentForOp = forOp->getParentOp();
    if (forOp != forOpRoot) {
      if (!isa<AffineForOp>(parentForOp)) {
        LDBG() << "Expected parent AffineForOp";
        return WalkResult::interrupt();
      }
      // Add mapping to 'forOp' from its parent AffineForOp.
      stats->loopMap[parentForOp].push_back(forOp);
    }

```
- **EN**: Implements logic around `getLoopNestStats`, `walk`, `getOperation`, `getParentOp`, and 4 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getLoopNestStats`, `walk`, `getOperation`, `getParentOp`, and 4 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 488-505
```cpp
    // Record the number of op operations in the body of 'forOp'.
    unsigned count = 0;
    stats->opCountMap[childForOp] = 0;
    for (auto &op : *forOp.getBody()) {
      if (!isa<AffineForOp, AffineIfOp>(op))
        ++count;
    }
    stats->opCountMap[childForOp] = count;

    // Record trip count for 'forOp'. Set flag if trip count is not
    // constant.
    std::optional<uint64_t> maybeConstTripCount = getConstantTripCount(forOp);
    if (!maybeConstTripCount) {
      // Currently only constant trip count loop nests are supported.
      LDBG() << "Non-constant trip count unsupported";
      return WalkResult::interrupt();
    }

```
- **EN**: Implements logic around `getBody`, `AffineIfOp>`, `getConstantTripCount`, `LDBG`, and 1 more symbols.
- **CN**: 围绕 `getBody`, `AffineIfOp>`, `getConstantTripCount`, `LDBG`, and 1 more symbols 实现具体逻辑。

### Lines 506-525
```cpp
    stats->tripCountMap[childForOp] = *maybeConstTripCount;
    return WalkResult::advance();
  });
  return !walkResult.wasInterrupted();
}

// Computes the total cost of the loop nest rooted at 'forOp'.
// Currently, the total cost is computed by counting the total operation
// instance count (i.e. total number of operations in the loop bodyloop
// operation count * loop trip count) for the entire loop nest.
// If 'tripCountOverrideMap' is non-null, overrides the trip count for loops
// specified in the map when computing the total op instance count.
// NOTEs: 1) This is used to compute the cost of computation slices, which are
// sliced along the iteration dimension, and thus reduce the trip count.
// If 'computeCostMap' is non-null, the total op count for forOps specified
// in the map is increased (not overridden) by adding the op count from the
// map to the existing op count for the for loop. This is done before
// multiplying by the loop's trip count, and is used to model the cost of
// inserting a sliced loop nest of known cost into the loop's body.
// 2) This is also used to compute the cost of fusing a slice of some loop nest
```
- **EN**: Implements logic around `advance`, `wasInterrupted`.
- **CN**: 围绕 `advance`, `wasInterrupted` 实现具体逻辑。

### Lines 526-545
```cpp
// within another loop.
static int64_t getComputeCostHelper(
    Operation *forOp, LoopNestStats &stats,
    llvm::SmallDenseMap<Operation *, uint64_t, 8> *tripCountOverrideMap,
    DenseMap<Operation *, int64_t> *computeCostMap) {
  // 'opCount' is the total number operations in one iteration of 'forOp' body,
  // minus terminator op which is a no-op.
  int64_t opCount = stats.opCountMap[forOp] - 1;
  if (stats.loopMap.count(forOp) > 0) {
    for (auto childForOp : stats.loopMap[forOp]) {
      opCount += getComputeCostHelper(childForOp, stats, tripCountOverrideMap,
                                      computeCostMap);
    }
  }
  // Add in additional op instances from slice (if specified in map).
  if (computeCostMap) {
    auto it = computeCostMap->find(forOp);
    if (it != computeCostMap->end()) {
      opCount += it->second;
    }
```
- **EN**: Implements logic around `getComputeCostHelper`, `count`, `find`, `end`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getComputeCostHelper`, `count`, `find`, `end` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 546-558
```cpp
  }
  // Override trip count (if specified in map).
  int64_t tripCount = stats.tripCountMap[forOp];
  if (tripCountOverrideMap) {
    auto it = tripCountOverrideMap->find(forOp);
    if (it != tripCountOverrideMap->end()) {
      tripCount = it->second;
    }
  }
  // Returns the total number of dynamic instances of operations in loop body.
  return tripCount * opCount;
}

```
- **EN**: Implements logic around `find`, `end`.
- **CN**: 围绕 `find`, `end` 实现具体逻辑。

### Lines 559-568
```cpp
/// Computes the total cost of the loop nest rooted at 'forOp' using 'stats'.
/// Currently, the total cost is computed by counting the total operation
/// instance count (i.e. total number of operations in the loop body * loop
/// trip count) for the entire loop nest.
int64_t mlir::affine::getComputeCost(AffineForOp forOp, LoopNestStats &stats) {
  return getComputeCostHelper(forOp, stats,
                              /*tripCountOverrideMap=*/nullptr,
                              /*computeCostMap=*/nullptr);
}

```
- **EN**: Implements logic around `getComputeCost`, `getComputeCostHelper`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getComputeCost`, `getComputeCostHelper` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 569-582
```cpp
/// Computes and returns in 'computeCost', the total compute cost of fusing the
/// 'slice' of the loop nest rooted at 'srcForOp' into 'dstForOp'. Currently,
/// the total cost is computed by counting the total operation instance count
/// (i.e. total number of operations in the loop body * loop trip count) for
/// the entire loop nest.
bool mlir::affine::getFusionComputeCost(AffineForOp srcForOp,
                                        LoopNestStats &srcStats,
                                        AffineForOp dstForOp,
                                        LoopNestStats &dstStats,
                                        const ComputationSliceState &slice,
                                        int64_t *computeCost) {
  llvm::SmallDenseMap<Operation *, uint64_t, 8> sliceTripCountMap;
  DenseMap<Operation *, int64_t> computeCostMap;

```
- **EN**: Implements logic around `getFusionComputeCost`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getFusionComputeCost` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 583-594
```cpp
  // Build trip count map for computation slice.
  if (!buildSliceTripCountMap(slice, &sliceTripCountMap))
    return false;
  // Checks whether a store to load forwarding will happen.
  uint64_t sliceIterationCount = getSliceIterationCount(sliceTripCountMap);
  // It's possible it's zero due to an overflow and a wraparound; being a cost
  // model, we fail.
  if (sliceIterationCount == 0)
    return false;
  bool storeLoadFwdGuaranteed = (sliceIterationCount == 1);
  auto *insertPointParent = slice.insertPoint->getParentOp();

```
- **EN**: Implements logic around `buildSliceTripCountMap`, `getSliceIterationCount`, `getParentOp`.
- **CN**: 围绕 `buildSliceTripCountMap`, `getSliceIterationCount`, `getParentOp` 实现具体逻辑。

### Lines 595-614
```cpp
  // The store and loads to this memref will disappear.
  if (storeLoadFwdGuaranteed) {
    // Subtract from operation count the loads/store we expect load/store
    // forwarding to remove.
    unsigned storeCount = 0;
    llvm::SmallDenseSet<Value, 4> storeMemrefs;
    srcForOp.walk([&](AffineWriteOpInterface storeOp) {
      storeMemrefs.insert(storeOp.getMemRef());
      ++storeCount;
    });
    // Subtract out any store ops in single-iteration src slice loop nest.
    if (storeCount > 0)
      computeCostMap[insertPointParent] = -storeCount;
    // Subtract out any load users of 'storeMemrefs' nested below
    // 'insertPointParent'.
    for (Value memref : storeMemrefs) {
      for (Operation *user : memref.getUsers()) {
        if (!isa<AffineReadOpInterface>(user))
          continue;
        SmallVector<AffineForOp, 4> loops;
```
- **EN**: Implements logic around `walk`, `insert`, `getUsers`, `isa`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `walk`, `insert`, `getUsers`, `isa` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 615-625
```cpp
        // Check if any loop in loop nest surrounding 'user' is
        // 'insertPointParent'.
        getAffineForIVs(*user, &loops);
        if (llvm::is_contained(loops, cast<AffineForOp>(insertPointParent))) {
          if (auto forOp = dyn_cast_or_null<AffineForOp>(user->getParentOp()))
            --computeCostMap[forOp];
        }
      }
    }
  }

```
- **EN**: Implements logic around `getAffineForIVs`, `is_contained`, `dyn_cast_or_null`.
- **CN**: 围绕 `getAffineForIVs`, `is_contained`, `dyn_cast_or_null` 实现具体逻辑。

### Lines 626-638
```cpp
  // Compute op instance count for the src loop nest with iteration slicing.
  int64_t sliceComputeCost = getComputeCostHelper(
      srcForOp, srcStats, &sliceTripCountMap, &computeCostMap);

  // Compute cost of fusion for this depth.
  computeCostMap[insertPointParent] = sliceComputeCost;

  *computeCost =
      getComputeCostHelper(dstForOp, dstStats,
                           /*tripCountOverrideMap=*/nullptr, &computeCostMap);
  return true;
}

```
- **EN**: Implements logic around `getComputeCostHelper`.
- **CN**: 围绕 `getComputeCostHelper` 实现具体逻辑。

### Lines 639-650
```cpp
/// Returns in 'producerConsumerMemrefs' the memrefs involved in a
/// producer-consumer dependence between write ops in 'srcOps' and read ops in
/// 'dstOps'.
void mlir::affine::gatherProducerConsumerMemrefs(
    ArrayRef<Operation *> srcOps, ArrayRef<Operation *> dstOps,
    DenseSet<Value> &producerConsumerMemrefs) {
  // Gather memrefs from stores in 'srcOps'.
  DenseSet<Value> srcStoreMemRefs;
  for (Operation *op : srcOps)
    if (auto storeOp = dyn_cast<AffineWriteOpInterface>(op))
      srcStoreMemRefs.insert(storeOp.getMemRef());

```
- **EN**: Implements logic around `gatherProducerConsumerMemrefs`, `dyn_cast`, `insert`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `gatherProducerConsumerMemrefs`, `dyn_cast`, `insert` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 651-657
```cpp
  // Compute the intersection between memrefs from stores in 'srcOps' and
  // memrefs from loads in 'dstOps'.
  for (Operation *op : dstOps)
    if (auto loadOp = dyn_cast<AffineReadOpInterface>(op))
      if (srcStoreMemRefs.count(loadOp.getMemRef()) > 0)
        producerConsumerMemrefs.insert(loadOp.getMemRef());
}
```
- **EN**: Implements logic around `dyn_cast`, `count`, `insert`.
- **CN**: 围绕 `dyn_cast`, `count`, `insert` 实现具体逻辑。

## Key Concepts / 关键概念

- **Utility helpers / 辅助工具**:
  - **EN**: Provides reusable helpers that keep dialect implementations and passes smaller.
  - **CN**: 提供可复用的辅助函数，使方言实现和 pass 保持精简。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
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
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics or assembly-like textual forms through LLVM/MLIR stream APIs.
  - **CN**: 通过 LLVM/MLIR 流式 API 输出诊断或类汇编文本。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/LoopFusionUtils.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`, `mlir/IR/IRMapping.h`, `mlir/IR/Operation.h` ... (+4 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), MLIR IR core abstractions / MLIR IR 核心抽象 (3), LLVM support-library helpers / LLVM Support 库辅助功能 (3), MLIR analysis interfaces / MLIR 分析接口 (2)
