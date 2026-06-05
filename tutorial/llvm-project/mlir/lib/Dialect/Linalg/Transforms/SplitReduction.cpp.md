# SplitReduction.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/SplitReduction.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements linalg transformation to break a reduction dimension between a parallel and a reduction dimension.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-------- SplitReduction.cpp - Split reduction dimesion ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements linalg transformation to break a reduction dimension
// between a parallel and a reduction dimension.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 13-26
```cpp

#include <optional>
#include <utility>

#include "mlir/Analysis/SliceAnalysis.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Bufferization/IR/Bufferization.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Tensor/Utils/Utils.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `optional`, `utility`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Arith/IR/Arith.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `optional`, `utility`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Arith/IR/Arith.h`。

### Lines 27-42
```cpp
using namespace mlir;
using namespace mlir::linalg;

FailureOr<SplitReductionResult> mlir::linalg::splitReduction(
    RewriterBase &b, LinalgOp op,
    const ControlSplitReductionFn &controlSplitReductionFn, bool useAlloc) {
  OpBuilder::InsertionGuard guard(b);
  b.setInsertionPoint(op);

  SplitReductionOptions control = controlSplitReductionFn(op);
  int64_t ratio = control.ratio;
  unsigned insertSplitIndex = control.index;
  unsigned insertSplitDimension = control.index;
  if (ratio <= 1)
    return b.notifyMatchFailure(op, "split ratio needs to be greater than 1");

```
- **EN**: Introduces declarations for `mlir`, `mlir::linalg`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::linalg` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 43-62
```cpp
  SmallVector<unsigned> dims;
  op.getReductionDims(dims);

  if (dims.size() != 1)
    return b.notifyMatchFailure(op, "needs a single reduction dimension");
  unsigned reductionDim = dims[0];
  if (control.innerParallel) {
    insertSplitDimension = reductionDim + 1;
  }
  SmallVector<int64_t, 4> loopRanges = op.getStaticLoopRanges();
  int64_t reductionDimSize = loopRanges[reductionDim];
  if (reductionDimSize == ShapedType::kDynamic || reductionDimSize % ratio != 0)
    return b.notifyMatchFailure(
        op, "Reduction dimension not divisible by split ratio");
  if (op.getNumDpsInits() != 1)
    return b.notifyMatchFailure(op, "More than one output in split reduction");
  if (insertSplitIndex > op.getShape(op.getDpsInitOperand(0)).size())
    return b.notifyMatchFailure(op, "Insert dimension position too large "
                                    "compared to intermediate tensor size");

```
- **EN**: Implements logic around `getReductionDims`, `size`, `notifyMatchFailure`, `getStaticLoopRanges`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getReductionDims`, `size`, `notifyMatchFailure`, `getStaticLoopRanges`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 63-72
```cpp
  SmallVector<Operation *, 4> combinerOps;
  if (!matchReduction(op.getRegionOutputArgs(), 0, combinerOps) ||
      combinerOps.size() != 1)
    return b.notifyMatchFailure(op, "Cannot match the reduction pattern");

  Operation *reductionOp = combinerOps[0];
  std::optional<TypedAttr> identity = arith::getNeutralElement(reductionOp);
  if (!identity.has_value())
    return b.notifyMatchFailure(op, "Unknown identity value for the reduction");

```
- **EN**: Implements logic around `matchReduction`, `size`, `notifyMatchFailure`, `getNeutralElement`, and 1 more symbols.
- **CN**: 围绕 `matchReduction`, `size`, `notifyMatchFailure`, `getNeutralElement`, and 1 more symbols 实现具体逻辑。

### Lines 73-92
```cpp
  Location loc = op->getLoc();
  SmallVector<Value> newInputs;
  SmallVector<AffineMap> newMaps;
  // Calculate the new shapes and indexing maps of the input operands.
  for (OpOperand *operand : op.getDpsInputOperands()) {
    AffineMap map = op.getMatchingIndexingMap(operand);
    SmallVector<int64_t> newShape;
    SmallVector<AffineExpr> exprs;
    SmallVector<ReassociationIndices> reassociation;
    unsigned index = 0;
    for (unsigned idx : llvm::seq<unsigned>(0, map.getNumResults())) {
      unsigned dim = map.getDimPosition(idx);
      if (reductionDim == dim) {
        if (control.innerParallel) {
          newShape.push_back(op.getShape(operand)[idx] / ratio); // reduce
          newShape.push_back(ratio); // parallel (insert)
          exprs.push_back(
              b.getAffineDimExpr(dim < insertSplitDimension ? dim : dim + 1));
          exprs.push_back(b.getAffineDimExpr(insertSplitDimension));
        } else {
```
- **EN**: Implements logic around `getLoc`, `getDpsInputOperands`, `getMatchingIndexingMap`, `seq`, and 3 more symbols.
- **CN**: 围绕 `getLoc`, `getDpsInputOperands`, `getMatchingIndexingMap`, `seq`, and 3 more symbols 实现具体逻辑。

### Lines 93-112
```cpp
          newShape.push_back(ratio); // parallel (insert)
          newShape.push_back(op.getShape(operand)[idx] / ratio); // reduce
          exprs.push_back(b.getAffineDimExpr(insertSplitDimension));
          exprs.push_back(
              b.getAffineDimExpr(dim < insertSplitDimension ? dim : dim + 1));
        }
        reassociation.push_back({index++, index++});
        continue;
      }
      newShape.push_back(op.getShape(operand)[idx]);
      exprs.push_back(
          b.getAffineDimExpr(dim < insertSplitDimension ? dim : dim + 1));
      reassociation.push_back({index++});
    }
    newMaps.push_back(
        AffineMap::get(map.getNumDims() + 1, 0, exprs, op.getContext()));
    // If the shape is unchanged the input doesn't change.
    if (newShape == op.getShape(operand)) {
      newInputs.push_back(operand->get());
      continue;
```
- **EN**: Implements logic around `push_back`, `getAffineDimExpr`, `get`, `getShape`.
- **CN**: 围绕 `push_back`, `getAffineDimExpr`, `get`, `getShape` 实现具体逻辑。

### Lines 113-122
```cpp
    }
    Type newType = RankedTensorType::get(
        newShape,
        cast<RankedTensorType>(operand->get().getType()).getElementType());

    Value newInput = tensor::ExpandShapeOp::create(
        b, loc, newType, operand->get(), reassociation);
    newInputs.push_back(newInput);
  }

```
- **EN**: Implements logic around `get`, `create`, `push_back`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `create`, `push_back` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 123-142
```cpp
  // Calculate the new output map and shape, we insert the new dimension based
  // on the index returned by `controlSplitReductionFn`.
  SmallVector<int64_t> newOutputShape;
  AffineMap oldOutputMap = op.getMatchingIndexingMap(op.getDpsInitOperand(0));
  ArrayRef<int64_t> oldShape = op.getShape(op.getDpsInitOperand(0));
  SmallVector<AffineExpr> outputExpr;
  for (unsigned idx : llvm::seq<unsigned>(0, oldShape.size() + 1)) {
    if (insertSplitIndex == idx) {
      newOutputShape.push_back(ratio);
      outputExpr.push_back(b.getAffineDimExpr(insertSplitDimension));
    }
    if (idx < oldShape.size()) {
      newOutputShape.push_back(oldShape[idx]);
      unsigned dim = oldOutputMap.getDimPosition(idx);
      outputExpr.push_back(
          b.getAffineDimExpr(dim < insertSplitDimension ? dim : dim + 1));
    }
  }
  Value emptyOrAllocTensor;
  if (useAlloc) {
```
- **EN**: Implements logic around `getMatchingIndexingMap`, `getShape`, `seq`, `push_back`, and 3 more symbols.
- **CN**: 围绕 `getMatchingIndexingMap`, `getShape`, `seq`, `push_back`, and 3 more symbols 实现具体逻辑。

### Lines 143-156
```cpp
    emptyOrAllocTensor = bufferization::AllocTensorOp::create(
        b, loc,
        RankedTensorType::get(newOutputShape,
                              op.getRegionOutputArgs()[0].getType()),
        ValueRange{});
  } else {
    emptyOrAllocTensor = tensor::EmptyOp::create(
        b, loc, newOutputShape, op.getRegionOutputArgs()[0].getType());
  }
  Value constantOp = arith::ConstantOp::create(b, loc, *identity);
  Value identityTensor =
      linalg::FillOp::create(b, op->getLoc(), constantOp, emptyOrAllocTensor)
          .getResult(0);

```
- **EN**: Implements logic around `create`, `get`, `getRegionOutputArgs`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `get`, `getRegionOutputArgs`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 157-176
```cpp
  newMaps.push_back(AffineMap::get(oldOutputMap.getNumDims() + 1, 0, outputExpr,
                                   op.getContext()));
  SmallVector<utils::IteratorType> newIteratorTypes;
  for (auto [index, iteratorType] :
       llvm::enumerate(op.getIteratorTypesArray())) {
    if (insertSplitDimension == index)
      newIteratorTypes.push_back(utils::IteratorType::parallel);
    newIteratorTypes.push_back(iteratorType);
  }
  if (insertSplitDimension == op.getIteratorTypesArray().size()) {
    newIteratorTypes.push_back(utils::IteratorType::parallel);
  }
  // Create the new op matching the original op with an extra parallel
  // dimension.
  GenericOp genericOp = GenericOp::create(
      b, loc, TypeRange({emptyOrAllocTensor.getType()}), newInputs,
      ValueRange({identityTensor}), newMaps, newIteratorTypes);
  b.inlineRegionBefore(op->getRegion(0), genericOp.getRegion(),
                       genericOp.getRegion().begin());

```
- **EN**: Implements logic around `push_back`, `getContext`, `enumerate`, `getIteratorTypesArray`, and 5 more symbols.
- **CN**: 围绕 `push_back`, `getContext`, `enumerate`, `getIteratorTypesArray`, and 5 more symbols 实现具体逻辑。

### Lines 177-193
```cpp
  // Then create a new reduction that only reduce the newly added dimension
  // from the previous op.
  unsigned intermRank = newOutputShape.size();
  AffineMap inputMap = b.getMultiDimIdentityMap(intermRank);
  SmallVector<utils::IteratorType> reductionIteratorTypes;
  SmallVector<AffineExpr> exprs;
  for (unsigned i : llvm::seq<unsigned>(0, intermRank)) {
    if (insertSplitIndex == i) {
      reductionIteratorTypes.push_back(utils::IteratorType::reduction);
    } else {
      exprs.push_back(b.getAffineDimExpr(i));
      reductionIteratorTypes.push_back(utils::IteratorType::parallel);
    }
  }
  AffineMap outputMap = AffineMap::get(intermRank, 0, exprs, op.getContext());
  SmallVector<AffineMap> reductionMaps = {inputMap, outputMap};

```
- **EN**: Implements logic around `size`, `getMultiDimIdentityMap`, `seq`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `size`, `getMultiDimIdentityMap`, `seq`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 194-204
```cpp
  auto reduction = GenericOp::create(
      b, loc, op->getResultTypes(), ValueRange({genericOp.getResult(0)}),
      op.getDpsInits(), reductionMaps, reductionIteratorTypes,
      [reductionOp](OpBuilder &b, Location loc, ValueRange inputs) {
        Operation *clonedReductionOp = b.clone(*reductionOp);
        clonedReductionOp->setOperand(0, inputs[0]);
        clonedReductionOp->setOperand(1, inputs[1]);
        linalg::YieldOp::create(b, loc, clonedReductionOp->getResult(0));
      });
  b.replaceOp(op, reduction.getResults());

```
- **EN**: Implements logic around `create`, `getResultTypes`, `getDpsInits`, `clone`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResultTypes`, `getDpsInits`, `clone`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 205-214
```cpp
  return SplitReductionResult{emptyOrAllocTensor.getDefiningOp(),
                              identityTensor.getDefiningOp<FillOp>(),
                              cast<LinalgOp>(genericOp.getOperation()),
                              reduction};
}

/// Rewrite f(i, j, k, ...) into f(i, j, k * ratio + kk, ...)
/// TODO: Additional pattern to rewrite f(i, j, k * ratio + kk, ...) into
/// f(i, j, k, kk, ...) with a proper ExpandShapeOp. This is probably better
/// done as a transform to enable better vectorization.
```
- **EN**: Implements logic around `getDefiningOp`, `getOperation`.
- **CN**: 围绕 `getDefiningOp`, `getOperation` 实现具体逻辑。

### Lines 215-229
```cpp
static AffineMap scaleReductionDim(LinalgOp op, OpOperand &opOperand,
                                   unsigned reductionDimPos,
                                   int64_t reductionRatio) {
  auto reductionDim = getAffineDimExpr(reductionDimPos, op.getContext());
  auto reductionDimP1 = getAffineDimExpr(reductionDimPos + 1, op.getContext());
  AffineMap map = op.getMatchingIndexingMap(&opOperand);
  AffineMap idMap =
      AffineMap::getMultiDimIdentityMap(map.getNumDims(), op.getContext());
  AffineMap shiftedIdMap = idMap.shiftDims(1, /*offset=*/reductionDimPos + 1);
  AffineMap composeMap = shiftedIdMap.replace(
      reductionDim, reductionDim * reductionRatio + reductionDimP1,
      shiftedIdMap.getNumDims(), /*numSymbols=*/0);
  return map.compose(composeMap);
}

```
- **EN**: Implements logic around `scaleReductionDim`, `getAffineDimExpr`, `getMatchingIndexingMap`, `getMultiDimIdentityMap`, and 4 more symbols.
- **CN**: 围绕 `scaleReductionDim`, `getAffineDimExpr`, `getMatchingIndexingMap`, `getMultiDimIdentityMap`, and 4 more symbols 实现具体逻辑。

### Lines 230-239
```cpp
static AffineMap insertParallelDim(LinalgOp op, OpOperand &opOperand,
                                   unsigned reductionDimPos, int64_t size) {
  auto reductionDim = getAffineDimExpr(reductionDimPos, op.getContext());
  AffineMap map = op.getMatchingIndexingMap(&opOperand);
  AffineMap idMap =
      AffineMap::getMultiDimIdentityMap(map.getNumDims(), op.getContext());
  AffineMap shiftedIdMap = idMap.shiftDims(1, /*offset=*/reductionDimPos + 1);
  return map.compose(shiftedIdMap).insertResult(reductionDim, reductionDimPos);
}

```
- **EN**: Implements logic around `insertParallelDim`, `getAffineDimExpr`, `getMatchingIndexingMap`, `getMultiDimIdentityMap`, and 2 more symbols.
- **CN**: 围绕 `insertParallelDim`, `getAffineDimExpr`, `getMatchingIndexingMap`, `getMultiDimIdentityMap`, and 2 more symbols 实现具体逻辑。

### Lines 240-251
```cpp
/// Core rewrite implementation.
FailureOr<SplitReductionResult> mlir::linalg::splitReductionByScaling(
    RewriterBase &b, LinalgOp op,
    const ControlSplitReductionFn &controlSplitReductionFn, bool useAlloc) {
  OpBuilder::InsertionGuard guard(b);
  b.setInsertionPoint(op);

  // Matcher part, enforce preconditions.
  SplitReductionOptions control = controlSplitReductionFn(op);
  if (control.innerParallel)
    return b.notifyMatchFailure(op, "innerParallel not supported");

```
- **EN**: Implements logic around `splitReductionByScaling`, `guard`, `setInsertionPoint`, `controlSplitReductionFn`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `splitReductionByScaling`, `guard`, `setInsertionPoint`, `controlSplitReductionFn`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 252-261
```cpp
  int64_t splitFactor = control.ratio;
  unsigned insertSplitDimension = control.index;
  if (splitFactor <= 1)
    return b.notifyMatchFailure(op, "split factor needs to be greater than 1");

  SmallVector<unsigned> dims;
  op.getReductionDims(dims);
  if (dims.empty())
    return b.notifyMatchFailure(op, "needs at least 1 reduction dimension");

```
- **EN**: Implements logic around `notifyMatchFailure`, `getReductionDims`, `empty`.
- **CN**: 围绕 `notifyMatchFailure`, `getReductionDims`, `empty` 实现具体逻辑。

### Lines 262-274
```cpp
  unsigned reductionDimPos = dims[0];
  SmallVector<int64_t> loopRanges = op.getStaticLoopRanges();
  int64_t reductionDimSize = loopRanges[reductionDimPos];
  if (reductionDimSize == ShapedType::kDynamic ||
      reductionDimSize % splitFactor != 0 ||
      insertSplitDimension >= loopRanges.size())
    return b.notifyMatchFailure(
        op, "first reduction dimension not divisible by split factor");

  SmallVector<Operation *> combinerOps;
  if (!matchReduction(op.getRegionOutputArgs(), 0, combinerOps))
    return b.notifyMatchFailure(op, "cannot match a reduction pattern");

```
- **EN**: Implements logic around `getStaticLoopRanges`, `size`, `notifyMatchFailure`, `matchReduction`.
- **CN**: 围绕 `getStaticLoopRanges`, `size`, `notifyMatchFailure`, `matchReduction` 实现具体逻辑。

### Lines 275-285
```cpp
  SmallVector<TypedAttr> neutralElements;
  for (Operation *reductionOp : combinerOps) {
    std::optional<TypedAttr> neutralElement =
        arith::getNeutralElement(reductionOp);
    if (!neutralElement.has_value())
      return b.notifyMatchFailure(op, "cannot find neutral element.");
    neutralElements.push_back(*neutralElement);
  }
  if (!llvm::all_of(neutralElements, [](Attribute attr) { return attr; }))
    return b.notifyMatchFailure(op, "unknown reduction neutral");

```
- **EN**: Implements logic around `getNeutralElement`, `has_value`, `notifyMatchFailure`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `getNeutralElement`, `has_value`, `notifyMatchFailure`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 286-305
```cpp
  // TODO: relax this when multi-reduction support is available.
  if (op.getNumDpsInits() != static_cast<int64_t>(neutralElements.size()))
    return b.notifyMatchFailure(op, "expect one reduction per output");

  // Rewrite part.
  // Step 1. Build the intermediate outputs filled with the proper
  // neutralElements. Such outputs are of the same shape with an extra dimension
  // inserted at `insertSplitDimension`.
  //
  // Consider a minimal example where `k` is reduced:
  //     O(i, j) += I(i, j, k)
  // Assume i=3, j=5, k=128, splitFactor=16 and insertSplitDimension=0.
  // The compute is rewritten as:
  //   a. O_i(kk, i, j) += I(i, j, 16 * k + kk)
  //   b. O(i, j) += O_i(kk, i, j)
  // The intermediate tensor O_i is of shape (128/16)x3x5 == 8x3x5.
  Location loc = op->getLoc();
  MLIRContext *context = op.getContext();
  // For now assume outputs are 1-1 with reduction neutralElements.
  // TODO: generalize when multi-reduction support is available.
```
- **EN**: Implements logic around `getNumDpsInits`, `notifyMatchFailure`, `getLoc`, `getContext`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumDpsInits`, `notifyMatchFailure`, `getLoc`, `getContext` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 306-325
```cpp
  SmallVector<Value> newOutputs;
  newOutputs.reserve(op.getNumDpsInits());
  SmallVector<Operation *> emptyOrAllocTensorOps;
  SmallVector<linalg::FillOp> fillOps;
  fillOps.reserve(op.getNumDpsInits());
  for (auto it : llvm::zip(op.getDpsInitsMutable(), neutralElements)) {
    Value rankedTensor = std::get<0>(it).get();
    auto t = cast<RankedTensorType>(rankedTensor.getType());
    RankedTensorType newT = RankedTensorType::Builder(t).insertDim(
        reductionDimSize / splitFactor, insertSplitDimension);
    SmallVector<Value> dims =
        tensor::createDynamicDimValues(b, loc, rankedTensor);
    Value emptyOrAllocTensor;
    if (useAlloc) {
      emptyOrAllocTensor =
          bufferization::AllocTensorOp::create(b, loc, newT, dims);
    } else {
      emptyOrAllocTensor = tensor::EmptyOp::create(b, loc, newT.getShape(),
                                                   t.getElementType(), dims);
    }
```
- **EN**: Implements logic around `reserve`, `zip`, `get`, `getType`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reserve`, `zip`, `get`, `getType`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 326-345
```cpp
    Value constantOp = arith::ConstantOp::create(b, loc, std::get<1>(it));
    fillOps.push_back(linalg::FillOp::create(b, op->getLoc(), constantOp,
                                             emptyOrAllocTensor));
    newOutputs.push_back(fillOps.back().getResult(0));
    emptyOrAllocTensorOps.push_back(emptyOrAllocTensor.getDefiningOp());
  }

  // Step 2. Reindex / expand indexing maps.
  // Reindex existing input indexings: k -> k * splitFactor + k'.
  SmallVector<AffineMap> newMaps;
  newMaps.reserve(op->getNumOperands() + 1);
  for (OpOperand *o : op.getDpsInputOperands())
    newMaps.push_back(scaleReductionDim(op, *o, reductionDimPos, splitFactor));
  // Provision a new indexing for the shape-only tensor.
  auto nDims = op.getNumLoops() + 1;
  auto redDim = getAffineDimExpr(reductionDimPos, context);
  auto redDimP1 = getAffineDimExpr(reductionDimPos + 1, context);
  newMaps.push_back(AffineMap::get(nDims, 0, {redDim, redDimP1}, context));
  // Expand existing output indexings.
  // TODO: a subset of these may not reduce along reducePos and should be
```
- **EN**: Implements logic around `create`, `push_back`, `reserve`, `getDpsInputOperands`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `push_back`, `reserve`, `getDpsInputOperands`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 346-361
```cpp
  // reindexed: k -> k * splitFactor + k', when multi-reduction support is
  // available.
  for (OpOperand &o : op.getDpsInitsMutable())
    newMaps.push_back(insertParallelDim(op, o, reductionDimPos,
                                        reductionDimSize / splitFactor));

  // Step 3. Handle operands.
  // Compute the new input tensors.
  SmallVector<Value> newInputs = op.getDpsInputs();
  // Add a single shape-only tensor to carry the dimensions without resorting to
  // more complex inversions.
  newInputs.push_back(tensor::EmptyOp::create(
      b, loc, ArrayRef<int64_t>{reductionDimSize / splitFactor, splitFactor},
      b.getIntegerType(1)));
  // Output tensors are already good to go.

```
- **EN**: Implements logic around `getDpsInitsMutable`, `push_back`, `getDpsInputs`, `getIntegerType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDpsInitsMutable`, `push_back`, `getDpsInputs`, `getIntegerType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 362-374
```cpp
  // Step 4. Create the new op matching the original op with an extra parallel
  // dimension.
  auto iteratorTypes = op.getIteratorTypesArray();
  iteratorTypes.insert(iteratorTypes.begin() + reductionDimPos,
                       utils::IteratorType::parallel);
  GenericOp genericOp =
      GenericOp::create(b, loc, ValueRange(newOutputs).getTypes(), newInputs,
                        newOutputs, newMaps, iteratorTypes);
  b.inlineRegionBefore(op->getRegion(0), genericOp.getRegion(),
                       genericOp.getRegion().begin());
  genericOp.getRegion().front().insertArgument(reductionDimPos,
                                               b.getIntegerType(1), loc);

```
- **EN**: Implements logic around `getIteratorTypesArray`, `insert`, `create`, `inlineRegionBefore`, and 2 more symbols.
- **CN**: 围绕 `getIteratorTypesArray`, `insert`, `create`, `inlineRegionBefore`, and 2 more symbols 实现具体逻辑。

### Lines 375-389
```cpp
  // Step 5. Create new reduction ops that only reduce the newly added
  // dimensions from the previous op.
  // For now assume outputs are 1-1 with reduction ops.
  // TODO: a subset of these may not reduce in the first place and do not
  // require a new op, when multi-reduction support is available.
  // TODO: all results can be handled in a single GenericOp, when
  // multi-reduction support is available.
  SmallVector<LinalgOp> results;
  for (auto it :
       llvm::zip(genericOp->getResults(), op.getDpsInits(), combinerOps)) {
    Value reindexedOutput = std::get<0>(it);
    Value originalOutput = std::get<1>(it);
    auto originalOutputType = cast<RankedTensorType>(originalOutput.getType());
    Operation *combinerOp = std::get<2>(it);

```
- **EN**: Implements logic around `zip`, `get`, `getType`.
- **CN**: 围绕 `zip`, `get`, `getType` 实现具体逻辑。

### Lines 390-409
```cpp
    AffineMap map = b.getMultiDimIdentityMap(originalOutputType.getRank() + 1);
    SmallVector<AffineMap> indexingMaps = {
        map, map.dropResult(insertSplitDimension)};
    SmallVector<utils::IteratorType> reductionIteratorTypes(
        originalOutputType.getRank() + 1, utils::IteratorType::parallel);
    reductionIteratorTypes[insertSplitDimension] =
        utils::IteratorType::reduction;

    // clang-format off
    auto reductionOp = GenericOp::create(b,
        loc,
        originalOutputType,
        reindexedOutput,
        originalOutput,
        indexingMaps,
        reductionIteratorTypes,
        [combinerOp](OpBuilder &b, Location loc, ValueRange bbArgs) {
          Operation *clonedReductionOp = b.clone(*combinerOp);
          clonedReductionOp->setOperand(0, bbArgs[0]);
          clonedReductionOp->setOperand(1, bbArgs[1]);
```
- **EN**: Implements logic around `getMultiDimIdentityMap`, `dropResult`, `reductionIteratorTypes`, `getRank`, and 3 more symbols.
- **CN**: 围绕 `getMultiDimIdentityMap`, `dropResult`, `reductionIteratorTypes`, `getRank`, and 3 more symbols 实现具体逻辑。

### Lines 410-424
```cpp
          linalg::YieldOp::create(b, loc, clonedReductionOp->getResult(0));
        });
    // clang-format on

    results.push_back(reductionOp);
  }

  // TODO: extend when multi-reduction support is available.
  assert(fillOps.size() == results.size() && results.size() == 1);
  b.replaceOp(op, results.front()->getResults());
  return SplitReductionResult{emptyOrAllocTensorOps.front(), fillOps.front(),
                              cast<LinalgOp>(genericOp.getOperation()),
                              results.front()};
}

```
- **EN**: Implements logic around `create`, `push_back`, `assert`, `replaceOp`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `push_back`, `assert`, `replaceOp`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 425-435
```cpp
namespace {

struct LinalgSplitReduction : public OpInterfaceRewritePattern<LinalgOp> {
  /// Construct a generic pattern applied to all LinalgOp that verify `filter`.
  LinalgSplitReduction(MLIRContext *context,
                       ControlSplitReductionFn controlSplitReductionFn,
                       bool useAlloc = false, PatternBenefit benefit = 1)
      : OpInterfaceRewritePattern<LinalgOp>(context, benefit),
        controlSplitReductionFn(std::move(controlSplitReductionFn)),
        useAlloc(useAlloc) {}

```
- **EN**: Introduces declarations for `LinalgSplitReduction`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgSplitReduction` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 436-445
```cpp
  LogicalResult matchAndRewrite(LinalgOp op,
                                PatternRewriter &rewriter) const override {
    return splitReduction(rewriter, op, controlSplitReductionFn, useAlloc);
  }

private:
  ControlSplitReductionFn controlSplitReductionFn;
  bool useAlloc;
};

```
- **EN**: Implements logic around `matchAndRewrite`, `splitReduction`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `splitReduction` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 446-453
```cpp
} // namespace

void linalg::populateSplitReductionPattern(
    RewritePatternSet &patterns,
    const ControlSplitReductionFn &controlSplitReductionFn, bool useAlloc) {
  patterns.add<LinalgSplitReduction>(patterns.getContext(),
                                     controlSplitReductionFn, useAlloc);
}
```
- **EN**: Implements logic around `populateSplitReductionPattern`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateSplitReductionPattern`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Bufferization/IR/Bufferization.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/Tensor/Utils/Utils.h`, `mlir/IR/PatternMatch.h`
- **Standard-library headers / 标准库头文件**: `<optional>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (7), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
