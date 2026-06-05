# SPIRVMemoryOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVMemoryOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR SPIRVMemoryOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file contains memory ops for the SPIR-V dialect. It corresponds.
- **用途（CN）**: 为 MLIR 的 SPIRVMemoryOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
````tablegen
//===-- SPIRVMemoryOps.td - MLIR SPIR-V Memory Ops ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains memory ops for the SPIR-V dialect. It corresponds
// to "3.32.8. Memory Instructions" of the SPIR-V spec.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_MEMORY_OPS
#define MLIR_DIALECT_SPIRV_IR_MEMORY_OPS

include "mlir/Dialect/SPIRV/IR/SPIRVBase.td"
include "mlir/Interfaces/AlignmentAttrInterface.td"


// -----
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 23-80
````tablegen
def SPIRV_AccessChainOp : SPIRV_Op<"AccessChain", [Pure]> {
  let summary = "Create a pointer into a composite object.";

  let description = [{
    Result Type must be an OpTypePointer. Its Type operand must be the type
    reached by walking the Base’s type hierarchy down to the last provided
    index in Indexes, and its Storage Class operand must be the same as the
    Storage Class of Base.

    Base must be a pointer, pointing to the base of a composite object.

    Indexes walk the type hierarchy to the desired depth, potentially down
    to scalar granularity. The first index in Indexes will select the top-
    level member/element/component/element of the base composite. All
    composite constituents use zero-based numbering, as described by their
    OpType… instruction. The second index will apply similarly to that
    result, and so on. Once any non-composite type is reached, there must be
    no remaining (unused) indexes.

     Each index in Indexes

    - must be a scalar integer type,

    - is treated as a signed count, and

    - must be an OpConstant when indexing into a structure.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    %0 = "spirv.Constant"() { value = 1: i32} : () -> i32
    %1 = spirv.Variable : !spirv.ptr<!spirv.struct<f32, !spirv.array<4xf32>>, Function>
    %2 = spirv.AccessChain %1[%0] : !spirv.ptr<!spirv.struct<f32, !spirv.array<4xf32>>, Function> -> !spirv.ptr<!spirv.array<4xf32>, Function>
    %3 = spirv.Load "Function" %2 ["Volatile"] : !spirv.array<4xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_AnyPtr:$base_ptr,
    Variadic<SPIRV_Integer>:$indices
  );

  let results = (outs
    SPIRV_AnyPtr:$component_ptr
  );

  let builders = [OpBuilder<(ins "Value":$basePtr, "ValueRange":$indices)>];

  let hasCanonicalizer = 1;

  let hasCustomAssemblyFormat = 0;

  let assemblyFormat = [{
    $base_ptr `[` $indices `]` attr-dict `:` type($base_ptr) `,` type($indices) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_AccessChainOp` as a `def` record for `SPIRVMemoryOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_AccessChainOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 81-81
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 84-134
````tablegen
def SPIRV_CopyMemoryOp : SPIRV_Op<"CopyMemory", [DeclareOpInterfaceMethods<AlignmentAttrOpInterface>]> {
  let summary = [{
    Copy from the memory pointed to by Source to the memory pointed to by
    Target. Both operands must be non-void pointers and having the same `<id>`
    Type operand in their OpTypePointer type declaration.  Matching Storage
    Class is not required.  The amount of memory copied is the size of the
    type pointed to. The copied type must have a fixed size; i.e., it must
    not be, nor include, any OpTypeRuntimeArray types.
  }];

  let description = [{
    If present, any Memory Operands must begin with a memory operand
    literal. If not present, it is the same as specifying the memory operand
    None. Before version 1.4, at most one memory operands mask can be
    provided. Starting with version 1.4 two masks can be provided, as
    described in Memory Operands. If no masks or only one mask is present,
    it applies to both Source and Target. If two masks are present, the
    first applies to Target and cannot include MakePointerVisible, and the
    second applies to Source and cannot include MakePointerAvailable.

    <!-- End of AutoGen section -->

    ```
    copy-memory-op ::= `spirv.CopyMemory ` storage-class ssa-use
                       storage-class ssa-use
                       (`[` memory-access `]` (`, [` memory-access `]`)?)?
                       ` : ` spirv-element-type
    ```

    #### Example:

    ```mlir
    %0 = spirv.Variable : !spirv.ptr<f32, Function>
    %1 = spirv.Variable : !spirv.ptr<f32, Function>
    spirv.CopyMemory "Function" %0, "Function" %1 : f32
    ```
  }];

  let arguments = (ins
    SPIRV_AnyPtr:$target,
    SPIRV_AnyPtr:$source,
    OptionalAttr<SPIRV_MemoryAccessAttr>:$memory_access,
    OptionalAttr<IntValidAlignment<I32Attr>>:$alignment,
    OptionalAttr<SPIRV_MemoryAccessAttr>:$source_memory_access,
    OptionalAttr<IntValidAlignment<I32Attr>>:$source_alignment
  );

  let results = (outs);

  let autogenSerialization = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_CopyMemoryOp` as a `def` record for `SPIRVMemoryOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_CopyMemoryOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 135-135
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 138-183
````tablegen
def SPIRV_InBoundsPtrAccessChainOp : SPIRV_Op<"InBoundsPtrAccessChain", [Pure]> {
  let summary = [{
    Has the same semantics as OpPtrAccessChain, with the addition that the
    resulting pointer is known to point within the base object.
  }];

  let description = [{


    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    func @inbounds_ptr_access_chain(%arg0: !spirv.ptr<f32, CrossWorkgroup>, %arg1 : i64) -> () {
      %0 = spirv.InBoundsPtrAccessChain %arg0[%arg1] : !spirv.ptr<f32, CrossWorkgroup>, i64 -> !spirv.ptr<f32, CrossWorkgroup>
      ...
    }
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[SPIRV_C_Addresses]>
  ];

  let arguments = (ins
    SPIRV_AnyPtr:$base_ptr,
    SPIRV_Integer:$element,
    Variadic<SPIRV_Integer>:$indices
  );

  let results = (outs
    SPIRV_AnyPtr:$result
  );

  let builders = [OpBuilder<(ins "Value":$basePtr, "Value":$element, "ValueRange":$indices)>];

  let hasCustomAssemblyFormat = 0;

  let assemblyFormat = [{
    $base_ptr `[` $element ($indices^)? `]` attr-dict `:` type($base_ptr) `,` type($element) (`,` type($indices)^)? `->` type($result)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_InBoundsPtrAccessChainOp` as a `def` record for `SPIRVMemoryOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_InBoundsPtrAccessChainOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 184-184
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 187-237
````tablegen
def SPIRV_LoadOp : SPIRV_Op<"Load", [DeclareOpInterfaceMethods<AlignmentAttrOpInterface>]> {
  let summary = "Load through a pointer.";

  let description = [{
    Result Type is the type of the loaded object. It must be a type with
    fixed size; i.e., it cannot be, nor include, any OpTypeRuntimeArray
    types.

    Pointer is the pointer to load through.  Its type must be an
    OpTypePointer whose Type operand is the same as Result Type.

    If present, any Memory Operands must begin with a memory operand
    literal. If not present, it is the same as specifying the memory operand
    None.

    <!-- End of AutoGen section -->

    ```
    memory-access ::= `"None"` | `"Volatile"` | `"Aligned", ` integer-literal
                    | `"NonTemporal"`

    load-op ::= ssa-id ` = spirv.Load ` storage-class ssa-use
                (`[` memory-access `]`)? ` : ` spirv-element-type
    ```

    #### Example:

    ```mlir
    %0 = spirv.Variable : !spirv.ptr<f32, Function>
    %1 = spirv.Load "Function" %0 : f32
    %2 = spirv.Load "Function" %0 ["Volatile"] : f32
    %3 = spirv.Load "Function" %0 ["Aligned", 4] : f32
    ```
  }];

  let arguments = (ins
    SPIRV_AnyPtr:$ptr,
    OptionalAttr<SPIRV_MemoryAccessAttr>:$memory_access,
    OptionalAttr<IntValidAlignment<I32Attr>>:$alignment
  );

  let results = (outs
    SPIRV_Type:$value
  );

  let builders = [
    OpBuilder<(ins "Value":$basePtr,
      CArg<"MemoryAccessAttr", "{}">:$memory_access,
      CArg<"IntegerAttr", "{}">:$alignment)>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_LoadOp` as a `def` record for `SPIRVMemoryOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_LoadOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 238-238
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 241-311
````tablegen
def SPIRV_PtrAccessChainOp : SPIRV_Op<"PtrAccessChain", [Pure]> {
  let summary = [{
    Has the same semantics as OpAccessChain, with the addition of the
    Element operand.
  }];

  let description = [{
    Element is used to do an initial dereference of Base: Base is treated as
    the address of an element in an array, and a new element address is
    computed from Base and Element to become the OpAccessChain Base to
    dereference as per OpAccessChain. This computed Base has the same type
    as the originating Base.

    To compute the new element address, Element is treated as a signed count
    of elements E, relative to the original Base element B, and the address
    of element B + E is computed using enough precision to avoid overflow
    and underflow. For objects in the Uniform, StorageBuffer, or
    PushConstant storage classes, the element's address or location is
    calculated using a stride, which will be the Base-type's Array Stride if
    the Base type is decorated with ArrayStride. For all other objects, the
    implementation calculates the element's address or location.

    With one exception, undefined behavior results when B + E is not an
    element in the same array (same innermost array, if array types are
    nested) as B. The exception being when B + E = L, where L is the length
    of the array: the address computation for element L is done with the
    same stride as any other B + E computation that stays within the array.

    Note: If Base is typed to be a pointer to an array and the desired
    operation is to select an element of that array, OpAccessChain should be
    directly used, as its first Index selects the array element.

    <!-- End of AutoGen section -->

    #### Example:

    ```mlir
    func @ptr_access_chain(%arg0: !spirv.ptr<f32, CrossWorkgroup>, %arg1 : i64) -> () {
      %0 = spirv.PtrAccessChain %arg0[%arg1] : !spirv.ptr<f32, CrossWorkgroup>, i64 -> !spirv.ptr<f32, CrossWorkgroup>
      ...
    }
    ```
  }];

  let availability = [
    MinVersion<SPIRV_V_1_0>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[]>,
    Capability<[
      SPIRV_C_Addresses, SPIRV_C_PhysicalStorageBufferAddresses,
      SPIRV_C_VariablePointers, SPIRV_C_VariablePointersStorageBuffer]>
  ];

  let arguments = (ins
    SPIRV_AnyPtr:$base_ptr,
    SPIRV_Integer:$element,
    Variadic<SPIRV_Integer>:$indices
  );

  let results = (outs
    SPIRV_AnyPtr:$result
  );

  let builders = [OpBuilder<(ins "Value":$basePtr, "Value":$element, "ValueRange":$indices)>];

  let hasCustomAssemblyFormat = 0;

  let assemblyFormat = [{
    $base_ptr `[` $element ($indices^)? `]` attr-dict `:` type($base_ptr) `,` type($element) (`,` type($indices)^)? `->` type($result)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_PtrAccessChainOp` as a `def` record for `SPIRVMemoryOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_PtrAccessChainOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 312-312
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 315-364
````tablegen
def SPIRV_StoreOp : SPIRV_Op<"Store", [DeclareOpInterfaceMethods<AlignmentAttrOpInterface>]> {
  let summary = "Store through a pointer.";

  let description = [{
    Pointer is the pointer to store through.  Its type must be an
    OpTypePointer whose Type operand is the same as the type of Object.

    Object is the object to store.

    If present, any Memory Operands must begin with a memory operand
    literal. If not present, it is the same as specifying the memory operand
    None.

    <!-- End of AutoGen section -->

    ```
    store-op ::= `spirv.Store ` storage-class ssa-use `, ` ssa-use `, `
                  (`[` memory-access `]`)? `:` spirv-element-type
    ```

    #### Example:

    ```mlir
    %0 = spirv.Variable : !spirv.ptr<f32, Function>
    %1 = spirv.FMul ... : f32
    spirv.Store "Function" %0, %1 : f32
    spirv.Store "Function" %0, %1 ["Volatile"] : f32
    spirv.Store "Function" %0, %1 ["Aligned", 4] : f32
    ```
  }];

  let arguments = (ins
    SPIRV_AnyPtr:$ptr,
    SPIRV_Type:$value,
    OptionalAttr<SPIRV_MemoryAccessAttr>:$memory_access,
    OptionalAttr<IntValidAlignment<I32Attr>>:$alignment
  );

  let results = (outs);

  let builders = [
    OpBuilder<(ins "Value":$ptr, "Value":$value,
      CArg<"ArrayRef<NamedAttribute>", "{}">:$namedAttrs),
    [{
      $_state.addOperands(ptr);
      $_state.addOperands(value);
      $_state.addAttributes(namedAttrs);
    }]>
  ];
}
````
- **EN**: This TableGen block defines `SPIRV_StoreOp` as a `def` record for `SPIRVMemoryOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_StoreOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 365-365
````tablegen
// -----
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 368-432
````tablegen
def SPIRV_VariableOp : SPIRV_Op<"Variable", []> {
  let summary = [{
    Allocate an object in memory, resulting in a pointer to it, which can be
    used with OpLoad and OpStore.
  }];

  let description = [{
    Result Type must be an OpTypePointer. Its Type operand is the type of
    object in memory.

    Storage Class is the Storage Class of the memory holding the object.
    Since the op is used to model function-level variables, the storage class
    must be the `Function` Storage Class.

    Initializer is optional. If Initializer is present, it will be the
    initial value of the variable's memory content. Initializer must be an
    `<id>` from a constant instruction or a global (module scope) OpVariable
    instruction. Initializer must have the same type as the type pointed to
    by Result Type.

    From `SPV_KHR_physical_storage_buffer`:
    If an OpVariable's pointee type is a pointer (or array of pointers) in
    PhysicalStorageBuffer storage class, then the variable must be decorated
    with exactly one of AliasedPointer or RestrictPointer.

    <!-- End of AutoGen section -->

    ```
    variable-op ::= ssa-id `=` `spirv.Variable` (`init(` ssa-use `)`)?
                    attribute-dict? `:` spirv-pointer-type
    ```

    where `init` specifies initializer.

    #### Example:

    ```mlir
    %0 = spirv.Constant ...

    %1 = spirv.Variable : !spirv.ptr<f32, Function>
    %2 = spirv.Variable init(%0): !spirv.ptr<f32, Function>

    %3 = spirv.Variable {aliased_pointer} :
      !spirv.ptr<!spirv.ptr<f32, PhysicalStorageBuffer>, Function>
    ```
  }];

  let arguments = (ins
    SPIRV_StorageClassAttr:$storage_class,
    Optional<AnyType>:$initializer
  );

  let results = (outs
    SPIRV_AnyPtr:$pointer
  );

  let extraClassDeclaration = [{
    ::mlir::spirv::PointerType getPointerType() {
      return ::llvm::cast<::mlir::spirv::PointerType>(getType());
    }
    ::mlir::Type getPointeeType() {
      return getPointerType().getPointeeType();
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_VariableOp` as a `def` record for `SPIRVMemoryOps`. It covers operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_VariableOp` 定义为 `def` 记录，用于描述 `SPIRVMemoryOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 433-433
````tablegen
#endif // MLIR_DIALECT_SPIRV_IR_MEMORY_OPS
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

- mlir/Dialect/SPIRV/IR/SPIRVBase.td
- mlir/Interfaces/AlignmentAttrInterface.td
- SPIRV_AccessChainOp builds on SPIRV_Op<"AccessChain", [Pure]>
- SPIRV_CopyMemoryOp builds on SPIRV_Op<"CopyMemory", [DeclareOpInterfaceMethods<AlignmentAttrOpInterface>]>
- SPIRV_InBoundsPtrAccessChainOp builds on SPIRV_Op<"InBoundsPtrAccessChain", [Pure]>
- SPIRV_LoadOp builds on SPIRV_Op<"Load", [DeclareOpInterfaceMethods<AlignmentAttrOpInterface>]>
- SPIRV_PtrAccessChainOp builds on SPIRV_Op<"PtrAccessChain", [Pure]>
- SPIRV_StoreOp builds on SPIRV_Op<"Store", [DeclareOpInterfaceMethods<AlignmentAttrOpInterface>]>
- SPIRV_VariableOp builds on SPIRV_Op<"Variable", []>
