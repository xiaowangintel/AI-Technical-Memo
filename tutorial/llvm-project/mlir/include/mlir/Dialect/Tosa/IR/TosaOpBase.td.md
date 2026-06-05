# TosaOpBase.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/IR/TosaOpBase.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TosaOpBase component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file defines the common definitions for the TOSA dialect.
- **用途（CN）**: 为 MLIR 的 TosaOpBase 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
````tablegen
//===-- TosaOpBase.td - TOSA dialect op builders -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the common definitions for the TOSA dialect.
//
//===----------------------------------------------------------------------===//

#ifndef TOSA_OP_BASE
#define TOSA_OP_BASE

include "mlir/IR/AttrTypeBase.td"
include "mlir/IR/EnumAttr.td"
include "mlir/IR/OpBase.td"

include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/LoopLikeInterface.td"

include "mlir/Dialect/Tosa/IR/TosaInterfaces.td"

//===----------------------------------------------------------------------===//
// The TOSA Dialect.
//===----------------------------------------------------------------------===//
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 30-56
````tablegen
def Tosa_Dialect : Dialect {
  let name = "tosa";

  let description = [{
    The Tensor Operator Set Architecture (TOSA) dialect.

    This dialect implements the TOSA standard described at
    https://www.mlplatform.org/tosa/tosa_spec.html .

    Tensor Operator Set Architecture (TOSA) provides a set of whole-tensor
    operations commonly employed by Deep Neural Networks. The intent is to
    enable a variety of implementations running on a diverse range of
    processors, with the results at the TOSA level consistent across those
    implementations. Applications or frameworks which target TOSA can therefore
    be deployed on a wide range of different processors, such as CPUs or GPUs,
    with defined accuracy and compatibility constraints. Most operators from the
    common ML frameworks should be expressible in TOSA. It is expected that
    there will be tools to lower from the ML frameworks into TOSA.
  }];

  let dependentDialects = ["tensor::TensorDialect", "quant::QuantDialect"];

  let cppNamespace = "mlir::tosa";
  let hasConstantMaterializer = 1;
  let useDefaultAttributePrinterParser = 1;
  let useDefaultTypePrinterParser = 1;
}
````
- **EN**: This TableGen block defines `Tosa_Dialect` as a `def` record for `TosaOpBase`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_Dialect` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 语义文档。

### Lines 57-59
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Attributes.
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 62-65
````tablegen
class Tosa_Attr<string attrName, string attrMnemonic, list<Trait> traits = []>
    : AttrDef<Tosa_Dialect, attrName, traits> {
  let mnemonic = attrMnemonic;
}
````
- **EN**: This TableGen block defines `Tosa_Attr` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_Attr` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 66-82
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Operator Quantization Attributes.
//===----------------------------------------------------------------------===//

// Quantization attributes used across TOSA operators. Quantization attributes
// feed numerical precision parameters to the functional implementation of TOSA
// operators.
// The functional behavior is defined in the TOSA specification maintained at
// https://www.mlplatform.org/tosa/tosa_spec.html. TOSA leverages MLIR's built in
// quantization support: https://mlir.llvm.org/docs/Quantization/, and supports
// uniform quantization. Depending on datatype, asymmetric and symmetric
// quantization are supported. The types themselves are described in
// TosaTypesBase.td .

// This quantization attribute expresses numerical behavior of operators where
// the operator has a numerical relationship between a single input and output.
// For example: tosa.negate.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 84-89
````tablegen
def Tosa_UnaryOpQuantizationAttr
    : Tosa_Attr<"UnaryOpQuantization", "unary_quant"> {
  let summary = "Attribute for UnaryOp quantization information.";
  let parameters = (ins "int64_t":$input_zp, "int64_t":$output_zp);
  let assemblyFormat = "`<` struct(params) `>`";
}
````
- **EN**: This TableGen block defines `Tosa_UnaryOpQuantizationAttr` as a `def` record for `TosaOpBase`. It covers assembly syntax, semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_UnaryOpQuantizationAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法, 语义文档。

### Lines 90-100
````tablegen
// There is no explicit BinaryOpQuantizationAttr for 2-input/1-output ops. In
// this case, a tosa.rescale is used to express the inputs to the same scale.
// TODO: Upload WIP legalization document describing this construction by
// example.

// This quantization attribute holds input and weight zero point. Both the
// ConvOp and MatMulOp QuantizationAttrs follow a common design semantic where
// their ownquantization attribute only expresses the numerical behavior at
// the inputs.
// The scaling of their accumulator output is done using an explicit
// tosa.rescale operator that scales the accumulator result to output scale.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 102-107
````tablegen
def Tosa_ConvOpQuantizationAttr
    : Tosa_Attr<"ConvOpQuantization", "conv_quant"> {
  let summary = "Attribute for Conv type op quantization information.";
  let parameters = (ins "int64_t":$input_zp, "int64_t":$weight_zp);
  let assemblyFormat = "`<` struct(params) `>`";
}
````
- **EN**: This TableGen block defines `Tosa_ConvOpQuantizationAttr` as a `def` record for `TosaOpBase`. It covers assembly syntax, semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_ConvOpQuantizationAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法, 语义文档。

### Lines 109-114
````tablegen
def Tosa_MatMulOpQuantizationAttr
    : Tosa_Attr< "MatMulOpQuantization", "matmul_quant"> {
  let summary = "Attribute for MatMulOp quantization information.";
  let parameters = (ins "int64_t":$a_zp, "int64_t":$b_zp);
  let assemblyFormat = "`<` struct(params) `>`";
}
````
- **EN**: This TableGen block defines `Tosa_MatMulOpQuantizationAttr` as a `def` record for `TosaOpBase`. It covers assembly syntax, semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_MatMulOpQuantizationAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法, 语义文档。

### Lines 115-118
````tablegen
// This attribute holds input zero point correction applied to the padding
// zeros to ensure numerical accuracy in the subsequent TOSA operations.
// Its functional application is described in the tosa.pad() operator
// description in the specification.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 120-124
````tablegen
def Tosa_PadOpQuantizationAttr : Tosa_Attr<"PadOpQuantization", "pad_quant"> {
  let summary = "Attribute for PadOp quantization information.";
  let parameters = (ins "int64_t":$input_zp);
  let assemblyFormat = "`<` struct(params) `>`";
}
````
- **EN**: This TableGen block defines `Tosa_PadOpQuantizationAttr` as a `def` record for `TosaOpBase`. It covers assembly syntax, semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_PadOpQuantizationAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法, 语义文档。

### Lines 125-131
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Operator Quantization Builders.
//===----------------------------------------------------------------------===//

// This builder is called on all convolution operators except for TransposeConv,
// which has specialized output shape semantics. The builder also defines the
// bitwidth of the output given the bit width of the input & weight content.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 133-143
````tablegen
def Tosa_ConvOpQuantInfoBuilder : OpBuilder<
  (ins "::mlir::Type":$outputType, "::mlir::Value":$input,
       "::mlir::Value":$weight, "::mlir::Value":$bias,
       "::mlir::DenseI64ArrayAttr":$pad, "::mlir::DenseI64ArrayAttr":$stride,
       "::mlir::DenseI64ArrayAttr":$dilation,
       "::mlir::TypeAttr":$acc_type),
  [{
    buildConvOpWithQuantInfo($_builder, $_state, outputType,
                             input, weight, bias,
                             pad, stride, dilation, acc_type);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_ConvOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ConvOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 144-144
````tablegen
// Handles tosa.transpose_conv2d which has an outpad and output shape attribute.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 146-156
````tablegen
def Tosa_TransConvOpQuantInfoBuilder : OpBuilder<
  (ins "::mlir::Type":$outputType, "::mlir::Value":$input,
       "::mlir::Value":$weight, "mlir::Value":$bias,
       "::mlir::DenseI64ArrayAttr":$outpad,
       "::mlir::DenseI64ArrayAttr":$stride,
       "::mlir::TypeAttr":$acc_type),
  [{
    buildTransConvOpWithQuantInfo($_builder, $_state, outputType,
                                  input, weight, bias,
                                  outpad, stride, acc_type);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_TransConvOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TransConvOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 157-160
````tablegen
// The tosa.matmul op is also intended to be generated where a fully_connected
// op must be constructed where the weight is not a constant. In this case,
// the fully_connected op must be expressed using matmul.
// TODO: Add link to the leglization document explaining this.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 162-167
````tablegen
def Tosa_MatMulOpQuantInfoBuilder : OpBuilder<
  (ins "Type":$outputType, "Value":$a, "Value":$b),
  [{
    buildMatMulOpWithQuantInfo($_builder, $_state, outputType,
                               a, b);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_MatMulOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MatMulOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 168-170
````tablegen
// Both the tosa.avg_pool2d and unary ops use the same
// UnaryOpQuantizationAttr but the avg_pool operator has its own builder as it
// has additional parameters not part of the unary ops.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 172-179
````tablegen
def Tosa_AvgPool2dOpQuantInfoBuilder : OpBuilder<
  (ins "::mlir::Type":$outputType, "::mlir::Value":$input,
       "::mlir::DenseI64ArrayAttr":$kernel, "::mlir::DenseI64ArrayAttr":$stride,
       "::mlir::DenseI64ArrayAttr":$pad, "::mlir::TypeAttr":$acc_type),
  [{
    buildAvgPool2dOpWithQuantInfo($_builder, $_state, outputType,
                                  input, kernel, stride, pad, acc_type);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_AvgPool2dOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_AvgPool2dOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 181-190
````tablegen
def Tosa_AvgPool2dAdaptiveOpQuantInfoBuilder
    : OpBuilder<(ins "::mlir::Type":$outputType, "::mlir::Value":$input,
                    "::mlir::DenseI64ArrayAttr":$kernel,
                    "::mlir::DenseI64ArrayAttr":$stride,
                    "::mlir::DenseI64ArrayAttr":$pad,
                    "::mlir::TypeAttr":$acc_type),
                [{
    buildAvgPool2dAdaptiveOpWithQuantInfo($_builder, $_state, outputType,
                                          input, kernel, stride, pad, acc_type);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_AvgPool2dAdaptiveOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_AvgPool2dAdaptiveOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 191-193
````tablegen
// This builder is called on single-parameter negate operators that have a scale
// relationship between their input and output, expressed by the
// UnaryOpQuantizationAttr.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 195-199
````tablegen
def Tosa_NegateOpQuantInfoBuilder : OpBuilder<
  (ins "Type":$outputType, "Value":$input),
  [{
    buildNegateOpWithQuantInfo($_builder, $_state, outputType, input);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_NegateOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_NegateOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 200-202
````tablegen
// These builders are called on the TOSA pad operator that needs to create its
// own OptionalAttr quantization_attr parameter to scale the padding values
// correctly.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 204-209
````tablegen
def Tosa_PadOpQuantInfoBuilder : OpBuilder<
  (ins "Type":$outputType, "Value":$input, "Value":$paddings),
  [{
    buildPadOpWithQuantInfo($_builder, $_state, outputType,
                            input, paddings);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_PadOpQuantInfoBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_PadOpQuantInfoBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 210-212
````tablegen
// This builder is called on the TOSA variable operator with a variable type
// and optional initial value. The builder will extract var_shape and element type
// attributes from variable type.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 214-218
````tablegen
def Tosa_VariableOpBuilder : OpBuilder<
  (ins "StringRef":$sym_name, "Type":$variable_type, "Attribute":$initial_value),
  [{
    buildVariableOp($_builder, $_state, sym_name, variable_type, initial_value);
  }]>;
````
- **EN**: This TableGen block defines `Tosa_VariableOpBuilder` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_VariableOpBuilder` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 219-219
````tablegen
// Wrapper over base I32EnumAttr to set common fields.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 222-226
````tablegen
class Tosa_I32Enum<string name, string description, list<I32EnumAttrCase> cases>
     : I32EnumAttr<name, description, cases> {
   let genSpecializedAttr = 0;
   let cppNamespace = "::mlir::tosa";
}
````
- **EN**: This TableGen block defines `Tosa_I32Enum` as a `class` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_I32Enum` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 228-232
````tablegen
class Tosa_I32EnumAttr<string name, string description, string mnemonic,
                         list<I32EnumAttrCase> cases>
    : EnumAttr<Tosa_Dialect, Tosa_I32Enum<name, description, cases>, mnemonic> {
   let assemblyFormat = "`<` $value `>`";
}
````
- **EN**: This TableGen block defines `Tosa_I32EnumAttr` as a `class` record for `TosaOpBase`. It covers assembly syntax.
- **CN**: 该 TableGen 代码块将 `Tosa_I32EnumAttr` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法。

### Lines 233-256
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Profiles and extensions
//
// Profile:
// INT : Integer Inference. Integer operations, primarily 8 and 32-bit values.
// FP  : Floating-Point Inference. Primarily FP16 and FP32 operations.
//
// Extension:
// INT64        : 64-bit integer operations.
// INT16        : 16-bit integer operations.
// INT4         : 4-bit integer weights.
// BF16         : BFloat16 operations.
// FP8E4M3      : 8-bit floating-point operations E4M3.
// FP8E5M2      : 8-bit floating-point operations E5M2.
// FFT          : Fast Fourier Transform operations.
// VARIABLE     : Stateful variable operations.
// CONTROLFLOW  : Control Flow operations.
// DOUBLEROUND  : Adds double rounding support to the RESCALE operator.
// INEXACTROUND : Adds inexact rounding support to the RESCALE operator.
// DYNAMIC      : Removes all Compile Time Constant state for CTC inputs.
// MXFP         : Microscaling formats.
// MXFP_CONV    : Microscaling format convolution.
// SHAPE        : Shape calcuation operators.
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 259-259
````tablegen
def Tosa_NONE : I32EnumAttrCase<"none", 0>;
````
- **EN**: This TableGen block defines `Tosa_NONE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_NONE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 260-260
````tablegen
def Tosa_PRO_INT   : I32EnumAttrCase<"pro_int", 1>;
````
- **EN**: This TableGen block defines `Tosa_PRO_INT` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_PRO_INT` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 261-261
````tablegen
def Tosa_PRO_FP   : I32EnumAttrCase<"pro_fp", 2>;
````
- **EN**: This TableGen block defines `Tosa_PRO_FP` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_PRO_FP` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 263-271
````tablegen
def Tosa_ProfileAttr
    : Tosa_I32EnumAttr<"Profile", "supported TOSA profiles", "prof",
                       [Tosa_PRO_INT, Tosa_PRO_FP, Tosa_NONE]> {
  let extraClassDeclaration = [{
    static llvm::SmallVector<Profile, 2> getAllValues() {
      return {Profile::pro_int, Profile::pro_fp};
    }
  }];
}
````
- **EN**: This TableGen block defines `Tosa_ProfileAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ProfileAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 273-274
````tablegen
def Tosa_ProfileArrayAttr
    : TypedArrayAttrBase<Tosa_ProfileAttr, "TOSA profile array attribute">;
````
- **EN**: This TableGen block defines `Tosa_ProfileArrayAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ProfileArrayAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 276-276
````tablegen
def Tosa_EXT_NONE         : I32EnumAttrCase<"none", 0>;
````
- **EN**: This TableGen block defines `Tosa_EXT_NONE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_NONE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 277-277
````tablegen
def Tosa_EXT_INT16        : I32EnumAttrCase<"int16", 1>;
````
- **EN**: This TableGen block defines `Tosa_EXT_INT16` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_INT16` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 278-278
````tablegen
def Tosa_EXT_INT4         : I32EnumAttrCase<"int4", 2>;
````
- **EN**: This TableGen block defines `Tosa_EXT_INT4` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_INT4` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 279-279
````tablegen
def Tosa_EXT_BF16         : I32EnumAttrCase<"bf16", 3>;
````
- **EN**: This TableGen block defines `Tosa_EXT_BF16` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_BF16` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 280-280
````tablegen
def Tosa_EXT_FP8E4M3      : I32EnumAttrCase<"fp8e4m3", 4>;
````
- **EN**: This TableGen block defines `Tosa_EXT_FP8E4M3` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_FP8E4M3` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 281-281
````tablegen
def Tosa_EXT_FP8E5M2      : I32EnumAttrCase<"fp8e5m2", 5>;
````
- **EN**: This TableGen block defines `Tosa_EXT_FP8E5M2` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_FP8E5M2` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 282-282
````tablegen
def Tosa_EXT_FFT          : I32EnumAttrCase<"fft", 6>;
````
- **EN**: This TableGen block defines `Tosa_EXT_FFT` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_FFT` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 283-283
````tablegen
def Tosa_EXT_VARIABLE     : I32EnumAttrCase<"variable", 7>;
````
- **EN**: This TableGen block defines `Tosa_EXT_VARIABLE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_VARIABLE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 284-284
````tablegen
def Tosa_EXT_CONTROLFLOW  : I32EnumAttrCase<"controlflow", 8>;
````
- **EN**: This TableGen block defines `Tosa_EXT_CONTROLFLOW` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_CONTROLFLOW` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 285-285
````tablegen
def Tosa_EXT_DOUBLEROUND  : I32EnumAttrCase<"doubleround", 9>;
````
- **EN**: This TableGen block defines `Tosa_EXT_DOUBLEROUND` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_DOUBLEROUND` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 286-286
````tablegen
def Tosa_EXT_INEXACTROUND : I32EnumAttrCase<"inexactround", 10>;
````
- **EN**: This TableGen block defines `Tosa_EXT_INEXACTROUND` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_INEXACTROUND` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 287-287
````tablegen
def Tosa_EXT_DYNAMIC      : I32EnumAttrCase<"dynamic", 11>;
````
- **EN**: This TableGen block defines `Tosa_EXT_DYNAMIC` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_DYNAMIC` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 288-288
````tablegen
def Tosa_EXT_MXFP         : I32EnumAttrCase<"mxfp", 12>;
````
- **EN**: This TableGen block defines `Tosa_EXT_MXFP` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_MXFP` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 289-289
````tablegen
def Tosa_EXT_INT64        : I32EnumAttrCase<"int64", 13>;
````
- **EN**: This TableGen block defines `Tosa_EXT_INT64` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_INT64` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 290-290
````tablegen
def Tosa_EXT_MXFP_CONV    : I32EnumAttrCase<"mxfp_conv", 14>;
````
- **EN**: This TableGen block defines `Tosa_EXT_MXFP_CONV` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_MXFP_CONV` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 291-291
````tablegen
def Tosa_EXT_SHAPE        : I32EnumAttrCase<"shape", 15>;
````
- **EN**: This TableGen block defines `Tosa_EXT_SHAPE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_EXT_SHAPE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 293-312
````tablegen
def Tosa_ExtensionAttr
    : Tosa_I32EnumAttr<"Extension", "supported TOSA extensions", "ext", [
      Tosa_EXT_NONE, Tosa_EXT_INT16, Tosa_EXT_INT4, Tosa_EXT_BF16,
      Tosa_EXT_FP8E4M3, Tosa_EXT_FP8E5M2, Tosa_EXT_FFT, Tosa_EXT_VARIABLE,
      Tosa_EXT_CONTROLFLOW, Tosa_EXT_DOUBLEROUND, Tosa_EXT_INEXACTROUND,
      Tosa_EXT_DYNAMIC, Tosa_EXT_MXFP, Tosa_EXT_INT64, Tosa_EXT_MXFP_CONV,
      Tosa_EXT_SHAPE,
    ]> {
  let extraClassDeclaration = [{
    static llvm::SmallVector<Extension, 14> getAllValues() {
      return {
        Extension::int16, Extension::int4, Extension::bf16,
        Extension::fp8e4m3, Extension::fp8e5m2, Extension::fft,
        Extension::variable, Extension::controlflow, Extension::doubleround,
        Extension::inexactround, Extension::dynamic, Extension::mxfp,
        Extension::int64, Extension::mxfp_conv, Extension::shape
      };
    }
  }];
}
````
- **EN**: This TableGen block defines `Tosa_ExtensionAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ExtensionAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 314-317
````tablegen
def Tosa_ExtensionArrayAttr
    : TypedArrayAttrBase<Tosa_ExtensionAttr, "TOSA extension array attribute">;

// The base class for defining op availability dimensions.
````
- **EN**: This TableGen block defines `Tosa_ExtensionArrayAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ExtensionArrayAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 318-353
````tablegen
class Availability {
  // The following are fields for controlling the generated C++ OpInterface.

  // The namespace for the generated C++ OpInterface subclass.
  string cppNamespace = "::mlir::tosa";

  // The name for the generated C++ OpInterface subclass.
  string interfaceName = ?;

  // The description for the generated C++ OpInterface subclass.
  string interfaceDescription = "";

  // The query function's return type in the generated C++ OpInterface subclass.
  string queryFnRetType = ?;

  // The query function's name in the generated C++ OpInterface subclass.
  string queryFnName = ?;

  // The logic for merging two availability requirements.
  code mergeAction = ?;

  // The initializer for the final availability requirement.
  string initializer = ?;

  // An availability instance's type.
  string instanceType = ?;

  // The following are fields for a concrete availability instance.

  // The code for preparing a concrete instance. This should be C++ statements
  // and will be generated before the `mergeAction` logic.
  code instancePreparation = "";

  // The availability requirement carried by a concrete instance.
  string instance = ?;
}
````
- **EN**: This TableGen block defines `Availability` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Availability` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 356-387
````tablegen
class Profile<list<I32EnumAttrCase> profiles> : Availability {
  let interfaceName = "QueryProfileInterface";
  let interfaceDescription = [{
    Querying interface for the supported set of Tosa profile.

    This interface provides a `getProfiles()` method to query
    the supported set of Tosa profile. The returned value is a
    list of `mlir::Tosa::Profile` enum number.
  }];

  let queryFnRetType = "::llvm::SmallVector<::llvm::ArrayRef<"
                          "::mlir::tosa::Profile>, 1>";
  let queryFnName = "getProfiles";

  let mergeAction = !if(
      !empty(profiles), "", "$overall.emplace_back($instance)");

  let initializer = "{}";

  let instanceType = "::llvm::ArrayRef<::mlir::tosa::Profile>";

  // Pack all profiles as a static array and get its reference.
  let instancePreparation = !if(!empty(profiles), "",
    "static const ::mlir::tosa::Profile profs[] = {" #
    !interleave(!foreach(prof, profiles,
                         "::mlir::tosa::Profile::" # prof.symbol), ", ") #
    "}; " #
    "ArrayRef<::mlir::tosa::Profile> " #
      "ref(profs, std::size(profs));");

  let instance = "ref";
}
````
- **EN**: This TableGen block defines `Profile` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Profile` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 389-420
````tablegen
class Extension<list<I32EnumAttrCase> extensions> : Availability {
  let interfaceName = "QueryExtensionInterface";
  let interfaceDescription = [{
    Querying interface for the supported set of TOSA extension.

    This interface provides a `getExtensions()` method to query
    the supported set of Tosa extension. The returned value is a
    list of `mlir::Tosa::Extension` enum number.
  }];

  let queryFnRetType = "::llvm::SmallVector<::llvm::ArrayRef<"
                          "::mlir::tosa::Extension>, 1>";
  let queryFnName = "getExtensions";

  let mergeAction = !if(
      !empty(extensions), "", "$overall.emplace_back($instance)");

  let initializer = "{}";

  let instanceType = "::llvm::ArrayRef<::mlir::tosa::Extension>";

  // Pack all extensions as a static array and get its reference.
  let instancePreparation = !if(!empty(extensions), "",
    "static const ::mlir::tosa::Extension exts[] = {" #
    !interleave(!foreach(ext, extensions,
                         "::mlir::tosa::Extension::" # ext.symbol), ", ") #
    "}; " #
    "ArrayRef<::mlir::tosa::Extension> " #
      "ref(exts, std::size(exts));");

  let instance = "ref";
}
````
- **EN**: This TableGen block defines `Extension` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Extension` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 421-423
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Levels
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 426-426
````tablegen
def Tosa_LVL_NONE : I32EnumAttrCase<"none", 0>;
````
- **EN**: This TableGen block defines `Tosa_LVL_NONE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_LVL_NONE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 427-427
````tablegen
def Tosa_LVL_8K   : I32EnumAttrCase<"eightK", 1, "8k">;
````
- **EN**: This TableGen block defines `Tosa_LVL_8K` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_LVL_8K` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 429-434
````tablegen
def Tosa_LevelAttr
    : Tosa_I32EnumAttr<"Level", "supported TOSA levels", "level", [Tosa_LVL_NONE, Tosa_LVL_8K]>;

//===----------------------------------------------------------------------===//
// TOSA Specification versions
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_LevelAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_LevelAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 436-436
````tablegen
def Tosa_V_1_0 : I32EnumAttrCase<"V_1_0", 0, "1.0">;
````
- **EN**: This TableGen block defines `Tosa_V_1_0` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_V_1_0` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 437-437
````tablegen
def Tosa_V_1_1_DRAFT : I32EnumAttrCase<"V_1_1_DRAFT", 1, "1.1.draft">;
````
- **EN**: This TableGen block defines `Tosa_V_1_1_DRAFT` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_V_1_1_DRAFT` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 439-445
````tablegen
def Tosa_SpecificationVersion : Tosa_I32EnumAttr<
      "SpecificationVersion", "TOSA specification version", "specification_version",
      [Tosa_V_1_0, Tosa_V_1_1_DRAFT]>;

//===----------------------------------------------------------------------===//
// TOSA target environment.
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_SpecificationVersion` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_SpecificationVersion` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 446-458
````tablegen
def Tosa_TargetEnv : Tosa_Attr<"TargetEnv", "target_env"> {
  let summary = "Target environment information.";
  let parameters = ( ins
    "SpecificationVersion": $specification_version,
    "Level": $level,
    ArrayRefParameter<"Profile">: $profiles,
    ArrayRefParameter<"Extension">: $extensions
  );

  let assemblyFormat = "`<` `specification_version` `=` $specification_version `,` "
                       "`level` `=` $level `,` `profiles` `=` `[` $profiles `]` `,` "
                       "`extensions` `=` `[` $extensions `]` `>`";
}
````
- **EN**: This TableGen block defines `Tosa_TargetEnv` as a `def` record for `TosaOpBase`. It covers assembly syntax, semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_TargetEnv` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法, 语义文档。

### Lines 459-462
````tablegen
//===----------------------------------------------------------------------===//
// Enum attributes.
//===----------------------------------------------------------------------===//
// Defined in `section 3. Enumerations` of the TOSA specification.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 465-465
````tablegen
def Tosa_RESIZE_NEAREST_NEIGHBOR          : I32EnumAttrCase<"NEAREST_NEIGHBOR", 1>;
````
- **EN**: This TableGen block defines `Tosa_RESIZE_NEAREST_NEIGHBOR` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_RESIZE_NEAREST_NEIGHBOR` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 466-466
````tablegen
def Tosa_RESIZE_BILINEAR                  : I32EnumAttrCase<"BILINEAR", 2>;
````
- **EN**: This TableGen block defines `Tosa_RESIZE_BILINEAR` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_RESIZE_BILINEAR` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 468-470
````tablegen
def Tosa_ResizeModeAttr
    : Tosa_I32EnumAttr<"ResizeMode", "Supported resize/upsampling strategies", "resize_mode",
                    [Tosa_RESIZE_NEAREST_NEIGHBOR, Tosa_RESIZE_BILINEAR]>;
````
- **EN**: This TableGen block defines `Tosa_ResizeModeAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ResizeModeAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 472-472
````tablegen
def Tosa_NANPROPAGATION_PROPAGATE : I32EnumAttrCase<"PROPAGATE", 1>;
````
- **EN**: This TableGen block defines `Tosa_NANPROPAGATION_PROPAGATE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_NANPROPAGATION_PROPAGATE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 473-473
````tablegen
def Tosa_NANPROPAGATION_IGNORE    : I32EnumAttrCase<"IGNORE", 2>;
````
- **EN**: This TableGen block defines `Tosa_NANPROPAGATION_IGNORE` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_NANPROPAGATION_IGNORE` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 475-477
````tablegen
def Tosa_NanPropagationModeAttr
    : Tosa_I32EnumAttr<"NanPropagationMode", "Supported NaN propagation strategies", "nan_mode",
                    [Tosa_NANPROPAGATION_PROPAGATE, Tosa_NANPROPAGATION_IGNORE]>;
````
- **EN**: This TableGen block defines `Tosa_NanPropagationModeAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_NanPropagationModeAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 479-479
````tablegen
def Tosa_ROUNDING_SINGLE_ROUND    : I32EnumAttrCase<"SINGLE_ROUND", 1>;
````
- **EN**: This TableGen block defines `Tosa_ROUNDING_SINGLE_ROUND` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ROUNDING_SINGLE_ROUND` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 480-480
````tablegen
def Tosa_ROUNDING_INEXACT_ROUND   : I32EnumAttrCase<"INEXACT_ROUND", 2>;
````
- **EN**: This TableGen block defines `Tosa_ROUNDING_INEXACT_ROUND` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ROUNDING_INEXACT_ROUND` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 481-481
````tablegen
def Tosa_ROUNDING_DOUBLE_ROUND    : I32EnumAttrCase<"DOUBLE_ROUND", 3>;
````
- **EN**: This TableGen block defines `Tosa_ROUNDING_DOUBLE_ROUND` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ROUNDING_DOUBLE_ROUND` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 483-485
````tablegen
def Tosa_RoundingModeAttr
    : Tosa_I32EnumAttr<"RoundingMode", "Supported rounding modes", "rounding_mode",
                    [Tosa_ROUNDING_SINGLE_ROUND, Tosa_ROUNDING_INEXACT_ROUND, Tosa_ROUNDING_DOUBLE_ROUND]>;
````
- **EN**: This TableGen block defines `Tosa_RoundingModeAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_RoundingModeAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 487-487
````tablegen
def Tosa_BLOCK_SIZE_1 : I32EnumAttrCase<"BLOCK_SIZE_1", 1>;
````
- **EN**: This TableGen block defines `Tosa_BLOCK_SIZE_1` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_BLOCK_SIZE_1` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 488-488
````tablegen
def Tosa_BLOCK_SIZE_32 : I32EnumAttrCase<"BLOCK_SIZE_32", 32>;
````
- **EN**: This TableGen block defines `Tosa_BLOCK_SIZE_32` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_BLOCK_SIZE_32` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 490-498
````tablegen
def Tosa_BlockSizeAttr
    : Tosa_I32EnumAttr<"BlockSize", "Block size for the block_scaled formats",
                       "block_size", [Tosa_BLOCK_SIZE_1, Tosa_BLOCK_SIZE_32]> {
  let extraClassDeclaration = [{
    static uint32_t getBlockSizeValue(BlockSize blockSize) {
      return static_cast<uint32_t>(blockSize);
    }
  }];
}
````
- **EN**: This TableGen block defines `Tosa_BlockSizeAttr` as a `def` record for `TosaOpBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_BlockSizeAttr` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。

### Lines 499-501
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Interfaces.
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 504-510
````tablegen
def QueryProfileInterface : OpInterface<"QueryProfileInterface"> {
  let cppNamespace = "::mlir::tosa";
  let methods = [InterfaceMethod<
    "get supported profiles",
    "::llvm::SmallVector<::llvm::ArrayRef<::mlir::tosa::Profile>, 1>",
    "getProfiles">];
}
````
- **EN**: This TableGen block defines `QueryProfileInterface` as a `def` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `QueryProfileInterface` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 512-518
````tablegen
def QueryExtensionInterface : OpInterface<"QueryExtensionInterface"> {
  let cppNamespace = "::mlir::tosa";
  let methods = [InterfaceMethod<
    "get supported extensions",
    "::llvm::SmallVector<::llvm::ArrayRef<::mlir::tosa::Extension>, 1>",
    "getExtensions">];
}
````
- **EN**: This TableGen block defines `QueryExtensionInterface` as a `def` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `QueryExtensionInterface` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 519-523
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Operator Trait.
//===----------------------------------------------------------------------===//

// Permits broadcasting. Elementwise trait is too strict.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 525-527
````tablegen
def TosaElementwiseOperator : NativeOpTrait<"TosaElementwiseOperator"> {
  let cppNamespace = "mlir::OpTrait::tosa";
}
````
- **EN**: This TableGen block defines `TosaElementwiseOperator` as a `def` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TosaElementwiseOperator` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 528-531
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Operator Trait.
//===----------------------------------------------------------------------===//
// Op operands with TOSA shape types must be compile time resolvable
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 533-535
````tablegen
def TosaResolvableShapeOperands : NativeOpTrait<"TosaResolvableShapeOperands"> {
  let cppNamespace = "mlir::OpTrait::tosa";
}
````
- **EN**: This TableGen block defines `TosaResolvableShapeOperands` as a `def` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TosaResolvableShapeOperands` 定义为 `def` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 536-538
````tablegen
//===----------------------------------------------------------------------===//
// TOSA Operator Class.
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 541-554
````tablegen
class Tosa_Op<string mnemonic, list<Trait> traits = []> :
    Op<Tosa_Dialect, mnemonic, !listconcat(traits, [TosaOpInterface,
    DeclareOpInterfaceMethods<QueryProfileInterface>,
    DeclareOpInterfaceMethods<QueryExtensionInterface>,
    TosaResolvableShapeOperands])> {

  // Default availability specification.
  list<Availability> availability = [
    Profile<[]>,
    Extension<[]>];

  // When not set, manual implementation of these methods is required.
  bit autogenAvailability = 1;
}
````
- **EN**: This TableGen block defines `Tosa_Op` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_Op` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 556-563
````tablegen
class Tosa_ElementwiseOp<string mnemonic, list<Trait> traits = []> :
    Tosa_Op<mnemonic, !listconcat(traits, [
              DeclareOpInterfaceMethods<InferShapedTypeOpInterface,
                                        ["inferReturnTypeComponents"]>,
              ResultsBroadcastableShape,
              TosaElementwiseOperator,
              SameOperandsAndResultRank,
              NoMemoryEffect])> {}
````
- **EN**: This TableGen block defines `Tosa_ElementwiseOp` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_ElementwiseOp` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 565-568
````tablegen
class Tosa_ElementwiseUnaryOp<string mnemonic, list<Trait> traits = []> :
    Tosa_ElementwiseOp<mnemonic, !listconcat(traits, [
              SameOperandsAndResultShape,
              SameOperandsAndResultElementType])> {}
````
- **EN**: This TableGen block defines `Tosa_ElementwiseUnaryOp` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_ElementwiseUnaryOp` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 570-571
````tablegen
class Tosa_InferTensorTypeOp<string mnemonic, list<Trait> traits = []>
    : Tosa_Op<mnemonic, !listconcat(traits, [InferTensorTypeAdaptor])> {}
````
- **EN**: This TableGen block defines `Tosa_InferTensorTypeOp` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_InferTensorTypeOp` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 573-574
````tablegen
class Tosa_InferShapedTypeOp<string mnemonic, list<Trait> traits = []>
    : Tosa_Op<mnemonic, !listconcat(traits, [InferShapedTypeOpAdaptor])> {}
````
- **EN**: This TableGen block defines `Tosa_InferShapedTypeOp` as a `class` record for `TosaOpBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_InferShapedTypeOp` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 575-576
````tablegen
// The "SameVariadicOperandSize" trait allows us to pass optional arguments
// for multiple zero points in convolution ops.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 578-583
````tablegen
class Tosa_ConvOp<string mnemonic, list<Trait> traits = []>
    : Tosa_InferShapedTypeOp<mnemonic, !listconcat(traits,
      [SameVariadicOperandSize, NoMemoryEffect])> {
  let assemblyFormat =
      "operands attr-dict `:` functional-type(operands, results)";
}
````
- **EN**: This TableGen block defines `Tosa_ConvOp` as a `class` record for `TosaOpBase`. It covers assembly syntax, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_ConvOp` 定义为 `class` 记录，用于描述 `TosaOpBase` 相关的声明式信息。 其中涉及 汇编语法, trait/接口组合。

### Lines 584-584
````tablegen
#endif // TOSA_OP_BASE
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

- mlir/IR/AttrTypeBase.td
- mlir/IR/EnumAttr.td
- mlir/IR/OpBase.td
- mlir/Interfaces/SideEffectInterfaces.td
- mlir/Interfaces/InferTypeOpInterface.td
- mlir/Interfaces/LoopLikeInterface.td
- mlir/Dialect/Tosa/IR/TosaInterfaces.td
- Tosa_Dialect builds on Dialect
- Tosa_UnaryOpQuantizationAttr builds on Tosa_Attr<"UnaryOpQuantization", "unary_quant">
- Tosa_ConvOpQuantizationAttr builds on Tosa_Attr<"ConvOpQuantization", "conv_quant">
- Tosa_MatMulOpQuantizationAttr builds on Tosa_Attr< "MatMulOpQuantization", "matmul_quant">
- Tosa_PadOpQuantizationAttr builds on Tosa_Attr<"PadOpQuantization", "pad_quant">
- Tosa_ConvOpQuantInfoBuilder builds on OpBuilder<
- Tosa_TransConvOpQuantInfoBuilder builds on OpBuilder<
- Tosa_MatMulOpQuantInfoBuilder builds on OpBuilder<
- Tosa_AvgPool2dOpQuantInfoBuilder builds on OpBuilder<
- Tosa_AvgPool2dAdaptiveOpQuantInfoBuilder builds on OpBuilder<(ins "::mlir::Type":$outputType, "::mlir::Value":$input,
- Tosa_NegateOpQuantInfoBuilder builds on OpBuilder<
- Tosa_PadOpQuantInfoBuilder builds on OpBuilder<
- Tosa_VariableOpBuilder builds on OpBuilder<
