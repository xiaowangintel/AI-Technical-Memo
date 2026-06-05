# ArmSVE.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/ArmSVE/IR/ArmSVE.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the basic operations for the ArmSVE dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/ArmSVE/IR`，围绕 ArmSVE 方言公开 `ArmSVE` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ArmSVE.td - ArmSVE dialect operation definitions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the basic operations for the ArmSVE dialect.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 12-21
```tablegen

#ifndef ARMSVE_OPS
#define ARMSVE_OPS

include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/Dialect/LLVMIR/LLVMOpBase.td"

//===----------------------------------------------------------------------===//
// ArmSVE dialect definition
//===----------------------------------------------------------------------===//
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 22-31
```tablegen

def ArmSVE_Dialect : Dialect {
  let name = "arm_sve";
  let cppNamespace = "::mlir::arm_sve";
  let summary = "Basic dialect to target Arm SVE architectures";
  let description = [{
    This dialect contains the definitions necessary to target specific Arm SVE
    scalable vector operations.
  }];

```
- **EN**: Introduces declarations for `ArmSVE_Dialect`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSVE_Dialect` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 32-44
```tablegen
  let dependentDialects = ["vector::VectorDialect"];
}

//===----------------------------------------------------------------------===//
// ArmSVE type definitions
//===----------------------------------------------------------------------===//

def SVBool : ScalableVectorOfRankAndLengthAndType<
  [1], [16], [I1]>
{
  let summary = "vector<[16]xi1>";
}

```
- **EN**: Introduces declarations for `SVBool`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SVBool` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 45-55
```tablegen
def SVEPredicate : ScalableVectorOfRankAndLengthAndType<
  [1], [16, 8, 4, 2, 1], [I1]>
{
  let summary = "vector<[1]xi1>, vector<[2]xi1>, vector<[4]xi1>, vector<[8]xi1>, or vector<[16]xi1>";
}

// Generalizations of SVBool and SVEPredicate to ranks >= 1.
// These are masks with a single trailing scalable dimension.
def SVBoolMask : VectorWithTrailingDimScalableOfSizeAndType<
  [16], [I1]>;
def SVEPredicateMask : VectorWithTrailingDimScalableOfSizeAndType<
```
- **EN**: Introduces declarations for `SVEPredicate`, `SVBoolMask`, `SVEPredicateMask`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SVEPredicate`, `SVBoolMask`, `SVEPredicateMask` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 56-70
```tablegen
  [16, 8, 4, 2, 1], [I1]>;

// A constraint for a 1-D scalable vector of `length`.
class Scalable1DVectorOfLength<int length, list<Type> elementTypes> : ShapedContainerType<
  elementTypes, And<[IsVectorOfShape<[length]>, IsVectorTypeWithAnyDimScalablePred]>,
  "a 1-D scalable vector with length " # length,
  "::mlir::VectorType">;

def SVEVector : AnyTypeOf<[
  Scalable1DVectorOfLength<2, [I64, F64]>,
  Scalable1DVectorOfLength<4, [I32, F32]>,
  Scalable1DVectorOfLength<8, [I16, F16, BF16]>,
  Scalable1DVectorOfLength<16, [I8]>],
  "an SVE vector with element size <= 64-bit">;

```
- **EN**: Introduces declarations for `Scalable1DVectorOfLength`, `SVEVector`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `Scalable1DVectorOfLength`, `SVEVector` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 71-90
```tablegen
//===----------------------------------------------------------------------===//
// ArmSVE op definitions
//===----------------------------------------------------------------------===//

class ArmSVE_Op<string mnemonic, list<Trait> traits = []> :
  Op<ArmSVE_Dialect, mnemonic, traits> {}

class ArmSVE_IntrOp<string mnemonic,
                    list<Trait> traits = [],
                    list<int> overloadedOperands = [],
                    list<int> overloadedResults = [],
                    int numResults = 1,
                    list<int> immArgPositions = [],
                    list<string> immArgAttrNames = [],
                    string enumName = "aarch64_sve_" # !subst(".", "_", mnemonic) > :
  LLVM_IntrOpBase</*Dialect dialect=*/ArmSVE_Dialect,
                  /*string opName=*/"intr." # mnemonic,
                  /*string enumName=*/enumName,
                  /*list<int> overloadedResults=*/overloadedResults,
                  /*list<int> overloadedOperands=*/overloadedOperands,
```
- **EN**: Introduces declarations for `ArmSVE_Op`, `ArmSVE_IntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSVE_Op`, `ArmSVE_IntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 91-100
```tablegen
                  /*list<Trait> traits=*/traits,
                  /*int numResults=*/numResults,
                  /*bit requiresAccessGroup=*/0,
                  /*bit requiresAliasAnalysis=*/0,
                  /*bit requiresFastmath=*/0,
                  /*bit requiresArgAndResultAttrs=*/0,
                  /*bit requiresOpBundles=*/0,
                  /*list<int> immArgPositions=*/immArgPositions,
                  /*list<string> immArgAttrNames=*/immArgAttrNames>;

```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 101-120
```tablegen
class ArmSVE_IntrBinaryOverloadedOp<string mnemonic,
                                    list<Trait> traits = []>:
  ArmSVE_IntrOp<mnemonic, traits,
    /*overloadedOperands=*/[], /*overloadedResults=*/[0]>;

class ScalableMaskedFOp<string mnemonic, string op_description,
                        list<Trait> traits = []> :
  ArmSVE_Op<mnemonic, !listconcat(traits,
                       [AllTypesMatch<["src1", "src2", "res"]>,
                        TypesMatchWith<
                          "mask has i1 element type and same shape as operands",
                          "src1", "mask", "getI1SameShape($_self)">])> {
  let summary = "masked " # op_description # " for scalable vectors of floats";
  let description = [{
    The `arm_sve.}] # mnemonic # [{` operation takes one scalable vector mask
    and two scalable vector operands, and perform floating point }] #
    op_description # [{ on active lanes. Inactive lanes will keep the value of
    the first operand.}];
  let arguments = (ins
          ScalableVectorOfAnyRank<[I1]>:$mask,
```
- **EN**: Introduces declarations for `ArmSVE_IntrBinaryOverloadedOp`, `ScalableMaskedFOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSVE_IntrBinaryOverloadedOp`, `ScalableMaskedFOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 121-140
```tablegen
          ScalableVectorOfAnyRank<[AnyFloat]>:$src1,
          ScalableVectorOfAnyRank<[AnyFloat]>:$src2
  );
  let results = (outs ScalableVectorOfAnyRank<[AnyFloat]>:$res);
  let assemblyFormat =
    "$mask `,` $src1 `,` $src2 attr-dict `:` type($mask) `,` type($res)";
}

class ScalableMaskedIOp<string mnemonic, string op_description,
                        list<Trait> traits = []> :
  ArmSVE_Op<mnemonic, !listconcat(traits,
                       [AllTypesMatch<["src1", "src2", "res"]>,
                        TypesMatchWith<
                          "mask has i1 element type and same shape as operands",
                          "src1", "mask", "getI1SameShape($_self)">])> {
  let summary = "masked " # op_description # " for scalable vectors of integers";
  let description = [{
    The `arm_sve.}] # mnemonic # [{` operation takes one scalable vector mask
    and two scalable vector operands, and perform integer }] #
    op_description # [{ on active lanes. Inactive lanes will keep the value of
```
- **EN**: Introduces declarations for `ScalableMaskedIOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedIOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 141-151
```tablegen
    the first operand.}];
  let arguments = (ins
          ScalableVectorOfAnyRank<[I1]>:$mask,
          ScalableVectorOfAnyRank<[I8, I16, I32, I64]>:$src1,
          ScalableVectorOfAnyRank<[I8, I16, I32, I64]>:$src2
  );
  let results = (outs ScalableVectorOfAnyRank<[I8, I16, I32, I64]>:$res);
  let assemblyFormat =
    "$mask `,` $src1 `,` $src2 attr-dict `:` type($mask) `,` type($res)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 152-164
```tablegen
def SdotOp : ArmSVE_Op<"sdot", [Pure,
                                AllTypesMatch<["src1", "src2"]>,
                                AllTypesMatch<["acc", "dst"]>]> {
  let summary = "Vector-vector dot product and accumulate op";
  let description = [{
    SDOT: Signed integer addition of dot product.

    This function maps to the SDOT instruction, and it takes signless integer
    operands that the operation interprets as signed. It partitions the second
    and third vector inputs into groups of four elements. They calculate the dot
    product of each group (without loss of precision) and then add each result
    to the overlapping element of the first vector input.

```
- **EN**: Introduces declarations for `SdotOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SdotOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 165-180
```tablegen
    Source:
    https://developer.arm.com/documentation/100987/0000
  }];
  // Supports either:
  //   (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  //   (vector<8xi16>. vector<8xi16>) -> (vector<2xi64>)
  let arguments = (ins
          ScalableVectorOfLengthAndType<[4, 2], [I32, I64]>:$acc,
          ScalableVectorOfLengthAndType<[16, 8], [I8, I16]>:$src1,
          ScalableVectorOfLengthAndType<[16, 8], [I8, I16]>:$src2
  );
  let results = (outs ScalableVectorOfLengthAndType<[4, 2], [I32, I64]>:$dst);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($dst)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 181-196
```tablegen
def SmmlaOp : ArmSVE_Op<"smmla", [Pure,
                                  AllTypesMatch<["src1", "src2"]>,
                                  AllTypesMatch<["acc", "dst"]>]> {
  let summary = "Matrix-matrix multiply and accumulate op";
  let description = [{
    SMMLA: Signed integer matrix multiply-accumulate.

    This function maps to the SMMLA instruction, and it takes signless integer
    operands that the operation interprets as signed. It partitions the inputs
    into 128-bit quadwords, with the first input containing a row-by-row 2×2
    matrix of 32-bit integers, the second input containing a row-by-row 2×8
    matrix of 8-bit integers, and the third input containing a column-by-column
    8×2 matrix of 8-bit integers. For each quadword, they multiply the second
    input matrix by the third input matrix using natural arithmetic and then add
    the result to the first input using modular arithmetic.

```
- **EN**: Introduces declarations for `SmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 197-210
```tablegen
    Source:
    https://developer.arm.com/documentation/100987/0000
  }];
  // Supports (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  let arguments = (ins
          ScalableVectorOfLengthAndType<[4], [I32]>:$acc,
          ScalableVectorOfLengthAndType<[16], [I8]>:$src1,
          ScalableVectorOfLengthAndType<[16], [I8]>:$src2
  );
  let results = (outs ScalableVectorOfLengthAndType<[4], [I32]>:$dst);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($dst)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 211-223
```tablegen
def UdotOp : ArmSVE_Op<"udot", [Pure,
                                AllTypesMatch<["src1", "src2"]>,
                                AllTypesMatch<["acc", "dst"]>]> {
  let summary = "Vector-vector dot product and accumulate op";
  let description = [{
    UDOT: Unsigned integer addition of dot product.

    This function maps to the UDOT instruction, and it takes signless integer
    operands that the operation interprets as unsigned. It partitions the second
    and third vector inputs into groups of four elements. They calculate the dot
    product of each group (without loss of precision) and then add each result
    to the overlapping element of the first vector input.

```
- **EN**: Introduces declarations for `UdotOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UdotOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 224-239
```tablegen
    Source:
    https://developer.arm.com/documentation/100987/0000
  }];
  // Supports either:
  //   (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  //   (vector<8xi16>. vector<8xi16>) -> (vector<2xi64>)
  let arguments = (ins
          ScalableVectorOfLengthAndType<[4, 2], [I32, I64]>:$acc,
          ScalableVectorOfLengthAndType<[16, 8], [I8, I16]>:$src1,
          ScalableVectorOfLengthAndType<[16, 8], [I8, I16]>:$src2
  );
  let results = (outs ScalableVectorOfLengthAndType<[4, 2], [I32, I64]>:$dst);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($dst)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 240-255
```tablegen
def UmmlaOp : ArmSVE_Op<"ummla", [Pure,
                                  AllTypesMatch<["src1", "src2"]>,
                                  AllTypesMatch<["acc", "dst"]>]> {
  let summary = "Matrix-matrix multiply and accumulate op";
  let description = [{
    UMMLA: Unsigned integer matrix multiply-accumulate.

    This function maps to the UMMLA instruction, and it takes signless integer
    operands that the operation interprets as unsigned. It partitions the inputs
    into 128-bit quadwords, with the first input containing a row-by-row 2×2
    matrix of 32-bit integers, the second input containing a row-by-row 2×8
    matrix of 8-bit integers, and the third input containing a column-by-column
    8×2 matrix of 8-bit integers. For each quadword, they multiply the second
    input matrix by the third input matrix using natural arithmetic and then add
    the result to the first input using modular arithmetic.

```
- **EN**: Introduces declarations for `UmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 256-269
```tablegen
    Source:
    https://developer.arm.com/documentation/100987/0000
  }];
  // Supports (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  let arguments = (ins
          ScalableVectorOfLengthAndType<[4], [I32]>:$acc,
          ScalableVectorOfLengthAndType<[16], [I8]>:$src1,
          ScalableVectorOfLengthAndType<[16], [I8]>:$src2
  );
  let results = (outs ScalableVectorOfLengthAndType<[4], [I32]>:$dst);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($dst)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 270-283
```tablegen
def UsmmlaOp : ArmSVE_Op<"usmmla", [Pure,
                                    AllTypesMatch<["src1", "src2"]>,
                                    AllTypesMatch<["acc", "dst"]>]> {
  let summary = "Matrix-matrix multiply and accumulate op";
  let description = [{
    USMMLA: Unsigned by signed integer matrix multiply-accumulate.

    The unsigned by signed integer matrix multiply-accumulate operation
    multiplies the 2×8 matrix of unsigned 8-bit integer values held
    the first source vector by the 8×2 matrix of signed 8-bit integer
    values in the second source vector. The resulting 2×2 widened 32-bit
    integer matrix product is then added to the 32-bit integer matrix
    accumulator.

```
- **EN**: Introduces declarations for `UsmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UsmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 284-297
```tablegen
    Source:
    https://developer.arm.com/documentation/100987/0000
  }];
  // Supports (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  let arguments = (ins
          ScalableVectorOfLengthAndType<[4], [I32]>:$acc,
          ScalableVectorOfLengthAndType<[16], [I8]>:$src1,
          ScalableVectorOfLengthAndType<[16], [I8]>:$src2
  );
  let results = (outs ScalableVectorOfLengthAndType<[4], [I32]>:$dst);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($dst)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 298-311
```tablegen
def BfmmlaOp : ArmSVE_IntrOp<"bfmmla", [Pure,
                                        AllTypesMatch<["src1", "src2"]>,
                                        AllTypesMatch<["acc", "res"]>,
                                        ],
                                        /*overloadedOperands=*/[1],
                                        /*overloadedResults=*/[0],
                                        /*numResults=*/1,
                                        /*immArgPositions=*/[],
                                        /*immArgAttrNames=*/[],
                                        /*enumName=*/"aarch64_sve_fmmla"> {
  let summary = "BFloat16 matrix multiply-accumulate";
  let description = [{
    BFMMLA: BFloat16 matrix multiply-accumulate into 2×2 matrices";

```
- **EN**: Introduces declarations for `BfmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BfmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 312-331
```tablegen
    This operation multiplies the 2x4 BFloat16 matrix held in each 128-bit
    segment of the first source vector by the 4x2 BFloat16 matrix in the
    corresponding segment of the second source vector, then accumulates
    this intermediate result with the 2x2 Float32 matrix in the corresponding
    segment of the accumulator vector, yielding the final 2x2 Float32
    segment of the result.

    Source:
    https://developer.arm.com/documentation/100987/0000
  }];
  // Supports (vector<[8]xbf16>, vector<[8]xbf16>) -> (vector<[4]xf32>)
  let arguments = (ins
          ScalableVectorOfLengthAndType<[4], [F32]>:$acc,
          ScalableVectorOfLengthAndType<[8], [BF16]>:$src1,
          ScalableVectorOfLengthAndType<[8], [BF16]>:$src2
  );
  let results = (outs ScalableVectorOfLengthAndType<[4], [F32]>:$res);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($res)";
}
```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 332-346
```tablegen

class SvboolTypeConstraint<string lhsArg, string rhsArg> : TypesMatchWith<
      "expected corresponding svbool type widened to [16]xi1",
      lhsArg, rhsArg,
      "VectorType(VectorType::Builder(::llvm::cast<VectorType>($_self)).setDim(::llvm::cast<VectorType>($_self).getRank() - 1, 16))">;

def ConvertFromSvboolOp : ArmSVE_Op<"convert_from_svbool",
                                    [Pure,
                                     SvboolTypeConstraint<"result", "source">]> {
  let summary = "Convert a svbool type to a SVE predicate type";
  let description = [{
    Converts svbool types (`vector<[16]xi1>` or vectors of that type, e.g.
    `vector<2x3x[16]xi1>`) to SVE predicate types. Note: Only the trailing
    dimension can be scalable.

```
- **EN**: Introduces declarations for `SvboolTypeConstraint`, `ConvertFromSvboolOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SvboolTypeConstraint`, `ConvertFromSvboolOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 347-358
```tablegen
    Example 1: Convert a 1-D svbool mask to a SVE predicate.
    ```mlir
    %source = vector.load %memref[%c0] : memref<?xi1>, vector<[16]xi1>
    %result = arm_sve.convert_from_svbool %source : vector<[4]xi1>
    ```

    Example 2: Convert a 2-D svbool mask to a mask of SVE predicates.
    ```mlir
    %source = vector.load %memref[%c0, %c0] : memref<2x?xi1>, vector<2x[16]xi1>
    %result = arm_sve.convert_from_svbool %source : vector<2x[8]xi1>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 359-371
```tablegen
    ---

    A `svbool` is the smallest SVE predicate type that has a in-memory
    representation (and maps to a full predicate register). In MLIR `svbool` is
    represented as `vector<[16]xi1>`. Smaller SVE predicate types
    (`vector<[1|2|4|8]xi1>`) must be stored as a `svbool` then converted back to
    the original predicate type after loading.
  }];
  let arguments = (ins SVBoolMask:$source);
  let results = (outs SVEPredicateMask:$result);
  let assemblyFormat = "$source attr-dict `:` type($result)";
}

```
- **EN**: Declares APIs or declarative rules around `representation`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `representation`, `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 372-387
```tablegen
def ConvertToSvboolOp : ArmSVE_Op<"convert_to_svbool",
                                  [Pure,
                                   SvboolTypeConstraint<"source", "result">]> {
  let summary = "Convert a SVE predicate type to a svbool type";
  let description = [{
    Converts SVE predicate types (or vectors of predicate types, e.g.
    `vector<4x[4]xi1>`) to svbool types. Note: Only the trailing dimension can
    be scalable.

    Example 1: Convert a 1-D SVE predicate to a svbool mask.
    ```mlir
    %source = vector.create_mask %dim_size : vector<[4]xi1>
    %result = arm_sve.convert_to_svbool %source : vector<[4]xi1>
    // => Results in vector<[16]xi1>
    ```

```
- **EN**: Introduces declarations for `ConvertToSvboolOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ConvertToSvboolOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 388-407
```tablegen
    Example 2: Convert a 2-D mask of SVE predicates to a svbool mask.
    ```mlir
    %source = vector.create_mask %c2, %dim_size : vector<2x[2]xi1>
    %result = arm_sve.convert_to_svbool %source : vector<2x[2]xi1>
    // => Results in vector<2x[16]xi1>
    ```

    ---

    A `svbool` is the smallest SVE predicate type that has a in-memory
    representation (and maps to a full predicate register). In MLIR `svbool` is
    represented as `vector<[16]xi1>`. Smaller SVE predicate types
    (`vector<[1|2|4|8]xi1>`) must be converted to a `svbool` before they can be
    stored.
  }];
  let arguments = (ins SVEPredicateMask:$source);
  let results = (outs SVBoolMask:$result);
  let assemblyFormat = "$source attr-dict `:` type($source)";
}

```
- **EN**: Declares APIs or declarative rules around `representation`, `type`; this block coordinates dialect conversion or lowering decisions; handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `representation`, `type` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 408-420
```tablegen
// Inputs valid for the multi-vector zips (not including the 128-bit element zipqs)
def ZipInputVectorType : AnyTypeOf<[
  Scalable1DVectorOfLength<2, [I64, F64]>,
  Scalable1DVectorOfLength<4, [I32, F32]>,
  Scalable1DVectorOfLength<8, [I16, F16, BF16]>,
  Scalable1DVectorOfLength<16, [I8]>],
  "an SVE vector with element size <= 64-bit">;

def ZipX2Op : ArmSVE_Op<"zip.x2", [Pure,
                                   AllTypesMatch<["sourceV1", "sourceV2",
                                                  "resultV1", "resultV2"]>]> {
  let summary = "Multi-vector two-way zip op";

```
- **EN**: Introduces declarations for `ZipInputVectorType`, `ZipX2Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ZipInputVectorType`, `ZipX2Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 421-433
```tablegen
  let description = [{
    This operation interleaves elements from two input SVE vectors, returning
    two new SVE vectors (`resultV1` and `resultV2`), which contain the low and
    high halves of the result respectively.

    Example:
    ```mlir
    // sourceV1 = [ A1, A2, A3, ... An ]
    // sourceV2 = [ B1, B2, B3, ... Bn ]
    // (resultV1, resultV2) = [ A1, B1, A2, B2, A3, B3, ... An, Bn ]
    %resultV1, %resultV2 = arm_sve.zip.x2 %sourceV1, %sourceV2 : vector<[16]xi8>
    ```

```
- **EN**: Implements logic around `vectors`; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `vectors` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 434-444
```tablegen
    Note: This requires SME 2 (`+sme2` in LLVM target features)

    [Source](https://developer.arm.com/documentation/ddi0602/2023-12/SME-Instructions/ZIP--two-registers---Interleave-elements-from-two-vectors-?lang=en)
  }];

  let arguments = (ins ZipInputVectorType:$sourceV1,
                       ZipInputVectorType:$sourceV2);

  let results = (outs ZipInputVectorType:$resultV1,
                      ZipInputVectorType:$resultV2);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 445-458
```tablegen
  let builders = [
    OpBuilder<(ins "Value":$v1, "Value":$v2), [{
      build($_builder, $_state, v1.getType(), v1.getType(), v1, v2);
  }]>];

  let assemblyFormat = "$sourceV1 `,` $sourceV2 attr-dict `:` type($sourceV1)";

  let extraClassDeclaration = [{
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getSourceV1().getType());
    }
  }];
}

```
- **EN**: Implements logic around `OpBuilder`, `build`, `type`, `getVectorType`, and 1 more symbols; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder`, `build`, `type`, `getVectorType`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 459-471
```tablegen
def ZipX4Op
  : ArmSVE_Op<"zip.x4",
              [Pure,
               AllTypesMatch<["sourceV1", "sourceV2", "sourceV3", "sourceV4",
                              "resultV1", "resultV2", "resultV3", "resultV4"]>]> {
  let summary = "Multi-vector four-way zip op";

  let description = [{
    This operation interleaves elements from four input SVE vectors, returning
    four new SVE vectors, each of which contain a quarter of the result. The
    first quarter will be in `resultV1`, second in `resultV2`, third in
    `resultV3`, and fourth in `resultV4`.

```
- **EN**: Introduces declarations for `ZipX4Op`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ZipX4Op` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 472-482
```tablegen
    ```mlir
    // sourceV1 = [ A1, A2, ... An ]
    // sourceV2 = [ B1, B2, ... Bn ]
    // sourceV3 = [ C1, C2, ... Cn ]
    // sourceV4 = [ D1, D2, ... Dn ]
    // (resultV1, resultV2, resultV3, resultV4)
    //   = [ A1, B1, C1, D1, A2, B2, C2, D2, ... An, Bn, Cn, Dn ]
    %resultV1, %resultV2, %resultV3, %resultV4 = arm_sve.zip.x4
      %sourceV1, %sourceV2, %sourceV3, %sourceV4 : vector<[16]xi8>
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 483-495
```tablegen
    **Warning:** The result of this op is undefined for 64-bit elements on
    hardware with less than 256-bit vectors!

    Note: This requires SME 2 (`+sme2` in LLVM target features)

    [Source](https://developer.arm.com/documentation/ddi0602/2023-12/SME-Instructions/ZIP--four-registers---Interleave-elements-from-four-vectors-?lang=en)
  }];

  let arguments = (ins ZipInputVectorType:$sourceV1,
                       ZipInputVectorType:$sourceV2,
                       ZipInputVectorType:$sourceV3,
                       ZipInputVectorType:$sourceV4);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 496-508
```tablegen
  let results = (outs ZipInputVectorType:$resultV1,
                      ZipInputVectorType:$resultV2,
                      ZipInputVectorType:$resultV3,
                      ZipInputVectorType:$resultV4);

  let builders = [
    OpBuilder<(ins "Value":$v1, "Value":$v2, "Value":$v3, "Value":$v4), [{
      build($_builder, $_state,
        v1.getType(), v1.getType(),
        v1.getType(), v1.getType(),
        v1, v2, v3, v4);
  }]>];

```
- **EN**: Implements logic around `OpBuilder`, `build`, `getType`.
- **CN**: 围绕 `OpBuilder`, `build`, `getType` 实现具体逻辑。

### Lines 509-520
```tablegen
  let assemblyFormat = [{
    $sourceV1 `,` $sourceV2 `,` $sourceV3 `,` $sourceV4 attr-dict
      `:` type($sourceV1)
  }];

  let extraClassDeclaration = [{
    VectorType getVectorType() {
      return ::llvm::cast<VectorType>(getSourceV1().getType());
    }
  }];
}

```
- **EN**: Implements logic around `type`, `getVectorType`, `getSourceV1`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `type`, `getVectorType`, `getSourceV1` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 521-532
```tablegen
def PselOp : ArmSVE_Op<"psel", [Pure, AllTypesMatch<["p1", "result"]>]> {
  let summary = "Predicate select";

  let description = [{
    This operation returns the input predicate `p1` or an all-false predicate
    based on the bit at `p2[index]`. Informally, the semantics are:
    ```
    if p2[index % num_elements(p2)] == 1:
      return p1 : type(p1)
    return all-false : type(p1)
    ```

```
- **EN**: Introduces declarations for `PselOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `PselOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 533-544
```tablegen
    Example:
    ```mlir
    // Note: p1 and p2 can have different sizes.
    %pd = arm_sve.psel %p1, %p2[%index] : vector<[4]xi1>, vector<[8]xi1>
    ```

    Note: This requires SME or SVE2.1 (`+sme` or `+sve2p1` in LLVM target features).
  }];

  let arguments = (ins SVEPredicate:$p1, SVEPredicate:$p2, Index:$index);
  let results = (outs SVEPredicate:$result);

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 545-554
```tablegen
  let builders = [
    OpBuilder<(ins "Value":$p1, "Value":$p2, "Value":$index), [{
      build($_builder, $_state, p1.getType(), p1, p2, index);
  }]>];

  let assemblyFormat = [{
    $p1 `,` $p2 `[` $index `]` attr-dict `:` type($p1) `,` type($p2)
  }];
}

```
- **EN**: Implements logic around `OpBuilder`, `build`, `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `OpBuilder`, `build`, `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 555-564
```tablegen
def ScalableMaskedAddIOp : ScalableMaskedIOp<"masked.addi", "addition",
                                             [Commutative]>;

def ScalableMaskedAddFOp : ScalableMaskedFOp<"masked.addf", "addition",
                            [Commutative]>;

def ScalableMaskedSubIOp : ScalableMaskedIOp<"masked.subi", "subtraction">;

def ScalableMaskedSubFOp : ScalableMaskedFOp<"masked.subf", "subtraction">;

```
- **EN**: Introduces declarations for `ScalableMaskedAddIOp`, `ScalableMaskedAddFOp`, `ScalableMaskedSubIOp`, `ScalableMaskedSubFOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedAddIOp`, `ScalableMaskedAddFOp`, `ScalableMaskedSubIOp`, `ScalableMaskedSubFOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 565-574
```tablegen
def ScalableMaskedMulIOp : ScalableMaskedIOp<"masked.muli", "multiplication",
                            [Commutative]>;

def ScalableMaskedMulFOp : ScalableMaskedFOp<"masked.mulf", "multiplication",
                            [Commutative]>;

def ScalableMaskedSDivIOp : ScalableMaskedIOp<"masked.divi_signed",
                                              "signed division">;

def ScalableMaskedUDivIOp : ScalableMaskedIOp<"masked.divi_unsigned",
```
- **EN**: Introduces declarations for `ScalableMaskedMulIOp`, `ScalableMaskedMulFOp`, `ScalableMaskedSDivIOp`, `ScalableMaskedUDivIOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedMulIOp`, `ScalableMaskedMulFOp`, `ScalableMaskedSDivIOp`, `ScalableMaskedUDivIOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 575-587
```tablegen
                                              "unsigned division">;

def ScalableMaskedDivFOp : ScalableMaskedFOp<"masked.divf", "division">;

def DupQLaneOp : ArmSVE_Op<"dupq_lane", [Pure, AllTypesMatch<["src", "dst"]>]> {
  let summary = "Broadcast indexed 128-bit segment to vector";

  let description = [{
    This operation fills each 128-bit segment of a vector with the elements
    from the indexed 128-bit segment of the source vector. If the VL is
    128 bits the operation is a NOP. If the index exceeds the number of
    128-bit segments in a vector the result is an all-zeroes vector.

```
- **EN**: Introduces declarations for `ScalableMaskedDivFOp`, `DupQLaneOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedDivFOp`, `DupQLaneOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 588-599
```tablegen
    Example:
    ```mlir
    // VL == 256
    // %X = [A B C D x x x x]
    %Y = arm_sve.dupq_lane %X[0] : vector<[4]xi32>
    // Y = [A B C D A B C D]

    // %U = [x x x x x x x x A B C D E F G H]
    %V = arm_sve.dupq_lane %U[1] : vector<[8]xf16>
    // %V = [A B C D E F H A B C D E F H]
    ```

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 600-612
```tablegen
    Note: The semantics of the operation match those of the `svdupq_lane` intrinsics.
    [Source](https://developer.arm.com/architectures/instruction-sets/intrinsics/#q=svdupq_lane)
  }];

  let arguments = (ins SVEVector:$src,
                       I64Attr:$lane);
  let results = (outs SVEVector:$dst);

  let builders = [
    OpBuilder<(ins "Value":$src, "int64_t":$lane), [{
      build($_builder, $_state, src.getType(), src, lane);
    }]>];

```
- **EN**: Implements logic around `OpBuilder`, `build`.
- **CN**: 围绕 `OpBuilder`, `build` 实现具体逻辑。

### Lines 613-622
```tablegen
  let assemblyFormat = [{
    $src `[` $lane `]` attr-dict `:` type($dst)
  }];
}

def UmmlaIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"ummla">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def SmmlaIntrOp :
```
- **EN**: Introduces declarations for `UmmlaIntrOp`, `SmmlaIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UmmlaIntrOp`, `SmmlaIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 623-633
```tablegen
  ArmSVE_IntrBinaryOverloadedOp<"smmla">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def UsmmlaIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"usmmla">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def SdotIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"sdot">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

```
- **EN**: Introduces declarations for `UsmmlaIntrOp`, `SdotIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UsmmlaIntrOp`, `SdotIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 634-645
```tablegen
def UdotIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"udot">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ScalableMaskedAddIIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"add">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ScalableMaskedAddFIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"fadd">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

```
- **EN**: Introduces declarations for `UdotIntrOp`, `ScalableMaskedAddIIntrOp`, `ScalableMaskedAddFIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UdotIntrOp`, `ScalableMaskedAddIIntrOp`, `ScalableMaskedAddFIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 646-657
```tablegen
def ScalableMaskedMulIIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"mul">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ScalableMaskedMulFIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"fmul">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ScalableMaskedSubIIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"sub">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

```
- **EN**: Introduces declarations for `ScalableMaskedMulIIntrOp`, `ScalableMaskedMulFIntrOp`, `ScalableMaskedSubIIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedMulIIntrOp`, `ScalableMaskedMulFIntrOp`, `ScalableMaskedSubIIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 658-669
```tablegen
def ScalableMaskedSubFIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"fsub">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ScalableMaskedSDivIIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"sdiv">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ScalableMaskedUDivIIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"udiv">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

```
- **EN**: Introduces declarations for `ScalableMaskedSubFIntrOp`, `ScalableMaskedSDivIIntrOp`, `ScalableMaskedUDivIIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedSubFIntrOp`, `ScalableMaskedSDivIIntrOp`, `ScalableMaskedUDivIIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 670-680
```tablegen
def ScalableMaskedDivFIntrOp :
  ArmSVE_IntrBinaryOverloadedOp<"fdiv">,
  Arguments<(ins AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank, AnyScalableVectorOfAnyRank)>;

def ConvertFromSvboolIntrOp :
  ArmSVE_IntrOp<"convert.from.svbool",
    [TypeIs<"res", SVEPredicate>],
    /*overloadedOperands=*/[],
    /*overloadedResults=*/[0]>,
  Arguments<(ins SVBool:$svbool)>;

```
- **EN**: Introduces declarations for `ScalableMaskedDivFIntrOp`, `ConvertFromSvboolIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ScalableMaskedDivFIntrOp`, `ConvertFromSvboolIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 681-696
```tablegen
def ConvertToSvboolIntrOp :
  ArmSVE_IntrOp<"convert.to.svbool",
    [TypeIs<"res", SVBool>],
    /*overloadedOperands=*/[0],
    /*overloadedResults=*/[]>,
    Arguments<(ins SVEPredicate:$mask)>;

// Note: This multi-vector intrinsic requires SME2.
def ZipX2IntrOp : ArmSVE_IntrOp<"zip.x2",
    /*traits=*/[],
    /*overloadedOperands=*/[0],
    /*overloadedResults=*/[],
    /*numResults=*/2>,
    Arguments<(ins Arg<AnyScalableVectorOfAnyRank, "v1">:$v1,
                   Arg<AnyScalableVectorOfAnyRank, "v2">:$v2)>;

```
- **EN**: Introduces declarations for `ConvertToSvboolIntrOp`, `ZipX2IntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ConvertToSvboolIntrOp`, `ZipX2IntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 697-707
```tablegen
// Note: This multi-vector intrinsic requires SME2.
def ZipX4IntrOp : ArmSVE_IntrOp<"zip.x4",
    /*traits=*/[],
    /*overloadedOperands=*/[0],
    /*overloadedResults=*/[],
    /*numResults=*/4>,
    Arguments<(ins Arg<AnyScalableVectorOfAnyRank, "v1">:$v1,
                   Arg<AnyScalableVectorOfAnyRank, "v2">:$v2,
                   Arg<AnyScalableVectorOfAnyRank, "v3">:$v3,
                   Arg<AnyScalableVectorOfAnyRank, "v3">:$v4)>;

```
- **EN**: Introduces declarations for `ZipX4IntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ZipX4IntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 708-722
```tablegen
// Note: This intrinsic requires SME or SVE2.1.
def PselIntrOp : ArmSVE_IntrOp<"psel",
  /*traits=*/[Pure, TypeIs<"res", SVBool>],
  /*overloadedOperands=*/[1]>,
  Arguments<(ins Arg<SVBool, "p1">:$p1,
                 Arg<SVEPredicate, "p2">:$p2,
                 Arg<I32, "index">:$index)>;

def WhileLTIntrOp :
  ArmSVE_IntrOp<"whilelt",
    [TypeIs<"res", SVEPredicate>, Pure],
    /*overloadedOperands=*/[0],
    /*overloadedResults=*/[0]>,
  Arguments<(ins I64:$base, I64:$n)>;

```
- **EN**: Introduces declarations for `PselIntrOp`, `WhileLTIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `PselIntrOp`, `WhileLTIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 723-732
```tablegen
def DupQLaneIntrOp : ArmSVE_IntrOp<"dupq_lane",
    /*traits=*/[],
    /*overloadedOperands=*/[0],
    /*overloadedResults=*/[],
    /*numResults=*/1,
    /*immArgPositions*/[1],
    /*immArgAttrNames*/["lane"]>,
    Arguments<(ins Arg<ScalableVectorOfRank<[1]>, "v">:$v,
                   Arg<I64Attr, "lane">:$lane)>;

```
- **EN**: Introduces declarations for `DupQLaneIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `DupQLaneIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 733-733
```tablegen
#endif // ARMSVE_OPS
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

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/Dialect/LLVMIR/LLVMOpBase.td`
- **Subsystem categories / 子系统类别**: cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (1)
