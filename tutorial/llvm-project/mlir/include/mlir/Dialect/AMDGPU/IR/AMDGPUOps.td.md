# AMDGPUOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/AMDGPU/IR/AMDGPUOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines declarative TableGen specifications for the AMDGPU dialect, focused on dialect IR declarations such as operations, attributes, types, enums, and registration hooks and `AMDGPUOps`.
  - **CN**: 为 AMDGPU 方言定义聚焦 `AMDGPUOps` 的声明式 TableGen 规格，覆盖方言 IR 声明，例如操作、属性、类型、枚举与注册钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- AMDGPUOps.td - AMDGPU dialect operations *- tablegen -*------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_AMDGPU_IR_AMDGPUOPS_TD
#define MLIR_DIALECT_AMDGPU_IR_AMDGPUOPS_TD

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 12-21
```tablegen
include "mlir/Dialect/AMDGPU/IR/AMDGPUBase.td"
include "mlir/Dialect/AMDGPU/IR/AMDGPUAttrs.td"
include "mlir/Dialect/AMDGPU/IR/AMDGPUTypes.td"

include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/Interfaces/ViewLikeInterface.td"
include "mlir/IR/OpBase.td"

//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 22-31
```tablegen
// AMDGPU common type constraints
//===----------------------------------------------------------------------===//

class AMDGPU_ConcreteVector<Type elem, int length> :
  FixedVectorOfLengthAndType<[length], [elem]>,
  BuildableType<
    "::mlir::VectorType::get({" # length # "} ,"
      # elem.builderCall # ")">;

def AnyIntegerOrFloat : AnyTypeOf<[AnySignlessInteger, AnyFloat], "Integer or Float">;
```
- **EN**: Introduces declarations for `AMDGPU_ConcreteVector`, `AnyIntegerOrFloat`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_ConcreteVector`, `AnyIntegerOrFloat` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 32-41
```tablegen

def AnyIntegerOrFloatOr1DVector :
  AnyTypeOf<[AnyIntegerOrFloat, FixedVectorOfRankAndType<[1], [AnyIntegerOrFloat]>]>;

// Types with element width up to 64 bits, used to keep dpp operands legal.
def AMDGPU_IntOrFloatWidthLeq64 : Type<
  CPred<"$_self.isIntOrFloat() && $_self.getIntOrFloatBitWidth() <= 64">,
  "integer or float with element bitwidth <= 64">;

def AMDGPU_IntOrFloatOr1DVectorWidthLeq64 :
```
- **EN**: Introduces declarations for `AnyIntegerOrFloatOr1DVector`, `AMDGPU_IntOrFloatWidthLeq64`, `AMDGPU_IntOrFloatOr1DVectorWidthLeq64`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AnyIntegerOrFloatOr1DVector`, `AMDGPU_IntOrFloatWidthLeq64`, `AMDGPU_IntOrFloatOr1DVectorWidthLeq64` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 42-51
```tablegen
  AnyTypeOf<[
    AMDGPU_IntOrFloatWidthLeq64,
    FixedVectorOfRankAndType<[1], [AMDGPU_IntOrFloatWidthLeq64]>
  ]>;

//===----------------------------------------------------------------------===//
// AMDGPU Op definitions
//===----------------------------------------------------------------------===//

class AMDGPU_Op<string mnemonic, list<Trait> traits = []> :
```
- **EN**: Introduces declarations for `AMDGPU_Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 52-61
```tablegen
  Op<AMDGPU_Dialect, mnemonic, traits> {}

def AMDGPU_ExtPackedFp8Op :
    AMDGPU_Op<"ext_packed_fp8", [Pure]>,
    Arguments<(ins AnyTypeOf<[F8E5M2FNUZ, F8E4M3FNUZ, F8E5M2, F8E4M3FN,
        VectorOfLengthAndType<[1, 2, 3, 4], [F8E5M2FNUZ, F8E4M3FNUZ, F8E5M2, F8E4M3FN]>]>:$source,
      ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<3>]>:$index)>,
    Results<(outs AnyTypeOf<[F32, FixedVectorOfLengthAndType<[2], [F32]>]>:$res)> {
  let summary = "Extend a fp8 value to a float or a vector of packed fp8 values to two floats";

```
- **EN**: Introduces declarations for `AMDGPU_ExtPackedFp8Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_ExtPackedFp8Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 62-79
```tablegen
  let description = [{
    Extend one or two 8-bit floats in `source[index]` to a 32-bit float or
    two floats and return them.

    This rather unusual signature arises from the fact that AMD GPUs cannot
    easily work with sub 32-bit quantities, so the compiler intrinsics for
    extending 8-bit floats (which are, currently, the only way to work with
    this operation) take packed vectors of 4 such floats.

    If the passed-in vector has fewer than four elements, or the input is scalar,
    the remaining values in the <4 x i8> will be filled with
    undefined values as needed.
  }];
  let assemblyFormat = [{
    attr-dict $source `[` $index `]` `:` type($source) `to` type($res)
  }];
}

```
- **EN**: Implements logic around `floats`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `floats`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 80-99
```tablegen
def AMDGPU_ScaledExtPackedMatrixOp
    : AMDGPU_Op<"scaled_ext_packed_matrix", [Pure, AllShapesMatch<["source", "res"]>]>,
      Arguments<(
          ins AnyTypeOf<[FixedVectorOfShapeAndType<[8], F4E2M1FN>,
                         FixedVectorOfShapeAndType<[8], F8E4M3FN>,
                         FixedVectorOfShapeAndType<[8], F8E5M2>,
                         FixedVectorOfShapeAndType<[16], F6E2M3FN>,
                         FixedVectorOfShapeAndType<[16], F6E3M2FN>]>:$source,
          FixedVectorOfShapeAndType<[4], F8E8M0FNU>:$scale,
          ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32]>]>:$blockSize,
          ConfinedAttr<I32Attr, [IntIsOneOf<[0, 16]>]>:$firstScaleLane,
          ConfinedAttr<I32Attr, [IntMinValue<0>, IntMaxValue<3>]>:$firstScaleByte)>,
      Results<(
          outs AnyTypeOf<[FixedVectorOfShapeAndType<[8], F32>,
                          FixedVectorOfShapeAndType<[8], F16>,
                          FixedVectorOfShapeAndType<[8], BF16>,
                          FixedVectorOfShapeAndType<[16], F32>,
                          FixedVectorOfShapeAndType<[16], F16>,
                          FixedVectorOfShapeAndType<[16], BF16>]>:$res)> {

```
- **EN**: Introduces declarations for `AMDGPU_ScaledExtPackedMatrixOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_ScaledExtPackedMatrixOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 100-112
```tablegen
  let summary = "Extend a wave-wide matrix of packed floating point values";

  let description = [{
    Extend matrix of microfloats (8 or 16 elements per lane) using a set of scales
    that may be stored on other lanes.

    The scales applied to the input microfloats are stored in bytes which
    come from the `scales` input provided in a *half* of the wave identified
    by `firstScaleLane`. The bytes used is selected by `firstScaleByte` and depends
    on the type of `source`. The 16 vectors in consecutive lanes starting from
    `firstScaleLane` (which we'll call the scale vectors) will be used by both
    halves of the wave (with lane L reading from L % 16'th scale vector).

```
- **EN**: Implements logic around `microfloats`, `wave`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `microfloats`, `wave` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 113-128
```tablegen
    When `source` is either F4E2M1FN, F6E2M3FN, or F6E3M2FN each half of the
    wave will use a different byte. The first one being `firstScaleByte` and
    the second one being `firstScaleByte` + 1. When the block size is 32,
    `firstScaleByte` can be either 0 or 2, selecting halves of the scale vectors.
    Lanes 0-15 will read from `firstScaleByte` and lanes 16-31 will read
    from `firstScaleByte` + 1.


    For example:
    ```mlir
    // Input: 8-element vector of F8E4M3FN, converting to F32
    // Lanes 0-15 read from byte 0, lanes 16-31 read from byte 1
    %result = amdgpu.scaled_ext_packed_matrix %source scale(%scales)
      blockSize(32) firstScaleLane(0) firstScaleByte(0)
      : vector<8xf8E4M3FN>, vector<4xf8E8M0FNU> -> vector<8xf32>

```
- **EN**: Implements logic around `scale`, `blockSize`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `scale`, `blockSize` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 129-147
```tablegen
    // Input: 16-element vector of F6E2M3FN, converting to F16
    // Lanes 0-15 read from byte 2, lanes 16-31 read from byte 3
    %result = amdgpu.scaled_ext_packed_matrix %source scale(%scales)
      blockSize(32) firstScaleLane(16) firstScaleByte(2)
      : vector<16xf6E2M3FN>, vector<4xf8E8M0FNU> -> vector<16xf16>
    ```

    When `source` is either F4E2M1FN, F6E2M3FN, or F6E3M2FN and
    the block size is 16, `firstScaleByte` can be 0 or 1.
    Lanes 0-15 read from the `firstScaleByte`th element of the scale vectors,
    while lanes 16-31 read from `firstScaleByte` + 2.
    For example:
    ```mlir
    // Input: 8-element vector of F8E5M2, converting to BF16
    // Lanes 0-15 read from byte 0, lanes 16-31 read from byte 2 (0+2)
    %result = amdgpu.scaled_ext_packed_matrix %source scale(%scales)
      blockSize(16) firstScaleLane(0) firstScaleByte(0)
      : vector<8xf8E5M2>, vector<4xf8E8M0FNU> -> vector<8xbf16>

```
- **EN**: Implements logic around `scale`, `blockSize`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `scale`, `blockSize` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 148-158
```tablegen
    // Input: 16-element vector of F6E3M2FN, converting to F32
    // Lanes 0-15 read from byte 1, lanes 16-31 read from byte 3 (1+2)
    %result = amdgpu.scaled_ext_packed_matrix %source scale(%scales)
      blockSize(16) firstScaleLane(16) firstScaleByte(1)
      : vector<16xf6E3M2FN>, vector<4xf8E8M0FNU> -> vector<16xf32>
    ```

    Note: the layout for the scales generally mirrors how the WMMA
    instructions use for matrix scales. These selection operands allows
    one to choose portions of the matrix to convert.

```
- **EN**: Implements logic around `scale`, `blockSize`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `scale`, `blockSize` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 159-168
```tablegen
    When `source` is either F8E4M3FN or F8E5M2 and `blockSize` is 32,
    then the same byte will be used by both halves of the wave.
    In this case, `firstScaleByte` can be any value from 0 to 3.

    When `source` is either F8E4M3FN or F8E5M2 and `blockSize` is 16,
    following combinations are allowed:
    * `firstScaleLane(0), firstScaleByte(0)`
    * `firstScaleLane(16), firstScaleByte(2)`
    all other combinations are reserved.

```
- **EN**: Implements logic around `firstScaleLane`.
- **CN**: 围绕 `firstScaleLane` 实现具体逻辑。

### Lines 169-180
```tablegen
    Available on gfx1250+.
  }];

  let assemblyFormat = [{
    attr-dict $source
    `scale` `(` $scale `)`
    `blockSize` `(` $blockSize `)`
    `firstScaleLane` `(` $firstScaleLane`)`
    `firstScaleByte` `(` $firstScaleByte `)`
    `:` type($source) `,` type($scale) `->` type($res)
  }];

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 181-198
```tablegen
  let hasVerifier = 1;

}

def AMDGPU_ScaledExtPackedOp
    : AMDGPU_Op<"scaled_ext_packed", [Pure]>,
      Arguments<(
          ins AnyTypeOf<[VectorOfLengthAndType<[1, 2, 3, 4], [F8E5M2, F8E4M3FN]>,
                         VectorOfLengthAndType<[1, 2, 3, 4, 5, 6, 7, 8],
                                               [F4E2M1FN]>]>:$source,
          F32:$scale,
          ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<7>]>:$index)>,
      Results<(
          outs AnyTypeOf<[FixedVectorOfLengthAndType<[2], [F32]>,
                          FixedVectorOfLengthAndType<[2], [F16]>,
                          FixedVectorOfLengthAndType<[2], [BF16]>]>:$res)> {
  let summary = "Extend a vector of packed floating point values";

```
- **EN**: Introduces declarations for `AMDGPU_ScaledExtPackedOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_ScaledExtPackedOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 199-216
```tablegen
  let description = [{
    Extend and scale two packed floats in `source[index]` to two floats and
    return them.

    This rather unusual signature arises from the fact that AMD GPUs cannot
    easily work with sub 32-bit quantities, so the compiler intrinsics for
    extending 8-bit floats (which are, currently, the only way to work with
    this operation) take packed vectors of 2 such floats.

    If the passed-in vector has fewer than two elements, or the input is scalar,
    the remaining values in the <2 x i8> will be filled with
    undefined values as needed.
  }];
  let assemblyFormat = [{
    attr-dict $source `[` $index `]` `,` $scale `:` type($source) `to` type($res)
  }];
}

```
- **EN**: Implements logic around `floats`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `floats`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 217-230
```tablegen
def AMDGPU_PackedTrunc2xFp8Op :
    AMDGPU_Op<"packed_trunc_2xfp8", [Pure, AttrSizedOperandSegments]>,
    Arguments<(ins F32:$sourceA,
      Optional<F32>:$sourceB,
      ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<1>]>:$wordIndex,
      Optional<FixedVectorOfLengthAndType<[4], [F8E4M3FNUZ, F8E5M2FNUZ, F8E4M3FN, F8E5M2]>>:$existing)>,
    Results<(outs FixedVectorOfLengthAndType<[4], [F8E4M3FNUZ, F8E5M2FNUZ, F8E4M3FN, F8E5M2]>:$res)> {
  let summary = "Round two floats into a packed vector of 8-bit floats";
  let description = [{
    Round the inputs `sourceA` and `sourceB` (which is undefined if not
    specified) into the low or high word (bottom two or top two) elements
    of the returned vector, keeping the other two elements of `existing`
    unchanged if present (or undefined if it was not passed in).

```
- **EN**: Introduces declarations for `AMDGPU_PackedTrunc2xFp8Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_PackedTrunc2xFp8Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 231-243
```tablegen
    The reason for this odd signature is that AMD GPUs cannot easily work with
    sub-registers, and so the conversion intrinsics (which are currently the
    only way to work with 8-bit float types) take packed vectors of 4 8-bit
    values.
  }];
  let assemblyFormat = [{
    attr-dict $sourceA `,` ($sourceB^):(`undef`)?
    `into` ($existing^):(`undef`)? `[` `word` $wordIndex `]`
    `:` type($sourceA) `to` type($res) (`into` type($existing)^)?
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `intrinsics`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `intrinsics`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 244-261
```tablegen
def AMDGPU_PackedScaledTruncOp
    : AMDGPU_Op<"packed_scaled_trunc", [Pure]>,
      Arguments<(ins VectorOfLengthAndType<[1, 2], [F32, F16, BF16]>:$source,
          F32:$scale,
          ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<7>]>:$index,
          Optional<AnyTypeOf<
              [FixedVectorOfLengthAndType<[4], [F8E5M2, F8E4M3FN]>,
               FixedVectorOfLengthAndType<[8], [F4E2M1FN]>]>>:$existing)>,
      Results<(
          outs AnyTypeOf<[FixedVectorOfLengthAndType<[4], [F8E5M2, F8E4M3FN]>,
                          FixedVectorOfLengthAndType<[8], [F4E2M1FN]>]>:$res)> {
  let summary = "Round two floats into a packed vector of floats";
  let description = [{
    Scale and round the inputs `source` (which is undefined if not
    specified) into the low or high word (bottom two or top two) elements
    of the returned vector, keeping the other two elements of `existing`
    unchanged if present (or undefined if it was not passed in).

```
- **EN**: Introduces declarations for `AMDGPU_PackedScaledTruncOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_PackedScaledTruncOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 262-273
```tablegen
    The reason for this odd signature is that AMD GPUs cannot easily work with
    sub-registers, and so the conversion intrinsics take 32-bit wide
    packed vectors of float values.
  }];
  let assemblyFormat = [{
    attr-dict $source `into` ($existing^):(`undef`)? `[` $index `]`
    `,` $scale
    `:` type($source) `to` type($res) (`into` type($existing)^)?
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 274-285
```tablegen
def AMDGPU_PackedStochRoundFp8Op :
    AMDGPU_Op<"packed_stoch_round_fp8", [Pure]>,
    Arguments<(ins F32:$source,
      I32:$stochiasticParam,
      ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<3>]>:$storeIndex,
      Optional<FixedVectorOfLengthAndType<[4], [F8E4M3FNUZ, F8E5M2FNUZ, F8E4M3FN, F8E5M2]>>:$existing)>,
    Results<(outs FixedVectorOfLengthAndType<[4], [F8E4M3FNUZ, F8E5M2FNUZ, F8E4M3FN, F8E5M2]>:$res)> {
  let summary = "Round float stochiastically into a packed vector of 8-bit floats";
  let description = [{
    Round the input `source`, adding in `stochiasticParam`, and place it into
    the `storeIndex`th element of `res`.

```
- **EN**: Introduces declarations for `AMDGPU_PackedStochRoundFp8Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_PackedStochRoundFp8Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 286-301
```tablegen
    If `existing` is passed in, elements of `res` other than the one at `storeIndex`
    are copied from `existing`.

    The reason for this odd signature is that AMD GPUs cannot easily work with
    sub-registers, and so the conversion intrinsics (which are currently the
    only way to work with 8-bit float types) take packed vectors of 4 8-bit
    values.
  }];
  let assemblyFormat = [{
    attr-dict $source `+` $stochiasticParam
    `into` ($existing^):(`undef`)? `[` $storeIndex `]`
    `:` type($source) `to` type($res) (`into` type($existing)^)?
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `intrinsics`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `intrinsics`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 302-321
```tablegen
def AMDGPU_FatRawBufferCastOp :
    AMDGPU_Op<"fat_raw_buffer_cast",
      [Pure,
       DeclareOpInterfaceMethods<InferTypeOpInterface>,
       DeclareOpInterfaceMethods<ReifyRankedShapedTypeOpInterface,
                                 ["reifyDimOfResult"]>,
       ViewLikeOpInterface, AttrSizedOperandSegments]>,
    Arguments<(ins AnyMemRef:$source,
      Optional<I64>:$validBytes,
      Optional<I<14>>:$cacheSwizzleStride,
      DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
      UnitAttr:$resetOffset)>,
    Results<(outs AnyMemRef:$result)> {
  // TODO: Set `resetOffset` and `boundsCheck` to use `Property` once
  //       we implemented pythonic binding for `Property`.
  let summary = "Create a raw buffer fat pointer that matches `memref`";
  let description = [{
    Wraps the memory pointed to by `source` as a raw buffer fat pointer, or,
    in LLVM terms, a `ptr addrspace(7)`, returning a memref that has the same
    sizes and layout but the `#amdgpu.address_space<fat_raw_buffer>`
```
- **EN**: Introduces declarations for `AMDGPU_FatRawBufferCastOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_FatRawBufferCastOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 322-333
```tablegen
    address space.

    This memref can be used with standard memref operations like `memref.load`,
    `memref.store`, and `memref.atomicrmw`, which will be lowered to the relevant
    buffer intrinsics. (`vector.masked_load/store` will work once there's backend
    support for lowering them, and then this document will be updated)

    If `validBytes` is given, it is the number of bytes that will be valid as
    an offset to `out`. If it is not provided, this will be inferred from
    the size of the memref during lowering. This size is
    max_{d = 0 upto rank(source)} (sizes[d] * strides[d]) * sizeof(element type).

```
- **EN**: Implements logic around `rank`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `rank` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 334-345
```tablegen
    The flags of the buffer descriptor will be set up to enable raw usage -
    for example, stride = 0, add_tid = 0, and so on. The `boundsCheck`
    property determines if bounds checking is enabled or not (on architectures
    where this can be controlled - that is, on RDNA chips).

    If `cacheSwizzleStride` is provided, L1 cache swizzling will be enabled
    on architectures that support it. This swizzling, unlike the main swizzling
    mode (whose usage makes a buffer non-raw) does not affect index calculation,
    but does affect cache behavior. Mixing access between cache-swizzled raw
    buffers and other forms of memory access, like ordinary pointer loads or
    unswizzled buffer pointers can cause incorrect behavior and must be avoided.

```
- **EN**: Implements logic around `not`, `mode`.
- **CN**: 围绕 `not`, `mode` 实现具体逻辑。

### Lines 346-357
```tablegen
    This operation preserves the sizes, strides, and offset of the input
    memref - they'll be added in by `memref.load` later. However, if
    `resetOffset` is set, that offset will be added to the base pointer.
    If the value of the memref's offset is not uniform (independent of the lane/thread ID),
    this will lead to substantially decreased performance due to the need for
    a waterfall loop on the base address of the buffer resource.
  }];

  let extraClassDeclaration = [{
    Value getViewSource() { return getSource(); }
  }];

```
- **EN**: Implements logic around `uniform`, `getViewSource`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `uniform`, `getViewSource` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 358-368
```tablegen
  let assemblyFormat = [{
    $source oilist (`validBytes` `(` $validBytes `)`
      | `cacheSwizzleStride` `(` $cacheSwizzleStride `)`
      | `boundsCheck` `(` $boundsCheck `)`
      | `resetOffset` $resetOffset )
    attr-dict `:` type($source) `to` type($result)
  }];

  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `oilist`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `oilist`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 369-379
```tablegen
/// Raw buffer load
def AMDGPU_RawBufferLoadOp :
    AMDGPU_Op<"raw_buffer_load", [AllElementTypesMatch<["value", "memref"]>,
      AttrSizedOperandSegments]>,
    Arguments<(ins Arg<AnyMemRef, "buffer to load from", [MemRead]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)>,
    Results<(outs AnyType:$value)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 380-389
```tablegen
  let summary = "Raw Buffer load, exposing GCN features";
  let description = [{
    The `amdgpu.raw_buffer_load` op is a wrapper around the buffer load intrinsics
    available on AMD GPUs, including extensions in newer GPUs.

    The index into the buffer is computed as for `memref.load` with the additon
    of `indexOffset` and `sgprOffset` (which **may or may not** be considered
    in bounds checks and includes any offset present on the memref type if it's
    non-zero).

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 390-409
```tablegen
    All indices and offsets are in units of the memref's data type and are
    converted to bytes during lowering.

    When a load is out of bounds, the instruction returns zero.
    Partially-out of bounds have chipset-dependent behavior: whether reading
    2 elements starting at index 7 of a `memref<8xf32>` returns the last element
    in the first vector component depends on the architecture.

    The memref struct is converted into a buffer resource (a V#) and the arguments
    are translated to intrinsic arguments as follows:
    - The base address of the buffer is the base address of the memref
    - The stride is 0 to enable raw mode
    - The number of records is the size of the memref, in bytes
      In the case of dynamically-shaped memrefs, this is computed at runtime
      as max_d (size(d) * stride(d)) * sizeof(elementType(memref))
    - The offset enable bit is 1, the index enable bit is 0.
    - The thread ID addition bit is off
    - If `boundsCheck` is false and the target chipset is RDNA, OOB_SELECT is set
      to 2 to disable bounds checks, otherwise it is 3
    - The cache coherency bits are off
```
- **EN**: Introduces declarations for `is`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `is` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 410-419
```tablegen
  }];
  let assemblyFormat = [{
    attr-dict $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($memref) (`,` type($indices)^)? `->` type($value)
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 420-430
```tablegen
/// Raw buffer store
def AMDGPU_RawBufferStoreOp :
    AMDGPU_Op<"raw_buffer_store", [AllElementTypesMatch<["value", "memref"]>,
      AttrSizedOperandSegments]>,
    Arguments<(ins AnyType:$value,
                   Arg<AnyMemRef, "buffer to store to", [MemWrite]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferStoreOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferStoreOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 431-442
```tablegen
  let summary = "Raw Buffer Store, exposing GCN features";
  let description = [{
    The `amdgpu.raw_buffer_store` op is a wrapper around the buffer store
    intrinsics available on AMD GPUs, including extensions in newer GPUs.

    The store index is computed as in `memref.store` with the addition of
    `indexOffset` (which is included for uniformity with atomics and may be useful
    when writing vectorized code) and `sgprOffset` (which is added after bounds
    checks and implicitly includes the offset of the memref type if non-zero).
    All index components are in terms of the elements of the memref, not bytes,
    and are scaled up appropriately.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 443-458
```tablegen
    Out of bounds stores are ignored in hardware.
    Wthether a vector write that includes some in-bounds and soeme out-of-bounds
    components is partically completed is chipset-dependent.

    See `amdgpu.raw_buffer_load` for a description of how the underlying
    instruction is constructed.
  }];
  let assemblyFormat = [{
    attr-dict $value `->` $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($value) `->` type($memref) (`,` type($indices)^)?
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 459-473
```tablegen
// Raw buffer atomic compare-and-swap
def AMDGPU_RawBufferAtomicCmpswapOp :
    AMDGPU_Op<"raw_buffer_atomic_cmpswap", [
      AttrSizedOperandSegments,
      AllTypesMatch<["src", "cmp", "value"]>,
      AllElementTypesMatch<["value", "memref"]>]>,
    Arguments<(ins AnyType:$src,
                   AnyType:$cmp,
                   Arg<AnyMemRef, "buffer to operate on", [MemRead, MemWrite]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)>,
    Results<(outs AnyType:$value)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferAtomicCmpswapOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferAtomicCmpswapOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 474-483
```tablegen
  let summary = "Raw Buffer Atomic compare-and-swap";
  let description = [{
    The `amdgpu.raw_buffer_atomic_cmpswap` op is a wrapper around the
    buffer-based atomic compare-and-swap min available on AMD GPUs.

    The index into the buffer is computed as for `memref.store` with the addition
    of `indexOffset` (which is used to aid in emitting vectorized code) and,
    if present `sgprOffset` (which is added after bounds checks and includes
    any non-zero offset on the memref type).

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 484-500
```tablegen
    All indexing components are given in terms of the memref's element size, not
    the byte lengths required by the intrinsic.

    Out of bounds atomic operations are ignored in hardware.

    See `amdgpu.raw_buffer_load` for a description of how the underlying
    instruction is constructed.
  }];
  let assemblyFormat = [{
    attr-dict $src `,` $cmp `->` $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($value) `->` type($memref) `,` type($indices)
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 501-511
```tablegen
// Raw buffer atomic floating point add
def AMDGPU_RawBufferAtomicFaddOp :
    AMDGPU_Op<"raw_buffer_atomic_fadd", [AllElementTypesMatch<["value", "memref"]>,
      AttrSizedOperandSegments]>,
    Arguments<(ins AnyTypeOf<[F32, VectorOfLengthAndType<[2], [F16, BF16]>]>:$value,
                   Arg<AnyMemRef, "buffer to operate on", [MemRead, MemWrite]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferAtomicFaddOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferAtomicFaddOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 512-522
```tablegen
  let summary = "Raw Buffer Floating-point Atomic Add (MI-* only)";
  let description = [{
    The `amdgpu.raw_buffer_atomic_fadd` op is a wrapper around the
    buffer-based atomic floating point addition available on the MI-* series
    of AMD GPUs.

    The index into the buffer is computed as for `memref.store` with the addition
    of `indexOffset` (which is used to aid in emitting vectorized code) and,
    if present `sgprOffset` (which is added after bounds checks and includes
    any non-zero offset on the memref type).

```
- **EN**: Implements logic around `Add`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Add` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 523-539
```tablegen
    All indexing components are given in terms of the memref's element size, not
    the byte lengths required by the intrinsic.

    Out of bounds atomic operations are ignored in hardware.

    See `amdgpu.raw_buffer_load` for a description of how the underlying
    instruction is constructed.
  }];
  let assemblyFormat = [{
    attr-dict $value `->` $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($value) `->` type($memref) `,` type($indices)
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 540-550
```tablegen
// Raw buffer atomic floating point max
def AMDGPU_RawBufferAtomicFmaxOp :
    AMDGPU_Op<"raw_buffer_atomic_fmax", [AllElementTypesMatch<["value", "memref"]>,
      AttrSizedOperandSegments]>,
    Arguments<(ins AnyTypeOf<[F32, F64]>:$value,
                   Arg<AnyMemRef, "buffer to operate on", [MemRead, MemWrite]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferAtomicFmaxOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferAtomicFmaxOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 551-560
```tablegen
  let summary = "Raw Buffer Floating-point Atomic Max (non-GFX9)";
  let description = [{
    The `amdgpu.raw_buffer_atomic_fmax` op is a wrapper around the
    buffer-based atomic floating point max available on AMD GPUs (except GFX9).

    The index into the buffer is computed as for `memref.store` with the addition
    of `indexOffset` (which is used to aid in emitting vectorized code) and,
    if present `sgprOffset` (which is added after bounds checks and includes
    any non-zero offset on the memref type).

```
- **EN**: Implements logic around `Max`, `GPUs`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Max`, `GPUs` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 561-577
```tablegen
    All indexing components are given in terms of the memref's element size, not
    the byte lengths required by the intrinsic.

    Out of bounds atomic operations are ignored in hardware.

    See `amdgpu.raw_buffer_load` for a description of how the underlying
    instruction is constructed.
  }];
  let assemblyFormat = [{
    attr-dict $value `->` $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($value) `->` type($memref) `,` type($indices)
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 578-588
```tablegen
// Raw buffer atomic signed integer max
def AMDGPU_RawBufferAtomicSmaxOp :
    AMDGPU_Op<"raw_buffer_atomic_smax", [
      AttrSizedOperandSegments]>,
    Arguments<(ins I32:$value,
                   Arg<AnyMemRef, "buffer to operate on", [MemRead, MemWrite]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferAtomicSmaxOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferAtomicSmaxOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 589-598
```tablegen
  let summary = "Raw Buffer Signed Integer Atomic Max";
  let description = [{
    The `amdgpu.raw_buffer_atomic_smax` op is a wrapper around the
    buffer-based atomic signed integer max available on AMD GPUs.

    The index into the buffer is computed as for `memref.store` with the addition
    of `indexOffset` (which is used to aid in emitting vectorized code) and,
    if present `sgprOffset` (which is added after bounds checks and includes
    any non-zero offset on the memref type).

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 599-615
```tablegen
    All indexing components are given in terms of the memref's element size, not
    the byte lengths required by the intrinsic.

    Out of bounds atomic operations are ignored in hardware.

    See `amdgpu.raw_buffer_load` for a description of how the underlying
    instruction is constructed.
  }];
  let assemblyFormat = [{
    attr-dict $value `->` $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($value) `->` type($memref) `,` type($indices)
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 616-626
```tablegen
// Raw buffer atomic unsigned integer min
def AMDGPU_RawBufferAtomicUminOp :
    AMDGPU_Op<"raw_buffer_atomic_umin", [
      AttrSizedOperandSegments]>,
    Arguments<(ins I32:$value,
                   Arg<AnyMemRef, "buffer to operate on", [MemRead, MemWrite]>:$memref,
                   Variadic<I32>:$indices,
                   DefaultValuedAttr<BoolAttr, "true">:$boundsCheck,
                   OptionalAttr<I32Attr>:$indexOffset,
                   Optional<I32>:$sgprOffset)> {

```
- **EN**: Introduces declarations for `AMDGPU_RawBufferAtomicUminOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_RawBufferAtomicUminOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 627-636
```tablegen
  let summary = "Raw Buffer Unsigned Integer Atomic Min";
  let description = [{
    The `amdgpu.raw_buffer_atomic_umin` op is a wrapper around the
    buffer-based atomic signed integer min available on AMD GPUs.

    The index into the buffer is computed as for `memref.store` with the addition
    of `indexOffset` (which is used to aid in emitting vectorized code) and,
    if present `sgprOffset` (which is added after bounds checks and includes
    any non-zero offset on the memref type).

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 637-653
```tablegen
    All indexing components are given in terms of the memref's element size, not
    the byte lengths required by the intrinsic.

    Out of bounds atomic operations are ignored in hardware.

    See `amdgpu.raw_buffer_load` for a description of how the underlying
    instruction is constructed.
  }];
  let assemblyFormat = [{
    attr-dict $value `->` $memref `[` $indices `]`
      (`sgprOffset` $sgprOffset^)? `:`
      type($value) `->` type($memref) `,` type($indices)
  }];
  let hasCanonicalizer = 1;
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 654-672
```tablegen

def AMDGPU_DPPOp : AMDGPU_Op<"dpp",
    [Pure, SameTypeOperands, AllTypesMatch<["result", "old", "src"]>]>,
  Arguments<(ins AMDGPU_IntOrFloatOr1DVectorWidthLeq64:$old,
                 AMDGPU_IntOrFloatOr1DVectorWidthLeq64:$src,
                 AMDGPU_DPPPermAttr:$kind,
                 OptionalAttr<AnyAttrOf<[I32Attr, ArrayAttr, UnitAttr]>>:$permArgument,
                 DefaultValuedAttr<I32Attr, "0xf">:$row_mask,
                 DefaultValuedAttr<I32Attr, "0xf">:$bank_mask,
                 DefaultValuedAttr<BoolAttr, "false">:$bound_ctrl)> {
  let summary = "AMDGPU DPP operation";
  let description = [{
    The `amdgpu.dpp` op performs a Data Parallel Primitives (DPP) lane
    permutation on a source value within a wavefront. Each lane reads its
    source data from another lane according to the permutation mode specified
    by `kind`. DPP operates at dword (32-bit) granularity: sub-32-bit types
    (e.g., f16, i16) are packed into an i32 during lowering, permuted, and
    extracted back.

```
- **EN**: Introduces declarations for `AMDGPU_DPPOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DPPOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 673-684
```tablegen
    - Lanes are organized into rows of 16.
    - A Wave64 wavefront has 4 rows of 16 lanes each: row 0 = lanes 0-15,
      row 1 = lanes 16-31, row 2 = lanes 32-47, row 3 = lanes 48-63.
    - Similarly, a Wave32 wavefront has two rows of 16 lanes each, organized
      in the same fashion.
    - Each row is divided into 4 banks of 4 consecutive lanes: bank 0 =
      lanes 0-3, bank 1 = lanes 4-7, bank 2 = lanes 8-11, bank 3 =
      lanes 12-15 (lane numbers shown for row 0; add 16/32/48 for other rows).

    The `kind` attribute selects the permutation. Some modes require a
    `permArgument`; others take no argument.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 685-701
```tablegen
    Quad permutation:
    - `quad_perm([a, b, c, d])`: Full permute within each group of 4
      consecutive lanes (a quad). Each element is in [0, 3] and selects which
      lane within the quad to read from. Lane 4k+i reads from lane 4k+perm[i].
      For example, `quad_perm([1, 0, 3, 2])` swaps adjacent pairs within
      every quad.

    Row shifts and rotates (operate within each 16-lane row independently):
    - `row_shl(N)`: Shift left by N (1-15) within the row. Lane n reads from
      lane (n % 16) + N in the same row. Lanes where the source index exceeds
      15 are out of bounds (see `bound_ctrl`).
    - `row_shr(N)`: Shift right by N (1-15) within the row. Lane n reads from
      lane (n % 16) - N in the same row. Lanes where the source index is
      negative are out of bounds.
    - `row_ror(N)`: Rotate right by N (1-15) within the row. Lane n reads from
      lane ((n % 16) - N) mod 16 in the same row. Always in bounds.

```
- **EN**: Implements logic around `quad_perm`, `lanes`, `rotates`, `row_shl`, and 4 more symbols.
- **CN**: 围绕 `quad_perm`, `lanes`, `rotates`, `row_shl`, and 4 more symbols 实现具体逻辑。

### Lines 702-711
```tablegen
    Wavefront shifts and rotates (not available on RDNA):
    - `wave_shl`: Shift left by 1. Lane n reads from lane n + 1. The last lane
      in the wavefront is out of bounds.
    - `wave_shr`: Shift right by 1. Lane n reads from lane n - 1. Lane 0 is
      out of bounds.
    - `wave_rol`: Rotate left by 1. Lane n reads from lane (n + 1) mod W, where
      W is the wavefront size.
    - `wave_ror`: Rotate right by 1. Lane n reads from lane (n - 1) mod W, where
      W is the wavefront size.

```
- **EN**: Implements logic around `rotates`, `lane`.
- **CN**: 围绕 `rotates`, `lane` 实现具体逻辑。

### Lines 712-723
```tablegen
    Row mirrors:
    - `row_mirror`: Reverse lanes within each 16-lane row. Lane n reads from
      lane 15 - (n % 16) within its row.
    - `row_half_mirror`: Reverse within each 8-lane half-row. Lane n reads
      from lane 7 - (n % 8) within its half-row.

    Row broadcasts (not available on RDNA):
    - `row_bcast_15`: Lane 15 of each row broadcasts to all lanes of the next
      row. Lanes in row 0 are not affected (retain `old`).
    - `row_bcast_31`: Lane 31 broadcasts to all lanes in rows 2 and 3.
      Lanes in rows 0 and 1 are not affected (retain `old`).

```
- **EN**: Implements logic around `broadcasts`, `affected`.
- **CN**: 围绕 `broadcasts`, `affected` 实现具体逻辑。

### Lines 724-735
```tablegen
    Example:
    ```mlir
    // Swap adjacent pairs within each quad (lanes 0<->1, 2<->3, etc.)
    %0 = amdgpu.dpp %old %src quad_perm( [1, 0, 3, 2] ) : i32

    // Shift right by 1 lane within each 16-lane row.
    // bound_ctrl=true -> lanes that would read past the row return 0.
    // row_mask=0x5 (0b0101) -> only rows 0 and 2 apply the shift;
    // rows 1 and 3 pass through %old unchanged.
    %1 = amdgpu.dpp %old %src row_shr( 0x1 : i32 )
      { row_mask = 0x5 : i32, bound_ctrl = true } : f32

```
- **EN**: Implements logic around `quad_perm`, `row_shr`; this block packages logic as an MLIR pass or pass-related API; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `quad_perm`, `row_shr` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并涉及目标平台或加速器专用语义。

### Lines 736-755
```tablegen
    // Rotate left across the full wavefront by 1 lane
    %2 = amdgpu.dpp %old %src wave_rol : i32
    ```

    Operands:
    * `$old`: Fallback value. Lanes that are masked off by `row_mask` /
      `bank_mask` retain `old`. For lanes with an out-of-bounds source, behavior
      depends on `bound_ctrl`.
    * `$src`: Source value to be permuted across lanes.
    * `$kind`: A `#amdgpu.dpp_perm` enum selecting the permutation mode.
    * `$permArgument`: Mode-specific argument. Required for `quad_perm`
      (array of 4 integers in [0, 3]) and `row_shl`/`row_shr`/`row_ror`
      (integer in [1, 15]). Absent for all other modes.
    * `$row_mask` (default 0xf): 4-bit mask controlling which rows write
      results. Bit i enables row i (bit 0 = lanes 0-15, bit 1 = lanes
      16-31, etc.). Disabled lanes retain `old`.
    * `$bank_mask` (default 0xf): 4-bit mask controlling which banks write
      results. Bit i enables bank i (bit 0 = lanes 0-3, 16-19, etc. across all rows).
      Disabled lanes retain `old`.
    * `$bound_ctrl` (default false): When false, out of bounds lanes retain
```
- **EN**: Introduces declarations for `selecting`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `selecting` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 756-765
```tablegen
      `old`. When true, out-of-bounds lanes receive zero.
  }];
  let results = (outs AnyType:$result);
  let assemblyFormat = [{
    $old $src $kind (`(` $permArgument^ `)`)? attr-dict `:` type($result)
  }];
  let hasVerifier = 1;
}

def AMDGPU_SwizzleBitModeOp : AMDGPU_Op<"swizzle_bitmode",
```
- **EN**: Introduces declarations for `AMDGPU_SwizzleBitModeOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_SwizzleBitModeOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 766-776
```tablegen
    [Pure, AllTypesMatch<["result", "src"]>]>,
  Arguments<(ins AnyIntegerOrFloatOr1DVector:$src,
                 I32Attr:$and_mask,
                 I32Attr:$or_mask,
                 I32Attr:$xor_mask
             )> {
  let summary = "AMDGPU ds_swizzle op, bitmode variant";
  let description = [{
    High-level wrapper on bitmode `rocdl.ds_swizzle` op, masks are represented
    as separate fields so user won't need to do manual bitpacking.

```
- **EN**: Implements logic around `Arguments`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Arguments` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 777-786
```tablegen
    Supports arbitrary int/float/vector types, which will be repacked to i32 and
    one or more `rocdl.ds_swizzle` ops during lowering.
  }];
  let results = (outs AnyIntegerOrFloatOr1DVector:$result);
  let assemblyFormat = [{
    $src $and_mask $or_mask $xor_mask attr-dict `:` type($result)
  }];
}

def AMDGPU_PermlaneSwapOp : AMDGPU_Op<"permlane_swap", [Pure, AllTypesMatch<["result", "src"]>]> {
```
- **EN**: Introduces declarations for `AMDGPU_PermlaneSwapOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_PermlaneSwapOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 787-797
```tablegen
  let summary = "AMDGPU permlane swap op";
  let description = [{
    High-level wrapper on `rocdl.permlane{16,32}.swap` variants for permutations
    on rows of lanes in a subgroup.

    Supports arbitrary int/float/vector types, which will be repacked to i32 and
    one or more `rocdl.permlane_swap` ops during lowering.
    Supported lane permutations:
    - Swap the data between odd and even rows of 16 lanes
    - Swap the data between the first 32 lanes and the last 32 lanes

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 798-814
```tablegen
    Example:
    ```mlir
    %0 = amdgpu.permlane_swap %src 16 : f16
    %1 = amdgpu.permlane_swap %src 32 { fetch_inactive = true, bound_ctrl = true } : f16
    ```

    Operands:
    * `$src`: Vector register to permute across lanes of the subgroup.
    * `$row_length`: The length of a row to permute in number of lanes (valid values are 16 and 32).
    * `$fetch_inactive`: Optional. Used to dertermine behavior of a fetch from a disabled lane.
      `fetch_inactive = false`: If the source lane is disabled, use `bound_ctrl` to determine the source value.
      `fetch_inactive = true`: If the source lane is disabled, fetch the source value anyway (ignoring `bound_ctrl`).
    * `$bound_ctrl`: Optional. Used to determine what a thread should do if its source operand is from
      a disabled lane: use the value zero, or disable the write.
      `bound_ctrl = false`: Do not write when source is from a disabled lane
      `bound_ctrl = true`: Use zero as input if source is from a disabled lane

```
- **EN**: Implements logic around `lanes`, `anyway`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `lanes`, `anyway` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 815-827
```tablegen
    Note: Lowering is only supported on gfx950 and up.
  }];
  let arguments = (ins AnyIntegerOrFloatOr1DVector:$src,
                       I32Attr:$row_length,
                       DefaultValuedAttr<BoolAttr, "false">:$fetch_inactive,
                       DefaultValuedAttr<BoolAttr, "false">:$bound_ctrl);
  let results = (outs AnyIntegerOrFloatOr1DVector:$result);
  let assemblyFormat = [{
    $src $row_length attr-dict `:` type($result)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block coordinates dialect conversion or lowering decisions; handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 828-837
```tablegen
def AMDGPU_LDSBarrierOp : AMDGPU_Op<"lds_barrier"> {
  let summary = "Barrier that includes a wait for LDS memory operations.";
  let description = [{
    **DEPRECATION NOTICE**: Unless you need the inline-assembly-based workaround
    for gfx908/MI-100, you should represent this pattern with the equivalent

    ```mlir
    gpu.barrier memfence [#gpu.address_space<workgroup>]
    ```

```
- **EN**: Introduces declarations for `AMDGPU_LDSBarrierOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_LDSBarrierOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 838-851
```tablegen
    instead.

    `amdgpu.lds_barrier` is both a barrier (all workitems in a workgroup must reach
    the barrier before any of them may proceed past it) and a wait for all
    operations that affect the Local Data Store (LDS) issued from that workgroup
    to complete before the workgroup may continue. Since the LDS is per-workgroup
    memory, this barrier may be used, for example, to ensure all workitems have
    written data to LDS before any workitem attempts to read from it.

    Note that `lds_barrier` does **not** force reads to or from global memory
    to complete before execution continues. Therefore, it should be used when
    operations on global memory can be issued far in advance of when their results
    are used (for example, by writing them to LDS).

```
- **EN**: Implements logic around `barrier`, `Store`, `used`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `barrier`, `Store`, `used` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 852-861
```tablegen
    WARNING: On architectures that do not support the BackOffBarrier feature,
    (those which will implement this barrier by emitting inline assembly),
    use of this operation will impede the usabiliity of memory watches (including
    breakpoints set on variables) when debugging.
  }];
  let assemblyFormat = "attr-dict";
  let hasCanonicalizer = 1;
}

def AMDGPU_SchedBarrierOp :
```
- **EN**: Introduces declarations for `AMDGPU_SchedBarrierOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_SchedBarrierOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 862-875
```tablegen
  AMDGPU_Op<"sched_barrier">,
  Arguments<(ins  AMDGPU_SchedBarrierOpOptAttr:$opts)>
  {
  let summary = "Barrier that limits the backend scheduler of instruction movement";
  let description = [{
    `amdgpu.sched_barrier` serves as a barrier that could be
    configured to restrict movements of instructions through it as
    defined by sched_barrier_opts.
  }];
  let assemblyFormat = [{
    `allow` `=` $opts attr-dict
  }];
}

```
- **EN**: Implements logic around `Arguments`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Arguments` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 876-890
```tablegen
def AMDGPU_MemoryCounterWaitOp :
  AMDGPU_Op<"memory_counter_wait">,
  Arguments<(ins
      OptionalAttr<I32Attr>:$load,
      OptionalAttr<I32Attr>:$store,
      OptionalAttr<I32Attr>:$ds,
      OptionalAttr<I32Attr>:$exp,
      OptionalAttr<I32Attr>:$tensor
    )>
  {
  let summary = "Wait for specified hardware counters";
  let description = [{
    Wait for the specified counters to be less-than or equal-to the provided
    values before continuing.

```
- **EN**: Introduces declarations for `AMDGPU_MemoryCounterWaitOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_MemoryCounterWaitOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 891-901
```tablegen
    Counters can lower to different instructions on different architectires,
    including clamping to the some HW supported max value or combining multiple
    counters into one.
  }];
  let assemblyFormat = [{
    oilist( `load` `(` $load `)` | `store` `(` $store `)` | `ds` `(` $ds `)` | `exp` `(` $exp `)` | `tensor` `(` $tensor `)` ) attr-dict
  }];

  let hasCanonicalizer = 1;
}

```
- **EN**: Implements logic around `oilist`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `oilist` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 902-912
```tablegen

// mfma
def MFMAInTypes : AnyTypeOf<[F32, F64, I32, I64,
                             VectorOfLengthAndType<[2], [F32]>,
                             VectorOfLengthAndType<[4, 8], [F16]>,
                             VectorOfLengthAndType<[2, 4, 8], [BF16]>,
                             VectorOfLengthAndType<[4, 8, 16], [I8]>,
                             VectorOfLengthAndType<[8], [F8E5M2FNUZ, F8E4M3FNUZ]>,
                             VectorOfLengthAndType<[8, 32], [F8E5M2, F8E4M3FN]>,
                             VectorOfLengthAndType<[32], [F6E2M3FN, F6E3M2FN, F4E2M1FN]>]>;
def MFMAOutTypes : AnyTypeOf<[F64,
```
- **EN**: Introduces declarations for `MFMAInTypes`, `MFMAOutTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MFMAInTypes`, `MFMAOutTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 913-925
```tablegen
                              VectorOfLengthAndType<[4, 16, 32], [F32]>,
                              VectorOfLengthAndType<[4, 16, 32], [I32]>,
                              VectorOfLengthAndType<[4], [F64]>]>;

// sparse_mfma (smfmac)
def SMFMACSparseInTypes : AnyTypeOf<[
    VectorOfLengthAndType<[4, 8], [F16]>,
    VectorOfLengthAndType<[4, 8], [BF16]>,
    VectorOfLengthAndType<[8, 16], [I8]>,
    VectorOfLengthAndType<[8, 16], [F8E4M3FN, F8E5M2]>,
    VectorOfLengthAndType<[8, 16], [F8E4M3FNUZ, F8E5M2FNUZ]>
]>;

```
- **EN**: Introduces declarations for `SMFMACSparseInTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMFMACSparseInTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 926-938
```tablegen
def SMFMACDenseInTypes : AnyTypeOf<[
    VectorOfLengthAndType<[8, 16], [F16]>,
    VectorOfLengthAndType<[8, 16], [BF16]>,
    VectorOfLengthAndType<[16, 32], [I8]>,
    VectorOfLengthAndType<[16, 32], [F8E4M3FN, F8E5M2]>,
    VectorOfLengthAndType<[16, 32], [F8E4M3FNUZ, F8E5M2FNUZ]>
]>;

def SMFMACOutTypes : AnyTypeOf<[
    VectorOfLengthAndType<[4, 16], [F32]>,
    VectorOfLengthAndType<[4, 16], [I32]>
]>;

```
- **EN**: Introduces declarations for `SMFMACDenseInTypes`, `SMFMACOutTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMFMACDenseInTypes`, `SMFMACOutTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 939-948
```tablegen
def SMFMACIdxTypes : AnyTypeOf<[
    FixedVectorOfLengthAndType<[4], [I8]>,
    FixedVectorOfLengthAndType<[2], [I16]>,
    I32
]>;

// scaled_mfma
def ScaledMFMAInTypes : AnyTypeOf<[VectorOfLengthAndType<[32], [F8E5M2, F8E4M3FN]>,
                                   VectorOfLengthAndType<[32], [F6E2M3FN, F6E3M2FN, F4E2M1FN]>]>;
def ScaledMFMAOutTypes : AnyTypeOf<[VectorOfLengthAndType<[4, 16], [F32]>]>;
```
- **EN**: Introduces declarations for `SMFMACIdxTypes`, `ScaledMFMAInTypes`, `ScaledMFMAOutTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMFMACIdxTypes`, `ScaledMFMAInTypes`, `ScaledMFMAOutTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 949-959
```tablegen

// scaled_wmma
def ScaledWMMAInTypes
    : AnyTypeOf<[VectorOfLengthAndType<[64], [F8E5M2, F8E4M3FN]>,
                 VectorOfLengthAndType<[64], [F6E2M3FN, F6E3M2FN]>,
                 VectorOfLengthAndType<[64, 128], [F4E2M1FN]>]>;

def ScaledWMMAOutTypes : AnyTypeOf<[VectorOfLengthAndType<[8, 16], [F32]>]>;

// wmma
def WMMAInTypes : AnyTypeOf<[VectorOfLengthAndType<[2], [F32]>,
```
- **EN**: Introduces declarations for `ScaledWMMAInTypes`, `ScaledWMMAOutTypes`, `WMMAInTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScaledWMMAInTypes`, `ScaledWMMAOutTypes`, `WMMAInTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 960-979
```tablegen
                             VectorOfLengthAndType<[4, 8, 16], [F16, BF16]>,
                             VectorOfLengthAndType<[4, 8, 16, 32], [I8, SI8, UI8]>,
                             VectorOfLengthAndType<[4, 8, 32, 64], [F8E4M3FN, F8E5M2]>,
                             VectorOfLengthAndType<[4, 8, 16], [I<4>, SI<4>, UI<4>]>]>;
def WMMAOutTypes : AnyTypeOf<[VectorOfLengthAndType<[4, 8], [F32, I32]>,
                              VectorOfLengthAndType<[4, 8, 16], [F16, BF16]>]>;

def AMDGPU_MFMAOp :
    AMDGPU_Op<"mfma", [AllTypesMatch<["destC", "destD"]>,
                        Pure]>,
    Arguments<(ins
                   ConfinedAttr<I32Attr, [IntIsOneOf<[4, 16, 32]>]>:$m,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[4, 16, 32]>]>:$n,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[1, 2, 4, 8, 16, 32, 64, 128]>]>:$k,
                   DefaultValuedAttr<ConfinedAttr<I32Attr, [IntIsOneOf<[1, 2, 4, 16]>]>, "1">:$blocks,
                   MFMAInTypes:$sourceA,
                   MFMAInTypes:$sourceB,
                   MFMAOutTypes:$destC,
                   DefaultValuedAttr<I32Attr, "0">:$cbsz,
                   DefaultValuedAttr<I32Attr, "0">:$abid,
```
- **EN**: Introduces declarations for `WMMAOutTypes`, `AMDGPU_MFMAOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `WMMAOutTypes`, `AMDGPU_MFMAOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 980-992
```tablegen
                   DefaultValuedAttr<AMDGPU_MFMAPermBAttr,
                    "::mlir::amdgpu::MFMAPermB::none">:$blgp,
                   UnitAttr:$reducePrecision,
                   UnitAttr:$negateA,
                   UnitAttr:$negateB,
                   UnitAttr:$negateC)>,
    Results<(outs MFMAOutTypes: $destD)> {
  let summary = "MLIR wrapper for CDNA mfma instructions";
  let description = [{
    The `amdgpu.mfma` op is an MLIR wrapper around intrinsics
    for various `mfma` instructions in the CDNA architecture, which perform
    multiple outer products in order to allow fast matrix multiplication.

```
- **EN**: Implements logic around `Results`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Results` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 993-1004
```tablegen
    The wrapper will select an appropriate `mfma` instruction, if one is available,
    based on the provided `m`, `k`, `n`, and `nBlks` attributes, along with the
    types of the source and destination arguments.

    For information on the layouts of the input and output matrices (which are stored
    in `sourceA`, `sourceB`, `destC`, and `destD`), see the CDNA ISA documentation.

    The `cbsz`, `abid`, and `blgp` parameters control how the lanes of the wave
    are permuted when matrix data is being loaded: `blgp` can be any number of
    fixed permutations, `cbsz` specifies the log_2 of the number of chunks the lanes
    holding sourceA are split into, and `abid` selects one of those chunks.

```
- **EN**: Implements logic around `matrices`.
- **CN**: 围绕 `matrices` 实现具体逻辑。

### Lines 1005-1014
```tablegen
    Note, this wrapper allows specifying `vector<4Kxi8>` arguments to MFMA
    intrinsics that take an integer type of width `4K`. For example,
    one can provide a vector<4xi8> as an argument to an MFMA instruction that
    logically takes 4 i8s but whose intrinsics are specified to take an i32.
    In these cases, the bytes in the vector will be concatenated in little-endian
    order (that is, v[0] will go to arg[7:0], v[1] to arg[15:8] and so on).

    The negateA, negateB, and negateC flags are only supported for double-precision
    operations on gfx94x.

```
- **EN**: Implements logic around `order`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `order` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1015-1033
```tablegen
    Example:
    ```mlir
      %0 = amdgpu.mfma 16x16x16 %matA * %matB + %matC
        : vector<4xf16>, vector<4xf16>, vector<4xf32>

      %1 = amdgpu.mfma 32x32x1 %matD * %matE + %matF
        { abid = 1 : i32, cbsz = 1 : i32, blocks = 2 : i32 }
        blgp = bcast_second_32 : f32, f32, vector<32xf32>
    ```
  }];
  let assemblyFormat = [{
    custom<MNKDimensionList>($m, $n, $k) $sourceA `*` $sourceB `+` $destC
    attr-dict
    `blgp` `=` $blgp
    `:` type($sourceA) `,` type($sourceB) `,` type($destC)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1034-1053
```tablegen
def AMDGPU_WMMAOp :
    AMDGPU_Op<"wmma", [AllTypesMatch<["destC", "destD"]>,
                       Pure]>,
    Arguments<(ins
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16]>]>:$m,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16]>]>:$n,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[4, 16, 32, 64, 128]>]>:$k,
                   WMMAInTypes:$sourceA,
                   WMMAInTypes:$sourceB,
                   WMMAOutTypes:$destC,
                   DefaultValuedAttr<ConfinedAttr<I32Attr, [IntIsOneOf<[0, 1]>]>, "0">:$subwordOffset,
                   UnitAttr:$unsignedA,
                   UnitAttr:$unsignedB,
                   UnitAttr:$clamp)>,
    Results<(outs WMMAOutTypes: $destD)> {
  let summary = "MLIR wrapper for wmma instructions";
  let description = [{
    The `amdgpu.wmma` op is an MLIR wrapper around intrinsics for various `wmma`
    instructions in the AMDGPU architecture, which perform matrix multiplication.

```
- **EN**: Introduces declarations for `AMDGPU_WMMAOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_WMMAOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1054-1068
```tablegen
    On gfx11/RDNA3, wmma intrinsics have M=N=K=16 dimensions.

    On gfx12/RDNA4, wmma intrinsics have M=N=16 dimensions and support K=16 for
    all element types, and K=32 for i4 sources.

    On gfx1250, wmma intrinsics have M=N=16 and K dimensions of 4, 32, 64, or 128,
    depending on the element types.

    On gfx11/RDNA3, emitting f16->f16 (or bf16->bf16) wmma the output is a 16xf16
    (or 16xbf16) vector containing only 8 valid values:
      - If `subwordOffset` is 0, then the output is stored at indices 0, 2, 4, ..., 14.
      - If `subwordOffset` is 1, then the output is stored at indices 1, 3, 5, ..., 15.
    On gfx12/RDNA4 and gfx1250, the result is instead returned as vector where all
    the values are valid and the `subwordOffset` must be `0`, as it cannot be used.

```
- **EN**: Implements logic around `f16`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `f16` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1069-1079
```tablegen
    `unsignedA` and `unsignedB` flag that the `int8` LLVM inputs are unsigned.

    The `clamp` flag is used to saturate the output of type T to `numeric_limits<T>::max()`
    in case of overflow.

    The `wave64`attribute indicates whether an op is designed for 64 threads wavefont.

    Example:
    ```mlir
      %0 = amdgpu.wmma 16x16x16 %matA * %matB + %matC : vector<8xf16>, vector<8xf16>, vector<8xf16>

```
- **EN**: Implements logic around `max`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `max` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1080-1094
```tablegen
      %1 = amdgpu.wmma 16x16x64 %matD * %matE + %matF : vector<32xi8>, vector<8xf32>, vector<8xf32>

      %2 = amdgpu.wmma 16x16x128 %matG * %matH + %matI : vector<64xf4E2M1FN>, vector<64xf4E2M1FN>, vector<8xf32>

      %3 = amdgpu.wmma 16x16x4 %matJ * %matK + %matL : vector<2xf32>, vector<2xf32>, vector<8xf32>
    ```
  }];
  let assemblyFormat = [{
    custom<MNKDimensionList>($m, $n, $k) $sourceA `*` $sourceB `+` $destC
    attr-dict
    `:` type($sourceA) `,` type($sourceB) `,` type($destC)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1095-1114
```tablegen
def DotInTypes : AnyTypeOf<[VectorOfLengthAndType<[2], [F16, BF16, I16]>,
                            VectorOfLengthAndType<[4], [I8, F8E4M3FN, F8E5M2]>,
                            VectorOfLengthAndType<[8], [I<4>]>]>;

def DotOutTypes : AnyTypeOf<[F32, F16, BF16, I32]>;

def AMDGPU_DotOp :
    AMDGPU_Op<"dot", [AllTypesMatch<["destC", "destD"]>, Pure]>,
    Arguments<(ins DotInTypes:$sourceA,
                   DotInTypes:$sourceB,
                   DotOutTypes:$destC,
                   UnitAttr:$unsignedA,
                   UnitAttr:$unsignedB,
                   UnitAttr:$clamp)>,
    Results<(outs DotOutTypes:$destD)> {
  let summary = "MLIR wrapper for AMDGPU v_dot* intrinsics";
  let description = [{
    The `amdgpu.dot` op is an MLIR wrapper over the `v_dot*` family of intrinsics,
    which compute `D = sum_i A[i] * B[i] + C`. 

```
- **EN**: Introduces declarations for `DotInTypes`, `DotOutTypes`, `AMDGPU_DotOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DotInTypes`, `DotOutTypes`, `AMDGPU_DotOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1115-1131
```tablegen
    Variants (source, dest, signedness, chipset -> intrinsic).

    ```text
    | A elem   | B elem   | destC | signedness | chipset                   | ROCDL op                     |
    |----------|----------|-------|------------|---------------------------|------------------------------|
    | f16      | f16      | f32   | n/a        | gfx906+                   | fdot2                        |
    | f16      | f16      | f16   | n/a        | gfx11+                    | fdot2.f16.f16                |
    | bf16     | bf16     | f32   | n/a        | gfx11+, gfx950+           | fdot2.f32.bf16               |
    | bf16     | bf16     | bf16  | n/a        | gfx11+                    | fdot2.bf16.bf16              |
    | i16      | i16      | i32   | s / u      | gfx906+, no gfx11+/gfx12+ | sdot2 / udot2                |
    | i8       | i8       | i32   | s / u      | gfx906+                   | sdot4 / udot4                |
    | i8       | i8       | i32   | mixed      | gfx11+                    | sudot4                       |
    | i4       | i4       | i32   | s / u      | gfx906+                   | sdot8 / udot8                |
    | i4       | i4       | i32   | mixed      | gfx11+                    | sudot8                       |
    | fp8/bf8  | fp8/bf8  | f32   | n/a        | gfx11.7, gfx12+           | dot4.f32.{fp8,bf8}.{fp8,bf8} |
    ```

```
- **EN**: Implements logic around `Variants`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Variants` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1132-1151
```tablegen
    Example:
    ```mlir
    %r0 = amdgpu.dot %a * %b + %c : vector<4xi8>, vector<4xi8>, i32
    %r1 = amdgpu.dot %a * %b + %c {unsignedA, unsignedB, clamp}
        : vector<8xi4>, vector<8xi4>, i32
    %r2 = amdgpu.dot %a * %b + %c {unsignedB}
        : vector<4xi8>, vector<4xi8>, i32
    %r3 = amdgpu.dot %a * %b + %c : vector<2xf16>, vector<2xf16>, f32
    %r4 = amdgpu.dot %a * %b + %c : vector<2xf16>, vector<2xf16>, f16
    %r5 = amdgpu.dot %a * %b + %c
        : vector<4xf8E4M3FN>, vector<4xf8E5M2>, f32
    ```
  }];
  let assemblyFormat = [{
    $sourceA `*` $sourceB `+` $destC attr-dict
      `:` type($sourceA) `,` type($sourceB) `,` type($destC)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1152-1171
```tablegen
def AMDGPU_SparseMFMAOp :
    AMDGPU_Op<"sparse_mfma", [AllTypesMatch<["destC", "destD"]>,
                              Pure]>,
    Arguments<(ins
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32]>]>:$m,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32]>]>:$n,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32, 64, 128]>]>:$k,
                   SMFMACSparseInTypes:$sourceA,
                   SMFMACDenseInTypes:$sourceB,
                   SMFMACOutTypes:$destC,
                   SMFMACIdxTypes:$sparseIdx,
                   DefaultValuedAttr<I32Attr, "0">:$cbsz,
                   DefaultValuedAttr<I32Attr, "0">:$abid)>,
    Results<(outs SMFMACOutTypes: $destD)> {
  let summary = "MLIR wrapper for CDNA sparse mfma (smfmac) instructions";
  let description = [{
    The `amdgpu.sparse_mfma` op is an MLIR wrapper around intrinsics for various
    `smfmac` instructions in the AMDGPU architecture, which perform matrix
    multiply-accumulate operations using 2:4 structured sparsity on matrix A
    with dense matrices B, C, and D.
```
- **EN**: Introduces declarations for `AMDGPU_SparseMFMAOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_SparseMFMAOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1172-1191
```tablegen

    On gfx942, smfmac intrinsics support:
      - M=N=16, K=32 and M=N=32, K=16 for f16 and bf16 sources
      - M=N=16, K=64 and M=N=32, K=32 for i8 and fp8 sources

    On gfx950, smfmac intrinsics additionally support:
      - M=N=16, K=64 and M=N=32, K=32 for f16 and bf16 sources
      - M=N=16, K=128 and M=N=32, K=64 for i8 and fp8 sources

    The `sparseIdx` parameter contains packed 2-bit indices identifying which
    of every 4 dense-K positions are non-zero in the 2:4 sparse matrix A.
    The required `sparseIdx` type depends on the variant:
      - gfx942 16-bit (`(m,k)` in `{(16,32), (32,16)}`): 8 bits per lane,
        carried as `vector<4xi8>` (one 8-bit set per i8 element).
      - gfx942 8-bit (`(m,k)` in `{(16,64), (32,32)}`) and gfx950 16-bit
        (`(m,k)` in `{(16,64), (32,32)}`): 16 bits per lane, carried as
        `vector<2xi16>` (one 16-bit set per i16 element).
      - gfx950 8-bit (`(m,k)` in `{(16,128), (32,64)}`): 32 bits per lane (a
        full VGPR with no internal set structure), carried as `i32`.

```
- **EN**: Implements logic around `bit`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bit` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1192-1204
```tablegen
    The `cbsz` and `abid` parameters select which index set within the VGPR is
    used:
      - gfx942 16-bit: `cbsz == 0` selects one of four 8-bit sets via
        `abid[1:0]` (range `[0, 3]`); `cbsz != 0` selects the first set.
      - gfx942 8-bit and gfx950 16-bit: `cbsz == 0` selects one of two 16-bit
        sets via `abid[0]` (range `[0, 1]`); `cbsz != 0` selects the first set.
      - gfx950 8-bit: hardware ignores both `cbsz` and `abid`; both must be 0.

    Example:
    ```mlir
      %0 = amdgpu.sparse_mfma 16x16x32 %matA * %matB + %matC sparse(%idx : vector<4xi8>)
        : vector<4xf16>, vector<8xf16>, vector<4xf32>

```
- **EN**: Declares APIs or declarative rules around `sparse`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `sparse` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1205-1224
```tablegen
      %1 = amdgpu.sparse_mfma 16x16x64 %matA * %matB + %matC sparse(%idx : vector<2xi16>)
        : vector<8xi8>, vector<16xi8>, vector<4xi32>

      %2 = amdgpu.sparse_mfma 16x16x64 %matA * %matB + %matC sparse(%idx : vector<2xi16>)
        { cbsz = 0 : i32, abid = 1 : i32 }
        : vector<8xf8E4M3FNUZ>, vector<16xf8E4M3FNUZ>, vector<4xf32>

      %3 = amdgpu.sparse_mfma 16x16x128 %matA * %matB + %matC sparse(%idx : i32)
        : vector<16xf8E4M3FN>, vector<32xf8E4M3FN>, vector<4xf32>
    ```
  }];
  let assemblyFormat = [{
    custom<MNKDimensionList>($m, $n, $k) $sourceA `*` $sourceB `+` $destC
    `sparse` `(` $sparseIdx `:` type($sparseIdx) `)`
    attr-dict
    `:` type($sourceA) `,` type($sourceB) `,` type($destC)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `sparse`, `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sparse`, `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1225-1234
```tablegen
// sparse_wmma (swmmac)
def SWMMACSparseInTypes : AnyTypeOf<[
    VectorOfLengthAndType<[4, 8, 16], [F16]>,
    VectorOfLengthAndType<[4, 8, 16], [BF16]>,
    VectorOfLengthAndType<[4, 8, 32], [I8]>,
    VectorOfLengthAndType<[8, 16], [I<4>]>,
    VectorOfLengthAndType<[4, 8, 16, 32], [F8E4M3FN, F8E5M2]>,
    VectorOfLengthAndType<[4, 8, 16, 32], [F8E4M3FNUZ, F8E5M2FNUZ]>
]>;

```
- **EN**: Introduces declarations for `SWMMACSparseInTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SWMMACSparseInTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1235-1244
```tablegen
def SWMMACDenseInTypes : AnyTypeOf<[
    VectorOfLengthAndType<[8, 16, 32], [F16]>,
    VectorOfLengthAndType<[8, 16, 32], [BF16]>,
    VectorOfLengthAndType<[4, 8, 16, 64], [I8]>,
    VectorOfLengthAndType<[8, 16, 32], [I<4>]>,
    VectorOfLengthAndType<[4, 8, 16, 64], [F8E4M3FN, F8E5M2]>,
    VectorOfLengthAndType<[4, 8, 16, 64], [F8E4M3FNUZ, F8E5M2FNUZ]>
]>;

def SWMMACOutTypes : AnyTypeOf<[
```
- **EN**: Introduces declarations for `SWMMACDenseInTypes`, `SWMMACOutTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SWMMACDenseInTypes`, `SWMMACOutTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1245-1254
```tablegen
    VectorOfLengthAndType<[4, 8, 16], [F32]>,
    VectorOfLengthAndType<[4, 8], [F16]>,
    VectorOfLengthAndType<[4, 8], [BF16]>,
    VectorOfLengthAndType<[4, 8], [I32]>
]>;

def SWMMACIdxTypes : AnyTypeOf<[
    FixedVectorOfLengthAndType<[4], [I8]>,
]>;

```
- **EN**: Introduces declarations for `SWMMACIdxTypes`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SWMMACIdxTypes` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1255-1274
```tablegen

def AMDGPU_SparseWMMAOp :
    AMDGPU_Op<"sparse_wmma", [AllTypesMatch<["destC", "destD"]>,
                              Pure]>,
    Arguments<(ins
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16]>]>:$m,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16]>]>:$n,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[32, 64, 128]>]>:$k,
                   SWMMACSparseInTypes:$sourceA,
                   SWMMACDenseInTypes:$sourceB,
                   SWMMACOutTypes:$destC,
                   SWMMACIdxTypes:$sparseIdx,
                   UnitAttr:$unsignedA,
                   UnitAttr:$unsignedB,
                   UnitAttr:$reuseA,
                   UnitAttr:$reuseB,
                   UnitAttr:$clamp,
                   UnitAttr:$wave64)>,
    Results<(outs SWMMACOutTypes: $destD)> {
  let summary = "MLIR wrapper for gfx12+ sparse wmma instructions";
```
- **EN**: Introduces declarations for `AMDGPU_SparseWMMAOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_SparseWMMAOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1275-1284
```tablegen
  let description = [{
    The `amdgpu.sparse_wmma` op is an MLIR wrapper around intrinsics for various
    `swmmac` instructions in the AMDGPU architecture, which perform matrix
    multiply-accumulate operations using 2:4 structured sparsity on matrix A
    with dense matrices B, C, and D.

    On gfx12, swmmac intrinsics support:
      - M=N=16, K=32 and M=N=32, K=16 for f16, bf16, i8 and i4 sources
      - M=N=16, K=64 for i4 sources

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1285-1295
```tablegen
    On gfx1250, swmmac intrinsics additionally support:
      - M=N=16, K=64 for f16 and bf16 sources
      - M=N=16, K=128 for f16, bf16 and i8 sources

    The `sparseIdx` parameter contains packed indices identifying the positions
    of non-zero elements in the 2:4 sparse matrix A. For 16-bit source data,
    use `vector<4xi8>` (four 8-bit indices). For 8-bit source data, use
    `vector<2xi16>` (two 16-bit indices).

    `unsignedA` and `unsignedB` flag that the `int8` LLVM inputs are unsigned.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1296-1306
```tablegen
    The `clamp` flag is used to saturate the output of type T to `numeric_limits<T>::max()`
    in case of overflow.

    Example:
    ```mlir
      %0 = amdgpu.sparse_wmma 16x16x32 %matA * %matB + %matC sparse(%idx : vector<4xi8>)
        : vector<4xf16>, vector<8xf16>, vector<4xf32>

      %1 = amdgpu.sparse_wmma 16x16x64 %matA * %matB + %matC sparse(%idx : vector<2xi16>)
        : vector<8xi8>, vector<16xi8>, vector<4xi32>

```
- **EN**: Implements logic around `max`, `sparse`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `max`, `sparse` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1307-1320
```tablegen
      %2 = amdgpu.sparse_wmma 16x16x64 %matA * %matB + %matC sparse(%idx : vector<2xi16>)
        { unsignedA = 0 : i1, unsignedB = 1 : i1, clamp = 0 : i1 }
        : vector<8xf8E4M3FNUZ>, vector<16xf8E4M3FNUZ>, vector<4xf32>
    ```
  }];
  let assemblyFormat = [{
    custom<MNKDimensionList>($m, $n, $k) $sourceA `*` $sourceB `+` $destC
    `sparse` `(` $sparseIdx `:` type($sparseIdx) `)`
    attr-dict
    `:` type($sourceA) `,` type($sourceB) `,` type($destC)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `sparse`, `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `sparse`, `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1321-1335
```tablegen
def AMDGPU_GatherToLDSOp :
    AMDGPU_Op<"gather_to_lds", [AttrSizedOperandSegments]>,
    Arguments<(ins
                   Arg<AnyMemRef, "buffer to gather from", [MemRead]>:$src,
                   Variadic<Index>:$srcIndices,
                   Arg<AnyMemRef, "buffer to write to", [MemWrite]>:$dst,
                   Variadic<Index>:$dstIndices,
                   TypeAttr:$transferType,
                   UnitAttr:$async
                   )>,
    Results<(outs)> {
  let summary = "MLIR wrapper for CDNA Gather to LDS instructions";
  let description = [{
    The `amdgpu.gather_to_lds` op is a wrapper around the `global_load_lds` instructions.

```
- **EN**: Introduces declarations for `AMDGPU_GatherToLDSOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_GatherToLDSOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1336-1355
```tablegen
    Operands:
    * `$src`: global memory (including fat buffer) memref to read from.
    * `$srcIndices`: indices into `$src` to read from for this thread.
    * `$dst`: LDS memory memref to write to.
    * `$dstIndices`: base indices into `$dst` to write to for the subgroup of this thread.
      The elements gathered by the subgroup will be written contiguously in order of lane ID
      starting at `$dst[$dstIndices]`. Byte-sized (ex. i8) or short-sized (ex. i16)
      types will be zero-padded/extended to 32 bits before being written. 96-bit types
      (ex. vector<3xf32>) will be zero-padded to 128 bits before being written. Only the
      offsets held by lane 0 are used.
    * `$transferType`: type of the data to be transferred by each thread. This is used to determine
      the size of the data to be transferred and the number of threads in the subgroup.
      The transfer type must be a scalar type or a vector type with a single element type.
    * If `$async` is set, the compiler will not attempt to infer the
      memory waits needed to ensure that the DMA operation has succeeded
      before a load that might access the stored-to LDS is performed.
      Instead, the `rocdl.asyncmark` and `rocdl.wait.asyncmark N`
      operations must be used to explicitly indicate the desired completion
      behavior. This enables more precise calculation of these waits at the
      cost of requiring user management of asynchrony.
```
- **EN**: Implements logic around `memory`, `sized`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `memory`, `sized` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1356-1368
```tablegen

    The `$dst`, along with its indices, points to the memory location the subgroup of this thread
    will write to.

    Note: only supported on gfx9 and gfx10.
  }];
  let assemblyFormat = [{
    (`async` $async^)? $src `[` $srcIndices `]` `,` $dst `[` $dstIndices `]` attr-dict `:` $transferType `,` type($src) `,` type($dst)
  }];
  let hasVerifier = 1;
  let hasCanonicalizer = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1369-1384
```tablegen
def AMDGPU_GlobalLoadAsyncToLDSOp :
    AMDGPU_Op<"global_load_async_to_lds", [AttrSizedOperandSegments]>,
    Arguments<(ins
                   Arg<AnyMemRef, "global memory to load from", [MemRead]>:$src,
                   Variadic<Index>:$srcIndices,
                   Arg<AnyMemRef, "LDS memory to write to", [MemWrite]>:$dst,
                   Variadic<Index>:$dstIndices,
                   TypeAttr:$transferType,
                   Optional<I1>: $mask
                   )>,
    Results<(outs)> {
  let summary = "MLIR wrapper for async global load to lds instructions";
  let description = [{
    AMDGPU wrapper for `global.load.async.to.lds` instructions, which performs
    asynchronous load of data from global memory into LDS while bypassing VGPRs.

```
- **EN**: Introduces declarations for `AMDGPU_GlobalLoadAsyncToLDSOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_GlobalLoadAsyncToLDSOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1385-1395
```tablegen
    * `$src`: global memory memref to read from (global addrspace only, no fat buffer).
    * `$srcIndices`: indices into `$src` for this thread's global read location.
    * `$dst`: LDS memref to write to (workgroup addrspace).
    * `$dstIndices`: indices into `$dst` for this thread's LDS write location.
    * `$transferType`: type of data to be transferred. Must be 8, 32, 64 or 128 bit scalar
     or vector type.
    * `$mask`: optional per-thread mask. When false, the thread's LDS write
      is masked off. The global read still occurs for all threads regardless of mask.

    Note: only supported on gfx1250 and later.

```
- **EN**: Implements logic around `from`, `to`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `from`, `to` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1396-1405
```tablegen
    Examples:
    ```mlir
      amdgpu.global_load_async_to_lds %src[%i, %j], %dst[%k, %l]
        : f32, memref<128x64xf32, #gpu.address_space<global>>,
          memref<64x64xf32, #gpu.address_space<workgroup>>

      amdgpu.global_load_async_to_lds %src[%i, %j], %dst[%k, %l]
        : vector<4xf32>, memref<128x64xf32, #gpu.address_space<global>>,
          memref<64x64xf32, #gpu.address_space<workgroup>>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1406-1418
```tablegen
      amdgpu.global_load_async_to_lds %src[%i], %dst[%j]
        : i8, memref<512xi8, #gpu.address_space<global>>,
          memref<256xi8, #gpu.address_space<workgroup>>
    ```
  }];
  let assemblyFormat = [{
    $src `[` $srcIndices `]` `,` $dst `[` $dstIndices `]`  (`,` $mask^)?
    attr-dict `:` $transferType `,` type($src) `,` type($dst)
  }];
  let hasVerifier = 1;
  let hasCanonicalizer = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1419-1432
```tablegen
def AMDGPU_TransposeLoadOp :
    AMDGPU_Op<"transpose_load", [SameVariadicOperandSize]>,
    Arguments<(ins Arg<AnyMemRef, "buffer to transpose load from", [MemRead]>:$src, Variadic<Index>:$srcIndices)>,
    Results<(outs AnyTypeOf<[AnyVectorOfNonZeroRank]>:$result)> {
  let summary = "MLIR wrapper for CDNA Transpose Load instructions";
  let description = [{
    The `amdgpu.transpose_load` op is a wrapper around the `ds_read_tr` instructions.
    The transpose load op represents a subgroup load from LDS memory,
    where the subgroup of threads collectively reads a matrix from the source
    memref, with each thread reading a vector of the matrix, and gets a transposed matrix
    in as the result. That is, each thread reads a vector of the col-major matrix at different
    indices, and the thread's read result is a vector of the corresponding row of the transposed
    matrix.

```
- **EN**: Introduces declarations for `AMDGPU_TransposeLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_TransposeLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1433-1444
```tablegen
    This op is a direct wrapper around the ROCDL `ds_read_tr` family intrinsics. Please refer
    to the CDNA4 ISA documentation for more details about its exact semantics.

    Format example:
    ```
    %0 = amdgpu.transpose_load %src[%srcIndices] : memref<128x256xf16> -> vector<4xf16>
    ```
    Operands:
    * `$src`: LDS memref to read from.
    * `$srcIndices`: indices into `$src` to read from for this thread.
    * `$result`: target register this transpose load instruction will write to.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1445-1464
```tablegen
    Note: Lowering is only supported on gfx950 and up.
  }];
  let assemblyFormat = [{
    $src `[` $srcIndices `]` attr-dict `:` type($src) `->` type($result)
  }];
  let hasVerifier = 1;
}

def AMDGPU_GlobalTransposeLoadOp :
    AMDGPU_Op<"global_transpose_load", [SameVariadicOperandSize]>,
    Arguments<(ins Arg<AnyMemRef, "buffer to transpose load from", [MemRead]>:$src,
                      Variadic<Index>:$srcIndices)>,
    Results<(outs AnyTypeOf<[
      FixedVectorOfLengthAndType<[8], [I8, F8E5M2FNUZ, F8E4M3FNUZ, F8E5M2,
                                       F8E4M3FN, F16, BF16, I16]>,
      FixedVectorOfLengthAndType<[16], [I<4>, F4E2M1FN, I<6>, F6E2M3FN, F6E3M2FN]>
    ]>:$result)> {
  let summary = "MLIR wrapper for global memory transpose load instructions";
  let description = [{
    The `amdgpu.global_transpose_load` op is a wrapper around the
```
- **EN**: Introduces declarations for `AMDGPU_GlobalTransposeLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_GlobalTransposeLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1465-1479
```tablegen
    `global_load_tr` family of instructions introduced in gfx1200.

    Each thread reads a column of a matrix stored in global memory and receives
    the corresponding row of the transposed matrix in its result register.
    The subgroup collectively performs a transpose of the tile.

    This op is a direct wrapper around the ROCDL `global.load.tr` family
    intrinsics. Refer to the ISA manual for exact semantics.

    Format example:
    ```mlir
    %0 = amdgpu.global_transpose_load %src[%i, %j]
           : memref<128x256xf16, #gpu.address_space<global>> -> vector<8xf16>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1480-1492
```tablegen
    Operands:
    * `$src`: Global address space memref to read from.
    * `$srcIndices`: indices into `$src` for this thread. Indices must be
      non-negative and in-bounds for the corresponding dimension of `$src`,
      matching the constraints of `memref.load`.
    * `$result`: register this transpose load instruction writes to.

    Valid (element bits, num elements) pairs:
    * (4, 16)  -> global_load_tr4_b64  (gfx1250+)
    * (6, 16)  -> global_load_tr6_b96  (gfx1250+)
    * (8, 8)   -> global_load_tr_b64   (gfx1200+)
    * (16, 8)  -> global_load_tr_b128  (gfx1200+)

```
- **EN**: Implements logic around `Valid`, `global_load_tr4_b64`, `global_load_tr6_b96`, `global_load_tr_b64`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Valid`, `global_load_tr4_b64`, `global_load_tr6_b96`, `global_load_tr_b64`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1493-1502
```tablegen
    Note: 8-bit and 16-bit element lowering requires gfx1200+.
          4-bit and 6-bit element lowering requires gfx1250+.
  }];
  let assemblyFormat = [{
    $src `[` $srcIndices `]` attr-dict `:` type($src) `->` type($result)
  }];
  let hasVerifier = 1;
}

def AMDGPU_ScaledMFMAOp :
```
- **EN**: Introduces declarations for `AMDGPU_ScaledMFMAOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_ScaledMFMAOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1503-1522
```tablegen
    AMDGPU_Op<"scaled_mfma", [AllTypesMatch<["destC", "destD"]>,
                        Pure]>,
    Arguments<(ins
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32]>]>:$m,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32]>]>:$n,
                   ConfinedAttr<I32Attr, [IntIsOneOf<[64, 128]>]>:$k,
                   ScaledMFMAInTypes:$sourceA,
                   ScaledMFMAInTypes:$sourceB,
                   ScaledMFMAOutTypes:$destC,
                   AnyTypeOf<[F8E8M0FNU, FixedVectorOfLengthAndType<[4], [F8E8M0FNU]>]>:$scalesA,
                   AnyTypeOf<[F8E8M0FNU, FixedVectorOfLengthAndType<[4], [F8E8M0FNU]>]>:$scalesB,
                   ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<3>]>:$scalesIdxA,
                   ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<3>]>:$scalesIdxB
                   )>,
    Results<(outs ScaledMFMAOutTypes: $destD)> {
  let summary = "MLIR wrapper for CDNA scaled mfma instructions";
  let description = [{
    The `amdgpu.scaled_mfma` op is an MLIR wrapper around intrinsics
    for various scaled versions of `mfma` instructions in the CDNA architecture, which
    perform multiple outer products in order to allow fast matrix multiplication.
```
- **EN**: Implements logic around `Arguments`, `Results`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `Arguments`, `Results` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1523-1534
```tablegen

    The wrapper will select an appropriate `mfma` instruction, if one is available,
    based on the provided `m`, `k`, `n`, and `nBlks` attributes, along with the
    types of the source and destination arguments.

    Note, this wrapper allows specifying `vector<4Kxi8>` arguments to MFMA
    intrinsics that take an integer type of width `4K`. For example,
    one can provide a `vector<4xi8>` as an argument to an MFMA instruction that
    logically takes 4 i8s but whose intrinsics are specified to take an i32.
    In these cases, the bytes in the vector will be concatenated in little-endian
    order (that is, v[0] will go to arg[7:0], v[1] to arg[15:8] and so on).

```
- **EN**: Implements logic around `order`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `order` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1535-1554
```tablegen
    This wrapper takes inspiration from `amdgpu.mfma`, but has some key differences:
    - `amdgpu.scaled_mfma` operates on fp4 (f4E2M1FN), fp6 (f6E2M3FN and f6E3M2FN) and
      fp8 (f8E4M3FN and f8E5M2) types using either M=N=16, K=128 or M=N=32, K=64 as
      their tile size.
    - `amdgpu.scaled_mfma` does not support broadcasting. So, `cbsz`, `abid`, and `blgp`
      are omitted from this wrapper.
    - The `negateA`, `negateB`, and `negateC` flags in `amdgpu.mfma` are only supported
      for double-precision operations on gfx94x and so are not included here.

    Example:
    ```mlir
      %0 = amdgpu.scaled_mfma 32x32x64 (%arg0[0] * %arg1) * (%arg0[1] * %arg1) + %arg2
        : vector<4xf8E8M0FNU>, vector<32xf6E2M3FN>, f8E8M0FNU, vector<32xf6E2M3FN>, vector<16xf32>
    ```
  }];
  let assemblyFormat = [{
    custom<MNKDimensionList>($m, $n, $k) ` `
    `(` $scalesA `[` $scalesIdxA `]` `*` $sourceA `)` `*`
    `(` $scalesB `[` $scalesIdxB `]` `*` $sourceB `)` `+` $destC
    attr-dict
```
- **EN**: Implements logic around `fp4`, `fp8`, `x32x64`, `custom`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `fp4`, `fp8`, `x32x64`, `custom` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1555-1574
```tablegen
    `:` type($scalesA) `,` type($sourceA) `,` type($scalesB) `,` type($sourceB) `,` type($destC)
  }];
  let hasCanonicalizer = 1;
}

def AMDGPU_ScaledWMMAOp
    : AMDGPU_Op<"scaled_wmma", [AllTypesMatch<["destC", "destD"]>, Pure]>,
      Arguments<(ins ConfinedAttr<I32Attr, [IntIsOneOf<[16, 32]>]>:$m,
          ConfinedAttr<I32Attr, [IntIsOneOf<[16]>]>:$n,
          ConfinedAttr<I32Attr, [IntIsOneOf<[128]>]>:$k,
          ScaledWMMAInTypes:$sourceA, ScaledWMMAInTypes:$sourceB,
          ScaledWMMAOutTypes:$destC,
          VectorOfLengthAndType<[4, 8], [F8E8M0FNU, F8E4M3FN]>:$scaleA,
          ConfinedAttr<I32Attr, [IntIsOneOf<[0, 16]>]>:$a_first_scale_lane,
          VectorOfLengthAndType<[4, 8], [F8E8M0FNU, F8E4M3FN]>:$scaleB,
          ConfinedAttr<I32Attr, [IntIsOneOf<[0, 16]>]>:$b_first_scale_lane)>,
      Results<(outs ScaledWMMAOutTypes:$destD)> {
  // TODO: E5M3FNU scales are supported, but there is not yet MLIR support for
  // this datatype. Once we have support for that, update the scaleA and scaleB
  // types here.
```
- **EN**: Introduces declarations for `AMDGPU_ScaledWMMAOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_ScaledWMMAOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1575-1584
```tablegen
  let summary = "MLIR wrapper for scaled wmma instructions";
  let description = [{
    The `amdgpu.scaled_wmma` op is an MLIR wrapper around intrinsics for scaled
    `wmma` instructions. These instructions perform matrix multiplication with
    per-block scaling of inputs, supporting fp4, fp6, and fp8 data formats.

    The scale instructions support a block size of 16 or 32 and two tile sizes:
    - 16x16x128 with mixed f8/f6/f4 formats (output: vector<8xf32>)
    - 32x16x128 with f4 format only (output: vector<16xf32>)

```
- **EN**: Implements logic around `formats`, `only`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `formats`, `only` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1585-1604
```tablegen
    Scale parameters (`scaleA`, `scaleB`) are small vectors of f8 scale values
    (either f8E8M0FNU, or f8E4M3FN) that are packed into i32/i64 values during
    lowering. Each lane can operate on 4 bytes (4 scale values), and the
    number of scales required for each matrix is determined by:
      num_scales_A = (M × K) / block_size
      num_scales_B = (N × K) / block_size

    The index attributes (`a_first_scale_lane`, `b_first_scale_lane`) select
    which lane to start reading scale values from (0 or 16):
    - For block size 32, 32 lanes across a single wave are used for the scale
    values. If the number of scales (num_scales_A or num_scales_B) can fit
    into half of the available lanes
    (i.e., num_scales / scales_per_lane == 16 (num_lanes)),
    then then first_scale_lane can be either 0 or 16. If all lanes are required
    for storing the scale values (num_scales / scales_per_lane == 32 (num_lanes)),
    then the first_scale_lane must be 0.
    - For block size 16, the same rules apply as above except that there are 64
    lanes across two waves that are used for the scale values. When
    num_scales / scales_per_lane == 32 (num lanes), then 16 lanes from each wave are used.
    first_scale_lane of 0 or 16 will decide which lanes are used for this. When
```
- **EN**: Implements logic around `parameters`, `bytes`, `attributes`, `from`, and 2 more symbols; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `parameters`, `bytes`, `attributes`, `from`, and 2 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 1605-1615
```tablegen
    num_scales / scales_per_lane == 64 (num_lanes), then first_scale_lane must
    be set to 0.

    Example:
    ```mlir
      // 16x16x128: fp8 inputs
      %0 = amdgpu.scaled_wmma 16x16x128 (%scaleVecA * %matA) * (%scaleVecB * %matB) + %matC
        {a_first_scale_lane = 0 : i32, b_first_scale_lane = 0 : i32}
        : vector<4xf8E8M0FNU>, vector<64xf8E4M3FN>,
        vector<4xf8E8M0FNU>, vector<64xf8E4M3FN>, vector<8xf32>

```
- **EN**: Implements logic around `x16x128`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `x16x128` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1616-1632
```tablegen
      // 32x16x128: fp4 inputs with different scale lanes
      %1 = amdgpu.scaled_wmma 32x16x128 (%scaleVecD * %matD) * (%scaleVecE * %matE) + %matF
        {a_first_scale_lane = 0 : i32, b_first_scale_lane = 16 : i32}
        : vector<8xf8E4M3FN>, vector<128xf4E2M1FN>,
        vector<8xf8E4M3FN>, vector<64xf4E2M1FN>, vector<16xf32>
    ```
  }];
  let assemblyFormat = [{
    custom<MNKDimensionList>($m, $n, $k) ` `
    `(` $scaleA `*` $sourceA `)` `*`
    `(` $scaleB `*` $sourceB `)` `+` $destC
    attr-dict
    `:` type($scaleA) `,` type($sourceA) `,` type($scaleB) `,` type($sourceB) `,` type($destC)
  }];
  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `x16x128`, `custom`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `x16x128`, `custom`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1633-1645
```tablegen
class AMDGPU_DmaBaseOp<string mnemonic, Type outType> :
    AMDGPU_Op<mnemonic, [Pure, AttrSizedOperandSegments, AllElementTypesMatch<["global", "lds"]>]>,
    Arguments<(ins Arg<AnyMemRef>:$global,
                   Variadic<Index>:$global_indices,
                   Arg<AnyMemRef>:$lds,
                   Variadic<Index>:$lds_indices)>,
    Results<(outs outType: $base)> {

  let assemblyFormat = [{
    $global `[` $global_indices `]` `,` $lds `[` $lds_indices `]` attr-dict `:` type($global) `,` type($lds) `->` type(results)
  }];
}

```
- **EN**: Introduces declarations for `AMDGPU_DmaBaseOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DmaBaseOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1646-1655
```tablegen
def AMDGPU_MakeGatherDmaBaseOp : AMDGPU_DmaBaseOp<"make_gather_dma_base", AMDGPU_TDMGatherBaseType> {
  let summary = "Pair of based addresses used when moving tiles between LDS and global memory.";

  let description = [{
    This operation creates a pair of addresses that will be used by `tensor_load_to_lds`
    and `tensor_store_from_lds`.

    This operation creates a value corresponding to the tensor descriptor (D#) group 0
    found in TensorLoadToLDSOp and TensorStoreFromLDSOp in the rocdl dialect.

```
- **EN**: Introduces declarations for `AMDGPU_MakeGatherDmaBaseOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_MakeGatherDmaBaseOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1656-1667
```tablegen
    Unlike `make_dma_base`, this operation returns `!amdgpu.tdm_gather_base<$element_type, $index_type>`
    which is only compatible with `make_gather_dma_descriptor`. Using the descriptor returned
    by `make_gather_dma_descriptor` will set the `tensor_load_to_lds` and `tensor_store_from_lds` to gather mode.

    ```mlir
      %base = amdgpu.make_gather_dma_base %global[%idx0, %idx1], %lds[%idx2, %idx3] : memref<64x64xi32>, memref<64x64xi32, #gpu.address_space<workgroup>> -> !amdgpu.tdm_gather_base<i32, i16>
      // %indices : i16
      %descriptor = amdgpu.make_gather_dma_descriptor %base[%indices] globalSize [2, 2] globalStride [2, 1] sharedSize [2, 2] : !amdgpu.tdm_gather_base<i32, i16>, i16 -> !amdgpu.tdm_descriptor
      amdgpu.tensor_load_to_lds %descriptor : !amdgpu.tdm_descriptor
    ```
  }];

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1668-1677
```tablegen
  let hasVerifier = 1;

  let extraClassDeclaration = [{
    static constexpr bool isGather() {
      return true;
    }
  }];
}


```
- **EN**: Implements logic around `isGather`.
- **CN**: 围绕 `isGather` 实现具体逻辑。

### Lines 1678-1687
```tablegen
def AMDGPU_MakeDmaBaseOp : AMDGPU_DmaBaseOp<"make_dma_base", AMDGPU_TDMBaseType> {

  let summary = "Pair of based addresses used when moving tiles between LDS and global memory.";
  let description = [{
    This operation creates a pair of addresses that will be used by tensor_load_to_lds
    and tensor_store_from_lds.

    This operation creates a value corresponding to the tensor descriptor (D#) group 0
    found in TensorLoadToLDSOp and TensorStoreFromLDSOp in the rocdl dialect.

```
- **EN**: Introduces declarations for `AMDGPU_MakeDmaBaseOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_MakeDmaBaseOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1688-1697
```tablegen
    For example:

    ```mlir
      %base = amdgpu.make_dma_base %global[%idx0, %idx1], %lds[%idx2, %idx3] : memref<64x64xi32>, memref<64x64xi32, #gpu.address_space<workgroup>> -> !amdgpu.tdm_base<i32>
      %descriptor = amdgpu.make_dma_descriptor %base globalSize [2, 2] globalStride [2, 1] sharedSize [2, 2] : !amdgpu.tdm_base<i32> -> !amdgpu.tdm_descriptor
      amdgpu.tensor_load_to_lds %descriptor : !amdgpu.tdm_descriptor
    ```

    to

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1698-1712
```tablegen
    ```mlir
      // pseudo-code
      %global_base = llvm.extractvalue %global_memref[1]
      %global_address = llvm.get_element_ptr ...

      %lds_base = llvm.extractvalue %lds_memref[1]
      %lds_address = llvm.get_element_ptr ...

      // Definition of %base
      %undef = llvm.mlir.undef : vector<4xi32>
      %v0 = llvm.insertelement %15, %undef[0] : vector<4xi32>
      %v1 = llvm.insertelement %lds_address, %v0[1] : vector<4xi32>
      %v2 = llvm.insertelement %global_address_low, %v1[2] : vector<4xi32>
      %base = llvm.insertelement %global_address_high, %v2[3] : vector<4xi32>

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1713-1727
```tablegen
      rocdl.tensor.load.to.lds %base, %dgroup1, %dgroup2, %dgroup3 cachepolicy 0 : vector<4xi32>, vector<8xi32>
    ```

    These tensor DMA operations were introduced in gfx1250.
  }];

  let hasVerifier = 1;

  let extraClassDeclaration = [{
    static constexpr bool isGather() {
      return false;
    }
  }];
}

```
- **EN**: Implements logic around `isGather`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isGather` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1728-1747
```tablegen
class AMDGPU_MakeDescriptorOp<string mnemonic> :
  AMDGPU_Op<mnemonic, [Pure, AttrSizedOperandSegments]>,
  Results<(outs AMDGPU_TDMDescriptorType: $desc)> {

  dag baseArgs = (ins
    Variadic<Index>: $global_dynamic_sizes,
    DenseI64ArrayAttr: $global_static_sizes,
    Variadic<Index>: $global_dynamic_strides,
    DenseI64ArrayAttr: $global_static_strides,
    Variadic<Index>: $shared_dynamic_sizes,
    DenseI64ArrayAttr: $shared_static_sizes,
    Optional<AMDGPU_ConcreteVector<I1, 16>>: $workgroup_mask,
    Optional<I1>: $early_timeout,
    Optional<I32>: $pad_amount,
    Optional<I32>: $pad_interval,
    Optional<MemRefOf<[AMDGPU_DsBarrierStateType]>>: $atomic_barrier_address,
    Variadic<Index>: $atomic_barrier_indices,
    Optional<Index>: $global_increment,
    Optional<I32>: $lds_increment,
    Optional<Index>: $iteration_count);
```
- **EN**: Introduces declarations for `AMDGPU_MakeDescriptorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_MakeDescriptorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1748-1757
```tablegen

  code extraClassDeclarationBase = [{
    int64_t getRank() {
      return getGlobalStaticSizes().size();
    }

    unsigned getElementTypeWidth() {
      return getBase().getType().getElementType().getIntOrFloatBitWidth();
    }

```
- **EN**: Implements logic around `getRank`, `getGlobalStaticSizes`, `getElementTypeWidth`, `getBase`.
- **CN**: 围绕 `getRank`, `getGlobalStaticSizes`, `getElementTypeWidth`, `getBase` 实现具体逻辑。

### Lines 1758-1769
```tablegen
    SmallVector<OpFoldResult> getMixedGlobalSizes() {
      return getMixedValues(getGlobalStaticSizes(), getGlobalDynamicSizes(), getContext());
    }

    SmallVector<OpFoldResult> getMixedGlobalStrides() {
      return getMixedValues(getGlobalStaticStrides(), getGlobalDynamicStrides(), getContext());
    }

    SmallVector<OpFoldResult> getMixedSharedSizes() {
      return getMixedValues(getSharedStaticSizes(), getSharedDynamicSizes(), getContext());
    }

```
- **EN**: Implements logic around `getMixedGlobalSizes`, `getMixedValues`, `getMixedGlobalStrides`, `getMixedSharedSizes`.
- **CN**: 围绕 `getMixedGlobalSizes`, `getMixedValues`, `getMixedGlobalStrides`, `getMixedSharedSizes` 实现具体逻辑。

### Lines 1770-1780
```tablegen
  }];

}

def AMDGPU_MakeGatherDmaDescriptorOp : AMDGPU_MakeDescriptorOp<"make_gather_dma_descriptor"> {
  dag args = (ins AMDGPU_TDMGatherBaseType: $base,
                  AnyTypeOf<[VectorOfMinMaxLengthAndType<1, 8, [I32]>,
                             VectorOfMinMaxLengthAndType<1, 16, [I16]>]>: $indices);
  let arguments = !con(args, baseArgs);
  let summary = "Make all descriptor groups needed by TensorLoadToLDS/TensorStoreFromLDS.";

```
- **EN**: Introduces declarations for `AMDGPU_MakeGatherDmaDescriptorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_MakeGatherDmaDescriptorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1781-1793
```tablegen
  let assemblyFormat = [{
    $base `[` $indices `]`
    `globalSize` custom<DynamicIndexList>($global_dynamic_sizes, $global_static_sizes)
    `globalStride` custom<DynamicIndexList>($global_dynamic_strides, $global_static_strides)
    `sharedSize` custom<DynamicIndexList>($shared_dynamic_sizes, $shared_static_sizes)
    ( `padShared` `(` $pad_amount^ `every` $pad_interval `)` )?
    ( `workgroupMask` $workgroup_mask^ ( `earlyTimeout` $early_timeout^)?)?
    ( `atomicBarrier` `(` $atomic_barrier_address^ `[` $atomic_barrier_indices `]`
                      `:` type($atomic_barrier_address) `)`)?
    ( `iterate` $global_increment^ `,` $lds_increment `,` $iteration_count )?
    attr-dict `:` qualified(type($base)) `,` type($indices) `->` type(results)
  }];

```
- **EN**: Implements logic around `custom`, `type`, `qualified`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type`, `qualified` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1794-1803
```tablegen
  let hasVerifier = 1;
  let hasFolder = 1;

  let extraClassDeclaration = extraClassDeclarationBase # [{
    static constexpr bool isGather() {
      return true;
    }
  }];
}

```
- **EN**: Implements logic around `isGather`.
- **CN**: 围绕 `isGather` 实现具体逻辑。

### Lines 1804-1813
```tablegen
def AMDGPU_MakeDmaDescriptorOp : AMDGPU_MakeDescriptorOp<"make_dma_descriptor"> {
  dag args = (ins AMDGPU_TDMBaseType: $base);
  let arguments = !con(args, baseArgs);
  let summary = "Make all descriptor groups needed by TensorLoadToLDS/TensorStoreFromLDS.";
  let description = [{
     Make all descriptor groups needed by tensor memory operations.

     The $base operand corresponds to the base pair addresses, one must be an address in LDS
     while the other must be a global memory location.

```
- **EN**: Introduces declarations for `AMDGPU_MakeDmaDescriptorOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_MakeDmaDescriptorOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1814-1823
```tablegen
     $global_{static/dynamic}_sizes determine the size of the tensor.
     $global_{static/dynamic}_strides determine the strides of the tensor.
     $shared_{static/dynamic}_sizes determines the size of the tile.

     $workgroup_mask broadcast load to workgroups inside of a workgroup cluster
     (0 = do not broadcast result to workgroup, 1 = broadcast result to workgroup). Ignored for stores.
     An all zeros mask is interpreted as a non-broadcasted load.

     $early_timeout return data to requesters as soon as cache supplies it.

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 1824-1837
```tablegen
     Padding can be applied to the LDS address when copying from memory to LDS,
     but not when copying from LDS to memory.
     The values in the padded target addresses remain the same as before the operation was applied.
     $pad_interval must be a power of two contained in [2, 256].
     $pad_amount must be a value contained in [1, 128].

     If an atomic barrier is provided, it will be arrived at **once** after
     each load/store using this descriptor is completed.

     2D and 3D tensors may be iterated over by setting $global_increment, $lds_increment, and $iteration_count.
     $global_increment determines how much to increment the starting global memory address per iteration in units of the $base's element type.
     $lds_increment determines how much to increment the starting LDS address per iteration in units of the $base's element type.
     $iterate_count determines how many times to iterate, it must be a value in the inclusive interval [1, 256].

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1838-1850
```tablegen
     ```mlir
      // Example of moving a two-dimensional tensor to LDS.
      %base = amdgpu.make_dma_base %global[0, 0], %lds[0, 0] : memref<64x64xi32>, memref<64x64xi32, #gpu.address_space<workgroup>> -> !amdgpu.tdm_base<i32>
      %descriptor = amdgpu.make_dma_descriptor %base globalSize [64, 64] globalStride [64, 1] sharedSize [64, 64] : !amdgpu.tdm_base<i32> -> !amdgpu.tdm_descriptor
      amdgpu.tensor_load_to_lds %descriptor : !amdgpu.tdm_descriptor

      // Example of moving a two dimension tensor to LDS where padding is applied after every integer.
      %base = amdgpu.make_dma_base %global[0, 0], %lds[0, 0] : memref<32x32xi32>, memref<64x64xi32, #gpu.address_space<workgroup>> -> !amdgpu.tdm_base<i32>
      %descriptor = amdgpu.make_dma_descriptor %base globalSize [32, 32] globalStride [32, 1] sharedSize [64, 64] padShared(%pad_amount every %pad_interval) : !amdgpu.tdm_base<i32> -> !amdgpu.tdm_descriptor
      amdgpu.tensor_load_to_lds %descriptor : !amdgpu.tdm_descriptor
     ```
  }];

```
- **EN**: Declares APIs or declarative rules around `padShared`; this block touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `padShared` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1851-1863
```tablegen
  let assemblyFormat = [{
    $base
    `globalSize` custom<DynamicIndexList>($global_dynamic_sizes, $global_static_sizes)
    `globalStride` custom<DynamicIndexList>($global_dynamic_strides, $global_static_strides)
    `sharedSize` custom<DynamicIndexList>($shared_dynamic_sizes, $shared_static_sizes)
    ( `padShared` `(` $pad_amount^ `every` $pad_interval `)` )?
    ( `workgroupMask` $workgroup_mask^ ( `earlyTimeout` $early_timeout^)?)?
    ( `atomicBarrier` `(` $atomic_barrier_address^ `[` $atomic_barrier_indices `]`
                      `:` type($atomic_barrier_address) `)`)?
    ( `iterate` $global_increment^ `,` $lds_increment `,` $iteration_count )?
    attr-dict `:` qualified(type($base)) `->` type(results)
  }];

```
- **EN**: Implements logic around `custom`, `type`, `qualified`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `custom`, `type`, `qualified` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1864-1874
```tablegen
  let hasVerifier = 1;
  let hasFolder = 1;

  let extraClassDeclaration = extraClassDeclarationBase # [{
    static constexpr bool isGather() {
      return false;
    }
  }];

}

```
- **EN**: Implements logic around `isGather`.
- **CN**: 围绕 `isGather` 实现具体逻辑。

### Lines 1875-1884
```tablegen
def AMDGPU_TensorLoadToLDSOp :
  AMDGPU_Op<"tensor_load_to_lds", [MemoryEffects<[MemWrite, MemRead]>]>,
  Arguments<(ins AMDGPU_TDMDescriptorType: $desc)> {
  let summary = "Load tensors from global memory to LDS.";
  let description = [{
    Load tensors of up to five dimensions from global memory to LDS.

    This operation was introduced in gfx1250.
  }];

```
- **EN**: Introduces declarations for `AMDGPU_TensorLoadToLDSOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_TensorLoadToLDSOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1885-1897
```tablegen
  let assemblyFormat = [{
    $desc attr-dict `:` qualified(type($desc))
  }];
}

def AMDGPU_TensorStoreFromLDSOp :
  AMDGPU_Op<"tensor_store_from_lds", [MemoryEffects<[MemWrite, MemRead]>]>,
  Arguments<(ins AMDGPU_TDMDescriptorType: $desc)> {

  let summary = "Store tensors from LDS to global memory.";
  let description = [{
    Store tensors of up to five dimensions from LDS to global memory.

```
- **EN**: Introduces declarations for `AMDGPU_TensorStoreFromLDSOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_TensorStoreFromLDSOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1898-1912
```tablegen
    This operation was introduced in gfx1250.
  }];

  let assemblyFormat = [{
    $desc attr-dict `:` qualified(type($desc))
  }];
}

//===----------------------------------------------------------------------===//
// In-LDS Barrier Operations
//
// General note: any of these operations that impact memory have read and write
// effects as a crude model of their atomic nature - we don't want "reads"
// being hoisted out of loops.
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `qualified`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `qualified` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 1913-1925
```tablegen

def AMDGPU_DsBarrierInitOp :
    AMDGPU_Op<"ds_barrier_init">,
    Arguments<(ins Arg<MemRefOf<[AMDGPU_DsBarrierStateType]>, "barrier(s)",
                       [MemRead, MemWrite]>:$base,
                   Variadic<Index>:$indices,
                   I32:$participants)> {
  let summary = "Initialize an in-LDS barrier.";
  let description = [{
    Given the location `!amdgpu.ds_barrier_state` in LDS (as specified by `base` and `indices`),
    initialize the barrier structure so that the pending and init counts are equal to
    `participants - 1`, which will have its high bits masked off, and its phase is equal to 0.

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierInitOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierInitOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1926-1935
```tablegen
    Note that we subtract 1 from `participants` when constructing the barrier state
    to provide clearer high-level semantics.

    The subtraction means that, when the `participant`th arrival occurs, the phase will change.
    In practical terms, this means that you can use (for example) the number of subgroups or
    waves per workgroup as `participants`, instead of manually needing to remove one.

    While the write of the initial state will be performed atomically, no synchronization
    between waves will be performed by this operation.

```
- **EN**: Implements logic around `use`.
- **CN**: 围绕 `use` 实现具体逻辑。

### Lines 1936-1947
```tablegen
    Example:
    ```mlir
    amdgpu.ds_barrier_init %barrier[], %c32 : memref<!amdgpu.ds_barrier_state, #gpu.address_space<workgroup>>, i32
    ```

    This operation is only available on gfx1250+.
  }];

  let assemblyFormat = [{
    $base `[` $indices `]` `,` $participants attr-dict `:` type($base) `,` type($participants)
  }];

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1948-1960
```tablegen
  let hasVerifier = 1;
}

def AMDGPU_DsBarrierPollStateOp :
    AMDGPU_Op<"ds_barrier_poll_state">,
    Arguments<(ins Arg<MemRefOf<[AMDGPU_DsBarrierStateType]>, "barrier(s)",
                       [MemRead, MemWrite]>:$base,
                 Variadic<Index>:$indices)>,
    Results<(outs AMDGPU_DsBarrierStateType:$out)> {
  let summary = "Atomically read the state of an in-LDS barrier.";
  let description = [{
    Atomically read and return the state of the barrier at `base[indices...]`.

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierPollStateOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierPollStateOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1961-1971
```tablegen
    This will ultimately act like a `memref.load`, but this operation will ensure
    that appropriate atomic orderings and syncscopes are set.

    Example:
    ```mlir
    %state = amdgpu.ds_barrier_poll_state %barrier[] : memref<!amdgpu.ds_barrier_state, #gpu.address_space<workgroup>> -> !amdgpu.ds_barrier_state
    ```

    This operation is only available on gfx1250+.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 1972-1988
```tablegen
  let assemblyFormat = [{
    $base `[` $indices `]` attr-dict `:` type($base) `->` type($out)
  }];

  let hasVerifier = 1;
}

def AMDGPU_DsAsyncBarrierArriveOp :
    AMDGPU_Op<"ds_async_barrier_arrive">,
    Arguments<(ins Arg<MemRefOf<[AMDGPU_DsBarrierStateType]>, "barrier(s)",
                       [MemRead, MemWrite]>:$base,
                 Variadic<Index>:$indices)> {
  let summary = "Asynchronously arrive at an in-LDS barrier.";
  let description = [{
    Add a arrival to the LDS barrier at `base[indices]` to the sequence of pending
    asynchronous memory operations.

```
- **EN**: Introduces declarations for `AMDGPU_DsAsyncBarrierArriveOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsAsyncBarrierArriveOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 1989-2001
```tablegen
    This will add an "asynchronous memory operation" to the in-order list of pending
    asynchronous loads from global memory to LDS. When the queue of such operations
    issued before this operation is complete, the specified barrier will be arrived at,
    decrementing the pending count by 1 **per lane that executes it** and rolling
    over the phase if applicable.

    This operation does not return the old barrier state.

    Example:
    ```mlir
    amdgpu.ds_async_barrier_arrive %barrier[] : memref<!amdgpu.ds_barrier_state, #gpu.address_space<workgroup>>
    ```

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 2002-2011
```tablegen
    This operation is only available on gfx1250+.
  }];

  let assemblyFormat = [{
    $base `[` $indices `]` attr-dict `:` type($base)
  }];

  let hasVerifier = 1;
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 2012-2023
```tablegen
def AMDGPU_DsBarrierArriveOp :
    AMDGPU_Op<"ds_barrier_arrive">,
    Arguments<(ins Arg<MemRefOf<[AMDGPU_DsBarrierStateType]>, "barrier(s)",
                       [MemRead, MemWrite]>:$base,
                 Variadic<Index>:$indices,
                 I64:$count)>,
    Results<(outs AMDGPU_DsBarrierStateType:$out)> {
  let summary = "Arrive at an in-LDS barrier and return old state.";
  let description = [{
    Atomically arrive at the LDS barrier at `base[indices]` and decrement it by `count`,
    rolling over the phase if needed and returning the old barrier state.

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierArriveOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierArriveOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2024-2034
```tablegen
    `count` is the number of participants that should be subtracted from the barrier's
    pending count **per lane that executes the operation**.

    Example:
    ```mlir
    %old_state = amdgpu.ds_barrier_arrive %barrier[], %c1 : memref<!amdgpu.ds_barrier_state, #gpu.address_space<workgroup>>, i64 -> !amdgpu.ds_barrier_state
    ```

    This operation is only available on gfx1250+.
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 2035-2049
```tablegen
  let assemblyFormat = [{
    $base `[` $indices `]` `,` $count attr-dict `:` type($base) `,` type($count) `->` type($out)
  }];

  let hasVerifier = 1;
}

def AMDGPU_DsBarrierStatePhaseOp :
    AMDGPU_Op<"ds_barrier_state_phase", [Pure]>,
    Arguments<(ins AMDGPU_DsBarrierStateType:$state)>,
    Results<(outs I32:$res)> {
  let summary = "Extract the phase of a barrier state.";
  let description = [{
    Extract the phase of the `!amdgpu.ds_barrier_state` `state` as a 32-bit value.

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierStatePhaseOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierStatePhaseOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2050-2060
```tablegen
    Example:
    ```mlir
    %phase = amdgpu.ds_barrier_state_phase %state : !amdgpu.ds_barrier_state -> i32
    ```
  }];

  let assemblyFormat = [{
    $state attr-dict `:` type($state) `->` type($res)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2061-2074
```tablegen
def AMDGPU_DsBarrierStatePendingCountOp :
    AMDGPU_Op<"ds_barrier_state_pending_count", [Pure]>,
    Arguments<(ins AMDGPU_DsBarrierStateType:$state)>,
    Results<(outs I32:$res)> {
  let summary = "Extract the pending count of a barrier state.";
  let description = [{
    Extract the pending count of the `!amdgpu.ds_barrier_state` `state` as a 32-bit value.

    Example:
    ```mlir
    %pending = amdgpu.ds_barrier_state_pending_count %state : !amdgpu.ds_barrier_state -> i32
    ```
  }];

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierStatePendingCountOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierStatePendingCountOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2075-2087
```tablegen
  let assemblyFormat = [{
    $state attr-dict `:` type($state) `->` type($res)
  }];
}

def AMDGPU_DsBarrierStateInitCountOp :
    AMDGPU_Op<"ds_barrier_state_init_count", [Pure]>,
    Arguments<(ins AMDGPU_DsBarrierStateType:$state)>,
    Results<(outs I32:$res)> {
  let summary = "Extract the init count of a barrier state.";
  let description = [{
    Extract the init count of the `!amdgpu.ds_barrier_state` `state` as a 32-bit value.

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierStateInitCountOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierStateInitCountOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2088-2098
```tablegen
    Example:
    ```mlir
    %init = amdgpu.ds_barrier_state_init_count %state : !amdgpu.ds_barrier_state -> i32
    ```
  }];

  let assemblyFormat = [{
    $state attr-dict `:` type($state) `->` type($res)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2099-2110
```tablegen
def AMDGPU_DsBarrierStatePhaseParity :
    AMDGPU_Op<"ds_barrier_state_phase_parity", [Pure]>,
    Arguments<(ins AMDGPU_DsBarrierStateType:$state)>,
    Results<(outs I1:$res)> {
  let summary = "Extract the phase parity of a barrier state.";
  let description = [{
    Return the parity of the phase of the `!amdgpu.ds_barrier_state` `state`.

    This is intended to simplify the case where the barrier is being used to repeatedly
    track completion of a task where the precise value of the phase won't mater, only that
    it changed since (or as a result of) the arrival.

```
- **EN**: Introduces declarations for `AMDGPU_DsBarrierStatePhaseParity`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_DsBarrierStatePhaseParity` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2111-2121
```tablegen
    Example:
    ```mlir
    %parity = amdgpu.ds_barrier_state_phase_parity %state : !amdgpu.ds_barrier_state -> i1
    ```
  }];

  let assemblyFormat = [{
    $state attr-dict `:` type($state) `->` type($res)
  }];
}

```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 2122-2139
```tablegen
def AMDGPU_GlobalPrefetchOp :
    AMDGPU_Op<"global_prefetch", [MemoryEffects<[MemWrite, MemRead]>]>,
    Arguments<(ins AnyMemRef:$src,
               Variadic<I64>:$indices,
               AMDGPU_LoadTemporalHintAttr:$temporalHint,
               AMDGPU_CacheScopeAttr:$cacheScope,
               UnitAttr:$speculative)>,
    Results<(outs)> {

  let summary = "Prefetch data to caches.";
  let description = [{
    Prefetches a cache line to high-level caches using the aligned address of
    the source `memref` and an offset provided by the indices of the element
    containing the cache line. This provides temporal hints (e.g., regular
    or high-priority). Note that out-of-bounds access is allowed in
    speculative mode. The provided memref must be in the global address space
    (`#gpu.address_space<global>` or 1).

```
- **EN**: Introduces declarations for `AMDGPU_GlobalPrefetchOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AMDGPU_GlobalPrefetchOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 2140-2151
```tablegen
    This operation was introduced in gfx1250.

    Example:
    ```mlir
    amdgpu.global_prefetch %src[%i, %j] RT SE speculative : memref<64x64xf16, #gpu.address_space<global>>
    ```
  }];

  let assemblyFormat = [{
    $src `[` $indices `]` $temporalHint $cacheScope (`speculative` $speculative^)? attr-dict `:` qualified(type($src))
  }];

```
- **EN**: Implements logic around `cacheScope`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `cacheScope` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 2152-2155
```tablegen
  let hasVerifier = 1;
}

#endif // MLIR_DIALECT_AMDGPU_IR_AMDGPUOPS_TD
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Describes the declarative or C++ interface for dialect operations, attributes, types, and registration helpers.
  - **CN**: 描述方言操作、属性、类型及注册辅助逻辑的声明式或 C++ 接口。
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

- **TableGen includes / TableGen 包含**: `mlir/Dialect/AMDGPU/IR/AMDGPUBase.td`, `mlir/Dialect/AMDGPU/IR/AMDGPUAttrs.td`, `mlir/Dialect/AMDGPU/IR/AMDGPUTypes.td`, `mlir/Interfaces/InferTypeOpInterface.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/Interfaces/ViewLikeInterface.td`, `mlir/IR/OpBase.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (3), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
