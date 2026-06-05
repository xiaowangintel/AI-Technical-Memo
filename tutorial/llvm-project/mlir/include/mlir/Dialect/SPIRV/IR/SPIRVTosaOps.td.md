# SPIRVTosaOps.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVTosaOps.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR SPIRVTosaOps component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This is the op definition spec of TOSA extension ops.
- **用途（CN）**: 为 MLIR 的 SPIRVTosaOps 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
````tablegen
//===- SPIRVTosaOps.td - TOSA extended insts spec file -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is the op definition spec of TOSA extension ops.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_TOSA_OPS
#define MLIR_DIALECT_SPIRV_IR_TOSA_OPS

include "mlir/Dialect/SPIRV/IR/SPIRVBase.td"
include "mlir/Dialect/SPIRV/IR/SPIRVGraphOps.td"
include "mlir/Dialect/SPIRV/IR/SPIRVTosaTypes.td"
include "mlir/Interfaces/SideEffectInterfaces.td"

//===----------------------------------------------------------------------===//
// SPIR-V TOSA opcode specification.
//===----------------------------------------------------------------------===//

// Base class for all TOSA ops.
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 26-38
````tablegen
class SPIRV_TosaOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_ExtInstOp<mnemonic, "Tosa", "TOSA.001000.1", opcode,
  !listconcat(traits, [InGraphScope])> {

  let availability = [
    MinVersion<SPIRV_V_1_5>,
    MaxVersion<SPIRV_V_1_6>,
    Extension<[SPV_ARM_graph]>,
    Capability<[SPIRV_C_GraphARM]>
  ];

  let hasVerifier = 0;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaOp` as a `class` record for `SPIRVTosaOps`. It covers verification hooks, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 验证钩子, trait/接口组合。

### Lines 40-48
````tablegen
class SPIRV_TosaOpWithResult<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOp<mnemonic, opcode, traits> {

  code extraBaseClassDeclaration = [{
    ::mlir::spirv::TensorArmType getResultType() {
      return cast<::mlir::spirv::TensorArmType>(getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaOpWithResult` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaOpWithResult` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 50-63
````tablegen
class SPIRV_TosaOpWithComplexResult<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOp<mnemonic, opcode, traits> {

  code extraBaseClassDeclaration = [{
    ::mlir::spirv::TensorArmType getResultRealType() {
      auto resultType = cast<StructType>(getType());
      return cast<::mlir::spirv::TensorArmType>(resultType.getElementType(0));
    }
    ::mlir::spirv::TensorArmType getResultImagType() {
      auto resultType = cast<StructType>(getType());
      return cast<::mlir::spirv::TensorArmType>(resultType.getElementType(1));
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaOpWithComplexResult` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaOpWithComplexResult` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 65-74
````tablegen
class SPIRV_TosaElementwiseUnaryOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOpWithResult<mnemonic, opcode, !listconcat(traits,
    [AllTypesMatch<["input1", "output"]>])> {

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaElementwiseUnaryOp` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaElementwiseUnaryOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 76-91
````tablegen
class SPIRV_TosaFloatElementwiseUnaryOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaElementwiseUnaryOp<mnemonic, opcode, !listconcat(traits, [Pure])> {

  let arguments = (ins
    SPIRV_F16OrF32OrBF16_TensorArm: $input1
  );

  let results = (outs
    SPIRV_F16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaFloatElementwiseUnaryOp` as a `class` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaFloatElementwiseUnaryOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, trait/接口组合。

### Lines 93-107
````tablegen
class SPIRV_TosaBinaryOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOpWithResult<mnemonic, opcode, !listconcat(traits, [
    AllElementTypesMatch<["input1", "input2"]>,
    AllRanksMatch<["input1", "input2", "output"]>,
    MatchBroadcastableShapes<"input1", "input2", "output">])> {

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
    ::mlir::spirv::TensorArmType getInput2Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput2().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaBinaryOp` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaBinaryOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 109-112
````tablegen
class SPIRV_TosaElementwiseBinaryOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaBinaryOp<mnemonic, opcode, !listconcat(traits, [
    AllElementTypesMatch<["input1", "output"]>])> {
}
````
- **EN**: This TableGen block defines `SPIRV_TosaElementwiseBinaryOp` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaElementwiseBinaryOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 114-131
````tablegen
class SPIRV_TosaIntegerElementwiseBinaryOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaElementwiseBinaryOp<mnemonic, opcode, traits> {

  let arguments = (ins
    SPIRV_I8OrI16OrI32_TensorArm: $input1,
    SPIRV_I8OrI16OrI32_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I8OrI16OrI32_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaIntegerElementwiseBinaryOp` as a `class` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaIntegerElementwiseBinaryOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, trait/接口组合。

### Lines 133-150
````tablegen
class SPIRV_TosaBoolElementwiseBinaryOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaElementwiseBinaryOp<mnemonic, opcode, !listconcat(traits, [Pure])> {

  let arguments = (ins
    SPIRV_Bool_TensorArm: $input1,
    SPIRV_Bool_TensorArm: $input2
  );

  let results = (outs
    SPIRV_Bool_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaBoolElementwiseBinaryOp` as a `class` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaBoolElementwiseBinaryOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, trait/接口组合。

### Lines 152-190
````tablegen
class SPIRV_TosaConvolutionOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOpWithResult<mnemonic, opcode, !listconcat(traits, [NoMemoryEffect,
    TypeConstraintImplicationOn<"input", I8, "output", [I32]>,
    TypeConstraintImplicationOn<"input", I16, "output", [I64]>,
    TypeConstraintImplicationOn<"input", BF16, "output", [BF16]>,
    TypeConstraintImplicationOn<"input", F16, "output", [F16]>,
    TypeConstraintImplicationOn<"input", F32, "output", [F32]>,
    TypeConstraintImplicationOn<"input", F8E4M3FN, "output", [F16]>,
    TypeConstraintImplicationOn<"input", F8E5M2, "output", [F16]>,
    TypeConstraintImplicationOn<"input", I8, "weight", [I8]>,
    TypeConstraintImplicationOn<"input", I16, "weight", [I8]>,
    TypeConstraintImplicationOn<"input", BF16, "weight", [BF16]>,
    TypeConstraintImplicationOn<"input", F16, "weight", [F16]>,
    TypeConstraintImplicationOn<"input", F32, "weight", [F32]>,
    TypeConstraintImplicationOn<"input", F8E4M3FN, "weight", [F8E4M3FN]>,
    TypeConstraintImplicationOn<"input", F8E5M2, "weight", [F8E5M2]>,
    TypeImpliesAccType<"input", I8, ["INT32"]>,
    TypeImpliesAccType<"input", I16, ["INT48"]>,
    TypeImpliesAccType<"input", F16, ["FP16", "FP32"]>,
    TypeImpliesAccType<"input", BF16, ["FP32"]>,
    TypeImpliesAccType<"input", F32, ["FP32"]>,
    TypeImpliesAccType<"input", F8E4M3FN, ["FP16"]>,
    TypeImpliesAccType<"input", F8E5M2, ["FP16"]>,
    AllElementTypesMatch<["bias", "output"]>,
    AllElementTypesMatch<["input", "input_zp"]>,
    AllElementTypesMatch<["weight", "weight_zp"]>])> {

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
    ::mlir::spirv::TensorArmType getWeightType() {
      return cast<::mlir::spirv::TensorArmType>(getWeight().getType());
    }
    ::mlir::spirv::TensorArmType getBiasType() {
      return cast<::mlir::spirv::TensorArmType>(getBias().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaConvolutionOp` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaConvolutionOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 192-220
````tablegen
class SPIRV_TosaComparisonOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOpWithResult<mnemonic, opcode, !listconcat(traits, [Pure,
  AllElementTypesMatch<["input1", "input2"]>,
  MatchBroadcastableShapes<"input1", "input2", "output">])> {

  let arguments = (ins
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input2
  );

  let results = (outs
    SPIRV_Bool_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
    ::mlir::spirv::TensorArmType getInput2Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput2().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaComparisonOp` as a `class` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaComparisonOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, trait/接口组合。

### Lines 222-234
````tablegen
class SPIRV_TosaReductionOp<string mnemonic, int opcode, list<Trait> traits = []> :
  SPIRV_TosaOpWithResult<mnemonic, opcode, !listconcat(traits, [
  AllElementTypesMatch<["input", "output"]>,
  AllRanksMatch<["input", "output"]>,
  AxisValueLessThanRankOf<"input">,
  ReductionOutputShapeMatchesInputAndAxis<"input", "output">])> {

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReductionOp` as a `class` record for `SPIRVTosaOps`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReductionOp` 定义为 `class` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 236-280
````tablegen
def SPIRV_TosaArgMaxOp : SPIRV_TosaOpWithResult<"ArgMax", 0, [Pure,
  OutputRankIsInputRankMinusOne<"input", "output">,
  AxisValueLessThanRankOf<"input">]> {
  let summary = "Perform argmax on the input.";

  let description = [{
    Returns the index with the largest value across the given axis of the
    input tensor. If multiple locations have equal values, returns the first
    match along the search axis.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_argmax
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_argmax

    #### Example:
    ```mlir
    %2 = spirv.Tosa.ArgMax axis = 3, nan_mode = <Propagate>, %input : !spirv.arm.tensor<3x28x17x17xi8> -> !spirv.arm.tensor<3x28x17xi32>
    %2 = spirv.Tosa.ArgMax axis = 2, nan_mode = <Propagate>, %input : !spirv.arm.tensor<2x2x7x14xf32> -> !spirv.arm.tensor<2x2x14xi32>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm: $input
  );

  let results = (outs
    SPIRV_I32_TensorArmUpTo5D: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    `nan_mode` `=` $nan_mode `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaArgMaxOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaArgMaxOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 283-347
````tablegen
def SPIRV_TosaAvgPool2DOp : SPIRV_TosaOpWithResult<"AvgPool2D", 1, [NoMemoryEffect,
  TypeImpliesAccType<"input", I8, ["INT32"]>,
  TypeImpliesAccType<"input", I16, ["INT32"]>,
  TypeImpliesAccType<"input", F16, ["FP16", "FP32"]>,
  TypeImpliesAccType<"input", BF16, ["FP32"]>,
  TypeImpliesAccType<"input", F32, ["FP32"]>,
  TypeImpliesAccType<"input", F8E4M3FN, ["FP16"]>,
  TypeImpliesAccType<"input", F8E5M2, ["FP16"]>,
  AllElementTypesMatch<["input", "input_zp", "output", "output_zp"]>,
  NHWCInputOutputShapeMatch<"input", "output">,
  Pool2DPadValuesLessThanKernel<"pad", "kernel">]> {
  let summary = "Performs average pooling on the input.";

  let description = [{
    Performs an average pooling over the given input tensor. A sliding
    window of size given by <kernel size> is passed over the input tensor, with
    the mean value being placed in the output tensor. When calculating the
    average, only the number of valid input tensor values, but not padding, are
    used to calculate the divisor. The behaviour is undefined if the accumulated
    result overflows or underflows.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_avg_pool2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_avg_pool2d

    #### Example:
    ```mlir
    %6 = spirv.Tosa.AvgPool2D kernel = [3, 3], stride = [1, 2], pad = [0, 1, 0, 0], acc_type = <INT32>, %input, %input_zp, %output_zp : !spirv.arm.tensor<1x3x65537x1xi8>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<1x2x32768x1xi8>
    %6 = spirv.Tosa.AvgPool2D kernel = [2, 2], stride = [1, 1], pad = [1, 0, 0, 0], acc_type = <FP32>, %input, %input_zp, %output_zp : !spirv.arm.tensor<1x2x65533x2xf32>, !spirv.arm.tensor<1xf32>, !spirv.arm.tensor<1xf32> -> !spirv.arm.tensor<1x2x65532x2xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $kernel,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $stride,
    SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr: $pad,
    SPIRV_TosaExtAccTypeAttr: $acc_type,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $input,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $input_zp,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $output_zp
  );

  let results = (outs
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $output
  );

  let assemblyFormat = [{
    `kernel` `=` custom<SPIRV_I32_1DArmTensor>($kernel) `,`
    `stride` `=` custom<SPIRV_I32_1DArmTensor>($stride) `,`
    `pad` `=` custom<SPIRV_I32_1DArmTensor>($pad) `,`
    `acc_type` `=` $acc_type `,`
    $input `,`
    $input_zp `,`
    $output_zp
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaAvgPool2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaAvgPool2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 350-406
````tablegen
def SPIRV_TosaConv2DOp : SPIRV_TosaConvolutionOp<"Conv2D", 2, [
  Conv2DShapeMatch<"input", "weight", "bias", "output">]> {
  let summary = "2D Convolution operator.";

  let description = [{
    Performs a 2D convolution over the given tensor input, using the weight
    tensor. Implementations may choose to skip calculation of multiplies in
    the padding area.

    Input and weight have respective zero point values provided in input_zp and weight_zp.

    The behaviour is undefined if the accumulated result overflows or underflows.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_conv2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_conv2d

    #### Example:
    ```mlir
    %7 = spirv.Tosa.Conv2D pad = [1, 0, 0, 0], stride = [1, 2], dilation = [7, 1], acc_type = <INT32>, local_bound = false, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x65535x3x1xi8>, !spirv.arm.tensor<7x1x1x1xi8>, !spirv.arm.tensor<1xi32>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<1x65536x2x7xi32>
    %7 = spirv.Tosa.Conv2D pad = [0, 0, 0, 0], stride = [1, 1], dilation = [1, 1], acc_type = <FP16>, local_bound = true, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x34x18x27xf16>, !spirv.arm.tensor<11x1x1x27xf16>, !spirv.arm.tensor<11xf16>, !spirv.arm.tensor<1xf16>, !spirv.arm.tensor<1xf16> -> !spirv.arm.tensor<1x34x18x11xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr: $pad,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $stride,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $dilation,
    SPIRV_TosaExtAccTypeAttr: $acc_type,
    SPIRV_BoolConstAttr: $local_bound,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $input,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm4D: $weight,
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D: $bias,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $input_zp,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $weight_zp
  );

  let results = (outs
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm4D: $output
  );

  let assemblyFormat = [{
    `pad` `=` custom<SPIRV_I32_1DArmTensor>($pad) `,`
    `stride` `=` custom<SPIRV_I32_1DArmTensor>($stride) `,`
    `dilation` `=` custom<SPIRV_I32_1DArmTensor>($dilation) `,`
    `acc_type` `=` $acc_type `,`
    `local_bound` `=` $local_bound `,`
    $input `,`
    $weight `,`
    $bias `,`
    $input_zp `,`
    $weight_zp
    attr-dict `:` type(operands) `->` type(results)
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaConv2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaConv2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 409-464
````tablegen
def SPIRV_TosaConv3DOp : SPIRV_TosaConvolutionOp<"Conv3D", 3, [
  Conv3DShapeMatch<"input", "weight", "bias", "output">]> {
  let summary = "3D Convolution operator.";

  let description = [{
    Performs a 3D convolution over the given input tensor. Implementations
    may choose to skip calculation of multiplies in the padding area.

    Input and weight have respective zero point values provided in input_zp and weight_zp.

    The behaviour is undefined if the accumulated result overflows or underflows.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_conv3d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_conv3d

    #### Example:
    ```mlir
    %7 = spirv.Tosa.Conv3D pad = [0, 0, 0, 0, 0, 0], stride = [1, 1, 1], dilation = [1, 1, 1], acc_type = <INT32>, local_bound = false, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x9x21x14x1xi8>, !spirv.arm.tensor<2x1x2x1x1xi8>, !spirv.arm.tensor<1xi32>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<1x9x20x14x2xi32>
    %7 = spirv.Tosa.Conv3D pad = [0, 1, 1, 0, 0, 1], stride = [1, 1, 1], dilation = [1, 1, 7], acc_type = <FP32>, local_bound = false, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x2x65539x1x2xf32>, !spirv.arm.tensor<1x1x1x1x2xf32>, !spirv.arm.tensor<1xf32>, !spirv.arm.tensor<1xf32>, !spirv.arm.tensor<1xf32> -> !spirv.arm.tensor<1x3x65540x2x1xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_NonNegativeI32_1DTensorArmOfLength6Attr: $pad,
    SPIRV_PositiveI32_1DTensorArmOfLength3Attr: $stride,
    SPIRV_PositiveI32_1DTensorArmOfLength3Attr: $dilation,
    SPIRV_TosaExtAccTypeAttr: $acc_type,
    SPIRV_BoolConstAttr: $local_bound,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm5D: $input,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm5D: $weight,
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D: $bias,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $input_zp,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $weight_zp
  );

  let results = (outs
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm5D: $output
  );

  let assemblyFormat = [{
    `pad` `=` custom<SPIRV_I32_1DArmTensor>($pad) `,`
    `stride` `=` custom<SPIRV_I32_1DArmTensor>($stride) `,`
    `dilation` `=` custom<SPIRV_I32_1DArmTensor>($dilation) `,`
    `acc_type` `=` $acc_type `,`
    `local_bound` `=` $local_bound `,`
    $input `,`
    $weight `,`
    $bias `,`
    $input_zp `,`
    $weight_zp
    attr-dict `:` type(operands) `->` type(results)
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaConv3DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaConv3DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 467-523
````tablegen
def SPIRV_TosaDepthwiseConv2DOp : SPIRV_TosaConvolutionOp<"DepthwiseConv2D", 4, [
  DepthwiseConv2DShapeMatch<"input", "weight", "bias", "output">]> {
  let summary = "Depthwise 2D Convolution operator.";

  let description = [{
    Performs 2D convolutions separately over each channel of the given tensor
    input, using the weight tensor. Implementations may choose to skip
    calculation of multiplies in the padding area.

    Input and weight have respective zero point values provided in input_zp and weight_zp.

    The behaviour is undefined if the accumulated result overflows or underflows.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_depthwise_conv2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_depthwise_conv2d

    #### Example:
    ```mlir
    %7 = spirv.Tosa.DepthwiseConv2D pad = [0, 0, 0, 0], stride = [1, 2], dilation = [7, 7], acc_type = <INT32>, local_bound = false, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x4x65537x1xi8>, !spirv.arm.tensor<1x3x1x4xi8>, !spirv.arm.tensor<4xi32>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<1x4x32762x4xi32>
    %7 = spirv.Tosa.DepthwiseConv2D pad = [0, 1, 1, 1], stride = [1, 2], dilation = [1, 7], acc_type = <FP32>, local_bound = true, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x65540x1x3xf32>, !spirv.arm.tensor<1x1x3x1xf32>, !spirv.arm.tensor<1xf32>, !spirv.arm.tensor<1xf32>, !spirv.arm.tensor<1xf32> -> !spirv.arm.tensor<1x65541x2x3xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr: $pad,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $stride,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $dilation,
    SPIRV_TosaExtAccTypeAttr: $acc_type,
    SPIRV_BoolConstAttr: $local_bound,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $input,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm4D: $weight,
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D: $bias,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $input_zp,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $weight_zp
  );

  let results = (outs
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm4D: $output
  );

  let assemblyFormat = [{
    `pad` `=` custom<SPIRV_I32_1DArmTensor>($pad) `,`
    `stride` `=` custom<SPIRV_I32_1DArmTensor>($stride) `,`
    `dilation` `=` custom<SPIRV_I32_1DArmTensor>($dilation) `,`
    `acc_type` `=` $acc_type `,`
    `local_bound` `=` $local_bound `,`
    $input `,`
    $weight `,`
    $bias `,`
    $input_zp `,`
    $weight_zp
    attr-dict `:` type(operands) `->` type(results)
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaDepthwiseConv2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaDepthwiseConv2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 526-578
````tablegen
def SPIRV_TosaFFT2DOp : SPIRV_TosaOpWithComplexResult<"FFT2D", 5, [Pure,
  AllShapesMatch<["input_real", "input_imag"]>,
  Struct2TensorArmShapesMatchInput<"input_real">,
  TensorDimsArePowerOfTwo<"input_real", [1, 2]>]> {
  let summary = "Performs FFT2D operation on the input.";

  let description = [{
    Performs a batched complex 2D Fast Fourier Transform over the input. The
    complex input values are constructed from the corresponding values in the
    input_real and input_imag tensors. The resulting values in the output are
    split into the output_real and output_imag tensors. No normalization is
    applied on either the forward or inverse versions of the operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_fft2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_fft2d

    #### Example:
    ```mlir
    %0 = spirv.Tosa.FFT2D inverse = true, local_bound = false, %input_real, %input_imag : !spirv.arm.tensor<1x32x32xf32>, !spirv.arm.tensor<1x32x32xf32> -> !spirv.struct<(!spirv.arm.tensor<1x32x32xf32>, !spirv.arm.tensor<1x32x32xf32>)>
    %1 = spirv.CompositeExtract %0[0 : i32] : !spirv.struct<(!spirv.arm.tensor<1x32x32xf32>, !spirv.arm.tensor<1x32x32xf32>)>
    %2 = spirv.CompositeExtract %0[1 : i32] : !spirv.struct<(!spirv.arm.tensor<1x32x32xf32>, !spirv.arm.tensor<1x32x32xf32>)>
    ```
  }];

  let arguments = (ins
    SPIRV_BoolConstAttr: $inverse,
    SPIRV_BoolConstAttr: $local_bound,
    SPIRV_F32_TensorArm3D: $input_real,
    SPIRV_F32_TensorArm3D: $input_imag
  );

  let results = (outs
    SPIRV_Struct_2_F32_TensorArm3D: $output
  );

  let assemblyFormat = [{
    `inverse` `=` $inverse `,`
    `local_bound` `=` $local_bound `,`
    $input_real `,`
    $input_imag
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputRealType() {
      return cast<::mlir::spirv::TensorArmType>(getInputReal().getType());
    }
    ::mlir::spirv::TensorArmType getInputImagType() {
      return cast<::mlir::spirv::TensorArmType>(getInputImag().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaFFT2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaFFT2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 581-638
````tablegen
def SPIRV_TosaMatMulOp : SPIRV_TosaOpWithResult<"MatMul", 6, [NoMemoryEffect,
  MatMulShapesMatch<"A", "B", "output">,
  TypeConstraintImplicationOn<"A", I8, "output", [I32]>,
  TypeConstraintImplicationOn<"A", I16, "output", [I64]>,
  TypeConstraintImplicationOn<"A", BF16, "output", [F32]>,
  TypeConstraintImplicationOn<"A", F16, "output", [F16, F32]>,
  TypeConstraintImplicationOn<"A", F32, "output", [F32]>,
  TypeConstraintImplicationOn<"A", F8E4M3FN, "output", [F16]>,
  TypeConstraintImplicationOn<"A", F8E5M2, "output", [F16]>,
  AllElementTypesMatch<["A", "A_zp", "B", "B_zp"]>]> {
  let summary = "Matrix Multiplication operator.";

  let description = [{
    Performs two dimensional matrix multiplications.

    A, B are the inputs with respective zero point values in A_zp, B_zp.

    The behaviour is undefined if the accumulated result overflows or underflows.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_matmul
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_matmul

    #### Example:
    ```mlir
    %2 = spirv.Tosa.MatMul %A, %B, %A_zp, %B_zp : !spirv.arm.tensor<8x2x3xi8>, !spirv.arm.tensor<8x3x8xi8>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<8x2x8xi32>
    %2 = spirv.Tosa.MatMul %A, %B, %A_zp, %B_zp : !spirv.arm.tensor<15x39x50xf16>, !spirv.arm.tensor<15x50x24xf16>, !spirv.arm.tensor<1xf16>, !spirv.arm.tensor<1xf16> -> !spirv.arm.tensor<15x39x24xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm3D: $A,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm3D: $B,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $A_zp,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $B_zp
  );

  let results = (outs
    SPIRV_I32OrI64OrF16OrF32_TensorArm3D: $output
  );

  let assemblyFormat = [{
    $A `,`
    $B `,`
    $A_zp `,`
    $B_zp
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getAType() {
      return cast<::mlir::spirv::TensorArmType>(getA().getType());
    }
    ::mlir::spirv::TensorArmType getBType() {
      return cast<::mlir::spirv::TensorArmType>(getB().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaMatMulOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaMatMulOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 641-692
````tablegen
def SPIRV_TosaMaxPool2DOp : SPIRV_TosaOpWithResult<"MaxPool2D", 7, [Pure,
  AllElementTypesMatch<["input", "output"]>,
  NHWCInputOutputShapeMatch<"input", "output">,
  Pool2DPadValuesLessThanKernel<"pad", "kernel">]> {
  let summary = "Performs max pooling on the input.";

  let description = [{
    Performs a max pooling over the given input tensor. A sliding window of
    size given by <kernel size> is passed over the input tensor, with the
    maximum value being placed in the output tensor.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_max_pool2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_max_pool2d

    #### Example:
    ```mlir
    %4 = spirv.Tosa.MaxPool2D kernel = [3, 2], stride = [1, 2], pad = [1, 0, 0, 1], nan_mode = <Propagate>, %input : !spirv.arm.tensor<1x3x65537x1xi8> -> !spirv.arm.tensor<1x2x32769x1xi8>
    %4 = spirv.Tosa.MaxPool2D kernel = [3, 2], stride = [2, 2], pad = [1, 0, 1, 1], nan_mode = <Propagate>, %input : !spirv.arm.tensor<1x6x65536x1xf32> -> !spirv.arm.tensor<1x3x32769x1xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $kernel,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $stride,
    SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr: $pad,
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $input
  );

  let results = (outs
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $output
  );

  let assemblyFormat = [{
    `kernel` `=` custom<SPIRV_I32_1DArmTensor>($kernel) `,`
    `stride` `=` custom<SPIRV_I32_1DArmTensor>($stride) `,`
    `pad` `=` custom<SPIRV_I32_1DArmTensor>($pad) `,`
    `nan_mode` `=` $nan_mode `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaMaxPool2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaMaxPool2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 695-742
````tablegen
def SPIRV_TosaRFFT2DOp : SPIRV_TosaOpWithComplexResult<"RFFT2D", 8, [Pure,
  Struct2TensorArmRFFT2DShapesMatchInput<"input_real">,
  TensorDimsArePowerOfTwo<"input_real", [1, 2]>]> {
  let summary = "Performs RFFT2D operation on the input.";

  let description = [{
    Performs a batched 2D real-valued Fast Fourier Transform over the input where
    the input tensor consists of real values producing complex valued output. The
    complex output values will be split into the output_real and output_imag
    tensor arguments. This operator takes advantage of Hermitian symmetry to only
    calculate the first half of the final output axis. Implementations may choose
    to skip calculation of the imaginary values at (0,0), (0,W/2), (H/2,0), and
    (H/2, W/2). If the calculation is skipped, the result at that location must be
    zero.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_rfft2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_rfft2d

    #### Example:
    ```mlir
    %0 = spirv.Tosa.RFFT2D local_bound = false, %input_real : !spirv.arm.tensor<1x32x32xf32> -> !spirv.struct<(!spirv.arm.tensor<1x32x17xf32>, !spirv.arm.tensor<1x32x17xf32>)>
    %1 = spirv.CompositeExtract %0[0 : i32] : !spirv.struct<(!spirv.arm.tensor<1x32x17xf32>, !spirv.arm.tensor<1x32x17xf32>)>
    %2 = spirv.CompositeExtract %0[1 : i32] : !spirv.struct<(!spirv.arm.tensor<1x32x17xf32>, !spirv.arm.tensor<1x32x17xf32>)>
    ```
  }];

  let arguments = (ins
    SPIRV_BoolConstAttr: $local_bound,
    SPIRV_F32_TensorArm3D: $input_real
  );

  let results = (outs
    SPIRV_Struct_2_F32_TensorArm3D: $output
  );

  let assemblyFormat = [{
    `local_bound` `=` $local_bound `,`
    $input_real
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputRealType() {
      return cast<::mlir::spirv::TensorArmType>(getInputReal().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaRFFT2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaRFFT2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 745-799
````tablegen
def SPIRV_TosaTransposeConv2DOp : SPIRV_TosaConvolutionOp<"TransposeConv2D", 9, [
  Conv2DShapeMatch<"input", "weight", "bias", "output">]> {
  let summary = "Transpose 2D Convolution operator.";

  let description = [{
    Performs a 2D transposed convolution over the given tensor input, using the
    weights tensor. Implementations may choose to skip calculation of multiplies
    by zero at fractional input positions.

    Input and weight have respective zero point values provided in input_zp and weight_zp.

    The behaviour is undefined if the accumulated result overflows or underflows.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_transpose_conv2d
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_transpose_conv2d

    #### Example:
    ```mlir
    %6 = spirv.Tosa.TransposeConv2D out_pad = [0, 0, 0, 0], stride = [1, 1], acc_type = <INT48>, local_bound = false, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<1x13x33x3xi16>, !spirv.arm.tensor<11x1x3x3xi8>, !spirv.arm.tensor<1xi64>, !spirv.arm.tensor<1xi16>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<1x13x35x11xi64>
    %6 = spirv.Tosa.TransposeConv2D out_pad = [0, 1, 0, 0], stride = [1, 8], acc_type = <FP16>, local_bound = true, %input, %weight, %bias, %input_zp, %weight_zp : !spirv.arm.tensor<10x24x9x13xf16>, !spirv.arm.tensor<14x1x1x13xf16>, !spirv.arm.tensor<14xf16>, !spirv.arm.tensor<1xf16>, !spirv.arm.tensor<1xf16> -> !spirv.arm.tensor<10x25x65x14xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_I32_1DTensorArmOfLength4Attr: $out_pad,
    SPIRV_PositiveI32_1DTensorArmOfLength2Attr: $stride,
    SPIRV_TosaExtAccTypeAttr: $acc_type,
    SPIRV_BoolConstAttr: $local_bound,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D: $input,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm4D: $weight,
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D: $bias,
    SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $input_zp,
    SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $weight_zp
  );

  let results = (outs
    SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm4D: $output
  );

  let assemblyFormat = [{
    `out_pad` `=` custom<SPIRV_I32_1DArmTensor>($out_pad) `,`
    `stride` `=` custom<SPIRV_I32_1DArmTensor>($stride) `,`
    `acc_type` `=` $acc_type `,`
    `local_bound` `=` $local_bound `,`
    $input `,`
    $weight `,`
    $bias `,`
    $input_zp `,`
    $weight_zp
    attr-dict `:` type(operands) `->` type(results)
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaTransposeConv2DOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaTransposeConv2DOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 802-852
````tablegen
def SPIRV_TosaClampOp : SPIRV_TosaOpWithResult<"Clamp", 10, [Pure,
  AllTypesMatch<["input", "output"]>,
  AllElementTypesMatch<["input", "output", "min_val", "max_val"]>,
  ClampBoundsAreNotNaN<"min_val", "max_val">,
  ClampMinValLessThanOrEqualToMaxVal<"min_val", "max_val">]> {
  let summary = "Computes Clamp(min, max).";

  let description = [{
    Clamp to an arbitrary minimum and maximum value. Maximum and minimum values
    are specified as values in the range of the input type. Integer element
    types are always interpreted as signed.
    No zero point subtraction is done to the values, thus to clamp to the zero
    point value, the zero point itself should be supplied as the minimum value.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_clamp
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_clamp

    #### Example:
    ```mlir
    %3 = spirv.Tosa.Clamp min_val = -102 : i8, max_val = -100 : i8, nan_mode = <Propagate>, %input : !spirv.arm.tensor<27x44x55xi8> -> !spirv.arm.tensor<27x44x55xi8>
    %3 = spirv.Tosa.Clamp min_val = -1.19339396E+38 : f32, max_val = 2.38255944E+38 : f32, nan_mode = <Ignore>, %input : !spirv.arm.tensor<18x5x17x6xf32> -> !spirv.arm.tensor<18x5x17x6xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrF16OrF32OrBF16ConstAttr: $min_val,
    SPIRV_I8OrI16OrF16OrF32OrBF16ConstAttr: $max_val,
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `min_val` `=` $min_val `,`
    `max_val` `=` $max_val `,`
    `nan_mode` `=` $nan_mode `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaClampOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaClampOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 855-891
````tablegen
def SPIRV_TosaErfOp : SPIRV_TosaOpWithResult<"Erf", 11, [Pure,
  AllTypesMatch<["input", "output"]>]> {
  let summary = "Gauss Error Function.";

  let description = [{
    Gauss Error Function: $ erf(x) = \frac{2}{\sqrt{\pi}} \int_{0}^{x} e^{-t^2} dt $
    For quantized integer data types, the `spirv.Tosa.Table` operator should be used instead.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_erf
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_erf

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Erf %input : !spirv.arm.tensor<47x38x51xf32> -> !spirv.arm.tensor<47x38x51xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_F16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_F16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaErfOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaErfOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 894-932
````tablegen
def SPIRV_TosaSigmoidOp : SPIRV_TosaOpWithResult<"Sigmoid", 12, [Pure,
  AllTypesMatch<["input", "output"]>]> {
  let summary = "Sigmoid operator.";

  let description = [{
    Applies the sigmoid logistic function to each element of the input tensor:
    $ sigmoid(x) = \frac{1}{1 + e^{-x}} $.

    For quantized integer data types, the `spirv.Tosa.Table` operator should be used instead.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_sigmoid
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_sigmoid

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Sigmoid %input : !spirv.arm.tensor<28x43x45xf32> -> !spirv.arm.tensor<28x43x45xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_F16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_F16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaSigmoidOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaSigmoidOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 935-972
````tablegen
def SPIRV_TosaTanhOp : SPIRV_TosaOpWithResult<"Tanh", 13, [Pure,
  AllTypesMatch<["input", "output"]>]> {
  let summary = "Hyperbolic Tangent operator.";

  let description = [{
    Elementwise Parameterized Hyperbolic Tangent: $ tanh(x) = \frac{1 - e^{-2x}}{1 + e^{-2x}} $.

    For quantized integer data types, the `spirv.Tosa.Table` operator should be used instead.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_tanh
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_tanh

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Tanh %input : !spirv.arm.tensor<46x50x36xf16> -> !spirv.arm.tensor<46x50x36xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_F16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_F16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaTanhOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaTanhOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 975-1009
````tablegen
def SPIRV_TosaAddOp : SPIRV_TosaElementwiseBinaryOp<"Add", 14, [NoMemoryEffect]> {
  let summary = "Addition operator.";

  let description = [{
    Elementwise Addition of input1 and input2. Axis of size 1 will be broadcast,
    as necessary. Rank of input tensors must match. Integer element types are
    always interpreted as signed. The behavior is undefined if the integer
    addition overflows or underflows the signed integer range.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_add
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_add

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Add %input1, %input2 : !spirv.arm.tensor<4x7x3x10xi32>, !spirv.arm.tensor<4x7x3x1xi32> -> !spirv.arm.tensor<4x7x3x10xi32>
    %0 = spirv.Tosa.Add %input1, %input2 : !spirv.arm.tensor<26x37x18xf16>, !spirv.arm.tensor<1x37x18xf16> -> !spirv.arm.tensor<26x37x18xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaAddOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaAddOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1012-1052
````tablegen
def SPIRV_TosaArithmeticRightShiftOp : SPIRV_TosaElementwiseBinaryOp<"ArithmeticRightShift", 15, [NoMemoryEffect]> {
  let summary = "Arithmetic Right Shift.";

  let description = [{
    Elementwise Arithmetic Right Shift of input1 by the amount specified in
    input2. Axis of size 1 will be broadcast, as necessary. Rank of input
    tensors must match. Integer element types are always interpreted as signed.
    The behavior is undefined if the shift value is negative or greater or
    equal to the bitwidth of the element type.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_arithmetic_right_shift
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_arithmetic_right_shift

    #### Example:
    ```mlir
    %1 = spirv.Tosa.ArithmeticRightShift round = true, %input1, %shift : !spirv.arm.tensor<1x47x22xi16>, !spirv.arm.tensor<49x47x22xi16> -> !spirv.arm.tensor<49x47x22xi16>
    ```

    If an element of `%shift` is `3`, the matching element of `%input1` is
    shifted right by 3 bits; because this is an arithmetic shift, the sign bit
    is preserved.
  }];

  let arguments = (ins
    SPIRV_BoolConstAttr: $round,
    SPIRV_I8OrI16OrI32_TensorArm: $input1,
    SPIRV_I8OrI16OrI32_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I8OrI16OrI32_TensorArm: $output
  );

  let assemblyFormat = [{
    `round` `=` $round `,`
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaArithmeticRightShiftOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaArithmeticRightShiftOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1055-1072
````tablegen
def SPIRV_TosaBitwiseAndOp : SPIRV_TosaIntegerElementwiseBinaryOp<"BitwiseAnd", 16, [Pure]> {
  let summary = "Bitwise AND operator.";

  let description = [{
    Elementwise Bitwise AND of input1 and input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match. Integer element
    types are always interpreted as signed.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_bitwise_and
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_bitwise_and

    #### Example:
    ```mlir
    %0 = spirv.Tosa.BitwiseAnd %input1, %input2 : !spirv.arm.tensor<4x1x7x12xi16>, !spirv.arm.tensor<4x13x7x12xi16> -> !spirv.arm.tensor<4x13x7x12xi16>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaBitwiseAndOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaBitwiseAndOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 1075-1092
````tablegen
def SPIRV_TosaBitwiseOrOp : SPIRV_TosaIntegerElementwiseBinaryOp<"BitwiseOr", 17, [Pure]> {
  let summary = "Bitwise OR operator.";

  let description = [{
    Elementwise Bitwise OR of input1 and input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match. Integer element
    types are always interpreted as signed.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_bitwise_or
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_bitwise_or

    #### Example:
    ```mlir
    %0 = spirv.Tosa.BitwiseOr %input1, %input2 : !spirv.arm.tensor<11x30x23xi32>, !spirv.arm.tensor<1x30x23xi32> -> !spirv.arm.tensor<11x30x23xi32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaBitwiseOrOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaBitwiseOrOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 1095-1112
````tablegen
def SPIRV_TosaBitwiseXorOp : SPIRV_TosaIntegerElementwiseBinaryOp<"BitwiseXor", 18, [Pure]> {
  let summary = "Bitwise XOR operator.";

  let description = [{
    Elementwise Bitwise XOR of input1 and input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match. Integer element
    types are always interpreted as signed.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_bitwise_xor
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_bitwise_xor

    #### Example:
    ```mlir
    %0 = spirv.Tosa.BitwiseXor %input1, %input2 : !spirv.arm.tensor<4x8x13x9xi16>, !spirv.arm.tensor<4x8x1x9xi16> -> !spirv.arm.tensor<4x8x13x9xi16>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaBitwiseXorOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaBitwiseXorOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 1115-1154
````tablegen
def SPIRV_TosaIntDivOp : SPIRV_TosaElementwiseBinaryOp<"IntDiv", 19, [NoMemoryEffect]> {
  let summary = "Integer Divide operator.";

  let description = [{
    Elementwise Integer Divide of input1 by input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match.  Integer element
    types are always interpreted as signed. The behavior is undefined if the
    divisor value is equal to zero and if the divisor value is min_value<si32>
    and the dividend is -1.

    The result of the divide is truncated towards zero. Expected use is for
    operations on non-scaled integers. Floating point divide should use
    `spirv.Tosa.Reciprocal` and `spirv.Tosa.Mul`. Quantized integer divide
    should use `spirv.Tosa.Table`(for $ 1/x $) and `spirv.Tosa.Mul`.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_intdiv
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_intdiv

    #### Example:
    ```mlir
    %0 = spirv.Tosa.IntDiv %input1, %input2 : !spirv.arm.tensor<1x65533x1xi32>, !spirv.arm.tensor<2x65533x1xi32> -> !spirv.arm.tensor<2x65533x1xi32>
    ```
  }];

  let arguments = (ins
    SPIRV_I32_TensorArm: $input1,
    SPIRV_I32_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I32_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaIntDivOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaIntDivOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1157-1173
````tablegen
def SPIRV_TosaLogicalAndOp : SPIRV_TosaBoolElementwiseBinaryOp<"LogicalAnd", 20> {
  let summary = "Logical AND operator.";

  let description = [{
    Elementwise Logical AND of input1 and input2. Axis of size 1 will be
    broadcast, as necessary. Rank of input tensors must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_logical_and
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_logical_and

    #### Example:
    ```mlir
    %0 = spirv.Tosa.LogicalAnd %input1, %input2 : !spirv.arm.tensor<2x1x7x11xi1>, !spirv.arm.tensor<2x4x7x11xi1> -> !spirv.arm.tensor<2x4x7x11xi1>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogicalAndOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogicalAndOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1176-1198
````tablegen
def SPIRV_TosaLogicalLeftShiftOp : SPIRV_TosaIntegerElementwiseBinaryOp<"LogicalLeftShift", 21, [NoMemoryEffect]> {
  let summary = "Logical Left Shift operator.";

  let description = [{
    Elementwise Logical Left Shift of input1 by the amount specified in input2.
    Axis of size 1 will be broadcast, as necessary. Rank of input tensors
    must match.  Integer element types are always interpreted as signed. The
    behavior is undefined if the shift value is negative or greater or equal to
    the bitwidth of the element type.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_logical_left_shift
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_logical_left_shift

    #### Example:
    ```mlir
    %0 = spirv.Tosa.LogicalLeftShift %input1, %shift : !spirv.arm.tensor<7x1x11x4xi8>, !spirv.arm.tensor<7x8x11x4xi8> -> !spirv.arm.tensor<7x8x11x4xi8>
    ```

    If an element of `%shift` is `2`, the matching element of `%input1` is
    shifted left by 2 bits.
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogicalLeftShiftOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogicalLeftShiftOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1201-1223
````tablegen
def SPIRV_TosaLogicalRightShiftOp : SPIRV_TosaIntegerElementwiseBinaryOp<"LogicalRightShift", 22, [NoMemoryEffect]> {
  let summary = "Logical Right Shift operator.";

  let description = [{
    Elementwise Logical Right Shift of input1 by the amount specified in input2.
    Axis of size 1 will be broadcast, as necessary. Rank of input tensors must
    match.  Integer element types are always interpreted as signed. The
    behavior is undefined if the shift value is negative or greater or equal to
    the bitwidth of the element type.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_logical_right_shift
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_logical_right_shift

    #### Example:
    ```mlir
    %0 = spirv.Tosa.LogicalRightShift %input1, %shift : !spirv.arm.tensor<6x13x1x19xi8>, !spirv.arm.tensor<6x13x6x19xi8> -> !spirv.arm.tensor<6x13x6x19xi8>
    ```

    If an element of `%shift` is `2`, the matching element of `%input1` is
    shifted right by 2 bits, filling the new high bits with zero.
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogicalRightShiftOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogicalRightShiftOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1226-1242
````tablegen
def SPIRV_TosaLogicalOrOp : SPIRV_TosaBoolElementwiseBinaryOp<"LogicalOr", 23> {
  let summary = "Logical OR operator.";

  let description = [{
    Elementwise logical OR of input1 and input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_logical_or
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_logical_or

    #### Example:
    ```mlir
    %0 = spirv.Tosa.LogicalOr %input1, %input2 : !spirv.arm.tensor<3x6x12x5xi1>, !spirv.arm.tensor<3x6x1x5xi1> -> !spirv.arm.tensor<3x6x12x5xi1>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogicalOrOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogicalOrOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1245-1261
````tablegen
def SPIRV_TosaLogicalXorOp : SPIRV_TosaBoolElementwiseBinaryOp<"LogicalXor", 24> {
  let summary = "Logical XOR operator.";

  let description = [{
    Elementwise logical XOR of input1 and input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_logical_xor
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_logical_xor

    #### Example:
    ```mlir
    %0 = spirv.Tosa.LogicalXor %input1, %input2 : !spirv.arm.tensor<11x4x9x12xi1>, !spirv.arm.tensor<11x4x9x1xi1> -> !spirv.arm.tensor<11x4x9x12xi1>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogicalXorOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogicalXorOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1264-1300
````tablegen
def SPIRV_TosaMaximumOp : SPIRV_TosaElementwiseBinaryOp<"Maximum", 25, [Pure]> {
  let summary = "Maximum.";

  let description = [{
    Elementwise maximum of input1 and input2. Axis of size 1 will be broadcast,
    as necessary. Rank of input tensors must match. Integer element types are
    always interpreted as signed.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_maximum
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_maximum

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Maximum nan_mode = <Propagate>, %input1, %input2 : !spirv.arm.tensor<1x2x65533x1xi32>, !spirv.arm.tensor<1x2x65533x2xi32> -> !spirv.arm.tensor<1x2x65533x2xi32>
    %1 = spirv.Tosa.Maximum nan_mode = <Ignore>, %input1, %input2 : !spirv.arm.tensor<1x12x14x7xf16>, !spirv.arm.tensor<11x12x14x7xf16> -> !spirv.arm.tensor<11x12x14x7xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `nan_mode` `=` $nan_mode `,`
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaMaximumOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaMaximumOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1303-1339
````tablegen
def SPIRV_TosaMinimumOp : SPIRV_TosaElementwiseBinaryOp<"Minimum", 26, [Pure]> {
  let summary = "Minimum.";

  let description = [{
    Elementwise minimum of input1 and input2. Axis of size 1 will be broadcast,
    as necessary. Rank of input tensors must match. Integer element types are
    always interpreted as signed.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_minimum
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_minimum

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Minimum nan_mode = <Propagate>, %input1, %input2 : !spirv.arm.tensor<15x2x10x11xi32>, !spirv.arm.tensor<15x1x10x11xi32> -> !spirv.arm.tensor<15x2x10x11xi32>
    %1 = spirv.Tosa.Minimum nan_mode = <Propagate>, %input1, %input2 : !spirv.arm.tensor<1x65531x2x1xf32>, !spirv.arm.tensor<1x1x2x1xf32> -> !spirv.arm.tensor<1x65531x2x1xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `nan_mode` `=` $nan_mode `,`
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaMinimumOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaMinimumOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1342-1392
````tablegen
def SPIRV_TosaMulOp : SPIRV_TosaBinaryOp<"Mul", 27, [NoMemoryEffect,
  AllElementTypesMatch<["input1", "input2"]>,
  AllRanksMatch<["input1", "input2", "output"]>,
  TypeConstraintImplicationOn<"input1", F16, "output", [F16]>,
  TypeConstraintImplicationOn<"input1", F32, "output", [F32]>,
  TypeConstraintImplicationOn<"input1", BF16, "output", [BF16]>,
  TypeConstraintImplicationOn<"input1", AnyInteger, "output", [I32]>]> {
  let summary = "Multiplication operator.";

  let description = [{
    Elementwise Multiplication (Hadamard product) of input1 and input2.
    Axis of size 1 will be broadcast, as necessary. Rank of input tensors must
    match.  Integer element types are always interpreted as signed.
    The shift value is used to right shift the result (rounded up) and
    it is used only if input element type is i32 and it must be zero for the
    other element types. The behavior is undefined if the integer
    multiplication overflows or underflows the signed integer range or the
    shift value exceeds the bitwidth of i32.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_mul
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_mul

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Mul %input1, %input2, %shift : !spirv.arm.tensor<34x21x39xi32>, !spirv.arm.tensor<34x21x1xi32>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<34x21x39xi32>
    %1 = spirv.Tosa.Mul %input1, %input2, %shift : !spirv.arm.tensor<57x1x55xf16>, !spirv.arm.tensor<57x37x55xf16>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<57x37x55xf16>
    ```

    For integer multiplication, `%shift = [7]` would round the product and
    shift it right by 7 bits; for floating-point inputs the shift tensor must
    be `[0]`.
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $input2,
    SPIRV_I8_1DTensorArmOfLength1: $shift
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2 `,`
    $shift
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaMulOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaMulOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1395-1429
````tablegen
def SPIRV_TosaPowOp : SPIRV_TosaElementwiseBinaryOp<"Pow", 28, [NoMemoryEffect]> {
  let summary = "Power operator.";

  let description = [{
    Elementwise input1 value raised to the Power of input2.
    Axis of size 1 will be broadcast, as necessary. Rank of input tensors must
    match. The behavior is undefined if the the input1 value is negative, or
    both input values are zero or negative, or both input values are NaN or
    infinite.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_pow
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_pow

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Pow %input1, %input2 : !spirv.arm.tensor<1x52x53xf16>, !spirv.arm.tensor<44x52x53xf16> -> !spirv.arm.tensor<44x52x53xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_F16OrF32OrBF16_TensorArm: $input1,
    SPIRV_F16OrF32OrBF16_TensorArm: $input2
  );

  let results = (outs
    SPIRV_F16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaPowOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaPowOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1432-1466
````tablegen
def SPIRV_TosaSubOp : SPIRV_TosaElementwiseBinaryOp<"Sub", 29, [NoMemoryEffect]> {
  let summary = "Subtraction operator.";

  let description = [{
    Elementwise Subtraction of input1 and input2. Axis of size 1 will be
    broadcast as necessary. Rank of input tensors must match. Integer element
    types are always interpreted as signed. The behavior is undefined if the
    integer subtraction overflows or underflows the signed integer range.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_sub
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_sub

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Sub %input1, %input2 : !spirv.arm.tensor<6x10x6x6xi32>, !spirv.arm.tensor<1x10x6x6xi32> -> !spirv.arm.tensor<6x10x6x6xi32>
    %0 = spirv.Tosa.Sub %input1, %input2 : !spirv.arm.tensor<1x10x13x12xf16>, !spirv.arm.tensor<6x10x13x12xf16> -> !spirv.arm.tensor<6x10x13x12xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input2
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input2
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaSubOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaSubOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1469-1530
````tablegen
def SPIRV_TosaTableOp : SPIRV_TosaOpWithResult<"Table", 30, [NoMemoryEffect,
  AllElementTypesMatch<["input1", "table"]>,
  AllShapesMatch<["input1", "output"]>,
  TypeConstraintImplicationOn<"input1", I8, "output", [I8]>,
  TypeConstraintImplicationOn<"input1", I16, "output", [I32]>,
  TableSizeConstraint<"input1", I8, 256>,
  TableSizeConstraint<"input1", I16, 513>]> {
  let summary = "Table lookup operator.";

  let description = [{
    Table lookup operation. For int8_t, perform a 256 entry table lookup
    returning an int8_t value. For int16_t tables, the int16_t input is treated
    as a fixed-point 9.7 value. The most significant 9 bits are used to index
    into the table. The fractional 7 bits are used to interpolate based on
    table[index] and table[index+1]. The behavior is undefined if during the
    interpolation step, the integer subtraction table[index+1] - table[index]
    overflows or underflows the signed int16 range.

    For int16_t inputs, this operator returns a 16.7 interpolated value in an
    int32_t. This value can then be input to the `spirv.Tosa.Rescale` operator
    to scale to the required output data type. Note that int16_t table has 513
    values to handle table[index+1] when index=511.

    An int16_t to int16_t table lookup can be constructed as follows:
    * Use the table operator to produce a fixed point 16.7 interpolated result
    * Use `spirv.Tosa.Rescale` (in_t=int32_t, out_t=int16_t, scale=1<<14, shift=21)
      to scale the output to int16_t range (or alternate scale as required)

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_table
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_table

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Table %input, %table : !spirv.arm.tensor<3x2x15x7xi8>, !spirv.arm.tensor<256xi8> -> !spirv.arm.tensor<3x2x15x7xi8>
    ```

    In the example, `%table` is the 256-entry lookup table, such as a
    precomputed quantized activation curve.
  }];

  let arguments = (ins
    SPIRV_I8OrI16_TensorArm: $input1,
    SPIRV_I8OrI16_TensorArm1D: $table
  );

  let results = (outs
    SPIRV_I8OrI32_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $table
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaTableOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaTableOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1533-1564
````tablegen
def SPIRV_TosaAbsOp : SPIRV_TosaElementwiseUnaryOp<"Abs", 31, [NoMemoryEffect]> {
  let summary = "Absolute value operation.";

  let description = [{
    Elementwise Absolute value operation. Input and Output types must match.
    Integer element types are always interpreted as signed. The behaviour is
    undefined if the input value is min_value<si32>.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_abs
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_abs

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Abs %input1 : !spirv.arm.tensor<5x1x4x4xi32> -> !spirv.arm.tensor<5x1x4x4xi32>
    %0 = spirv.Tosa.Abs %input1 : !spirv.arm.tensor<3x6x14x8xf16> -> !spirv.arm.tensor<3x6x14x8xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input1
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaAbsOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaAbsOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1567-1595
````tablegen
def SPIRV_TosaBitwiseNotOp : SPIRV_TosaElementwiseUnaryOp<"BitwiseNot", 32, [Pure]> {
  let summary = "Bitwise NOT operator.";

  let description = [{
    Elementwise Bitwise NOT of input tensor. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_bitwise_not
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_bitwise_not

    #### Example:
    ```mlir
    %0 = spirv.Tosa.BitwiseNot %input1 : !spirv.arm.tensor<12x56x50xi32> -> !spirv.arm.tensor<12x56x50xi32>
    ```
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrI32_TensorArm: $input1
  );

  let results = (outs
    SPIRV_I8OrI16OrI32_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaBitwiseNotOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaBitwiseNotOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1598-1613
````tablegen
def SPIRV_TosaCeilOp : SPIRV_TosaFloatElementwiseUnaryOp<"Ceil", 33> {
  let summary = "Ceil operator.";

  let description = [{
    Elementwise Ceiling operation. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_ceil
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_ceil

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Ceil %input1 : !spirv.arm.tensor<46x55x53xf16> -> !spirv.arm.tensor<46x55x53xf16>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaCeilOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaCeilOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1616-1644
````tablegen
def SPIRV_TosaClzOp : SPIRV_TosaElementwiseUnaryOp<"Clz", 34, [Pure]> {
  let summary = "Count Leading Zero operator.";

  let description = [{
    Elementwise Count Leading Zeros operation. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_clz
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_clz

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Clz %input1 : !spirv.arm.tensor<14x10x7x5xi32> -> !spirv.arm.tensor<14x10x7x5xi32>
    ```
  }];

  let arguments = (ins
    SPIRV_I32_TensorArm: $input1
  );

  let results = (outs
    SPIRV_I32_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaClzOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaClzOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1647-1662
````tablegen
def SPIRV_TosaCosOp : SPIRV_TosaFloatElementwiseUnaryOp<"Cos", 35> {
  let summary = "Cosine operator.";

  let description = [{
    Elementwise Cosine operation for values given in radians. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_cos
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_cos

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Cos %input1 : !spirv.arm.tensor<44x49x51xf32> -> !spirv.arm.tensor<44x49x51xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaCosOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaCosOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1665-1680
````tablegen
def SPIRV_TosaExpOp : SPIRV_TosaFloatElementwiseUnaryOp<"Exp", 36> {
  let summary = "Exp operator.";

  let description = [{
    Elementwise e to the power of x operation. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_exp
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_exp

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Exp %input1 : !spirv.arm.tensor<37x53x47xf32> -> !spirv.arm.tensor<37x53x47xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaExpOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaExpOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1683-1698
````tablegen
def SPIRV_TosaFloorOp : SPIRV_TosaFloatElementwiseUnaryOp<"Floor", 37> {
  let summary = "Floor operator.";

  let description = [{
    Elementwise Floor operation. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_floor
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_floor

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Floor %input1 : !spirv.arm.tensor<40x52x42xf32> -> !spirv.arm.tensor<40x52x42xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaFloorOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaFloorOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1701-1716
````tablegen
def SPIRV_TosaLogOp : SPIRV_TosaFloatElementwiseUnaryOp<"Log", 38> {
  let summary = "Log operator.";

  let description = [{
    Elementwise Natural Logarithm operation. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_log
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_log

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Log %input1 : !spirv.arm.tensor<45x43x36xf16> -> !spirv.arm.tensor<45x43x36xf16>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1719-1747
````tablegen
def SPIRV_TosaLogicalNotOp : SPIRV_TosaElementwiseUnaryOp<"LogicalNot", 39, [Pure]> {
  let summary = "Logical NOT operator.";

  let description = [{
    Elementwise Logical NOT of input. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_logical_not
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_logical_not

    #### Example:
    ```mlir
    %0 = spirv.Tosa.LogicalNot %input1 : !spirv.arm.tensor<54x26x10xi1> -> !spirv.arm.tensor<54x26x10xi1>
    ```
  }];

  let arguments = (ins
    SPIRV_Bool_TensorArm: $input1
  );

  let results = (outs
    SPIRV_Bool_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaLogicalNotOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaLogicalNotOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1750-1786
````tablegen
def SPIRV_TosaNegateOp : SPIRV_TosaElementwiseUnaryOp<"Negate", 40, [NoMemoryEffect,
  AllElementTypesMatch<["input1", "output", "input1_zp", "output_zp"]>]> {
  let summary = "Negate operator.";

  let description = [{
    Elementwise Negation operation. Input and Output types must match. Integer
    element types are always interpreted as signed. The behaviour is undefined
    if the input value is min_value<si32>.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_negate
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_negate

    #### Example:
    ```mlir
    %2 = spirv.Tosa.Negate %input1, %input1_zp, %output_zp : !spirv.arm.tensor<3x1x65540x1xi8>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<3x1x65540x1xi8>
    %2 = spirv.Tosa.Negate %input1, %input1_zp, %output_zp : !spirv.arm.tensor<2x7x15x13xf16>, !spirv.arm.tensor<1xf16>, !spirv.arm.tensor<1xf16> -> !spirv.arm.tensor<2x7x15x13xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $input1,
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_1DTensorArmOfLength1: $input1_zp,
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_1DTensorArmOfLength1: $output_zp
  );

  let results = (outs
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $input1_zp `,`
    $output_zp
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaNegateOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaNegateOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1789-1805
````tablegen
def SPIRV_TosaReciprocalOp : SPIRV_TosaFloatElementwiseUnaryOp<"Reciprocal", 41> {
  let summary = "Reciprocal operator.";

  let description = [{
    Elementwise Reciprocal operation. Input and Output types must match. For
    integer operation, a table should be used with the appropriate ranges.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reciprocal
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reciprocal

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Reciprocal %input1 : !spirv.arm.tensor<38x47x44xf32> -> !spirv.arm.tensor<38x47x44xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReciprocalOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReciprocalOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1808-1825
````tablegen
def SPIRV_TosaRsqrtOp : SPIRV_TosaFloatElementwiseUnaryOp<"Rsqrt", 42> {
  let summary = "Reverse Square Root operator.";

  let description = [{
    Elementwise Reciprocal Square Root operation ($ 1/sqrt $). Input and
    Output types must match. For integer operation, a table should be used
    with the appropriate ranges.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_rsqrt
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_rsqrt

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Rsqrt %input1 : !spirv.arm.tensor<40x57x56xf32> -> !spirv.arm.tensor<40x57x56xf32>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaRsqrtOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaRsqrtOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1828-1843
````tablegen
def SPIRV_TosaSinOp : SPIRV_TosaFloatElementwiseUnaryOp<"Sin", 43> {
  let summary = "Sin operator.";

  let description = [{
    Elementwise Sine operation for values given in radians. Input and Output types must match.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_sin
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_sin

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Sin %input1 : !spirv.arm.tensor<49x38x58xf16> -> !spirv.arm.tensor<49x38x58xf16>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaSinOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaSinOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1846-1914
````tablegen
def SPIRV_TosaSelectOp : SPIRV_TosaOpWithResult<"Select", 44, [Pure,
  AllElementTypesMatch<["true_value", "false_value", "output"]>,
  AllRanksMatch<["condition", "true_value", "false_value", "output"]>,
  DeclareOpInterfaceMethods<SelectLikeOpInterface>]> {
  let summary = "Select operator.";

  let description = [{
    Elementwise Select of the output based on a condition.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_select
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_select

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Select %cond, %trueVal, %falseVal : !spirv.arm.tensor<4x1x4x5xi1>, !spirv.arm.tensor<4x6x4x5xi8>, !spirv.arm.tensor<4x6x4x5xi8> -> !spirv.arm.tensor<4x6x4x5xi8>
    %0 = spirv.Tosa.Select %cond, %trueVal, %falseVal : !spirv.arm.tensor<9x2x15x8xi1>, !spirv.arm.tensor<9x2x15x8xf16>, !spirv.arm.tensor<9x1x15x8xf16> -> !spirv.arm.tensor<9x2x15x8xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_Bool_TensorArm: $condition,
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $true_value,
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $false_value
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $output
  );

  let hasVerifier = 1;

  let assemblyFormat = [{
    $condition `,`
    $true_value `,`
    $false_value
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getConditionType() {
      return cast<::mlir::spirv::TensorArmType>(getCondition().getType());
    }
    ::mlir::spirv::TensorArmType getTrueValueType() {
      return cast<::mlir::spirv::TensorArmType>(getTrueValue().getType());
    }
    ::mlir::spirv::TensorArmType getFalseValueType() {
      return cast<::mlir::spirv::TensorArmType>(getFalseValue().getType());
    }
    ::mlir::Value getInput1() {
      return getCondition();
    }
    ::mlir::Value getInput2() {
      return getTrueValue();
    }
    ::mlir::Value getInput3() {
      return getFalseValue();
    }
    ::mlir::spirv::TensorArmType getInput1Type() {
      return getConditionType();
    }
    ::mlir::spirv::TensorArmType getInput2Type() {
      return getTrueValueType();
    }
    ::mlir::spirv::TensorArmType getInput3Type() {
      return getFalseValueType();
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaSelectOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaSelectOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 1917-1934
````tablegen
def SPIRV_TosaEqualOp : SPIRV_TosaComparisonOp<"Equal", 45> {
  let summary = "Equal comparison operation";

  let description = [{
    Elementwise Equal comparison operation: returns the truth value of
    (input1 == input2) element-wise.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_equal
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_equal

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Equal %input1, %input2 : !spirv.arm.tensor<51x28x59xi32>, !spirv.arm.tensor<51x1x59xi32> -> !spirv.arm.tensor<51x28x59xi1>
    %0 = spirv.Tosa.Equal %input1, %input2 : !spirv.arm.tensor<16x11x5x3xf32>, !spirv.arm.tensor<16x1x5x3xf32> -> !spirv.arm.tensor<16x11x5x3xi1>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaEqualOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaEqualOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1937-1954
````tablegen
def SPIRV_TosaGreaterOp : SPIRV_TosaComparisonOp<"Greater", 46> {
  let summary = "Greater comparison operation";

  let description = [{
    Elementwise Greater than comparison operation: returns the truth value of
    (input1 > input2) element-wise.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_greater
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_greater

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Greater %input1, %input2 : !spirv.arm.tensor<11x10x10x2xi32>, !spirv.arm.tensor<11x10x10x1xi32> -> !spirv.arm.tensor<11x10x10x2xi1>
    %0 = spirv.Tosa.Greater %input1, %input2 : !spirv.arm.tensor<6x3x12x4xf16>, !spirv.arm.tensor<6x3x1x4xf16> -> !spirv.arm.tensor<6x3x12x4xi1>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaGreaterOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaGreaterOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1957-1974
````tablegen
def SPIRV_TosaGreaterEqualOp : SPIRV_TosaComparisonOp<"GreaterEqual", 47> {
  let summary = "Greater or Equal comparison operation";

  let description = [{
    Elementwise Greater or Equal than comparison operation: returns the truth value of
    (input1 >= input2) element-wise.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_greater_equal
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_greater_equal

    #### Example:
    ```mlir
    %0 = spirv.Tosa.GreaterEqual %input1, %input2 : !spirv.arm.tensor<10x17x7x1xi32>, !spirv.arm.tensor<10x17x7x16xi32> -> !spirv.arm.tensor<10x17x7x16xi1>
    %0 = spirv.Tosa.GreaterEqual %input1, %input2 : !spirv.arm.tensor<3x17x6x3xf32>, !spirv.arm.tensor<1x17x6x3xf32> -> !spirv.arm.tensor<3x17x6x3xi1>
    ```
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaGreaterEqualOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaGreaterEqualOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档。

### Lines 1977-2007
````tablegen
def SPIRV_TosaReduceAllOp : SPIRV_TosaReductionOp<"ReduceAll", 48, [Pure]> {
  let summary = "Reduce All operator.";

  let description = [{
    Reduces a tensor along the given axis with a Logical AND operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reduce_all
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reduce_all

    #### Example:
    ```mlir
    %1 = spirv.Tosa.ReduceAll axis = 2, %input1 : !spirv.arm.tensor<18x22x23x12xi1> -> !spirv.arm.tensor<18x22x1x12xi1>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_Bool_TensorArm: $input
  );

  let results = (outs
    SPIRV_Bool_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReduceAllOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReduceAllOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2010-2040
````tablegen
def SPIRV_TosaReduceAnyOp : SPIRV_TosaReductionOp<"ReduceAny", 49, [Pure]> {
  let summary = "Reduce Any operator.";

  let description = [{
    Reduces a tensor along the given axis with a Logical OR operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reduce_any
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reduce_any

    #### Example:
    ```mlir
    %1 = spirv.Tosa.ReduceAny axis = 2, %input1 : !spirv.arm.tensor<25x13x30x8xi1> -> !spirv.arm.tensor<25x13x1x8xi1>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_Bool_TensorArm: $input
  );

  let results = (outs
    SPIRV_Bool_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReduceAnyOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReduceAnyOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2043-2077
````tablegen
def SPIRV_TosaReduceMaxOp : SPIRV_TosaReductionOp<"ReduceMax", 50, [Pure]> {
  let summary = "Reduce Max operator.";

  let description = [{
    Reduces a tensor along the given axis with a Maximum operation.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reduce_max
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reduce_max

    #### Example:
    ```mlir
    %2 = spirv.Tosa.ReduceMax axis = 2, nan_mode = <Propagate>, %input1 : !spirv.arm.tensor<8x30x12x3xi8> -> !spirv.arm.tensor<8x30x1x3xi8>
    %2 = spirv.Tosa.ReduceMax axis = 2, nan_mode = <Propagate>, %input1 : !spirv.arm.tensor<16x20x10xf16> -> !spirv.arm.tensor<16x20x1xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    `nan_mode` `=` $nan_mode `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReduceMaxOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReduceMaxOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2080-2114
````tablegen
def SPIRV_TosaReduceMinOp : SPIRV_TosaReductionOp<"ReduceMin", 51, [Pure]> {
  let summary = "Reduce Min operator.";

  let description = [{
    Reduces a tensor along the given axis with a Minimum operation.
    NaN Propagation Mode is ignored for inputs using integer element types.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reduce_min
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reduce_min

    #### Example:
    ```mlir
    %2 = spirv.Tosa.ReduceMin axis = 2, nan_mode = <Propagate>, %input1 : !spirv.arm.tensor<2x5x5x1xi8> -> !spirv.arm.tensor<2x5x1x1xi8>
    %2 = spirv.Tosa.ReduceMin axis = 2, nan_mode = <Propagate>, %input1 : !spirv.arm.tensor<27x10x25x9xf16> -> !spirv.arm.tensor<27x10x1x9xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_TosaExtNaNPropagationModeAttr: $nan_mode,
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    `nan_mode` `=` $nan_mode `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReduceMinOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReduceMinOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2117-2147
````tablegen
def SPIRV_TosaReduceProductOp : SPIRV_TosaReductionOp<"ReduceProduct", 52, [Pure]> {
  let summary = "Reduce Product operator.";

  let description = [{
    Reduces a tensor along the given axis by computing the Product of the axis.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reduce_product
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reduce_product

    #### Example:
    ```mlir
    %1 = spirv.Tosa.ReduceProduct axis = 2, %input1 : !spirv.arm.tensor<2x16x25xf16> -> !spirv.arm.tensor<2x16x1xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_F16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_F16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReduceProductOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReduceProductOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2150-2184
````tablegen
def SPIRV_TosaReduceSumOp : SPIRV_TosaReductionOp<"ReduceSum", 53, [NoMemoryEffect]> {
  let summary = "Reduce Sum operator.";

  let description = [{
    Reduces a tensor along the given axis by computing the Sum of the axis.
    Integer element types are always interpreted as signed. The behavior is
    undefined if the integer addition overflows or underflows the signed
    integer range.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reduce_sum
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reduce_sum

    #### Example:
    ```mlir
    %1 = spirv.Tosa.ReduceSum axis = 1, %input1 : !spirv.arm.tensor<20x24x22xi32> -> !spirv.arm.tensor<20x1x22xi32>
    %1 = spirv.Tosa.ReduceSum axis = 1, %input1 : !spirv.arm.tensor<32x32x33xf32> -> !spirv.arm.tensor<32x1x33xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $input
  );

  let results = (outs
    SPIRV_I32OrF16OrF32OrBF16_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReduceSumOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReduceSumOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2187-2231
````tablegen
def SPIRV_TosaConcatOp : SPIRV_TosaOpWithResult<"Concat", 54, [Pure,
  VariadicInputWithMinSize<"input1", 1>,
  VariadicInputAllSameElementType<"output", "input1">,
  VariadicInputAllSameRank<"output", "input1">,
  AxisValueLessThanRankOf<"output">]> {
  let summary = "Concatenates tensors along one dimension.";

  let description = [{
    Concatenates a list of tensors along a given axis.
    No data conversion happens during a concat operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_concat
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_concat

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Concat axis = 2, %input1, %input2, %input3, %input4 : !spirv.arm.tensor<12x13x3x14xi8>, !spirv.arm.tensor<12x13x3x14xi8>, !spirv.arm.tensor<12x13x3x14xi8>, !spirv.arm.tensor<12x13x3x14xi8> -> !spirv.arm.tensor<12x13x12x14xi8>
    %1 = spirv.Tosa.Concat axis = 1, %input1, %input2, %input3 : !spirv.arm.tensor<40x31x19xf32>, !spirv.arm.tensor<40x15x19xf32>, !spirv.arm.tensor<40x16x19xf32> -> !spirv.arm.tensor<40x62x19xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    Variadic<SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm>: $input1
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::TypeRange getInput1Types() {
      return getInput1().getTypes();
    }
  }];

  let hasVerifier = 1;
}
````
- **EN**: This TableGen block defines `SPIRV_TosaConcatOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaConcatOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2234-2287
````tablegen
def SPIRV_TosaPadOp : SPIRV_TosaOpWithResult<"Pad", 55, [Pure,
  AllElementTypesMatch<["input1", "pad_const", "output"]>,
  AllRanksMatch<["input1", "output"]>,
  ShapeConstraintFromInputRank<"input1", "padding", 2>]> {
  let summary = "Pads a tensor with value specified.";

  let description = [{
    Pads a tensor along the borders of each dimension with a supplied value.
    Returns a new tensor with the padding included. The pad_const value includes
    the zero point if the tensor uses a zero point.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_pad
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_pad

    #### Example:
    ```mlir
    %2 = spirv.Tosa.Pad %input1, %padding, %pad_const : !spirv.arm.tensor<4x7xi8>, !spirv.arm.tensor<4xi32>, !spirv.arm.tensor<1xi8> -> !spirv.arm.tensor<21x19xi8>
    %2 = spirv.Tosa.Pad %input1, %padding, %pad_const : !spirv.arm.tensor<2x9x2x3xf32>, !spirv.arm.tensor<8xi32>, !spirv.arm.tensor<1xf32> -> !spirv.arm.tensor<4x9x4x4xf32>
    ```

    For the first example, `%padding = [8, 9, 6, 6]` pads dimension 0 by 8
    before and 9 after, and dimension 1 by 6 on both sides.
  }];

  let arguments = (ins
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input1,
    SPIRV_I32_1DTensorArmOfEvenLength2To12: $padding,
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1: $pad_const
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $padding `,`
    $pad_const
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
    ::mlir::spirv::TensorArmType getPaddingType() {
      return cast<::mlir::spirv::TensorArmType>(getPadding().getType());
    }
    ::mlir::spirv::TensorArmType getPadConstType() {
      return cast<::mlir::spirv::TensorArmType>(getPadConst().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaPadOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaPadOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2290-2335
````tablegen
def SPIRV_TosaReshapeOp : SPIRV_TosaOpWithResult<"Reshape", 56, [Pure,
  AllElementTypesMatch<["input1", "output"]>,
  AllElementCountsMatch<["input1", "output"]>,
  ShapeConstraintFromInputRank<"output", "shape">]> {
  let summary = "Reshape operator.";

  let description = [{
    Returns a tensor with the same type/values as the input, with a new shape
    specified by the shape argument. Reshape may operate on tensors of any rank.
    No data conversion happens during a reshape operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reshape
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reshape

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Reshape %input1, %shape : !spirv.arm.tensor<25x6x29x35xi16>, !spirv.arm.tensor<4xi32> -> !spirv.arm.tensor<125x6x7x29xi16>
    %1 = spirv.Tosa.Reshape %input1, %shape : !spirv.arm.tensor<1x2x7x2xf32>, !spirv.arm.tensor<3xi32> -> !spirv.arm.tensor<2x1x14xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input1,
    SPIRV_I32_1DTensorArmOfLength1To6: $shape
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $shape
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
    ::mlir::spirv::TensorArmType getShapeType() {
      return cast<::mlir::spirv::TensorArmType>(getShape().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReshapeOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReshapeOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2338-2379
````tablegen
def SPIRV_TosaReverseOp : SPIRV_TosaOpWithResult<"Reverse", 57, [Pure,
  AllTypesMatch<["input1", "output"]>,
  AxisValueLessThanRankOf<"input1">]> {
  let summary = "Reverse operator.";

  let description = [{
    Returns a tensor with the same type/values as the input, with the data
    reversed along the given axis. No data conversion happens during a reverse
    operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_reverse
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_reverse

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Reverse axis = 2, %input1 : !spirv.arm.tensor<20x5x28x31xi32> -> !spirv.arm.tensor<20x5x28x31xi32>
    %1 = spirv.Tosa.Reverse axis = 1, %input1 : !spirv.arm.tensor<21x34x47xf32> -> !spirv.arm.tensor<21x34x47xf32>
    ```
  }];

  let arguments = (ins
    SPIRV_TensorArmAxisAttr: $axis,
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input1
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    `axis` `=` $axis `,`
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaReverseOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaReverseOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2382-2435
````tablegen
def SPIRV_TosaSliceOp : SPIRV_TosaOpWithResult<"Slice", 58, [Pure,
  AllElementTypesMatch<["input1", "output"]>,
  ShapeConstraintFromInputRank<"input1", "start">,
  ShapeConstraintFromInputRank<"input1", "size">]> {
  let summary = "Slice operator.";

  let description = [{
    Extracts a slice of input1, beginning at the start coordinates,
    and extending for size elements in each direction.
    No data conversion happens during a slice operation.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_slice
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_slice

    #### Example:
    ```mlir
    %2 = spirv.Tosa.Slice %input1, %start, %size : !spirv.arm.tensor<32x19x41xi8>, !spirv.arm.tensor<3xi32>, !spirv.arm.tensor<3xi32> -> !spirv.arm.tensor<21x5x2xi8>
    %2 = spirv.Tosa.Slice %input1, %start, %size : !spirv.arm.tensor<30x45x29xf32>, !spirv.arm.tensor<3xi32>, !spirv.arm.tensor<3xi32> -> !spirv.arm.tensor<5x12x11xf32>
    ```

    For the first example, `%start = [3, 10, 20]` and `%size = [21, 5, 2]`
    extract `%input1[3:24, 10:15, 20:22]`.
  }];

  let arguments = (ins
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input1,
    SPIRV_I32_1DTensorArmOfLength1To6: $start,
    SPIRV_I32_1DTensorArmOfLength1To6: $size
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $start `,`
    $size
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
    ::mlir::spirv::TensorArmType getStartType() {
      return cast<::mlir::spirv::TensorArmType>(getStart().getType());
    }
    ::mlir::spirv::TensorArmType getSizeType() {
      return cast<::mlir::spirv::TensorArmType>(getSize().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaSliceOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaSliceOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2438-2484
````tablegen
def SPIRV_TosaTileOp : SPIRV_TosaOpWithResult<"Tile", 59, [Pure,
  AllElementTypesMatch<["input1", "output"]>,
  AllRanksMatch<["input1", "output"]>,
  ShapeConstraintFromInputRank<"input1", "multiples">]> {
  let summary = "Tile operator.";

  let description = [{
    Replicates input1 multiples times along each dimension.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_tile
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_tile

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Tile %input1, %multiples : !spirv.arm.tensor<10x28x21xi16>, !spirv.arm.tensor<3xi32> -> !spirv.arm.tensor<10x28x63xi16>
    %1 = spirv.Tosa.Tile %input1, %multiples : !spirv.arm.tensor<31x19x5xf16>, !spirv.arm.tensor<3xi32> -> !spirv.arm.tensor<62x57x10xf16>
    ```

    For `%multiples = [1, 1, 3]`, the first two dimensions will be tiled once,
    while the third will be tiled 3 times.
  }];

  let arguments = (ins
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input1,
    SPIRV_I32_1DTensorArmOfLength1To6: $multiples
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    $input1 `,`
    $multiples
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
    ::mlir::spirv::TensorArmType getMultiplesType() {
      return cast<::mlir::spirv::TensorArmType>(getMultiples().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaTileOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaTileOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2487-2530
````tablegen
def SPIRV_TosaTransposeOp : SPIRV_TosaOpWithResult<"Transpose", 60, [Pure,
  AllElementTypesMatch<["input1", "output"]>,
  AllRanksMatch<["input1", "output"]>,
  AllElementCountsMatch<["input1", "output"]>,
  ShapeConstraintFromInputRank<"input1", "perms">]> {
  let summary = "Transpose operator.";

  let description = [{
    Permutes the dimensions of the input tensor input1 based on the perms
    argument. Each value in the perms list must be a valid dimension of the
    input tensor and may not be repeated.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_transpose
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_transpose

    #### Example:
    ```mlir
    %1 = spirv.Tosa.Transpose perms = [2, 0, 1, 3], %input1 : !spirv.arm.tensor<14x28x1x61xi16> -> !spirv.arm.tensor<1x14x28x61xi16>
    %1 = spirv.Tosa.Transpose perms = [2, 0, 1], %input1 : !spirv.arm.tensor<42x22x49xi1> -> !spirv.arm.tensor<49x42x22xi1>
    ```
  }];

  let arguments = (ins
    SPIRV_I32_1DTensorArmOfLength1To6Attr: $perms,
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input1
  );

  let results = (outs
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    `perms` `=` custom<SPIRV_I32_1DArmTensor>($perms) `,`
    $input1
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInput1Type() {
      return cast<::mlir::spirv::TensorArmType>(getInput1().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaTransposeOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaTransposeOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2533-2580
````tablegen
def SPIRV_TosaGatherOp : SPIRV_TosaOpWithResult<"Gather", 61, [NoMemoryEffect,
  AllElementTypesMatch<["values", "output"]>,
  ValuesIndicesShapesMatch<"values", "indices", "output">]> {
  let summary = "Gather operation.";

  let description = [{
    Generate a tensor for which each element in the output is a subtensor of the
    values tensor based on the indices. Undefined behaviour may occur if the
    specified indices are out of range.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_gather
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_gather

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Gather %values, %indices : !spirv.arm.tensor<31x11x45xi32>, !spirv.arm.tensor<31x15xi32> -> !spirv.arm.tensor<31x15x45xi32>
    %0 = spirv.Tosa.Gather %values, %indices : !spirv.arm.tensor<59x61x19xf32>, !spirv.arm.tensor<59x65xi32> -> !spirv.arm.tensor<59x65x19xf32>
    ```

    If a row of `%indices` is `[4, 4, 0]`, the corresponding output row copies
    slices 4, 4, and 0 from the matching batch of `%values`.
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D: $values,
    SPIRV_I32_TensorArm2D: $indices
  );

  let results = (outs
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D: $output
  );

  let assemblyFormat = [{
    $values `,`
    $indices
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getValuesType() {
      return cast<::mlir::spirv::TensorArmType>(getValues().getType());
    }
    ::mlir::spirv::TensorArmType getIndicesType() {
      return cast<::mlir::spirv::TensorArmType>(getIndices().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaGatherOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaGatherOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2583-2639
````tablegen
def SPIRV_TosaScatterOp : SPIRV_TosaOpWithResult<"Scatter", 62, [NoMemoryEffect,
  AllElementTypesMatch<["values_in", "input", "values_out"]>,
  AllTypesMatch<["values_in", "values_out"]>,
  ValuesIndicesShapesMatch<"values_in", "indices", "input">]> {
  let summary = "Scatter operation.";

  let description = [{
    The values_out tensor is set to the values_in tensor with data modified as
    follows: data from the input tensor is inserted at the positions specified
    by the indices tensor. In use cases that require multiple updates to the
    same output position, these must be decomposed into multiple scatter
    operations.  Undefined behaviour may occur if the specified indices are
    out of range or duplicate indices are provided.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_scatter
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_scatter

    #### Example:
    ```mlir
    %0 = spirv.Tosa.Scatter %values_in, %indices, %input1 : !spirv.arm.tensor<34x28x54xi32>, !spirv.arm.tensor<34x18xi32>, !spirv.arm.tensor<34x18x54xi32> -> !spirv.arm.tensor<34x28x54xi32>
    %0 = spirv.Tosa.Scatter %values_in, %indices, %input1 : !spirv.arm.tensor<18x34x25xf16>, !spirv.arm.tensor<18x20xi32>, !spirv.arm.tensor<18x20x25xf16> -> !spirv.arm.tensor<18x34x25xf16>
    ```

    If a row of `%indices` is `[4, 9]`, the two subtensors from `%input1` are
    written into slices 4 and 9 of `%values_in` for that batch.
  }];

  let arguments = (ins
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D: $values_in,
    SPIRV_I32_TensorArm2D: $indices,
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D: $input
  );

  let results = (outs
    SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D: $values_out
  );

  let assemblyFormat = [{
    $values_in `,`
    $indices `,`
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getValuesInType() {
      return cast<::mlir::spirv::TensorArmType>(getValuesIn().getType());
    }
    ::mlir::spirv::TensorArmType getIndicesType() {
      return cast<::mlir::spirv::TensorArmType>(getIndices().getType());
    }
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This TableGen block defines `SPIRV_TosaScatterOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaScatterOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2642-2713
````tablegen
def SPIRV_TosaResizeOp : SPIRV_TosaOpWithResult<"Resize", 63, [Pure,
  TypeConstraintImplicationOn<"input", F16, "output", [F16]>,
  TypeConstraintImplicationOn<"input", F32, "output", [F32]>,
  TypeConstraintImplicationOn<"input", BF16, "output", [BF16]>,
  TypeConstraintImplicationOn<"input", I8, "output", [I8, I32]>,
  TypeConstraintImplicationOn<"input", I16, "output", [I16, I64]>,
  NHWCInputOutputShapeMatch<"input", "output">]> {
  let summary = "Resize operation, supports various resize/upsample modes.";

  let description = [{
    Resizes a tensor. Resize is only allowed in the H and W dimensions, given the input
    shape = [N,H,W,C].

    The height dimension (H) is scaled by factor ($ scale_y_n/scale_y_d $). The width
    dimension (W) is scaled by factor ($ scale_x_n/scale_x_d $).

    The NearestNeighbor mode returns the value of the input tensor closest to
    the calculated sample position for both floating-point and integer data
    formats.

    Floating-point Bilinear mode returns a bilinearly interpolated output value
    based on the four closest input sample positions.

    For integer Bilinear interpolation mode, the output value must be scaled by
    $ 1/(scale_y_n * scale_x_n) $ in a following operation to complete the
    interpolation (for example with a rescale operator).

    The output dimensions can be derived from the input dimensions by inverting
    the scale. The [border_y, border_x] values adjust the output size to allow
    fractional sampling beyond integer input position (H - 1,W - 1).

    The limit MAX_SCALE=256 is applied to each scale ratio after reduction of the
    ratio. Individual scale numerator and denominator values are allowed to be
    larger than MAX_SCALE.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_resize
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_resize

    #### Example:
    ```mlir
    %4 = spirv.Tosa.Resize mode = <NearestNeighbor>, %input, %scale, %offset, %border : !spirv.arm.tensor<1x1x31x55xi8>, !spirv.arm.tensor<4xi32>, !spirv.arm.tensor<2xi32>, !spirv.arm.tensor<2xi32> -> !spirv.arm.tensor<1x1x278x55xi8>
    %4 = spirv.Tosa.Resize mode = <Bilinear>, %input, %scale, %offset, %border : !spirv.arm.tensor<1x48x33x63xf32>, !spirv.arm.tensor<4xi32>, !spirv.arm.tensor<2xi32>, !spirv.arm.tensor<2xi32> -> !spirv.arm.tensor<1x753x297x63xf32>
    ```

    The resize tensors are ordered as
    `%scale = [scale_y_n, scale_y_d, scale_x_n, scale_x_d]`,
    `%offset = [offset_y, offset_x]`, and `%border = [border_y, border_x]`;
    for example, `%scale = [2, 1, 3, 2]` means "double height and scale width
    by 3/2".
  }];

  let arguments = (ins
    SPIRV_TosaExtResizeModeAttr: $mode,
    SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm4D: $input,
    SPIRV_I32_1DTensorArmOfLength4: $scale,
    SPIRV_I32_1DTensorArmOfLength2: $offset,
    SPIRV_I32_1DTensorArmOfLength2: $border
  );

  let results = (outs
    SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16_TensorArm4D: $output
  );

  let assemblyFormat = [{
    `mode` `=` $mode `,`
    $input `,`
    $scale `,`
    $offset `,`
    $border
    attr-dict `:` type(operands) `->` type(results)
  }];
````
- **EN**: This TableGen block defines `SPIRV_TosaResizeOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaResizeOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2714-2728
````tablegen
  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
    ::mlir::spirv::TensorArmType getScaleType() {
      return cast<::mlir::spirv::TensorArmType>(getScale().getType());
    }
    ::mlir::spirv::TensorArmType getOffsetType() {
      return cast<::mlir::spirv::TensorArmType>(getOffset().getType());
    }
    ::mlir::spirv::TensorArmType getBorderType() {
      return cast<::mlir::spirv::TensorArmType>(getBorder().getType());
    }
  }];
}
````
- **EN**: This block groups callable interfaces such as `getInputType`, `getInput`, `getType`, `getScaleType`, indicating how `SPIRVTosaOps` is queried or updated.
- **CN**: 该代码块聚合了 `getInputType`, `getInput`, `getType`, `getScaleType` 等可调用接口，展示了如何查询或更新 `SPIRVTosaOps`。

### Lines 2732-2801
````tablegen
def SPIRV_TosaCastOp : SPIRV_TosaOpWithResult<"Cast", 64, [Pure,
  AllShapesMatch<["input", "output"]>,
  TypeConstraintImplicationOn<"input", F16, "output", [F32, I16, I32, I8, F8E4M3FN, F8E5M2]>,
  TypeConstraintImplicationOn<"input", F32, "output", [F16, I16, I32, I8, BF16, F8E4M3FN, F8E5M2]>,
  TypeConstraintImplicationOn<"input", I16, "output", [F16, F32, I32, I8, SPIRV_Bool, BF16]>,
  TypeConstraintImplicationOn<"input", I32, "output", [F16, F32, I16, I8, SPIRV_Bool, BF16]>,
  TypeConstraintImplicationOn<"input", I8, "output", [F16, F32, I16, I32, SPIRV_Bool, BF16]>,
  TypeConstraintImplicationOn<"input", SPIRV_Bool, "output", [I16, I32, I8]>,
  TypeConstraintImplicationOn<"input", BF16, "output", [F32, I16, I32, I8, F8E4M3FN, F8E5M2]>,
  TypeConstraintImplicationOn<"input", F8E4M3FN, "output", [F16, F32, BF16]>,
  TypeConstraintImplicationOn<"input", F8E5M2, "output", [F16, F32, BF16]>]> {
  let summary = "Cast operation.";

  let description = [{
    Casts a tensor from one data type to another.

    Valid casting combinations are defined in the following table:

    | From    | To      |
    |---------|---------|
    | float16 | float32 |
    | float16 | int16   |
    | float16 | int32   |
    | float16 | int8    |
    | float32 | float16 |
    | float32 | int16   |
    | float32 | int32   |
    | float32 | int8    |
    | int16   | float16 |
    | int16   | float32 |
    | int32   | float16 |
    | int32   | float32 |
    | int8    | float16 |
    | int8    | float32 |
    | Boolean | int16   |
    | Boolean | int32   |
    | Boolean | int8    |
    | int16   | Boolean |
    | int16   | int32   |
    | int16   | int8    |
    | int32   | Boolean |
    | int32   | int16   |
    | int32   | int8    |
    | int8    | Boolean |
    | int8    | int16   |
    | int8    | int32   |
    | bf16    | float32 |
    | bf16    | int16   |
    | bf16    | int32   |
    | bf16    | int8    |
    | float32 | bf16    |
    | int16   | bf16    |
    | int32   | bf16    |
    | int8    | bf16    |
    | bf16    | fp8e4m3 |
    | fp8e4m3 | bf16    |
    | bf16    | fp8e5m2 |
    | fp8e5m2 | bf16    |
    | float16 | fp8e4m3 |
    | float32 | fp8e4m3 |
    | fp8e4m3 | float16 |
    | fp8e4m3 | float32 |
    | float16 | fp8e5m2 |
    | float32 | fp8e5m2 |
    | fp8e5m2 | float16 |
    | fp8e5m2 | float32 |

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_cast
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_cast
````
- **EN**: This TableGen block defines `SPIRV_TosaCastOp` as a `def` record for `SPIRVTosaOps`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaCastOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 2802-2827
````tablegen
    #### Example:
    ```mlir
    %0 = spirv.Tosa.Cast %input : !spirv.arm.tensor<1x65538x1x2xi8> -> !spirv.arm.tensor<1x65538x1x2xi32>
    %0 = spirv.Tosa.Cast %input : !spirv.arm.tensor<11x5x14x4xf32> -> !spirv.arm.tensor<11x5x14x4xf16>
    ```
  }];

  let arguments = (ins
    SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm: $input
  );

  let results = (outs
    SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8_TensorArm: $output
  );

  let assemblyFormat = [{
    $input
    attr-dict `:` type(operands) `->` type(results)
  }];

  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
  }];
}
````
- **EN**: This block groups callable interfaces such as `type`, `getInputType`, `getInput`, `getType`, indicating how `SPIRVTosaOps` is queried or updated.
- **CN**: 该代码块聚合了 `type`, `getInputType`, `getInput`, `getType` 等可调用接口，展示了如何查询或更新 `SPIRVTosaOps`。

### Lines 2831-2912
````tablegen
def SPIRV_TosaRescaleOp : SPIRV_TosaOpWithResult<"Rescale", 65, [NoMemoryEffect,
  AllShapesMatch<["input", "output"]>,
  AllElementTypesMatch<["input", "input_zp"]>,
  AllElementTypesMatch<["output", "output_zp"]>,
  ElementTypeMatchesScale32<"multiplier">,
  TensorLengthMatchesPerChannel<"multiplier">,
  TensorLengthMatchesPerChannel<"shift">,
  BoolAttrTypeConstraintImplicationOn<"input_unsigned", "input", [I8, I16]>,
  BoolAttrTypeConstraintImplicationOn<"output_unsigned", "input", [I8, I16]>,
  BoolAttrTypeConstraintImplicationOn<"input_unsigned", "output", [I8, I16]>,
  BoolAttrTypeConstraintImplicationOn<"output_unsigned", "output", [I8, I16]>]> {
  let summary = "Rescale operator.";

  let description = [{
    Rescale is defined using an integer multiply, add, and shift.

    Rescale supports two precisions of multiplier: 16-bit and 32-bit. The
    32-bit multiplier version supports two rounding modes to enable simpler
    lowering of existing frameworks that use two stage rounding. All arithmetic
    is designed so that it does not overflow a 64-bit accumulator and that the
    result fits in 32 bits. In particular, a 48-bit value (represented as a
    64-bit value in SPIR-V) cannot be scaled with the 32-bit multiplier because
    the accumulator would need to have 80 bits.

    The shift and value range are limited to allow a variety of implementations.
    The limit of 62 on shift allows the shift to be decomposed as two right
    shifts of 31.

    Unsigned 8- and 16-bit values are only allowed in the Rescale operation,
    to allow for compatibility with networks which expect unsigned 8-bit or
    16-bit tensors for input and output.

    Undefined behaviour may occur if the calculated result underflows or overflows
    their integer ranges.

    References:
      * https://github.khronos.org/SPIRV-Registry/extended/TOSA.001000.1.html#_rescale
      * https://www.mlplatform.org/tosa/tosa_spec_1_0_1.html#_rescale

    #### Example:
    ```mlir
    %9 = spirv.Tosa.Rescale scale32 = true, rounding_mode = <DoubleRound>, per_channel = false, input_unsigned = false, output_unsigned = true, %input, %multiplier, %shift, %input_zp, %output_zp : !spirv.arm.tensor<17x29x19xi16>, !spirv.arm.tensor<1xi32>, !spirv.arm.tensor<1xi8>, !spirv.arm.tensor<1xi16>, !spirv.arm.tensor<1xi16> -> !spirv.arm.tensor<17x29x19xi16>
    ```

    With `per_channel = false`, `%multiplier` and `%shift` are 1D tensors with
    a single value, so they define one global scale for the whole tensor, while
    `%input_zp` and `%output_zp` are scalar zero points. For example,
    `%multiplier = [1073741824]`, `%shift = [30]`, `%input_zp = [0]`, and
    `%output_zp = [128]` mean "apply one global rescale to every element, then
    store the result using output zero point 128".
  }];

  let arguments = (ins
    SPIRV_BoolConstAttr: $scale32,
    SPIRV_TosaExtRoundingModeAttr: $rounding_mode,
    SPIRV_BoolConstAttr: $per_channel,
    SPIRV_BoolConstAttr: $input_unsigned,
    SPIRV_BoolConstAttr: $output_unsigned,
    SPIRV_I8OrI16OrI32OrI64_TensorArm: $input,
    SPIRV_I16OrI32_TensorArm1D: $multiplier,
    SPIRV_I8_TensorArm1D: $shift,
    SPIRV_I8OrI16OrI32OrI64_1DTensorArmOfLength1: $input_zp,
    SPIRV_I8OrI16OrI32_1DTensorArmOfLength1: $output_zp
  );

  let results = (outs
    SPIRV_I8OrI16OrI32_TensorArm: $output
  );

  let assemblyFormat = [{
    `scale32` `=` $scale32 `,`
    `rounding_mode` `=` $rounding_mode `,`
    `per_channel` `=` $per_channel `,`
    `input_unsigned` `=` $input_unsigned `,`
    `output_unsigned` `=` $output_unsigned `,`
    $input `,`
    $multiplier `,`
    $shift `,`
    $input_zp `,`
    $output_zp
    attr-dict `:` type(operands) `->` type(results)
  }];
````
- **EN**: This TableGen block defines `SPIRV_TosaRescaleOp` as a `def` record for `SPIRVTosaOps`. It covers assembly syntax, operand or attribute schema, result typing, semantic documentation.
- **CN**: 该 TableGen 代码块将 `SPIRV_TosaRescaleOp` 定义为 `def` 记录，用于描述 `SPIRVTosaOps` 相关的声明式信息。 其中涉及 汇编语法, 操作数或属性模式, 结果类型约束, 语义文档。

### Lines 2913-2924
````tablegen
  let extraClassDeclaration = extraBaseClassDeclaration#[{
    ::mlir::spirv::TensorArmType getInputType() {
      return cast<::mlir::spirv::TensorArmType>(getInput().getType());
    }
    ::mlir::spirv::TensorArmType getMultiplierType() {
      return cast<::mlir::spirv::TensorArmType>(getMultiplier().getType());
    }
    ::mlir::spirv::TensorArmType getShiftType() {
      return cast<::mlir::spirv::TensorArmType>(getShift().getType());
    }
  }];
}
````
- **EN**: This block groups callable interfaces such as `getInputType`, `getInput`, `getType`, `getMultiplierType`, indicating how `SPIRVTosaOps` is queried or updated.
- **CN**: 该代码块聚合了 `getInputType`, `getInput`, `getType`, `getMultiplierType` 等可调用接口，展示了如何查询或更新 `SPIRVTosaOps`。

### Lines 2926-2926
````tablegen
#endif // MLIR_DIALECT_SPIRV_IR_TOSA_OPS
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

- mlir/Dialect/SPIRV/IR/SPIRVBase.td
- mlir/Dialect/SPIRV/IR/SPIRVGraphOps.td
- mlir/Dialect/SPIRV/IR/SPIRVTosaTypes.td
- mlir/Interfaces/SideEffectInterfaces.td
- SPIRV_TosaArgMaxOp builds on SPIRV_TosaOpWithResult<"ArgMax", 0, [Pure,
- SPIRV_TosaAvgPool2DOp builds on SPIRV_TosaOpWithResult<"AvgPool2D", 1, [NoMemoryEffect,
- SPIRV_TosaConv2DOp builds on SPIRV_TosaConvolutionOp<"Conv2D", 2, [
- SPIRV_TosaConv3DOp builds on SPIRV_TosaConvolutionOp<"Conv3D", 3, [
- SPIRV_TosaDepthwiseConv2DOp builds on SPIRV_TosaConvolutionOp<"DepthwiseConv2D", 4, [
- SPIRV_TosaFFT2DOp builds on SPIRV_TosaOpWithComplexResult<"FFT2D", 5, [Pure,
- SPIRV_TosaMatMulOp builds on SPIRV_TosaOpWithResult<"MatMul", 6, [NoMemoryEffect,
- SPIRV_TosaMaxPool2DOp builds on SPIRV_TosaOpWithResult<"MaxPool2D", 7, [Pure,
- SPIRV_TosaRFFT2DOp builds on SPIRV_TosaOpWithComplexResult<"RFFT2D", 8, [Pure,
- SPIRV_TosaTransposeConv2DOp builds on SPIRV_TosaConvolutionOp<"TransposeConv2D", 9, [
- SPIRV_TosaClampOp builds on SPIRV_TosaOpWithResult<"Clamp", 10, [Pure,
- SPIRV_TosaErfOp builds on SPIRV_TosaOpWithResult<"Erf", 11, [Pure,
- SPIRV_TosaSigmoidOp builds on SPIRV_TosaOpWithResult<"Sigmoid", 12, [Pure,
- SPIRV_TosaTanhOp builds on SPIRV_TosaOpWithResult<"Tanh", 13, [Pure,
- SPIRV_TosaAddOp builds on SPIRV_TosaElementwiseBinaryOp<"Add", 14, [NoMemoryEffect]>
- SPIRV_TosaArithmeticRightShiftOp builds on SPIRV_TosaElementwiseBinaryOp<"ArithmeticRightShift", 15, [NoMemoryEffect]>
