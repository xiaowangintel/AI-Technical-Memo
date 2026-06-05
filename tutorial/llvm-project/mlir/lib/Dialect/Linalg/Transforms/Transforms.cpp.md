# Transforms.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/Transforms.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements logic and helpers to expose Linalg transforms as rewrite patterns.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36
```cpp
//===- Transforms.cpp - Linalg transformations as patterns ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements logic and helpers to expose Linalg transforms as rewrite
// patterns.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/SCF/Transforms/Transforms.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Tensor/IR/TensorTilingInterfaceImpl.h"
#include "mlir/Dialect/Tensor/Utils/Utils.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/Dialect/Utils/StructuredOpsUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/BuiltinTypeInterfaces.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/InterleavedRange.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`。

### Lines 37-54
```cpp
#include "llvm/Support/raw_ostream.h"
#include <type_traits>
#include <utility>

#define DEBUG_TYPE "linalg-transforms"

using namespace mlir;
using namespace mlir::linalg;

//===----------------------------------------------------------------------===//
// Transformations exposed as functional-style API calls.
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// peelLoop transformation.
//===----------------------------------------------------------------------===//

/// Try to peel and canonicalize loop `op` and return the new result.
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/raw_ostream.h`, `type_traits`, `utility`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/raw_ostream.h`, `type_traits`, `utility`。

### Lines 55-72
```cpp
/// Also applies affine_min/max bounds simplification on the fly where relevant.
// TODO: Add support for scf.parallel and affine.for loops.
SmallVector<Value> mlir::linalg::peelLoop(RewriterBase &rewriter,
                                          Operation *op) {
  return llvm::TypeSwitch<Operation *, SmallVector<Value, 4>>(op)
      .Case([&](scf::ForOp forOp) {
        scf::ForOp partialIteration;
        if (succeeded(scf::peelForLoopAndSimplifyBounds(rewriter, forOp,
                                                        partialIteration)))
          return partialIteration->getResults();
        assert(!partialIteration && "expected that loop was not peeled");
        return forOp->getResults();
      })
      .Default([&](Operation *op) { return op->getResults(); });
}

/// Peel 'loops' and applies affine_min/max bounds simplification on the fly
/// where relevant.
```
- **EN**: Implements logic around `peelLoop`, `Case`, `succeeded`, `getResults`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `peelLoop`, `Case`, `succeeded`, `getResults`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 73-97
```cpp
void mlir::linalg::peelLoops(RewriterBase &rewriter,
                             ArrayRef<scf::ForOp> loops) {
  for (auto loopOp : loops)
    peelLoop(rewriter, loopOp);
}

//===----------------------------------------------------------------------===//
// pack transformation.
//===----------------------------------------------------------------------===//

#ifndef NDEBUG
/// Return true if `map` has 0 or 1 result function of AffineDimExpr(dim).
static bool hasAtMostOneResultFunctionOfDim(AffineMap map, int64_t dim) {
  bool found = false;
  for (AffineExpr e : map.getResults()) {
    if (!e.isFunctionOfDim(dim))
      continue;
    if (found)
      return false;
    found = true;
  }
  return true;
}
#endif // NDEBUG

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 98-115
```cpp
static std::string stringifyReassocIndices(ReassociationIndicesRef ri) {
  return llvm::interleaved(ri, ", ", /*Prefix=*/"|", /*Suffix=*/"");
}

/// Return the index of the first result of `map` that is a function of
/// AffineDimExpr(dim), std::nullopt otherwise.
static std::optional<int64_t> getFirstResultIndexFunctionOf(AffineMap map,
                                                            int64_t dim) {
  for (int64_t i = 0, e = map.getNumResults(); i < e; ++i) {
    AffineExpr expr = map.getResult(i);
    if (!expr.isFunctionOfDim(dim))
      continue;
    return i;
  }
  return std::nullopt;
}

/// Perform one step of packing of a LinalgOp's metadata along `dim` into the
```
- **EN**: Implements logic around `stringifyReassocIndices`, `interleaved`, `getFirstResultIndexFunctionOf`, `getNumResults`, and 2 more symbols.
- **CN**: 围绕 `stringifyReassocIndices`, `interleaved`, `getFirstResultIndexFunctionOf`, `getNumResults`, and 2 more symbols 实现具体逻辑。

### Lines 116-133
```cpp
/// `newDim` at `iteratorTypes.size()` by:
///   1. Appending `iteratorTypes[newDim]`, equal to `iteratorTypes[dim]`.
///   2. Appending a `newDim` to the domain of every indexing map.
///   3. For each operand (i.e. for each map in `indexingMaps`), perform packing
///      by potentially adding a `newDim` result to `map`.
/// The preserved invariant is that `iteratorTypes.size()` is always equal to
/// `map.getNumDims()` for every map in `indexingMaps`.
///
/// Update `indexingMaps` and `iteratorTypes` inplace as one step of the update.
/// Return a vector that records the optional packing for each operand.
/// Return failure if the packed indexing cannot be represented with a LinalgOp.
///
/// Further details:
/// ================
/// The current implementation of packing (i.e. data tiling) consists of
/// rewriting a linearized strip-mined form into a higher-dimensional access.
/// e.g. consider an access `A[I][f(j, k, l)]` and packing by 4; we rewrite
/// `I` into `4 * i + ii`, where `0 <= ii < 4`.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 134-156
```cpp
/// The access is further rewritten as `A[i][f(j, k, l)][ii]`.
///
/// This rewrite into higher dimensional access is not possible for general
/// AffineExpr in Linalg atm, it is restricted to an AffineDimExpr:
/// e.g. consider an access `A[I + J][f(j, k, l)]` and packing by 4; we
/// rewrite `I + J` into `4 * i + ii + J`, where `0 <= ii < 4`.
/// The rewrite of the access would be a form not representable in Linalg:
///   `A[i + (ii + J) / 4][f(j, k, l)][(ii + J) % 4]`.
/// Note however that as `J` and `ii` iterate, the accesses do not have a
/// particular alignment, so packing does not achieve alignment in this case
///
/// In the future, we may want to consider a mixed-form that allows some
/// alignment in the presence of multiple accesses:
///   `A[I][f(j, k, l)]` and `B[I + J][f(j, k, l)]`
/// And would rewrite accesses as:
///   `A[i][f(j, k, l)][ii]` and `B[4 * i + ii + J][f(j, k, l)]`
static FailureOr<SmallVector<std::optional<int64_t>>>
packLinalgMetadataOnce(SmallVectorImpl<AffineMap> &indexingMaps,
                       SmallVectorImpl<utils::IteratorType> &iteratorTypes,
                       int64_t dim) {
  int64_t newDim = iteratorTypes.size();
  iteratorTypes.push_back(iteratorTypes[dim]);

```
- **EN**: Implements logic around `packLinalgMetadataOnce`, `size`, `push_back`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `packLinalgMetadataOnce`, `size`, `push_back` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 157-179
```cpp
  SmallVector<std::optional<int64_t>> packedDimPerIndexingMap(
      indexingMaps.size(), std::nullopt);
  SmallVector<AffineMap> newMaps;
  for (int64_t operandIdx = 0, e = indexingMaps.size(); operandIdx < e;
       ++operandIdx) {
    AffineMap map = indexingMaps[operandIdx];

    // Add the `newDim` to map whatever the case.
    assert(map.getNumDims() == newDim && "num dims invariant violation");
    map = map.shiftDims(1, newDim);

    // Get the at-most-1 index of the result that is a function of `dim`.
    // If we can find one, we insert `AffineDimExpr(newDim)` to the map, which
    // logically chunks dimension `dim` into `K * dim + newDim`, where the
    // packing factor `K` is specified separately.
    assert(hasAtMostOneResultFunctionOfDim(map, dim) &&
           "num results invariant violation");
    auto maybeOperandDimensionToPack = getFirstResultIndexFunctionOf(map, dim);
    if (!maybeOperandDimensionToPack.has_value()) {
      newMaps.push_back(map);
      continue;
    }

```
- **EN**: Implements logic around `packedDimPerIndexingMap`, `size`, `assert`, `shiftDims`, and 3 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `packedDimPerIndexingMap`, `size`, `assert`, `shiftDims`, and 3 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 180-197
```cpp
    // We can only pack AffineDimExpr atm.
    if (!isa<AffineDimExpr>(map.getResult(maybeOperandDimensionToPack.value())))
      return failure();

    // Add `newDim` to the results of the map.
    map = map.insertResult(Builder(map.getContext()).getAffineDimExpr(newDim),
                           map.getNumResults());
    newMaps.push_back(map);

    // Record the that `operandIdx` is packed.
    packedDimPerIndexingMap[operandIdx] = maybeOperandDimensionToPack;
  }
  indexingMaps = newMaps;

  return packedDimPerIndexingMap;
}

namespace {
```
- **EN**: Implements logic around `getResult`, `failure`, `insertResult`, `getNumResults`, and 1 more symbols.
- **CN**: 围绕 `getResult`, `failure`, `insertResult`, `getNumResults`, and 1 more symbols 实现具体逻辑。

### Lines 198-215
```cpp

/// Helper struct to encode packing along one dimension of a LinalgOp.
struct PackedOperandsDim {
  OpFoldResult packedSize;
  SmallVector<std::optional<int64_t>> packedDimForEachOperand;
};

/// Helper struct to encode packing along all dimensions of a LinalgOp.
struct PackedOperandsDimList {
  void pushBack(PackedOperandsDim &&packedOperandsDims) {
    spec.emplace_back(packedOperandsDims);
  }
  /// Return all the dims that have been packed for operand @ `operandPos`.
  SmallVector<int64_t> extractPackedDimsForOperand(int64_t operandPos);
  /// Return all the pack sizes by which an operand @ `operandPos` is packed.
  SmallVector<OpFoldResult> extractPackSizesForOperand(int64_t operandPos);

private:
```
- **EN**: Introduces declarations for `to`, `PackedOperandsDim`, `PackedOperandsDimList`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `to`, `PackedOperandsDim`, `PackedOperandsDimList` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 216-236
```cpp
  SmallVector<PackedOperandsDim> spec;
};

} // namespace

FailureOr<LowerPackResult> linalg::lowerPack(RewriterBase &rewriter,
                                             linalg::PackOp packOp,
                                             bool lowerPadLikeWithInsertSlice) {
  // TODO: Support Memref PackOp. Temporarily return failure.
  if (!packOp.hasPureTensorSemantics())
    return failure();

  // 1. Filter out NYI cases.
  auto packedTensorType =
      cast<RankedTensorType>(packOp->getResultTypes().front());
  if (llvm::any_of(packOp.getStaticInnerTiles(), ShapedType::isDynamic)) {
    return rewriter.notifyMatchFailure(
        packOp,
        "non-static shape NYI, needs a more powerful tensor.expand_shape op");
  }

```
- **EN**: Implements logic around `lowerPack`, `hasPureTensorSemantics`, `failure`, `getResultTypes`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `lowerPack`, `hasPureTensorSemantics`, `failure`, `getResultTypes`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 237-272
```cpp
  Location loc = packOp->getLoc();
  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPoint(packOp);

  // 2. Compute the permutation vector to shuffle packed shape into the shape
  // before any outer or inner permutations have been applied.
  PackingMetadata packingMetadata;
  SmallVector<int64_t> packedToStripMinedShapePerm =
      getPackInverseDestPerm(packOp, packingMetadata);

  // 3. Compute the stripMinedShape: this is the packed shape before any outer
  // or inner permutations have been applied.
  SmallVector<int64_t> stripMinedShape(packedTensorType.getShape());
  applyPermutationToVector(stripMinedShape, packedToStripMinedShapePerm);

  // 4. Pad the source of packOp to a shape we can expand into stripMinedShape.
  SmallVector<OpFoldResult> lows(packOp.getSourceRank(),
                                 rewriter.getIndexAttr(0));
  SmallVector<OpFoldResult> highs(packOp.getSourceRank(),
                                  rewriter.getIndexAttr(0));
  for (auto [pos, innerSize] :
       llvm::zip_equal(packOp.getInnerDimsPos(), packOp.getMixedTiles())) {
    int outerPos =
        packedToStripMinedShapePerm[packingMetadata.outerPositions[pos]];
    OpFoldResult origSize =
        tensor::getMixedSize(rewriter, loc, packOp.getSource(), pos);
    OpFoldResult outerSize =
        tensor::getMixedSize(rewriter, loc, packOp.getDest(), outerPos);
    AffineExpr s0, d0, d1;
    bindDims(rewriter.getContext(), d0, d1);
    bindSymbols(rewriter.getContext(), s0);
    auto map = AffineMap::get(/*dimCount=*/2, /*symbolCount=*/1, d0 * s0 - d1);
    highs[pos] = affine::makeComposedFoldedAffineApply(
        rewriter, loc, map, {outerSize, origSize, innerSize});
  }
  RankedTensorType collapsed = tensor::CollapseShapeOp::inferCollapsedType(
```
- **EN**: Implements logic around `getLoc`, `g`, `setInsertionPoint`, `getPackInverseDestPerm`, and 12 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `g`, `setInsertionPoint`, `getPackInverseDestPerm`, and 12 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 273-296
```cpp
      RankedTensorType::Builder(packedTensorType).setShape(stripMinedShape),
      packingMetadata.reassociations);
  Value paddingValue = packOp.getPaddingValue();
  if (!paddingValue) {
    paddingValue = arith::ConstantOp::create(
        rewriter, loc, rewriter.getZeroAttr(getElementTypeOrSelf(collapsed)));
  }
  auto padOp =
      tensor::PadOp::create(rewriter, loc, collapsed, packOp.getSource(), lows,
                            highs, paddingValue, /*nofold=*/false);

  LDBG() << "insertPositions: "
         << llvm::interleaved(packingMetadata.insertPositions);
  LDBG() << "outerPositions: "
         << llvm::interleaved(packingMetadata.outerPositions);
  LDBG() << "packedShape: " << llvm::interleaved(packedTensorType.getShape());
  LDBG() << "packedToStripMinedShapePerm: "
         << llvm::interleaved(packedToStripMinedShapePerm);
  LDBG() << "reassociations: "
         << llvm::interleaved(llvm::map_range(packingMetadata.reassociations,
                                              stringifyReassocIndices));
  LDBG() << "stripMinedShape: " << llvm::interleaved(stripMinedShape);
  LDBG() << "collapsed type: " << collapsed;

```
- **EN**: Implements logic around `Builder`, `getPaddingValue`, `create`, `getZeroAttr`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Builder`, `getPaddingValue`, `create`, `getZeroAttr`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 297-315
```cpp
  if (lowerPadLikeWithInsertSlice && packOp.isLikePad()) {
    // Pack ops which operate as simple pads may not produce legal
    // tensor.insert_slice operations when the packed type does not rank reduce
    // to the padded type.
    SliceVerificationResult rankReduces =
        isRankReducedType(packedTensorType, padOp.getResultType());

    if (rankReduces == SliceVerificationResult::Success) {
      // This pack is just a plain pad.
      // Just insert the pad in the higher ranked tensor.
      // Offsets.
      SmallVector<OpFoldResult> zeros(packOp.getDestRank(),
                                      rewriter.getIndexAttr(0));
      // Strides.
      SmallVector<OpFoldResult> ones(packOp.getDestRank(),
                                     rewriter.getIndexAttr(1));
      SmallVector<OpFoldResult> sizes =
          tensor::getMixedSizes(rewriter, loc, packOp.getDest());

```
- **EN**: Implements logic around `isLikePad`, `isRankReducedType`, `zeros`, `getIndexAttr`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isLikePad`, `isRankReducedType`, `zeros`, `getIndexAttr`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 316-335
```cpp
      auto insertSliceOp = tensor::InsertSliceOp::create(
          rewriter, loc, /*source=*/padOp, /*dest=*/packOp.getDest(),
          /*offsets=*/zeros, sizes, /*strides=*/ones);

      LDBG() << "insert_slice op: " << insertSliceOp;

      rewriter.replaceOp(packOp, insertSliceOp->getResults());

      return LowerPackResult{padOp, /*reshapeOp=*/nullptr,
                             /*transposeOp=*/nullptr};
    }
  }

  // 5. Expand from the padded result to the stripMinedShape.
  auto expandShapeResultType =
      RankedTensorType::Builder(packedTensorType).setShape(stripMinedShape);
  auto reshapeOp = tensor::ExpandShapeOp::create(
      rewriter, loc, expandShapeResultType, padOp.getResult(),
      packingMetadata.reassociations);

```
- **EN**: Implements logic around `create`, `getDest`, `LDBG`, `replaceOp`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getDest`, `LDBG`, `replaceOp`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 336-358
```cpp
  // 6. Transpose stripMinedShape to packedShape.
  SmallVector<int64_t> transpPerm =
      invertPermutationVector(packedToStripMinedShapePerm);
  auto transposeOp = linalg::TransposeOp::create(
      rewriter, loc, reshapeOp.getResult(), packOp.getDest(), transpPerm);

  LDBG() << "reshape op: " << reshapeOp;
  LDBG() << "transpPerm: " << llvm::interleaved(transpPerm);
  LDBG() << "transpose op: " << transposeOp;

  // 7. Replace packOp by transposeOp.
  rewriter.replaceOp(packOp, transposeOp->getResults());

  return LowerPackResult{padOp, reshapeOp, transposeOp};
}

FailureOr<LowerUnPackOpResult>
linalg::lowerUnPack(RewriterBase &rewriter, linalg::UnPackOp unPackOp,
                    bool lowerUnpadLikeWithExtractSlice) {
  // TODO: Support Memref UnPackOp. Temporarily return failure.
  if (!unPackOp.hasPureTensorSemantics())
    return failure();

```
- **EN**: Implements logic around `invertPermutationVector`, `create`, `getResult`, `LDBG`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `invertPermutationVector`, `create`, `getResult`, `LDBG`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 359-376
```cpp
  Location loc = unPackOp->getLoc();
  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPoint(unPackOp);

  auto packedTensorType = cast<RankedTensorType>(unPackOp.getSourceType());
  int64_t packedRank = packedTensorType.getRank();

  OpFoldResult zero = rewriter.getIndexAttr(0), one = rewriter.getIndexAttr(1);
  auto destTensorType = cast<RankedTensorType>(unPackOp.getDest().getType());
  if (lowerUnpadLikeWithExtractSlice && unPackOp.isLikeUnPad()) {
    // This unpack is just a plain unpad.
    // Just extract the slice from the higher ranked tensor.
    ArrayRef<int64_t> destShape = destTensorType.getShape();
    // The inner dimensions stay the same as the destination tensor, but the
    // outer ones are additional 1s.
    SmallVector<OpFoldResult> sizes(packedRank - destShape.size(), one);
    sizes.append(tensor::getMixedSizes(rewriter, loc, unPackOp.getDest()));

```
- **EN**: Implements logic around `getLoc`, `g`, `setInsertionPoint`, `getSourceType`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `g`, `setInsertionPoint`, `getSourceType`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 377-394
```cpp
    auto extractSliceOp = tensor::ExtractSliceOp::create(
        rewriter, loc, destTensorType, unPackOp.getSource(),
        SmallVector<OpFoldResult>(packedRank, zero), sizes,
        SmallVector<OpFoldResult>(packedRank, one));

    rewriter.replaceOp(unPackOp, extractSliceOp->getResults());

    return LowerUnPackOpResult{/*emptyOp=*/nullptr, /*transposeOp=*/nullptr,
                               /*reshapeOp=*/nullptr, extractSliceOp,
                               /*copyOp=*/nullptr};
  }

  // 1. Compute the permutation vector to shuffle packed shape into the shape
  // before any outer or inner permutations have been applied.
  PackingMetadata packingMetadata;
  SmallVector<int64_t> packedToStripMinedShapePerm =
      getUnPackInverseSrcPerm(unPackOp, packingMetadata);

```
- **EN**: Implements logic around `create`, `getSource`, `SmallVector`, `replaceOp`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getSource`, `SmallVector`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 395-416
```cpp
  // 2. Compute the stripMinedShape: this is the packed shape without outer and
  // inner permutations.
  SmallVector<int64_t> stripMinedShape(packedTensorType.getShape());
  applyPermutationToVector(stripMinedShape, packedToStripMinedShapePerm);

  // 3. Transpose packedShape to stripMinedShape.
  RankedTensorType stripMinedTensorType =
      RankedTensorType::Builder(packedTensorType).setShape(stripMinedShape);
  RankedTensorType collapsedType = tensor::CollapseShapeOp::inferCollapsedType(
      stripMinedTensorType, packingMetadata.reassociations);

  // Get dynamic dims from input tensor based on packedToStripMinedShapePerm
  // permutation.
  SmallVector<OpFoldResult, 4> dims =
      tensor::getMixedSizes(rewriter, loc, unPackOp.getSource());
  applyPermutationToVector(dims, packedToStripMinedShapePerm);
  auto emptyOp = tensor::EmptyOp::create(rewriter, loc, dims,
                                         stripMinedTensorType.getElementType());
  auto transposeOp =
      linalg::TransposeOp::create(rewriter, loc, unPackOp.getSource(), emptyOp,
                                  packedToStripMinedShapePerm);

```
- **EN**: Implements logic around `stripMinedShape`, `applyPermutationToVector`, `Builder`, `inferCollapsedType`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `stripMinedShape`, `applyPermutationToVector`, `Builder`, `inferCollapsedType`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 417-440
```cpp
  LDBG() << "insertPositions: "
         << llvm::interleaved(packingMetadata.insertPositions);
  LDBG() << "packedShape: " << llvm::interleaved(packedTensorType.getShape());
  LDBG() << "packedToStripMinedShapePerm: "
         << llvm::interleaved(packedToStripMinedShapePerm);
  LDBG() << "reassociations: "
         << llvm::interleaved(llvm::map_range(packingMetadata.reassociations,
                                              stringifyReassocIndices));
  LDBG() << "stripMinedShape: " << llvm::interleaved(stripMinedShape);
  LDBG() << "collapsed type: " << collapsedType;

  // 4. Collapse from the stripMinedShape to the padded result.
  auto reshapeOp = tensor::CollapseShapeOp::create(
      rewriter, loc, collapsedType, transposeOp->getResult(0),
      packingMetadata.reassociations);

  // 5. ExtractSlice.
  int64_t destRank = destTensorType.getRank();
  auto extractSliceOp = tensor::ExtractSliceOp::create(
      rewriter, loc, destTensorType, reshapeOp->getResult(0),
      SmallVector<OpFoldResult>(destRank, zero),
      tensor::getMixedSizes(rewriter, loc, unPackOp.getDest()),
      SmallVector<OpFoldResult>(destRank, one));

```
- **EN**: Implements logic around `LDBG`, `interleaved`, `create`, `getResult`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LDBG`, `interleaved`, `create`, `getResult`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 441-462
```cpp
  // 6. Inject a copy to preserve DPS.
  auto copyOp = linalg::CopyOp::create(
      rewriter, loc, extractSliceOp->getResult(0), unPackOp.getDest());

  // 7. Replace unPackOp by copyOp.
  rewriter.replaceOp(unPackOp, copyOp->getResults());

  return LowerUnPackOpResult{emptyOp, transposeOp, reshapeOp, extractSliceOp,
                             copyOp};
}

SmallVector<int64_t>
PackedOperandsDimList::extractPackedDimsForOperand(int64_t operandPos) {
  SmallVector<int64_t> res;
  for (auto &i : spec) {
    if (!i.packedDimForEachOperand[operandPos].has_value())
      continue;
    res.push_back(i.packedDimForEachOperand[operandPos].value());
  }
  return res;
}

```
- **EN**: Implements logic around `create`, `getResult`, `replaceOp`, `extractPackedDimsForOperand`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResult`, `replaceOp`, `extractPackedDimsForOperand`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 463-484
```cpp
SmallVector<OpFoldResult>
PackedOperandsDimList::extractPackSizesForOperand(int64_t operandPos) {
  SmallVector<OpFoldResult> res;
  for (auto &i : spec) {
    if (!i.packedDimForEachOperand[operandPos].has_value())
      continue;
    res.push_back(i.packedSize);
  }
  return res;
}

/// Implement packing of a single LinalgOp by performing packing by
/// `packedSizes`. There must be one packedSizes entry per `linalgOp` iterator.
/// Return the packed Linalg op on success, failure otherwise.
FailureOr<PackResult> linalg::pack(RewriterBase &rewriter,
                                   linalg::LinalgOp linalgOp,
                                   ArrayRef<OpFoldResult> packedSizes) {
  if (packedSizes.size() != linalgOp.getNumLoops()) {
    return rewriter.notifyMatchFailure(linalgOp,
                                       "incorrect number of pack sizes");
  }

```
- **EN**: Implements logic around `extractPackSizesForOperand`, `has_value`, `push_back`, `pack`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `extractPackSizesForOperand`, `has_value`, `push_back`, `pack`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 485-502
```cpp
  Location loc = linalgOp->getLoc();
  SmallVector<AffineMap> indexingMaps = linalgOp.getIndexingMapsArray();
  SmallVector<utils::IteratorType> iteratorTypes =
      linalgOp.getIteratorTypesArray();
  LDBG() << "Start packing: " << linalgOp;
  LDBG() << "maps: " << llvm::interleaved(indexingMaps);
  LDBG() << "iterators: " << llvm::interleaved(iteratorTypes);

  SmallVector<linalg::PackOp> packOps;
  SmallVector<linalg::UnPackOp> unPackOps;
  // Step 1. Pack each dim of the LinalgOp metadata by packedSizes[i].
  PackedOperandsDimList listOfPackedOperandsDim;
  for (int64_t i = 0, e = packedSizes.size(); i < e; ++i) {
    std::optional<int64_t> maybeConstant = getConstantIntValue(packedSizes[i]);
    // Skip tile sizes explicitly set to 0.
    if (maybeConstant.has_value() && maybeConstant.value() == 0)
      continue;

```
- **EN**: Implements logic around `getLoc`, `getIndexingMapsArray`, `getIteratorTypesArray`, `LDBG`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `getIndexingMapsArray`, `getIteratorTypesArray`, `LDBG`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 503-520
```cpp
    PackedOperandsDim packedOperandsDims;
    packedOperandsDims.packedSize = packedSizes[i];
    FailureOr<SmallVector<std::optional<int64_t>>>
        maybePackedDimForEachOperand =
            packLinalgMetadataOnce(indexingMaps, iteratorTypes, i);
    if (failed(maybePackedDimForEachOperand))
      return failure();
    packedOperandsDims.packedDimForEachOperand = *maybePackedDimForEachOperand;

    LDBG() << "++++ After pack size #" << i << ": " << packedSizes[i];
    LDBG() << "maps: " << llvm::interleaved(indexingMaps);
    LDBG() << "iterators: " << llvm::interleaved(iteratorTypes);
    LDBG() << "packedDimForEachOperand: "
           << llvm::interleaved(packedOperandsDims.packedDimForEachOperand);

    listOfPackedOperandsDim.pushBack(std::move(packedOperandsDims));
  }

```
- **EN**: Implements logic around `packLinalgMetadataOnce`, `failed`, `failure`, `LDBG`, and 2 more symbols.
- **CN**: 围绕 `packLinalgMetadataOnce`, `failed`, `failure`, `LDBG`, and 2 more symbols 实现具体逻辑。

### Lines 521-556
```cpp
  // Step 2. Propagate packing to all LinalgOp operands.
  SmallVector<Value> inputsAndInits, results;
  SmallVector<OpOperand *> initOperands =
      llvm::to_vector(llvm::make_pointer_range(linalgOp.getDpsInitsMutable()));
  SmallVector<OpOperand *> inputOperands = linalgOp.getDpsInputOperands();
  for (const auto &operandsList : {inputOperands, initOperands}) {
    for (OpOperand *opOperand : operandsList) {
      int64_t pos = opOperand->getOperandNumber();
      Value operand = opOperand->get();
      SmallVector<int64_t> innerPos =
          listOfPackedOperandsDim.extractPackedDimsForOperand(pos);
      SmallVector<OpFoldResult> innerPackSizes =
          listOfPackedOperandsDim.extractPackSizesForOperand(pos);
      LDBG() << "operand: " << operand;
      LDBG() << "innerPos: " << llvm::interleaved(innerPos);
      LDBG() << "innerPackSizes: " << llvm::interleaved(innerPackSizes);
      if (innerPackSizes.empty()) {
        inputsAndInits.push_back(operand);
        continue;
      }
      Value dest = linalg::PackOp::createDestinationTensor(
          rewriter, loc, operand, innerPackSizes, innerPos,
          /*outerDimsPerm=*/{});
      ShapedType operandType = cast<ShapedType>(operand.getType());
      bool areConstantTiles =
          llvm::all_of(innerPackSizes, [](OpFoldResult tile) {
            return getConstantIntValue(tile).has_value();
          });
      if (areConstantTiles && operandType.hasStaticShape() &&
          !linalg::PackOp::requirePaddingValue(
              operandType.getShape(), innerPos,
              cast<ShapedType>(dest.getType()).getShape(), {},
              innerPackSizes)) {
        packOps.push_back(linalg::PackOp::create(rewriter, loc, operand, dest,
                                                 innerPos, innerPackSizes));
      } else {
```
- **EN**: Implements logic around `to_vector`, `getDpsInputOperands`, `getOperandNumber`, `get`, and 12 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `to_vector`, `getDpsInputOperands`, `getOperandNumber`, `get`, and 12 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 557-578
```cpp
        // TODO: value of the padding attribute should be determined by
        // consumers.
        auto zeroAttr =
            rewriter.getZeroAttr(getElementTypeOrSelf(dest.getType()));
        Value zero = arith::ConstantOp::create(rewriter, loc, zeroAttr);
        packOps.push_back(linalg::PackOp::create(
            rewriter, loc, operand, dest, innerPos, innerPackSizes, zero));
      }
      inputsAndInits.push_back(packOps.back().getResult());
    }
  }

  // Step 3. Build the packed op, use the type of `inits` as result types.
  ValueRange inputs =
      ValueRange{inputsAndInits}.take_front(linalgOp.getNumDpsInputs());
  ValueRange inits =
      ValueRange{inputsAndInits}.take_back(linalgOp.getNumDpsInits());
  auto packedLinalgOp =
      linalg::GenericOp::create(rewriter, linalgOp.getLoc(), inits.getTypes(),
                                inputs, inits, indexingMaps, iteratorTypes);
  packedLinalgOp.getRegion().takeBody(linalgOp->getRegion(0));

```
- **EN**: Implements logic around `getZeroAttr`, `create`, `push_back`, `take_front`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getZeroAttr`, `create`, `push_back`, `take_front`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 579-597
```cpp
  // Step 4. Propagate packing to all the op results.
  for (OpResult result : packedLinalgOp->getResults()) {
    int64_t resultNum = result.getResultNumber();
    linalg::PackOp maybePackedInit =
        inits[resultNum].getDefiningOp<linalg::PackOp>();
    if (!maybePackedInit) {
      results.push_back(result);
      continue;
    }
    // Build the symmetrical UnPackOp to the existing PackOp.
    unPackOps.push_back(linalg::UnPackOp::create(
        rewriter, packedLinalgOp->getLoc(), result, maybePackedInit.getSource(),
        maybePackedInit.getInnerDimsPos(), maybePackedInit.getMixedTiles()));
    results.push_back(unPackOps.back().getResult());
  }

  // Step 5. Replace `linalgOp`.
  rewriter.replaceOp(linalgOp, results);

```
- **EN**: Implements logic around `getResults`, `getResultNumber`, `PackOp>`, `push_back`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getResults`, `getResultNumber`, `PackOp>`, `push_back`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 598-618
```cpp
  // Return packedLinalgOp.
  return PackResult{packOps,
                    cast<linalg::LinalgOp>(packedLinalgOp.getOperation()),
                    unPackOps};
}

//===----------------------------------------------------------------------===//
// packTranspose transformation.
//===----------------------------------------------------------------------===//

/// Return a copy of `tensorType` after permutation by `permutationVector`.
// Note: Should be a new method in of MemRef/RankedTensor/VectorType::Builder
// but this would introduce a dependence on Dialect in IR.
// TODO: Restructure.
static RankedTensorType permuteShape(RankedTensorType tensorType,
                                     ArrayRef<int64_t> permutationVector) {
  SmallVector<int64_t> shape(tensorType.getShape());
  applyPermutationToVector(shape, permutationVector);
  return RankedTensorType::Builder(tensorType).setShape(shape);
}

```
- **EN**: Implements logic around `LinalgOp>`, `permuteShape`, `shape`, `applyPermutationToVector`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LinalgOp>`, `permuteShape`, `shape`, `applyPermutationToVector`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 619-637
```cpp
/// Return a new GenericOp obtained by transposing opOperand by the permutation
/// vector:
///   - the corresponding indexing map is transposed by `permutation`
///   - the corresponding operand value is replaced by `transposedValue`
/// `linalgOp` is replaced by the return op in the process.
/// Asserts that `transposedValue` is of the proper transposed ShapedType.
static LinalgOp transposeOneLinalgOperandAndReplace(
    RewriterBase &rewriter, LinalgOp linalgOp, OpOperand &opOperand,
    ArrayRef<int64_t> permutation, Value transposedValue) {
  // Sanity check the operand.
  assert(linalgOp == opOperand.getOwner() && "linalg op must own the operand");

  // Sanity check of the expected transposed tensor type.
  auto tensorType = permuteShape(
      cast<RankedTensorType>(opOperand.get().getType()), permutation);
  (void)tensorType;
  assert(tensorType == transposedValue.getType() &&
         "expected tensor type mismatch");

```
- **EN**: Implements logic around `transposeOneLinalgOperandAndReplace`, `assert`, `permuteShape`, `get`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `transposeOneLinalgOperandAndReplace`, `assert`, `permuteShape`, `get` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 638-666
```cpp
  // Compute the transposed indexing map.
  // Sigh unsigned pollution.
  SmallVector<unsigned> tmpTransposition =
      llvm::map_to_vector(permutation, [](int64_t i) -> unsigned { return i; });
  AffineMap permutationMap =
      AffineMap::getPermutationMap(tmpTransposition, rewriter.getContext());
  AffineMap transposedMap =
      permutationMap.compose(linalgOp.getMatchingIndexingMap(&opOperand));

  // Set the transposed indexing map in the proper position.
  SmallVector<AffineMap> indexingMaps = linalgOp.getIndexingMapsArray();
  indexingMaps[linalgOp.getIndexingMapIndex(&opOperand)] = transposedMap;
  // Set the transposedValue in the proper operand position.
  SmallVector<Value> operands = linalgOp->getOperands();
  operands[opOperand.getOperandNumber()] = transposedValue;

  ValueRange operandsRef(operands);
  auto transposedGenericOp = linalg::GenericOp::create(
      rewriter,
      /*location=*/linalgOp->getLoc(),
      /*resultTensorTypes=*/
      operandsRef.drop_front(linalgOp.getNumDpsInputs()).getTypes(),
      /*inputs=*/operandsRef.take_front(linalgOp.getNumDpsInputs()),
      /*outputs=*/operandsRef.drop_front(linalgOp.getNumDpsInputs()),
      /*indexingMaps=*/indexingMaps,
      /*iteratorTypes=*/linalgOp.getIteratorTypesArray());
  transposedGenericOp.getRegion().takeBody(linalgOp->getRegion(0));
  rewriter.replaceOp(linalgOp, transposedGenericOp->getResults());

```
- **EN**: Implements logic around `map_to_vector`, `getPermutationMap`, `compose`, `getIndexingMapsArray`, and 11 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `map_to_vector`, `getPermutationMap`, `compose`, `getIndexingMapsArray`, and 11 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 667-684
```cpp
  return cast<linalg::LinalgOp>(transposedGenericOp.getOperation());
}

FailureOr<PackTransposeResult>
linalg::packTranspose(RewriterBase &rewriter, linalg::PackOp packOp,
                      linalg::LinalgOp linalgOp, linalg::UnPackOp maybeUnPackOp,
                      ArrayRef<int64_t> outerPerm,
                      ArrayRef<int64_t> innerPerm) {
  Location loc = linalgOp.getLoc();

  // Step 1. Transpose packOp.
  rewriter.setInsertionPoint(packOp);
  linalg::PackOp transposedPackOp =
      packOp.createTransposedClone(rewriter, loc, innerPerm, outerPerm);

  if (packOp.hasPureBufferSemantics() || !packOp.getResult().hasOneUse())
    return rewriter.notifyMatchFailure(linalgOp, "expect single pack use");

```
- **EN**: Implements logic around `LinalgOp>`, `packTranspose`, `getLoc`, `setInsertionPoint`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LinalgOp>`, `packTranspose`, `getLoc`, `setInsertionPoint`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 685-720
```cpp
  OpOperand &packUse = *packOp->getUses().begin();
  if (packUse.getOwner() != linalgOp) {
    return rewriter.notifyMatchFailure(
        linalgOp, "not a single use by the LinalgOp target");
  }
  if (maybeUnPackOp &&
      (!linalgOp.isDpsInit(&packUse) ||
       maybeUnPackOp.getSource() != linalgOp.getTiedOpResult(&packUse))) {
    return rewriter.notifyMatchFailure(linalgOp,
                                       "not produced by the LinalgOp target");
  }

  // Step 2. Transpose linalgOp.
  // transposedPackOp.getOuterDimsPerm() may be empty, in which case it is the
  // identity. Don't rely on it.
  int64_t numLeadingDims = packOp.getSourceRank();
  int64_t numTrailingDims = packOp.getInnerDimsPos().size();
  // Step 2.a. Compute the permutation on the whole operand.
  // Leading part just reuse the outerPerm.
  SmallVector<int64_t> permutation(outerPerm);
  if (permutation.empty())
    llvm::append_range(permutation, llvm::seq<int64_t>(0, numLeadingDims));
  // Trailing part needs to reindex positions by `numLeadingDims`.
  if (innerPerm.empty()) {
    llvm::append_range(
        permutation,
        llvm::seq<int64_t>(numLeadingDims, numLeadingDims + numTrailingDims));
  } else {
    llvm::append_range(permutation,
                       llvm::map_range(innerPerm, [&](int64_t pos) {
                         return numLeadingDims + pos;
                       }));
  }
  if (!isPermutationVector(permutation))
    return rewriter.notifyMatchFailure(linalgOp, "invalid permutation");

```
- **EN**: Implements logic around `getUses`, `getOwner`, `notifyMatchFailure`, `isDpsInit`, and 9 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getUses`, `getOwner`, `notifyMatchFailure`, `isDpsInit`, and 9 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 721-738
```cpp
  // Step 2.b. Save the transposedPackUse operand number in case we need to
  // get the tied OpResult after `linalgOp` has been replaced.
  int64_t packUseOperandNumber = packUse.getOperandNumber();
  // Step 2.c. Actually perform the transposition.
  rewriter.setInsertionPoint(linalgOp);
  linalg::LinalgOp transposedLinalgOp = transposeOneLinalgOperandAndReplace(
      rewriter, linalgOp, packUse, permutation, transposedPackOp.getResult());

  // Step 3. Maybe transpose unPackOp.
  linalg::UnPackOp transposedUnPackOp;
  if (maybeUnPackOp) {
    OpOperand &opOperand =
        transposedLinalgOp->getOpOperand(packUseOperandNumber);
    OpResult transposedResult = transposedLinalgOp.getTiedOpResult(&opOperand);
    rewriter.setInsertionPoint(maybeUnPackOp);
    transposedUnPackOp = maybeUnPackOp.createTransposedClone(
        rewriter, loc, transposedResult, innerPerm, outerPerm);

```
- **EN**: Implements logic around `getOperandNumber`, `setInsertionPoint`, `transposeOneLinalgOperandAndReplace`, `getResult`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOperandNumber`, `setInsertionPoint`, `transposeOneLinalgOperandAndReplace`, `getResult`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 739-756
```cpp
    rewriter.replaceOp(maybeUnPackOp, transposedUnPackOp->getResults());
  }

  // Step 4. Finally, replace packOp now that we don't need it anymore.
  if (packOp.hasPureTensorSemantics())
    rewriter.replaceOp(packOp, transposedPackOp->getResults());
  else
    rewriter.eraseOp(packOp);

  return PackTransposeResult{transposedPackOp, transposedLinalgOp,
                             transposedUnPackOp};
}

//===----------------------------------------------------------------------===//
// packMatmulGreedily transformation.
//===----------------------------------------------------------------------===//

/// Pack a LinalgOp by greedily inferring matmul dimensions (m, n, k) where m
```
- **EN**: Implements logic around `replaceOp`, `hasPureTensorSemantics`, `eraseOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOp`, `hasPureTensorSemantics`, `eraseOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 757-775
```cpp
/// and n are proper parallel dimensions and k is a proper reduction
/// dimension. Packing occurs by rewriting the op as a linalg.generic and
/// calling linalg::pack by `mnkPackedSizes`. The order of the packed
/// dimensions is customizable: the `mnkOrder` is a permutation of {0, 1, 2}
/// to reorder {m, n, k} into one of the 8 possible forms. The outer
/// dimensions of the operands are not permuted at this time, this is left for
/// future work.
FailureOr<PackResult>
linalg::packMatmulGreedily(RewriterBase &rewriter, LinalgOp linalgOp,
                           ArrayRef<OpFoldResult> mnkPackedSizes,
                           ArrayRef<int64_t> mnkPaddedSizesNextMultipleOf,
                           ArrayRef<int64_t> mnkOrder) {
  assert(mnkPackedSizes.size() == 3 && "unexpected num of packing sizes");
  assert((mnkPaddedSizesNextMultipleOf.empty() ||
          mnkPaddedSizesNextMultipleOf.size() == 3) &&
         "num of packing sizes next multiple should be empty or of size 3");
  assert(mnkOrder.size() == 3 && "unexpected mnkOrder size");
  assert(isPermutationVector(mnkOrder) && "expected a permutation");

```
- **EN**: Implements logic around `packMatmulGreedily`, `assert`, `size`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `packMatmulGreedily`, `assert`, `size` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 776-798
```cpp
  int64_t numLoops = linalgOp.getNumLoops();
  if (numLoops <= 2) {
    LDBG() << "need 3+ loops to find a matmul to pack, got " << numLoops
           << " in: " << linalgOp;
    return rewriter.notifyMatchFailure(
        linalgOp, "need 3+ loops to find a matmul to pack");
  }

  // Locally adjust the desired iterator position of mnk and packing sizes.
  int64_t numPackedDims = mnkPackedSizes.size();
  SmallVector<int64_t> mmnnkkPos(numPackedDims);
  for (int64_t i = 0, e = numPackedDims; i < e; ++i)
    mmnnkkPos[i] = numLoops - numPackedDims + mnkOrder[i];
  SmallVector<OpFoldResult> packedSizes(numPackedDims);
  for (int64_t i = 0, e = numPackedDims; i < e; ++i)
    packedSizes[mnkOrder[i]] = mnkPackedSizes[i];
  SmallVector<int64_t> paddedSizesNextMultipleOf(numPackedDims);
  for (int64_t i = 0, e = numPackedDims; i < e; ++i) {
    paddedSizesNextMultipleOf[mnkOrder[i]] =
        mnkPaddedSizesNextMultipleOf.empty() ? 0
                                             : mnkPaddedSizesNextMultipleOf[i];
  }

```
- **EN**: Implements logic around `getNumLoops`, `LDBG`, `notifyMatchFailure`, `size`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getNumLoops`, `LDBG`, `notifyMatchFailure`, `size`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 799-817
```cpp
  // 1. Infer dims that are important for matmul.
  FailureOr<ContractionDimensions> maybeDimensions =
      inferContractionDims(linalgOp);
  if (failed(maybeDimensions)) {
    LDBG() << "couldn't infer matmul iterators in: " << linalgOp;
    return rewriter.notifyMatchFailure(linalgOp,
                                       "couldn't infer matmul iterators");
  }

  // 2. Normalize linalgOp to an kmn-matmul-like with [red, par, par] most
  // minor iterators. In cases with multiple options for m, n, k bias towards
  // the most minor embedding.
  // If we wanted a different normalization order, this is where it would have
  // to plug a heuristic.
  int64_t mPos = maybeDimensions->m.back(), nPos = maybeDimensions->n.back(),
          kPos = maybeDimensions->k.back();
  LDBG() << "Start packing generic op greedily with (m@" << mPos << ", n@"
         << nPos << ", k@" << kPos << "): " << linalgOp;

```
- **EN**: Implements logic around `inferContractionDims`, `failed`, `LDBG`, `notifyMatchFailure`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `inferContractionDims`, `failed`, `LDBG`, `notifyMatchFailure`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 818-840
```cpp
  // 2.a. Rewrite as a generic.
  auto genericOp = dyn_cast<GenericOp>(linalgOp.getOperation());
  if (!genericOp) {
    FailureOr<GenericOp> generalizeResult =
        generalizeNamedOp(rewriter, linalgOp);
    assert(succeeded(generalizeResult) && "unexpected failure generalizing op");
    genericOp = *generalizeResult;
  }

  // 2.b. Interchange to move the dimensions (k, m, n) as most-minor
  // iterators. Note that this only normalized the iteration order and does
  // not change the indexings of any operand.
  SmallVector<int64_t> permutation =
      computePermutationVector(numLoops, {mPos, nPos, kPos}, mmnnkkPos);
  LDBG() << "perm: " << llvm::interleaved(permutation);
  // Sign .. unsigned pollution.
  SmallVector<unsigned> unsignedPerm(permutation.begin(), permutation.end());
  FailureOr<GenericOp> interchangeResult =
      interchangeGenericOp(rewriter, genericOp, unsignedPerm);
  assert(succeeded(interchangeResult) && "unexpected failure interchanging op");
  genericOp = *interchangeResult;
  LDBG() << "Generalized Op to pack: " << genericOp;

```
- **EN**: Implements logic around `getOperation`, `generalizeNamedOp`, `assert`, `computePermutationVector`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getOperation`, `generalizeNamedOp`, `assert`, `computePermutationVector`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 841-858
```cpp
  // At this point, the op iterators are normalized to {leading, k, m, n}.
  // The layouts induced by packing will always be:
  //   - LHS{leading_lhs, kk, mm}
  //   - RHS{leading_rhs, kk, nn}
  //   - RES{leading_res, mm, nn}
  // If we wanted to change the packed order, we would reorder (k, m, n) to
  // something else above.
  //
  // Additional permutations of the outer dims of the operands (i.e.
  // leading_lhs, leading_rhs and leading_res) could follow by computing the
  // desired outerPerm for each operand.
  // This is left for future work.

  // TODO: this creates too much IR, go use reifyResultShapes.
  SmallVector<Range, 4> loopRanges =
      cast<LinalgOp>(genericOp.getOperation())
          .createLoopRanges(rewriter, genericOp.getLoc());

```
- **EN**: Implements logic around `getOperation`, `createLoopRanges`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getOperation`, `createLoopRanges` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 859-882
```cpp
  // Add leading zeros to match numLoops, we only pack the last 3 dimensions
  // post interchange.
  LDBG() << "paddedSizesNextMultipleOf: "
         << llvm::interleaved(paddedSizesNextMultipleOf);
  LDBG() << "loopRanges: "
         << llvm::interleaved(
                llvm::map_range(loopRanges, [](Range r) { return r.size; }));
  SmallVector<OpFoldResult> adjustedPackedSizes(numLoops - packedSizes.size(),
                                                rewriter.getIndexAttr(0));
  for (int64_t i = 0, e = numPackedDims; i < e; ++i) {
    if (paddedSizesNextMultipleOf[i] == 0) {
      adjustedPackedSizes.push_back(packedSizes[i]);
      continue;
    }
    AffineExpr d0, s0;
    bindDims(rewriter.getContext(), d0);
    bindSymbols(rewriter.getContext(), s0);
    adjustedPackedSizes.push_back(affine::makeComposedFoldedAffineApply(
        rewriter, genericOp->getLoc(), d0.ceilDiv(s0) * s0,
        {loopRanges[adjustedPackedSizes.size()].size,
         rewriter.getIndexAttr(paddedSizesNextMultipleOf[i])}));
  }
  LDBG() << "adjustedPackedSizes: " << llvm::interleaved(adjustedPackedSizes);

```
- **EN**: Implements logic around `LDBG`, `interleaved`, `map_range`, `adjustedPackedSizes`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LDBG`, `interleaved`, `map_range`, `adjustedPackedSizes`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 883-909
```cpp
  // TODO: If we wanted to give the genericOp a name after packing, after
  // calling `pack` would be a good time. One would still need to check that
  // `containsMostMinorMatmul(packingRes->packedLinalgOp)` is true, since we
  // also allow degenerate matmul cases (i.e. matvec, dot).
  return pack(rewriter, genericOp, adjustedPackedSizes);
}

//===----------------------------------------------------------------------===//
// Transformations exposed as rewrite patterns.
//===----------------------------------------------------------------------===//

LinalgTilingOptions &
mlir::linalg::LinalgTilingOptions::setTileSizes(ArrayRef<int64_t> ts) {
  assert(!tileSizeComputationFunction && "tile sizes already set");
  SmallVector<int64_t, 4> tileSizes(ts);
  tileSizeComputationFunction = [tileSizes](OpBuilder &b, Operation *op) {
    OpBuilder::InsertionGuard guard(b);
    b.setInsertionPointToStart(
        &op->getParentOfType<func::FuncOp>().getBody().front());
    return llvm::map_to_vector<4>(tileSizes, [&](int64_t s) {
      Value v = arith::ConstantIndexOp::create(b, op->getLoc(), s);
      return v;
    });
  };
  return *this;
}

```
- **EN**: Implements logic around `pack`, `setTileSizes`, `assert`, `tileSizes`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pack`, `setTileSizes`, `assert`, `tileSizes`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 910-927
```cpp
LogicalResult mlir::linalg::CopyVectorizationPattern::matchAndRewrite(
    memref::CopyOp copyOp, PatternRewriter &rewriter) const {
  return vectorizeCopy(rewriter, copyOp);
}

/// Filling `dest` using FillOp constant padding value if possible.
/// Otherwise, generate a tensor::GenerateOp.
Value DecomposePadOpPattern::createFillOrGenerateOp(
    RewriterBase &rewriter, tensor::PadOp padOp, Value dest,
    const SmallVector<Value> &dynSizes) const {
  auto padValue = padOp.getConstantPaddingValue();
  if (padValue) {
    // Move the padding value defined inside the PadOp block to outside.
    if (padValue.getParentBlock() == &padOp.getRegion().front())
      rewriter.moveOpBefore(padValue.getDefiningOp(), padOp);
    return FillOp::create(rewriter, padOp.getLoc(), padValue, dest).result();
  }

```
- **EN**: Implements logic around `matchAndRewrite`, `vectorizeCopy`, `createFillOrGenerateOp`, `getConstantPaddingValue`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `vectorizeCopy`, `createFillOrGenerateOp`, `getConstantPaddingValue`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 928-949
```cpp
  // Fill could not be optimized: Lower to tensor::GenerateOp with region.
  auto generateOp = tensor::GenerateOp::create(rewriter, padOp.getLoc(),
                                               padOp.getResultType(), dynSizes);
  // Copy region to new op.
  IRMapping bvm;
  padOp.getRegion().cloneInto(&generateOp.getRegion(), bvm);
  return generateOp;
}

LogicalResult
DecomposePadOpPattern::matchAndRewrite(tensor::PadOp padOp,
                                       PatternRewriter &rewriter) const {
  // Given an OpFoldResult, return an index-typed value.
  auto getIdxValue = [&](OpFoldResult ofr) {
    if (auto val = llvm::dyn_cast_if_present<Value>(ofr))
      return val;
    return arith::ConstantIndexOp::create(
               rewriter, padOp.getLoc(),
               cast<IntegerAttr>(cast<Attribute>(ofr)).getInt())
        .getResult();
  };

```
- **EN**: Implements logic around `create`, `getResultType`, `getRegion`, `matchAndRewrite`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResultType`, `getRegion`, `matchAndRewrite`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 950-967
```cpp
  auto resultType = padOp.getResultType();
  // Compute size of EmptyOp. Any combination of static/dynamic is supported.
  SmallVector<Value> dynSizes;
  SmallVector<int64_t> staticSizes;
  for (unsigned dim = 0; dim < resultType.getRank(); ++dim) {
    if (resultType.isDynamicDim(dim)) {
      auto srcSize = getIdxValue(tensor::getMixedSize(rewriter, padOp.getLoc(),
                                                      padOp.getSource(), dim));
      // Add low and high padding value.
      auto plusLow = rewriter.createOrFold<arith::AddIOp>(
          padOp.getLoc(), srcSize, getIdxValue(padOp.getMixedLowPad()[dim]));
      auto plusHigh = rewriter.createOrFold<arith::AddIOp>(
          padOp.getLoc(), plusLow, getIdxValue(padOp.getMixedHighPad()[dim]));
      dynSizes.push_back(plusHigh);
    }
    staticSizes.push_back(resultType.getDimSize(dim));
  }

```
- **EN**: Implements logic around `getResultType`, `getRank`, `isDynamicDim`, `getIdxValue`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getResultType`, `getRank`, `isDynamicDim`, `getIdxValue`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 968-985
```cpp
  // Init tensor and fill it with padding.
  Value emptyTensor =
      tensor::EmptyOp::create(rewriter, padOp.getLoc(), staticSizes,
                              resultType.getElementType(), dynSizes);
  Value fill = createFillOrGenerateOp(rewriter, padOp, emptyTensor, dynSizes);

  // Generate a InsertSliceOp for copying the PadOp source.
  auto sourceType = padOp.getSourceType();
  // Compute size of source of tensor::PadOp.
  SmallVector<OpFoldResult> srcSizes =
      tensor::getMixedSizes(rewriter, padOp.getLoc(), padOp.getSource());
  // Strides of InsertSliceOp are all 1.
  SmallVector<OpFoldResult> strides(sourceType.getRank(),
                                    rewriter.getIndexAttr(1));
  rewriter.replaceOpWithNewOp<tensor::InsertSliceOp>(
      padOp, padOp.getSource(), fill, padOp.getMixedLowPad(), srcSizes,
      strides);

```
- **EN**: Implements logic around `create`, `getElementType`, `createFillOrGenerateOp`, `getSourceType`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getElementType`, `createFillOrGenerateOp`, `getSourceType`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 986-1005
```cpp
  return success();
}

LogicalResult ExtractSliceOfPadTensorSwapPattern::matchAndRewrite(
    tensor::ExtractSliceOp sliceOp, PatternRewriter &rewriter) const {
  if (!sliceOp.hasUnitStride())
    return failure();

  auto padOp = sliceOp.getSource().getDefiningOp<tensor::PadOp>();
  if (!padOp)
    return failure();

  bool zeroSliceGuard = true;
  if (controlFn) {
    if (std::optional<bool> control = controlFn(sliceOp))
      zeroSliceGuard = *control;
    else
      return failure();
  }

```
- **EN**: Implements logic around `success`, `matchAndRewrite`, `hasUnitStride`, `failure`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `success`, `matchAndRewrite`, `hasUnitStride`, `failure`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1006-1025
```cpp
  FailureOr<TilingResult> tilingResult =
      tensor::bubbleUpPadSlice(rewriter, padOp, sliceOp.getMixedOffsets(),
                               sliceOp.getMixedSizes(), zeroSliceGuard);
  if (failed(tilingResult))
    return failure();

  RankedTensorType sourceType = sliceOp.getSourceType();
  RankedTensorType resultType = sliceOp.getResultType();

  // If the extract_slice is not rank-reduced, all shapes are static and the
  // data source is actually used. Rewrite into pad(extract_slice(x)).
  if (sourceType.getRank() == resultType.getRank()) {
    rewriter.replaceOp(sliceOp, tilingResult->tiledValues);
    return success();
  }

  // Handle rank-reduced slice by creating another extract_slice op.
  Value rankReduced = tensor::createCanonicalRankReducingExtractSliceOp(
      rewriter, sliceOp.getLoc(), tilingResult->tiledValues[0], resultType);

```
- **EN**: Implements logic around `bubbleUpPadSlice`, `getMixedSizes`, `failed`, `failure`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bubbleUpPadSlice`, `getMixedSizes`, `failed`, `failure`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1026-1045
```cpp
  rewriter.replaceOp(sliceOp, rankReduced);
  return success();
}

/// If padding value is set, returns a tensor.pad Op for the source tensor,
/// with the output shape matching the output of `packOp`. Otherwise, returns
/// the source directly.
///
/// This method assumes that all outer dims for this pack Op are 1.
static Value getPackOpSourceOrPaddedSource(OpBuilder &builder,
                                           linalg::PackOp packOp) {
  Value input = packOp.getSource();
  // TODO: Support Memref PackOp. Temporarily return just Op Source.
  if (!packOp.hasPureTensorSemantics())
    return input;

  if (!packOp.getPaddingValue()) {
    return input;
  }

```
- **EN**: Implements logic around `replaceOp`, `success`, `getPackOpSourceOrPaddedSource`, `getSource`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceOp`, `success`, `getPackOpSourceOrPaddedSource`, `getSource`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1046-1072
```cpp
  assert(llvm::all_of(packOp.getAllOuterDims(),
                      [](int64_t val) { return val == 1; }) &&
         "some outer dims are != 1");

  Location loc = packOp.getLoc();
  ShapedType inputType = packOp.getSourceType();
  int64_t inputRank = inputType.getRank();

  DenseMap<int64_t, OpFoldResult> tileAndPosMapping =
      packOp.getDimAndTileMapping();

  // The sizes of dynamic tiles
  SmallVector<Value> dynamicTileSizes;

  // Collect dims for the padded shape.
  SmallVector<int64_t> paddedShape;
  for (int64_t dimIdx = 0; dimIdx < inputRank; ++dimIdx) {
    // 1. Non-tiled outer dims.
    // These dims should be 1 and we simply preserve them.
    if (!tileAndPosMapping.count(dimIdx)) {
      int64_t inputDimSize = inputType.getDimSize(dimIdx);
      assert(inputDimSize == 1 &&
             "with all outer dims == 1, this non-tiled input dim should be 1!");
      paddedShape.push_back(inputDimSize);
      continue;
    }

```
- **EN**: Implements logic around `assert`, `getLoc`, `getSourceType`, `getRank`, and 4 more symbols.
- **CN**: 围绕 `assert`, `getLoc`, `getSourceType`, `getRank`, and 4 more symbols 实现具体逻辑。

### Lines 1073-1097
```cpp
    // 2. Tiled outer dims
    // As all outer dims == 1, it is safe to use the tile size for the padded
    // shape.
    OpFoldResult tileSizeForDim = tileAndPosMapping.lookup(dimIdx);

    // 2.1 Static tile sizes
    std::optional<int64_t> cstTileSize = getConstantIntValue(tileSizeForDim);
    if (cstTileSize.has_value()) {
      paddedShape.push_back(cstTileSize.value());
      continue;
    }

    // 2.2 Dynamic tile sizes
    paddedShape.push_back(ShapedType::kDynamic);

    // Get the value that holds the dynamic size.
    dynamicTileSizes.push_back(llvm::dyn_cast<Value>(tileSizeForDim));
  }
  auto resultType =
      RankedTensorType::get(paddedShape, inputType.getElementType());
  return tensor::createPadHighOp(resultType, input, packOp.getPaddingValue(),
                                 /*nofold=*/false, loc, builder,
                                 dynamicTileSizes);
}

```
- **EN**: Implements logic around `lookup`, `getConstantIntValue`, `has_value`, `push_back`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `lookup`, `getConstantIntValue`, `has_value`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1098-1133
```cpp
// Normalizes a permutation on a higher rank space to its actual size, e.g.
//   perm = [1, 4, 2]
// becomes
//   norm = [0, 2, 1]
static SmallVector<int64_t>
getPackUnpackNormalizedPerm(int rank, ArrayRef<int64_t> perm) {
  constexpr int64_t kNonTiledMarker = -1;
  SmallVector<int64_t> vec(rank, kNonTiledMarker);
  for (auto [index, value] : llvm::enumerate(perm))
    vec[value] = index;
  SmallVector<int64_t> normalizedPerm = llvm::filter_to_vector(
      vec, [&](int64_t v) { return v != kNonTiledMarker; });
  // This inverts the permutation in addition to normalizing so invert back.
  return invertPermutationVector(normalizedPerm);
}

// Gets the normalized permutation implied by innerDimsPos and outerDimsPerm
// assuming rank reduction of unit outer dims.
static SmallVector<int64_t>
getPackUnpackRankReducedPerm(ArrayRef<int64_t> shape,
                             ArrayRef<int64_t> innerDimsPos,
                             ArrayRef<int64_t> outerDimsPerm) {
  SmallVector<int64_t> rankReducedOuterDimsPerm;
  SmallVector<int64_t> outerDims;
  SmallVector<int64_t> innerDims;
  int64_t dim = 0;
  int64_t unpackedRank = shape.size();
  for (auto i : llvm::seq<unsigned>(0, unpackedRank)) {
    if (llvm::is_contained(innerDimsPos, i)) {
      innerDims.push_back(dim++);
      continue;
    }
    if (shape[i] == 1)
      continue;
    outerDims.push_back(dim++);
    if (!outerDimsPerm.empty())
```
- **EN**: Implements logic around `getPackUnpackNormalizedPerm`, `vec`, `enumerate`, `filter_to_vector`, and 7 more symbols.
- **CN**: 围绕 `getPackUnpackNormalizedPerm`, `vec`, `enumerate`, `filter_to_vector`, and 7 more symbols 实现具体逻辑。

### Lines 1134-1152
```cpp
      rankReducedOuterDimsPerm.push_back(outerDimsPerm[i]);
  }

  // Get the position of the inner dims after permutation.
  SmallVector<int64_t> innerPerm =
      getPackUnpackNormalizedPerm(unpackedRank, innerDimsPos);
  applyPermutationToVector<int64_t>(innerDims, innerPerm);

  // Ditto for the outer dims.
  SmallVector<int64_t> perm = outerDims;

  rankReducedOuterDimsPerm =
      getPackUnpackNormalizedPerm(unpackedRank, rankReducedOuterDimsPerm);
  if (!rankReducedOuterDimsPerm.empty())
    applyPermutationToVector<int64_t>(perm, rankReducedOuterDimsPerm);

  // The tile always ends up as the inner most dims after packing.
  perm.append(innerDims);

```
- **EN**: Implements logic around `push_back`, `getPackUnpackNormalizedPerm`, `applyPermutationToVector`, `empty`, and 1 more symbols.
- **CN**: 围绕 `push_back`, `getPackUnpackNormalizedPerm`, `applyPermutationToVector`, `empty`, and 1 more symbols 实现具体逻辑。

### Lines 1153-1170
```cpp
  return perm;
}

LogicalResult DecomposeOuterUnitDimsPackOpPattern::matchAndRewrite(
    linalg::PackOp packOp, PatternRewriter &rewriter) const {
  // TODO: Support Memref PackOp. Temporarily return failure.
  if (!packOp.hasPureTensorSemantics())
    return failure();

  if (llvm::any_of(packOp.getTiledOuterDims(),
                   [](int64_t dim) { return dim != 1; })) {
    return rewriter.notifyMatchFailure(
        packOp, "not all outer dimensions of the result are 1s");
  }

  ArrayRef<int64_t> innerDimsPos = packOp.getInnerDimsPos();
  auto outerDimsPerm = packOp.getOuterDimsPerm();

```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `failure`, `any_of`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `failure`, `any_of`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1171-1194
```cpp
  // Verify that there are no:
  //   * non-unit + un-tiled-outer-dims,
  // that are permuted. Supporting such cases would require refining the logic
  // that generates the Transpose Op.
  if (!llvm::all_of(outerDimsPerm, [&innerDimsPos, &packOp](int64_t dim) {
        static int prev = 0;
        // Skip tiled dims - these can be permuted.
        if (llvm::is_contained(innerDimsPos, dim))
          return true;

        // Check whether this dim has been permuted. Permuting unit dims is fine
        // as that's effectively a no-op.
        if (dim < prev && (packOp.getResult().getType().getShape()[prev] != 1 ||
                           packOp.getResult().getType().getShape()[dim] != 1))
          return false;

        prev = dim;
        return true;
      })) {
    return rewriter.notifyMatchFailure(
        packOp, "At least one non-unit and un-tiled outer dim is permuted, "
                "this is not supported ATM!");
  }

```
- **EN**: Implements logic around `all_of`, `is_contained`, `getResult`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; checks operation invariants or verification rules.
- **CN**: 围绕 `all_of`, `is_contained`, `getResult`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并检查操作不变式或验证规则。

### Lines 1195-1229
```cpp
  Location loc = packOp.getLoc();

  int64_t srcRank = packOp.getSourceRank();

  // 1. Get the input that is going to be packed. If the input requires padding,
  // add a padding operation and return that as the input.
  Value input = getPackOpSourceOrPaddedSource(rewriter, packOp);

  // 2. Transpose the input to match the inner tile order:
  //    %init = tensor.empty()
  //    %transposed_tile = linalg.transpose ins(%source_or_padded_source),
  //                                        outs(%init)
  // Assumptions made:
  //  - All tiled outer dims are 1 - the corresponding transposition order
  //    doesn't matter, but requires all dim indices to be present.
  //  - Un-tiled outer dims remain un-permuted.

  // 2.1 Get the permutation for linalg.transpose:
  //   [ untiled-dims, inner-dims-pos ]
  // Note, this logic assumes that the untiled dims are not permuted.
  SmallVector<int64_t> srcPermForTranspose;
  for (int64_t i = 0; i < srcRank; i++) {
    // We assume the `k` dimensions of the inner dim position, where `k` is the
    // rank of the inner tiling, correspond to the last `k` indices of the
    // transpose permutation. This is done by adding the indices not contained
    // in the inner dimension position in order from 0 to `n`. Where n is the
    // rank of the source tensor. For example if we have a source tensor with
    // indices [0, 1, 2, 3] and inner dim position of [3, 0], the remaining
    // indices are [1, 2]. and the transpose will be [1, 2, 3, 0].
    if (llvm::is_contained(innerDimsPos, i))
      continue;
    srcPermForTranspose.push_back(i);
  }
  srcPermForTranspose.append(innerDimsPos.begin(), innerDimsPos.end());

```
- **EN**: Implements logic around `getLoc`, `getSourceRank`, `getPackOpSourceOrPaddedSource`, `is_contained`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `getSourceRank`, `getPackOpSourceOrPaddedSource`, `is_contained`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1230-1258
```cpp
  // 2.2 Create the init tensor for linalg.transpose with the correct shape:
  //    [ untiled-dims, tiled-dims ]
  ShapedType inputTy = cast<ShapedType>(input.getType());
  SmallVector<OpFoldResult> shapeForEmptyOp;
  for (int64_t i = 0; i < srcRank; i++) {
    if (llvm::is_contained(innerDimsPos, i)) {
      // The tiled dims are appended after this loop.
      continue;
    }
    if (inputTy.isStaticDim(i))
      shapeForEmptyOp.push_back(rewriter.getIndexAttr(inputTy.getShape()[i]));
    else
      shapeForEmptyOp.emplace_back(
          tensor::DimOp::create(rewriter, loc, input, i).getResult());
  }
  shapeForEmptyOp.append(packOp.getMixedTiles());

  // getMixedTiles() may contain Values pointing to constant ops (as opposed to
  // constant attributes with the corresponding value). Replace those with
  // attributes. This is to match the behaviour in
  // `getPackOpSourceOrPaddedSource`, which replaces constant SSA values with
  // attributes.
  llvm::transform(shapeForEmptyOp, shapeForEmptyOp.begin(),
                  [&](OpFoldResult ofr) {
                    if (auto val = llvm::dyn_cast<Value>(ofr))
                      return getAsOpFoldResult(val);
                    return ofr;
                  });

```
- **EN**: Implements logic around `getType`, `is_contained`, `isStaticDim`, `push_back`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `is_contained`, `isStaticDim`, `push_back`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1259-1283
```cpp
  LDBG() << "Pack permutation: " << packOp;
  LDBG() << "perm: " << llvm::interleaved(srcPermForTranspose);
  LDBG() << "Shape of empty tensor: " << llvm::interleaved(shapeForEmptyOp);

  Value empty = tensor::EmptyOp::create(
      rewriter, loc, shapeForEmptyOp, packOp.getSourceType().getElementType());

  // 2.3 Create linalg.transpose
  auto transposedOp = linalg::TransposeOp::create(rewriter, loc, input, empty,
                                                  srcPermForTranspose);

  // 3. Insert the inner tile into the destination tensor:
  //  %inserted_tile = tensor.insert_slice(%transposed_tile)

  // Compute the sizes attribute:
  //    [ outer-dims, tile-sizes ]
  // Note that the output from the transpose Op excludes the tiled outer dims.
  // However, given the assumption that:
  //  * all tiled outer dims == 1,
  // we can just use a rank-expanding tensor.insert_slice.
  SmallVector<OpFoldResult> writeSizes;
  for (auto size : packOp.getAllOuterDims()) {
    writeSizes.push_back(rewriter.getIndexAttr(size));
  }

```
- **EN**: Implements logic around `LDBG`, `create`, `getSourceType`, `getAllOuterDims`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LDBG`, `create`, `getSourceType`, `getAllOuterDims`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1284-1303
```cpp
  for (auto tileSize : packOp.getMixedTiles()) {
    auto [_, tileSizeOfr] =
        getSimplifiedOfrAndStaticSizePair(tileSize, rewriter);
    writeSizes.push_back(tileSizeOfr);
  }

  auto insert = tensor::InsertSliceOp::create(
      rewriter, loc, transposedOp.getResult()[0], packOp.getDest(), writeSizes);

  // 4. Replace tensor.packOp with tensor.insert_slice created above
  rewriter.replaceOp(packOp, insert.getResult());

  return success();
}

LogicalResult DecomposeOuterUnitDimsUnPackOpPattern::matchAndRewrite(
    linalg::UnPackOp unpackOp, PatternRewriter &rewriter) const {
  if (!unpackOp.hasPureTensorSemantics())
    return failure();

```
- **EN**: Implements logic around `getMixedTiles`, `getSimplifiedOfrAndStaticSizePair`, `push_back`, `create`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMixedTiles`, `getSimplifiedOfrAndStaticSizePair`, `push_back`, `create`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1304-1321
```cpp
  int64_t destRank = unpackOp.getDestRank();
  ArrayRef<int64_t> srcShape = unpackOp.getSourceType().getShape();
  ArrayRef<int64_t> innerDimsPos = unpackOp.getInnerDimsPos();
  if (llvm::any_of(unpackOp.getTiledOuterDims(),
                   [](int64_t dim) { return dim != 1; })) {
    return rewriter.notifyMatchFailure(
        unpackOp,
        "require the tiled outer dimensions of the result are all 1s");
  }

  // 1. Use rank-reduced tensor.extract_slice op to extract the tile:
  //    %extracted_tile = tensor.extract_slice(%unpack_op_input)
  Location loc = unpackOp.getLoc();
  Value source = unpackOp.getSource();
  DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
      unpackOp.getDimAndTileMapping();
  Attribute oneIdxAttr = rewriter.getIndexAttr(1);

```
- **EN**: Implements logic around `getDestRank`, `getSourceType`, `getInnerDimsPos`, `any_of`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDestRank`, `getSourceType`, `getInnerDimsPos`, `any_of`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1322-1351
```cpp
  // The shape for ExtractSliceOp. Note that this will consist of 3 blocks of
  // dims:
  //    [ outer-untiled-dims, outer-tiled-dims, tile-sizes ]
  SmallVector<int64_t> readShapeForExtractSlice;
  // The sizes attribute for ExtractSliceOp. Due to rank-reducing (and
  // outer-tiled-dims being all 1), this will be
  //    [ outer-untiled-dims, tile-sizes ]
  SmallVector<OpFoldResult> extractSliceSizes;

  // Shape for EmptyOp that's used as the init value for TransposeOp below.
  // This should be:
  //    [ outer-untiled-dims, tile-sizes ]
  // However, skip unit dims - TransposeOp (below) applies rank-reduced
  // permutation.
  SmallVector<OpFoldResult> shapeForEmptyOp;

  for (auto i : llvm::seq<unsigned>(0, destRank)) {
    // Compute sizes attribute for ExtractSliceOp - outer-tiled-dims.
    //
    // As all outer tiled dims are 1, so the corresponding
    // slice size to read will also 1. As this will be rank-reducing "extract
    // slice" (i.e. the unit dims will be "collapsed"), there's no need to
    // update:
    //  * the output shape for ExtractSliceOp, nor
    //  * the shape for EmptyOp.
    if (dimAndTileMapping.count(i)) {
      extractSliceSizes.push_back(oneIdxAttr);
      continue;
    }

```
- **EN**: Implements logic around `seq`, `count`, `push_back`.
- **CN**: 围绕 `seq`, `count`, `push_back` 实现具体逻辑。

### Lines 1352-1375
```cpp
    // Compute sizes attribute for ExtractSliceOp + EmptyOp -
    // outer-untiled-dims
    if (ShapedType::isDynamic(srcShape[i])) {
      OpFoldResult dynamicDim =
          tensor::DimOp::create(rewriter, loc, source, i).getResult();
      extractSliceSizes.push_back(dynamicDim);
      shapeForEmptyOp.push_back(dynamicDim);
    } else {
      extractSliceSizes.push_back(rewriter.getIndexAttr(srcShape[i]));
      if (srcShape[i] != 1)
        shapeForEmptyOp.push_back(rewriter.getIndexAttr(srcShape[i]));
    }
    // Compute the output shape for ExtractSliceOp  - outer-untiled-dims (take
    // into account rank-reducing)
    if (srcShape[i] != 1) {
      readShapeForExtractSlice.push_back(srcShape[i]);
    }
  }
  // Append the tile sizes to "sizes attribute" for ExtractSliceOp and the
  // shape for EmptyOp.
  auto mixedTiles = unpackOp.getMixedTiles();
  extractSliceSizes.append(mixedTiles.begin(), mixedTiles.end());
  shapeForEmptyOp.append(mixedTiles.begin(), mixedTiles.end());

```
- **EN**: Implements logic around `isDynamic`, `create`, `push_back`, `getMixedTiles`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isDynamic`, `create`, `push_back`, `getMixedTiles`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1376-1397
```cpp
  // Explicitly create the type for extract_slice op because the inner tile
  // size could be 1. We want to represent the whole inner tile in this case.
  auto tileShape = srcShape.drop_front(destRank);
  // Append the inner tile shape to the permuted and rank-reduced outer shape.
  readShapeForExtractSlice.append(tileShape.begin(), tileShape.end());
  Type elemType = unpackOp.getSourceType().getElementType();
  auto readType = RankedTensorType::get(readShapeForExtractSlice, elemType);
  Value innerTile = tensor::ExtractSliceOp::create(
      rewriter, loc, readType, unpackOp.getSource(), extractSliceSizes);

  // 2. Transpose the tile to match the outer corresponding tile order.
  SmallVector<int64_t> perm = getPackUnpackRankReducedPerm(
      srcShape.take_front(destRank), innerDimsPos, unpackOp.getOuterDimsPerm());
  // Unpack is a transition out of packed space so we invert the permutation.
  perm = invertPermutationVector(perm);
  applyPermutationToVector<OpFoldResult>(shapeForEmptyOp, perm);

  Value empty =
      tensor::EmptyOp::create(rewriter, loc, shapeForEmptyOp, elemType);
  auto transposedOp =
      linalg::TransposeOp::create(rewriter, loc, innerTile, empty, perm);

```
- **EN**: Implements logic around `drop_front`, `append`, `getSourceType`, `get`, and 6 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `drop_front`, `append`, `getSourceType`, `get`, and 6 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1398-1423
```cpp
  // 3. Handle in-complete tiles if needed. It truncates trailing data from the
  // transposed tile.
  SmallVector<OpFoldResult> tileSizes;
  ArrayRef<int64_t> destShape = unpackOp.getDestType().getShape();
  for (auto i : llvm::seq<unsigned>(0, destRank)) {
    if (dimAndTileMapping.count(i) || destShape[i] != 1)
      tileSizes.push_back(
          tensor::getMixedSize(rewriter, loc, unpackOp.getDest(), i));
  }

  auto partialTile =
      tensor::ExtractSliceOp::create(rewriter, loc, RankedTensorType(),
                                     transposedOp.getResult()[0], tileSizes);

  // 4. Insert the result to the destination tensor.
  SmallVector<OpFoldResult> writeSizes;
  for (int i = 0, idx = 0; i < destRank; ++i) {
    if (dimAndTileMapping.count(i) || destShape[i] != 1)
      writeSizes.push_back(tileSizes[idx++]);
    else
      writeSizes.push_back(oneIdxAttr);
  }
  auto insert = tensor::InsertSliceOp::create(rewriter, loc, partialTile,
                                              unpackOp.getDest(), writeSizes);
  rewriter.replaceOp(unpackOp, insert.getResult());

```
- **EN**: Implements logic around `getDestType`, `seq`, `count`, `push_back`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDestType`, `seq`, `count`, `push_back`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1424-1447
```cpp
  return success();
}

//===----------------------------------------------------------------------===//
// Generic DownscaleSizeOneWindowedConvolution
//===----------------------------------------------------------------------===//
//
/// Returns the indices of affine map results that reference any of the given
/// dimensions.
static SmallVector<unsigned>
getResultIndicesReferencingDims(AffineMap map, ArrayRef<unsigned> dims) {
  SmallVector<unsigned> resultIndices;
  for (unsigned dim : dims) {
    for (unsigned i = 0, e = map.getNumResults(); i < e; ++i) {
      AffineExpr expr = map.getResult(i);
      if (expr.isFunctionOfDim(dim)) {
        resultIndices.push_back(i);
        break;
      }
    }
  }
  return resultIndices;
}

```
- **EN**: Implements logic around `success`, `getResultIndicesReferencingDims`, `getNumResults`, `getResult`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `success`, `getResultIndicesReferencingDims`, `getNumResults`, `getResult`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1448-1467
```cpp
/// Helper to create a rank-reducing extract_slice that removes specific
/// dimensions from a tensor.
static Value createRankReducingExtractSlice(RewriterBase &rewriter,
                                            Location loc, Value tensor,
                                            ArrayRef<unsigned> dimsToRemove) {
  auto tensorType = cast<RankedTensorType>(tensor.getType());
  int64_t rank = tensorType.getRank();

  // Compute new shape by removing the specified dimensions.
  SmallVector<int64_t> newShape;
  for (int64_t i = 0; i < rank; ++i) {
    if (!llvm::is_contained(dimsToRemove, i))
      newShape.push_back(tensorType.getDimSize(i));
  }

  auto newType = RankedTensorType::get(newShape, tensorType.getElementType());
  return tensor::createCanonicalRankReducingExtractSliceOp(rewriter, loc,
                                                           tensor, newType);
}

```
- **EN**: Implements logic around `createRankReducingExtractSlice`, `getType`, `getRank`, `is_contained`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createRankReducingExtractSlice`, `getType`, `getRank`, `is_contained`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1468-1486
```cpp
/// Drops specified dimensions from an AffineExpr and compresses remaining
/// dimension indices. Returns std::nullopt if the expression only references
/// the dropped dimensions.
static std::optional<AffineExpr>
dropDimsAndCompress(AffineExpr expr, ArrayRef<unsigned> dimsToDrop,
                    unsigned newNumDims, MLIRContext *ctx) {
  // Check if expr only references dimensions to be dropped.
  bool onlyReferencesDroppedDims = true;
  for (unsigned d = 0; d < newNumDims + dimsToDrop.size(); ++d) {
    if (expr.isFunctionOfDim(d) && !llvm::is_contained(dimsToDrop, d)) {
      onlyReferencesDroppedDims = false;
      break;
    }
  }
  if (onlyReferencesDroppedDims && llvm::any_of(dimsToDrop, [&](unsigned d) {
        return expr.isFunctionOfDim(d);
      }))
    return std::nullopt;

```
- **EN**: Implements logic around `dropDimsAndCompress`, `size`, `isFunctionOfDim`, `any_of`.
- **CN**: 围绕 `dropDimsAndCompress`, `size`, `isFunctionOfDim`, `any_of` 实现具体逻辑。

### Lines 1487-1508
```cpp
  // Replace dimensions: compute new index for each old dimension.
  // Dropped dimensions get mapped to constant 0, others get compressed.
  SmallVector<AffineExpr> dimReplacements;
  unsigned newDimIdx = 0;
  for (unsigned d = 0; d < newNumDims + dimsToDrop.size(); ++d) {
    if (llvm::is_contained(dimsToDrop, d)) {
      dimReplacements.push_back(getAffineConstantExpr(0, ctx));
    } else {
      dimReplacements.push_back(getAffineDimExpr(newDimIdx++, ctx));
    }
  }

  return expr.replaceDims(dimReplacements);
}

FailureOr<LinalgOp>
linalg::downscaleSizeOneWindowedConvolution(RewriterBase &rewriter,
                                            LinalgOp op) {
  auto maybeDims = inferConvolutionDims(op);
  if (failed(maybeDims))
    return failure();

```
- **EN**: Implements logic around `size`, `is_contained`, `push_back`, `replaceDims`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `size`, `is_contained`, `push_back`, `replaceDims`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1509-1528
```cpp
  // Currently supports only 2D convolutions.
  if (maybeDims->outputImage.size() != 2 || maybeDims->filterLoop.size() != 2)
    return failure();

  if (op.hasPureBufferSemantics())
    return failure();

  // Get loop domain indices for spatial dimensions.
  unsigned outSpatial0 = maybeDims->outputImage[0];
  unsigned outSpatial1 = maybeDims->outputImage[1];
  unsigned filterSpatial0 = maybeDims->filterLoop[0];
  unsigned filterSpatial1 = maybeDims->filterLoop[1];

  // Get sizes from loop bounds.
  SmallVector<int64_t, 4> loopRanges = op.getStaticLoopRanges();
  int64_t outSize0 = loopRanges[outSpatial0];
  int64_t outSize1 = loopRanges[outSpatial1];
  int64_t filterSize0 = loopRanges[filterSpatial0];
  int64_t filterSize1 = loopRanges[filterSpatial1];

```
- **EN**: Implements logic around `size`, `failure`, `hasPureBufferSemantics`, `getStaticLoopRanges`.
- **CN**: 围绕 `size`, `failure`, `hasPureBufferSemantics`, `getStaticLoopRanges` 实现具体逻辑。

### Lines 1529-1547
```cpp
  // Check if we can downscale by removing a spatial dimension.
  bool canRemoveSpatial0 = (filterSize0 == 1 && outSize0 == 1);
  bool canRemoveSpatial1 = (filterSize1 == 1 && outSize1 == 1);
  if (!canRemoveSpatial0 && !canRemoveSpatial1)
    return failure();

  // Determine which loop dims to remove (output spatial + corresponding filter)
  // and sort for correct index compression when removing dimensions from affine
  // maps.
  SmallVector<unsigned> loopDimsToRemove;
  if (canRemoveSpatial0) {
    loopDimsToRemove.push_back(outSpatial0);
    loopDimsToRemove.push_back(filterSpatial0);
  } else {
    loopDimsToRemove.push_back(outSpatial1);
    loopDimsToRemove.push_back(filterSpatial1);
  }
  llvm::sort(loopDimsToRemove);

```
- **EN**: Implements logic around `failure`, `push_back`, `sort`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `push_back`, `sort` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1548-1571
```cpp
  // Create new indexing maps with dimensions removed.
  SmallVector<AffineMap> newMaps;
  MLIRContext *ctx = op.getContext();
  unsigned numDims = op.getNumLoops();
  unsigned newNumDims = numDims - loopDimsToRemove.size();
  for (AffineMap map : op.getIndexingMapsArray()) {
    SmallVector<AffineExpr> newResults;
    for (AffineExpr expr : map.getResults()) {
      auto newExpr =
          dropDimsAndCompress(expr, loopDimsToRemove, newNumDims, ctx);
      if (newExpr)
        newResults.push_back(*newExpr);
    }
    newMaps.push_back(AffineMap::get(newNumDims, 0, newResults, ctx));
  }

  // Create new iterator types.
  SmallVector<utils::IteratorType> newIterTypes;
  auto iterTypes = op.getIteratorTypesArray();
  for (unsigned idx = 0; idx < iterTypes.size(); ++idx) {
    if (!llvm::is_contained(loopDimsToRemove, idx))
      newIterTypes.push_back(iterTypes[idx]);
  }

```
- **EN**: Implements logic around `getContext`, `getNumLoops`, `size`, `getIndexingMapsArray`, and 5 more symbols.
- **CN**: 围绕 `getContext`, `getNumLoops`, `size`, `getIndexingMapsArray`, and 5 more symbols 实现具体逻辑。

### Lines 1572-1590
```cpp
  // Rank-reduce operands using extract_slice.
  Location loc = op.getLoc();
  SmallVector<Value> newInputs;
  for (OpOperand *input : op.getDpsInputOperands()) {
    AffineMap map = op.getMatchingIndexingMap(input);
    SmallVector<unsigned> tensorDimsToRemove =
        getResultIndicesReferencingDims(map, loopDimsToRemove);
    Value reduced = createRankReducingExtractSlice(rewriter, loc, input->get(),
                                                   tensorDimsToRemove);
    newInputs.push_back(reduced);
  }

  OpOperand &output = *op.getDpsInitsMutable().begin();
  AffineMap outputMap = op.getMatchingIndexingMap(&output);
  SmallVector<unsigned> outputDimsToRemove =
      getResultIndicesReferencingDims(outputMap, loopDimsToRemove);
  Value newOutput = createRankReducingExtractSlice(rewriter, loc, output.get(),
                                                   outputDimsToRemove);

```
- **EN**: Implements logic around `getLoc`, `getDpsInputOperands`, `getMatchingIndexingMap`, `getResultIndicesReferencingDims`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getLoc`, `getDpsInputOperands`, `getMatchingIndexingMap`, `getResultIndicesReferencingDims`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1591-1610
```cpp
  // Create new linalg.generic with reduced dimensions.
  auto newOp =
      linalg::GenericOp::create(rewriter, loc, TypeRange{newOutput.getType()},
                                newInputs, newOutput, newMaps, newIterTypes);
  rewriter.inlineRegionBefore(op->getRegion(0), newOp.getRegion(),
                              newOp.getRegion().begin());

  // Try to specialize the generic back to a named op only if the input was
  // already a specialized (named) op.
  LinalgOp resultOp = newOp;
  if (!isa<GenericOp>(op)) {
    FailureOr<LinalgOp> specializedOp = specializeGenericOp(rewriter, newOp);
    if (succeeded(specializedOp))
      resultOp = *specializedOp;
  }

  // Insert result back into original shape.
  Value result = tensor::createCanonicalRankReducingInsertSliceOp(
      rewriter, loc, resultOp->getResult(0), output.get());

```
- **EN**: Implements logic around `create`, `inlineRegionBefore`, `getRegion`, `specializeGenericOp`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `inlineRegionBefore`, `getRegion`, `specializeGenericOp`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1611-1629
```cpp
  rewriter.replaceOp(op, result);
  return resultOp;
}

namespace {
/// Pattern wrapper around `downscaleSizeOneWindowedConvolution`.
struct DownscaleSizeOneWindowedConvolution final
    : public OpInterfaceRewritePattern<LinalgOp> {
  DownscaleSizeOneWindowedConvolution(MLIRContext *context,
                                      PatternBenefit benefit = 1)
      : OpInterfaceRewritePattern<LinalgOp>(context, benefit) {}

  LogicalResult matchAndRewrite(LinalgOp op,
                                PatternRewriter &rewriter) const override {
    return linalg::downscaleSizeOneWindowedConvolution(rewriter, op);
  }
};
} // namespace

```
- **EN**: Introduces declarations for `DownscaleSizeOneWindowedConvolution`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DownscaleSizeOneWindowedConvolution` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1630-1643
```cpp
void linalg::populateDecomposeConvolutionPatterns(RewritePatternSet &patterns,
                                                  PatternBenefit benefit) {
  patterns.add<DownscaleSizeOneWindowedConvolution>(patterns.getContext(),
                                                    benefit);
}

void linalg::populateDecomposePackUnpackPatterns(RewritePatternSet &patterns) {
  patterns.add<DecomposeOuterUnitDimsPackOpPattern>(patterns.getContext());
  patterns.add<DecomposeOuterUnitDimsUnPackOpPattern>(patterns.getContext());
}

void linalg::populateDecomposePadPatterns(RewritePatternSet &patterns) {
  patterns.add<DecomposePadOpPattern>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateDecomposeConvolutionPatterns`, `add`, `populateDecomposePackUnpackPatterns`, `populateDecomposePadPatterns`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateDecomposeConvolutionPatterns`, `add`, `populateDecomposePackUnpackPatterns`, `populateDecomposePadPatterns` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/Tensor/IR/TensorTilingInterfaceImpl.h`, `mlir/Dialect/Tensor/Utils/Utils.h` ... (+14 more)
- **Standard-library headers / 标准库头文件**: `<type_traits>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (14), LLVM support-library facilities / LLVM Support 库设施 (4), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1)
