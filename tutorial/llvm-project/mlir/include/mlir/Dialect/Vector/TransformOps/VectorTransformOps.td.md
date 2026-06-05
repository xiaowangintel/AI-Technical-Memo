# VectorTransformOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Vector/TransformOps/VectorTransformOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR VectorTransformOps component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 VectorTransformOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
````tablegen
//===- VectorTransformOps.td - Vector transform ops --------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef VECTOR_TRANSFORM_OPS
#define VECTOR_TRANSFORM_OPS

include "mlir/Dialect/Transform/IR/TransformDialect.td"
include "mlir/Dialect/Transform/Interfaces/TransformInterfaces.td"
include "mlir/Dialect/Vector/Transforms/VectorTransformsBase.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 18-38
````tablegen
def ApplyVectorToLLVMConversionPatternsOp : Op<Transform_Dialect,
    "apply_conversion_patterns.vector.vector_to_llvm",
    [DeclareOpInterfaceMethods<ConversionPatternDescriptorOpInterface,
                               ["verifyTypeConverter"]>]> {
  let description = [{
    Collects patterns that convert vector dialect ops to LLVM dialect ops. These
    patterns require an "LLVMTypeConverter".

    The patterns can be customized as follows:
    - `reassociate_fp_reductions`: Allows LLVM to reassociate floating-point
      reductions for speed.
    - `force_32bit_vector_indices`: Allows the compiler to assume that vector
      indices fit in 32-bit if that yields faster code.
  }];

  let arguments = (ins
      DefaultValuedAttr<BoolAttr, "false">:$reassociate_fp_reductions,
      DefaultValuedAttr<BoolAttr, "true">:$force_32bit_vector_indices,
      DefaultValuedAttr<BoolAttr, "false">:$use_vector_alignment);
  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyVectorToLLVMConversionPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `ApplyVectorToLLVMConversionPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, 验证钩子。

### Lines 41-54
````tablegen
def ApplyCastAwayVectorLeadingOneDimPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.cast_away_vector_leading_one_dim",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Collect a set of leading one dimension removal patterns.

    These patterns insert vector.shape_cast to remove leading one dimensions
    to expose more canonical forms of read/write/insert/extract operations.
    With them, there are more chances that we can cancel out extract-insert
    pairs or forward write-read pairs.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyCastAwayVectorLeadingOneDimPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyCastAwayVectorLeadingOneDimPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 56-70
````tablegen
def ApplyRankReducingSubviewPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.rank_reducing_subview_patterns",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Apply opt-in vector transfer permutation patterns that include:
      - TransferReadDropUnitDimsPattern
      - TransferWriteDropUnitDimsPattern

    These patterns have the effect of rewriting a vector.transfer with unit
    dimensions into a rank-reduced version thanks to subview operations.
    This is complemented by shape_cast folding patterns.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyRankReducingSubviewPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyRankReducingSubviewPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 72-86
````tablegen
def ApplyDropUnitDimWithShapeCastPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.drop_unit_dims_with_shape_cast",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
     Apply vector patterns to fold unit dims with vector.shape_cast Ops:
      - DropUnitDimFromElementwiseOps
      - DropUnitDimsFromScfForOp
      - DropUnitDimsFromTransposeOp

    Excludes patterns for vector.transfer Ops. This is complemented by
    shape_cast folding patterns.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyDropUnitDimWithShapeCastPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyDropUnitDimWithShapeCastPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 88-100
````tablegen
def ApplyDropInnerMostUnitDimsFromXferOpsPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.drop_inner_most_unit_dims_from_xfer_ops",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Apply vector patterns to drop the inner most unit dims from
    vector.transfer_read and vector.transfer_write Ops by taking a subview (via
    memref.subview) of the original source/destination MemRef. Since it
    requires the input/ouptu to be MemRefs, this Op is only helpful
    past-bufferization.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyDropInnerMostUnitDimsFromXferOpsPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyDropInnerMostUnitDimsFromXferOpsPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 102-121
````tablegen
def ApplyTransferPermutationPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.transfer_permutation_patterns",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Apply opt-in vector transfer permutation patterns that include:
      - TransferReadPermutationLowering
      - TransferWritePermutationLowering
      - TransferOpReduceRank
      - TransferWriteNonPermutationLowering

    These patterns have the effect of rewriting a vector.transfer with an
    arbitrary permutation_map to a vector.transfer with a permutation_map that
    is a minor identity followed by a vector.transpose.

    In other words, this makes the vector.transfer contiguous on the most minor
    dimensions and materializes the permutation_map as a vector.transpose.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyTransferPermutationPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyTransferPermutationPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 123-135
````tablegen
def ApplyLowerBitCastPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_bitcast",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector bitcast operations should be lowered to
    finer-grained vector primitives.

    This is usally a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerBitCastPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerBitCastPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 137-149
````tablegen
def ApplyLowerBroadcastPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_broadcast",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector broadcast operations should be lowered to
    finer-grained vector primitives.

    This is usally a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerBroadcastPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerBroadcastPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 151-168
````tablegen
def ApplyLowerContractionPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_contraction",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector contraction-like operations should be lowered to
    finer-grained vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let arguments = (ins DefaultValuedAttr<VectorContractLoweringAttr,
      "vector::VectorContractLowering::OuterProduct">:$lowering_strategy
  );
  let assemblyFormat = [{
    (`lowering_strategy` `=` $lowering_strategy^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyLowerContractionPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerContractionPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 170-179
````tablegen
def ApplyLowerCreateMaskPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_create_mask",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector create_mask-like operations should be lowered to
    finer-grained vector primitives.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerCreateMaskPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerCreateMaskPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 181-193
````tablegen
def ApplyLowerMasksPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_masks",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector.create_mask and vector.constant_mask operations
    should be lowered to finer-grained vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerMasksPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerMasksPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 195-210
````tablegen
def ApplyLowerMaskedTransfersPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_masked_transfers",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Apply opt-in patterns that lower vector.mask operations surrounding
    side-effecting ops:
      - MaskedTransferReadOpPattern
      - MaskedTransferWriteOpPattern
      - MaskedGatherOpPattern

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerMaskedTransfersPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerMaskedTransfersPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 212-224
````tablegen
def ApplyMaterializeMasksPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.materialize_masks",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that mask operations should be lowered to fine-grained arithemtic
    operations.

    This is usually the last step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyMaterializeMasksPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyMaterializeMasksPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 226-246
````tablegen
def ApplyReorderMultiReductionPatternsOp: Op<Transform_Dialect,
    "apply_patterns.vector.reorder_multi_reduction_dims",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector multi_reduction-like operations should be
    transformed such that all reduction dimensions become innermost or
    outermost, depending on `lowering_strategy`.

    This populates the patterns from
    `populateVectorMultiReductionReorderPatterns`, i.e.:
    * `InnerOuterDimReductionConversion`
  }];

  let arguments = (ins DefaultValuedAttr<VectorMultiReductionLoweringAttr,
      "vector::VectorMultiReductionLowering::InnerParallel">:$lowering_strategy
  );

  let assemblyFormat = [{
    (`lowering_strategy` `=` $lowering_strategy^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyReorderMultiReductionPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyReorderMultiReductionPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 248-263
````tablegen
def ApplyMultiReductionFlatteningPatternsOp: Op<Transform_Dialect,
    "apply_patterns.vector.multi_reduction_flattening",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector multi_reduction operations should be flattened from
    more than 2-D to 2-D.
  }];

  let arguments = (ins DefaultValuedAttr<VectorMultiReductionLoweringAttr,
      "vector::VectorMultiReductionLowering::InnerParallel">:$lowering_strategy
  );

  let assemblyFormat = [{
    (`lowering_strategy` `=` $lowering_strategy^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyMultiReductionFlatteningPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyMultiReductionFlatteningPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 265-289
````tablegen
def ApplyMultiReductionUnrollingPatternsOp: Op<Transform_Dialect,
    "apply_patterns.vector.multi_reduction_unrolling",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector multi_reduction operations should be unrolled.
    1-D multi_reductions are converted directly to vector.reduction.
    2-D multi_reductions are unrolled into either a sequence of
    vector.reduction ops (innerreduction) or element-wise arith ops
    (innerparallel).

    This populates the patterns from
    `populateVectorMultiReductionUnrollingPatterns`, i.e.:
    * `OneDimMultiReductionToReduction`
    * `TwoDimMultiReductionToReduction` (innerreduction)
    * `TwoDimMultiReductionToElementWise` (innerparallel)
  }];

  let arguments = (ins DefaultValuedAttr<VectorMultiReductionLoweringAttr,
      "vector::VectorMultiReductionLowering::InnerParallel">:$lowering_strategy
  );

  let assemblyFormat = [{
    (`lowering_strategy` `=` $lowering_strategy^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyMultiReductionUnrollingPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyMultiReductionUnrollingPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 291-303
````tablegen
def ApplyLowerOuterProductPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_outerproduct",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that the vector outerproduct operations should be lowered to
    finer-grained vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerOuterProductPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerOuterProductPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 305-314
````tablegen
def ApplyLowerGatherPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_gather",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector.gather operations should be lowered to
    finer-grained vector primitives.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerGatherPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerGatherPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 316-325
````tablegen
def ApplyUnrollFromElementsPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.unroll_from_elements",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector from_elements operations should be unrolled
    along the outermost dimension.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyUnrollFromElementsPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyUnrollFromElementsPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 327-336
````tablegen
def ApplyUnrollToElementsPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.unroll_to_elements",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector to_elements operations should be unrolled
    along the outermost dimension.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyUnrollToElementsPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyUnrollToElementsPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 338-347
````tablegen
def ApplyLowerScanPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_scan",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector.scan operations should be lowered to
    finer-grained vector primitives.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerScanPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerScanPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 349-361
````tablegen
def ApplyLowerShapeCastPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_shape_cast",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector shape_cast operations should be lowered to
    finer-grained vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerShapeCastPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerShapeCastPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 363-379
````tablegen
def ApplyLowerTransferPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_transfer",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector transfer operations should be lowered to finer-grained
    vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let arguments = (ins DefaultValuedAttr<I64Attr, "1">:$max_transfer_rank);

  let assemblyFormat = [{
    (`max_transfer_rank` `=` $max_transfer_rank^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyLowerTransferPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerTransferPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 381-405
````tablegen
def ApplyLowerTransposePatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_transpose",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector transpose-like operations should be lowered to
    finer-grained vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let arguments = (ins
     DefaultValuedAttr<VectorTransposeLoweringAttr,
       "vector::VectorTransposeLowering::EltWise">:$lowering_strategy,
     DefaultValuedAttr<BoolAttr, "false">:$avx2_lowering_strategy
  );

  let assemblyFormat = [{
    oilist (
      `lowering_strategy` `=` $lowering_strategy
      | `avx2_lowering_strategy` `=` $avx2_lowering_strategy
    )
    attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyLowerTransposePatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerTransposePatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 407-419
````tablegen
def ApplyLowerInterleavePatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.lower_interleave",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector interleave operations should be lowered to
    finer-grained vector primitives.

    This is usally a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyLowerInterleavePatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyLowerInterleavePatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 421-433
````tablegen
def ApplyInterleaveAndDeinterleaveToShufflePatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.interleave_and_deinterleave_to_shuffle",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that 1D vector interleave and deinterleave operations should be
    rewritten as vector shuffle operations.

    This is motivated by some current codegen backends not handling vector
    interleave and deinterleave operations.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyInterleaveAndDeinterleaveToShufflePatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyInterleaveAndDeinterleaveToShufflePatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 435-448
````tablegen
def ApplyRewriteNarrowTypePatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.rewrite_narrow_types",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector narrow rewrite operations should be applied.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.

    Warning: these patterns currently only work for little endian targets.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyRewriteNarrowTypePatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyRewriteNarrowTypePatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 450-469
````tablegen
def ApplySplitTransferFullPartialPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.split_transfer_full_partial",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector transfer operations should be split to full and
    partial parts.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let arguments = (ins
     DefaultValuedAttr<VectorTransferSplitAttr,
       "vector::VectorTransferSplit::LinalgCopy">:$split_transfer_strategy
  );

  let assemblyFormat = [{
    (`split_transfer_strategy` `=` $split_transfer_strategy^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplySplitTransferFullPartialPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplySplitTransferFullPartialPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 471-494
````tablegen
def ApplyTransferToScfPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.transfer_to_scf",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Indicates that vector transfer operations should be rewritten with scf.for
    loops over finer-grained vector primitives.

    This is usually a late step that is run after bufferization as part of the
    process of lowering to e.g. LLVM or NVVM.
  }];

  let arguments = (ins
     DefaultValuedAttr<I64Attr, "1">:$max_transfer_rank,
     DefaultValuedAttr<BoolAttr, "false">:$full_unroll
  );

  let assemblyFormat = [{
    oilist (
        `max_transfer_rank` `=` $max_transfer_rank
      | `full_unroll` `=` $full_unroll
    )
    attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyTransferToScfPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyTransferToScfPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 496-505
````tablegen
def ApplyFoldArithExtensionPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.fold_arith_extension",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Collect a set of patterns that fold arithmetic extension on floating point
    into vector contract for the backends with native support.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyFoldArithExtensionPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyFoldArithExtensionPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 507-516
````tablegen
def ApplyFoldElementwiseToVectorPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.elementwise_to_vector",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Collect a set of patterns that fold elementwise op on vectors to the vector
    dialect.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyFoldElementwiseToVectorPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyFoldElementwiseToVectorPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 518-536
````tablegen
def ApplyVectorReductionToContractPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.reduction_to_contract",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Apply opt-in patterns that convert reductions to contract:
      - MultiReduceToContract
      - CombineContractBroadcast
      - CombineContractABTranspose
      - CombineContractResultTranspose
      - ReorderElementwiseOpsOnTranspose
      - ReorderElementwiseOpsOnBroadcast
      - ReorderCastOpsOnBroadcast

    These patterns have the effect of rewriting a vector.multi_reduce into a
    vector.contract.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplyVectorReductionToContractPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyVectorReductionToContractPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 538-561
````tablegen
def ApplySinkVectorPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.sink_ops",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Patterns that remove redundant Vector Ops by re-ordering them with
    e.g. elementwise Ops.

    Example:
    ```
    %at = vector.transpose %a, [1, 0]: vector<4x2xf32> to vector<2x4xf32>
    %bt = vector.transpose %b, [1, 0]: vector<4x2xf32> to vector<2x4xf32>
    %r = arith.addf %at, %bt : vector<2x4xf32>
    ```
    gets converted to:
    ```
    %0 = arith.addf %a, %b : vector<4x2xf32>
    %r = vector.transpose %0, [1, 0] : vector<2x4xf32>
    ```
    At the moment, these patterns are limited to vector.broadcast,
    vector.transpose and vector.extract.
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplySinkVectorPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplySinkVectorPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 563-585
````tablegen
def ApplySinkVectorMemPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.sink_mem_ops",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Patterns that replace redundant Vector Ops (followed by
    `vector.load`/`vector.store`) with either vector.load/vector.store or
    `memref.load`/`memref.store`. Currently limited to 1-element vectors.

    Example:
    ```
    vector.load %arg0[%arg1] : memref<?xf32>, vector<4xf32>
    vector.extract %0[1] : f32 from vector<4xf32>
    ```
    Gets converted to:
    ```
    %c1 = arith.constant 1 : index
    %0 = arith.addi %arg1, %c1 overflow<nsw> : index
    %1 = memref.load %arg0[%0] : memref<?xf32>
    ```
  }];

  let assemblyFormat = "attr-dict";
}
````
- **EN**: This TableGen block defines `ApplySinkVectorMemPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplySinkVectorMemPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, trait/接口组合。

### Lines 587-603
````tablegen
def ApplyFlattenVectorTransferOpsPatternsOp : Op<Transform_Dialect,
    "apply_patterns.vector.flatten_vector_transfer_ops",
    [DeclareOpInterfaceMethods<PatternDescriptorOpInterface>]> {
  let description = [{
    Collect patterns to rewrite contiguous row-major vector.transfer_read or 
    vector.transfer_write operations to a 1D operation.
  }];

  let arguments = (ins
  DefaultValuedAttr<UI32Attr,
    "std::numeric_limits<unsigned>::max()">:$target_vector_bitwidth
  );

  let assemblyFormat = [{
    (`target_vector_bitwidth` `=` $target_vector_bitwidth^)? attr-dict
  }];
}
````
- **EN**: This TableGen block defines `ApplyFlattenVectorTransferOpsPatternsOp` as a `def` record for `VectorTransformOps`. It covers assembly syntax, operand or attribute schema, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ApplyFlattenVectorTransferOpsPatternsOp` 定义为 `def` 记录，用于描述 `VectorTransformOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 语义文档, trait/接口组合。

### Lines 604-604
````tablegen
#endif // VECTOR_TRANSFORM_OPS
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
- mlir/Dialect/Vector/Transforms/VectorTransformsBase.td
- mlir/Interfaces/SideEffectInterfaces.td
- mlir/IR/OpBase.td
- ApplyVectorToLLVMConversionPatternsOp builds on Op<Transform_Dialect,
- ApplyCastAwayVectorLeadingOneDimPatternsOp builds on Op<Transform_Dialect,
- ApplyRankReducingSubviewPatternsOp builds on Op<Transform_Dialect,
- ApplyDropUnitDimWithShapeCastPatternsOp builds on Op<Transform_Dialect,
- ApplyDropInnerMostUnitDimsFromXferOpsPatternsOp builds on Op<Transform_Dialect,
- ApplyTransferPermutationPatternsOp builds on Op<Transform_Dialect,
- ApplyLowerBitCastPatternsOp builds on Op<Transform_Dialect,
- ApplyLowerBroadcastPatternsOp builds on Op<Transform_Dialect,
- ApplyLowerContractionPatternsOp builds on Op<Transform_Dialect,
- ApplyLowerCreateMaskPatternsOp builds on Op<Transform_Dialect,
- ApplyLowerMasksPatternsOp builds on Op<Transform_Dialect,
- ApplyLowerMaskedTransfersPatternsOp builds on Op<Transform_Dialect,
- ApplyMaterializeMasksPatternsOp builds on Op<Transform_Dialect,
- ApplyReorderMultiReductionPatternsOp builds on Op<Transform_Dialect,
- ApplyMultiReductionFlatteningPatternsOp builds on Op<Transform_Dialect,
