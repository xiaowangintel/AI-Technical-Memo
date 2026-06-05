# DataLayoutPropagation.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/DataLayoutPropagation.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `DataLayoutPropagation`.
  - **CN**: 实现 Linalg 方言中围绕 `DataLayoutPropagation` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
//===- DataLayoutPropagation.cpp -----------------------------------------===///
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/UB/IR/UBOps.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/TypeUtilities.h"
#include "llvm/ADT/SetOperations.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/Debug.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`。

### Lines 24-42
```cpp
namespace mlir {
#define GEN_PASS_DEF_LINALGDATALAYOUTPROPAGATION
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::linalg;

#define DEBUG_TYPE "linalg-data-layout-propagation"

namespace {

static bool hasGatherSemantics(linalg::GenericOp genericOp) {
  for (Operation &op : genericOp.getBody()->getOperations())
    if (isa<tensor::ExtractOp, linalg::IndexOp>(op))
      return true;
  return false;
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 43-66
```cpp
// The struct contains the infomation about mapping packing information to
// the iteration domain of Linalg ops.
struct PackInfo {
  int64_t getNumTiledLoops() const { return tileToPointMapping.size(); };
  // InnerDimsPos on iteration domain, which follows the order in pack ops.
  SmallVector<int64_t> tiledDimsPos;
  // The sizes of tiling data dimensions on iteration domain.
  llvm::DenseMap<int64_t, OpFoldResult> domainDimAndTileMapping;
  // The mapping from a dimension of iteration domain to the corresponding inner
  // tiling dimension on iteration domain.
  llvm::DenseMap<int64_t, int64_t> tileToPointMapping;
  // The permutation of outer dims (on domain).
  SmallVector<int64_t> outerDimsOnDomainPerm;
};

template <typename OpTy>
static FailureOr<PackInfo>
getPackingInfoFromOperand(OpOperand *opOperand, linalg::GenericOp genericOp,
                          OpTy packOrUnPackOp) {
  static_assert(llvm::is_one_of<OpTy, linalg::PackOp, linalg::UnPackOp>::value,
                "applies to only pack or unpack operations");
  LLVM_DEBUG(
      { llvm::dbgs() << "--- Construct PackInfo From an operand ---\n"; });

```
- **EN**: Introduces declarations for `contains`, `PackInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `contains`, `PackInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 67-97
```cpp
  AffineMap indexingMap = genericOp.getMatchingIndexingMap(opOperand);
  SmallVector<AffineMap> indexingMaps = genericOp.getIndexingMapsArray();
  SmallVector<utils::IteratorType> iterators =
      genericOp.getIteratorTypesArray();

  PackInfo packInfo;
  int64_t origNumDims = indexingMap.getNumDims();
  SmallVector<AffineExpr> exprs(indexingMap.getResults());
  ArrayRef<int64_t> innerDimsPos = packOrUnPackOp.getInnerDimsPos();
  for (auto [index, innerDimPos, tileSize] :
       llvm::zip_equal(llvm::seq<unsigned>(0, innerDimsPos.size()),
                       innerDimsPos, packOrUnPackOp.getMixedTiles())) {
    auto expr = exprs[innerDimPos];
    if (!isa<AffineDimExpr>(expr))
      return failure();
    int64_t domainDimPos =
        cast<AffineDimExpr>(exprs[innerDimPos]).getPosition();
    if (!isParallelIterator(iterators[domainDimPos]))
      return failure();
    packInfo.tiledDimsPos.push_back(domainDimPos);
    packInfo.domainDimAndTileMapping[domainDimPos] = tileSize;
    packInfo.tileToPointMapping[domainDimPos] = origNumDims + index;
    LLVM_DEBUG({
      llvm::dbgs() << "map innerDimPos=" << innerDimPos
                   << " to iteration dimension (d" << domainDimPos << ", d"
                   << packInfo.tileToPointMapping[domainDimPos]
                   << "), which has size=("
                   << packInfo.domainDimAndTileMapping[domainDimPos] << ")\n";
    });
  }

```
- **EN**: Implements logic around `getMatchingIndexingMap`, `getIndexingMapsArray`, `getIteratorTypesArray`, `getNumDims`, and 10 more symbols.
- **CN**: 围绕 `getMatchingIndexingMap`, `getIndexingMapsArray`, `getIteratorTypesArray`, `getNumDims`, and 10 more symbols 实现具体逻辑。

### Lines 98-133
```cpp
  // Bail out if a tiled dimension is present in a map but not as an affine dim
  // expression.
  auto areAllAffineDimExpr = [&](int dim) {
    for (AffineMap map : indexingMaps) {
      if (llvm::any_of(map.getResults(), [dim](AffineExpr expr) {
            return expr.isFunctionOfDim(dim) && !isa<AffineDimExpr>(expr);
          })) {
        return false;
      }
    }
    return true;
  };
  for (int64_t i : packInfo.tiledDimsPos)
    if (!areAllAffineDimExpr(i))
      return failure();

  // Get the outer dims perm on the iteration domain. Start by identifying the
  // set of domain dims affected by the outer permutation along with the
  // permuted ordering for those dims. Then the full outer dims permutation can
  // be constructed by replacing the affected dims with the permuted result in a
  // numLoops-rank identity. e.g.
  //   outerDimsPerm = [1, 2, 0]
  //   indexingMap = (d0, d1, d2, d3, d4) -> (d1, d4, d3)
  //
  //   permutedOuterDims =        [4,    3, 1]
  //   outerDimsOnDomainPerm = [0, 4, 2, 3, 1]
  //
  // Non-affine dim expressions must not be permuted by the outer dims
  // permutation.
  SmallVector<int64_t> permutedOuterDims;
  for (auto [index, dim] : llvm::enumerate(packOrUnPackOp.getOuterDimsPerm())) {
    auto permutedExpr = indexingMap.getResult(dim);
    if (auto dimExpr = dyn_cast<AffineDimExpr>(permutedExpr)) {
      permutedOuterDims.push_back(dimExpr.getPosition());
      continue;
    }
```
- **EN**: Implements logic around `any_of`, `isFunctionOfDim`, `areAllAffineDimExpr`, `failure`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `any_of`, `isFunctionOfDim`, `areAllAffineDimExpr`, `failure`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 134-156
```cpp

    // TODO: Allow propagation with transposes on non affine dim expressions,
    // e.g. d0 + d1 which implies transposing both dims simultaneously while
    // maintaining the relative position between them.
    if (static_cast<int64_t>(index) != dim)
      return failure();
  }
  if (!permutedOuterDims.empty()) {
    int64_t outerDimIndex = 0;
    llvm::DenseSet<int64_t> permutedDomainDims(permutedOuterDims.begin(),
                                               permutedOuterDims.end());
    for (int i = 0, e = indexingMap.getNumDims(); i < e; i++)
      packInfo.outerDimsOnDomainPerm.push_back(
          permutedDomainDims.contains(i) ? permutedOuterDims[outerDimIndex++]
                                         : i);
    LLVM_DEBUG({
      llvm::dbgs() << "map outer dimsDimsPerm to ";
      for (auto dim : packInfo.outerDimsOnDomainPerm)
        llvm::dbgs() << dim << " ";
      llvm::dbgs() << "\n";
    });
  }

```
- **EN**: Implements logic around `static_cast`, `failure`, `empty`, `permutedDomainDims`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `static_cast`, `failure`, `empty`, `permutedDomainDims`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 157-192
```cpp
  return packInfo;
}

static SmallVector<int64_t> computeOuterDims(ArrayRef<int64_t> perm,
                                             ArrayRef<AffineExpr> exprs) {
  // Compute `outer_dims_perm`. See example:
  // current exprs      : (d0, d1, d2, d3) -> (d2, d3)
  // perm               : [0, 3, 1, 2]
  // First map d2, d3 with their position in the array as:
  // currentPositionTileLoops: dim | pos
  //                           d2  | 0
  //                           d3  | 1
  // then scan `perm` in order and get the `outer_dims_perm`
  // to be used, here it would be [1, 0].
  assert(!perm.empty() && "expect perm not to be empty");
  assert(!exprs.empty() && "expect exprs not to be empty");
  if (exprs.size() == 1)
    return {};
  SmallVector<int64_t> outerDimsPerm;
  DenseMap<int64_t, int64_t> currentPositionTileLoops;
  for (auto [pos, expr] : llvm::enumerate(exprs)) {
    // Here we rely on the assumption that the outer dims permutation
    // when propagating currently requires that non-affine dim expressions
    // are not permuted, thus allowing the identity assignment below.
    if (auto dimExpr = dyn_cast<AffineDimExpr>(expr))
      currentPositionTileLoops[dimExpr.getPosition()] = pos;
    else
      currentPositionTileLoops[pos] = pos;
  }
  for (int64_t loopIdx : perm) {
    if (currentPositionTileLoops.count(loopIdx))
      outerDimsPerm.push_back(currentPositionTileLoops.lookup(loopIdx));
  }
  return outerDimsPerm;
}

```
- **EN**: Implements logic around `computeOuterDims`, `assert`, `size`, `enumerate`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `computeOuterDims`, `assert`, `size`, `enumerate`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 193-213
```cpp
struct PackedOperandDetails {
  SmallVector<OpFoldResult> innerTileSizes;
  SmallVector<int64_t> innerDimsPos;
  SmallVector<int64_t> outerDimsPerm;
  AffineMap indexingMap;
};

/// Helper function for getOrCreatePackedViewOfOperand that populates
/// the details of the packedOperand that needs to be formed and also
/// returns if the packing would require padding.
static bool getPackedOperandDetails(
    OpBuilder &b, PackInfo packInfo, GenericOp genericOp, OpOperand *opOperand,
    DenseMap<OpOperand *, PackedOperandDetails> &packedOperandMap) {
  PackedOperandDetails currOperandDetails;
  int64_t numOrigLoops = genericOp.getNumLoops();
  int64_t numInnerLoops = packInfo.getNumTiledLoops();
  int64_t numLoops = numOrigLoops + numInnerLoops;
  AffineMap origIndexingMap = genericOp.getMatchingIndexingMap(opOperand);
  llvm::DenseMap<int64_t, int64_t> domainDimToOperandDim;
  SmallVector<AffineExpr> exprs(origIndexingMap.getResults());

```
- **EN**: Introduces declarations for `PackedOperandDetails`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PackedOperandDetails` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 214-241
```cpp
  // If the OpOperand is a scalar or a zero-rank tensor, no need to pack.
  if (genericOp.isScalar(opOperand) || exprs.empty()) {
    currOperandDetails.indexingMap =
        AffineMap::get(numLoops, 0, exprs, b.getContext());
    packedOperandMap[opOperand] = currOperandDetails;
    return false;
  }

  // Step 1. Construct the information of packing data dimensions; append inner
  // dimensions to the indexing maps for the operand.
  for (auto [index, expr] : llvm::enumerate(exprs)) {
    if (auto dimExpr = dyn_cast<AffineDimExpr>(expr)) {
      int64_t dimPos = dimExpr.getPosition();
      domainDimToOperandDim[dimPos] = index;
      continue;
    }
  }
  SmallVector<int64_t> innerDimsPos;
  SmallVector<OpFoldResult> innerTileSizes;
  for (auto dimPos : packInfo.tiledDimsPos) {
    if (!domainDimToOperandDim.count(dimPos))
      continue;
    int64_t index = domainDimToOperandDim[dimPos];
    innerTileSizes.push_back(packInfo.domainDimAndTileMapping[dimPos]);
    innerDimsPos.push_back(index);
    exprs.push_back(b.getAffineDimExpr(packInfo.tileToPointMapping[dimPos]));
  }

```
- **EN**: Implements logic around `isScalar`, `get`, `enumerate`, `getPosition`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isScalar`, `get`, `enumerate`, `getPosition`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 242-270
```cpp
  // Step 2. Handle outer dim permutations.
  SmallVector<int64_t> outerDimsPerm;
  if (!packInfo.outerDimsOnDomainPerm.empty()) {
    outerDimsPerm = computeOuterDims(packInfo.outerDimsOnDomainPerm, exprs);

    // Step 2.1: Fold transpose into the linalg.generic.
    SmallVector<int64_t> inversedOuterPerm =
        invertPermutationVector(packInfo.outerDimsOnDomainPerm);
    for (auto i : llvm::seq<unsigned>(0, origIndexingMap.getNumResults())) {
      if (auto dimExpr = dyn_cast<AffineDimExpr>(exprs[i])) {
        int64_t dimPos = dimExpr.getPosition();
        exprs[i] = b.getAffineDimExpr(inversedOuterPerm[dimPos]);
        continue;
      }
      assert(isa<AffineConstantExpr>(exprs[i]) &&
             "Attempted to permute non-constant and non-affine dim expression");
    }
    // Step 2.2: Undo the transposition on `exprs` and propagate the
    // transposition on the pack using outerDimsPerm.
    if (!outerDimsPerm.empty()) {
      SmallVector<AffineExpr> auxVec = exprs;
      for (const auto &en : enumerate(outerDimsPerm))
        auxVec[en.index()] = exprs[en.value()];
      exprs = auxVec;
    }
  }
  currOperandDetails.indexingMap =
      AffineMap::get(numLoops, 0, exprs, b.getContext());

```
- **EN**: Implements logic around `empty`, `computeOuterDims`, `invertPermutationVector`, `seq`, and 6 more symbols; this block performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `empty`, `computeOuterDims`, `invertPermutationVector`, `seq`, and 6 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 271-293
```cpp
  // The operand does not have dimensions that relates to pack op.
  if (innerDimsPos.empty() && outerDimsPerm.empty()) {
    packedOperandMap[opOperand] = currOperandDetails;
    return false;
  }
  auto inputType = cast<RankedTensorType>(opOperand->get().getType());

  auto maybeIntInnerTileSizes =
      llvm::map_to_vector(innerTileSizes, [](OpFoldResult ofr) -> int64_t {
        std::optional<int64_t> maybeCst = getConstantIntValue(ofr);
        return maybeCst.value_or(ShapedType::kDynamic);
      });
  bool requirePadding = linalg::PackOp::requirePaddingValueStrict(
      inputType.getShape(), innerDimsPos,
      linalg::PackOp::inferPackedTensorType(inputType, maybeIntInnerTileSizes,
                                            innerDimsPos, outerDimsPerm)
          .getShape(),
      outerDimsPerm, innerTileSizes);
  currOperandDetails.innerDimsPos = innerDimsPos;
  currOperandDetails.innerTileSizes = innerTileSizes;
  currOperandDetails.outerDimsPerm = outerDimsPerm;
  packedOperandMap[opOperand] = currOperandDetails;

```
- **EN**: Implements logic around `empty`, `get`, `map_to_vector`, `getConstantIntValue`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `empty`, `get`, `map_to_vector`, `getConstantIntValue`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 294-311
```cpp
  return requirePadding;
}

/// Returns a tuple for packed operand and indexing_map with the assumptions:
///   1) The generic op is the producer of the pack op.
///   2) The generic op has only one result.
/// If the operand is a scalar or packing dimensions are all irrelevant to the
/// operand, the operand and the updated indexing map will be returned.
/// Otherwise, it returns the packed operand and the updated indexing map. E.g.,
///
///   #map0 = affine_map<(d0, d1) -> (d0, d1)>
///   #map1 = affine_map<(d0, d1) -> (d0)>
///   #map2 = affine_map<(d0, d1) -> (d1)>
///   %0 = linalg.generic {indexing_maps = [#map1, #map2, #map0],
///                        iterator_types = ["parallel", "parallel"]}
///      ins(%arg0, %arg1 : tensor<?xf32>, tensor<?xf32>)
///      outs(%init : tensor<?x?xf32>) {
///    ^bb0(%arg3: f32, %arg4: f32, %arg5: f32):
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 312-339
```cpp
///      %4 = arith.addf %arg3, %arg4 : f32
///      linalg.yield %4 : f32
///  } -> tensor<?x?xf32>
///  %1 = linalg.pack %0
///    inner_dims_pos = [0, 1]
///    inner_tiles = [8, 2]
///    into %dest : tensor<?x?xf32> -> tensor<?x?x8x2xf32>
///
///  Taking the first input operand as an example, the inner tile size of d1 is
///  8. Thus, the below operation and `affine_map<(d0, d1, d2, d3)> ->
///  affine_map<(d1, d3)>` will be returned.
///
///  %pack = linalg.pack %arg0
///    inner_dims_pos = [0]
///    inner_tiles = [8]
///    into %init : tensor<?xf32> -> tensor<?x8xf32>
static std::tuple<Value, AffineMap> getOrCreatePackedViewOfOperand(
    OpBuilder &b, Location loc, OpOperand *opOperand,
    const DenseMap<OpOperand *, PackedOperandDetails> &packedOperandMap) {
  assert(packedOperandMap.contains(opOperand) &&
         "packed operand details expected to be populated");
  auto currOperandDetails = packedOperandMap.at(opOperand);
  auto innerDimsPos = currOperandDetails.innerDimsPos;
  auto outerDimsPerm = currOperandDetails.outerDimsPerm;
  auto innerTileSizes = currOperandDetails.innerTileSizes;
  if (innerDimsPos.empty() && outerDimsPerm.empty())
    return std::make_tuple(opOperand->get(), currOperandDetails.indexingMap);

```
- **EN**: Implements logic around `getOrCreatePackedViewOfOperand`, `assert`, `at`, `empty`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOrCreatePackedViewOfOperand`, `assert`, `at`, `empty`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 340-375
```cpp
  auto empty = linalg::PackOp::createDestinationTensor(
      b, loc, opOperand->get(), innerTileSizes, innerDimsPos, outerDimsPerm);
  auto poison = ub::PoisonOp::create(
      b, loc, getElementTypeOrSelf(opOperand->get().getType()));
  PackOp packedOperand =
      linalg::PackOp::create(b, loc, opOperand->get(), empty, innerDimsPos,
                             innerTileSizes, poison, outerDimsPerm);
  return std::make_tuple(packedOperand.getResult(),
                         currOperandDetails.indexingMap);
}

/// This function is a helper subroutine to pack a genericOp and return it. It
/// will create a new generic op with the packed operand and the packed output
/// according to packInfo when we attempt to push down unpack or bubble up pack
/// around it. Implicitly this will only work when a packInfo can be obtained.
/// This make sure that we are only using this function on parallel permuted
/// dimensions.
static FailureOr<GenericOp>
packGenericOp(RewriterBase &rewriter, GenericOp genericOp, Value dest,
              AffineMap packedOutIndexingMap, const PackInfo &packInfo,
              bool isFoldableUnpackPack, bool poisonPaddingOk) {
  Location loc = genericOp.getLoc();
  SmallVector<Value> inputOperands;
  SmallVector<Value> inputOperandsFromUnpackedSource;
  SmallVector<AffineMap> indexingMaps;
  auto hasEquivalentTiles = [](PackOp packOp, UnPackOp unPackOp) {
    return packOp.getOuterDimsPerm() == unPackOp.getOuterDimsPerm() &&
           packOp.getInnerDimsPos() == unPackOp.getInnerDimsPos() &&
           llvm::equal(packOp.getMixedTiles(), unPackOp.getMixedTiles());
  };
  DenseMap<OpOperand *, PackedOperandDetails> packedOperandMap;
  bool requiresPadding = false;
  for (OpOperand *inputOperand : genericOp.getDpsInputOperands()) {
    requiresPadding |= getPackedOperandDetails(rewriter, packInfo, genericOp,
                                               inputOperand, packedOperandMap);
  }
```
- **EN**: Implements logic around `createDestinationTensor`, `get`, `create`, `getElementTypeOrSelf`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createDestinationTensor`, `get`, `create`, `getElementTypeOrSelf`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 376-404
```cpp
  if (requiresPadding && !poisonPaddingOk)
    return failure();

  for (OpOperand *inputOperand : genericOp.getDpsInputOperands()) {
    auto [packedOperand, packedIndexingMap] = getOrCreatePackedViewOfOperand(
        rewriter, loc, inputOperand, packedOperandMap);
    auto unpackOp = inputOperand->get().getDefiningOp<linalg::UnPackOp>();
    auto packOp = packedOperand.getDefiningOp<linalg::PackOp>();
    if (packOp && unpackOp && hasEquivalentTiles(packOp, unpackOp)) {
      inputOperandsFromUnpackedSource.push_back(unpackOp.getSource());
    } else {
      inputOperandsFromUnpackedSource.push_back(packedOperand);
    }
    inputOperands.push_back(packedOperand);
    indexingMaps.push_back(packedIndexingMap);
  }

  // If the unpack->pack sequences can be folded, replace use the sources of
  // the unpack ops in any unpack->pack chains on the generic op operands.
  if (isFoldableUnpackPack) {
    inputOperands = inputOperandsFromUnpackedSource;
    if (auto destPack = dest.getDefiningOp<linalg::PackOp>()) {
      auto destUnPack = destPack.getSource().getDefiningOp<linalg::UnPackOp>();
      if (destUnPack && hasEquivalentTiles(destPack, destUnPack)) {
        dest = destUnPack.getSource();
      }
    }
  }

```
- **EN**: Implements logic around `failure`, `getDpsInputOperands`, `getOrCreatePackedViewOfOperand`, `get`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `getDpsInputOperands`, `getOrCreatePackedViewOfOperand`, `get`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 405-426
```cpp
  int64_t numInnerLoops = packInfo.getNumTiledLoops();
  SmallVector<utils::IteratorType> iterTypes =
      genericOp.getIteratorTypesArray();
  iterTypes.append(numInnerLoops, utils::IteratorType::parallel);

  indexingMaps.push_back(packedOutIndexingMap);

  auto newGenericOp = linalg::GenericOp::create(
      rewriter, loc, dest.getType(), inputOperands, dest, indexingMaps,
      iterTypes,
      /*bodyBuild=*/nullptr, linalg::getPrunedAttributeList(genericOp));
  rewriter.cloneRegionBefore(genericOp.getRegion(), newGenericOp.getRegion(),
                             newGenericOp.getRegion().begin());
  return newGenericOp;
}

static bool isGenericOutsNotUsed(linalg::GenericOp genericOp) {
  return llvm::all_of(genericOp.getDpsInitsMutable(), [&](OpOperand &operand) {
    return genericOp.getMatchingBlockArgument(&operand).use_empty();
  });
}

```
- **EN**: Implements logic around `getNumTiledLoops`, `getIteratorTypesArray`, `append`, `push_back`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumTiledLoops`, `getIteratorTypesArray`, `append`, `push_back`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 427-444
```cpp
/// Bubbles up linalg.pack op through a producer generic op. This
/// swap pack(generic) to generic(pack). The new generic op works on packed
/// domain; pack ops are created for input and output operands. E.g.,
///
///     #map0 = affine_map<(d0, d1) -> (d0, d1)>
///     %0 = tensor.dim %arg0, %c0 : tensor<?x?xf32>
///     %1 = tensor.dim %arg0, %c1 : tensor<?x?xf32>
///     %2 = tensor.empty(%0, %1) : tensor<?x?xf32>
///     %3 = linalg.generic {indexing_maps = [#map0, #map0],
///                          iterator_types = ["parallel", "parallel"]}
///         ins(%arg0 : tensor<?x?xf32>)
///         outs(%2 : tensor<?x?xf32>) {
///       ^bb0(%arg3: f32, %arg4: f32):
///         %4 = arith.addf %arg3, %arg3 : f32
///         linalg.yield %4 : f32
///     } -> tensor<?x?xf32>
///     %4 = linalg.pack %3
///       inner_dims_pos = [0, 1]
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 445-462
```cpp
///       inner_tiles = [8, 2]
///       into %dest : tensor<?x?xf32> -> tensor<?x?x8x2xf32>
///
/// will be converted to
///
///     #map = affine_map<()[s0] -> (s0 ceildiv 8)>
///     #map1 = affine_map<()[s0] -> (s0 ceildiv 2)>
///     #map2 = affine_map<(d0, d1, d2, d3) -> (d0, d1, d2, d3)>
///     %dim = tensor.dim %arg0, %c0 : tensor<?x?xf32>
///     %dim_0 = tensor.dim %arg0, %c1 : tensor<?x?xf32>
///     %0 = affine.apply #map()[%dim]
///     %1 = affine.apply #map1()[%dim_0]
///     %2 = tensor.empty(%0, %1) : tensor<?x?x8x2xf32>
///     %pack = linalg.pack %arg0
///       inner_dims_pos = [0, 1]
///       inner_tiles = [8, 2]
///       into %2 : tensor<?x?xf32> -> tensor<?x?x8x2xf32>
///     %3 = linalg.generic {indexing_maps = [#map2, #map2],
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 463-481
```cpp
///       iterator_types = ["parallel", "parallel", "parallel", "parallel"]}
///       ins(%pack : tensor<?x?x8x2xf32>)
///       outs(%arg1 : tensor<?x?x8x2xf32>) {
///     ^bb0(%in: f32, %out: f32):
///       %4 = arith.addf %in, %in : f32
///       linalg.yield %4 : f32
///     } -> tensor<?x?x8x2xf32>
static FailureOr<GenericOp>
bubbleUpPackOpThroughGenericOp(RewriterBase &rewriter, linalg::PackOp packOp,
                               const ControlPropagationFn &controlFn,
                               bool poisonPaddingOk) {
  auto genericOp = packOp.getSource().getDefiningOp<GenericOp>();
  if (!genericOp)
    return failure();

  // User controlled propagation function.
  if (!controlFn(&packOp.getSourceMutable()))
    return failure();

```
- **EN**: Implements logic around `bubbleUpPackOpThroughGenericOp`, `getSource`, `failure`, `controlFn`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bubbleUpPackOpThroughGenericOp`, `getSource`, `failure`, `controlFn` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 482-499
```cpp
  // TODO: Enable propagation in the presence of linalg.index and
  // tensor.extract, likely as a separate pattern as the pack information and
  // propagation decision needs to be inferred from the region of the generic.
  if (hasGatherSemantics(genericOp))
    return failure();

  // TODO: Relax the restriction. We are able to bubble up the pack op through
  // multi-result generic op. It just needs more work.
  if (genericOp.getNumResults() != 1)
    return failure();

  // Bail-out if the result of the generic has multiple uses, as bubbling up
  // creates recomputation if the generic has multiple users.
  // TODO: Enable the case where every use is an identical pack op as no
  // recomputation is needed in that case.
  if (!genericOp->getResult(0).hasOneUse())
    return failure();

```
- **EN**: Implements logic around `hasGatherSemantics`, `failure`, `getNumResults`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasGatherSemantics`, `failure`, `getNumResults`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 500-534
```cpp
  // TODO: Add an option for allowing padding values. It could introduce
  // undefined behavior if we unconditionally propagate pack op through all
  // the ops. E.g., if the padding value is zero and there are division ops in
  // a generic op. Some values of padding area could be NaN (0/0).
  if (packOp.getPaddingValue())
    return failure();

  OpOperand *opOperand = genericOp.getDpsInitOperand(0);
  auto packInfo = getPackingInfoFromOperand(opOperand, genericOp, packOp);
  if (failed(packInfo))
    return failure();

  // We want to move the pack not the generic.
  OpBuilder::InsertionGuard guard(rewriter);
  rewriter.setInsertionPoint(genericOp);

  // We need to handle two cases:
  // 1) The linalg.pack destination is a tensor.empty. If this is the case, we
  // create a new tensor.empty to avoid breaking dominance, as we are moving the
  // linalg.pack above the linalg.generic.
  // 2) The destination is not a tensor.empty. In this case we can replace only
  // if the destination of the linalg.pack dominates the linalg.generic.
  Value packOpDest = packOp.getDest();
  if (!packOpDest.hasOneUse())
    return failure();
  if (auto emptyOp = packOpDest.getDefiningOp<tensor::EmptyOp>()) {
    packOpDest = tensor::EmptyOp::create(rewriter, genericOp->getLoc(),
                                         emptyOp.getMixedSizes(),
                                         emptyOp.getType().getElementType());
  } else {
    DominanceInfo dom(genericOp);
    if (!dom.properlyDominates(packOpDest, genericOp))
      return failure();
  }

```
- **EN**: Implements logic around `getPaddingValue`, `failure`, `getDpsInitOperand`, `getPackingInfoFromOperand`, and 11 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPaddingValue`, `failure`, `getDpsInitOperand`, `getPackingInfoFromOperand`, and 11 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 535-562
```cpp
  // Rebuild the indexing map for the corresponding init operand.
  DenseMap<OpOperand *, PackedOperandDetails> packedOperandMap;
  bool requiresPadding = getPackedOperandDetails(rewriter, *packInfo, genericOp,
                                                 opOperand, packedOperandMap);
  if (requiresPadding && !poisonPaddingOk)
    return failure();

  auto [packedOutOperand, packedOutIndexingMap] =
      getOrCreatePackedViewOfOperand(rewriter, genericOp.getLoc(), opOperand,
                                     packedOperandMap);
  // Forward the new tensor.empty as a destination if it is one of the following
  // situations:
  // 1) The dps init operand is a tensor.empty.
  // 2) The dps init is a write-only operand, i.e., it is not used in the
  // genericOp
  Value dest = packedOutOperand;
  auto initTensor =
      genericOp.getDpsInitOperand(0)->get().getDefiningOp<tensor::EmptyOp>();
  if (initTensor || isGenericOutsNotUsed(genericOp)) {
    dest = packOpDest;
  }
  // pack(unpack) isn't naively foldable because the unpack op can be from
  // an arbitrary domain so we need to keep both.
  return packGenericOp(rewriter, genericOp, dest, packedOutIndexingMap,
                       *packInfo, /*isFoldableUnpackPack=*/false,
                       poisonPaddingOk);
}

```
- **EN**: Implements logic around `getPackedOperandDetails`, `failure`, `getOrCreatePackedViewOfOperand`, `getDpsInitOperand`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPackedOperandDetails`, `failure`, `getOrCreatePackedViewOfOperand`, `getDpsInitOperand`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 563-585
```cpp
/// Wrapper pattern that applies bubbleUpPackOpThroughGenericOp method.
struct BubbleUpPackOpThroughGenericOpPattern
    : public OpRewritePattern<linalg::PackOp> {
public:
  BubbleUpPackOpThroughGenericOpPattern(MLIRContext *context,
                                        ControlPropagationFn fun,
                                        bool poisonPaddingOk)
      : OpRewritePattern<linalg::PackOp>(context), controlFn(std::move(fun)),
        poisonPaddingOk(std::move(poisonPaddingOk)) {}

  LogicalResult matchAndRewrite(linalg::PackOp packOp,
                                PatternRewriter &rewriter) const override {
    if (!packOp.hasPureTensorSemantics())
      return failure();

    auto genericOp = bubbleUpPackOpThroughGenericOp(rewriter, packOp, controlFn,
                                                    poisonPaddingOk);
    if (failed(genericOp))
      return failure();
    rewriter.replaceOp(packOp, genericOp->getResults());
    return success();
  }

```
- **EN**: Introduces declarations for `BubbleUpPackOpThroughGenericOpPattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `BubbleUpPackOpThroughGenericOpPattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 586-603
```cpp
private:
  ControlPropagationFn controlFn;
  bool poisonPaddingOk;
};

/// Propagate a linalg.pack operation up through a tensor.pad. The idea is to
/// add as many zero padding dimensions in `high` and `low` based on the number
/// of point loops.
class BubbleUpPackThroughPadOp final : public OpRewritePattern<linalg::PackOp> {
public:
  BubbleUpPackThroughPadOp(MLIRContext *context, ControlPropagationFn fun)
      : OpRewritePattern<linalg::PackOp>(context), controlFn(std::move(fun)) {}

  LogicalResult matchAndRewrite(linalg::PackOp packOp,
                                PatternRewriter &rewriter) const override {
    if (!packOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Introduces declarations for `BubbleUpPackThroughPadOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `BubbleUpPackThroughPadOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 604-623
```cpp
    auto padOp = packOp.getSource().getDefiningOp<tensor::PadOp>();
    if (!padOp)
      return failure();

    // User controlled propagation function.
    if (!controlFn(&packOp.getSourceMutable()))
      return failure();

    // TODO: Enable padding when the padding values are the same.
    if (packOp.getPaddingValue())
      return failure();

    // Fail for non-constant padding values. The body of the pad could
    // depend on the padding indices and/or properties of the padded
    // tensor so for now we fail.
    // TODO: Support non-constant padding values.
    Value paddingVal = padOp.getConstantPaddingValue();
    if (!paddingVal)
      return failure();

```
- **EN**: Implements logic around `getSource`, `failure`, `controlFn`, `getPaddingValue`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSource`, `failure`, `controlFn`, `getPaddingValue`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 624-649
```cpp
    if (!packOp.getDest().getDefiningOp<tensor::EmptyOp>())
      return failure();

    ArrayRef<int64_t> innerDimsPos = packOp.getInnerDimsPos();

    // Bail out if one of the padded dimension is a tiled one.
    llvm::SmallBitVector paddedDims = padOp.getPaddedDims();
    llvm::SmallBitVector innerDims(paddedDims.size());
    for (int64_t dim : innerDimsPos)
      innerDims.flip(dim);
    if (paddedDims.anyCommon(innerDims))
      return failure();

    Location loc = padOp->getLoc();
    OpBuilder::InsertionGuard guard(rewriter);
    rewriter.setInsertionPoint(padOp);

    ArrayRef<int64_t> outerDimsPerm = packOp.getOuterDimsPerm();
    SmallVector<OpFoldResult> mixedTiles = packOp.getMixedTiles();
    auto empty = linalg::PackOp::createDestinationTensor(
        rewriter, loc, padOp.getSource(), mixedTiles, innerDimsPos,
        outerDimsPerm);
    auto sourcePack = linalg::PackOp::create(
        rewriter, loc, padOp.getSource(), empty, innerDimsPos, mixedTiles,
        /*padding=*/std::nullopt, outerDimsPerm);

```
- **EN**: Implements logic around `getDest`, `failure`, `getInnerDimsPos`, `getPaddedDims`, and 11 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDest`, `failure`, `getInnerDimsPos`, `getPaddedDims`, and 11 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 650-677
```cpp
    // If we have `outer_dims_perms` we need to adjust the padded dimensions.
    SmallVector<OpFoldResult> lowPad = padOp.getMixedLowPad();
    SmallVector<OpFoldResult> highPad = padOp.getMixedHighPad();
    if (!outerDimsPerm.empty()) {
      applyPermutationToVector<OpFoldResult>(lowPad, outerDimsPerm);
      applyPermutationToVector<OpFoldResult>(highPad, outerDimsPerm);
    }
    // The tiled dimensions were verified to be unpadded above, so here we
    // just append 0 for the inner tile dimensions.
    size_t pointLoopsSize = innerDimsPos.size();
    lowPad.append(pointLoopsSize, rewriter.getIndexAttr(0));
    highPad.append(pointLoopsSize, rewriter.getIndexAttr(0));

    auto newPadOp = tensor::PadOp::create(
        rewriter, loc, /*result=*/Type(), sourcePack.getResult(), lowPad,
        highPad, paddingVal, padOp.getNofold());

    // If the pad has more than one user, create an unpack on the new pad to
    // replace the other uses.
    if (!padOp->hasOneUse()) {
      auto unpackEmpty = linalg::UnPackOp::createDestinationTensor(
          rewriter, loc, newPadOp, mixedTiles, innerDimsPos, outerDimsPerm);
      UnPackOp unpackedPad =
          linalg::UnPackOp::create(rewriter, loc, newPadOp, unpackEmpty,
                                   innerDimsPos, mixedTiles, outerDimsPerm);
      rewriter.replaceAllUsesExcept(padOp, unpackedPad.getResult(), sourcePack);
    }

```
- **EN**: Implements logic around `getMixedLowPad`, `getMixedHighPad`, `empty`, `applyPermutationToVector`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMixedLowPad`, `getMixedHighPad`, `empty`, `applyPermutationToVector`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 678-695
```cpp
    // Replace the pack with the new pad.
    rewriter.replaceOp(packOp, newPadOp.getResult());

    return success();
  }

private:
  ControlPropagationFn controlFn;
};

/// Project dimsPos to the inner-most non-unit dim pos with reassocIndices.
///
/// For example, given dimsPos [0, 2], reassocIndices [[0, 1], [2, 3]], and
/// targetShape [16, 16, 32, 1], it returns [1, 2]. Because for pos 0, the
/// inner-most projected dim in pos [0, 1] is 1. And for pos 2, the inner-most
/// non-unit projected dims in pos [2, 3] is 2.
///
/// If all candidates in a reassociation are unit dims, it chooses the
```
- **EN**: Implements logic around `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 696-716
```cpp
/// inner-most dim pos.
static SmallVector<int64_t>
projectToInnerMostNonUnitDimsPos(ArrayRef<int64_t> dimsPos,
                                 ArrayRef<ReassociationIndices> reassocIndices,
                                 ArrayRef<int64_t> targetShape) {
  SmallVector<int64_t> projectedDimsPos;
  for (auto pos : dimsPos) {
    // In the case all dims are unit, this will return the inner-most one.
    int64_t projectedPos = reassocIndices[pos].back();
    for (auto i : llvm::reverse(reassocIndices[pos])) {
      int64_t dim = targetShape[i];
      if (dim > 1 || ShapedType::isDynamic(dim)) {
        projectedPos = i;
        break;
      }
    }
    projectedDimsPos.push_back(projectedPos);
  }
  return projectedDimsPos;
}

```
- **EN**: Implements logic around `projectToInnerMostNonUnitDimsPos`, `back`, `reverse`, `isDynamic`, and 1 more symbols.
- **CN**: 围绕 `projectToInnerMostNonUnitDimsPos`, `back`, `reverse`, `isDynamic`, and 1 more symbols 实现具体逻辑。

### Lines 717-734
```cpp
/// Check if all dims in dimsPos are divisible by the corresponding tile sizes.
static bool isDimsDivisibleByTileSizes(ArrayRef<int64_t> dimsPos,
                                       ArrayRef<int64_t> shape,
                                       ArrayRef<int64_t> tileSizes) {
  for (auto [pos, tileSize] : llvm::zip_equal(dimsPos, tileSizes)) {
    int64_t dim = shape[pos];
    if (ShapedType::isDynamic(dim) || (dim % tileSize) != 0)
      return false;
  }
  return true;
}

/// Permutate the reassociation indices and reindex them in the sequence order.
/// Returns the next dim pos in the sequence.
///
/// For example, given reassocIndices [[0, 1], [2]] and permutation [1, 0], it
/// applies the permutation to get [[2], [0, 1]] and reindexes the indices into
/// [[0], [1, 2]].
```
- **EN**: Implements logic around `isDimsDivisibleByTileSizes`, `zip_equal`, `isDynamic`.
- **CN**: 围绕 `isDimsDivisibleByTileSizes`, `zip_equal`, `isDynamic` 实现具体逻辑。

### Lines 735-752
```cpp
static int64_t applyPermutationAndReindexReassoc(
    SmallVector<ReassociationIndices> &reassocIndices,
    ArrayRef<int64_t> permutation) {
  if (!permutation.empty())
    applyPermutationToVector<ReassociationIndices>(reassocIndices, permutation);
  int64_t nextPos = 0;
  for (ReassociationIndices &indices : reassocIndices) {
    for (auto &index : indices) {
      index = nextPos;
      nextPos += 1;
    }
  }
  return nextPos;
}

/// Bubble up pack op through collapse shape op when the packed dims can be
/// projected to the dims before collapsing. This is possible when the inner
/// tile sizes can divide the projected dims.
```
- **EN**: Implements logic around `applyPermutationAndReindexReassoc`, `empty`, `applyPermutationToVector`.
- **CN**: 围绕 `applyPermutationAndReindexReassoc`, `empty`, `applyPermutationToVector` 实现具体逻辑。

### Lines 753-775
```cpp
///
/// For example:
///
/// %collapsed = tensor.collapse_shape %in [[0, 1], 2]
///     : tensor<?x16x4xf32> into tensor<?x4xf32>
/// %pack = linalg.pack %collapsed outer_dims_perm = [0, 1]
///     inner_dims_pos = [0, 1] inner_tiles = [8, 1] into %empty
///     : tensor<?x4xf32> -> tensor<?x4x8x1xf32>
///
/// can be transformed into:
///
/// %pack = linalg.pack %in outer_dims_perm = [1, 2]
///     inner_dims_pos = [1, 2] inner_tiles = [8, 1] into %empty
///     : tensor<?x16x4xf32> -> tensor<?x2x4x8x1xf32>
/// %collapsed = tensor.collapse_shape %pack [[0, 1], 2, 3, 4]
///     : tensor<?x2x4x8x1xf32> into tensor<?x4x8x1>
static LogicalResult
bubbleUpPackOpThroughCollapseShape(tensor::CollapseShapeOp collapseOp,
                                   linalg::PackOp packOp,
                                   PatternRewriter &rewriter) {
  if (!packOp.hasPureTensorSemantics())
    return failure();

```
- **EN**: Implements logic around `bubbleUpPackOpThroughCollapseShape`, `hasPureTensorSemantics`, `failure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bubbleUpPackOpThroughCollapseShape`, `hasPureTensorSemantics`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 776-803
```cpp
  SmallVector<int64_t> innerTileSizes = packOp.getStaticTiles();
  ArrayRef<int64_t> innerDimsPos = packOp.getInnerDimsPos();
  ArrayRef<int64_t> outerDimsPerm = packOp.getOuterDimsPerm();

  ArrayRef<int64_t> srcShape = collapseOp.getSrcType().getShape();
  SmallVector<ReassociationIndices> reassocIndices =
      collapseOp.getReassociationIndices();
  // Project inner tile pos to the dim pos before collapsing. For example, if
  // dims [x, y] is collapsed into [z], packing on dim z can be projected back
  // to pack on dim y.
  //
  // Project to inner-most non-unit dims to increase the chance that they can be
  // divided by the inner tile sizes. This is correct because for [..., x, 1],
  // packing on dim 1 is equivalent to packing on dim x.
  SmallVector<int64_t> projectedInnerDimsPos =
      projectToInnerMostNonUnitDimsPos(innerDimsPos, reassocIndices, srcShape);

  if (!isDimsDivisibleByTileSizes(projectedInnerDimsPos, srcShape,
                                  innerTileSizes)) {
    return failure();
  }
  // Expand the outer dims permutation with the associated source dims for the
  // new permutation after bubbling. This is because moving a collapsed dim is
  // equivalent to moving the associated source dims together.
  SmallVector<int64_t> newOuterDimsPerm;
  for (auto outerPos : outerDimsPerm)
    llvm::append_range(newOuterDimsPerm, reassocIndices[outerPos]);

```
- **EN**: Implements logic around `getStaticTiles`, `getInnerDimsPos`, `getOuterDimsPerm`, `getSrcType`, and 5 more symbols.
- **CN**: 围绕 `getStaticTiles`, `getInnerDimsPos`, `getOuterDimsPerm`, `getSrcType`, and 5 more symbols 实现具体逻辑。

### Lines 804-823
```cpp
  auto emptyOp = linalg::PackOp::createDestinationTensor(
      rewriter, packOp.getLoc(), collapseOp.getSrc(), packOp.getMixedTiles(),
      projectedInnerDimsPos, newOuterDimsPerm);
  auto newPackOp = linalg::PackOp::create(
      rewriter, packOp.getLoc(), collapseOp.getSrc(), emptyOp,
      projectedInnerDimsPos, packOp.getMixedTiles(), packOp.getPaddingValue(),
      newOuterDimsPerm);

  SmallVector<ReassociationIndices> newReassocIndices = reassocIndices;
  // First apply the permutation on the reassociations of the outer dims.
  // For example given the permutation [1, 0], the reassociations [[0, 1], [2]]
  // -> [[0], [1, 2]]
  int64_t nextPos =
      applyPermutationAndReindexReassoc(newReassocIndices, outerDimsPerm);
  // Then add direct mapping for the inner tile dims.
  for (size_t i = 0; i < innerDimsPos.size(); ++i) {
    newReassocIndices.push_back({nextPos});
    nextPos += 1;
  }

```
- **EN**: Implements logic around `createDestinationTensor`, `getLoc`, `create`, `getMixedTiles`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createDestinationTensor`, `getLoc`, `create`, `getMixedTiles`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 824-842
```cpp
  auto newCollapseOp = tensor::CollapseShapeOp::create(
      rewriter, collapseOp.getLoc(), packOp.getResult().getType(),
      newPackOp.getResult(), newReassocIndices);
  rewriter.replaceOp(packOp, newCollapseOp);

  return success();
}

/// Project dimsPos to their collapsed positions in the reassocIndices.
///
/// For example, given dimsPos [0, 1, 2, 4], and matching reassocIndices
/// [[0], [1, 2], [3], [4]], it returns [0, 1, 1, 3]. Because for pos 0,
/// the reassoc dim [0] is 0. For pos 1 and 2, the reassoc dim in pos
/// [1, 2] is 1. And for pos 4, the reassoc dim [4] is 3.
static SmallVector<int64_t>
projectDimsPosIntoReassocPos(ArrayRef<int64_t> dimsPos,
                             ArrayRef<ReassociationIndices> reassocIndices) {
  SmallVector<int64_t> projectedPos;

```
- **EN**: Implements logic around `create`, `getLoc`, `getResult`, `replaceOp`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getLoc`, `getResult`, `replaceOp`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 843-860
```cpp
  // Map each dimension to the position of corresponding reassociation index.
  for (auto pos : dimsPos) {
    for (auto [idx, indices] : llvm::enumerate(reassocIndices)) {
      // If the dimension is present in the current indices group, the group
      // position within the reassociation map is the desired projected
      // dimension position.
      if (llvm::is_contained(indices, pos)) {
        projectedPos.push_back(idx);
        break;
      }
    }
  }
  assert(projectedPos.size() == dimsPos.size() && "Invalid dim pos projection");

  return projectedPos;
}

/// Bubble up pack op through expand shape op.
```
- **EN**: Implements logic around `enumerate`, `is_contained`, `push_back`, `assert`.
- **CN**: 围绕 `enumerate`, `is_contained`, `push_back`, `assert` 实现具体逻辑。

### Lines 861-883
```cpp
///
/// For example:
///
/// %expand = tensor.expand_shape %in [[0], [1, 2]]
///     : tensor<?x64xf32> into tensor<?x4x16xf32>
/// %pack = linalg.pack %expand outer_dims_perm = [0, 1]
///     inner_dims_pos = [2] inner_tiles = [8] into %empty
///     : tensor<?x4x16xf32> -> tensor<?x4x2x8xf32>
///
/// can be transformed into:
///
/// %pack = linalg.pack %in outer_dims_perm = [1, 2]
///     inner_dims_pos = [1] inner_tiles = [8] into %empty
///     : tensor<?x64xf32> -> tensor<?x8x8xf32>
/// %expand = tensor.expand_shape %pack [[0], [1, 2], [3]]
///     : tensor<?x8x8xf32> into tensor<?x4x2x8xf32>
static LogicalResult
bubbleUpPackOpThroughExpandShape(tensor::ExpandShapeOp expandOp,
                                 linalg::PackOp packOp,
                                 PatternRewriter &rewriter) {
  if (!packOp.hasPureTensorSemantics())
    return failure();

```
- **EN**: Implements logic around `bubbleUpPackOpThroughExpandShape`, `hasPureTensorSemantics`, `failure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bubbleUpPackOpThroughExpandShape`, `hasPureTensorSemantics`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 884-904
```cpp
  // Outer dimensions permutation is not supported currently.
  // TODO: Handle outer_dims_perm variants.
  ArrayRef<int64_t> outerDimsPerm = packOp.getOuterDimsPerm();
  if (!outerDimsPerm.empty() && !isIdentityPermutation(outerDimsPerm)) {
    return rewriter.notifyMatchFailure(packOp,
                                       "non-identity outer dims perm NYI");
  }

  // Validate dimensions' relations between shape expansion and packing.
  SmallVector<ReassociationIndices, 4> reassoc =
      expandOp.getReassociationIndices();
  ArrayRef<int64_t> packInnerDims = packOp.getInnerDimsPos();
  llvm::SetVector<int64_t> packDimsPos(llvm::from_range, packInnerDims);

  for (auto [idx, indices] : llvm::enumerate(reassoc)) {
    // For each expand_shape reassociation, figure out which dimensions get
    // packed if any.
    llvm::SetVector<int64_t> expandDimPos(llvm::from_range, indices);
    llvm::SetVector<int64_t> packedDims =
        llvm::set_intersection(packDimsPos, expandDimPos);

```
- **EN**: Implements logic around `getOuterDimsPerm`, `empty`, `notifyMatchFailure`, `getReassociationIndices`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getOuterDimsPerm`, `empty`, `notifyMatchFailure`, `getReassociationIndices`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 905-925
```cpp
    // The expanded dimension is not packed so, it does not affect moving pack
    // before shape expansion - simply continue.
    if (packedDims.empty())
      continue;
    // Shape expansion cannot be propagated when multiple expanded dimension are
    // packed - in this case operation reordering would affect final element
    // positions and/or shapes can no longer be projected.
    if (packedDims.size() != 1)
      return rewriter.notifyMatchFailure(
          packOp, "only one of the expanded dimensions can be packed");
    // Only the inner-most expanded dimension should be packed. Otherwise,
    // elements order will be affected after operation reordering.
    if (packedDims.front() != indices.back())
      return rewriter.notifyMatchFailure(
          packOp, "can only pack the inner-most expanded dimension");
  }

  // Project pack.inner_dims_pos to positions before shape expansion.
  SmallVector<int64_t> projectedInnerDimsPos =
      projectDimsPosIntoReassocPos(packInnerDims, reassoc);

```
- **EN**: Implements logic around `empty`, `size`, `notifyMatchFailure`, `front`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `empty`, `size`, `notifyMatchFailure`, `front`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 926-950
```cpp
  // Project the shape expansion to new packed shape.
  // The pack.outer_dims_perm is restricted to identity so, the permutation can
  // be omitted for simplicity.
  // TODO: Account for outer dimensions permutation.
  //
  // If reassociation is not possible, then reordering cannot happen.
  // This can be caused by pack padding affecting previously expanded
  // dimensions or packing extending dimensions.
  RankedTensorType newPackType = linalg::PackOp::inferPackedTensorType(
      expandOp.getSrcType(), packOp.getStaticInnerTiles(),
      projectedInnerDimsPos, /*outerDimsPerm=*/SmallVector<int64_t>{});
  auto reassocExpand =
      getReassociationIndicesForReshape(newPackType, packOp.getDestType());
  if (!reassocExpand)
    return rewriter.notifyMatchFailure(
        packOp, "could not reassociate dims after bubbling up");

  Value destTensor = linalg::PackOp::createDestinationTensor(
      rewriter, packOp.getLoc(), expandOp.getSrc(), packOp.getMixedTiles(),
      projectedInnerDimsPos, /*outerDimsPerm=*/SmallVector<int64_t>{});
  PackOp packedVal = linalg::PackOp::create(
      rewriter, packOp.getLoc(), expandOp.getSrc(), destTensor,
      projectedInnerDimsPos, packOp.getMixedTiles(), packOp.getPaddingValue(),
      /*outerDimsPerm=*/SmallVector<int64_t>{});

```
- **EN**: Implements logic around `inferPackedTensorType`, `getSrcType`, `getReassociationIndicesForReshape`, `notifyMatchFailure`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `inferPackedTensorType`, `getSrcType`, `getReassociationIndicesForReshape`, `notifyMatchFailure`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 951-969
```cpp
  Value newExpandOp = tensor::ExpandShapeOp::create(
      rewriter, packOp.getLoc(), packOp.getDestType(), packedVal.getResult(),
      *reassocExpand);
  rewriter.replaceOp(packOp, newExpandOp);

  return success();
}

class BubbleUpPackOpThroughReshapeOp final
    : public OpRewritePattern<linalg::PackOp> {
public:
  BubbleUpPackOpThroughReshapeOp(MLIRContext *context, ControlPropagationFn fun)
      : OpRewritePattern<linalg::PackOp>(context), controlFn(std::move(fun)) {}

  LogicalResult matchAndRewrite(linalg::PackOp packOp,
                                PatternRewriter &rewriter) const override {
    if (!packOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Introduces declarations for `BubbleUpPackOpThroughReshapeOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `BubbleUpPackOpThroughReshapeOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 970-993
```cpp
    Operation *srcOp = packOp.getSource().getDefiningOp();
    // Currently only support when the pack op is the only user.
    if (!srcOp || !(srcOp->getNumResults() == 1) ||
        !srcOp->getResult(0).hasOneUse()) {
      return failure();
    }
    // Currently only support static inner tile sizes.
    if (llvm::any_of(packOp.getStaticTiles(), ShapedType::isDynamic))
      return failure();

    // User controlled propagation function.
    if (!controlFn(&packOp.getSourceMutable()))
      return failure();

    return TypeSwitch<Operation *, LogicalResult>(srcOp)
        .Case([&](tensor::CollapseShapeOp op) {
          return bubbleUpPackOpThroughCollapseShape(op, packOp, rewriter);
        })
        .Case([&](tensor::ExpandShapeOp op) {
          return bubbleUpPackOpThroughExpandShape(op, packOp, rewriter);
        })
        .Default(failure());
  }

```
- **EN**: Implements logic around `getSource`, `getNumResults`, `getResult`, `failure`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSource`, `getNumResults`, `getResult`, `failure`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 994-1011
```cpp
private:
  ControlPropagationFn controlFn;
};

/// Push down unpack op through expand shape op when the packed dims can be
/// projected to the dims after expanding. This is possible when the inner tile
/// sizes can divide the projected dims.
///
/// For example:
///
/// %unpack = linalg.unpack %in outer_dims_perm = [0, 1]
///     inner_dims_pos = [0, 1] inner_tiles = [8, 8] into %empty
///     : tensor<?x32x8x8xf32> -> tensor<?x256xf32>
/// %expanded = tensor.expand_shape %unpack [[0, 1], [2]]
///     : tensor<?x256xf32> into tensor<?x256x256xf32>
///
/// can be transformed into:
///
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 1012-1030
```cpp
/// %expanded = tensor.expand_shape %ain [[0, 1], [2], [3], [4]]
///     : tensor<?x32x8x8xf32> into tensor<?x32x32x8x8xf32>
/// %unpack = linalg.unpack %expanded outer_dims_perm = [0, 1, 2]
///     inner_dims_pos = [1, 2] inner_tiles = [8, 8] into %empty
///     : tensor<?x32x32x8x8xf32> -> tensor<?x256x256xf32>
static LogicalResult pushDownUnPackOpThroughExpandShape(
    linalg::UnPackOp unPackOp, tensor::ExpandShapeOp expandOp,
    PatternRewriter &rewriter, ControlPropagationFn controlFn) {
  if (!unPackOp.hasPureTensorSemantics())
    return failure();

  // User controlled propagation function.
  if (!controlFn(&expandOp.getSrcMutable()))
    return failure();

  SmallVector<int64_t> innerTileSizes = unPackOp.getStaticTiles();
  ArrayRef<int64_t> innerDimsPos = unPackOp.getInnerDimsPos();
  ArrayRef<int64_t> outerDimsPerm = unPackOp.getOuterDimsPerm();

```
- **EN**: Implements logic around `pushDownUnPackOpThroughExpandShape`, `hasPureTensorSemantics`, `failure`, `controlFn`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pushDownUnPackOpThroughExpandShape`, `hasPureTensorSemantics`, `failure`, `controlFn`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1031-1057
```cpp
  auto expandTy = dyn_cast<RankedTensorType>(expandOp.getType());
  if (!expandTy)
    return failure();
  ArrayRef<int64_t> dstShape = expandTy.getShape();
  SmallVector<ReassociationIndices> reassocIndices =
      expandOp.getReassociationIndices();
  // Project inner tile pos to the dim pos after expanding. For example, if dims
  // [z] is expanded into [x, y], unpacking on dim z can be projected to unpack
  // on dim y.
  //
  // Project to inner-most non-unit dims to increase the chance that they can be
  // divided by the inner tile sizes. This is correct because for [..., x, 1],
  // unpacking on dim 1 is equivalent to unpacking on dim x.
  SmallVector<int64_t> projectedInnerDimsPos =
      projectToInnerMostNonUnitDimsPos(innerDimsPos, reassocIndices, dstShape);

  if (!isDimsDivisibleByTileSizes(projectedInnerDimsPos, dstShape,
                                  innerTileSizes)) {
    return failure();
  }
  // Expand the outer dims permutation with the associated expanded dims for the
  // new permutation after pushing. This is because moving a source dim is
  // equivalent to moving the associated expanded dims together.
  SmallVector<int64_t> newOuterDimsPerm;
  for (auto outerPos : outerDimsPerm)
    llvm::append_range(newOuterDimsPerm, reassocIndices[outerPos]);

```
- **EN**: Implements logic around `getType`, `failure`, `getShape`, `getReassociationIndices`, and 3 more symbols.
- **CN**: 围绕 `getType`, `failure`, `getShape`, `getReassociationIndices`, and 3 more symbols 实现具体逻辑。

### Lines 1058-1075
```cpp
  SmallVector<ReassociationIndices> newReassocIndices = reassocIndices;
  // First apply the permutation on the reassociations of the outer dims.
  // For example given the permutation [1, 0], the reassociations [[0, 1], [2]]
  // -> [[0], [1, 2]]
  int64_t nextPos =
      applyPermutationAndReindexReassoc(newReassocIndices, outerDimsPerm);
  // Then add direct mapping for the inner tile dims.
  for (size_t i = 0; i < innerDimsPos.size(); ++i) {
    newReassocIndices.push_back({nextPos});
    nextPos += 1;
  }

  RankedTensorType newExpandType = linalg::PackOp::inferPackedTensorType(
      expandTy, innerTileSizes, projectedInnerDimsPos, newOuterDimsPerm);
  auto newExpandOp =
      tensor::ExpandShapeOp::create(rewriter, expandOp.getLoc(), newExpandType,
                                    unPackOp.getSource(), newReassocIndices);

```
- **EN**: Implements logic around `applyPermutationAndReindexReassoc`, `size`, `push_back`, `inferPackedTensorType`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `applyPermutationAndReindexReassoc`, `size`, `push_back`, `inferPackedTensorType`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1076-1094
```cpp
  auto emptyOp = linalg::UnPackOp::createDestinationTensor(
      rewriter, unPackOp.getLoc(), newExpandOp, unPackOp.getMixedTiles(),
      projectedInnerDimsPos, newOuterDimsPerm);
  auto newUnPackOp = linalg::UnPackOp::create(
      rewriter, unPackOp.getLoc(), newExpandOp.getResult(), emptyOp,
      projectedInnerDimsPos, unPackOp.getMixedTiles(), newOuterDimsPerm);
  rewriter.replaceOp(expandOp, newUnPackOp);

  return success();
}

class PushDownUnPackOpThroughReshapeOp final
    : public OpRewritePattern<linalg::UnPackOp> {
public:
  PushDownUnPackOpThroughReshapeOp(MLIRContext *context,
                                   ControlPropagationFn fun)
      : OpRewritePattern<linalg::UnPackOp>(context), controlFn(std::move(fun)) {
  }

```
- **EN**: Introduces declarations for `PushDownUnPackOpThroughReshapeOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PushDownUnPackOpThroughReshapeOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1095-1117
```cpp
  LogicalResult matchAndRewrite(linalg::UnPackOp unPackOp,
                                PatternRewriter &rewriter) const override {
    if (!unPackOp.hasPureTensorSemantics())
      return failure();

    Value result = unPackOp.getResult();
    // Currently only support unpack op with the single user.
    if (!result.hasOneUse()) {
      return failure();
    }
    // Currently only support static inner tile sizes.
    if (llvm::any_of(unPackOp.getStaticTiles(), ShapedType::isDynamic))
      return failure();

    Operation *consumerOp = *result.user_begin();
    return TypeSwitch<Operation *, LogicalResult>(consumerOp)
        .Case([&](tensor::ExpandShapeOp op) {
          return pushDownUnPackOpThroughExpandShape(unPackOp, op, rewriter,
                                                    controlFn);
        })
        .Default(failure());
  }

```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `failure`, `getResult`, and 7 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `failure`, `getResult`, and 7 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1118-1139
```cpp
private:
  ControlPropagationFn controlFn;
};

// TODO: Relax this restriction. We should unpack a generic op also
// in the presence of multiple unpack ops as producers.
/// Return the unpacked operand, if present, for the current generic op.
static FailureOr<OpOperand *> getUnPackedOperand(GenericOp genericOp) {
  OpOperand *unPackedOperand = nullptr;
  for (OpOperand &operand : genericOp->getOpOperands()) {
    auto unPackOp = operand.get().getDefiningOp<linalg::UnPackOp>();
    if (!unPackOp)
      continue;
    if (unPackedOperand)
      return failure();
    unPackedOperand = &operand;
  }
  if (!unPackedOperand)
    return failure();
  return unPackedOperand;
}

```
- **EN**: Implements logic around `getUnPackedOperand`, `getOpOperands`, `get`, `failure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getUnPackedOperand`, `getOpOperands`, `get`, `failure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1140-1157
```cpp
/// Push down a linalg.unpack op through a generic op.
/// The new generic op works on packed domain; pack ops are created for input
/// and output operands. A linalg.unpack op is inserted right after the packed
/// generic. E.g.
///
/// #map = affine_map<(d0, d1, d2, d3) -> (d0, d1, d2, d3)>
///
/// %arg0 = tensor<12x2x56x56x32xf32> // packed arg.
///
/// %0 = tensor.empty() : tensor<12x56x56x64xf32>
/// %1 = linalg.unpack %arg0 outer_dims_perm = [0, 3, 1, 2]
///                          inner_dims_pos = [3] inner_tiles = [32] into %0
/// %2 = linalg.generic {indexing_maps = [#map],
///      iterator_types = ["parallel", "parallel", "parallel", "parallel"]}
///      outs(%1 : tensor<12x56x56x64xf32>) {
///      ^bb0(%out : f32):
///         linalg.yield %out : f32
///      } -> tensor<12x56x56x64xf32>
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 1158-1180
```cpp
///
/// will be converted to
///
/// #map = affine_map<(d0, d1, d2, d3, d4) -> (d0, d1, d2, d3, d4)>
///
/// %0 = tensor.empty() : tensor<12x56x56x64xf32>
/// %1 = linalg.generic {indexing_maps = [#map],
///      iterator_types = ["parallel", "parallel", "parallel",
///                        "parallel", "parallel"]}
///      outs(%arg0 : tensor<12x2x56x56x32xf32>) {
///      ^bb0(%out : f32):
///         linalg.yield %out : f32
///      } -> tensor<12x2x56x56x32xf32>
/// %2 = linalg.unpack %1 outer_dims_perm = [0, 3, 1, 2]
///                       inner_dims_pos = [3] inner_tiles = [32] into %0
///
static FailureOr<std::tuple<GenericOp, Value>>
pushDownUnPackOpThroughGenericOp(RewriterBase &rewriter, GenericOp genericOp,
                                 ControlPropagationFn controlFn,
                                 bool poisonPaddingOk) {
  if (genericOp.getNumResults() != 1)
    return failure();

```
- **EN**: Implements logic around `pushDownUnPackOpThroughGenericOp`, `getNumResults`, `failure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `pushDownUnPackOpThroughGenericOp`, `getNumResults`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1181-1202
```cpp
  if (hasGatherSemantics(genericOp))
    return failure();

  // Collect the unPacked operand, if present.
  auto maybeUnPackedOperand = getUnPackedOperand(genericOp);
  if (failed(maybeUnPackedOperand))
    return failure();
  OpOperand *unPackedOperand = *(maybeUnPackedOperand);

  // Extract packing information.
  linalg::UnPackOp producerUnPackOp =
      unPackedOperand->get().getDefiningOp<linalg::UnPackOp>();
  assert(producerUnPackOp && "expect a valid UnPackOp");

  if (!controlFn(unPackedOperand))
    return failure();

  auto packInfo =
      getPackingInfoFromOperand(unPackedOperand, genericOp, producerUnPackOp);
  if (failed(packInfo))
    return failure();

```
- **EN**: Implements logic around `hasGatherSemantics`, `failure`, `getUnPackedOperand`, `failed`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasGatherSemantics`, `failure`, `getUnPackedOperand`, `failed`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1203-1229
```cpp
  // Rebuild the indexing map for the corresponding init operand.
  DenseMap<OpOperand *, PackedOperandDetails> packedOperandMap;
  bool requiresPadding =
      getPackedOperandDetails(rewriter, *packInfo, genericOp,
                              genericOp.getDpsInitOperand(0), packedOperandMap);
  if (requiresPadding && !poisonPaddingOk)
    return failure();

  auto [packedOutOperand, packedOutIndexingMap] =
      getOrCreatePackedViewOfOperand(rewriter, genericOp.getLoc(),
                                     genericOp.getDpsInitOperand(0),
                                     packedOperandMap);
  auto destPack = packedOutOperand.getDefiningOp<linalg::PackOp>();

  // Forward the new tensor.empty as a destination if it is one of the following
  // situations:
  // 1) The dps init operand is a tensor.empty.
  // 2) The dps init is a write-only operand, i.e., it is not used in the
  // genericOp
  Value dest = packedOutOperand;
  auto initTensor =
      genericOp.getDpsInitOperand(0)->get().getDefiningOp<tensor::EmptyOp>();
  if (initTensor || isGenericOutsNotUsed(genericOp)) {
    if (destPack)
      dest = destPack.getDest();
  }

```
- **EN**: Implements logic around `getPackedOperandDetails`, `getDpsInitOperand`, `failure`, `getOrCreatePackedViewOfOperand`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPackedOperandDetails`, `getDpsInitOperand`, `failure`, `getOrCreatePackedViewOfOperand`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1230-1250
```cpp
  // Pack the genericOp.
  // pack(unpack) is foldable in this case. This is because in pushing down the
  // unpack, by default we will populate an additional pack op after the unpack.
  // This guarantees them to be foldable.
  auto maybeGenericOp =
      packGenericOp(rewriter, genericOp, dest, packedOutIndexingMap, *packInfo,
                    /*isFoldableUnpackPack=*/true, poisonPaddingOk);
  if (failed(maybeGenericOp))
    return failure();
  GenericOp newGenericOp = *maybeGenericOp;
  Value newResult =
      newGenericOp.getTiedOpResult(newGenericOp.getDpsInitOperand(0));

  // If the output is unaffected, no need to unpack.
  if (!destPack)
    return std::make_tuple(newGenericOp, newResult);

  auto mixedTiles = destPack.getMixedTiles();
  auto innerDimsPos = destPack.getInnerDimsPos();
  auto outerDimsPerm = destPack.getOuterDimsPerm();

```
- **EN**: Implements logic around `packGenericOp`, `failed`, `failure`, `getTiedOpResult`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `packGenericOp`, `failed`, `failure`, `getTiedOpResult`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1251-1269
```cpp
  // Insert an unPackOp right after the packed generic.
  Value unPackOpRes =
      linalg::UnPackOp::create(rewriter, genericOp.getLoc(), newResult,
                               destPack.getSource(), innerDimsPos, mixedTiles,
                               outerDimsPerm)
          .getResult();

  return std::make_tuple(newGenericOp, unPackOpRes);
}

// Wrapper pattern that applies pushDownUnPackOpThroughGenericOp method.
struct PushDownUnPackOpThroughGenericOp : public OpRewritePattern<GenericOp> {
public:
  PushDownUnPackOpThroughGenericOp(MLIRContext *context,
                                   ControlPropagationFn fun,
                                   bool poisonPaddingOk)
      : OpRewritePattern<GenericOp>(context), controlFn(std::move(fun)),
        poisonPaddingOk(std::move(poisonPaddingOk)) {}

```
- **EN**: Introduces declarations for `PushDownUnPackOpThroughGenericOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PushDownUnPackOpThroughGenericOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1270-1287
```cpp
  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    auto genericAndRepl = pushDownUnPackOpThroughGenericOp(
        rewriter, genericOp, controlFn, poisonPaddingOk);
    if (failed(genericAndRepl))
      return failure();
    rewriter.replaceOp(genericOp, std::get<1>(*genericAndRepl));
    return success();
  }

private:
  ControlPropagationFn controlFn;
  bool poisonPaddingOk;
};

/// Propagate a linalg.unpack operation through a tensor.pad. The idea is to
/// add as many zero padding dimensions in `high` and `low` based on the number
/// of point loops.
```
- **EN**: Implements logic around `matchAndRewrite`, `pushDownUnPackOpThroughGenericOp`, `failed`, `failure`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `pushDownUnPackOpThroughGenericOp`, `failed`, `failure`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1288-1314
```cpp
struct PushDownUnPackThroughPadOp : public OpRewritePattern<tensor::PadOp> {
  PushDownUnPackThroughPadOp(MLIRContext *context, ControlPropagationFn fun)
      : OpRewritePattern<tensor::PadOp>(context), controlFn(std::move(fun)) {}

  LogicalResult matchAndRewrite(tensor::PadOp padOp,
                                PatternRewriter &rewriter) const override {
    linalg::UnPackOp unpackOp =
        padOp.getSource().getDefiningOp<linalg::UnPackOp>();
    if (!unpackOp)
      return failure();

    if (!unpackOp.hasPureTensorSemantics())
      return failure();

    if (!controlFn(&padOp.getSourceMutable()))
      return failure();

    Location loc = padOp.getLoc();
    // Bail out if one of the padded dimension is a tiled one.
    llvm::SmallBitVector paddedDims = padOp.getPaddedDims();
    ArrayRef<int64_t> innerDimsPos = unpackOp.getInnerDimsPos();
    llvm::SmallBitVector innerDims(paddedDims.size());
    for (int64_t dim : innerDimsPos)
      innerDims.flip(dim);
    if (paddedDims.anyCommon(innerDims))
      return failure();

```
- **EN**: Introduces declarations for `PushDownUnPackThroughPadOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PushDownUnPackThroughPadOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1315-1335
```cpp
    Value paddingVal = padOp.getConstantPaddingValue();
    if (!paddingVal)
      return failure();

    // If we have `outer_dims_perms` we need to adjust the padded dimensions.
    ArrayRef<int64_t> outerDimsPerm = unpackOp.getOuterDimsPerm();
    SmallVector<OpFoldResult> lowPad = padOp.getMixedLowPad();
    SmallVector<OpFoldResult> highPad = padOp.getMixedHighPad();
    if (!outerDimsPerm.empty()) {
      applyPermutationToVector<OpFoldResult>(lowPad, outerDimsPerm);
      applyPermutationToVector<OpFoldResult>(highPad, outerDimsPerm);
    }
    // Add zero padding for the point loops.
    size_t pointLoopsSize = innerDimsPos.size();
    lowPad.append(pointLoopsSize, rewriter.getIndexAttr(0));
    highPad.append(pointLoopsSize, rewriter.getIndexAttr(0));

    auto newPadOp = tensor::PadOp::create(rewriter, loc, /*result=*/Type(),
                                          unpackOp.getSource(), lowPad, highPad,
                                          paddingVal, padOp.getNofold());

```
- **EN**: Implements logic around `getConstantPaddingValue`, `failure`, `getOuterDimsPerm`, `getMixedLowPad`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getConstantPaddingValue`, `failure`, `getOuterDimsPerm`, `getMixedLowPad`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1336-1353
```cpp
    // Inject the linalg.unpack right after the packed padOp.
    Value outputUnPack =
        tensor::EmptyOp::create(rewriter, loc, padOp.getResultType().getShape(),
                                padOp.getResultType().getElementType());

    UnPackOp replacement = linalg::UnPackOp::create(
        rewriter, loc, newPadOp.getResult(), outputUnPack, innerDimsPos,
        unpackOp.getMixedTiles(), outerDimsPerm);
    rewriter.replaceOp(padOp, replacement);
    return success();
  }

private:
  ControlPropagationFn controlFn;
};

// This struct contains infomation about extract_slice dims.
struct SliceDimInfo {
```
- **EN**: Introduces declarations for `contains`, `SliceDimInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `contains`, `SliceDimInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1354-1375
```cpp
  OpFoldResult offset;
  OpFoldResult sliceSize;
  OpFoldResult outputSize;
};

/// Return all extract slice operands, if present, for the current
/// generic op.
static FailureOr<SmallVector<OpOperand *>>
getSliceOperands(GenericOp genericOp) {
  SmallVector<OpOperand *> sliceOperands;
  for (auto operand : genericOp.getDpsInputOperands()) {
    auto extractOp = operand->get().getDefiningOp<tensor::ExtractSliceOp>();
    if (!extractOp)
      continue;
    sliceOperands.push_back(operand);
  }
  if (sliceOperands.empty()) {
    return failure();
  }
  return sliceOperands;
}

```
- **EN**: Implements logic around `getSliceOperands`, `getDpsInputOperands`, `get`, `push_back`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getSliceOperands`, `getDpsInputOperands`, `get`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 1376-1411
```cpp
// Return a map of dims that have partial slices on them so that other operands
// can use this information. Also return a bool mentioning if a reduction dim
// has a non full slice as that can be used to fold the original extract slice.
static FailureOr<llvm::DenseMap<int64_t, SliceDimInfo>>
getPartialSliceDimInfo(GenericOp genericOp, OpOperand *sliceOperand) {
  tensor::ExtractSliceOp producerSliceOp =
      sliceOperand->get().getDefiningOp<tensor::ExtractSliceOp>();
  assert(producerSliceOp && "expect a valid ExtractSliceOp");
  llvm::DenseMap<int64_t, SliceDimInfo> partialSliceDimMap;
  SmallVector<OpFoldResult> offsets = producerSliceOp.getMixedOffsets();
  SmallVector<OpFoldResult> sizes = producerSliceOp.getMixedSizes();

  SmallVector<OpFoldResult> shape = getAsIndexOpFoldResult(
      genericOp.getContext(), producerSliceOp.getSourceType().getShape());

  for (auto [idx, expr] : llvm::enumerate(
           genericOp.getMatchingIndexingMap(sliceOperand).getResults())) {
    // If we have a full slice in a dimension then we dont need to add it to
    // the partial slice map.
    if (isConstantIntValue(offsets[idx], 0) &&
        isEqualConstantIntOrValue(sizes[idx], shape[idx])) {
      continue;
    }
    // We only support partial slices of AffineDimExprs so bail-out if thats not
    // the case.
    if (!isa<AffineDimExpr>(expr)) {
      return failure();
    }
    SliceDimInfo sliceDimInfo{offsets[idx], sizes[idx], shape[idx]};
    int64_t dimPos = cast<AffineDimExpr>(expr).getPosition();
    partialSliceDimMap[dimPos] = sliceDimInfo;
  }
  // Next check if the dims with partial slice info are used in non
  // AffineDimExpr in other operands and if they are then bail-out.
  for (OpOperand &operand : genericOp->getOpOperands()) {
    if (operand == *sliceOperand) {
```
- **EN**: Implements logic around `getPartialSliceDimInfo`, `get`, `assert`, `getMixedOffsets`, and 10 more symbols; this block performs canonicalization or folding-oriented reasoning; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPartialSliceDimInfo`, `get`, `assert`, `getMixedOffsets`, and 10 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1412-1437
```cpp
      continue;
    }
    AffineMap IndexingMap = genericOp.getMatchingIndexingMap(&operand);
    if (llvm::any_of(IndexingMap.getResults(), [&](AffineExpr expr) {
          if (isa<AffineDimExpr>(expr)) {
            return false;
          }
          WalkResult status = expr.walk([&](AffineExpr expr) {
            if (auto dimExpr = dyn_cast<AffineDimExpr>(expr)) {
              if (partialSliceDimMap.contains(dimExpr.getPosition())) {
                return WalkResult::interrupt();
              }
            }
            return WalkResult::advance();
          });
          if (status.wasInterrupted()) {
            return true;
          }
          return false;
        })) {
      return failure();
    }
  }
  return partialSliceDimMap;
}

```
- **EN**: Implements logic around `getMatchingIndexingMap`, `any_of`, `walk`, `contains`, and 4 more symbols.
- **CN**: 围绕 `getMatchingIndexingMap`, `any_of`, `walk`, `contains`, and 4 more symbols 实现具体逻辑。

### Lines 1438-1455
```cpp
static FailureOr<std::tuple<GenericOp, Value>>
pushDownExtractSliceOpThroughGenericOp(RewriterBase &rewriter,
                                       GenericOp genericOp,
                                       ControlPropagationFn controlFn) {
  if (genericOp.getNumResults() != 1)
    return rewriter.notifyMatchFailure(
        genericOp, "propagation through multi-result generic is unsupported.");
  if (hasGatherSemantics(genericOp))
    return rewriter.notifyMatchFailure(
        genericOp,
        "propagation through generic with gather semantics is unsupported.");
  // Collect the sliced operand, if present.
  auto maybeSliceOperands = getSliceOperands(genericOp);
  if (failed(maybeSliceOperands))
    return failure();
  SmallVector<OpOperand *> sliceOperands = *maybeSliceOperands;
  OpOperand *sliceOperand;

```
- **EN**: Implements logic around `pushDownExtractSliceOpThroughGenericOp`, `getNumResults`, `notifyMatchFailure`, `hasGatherSemantics`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `pushDownExtractSliceOpThroughGenericOp`, `getNumResults`, `notifyMatchFailure`, `hasGatherSemantics`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1456-1479
```cpp
  bool foundValidOperand = false;
  for (auto currSliceOperand : sliceOperands) {
    if (controlFn(currSliceOperand)) {
      sliceOperand = currSliceOperand;
      foundValidOperand = true;
      break;
    }
  }
  if (!foundValidOperand) {
    return failure();
  }
  unsigned OperandIndex = sliceOperand->getOperandNumber();

  tensor::ExtractSliceOp producerSliceOp =
      sliceOperand->get().getDefiningOp<tensor::ExtractSliceOp>();
  assert(producerSliceOp && "expect a valid ExtractSliceOp");

  if (producerSliceOp.getSource().getType().getRank() !=
      producerSliceOp.getResult().getType().getRank()) {
    return rewriter.notifyMatchFailure(
        genericOp,
        "propagation of rank-reducing extract slice is unsupported.");
  }

```
- **EN**: Implements logic around `controlFn`, `failure`, `getOperandNumber`, `get`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `controlFn`, `failure`, `getOperandNumber`, `get`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1480-1504
```cpp
  SmallVector<OpFoldResult> strides = producerSliceOp.getMixedStrides();
  if (!areAllConstantIntValue(strides, 1))
    return rewriter.notifyMatchFailure(
        genericOp, "propagation of strided extract slice is unsupported.");

  // check if we can support the propagation of this extractSlice
  // through the generic op and if so return the dimensions that

  auto maybePartialSliceDimMap =
      getPartialSliceDimInfo(genericOp, sliceOperand);

  if (failed(maybePartialSliceDimMap)) {
    return failure();
  }

  auto partialSliceDimMap = *maybePartialSliceDimMap;

  SmallVector<utils::IteratorType> iterators =
      genericOp.getIteratorTypesArray();
  bool hasPartialReductionDimSlice =
      llvm::any_of(partialSliceDimMap, [&](const auto &slice) {
        int64_t sliceDim = slice.first;
        return iterators[sliceDim] == utils::IteratorType::reduction;
      });

```
- **EN**: Implements logic around `getMixedStrides`, `areAllConstantIntValue`, `notifyMatchFailure`, `getPartialSliceDimInfo`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getMixedStrides`, `areAllConstantIntValue`, `notifyMatchFailure`, `getPartialSliceDimInfo`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1505-1540
```cpp
  // Store the padding information as (dimPos, lowPad, highPad, PaddedShape).
  Location loc = genericOp->getLoc();
  AffineExpr dim0, dim1;
  bindDims(rewriter.getContext(), dim0, dim1);
  auto subMap = AffineMap::get(2, 0, {dim0 - dim1});
  auto sub = [&](OpFoldResult v1, OpFoldResult v2) {
    return affine::makeComposedFoldedAffineApply(rewriter, loc, subMap,
                                                 {v1, v2});
  };

  MLIRContext *ctx = genericOp.getContext();
  SmallVector<Value> paddedInputs;
  for (auto [idx, operand] : llvm::enumerate(genericOp.getDpsInputOperands())) {
    if (idx == OperandIndex && !hasPartialReductionDimSlice) {
      paddedInputs.push_back(producerSliceOp.getSource());
      continue;
    }
    AffineMap IndexingMap = genericOp.getMatchingIndexingMap(operand);
    if (IndexingMap.getNumResults() == 0) {
      paddedInputs.push_back(operand->get());
      continue;
    }
    SmallVector<OpFoldResult> operandLowPads(IndexingMap.getNumResults(),
                                             getAsIndexOpFoldResult(ctx, 0));
    SmallVector<OpFoldResult> operandHighPads(IndexingMap.getNumResults(),
                                              getAsIndexOpFoldResult(ctx, 0));
    for (auto [idx, expr] : llvm::enumerate(IndexingMap.getResults())) {
      if (!isa<AffineDimExpr>(expr)) {
        continue;
      }
      AffineDimExpr dimExpr = cast<AffineDimExpr>(expr);
      if (!partialSliceDimMap.contains(dimExpr.getPosition())) {
        continue;
      }
      SliceDimInfo sliceDimInfo = partialSliceDimMap[dimExpr.getPosition()];
      operandLowPads[idx] = sliceDimInfo.offset;
```
- **EN**: Implements logic around `getLoc`, `bindDims`, `get`, `makeComposedFoldedAffineApply`, and 10 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `bindDims`, `get`, `makeComposedFoldedAffineApply`, and 10 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1541-1576
```cpp
      operandHighPads[idx] =
          sub(sub(sliceDimInfo.outputSize, sliceDimInfo.offset),
              sliceDimInfo.sliceSize);
    }
    auto paddingValue = ub::PoisonOp::create(
        rewriter, loc, getElementTypeOrSelf(operand->get().getType()));
    auto paddedOperand = tensor::PadOp::create(
        rewriter, loc, Type(), operand->get(), operandLowPads, operandHighPads,
        paddingValue, /*nofold=*/false);
    paddedInputs.push_back(paddedOperand);
  }
  AffineMap outputIndexingMap =
      genericOp.getMatchingIndexingMap(genericOp.getDpsInitOperand(0));

  auto outputShapeType =
      llvm::cast<ShapedType>(genericOp.getDpsInitOperand(0)->get().getType());
  SmallVector<OpFoldResult> OutputShape = llvm::map_to_vector(
      outputShapeType.getShape(),
      [&](int64_t sz) -> OpFoldResult { return rewriter.getIndexAttr(sz); });
  SmallVector<OpFoldResult> newSizes = OutputShape;
  SmallVector<OpFoldResult> outputLowPads(outputIndexingMap.getNumResults(),
                                          getAsIndexOpFoldResult(ctx, 0));
  SmallVector<OpFoldResult> outputHighPads(outputIndexingMap.getNumResults(),
                                           getAsIndexOpFoldResult(ctx, 0));
  SmallVector<OpFoldResult> newStrides(outputIndexingMap.getNumResults(),
                                       getAsIndexOpFoldResult(ctx, 1));
  for (auto [idx, expr] : llvm::enumerate(outputIndexingMap.getResults())) {
    if (!isa<AffineDimExpr>(expr)) {
      continue;
    }
    AffineDimExpr dimExpr = cast<AffineDimExpr>(expr);
    if (!partialSliceDimMap.contains(dimExpr.getPosition())) {
      continue;
    }
    SliceDimInfo sliceDimInfo = partialSliceDimMap[dimExpr.getPosition()];
    outputLowPads[idx] = sliceDimInfo.offset;
```
- **EN**: Implements logic around `sub`, `create`, `getElementTypeOrSelf`, `Type`, and 13 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sub`, `create`, `getElementTypeOrSelf`, `Type`, and 13 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1577-1594
```cpp
    outputHighPads[idx] = sub(sub(sliceDimInfo.outputSize, sliceDimInfo.offset),
                              sliceDimInfo.sliceSize);
    OutputShape[idx] = sliceDimInfo.outputSize;
    newSizes[idx] = sliceDimInfo.sliceSize;
  }
  Value newPadOutput;
  auto outputElType =
      getElementTypeOrSelf(genericOp.getDpsInits()[0].getType());
  if (isGenericOutsNotUsed(genericOp)) {
    newPadOutput =
        tensor::EmptyOp::create(rewriter, loc, OutputShape, outputElType);
  } else {
    auto paddingValue = ub::PoisonOp::create(rewriter, loc, outputElType);
    newPadOutput = tensor::PadOp::create(
        rewriter, loc, Type(), genericOp.getDpsInits()[0], outputLowPads,
        outputHighPads, paddingValue, /*nofold=*/false);
  }

```
- **EN**: Implements logic around `sub`, `getElementTypeOrSelf`, `isGenericOutsNotUsed`, `create`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sub`, `getElementTypeOrSelf`, `isGenericOutsNotUsed`, `create`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1595-1613
```cpp
  auto newGenericOp = linalg::GenericOp::create(
      rewriter, loc, newPadOutput.getType(), paddedInputs, {newPadOutput},
      genericOp.getIndexingMapsArray(), genericOp.getIteratorTypesArray(),
      /*bodyBuild=*/nullptr, linalg::getPrunedAttributeList(genericOp));
  rewriter.cloneRegionBefore(genericOp.getRegion(), newGenericOp.getRegion(),
                             newGenericOp.getRegion().begin());

  auto extractOp = tensor::ExtractSliceOp::create(
      rewriter, loc,
      newGenericOp.getTiedOpResult(newGenericOp.getDpsInitOperand(0)),
      outputLowPads, newSizes, newStrides);
  Value extractRes = extractOp.getResult();

  return std::make_tuple(newGenericOp, extractRes);
}

class PushDownExtractSliceOpThroughGenericOp final
    : public OpRewritePattern<GenericOp> {
public:
```
- **EN**: Introduces declarations for `PushDownExtractSliceOpThroughGenericOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PushDownExtractSliceOpThroughGenericOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1614-1631
```cpp
  PushDownExtractSliceOpThroughGenericOp(MLIRContext *context,
                                         ControlPropagationFn fun)
      : OpRewritePattern<GenericOp>(context), controlFn(std::move(fun)) {}

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    auto genericAndRepl =
        pushDownExtractSliceOpThroughGenericOp(rewriter, genericOp, controlFn);
    if (failed(genericAndRepl))
      return failure();
    rewriter.replaceOp(genericOp, std::get<1>(*genericAndRepl));
    return success();
  }

private:
  ControlPropagationFn controlFn;
};

```
- **EN**: Implements logic around `PushDownExtractSliceOpThroughGenericOp`, `OpRewritePattern`, `matchAndRewrite`, `pushDownExtractSliceOpThroughGenericOp`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `PushDownExtractSliceOpThroughGenericOp`, `OpRewritePattern`, `matchAndRewrite`, `pushDownExtractSliceOpThroughGenericOp`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 1632-1651
```cpp
} // namespace

void mlir::linalg::populateDataLayoutPropagationPatterns(
    RewritePatternSet &patterns,
    const ControlPropagationFn &controlPackUnPackPropagation,
    bool PoisonPaddingOk) {
  patterns.insert<BubbleUpPackThroughPadOp, BubbleUpPackOpThroughReshapeOp,
                  PushDownUnPackThroughPadOp, PushDownUnPackOpThroughReshapeOp>(
      patterns.getContext(), controlPackUnPackPropagation);
  patterns.insert<BubbleUpPackOpThroughGenericOpPattern,
                  PushDownUnPackOpThroughGenericOp>(
      patterns.getContext(), controlPackUnPackPropagation, PoisonPaddingOk);
}

void mlir::linalg::populateExtractSliceSinkingPatterns(
    RewritePatternSet &patterns,
    const ControlPropagationFn &controlPackUnPackPropagation) {
  patterns.insert<PushDownExtractSliceOpThroughGenericOp>(
      patterns.getContext(), controlPackUnPackPropagation);
}
```
- **EN**: Implements logic around `populateDataLayoutPropagationPatterns`, `PushDownUnPackOpThroughReshapeOp>`, `getContext`, `PushDownUnPackOpThroughGenericOp>`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateDataLayoutPropagationPatterns`, `PushDownUnPackOpThroughReshapeOp>`, `getContext`, `PushDownUnPackOpThroughGenericOp>`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/IR/Dominance.h`, `mlir/IR/TypeUtilities.h`, `llvm/ADT/SetOperations.h` ... (+4 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (8), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), LLVM support-library facilities / LLVM Support 库设施 (1)
