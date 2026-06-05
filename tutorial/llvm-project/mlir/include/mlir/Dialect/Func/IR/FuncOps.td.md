# FuncOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Func/IR/FuncOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the Func dialect, focused on dialect IR declarations such as operations, attributes, types, enums, and registration hooks and `FuncOps`.
  - **CN**: 为 Func 方言定义聚焦 `FuncOps` 的声明式 TableGen 规格，覆盖方言 IR 声明，例如操作、属性、类型、枚举与注册钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```tablegen
//===- FuncOps.td - Func operation definitions -------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 9-20
```tablegen
#ifndef MLIR_DIALECT_FUNC_IR_FUNCOPS_TD
#define MLIR_DIALECT_FUNC_IR_FUNCOPS_TD

include "mlir/IR/EnumAttr.td"
include "mlir/IR/OpAsmInterface.td"
include "mlir/IR/SymbolInterfaces.td"
include "mlir/Interfaces/CallInterfaces.td"
include "mlir/Interfaces/ControlFlowInterfaces.td"
include "mlir/Interfaces/FunctionInterfaces.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/SideEffectInterfaces.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 21-28
```tablegen
def Func_Dialect : Dialect {
  let name = "func";
  let cppNamespace = "::mlir::func";
  let hasConstantMaterializer = 1;
}

// Base class for Func dialect ops.
class Func_Op<string mnemonic, list<Trait> traits = []> :
```
- **EN**: Introduces declarations for `Func_Dialect`, `for`, `Func_Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Func_Dialect`, `for`, `Func_Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 29-44
```tablegen
    Op<Func_Dialect, mnemonic, traits>;

//===----------------------------------------------------------------------===//
// CallOp
//===----------------------------------------------------------------------===//

def CallOp : Func_Op<"call",
    [CallOpInterface, MemRefsNormalizable,
     DeclareOpInterfaceMethods<SymbolUserOpInterface>]> {
  let summary = "call operation";
  let description = [{
    The `func.call` operation represents a direct call to a function that is
    within the same symbol scope as the call. The operands and result types of
    the call must match the specified function type. The callee is encoded as a
    symbol reference attribute named "callee".

```
- **EN**: Introduces declarations for `CallOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CallOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 45-59
```tablegen
    Example:

    ```mlir
    %2 = func.call @my_add(%0, %1) : (f32, f32) -> f32
    ```
  }];

  let arguments = (ins
    FlatSymbolRefAttr:$callee,
    Variadic<AnyType>:$operands,
    OptionalAttr<DictArrayAttr>:$arg_attrs,
    OptionalAttr<DictArrayAttr>:$res_attrs, 
    UnitAttr:$no_inline
  );

```
- **EN**: Declares APIs or declarative rules around `my_add`.
- **CN**: 声明与 `my_add` 相关的 API 或声明式规则。

### Lines 60-75
```tablegen
  let results = (outs Variadic<AnyType>);

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
```
- **EN**: Implements logic around `OpBuilder`, `addOperands`, `addAttribute`, `addTypes`.
- **CN**: 围绕 `OpBuilder`, `addOperands`, `addAttribute`, `addTypes` 实现具体逻辑。

### Lines 76-91
```tablegen
      build($_builder, $_state, SymbolRefAttr::get(callee), results, operands);
    }]>,
    OpBuilder<(ins "StringRef":$callee, "TypeRange":$results,
      CArg<"ValueRange", "{}">:$operands), [{
      build($_builder, $_state, StringAttr::get($_builder.getContext(), callee),
            results, operands);
    }]>,
    OpBuilder<(ins "TypeRange":$results, "FlatSymbolRefAttr":$callee,
      CArg<"ValueRange", "{}">:$operands), [{
      build($_builder, $_state, callee, results, operands);
    }]>,
    OpBuilder<(ins "TypeRange":$results, "StringAttr":$callee,
      CArg<"ValueRange", "{}">:$operands), [{
      build($_builder, $_state, callee, results, operands);
    }]>,
    OpBuilder<(ins "TypeRange":$results, "StringRef":$callee,
```
- **EN**: Implements logic around `build`, `OpBuilder`.
- **CN**: 围绕 `build`, `OpBuilder` 实现具体逻辑。

### Lines 92-99
```tablegen
      CArg<"ValueRange", "{}">:$operands), [{
      build($_builder, $_state, callee, results, operands);
    }]>];

  let extraClassDeclaration = [{
    FunctionType getCalleeType();

    /// Get the argument operands to the called function.
```
- **EN**: Implements logic around `build`, `getCalleeType`.
- **CN**: 围绕 `build`, `getCalleeType` 实现具体逻辑。

### Lines 100-107
```tablegen
    operand_range getArgOperands() {
      return {arg_operand_begin(), arg_operand_end()};
    }

    MutableOperandRange getArgOperandsMutable() {
      return getOperandsMutable();
    }

```
- **EN**: Implements logic around `getArgOperands`, `arg_operand_begin`, `getArgOperandsMutable`, `getOperandsMutable`.
- **CN**: 围绕 `getArgOperands`, `arg_operand_begin`, `getArgOperandsMutable`, `getOperandsMutable` 实现具体逻辑。

### Lines 108-115
```tablegen
    operand_iterator arg_operand_begin() { return operand_begin(); }
    operand_iterator arg_operand_end() { return operand_end(); }

    /// Return the callee of this operation.
    CallInterfaceCallable getCallableForCallee() {
      return (*this)->getAttrOfType<SymbolRefAttr>("callee");
    }

```
- **EN**: Implements logic around `arg_operand_begin`, `arg_operand_end`, `getCallableForCallee`, `getAttrOfType`.
- **CN**: 围绕 `arg_operand_begin`, `arg_operand_end`, `getCallableForCallee`, `getAttrOfType` 实现具体逻辑。

### Lines 116-126
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

### Lines 127-142
```tablegen
//===----------------------------------------------------------------------===//
// CallIndirectOp
//===----------------------------------------------------------------------===//

def CallIndirectOp : Func_Op<"call_indirect", [
      CallOpInterface,
      TypesMatchWith<"callee input types match argument types",
                     "callee", "callee_operands",
                     "::llvm::cast<FunctionType>($_self).getInputs()">,
      TypesMatchWith<"callee result types match result types",
                     "callee", "results",
                     "::llvm::cast<FunctionType>($_self).getResults()">
    ]> {
  let summary = "indirect call operation";
  let description = [{
    The `func.call_indirect` operation represents an indirect call to a value
```
- **EN**: Introduces declarations for `CallIndirectOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CallIndirectOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 143-150
```tablegen
    of function type. The operands and result types of the call must match the
    specified function type.

    Function values can be created with the
    [`func.constant` operation](#funcconstant-constantop).

    Example:

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 151-163
```tablegen
    ```mlir
    %func = func.constant @my_func : (tensor<16xf32>, tensor<16xf32>) -> tensor<16xf32>
    %result = func.call_indirect %func(%0, %1) : (tensor<16xf32>, tensor<16xf32>) -> tensor<16xf32>
    ```
  }];

  let arguments = (ins
    FunctionType:$callee,
    Variadic<AnyType>:$callee_operands,
    OptionalAttr<DictArrayAttr>:$arg_attrs,
    OptionalAttr<DictArrayAttr>:$res_attrs
  );

```
- **EN**: Declares APIs or declarative rules around `func`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `func` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 164-172
```tablegen
  let results = (outs Variadic<AnyType>:$results);

  let builders = [
    OpBuilder<(ins "Value":$callee, CArg<"ValueRange", "{}">:$operands), [{
      $_state.operands.push_back(callee);
      $_state.addOperands(operands);
      $_state.addTypes(::llvm::cast<FunctionType>(callee.getType()).getResults());
    }]>];

```
- **EN**: Implements logic around `OpBuilder`, `push_back`, `addOperands`, `addTypes`.
- **CN**: 围绕 `OpBuilder`, `push_back`, `addOperands`, `addTypes` 实现具体逻辑。

### Lines 173-181
```tablegen
  let extraClassDeclaration = [{
    // TODO: Remove once migrated callers.
    ValueRange operands() { return getCalleeOperands(); }

    /// Get the argument operands to the called function.
    operand_range getArgOperands() {
      return {arg_operand_begin(), arg_operand_end()};
    }

```
- **EN**: Implements logic around `operands`, `getArgOperands`, `arg_operand_begin`.
- **CN**: 围绕 `operands`, `getArgOperands`, `arg_operand_begin` 实现具体逻辑。

### Lines 182-189
```tablegen
    MutableOperandRange getArgOperandsMutable() {
      return getCalleeOperandsMutable();
    }

    operand_iterator arg_operand_begin() { return ++operand_begin(); }
    operand_iterator arg_operand_end() { return operand_end(); }

    /// Return the callee of this operation.
```
- **EN**: Implements logic around `getArgOperandsMutable`, `getCalleeOperandsMutable`, `arg_operand_begin`, `arg_operand_end`.
- **CN**: 围绕 `getArgOperandsMutable`, `getCalleeOperandsMutable`, `arg_operand_begin`, `arg_operand_end` 实现具体逻辑。

### Lines 190-197
```tablegen
    CallInterfaceCallable getCallableForCallee() { return getCallee(); }

    /// Set the callee for this operation.
    void setCalleeFromCallable(CallInterfaceCallable callee) {
      setOperand(0, cast<Value>(callee));
    }
  }];

```
- **EN**: Implements logic around `getCallableForCallee`, `setCalleeFromCallable`, `setOperand`.
- **CN**: 围绕 `getCallableForCallee`, `setCalleeFromCallable`, `setOperand` 实现具体逻辑。

### Lines 198-206
```tablegen
  let hasCanonicalizeMethod = 1;
  let assemblyFormat = [{
    $callee `(` $callee_operands `)` attr-dict `:` type($callee)
  }];
}

//===----------------------------------------------------------------------===//
// ConstantOp
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 207-216
```tablegen

def ConstantOp : Func_Op<"constant",
    [ConstantLike, Pure,
     DeclareOpInterfaceMethods<SymbolUserOpInterface>,
     DeclareOpInterfaceMethods<OpAsmOpInterface, ["getAsmResultNames"]>]> {
  let summary = "constant";
  let description = [{
    The `func.constant` operation produces an SSA value from a symbol reference
    to a `func.func` operation

```
- **EN**: Introduces declarations for `ConstantOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ConstantOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 217-226
```tablegen
    Example:

    ```mlir
    // Reference to function @myfn.
    %2 = func.constant @myfn : (tensor<16xf32>, f32) -> tensor<16xf32>

    // Equivalent generic forms
    %2 = "func.constant"() { value = @myfn } : () -> ((tensor<16xf32>, f32) -> tensor<16xf32>)
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 227-236
```tablegen
    MLIR does not allow direct references to functions in SSA operands because
    the compiler is multithreaded, and disallowing SSA values to directly
    reference a function simplifies this
    ([rationale](../Rationale/Rationale.md#multithreading-the-compiler)).
  }];

  let arguments = (ins FlatSymbolRefAttr:$value);
  let results = (outs AnyType);
  let assemblyFormat = "attr-dict $value `:` type(results)";

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 237-245
```tablegen
  let extraClassDeclaration = [{
    /// Returns true if a constant operation can be built with the given value
    /// and result type.
    static bool isBuildableWith(Attribute value, Type type);
  }];

  let hasFolder = 1;
}

```
- **EN**: Implements logic around `isBuildableWith`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isBuildableWith` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 246-261
```tablegen
//===----------------------------------------------------------------------===//
// FuncOp
//===----------------------------------------------------------------------===//

def FuncOp : Func_Op<"func", [
  AffineScope, AutomaticAllocationScope,
  FunctionOpInterface, IsolatedFromAbove, OpAsmOpInterface
]> {
  let summary = "An operation with a name containing a single `SSACFG` region";
  let description = [{
    Operations within the function cannot implicitly capture values defined
    outside of the function, i.e. Functions are `IsolatedFromAbove`. All
    external references must use function arguments or attributes that establish
    a symbolic connection (e.g. symbols referenced by name via a string
    attribute like SymbolRefAttr). An external function declaration (used when
    referring to a function declared in some other module) has no body. While
```
- **EN**: Introduces declarations for `FuncOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `FuncOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 262-270
```tablegen
    the MLIR textual form provides a nice inline syntax for function arguments,
    they are internally represented as “block arguments” to the first block in
    the region.

    Only dialect attribute names may be specified in the attribute dictionaries
    for function arguments, results, or the function itself.

    Example:

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 271-281
```tablegen
    ```mlir
    // External function definitions.
    func.func private @abort()
    func.func private @scribble(i32, i64, memref<? x 128 x f32, #layout_map0>) -> f64

    // A function that returns its argument twice:
    func.func @count(%x: i64) -> (i64, i64)
      attributes {fruit = "banana"} {
      return %x, %x: i64, i64
    }

```
- **EN**: Implements logic around `abort`, `scribble`, `count`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `abort`, `scribble`, `count` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 282-292
```tablegen
    // A function with an argument attribute
    func.func private @example_fn_arg(%x: i32 {swift.self = unit})

    // A function with a result attribute
    func.func private @example_fn_result() -> (f64 {dialectName.attrName = 0 : i64})

    // A function with an attribute
    func.func private @example_fn_attr() attributes {dialectName.attrName = false}
    ```
  }];

```
- **EN**: Implements logic around `example_fn_arg`, `example_fn_result`, `example_fn_attr`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `example_fn_arg`, `example_fn_result`, `example_fn_attr` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 293-300
```tablegen
  let arguments = (ins SymbolNameAttr:$sym_name,
                       TypeAttrOf<FunctionType>:$function_type,
                       OptionalAttr<StrAttr>:$sym_visibility,
                       OptionalAttr<DictArrayAttr>:$arg_attrs,
                       OptionalAttr<DictArrayAttr>:$res_attrs,
                       UnitAttr:$no_inline);
  let regions = (region AnyRegion:$body);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 301-314
```tablegen
  let builders = [OpBuilder<(ins
    "StringRef":$name, "FunctionType":$type,
    CArg<"ArrayRef<NamedAttribute>", "{}">:$attrs,
    CArg<"ArrayRef<DictionaryAttr>", "{}">:$argAttrs)
  >];
  let extraClassDeclaration = [{
    static FuncOp create(Location location, StringRef name, FunctionType type,
                         ArrayRef<NamedAttribute> attrs = {});
    static FuncOp create(Location location, StringRef name, FunctionType type,
                         Operation::dialect_attr_range attrs);
    static FuncOp create(Location location, StringRef name, FunctionType type,
                         ArrayRef<NamedAttribute> attrs,
                         ArrayRef<DictionaryAttr> argAttrs);

```
- **EN**: Implements logic around `OpBuilder`, `create`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder`, `create` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 315-324
```tablegen
    /// Create a deep copy of this function and all of its blocks, remapping any
    /// operands that use values outside of the function using the map that is
    /// provided (leaving them alone if no entry is present). If the mapper
    /// contains entries for function arguments, these arguments are not
    /// included in the new function. Replaces references to cloned sub-values
    /// with the corresponding value that is copied, and adds those mappings to
    /// the mapper.
    FuncOp clone(IRMapping &mapper);
    FuncOp clone();

```
- **EN**: Declares APIs or declarative rules around `clone`.
- **CN**: 声明与 `clone` 相关的 API 或声明式规则。

### Lines 325-333
```tablegen
    /// Clone the internal blocks and attributes from this function into dest.
    /// Any cloned blocks are appended to the back of dest. This function
    /// asserts that the attributes of the current function and dest are
    /// compatible.
    void cloneInto(FuncOp dest, IRMapping &mapper);

    //===------------------------------------------------------------------===//
    // FunctionOpInterface Methods
    //===------------------------------------------------------------------===//
```
- **EN**: Declares APIs or declarative rules around `cloneInto`.
- **CN**: 声明与 `cloneInto` 相关的 API 或声明式规则。

### Lines 334-342
```tablegen

    /// Returns the region on the current operation that is callable. This may
    /// return null in the case of an external callable object, e.g. an external
    /// function.
    ::mlir::Region *getCallableRegion() { return isExternal() ? nullptr : &getBody(); }

    /// Returns the argument types of this function.
    ArrayRef<Type> getArgumentTypes() { return getFunctionType().getInputs(); }

```
- **EN**: Implements logic around `getCallableRegion`, `getArgumentTypes`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getCallableRegion`, `getArgumentTypes` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 343-350
```tablegen
    /// Returns the result types of this function.
    ArrayRef<Type> getResultTypes() { return getFunctionType().getResults(); }

    //===------------------------------------------------------------------===//
    // OpAsmOpInterface Methods
    //===------------------------------------------------------------------===//

    /// Allow the dialect prefix to be omitted.
```
- **EN**: Implements logic around `getResultTypes`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getResultTypes` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 351-362
```tablegen
    static StringRef getDefaultDialect() { return "func"; }

    //===------------------------------------------------------------------===//
    // SymbolOpInterface Methods
    //===------------------------------------------------------------------===//

    bool isDeclaration() { return isExternal(); }
  }];
  let hasCustomAssemblyFormat = 1;
  let hasRegionVerifier = 1;
}

```
- **EN**: Implements logic around `getDefaultDialect`, `isDeclaration`.
- **CN**: 围绕 `getDefaultDialect`, `isDeclaration` 实现具体逻辑。

### Lines 363-375
```tablegen
//===----------------------------------------------------------------------===//
// ReturnOp
//===----------------------------------------------------------------------===//

def ReturnOp : Func_Op<"return", [Pure, HasParent<"FuncOp">,
                                MemRefsNormalizable, ReturnLike, Terminator]> {
  let summary = "Function return operation";
  let description = [{
    The `func.return` operation represents a return operation within a function.
    The operation takes variable number of operands and produces no results.
    The operand number and types must match the signature of the function
    that contains the operation.

```
- **EN**: Introduces declarations for `ReturnOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ReturnOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 376-385
```tablegen
    Example:

    ```mlir
    func.func @foo() -> (i32, f8) {
      ...
      return %0, %1 : i32, f8
    }
    ```
  }];

```
- **EN**: Implements logic around `foo`.
- **CN**: 围绕 `foo` 实现具体逻辑。

### Lines 386-394
```tablegen
  let arguments = (ins Variadic<AnyType>:$operands);

  let builders = [OpBuilder<(ins), [{
    build($_builder, $_state, {});
  }]>];

  let assemblyFormat = "attr-dict ($operands^ `:` type($operands))?";
}

```
- **EN**: Implements logic around `OpBuilder`, `build`, `dict`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder`, `build`, `dict` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 395-395
```tablegen
#endif // MLIR_DIALECT_FUNC_IR_FUNCOPS_TD
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
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/IR/EnumAttr.td`, `mlir/IR/OpAsmInterface.td`, `mlir/IR/SymbolInterfaces.td`, `mlir/Interfaces/CallInterfaces.td`, `mlir/Interfaces/ControlFlowInterfaces.td`, `mlir/Interfaces/FunctionInterfaces.td`, `mlir/Interfaces/InferTypeOpInterface.td`, `mlir/Interfaces/SideEffectInterfaces.td`
- **Subsystem categories / 子系统类别**: cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3)
