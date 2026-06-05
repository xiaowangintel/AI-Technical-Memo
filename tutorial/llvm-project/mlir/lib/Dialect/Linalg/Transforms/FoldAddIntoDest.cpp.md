# FoldAddIntoDest.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/FoldAddIntoDest.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `FoldAddIntoDest`.
  - **CN**: 实现 Linalg 方言中围绕 `FoldAddIntoDest` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- FoldAddIntoDest.cpp ---------------------------------------*- C++-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp

#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/IR/LinalgInterfaces.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/Dominance.h"
#include "mlir/Interfaces/DestinationStyleOpInterface.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/Dominance.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/Dominance.h`。

### Lines 15-21
```cpp
using namespace mlir;

// Determine whether the value is defined to be zero.
static bool isDefinedAsZero(Value val) {
  if (!val)
    return false;

```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 22-26
```cpp
  // Check whether val is a constant scalar / vector splat / tensor splat float
  // or integer zero.
  if (isZeroIntegerOrFloat(val))
    return true;

```
- **EN**: Implements logic around `isZeroIntegerOrFloat`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isZeroIntegerOrFloat` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 27-36
```cpp
  auto *defOp = val.getDefiningOp();
  if (!defOp)
    return false;

  return TypeSwitch<Operation *, bool>(defOp)
      .Case<linalg::FillOp, linalg::CopyOp>([&](auto op) {
        return op.getInputs().size() == 1 && isDefinedAsZero(op.getInputs()[0]);
      })
      .Default([&](auto) { return false; });
}
```
- **EN**: Implements logic around `getDefiningOp`, `bool>`, `CopyOp>`, `getInputs`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDefiningOp`, `bool>`, `CopyOp>`, `getInputs`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 37-41
```cpp

/// Replace a linalg.add with one operand the single user of a contraction,
/// which has a zero-filled, "identity-mapped" destination and is dominated by
/// the `other` operand, by the contraction with `other` as its dest.
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 42-46
```cpp
/// As an example, the following pseudo-code will be rewritten
///   %cst = arith.constant 0.000000e+00
///   %empty = tensor.empty()
///   %zeroed = linalg.fill ins(%cst : f32) outs(%empty : !type) -> !type
///   %C = linalg.matmul ins(%A, %B) outs(%zeroed)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 47-51
```cpp
///   %empty2 = tensor.empty()
///   %zeroed2 = linalg.fill ins(%cst : f32) outs(%empty2 : !type) -> !type
///   %F = linalg.matmul ins(%D, %E) outs(%zeroed2)
///   %out = linalg.add ins(%C, %F) outs(%empty)
/// to:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 52-56
```cpp
///   %cst = arith.constant 0.000000e+00
///   %empty = tensor.empty()
///   %zeroed = linalg.fill ins(%cst : f32) outs(%empty : !type) -> !type
///   %C = linalg.matmul ins(%A, %B) outs(%zeroed)
///   %out = linalg.matmul ins(%D, %E) outs(%C)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 57-66
```cpp
///
struct FoldAddIntoDest final : public OpRewritePattern<linalg::AddOp> {
  using OpRewritePattern<linalg::AddOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(linalg::AddOp addOp,
                                PatternRewriter &rewriter) const override {
    // For now, pattern only applies on tensor types (memref support is TODO).
    if (!addOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Introduces declarations for `FoldAddIntoDest`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldAddIntoDest` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 67-72
```cpp
    Value dominatingOperand = nullptr;
    linalg::LinalgOp dominatedOp = nullptr;
    { // We will forget about which operand was left or right after this block.
      Value lhs = addOp.getInputs()[0];
      Value rhs = addOp.getInputs()[1];

```
- **EN**: Implements logic around `getInputs`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getInputs` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 73-82
```cpp
      // Can only put one of addOp's operands in the dest/out arg of the other's
      // defining op based on suitable dominance.
      // TODO: Can be generalized to move ops around as long as that still
      //       respects use-def chains and doesn't affect side-effects.
      if (auto rhsOp = rhs.getDefiningOp<linalg::LinalgOp>()) {
        DominanceInfo domInfo(rhsOp);
        if (domInfo.properlyDominates(lhs, rhsOp)) {
          dominatingOperand = lhs;
          dominatedOp = rhsOp;
        }
```
- **EN**: Implements logic around `LinalgOp>`, `domInfo`, `properlyDominates`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LinalgOp>`, `domInfo`, `properlyDominates` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 83-92
```cpp
      }
      if (auto lhsOp = lhs.getDefiningOp<linalg::LinalgOp>()) {
        DominanceInfo domInfo(lhsOp);
        if (domInfo.properlyDominates(rhs, lhsOp)) {
          dominatingOperand = rhs;
          dominatedOp = lhsOp;
        }
      }
      if (!dominatingOperand || !dominatedOp)
        return failure();
```
- **EN**: Implements logic around `LinalgOp>`, `domInfo`, `properlyDominates`, `failure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `LinalgOp>`, `domInfo`, `properlyDominates`, `failure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 93-102
```cpp
      // NB: As linalg.add's generalisation ignores the out argument in its
      //     region there is no need to perform checks on addOp's out argument.
    }

    // When dominated op is a contraction we know it accumulates on its out arg.
    // E.g., AddOp is not a contraction and hence ignores its out arg's value.
    // TODO: Generalize check to also pass in case of other LinalgOps that
    //       accumulate on their out arg but are not (binary) contraction ops.
    auto dominatedDestOp =
        dyn_cast<DestinationStyleOpInterface>((Operation *)dominatedOp);
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 103-109
```cpp
    if (dominatedOp->getNumResults() != 1 ||
        !linalg::isaContractionOpInterface(dominatedOp) ||
        (!dominatedDestOp || dominatedDestOp.getNumDpsInits() != 1))
      return rewriter.notifyMatchFailure(
          dominatedOp, "expected dominated op to be single-result "
                       "destination-passing contraction");

```
- **EN**: Implements logic around `getNumResults`, `isaContractionOpInterface`, `getNumDpsInits`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumResults`, `isaContractionOpInterface`, `getNumDpsInits`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 110-115
```cpp
    // To change the contraction's result, `addOp` must be its only user.
    if (!dominatedOp->getResult(0).hasOneUse())
      return rewriter.notifyMatchFailure(
          dominatedOp,
          "expected linalg.add to be single user of contraction's result");

```
- **EN**: Implements logic around `getResult`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getResult`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 116-124
```cpp
    // As `dominatedOp` was already accumulating on its out argument, it is only
    // safe to no longer use its current out arg when it is the additive ident.
    auto *destOperand = dominatedDestOp.getDpsInitOperand(0);
    if (!isDefinedAsZero(destOperand->get()))
      return rewriter.notifyMatchFailure(
          dominatedOp, "expected dominated op's dest to be additive zero");
    // TODO: If the other op is a contraction and has additive ident as dest, we
    // can swap the dests and achieve the proper sum, given suitable dominance.

```
- **EN**: Implements logic around `getDpsInitOperand`, `isDefinedAsZero`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getDpsInitOperand`, `isDefinedAsZero`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 125-134
```cpp
    // As an operand to `addOp`, `dominatingOperand` has an identity affine_map.
    // Hence, we can only substitute `dominatingOperand` for the dest of the
    // contraction when dest's indexing_map corresponds to an identity map
    // w.r.t. just the dimensions of dest, i.e. is an ordered projection.
    SmallVector<AffineMap> indexMaps = dominatedOp.getIndexingMapsArray();
    int prevDimPos = -1;
    for (auto expr : indexMaps[destOperand->getOperandNumber()].getResults()) {
      auto dim = dyn_cast<AffineDimExpr>(expr);
      if (!dim || prevDimPos > static_cast<int>(dim.getPosition()))
        return rewriter.notifyMatchFailure(
```
- **EN**: Implements logic around `getIndexingMapsArray`, `getOperandNumber`, `static_cast`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getIndexingMapsArray`, `getOperandNumber`, `static_cast`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 135-139
```cpp
            dominatedOp, "expected index_map for contraction's dest to be an "
                         "ordered projection");
      prevDimPos = dim.getPosition();
    }

```
- **EN**: Implements logic around `getPosition`.
- **CN**: 围绕 `getPosition` 实现具体逻辑。

### Lines 140-149
```cpp
    // Replace the additive-ident, i.e. zero, out arg of the dominated op by the
    // dominating summand. This makes the dominated op's result the sum of both
    // of addOp's arguments - therefore we replace addOp and it uses by it.
    rewriter.modifyOpInPlace(
        dominatedOp, [&]() { dominatedOp->setOperand(2, dominatingOperand); });
    rewriter.replaceAllOpUsesWith(addOp, dominatedOp->getResult(0));
    return success();
  }
};

```
- **EN**: Implements logic around `modifyOpInPlace`, `setOperand`, `replaceAllOpUsesWith`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `modifyOpInPlace`, `setOperand`, `replaceAllOpUsesWith`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 150-153
```cpp
void linalg::populateFoldAddIntoDestPatterns(RewritePatternSet &patterns) {
  // Replace linalg.add when destination passing suffices for achieving the sum.
  patterns.add<FoldAddIntoDest>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateFoldAddIntoDestPatterns`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateFoldAddIntoDestPatterns`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/IR/LinalgInterfaces.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/Dominance.h`, `mlir/Interfaces/DestinationStyleOpInterface.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
