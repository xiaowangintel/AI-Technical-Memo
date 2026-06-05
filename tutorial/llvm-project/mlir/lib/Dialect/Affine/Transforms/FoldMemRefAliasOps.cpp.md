# FoldMemRefAliasOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/FoldMemRefAliasOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass contains affine-specif versions of the folding patterns for memref.expand_shape, memref.collapse_shape, and memref.subview, since those all need affine-specific handling that won't fit a general interface.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- FoldMemRefAliasOps.cpp - Fold memref alias ops for affine ops ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
//
// This pass contains affine-specif versions of the folding patterns for
// memref.expand_shape, memref.collapse_shape, and memref.subview, since
// those all need affine-specific handling that won't fit a general interface.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-23
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Transforms/Passes.h"
#include "mlir/Dialect/Affine/ViewLikeInterfaceUtils.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/MemRef/Utils/MemRefUtils.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/ViewLikeInterfaceUtils.h`, `mlir/Dialect/Arith/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/ViewLikeInterfaceUtils.h`, `mlir/Dialect/Arith/Utils/Utils.h`。

### Lines 24-30
```cpp
namespace mlir {
namespace affine {
#define GEN_PASS_DEF_AFFINEFOLDMEMREFALIASOPS
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 31-37
```cpp
using namespace mlir;
using namespace mlir::affine;

//===----------------------------------------------------------------------===//
// Utility functions
//===----------------------------------------------------------------------===//

```
- **EN**: Pulls in the headers needed by this translation unit, including local and MLIR/LLVM headers.
- **CN**: 引入该编译单元所需的头文件，其中包括 local and MLIR/LLVM headers。

### Lines 38-51
```cpp
/// Given an AffineMap and a list of indices, apply the map to get the
/// underlying indices (expanding the affine map).
static void expandToUnderlyingIndices(AffineMap affineMap, ValueRange indices,
                                      Location loc, PatternRewriter &rewriter,
                                      SmallVectorImpl<Value> &result) {
  SmallVector<OpFoldResult> indicesOfr(
      llvm::map_to_vector(indices, [](Value v) -> OpFoldResult { return v; }));
  for (unsigned i : llvm::seq(0u, affineMap.getNumResults())) {
    OpFoldResult ofr = affine::makeComposedFoldedAffineApply(
        rewriter, loc, affineMap.getSubMap({i}), indicesOfr);
    result.push_back(getValueOrCreateConstantIndexOp(rewriter, loc, ofr));
  }
}

```
- **EN**: Implements logic around `expandToUnderlyingIndices`, `indicesOfr`, `map_to_vector`, `seq`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `expandToUnderlyingIndices`, `indicesOfr`, `map_to_vector`, `seq`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 52-58
```cpp
//===----------------------------------------------------------------------===//
// Patterns
//===----------------------------------------------------------------------===//

namespace {

struct AffineLoadOpOfSubViewOpFolder final : OpRewritePattern<AffineLoadOp> {
```
- **EN**: Introduces declarations for `AffineLoadOpOfSubViewOpFolder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineLoadOpOfSubViewOpFolder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 59-67
```cpp
  using Base::Base;

  LogicalResult matchAndRewrite(AffineLoadOp loadOp,
                                PatternRewriter &rewriter) const override {
    auto subViewOp = loadOp.getMemref().getDefiningOp<memref::SubViewOp>();

    if (!subViewOp)
      return rewriter.notifyMatchFailure(loadOp, "not a subview producer");

```
- **EN**: Implements logic around `matchAndRewrite`, `getMemref`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `getMemref`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 68-77
```cpp
    SmallVector<Value> indices;
    expandToUnderlyingIndices(loadOp.getAffineMap(), loadOp.getIndices(),
                              loadOp.getLoc(), rewriter, indices);

    SmallVector<Value> sourceIndices;
    affine::resolveIndicesIntoOpWithOffsetsAndStrides(
        rewriter, loadOp.getLoc(), subViewOp.getMixedOffsets(),
        subViewOp.getMixedStrides(), subViewOp.getDroppedDims(), indices,
        sourceIndices);

```
- **EN**: Implements logic around `expandToUnderlyingIndices`, `getLoc`, `resolveIndicesIntoOpWithOffsetsAndStrides`, `getMixedStrides`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `expandToUnderlyingIndices`, `getLoc`, `resolveIndicesIntoOpWithOffsetsAndStrides`, `getMixedStrides` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 78-84
```cpp
    rewriter.replaceOpWithNewOp<AffineLoadOp>(loadOp, subViewOp.getSource(),
                                              sourceIndices);
    return success();
  }
};

struct AffineLoadOpOfExpandShapeOpFolder final
```
- **EN**: Introduces declarations for `AffineLoadOpOfExpandShapeOpFolder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineLoadOpOfExpandShapeOpFolder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 85-92
```cpp
    : OpRewritePattern<AffineLoadOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(AffineLoadOp loadOp,
                                PatternRewriter &rewriter) const override {
    auto expandShapeOp =
        loadOp.getMemref().getDefiningOp<memref::ExpandShapeOp>();

```
- **EN**: Implements logic around `matchAndRewrite`, `getMemref`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `getMemref` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 93-99
```cpp
    if (!expandShapeOp)
      return failure();

    SmallVector<Value> indices;
    expandToUnderlyingIndices(loadOp.getAffineMap(), loadOp.getIndices(),
                              loadOp.getLoc(), rewriter, indices);

```
- **EN**: Implements logic around `failure`, `expandToUnderlyingIndices`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `failure`, `expandToUnderlyingIndices`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 100-106
```cpp
    SmallVector<Value> sourceIndices;
    // affine.load guarantees that indexes start inbounds, which impacts if our
    // linearization is `disjoint`.
    memref::resolveSourceIndicesExpandShape(
        loadOp.getLoc(), rewriter, expandShapeOp, indices, sourceIndices,
        /*startsInbounds=*/true);

```
- **EN**: Implements logic around `resolveSourceIndicesExpandShape`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `resolveSourceIndicesExpandShape`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 107-113
```cpp
    rewriter.replaceOpWithNewOp<AffineLoadOp>(
        loadOp, expandShapeOp.getViewSource(), sourceIndices);
    return success();
  }
};

struct AffineLoadOpOfCollapseShapeOpFolder final
```
- **EN**: Introduces declarations for `AffineLoadOpOfCollapseShapeOpFolder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineLoadOpOfCollapseShapeOpFolder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 114-121
```cpp
    : OpRewritePattern<AffineLoadOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(AffineLoadOp loadOp,
                                PatternRewriter &rewriter) const override {
    auto collapseShapeOp =
        loadOp.getMemref().getDefiningOp<memref::CollapseShapeOp>();

```
- **EN**: Implements logic around `matchAndRewrite`, `getMemref`; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `matchAndRewrite`, `getMemref` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 122-128
```cpp
    if (!collapseShapeOp)
      return failure();

    SmallVector<Value> indices;
    expandToUnderlyingIndices(loadOp.getAffineMap(), loadOp.getIndices(),
                              loadOp.getLoc(), rewriter, indices);

```
- **EN**: Implements logic around `failure`, `expandToUnderlyingIndices`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `failure`, `expandToUnderlyingIndices`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 129-139
```cpp
    SmallVector<Value> sourceIndices;
    memref::resolveSourceIndicesCollapseShape(
        loadOp.getLoc(), rewriter, collapseShapeOp, indices, sourceIndices,
        /*startsInbounds=*/true);

    rewriter.replaceOpWithNewOp<AffineLoadOp>(
        loadOp, collapseShapeOp.getViewSource(), sourceIndices);
    return success();
  }
};

```
- **EN**: Implements logic around `resolveSourceIndicesCollapseShape`, `getLoc`, `replaceOpWithNewOp`, `getViewSource`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `resolveSourceIndicesCollapseShape`, `getLoc`, `replaceOpWithNewOp`, `getViewSource`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 140-146
```cpp
struct AffineStoreOpOfSubViewOpFolder final : OpRewritePattern<AffineStoreOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(AffineStoreOp storeOp,
                                PatternRewriter &rewriter) const override {
    auto subViewOp = storeOp.getMemref().getDefiningOp<memref::SubViewOp>();

```
- **EN**: Introduces declarations for `AffineStoreOpOfSubViewOpFolder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineStoreOpOfSubViewOpFolder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 147-154
```cpp
    if (!subViewOp)
      return rewriter.notifyMatchFailure(storeOp, "not a subview producer");

    // For affine ops, we need to apply the map to get the "actual" indices.
    SmallVector<Value> indices;
    expandToUnderlyingIndices(storeOp.getAffineMap(), storeOp.getIndices(),
                              storeOp.getLoc(), rewriter, indices);

```
- **EN**: Implements logic around `notifyMatchFailure`, `expandToUnderlyingIndices`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `notifyMatchFailure`, `expandToUnderlyingIndices`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 155-166
```cpp
    SmallVector<Value> sourceIndices;
    affine::resolveIndicesIntoOpWithOffsetsAndStrides(
        rewriter, storeOp.getLoc(), subViewOp.getMixedOffsets(),
        subViewOp.getMixedStrides(), subViewOp.getDroppedDims(), indices,
        sourceIndices);

    rewriter.replaceOpWithNewOp<AffineStoreOp>(
        storeOp, storeOp.getValue(), subViewOp.getSource(), sourceIndices);
    return success();
  }
};

```
- **EN**: Implements logic around `resolveIndicesIntoOpWithOffsetsAndStrides`, `getLoc`, `getMixedStrides`, `replaceOpWithNewOp`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `resolveIndicesIntoOpWithOffsetsAndStrides`, `getLoc`, `getMixedStrides`, `replaceOpWithNewOp`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 167-175
```cpp
struct AffineStoreOpOfExpandShapeOpFolder final
    : OpRewritePattern<AffineStoreOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(AffineStoreOp storeOp,
                                PatternRewriter &rewriter) const override {
    auto expandShapeOp =
        storeOp.getMemref().getDefiningOp<memref::ExpandShapeOp>();

```
- **EN**: Introduces declarations for `AffineStoreOpOfExpandShapeOpFolder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineStoreOpOfExpandShapeOpFolder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 176-182
```cpp
    if (!expandShapeOp)
      return failure();

    SmallVector<Value> indices;
    expandToUnderlyingIndices(storeOp.getAffineMap(), storeOp.getIndices(),
                              storeOp.getLoc(), rewriter, indices);

```
- **EN**: Implements logic around `failure`, `expandToUnderlyingIndices`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `failure`, `expandToUnderlyingIndices`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 183-189
```cpp
    SmallVector<Value> sourceIndices;
    // affine.store guarantees that indexes start inbounds, which impacts if our
    // linearization is `disjoint`.
    memref::resolveSourceIndicesExpandShape(
        storeOp.getLoc(), rewriter, expandShapeOp, indices, sourceIndices,
        /*startsInbounds=*/true);

```
- **EN**: Implements logic around `resolveSourceIndicesExpandShape`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `resolveSourceIndicesExpandShape`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 190-196
```cpp
    rewriter.replaceOpWithNewOp<AffineStoreOp>(
        storeOp, storeOp.getValueToStore(), expandShapeOp.getViewSource(),
        sourceIndices);
    return success();
  }
};

```
- **EN**: Implements logic around `replaceOpWithNewOp`, `getValueToStore`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOpWithNewOp`, `getValueToStore`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 197-205
```cpp
struct AffineStoreOpOfCollapseShapeOpFolder final
    : OpRewritePattern<AffineStoreOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(AffineStoreOp storeOp,
                                PatternRewriter &rewriter) const override {
    auto collapseShapeOp =
        storeOp.getMemref().getDefiningOp<memref::CollapseShapeOp>();

```
- **EN**: Introduces declarations for `AffineStoreOpOfCollapseShapeOpFolder`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineStoreOpOfCollapseShapeOpFolder` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 206-213
```cpp
    if (!collapseShapeOp)
      return failure();

    // For affine ops, we need to apply the map to get the "actual" indices.
    SmallVector<Value> indices;
    expandToUnderlyingIndices(storeOp.getAffineMap(), storeOp.getIndices(),
                              storeOp.getLoc(), rewriter, indices);

```
- **EN**: Implements logic around `failure`, `expandToUnderlyingIndices`, `getLoc`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `failure`, `expandToUnderlyingIndices`, `getLoc` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 214-225
```cpp
    SmallVector<Value> sourceIndices;
    memref::resolveSourceIndicesCollapseShape(
        storeOp.getLoc(), rewriter, collapseShapeOp, indices, sourceIndices,
        /*startsInbounds=*/true);

    rewriter.replaceOpWithNewOp<AffineStoreOp>(
        storeOp, storeOp.getValueToStore(), collapseShapeOp.getViewSource(),
        sourceIndices);
    return success();
  }
};

```
- **EN**: Implements logic around `resolveSourceIndicesCollapseShape`, `getLoc`, `replaceOpWithNewOp`, `getValueToStore`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `resolveSourceIndicesCollapseShape`, `getLoc`, `replaceOpWithNewOp`, `getValueToStore`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并在张量风格值与显式缓冲区之间移动数据。

### Lines 226-236
```cpp
} // namespace

void affine::populateAffineFoldMemRefAliasOpPatterns(
    RewritePatternSet &patterns) {
  patterns
      .add<AffineLoadOpOfSubViewOpFolder, AffineLoadOpOfExpandShapeOpFolder,
           AffineLoadOpOfCollapseShapeOpFolder, AffineStoreOpOfSubViewOpFolder,
           AffineStoreOpOfExpandShapeOpFolder,
           AffineStoreOpOfCollapseShapeOpFolder>(patterns.getContext());
}

```
- **EN**: Implements logic around `populateAffineFoldMemRefAliasOpPatterns`, `AffineStoreOpOfCollapseShapeOpFolder>`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `populateAffineFoldMemRefAliasOpPatterns`, `AffineStoreOpOfCollapseShapeOpFolder>` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 237-243
```cpp
//===----------------------------------------------------------------------===//
// Pass registration
//===----------------------------------------------------------------------===//

namespace {

struct AffineFoldMemRefAliasOpsPass final
```
- **EN**: Introduces declarations for `AffineFoldMemRefAliasOpsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineFoldMemRefAliasOpsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 244-250
```cpp
    : public affine::impl::AffineFoldMemRefAliasOpsBase<
          AffineFoldMemRefAliasOpsPass> {
  void runOnOperation() override;
};

} // namespace

```
- **EN**: Implements logic around `runOnOperation`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `runOnOperation` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理。

### Lines 251-255
```cpp
void AffineFoldMemRefAliasOpsPass::runOnOperation() {
  RewritePatternSet patterns(&getContext());
  affine::populateAffineFoldMemRefAliasOpPatterns(patterns);
  (void)applyPatternsGreedily(getOperation(), std::move(patterns));
}
```
- **EN**: Implements logic around `runOnOperation`, `patterns`, `populateAffineFoldMemRefAliasOpPatterns`, `applyPatternsGreedily`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `runOnOperation`, `patterns`, `populateAffineFoldMemRefAliasOpPatterns`, `applyPatternsGreedily` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/ViewLikeInterfaceUtils.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Utils/MemRefUtils.h`, `mlir/IR/AffineMap.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (7), MLIR IR core abstractions / MLIR IR 核心抽象 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
