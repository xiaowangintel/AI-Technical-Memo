# TransformTypes.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/IR/TransformTypes.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TransformTypes component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 TransformTypes 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
````tablegen
//===- TransformTypes.td - Transform dialect types ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_IR_TRANSFORMTYPES
#define MLIR_DIALECT_TRANSFORM_IR_TRANSFORMTYPES

include "mlir/IR/AttrTypeBase.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Dialect/Transform/IR/TransformDialect.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 16-24
````tablegen
def Transform_AffineMapParamType : TypeDef<Transform_Dialect, "AffineMapParam",
    [DeclareTypeInterfaceMethods<TransformParamTypeInterface>]> {
  let description = [{
    Transform IR parameter value that can be associated with a list of affine
    map attributes.
  }];
  let mnemonic = "affine_map";
  let assemblyFormat = "";
}
````
- **EN**: This TableGen block defines `Transform_AffineMapParamType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_AffineMapParamType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 26-34
````tablegen
def Transform_AnyOpType : TypeDef<Transform_Dialect, "AnyOp",
    [DeclareTypeInterfaceMethods<TransformHandleTypeInterface>]> {
  let description = [{
    Transform IR handle that can be associated with a list of arbitrary
    Payload IR operations.
  }];
  let mnemonic = "any_op";
  let assemblyFormat = "";
}
````
- **EN**: This TableGen block defines `Transform_AnyOpType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_AnyOpType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 36-43
````tablegen
def Transform_AnyValue : TypeDef<Transform_Dialect, "AnyValue",
    [DeclareTypeInterfaceMethods<TransformValueHandleTypeInterface>]> {
  let description = [{
    Transform IR value that can be associated with a list of Payload IR values.
  }];
  let mnemonic = "any_value";
  let assemblyFormat = "";
}
````
- **EN**: This TableGen block defines `Transform_AnyValue` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_AnyValue` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 45-56
````tablegen
def Transform_OperationType : TypeDef<Transform_Dialect, "Operation",
    [DeclareTypeInterfaceMethods<TransformHandleTypeInterface>]> {
  let description = [{
    Transform IR handle that can be associated with a list of Payload IR
    operations with the specified operation name.
  }];
  let mnemonic = "op";
  let parameters = (ins
    StringRefParameter<"Name of the allowed payload operation">:$operation_name
  );
  let assemblyFormat = "`<` $operation_name `>`";
}
````
- **EN**: This TableGen block defines `Transform_OperationType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_OperationType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 58-66
````tablegen
def Transform_AnyParamType : TypeDef<Transform_Dialect, "AnyParam",
    [DeclareTypeInterfaceMethods<TransformParamTypeInterface>]> {
  let description = [{
    Transform IR value that can be associated with a list of parameters
    of any type.
  }];
  let mnemonic = "any_param";
  let assemblyFormat = "";
}
````
- **EN**: This TableGen block defines `Transform_AnyParamType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_AnyParamType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 68-83
````tablegen
def Transform_NormalizedOpType
    : TypeDef<Transform_Dialect, "NormalizedOp",
              [DeclareTypeInterfaceMethods<TransformHandleTypeInterface>]> {
  let description = [{
    Transform IR handle to operations that satisfy additional constraints
    required by normal form attributes parameterizing this type. When used for
    operands, serves as a guarantee of transform preconditions being satisfied.
    When used for results, serves as a guarantee of transform postcondition.
  }];
  let mnemonic = "normalized_op";
  let parameters = (ins ArrayRefParameter<
      "::mlir::transform::NormalFormAttrInterface",
      "Normal forms satisfied by the associated payload">:$normal_forms);
  let assemblyFormat = "`<` $normal_forms `>`";
  let genVerifyDecl = 1;
}
````
- **EN**: This TableGen block defines `Transform_NormalizedOpType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, verification hooks, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_NormalizedOpType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, 验证钩子, trait/接口组合。

### Lines 85-99
````tablegen
def Transform_ParamType : TypeDef<Transform_Dialect, "Param",
    [DeclareTypeInterfaceMethods<TransformParamTypeInterface>]> {
  let description = [{
    Transform IR value that can be associated with the list of parameters
    of the given type. Types are currently limited to integers, but may be
    extended in the future to other types values of which can be contained
    in attributes.
  }];
  let mnemonic = "param";
  let parameters = (ins
    TypeParameter<"::mlir::Type", "Underlying type of the parameter">:$type
  );
  let assemblyFormat = "`<` $type `>`";
  let genVerifyDecl = 1;
}
````
- **EN**: This TableGen block defines `Transform_ParamType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, verification hooks, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_ParamType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, 验证钩子, trait/接口组合。

### Lines 101-109
````tablegen
def Transform_TypeParamType : TypeDef<Transform_Dialect, "TypeParam",
    [DeclareTypeInterfaceMethods<TransformParamTypeInterface>]> {
  let description = [{
    Transform IR parameter value that can be associated with a list of type
    attributes.
  }];
  let mnemonic = "type";
  let assemblyFormat = "";
}
````
- **EN**: This TableGen block defines `Transform_TypeParamType` as a `def` record for `TransformTypes`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_TypeParamType` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 111-116
````tablegen
class Transform_ConcreteOpType<string opname>
  : Type<And<[Transform_OperationType.predicate,
              CPred<"::llvm::cast<::mlir::transform::OperationType>($_self)"
                    ".getOperationName() == \"" # opname # "\"">]>,
         "Transform IR handle to " # opname # " operations",
         "::mlir::transform::OperationType">;
````
- **EN**: This TableGen block defines `Transform_ConcreteOpType` as a `class` record for `TransformTypes`.
- **CN**: 该 TableGen 代码块将 `Transform_ConcreteOpType` 定义为 `class` 记录，用于描述 `TransformTypes` 相关的声明式信息。

### Lines 118-121
````tablegen
def TransformAnyHandle : Type<
    Or<[TransformHandleTypeInterface.predicate,
        TransformValueHandleTypeInterface.predicate]>,
    "transform operation or value handle">;
````
- **EN**: This TableGen block defines `TransformAnyHandle` as a `def` record for `TransformTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TransformAnyHandle` 定义为 `def` 记录，用于描述 `TransformTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 123-128
````tablegen
def TransformAnyParamTypeOrAnyHandle : Type<
    Or<[TransformHandleTypeInterface.predicate,
        TransformParamTypeInterface.predicate]>,
    "transform any param type or any handle type">;

#endif  // MLIR_DIALECT_TRANSFORM_IR_TRANSFORMTYPES
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Custom assembly syntax specification
  **CN**: 自定义汇编语法规格
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/AttrTypeBase.td
- mlir/Dialect/Transform/Interfaces/TransformInterfaces.td
- mlir/Dialect/Transform/IR/TransformDialect.td
- Transform_AffineMapParamType builds on TypeDef<Transform_Dialect, "AffineMapParam",
- Transform_AnyOpType builds on TypeDef<Transform_Dialect, "AnyOp",
- Transform_AnyValue builds on TypeDef<Transform_Dialect, "AnyValue",
- Transform_OperationType builds on TypeDef<Transform_Dialect, "Operation",
- Transform_AnyParamType builds on TypeDef<Transform_Dialect, "AnyParam",
- Transform_NormalizedOpType builds on TypeDef<Transform_Dialect, "NormalizedOp",
- Transform_ParamType builds on TypeDef<Transform_Dialect, "Param",
- Transform_TypeParamType builds on TypeDef<Transform_Dialect, "TypeParam",
- TransformAnyHandle builds on Type<
- TransformAnyParamTypeOrAnyHandle builds on Type<
