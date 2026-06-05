# TilingInterfaceImpl.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/TilingInterfaceImpl.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `TilingInterfaceImpl`.
  - **CN**: 实现 Linalg 方言中围绕 `TilingInterfaceImpl` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
//===- TilingInterfaceImpl.cpp - Implementation of TilingInterface -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Transforms/TilingInterfaceImpl.h"

#include "mlir/Analysis/SliceAnalysis.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/Dialect/Utils/StructuredOpsUtils.h"
#include "mlir/IR/BuiltinTypeInterfaces.h"
#include "mlir/Interfaces/TilingInterface.h"
#include "mlir/Interfaces/ValueBoundsOpInterface.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/Support/Debug.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/TilingInterfaceImpl.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/TilingInterfaceImpl.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Utils.h`。

### Lines 31-55
```cpp
#define DEBUG_TYPE "linalg-tiling-interface-impl"

using namespace mlir;
using namespace mlir::linalg;

//===----------------------------------------------------------------------===//
// Utility methods for implementation of Tiling Interface for Linalg ops
//===----------------------------------------------------------------------===//

/// Return the SSA values that represent the data point accessed using a given
/// `indexingMap` for a given point in the iteration space represented by `ivs`.
static SmallVector<Value> getIndicesForAccess(OpBuilder &b, Location loc,
                                              AffineMap indexingMap,
                                              ValueRange ivs) {
  SmallVector<Value> indices;
  indices.reserve(indexingMap.getNumResults());
  for (auto result : indexingMap.getResults()) {
    AffineMap m = AffineMap::get(indexingMap.getNumDims(),
                                 indexingMap.getNumSymbols(), result);
    Value v = affine::AffineApplyOp::create(b, loc, m, ivs);
    indices.push_back(v);
  }
  return indices;
}

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 56-84
```cpp
/// Method to inline the payload of a `linalgOp` given the iteration space
/// point and values for the arguments of the payload.
static LogicalResult inlinePayload(OpBuilder &b, LinalgOp linalgOp,
                                   ValueRange ivs, ValueRange argValues) {
  Block *body = linalgOp.getBlock();
  IRMapping map;
  map.map(body->getArguments(), argValues);
  for (auto &op : body->without_terminator()) {
    if (auto indexOp = dyn_cast<IndexOp>(&op)) {
      map.map(indexOp.getResult(), ivs[indexOp.getDim()]);
      continue;
    }
    b.clone(op, map);
  }

  Operation *terminator = body->getTerminator();
  Location loc = terminator->getLoc();
  for (const auto &operand : llvm::enumerate(terminator->getOperands())) {
    Value toStore = map.lookupOrDefault(operand.value());
    OpOperand *storeInto = linalgOp.getDpsInitOperand(operand.index());
    auto indices = getIndicesForAccess(
        b, loc, linalgOp.getMatchingIndexingMap(storeInto), ivs);
    memref::StoreOp::create(b, loc, toStore,
                            linalgOp.getDpsInitOperand(operand.index())->get(),
                            indices);
  }
  return success();
}

```
- **EN**: Implements logic around `inlinePayload`, `getBlock`, `map`, `without_terminator`, and 10 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `inlinePayload`, `getBlock`, `map`, `without_terminator`, and 10 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 85-104
```cpp
//===----------------------------------------------------------------------===//
// External Model for implementing `TilingInterface` for `LinalgOp`s.
//===----------------------------------------------------------------------===//

namespace {
/// External model implementation of TilingInterface for LinalgOps. An external
/// model implementation is used for now till the use of `TilingInterface` is
/// on-par with the current Linalg tiling + fusion patterns. Once it is
/// maybe possible to move this into the op-definition (though there are
/// advantages to leaving it as an external model)
template <typename LinalgOpTy>
struct LinalgOpTilingInterface
    : public TilingInterface::ExternalModel<LinalgOpTilingInterface<LinalgOpTy>,
                                            LinalgOpTy> {
  /// Return the loop iterator type.
  SmallVector<utils::IteratorType> getLoopIteratorTypes(Operation *op) const {
    LinalgOpTy concreteOp = cast<LinalgOpTy>(op);
    return concreteOp.getIteratorTypesArray();
  }

```
- **EN**: Introduces declarations for `LinalgOpTilingInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgOpTilingInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 105-122
```cpp
  /// Return the iteration domain range.
  SmallVector<Range> getIterationDomain(Operation *op, OpBuilder &b) const {
    OpBuilder::InsertionGuard g(b);
    b.setInsertionPoint(op);
    Location loc = op->getLoc();
    LinalgOp linalgOp = cast<LinalgOp>(op);
    SmallVector<OpFoldResult> allShapesSizes =
        linalgOp.createFlatListOfOperandDims(b, loc);
    AffineMap map = linalgOp.getShapesToLoopsMap();

    return llvm::map_to_vector(map.getResults(), [&](AffineExpr loopExpr) {
      OpFoldResult ofr = affine::makeComposedFoldedAffineApply(b, loc, loopExpr,
                                                               allShapesSizes);
      return Range{b.getIndexAttr(0), ofr, b.getIndexAttr(1)};
    });
  }

  /// Instantiate the tiled implementation of the operation.
```
- **EN**: Implements logic around `getIterationDomain`, `g`, `setInsertionPoint`, `getLoc`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIterationDomain`, `g`, `setInsertionPoint`, `getLoc`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 123-142
```cpp
  FailureOr<TilingResult>
  getTiledImplementation(Operation *op, OpBuilder &b,
                         ArrayRef<OpFoldResult> offsets,
                         ArrayRef<OpFoldResult> sizes) const {
    // Leave the `sizeBounds` value empty. That is only needed when the `sizes`
    // specified could lead to out of bounds accesses.
    Location loc = op->getLoc();
    LinalgOp linalgOp = cast<LinalgOp>(op);
    SmallVector<Value> valuesToTile = linalgOp->getOperands();
    SmallVector<Value> tiledOperands = makeTiledShapes(
        b, loc, linalgOp, valuesToTile, offsets, sizes, {}, true);
    SmallVector<Operation *> generatedSlices = llvm::map_to_vector(
        llvm::make_filter_range(
            tiledOperands,
            [](Value v) -> bool {
              return isa_and_nonnull<tensor::ExtractSliceOp, memref::SubViewOp>(
                  v.getDefiningOp());
            }),
        [](Value v) -> Operation * { return v.getDefiningOp(); });

```
- **EN**: Implements logic around `getTiledImplementation`, `getLoc`, `getOperands`, `makeTiledShapes`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTiledImplementation`, `getLoc`, `getOperands`, `makeTiledShapes`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 143-164
```cpp
    SmallVector<Type> resultTensorTypes =
        getTensorOutputTypes(linalgOp, tiledOperands);

    Operation *tiledOp = clone(b, linalgOp, resultTensorTypes, tiledOperands);
    offsetIndices(b, cast<LinalgOp>(tiledOp), offsets);

    return TilingResult{
        {tiledOp}, SmallVector<Value>(tiledOp->getResults()), generatedSlices};
  }

  /// Utility to fetch the offsets and sizes when applied as per the indexing
  /// map of the linalg op. This helps in fusing the linalg op as a consumer of
  /// a given slice op.
  static LogicalResult
  getMappedOffsetAndSize(LinalgOp linalgOp, OpBuilder &b,
                         ArrayRef<AffineMap> indexingMaps,
                         ArrayRef<SmallVector<OpFoldResult>> allOffsets,
                         ArrayRef<SmallVector<OpFoldResult>> allSizes,
                         SmallVectorImpl<OpFoldResult> &mappedOffsetsVec,
                         SmallVectorImpl<OpFoldResult> &mappedSizesVec) {
    DenseMap<unsigned, OpFoldResult> mappedOffsets, mappedSizes;

```
- **EN**: Implements logic around `getTensorOutputTypes`, `clone`, `offsetIndices`, `SmallVector`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTensorOutputTypes`, `clone`, `offsetIndices`, `SmallVector`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 165-190
```cpp
    for (auto [indexingMap, offsets, sizes] :
         llvm::zip_equal(indexingMaps, allOffsets, allSizes)) {
      for (auto [resultExpr, offset, size] :
           llvm::zip_equal(indexingMap.getResults(), offsets, sizes)) {
        auto dimExpr = dyn_cast<AffineDimExpr>(resultExpr);
        if (!dimExpr)
          return failure();
        unsigned position = dimExpr.getPosition();
        auto it = mappedOffsets.find(position);
        if (it != mappedOffsets.end()) {
          OpFoldResult seenOffset = it->second;
          OpFoldResult seenSize = mappedSizes.lookup(position);
          if (seenOffset != offset || seenSize != size) {
            LLVM_DEBUG({
              llvm::dbgs() << "inconsistent iteration space mapping from "
                              "offsets/sizes of operands/results";
            });
            return failure();
          }
        } else {
          mappedOffsets[position] = offset;
          mappedSizes[position] = size;
        }
      }
    }

```
- **EN**: Implements logic around `zip_equal`, `failure`, `getPosition`, `find`, and 3 more symbols.
- **CN**: 围绕 `zip_equal`, `failure`, `getPosition`, `find`, and 3 more symbols 实现具体逻辑。

### Lines 191-209
```cpp
    // Aggregate from the given operand offsets and sizes, or default to
    // iteration space values.
    SmallVector<Range> iterationDomain =
        cast<TilingInterface>(linalgOp.getOperation()).getIterationDomain(b);
    mappedOffsetsVec.resize(iterationDomain.size());
    mappedSizesVec.resize(iterationDomain.size());
    for (auto [index, domain] : llvm::enumerate(iterationDomain)) {
      auto it = mappedOffsets.find(index);
      if (it != mappedOffsets.end()) {
        mappedOffsetsVec[index] = it->second;
        mappedSizesVec[index] = mappedSizes.lookup(index);
        continue;
      }
      mappedOffsetsVec[index] = domain.offset;
      mappedSizesVec[index] = domain.size;
    }
    return success();
  }

```
- **EN**: Implements logic around `getOperation`, `resize`, `enumerate`, `find`, and 3 more symbols.
- **CN**: 围绕 `getOperation`, `resize`, `enumerate`, `find`, and 3 more symbols 实现具体逻辑。

### Lines 210-232
```cpp
  /// Method to return the position of the result tile computed by the tiled
  /// operation.
  LogicalResult getIterationDomainTileFromOperandTiles(
      Operation *op, OpBuilder &b, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes,
      SmallVectorImpl<OpFoldResult> &iterDomainOffsets,
      SmallVectorImpl<OpFoldResult> &iterDomainSizes) const {
    auto linalgOp = cast<LinalgOp>(op);

    SmallVector<AffineMap> indexingMaps =
        llvm::map_to_vector(operandNumbers, [&](unsigned operandNumber) {
          OpOperand &opOperand = linalgOp->getOpOperand(operandNumber);
          return linalgOp.getMatchingIndexingMap(&opOperand);
        });
    if (failed(getMappedOffsetAndSize(linalgOp, b, indexingMaps, allOffsets,
                                      allSizes, iterDomainOffsets,
                                      iterDomainSizes))) {
      return failure();
    }
    return success();
  }

```
- **EN**: Implements logic around `getIterationDomainTileFromOperandTiles`, `map_to_vector`, `getOpOperand`, `getMatchingIndexingMap`, and 3 more symbols.
- **CN**: 围绕 `getIterationDomainTileFromOperandTiles`, `map_to_vector`, `getOpOperand`, `getMatchingIndexingMap`, and 3 more symbols 实现具体逻辑。

### Lines 233-250
```cpp
  /// Return the details of the output tile generated by the tiled
  /// implementation.
  LogicalResult
  getResultTilePosition(Operation *op, OpBuilder &b, unsigned resultNumber,
                        ArrayRef<OpFoldResult> offsets,
                        ArrayRef<OpFoldResult> sizes,
                        SmallVector<OpFoldResult> &resultOffsets,
                        SmallVector<OpFoldResult> &resultSizes) const {
    Location loc = op->getLoc();
    LinalgOp linalgOp = cast<LinalgOp>(op);

    AffineExpr d0;
    bindDims(b.getContext(), d0);
    SmallVector<OpFoldResult> subShapeSizes =
        llvm::map_to_vector(sizes, [&](OpFoldResult ofr) {
          return affine::makeComposedFoldedAffineApply(b, loc, d0 - 1, ofr);
        });

```
- **EN**: Implements logic around `getResultTilePosition`, `getLoc`, `bindDims`, `map_to_vector`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getResultTilePosition`, `getLoc`, `bindDims`, `map_to_vector`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 251-279
```cpp
    OpOperand *outOperand = linalgOp.getDpsInitOperand(resultNumber);
    SliceParameters sliceParams = computeSliceParameters(
        b, loc, outOperand->get(), sizes,
        linalgOp.getMatchingIndexingMap(outOperand), offsets,
        /*ubs*/ {}, subShapeSizes, true);
    resultOffsets = sliceParams.offsets;
    resultSizes = sliceParams.sizes;
    return success();
  }

  LogicalResult getIterationDomainTileFromResultTile(
      Operation *op, OpBuilder &b, unsigned resultNumber,
      ArrayRef<OpFoldResult> offsets, ArrayRef<OpFoldResult> sizes,
      SmallVectorImpl<OpFoldResult> &iterDomainOffsets,
      SmallVectorImpl<OpFoldResult> &iterDomainSizes) const {
    auto linalgOp = cast<LinalgOp>(op);

    // Check that the indexing map used for the output is a projected
    // permutation. This could be relaxed with a more general approach that can
    // map the offsets and sizes from the result to iteration space tiles
    // (filling in full extent for dimensions not used to access the result).
    AffineMap indexingMap =
        linalgOp.getIndexingMapMatchingResult(op->getResult(resultNumber));
    if (!indexingMap.isProjectedPermutation()) {
      return op->emitOpError(
          "unhandled tiled implementation generation when result is not "
          "accessed using a permuted projection");
    }

```
- **EN**: Implements logic around `getDpsInitOperand`, `computeSliceParameters`, `get`, `getMatchingIndexingMap`, and 5 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `getDpsInitOperand`, `computeSliceParameters`, `get`, `getMatchingIndexingMap`, and 5 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 280-302
```cpp
    SmallVector<OpFoldResult> allOffsets = llvm::to_vector(offsets);
    SmallVector<OpFoldResult> allSizes = llvm::to_vector(sizes);
    auto status =
        getMappedOffsetAndSize(linalgOp, b, indexingMap, {allOffsets},
                               {allSizes}, iterDomainOffsets, iterDomainSizes);
    (void)status;
    assert(succeeded(status) && "unexpected error in offset calculation");
    return success();
  }

  FailureOr<TilingResult>
  generateResultTileValue(Operation *op, OpBuilder &b, unsigned resultNumber,
                          ArrayRef<OpFoldResult> offsets,
                          ArrayRef<OpFoldResult> sizes) const {
    SmallVector<OpFoldResult> mappedOffsets, mappedSizes;
    if (failed(getIterationDomainTileFromResultTile(
            op, b, resultNumber, offsets, sizes, mappedOffsets, mappedSizes))) {
      return failure();
    }
    auto tilingInterfaceOp = cast<TilingInterface>(op);
    FailureOr<TilingResult> tilingResult =
        tilingInterfaceOp.getTiledImplementation(b, mappedOffsets, mappedSizes);

```
- **EN**: Implements logic around `to_vector`, `getMappedOffsetAndSize`, `assert`, `success`, and 4 more symbols.
- **CN**: 围绕 `to_vector`, `getMappedOffsetAndSize`, `assert`, `success`, and 4 more symbols 实现具体逻辑。

### Lines 303-329
```cpp
    if (failed(tilingResult))
      return failure();

    if (tilingResult->tiledOps.size() != 1)
      return op->emitOpError("failed to generate tiled implementation");

    return TilingResult{
        tilingResult->tiledOps,
        SmallVector<Value>{tilingResult->tiledValues[resultNumber]},
        tilingResult->generatedSlices};
  }

  /// Method to generate the tiled implementation of an operation from the tile
  /// of the operand.
  FailureOr<TilingResult> getTiledImplementationFromOperandTiles(
      Operation *op, OpBuilder &b, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes) const {
    SmallVector<OpFoldResult> mappedOffsets, mappedSizes;
    if (failed(getIterationDomainTileFromOperandTiles(
            op, b, operandNumbers, allOffsets, allSizes, mappedOffsets,
            mappedSizes))) {
      return failure();
    }
    return getTiledImplementation(op, b, mappedOffsets, mappedSizes);
  }

```
- **EN**: Implements logic around `failed`, `failure`, `size`, `emitOpError`, and 2 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `failed`, `failure`, `size`, `emitOpError`, and 2 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 330-357
```cpp
  LogicalResult generateScalarImplementation(Operation *op, OpBuilder &builder,
                                             Location loc,
                                             ValueRange ivs) const {
    auto linalgOp = cast<LinalgOp>(op);
    if (!linalgOp.hasPureBufferSemantics())
      return op->emitOpError("expected operation to have buffer semantics");

    SmallVector<Value> indexedValues;
    indexedValues.reserve(linalgOp->getNumOperands());
    Location linalgOpLoc = op->getLoc();
    /// Load the data corresponding to the block arguments that
    /// represent input operands.
    for (OpOperand &operand : linalgOp->getOpOperands()) {
      if (!linalgOp.payloadUsesValueFromOperand(&operand)) {
        indexedValues.push_back(nullptr);
        continue;
      }
      if (linalgOp.isScalar(&operand)) {
        indexedValues.push_back(operand.get());
        continue;
      }
      SmallVector<Value> indices = getIndicesForAccess(
          builder, linalgOpLoc, linalgOp.getMatchingIndexingMap(&operand), ivs);
      Value load =
          memref::LoadOp::create(builder, linalgOpLoc, operand.get(), indices);
      indexedValues.push_back(load);
    }

```
- **EN**: Implements logic around `generateScalarImplementation`, `hasPureBufferSemantics`, `emitOpError`, `reserve`, and 8 more symbols; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `generateScalarImplementation`, `hasPureBufferSemantics`, `emitOpError`, `reserve`, and 8 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 358-388
```cpp
    /// Inline the op payload and store the result.
    return inlinePayload(builder, linalgOp, ivs, indexedValues);
  }

  bool isOpFusableWithConsumerSlice(Operation *op, unsigned resultNumber,
                                    ArrayRef<OpFoldResult> offsets,
                                    ArrayRef<OpFoldResult> sizes) const {
    // The verifier gives all the necessary requirements for consumer fusion.
    return true;
  }

  bool isOpFusableWithProducerSlices(
      Operation *op, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes) const {

    auto linalgOp = cast<LinalgOp>(op);
    SmallVector<AffineMap> indexingMaps =
        llvm::map_to_vector(operandNumbers, [&](unsigned operandNumber) {
          OpOperand &opOperand = linalgOp->getOpOperand(operandNumber);
          return linalgOp.getMatchingIndexingMap(&opOperand);
        });
    // Check that offsets/sizes are consistent across all operands.
    OpBuilder b(op);
    SmallVector<OpFoldResult> mappedOffsets, mappedSizes;
    return succeeded(getMappedOffsetAndSize(linalgOp, b, indexingMaps,
                                            allOffsets, allSizes, mappedOffsets,
                                            mappedSizes));
  }
};

```
- **EN**: Implements logic around `inlinePayload`, `isOpFusableWithConsumerSlice`, `isOpFusableWithProducerSlices`, `map_to_vector`, and 4 more symbols.
- **CN**: 围绕 `inlinePayload`, `isOpFusableWithConsumerSlice`, `isOpFusableWithProducerSlices`, `map_to_vector`, and 4 more symbols 实现具体逻辑。

### Lines 389-406
```cpp
//===----------------------------------------------------------------------===//
// External Model for implementing `PartialReductionInterface` for `LinalgOp`s.
//===----------------------------------------------------------------------===//

/// In a given set vector, get the position of a particular element.
std::optional<int> getPositionIn(const llvm::SetVector<unsigned> &reductionDims,
                                 unsigned value) {
  for (auto [index, reductionDim] : llvm::enumerate(reductionDims)) {
    if (reductionDim == value) {
      return index;
    }
  }
  return std::nullopt;
}

/// Return an AffineMaps to use for the `outs` operands of the linalg op
/// generated for partial results. The new AffineMap is the AffineMap of the
/// untiled op with reduction dimensions appended at end in order in which they
```
- **EN**: Implements logic around `getPositionIn`, `enumerate`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPositionIn`, `enumerate` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 407-424
```cpp
/// were specified during tiling.
static SmallVector<AffineMap>
getPartialResultAffineMaps(LinalgOp linalgOp,
                           const SetVector<unsigned> &reductionDims) {
  auto partialReductionMaps = llvm::map_to_vector(
      linalgOp.getDpsInitsMutable(), [&](OpOperand &opOperand) {
        AffineMap map = linalgOp.getMatchingIndexingMap(&opOperand);
        for (auto redPos : reductionDims) {
          map =
              map.insertResult(getAffineDimExpr(redPos, linalgOp.getContext()),
                               map.getNumResults());
        }
        return map;
      });
  return partialReductionMaps;
}

struct InitSliceInfo {
```
- **EN**: Introduces declarations for `InitSliceInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `InitSliceInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 425-460
```cpp
  SmallVector<int64_t> resultShape;
  SmallVector<OpFoldResult> offsets;
  SmallVector<OpFoldResult> sizes;
  SmallVector<OpFoldResult> strides;
};

/// Return the result shape, offsets, sizes and strides of the slice of the
/// `initValue` to use as the destination of the partial reduction op generated
/// with outer reduction strategy.
static InitSliceInfo getInitSliceInfoForOuterReduction(
    MLIRContext *context, ArrayRef<OpFoldResult> offsets,
    ArrayRef<OpFoldResult> sizes, const SetVector<unsigned> &reductionDims,
    ArrayRef<OpFoldResult> splitReductionIvs, AffineMap partialReductionMap,
    ArrayRef<OpFoldResult> initOperandShape) {
  int64_t initRank = partialReductionMap.getNumResults();
  SmallVector<OpFoldResult> initOffsets, initSizes;
  Attribute zero = IntegerAttr::get(IndexType::get(context), 0);
  Attribute one = IntegerAttr::get(IndexType::get(context), 1);
  SmallVector<OpFoldResult> initStrides(initRank, one);
  for (auto [resultIdx, dimExpr] :
       llvm::enumerate(partialReductionMap.getResults())) {
    if (isa<AffineConstantExpr>(dimExpr)) {
      // A constant index in the output map accesses a fixed position; keep
      // the full output dimension to match the original output operand shape.
      initOffsets.push_back(zero);
      initSizes.push_back(initOperandShape[resultIdx]);
      continue;
    }
    unsigned dim = cast<AffineDimExpr>(dimExpr).getPosition();
    if (reductionDims.contains(dim)) {
      initOffsets.push_back(zero);
    } else {
      initOffsets.push_back(offsets[dim]);
    }
    initSizes.push_back(sizes[dim]);
  }
```
- **EN**: Implements logic around `getInitSliceInfoForOuterReduction`, `getNumResults`, `get`, `initStrides`, and 4 more symbols.
- **CN**: 围绕 `getInitSliceInfoForOuterReduction`, `getNumResults`, `get`, `initStrides`, and 4 more symbols 实现具体逻辑。

### Lines 461-496
```cpp
  SmallVector<int64_t> resultShape;
  std::tie(resultShape, std::ignore) = decomposeMixedValues(initSizes);
  return {resultShape, initOffsets, initSizes, initStrides};
}

/// Return the result shape, offsets, sizes and strides of the slice of the
/// `initValue` to use as destination of the partial reduction op generated with
/// outer parallel strategy.
static InitSliceInfo getInitSliceInfoForOuterParallel(
    MLIRContext *context, ArrayRef<OpFoldResult> offsets,
    ArrayRef<OpFoldResult> sizes, const SetVector<unsigned> &reductionDims,
    ArrayRef<OpFoldResult> splitReductionIvs, AffineMap partialReductionMap,
    ArrayRef<OpFoldResult> initOperandShape) {
  int64_t initRank = partialReductionMap.getNumResults();
  SmallVector<OpFoldResult> initOffsets, initSizes;
  Attribute zero = IntegerAttr::get(IndexType::get(context), 0);
  Attribute one = IntegerAttr::get(IndexType::get(context), 1);
  SmallVector<OpFoldResult> initStrides(initRank, one);
  SmallVector<OpFoldResult> resultShape;
  for (auto [resultIdx, dimExpr] :
       llvm::enumerate(partialReductionMap.getResults())) {
    if (isa<AffineConstantExpr>(dimExpr)) {
      // A constant index accesses a fixed position; keep the full output
      // dimension to match the original output operand shape.
      initOffsets.push_back(zero);
      initSizes.push_back(initOperandShape[resultIdx]);
      resultShape.push_back(initOperandShape[resultIdx]);
      continue;
    }
    unsigned dim = cast<AffineDimExpr>(dimExpr).getPosition();
    if (std::optional<unsigned> dimPos = getPositionIn(reductionDims, dim)) {
      initOffsets.push_back(splitReductionIvs[dimPos.value()]);
      initSizes.push_back(one);
    } else {
      initOffsets.push_back(offsets[dim]);
      initSizes.push_back(sizes[dim]);
```
- **EN**: Implements logic around `tie`, `getInitSliceInfoForOuterParallel`, `getNumResults`, `get`, and 5 more symbols.
- **CN**: 围绕 `tie`, `getInitSliceInfoForOuterParallel`, `getNumResults`, `get`, and 5 more symbols 实现具体逻辑。

### Lines 497-526
```cpp
      resultShape.push_back(sizes[dim]);
    }
  }
  SmallVector<int64_t> staticShapes;
  std::tie(staticShapes, std::ignore) = decomposeMixedValues(resultShape);
  return {staticShapes, initOffsets, initSizes, initStrides};
}

/// Return the result shape, offsets, sizes and strides of the slice of the
/// `initValue` to use as destination of the partial reduction op.
static InitSliceInfo getInitSliceInfo(MLIRContext *context,
                                      ReductionTilingStrategy strategy,
                                      ArrayRef<OpFoldResult> offsets,
                                      ArrayRef<OpFoldResult> sizes,
                                      const SetVector<unsigned> &reductionDims,
                                      ArrayRef<OpFoldResult> splitReductionIvs,
                                      AffineMap partialReductionMap,
                                      ArrayRef<OpFoldResult> initOperandShape) {
  if (strategy == ReductionTilingStrategy::PartialReductionOuterReduction) {
    return getInitSliceInfoForOuterReduction(
        context, offsets, sizes, reductionDims, splitReductionIvs,
        partialReductionMap, initOperandShape);
  }
  assert(strategy == ReductionTilingStrategy::PartialReductionOuterParallel &&
         "unexpected ReductionTilingStrategy");
  return getInitSliceInfoForOuterParallel(
      context, offsets, sizes, reductionDims, splitReductionIvs,
      partialReductionMap, initOperandShape);
}

```
- **EN**: Implements logic around `push_back`, `tie`, `getInitSliceInfo`, `getInitSliceInfoForOuterReduction`, and 2 more symbols.
- **CN**: 围绕 `push_back`, `tie`, `getInitSliceInfo`, `getInitSliceInfoForOuterReduction`, and 2 more symbols 实现具体逻辑。

### Lines 527-544
```cpp
/// External model implementation of PartialReductionInterface for
/// LinalgOps.
template <typename LinalgOpTy>
struct LinalgOpPartialReductionInterface
    : public PartialReductionOpInterface::ExternalModel<
          LinalgOpPartialReductionInterface<LinalgOpTy>, LinalgOpTy> {
  FailureOr<SmallVector<Value>> generateInitialTensorForPartialReduction(
      Operation *op, OpBuilder &b, Location loc, ArrayRef<OpFoldResult> sizes,
      const SetVector<unsigned> &reductionDims) const {
    auto linalgOp = cast<LinalgOp>(op);

    OpBuilder::InsertionGuard guard(b);
    if (linalgOp.hasPureBufferSemantics())
      return op->emitOpError("expected operation to have tensor semantics");

    SmallVector<AffineMap> partialResultMaps =
        getPartialResultAffineMaps(linalgOp, reductionDims);

```
- **EN**: Introduces declarations for `LinalgOpPartialReductionInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgOpPartialReductionInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 545-576
```cpp
    SmallVector<Value> inits;
    for (auto [initIdx, result, partialMap] :
         llvm::enumerate(linalgOp->getResults(), partialResultMaps)) {
      SmallVector<Operation *, 4> combinerOps;
      if (!matchReduction(linalgOp.getRegionOutputArgs(), initIdx,
                          combinerOps) ||
          combinerOps.size() != 1)
        return op->emitOpError("Failed to anaysis the reduction operation.");

      Operation *reductionOp = combinerOps[0];
      std::optional<TypedAttr> identity = arith::getNeutralElement(reductionOp);
      if (!identity.has_value())
        return op->emitOpError(
            "Failed to get an identity value for the reduction operation.");

      // Append the new partial result dimensions.
      SmallVector<OpFoldResult> partialResultShape;
      Value initValue = linalgOp.getDpsInits()[initIdx];
      SmallVector<OpFoldResult> initShape =
          tensor::getMixedSizes(b, loc, initValue);
      for (auto [resultIdx, dimExpr] :
           llvm::enumerate(partialMap.getResults())) {
        if (isa<AffineConstantExpr>(dimExpr)) {
          // A constant index in the output map accesses a fixed position; use
          // the actual output dimension size (not a hardcoded 1).
          partialResultShape.push_back(initShape[resultIdx]);
          continue;
        }
        auto dim = cast<AffineDimExpr>(dimExpr);
        partialResultShape.push_back(sizes[dim.getPosition()]);
      }

```
- **EN**: Implements logic around `enumerate`, `matchReduction`, `size`, `emitOpError`, and 5 more symbols; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `enumerate`, `matchReduction`, `size`, `emitOpError`, and 5 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 577-598
```cpp
      Type elType = getElementTypeOrSelf(result.getType());
      Value emptyTensor =
          tensor::EmptyOp::create(b, loc, partialResultShape, elType);
      Value constantOp = arith::ConstantOp::create(b, loc, *identity);
      auto identityTensor =
          linalg::FillOp::create(b, loc, constantOp, emptyTensor);
      inits.push_back(identityTensor.getResult(0));
    }

    return inits;
  }

  FailureOr<TilingResult>
  tileToPartialReduction(Operation *op, OpBuilder &b, Location loc,
                         ReductionTilingStrategy tilingStrategy,
                         ValueRange init, ArrayRef<OpFoldResult> offsets,
                         ArrayRef<OpFoldResult> sizes,
                         const SetVector<unsigned> &reductionDims,
                         ArrayRef<OpFoldResult> splitReductionIvs) const {
    OpBuilder::InsertionGuard guard(b);
    auto linalgOp = cast<LinalgOp>(op);

```
- **EN**: Implements logic around `getElementTypeOrSelf`, `create`, `push_back`, `tileToPartialReduction`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getElementTypeOrSelf`, `create`, `push_back`, `tileToPartialReduction`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 599-622
```cpp
    SmallVector<AffineMap> partialReductionMaps =
        getPartialResultAffineMaps(linalgOp, reductionDims);

    // Step 1. Extend init maps to have reduction dimension dims, since we
    // are converting them to parallel dimensions.
    SmallVector<AffineMap> newInitMaps;
    if (tilingStrategy ==
        ReductionTilingStrategy::PartialReductionOuterReduction) {
      newInitMaps = llvm::to_vector(partialReductionMaps);
    } else {
      newInitMaps = llvm::map_to_vector(
          linalgOp.getDpsInitsMutable(), [&](OpOperand &opOperand) {
            return linalgOp.getMatchingIndexingMap(&opOperand);
          });
    }

    // Step 2a: Extract a slice of the input operands.
    SmallVector<Value> tiledInputs = makeTiledShapes(
        b, loc, linalgOp, linalgOp.getDpsInputs(), offsets, sizes, {}, true);
    SmallVector<Operation *> generatedSlices = llvm::map_to_vector(
        llvm::make_filter_range(
            tiledInputs, [](Value v) -> bool { return v.getDefiningOp(); }),
        [](Value v) -> Operation * { return v.getDefiningOp(); });

```
- **EN**: Implements logic around `getPartialResultAffineMaps`, `to_vector`, `map_to_vector`, `getDpsInitsMutable`, and 5 more symbols.
- **CN**: 围绕 `getPartialResultAffineMaps`, `to_vector`, `map_to_vector`, `getDpsInitsMutable`, and 5 more symbols 实现具体逻辑。

### Lines 623-644
```cpp
    // Step 2b: Extract a slice of the init operands.
    SmallVector<Value, 1> tiledInits;
    for (auto [partialReductionMap, valueToTile, initOperandValue] :
         llvm::zip_equal(partialReductionMaps, init, linalgOp.getDpsInits())) {
      // Compute the actual shape of the original init operand for handling
      // constant expressions in the partial reduction map.
      SmallVector<OpFoldResult> initOperandShape =
          tensor::getMixedSizes(b, loc, initOperandValue);
      InitSliceInfo sliceInfo = getInitSliceInfo(
          b.getContext(), tilingStrategy, offsets, sizes, reductionDims,
          splitReductionIvs, partialReductionMap, initOperandShape);
      auto valueToTileType = cast<RankedTensorType>(valueToTile.getType());
      RankedTensorType sliceResultType = RankedTensorType::get(
          sliceInfo.resultShape, valueToTileType.getElementType(),
          valueToTileType.getEncoding());
      auto sliceOp = tensor::ExtractSliceOp::create(
          b, loc, sliceResultType, valueToTile, sliceInfo.offsets,
          sliceInfo.sizes, sliceInfo.strides);
      tiledInits.push_back(sliceOp.getResult());
      generatedSlices.push_back(sliceOp);
    }

```
- **EN**: Implements logic around `zip_equal`, `getMixedSizes`, `getInitSliceInfo`, `getContext`, and 6 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `zip_equal`, `getMixedSizes`, `getInitSliceInfo`, `getContext`, and 6 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 645-680
```cpp
    // Update the indexing maps.
    SmallVector<AffineMap> newMaps = linalgOp.getIndexingMapsArray();
    for (auto [initOperand, newInitMap] :
         llvm::zip_equal(linalgOp.getDpsInitsMutable(), newInitMaps)) {
      int mapIdx = linalgOp.getIndexingMapIndex(&initOperand);
      newMaps[mapIdx] = newInitMap;
    }

    // Step 3. Change the reduction dim iterator types.
    SmallVector<utils::IteratorType> newIteratorTypes =
        linalgOp.getIteratorTypesArray();
    if (tilingStrategy ==
        ReductionTilingStrategy::PartialReductionOuterReduction) {
      for (int dim : reductionDims)
        newIteratorTypes[dim] = utils::IteratorType::parallel;
    }

    // Step 4. Create the new generic op.
    Operation *partialReductionOp;
    auto resultTypes = ValueRange(tiledInits).getTypes();
    if (tilingStrategy ==
        ReductionTilingStrategy::PartialReductionOuterReduction) {
      auto genericOp = GenericOp::create(b, loc, resultTypes, tiledInputs,
                                         tiledInits, newMaps, newIteratorTypes);
      IRMapping mapping;
      op->getRegion(0).cloneInto(&genericOp.getRegion(),
                                 genericOp.getRegion().begin(), mapping);
      offsetIndices(b, genericOp, offsets);
      partialReductionOp = genericOp.getOperation();
    } else {
      SmallVector<Value> operands = std::move(tiledInputs);
      llvm::append_range(operands, tiledInits);
      partialReductionOp = mlir::clone(b, op, resultTypes, operands);
      offsetIndices(b, cast<LinalgOp>(partialReductionOp), offsets);
    }
    return TilingResult{
```
- **EN**: Implements logic around `getIndexingMapsArray`, `zip_equal`, `getIndexingMapIndex`, `getIteratorTypesArray`, and 8 more symbols.
- **CN**: 围绕 `getIndexingMapsArray`, `zip_equal`, `getIndexingMapIndex`, `getIteratorTypesArray`, and 8 more symbols 实现具体逻辑。

### Lines 681-715
```cpp
        {partialReductionOp},
        llvm::map_to_vector(partialReductionOp->getResults(),
                            [](OpResult r) -> Value { return r; }),
        generatedSlices};
  }

  FailureOr<MergeResult>
  mergeReductions(Operation *op, OpBuilder &b, Location loc,
                  ValueRange partialReduce,
                  const SetVector<unsigned> &reductionDims) const {
    auto linalgOp = cast<LinalgOp>(op);
    SmallVector<AffineMap> partialReductionMaps =
        getPartialResultAffineMaps(linalgOp, reductionDims);

    // Permute the reduction dims as permuted by the partial result map.
    SmallVector<Operation *> mergeOperations;
    SmallVector<Value> replacements;
    for (auto [idx, init, partialResult, partialMap] : llvm::enumerate(
             linalgOp.getDpsInits(), partialReduce, partialReductionMaps)) {
      unsigned initIdx = idx;
      // linalg.reduce's iteration space is the tiled result's iteration space
      // (and not the tiled operation's iteration space). To account for this,
      // permute the reduction dimensions based on the partial result map of the
      // tiled result.
      SmallVector<int64_t> partialReductionDims;
      for (auto [resultNum, dimExpr] :
           llvm::enumerate(partialMap.getResults())) {
        if (isa<AffineConstantExpr>(dimExpr))
          continue; // Constant dims are never reduction dims.
        unsigned dim = cast<AffineDimExpr>(dimExpr).getPosition();
        if (llvm::is_contained(reductionDims, dim)) {
          partialReductionDims.push_back(resultNum);
        }
      }

```
- **EN**: Implements logic around `map_to_vector`, `mergeReductions`, `getPartialResultAffineMaps`, `enumerate`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `map_to_vector`, `mergeReductions`, `getPartialResultAffineMaps`, `enumerate`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 716-733
```cpp
      auto reduction = linalg::ReduceOp::create(
          b, loc, partialResult, init, partialReductionDims,
          [&linalgOp, &initIdx](OpBuilder &b, Location loc, ValueRange inputs) {
            // Get the combiner op.
            SmallVector<Operation *, 4> combinerOps;
            matchReduction(linalgOp.getRegionOutputArgs(), initIdx,
                           combinerOps);
            Operation *clonedReductionOp = b.clone(*combinerOps[0]);
            // Combine the input at idx and output at numInits + idx.
            clonedReductionOp->setOperand(0, inputs[0]);
            clonedReductionOp->setOperand(1, inputs[1]);
            linalg::YieldOp::create(b, loc, clonedReductionOp->getResult(0));
          });

      mergeOperations.push_back(reduction);
      replacements.push_back(reduction->getResult(0));
    }

```
- **EN**: Implements logic around `create`, `matchReduction`, `clone`, `setOperand`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `matchReduction`, `clone`, `setOperand`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 734-759
```cpp
    return MergeResult{mergeOperations, replacements};
  }

  LogicalResult getPartialResultTilePosition(
      Operation *op, OpBuilder &b, unsigned resultNumber,
      ReductionTilingStrategy tilingStrategy, ArrayRef<OpFoldResult> offsets,
      ArrayRef<OpFoldResult> sizes, const SetVector<unsigned> &reductionDims,
      ArrayRef<OpFoldResult> splitReductionIvs,
      SmallVector<OpFoldResult> &resultOffsets,
      SmallVector<OpFoldResult> &resultSizes) const {
    auto linalgOp = cast<LinalgOp>(op);
    SmallVector<AffineMap> partialReductionMaps =
        getPartialResultAffineMaps(linalgOp, reductionDims);
    // Compute the actual shape of the init operand for handling constant
    // expressions in the partial reduction map.
    Value initOperandValue = linalgOp.getDpsInits()[resultNumber];
    Location loc = op->getLoc();
    SmallVector<OpFoldResult> initOperandShape =
        tensor::getMixedSizes(b, loc, initOperandValue);
    InitSliceInfo sliceInfo =
        getInitSliceInfo(b.getContext(), tilingStrategy, offsets, sizes,
                         reductionDims, splitReductionIvs,
                         partialReductionMaps[resultNumber], initOperandShape);
    std::swap(resultOffsets, sliceInfo.offsets);
    std::swap(resultSizes, sliceInfo.sizes);

```
- **EN**: Implements logic around `getPartialResultTilePosition`, `getPartialResultAffineMaps`, `getDpsInits`, `getLoc`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPartialResultTilePosition`, `getPartialResultAffineMaps`, `getDpsInits`, `getLoc`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 760-784
```cpp
    return success();
  }
};

template <typename OpTy>
static SmallVector<Range> getPackUnPackIterationDomain(OpTy op,
                                                       OpBuilder &builder) {
  static_assert(llvm::is_one_of<OpTy, PackOp, UnPackOp>::value,
                "applies to only pack or unpack operations");
  OpBuilder::InsertionGuard g(builder);
  int64_t rank = (std::is_same<OpTy, PackOp>::value) ? op.getSourceRank()
                                                     : op.getDestRank();
  OpFoldResult zero = builder.getIndexAttr(0);
  OpFoldResult one = builder.getIndexAttr(1);
  ReifiedRankedShapedTypeDims resultShape;
  (void)op.reifyResultShapes(builder, resultShape);
  SmallVector<Range> loopBounds(rank);
  for (auto dim : llvm::seq<int64_t>(0, rank)) {
    loopBounds[dim].offset = zero;
    loopBounds[dim].stride = one;
    loopBounds[dim].size = resultShape[0][dim];
  }
  return loopBounds;
}

```
- **EN**: Implements logic around `success`, `getPackUnPackIterationDomain`, `static_assert`, `g`, and 6 more symbols.
- **CN**: 围绕 `success`, `getPackUnPackIterationDomain`, `static_assert`, `g`, and 6 more symbols 实现具体逻辑。

### Lines 785-811
```cpp
static void applyPermToRange(SmallVector<OpFoldResult> &offsets,
                             SmallVector<OpFoldResult> &sizes,
                             ArrayRef<int64_t> permutation) {
  if (permutation.empty())
    return;
  applyPermutationToVector<OpFoldResult>(offsets, permutation);
  applyPermutationToVector<OpFoldResult>(sizes, permutation);
}

/// Compute the permutation vector to interchange `elements` such that the
/// elements at positions in `dimsPos` are moved to the positions `[0, ...,
/// dimsPos.size())` in order.
static SmallVector<int64_t>
computeInterchangeFromDimPos(ArrayRef<int64_t> dimsPos, int64_t rank) {
  SmallVector<int64_t> interchangeVector;
  interchangeVector.reserve(dimsPos.size());
  // First map dims and their position. For example, dims_pos = [2, 0] will map
  // to:
  // [
  //  [ key: 2, value: 0]
  //  [ key: 0, value: 1]
  // ]
  // where key is the idx in dims_pos while value its position in dims_pos.
  DenseMap<int64_t, int64_t> dimsAndPosMapping;
  for (int64_t dimsIdx = 0, end = dimsPos.size(); dimsIdx < end; dimsIdx++)
    dimsAndPosMapping[dimsPos[dimsIdx]] = dimsIdx;

```
- **EN**: Implements logic around `applyPermToRange`, `empty`, `applyPermutationToVector`, `computeInterchangeFromDimPos`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `applyPermToRange`, `empty`, `applyPermutationToVector`, `computeInterchangeFromDimPos`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 812-829
```cpp
  // Scan the position in order and insert the value in the map
  // to compute the interchange vector.
  for (int64_t dimsIdx = 0; dimsIdx < rank; dimsIdx++) {
    if (dimsAndPosMapping.count(dimsIdx))
      interchangeVector.push_back(dimsAndPosMapping[dimsIdx]);
  }
  return interchangeVector;
}

/// Permute the elements of `vec` starting at position `offset` according to
/// `interchangeVector`. The permutation maps position `i` in the permuted range
/// to position `interchangeVector[i]` in the original range. Elements before
/// `offset` are unchanged.
///
/// Example: interchange([a, b, c, d, e], [2, 0, 1], offset=2)
///          returns [a, b, e, c, d] (permutes the suffix [c, d, e])
///
/// Note: This is similar to `applyPermutationToVector` but supports an offset
```
- **EN**: Implements logic around `count`, `push_back`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `count`, `push_back` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 830-857
```cpp
/// for permuting a suffix of the vector. It is only used for pack/unpack scalar
/// implementation where we need to permute inner tile dimensions which are
/// stored at the end of the index vector.
template <typename T>
static SmallVector<T> interchange(ArrayRef<T> elements,
                                  ArrayRef<int64_t> interchangeVector,
                                  int offset = 0) {
  SmallVector<T> vec = llvm::to_vector(elements);
  for (auto [idx, val] : llvm::enumerate(interchangeVector))
    vec[idx + offset] = elements[val + offset];
  return vec;
}

/// Generate the body of the innermost loop of the scalar implementation
/// of `pack` operation.
static void generatePackOpScalarImplementationBody(PackOp packOp,
                                                   OpBuilder &builder,
                                                   Location loc,
                                                   ValueRange ivs) {
  // Note: `ivs` are already in the correct order, possibly interchanged based
  // on `dims_pos`. However, connecting the loops with the access patterns is
  // difficult - What is the relation between the position of the tile loop and
  // the point loop? However, if we interchange `ivs` once more to go to the
  // canonical blocking format: ABCabc, this connection becomes trivial: Each
  // point loop is pointLoopsOffset + inputRank away from the tiled loop.
  ArrayRef<int64_t> dimsToInnerBlock = packOp.getInnerDimsPos();
  ArrayRef<int64_t> dimsToOuterBlock = packOp.getOuterDimsPerm();

```
- **EN**: Implements logic around `interchange`, `to_vector`, `enumerate`, `generatePackOpScalarImplementationBody`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `interchange`, `to_vector`, `enumerate`, `generatePackOpScalarImplementationBody`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 858-891
```cpp
  SmallVector<Value> interchangedIvs = ivs;
  SmallVector<int64_t> interchangeVector =
      computeInterchangeFromDimPos(dimsToInnerBlock, packOp.getSourceRank());
  interchangedIvs = interchange<Value>(interchangedIvs, interchangeVector,
                                       /*offset=*/packOp.getSourceRank());
  if (!dimsToOuterBlock.empty()) {
    interchangeVector =
        computeInterchangeFromDimPos(dimsToOuterBlock, packOp.getSourceRank());
    interchangedIvs =
        interchange<Value>(interchangedIvs, interchangeVector, /*offset=*/0);
  }
  DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
      packOp.getDimAndTileMapping();
  SmallVector<OpFoldResult> sourceIndices;
  size_t pointLoopsOffset = 0;
  int64_t sourceRank = packOp.getSourceRank();
  for (auto dim : llvm::seq<int64_t>(0, sourceRank)) {
    if (dimAndTileMapping.contains(dim)) {
      AffineExpr i, j, tile;
      bindDims(builder.getContext(), i, j);
      bindSymbols(builder.getContext(), tile);
      OpFoldResult sourceIndex = affine::makeComposedFoldedAffineApply(
          builder, loc, i * tile + j,
          ArrayRef<OpFoldResult>{
              interchangedIvs[dim],
              interchangedIvs[pointLoopsOffset + packOp.getSourceRank()],
              dimAndTileMapping[dim]});
      sourceIndices.push_back(sourceIndex);
      ++pointLoopsOffset;
    } else {
      sourceIndices.push_back(interchangedIvs[dim]);
    }
  }

```
- **EN**: Implements logic around `computeInterchangeFromDimPos`, `interchange`, `getSourceRank`, `empty`, and 7 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `computeInterchangeFromDimPos`, `interchange`, `getSourceRank`, `empty`, and 7 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 892-921
```cpp
  auto createLoad = [&]() -> Value {
    return memref::LoadOp::create(
        builder, loc, packOp.getSource(),
        getValueOrCreateConstantIndexOp(builder, loc, sourceIndices));
  };
  Value scalar;
  if (auto paddingValue = packOp.getPaddingValue()) {
    ArithBuilder arithBuilder(builder, loc);
    Value isInBounds;
    for (auto dim : llvm::seq<int64_t>(0, sourceRank)) {
      Value idx =
          getValueOrCreateConstantIndexOp(builder, loc, sourceIndices[dim]);
      Value cond = arithBuilder.slt(
          idx, createOrFoldDimOp(builder, loc, packOp.getSource(), dim));
      isInBounds = dim == 0 ? cond : arithBuilder._and(isInBounds, cond);
    }
    scalar = scf::IfOp::create(
                 builder, loc, isInBounds, /*thenBuilder=*/
                 [&](OpBuilder &b, Location l) {
                   scf::YieldOp::create(b, l, createLoad());
                 },
                 /*elseBuilder=*/
                 [&](OpBuilder &b, Location l) {
                   scf::YieldOp::create(b, l, paddingValue);
                 })
                 .getResult(0);
  } else {
    scalar = createLoad();
  }

```
- **EN**: Implements logic around `create`, `getSource`, `getValueOrCreateConstantIndexOp`, `getPaddingValue`, and 7 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getSource`, `getValueOrCreateConstantIndexOp`, `getPaddingValue`, and 7 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 922-941
```cpp
  memref::StoreOp::create(builder, loc, scalar, packOp.getDest(), ivs);
}

struct PackOpTiling
    : public TilingInterface::ExternalModel<PackOpTiling, linalg::PackOp> {

  SmallVector<utils::IteratorType> getLoopIteratorTypes(Operation *op) const {
    // Note that here we only consider untiled dimensions and outer tiled data
    // dimensions, the inner tiled data dimensions are materialized when
    // building the body of the operation.
    auto packOp = cast<PackOp>(op);
    SmallVector<utils::IteratorType> iteratorTypes(
        packOp.getSourceRank(), utils::IteratorType::parallel);
    return iteratorTypes;
  }

  SmallVector<Range> getIterationDomain(Operation *op, OpBuilder &b) const {
    return getPackUnPackIterationDomain<PackOp>(cast<PackOp>(op), b);
  }

```
- **EN**: Introduces declarations for `PackOpTiling`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PackOpTiling` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 942-960
```cpp
  FailureOr<TilingResult>
  getTiledImplementation(Operation *op, OpBuilder &b,
                         ArrayRef<OpFoldResult> offsets,
                         ArrayRef<OpFoldResult> sizes) const {
    auto packOp = cast<PackOp>(op);
    // TODO: Support Memref PackOp. Temporarily return failure.
    if (!packOp.hasPureTensorSemantics())
      return failure();

    Location loc = packOp.getLoc();

    // The tiling is applied on interchanged dimensions. We have to undo the
    // interchange to map sizes and offsets to the original input.
    int64_t inputRank = packOp.getSourceRank();
    SmallVector<OpFoldResult> origOffsets(offsets);
    SmallVector<OpFoldResult> origSizes(sizes);
    applyPermToRange(origOffsets, origSizes,
                     invertPermutationVector(packOp.getOuterDimsPerm()));

```
- **EN**: Implements logic around `getTiledImplementation`, `hasPureTensorSemantics`, `failure`, `getLoc`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTiledImplementation`, `hasPureTensorSemantics`, `failure`, `getLoc`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 961-985
```cpp
    DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
        packOp.getDimAndTileMapping();
    SmallVector<OpFoldResult> srcDimValues =
        tensor::getMixedSizes(b, loc, packOp.getSource());
    SmallVector<OpFoldResult> inputIndices, inputSizes;
    for (auto dim : llvm::seq<int64_t>(0, inputRank)) {
      using AV = affine::AffineValueExpr;
      affine::AffineBuilder ab(b, loc);
      AffineExpr dim0, dim1, sym;
      bindDims(b.getContext(), dim0, dim1);
      bindSymbols(b.getContext(), sym);
      if (dimAndTileMapping.count(dim)) {
        // If the data dimension is tiled, the i-th index is the product of
        // offset_i and tile_i, and the i-th size is the product of sizes_i and
        // tile_i.
        auto avOffset = AV(dim0).bind(origOffsets[dim]);
        auto avSize = AV(dim0).bind(origSizes[dim]);
        auto avTileSize = AV(sym).bind(dimAndTileMapping[dim]);
        inputIndices.push_back(ab.mul(avOffset, avTileSize));
        inputSizes.push_back(ab.mul(avSize, avTileSize));
      } else {
        inputIndices.push_back(origOffsets[dim]);
        inputSizes.push_back(origSizes[dim]);
      }

```
- **EN**: Implements logic around `getDimAndTileMapping`, `getMixedSizes`, `seq`, `ab`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDimAndTileMapping`, `getMixedSizes`, `seq`, `ab`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 986-1003
```cpp
      // Limit the size of the input operand for incomplete tiles.
      if (packOp.getPaddingValue()) {
        OpFoldResult dimSize = srcDimValues[dim];
        auto avDimSize = AV(dim0).bind(dimSize);
        auto avInputIdx = AV(dim1).bind(inputIndices.back());
        inputSizes.back() =
            ab.min({inputSizes.back(), ab.sub(avDimSize, avInputIdx)});
      }
    }

    auto oneAttr = b.getI64IntegerAttr(1);
    SmallVector<OpFoldResult> strides(inputRank, oneAttr);

    SmallVector<Value> tiledOperands;
    auto sourceSlice = tensor::ExtractSliceOp::create(
        b, loc, packOp.getSource(), inputIndices, inputSizes, strides);
    tiledOperands.push_back(sourceSlice);

```
- **EN**: Implements logic around `getPaddingValue`, `AV`, `back`, `min`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPaddingValue`, `AV`, `back`, `min`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1004-1021
```cpp
    SmallVector<OpFoldResult> outputOffsets, outputSizes;
    if (failed(getResultTilePosition(op, b, 0, offsets, sizes, outputOffsets,
                                     outputSizes)))
      return {};

    strides.append(packOp.getDestRank() - inputRank, oneAttr);
    auto outSlice = tensor::ExtractSliceOp::create(
        b, loc, packOp.getDest(), outputOffsets, outputSizes, strides);
    tiledOperands.push_back(outSlice);

    if (auto val = packOp.getPaddingValue())
      tiledOperands.push_back(val);
    for (auto tile : packOp.getInnerTiles())
      tiledOperands.push_back(tile);

    Operation *tiledPackOp = PackOp::create(
        b, loc, TypeRange{outSlice.getType()}, tiledOperands, op->getAttrs());

```
- **EN**: Implements logic around `failed`, `append`, `create`, `getDest`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failed`, `append`, `create`, `getDest`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1022-1044
```cpp
    return TilingResult{
        {tiledPackOp},
        SmallVector<Value>(tiledPackOp->getResults()),
        llvm::to_vector(ArrayRef<Operation *>{sourceSlice, outSlice})};
  }

  LogicalResult
  getResultTilePosition(Operation *op, OpBuilder &b, unsigned resultNumber,
                        ArrayRef<OpFoldResult> offsets,
                        ArrayRef<OpFoldResult> sizes,
                        SmallVector<OpFoldResult> &resultOffsets,
                        SmallVector<OpFoldResult> &resultSizes) const {
    // The iteration domain is over outer dimensions of packed layout. In this
    // context, the outer dimensions of `resultOffsets` are `offsets`. The
    // inner dimensions of `resultOffsets` are zeros because tiling is not
    // applied to them.
    auto packOp = cast<PackOp>(op);
    int64_t inputRank = packOp.getSourceRank();
    int64_t outputRank = packOp.getDestRank();
    auto zeroAttr = b.getI64IntegerAttr(0);
    resultOffsets.assign(offsets.begin(), offsets.end());
    resultOffsets.append(outputRank - inputRank, zeroAttr);

```
- **EN**: Implements logic around `SmallVector`, `to_vector`, `getResultTilePosition`, `getSourceRank`, and 4 more symbols.
- **CN**: 围绕 `SmallVector`, `to_vector`, `getResultTilePosition`, `getSourceRank`, and 4 more symbols 实现具体逻辑。

### Lines 1045-1067
```cpp
    ReifiedRankedShapedTypeDims outputShape;
    (void)reifyResultShapes(b, packOp, outputShape);
    resultSizes.assign(sizes.begin(), sizes.end());
    for (auto dataTileDim : llvm::seq<unsigned>(inputRank, outputRank))
      resultSizes.push_back(outputShape[0][dataTileDim]);

    return success();
  }

  FailureOr<TilingResult>
  generateResultTileValue(Operation *op, OpBuilder &b, unsigned resultNumber,
                          ArrayRef<OpFoldResult> offsets,
                          ArrayRef<OpFoldResult> sizes) const {
    auto packOp = cast<PackOp>(op);
    int64_t numTiles = packOp.getInnerDimsPos().size();

    // tensor.pack op is fusible (as a producer) only if full inner tiles are
    // iterated or inner dims are not tiled. Otherwise, it will generate a
    // sequence of non-trivial ops (for partial tiles).
    for (auto offset : offsets.take_back(numTiles))
      if (!isZeroInteger(offset))
        return failure();

```
- **EN**: Implements logic around `reifyResultShapes`, `assign`, `seq`, `push_back`, and 6 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reifyResultShapes`, `assign`, `seq`, `push_back`, and 6 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1068-1090
```cpp
    for (auto iter :
         llvm::zip_equal(packOp.getMixedTiles(), sizes.take_back(numTiles)))
      if (!isEqualConstantIntOrValue(std::get<0>(iter), std::get<1>(iter)))
        return failure();

    FailureOr<TilingResult> tilingResult = getTiledImplementation(
        op, b, offsets.drop_back(numTiles), sizes.drop_back(numTiles));
    if (failed(tilingResult))
      return failure();
    return tilingResult.value();
  }

  LogicalResult generateScalarImplementation(Operation *op, OpBuilder &builder,
                                             Location loc,
                                             ValueRange ivs) const {
    auto packOp = cast<PackOp>(op);
    assert(packOp.hasPureBufferSemantics() &&
           "expected operation to have buffer semantics");
    OpBuilder::InsertionGuard g(builder);
    // The `ivs` already represent the position into the output for the non
    // data-tile dimensions.
    SmallVector<Value> ivVec(ivs);

```
- **EN**: Implements logic around `zip_equal`, `isEqualConstantIntOrValue`, `failure`, `getTiledImplementation`, and 7 more symbols.
- **CN**: 围绕 `zip_equal`, `isEqualConstantIntOrValue`, `failure`, `getTiledImplementation`, and 7 more symbols 实现具体逻辑。

### Lines 1091-1125
```cpp
    // Get output shape - for memrefs, get dimensions from dest directly.
    SmallVector<OpFoldResult> outputShape;
    Value dest = packOp.getDest();
    for (auto dim : llvm::seq<int64_t>(0, packOp.getDestRank()))
      outputShape.push_back(createOrFoldDimOp(builder, loc, dest, dim));

    // Generate the loops that iterate over the data tile.
    Value zero = arith::ConstantIndexOp::create(builder, loc, 0);
    Value one = arith::ConstantIndexOp::create(builder, loc, 1);

    // All loops except the innermost are simple loops that just iterate
    // over the tile dimensions.
    for (auto dataTileDim : llvm::seq<unsigned>(packOp.getSourceRank(),
                                                packOp.getDestRank() - 1)) {
      Value ub = getValueOrCreateConstantIndexOp(builder, loc,
                                                 outputShape[dataTileDim]);
      scf::ForOp loop = scf::ForOp::create(builder, loc, zero, ub, one);
      builder.setInsertionPointToStart(loop.getBody());
      ivVec.push_back(loop.getInductionVar());
    }
    // The body of the innermost loops does the actual data movement.
    scf::ForOp::create(
        builder, loc, zero,
        getValueOrCreateConstantIndexOp(builder, loc, outputShape.back()), one,
        ValueRange{},
        [&](OpBuilder &bodyBuilder, Location bodyLoc, Value iv,
            ValueRange regionIterArgs) {
          ivVec.push_back(iv);
          generatePackOpScalarImplementationBody(packOp, bodyBuilder, bodyLoc,
                                                 ivVec);
          scf::YieldOp::create(bodyBuilder, bodyLoc);
        });
    return success();
  }

```
- **EN**: Implements logic around `getDest`, `seq`, `push_back`, `create`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDest`, `seq`, `push_back`, `create`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1126-1161
```cpp
  /// Method to return the position of iteration domain tile computed by the
  /// tiled operation. In current `tensor.pack` context, the `resultOffsets` and
  /// `resultSizes` only cover outer dimensions.
  LogicalResult getIterationDomainTileFromOperandTiles(
      Operation *op, OpBuilder &b, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes,
      SmallVectorImpl<OpFoldResult> &resultOffsets,
      SmallVectorImpl<OpFoldResult> &resultSizes) const {
    if (operandNumbers.size() != 1 || operandNumbers[0] != 0) {
      LLVM_DEBUG(
          { llvm::dbgs() << "unsupported operands for consumer fusion"; });
      return failure();
    }

    ArrayRef<OpFoldResult> offsets(allOffsets[0]);
    ArrayRef<OpFoldResult> sizes(allSizes[0]);
    auto packOp = cast<PackOp>(op);
    Location loc = packOp.getLoc();
    SmallVector<OpFoldResult> outerDimOffsets, outerDimSizes;
    DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
        packOp.getDimAndTileMapping();
    SmallVector<int64_t> outerShapeWithoutTranspose(
        packOp.getDestType().getShape().take_front(packOp.getSourceRank()));
    if (!packOp.getOuterDimsPerm().empty()) {
      applyPermutationToVector(
          outerShapeWithoutTranspose,
          invertPermutationVector(packOp.getOuterDimsPerm()));
    }
    for (auto dim : llvm::seq<int64_t>(packOp.getSourceRank())) {
      if (dimAndTileMapping.count(dim)) {
        FailureOr<int64_t> cstTileSize =
            ValueBoundsConstraintSet::computeConstantBound(
                presburger::BoundType::UB, sizes[dim],
                /*stopCondition=*/nullptr,
                ValueBoundsOptions{/*closedUB=*/true});
```
- **EN**: Implements logic around `getIterationDomainTileFromOperandTiles`, `size`, `dbgs`, `failure`, and 12 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIterationDomainTileFromOperandTiles`, `size`, `dbgs`, `failure`, and 12 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1162-1187
```cpp
        std::optional<int64_t> cstInnerSize =
            getConstantIntValue(dimAndTileMapping[dim]);

        // If a dimension is not tiled, it is always valid to fuse the pack op,
        // even if the op has padding semantics. Because it always generates a
        // full slice along the dimension. The tile sizes are for unpacked
        // domain, i.e., `srcDimSize`, so `tileSize < srcDimSize` means that the
        // dimension is tiled.
        // TODO: It could be untiled if the `srcDimSize` is dynamic. It is a
        // hard check to determine if a dimension is tiled or not.
        int64_t srcDimSize = packOp.getSourceType().getDimSize(dim);
        int64_t destDimSize = outerShapeWithoutTranspose[dim];
        bool isTiled = failed(cstTileSize) ||
                       ShapedType::isDynamic(srcDimSize) ||
                       cstTileSize.value() < srcDimSize;
        if (!isTiled) {
          outerDimOffsets.push_back(offsets[dim]);
          if (ShapedType::isStatic(destDimSize)) {
            outerDimSizes.push_back(b.getIndexAttr(destDimSize));
          } else {
            outerDimSizes.push_back(
                b.createOrFold<tensor::DimOp>(loc, packOp.getDest(), dim));
          }
          continue;
        }

```
- **EN**: Implements logic around `getConstantIntValue`, `getSourceType`, `failed`, `isDynamic`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getConstantIntValue`, `getSourceType`, `failed`, `isDynamic`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1188-1205
```cpp
        // Currently fusing `packOp` as consumer only expects perfect tiling
        // scenario because even if without padding semantic, the `packOp` may
        // also yield incomplete tiles. E.g. tensor<30xf32> -> tensor<5x6xf32>,
        // where the `tileSize` from operand of `packOp` is 5, which is not
        // exactly divided by `innerTile`(=6) of `packOp`. As the result:
        // 1. the first slice is extracted from (0) to (4) and inserted into
        // (0,0)~(0,4) at first row.
        // 2. the second slice is extracted from (5) to (9) and SHOULD BE
        // respectively inserted into two rows with different length, including
        // first row: (0,5) and second row (1,0)~(1,3). It is hard to coordinate
        // them, thus adding below constraint to bypass them temporarily. In
        // another word, we can only support tiling with consumer if the tile
        // size for the producer is a multiple of the inner tile size for the
        // packed dimensions at this moment.
        if ((failed(cstTileSize) || !cstInnerSize ||
             *cstTileSize % *cstInnerSize != 0))
          return failure();

```
- **EN**: Implements logic around `failed`, `failure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failed`, `failure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1206-1226
```cpp
        using AV = affine::AffineValueExpr;
        affine::AffineBuilder ab(b, loc);
        AffineExpr dim0, sym;
        bindDims(b.getContext(), dim0);
        bindSymbols(b.getContext(), sym);
        auto avOffset = AV(dim0).bind(offsets[dim]);
        auto avSize = AV(dim0).bind(sizes[dim]);
        auto avTileSize = AV(sym).bind(dimAndTileMapping[dim]);
        outerDimOffsets.push_back(ab.floor(avOffset, avTileSize));
        outerDimSizes.push_back(ab.ceil(avSize, avTileSize));
      } else {
        outerDimOffsets.push_back(offsets[dim]);
        outerDimSizes.push_back(sizes[dim]);
      }
    }
    applyPermToRange(outerDimOffsets, outerDimSizes, packOp.getOuterDimsPerm());
    resultOffsets = outerDimOffsets;
    resultSizes = outerDimSizes;
    return success();
  }

```
- **EN**: Implements logic around `ab`, `bindDims`, `bindSymbols`, `AV`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ab`, `bindDims`, `bindSymbols`, `AV`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1227-1245
```cpp
  /// Method to return the tiled implementation of tensor.pack as a consumer.
  FailureOr<TilingResult> getTiledImplementationFromOperandTiles(
      Operation *op, OpBuilder &b, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes) const {
    if (operandNumbers.size() != 1 || operandNumbers[0] != 0) {
      LLVM_DEBUG(
          { llvm ::dbgs() << "unhandled operands for consumer fusion"; });
      return failure();
    }

    ArrayRef<OpFoldResult> offsets(allOffsets[0]);
    ArrayRef<OpFoldResult> sizes(allSizes[0]);

    auto packOp = cast<PackOp>(op);
    // TODO: Support Memref UnPackOp. Temporarily return failure.
    if (!packOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Implements logic around `getTiledImplementationFromOperandTiles`, `size`, `dbgs`, `failure`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTiledImplementationFromOperandTiles`, `size`, `dbgs`, `failure`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1246-1267
```cpp
    Location loc = packOp.getLoc();

    int64_t inputRank = packOp.getSourceRank();
    auto oneAttr = b.getI64IntegerAttr(1);
    SmallVector<OpFoldResult> strides(inputRank, oneAttr);

    SmallVector<Value> tiledOperands;
    auto sourceSlice = tensor::ExtractSliceOp::create(
        b, loc, packOp.getSource(), offsets, sizes, strides);
    tiledOperands.push_back(sourceSlice);

    SmallVector<OpFoldResult> outerDimOffsets, outerDimSizes;
    if (failed(getIterationDomainTileFromOperandTiles(
            op, b, operandNumbers, allOffsets, allSizes, outerDimOffsets,
            outerDimSizes)))
      return failure();

    SmallVector<OpFoldResult> outputOffsets, outputSizes;
    if (failed(getResultTilePosition(op, b, 0, outerDimOffsets, outerDimSizes,
                                     outputOffsets, outputSizes)))
      return failure();

```
- **EN**: Implements logic around `getLoc`, `getSourceRank`, `getI64IntegerAttr`, `strides`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `getSourceRank`, `getI64IntegerAttr`, `strides`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1268-1287
```cpp
    strides.append(packOp.getDestRank() - inputRank, oneAttr);
    auto outSlice = tensor::ExtractSliceOp::create(
        b, loc, packOp.getDest(), outputOffsets, outputSizes, strides);
    tiledOperands.push_back(outSlice);

    if (auto val = packOp.getPaddingValue())
      tiledOperands.push_back(val);
    for (auto tile : packOp.getInnerTiles())
      tiledOperands.push_back(tile);

    Operation *tiledPackOp = PackOp::create(
        b, loc, TypeRange{outSlice.getType()}, tiledOperands, op->getAttrs());

    return TilingResult{
        {tiledPackOp},
        SmallVector<Value>(tiledPackOp->getResults()),
        llvm::to_vector(ArrayRef<Operation *>{sourceSlice, outSlice})};
  }
};

```
- **EN**: Implements logic around `append`, `create`, `getDest`, `push_back`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `append`, `create`, `getDest`, `push_back`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1288-1317
```cpp
struct UnpackTileDimInfo {
  bool isAlignedToInnerTileSize;
  OpFoldResult sourceOffset;
  OpFoldResult sourceSize;
  OpFoldResult resultOffset;
  OpFoldResult destExpandedSize;
};

/// Returns the needed information for tiling unpack op on `tileDim` with given
/// `tileOffset` and `tileSize`. For more details, see the comment of the
/// `getTiledImplementation`.
static UnpackTileDimInfo getUnpackTileDimInfo(OpBuilder &b, UnPackOp unpackOp,
                                              int64_t tileDim,
                                              OpFoldResult tileOffset,
                                              OpFoldResult tileSize) {
  UnpackTileDimInfo info;
  Attribute zeroAttr = b.getIndexAttr(0);
  Attribute oneAttr = b.getIndexAttr(1);
  DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
      unpackOp.getDimAndTileMapping();
  // The dimension is not one of packed data dimension.
  if (!dimAndTileMapping.count(tileDim)) {
    info.isAlignedToInnerTileSize = true;
    info.sourceOffset = tileOffset;
    info.sourceSize = tileSize;
    info.resultOffset = zeroAttr;
    info.destExpandedSize = tileSize;
    return info;
  }

```
- **EN**: Introduces declarations for `UnpackTileDimInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `UnpackTileDimInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1318-1335
```cpp
  Location loc = unpackOp.getLoc();
  using AV = affine::AffineValueExpr;
  affine::AffineBuilder ab(b, loc);
  AffineExpr dim0, dim1, sym0;
  bindDims(b.getContext(), dim0, dim1);
  bindSymbols(b.getContext(), sym0);

  OpFoldResult innerTileSize = dimAndTileMapping[tileDim];

  info.isAlignedToInnerTileSize = false;
  FailureOr<int64_t> cstSize = ValueBoundsConstraintSet::computeConstantBound(
      presburger::BoundType::UB, tileSize,
      /*stopCondition=*/nullptr, ValueBoundsOptions{/*closedUB=*/true});
  std::optional<int64_t> cstInnerSize = getConstantIntValue(innerTileSize);
  if (!failed(cstSize) && cstInnerSize) {
    if (*cstSize % *cstInnerSize == 0)
      info.isAlignedToInnerTileSize = true;

```
- **EN**: Implements logic around `getLoc`, `ab`, `bindDims`, `bindSymbols`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `ab`, `bindDims`, `bindSymbols`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1336-1354
```cpp
    // If the tiling size equals to the inner tiling size, the outer dims are
    // always 1.
    if (*cstInnerSize == *cstSize) {
      auto lhs = AV(dim0).bind(tileOffset);
      auto rhs = AV(dim1).bind(innerTileSize);
      info.sourceOffset = ab.floor(lhs, rhs);
      info.sourceSize = oneAttr;
      info.resultOffset = zeroAttr;
      info.destExpandedSize = tileSize;
      return info;
    }
  }

  if (info.isAlignedToInnerTileSize) {
    info.sourceOffset =
        ab.floor(AV(dim0).bind(tileOffset), AV(dim1).bind(innerTileSize));
    info.resultOffset = zeroAttr;
    info.destExpandedSize = tileSize;

```
- **EN**: Implements logic around `AV`, `floor`.
- **CN**: 围绕 `AV`, `floor` 实现具体逻辑。

### Lines 1355-1377
```cpp
    // The ceilDiv is needed here because there could be incomplete tile even
    // it is perfect tiling cases. E.g.,
    //   %0 = unpack tensor<33x2xf32> into tensor<64xf32>
    // If the tiling size is 32, there will be 3 tiles. Two of them have
    // size=32; one of them have size=2. The size is represented using
    // affine_min op; we need ceilDiv.
    info.sourceSize =
        ab.ceil(AV(dim0).bind(tileSize), AV(dim1).bind(innerTileSize));
    return info;
  }

  affine::DivModValue firstCoord = affine::getDivMod(
      b, loc, getValueOrCreateConstantIndexOp(b, loc, tileOffset),
      getValueOrCreateConstantIndexOp(b, loc, innerTileSize));
  OpFoldResult tileExclusiveBound =
      ab.add(AV(dim0).bind(tileOffset), AV(dim1).bind(tileSize));
  affine::DivModValue lastCoord = affine::getDivMod(
      b, loc,
      getValueOrCreateConstantIndexOp(
          b, loc,
          ab.sub(AV(dim0).bind(tileExclusiveBound), AV(dim1).bind(oneAttr))),
      getValueOrCreateConstantIndexOp(b, loc, innerTileSize));

```
- **EN**: Implements logic around `ceil`, `getDivMod`, `getValueOrCreateConstantIndexOp`, `add`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ceil`, `getDivMod`, `getValueOrCreateConstantIndexOp`, `add`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1378-1401
```cpp
  OpFoldResult lengthMinusOne = ab.sub(AV(dim0).bind(lastCoord.quotient),
                                       AV(dim1).bind(firstCoord.quotient));
  info.sourceSize =
      ab.add(AV(dim0).bind(lengthMinusOne), AV(dim1).bind(oneAttr));
  info.sourceOffset = firstCoord.quotient;
  info.resultOffset = firstCoord.remainder;
  // Do not create an Affine ops for expanded size because the affine op is too
  // complicated which would trigger an issue in affine ops simplification.
  info.destExpandedSize = b.createOrFold<arith::MulIOp>(
      loc, getValueOrCreateConstantIndexOp(b, loc, info.sourceSize),
      getValueOrCreateConstantIndexOp(b, loc, innerTileSize));
  return info;
}

struct UnPackOpTiling
    : public TilingInterface::ExternalModel<UnPackOpTiling, linalg::UnPackOp> {

  SmallVector<utils::IteratorType> getLoopIteratorTypes(Operation *op) const {
    auto unpackOp = cast<UnPackOp>(op);
    SmallVector<utils::IteratorType> iteratorTypes(
        unpackOp.getDestRank(), utils::IteratorType::parallel);
    return iteratorTypes;
  }

```
- **EN**: Introduces declarations for `UnPackOpTiling`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `UnPackOpTiling` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1402-1419
```cpp
  SmallVector<Range> getIterationDomain(Operation *op, OpBuilder &b) const {
    return getPackUnPackIterationDomain<UnPackOp>(cast<UnPackOp>(op), b);
  }

  /// There are two cases in tiling unpack ops. If the tiling size is aligned to
  /// the inner tile size, the corresponding tiles of source are all complete.
  /// Otherwise, there are in-complete tiles. We will need to expand the slice
  /// of source for getting complete tiles. The tiled unpack op unpacks more
  /// data from source, so We'll need an extract_slice op to shift and truncate
  /// the output.
  /// Take Nn_to_N as an example. Say that N=32, n=8, and tiling_size=15. The
  /// coordinates of second tile (i.e., result[15..31]) are
  /// [(1, 7), (2, 0,), (2, 1) ... (3, 6), (3, 7)]. The first row and the last
  /// row are incomplete tiles. To represent the unpack op, we have to complete
  /// the rows. I.e., the input coordinates would start with (1, 0); end with
  /// (3, 7). In this context, the tiled unpack produces a (3 * n) elements
  /// because there are 3 rows in total. Follow by a tensor.extract_slice op, we
  /// can get the actual result.
```
- **EN**: Implements logic around `getIterationDomain`, `getPackUnPackIterationDomain`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIterationDomain`, `getPackUnPackIterationDomain` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1420-1452
```cpp
  FailureOr<TilingResult>
  getTiledImplementation(Operation *op, OpBuilder &b,
                         ArrayRef<OpFoldResult> offsets,
                         ArrayRef<OpFoldResult> sizes) const {
    auto unpackOp = cast<UnPackOp>(op);
    // TODO: Support Memref UnPackOp. Temporarily return failure.
    if (!unpackOp.hasPureTensorSemantics())
      return failure();

    int64_t srcRank = unpackOp.getSourceRank();
    int64_t destRank = unpackOp.getDestRank();
    int64_t numInnerTiles = srcRank - destRank;
    Location loc = unpackOp.getLoc();

    // The perfect tiling case indicates that the tiling sizes are multiple of
    // inner_tile_size. In this context, no extra data is needed when
    // representing the tiled unpack op.
    bool isPerfectTilingCase = true;
    Attribute oneAttr = b.getIndexAttr(1);
    SmallVector<OpFoldResult> sliceSrcStrides(destRank, oneAttr);
    SmallVector<OpFoldResult> sliceSrcIndices, sliceSrcSizes;
    SmallVector<OpFoldResult> destExpandedSizes, resultOffsetsFromDest;
    for (auto dim : llvm::seq<int64_t>(0, destRank)) {
      UnpackTileDimInfo info =
          getUnpackTileDimInfo(b, unpackOp, dim, offsets[dim], sizes[dim]);
      if (!info.isAlignedToInnerTileSize)
        isPerfectTilingCase = false;
      sliceSrcIndices.push_back(info.sourceOffset);
      sliceSrcSizes.push_back(info.sourceSize);
      destExpandedSizes.push_back(info.destExpandedSize);
      resultOffsetsFromDest.push_back(info.resultOffset);
    }

```
- **EN**: Implements logic around `getTiledImplementation`, `hasPureTensorSemantics`, `failure`, `getSourceRank`, and 7 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTiledImplementation`, `hasPureTensorSemantics`, `failure`, `getSourceRank`, and 7 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1453-1478
```cpp
    // The tiling is applied on destination dimensions. We have to apply the
    // interchange on source dimensions if outer_dims_perm is set.
    applyPermToRange(sliceSrcIndices, sliceSrcSizes,
                     unpackOp.getOuterDimsPerm());
    Attribute zeroAttr = b.getIndexAttr(0);
    sliceSrcIndices.append(numInnerTiles, zeroAttr);
    sliceSrcSizes.append(unpackOp.getMixedTiles());
    sliceSrcStrides.append(numInnerTiles, oneAttr);
    SmallVector<Operation *> generatedSlices;
    tensor::ExtractSliceOp sliceSource = tensor::ExtractSliceOp::create(
        b, loc, unpackOp.getSource(), sliceSrcIndices, sliceSrcSizes,
        sliceSrcStrides);
    generatedSlices.push_back(sliceSource);

    SmallVector<OpFoldResult> destStrides(destRank, oneAttr);
    Value sliceDest;
    if (isPerfectTilingCase) {
      auto destSliceOp = tensor::ExtractSliceOp::create(
          b, loc, unpackOp.getDest(), offsets, sizes, destStrides);
      sliceDest = destSliceOp;
      generatedSlices.push_back(destSliceOp);
    } else {
      sliceDest = tensor::EmptyOp::create(
          b, loc, destExpandedSizes, unpackOp.getDestType().getElementType());
    }

```
- **EN**: Implements logic around `applyPermToRange`, `getOuterDimsPerm`, `getIndexAttr`, `append`, and 6 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `applyPermToRange`, `getOuterDimsPerm`, `getIndexAttr`, `append`, and 6 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1479-1497
```cpp
    SmallVector<Value> tiledOperands = {sliceSource.getResult(), sliceDest};
    for (auto tile : unpackOp.getInnerTiles())
      tiledOperands.push_back(tile);

    Operation *tiledUnpackOp = UnPackOp::create(
        b, loc, TypeRange{sliceDest.getType()}, tiledOperands, op->getAttrs());

    if (isPerfectTilingCase)
      return TilingResult{{tiledUnpackOp},
                          SmallVector<Value>(tiledUnpackOp->getResults()),
                          generatedSlices};

    auto extractSlice = tensor::ExtractSliceOp::create(
        b, loc, tiledUnpackOp->getResult(0), resultOffsetsFromDest, sizes,
        destStrides);
    return TilingResult{
        {tiledUnpackOp}, {extractSlice.getResult()}, generatedSlices};
  }

```
- **EN**: Implements logic around `getResult`, `getInnerTiles`, `push_back`, `create`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getResult`, `getInnerTiles`, `push_back`, `create`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1498-1519
```cpp
  LogicalResult
  getResultTilePosition(Operation *op, OpBuilder &b, unsigned resultNumber,
                        ArrayRef<OpFoldResult> offsets,
                        ArrayRef<OpFoldResult> sizes,
                        SmallVector<OpFoldResult> &resultOffsets,
                        SmallVector<OpFoldResult> &resultSizes) const {
    resultOffsets = llvm::to_vector(offsets);
    resultSizes = llvm::to_vector(sizes);
    return success();
  }

  FailureOr<TilingResult>
  generateResultTileValue(Operation *op, OpBuilder &b, unsigned resultNumber,
                          ArrayRef<OpFoldResult> offsets,
                          ArrayRef<OpFoldResult> sizes) const {
    FailureOr<TilingResult> tilingResult =
        getTiledImplementation(op, b, offsets, sizes);
    if (failed(tilingResult))
      return failure();
    return tilingResult.value();
  }

```
- **EN**: Implements logic around `getResultTilePosition`, `to_vector`, `success`, `generateResultTileValue`, and 4 more symbols.
- **CN**: 围绕 `getResultTilePosition`, `to_vector`, `success`, `generateResultTileValue`, and 4 more symbols 实现具体逻辑。

### Lines 1520-1550
```cpp
  LogicalResult generateScalarImplementation(Operation *op, OpBuilder &builder,
                                             Location loc,
                                             ValueRange ivs) const {
    auto unpackOp = cast<UnPackOp>(op);
    assert(unpackOp.hasPureBufferSemantics() &&
           "expected operation to have buffer semantics");
    assert(ivs.size() == unpackOp.getDestRank() &&
           "number of ivs must match the rank of the output tensor");
    OpBuilder::InsertionGuard g(builder);

    DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
        unpackOp.getDimAndTileMapping();
    // Untiled loops and tile loops induction variables.
    SmallVector<Value> inputIvs;
    // Point loops induction variables.
    SmallVector<Value> inputIvsPointLoops;
    inputIvs.reserve(unpackOp.getDestRank());
    inputIvsPointLoops.reserve(dimAndTileMapping.size());
    for (auto dim : llvm::seq<int64_t>(0, unpackOp.getDestRank())) {
      if (dimAndTileMapping.count(dim)) {
        affine::DivModValue divMod =
            affine::getDivMod(builder, loc, ivs[dim],
                              getValueOrCreateConstantIndexOp(
                                  builder, loc, dimAndTileMapping[dim]));
        inputIvsPointLoops.push_back(divMod.remainder);
        inputIvs.push_back(divMod.quotient);
      } else {
        inputIvs.push_back(ivs[dim]);
      }
    }

```
- **EN**: Implements logic around `generateScalarImplementation`, `assert`, `g`, `getDimAndTileMapping`, and 6 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `generateScalarImplementation`, `assert`, `g`, `getDimAndTileMapping`, and 6 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1551-1568
```cpp
    // TODO: (lorenzo) simplify the logic a bit. There is `ivs`,
    // `inputIvsPointLoops` and `inputIvs`.
    assert(inputIvsPointLoops.size() + inputIvs.size() ==
               unpackOp.getSourceRank() &&
           "expect same number of induction variables equals to input rank");
    // Interchange the point loops induction variables based on `inner_dim_pos`.
    ArrayRef<int64_t> innerDims = unpackOp.getInnerDimsPos();
    SmallVector<int64_t> interchangeVector =
        computeInterchangeFromDimPos(innerDims, unpackOp.getDestRank());
    SmallVector<Value> interchangedInputIvsPointLoops = inputIvsPointLoops;
    interchangedInputIvsPointLoops = interchange<Value>(
        interchangedInputIvsPointLoops, interchangeVector, /*offset=*/0);
    // Interchange the tiled loops induction variables based on
    // `outer_dims_perm`.
    ArrayRef<int64_t> outerDims = unpackOp.getOuterDimsPerm();
    if (!outerDims.empty())
      inputIvs = interchange<Value>(inputIvs, outerDims, /*offset=*/0);

```
- **EN**: Implements logic around `assert`, `getSourceRank`, `getInnerDimsPos`, `computeInterchangeFromDimPos`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `assert`, `getSourceRank`, `getInnerDimsPos`, `computeInterchangeFromDimPos`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 1569-1592
```cpp
    llvm::append_range(inputIvs, interchangedInputIvsPointLoops);
    Value scalar =
        memref::LoadOp::create(builder, loc, unpackOp.getSource(), inputIvs);
    memref::StoreOp::create(builder, loc, scalar, unpackOp.getDest(), ivs);
    return success();
  }

  /// Method to return the position of iteration domain tile computed by the
  /// tiled operation.
  LogicalResult getIterationDomainTileFromOperandTiles(
      Operation *op, OpBuilder &b, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes,
      SmallVectorImpl<OpFoldResult> &resultOffsets,
      SmallVectorImpl<OpFoldResult> &resultSizes) const {
    if (operandNumbers.size() != 1) {
      LLVM_DEBUG({ llvm::dbgs() << "unable to handle multiple operands"; });
      return failure();
    }
    auto unPackOp = cast<UnPackOp>(op);
    unsigned operandNumber = operandNumbers[0];
    ArrayRef<OpFoldResult> offsets(allOffsets[0]);
    ArrayRef<OpFoldResult> sizes(allSizes[0]);

```
- **EN**: Implements logic around `append_range`, `create`, `success`, `getIterationDomainTileFromOperandTiles`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `append_range`, `create`, `success`, `getIterationDomainTileFromOperandTiles`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1593-1615
```cpp
    // If the operand tile is the dest, then no adjustment is needed.
    if (operandNumber == unPackOp.getDestMutable().getOperandNumber()) {
      resultOffsets = llvm::to_vector(offsets);
      resultSizes = llvm::to_vector(sizes);
      return success();
    }
    Location loc = unPackOp.getLoc();

    int64_t numTiles = unPackOp.getInnerDimsPos().size();
    auto destOffsets = offsets.drop_back(numTiles);
    auto destSizes = sizes.drop_back(numTiles);
    // The tiling is applied on interchanged dimensions. We have to undo the
    // interchange to map sizes and offsets to the original input.
    int64_t outputRank = unPackOp.getDestRank();
    ReifiedRankedShapedTypeDims reifiedReturnShapes;
    if (failed(reifyResultShapes(b, unPackOp, reifiedReturnShapes)))
      return failure();
    SmallVector<OpFoldResult> outputMixedSizes = reifiedReturnShapes.front();
    SmallVector<OpFoldResult> origOffsets(destOffsets);
    SmallVector<OpFoldResult> origSizes(destSizes);
    applyPermToRange(origOffsets, origSizes,
                     invertPermutationVector(unPackOp.getOuterDimsPerm()));

```
- **EN**: Implements logic around `getDestMutable`, `to_vector`, `success`, `getLoc`, and 10 more symbols.
- **CN**: 围绕 `getDestMutable`, `to_vector`, `success`, `getLoc`, and 10 more symbols 实现具体逻辑。

### Lines 1616-1644
```cpp
    DenseMap<int64_t, OpFoldResult> dimAndTileMapping =
        unPackOp.getDimAndTileMapping();

    for (auto dim : llvm::seq<int64_t>(0, outputRank)) {
      using AV = affine::AffineValueExpr;
      affine::AffineBuilder ab(b, loc);
      AffineExpr dim0, dim1, sym0;
      bindDims(b.getContext(), dim0, dim1);
      bindSymbols(b.getContext(), sym0);
      if (dimAndTileMapping.count(dim)) {
        // If the data dimension is tiled, the i-th index is the product of
        // offset_i and tile_i, and the i-th size is the product of sizes_i and
        // tile_i. The sizes must be clamped to the sizes of the unpack result.
        auto avOffset = AV(dim0).bind(origOffsets[dim]);
        auto avSize = AV(dim0).bind(origSizes[dim]);
        auto avTileSize = AV(sym0).bind(dimAndTileMapping[dim]);
        auto avResultSize = AV(dim0).bind(outputMixedSizes[dim]);
        resultOffsets.push_back(ab.mul(avOffset, avTileSize));
        auto avResultOffset = AV(dim1).bind(resultOffsets.back());
        resultSizes.push_back(ab.min({ab.mul(avSize, avTileSize),
                                      ab.sub(avResultSize, avResultOffset)}));
      } else {
        resultOffsets.push_back(origOffsets[dim]);
        resultSizes.push_back(origSizes[dim]);
      }
    }
    return success();
  }

```
- **EN**: Implements logic around `getDimAndTileMapping`, `seq`, `ab`, `bindDims`, and 6 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDimAndTileMapping`, `seq`, `ab`, `bindDims`, and 6 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1645-1670
```cpp
  /// Method to return the tiled implementation of tensor.unpack as a consumer.
  FailureOr<TilingResult> getTiledImplementationFromOperandTiles(
      Operation *op, OpBuilder &b, ArrayRef<unsigned> operandNumbers,
      ArrayRef<SmallVector<OpFoldResult>> allOffsets,
      ArrayRef<SmallVector<OpFoldResult>> allSizes) const {
    if (operandNumbers.size() != 1 || operandNumbers[0] != 0) {
      LLVM_DEBUG({ llvm::dbgs() << "unhandled operands for consumer fusion"; });
      return failure();
    }
    auto unPackOp = cast<UnPackOp>(op);
    // TODO: Support Memref UnPackOp. Temporarily return failure.
    if (!unPackOp.hasPureTensorSemantics())
      return failure();

    ArrayRef<OpFoldResult> offsets(allOffsets[0]);
    ArrayRef<OpFoldResult> sizes(allSizes[0]);

    // tensor.unpack op is fusible (as a consumer) only if inner dims are not
    // tiled.
    int64_t numTiles = unPackOp.getInnerDimsPos().size();
    for (auto iter :
         llvm::zip_equal(unPackOp.getMixedTiles(), sizes.take_back(numTiles))) {
      if (!isEqualConstantIntOrValue(std::get<0>(iter), std::get<1>(iter)))
        return failure();
    }

```
- **EN**: Implements logic around `getTiledImplementationFromOperandTiles`, `size`, `dbgs`, `failure`, and 6 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getTiledImplementationFromOperandTiles`, `size`, `dbgs`, `failure`, and 6 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1671-1690
```cpp
    Location loc = unPackOp.getLoc();

    // Fetch offset/size for creating the slice of the dest operand of
    // unpack op.
    SmallVector<OpFoldResult> outputOffsets, outputSizes;
    if (failed(getIterationDomainTileFromOperandTiles(
            op, b, operandNumbers, allOffsets, allSizes, outputOffsets,
            outputSizes)))
      return failure();

    auto oneAttr = b.getI64IntegerAttr(1);
    int64_t outputRank = unPackOp.getDestRank();
    SmallVector<OpFoldResult> strides(outputRank, oneAttr);

    SmallVector<Value> tiledOperands;
    // Create slice of the dest operand.
    auto extractDestSlice = tensor::ExtractSliceOp::create(
        b, loc, unPackOp.getDest(), outputOffsets, outputSizes, strides);
    tiledOperands.push_back(extractDestSlice);

```
- **EN**: Implements logic around `getLoc`, `failed`, `failure`, `getI64IntegerAttr`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `failed`, `failure`, `getI64IntegerAttr`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1691-1710
```cpp
    strides.append(unPackOp.getSourceRank() - outputRank, oneAttr);
    // Create slice of the source operand.
    auto extractSourceSlice = tensor::ExtractSliceOp::create(
        b, loc, unPackOp.getSource(), offsets, sizes, strides);
    tiledOperands.insert(tiledOperands.begin(), extractSourceSlice);
    for (auto tile : unPackOp.getInnerTiles())
      tiledOperands.push_back(tile);

    // Create tiled unpack op.
    Operation *tiledUnPackOp =
        UnPackOp::create(b, loc, TypeRange{extractDestSlice.getType()},
                         tiledOperands, op->getAttrs());

    return TilingResult{{tiledUnPackOp},
                        SmallVector<Value>(tiledUnPackOp->getResults()),
                        llvm::to_vector(ArrayRef<Operation *>{
                            extractSourceSlice, extractDestSlice})};
  }
};

```
- **EN**: Implements logic around `append`, `create`, `getSource`, `insert`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `append`, `create`, `getSource`, `insert`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1711-1739
```cpp
} // namespace

template <typename OpType>
static void registerOne(MLIRContext *ctx) {
  OpType::template attachInterface<LinalgOpTilingInterface<OpType>>(*ctx);
  OpType::template attachInterface<LinalgOpPartialReductionInterface<OpType>>(
      *ctx);
}

/// Variadic helper function.
template <typename... OpTypes>
static void registerAll(MLIRContext *ctx) {
  (registerOne<OpTypes>(ctx), ...);
}

#define GET_OP_LIST

void mlir::linalg::registerTilingInterfaceExternalModels(
    DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, linalg::LinalgDialect *dialect) {
    registerOne<linalg::GenericOp>(ctx);
    linalg::PackOp::attachInterface<PackOpTiling>(*ctx);
    linalg::UnPackOp::attachInterface<UnPackOpTiling>(*ctx);
    registerAll<
#include "mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc"
        >(ctx);
  });
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`。

### Lines 1740-1746
```cpp
void mlir::linalg::registerTilingInterfaceExternalModelsForPackUnPackOps(
    DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, LinalgDialect *dialect) {
    linalg::PackOp::attachInterface<PackOpTiling>(*ctx);
    linalg::UnPackOp::attachInterface<UnPackOpTiling>(*ctx);
  });
}
```
- **EN**: Implements logic around `registerTilingInterfaceExternalModelsForPackUnPackOps`, `addExtension`, `attachInterface`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `registerTilingInterfaceExternalModelsForPackUnPackOps`, `addExtension`, `attachInterface` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Operation verification / 操作验证**:
  - **EN**: Checks structural and semantic invariants so malformed IR is rejected early.
  - **CN**: 检查结构与语义不变式，以便尽早拒绝非法 IR。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/TilingInterfaceImpl.h`, `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/SCF/IR/SCF.h` ... (+10 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (14), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`
