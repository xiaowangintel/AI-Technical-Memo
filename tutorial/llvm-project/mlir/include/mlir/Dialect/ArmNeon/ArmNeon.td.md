# ArmNeon.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/ArmNeon/ArmNeon.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the basic operations for the ArmNeon dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/ArmNeon`，围绕 ArmNeon 方言公开 `ArmNeon` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- ArmNeonOps.td - ArmNeon dialect op definitions -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```tablegen
//
// This file defines the basic operations for the ArmNeon dialect.
//
//===----------------------------------------------------------------------===//

#ifndef ARMNEON_OPS
#define ARMNEON_OPS

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-22
```tablegen
include "mlir/Dialect/LLVMIR/LLVMOpBase.td"
include "mlir/Interfaces/SideEffectInterfaces.td"
include "mlir/IR/OpBase.td"

//===----------------------------------------------------------------------===//
// ArmNeon dialect definition
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 23-32
```tablegen

def ArmNeon_Dialect : Dialect {
  let name = "arm_neon";
  let cppNamespace = "::mlir::arm_neon";

  // Note: this does not need to depend on LLVMDialect as long as functions in
  // this dialect (such as canonicalization) do not produce entities belonging
  // to the LLVMDialect (ops or types).
}

```
- **EN**: Introduces declarations for `ArmNeon_Dialect`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmNeon_Dialect` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 33-41
```tablegen
//===----------------------------------------------------------------------===//
// ArmNeon type definition
//===----------------------------------------------------------------------===//

class NeonVectorOfLength<int length, Type elementType> : ShapedContainerType<
  [elementType], And<[IsVectorOfShape<[length]>, IsFixedVectorOfAnyRankTypePred]>,
  "a vector with length " # length,
  "::mlir::VectorType">;

```
- **EN**: Introduces declarations for `NeonVectorOfLength`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NeonVectorOfLength` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 42-48
```tablegen
//===----------------------------------------------------------------------===//
// ArmNeon op definitions
//===----------------------------------------------------------------------===//

// ArmNeon dialect op that corresponds (and is convertible to) an LLVM IR
// intrinsic.
class ArmNeon_IntrOp<string mnemonic, list<int> overloadedResults,
```
- **EN**: Introduces declarations for `ArmNeon_IntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmNeon_IntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 49-60
```tablegen
                     list<int> overloadedOperands, int numResults,
                     list<Trait> traits = [], bit requiresAccessGroup = 0,
                     bit requiresAliasAnalysis = 0>
    : LLVM_IntrOpBase</*dialect=*/ArmNeon_Dialect,
                      /*opName=*/"intr." # mnemonic,
                      /*enumName=*/"aarch64_neon_" # !subst(".", "_", mnemonic),
                      /*overloadedResults=*/overloadedResults,
                      /*overloadedOperands=*/overloadedOperands,
                      /*traits=*/traits,
                      /*numResults=*/numResults,
                      /*requiresAccessGroup=*/requiresAccessGroup,
                      /*requiresAliasAnalysis=*/requiresAliasAnalysis>;
```
- **EN**: Declares APIs or declarative rules around `subst`.
- **CN**: 声明与 `subst` 相关的 API 或声明式规则。

### Lines 61-67
```tablegen

// ArmNeon dialect op that corresponds to an LLVM IR intrinsic with one
// overloaded result.
class ArmNeon_OverloadedOneResultIntrOp<string mnemonic,
                                        list<Trait> traits = []>
  : ArmNeon_IntrOp<mnemonic, [0], [], 1, traits>;

```
- **EN**: Introduces declarations for `ArmNeon_OverloadedOneResultIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmNeon_OverloadedOneResultIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 68-74
```tablegen
// ArmNeon dialect op that corresponds to an LLVM IR intrinsic with one
// overloaded result and overloaded operands list.
class ArmNeon_OverloadedOperandsWithOneResultIntrOp<string mnemonic,
                                                    list<int> overloadedOperands,
                                                    list<Trait> traits = []>
  : ArmNeon_IntrOp<mnemonic, [0], overloadedOperands, 1, traits>;

```
- **EN**: Introduces declarations for `ArmNeon_OverloadedOperandsWithOneResultIntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmNeon_OverloadedOperandsWithOneResultIntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 75-86
```tablegen
def SMullOp : ArmNeon_OverloadedOneResultIntrOp<"smull", [
       Pure,
       AllTypesMatch<["a", "b"]>,
       TypesMatchWith<
         "res has same vector shape and element bitwidth scaled by 2 as a",
         "a", "res", "::llvm::cast<VectorType>($_self).scaleElementBitwidth(2)">
    ]> {
  let summary = "smull roundscale op";
  let description = [{
    Signed Multiply Long (vector). This instruction multiplies corresponding
    signed integer values in the lower or upper half of the vectors of the two
    source SIMD&FP registers, places the results in a vector, and writes the
```
- **EN**: Introduces declarations for `SMullOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SMullOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 87-92
```tablegen
    vector to the destination SIMD&FP register.

    Source:
    https://developer.arm.com/architectures/instruction-sets/simd-isas/neon/intrinsics
  }];

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 93-103
```tablegen
  // Supports either:
  //   (vector<8xi8>, vector<8xi8>) -> (vector<8xi16>)
  //   (vector<4xi16>, vector<4xi16>) -> (vector<4xi32>)
  //   (vector<2xi32>, vector<2xi32>) -> (vector<2xi64>)
  let arguments = (ins VectorOfLengthAndType<[8, 4, 2], [I8, I16, I32]>:$a,
                       VectorOfLengthAndType<[8, 4, 2], [I8, I16, I32]>:$b);
  let results = (outs VectorOfLengthAndType<[8, 4, 2], [I16, I32, I64]>:$res);
  let assemblyFormat =
    "$a `,` $b attr-dict `:` type($a) `to` type($res)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 104-115
```tablegen
def SdotOp : ArmNeon_OverloadedOperandsWithOneResultIntrOp<"sdot",[1], [
      Pure,
      AllTypesMatch<["b", "c"]>,
      AllTypesMatch<["a", "res"]>,
      TypesMatchWith<"res has the same number of elements as operand b",
                     "b", "res",
                     "VectorType::get({::llvm::cast<VectorType>($_self).getShape()[0] / 4},"
                     "IntegerType::get($_self.getContext(), 32))">]> {
  let summary = "sdot op";
  let description = [{
    Signed integer addition of dot product (vector). This instruction performs
    the following operation on signed integer vectors: res = dot(b, c) + a,
```
- **EN**: Introduces declarations for `SdotOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SdotOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 116-127
```tablegen
    where vector operands are partitioned into groups of four elements.

    Source:
    https://developer.arm.com/architectures/instruction-sets/simd-isas/neon/intrinsics
  }];
  // Supports either:
  //   (vector<2xi32>, vector<8xi8>, vector<8xi8>) -> vector<2xi32>
  //   (vector<4xi32>, vector<16xi8>, vector<16xi8>) -> vector<4xi32>
  let arguments = (ins VectorOfLengthAndType<[4, 2], [I32]>:$a,
                       VectorOfLengthAndType<[16, 8], [I8]>:$b,
                       VectorOfLengthAndType<[16, 8], [I8]>:$c);
  let results = (outs VectorOfLengthAndType<[4, 2], [I32]>:$res);
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 128-139
```tablegen
  let assemblyFormat =
    "$a `,` $b `,` $c attr-dict `:` type($b) `,` type($c) `to` type($res)";
  }

def SmmlaOp : ArmNeon_OverloadedOperandsWithOneResultIntrOp<"smmla",[1], [
                Pure,
                AllTypesMatch<["src1", "src2"]>,
                AllTypesMatch<["acc", "res"]>,
              ]> {
  let summary = "Matrix-matrix multiply and accumulate op";
  let description = [{
    SMMLA: Signed integer matrix multiply-accumulate.
```
- **EN**: Introduces declarations for `SmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 140-147
```tablegen

    Signed 8-bit integer matrix multiply-accumulate. This instruction multiplies
    the 2x8 matrix of signed 8-bit integer values in the first source vector by
    the 8x2 matrix of signed 8-bit integer values in the second source vector.
    The resulting 2x2 32-bit integer matrix product is destructively added to
    the 32-bit integer matrix accumulator in the destination vector. This is
    equivalent to performing an 8-way dot product per destination element.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 148-159
```tablegen
    Source:
    https://developer.arm.com/architectures/instruction-sets/intrinsics/#f:@navigationhierarchiessimdisa=[Neon]&q=smmla
  }];
  // Supports (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  let arguments = (ins
          NeonVectorOfLength<4, I32>:$acc,
          NeonVectorOfLength<16, I8>:$src1,
          NeonVectorOfLength<16, I8>:$src2
  );
  let results = (outs NeonVectorOfLength<4, I32>:$res);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($res)";
```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 160-170
```tablegen
}

def UmmlaOp : ArmNeon_OverloadedOperandsWithOneResultIntrOp<"ummla",[1], [
                Pure,
                AllTypesMatch<["src1", "src2"]>,
                AllTypesMatch<["acc", "res"]>,
              ]> {
  let summary = "Unsinged matrix-matrix multiply and accumulate op";
  let description = [{
    UMMLA: Signed integer matrix multiply-accumulate.

```
- **EN**: Introduces declarations for `UmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 171-178
```tablegen
    Unsigned 8-bit integer matrix multiply-accumulate. This instruction
    multiplies the 2x8 matrix of unsigned 8-bit integer values in the first
    source vector by the 8x2 matrix of unsigned 8-bit integer values in the
    second source vector. The resulting 2x2 32-bit integer matrix product is
    destructively added to the 32-bit integer matrix accumulator in the
    destination vector. This is equivalent to performing an 8-way dot product
    per destination element.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 179-190
```tablegen
    Source:
    https://developer.arm.com/architectures/instruction-sets/intrinsics/#f:@navigationhierarchiessimdisa=[Neon]&q=ummla
  }];
  // Supports (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  let arguments = (ins
          NeonVectorOfLength<4, I32>:$acc,
          NeonVectorOfLength<16, I8>:$src1,
          NeonVectorOfLength<16, I8>:$src2
  );
  let results = (outs NeonVectorOfLength<4, I32>:$res);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($res)";
```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 191-201
```tablegen
}

def UsmmlaOp : ArmNeon_OverloadedOperandsWithOneResultIntrOp<"usmmla",[1], [
                Pure,
                AllTypesMatch<["src1", "src2"]>,
                AllTypesMatch<["acc", "res"]>,
              ]> {
  let summary = "Unsignged and signed matrix-matrix multiply and accumulate op";
  let description = [{
    USMMLA: Signed integer matrix multiply-accumulate.

```
- **EN**: Introduces declarations for `UsmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `UsmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 202-209
```tablegen
    Unsigned and signed 8-bit integer matrix multiply-accumulate. This
    instruction multiplies the 2x8 matrix of unsigned 8-bit integer values in
    the first source vector by the 8x2 matrix of signed 8-bit integer values in
    the second source vector. The resulting 2x2 32-bit integer matrix product is
    destructively added to the 32-bit integer matrix accumulator in the
    destination vector. This is equivalent to performing an 8-way dot product
     per destination element.

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 210-221
```tablegen

    Source:
    https://developer.arm.com/architectures/instruction-sets/intrinsics/#f:@navigationhierarchiessimdisa=[Neon]&q=usmmla
  }];
  // Supports (vector<16xi8>, vector<16xi8>) -> (vector<4xi32>)
  let arguments = (ins
          NeonVectorOfLength<4, I32>:$acc,
          NeonVectorOfLength<16, I8>:$src1,
          NeonVectorOfLength<16, I8>:$src2
  );
  let results = (outs NeonVectorOfLength<4, I32>:$res);
  let assemblyFormat =
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 222-233
```tablegen
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($res)";
}

def BfmmlaOp : ArmNeon_IntrOp<"bfmmla", [], [], 1, [
                 Pure,
                 AllTypesMatch<["src1", "src2"]>,
                 AllTypesMatch<["acc", "res"]>,
               ]> {
  let summary = "BFloat16 matrix multiply-accumulate to single-precision";
  let description = [{
    BFMMLA: BFloat16 matrix multiply-accumulate to single-precision.

```
- **EN**: Introduces declarations for `BfmmlaOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `BfmmlaOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 234-245
```tablegen
    The operation multiplies the 2x4 BFloat16 matrix in the first source vector
    with the 4x2 BFloat16 matrix in the second source vector, then accumulates
    this intermediate result with the 2x2 Float32 matrix in the accumulator
    vector, yielding the final 2x2 Float32 result.

    Source:
    https://developer.arm.com/architectures/instruction-sets/intrinsics/vbfmmlaq_f32
  }];
  // Supports (vector<8xbf16>, vector<8xbf16>) -> (vector<2xf32>)
  let arguments = (ins
    NeonVectorOfLength<4, F32>:$acc,
    NeonVectorOfLength<8, BF16>:$src1,
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 246-252
```tablegen
    NeonVectorOfLength<8, BF16>:$src2
  );
  let results = (outs NeonVectorOfLength<4, F32>:$res);
  let assemblyFormat =
    "$acc `,` $src1 `,` $src2 attr-dict `:` type($src1) `to` type($res)";
}

```
- **EN**: Declares APIs or declarative rules around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `type` 相关的 API 或声明式规则；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性。

### Lines 253-258
```tablegen
class ArmNeon_2dOp<string mnemonic, list<Trait> traits = []>
    : Op</*dialect=*/ArmNeon_Dialect,
         /*opName=*/"2d." # mnemonic,
         /*traits=*/traits>;

def Sdot2dOp : ArmNeon_2dOp<"sdot", [
```
- **EN**: Introduces declarations for `ArmNeon_2dOp`, `Sdot2dOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmNeon_2dOp`, `Sdot2dOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 259-270
```tablegen
      Pure,
      AllTypesMatch<["b", "c"]>,
      AllTypesMatch<["a", "res"]>,
      PredOpTrait<
        "operand `a` should be 1-dimensional",
        CPred<"::llvm::cast<VectorType>(getA().getType()).getShape().size() == 1">
      >,
      PredOpTrait<
        "operand `b` should be 2-dimensional",
        CPred<"::llvm::cast<VectorType>(getB().getType()).getShape().size() == 2">
      >,
      PredOpTrait<
```
- **EN**: Implements logic around `getA`, `getB`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getA`, `getB` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 271-282
```tablegen
        "operand `b` should have 4 columns",
        CPred<"::llvm::cast<VectorType>(getB().getType()).getShape()[1] == 4">
      >,
      PredOpTrait<
        "operand `b` should have as many rows as the size of operand `a`",
        CPred<"::llvm::cast<VectorType>(getB().getType()).getShape()[0] == ::llvm::cast<VectorType>(getA().getType()).getShape()[0]">
      >,
      ]
  > {
  let summary = "sdot op";
  let description = [{
    The two input vectors `b` and `c` have a 2D shape, consisting of either 2
```
- **EN**: Implements logic around `getB`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getB` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 283-290
```tablegen
    or 4 rows, each row having length 4. This operation computes the pair-wise
    dot-products of the rows of `b` and `c` and accumulates them with the
    corresponding entry of `a`:

    ```
    res[i] := a[i] + dot_product(b[i, ...], c[i, ...])
    ```

```
- **EN**: Implements logic around `dot_product`.
- **CN**: 围绕 `dot_product` 实现具体逻辑。

### Lines 291-302
```tablegen
  }];
  // Supports either:
  //   (vector<2xi32>, vector<2x4xi8>, vector<2x4xi8>) -> vector<2xi32>
  //   (vector<4xi32>, vector<4x4xi8>, vector<4x4xi8>) -> vector<4xi32>
  // TODO: how do we express 2D shape requirements here?
  let arguments = (ins VectorOfLengthAndType<[4, 2], [I32]>:$a,
                       VectorOfLengthAndType<[16, 8], [I8]>:$b,
                       VectorOfLengthAndType<[16, 8], [I8]>:$c);
  let results = (outs VectorOfLengthAndType<[4, 2], [I32]>:$res);
  let assemblyFormat =
    "$a `,` $b `,` $c attr-dict `:` type($b) `,` type($c) `to` type($res)";
  let extraClassDeclaration = [{
```
- **EN**: Implements logic around `type`; this block handles textual assembly parsing or printing concerns; works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `type` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题，并处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 303-307
```tablegen
    static constexpr int kReductionSize = 4;
  }];
}

#endif // ARMNEON_OPS
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
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

- **TableGen includes / TableGen 包含**: `mlir/Dialect/LLVMIR/LLVMOpBase.td`, `mlir/Interfaces/SideEffectInterfaces.td`, `mlir/IR/OpBase.td`
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (1), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
