# UpliftWhileToFor.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/Transforms/UpliftWhileToFor.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Transforms SCF.WhileOp's into SCF.ForOp's.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SCF/Transforms`，围绕 SCF 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- UpliftWhileToFor.cpp - scf.while to scf.for loop uplifting ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-18
```cpp
//
// Transforms SCF.WhileOp's into SCF.ForOp's.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/IR/Dominance.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/IR/Dominance.h`。

### Lines 19-31
```cpp
using namespace mlir;

namespace {
struct UpliftWhileOp : public OpRewritePattern<scf::WhileOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(scf::WhileOp loop,
                                PatternRewriter &rewriter) const override {
    return upliftWhileToForLoop(rewriter, loop);
  }
};
} // namespace

```
- **EN**: Introduces declarations for `mlir`, `UpliftWhileOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `UpliftWhileOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 32-42
```cpp
FailureOr<scf::ForOp> mlir::scf::upliftWhileToForLoop(RewriterBase &rewriter,
                                                      scf::WhileOp loop) {
  Block *beforeBody = loop.getBeforeBody();
  if (!llvm::hasSingleElement(beforeBody->without_terminator()))
    return rewriter.notifyMatchFailure(loop, "Loop body must have single op");

  auto cmp = dyn_cast<arith::CmpIOp>(beforeBody->front());
  if (!cmp)
    return rewriter.notifyMatchFailure(loop,
                                       "Loop body must have single cmp op");

```
- **EN**: Implements logic around `upliftWhileToForLoop`, `getBeforeBody`, `hasSingleElement`, `notifyMatchFailure`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `upliftWhileToForLoop`, `getBeforeBody`, `hasSingleElement`, `notifyMatchFailure`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 43-56
```cpp
  scf::ConditionOp beforeTerm = loop.getConditionOp();
  if (!cmp->hasOneUse() || beforeTerm.getCondition() != cmp.getResult())
    return rewriter.notifyMatchFailure(loop, [&](Diagnostic &diag) {
      diag << "Expected single condition use: " << *cmp;
    });

  // If all 'before' arguments are forwarded but the order is different from
  // 'after' arguments, here is the mapping from the 'after' argument index to
  // the 'before' argument index.
  std::optional<SmallVector<unsigned>> argReorder;
  // All `before` block args must be directly forwarded to ConditionOp.
  // They will be converted to `scf.for` `iter_vars` except induction var.
  if (ValueRange(beforeBody->getArguments()) != beforeTerm.getArgs()) {
    auto getArgReordering =
```
- **EN**: Implements logic around `getConditionOp`, `hasOneUse`, `notifyMatchFailure`, `ValueRange`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getConditionOp`, `hasOneUse`, `notifyMatchFailure`, `ValueRange` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 57-70
```cpp
        [](Block *beforeBody,
           scf::ConditionOp cond) -> std::optional<SmallVector<unsigned>> {
      // Skip further checking if their sizes mismatch.
      if (beforeBody->getNumArguments() != cond.getArgs().size())
        return std::nullopt;
      // Bitset on which 'before' argument is forwarded.
      llvm::SmallBitVector forwarded(beforeBody->getNumArguments(), false);
      // The forwarding order of 'before' arguments.
      SmallVector<unsigned> order;
      for (Value a : cond.getArgs()) {
        BlockArgument arg = dyn_cast<BlockArgument>(a);
        // Skip if 'arg' is not a 'before' argument.
        if (!arg || arg.getOwner() != beforeBody)
          return std::nullopt;
```
- **EN**: Implements logic around `getNumArguments`, `forwarded`, `getArgs`, `getOwner`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumArguments`, `forwarded`, `getArgs`, `getOwner` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 71-84
```cpp
        unsigned idx = arg.getArgNumber();
        // Skip if 'arg' is already forwarded in another place.
        if (forwarded[idx])
          return std::nullopt;
        // Record the presence of 'arg' and its order.
        forwarded[idx] = true;
        order.push_back(idx);
      }
      // Skip if not all 'before' arguments are forwarded.
      if (!forwarded.all())
        return std::nullopt;
      return order;
    };
    // Check if 'before' arguments are all forwarded but just reordered.
```
- **EN**: Implements logic around `getArgNumber`, `push_back`, `all`.
- **CN**: 围绕 `getArgNumber`, `push_back`, `all` 实现具体逻辑。

### Lines 85-96
```cpp
    argReorder = getArgReordering(beforeBody, beforeTerm);
    if (!argReorder)
      return rewriter.notifyMatchFailure(loop, "Invalid args order");
  }

  using Pred = arith::CmpIPredicate;
  Pred predicate = cmp.getPredicate();
  if (predicate != Pred::slt && predicate != Pred::sgt)
    return rewriter.notifyMatchFailure(loop, [&](Diagnostic &diag) {
      diag << "Expected 'slt' or 'sgt' predicate: " << *cmp;
    });

```
- **EN**: Implements logic around `getArgReordering`, `notifyMatchFailure`, `getPredicate`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getArgReordering`, `notifyMatchFailure`, `getPredicate` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 97-108
```cpp
  BlockArgument inductionVar;
  Value ub;
  DominanceInfo dom;

  // Check if cmp has a suitable form. One of the arguments must be a `before`
  // block arg, other must be defined outside `scf.while` and will be treated
  // as upper bound.
  for (bool reverse : {false, true}) {
    auto expectedPred = reverse ? Pred::sgt : Pred::slt;
    if (cmp.getPredicate() != expectedPred)
      continue;

```
- **EN**: Implements logic around `getPredicate`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getPredicate` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 109-115
```cpp
    auto arg1 = reverse ? cmp.getRhs() : cmp.getLhs();
    auto arg2 = reverse ? cmp.getLhs() : cmp.getRhs();

    auto blockArg = dyn_cast<BlockArgument>(arg1);
    if (!blockArg || blockArg.getOwner() != beforeBody)
      continue;

```
- **EN**: Implements logic around `getRhs`, `getLhs`, `getOwner`.
- **CN**: 围绕 `getRhs`, `getLhs`, `getOwner` 实现具体逻辑。

### Lines 116-123
```cpp
    if (!dom.properlyDominates(arg2, loop))
      continue;

    inductionVar = blockArg;
    ub = arg2;
    break;
  }

```
- **EN**: Implements logic around `properlyDominates`.
- **CN**: 围绕 `properlyDominates` 实现具体逻辑。

### Lines 124-135
```cpp
  if (!inductionVar)
    return rewriter.notifyMatchFailure(loop, [&](Diagnostic &diag) {
      diag << "Unrecognized cmp form: " << *cmp;
    });

  // inductionVar must have 2 uses: one is in `cmp` and other is `condition`
  // arg.
  if (!llvm::hasNItems(inductionVar.getUses(), 2))
    return rewriter.notifyMatchFailure(loop, [&](Diagnostic &diag) {
      diag << "Unrecognized induction var: " << inductionVar;
    });

```
- **EN**: Implements logic around `notifyMatchFailure`, `hasNItems`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `notifyMatchFailure`, `hasNItems` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 136-149
```cpp
  Block *afterBody = loop.getAfterBody();
  scf::YieldOp afterTerm = loop.getYieldOp();
  unsigned argNumber = inductionVar.getArgNumber();
  Value afterTermIndArg = afterTerm.getResults()[argNumber];

  auto findAfterArgNo = [](ArrayRef<unsigned> indices, unsigned beforeArgNo) {
    return std::distance(indices.begin(),
                         llvm::find_if(indices, [beforeArgNo](unsigned n) {
                           return n == beforeArgNo;
                         }));
  };
  Value inductionVarAfter = afterBody->getArgument(
      argReorder ? findAfterArgNo(*argReorder, argNumber) : argNumber);

```
- **EN**: Implements logic around `getAfterBody`, `getYieldOp`, `getArgNumber`, `getResults`, and 4 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getAfterBody`, `getYieldOp`, `getArgNumber`, `getResults`, and 4 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 150-157
```cpp
  // Find suitable `addi` op inside `after` block, one of the args must be an
  // Induction var passed from `before` block and second arg must be defined
  // outside of the loop and will be considered step value.
  // TODO: Add `subi` support?
  auto addOp = afterTermIndArg.getDefiningOp<arith::AddIOp>();
  if (!addOp)
    return rewriter.notifyMatchFailure(loop, "Didn't found suitable 'addi' op");

```
- **EN**: Implements logic around `AddIOp>`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `AddIOp>`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 158-164
```cpp
  Value step;
  if (addOp.getLhs() == inductionVarAfter) {
    step = addOp.getRhs();
  } else if (addOp.getRhs() == inductionVarAfter) {
    step = addOp.getLhs();
  }

```
- **EN**: Implements logic around `getLhs`, `getRhs`.
- **CN**: 围绕 `getLhs`, `getRhs` 实现具体逻辑。

### Lines 165-173
```cpp
  if (!step || !dom.properlyDominates(step, loop))
    return rewriter.notifyMatchFailure(loop, "Invalid 'addi' form");

  Value lb = loop.getInits()[argNumber];

  assert(lb.getType().isIntOrIndex());
  assert(lb.getType() == ub.getType());
  assert(lb.getType() == step.getType());

```
- **EN**: Implements logic around `properlyDominates`, `notifyMatchFailure`, `getInits`, `assert`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `properlyDominates`, `notifyMatchFailure`, `getInits`, `assert` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 174-181
```cpp
  SmallVector<Value> newArgs;

  // Populate inits for new `scf.for`, skip induction var.
  newArgs.reserve(loop.getInits().size());
  for (auto &&[i, init] : llvm::enumerate(loop.getInits())) {
    if (i == argNumber)
      continue;

```
- **EN**: Implements logic around `reserve`, `enumerate`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `reserve`, `enumerate` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 182-193
```cpp
    newArgs.emplace_back(init);
  }

  Location loc = loop.getLoc();

  // With `builder == nullptr`, ForOp::build will try to insert terminator at
  // the end of newly created block and we don't want it. Provide empty
  // dummy builder instead.
  auto emptyBuilder = [](OpBuilder &, Location, Value, ValueRange) {};
  auto newLoop =
      scf::ForOp::create(rewriter, loc, lb, ub, step, newArgs, emptyBuilder);

```
- **EN**: Implements logic around `emplace_back`, `getLoc`, `create`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `emplace_back`, `getLoc`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 194-207
```cpp
  Block *newBody = newLoop.getBody();

  // Populate block args for `scf.for` body, move induction var to the front.
  newArgs.clear();
  ValueRange newBodyArgs = newBody->getArguments();
  for (auto i : llvm::seq<size_t>(0, newBodyArgs.size())) {
    if (i < argNumber) {
      newArgs.emplace_back(newBodyArgs[i + 1]);
    } else if (i == argNumber) {
      newArgs.emplace_back(newBodyArgs.front());
    } else {
      newArgs.emplace_back(newBodyArgs[i]);
    }
  }
```
- **EN**: Implements logic around `getBody`, `clear`, `getArguments`, `seq`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getBody`, `clear`, `getArguments`, `seq`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 208-216
```cpp
  if (argReorder) {
    // Reorder arguments following the 'after' argument order from the original
    // 'while' loop.
    SmallVector<Value> args;
    for (unsigned order : *argReorder)
      args.push_back(newArgs[order]);
    newArgs = args;
  }

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 217-227
```cpp
  rewriter.inlineBlockBefore(loop.getAfterBody(), newBody, newBody->end(),
                             newArgs);

  auto term = cast<scf::YieldOp>(newBody->getTerminator());

  // Populate new yield args, skipping the induction var.
  newArgs.clear();
  for (auto &&[i, arg] : llvm::enumerate(term.getResults())) {
    if (i == argNumber)
      continue;

```
- **EN**: Implements logic around `inlineBlockBefore`, `YieldOp>`, `clear`, `enumerate`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `inlineBlockBefore`, `YieldOp>`, `clear`, `enumerate` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 228-234
```cpp
    newArgs.emplace_back(arg);
  }

  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPoint(term);
  rewriter.replaceOpWithNewOp<scf::YieldOp>(term, newArgs);

```
- **EN**: Implements logic around `emplace_back`, `g`, `setInsertionPoint`, `YieldOp>`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `emplace_back`, `g`, `setInsertionPoint`, `YieldOp>` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 235-243
```cpp
  // Compute induction var value after loop execution.
  rewriter.setInsertionPointAfter(newLoop);
  Value one;
  if (isa<IndexType>(step.getType())) {
    one = arith::ConstantIndexOp::create(rewriter, loc, 1);
  } else {
    one = arith::ConstantIntOp::create(rewriter, loc, step.getType(), 1);
  }

```
- **EN**: Implements logic around `setInsertionPointAfter`, `getType`, `create`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPointAfter`, `getType`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 244-251
```cpp
  Value stepDec = arith::SubIOp::create(rewriter, loc, step, one);
  Value len = arith::SubIOp::create(rewriter, loc, ub, lb);
  len = arith::AddIOp::create(rewriter, loc, len, stepDec);
  len = arith::DivSIOp::create(rewriter, loc, len, step);
  len = arith::SubIOp::create(rewriter, loc, len, one);
  Value res = arith::MulIOp::create(rewriter, loc, len, step);
  res = arith::AddIOp::create(rewriter, loc, lb, res);

```
- **EN**: Implements logic around `create`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 252-265
```cpp
  // Reconstruct `scf.while` results, inserting final induction var value
  // into proper place.
  newArgs.clear();
  llvm::append_range(newArgs, newLoop.getResults());
  newArgs.insert(newArgs.begin() + argNumber, res);
  if (argReorder) {
    // Reorder arguments following the 'after' argument order from the original
    // 'while' loop.
    SmallVector<Value> results;
    for (unsigned order : *argReorder)
      results.push_back(newArgs[order]);
    newArgs = results;
  }
  rewriter.replaceOp(loop, newArgs);
```
- **EN**: Implements logic around `clear`, `append_range`, `insert`, `push_back`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `clear`, `append_range`, `insert`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 266-271
```cpp
  return newLoop;
}

void mlir::scf::populateUpliftWhileToForPatterns(RewritePatternSet &patterns) {
  patterns.add<UpliftWhileOp>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateUpliftWhileToForPatterns`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateUpliftWhileToForPatterns`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/SCF/Transforms/Patterns.h`, `mlir/IR/Dominance.h`, `mlir/IR/PatternMatch.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2)
