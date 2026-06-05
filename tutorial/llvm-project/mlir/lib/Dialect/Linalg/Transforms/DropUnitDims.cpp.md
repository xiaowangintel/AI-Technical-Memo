# DropUnitDims.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/DropUnitDims.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements patterns/pass to remove usage of unit-extent dimensions to specify broadcasting in favor of more canonical representation of the computation.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- DropUnitDims.cpp - Pass to drop use of unit-extent for broadcasting ===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements patterns/pass to remove usage of unit-extent dimensions
// to specify broadcasting in favor of more canonical representation of the
// computation
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 15-33
```cpp
#include "mlir/Dialect/Linalg/Passes.h"

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/MemRef/Transforms/Transforms.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Tensor/Transforms/Transforms.h"
#include "mlir/Dialect/Utils/ReshapeOpsUtils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/Transforms/FoldUtils.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "mlir/Transforms/WalkPatternRewriteDriver.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Linalg/IR/Linalg.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Linalg/IR/Linalg.h`。

### Lines 34-47
```cpp
namespace mlir {
#define GEN_PASS_DEF_LINALGFOLDUNITEXTENTDIMSPASS
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

#define DEBUG_TYPE "linalg-drop-unit-dims"

using namespace mlir;
using namespace mlir::linalg;

namespace {
/// Pattern to move init operands to ins when all the loops are parallel and
/// blockArgument corresponding to init is used in the region. This is a fix-up
/// when unit reduction dimensions are all folded away. In this context, it
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 48-61
```cpp
/// becomes a elementwise generic op. E.g., it converts
///
///  %0 = tensor.empty() : tensor<1x1xf32>
///  %1 = linalg.fill
///    ins(%cst : f32)
///    outs(%0 : tensor<1x1xf32>) -> tensor<1x1xf32>
///  %2 = linalg.generic {indexing_maps = [affine_map<(d0) -> (0, d0, 0, 0)>,
///                                        affine_map<(d0) -> (0, d0)>],
///                       iterator_types = ["parallel"]}
///    ins(%arg0 : tensor<1x?x1x1xf32>)
///    outs(%1 : tensor<1x1xf32>) {
///  ^bb0(%in: f32, %out: f32):
///    %3 = arith.addf %in, %out : f32
///    linalg.yield %3 : f32
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 62-75
```cpp
///  } -> tensor<1x1xf32>
///
///  into
///
///  %0 = tensor.empty() : tensor<1x1xf32>
///  %1 = linalg.fill
///    ins(%cst : f32)
///    outs(%0 : tensor<1x1xf32>) -> tensor<1x1xf32>
///  %2 = tensor.empty() : tensor<1x1xf32>
///  %3 = linalg.generic {indexing_maps = [affine_map<(d0) -> (0, d0, 0, 0)>,
///                                        affine_map<(d0) -> (0, d0)>,
///                                        affine_map<(d0) -> (0, d0)>],
///                       iterator_types = ["parallel"]}
///   ins(%arg0, %1 : tensor<1x?x1x1xf32>, tensor<1x1xf32>)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 76-89
```cpp
///   outs(%2 : tensor<1x1xf32>) {
///  ^bb0(%in: f32, %in_0: f32, %out: f32):
///    %4 = arith.addf %in, %in_0 : f32
///    linalg.yield %4 : f32
///  } -> tensor<1x1xf32>
struct MoveInitOperandsToInput : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;
  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    if (!genericOp.hasPureTensorSemantics())
      return failure();
    if (genericOp.getNumParallelLoops() != genericOp.getNumLoops())
      return failure();

```
- **EN**: Introduces declarations for `MoveInitOperandsToInput`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `MoveInitOperandsToInput` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 90-114
```cpp
    auto outputOperands = genericOp.getDpsInitsMutable();
    SetVector<OpOperand *> candidates;
    for (OpOperand &op : outputOperands) {
      if (genericOp.getMatchingBlockArgument(&op).use_empty())
        continue;
      candidates.insert(&op);
    }

    if (candidates.empty())
      return failure();

    // Compute the modified indexing maps.
    int64_t origNumInput = genericOp.getNumDpsInputs();
    SmallVector<Value> newInputOperands = genericOp.getDpsInputs();
    SmallVector<AffineMap> indexingMaps = genericOp.getIndexingMapsArray();
    SmallVector<AffineMap> newIndexingMaps;
    newIndexingMaps.append(indexingMaps.begin(),
                           std::next(indexingMaps.begin(), origNumInput));
    for (OpOperand *op : candidates) {
      newInputOperands.push_back(op->get());
      newIndexingMaps.push_back(genericOp.getMatchingIndexingMap(op));
    }
    newIndexingMaps.append(std::next(indexingMaps.begin(), origNumInput),
                           indexingMaps.end());

```
- **EN**: Implements logic around `getDpsInitsMutable`, `getMatchingBlockArgument`, `insert`, `empty`, and 8 more symbols.
- **CN**: 围绕 `getDpsInitsMutable`, `getMatchingBlockArgument`, `insert`, `empty`, and 8 more symbols 实现具体逻辑。

### Lines 115-129
```cpp
    Location loc = genericOp.getLoc();
    SmallVector<Value> newOutputOperands =
        llvm::to_vector(genericOp.getDpsInits());
    for (OpOperand *op : candidates) {
      OpBuilder::InsertionGuard guard(rewriter);
      rewriter.setInsertionPointAfterValue(op->get());
      auto elemType = cast<ShapedType>(op->get().getType()).getElementType();
      auto empty = tensor::EmptyOp::create(
          rewriter, loc, tensor::getMixedSizes(rewriter, loc, op->get()),
          elemType);

      unsigned start = genericOp.getDpsInits().getBeginOperandIndex();
      newOutputOperands[op->getOperandNumber() - start] = empty.getResult();
    }

```
- **EN**: Implements logic around `getLoc`, `to_vector`, `guard`, `setInsertionPointAfterValue`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `to_vector`, `guard`, `setInsertionPointAfterValue`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 130-146
```cpp
    auto newOp = GenericOp::create(
        rewriter, loc, genericOp.getResultTypes(), newInputOperands,
        newOutputOperands, newIndexingMaps, genericOp.getIteratorTypesArray(),
        /*bodyBuild=*/nullptr, linalg::getPrunedAttributeList(genericOp));

    OpBuilder::InsertionGuard guard(rewriter);
    Region &region = newOp.getRegion();
    Block *block = rewriter.createBlock(&region);
    IRMapping mapper;
    for (auto bbarg : genericOp.getRegionInputArgs())
      mapper.map(bbarg, block->addArgument(bbarg.getType(), loc));

    for (OpOperand *op : candidates) {
      BlockArgument bbarg = genericOp.getMatchingBlockArgument(op);
      mapper.map(bbarg, block->addArgument(bbarg.getType(), loc));
    }

```
- **EN**: Implements logic around `create`, `getResultTypes`, `getIteratorTypesArray`, `getPrunedAttributeList`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResultTypes`, `getIteratorTypesArray`, `getPrunedAttributeList`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 147-164
```cpp
    for (OpOperand &op : outputOperands) {
      BlockArgument bbarg = genericOp.getMatchingBlockArgument(&op);
      if (candidates.count(&op))
        block->addArgument(bbarg.getType(), loc);
      else
        mapper.map(bbarg, block->addArgument(bbarg.getType(), loc));
    }

    for (auto &op : genericOp.getBody()->getOperations()) {
      rewriter.clone(op, mapper);
    }
    rewriter.replaceOp(genericOp, newOp.getResults());

    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `getMatchingBlockArgument`, `count`, `addArgument`, `map`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getMatchingBlockArgument`, `count`, `addArgument`, `map`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 165-178
```cpp
//===---------------------------------------------------------------------===//
// Drop loops that are unit-extents within Linalg operations.
//===---------------------------------------------------------------------===//

/// Implements a pass that canonicalizes the uses of unit-extent dimensions for
/// broadcasting. For example,
///
/// ```mlir
/// #accesses = [
///   affine_map<(d0, d1) -> (0, d1)>,
///   affine_map<(d0, d1) -> (d0, 0)>,
///   affine_map<(d0, d1) -> (d0, d1)>
/// ]
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 179-192
```cpp
/// #trait = {
///   indexing_maps = #accesses,
///   iterator_types = ["parallel", "parallel"],
///   library_call = "some_external_fn"
/// }
///
/// func @broadcast_test(%arg0 : tensor<5xf32>, %arg1 : tensor<5xf32>) ->
/// tensor<5x5xf32>
/// {
///   %0 = linalg.tensor_reshape %arg0 [affine_map<(d0, d1) -> (d0, d1)>] :
///        tensor<5xf32> into tensor<1x5xf32>
///   %1 = linalg.tensor_reshape %arg1 [affine_map<(d0, d1) -> (d0, d1)>] :
///        tensor<5xf32> into tensor<5x1xf32>
///   %2 = linalg.generic #trait %0, %1 {
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 193-206
```cpp
///        ^bb0(%arg2: f32, %arg3: f32):
///          %3 = arith.addf %arg2, %arg3 : f32
///          linalg.yield %3 : f32
///        } : tensor<1x5xf32>, tensor<5x1xf32> -> tensor<5x5xf32>
///   return %2 : tensor<5x5xf32>
/// }
///
/// would canonicalize to
///
/// ```mlir
/// #accesses = [
///   affine_map<(d0, d1) -> (d1)>,
///   affine_map<(d0, d1) -> (d0)>,
///   affine_map<(d0, d1) -> (d0, d1)>
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 207-220
```cpp
/// ]
///
/// #trait = {
///   indexing_maps = #accesses,
///   iterator_types = ["parallel", "parallel"],
///   library_call = "some_external_fn"
/// }
///
/// func @broadcast_test(%arg0 : tensor<5xf32>, %arg1 : tensor<5xf32>) ->
/// tensor<5x5xf32>
/// {
///   %0 = linalg.generic #trait %arg0, %arg1 {
///        ^bb0(%arg2: f32, %arg3: f32):
///          %3 = arith.addf %arg2, %arg3 : f32
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 221-247
```cpp
///          linalg.yield %3 : f32
///        } : tensor<5xf32>, tensor<5xf32> -> tensor<5x5xf32>
///   return %0 : tensor<5x5xf32>
/// }

/// Update the index accesses of linalg operations having index semantics.
static void
replaceUnitDimIndexOps(GenericOp genericOp,
                       const llvm::SmallDenseSet<unsigned> &unitDims,
                       RewriterBase &rewriter) {
  for (IndexOp indexOp :
       llvm::make_early_inc_range(genericOp.getBody()->getOps<IndexOp>())) {
    OpBuilder::InsertionGuard guard(rewriter);
    rewriter.setInsertionPoint(indexOp);
    if (unitDims.count(indexOp.getDim()) != 0) {
      rewriter.replaceOpWithNewOp<arith::ConstantIndexOp>(indexOp, 0);
    } else {
      // Update the dimension of the index operation if needed.
      unsigned droppedDims = llvm::count_if(
          unitDims, [&](unsigned dim) { return dim < indexOp.getDim(); });
      if (droppedDims != 0)
        rewriter.replaceOpWithNewOp<IndexOp>(indexOp,
                                             indexOp.getDim() - droppedDims);
    }
  }
}

```
- **EN**: Implements logic around `replaceUnitDimIndexOps`, `make_early_inc_range`, `guard`, `setInsertionPoint`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceUnitDimIndexOps`, `make_early_inc_range`, `guard`, `setInsertionPoint`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 248-269
```cpp
FailureOr<Value>
ControlDropUnitDims::expandValue(RewriterBase &rewriter, Location loc,
                                 Value result, Value origDest,
                                 ArrayRef<ReassociationIndices> reassociation,
                                 const ControlDropUnitDims &control) {
  // There are no results for memref outputs.
  auto origResultType = cast<RankedTensorType>(origDest.getType());
  if (origResultType.getEncoding() != nullptr) {
    // Do not expand tensors with encoding.
    return failure();
  }
  if (control.rankReductionStrategy ==
      ControlDropUnitDims::RankReductionStrategy::ExtractInsertSlice) {
    unsigned rank = origResultType.getRank();
    SmallVector<OpFoldResult> offsets(rank, rewriter.getIndexAttr(0));
    SmallVector<OpFoldResult> sizes =
        tensor::getMixedSizes(rewriter, loc, origDest);
    SmallVector<OpFoldResult> strides(rank, rewriter.getIndexAttr(1));
    return rewriter.createOrFold<tensor::InsertSliceOp>(
        loc, result, origDest, offsets, sizes, strides);
  }

```
- **EN**: Implements logic around `expandValue`, `getType`, `getEncoding`, `failure`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `expandValue`, `getType`, `getEncoding`, `failure`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 270-296
```cpp
  assert(control.rankReductionStrategy ==
             ControlDropUnitDims::RankReductionStrategy::ReassociativeReshape &&
         "unknown rank reduction strategy");
  return tensor::ExpandShapeOp::create(rewriter, loc, origResultType, result,
                                       reassociation)
      .getResult();
}

FailureOr<Value>
ControlDropUnitDims::collapseValue(RewriterBase &rewriter, Location loc,
                                   Value operand, ArrayRef<int64_t> targetShape,
                                   ArrayRef<ReassociationIndices> reassociation,
                                   const ControlDropUnitDims &control) {
  if (auto memrefType = dyn_cast<MemRefType>(operand.getType())) {
    if (!memrefType.getLayout().isIdentity()) {
      // Do not collapse memrefs with a non-identity layout.
      return failure();
    }
    if (control.rankReductionStrategy ==
        ControlDropUnitDims::RankReductionStrategy::ExtractInsertSlice) {
      FailureOr<Value> rankReducingExtract =
          memref::SubViewOp::rankReduceIfNeeded(rewriter, loc, operand,
                                                targetShape);
      assert(succeeded(rankReducingExtract) && "not a unit-extent collapse");
      return *rankReducingExtract;
    }

```
- **EN**: Implements logic around `assert`, `create`, `getResult`, `collapseValue`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `create`, `getResult`, `collapseValue`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 297-321
```cpp
    assert(
        control.rankReductionStrategy ==
            ControlDropUnitDims::RankReductionStrategy::ReassociativeReshape &&
        "unknown rank reduction strategy");
    MemRefLayoutAttrInterface layout;
    auto targetType = MemRefType::get(targetShape, memrefType.getElementType(),
                                      layout, memrefType.getMemorySpace());
    return memref::CollapseShapeOp::create(rewriter, loc, targetType, operand,
                                           reassociation)
        .getResult();
  }
  if (auto tensorType = dyn_cast<RankedTensorType>(operand.getType())) {
    if (tensorType.getEncoding() != nullptr) {
      // Do not collapse tensors with an encoding.
      return failure();
    }
    if (control.rankReductionStrategy ==
        ControlDropUnitDims::RankReductionStrategy::ExtractInsertSlice) {
      FailureOr<Value> rankReducingExtract =
          tensor::ExtractSliceOp::rankReduceIfNeeded(rewriter, loc, operand,
                                                     targetShape);
      assert(succeeded(rankReducingExtract) && "not a unit-extent collapse");
      return *rankReducingExtract;
    }

```
- **EN**: Implements logic around `assert`, `get`, `getMemorySpace`, `create`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `get`, `getMemorySpace`, `create`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 322-335
```cpp
    assert(
        control.rankReductionStrategy ==
            ControlDropUnitDims::RankReductionStrategy::ReassociativeReshape &&
        "unknown rank reduction strategy");
    auto targetType =
        RankedTensorType::get(targetShape, tensorType.getElementType());
    return tensor::CollapseShapeOp::create(rewriter, loc, targetType, operand,
                                           reassociation)
        .getResult();
  }
  llvm_unreachable("unsupported operand type");
}

/// Compute the modified metadata for an operands of operation
```
- **EN**: Implements logic around `assert`, `get`, `create`, `getResult`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `get`, `create`, `getResult` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 336-355
```cpp
/// whose unit dims are being dropped. Return the new indexing map
/// to use, the shape of the operand in the replacement op
/// and the `reassocation` to use to go from original operand shape
/// to modified operand shape.
struct UnitExtentReplacementInfo {
  AffineMap indexMap;
  SmallVector<ReassociationIndices> reassociation;
  SmallVector<int64_t> targetShape;
};
static UnitExtentReplacementInfo dropUnitExtentFromOperandMetadata(
    MLIRContext *context, IndexingMapOpInterface op, OpOperand *opOperand,
    llvm::SmallDenseMap<unsigned, unsigned> &oldDimsToNewDimsMap,
    ArrayRef<AffineExpr> dimReplacements) {
  UnitExtentReplacementInfo info;
  ReassociationIndices reassociationGroup;
  SmallVector<AffineExpr> newIndexExprs;
  AffineMap indexingMap = op.getMatchingIndexingMap(opOperand);
  SmallVector<int64_t> operandShape = op.getStaticOperandShape(opOperand);
  ArrayRef<AffineExpr> exprs = indexingMap.getResults();

```
- **EN**: Introduces declarations for `UnitExtentReplacementInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `UnitExtentReplacementInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 356-375
```cpp
  auto isUnitDim = [&](unsigned dim) {
    if (auto dimExpr = dyn_cast<AffineDimExpr>(exprs[dim])) {
      unsigned oldPosition = dimExpr.getPosition();
      return !oldDimsToNewDimsMap.count(oldPosition) &&
             (operandShape[dim] == 1);
    }
    // Handle the other case where the shape is 1, and is accessed using a
    // constant 0.
    if (operandShape[dim] == 1) {
      // Use the new expression after replacing dimensions that will be dropped
      // here to handle cases where an affine expression with multiple
      // dimensions (e.g., `d0 + d2`) can be simplified to 0 after dropping all
      // dimensions used in the expression (`d0` and `d2` in this example).
      AffineExpr newExpr = exprs[dim].replaceDims(dimReplacements);
      auto constAffineExpr = dyn_cast<AffineConstantExpr>(newExpr);
      return constAffineExpr && constAffineExpr.getValue() == 0;
    }
    return false;
  };

```
- **EN**: Implements logic around `getPosition`, `count`, `replaceDims`, `getValue`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPosition`, `count`, `replaceDims`, `getValue` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 376-398
```cpp
  unsigned dim = 0;
  while (dim < operandShape.size() && isUnitDim(dim))
    reassociationGroup.push_back(dim++);
  while (dim < operandShape.size()) {
    assert(!isUnitDim(dim) && "expected non unit-extent");
    reassociationGroup.push_back(dim);
    AffineExpr newExpr = exprs[dim].replaceDims(dimReplacements);
    newIndexExprs.push_back(newExpr);
    info.targetShape.push_back(operandShape[dim]);
    ++dim;
    // Fold all following dimensions that are unit-extent.
    while (dim < operandShape.size() && isUnitDim(dim)) {
      reassociationGroup.push_back(dim++);
    }
    info.reassociation.push_back(reassociationGroup);
    reassociationGroup.clear();
  }
  info.indexMap =
      AffineMap::get(oldDimsToNewDimsMap.size(), indexingMap.getNumSymbols(),
                     newIndexExprs, context);
  return info;
}

```
- **EN**: Implements logic around `size`, `push_back`, `assert`, `replaceDims`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `size`, `push_back`, `assert`, `replaceDims`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 399-412
```cpp
FailureOr<DropUnitDimsResult>
linalg::dropUnitDims(RewriterBase &rewriter, IndexingMapOpInterface op,
                     const DroppedUnitDimsBuilder &droppedUnitDimsBuilder,
                     const ControlDropUnitDims &options) {
  auto dpsOp = dyn_cast<DestinationStyleOpInterface>(op.getOperation());
  if (!dpsOp) {
    return rewriter.notifyMatchFailure(
        op, "op should implement DestinationStyleOpInterface");
  }

  SmallVector<AffineMap> indexingMaps = op.getIndexingMapsArray();
  if (indexingMaps.empty())
    return failure();

```
- **EN**: Implements logic around `dropUnitDims`, `getOperation`, `notifyMatchFailure`, `getIndexingMapsArray`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `dropUnitDims`, `getOperation`, `notifyMatchFailure`, `getIndexingMapsArray`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 413-426
```cpp
  // 1. Check if any of the iteration dimensions are unit-trip count. They will
  //    end up being unit-trip count if they are used to index into a unit-dim
  //    tensor/memref.
  AffineMap invertedMap =
      inversePermutation(concatAffineMaps(indexingMaps, rewriter.getContext()));
  if (!invertedMap) {
    return rewriter.notifyMatchFailure(op,
                                       "invalid indexing maps for operation");
  }

  SmallVector<int64_t> allShapesSizes;
  for (OpOperand &opOperand : op->getOpOperands())
    llvm::append_range(allShapesSizes, op.getStaticOperandShape(&opOperand));

```
- **EN**: Implements logic around `inversePermutation`, `notifyMatchFailure`, `getOpOperands`, `append_range`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `inversePermutation`, `notifyMatchFailure`, `getOpOperands`, `append_range` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 427-443
```cpp
  // 1a. Get the allowed list of dimensions to drop from the `options`.
  SmallVector<unsigned> allowedUnitDims = options.controlFn(op);
  if (allowedUnitDims.empty()) {
    return rewriter.notifyMatchFailure(
        op, "control function returns no allowed unit dims to prune");
  }
  llvm::SmallDenseSet<unsigned> unitDimsFilter(allowedUnitDims.begin(),
                                               allowedUnitDims.end());
  llvm::SmallDenseSet<unsigned> unitDims;
  for (const auto &expr : enumerate(invertedMap.getResults())) {
    if (AffineDimExpr dimExpr = dyn_cast<AffineDimExpr>(expr.value())) {
      if (allShapesSizes[dimExpr.getPosition()] == 1 &&
          unitDimsFilter.count(expr.index()))
        unitDims.insert(expr.index());
    }
  }

```
- **EN**: Implements logic around `controlFn`, `empty`, `notifyMatchFailure`, `unitDimsFilter`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `controlFn`, `empty`, `notifyMatchFailure`, `unitDimsFilter`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 444-460
```cpp
  // 2. Compute the new loops of the modified op by dropping the one-trip
  //    count loops.
  llvm::SmallDenseMap<unsigned, unsigned> oldDimToNewDimMap;
  SmallVector<AffineExpr> dimReplacements;
  unsigned newDims = 0;
  for (auto index : llvm::seq<int64_t>(op.getStaticLoopRanges().size())) {
    if (unitDims.count(index)) {
      dimReplacements.push_back(
          getAffineConstantExpr(0, rewriter.getContext()));
    } else {
      oldDimToNewDimMap[index] = newDims;
      dimReplacements.push_back(
          getAffineDimExpr(newDims, rewriter.getContext()));
      newDims++;
    }
  }

```
- **EN**: Implements logic around `seq`, `count`, `push_back`, `getAffineConstantExpr`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `seq`, `count`, `push_back`, `getAffineConstantExpr`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 461-487
```cpp
  // 3. For each of the operands, find the
  //    - modified affine map to use.
  //    - shape of the operands after the unit-dims are dropped.
  //    - the reassociation indices used to convert from the original
  //      operand type to modified operand (needed only when using reshapes
  //      for rank reduction strategy)
  // Note that the indexing maps might need changing even if there are no
  // unit dimensions that are dropped to handle cases where `0` is used to
  // access a unit-extent tensor. Consider moving this out of this specific
  // transformation as a stand-alone transformation. Kept here right now due
  // to legacy.
  SmallVector<AffineMap> newIndexingMaps;
  SmallVector<SmallVector<ReassociationIndices>> reassociations;
  SmallVector<SmallVector<int64_t>> targetShapes;
  SmallVector<bool> collapsed;
  for (OpOperand &opOperand : op->getOpOperands()) {
    auto indexingMap = op.getMatchingIndexingMap(&opOperand);
    auto replacementInfo =
        dropUnitExtentFromOperandMetadata(rewriter.getContext(), op, &opOperand,
                                          oldDimToNewDimMap, dimReplacements);
    reassociations.push_back(replacementInfo.reassociation);
    newIndexingMaps.push_back(replacementInfo.indexMap);
    targetShapes.push_back(replacementInfo.targetShape);
    collapsed.push_back(!(replacementInfo.indexMap.getNumResults() ==
                          indexingMap.getNumResults()));
  }

```
- **EN**: Implements logic around `getOpOperands`, `getMatchingIndexingMap`, `dropUnitExtentFromOperandMetadata`, `push_back`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOpOperands`, `getMatchingIndexingMap`, `dropUnitExtentFromOperandMetadata`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 488-515
```cpp
  // Abort if the indexing maps of the result operation are not invertible
  // (i.e. not legal) or if no dimension was reduced.
  if (newIndexingMaps == indexingMaps ||
      !inversePermutation(
          concatAffineMaps(newIndexingMaps, rewriter.getContext())))
    return failure();

  Location loc = op.getLoc();
  // 4. For each of the operands, collapse the operand to convert
  //    from original shape to shape in the modified operation if needed,
  //    either through use of reshapes or rank-reducing slices as
  //    specified in `options`.
  //    Abort if one of the operands cannot be collapsed.
  SmallVector<Value> newOperands;
  for (OpOperand &opOperand : op->getOpOperands()) {
    int64_t idx = opOperand.getOperandNumber();
    if (!collapsed[idx]) {
      newOperands.push_back(opOperand.get());
      continue;
    }
    FailureOr<Value> collapsed =
        options.collapseFn(rewriter, loc, opOperand.get(), targetShapes[idx],
                           reassociations[idx], options);
    if (failed(collapsed)) {
      // Abort if the operand could not be collapsed.
      return failure();
    }
    newOperands.push_back(collapsed.value());
```
- **EN**: Implements logic around `inversePermutation`, `concatAffineMaps`, `failure`, `getLoc`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `inversePermutation`, `concatAffineMaps`, `failure`, `getLoc`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策。

### Lines 516-542
```cpp
  }

  IndexingMapOpInterface replacementOp = droppedUnitDimsBuilder(
      loc, rewriter, op, newOperands, newIndexingMaps, unitDims);

  // 6. If any result type changes, insert a reshape/slice to convert from the
  //    original type to the new type.
  //    Abort the transformation if the result cannot be expanded back to its
  //    original shape.
  SmallVector<Value> resultReplacements;
  for (auto [index, result] : llvm::enumerate(replacementOp->getResults())) {
    unsigned opOperandIndex = index + dpsOp.getNumDpsInputs();
    Value origDest = dpsOp.getDpsInitOperand(index)->get();
    if (!collapsed[opOperandIndex]) {
      resultReplacements.push_back(result);
      continue;
    }
    FailureOr<Value> expanded =
        options.expandFn(rewriter, loc, result, origDest,
                         reassociations[opOperandIndex], options);
    if (failed(expanded)) {
      // Abort if expansion is not successful.
      return failure();
    }
    resultReplacements.push_back(expanded.value());
  }

```
- **EN**: Implements logic around `droppedUnitDimsBuilder`, `enumerate`, `getNumDpsInputs`, `getDpsInitOperand`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `droppedUnitDimsBuilder`, `enumerate`, `getNumDpsInputs`, `getDpsInitOperand`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策。

### Lines 543-564
```cpp
  return DropUnitDimsResult{replacementOp, resultReplacements};
}

FailureOr<DropUnitDimsResult>
linalg::dropUnitDims(RewriterBase &rewriter, GenericOp genericOp,
                     const ControlDropUnitDims &options) {

  DroppedUnitDimsBuilder build =
      [](Location loc, OpBuilder &b, IndexingMapOpInterface op,
         ArrayRef<Value> newOperands, ArrayRef<AffineMap> newIndexingMaps,
         const llvm::SmallDenseSet<unsigned> &droppedDims)
      -> IndexingMapOpInterface {
    auto genericOp = cast<GenericOp>(op);
    // Compute the iterator types of the modified op by dropping the one-trip
    // count loops.
    SmallVector<utils::IteratorType> newIteratorTypes;
    for (auto [index, attr] :
         llvm::enumerate(genericOp.getIteratorTypesArray())) {
      if (!droppedDims.count(index))
        newIteratorTypes.push_back(attr);
    }

```
- **EN**: Implements logic around `dropUnitDims`, `enumerate`, `count`, `push_back`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `dropUnitDims`, `enumerate`, `count`, `push_back` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 565-584
```cpp
    // Create the `linalg.generic` operation with the new operands,
    //    indexing maps, iterator types and result types.
    ArrayRef<Value> newInputs =
        ArrayRef<Value>(newOperands).take_front(genericOp.getNumDpsInputs());
    ArrayRef<Value> newOutputs =
        ArrayRef<Value>(newOperands).take_back(genericOp.getNumDpsInits());
    SmallVector<Type> resultTypes;
    resultTypes.reserve(genericOp.getNumResults());
    for (unsigned i : llvm::seq<unsigned>(0, genericOp.getNumResults()))
      resultTypes.push_back(newOutputs[i].getType());
    GenericOp replacementOp =
        GenericOp::create(b, loc, resultTypes, newInputs, newOutputs,
                          newIndexingMaps, newIteratorTypes);
    b.cloneRegionBefore(genericOp.getRegion(), replacementOp.getRegion(),
                        replacementOp.getRegion().begin());
    // 5a. Replace `linalg.index` operations that refer to the dropped unit
    //     dimensions.
    IRRewriter rewriter(b);
    replaceUnitDimIndexOps(replacementOp, droppedDims, rewriter);

```
- **EN**: Implements logic around `ArrayRef`, `reserve`, `seq`, `push_back`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ArrayRef`, `reserve`, `seq`, `push_back`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 585-607
```cpp
    return replacementOp;
  };

  return dropUnitDims(rewriter, genericOp, build, options);
}

namespace {
struct DropUnitDims : public OpRewritePattern<GenericOp> {
  DropUnitDims(MLIRContext *context, ControlDropUnitDims options = {},
               PatternBenefit benefit = 1)
      : OpRewritePattern(context, benefit), options(std::move(options)) {}

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    FailureOr<DropUnitDimsResult> result =
        dropUnitDims(rewriter, genericOp, options);
    if (failed(result)) {
      return failure();
    }
    rewriter.replaceOp(genericOp, result->replacements);
    return success();
  }

```
- **EN**: Introduces declarations for `DropUnitDims`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DropUnitDims` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 608-622
```cpp
private:
  ControlDropUnitDims options;
};
} // namespace

//===---------------------------------------------------------------------===//
// Drop dimensions that are unit-extents within tensor operations.
//===---------------------------------------------------------------------===//

namespace {
struct DropPadUnitDims : public OpRewritePattern<tensor::PadOp> {
  DropPadUnitDims(MLIRContext *context, ControlDropUnitDims options = {},
                  PatternBenefit benefit = 1)
      : OpRewritePattern(context, benefit), options(std::move(options)) {}

```
- **EN**: Introduces declarations for `DropPadUnitDims`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DropPadUnitDims` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 623-636
```cpp
  LogicalResult matchAndRewrite(tensor::PadOp padOp,
                                PatternRewriter &rewriter) const override {
    // 1a. Get the allowed list of dimensions to drop from the `options`.
    SmallVector<unsigned> allowedUnitDims = options.controlFn(padOp);
    if (allowedUnitDims.empty()) {
      return rewriter.notifyMatchFailure(
          padOp, "control function returns no allowed unit dims to prune");
    }

    if (padOp.getSourceType().getEncoding()) {
      return rewriter.notifyMatchFailure(
          padOp, "cannot collapse dims of tensor with encoding");
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `controlFn`, `empty`, `notifyMatchFailure`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `controlFn`, `empty`, `notifyMatchFailure`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 637-650
```cpp
    // Fail for non-constant padding values. The body of the pad could
    // depend on the padding indices and/or properties of the padded
    // tensor so for now we fail.
    // TODO: Support non-constant padding values.
    Value paddingVal = padOp.getConstantPaddingValue();
    if (!paddingVal) {
      return rewriter.notifyMatchFailure(
          padOp, "unimplemented: non-constant padding value");
    }

    ArrayRef<int64_t> sourceShape = padOp.getSourceType().getShape();
    ArrayRef<int64_t> resultShape = padOp.getResultType().getShape();
    int64_t padRank = sourceShape.size();

```
- **EN**: Implements logic around `getConstantPaddingValue`, `notifyMatchFailure`, `getSourceType`, `getResultType`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getConstantPaddingValue`, `notifyMatchFailure`, `getSourceType`, `getResultType`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 651-675
```cpp
    auto isStaticZero = [](OpFoldResult f) {
      return getConstantIntValue(f) == 0;
    };

    llvm::SmallDenseSet<unsigned> unitDimsFilter(allowedUnitDims.begin(),
                                                 allowedUnitDims.end());
    llvm::SmallDenseSet<unsigned> unitDims;
    SmallVector<int64_t> newShape;
    SmallVector<int64_t> newResultShape;
    SmallVector<OpFoldResult> newLowPad;
    SmallVector<OpFoldResult> newHighPad;
    for (const auto [dim, size, outSize, low, high] : zip_equal(
             llvm::seq(static_cast<int64_t>(0), padRank), sourceShape,
             resultShape, padOp.getMixedLowPad(), padOp.getMixedHighPad())) {
      if (unitDimsFilter.contains(dim) && size == 1 && isStaticZero(low) &&
          isStaticZero(high)) {
        unitDims.insert(dim);
      } else {
        newShape.push_back(size);
        newResultShape.push_back(outSize);
        newLowPad.push_back(low);
        newHighPad.push_back(high);
      }
    }

```
- **EN**: Implements logic around `getConstantIntValue`, `unitDimsFilter`, `end`, `zip_equal`, and 6 more symbols.
- **CN**: 围绕 `getConstantIntValue`, `unitDimsFilter`, `end`, `zip_equal`, and 6 more symbols 实现具体逻辑。

### Lines 676-695
```cpp
    if (unitDims.empty()) {
      return rewriter.notifyMatchFailure(padOp, "no unit dims to collapse");
    }

    ReassociationIndices reassociationGroup;
    SmallVector<ReassociationIndices> reassociationMap;
    int64_t dim = 0;
    while (dim < padRank && unitDims.contains(dim))
      reassociationGroup.push_back(dim++);
    while (dim < padRank) {
      assert(!unitDims.contains(dim) && "expected non unit-extent");
      reassociationGroup.push_back(dim);
      dim++;
      // Fold all following dimensions that are unit-extent.
      while (dim < padRank && unitDims.contains(dim))
        reassociationGroup.push_back(dim++);
      reassociationMap.push_back(reassociationGroup);
      reassociationGroup.clear();
    }

```
- **EN**: Implements logic around `empty`, `notifyMatchFailure`, `contains`, `push_back`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `empty`, `notifyMatchFailure`, `contains`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并执行面向规范化或折叠的推理。

### Lines 696-709
```cpp
    FailureOr<Value> collapsedSource =
        options.collapseFn(rewriter, padOp.getLoc(), padOp.getSource(),
                           newShape, reassociationMap, options);
    if (failed(collapsedSource)) {
      return rewriter.notifyMatchFailure(padOp, "Failed to collapse source");
    }

    auto newResultType = RankedTensorType::get(
        newResultShape, padOp.getResultType().getElementType());
    auto newPadOp = tensor::PadOp::create(
        rewriter, padOp.getLoc(), /*result=*/newResultType,
        collapsedSource.value(), newLowPad, newHighPad, paddingVal,
        padOp.getNofold());

```
- **EN**: Implements logic around `collapseFn`, `failed`, `notifyMatchFailure`, `get`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `collapseFn`, `failed`, `notifyMatchFailure`, `get`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 710-727
```cpp
    Value dest = padOp.getResult();
    if (options.rankReductionStrategy ==
        ControlDropUnitDims::RankReductionStrategy::ExtractInsertSlice) {
      SmallVector<OpFoldResult> expandedSizes;
      int64_t numUnitDims = 0;
      for (auto dim : llvm::seq(static_cast<int64_t>(0), padRank)) {
        if (unitDims.contains(dim)) {
          expandedSizes.push_back(rewriter.getIndexAttr(1));
          numUnitDims++;
          continue;
        }
        expandedSizes.push_back(tensor::getMixedSize(
            rewriter, padOp.getLoc(), newPadOp, dim - numUnitDims));
      }
      dest = tensor::EmptyOp::create(rewriter, padOp.getLoc(), expandedSizes,
                                     padOp.getResultType().getElementType());
    }

```
- **EN**: Implements logic around `getResult`, `seq`, `contains`, `push_back`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getResult`, `seq`, `contains`, `push_back`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 728-742
```cpp
    FailureOr<Value> expandedValue =
        options.expandFn(rewriter, padOp.getLoc(), newPadOp.getResult(), dest,
                         reassociationMap, options);
    if (failed(expandedValue)) {
      return rewriter.notifyMatchFailure(padOp, "Failed to expand result");
    }
    rewriter.replaceOp(padOp, expandedValue.value());
    return success();
  }

private:
  ControlDropUnitDims options;
};
} // namespace

```
- **EN**: Implements logic around `expandFn`, `failed`, `notifyMatchFailure`, `replaceOp`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `expandFn`, `failed`, `notifyMatchFailure`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 743-759
```cpp
namespace {
/// Convert `extract_slice` operations to rank-reduced versions.
struct RankReducedExtractSliceOp
    : public OpRewritePattern<tensor::ExtractSliceOp> {
  using OpRewritePattern<tensor::ExtractSliceOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(tensor::ExtractSliceOp sliceOp,
                                PatternRewriter &rewriter) const override {
    RankedTensorType resultType = sliceOp.getType();
    SmallVector<OpFoldResult> targetShape;
    for (auto size : resultType.getShape())
      targetShape.push_back(rewriter.getIndexAttr(size));
    auto reassociation = getReassociationMapForFoldingUnitDims(targetShape);
    if (!reassociation ||
        reassociation->size() == static_cast<size_t>(resultType.getRank()))
      return failure();

```
- **EN**: Introduces declarations for `RankReducedExtractSliceOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `RankReducedExtractSliceOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 760-776
```cpp
    SmallVector<OpFoldResult> offsets = sliceOp.getMixedOffsets();
    SmallVector<OpFoldResult> strides = sliceOp.getMixedStrides();
    SmallVector<OpFoldResult> sizes = sliceOp.getMixedSizes();
    auto rankReducedType = cast<RankedTensorType>(
        tensor::ExtractSliceOp::inferCanonicalRankReducedResultType(
            reassociation->size(), sliceOp.getSourceType(), sizes));

    Location loc = sliceOp.getLoc();
    Value newSlice = tensor::ExtractSliceOp::create(
        rewriter, loc, rankReducedType, sliceOp.getSource(), offsets, sizes,
        strides);
    rewriter.replaceOpWithNewOp<tensor::ExpandShapeOp>(
        sliceOp, resultType, newSlice, *reassociation);
    return success();
  }
};

```
- **EN**: Implements logic around `getMixedOffsets`, `getMixedStrides`, `getMixedSizes`, `inferCanonicalRankReducedResultType`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMixedOffsets`, `getMixedStrides`, `getMixedSizes`, `inferCanonicalRankReducedResultType`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 777-793
```cpp
/// Convert `insert_slice` operations to rank-reduced versions.
/// This patterns works with both InsertSliceOp and ParallelInsertSliceOp.
template <typename InsertOpTy>
struct RankReducedInsertSliceOp : public OpRewritePattern<InsertOpTy> {
  using OpRewritePattern<InsertOpTy>::OpRewritePattern;

  LogicalResult matchAndRewrite(InsertOpTy insertSliceOp,
                                PatternRewriter &rewriter) const override {
    RankedTensorType sourceType = insertSliceOp.getSourceType();
    SmallVector<OpFoldResult> targetShape;
    for (auto size : sourceType.getShape())
      targetShape.push_back(rewriter.getIndexAttr(size));
    auto reassociation = getReassociationMapForFoldingUnitDims(targetShape);
    if (!reassociation ||
        reassociation->size() == static_cast<size_t>(sourceType.getRank()))
      return failure();

```
- **EN**: Introduces declarations for `RankReducedInsertSliceOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `RankReducedInsertSliceOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 794-814
```cpp
    Location loc = insertSliceOp.getLoc();
    tensor::CollapseShapeOp reshapedSource;
    {
      OpBuilder::InsertionGuard g(rewriter);
      // The only difference between InsertSliceOp and ParallelInsertSliceOp
      // is the insertion point is just before the ParallelCombiningOp in the
      // parallel case.
      if (std::is_same<InsertOpTy, tensor::ParallelInsertSliceOp>::value)
        rewriter.setInsertionPoint(insertSliceOp->getParentOp());
      reshapedSource = tensor::CollapseShapeOp::create(
          rewriter, loc, insertSliceOp.getSource(), *reassociation);
    }
    rewriter.replaceOpWithNewOp<InsertOpTy>(
        insertSliceOp, reshapedSource, insertSliceOp.getDest(),
        insertSliceOp.getMixedOffsets(), insertSliceOp.getMixedSizes(),
        insertSliceOp.getMixedStrides());
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `getLoc`, `g`, `setInsertionPoint`, `create`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `g`, `setInsertionPoint`, `create`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 815-842
```cpp
/// Patterns that are used to canonicalize the use of unit-extent dims for
/// broadcasting.
void mlir::linalg::populateFoldUnitExtentDimsPatterns(
    RewritePatternSet &patterns, linalg::ControlDropUnitDims &options) {
  auto *context = patterns.getContext();
  patterns.add<DropUnitDims>(context, options);
  patterns.add<DropPadUnitDims>(context, options);
}

void mlir::linalg::populateFoldUnitExtentDimsCanonicalizationPatterns(
    RewritePatternSet &patterns, linalg::ControlDropUnitDims &options) {
  auto *context = patterns.getContext();
  bool reassociativeReshape =
      options.rankReductionStrategy ==
      linalg::ControlDropUnitDims::RankReductionStrategy::ReassociativeReshape;
  if (reassociativeReshape) {
    patterns.add<RankReducedExtractSliceOp,
                 RankReducedInsertSliceOp<tensor::InsertSliceOp>,
                 RankReducedInsertSliceOp<tensor::ParallelInsertSliceOp>>(
        context);
    tensor::CollapseShapeOp::getCanonicalizationPatterns(patterns, context);
    tensor::ExpandShapeOp::getCanonicalizationPatterns(patterns, context);
  }
  linalg::FillOp::getCanonicalizationPatterns(patterns, context);
  tensor::EmptyOp::getCanonicalizationPatterns(patterns, context);
  tensor::populateFoldTensorEmptyPatterns(patterns);
  memref::populateResolveRankedShapedTypeResultDimsPatterns(patterns);
  memref::populateResolveShapedTypeResultDimsPatterns(patterns);
```
- **EN**: Implements logic around `populateFoldUnitExtentDimsPatterns`, `getContext`, `add`, `populateFoldUnitExtentDimsCanonicalizationPatterns`, and 5 more symbols; this block performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateFoldUnitExtentDimsPatterns`, `getContext`, `add`, `populateFoldUnitExtentDimsCanonicalizationPatterns`, and 5 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 843-865
```cpp
}

void mlir::linalg::populateMoveInitOperandsToInputPattern(
    RewritePatternSet &patterns) {
  patterns.add<MoveInitOperandsToInput>(patterns.getContext());
}

namespace {
/// Pass that removes unit-extent dims within generic ops.
struct LinalgFoldUnitExtentDimsPass
    : public impl::LinalgFoldUnitExtentDimsPassBase<
          LinalgFoldUnitExtentDimsPass> {
  using impl::LinalgFoldUnitExtentDimsPassBase<
      LinalgFoldUnitExtentDimsPass>::LinalgFoldUnitExtentDimsPassBase;
  void runOnOperation() override {
    Operation *op = getOperation();
    MLIRContext *context = op->getContext();
    ControlDropUnitDims options;
    if (useRankReducingSlices) {
      options.rankReductionStrategy = linalg::ControlDropUnitDims::
          RankReductionStrategy::ExtractInsertSlice;
    }

```
- **EN**: Introduces declarations for `LinalgFoldUnitExtentDimsPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgFoldUnitExtentDimsPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 866-883
```cpp
    // Apply fold unit extent dims patterns with walk-based driver.
    {
      RewritePatternSet patterns(context);
      linalg::populateFoldUnitExtentDimsPatterns(patterns, options);
      walkAndApplyPatterns(op, std::move(patterns));
    }

    // Apply canonicalization patterns with greedy driver.
    {
      RewritePatternSet patterns(context);
      populateMoveInitOperandsToInputPattern(patterns);
      linalg::populateFoldUnitExtentDimsCanonicalizationPatterns(patterns,
                                                                 options);
      (void)applyPatternsGreedily(op, std::move(patterns));
    }
  }
};

```
- **EN**: Implements logic around `patterns`, `populateFoldUnitExtentDimsPatterns`, `walkAndApplyPatterns`, `populateMoveInitOperandsToInputPattern`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `patterns`, `populateFoldUnitExtentDimsPatterns`, `walkAndApplyPatterns`, `populateMoveInitOperandsToInputPattern`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 884-906
```cpp
} // namespace

namespace {

/// Returns reassociation indices for collapsing/expanding a
/// tensor of rank `rank` at position `pos`.
static SmallVector<ReassociationIndices>
getReassociationForReshapeAtDim(int64_t rank, int64_t pos) {
  SmallVector<ReassociationIndices> reassociation(rank - 1, {0, 1});
  bool lastDim = pos == rank - 1;
  if (rank > 2) {
    for (int64_t i = 0; i < rank - 1; i++) {
      if (i == pos || (lastDim && i == pos - 1))
        reassociation[i] = ReassociationIndices{i, i + 1};
      else if (i < pos)
        reassociation[i] = ReassociationIndices{i};
      else
        reassociation[i] = ReassociationIndices{i + 1};
    }
  }
  return reassociation;
}

```
- **EN**: Implements logic around `getReassociationForReshapeAtDim`, `reassociation`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getReassociationForReshapeAtDim`, `reassociation` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 907-923
```cpp
/// Returns a collapsed `val` where the collapsing occurs at dim `pos`.
/// If `pos < 0`, then don't collapse.
static Value collapseSingletonDimAt(PatternRewriter &rewriter, Value val,
                                    int64_t pos) {
  if (pos < 0)
    return val;
  auto valType = cast<ShapedType>(val.getType());
  SmallVector<int64_t> collapsedShape(valType.getShape());
  collapsedShape.erase(collapsedShape.begin() + pos);
  ControlDropUnitDims control{};
  FailureOr<Value> collapsed = control.collapseFn(
      rewriter, val.getLoc(), val, collapsedShape,
      getReassociationForReshapeAtDim(valType.getRank(), pos), control);
  assert(llvm::succeeded(collapsed) && "Collapsing the value failed");
  return collapsed.value();
}

```
- **EN**: Implements logic around `collapseSingletonDimAt`, `getType`, `collapsedShape`, `erase`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `collapseSingletonDimAt`, `getType`, `collapsedShape`, `erase`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 924-946
```cpp
/// Base class for all rank reduction patterns for contraction ops
/// with unit dimensions.  All patterns should convert one named op
/// to another named op.  Intended to reduce only one iteration space dim
/// at a time.
/// Reducing multiple dims will happen with recusive application of
/// pattern rewrites.
template <typename FromOpTy, typename ToOpTy>
struct RankReduceContractionOps : OpRewritePattern<FromOpTy> {
  using OpRewritePattern<FromOpTy>::OpRewritePattern;

  /// Collapse all collapsable operands.
  SmallVector<Value>
  collapseOperands(PatternRewriter &rewriter, ArrayRef<Value> operands,
                   ArrayRef<int64_t> operandCollapseDims) const {
    assert(operandCollapseDims.size() == 3 && operands.size() == 3 &&
           "expected 3 operands and dims");
    return llvm::map_to_vector(
        llvm::zip(operands, operandCollapseDims), [&](auto pair) {
          return collapseSingletonDimAt(rewriter, std::get<0>(pair),
                                        std::get<1>(pair));
        });
  }

```
- **EN**: Introduces declarations for `for`, `RankReduceContractionOps`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `for`, `RankReduceContractionOps` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 947-961
```cpp
  /// Expand result tensor.
  Value expandResult(PatternRewriter &rewriter, Value result,
                     RankedTensorType expandedType, int64_t dim) const {
    return tensor::ExpandShapeOp::create(
        rewriter, result.getLoc(), expandedType, result,
        getReassociationForReshapeAtDim(expandedType.getRank(), dim));
  }

  LogicalResult matchAndRewrite(FromOpTy contractionOp,
                                PatternRewriter &rewriter) const override {
    if (contractionOp.hasUserDefinedMaps()) {
      return rewriter.notifyMatchFailure(
          contractionOp, "ops with user-defined maps are not supported");
    }

```
- **EN**: Implements logic around `expandResult`, `create`, `getLoc`, `getReassociationForReshapeAtDim`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `expandResult`, `create`, `getLoc`, `getReassociationForReshapeAtDim`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 962-977
```cpp
    auto loc = contractionOp.getLoc();
    auto inputs = contractionOp.getDpsInputs();
    auto inits = contractionOp.getDpsInits();
    if (inputs.size() != 2 || inits.size() != 1)
      return rewriter.notifyMatchFailure(contractionOp,
                                         "expected 2 inputs and 1 init");
    auto lhs = inputs[0];
    auto rhs = inputs[1];
    auto init = inits[0];
    SmallVector<Value> operands{lhs, rhs, init};

    SmallVector<int64_t> operandUnitDims;
    if (failed(getOperandUnitDims(contractionOp, operandUnitDims)))
      return rewriter.notifyMatchFailure(contractionOp,
                                         "no reducable dims found");

```
- **EN**: Implements logic around `getLoc`, `getDpsInputs`, `getDpsInits`, `size`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getLoc`, `getDpsInputs`, `getDpsInits`, `size`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 978-995
```cpp
    SmallVector<Value> collapsedOperands =
        collapseOperands(rewriter, operands, operandUnitDims);
    Value collapsedLhs = collapsedOperands[0];
    Value collapsedRhs = collapsedOperands[1];
    Value collapsedInit = collapsedOperands[2];
    SmallVector<Type, 1> collapsedResultTy;
    if (isa<RankedTensorType>(collapsedInit.getType()))
      collapsedResultTy.push_back(collapsedInit.getType());
    auto collapsedOp = ToOpTy::create(rewriter, loc, collapsedResultTy,
                                      ValueRange{collapsedLhs, collapsedRhs},
                                      ValueRange{collapsedInit});
    for (auto attr : contractionOp->getAttrs()) {
      if (attr.getName() == LinalgDialect::kMemoizedIndexingMapsAttrName ||
          attr.getName() == "indexing_maps")
        continue;
      collapsedOp->setAttr(attr.getName(), attr.getValue());
    }

```
- **EN**: Implements logic around `collapseOperands`, `getType`, `push_back`, `create`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `collapseOperands`, `getType`, `push_back`, `create`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 996-1010
```cpp
    auto results = contractionOp.getResults();
    assert(results.size() < 2 && "expected at most one result");
    if (results.empty()) {
      rewriter.replaceOp(contractionOp, collapsedOp);
    } else {
      rewriter.replaceOp(
          contractionOp,
          expandResult(rewriter, collapsedOp.getResultTensors()[0],
                       cast<RankedTensorType>(results[0].getType()),
                       operandUnitDims[2]));
    }

    return success();
  }

```
- **EN**: Implements logic around `getResults`, `assert`, `empty`, `replaceOp`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getResults`, `assert`, `empty`, `replaceOp`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1011-1024
```cpp
  /// Populate `operandUnitDims` with 3 indices indicating the unit dim
  /// for each operand that should be collapsed in this pattern.  If an
  /// operand shouldn't be collapsed, the index should be negative.
  virtual LogicalResult
  getOperandUnitDims(LinalgOp op,
                     SmallVectorImpl<int64_t> &operandUnitDims) const = 0;
};

/// Patterns for unbatching batched contraction ops
template <typename FromOpTy, typename ToOpTy>
struct RankReduceToUnBatched : RankReduceContractionOps<FromOpTy, ToOpTy> {
  using RankReduceContractionOps<FromOpTy, ToOpTy>::RankReduceContractionOps;

  /// Look for unit batch dims to collapse.
```
- **EN**: Introduces declarations for `RankReduceToUnBatched`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `RankReduceToUnBatched` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1025-1048
```cpp
  LogicalResult
  getOperandUnitDims(LinalgOp op,
                     SmallVectorImpl<int64_t> &operandUnitDims) const override {
    FailureOr<ContractionDimensions> maybeContractionDims =
        inferContractionDims(op);
    if (failed(maybeContractionDims)) {
      LLVM_DEBUG(llvm::dbgs() << "could not infer contraction dims");
      return failure();
    }
    const ContractionDimensions &contractionDims = maybeContractionDims.value();

    if (contractionDims.batch.size() != 1)
      return failure();
    auto batchDim = contractionDims.batch[0];
    SmallVector<std::pair<Value, unsigned>, 3> bOperands;
    op.mapIterationSpaceDimToAllOperandDims(batchDim, bOperands);
    if (bOperands.size() != 3 || llvm::any_of(bOperands, [](auto pair) {
          return cast<ShapedType>(std::get<0>(pair).getType())
                     .getShape()[std::get<1>(pair)] != 1;
        })) {
      LLVM_DEBUG(llvm::dbgs() << "specified unit dims not found");
      return failure();
    }

```
- **EN**: Implements logic around `getOperandUnitDims`, `inferContractionDims`, `failed`, `dbgs`, and 6 more symbols.
- **CN**: 围绕 `getOperandUnitDims`, `inferContractionDims`, `failed`, `dbgs`, and 6 more symbols 实现具体逻辑。

### Lines 1049-1068
```cpp
    operandUnitDims = SmallVector<int64_t>{std::get<1>(bOperands[0]),
                                           std::get<1>(bOperands[1]),
                                           std::get<1>(bOperands[2])};
    return success();
  }
};

/// Patterns for reducing non-batch dimensions
template <typename FromOpTy, typename ToOpTy>
struct RankReduceMatmul : RankReduceContractionOps<FromOpTy, ToOpTy> {
  using RankReduceContractionOps<FromOpTy, ToOpTy>::RankReduceContractionOps;

  /// Helper for determining whether the lhs/init or rhs/init are reduced.
  static bool constexpr reduceLeft =
      (std::is_same_v<FromOpTy, BatchMatmulOp> &&
       std::is_same_v<ToOpTy, BatchVecmatOp>) ||
      (std::is_same_v<FromOpTy, MatmulOp> &&
       std::is_same_v<ToOpTy, VecmatOp>) ||
      (std::is_same_v<FromOpTy, MatvecOp> && std::is_same_v<ToOpTy, DotOp>);

```
- **EN**: Introduces declarations for `RankReduceMatmul`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `RankReduceMatmul` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1069-1096
```cpp
  /// Look for non-batch spatial dims to collapse.
  LogicalResult
  getOperandUnitDims(LinalgOp op,
                     SmallVectorImpl<int64_t> &operandUnitDims) const override {
    FailureOr<ContractionDimensions> maybeContractionDims =
        inferContractionDims(op);
    if (failed(maybeContractionDims)) {
      LLVM_DEBUG(llvm::dbgs() << "could not infer contraction dims");
      return failure();
    }
    const ContractionDimensions &contractionDims = maybeContractionDims.value();

    if constexpr (reduceLeft) {
      auto m = contractionDims.m[0];
      SmallVector<std::pair<Value, unsigned>, 2> mOperands;
      op.mapIterationSpaceDimToAllOperandDims(m, mOperands);
      if (mOperands.size() != 2)
        return failure();
      if (llvm::all_of(mOperands, [](auto pair) {
            return cast<ShapedType>(std::get<0>(pair).getType())
                       .getShape()[std::get<1>(pair)] == 1;
          })) {
        operandUnitDims = SmallVector<int64_t>{std::get<1>(mOperands[0]), -1,
                                               std::get<1>(mOperands[1])};
        return success();
      }
    } else {
      auto n = contractionDims.n[0];
```
- **EN**: Implements logic around `getOperandUnitDims`, `inferContractionDims`, `failed`, `dbgs`, and 9 more symbols.
- **CN**: 围绕 `getOperandUnitDims`, `inferContractionDims`, `failed`, `dbgs`, and 9 more symbols 实现具体逻辑。

### Lines 1097-1114
```cpp
      SmallVector<std::pair<Value, unsigned>, 2> nOperands;
      op.mapIterationSpaceDimToAllOperandDims(n, nOperands);
      if (nOperands.size() != 2)
        return failure();
      if (llvm::all_of(nOperands, [](auto pair) {
            return cast<ShapedType>(std::get<0>(pair).getType())
                       .getShape()[std::get<1>(pair)] == 1;
          })) {
        operandUnitDims = SmallVector<int64_t>{-1, std::get<1>(nOperands[0]),
                                               std::get<1>(nOperands[1])};
        return success();
      }
    }
    LLVM_DEBUG(llvm::dbgs() << "specified unit dims not found");
    return failure();
  }
};

```
- **EN**: Implements logic around `mapIterationSpaceDimToAllOperandDims`, `size`, `failure`, `all_of`, and 4 more symbols.
- **CN**: 围绕 `mapIterationSpaceDimToAllOperandDims`, `size`, `failure`, `all_of`, and 4 more symbols 实现具体逻辑。

### Lines 1115-1131
```cpp
} // namespace

void mlir::linalg::populateContractionOpRankReducingPatterns(
    RewritePatternSet &patterns) {
  MLIRContext *context = patterns.getContext();
  // Unbatching patterns for unit batch size
  patterns.add<RankReduceToUnBatched<BatchMatmulOp, MatmulOp>>(context);
  patterns.add<RankReduceToUnBatched<BatchMatvecOp, MatvecOp>>(context);
  patterns.add<RankReduceToUnBatched<BatchVecmatOp, VecmatOp>>(context);

  // Non-batch rank 1 reducing patterns
  patterns.add<RankReduceMatmul<MatmulOp, VecmatOp>>(context);
  patterns.add<RankReduceMatmul<MatmulOp, MatvecOp>>(context);
  // Batch rank 1 reducing patterns
  patterns.add<RankReduceMatmul<BatchMatmulOp, BatchVecmatOp>>(context);
  patterns.add<RankReduceMatmul<BatchMatmulOp, BatchMatvecOp>>(context);

```
- **EN**: Implements logic around `populateContractionOpRankReducingPatterns`, `getContext`, `MatmulOp>>`, `MatvecOp>>`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateContractionOpRankReducingPatterns`, `getContext`, `MatmulOp>>`, `MatvecOp>>`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1132-1135
```cpp
  // Non-batch rank 0 reducing patterns
  patterns.add<RankReduceMatmul<MatvecOp, DotOp>>(context);
  patterns.add<RankReduceMatmul<VecmatOp, DotOp>>(context);
}
```
- **EN**: Implements logic around `DotOp>>`.
- **CN**: 围绕 `DotOp>>` 实现具体逻辑。

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
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/MemRef/Transforms/Transforms.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/Tensor/Transforms/Transforms.h`, `mlir/Dialect/Utils/ReshapeOpsUtils.h` ... (+8 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (11), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (3), LLVM support-library facilities / LLVM Support 库设施 (1)
