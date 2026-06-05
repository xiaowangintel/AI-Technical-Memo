# AsyncRuntime.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/ExecutionEngine/AsyncRuntime.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR AsyncRuntime component. The leading comments describe it as: This file declares basic Async runtime API for supporting Async dialect.
- **用途（CN）**: 声明 MLIR AsyncRuntime 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===- AsyncRuntime.h - Async runtime reference implementation ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares basic Async runtime API for supporting Async dialect
// to LLVM dialect lowering.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_EXECUTIONENGINE_ASYNCRUNTIME_H_
#define MLIR_EXECUTIONENGINE_ASYNCRUNTIME_H_

#include <cstddef>
#include <stdint.h>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-20
````cpp
#ifdef _WIN32
````
- **EN**: This section focuses on #ifdef _win32, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“#ifdef _WIN32”这一主题，把相关声明与辅助接口组织在一起。

### Lines 21-29
````cpp
#ifndef MLIR_ASYNC_RUNTIME_EXPORT
#ifdef mlir_async_runtime_EXPORTS
// We are building this library
#define MLIR_ASYNC_RUNTIME_EXPORT __declspec(dllexport)
#else
// We are using this library
#define MLIR_ASYNC_RUNTIME_EXPORT __declspec(dllimport)
#endif // mlir_async_runtime_EXPORTS
#endif // MLIR_ASYNC_RUNTIME_EXPORT
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 30-33
````cpp
#else
// Non-windows: use visibility attributes.
#define MLIR_ASYNC_RUNTIME_EXPORT __attribute__((visibility("default")))
#endif // _WIN32
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 35-36
````cpp
namespace mlir {
namespace runtime {
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `AsyncRuntime`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `AsyncRuntime` API 表面的一部分。

### Lines 38-84
````cpp
//===----------------------------------------------------------------------===//
// Async runtime API.
//===----------------------------------------------------------------------===//

// Runtime implementation of `async.token` data type.
using AsyncToken = struct AsyncToken;

// Runtime implementation of `async.group` data type.
using AsyncGroup = struct AsyncGroup;

// Runtime implementation of `async.value` data type.
using AsyncValue = struct AsyncValue;

// Async value payload stored in a memory owned by the async.value.
using ValueStorage = std::byte *;

// Async runtime uses LLVM coroutines to represent asynchronous tasks. Task
// function is a coroutine handle and a resume function that continue coroutine
// execution from a suspension point.
using CoroHandle = void *;           // coroutine handle
using CoroResume = void (*)(void *); // coroutine resume function

// Async runtime uses reference counting to manage the lifetime of async values
// (values of async types like tokens, values and groups).
using RefCountedObjPtr = void *;

// Adds references to reference counted runtime object.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
    mlirAsyncRuntimeAddRef(RefCountedObjPtr, int64_t);

// Drops references from reference counted runtime object.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
    mlirAsyncRuntimeDropRef(RefCountedObjPtr, int64_t);

// Create a new `async.token` in not-ready state.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT AsyncToken *mlirAsyncRuntimeCreateToken();

// Create a new `async.value` in not-ready state. Size parameter specifies the
// number of bytes that will be allocated for the async value storage. Storage
// is owned by the `async.value` and deallocated when the async value is
// destructed (reference count drops to zero).
extern "C" MLIR_ASYNC_RUNTIME_EXPORT AsyncValue *
    mlirAsyncRuntimeCreateValue(int64_t);

// Create a new `async.group` in empty state.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT AsyncGroup *
mlirAsyncRuntimeCreateGroup(int64_t size);
````
- **EN**: This block groups callable interfaces such as `void`, `mlirAsyncRuntimeAddRef`, `mlirAsyncRuntimeDropRef`, `mlirAsyncRuntimeCreateToken`, indicating how `AsyncRuntime` is queried or updated.
- **CN**: 该代码块聚合了 `void`, `mlirAsyncRuntimeAddRef`, `mlirAsyncRuntimeDropRef`, `mlirAsyncRuntimeCreateToken` 等可调用接口，展示了如何查询或更新 `AsyncRuntime`。

### Lines 85-131
````cpp
extern "C" MLIR_ASYNC_RUNTIME_EXPORT int64_t
mlirAsyncRuntimeAddTokenToGroup(AsyncToken *, AsyncGroup *);

// Switches `async.token` to ready state and runs all awaiters.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeEmplaceToken(AsyncToken *);

// Switches `async.value` to ready state and runs all awaiters.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeEmplaceValue(AsyncValue *);

// Switches `async.token` to error state and runs all awaiters.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeSetTokenError(AsyncToken *);

// Switches `async.value` to error state and runs all awaiters.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeSetValueError(AsyncValue *);

// Returns true if token is in the error state.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT bool
mlirAsyncRuntimeIsTokenError(AsyncToken *);

// Returns true if value is in the error state.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT bool
mlirAsyncRuntimeIsValueError(AsyncValue *);

// Returns true if group is in the error state (any of the tokens or values
// added to the group are in the error state).
extern "C" MLIR_ASYNC_RUNTIME_EXPORT bool
mlirAsyncRuntimeIsGroupError(AsyncGroup *);

// Blocks the caller thread until the token becomes ready.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeAwaitToken(AsyncToken *);

// Blocks the caller thread until the value becomes ready.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeAwaitValue(AsyncValue *);

// Blocks the caller thread until the elements in the group become ready.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeAwaitAllInGroup(AsyncGroup *);

// Returns a pointer to the storage owned by the async value.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT ValueStorage
mlirAsyncRuntimeGetValueStorage(AsyncValue *);
````
- **EN**: This block groups callable interfaces such as `mlirAsyncRuntimeAddTokenToGroup`, `mlirAsyncRuntimeEmplaceToken`, `mlirAsyncRuntimeEmplaceValue`, `mlirAsyncRuntimeSetTokenError`, indicating how `AsyncRuntime` is queried or updated.
- **CN**: 该代码块聚合了 `mlirAsyncRuntimeAddTokenToGroup`, `mlirAsyncRuntimeEmplaceToken`, `mlirAsyncRuntimeEmplaceValue`, `mlirAsyncRuntimeSetTokenError` 等可调用接口，展示了如何查询或更新 `AsyncRuntime`。

### Lines 133-155
````cpp
// Executes the task (coro handle + resume function) in one of the threads
// managed by the runtime.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void mlirAsyncRuntimeExecute(CoroHandle,
                                                                  CoroResume);

// Executes the task (coro handle + resume function) in one of the threads
// managed by the runtime after the token becomes ready.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeAwaitTokenAndExecute(AsyncToken *, CoroHandle, CoroResume);

// Executes the task (coro handle + resume function) in one of the threads
// managed by the runtime after the value becomes ready.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeAwaitValueAndExecute(AsyncValue *, CoroHandle, CoroResume);

// Executes the task (coro handle + resume function) in one of the threads
// managed by the runtime after the all members of the group become ready.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimeAwaitAllInGroupAndExecute(AsyncGroup *, CoroHandle, CoroResume);

// Returns the current number of available worker threads in the threadpool.
extern "C" MLIR_ASYNC_RUNTIME_EXPORT int64_t
mlirAsyncRuntimGetNumWorkerThreads();
````
- **EN**: This block groups callable interfaces such as `mlirAsyncRuntimeExecute`, `mlirAsyncRuntimeAwaitTokenAndExecute`, `mlirAsyncRuntimeAwaitValueAndExecute`, `mlirAsyncRuntimeAwaitAllInGroupAndExecute`, indicating how `AsyncRuntime` is queried or updated.
- **CN**: 该代码块聚合了 `mlirAsyncRuntimeExecute`, `mlirAsyncRuntimeAwaitTokenAndExecute`, `mlirAsyncRuntimeAwaitValueAndExecute`, `mlirAsyncRuntimeAwaitAllInGroupAndExecute` 等可调用接口，展示了如何查询或更新 `AsyncRuntime`。

### Lines 158-166
````cpp
//===----------------------------------------------------------------------===//
// Small async runtime support library for testing.
//===----------------------------------------------------------------------===//

extern "C" MLIR_ASYNC_RUNTIME_EXPORT void
mlirAsyncRuntimePrintCurrentThreadId();

} // namespace runtime
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `mlirAsyncRuntimePrintCurrentThreadId`, indicating how `AsyncRuntime` is queried or updated.
- **CN**: 该代码块聚合了 `mlirAsyncRuntimePrintCurrentThreadId` 等可调用接口，展示了如何查询或更新 `AsyncRuntime`。

### Lines 168-168
````cpp
#endif // MLIR_EXECUTIONENGINE_ASYNCRUNTIME_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层

## Dependencies / 依赖关系

- Primarily relies on nearby MLIR declarations surrounding AsyncRuntime.h.
