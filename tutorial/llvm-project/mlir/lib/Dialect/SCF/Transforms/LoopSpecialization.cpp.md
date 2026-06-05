# LoopSpecialization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Transforms/LoopSpecialization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Specializes parallel loops and for loops for easier unrolling and vectorization.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Transforms`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- LoopSpecialization.cpp - scf.parallel/SCR.for specialization -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Specializes parallel loops and for loops for easier unrolling and
// vectorization.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 13-27
```cpp

#include "mlir/Dialect/SCF/Transforms/Passes.h"

#include "mlir/Dialect/Affine/Analysis/AffineStructures.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Transforms.h"
#include "mlir/Dialect/SCF/Utils/AffineCanonicalizationUtils.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`。

### Lines 28-39
```cpp
namespace mlir {
#define GEN_PASS_DEF_SCFFORLOOPPEELING
#define GEN_PASS_DEF_SCFFORLOOPSPECIALIZATION
#define GEN_PASS_DEF_SCFPARALLELLOOPSPECIALIZATION
#include "mlir/Dialect/SCF/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::affine;
using scf::ForOp;
using scf::ParallelOp;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h.inc`。

### Lines 40-59
```cpp
/// Rewrite a parallel loop with bounds defined by an affine.min with a constant
/// into 2 loops after checking if the bounds are equal to that constant. This
/// is beneficial if the loop will almost always have the constant bound and
/// that version can be fully unrolled and vectorized.
static void specializeParallelLoopForUnrolling(ParallelOp op) {
  SmallVector<int64_t, 2> constantIndices;
  constantIndices.reserve(op.getUpperBound().size());
  for (auto bound : op.getUpperBound()) {
    auto minOp = bound.getDefiningOp<AffineMinOp>();
    if (!minOp)
      return;
    int64_t minConstant = std::numeric_limits<int64_t>::max();
    for (AffineExpr expr : minOp.getMap().getResults()) {
      if (auto constantIndex = dyn_cast<AffineConstantExpr>(expr))
        minConstant = std::min(minConstant, constantIndex.getValue());
    }
    if (minConstant == std::numeric_limits<int64_t>::max())
      return;
    constantIndices.push_back(minConstant);
  }
```
- **EN**: Implements logic around `specializeParallelLoopForUnrolling`, `reserve`, `getUpperBound`, `getDefiningOp`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `specializeParallelLoopForUnrolling`, `reserve`, `getUpperBound`, `getDefiningOp`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 60-77
```cpp

  OpBuilder b(op);
  IRMapping map;
  Value cond;
  for (auto bound : llvm::zip(op.getUpperBound(), constantIndices)) {
    Value constant =
        arith::ConstantIndexOp::create(b, op.getLoc(), std::get<1>(bound));
    Value cmp = arith::CmpIOp::create(b, op.getLoc(), arith::CmpIPredicate::eq,
                                      std::get<0>(bound), constant);
    cond = cond ? arith::AndIOp::create(b, op.getLoc(), cond, cmp) : cmp;
    map.map(std::get<0>(bound), constant);
  }
  auto ifOp = scf::IfOp::create(b, op.getLoc(), cond, /*withElseRegion=*/true);
  ifOp.getThenBodyBuilder().clone(*op.getOperation(), map);
  ifOp.getElseBodyBuilder().clone(*op.getOperation());
  op.erase();
}

```
- **EN**: Implements logic around `b`, `zip`, `create`, `get`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `b`, `zip`, `create`, `get`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 78-94
```cpp
/// Rewrite a for loop with bounds defined by an affine.min with a constant into
/// 2 loops after checking if the bounds are equal to that constant. This is
/// beneficial if the loop will almost always have the constant bound and that
/// version can be fully unrolled and vectorized.
static void specializeForLoopForUnrolling(ForOp op) {
  auto bound = op.getUpperBound();
  auto minOp = bound.getDefiningOp<AffineMinOp>();
  if (!minOp)
    return;
  int64_t minConstant = std::numeric_limits<int64_t>::max();
  for (AffineExpr expr : minOp.getMap().getResults()) {
    if (auto constantIndex = dyn_cast<AffineConstantExpr>(expr))
      minConstant = std::min(minConstant, constantIndex.getValue());
  }
  if (minConstant == std::numeric_limits<int64_t>::max())
    return;

```
- **EN**: Implements logic around `specializeForLoopForUnrolling`, `getUpperBound`, `getDefiningOp`, `max`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `specializeForLoopForUnrolling`, `getUpperBound`, `getDefiningOp`, `max`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 95-108
```cpp
  OpBuilder b(op);
  IRMapping map;
  Value constant = arith::ConstantOp::create(
      b, op.getLoc(),
      IntegerAttr::get(op.getUpperBound().getType(), minConstant));
  Value cond = arith::CmpIOp::create(b, op.getLoc(), arith::CmpIPredicate::eq,
                                     bound, constant);
  map.map(bound, constant);
  auto ifOp = scf::IfOp::create(b, op.getLoc(), cond, /*withElseRegion=*/true);
  ifOp.getThenBodyBuilder().clone(*op.getOperation(), map);
  ifOp.getElseBodyBuilder().clone(*op.getOperation());
  op.erase();
}

```
- **EN**: Implements logic around `b`, `create`, `getLoc`, `get`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `b`, `create`, `getLoc`, `get`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 109-118
```cpp
/// Rewrite a for loop with bounds/step that potentially do not divide evenly
/// into a for loop where the step divides the iteration space evenly, followed
/// by an scf.if for the last (partial) iteration (if any).
///
/// This function rewrites the given scf.for loop in-place and creates a new
/// scf.if operation for the last iteration. It replaces all uses of the
/// unpeeled loop with the results of the newly generated scf.if.
///
/// The newly generated scf.if operation is returned via `ifOp`. The boundary
/// at which the loop is split (new upper bound) is returned via `splitBound`.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 119-129
```cpp
/// The return value indicates whether the loop was rewritten or not.
///
/// Note: Loops with a step size of 0 cannot be peeled. Applying this function
/// to such a loop may result in IR with undefined behavior.
static LogicalResult peelForLoop(RewriterBase &b, ForOp forOp,
                                 ForOp &partialIteration, Value &splitBound) {
  RewriterBase::InsertionGuard guard(b);
  auto lbInt = getConstantIntValue(forOp.getLowerBound());
  auto ubInt = getConstantIntValue(forOp.getUpperBound());
  auto stepInt = getConstantIntValue(forOp.getStep());

```
- **EN**: Implements logic around `peelForLoop`, `guard`, `getConstantIntValue`.
- **CN**: 围绕 `peelForLoop`, `guard`, `getConstantIntValue` 实现具体逻辑。

### Lines 130-149
```cpp
  // No specialization necessary if step size is 1. Also bail out in case of an
  // invalid zero or negative step which might have happened during folding.
  if (stepInt && *stepInt <= 1)
    return failure();

  // No specialization necessary if step already divides upper bound evenly.
  // Fast path: lb, ub and step are constants.
  if (lbInt && ubInt && stepInt && (*ubInt - *lbInt) % *stepInt == 0)
    return failure();
  // Slow path: Examine the ops that define lb, ub and step.
  AffineExpr sym0, sym1, sym2;
  bindSymbols(b.getContext(), sym0, sym1, sym2);
  SmallVector<Value> operands{forOp.getLowerBound(), forOp.getUpperBound(),
                              forOp.getStep()};
  AffineMap map = AffineMap::get(0, 3, {(sym1 - sym0) % sym2});
  affine::fullyComposeAffineMapAndOperands(&map, &operands);
  if (auto constExpr = dyn_cast<AffineConstantExpr>(map.getResult(0)))
    if (constExpr.getValue() == 0)
      return failure();

```
- **EN**: Implements logic around `failure`, `bindSymbols`, `getLowerBound`, `getStep`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `bindSymbols`, `getLowerBound`, `getStep`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 150-161
```cpp
  // New upper bound: %ub - (%ub - %lb) mod %step
  auto modMap = AffineMap::get(0, 3, {sym1 - ((sym1 - sym0) % sym2)});
  b.setInsertionPoint(forOp);
  auto loc = forOp.getLoc();
  splitBound = b.createOrFold<AffineApplyOp>(loc, modMap,
                                             ValueRange{forOp.getLowerBound(),
                                                        forOp.getUpperBound(),
                                                        forOp.getStep()});
  if (splitBound.getType() != forOp.getLowerBound().getType())
    splitBound = b.createOrFold<arith::IndexCastOp>(
        loc, forOp.getLowerBound().getType(), splitBound);

```
- **EN**: Implements logic around `get`, `setInsertionPoint`, `getLoc`, `createOrFold`, and 5 more symbols.
- **CN**: 围绕 `get`, `setInsertionPoint`, `getLoc`, `createOrFold`, and 5 more symbols 实现具体逻辑。

### Lines 162-172
```cpp
  // Create ForOp for partial iteration.
  b.setInsertionPointAfter(forOp);
  partialIteration = cast<ForOp>(b.clone(*forOp.getOperation()));
  partialIteration.getLowerBoundMutable().assign(splitBound);
  b.replaceAllUsesWith(forOp.getResults(), partialIteration->getResults());
  partialIteration.getInitArgsMutable().assign(forOp->getResults());

  // Set new upper loop bound.
  b.modifyOpInPlace(forOp,
                    [&]() { forOp.getUpperBoundMutable().assign(splitBound); });

```
- **EN**: Implements logic around `setInsertionPointAfter`, `clone`, `getLowerBoundMutable`, `replaceAllUsesWith`, and 3 more symbols.
- **CN**: 围绕 `setInsertionPointAfter`, `clone`, `getLowerBoundMutable`, `replaceAllUsesWith`, and 3 more symbols 实现具体逻辑。

### Lines 173-184
```cpp
  return success();
}

static void rewriteAffineOpAfterPeeling(RewriterBase &rewriter, ForOp forOp,
                                        ForOp partialIteration,
                                        Value previousUb) {
  Value mainIv = forOp.getInductionVar();
  Value partialIv = partialIteration.getInductionVar();
  assert(forOp.getStep() == partialIteration.getStep() &&
         "expected same step in main and partial loop");
  Value step = forOp.getStep();

```
- **EN**: Implements logic around `success`, `rewriteAffineOpAfterPeeling`, `getInductionVar`, `assert`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `success`, `rewriteAffineOpAfterPeeling`, `getInductionVar`, `assert`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 185-201
```cpp
  forOp.walk([&](Operation *affineOp) {
    if (!isa<AffineMinOp, AffineMaxOp>(affineOp))
      return WalkResult::advance();
    (void)scf::rewritePeeledMinMaxOp(rewriter, affineOp, mainIv, previousUb,
                                     step,
                                     /*insideLoop=*/true);
    return WalkResult::advance();
  });
  partialIteration.walk([&](Operation *affineOp) {
    if (!isa<AffineMinOp, AffineMaxOp>(affineOp))
      return WalkResult::advance();
    (void)scf::rewritePeeledMinMaxOp(rewriter, affineOp, partialIv, previousUb,
                                     step, /*insideLoop=*/false);
    return WalkResult::advance();
  });
}

```
- **EN**: Implements logic around `walk`, `AffineMaxOp>`, `advance`, `rewritePeeledMinMaxOp`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `walk`, `AffineMaxOp>`, `advance`, `rewritePeeledMinMaxOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 202-212
```cpp
LogicalResult mlir::scf::peelForLoopAndSimplifyBounds(RewriterBase &rewriter,
                                                      ForOp forOp,
                                                      ForOp &partialIteration) {
  Value previousUb = forOp.getUpperBound();
  Value splitBound;
  if (failed(peelForLoop(rewriter, forOp, partialIteration, splitBound)))
    return failure();

  // Rewrite affine.min and affine.max ops.
  rewriteAffineOpAfterPeeling(rewriter, forOp, partialIteration, previousUb);

```
- **EN**: Implements logic around `peelForLoopAndSimplifyBounds`, `getUpperBound`, `failed`, `failure`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `peelForLoopAndSimplifyBounds`, `getUpperBound`, `failed`, `failure`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 213-227
```cpp
  return success();
}

/// Rewrites the original scf::ForOp as two scf::ForOp Ops, the first
/// scf::ForOp corresponds to the first iteration of the loop which can be
/// canonicalized away in the following optimizations. The second loop Op
/// contains the remaining iterations, with a lower bound updated as the
/// original lower bound plus the step (i.e. skips the first iteration).
LogicalResult mlir::scf::peelForLoopFirstIteration(RewriterBase &b, ForOp forOp,
                                                   ForOp &firstIteration) {
  RewriterBase::InsertionGuard guard(b);
  auto lbInt = getConstantIntValue(forOp.getLowerBound());
  auto ubInt = getConstantIntValue(forOp.getUpperBound());
  auto stepInt = getConstantIntValue(forOp.getStep());

```
- **EN**: Implements logic around `success`, `peelForLoopFirstIteration`, `guard`, `getConstantIntValue`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `success`, `peelForLoopFirstIteration`, `guard`, `getConstantIntValue` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 228-244
```cpp
  // Peeling is not needed if there is one or less iteration.
  if (lbInt && ubInt && stepInt && ceil(float(*ubInt - *lbInt) / *stepInt) <= 1)
    return failure();

  AffineExpr lbSymbol, stepSymbol;
  bindSymbols(b.getContext(), lbSymbol, stepSymbol);

  // New lower bound for main loop: %lb + %step
  auto ubMap = AffineMap::get(0, 2, {lbSymbol + stepSymbol});
  b.setInsertionPoint(forOp);
  auto loc = forOp.getLoc();
  Value splitBound = b.createOrFold<AffineApplyOp>(
      loc, ubMap, ValueRange{forOp.getLowerBound(), forOp.getStep()});
  if (splitBound.getType() != forOp.getUpperBound().getType())
    splitBound = b.createOrFold<arith::IndexCastOp>(
        loc, forOp.getUpperBound().getType(), splitBound);

```
- **EN**: Implements logic around `ceil`, `failure`, `bindSymbols`, `get`, and 7 more symbols.
- **CN**: 围绕 `ceil`, `failure`, `bindSymbols`, `get`, and 7 more symbols 实现具体逻辑。

### Lines 245-255
```cpp
  // Peel the first iteration.
  firstIteration = cast<ForOp>(b.clone(*forOp.getOperation()));
  b.modifyOpInPlace(firstIteration, [&]() {
    firstIteration.getUpperBoundMutable().assign(splitBound);
  });
  // Update main loop with new lower bound.
  b.modifyOpInPlace(forOp, [&]() {
    forOp.getInitArgsMutable().assign(firstIteration->getResults());
    forOp.getLowerBoundMutable().assign(splitBound);
  });

```
- **EN**: Implements logic around `clone`, `modifyOpInPlace`, `getUpperBoundMutable`, `getInitArgsMutable`, and 1 more symbols.
- **CN**: 围绕 `clone`, `modifyOpInPlace`, `getUpperBoundMutable`, `getInitArgsMutable`, and 1 more symbols 实现具体逻辑。

### Lines 256-267
```cpp
  return success();
}

static constexpr char kPeeledLoopLabel[] = "__peeled_loop__";
static constexpr char kPartialIterationLabel[] = "__partial_iteration__";

namespace {
struct ForLoopPeelingPattern : public OpRewritePattern<ForOp> {
  ForLoopPeelingPattern(MLIRContext *ctx, bool peelFront, bool skipPartial)
      : OpRewritePattern<ForOp>(ctx), peelFront(peelFront),
        skipPartial(skipPartial) {}

```
- **EN**: Introduces declarations for `ForLoopPeelingPattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ForLoopPeelingPattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 268-277
```cpp
  LogicalResult matchAndRewrite(ForOp forOp,
                                PatternRewriter &rewriter) const override {
    if (forOp.getUnsignedCmp())
      return rewriter.notifyMatchFailure(forOp,
                                         "unsigned loops are not supported");

    // Do not peel already peeled loops.
    if (forOp->hasAttr(kPeeledLoopLabel))
      return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `getUnsignedCmp`, `notifyMatchFailure`, `hasAttr`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `getUnsignedCmp`, `notifyMatchFailure`, `hasAttr`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 278-297
```cpp
    scf::ForOp partialIteration;
    // The case for peeling the first iteration of the loop.
    if (peelFront) {
      if (failed(
              peelForLoopFirstIteration(rewriter, forOp, partialIteration))) {
        return failure();
      }
    } else {
      if (skipPartial) {
        // No peeling of loops inside the partial iteration of another peeled
        // loop.
        Operation *op = forOp.getOperation();
        while ((op = op->getParentOfType<scf::ForOp>())) {
          if (op->hasAttr(kPartialIterationLabel))
            return failure();
        }
      }
      // Apply loop peeling.
      if (failed(
              peelForLoopAndSimplifyBounds(rewriter, forOp, partialIteration)))
```
- **EN**: Implements logic around `failed`, `peelForLoopFirstIteration`, `failure`, `getOperation`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failed`, `peelForLoopFirstIteration`, `failure`, `getOperation`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 298-311
```cpp
        return failure();
    }

    // Apply label, so that the same loop is not rewritten a second time.
    rewriter.modifyOpInPlace(partialIteration, [&]() {
      partialIteration->setAttr(kPeeledLoopLabel, rewriter.getUnitAttr());
      partialIteration->setAttr(kPartialIterationLabel, rewriter.getUnitAttr());
    });
    rewriter.modifyOpInPlace(forOp, [&]() {
      forOp->setAttr(kPeeledLoopLabel, rewriter.getUnitAttr());
    });
    return success();
  }

```
- **EN**: Implements logic around `failure`, `modifyOpInPlace`, `setAttr`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `failure`, `modifyOpInPlace`, `setAttr`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 312-324
```cpp
  // If set to true, the first iteration of the loop will be peeled. Otherwise,
  // the unevenly divisible loop will be peeled at the end.
  bool peelFront;

  /// If set to true, loops inside partial iterations of another peeled loop
  /// are not peeled. This reduces the size of the generated code. Partial
  /// iterations are not usually performance critical.
  /// Note: Takes into account the entire chain of parent operations, not just
  /// the direct parent.
  bool skipPartial;
};
} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 325-334
```cpp
namespace {
struct ParallelLoopSpecialization
    : public impl::SCFParallelLoopSpecializationBase<
          ParallelLoopSpecialization> {
  void runOnOperation() override {
    getOperation()->walk(
        [](ParallelOp op) { specializeParallelLoopForUnrolling(op); });
  }
};

```
- **EN**: Introduces declarations for `ParallelLoopSpecialization`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ParallelLoopSpecialization` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 335-344
```cpp
struct ForLoopSpecialization
    : public impl::SCFForLoopSpecializationBase<ForLoopSpecialization> {
  void runOnOperation() override {
    getOperation()->walk([](ForOp op) { specializeForLoopForUnrolling(op); });
  }
};

struct ForLoopPeeling : public impl::SCFForLoopPeelingBase<ForLoopPeeling> {
  using impl::SCFForLoopPeelingBase<ForLoopPeeling>::SCFForLoopPeelingBase;

```
- **EN**: Introduces declarations for `ForLoopSpecialization`, `ForLoopPeeling`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ForLoopSpecialization`, `ForLoopPeeling` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 345-360
```cpp
  void runOnOperation() override {
    auto *parentOp = getOperation();
    MLIRContext *ctx = parentOp->getContext();
    RewritePatternSet patterns(ctx);
    patterns.add<ForLoopPeelingPattern>(ctx, peelFront, skipPartial);
    (void)applyPatternsGreedily(parentOp, std::move(patterns));

    // Drop the markers.
    parentOp->walk([](Operation *op) {
      op->removeAttr(kPeeledLoopLabel);
      op->removeAttr(kPartialIterationLabel);
    });
  }
};
} // namespace

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `getContext`, `patterns`, and 4 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `getContext`, `patterns`, and 4 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 361-367
```cpp
std::unique_ptr<Pass> mlir::createParallelLoopSpecializationPass() {
  return std::make_unique<ParallelLoopSpecialization>();
}

std::unique_ptr<Pass> mlir::createForLoopSpecializationPass() {
  return std::make_unique<ForLoopSpecialization>();
}
```
- **EN**: Implements logic around `createParallelLoopSpecializationPass`, `make_unique`, `createForLoopSpecializationPass`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `createParallelLoopSpecializationPass`, `make_unique`, `createForLoopSpecializationPass` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`, `mlir/Dialect/SCF/Utils/AffineCanonicalizationUtils.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/IRMapping.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (9), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
