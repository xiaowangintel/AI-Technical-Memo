# AsyncOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Async/IR/AsyncOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This is the operation definition file for Async dialect operations.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/Async/IR`，围绕 Async 方言公开 `AsyncOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- AsyncOps.td - Async operations definition -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is the operation definition file for Async dialect operations.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-25
```tablegen

#ifndef ASYNC_OPS
#define ASYNC_OPS

include "mlir/Dialect/Async/IR/AsyncDialect.td"
include "mlir/Dialect/Async/IR/AsyncTypes.td"
include "mlir/Interfaces/ControlFlowInterfaces.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/Interfaces/CallInterfaces.td"
include "mlir/IR/SymbolInterfaces.td"
include "mlir/Interfaces/FunctionInterfaces.td"
include "mlir/IR/OpAsmInterface.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 26-35
```tablegen

//===----------------------------------------------------------------------===//
// Async op definitions
//===----------------------------------------------------------------------===//

// Base class for the operation in this dialect
class Async_Op<string mnemonic, list<Trait> traits = []> :
    Op<AsyncDialect, mnemonic, traits>;

def Async_ExecuteOp :
```
- **EN**: Introduces declarations for `for`, `Async_Op`, `Async_ExecuteOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `for`, `Async_Op`, `Async_ExecuteOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 36-49
```tablegen
  Async_Op<"execute", [SingleBlockImplicitTerminator<"YieldOp">,
                       DeclareOpInterfaceMethods<RegionBranchOpInterface,
                                                 ["getEntrySuccessorOperands",
                                                  "getSuccessorInputs",
                                                  "areTypesCompatible"]>,
                       AttrSizedOperandSegments,
                       AutomaticAllocationScope,
                       RecursiveMemoryEffects]> {
  let summary = "Asynchronous execute operation";
  let description = [{
    The `body` region attached to the `async.execute` operation semantically
    can be executed concurrently with the successor operation. In the followup
    example "compute0" can be executed concurrently with "compute1".

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 50-62
```tablegen
    The actual concurrency semantics depends on the dialect lowering to the
    executable format. Fully sequential execution ("compute0" completes before
    "compute1" starts) is a completely legal execution.

    Because concurrent execution is not guaranteed, it is illegal to create an
    implicit dependency from "compute1" to "compute0" (e.g. via shared global
    state). All dependencies must be made explicit with async execute arguments
    (`async.token` or `async.value`).

   `async.execute` operation takes `async.token` dependencies and `async.value`
    operands separately, and starts execution of the attached body region only
    when all tokens and values become ready.

```
- **EN**: Implements logic around `execution`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `execution` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性。

### Lines 63-76
```tablegen
    Example:

    ```mlir
    %dependency = ... : !async.token
    %value = ... : !async.value<f32>

    %token, %results =
      async.execute [%dependency](%value as %unwrapped: !async.value<f32>)
                 -> !async.value<!some.type>
      {
        %0 = "compute0"(%unwrapped): (f32) -> !some.type
        async.yield %0 : !some.type
      }

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 77-87
```tablegen
    %1 = "compute1"(...) : !some.type
    ```

    In the example above asynchronous execution starts only after dependency
    token and value argument become ready. Unwrapped value passed to the
    attached body region as an %unwrapped value of f32 type.
  }];

  let arguments = (ins Variadic<Async_TokenType>:$dependencies,
                       Variadic<Async_AnyValueOrTokenType>:$bodyOperands);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 88-101
```tablegen
  let results = (outs Async_TokenType:$token,
                      Variadic<Async_ValueType>:$bodyResults);
  let regions = (region SizedRegion<1>:$bodyRegion);

  let hasCustomAssemblyFormat = 1;
  let skipDefaultBuilders = 1;
  let hasRegionVerifier = 1;
  let builders = [
    OpBuilder<(ins "TypeRange":$resultTypes, "ValueRange":$dependencies,
      "ValueRange":$operands,
      CArg<"function_ref<void(OpBuilder &, Location, ValueRange)>",
           "nullptr">:$bodyBuilder)>,
  ];

```
- **EN**: Declares APIs or declarative rules around `OpBuilder`, `function_ref`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `OpBuilder`, `function_ref` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 102-119
```tablegen
  let extraClassDeclaration = [{
    using BodyBuilderFn =
        function_ref<void(OpBuilder &, Location, ValueRange)>;

  }];
}

def Async_FuncOp : Async_Op<"func",
    [FunctionOpInterface, IsolatedFromAbove, OpAsmOpInterface]> {
  let summary = "async function operation";
  let description = [{
    An async function is like a normal function, but supports non-blocking
    await. Internally, async function is lowered to the LLVM coroutinue with
    async runtime intrinsic. It can return an async token and/or async values.
    The token represents the execution state of async function and can be used
    when users want to express dependencies on some side effects, e.g.,
    the token becomes available once every thing in the func body is executed.

```
- **EN**: Introduces declarations for `Async_FuncOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_FuncOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 120-133
```tablegen
    Example:

    ```mlir
    // Async function can't return void, it always must be some async thing.
    async.func @async.0() -> !async.token {
      return
    }

    // Function returns only async value.
    async.func @async.1() -> !async.value<i32> {
      %0 = arith.constant 42 : i32
      return %0 : i32
    }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 134-147
```tablegen
    // Implicit token can be added to return types.
    async.func @async.2() -> !async.token, !async.value<i32> {
      %0 = arith.constant 42 : i32
      return %0 : i32
    }
    ```
  }];

  let arguments = (ins SymbolNameAttr:$sym_name,
                       TypeAttrOf<FunctionType>:$function_type,
                       OptionalAttr<StrAttr>:$sym_visibility,
                       OptionalAttr<DictArrayAttr>:$arg_attrs,
                       OptionalAttr<DictArrayAttr>:$res_attrs);

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 148-157
```tablegen
  let regions = (region AnyRegion:$body);

  let builders = [
    OpBuilder<(ins "StringRef":$name, "FunctionType":$type,
      CArg<"ArrayRef<NamedAttribute>", "{}">:$attrs,
      CArg<"ArrayRef<DictionaryAttr>", "{}">:$argAttrs)>
  ];

  let extraClassDeclaration = [{
    //===------------------------------------------------------------------===//
```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 158-167
```tablegen
    // FunctionOpInterface Methods
    //===------------------------------------------------------------------===//

    /// Returns the region on the current operation that is callable. This may
    /// return null in the case of an external callable object, e.g. an external
    /// function.
    ::mlir::Region *getCallableRegion() { return isExternal() ? nullptr
                                                              : &getBody(); }

    /// Returns the argument types of this async function.
```
- **EN**: Implements logic around `getCallableRegion`, `getBody`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getCallableRegion`, `getBody` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 168-178
```tablegen
    ArrayRef<Type> getArgumentTypes() { return getFunctionType().getInputs(); }

    /// Returns the result types of this async function.
    ArrayRef<Type> getResultTypes() { return getFunctionType().getResults(); }

    /// Returns the number of results of this async function
    unsigned getNumResults() {return getResultTypes().size();}

    /// Is the async func stateful
    bool isStateful() { return isa<TokenType>(getFunctionType().getResult(0));}

```
- **EN**: Implements logic around `getArgumentTypes`, `getResultTypes`, `getNumResults`, `isStateful`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getArgumentTypes`, `getResultTypes`, `getNumResults`, `isStateful` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 179-188
```tablegen
    //===------------------------------------------------------------------===//
    // OpAsmOpInterface Methods
    //===------------------------------------------------------------------===//

    /// Allow the dialect prefix to be omitted.
    static StringRef getDefaultDialect() { return "async"; }

    //===------------------------------------------------------------------===//
    // SymbolOpInterface Methods
    //===------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getDefaultDialect`.
- **CN**: 围绕 `getDefaultDialect` 实现具体逻辑。

### Lines 189-205
```tablegen

    bool isDeclaration() { return isExternal(); }
  }];
  let hasCustomAssemblyFormat = 1;

  let hasVerifier = 1;
}

def Async_CallOp : Async_Op<"call",
    [CallOpInterface, DeclareOpInterfaceMethods<SymbolUserOpInterface>]> {
  let summary = "async call operation";
  let description = [{
    The `async.call` operation represents a direct call to an async function
    that is within the same symbol scope as the call. The operands and result
    types of the call must match the specified async function type. The callee
    is encoded as a symbol reference attribute named "callee".

```
- **EN**: Introduces declarations for `Async_CallOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CallOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 206-219
```tablegen
    Example:

    ```mlir
    %2 = async.call @my_add(%0, %1) : (f32, f32) -> !async.value<f32>
    ```
  }];

  let arguments = (ins
    FlatSymbolRefAttr:$callee,
    Variadic<AnyType>:$operands,
    OptionalAttr<DictArrayAttr>:$arg_attrs,
    OptionalAttr<DictArrayAttr>:$res_attrs
  );

```
- **EN**: Declares APIs or declarative rules around `my_add`.
- **CN**: 声明与 `my_add` 相关的 API 或声明式规则。

### Lines 220-239
```tablegen
  let results = (outs Variadic<Async_AnyValueOrTokenType>);

  let builders = [
    OpBuilder<(ins "FuncOp":$callee, CArg<"ValueRange", "{}">:$operands), [{
      $_state.addOperands(operands);
      $_state.addAttribute("callee", SymbolRefAttr::get(callee));
      $_state.addTypes(callee.getFunctionType().getResults());
    }]>,
    OpBuilder<(ins "SymbolRefAttr":$callee, "TypeRange":$results,
      CArg<"ValueRange", "{}">:$operands), [{
      $_state.addOperands(operands);
      $_state.addAttribute("callee", callee);
      $_state.addTypes(results);
    }]>,
    OpBuilder<(ins "StringAttr":$callee, "TypeRange":$results,
      CArg<"ValueRange", "{}">:$operands), [{
      build($_builder, $_state, SymbolRefAttr::get(callee), results, operands);
    }]>,
    OpBuilder<(ins "StringRef":$callee, "TypeRange":$results,
      CArg<"ValueRange", "{}">:$operands), [{
```
- **EN**: Implements logic around `OpBuilder`, `addOperands`, `addAttribute`, `addTypes`, and 1 more symbols.
- **CN**: 围绕 `OpBuilder`, `addOperands`, `addAttribute`, `addTypes`, and 1 more symbols 实现具体逻辑。

### Lines 240-252
```tablegen
      build($_builder, $_state, StringAttr::get($_builder.getContext(), callee),
            results, operands);
    }]>
  ];

  let extraClassDeclaration = [{
    FunctionType getCalleeType();

    /// Get the argument operands to the called function.
    operand_range getArgOperands() {
      return {arg_operand_begin(), arg_operand_end()};
    }

```
- **EN**: Implements logic around `build`, `getCalleeType`, `getArgOperands`, `arg_operand_begin`.
- **CN**: 围绕 `build`, `getCalleeType`, `getArgOperands`, `arg_operand_begin` 实现具体逻辑。

### Lines 253-264
```tablegen
    MutableOperandRange getArgOperandsMutable() {
      return getOperandsMutable();
    }

    operand_iterator arg_operand_begin() { return operand_begin(); }
    operand_iterator arg_operand_end() { return operand_end(); }

    /// Return the callee of this operation.
    CallInterfaceCallable getCallableForCallee() {
      return (*this)->getAttrOfType<SymbolRefAttr>("callee");
    }

```
- **EN**: Implements logic around `getArgOperandsMutable`, `getOperandsMutable`, `arg_operand_begin`, `arg_operand_end`, and 2 more symbols.
- **CN**: 围绕 `getArgOperandsMutable`, `getOperandsMutable`, `arg_operand_begin`, `arg_operand_end`, and 2 more symbols 实现具体逻辑。

### Lines 265-275
```tablegen
    /// Set the callee for this operation.
    void setCalleeFromCallable(CallInterfaceCallable callee) {
      (*this)->setAttr("callee", cast<SymbolRefAttr>(callee));
    }
  }];

  let assemblyFormat = [{
    $callee `(` $operands `)` attr-dict `:` functional-type($operands, results)
  }];
}

```
- **EN**: Implements logic around `setCalleeFromCallable`, `setAttr`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setCalleeFromCallable`, `setAttr`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 276-290
```tablegen
def Async_ReturnOp : Async_Op<"return",
    [Pure, HasParent<"FuncOp">, ReturnLike, Terminator]> {
  let summary = "Async function return operation";
  let description = [{
    The `async.return` is a special terminator operation for Async function.

    Example:

    ```mlir
    async.func @foo() : !async.token {
      return
    }
    ```
  }];

```
- **EN**: Introduces declarations for `Async_ReturnOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_ReturnOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 291-307
```tablegen
  let arguments = (ins Variadic<AnyType>:$operands);

  let builders = [OpBuilder<(ins), [{build($_builder, $_state, {});}]>];

  let assemblyFormat = "attr-dict ($operands^ `:` type($operands))?";
  let hasVerifier = 1;
}

def Async_YieldOp :
    Async_Op<"yield", [
      HasParent<"ExecuteOp">, Pure, Terminator, ReturnLike]> {
  let summary = "terminator for Async execute operation";
  let description = [{
    The `async.yield` is a special terminator operation for the block inside
    `async.execute` operation.
  }];

```
- **EN**: Introduces declarations for `Async_YieldOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_YieldOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 308-317
```tablegen
  let arguments = (ins Variadic<AnyType>:$operands);
  let assemblyFormat = "($operands^ `:` type($operands))? attr-dict";
}

def Async_AwaitOp : Async_Op<"await"> {
  let summary = "waits for the argument to become ready";
  let description = [{
    The `async.await` operation waits until the argument becomes ready, and for
    the `async.value` arguments it unwraps the underlying value

```
- **EN**: Introduces declarations for `Async_AwaitOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_AwaitOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 318-328
```tablegen
    Example:

    ```mlir
    %0 = ... : !async.token
    async.await %0 : !async.token

    %1 = ... : !async.value<f32>
    %2 = async.await %1 : !async.value<f32>
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 329-339
```tablegen
  let arguments = (ins Async_AnyValueOrTokenType:$operand);
  let results = (outs Optional<AnyType>:$result);

  let skipDefaultBuilders = 1;
  let hasVerifier = 1;

  let builders = [
    OpBuilder<(ins "Value":$operand,
      CArg<"ArrayRef<NamedAttribute>", "{}">:$attrs)>,
  ];

```
- **EN**: Implements logic around `OpBuilder`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 340-353
```tablegen
  let extraClassDeclaration = [{
    std::optional<Type> getResultType() {
      if (getResultTypes().empty()) return std::nullopt;
      return getResultTypes()[0];
    }
  }];

  let assemblyFormat = [{
    $operand `:` custom<AwaitResultType>(
      type($operand), type($result)
    ) attr-dict
  }];
}

```
- **EN**: Implements logic around `getResultType`, `getResultTypes`, `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getResultType`, `getResultTypes`, `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 354-363
```tablegen
def Async_CreateGroupOp : Async_Op<"create_group", [Pure]> {
  let summary = "creates an empty async group";
  let description = [{
    The `async.create_group` allocates an empty async group. Async tokens or
    values can be added to this group later. The size of the group must be
    specified at construction time, and `await_all` operation will first
    wait until the number of added tokens or values reaches the group size.

    Example:

```
- **EN**: Introduces declarations for `Async_CreateGroupOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CreateGroupOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 364-374
```tablegen
    ```mlir
    %size = ... : index
    %group = async.create_group %size : !async.group
    ...
    async.await_all %group
    ```
  }];

  let arguments = (ins Index:$size);
  let results = (outs Async_GroupType:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 375-386
```tablegen
  let hasCanonicalizeMethod = 1;

  let assemblyFormat = "$size `:` type($result) attr-dict";
}

def Async_AddToGroupOp : Async_Op<"add_to_group", []> {
  let summary = "adds an async token or value to the group";
  let description = [{
    The `async.add_to_group` adds an async token or value to the async group.
    Returns the rank of the added element in the group. This rank is fixed
    for the group lifetime.

```
- **EN**: Introduces declarations for `Async_AddToGroupOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_AddToGroupOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 387-399
```tablegen
    Example:

    ```mlir
    %0 = async.create_group %size : !async.group
    %1 = ... : !async.token
    %2 = async.add_to_group %1, %0 : !async.token
    ```
  }];

  let arguments = (ins Async_AnyValueOrTokenType:$operand,
                       Async_GroupType:$group);
  let results = (outs Index:$rank);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 400-409
```tablegen
  let assemblyFormat = "$operand `,` $group `:` type($operand) attr-dict";
}

def Async_AwaitAllOp : Async_Op<"await_all", []> {
  let summary = "waits for the all async tokens or values in the group to "
                "become ready";
  let description = [{
    The `async.await_all` operation waits until all the tokens or values in the
    group become ready.

```
- **EN**: Introduces declarations for `Async_AwaitAllOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_AwaitAllOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 410-420
```tablegen
    Example:

    ```mlir
    %0 = async.create_group %size : !async.group

    %1 = ... : !async.token
    %2 = async.add_to_group %1, %0 : !async.token

    %3 = ... : !async.token
    %4 = async.add_to_group %2, %0 : !async.token

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 421-430
```tablegen
    async.await_all %0
    ```
  }];

  let arguments = (ins Async_GroupType:$operand);
  let results = (outs);

  let assemblyFormat = "$operand attr-dict";
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 431-447
```tablegen
//===----------------------------------------------------------------------===//
// Async Dialect LLVM Coroutines Operations.
//===----------------------------------------------------------------------===//

// Async to LLVM dialect lowering converts async tasks (regions inside async
// execute operations) to LLVM coroutines [1], and relies on switched-resume
// lowering [2] to produce an asynchronous executable.
//
// We define LLVM coro intrinsics in the async dialect to facilitate progressive
// lowering with verifiable and type-safe IR during the multi-step lowering
// pipeline. First we convert from high level async operations (e.g. execute) to
// the explicit calls to coro intrinsics and runtime API, and then finalize
// lowering to LLVM with a simple dialect conversion pass.
//
// [1] https://llvm.org/docs/Coroutines.html
// [2] https://llvm.org/docs/Coroutines.html#switched-resume-lowering

```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 448-457
```tablegen
def Async_CoroIdOp : Async_Op<"coro.id"> {
  let summary = "returns a switched-resume coroutine identifier";
  let description = [{
    The `async.coro.id` returns a switched-resume coroutine identifier.
  }];

  let results = (outs Async_CoroIdType:$id);
  let assemblyFormat = "attr-dict";
}

```
- **EN**: Introduces declarations for `Async_CoroIdOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CoroIdOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 458-469
```tablegen
def Async_CoroBeginOp : Async_Op<"coro.begin"> {
  let summary = "returns a handle to the coroutine";
  let description = [{
    The `async.coro.begin` allocates a coroutine frame and returns a handle to
    the coroutine.
  }];

  let arguments = (ins Async_CoroIdType:$id);
  let results = (outs Async_CoroHandleType:$handle);
  let assemblyFormat = "$id attr-dict";
}

```
- **EN**: Introduces declarations for `Async_CoroBeginOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CoroBeginOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 470-481
```tablegen
def Async_CoroFreeOp : Async_Op<"coro.free"> {
  let summary = "deallocates the coroutine frame";
  let description = [{
    The `async.coro.free` deallocates the coroutine frame created by the
    async.coro.begin operation.
  }];

  let arguments = (ins Async_CoroIdType:$id,
                       Async_CoroHandleType:$handle);
  let assemblyFormat = "$id `,` $handle attr-dict";
}

```
- **EN**: Introduces declarations for `Async_CoroFreeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CoroFreeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 482-493
```tablegen
def Async_CoroEndOp : Async_Op<"coro.end"> {
  let summary = "marks the end of the coroutine in the suspend block";
  let description = [{
    The `async.coro.end` marks the point where a coroutine needs to return
    control back to the caller if it is not an initial invocation of the
    coroutine. It the start part of the coroutine is is no-op.
  }];

  let arguments = (ins Async_CoroHandleType:$handle);
  let assemblyFormat = "$handle attr-dict";
}

```
- **EN**: Introduces declarations for `Async_CoroEndOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CoroEndOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 494-504
```tablegen
def Async_CoroSaveOp : Async_Op<"coro.save"> {
  let summary = "saves the coroutine state";
  let description = [{
    The `async.coro.saves` saves the coroutine state.
  }];

  let arguments = (ins Async_CoroHandleType:$handle);
  let results = (outs Async_CoroStateType:$state);
  let assemblyFormat = "$handle attr-dict";
}

```
- **EN**: Introduces declarations for `Async_CoroSaveOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CoroSaveOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 505-517
```tablegen
def Async_CoroSuspendOp : Async_Op<"coro.suspend", [Terminator]> {
  let summary = "suspends the coroutine";
  let description = [{
    The `async.coro.suspend` suspends the coroutine and transfers control to the
    `suspend` successor. If suspended coroutine later resumed it will transfer
    control to the `resume` successor. If it is destroyed it will transfer
    control to the the `cleanup` successor.

    In switched-resume lowering coroutine can be already in resumed state when
    suspend operation is called, in this case control will be transferred to the
    `resume` successor skipping the `suspend` successor.
  }];

```
- **EN**: Introduces declarations for `Async_CoroSuspendOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_CoroSuspendOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 518-528
```tablegen
  let arguments = (ins Async_CoroStateType:$state);
  let successors = (successor AnySuccessor:$suspendDest,
                              AnySuccessor:$resumeDest,
                              AnySuccessor:$cleanupDest);
  let assemblyFormat =
    "$state `,` $suspendDest `,` $resumeDest  `,` $cleanupDest attr-dict";
}

//===----------------------------------------------------------------------===//
// Async Dialect Runtime Operations.
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 529-540
```tablegen

// The following operations are intermediate async dialect operations to help
// lowering from high level async operation like `async.execute` to the Async
// Runtime API defined in the `ExecutionEngine/AsyncRuntime.h`.

def Async_RuntimeCreateOp : Async_Op<"runtime.create"> {
  let summary = "creates an async runtime token or value";
  let description = [{
    The `async.runtime.create` operation creates an async dialect token or
    value. Tokens and values are created in the non-ready state.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeCreateOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeCreateOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 541-551
```tablegen
  let results = (outs Async_AnyValueOrTokenType:$result);
  let assemblyFormat = "attr-dict `:` type($result)";
}

def Async_RuntimeCreateGroupOp : Async_Op<"runtime.create_group"> {
  let summary = "creates an async runtime group";
  let description = [{
    The `async.runtime.create_group` operation creates an async dialect group
    of the given size. Group created in the empty state.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeCreateGroupOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeCreateGroupOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 552-563
```tablegen
  let arguments = (ins Index:$size);
  let results = (outs Async_GroupType:$result);
  let assemblyFormat = "$size `:` type($result) attr-dict ";
}

def Async_RuntimeSetAvailableOp : Async_Op<"runtime.set_available"> {
  let summary = "switches token or value to available state";
  let description = [{
    The `async.runtime.set_available` operation switches async token or value
    state to available.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeSetAvailableOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeSetAvailableOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 564-574
```tablegen
  let arguments = (ins Async_AnyValueOrTokenType:$operand);
  let assemblyFormat = "$operand attr-dict `:` type($operand)";
}

def Async_RuntimeSetErrorOp : Async_Op<"runtime.set_error"> {
  let summary = "switches token or value to error state";
  let description = [{
    The `async.runtime.set_error` operation switches async token or value
    state to error.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeSetErrorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeSetErrorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 575-587
```tablegen
  let arguments = (ins Async_AnyValueOrTokenType:$operand);
  let assemblyFormat = "$operand attr-dict `:` type($operand)";
}

def Async_RuntimeIsErrorOp : Async_Op<"runtime.is_error"> {
  let summary = "returns true if token, value or group is in error state";
  let description = [{
    The `async.runtime.is_error` operation returns true if the token, value or
    group (any of the async runtime values) is in the error state. It is the
    caller responsibility to check error state after the call to `await` or
    resuming after `await_and_resume`.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeIsErrorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeIsErrorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 588-600
```tablegen
  let arguments = (ins Async_AnyAsyncType:$operand);
  let results = (outs I1:$is_error);

  let assemblyFormat = "$operand attr-dict `:` type($operand)";
}

def Async_RuntimeAwaitOp : Async_Op<"runtime.await"> {
  let summary = "blocks the caller thread until the operand becomes available";
  let description = [{
    The `async.runtime.await` operation blocks the caller thread until the
    operand becomes available or error.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeAwaitOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeAwaitOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 601-611
```tablegen
  let arguments = (ins Async_AnyAsyncType:$operand);
  let assemblyFormat = "$operand attr-dict `:` type($operand)";
}

def Async_RuntimeResumeOp : Async_Op<"runtime.resume"> {
  let summary = "resumes the coroutine on a thread managed by the runtime";
  let description = [{
    The `async.runtime.resume` operation resumes the coroutine on a thread
    managed by the runtime.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeResumeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeResumeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 612-623
```tablegen
  let arguments = (ins Async_CoroHandleType:$handle);
  let assemblyFormat = "$handle attr-dict";
}

def Async_RuntimeAwaitAndResumeOp : Async_Op<"runtime.await_and_resume"> {
  let summary = "awaits the async operand and resumes the coroutine";
  let description = [{
    The `async.runtime.await_and_resume` operation awaits for the operand to
    become available or error and resumes the coroutine on a thread managed by
    the runtime.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeAwaitAndResumeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeAwaitAndResumeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 624-638
```tablegen
  let arguments = (ins Async_AnyAsyncType:$operand,
                       Async_CoroHandleType:$handle);
  let assemblyFormat = "$operand `,` $handle attr-dict `:` type($operand)";
}

def Async_RuntimeStoreOp : Async_Op<"runtime.store",
      [TypesMatchWith<"type of 'value' matches element type of 'storage'",
                     "storage", "value",
                     "::llvm::cast<ValueType>($_self).getValueType()">]> {
  let summary = "stores the value into the runtime async.value";
  let description = [{
    The `async.runtime.store` operation stores the value into the runtime
    async.value storage.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeStoreOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeStoreOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 639-653
```tablegen
  let arguments = (ins AnyType:$value,
                       Async_ValueType:$storage);
  let assemblyFormat = "$value `,` $storage attr-dict `:` type($storage)";
}

def Async_RuntimeLoadOp : Async_Op<"runtime.load",
      [TypesMatchWith<"type of 'value' matches element type of 'storage'",
                     "storage", "result",
                     "::llvm::cast<ValueType>($_self).getValueType()">]> {
  let summary = "loads the value from the runtime async.value";
  let description = [{
    The `async.runtime.load` operation loads the value from the runtime
    async.value storage.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 654-665
```tablegen
  let arguments = (ins Async_ValueType:$storage);
  let results = (outs AnyType:$result);
  let assemblyFormat = "$storage attr-dict `:` type($storage)";
}

def Async_RuntimeAddToGroupOp : Async_Op<"runtime.add_to_group", []> {
  let summary = "adds an async token or value to the group";
  let description = [{
    The `async.runtime.add_to_group` adds an async token or value to the async
    group. Returns the rank of the added element in the group.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeAddToGroupOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeAddToGroupOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 666-685
```tablegen
  let arguments = (ins Async_AnyValueOrTokenType:$operand,
                       Async_GroupType:$group);
  let results = (outs Index:$rank);

  let assemblyFormat = "$operand `,` $group attr-dict `:` type($operand)";
}

// All async values (values, tokens, groups) are reference counted at runtime
// and automatically destructed when reference count drops to 0.
//
// All values are semantically created with a reference count of +1 and it is
// the responsibility of the last async value user to drop reference count.
//
// Async values created when:
//   1. Operation returns async result (e.g. the result of an `async.execute`).
//   2. Async value passed in as a block argument.
//
// It is the responsibility of the async value user to extend the lifetime by
// adding a +1 reference, if the reference counted value captured by the
// asynchronously executed region (`async.execute` operation), and drop it after
```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 686-695
```tablegen
// the last nested use.
//
// Reference counting operations can be added to the IR using automatic
// reference count pass, that relies on liveness analysis to find the last uses
// of all reference counted values and automatically inserts
// `drop_ref` operations.
//
// See `AsyncRefCountingPass` documentation for the implementation details.

def Async_RuntimeAddRefOp : Async_Op<"runtime.add_ref"> {
```
- **EN**: Introduces declarations for `Async_RuntimeAddRefOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeAddRefOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 696-709
```tablegen
  let summary = "adds a reference to async value";
  let description = [{
    The `async.runtime.add_ref` operation adds a reference(s) to async value
    (token, value or group).
  }];

  let arguments = (ins Async_AnyAsyncType:$operand,
                       ConfinedAttr<I64Attr, [IntPositive]>:$count);

  let assemblyFormat = [{
    $operand attr-dict `:` type($operand)
  }];
}

```
- **EN**: Implements logic around `reference`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `reference`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 710-719
```tablegen
def Async_RuntimeDropRefOp : Async_Op<"runtime.drop_ref"> {
  let summary = "drops a reference to async value";
  let description = [{
    The `async.runtime.drop_ref` operation drops a reference(s) to async value
    (token, value or group).
  }];

  let arguments = (ins Async_AnyAsyncType:$operand,
                       ConfinedAttr<I64Attr, [IntPositive]>:$count);

```
- **EN**: Introduces declarations for `Async_RuntimeDropRefOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeDropRefOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 720-733
```tablegen
  let assemblyFormat = [{
    $operand attr-dict `:` type($operand)
  }];
}

def Async_RuntimeNumWorkerThreadsOp :
  Async_Op<"runtime.num_worker_threads",
           [DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
  let summary = "gets the number of threads in the threadpool from the runtime";
  let description = [{
    The `async.runtime.num_worker_threads` operation gets the number of threads
    in the threadpool from the runtime.
  }];

```
- **EN**: Introduces declarations for `Async_RuntimeNumWorkerThreadsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Async_RuntimeNumWorkerThreadsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 734-738
```tablegen
  let results = (outs Index:$result);
  let assemblyFormat = "attr-dict `:` type($result)";
}

#endif // ASYNC_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Describes the declarative or C++ interface for dialect operations, attributes, types, and registration helpers.
  - **CN**: 描述方言操作、属性、类型及注册辅助逻辑的声明式或 C++ 接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/Async/IR/AsyncDialect.td`, `mlir/Dialect/Async/IR/AsyncTypes.td`, `mlir/Interfaces/ControlFlowInterfaces.td`, `mlir/Interfaces/InferTypeOpInterface.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/Interfaces/CallInterfaces.td`, `mlir/IR/SymbolInterfaces.td`, `mlir/Interfaces/FunctionInterfaces.td`, `mlir/IR/OpAsmInterface.td`
- **Subsystem categories / 子系统类别**: cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (5), dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2)
