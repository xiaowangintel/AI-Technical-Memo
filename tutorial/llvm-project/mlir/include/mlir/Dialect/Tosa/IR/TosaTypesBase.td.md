# TosaTypesBase.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/IR/TosaTypesBase.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR TosaTypesBase component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file defines the type definitions for the TOSA dialect.
- **用途（CN）**: 为 MLIR 的 TosaTypesBase 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
````tablegen
//===-- TosaTypesBase.td - TOSA type definitions -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the type definitions for the TOSA dialect.
//
//===----------------------------------------------------------------------===//

#ifndef TOSA_TYPES_BASE
#define TOSA_TYPES_BASE

include "mlir/IR/AttrTypeBase.td"
include "mlir/IR/OpBase.td"

include "mlir/Dialect/Tosa/IR/TosaOpBase.td"

//===----------------------------------------------------------------------===//
// Tosa Type Definitions.
//===----------------------------------------------------------------------===//

// The base class for Tosa dialect types.
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 26-29
````tablegen
class Tosa_Type<string name, string typeMnemonic, list<Trait> traits = []>
    : TypeDef<Tosa_Dialect, name, traits> {
  let mnemonic = typeMnemonic;
}
````
- **EN**: This TableGen block defines `Tosa_Type` as a `class` record for `TosaTypesBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_Type` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 30-33
````tablegen
// The base class of a quantized type.
// Param tuple is: [bitwidth, zeropt, smantissa, sexp, low_end, high_end].
// Where low and high ends are 0,255 when unsigned, -128,127 when signed, for
// the 8-bit case.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 35-43
````tablegen
class Tosa_QuantizedType<string n, list<int> params, bit signed>
  : Type<And<[CPred<"::llvm::isa<mlir::quant::QuantizedType>($_self)">,
              CPred<"::llvm::cast<mlir::quant::QuantizedType>($_self)" #
                    ".getStorageTypeIntegralWidth() == " # !head(params)>]>,
    "Q" # !if (signed, "int", "uint") # !head(params) # " type"> {
  string name = n;
  string asTraitArgsStr = !interleave(params, ", ") #
                          !if(signed, ", true", ", false");
}
````
- **EN**: This TableGen block defines `Tosa_QuantizedType` as a `class` record for `TosaTypesBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Tosa_QuantizedType` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 44-47
````tablegen
//===----------------------------------------------------------------------===//
// Non-Quantized Signed Integer Types.
// Used to express accumulator results or compare results.
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 50-50
````tablegen
def Tosa_Int4 : I<4>;
````
- **EN**: This TableGen block defines `Tosa_Int4` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int4` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 51-51
````tablegen
def Tosa_Int8 : I<8>;
````
- **EN**: This TableGen block defines `Tosa_Int8` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int8` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 52-52
````tablegen
def Tosa_Int16 : I<16>;
````
- **EN**: This TableGen block defines `Tosa_Int16` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int16` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 53-53
````tablegen
def Tosa_Int32 : I<32>;
````
- **EN**: This TableGen block defines `Tosa_Int32` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int32` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 54-59
````tablegen
def Tosa_Int64 : I<64>;

// The TOSA dialect allows more types than the TOSA standard to allow for
// experimentation. For historical reasons, signless is used in the place of
// signed.
// The TosaValidation pass can be used to check for standard conformance.
````
- **EN**: This TableGen block defines `Tosa_Int64` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int64` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 60-61
````tablegen
def Tosa_Int : AnyTypeOf<[AnyUnsignedInteger,
                          AnySignlessInteger]>;
````
- **EN**: This TableGen block defines `Tosa_Int` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 63-64
````tablegen
def Tosa_Int32Or64 : AnyTypeOf<[Tosa_Int32,
                                Tosa_Int64]>;
````
- **EN**: This TableGen block defines `Tosa_Int32Or64` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int32Or64` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 66-80
````tablegen
def Tosa_Int16Or32 : AnyTypeOf<[Tosa_Int16,
                                Tosa_Int32]>;

//===----------------------------------------------------------------------===//
// Quantized Integer Types.
// Datatype for network feature map or weight content.
//===----------------------------------------------------------------------===//
//===----------------------------------------------------------------------===//
// Name    Symmetry   Grouping                Sign
//===----------------------------------------------------------------------===//
// uint8 : asymmetric per tensor ,            unsigned
// int4  : symmetric  per channel,            signed
// int8  : symmetric  per tensor/per channel, signed
// int16 : symmetric  per tensor,             signed
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_Int16Or32` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int16Or32` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 81-92
````tablegen
def Tosa_QuantizedInt : AnyTypeOf<[Tosa_QuantizedType<"uint8", [8], 0>,
                                   Tosa_QuantizedType<"int4", [4, 0], 1>,
                                   Tosa_QuantizedType<"int8", [8, 0], 1>,
                                   Tosa_QuantizedType<"int16", [16, 0], 1>,
                                   Tosa_QuantizedType<"int32", [32, 0], 1>]>;

//===----------------------------------------------------------------------===//
// Custom TOSA element types.
//===----------------------------------------------------------------------===//

// MLIR doesn't have a builtin type for mxint8 yet. For now declared it as a
// custom TOSA type. This may be changed in the future.
````
- **EN**: This TableGen block defines `Tosa_QuantizedInt` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_QuantizedInt` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 93-98
````tablegen
def Tosa_MXInt8 : Tosa_Type<"mxint8", "mxint8"> {
  let summary = "INT8 type as defined by OCP-MX";
  let description = [{
    8-bit integer format with an implicit 1/64 scale defined by OCP-MX.
  }];
}
````
- **EN**: This TableGen block defines `Tosa_MXInt8` as a `def` record for `TosaTypesBase`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `Tosa_MXInt8` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 语义文档。

### Lines 99-101
````tablegen
//===----------------------------------------------------------------------===//
// Multi-category types.
//===----------------------------------------------------------------------===//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 103-104
````tablegen
def Tosa_AnyNumber : AnyTypeOf<[Tosa_Int, Tosa_QuantizedInt, AnyFloat, Tosa_MXInt8],
                                "number">;
````
- **EN**: This TableGen block defines `Tosa_AnyNumber` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_AnyNumber` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 106-107
````tablegen
def Tosa_MXFPNumber : AnyTypeOf<[F8E4M3FN, F8E5M2, F4E2M1FN, F6E2M3FN, F6E3M2FN, Tosa_MXInt8],
                                "micro-scaling format number">;
````
- **EN**: This TableGen block defines `Tosa_MXFPNumber` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPNumber` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 108-112
````tablegen
def Tosa_MXFPScaleNumber : AnyTypeOf<[F8E8M0FNU], "micro-scaling format scale number">;

//===----------------------------------------------------------------------===//
// TOSA Tensor Conformance
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_MXFPScaleNumber` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPScaleNumber` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 114-116
````tablegen
def HasNo0Dimensions : And<[
    IsRankedTensorTypePred,
    CPred<"::llvm::all_of(::llvm::cast<::mlir::RankedTensorType>($_self).getShape(), [](auto v) { return v != 0; })">]>;
````
- **EN**: This TableGen block defines `HasNo0Dimensions` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `HasNo0Dimensions` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 118-120
````tablegen
def AllDimensionsAreSizeOne : And<[
    IsRankedTensorTypePred,
    CPred<"::llvm::all_of(::llvm::cast<::mlir::RankedTensorType>($_self).getShape(), [](auto v) { return v == 1; })">]>;
````
- **EN**: This TableGen block defines `AllDimensionsAreSizeOne` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `AllDimensionsAreSizeOne` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 122-124
````tablegen
def AtLeastRankOne : And<[
  IsRankedTensorTypePred,
  CPred<"::llvm::cast<::mlir::RankedTensorType>($_self).getRank() >= 1">]>;
````
- **EN**: This TableGen block defines `AtLeastRankOne` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `AtLeastRankOne` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 126-128
````tablegen
class TosaTensorOf<
    list<Type> allowedTypes, string summary = "tosa-conformant tensor">
    : TensorOf<allowedTypes, [Or<[HasNo0Dimensions, IsUnrankedTensorTypePred]>], summary>;
````
- **EN**: This TableGen block defines `TosaTensorOf` as a `class` record for `TosaTypesBase`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `TosaTensorOf` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 语义文档。

### Lines 130-132
````tablegen
class TosaRankedTensorOf<
    list<Type> allowedTypes, list<Pred> preds = [], string summary = "tosa-conformant ranked tensor">
    : RankedTensorOf<allowedTypes, !listconcat([HasNo0Dimensions], preds), summary>;
````
- **EN**: This TableGen block defines `TosaRankedTensorOf` as a `class` record for `TosaTypesBase`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `TosaRankedTensorOf` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 语义文档。

### Lines 134-135
````tablegen
class TosaUnrankedTensorOf<list<Type> allowedTypes, list<Pred> preds = [], string summary = "tosa-conformant unranked tensor">
    : UnrankedTensorOf<allowedTypes, preds, summary>;
````
- **EN**: This TableGen block defines `TosaUnrankedTensorOf` as a `class` record for `TosaTypesBase`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `TosaUnrankedTensorOf` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 语义文档。

### Lines 137-140
````tablegen
class TosaTensorRankOf<list<Type> allowedTypes, list<int> ranks>
    : TosaRankedTensorOf<allowedTypes,
      [HasAnyRankOfPred<ranks>],
      !interleave(!foreach(rank, ranks, rank # "D"), "/") # " tensor">;
````
- **EN**: This TableGen block defines `TosaTensorRankOf` as a `class` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `TosaTensorRankOf` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 142-149
````tablegen
class TosaScalarTensorOf<list<Type> allowedTypes, list<int> ranks>
    : TosaRankedTensorOf<allowedTypes,
      [HasAnyRankOfPred<ranks>, AllDimensionsAreSizeOne],
      "tosa-conformant scalar tensor">;

//===----------------------------------------------------------------------===//
// Tensor types
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `TosaScalarTensorOf` as a `class` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `TosaScalarTensorOf` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 151-151
````tablegen
def Tosa_I1Tensor : TosaTensorOf<[I1]>;
````
- **EN**: This TableGen block defines `Tosa_I1Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_I1Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 152-152
````tablegen
def Tosa_Int32Tensor : TosaTensorOf<[Tosa_Int32]>;
````
- **EN**: This TableGen block defines `Tosa_Int32Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int32Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 153-153
````tablegen
def Tosa_Int32Or64Tensor : TosaTensorOf<[Tosa_Int32Or64]>;
````
- **EN**: This TableGen block defines `Tosa_Int32Or64Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int32Or64Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 155-157
````tablegen
def Tosa_FloatTensor : TosaTensorOf<[AnyFloat]>;

// Either ranked or unranked tensor of TOSA supported element types.
````
- **EN**: This TableGen block defines `Tosa_FloatTensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_FloatTensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 158-160
````tablegen
def Tosa_Tensor : TosaTensorOf<[Tosa_AnyNumber]>;

// Must be ranked but no further constraints
````
- **EN**: This TableGen block defines `Tosa_Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 161-163
````tablegen
def Tosa_RankedTensor : TosaRankedTensorOf<[Tosa_AnyNumber]>;

// Any tensor element type allowed in Tosa ops.
````
- **EN**: This TableGen block defines `Tosa_RankedTensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_RankedTensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 164-165
````tablegen
def Tosa_ElementType : Type<Or<[Tosa_Int.predicate, Tosa_QuantizedInt.predicate,
                                AnyFloat.predicate]>, "tosa.dtype">;
````
- **EN**: This TableGen block defines `Tosa_ElementType` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ElementType` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 167-172
````tablegen
class Tosa_TensorOfOrNone<list<Type> allowedTypes, string description = ""> :
  AnyTypeOf<[TosaTensorOf<allowedTypes>, NoneType], description>;

//===----------------------------------------------------------------------===//
// Tensor types with constrained ranks.
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_TensorOfOrNone` as a `class` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TensorOfOrNone` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 174-174
````tablegen
def Tosa_Rank0Tensor : TosaTensorRankOf<[Tosa_AnyNumber], [0]>;
````
- **EN**: This TableGen block defines `Tosa_Rank0Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Rank0Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 176-176
````tablegen
def Tosa_ScalarTensor : AnyTypeOf<[TosaUnrankedTensorOf<[Tosa_Int, AnyFloat]>, TosaScalarTensorOf<[Tosa_AnyNumber], [1]>]>;
````
- **EN**: This TableGen block defines `Tosa_ScalarTensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ScalarTensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 177-177
````tablegen
def Tosa_ScalarInt8Tensor : AnyTypeOf<[TosaUnrankedTensorOf<[Tosa_Int8]>, TosaScalarTensorOf<[Tosa_Int8], [1]>]>;
````
- **EN**: This TableGen block defines `Tosa_ScalarInt8Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ScalarInt8Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 178-178
````tablegen
def Tosa_ScalarIntOrFloatTensor : AnyTypeOf<[TosaUnrankedTensorOf<[Tosa_Int, AnyFloat]>, TosaScalarTensorOf<[Tosa_Int, AnyFloat], [1]>]>;
````
- **EN**: This TableGen block defines `Tosa_ScalarIntOrFloatTensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ScalarIntOrFloatTensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 179-184
````tablegen
def Tosa_ScalarInt32Tensor : AnyTypeOf<[TosaUnrankedTensorOf<[Tosa_Int32]>, TosaScalarTensorOf<[Tosa_Int32], [1]>]>;

// We include unranked tensors as a supported type for all possible tosa
// Tensors as unranked does not guarantee invalid. If unranked tensors exist
// they should be shape propagate used Tosa's shape inference pass and verified
// to not include any remaining unranked tensors.
````
- **EN**: This TableGen block defines `Tosa_ScalarInt32Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_ScalarInt32Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 185-185
````tablegen
def Tosa_UnrankedTensor : TosaUnrankedTensorOf<[Tosa_AnyNumber]>;
````
- **EN**: This TableGen block defines `Tosa_UnrankedTensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_UnrankedTensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 187-187
````tablegen
def Tosa_Tensor1D : AnyTypeOf<[Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [1]>], "1-d tosa-conformant tensor", "::mlir::TensorType">;
````
- **EN**: This TableGen block defines `Tosa_Tensor1D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor1D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 188-188
````tablegen
def Tosa_Tensor2D : AnyTypeOf<[Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [2]>], "2-d tosa-conformant tensor", "::mlir::TensorType">;
````
- **EN**: This TableGen block defines `Tosa_Tensor2D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor2D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 189-189
````tablegen
def Tosa_Tensor3D : AnyTypeOf<[Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [3]>], "3-d tosa-conformant tensor", "::mlir::TensorType">;
````
- **EN**: This TableGen block defines `Tosa_Tensor3D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor3D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 190-190
````tablegen
def Tosa_Tensor4D : AnyTypeOf<[Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [4]>], "4-d tosa-conformant tensor", "::mlir::TensorType">;
````
- **EN**: This TableGen block defines `Tosa_Tensor4D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor4D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 191-193
````tablegen
def Tosa_Tensor5D : AnyTypeOf<[Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [5]>], "5-d tosa-conformant tensor", "::mlir::TensorType">;

// 1D tensor of specific types
````
- **EN**: This TableGen block defines `Tosa_Tensor5D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor5D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 194-194
````tablegen
def Tosa_1DInt8Tensor : 1DTensorOf<[Tosa_Int8]>;
````
- **EN**: This TableGen block defines `Tosa_1DInt8Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_1DInt8Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 195-197
````tablegen
def Tosa_1DInt16Or32Tensor : 1DTensorOf<[Tosa_Int16Or32]>;

// Ranked tensors up to given rank.
````
- **EN**: This TableGen block defines `Tosa_1DInt16Or32Tensor` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_1DInt16Or32Tensor` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 198-199
````tablegen
def Tosa_Tensor1Dto4D : AnyTypeOf<[
  Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [1,2,3,4]>]>;
````
- **EN**: This TableGen block defines `Tosa_Tensor1Dto4D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor1Dto4D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 200-201
````tablegen
def Tosa_Tensor1Dto6D : AnyTypeOf<[
  Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [1,2,3,4,5,6]>]>;
````
- **EN**: This TableGen block defines `Tosa_Tensor1Dto6D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Tensor1Dto6D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 203-204
````tablegen
def Tosa_TensorUpto4D : AnyTypeOf<[
  Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_AnyNumber], [0,1,2,3,4]>]>;
````
- **EN**: This TableGen block defines `Tosa_TensorUpto4D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TensorUpto4D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 206-207
````tablegen
def Tosa_IndexTensor1D : AnyTypeOf<[
  Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_Int32, Tosa_Int64], [1]>]>;
````
- **EN**: This TableGen block defines `Tosa_IndexTensor1D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IndexTensor1D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 208-209
````tablegen
def Tosa_IndexTensor2D : AnyTypeOf<[
  Tosa_UnrankedTensor, TosaTensorRankOf<[Tosa_Int32, Tosa_Int64], [2]>]>;
````
- **EN**: This TableGen block defines `Tosa_IndexTensor2D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IndexTensor2D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 211-212
````tablegen
def Tosa_TensorAtLeast1D : AnyTypeOf<[
  Tosa_UnrankedTensor, TosaRankedTensorOf<[Tosa_AnyNumber], [AtLeastRankOne]>], "tosa-conformant tensor of at least rank 1", "::mlir::TensorType">;
````
- **EN**: This TableGen block defines `Tosa_TensorAtLeast1D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TensorAtLeast1D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 214-217
````tablegen
def Tosa_MXFPDataTensor3D : AnyTypeOf<[
  TosaUnrankedTensorOf<[Tosa_MXFPNumber]>,
  TosaTensorRankOf<[Tosa_MXFPNumber], [3]>
]>;
````
- **EN**: This TableGen block defines `Tosa_MXFPDataTensor3D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPDataTensor3D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 218-221
````tablegen
def Tosa_MXFPScaleTensor3D : AnyTypeOf<[
  TosaUnrankedTensorOf<[Tosa_MXFPScaleNumber]>,
  TosaTensorRankOf<[Tosa_MXFPScaleNumber], [3]>
]>;
````
- **EN**: This TableGen block defines `Tosa_MXFPScaleTensor3D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPScaleTensor3D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 222-225
````tablegen
def Tosa_MXFPDataTensor4D : AnyTypeOf<[
  TosaUnrankedTensorOf<[Tosa_MXFPNumber]>,
  TosaTensorRankOf<[Tosa_MXFPNumber], [4]>
]>;
````
- **EN**: This TableGen block defines `Tosa_MXFPDataTensor4D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPDataTensor4D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 226-229
````tablegen
def Tosa_MXFPScaleTensor4D : AnyTypeOf<[
  TosaUnrankedTensorOf<[Tosa_MXFPScaleNumber]>,
  TosaTensorRankOf<[Tosa_MXFPScaleNumber], [4]>
]>;
````
- **EN**: This TableGen block defines `Tosa_MXFPScaleTensor4D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPScaleTensor4D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 230-234
````tablegen
def Tosa_MXFPDataTensorAtLeast1D : AnyTypeOf<[
  TosaUnrankedTensorOf<[Tosa_MXFPNumber]>,
  TosaRankedTensorOf<[Tosa_MXFPNumber], [AtLeastRankOne]>],
  "tosa-conformant tensor of at least rank 1", "::mlir::TensorType"
>;
````
- **EN**: This TableGen block defines `Tosa_MXFPDataTensorAtLeast1D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPDataTensorAtLeast1D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 235-243
````tablegen
def Tosa_MXFPScaleTensorAtLeast1D : AnyTypeOf<[
  TosaUnrankedTensorOf<[Tosa_MXFPScaleNumber]>,
  TosaRankedTensorOf<[Tosa_MXFPScaleNumber], [AtLeastRankOne]>],
  "tosa-conformant tensor of at least rank 1", "::mlir::TensorType"
>;

//===----------------------------------------------------------------------===//
// Generic scalar, vector, or tensor of a particular type.
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_MXFPScaleTensorAtLeast1D` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_MXFPScaleTensorAtLeast1D` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 245-249
````tablegen
class Tosa_TypeLike<list<Type> types, string description = ""> : TypeConstraint<Or<[
     AnyTypeOf<types>.predicate,
     VectorOfNonZeroRankOf<types>.predicate,
     TosaTensorOf<types>.predicate]>,
     description>;
````
- **EN**: This TableGen block defines `Tosa_TypeLike` as a `class` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TypeLike` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 251-251
````tablegen
def Tosa_IntLike : Tosa_TypeLike<[Tosa_Int], "signless-integer-like">;
````
- **EN**: This TableGen block defines `Tosa_IntLike` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntLike` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 252-256
````tablegen
def Tosa_Int8Like : Tosa_TypeLike<[Tosa_Int8], "signless-integer-8-bit-like">;

//===----------------------------------------------------------------------===//
// Attribute predicates and classes.
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_Int8Like` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Int8Like` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 258-258
````tablegen
def Tosa_Fp32ArrayAttr2 : ConfinedAttr<DenseF32ArrayAttr, [DenseArrayCount<2>]>;
````
- **EN**: This TableGen block defines `Tosa_Fp32ArrayAttr2` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Fp32ArrayAttr2` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 259-259
````tablegen
def Tosa_Fp32ArrayAttr3 : ConfinedAttr<DenseF32ArrayAttr, [DenseArrayCount<3>]>;
````
- **EN**: This TableGen block defines `Tosa_Fp32ArrayAttr3` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Fp32ArrayAttr3` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 260-260
````tablegen
def Tosa_Fp32ArrayAttr4 : ConfinedAttr<DenseF32ArrayAttr, [DenseArrayCount<4>]>;
````
- **EN**: This TableGen block defines `Tosa_Fp32ArrayAttr4` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Fp32ArrayAttr4` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 261-261
````tablegen
def Tosa_Fp32ArrayAttr5 : ConfinedAttr<DenseF32ArrayAttr, [DenseArrayCount<5>]>;
````
- **EN**: This TableGen block defines `Tosa_Fp32ArrayAttr5` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Fp32ArrayAttr5` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 262-262
````tablegen
def Tosa_Fp32ArrayAttr6 : ConfinedAttr<DenseF32ArrayAttr, [DenseArrayCount<6>]>;
````
- **EN**: This TableGen block defines `Tosa_Fp32ArrayAttr6` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Fp32ArrayAttr6` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 264-264
````tablegen
def Tosa_IntArrayAttr2 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayCount<2>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttr2` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttr2` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 265-265
````tablegen
def Tosa_IntArrayAttr3 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayCount<3>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttr3` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttr3` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 266-266
````tablegen
def Tosa_IntArrayAttr4 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayCount<4>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttr4` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttr4` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 267-267
````tablegen
def Tosa_IntArrayAttr5 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayCount<5>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttr5` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttr5` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 268-268
````tablegen
def Tosa_IntArrayAttr6 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayCount<6>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttr6` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttr6` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 270-270
````tablegen
def Tosa_IntArrayAttrUpto2 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayMaxCt<2>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttrUpto2` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttrUpto2` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 271-271
````tablegen
def Tosa_IntArrayAttrUpto4 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayMaxCt<4>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttrUpto4` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttrUpto4` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 272-272
````tablegen
def Tosa_IntArrayAttrUpto5 : ConfinedAttr<DenseI64ArrayAttr, [DenseArrayMaxCt<5>]>;
````
- **EN**: This TableGen block defines `Tosa_IntArrayAttrUpto5` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntArrayAttrUpto5` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 274-278
````tablegen
def Tosa_FloatAttr : Attr<CPred<"::llvm::isa<::mlir::FloatAttr>($_self)">,
                          "arbitrary float attribute"> {
  let storageType = [{ ::mlir::FloatAttr }];
  let returnType = [{ ::mlir::APFloat }];
}
````
- **EN**: This TableGen block defines `Tosa_FloatAttr` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_FloatAttr` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 280-284
````tablegen
def Tosa_IntegerAttr : Attr<CPred<"::llvm::isa<::mlir::IntegerAttr>($_self)">,
                          "arbitrary integer attribute"> {
  let storageType = [{ ::mlir::IntegerAttr }];
  let returnType = [{ ::llvm::APInt }];
}
````
- **EN**: This TableGen block defines `Tosa_IntegerAttr` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntegerAttr` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 286-286
````tablegen
def Tosa_IntOrFloatAttr : AnyAttrOf<[Tosa_IntegerAttr, Tosa_FloatAttr]>;
````
- **EN**: This TableGen block defines `Tosa_IntOrFloatAttr` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_IntOrFloatAttr` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 289-291
````tablegen
def Tosa_TensorTypeAttr : TypeAttrBase<"TensorType", "Tensor type attribute">;

// Tensor to buffer types.
````
- **EN**: This TableGen block defines `Tosa_TensorTypeAttr` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TensorTypeAttr` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 292-292
````tablegen
def Tosa_Buffer : MemRefOf<[Tosa_AnyNumber]>;
````
- **EN**: This TableGen block defines `Tosa_Buffer` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_Buffer` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 293-293
````tablegen
def Tosa_TupleBuffer : NestedTupleOf<[Tosa_Buffer]>;
````
- **EN**: This TableGen block defines `Tosa_TupleBuffer` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_TupleBuffer` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 294-298
````tablegen
def Tosa_BufOrTuple : AnyTypeOf<[Tosa_Buffer, Tosa_TupleBuffer]>;

//===----------------------------------------------------------------------===//
// ShapeType
//===----------------------------------------------------------------------===//
````
- **EN**: This TableGen block defines `Tosa_BufOrTuple` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Tosa_BufOrTuple` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 299-330
````tablegen
def Tosa_Shape : Tosa_Type<"shape", "shape"> {
  let summary = "Shape with static rank and Index element type";
  let description = [{
    Syntax:

    ``` shape - type :: = `shape` `<` rank `>`
    ``` Values with shape type represents a shape with a fixed rank and a list
                               of dimensions
                                   .Rank must be zero or a positive integer
                                   .Each dimension is represented by the builtin
                                       Index type.

    Examples:

    ```mlir
     // Shape with rank of four, for example, [1, 1, 8, 16]:
     !tosa
         .shape<4>

     // Shape with rank of one, for example, [16]:
     !tosa
         .shape<1>

     // Shape with rank zero, for example, [] (i.e., shape of scalar values):
     !tosa.shape<0>
    ```
  }];
  let parameters = (ins "int":$rank);
  let assemblyFormat = "`<` $rank `>`";

  let genVerifyDecl = 1;
}
````
- **EN**: This TableGen block defines `Tosa_Shape` as a `def` record for `TosaTypesBase`. It covers assembly syntax, semantic documentation, verification hooks.
- **CN**: 该 TableGen 代码块将 `Tosa_Shape` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。 其中涉及 汇编语法, 语义文档, 验证钩子。

### Lines 332-334
````tablegen
def IsTosaShapeType : CPred<"mlir::tosa::isa_tosa_shape_type($_self)">;

// Whether a Tosa Shape type has a rank equal to the specified rank.
````
- **EN**: This TableGen block defines `IsTosaShapeType` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `IsTosaShapeType` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 335-337
````tablegen
class IsTosaShapeOfRankPred<int rank> : And<[
  IsTosaShapeType,
  CPred<[{::llvm::cast<::mlir::tosa::shapeType>($_self).getRank() == }] # rank>
````
- **EN**: This TableGen block defines `IsTosaShapeOfRankPred` as a `class` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `IsTosaShapeOfRankPred` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 338-338
````tablegen
]>;
````
- **EN**: This section focuses on ]>;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“]>;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 340-341
````tablegen
class TosaShapeOfRank<int rank>
    : Type<IsTosaShapeOfRankPred<rank>, "Tosa shape type of rank " #rank>;
````
- **EN**: This TableGen block defines `TosaShapeOfRank` as a `class` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `TosaShapeOfRank` 定义为 `class` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 343-343
````tablegen
def Rank1TosaShape : TosaShapeOfRank<1>;
````
- **EN**: This TableGen block defines `Rank1TosaShape` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Rank1TosaShape` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 344-344
````tablegen
def Rank2TosaShape : TosaShapeOfRank<2>;
````
- **EN**: This TableGen block defines `Rank2TosaShape` as a `def` record for `TosaTypesBase`.
- **CN**: 该 TableGen 代码块将 `Rank2TosaShape` 定义为 `def` 记录，用于描述 `TosaTypesBase` 相关的声明式信息。

### Lines 345-347
````tablegen
def Rank4TosaShape : TosaShapeOfRank<4>;

#endif // TOSA_TYPES_BASE
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
- mlir/IR/OpBase.td
- mlir/Dialect/Tosa/IR/TosaOpBase.td
- Tosa_Int4 builds on I<4>;
- Tosa_Int8 builds on I<8>;
- Tosa_Int16 builds on I<16>;
- Tosa_Int32 builds on I<32>;
- Tosa_Int64 builds on I<64>;
- Tosa_Int builds on AnyTypeOf<[AnyUnsignedInteger,
- Tosa_Int32Or64 builds on AnyTypeOf<[Tosa_Int32,
- Tosa_Int16Or32 builds on AnyTypeOf<[Tosa_Int16,
- Tosa_QuantizedInt builds on AnyTypeOf<[Tosa_QuantizedType<"uint8", [8], 0>,
- Tosa_MXInt8 builds on Tosa_Type<"mxint8", "mxint8">
- Tosa_AnyNumber builds on AnyTypeOf<[Tosa_Int, Tosa_QuantizedInt, AnyFloat, Tosa_MXInt8],
- Tosa_MXFPNumber builds on AnyTypeOf<[F8E4M3FN, F8E5M2, F4E2M1FN, F6E2M3FN, F6E3M2FN, Tosa_MXInt8],
- Tosa_MXFPScaleNumber builds on AnyTypeOf<[F8E8M0FNU], "micro-scaling format scale number">;
- HasNo0Dimensions builds on And<[
- AllDimensionsAreSizeOne builds on And<[
- AtLeastRankOne builds on And<[
- Tosa_I1Tensor builds on TosaTensorOf<[I1]>;
