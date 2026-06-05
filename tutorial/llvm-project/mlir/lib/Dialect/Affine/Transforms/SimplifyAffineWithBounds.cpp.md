# SimplifyAffineWithBounds.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/SimplifyAffineWithBounds.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements simplification patterns for affine.delinearize_index / affine.linearize_index pairs using value bounds analysis.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- SimplifyAffineWithBounds.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-15
```cpp
//
// This file implements simplification patterns for affine.delinearize_index /
// affine.linearize_index pairs using value bounds analysis.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`。

### Lines 16-23
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Transforms/Transforms.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Interfaces/ValueBoundsOpInterface.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Utils/StaticValueUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Utils/StaticValueUtils.h`。

### Lines 24-30
```cpp
#define DEBUG_TYPE "affine-simplify-with-bounds"

using namespace mlir;
using namespace mlir::affine;

/// Accumulate a single basis element into the running product expression.
/// Static values become affine constants, and dynamic values become symbols.
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or generated-code switches.
- **CN**: 定义受预处理器控制的结构、特性开关或生成代码开关。

### Lines 31-41
```cpp
static void buildProductExpr(OpFoldResult basis, AffineExpr &productExpr,
                             SmallVectorImpl<Value> &operands,
                             MLIRContext *ctx) {
  if (auto val = getConstantIntValue(basis)) {
    productExpr = productExpr * getAffineConstantExpr(*val, ctx);
  } else {
    operands.push_back(cast<Value>(basis));
    productExpr = productExpr * getAffineSymbolExpr(operands.size() - 1, ctx);
  }
}

```
- **EN**: Implements logic around `buildProductExpr`, `getConstantIntValue`, `getAffineConstantExpr`, `push_back`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `buildProductExpr`, `getConstantIntValue`, `getAffineConstantExpr`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 42-55
```cpp
/// Try to find k consecutive elements from `lhs` (starting from tail offset)
/// whose product equals the single next element from `rhs`.
/// The product is accumulated incrementally to avoid redundant computation.
/// Returns the number of matched elements k, or std::nullopt if no match.
static std::optional<size_t> tryMatchProduct(ArrayRef<OpFoldResult> lhs,
                                             size_t lhsTailConsumed,
                                             ArrayRef<OpFoldResult> rhs,
                                             size_t rhsTailConsumed,
                                             MLIRContext *ctx) {
  // Build a Variable for the single rhs element.
  AffineExpr rhsExpr = getAffineConstantExpr(1, ctx);
  SmallVector<Value> rhsOperands;
  buildProductExpr(rhs[rhs.size() - rhsTailConsumed - 1], rhsExpr, rhsOperands,
                   ctx);
```
- **EN**: Implements logic around `tryMatchProduct`, `getAffineConstantExpr`, `buildProductExpr`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `tryMatchProduct`, `getAffineConstantExpr`, `buildProductExpr` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 56-69
```cpp
  ValueBoundsConstraintSet::Variable rhsVar(
      AffineMap::get(0, rhsOperands.size(), rhsExpr, ctx), rhsOperands);

  // Incrementally accumulate lhs product and check for equality.
  AffineExpr lhsExpr = getAffineConstantExpr(1, ctx);
  SmallVector<Value> lhsOperands;
  for (size_t k = 1; k + lhsTailConsumed <= lhs.size(); ++k) {
    buildProductExpr(lhs[lhs.size() - lhsTailConsumed - k], lhsExpr,
                     lhsOperands, ctx);
    AffineMap lhsMap = AffineMap::get(0, lhsOperands.size(), lhsExpr, ctx);
    ValueBoundsConstraintSet::Variable lhsVar(lhsMap, lhsOperands);
    FailureOr<bool> result = ValueBoundsConstraintSet::areEqual(lhsVar, rhsVar);
    if (succeeded(result) && *result)
      return k;
```
- **EN**: Implements logic around `rhsVar`, `get`, `getAffineConstantExpr`, `size`, and 4 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `rhsVar`, `get`, `getAffineConstantExpr`, `size`, and 4 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 70-76
```cpp
  }
  return std::nullopt;
}

namespace {

/// Simplify delinearize(linearize) pairs from the tail by matching groups of
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 77-83
```cpp
/// dimensions whose basis products are equal via ValueBounds analysis.
///
/// For each step from the tail, tries:
///   1. Many-to-one: k linearize dims -> 1 delinearize dim
///   2. One-to-many: 1 linearize dim -> k delinearize dims
///
/// Matched trailing dimensions are peeled off. Unmatched prefix dimensions
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 84-90
```cpp
/// are left as residual linearize/delinearize operations.
///
/// Example (many-to-one, D*E == Z):
///   %lin = affine.linearize_index disjoint [%a, %b, %c, %d, %e]
///              by (A, B, C, D, E)
///   %result:3 = affine.delinearize_index %lin into (X, Y, Z)
/// ->
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 91-98
```cpp
///   %prefix_lin = affine.linearize_index disjoint [%a, %b, %c] by (A, B, C)
///   %prefix:2 = affine.delinearize_index %prefix_lin into (X, Y)
///   %tail = affine.linearize_index disjoint [%d, %e] by (D, E)
///   %result = [%prefix#0, %prefix#1, %tail]
struct SimplifyDelinearizeOfLinearizeDisjoint final
    : OpRewritePattern<AffineDelinearizeIndexOp> {
  using Base::Base;

```
- **EN**: Introduces declarations for `SimplifyDelinearizeOfLinearizeDisjoint`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SimplifyDelinearizeOfLinearizeDisjoint` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 99-106
```cpp
  LogicalResult matchAndRewrite(AffineDelinearizeIndexOp delinearizeOp,
                                PatternRewriter &rewriter) const override {
    auto linearizeOp =
        delinearizeOp.getLinearIndex().getDefiningOp<AffineLinearizeIndexOp>();
    if (!linearizeOp)
      return rewriter.notifyMatchFailure(delinearizeOp,
                                         "index doesn't come from linearize");

```
- **EN**: Implements logic around `matchAndRewrite`, `getLinearIndex`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getLinearIndex`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 107-114
```cpp
    if (!linearizeOp.getDisjoint())
      return rewriter.notifyMatchFailure(linearizeOp, "not disjoint");

    SmallVector<OpFoldResult> linBasis = linearizeOp.getMixedBasis();
    SmallVector<OpFoldResult> delinBasis = delinearizeOp.getMixedBasis();
    ValueRange linInputs = linearizeOp.getMultiIndex();
    MLIRContext *ctx = rewriter.getContext();

```
- **EN**: Implements logic around `getDisjoint`, `notifyMatchFailure`, `getMixedBasis`, `getMultiIndex`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getDisjoint`, `notifyMatchFailure`, `getMixedBasis`, `getMultiIndex`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 115-123
```cpp
    // Track how many elements consumed from each tail.
    size_t linTailConsumed = 0;
    size_t delinTailConsumed = 0;

    // For each matched group (innermost first), record the number of
    // linearize and delinearize dimensions it spans. Many-to-one groups
    // have linCount > 1, one-to-many groups have delinCount > 1.
    SmallVector<std::pair<size_t, size_t>> matchedGroups;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 124-137
```cpp
    while (linTailConsumed < linBasis.size() &&
           delinTailConsumed < delinBasis.size()) {
      // Try many-to-one: k lin dims -> 1 delin dim.
      if (std::optional<size_t> k = tryMatchProduct(
              linBasis, linTailConsumed, delinBasis, delinTailConsumed, ctx)) {
        matchedGroups.emplace_back(*k, 1);
        linTailConsumed += *k;
        delinTailConsumed += 1;
        continue;
      }
      // Try one-to-many: 1 lin dim -> k delin dims.
      if (std::optional<size_t> k = tryMatchProduct(
              delinBasis, delinTailConsumed, linBasis, linTailConsumed, ctx)) {
        matchedGroups.emplace_back(1, *k);
```
- **EN**: Implements logic around `size`, `tryMatchProduct`, `emplace_back`.
- **CN**: 围绕 `size`, `tryMatchProduct`, `emplace_back` 实现具体逻辑。

### Lines 138-144
```cpp
        delinTailConsumed += *k;
        linTailConsumed += 1;
        continue;
      }
      break;
    }

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 145-158
```cpp
    if (matchedGroups.empty())
      return rewriter.notifyMatchFailure(delinearizeOp,
                                         "no trailing dimensions matched");

    SmallVector<Value> results;

    // Build residual prefix ops for unmatched dimensions.
    if (delinTailConsumed < delinBasis.size()) {
      // Partial match: create residual linearize + delinearize for the
      // unmatched prefix.
      Value residualLinearize = AffineLinearizeIndexOp::create(
          rewriter, linearizeOp.getLoc(), linInputs.drop_back(linTailConsumed),
          ArrayRef(linBasis).drop_back(linTailConsumed),
          linearizeOp.getDisjoint());
```
- **EN**: Implements logic around `empty`, `notifyMatchFailure`, `size`, `create`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `empty`, `notifyMatchFailure`, `size`, `create`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 159-172
```cpp
      auto residualDelinearize = AffineDelinearizeIndexOp::create(
          rewriter, delinearizeOp.getLoc(), residualLinearize,
          ArrayRef(delinBasis).drop_back(delinTailConsumed),
          delinearizeOp.hasOuterBound());
      results.append(residualDelinearize.getResults().begin(),
                     residualDelinearize.getResults().end());
    } else if (!delinearizeOp.hasOuterBound()) {
      // All basis elements consumed, but the original delinearize has no outer
      // bound which requires special handling.
      ValueRange remainingInputs = linInputs.drop_back(linTailConsumed);
      if (remainingInputs.empty()) {
        // The outermost delinearize result is guaranteed to be zero.
        results.push_back(arith::ConstantIndexOp::create(
            rewriter, delinearizeOp.getLoc(), 0));
```
- **EN**: Implements logic around `create`, `getLoc`, `ArrayRef`, `hasOuterBound`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `getLoc`, `ArrayRef`, `hasOuterBound`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 173-185
```cpp
      } else if (remainingInputs.size() == 1) {
        // Pass through the single remaining input.
        results.push_back(remainingInputs.front());
      } else {
        // Re-linearize the remaining inputs to produce the outermost result.
        Value newLin = AffineLinearizeIndexOp::create(
            rewriter, linearizeOp.getLoc(), remainingInputs,
            ArrayRef(linBasis).drop_back(linTailConsumed),
            linearizeOp.getDisjoint());
        results.push_back(newLin);
      }
    }

```
- **EN**: Implements logic around `size`, `push_back`, `create`, `getLoc`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `size`, `push_back`, `create`, `getLoc`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 186-199
```cpp
    // Build results for each matched group.
    size_t linInputOffset = linInputs.size() - linTailConsumed;
    size_t linBasisOffset = linBasis.size() - linTailConsumed;
    size_t delinBasisOffset = delinBasis.size() - delinTailConsumed;
    for (auto [linCount, delinCount] : llvm::reverse(matchedGroups)) {
      if (linCount == 1 && delinCount == 1) {
        // Exact 1:1 match: pass through directly.
        results.push_back(linInputs[linInputOffset]);
      } else if (linCount > 1) {
        // Many-to-one: re-linearize the group's lin inputs.
        Value newLin = AffineLinearizeIndexOp::create(
            rewriter, linearizeOp.getLoc(),
            linInputs.slice(linInputOffset, linCount),
            ArrayRef(linBasis).slice(linBasisOffset, linCount),
```
- **EN**: Implements logic around `size`, `reverse`, `push_back`, `create`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `size`, `reverse`, `push_back`, `create`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 200-213
```cpp
            /*disjoint=*/true);
        results.push_back(newLin);
      } else {
        // One-to-many: delinearize the single lin input.
        auto newDelin = AffineDelinearizeIndexOp::create(
            rewriter, delinearizeOp.getLoc(), linInputs[linInputOffset],
            ArrayRef(delinBasis).slice(delinBasisOffset, delinCount),
            /*hasOuterBound=*/true);
        results.append(newDelin.getResults().begin(),
                       newDelin.getResults().end());
      }
      linInputOffset += linCount;
      linBasisOffset += linCount;
      delinBasisOffset += delinCount;
```
- **EN**: Implements logic around `push_back`, `create`, `getLoc`, `ArrayRef`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `push_back`, `create`, `getLoc`, `ArrayRef`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 214-220
```cpp
    }

    rewriter.replaceOp(delinearizeOp, results);
    return success();
  }
};

```
- **EN**: Implements logic around `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 221-227
```cpp
} // namespace

void affine::populateSimplifyAffineWithBoundsPatterns(
    RewritePatternSet &patterns) {
  patterns.add<SimplifyDelinearizeOfLinearizeDisjoint>(patterns.getContext());
}

```
- **EN**: Implements logic around `populateSimplifyAffineWithBoundsPatterns`, `add`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `populateSimplifyAffineWithBoundsPatterns`, `add` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 228-238
```cpp
//===----------------------------------------------------------------------===//
// Pass definition
//===----------------------------------------------------------------------===//

namespace mlir {
namespace affine {
#define GEN_PASS_DEF_SIMPLIFYAFFINEWITHBOUNDS
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 239-252
```cpp
namespace {
struct SimplifyAffineWithBoundsPass
    : affine::impl::SimplifyAffineWithBoundsBase<SimplifyAffineWithBoundsPass> {
  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());
    // Add canonicalization patterns first so cheap exact-match cases are
    // handled without invoking value bounds analysis.
    AffineDelinearizeIndexOp::getCanonicalizationPatterns(patterns,
                                                          &getContext());
    AffineLinearizeIndexOp::getCanonicalizationPatterns(patterns,
                                                        &getContext());
    populateSimplifyAffineWithBoundsPatterns(patterns);
    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns))))
      return signalPassFailure();
```
- **EN**: Introduces declarations for `SimplifyAffineWithBoundsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SimplifyAffineWithBoundsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 253-255
```cpp
  }
};
} // namespace
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

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
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/PatternMatch.h`, `mlir/Interfaces/ValueBoundsOpInterface.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), MLIR IR core abstractions / MLIR IR 核心抽象 (1), MLIR interface declarations / MLIR 接口声明 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
