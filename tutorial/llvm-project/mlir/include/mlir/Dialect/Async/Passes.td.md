# Passes.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Async/Passes.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the Async dialect, focused on public header declarations and `Passes`.
  - **CN**: 为 Async 方言定义聚焦 `Passes` 的声明式 TableGen 规格，覆盖公共头文件声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- Passes.td - Async pass definition file -------------*- tablegen -*-===//
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
```tablegen

#ifndef MLIR_DIALECT_ASYNC_PASSES
#define MLIR_DIALECT_ASYNC_PASSES

include "mlir/Pass/PassBase.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 14-23
```tablegen
def AsyncParallelForPass : Pass<"async-parallel-for", "ModuleOp"> {
  let summary = "Convert scf.parallel operations to multiple async compute ops "
                "executed concurrently for non-overlapping iteration ranges";

  let options = [
    Option<"asyncDispatch", "async-dispatch",
      "bool", /*default=*/"true",
      "Dispatch async compute tasks using recursive work splitting. If `false` "
      "async compute tasks will be launched using simple for loop in the "
      "caller thread.">,
```
- **EN**: Introduces declarations for `AsyncParallelForPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncParallelForPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 24-29
```tablegen

    Option<"numWorkerThreads", "num-workers",
      "int32_t", /*default=*/"8",
      "The number of available workers to execute async operations. If `-1` "
      "the value will be retrieved from the runtime.">,

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 30-34
```tablegen
    Option<"minTaskSize", "min-task-size",
      "int32_t", /*default=*/"1000",
      "The minimum task size for sharding parallel operation.">
  ];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 35-42
```tablegen
  let dependentDialects = [
    "arith::ArithDialect",
    "async::AsyncDialect",
    "func::FuncDialect",
    "scf::SCFDialect"
  ];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 43-48
```tablegen
def AsyncToAsyncRuntimePass : Pass<"async-to-async-runtime", "ModuleOp"> {
  let summary = "Lower all high level async operations (e.g. async.execute) to"
                "the explicit async.runtime and async.coro operations";
  let dependentDialects = ["async::AsyncDialect", "func::FuncDialect", "cf::ControlFlowDialect"];
}

```
- **EN**: Introduces declarations for `AsyncToAsyncRuntimePass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncToAsyncRuntimePass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 49-55
```tablegen
def AsyncFuncToAsyncRuntimePass
    : Pass<"async-func-to-async-runtime", "ModuleOp"> {
  let summary = "Lower async.func operations to the explicit async.runtime and"
                "async.coro operations";
  let dependentDialects = ["async::AsyncDialect", "func::FuncDialect"];
}

```
- **EN**: Introduces declarations for `AsyncFuncToAsyncRuntimePass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncFuncToAsyncRuntimePass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 56-62
```tablegen
def AsyncRuntimeRefCountingPass : Pass<"async-runtime-ref-counting"> {
  let summary = "Automatic reference counting for Async runtime operations";
  let description = [{
    This pass works at the async runtime abtraction level, after all
    `async.execute` and `async.await` operations are lowered to the async
    runtime API calls, and async coroutine operations.

```
- **EN**: Introduces declarations for `AsyncRuntimeRefCountingPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncRuntimeRefCountingPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 63-68
```tablegen
    It relies on the LLVM coroutines switched-resume lowering semantics for
    the correct placing of the reference counting operations.

    See: https://llvm.org/docs/Coroutines.html#switched-resume-lowering
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 69-75
```tablegen
  let dependentDialects = ["async::AsyncDialect"];
}

def AsyncRuntimeRefCountingOptPass : Pass<"async-runtime-ref-counting-opt"> {
  let summary = "Optimize automatic reference counting operations for the"
                "Async runtime by removing redundant operations";

```
- **EN**: Introduces declarations for `AsyncRuntimeRefCountingOptPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncRuntimeRefCountingOptPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 76-85
```tablegen
  let dependentDialects = ["async::AsyncDialect"];
}

def AsyncRuntimePolicyBasedRefCountingPass
    : Pass<"async-runtime-policy-based-ref-counting"> {
  let summary = "Policy based reference counting for Async runtime operations";
  let description = [{
    This pass works at the async runtime abtraction level, after all
    `async.execute` and `async.await` operations are lowered to the async
    runtime API calls, and async coroutine operations.
```
- **EN**: Introduces declarations for `AsyncRuntimePolicyBasedRefCountingPass`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncRuntimePolicyBasedRefCountingPass` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 86-91
```tablegen

    This pass doesn't rely on reference counted values liveness analysis, and
    instead uses simple policy to create reference counting operations. If the
    program violates any of the assumptions, then this pass might lead to
    memory leaks or runtime errors.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 92-101
```tablegen
    The default reference counting policy assumptions:
      1. Async token can be awaited or added to the group only once.
      2. Async value or group can be awaited only once.

    Under these assumptions reference counting only needs to drop reference:
      1. After `async.runtime.await` operation for async tokens and groups
         (until error handling is not implemented for the sync await).
      2. After `async.runtime.is_error` operation for async tokens and groups
         (this is the last operation in the coroutine resume function).
      3. After `async.runtime.load` operation for async values.
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 102-106
```tablegen

    This pass introduces significanly less runtime overhead compared to the
    automatic reference counting.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 107-110
```tablegen
  let dependentDialects = ["async::AsyncDialect"];
}

#endif // MLIR_DIALECT_ASYNC_PASSES
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Pass/PassBase.td`
- **Subsystem categories / 子系统类别**: pass infrastructure and registration support / Pass 基础设施与注册支持 (1)
