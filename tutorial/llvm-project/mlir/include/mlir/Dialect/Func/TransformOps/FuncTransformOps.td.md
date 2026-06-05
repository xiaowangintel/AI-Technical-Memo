# FuncTransformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Func/TransformOps/FuncTransformOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the Func dialect, focused on transform dialect operation declarations and orchestration hooks and `FuncTransformOps`.
  - **CN**: 为 Func 方言定义聚焦 `FuncTransformOps` 的声明式 TableGen 规格，覆盖Transform Dialect 操作声明与编排钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- FuncTransformOps.td - CF transformation ops -*- tablegen -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-18
```tablegen

#ifndef FUNC_TRANSFORM_OPS
#define FUNC_TRANSFORM_OPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Dialect/Transform/IR/TransformTypes.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/RegionKindInterface.td"
include "mlir/IR/OpBase.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 19-27
```tablegen
def ApplyFuncToLLVMConversionPatternsOp : Op<Transform_Dialect,
    "apply_conversion_patterns.func.func_to_llvm",
    [DeclareOpInterfaceMethods<ConversionPatternDescriptorOpInterface,
                               ["verifyTypeConverter"]>]> {
  let description = [{
    Collects patterns that convert Func dialect ops to LLVM dialect ops.
    These patterns require an "LLVMTypeConverter".
  }];

```
- **EN**: Introduces declarations for `ApplyFuncToLLVMConversionPatternsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ApplyFuncToLLVMConversionPatternsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-39
```tablegen
  let assemblyFormat = "attr-dict";
}

def CastAndCallOp : Op<Transform_Dialect,
    "func.cast_and_call",
    [DeclareOpInterfaceMethods<TransformOpInterface>,
     DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
     AttrSizedOperandSegments,
     ReportTrackingListenerFailuresOpTrait]
        # GraphRegionNoTerminator.traits> {
  let summary = "Casts values to the signature of a function and replaces them "
                "with a call";
```
- **EN**: Introduces declarations for `CastAndCallOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `CastAndCallOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 40-48
```tablegen
  let description = [{
    This transform takes value handles to a set of `inputs` and `outputs` and
    attempts to cast them to the function signature of the attached function
    op, then builds a call to the function and replaces the users of the
    outputs. It is the responsibility of the user to ensure that the slice of
    the program replaced by this operation makes sense, i.e. there is no
    verification that the inputs to this operation have any relation to the
    outputs outside of basic dominance requirements needed for the call.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 49-55
```tablegen
    The casting materialization functions are specified in the graph region of
    this op. They must implement the `TypeConverterBuilderOpInterface`. The
    order of ops within the region is irrelevant.

    The target function can be specified by a symbol name or by a handle to the
    operation.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 56-67
```tablegen
    This transform only reads the operand handles and only replaces the users of
    the outputs with the results of the call. No handles are consumed and no
    operations are removed. Users are expected to run cleanup separately if
    desired.

    Warning: The replacement of the uses of the outputs could invalidate certain
    restricted value handle types (e.g. `transform.block_arg` if it existed, by
    replacing the use with something not coming from a block argument). The
    value will still exist in such cases but wouldn't verify against the type.
    See the discussion here for more information:
    https://github.com/llvm/llvm-project/pull/78398#discussion_r1455070087

```
- **EN**: Implements logic around `types`; this block checks structural or semantic invariants; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `types` 实现具体逻辑；该代码块检查结构或语义不变式，并处理方言 IR 实体，如操作、类型或属性。

### Lines 68-74
```tablegen
    This transform will emit a silenceable failure if:
     - The set of outputs isn't unique
     - The handle for the insertion point does not include exactly one operation
     - The insertion point op does not dominate any of the output users
     - The insertion point op is not dominated by any of the inputs
     - The function signature does not match the number of inputs/outputs

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 75-80
```tablegen
    This transform will emit a definite failure if it fails to resolve the
    target function, or if it fails to materialize the conversion casts of
    either the inputs to the function argument types, or the call results to
    the output types.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 81-90
```tablegen
  let arguments = (ins
    TransformHandleTypeInterface:$insertion_point,
    UnitAttr:$insert_after,
    Optional<TransformValueHandleTypeInterface>:$inputs,
    Optional<TransformValueHandleTypeInterface>:$outputs,
    OptionalAttr<SymbolRefAttr>:$function_name,
    Optional<TransformHandleTypeInterface>:$function);
  let results = (outs TransformHandleTypeInterface:$result);
  let regions = (region MaxSizedRegion<1>:$conversions);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 91-100
```tablegen
  let assemblyFormat = [{
    ($function_name^)? ($function^)?
    ( `(` $inputs^ `)` )?
    ( `->` $outputs^ )?
    (`after` $insert_after^):(`before`)? $insertion_point
    ($conversions^)? attr-dict `:` functional-type(operands, results)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 101-111
```tablegen
def ReplaceFuncSignatureOp
    : Op<Transform_Dialect, "func.replace_func_signature",
         [DeclareOpInterfaceMethods<TransformOpInterface>,
          DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
  let description = [{
      This transform takes a module and a function name, and replaces the
      signature of the function by reordering the arguments and results
      according to the interchange arrays. The function is expected to be
      defined in the module, and the interchange arrays must match the number
      of arguments and results of the function.

```
- **EN**: Introduces declarations for `ReplaceFuncSignatureOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ReplaceFuncSignatureOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 112-122
```tablegen
      The `adjust_func_calls` attribute indicates whether the function calls
      should be adjusted to match the new signature. If set to `true`, the
      function calls will be adjusted to match the new signature, otherwise
      they will not be adjusted.

      This transform will emit a silenceable failure if:
       - The function with the given name does not exist in the module.
       - The interchange arrays do not match the number of arguments/results.
       - The interchange arrays contain out of bound indices.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 123-128
```tablegen
  let arguments = (ins TransformHandleTypeInterface:$module,
      SymbolRefAttr:$function_name, DenseI32ArrayAttr:$args_interchange,
      DenseI32ArrayAttr:$results_interchange, UnitAttr:$adjust_func_calls);
  let results = (outs TransformHandleTypeInterface:$transformed_module,
                      TransformHandleTypeInterface:$transformed_function);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 129-136
```tablegen
  let assemblyFormat = [{
    $function_name
    `args_interchange` `=` $args_interchange
    `results_interchange` `=` $results_interchange
    `at` $module attr-dict `:` functional-type(operands, results)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 137-145
```tablegen
def DeduplicateFuncArgsOp
    : Op<Transform_Dialect, "func.deduplicate_func_args",
         [DeclareOpInterfaceMethods<TransformOpInterface>,
          DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
  let description = [{
      This transform takes a module and a function name, and deduplicates
      the arguments of the function. The function is expected to be defined in
      the module.

```
- **EN**: Introduces declarations for `DeduplicateFuncArgsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DeduplicateFuncArgsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 146-151
```tablegen
      This transform will emit a silenceable failure if:
       - The function with the given name does not exist in the module.
       - The function does not have duplicate arguments.
       - The function does not have a single call.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 152-162
```tablegen
  let arguments = (ins TransformHandleTypeInterface:$module,
      SymbolRefAttr:$function_name);
  let results = (outs TransformHandleTypeInterface:$transformed_module,
                      TransformHandleTypeInterface:$transformed_function);

  let assemblyFormat = [{
    $function_name
    `at` $module attr-dict `:` functional-type(operands, results)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 163-163
```tablegen
#endif // FUNC_TRANSFORM_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/Transform/IR/TransformDialect.td`, `mlir/Dialect/Transform/Interfaces/TransformInterfaces.td`, `mlir/Dialect/Transform/IR/TransformTypes.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/IR/RegionKindInterface.td`, `mlir/IR/OpBase.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
