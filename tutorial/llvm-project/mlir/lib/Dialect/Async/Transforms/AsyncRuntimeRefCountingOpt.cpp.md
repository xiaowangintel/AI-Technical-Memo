# AsyncRuntimeRefCountingOpt.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Async/Transforms/AsyncRuntimeRefCountingOpt.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Optimize Async dialect reference counting operations.
  - **CN**: 实现 Async 方言与异步执行原语 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AsyncRuntimeRefCountingOpt.cpp - Async Ref Counting --------------===//
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
// Optimize Async dialect reference counting operations.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Async/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Async/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Async/Passes.h`。

### Lines 15-23
```cpp
#include "mlir/Dialect/Async/IR/Async.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "llvm/Support/Debug.h"

namespace mlir {
#define GEN_PASS_DEF_ASYNCRUNTIMEREFCOUNTINGOPTPASS
#include "mlir/Dialect/Async/Passes.h.inc"
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 24-30
```cpp
#define DEBUG_TYPE "async-ref-counting"

using namespace mlir;
using namespace mlir::async;

namespace {

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 31-37
```cpp
class AsyncRuntimeRefCountingOptPass
    : public impl::AsyncRuntimeRefCountingOptPassBase<
          AsyncRuntimeRefCountingOptPass> {
public:
  AsyncRuntimeRefCountingOptPass() = default;
  void runOnOperation() override;

```
- **EN**: Introduces declarations for `AsyncRuntimeRefCountingOptPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncRuntimeRefCountingOptPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 38-44
```cpp
private:
  LogicalResult optimizeReferenceCounting(
      Value value, llvm::SmallDenseMap<Operation *, Operation *> &cancellable);
};

} // namespace

```
- **EN**: Implements logic around `optimizeReferenceCounting`.
- **CN**: 围绕 `optimizeReferenceCounting` 实现具体逻辑。

### Lines 45-58
```cpp
LogicalResult AsyncRuntimeRefCountingOptPass::optimizeReferenceCounting(
    Value value, llvm::SmallDenseMap<Operation *, Operation *> &cancellable) {
  Region *definingRegion = value.getParentRegion();

  // Find all users of the `value` inside each block, including operations that
  // do not use `value` directly, but have a direct use inside nested region(s).
  //
  // Example:
  //
  //  ^bb1:
  //    %token = ...
  //    scf.if %cond {
  //      ^bb2:
  //      async.runtime.await %token : !async.token
```
- **EN**: Implements logic around `optimizeReferenceCounting`, `getParentRegion`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `optimizeReferenceCounting`, `getParentRegion` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 59-69
```cpp
  //    }
  //
  // %token has a use inside ^bb2 (`async.runtime.await`) and inside ^bb1
  // (`scf.if`).

  struct BlockUsersInfo {
    llvm::SmallVector<RuntimeAddRefOp, 4> addRefs;
    llvm::SmallVector<RuntimeDropRefOp, 4> dropRefs;
    llvm::SmallVector<Operation *, 4> users;
  };

```
- **EN**: Introduces declarations for `BlockUsersInfo`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `BlockUsersInfo` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 70-81
```cpp
  llvm::DenseMap<Block *, BlockUsersInfo> blockUsers;

  auto updateBlockUsersInfo = [&](Operation *user) {
    BlockUsersInfo &info = blockUsers[user->getBlock()];
    info.users.push_back(user);

    if (auto addRef = dyn_cast<RuntimeAddRefOp>(user))
      info.addRefs.push_back(addRef);
    if (auto dropRef = dyn_cast<RuntimeDropRefOp>(user))
      info.dropRefs.push_back(dropRef);
  };

```
- **EN**: Implements logic around `getBlock`, `push_back`, `dyn_cast`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBlock`, `push_back`, `dyn_cast` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 82-88
```cpp
  for (Operation *user : value.getUsers()) {
    while (user->getParentRegion() != definingRegion) {
      updateBlockUsersInfo(user);
      user = user->getParentOp();
      assert(user != nullptr && "value user lies outside of the value region");
    }

```
- **EN**: Implements logic around `getUsers`, `getParentRegion`, `updateBlockUsersInfo`, `getParentOp`, and 1 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getUsers`, `getParentRegion`, `updateBlockUsersInfo`, `getParentOp`, and 1 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 89-102
```cpp
    updateBlockUsersInfo(user);
  }

  // Sort all operations found in the block.
  auto preprocessBlockUsersInfo = [](BlockUsersInfo &info) -> BlockUsersInfo & {
    auto isBeforeInBlock = [](Operation *a, Operation *b) -> bool {
      return a->isBeforeInBlock(b);
    };
    llvm::sort(info.addRefs, isBeforeInBlock);
    llvm::sort(info.dropRefs, isBeforeInBlock);
    llvm::sort(info.users, [&](Operation *a, Operation *b) -> bool {
      return isBeforeInBlock(a, b);
    });

```
- **EN**: Implements logic around `updateBlockUsersInfo`, `isBeforeInBlock`, `sort`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `updateBlockUsersInfo`, `isBeforeInBlock`, `sort` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 103-110
```cpp
    return info;
  };

  // Find and erase matching pairs of `add_ref` / `drop_ref` operations in the
  // blocks that modify the reference count of the `value`.
  for (auto &kv : blockUsers) {
    BlockUsersInfo &info = preprocessBlockUsersInfo(kv.second);

```
- **EN**: Implements logic around `preprocessBlockUsersInfo`.
- **CN**: 围绕 `preprocessBlockUsersInfo` 实现具体逻辑。

### Lines 111-117
```cpp
    for (RuntimeAddRefOp addRef : info.addRefs) {
      for (RuntimeDropRefOp dropRef : info.dropRefs) {
        // `drop_ref` operation after the `add_ref` with matching count.
        if (dropRef.getCount() != addRef.getCount() ||
            dropRef->isBeforeInBlock(addRef.getOperation()))
          continue;

```
- **EN**: Implements logic around `getCount`, `isBeforeInBlock`.
- **CN**: 围绕 `getCount`, `isBeforeInBlock` 实现具体逻辑。

### Lines 118-131
```cpp
        // When reference counted value passed to a function as an argument,
        // function takes ownership of +1 reference and it will drop it before
        // returning.
        //
        // Example:
        //
        //   %token = ... : !async.token
        //
        //   async.runtime.add_ref %token {count = 1 : i64} : !async.token
        //   call @pass_token(%token: !async.token, ...)
        //
        //   async.await %token : !async.token
        //   async.runtime.drop_ref %token {count = 1 : i64} : !async.token
        //
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 132-145
```cpp
        // In this example if we'll cancel a pair of reference counting
        // operations we might end up with a deallocated token when we'll
        // reach `async.await` operation.
        Operation *firstFunctionCallUser = nullptr;
        Operation *lastNonFunctionCallUser = nullptr;

        for (Operation *user : info.users) {
          // `user` operation lies after `addRef` ...
          if (user == addRef || user->isBeforeInBlock(addRef))
            continue;
          // ... and before `dropRef`.
          if (user == dropRef || dropRef->isBeforeInBlock(user))
            break;

```
- **EN**: Implements logic around `isBeforeInBlock`.
- **CN**: 围绕 `isBeforeInBlock` 实现具体逻辑。

### Lines 146-154
```cpp
          // Find the first function call user of the reference counted value.
          Operation *functionCall = dyn_cast<func::CallOp>(user);
          if (functionCall &&
              (!firstFunctionCallUser ||
               functionCall->isBeforeInBlock(firstFunctionCallUser))) {
            firstFunctionCallUser = functionCall;
            continue;
          }

```
- **EN**: Implements logic around `CallOp>`, `isBeforeInBlock`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `CallOp>`, `isBeforeInBlock` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 155-163
```cpp
          // Find the last regular user of the reference counted value.
          if (!functionCall &&
              (!lastNonFunctionCallUser ||
               lastNonFunctionCallUser->isBeforeInBlock(user))) {
            lastNonFunctionCallUser = user;
            continue;
          }
        }

```
- **EN**: Implements logic around `isBeforeInBlock`.
- **CN**: 围绕 `isBeforeInBlock` 实现具体逻辑。

### Lines 164-173
```cpp
        // Non function call user after the function call user of the reference
        // counted value.
        if (firstFunctionCallUser && lastNonFunctionCallUser &&
            firstFunctionCallUser->isBeforeInBlock(lastNonFunctionCallUser))
          continue;

        // Try to cancel the pair of `add_ref` and `drop_ref` operations.
        auto emplaced = cancellable.try_emplace(dropRef.getOperation(),
                                                addRef.getOperation());

```
- **EN**: Implements logic around `isBeforeInBlock`, `try_emplace`, `getOperation`.
- **CN**: 围绕 `isBeforeInBlock`, `try_emplace`, `getOperation` 实现具体逻辑。

### Lines 174-182
```cpp
        if (!emplaced.second) // `drop_ref` was already marked for removal
          continue;           // go to the next `drop_ref`

        if (emplaced.second) // successfully cancelled `add_ref` <-> `drop_ref`
          break;             // go to the next `add_ref`
      }
    }
  }

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 183-194
```cpp
  return success();
}

void AsyncRuntimeRefCountingOptPass::runOnOperation() {
  Operation *op = getOperation();

  // Mapping from `dropRef.getOperation()` to `addRef.getOperation()`.
  //
  // Find all cancellable pairs of operation and erase them in the end to keep
  // all iterators valid while we are walking the function operations.
  llvm::SmallDenseMap<Operation *, Operation *> cancellable;

```
- **EN**: Implements logic around `success`, `runOnOperation`, `getOperation`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `success`, `runOnOperation`, `getOperation` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 195-201
```cpp
  // Optimize reference counting for values defined by block arguments.
  WalkResult blockWalk = op->walk([&](Block *block) -> WalkResult {
    for (BlockArgument arg : block->getArguments())
      if (isRefCounted(arg.getType()))
        if (failed(optimizeReferenceCounting(arg, cancellable)))
          return WalkResult::interrupt();

```
- **EN**: Implements logic around `walk`, `getArguments`, `isRefCounted`, `failed`, and 1 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `walk`, `getArguments`, `isRefCounted`, `failed`, and 1 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 202-214
```cpp
    return WalkResult::advance();
  });

  if (blockWalk.wasInterrupted())
    signalPassFailure();

  // Optimize reference counting for values defined by operation results.
  WalkResult opWalk = op->walk([&](Operation *op) -> WalkResult {
    for (unsigned i = 0; i < op->getNumResults(); ++i)
      if (isRefCounted(op->getResultTypes()[i]))
        if (failed(optimizeReferenceCounting(op->getResult(i), cancellable)))
          return WalkResult::interrupt();

```
- **EN**: Implements logic around `advance`, `wasInterrupted`, `signalPassFailure`, `walk`, and 4 more symbols.
- **CN**: 围绕 `advance`, `wasInterrupted`, `signalPassFailure`, `walk`, and 4 more symbols 实现具体逻辑。

### Lines 215-225
```cpp
    return WalkResult::advance();
  });

  if (opWalk.wasInterrupted())
    signalPassFailure();

  LLVM_DEBUG({
    llvm::dbgs() << "Found " << cancellable.size()
                 << " cancellable reference counting operations\n";
  });

```
- **EN**: Implements logic around `advance`, `wasInterrupted`, `signalPassFailure`, `dbgs`.
- **CN**: 围绕 `advance`, `wasInterrupted`, `signalPassFailure`, `dbgs` 实现具体逻辑。

### Lines 226-231
```cpp
  // Erase all cancellable `add_ref <-> drop_ref` operation pairs.
  for (auto &kv : cancellable) {
    kv.first->erase();
    kv.second->erase();
  }
}
```
- **EN**: Implements logic around `erase`.
- **CN**: 围绕 `erase` 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Async/Passes.h`, `mlir/Dialect/Async/IR/Async.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `llvm/Support/Debug.h`, `mlir/Dialect/Async/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (4), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
