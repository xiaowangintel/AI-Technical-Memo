# ForToWhile.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Transforms/ForToWhile.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Transforms SCF.ForOp's into SCF.WhileOp's.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Transforms`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ForToWhile.cpp - scf.for to scf.while loop conversion --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-12
```cpp
//
// Transforms SCF.ForOp's into SCF.WhileOp's.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 13-20
```cpp
#include "mlir/Dialect/SCF/Transforms/Passes.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Transforms.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`。

### Lines 21-25
```cpp
namespace mlir {
#define GEN_PASS_DEF_SCFFORTOWHILELOOP
#include "mlir/Dialect/SCF/Transforms/Passes.h.inc"
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/Transforms/Passes.h.inc`。

### Lines 26-30
```cpp
using namespace mlir;
using scf::ForOp;
using scf::WhileOp;

namespace {
```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 31-40
```cpp

struct ForLoopLoweringPattern : public OpRewritePattern<ForOp> {
  using OpRewritePattern<ForOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(ForOp forOp,
                                PatternRewriter &rewriter) const override {
    // Generate type signature for the loop-carried values. The induction
    // variable is placed first, followed by the forOp.iterArgs.
    SmallVector<Type> lcvTypes;
    SmallVector<Location> lcvLocs;
```
- **EN**: Introduces declarations for `ForLoopLoweringPattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ForLoopLoweringPattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 41-47
```cpp
    lcvTypes.push_back(forOp.getInductionVar().getType());
    lcvLocs.push_back(forOp.getInductionVar().getLoc());
    for (Value value : forOp.getInitArgs()) {
      lcvTypes.push_back(value.getType());
      lcvLocs.push_back(value.getLoc());
    }

```
- **EN**: Implements logic around `push_back`, `getInitArgs`.
- **CN**: 围绕 `push_back`, `getInitArgs` 实现具体逻辑。

### Lines 48-54
```cpp
    // Build scf.WhileOp
    SmallVector<Value> initArgs;
    initArgs.push_back(forOp.getLowerBound());
    llvm::append_range(initArgs, forOp.getInitArgs());
    auto whileOp = WhileOp::create(rewriter, forOp.getLoc(), lcvTypes, initArgs,
                                   forOp->getAttrs());

```
- **EN**: Implements logic around `push_back`, `append_range`, `create`, `getAttrs`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `append_range`, `create`, `getAttrs` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 55-64
```cpp
    // 'before' region contains the loop condition and forwarding of iteration
    // arguments to the 'after' region.
    auto *beforeBlock = rewriter.createBlock(
        &whileOp.getBefore(), whileOp.getBefore().begin(), lcvTypes, lcvLocs);
    rewriter.setInsertionPointToStart(whileOp.getBeforeBody());
    arith::CmpIPredicate predicate = forOp.getUnsignedCmp()
                                         ? arith::CmpIPredicate::ult
                                         : arith::CmpIPredicate::slt;
    auto cmpOp = arith::CmpIOp::create(rewriter, whileOp.getLoc(), predicate,
                                       beforeBlock->getArgument(0),
```
- **EN**: Implements logic around `createBlock`, `getBefore`, `setInsertionPointToStart`, `getUnsignedCmp`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createBlock`, `getBefore`, `setInsertionPointToStart`, `getUnsignedCmp`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 65-74
```cpp
                                       forOp.getUpperBound());
    scf::ConditionOp::create(rewriter, whileOp.getLoc(), cmpOp.getResult(),
                             beforeBlock->getArguments());

    // Inline for-loop body into an executeRegion operation in the "after"
    // region. The return type of the execRegionOp does not contain the
    // iv - yields in the source for-loop contain only iterArgs.
    auto *afterBlock = rewriter.createBlock(
        &whileOp.getAfter(), whileOp.getAfter().begin(), lcvTypes, lcvLocs);

```
- **EN**: Implements logic around `getUpperBound`, `create`, `getArguments`, `createBlock`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getUpperBound`, `create`, `getArguments`, `createBlock`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 75-80
```cpp
    // Add induction variable incrementation
    rewriter.setInsertionPointToEnd(afterBlock);
    auto ivIncOp =
        arith::AddIOp::create(rewriter, whileOp.getLoc(),
                              afterBlock->getArgument(0), forOp.getStep());

```
- **EN**: Implements logic around `setInsertionPointToEnd`, `create`, `getArgument`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPointToEnd`, `create`, `getArgument` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 81-86
```cpp
    // Rewrite uses of the for-loop block arguments to the new while-loop
    // "after" arguments
    for (const auto &barg : enumerate(forOp.getBody(0)->getArguments()))
      rewriter.replaceAllUsesWith(barg.value(),
                                  afterBlock->getArgument(barg.index()));

```
- **EN**: Implements logic around `enumerate`, `replaceAllUsesWith`, `getArgument`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `enumerate`, `replaceAllUsesWith`, `getArgument` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 87-96
```cpp
    // Inline for-loop body operations into 'after' region.
    for (auto &arg : llvm::make_early_inc_range(*forOp.getBody()))
      rewriter.moveOpBefore(&arg, afterBlock, afterBlock->end());

    // Add incremented IV to yield operations
    for (auto yieldOp : afterBlock->getOps<scf::YieldOp>()) {
      SmallVector<Value> yieldOperands = yieldOp.getOperands();
      yieldOperands.insert(yieldOperands.begin(), ivIncOp.getResult());
      rewriter.modifyOpInPlace(yieldOp,
                               [&]() { yieldOp->setOperands(yieldOperands); });
```
- **EN**: Implements logic around `make_early_inc_range`, `moveOpBefore`, `YieldOp>`, `getOperands`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `make_early_inc_range`, `moveOpBefore`, `YieldOp>`, `getOperands`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 97-106
```cpp
    }

    // We cannot do a direct replacement of the forOp since the while op returns
    // an extra value (the induction variable escapes the loop through being
    // carried in the set of iterargs). Instead, rewrite uses of the forOp
    // results.
    for (const auto &arg : llvm::enumerate(forOp.getResults()))
      rewriter.replaceAllUsesWith(arg.value(),
                                  whileOp.getResult(arg.index() + 1));

```
- **EN**: Implements logic around `enumerate`, `replaceAllUsesWith`, `getResult`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `enumerate`, `replaceAllUsesWith`, `getResult` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 107-111
```cpp
    rewriter.eraseOp(forOp);
    return success();
  }
};

```
- **EN**: Implements logic around `eraseOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `eraseOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 112-121
```cpp
struct ForToWhileLoop : public impl::SCFForToWhileLoopBase<ForToWhileLoop> {
  void runOnOperation() override {
    auto *parentOp = getOperation();
    MLIRContext *ctx = parentOp->getContext();
    RewritePatternSet patterns(ctx);
    patterns.add<ForLoopLoweringPattern>(ctx);
    (void)applyPatternsGreedily(parentOp, std::move(patterns));
  }
};
} // namespace
```
- **EN**: Introduces declarations for `ForToWhileLoop`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ForToWhileLoop` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 122-125
```cpp

std::unique_ptr<Pass> mlir::createForToWhileLoopPass() {
  return std::make_unique<ForToWhileLoop>();
}
```
- **EN**: Implements logic around `createForToWhileLoopPass`, `make_unique`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `createForToWhileLoopPass`, `make_unique` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SCF/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/SCF/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
