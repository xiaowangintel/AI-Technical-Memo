# AArch64SchedPredNeoverse.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64SchedPredNeoverse.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This file defines scheduling predicate definitions that are used by the AArch64 Neoverse processors. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的调度模型。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Documented TableGen section
```tablegen
//===- AArch64SchedPredNeoverse.td - AArch64 Sched Preds -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines scheduling predicate definitions that are used by the
// AArch64 Neoverse processors.
//
//===----------------------------------------------------------------------===//

// Auxiliary predicates.
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 15-31: TableGen definition NeoverseNoLSL
```tablegen

// Check for LSL shift == 0
def NeoverseNoLSL : MCSchedPredicate<
                      CheckAll<[CheckShiftLSL,
                                CheckShiftBy0]>>;

// Identify LDR/STR H/Q-form scaled (and potentially extended) FP instructions
def NeoverseHQForm : MCSchedPredicate<
                       CheckAll<[
                         CheckAny<[CheckHForm, CheckQForm]>,
                         CheckImmOperand<4, 1>]>>;

// Identify LDR/STR Q-form scaled (and potentially extended) FP instructions
def NeoverseQForm : MCSchedPredicate<
                       CheckAll<[
                         CheckAny<[CheckQForm]>,
                         CheckImmOperand<4, 1>]>>;
```
**EN:** This definition materializes NeoverseNoLSL as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 NeoverseNoLSL 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 32-40: TableGen definition NeoversePdIsPgFn
```tablegen

// Check if <Pd> == <Pg>
def NeoversePdIsPgFn : TIIPredicate<
                         "isNeoversePdSameAsPg",
                         MCOpcodeSwitchStatement<
                           [MCOpcodeSwitchCase<[BRKA_PPmP, BRKB_PPmP],
                             MCReturnStatement<CheckSameRegOperand<1, 2>>>],
                           MCReturnStatement<CheckSameRegOperand<0, 1>>>>;
def NeoversePdIsPg : MCSchedPredicate<NeoversePdIsPgFn>;
```
**EN:** This definition materializes NeoversePdIsPgFn as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 NeoversePdIsPgFn 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 41-53: TableGen definition NeoverseCheapIncDec
```tablegen

// Check if SVE INC/DEC (scalar), ALL, {1, 2, 4}
def NeoverseCheapIncDec : MCSchedPredicate<
                            CheckAll<[CheckOpcode<[
                                        INCB_XPiI, INCH_XPiI,
                                        INCW_XPiI, INCD_XPiI,
                                        DECB_XPiI, DECH_XPiI,
                                        DECW_XPiI, DECD_XPiI]>,
                                      CheckImmOperand<2, 31>,
                                      CheckAny<[
                                        CheckImmOperand<3, 1>,
                                        CheckImmOperand<3, 2>,
                                        CheckImmOperand<3, 4>]>]>>;
```
**EN:** This definition materializes NeoverseCheapIncDec as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 NeoverseCheapIncDec 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 54-62: TableGen definition NeoverseMULIdiomPred
```tablegen

// Identify "[SU]?(MADD|MSUB)L?" as the alias for "[SU]?(MUL|MNEG)L?".
def NeoverseMULIdiomPred : MCSchedPredicate< // <op> Rd, Rs, Rv, ZR
                             CheckAll<[CheckOpcode<
                                         [MADDWrrr, MADDXrrr,
                                          MSUBWrrr, MSUBXrrr,
                                          SMADDLrrr, UMADDLrrr,
                                          SMSUBLrrr, UMSUBLrrr]>,
                                       CheckIsReg3Zero]>>;
```
**EN:** This definition materializes NeoverseMULIdiomPred as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 NeoverseMULIdiomPred 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 63-80: TableGen definition NeoverseZeroMove
```tablegen

def NeoverseZeroMove : MCSchedPredicate<
                         CheckAny<[
                           // MOV Wd, #0
                           // MOV Xd, #0
                           CheckAll<[CheckOpcode<[MOVZWi, MOVZXi]>,
                                     CheckIsImmOperand<1>,
                                     CheckImmOperand<1, 0>,
                                     CheckImmOperand<2, 0>]>,
                           // MOV Wd, WZR
                           // MOV Xd, XZR
                           // MOV Wd, Wn
                           // MOV Xd, Xn
                           CheckAll<[CheckOpcode<[ORRWrs, ORRXrs]>,
                                     CheckAll<[CheckIsReg1Zero,
                                               CheckImmOperand<3, 0>]>]>,
                           // FMOV Hd, WZR
                           // FMOV Hd, XZR
```
**EN:** This definition materializes NeoverseZeroMove as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 NeoverseZeroMove 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 81-96: Documented TableGen section
```tablegen
                           // FMOV Sd, WZR
                           // FMOV Dd, XZR
                           CheckAll<[CheckOpcode<[FMOVWHr, FMOVXHr,
                                                  FMOVWSr, FMOVXDr]>,
                                     CheckIsReg1Zero]>,
                           // MOVI Dd, #0
                           // MOVI Vd.2D, #0
                           CheckAll<[CheckOpcode<[MOVID, MOVIv2d_ns]>,
                                     CheckImmOperand<1, 0>]>,
                           // MOV Zd, Zn
                           CheckAll<[CheckOpcode<[ORR_ZZZ]>,
                                     CheckSameRegOperand<1, 2>]>,
                           // MOV Vd, Vn
                           CheckAll<[CheckOpcode<[ORRv16i8, ORRv8i8]>,
                                     CheckSameRegOperand<1, 2>]>,
                         ]>>;
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 97-108: TableGen definition NeoverseAllActivePredicate
```tablegen

def NeoverseAllActivePredicate : MCSchedPredicate<
                                   CheckAny<[
                                     // PTRUE Pd, ALL
                                     // PTRUES Pd, ALL
                                     CheckAll<[
                                       CheckOpcode<[
                                         PTRUE_B, PTRUE_H, PTRUE_S, PTRUE_D,
                                         PTRUES_B, PTRUES_H, PTRUES_S, PTRUES_D]>,
                                       CheckIsImmOperand<1>,
                                       CheckImmOperand<1, 31>]>,
                                   ]>>;
```
**EN:** This definition materializes NeoverseAllActivePredicate as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 NeoverseAllActivePredicate 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 109-109: Whitespace and separators
```tablegen

```
**EN:** This TableGen block continues the file's scheduling models work and contributes declarative data for LLVM's code generators.  
**CN:** 该 TableGen 代码块延续本文件的调度模型工作，并为 LLVM 代码生成器提供声明式数据。
### Lines 110-110: Whitespace and separators
```tablegen

```
**EN:** This TableGen block continues the file's scheduling models work and contributes declarative data for LLVM's code generators.  
**CN:** 该 TableGen 代码块延续本文件的调度模型工作，并为 LLVM 代码生成器提供声明式数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Per-core scheduling itineraries and latency models **CN:** 面向具体内核的调度行程与时延模型
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for scheduling models. **CN:** 与周边负责调度模型的 AArch64 后端组件紧密协作。
