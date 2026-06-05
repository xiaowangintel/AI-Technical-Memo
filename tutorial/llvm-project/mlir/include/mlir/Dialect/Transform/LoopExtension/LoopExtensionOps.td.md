# LoopExtensionOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/LoopExtension/LoopExtensionOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR LoopExtensionOps component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 LoopExtensionOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
````tablegen
//===- LoopExtensionOps.td - Transform dialect operations --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_LOOPEXTENSION_LOOPEXTENSIONOPS
#define MLIR_DIALECT_TRANSFORM_LOOPEXTENSION_LOOPEXTENSIONOPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 16-74
````tablegen
def HoistLoopInvariantSubsetsOp
    : TransformDialectOp<"loop.hoist_loop_invariant_subsets",
        [TransformOpInterface, TransformEachOpTrait,
         DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
         ReportTrackingListenerFailuresOpTrait]> {
  let summary = "Hoist loop invariant subset ops";
  let description = [{
    This transform hoists loop-invariant subset ops out of the targeted
    loop-like op. It looks for matching subset extraction/insertion op pairs and
    hoists them. The loop body operates on a newly introduced region iter_arg.

    Subset ops are hoisted only from the targeted op. If subset ops should be
    hoisted from an entire loop nest, this transformation must be applied to
    each loop-like op of the loop nest, starting with the innermost loop and
    ending with the outermost loop.

    Example:
    ```
    %r = scf.for ... iter_args(%t = %a) -> (tensor<?xf32>) {
      %0 = tensor.extract_slice %t[0][5][1] : tensor<?xf32> to tensor<5xf32>
      %1 = "test.foo"(%0) : (tensor<5xf32>) -> (tensor<5xf32>)
      %2 = tensor.insert_slice %1 into %t[0][5][1]
          : tensor<5xf32> into tensor<?xf32>
      scf.yield %2 : tensor<?xf32>
    }
    ```
    Is transformed to:
    ```
    %0 = tensor.extract_slice %a[0][5][1] : tensor<?xf32> to tensor<5xf32>
    %new_loop:2 = scf.for ... iter_args(%t = %a, %h = %0) -> (tensor<?xf32>) {
      %1 = "test.foo"(%h) : (tensor<5xf32>) -> (tensor<5xf32>)
      scf.yield %t, %2 : tensor<?xf32>, tensor<5xf32>
    }
    %r = tensor.insert_slice %new_loop#1 into %new_loop#0
        : tensor<5xf32> into tensor<?xf32>
    ```

    Subset ops are hoisted only if there are no conflicting subset ops. E.g.,
    if there were a second overlapping extraction in the above example, no ops
    could be hoisted safely.

    This transform reads the target handle and modifies the payload. This
    transform does not invalidate any handles, but loop-like ops are replaced
    with new loop-like ops when a subset op is hoisted. The transform rewriter
    updates all handles accordingly.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target);
  let results = (outs);
  let assemblyFormat = "$target attr-dict `:` type($target)";

  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure applyToOne(
      ::mlir::transform::TransformRewriter &rewriter,
      ::mlir::LoopLikeOpInterface loopLikeOp,
      ::mlir::transform::ApplyToEachResultList &results,
      ::mlir::transform::TransformState &state);
  }];
}
````
- **EN**: This TableGen block defines `HoistLoopInvariantSubsetsOp` as a `def` record for `LoopExtensionOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `HoistLoopInvariantSubsetsOp` 定义为 `def` 记录，用于描述 `LoopExtensionOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 75-75
````tablegen
#endif // MLIR_DIALECT_TRANSFORM_LOOPEXTENSION_LOOPEXTENSIONOPS
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
- mlir/Interfaces/SideEffectInterfaces.td
- HoistLoopInvariantSubsetsOp builds on TransformDialectOp<"loop.hoist_loop_invariant_subsets",
