# InlineScalarOperands.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/InlineScalarOperands.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements patterns/pass to inline scalar operands into a generic operation. A scalar operand is an operand whose indexing map has a constant rhs.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- InlineScalarOperands.cpp - Pass to inline scalar operands =============//
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
// This file implements patterns/pass to inline scalar operands into a generic
// operation. A scalar operand is an operand whose indexing map has a constant
// rhs.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 14-23
```cpp

#include "mlir/Dialect/Linalg/Passes.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`。

### Lines 24-29
```cpp

namespace mlir {
#define GEN_PASS_DEF_LINALGINLINESCALAROPERANDSPASS
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 30-34
```cpp
using namespace mlir;
using namespace mlir::linalg;

namespace {
struct InlineScalarOperands : public OpRewritePattern<GenericOp> {
```
- **EN**: Introduces declarations for `mlir`, `mlir::linalg`, `InlineScalarOperands`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::linalg`, `InlineScalarOperands` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 35-40
```cpp
  using OpRewritePattern<GenericOp>::OpRewritePattern;
  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override {
    if (!genericOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `failure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 41-50
```cpp
    SmallVector<size_t> scalarOperands;
    SmallVector<AffineMap> newIndexingMaps;
    SmallVector<Value> newOperands;
    for (OpOperand *opOperand : genericOp.getDpsInputOperands()) {
      AffineMap map = genericOp.getMatchingIndexingMap(opOperand);
      if (genericOp.isDpsInput(opOperand) && map.isConstant()) {
        scalarOperands.emplace_back(opOperand->getOperandNumber());
      } else {
        newIndexingMaps.emplace_back(map);
        newOperands.emplace_back(opOperand->get());
```
- **EN**: Implements logic around `getDpsInputOperands`, `getMatchingIndexingMap`, `isDpsInput`, `emplace_back`.
- **CN**: 围绕 `getDpsInputOperands`, `getMatchingIndexingMap`, `isDpsInput`, `emplace_back` 实现具体逻辑。

### Lines 51-56
```cpp
      }
    }

    if (scalarOperands.empty())
      return failure();

```
- **EN**: Implements logic around `empty`, `failure`.
- **CN**: 围绕 `empty`, `failure` 实现具体逻辑。

### Lines 57-66
```cpp
    for (OpOperand &opOperand : genericOp.getDpsInitsMutable())
      newIndexingMaps.emplace_back(
          genericOp.getMatchingIndexingMap(&opOperand));

    Location loc = genericOp->getLoc();
    SmallVector<Value> outputOperands = genericOp.getOutputs();
    auto newOp = GenericOp::create(rewriter, loc, genericOp->getResultTypes(),
                                   newOperands, outputOperands, newIndexingMaps,
                                   genericOp.getIteratorTypesArray());
    rewriter.cloneRegionBefore(genericOp.getRegion(), newOp.getRegion(),
```
- **EN**: Implements logic around `getDpsInitsMutable`, `emplace_back`, `getMatchingIndexingMap`, `getLoc`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getDpsInitsMutable`, `emplace_back`, `getMatchingIndexingMap`, `getLoc`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 67-72
```cpp
                               newOp.getRegion().begin());

    Block *body = newOp.getBody();
    PatternRewriter::InsertionGuard guard(rewriter);
    rewriter.setInsertionPointToStart(body);

```
- **EN**: Implements logic around `getRegion`, `getBody`, `guard`, `setInsertionPointToStart`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getRegion`, `getBody`, `guard`, `setInsertionPointToStart` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 73-82
```cpp
    for (auto idx : llvm::reverse(scalarOperands)) {
      OpOperand *opOperand = genericOp.getDpsInputOperand(idx);
      AffineMap map = genericOp.getMatchingIndexingMap(opOperand);
      SmallVector<int64_t> indices = map.getConstantResults();
      SmallVector<Value> indicesValues;
      for (auto idx : indices)
        indicesValues.emplace_back(
            arith::ConstantIndexOp::create(rewriter, loc, idx));
      Value scalarValue = opOperand->get();
      if (isa<RankedTensorType>(scalarValue.getType())) {
```
- **EN**: Implements logic around `reverse`, `getDpsInputOperand`, `getMatchingIndexingMap`, `getConstantResults`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `reverse`, `getDpsInputOperand`, `getMatchingIndexingMap`, `getConstantResults`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 83-89
```cpp
        scalarValue = tensor::ExtractOp::create(rewriter, loc, scalarValue,
                                                indicesValues);
      }
      body->getArgument(idx).replaceAllUsesWith(scalarValue);
      body->eraseArgument(idx);
    }

```
- **EN**: Implements logic around `create`, `getArgument`, `eraseArgument`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getArgument`, `eraseArgument` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 90-95
```cpp
    rewriter.replaceOp(genericOp, newOp->getResults());
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 96-103
```cpp
/// Patterns that are used to inline constant operands into linalg generic
/// ops.
void mlir::linalg::populateInlineConstantOperandsPatterns(
    RewritePatternSet &patterns) {
  auto *context = patterns.getContext();
  patterns.add<InlineScalarOperands>(context);
}

```
- **EN**: Implements logic around `populateInlineConstantOperandsPatterns`, `getContext`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateInlineConstantOperandsPatterns`, `getContext`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 104-113
```cpp
namespace {
/// Pass that removes unit-extent dims within generic ops.
struct LinalgInlineScalarOperandsPass
    : public impl::LinalgInlineScalarOperandsPassBase<
          LinalgInlineScalarOperandsPass> {
  using impl::LinalgInlineScalarOperandsPassBase<
      LinalgInlineScalarOperandsPass>::LinalgInlineScalarOperandsPassBase;
  void runOnOperation() override {
    Operation *op = getOperation();
    MLIRContext &ctx = getContext();
```
- **EN**: Introduces declarations for `LinalgInlineScalarOperandsPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgInlineScalarOperandsPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 114-119
```cpp
    RewritePatternSet patterns(&ctx);
    populateInlineConstantOperandsPatterns(patterns);
    (void)applyPatternsGreedily(op, std::move(patterns));
  }
};
} // namespace
```
- **EN**: Implements logic around `patterns`, `populateInlineConstantOperandsPatterns`, `applyPatternsGreedily`.
- **CN**: 围绕 `patterns`, `populateInlineConstantOperandsPatterns`, `applyPatternsGreedily` 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/AffineExpr.h`, `mlir/IR/AffineMap.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/Linalg/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
