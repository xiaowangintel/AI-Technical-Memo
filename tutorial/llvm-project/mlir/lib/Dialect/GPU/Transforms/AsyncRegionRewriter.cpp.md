# AsyncRegionRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/AsyncRegionRewriter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the GPU dialect pattern rewriters that make GPU op within a region execute asynchronously.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- AsyncRegionRewriter.cpp - Implementation of GPU async rewriters ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the GPU dialect pattern rewriters that make GPU op
// within a region execute asynchronously.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 13-25
```cpp

#include "mlir/Dialect/GPU/Transforms/Passes.h"

#include "mlir/Dialect/Async/IR/Async.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Utils/GPUUtils.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/Async/IR/Async.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/Async/IR/Async.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`。

### Lines 26-36
```cpp
namespace mlir {
#define GEN_PASS_DEF_GPUASYNCREGIONPASS
#include "mlir/Dialect/GPU/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;

namespace {
class GpuAsyncRegionPass
    : public impl::GpuAsyncRegionPassBase<GpuAsyncRegionPass> {
  struct ThreadTokenCallback;
```
- **EN**: Introduces declarations for `mlir`, `GpuAsyncRegionPass`, `ThreadTokenCallback`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `GpuAsyncRegionPass`, `ThreadTokenCallback` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 37-47
```cpp
  struct DeferWaitCallback;
  struct SingleTokenUseCallback;
  void runOnOperation() override;
};
} // namespace

static bool isTerminator(Operation *op) {
  return op->mightHaveTrait<OpTrait::IsTerminator>();
}
static bool hasSideEffects(Operation *op) { return !isMemoryEffectFree(op); }

```
- **EN**: Introduces declarations for `DeferWaitCallback`, `SingleTokenUseCallback`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DeferWaitCallback`, `SingleTokenUseCallback` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 48-60
```cpp
// Region walk callback which makes GPU ops implementing the AsyncOpInterface
// execute asynchronously.
struct GpuAsyncRegionPass::ThreadTokenCallback {
  ThreadTokenCallback(MLIRContext &context) : builder(&context) {}

  WalkResult operator()(Block *block) {
    for (Operation &op : make_early_inc_range(*block)) {
      if (failed(visit(&op)))
        return WalkResult::interrupt();
    }
    return WalkResult::advance();
  }

```
- **EN**: Introduces declarations for `GpuAsyncRegionPass::ThreadTokenCallback`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuAsyncRegionPass::ThreadTokenCallback` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 61-80
```cpp
private:
  // If `op` implements the AsyncOpInterface, insert a `gpu.wait async` to
  // create a current token (unless it already exists), and 'thread' that token
  // through the `op` so that it executes asynchronously.
  //
  // If `op` is a terminator or an op with side-effects, insert a `gpu.wait` to
  // host-synchronize execution. A `!gpu.async.token` will therefore only be
  // used inside of its block and GPU execution will always synchronize with
  // the host at block boundaries.
  LogicalResult visit(Operation *op) {
    if (isa<gpu::LaunchOp>(op))
      return op->emitOpError("replace with gpu.launch_func first");
    if (auto waitOp = llvm::dyn_cast<gpu::WaitOp>(op)) {
      if (currentToken)
        waitOp.addAsyncDependency(currentToken);
      currentToken = waitOp.getAsyncToken();
      return success();
    }
    builder.setInsertionPoint(op);
    if (auto asyncOp = dyn_cast<gpu::AsyncOpInterface>(op))
```
- **EN**: Implements logic around `visit`, `LaunchOp>`, `emitOpError`, `WaitOp>`, and 5 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `visit`, `LaunchOp>`, `emitOpError`, `WaitOp>`, and 5 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 81-94
```cpp
      return rewriteAsyncOp(asyncOp); // Replace GPU op with async version.
    if (!currentToken)
      return success();
    // Insert host synchronization before terminator or op with side effects.
    if (isTerminator(op) || hasSideEffects(op))
      currentToken = createWaitOp(op->getLoc(), Type(), {currentToken});
    return success();
  }

  // Replaces asyncOp with a clone that returns a token.
  LogicalResult rewriteAsyncOp(gpu::AsyncOpInterface asyncOp) {
    auto *op = asyncOp.getOperation();
    auto tokenType = builder.getType<gpu::AsyncTokenType>();

```
- **EN**: Implements logic around `rewriteAsyncOp`, `success`, `isTerminator`, `createWaitOp`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `rewriteAsyncOp`, `success`, `isTerminator`, `createWaitOp`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 95-105
```cpp
    // If there is no current token, insert a `gpu.wait async` without
    // dependencies to create one.
    if (!currentToken)
      currentToken = createWaitOp(op->getLoc(), tokenType, {});
    asyncOp.addAsyncDependency(currentToken);

    // Return early if op returns a token already.
    currentToken = asyncOp.getAsyncToken();
    if (currentToken)
      return success();

```
- **EN**: Implements logic around `createWaitOp`, `addAsyncDependency`, `getAsyncToken`, `success`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `createWaitOp`, `addAsyncDependency`, `getAsyncToken`, `success` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 106-115
```cpp
    // Clone the op to return a token in addition to the other results.
    SmallVector<Type, 1> resultTypes;
    resultTypes.reserve(1 + op->getNumResults());
    copy(op->getResultTypes(), std::back_inserter(resultTypes));
    resultTypes.push_back(tokenType);
    auto *newOp = Operation::create(
        op->getLoc(), op->getName(), resultTypes, op->getOperands(),
        op->getDiscardableAttrDictionary(), op->getPropertiesStorage(),
        op->getSuccessors(), op->getNumRegions());

```
- **EN**: Implements logic around `reserve`, `copy`, `push_back`, `create`, and 3 more symbols.
- **CN**: 围绕 `reserve`, `copy`, `push_back`, `create`, and 3 more symbols 实现具体逻辑。

### Lines 116-127
```cpp
    // Clone regions into new op.
    IRMapping mapping;
    for (auto pair : llvm::zip_first(op->getRegions(), newOp->getRegions()))
      std::get<0>(pair).cloneInto(&std::get<1>(pair), mapping);

    // Replace the op with the async clone.
    auto results = newOp->getResults();
    currentToken = results.back();
    builder.insert(newOp);
    op->replaceAllUsesWith(results.drop_back());
    op->erase();

```
- **EN**: Implements logic around `zip_first`, `get`, `getResults`, `back`, and 3 more symbols.
- **CN**: 围绕 `zip_first`, `get`, `getResults`, `back`, and 3 more symbols 实现具体逻辑。

### Lines 128-137
```cpp
    return success();
  }

  Value createWaitOp(Location loc, Type resultType, ValueRange operands) {
    return gpu::WaitOp::create(builder, loc, resultType, operands)
        .getAsyncToken();
  }

  OpBuilder builder;

```
- **EN**: Implements logic around `success`, `createWaitOp`, `create`, `getAsyncToken`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `success`, `createWaitOp`, `create`, `getAsyncToken` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 138-151
```cpp
  // The token that represents the current asynchronous dependency. It's valid
  // range starts with a `gpu.wait async` op, and ends with a `gpu.wait` op.
  // In between, each gpu::AsyncOpInterface depends on the current token and
  // produces the new one.
  Value currentToken = {};
};

/// Erases `executeOp` and returns a clone with additional `results`.
static async::ExecuteOp addExecuteResults(async::ExecuteOp executeOp,
                                          ValueRange results) {
  // Add values to async.yield op.
  Operation *yieldOp = executeOp.getBody()->getTerminator();
  yieldOp->insertOperands(yieldOp->getNumOperands(), results);

```
- **EN**: Implements logic around `addExecuteResults`, `getBody`, `insertOperands`; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `addExecuteResults`, `getBody`, `insertOperands` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 152-165
```cpp
  // Construct new result type list with additional types.
  SmallVector<Type, 2> resultTypes;
  resultTypes.reserve(executeOp.getNumResults() + results.size());
  transform(executeOp.getResultTypes(), std::back_inserter(resultTypes),
            [](Type type) {
              // Extract value type from !async.value.
              if (auto valueType = dyn_cast<async::ValueType>(type))
                return valueType.getValueType();
              assert(isa<async::TokenType>(type) && "expected token type");
              return type;
            });
  transform(results, std::back_inserter(resultTypes),
            [](Value value) { return value.getType(); });

```
- **EN**: Implements logic around `reserve`, `transform`, `ValueType>`, `getValueType`, and 2 more symbols.
- **CN**: 围绕 `reserve`, `transform`, `ValueType>`, `getValueType`, and 2 more symbols 实现具体逻辑。

### Lines 166-175
```cpp
  // Clone executeOp with the extra results.
  OpBuilder builder(executeOp);
  auto newOp = async::ExecuteOp::create(
      builder, executeOp.getLoc(),
      TypeRange{resultTypes}.drop_front() /*drop token*/,
      executeOp.getDependencies(), executeOp.getBodyOperands());
  IRMapping mapper;
  newOp.getRegion().getBlocks().clear();
  executeOp.getRegion().cloneInto(&newOp.getRegion(), mapper);

```
- **EN**: Implements logic around `builder`, `create`, `getLoc`, `drop_front`, and 2 more symbols.
- **CN**: 围绕 `builder`, `create`, `getLoc`, `drop_front`, and 2 more symbols 实现具体逻辑。

### Lines 176-186
```cpp
  // Replace executeOp with cloned one.
  executeOp.getOperation()->replaceAllUsesWith(
      newOp.getResults().drop_back(results.size()));
  executeOp.erase();

  return newOp;
}

// Callback for `async.execute` ops which tries to push the contained
// synchronous `gpu.wait` op to the dependencies of the `async.execute`.
struct GpuAsyncRegionPass::DeferWaitCallback {
```
- **EN**: Introduces declarations for `GpuAsyncRegionPass::DeferWaitCallback`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuAsyncRegionPass::DeferWaitCallback` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 187-204
```cpp
  // If the `executeOp`s token is used only in `async.execute` or `async.await`
  // ops, add the region's last `gpu.wait` op to the worklist if it is
  // synchronous and is the last op with side effects.
  void operator()(async::ExecuteOp executeOp) {
    if (!areAllUsersExecuteOrAwait(executeOp.getToken()))
      return;
    // async.execute's region is currently restricted to one block.
    for (auto &op : llvm::reverse(executeOp.getBody()->without_terminator())) {
      if (auto waitOp = dyn_cast<gpu::WaitOp>(op)) {
        if (!waitOp.getAsyncToken())
          worklist.push_back(waitOp);
        return;
      }
      if (hasSideEffects(&op))
        return;
    }
  }

```
- **EN**: Implements logic around `operator`, `areAllUsersExecuteOrAwait`, `reverse`, `WaitOp>`, and 3 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `operator`, `areAllUsersExecuteOrAwait`, `reverse`, `WaitOp>`, and 3 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 205-215
```cpp
  // The destructor performs the actual rewrite work.
  ~DeferWaitCallback() {
    for (size_t i = 0; i < worklist.size(); ++i) {
      auto waitOp = worklist[i];
      auto executeOp = waitOp->getParentOfType<async::ExecuteOp>();

      // Erase `gpu.wait` and return async dependencies from execute op instead.
      SmallVector<Value, 4> dependencies = waitOp.getAsyncDependencies();
      waitOp.erase();
      executeOp = addExecuteResults(executeOp, dependencies);

```
- **EN**: Implements logic around `~DeferWaitCallback`, `size`, `ExecuteOp>`, `getAsyncDependencies`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `~DeferWaitCallback`, `size`, `ExecuteOp>`, `getAsyncDependencies`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 216-225
```cpp
      // Add the async dependency to each user of the `async.execute` token.
      auto asyncTokens = executeOp.getResults().take_back(dependencies.size());
      SmallVector<Operation *, 4> users(executeOp.getToken().user_begin(),
                                        executeOp.getToken().user_end());
      for (Operation *user : users)
        addAsyncDependencyAfter(asyncTokens, user);
    }
  }

private:
```
- **EN**: Implements logic around `getResults`, `users`, `getToken`, `addAsyncDependencyAfter`.
- **CN**: 围绕 `getResults`, `users`, `getToken`, `addAsyncDependencyAfter` 实现具体逻辑。

### Lines 226-236
```cpp
  // Returns whether all token users are either 'async.execute' or 'async.await'
  // ops. This is used as a requirement for pushing 'gpu.wait' ops from a
  // 'async.execute' body to it's users. Specifically, we do not allow
  // terminator users, because it could mean that the `async.execute` is inside
  // control flow code.
  static bool areAllUsersExecuteOrAwait(Value token) {
    return !token.use_empty() &&
           llvm::all_of(token.getUsers(),
                        llvm::IsaPred<async::ExecuteOp, async::AwaitOp>);
  }

```
- **EN**: Implements logic around `areAllUsersExecuteOrAwait`, `use_empty`, `all_of`; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `areAllUsersExecuteOrAwait`, `use_empty`, `all_of` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 237-256
```cpp
  // Add the `asyncToken` as dependency as needed after `op`.
  void addAsyncDependencyAfter(ValueRange asyncTokens, Operation *op) {
    OpBuilder builder(op->getContext());
    auto loc = op->getLoc();

    Block::iterator it;
    SmallVector<Value, 1> tokens;
    tokens.reserve(asyncTokens.size());
    TypeSwitch<Operation *>(op)
        .Case([&](async::AwaitOp awaitOp) {
          // Add async.await ops to wait for the !gpu.async.tokens.
          builder.setInsertionPointAfter(op);
          for (auto asyncToken : asyncTokens)
            tokens.push_back(
                async::AwaitOp::create(builder, loc, asyncToken).getResult());
          // Set `it` after the inserted async.await ops.
          it = builder.getInsertionPoint();
        })
        .Case([&](async::ExecuteOp executeOp) {
          // Set `it` to the beginning of the region and add asyncTokens to the
```
- **EN**: Implements logic around `addAsyncDependencyAfter`, `builder`, `getLoc`, `reserve`, and 5 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `addAsyncDependencyAfter`, `builder`, `getLoc`, `reserve`, and 5 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 257-267
```cpp
          // async.execute operands.
          it = executeOp.getBody()->begin();
          executeOp.getBodyOperandsMutable().append(asyncTokens);
          SmallVector<Type, 1> tokenTypes(
              asyncTokens.size(), builder.getType<gpu::AsyncTokenType>());
          SmallVector<Location, 1> tokenLocs(asyncTokens.size(),
                                             executeOp.getLoc());
          copy(executeOp.getBody()->addArguments(tokenTypes, tokenLocs),
               std::back_inserter(tokens));
        });

```
- **EN**: Implements logic around `getBody`, `getBodyOperandsMutable`, `tokenTypes`, `size`, and 4 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getBody`, `getBodyOperandsMutable`, `tokenTypes`, `size`, and 4 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 268-280
```cpp
    // Advance `it` to terminator or op with side-effects.
    it = std::find_if(it, Block::iterator(), [](Operation &op) {
      return isTerminator(&op) || hasSideEffects(&op);
    });

    // If `op` implements the AsyncOpInterface, add `token` to the list of async
    // dependencies.
    if (auto asyncOp = dyn_cast<gpu::AsyncOpInterface>(*it)) {
      for (auto token : tokens)
        asyncOp.addAsyncDependency(token);
      return;
    }

```
- **EN**: Implements logic around `find_if`, `isTerminator`, `AsyncOpInterface>`, `addAsyncDependency`; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `find_if`, `isTerminator`, `AsyncOpInterface>`, `addAsyncDependency` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 281-292
```cpp
    // Otherwise, insert a gpu.wait before 'it'.
    builder.setInsertionPoint(it->getBlock(), it);
    auto waitOp = gpu::WaitOp::create(builder, loc, Type{}, tokens);

    // If the new waitOp is at the end of an async.execute region, add it to the
    // worklist. 'operator()(executeOp)' would do the same, but this is faster.
    auto executeOp = dyn_cast<async::ExecuteOp>(it->getParentOp());
    if (executeOp && areAllUsersExecuteOrAwait(executeOp.getToken()) &&
        !it->getNextNode())
      worklist.push_back(waitOp);
  }

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `ExecuteOp>`, `areAllUsersExecuteOrAwait`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `setInsertionPoint`, `create`, `ExecuteOp>`, `areAllUsersExecuteOrAwait`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 293-311
```cpp
  SmallVector<gpu::WaitOp, 8> worklist;
};

// Callback for `async.execute` ops which repeats !gpu.async.token results
// so that each of them is only used once.
struct GpuAsyncRegionPass::SingleTokenUseCallback {
  void operator()(async::ExecuteOp executeOp) {
    // Extract !gpu.async.token results which have multiple uses.
    auto multiUseResults = llvm::make_filter_range(
        executeOp.getBodyResults(), [](OpResult result) {
          if (result.use_empty() || result.hasOneUse())
            return false;
          auto valueType = dyn_cast<async::ValueType>(result.getType());
          return valueType &&
                 isa<gpu::AsyncTokenType>(valueType.getValueType());
        });
    if (multiUseResults.empty())
      return;

```
- **EN**: Introduces declarations for `GpuAsyncRegionPass::SingleTokenUseCallback`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuAsyncRegionPass::SingleTokenUseCallback` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 312-331
```cpp
    // Indices within !async.execute results (i.e. without the async.token).
    SmallVector<int, 4> indices;
    transform(multiUseResults, std::back_inserter(indices),
              [](OpResult result) {
                return result.getResultNumber() - 1; // Index without token.
              });

    for (auto index : indices) {
      assert(!executeOp.getBodyResults()[index].getUses().empty());
      // Repeat async.yield token result, one for each use after the first one.
      auto uses = llvm::drop_begin(executeOp.getBodyResults()[index].getUses());
      auto count = std::distance(uses.begin(), uses.end());
      auto yieldOp = cast<async::YieldOp>(executeOp.getBody()->getTerminator());
      SmallVector<Value, 4> operands(count, yieldOp.getOperand(index));
      executeOp = addExecuteResults(executeOp, operands);
      // Update 'uses' to refer to the new executeOp.
      uses = llvm::drop_begin(executeOp.getBodyResults()[index].getUses());
      auto results = executeOp.getBodyResults().take_back(count);
      for (auto pair : llvm::zip(uses, results))
        std::get<0>(pair).set(std::get<1>(pair));
```
- **EN**: Implements logic around `transform`, `getResultNumber`, `assert`, `drop_begin`, and 7 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `transform`, `getResultNumber`, `assert`, `drop_begin`, and 7 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 332-342
```cpp
    }
  }
};

// Replaces synchronous GPU ops in the op's region with asynchronous ones and
// inserts the necessary synchronization (as gpu.wait ops). Assumes sequential
// execution semantics and that no GPU ops are asynchronous yet.
void GpuAsyncRegionPass::runOnOperation() {
  if (getOperation()->walk(ThreadTokenCallback(getContext())).wasInterrupted())
    return signalPassFailure();

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `signalPassFailure`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `signalPassFailure` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 343-347
```cpp
  // Collect gpu.wait ops that we can move out of async.execute regions.
  getOperation().getRegion().walk(DeferWaitCallback());
  // Makes each !gpu.async.token returned from async.execute op have single use.
  getOperation().getRegion().walk(SingleTokenUseCallback());
}
```
- **EN**: Implements logic around `getOperation`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getOperation` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Async tokens and groups / 异步 token 与 group**:
  - **EN**: Represents async dependencies, tasks, await operations, and runtime integration.
  - **CN**: 表示异步依赖、任务、await 操作与运行时集成。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/Async/IR/Async.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Utils/GPUUtils.h`, `mlir/IR/Builders.h`, `mlir/IR/IRMapping.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `mlir/Support/LLVM.h`, `llvm/ADT/TypeSwitch.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), MLIR IR core abstractions / MLIR IR 核心抽象 (2), MLIR interface declarations / MLIR 接口声明 (1), MLIR support-library helpers / MLIR Support 库辅助功能 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
