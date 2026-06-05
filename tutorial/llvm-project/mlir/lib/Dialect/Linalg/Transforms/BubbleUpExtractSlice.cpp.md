# BubbleUpExtractSlice.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/BubbleUpExtractSlice.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements patterns that transforms linalg.<op> + tensor.extract_slice into tensor.extract_slice + linalg.<op> to reduce the computation for the linalg op.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- BubbleUpExtractSlice.cpp - bubble up tensor.extract_slice ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```cpp
//
// This file implements patterns that transforms linalg.<op> +
// tensor.extract_slice into tensor.extract_slice + linalg.<op> to reduce
// the computation for the linalg op.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 14-19
```cpp

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`。

### Lines 20-24
```cpp
using namespace mlir;
using namespace mlir::linalg;

namespace {
/// Bubble up extract_slice above Linalg operation.
```
- **EN**: Introduces declarations for `mlir`, `mlir::linalg`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::linalg` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 25-29
```cpp
///
/// A sequence of operations
///
/// ```mlir
/// %0 = linalg.<op> ... arg0, arg1, ...
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 30-34
```cpp
/// %1 = tensor.extract_slice %0 ...
/// ```
///
/// can be replaced with
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 35-39
```cpp
/// ```mlir
/// %0 = tensor.extract_slice %arg0
/// %1 = tensor.extract_slice %arg1
/// %2 = linalg.<op> ... %0, %1, ...
/// ```
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 40-46
```cpp
///
/// This results in the reduce computation of the linalg operation.
///
struct BubbleUpExtractSliceOpPattern
    : OpRewritePattern<tensor::ExtractSliceOp> {
  using OpRewritePattern<tensor::ExtractSliceOp>::OpRewritePattern;

```
- **EN**: Introduces declarations for `BubbleUpExtractSliceOpPattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `BubbleUpExtractSliceOpPattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 47-55
```cpp
  LogicalResult matchAndRewrite(tensor::ExtractSliceOp sliceOp,
                                PatternRewriter &rewriter) const final {
    Value source = sliceOp.getSource();
    auto linalgOp = source.getDefiningOp<LinalgOp>();
    if (!linalgOp) {
      return rewriter.notifyMatchFailure(sliceOp,
                                         "expected source to be linalg op");
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `getSource`, `getDefiningOp`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `getSource`, `getDefiningOp`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 56-62
```cpp
    // TODO: we might relax this if we want heuristics to detect that all uses
    // are small portion of the output.
    if (!linalgOp->hasOneUse()) {
      return rewriter.notifyMatchFailure(sliceOp,
                                         "expected single use of linalg op");
    }

```
- **EN**: Implements logic around `hasOneUse`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasOneUse`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 63-67
```cpp
    if (linalgOp.getNumDpsInits() != 1) {
      return rewriter.notifyMatchFailure(sliceOp,
                                         "expected single output of linalg op");
    }

```
- **EN**: Implements logic around `getNumDpsInits`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumDpsInits`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 68-72
```cpp
    if (!linalgOp.hasPureTensorSemantics()) {
      return rewriter.notifyMatchFailure(sliceOp,
                                         "expected tensor of linalg op");
    }

```
- **EN**: Implements logic around `hasPureTensorSemantics`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureTensorSemantics`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 73-79
```cpp
    if (!sliceOp.hasUnitStride())
      return rewriter.notifyMatchFailure(sliceOp, "expected unit stride");

    if (sliceOp.getType().getRank() != sliceOp.getSourceType().getRank()) {
      return rewriter.notifyMatchFailure(sliceOp, "expected no rank reduction");
    }

```
- **EN**: Implements logic around `hasUnitStride`, `notifyMatchFailure`, `getType`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasUnitStride`, `notifyMatchFailure`, `getType` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 80-86
```cpp
    OpOperand *outOperand = linalgOp.getDpsInitOperand(0);
    AffineMap indexingMap = linalgOp.getMatchingIndexingMap(outOperand);
    if (!indexingMap.isProjectedPermutation()) {
      return rewriter.notifyMatchFailure(
          sliceOp, "expected a projected permutation for output");
    }

```
- **EN**: Implements logic around `getDpsInitOperand`, `getMatchingIndexingMap`, `isProjectedPermutation`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getDpsInitOperand`, `getMatchingIndexingMap`, `isProjectedPermutation`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 87-96
```cpp
    auto linalgLoc = linalgOp.getLoc();
    SmallVector<OpFoldResult> allShapeSizes =
        linalgOp.createFlatListOfOperandDims(rewriter, linalgLoc);
    AffineMap shapeSizesToLoopsMap = linalgOp.getShapesToLoopsMap();
    if (!shapeSizesToLoopsMap) {
      return rewriter.notifyMatchFailure(
          linalgOp, "failed to get loops map from shape sizes");
    }
    SmallVector<OpFoldResult> sizeBounds =
        affine::makeComposedFoldedMultiResultAffineApply(
```
- **EN**: Implements logic around `getLoc`, `createFlatListOfOperandDims`, `getShapesToLoopsMap`, `notifyMatchFailure`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `createFlatListOfOperandDims`, `getShapesToLoopsMap`, `notifyMatchFailure`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 97-106
```cpp
            rewriter, linalgLoc, shapeSizesToLoopsMap, allShapeSizes);

    // The offsets and sizes from the slice operation only give you the tile
    // size of the output. Use that compute the tile sizes and offsets of the
    // loops. For loops not used to access the output, set the tile sizes to
    // loop bounds and set the offset to 0.
    SmallVector<OpFoldResult> tileOffsets(sizeBounds.size(),
                                          rewriter.getIndexAttr(0));
    SmallVector<OpFoldResult> tileSizes = sizeBounds;
    for (auto const &result : enumerate(indexingMap.getResults())) {
```
- **EN**: Implements logic around `tileOffsets`, `getIndexAttr`, `enumerate`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `tileOffsets`, `getIndexAttr`, `enumerate` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 107-111
```cpp
      unsigned position = cast<AffineDimExpr>(result.value()).getPosition();
      tileOffsets[position] = sliceOp.getMixedOffsets()[result.index()];
      tileSizes[position] = sliceOp.getMixedSizes()[result.index()];
    }

```
- **EN**: Implements logic around `value`, `getMixedOffsets`, `getMixedSizes`.
- **CN**: 围绕 `value`, `getMixedOffsets`, `getMixedSizes` 实现具体逻辑。

### Lines 112-117
```cpp
    SmallVector<Value> valuesToTile = linalgOp->getOperands();
    SmallVector<Value> tiledOperands =
        makeTiledShapes(rewriter, linalgLoc, linalgOp, valuesToTile,
                        tileOffsets, tileSizes, sizeBounds,
                        /*omitPartialTileCheck=*/true);

```
- **EN**: Implements logic around `getOperands`, `makeTiledShapes`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getOperands`, `makeTiledShapes` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 118-122
```cpp
    SmallVector<Type, 4> resultTensorTypes;
    for (OpOperand &opOperand : linalgOp.getDpsInitsMutable())
      resultTensorTypes.push_back(
          tiledOperands[opOperand.getOperandNumber()].getType());

```
- **EN**: Implements logic around `getDpsInitsMutable`, `push_back`, `getOperandNumber`.
- **CN**: 围绕 `getDpsInitsMutable`, `push_back`, `getOperandNumber` 实现具体逻辑。

### Lines 123-130
```cpp
    Operation *newOp =
        clone(rewriter, linalgOp, resultTensorTypes, tiledOperands);
    rewriter.replaceOp(sliceOp, newOp->getResults());
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `clone`, `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `clone`, `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 131-135
```cpp
void mlir::linalg::populateBubbleUpExtractSliceOpPatterns(
    RewritePatternSet &patterns) {
  auto *context = patterns.getContext();
  patterns.add<BubbleUpExtractSliceOpPattern>(context);
}
```
- **EN**: Implements logic around `populateBubbleUpExtractSliceOpPatterns`, `getContext`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateBubbleUpExtractSliceOpPatterns`, `getContext`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4)
