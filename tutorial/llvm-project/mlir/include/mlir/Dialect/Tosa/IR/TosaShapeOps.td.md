# TosaShapeOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/IR/TosaShapeOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TosaShapeOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file defines shape operators for the TOSA dialect.
- **用途（CN）**: 为 MLIR 的 TosaShapeOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
````tablegen
//===-- TosaShapeOps.td - TOSA dialect utility operations --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines shape operators for the TOSA dialect.
//
//===----------------------------------------------------------------------===//

#ifndef TOSA_SHAPE_OPS
#define TOSA_SHAPE_OPS

include "mlir/IR/OpBase.td"

include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/LoopLikeInterface.td"
include "mlir/Dialect/Tosa/IR/TosaInterfaces.td"

include "mlir/Dialect/Tosa/IR/TosaTypesBase.td"
include "mlir/Dialect/Tosa/IR/TosaOpBase.td"

// Op trait: operator has operands and results with TOSA shape type
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 27-29
````tablegen
def TosaShapeOperator : NativeOpTrait<"TosaShapeOperator"> {
  let cppNamespace = "mlir::OpTrait::tosa";
}
````
- **EN**: This TableGen block defines `TosaShapeOperator` as a `def` record for `TosaShapeOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TosaShapeOperator` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 31-40
````tablegen
class Tosa_ShapeOp<string mnemonic, list<Trait> traits = []>
    : Tosa_Op<mnemonic, !listconcat(traits, [TosaShapeOperator])> {
  list<Availability> availability = [
    Profile<[Tosa_PRO_INT, Tosa_PRO_FP]>,
    Extension<[Tosa_EXT_SHAPE]>,
  ];

  let assemblyFormat =
      "operands attr-dict `:` functional-type(operands, results)";
}
````
- **EN**: This TableGen block defines `Tosa_ShapeOp` as a `class` record for `TosaShapeOps`. It covers assembly syntax, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_ShapeOp` 定义为 `class` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 汇编语法, trait/接口组合。

### Lines 41-41
````tablegen
// op trait: shape operator has same ranks for operands and results
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 43-46
````tablegen
def TosaShapeOperatorWithSameRanks
    : NativeOpTrait<"TosaShapeOperatorWithSameRanks"> {
  let cppNamespace = "mlir::OpTrait::tosa";
}
````
- **EN**: This TableGen block defines `TosaShapeOperatorWithSameRanks` as a `def` record for `TosaShapeOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TosaShapeOperatorWithSameRanks` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 48-51
````tablegen
class Tosa_ElementwiseShapeOp<string mnemonic, list<Trait> traits = []>
    : Tosa_ShapeOp<mnemonic,
                   !listconcat(traits, [TosaShapeOperatorWithSameRanks])> {
}
````
- **EN**: This TableGen block defines `Tosa_ElementwiseShapeOp` as a `class` record for `TosaShapeOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_ElementwiseShapeOp` 定义为 `class` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 52-54
````tablegen
//===----------------------------------------------------------------------===//
// Operator: AddShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 57-72
````tablegen
def Tosa_AddShapeOp : Tosa_ElementwiseShapeOp<"add_shape", [Pure]> {
  let summary = "Elementwise addition of shapes.";

  let description = [{
      Elementwise addition of input1 and input2. Size of shapes must match.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_AddShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_AddShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 73-75
````tablegen
//===----------------------------------------------------------------------===//
// Operator: ConcatShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 77-94
````tablegen
def Tosa_ConcatShapeOp : Tosa_ShapeOp<"concat_shape", [Pure]> {
  let summary = "Concatenates a list of shapes.";

  let description = [{
    Concatenates a list of shapes into a new shape with length equal to the sum
    of the lengths of the inputs.
  }];

  let arguments = (ins
    Variadic<Tosa_Shape>:$input
  );

  let results = (outs Tosa_Shape:$output);

  let hasVerifier = 1;

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_ConcatShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `Tosa_ConcatShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, 验证钩子。

### Lines 95-97
````tablegen
//===----------------------------------------------------------------------===//
// Operator: ConstShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 99-124
````tablegen
def Tosa_ConstShapeOp : Tosa_ShapeOp<"const_shape", [ConstantLike, Pure]> {
  let summary = "Constant Shape operator.";

  let description = [{
    A node containing a constant shape.

    Example:

    ```mlir
    // Generic form
    %out = "tosa.const_shape"() {values = dense<0> : tensor<4xindex>} : () -> !tosa.shape<4>
    ```
  }];

  let arguments = (ins IndexElementsAttr:$values);

  let results = (outs Tosa_Shape:$output);

  list<Availability> availability = [
    Profile<[Tosa_PRO_INT, Tosa_PRO_FP]>,
    Extension<[]>,
  ];

  let hasVerifier = 1;
  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_ConstShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `Tosa_ConstShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, 验证钩子。

### Lines 125-127
````tablegen
//===----------------------------------------------------------------------===//
// Operator: Dim
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 129-151
````tablegen
def Tosa_DimOp : Tosa_ShapeOp<"dim", [Pure]> {
  let summary = "Extract size of dimension from input tensor.";

  let description = [{
    Returns a length 1 shape_t of the size of the input tensor for the given axis.
  }];

  let arguments = (ins
    Tosa_TensorAtLeast1D:$input1,
    I32Attr:$axis
  );

  let results = (outs Tosa_Shape:$output);

  list<Availability> availability = [
    Profile<[Tosa_PRO_INT, Tosa_PRO_FP]>,
    Extension<[Tosa_EXT_SHAPE, Tosa_EXT_FP8E4M3, Tosa_EXT_FP8E5M2, Tosa_EXT_BF16, Tosa_EXT_MXFP, Tosa_EXT_INT64]>,
  ];

  let hasVerifier = 1;

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_DimOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `Tosa_DimOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, 验证钩子。

### Lines 152-154
````tablegen
//===----------------------------------------------------------------------===//
// Operator: DivCeilShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 156-171
````tablegen
def Tosa_DivCeilShapeOp : Tosa_ElementwiseShapeOp<"div_ceil_shape", [Pure]> {
  let summary = "Elementwise ceiling divide of shapes.";

  let description = [{
      Elementwise divide of input1 by input2. The result of the divide is rounded up.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_DivCeilShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_DivCeilShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 172-174
````tablegen
//===----------------------------------------------------------------------===//
// Operator: DivFloorShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 176-191
````tablegen
def Tosa_DivFloorShapeOp : Tosa_ElementwiseShapeOp<"div_floor_shape", [Pure]> {
  let summary = "Elementwise floor divide of shapes.";

  let description = [{
      Elementwise integer divide of input1 by input2. The result of the divide is rounded down.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_DivFloorShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_DivFloorShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 192-194
````tablegen
//===----------------------------------------------------------------------===//
// Operator: Exp2Shape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 196-210
````tablegen
def Tosa_Exp2ShapeOp : Tosa_ElementwiseShapeOp<"exp2_shape", [Pure]> {
  let summary = "Elementwise base-2 exponential of shapes.";

  let description = [{
      Computation of raising two to the power of each element in input.
  }];

  let arguments = (ins
    Tosa_Shape:$input
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_Exp2ShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_Exp2ShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 211-213
````tablegen
//===----------------------------------------------------------------------===//
// Operator: Log2CeilShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 215-229
````tablegen
def Tosa_Log2CeilShapeOp : Tosa_ElementwiseShapeOp<"log2_ceil_shape", [Pure]> {
  let summary = "Elementwise ceil base-2 logarithm of shapes.";

  let description = [{
      Computation of the base two logarithm of each element in input. Result is rounded up.
  }];

  let arguments = (ins
    Tosa_Shape:$input
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_Log2CeilShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_Log2CeilShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 230-232
````tablegen
//===----------------------------------------------------------------------===//
// Operator: Log2FloorShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 234-248
````tablegen
def Tosa_Log2FloorShapeOp : Tosa_ElementwiseShapeOp<"log2_floor_shape", [Pure]> {
  let summary = "Elementwise floor base-2 logarithm of shapes.";

  let description = [{
      Computation of the base two logarithm of each element in input. Result is rounded down.
  }];

  let arguments = (ins
    Tosa_Shape:$input
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_Log2FloorShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_Log2FloorShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 249-251
````tablegen
//===----------------------------------------------------------------------===//
// Operator: MaxShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 253-268
````tablegen
def Tosa_MaxShapeOp : Tosa_ElementwiseShapeOp<"max_shape", [Pure]> {
  let summary = "Elementwise maximum of shapes.";

  let description = [{
      Elementwise maximum of input1 and input2.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_MaxShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_MaxShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 269-271
````tablegen
//===----------------------------------------------------------------------===//
// Operator: MinShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 273-288
````tablegen
def Tosa_MinShapeOp : Tosa_ElementwiseShapeOp<"min_shape", [Pure]> {
  let summary = "Elementwise minimum of shapes.";

  let description = [{
      Elementwise minimum of input1 and input2.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_MinShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_MinShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 289-291
````tablegen
//===----------------------------------------------------------------------===//
// Operator: ModShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 293-308
````tablegen
def Tosa_ModShapeOp : Tosa_ElementwiseShapeOp<"mod_shape", [Pure]> {
  let summary = "Elementwise modulo of shapes.";

  let description = [{
    Elementwise modulo of input1 divided by input2.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_ModShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_ModShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 309-311
````tablegen
//===----------------------------------------------------------------------===//
// Operator: MulShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 313-328
````tablegen
def Tosa_MulShapeOp : Tosa_ElementwiseShapeOp<"mul_shape", [Pure]> {
  let summary = "Elementwise multiplication of shapes.";

  let description = [{
      Elementwise multiplication of input1 and input2.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_MulShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_MulShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 329-331
````tablegen
//===----------------------------------------------------------------------===//
// Operator: SliceShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 333-351
````tablegen
def Tosa_SliceShapeOp : Tosa_ShapeOp<"slice_shape", [Pure]> {
  let summary = "Extract slice of a shape.";

  let description = [{
    Extract a shape of size from input.
  }];

  let arguments = (ins
    Tosa_Shape:$input,
    Tosa_ScalarInt32Tensor:$start,
    Tosa_ScalarInt32Tensor:$size
  );

  let results = (outs Tosa_Shape:$output);

  let hasVerifier = 1;

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_SliceShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `Tosa_SliceShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, 验证钩子。

### Lines 352-354
````tablegen
//===----------------------------------------------------------------------===//
// Operator: SubShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 356-371
````tablegen
def Tosa_SubShapeOp : Tosa_ElementwiseShapeOp<"sub_shape", [Pure]> {
  let summary = "Elementwise subtraction of shapes.";

  let description = [{
      Elementwise subtraction of input1 and input2. Size of shapes must match.
  }];

  let arguments = (ins
    Tosa_Shape:$input1,
    Tosa_Shape:$input2
  );

  let results = (outs Tosa_Shape:$output);

  let hasFolder = 1;
}
````
- **EN**: This TableGen block defines `Tosa_SubShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_SubShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档, trait/接口组合。

### Lines 372-374
````tablegen
//===----------------------------------------------------------------------===//
// Operator: AssertEqualShape
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 376-389
````tablegen
def Tosa_AssertEqualShapeOp
    : Tosa_ShapeOp<"assert_equal_shape", [TosaShapeOperatorWithSameRanks]> {
  let summary = "Verify two shapes are equal.";

  let description = [{
      Verify input1 and input2 are equal. If allow_broadcast is set, shapes which
      are broadcast compatible are allowed.
  }];

  let arguments = (ins Tosa_Shape:$input1, Tosa_Shape:$input2,
      BoolAttr:$allow_broadcast);

  let results = (outs);
}
````
- **EN**: This TableGen block defines `Tosa_AssertEqualShapeOp` as a `def` record for `TosaShapeOps`. It covers operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_AssertEqualShapeOp` 定义为 `def` 记录，用于描述 `TosaShapeOps` 相关的声明式信息。 其中涉及 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 390-390
````tablegen
#endif // TOSA_SHAPE_OPS
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

- mlir/IR/OpBase.td
- mlir/Interfaces/SideEffectInterfaces.td
- mlir/Interfaces/InferTypeOpInterface.td
- mlir/Interfaces/LoopLikeInterface.td
- mlir/Dialect/Tosa/IR/TosaInterfaces.td
- mlir/Dialect/Tosa/IR/TosaTypesBase.td
- mlir/Dialect/Tosa/IR/TosaOpBase.td
- TosaShapeOperator builds on NativeOpTrait<"TosaShapeOperator">
- TosaShapeOperatorWithSameRanks builds on NativeOpTrait<"TosaShapeOperatorWithSameRanks">
- Tosa_AddShapeOp builds on Tosa_ElementwiseShapeOp<"add_shape", [Pure]>
- Tosa_ConcatShapeOp builds on Tosa_ShapeOp<"concat_shape", [Pure]>
- Tosa_ConstShapeOp builds on Tosa_ShapeOp<"const_shape", [ConstantLike, Pure]>
- Tosa_DimOp builds on Tosa_ShapeOp<"dim", [Pure]>
- Tosa_DivCeilShapeOp builds on Tosa_ElementwiseShapeOp<"div_ceil_shape", [Pure]>
- Tosa_DivFloorShapeOp builds on Tosa_ElementwiseShapeOp<"div_floor_shape", [Pure]>
- Tosa_Exp2ShapeOp builds on Tosa_ElementwiseShapeOp<"exp2_shape", [Pure]>
- Tosa_Log2CeilShapeOp builds on Tosa_ElementwiseShapeOp<"log2_ceil_shape", [Pure]>
- Tosa_Log2FloorShapeOp builds on Tosa_ElementwiseShapeOp<"log2_floor_shape", [Pure]>
- Tosa_MaxShapeOp builds on Tosa_ElementwiseShapeOp<"max_shape", [Pure]>
- Tosa_MinShapeOp builds on Tosa_ElementwiseShapeOp<"min_shape", [Pure]>
