# AArch64SchedPredExynos.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64SchedPredExynos.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This file defines scheduling predicate definitions that are used by the AArch64 Exynos processors. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的调度模型。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Documented TableGen section
```tablegen
//===- AArch64SchedPredExynos.td - AArch64 Sched Preds -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines scheduling predicate definitions that are used by the
// AArch64 Exynos processors.
//
//===----------------------------------------------------------------------===//

// Auxiliary predicates.

// Check the shift in arithmetic and logic instructions.
def ExynosCheckShift : CheckAny<[CheckShiftBy0,
                                 CheckAll<
                                   [CheckShiftLSL,
                                    CheckAny<
                                      [CheckShiftBy1,
                                       CheckShiftBy2,
                                       CheckShiftBy3]>]>]>;

// Exynos predicates.
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 26-53: TableGen definition ExynosBranchLinkLRPred
```tablegen

// Identify BLR specifying the LR register as the indirect target register.
def ExynosBranchLinkLRPred : MCSchedPredicate<
                               CheckAll<[CheckOpcode<[BLR]>,
                                         CheckRegOperand<0, LR>]>>;

// Identify arithmetic instructions without or with limited extension or shift.
def ExynosArithFn   : TIIPredicate<
                        "isExynosArithFast",
                        MCOpcodeSwitchStatement<
                          [MCOpcodeSwitchCase<
                             IsArithExtOp.ValidOpcodes,
                             MCReturnStatement<
                               CheckAny<[CheckExtBy0,
                                         CheckAll<
                                           [CheckAny<
                                             [CheckExtUXTW,
                                              CheckExtUXTX]>,
                                            CheckAny<
                                              [CheckExtBy1,
                                               CheckExtBy2,
                                               CheckExtBy3]>]>]>>>,
                           MCOpcodeSwitchCase<
                             IsArithShiftOp.ValidOpcodes,
                             MCReturnStatement<ExynosCheckShift>>,
                           MCOpcodeSwitchCase<
                             IsArithUnshiftOp.ValidOpcodes,
                             MCReturnStatement<TruePred>>,
```
**EN:** This definition materializes ExynosBranchLinkLRPred as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 ExynosBranchLinkLRPred 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 54-74: TableGen definition ExynosArithPred
```tablegen
                           MCOpcodeSwitchCase<
                             IsArithImmOp.ValidOpcodes,
                             MCReturnStatement<TruePred>>],
                          MCReturnStatement<FalsePred>>>;
def ExynosArithPred : MCSchedPredicate<ExynosArithFn>;

// Identify logic instructions with limited shift.
def ExynosLogicFn   : TIIPredicate<
                        "isExynosLogicFast",
                        MCOpcodeSwitchStatement<
                          [MCOpcodeSwitchCase<
                             IsLogicShiftOp.ValidOpcodes,
                             MCReturnStatement<ExynosCheckShift>>,
                           MCOpcodeSwitchCase<
                             IsLogicUnshiftOp.ValidOpcodes,
                             MCReturnStatement<TruePred>>,
                           MCOpcodeSwitchCase<
                             IsLogicImmOp.ValidOpcodes,
                             MCReturnStatement<TruePred>>],
                          MCReturnStatement<FalsePred>>>;
def ExynosLogicPred : MCSchedPredicate<ExynosLogicFn>;
```
**EN:** This definition materializes ExynosArithPred as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 ExynosArithPred 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 75-95: TableGen definition ExynosLogicExFn
```tablegen

// Identify more logic instructions with limited shift.
def ExynosLogicExFn   : TIIPredicate<
                          "isExynosLogicExFast",
                          MCOpcodeSwitchStatement<
                            [MCOpcodeSwitchCase<
                               IsLogicShiftOp.ValidOpcodes,
                               MCReturnStatement<
                                 CheckAny<
                                   [ExynosCheckShift,
                                    CheckAll<
                                     [CheckShiftLSL,
                                      CheckShiftBy8]>]>>>,
                             MCOpcodeSwitchCase<
                               IsLogicUnshiftOp.ValidOpcodes,
                               MCReturnStatement<TruePred>>,
                             MCOpcodeSwitchCase<
                               IsLogicImmOp.ValidOpcodes,
                               MCReturnStatement<TruePred>>],
                            MCReturnStatement<FalsePred>>>;
def ExynosLogicExPred : MCSchedPredicate<ExynosLogicExFn>;
```
**EN:** This definition materializes ExynosLogicExFn as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 ExynosLogicExFn 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 96-115: TableGen definition ExynosScaledIdxFn
```tablegen

// Identify a load or store using the register offset addressing mode
// with a scaled non-extended register.
def ExynosScaledIdxFn   : TIIPredicate<"isExynosScaledAddr",
                                       MCOpcodeSwitchStatement<
                                         [MCOpcodeSwitchCase<
                                            IsLoadStoreRegOffsetOp.ValidOpcodes,
                                            MCReturnStatement<
                                              CheckAny<
                                                [CheckMemExtSXTW,
                                                 CheckMemExtUXTW,
                                                 CheckMemScaled]>>>],
                                         MCReturnStatement<FalsePred>>>;
def ExynosScaledIdxPred : MCSchedPredicate<ExynosScaledIdxFn>;

// Identify FP instructions.
def ExynosFPPred : MCSchedPredicate<CheckFpOrNEON>;

// Identify 128-bit NEON instructions.
def ExynosQFormPred : MCSchedPredicate<CheckQForm>;
```
**EN:** This definition materializes ExynosScaledIdxFn as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 ExynosScaledIdxFn 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 116-133: TableGen definition ExynosResetFn
```tablegen

// Identify instructions that reset a register efficiently.
def ExynosResetFn   : TIIPredicate<
                        "isExynosResetFast",
                        MCOpcodeSwitchStatement<
                          [MCOpcodeSwitchCase<
                             [ADR, ADRP,
                              MOVNWi, MOVNXi,
                              MOVZWi, MOVZXi],
                             MCReturnStatement<TruePred>>,
                           MCOpcodeSwitchCase<
                             [ORRWri, ORRXri],
                             MCReturnStatement<CheckIsReg1Zero>>],
                          MCReturnStatement<
                            CheckAny<
                              [IsCopyIdiomFn,
                               IsZeroFPIdiomFn]>>>>;
def ExynosResetPred : MCSchedPredicate<ExynosResetFn>;
```
**EN:** This definition materializes ExynosResetFn as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 ExynosResetFn 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 134-144: TableGen definition ExynosCheapFn
```tablegen

// Identify cheap arithmetic and logic immediate instructions.
def ExynosCheapFn : TIIPredicate<
                      "isExynosCheapAsMove",
                      MCOpcodeSwitchStatement<
                        [MCOpcodeSwitchCase<
                           IsArithLogicImmOp.ValidOpcodes,
                           MCReturnStatement<TruePred>>],
                        MCReturnStatement<
                          CheckAny<
                            [ExynosArithFn, ExynosResetFn, ExynosLogicFn]>>>>;
```
**EN:** This definition materializes ExynosCheapFn as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 ExynosCheapFn 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Per-core scheduling itineraries and latency models **CN:** 面向具体内核的调度行程与时延模型
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for scheduling models. **CN:** 与周边负责调度模型的 AArch64 后端组件紧密协作。
