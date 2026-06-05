# TransformAttrs.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/IR/TransformAttrs.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TransformAttrs component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 TransformAttrs 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
````tablegen
//===- TransformAttrs.td - Transform dialect attributes ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_IR_TRANSFORMATTRS
#define MLIR_DIALECT_TRANSFORM_IR_TRANSFORMATTRS

include "mlir/IR/EnumAttr.td"
include "mlir/Dialect/Transform/IR/TransformDialect.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 15-20
````tablegen
class Transform_Attr<string name, string attrMnemonic,
                     list<Trait> traits = [],
                     string baseCppClass = "::mlir::Attribute">
    : AttrDef<Transform_Dialect, name, traits, baseCppClass> {
  let mnemonic = attrMnemonic;
}
````
- **EN**: This TableGen block defines `Transform_Attr` as a `class` record for `TransformAttrs`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Transform_Attr` 定义为 `class` 记录，用于描述 `TransformAttrs` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 22-22
````tablegen
def PropagateFailuresCase : I32EnumAttrCase<"Propagate", 1, "propagate">;
````
- **EN**: This TableGen block defines `PropagateFailuresCase` as a `def` record for `TransformAttrs`.
- **CN**: 该 TableGen 代码块将 `PropagateFailuresCase` 定义为 `def` 记录，用于描述 `TransformAttrs` 相关的声明式信息。

### Lines 23-23
````tablegen
def SuppressFailuresCase : I32EnumAttrCase<"Suppress", 2, "suppress">;
````
- **EN**: This TableGen block defines `SuppressFailuresCase` as a `def` record for `TransformAttrs`.
- **CN**: 该 TableGen 代码块将 `SuppressFailuresCase` 定义为 `def` 记录，用于描述 `TransformAttrs` 相关的声明式信息。

### Lines 25-29
````tablegen
def FailurePropagationMode : I32EnumAttr<
    "FailurePropagationMode", "Silenceable error propagation policy",
    [PropagateFailuresCase, SuppressFailuresCase]> {
  let cppNamespace = "::mlir::transform";
}
````
- **EN**: This TableGen block defines `FailurePropagationMode` as a `def` record for `TransformAttrs`.
- **CN**: 该 TableGen 代码块将 `FailurePropagationMode` 定义为 `def` 记录，用于描述 `TransformAttrs` 相关的声明式信息。

### Lines 31-42
````tablegen
def MatchCmpIPredicateAttr : I32EnumAttr<
    "MatchCmpIPredicate", "",
    [
      I32EnumAttrCase<"eq", 0>,
      I32EnumAttrCase<"ne", 1>,
      I32EnumAttrCase<"lt", 2>,
      I32EnumAttrCase<"le", 3>,
      I32EnumAttrCase<"gt", 4>,
      I32EnumAttrCase<"ge", 5>,
    ]> {
  let cppNamespace = "::mlir::transform";
}
````
- **EN**: This TableGen block defines `MatchCmpIPredicateAttr` as a `def` record for `TransformAttrs`.
- **CN**: 该 TableGen 代码块将 `MatchCmpIPredicateAttr` 定义为 `def` 记录，用于描述 `TransformAttrs` 相关的声明式信息。

### Lines 44-53
````tablegen
def ParamOperandAttr : Transform_Attr<"ParamOperand", "param_operand"> {
  let description = [{
    Used to refer to a specific param-operand (via its index) from within an
    attribute on a transform operation.
  }];
  let parameters = (ins
    "IntegerAttr":$index
  );
  let assemblyFormat = "`<` `index` `=` $index `>`";
}
````
- **EN**: This TableGen block defines `ParamOperandAttr` as a `def` record for `TransformAttrs`. It covers assembly syntax, semantic documentation.
- **CN**: 该 TableGen 代码块将 `ParamOperandAttr` 定义为 `def` 记录，用于描述 `TransformAttrs` 相关的声明式信息。 其中涉及 汇编语法, 语义文档。

### Lines 54-54
````tablegen
#endif  // MLIR_DIALECT_TRANSFORM_IR_TRANSFORMATTRS
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

- mlir/IR/EnumAttr.td
- mlir/Dialect/Transform/IR/TransformDialect.td
- PropagateFailuresCase builds on I32EnumAttrCase<"Propagate", 1, "propagate">;
- SuppressFailuresCase builds on I32EnumAttrCase<"Suppress", 2, "suppress">;
- FailurePropagationMode builds on I32EnumAttr<
- MatchCmpIPredicateAttr builds on I32EnumAttr<
- ParamOperandAttr builds on Transform_Attr<"ParamOperand", "param_operand">
