# DecomposeAffineOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/DecomposeAffineOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements functionality to progressively decompose coarse-grained affine ops into finer-grained ops.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- DecomposeAffineOps.cpp - Decompose affine ops into finer-grained ---===//
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
// This file implements functionality to progressively decompose coarse-grained
// affine ops into finer-grained ops.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Transforms/Transforms.h"
#include "mlir/IR/PatternMatch.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/InterleavedRange.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `llvm/ADT/SmallVectorExtras.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `llvm/ADT/SmallVectorExtras.h`。

### Lines 22-28
```cpp
using namespace mlir;
using namespace mlir::affine;

#define DEBUG_TYPE "decompose-affine-ops"

/// Count the number of loops surrounding `operand` such that operand could be
/// hoisted above.
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or generated-code switches.
- **CN**: 定义受预处理器控制的结构、特性开关或生成代码开关。

### Lines 29-41
```cpp
/// Stop counting at the first loop over which the operand cannot be hoisted.
static int64_t numEnclosingInvariantLoops(OpOperand &operand) {
  int64_t count = 0;
  Operation *currentOp = operand.getOwner();
  while (auto loopOp = currentOp->getParentOfType<LoopLikeOpInterface>()) {
    if (!loopOp.isDefinedOutsideOfLoop(operand.get()))
      break;
    currentOp = loopOp;
    count++;
  }
  return count;
}

```
- **EN**: Implements logic around `numEnclosingInvariantLoops`, `getOwner`, `getParentOfType`, `isDefinedOutsideOfLoop`.
- **CN**: 围绕 `numEnclosingInvariantLoops`, `getOwner`, `getParentOfType`, `isDefinedOutsideOfLoop` 实现具体逻辑。

### Lines 42-48
```cpp
void mlir::affine::reorderOperandsByHoistability(RewriterBase &rewriter,
                                                 AffineApplyOp op) {
  SmallVector<int64_t> numInvariant =
      llvm::map_to_vector(op->getOpOperands(), [&](OpOperand &operand) {
        return numEnclosingInvariantLoops(operand);
      });

```
- **EN**: Implements logic around `reorderOperandsByHoistability`, `map_to_vector`, `numEnclosingInvariantLoops`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `reorderOperandsByHoistability`, `map_to_vector`, `numEnclosingInvariantLoops` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 49-55
```cpp
  int64_t numOperands = op.getNumOperands();
  SmallVector<int64_t> operandPositions =
      llvm::to_vector(llvm::seq<int64_t>(0, numOperands));
  llvm::stable_sort(operandPositions, [&numInvariant](size_t i1, size_t i2) {
    return numInvariant[i1] > numInvariant[i2];
  });

```
- **EN**: Implements logic around `getNumOperands`, `to_vector`, `stable_sort`.
- **CN**: 围绕 `getNumOperands`, `to_vector`, `stable_sort` 实现具体逻辑。

### Lines 56-62
```cpp
  SmallVector<AffineExpr> replacements(numOperands);
  SmallVector<Value> operands(numOperands);
  for (int64_t i = 0; i < numOperands; ++i) {
    operands[i] = op.getOperand(operandPositions[i]);
    replacements[operandPositions[i]] = getAffineSymbolExpr(i, op.getContext());
  }

```
- **EN**: Implements logic around `replacements`, `operands`, `getOperand`, `getAffineSymbolExpr`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `replacements`, `operands`, `getOperand`, `getAffineSymbolExpr` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 63-73
```cpp
  AffineMap map = op.getAffineMap();
  ArrayRef<AffineExpr> repls{replacements};
  map = map.replaceDimsAndSymbols(repls.take_front(map.getNumDims()),
                                  repls.drop_front(map.getNumDims()),
                                  /*numResultDims=*/0,
                                  /*numResultSyms=*/numOperands);
  map = AffineMap::get(0, numOperands,
                       simplifyAffineExpr(map.getResult(0), 0, numOperands),
                       op->getContext());
  canonicalizeMapAndOperands(&map, &operands);

```
- **EN**: Implements logic around `getAffineMap`, `replaceDimsAndSymbols`, `drop_front`, `get`, and 3 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getAffineMap`, `replaceDimsAndSymbols`, `drop_front`, `get`, and 3 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 74-80
```cpp
  rewriter.startOpModification(op);
  op.setMap(map);
  op->setOperands(operands);
  rewriter.finalizeOpModification(op);
}

/// Build an affine.apply that is a subexpression `expr` of `originalOp`s affine
```
- **EN**: Implements logic around `startOpModification`, `setMap`, `setOperands`, `finalizeOpModification`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `startOpModification`, `setMap`, `setOperands`, `finalizeOpModification` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 81-94
```cpp
/// map and with the same operands.
/// Canonicalize the map and operands to deduplicate and drop dead operands
/// before returning but do not perform maximal composition of AffineApplyOp
/// which would defeat the purpose.
static AffineApplyOp createSubApply(RewriterBase &rewriter,
                                    AffineApplyOp originalOp, AffineExpr expr) {
  MLIRContext *ctx = originalOp->getContext();
  AffineMap m = originalOp.getAffineMap();
  auto rhsMap = AffineMap::get(m.getNumDims(), m.getNumSymbols(), expr, ctx);
  SmallVector<Value> rhsOperands = originalOp->getOperands();
  canonicalizeMapAndOperands(&rhsMap, &rhsOperands);
  return AffineApplyOp::create(rewriter, originalOp.getLoc(), rhsMap,
                               rhsOperands);
}
```
- **EN**: Implements logic around `createSubApply`, `getContext`, `getAffineMap`, `get`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `createSubApply`, `getContext`, `getAffineMap`, `get`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 95-103
```cpp

FailureOr<AffineApplyOp> mlir::affine::decompose(RewriterBase &rewriter,
                                                 AffineApplyOp op) {
  // 1. Preconditions: only handle dimensionless AffineApplyOp maps with a
  // top-level binary expression that we can reassociate (i.e. add or mul).
  AffineMap m = op.getAffineMap();
  if (m.getNumDims() > 0)
    return rewriter.notifyMatchFailure(op, "expected no dims");

```
- **EN**: Implements logic around `decompose`, `getAffineMap`, `getNumDims`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `decompose`, `getAffineMap`, `getNumDims`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 104-112
```cpp
  AffineExpr remainingExp = m.getResult(0);
  auto binExpr = dyn_cast<AffineBinaryOpExpr>(remainingExp);
  if (!binExpr)
    return rewriter.notifyMatchFailure(op, "terminal affine.apply");

  if (!isa<AffineBinaryOpExpr>(binExpr.getLHS()) &&
      !isa<AffineBinaryOpExpr>(binExpr.getRHS()))
    return rewriter.notifyMatchFailure(op, "terminal affine.apply");

```
- **EN**: Implements logic around `getResult`, `dyn_cast`, `notifyMatchFailure`, `isa`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getResult`, `dyn_cast`, `notifyMatchFailure`, `isa` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 113-120
```cpp
  bool supportedKind = ((binExpr.getKind() == AffineExprKind::Add) ||
                        (binExpr.getKind() == AffineExprKind::Mul));
  if (!supportedKind)
    return rewriter.notifyMatchFailure(
        op, "only add or mul binary expr can be reassociated");

  LDBG() << "Start decomposeIntoFinerGrainedOps: " << op;

```
- **EN**: Implements logic around `getKind`, `notifyMatchFailure`, `LDBG`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getKind`, `notifyMatchFailure`, `LDBG` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 121-134
```cpp
  // 2. Iteratively extract the RHS subexpressions while the top-level binary
  // expr kind remains the same.
  MLIRContext *ctx = op->getContext();
  SmallVector<AffineExpr> subExpressions;
  while (true) {
    auto currentBinExpr = dyn_cast<AffineBinaryOpExpr>(remainingExp);
    if (!currentBinExpr || currentBinExpr.getKind() != binExpr.getKind()) {
      subExpressions.push_back(remainingExp);
      LDBG() << "--terminal: " << subExpressions.back();
      break;
    }
    subExpressions.push_back(currentBinExpr.getRHS());
    LDBG() << "--subExpr: " << subExpressions.back();
    remainingExp = currentBinExpr.getLHS();
```
- **EN**: Implements logic around `getContext`, `dyn_cast`, `getKind`, `push_back`, and 2 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getContext`, `dyn_cast`, `getKind`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 135-148
```cpp
  }

  // 3. Reorder subExpressions by the min symbol they are a function of.
  // This also takes care of properly reordering local variables.
  // This however won't be able to split expression that cannot be reassociated
  // such as ones that involve divs and multiple symbols.
  auto getMaxSymbol = [&](AffineExpr e) -> int64_t {
    for (int64_t i = m.getNumSymbols(); i >= 0; --i)
      if (e.isFunctionOfSymbol(i))
        return i;
    return -1;
  };
  llvm::stable_sort(subExpressions, [&](AffineExpr e1, AffineExpr e2) {
    return getMaxSymbol(e1) < getMaxSymbol(e2);
```
- **EN**: Implements logic around `getNumSymbols`, `isFunctionOfSymbol`, `stable_sort`, `getMaxSymbol`; this block performs affine reasoning or shape/bounds manipulation; works with symbol tables or function-like operations.
- **CN**: 围绕 `getNumSymbols`, `isFunctionOfSymbol`, `stable_sort`, `getMaxSymbol` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理符号表或类函数操作。

### Lines 149-158
```cpp
  });
  LDBG() << "--sorted subexprs: " << llvm::interleaved(subExpressions);

  // 4. Merge sorted subExpressions iteratively, thus achieving reassociation.
  auto s0 = getAffineSymbolExpr(0, ctx);
  auto s1 = getAffineSymbolExpr(1, ctx);
  AffineMap binMap = AffineMap::get(
      /*dimCount=*/0, /*symbolCount=*/2,
      getAffineBinaryOpExpr(binExpr.getKind(), s0, s1), ctx);

```
- **EN**: Implements logic around `LDBG`, `getAffineSymbolExpr`, `get`, `getAffineBinaryOpExpr`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `LDBG`, `getAffineSymbolExpr`, `get`, `getAffineBinaryOpExpr` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 159-166
```cpp
  auto current = createSubApply(rewriter, op, subExpressions[0]);
  for (int64_t i = 1, e = subExpressions.size(); i < e; ++i) {
    Value tmp = createSubApply(rewriter, op, subExpressions[i]);
    current = AffineApplyOp::create(rewriter, op.getLoc(), binMap,
                                    ValueRange{current, tmp});
    LDBG() << "--reassociate into: " << current;
  }

```
- **EN**: Implements logic around `createSubApply`, `size`, `create`, `LDBG`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `createSubApply`, `size`, `create`, `LDBG` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 167-170
```cpp
  // 5. Replace original op.
  rewriter.replaceOp(op, current.getResult());
  return current;
}
```
- **EN**: Implements logic around `replaceOp`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugLog.h`, `llvm/Support/InterleavedRange.h`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助功能 (3), other MLIR dialect declarations / 其他 MLIR 方言声明 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
