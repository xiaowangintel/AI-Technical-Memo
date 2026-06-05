# AsyncRuntimeRefCounting.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Async/Transforms/AsyncRuntimeRefCounting.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements automatic reference counting for Async runtime operations and types.
  - **CN**: 实现 Async 方言与异步执行原语 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- AsyncRuntimeRefCounting.cpp - Async Runtime Ref Counting -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements automatic reference counting for Async runtime
// operations and types.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 13-22
```cpp

#include "mlir/Dialect/Async/Passes.h"

#include "mlir/Analysis/Liveness.h"
#include "mlir/Dialect/Async/IR/Async.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "llvm/ADT/SmallSet.h"

namespace mlir {
```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 23-32
```cpp
#define GEN_PASS_DEF_ASYNCRUNTIMEREFCOUNTINGPASS
#define GEN_PASS_DEF_ASYNCRUNTIMEPOLICYBASEDREFCOUNTINGPASS
#include "mlir/Dialect/Async/Passes.h.inc"
} // namespace mlir

#define DEBUG_TYPE "async-runtime-ref-counting"

using namespace mlir;
using namespace mlir::async;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 33-43
```cpp
//===----------------------------------------------------------------------===//
// Utility functions shared by reference counting passes.
//===----------------------------------------------------------------------===//

// Drop the reference count immediately if the value has no uses.
static LogicalResult dropRefIfNoUses(Value value, unsigned count = 1) {
  if (!value.getUses().empty())
    return failure();

  OpBuilder b(value.getContext());

```
- **EN**: Implements logic around `dropRefIfNoUses`, `getUses`, `failure`, `b`.
- **CN**: 围绕 `dropRefIfNoUses`, `getUses`, `failure`, `b` 实现具体逻辑。

### Lines 44-54
```cpp
  // Set insertion point after the operation producing a value, or at the
  // beginning of the block if the value defined by the block argument.
  if (Operation *op = value.getDefiningOp())
    b.setInsertionPointAfter(op);
  else
    b.setInsertionPointToStart(value.getParentBlock());

  RuntimeDropRefOp::create(b, value.getLoc(), value, b.getI64IntegerAttr(1));
  return success();
}

```
- **EN**: Implements logic around `getDefiningOp`, `setInsertionPointAfter`, `setInsertionPointToStart`, `create`, and 1 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getDefiningOp`, `setInsertionPointAfter`, `setInsertionPointToStart`, `create`, and 1 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 55-65
```cpp
// Calls `addRefCounting` for every reference counted value defined by the
// operation `op` (block arguments and values defined in nested regions).
static LogicalResult walkReferenceCountedValues(
    Operation *op, llvm::function_ref<LogicalResult(Value)> addRefCounting) {
  // Check that we do not have high level async operations in the IR because
  // otherwise reference counting will produce incorrect results after high
  // level async operations will be lowered to `async.runtime`
  WalkResult checkNoAsyncWalk = op->walk([&](Operation *op) -> WalkResult {
    if (!isa<ExecuteOp, AwaitOp, AwaitAllOp, YieldOp>(op))
      return WalkResult::advance();

```
- **EN**: Implements logic around `walkReferenceCountedValues`, `function_ref`, `walk`, `YieldOp>`, and 1 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `walkReferenceCountedValues`, `function_ref`, `walk`, `YieldOp>`, and 1 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 66-79
```cpp
    return op->emitError()
           << "async operations must be lowered to async runtime operations";
  });

  if (checkNoAsyncWalk.wasInterrupted())
    return failure();

  // Add reference counting to block arguments.
  WalkResult blockWalk = op->walk([&](Block *block) -> WalkResult {
    for (BlockArgument arg : block->getArguments())
      if (isRefCounted(arg.getType()))
        if (failed(addRefCounting(arg)))
          return WalkResult::interrupt();

```
- **EN**: Implements logic around `emitError`, `wasInterrupted`, `failure`, `walk`, and 4 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `emitError`, `wasInterrupted`, `failure`, `walk`, and 4 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 80-92
```cpp
    return WalkResult::advance();
  });

  if (blockWalk.wasInterrupted())
    return failure();

  // Add reference counting to operation results.
  WalkResult opWalk = op->walk([&](Operation *op) -> WalkResult {
    for (unsigned i = 0; i < op->getNumResults(); ++i)
      if (isRefCounted(op->getResultTypes()[i]))
        if (failed(addRefCounting(op->getResult(i))))
          return WalkResult::interrupt();

```
- **EN**: Implements logic around `advance`, `wasInterrupted`, `failure`, `walk`, and 4 more symbols.
- **CN**: 围绕 `advance`, `wasInterrupted`, `failure`, `walk`, and 4 more symbols 实现具体逻辑。

### Lines 93-102
```cpp
    return WalkResult::advance();
  });

  if (opWalk.wasInterrupted())
    return failure();

  return success();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `advance`, `wasInterrupted`, `failure`, `success`.
- **CN**: 围绕 `advance`, `wasInterrupted`, `failure`, `success` 实现具体逻辑。

### Lines 103-114
```cpp
// Automatic reference counting based on the liveness analysis.
//===----------------------------------------------------------------------===//

namespace {

class AsyncRuntimeRefCountingPass
    : public impl::AsyncRuntimeRefCountingPassBase<
          AsyncRuntimeRefCountingPass> {
public:
  AsyncRuntimeRefCountingPass() = default;
  void runOnOperation() override;

```
- **EN**: Introduces declarations for `AsyncRuntimeRefCountingPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncRuntimeRefCountingPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 115-124
```cpp
private:
  /// Adds an automatic reference counting to the `value`.
  ///
  /// All values (token, group or value) are semantically created with a
  /// reference count of +1 and it is the responsibility of the async value user
  /// to place the `add_ref` and `drop_ref` operations to ensure that the value
  /// is destroyed after the last use.
  ///
  /// The function returns failure if it can't deduce the locations where
  /// to place the reference counting operations.
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 125-134
```cpp
  ///
  /// Async values "semantically created" when:
  ///   1. Operation returns async result (e.g. `async.runtime.create`)
  ///   2. Async value passed in as a block argument (or function argument,
  ///      because function arguments are just entry block arguments)
  ///
  /// Passing async value as a function argument (or block argument) does not
  /// really mean that a new async value is created, it only means that the
  /// caller of a function transfered ownership of `+1` reference to the callee.
  /// It is convenient to think that from the callee perspective async value was
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 135-144
```cpp
  /// "created" with `+1` reference by the block argument.
  ///
  /// Automatic reference counting algorithm outline:
  ///
  /// #1 Insert `drop_ref` operations after last use of the `value`.
  /// #2 Insert `add_ref` operations before functions calls with reference
  ///    counted `value` operand (newly created `+1` reference will be
  ///    transferred to the callee).
  /// #3 Verify that divergent control flow does not lead to leaked reference
  ///    counted objects.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 145-154
```cpp
  ///
  /// Async runtime reference counting optimization pass will optimize away
  /// some of the redundant `add_ref` and `drop_ref` operations inserted by this
  /// strategy (see `async-runtime-ref-counting-opt`).
  LogicalResult addAutomaticRefCounting(Value value);

  /// (#1) Adds the `drop_ref` operation after the last use of the `value`
  /// relying on the liveness analysis.
  ///
  /// If the `value` is in the block `liveIn` set and it is not in the block
```
- **EN**: Implements logic around `addAutomaticRefCounting`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `addAutomaticRefCounting` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 155-164
```cpp
  /// `liveOut` set, it means that it "dies" in the block. We find the last
  /// use of the value in such block and:
  ///
  ///   1. If the last user is a `ReturnLike` operation we do nothing, because
  ///      it forwards the ownership to the caller.
  ///   2. Otherwise we add a `drop_ref` operation immediately after the last
  ///      use.
  LogicalResult addDropRefAfterLastUse(Value value);

  /// (#2) Adds the `add_ref` operation before the function call taking `value`
```
- **EN**: Implements logic around `addDropRefAfterLastUse`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `addDropRefAfterLastUse` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 165-174
```cpp
  /// operand to ensure that the value passed to the function entry block
  /// has a `+1` reference count.
  LogicalResult addAddRefBeforeFunctionCall(Value value);

  /// (#3) Adds the `drop_ref` operation to account for successor blocks with
  /// divergent `liveIn` property: `value` is not in the `liveIn` set of all
  /// successor blocks.
  ///
  /// Example:
  ///
```
- **EN**: Implements logic around `addAddRefBeforeFunctionCall`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `addAddRefBeforeFunctionCall` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 175-184
```cpp
  ///   ^entry:
  ///     %token = async.runtime.create : !async.token
  ///     cf.cond_br %cond, ^bb1, ^bb2
  ///   ^bb1:
  ///     async.runtime.await %token
  ///     async.runtime.drop_ref %token
  ///     cf.br ^bb2
  ///   ^bb2:
  ///     return
  ///
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 185-194
```cpp
  /// In this example ^bb2 does not have `value` in the `liveIn` set, so we have
  /// to branch into a special "reference counting block" from the ^entry that
  /// will have a `drop_ref` operation, and then branch into the ^bb2.
  ///
  /// After transformation:
  ///
  ///   ^entry:
  ///     %token = async.runtime.create : !async.token
  ///     cf.cond_br %cond, ^bb1, ^reference_counting
  ///   ^bb1:
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 195-204
```cpp
  ///     async.runtime.await %token
  ///     async.runtime.drop_ref %token
  ///     cf.br ^bb2
  ///   ^reference_counting:
  ///     async.runtime.drop_ref %token
  ///     cf.br ^bb2
  ///   ^bb2:
  ///     return
  ///
  /// An exception to this rule are blocks with `async.coro.suspend` terminator,
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 205-214
```cpp
  /// because in Async to LLVM lowering it is guaranteed that the control flow
  /// will jump into the resume block, and then follow into the cleanup and
  /// suspend blocks.
  ///
  /// Example:
  ///
  ///  ^entry(%value: !async.value<f32>):
  ///     async.runtime.await_and_resume %value, %hdl : !async.value<f32>
  ///     async.coro.suspend %ret, ^suspend, ^resume, ^cleanup
  ///   ^resume:
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 215-224
```cpp
  ///     %0 = async.runtime.load %value
  ///     cf.br ^cleanup
  ///   ^cleanup:
  ///     ...
  ///   ^suspend:
  ///     ...
  ///
  /// Although cleanup and suspend blocks do not have the `value` in the
  /// `liveIn` set, it is guaranteed that execution will eventually continue in
  /// the resume block (we never explicitly destroy coroutines).
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 225-236
```cpp
  LogicalResult addDropRefInDivergentLivenessSuccessor(Value value);
};

} // namespace

LogicalResult AsyncRuntimeRefCountingPass::addDropRefAfterLastUse(Value value) {
  OpBuilder builder(value.getContext());
  Location loc = value.getLoc();

  // Use liveness analysis to find the placement of `drop_ref`operation.
  auto &liveness = getAnalysis<Liveness>();

```
- **EN**: Implements logic around `addDropRefInDivergentLivenessSuccessor`, `addDropRefAfterLastUse`, `builder`, `getLoc`, and 1 more symbols.
- **CN**: 围绕 `addDropRefInDivergentLivenessSuccessor`, `addDropRefAfterLastUse`, `builder`, `getLoc`, and 1 more symbols 实现具体逻辑。

### Lines 237-249
```cpp
  // We analyse only the blocks of the region that defines the `value`, and do
  // not check nested blocks attached to operations.
  //
  // By analyzing only the `definingRegion` CFG we potentially loose an
  // opportunity to drop the reference count earlier and can extend the lifetime
  // of reference counted value longer then it is really required.
  //
  // We also assume that all nested regions finish their execution before the
  // completion of the owner operation. The only exception to this rule is
  // `async.execute` operation, and we verify that they are lowered to the
  // `async.runtime` operations before adding automatic reference counting.
  Region *definingRegion = value.getParentRegion();

```
- **EN**: Implements logic around `getParentRegion`; this block implements verifier, folding, parsing, or printing hooks; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getParentRegion` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理 MLIR region、block 或控制流边。

### Lines 250-259
```cpp
  // Last users of the `value` inside all blocks where the value dies.
  llvm::SmallPtrSet<Operation *, 4> lastUsers;

  // Find blocks in the `definingRegion` that have users of the `value` (if
  // there are multiple users in the block, which one will be selected is
  // undefined). User operation might be not the actual user of the value, but
  // the operation in the block that has a "real user" in one of the attached
  // regions.
  llvm::DenseMap<Block *, Operation *> usersInTheBlocks;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 260-279
```cpp
  for (Operation *user : value.getUsers()) {
    Block *userBlock = user->getBlock();
    Block *ancestor = definingRegion->findAncestorBlockInRegion(*userBlock);
    usersInTheBlocks[ancestor] = ancestor->findAncestorOpInBlock(*user);
    assert(ancestor && "ancestor block must be not null");
    assert(usersInTheBlocks[ancestor] && "ancestor op must be not null");
  }

  // Find blocks where the `value` dies: the value is in `liveIn` set and not
  // in the `liveOut` set. We place `drop_ref` immediately after the last use
  // of the `value` in such regions (after handling few special cases).
  //
  // We do not traverse all the blocks in the `definingRegion`, because the
  // `value` can be in the live in set only if it has users in the block, or it
  // is defined in the block.
  //
  // Values with zero users (only definition) handled explicitly above.
  for (auto &blockAndUser : usersInTheBlocks) {
    Block *block = blockAndUser.getFirst();
    Operation *userInTheBlock = blockAndUser.getSecond();
```
- **EN**: Implements logic around `getUsers`, `getBlock`, `findAncestorBlockInRegion`, `findAncestorOpInBlock`, and 3 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getUsers`, `getBlock`, `findAncestorBlockInRegion`, `findAncestorOpInBlock`, and 3 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 280-290
```cpp

    const LivenessBlockInfo *blockLiveness = liveness.getLiveness(block);

    // Value must be in the live input set or defined in the block.
    assert(blockLiveness->isLiveIn(value) ||
           blockLiveness->getBlock() == value.getParentBlock());

    // If value is in the live out set, it means it doesn't "die" in the block.
    if (blockLiveness->isLiveOut(value))
      continue;

```
- **EN**: Implements logic around `getLiveness`, `assert`, `getBlock`, `isLiveOut`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getLiveness`, `assert`, `getBlock`, `isLiveOut` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 291-304
```cpp
    // At this point we proved that `value` dies in the `block`. Find the last
    // use of the `value` inside the `block`, this is where it "dies".
    Operation *lastUser = blockLiveness->getEndOperation(value, userInTheBlock);
    assert(lastUsers.count(lastUser) == 0 && "last users must be unique");
    lastUsers.insert(lastUser);
  }

  // Process all the last users of the `value` inside each block where the value
  // dies.
  for (Operation *lastUser : lastUsers) {
    // Return like operations forward reference count.
    if (lastUser->hasTrait<OpTrait::ReturnLike>())
      continue;

```
- **EN**: Implements logic around `getEndOperation`, `assert`, `insert`, `ReturnLike>`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getEndOperation`, `assert`, `insert`, `ReturnLike>` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 305-314
```cpp
    // We can't currently handle other types of terminators.
    if (lastUser->hasTrait<OpTrait::IsTerminator>())
      return lastUser->emitError() << "async reference counting can't handle "
                                      "terminators that are not ReturnLike";

    // Add a drop_ref immediately after the last user.
    builder.setInsertionPointAfter(lastUser);
    RuntimeDropRefOp::create(builder, loc, value, builder.getI64IntegerAttr(1));
  }

```
- **EN**: Implements logic around `IsTerminator>`, `emitError`, `setInsertionPointAfter`, `create`.
- **CN**: 围绕 `IsTerminator>`, `emitError`, `setInsertionPointAfter`, `create` 实现具体逻辑。

### Lines 315-326
```cpp
  return success();
}

LogicalResult
AsyncRuntimeRefCountingPass::addAddRefBeforeFunctionCall(Value value) {
  OpBuilder builder(value.getContext());
  Location loc = value.getLoc();

  for (Operation *user : value.getUsers()) {
    if (!isa<func::CallOp>(user))
      continue;

```
- **EN**: Implements logic around `success`, `addAddRefBeforeFunctionCall`, `builder`, `getLoc`, and 2 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `success`, `addAddRefBeforeFunctionCall`, `builder`, `getLoc`, and 2 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 327-340
```cpp
    // Add a reference before the function call to pass the value at `+1`
    // reference to the function entry block.
    builder.setInsertionPoint(user);
    RuntimeAddRefOp::create(builder, loc, value, builder.getI64IntegerAttr(1));
  }

  return success();
}

LogicalResult
AsyncRuntimeRefCountingPass::addDropRefInDivergentLivenessSuccessor(
    Value value) {
  using BlockSet = llvm::SmallPtrSet<Block *, 4>;

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `success`, `addDropRefInDivergentLivenessSuccessor`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `setInsertionPoint`, `create`, `success`, `addDropRefInDivergentLivenessSuccessor` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 341-353
```cpp
  OpBuilder builder(value.getContext());

  // If a block has successors with different `liveIn` property of the `value`,
  // record block successors that do not thave the `value` in the `liveIn` set.
  llvm::SmallDenseMap<Block *, BlockSet> divergentLivenessBlocks;

  // Use liveness analysis to find the placement of `drop_ref`operation.
  auto &liveness = getAnalysis<Liveness>();

  // Because we only add `drop_ref` operations to the region that defines the
  // `value` we can only process CFG for the same region.
  Region *definingRegion = value.getParentRegion();

```
- **EN**: Implements logic around `builder`, `getAnalysis`, `getParentRegion`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `builder`, `getAnalysis`, `getParentRegion` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 354-364
```cpp
  // Collect blocks with successors with mismatching `liveIn` sets.
  for (Block &block : definingRegion->getBlocks()) {
    const LivenessBlockInfo *blockLiveness = liveness.getLiveness(&block);

    // Skip the block if value is not in the `liveOut` set.
    if (!blockLiveness || !blockLiveness->isLiveOut(value))
      continue;

    BlockSet liveInSuccessors;   // `value` is in `liveIn` set
    BlockSet noLiveInSuccessors; // `value` is not in the `liveIn` set

```
- **EN**: Implements logic around `getBlocks`, `getLiveness`, `isLiveOut`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBlocks`, `getLiveness`, `isLiveOut` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 365-378
```cpp
    // Collect successors that do not have `value` in the `liveIn` set.
    for (Block *successor : block.getSuccessors()) {
      const LivenessBlockInfo *succLiveness = liveness.getLiveness(successor);
      if (succLiveness && succLiveness->isLiveIn(value))
        liveInSuccessors.insert(successor);
      else
        noLiveInSuccessors.insert(successor);
    }

    // Block has successors with different `liveIn` property of the `value`.
    if (!liveInSuccessors.empty() && !noLiveInSuccessors.empty())
      divergentLivenessBlocks.try_emplace(&block, noLiveInSuccessors);
  }

```
- **EN**: Implements logic around `getSuccessors`, `getLiveness`, `isLiveIn`, `insert`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getSuccessors`, `getLiveness`, `isLiveIn`, `insert`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 379-390
```cpp
  // Try to insert `dropRef` operations to handle blocks with divergent liveness
  // in successors blocks.
  for (auto kv : divergentLivenessBlocks) {
    Block *block = kv.getFirst();
    BlockSet &successors = kv.getSecond();

    // Coroutine suspension is a special case terminator for wich we do not
    // need to create additional reference counting (see details above).
    Operation *terminator = block->getTerminator();
    if (isa<CoroSuspendOp>(terminator))
      continue;

```
- **EN**: Implements logic around `getFirst`, `getSecond`, `getTerminator`, `isa`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getFirst`, `getSecond`, `getTerminator`, `isa` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 391-407
```cpp
    // We only support successor blocks with empty block argument list.
    auto hasArgs = [](Block *block) { return !block->getArguments().empty(); };
    if (llvm::any_of(successors, hasArgs))
      return terminator->emitOpError()
             << "successor have different `liveIn` property of the reference "
                "counted value";

    // Make sure that `dropRef` operation is called when branched into the
    // successor block without `value` in the `liveIn` set.
    for (Block *successor : successors) {
      // If successor has a unique predecessor, it is safe to create `dropRef`
      // operations directly in the successor block.
      //
      // Otherwise we need to create a special block for reference counting
      // operations, and branch from it to the original successor block.
      Block *refCountingBlock = nullptr;

```
- **EN**: Implements logic around `getArguments`, `any_of`, `emitOpError`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getArguments`, `any_of`, `emitOpError` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 408-420
```cpp
      if (successor->getUniquePredecessor() == block) {
        refCountingBlock = successor;
      } else {
        refCountingBlock = &successor->getParent()->emplaceBlock();
        refCountingBlock->moveBefore(successor);
        OpBuilder builder = OpBuilder::atBlockEnd(refCountingBlock);
        cf::BranchOp::create(builder, value.getLoc(), successor);
      }

      OpBuilder builder = OpBuilder::atBlockBegin(refCountingBlock);
      RuntimeDropRefOp::create(builder, value.getLoc(), value,
                               builder.getI64IntegerAttr(1));

```
- **EN**: Implements logic around `getUniquePredecessor`, `getParent`, `moveBefore`, `atBlockEnd`, and 3 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getUniquePredecessor`, `getParent`, `moveBefore`, `atBlockEnd`, and 3 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 421-431
```cpp
      // No need to update the terminator operation.
      if (successor == refCountingBlock)
        continue;

      // Update terminator `successor` block to `refCountingBlock`.
      for (const auto &pair : llvm::enumerate(terminator->getSuccessors()))
        if (pair.value() == successor)
          terminator->setSuccessor(refCountingBlock, pair.index());
    }
  }

```
- **EN**: Implements logic around `enumerate`, `value`, `setSuccessor`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `enumerate`, `value`, `setSuccessor` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 432-444
```cpp
  return success();
}

LogicalResult
AsyncRuntimeRefCountingPass::addAutomaticRefCounting(Value value) {
  // Short-circuit reference counting for values without uses.
  if (succeeded(dropRefIfNoUses(value)))
    return success();

  // Add `drop_ref` operations based on the liveness analysis.
  if (failed(addDropRefAfterLastUse(value)))
    return failure();

```
- **EN**: Implements logic around `success`, `addAutomaticRefCounting`, `succeeded`, `failed`, and 1 more symbols.
- **CN**: 围绕 `success`, `addAutomaticRefCounting`, `succeeded`, `failed`, and 1 more symbols 实现具体逻辑。

### Lines 445-455
```cpp
  // Add `add_ref` operations before function calls.
  if (failed(addAddRefBeforeFunctionCall(value)))
    return failure();

  // Add `drop_ref` operations to successors with divergent `value` liveness.
  if (failed(addDropRefInDivergentLivenessSuccessor(value)))
    return failure();

  return success();
}

```
- **EN**: Implements logic around `failed`, `failure`, `success`.
- **CN**: 围绕 `failed`, `failure`, `success` 实现具体逻辑。

### Lines 456-465
```cpp
void AsyncRuntimeRefCountingPass::runOnOperation() {
  auto functor = [&](Value value) { return addAutomaticRefCounting(value); };
  if (failed(walkReferenceCountedValues(getOperation(), functor)))
    signalPassFailure();
}

//===----------------------------------------------------------------------===//
// Reference counting based on the user defined policy.
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `runOnOperation`, `addAutomaticRefCounting`, `failed`, `signalPassFailure`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `addAutomaticRefCounting`, `failed`, `signalPassFailure` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 466-475
```cpp
namespace {

class AsyncRuntimePolicyBasedRefCountingPass
    : public impl::AsyncRuntimePolicyBasedRefCountingPassBase<
          AsyncRuntimePolicyBasedRefCountingPass> {
public:
  AsyncRuntimePolicyBasedRefCountingPass() { initializeDefaultPolicy(); }

  void runOnOperation() override;

```
- **EN**: Introduces declarations for `AsyncRuntimePolicyBasedRefCountingPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncRuntimePolicyBasedRefCountingPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 476-485
```cpp
private:
  // Adds a reference counting operations for all uses of the `value` according
  // to the reference counting policy.
  LogicalResult addRefCounting(Value value);

  void initializeDefaultPolicy();

  llvm::SmallVector<std::function<FailureOr<int>(OpOperand &)>> policy;
};

```
- **EN**: Implements logic around `addRefCounting`, `initializeDefaultPolicy`, `function`.
- **CN**: 围绕 `addRefCounting`, `initializeDefaultPolicy`, `function` 实现具体逻辑。

### Lines 486-495
```cpp
} // namespace

LogicalResult
AsyncRuntimePolicyBasedRefCountingPass::addRefCounting(Value value) {
  // Short-circuit reference counting for values without uses.
  if (succeeded(dropRefIfNoUses(value)))
    return success();

  OpBuilder b(value.getContext());

```
- **EN**: Implements logic around `addRefCounting`, `succeeded`, `success`, `b`.
- **CN**: 围绕 `addRefCounting`, `succeeded`, `success`, `b` 实现具体逻辑。

### Lines 496-506
```cpp
  // Consult the user defined policy for every value use.
  for (OpOperand &operand : value.getUses()) {
    Location loc = operand.getOwner()->getLoc();

    for (auto &func : policy) {
      FailureOr<int> refCount = func(operand);
      if (failed(refCount))
        return failure();

      int cnt = *refCount;

```
- **EN**: Implements logic around `getUses`, `getOwner`, `func`, `failed`, and 1 more symbols.
- **CN**: 围绕 `getUses`, `getOwner`, `func`, `failed`, and 1 more symbols 实现具体逻辑。

### Lines 507-520
```cpp
      // Create `add_ref` operation before the operand owner.
      if (cnt > 0) {
        b.setInsertionPoint(operand.getOwner());
        RuntimeAddRefOp::create(b, loc, value, b.getI64IntegerAttr(cnt));
      }

      // Create `drop_ref` operation after the operand owner.
      if (cnt < 0) {
        b.setInsertionPointAfter(operand.getOwner());
        RuntimeDropRefOp::create(b, loc, value, b.getI64IntegerAttr(-cnt));
      }
    }
  }

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `setInsertionPointAfter`.
- **CN**: 围绕 `setInsertionPoint`, `create`, `setInsertionPointAfter` 实现具体逻辑。

### Lines 521-532
```cpp
  return success();
}

void AsyncRuntimePolicyBasedRefCountingPass::initializeDefaultPolicy() {
  policy.push_back([](OpOperand &operand) -> FailureOr<int> {
    Operation *op = operand.getOwner();
    Type type = operand.get().getType();

    bool isToken = isa<TokenType>(type);
    bool isGroup = isa<GroupType>(type);
    bool isValue = isa<ValueType>(type);

```
- **EN**: Implements logic around `success`, `initializeDefaultPolicy`, `push_back`, `getOwner`, and 2 more symbols.
- **CN**: 围绕 `success`, `initializeDefaultPolicy`, `push_back`, `getOwner`, and 2 more symbols 实现具体逻辑。

### Lines 533-544
```cpp
    // Drop reference after async token or group error check (coro await).
    if (isa<RuntimeIsErrorOp>(op))
      return (isToken || isGroup) ? -1 : 0;

    // Drop reference after async value load.
    if (isa<RuntimeLoadOp>(op))
      return isValue ? -1 : 0;

    // Drop reference after async token added to the group.
    if (isa<RuntimeAddToGroupOp>(op))
      return isToken ? -1 : 0;

```
- **EN**: Implements logic around `isa`.
- **CN**: 围绕 `isa` 实现具体逻辑。

### Lines 545-553
```cpp
    return 0;
  });
}

void AsyncRuntimePolicyBasedRefCountingPass::runOnOperation() {
  auto functor = [&](Value value) { return addRefCounting(value); };
  if (failed(walkReferenceCountedValues(getOperation(), functor)))
    signalPassFailure();
}
```
- **EN**: Implements logic around `runOnOperation`, `addRefCounting`, `failed`, `signalPassFailure`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `addRefCounting`, `failed`, `signalPassFailure` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Async tokens and groups / 异步 token 与 group**:
  - **EN**: Represents async dependencies, tasks, await operations, and runtime integration.
  - **CN**: 表示异步依赖、任务、await 操作与运行时集成。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Async/Passes.h`, `mlir/Analysis/Liveness.h`, `mlir/Dialect/Async/IR/Async.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `llvm/ADT/SmallSet.h`, `mlir/Dialect/Async/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), MLIR analysis interfaces / MLIR 分析接口 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
