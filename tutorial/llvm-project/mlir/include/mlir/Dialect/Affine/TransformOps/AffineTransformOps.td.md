# AffineTransformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/TransformOps/AffineTransformOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the Affine dialect, focused on transform dialect operation declarations and orchestration hooks and `AffineTransformOps`.
  - **CN**: 为 Affine 方言定义聚焦 `AffineTransformOps` 的声明式 TableGen 规格，覆盖Transform Dialect 操作声明与编排钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- AffineTransformOps.td - Affine transformation ops ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-17
```tablegen

#ifndef AFFINE_TRANSFORM_OPS
#define AFFINE_TRANSFORM_OPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Dialect/Transform/IR/TransformTypes.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/OpBase.td"

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 18-27
```tablegen
def Transform_AffineForOp : Transform_ConcreteOpType<"affine.for">;

def SimplifyBoundedAffineOpsOp
    : Op<Transform_Dialect, "affine.simplify_bounded_affine_ops",
         [DeclareOpInterfaceMethods<TransformOpInterface>,
          DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
  let description = [{
    Simplify the targeted affine.min / affine.max ops given the supplied
    lower and upper bounds for values that may be used as target op operands.

```
- **EN**: Introduces declarations for `Transform_AffineForOp`, `SimplifyBoundedAffineOpsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Transform_AffineForOp`, `SimplifyBoundedAffineOpsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-33
```tablegen
    Example:
    ```
    %0 = transform.structured.match ops{["affine.min", "affine.max"]} in %arg1
    %1 = transform.structured.match ops{["gpu.lane_id"]} in %arg1
    transform.affine.simplify_bounded_affine_ops %0 with [%1] within [0] and [32]

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 34-41
```tablegen
    // Multiple bounds can be specified.
    transform.affine.simplify_bounded_affine_ops %0 with [%1, %2] within [0, 5] and [32, 50]
    ```

    Bounded op handles (`%1` and `%2) must be mapped to ops that have a single
    result of index type. The sets of target ops and bounded ops must not
    overlap.

```
- **EN**: Implements logic around `handles`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `handles` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 42-47
```tablegen
    #### Return modes

    Target ops must be affine.min or affine.max ops. This transform consumes the
    target handle and does not produce any handle. It reads the bounded op
    handles.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 48-57
```tablegen
    TODO: Support affine.apply targets.
    TODO: Allow mixed PDL_Operation/int64_t for lower_bounds and upper_bounds.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target,
                       Variadic<TransformHandleTypeInterface>:$bounded_values,
                       DenseI64ArrayAttr:$lower_bounds,
                       DenseI64ArrayAttr:$upper_bounds);
  let results = (outs);
  let hasVerifier = 1;
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 58-65
```tablegen

  let assemblyFormat = [{
      $target `with` `[` ($bounded_values^ `:` type($bounded_values))? `]`
          `within` $lower_bounds `and` $upper_bounds attr-dict
          `:` type($target)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 66-74
```tablegen
def SimplifyMinMaxAffineOpsOp :
  Op<Transform_Dialect, "affine.simplify_min_max_affine_ops", [
    DeclareOpInterfaceMethods<TransformOpInterface>,
    DeclareOpInterfaceMethods<MemoryEffectsOpInterface>
  ]> {
  let description = [{
    Simplify the targeted `affine.min` / `affine.max` ops using the
    `mlir::affine::simplifyAffineMinMaxOps` transform.

```
- **EN**: Introduces declarations for `SimplifyMinMaxAffineOpsOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SimplifyMinMaxAffineOpsOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 75-80
```tablegen
    Example:
    ```
    %0 = transform.structured.match ops{["affine.max"]} in %arg1
    transform.affine.simplify_min_max_affine_ops %0 : !transform.any_op
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 81-90
```tablegen
    #### Return modes

    This transform consumes the target handle and does not produce any results.
    This transforms definitely fails if any of the targeted operations is not an
    `affine.min` or `affine.max` operation, or if the canonicalization patterns
    failed to converge.
    This transform silently fails if none of the operations were simplified.
    Otherwise, it succeeds.
  }];
  let arguments = (ins TransformHandleTypeInterface:$target);
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 91-96
```tablegen
  let results = (outs);
  let assemblyFormat = [{
      $target attr-dict `:` type($target)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 97-106
```tablegen
def SuperVectorizeOp
    : Op<Transform_Dialect, "affine.super_vectorize",
         [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
         DeclareOpInterfaceMethods<TransformOpInterface>]> {
  let description = [{
    Vectorize to a target independent n-D vector abstraction.
    This operation is an exposition to the transform dialect of the affine-super-vectorize pass.
    To make its usage easier, it ignores inputs which are children of an affine.for op (itself excluded),
    this way the matcher can be simpler.

```
- **EN**: Introduces declarations for `SuperVectorizeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SuperVectorizeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 107-115
```tablegen
    This operation may generate operations from the vector dialect.

    Example:
    ```
    %0 = transform.structured.match ops{["affine.for"]} in %arg1 : (!transform.any_op) -> !transform.any_op
    transform.affine.super_vectorize %0 [8, 16] fastest_varying_pattern=[1,0] vectorize_reductions=true : !transform.any_op
    ```
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 116-121
```tablegen
  let arguments = (ins TransformHandleTypeInterface:$target,
                       DenseI64ArrayAttr:$vector_sizes,
                       OptionalAttr<DenseI64ArrayAttr>:$fastest_varying_pattern,
                       DefaultValuedAttr<BoolAttr, "false">:$vectorize_reductions);
  let results = (outs);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 122-130
```tablegen
  let assemblyFormat = [{
      $target $vector_sizes
          (`fastest_varying_pattern` `=` $fastest_varying_pattern^)? 
          (`vectorize_reductions` `=` $vectorize_reductions^)? 
          attr-dict `:` type($target)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 131-131
```tablegen
#endif // Affine_TRANSFORM_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Dialect/Transform/IR/TransformDialect.td`, `mlir/Dialect/Transform/Interfaces/TransformInterfaces.td`, `mlir/Dialect/Transform/IR/TransformTypes.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/IR/OpBase.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (3), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
