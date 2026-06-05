# ARMScheduleA57.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleA57.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines the machine model for ARM Cortex-A57 to support instruction scheduling and other instruction cost heuristics.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleA57`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```tablegen
//=- ARMScheduleA57.td - ARM Cortex-A57 Scheduling Defs -----*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the machine model for ARM Cortex-A57 to support
// instruction scheduling and other instruction cost heuristics.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 14-22
```tablegen
//===----------------------------------------------------------------------===//
// *** Common description and scheduling model parameters taken from AArch64 ***
// The Cortex-A57 is a traditional superscalar microprocessor with a
// conservative 3-wide in-order stage for decode and dispatch. Combined with the
// much wider out-of-order issue stage, this produced a need to carefully
// schedule micro-ops so that all three decoded each cycle are successfully
// issued as the reservation station(s) simply don't stay occupied for long.
// Therefore, IssueWidth is set to the narrower of the two at three, while still
// modeling the machine as out-of-order.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 24-26
```tablegen
def IsCPSRDefinedAndPredicated : CheckAll<[IsCPSRDefined, IsPredicated]>;
def IsCPSRDefinedAndPredicatedPred :
    MCSchedPredicate<IsCPSRDefinedAndPredicated>;
```
- EN: Defines TableGen record `IsCPSRDefinedAndPredicated` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsCPSRDefinedAndPredicated`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 28-29
```tablegen
// Cortex A57 rev. r1p0 or later (false = r0px)
def IsR1P0AndLaterPred : MCSchedPredicate<FalsePred>;
```
- EN: Defines TableGen record `IsR1P0AndLaterPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsR1P0AndLaterPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 31-33
```tablegen
def IsLdrAm3RegOffPred : MCSchedPredicate<CheckInvalidRegOperand<2>>;
def IsLdrAm3RegOffPredX2 : MCSchedPredicate<CheckInvalidRegOperand<3>>;
def IsLdrAm3RegOffPredX3 : MCSchedPredicate<CheckInvalidRegOperand<4>>;
```
- EN: Defines TableGen record `IsLdrAm3RegOffPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLdrAm3RegOffPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 35-38
```tablegen
// If Addrmode3 contains "minus register"
class Am3NegativeRegOffset<int n> : MCSchedPredicate<CheckAll<[
                                      CheckValidRegOperand<n>,
                                      CheckAM3OpSub<!add(n, 1)>]>>;
```
- EN: Declares reusable TableGen class `Am3NegativeRegOffset` for `ARMScheduleA57`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMScheduleA57` 声明可复用的 TableGen 类 `Am3NegativeRegOffset`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-42
```tablegen
def IsLdrAm3NegRegOffPred : Am3NegativeRegOffset<2>;
def IsLdrAm3NegRegOffPredX2 : Am3NegativeRegOffset<3>;
def IsLdrAm3NegRegOffPredX3 : Am3NegativeRegOffset<4>;
```
- EN: Defines TableGen record `IsLdrAm3NegRegOffPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLdrAm3NegRegOffPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 44-54
```tablegen
// Load, scaled register offset, not plus LSL2
class ScaledRegNotPlusLsl2<int n> : CheckNot<
                                      CheckAny<[
                                        CheckAM2NoShift<n>,
                                        CheckAll<[
                                          CheckAM2OpAdd<n>,
                                          CheckAM2ShiftLSL<n>,
                                          CheckAM2Offset<n, 2>
                                        ]>
                                      ]>
                                    >;
```
- EN: Declares reusable TableGen class `ScaledRegNotPlusLsl2` for `ARMScheduleA57`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMScheduleA57` 声明可复用的 TableGen 类 `ScaledRegNotPlusLsl2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 56-58
```tablegen
def IsLdstsoScaledNotOptimalPredX0 : MCSchedPredicate<ScaledRegNotPlusLsl2<2>>;
def IsLdstsoScaledNotOptimalPred : MCSchedPredicate<ScaledRegNotPlusLsl2<3>>;
def IsLdstsoScaledNotOptimalPredX2 : MCSchedPredicate<ScaledRegNotPlusLsl2<4>>;
```
- EN: Defines TableGen record `IsLdstsoScaledNotOptimalPredX0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLdstsoScaledNotOptimalPredX0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 60-60
```tablegen
def IsLdstsoScaledPredX2 : MCSchedPredicate<CheckNot<CheckAM2NoShift<4>>>;
```
- EN: Defines TableGen record `IsLdstsoScaledPredX2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLdstsoScaledPredX2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 62-64
```tablegen
def IsLdstsoMinusRegPredX0 : MCSchedPredicate<CheckAM2OpSub<2>>;
def IsLdstsoMinusRegPred : MCSchedPredicate<CheckAM2OpSub<3>>;
def IsLdstsoMinusRegPredX2 : MCSchedPredicate<CheckAM2OpSub<4>>;
```
- EN: Defines TableGen record `IsLdstsoMinusRegPredX0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLdstsoMinusRegPredX0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 66-69
```tablegen
class A57WriteLMOpsListType<list<SchedWriteRes> writes> {
  list <SchedWriteRes> Writes = writes;
  SchedMachineModel SchedModel = ?;
}
```
- EN: Declares reusable TableGen class `A57WriteLMOpsListType` for `ARMScheduleA57`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMScheduleA57` 声明可复用的 TableGen 类 `A57WriteLMOpsListType`，通常用于抽象共享字段、谓词或编码结构。

### Lines 71-77
```tablegen
// *** Common description and scheduling model parameters taken from AArch64 ***
// (AArch64SchedA57.td)
def CortexA57Model : SchedMachineModel {
  let IssueWidth        =   3; // 3-way decode and dispatch
  let MicroOpBufferSize = 128; // 128 micro-op re-order buffer
  let LoadLatency       =   4; // Optimistic load latency
  let MispredictPenalty =  16; // Fetch + Decode/Rename/Dispatch + Branch
```
- EN: Defines TableGen record `CortexA57Model` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CortexA57Model`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 79-81
```tablegen
  // Enable partial & runtime unrolling.
  let LoopMicroOpBufferSize = 16;
  let CompleteModel = 1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 83-84
```tablegen
  // FIXME: Remove when all errors have been fixed.
  let FullInstRWOverlapCheck = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 86-88
```tablegen
  let UnsupportedFeatures = [HasV8_1MMainline, HasMVEInt, HasMVEFloat, IsMClass,
                             HasFPRegsV8_1M, HasFP16FML, HasMatMulInt8, HasBF16];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 90-93
```tablegen
//===----------------------------------------------------------------------===//
// Define each kind of processor resource and number available on Cortex-A57.
// Cortex A-57 has 8 pipelines that each has its own 8-entry queue where
// micro-ops wait for their operands and then issue out-of-order.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 95-99
```tablegen
def A57UnitB : ProcResource<1>;  // Type B micro-ops
def A57UnitI : ProcResource<2>;  // Type I micro-ops
def A57UnitM : ProcResource<1>;  // Type M micro-ops
def A57UnitL : ProcResource<1>;  // Type L micro-ops
def A57UnitS : ProcResource<1>;  // Type S micro-ops
```
- EN: Defines TableGen record `A57UnitB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57UnitB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 101-102
```tablegen
def A57UnitX : ProcResource<1>;  // Type X micro-ops (F1)
def A57UnitW : ProcResource<1>;  // Type W micro-ops (F0)
```
- EN: Defines TableGen record `A57UnitX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57UnitX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 104-106
```tablegen
let SchedModel = CortexA57Model in {
  def A57UnitV : ProcResGroup<[A57UnitX, A57UnitW]>;    // Type V micro-ops
}
```
- EN: Defines TableGen record `A57UnitV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57UnitV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 108-108
```tablegen
let SchedModel = CortexA57Model in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 110-111
```tablegen
//===----------------------------------------------------------------------===//
// Define customized scheduler read/write types specific to the Cortex-A57.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 113-113
```tablegen
include "ARMScheduleA57WriteRes.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 115-122
```tablegen
// To have "CompleteModel = 1", support of pseudos and special instructions
def : InstRW<[WriteNoop], (instregex "(t)?BKPT$", "(t2)?CDP(2)?$",
  "(t2)?CLREX$", "CONSTPOOL_ENTRY$", "COPY_STRUCT_BYVAL_I32$",
  "(t2)?CPS[123]p$", "(t2)?DBG$", "(t2)?DMB$", "(t2)?DSB$", "ERET$",
  "(t2|t)?HINT$", "(t)?HLT$", "(t2)?HVC$", "(t2)?ISB$", "ITasm$",
  "(t2)?RFE(DA|DB|IA|IB)", "(t)?SETEND", "(t2)?SETPAN", "(t2)?SMC", "SPACE",
  "(t2)?SRS(DA|DB|IA|IB)", "SWP(B)?", "t?TRAP", "(t2|t)?UDF$", "t2DCPS", "t2SG",
  "t2TT", "tCPS", "CMP_SWAP", "t?SVC", "t2IT", "t__brkdiv0")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 124-124
```tablegen
def : InstRW<[WriteNoop], (instregex "VMRS", "VMSR", "FMSTAT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 126-128
```tablegen
// Specific memory instrs
def : InstRW<[WriteNoop, WriteNoop], (instregex "(t2)?LDA", "(t2)?LDC", "(t2)?STC",
  "(t2)?STL", "(t2)?LDREX", "(t2)?STREX", "MEMCPY")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 130-134
```tablegen
// coprocessor moves
def : InstRW<[WriteNoop, WriteNoop], (instregex
  "(t2)?MCR(2|R|R2)?$", "(t2)?MRC(2)?$",
  "(t2)?MRRC(2)?$", "(t2)?MRS(banked|sys|_AR|_M|sys_AR)?$",
  "(t2)?MSR(banked|i|_AR|_M)?$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 136-137
```tablegen
// Deprecated instructions
def : InstRW<[WriteNoop], (instregex "FLDM", "FSTM")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 139-147
```tablegen
// Pseudos
def : InstRW<[WriteNoop], (instregex "(t)?ADJCALLSTACKDOWN$", "(t)?ADJCALLSTACKUP$",
  "(t2|t)?Int_eh_sjlj", "tLDRpci_pic", "(t2)?SUBS_PC_LR",
  "JUMPTABLE", "tInt_WIN_eh_sjlj_longjmp",
  "VLD(1|2)LN(d|q)(WB_fixed_|WB_register_)?Asm",
  "VLD(3|4)(DUP|LN)?(d|q)(WB_fixed_|WB_register_)?Asm",
  "VST(1|2)LN(d|q)(WB_fixed_|WB_register_)?Asm",
  "VST(3|4)(DUP|LN)?(d|q)(WB_fixed_|WB_register_)?Asm",
  "WIN__CHKSTK", "WIN__DBZCHK")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 149-150
```tablegen
// Miscellaneous
// -----------------------------------------------------------------------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 152-152
```tablegen
def : InstRW<[A57Write_1cyc_1I], (instrs COPY)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 154-155
```tablegen
// --- 3.2 Branch Instructions ---
// B, BX, BL, BLX (imm, reg != LR, reg == LR), CBZ, CBNZ
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 157-166
```tablegen
def : InstRW<[A57Write_1cyc_1B], (instregex "(t2|t)?B$", "t?BX", "(t2|t)?Bcc$",
  "t?TAILJMP(d|r)", "TCRETURN(d|r)i", "tBfar", "tCBN?Z")>;
def : InstRW<[A57Write_1cyc_1B_1I],
  (instregex "t?BL$", "BL_pred$", "t?BLXi", "t?TPsoft")>;
def : InstRW<[A57Write_2cyc_1B_1I], (instregex "BLX", "tBLX(NS)?r")>;
// Pseudos
def : InstRW<[A57Write_2cyc_1B_1I], (instregex "BCCi64", "BCCZi64")>;
def : InstRW<[A57Write_3cyc_1B_1I], (instregex "BR_JTadd", "t?BR_JTr",
  "t2BR_JT", "t2BXJ", "(t2)?TB(B|H)(_JT)?$", "tBRIND")>;
def : InstRW<[A57Write_6cyc_1B_1L], (instregex "BR_JTm")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 168-170
```tablegen
// --- 3.3 Arithmetic and Logical Instructions ---
// ADD{S}, ADC{S}, ADR,	AND{S},	BIC{S},	CMN, CMP, EOR{S}, ORN{S}, ORR{S},
// RSB{S}, RSC{S}, SUB{S}, SBC{S}, TEQ, TST
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 172-172
```tablegen
def : InstRW<[A57Write_1cyc_1I], (instregex "tADDframe")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 174-178
```tablegen
// Check branch forms of ALU ops:
// check reg 0 for ARM_AM::PC
// if so adds 2 cyc to latency, 1 uop, 1 res cycle for A57UnitB
class A57BranchForm<SchedWriteRes non_br> :
  BranchWriteRes<2, 1, [A57UnitB], [1], non_br>;
```
- EN: Declares reusable TableGen class `A57BranchForm` for `ARMScheduleA57`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMScheduleA57` 声明可复用的 TableGen 类 `A57BranchForm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 180-197
```tablegen
// shift by register, conditional or unconditional
// TODO: according to the doc, conditional uses I0/I1, unconditional uses M
// Why more complex instruction uses more simple pipeline?
// May be an error in doc.
def A57WriteALUsr : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [CheckBranchForm<0, A57BranchForm<A57Write_2cyc_1I>>]>,
  SchedVar<NoSchedPred,      [CheckBranchForm<0, A57BranchForm<A57Write_2cyc_1M>>]>
]>;
def A57WriteALUSsr : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [CheckBranchForm<0, A57BranchForm<A57Write_2cyc_1I>>]>,
  SchedVar<NoSchedPred,      [CheckBranchForm<0, A57BranchForm<A57Write_2cyc_1M>>]>
]>;
def A57ReadALUsr : SchedReadVariant<[
  SchedVar<IsPredicatedPred, [ReadDefault]>,
  SchedVar<NoSchedPred,      [ReadDefault]>
]>;
def : SchedAlias<WriteALUsi,  CheckBranchForm<0, A57BranchForm<A57Write_2cyc_1M>>>;
def : SchedAlias<WriteALUsr,  A57WriteALUsr>;
```
- EN: Defines TableGen record `A57WriteALUsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteALUsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 198-199
```tablegen
def : SchedAlias<WriteALUSsr, A57WriteALUSsr>;
def : SchedAlias<ReadALUsr,   A57ReadALUsr>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 201-207
```tablegen
def A57WriteCMPsr : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57Write_2cyc_1I]>,
  SchedVar<NoSchedPred,      [A57Write_2cyc_1M]>
]>;
def : SchedAlias<WriteCMP,   A57Write_1cyc_1I>;
def : SchedAlias<WriteCMPsi, A57Write_2cyc_1M>;
def : SchedAlias<WriteCMPsr, A57WriteCMPsr>;
```
- EN: Defines TableGen record `A57WriteCMPsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteCMPsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 209-214
```tablegen
// --- 3.4 Move and Shift Instructions ---
// Move, basic
// MOV{S}, MOVW, MVN{S}
def : InstRW<[A57Write_1cyc_1I], (instregex "MOV(r|i|i16|r_TC)",
  "(t2)?MVN(CC)?(r|i)", "BMOVPCB_CALL", "BMOVPCRX_CALL",
  "MOVCC(r|i|i16|i32imm)", "tMOV", "tMVN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 216-225
```tablegen
// Move, shift by immed, setflags/no setflags
// (ASR, LSL, LSR, ROR, RRX)=MOVsi, MVN
// setflags = isCPSRDefined
def A57WriteMOVsi : SchedWriteVariant<[
  SchedVar<IsCPSRDefinedPred,              [A57Write_2cyc_1M]>,
  SchedVar<NoSchedPred,                    [A57Write_1cyc_1I]>
]>;
def : InstRW<[A57WriteMOVsi], (instregex "MOV(CC)?si", "MVNsi",
  "ASRi", "(t2|t)ASRri", "LSRi", "(t2|t)LSRri", "LSLi", "(t2|t)LSLri", "RORi",
  "(t2|t)RORri", "(t2)?RRX", "t2MOV", "tROR")>;
```
- EN: Defines TableGen record `A57WriteMOVsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteMOVsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 227-236
```tablegen
// shift by register, conditional or unconditional, setflags/no setflags
def A57WriteMOVsr : SchedWriteVariant<[
  SchedVar<IsCPSRDefinedAndPredicatedPred, [A57Write_2cyc_1I]>,
  SchedVar<IsCPSRDefinedPred,              [A57Write_2cyc_1M]>,
  SchedVar<IsPredicatedPred,               [A57Write_2cyc_1I]>,
  SchedVar<NoSchedPred,                    [A57Write_1cyc_1I]>
]>;
def : InstRW<[A57WriteMOVsr], (instregex "MOV(CC)?sr", "MVNsr", "t2MVNs",
  "ASRr", "(t2|t)ASRrr", "LSRr", "(t2|t)LSRrr", "LSLr", "(t2|t)?LSLrr", "RORr",
  "(t2|t)RORrr")>;
```
- EN: Defines TableGen record `A57WriteMOVsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteMOVsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-244
```tablegen
// Move, top
// MOVT - A57Write_2cyc_1M for r0px, A57Write_1cyc_1I for r1p0 and later
def A57WriteMOVT : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred,             [A57Write_1cyc_1I]>,
  SchedVar<NoSchedPred,                    [A57Write_2cyc_1M]>
]>;
def : InstRW<[A57WriteMOVT], (instregex "MOVTi16")>;
```
- EN: Defines TableGen record `A57WriteMOVT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteMOVT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 246-251
```tablegen
def A57WriteI2pc :
  WriteSequence<[A57Write_1cyc_1I, A57Write_1cyc_1I, A57Write_1cyc_1I]>;
def A57WriteI2ld :
  WriteSequence<[A57Write_1cyc_1I, A57Write_1cyc_1I, A57Write_4cyc_1L]>;
def : InstRW< [A57WriteI2pc], (instregex "MOV_ga_pcrel")>;
def : InstRW< [A57WriteI2ld], (instregex "MOV_ga_pcrel_ldr")>;
```
- EN: Defines TableGen record `A57WriteI2pc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteI2pc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 253-254
```tablegen
// +2cyc for branch forms
def : InstRW<[A57Write_3cyc_1I], (instregex "MOVPC(LR|RX)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 256-264
```tablegen
// --- 3.5 Divide and Multiply Instructions ---
// Divide: SDIV, UDIV
// latency from documentration: 4 ­‐ 20, maximum taken
def : SchedAlias<WriteDIV, A57Write_20cyc_1M>;
// Multiply: tMul not bound to common WriteRes types
def : InstRW<[A57Write_3cyc_1M], (instregex "tMUL")>;
def : SchedAlias<WriteMUL16, A57Write_3cyc_1M>;
def : SchedAlias<WriteMUL32, A57Write_3cyc_1M>;
def : ReadAdvance<ReadMUL, 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 266-275
```tablegen
// Multiply accumulate: MLA, MLS, SMLABB, SMLABT, SMLATB, SMLATT, SMLAWB,
// SMLAWT, SMLAD{X}, SMLSD{X}, SMMLA{R}, SMMLS{R}
// Multiply-accumulate pipelines support late-forwarding of accumulate operands
// from similar μops, allowing a typical sequence of multiply-accumulate μops
// to issue one every 1 cycle (sched advance = 2).
def A57WriteMLA : SchedWriteRes<[A57UnitM]> { let Latency = 3; }
def A57WriteMLAL : SchedWriteVariant<[
  SchedVar<IsCPSRDefinedPred, [A57Write_5cyc_1I_1M]>,
  SchedVar<NoSchedPred,       [A57Write_4cyc_1M]>
]>;
```
- EN: Defines TableGen record `A57WriteMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 277-277
```tablegen
def A57ReadMLA  : SchedReadAdvance<2, [A57WriteMLA, A57WriteMLAL]>;
```
- EN: Defines TableGen record `A57ReadMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57ReadMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 279-280
```tablegen
def : InstRW<[A57WriteMLA],
  (instregex "t2SMLAD", "t2SMLADX", "t2SMLSD", "t2SMLSDX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 282-284
```tablegen
def : SchedAlias<WriteMAC16, A57WriteMLA>;
def : SchedAlias<WriteMAC32, A57WriteMLA>;
def : SchedAlias<ReadMAC,    A57ReadMLA>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 286-287
```tablegen
def : SchedAlias<WriteMAC64Lo, A57WriteMLAL>;
def : SchedAlias<WriteMAC64Hi, A57WriteMLAL>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 289-291
```tablegen
// Multiply long: SMULL, UMULL
def : SchedAlias<WriteMUL64Lo, A57Write_4cyc_1M>;
def : SchedAlias<WriteMUL64Hi, A57Write_4cyc_1M>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 293-304
```tablegen
// --- 3.6 Saturating and Parallel Arithmetic Instructions ---
// Parallel	arith
// SADD16, SADD8, SSUB16, SSUB8, UADD16, UADD8, USUB16, USUB8
// Conditional GE-setting instructions require three extra μops
// and two additional cycles to conditionally update the GE field.
def A57WriteParArith : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57Write_4cyc_1I_1M]>,
  SchedVar<NoSchedPred,      [A57Write_2cyc_1I_1M]>
]>;
def : InstRW< [A57WriteParArith], (instregex
  "(t2)?SADD(16|8)", "(t2)?SSUB(16|8)",
  "(t2)?UADD(16|8)", "(t2)?USUB(16|8)")>;
```
- EN: Defines TableGen record `A57WriteParArith` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteParArith`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 306-312
```tablegen
// Parallel	arith with exchange: SASX, SSAX, UASX, USAX
def A57WriteParArithExch : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57Write_5cyc_1I_1M]>,
  SchedVar<NoSchedPred,      [A57Write_3cyc_1I_1M]>
]>;
def : InstRW<[A57WriteParArithExch],
  (instregex "(t2)?SASX", "(t2)?SSAX", "(t2)?UASX", "(t2)?USAX")>;
```
- EN: Defines TableGen record `A57WriteParArithExch` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteParArithExch`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 314-318
```tablegen
// Parallel	halving	arith
// SHADD16, SHADD8, SHSUB16, SHSUB8, UHADD16, UHADD8, UHSUB16,	UHSUB8
def : InstRW<[A57Write_2cyc_1M], (instregex
  "(t2)?SHADD(16|8)", "(t2)?SHSUB(16|8)",
  "(t2)?UHADD(16|8)", "(t2)?UHSUB(16|8)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 320-323
```tablegen
// Parallel halving arith with exchange
// SHASX, SHSAX, UHASX, UHSAX
def : InstRW<[A57Write_3cyc_1I_1M], (instregex "(t2)?SHASX", "(t2)?SHSAX",
  "(t2)?UHASX", "(t2)?UHSAX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 325-328
```tablegen
// Parallel	saturating arith
// QADD16, QADD8, QSUB16, QSUB8, UQADD16, UQADD8, UQSUB16, UQSUB8
def : InstRW<[A57Write_2cyc_1M], (instregex "QADD(16|8)", "QSUB(16|8)",
  "UQADD(16|8)", "UQSUB(16|8)", "t2(U?)QADD", "t2(U?)QSUB")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 330-333
```tablegen
// Parallel	saturating arith with exchange
// QASX, QSAX, UQASX, UQSAX
def : InstRW<[A57Write_3cyc_1I_1M], (instregex "(t2)?QASX", "(t2)?QSAX",
  "(t2)?UQASX", "(t2)?UQSAX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 335-337
```tablegen
// Saturate: SSAT, SSAT16, USAT, USAT16
def : InstRW<[A57Write_2cyc_1M],
  (instregex "(t2)?SSAT(16)?", "(t2)?USAT(16)?")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 339-340
```tablegen
// Saturating arith: QADD, QSUB
def : InstRW<[A57Write_2cyc_1M], (instregex "QADD$", "QSUB$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 342-343
```tablegen
// Saturating doubling arith: QDADD, QDSUB
def : InstRW<[A57Write_3cyc_1I_1M], (instregex "(t2)?QDADD", "(t2)?QDSUB")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 345-347
```tablegen
// --- 3.7 Miscellaneous Data-Processing Instructions ---
// Bit field extract: SBFX, UBFX
def : InstRW<[A57Write_1cyc_1I], (instregex "(t2)?SBFX", "(t2)?UBFX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 349-350
```tablegen
// Bit field insert/clear: BFI, BFC
def : InstRW<[A57Write_2cyc_1M], (instregex "(t2)?BFI", "(t2)?BFC")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 352-357
```tablegen
// Select bytes, conditional/unconditional
def A57WriteSEL : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57Write_2cyc_1I]>,
  SchedVar<NoSchedPred,      [A57Write_1cyc_1I]>
]>;
def : InstRW<[A57WriteSEL], (instregex "(t2)?SEL")>;
```
- EN: Defines TableGen record `A57WriteSEL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteSEL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 359-361
```tablegen
// Sign/zero extend, normal: SXTB, SXTH, UXTB, UXTH
def : InstRW<[A57Write_1cyc_1I],
  (instregex "(t2|t)?SXT(B|H)$", "(t2|t)?UXT(B|H)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 363-365
```tablegen
// Sign/zero extend and add, normal: SXTAB, SXTAH, UXTAB, UXTAH
def : InstRW<[A57Write_2cyc_1M],
  (instregex "(t2)?SXTA(B|H)$", "(t2)?UXTA(B|H)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 367-368
```tablegen
// Sign/zero extend and add, parallel: SXTAB16, UXTAB16
def : InstRW<[A57Write_4cyc_1M], (instregex "(t2)?SXTAB16", "(t2)?UXTAB16")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 370-371
```tablegen
// Sum of absolute differences: USAD8, USADA8
def : InstRW<[A57Write_3cyc_1M], (instregex "(t2)?USAD8", "(t2)?USADA8")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 373-373
```tablegen
// --- 3.8 Load Instructions ---
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 375-379
```tablegen
// Load, immed offset
// LDR and LDRB have LDRi12 and LDRBi12 forms for immediate
def : InstRW<[A57Write_4cyc_1L], (instregex "LDRi12", "LDRBi12",
  "LDRcp", "(t2|t)?LDRConstPool", "LDRLIT_ga_(pcrel|abs)",
  "PICLDR", "tLDR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 381-382
```tablegen
def : InstRW<[A57Write_4cyc_1L],
  (instregex "t2LDRS?(B|H)?(pcrel|T|i8|i12|pci|pci_pic|s)?$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 384-395
```tablegen
// For "Load, register offset, minus" we need +1cyc, +1I
def A57WriteLdrAm3 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPred, [A57Write_5cyc_1I_1L]>,
  SchedVar<NoSchedPred,           [A57Write_4cyc_1L]>
]>;
def : InstRW<[A57WriteLdrAm3], (instregex "LDR(H|SH|SB)$")>;
def A57WriteLdrAm3X2 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPredX2, [A57Write_5cyc_1I_1L]>,
  SchedVar<NoSchedPred,             [A57Write_4cyc_1L]>
]>;
def : InstRW<[A57WriteLdrAm3X2, A57WriteLdrAm3X2], (instregex "LDRD$")>;
def : InstRW<[A57Write_4cyc_1L, A57Write_4cyc_1L], (instregex "t2LDRDi8")>;
```
- EN: Defines TableGen record `A57WriteLdrAm3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrAm3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 397-402
```tablegen
def A57WriteLdrAmLDSTSO : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledNotOptimalPred, [A57Write_5cyc_1I_1L]>,
  SchedVar<IsLdstsoMinusRegPred,         [A57Write_5cyc_1I_1L]>,
  SchedVar<NoSchedPred,                  [A57Write_4cyc_1L]>
]>;
def : InstRW<[A57WriteLdrAmLDSTSO], (instregex "LDRrs", "LDRBrs")>;
```
- EN: Defines TableGen record `A57WriteLdrAmLDSTSO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrAmLDSTSO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 404-415
```tablegen
def A57WrBackOne : SchedWriteRes<[]> {
  let Latency = 1;
  let NumMicroOps = 0;
}
def A57WrBackTwo : SchedWriteRes<[]> {
  let Latency = 2;
  let NumMicroOps = 0;
}
def A57WrBackThree : SchedWriteRes<[]> {
  let Latency = 3;
  let NumMicroOps = 0;
}
```
- EN: Defines TableGen record `A57WrBackOne` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WrBackOne`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 417-420
```tablegen
// --- LDR pre-indexed ---
// Load, immed pre-indexed (4 cyc for load result, 1 cyc for Base update)
def : InstRW<[A57Write_4cyc_1L_1I, A57WrBackOne], (instregex "LDR_PRE_IMM",
  "LDRB_PRE_IMM", "t2LDRB_PRE")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 422-429
```tablegen
// Load, register pre-indexed (4 cyc for load result, 2 cyc for Base update)
// (5 cyc load result for not-lsl2 scaled)
def A57WriteLdrAmLDSTSOPre : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledNotOptimalPredX2, [A57Write_5cyc_1I_1L]>,
  SchedVar<NoSchedPred,                    [A57Write_4cyc_1L_1I]>
]>;
def : InstRW<[A57WriteLdrAmLDSTSOPre, A57WrBackTwo],
  (instregex "LDR_PRE_REG", "LDRB_PRE_REG")>;
```
- EN: Defines TableGen record `A57WriteLdrAmLDSTSOPre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrAmLDSTSOPre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 431-438
```tablegen
def A57WriteLdrAm3PreWrBack : SchedWriteVariant<[
  SchedVar<IsLdrAm3RegOffPredX2, [A57WrBackTwo]>,
  SchedVar<NoSchedPred,          [A57WrBackOne]>
]>;
def : InstRW<[A57Write_4cyc_1L, A57WriteLdrAm3PreWrBack],
  (instregex "LDR(H|SH|SB)_PRE")>;
def : InstRW<[A57Write_4cyc_1L, A57WrBackOne],
  (instregex "t2LDR(H|SH|SB)?_PRE")>;
```
- EN: Defines TableGen record `A57WriteLdrAm3PreWrBack` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrAm3PreWrBack`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 440-452
```tablegen
// LDRD pre-indexed: 5(2) cyc for reg, 4(1) cyc for imm.
def A57WriteLdrDAm3Pre : SchedWriteVariant<[
  SchedVar<IsLdrAm3RegOffPredX3, [A57Write_5cyc_1I_1L]>,
  SchedVar<NoSchedPred,          [A57Write_4cyc_1L_1I]>
]>;
def A57WriteLdrDAm3PreWrBack : SchedWriteVariant<[
  SchedVar<IsLdrAm3RegOffPredX3, [A57WrBackTwo]>,
  SchedVar<NoSchedPred,          [A57WrBackOne]>
]>;
def : InstRW<[A57WriteLdrDAm3Pre, A57WriteLdrDAm3Pre, A57WriteLdrDAm3PreWrBack],
  (instregex "LDRD_PRE")>;
def : InstRW<[A57Write_4cyc_1L_1I, A57Write_4cyc_1L_1I, A57WrBackOne],
  (instregex "t2LDRD_PRE")>;
```
- EN: Defines TableGen record `A57WriteLdrDAm3Pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrDAm3Pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 454-456
```tablegen
// --- LDR post-indexed ---
def : InstRW<[A57Write_4cyc_1L_1I, A57WrBackOne], (instregex "LDR(T?)_POST_IMM",
  "LDRB(T?)_POST_IMM", "LDR(SB|H|SH)Ti", "t2LDRB_POST")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 458-465
```tablegen
def A57WriteLdrAm3PostWrBack : SchedWriteVariant<[
  SchedVar<IsLdrAm3RegOffPred, [A57WrBackTwo]>,
  SchedVar<NoSchedPred,        [A57WrBackOne]>
]>;
def : InstRW<[A57Write_4cyc_1L_1I, A57WriteLdrAm3PostWrBack],
  (instregex "LDR(H|SH|SB)_POST")>;
def : InstRW<[A57Write_4cyc_1L, A57WrBackOne],
  (instregex "t2LDR(H|SH|SB)?_POST")>;
```
- EN: Defines TableGen record `A57WriteLdrAm3PostWrBack` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrAm3PostWrBack`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 467-468
```tablegen
def : InstRW<[A57Write_4cyc_1L_1I, A57WrBackTwo], (instregex "LDR_POST_REG",
  "LDRB_POST_REG", "LDR(B?)T_POST$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 470-480
```tablegen
def A57WriteLdrTRegPost : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledPredX2, [A57Write_4cyc_1I_1L_1M]>,
  SchedVar<NoSchedPred,        [A57Write_4cyc_1L_1I]>
]>;
def A57WriteLdrTRegPostWrBack : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledPredX2, [A57WrBackThree]>,
  SchedVar<NoSchedPred,        [A57WrBackTwo]>
]>;
// 4(3) "I0/I1,L,M" for scaled register, otherwise 4(2) "I0/I1,L"
def : InstRW<[A57WriteLdrTRegPost, A57WriteLdrTRegPostWrBack],
  (instregex "LDRT_POST_REG", "LDRBT_POST_REG")>;
```
- EN: Defines TableGen record `A57WriteLdrTRegPost` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrTRegPost`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 482-482
```tablegen
def : InstRW<[A57Write_4cyc_1L_1I, A57WrBackTwo], (instregex "LDR(SB|H|SH)Tr")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 484-492
```tablegen
def A57WriteLdrAm3PostWrBackX3 : SchedWriteVariant<[
  SchedVar<IsLdrAm3RegOffPredX3, [A57WrBackTwo]>,
  SchedVar<NoSchedPred,          [A57WrBackOne]>
]>;
// LDRD post-indexed: 4(2) cyc for reg, 4(1) cyc for imm.
def : InstRW<[A57Write_4cyc_1L_1I, A57Write_4cyc_1L_1I,
  A57WriteLdrAm3PostWrBackX3], (instregex "LDRD_POST")>;
def : InstRW<[A57Write_4cyc_1L_1I, A57Write_4cyc_1L_1I, A57WrBackOne],
  (instregex "t2LDRD_POST")>;
```
- EN: Defines TableGen record `A57WriteLdrAm3PostWrBackX3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLdrAm3PostWrBackX3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 494-497
```tablegen
// --- Preload instructions ---
// Preload, immed offset
def : InstRW<[A57Write_4cyc_1L], (instregex "(t2)?PLDi12", "(t2)?PLDWi12",
  "t2PLDW?(i8|pci|s)", "(t2)?PLI")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 499-507
```tablegen
// Preload, register offset,
// 5cyc "I0/I1,L" for minus reg or scaled not plus lsl2
// otherwise 4cyc "L"
def A57WritePLD : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledNotOptimalPredX0, [A57Write_5cyc_1I_1L]>,
  SchedVar<IsLdstsoMinusRegPredX0,         [A57Write_5cyc_1I_1L]>,
  SchedVar<NoSchedPred,                    [A57Write_4cyc_1L]>
]>;
def : InstRW<[A57WritePLD], (instregex "PLDrs", "PLDWrs")>;
```
- EN: Defines TableGen record `A57WritePLD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WritePLD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 509-517
```tablegen
// --- Load multiple instructions ---
foreach NumAddr = 1-8 in {
  def A57LMAddrPred#NumAddr : MCSchedPredicate<CheckAny<[
                                CheckNumOperands<!add(!shl(NumAddr, 1), 2)>,
                                CheckNumOperands<!add(!shl(NumAddr, 1), 3)>]>>;
  def A57LMAddrUpdPred#NumAddr : MCSchedPredicate<CheckAny<[
                                   CheckNumOperands<!add(!shl(NumAddr, 1), 3)>,
                                   CheckNumOperands<!add(!shl(NumAddr, 1), 4)>]>>;
}
```
- EN: Defines TableGen record `A57LMAddrPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57LMAddrPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 519-536
```tablegen
def A57LDMOpsListNoregin : A57WriteLMOpsListType<
                [A57Write_3cyc_1L, A57Write_3cyc_1L,
                 A57Write_4cyc_1L, A57Write_4cyc_1L,
                 A57Write_5cyc_1L, A57Write_5cyc_1L,
                 A57Write_6cyc_1L, A57Write_6cyc_1L,
                 A57Write_7cyc_1L, A57Write_7cyc_1L,
                 A57Write_8cyc_1L, A57Write_8cyc_1L,
                 A57Write_9cyc_1L, A57Write_9cyc_1L,
                 A57Write_10cyc_1L, A57Write_10cyc_1L]>;
def A57WriteLDMnoreginlist : SchedWriteVariant<[
  SchedVar<A57LMAddrPred1,     A57LDMOpsListNoregin.Writes[0-1]>,
  SchedVar<A57LMAddrPred2,     A57LDMOpsListNoregin.Writes[0-3]>,
  SchedVar<A57LMAddrPred3,     A57LDMOpsListNoregin.Writes[0-5]>,
  SchedVar<A57LMAddrPred4,     A57LDMOpsListNoregin.Writes[0-7]>,
  SchedVar<A57LMAddrPred5,     A57LDMOpsListNoregin.Writes[0-9]>,
  SchedVar<A57LMAddrPred6,     A57LDMOpsListNoregin.Writes[0-11]>,
  SchedVar<A57LMAddrPred7,     A57LDMOpsListNoregin.Writes[0-13]>,
  SchedVar<A57LMAddrPred8,     A57LDMOpsListNoregin.Writes[0-15]>,
```
- EN: Defines TableGen record `A57LDMOpsListNoregin` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57LDMOpsListNoregin`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 537-538
```tablegen
  SchedVar<NoSchedPred,        A57LDMOpsListNoregin.Writes[0-15]>
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 540-557
```tablegen
def A57LDMOpsListRegin : A57WriteLMOpsListType<
                [A57Write_4cyc_1L_1I, A57Write_4cyc_1L_1I,
                 A57Write_5cyc_1L_1I, A57Write_5cyc_1L_1I,
                 A57Write_6cyc_1L_1I, A57Write_6cyc_1L_1I,
                 A57Write_7cyc_1L_1I, A57Write_7cyc_1L_1I,
                 A57Write_8cyc_1L_1I, A57Write_8cyc_1L_1I,
                 A57Write_9cyc_1L_1I, A57Write_9cyc_1L_1I,
                 A57Write_10cyc_1L_1I, A57Write_10cyc_1L_1I,
                 A57Write_11cyc_1L_1I, A57Write_11cyc_1L_1I]>;
def A57WriteLDMreginlist : SchedWriteVariant<[
  SchedVar<A57LMAddrPred1,     A57LDMOpsListRegin.Writes[0-1]>,
  SchedVar<A57LMAddrPred2,     A57LDMOpsListRegin.Writes[0-3]>,
  SchedVar<A57LMAddrPred3,     A57LDMOpsListRegin.Writes[0-5]>,
  SchedVar<A57LMAddrPred4,     A57LDMOpsListRegin.Writes[0-7]>,
  SchedVar<A57LMAddrPred5,     A57LDMOpsListRegin.Writes[0-9]>,
  SchedVar<A57LMAddrPred6,     A57LDMOpsListRegin.Writes[0-11]>,
  SchedVar<A57LMAddrPred7,     A57LDMOpsListRegin.Writes[0-13]>,
  SchedVar<A57LMAddrPred8,     A57LDMOpsListRegin.Writes[0-15]>,
```
- EN: Defines TableGen record `A57LDMOpsListRegin` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57LDMOpsListRegin`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 558-559
```tablegen
  SchedVar<NoSchedPred,        A57LDMOpsListRegin.Writes[0-15]>
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 561-578
```tablegen
def A57LDMOpsList_Upd : A57WriteLMOpsListType<
              [A57WrBackOne,
               A57Write_3cyc_1L_1I, A57Write_3cyc_1L_1I,
               A57Write_4cyc_1L_1I, A57Write_4cyc_1L_1I,
               A57Write_5cyc_1L_1I, A57Write_5cyc_1L_1I,
               A57Write_6cyc_1L_1I, A57Write_6cyc_1L_1I,
               A57Write_7cyc_1L_1I, A57Write_7cyc_1L_1I,
               A57Write_8cyc_1L_1I, A57Write_8cyc_1L_1I,
               A57Write_9cyc_1L_1I, A57Write_9cyc_1L_1I,
               A57Write_10cyc_1L_1I, A57Write_10cyc_1L_1I]>;
def A57WriteLDM_Upd : SchedWriteVariant<[
  SchedVar<A57LMAddrUpdPred1,     A57LDMOpsList_Upd.Writes[0-2]>,
  SchedVar<A57LMAddrUpdPred2,     A57LDMOpsList_Upd.Writes[0-4]>,
  SchedVar<A57LMAddrUpdPred3,     A57LDMOpsList_Upd.Writes[0-6]>,
  SchedVar<A57LMAddrUpdPred4,     A57LDMOpsList_Upd.Writes[0-8]>,
  SchedVar<A57LMAddrUpdPred5,     A57LDMOpsList_Upd.Writes[0-10]>,
  SchedVar<A57LMAddrUpdPred6,     A57LDMOpsList_Upd.Writes[0-12]>,
  SchedVar<A57LMAddrUpdPred7,     A57LDMOpsList_Upd.Writes[0-14]>,
```
- EN: Defines TableGen record `A57LDMOpsList_Upd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57LDMOpsList_Upd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 579-581
```tablegen
  SchedVar<A57LMAddrUpdPred8,     A57LDMOpsList_Upd.Writes[0-16]>,
  SchedVar<NoSchedPred,           A57LDMOpsList_Upd.Writes[0-16]>
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 583-586
```tablegen
def A57WriteLDM : SchedWriteVariant<[
  SchedVar<IsLDMBaseRegInListPred, [A57WriteLDMreginlist]>,
  SchedVar<NoSchedPred,            [A57WriteLDMnoreginlist]>
]> { let Variadic=1; }
```
- EN: Defines TableGen record `A57WriteLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 588-588
```tablegen
def : InstRW<[A57WriteLDM], (instregex "(t|t2|sys)?LDM(IA|DA|DB|IB)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 590-592
```tablegen
// TODO: no writeback latency defined in documentation (implemented as 1 cyc)
def : InstRW<[A57WriteLDM_Upd],
  (instregex "(t|t2|sys)?LDM(IA_UPD|DA_UPD|DB_UPD|IB_UPD|IA_RET)", "tPOP")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 594-594
```tablegen
def : InstRW<[A57Write_5cyc_1L], (instregex "VLLDM")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 596-596
```tablegen
// --- 3.9 Store Instructions ---
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 598-600
```tablegen
// Store, immed offset
def : InstRW<[A57Write_1cyc_1S], (instregex "STRi12", "STRBi12", "PICSTR",
  "t2STR(B?)(T|i12|i8|s)", "t2STRDi8", "t2STRH(i12|i8|s)", "tSTR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 602-610
```tablegen
// Store, register offset
// For minus or for not plus lsl2 scaled we need 3cyc "I0/I1, S",
// otherwise 1cyc S.
def A57WriteStrAmLDSTSO : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledNotOptimalPred, [A57Write_3cyc_1I_1S]>,
  SchedVar<IsLdstsoMinusRegPred,         [A57Write_3cyc_1I_1S]>,
  SchedVar<NoSchedPred,                  [A57Write_1cyc_1S]>
]>;
def : InstRW<[A57WriteStrAmLDSTSO], (instregex "STRrs", "STRBrs")>;
```
- EN: Defines TableGen record `A57WriteStrAmLDSTSO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteStrAmLDSTSO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 612-622
```tablegen
// STRH,STRD: 3cyc "I0/I1, S" for minus reg, 1cyc S for imm or for plus reg.
def A57WriteStrAm3 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPred, [A57Write_3cyc_1I_1S]>,
  SchedVar<NoSchedPred,           [A57Write_1cyc_1S]>
]>;
def : InstRW<[A57WriteStrAm3], (instregex "STRH$")>;
def A57WriteStrAm3X2 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPredX2, [A57Write_3cyc_1I_1S]>,
  SchedVar<NoSchedPred,             [A57Write_1cyc_1S]>
]>;
def : InstRW<[A57WriteStrAm3X2], (instregex "STRD$")>;
```
- EN: Defines TableGen record `A57WriteStrAm3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteStrAm3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 624-627
```tablegen
// Store, immed pre-indexed (1cyc "S, I0/I1", 1cyc writeback)
def : InstRW<[A57WrBackOne, A57Write_1cyc_1S_1I], (instregex "STR_PRE_IMM",
  "STRB_PRE_IMM", "STR(B)?(r|i)_preidx", "(t2)?STRH_(preidx|PRE)",
  "t2STR(B?)_(PRE|preidx)", "t2STRD_PRE")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 629-646
```tablegen
// Store, register pre-indexed:
// 1(1) "S, I0/I1" for plus reg
// 3(2) "I0/I1, S" for minus reg
// 1(2) "S, M" for scaled plus lsl2
// 3(2) "I0/I1, S" for other scaled
def A57WriteStrAmLDSTSOPre : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledNotOptimalPredX2, [A57Write_3cyc_1I_1S]>,
  SchedVar<IsLdstsoMinusRegPredX2,         [A57Write_3cyc_1I_1S]>,
  SchedVar<IsLdstsoScaledPredX2,           [A57Write_1cyc_1S_1M]>,
  SchedVar<NoSchedPred,                    [A57Write_1cyc_1S_1I]>
]>;
def A57WriteStrAmLDSTSOPreWrBack : SchedWriteVariant<[
  SchedVar<IsLdstsoScaledPredX2,           [A57WrBackTwo]>,
  SchedVar<IsLdstsoMinusRegPredX2,         [A57WrBackTwo]>,
  SchedVar<NoSchedPred,                    [A57WrBackOne]>
]>;
def : InstRW<[A57WriteStrAmLDSTSOPreWrBack, A57WriteStrAmLDSTSOPre],
  (instregex "STR_PRE_REG", "STRB_PRE_REG")>;
```
- EN: Defines TableGen record `A57WriteStrAmLDSTSOPre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteStrAmLDSTSOPre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 648-660
```tablegen
// pre-indexed STRH/STRD (STRH_PRE, STRD_PRE)
// 1(1) "S, I0/I1" for imm or reg plus
// 3(2) "I0/I1, S" for reg minus
def A57WriteStrAm3PreX2 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPredX2, [A57Write_3cyc_1I_1S]>,
  SchedVar<NoSchedPred,             [A57Write_1cyc_1S_1I]>
]>;
def A57WriteStrAm3PreWrBackX2 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPredX2, [A57WrBackTwo]>,
  SchedVar<NoSchedPred,             [A57WrBackOne]>
]>;
def : InstRW<[A57WriteStrAm3PreWrBackX2, A57WriteStrAm3PreX2],
  (instregex "STRH_PRE")>;
```
- EN: Defines TableGen record `A57WriteStrAm3PreX2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteStrAm3PreX2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 662-671
```tablegen
def A57WriteStrAm3PreX3 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPredX3, [A57Write_3cyc_1I_1S]>,
  SchedVar<NoSchedPred,             [A57Write_1cyc_1S_1I]>
]>;
def A57WriteStrAm3PreWrBackX3 : SchedWriteVariant<[
  SchedVar<IsLdrAm3NegRegOffPredX3, [A57WrBackTwo]>,
  SchedVar<NoSchedPred,             [A57WrBackOne]>
]>;
def : InstRW<[A57WriteStrAm3PreWrBackX3, A57WriteStrAm3PreX3],
  (instregex "STRD_PRE")>;
```
- EN: Defines TableGen record `A57WriteStrAm3PreX3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteStrAm3PreX3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 673-674
```tablegen
def : InstRW<[A57WrBackOne, A57Write_1cyc_1S_1I], (instregex "STR(T?)_POST_IMM",
  "STRB(T?)_POST_IMM", "t2STR(B?)_POST")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 676-678
```tablegen
// 1(2) "S, M" for STR/STRB register post-indexed (both scaled or not)
def : InstRW<[A57WrBackTwo, A57Write_1cyc_1S_1M], (instregex "STR(T?)_POST_REG",
  "STRB(T?)_POST_REG", "STR(B?)T_POST$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 680-683
```tablegen
// post-indexed STRH/STRD(STRH_POST, STRD_POST), STRHTi, STRHTr
// 1(1) "S, I0/I1" both for reg or imm
def : InstRW<[A57WrBackOne, A57Write_1cyc_1S_1I],
  (instregex "(t2)?STR(H|D)_POST", "STRHT(i|r)", "t2STRHT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 685-702
```tablegen
// --- Store multiple instructions ---
// TODO: no writeback latency defined in documentation
def A57WriteSTM : SchedWriteVariant<[
    SchedVar<A57LMAddrPred1, [A57Write_1cyc_1S]>,
    SchedVar<A57LMAddrPred2, [A57Write_2cyc_1S]>,
    SchedVar<A57LMAddrPred3, [A57Write_3cyc_1S]>,
    SchedVar<A57LMAddrPred4, [A57Write_4cyc_1S]>,
    SchedVar<A57LMAddrPred5, [A57Write_5cyc_1S]>,
    SchedVar<A57LMAddrPred6, [A57Write_6cyc_1S]>,
    SchedVar<A57LMAddrPred7, [A57Write_7cyc_1S]>,
    SchedVar<A57LMAddrPred8, [A57Write_8cyc_1S]>,
    SchedVar<NoSchedPred,    [A57Write_2cyc_1S]>
]>;
def A57WriteSTM_Upd : SchedWriteVariant<[
    SchedVar<A57LMAddrPred1, [A57Write_1cyc_1S_1I]>,
    SchedVar<A57LMAddrPred2, [A57Write_2cyc_1S_1I]>,
    SchedVar<A57LMAddrPred3, [A57Write_3cyc_1S_1I]>,
    SchedVar<A57LMAddrPred4, [A57Write_4cyc_1S_1I]>,
```
- EN: Defines TableGen record `A57WriteSTM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteSTM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 703-708
```tablegen
    SchedVar<A57LMAddrPred5, [A57Write_5cyc_1S_1I]>,
    SchedVar<A57LMAddrPred6, [A57Write_6cyc_1S_1I]>,
    SchedVar<A57LMAddrPred7, [A57Write_7cyc_1S_1I]>,
    SchedVar<A57LMAddrPred8, [A57Write_8cyc_1S_1I]>,
    SchedVar<NoSchedPred,    [A57Write_2cyc_1S_1I]>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 710-712
```tablegen
def : InstRW<[A57WriteSTM], (instregex "(t2|sys|t)?STM(IA|DA|DB|IB)$")>;
def : InstRW<[A57WrBackOne, A57WriteSTM_Upd],
  (instregex "(t2|sys|t)?STM(IA_UPD|DA_UPD|DB_UPD|IB_UPD)", "tPUSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 714-714
```tablegen
def : InstRW<[A57Write_5cyc_1S], (instregex "VLSTM")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 716-718
```tablegen
// --- 3.10 FP Data Processing Instructions ---
def : SchedAlias<WriteFPALU32, A57Write_5cyc_1V>;
def : SchedAlias<WriteFPALU64, A57Write_5cyc_1V>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 720-720
```tablegen
def : InstRW<[A57Write_3cyc_1V], (instregex "VABS(S|D|H)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 722-728
```tablegen
// fp compare - 3cyc F1 for unconditional, 6cyc "F0/F1, F1" for conditional
def A57WriteVcmp : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57Write_6cyc_1V_1X]>,
  SchedVar<NoSchedPred,      [A57Write_3cyc_1X]>
]>;
def : InstRW<[A57WriteVcmp],
  (instregex "VCMP(D|S|H|ZD|ZS|ZH)$", "VCMPE(D|S|H|ZD|ZS|ZH)")>;
```
- EN: Defines TableGen record `A57WriteVcmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVcmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 730-734
```tablegen
// fp convert
def : InstRW<[A57Write_5cyc_1V], (instregex
  "VCVT(A|N|P|M)(SH|UH|SS|US|SD|UD)", "VCVT(BDH|THD|TDH)")>;
def : InstRW<[A57Write_5cyc_1V], (instregex "VTOSLS", "VTOUHS", "VTOULS")>;
def : SchedAlias<WriteFPCVT, A57Write_5cyc_1V>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 736-736
```tablegen
def : InstRW<[A57Write_5cyc_1V], (instregex "VJCVT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 738-739
```tablegen
// FP round to integral
def : InstRW<[A57Write_5cyc_1V], (instregex "VRINT(A|N|P|M|Z|R|X)(H|S|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 741-745
```tablegen
// FP divide, FP square root
def : SchedAlias<WriteFPDIV32, A57Write_17cyc_1W>;
def : SchedAlias<WriteFPDIV64, A57Write_32cyc_1W>;
def : SchedAlias<WriteFPSQRT32, A57Write_17cyc_1W>;
def : SchedAlias<WriteFPSQRT64, A57Write_32cyc_1W>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 747-747
```tablegen
def : InstRW<[A57Write_17cyc_1W], (instregex "VSQRTH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 749-750
```tablegen
// FP max/min
def : InstRW<[A57Write_5cyc_1V], (instregex "VMAX", "VMIN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 752-757
```tablegen
// FP multiply-accumulate pipelines support late forwarding of the result
// from FP multiply μops to the accumulate operands of an
// FP multiply-accumulate μop. The latter can potentially be issued 1 cycle
// after the FP multiply μop has been issued
// FP multiply, FZ
def A57WriteVMUL : SchedWriteRes<[A57UnitV]> { let Latency = 5; }
```
- EN: Defines TableGen record `A57WriteVMUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 759-761
```tablegen
def : SchedAlias<WriteFPMUL32, A57WriteVMUL>;
def : SchedAlias<WriteFPMUL64, A57WriteVMUL>;
def : ReadAdvance<ReadFPMUL, 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 763-765
```tablegen
// FP multiply accumulate, FZ: 9cyc "F0/F1" or 4 cyc for sequenced accumulate
// VFMA, VFMS, VFNMA, VFNMS, VMLA, VMLS, VNMLA, VNMLS
def A57WriteVFMA : SchedWriteRes<[A57UnitV]> { let Latency = 9;  }
```
- EN: Defines TableGen record `A57WriteVFMA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVFMA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 767-776
```tablegen
// VFMA takes 9 cyc for common case and 4 cyc for VFMA->VFMA chain (5 read adv.)
// VMUL takes 5 cyc for common case and 1 cyc for VMUL->VFMA chain (4 read adv.)
// Currently, there is no way to define different read advances for VFMA operand
// from VFMA or from VMUL, so there will be 5 read advance.
// Zero latency (instead of one) for VMUL->VFMA shouldn't break something.
// The same situation with ASIMD VMUL/VFMA instructions
// def A57ReadVFMA : SchedRead;
// def : ReadAdvance<A57ReadVFMA, 5, [A57WriteVFMA]>;
// def : ReadAdvance<A57ReadVFMA, 4, [A57WriteVMUL]>;
def A57ReadVFMA5 : SchedReadAdvance<5, [A57WriteVFMA, A57WriteVMUL]>;
```
- EN: Defines TableGen record `A57ReadVFMA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57ReadVFMA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 778-780
```tablegen
def : SchedAlias<WriteFPMAC32, A57WriteVFMA>;
def : SchedAlias<WriteFPMAC64, A57WriteVFMA>;
def : SchedAlias<ReadFPMAC, A57ReadVFMA5>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 782-784
```tablegen
// VMLAH/VMLSH are not binded to scheduling classes by default, so here custom:
def : InstRW<[A57WriteVFMA, A57ReadVFMA5, ReadFPMUL, ReadFPMUL],
  (instregex "VMLAH", "VMLSH", "VNMLAH", "VNMLSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 786-787
```tablegen
def : InstRW<[A57WriteVMUL],
  (instregex "VUDOTD", "VSDOTD", "VUDOTQ", "VSDOTQ")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 789-790
```tablegen
def : InstRW<[A57Write_3cyc_1V], (instregex "VNEG")>;
def : InstRW<[A57Write_3cyc_1V], (instregex "VSEL")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 792-795
```tablegen
// --- 3.11 FP Miscellaneous Instructions ---
// VMOV: 3cyc "F0/F1" for imm/reg
def : InstRW<[A57Write_3cyc_1V], (instregex "FCONST(D|S|H)")>;
def : InstRW<[A57Write_3cyc_1V], (instregex "VMOV(D|S|H)(cc)?$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 797-797
```tablegen
def : InstRW<[A57Write_3cyc_1V], (instregex "VINSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 799-803
```tablegen
// 5cyc L for FP transfer, vfp to core reg,
// 5cyc L for FP transfer, core reg to vfp
def : SchedAlias<WriteFPMOV, A57Write_5cyc_1L>;
// VMOVRRS/VMOVRRD in common code declared with one WriteFPMOV (instead of 2).
def : InstRW<[A57Write_5cyc_1L, A57Write_5cyc_1L], (instregex "VMOV(RRS|RRD)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 805-806
```tablegen
// 8cyc "L,F0/F1" for FP transfer, core reg to upper or lower half of vfp D-reg
def : InstRW<[A57Write_8cyc_1L_1I], (instregex "VMOVDRR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 808-809
```tablegen
// --- 3.12 FP Load Instructions ---
def : InstRW<[A57Write_5cyc_1L], (instregex "VLDR(D|S|H)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 811-811
```tablegen
def : InstRW<[A57Write_5cyc_1L], (instregex "VLDMQIA$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 813-813
```tablegen
// FP load multiple (VLDM)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 815-832
```tablegen
def A57VLDMOpsListUncond : A57WriteLMOpsListType<
               [A57Write_5cyc_1L, A57Write_5cyc_1L,
                A57Write_6cyc_1L, A57Write_6cyc_1L,
                A57Write_7cyc_1L, A57Write_7cyc_1L,
                A57Write_8cyc_1L, A57Write_8cyc_1L,
                A57Write_9cyc_1L, A57Write_9cyc_1L,
                A57Write_10cyc_1L, A57Write_10cyc_1L,
                A57Write_11cyc_1L, A57Write_11cyc_1L,
                A57Write_12cyc_1L, A57Write_12cyc_1L]>;
def A57WriteVLDMuncond : SchedWriteVariant<[
  SchedVar<A57LMAddrPred1,  A57VLDMOpsListUncond.Writes[0-1]>,
  SchedVar<A57LMAddrPred2,  A57VLDMOpsListUncond.Writes[0-3]>,
  SchedVar<A57LMAddrPred3,  A57VLDMOpsListUncond.Writes[0-5]>,
  SchedVar<A57LMAddrPred4,  A57VLDMOpsListUncond.Writes[0-7]>,
  SchedVar<A57LMAddrPred5,  A57VLDMOpsListUncond.Writes[0-9]>,
  SchedVar<A57LMAddrPred6,  A57VLDMOpsListUncond.Writes[0-11]>,
  SchedVar<A57LMAddrPred7,  A57VLDMOpsListUncond.Writes[0-13]>,
  SchedVar<NoSchedPred,     A57VLDMOpsListUncond.Writes[0-15]>
```
- EN: Defines TableGen record `A57VLDMOpsListUncond` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57VLDMOpsListUncond`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 833-833
```tablegen
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 835-852
```tablegen
def A57VLDMOpsListCond : A57WriteLMOpsListType<
               [A57Write_5cyc_1L, A57Write_6cyc_1L,
                A57Write_7cyc_1L, A57Write_8cyc_1L,
                A57Write_9cyc_1L, A57Write_10cyc_1L,
                A57Write_11cyc_1L, A57Write_12cyc_1L,
                A57Write_13cyc_1L, A57Write_14cyc_1L,
                A57Write_15cyc_1L, A57Write_16cyc_1L,
                A57Write_17cyc_1L, A57Write_18cyc_1L,
                A57Write_19cyc_1L, A57Write_20cyc_1L]>;
def A57WriteVLDMcond : SchedWriteVariant<[
  SchedVar<A57LMAddrPred1,  A57VLDMOpsListCond.Writes[0-1]>,
  SchedVar<A57LMAddrPred2,  A57VLDMOpsListCond.Writes[0-3]>,
  SchedVar<A57LMAddrPred3,  A57VLDMOpsListCond.Writes[0-5]>,
  SchedVar<A57LMAddrPred4,  A57VLDMOpsListCond.Writes[0-7]>,
  SchedVar<A57LMAddrPred5,  A57VLDMOpsListCond.Writes[0-9]>,
  SchedVar<A57LMAddrPred6,  A57VLDMOpsListCond.Writes[0-11]>,
  SchedVar<A57LMAddrPred7,  A57VLDMOpsListCond.Writes[0-13]>,
  SchedVar<NoSchedPred,     A57VLDMOpsListCond.Writes[0-15]>
```
- EN: Defines TableGen record `A57VLDMOpsListCond` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57VLDMOpsListCond`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 853-853
```tablegen
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 855-858
```tablegen
def A57WriteVLDM : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57WriteVLDMcond]>,
  SchedVar<NoSchedPred,      [A57WriteVLDMuncond]>
]> { let Variadic=1; }
```
- EN: Defines TableGen record `A57WriteVLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 860-860
```tablegen
def : InstRW<[A57WriteVLDM], (instregex "VLDM(DIA|SIA)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 862-879
```tablegen
def A57VLDMOpsListUncond_Upd : A57WriteLMOpsListType<
               [A57Write_5cyc_1L_1I, A57Write_5cyc_1L_1I,
                A57Write_6cyc_1L_1I, A57Write_6cyc_1L_1I,
                A57Write_7cyc_1L_1I, A57Write_7cyc_1L_1I,
                A57Write_8cyc_1L_1I, A57Write_8cyc_1L_1I,
                A57Write_9cyc_1L_1I, A57Write_9cyc_1L_1I,
                A57Write_10cyc_1L_1I, A57Write_10cyc_1L_1I,
                A57Write_11cyc_1L_1I, A57Write_11cyc_1L_1I,
                A57Write_12cyc_1L_1I, A57Write_12cyc_1L_1I]>;
def A57WriteVLDMuncond_UPD : SchedWriteVariant<[
  SchedVar<A57LMAddrPred1,  A57VLDMOpsListUncond_Upd.Writes[0-1]>,
  SchedVar<A57LMAddrPred2,  A57VLDMOpsListUncond_Upd.Writes[0-3]>,
  SchedVar<A57LMAddrPred3,  A57VLDMOpsListUncond_Upd.Writes[0-5]>,
  SchedVar<A57LMAddrPred4,  A57VLDMOpsListUncond_Upd.Writes[0-7]>,
  SchedVar<A57LMAddrPred5,  A57VLDMOpsListUncond_Upd.Writes[0-9]>,
  SchedVar<A57LMAddrPred6,  A57VLDMOpsListUncond_Upd.Writes[0-11]>,
  SchedVar<A57LMAddrPred7,  A57VLDMOpsListUncond_Upd.Writes[0-13]>,
  SchedVar<NoSchedPred,     A57VLDMOpsListUncond_Upd.Writes[0-15]>
```
- EN: Defines TableGen record `A57VLDMOpsListUncond_Upd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57VLDMOpsListUncond_Upd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 880-880
```tablegen
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 882-899
```tablegen
def A57VLDMOpsListCond_Upd : A57WriteLMOpsListType<
               [A57Write_5cyc_1L_1I, A57Write_6cyc_1L_1I,
                A57Write_7cyc_1L_1I, A57Write_8cyc_1L_1I,
                A57Write_9cyc_1L_1I, A57Write_10cyc_1L_1I,
                A57Write_11cyc_1L_1I, A57Write_12cyc_1L_1I,
                A57Write_13cyc_1L_1I, A57Write_14cyc_1L_1I,
                A57Write_15cyc_1L_1I, A57Write_16cyc_1L_1I,
                A57Write_17cyc_1L_1I, A57Write_18cyc_1L_1I,
                A57Write_19cyc_1L_1I, A57Write_20cyc_1L_1I]>;
def A57WriteVLDMcond_UPD : SchedWriteVariant<[
  SchedVar<A57LMAddrPred1,  A57VLDMOpsListCond_Upd.Writes[0-1]>,
  SchedVar<A57LMAddrPred2,  A57VLDMOpsListCond_Upd.Writes[0-3]>,
  SchedVar<A57LMAddrPred3,  A57VLDMOpsListCond_Upd.Writes[0-5]>,
  SchedVar<A57LMAddrPred4,  A57VLDMOpsListCond_Upd.Writes[0-7]>,
  SchedVar<A57LMAddrPred5,  A57VLDMOpsListCond_Upd.Writes[0-9]>,
  SchedVar<A57LMAddrPred6,  A57VLDMOpsListCond_Upd.Writes[0-11]>,
  SchedVar<A57LMAddrPred7,  A57VLDMOpsListCond_Upd.Writes[0-13]>,
  SchedVar<NoSchedPred,     A57VLDMOpsListCond_Upd.Writes[0-15]>
```
- EN: Defines TableGen record `A57VLDMOpsListCond_Upd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57VLDMOpsListCond_Upd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 900-900
```tablegen
]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 902-905
```tablegen
def A57WriteVLDM_UPD : SchedWriteVariant<[
  SchedVar<IsPredicatedPred, [A57WriteVLDMcond_UPD]>,
  SchedVar<NoSchedPred,      [A57WriteVLDMuncond_UPD]>
]> { let Variadic=1; }
```
- EN: Defines TableGen record `A57WriteVLDM_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVLDM_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 907-908
```tablegen
def : InstRW<[A57WrBackOne, A57WriteVLDM_UPD],
  (instregex "VLDM(DIA_UPD|DDB_UPD|SIA_UPD|SDB_UPD)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 910-911
```tablegen
// --- 3.13 FP Store Instructions ---
def : InstRW<[A57Write_1cyc_1S], (instregex "VSTR(D|S|H)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 913-913
```tablegen
def : InstRW<[A57Write_2cyc_1S], (instregex "VSTMQIA$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 915-932
```tablegen
def A57WriteVSTMs : SchedWriteVariant<[
    SchedVar<A57LMAddrPred1, [A57Write_1cyc_1S]>,
    SchedVar<A57LMAddrPred2, [A57Write_2cyc_1S]>,
    SchedVar<A57LMAddrPred3, [A57Write_3cyc_1S]>,
    SchedVar<A57LMAddrPred4, [A57Write_4cyc_1S]>,
    SchedVar<A57LMAddrPred5, [A57Write_5cyc_1S]>,
    SchedVar<A57LMAddrPred6, [A57Write_6cyc_1S]>,
    SchedVar<A57LMAddrPred7, [A57Write_7cyc_1S]>,
    SchedVar<A57LMAddrPred8, [A57Write_8cyc_1S]>,
    SchedVar<NoSchedPred,    [A57Write_2cyc_1S]>
]>;
def A57WriteVSTMd : SchedWriteVariant<[
    SchedVar<A57LMAddrPred1, [A57Write_2cyc_1S]>,
    SchedVar<A57LMAddrPred2, [A57Write_4cyc_1S]>,
    SchedVar<A57LMAddrPred3, [A57Write_6cyc_1S]>,
    SchedVar<A57LMAddrPred4, [A57Write_8cyc_1S]>,
    SchedVar<A57LMAddrPred5, [A57Write_10cyc_1S]>,
    SchedVar<A57LMAddrPred6, [A57Write_12cyc_1S]>,
```
- EN: Defines TableGen record `A57WriteVSTMs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVSTMs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 933-950
```tablegen
    SchedVar<A57LMAddrPred7, [A57Write_14cyc_1S]>,
    SchedVar<A57LMAddrPred8, [A57Write_16cyc_1S]>,
    SchedVar<NoSchedPred,    [A57Write_4cyc_1S]>
]>;
def A57WriteVSTMs_Upd : SchedWriteVariant<[
    SchedVar<A57LMAddrPred1, [A57Write_1cyc_1S_1I]>,
    SchedVar<A57LMAddrPred2, [A57Write_2cyc_1S_1I]>,
    SchedVar<A57LMAddrPred3, [A57Write_3cyc_1S_1I]>,
    SchedVar<A57LMAddrPred4, [A57Write_4cyc_1S_1I]>,
    SchedVar<A57LMAddrPred5, [A57Write_5cyc_1S_1I]>,
    SchedVar<A57LMAddrPred6, [A57Write_6cyc_1S_1I]>,
    SchedVar<A57LMAddrPred7, [A57Write_7cyc_1S_1I]>,
    SchedVar<A57LMAddrPred8, [A57Write_8cyc_1S_1I]>,
    SchedVar<NoSchedPred,    [A57Write_2cyc_1S_1I]>
]>;
def A57WriteVSTMd_Upd : SchedWriteVariant<[
    SchedVar<A57LMAddrPred1, [A57Write_2cyc_1S_1I]>,
    SchedVar<A57LMAddrPred2, [A57Write_4cyc_1S_1I]>,
```
- EN: Defines TableGen record `A57WriteVSTMs_Upd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVSTMs_Upd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 951-958
```tablegen
    SchedVar<A57LMAddrPred3, [A57Write_6cyc_1S_1I]>,
    SchedVar<A57LMAddrPred4, [A57Write_8cyc_1S_1I]>,
    SchedVar<A57LMAddrPred5, [A57Write_10cyc_1S_1I]>,
    SchedVar<A57LMAddrPred6, [A57Write_12cyc_1S_1I]>,
    SchedVar<A57LMAddrPred7, [A57Write_14cyc_1S_1I]>,
    SchedVar<A57LMAddrPred8, [A57Write_16cyc_1S_1I]>,
    SchedVar<NoSchedPred,    [A57Write_2cyc_1S_1I]>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 960-965
```tablegen
def : InstRW<[A57WriteVSTMs], (instregex "VSTMSIA$")>;
def : InstRW<[A57WriteVSTMd], (instregex "VSTMDIA$")>;
def : InstRW<[A57WrBackOne, A57WriteVSTMs_Upd],
  (instregex "VSTM(SIA_UPD|SDB_UPD)")>;
def : InstRW<[A57WrBackOne, A57WriteVSTMd_Upd],
  (instregex "VSTM(DIA_UPD|DDB_UPD)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 967-967
```tablegen
// --- 3.14 ASIMD Integer Instructions ---
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 969-970
```tablegen
// ASIMD absolute diff, 3cyc F0/F1 for integer VABD
def : InstRW<[A57Write_3cyc_1V], (instregex "VABD(s|u)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 972-980
```tablegen
// ASIMD absolute diff accum: 4(1) F1 for D-form, 5(2) F1 for Q-form
def A57WriteVABAD : SchedWriteRes<[A57UnitX]> { let Latency = 4; }
def A57ReadVABAD  : SchedReadAdvance<3, [A57WriteVABAD]>;
def : InstRW<[A57WriteVABAD, A57ReadVABAD],
  (instregex "VABA(s|u)(v8i8|v4i16|v2i32)")>;
def A57WriteVABAQ : SchedWriteRes<[A57UnitX]> { let Latency = 5; }
def A57ReadVABAQ  : SchedReadAdvance<3, [A57WriteVABAQ]>;
def : InstRW<[A57WriteVABAQ, A57ReadVABAQ],
  (instregex "VABA(s|u)(v16i8|v8i16|v4i32)")>;
```
- EN: Defines TableGen record `A57WriteVABAD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVABAD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 982-985
```tablegen
// ASIMD absolute diff accum long: 4(1) F1 for VABAL
def A57WriteVABAL : SchedWriteRes<[A57UnitX]> { let Latency = 4; }
def A57ReadVABAL  : SchedReadAdvance<3, [A57WriteVABAL]>;
def : InstRW<[A57WriteVABAL, A57ReadVABAL], (instregex "VABAL(s|u)")>;
```
- EN: Defines TableGen record `A57WriteVABAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVABAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 987-988
```tablegen
// ASIMD absolute diff long: 3cyc F0/F1 for VABDL
def : InstRW<[A57Write_3cyc_1V], (instregex "VABDL(s|u)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 990-993
```tablegen
// ASIMD arith, basic
def : InstRW<[A57Write_3cyc_1V], (instregex "VADDv", "VADDL", "VADDW",
  "VNEG(s8d|s16d|s32d|s8q|s16q|s32q|d|q)",
  "VPADDi", "VPADDL", "VSUBv", "VSUBL", "VSUBW")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 995-998
```tablegen
// ASIMD arith, complex
def : InstRW<[A57Write_3cyc_1V], (instregex "VABS", "VADDHN", "VHADD", "VHSUB",
  "VQABS", "VQADD", "VQNEG", "VQSUB",
  "VRADDHN", "VRHADD", "VRSUBHN", "VSUBHN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1000-1002
```tablegen
// ASIMD compare
def : InstRW<[A57Write_3cyc_1V],
  (instregex "VCEQ", "VCGE", "VCGT", "VCLE", "VTST", "VCLT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1004-1006
```tablegen
// ASIMD logical
def : InstRW<[A57Write_3cyc_1V],
  (instregex "VAND", "VBIC", "VMVN", "VORR", "VORN", "VEOR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1008-1010
```tablegen
// ASIMD max/min
def : InstRW<[A57Write_3cyc_1V],
  (instregex "(VMAX|VMIN)(s|u)", "(VPMAX|VPMIN)(s8|s16|s32|u8|u16|u32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1012-1020
```tablegen
// ASIMD multiply, D-form: 5cyc F0 for r0px, 4cyc F0 for r1p0 and later
// Cortex-A57 r1p0 and later reduce the latency of ASIMD multiply
// and multiply-with-accumulate instructions relative to r0pX.
def A57WriteVMULD_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_4cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_5cyc_1W]>]>;
def : InstRW<[A57WriteVMULD_VecInt], (instregex
  "VMUL(v8i8|v4i16|v2i32|pd)", "VMULsl(v4i16|v2i32)",
  "VQDMULH(sl)?(v4i16|v2i32)", "VQRDMULH(sl)?(v4i16|v2i32)")>;
```
- EN: Defines TableGen record `A57WriteVMULD_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMULD_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1022-1028
```tablegen
// ASIMD multiply, Q-form: 6cyc F0 for r0px, 5cyc F0 for r1p0 and later
def A57WriteVMULQ_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_5cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_6cyc_1W]>]>;
def : InstRW<[A57WriteVMULQ_VecInt], (instregex
  "VMUL(v16i8|v8i16|v4i32|pq)", "VMULsl(v8i16|v4i32)",
  "VQDMULH(sl)?(v8i16|v4i32)", "VQRDMULH(sl)?(v8i16|v4i32)")>;
```
- EN: Defines TableGen record `A57WriteVMULQ_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMULQ_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1030-1041
```tablegen
// ASIMD multiply accumulate, D-form
// 5cyc F0 for r0px, 4cyc F0 for r1p0 and later, 1cyc for accumulate sequence
// (4 or 3 ReadAdvance)
def A57WriteVMLAD_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_4cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_5cyc_1W]>]>;
def A57ReadVMLAD_VecInt : SchedReadVariant<[
  SchedVar<IsR1P0AndLaterPred, [SchedReadAdvance<3, [A57WriteVMLAD_VecInt]>]>,
  SchedVar<NoSchedPred,        [SchedReadAdvance<4, [A57WriteVMLAD_VecInt]>]>
]>;
def : InstRW<[A57WriteVMLAD_VecInt, A57ReadVMLAD_VecInt],
  (instregex "VMLA(sl)?(v8i8|v4i16|v2i32)", "VMLS(sl)?(v8i8|v4i16|v2i32)")>;
```
- EN: Defines TableGen record `A57WriteVMLAD_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMLAD_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1043-1054
```tablegen
// ASIMD multiply accumulate, Q-form
// 6cyc F0 for r0px, 5cyc F0 for r1p0 and later, 2cyc for accumulate sequence
// (4 or 3 ReadAdvance)
def A57WriteVMLAQ_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_5cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_6cyc_1W]>]>;
def A57ReadVMLAQ_VecInt : SchedReadVariant<[
  SchedVar<IsR1P0AndLaterPred, [SchedReadAdvance<3, [A57WriteVMLAQ_VecInt]>]>,
  SchedVar<NoSchedPred,        [SchedReadAdvance<4, [A57WriteVMLAQ_VecInt]>]>
]>;
def : InstRW<[A57WriteVMLAQ_VecInt, A57ReadVMLAQ_VecInt],
  (instregex "VMLA(sl)?(v16i8|v8i16|v4i32)", "VMLS(sl)?(v16i8|v8i16|v4i32)")>;
```
- EN: Defines TableGen record `A57WriteVMLAQ_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMLAQ_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1056-1067
```tablegen
// ASIMD multiply accumulate long
// 5cyc F0 for r0px, 4cyc F0 for r1p0 and later, 1cyc for accumulate sequence
// (4 or 3 ReadAdvance)
def A57WriteVMLAL_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_4cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_5cyc_1W]>]>;
def A57ReadVMLAL_VecInt : SchedReadVariant<[
  SchedVar<IsR1P0AndLaterPred, [SchedReadAdvance<3, [A57WriteVMLAL_VecInt]>]>,
  SchedVar<NoSchedPred,        [SchedReadAdvance<4, [A57WriteVMLAL_VecInt]>]>
]>;
def : InstRW<[A57WriteVMLAL_VecInt, A57ReadVMLAL_VecInt],
  (instregex "VMLAL(s|u)", "VMLSL(s|u)")>;
```
- EN: Defines TableGen record `A57WriteVMLAL_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMLAL_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1069-1080
```tablegen
// ASIMD multiply accumulate saturating long
// 5cyc F0 for r0px, 4cyc F0 for r1p0 and later, 2cyc for accumulate sequence
// (3 or 2 ReadAdvance)
def A57WriteVQDMLAL_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_4cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_5cyc_1W]>]>;
def A57ReadVQDMLAL_VecInt : SchedReadVariant<[
  SchedVar<IsR1P0AndLaterPred, [SchedReadAdvance<2, [A57WriteVQDMLAL_VecInt]>]>,
  SchedVar<NoSchedPred,        [SchedReadAdvance<3, [A57WriteVQDMLAL_VecInt]>]>
]>;
def : InstRW<[A57WriteVQDMLAL_VecInt, A57ReadVQDMLAL_VecInt],
  (instregex "VQDMLAL", "VQDMLSL")>;
```
- EN: Defines TableGen record `A57WriteVQDMLAL_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVQDMLAL_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1082-1085
```tablegen
// Vector Saturating Rounding Doubling Multiply Accumulate/Subtract Long
// Scheduling info from VQDMLAL/VQDMLSL
def : InstRW<[A57WriteVQDMLAL_VecInt, A57ReadVQDMLAL_VecInt],
  (instregex "VQRDMLAH", "VQRDMLSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1087-1093
```tablegen
// ASIMD multiply long
// 5cyc F0 for r0px, 4cyc F0 for r1p0 and later
def A57WriteVMULL_VecInt : SchedWriteVariant<[
  SchedVar<IsR1P0AndLaterPred, [A57Write_4cyc_1W]>,
  SchedVar<NoSchedPred,        [A57Write_5cyc_1W]>]>;
def : InstRW<[A57WriteVMULL_VecInt],
  (instregex "VMULL(s|u|p8|sls|slu)", "VQDMULL")>;
```
- EN: Defines TableGen record `A57WriteVMULL_VecInt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMULL_VecInt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1095-1099
```tablegen
// ASIMD pairwise add and accumulate
// 4cyc F1, 1cyc for accumulate sequence (3cyc ReadAdvance)
def A57WriteVPADAL : SchedWriteRes<[A57UnitX]> { let Latency = 4; }
def A57ReadVPADAL  : SchedReadAdvance<3, [A57WriteVPADAL]>;
def : InstRW<[A57WriteVPADAL, A57ReadVPADAL], (instregex "VPADAL(s|u)")>;
```
- EN: Defines TableGen record `A57WriteVPADAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVPADAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1101-1105
```tablegen
// ASIMD shift accumulate
// 4cyc F1, 1cyc for accumulate sequence (3cyc ReadAdvance)
def A57WriteVSRA : SchedWriteRes<[A57UnitX]> { let Latency = 4;  }
def A57ReadVSRA  : SchedReadAdvance<3, [A57WriteVSRA]>;
def : InstRW<[A57WriteVSRA, A57ReadVSRA], (instregex "VSRA", "VRSRA")>;
```
- EN: Defines TableGen record `A57WriteVSRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVSRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1107-1109
```tablegen
// ASIMD shift by immed, basic
def : InstRW<[A57Write_3cyc_1X],
  (instregex "VMOVL", "VSHLi", "VSHLL", "VSHR(s|u)", "VSHRN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1111-1114
```tablegen
// ASIMD shift by immed, complex
def : InstRW<[A57Write_4cyc_1X], (instregex
  "VQRSHRN", "VQRSHRUN", "VQSHL(si|ui|su)", "VQSHRN", "VQSHRUN", "VRSHR(s|u)",
  "VRSHRN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1116-1118
```tablegen
// ASIMD shift by immed and insert, basic, D-form
def : InstRW<[A57Write_4cyc_1X], (instregex
  "VSLI(v8i8|v4i16|v2i32|v1i64)", "VSRI(v8i8|v4i16|v2i32|v1i64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1120-1122
```tablegen
// ASIMD shift by immed and insert, basic, Q-form
def : InstRW<[A57Write_5cyc_1X], (instregex
  "VSLI(v16i8|v8i16|v4i32|v2i64)", "VSRI(v16i8|v8i16|v4i32|v2i64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1124-1126
```tablegen
// ASIMD shift by register, basic, D-form
def : InstRW<[A57Write_3cyc_1X], (instregex
  "VSHL(s|u)(v8i8|v4i16|v2i32|v1i64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1128-1130
```tablegen
// ASIMD shift by register, basic, Q-form
def : InstRW<[A57Write_4cyc_1X], (instregex
  "VSHL(s|u)(v16i8|v8i16|v4i32|v2i64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1132-1136
```tablegen
// ASIMD shift by register, complex, D-form
// VQRSHL, VQSHL, VRSHL
def : InstRW<[A57Write_4cyc_1X], (instregex
  "VQRSHL(s|u)(v8i8|v4i16|v2i32|v1i64)", "VQSHL(s|u)(v8i8|v4i16|v2i32|v1i64)",
  "VRSHL(s|u)(v8i8|v4i16|v2i32|v1i64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1138-1141
```tablegen
// ASIMD shift by register, complex, Q-form
def : InstRW<[A57Write_5cyc_1X], (instregex
  "VQRSHL(s|u)(v16i8|v8i16|v4i32|v2i64)", "VQSHL(s|u)(v16i8|v8i16|v4i32|v2i64)",
  "VRSHL(s|u)(v16i8|v8i16|v4i32|v2i64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1143-1145
```tablegen
// --- 3.15 ASIMD Floating-Point Instructions ---
// ASIMD FP absolute value
def : InstRW<[A57Write_3cyc_1V], (instregex "VABS(fd|fq|hd|hq)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1147-1149
```tablegen
// ASIMD FP arith
def : InstRW<[A57Write_5cyc_1V], (instregex "VABD(fd|fq|hd|hq)",
  "VADD(fd|fq|hd|hq)", "VPADD(f|h)", "VSUB(fd|fq|hd|hq)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1151-1151
```tablegen
def : InstRW<[A57Write_5cyc_1V], (instregex "VCADD", "VCMLA")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1153-1155
```tablegen
// ASIMD FP compare
def : InstRW<[A57Write_5cyc_1V], (instregex "VAC(GE|GT|LE|LT)",
  "VC(EQ|GE|GT|LE)(fd|fq|hd|hq)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1157-1161
```tablegen
// ASIMD FP convert, integer
def : InstRW<[A57Write_5cyc_1V], (instregex
  "VCVT(f2sd|f2ud|s2fd|u2fd|f2sq|f2uq|s2fq|u2fq|f2xsd|f2xud|xs2fd|xu2fd)",
  "VCVT(f2xsq|f2xuq|xs2fq|xu2fq)",
  "VCVT(AN|MN|NN|PN)(SDf|SQf|UDf|UQf|SDh|SQh|UDh|UQh)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1163-1167
```tablegen
// ASIMD FP convert, half-precision: 8cyc F0/F1
def : InstRW<[A57Write_8cyc_1V], (instregex
  "VCVT(h2sd|h2ud|s2hd|u2hd|h2sq|h2uq|s2hq|u2hq|h2xsd|h2xud|xs2hd|xu2hd)",
  "VCVT(h2xsq|h2xuq|xs2hq|xu2hq)",
  "VCVT(f2h|h2f)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1169-1172
```tablegen
// ASIMD FP max/min
def : InstRW<[A57Write_5cyc_1V], (instregex
  "(VMAX|VMIN)(fd|fq|hd|hq)", "(VPMAX|VPMIN)(f|h)", "(NEON|VFP)_VMAXNM",
  "(NEON|VFP)_VMINNM")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1174-1176
```tablegen
// ASIMD FP multiply
def A57WriteVMUL_VecFP  : SchedWriteRes<[A57UnitV]> { let Latency = 5;  }
def : InstRW<[A57WriteVMUL_VecFP], (instregex "VMUL(sl)?(fd|fq|hd|hq)")>;
```
- EN: Defines TableGen record `A57WriteVMUL_VecFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMUL_VecFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1178-1183
```tablegen
// ASIMD FP multiply accumulate: 9cyc F0/F1, 4cyc for accumulate sequence
def A57WriteVMLA_VecFP  : SchedWriteRes<[A57UnitV]> { let Latency = 9;  }
def A57ReadVMLA_VecFP  :
  SchedReadAdvance<5, [A57WriteVMLA_VecFP, A57WriteVMUL_VecFP]>;
def : InstRW<[A57WriteVMLA_VecFP, A57ReadVMLA_VecFP],
  (instregex "(VMLA|VMLS)(sl)?(fd|fq|hd|hq)", "(VFMA|VFMS)(fd|fq|hd|hq)")>;
```
- EN: Defines TableGen record `A57WriteVMLA_VecFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57WriteVMLA_VecFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1185-1186
```tablegen
// ASIMD FP negate
def : InstRW<[A57Write_3cyc_1V], (instregex "VNEG(fd|f32q|hd|hq)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1188-1190
```tablegen
// ASIMD FP round to integral
def : InstRW<[A57Write_5cyc_1V], (instregex
  "VRINT(AN|MN|NN|PN|XN|ZN)(Df|Qf|Dh|Qh)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1192-1192
```tablegen
// --- 3.16 ASIMD Miscellaneous Instructions ---
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1194-1195
```tablegen
// ASIMD bitwise insert
def : InstRW<[A57Write_3cyc_1V], (instregex "VBIF", "VBIT", "VBSL", "VBSP")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1197-1198
```tablegen
// ASIMD count
def : InstRW<[A57Write_3cyc_1V], (instregex "VCLS", "VCLZ", "VCNT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1200-1201
```tablegen
// ASIMD duplicate, core reg: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V], (instregex "VDUP(8|16|32)(d|q)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1203-1204
```tablegen
// ASIMD duplicate, scalar: 3cyc "F0/F1"
def : InstRW<[A57Write_3cyc_1V], (instregex "VDUPLN(8|16|32)(d|q)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1206-1207
```tablegen
// ASIMD extract
def : InstRW<[A57Write_3cyc_1V], (instregex "VEXT(d|q)(8|16|32|64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1209-1212
```tablegen
// ASIMD move, immed
def : InstRW<[A57Write_3cyc_1V], (instregex
  "VMOV(v8i8|v16i8|v4i16|v8i16|v2i32|v4i32|v1i64|v2i64|v2f32|v4f32)",
  "VMOVD0", "VMOVQ0")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1214-1215
```tablegen
// ASIMD move, narrowing
def : InstRW<[A57Write_3cyc_1V], (instregex "VMOVN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1217-1218
```tablegen
// ASIMD move, saturating
def : InstRW<[A57Write_4cyc_1X], (instregex "VQMOVN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1220-1221
```tablegen
// ASIMD reciprocal estimate
def : InstRW<[A57Write_5cyc_1V], (instregex "VRECPE", "VRSQRTE")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1223-1224
```tablegen
// ASIMD reciprocal step, FZ
def : InstRW<[A57Write_9cyc_1V], (instregex "VRECPS", "VRSQRTS")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1226-1227
```tablegen
// ASIMD reverse, swap, table lookup (1-2 reg)
def : InstRW<[A57Write_3cyc_1V], (instregex "VREV", "VSWP", "VTB(L|X)(1|2)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1229-1230
```tablegen
// ASIMD table lookup (3-4 reg)
def : InstRW<[A57Write_6cyc_1V], (instregex "VTBL(3|4)", "VTBX(3|4)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1232-1233
```tablegen
// ASIMD transfer, scalar to core reg: 6cyc "L, I0/I1"
def : InstRW<[A57Write_6cyc_1L_1I], (instregex "VGETLN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1235-1236
```tablegen
// ASIMD transfer, core reg to scalar: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V], (instregex "VSETLN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1238-1239
```tablegen
// ASIMD transpose
def : InstRW<[A57Write_3cyc_1V, A57Write_3cyc_1V], (instregex "VTRN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1241-1243
```tablegen
// ASIMD unzip/zip, D-form
def : InstRW<[A57Write_3cyc_1V, A57Write_3cyc_1V],
  (instregex "VUZPd", "VZIPd")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1245-1247
```tablegen
// ASIMD unzip/zip, Q-form
def : InstRW<[A57Write_6cyc_1V, A57Write_6cyc_1V],
  (instregex "VUZPq", "VZIPq")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1249-1249
```tablegen
// --- 3.17 ASIMD Load Instructions ---
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1251-1259
```tablegen
// Overriden via InstRW for this processor.
def : WriteRes<WriteVLD1, []>;
def : WriteRes<WriteVLD2, []>;
def : WriteRes<WriteVLD3, []>;
def : WriteRes<WriteVLD4, []>;
def : WriteRes<WriteVST1, []>;
def : WriteRes<WriteVST2, []>;
def : WriteRes<WriteVST3, []>;
def : WriteRes<WriteVST4, []>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1261-1264
```tablegen
// 1-2 reg: 5cyc L, +I for writeback, 1 cyc wb latency
def : InstRW<[A57Write_5cyc_1L], (instregex "VLD1(d|q)(8|16|32|64)$")>;
def : InstRW<[A57Write_5cyc_1L_1I, A57WrBackOne],
  (instregex "VLD1(d|q)(8|16|32|64)wb")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1266-1268
```tablegen
// 3-4 reg: 6cyc L, +I for writeback, 1 cyc wb latency
def : InstRW<[A57Write_6cyc_1L],
  (instregex "VLD1(d|q)(8|16|32|64)(T|Q)$", "VLD1d64(T|Q)Pseudo")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1270-1271
```tablegen
def : InstRW<[A57Write_6cyc_1L_1I, A57WrBackOne],
  (instregex "VLD1(d|q)(8|16|32|64)(T|Q)wb")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1273-1277
```tablegen
// ASIMD load, 1 element, one lane and all lanes: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V], (instregex
  "VLD1(LN|DUP)(d|q)(8|16|32)$", "VLD1(LN|DUP)(d|q)(8|16|32)Pseudo$")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne], (instregex
  "VLD1(LN|DUP)(d|q)(8|16|32)(wb|_UPD)", "VLD1LNq(8|16|32)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1279-1283
```tablegen
// ASIMD load, 2 element, multiple, 2 reg: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V],
      (instregex "VLD2(d|q)(8|16|32)$", "VLD2q(8|16|32)Pseudo$")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD2(d|q)(8|16|32)wb", "VLD2q(8|16|32)PseudoWB")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1285-1288
```tablegen
// ASIMD load, 2 element, multiple, 4 reg: 9cyc "L, F0/F1"
def : InstRW<[A57Write_9cyc_1L_1V], (instregex "VLD2b(8|16|32)$")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD2b(8|16|32)wb")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1290-1300
```tablegen
// ASIMD load, 2 element, one lane and all lanes: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V],
      (instregex "VLD2(DUP|LN)(d|q)(8|16|32|8x2|16x2|32x2)$",
                 "VLD2LN(d|q)(8|16|32)Pseudo$")>;
// 2 results + wb result
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V, A57WrBackOne],
      (instregex "VLD2LN(d|q)(8|16|32)_UPD$")>;
// 1 result + wb result
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD2DUPd(8|16|32|8x2|16x2|32x2)wb",
                 "VLD2LN(d|q)(8|16|32)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1302-1315
```tablegen
// ASIMD load, 3 element, multiple, 3 reg: 9cyc "L, F0/F1"
// 3 results
def : InstRW<[A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V],
      (instregex "VLD3(d|q)(8|16|32)$")>;
// 1 result
def : InstRW<[A57Write_9cyc_1L_1V],
      (instregex "VLD3(d|q)(8|16|32)(oddP|P)seudo$")>;
// 3 results + wb
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57Write_9cyc_1L_1V_1I,
              A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3(d|q)(8|16|32)_UPD$")>;
// 1 result + wb
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3(d|q)(8|16|32)(oddP|P)seudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1317-1325
```tablegen
// ASIMD load, 3 element, one lane, size 32: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V],
      (instregex "VLD3LN(d|q)32$",
                 "VLD3LN(d|q)32Pseudo$")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V_1I,
              A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3LN(d|q)32_UPD")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3LN(d|q)32Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1327-1335
```tablegen
// ASIMD load, 3 element, one lane, size 8/16: 9cyc "L, F0/F1"
def : InstRW<[A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V],
      (instregex "VLD3LN(d|q)(8|16)$",
                 "VLD3LN(d|q)(8|16)Pseudo$")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57Write_9cyc_1L_1V_1I,
              A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3LN(d|q)(8|16)_UPD")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3LN(d|q)(8|16)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1337-1345
```tablegen
// ASIMD load, 3 element, all lanes: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V],
      (instregex "VLD3DUP(d|q)(8|16|32)$",
                 "VLD3DUP(d|q)(8|16|32)Pseudo$")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V_1I,
              A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3DUP(d|q)(8|16|32)_UPD")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD3DUP(d|q)(8|16|32)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1347-1357
```tablegen
// ASIMD load, 4 element, multiple, 4 reg: 9cyc "L, F0/F1"
def : InstRW<[A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V,
              A57Write_9cyc_1L_1V],
      (instregex "VLD4(d|q)(8|16|32)$")>;
def : InstRW<[A57Write_9cyc_1L_1V],
      (instregex "VLD4(d|q)(8|16|32)(oddP|P)seudo$")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57Write_9cyc_1L_1V_1I,
              A57Write_9cyc_1L_1V_1I, A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD4(d|q)(8|16|32)_UPD")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex  "VLD4(d|q)(8|16|32)(oddP|P)seudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1359-1369
```tablegen
// ASIMD load, 4 element, one lane, size 32: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V,
              A57Write_8cyc_1L_1V],
      (instregex "VLD4LN(d|q)32$",
                 "VLD4LN(d|q)32Pseudo$")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V_1I,
              A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V_1I,
              A57WrBackOne],
      (instregex "VLD4LN(d|q)32_UPD")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD4LN(d|q)32Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1371-1381
```tablegen
// ASIMD load, 4 element, one lane, size 8/16: 9cyc "L, F0/F1"
def : InstRW<[A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V, A57Write_9cyc_1L_1V,
              A57Write_9cyc_1L_1V],
      (instregex "VLD4LN(d|q)(8|16)$",
                 "VLD4LN(d|q)(8|16)Pseudo$")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57Write_9cyc_1L_1V_1I,
              A57Write_9cyc_1L_1V_1I, A57Write_9cyc_1L_1V_1I,
              A57WrBackOne],
      (instregex "VLD4LN(d|q)(8|16)_UPD")>;
def : InstRW<[A57Write_9cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD4LN(d|q)(8|16)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1383-1393
```tablegen
// ASIMD load, 4 element, all lanes: 8cyc "L, F0/F1"
def : InstRW<[A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V, A57Write_8cyc_1L_1V,
              A57Write_8cyc_1L_1V],
      (instregex "VLD4DUP(d|q)(8|16|32)$",
                 "VLD4DUP(d|q)(8|16|32)Pseudo$")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V_1I,
              A57Write_8cyc_1L_1V_1I, A57Write_8cyc_1L_1V_1I,
              A57WrBackOne],
      (instregex "VLD4DUP(d|q)(8|16|32)_UPD")>;
def : InstRW<[A57Write_8cyc_1L_1V_1I, A57WrBackOne],
      (instregex "VLD4DUP(d|q)(8|16|32)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1395-1395
```tablegen
// --- 3.18 ASIMD Store Instructions ---
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1397-1414
```tablegen
// ASIMD store, 1 element, multiple, 1 reg: 1cyc S
def : InstRW<[A57Write_1cyc_1S], (instregex "VST1d(8|16|32|64)$")>;
def : InstRW<[A57WrBackOne, A57Write_1cyc_1S_1I],
      (instregex "VST1d(8|16|32|64)wb")>;
// ASIMD store, 1 element, multiple, 2 reg: 2cyc S
def : InstRW<[A57Write_2cyc_1S], (instregex "VST1q(8|16|32|64)$")>;
def : InstRW<[A57WrBackOne, A57Write_2cyc_1S_1I],
      (instregex "VST1q(8|16|32|64)wb")>;
// ASIMD store, 1 element, multiple, 3 reg: 3cyc S
def : InstRW<[A57Write_3cyc_1S],
      (instregex "VST1d(8|16|32|64)T$", "VST1d64TPseudo$")>;
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1I],
      (instregex "VST1d(8|16|32|64)Twb", "VST1d64TPseudoWB")>;
// ASIMD store, 1 element, multiple, 4 reg: 4cyc S
def : InstRW<[A57Write_4cyc_1S],
      (instregex "VST1d(8|16|32|64)(Q|QPseudo)$")>;
def : InstRW<[A57WrBackOne, A57Write_4cyc_1S_1I],
      (instregex "VST1d(8|16|32|64)(Qwb|QPseudoWB)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1415-1432
```tablegen
// ASIMD store, 1 element, one lane: 3cyc "F0/F1, S"
def : InstRW<[A57Write_3cyc_1S_1V],
      (instregex "VST1LNd(8|16|32)$", "VST1LNq(8|16|32)Pseudo$")>;
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1V_1I],
      (instregex "VST1LNd(8|16|32)_UPD", "VST1LNq(8|16|32)Pseudo_UPD")>;
// ASIMD store, 2 element, multiple, 2 reg: 3cyc "F0/F1, S"
def : InstRW<[A57Write_3cyc_1S_1V],
      (instregex "VST2(d|b)(8|16|32)$")>;
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1V_1I],
      (instregex "VST2(b|d)(8|16|32)wb")>;
// ASIMD store, 2 element, multiple, 4 reg: 4cyc "F0/F1, S"
def : InstRW<[A57Write_4cyc_1S_1V],
      (instregex "VST2q(8|16|32)$", "VST2q(8|16|32)Pseudo$")>;
def : InstRW<[A57WrBackOne, A57Write_4cyc_1S_1V_1I],
      (instregex "VST2q(8|16|32)wb", "VST2q(8|16|32)PseudoWB")>;
// ASIMD store, 2 element, one lane: 3cyc "F0/F1, S"
def : InstRW<[A57Write_3cyc_1S_1V],
      (instregex "VST2LN(d|q)(8|16|32)$", "VST2LN(d|q)(8|16|32)Pseudo$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1433-1450
```tablegen
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1V_1I],
      (instregex "VST2LN(d|q)(8|16|32)_UPD",
                 "VST2LN(d|q)(8|16|32)Pseudo_UPD")>;
// ASIMD store, 3 element, multiple, 3 reg
def : InstRW<[A57Write_3cyc_1S_1V],
      (instregex "VST3(d|q)(8|16|32)$", "VST3(d|q)(8|16|32)(oddP|P)seudo$")>;
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1V_1I],
      (instregex "VST3(d|q)(8|16|32)_UPD",
                 "VST3(d|q)(8|16|32)(oddP|P)seudo_UPD$")>;
// ASIMD store, 3 element, one lane
def : InstRW<[A57Write_3cyc_1S_1V],
      (instregex "VST3LN(d|q)(8|16|32)$", "VST3LN(d|q)(8|16|32)Pseudo$")>;
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1V_1I],
      (instregex "VST3LN(d|q)(8|16|32)_UPD",
                 "VST3LN(d|q)(8|16|32)Pseudo_UPD")>;
// ASIMD store, 4 element, multiple, 4 reg
def : InstRW<[A57Write_4cyc_1S_1V],
      (instregex "VST4(d|q)(8|16|32)$", "VST4(d|q)(8|16|32)(oddP|P)seudo$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1451-1459
```tablegen
def : InstRW<[A57WrBackOne, A57Write_4cyc_1S_1V_1I],
      (instregex "VST4(d|q)(8|16|32)_UPD",
                 "VST4(d|q)(8|16|32)(oddP|P)seudo_UPD$")>;
// ASIMD store, 4 element, one lane
def : InstRW<[A57Write_3cyc_1S_1V],
      (instregex "VST4LN(d|q)(8|16|32)$", "VST4LN(d|q)(8|16|32)Pseudo$")>;
def : InstRW<[A57WrBackOne, A57Write_3cyc_1S_1V_1I],
      (instregex "VST4LN(d|q)(8|16|32)_UPD",
                 "VST4LN(d|q)(8|16|32)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1461-1476
```tablegen
// --- 3.19 Cryptography Extensions ---
// Crypto AES ops
// AESD, AESE, AESIMC, AESMC: 3cyc F0
def : InstRW<[A57Write_3cyc_1W], (instregex "^AES")>;
// Crypto polynomial (64x64) multiply long (VMULL.P64): 3cyc F0
def : InstRW<[A57Write_3cyc_1W], (instregex "^VMULLp64")>;
// Crypto SHA1 xor ops: 6cyc F0/F1
def : InstRW<[A57Write_6cyc_2V], (instregex "^SHA1SU0")>;
// Crypto SHA1 fast ops: 3cyc F0
def : InstRW<[A57Write_3cyc_1W], (instregex "^SHA1(H|SU1)")>;
// Crypto SHA1 slow ops: 6cyc F0
def : InstRW<[A57Write_6cyc_2W], (instregex "^SHA1[CMP]")>;
// Crypto SHA256 fast ops: 3cyc F0
def : InstRW<[A57Write_3cyc_1W], (instregex "^SHA256SU0")>;
// Crypto SHA256 slow ops: 6cyc F0
def : InstRW<[A57Write_6cyc_2W], (instregex "^SHA256(H|H2|SU1)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1478-1479
```tablegen
// --- 3.20 CRC ---
def : InstRW<[A57Write_3cyc_1W], (instregex "^(t2)?CRC32")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1481-1484
```tablegen
// -----------------------------------------------------------------------------
// Common definitions
def : WriteRes<WriteNoop, []> { let Latency = 0; let NumMicroOps = 0; }
def : SchedAlias<WriteALU, CheckBranchForm<0, A57BranchForm<A57Write_1cyc_1I>>>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1486-1489
```tablegen
def : SchedAlias<WriteBr, A57Write_1cyc_1B>;
def : SchedAlias<WriteBrL, A57Write_1cyc_1B_1I>;
def : SchedAlias<WriteBrTbl, A57Write_1cyc_1B_1I>;
def : SchedAlias<WritePreLd, A57Write_4cyc_1L>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1491-1493
```tablegen
def : SchedAlias<WriteLd, A57Write_4cyc_1L>;
def : SchedAlias<WriteST, A57Write_1cyc_1S>;
def : ReadAdvance<ReadALU, 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1495-1495
```tablegen
} // SchedModel = CortexA57Model
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `ARMScheduleA57WriteRes.td`.
  - CN: TableGen 包含项：`ARMScheduleA57WriteRes.td`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
