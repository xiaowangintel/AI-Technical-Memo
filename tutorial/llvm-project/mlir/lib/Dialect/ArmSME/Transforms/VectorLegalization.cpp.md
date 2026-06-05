# VectorLegalization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSME/Transforms/VectorLegalization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass legalizes vector operations so they can be lowered to ArmSME.
  - **CN**: 实现 ArmSME 方言与 Arm SME tile/vector 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- VectorLegalization.cpp - Legalize vectors for lowering to ArmSME ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass legalizes vector operations so they can be lowered to ArmSME.
//
// Note: In the context of this pass 'tile' always refers to an SME tile.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 15-30
```cpp
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/ArmSME/IR/ArmSME.h"
#include "mlir/Dialect/ArmSME/Transforms/Passes.h"
#include "mlir/Dialect/ArmSME/Utils/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Func/Transforms/FuncConversions.h"
#include "mlir/Dialect/Index/IR/IndexDialect.h"
#include "mlir/Dialect/Index/IR/IndexOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Vector/Utils/VectorUtils.h"
#include "mlir/Transforms/DialectConversion.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ArmSME/IR/ArmSME.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h`, `mlir/Dialect/ArmSME/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ArmSME/IR/ArmSME.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h`, `mlir/Dialect/ArmSME/Utils/Utils.h`。

### Lines 31-45
```cpp
#define DEBUG_TYPE "arm-sme-vector-legalization"

namespace mlir::arm_sme {
#define GEN_PASS_DEF_VECTORLEGALIZATION
#include "mlir/Dialect/ArmSME/Transforms/Passes.h.inc"
} // namespace mlir::arm_sme

using namespace mlir;
using namespace mlir::arm_sme;

namespace {

//===----------------------------------------------------------------------===//
// Decomposition of vector operations larger than an SME tile
//===----------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `mlir::arm_sme`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arm_sme` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 46-59
```cpp

// Common match failure reasons.
static constexpr StringLiteral kMatchFailureNotSMETileTypeMultiple(
    "op vector size is not multiple of SME tiles");
static constexpr StringLiteral kMatchFailureUnsupportedMaskOp(
    "op mask is unsupported for legalization/decomposition");
static constexpr StringLiteral
    kMatchFailureNonPermutationMap("op affine map is not a permutation");
static constexpr StringLiteral kMatchFailureNotIllegalToLegal(
    "expected transpose from illegal type to legal type");

/// An SMESubTile represents a single SME-sized sub-tile from decomposing a
/// larger vector type. The (`row`, `col`) are the position of the tile in the
/// original vector type. For example for an [8]x[8] tile with four [4]x[4]
```
- **EN**: Implements logic around `kMatchFailureNotSMETileTypeMultiple`, `kMatchFailureUnsupportedMaskOp`, `kMatchFailureNonPermutationMap`, `kMatchFailureNotIllegalToLegal`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `kMatchFailureNotSMETileTypeMultiple`, `kMatchFailureUnsupportedMaskOp`, `kMatchFailureNonPermutationMap`, `kMatchFailureNotIllegalToLegal` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 60-77
```cpp
/// sub-tiles, we would have:
///
///           8 x vscale
/// ┌─────────────┬─────────────┐
/// │(0,0)        │(0,4)        │
/// │             │             │
/// ├─────────────┼─────────────┤ 8 x vscale
/// │(4,0)        │(4,4)        │
/// │             │             │
/// └─────────────┴─────────────┘
struct SMESubTile {
  // Note: The units of (row, col) are vscale (as SME tiles are scalable).
  int row{0};
  int col{0};
  // The SME tile type.
  VectorType type;
};

```
- **EN**: Introduces declarations for `SMESubTile`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SMESubTile` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 78-95
```cpp
/// Adds a constant elementwise scalable offset to `indices` (which are of equal
/// length). For example, in the 2D case this would return:
// { indices[0] + offset[0] * vscale, indices[1] + offset[1] *  vscale }
SmallVector<Value, 2> addConstantScalableOffset(OpBuilder &builder,
                                                Location loc,
                                                ValueRange indices,
                                                ArrayRef<int> scalableOffsets) {
  auto vscale = vector::VectorScaleOp::create(builder, loc);
  return llvm::map_to_vector(
      llvm::zip_equal(indices, scalableOffsets), [&](auto pair) -> Value {
        auto [index, base] = pair;
        auto offset = arith::MulIOp::create(
            builder, loc, arith::ConstantIndexOp::create(builder, loc, base),
            vscale);
        return arith::AddIOp::create(builder, loc, index, offset);
      });
}

```
- **EN**: Implements logic around `addConstantScalableOffset`, `create`, `map_to_vector`, `zip_equal`.
- **CN**: 围绕 `addConstantScalableOffset`, `create`, `map_to_vector`, `zip_equal` 实现具体逻辑。

### Lines 96-109
```cpp
/// Adjusts `indices` (e.g. from a load/store) for a larger vector type to
/// indices for one of the SME sub-tiles it will decompose into.
///
/// For example, if you were to decompose an 8x8 load into four 4x4 tiles, the
/// indices for each tile would need to be adjusted as follows:
///
/// initial indices = [a,b], inital size = 8x8, target size = 4x4
/// ┌─────────────┬─────────────┐
/// │[a,b]        │[a,b+4]      │
/// │             │             │
/// ├─────────────┼─────────────┤
/// │[a+4,b]      │[a+4,b+4]    │
/// │             │             │
/// └─────────────┴─────────────┘
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 110-123
```cpp
SmallVector<Value, 2> getSMESubTileIndices(OpBuilder &builder, Location loc,
                                           ValueRange indices,
                                           SMESubTile smeTile) {
  return addConstantScalableOffset(builder, loc, indices,
                                   {smeTile.row, smeTile.col});
}

/// Returns true if `mask` is generated by an operation that can be decomposed
/// for SME. Currently, that is just no mask, or vector.create_mask.
/// TODO: Add support for vector.constant_mask once required for SME.
bool isSupportedMaskOp(Value mask) {
  return !mask || mask.getDefiningOp<vector::CreateMaskOp>();
}

```
- **EN**: Implements logic around `getSMESubTileIndices`, `addConstantScalableOffset`, `isSupportedMaskOp`, `CreateMaskOp>`.
- **CN**: 围绕 `getSMESubTileIndices`, `addConstantScalableOffset`, `isSupportedMaskOp`, `CreateMaskOp>` 实现具体逻辑。

### Lines 124-140
```cpp
/// Extracts a mask for an SME sub-tile from the mask of a larger vector type.
Value extractSMEMask(OpBuilder &builder, Location loc, Value mask,
                     SMESubTile smeTile) {
  assert(isSupportedMaskOp(mask));
  if (!mask)
    return Value{};
  auto createMask = mask.getDefiningOp<vector::CreateMaskOp>();
  // The operands of `vector.create_mask` (from a 2D perspective) are the
  // coordinates where the mask ends. So we subtract where this tile starts,
  // from the mask operands to get the parameters for this sub-tile.
  auto smeTileMaskDims = addConstantScalableOffset(
      builder, loc, createMask.getOperands(), {-smeTile.row, -smeTile.col});
  auto smeTileCreateMask = vector::CreateMaskOp::create(
      builder, loc, smeTile.type.clone(builder.getI1Type()), smeTileMaskDims);
  return smeTileCreateMask.getResult();
}

```
- **EN**: Implements logic around `extractSMEMask`, `assert`, `CreateMaskOp>`, `addConstantScalableOffset`, and 4 more symbols.
- **CN**: 围绕 `extractSMEMask`, `assert`, `CreateMaskOp>`, `addConstantScalableOffset`, and 4 more symbols 实现具体逻辑。

### Lines 141-161
```cpp
/// Constructs an iterator that returns each SME tile (with coordinates)
/// contained within a VectorType. For example, if decomposing an [8]x[8] into
/// [4]x[4] tiles, the iterator would yield the tiles: (0, 0), (0, 4), (4, 0),
/// (4, 4).
auto decomposeToSMETiles(OpBuilder &builder, VectorType type,
                         VectorType smeTileType,
                         bool transposeIndices = false) {
  return llvm::map_range(
      StaticTileOffsetRange(
          type.getShape(),
          {std::min(type.getDimSize(0), smeTileType.getDimSize(0)),
           std::min(type.getDimSize(1), smeTileType.getDimSize(1))}),
      [=](auto indices) {
        int row = int(indices[0]);
        int col = int(indices[1]);
        if (transposeIndices)
          std::swap(row, col);
        return SMESubTile{row, col, smeTileType};
      });
}

```
- **EN**: Implements logic around `decomposeToSMETiles`, `map_range`, `StaticTileOffsetRange`, `getShape`, and 3 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `decomposeToSMETiles`, `map_range`, `StaticTileOffsetRange`, `getShape`, and 3 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 162-175
```cpp
/// Returns the number of SME tiles that fit into the (2D-scalable) vector type
/// `type`.
int getNumberOfSMETilesForVectorType(VectorType type) {
  assert(isMultipleOfSMETileVectorType(type) &&
         "`type` not multiple of SME tiles");
  int64_t vectorRows = type.getDimSize(0);
  int64_t vectorCols = type.getDimSize(1);
  auto elementType = type.getElementType();
  unsigned minNumElts = getSMETileSliceMinNumElts(elementType);
  return (vectorRows * vectorCols) / (minNumElts * minNumElts);
}

/// Legalize `arith.constant dense<value>` splat operations to fit within SME
/// tiles by decomposing them into tile-sized operations.
```
- **EN**: Implements logic around `getNumberOfSMETilesForVectorType`, `assert`, `getDimSize`, `getElementType`, and 1 more symbols.
- **CN**: 围绕 `getNumberOfSMETilesForVectorType`, `assert`, `getDimSize`, `getElementType`, and 1 more symbols 实现具体逻辑。

### Lines 176-191
```cpp
struct LegalizeArithConstantOpsByDecomposition
    : public OpConversionPattern<arith::ConstantOp> {
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(arith::ConstantOp constantOp, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto vectorType = dyn_cast<VectorType>(constantOp.getType());
    auto denseAttr = dyn_cast<DenseElementsAttr>(constantOp.getValueAttr());
    if (!vectorType || !denseAttr || !denseAttr.isSplat())
      return failure();

    if (!isMultipleOfSMETileVectorType(vectorType))
      return rewriter.notifyMatchFailure(constantOp,
                                         kMatchFailureNotSMETileTypeMultiple);

```
- **EN**: Introduces declarations for `LegalizeArithConstantOpsByDecomposition`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeArithConstantOpsByDecomposition` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 192-205
```cpp
    auto smeTileType = getSMETileTypeForElement(vectorType.getElementType());
    auto tileCount = getNumberOfSMETilesForVectorType(vectorType);
    auto tileSplat = arith::ConstantOp::create(
        rewriter, constantOp.getLoc(), denseAttr.resizeSplat(smeTileType));
    SmallVector<Value> repl(tileCount, tileSplat);
    rewriter.replaceOpWithMultiple(constantOp, {repl});

    return success();
  }
};

/// Legalize `vector.outerproduct` operations to fit within SME tiles by
/// decomposing them into tile-sized operations.
struct LegalizeVectorOuterProductOpsByDecomposition
```
- **EN**: Introduces declarations for `LegalizeVectorOuterProductOpsByDecomposition`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeVectorOuterProductOpsByDecomposition` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 206-227
```cpp
    : public OpConversionPattern<vector::OuterProductOp> {
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::OuterProductOp outerProductOp,
                  OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto vectorType = outerProductOp.getResultVectorType();
    if (!isMultipleOfSMETileVectorType(vectorType))
      return rewriter.notifyMatchFailure(outerProductOp,
                                         kMatchFailureNotSMETileTypeMultiple);

    Value mask;
    Operation *rootOp = outerProductOp;
    auto loc = outerProductOp.getLoc();
    if (outerProductOp.isMasked()) {
      auto maskOp = outerProductOp.getMaskingOp();
      mask = maskOp.getMask();
      rootOp = maskOp;
      rewriter.setInsertionPoint(rootOp);
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `getResultVectorType`, `isMultipleOfSMETileVectorType`, `notifyMatchFailure`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getResultVectorType`, `isMultipleOfSMETileVectorType`, `notifyMatchFailure`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 228-249
```cpp
    if (!isSupportedMaskOp(mask))
      return rewriter.notifyMatchFailure(outerProductOp,
                                         kMatchFailureUnsupportedMaskOp);

    ValueRange accSMETiles = adaptor.getAcc();
    auto smeTileType = getSMETileTypeForElement(vectorType.getElementType());
    VectorType sliceType = VectorType::Builder(smeTileType).dropDim(0);

    SmallVector<Value> resultSMETiles;
    for (auto [index, smeTile] : llvm::enumerate(
             decomposeToSMETiles(rewriter, vectorType, smeTileType))) {

      auto smeMask = extractSMEMask(rewriter, loc, mask, smeTile);
      auto lhs = vector::ScalableExtractOp::create(
          rewriter, loc, sliceType, outerProductOp.getLhs(), smeTile.row);
      auto rhs = vector::ScalableExtractOp::create(
          rewriter, loc, sliceType, outerProductOp.getRhs(), smeTile.col);
      auto smeOuterProduct = vector::OuterProductOp::create(
          rewriter, loc, smeTileType, lhs, rhs,
          !accSMETiles.empty() ? accSMETiles[index] : Value{},
          outerProductOp.getKind());

```
- **EN**: Implements logic around `isSupportedMaskOp`, `notifyMatchFailure`, `getAcc`, `getSMETileTypeForElement`, and 9 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isSupportedMaskOp`, `notifyMatchFailure`, `getAcc`, `getSMETileTypeForElement`, and 9 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 250-265
```cpp
      auto *maskedOuterProduct =
          vector::maskOperation(rewriter, smeOuterProduct, smeMask);
      resultSMETiles.push_back(maskedOuterProduct->getResult(0));
    }

    rewriter.replaceOpWithMultiple(rootOp, {resultSMETiles});
    return success();
  }
};

// Workaround for `vector.mask`. We want to match on `vector.outerproduct` (to
// get the help of the type conversion), but doing so results in the type
// conversion adding target materializations in the `vector.mask` region
// (invalid). This pattern matches on `vector.mask` then calls into the
// `vector.outerproduct` pattern to work around this issue.
struct LegalizeMaskedVectorOuterProductOpsByDecomposition
```
- **EN**: Introduces declarations for `LegalizeMaskedVectorOuterProductOpsByDecomposition`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeMaskedVectorOuterProductOpsByDecomposition` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 266-282
```cpp
    : public OpConversionPattern<vector::MaskOp> {
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::MaskOp maskOp, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    if (auto outerProductOp = llvm::dyn_cast_or_null<vector::OuterProductOp>(
            maskOp.getMaskableOp())) {
      LegalizeVectorOuterProductOpsByDecomposition pattern(*getTypeConverter(),
                                                           getContext());
      return static_cast<RewritePattern &>(pattern).matchAndRewrite(
          outerProductOp, rewriter);
    }
    return failure();
  }
};

```
- **EN**: Implements logic around `matchAndRewrite`, `OuterProductOp>`, `getMaskableOp`, `pattern`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `OuterProductOp>`, `getMaskableOp`, `pattern`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 283-296
```cpp
/// Legalize `vector.transfer_read` operations to fit within SME tiles by
/// decomposing them into tile-sized operations.
struct LegalizeTransferReadOpsByDecomposition
    : public OpConversionPattern<vector::TransferReadOp> {
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::TransferReadOp readOp, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto vectorType = readOp.getVectorType();
    if (!isMultipleOfSMETileVectorType(vectorType))
      return rewriter.notifyMatchFailure(readOp,
                                         kMatchFailureNotSMETileTypeMultiple);

```
- **EN**: Introduces declarations for `LegalizeTransferReadOpsByDecomposition`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeTransferReadOpsByDecomposition` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 297-310
```cpp
    auto mask = readOp.getMask();
    if (!isSupportedMaskOp(mask))
      return rewriter.notifyMatchFailure(readOp,
                                         kMatchFailureUnsupportedMaskOp);

    auto permutationMap = readOp.getPermutationMap();
    if (!permutationMap.isPermutation())
      return rewriter.notifyMatchFailure(readOp,
                                         kMatchFailureNonPermutationMap);

    // Note: For 2D vector types the only non-identity permutation is a simple
    // transpose [1, 0].
    bool transposed = !permutationMap.isIdentity();

```
- **EN**: Implements logic around `getMask`, `isSupportedMaskOp`, `notifyMatchFailure`, `getPermutationMap`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getMask`, `isSupportedMaskOp`, `notifyMatchFailure`, `getPermutationMap`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 311-325
```cpp
    auto loc = readOp.getLoc();
    auto smeTileType = getSMETileTypeForElement(vectorType.getElementType());

    SmallVector<Value> resultSMETiles;
    for (SMESubTile smeTile :
         decomposeToSMETiles(rewriter, vectorType, smeTileType, transposed)) {
      auto smeMask = extractSMEMask(rewriter, loc, mask, smeTile);
      auto smeRead = vector::TransferReadOp::create(
          rewriter, loc, smeTileType, readOp.getBase(),
          getSMESubTileIndices(rewriter, loc, readOp.getIndices(), smeTile),
          readOp.getPermutationMapAttr(), readOp.getPadding(), smeMask,
          readOp.getInBoundsAttr());
      resultSMETiles.push_back(smeRead);
    }

```
- **EN**: Implements logic around `getLoc`, `getSMETileTypeForElement`, `decomposeToSMETiles`, `extractSMEMask`, and 6 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `getSMETileTypeForElement`, `decomposeToSMETiles`, `extractSMEMask`, and 6 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 326-344
```cpp
    rewriter.replaceOpWithMultiple(readOp, {resultSMETiles});
    return success();
  }
};

/// Legalize `vector.transfer_write` operations to fit within SME tiles by
/// decomposing them into tile-sized operations.
struct LegalizeTransferWriteOpsByDecomposition
    : public OpConversionPattern<vector::TransferWriteOp> {
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::TransferWriteOp writeOp, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto vectorType = writeOp.getVectorType();
    if (!isMultipleOfSMETileVectorType(vectorType))
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureNotSMETileTypeMultiple);

```
- **EN**: Introduces declarations for `LegalizeTransferWriteOpsByDecomposition`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeTransferWriteOpsByDecomposition` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 345-358
```cpp
    auto mask = writeOp.getMask();
    if (!isSupportedMaskOp(mask))
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureUnsupportedMaskOp);

    auto permutationMap = writeOp.getPermutationMap();
    if (!permutationMap.isPermutation())
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureNonPermutationMap);

    // Note: For 2D vector types the only non-identity permutation is a simple
    // transpose [1, 0].
    bool transposed = !permutationMap.isIdentity();

```
- **EN**: Implements logic around `getMask`, `isSupportedMaskOp`, `notifyMatchFailure`, `getPermutationMap`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getMask`, `isSupportedMaskOp`, `notifyMatchFailure`, `getPermutationMap`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 359-374
```cpp
    auto loc = writeOp.getLoc();
    auto smeTileType = getSMETileTypeForElement(vectorType.getElementType());
    auto inputSMETiles = adaptor.getValueToStore();

    Value destTensorOrMemref = writeOp.getBase();
    for (auto [index, smeTile] : llvm::enumerate(decomposeToSMETiles(
             rewriter, vectorType, smeTileType, transposed))) {
      auto smeMask = extractSMEMask(rewriter, loc, mask, smeTile);
      auto smeWrite = vector::TransferWriteOp::create(
          rewriter, loc, inputSMETiles[index], destTensorOrMemref,
          getSMESubTileIndices(rewriter, loc, writeOp.getIndices(), smeTile),
          writeOp.getPermutationMapAttr(), smeMask, writeOp.getInBoundsAttr());
      if (writeOp.hasPureTensorSemantics())
        destTensorOrMemref = smeWrite.getResult();
    }

```
- **EN**: Implements logic around `getLoc`, `getSMETileTypeForElement`, `getValueToStore`, `getBase`, and 7 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `getSMETileTypeForElement`, `getValueToStore`, `getBase`, and 7 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 375-388
```cpp
    if (writeOp.hasPureTensorSemantics())
      rewriter.replaceOp(writeOp, destTensorOrMemref);
    else
      rewriter.eraseOp(writeOp);

    return success();
  }
};

/// Legalize a multi-tile transfer_write as a single store loop. This is done as
/// part of type decomposition as at this level we know each tile write is
/// disjoint, but that information is lost after decomposition (without analysis
/// to reconstruct it).
///
```
- **EN**: Implements logic around `hasPureTensorSemantics`, `replaceOp`, `eraseOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `hasPureTensorSemantics`, `replaceOp`, `eraseOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 389-402
```cpp
/// Example (pseudo-MLIR):
///
/// ```
/// vector.transfer_write %vector, %dest[%y, %x], %mask
///   : vector<[16]x[8]xi16>, memref<?x?xi16>
/// ```
/// Is rewritten to:
/// ```
/// scf.for %slice_idx = %c0 to %c8_vscale step %c1 {
///   %upper_slice_mask = vector.extract %mask[%slice_idx] ─┐
///     : vector<[8]xi1> from vector<[16]x[8]xi1>           |
///   %upper_slice = vector.extract %upper_tile[%slice_idx] |- Store upper tile
///     : vector<[8]xi16> from vector<[8]x[8]xi16>          |
///   vector.transfer_write %upper_slice,                   |
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 403-418
```cpp
///     %dest[%slice_idx + %y, %x], %upper_slice_mask       |
///     : vector<[8]xi16>, memref<?x?xi16>                  ┘
///   %lower_slice_idx = %slice_idx + %c8_vscale                 ─┐
///   %lower_slice_mask = vector.extract %mask[%lower_slice_idx]  |
///     : vector<[8]xi1> from vector<[16]x[8]xi1>                 |
///   %lower_slice = vector.extract %lower_tile[%slice_idx]       |- Store lower
///     : vector<[8]xi16> from vector<[8]x[8]xi16>                |  tile
///   vector.transfer_write %lower_slice,                         |
///     %dest[%lower_slice_idx + %y, %x], %lower_slice_mask       |
///     : vector<[8]xi16>, memref<?x?xi16>                        ┘
/// }
/// ```
struct LegalizeMultiTileTransferWriteAsStoreLoop
    : public OpConversionPattern<vector::TransferWriteOp> {
  using OpConversionPattern::OpConversionPattern;

```
- **EN**: Introduces declarations for `LegalizeMultiTileTransferWriteAsStoreLoop`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LegalizeMultiTileTransferWriteAsStoreLoop` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 419-435
```cpp
  LogicalResult
  matchAndRewrite(vector::TransferWriteOp writeOp, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    if (writeOp.hasPureTensorSemantics())
      return rewriter.notifyMatchFailure(
          writeOp, "TODO: tensor semantics are unsupported");

    auto permutationMap = writeOp.getPermutationMap();
    if (!permutationMap.isPermutation())
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureNonPermutationMap);

    bool transposed = !permutationMap.isIdentity();
    if (transposed)
      return rewriter.notifyMatchFailure(writeOp,
                                         "TODO: transpose unsupported");

```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `notifyMatchFailure`, `getPermutationMap`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `notifyMatchFailure`, `getPermutationMap`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 436-452
```cpp
    auto vectorType = writeOp.getVectorType();
    if (!isMultipleOfSMETileVectorType(vectorType))
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureNotSMETileTypeMultiple);

    // Note: We also disallow masks where any dimension is > 16 because that
    // prevents the masking from being lowered to use arm_sve.psel.
    auto mask = writeOp.getMask();
    if (!isSupportedMaskOp(mask) || (mask && (vectorType.getDimSize(0) > 16 ||
                                              vectorType.getDimSize(1) > 16)))
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureUnsupportedMaskOp);

    auto loc = writeOp.getLoc();
    auto createVscaleMultiple =
        vector::makeVscaleConstantBuilder(rewriter, loc);

```
- **EN**: Implements logic around `getVectorType`, `isMultipleOfSMETileVectorType`, `notifyMatchFailure`, `getMask`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getVectorType`, `isMultipleOfSMETileVectorType`, `notifyMatchFailure`, `getMask`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 453-466
```cpp
    // Get SME tile and slice types.
    auto smeTileType = getSMETileTypeForElement(vectorType.getElementType());
    auto minTileSlices = smeTileType.getDimSize(0);
    VectorType sliceMaskType =
        VectorType::get(minTileSlices, rewriter.getI1Type(), true);

    // Create loop over all tile slices.
    auto lowerBound = arith::ConstantIndexOp::create(rewriter, loc, 0);
    auto upperBound = createVscaleMultiple(minTileSlices);
    auto step = arith::ConstantIndexOp::create(rewriter, loc, 1);
    auto storeLoop =
        scf::ForOp::create(rewriter, loc, lowerBound, upperBound, step);
    rewriter.setInsertionPointToStart(storeLoop.getBody());

```
- **EN**: Implements logic around `getSMETileTypeForElement`, `getDimSize`, `get`, `create`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getSMETileTypeForElement`, `getDimSize`, `get`, `create`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 467-485
```cpp
    // For each sub-tile of the multi-tile `vectorType`.
    auto inputSMETiles = adaptor.getValueToStore();
    auto tileSliceIndex = storeLoop.getInductionVar();
    for (auto [index, smeTile] : llvm::enumerate(
             decomposeToSMETiles(rewriter, vectorType, smeTileType))) {
      // The coordinates of the tile within `vectorType`.
      auto tileRow = createVscaleMultiple(smeTile.row);
      auto tileCol = createVscaleMultiple(smeTile.col);

      // The current slice of `vectorType` we are processing.
      auto sliceIndex =
          arith::AddIOp::create(rewriter, loc, tileRow, tileSliceIndex);

      // Where in the destination memref the current slice will be stored.
      auto storeRow = arith::AddIOp::create(rewriter, loc, sliceIndex,
                                            writeOp.getIndices()[0]);
      auto storeCol = arith::AddIOp::create(rewriter, loc, tileCol,
                                            writeOp.getIndices()[1]);

```
- **EN**: Implements logic around `getValueToStore`, `getInductionVar`, `enumerate`, `decomposeToSMETiles`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getValueToStore`, `getInductionVar`, `enumerate`, `decomposeToSMETiles`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 486-508
```cpp
      // Extract the mask for the current slice.
      Value sliceMask = nullptr;
      if (mask) {
        sliceMask = vector::ExtractOp::create(rewriter, loc, mask,
                                              OpFoldResult(sliceIndex));
        if (sliceMaskType != sliceMask.getType())
          sliceMask = vector::ScalableExtractOp::create(
              rewriter, loc, sliceMaskType, sliceMask, smeTile.col);
      }

      // Extract and store the current slice.
      Value tile = inputSMETiles[index];
      auto slice =
          vector::ExtractOp::create(rewriter, loc, tile, tileSliceIndex);
      vector::TransferWriteOp::create(
          rewriter, loc, slice, writeOp.getBase(),
          ValueRange{storeRow, storeCol},
          AffineMapAttr::get(writeOp.getPermutationMap().dropResult(0)),
          sliceMask,
          rewriter.getBoolArrayAttr(
              ArrayRef<bool>(writeOp.getInBoundsValues()).drop_front()));
    }

```
- **EN**: Implements logic around `create`, `OpFoldResult`, `getType`, `getBase`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `OpFoldResult`, `getType`, `getBase`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 509-522
```cpp
    rewriter.eraseOp(writeOp);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ArmSME-specific fixup canonicalizations/folds
//===----------------------------------------------------------------------===//

/// Folds an extract from a 3D `vector.create_mask` (which is a vector of
/// SME-like masks), into a compare and a 2D `vector.create_mask`. This is
/// necessary for the mask to be lowered to ArmSME.
///
/// Example:
```
- **EN**: Implements logic around `eraseOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `eraseOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子。

### Lines 523-536
```cpp
///
///  BEFORE:
///  ```mlir
///  %mask = vector.create_mask %nonConstantDim, %a, %b : vector<4x[4]x[4]xi1>
///  %subMask = vector.extract %mask[2]
///          : vector<[4]x[4]xi1> from vector<4x[4]x[4]xi1>
///  ```
///
///  AFTER:
///  ```mlir
///  %extractionInTrueRegion = arith.cmpi slt, %c2, %nonConstantDim : index
///  %newMaskFrontDim = arith.select %extractionInTrueRegion, %a, %c0 : index
///  %subMask = vector.create_mask %newMaskFrontDim, %b : vector<[4]x[4]xi1>
///  ```
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 537-555
```cpp
struct FoldExtractFromVectorOfSMELikeCreateMasks
    : public OpRewritePattern<vector::ExtractOp> {
  using OpRewritePattern<vector::ExtractOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::ExtractOp extractOp,
                                PatternRewriter &rewriter) const override {
    auto loc = extractOp.getLoc();
    auto createMaskOp =
        extractOp.getSource().getDefiningOp<vector::CreateMaskOp>();
    if (!createMaskOp)
      return rewriter.notifyMatchFailure(
          extractOp, "extract not from vector.create_mask op");

    VectorType extractedMaskType =
        llvm::dyn_cast<VectorType>(extractOp.getResult().getType());
    if (!extractedMaskType)
      return rewriter.notifyMatchFailure(extractOp,
                                         "extracted type is not a vector type");

```
- **EN**: Introduces declarations for `FoldExtractFromVectorOfSMELikeCreateMasks`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FoldExtractFromVectorOfSMELikeCreateMasks` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 556-571
```cpp
    auto numScalable = extractedMaskType.getNumScalableDims();
    if (numScalable != 2)
      return rewriter.notifyMatchFailure(
          extractOp, "expected extracted type to be an SME-like mask");

    // TODO: Support multiple extraction indices.
    if (extractOp.getStaticPosition().size() != 1)
      return rewriter.notifyMatchFailure(
          extractOp, "only a single extraction index is supported");

    auto frontMaskDim = createMaskOp.getOperand(0);
    if (frontMaskDim.getDefiningOp<arith::ConstantOp>())
      return rewriter.notifyMatchFailure(
          extractOp,
          "constant vector.create_masks dims should be folded elsewhere");

```
- **EN**: Implements logic around `getNumScalableDims`, `notifyMatchFailure`, `getStaticPosition`, `getOperand`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getNumScalableDims`, `notifyMatchFailure`, `getStaticPosition`, `getOperand`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子。

### Lines 572-588
```cpp
    auto zero = arith::ConstantIndexOp::create(rewriter, loc, 0);
    auto extractionIndex = getValueOrCreateConstantIndexOp(
        rewriter, loc, extractOp.getMixedPosition()[0]);
    auto extractionInTrueRegion = arith::CmpIOp::create(
        rewriter, loc, rewriter.getI1Type(), arith::CmpIPredicate::slt,
        extractionIndex, frontMaskDim);
    auto newMaskFrontDim =
        arith::SelectOp::create(rewriter, loc, extractionInTrueRegion,
                                createMaskOp.getOperand(1), zero);

    rewriter.replaceOpWithNewOp<vector::CreateMaskOp>(
        extractOp, extractedMaskType,
        ValueRange{newMaskFrontDim, createMaskOp.getOperand(2)});
    return success();
  }
};

```
- **EN**: Implements logic around `create`, `getValueOrCreateConstantIndexOp`, `getMixedPosition`, `getI1Type`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `getValueOrCreateConstantIndexOp`, `getMixedPosition`, `getI1Type`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 589-602
```cpp
/// A vector type where no fixed dimension comes after a scalable dimension.
bool isLegalVectorType(VectorType vType) {
  bool seenFixedDim = false;
  for (bool scalableFlag : llvm::reverse(vType.getScalableDims())) {
    seenFixedDim |= !scalableFlag;
    if (seenFixedDim && scalableFlag)
      return false;
  }
  return true;
}

/// Lifts an illegal vector.transpose and vector.transfer_read to a
/// memref.subview + memref.transpose, followed by a legal read.
///
```
- **EN**: Implements logic around `isLegalVectorType`, `reverse`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `isLegalVectorType`, `reverse` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 603-616
```cpp
/// 'Illegal' here means a leading scalable dimension and a fixed trailing
/// dimension, which has no valid lowering.
///
/// The memref.transpose is metadata-only transpose that produces a strided
/// memref, which eventually becomes a loop reading individual elements.
///
/// Example:
///
///  BEFORE:
///  ```mlir
///  %illegalRead = vector.transfer_read %memref[%a, %b]
///                  : memref<?x?xf32>, vector<[8]x4xf32>
///  %legalType = vector.transpose %illegalRead, [1, 0]
///                  : vector<[8]x4xf32> to vector<4x[8]xf32>
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 617-631
```cpp
///  ```
///
///  AFTER:
///  ```mlir
///  %readSubview = memref.subview %memref[%a, %b] [%c8_vscale, %c4] [%c1, %c1]
///                  : memref<?x?xf32> to memref<?x?xf32>
///  %transpose = memref.transpose %readSubview (d0, d1) -> (d1, d0)
///                  : memref<?x?xf32> to memref<?x?xf32>
///  %legalType = vector.transfer_read %transpose[%c0, %c0]
///                  : memref<?x?xf32>, vector<4x[8]xf32>
///  ```
struct LiftIllegalVectorTransposeToMemory
    : public OpRewritePattern<vector::TransposeOp> {
  using OpRewritePattern<vector::TransposeOp>::OpRewritePattern;

```
- **EN**: Introduces declarations for `LiftIllegalVectorTransposeToMemory`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LiftIllegalVectorTransposeToMemory` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 632-645
```cpp
  static Value getExtensionSource(Operation *op) {
    if (isa_and_present<arith::ExtSIOp, arith::ExtUIOp, arith::ExtFOp>(op))
      return op->getOperand(0);
    return {};
  }

  LogicalResult matchAndRewrite(vector::TransposeOp transposeOp,
                                PatternRewriter &rewriter) const override {
    auto sourceType = transposeOp.getSourceVectorType();
    auto resultType = transposeOp.getResultVectorType();
    if (isLegalVectorType(sourceType) || !isLegalVectorType(resultType))
      return rewriter.notifyMatchFailure(transposeOp,
                                         kMatchFailureNotIllegalToLegal);

```
- **EN**: Implements logic around `getExtensionSource`, `ExtFOp>`, `getOperand`, `matchAndRewrite`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getExtensionSource`, `ExtFOp>`, `getOperand`, `matchAndRewrite`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 646-660
```cpp
    // Look through extend for transfer_read.
    Value maybeRead = transposeOp.getVector();
    auto *transposeSourceOp = maybeRead.getDefiningOp();
    Operation *extendOp = nullptr;
    if (Value extendSource = getExtensionSource(transposeSourceOp)) {
      maybeRead = extendSource;
      extendOp = transposeSourceOp;
    }

    auto illegalRead = maybeRead.getDefiningOp<vector::TransferReadOp>();
    if (!illegalRead)
      return rewriter.notifyMatchFailure(
          transposeOp,
          "expected source to be (possibly extended) transfer_read");

```
- **EN**: Implements logic around `getVector`, `getDefiningOp`, `getExtensionSource`, `TransferReadOp>`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getVector`, `getDefiningOp`, `getExtensionSource`, `TransferReadOp>`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 661-685
```cpp
    if (!illegalRead.getPermutationMap().isIdentity())
      return rewriter.notifyMatchFailure(
          illegalRead, "expected read to have identity permutation map");

    auto loc = transposeOp.getLoc();
    auto zero = arith::ConstantIndexOp::create(rewriter, loc, 0);
    auto one = arith::ConstantIndexOp::create(rewriter, loc, 1);

    // Create a subview that matches the size of the illegal read vector type.
    auto readType = illegalRead.getVectorType();
    auto readSizes = llvm::map_to_vector(
        llvm::zip_equal(readType.getShape(), readType.getScalableDims()),
        [&](auto dim) -> Value {
          auto [size, isScalable] = dim;
          auto dimSize = arith::ConstantIndexOp::create(rewriter, loc, size);
          if (!isScalable)
            return dimSize;
          auto vscale = vector::VectorScaleOp::create(rewriter, loc);
          return arith::MulIOp::create(rewriter, loc, vscale, dimSize);
        });
    SmallVector<Value> strides(readType.getRank(), Value(one));
    auto readSubview =
        memref::SubViewOp::create(rewriter, loc, illegalRead.getBase(),
                                  illegalRead.getIndices(), readSizes, strides);

```
- **EN**: Implements logic around `getPermutationMap`, `notifyMatchFailure`, `getLoc`, `create`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getPermutationMap`, `notifyMatchFailure`, `getLoc`, `create`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 686-708
```cpp
    // Apply the transpose to all values/attributes of the transfer_read:
    // - The mask
    Value mask = illegalRead.getMask();
    if (mask) {
      // Note: The transpose for the mask should fold into the
      // vector.create_mask/constant_mask op, which will then become legal.
      mask = vector::TransposeOp::create(rewriter, loc, mask,
                                         transposeOp.getPermutation());
    }
    // - The source memref
    mlir::AffineMap transposeMap = AffineMap::getPermutationMap(
        transposeOp.getPermutation(), getContext());
    auto transposedSubview = memref::TransposeOp::create(
        rewriter, loc, readSubview, AffineMapAttr::get(transposeMap));
    ArrayAttr inBoundsAttr = illegalRead.getInBoundsAttr();
    // - The `in_bounds` attribute
    if (inBoundsAttr) {
      SmallVector<Attribute> inBoundsValues(inBoundsAttr.begin(),
                                            inBoundsAttr.end());
      applyPermutationToVector(inBoundsValues, transposeOp.getPermutation());
      inBoundsAttr = rewriter.getArrayAttr(inBoundsValues);
    }

```
- **EN**: Implements logic around `getMask`, `create`, `getPermutation`, `getPermutationMap`, and 6 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getMask`, `create`, `getPermutation`, `getPermutationMap`, and 6 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 709-725
```cpp
    VectorType legalReadType = resultType.clone(readType.getElementType());
    // Note: The indices are all zero as the subview is already offset.
    SmallVector<Value> readIndices(illegalRead.getIndices().size(), zero);
    auto legalRead = vector::TransferReadOp::create(
        rewriter, loc, legalReadType, transposedSubview, readIndices,
        illegalRead.getPermutationMapAttr(), illegalRead.getPadding(), mask,
        inBoundsAttr);

    // Replace the transpose with the new read, extending the result if
    // necessary.
    rewriter.replaceOp(transposeOp, [&]() -> Operation * {
      if (extendOp)
        return rewriter.create(loc, extendOp->getName().getIdentifier(),
                               Value(legalRead), resultType);
      return legalRead;
    }());

```
- **EN**: Implements logic around `clone`, `readIndices`, `create`, `getPermutationMapAttr`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `clone`, `readIndices`, `create`, `getPermutationMapAttr`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 726-739
```cpp
    return success();
  }
};

/// Rewrites an illegal/unsupported SVE transfer_write(transpose) to instead use
/// the ZA state. This workaround rewrite to support these transposes when ZA is
/// available.
///
/// Example:
///
///  BEFORE:
///  ```mlir
///  %transpose = vector.transpose %vec, [1, 0]
///     : vector<2x[4]xf32> to vector<[4]x2xf32>
```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

### Lines 740-753
```cpp
///  vector.transfer_write %transpose, %dest[%y, %x]
///     : vector<[4]x2xf32>,  memref<?x?xf32>
///  ```
///
///  AFTER:
///  ```mlir
///   %0 = arm_sme.get_tile : vector<[4]x[4]xf32>
///   %1 = vector.extract %vec[0] : vector<[4]xf32> from vector<2x[4]xf32>
///   %2 = vector.insert %1, %0 [0] : vector<[4]xf32> into vector<[4]x[4]xf32>
///   %3 = vector.extract %vec[1] : vector<[4]xf32> from vector<2x[4]xf32>
///   %4 = vector.insert %3, %2 [1] : vector<[4]xf32> into vector<[4]x[4]xf32>
///   %c4_vscale = arith.muli %vscale, %c4 : index
///   %mask = vector.create_mask %c4_vscale, %c2 : vector<[4]x[4]xi1>
///   vector.transfer_write %4, %dest[%y, %x], %mask
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 754-768
```cpp
///      {permutation_map = affine_map<(d0, d1) -> (d1, d0)>}
///      : vector<[4]x[4]xf32>, memref<?x?xf32>
///  ```
///
/// Values larger than a single tile are supported via decomposition.
struct LowerIllegalTransposeStoreViaZA
    : public OpRewritePattern<vector::TransferWriteOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::TransferWriteOp writeOp,
                                PatternRewriter &rewriter) const override {
    if (!isSupportedMaskOp(writeOp.getMask()))
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureUnsupportedMaskOp);

```
- **EN**: Introduces declarations for `LowerIllegalTransposeStoreViaZA`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LowerIllegalTransposeStoreViaZA` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 769-783
```cpp
    auto permutationMap = writeOp.getPermutationMap();
    if (!permutationMap.isIdentity())
      return rewriter.notifyMatchFailure(writeOp,
                                         kMatchFailureNonPermutationMap);

    auto transposeOp = writeOp.getVector().getDefiningOp<vector::TransposeOp>();
    if (!transposeOp)
      return failure();

    auto sourceType = transposeOp.getSourceVectorType();
    auto resultType = transposeOp.getResultVectorType();

    if (resultType.getRank() != 2)
      return rewriter.notifyMatchFailure(transposeOp, "TransposeOp not rank 2");

```
- **EN**: Implements logic around `getPermutationMap`, `isIdentity`, `notifyMatchFailure`, `getVector`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getPermutationMap`, `isIdentity`, `notifyMatchFailure`, `getVector`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 784-798
```cpp
    if (!isLegalVectorType(sourceType) || isLegalVectorType(resultType))
      return rewriter.notifyMatchFailure(
          transposeOp, "not illegal/unsupported SVE transpose");

    auto smeTileType = getSMETileTypeForElement(resultType.getElementType());
    VectorType smeSliceType = VectorType::Builder(smeTileType).dropDim(0);

    if (sourceType.getDimSize(0) <= 1 ||
        sourceType.getDimSize(1) % smeSliceType.getDimSize(0) != 0)
      return rewriter.notifyMatchFailure(writeOp, "unsupported source shape");

    auto loc = writeOp.getLoc();
    auto createVscaleMultiple =
        vector::makeVscaleConstantBuilder(rewriter, loc);

```
- **EN**: Implements logic around `isLegalVectorType`, `notifyMatchFailure`, `getSMETileTypeForElement`, `Builder`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isLegalVectorType`, `notifyMatchFailure`, `getSMETileTypeForElement`, `Builder`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 799-826
```cpp
    auto transposeMap = AffineMapAttr::get(
        AffineMap::getPermutationMap(ArrayRef<int64_t>{1, 0}, getContext()));

    // Note: We need to use `get_tile` as there's no vector-level `undef`.
    Value undefTile = arm_sme::GetTileOp::create(rewriter, loc, smeTileType);
    Value destTensorOrMemref = writeOp.getBase();
    auto numSlicesPerTile =
        std::min(sourceType.getDimSize(0), smeTileType.getDimSize(0));
    auto numSlices =
        arith::ConstantIndexOp::create(rewriter, loc, numSlicesPerTile);
    for (auto [index, smeTile] : llvm::enumerate(
             decomposeToSMETiles(rewriter, sourceType, smeTileType))) {
      // 1. _Deliberately_ drop a scalable dimension and insert a fixed number
      // of slices from the source type into the SME tile. Without checking
      // vscale (and emitting multiple implementations) we can't make use of the
      // rows of the tile after 1*vscale rows.
      Value tile = undefTile;
      for (int d = 0; d < numSlicesPerTile; ++d) {
        Value vector =
            vector::ExtractOp::create(rewriter, loc, transposeOp.getVector(),
                                      rewriter.getIndexAttr(d + smeTile.row));
        if (vector.getType() != smeSliceType) {
          vector = vector::ScalableExtractOp::create(
              rewriter, loc, smeSliceType, vector, smeTile.col);
        }
        tile = vector::InsertOp::create(rewriter, loc, vector, tile, d);
      }

```
- **EN**: Implements logic around `get`, `getPermutationMap`, `create`, `getBase`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `get`, `getPermutationMap`, `create`, `getBase`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 827-849
```cpp
      // 2. Transpose the tile position.
      auto transposedRow = createVscaleMultiple(smeTile.col);
      auto transposedCol =
          arith::ConstantIndexOp::create(rewriter, loc, smeTile.row);

      // 3. Compute mask for tile store.
      Value maskRows;
      Value maskCols;
      if (auto mask = writeOp.getMask()) {
        auto createMask = mask.getDefiningOp<vector::CreateMaskOp>();
        maskRows = arith::SubIOp::create(
            rewriter, loc, createMask.getOperand(0), transposedRow);
        maskCols = arith::SubIOp::create(
            rewriter, loc, createMask.getOperand(1), transposedCol);
        maskCols = index::MinSOp::create(rewriter, loc, maskCols, numSlices);
      } else {
        maskRows = createVscaleMultiple(smeTileType.getDimSize(0));
        maskCols = numSlices;
      }
      auto subMask = vector::CreateMaskOp::create(
          rewriter, loc, smeTileType.clone(rewriter.getI1Type()),
          ValueRange{maskRows, maskCols});

```
- **EN**: Implements logic around `createVscaleMultiple`, `create`, `getMask`, `CreateMaskOp>`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `createVscaleMultiple`, `create`, `getMask`, `CreateMaskOp>`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 850-863
```cpp
      // 4. Emit a transposed tile write.
      auto writeIndices = writeOp.getIndices();
      Value destRow =
          arith::AddIOp::create(rewriter, loc, transposedRow, writeIndices[0]);
      Value destCol =
          arith::AddIOp::create(rewriter, loc, transposedCol, writeIndices[1]);
      auto smeWrite = vector::TransferWriteOp::create(
          rewriter, loc, tile, destTensorOrMemref, ValueRange{destRow, destCol},
          transposeMap, subMask, writeOp.getInBounds());

      if (writeOp.hasPureTensorSemantics())
        destTensorOrMemref = smeWrite.getResult();
    }

```
- **EN**: Implements logic around `getIndices`, `create`, `getInBounds`, `hasPureTensorSemantics`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getIndices`, `create`, `getInBounds`, `hasPureTensorSemantics`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 864-877
```cpp
    if (writeOp.hasPureTensorSemantics())
      rewriter.replaceOp(writeOp, destTensorOrMemref);
    else
      rewriter.eraseOp(writeOp);

    return success();
  }
};

/// Lower `vector.transfer_read` of a scalable column to `scf::for`
///
/// Lowers a "read" of a scalable column from a MemRef for which there is no
/// hardware pperation that we could use to a loop over the rows to read and
/// loads one element at a time.
```
- **EN**: Implements logic around `hasPureTensorSemantics`, `replaceOp`, `eraseOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `hasPureTensorSemantics`, `replaceOp`, `eraseOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 878-891
```cpp
///
///  BEFORE:
///  ```
///  %res = vector.transfer_read %mem[%a, %b] (...)
///    : memref<?x?xf32>, vector<[4]x1xf32>
///  ```
///
///  AFTER:
///  ```
///    %cst = arith.constant (...) : vector<[4]xf32>
///    %vscale = vector.vscale
///    %c4_vscale = arith.muli %vscale, %c4 : index
///    %scf = scf.for %lb = %c0 to %c4_vscale step %c1 iter_args(%arg4 = %cst)
///      -> (vector<[4]xf32>) {
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 892-907
```cpp
///
///        %load = memref.load %mem[%arg3 + %a, %b] : memref<?x?xf32>
///        %vec = vector.insert %load, %cst [%arg3] : f32 into vector<[4]xf32>
///        scf.yield %vec : vector<[4]xf32>
///    }
///    %res = vector.shape_cast %scf : vector<[4]xf32> to vector<[4]x1xf32>
///  ```
///
///  TODO: This transformation isn't specific to SME - move it to the SVE
///  dialect.
///  TODO: Check the in_bounds attribute and generate vector.maskedload if
///  required.
struct LowerColumnTransferReadToLoops
    : public OpRewritePattern<vector::TransferReadOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `LowerColumnTransferReadToLoops`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LowerColumnTransferReadToLoops` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 908-922
```cpp
  LogicalResult matchAndRewrite(vector::TransferReadOp readOp,
                                PatternRewriter &rewriter) const override {
    // NOTE: This is a fairly low-level transformation, so we shouldn't be
    // adding support for Tensors without good rationale.
    if (readOp.hasPureTensorSemantics())
      return rewriter.notifyMatchFailure(
          readOp, "Tensor semantics are unsupported (either bufferize or "
                  "extend this pattern)");

    auto resType = readOp.getVectorType();

    if (resType.getRank() != 2)
      return rewriter.notifyMatchFailure(readOp,
                                         "Only 2D vectors are supported!");

```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `notifyMatchFailure`, `unsupported`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `notifyMatchFailure`, `unsupported`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 923-937
```cpp
    if (resType.getShape()[1] != 1)
      return rewriter.notifyMatchFailure(
          readOp, "The trailing output dim is != 1 (not supported ATM)");

    if (!resType.getScalableDims()[0] || resType.getScalableDims()[1])
      return rewriter.notifyMatchFailure(
          readOp, "Expected the leading dim to be scalable and the trailing "
                  "dim to be fixed.");

    // Create new result type - similar to the original vector with the
    // trailing unit dim collapsed.
    int64_t numRows = resType.getShape()[0];
    VectorType newResType = VectorType::get(numRows, resType.getElementType(),
                                            /*scalableDims=*/{true});

```
- **EN**: Implements logic around `getShape`, `notifyMatchFailure`, `getScalableDims`, `get`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getShape`, `notifyMatchFailure`, `getScalableDims`, `get` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 938-954
```cpp
    // Create a loop over all rows and load one element at a time.
    auto loc = readOp.getLoc();
    auto lowerBound = arith::ConstantIndexOp::create(rewriter, loc, 0);
    auto createVscaleMultiple =
        vector::makeVscaleConstantBuilder(rewriter, loc);
    auto upperBound = createVscaleMultiple(numRows);
    auto step = arith::ConstantIndexOp::create(rewriter, loc, 1);
    Value init = arith::ConstantOp::create(
        rewriter, loc, newResType, DenseElementsAttr::get(newResType, 0.0f));

    scf::ForOp loadLoop;
    {
      OpBuilder::InsertionGuard g(rewriter);
      loadLoop = scf::ForOp::create(rewriter, loc, lowerBound, upperBound, step,
                                    ValueRange{init});
      rewriter.setInsertionPointToStart(loadLoop.getBody());

```
- **EN**: Implements logic around `getLoc`, `create`, `makeVscaleConstantBuilder`, `createVscaleMultiple`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `create`, `makeVscaleConstantBuilder`, `createVscaleMultiple`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 955-969
```cpp
      auto tileSliceIndex = loadLoop.getInductionVar();

      auto idx0 = arith::AddIOp::create(rewriter, loc, tileSliceIndex,
                                        readOp.getIndices()[0]);
      auto idx1 = readOp.getIndices()[1];

      Value scalar = memref::LoadOp::create(rewriter, loc, readOp.getBase(),
                                            SmallVector<Value>({idx0, idx1}));

      Operation *updateInit = vector::InsertOp::create(
          rewriter, loc, scalar, loadLoop.getRegionIterArg(0), tileSliceIndex);

      scf::YieldOp::create(rewriter, loc, updateInit->getResult(0));
    }

```
- **EN**: Implements logic around `getInductionVar`, `create`, `getIndices`, `SmallVector`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getInductionVar`, `create`, `getIndices`, `SmallVector`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 970-983
```cpp
    // The read operation has been "legalized", but since the original result
    // type was a 2D vector, we need to cast before returning the result. This
    // ShapeCast should cancel-out with some other ShapeCast (i.e. it's a
    // no-op).
    auto sc = vector::ShapeCastOp::create(
        rewriter, loc, readOp.getResult().getType(), loadLoop.getResult(0));

    rewriter.replaceOp(readOp, sc);

    return success();
  }
};

struct VectorLegalizationPass
```
- **EN**: Introduces declarations for `VectorLegalizationPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `VectorLegalizationPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 984-1001
```cpp
    : public arm_sme::impl::VectorLegalizationBase<VectorLegalizationPass> {
  void runOnOperation() override {
    auto *context = &getContext();
    TypeConverter converter;
    RewritePatternSet patterns(context);
    converter.addConversion([](Type type) { return type; });
    converter.addConversion(
        [](VectorType vectorType,
           SmallVectorImpl<Type> &types) -> std::optional<LogicalResult> {
          if (!isMultipleOfSMETileVectorType(vectorType))
            return std::nullopt;
          auto smeTileCount = getNumberOfSMETilesForVectorType(vectorType);
          auto smeTileType =
              getSMETileTypeForElement(vectorType.getElementType());
          types = SmallVector<Type>(smeTileCount, smeTileType);
          return success();
        });

```
- **EN**: Implements logic around `runOnOperation`, `getContext`, `patterns`, `addConversion`, and 5 more symbols; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getContext`, `patterns`, `addConversion`, and 5 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 1002-1027
```cpp
    // Apply preprocessing patterns.
    RewritePatternSet rewritePatterns(context);
    rewritePatterns
        .add<FoldExtractFromVectorOfSMELikeCreateMasks,
             LowerColumnTransferReadToLoops, LiftIllegalVectorTransposeToMemory,
             LowerIllegalTransposeStoreViaZA>(context);
    if (failed(
            applyPatternsGreedily(getOperation(), std::move(rewritePatterns))))
      return signalPassFailure();

    // Note: These two patterns are added with a high benefit to ensure:
    //  - Masked outer products are handled before unmasked ones
    //  - Multi-tile writes are lowered as a store loop (if possible)
    patterns.add<LegalizeMaskedVectorOuterProductOpsByDecomposition,
                 LegalizeMultiTileTransferWriteAsStoreLoop>(converter, context,
                                                            /*benefit=*/1024);
    patterns.add<LegalizeArithConstantOpsByDecomposition,
                 LegalizeVectorOuterProductOpsByDecomposition,
                 LegalizeTransferReadOpsByDecomposition,
                 LegalizeTransferWriteOpsByDecomposition>(converter, context);
    populateFunctionOpInterfaceTypeConversionPattern<func::FuncOp>(patterns,
                                                                   converter);
    populateCallOpTypeConversionPattern(patterns, converter);
    populateReturnOpTypeConversionPattern(patterns, converter);
    scf::populateSCFStructuralTypeConversions(converter, patterns);

```
- **EN**: Implements logic around `rewritePatterns`, `LowerIllegalTransposeStoreViaZA>`, `failed`, `applyPatternsGreedily`, and 7 more symbols; this block implements verifier, folding, parsing, or printing hooks; works with symbol tables or function-like operations.
- **CN**: 围绕 `rewritePatterns`, `LowerIllegalTransposeStoreViaZA>`, `failed`, `applyPatternsGreedily`, and 7 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理符号表或类函数操作。

### Lines 1028-1041
```cpp
    ConversionTarget target(getContext());
    target.markUnknownOpDynamicallyLegal(
        [&](Operation *op) { return converter.isLegal(op); });
    target.addDynamicallyLegalOp<func::FuncOp>([&](func::FuncOp op) {
      return converter.isSignatureLegal(op.getFunctionType());
    });
    if (failed(applyPartialConversion(getOperation(), target,
                                      std::move(patterns))))
      return signalPassFailure();
  }
};

} // namespace

```
- **EN**: Implements logic around `target`, `markUnknownOpDynamicallyLegal`, `isLegal`, `FuncOp>`, and 4 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `target`, `markUnknownOpDynamicallyLegal`, `isLegal`, `FuncOp>`, and 4 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 1042-1044
```cpp
std::unique_ptr<Pass> mlir::arm_sme::createVectorLegalizationPass() {
  return std::make_unique<VectorLegalizationPass>();
}
```
- **EN**: Implements logic around `createVectorLegalizationPass`, `make_unique`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `createVectorLegalizationPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Tensor/buffer boundary / 张量/缓冲区边界**:
  - **EN**: Tracks how abstract tensor values are converted into explicit memory effects and memref-based IR.
  - **CN**: 跟踪抽象张量值如何转换成显式内存效应与基于 memref 的 IR。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ArmSME/IR/ArmSME.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h`, `mlir/Dialect/ArmSME/Utils/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Index/IR/IndexDialect.h`, `mlir/Dialect/Index/IR/IndexOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/SCF/IR/SCF.h` ... (+6 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (14), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (2)
