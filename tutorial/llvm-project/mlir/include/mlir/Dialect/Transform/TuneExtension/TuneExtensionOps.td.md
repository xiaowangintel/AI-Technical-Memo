# TuneExtensionOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/TuneExtension/TuneExtensionOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TuneExtensionOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: KnobOp.
- **用途（CN）**: 为 MLIR 的 TuneExtensionOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
````tablegen
//===- TuneExtensionOps.td - Transform dialect operations --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_TUNEEXTENSION_TUNEEXTENSIONOPS
#define MLIR_DIALECT_TRANSFORM_TUNEEXTENSION_TUNEEXTENSIONOPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Interfaces/ControlFlowInterfaces.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/BuiltinAttributes.td"
include "mlir/IR/CommonAttrConstraints.td"

//===----------------------------------------------------------------------===//
// KnobOp
//===----------------------------------------------------------------------===//
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 23-58
````tablegen
def KnobOp : Op<Transform_Dialect, "tune.knob", [
  DeclareOpInterfaceMethods<TransformOpInterface>,
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
]> {
  let summary = "Represents a tunable parameter with a set of options";

  let description = [{
    Provides a representation for "tunables" within schedules.

    Each op represents a single tunable, which has a `name` and a set
    of valid `options` described by an attribute. Without a specified
    `selected` option, this op represents a non-deterministic choice
    that has yet to be resolved -- as such, the interpreter runtime
    semantics is to raise a failure.

    The non-deterministic choice is resolved through providing a
    `selected` attribute. When provided, the interpreter runtime
    semantics are to return the `selected` attribute as a param through
    the op's result.

    -----

    In case the `options` attribute is an `ArrayAttr`, the verifier
    checks that the provided `selected` attribute occurs in `options`.
  }];
  let cppNamespace = [{ mlir::transform::tune }];
  let hasVerifier = 1;

  let arguments = (ins Builtin_StringAttr:$name,
                       AnyAttr:$options,
                       OptionalAttr<AnyAttr>:$selected);
  let results = (outs TransformParamTypeInterface:$result);

  let assemblyFormat =
      "`<` $name `>` (`=` $selected^ `from`)? `options` `=` $options attr-dict `->` type(results)";
}
````
- **EN**: This TableGen block defines `KnobOp` as a `def` record for `TuneExtensionOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `KnobOp` 定义为 `def` 记录，用于描述 `TuneExtensionOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 59-61
````tablegen
//===----------------------------------------------------------------------===//
// AlternativesOp
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 64-108
````tablegen
def AlternativesOp : Op<Transform_Dialect, "tune.alternatives", [
  DeclareOpInterfaceMethods<RegionBranchOpInterface,
        ["getEntrySuccessorOperands",
         "getRegionInvocationBounds",
         "getSuccessorInputs"]>,
  DeclareOpInterfaceMethods<TransformOpInterface>,
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
  SingleBlockImplicitTerminator<"::mlir::transform::YieldOp">,
  NoRegionArguments
]> {
  let summary = "Represents a choice among its regions, i.e. sub-schedules";

  let description = [{
    This op represents a choice over which of its regions is to be used.

    When `selected_region` is provided, the semantics are that this op is to be
    substituted for by the selected region, meaning the region's results become
    the results of this op. Without a provided `selected_region`, the semantics
    are that this non-deterministic choice is yet to be resolved -- which in
    terms of the op's interpreted semantics is a failure.

    The `selected_region` argument is either an `IntegerAttr` or a param holding
    an `IntegerAttr`, which should provide a valid zero-based index with respect
    to the number of alternatives, i.e. regions.
  }];
  let cppNamespace = [{ mlir::transform::tune }];

  let arguments = (ins Builtin_StringAttr:$name,
                       OptionalAttr<APIntAttr>:$selected_region_attr,
                       Optional<TransformParamTypeInterface>:$selected_region_param);
  let results = (outs Variadic<Transform_AnyHandleOrParamType>:$results);
  let regions = (region VariadicRegion<SizedRegion<1>>:$alternatives);

  let assemblyFormat = [{
    `<` $name `>`
    (`selected_region` `=` custom<AlternativesOpSelectedRegion>(
        $selected_region_attr, $selected_region_param)^)?
    attr-dict-with-keyword
    (`:` type($selected_region_param)^)?
    (`->` type($results)^)?
    regions
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `AlternativesOp` as a `def` record for `TuneExtensionOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `AlternativesOp` 定义为 `def` 记录，用于描述 `TuneExtensionOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 109-109
````tablegen
#endif // MLIR_DIALECT_TRANSFORM_TUNEEXTENSION_TUNEEXTENSIONOPS
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

- mlir/Dialect/Transform/IR/TransformDialect.td
- mlir/Dialect/Transform/Interfaces/TransformInterfaces.td
- mlir/Interfaces/ControlFlowInterfaces.td
- mlir/Interfaces/SideEffectInterfaces.td
- mlir/IR/BuiltinAttributes.td
- mlir/IR/CommonAttrConstraints.td
- KnobOp builds on Op<Transform_Dialect, "tune.knob", [
- AlternativesOp builds on Op<Transform_Dialect, "tune.alternatives", [
