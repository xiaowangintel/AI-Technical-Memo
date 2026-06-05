# XeGPUTransformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/XeGPU/TransformOps/XeGPUTransformOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR XeGPUTransformOps component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 XeGPUTransformOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````tablegen
//===- XeGPUTransformOps.td - XeGPU transformation ops -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef XEGPU_TRANSFORM_OPS
#define XEGPU_TRANSFORM_OPS

include "mlir/Dialect/Transform/IR/TransformAttrs.td"
include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Dialect/Transform/IR/TransformTypes.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 19-35
````tablegen
def GetLoadOp : Op<Transform_Dialect, "xegpu.get_load_op", [
  DeclareOpInterfaceMethods<TransformOpInterface>,
  NavigationTransformOpTrait, MemoryEffectsOpInterface
]> {

  let summary = "Get a handle to the load_nd op in producer chain of a value.";
  let description = [{
    Traces the producers of the given value until an `xegpu.load_nd` or
    `xegpu.load` op is found. Returns a handle to it. Currently traces
    producers by following only the first operand of producer ops.
  }];

  let arguments = (ins TransformValueHandleTypeInterface:$target);

  let results = (outs TransformHandleTypeInterface:$loadNdHandle);
  let assemblyFormat = "$target attr-dict `:` functional-type(operands, results)";
}
````
- **EN**: This TableGen block defines `GetLoadOp` as a `def` record for `XeGPUTransformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `GetLoadOp` 定义为 `def` 记录，用于描述 `XeGPUTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 37-107
````tablegen
def SetAnchorLayoutOp : Op<Transform_Dialect, "xegpu.set_anchor_layout", [
  AttrSizedOperandSegments,
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
  TransformOpInterface
]> {

  let summary = "Set anchor layout of an op.";
  let description = [{
    Sets the `xegpu.layout` anchor layout for XeGPU ops that support it. The
    target operand value can be set by the `index` argument (currently only
    applicable to a DPAS op). The layout is defined by the `sg_layout`,
    `sg_data` and optional `inst_data` and `order` attributes. If `slice_dims`
    is provided, the `xegpu.layout` attribute is wrapped in an
    `xegpu.slice<..., dims=slice_dims>` attribute. Emits a silenceable failure
    if the target op does not support anchor layouts.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target,
                   DefaultValuedOptionalAttr<I64Attr, "0">:$index,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$sg_layout,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$sg_data,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$inst_data,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_sg_layout,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_sg_data,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_inst_data,
                   DefaultValuedOptionalAttr<DenseI32ArrayAttr, "{}">:$order,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$slice_dims
                   );

  let results = (outs);
  let builders = [
    OpBuilder<(ins "Value":$target,
                   "int64_t":$index,
                   "ArrayRef<OpFoldResult>":$mixedSgLayout,
                   "ArrayRef<OpFoldResult>":$mixedSgData,
                   "ArrayRef<OpFoldResult>":$mixedInstData,
                   "ArrayRef<int32_t>":$order,
                   "ArrayRef<int64_t>":$sliceDims
                   )>,
  ];

  let assemblyFormat = [{
    $target (`index` `=` $index^)?
    `sg_layout` `=` custom<DynamicIndexList>($sg_layout, $static_sg_layout)
    `sg_data` `=` custom<DynamicIndexList>($sg_data, $static_sg_data)
    (`inst_data` `=` custom<DynamicIndexList>($inst_data, $static_inst_data)^)?
    (`order` `=` $order^)?
    (`slice_dims` `=` $slice_dims^)?
    attr-dict `:` qualified(type(operands))
  }];

  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure apply(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::transform::TransformResults &transformResults,
        ::mlir::transform::TransformState &state);

    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedSgLayout() {
      Builder b(getContext());
      return getMixedValues(getStaticSgLayout(), getSgLayout(), b);
    }
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedSgData() {
      Builder b(getContext());
      return getMixedValues(getStaticSgData(), getSgData(), b);
    }
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedInstData() {
      Builder b(getContext());
      return getMixedValues(getStaticInstData(), getInstData(), b);
    }
  }];
}
````
- **EN**: This TableGen block defines `SetAnchorLayoutOp` as a `def` record for `XeGPUTransformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SetAnchorLayoutOp` 定义为 `def` 记录，用于描述 `XeGPUTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 109-146
````tablegen
def SetGPULaunchThreadsOp
    : Op<Transform_Dialect, "xegpu.set_gpu_launch_threads", [
      DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
      TransformOpInterface
    ]> {

  let summary = "Set number of threads for a given gpu.launch operation";
  let description = [{
    Overrides the x,y,z threads operands of a given `gpu.launch` operation in-place.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$threads,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_threads
                   );
  let results = (outs);
  let builders = [
    OpBuilder<(ins "Value":$target, "ArrayRef<OpFoldResult>":$mixedThreads)>,
  ];

  let assemblyFormat = [{
    $target
    `threads` `=` custom<DynamicIndexList>($threads, $static_threads)
    attr-dict `:` qualified(type(operands))
  }];

  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure apply(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::transform::TransformResults &transformResults,
        ::mlir::transform::TransformState &state);

    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedThreads() {
      Builder b(getContext());
      return getMixedValues(getStaticThreads(), getThreads(), b);
    }
  }];
}
````
- **EN**: This TableGen block defines `SetGPULaunchThreadsOp` as a `def` record for `XeGPUTransformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SetGPULaunchThreadsOp` 定义为 `def` 记录，用于描述 `XeGPUTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 148-188
````tablegen
def InsertPrefetchOp : Op<Transform_Dialect, "xegpu.insert_prefetch", [
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
  TransformOpInterface
]> {

  let summary = "Adds xegpu prefetch ops to a load op.";
  let description = [{
    Inserts `xegpu.prefetch_nd` operations for the given `xegpu.load_nd` op.
    The load op must reside within the `scf.for` loop. Number of prefetch steps
    is set by the `nb_prefetch` argument (default value is 1). Returns a handle
    to the created `xegpu.create_nd_desc` op.
  }];

  let arguments = (ins TransformHandleTypeInterface:$target,
                   Optional<TransformAnyParamTypeOrAnyHandle>:$dynamic_nb_prefetch,
                   DefaultValuedOptionalAttr<I64Attr, "1">:$static_nb_prefetch
                   );

  let results = (outs TransformHandleTypeInterface:$desc_op);

  let assemblyFormat = [{
    $target
    `nb_prefetch` `=` ($dynamic_nb_prefetch^):($static_nb_prefetch)?
    attr-dict `:` functional-type(operands, results)
  }];

  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure apply(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::transform::TransformResults &transformResults,
        ::mlir::transform::TransformState &state);

    OpFoldResult getNbPrefetch() {
      auto cxt = getContext();
      if (getDynamicNbPrefetch())
        return OpFoldResult(getDynamicNbPrefetch());
      return OpFoldResult(IntegerAttr::get(
                          IntegerType::get(cxt, 64), getStaticNbPrefetch()));
    }
  }];
}
````
- **EN**: This TableGen block defines `InsertPrefetchOp` as a `def` record for `XeGPUTransformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `InsertPrefetchOp` 定义为 `def` 记录，用于描述 `XeGPUTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 190-265
````tablegen
def ConvertLayoutOp : Op<Transform_Dialect, "xegpu.convert_layout", [
  AttrSizedOperandSegments,
  DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
  TransformOpInterface
]> {

  let summary = "Convert xegpu.layout attribute for a value.";
  let description = [{
    Adds an `xegpu.convert_layout` op to convert the `xegpu.layout` attribute
    of a value before its first use. The input and target layouts are defined
    by the `*sg_layout`, `*sg_data` and optional `*inst_data` and `*order`
    attributes. Returns a handle to the emitted `xegpu.convert_layout` op.
  }];

  let arguments = (ins TransformValueHandleTypeInterface:$target,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$input_sg_layout,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$input_sg_data,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$input_inst_data,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$target_sg_layout,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$target_sg_data,
                   Variadic<TransformAnyParamTypeOrAnyHandle>:$target_inst_data,
                   DefaultValuedOptionalAttr<DenseI32ArrayAttr, "{}">:$input_order,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_input_sg_layout,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_input_sg_data,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_input_inst_data,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_target_sg_layout,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_target_sg_data,
                   DefaultValuedOptionalAttr<DenseI64ArrayAttr, "{}">:$static_target_inst_data,
                   DefaultValuedOptionalAttr<DenseI32ArrayAttr, "{}">:$target_order
                   );

  let results = (outs TransformHandleTypeInterface:$newConvertOp);
  let builders = [
    OpBuilder<(ins "Value":$target,
                   "ArrayRef<OpFoldResult>":$mixedInputSgLayout,
                   "ArrayRef<OpFoldResult>":$mixedInputSgData,
                   "ArrayRef<OpFoldResult>":$mixedInputInstData,
                   "ArrayRef<int32_t>":$inputOrder,
                   "ArrayRef<OpFoldResult>":$mixedTargetSgLayout,
                   "ArrayRef<OpFoldResult>":$mixedTargetSgData,
                   "ArrayRef<OpFoldResult>":$mixedTargetInstData,
                   "ArrayRef<int32_t>":$targetOrder
                   )>,
  ];

  let assemblyFormat = [{
    $target
    `input_sg_layout` `=` custom<DynamicIndexList>($input_sg_layout, $static_input_sg_layout)
    `input_sg_data` `=` custom<DynamicIndexList>($input_sg_data, $static_input_sg_data)
    (`input_inst_data` `=` custom<DynamicIndexList>($input_inst_data, $static_input_inst_data)^)?
    (`input_order` `=` $input_order^)?
    `target_sg_layout` `=` custom<DynamicIndexList>($target_sg_layout, $static_target_sg_layout)
    `target_sg_data` `=` custom<DynamicIndexList>($target_sg_data, $static_target_sg_data)
    (`target_inst_data` `=` custom<DynamicIndexList>($target_inst_data, $static_target_inst_data)^)?
    (`target_order` `=` $target_order^)?
    attr-dict `:` functional-type(operands, results)
  }];

  let extraClassDeclaration = [{
    ::mlir::DiagnosedSilenceableFailure apply(
        ::mlir::transform::TransformRewriter &rewriter,
        ::mlir::transform::TransformResults &transformResults,
        ::mlir::transform::TransformState &state);

    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedInputSgLayout() {
      Builder b(getContext());
      return getMixedValues(getStaticInputSgLayout(), getInputSgLayout(), b);
    }
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedInputSgData() {
      Builder b(getContext());
      return getMixedValues(getStaticInputSgData(), getInputSgData(), b);
    }
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedInputInstData() {
      Builder b(getContext());
      return getMixedValues(getStaticInputInstData(), getInputInstData(), b);
    }
````
- **EN**: This TableGen block defines `ConvertLayoutOp` as a `def` record for `XeGPUTransformOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `ConvertLayoutOp` 定义为 `def` 记录，用于描述 `XeGPUTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 266-279
````tablegen
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedTargetSgLayout() {
      Builder b(getContext());
      return getMixedValues(getStaticTargetSgLayout(), getTargetSgLayout(), b);
    }
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedTargetSgData() {
      Builder b(getContext());
      return getMixedValues(getStaticTargetSgData(), getTargetSgData(), b);
    }
    ::llvm::SmallVector<::mlir::OpFoldResult> getMixedTargetInstData() {
      Builder b(getContext());
      return getMixedValues(getStaticTargetInstData(), getTargetInstData(), b);
    }
  }];
}
````
- **EN**: This block groups callable interfaces such as `getMixedTargetSgLayout`, `b`, `getContext`, `getMixedValues`, indicating how `XeGPUTransformOps` is queried or updated.
- **CN**: 该代码块聚合了 `getMixedTargetSgLayout`, `b`, `getContext`, `getMixedValues` 等可调用接口，展示了如何查询或更新 `XeGPUTransformOps`。

### Lines 281-281
````tablegen
#endif // XEGPU_TRANSFORM_OPS
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

- mlir/Dialect/Transform/IR/TransformAttrs.td
- mlir/Dialect/Transform/IR/TransformDialect.td
- mlir/Dialect/Transform/Interfaces/TransformInterfaces.td
- mlir/Dialect/Transform/IR/TransformTypes.td
- mlir/Interfaces/SideEffectInterfaces.td
- mlir/IR/OpBase.td
- GetLoadOp builds on Op<Transform_Dialect, "xegpu.get_load_op", [
- SetAnchorLayoutOp builds on Op<Transform_Dialect, "xegpu.set_anchor_layout", [
- SetGPULaunchThreadsOp builds on Op<Transform_Dialect, "xegpu.set_gpu_launch_threads", [
- InsertPrefetchOp builds on Op<Transform_Dialect, "xegpu.insert_prefetch", [
- ConvertLayoutOp builds on Op<Transform_Dialect, "xegpu.convert_layout", [
