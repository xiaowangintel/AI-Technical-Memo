# TensorTransformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tensor/TransformOps/TensorTransformOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TensorTransformOps component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 TensorTransformOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
````tablegen
//===- TensorTransformOps.td - Tensor transformation ops ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef TENSOR_TRANSFORM_OPS
#define TENSOR_TRANSFORM_OPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Dialect/Transform/IR/TransformTypes.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 18-27
````tablegen
def ApplyDecomposeTensorConcatPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.decompose_concat",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that tensor.concat ops should be decomposed into a chain of
    tensor.insert_slice operations inserting into a materialized destination.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyDecomposeTensorConcatPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyDecomposeTensorConcatPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 30-40
````tablegen
def ApplyDropRedundantInsertSliceRankExpansionPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.drop_redundant_insert_slice_rank_expansion",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that redundant tensor.insert_slice rank reductions should be
    dropped. E.g., cases where a tensor.extract_slice rank reduction immediately
    follows an inverse tensor.insert_slice rank expansion.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyDropRedundantInsertSliceRankExpansionPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyDropRedundantInsertSliceRankExpansionPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 42-55
````tablegen
def ApplyFoldTensorEmptyPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.fold_tensor_empty",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that tensor.extract_slice and reassociative reshapes should be
    folded into tensor.empty.

    If `fold_single_use_only` is set to "true", only tensor.empty that have a
    single use are folded.
  }];

  let arguments = (ins DefaultValuedAttr<BoolAttr, "false">:$fold_single_use_only);
  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyFoldTensorEmptyPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyFoldTensorEmptyPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 57-66
````tablegen
def ApplyFoldTensorSubsetOpsPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.fold_tensor_subset_ops",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that tensor.empty should be folded with tensor.extract_slice,
    tensor.expand_shape and tensor.collapse_shape.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyFoldTensorSubsetOpsPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyFoldTensorSubsetOpsPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 68-78
````tablegen
def ApplyFoldTensorSubsetOpsIntoVectorTransfersPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.fold_tensor_subset_ops_into_vector_transfers",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that tensor.extract_slice -> vector.transfer_read and
    vector.transfer_write -> tensor.insert_slice op chains should be folded into
    vector tranfer read and write ops
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyFoldTensorSubsetOpsIntoVectorTransfersPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyFoldTensorSubsetOpsIntoVectorTransfersPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 80-90
````tablegen
def ApplyMergeConsecutiveInsertExtractSlicePatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.merge_consecutive_insert_extract_slice",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that consecutive tensor.extract_slice/tensor.insert_slice ops
    should be merged into a single op. These patterns are not canonicalizations
    because the bufferization is sensitive to IR structure.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyMergeConsecutiveInsertExtractSlicePatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyMergeConsecutiveInsertExtractSlicePatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 92-102
````tablegen
def ApplyReassociativeReshapeFoldingPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.reassociative_reshape_folding",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that reassociative reshapes (tensor.collapse_shape /
    tensor.expand_shape) should be folded with inverse rank expansions / rank
    reductions (via tensor.insert_slice / tensor.extract_slice).
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyReassociativeReshapeFoldingPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyReassociativeReshapeFoldingPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 104-113
````tablegen
def ApplyBubbleUpExtractSlicePatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.bubble_up_extract_slice",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that producers of tensor.extract_slice should swap and operate on 
    the result of the slice.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyBubbleUpExtractSlicePatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyBubbleUpExtractSlicePatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 115-126
````tablegen
def ApplyRewriteTensorOpsAsConstantPatternsOp : Op<Transform_Dialect,
    "apply_patterns.tensor.rewrite_as_constant",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let arguments = (ins UnitAttr:$aggressive);
  let description = [{
    Indicates that tensor ops (such as tensor.generate) should be replaced with
    constants (arith.constant) when possible.
  }];

  let assemblyFormat =
      "(`aggressive` $aggressive^)? attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyRewriteTensorOpsAsConstantPatternsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyRewriteTensorOpsAsConstantPatternsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 128-128
````tablegen
def Transform_TensorPadOp : Transform_ConcreteOpType<"tensor.pad">;
````
- **EN**: This TableGen block defines `Transform_TensorPadOp` as a `def` record for `TensorTransformOps`.
- **CN**: 该 TableGen 代码块将 `Transform_TensorPadOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。

### Lines 130-173
````tablegen
def MakeLoopIndependentOp
    : Op<Transform_Dialect, "tensor.make_loop_independent",
         [FunctionalStyleTransformOpTrait, MemoryEffectsOpInterface,
          TransformOpInterface, TransformEachOpTrait]> {
  let description = [{
    Rewrite the targeted ops such that their index-typed operands no longer
    depend on any loop induction variable of the `num_loop` enclosing `scf.for`
    loops. I.e., compute an upper bound that is independent of any such loop IV
    for every tensor dimension. The transformed op could then be hoisted from
    the `num_loop` enclosing loops. To preserve the original semantics, place a
    `tensor.extract_slice` inside the loop.

    Currently supported operations are:
    - tensor.empty: Replaced with a new tensor.empty with upper bound sizes,
      followed by a tensor.extract_slice.
    - tensor.pad: Replaced by an upper bound padding, followed by a
      tensor.extract_slice.

    #### Return modes

    This operation fails if at least one induction variable could not be
    eliminated. In case the targeted op is already independent of induction
    variables, this transform succeeds and returns the unmodified target op.

    Otherwise, the returned handle points to a subset of the produced ops:
    - tensor.empty: The returned handle points to the tensor.extract_slice op.
    - tensor.pad: The returned handle points to the tensor.extract_slice op.

    This transform op consumes the target handle and produces a result handle.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target, I64Attr:$num_loops);
  let results = (outs TransformHandleTypeInterface:$transformed);
  let assemblyFormat =
      "$target attr-dict `:` functional-type($target, $transformed)";

  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure applyToOne(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::Operation *target,
        ::mlir::transform::ApplyToEachResultList &results,
        ::mlir::transform::TransformState &state);
  }];
}
````
- **EN**: This TableGen block defines `MakeLoopIndependentOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `MakeLoopIndependentOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 175-191
````tablegen
def TypeConversionCastShapeDynamicDimsOp : Op<Transform_Dialect,
    "type_conversion.tensor.cast_shape_dynamic_dims",
    [DeclareOpInterfaceMethods<TypeConverterBuilderOpInterface,
                               ["populateTypeMaterializations"]>]> {
  let description = [{
    Populates a type converter with conversion materialization functions that
    cast a tensor value between two cast-compatible tensors. See `tensor.cast`
    for more information on cast compatibility between tensors.

    If `ignore_dynamic_info` is not set, this will set an additional constraint
    that source materializations do not cast dynamic dimensions to static ones.
  }];
  let arguments = (ins UnitAttr:$ignore_dynamic_info);

  let assemblyFormat =
      "(`ignore_dynamic_info` $ignore_dynamic_info^)? attr-dict";
}
````
- **EN**: This TableGen block defines `TypeConversionCastShapeDynamicDimsOp` as a `def` record for `TensorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TypeConversionCastShapeDynamicDimsOp` 定义为 `def` 记录，用于描述 `TensorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 192-192
````tablegen
#endif // TENSOR_TRANSFORM_OPS
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
- mlir/Dialect/Transform/IR/TransformTypes.td
- mlir/Interfaces/SideEffectInterfaces.td
- mlir/IR/OpBase.td
- ApplyDecomposeTensorConcatPatternsOp builds on Op<Transform_Dialect,
- ApplyDropRedundantInsertSliceRankExpansionPatternsOp builds on Op<Transform_Dialect,
- ApplyFoldTensorEmptyPatternsOp builds on Op<Transform_Dialect,
- ApplyFoldTensorSubsetOpsPatternsOp builds on Op<Transform_Dialect,
- ApplyFoldTensorSubsetOpsIntoVectorTransfersPatternsOp builds on Op<Transform_Dialect,
- ApplyMergeConsecutiveInsertExtractSlicePatternsOp builds on Op<Transform_Dialect,
- ApplyReassociativeReshapeFoldingPatternsOp builds on Op<Transform_Dialect,
- ApplyBubbleUpExtractSlicePatternsOp builds on Op<Transform_Dialect,
- ApplyRewriteTensorOpsAsConstantPatternsOp builds on Op<Transform_Dialect,
- Transform_TensorPadOp builds on Transform_ConcreteOpType<"tensor.pad">;
- MakeLoopIndependentOp builds on Op<Transform_Dialect, "tensor.make_loop_independent",
- TypeConversionCastShapeDynamicDimsOp builds on Op<Transform_Dialect,
