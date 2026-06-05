# ParallelLoopTiling.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Transforms/ParallelLoopTiling.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements loop tiling on parallel loops.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Transforms`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ParallelLoopTiling.cpp - Tiles scf.parallel ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp
//
// This file implements loop tiling on parallel loops.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SCF/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h`。

### Lines 15-21
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Transforms.h"
#include "mlir/Dialect/SCF/Utils/Utils.h"

namespace mlir {
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`。

### Lines 22-28
```cpp
#define GEN_PASS_DEF_SCFPARALLELLOOPTILING
#include "mlir/Dialect/SCF/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::scf;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h.inc`。

### Lines 29-35
```cpp
/// Tile a parallel loop of the form
///   scf.parallel (%i0, %i1) = (%arg0, %arg1) to (%arg2, %arg3)
///                                            step (%arg4, %arg5)
///
/// into
///   scf.parallel (%i0, %i1) = (%arg0, %arg1) to (%arg2, %arg3)
///                                            step (%arg4*tileSize[0],
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-42
```cpp
///                                                  %arg5*tileSize[1])
///     scf.parallel (%j0, %j1) = (0, 0) to (min(%arg4*tileSize[0], %arg2-%i0)
///                                          min(%arg5*tileSize[1], %arg3-%i1))
///                                      step (%arg4, %arg5)
///
/// or, when no-min-max-bounds is true, into
///   scf.parallel (%i0, %i1) = (%arg0, %arg1) to (%arg2, %arg3)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 43-49
```cpp
///                                            step (%arg4*tileSize[0],
///                                                  %arg5*tileSize[1])
///     scf.parallel (%j0, %j1) = (0, 0) to (%arg4*tileSize[0],
///                                          %arg5*tileSize[1])
///                                      step (%arg4, %arg5)
///        %inbound = (%j0 * %arg4 + %i0 < %arg2) &&
///                   (%j1 * %arg5 + %i1 < %arg3)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 50-56
```cpp
///        scf.if (%inbound)
///          ....
///
/// where the uses of %i0 and %i1 in the loop body are replaced by
/// %i0 + j0 and %i1 + %j1.
///
/// The old loop is replaced with the new one.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 57-70
```cpp
std::pair<ParallelOp, ParallelOp>
mlir::scf::tileParallelLoop(ParallelOp op, ArrayRef<int64_t> tileSizes,
                            bool noMinMaxBounds) {
  OpBuilder b(op);
  auto zero = arith::ConstantIndexOp::create(b, op.getLoc(), 0);
  SmallVector<Value, 2> tileSizeConstants;
  tileSizeConstants.reserve(op.getUpperBound().size());
  for (size_t i = 0, end = op.getUpperBound().size(); i != end; ++i) {
    if (i < tileSizes.size())
      tileSizeConstants.push_back(
          arith::ConstantIndexOp::create(b, op.getLoc(), tileSizes[i]));
    else
      // Just pick 1 for the remaining dimensions.
      tileSizeConstants.push_back(
```
- **EN**: Implements logic around `tileParallelLoop`, `b`, `create`, `reserve`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `tileParallelLoop`, `b`, `create`, `reserve`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 71-84
```cpp
          arith::ConstantIndexOp::create(b, op.getLoc(), 1));
  }

  // Create the outer loop with adjusted steps.
  SmallVector<Value, 2> newSteps;
  newSteps.reserve(op.getStep().size());
  for (auto step : llvm::zip(op.getStep(), tileSizeConstants)) {
    newSteps.push_back(arith::MulIOp::create(b, op.getLoc(), std::get<0>(step),
                                             std::get<1>(step)));
  }
  auto outerLoop = ParallelOp::create(b, op.getLoc(), op.getLowerBound(),
                                      op.getUpperBound(), newSteps);
  b.setInsertionPointToStart(outerLoop.getBody());

```
- **EN**: Implements logic around `create`, `reserve`, `zip`, `push_back`, and 3 more symbols.
- **CN**: 围绕 `create`, `reserve`, `zip`, `push_back`, and 3 more symbols 实现具体逻辑。

### Lines 85-92
```cpp
  // Compute min(size, dim - offset) to avoid out-of-bounds accesses.
  auto minMap = AffineMap::get(
      /*dimCount=*/3, /*symbolCount=*/0,
      {getAffineDimExpr(/*position=*/0, b.getContext()),
       getAffineDimExpr(/*position=*/1, b.getContext()) -
           getAffineDimExpr(/*position=*/2, b.getContext())},
      b.getContext());

```
- **EN**: Implements logic around `get`, `getAffineDimExpr`, `getContext`.
- **CN**: 围绕 `get`, `getAffineDimExpr`, `getContext` 实现具体逻辑。

### Lines 93-106
```cpp
  // Create the inner loop with adjusted bounds.
  SmallVector<Value, 2> newBounds;
  newBounds.reserve(op.getUpperBound().size());
  bool needInboundCheck = false;
  for (auto [lowerBound, upperBound, newStep, iv, step, tileSizeConstant] :
       llvm::zip(outerLoop.getLowerBound(), outerLoop.getUpperBound(),
                 outerLoop.getStep(), outerLoop.getInductionVars(),
                 op.getStep(), tileSizeConstants)) {
    // Collect the statically known loop bounds
    auto lowerBoundConstant =
        lowerBound.getDefiningOp<arith::ConstantIndexOp>();
    auto upperBoundConstant =
        upperBound.getDefiningOp<arith::ConstantIndexOp>();
    auto stepConstant = step.getDefiningOp<arith::ConstantIndexOp>();
```
- **EN**: Implements logic around `reserve`, `zip`, `getStep`, `ConstantIndexOp>`.
- **CN**: 围绕 `reserve`, `zip`, `getStep`, `ConstantIndexOp>` 实现具体逻辑。

### Lines 107-120
```cpp
    auto tileSize =
        cast<arith::ConstantIndexOp>(tileSizeConstant.getDefiningOp()).value();
    // If the loop bounds and the loop step are constant and if the number of
    // loop iterations is an integer multiple of the tile size, we use a static
    // bound for the inner loop.
    if (lowerBoundConstant && upperBoundConstant && stepConstant) {
      auto numIterations = llvm::divideCeil(upperBoundConstant.value() -
                                                lowerBoundConstant.value(),
                                            stepConstant.value());
      if (numIterations % tileSize == 0) {
        newBounds.push_back(newStep);
        continue;
      }
    }
```
- **EN**: Implements logic around `ConstantIndexOp>`, `divideCeil`, `value`, `push_back`.
- **CN**: 围绕 `ConstantIndexOp>`, `divideCeil`, `value`, `push_back` 实现具体逻辑。

### Lines 121-128
```cpp

    // For InboundCheck mode, just use the variable outer step
    if (noMinMaxBounds) {
      newBounds.push_back(newStep);
      needInboundCheck = true;
      continue;
    }

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 129-138
```cpp
    // Otherwise, we dynamically compute the bound for
    // each iteration of the outer loop.
    newBounds.push_back(
        affine::AffineMinOp::create(b, op.getLoc(), b.getIndexType(), minMap,
                                    ValueRange{newStep, upperBound, iv}));
  }
  auto innerLoop = ParallelOp::create(
      b, op.getLoc(), SmallVector<Value, 2>(newBounds.size(), zero), newBounds,
      op.getStep());

```
- **EN**: Implements logic around `push_back`, `create`, `getLoc`, `getStep`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `create`, `getLoc`, `getStep` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 139-152
```cpp
  if (noMinMaxBounds && needInboundCheck) {
    b.setInsertionPointToStart(innerLoop.getBody());
    // Insert in-bound check
    Value inbound =
        arith::ConstantIntOp::create(b, op.getLoc(), b.getIntegerType(1), 1);
    for (auto [outerUpperBound, outerIV, innerIV, innerStep] :
         llvm::zip(outerLoop.getUpperBound(), outerLoop.getInductionVars(),
                   innerLoop.getInductionVars(), innerLoop.getStep())) {
      // %in_bound = %in_bound &&
      //             (%inner_iv * %inner_step + %outer_iv < %outer_upper_bound)
      Value index = arith::AddIOp::create(
          b, op.getLoc(),
          arith::MulIOp::create(b, op.getLoc(), innerIV, innerStep), outerIV);
      Value dimInbound = arith::CmpIOp::create(
```
- **EN**: Implements logic around `setInsertionPointToStart`, `create`, `zip`, `getInductionVars`, and 1 more symbols.
- **CN**: 围绕 `setInsertionPointToStart`, `create`, `zip`, `getInductionVars`, and 1 more symbols 实现具体逻辑。

### Lines 153-166
```cpp
          b, op.getLoc(), arith::CmpIPredicate::ult, index, outerUpperBound);
      inbound = arith::AndIOp::create(b, op.getLoc(), inbound, dimInbound);
    }
    auto ifInbound = IfOp::create(b, op.getLoc(),
                                  /*resultTypes*/ ArrayRef<Type>{}, inbound,
                                  /*hasElseRegion*/ false);
    ifInbound.getThenRegion().takeBody(op.getRegion());
    Block &thenBlock = ifInbound.getThenRegion().front();
    // Replace the scf.reduce terminator with an scf.yield terminator.
    Operation *reduceOp = thenBlock.getTerminator();
    b.setInsertionPointToEnd(&thenBlock);
    scf::YieldOp::create(b, reduceOp->getLoc());
    reduceOp->erase();
    b.setInsertionPointToStart(innerLoop.getBody());
```
- **EN**: Implements logic around `getLoc`, `create`, `getThenRegion`, `getTerminator`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `create`, `getThenRegion`, `getTerminator`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 167-180
```cpp
    for (const auto &ivs : llvm::enumerate(llvm::zip(
             innerLoop.getInductionVars(), outerLoop.getInductionVars()))) {
      auto newIndex = arith::AddIOp::create(
          b, op.getLoc(), std::get<0>(ivs.value()), std::get<1>(ivs.value()));
      thenBlock.getArgument(ivs.index())
          .replaceAllUsesExcept(newIndex, newIndex);
    }
    thenBlock.eraseArguments(0, thenBlock.getNumArguments());
  } else {
    innerLoop.getRegion().takeBody(op.getRegion());
    b.setInsertionPointToStart(innerLoop.getBody());
    for (auto ivs : llvm::zip(innerLoop.getInductionVars(),
                              outerLoop.getInductionVars())) {
      Value innerIndex = std::get<0>(ivs);
```
- **EN**: Implements logic around `enumerate`, `getInductionVars`, `create`, `getLoc`, and 7 more symbols.
- **CN**: 围绕 `enumerate`, `getInductionVars`, `create`, `getLoc`, and 7 more symbols 实现具体逻辑。

### Lines 181-190
```cpp
      auto newIndex = arith::AddIOp::create(b, op.getLoc(), std::get<0>(ivs),
                                            std::get<1>(ivs));
      innerIndex.replaceAllUsesExcept(newIndex, newIndex);
    }
  }

  op.erase();
  return std::make_pair(outerLoop, innerLoop);
}

```
- **EN**: Implements logic around `create`, `get`, `replaceAllUsesExcept`, `erase`, and 1 more symbols.
- **CN**: 围绕 `create`, `get`, `replaceAllUsesExcept`, `erase`, and 1 more symbols 实现具体逻辑。

### Lines 191-200
```cpp
namespace {
struct ParallelLoopTiling
    : public impl::SCFParallelLoopTilingBase<ParallelLoopTiling> {
  ParallelLoopTiling() = default;
  explicit ParallelLoopTiling(ArrayRef<int64_t> tileSizes,
                              bool noMinMaxBounds = false) {
    this->tileSizes = tileSizes;
    this->noMinMaxBounds = noMinMaxBounds;
  }

```
- **EN**: Introduces declarations for `ParallelLoopTiling`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ParallelLoopTiling` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 201-214
```cpp
  void runOnOperation() override {
    for (auto tileSize : tileSizes)
      if (tileSize == 0) {
        mlir::emitError(mlir::UnknownLoc::get(&Pass::getContext()),
                        "tile size cannot be 0");
        return signalPassFailure();
      }
    auto *parentOp = getOperation();
    SmallVector<ParallelOp, 2> innermostPloops;
    getInnermostParallelLoops(parentOp, innermostPloops);
    for (ParallelOp ploop : innermostPloops) {
      // FIXME: Add reduction support.
      if (ploop.getNumReductions() == 0)
        tileParallelLoop(ploop, tileSizes, noMinMaxBounds);
```
- **EN**: Implements logic around `runOnOperation`, `emitError`, `signalPassFailure`, `getOperation`, and 3 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `emitError`, `signalPassFailure`, `getOperation`, and 3 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 215-224
```cpp
    }
  }
};
} // namespace

std::unique_ptr<Pass>
mlir::createParallelLoopTilingPass(ArrayRef<int64_t> tileSizes,
                                   bool noMinMaxBounds) {
  return std::make_unique<ParallelLoopTiling>(tileSizes, noMinMaxBounds);
}
```
- **EN**: Implements logic around `createParallelLoopTilingPass`, `make_unique`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `createParallelLoopTilingPass`, `make_unique` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`, `mlir/Dialect/SCF/Utils/Utils.h`, `mlir/Dialect/SCF/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (7)
