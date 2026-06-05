# PipelineDataTransfer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/PipelineDataTransfer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass to pipeline data transfers.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- PipelineDataTransfer.cpp --- Pass for pipelining data movement ---*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a pass to pipeline data transfers.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-26
```cpp

#include "mlir/Dialect/Affine/Transforms/Passes.h"

#include "mlir/Dialect/Affine/Analysis/AffineAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/LoopAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/Utils.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/LoopUtils.h"
#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/IR/Builders.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`。

### Lines 27-38
```cpp
namespace mlir {
namespace affine {
#define GEN_PASS_DEF_AFFINEPIPELINEDATATRANSFER
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

#define DEBUG_TYPE "affine-pipeline-data-transfer"

using namespace mlir;
using namespace mlir::affine;

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 39-48
```cpp
namespace {
struct PipelineDataTransfer
    : public affine::impl::AffinePipelineDataTransferBase<
          PipelineDataTransfer> {
  void runOnOperation() override;
  void runOnAffineForOp(AffineForOp forOp);

  std::vector<AffineForOp> forOps;
};

```
- **EN**: Introduces declarations for `PipelineDataTransfer`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PipelineDataTransfer` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 49-68
```cpp
} // namespace

/// Creates a pass to pipeline explicit movement of data across levels of the
/// memory hierarchy.
std::unique_ptr<OperationPass<func::FuncOp>>
mlir::affine::createPipelineDataTransferPass() {
  return std::make_unique<PipelineDataTransfer>();
}

// Returns the position of the tag memref operand given a DMA operation.
// Temporary utility: will be replaced when DmaStart/DmaFinish abstract op's are
// added.
static unsigned getTagMemRefPos(Operation &dmaOp) {
  assert((isa<AffineDmaStartOp, AffineDmaWaitOp>(dmaOp)));
  if (auto dmaStartOp = dyn_cast<AffineDmaStartOp>(dmaOp)) {
    return dmaStartOp.getTagMemRefOperandIndex();
  }
  // First operand for a dma finish operation.
  return 0;
}
```
- **EN**: Implements logic around `createPipelineDataTransferPass`, `make_unique`, `getTagMemRefPos`, `assert`, and 2 more symbols; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers; works with symbol tables or function-like operations.
- **CN**: 围绕 `createPipelineDataTransferPass`, `make_unique`, `getTagMemRefPos`, `assert`, and 2 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据，并处理符号表或类函数操作。

### Lines 69-78
```cpp

/// Doubles the buffer of the supplied memref on the specified 'affine.for'
/// operation by adding a leading dimension of size two to the memref.
/// Replaces all uses of the old memref by the new one while indexing the newly
/// added dimension by the loop IV of the specified 'affine.for' operation
/// modulo 2. Returns false if such a replacement cannot be performed.
static bool doubleBuffer(Value oldMemRef, AffineForOp forOp) {
  auto *forBody = forOp.getBody();
  OpBuilder bInner(forBody, forBody->begin());

```
- **EN**: Implements logic around `doubleBuffer`, `getBody`, `bInner`; this block performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `doubleBuffer`, `getBody`, `bInner` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 79-88
```cpp
  // Doubles the shape with a leading dimension extent of 2.
  auto doubleShape = [&](MemRefType oldMemRefType) -> MemRefType {
    // Add the leading dimension in the shape for the double buffer.
    ArrayRef<int64_t> oldShape = oldMemRefType.getShape();
    SmallVector<int64_t, 4> newShape(1 + oldMemRefType.getRank());
    newShape[0] = 2;
    llvm::copy(oldShape, newShape.begin() + 1);
    return MemRefType::Builder(oldMemRefType).setShape(newShape).setLayout({});
  };

```
- **EN**: Implements logic around `getShape`, `newShape`, `copy`, `Builder`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getShape`, `newShape`, `copy`, `Builder` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 89-101
```cpp
  auto oldMemRefType = cast<MemRefType>(oldMemRef.getType());
  auto newMemRefType = doubleShape(oldMemRefType);

  // The double buffer is allocated right before 'forOp'.
  OpBuilder bOuter(forOp);
  // Put together alloc operands for any dynamic dimensions of the memref.
  SmallVector<Value, 4> allocOperands;
  for (const auto &dim : llvm::enumerate(oldMemRefType.getShape())) {
    if (dim.value() == ShapedType::kDynamic)
      allocOperands.push_back(bOuter.createOrFold<memref::DimOp>(
          forOp.getLoc(), oldMemRef, dim.index()));
  }

```
- **EN**: Implements logic around `cast`, `doubleShape`, `bOuter`, `enumerate`, and 3 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `cast`, `doubleShape`, `bOuter`, `enumerate`, and 3 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 102-113
```cpp
  // Create and place the alloc right before the 'affine.for' operation.
  Value newMemRef = memref::AllocOp::create(bOuter, forOp.getLoc(),
                                            newMemRefType, allocOperands);

  // Create 'iv mod 2' value to index the leading dimension.
  auto d0 = bInner.getAffineDimExpr(0);
  int64_t step = forOp.getStepAsInt();
  auto modTwoMap =
      AffineMap::get(/*dimCount=*/1, /*symbolCount=*/0, d0.floorDiv(step) % 2);
  auto ivModTwoOp = AffineApplyOp::create(bInner, forOp.getLoc(), modTwoMap,
                                          forOp.getInductionVar());

```
- **EN**: Implements logic around `create`, `getAffineDimExpr`, `getStepAsInt`, `get`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `create`, `getAffineDimExpr`, `getStepAsInt`, `get`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 114-133
```cpp
  // replaceAllMemRefUsesWith will succeed unless the forOp body has
  // non-dereferencing uses of the memref (dealloc's are fine though).
  auto userFilterFn = [&](Operation *user) {
    auto domInfo = std::make_unique<DominanceInfo>(
        forOp->getParentOfType<FunctionOpInterface>());
    return domInfo->dominates(&*forOp.getBody()->begin(), user);
  };
  if (failed(replaceAllMemRefUsesWith(oldMemRef, newMemRef,
                                      /*extraIndices=*/{ivModTwoOp},
                                      /*indexRemap=*/AffineMap(),
                                      /*extraOperands=*/{},
                                      /*symbolOperands=*/{}, userFilterFn))) {
    LLVM_DEBUG(
        forOp.emitError("memref replacement for double buffering failed"));
    ivModTwoOp.erase();
    return false;
  }
  // Insert the dealloc op right after the for loop.
  bOuter.setInsertionPointAfter(forOp);
  memref::DeallocOp::create(bOuter, forOp.getLoc(), newMemRef);
```
- **EN**: Implements logic around `make_unique`, `getParentOfType`, `dominates`, `failed`, and 5 more symbols; this block performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `make_unique`, `getParentOfType`, `dominates`, `failed`, and 5 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 134-150
```cpp

  return true;
}

/// Returns success if the IR is in a valid state.
void PipelineDataTransfer::runOnOperation() {
  // Do a post order walk so that inner loop DMAs are processed first. This is
  // necessary since 'affine.for' operations nested within would otherwise
  // become invalid (erased) when the outer loop is pipelined (the pipelined one
  // gets deleted and replaced by a prologue, a new steady-state loop and an
  // epilogue).
  forOps.clear();
  getOperation().walk([&](AffineForOp forOp) { forOps.push_back(forOp); });
  for (auto forOp : forOps)
    runOnAffineForOp(forOp);
}

```
- **EN**: Implements logic around `runOnOperation`, `clear`, `getOperation`, `runOnAffineForOp`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `runOnOperation`, `clear`, `getOperation`, `runOnAffineForOp` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理。

### Lines 151-170
```cpp
// Check if tags of the dma start op and dma wait op match.
static bool checkTagMatch(AffineDmaStartOp startOp, AffineDmaWaitOp waitOp) {
  if (startOp.getTagMemRef() != waitOp.getTagMemRef())
    return false;
  auto startIndices = startOp.getTagIndices();
  auto waitIndices = waitOp.getTagIndices();
  // Both of these have the same number of indices since they correspond to the
  // same tag memref.
  for (auto it = startIndices.begin(), wIt = waitIndices.begin(),
            e = startIndices.end();
       it != e; ++it, ++wIt) {
    // Keep it simple for now, just checking if indices match.
    // TODO: this would in general need to check if there is no
    // intervening write writing to the same tag location, i.e., memory last
    // write/data flow analysis. This is however sufficient/powerful enough for
    // now since the DMA generation pass or the input for it will always have
    // start/wait with matching tags (same SSA operand indices).
    if (*it != *wIt)
      return false;
  }
```
- **EN**: Implements logic around `checkTagMatch`, `getTagMemRef`, `getTagIndices`, `begin`, and 1 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `checkTagMatch`, `getTagMemRef`, `getTagIndices`, `begin`, and 1 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 171-186
```cpp
  return true;
}

// Identify matching DMA start/finish operations to overlap computation with.
static void findMatchingStartFinishInsts(
    AffineForOp forOp,
    SmallVectorImpl<std::pair<Operation *, Operation *>> &startWaitPairs) {

  // Collect outgoing DMA operations - needed to check for dependences below.
  SmallVector<AffineDmaStartOp, 4> outgoingDmaOps;
  for (auto &op : *forOp.getBody()) {
    auto dmaStartOp = dyn_cast<AffineDmaStartOp>(op);
    if (dmaStartOp && dmaStartOp.isSrcMemorySpaceFaster())
      outgoingDmaOps.push_back(dmaStartOp);
  }

```
- **EN**: Implements logic around `findMatchingStartFinishInsts`, `getBody`, `dyn_cast`, `isSrcMemorySpaceFaster`, and 1 more symbols.
- **CN**: 围绕 `findMatchingStartFinishInsts`, `getBody`, `dyn_cast`, `isSrcMemorySpaceFaster`, and 1 more symbols 实现具体逻辑。

### Lines 187-197
```cpp
  SmallVector<Operation *, 4> dmaStartInsts, dmaFinishInsts;
  for (auto &op : *forOp.getBody()) {
    // Collect DMA finish operations.
    if (isa<AffineDmaWaitOp>(op)) {
      dmaFinishInsts.push_back(&op);
      continue;
    }
    auto dmaStartOp = dyn_cast<AffineDmaStartOp>(op);
    if (!dmaStartOp)
      continue;

```
- **EN**: Implements logic around `getBody`, `isa`, `push_back`, `dyn_cast`.
- **CN**: 围绕 `getBody`, `isa`, `push_back`, `dyn_cast` 实现具体逻辑。

### Lines 198-213
```cpp
    // Only DMAs incoming into higher memory spaces are pipelined for now.
    // TODO: handle outgoing DMA pipelining.
    if (!dmaStartOp.isDestMemorySpaceFaster())
      continue;

    // Check for dependence with outgoing DMAs. Doing this conservatively.
    // TODO: use the dependence analysis to check for
    // dependences between an incoming and outgoing DMA in the same iteration.
    auto *it = outgoingDmaOps.begin();
    for (; it != outgoingDmaOps.end(); ++it) {
      if (it->getDstMemRef() == dmaStartOp.getSrcMemRef())
        break;
    }
    if (it != outgoingDmaOps.end())
      continue;

```
- **EN**: Implements logic around `isDestMemorySpaceFaster`, `begin`, `end`, `getDstMemRef`.
- **CN**: 围绕 `isDestMemorySpaceFaster`, `begin`, `end`, `getDstMemRef` 实现具体逻辑。

### Lines 214-231
```cpp
    // We only double buffer if the buffer is not live out of loop.
    auto memref = dmaStartOp.getOperand(dmaStartOp.getFasterMemPos());
    bool escapingUses = false;
    for (auto *user : memref.getUsers()) {
      // We can double buffer regardless of dealloc's outside the loop.
      if (isa<memref::DeallocOp>(user))
        continue;
      if (!forOp.getBody()->findAncestorOpInBlock(*user)) {
        LLVM_DEBUG(llvm::dbgs()
                       << "can't pipeline: buffer is live out of loop\n";);
        escapingUses = true;
        break;
      }
    }
    if (!escapingUses)
      dmaStartInsts.push_back(&op);
  }

```
- **EN**: Implements logic around `getOperand`, `getUsers`, `DeallocOp>`, `getBody`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getOperand`, `getUsers`, `DeallocOp>`, `getBody`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 232-243
```cpp
  // For each start operation, we look for a matching finish operation.
  for (auto *dmaStartOp : dmaStartInsts) {
    for (auto *dmaFinishOp : dmaFinishInsts) {
      if (checkTagMatch(cast<AffineDmaStartOp>(dmaStartOp),
                        cast<AffineDmaWaitOp>(dmaFinishOp))) {
        startWaitPairs.push_back({dmaStartOp, dmaFinishOp});
        break;
      }
    }
  }
}

```
- **EN**: Implements logic around `checkTagMatch`, `cast`, `push_back`.
- **CN**: 围绕 `checkTagMatch`, `cast`, `push_back` 实现具体逻辑。

### Lines 244-253
```cpp
/// Overlap DMA transfers with computation in this loop. If successful,
/// 'forOp' is deleted, and a prologue, a new pipelined loop, and epilogue are
/// inserted right before where it was.
void PipelineDataTransfer::runOnAffineForOp(AffineForOp forOp) {
  auto mayBeConstTripCount = getConstantTripCount(forOp);
  if (!mayBeConstTripCount) {
    LLVM_DEBUG(forOp.emitRemark("won't pipeline due to unknown trip count"));
    return;
  }

```
- **EN**: Implements logic around `runOnAffineForOp`, `getConstantTripCount`, `emitRemark`.
- **CN**: 围绕 `runOnAffineForOp`, `getConstantTripCount`, `emitRemark` 实现具体逻辑。

### Lines 254-273
```cpp
  SmallVector<std::pair<Operation *, Operation *>, 4> startWaitPairs;
  findMatchingStartFinishInsts(forOp, startWaitPairs);

  if (startWaitPairs.empty()) {
    LLVM_DEBUG(forOp.emitRemark("No dma start/finish pairs\n"));
    return;
  }

  // Double the buffers for the higher memory space memref's.
  // Identify memref's to replace by scanning through all DMA start
  // operations. A DMA start operation has two memref's - the one from the
  // higher level of memory hierarchy is the one to double buffer.
  // TODO: check whether double-buffering is even necessary.
  // TODO: make this work with different layouts: assuming here that
  // the dimension we are adding here for the double buffering is the outermost
  // dimension.
  for (auto &pair : startWaitPairs) {
    auto *dmaStartOp = pair.first;
    Value oldMemRef = dmaStartOp->getOperand(
        cast<AffineDmaStartOp>(dmaStartOp).getFasterMemPos());
```
- **EN**: Implements logic around `findMatchingStartFinishInsts`, `empty`, `emitRemark`, `getOperand`, and 1 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `findMatchingStartFinishInsts`, `empty`, `emitRemark`, `getOperand`, and 1 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 274-293
```cpp
    if (!doubleBuffer(oldMemRef, forOp)) {
      // Normally, double buffering should not fail because we already checked
      // that there are no uses outside.
      LLVM_DEBUG(llvm::dbgs()
                     << "double buffering failed for" << dmaStartOp << "\n";);
      // IR still valid and semantically correct.
      return;
    }
    // If the old memref has no more uses, remove its 'dead' alloc if it was
    // alloc'ed. (note: DMA buffers are rarely function live-in; but a 'dim'
    // operation could have been used on it if it was dynamically shaped in
    // order to create the double buffer above.)
    // '-canonicalize' does this in a more general way, but we'll anyway do the
    // simple/common case so that the output / test cases looks clear.
    if (auto *allocOp = oldMemRef.getDefiningOp()) {
      if (oldMemRef.use_empty()) {
        allocOp->erase();
      } else if (oldMemRef.hasOneUse()) {
        if (auto dealloc =
                dyn_cast<memref::DeallocOp>(*oldMemRef.user_begin())) {
```
- **EN**: Implements logic around `doubleBuffer`, `dbgs`, `getDefiningOp`, `use_empty`, and 3 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `doubleBuffer`, `dbgs`, `getDefiningOp`, `use_empty`, and 3 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 294-313
```cpp
          dealloc.erase();
          allocOp->erase();
        }
      }
    }
  }

  // Double the buffers for tag memrefs.
  for (auto &pair : startWaitPairs) {
    auto *dmaFinishOp = pair.second;
    Value oldTagMemRef = dmaFinishOp->getOperand(getTagMemRefPos(*dmaFinishOp));
    if (!doubleBuffer(oldTagMemRef, forOp)) {
      LLVM_DEBUG(llvm::dbgs() << "tag double buffering failed\n";);
      return;
    }
    // If the old tag has no uses or a single dealloc use, remove it.
    // (canonicalization handles more complex cases).
    if (auto *tagAllocOp = oldTagMemRef.getDefiningOp()) {
      if (oldTagMemRef.use_empty()) {
        tagAllocOp->erase();
```
- **EN**: Implements logic around `erase`, `getOperand`, `doubleBuffer`, `dbgs`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `erase`, `getOperand`, `doubleBuffer`, `dbgs`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 314-323
```cpp
      } else if (oldTagMemRef.hasOneUse()) {
        if (auto dealloc =
                dyn_cast<memref::DeallocOp>(*oldTagMemRef.user_begin())) {
          dealloc.erase();
          tagAllocOp->erase();
        }
      }
    }
  }

```
- **EN**: Implements logic around `hasOneUse`, `DeallocOp>`, `erase`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `hasOneUse`, `DeallocOp>`, `erase` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 324-343
```cpp
  // Double buffering would have invalidated all the old DMA start/wait insts.
  startWaitPairs.clear();
  findMatchingStartFinishInsts(forOp, startWaitPairs);

  // Store shift for operation for later lookup for AffineApplyOp's.
  DenseMap<Operation *, unsigned> instShiftMap;
  for (auto &pair : startWaitPairs) {
    auto *dmaStartOp = pair.first;
    assert(isa<AffineDmaStartOp>(dmaStartOp));
    instShiftMap[dmaStartOp] = 0;
    // Set shifts for DMA start op's affine operand computation slices to 0.
    SmallVector<AffineApplyOp, 4> sliceOps;
    affine::createAffineComputationSlice(dmaStartOp, &sliceOps);
    if (!sliceOps.empty()) {
      for (auto sliceOp : sliceOps) {
        instShiftMap[sliceOp.getOperation()] = 0;
      }
    } else {
      // If a slice wasn't created, the reachable affine.apply op's from its
      // operands are the ones that go with it.
```
- **EN**: Implements logic around `clear`, `findMatchingStartFinishInsts`, `assert`, `createAffineComputationSlice`, and 2 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `clear`, `findMatchingStartFinishInsts`, `assert`, `createAffineComputationSlice`, and 2 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 344-355
```cpp
      SmallVector<Operation *, 4> affineApplyInsts;
      SmallVector<Value, 4> operands(dmaStartOp->getOperands());
      getReachableAffineApplyOps(operands, affineApplyInsts);
      for (auto *op : affineApplyInsts) {
        instShiftMap[op] = 0;
      }
    }
  }
  // Everything else (including compute ops and dma finish) are shifted by one.
  for (auto &op : forOp.getBody()->without_terminator())
    instShiftMap.try_emplace(&op, 1);

```
- **EN**: Implements logic around `operands`, `getReachableAffineApplyOps`, `getBody`, `try_emplace`.
- **CN**: 围绕 `operands`, `getReachableAffineApplyOps`, `getBody`, `try_emplace` 实现具体逻辑。

### Lines 356-369
```cpp
  // Get shifts stored in map.
  SmallVector<uint64_t, 8> shifts(forOp.getBody()->getOperations().size());
  unsigned s = 0;
  for (auto &op : forOp.getBody()->without_terminator()) {
    assert(instShiftMap.contains(&op));
    shifts[s++] = instShiftMap[&op];

    // Tagging operations with shifts for debugging purposes.
    LLVM_DEBUG({
      OpBuilder b(&op);
      op.setAttr("shift", b.getI64IntegerAttr(shifts[s - 1]));
    });
  }

```
- **EN**: Implements logic around `shifts`, `getBody`, `assert`, `b`, and 1 more symbols.
- **CN**: 围绕 `shifts`, `getBody`, `assert`, `b`, and 1 more symbols 实现具体逻辑。

### Lines 370-380
```cpp
  if (!isOpwiseShiftValid(forOp, shifts)) {
    // Violates dependences.
    LLVM_DEBUG(llvm::dbgs() << "Shifts invalid - unexpected\n";);
    return;
  }

  if (failed(affineForOpBodySkew(forOp, shifts))) {
    LLVM_DEBUG(llvm::dbgs() << "op body skewing failed - unexpected\n";);
    return;
  }
}
```
- **EN**: Implements logic around `isOpwiseShiftValid`, `dbgs`, `failed`.
- **CN**: 围绕 `isOpwiseShiftValid`, `dbgs`, `failed` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
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

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/IR/Builders.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (10), MLIR IR core abstractions / MLIR IR 核心抽象 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
