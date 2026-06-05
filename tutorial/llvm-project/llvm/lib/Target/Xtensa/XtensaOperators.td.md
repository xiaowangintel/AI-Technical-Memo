# XtensaOperators.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaOperators.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines SelectionDAG/GlobalISel operators and pattern helper records in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义 SelectionDAG/GlobalISel 操作符及模式辅助记录。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- XtensaOperators.td - Xtensa-specific operators ---------*- tblgen-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-15
```tablegen

//===----------------------------------------------------------------------===//
// Type profiles
//===----------------------------------------------------------------------===//
def SDT_XtensaCallSeqStart        : SDCallSeqStart<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
def SDT_XtensaCallSeqEnd          : SDCallSeqEnd<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
def SDT_XtensaCall                : SDTypeProfile<0, -1, [SDTCisPtrTy<0>]>;

```
- **EN**: Declares TableGen records such as `SDT_XtensaCallSeqStart`, `SDT_XtensaCallSeqEnd`, `SDT_XtensaCall` for the backend description.
- **CN**: 为后端描述声明了 `SDT_XtensaCallSeqStart`, `SDT_XtensaCallSeqEnd`, `SDT_XtensaCall` 等 TableGen 记录。

### Lines 16-22
```tablegen
def SDT_XtensaWrapPtr             : SDTypeProfile<1, 1,
                                                 [SDTCisSameAs<0, 1>,
                                                  SDTCisPtrTy<0>]>;

def SDT_XtensaBrJT                : SDTypeProfile<0, 2,
                                                 [SDTCisPtrTy<0>, SDTCisVT<1, i32>]>;

```
- **EN**: Declares TableGen records such as `SDT_XtensaWrapPtr`, `SDT_XtensaBrJT` for the backend description.
- **CN**: 为后端描述声明了 `SDT_XtensaWrapPtr`, `SDT_XtensaBrJT` 等 TableGen 记录。

### Lines 23-27
```tablegen
def SDT_XtensaSelectCC            : SDTypeProfile<1, 5,
                                                 [SDTCisSameAs<0, 3>, SDTCisSameAs<1, 2>,
                                                  SDTCisSameAs<3, 4>,
                                                  SDTCisVT<5, i32>]>;

```
- **EN**: Declares TableGen records such as `SDT_XtensaSelectCC` for the backend description.
- **CN**: 为后端描述声明了 `SDT_XtensaSelectCC` 等 TableGen 记录。

### Lines 28-33
```tablegen
def SDT_XtensaCmp                 : SDTypeProfile<1, 2, [SDTCisVT<0, v1i1>, SDTCisVT<1, f32>, SDTCisVT<2, f32>]>;
def SDT_XtensaMADD                : SDTypeProfile<1, 3, [SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisSameAs<0, 3>, SDTCisVT<0, f32>]>;
def SDT_XtensaMOVS                : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>, SDTCisVT<0, f32>]>;
def SDT_XtensaSelectCCFP          : SDTypeProfile<1, 6, [SDTCisSameAs<0, 3>, SDTCisSameAs<1, 2>, SDTCisSameAs<3, 4>,
                                                         SDTCisVT<5, i32>, SDTCisVT<6, i32>]>;

```
- **EN**: Declares TableGen records such as `SDT_XtensaCmp`, `SDT_XtensaMADD`, `SDT_XtensaMOVS`, `SDT_XtensaSelectCCFP` for the backend description.
- **CN**: 为后端描述声明了 `SDT_XtensaCmp`, `SDT_XtensaMADD`, `SDT_XtensaMOVS`, `SDT_XtensaSelectCCFP` 等 TableGen 记录。

### Lines 34-39
```tablegen
def SDT_XtensaSRC                 : SDTypeProfile<1, 3, [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
                                                         SDTCisVT<2, i32>, SDTCisVT<3, i32>]>;

def SDT_XtensaEXTUI               : SDTypeProfile<1, 3, [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
                                                         SDTCisVT<2, i32>, SDTCisVT<3, i32>]>;

```
- **EN**: Declares TableGen records such as `SDT_XtensaSRC`, `SDT_XtensaEXTUI` for the backend description.
- **CN**: 为后端描述声明了 `SDT_XtensaSRC`, `SDT_XtensaEXTUI` 等 TableGen 记录。

### Lines 40-44
```tablegen
def SDT_XtensaMOVSP               : SDTypeProfile<0, 1, [SDTCisVT<0, i32>]>;

def SDT_XtensaRUR                : SDTypeProfile<1, 1, [SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;

//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `SDT_XtensaMOVSP`, `SDT_XtensaRUR` for the backend description.
- **CN**: 为后端描述声明了 `SDT_XtensaMOVSP`, `SDT_XtensaRUR` 等 TableGen 记录。

### Lines 45-49
```tablegen
// Node definitions
//===----------------------------------------------------------------------===//
def Xtensa_call: SDNode<"XtensaISD::CALL", SDT_XtensaCall,
                       [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;

```
- **EN**: Declares TableGen records such as `Xtensa_call` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_call` 等 TableGen 记录。

### Lines 50-55
```tablegen
def Xtensa_callw8: SDNode<"XtensaISD::CALLW8", SDT_XtensaCall,
                        [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;

def Xtensa_ret: SDNode<"XtensaISD::RET", SDTNone,
                       [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;

```
- **EN**: Declares TableGen records such as `Xtensa_callw8`, `Xtensa_ret` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_callw8`, `Xtensa_ret` 等 TableGen 记录。

### Lines 56-60
```tablegen
def Xtensa_retw: SDNode<"XtensaISD::RETW", SDTNone,
                       [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;

def Xtensa_pcrel_wrapper: SDNode<"XtensaISD::PCREL_WRAPPER", SDT_XtensaWrapPtr, []>;

```
- **EN**: Declares TableGen records such as `Xtensa_retw`, `Xtensa_pcrel_wrapper` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_retw`, `Xtensa_pcrel_wrapper` 等 TableGen 记录。

### Lines 61-67
```tablegen
def Xtensa_callseq_start: SDNode<"ISD::CALLSEQ_START", SDT_XtensaCallSeqStart,
                                [SDNPHasChain, SDNPSideEffect, SDNPOutGlue]>;

def Xtensa_callseq_end  : SDNode<"ISD::CALLSEQ_END",   SDT_XtensaCallSeqEnd,
                                [SDNPHasChain, SDNPSideEffect, SDNPOptInGlue,
                                 SDNPOutGlue]>;

```
- **EN**: Declares TableGen records such as `Xtensa_callseq_start`, `Xtensa_callseq_end` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_callseq_start`, `Xtensa_callseq_end` 等 TableGen 记录。

### Lines 68-72
```tablegen
def Xtensa_brjt: SDNode<"XtensaISD::BR_JT", SDT_XtensaBrJT, [SDNPHasChain]>;

def Xtensa_select_cc: SDNode<"XtensaISD::SELECT_CC", SDT_XtensaSelectCC,
                            [SDNPInGlue]>;

```
- **EN**: Declares TableGen records such as `Xtensa_brjt`, `Xtensa_select_cc` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_brjt`, `Xtensa_select_cc` 等 TableGen 记录。

### Lines 73-78
```tablegen
def Xtensa_srcl: SDNode<"XtensaISD::SRCL", SDT_XtensaSRC>;

def Xtensa_srcr: SDNode<"XtensaISD::SRCR", SDT_XtensaSRC>;

def Xtensa_extui: SDNode<"XtensaISD::EXTUI", SDT_XtensaEXTUI>;

```
- **EN**: Declares TableGen records such as `Xtensa_srcl`, `Xtensa_srcr`, `Xtensa_extui` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_srcl`, `Xtensa_srcr`, `Xtensa_extui` 等 TableGen 记录。

### Lines 79-84
```tablegen
def Xtensa_movsp: SDNode<"XtensaISD::MOVSP", SDT_XtensaMOVSP,
                        [SDNPHasChain, SDNPSideEffect, SDNPInGlue]>;

def Xtensa_rur: SDNode<"XtensaISD::RUR", SDT_XtensaRUR,
                      [SDNPInGlue]>;

```
- **EN**: Declares TableGen records such as `Xtensa_movsp`, `Xtensa_rur` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_movsp`, `Xtensa_rur` 等 TableGen 记录。

### Lines 85-92
```tablegen
def Xtensa_cmpoeq     : SDNode<"XtensaISD::CMPOEQ", SDT_XtensaCmp, [SDNPOutGlue]>;
def Xtensa_cmpolt     : SDNode<"XtensaISD::CMPOLT", SDT_XtensaCmp, [SDNPOutGlue]>;
def Xtensa_cmpole     : SDNode<"XtensaISD::CMPOLE", SDT_XtensaCmp, [SDNPOutGlue]>;
def Xtensa_cmpueq     : SDNode<"XtensaISD::CMPUEQ", SDT_XtensaCmp, [SDNPOutGlue]>;
def Xtensa_cmpult     : SDNode<"XtensaISD::CMPULT", SDT_XtensaCmp, [SDNPOutGlue]>;
def Xtensa_cmpule     : SDNode<"XtensaISD::CMPULE", SDT_XtensaCmp, [SDNPOutGlue]>;
def Xtensa_cmpuo      : SDNode<"XtensaISD::CMPUO", SDT_XtensaCmp, [SDNPOutGlue]>;

```
- **EN**: Declares TableGen records such as `Xtensa_cmpoeq`, `Xtensa_cmpolt`, `Xtensa_cmpole`, `Xtensa_cmpueq`, ... for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_cmpoeq`, `Xtensa_cmpolt`, `Xtensa_cmpole`, `Xtensa_cmpueq`, ... 等 TableGen 记录。

### Lines 93-98
```tablegen
def Xtensa_madd: SDNode<"XtensaISD::MADD", SDT_XtensaMADD, [SDNPInGlue]>;
def Xtensa_msub: SDNode<"XtensaISD::MSUB", SDT_XtensaMADD, [SDNPInGlue]>;
def Xtensa_movs: SDNode<"XtensaISD::MOVS", SDT_XtensaMOVS, [SDNPInGlue]>;

def Xtensa_select_cc_fp: SDNode<"XtensaISD::SELECT_CC_FP", SDT_XtensaSelectCCFP,
                               [SDNPInGlue]>;
```
- **EN**: Declares TableGen records such as `Xtensa_madd`, `Xtensa_msub`, `Xtensa_movs`, `Xtensa_select_cc_fp` for the backend description.
- **CN**: 为后端描述声明了 `Xtensa_madd`, `Xtensa_msub`, `Xtensa_movs`, `Xtensa_select_cc_fp` 等 TableGen 记录。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
