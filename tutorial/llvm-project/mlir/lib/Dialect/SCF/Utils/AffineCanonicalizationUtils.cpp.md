# AffineCanonicalizationUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Utils/AffineCanonicalizationUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Utility functions to canonicalize affine ops within SCF op regions.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Utils`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineCanonicalizationUtils.cpp - Affine Canonicalization in SCF ---===//
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
//
// Utility functions to canonicalize affine ops within SCF op regions.
//
//===----------------------------------------------------------------------===//

#include <utility>

```
- **EN**: Pulls in the headers needed by this translation unit, including `utility`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `utility`。

### Lines 15-24
```cpp
#include "mlir/Dialect/Affine/Analysis/AffineStructures.h"
#include "mlir/Dialect/Affine/Analysis/Utils.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/IR/AffineValueMap.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Utils/AffineCanonicalizationUtils.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/IR/AffineValueMap.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/IR/AffineValueMap.h`。

### Lines 25-38
```cpp
#define DEBUG_TYPE "mlir-scf-affine-utils"

using namespace mlir;
using namespace affine;
using namespace presburger;

LogicalResult scf::matchForLikeLoop(Value iv, OpFoldResult &lb,
                                    OpFoldResult &ub, OpFoldResult &step) {
  if (scf::ForOp forOp = scf::getForInductionVarOwner(iv)) {
    lb = forOp.getLowerBound();
    ub = forOp.getUpperBound();
    step = forOp.getStep();
    return success();
  }
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 39-52
```cpp
  if (scf::ParallelOp parOp = scf::getParallelForInductionVarOwner(iv)) {
    for (unsigned idx = 0; idx < parOp.getNumLoops(); ++idx) {
      if (parOp.getInductionVars()[idx] == iv) {
        lb = parOp.getLowerBound()[idx];
        ub = parOp.getUpperBound()[idx];
        step = parOp.getStep()[idx];
        return success();
      }
    }
    return failure();
  }
  if (scf::ForallOp forallOp = scf::getForallOpThreadIndexOwner(iv)) {
    for (int64_t idx = 0; idx < forallOp.getRank(); ++idx) {
      if (forallOp.getInductionVar(idx) == iv) {
```
- **EN**: Implements logic around `getParallelForInductionVarOwner`, `getNumLoops`, `getInductionVars`, `getLowerBound`, and 7 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getParallelForInductionVarOwner`, `getNumLoops`, `getInductionVars`, `getLowerBound`, and 7 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 53-63
```cpp
        lb = forallOp.getMixedLowerBound()[idx];
        ub = forallOp.getMixedUpperBound()[idx];
        step = forallOp.getMixedStep()[idx];
        return success();
      }
    }
    return failure();
  }
  return failure();
}

```
- **EN**: Implements logic around `getMixedLowerBound`, `getMixedUpperBound`, `getMixedStep`, `success`, and 1 more symbols.
- **CN**: 围绕 `getMixedLowerBound`, `getMixedUpperBound`, `getMixedStep`, `success`, and 1 more symbols 实现具体逻辑。

### Lines 64-76
```cpp
static FailureOr<AffineApplyOp>
canonicalizeMinMaxOp(RewriterBase &rewriter, Operation *op,
                     FlatAffineValueConstraints constraints) {
  RewriterBase::InsertionGuard guard(rewriter);
  rewriter.setInsertionPoint(op);
  FailureOr<AffineValueMap> simplified =
      affine::simplifyConstrainedMinMaxOp(op, std::move(constraints));
  if (failed(simplified))
    return failure();
  return rewriter.replaceOpWithNewOp<AffineApplyOp>(
      op, simplified->getAffineMap(), simplified->getOperands());
}

```
- **EN**: Implements logic around `canonicalizeMinMaxOp`, `guard`, `setInsertionPoint`, `simplifyConstrainedMinMaxOp`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `canonicalizeMinMaxOp`, `guard`, `setInsertionPoint`, `simplifyConstrainedMinMaxOp`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 77-87
```cpp
LogicalResult scf::addLoopRangeConstraints(FlatAffineValueConstraints &cstr,
                                           Value iv, OpFoldResult lb,
                                           OpFoldResult ub, OpFoldResult step) {
  Builder b(iv.getContext());

  // IntegerPolyhedron does not support semi-affine expressions.
  // Therefore, only constant step values are supported.
  auto stepInt = getConstantIntValue(step);
  if (!stepInt)
    return failure();

```
- **EN**: Implements logic around `addLoopRangeConstraints`, `b`, `getConstantIntValue`, `failure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `addLoopRangeConstraints`, `b`, `getConstantIntValue`, `failure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 88-95
```cpp
  unsigned dimIv = cstr.appendDimVar(iv);
  auto lbv = llvm::dyn_cast_if_present<Value>(lb);
  unsigned symLb =
      lbv ? cstr.appendSymbolVar(lbv) : cstr.appendSymbolVar(/*num=*/1);
  auto ubv = llvm::dyn_cast_if_present<Value>(ub);
  unsigned symUb =
      ubv ? cstr.appendSymbolVar(ubv) : cstr.appendSymbolVar(/*num=*/1);

```
- **EN**: Implements logic around `appendDimVar`, `dyn_cast_if_present`, `appendSymbolVar`.
- **CN**: 围绕 `appendDimVar`, `dyn_cast_if_present`, `appendSymbolVar` 实现具体逻辑。

### Lines 96-103
```cpp
  // If loop lower/upper bounds are constant: Add EQ constraint.
  std::optional<int64_t> lbInt = getConstantIntValue(lb);
  std::optional<int64_t> ubInt = getConstantIntValue(ub);
  if (lbInt)
    cstr.addBound(BoundType::EQ, symLb, *lbInt);
  if (ubInt)
    cstr.addBound(BoundType::EQ, symUb, *ubInt);

```
- **EN**: Implements logic around `getConstantIntValue`, `addBound`.
- **CN**: 围绕 `getConstantIntValue`, `addBound` 实现具体逻辑。

### Lines 104-117
```cpp
  // Lower bound: iv >= lb (equiv.: iv - lb >= 0)
  SmallVector<int64_t> ineqLb(cstr.getNumCols(), 0);
  ineqLb[dimIv] = 1;
  ineqLb[symLb] = -1;
  cstr.addInequality(ineqLb);

  // Upper bound
  AffineExpr ivUb;
  if (lbInt && ubInt && (*lbInt + *stepInt >= *ubInt)) {
    // The loop has at most one iteration.
    // iv < lb + 1
    // TODO: Try to derive this constraint by simplifying the expression in
    // the else-branch.
    ivUb = b.getAffineSymbolExpr(symLb - cstr.getNumDimVars()) + 1;
```
- **EN**: Implements logic around `ineqLb`, `addInequality`, `getAffineSymbolExpr`.
- **CN**: 围绕 `ineqLb`, `addInequality`, `getAffineSymbolExpr` 实现具体逻辑。

### Lines 118-131
```cpp
  } else {
    // The loop may have more than one iteration.
    // iv < lb + step * ((ub - lb - 1) floorDiv step) + 1
    AffineExpr exprLb =
        lbInt ? b.getAffineConstantExpr(*lbInt)
              : b.getAffineSymbolExpr(symLb - cstr.getNumDimVars());
    AffineExpr exprUb =
        ubInt ? b.getAffineConstantExpr(*ubInt)
              : b.getAffineSymbolExpr(symUb - cstr.getNumDimVars());
    ivUb = exprLb + 1 + (*stepInt * ((exprUb - exprLb - 1).floorDiv(*stepInt)));
  }
  auto map = AffineMap::get(
      /*dimCount=*/cstr.getNumDimVars(),
      /*symbolCount=*/cstr.getNumSymbolVars(), /*result=*/ivUb);
```
- **EN**: Implements logic around `getAffineConstantExpr`, `getAffineSymbolExpr`, `floorDiv`, `get`, and 2 more symbols.
- **CN**: 围绕 `getAffineConstantExpr`, `getAffineSymbolExpr`, `floorDiv`, `get`, and 2 more symbols 实现具体逻辑。

### Lines 132-138
```cpp

  return cstr.addBound(BoundType::UB, dimIv, map);
}

/// Canonicalize min/max operations in the context of for loops with a known
/// range. Call `canonicalizeMinMaxOp` and add the following constraints to
/// the constraint system (along with the missing dimensions):
```
- **EN**: Implements logic around `addBound`; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `addBound` 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 139-150
```cpp
///
/// * iv >= lb
/// * iv < lb + step * ((ub - lb - 1) floorDiv step) + 1
///
/// Note: Due to limitations of IntegerPolyhedron, only constant step sizes
/// are currently supported.
LogicalResult scf::canonicalizeMinMaxOpInLoop(RewriterBase &rewriter,
                                              Operation *op,
                                              LoopMatcherFn loopMatcher) {
  FlatAffineValueConstraints constraints;
  DenseSet<Value> allIvs;

```
- **EN**: Implements logic around `canonicalizeMinMaxOpInLoop`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `canonicalizeMinMaxOpInLoop` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 151-164
```cpp
  // Find all iteration variables among `minOp`'s operands add constrain them.
  for (Value operand : op->getOperands()) {
    // Skip duplicate ivs.
    if (allIvs.contains(operand))
      continue;

    // If `operand` is an iteration variable: Find corresponding loop
    // bounds and step.
    Value iv = operand;
    OpFoldResult lb, ub, step;
    if (failed(loopMatcher(operand, lb, ub, step)))
      continue;
    allIvs.insert(iv);

```
- **EN**: Implements logic around `getOperands`, `contains`, `failed`, `insert`.
- **CN**: 围绕 `getOperands`, `contains`, `failed`, `insert` 实现具体逻辑。

### Lines 165-171
```cpp
    if (failed(addLoopRangeConstraints(constraints, iv, lb, ub, step)))
      return failure();
  }

  return canonicalizeMinMaxOp(rewriter, op, constraints);
}

```
- **EN**: Implements logic around `failed`, `failure`, `canonicalizeMinMaxOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `failed`, `failure`, `canonicalizeMinMaxOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 172-178
```cpp
/// Try to simplify the given affine.min/max operation `op` after loop peeling.
/// This function can simplify min/max operations such as (ub is the previous
/// upper bound of the unpeeled loop):
/// ```
/// #map = affine_map<(d0)[s0, s1] -> (s0, -d0 + s1)>
/// %r = affine.min #affine.min #map(%iv)[%step, %ub]
/// ```
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 179-185
```cpp
/// and rewrites them into (in the case the peeled loop):
/// ```
/// %r = %step
/// ```
/// min/max operations inside the partial iteration are rewritten in a similar
/// way.
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 186-192
```cpp
/// This function builds up a set of constraints, capable of proving that:
/// * Inside the peeled loop: min(step, ub - iv) == step
/// * Inside the partial iteration: min(step, ub - iv) == ub - iv
///
/// Returns `success` if the given operation was replaced by a new operation;
/// `failure` otherwise.
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 193-206
```cpp
/// Note: `ub` is the previous upper bound of the loop (before peeling).
/// `insideLoop` must be true for min/max ops inside the loop and false for
/// affine.min ops inside the partial iteration. For an explanation of the other
/// parameters, see comment of `canonicalizeMinMaxOpInLoop`.
LogicalResult scf::rewritePeeledMinMaxOp(RewriterBase &rewriter, Operation *op,
                                         Value iv, Value ub, Value step,
                                         bool insideLoop) {
  FlatAffineValueConstraints constraints;
  constraints.appendDimVar({iv});
  constraints.appendSymbolVar({ub, step});
  if (auto constUb = getConstantIntValue(ub))
    constraints.addBound(BoundType::EQ, 1, *constUb);
  if (auto constStep = getConstantIntValue(step))
    constraints.addBound(BoundType::EQ, 2, *constStep);
```
- **EN**: Implements logic around `rewritePeeledMinMaxOp`, `appendDimVar`, `appendSymbolVar`, `getConstantIntValue`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `rewritePeeledMinMaxOp`, `appendDimVar`, `appendSymbolVar`, `getConstantIntValue`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 207-220
```cpp

  // Add loop peeling invariant. This is the main piece of knowledge that
  // enables AffineMinOp simplification.
  if (insideLoop) {
    // ub - iv >= step (equiv.: -iv + ub - step + 0 >= 0)
    // Intuitively: Inside the peeled loop, every iteration is a "full"
    // iteration, i.e., step divides the iteration space `ub - lb` evenly.
    constraints.addInequality({-1, 1, -1, 0});
  } else {
    // ub - iv < step (equiv.: iv + -ub + step - 1 >= 0)
    // Intuitively: `iv` is the split bound here, i.e., the iteration variable
    // value of the very last iteration (in the unpeeled loop). At that point,
    // there are less than `step` elements remaining. (Otherwise, the peeled
    // loop would run for at least one more iteration.)
```
- **EN**: Implements logic around `addInequality`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `addInequality` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 221-225
```cpp
    constraints.addInequality({1, -1, 1, -1});
  }

  return canonicalizeMinMaxOp(rewriter, op, constraints);
}
```
- **EN**: Implements logic around `addInequality`, `canonicalizeMinMaxOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `addInequality`, `canonicalizeMinMaxOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

## Key Concepts / 关键概念

- **Dialect utilities / 方言工具**:
  - **EN**: Collects reusable helpers that keep core dialect logic factored and shareable.
  - **CN**: 汇集可复用辅助函数，使核心方言逻辑保持解耦并可共享。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/IR/AffineValueMap.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Utils/AffineCanonicalizationUtils.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/AffineMap.h`, `mlir/IR/PatternMatch.h`
- **Standard-library headers / 标准库头文件**: `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (7), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2)
