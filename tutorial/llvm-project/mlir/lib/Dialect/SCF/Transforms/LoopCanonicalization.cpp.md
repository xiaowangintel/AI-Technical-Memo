# LoopCanonicalization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Transforms/LoopCanonicalization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains cross-dialect canonicalization patterns that cannot be actual canonicalization patterns due to undesired additional dependencies.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Transforms`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LoopCanonicalization.cpp - Cross-dialect canonicalization patterns -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```cpp
//
// This file contains cross-dialect canonicalization patterns that cannot be
// actual canonicalization patterns due to undesired additional dependencies.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SCF/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h`。

### Lines 16-25
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/Dialect/SCF/Utils/AffineCanonicalizationUtils.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`。

### Lines 26-33
```cpp
namespace mlir {
#define GEN_PASS_DEF_SCFFORLOOPCANONICALIZATION
#include "mlir/Dialect/SCF/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::scf;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h.inc`。

### Lines 34-47
```cpp
/// A simple, conservative analysis to determine if the loop is shape
/// conserving. I.e., the type of the arg-th yielded value is the same as the
/// type of the corresponding basic block argument of the loop.
/// Note: This function handles only simple cases. Expand as needed.
static bool isShapePreserving(ForOp forOp, int64_t arg) {
  assert(arg < static_cast<int64_t>(forOp.getNumResults()) &&
         "arg is out of bounds");
  Value value = forOp.getYieldedValues()[arg];
  while (value) {
    if (value == forOp.getRegionIterArgs()[arg])
      return true;
    OpResult opResult = dyn_cast<OpResult>(value);
    if (!opResult)
      return false;
```
- **EN**: Implements logic around `isShapePreserving`, `assert`, `getYieldedValues`, `getRegionIterArgs`.
- **CN**: 围绕 `isShapePreserving`, `assert`, `getYieldedValues`, `getRegionIterArgs` 实现具体逻辑。

### Lines 48-61
```cpp

    using tensor::InsertSliceOp;
    value = llvm::TypeSwitch<Operation *, Value>(opResult.getOwner())
                .Case([&](InsertSliceOp op) { return op.getDest(); })
                .Case([&](ForOp forOp) {
                  return isShapePreserving(forOp, opResult.getResultNumber())
                             ? forOp.getInitArgs()[opResult.getResultNumber()]
                             : Value();
                })
                .Default(nullptr);
  }
  return false;
}

```
- **EN**: Implements logic around `Value>`, `Case`, `isShapePreserving`, `getInitArgs`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Value>`, `Case`, `isShapePreserving`, `getInitArgs`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 62-68
```cpp
namespace {
/// Fold dim ops of iter_args to dim ops of their respective init args. E.g.:
///
/// ```
/// %0 = ... : tensor<?x?xf32>
/// scf.for ... iter_args(%arg0 = %0) -> (tensor<?x?xf32>) {
///   %1 = tensor.dim %arg0, %c0 : tensor<?x?xf32>
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 69-75
```cpp
///   ...
/// }
/// ```
///
/// is folded to:
///
/// ```
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 76-82
```cpp
/// %0 = ... : tensor<?x?xf32>
/// scf.for ... iter_args(%arg0 = %0) -> (tensor<?x?xf32>) {
///   %1 = tensor.dim %0, %c0 : tensor<?x?xf32>
///   ...
/// }
/// ```
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 83-96
```cpp
/// Note: Dim ops are folded only if it can be proven that the runtime type of
/// the iter arg does not change with loop iterations.
template <typename OpTy>
struct DimOfIterArgFolder : public OpRewritePattern<OpTy> {
  using OpRewritePattern<OpTy>::OpRewritePattern;

  LogicalResult matchAndRewrite(OpTy dimOp,
                                PatternRewriter &rewriter) const override {
    auto blockArg = dyn_cast<BlockArgument>(dimOp.getSource());
    if (!blockArg)
      return failure();
    auto forOp = dyn_cast<ForOp>(blockArg.getParentBlock()->getParentOp());
    if (!forOp)
      return failure();
```
- **EN**: Introduces declarations for `DimOfIterArgFolder`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DimOfIterArgFolder` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 97-103
```cpp
    if (!isShapePreserving(forOp, blockArg.getArgNumber() - 1))
      return failure();

    Value initArg = forOp.getTiedLoopInit(blockArg)->get();
    rewriter.modifyOpInPlace(
        dimOp, [&]() { dimOp.getSourceMutable().assign(initArg); });

```
- **EN**: Implements logic around `isShapePreserving`, `failure`, `getTiedLoopInit`, `modifyOpInPlace`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `isShapePreserving`, `failure`, `getTiedLoopInit`, `modifyOpInPlace`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 104-110
```cpp
    return success();
  };
};

/// Fold dim ops of loop results to dim ops of their respective init args. E.g.:
///
/// ```
```
- **EN**: Implements logic around `success`; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `success` 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 111-117
```cpp
/// %0 = ... : tensor<?x?xf32>
/// %r = scf.for ... iter_args(%arg0 = %0) -> (tensor<?x?xf32>) {
///   ...
/// }
/// %1 = tensor.dim %r, %c0 : tensor<?x?xf32>
/// ```
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 118-124
```cpp
/// is folded to:
///
/// ```
/// %0 = ... : tensor<?x?xf32>
/// %r = scf.for ... iter_args(%arg0 = %0) -> (tensor<?x?xf32>) {
///   ...
/// }
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 125-131
```cpp
/// %1 = tensor.dim %0, %c0 : tensor<?x?xf32>
/// ```
///
/// Note: Dim ops are folded only if it can be proven that the runtime type of
/// the iter arg does not change with loop iterations.
template <typename OpTy>
struct DimOfLoopResultFolder : public OpRewritePattern<OpTy> {
```
- **EN**: Introduces declarations for `DimOfLoopResultFolder`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DimOfLoopResultFolder` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 132-145
```cpp
  using OpRewritePattern<OpTy>::OpRewritePattern;

  LogicalResult matchAndRewrite(OpTy dimOp,
                                PatternRewriter &rewriter) const override {
    auto forOp = dimOp.getSource().template getDefiningOp<scf::ForOp>();
    if (!forOp)
      return failure();
    auto opResult = cast<OpResult>(dimOp.getSource());
    unsigned resultNumber = opResult.getResultNumber();
    if (!isShapePreserving(forOp, resultNumber))
      return failure();
    rewriter.modifyOpInPlace(dimOp, [&]() {
      dimOp.getSourceMutable().assign(forOp.getInitArgs()[resultNumber]);
    });
```
- **EN**: Implements logic around `matchAndRewrite`, `getSource`, `failure`, `getResultNumber`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `getSource`, `failure`, `getResultNumber`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 146-153
```cpp
    return success();
  }
};

/// Canonicalize AffineMinOp/AffineMaxOp operations in the context of scf.for
/// and scf.parallel loops with a known range.
template <typename OpTy>
struct AffineOpSCFCanonicalizationPattern : public OpRewritePattern<OpTy> {
```
- **EN**: Introduces declarations for `AffineOpSCFCanonicalizationPattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `AffineOpSCFCanonicalizationPattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 154-161
```cpp
  using OpRewritePattern<OpTy>::OpRewritePattern;

  LogicalResult matchAndRewrite(OpTy op,
                                PatternRewriter &rewriter) const override {
    return scf::canonicalizeMinMaxOpInLoop(rewriter, op, scf::matchForLikeLoop);
  }
};

```
- **EN**: Implements logic around `matchAndRewrite`, `canonicalizeMinMaxOpInLoop`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `canonicalizeMinMaxOpInLoop` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 162-174
```cpp
struct SCFForLoopCanonicalization
    : public impl::SCFForLoopCanonicalizationBase<SCFForLoopCanonicalization> {
  void runOnOperation() override {
    auto *parentOp = getOperation();
    MLIRContext *ctx = parentOp->getContext();
    RewritePatternSet patterns(ctx);
    scf::populateSCFForLoopCanonicalizationPatterns(patterns);
    if (failed(applyPatternsGreedily(parentOp, std::move(patterns))))
      signalPassFailure();
  }
};
} // namespace

```
- **EN**: Introduces declarations for `SCFForLoopCanonicalization`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SCFForLoopCanonicalization` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 175-185
```cpp
void mlir::scf::populateSCFForLoopCanonicalizationPatterns(
    RewritePatternSet &patterns) {
  MLIRContext *ctx = patterns.getContext();
  patterns
      .add<AffineOpSCFCanonicalizationPattern<affine::AffineMinOp>,
           AffineOpSCFCanonicalizationPattern<affine::AffineMaxOp>,
           DimOfIterArgFolder<tensor::DimOp>, DimOfIterArgFolder<memref::DimOp>,
           DimOfLoopResultFolder<tensor::DimOp>,
           DimOfLoopResultFolder<memref::DimOp>>(ctx);
}

```
- **EN**: Implements logic around `populateSCFForLoopCanonicalizationPatterns`, `getContext`, `DimOp>>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateSCFForLoopCanonicalizationPatterns`, `getContext`, `DimOp>>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 186-188
```cpp
std::unique_ptr<Pass> mlir::createSCFForLoopCanonicalizationPass() {
  return std::make_unique<SCFForLoopCanonicalization>();
}
```
- **EN**: Implements logic around `createSCFForLoopCanonicalizationPass`, `make_unique`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `createSCFForLoopCanonicalizationPass`, `make_unique` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

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
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/Dialect/SCF/Utils/AffineCanonicalizationUtils.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/TypeSwitch.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (8), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
