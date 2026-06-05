# SPIRVTosaTypes.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVTosaTypes.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR SPIRVTosaTypes component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This specifies Tosa types used by the Graph Extension and Tosa Ops.
- **用途（CN）**: 为 MLIR 的 SPIRVTosaTypes 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````tablegen
//===- SPIRVTosaTypes.td - Tosa Types insts spec file --------*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This specifies Tosa types used by the Graph Extension and Tosa Ops.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_TOSA_TYPES
#define MLIR_DIALECT_SPIRV_IR_TOSA_TYPES

include "mlir/IR/CommonAttrConstraints.td"
include "mlir/Dialect/SPIRV/IR/SPIRVBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 19-19
````tablegen
def SPIRV_I8OrI16 : AnyIntOfWidths<[8, 16]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 20-20
````tablegen
def SPIRV_I8OrI16OrI32 : AnyIntOfWidths<[8, 16, 32]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 21-21
````tablegen
def SPIRV_I8OrI16OrI32OrI64 : AnyIntOfWidths<[8, 16, 32, 64]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrI64` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrI64` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 22-22
````tablegen
def SPIRV_I16OrI32 : AnyIntOfWidths<[16, 32]>;
````
- **EN**: This TableGen block defines `SPIRV_I16OrI32` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I16OrI32` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 23-23
````tablegen
def SPIRV_I32OrI64 : AnyIntOfWidths<[32, 64]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 24-24
````tablegen
def SPIRV_F16OrF32OrBF16 : AnyTypeOf<[SPIRV_Float16, SPIRV_Float32, SPIRV_BFloat16KHR]>;
````
- **EN**: This TableGen block defines `SPIRV_F16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_F16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 25-25
````tablegen
def SPIRV_F16OrF32OrBF16OrFP8 : AnyTypeOf<[SPIRV_Float16, SPIRV_Float32, SPIRV_BFloat16KHR, SPIRV_Float8E4M3EXT, SPIRV_Float8E5M2EXT]>;
````
- **EN**: This TableGen block defines `SPIRV_F16OrF32OrBF16OrFP8` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_F16OrF32OrBF16OrFP8` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 26-26
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16 : AnyTypeOf<[SPIRV_I8OrI16, SPIRV_F16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 27-27
````tablegen
def SPIRV_I8OrF16OrF32OrBF16OrFP8 : AnyTypeOf<[SPIRV_Int8, SPIRV_F16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrF16OrF32OrBF16OrFP8` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrF16OrF32OrBF16OrFP8` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 28-28
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8 : AnyTypeOf<[SPIRV_I8OrI16, SPIRV_F16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 29-29
````tablegen
def SPIRV_I32OrF16OrF32OrBF16 : AnyTypeOf<[SPIRV_Int32, SPIRV_F16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrF16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrF16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 30-30
````tablegen
def SPIRV_I8OrI16OrI32OrF16OrF32OrBF16 : AnyTypeOf<[SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 31-31
````tablegen
def SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8 : AnyTypeOf<[SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 32-32
````tablegen
def SPIRV_I32OrI64OrF16OrF32OrBF16 : AnyTypeOf<[SPIRV_I32OrI64, SPIRV_F16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64OrF16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64OrF16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 33-33
````tablegen
def SPIRV_I32OrI64OrF16OrF32 : AnyTypeOf<[SPIRV_I32OrI64, SPIRV_Float16, SPIRV_Float32]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64OrF16OrF32` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64OrF16OrF32` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 34-34
````tablegen
def SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16 : AnyTypeOf<[SPIRV_I8OrI16OrI32OrI64, SPIRV_F16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 35-35
````tablegen
def SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16 : AnyTypeOf<[SPIRV_Bool, SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 36-36
````tablegen
def SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8 : AnyTypeOf<[SPIRV_Bool, SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 37-37
````tablegen
def SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8 : AnyTypeOf<[SPIRV_I8OrI16OrI32, SPIRV_Bool, SPIRV_F16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 38-38
````tablegen
def SPIRV_I8OrI32 : AnyTypeOf<[SPIRV_Int8, SPIRV_Int32]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI32` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI32` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 40-40
````tablegen
def SPIRV_TensorArmAxisAttr : ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<5>]>;
````
- **EN**: This TableGen block defines `SPIRV_TensorArmAxisAttr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_TensorArmAxisAttr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 41-41
````tablegen
def SPIRV_BoolConstAttr : ConfinedAttr<BoolAttr, []>;
````
- **EN**: This TableGen block defines `SPIRV_BoolConstAttr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_BoolConstAttr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 42-44
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16ConstAttr : AnyAttrOf<[I8Attr, I16Attr, F16Attr, F32Attr, BF16Attr]>;

// TensorARM Types
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16ConstAttr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16ConstAttr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 46-50
````tablegen
class RankedTensorArmOf<list<Type> allowedTypes, list<Pred> preds = [],
                     string summary = "ranked tensorArm">
  : ShapedContainerType<
      allowedTypes, And<!listconcat([SPIRV_IsTensorArmType], preds)>,
      summary, "::mlir::spirv::TensorArmType">;
````
- **EN**: This TableGen block defines `RankedTensorArmOf` as a `class` record for `SPIRVTosaTypes`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `RankedTensorArmOf` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 语义文档。

### Lines 52-55
````tablegen
class TensorArmRankOf<list<Type> allowedTypes, list<int> ranks>
  : RankedTensorArmOf<allowedTypes,
      [HasAnyRankOfPred<ranks>],
      !interleave(!foreach(rank, ranks, rank # "D"), "/") # " tensorArm">;
````
- **EN**: This TableGen block defines `TensorArmRankOf` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `TensorArmRankOf` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 57-57
````tablegen
def SPIRV_I8_TensorArm1D : TensorArmRankOf<[SPIRV_Int8], [1]>;
````
- **EN**: This TableGen block defines `SPIRV_I8_TensorArm1D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8_TensorArm1D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 58-58
````tablegen
def SPIRV_I16OrI32_TensorArm1D : TensorArmRankOf<[SPIRV_I16OrI32], [1]>;
````
- **EN**: This TableGen block defines `SPIRV_I16OrI32_TensorArm1D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I16OrI32_TensorArm1D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 59-59
````tablegen
def SPIRV_I32_TensorArm2D : TensorArmRankOf<[SPIRV_Int32], [2]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_TensorArm2D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_TensorArm2D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 60-60
````tablegen
def SPIRV_F32_TensorArm3D: TensorArmRankOf<[SPIRV_Float32], [3]>;
````
- **EN**: This TableGen block defines `SPIRV_F32_TensorArm3D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_F32_TensorArm3D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 61-61
````tablegen
def SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D : TensorArmRankOf<[SPIRV_I32OrI64OrF16OrF32OrBF16], [1]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm1D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 62-62
````tablegen
def SPIRV_I8OrI16_TensorArm1D : TensorArmRankOf<[SPIRV_I8OrI16], [1]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16_TensorArm1D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16_TensorArm1D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 63-63
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm3D : TensorArmRankOf<[SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8], [3]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm3D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm3D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 64-64
````tablegen
def SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D : TensorArmRankOf<[SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8], [3]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm3D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 65-65
````tablegen
def SPIRV_I32OrI64OrF16OrF32_TensorArm3D : TensorArmRankOf<[SPIRV_I32OrI64OrF16OrF32], [3]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64OrF16OrF32_TensorArm3D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64OrF16OrF32_TensorArm3D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 66-66
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm4D : TensorArmRankOf<[SPIRV_I8OrI16OrF16OrF32OrBF16], [4]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm4D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm4D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 67-67
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D : TensorArmRankOf<[SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8], [4]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm4D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 68-68
````tablegen
def SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm4D : TensorArmRankOf<[SPIRV_I8OrF16OrF32OrBF16OrFP8], [4]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm4D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm4D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 69-69
````tablegen
def SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm4D : TensorArmRankOf<[SPIRV_I32OrI64OrF16OrF32OrBF16], [4]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm4D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm4D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 70-70
````tablegen
def SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16_TensorArm4D : TensorArmRankOf<[SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16], [4]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16_TensorArm4D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16_TensorArm4D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 71-71
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm5D : TensorArmRankOf<[SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8], [5]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm5D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm5D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 72-72
````tablegen
def SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm5D : TensorArmRankOf<[SPIRV_I8OrF16OrF32OrBF16OrFP8], [5]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm5D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrF16OrF32OrBF16OrFP8_TensorArm5D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 73-73
````tablegen
def SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm5D : TensorArmRankOf<[SPIRV_I32OrI64OrF16OrF32OrBF16], [5]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm5D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrI64OrF16OrF32OrBF16_TensorArm5D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 75-75
````tablegen
def SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16_TensorArm : TensorArmRankOf<[SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 76-76
````tablegen
def SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm : TensorArmRankOf<[SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 77-77
````tablegen
def SPIRV_F16OrF32OrBF16_TensorArm : TensorArmRankOf<[SPIRV_F16OrF32OrBF16], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_F16OrF32OrBF16_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_F16OrF32OrBF16_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 78-78
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16OrF16OrF32OrBF16], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 79-79
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 80-80
````tablegen
def SPIRV_I32OrF16OrF32OrBF16_TensorArm : TensorArmRankOf<[SPIRV_I32OrF16OrF32OrBF16], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I32OrF16OrF32OrBF16_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32OrF16OrF32OrBF16_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 81-81
````tablegen
def SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrBoolOrF16OrF32OrBF16OrFP8_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 82-82
````tablegen
def SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16OrI32OrF16OrF32OrBF16], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 83-83
````tablegen
def SPIRV_I8OrI16OrI32OrI64_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16OrI32OrI64], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrI64_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrI64_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 84-84
````tablegen
def SPIRV_I8OrI16OrI32_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16OrI32], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 85-85
````tablegen
def SPIRV_I8OrI16_TensorArm : TensorArmRankOf<[SPIRV_I8OrI16], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 86-86
````tablegen
def SPIRV_I8OrI32_TensorArm : TensorArmRankOf<[SPIRV_I8OrI32], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI32_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI32_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 87-87
````tablegen
def SPIRV_Bool_TensorArm : TensorArmRankOf<[SPIRV_Bool], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_Bool_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_Bool_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 88-88
````tablegen
def SPIRV_I32_TensorArm : TensorArmRankOf<[SPIRV_Int32], [1, 2, 3, 4, 5, 6]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_TensorArm` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_TensorArm` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 89-89
````tablegen
def SPIRV_I32_TensorArmUpTo5D : TensorArmRankOf<[SPIRV_Int32], [1, 2, 3, 4, 5]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_TensorArmUpTo5D` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_TensorArmUpTo5D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 91-94
````tablegen
class Is1DTensorArmOfLength<list<int> allowedLengths> :
  And<[HasAnyRankOfPred<[1]>,
       Or<!foreach(allowedlength, allowedLengths,
                   CPred<[{::llvm::cast<::mlir::spirv::TensorArmType>($_self).getShape()[0] == }]
````
- **EN**: This TableGen block defines `Is1DTensorArmOfLength` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `Is1DTensorArmOfLength` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 95-95
````tablegen
                         # allowedlength>)>]>;
````
- **EN**: This section focuses on # allowedlength>)>]>;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“# allowedlength>)>]>;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 97-101
````tablegen
class SPIRV_1DTensorArmOfLengthAndType<list<int> allowedLengths, list<Type> allowedTypes> :
  ContainerType<AnyTypeOf<allowedTypes>, Is1DTensorArmOfLength<allowedLengths>,
    "::llvm::cast<::mlir::spirv::TensorArmType>($_self).getElementType()",
    "rank 1 tensorArm of length " # !interleave(allowedLengths, "/"),
    "::mlir::spirv::TensorArmType">;
````
- **EN**: This TableGen block defines `SPIRV_1DTensorArmOfLengthAndType` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_1DTensorArmOfLengthAndType` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 103-103
````tablegen
def SPIRV_I32_1DTensorArmOfLength2 : SPIRV_1DTensorArmOfLengthAndType<[2], [SPIRV_Int32]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength2` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength2` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 104-104
````tablegen
def SPIRV_I32_1DTensorArmOfLength4 : SPIRV_1DTensorArmOfLengthAndType<[4], [SPIRV_Int32]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength4` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength4` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 106-106
````tablegen
def SPIRV_I32_1DTensorArmOfLength1To6 : SPIRV_1DTensorArmOfLengthAndType<[1, 2, 3, 4, 5, 6], [SPIRV_Int32]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength1To6` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength1To6` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 107-107
````tablegen
def SPIRV_I32_1DTensorArmOfEvenLength2To12 : SPIRV_1DTensorArmOfLengthAndType<[2, 4, 6, 8, 10, 12], [SPIRV_Int32]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfEvenLength2To12` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfEvenLength2To12` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 109-111
````tablegen
def SPIRV_DenseElementAttrsWithTensorArmType : AttrConstraint<
  CPred<"::llvm::isa<::mlir::spirv::TensorArmType>(::llvm::cast<::mlir::DenseElementsAttr>($_self).getType())">,
  "Attr with type = spirv::TensorArmType">;
````
- **EN**: This TableGen block defines `SPIRV_DenseElementAttrsWithTensorArmType` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_DenseElementAttrsWithTensorArmType` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 113-113
````tablegen
def SPIRV_I32_1DTensorArmOfLength2Attr : ConfinedAttr<RankedI32ElementsAttr<[2]>, [SPIRV_DenseElementAttrsWithTensorArmType]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength2Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength2Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 114-114
````tablegen
def SPIRV_I32_1DTensorArmOfLength3Attr : ConfinedAttr<RankedI32ElementsAttr<[3]>, [SPIRV_DenseElementAttrsWithTensorArmType]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength3Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength3Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 115-115
````tablegen
def SPIRV_I32_1DTensorArmOfLength4Attr : ConfinedAttr<RankedI32ElementsAttr<[4]>, [SPIRV_DenseElementAttrsWithTensorArmType]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength4Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength4Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 116-116
````tablegen
def SPIRV_I32_1DTensorArmOfLength5Attr : ConfinedAttr<RankedI32ElementsAttr<[5]>, [SPIRV_DenseElementAttrsWithTensorArmType]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength5Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength5Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 117-117
````tablegen
def SPIRV_I32_1DTensorArmOfLength6Attr : ConfinedAttr<RankedI32ElementsAttr<[6]>, [SPIRV_DenseElementAttrsWithTensorArmType]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength6Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength6Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 118-121
````tablegen
class IntElementsAttrAllValuesAtLeast<int minValue> : AttrConstraint<
  CPred<"::llvm::all_of(::llvm::cast<::mlir::DenseElementsAttr>($_self).getValues<::llvm::APInt>(), "
        "[](const ::llvm::APInt &value) { return value.getSExtValue() >= " #
        minValue # "; })">,
````
- **EN**: This TableGen block defines `IntElementsAttrAllValuesAtLeast` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `IntElementsAttrAllValuesAtLeast` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 122-122
````tablegen
  "all values must be >= " # minValue>;
````
- **EN**: This section focuses on "all values must be >= " # minvalue>;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“"all values must be >= " # minValue>;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 124-124
````tablegen
def SPIRV_PositiveI32_1DTensorArmOfLength2Attr : ConfinedAttr<RankedI32ElementsAttr<[2]>, [SPIRV_DenseElementAttrsWithTensorArmType, IntElementsAttrAllValuesAtLeast<1>]>;
````
- **EN**: This TableGen block defines `SPIRV_PositiveI32_1DTensorArmOfLength2Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_PositiveI32_1DTensorArmOfLength2Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 125-125
````tablegen
def SPIRV_PositiveI32_1DTensorArmOfLength3Attr : ConfinedAttr<RankedI32ElementsAttr<[3]>, [SPIRV_DenseElementAttrsWithTensorArmType, IntElementsAttrAllValuesAtLeast<1>]>;
````
- **EN**: This TableGen block defines `SPIRV_PositiveI32_1DTensorArmOfLength3Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_PositiveI32_1DTensorArmOfLength3Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 126-126
````tablegen
def SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr : ConfinedAttr<RankedI32ElementsAttr<[4]>, [SPIRV_DenseElementAttrsWithTensorArmType, IntElementsAttrAllValuesAtLeast<0>]>;
````
- **EN**: This TableGen block defines `SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_NonNegativeI32_1DTensorArmOfLength4Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 127-127
````tablegen
def SPIRV_NonNegativeI32_1DTensorArmOfLength6Attr : ConfinedAttr<RankedI32ElementsAttr<[6]>, [SPIRV_DenseElementAttrsWithTensorArmType, IntElementsAttrAllValuesAtLeast<0>]>;
````
- **EN**: This TableGen block defines `SPIRV_NonNegativeI32_1DTensorArmOfLength6Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_NonNegativeI32_1DTensorArmOfLength6Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 129-130
````tablegen
class Is1DTensorArmAttrOfLength<list<int> allowedLengths> :
  AttrConstraint<And<[CPred<[{::llvm::cast<::mlir::spirv::TensorArmType>(::llvm::cast<::mlir::DenseElementsAttr>($_self).getType()).getShape().size() == 1 }]>,
````
- **EN**: This TableGen block defines `Is1DTensorArmAttrOfLength` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `Is1DTensorArmAttrOfLength` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 131-133
````tablegen
    Or<!foreach(allowedlength, allowedLengths,
      CPred<[{::llvm::cast<::mlir::spirv::TensorArmType>(::llvm::cast<::mlir::DenseElementsAttr>($_self).getType()).getShape()[0] == }]
              # allowedlength>)>]>>;
````
- **EN**: This block groups callable interfaces such as `foreach`, `getType`, `getShape`, indicating how `SPIRVTosaTypes` is queried or updated.
- **CN**: 该代码块聚合了 `foreach`, `getType`, `getShape` 等可调用接口，展示了如何查询或更新 `SPIRVTosaTypes`。

### Lines 135-136
````tablegen
def SPIRV_I32_1DTensorArmOfLength1To6Attr : ConfinedAttr<
  I32ElementsAttr, [SPIRV_DenseElementAttrsWithTensorArmType, Is1DTensorArmAttrOfLength<[1, 2, 3, 4, 5, 6]>]>;
````
- **EN**: This TableGen block defines `SPIRV_I32_1DTensorArmOfLength1To6Attr` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I32_1DTensorArmOfLength1To6Attr` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 138-138
````tablegen
def SPIRV_I8_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_Int8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 139-139
````tablegen
def SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 140-140
````tablegen
def SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_I8OrI16OrI32OrF16OrF32OrBF16]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrF16OrF32OrBF16_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 141-141
````tablegen
def SPIRV_I8OrI16OrI32OrI64_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_I8OrI16OrI32OrI64]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32OrI64_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32OrI64_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 142-142
````tablegen
def SPIRV_I8OrI16OrI32_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_I8OrI16OrI32]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrI16OrI32_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrI16OrI32_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 143-143
````tablegen
def SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_I8OrF16OrF32OrBF16OrFP8]>;
````
- **EN**: This TableGen block defines `SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_I8OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 144-146
````tablegen
def SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1 : SPIRV_1DTensorArmOfLengthAndType<[1], [SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8]>;

// Struct type
````
- **EN**: This TableGen block defines `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1` as a `def` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8_1DTensorArmOfLength1` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 148-151
````tablegen
class IsStructOfNumElementsPred<int numElements> :
  And<[SPIRV_IsStructType,
      CPred<[{::llvm::cast<::mlir::spirv::StructType>($_self).getNumElements()
              == }]
````
- **EN**: This TableGen block defines `IsStructOfNumElementsPred` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `IsStructOfNumElementsPred` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 152-152
````tablegen
            # numElements>]>;
````
- **EN**: This section focuses on # numelements>]>;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“# numElements>]>;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 154-157
````tablegen
class IsStructOfNumElementsAndType<int numElements, list<Type> allowedTypes>
    : MixedContainerType<AnyTypeOf<allowedTypes>, IsStructOfNumElementsPred<numElements>,
                         "::llvm::cast<::mlir::spirv::StructType>($_self).getElementTypes()",
                         "Struct">;
````
- **EN**: This TableGen block defines `IsStructOfNumElementsAndType` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `IsStructOfNumElementsAndType` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 159-161
````tablegen
def SPIRV_Struct_2_F32_TensorArm3D : IsStructOfNumElementsAndType<2, [SPIRV_F32_TensorArm3D]>;

// Op Trait constraints:
````
- **EN**: This TableGen block defines `SPIRV_Struct_2_F32_TensorArm3D` as a `def` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `SPIRV_Struct_2_F32_TensorArm3D` 定义为 `def` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 163-163
````tablegen
class Implies<Pred left, list<Pred> right>: Or<[Neg<left>, Or<right>]>;
````
- **EN**: This TableGen block defines `Implies` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `Implies` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 165-170
````tablegen
class TypeConstraintImplicationOn<string name, Type type, string other, list<Type> allowedTypes>:
  PredOpTrait<"if " # name # " has type " # type.summary # " then " #
              other # " must have a type in [" #
              !interleave(!foreach(type, allowedTypes, type.summary), ",") # "]",
    Implies<ElementTypeIsPred<name, type>,
    !foreach(allowedType, allowedTypes, ElementTypeIsPred<other, allowedType>)>>;
````
- **EN**: This TableGen block defines `TypeConstraintImplicationOn` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TypeConstraintImplicationOn` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 172-177
````tablegen
class BoolAttrTypeConstraintImplicationOn<string boolAttr, string other, list<Type> allowedTypes>:
  PredOpTrait<"if " # boolAttr # " is true then " #
              other # " must have a type in [" #
              !interleave(!foreach(type, allowedTypes, type.summary), ",") # "]",
    Implies<CPred<"get" # snakeCaseToCamelCase<boolAttr>.ret # "()" >,
    !foreach(allowedType, allowedTypes, ElementTypeIsPred<other, allowedType>)>>;
````
- **EN**: This TableGen block defines `BoolAttrTypeConstraintImplicationOn` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `BoolAttrTypeConstraintImplicationOn` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 179-181
````tablegen
class AxisValueLessThanRankOf<string input>:
PredOpTrait<"axis attribute value should be lower than rank(" # input # ")",
  Implies<CPred<HasRank<input>.result>, [CPred<"getAxis() < " # Rank<input>.result>]>>;
````
- **EN**: This TableGen block defines `AxisValueLessThanRankOf` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `AxisValueLessThanRankOf` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 183-187
````tablegen
class OutputRankIsInputRankMinusOne<string input, string output>:
PredOpTrait<output # " rank must be equal to max(1, rank(" # input # "))",
  Implies<
      And<[CPred<HasRank<input>.result>, CPred<HasRank<input>.result>]>,
      [CPred<"std::max(int64_t(1), " # Rank<input>.result # " - int64_t(1)) == " # Rank<output>.result>]>>;
````
- **EN**: This TableGen block defines `OutputRankIsInputRankMinusOne` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `OutputRankIsInputRankMinusOne` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 189-190
````tablegen
class DimOfType<string type, int dim> :
  StrFunc<type # ".getDimSize(" # dim # ")">;
````
- **EN**: This TableGen block defines `DimOfType` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimOfType` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 192-193
````tablegen
class ShapedTypeOf<string input> :
  StrFunc<"::llvm::cast<::mlir::ShapedType>($" # input # ".getType())">;
````
- **EN**: This TableGen block defines `ShapedTypeOf` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `ShapedTypeOf` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 195-196
````tablegen
class DimOf<string input, int dim> :
  DimOfType<ShapedTypeOf<input>.result, dim>;
````
- **EN**: This TableGen block defines `DimOf` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimOf` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 198-199
````tablegen
class DimIsDynamic<string input, int dim> :
  CPred<"::mlir::ShapedType::isDynamic(" # DimOf<input, dim>.result # ")">;
````
- **EN**: This TableGen block defines `DimIsDynamic` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimIsDynamic` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 201-203
````tablegen
class DimOfTypeIsDynamic<string type, int dim> :
  CPred<"::mlir::ShapedType::isDynamic(" #
        DimOfType<type, dim>.result # ")">;
````
- **EN**: This TableGen block defines `DimOfTypeIsDynamic` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimOfTypeIsDynamic` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 205-206
````tablegen
class DimIsOne<string input, int dim> :
  CPred<DimOf<input, dim>.result # " == 1">;
````
- **EN**: This TableGen block defines `DimIsOne` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimIsOne` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 208-209
````tablegen
class DimsMatch<string lhs, int lhsDim, string rhs, int rhsDim> :
  CPred<DimOf<lhs, lhsDim>.result # " == " # DimOf<rhs, rhsDim>.result>;
````
- **EN**: This TableGen block defines `DimsMatch` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimsMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 211-214
````tablegen
class DimsOfTypeMatch<string lhsType, int lhsDim, string rhsType,
                      int rhsDim> :
  CPred<DimOfType<lhsType, lhsDim>.result # " == " #
        DimOfType<rhsType, rhsDim>.result>;
````
- **EN**: This TableGen block defines `DimsOfTypeMatch` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimsOfTypeMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 216-219
````tablegen
class ProductDimsMatch<string lhs, int lhsDim, string rhs, int rhsDim,
                       string output, int outputDim> :
  CPred<DimOf<lhs, lhsDim>.result # " * " # DimOf<rhs, rhsDim>.result #
        " == " # DimOf<output, outputDim>.result>;
````
- **EN**: This TableGen block defines `ProductDimsMatch` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `ProductDimsMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 221-226
````tablegen
class SameDimsOrDynamicPred<string lhs, int lhsDim, string rhs, int rhsDim> :
  Or<[
    DimIsDynamic<lhs, lhsDim>,
    DimIsDynamic<rhs, rhsDim>,
    DimsMatch<lhs, lhsDim, rhs, rhsDim>
  ]>;
````
- **EN**: This TableGen block defines `SameDimsOrDynamicPred` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SameDimsOrDynamicPred` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 228-232
````tablegen
class SameDimsOrOneOrDynamicPred<string lhs, int lhsDim, string rhs, int rhsDim> :
  Or<[
    SameDimsOrDynamicPred<lhs, lhsDim, rhs, rhsDim>,
    DimIsOne<lhs, lhsDim>
  ]>;
````
- **EN**: This TableGen block defines `SameDimsOrOneOrDynamicPred` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `SameDimsOrOneOrDynamicPred` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 234-242
````tablegen
class ProductDimOrOneOrDynamicPred<string lhs, int lhsDim, string rhs,
                                   int rhsDim, string output, int outputDim> :
  Or<[
    DimIsDynamic<lhs, lhsDim>,
    DimIsDynamic<rhs, rhsDim>,
    DimIsDynamic<output, outputDim>,
    ProductDimsMatch<lhs, lhsDim, rhs, rhsDim, output, outputDim>,
    DimIsOne<output, outputDim>
  ]>;
````
- **EN**: This TableGen block defines `ProductDimOrOneOrDynamicPred` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `ProductDimOrOneOrDynamicPred` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 244-246
````tablegen
class AttrIsFloat<string attr> :
  CPred<"::llvm::isa<::mlir::FloatAttr>(get" #
        snakeCaseToCamelCase<attr>.ret # "())">;
````
- **EN**: This TableGen block defines `AttrIsFloat` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `AttrIsFloat` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 248-250
````tablegen
class AttrIsInteger<string attr> :
  CPred<"::llvm::isa<::mlir::IntegerAttr>(get" #
        snakeCaseToCamelCase<attr>.ret # "())">;
````
- **EN**: This TableGen block defines `AttrIsInteger` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `AttrIsInteger` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 252-254
````tablegen
class IntegerAttrValue<string attr> :
  StrFunc<"::llvm::cast<::mlir::IntegerAttr>(get" #
          snakeCaseToCamelCase<attr>.ret # "()).getValue()">;
````
- **EN**: This TableGen block defines `IntegerAttrValue` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `IntegerAttrValue` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 256-258
````tablegen
class FloatAttrValue<string attr> :
  StrFunc<"::llvm::cast<::mlir::FloatAttr>(get" #
          snakeCaseToCamelCase<attr>.ret # "()).getValue()">;
````
- **EN**: This TableGen block defines `FloatAttrValue` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `FloatAttrValue` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 260-262
````tablegen
class FloatAttrIsNaN<string attr> :
  And<[AttrIsFloat<attr>,
       CPred<FloatAttrValue<attr>.result # ".isNaN()">]>;
````
- **EN**: This TableGen block defines `FloatAttrIsNaN` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `FloatAttrIsNaN` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 264-270
````tablegen
class ReductionDimMatchPred<string input, string output, int dim> :
  Or<[
    DimIsDynamic<input, dim>,
    DimIsDynamic<output, dim>,
    And<[CPred<"getAxis() == " # dim>, DimIsOne<output, dim>]>,
    And<[CPred<"getAxis() != " # dim>, DimsMatch<input, dim, output, dim>]>
  ]>;
````
- **EN**: This TableGen block defines `ReductionDimMatchPred` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `ReductionDimMatchPred` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 272-281
````tablegen
class ReductionOutputShapeMatchesInputAndAxis<string input, string output>:
PredOpTrait<"shape of " # output # " must match the shape of " # input #
                " with the dimension at axis replaced by 1",
  And<!foreach(dim, [0, 1, 2, 3, 4, 5],
               Implies<
                 And<[CPred<HasRank<input>.result>,
                      CPred<HasRank<output>.result>,
                      CPred<Rank<input>.result # " == " # Rank<output>.result>,
                      CPred<Rank<input>.result # " > " # dim>]>,
                 [ReductionDimMatchPred<input, output, dim>]>)>>;
````
- **EN**: This TableGen block defines `ReductionOutputShapeMatchesInputAndAxis` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ReductionOutputShapeMatchesInputAndAxis` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 283-284
````tablegen
class AccTypeIn<list<string> allowedValues> :
  CPred<"llvm::is_contained({" # !interleave(!foreach(value, allowedValues, "::mlir::spirv::TosaExtAccType::" # value), ",") # "}, getAccType())">;
````
- **EN**: This TableGen block defines `AccTypeIn` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `AccTypeIn` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 286-288
````tablegen
class TypeImpliesAccType<string input, Type type, list<string> allowedAccTypes>:
  PredOpTrait<"acc_type must be one in [" # !interleave(allowedAccTypes, ",") # "] when type has value " # type.summary,
  Implies<ElementTypeIsPred<input, type>, [AccTypeIn<allowedAccTypes>]>>;
````
- **EN**: This TableGen block defines `TypeImpliesAccType` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TypeImpliesAccType` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 290-300
````tablegen
class MatchBroadcastableShapes<string input1, string input2, string output>:
  PredOpTrait<"the shape of " # input1 # " and " # input2 # " are compatible for broadcasting and the broadcast shape is equal to the " # output # " shape",
  Implies<And<[CPred<HasRank<input1>.result>, CPred<HasRank<input2>.result>, CPred<HasRank<output>.result>,
               CPred<Rank<input1>.result # " == " # Rank<input2>.result # " && " # Rank<input1>.result # " == " # Rank<output>.result>]>,
    [CPred<"llvm::all_of_zip(" # Shape<input1>.result # ", " # Shape<input2>.result # ", " # Shape<output>.result # ", " #
    "[](int64_t input1Dim, int64_t input2Dim, int64_t outputDim) { " #
    "   bool dynamic = ShapedType::isDynamic(input1Dim) || ShapedType::isDynamic(input2Dim) || ShapedType::isDynamic(outputDim);"
    "   bool broadcastableInputs = input1Dim == input2Dim || input1Dim == 1 || input2Dim == 1;" #
    "   bool broacastDimMatchesOutputDim = std::max(input1Dim, input2Dim) == outputDim;"
    "   return dynamic || (broadcastableInputs && broacastDimMatchesOutputDim);" #
    "})">]>
````
- **EN**: This TableGen block defines `MatchBroadcastableShapes` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `MatchBroadcastableShapes` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 301-301
````tablegen
  >;
````
- **EN**: This section focuses on >;, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“>;”这一主题，把相关声明与辅助接口组织在一起。

### Lines 303-314
````tablegen
class ValuesIndicesShapesMatch<string values, string indices, string tensor>:
  PredOpTrait<"shapes of " # values # ", " # indices # ", and " # tensor #
                  " must satisfy [N,K,C], [N,W], [N,W,C]",
    And<[
      SameDimsOrDynamicPred<values, 0, indices, 0>,
      SameDimsOrDynamicPred<values, 0, tensor, 0>,
      SameDimsOrDynamicPred<indices, 0, tensor, 0>,
      SameDimsOrDynamicPred<indices, 1, tensor, 1>,
      SameDimsOrDynamicPred<values, 2, tensor, 2>
    ]>>;

// The tensor shapes are [N,H,W,C] where N,H,W,C are the dimension values.
````
- **EN**: This TableGen block defines `ValuesIndicesShapesMatch` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ValuesIndicesShapesMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 315-321
````tablegen
class NHWCInputOutputShapeMatch<string input, string output>:
  PredOpTrait<"shapes of " # input # " and " # output #
                  " must satisfy [N,*,*,C] and [N,*,*,C]",
    And<[
      SameDimsOrDynamicPred<input, 0, output, 0>,
      SameDimsOrDynamicPred<input, 3, output, 3>
    ]>>;
````
- **EN**: This TableGen block defines `NHWCInputOutputShapeMatch` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `NHWCInputOutputShapeMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 324-333
````tablegen
class Conv2DShapeMatch<string input, string weight, string bias, string output>:
  PredOpTrait<"shapes of " # input # ", " # weight # ", " # bias # ", and " #
                  output # " must satisfy [N,IH,IW,IC], [OC,KH,KW,IC], "
                  "[OC/1], [N,OH,OW,OC]",
    And<[
      SameDimsOrDynamicPred<input, 0, output, 0>,
      SameDimsOrDynamicPred<input, 3, weight, 3>,
      SameDimsOrDynamicPred<weight, 0, output, 3>,
      SameDimsOrOneOrDynamicPred<bias, 0, output, 3>
    ]>>;
````
- **EN**: This TableGen block defines `Conv2DShapeMatch` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Conv2DShapeMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 335-344
````tablegen
class Conv3DShapeMatch<string input, string weight, string bias, string output>:
  PredOpTrait<"shapes of " # input # ", " # weight # ", " # bias # ", and " #
                  output # " must satisfy [N,ID,IH,IW,IC], [OC,KD,KH,KW,IC], "
                  "[OC/1], [N,OD,OH,OW,OC]",
    And<[
      SameDimsOrDynamicPred<input, 0, output, 0>,
      SameDimsOrDynamicPred<input, 4, weight, 4>,
      SameDimsOrDynamicPred<weight, 0, output, 4>,
      SameDimsOrOneOrDynamicPred<bias, 0, output, 4>
    ]>>;
````
- **EN**: This TableGen block defines `Conv3DShapeMatch` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Conv3DShapeMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 346-356
````tablegen
class DepthwiseConv2DShapeMatch<string input, string weight, string bias,
                                string output>:
  PredOpTrait<"shapes of " # input # ", " # weight # ", " # bias # ", and " #
                  output # " must satisfy [N,IH,IW,IC], [KH,KW,IC,M], "
                  "[IC*M/1], [N,OH,OW,IC*M]",
    And<[
      SameDimsOrDynamicPred<input, 0, output, 0>,
      SameDimsOrDynamicPred<input, 3, weight, 2>,
      ProductDimOrOneOrDynamicPred<input, 3, weight, 3, output, 3>,
      SameDimsOrOneOrDynamicPred<bias, 0, output, 3>
    ]>>;
````
- **EN**: This TableGen block defines `DepthwiseConv2DShapeMatch` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `DepthwiseConv2DShapeMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 358-359
````tablegen
class FetchNthIntElementsAttr<string attrName, int idx> :
  StrFunc<"get" # snakeCaseToCamelCase<attrName>.ret # "().getValues<APInt>()[" # idx # "].getSExtValue()">;
````
- **EN**: This TableGen block defines `FetchNthIntElementsAttr` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `FetchNthIntElementsAttr` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 361-362
````tablegen
class ElementsAttrValueLessThan<string leftAttrName, int leftIdx,string rightAttrName, int rightIdx> :
   CPred<FetchNthIntElementsAttr<leftAttrName, leftIdx>.result # " < " # FetchNthIntElementsAttr<rightAttrName, rightIdx>.result>;
````
- **EN**: This TableGen block defines `ElementsAttrValueLessThan` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `ElementsAttrValueLessThan` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 364-370
````tablegen
class Pool2DPadValuesLessThanKernel<string padAttr, string kernelAttr> :
    PredOpTrait<"op pad values must satisfy pad_top/pad_bottom < kernel_y and pad_left/pad_right < kernel_x",
    And<[ElementsAttrValueLessThan<padAttr, 0, kernelAttr, 0>,
         ElementsAttrValueLessThan<padAttr, 1, kernelAttr, 0>,
         ElementsAttrValueLessThan<padAttr, 2, kernelAttr, 1>,
         ElementsAttrValueLessThan<padAttr, 3, kernelAttr, 1>]
    >>;
````
- **EN**: This TableGen block defines `Pool2DPadValuesLessThanKernel` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Pool2DPadValuesLessThanKernel` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 372-381
````tablegen
class MatMulShapesMatch<string lhs, string rhs, string output> : PredOpTrait<
  "shapes of " # lhs # ", " # rhs # ", and " # output #
      " must satisfy [N,H,C], [N,C,W], [N,H,W]",
  And<[
    SameDimsOrDynamicPred<lhs, 0, rhs, 0>,
    SameDimsOrDynamicPred<lhs, 0, output, 0>,
    SameDimsOrDynamicPred<lhs, 1, output, 1>,
    SameDimsOrDynamicPred<lhs, 2, rhs, 1>,
    SameDimsOrDynamicPred<rhs, 2, output, 2>
  ]>>;
````
- **EN**: This TableGen block defines `MatMulShapesMatch` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `MatMulShapesMatch` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 383-391
````tablegen
class DimsOfTypeAllMatchOrDynamic<string lhsType, string rhsType,
                                  string otherType, int dim> :
  Or<[
    DimOfTypeIsDynamic<lhsType, dim>,
    DimOfTypeIsDynamic<rhsType, dim>,
    DimOfTypeIsDynamic<otherType, dim>,
    And<[DimsOfTypeMatch<lhsType, dim, rhsType, dim>,
         DimsOfTypeMatch<lhsType, dim, otherType, dim>]>
  ]>;
````
- **EN**: This TableGen block defines `DimsOfTypeAllMatchOrDynamic` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimsOfTypeAllMatchOrDynamic` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 393-400
````tablegen
class DimOfTypeMatchesHalfPlusOne<string inputType, int inputDim,
                                  string outputType, int outputDim> :
  Or<[
    DimOfTypeIsDynamic<inputType, inputDim>,
    DimOfTypeIsDynamic<outputType, outputDim>,
    CPred<DimOfType<outputType, outputDim>.result # " == " #
          DimOfType<inputType, inputDim>.result # " / 2 + 1">
  ]>;
````
- **EN**: This TableGen block defines `DimOfTypeMatchesHalfPlusOne` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimOfTypeMatchesHalfPlusOne` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 402-407
````tablegen
class Struct2TensorArmShapesMatchInput<string input> : PredOpTrait<
  "shapes of output_real and output_imag must match the shape of " # input,
  And<!foreach(dim, [0, 1, 2],
               DimsOfTypeAllMatchOrDynamic<
                 ShapedTypeOf<input>.result,
                 "getResultRealType()", "getResultImagType()", dim>)>>;
````
- **EN**: This TableGen block defines `Struct2TensorArmShapesMatchInput` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Struct2TensorArmShapesMatchInput` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 409-421
````tablegen
class Struct2TensorArmRFFT2DShapesMatchInput<string input> : PredOpTrait<
  "shapes of output_real and output_imag must satisfy [N,H,W/2 + 1] when " #
      input # " has shape [N,H,W]",
  And<[
    DimsOfTypeAllMatchOrDynamic<ShapedTypeOf<input>.result,
      "getResultRealType()", "getResultImagType()", 0>,
    DimsOfTypeAllMatchOrDynamic<ShapedTypeOf<input>.result,
      "getResultRealType()", "getResultImagType()", 1>,
    DimOfTypeMatchesHalfPlusOne<ShapedTypeOf<input>.result,
      2, "getResultRealType()", 2>,
    DimOfTypeMatchesHalfPlusOne<ShapedTypeOf<input>.result,
      2, "getResultImagType()", 2>
  ]>>;
````
- **EN**: This TableGen block defines `Struct2TensorArmRFFT2DShapesMatchInput` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `Struct2TensorArmRFFT2DShapesMatchInput` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 423-425
````tablegen
class DimIsPowerOfTwo<string input, int dim> :
  Or<[DimIsDynamic<input, dim>,
      CPred<"llvm::isPowerOf2_64(static_cast<uint64_t>(" # DimOf<input, dim>.result # "))">]>;
````
- **EN**: This TableGen block defines `DimIsPowerOfTwo` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `DimIsPowerOfTwo` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 427-429
````tablegen
class TensorDimsArePowerOfTwo<string input, list<int> dims> : PredOpTrait<
  "dimensions " # !interleave(dims, ", ") # " of " # input # " must be powers of two",
  And<!foreach(dim, dims, DimIsPowerOfTwo<input, dim>)>>;
````
- **EN**: This TableGen block defines `TensorDimsArePowerOfTwo` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TensorDimsArePowerOfTwo` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 431-433
````tablegen
class ClampBoundsAreNotNaN<string minVal, string maxVal>:
  PredOpTrait<minVal # " and " # maxVal # " must not be NaN",
    Neg<Or<[FloatAttrIsNaN<minVal>, FloatAttrIsNaN<maxVal>]>>>;
````
- **EN**: This TableGen block defines `ClampBoundsAreNotNaN` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ClampBoundsAreNotNaN` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 435-438
````tablegen
class IntegerAttrLessThanOrEqual<string minVal, string maxVal> :
  Implies<AttrIsInteger<minVal>,
          [CPred<IntegerAttrValue<minVal>.result # ".sle(" #
                 IntegerAttrValue<maxVal>.result # ")">]>;
````
- **EN**: This TableGen block defines `IntegerAttrLessThanOrEqual` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `IntegerAttrLessThanOrEqual` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 440-444
````tablegen
class FloatAttrLessThanOrEqual<string minVal, string maxVal> :
  Implies<AttrIsFloat<minVal>,
          [CPred<FloatAttrValue<minVal>.result # ".compare(" #
                 FloatAttrValue<maxVal>.result #
                 ") != ::llvm::APFloat::cmpGreaterThan">]>;
````
- **EN**: This TableGen block defines `FloatAttrLessThanOrEqual` as a `class` record for `SPIRVTosaTypes`.
- **CN**: 该 TableGen 代码块将 `FloatAttrLessThanOrEqual` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。

### Lines 446-449
````tablegen
class ClampMinValLessThanOrEqualToMaxVal<string minVal, string maxVal>:
  PredOpTrait<minVal # " must be <= " # maxVal,
    And<[IntegerAttrLessThanOrEqual<minVal, maxVal>,
         FloatAttrLessThanOrEqual<minVal, maxVal>]>>;
````
- **EN**: This TableGen block defines `ClampMinValLessThanOrEqualToMaxVal` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ClampMinValLessThanOrEqualToMaxVal` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 451-454
````tablegen
class TableSizeConstraint<string input, Type type, int size>:
  PredOpTrait<"table must have size " # size # " if " # input # " has element type " # type.summary,
      Implies<ElementTypeIsPred<input, type>, [CPred<"::llvm::cast<::mlir::ShapedType>(getTable().getType()).getShape()[0] == " # size>]>
    >;
````
- **EN**: This TableGen block defines `TableSizeConstraint` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TableSizeConstraint` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 456-460
````tablegen
class ShapeConstraintFromInputRank<string input, string other, int mul=1>:
  PredOpTrait< "the number of elements of " # other # " must be rank(" # input # ")" # !if(!eq(mul, 1), "", " * " # mul),
      Implies<CPred<HasRank<input>.result>,
        [CPred<ElementCount<other>.result # " == " # mul # " * " # Rank<input>.result>]>
    >;
````
- **EN**: This TableGen block defines `ShapeConstraintFromInputRank` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ShapeConstraintFromInputRank` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 462-464
````tablegen
class VariadicInputWithMinSize<string input, int min_size>:
    PredOpTrait<"variadic " # input # " must has at least " # min_size # " elements",
      CPred<"static_cast<int64_t>($" # input # ".getTypes().size()) >= " # min_size>>;
````
- **EN**: This TableGen block defines `VariadicInputWithMinSize` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `VariadicInputWithMinSize` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 466-470
````tablegen
class VariadicInputAllSameElementType<string reference, string input>:
    PredOpTrait<"all elements of variadic " # input # " must have same element type",
      CPred<"::llvm::all_of($" # input # ".getTypes(), "
                     "[&](::mlir::Type t) { return ::llvm::cast<::mlir::ShapedType>(t).getElementType() == "
                     # ElementType<reference>.result # "; })">>;
````
- **EN**: This TableGen block defines `VariadicInputAllSameElementType` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `VariadicInputAllSameElementType` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 472-478
````tablegen
class VariadicInputAllSameRank<string reference, string input>:
    PredOpTrait<"all elements of variadic " # input # " must have same element type",
      CPred<"::llvm::all_of($" # input # ".getTypes(), "
                     "[&](::mlir::Type t) { return ::llvm::cast<::mlir::ShapedType>(t).hasRank() && "
                     # HasRank<reference>.result #
                     " && ::llvm::cast<::mlir::ShapedType>(t).getRank() == "
                     # Rank<reference>.result # "; })">>;
````
- **EN**: This TableGen block defines `VariadicInputAllSameRank` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `VariadicInputAllSameRank` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 480-483
````tablegen
class ElementTypeMatchesScale32<string tensor> :
  PredOpTrait<tensor # " must have element type i32 when scale32 is true, otherwise i16",
    CPred<"::llvm::cast<::mlir::ShapedType>($" # tensor # ".getType()).getElementType()."
          "isInteger(getScale32() ? 32 : 16)">>;
````
- **EN**: This TableGen block defines `ElementTypeMatchesScale32` as a `class` record for `SPIRVTosaTypes`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ElementTypeMatchesScale32` 定义为 `class` 记录，用于描述 `SPIRVTosaTypes` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 485-492
````tablegen
class TensorLengthMatchesPerChannel<string tensor> :
  PredOpTrait<tensor # " must have length input_shape[rank(input) - 1] when per_channel is true, otherwise length 1",
    CPred<"::llvm::cast<::mlir::ShapedType>($" # tensor # ".getType()).getShape()[0] == "
          "(getPerChannel() ? "
          "::llvm::cast<::mlir::ShapedType>($input.getType()).getShape().back() : 1)">>;


#endif // MLIR_DIALECT_SPIRV_IR_TOSA_TYPES
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/CommonAttrConstraints.td
- mlir/Dialect/SPIRV/IR/SPIRVBase.td
- SPIRV_I8OrI16 builds on AnyIntOfWidths<[8, 16]>;
- SPIRV_I8OrI16OrI32 builds on AnyIntOfWidths<[8, 16, 32]>;
- SPIRV_I8OrI16OrI32OrI64 builds on AnyIntOfWidths<[8, 16, 32, 64]>;
- SPIRV_I16OrI32 builds on AnyIntOfWidths<[16, 32]>;
- SPIRV_I32OrI64 builds on AnyIntOfWidths<[32, 64]>;
- SPIRV_F16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_Float16, SPIRV_Float32, SPIRV_BFloat16KHR]>;
- SPIRV_F16OrF32OrBF16OrFP8 builds on AnyTypeOf<[SPIRV_Float16, SPIRV_Float32, SPIRV_BFloat16KHR, SPIRV_Float8E4M3EXT, SPIRV_Float8E5M2EXT]>;
- SPIRV_I8OrI16OrF16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_I8OrI16, SPIRV_F16OrF32OrBF16]>;
- SPIRV_I8OrF16OrF32OrBF16OrFP8 builds on AnyTypeOf<[SPIRV_Int8, SPIRV_F16OrF32OrBF16OrFP8]>;
- SPIRV_I8OrI16OrF16OrF32OrBF16OrFP8 builds on AnyTypeOf<[SPIRV_I8OrI16, SPIRV_F16OrF32OrBF16OrFP8]>;
- SPIRV_I32OrF16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_Int32, SPIRV_F16OrF32OrBF16]>;
- SPIRV_I8OrI16OrI32OrF16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16]>;
- SPIRV_I8OrI16OrI32OrF16OrF32OrBF16OrFP8 builds on AnyTypeOf<[SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16OrFP8]>;
- SPIRV_I32OrI64OrF16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_I32OrI64, SPIRV_F16OrF32OrBF16]>;
- SPIRV_I32OrI64OrF16OrF32 builds on AnyTypeOf<[SPIRV_I32OrI64, SPIRV_Float16, SPIRV_Float32]>;
- SPIRV_I32OrI8OrI64OrI16OrF16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_I8OrI16OrI32OrI64, SPIRV_F16OrF32OrBF16]>;
- SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16 builds on AnyTypeOf<[SPIRV_Bool, SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16]>;
- SPIRV_BoolOrI8OrI16OrI32OrF16OrF32OrBF16OrFP8 builds on AnyTypeOf<[SPIRV_Bool, SPIRV_I8OrI16OrI32, SPIRV_F16OrF32OrBF16OrFP8]>;
