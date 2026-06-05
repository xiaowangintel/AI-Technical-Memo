# WasmSSAOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/WasmSSA/IR/WasmSSAOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR WasmSSAOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: Base class for WasmSSA operations.
- **用途（CN）**: 为 MLIR 的 WasmSSAOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
````tablegen
//===- WasmSSAOps.td - WasmSSA op definitions -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef WasmSSA_OPS
#define WasmSSA_OPS


include "mlir/Dialect/WasmSSA/IR/WasmSSABase.td"
include "mlir/Dialect/WasmSSA/IR/WasmSSATypes.td"
include "mlir/Dialect/WasmSSA/IR/WasmSSAInterfaces.td"

include "mlir/Interfaces/FunctionInterfaces.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/IR/BuiltinAttributeInterfaces.td"
include "mlir/IR/SymbolInterfaces.td"

// Base class for WasmSSA operations.
// Most operations are made to match 1:1, only ignoring the stack-based approach of Wasm
// for an SSA based approach. In cases where operations match 1:1 the Wasm spec,
// no description is provided.
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 26-27
````tablegen
class WasmSSA_Op<string mnemonic, list<Trait> traits = []> :
    Op<WasmSSA_Dialect, mnemonic, traits>;
````
- **EN**: This TableGen block defines `WasmSSA_Op` as a `class` record for `WasmSSAOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_Op` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 29-44
````tablegen
class WasmSSA_BlockLikeOp<string mnemonic, string summaryStr> :
  WasmSSA_Op<mnemonic, [Terminator, DeclareOpInterfaceMethods<LabelLevelOpInterface>]> {
  let summary = summaryStr;
  let arguments = (ins Variadic<WasmSSA_ValType>: $inputs);
  let regions = (region AnyRegion: $body);
  let successors = (successor AnySuccessor: $target);
  let extraClassDeclaration = [{
    ::mlir::Block* createBlock() {
      auto &block = getBody().emplaceBlock();
      for (auto input : getInputs())
        block.addArgument(input.getType(), input.getLoc());
      return &block;
    }
  }];
  let assemblyFormat = "(`(`$inputs^`)` `:` type($inputs))? attr-dict  `:` $body `>` $target";
}
````
- **EN**: This TableGen block defines `WasmSSA_BlockLikeOp` as a `class` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_BlockLikeOp` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 46-63
````tablegen
def WasmSSA_BlockOp : WasmSSA_BlockLikeOp<
    "block",
    "Create a nesting level with a label at its exit."> {
  let description = [{
  Defines a Wasm block, creating a new nested scope.
  A block contains a body region and an optional list of input values.
  Control can enter the block and later branch out to the block target.
  Example:

  ```mlir
  wasmssa.block {

    // instructions

  } > ^successor
  ```
  }];
}
````
- **EN**: This TableGen block defines `WasmSSA_BlockOp` as a `def` record for `WasmSSAOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_BlockOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 65-80
````tablegen
def WasmSSA_LoopOp : WasmSSA_BlockLikeOp<
    "loop",
    "Create a nesting level that define its entry as jump target."> {
  let description = [{
  Represents a Wasm loop construct. This defines a nesting level with
  a label at the entry of the region.

  Example:

  ```mlir
  wasmssa.loop {

  } > ^successor
  ```
  }];
}
````
- **EN**: This TableGen block defines `WasmSSA_LoopOp` as a `def` record for `WasmSSAOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LoopOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 82-106
````tablegen
def WasmSSA_BlockReturnOp : WasmSSA_Op<"block_return", [Terminator,
    DeclareOpInterfaceMethods<LabelBranchingOpInterface>]> {
  let summary = "Return from the current block";
  let arguments = (ins Variadic<WasmSSA_ValType>: $inputs);
  let extraClassDeclaration = [{
    ::mlir::Block* getTarget();
  }];
  let description = [{
    Escape from the current nesting level and return the control flow to its successor.
    Optionally, mark the arguments that should be transfered to the successor block.

    This shouldn't be confused with branch operations that targets the label defined
    by the nesting level operation.

    For instance, a `wasmssa.block_return` in a loop will give back control to the
    successor of the loop, where a `branch` targeting the loop will flow back to the entry block of the loop.

    Example:

    ```mlir
    wasmssa.block_return
    ```
  }];
  let assemblyFormat = "($inputs^ `:` type($inputs))? attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_BlockReturnOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_BlockReturnOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 108-126
````tablegen
def WasmSSA_BranchIfOp : WasmSSA_Op<"branch_if", [
    Terminator,
    DeclareOpInterfaceMethods<LabelBranchingOpInterface>]> {
  let summary = "Jump to target level if condition has non-zero value";
  let arguments = (ins I32: $condition,
                       UI32Attr: $exitLevel,
                       Variadic<WasmSSA_ValType>: $inputs);
  let description = [{
     Jump to target level if the condition is has a non-zero value.

     Example:

     ```mlir
     wasmssa.branch_if %a to level 0 with args(%b : i32) else ^bb1
     ```
    }];
  let successors = (successor AnySuccessor: $elseSuccessor);
  let assemblyFormat = "$condition `to` `level` $exitLevel (`with` `args`  `(`$inputs^ `:` type($inputs)`)`)?  `else` $elseSuccessor  attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_BranchIfOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_BranchIfOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 128-148
````tablegen
def WasmSSA_ConstOp : WasmSSA_Op<"const", [
    AllTypesMatch<["value", "result"]>,
    ConstantExprOpTrait]> {
  let summary = "Operator that represents a constant value";
  let description = [{
     Defines a constant value.

     Example:

     ```mlir
     // Example of integer constant
     %a = wasmssa.const 1 : i32

     // Example of floating point constant
     %b = wasmssa.const 9.000000e+00 : f64
     ```
    }];
  let arguments = (ins TypedAttrInterface: $value);
  let results = (outs WasmSSA_NumericType: $result);
  let assemblyFormat = "$value attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_ConstOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ConstOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 150-220
````tablegen
def WasmSSA_FuncOp : WasmSSA_Op<"func", [
    AffineScope, AutomaticAllocationScope,
    DeclareOpInterfaceMethods<FunctionOpInterface, ["verifyBody"]>,
    IsolatedFromAbove,
    Symbol]> {
  let description = [{
    Represents a Wasm function definition.

    In Wasm function, locals and function arguments are interchangeable.
    They are for instance both accessed using `local.get` instruction.

    On the other hand, a function type is defined as a pair of tuples of Wasm value types.
    To model this, the wasm.func operation has:

    - A function type that represents the corresponding Wasm type (tuples of value types)

    - Arguments of the entry block of type `!wasm<local T>`, with T the corresponding type
     in the function type.

    By default, `wasmssa.func` have nested visibility. Functions exported by the module
    are marked with the exported attribute. This gives them public visibility.

     Example:

     ```mlir
     // Internal function with no arguments that returns a float32
     wasmssa.func @my_f32_func() -> f32

     // Exported function with no arguments that returns a float32
     wasmssa.func exported @my_f32_func() -> f32

     // A function that takes a local ref argument
     wasmssa.func @i64_wrap(%a: !wasmssa<local ref to i64>) -> i32
     ```
  }];
  let arguments = (ins SymbolNameAttr: $sym_name,
                     WasmSSA_FuncTypeAttr: $functionType,
                     OptionalAttr<DictArrayAttr>:$arg_attrs,
                     OptionalAttr<DictArrayAttr>:$res_attrs,
                     UnitAttr: $exported);
  let regions = (region AnyRegion: $body);
  let extraClassDeclaration = [{

    /// Create the entry block for the function with parameters wrapped in local ref.
    ::mlir::Block* addEntryBlock();

    //===------------------------------------------------------------------===//
    // FunctionOpInterface Methods
    //===------------------------------------------------------------------===//

    /// Returns the region on the current operation that is callable. This may
    /// return null in the case of an external callable object, e.g. an external
    /// function.
    ::mlir::Region *getCallableRegion() { return isExternal() ? nullptr : &getBody(); }

    /// Returns the argument types of this function.
    ArrayRef<Type> getArgumentTypes() { return getFunctionType().getInputs(); }

    /// Returns the result types of this function.
    ArrayRef<Type> getResultTypes() { return getFunctionType().getResults(); }

    ::mlir::SymbolTable::Visibility getVisibility() {
      return getExported() ?
        ::mlir::SymbolTable::Visibility::Public :
        ::mlir::SymbolTable::Visibility::Nested;
    };
  }];

  let builders = [
    OpBuilder<(ins "::llvm::StringRef":$symbol, "FunctionType":$funcType )>
  ];
````
- **EN**: This TableGen block defines `WasmSSA_FuncOp` as a `def` record for `WasmSSAOps`. It covers operand or attribute schema, semantic documentation, verification hooks, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_FuncOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 语义文档, 验证钩子, trait/接口组合。

### Lines 221-222
````tablegen
  let hasCustomAssemblyFormat = 1;
}
````
- **EN**: This section focuses on let hascustomassemblyformat = 1;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“let hasCustomAssemblyFormat = 1;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 224-239
````tablegen
def WasmSSA_FuncCallOp : WasmSSA_Op<"call"> {
  let summary = "Calling a Wasm function";
  let description = [{
     Emits a call to a defined function

     Example:

     ```mlir
     %a = wasmssa.call @func_0 : () -> i32
     ```
    }];
  let arguments = (ins FlatSymbolRefAttr: $callee,
                       Variadic<WasmSSA_ValType>:  $operands);
  let results = (outs Variadic<WasmSSA_ValType>: $results);
  let assemblyFormat = "$callee (`(`$operands^`)`)? attr-dict `:` functional-type($operands, $results)";
}
````
- **EN**: This TableGen block defines `WasmSSA_FuncCallOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_FuncCallOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 241-286
````tablegen
def WasmSSA_FuncImportOp : WasmSSA_Op<"import_func", [
    Symbol,
    CallableOpInterface,
    ImportOpInterface]> {
  let summary = "Importing a function variable";
  let description = [{
     Imports a function from another module

     Example:

     ```mlir
     // Imports foo(i32) -> () from the module my_module
     wasmssa.import_func "foo" from "my_module" as @func_0 {sym_visibility = "nested", type = (i32) -> ()}
     ```
    }];
  let arguments = (ins SymbolNameAttr: $sym_name,
                     StrAttr: $moduleName,
                     StrAttr: $importName,
                     WasmSSA_FuncTypeAttr: $type,
                     OptionalAttr<DictArrayAttr>:$arg_attrs,
                     OptionalAttr<DictArrayAttr>:$res_attrs);
  let extraClassDeclaration = [{
    bool isDeclaration() const { return true; }

    Region *getCallableRegion() { return nullptr; }

    ::llvm::ArrayRef<Type> getArgumentTypes() {
      return getType().getInputs();
    }

    ::llvm::ArrayRef<Type> getResultTypes() {
      return getType().getResults();
    }

    ::mlir::SymbolTable::Visibility getVisibility() {
      return ::mlir::SymbolTable::Visibility::Nested;
    };
  }];
  let builders = [
    OpBuilder<(ins "StringRef":$symbol,
                   "StringRef":$moduleName,
                   "StringRef":$importName,
                   "FunctionType": $type)>
  ];
  let assemblyFormat = "$importName `from` $moduleName `as` $sym_name attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_FuncImportOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_FuncImportOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 288-331
````tablegen
def WasmSSA_GlobalOp : WasmSSA_Op<"global", [
  AffineScope, AutomaticAllocationScope,
  IsolatedFromAbove, Symbol, ConstantExpressionInitializerOpTrait]> {
  let summary= "WebAssembly global value";
  let arguments = (ins SymbolNameAttr: $sym_name,
                     WasmSSA_ValTypeAttr: $type,
                     UnitAttr: $isMutable,
                     UnitAttr: $exported);
  let description = [{
    WebAssembly global variable.
    Body contains the initialization instructions for the variable value.
    The body must contain only instructions considered `const` in a webassembly context,
    such as `wasmssa.const` or `global.get`.

    By default, `wasmssa.global` have nested visibility. Global exported by the module
    are marked with the exported attribute. This gives them public visibility.

    Example:

    ```mlir
    // Define module_global_var, an internal mutable i32 global variable equal to 10.
    wasmssa.global @module_global_var i32 mutable : {
          %[[VAL_0:.*]] = wasmssa.const 10 : i32
          wasmssa.return %[[VAL_0]] : i32
    }

    // Define global_var, an exported constant i32 global variable equal to 42.
    wasmssa.global @global_var i32 : {
          %[[VAL_0:.*]] = wasmssa.const 42 : i32
          wasmssa.return %[[VAL_0]] : i32
    }
    ```
  }];
  let regions = (region AnyRegion: $initializer);

  let extraClassDeclaration = [{
    ::mlir::SymbolTable::Visibility getVisibility() {
      return getExported() ?
        ::mlir::SymbolTable::Visibility::Public :
        ::mlir::SymbolTable::Visibility::Nested;
    };
  }];
  let hasCustomAssemblyFormat = 1;
}
````
- **EN**: This TableGen block defines `WasmSSA_GlobalOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GlobalOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 333-360
````tablegen
def WasmSSA_GlobalImportOp : WasmSSA_Op<"import_global", [
    Symbol,
    ImportOpInterface]> {
  let summary = "Importing a global variable";
  let description = [{
     Imports a global from another module

     Example:

     ```mlir
     // Imports the "glob" i32 global from the module my_module as "global_0"
     wasmssa.import_global "glob" from "my_module" as @global_0 nested : i32
     ```
    }];
  let arguments = (ins SymbolNameAttr: $sym_name,
                     StrAttr: $moduleName,
                     StrAttr: $importName,
                     WasmSSA_ValTypeAttr: $type,
                     UnitAttr: $isMutable);
  let extraClassDeclaration = [{
    bool isDeclaration() const { return true; }

    ::mlir::SymbolTable::Visibility getVisibility() {
      return ::mlir::SymbolTable::Visibility::Nested;
    };
  }];
  let hasCustomAssemblyFormat = 1;
}
````
- **EN**: This TableGen block defines `WasmSSA_GlobalImportOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GlobalImportOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 362-379
````tablegen
def WasmSSA_GlobalGetOp : WasmSSA_Op<"global_get", [DeclareOpInterfaceMethods<SymbolUserOpInterface>,
  ConstantExprOpTrait]> {
  let summary = "Returns the value of the global passed as argument.";
  let description = [{
     Retrieves the value of the global passed as argument and stores it in a
     variable

     Example:

     ```mlir
     // Gets the value of `@global_0` and stores its value in %a
     %a = wasmssa.global_get @global_0 : i32
     ```
    }];
  let arguments = (ins FlatSymbolRefAttr: $global);
  let results = (outs WasmSSA_ValType: $global_val);
  let assemblyFormat = "$global attr-dict `:` type($global_val)";
}
````
- **EN**: This TableGen block defines `WasmSSA_GlobalGetOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GlobalGetOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 381-424
````tablegen
def WasmSSA_IfOp : WasmSSA_Op<"if", [Terminator,
    DeclareOpInterfaceMethods<LabelLevelOpInterface>]> {
  let summary = "Execute the if region if condition value is non-zero, the else region otherwise.";
  let description = [{
    Execute the if region if the condition is non-zero. Otherwise the else region is executed.
    The else region can be empty but must return the same datatype as the if region.
    If clauses can be nested.

     Example:

     ```mlir
     // Runs the if clause is %a is non-zero
     wasmssa.if %a {
        // Execute if %a is non-zero
     } else {
        // else clause
     }
     ```
    }];
  let arguments = (ins I32:$condition, Variadic<WasmSSA_ValType>: $inputs);
  let regions = (region AnyRegion: $if, AnyRegion: $else);
  let successors = (successor AnySuccessor: $target);
  let extraClassDeclaration = [{
    private:
    inline ::mlir::Block* createBlock(::mlir::Region& region) {
      assert(region.empty() && "Creating entry block on non empty region");
      assert(region.getParentOp() == this->getOperation() &&
        "Creating block for region that isn't part of the current op");
      auto &block = region.emplaceBlock();
      for (auto input : getInputs())
        block.addArgument(input.getType(), input.getLoc());
      return &block;
    }

    public:
    ::mlir::Block* createIfBlock() {
      return createBlock(getIf());
    }
    ::mlir::Block* createElseBlock() {
      return createBlock(getElse());
    }
  }];
  let assemblyFormat = "$condition (`(`$inputs^`)` `:` type($inputs))? attr-dict  `:` $if custom<ElseRegion>($else) `>` $target";
}
````
- **EN**: This TableGen block defines `WasmSSA_IfOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_IfOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 426-442
````tablegen
def WasmSSA_LocalOp : WasmSSA_Op<"local", [
    DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
  let summary = "Declaration of local variable";
  let description = [{
    Declares a local variable

     Example:

     ```mlir
     // Declares `%a`, a float32 local
     %a = wasmssa.local of type f32
     ```
    }];
  let arguments = (ins WasmSSA_ValTypeAttr: $type);
  let results = (outs WasmSSA_LocalRef: $result);
  let assemblyFormat = "`of` `type` $type attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_LocalOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LocalOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 444-460
````tablegen
def WasmSSA_LocalGetOp : WasmSSA_Op<"local_get", [
    DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
  let summary = "Set local to value and return the operand.";
  let description = [{
    Gets the value of a local variable and returns a reference to it.

     Example:

     ```mlir
     // Retrieves a reference to `%a`, a float32 local
     %b = wasmssa.local_get %a : ref to f32
     ```
    }];
  let arguments = (ins WasmSSA_LocalRef: $localVar);
  let results = (outs WasmSSA_ValType: $result);
  let assemblyFormat = "$localVar `:` type($localVar) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_LocalGetOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LocalGetOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 462-478
````tablegen
def WasmSSA_LocalSetOp : WasmSSA_Op<"local_set"> {
  let summary = "Set local to given value";
  let description = [{
    Sets the value of a local variable.

     Example:

     ```mlir
     // Sets `%d`, to the value of `%c`
     wasmssa.local_set %d :  ref to i32 to %c : i32
     ```
    }];
  let arguments = (ins WasmSSA_LocalRef: $localVar,
                       WasmSSA_ValType: $value);
  let hasVerifier = 1;
  let assemblyFormat = "$localVar `:` type($localVar) `to` $value `:` type($value) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_LocalSetOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LocalSetOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, 验证钩子。

### Lines 480-498
````tablegen
def WasmSSA_LocalTeeOp : WasmSSA_Op<"local_tee", [
    DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
  let summary = "Set local to value and return the operand.";
  let description = [{
    Sets the value of a local variable and returns it.

     Example:

     ```mlir
     // Sets `%b`, to the value of `%c` and returns it in %a
      %a = wasmssa.local_tee %b :  ref to i32 to %c : i32
     ```
    }];
  let arguments = (ins WasmSSA_LocalRef: $localVar,
                       WasmSSA_ValType: $value);
  let results = (outs WasmSSA_ValType: $result);
  let hasVerifier = 1;
  let assemblyFormat = "$localVar `:` type($localVar) `to` $value `:` type($value) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_LocalTeeOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LocalTeeOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 500-533
````tablegen
def WasmSSA_MemOp : WasmSSA_Op<"memory", [Symbol]> {
  let summary= "WebAssembly memory definition";
  let description = [{
    Define a memory to be used by the program.
    Multiple memories can be defined in the same module.

    By default, `wasmssa.memory` have nested visibility. Memory exported by
    the module are marked with the exported attribute. This gives them public
    visibility.

     Example:

     ```mlir
     // Define the `mem_0` (internal)  memory with defined size bounds of [0:65536]
     wasmssa.memory @mem_0 !wasmssa<limit[0:65536]>

     // Define the `mem_1` exported  memory with minimal size of 512
     wasmssa.memory exported @mem_1 !wasmssa<limit[512:]>
     ```
    }];
  let arguments = (ins SymbolNameAttr: $sym_name,
                     WasmSSA_LimitTypeAttr: $limits,
                     UnitAttr: $exported);

  let extraClassDeclaration = [{
    ::mlir::SymbolTable::Visibility getVisibility() {
      return getExported() ?
        ::mlir::SymbolTable::Visibility::Public :
        ::mlir::SymbolTable::Visibility::Nested;
    };
  }];

  let assemblyFormat = "(`exported` $exported^)? $sym_name $limits attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_MemOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_MemOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档。

### Lines 535-558
````tablegen
def WasmSSA_MemImportOp : WasmSSA_Op<"import_mem", [Symbol, ImportOpInterface]> {
  let summary = "Importing a memory";
  let description = [{
    Import a memory from another module.

     Example:

     ```mlir
     // Import the memory `mem` from `my_module` as @mem_0
     wasmssa.import_mem "mem" from "my_module" as @mem_0 {limits = !wasmssa<limit[2:]>}
     ```
    }];
  let arguments = (ins SymbolNameAttr: $sym_name,
                     StrAttr: $moduleName,
                     StrAttr: $importName,
                     WasmSSA_LimitTypeAttr: $limits);
  let extraClassDeclaration = [{
      bool isDeclaration() const { return true; }
      ::mlir::SymbolTable::Visibility getVisibility() {
       return ::mlir::SymbolTable::Visibility::Nested;
      };
   }];
  let assemblyFormat = "$importName `from` $moduleName `as` $sym_name attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_MemImportOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_MemImportOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 560-573
````tablegen
def WasmSSA_TableOp : WasmSSA_Op<"table", [Symbol]> {
  let summary= "WebAssembly table value";
  let arguments = (ins SymbolNameAttr: $sym_name,
                     WasmSSA_TableTypeAttr: $type,
                     UnitAttr: $exported);
  let extraClassDeclaration = [{
    ::mlir::SymbolTable::Visibility getVisibility() {
      return getExported() ?
        ::mlir::SymbolTable::Visibility::Public :
        ::mlir::SymbolTable::Visibility::Nested;
    };
  }];
  let assemblyFormat = "(`exported` $exported^)? $sym_name $type attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_TableOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema.
- **CN**: 该 TableGen 代码块将 `WasmSSA_TableOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式。

### Lines 575-598
````tablegen
def WasmSSA_TableImportOp : WasmSSA_Op<"import_table", [Symbol, ImportOpInterface]> {
  let summary = "Importing a table";
  let description = [{
    Import a table from another module.

     Example:

     ```mlir
     // Import the table `table` from `my_module` as @table_0
     wasmssa.import_table "table" from "my_module" as @table_0 {type = !wasmssa<tabletype !wasmssa.funcref [2:]>}
     ```
    }];
  let arguments = (ins SymbolNameAttr: $sym_name,
                     StrAttr: $moduleName,
                     StrAttr: $importName,
                     WasmSSA_TableTypeAttr: $type);
  let extraClassDeclaration = [{
    bool isDeclaration() const { return true; }
    ::mlir::SymbolTable::Visibility getVisibility() {
       return ::mlir::SymbolTable::Visibility::Nested;
    };
  }];
  let assemblyFormat = "$importName `from` $moduleName `as` $sym_name attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_TableImportOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `WasmSSA_TableImportOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 600-607
````tablegen
def WasmSSA_ReturnOp : WasmSSA_Op<"return", [Terminator]> {
  let summary = "Return from the current function frame";
  let arguments = (ins Variadic<WasmSSA_ValType>:  $operands);
  let assemblyFormat = "attr-dict ($operands^ `:` type($operands))?";
  let builders = [
    OpBuilder<(ins)>
  ];
}
````
- **EN**: This TableGen block defines `WasmSSA_ReturnOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ReturnOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档。

### Lines 608-608
````tablegen
// ---- Numeric ops
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 611-619
````tablegen
class WasmSSA_BinaryNumericalOp<string mnemonic, string summaryStr, string descStr,
                       list<Type> validOpTypes> :
  WasmSSA_Op<mnemonic, [AllTypesMatch<["lhs", "rhs", "result"]>]> {
  let summary = summaryStr;
  let description = descStr;
  let arguments = (ins AnyTypeOf<validOpTypes>:$lhs, AnyTypeOf<validOpTypes>:$rhs);
  let results = (outs AnyTypeOf<validOpTypes>:$result);
  let assemblyFormat = "$lhs $rhs `:` type($lhs) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_BinaryNumericalOp` as a `class` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_BinaryNumericalOp` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 621-628
````tablegen
def WasmSSA_AddOp : WasmSSA_BinaryNumericalOp<"add",
    "Sum two values",
    [{Example:

     ```mlir
     %a = wasmssa.add %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_AddOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_AddOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 629-629
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 631-638
````tablegen
def WasmSSA_AndOp : WasmSSA_BinaryNumericalOp<"and",
    "Compute the bitwise AND between two values",
    [{Example:

     ```mlir
     %a = wasmssa.and %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_AndOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_AndOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 639-639
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 641-648
````tablegen
def WasmSSA_DivOp : WasmSSA_BinaryNumericalOp<"div",
    "Division between floating point values",
    [{Example:

     ```mlir
     %a = wasmssa.div %b %c : f32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_DivOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_DivOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 649-649
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 651-658
````tablegen
def WasmSSA_DivUIOp : WasmSSA_BinaryNumericalOp<"div_ui",
    "Divide values interpreted as unsigned int",
    [{Example:

     ```mlir
     %a = wasmssa.div_ui %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_DivUIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_DivUIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 659-659
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 661-668
````tablegen
def WasmSSA_DivSIOp : WasmSSA_BinaryNumericalOp<"div_si",
    "Divide values interpreted as signed int",
    [{Example:

     ```mlir
     %a = wasmssa.div_si %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_DivSIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_DivSIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 669-669
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 671-678
````tablegen
def WasmSSA_MulOp : WasmSSA_BinaryNumericalOp<"mul",
    "Multiply two values",
    [{Example:

     ```mlir
     %a = wasmssa.mul %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_MulOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_MulOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 679-679
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 681-688
````tablegen
def WasmSSA_OrOp : WasmSSA_BinaryNumericalOp<"or",
    "Compute the bitwise OR of two values",
    [{Example:

     ```mlir
     %a = wasmssa.or %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_OrOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_OrOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 689-689
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 691-698
````tablegen
def WasmSSA_SubOp : WasmSSA_BinaryNumericalOp<"sub",
    "Subtract two values",
    [{Example:

     ```mlir
     %a = wasmssa.sub %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_SubOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_SubOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 699-699
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 701-708
````tablegen
def WasmSSA_RemUIOp : WasmSSA_BinaryNumericalOp<"rem_ui",
    "Calculate the remainder of dividing two integer values as an unsigned integer",
    [{Example:

     ```mlir
     %a = wasmssa.rem_ui %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_RemUIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_RemUIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 709-709
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 711-718
````tablegen
def WasmSSA_RemSIOp : WasmSSA_BinaryNumericalOp<"rem_si",
    "Calculate the remainder of dividing two integer values as signed integer",
    [{Example:

     ```mlir
     %a = wasmssa.rem_si %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_RemSIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_RemSIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 719-719
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 721-728
````tablegen
def WasmSSA_XOrOp : WasmSSA_BinaryNumericalOp<"xor",
    "Compute the bitwise XOR of two values",
    [{Example:

     ```mlir
     %a = wasmssa.xor %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_XOrOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_XOrOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 729-729
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 731-738
````tablegen
def WasmSSA_MinOp : WasmSSA_BinaryNumericalOp<"min",
    "Compute the minimum of two floating point values.",
    [{Example:

     ```mlir
     %a = wasmssa.min %b %c : f32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_MinOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_MinOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 739-739
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 741-748
````tablegen
def WasmSSA_MaxOp : WasmSSA_BinaryNumericalOp<"max",
    "Compute the minimum of two floating point values.",
    [{Example:

     ```mlir
     %a = wasmssa.max %b %c : f32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_MaxOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_MaxOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 749-749
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 751-758
````tablegen
def WasmSSA_CopySignOp : WasmSSA_BinaryNumericalOp<"copysign",
    "Copy sign from one floating point value to the other.",
    [{Example:

     ```mlir
     %a = wasmssa.copysign %b %c : f32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_CopySignOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_CopySignOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 759-759
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 761-769
````tablegen
class WasmSSA_BinaryComparisonOp<string mnemonic, string summaryStr, string descStr,
                       list<Type> validOpTypes> :
  WasmSSA_Op<mnemonic, [AllTypesMatch<["lhs", "rhs"]>]> {
  let summary = summaryStr;
  let description = descStr;
  let arguments = (ins AnyTypeOf<validOpTypes>:$lhs, AnyTypeOf<validOpTypes>:$rhs);
  let results = (outs I32:$result);
  let assemblyFormat = "$lhs $rhs `:` type($lhs) `->` type($result) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_BinaryComparisonOp` as a `class` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_BinaryComparisonOp` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 771-778
````tablegen
def WasmSSA_EqOp : WasmSSA_BinaryComparisonOp<"eq",
    "Check if two values are equal",
    [{Example:

     ```mlir
     %a = wasmssa.eq %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_EqOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_EqOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 779-779
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 781-788
````tablegen
def WasmSSA_NeOp : WasmSSA_BinaryComparisonOp<"ne",
    "Check if two values are different",
    [{Example:

     ```mlir
     %a = wasmssa.ne %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_NeOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_NeOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 789-789
````tablegen
    [WasmSSA_NumericType]>{}
````
- **EN**: This section focuses on [wasmssa_numerictype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_NumericType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 791-798
````tablegen
def WasmSSA_LtSIOp : WasmSSA_BinaryComparisonOp<"lt_si",
    "Check if a signed integer value is less than another",
    [{Example:

     ```mlir
     %a = wasmssa.lt_si %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_LtSIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LtSIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 799-799
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 801-808
````tablegen
def WasmSSA_LtUIOp : WasmSSA_BinaryComparisonOp<"lt_ui",
    "Check if an unsigned integer value is less than another",
    [{Example:

     ```mlir
     %a = wasmssa.lt_ui %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_LtUIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LtUIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 809-809
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 811-818
````tablegen
def WasmSSA_LeSIOp : WasmSSA_BinaryComparisonOp<"le_si",
    "Check if a signed integer value is less or equal to another",
    [{Example:

     ```mlir
     %a = wasmssa.le_si %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_LeSIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LeSIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 819-819
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 821-828
````tablegen
def WasmSSA_LeUIOp : WasmSSA_BinaryComparisonOp<"le_ui",
    "Check if an unsigned integer value is less or equal to another",
    [{Example:

     ```mlir
     %a = wasmssa.le_ui %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_LeUIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LeUIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 829-829
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 831-838
````tablegen
def WasmSSA_GtSIOp : WasmSSA_BinaryComparisonOp<"gt_si",
    "Check if a signed integer value is greater than another",
    [{Example:

     ```mlir
     %a = wasmssa.gt_si %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_GtSIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GtSIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 839-839
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 841-848
````tablegen
def WasmSSA_GtUIOp : WasmSSA_BinaryComparisonOp<"gt_ui",
    "Check if an unsigned integer value is greater than another",
    [{Example:

     ```mlir
     %a = wasmssa.gt_ui %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_GtUIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GtUIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 849-849
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 851-858
````tablegen
def WasmSSA_GeSIOp : WasmSSA_BinaryComparisonOp<"ge_si",
    "Check if a signed integer value is greater or equal to another",
    [{Example:

     ```mlir
     %a = wasmssa.ge_si %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_GeSIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GeSIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 859-859
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 861-868
````tablegen
def WasmSSA_GeUIOp : WasmSSA_BinaryComparisonOp<"ge_ui",
    "Check if an unsigned integer value is greater or equal to another",
    [{Example:

     ```mlir
     %a = wasmssa.ge_ui %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_GeUIOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GeUIOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 869-869
````tablegen
    [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 871-878
````tablegen
def WasmSSA_LtOp : WasmSSA_BinaryComparisonOp<"lt",
    "Check if a float value is less than another",
    [{Example:

     ```mlir
     %a = wasmssa.lt %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_LtOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LtOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 879-879
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 881-888
````tablegen
def WasmSSA_LeOp : WasmSSA_BinaryComparisonOp<"le",
    "Check if a float value is less or equal to another",
    [{Example:

     ```mlir
     %a = wasmssa.le %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_LeOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_LeOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 889-889
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 891-898
````tablegen
def WasmSSA_GtOp : WasmSSA_BinaryComparisonOp<"gt",
    "Check if a float value is greater than another",
    [{Example:

     ```mlir
     %a = wasmssa.gt %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_GtOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GtOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 899-899
````tablegen
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 901-908
````tablegen
def WasmSSA_GeOp : WasmSSA_BinaryComparisonOp<"ge",
    "Check if a float value is greater or equal to another",
    [{Example:

     ```mlir
     %a = wasmssa.ge %b %c : i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_GeOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_GeOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 909-911
````tablegen
    [WasmSSA_FPType]>{}

// Integer shift and rotate operations.
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 912-919
````tablegen
class WasmSSA_ShiftRotateOp<string mnemonic, string summaryStr, string descStr> :
  WasmSSA_Op<mnemonic, [AllTypesMatch<["val", "bits", "result"]>]> {
  let summary = summaryStr;
  let description = descStr;
  let arguments = (ins WasmSSA_IntegerType:$val, WasmSSA_IntegerType:$bits);
  let results = (outs WasmSSA_IntegerType:$result);
  let assemblyFormat = "$val `by` $bits `bits` `:` type($val) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_ShiftRotateOp` as a `class` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ShiftRotateOp` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 921-924
````tablegen
def WasmSSA_ShLOp : WasmSSA_ShiftRotateOp<"shl",
    [{Consume an integer and an integer shift amount. The first
    integer shall be shifted left by N bits, where N is the value of the second
    integer.}],
````
- **EN**: This TableGen block defines `WasmSSA_ShLOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ShLOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 925-931
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.shl %b by %c bits : i64
     ```
    }]
    >{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 933-940
````tablegen
def WasmSSA_ShRSOp : WasmSSA_ShiftRotateOp<"shr_s",
    [{Arithmetic right shift.

    Consume an integer and an integer shift amount. The first
    integer shall be shifted right by N bits, where N is the value of the
    second integer.

    Vacated bits on the left shall be filled with the sign bit.}],
````
- **EN**: This TableGen block defines `WasmSSA_ShRSOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ShRSOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 941-947
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.shr_s %b by %c bits : i64
     ```
    }]
    >{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 949-956
````tablegen
def WasmSSA_ShRUOp : WasmSSA_ShiftRotateOp<"shr_u",
    [{Logical right shift.

    Consume an integer, and an integer shift amount. The first
    integer shall be shifted right by N bits, where N is the value of the
    second integer.

    Vacated bits on the left shall be filled with zeroes.}],
````
- **EN**: This TableGen block defines `WasmSSA_ShRUOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ShRUOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 957-963
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.shr_u %b by %c bits : i64
     ```
    }]
    >{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 965-970
````tablegen
def WasmSSA_RotlOp : WasmSSA_ShiftRotateOp<"rotl",
    [{Rotate left.

    Consume an integer and an integer rotate. The first
    integer shall be rotated left by N bits, where N is the value of the
    second integer.}],
````
- **EN**: This TableGen block defines `WasmSSA_RotlOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_RotlOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 971-977
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.rotl %b by %c bits : i64
     ```
    }]
    >{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 979-984
````tablegen
def WasmSSA_RotrOp : WasmSSA_ShiftRotateOp<"rotr",
    [{Rotate right.

    Consume an integer, and an integer rotate. The first
    integer shall be rotated right by N bits, where N is the value of the
    second integer.}],
````
- **EN**: This TableGen block defines `WasmSSA_RotrOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_RotrOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 985-991
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.rotr %b by %c bits : i64
     ```
    }]
    >{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 993-1002
````tablegen
class WasmSSA_ConversionOp<string mnemonic, string summaryStr, string descStr,
                       list<Type> ValidInputTypes,
                       list<Type> ValidOutputTypes> :
    WasmSSA_Op<mnemonic> {
  let summary = summaryStr;
  let description = descStr;
  let arguments = (ins AnyTypeOf<ValidInputTypes>:$input);
  let results = (outs AnyTypeOf<ValidOutputTypes>:$result);
  let assemblyFormat = "$input `:` type($input) `to` type($result)  attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_ConversionOp` as a `class` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ConversionOp` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1004-1007
````tablegen
def WasmSSA_ConvertUOp : WasmSSA_ConversionOp<"convert_u",
    [{Convert integer, interpreted as binary encoded positive value, to floating-point value.

    Consume an integer and produces a floating point value containing the rounded value of the original operand. Rounding is round to nearest, tie to even.}],
````
- **EN**: This TableGen block defines `WasmSSA_ConvertUOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ConvertUOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1008-1015
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.convert_u %b : i32 to f64
     ```
    }],
    [WasmSSA_IntegerType],
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1017-1020
````tablegen
def WasmSSA_ConvertSOp : WasmSSA_ConversionOp<"convert_s",
    [{Convert integer interpreted as 2's complement signed value to floating-point value.

    Consume an integer and produces a floating point value containing the rounded value of the original operand. Rounding is round to nearest, tie to even.}],
````
- **EN**: This TableGen block defines `WasmSSA_ConvertSOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ConvertSOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1021-1028
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.convert_s %b : i32 to f64
     ```
    }],
    [WasmSSA_IntegerType],
    [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [{example, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[{Example”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1030-1037
````tablegen
def WasmSSA_DemoteOp : WasmSSA_ConversionOp<"demote",
    "Convert a f64 value to f32",
    [{Example:

     ```mlir
     %a = wasmssa.demote %b : f64 to f32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_DemoteOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_DemoteOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1038-1039
````tablegen
    [F64],
    [F32]>{}
````
- **EN**: This section focuses on [f64],, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[F64],”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1041-1052
````tablegen
def WasmSSA_ExtendSI32Op : WasmSSA_Op<"extend_i32_s">{
  let summary = [{Sign extend i32 to i64.}];
  let description =  [{Example:

     ```mlir
     %a = wasmssa.extend_i32_s %b to i64
     ```
    }];
  let arguments = (ins I32:$input);
  let results = (outs I64:$result);
  let assemblyFormat = "$input `to` type($result)  attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_ExtendSI32Op` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ExtendSI32Op` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1054-1065
````tablegen
def WasmSSA_ExtendUI32Op : WasmSSA_Op<"extend_i32_u">{
  let summary = [{Zero extend i32 to i64.}];
  let description = [{Example:

     ```mlir
     %a = wasmssa.extend_i32_s %b to i64
     ```
    }];
  let arguments = (ins I32:$input);
  let results = (outs I64:$result);
  let assemblyFormat = "$input `to` type($result)  attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_ExtendUI32Op` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ExtendUI32Op` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1067-1089
````tablegen
def WasmSSA_ExtendLowBitsSOp : WasmSSA_Op<"extend", [AllTypesMatch<["input", "result"]>]> {
  let summary = "";
  let description = [{
  Extend low bytes of a value to fit a given width.
  For instance, signed extension from 8 low bits of the 32-bits integer value
  254 (0x000000FE) would produce the value -2 (0xFFFFFFFE).

  This corresponds to the `extendnn` instruction of Wasm, which shouldn't be
  confused with the `extend_inn` Wasm instruction, for which all input bits
  are used and widened to wider output type.
  In this operation, input and output types are the same.

  Example:

  ```mlir
  %a = wasmssa.extend 16 low bits from %[[VAL_0]]: i64
  ```
  }];
  let arguments = (ins WasmSSA_IntegerType:$input, Builtin_IntegerAttr: $bitsToTake);
  let results = (outs WasmSSA_IntegerType: $result);
  let hasVerifier = 1;
  let assemblyFormat = "$bitsToTake `low` `bits` `from` $input `:` type($input) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_ExtendLowBitsSOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ExtendLowBitsSOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1091-1098
````tablegen
def WasmSSA_PromoteOp : WasmSSA_ConversionOp<"promote",
    "Get f64 representation of a f32 value.",
    [{Example:

     ```mlir
     %a = wasmssa.promote %b : f32 to f64
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_PromoteOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_PromoteOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1099-1100
````tablegen
    [Builtin_Float32],
    [Builtin_Float64]>{}
````
- **EN**: This section focuses on [builtin_float32],, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[Builtin_Float32],”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1102-1109
````tablegen
def WasmSSA_WrapOp : WasmSSA_ConversionOp<"wrap",
    "Cast an i64 to i32 by using a wrapping mechanism: y = x mod 2^32",
    [{Example:

     ```mlir
     %a = wasmssa.wrap %b : i64 to i32
     ```
    }],
````
- **EN**: This TableGen block defines `WasmSSA_WrapOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_WrapOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1110-1114
````tablegen
    [I64],
    [I32]>{}

// Reinterpret ops are basically all one-offs. They all have an unique,
// type-postfixed opcode, and support exactly one input and output type.
````
- **EN**: This section focuses on [i64],, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[I64],”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1115-1117
````tablegen
def WasmSSA_ReinterpretOp : WasmSSA_ConversionOp<"reinterpret",
    [{Reinterpret the value represented by a bit vector by
      bit-casting it to another type of same representation width.}],
````
- **EN**: This TableGen block defines `WasmSSA_ReinterpretOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ReinterpretOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1118-1127
````tablegen
    [{Example:

     ```mlir
     %a = wasmssa.reinterpret %b : f32 as i32
     ```
    }],
    [WasmSSA_NumericType], [WasmSSA_NumericType]>{
    let assemblyFormat = "$input `:` type($input) `as` type($result) attr-dict";
    let hasVerifier = 1;
}
````
- **EN**: This block groups callable interfaces such as `type`, indicating how `WasmSSAOps` is queried or updated.
- **CN**: 该代码块聚合了 `type` 等可调用接口，展示了如何查询或更新 `WasmSSAOps`。

### Lines 1129-1139
````tablegen
class WasmSSA_UnaryNumericalOp<string mnemonic,
                            string summaryStr,
                            string descStr,
                            list<Type> validOpTypes> :
  WasmSSA_Op<mnemonic, [AllTypesMatch<["src", "result"]>]> {
    let summary = summaryStr;
    let description = descStr;
    let arguments = (ins AnyTypeOf<validOpTypes>:$src);
    let results = (outs AnyTypeOf<validOpTypes>:$result);
    let assemblyFormat = "$src`:` type($src) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_UnaryNumericalOp` as a `class` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_UnaryNumericalOp` 定义为 `class` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1141-1148
````tablegen
def WasmSSA_AbsOp : WasmSSA_UnaryNumericalOp<"abs",
                                       "Floating point absolute value",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.abs %b : f32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_AbsOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_AbsOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1149-1149
````tablegen
                                       [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1151-1158
````tablegen
def WasmSSA_CeilOp : WasmSSA_UnaryNumericalOp<"ceil",
                                       "Ceil rounding of floating point value",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.ceil %b : f32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_CeilOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_CeilOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1159-1159
````tablegen
                                       [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1161-1168
````tablegen
def WasmSSA_FloorOp : WasmSSA_UnaryNumericalOp<"floor",
                                       "Floor rounding of floating point value",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.floor %b : f32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_FloorOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_FloorOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1169-1169
````tablegen
                                       [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1171-1178
````tablegen
def WasmSSA_NegOp : WasmSSA_UnaryNumericalOp<"neg",
                                       "Floating point negation",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.neg %b : f32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_NegOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_NegOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1179-1179
````tablegen
                                       [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1181-1188
````tablegen
def WasmSSA_SqrtOp : WasmSSA_UnaryNumericalOp<"sqrt",
                                       "Floating point square root",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.sqrt %b : f32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_SqrtOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_SqrtOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1189-1189
````tablegen
                                       [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1191-1198
````tablegen
def WasmSSA_TruncOp : WasmSSA_UnaryNumericalOp<"trunc",
                                       "Trunc of floating point value",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.trunc %b : f32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_TruncOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_TruncOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1199-1199
````tablegen
                                       [WasmSSA_FPType]>{}
````
- **EN**: This section focuses on [wasmssa_fptype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_FPType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1201-1208
````tablegen
def WasmSSA_CtzOp : WasmSSA_UnaryNumericalOp<"ctz",
                                       "Count trailing zeroes of an integer",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.ctz %b : i32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_CtzOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_CtzOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1209-1209
````tablegen
                                       [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1211-1218
````tablegen
def WasmSSA_ClzOp : WasmSSA_UnaryNumericalOp<"clz",
                                       "Count leading zeroes of an integer",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.clz %b : i32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_ClzOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_ClzOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1219-1219
````tablegen
                                       [WasmSSA_IntegerType]>{}
````
- **EN**: This section focuses on [wasmssa_integertype]>{}, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“[WasmSSA_IntegerType]>{}”这一主题，把相关声明与辅助接口组织在一起。

### Lines 1221-1233
````tablegen
def WasmSSA_EqzOp : WasmSSA_Op<"eqz", []> {
  let summary = "Check if the given value is equal to zero";
  let description =
    [{Example:

     ```mlir
     %a = wasmssa.eqz %b : i64 -> i32
     ```
    }];
  let arguments = (ins WasmSSA_IntegerType: $input);
  let results = (outs I32: $result);
  let assemblyFormat = "$input`:` type($input) `->` type($result) attr-dict";
}
````
- **EN**: This TableGen block defines `WasmSSA_EqzOp` as a `def` record for `WasmSSAOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `WasmSSA_EqzOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1236-1243
````tablegen
def WasmSSA_PopCntOp : WasmSSA_UnaryNumericalOp<"popcnt",
                                       "Population count of an integer.",
                                        [{Example:

                                         ```mlir
                                         %a = wasmssa.popcnt %b : i32
                                         ```
                                        }],
````
- **EN**: This TableGen block defines `WasmSSA_PopCntOp` as a `def` record for `WasmSSAOps`.
- **CN**: 该 TableGen 代码块将 `WasmSSA_PopCntOp` 定义为 `def` 记录，用于描述 `WasmSSAOps` 相关的声明式信息。

### Lines 1244-1247
````tablegen
                                       [WasmSSA_IntegerType]>{}


#endif // WasmSSA_OPS
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Custom assembly syntax specification
  **CN**: 自定义汇编语法规格
- **EN**: Operand/result schema definition
  **CN**: 操作数/结果模式定义
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/Dialect/WasmSSA/IR/WasmSSABase.td
- mlir/Dialect/WasmSSA/IR/WasmSSATypes.td
- mlir/Dialect/WasmSSA/IR/WasmSSAInterfaces.td
- mlir/Interfaces/FunctionInterfaces.td
- mlir/Interfaces/InferTypeOpInterface.td
- mlir/IR/BuiltinAttributeInterfaces.td
- mlir/IR/SymbolInterfaces.td
- WasmSSA_BlockOp builds on WasmSSA_BlockLikeOp<
- WasmSSA_LoopOp builds on WasmSSA_BlockLikeOp<
- WasmSSA_BlockReturnOp builds on WasmSSA_Op<"block_return", [Terminator,
- WasmSSA_BranchIfOp builds on WasmSSA_Op<"branch_if", [
- WasmSSA_ConstOp builds on WasmSSA_Op<"const", [
- WasmSSA_FuncOp builds on WasmSSA_Op<"func", [
- WasmSSA_FuncCallOp builds on WasmSSA_Op<"call">
- WasmSSA_FuncImportOp builds on WasmSSA_Op<"import_func", [
- WasmSSA_GlobalOp builds on WasmSSA_Op<"global", [
- WasmSSA_GlobalImportOp builds on WasmSSA_Op<"import_global", [
- WasmSSA_GlobalGetOp builds on WasmSSA_Op<"global_get", [DeclareOpInterfaceMethods<SymbolUserOpInterface>,
- WasmSSA_IfOp builds on WasmSSA_Op<"if", [Terminator,
- WasmSSA_LocalOp builds on WasmSSA_Op<"local", [
