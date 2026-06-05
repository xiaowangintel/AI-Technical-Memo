# OpenACCUtilsTiling.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Utils/OpenACCUtilsTiling.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains utility functions for tiling OpenACC loops.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Utils`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- OpenACCUtilsTiling.cpp - OpenACC Loop Tiling Utilities -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains utility functions for tiling OpenACC loops.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-31
```cpp

#include "mlir/Dialect/OpenACC/OpenACCUtilsTiling.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/Transforms/RegionUtils.h"

// Resolve unknown tile sizes (represented as -1 for tile(*)) to the default.
// Returns a value with the same type as targetType.
static mlir::Value resolveAndCastTileSize(mlir::Value tileSize,
                                          int32_t defaultTileSize,
                                          mlir::Type targetType,
                                          mlir::RewriterBase &rewriter,
                                          mlir::Location loc) {
  auto constVal = mlir::getConstantIntValue(tileSize);
  if (constVal && *constVal < 0) {
    // Create constant with the target type directly
    return mlir::arith::ConstantOp::create(
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/OpenACCUtilsTiling.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/OpenACC/OpenACC.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/OpenACCUtilsTiling.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/OpenACC/OpenACC.h`。

### Lines 32-49
```cpp
        rewriter, loc, targetType,
        rewriter.getIntegerAttr(targetType, defaultTileSize));
  }
  return mlir::getValueOrCreateCastToIndexLike(rewriter, loc, targetType,
                                               tileSize);
}

// Remove vector/worker attributes from loop
static void removeWorkerVectorFromLoop(mlir::acc::LoopOp loop) {
  if (loop.hasVector() || loop.getVectorValue()) {
    loop.removeVectorAttr();
    loop.removeVectorOperandsDeviceTypeAttr();
  } else if (loop.hasWorker() || loop.getWorkerValue()) {
    loop.removeWorkerAttr();
    loop.removeWorkerNumOperandsDeviceTypeAttr();
  }
}

```
- **EN**: Implements logic around `getIntegerAttr`, `getValueOrCreateCastToIndexLike`, `removeWorkerVectorFromLoop`, `hasVector`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIntegerAttr`, `getValueOrCreateCastToIndexLike`, `removeWorkerVectorFromLoop`, `hasVector`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 50-69
```cpp
// Create a new ACC loop with new steps, lb, ub from original loop
static mlir::acc::LoopOp
createACCLoopFromOriginal(mlir::acc::LoopOp origLoop,
                          mlir::RewriterBase &rewriter, mlir::ValueRange lb,
                          mlir::ValueRange ub, mlir::ValueRange step,
                          mlir::DenseBoolArrayAttr inclusiveUBAttr,
                          mlir::acc::CombinedConstructsTypeAttr combinedAttr,
                          mlir::Location loc, bool preserveCollapse) {
  mlir::ArrayAttr collapseAttr = mlir::ArrayAttr{};
  mlir::ArrayAttr collapseDeviceTypeAttr = mlir::ArrayAttr{};
  if (preserveCollapse) {
    collapseAttr = origLoop.getCollapseAttr();
    collapseDeviceTypeAttr = origLoop.getCollapseDeviceTypeAttr();
  }
  auto newLoop = mlir::acc::LoopOp::create(
      rewriter, loc, origLoop->getResultTypes(), lb, ub, step, inclusiveUBAttr,
      collapseAttr, collapseDeviceTypeAttr, origLoop.getGangOperands(),
      origLoop.getGangOperandsArgTypeAttr(),
      origLoop.getGangOperandsSegmentsAttr(),
      origLoop.getGangOperandsDeviceTypeAttr(), origLoop.getWorkerNumOperands(),
```
- **EN**: Implements logic around `createACCLoopFromOriginal`, `getCollapseAttr`, `getCollapseDeviceTypeAttr`, `create`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createACCLoopFromOriginal`, `getCollapseAttr`, `getCollapseDeviceTypeAttr`, `create`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 70-80
```cpp
      origLoop.getWorkerNumOperandsDeviceTypeAttr(),
      origLoop.getVectorOperands(), origLoop.getVectorOperandsDeviceTypeAttr(),
      origLoop.getSeqAttr(), origLoop.getIndependentAttr(),
      origLoop.getAuto_Attr(), origLoop.getGangAttr(), origLoop.getWorkerAttr(),
      origLoop.getVectorAttr(), mlir::ValueRange{}, mlir::DenseI32ArrayAttr{},
      mlir::ArrayAttr{}, origLoop.getCacheOperands(),
      origLoop.getPrivateOperands(), origLoop.getFirstprivateOperands(),
      origLoop.getReductionOperands(), combinedAttr);
  return newLoop;
}

```
- **EN**: Implements logic around `getWorkerNumOperandsDeviceTypeAttr`, `getVectorOperands`, `getSeqAttr`, `getAuto_Attr`, and 4 more symbols.
- **CN**: 围绕 `getWorkerNumOperandsDeviceTypeAttr`, `getVectorOperands`, `getSeqAttr`, `getAuto_Attr`, and 4 more symbols 实现具体逻辑。

### Lines 81-100
```cpp
// Create inner loop inside input loop
static mlir::acc::LoopOp
createInnerLoop(mlir::acc::LoopOp inputLoop, mlir::RewriterBase &rewriter,
                mlir::ValueRange lb, mlir::ValueRange ub, mlir::ValueRange step,
                mlir::DenseBoolArrayAttr inclusiveUBAttr, mlir::Location loc) {
  mlir::acc::LoopOp elementLoop = createACCLoopFromOriginal(
      inputLoop, rewriter, lb, ub, step, inclusiveUBAttr,
      mlir::acc::CombinedConstructsTypeAttr{}, loc, /*preserveCollapse*/ false);

  // Remove gang/worker attributes from inner loops
  rewriter.startOpModification(elementLoop);
  if (inputLoop.hasGang() ||
      inputLoop.getGangValue(mlir::acc::GangArgType::Num) ||
      inputLoop.getGangValue(mlir::acc::GangArgType::Dim) ||
      inputLoop.getGangValue(mlir::acc::GangArgType::Static)) {
    elementLoop.removeGangAttr();
    elementLoop.removeGangOperandsArgTypeAttr();
    elementLoop.removeGangOperandsSegmentsAttr();
    elementLoop.removeGangOperandsDeviceTypeAttr();
  }
```
- **EN**: Implements logic around `createInnerLoop`, `createACCLoopFromOriginal`, `startOpModification`, `hasGang`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createInnerLoop`, `createACCLoopFromOriginal`, `startOpModification`, `hasGang`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 101-114
```cpp
  if (inputLoop.hasVector() || inputLoop.getVectorValue()) {
    elementLoop.removeWorkerAttr();
    elementLoop.removeWorkerNumOperandsDeviceTypeAttr();
  }
  rewriter.finalizeOpModification(elementLoop);

  // Create empty block in elementLoop and add IV argument
  mlir::Block *blk = rewriter.createBlock(&elementLoop.getRegion(),
                                          elementLoop.getRegion().begin());
  rewriter.setInsertionPointToEnd(blk);
  mlir::acc::YieldOp::create(rewriter, loc);
  elementLoop.getBody().addArgument(
      inputLoop.getBody().getArgument(0).getType(), loc);

```
- **EN**: Implements logic around `hasVector`, `removeWorkerAttr`, `removeWorkerNumOperandsDeviceTypeAttr`, `finalizeOpModification`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasVector`, `removeWorkerAttr`, `removeWorkerNumOperandsDeviceTypeAttr`, `finalizeOpModification`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 115-128
```cpp
  return elementLoop;
}

// Move ops from source to target Loop and replace uses of IVs
static void moveOpsAndReplaceIVs(mlir::acc::LoopOp sourceLoop,
                                 mlir::acc::LoopOp targetLoop,
                                 llvm::ArrayRef<mlir::Value> newIVs,
                                 llvm::ArrayRef<mlir::Value> origIVs,
                                 size_t nOps, mlir::RewriterBase &rewriter) {
  // nOps includes the terminator; move all ops except the terminator:
  // [begin, begin + nOps - 1)
  mlir::Block::iterator begin = sourceLoop.getBody().begin();
  mlir::Block::iterator end = std::next(begin, nOps - 1);

```
- **EN**: Implements logic around `moveOpsAndReplaceIVs`, `getBody`, `next`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `moveOpsAndReplaceIVs`, `getBody`, `next` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 129-139
```cpp
  // Notify the rewriter about all ops being moved (and their nested ops).
  // Directly moved ops have their parent block changed (rewriter fingerprint
  // tracking invalidated). Nested ops may have operands replaced by
  // replaceAllUsesInRegionWith below.
  llvm::SmallVector<mlir::Operation *> movedOps;
  for (mlir::Block::iterator it = begin; it != end; ++it)
    it->walk([&](mlir::Operation *op) {
      movedOps.push_back(op);
      rewriter.startOpModification(op);
    });

```
- **EN**: Implements logic around `walk`, `push_back`, `startOpModification`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `walk`, `push_back`, `startOpModification` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 140-151
```cpp
  targetLoop.getBody().getOperations().splice(
      targetLoop.getBody().getOperations().begin(),
      sourceLoop.getBody().getOperations(), begin, end);

  // Replace uses of origIV with newIV
  for (auto [i, newIV] : llvm::enumerate(newIVs))
    mlir::replaceAllUsesInRegionWith(origIVs[i], newIV, targetLoop.getRegion());

  for (mlir::Operation *op : movedOps)
    rewriter.finalizeOpModification(op);
}

```
- **EN**: Implements logic around `getBody`, `enumerate`, `replaceAllUsesInRegionWith`, `finalizeOpModification`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getBody`, `enumerate`, `replaceAllUsesInRegionWith`, `finalizeOpModification` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 152-168
```cpp
mlir::acc::LoopOp
mlir::acc::tileACCLoops(llvm::SmallVector<mlir::acc::LoopOp> &tileLoops,
                        const llvm::SmallVector<mlir::Value> &tileSizes,
                        int32_t defaultTileSize, mlir::RewriterBase &rewriter) {
  // Tile collapsed and/or nested loops
  mlir::acc::LoopOp outerLoop = tileLoops[0];
  const mlir::Location loc = outerLoop.getLoc();

  mlir::acc::LoopOp innerLoop = tileLoops[tileLoops.size() - 1];
  llvm::SmallVector<mlir::Value, 3> origIVs;
  llvm::SmallVector<mlir::Value, 3> origSteps;
  llvm::SmallVector<mlir::Value, 3> origUBs;
  llvm::SmallVector<mlir::Value, 3> newSteps;
  llvm::SmallVector<mlir::Value, 3> newUBs;
  llvm::SmallVector<mlir::Value, 3> newIVs;
  size_t nOps = innerLoop.getBody().getOperations().size();

```
- **EN**: Implements logic around `tileACCLoops`, `getLoc`, `size`, `getBody`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `tileACCLoops`, `getLoc`, `size`, `getBody` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 169-183
```cpp
  // Extract original inclusiveUBs
  llvm::SmallVector<bool> inclusiveUBs;
  for (auto tileLoop : tileLoops) {
    for (auto [j, step] : llvm::enumerate(tileLoop.getStep())) {
      // inclusiveUBs are present on the IR from Fortran frontend for DO loops
      // but might not be present from other frontends (python)
      // So check if it exists
      if (tileLoop.getInclusiveUpperboundAttr())
        inclusiveUBs.push_back(
            tileLoop.getInclusiveUpperboundAttr().asArrayRef()[j]);
      else
        inclusiveUBs.push_back(false);
    }
  }

```
- **EN**: Implements logic around `enumerate`, `getInclusiveUpperboundAttr`, `push_back`.
- **CN**: 围绕 `enumerate`, `getInclusiveUpperboundAttr`, `push_back` 实现具体逻辑。

### Lines 184-203
```cpp
  // Extract original ivs, UBs, steps, and calculate new steps
  rewriter.setInsertionPoint(outerLoop);
  for (auto [i, tileLoop] : llvm::enumerate(tileLoops)) {
    for (auto arg : tileLoop.getBody().getArguments())
      origIVs.push_back(arg);
    for (auto ub : tileLoop.getUpperbound())
      origUBs.push_back(ub);

    llvm::SmallVector<mlir::Value, 3> currentLoopSteps;
    for (auto [j, step] : llvm::enumerate(tileLoop.getStep())) {
      origSteps.push_back(step);
      if (i + j >= tileSizes.size()) {
        currentLoopSteps.push_back(step);
      } else {
        mlir::Value tileSize = resolveAndCastTileSize(
            tileSizes[i + j], defaultTileSize, step.getType(), rewriter, loc);
        auto newLoopStep =
            mlir::arith::MulIOp::create(rewriter, loc, step, tileSize);
        currentLoopSteps.push_back(newLoopStep);
        newSteps.push_back(newLoopStep);
```
- **EN**: Implements logic around `setInsertionPoint`, `enumerate`, `getBody`, `push_back`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPoint`, `enumerate`, `getBody`, `push_back`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 204-223
```cpp
      }
    }

    rewriter.startOpModification(tileLoop);
    tileLoop.getStepMutable().clear();
    tileLoop.getStepMutable().append(currentLoopSteps);
    rewriter.finalizeOpModification(tileLoop);
  }

  // Calculate new upper bounds for element loops
  for (size_t i = 0; i < newSteps.size(); i++) {
    rewriter.setInsertionPoint(innerLoop.getBody().getTerminator());
    // UpperBound: min(origUB, origIV+(originalStep*tile_size))
    auto stepped =
        mlir::arith::AddIOp::create(rewriter, loc, origIVs[i], newSteps[i]);
    mlir::Value newUB = stepped;
    if (inclusiveUBs[i]) {
      // Handle InclusiveUB
      // UpperBound: min(origUB, origIV+(originalStep*tile_size - 1))
      auto c1 = mlir::arith::ConstantOp::create(
```
- **EN**: Implements logic around `startOpModification`, `getStepMutable`, `finalizeOpModification`, `size`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `startOpModification`, `getStepMutable`, `finalizeOpModification`, `size`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 224-239
```cpp
          rewriter, loc, newSteps[i].getType(),
          rewriter.getIntegerAttr(newSteps[i].getType(), 1));
      newUB = mlir::arith::SubIOp::create(rewriter, loc, stepped, c1);
    }
    newUBs.push_back(
        mlir::arith::MinSIOp::create(rewriter, loc, origUBs[i], newUB));
  }

  // Create and insert nested elementLoopOps before terminator of outer loopOp
  mlir::acc::LoopOp currentLoop = innerLoop;
  for (size_t i = 0; i < tileSizes.size(); i++) {
    rewriter.setInsertionPoint(currentLoop.getBody().getTerminator());
    mlir::DenseBoolArrayAttr inclusiveUBAttr = mlir::DenseBoolArrayAttr{};
    if (inclusiveUBs[i])
      inclusiveUBAttr = rewriter.getDenseBoolArrayAttr({true});

```
- **EN**: Implements logic around `getType`, `getIntegerAttr`, `create`, `push_back`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getType`, `getIntegerAttr`, `create`, `push_back`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 240-255
```cpp
    mlir::acc::LoopOp elementLoop =
        createInnerLoop(innerLoop, rewriter, mlir::ValueRange{origIVs[i]},
                        mlir::ValueRange{newUBs[i]},
                        mlir::ValueRange{origSteps[i]}, inclusiveUBAttr, loc);

    // Remove vector/worker attributes from inner element loops except
    // outermost element loop
    if (i > 0) {
      rewriter.startOpModification(elementLoop);
      removeWorkerVectorFromLoop(elementLoop);
      rewriter.finalizeOpModification(elementLoop);
    }
    newIVs.push_back(elementLoop.getBody().getArgument(0));
    currentLoop = elementLoop;
  }

```
- **EN**: Implements logic around `createInnerLoop`, `startOpModification`, `removeWorkerVectorFromLoop`, `finalizeOpModification`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createInnerLoop`, `startOpModification`, `removeWorkerVectorFromLoop`, `finalizeOpModification`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 256-265
```cpp
  // Remove vector/worker attributes from outer tile loops
  for (auto tileLoop : tileLoops) {
    rewriter.startOpModification(tileLoop);
    removeWorkerVectorFromLoop(tileLoop);
    rewriter.finalizeOpModification(tileLoop);
  }

  // Move ops from inner tile loop to inner element loop and replace IV uses
  moveOpsAndReplaceIVs(innerLoop, currentLoop, newIVs, origIVs, nOps, rewriter);

```
- **EN**: Implements logic around `startOpModification`, `removeWorkerVectorFromLoop`, `finalizeOpModification`, `moveOpsAndReplaceIVs`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `startOpModification`, `removeWorkerVectorFromLoop`, `finalizeOpModification`, `moveOpsAndReplaceIVs` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 266-285
```cpp
  return outerLoop;
}

llvm::SmallVector<mlir::acc::LoopOp>
mlir::acc::uncollapseLoops(mlir::acc::LoopOp origLoop, unsigned tileCount,
                           unsigned collapseCount,
                           mlir::RewriterBase &rewriter) {
  llvm::SmallVector<mlir::acc::LoopOp> newLoops;
  llvm::SmallVector<mlir::Value, 3> newIVs;
  mlir::Location loc = origLoop.getLoc();
  llvm::SmallVector<bool> newInclusiveUBs;
  llvm::SmallVector<mlir::Value, 3> lbs, ubs, steps;
  for (unsigned i = 0; i < collapseCount; i++) {
    // inclusiveUpperbound attribute might not be set, default to false
    bool inclusiveUB = false;
    if (origLoop.getInclusiveUpperboundAttr())
      inclusiveUB = origLoop.getInclusiveUpperboundAttr().asArrayRef()[i];
    newInclusiveUBs.push_back(inclusiveUB);
    lbs.push_back(origLoop.getLowerbound()[i]);
    ubs.push_back(origLoop.getUpperbound()[i]);
```
- **EN**: Implements logic around `uncollapseLoops`, `getLoc`, `getInclusiveUpperboundAttr`, `push_back`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `uncollapseLoops`, `getLoc`, `getInclusiveUpperboundAttr`, `push_back` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 286-302
```cpp
    steps.push_back(origLoop.getStep()[i]);
  }
  mlir::acc::LoopOp outerLoop = createACCLoopFromOriginal(
      origLoop, rewriter, lbs, ubs, steps,
      rewriter.getDenseBoolArrayAttr(newInclusiveUBs),
      origLoop.getCombinedAttr(), loc, /*preserveCollapse*/ true);
  mlir::Block *blk = rewriter.createBlock(&outerLoop.getRegion(),
                                          outerLoop.getRegion().begin());
  rewriter.setInsertionPointToEnd(blk);
  mlir::acc::YieldOp::create(rewriter, loc);
  for (unsigned i = 0; i < collapseCount; i++) {
    outerLoop.getBody().addArgument(origLoop.getBody().getArgument(i).getType(),
                                    loc);
    newIVs.push_back(outerLoop.getBody().getArgument(i));
  }
  newLoops.push_back(outerLoop);

```
- **EN**: Implements logic around `push_back`, `createACCLoopFromOriginal`, `getDenseBoolArrayAttr`, `getCombinedAttr`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `push_back`, `createACCLoopFromOriginal`, `getDenseBoolArrayAttr`, `getCombinedAttr`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 303-322
```cpp
  mlir::acc::LoopOp currentLoopOp = outerLoop;
  for (unsigned i = collapseCount; i < tileCount; i++) {
    rewriter.setInsertionPoint(currentLoopOp.getBody().getTerminator());
    bool inclusiveUB = false;
    if (origLoop.getInclusiveUpperboundAttr())
      inclusiveUB = origLoop.getInclusiveUpperboundAttr().asArrayRef()[i];
    mlir::DenseBoolArrayAttr inclusiveUBAttr =
        rewriter.getDenseBoolArrayAttr({inclusiveUB});
    mlir::acc::LoopOp innerLoop = createInnerLoop(
        origLoop, rewriter, mlir::ValueRange{origLoop.getLowerbound()[i]},
        mlir::ValueRange{origLoop.getUpperbound()[i]},
        mlir::ValueRange{origLoop.getStep()[i]}, inclusiveUBAttr, loc);
    newIVs.push_back(innerLoop.getBody().getArgument(0));
    newLoops.push_back(innerLoop);
    currentLoopOp = innerLoop;
  }
  // Move ops from origLoop to innermost loop and replace uses of IVs
  size_t nOps = origLoop.getBody().getOperations().size();
  llvm::SmallVector<mlir::Value, 3> origIVs;
  for (auto arg : origLoop.getBody().getArguments())
```
- **EN**: Implements logic around `setInsertionPoint`, `getInclusiveUpperboundAttr`, `getDenseBoolArrayAttr`, `createInnerLoop`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPoint`, `getInclusiveUpperboundAttr`, `getDenseBoolArrayAttr`, `createInnerLoop`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 323-328
```cpp
    origIVs.push_back(arg);
  moveOpsAndReplaceIVs(origLoop, currentLoopOp, newIVs, origIVs, nOps,
                       rewriter);

  return newLoops;
}
```
- **EN**: Implements logic around `push_back`, `moveOpsAndReplaceIVs`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `push_back`, `moveOpsAndReplaceIVs` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

## Key Concepts / 关键概念

- **Dialect utilities / 方言工具**:
  - **EN**: Collects reusable helpers that keep core dialect logic factored and shareable.
  - **CN**: 汇集可复用辅助函数，使核心方言逻辑保持解耦并可共享。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/OpenACCUtilsTiling.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/Transforms/RegionUtils.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
