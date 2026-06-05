# ArmSMEIntrinsicOps.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/ArmSME/IR/ArmSMEIntrinsicOps.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains definitions of the intrinsic Ops for the ArmSME dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/ArmSME/IR`，围绕 ArmSME 方言公开 `ArmSMEIntrinsicOps` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- ArmSMEIntrinsicOps.td ------------------------------*- tablegen -*-===//
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
// This file contains definitions of the intrinsic Ops for the ArmSME dialect.
//
//===----------------------------------------------------------------------===//

#ifndef ARMSME_INTRINSIC_OPS
#define ARMSME_INTRINSIC_OPS

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-21
```tablegen
include "ArmSME.td"

//===----------------------------------------------------------------------===//
// ArmSME Intrinsic op definitions
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 22-27
```tablegen
def MOPPredicate : ScalableVectorOfRankAndLengthAndType<[1], [16, 8, 4, 2], [I1]>
{
  let summary = "a vector type that is a supported predicate for the SME MOP instructions";
  let description = [{
    Possible vector types:

```
- **EN**: Introduces declarations for `MOPPredicate`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MOPPredicate` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 28-34
```tablegen
    * `vector<[16]xi1>`
    * `vector<[8]xi1>`
    * `vector<[4]xi1>`
    * `vector<[2]xi1>`
  }];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 35-42
```tablegen
// FIXME: This allows types that are not SVE vectors, e.g. vector<[16]xf32>.
def MOPVector : ScalableVectorOfRankAndLengthAndType<[1], [16, 8, 4, 2],
                                              [I8, I16, BF16, F16, F32, F64]>
{
  let summary = "a vector type that is a supported input for the SME MOP instructions";
  let description = [{
    Possible vector types:

```
- **EN**: Introduces declarations for `MOPVector`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MOPVector` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 43-49
```tablegen
    Integer elements:

    * `vector<[16]xi8>`
    * `vector<[8]xi16>`

    Floating point elements:

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 50-56
```tablegen
    * `vector<[8]xf16>`
    * `vector<[8]xbf16>`
    * `vector<[4]xf32>`
    * `vector<[2]xf64>`
  }];
}

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 57-68
```tablegen
class ArmSME_IntrOp<string mnemonic,
                    list<int> immArgPositions = [],
                    list<string> immArgAttrNames = [],
                    list<int> overloadedOperands = [],
                    list<Trait> traits = [], int numResults = 0,
                    list<int> overloadedResults = []>
    : LLVM_IntrOpBase<
          /*Dialect dialect=*/ArmSME_Dialect,
          /*string opName=*/"intr." # mnemonic,
          /*string enumName=*/"aarch64_sme_" # !subst(".", "_", mnemonic),
          /*list<int> overloadedResults=*/overloadedResults,
          /*list<int> overloadedOperands=*/overloadedOperands,
```
- **EN**: Introduces declarations for `ArmSME_IntrOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSME_IntrOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 69-78
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

### Lines 79-85
```tablegen
// Zero
def LLVM_aarch64_sme_zero
   : ArmSME_IntrOp<"zero",
                   /*immArgPositions=*/[0],
                   /*immArgAttrNames=*/["tile_mask"]>,
     Arguments<(ins Arg<I32Attr, "Tile mask">:$tile_mask)>;

```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_zero`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_zero` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 86-97
```tablegen
// MOP's
class ArmSME_IntrMopOverloadedOp<string mnemonic>
    : ArmSME_IntrOp<mnemonic,
                    /*immArgPositions=*/[0],
                    /*immArgAttrNames=*/["tile_id"],
                    /*overloadedOperands=*/[4]>,
      Arguments<(ins Arg<I32Attr, "Virtual tile ID">:$tile_id,
                 Arg<MOPPredicate, "LHS predicate">:$lhs_predicate,
                 Arg<MOPPredicate, "RHS predicate">:$rhs_predicate,
                 Arg<MOPVector, "LHS vector operand">:$lhs_vector,
                 Arg<MOPVector, "RHS vector operand">:$rhs_vector)>;

```
- **EN**: Introduces declarations for `ArmSME_IntrMopOverloadedOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSME_IntrMopOverloadedOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 98-103
```tablegen
def LLVM_aarch64_sme_mopa : ArmSME_IntrMopOverloadedOp<"mopa">;
def LLVM_aarch64_sme_mops : ArmSME_IntrMopOverloadedOp<"mops">;
def LLVM_aarch64_sme_mopa_wide : ArmSME_IntrMopOverloadedOp<"mopa.wide">;
def LLVM_aarch64_sme_mops_wide : ArmSME_IntrMopOverloadedOp<"mops.wide">;
def LLVM_aarch64_sme_smopa_wide : ArmSME_IntrMopOverloadedOp<"smopa.wide">;
def LLVM_aarch64_sme_smops_wide : ArmSME_IntrMopOverloadedOp<"smops.wide">;
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_mopa`, `LLVM_aarch64_sme_mops`, `LLVM_aarch64_sme_mopa_wide`, `LLVM_aarch64_sme_mops_wide`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_mopa`, `LLVM_aarch64_sme_mops`, `LLVM_aarch64_sme_mopa_wide`, `LLVM_aarch64_sme_mops_wide`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 104-109
```tablegen
def LLVM_aarch64_sme_umopa_wide : ArmSME_IntrMopOverloadedOp<"umopa.wide">;
def LLVM_aarch64_sme_umops_wide : ArmSME_IntrMopOverloadedOp<"umops.wide">;
def LLVM_aarch64_sme_sumopa_wide : ArmSME_IntrMopOverloadedOp<"sumopa.wide">;
def LLVM_aarch64_sme_sumops_wide : ArmSME_IntrMopOverloadedOp<"sumops.wide">;
def LLVM_aarch64_sme_usmopa_wide : ArmSME_IntrMopOverloadedOp<"usmopa.wide">;
def LLVM_aarch64_sme_usmops_wide : ArmSME_IntrMopOverloadedOp<"usmops.wide">;
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_umopa_wide`, `LLVM_aarch64_sme_umops_wide`, `LLVM_aarch64_sme_sumopa_wide`, `LLVM_aarch64_sme_sumops_wide`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_umopa_wide`, `LLVM_aarch64_sme_umops_wide`, `LLVM_aarch64_sme_sumopa_wide`, `LLVM_aarch64_sme_sumops_wide`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 110-115
```tablegen
def LLVM_aarch64_sme_smopa_za32 : ArmSME_IntrMopOverloadedOp<"smopa.za32">;
def LLVM_aarch64_sme_umopa_za32 : ArmSME_IntrMopOverloadedOp<"umopa.za32">;
def LLVM_aarch64_sme_smops_za32 : ArmSME_IntrMopOverloadedOp<"smops.za32">;
def LLVM_aarch64_sme_umops_za32 : ArmSME_IntrMopOverloadedOp<"umops.za32">;

class ArmSME_IntrLoadStoreOp<string mnemonic>
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_smopa_za32`, `LLVM_aarch64_sme_umopa_za32`, `LLVM_aarch64_sme_smops_za32`, `LLVM_aarch64_sme_umops_za32`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_smopa_za32`, `LLVM_aarch64_sme_umopa_za32`, `LLVM_aarch64_sme_smops_za32`, `LLVM_aarch64_sme_umops_za32`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 116-122
```tablegen
    : ArmSME_IntrOp<mnemonic,
                    /*immArgPositions=*/[2],
                    /*immArgAttrNames=*/["tile_id"],
                    /*overloadedOperands=*/[1]>;

// Loads (from memory to ZA tile slice)
class ArmSME_IntrLoadOp<string mnemonic>
```
- **EN**: Introduces declarations for `ArmSME_IntrLoadOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `ArmSME_IntrLoadOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 123-128
```tablegen
    : ArmSME_IntrLoadStoreOp<mnemonic>,
      Arguments<(ins Arg<SVEPredicate, "Vector predicate">:$predicate,
                 Arg<LLVM_AnyPointer, "Load address">:$load_address,
                 Arg<I32Attr, "Virtual tile ID">:$tile_id,
                 Arg<I32, "Tile slice">:$tile_slice_index)>;

```
- **EN**: Declares APIs or declarative rules around `Arguments`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `Arguments` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 129-134
```tablegen
def LLVM_aarch64_sme_ld1b_horiz : ArmSME_IntrLoadOp<"ld1b.horiz">;
def LLVM_aarch64_sme_ld1h_horiz : ArmSME_IntrLoadOp<"ld1h.horiz">;
def LLVM_aarch64_sme_ld1w_horiz : ArmSME_IntrLoadOp<"ld1w.horiz">;
def LLVM_aarch64_sme_ld1d_horiz : ArmSME_IntrLoadOp<"ld1d.horiz">;
def LLVM_aarch64_sme_ld1q_horiz : ArmSME_IntrLoadOp<"ld1q.horiz">;
def LLVM_aarch64_sme_ld1b_vert : ArmSME_IntrLoadOp<"ld1b.vert">;
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_ld1b_horiz`, `LLVM_aarch64_sme_ld1h_horiz`, `LLVM_aarch64_sme_ld1w_horiz`, `LLVM_aarch64_sme_ld1d_horiz`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_ld1b_horiz`, `LLVM_aarch64_sme_ld1h_horiz`, `LLVM_aarch64_sme_ld1w_horiz`, `LLVM_aarch64_sme_ld1d_horiz`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 135-141
```tablegen
def LLVM_aarch64_sme_ld1h_vert : ArmSME_IntrLoadOp<"ld1h.vert">;
def LLVM_aarch64_sme_ld1w_vert : ArmSME_IntrLoadOp<"ld1w.vert">;
def LLVM_aarch64_sme_ld1d_vert : ArmSME_IntrLoadOp<"ld1d.vert">;
def LLVM_aarch64_sme_ld1q_vert : ArmSME_IntrLoadOp<"ld1q.vert">;

// Stores (ZA tile slice to memory)
class ArmSME_IntrStoreOp<string mnemonic>
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_ld1h_vert`, `LLVM_aarch64_sme_ld1w_vert`, `LLVM_aarch64_sme_ld1d_vert`, `LLVM_aarch64_sme_ld1q_vert`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_ld1h_vert`, `LLVM_aarch64_sme_ld1w_vert`, `LLVM_aarch64_sme_ld1d_vert`, `LLVM_aarch64_sme_ld1q_vert`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 142-147
```tablegen
    : ArmSME_IntrLoadStoreOp<mnemonic>,
      Arguments<(ins Arg<SVEPredicate, "Vector predicate">:$predicate,
                 Arg<LLVM_AnyPointer, "Store address", [MemWrite]>:$store_address,
                 Arg<I32Attr, "Virtual tile ID">:$tile_id,
                 Arg<I32, "Tile slice">:$tile_slice_index)>;

```
- **EN**: Declares APIs or declarative rules around `Arguments`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `Arguments` 相关的 API 或声明式规则；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 148-153
```tablegen
def LLVM_aarch64_sme_st1b_horiz : ArmSME_IntrStoreOp<"st1b.horiz">;
def LLVM_aarch64_sme_st1h_horiz : ArmSME_IntrStoreOp<"st1h.horiz">;
def LLVM_aarch64_sme_st1w_horiz : ArmSME_IntrStoreOp<"st1w.horiz">;
def LLVM_aarch64_sme_st1d_horiz : ArmSME_IntrStoreOp<"st1d.horiz">;
def LLVM_aarch64_sme_st1q_horiz : ArmSME_IntrStoreOp<"st1q.horiz">;
def LLVM_aarch64_sme_st1b_vert : ArmSME_IntrStoreOp<"st1b.vert">;
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_st1b_horiz`, `LLVM_aarch64_sme_st1h_horiz`, `LLVM_aarch64_sme_st1w_horiz`, `LLVM_aarch64_sme_st1d_horiz`, and 2 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_st1b_horiz`, `LLVM_aarch64_sme_st1h_horiz`, `LLVM_aarch64_sme_st1w_horiz`, `LLVM_aarch64_sme_st1d_horiz`, and 2 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 154-159
```tablegen
def LLVM_aarch64_sme_st1h_vert : ArmSME_IntrStoreOp<"st1h.vert">;
def LLVM_aarch64_sme_st1w_vert : ArmSME_IntrStoreOp<"st1w.vert">;
def LLVM_aarch64_sme_st1d_vert : ArmSME_IntrStoreOp<"st1d.vert">;
def LLVM_aarch64_sme_st1q_vert : ArmSME_IntrStoreOp<"st1q.vert">;

def LLVM_aarch64_sme_str
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_st1h_vert`, `LLVM_aarch64_sme_st1w_vert`, `LLVM_aarch64_sme_st1d_vert`, `LLVM_aarch64_sme_st1q_vert`, and 1 more symbols, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_st1h_vert`, `LLVM_aarch64_sme_st1w_vert`, `LLVM_aarch64_sme_st1d_vert`, `LLVM_aarch64_sme_st1q_vert`, and 1 more symbols 等声明，建立后续使用的公共符号或声明式记录。

### Lines 160-167
```tablegen
    : ArmSME_IntrOp<"str",
                    /*immArgPositions=*/[],
                    /*immArgAttrNames=*/[],
                    /*overloadedOperands=*/[1]>,
      Arguments<(ins Arg<I32, "Index">:$index,
                 Arg<LLVM_AnyPointer, "Store address", [MemWrite]>:$store_address,
                 Arg<I32, "Offset">:$offset)>;

```
- **EN**: Declares APIs or declarative rules around `Arguments`.
- **CN**: 声明与 `Arguments` 相关的 API 或声明式规则。

### Lines 168-179
```tablegen
// Vector to tile slice
class LLVM_aarch64_sme_write<string direction>
    : ArmSME_IntrOp<"write." # direction,
                    /*immArgPositions=*/[0],
                    /*immArgAttrNames=*/["tile_id"],
                    /*overloadedOperands=*/[3],
                    [AllShapesMatch<["predicate", "vector"]>]>,
      Arguments<(ins Arg<I32Attr, "Virtual tile ID">:$tile_id,
                     Arg<I32, "Tile slice">:$tile_slice_index,
                     Arg<SVEPredicate, "Vector predicate">:$predicate,
                     Arg<SVEVector, "Vector operand">:$vector)>;

```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_write`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_write` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 180-191
```tablegen
// Tile slice to vector
class LLVM_aarch64_sme_read<string direction>
    : ArmSME_IntrOp<"read." # direction,
                    /*immArgPositions=*/[2],
                    /*immArgAttrNames=*/["tile_id"],
                    /*overloadedOperands=*/[],
                    [AllShapesMatch<["vector", "predicate", "res"]>,
                     AllElementTypesMatch<["vector", "res"]>],
                    /*numResults=*/1, /*overloadedResults=*/[0]>,
      Arguments<(ins Arg<SVEVector, "Vector operand">:$vector,
                     Arg<SVEPredicate, "Vector predicate">:$predicate,
                     Arg<I32Attr, "Virtual tile ID">:$tile_id,
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_read`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_read` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 192-197
```tablegen
                     Arg<I32, "Tile slice">:$tile_slice_index)>;

def LLVM_aarch64_sme_write_horiz : LLVM_aarch64_sme_write<"horiz">;
def LLVM_aarch64_sme_write_vert : LLVM_aarch64_sme_write<"vert">;

def LLVM_aarch64_sme_read_horiz : LLVM_aarch64_sme_read<"horiz">;
```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_write_horiz`, `LLVM_aarch64_sme_write_vert`, `LLVM_aarch64_sme_read_horiz`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_write_horiz`, `LLVM_aarch64_sme_write_vert`, `LLVM_aarch64_sme_read_horiz` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 198-207
```tablegen
def LLVM_aarch64_sme_read_vert : LLVM_aarch64_sme_read<"vert">;

class ArmSME_IntrCountOp<string mnemonic>
    : ArmSME_IntrOp<mnemonic,
                    /*immArgPositions=*/[],
                    /*immArgAttrNames=*/[],
                    /*overloadedOperands=*/[],
                    /*traits*/[PredOpTrait<"`res` is i64", TypeIsPred<"res", I64>>],
                    /*numResults=*/1, /*overloadedResults=*/[]>;

```
- **EN**: Introduces declarations for `LLVM_aarch64_sme_read_vert`, `ArmSME_IntrCountOp`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `LLVM_aarch64_sme_read_vert`, `ArmSME_IntrCountOp` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 208-210
```tablegen
def LLVM_aarch64_sme_cntsd : ArmSME_IntrCountOp<"cntsd">;

#endif // ARMSME_INTRINSIC_OPS
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
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `ArmSME.td`
