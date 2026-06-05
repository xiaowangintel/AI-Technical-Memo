# DecomposeMemRefs.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/DecomposeMemRefs.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements decompose memrefs pass.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- DecomposeMemRefs.cpp - Decompose memrefs pass implementation -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-21
```cpp
//
// This file implements decompose memrefs pass.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Transforms/Passes.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/MemRef/IR/MemRef.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/MemRef/IR/MemRef.h`。

### Lines 22-29
```cpp

namespace mlir {
#define GEN_PASS_DEF_GPUDECOMPOSEMEMREFSPASS
#include "mlir/Dialect/GPU/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;

```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 30-40
```cpp
static MemRefType inferCastResultType(Value source, OpFoldResult offset) {
  auto sourceType = cast<BaseMemRefType>(source.getType());
  SmallVector<int64_t> staticOffsets;
  SmallVector<Value> dynamicOffsets;
  dispatchIndexOpFoldResults(offset, dynamicOffsets, staticOffsets);
  auto stridedLayout =
      StridedLayoutAttr::get(source.getContext(), staticOffsets.front(), {});
  return MemRefType::get({}, sourceType.getElementType(), stridedLayout,
                         sourceType.getMemorySpace());
}

```
- **EN**: Implements logic around `inferCastResultType`, `cast`, `dispatchIndexOpFoldResults`, `get`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `inferCastResultType`, `cast`, `dispatchIndexOpFoldResults`, `get`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 41-48
```cpp
static void setInsertionPointToStart(OpBuilder &builder, Value val) {
  if (auto *parentOp = val.getDefiningOp()) {
    builder.setInsertionPointAfter(parentOp);
  } else {
    builder.setInsertionPointToStart(val.getParentBlock());
  }
}

```
- **EN**: Implements logic around `setInsertionPointToStart`, `getDefiningOp`, `setInsertionPointAfter`.
- **CN**: 围绕 `setInsertionPointToStart`, `getDefiningOp`, `setInsertionPointAfter` 实现具体逻辑。

### Lines 49-59
```cpp
static bool isInsideLaunch(Operation *op) {
  return op->getParentOfType<gpu::LaunchOp>();
}

static std::tuple<Value, OpFoldResult, SmallVector<OpFoldResult>>
getFlatOffsetAndStrides(OpBuilder &rewriter, Location loc, Value source,
                        ArrayRef<OpFoldResult> subOffsets,
                        ArrayRef<OpFoldResult> subStrides = {}) {
  auto sourceType = cast<MemRefType>(source.getType());
  auto sourceRank = static_cast<unsigned>(sourceType.getRank());

```
- **EN**: Implements logic around `isInsideLaunch`, `LaunchOp>`, `getFlatOffsetAndStrides`, `cast`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `isInsideLaunch`, `LaunchOp>`, `getFlatOffsetAndStrides`, `cast`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 60-67
```cpp
  memref::ExtractStridedMetadataOp newExtractStridedMetadata;
  {
    OpBuilder::InsertionGuard g(rewriter);
    setInsertionPointToStart(rewriter, source);
    newExtractStridedMetadata =
        memref::ExtractStridedMetadataOp::create(rewriter, loc, source);
  }

```
- **EN**: Implements logic around `g`, `setInsertionPointToStart`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `g`, `setInsertionPointToStart`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 68-74
```cpp
  auto &&[sourceStrides, sourceOffset] = sourceType.getStridesAndOffset();

  auto getDim = [&](int64_t dim, Value dimVal) -> OpFoldResult {
    return ShapedType::isDynamic(dim) ? getAsOpFoldResult(dimVal)
                                      : rewriter.getIndexAttr(dim);
  };

```
- **EN**: Implements logic around `getStridesAndOffset`, `isDynamic`, `getIndexAttr`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getStridesAndOffset`, `isDynamic`, `getIndexAttr` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 75-81
```cpp
  OpFoldResult origOffset =
      getDim(sourceOffset, newExtractStridedMetadata.getOffset());
  ValueRange sourceStridesVals = newExtractStridedMetadata.getStrides();

  SmallVector<OpFoldResult> origStrides;
  origStrides.reserve(sourceRank);

```
- **EN**: Implements logic around `getDim`, `getStrides`, `reserve`.
- **CN**: 围绕 `getDim`, `getStrides`, `reserve` 实现具体逻辑。

### Lines 82-89
```cpp
  SmallVector<OpFoldResult> strides;
  strides.reserve(sourceRank);

  AffineExpr s0 = rewriter.getAffineSymbolExpr(0);
  AffineExpr s1 = rewriter.getAffineSymbolExpr(1);
  for (auto i : llvm::seq(0u, sourceRank)) {
    OpFoldResult origStride = getDim(sourceStrides[i], sourceStridesVals[i]);

```
- **EN**: Implements logic around `reserve`, `getAffineSymbolExpr`, `seq`, `getDim`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `reserve`, `getAffineSymbolExpr`, `seq`, `getDim` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 90-97
```cpp
    if (!subStrides.empty()) {
      strides.push_back(affine::makeComposedFoldedAffineApply(
          rewriter, loc, s0 * s1, {subStrides[i], origStride}));
    }

    origStrides.emplace_back(origStride);
  }

```
- **EN**: Implements logic around `empty`, `push_back`, `emplace_back`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `empty`, `push_back`, `emplace_back` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 98-104
```cpp
  auto &&[expr, values] =
      computeLinearIndex(origOffset, origStrides, subOffsets);
  OpFoldResult finalOffset =
      affine::makeComposedFoldedAffineApply(rewriter, loc, expr, values);
  return {newExtractStridedMetadata.getBaseBuffer(), finalOffset, strides};
}

```
- **EN**: Implements logic around `computeLinearIndex`, `makeComposedFoldedAffineApply`, `getBaseBuffer`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `computeLinearIndex`, `makeComposedFoldedAffineApply`, `getBaseBuffer` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 105-115
```cpp
static Value getFlatMemref(OpBuilder &rewriter, Location loc, Value source,
                           ValueRange offsets) {
  SmallVector<OpFoldResult> offsetsTemp = getAsOpFoldResult(offsets);
  auto &&[base, offset, ignore] =
      getFlatOffsetAndStrides(rewriter, loc, source, offsetsTemp);
  MemRefType retType = inferCastResultType(base, offset);
  return memref::ReinterpretCastOp::create(rewriter, loc, retType, base, offset,
                                           ArrayRef<OpFoldResult>(),
                                           ArrayRef<OpFoldResult>());
}

```
- **EN**: Implements logic around `getFlatMemref`, `getAsOpFoldResult`, `getFlatOffsetAndStrides`, `inferCastResultType`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getFlatMemref`, `getAsOpFoldResult`, `getFlatOffsetAndStrides`, `inferCastResultType`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 116-126
```cpp
static bool needFlatten(Value val) {
  auto type = cast<MemRefType>(val.getType());
  return type.getRank() != 0;
}

static bool checkLayout(Value val) {
  auto type = cast<MemRefType>(val.getType());
  return type.getLayout().isIdentity() ||
         isa<StridedLayoutAttr>(type.getLayout());
}

```
- **EN**: Implements logic around `needFlatten`, `cast`, `getRank`, `checkLayout`, and 2 more symbols.
- **CN**: 围绕 `needFlatten`, `cast`, `getRank`, `checkLayout`, and 2 more symbols 实现具体逻辑。

### Lines 127-135
```cpp
namespace {
struct FlattenLoad : public OpRewritePattern<memref::LoadOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(memref::LoadOp op,
                                PatternRewriter &rewriter) const override {
    if (!isInsideLaunch(op))
      return rewriter.notifyMatchFailure(op, "not inside gpu.launch");

```
- **EN**: Introduces declarations for `FlattenLoad`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FlattenLoad` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 136-142
```cpp
    Value memref = op.getMemref();
    if (!needFlatten(memref))
      return rewriter.notifyMatchFailure(op, "nothing to do");

    if (!checkLayout(memref))
      return rewriter.notifyMatchFailure(op, "unsupported layout");

```
- **EN**: Implements logic around `getMemref`, `needFlatten`, `notifyMatchFailure`, `checkLayout`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getMemref`, `needFlatten`, `notifyMatchFailure`, `checkLayout` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 143-149
```cpp
    Location loc = op.getLoc();
    Value flatMemref = getFlatMemref(rewriter, loc, memref, op.getIndices());
    rewriter.replaceOpWithNewOp<memref::LoadOp>(op, flatMemref);
    return success();
  }
};

```
- **EN**: Implements logic around `getLoc`, `getFlatMemref`, `LoadOp>`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getLoc`, `getFlatMemref`, `LoadOp>`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 150-157
```cpp
struct FlattenStore : public OpRewritePattern<memref::StoreOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(memref::StoreOp op,
                                PatternRewriter &rewriter) const override {
    if (!isInsideLaunch(op))
      return rewriter.notifyMatchFailure(op, "not inside gpu.launch");

```
- **EN**: Introduces declarations for `FlattenStore`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FlattenStore` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 158-164
```cpp
    Value memref = op.getMemref();
    if (!needFlatten(memref))
      return rewriter.notifyMatchFailure(op, "nothing to do");

    if (!checkLayout(memref))
      return rewriter.notifyMatchFailure(op, "unsupported layout");

```
- **EN**: Implements logic around `getMemref`, `needFlatten`, `notifyMatchFailure`, `checkLayout`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getMemref`, `needFlatten`, `notifyMatchFailure`, `checkLayout` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 165-172
```cpp
    Location loc = op.getLoc();
    Value flatMemref = getFlatMemref(rewriter, loc, memref, op.getIndices());
    Value value = op.getValue();
    rewriter.replaceOpWithNewOp<memref::StoreOp>(op, value, flatMemref);
    return success();
  }
};

```
- **EN**: Implements logic around `getLoc`, `getFlatMemref`, `getValue`, `StoreOp>`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getLoc`, `getFlatMemref`, `getValue`, `StoreOp>`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 173-180
```cpp
struct FlattenSubview : public OpRewritePattern<memref::SubViewOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(memref::SubViewOp op,
                                PatternRewriter &rewriter) const override {
    if (!isInsideLaunch(op))
      return rewriter.notifyMatchFailure(op, "not inside gpu.launch");

```
- **EN**: Introduces declarations for `FlattenSubview`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FlattenSubview` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 181-187
```cpp
    Value memref = op.getSource();
    if (!needFlatten(memref))
      return rewriter.notifyMatchFailure(op, "nothing to do");

    if (!checkLayout(memref))
      return rewriter.notifyMatchFailure(op, "unsupported layout");

```
- **EN**: Implements logic around `getSource`, `needFlatten`, `notifyMatchFailure`, `checkLayout`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getSource`, `needFlatten`, `notifyMatchFailure`, `checkLayout` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 188-194
```cpp
    Location loc = op.getLoc();
    SmallVector<OpFoldResult> subOffsets = op.getMixedOffsets();
    SmallVector<OpFoldResult> subSizes = op.getMixedSizes();
    SmallVector<OpFoldResult> subStrides = op.getMixedStrides();
    auto &&[base, finalOffset, strides] =
        getFlatOffsetAndStrides(rewriter, loc, memref, subOffsets, subStrides);

```
- **EN**: Implements logic around `getLoc`, `getMixedOffsets`, `getMixedSizes`, `getMixedStrides`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getLoc`, `getMixedOffsets`, `getMixedSizes`, `getMixedStrides`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 195-203
```cpp
    auto srcType = cast<MemRefType>(memref.getType());
    auto resultType = cast<MemRefType>(op.getType());
    unsigned subRank = static_cast<unsigned>(resultType.getRank());

    llvm::SmallBitVector droppedDims = op.getDroppedDims();

    SmallVector<OpFoldResult> finalSizes;
    finalSizes.reserve(subRank);

```
- **EN**: Implements logic around `cast`, `static_cast`, `getDroppedDims`, `reserve`; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `cast`, `static_cast`, `getDroppedDims`, `reserve` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 204-210
```cpp
    SmallVector<OpFoldResult> finalStrides;
    finalStrides.reserve(subRank);

    for (auto i : llvm::seq(0u, static_cast<unsigned>(srcType.getRank()))) {
      if (droppedDims.test(i))
        continue;

```
- **EN**: Implements logic around `reserve`, `seq`, `test`.
- **CN**: 围绕 `reserve`, `seq`, `test` 实现具体逻辑。

### Lines 211-220
```cpp
      finalSizes.push_back(subSizes[i]);
      finalStrides.push_back(strides[i]);
    }

    rewriter.replaceOpWithNewOp<memref::ReinterpretCastOp>(
        op, resultType, base, finalOffset, finalSizes, finalStrides);
    return success();
  }
};

```
- **EN**: Implements logic around `push_back`, `ReinterpretCastOp>`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `push_back`, `ReinterpretCastOp>`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 221-228
```cpp
struct GpuDecomposeMemrefsPass
    : public impl::GpuDecomposeMemrefsPassBase<GpuDecomposeMemrefsPass> {

  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());

    populateGpuDecomposeMemrefsPatterns(patterns);

```
- **EN**: Introduces declarations for `GpuDecomposeMemrefsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuDecomposeMemrefsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 229-235
```cpp
    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns))))
      return signalPassFailure();
  }
};

} // namespace

```
- **EN**: Implements logic around `failed`, `signalPassFailure`.
- **CN**: 围绕 `failed`, `signalPassFailure` 实现具体逻辑。

### Lines 236-239
```cpp
void mlir::populateGpuDecomposeMemrefsPatterns(RewritePatternSet &patterns) {
  patterns.insert<FlattenLoad, FlattenStore, FlattenSubview>(
      patterns.getContext());
}
```
- **EN**: Implements logic around `populateGpuDecomposeMemrefsPatterns`, `FlattenSubview>`, `getContext`.
- **CN**: 围绕 `populateGpuDecomposeMemrefsPatterns`, `FlattenSubview>`, `getContext` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/Builders.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/GPU/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), MLIR IR core abstractions / MLIR IR 核心抽象 (3), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
