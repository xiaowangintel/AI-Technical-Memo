# AsyncToAsyncRuntime.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Async/Transforms/AsyncToAsyncRuntime.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements lowering from high level async operations to async.coro and async.runtime operations.
  - **CN**: 实现 Async 方言与异步执行原语 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- AsyncToAsyncRuntime.cpp - Lower from Async to Async Runtime --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements lowering from high level async operations to async.coro
// and async.runtime operations.
//
//===----------------------------------------------------------------------===//

#include <utility>

```
- **EN**: Pulls in the headers needed by this translation unit, including `utility`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `utility`。

### Lines 16-31
```cpp
#include "mlir/Dialect/Async/Passes.h"

#include "PassDetail.h"
#include "mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Async/IR/Async.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/DialectConversion.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/Support/Debug.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Async/Passes.h`, `PassDetail.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Dialect/Arith/IR/Arith.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Async/Passes.h`, `PassDetail.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Dialect/Arith/IR/Arith.h`。

### Lines 32-45
```cpp
namespace mlir {
#define GEN_PASS_DEF_ASYNCTOASYNCRUNTIMEPASS
#define GEN_PASS_DEF_ASYNCFUNCTOASYNCRUNTIMEPASS
#include "mlir/Dialect/Async/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::async;

#define DEBUG_TYPE "async-to-async-runtime"
// Prefix for functions outlined from `async.execute` op regions.
static constexpr const char kAsyncFnPrefix[] = "async_execute_fn";

namespace {
```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 46-61
```cpp

class AsyncToAsyncRuntimePass
    : public impl::AsyncToAsyncRuntimePassBase<AsyncToAsyncRuntimePass> {
public:
  AsyncToAsyncRuntimePass() = default;
  void runOnOperation() override;
};

} // namespace

namespace {

class AsyncFuncToAsyncRuntimePass
    : public impl::AsyncFuncToAsyncRuntimePassBase<
          AsyncFuncToAsyncRuntimePass> {
public:
```
- **EN**: Introduces declarations for `AsyncToAsyncRuntimePass`, `AsyncFuncToAsyncRuntimePass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncToAsyncRuntimePass`, `AsyncFuncToAsyncRuntimePass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 62-76
```cpp
  AsyncFuncToAsyncRuntimePass() = default;
  void runOnOperation() override;
};

} // namespace

/// Function targeted for coroutine transformation has two additional blocks at
/// the end: coroutine cleanup and coroutine suspension.
///
/// async.await op lowering additionaly creates a resume block for each
/// operation to enable non-blocking waiting via coroutine suspension.
namespace {
struct CoroMachinery {
  func::FuncOp func;

```
- **EN**: Introduces declarations for `CoroMachinery`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CoroMachinery` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 77-92
```cpp
  // Async function returns an optional token, followed by some async values
  //
  //  async.func @foo() -> !async.value<T> {
  //    %cst = arith.constant 42.0 : T
  //    return %cst: T
  //  }
  // Async execute region returns a completion token, and an async value for
  // each yielded value.
  //
  //   %token, %result = async.execute -> !async.value<T> {
  //     %0 = arith.constant ... : T
  //     async.yield %0 : T
  //   }
  std::optional<Value> asyncToken;          // returned completion token
  llvm::SmallVector<Value, 4> returnValues; // returned async values

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 93-120
```cpp
  Value coroHandle; // coroutine handle (!async.coro.getHandle value)
  Block *entry;     // coroutine entry block
  std::optional<Block *> setError; // set returned values to error state
  Block *cleanup;                  // coroutine cleanup block

  // Coroutine cleanup block for destroy after the coroutine is resumed,
  //   e.g. async.coro.suspend state, [suspend], [resume], [destroy]
  //
  // This cleanup block is a duplicate of the cleanup block followed by the
  // resume block. The purpose of having a duplicate cleanup block for destroy
  // is to make the CFG clear so that the control flow analysis won't confuse.
  //
  // The overall structure of the lowered CFG can be the following,
  //
  //     Entry (calling async.coro.suspend)
  //       |                \
  //     Resume           Destroy (duplicate of Cleanup)
  //       |                 |
  //     Cleanup             |
  //       |                 /
  //      End (ends the corontine)
  //
  // If there is resume-specific cleanup logic, it can go into the Cleanup
  // block but not the destroy block. Otherwise, it can fail block dominance
  // check.
  Block *cleanupForDestroy;
  Block *suspend; // coroutine suspension block
};
```
- **EN**: Implements logic around `handle`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `handle` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 121-134
```cpp
} // namespace

using FuncCoroMapPtr =
    std::shared_ptr<llvm::DenseMap<func::FuncOp, CoroMachinery>>;

/// Utility to partially update the regular function CFG to the coroutine CFG
/// compatible with LLVM coroutines switched-resume lowering using
/// `async.runtime.*` and `async.coro.*` operations. Adds a new entry block
/// that branches into preexisting entry block. Also inserts trailing blocks.
///
/// The result types of the passed `func` start with an optional `async.token`
/// and be continued with some number of `async.value`s.
///
/// See LLVM coroutines documentation: https://llvm.org/docs/Coroutines.html
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 135-148
```cpp
///
///  - `entry` block sets up the coroutine.
///  - `set_error` block sets completion token and async values state to error.
///  - `cleanup` block cleans up the coroutine state.
///  - `suspend block after the @llvm.coro.end() defines what value will be
///    returned to the initial caller of a coroutine. Everything before the
///    @llvm.coro.end() will be executed at every suspension point.
///
/// Coroutine structure (only the important bits):
///
///   func @some_fn(<function-arguments>) -> (!async.token, !async.value<T>)
///   {
///     ^entry(<function-arguments>):
///       %token = <async token> : !async.token    // create async runtime token
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 149-162
```cpp
///       %value = <async value> : !async.value<T> // create async value
///       %id = async.coro.getId                   // create a coroutine id
///       %hdl = async.coro.begin %id              // create a coroutine handle
///       cf.br ^preexisting_entry_block
///
///     /*  preexisting blocks modified to branch to the cleanup block */
///
///     ^set_error: // this block created lazily only if needed (see code below)
///       async.runtime.set_error %token : !async.token
///       async.runtime.set_error %value : !async.value<T>
///       cf.br ^cleanup
///
///     ^cleanup:
///       async.coro.free %hdl // delete the coroutine state
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 163-178
```cpp
///       cf.br ^suspend
///
///     ^suspend:
///       async.coro.end %hdl // marks the end of a coroutine
///       return %token, %value : !async.token, !async.value<T>
///   }
///
static CoroMachinery setupCoroMachinery(func::FuncOp func) {
  assert(!func.getBlocks().empty() && "Function must have an entry block");

  MLIRContext *ctx = func.getContext();
  Block *entryBlock = &func.getBlocks().front();
  Block *originalEntryBlock =
      entryBlock->splitBlock(entryBlock->getOperations().begin());
  auto builder = ImplicitLocOpBuilder::atBlockBegin(func->getLoc(), entryBlock);

```
- **EN**: Implements logic around `setupCoroMachinery`, `assert`, `getContext`, `getBlocks`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `setupCoroMachinery`, `assert`, `getContext`, `getBlocks`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 179-197
```cpp
  // ------------------------------------------------------------------------ //
  // Allocate async token/values that we will return from a ramp function.
  // ------------------------------------------------------------------------ //

  // We treat TokenType as state update marker to represent side-effects of
  // async computations
  bool isStateful = isa<TokenType>(func.getResultTypes().front());

  std::optional<Value> retToken;
  if (isStateful)
    retToken.emplace(RuntimeCreateOp::create(builder, TokenType::get(ctx)));

  llvm::SmallVector<Value, 4> retValues;
  ArrayRef<Type> resValueTypes =
      isStateful ? func.getResultTypes().drop_front() : func.getResultTypes();
  for (auto resType : resValueTypes)
    retValues.emplace_back(
        RuntimeCreateOp::create(builder, resType).getResult());

```
- **EN**: Implements logic around `isa`, `emplace`, `getResultTypes`, `emplace_back`, and 1 more symbols.
- **CN**: 围绕 `isa`, `emplace`, `getResultTypes`, `emplace_back`, and 1 more symbols 实现具体逻辑。

### Lines 198-216
```cpp
  // ------------------------------------------------------------------------ //
  // Initialize coroutine: get coroutine id and coroutine handle.
  // ------------------------------------------------------------------------ //
  auto coroIdOp = CoroIdOp::create(builder, CoroIdType::get(ctx));
  auto coroHdlOp =
      CoroBeginOp::create(builder, CoroHandleType::get(ctx), coroIdOp.getId());
  cf::BranchOp::create(builder, originalEntryBlock);

  Block *cleanupBlock = func.addBlock();
  Block *cleanupBlockForDestroy = func.addBlock();
  Block *suspendBlock = func.addBlock();

  // ------------------------------------------------------------------------ //
  // Coroutine cleanup blocks: deallocate coroutine frame, free the memory.
  // ------------------------------------------------------------------------ //
  auto buildCleanupBlock = [&](Block *cb) {
    builder.setInsertionPointToStart(cb);
    CoroFreeOp::create(builder, coroIdOp.getId(), coroHdlOp.getHandle());

```
- **EN**: Implements logic around `create`, `addBlock`, `setInsertionPointToStart`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `create`, `addBlock`, `setInsertionPointToStart` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 217-231
```cpp
    // Branch into the suspend block.
    cf::BranchOp::create(builder, suspendBlock);
  };
  buildCleanupBlock(cleanupBlock);
  buildCleanupBlock(cleanupBlockForDestroy);

  // ------------------------------------------------------------------------ //
  // Coroutine suspend block: mark the end of a coroutine and return allocated
  // async token.
  // ------------------------------------------------------------------------ //
  builder.setInsertionPointToStart(suspendBlock);

  // Mark the end of a coroutine: async.coro.end
  CoroEndOp::create(builder, coroHdlOp.getHandle());

```
- **EN**: Implements logic around `create`, `buildCleanupBlock`, `setInsertionPointToStart`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `create`, `buildCleanupBlock`, `setInsertionPointToStart` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 232-247
```cpp
  // Return created optional `async.token` and `async.values` from the suspend
  // block. This will be the return value of a coroutine ramp function.
  SmallVector<Value, 4> ret;
  if (retToken)
    ret.push_back(*retToken);
  llvm::append_range(ret, retValues);
  func::ReturnOp::create(builder, ret);

  // `async.await` op lowering will create resume blocks for async
  // continuations, and will conditionally branch to cleanup or suspend blocks.

  // The switch-resumed API based coroutine should be marked with
  // presplitcoroutine attribute to mark the function as a coroutine.
  func->setAttr("llvm.passthrough", builder.getArrayAttr(StringAttr::get(
                                        ctx, "presplitcoroutine")));

```
- **EN**: Implements logic around `push_back`, `append_range`, `create`, `setAttr`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `push_back`, `append_range`, `create`, `setAttr` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 248-266
```cpp
  CoroMachinery machinery;
  machinery.func = func;
  machinery.asyncToken = retToken;
  machinery.returnValues = retValues;
  machinery.coroHandle = coroHdlOp.getHandle();
  machinery.entry = entryBlock;
  machinery.setError = std::nullopt; // created lazily only if needed
  machinery.cleanup = cleanupBlock;
  machinery.cleanupForDestroy = cleanupBlockForDestroy;
  machinery.suspend = suspendBlock;
  return machinery;
}

// Lazily creates `set_error` block only if it is required for lowering to the
// runtime operations (see for example lowering of assert operation).
static Block *setupSetErrorBlock(CoroMachinery &coro) {
  if (coro.setError)
    return *coro.setError;

```
- **EN**: Implements logic around `getHandle`, `setupSetErrorBlock`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getHandle`, `setupSetErrorBlock` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 267-282
```cpp
  coro.setError = coro.func.addBlock();
  (*coro.setError)->moveBefore(coro.cleanup);

  auto builder =
      ImplicitLocOpBuilder::atBlockBegin(coro.func->getLoc(), *coro.setError);

  // Coroutine set_error block: set error on token and all returned values.
  if (coro.asyncToken)
    RuntimeSetErrorOp::create(builder, *coro.asyncToken);

  for (Value retValue : coro.returnValues)
    RuntimeSetErrorOp::create(builder, retValue);

  // Branch into the cleanup block.
  cf::BranchOp::create(builder, coro.cleanup);

```
- **EN**: Implements logic around `addBlock`, `moveBefore`, `atBlockBegin`, `create`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `addBlock`, `moveBefore`, `atBlockBegin`, `create` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 283-297
```cpp
  return *coro.setError;
}

//===----------------------------------------------------------------------===//
// async.execute op outlining to the coroutine functions.
//===----------------------------------------------------------------------===//

/// Outline the body region attached to the `async.execute` op into a standalone
/// function.
///
/// Note that this is not reversible transformation.
static std::pair<func::FuncOp, CoroMachinery>
outlineExecuteOp(SymbolTable &symbolTable, ExecuteOp execute) {
  ModuleOp module = execute->getParentOfType<ModuleOp>();

```
- **EN**: Implements logic around `outlineExecuteOp`, `getParentOfType`; this block manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `outlineExecuteOp`, `getParentOfType` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 298-316
```cpp
  MLIRContext *ctx = module.getContext();
  Location loc = execute.getLoc();

  // Make sure that all constants will be inside the outlined async function to
  // reduce the number of function arguments.
  cloneConstantsIntoTheRegion(execute.getBodyRegion());

  // Collect all outlined function inputs.
  SetVector<mlir::Value> functionInputs(llvm::from_range,
                                        execute.getDependencies());
  functionInputs.insert_range(execute.getBodyOperands());
  getUsedValuesDefinedAbove(execute.getBodyRegion(), functionInputs);

  // Collect types for the outlined function inputs and outputs.
  auto typesRange = llvm::map_range(
      functionInputs, [](Value value) { return value.getType(); });
  SmallVector<Type, 4> inputTypes(typesRange.begin(), typesRange.end());
  auto outputTypes = execute.getResultTypes();

```
- **EN**: Implements logic around `getContext`, `getLoc`, `cloneConstantsIntoTheRegion`, `functionInputs`, and 7 more symbols.
- **CN**: 围绕 `getContext`, `getLoc`, `cloneConstantsIntoTheRegion`, `functionInputs`, and 7 more symbols 实现具体逻辑。

### Lines 317-333
```cpp
  auto funcType = FunctionType::get(ctx, inputTypes, outputTypes);
  auto funcAttrs = ArrayRef<NamedAttribute>();

  // TODO: Derive outlined function name from the parent FuncOp (support
  // multiple nested async.execute operations).
  func::FuncOp func =
      func::FuncOp::create(loc, kAsyncFnPrefix, funcType, funcAttrs);
  symbolTable.insert(func);

  SymbolTable::setSymbolVisibility(func, SymbolTable::Visibility::Private);
  auto builder = ImplicitLocOpBuilder::atBlockBegin(loc, func.addEntryBlock());

  // Prepare for coroutine conversion by creating the body of the function.
  {
    size_t numDependencies = execute.getDependencies().size();
    size_t numOperands = execute.getBodyOperands().size();

```
- **EN**: Implements logic around `get`, `ArrayRef`, `create`, `insert`, and 4 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `get`, `ArrayRef`, `create`, `insert`, and 4 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 334-350
```cpp
    // Await on all dependencies before starting to execute the body region.
    for (size_t i = 0; i < numDependencies; ++i)
      AwaitOp::create(builder, func.getArgument(i));

    // Await on all async value operands and unwrap the payload.
    SmallVector<Value, 4> unwrappedOperands(numOperands);
    for (size_t i = 0; i < numOperands; ++i) {
      Value operand = func.getArgument(numDependencies + i);
      unwrappedOperands[i] = AwaitOp::create(builder, loc, operand).getResult();
    }

    // Map from function inputs defined above the execute op to the function
    // arguments.
    IRMapping valueMapping;
    valueMapping.map(functionInputs, func.getArguments());
    valueMapping.map(execute.getBodyRegion().getArguments(), unwrappedOperands);

```
- **EN**: Implements logic around `create`, `unwrappedOperands`, `getArgument`, `map`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `create`, `unwrappedOperands`, `getArgument`, `map` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 351-366
```cpp
    // Clone all operations from the execute operation body into the outlined
    // function body.
    for (Operation &op : execute.getBodyRegion().getOps())
      builder.clone(op, valueMapping);
  }

  // Adding entry/cleanup/suspend blocks.
  CoroMachinery coro = setupCoroMachinery(func);

  // Suspend async function at the end of an entry block, and resume it using
  // Async resume operation (execution will be resumed in a thread managed by
  // the async runtime).
  {
    cf::BranchOp branch = cast<cf::BranchOp>(coro.entry->getTerminator());
    builder.setInsertionPointToEnd(coro.entry);

```
- **EN**: Implements logic around `getBodyRegion`, `clone`, `setupCoroMachinery`, `BranchOp>`, and 1 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBodyRegion`, `clone`, `setupCoroMachinery`, `BranchOp>`, and 1 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 367-381
```cpp
    // Save the coroutine state: async.coro.save
    auto coroSaveOp =
        CoroSaveOp::create(builder, CoroStateType::get(ctx), coro.coroHandle);

    // Pass coroutine to the runtime to be resumed on a runtime managed
    // thread.
    RuntimeResumeOp::create(builder, coro.coroHandle);

    // Add async.coro.suspend as a suspended block terminator.
    CoroSuspendOp::create(builder, coroSaveOp.getState(), coro.suspend,
                          branch.getDest(), coro.cleanupForDestroy);

    branch.erase();
  }

```
- **EN**: Implements logic around `create`, `getDest`, `erase`; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `create`, `getDest`, `erase` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 382-395
```cpp
  // Replace the original `async.execute` with a call to outlined function.
  {
    ImplicitLocOpBuilder callBuilder(loc, execute);
    auto callOutlinedFunc = func::CallOp::create(callBuilder, func.getName(),
                                                 execute.getResultTypes(),
                                                 functionInputs.getArrayRef());
    execute.replaceAllUsesWith(callOutlinedFunc.getResults());
    execute.erase();
  }

  return {func, coro};
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `callBuilder`, `create`, `getResultTypes`, `getArrayRef`, and 2 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `callBuilder`, `create`, `getResultTypes`, `getArrayRef`, and 2 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 396-413
```cpp
// Convert async.create_group operation to async.runtime.create_group
//===----------------------------------------------------------------------===//

namespace {
class CreateGroupOpLowering : public OpConversionPattern<CreateGroupOp> {
public:
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(CreateGroupOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    rewriter.replaceOpWithNewOp<RuntimeCreateGroupOp>(
        op, GroupType::get(op->getContext()), adaptor.getOperands());
    return success();
  }
};
} // namespace

```
- **EN**: Introduces declarations for `CreateGroupOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CreateGroupOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 414-432
```cpp
//===----------------------------------------------------------------------===//
// Convert async.add_to_group operation to async.runtime.add_to_group.
//===----------------------------------------------------------------------===//

namespace {
class AddToGroupOpLowering : public OpConversionPattern<AddToGroupOp> {
public:
  using OpConversionPattern::OpConversionPattern;

  LogicalResult
  matchAndRewrite(AddToGroupOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    rewriter.replaceOpWithNewOp<RuntimeAddToGroupOp>(
        op, rewriter.getIndexType(), adaptor.getOperands());
    return success();
  }
};
} // namespace

```
- **EN**: Introduces declarations for `AddToGroupOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AddToGroupOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 433-448
```cpp
//===----------------------------------------------------------------------===//
// Convert async.func, async.return and async.call operations to non-blocking
// operations based on llvm coroutine
//===----------------------------------------------------------------------===//

namespace {

//===----------------------------------------------------------------------===//
// Convert async.func operation to func.func
//===----------------------------------------------------------------------===//

class AsyncFuncOpLowering : public OpConversionPattern<async::FuncOp> {
public:
  AsyncFuncOpLowering(MLIRContext *ctx, FuncCoroMapPtr coros)
      : OpConversionPattern<async::FuncOp>(ctx), coros(std::move(coros)) {}

```
- **EN**: Introduces declarations for `AsyncFuncOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncFuncOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 449-464
```cpp
  LogicalResult
  matchAndRewrite(async::FuncOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();

    auto newFuncOp =
        func::FuncOp::create(rewriter, loc, op.getName(), op.getFunctionType());

    SymbolTable::setSymbolVisibility(newFuncOp,
                                     SymbolTable::getSymbolVisibility(op));
    // Copy over all attributes other than the name.
    for (const auto &namedAttr : op->getAttrs()) {
      if (namedAttr.getName() != SymbolTable::getSymbolAttrName())
        newFuncOp->setAttr(namedAttr.getName(), namedAttr.getValue());
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `create`, `setSymbolVisibility`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `create`, `setSymbolVisibility`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 465-479
```cpp
    rewriter.inlineRegionBefore(op.getBody(), newFuncOp.getBody(),
                                newFuncOp.end());

    CoroMachinery coro = setupCoroMachinery(newFuncOp);
    (*coros)[newFuncOp] = coro;
    // no initial suspend, we should hot-start

    rewriter.eraseOp(op);
    return success();
  }

private:
  FuncCoroMapPtr coros;
};

```
- **EN**: Implements logic around `inlineRegionBefore`, `end`, `setupCoroMachinery`, `eraseOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `inlineRegionBefore`, `end`, `setupCoroMachinery`, `eraseOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 480-497
```cpp
//===----------------------------------------------------------------------===//
// Convert async.call operation to func.call
//===----------------------------------------------------------------------===//

class AsyncCallOpLowering : public OpConversionPattern<async::CallOp> {
public:
  AsyncCallOpLowering(MLIRContext *ctx)
      : OpConversionPattern<async::CallOp>(ctx) {}

  LogicalResult
  matchAndRewrite(async::CallOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    rewriter.replaceOpWithNewOp<func::CallOp>(
        op, op.getCallee(), op.getResultTypes(), op.getOperands());
    return success();
  }
};

```
- **EN**: Introduces declarations for `AsyncCallOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncCallOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 498-515
```cpp
//===----------------------------------------------------------------------===//
// Convert async.return operation to async.runtime operations.
//===----------------------------------------------------------------------===//

class AsyncReturnOpLowering : public OpConversionPattern<async::ReturnOp> {
public:
  AsyncReturnOpLowering(MLIRContext *ctx, FuncCoroMapPtr coros)
      : OpConversionPattern<async::ReturnOp>(ctx), coros(std::move(coros)) {}

  LogicalResult
  matchAndRewrite(async::ReturnOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto func = op->template getParentOfType<func::FuncOp>();
    auto funcCoro = coros->find(func);
    if (funcCoro == coros->end())
      return rewriter.notifyMatchFailure(
          op, "operation is not inside the async coroutine function");

```
- **EN**: Introduces declarations for `AsyncReturnOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AsyncReturnOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 516-532
```cpp
    Location loc = op->getLoc();
    const CoroMachinery &coro = funcCoro->getSecond();
    rewriter.setInsertionPointAfter(op);

    // Store return values into the async values storage and switch async
    // values state to available.
    for (auto tuple : llvm::zip(adaptor.getOperands(), coro.returnValues)) {
      Value returnValue = std::get<0>(tuple);
      Value asyncValue = std::get<1>(tuple);
      RuntimeStoreOp::create(rewriter, loc, returnValue, asyncValue);
      RuntimeSetAvailableOp::create(rewriter, loc, asyncValue);
    }

    if (coro.asyncToken)
      // Switch the coroutine completion token to available state.
      RuntimeSetAvailableOp::create(rewriter, loc, *coro.asyncToken);

```
- **EN**: Implements logic around `getLoc`, `getSecond`, `setInsertionPointAfter`, `zip`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `getSecond`, `setInsertionPointAfter`, `zip`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 533-546
```cpp
    rewriter.eraseOp(op);
    cf::BranchOp::create(rewriter, loc, coro.cleanup);
    return success();
  }

private:
  FuncCoroMapPtr coros;
};
} // namespace

//===----------------------------------------------------------------------===//
// Convert async.await and async.await_all operations to the async.runtime.await
// or async.runtime.await_and_resume operations.
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `eraseOp`, `create`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `eraseOp`, `create`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 547-566
```cpp

namespace {
template <typename AwaitType, typename AwaitableType>
class AwaitOpLoweringBase : public OpConversionPattern<AwaitType> {
  using AwaitAdaptor = typename AwaitType::Adaptor;

public:
  AwaitOpLoweringBase(MLIRContext *ctx, FuncCoroMapPtr coros,
                      bool shouldLowerBlockingWait)
      : OpConversionPattern<AwaitType>(ctx), coros(std::move(coros)),
        shouldLowerBlockingWait(shouldLowerBlockingWait) {}

  LogicalResult
  matchAndRewrite(AwaitType op, typename AwaitType::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    // We can only await on one the `AwaitableType` (for `await` it can be
    // a `token` or a `value`, for `await_all` it must be a `group`).
    if (!isa<AwaitableType>(op.getOperand().getType()))
      return rewriter.notifyMatchFailure(op, "unsupported awaitable type");

```
- **EN**: Introduces declarations for `AwaitOpLoweringBase`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AwaitOpLoweringBase` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 567-580
```cpp
    // Check if await operation is inside the coroutine function.
    auto func = op->template getParentOfType<func::FuncOp>();
    auto funcCoro = coros->find(func);
    const bool isInCoroutine = funcCoro != coros->end();

    Location loc = op->getLoc();
    Value operand = adaptor.getOperand();

    Type i1 = rewriter.getI1Type();

    // Delay lowering to block wait in case await op is inside async.execute
    if (!isInCoroutine && !shouldLowerBlockingWait)
      return failure();

```
- **EN**: Implements logic around `FuncOp>`, `find`, `end`, `getLoc`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `FuncOp>`, `find`, `end`, `getLoc`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 581-597
```cpp
    // Inside regular functions we use the blocking wait operation to wait for
    // the async object (token, value or group) to become available.
    if (!isInCoroutine) {
      ImplicitLocOpBuilder builder(loc, rewriter);
      RuntimeAwaitOp::create(builder, loc, operand);

      // Assert that the awaited operands is not in the error state.
      Value isError = RuntimeIsErrorOp::create(builder, i1, operand);
      Value notError = arith::XOrIOp::create(
          builder, isError,
          arith::ConstantOp::create(builder, loc, i1,
                                    builder.getIntegerAttr(i1, 1)));

      cf::AssertOp::create(builder, notError,
                           "Awaited async operand is in error state");
    }

```
- **EN**: Implements logic around `builder`, `create`, `getIntegerAttr`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `builder`, `create`, `getIntegerAttr` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 598-612
```cpp
    // Inside the coroutine we convert await operation into coroutine suspension
    // point, and resume execution asynchronously.
    if (isInCoroutine) {
      CoroMachinery &coro = funcCoro->getSecond();
      Block *suspended = op->getBlock();

      ImplicitLocOpBuilder builder(loc, rewriter);
      MLIRContext *ctx = op->getContext();

      // Save the coroutine state and resume on a runtime managed thread when
      // the operand becomes available.
      auto coroSaveOp =
          CoroSaveOp::create(builder, CoroStateType::get(ctx), coro.coroHandle);
      RuntimeAwaitAndResumeOp::create(builder, operand, coro.coroHandle);

```
- **EN**: Implements logic around `getSecond`, `getBlock`, `builder`, `getContext`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getSecond`, `getBlock`, `builder`, `getContext`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 613-632
```cpp
      // Split the entry block before the await operation.
      Block *resume = rewriter.splitBlock(suspended, Block::iterator(op));

      // Add async.coro.suspend as a suspended block terminator.
      builder.setInsertionPointToEnd(suspended);
      CoroSuspendOp::create(builder, coroSaveOp.getState(), coro.suspend,
                            resume, coro.cleanupForDestroy);

      // Split the resume block into error checking and continuation.
      Block *continuation = rewriter.splitBlock(resume, Block::iterator(op));

      // Check if the awaited value is in the error state.
      builder.setInsertionPointToStart(resume);
      auto isError = RuntimeIsErrorOp::create(builder, loc, i1, operand);
      cf::CondBranchOp::create(builder, isError,
                               /*trueDest=*/setupSetErrorBlock(coro),
                               /*trueArgs=*/ArrayRef<Value>(),
                               /*falseDest=*/continuation,
                               /*falseArgs=*/ArrayRef<Value>());

```
- **EN**: Implements logic around `splitBlock`, `setInsertionPointToEnd`, `create`, `setInsertionPointToStart`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `splitBlock`, `setInsertionPointToEnd`, `create`, `setInsertionPointToStart`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 633-646
```cpp
      // Make sure that replacement value will be constructed in the
      // continuation block.
      rewriter.setInsertionPointToStart(continuation);
    }

    // Erase or replace the await operation with the new value.
    if (Value replaceWith = getReplacementValue(op, operand, rewriter))
      rewriter.replaceOp(op, replaceWith);
    else
      rewriter.eraseOp(op);

    return success();
  }

```
- **EN**: Implements logic around `setInsertionPointToStart`, `getReplacementValue`, `replaceOp`, `eraseOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `setInsertionPointToStart`, `getReplacementValue`, `replaceOp`, `eraseOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 647-660
```cpp
  virtual Value getReplacementValue(AwaitType op, Value operand,
                                    ConversionPatternRewriter &rewriter) const {
    return Value();
  }

private:
  FuncCoroMapPtr coros;
  bool shouldLowerBlockingWait;
};

/// Lowering for `async.await` with a token operand.
class AwaitTokenOpLowering : public AwaitOpLoweringBase<AwaitOp, TokenType> {
  using Base = AwaitOpLoweringBase<AwaitOp, TokenType>;

```
- **EN**: Introduces declarations for `AwaitTokenOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AwaitTokenOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 661-680
```cpp
public:
  using Base::Base;
};

/// Lowering for `async.await` with a value operand.
class AwaitValueOpLowering : public AwaitOpLoweringBase<AwaitOp, ValueType> {
  using Base = AwaitOpLoweringBase<AwaitOp, ValueType>;

public:
  using Base::Base;

  Value
  getReplacementValue(AwaitOp op, Value operand,
                      ConversionPatternRewriter &rewriter) const override {
    // Load from the async value storage.
    auto valueType = cast<ValueType>(operand.getType()).getValueType();
    return RuntimeLoadOp::create(rewriter, op->getLoc(), valueType, operand);
  }
};

```
- **EN**: Introduces declarations for `AwaitValueOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AwaitValueOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 681-694
```cpp
/// Lowering for `async.await_all` operation.
class AwaitAllOpLowering : public AwaitOpLoweringBase<AwaitAllOp, GroupType> {
  using Base = AwaitOpLoweringBase<AwaitAllOp, GroupType>;

public:
  using Base::Base;
};

} // namespace

//===----------------------------------------------------------------------===//
// Convert async.yield operation to async.runtime operations.
//===----------------------------------------------------------------------===//

```
- **EN**: Introduces declarations for `AwaitAllOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AwaitAllOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 695-709
```cpp
class YieldOpLowering : public OpConversionPattern<async::YieldOp> {
public:
  YieldOpLowering(MLIRContext *ctx, FuncCoroMapPtr coros)
      : OpConversionPattern<async::YieldOp>(ctx), coros(std::move(coros)) {}

  LogicalResult
  matchAndRewrite(async::YieldOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    // Check if yield operation is inside the async coroutine function.
    auto func = op->template getParentOfType<func::FuncOp>();
    auto funcCoro = coros->find(func);
    if (funcCoro == coros->end())
      return rewriter.notifyMatchFailure(
          op, "operation is not inside the async coroutine function");

```
- **EN**: Introduces declarations for `YieldOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `YieldOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 710-725
```cpp
    Location loc = op->getLoc();
    const CoroMachinery &coro = funcCoro->getSecond();

    // Store yielded values into the async values storage and switch async
    // values state to available.
    for (auto tuple : llvm::zip(adaptor.getOperands(), coro.returnValues)) {
      Value yieldValue = std::get<0>(tuple);
      Value asyncValue = std::get<1>(tuple);
      RuntimeStoreOp::create(rewriter, loc, yieldValue, asyncValue);
      RuntimeSetAvailableOp::create(rewriter, loc, asyncValue);
    }

    if (coro.asyncToken)
      // Switch the coroutine completion token to available state.
      RuntimeSetAvailableOp::create(rewriter, loc, *coro.asyncToken);

```
- **EN**: Implements logic around `getLoc`, `getSecond`, `zip`, `get`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `getSecond`, `zip`, `get`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 726-739
```cpp
    cf::BranchOp::create(rewriter, loc, coro.cleanup);
    rewriter.eraseOp(op);

    return success();
  }

private:
  FuncCoroMapPtr coros;
};

//===----------------------------------------------------------------------===//
// Convert cf.assert operation to cf.cond_br into `set_error` block.
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `create`, `eraseOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `create`, `eraseOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 740-754
```cpp
class AssertOpLowering : public OpConversionPattern<cf::AssertOp> {
public:
  AssertOpLowering(MLIRContext *ctx, FuncCoroMapPtr coros)
      : OpConversionPattern<cf::AssertOp>(ctx), coros(std::move(coros)) {}

  LogicalResult
  matchAndRewrite(cf::AssertOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    // Check if assert operation is inside the async coroutine function.
    auto func = op->template getParentOfType<func::FuncOp>();
    auto funcCoro = coros->find(func);
    if (funcCoro == coros->end())
      return rewriter.notifyMatchFailure(
          op, "operation is not inside the async coroutine function");

```
- **EN**: Introduces declarations for `AssertOpLowering`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AssertOpLowering` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 755-769
```cpp
    Location loc = op->getLoc();
    CoroMachinery &coro = funcCoro->getSecond();

    Block *cont = rewriter.splitBlock(op->getBlock(), Block::iterator(op));
    rewriter.setInsertionPointToEnd(cont->getPrevNode());
    cf::CondBranchOp::create(rewriter, loc, adaptor.getArg(),
                             /*trueDest=*/cont,
                             /*trueArgs=*/ArrayRef<Value>(),
                             /*falseDest=*/setupSetErrorBlock(coro),
                             /*falseArgs=*/ArrayRef<Value>());
    rewriter.eraseOp(op);

    return success();
  }

```
- **EN**: Implements logic around `getLoc`, `getSecond`, `splitBlock`, `setInsertionPointToEnd`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getLoc`, `getSecond`, `splitBlock`, `setInsertionPointToEnd`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 770-783
```cpp
private:
  FuncCoroMapPtr coros;
};

//===----------------------------------------------------------------------===//
void AsyncToAsyncRuntimePass::runOnOperation() {
  ModuleOp module = getOperation();
  SymbolTable symbolTable(module);

  // Functions with coroutine CFG setups, which are results of outlining
  // `async.execute` body regions
  FuncCoroMapPtr coros =
      std::make_shared<llvm::DenseMap<func::FuncOp, CoroMachinery>>();

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `symbolTable`, `CoroMachinery>>`; this block participates in pass execution or pass construction; works with symbol tables or function-like operations.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `symbolTable`, `CoroMachinery>>` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并处理符号表或类函数操作。

### Lines 784-798
```cpp
  module.walk([&](ExecuteOp execute) {
    coros->insert(outlineExecuteOp(symbolTable, execute));
  });

  LLVM_DEBUG({
    llvm::dbgs() << "Outlined " << coros->size()
                 << " functions built from async.execute operations\n";
  });

  // Returns true if operation is inside the coroutine.
  auto isInCoroutine = [&](Operation *op) -> bool {
    auto parentFunc = op->getParentOfType<func::FuncOp>();
    return coros->contains(parentFunc);
  };

```
- **EN**: Implements logic around `walk`, `insert`, `dbgs`, `FuncOp>`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `walk`, `insert`, `dbgs`, `FuncOp>`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 799-812
```cpp
  // Lower async operations to async.runtime operations.
  MLIRContext *ctx = module->getContext();
  RewritePatternSet asyncPatterns(ctx);

  // Conversion to async runtime augments original CFG with the coroutine CFG,
  // and we have to make sure that structured control flow operations with async
  // operations in nested regions will be converted to branch-based control flow
  // before we add the coroutine basic blocks.
  populateSCFToControlFlowConversionPatterns(asyncPatterns);

  // Async lowering does not use type converter because it must preserve all
  // types for async.runtime operations.
  asyncPatterns.add<CreateGroupOpLowering, AddToGroupOpLowering>(ctx);

```
- **EN**: Implements logic around `getContext`, `asyncPatterns`, `populateSCFToControlFlowConversionPatterns`, `AddToGroupOpLowering>`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getContext`, `asyncPatterns`, `populateSCFToControlFlowConversionPatterns`, `AddToGroupOpLowering>` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 813-837
```cpp
  asyncPatterns
      .add<AwaitTokenOpLowering, AwaitValueOpLowering, AwaitAllOpLowering>(
          ctx, coros, /*should_lower_blocking_wait=*/true);

  // Lower assertions to conditional branches into error blocks.
  asyncPatterns.add<YieldOpLowering, AssertOpLowering>(ctx, coros);

  // All high level async operations must be lowered to the runtime operations.
  ConversionTarget runtimeTarget(*ctx);
  runtimeTarget.addLegalDialect<AsyncDialect, func::FuncDialect>();
  runtimeTarget.addIllegalOp<CreateGroupOp, AddToGroupOp>();
  runtimeTarget.addIllegalOp<ExecuteOp, AwaitOp, AwaitAllOp, async::YieldOp>();

  // Decide if structured control flow has to be lowered to branch-based CFG.
  runtimeTarget.addDynamicallyLegalDialect<scf::SCFDialect>([&](Operation *op) {
    auto walkResult = op->walk([&](Operation *nested) {
      bool isAsync = isa<async::AsyncDialect>(nested->getDialect());
      return isAsync && isInCoroutine(nested) ? WalkResult::interrupt()
                                              : WalkResult::advance();
    });
    return !walkResult.wasInterrupted();
  });
  runtimeTarget.addLegalOp<cf::AssertOp, arith::XOrIOp, arith::ConstantOp,
                           func::ConstantOp, cf::BranchOp, cf::CondBranchOp>();

```
- **EN**: Implements logic around `AwaitAllOpLowering>`, `AssertOpLowering>`, `runtimeTarget`, `FuncDialect>`, and 9 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `AwaitAllOpLowering>`, `AssertOpLowering>`, `runtimeTarget`, `FuncDialect>`, and 9 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 838-851
```cpp
  // Assertions must be converted to runtime errors inside async functions.
  runtimeTarget.addDynamicallyLegalOp<cf::AssertOp>(
      [&](cf::AssertOp op) -> bool {
        auto func = op->getParentOfType<func::FuncOp>();
        return !coros->contains(func);
      });

  if (failed(applyPartialConversion(module, runtimeTarget,
                                    std::move(asyncPatterns)))) {
    signalPassFailure();
    return;
  }
}

```
- **EN**: Implements logic around `AssertOp>`, `FuncOp>`, `contains`, `failed`, and 2 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `AssertOp>`, `FuncOp>`, `contains`, `failed`, and 2 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 852-867
```cpp
//===----------------------------------------------------------------------===//
void mlir::populateAsyncFuncToAsyncRuntimeConversionPatterns(
    RewritePatternSet &patterns, ConversionTarget &target) {
  // Functions with coroutine CFG setups, which are results of converting
  // async.func.
  FuncCoroMapPtr coros =
      std::make_shared<llvm::DenseMap<func::FuncOp, CoroMachinery>>();
  MLIRContext *ctx = patterns.getContext();
  // Lower async.func to func.func with coroutine cfg.
  patterns.add<AsyncCallOpLowering>(ctx);
  patterns.add<AsyncFuncOpLowering, AsyncReturnOpLowering>(ctx, coros);

  patterns.add<AwaitTokenOpLowering, AwaitValueOpLowering, AwaitAllOpLowering>(
      ctx, coros, /*should_lower_blocking_wait=*/false);
  patterns.add<YieldOpLowering, AssertOpLowering>(ctx, coros);

```
- **EN**: Implements logic around `populateAsyncFuncToAsyncRuntimeConversionPatterns`, `CoroMachinery>>`, `getContext`, `add`, and 3 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `populateAsyncFuncToAsyncRuntimeConversionPatterns`, `CoroMachinery>>`, `getContext`, `add`, and 3 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 868-883
```cpp
  target.addDynamicallyLegalOp<AwaitOp, AwaitAllOp, YieldOp, cf::AssertOp>(
      [coros](Operation *op) {
        auto exec = op->getParentOfType<ExecuteOp>();
        auto func = op->getParentOfType<func::FuncOp>();
        return exec || !coros->contains(func);
      });
}

void AsyncFuncToAsyncRuntimePass::runOnOperation() {
  ModuleOp module = getOperation();

  // Lower async operations to async.runtime operations.
  MLIRContext *ctx = module->getContext();
  RewritePatternSet asyncPatterns(ctx);
  ConversionTarget runtimeTarget(*ctx);

```
- **EN**: Implements logic around `AssertOp>`, `getParentOfType`, `FuncOp>`, `contains`, and 5 more symbols; this block participates in pass execution or pass construction; works with symbol tables or function-like operations.
- **CN**: 围绕 `AssertOp>`, `getParentOfType`, `FuncOp>`, `contains`, and 5 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并处理符号表或类函数操作。

### Lines 884-899
```cpp
  // Lower async.func to func.func with coroutine cfg.
  populateAsyncFuncToAsyncRuntimeConversionPatterns(asyncPatterns,
                                                    runtimeTarget);

  runtimeTarget.addLegalDialect<AsyncDialect, func::FuncDialect>();
  runtimeTarget.addIllegalOp<async::FuncOp, async::CallOp, async::ReturnOp>();

  runtimeTarget.addLegalOp<arith::XOrIOp, arith::ConstantOp, func::ConstantOp,
                           cf::BranchOp, cf::CondBranchOp>();

  if (failed(applyPartialConversion(module, runtimeTarget,
                                    std::move(asyncPatterns)))) {
    signalPassFailure();
    return;
  }
}
```
- **EN**: Implements logic around `populateAsyncFuncToAsyncRuntimeConversionPatterns`, `FuncDialect>`, `ReturnOp>`, `CondBranchOp>`, and 3 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `populateAsyncFuncToAsyncRuntimeConversionPatterns`, `FuncDialect>`, `ReturnOp>`, `CondBranchOp>`, and 3 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Async/Passes.h`, `PassDetail.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Async/IR/Async.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h` ... (+4 more)
- **Standard-library headers / 标准库头文件**: `<utility>`, `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (7), MLIR IR core abstractions / MLIR IR 核心抽象 (2), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (2), dialect conversion infrastructure / 方言转换基础设施 (1), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
