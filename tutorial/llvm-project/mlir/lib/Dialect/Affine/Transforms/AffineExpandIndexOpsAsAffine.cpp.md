# AffineExpandIndexOpsAsAffine.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/AffineExpandIndexOpsAsAffine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass to expand affine index ops into one or more more fundamental operations.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineExpandIndexOpsAsAffine.cpp - Expand index ops to apply pass --===//
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
// This file implements a pass to expand affine index ops into one or more more
// fundamental operations.
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`。

### Lines 15-23
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Transforms/Transforms.h"
#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/UB/IR/UBOps.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/Utils/Utils.h`。

### Lines 24-30
```cpp
namespace mlir {
namespace affine {
#define GEN_PASS_DEF_AFFINEEXPANDINDEXOPSASAFFINE
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

namespace {
/// Lowers `affine.delinearize_index` into a sequence of division and remainder
/// operations via affine.apply. For vector types, unrolls to per-element
/// scalar affine.apply operations.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 38-46
```cpp
struct LowerDelinearizeIndexOps
    : public OpRewritePattern<AffineDelinearizeIndexOp> {
  using OpRewritePattern<AffineDelinearizeIndexOp>::OpRewritePattern;
  LogicalResult matchAndRewrite(AffineDelinearizeIndexOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    Value linearIndex = op.getLinearIndex();
    auto vecTy = dyn_cast<VectorType>(linearIndex.getType());

```
- **EN**: Introduces declarations for `LowerDelinearizeIndexOps`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LowerDelinearizeIndexOps` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 47-57
```cpp
    // Scalar case: use the existing affine lowering path.
    if (!vecTy) {
      FailureOr<SmallVector<Value>> multiIndex =
          delinearizeIndex(rewriter, loc, linearIndex, op.getEffectiveBasis(),
                           /*hasOuterBound=*/false);
      if (failed(multiIndex))
        return failure();
      rewriter.replaceOp(op, *multiIndex);
      return success();
    }

```
- **EN**: Implements logic around `delinearizeIndex`, `failed`, `failure`, `replaceOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `delinearizeIndex`, `failed`, `failure`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 58-66
```cpp
    // Vector case: unroll to per-element scalar affine.apply operations
    // using StaticTileOffsetRange for multi-dimensional vector support.
    if (vecTy.isScalable())
      return rewriter.notifyMatchFailure(op, "scalable vectors not supported");

    unsigned numResults = op.getNumResults();
    ArrayRef<int64_t> shape = vecTy.getShape();
    SmallVector<int64_t> tileShape(shape.size(), 1);

```
- **EN**: Implements logic around `isScalable`, `notifyMatchFailure`, `getNumResults`, `getShape`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `isScalable`, `notifyMatchFailure`, `getNumResults`, `getShape`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 67-74
```cpp
    SmallVector<Value> resultVecs(numResults);
    Value poison = ub::PoisonOp::create(rewriter, loc, vecTy);
    for (unsigned r = 0; r < numResults; ++r)
      resultVecs[r] = poison;

    for (SmallVector<int64_t> pos : StaticTileOffsetRange(shape, tileShape)) {
      Value scalar = vector::ExtractOp::create(rewriter, loc, linearIndex, pos);

```
- **EN**: Implements logic around `resultVecs`, `create`, `StaticTileOffsetRange`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `resultVecs`, `create`, `StaticTileOffsetRange` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 75-85
```cpp
      FailureOr<SmallVector<Value>> scalarResults =
          delinearizeIndex(rewriter, loc, scalar, op.getEffectiveBasis(),
                           /*hasOuterBound=*/false);
      if (failed(scalarResults))
        return failure();

      for (unsigned r = 0; r < numResults; ++r)
        resultVecs[r] = vector::InsertOp::create(
            rewriter, loc, (*scalarResults)[r], resultVecs[r], pos);
    }

```
- **EN**: Implements logic around `delinearizeIndex`, `failed`, `failure`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `delinearizeIndex`, `failed`, `failure`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 86-92
```cpp
    rewriter.replaceOp(op, resultVecs);
    return success();
  }
};

/// Lowers `affine.linearize_index` into a sequence of multiplications and
/// additions via affine.apply. For vector types, unrolls to per-element
```
- **EN**: Implements logic around `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 93-100
```cpp
/// scalar affine.apply operations.
struct LowerLinearizeIndexOps final : OpRewritePattern<AffineLinearizeIndexOp> {
  using OpRewritePattern::OpRewritePattern;
  LogicalResult matchAndRewrite(AffineLinearizeIndexOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    auto vecTy = dyn_cast<VectorType>(op.getLinearIndex().getType());

```
- **EN**: Introduces declarations for `LowerLinearizeIndexOps`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LowerLinearizeIndexOps` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 101-108
```cpp
    // Scalar case: use the existing affine lowering path.
    if (!vecTy) {
      // Should be folded away, included here for safety.
      if (op.getMultiIndex().empty()) {
        rewriter.replaceOpWithNewOp<arith::ConstantIndexOp>(op, 0);
        return success();
      }

```
- **EN**: Implements logic around `getMultiIndex`, `ConstantIndexOp>`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getMultiIndex`, `ConstantIndexOp>`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并执行仿射推理或形状/边界处理。

### Lines 109-118
```cpp
      SmallVector<OpFoldResult> multiIndex =
          getAsOpFoldResult(op.getMultiIndex());
      OpFoldResult linearIndex =
          linearizeIndex(rewriter, loc, multiIndex, op.getMixedBasis());
      Value linearIndexValue =
          getValueOrCreateConstantIntOp(rewriter, loc, linearIndex);
      rewriter.replaceOp(op, linearIndexValue);
      return success();
    }

```
- **EN**: Implements logic around `getAsOpFoldResult`, `linearizeIndex`, `getValueOrCreateConstantIntOp`, `replaceOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getAsOpFoldResult`, `linearizeIndex`, `getValueOrCreateConstantIntOp`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 119-127
```cpp
    // Vector case: unroll to per-element scalar affine.apply operations
    // using StaticTileOffsetRange for multi-dimensional vector support.
    if (vecTy.isScalable())
      return rewriter.notifyMatchFailure(op, "scalable vectors not supported");

    ArrayRef<int64_t> shape = vecTy.getShape();
    SmallVector<int64_t> tileShape(shape.size(), 1);
    ValueRange multiIndex = op.getMultiIndex();

```
- **EN**: Implements logic around `isScalable`, `notifyMatchFailure`, `getShape`, `tileShape`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `isScalable`, `notifyMatchFailure`, `getShape`, `tileShape`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 128-135
```cpp
    Value result = ub::PoisonOp::create(rewriter, loc, vecTy);

    for (SmallVector<int64_t> pos : StaticTileOffsetRange(shape, tileShape)) {
      SmallVector<OpFoldResult> scalarIndices;
      for (Value vec : multiIndex)
        scalarIndices.push_back(
            vector::ExtractOp::create(rewriter, loc, vec, pos).getResult());

```
- **EN**: Implements logic around `create`, `StaticTileOffsetRange`, `push_back`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `StaticTileOffsetRange`, `push_back` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 136-144
```cpp
      OpFoldResult linearIndex =
          linearizeIndex(rewriter, loc, scalarIndices, op.getMixedBasis());
      Value scalarResult =
          getValueOrCreateConstantIntOp(rewriter, loc, linearIndex);

      result =
          vector::InsertOp::create(rewriter, loc, scalarResult, result, pos);
    }

```
- **EN**: Implements logic around `linearizeIndex`, `getValueOrCreateConstantIntOp`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `linearizeIndex`, `getValueOrCreateConstantIntOp`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 145-153
```cpp
    rewriter.replaceOp(op, result);
    return success();
  }
};

class ExpandAffineIndexOpsAsAffinePass
    : public affine::impl::AffineExpandIndexOpsAsAffineBase<
          ExpandAffineIndexOpsAsAffinePass> {
public:
```
- **EN**: Introduces declarations for `ExpandAffineIndexOpsAsAffinePass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ExpandAffineIndexOpsAsAffinePass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 154-164
```cpp
  ExpandAffineIndexOpsAsAffinePass() = default;

  void runOnOperation() override {
    MLIRContext *context = &getContext();
    RewritePatternSet patterns(context);
    populateAffineExpandIndexOpsAsAffinePatterns(patterns);
    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns))))
      return signalPassFailure();
  }
};

```
- **EN**: Implements logic around `ExpandAffineIndexOpsAsAffinePass`, `runOnOperation`, `getContext`, `patterns`, and 3 more symbols; this block participates in pass execution or pass construction.
- **CN**: 围绕 `ExpandAffineIndexOpsAsAffinePass`, `runOnOperation`, `getContext`, `patterns`, and 3 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 165-172
```cpp
} // namespace

void mlir::affine::populateAffineExpandIndexOpsAsAffinePatterns(
    RewritePatternSet &patterns) {
  patterns.insert<LowerDelinearizeIndexOps, LowerLinearizeIndexOps>(
      patterns.getContext());
}

```
- **EN**: Implements logic around `populateAffineExpandIndexOpsAsAffinePatterns`, `LowerLinearizeIndexOps>`, `getContext`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `populateAffineExpandIndexOpsAsAffinePatterns`, `LowerLinearizeIndexOps>`, `getContext` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 173-175
```cpp
std::unique_ptr<Pass> mlir::affine::createAffineExpandIndexOpsAsAffinePass() {
  return std::make_unique<ExpandAffineIndexOpsAsAffinePass>();
}
```
- **EN**: Implements logic around `createAffineExpandIndexOpsAsAffinePass`, `make_unique`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `createAffineExpandIndexOpsAsAffinePass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (9), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
