# MipsScheduleP5600.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsScheduleP5600.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MipsScheduleP5600` in LLVM TableGen DSL for the Mips backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsScheduleP5600`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//==- MipsScheduleP5600.td - P5600 Scheduling Definitions --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-13
```tablegen
def MipsP5600Model : SchedMachineModel {
  int IssueWidth = 2; // 2x dispatched per cycle
  int MicroOpBufferSize = 48; // min(48, 48, 64)
  int LoadLatency = 4;
  int MispredictPenalty = 8; // TODO: Estimated
```
- EN: Defines TableGen record `MipsP5600Model` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsP5600Model`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 15-16
```tablegen
  let CompleteModel = 1;
  let FullInstRWOverlapCheck = 1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 18-25
```tablegen
  list<Predicate> UnsupportedFeatures = [HasMips3, HasMips32r6, HasMips64,
                                         HasMips64r2, HasMips64r5, HasMips64r6,
                                         IsGP64bit, IsPTR64bit,
                                         InMicroMips, InMips16Mode,
                                         HasCnMips, HasCnMipsP,
                                         HasDSP, HasDSPR2, HasMips3D, HasMT,
                                         HasCRC];
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 27-27
```tablegen
let SchedModel = MipsP5600Model in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 29-30
```tablegen
// ALQ Pipelines
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 32-33
```tablegen
def P5600ALQ : ProcResource<1> { let BufferSize = 16; }
def P5600IssueALU : ProcResource<1> { let Super = P5600ALQ; }
```
- EN: Defines TableGen record `P5600ALQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600ALQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 35-36
```tablegen
// ALU Pipeline
// ------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 38-38
```tablegen
def P5600WriteALU : SchedWriteRes<[P5600IssueALU]>;
```
- EN: Defines TableGen record `P5600WriteALU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteALU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 40-42
```tablegen
// and, lui, nor, or, slti, sltiu, sub, subu, xor
def : InstRW<[P5600WriteALU], (instrs AND, LUi, NOR, OR, SLTi, SLTiu, SUB,
                               SUBu, XOR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 44-45
```tablegen
// AGQ Pipelines
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 47-50
```tablegen
def P5600AGQ : ProcResource<3> { let BufferSize = 16; }
def P5600IssueAL2 : ProcResource<1> { let Super = P5600AGQ; }
def P5600IssueCTISTD : ProcResource<1> { let Super = P5600AGQ; }
def P5600IssueLDST : ProcResource<1> { let Super = P5600AGQ; }
```
- EN: Defines TableGen record `P5600AGQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600AGQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 52-54
```tablegen
def P5600AL2Div : ProcResource<1>;
// Pseudo-resource used to block CTISTD when handling multi-pipeline splits.
def P5600CTISTD : ProcResource<1>;
```
- EN: Defines TableGen record `P5600AL2Div` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600AL2Div`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 56-57
```tablegen
// CTISTD Pipeline
// ---------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 59-62
```tablegen
def P5600WriteJump : SchedWriteRes<[P5600IssueCTISTD, P5600CTISTD]>;
def P5600WriteJumpAndLink : SchedWriteRes<[P5600IssueCTISTD, P5600CTISTD]> {
  let Latency = 2;
}
```
- EN: Defines TableGen record `P5600WriteJump` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteJump`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 64-66
```tablegen
def P5600Nop : SchedWriteRes<[P5600IssueCTISTD]> {
  let Latency = 0;
}
```
- EN: Defines TableGen record `P5600Nop` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600Nop`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 68-68
```tablegen
def : InstRW<[P5600Nop], (instrs SSNOP, NOP)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 70-81
```tablegen
// b, beq, beql, bg[et]z, bl[et]z, bne, bnel, j, syscall, jal, bltzal,
// jalr, jr.hb, jr
def : InstRW<[P5600WriteJump], (instrs B, BAL, BAL_BR, BEQ, BEQL, BGEZ, BGEZAL,
                                BGEZALL, BGEZL, BGTZ, BGTZL, BLEZ, BLEZL, BLTZ,
                                BLTZAL, BLTZALL, BLTZL, BNE, BNEL, BREAK,
                                DERET, ERET, ERet, ERETNC, J, JR, JR_HB,
                                PseudoIndirectBranch,
                                PseudoIndirectHazardBranch, PseudoReturn,
                                SDBBP, SYSCALL, RetRA, TAILCALL, TAILCALLREG,
                                TAILCALLREGHB, TEQ, TEQI, TGE, TGEI, TGEIU,
                                TGEU, TLT, TLTI, TLTU, TNE, TNEI, TRAP,
                                TTLTIU, WAIT, PAUSE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 83-84
```tablegen
def : InstRW<[P5600WriteJumpAndLink], (instrs JAL, JALR, JALRHBPseudo,
                                       JALRPseudo, JALR_HB)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 86-88
```tablegen
def : InstRW<[P5600WriteJumpAndLink], (instrs JALX)> {
  let Unsupported = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 90-90
```tablegen
def P5600COP0 : SchedWriteRes<[P5600IssueCTISTD, P5600CTISTD]>;
```
- EN: Defines TableGen record `P5600COP0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600COP0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 92-93
```tablegen
def : InstRW<[P5600COP0], (instrs TLBINV, TLBINVF, TLBP, TLBR, TLBWI, TLBWR,
                           MFC0, MTC0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 95-95
```tablegen
def P5600COP2 : SchedWriteRes<[P5600IssueCTISTD, P5600CTISTD]>;
```
- EN: Defines TableGen record `P5600COP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600COP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 97-99
```tablegen
def : InstRW<[P5600COP2], (instrs MFC2, MTC2)> {
  let Unsupported = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 101-104
```tablegen
// MIPS Virtualization ASE
// =======================
def : InstRW<[P5600COP0], (instrs HYPCALL, MFGC0, MFHGC0, MTGC0, MTHGC0,
                           TLBGINV, TLBGINVF, TLBGP, TLBGR, TLBGWI, TLBGWR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 106-107
```tablegen
// LDST Pipeline
// -------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 109-111
```tablegen
def P5600WriteLoad : SchedWriteRes<[P5600IssueLDST]> {
  let Latency = 4;
}
```
- EN: Defines TableGen record `P5600WriteLoad` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteLoad`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 113-115
```tablegen
def P5600WriteLoadShifted : SchedWriteRes<[P5600IssueLDST, P5600CTISTD]> {
  let Latency = 4;
}
```
- EN: Defines TableGen record `P5600WriteLoadShifted` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteLoadShifted`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 117-117
```tablegen
def P5600WriteCache : SchedWriteRes<[P5600IssueLDST]>;
```
- EN: Defines TableGen record `P5600WriteCache` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteCache`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 119-123
```tablegen
def P5600WriteStore : SchedWriteRes<[P5600IssueLDST, P5600CTISTD]> {
  // FIXME: This is a bit pessimistic. P5600CTISTD is only used during cycle 2
  //        not during 0, 1, and 2.
  let ReleaseAtCycles = [ 1, 3 ];
}
```
- EN: Defines TableGen record `P5600WriteStore` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteStore`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 125-127
```tablegen
def P5600WriteGPRFromBypass : SchedWriteRes<[P5600IssueLDST]> {
  let Latency = 2;
}
```
- EN: Defines TableGen record `P5600WriteGPRFromBypass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteGPRFromBypass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 129-132
```tablegen
def P5600WriteStoreFromOtherUnits : SchedWriteRes<[P5600IssueLDST]>;
def P5600WriteLoadToOtherUnits : SchedWriteRes<[P5600IssueLDST]> {
  let Latency = 0;
}
```
- EN: Defines TableGen record `P5600WriteStoreFromOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteStoreFromOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 134-137
```tablegen
// l[bhw], l[bh]u, ll
def : InstRW<[P5600WriteLoad], (instrs LB, LBu, LH, LHu, LW, LL, LWC2, LWC3,
                                LDC2, LDC3, LBE, LBuE, LHE, LHuE, LWE, LLE,
                                LWPC)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 139-140
```tablegen
// lw[lr]
def : InstRW<[P5600WriteLoadShifted], (instrs LWL, LWR, LWLE, LWRE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 142-144
```tablegen
// s[bhw], sw[lr]
def : InstRW<[P5600WriteStore], (instrs SB, SH, SW, SWC2, SWC3, SDC2, SDC3, SC,
                                 SBE, SHE, SWE, SCE, SWL, SWR, SWLE, SWRE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 146-148
```tablegen
// pref, cache, sync, synci
def : InstRW<[P5600WriteCache], (instrs PREF, PREFE, CACHE, CACHEE, SYNC,
                                 SYNCI)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 150-154
```tablegen
// LDST is also used in moves from general purpose registers to floating point
// and MSA.
def P5600WriteMoveGPRToOtherUnits : SchedWriteRes<[P5600IssueLDST]> {
  let Latency = 0;
}
```
- EN: Defines TableGen record `P5600WriteMoveGPRToOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteMoveGPRToOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 156-157
```tablegen
// AL2 Pipeline
// ------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 159-176
```tablegen
def P5600WriteAL2 : SchedWriteRes<[P5600IssueAL2]>;
def P5600WriteAL2BitExt : SchedWriteRes<[P5600IssueAL2]> { let Latency = 2; }
def P5600WriteAL2ShadowMov : SchedWriteRes<[P5600IssueAL2]> { let Latency = 2; }
def P5600WriteAL2CondMov : SchedWriteRes<[P5600IssueAL2, P5600CTISTD]> {
  let Latency = 2;
}
def P5600WriteAL2Div : SchedWriteRes<[P5600IssueAL2, P5600AL2Div]> {
  // Estimated worst case
  let Latency = 34;
  let ReleaseAtCycles = [1, 34];
}
def P5600WriteAL2DivU : SchedWriteRes<[P5600IssueAL2, P5600AL2Div]> {
  // Estimated worst case
  let Latency = 34;
  let ReleaseAtCycles = [1, 34];
}
def P5600WriteAL2Mul : SchedWriteRes<[P5600IssueAL2]> { let Latency = 3; }
def P5600WriteAL2Mult: SchedWriteRes<[P5600IssueAL2]> { let Latency = 5; }
```
- EN: Defines TableGen record `P5600WriteAL2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteAL2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 177-179
```tablegen
def P5600WriteAL2MAdd: SchedWriteRes<[P5600IssueAL2, P5600CTISTD]> {
  let Latency = 5;
}
```
- EN: Defines TableGen record `P5600WriteAL2MAdd:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteAL2MAdd:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 181-183
```tablegen
// clo, clz, di, ei, mfhi, mflo
def : InstRW<[P5600WriteAL2], (instrs CLO, CLZ, DI, EI, MFHI, MFLO,
                               PseudoMFHI, PseudoMFLO)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 185-186
```tablegen
// ehb, rdhwr, rdpgpr, wrpgpr, wsbh
def : InstRW<[P5600WriteAL2ShadowMov], (instrs EHB, RDHWR, WSBH)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 188-189
```tablegen
// mov[nz]
def : InstRW<[P5600WriteAL2CondMov], (instrs MOVN_I_I, MOVZ_I_I)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 191-193
```tablegen
// divu?
def : InstRW<[P5600WriteAL2Div], (instrs DIV, PseudoSDIV, SDIV)>;
def : InstRW<[P5600WriteAL2DivU], (instrs DIVU, PseudoUDIV, UDIV)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 195-203
```tablegen
// mul
def : InstRW<[P5600WriteAL2Mul], (instrs MUL)>;
// multu?, multu?
def : InstRW<[P5600WriteAL2Mult], (instrs MULT, MULTu, PseudoMULT,
                                   PseudoMULTu)>;
// maddu?, msubu?, mthi, mtlo
def : InstRW<[P5600WriteAL2MAdd], (instrs MADD, MADDU, MSUB, MSUBU,
                                   MTHI, MTLO, PseudoMADD, PseudoMADDU,
                                   PseudoMSUB, PseudoMSUBU, PseudoMTLOHI)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 205-206
```tablegen
// ext, ins
def : InstRW<[P5600WriteAL2BitExt], (instrs EXT, INS)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 208-214
```tablegen
// Either ALU or AL2 Pipelines
// ---------------------------
//
// Some instructions can choose between ALU and AL2, but once dispatched to
// ALQ or AGQ respectively they are committed to that path.
// The decision is based on the outcome of the most recent selection when the
// choice was last available. For now, we assume ALU is always chosen.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 216-220
```tablegen
def P5600WriteEitherALU : SchedWriteVariant<
  // FIXME: Implement selection predicate
  [SchedVar<SchedPredicate<[{1}]>, [P5600WriteALU]>,
   SchedVar<SchedPredicate<[{0}]>, [P5600WriteAL2]>
  ]>;
```
- EN: Defines TableGen record `P5600WriteEitherALU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteEitherALU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 222-226
```tablegen
// add, addi, addiu, addu, andi, ori, rotr, se[bh], sllv?, sr[al]v?, slt, sltu,
// xori
def : InstRW<[P5600WriteEitherALU], (instrs ADD, ADDi, ADDiu, ANDi, ORi, ROTR,
                                     SEB, SEH, SLT, SLTu, SLL, SRA, SRL, XORi,
                                     ADDu, SLLV, SRAV, SRLV, LSA, COPY)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 228-229
```tablegen
// FPU Pipelines
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 231-234
```tablegen
def P5600FPQ : ProcResource<3> { let BufferSize = 16; }
def P5600IssueFPUS : ProcResource<1> { let Super = P5600FPQ; }
def P5600IssueFPUL : ProcResource<1> { let Super = P5600FPQ; }
def P5600IssueFPULoad : ProcResource<1> { let Super = P5600FPQ; }
```
- EN: Defines TableGen record `P5600FPQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600FPQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 236-236
```tablegen
def P5600FPUDivSqrt : ProcResource<2>;
```
- EN: Defines TableGen record `P5600FPUDivSqrt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600FPUDivSqrt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-255
```tablegen
def P5600WriteFPUS : SchedWriteRes<[P5600IssueFPUS]>;
def P5600WriteFPUL : SchedWriteRes<[P5600IssueFPUL]> { let Latency = 4; }
def P5600WriteFPUL_MADDSUB : SchedWriteRes<[P5600IssueFPUL]> { let Latency = 6; }
def P5600WriteFPUDivI : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 23 / 27
  let Latency = 23; // Using common case
  let ReleaseAtCycles = [ 1, 23 ];
}
def P5600WriteFPUDivS : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 23 / 27
  let Latency = 23; // Using common case
  let ReleaseAtCycles = [ 1, 23 ];
}
def P5600WriteFPUDivD : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 31 / 35
  let Latency = 31; // Using common case
  let ReleaseAtCycles = [ 1, 31 ];
}
```
- EN: Defines TableGen record `P5600WriteFPUS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteFPUS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 256-273
```tablegen
def P5600WriteFPURcpS : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 19 / 23
  let Latency = 19; // Using common case
  let ReleaseAtCycles = [ 1, 19 ];
}
def P5600WriteFPURcpD : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 27 / 31
  let Latency = 27; // Using common case
  let ReleaseAtCycles = [ 1, 27 ];
}
def P5600WriteFPURsqrtS : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 27 / 27
  let Latency = 27; // Using common case
  let ReleaseAtCycles = [ 1, 27 ];
}
def P5600WriteFPURsqrtD : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 27 / 31
  let Latency = 27; // Using common case
```
- EN: Defines TableGen record `P5600WriteFPURcpS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteFPURcpS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 274-290
```tablegen
  let ReleaseAtCycles = [ 1, 27 ];
}
def P5600WriteFPUSqrtS : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 27 / 31
  let Latency = 27; // Using common case
  let ReleaseAtCycles = [ 1, 27 ];
}
def P5600WriteFPUSqrtD : SchedWriteRes<[P5600IssueFPUL, P5600FPUDivSqrt]> {
  // Best/Common/Worst case = 7 / 35 / 39
  let Latency = 35; // Using common case
  let ReleaseAtCycles = [ 1, 35 ];
}
def P5600WriteMSAShortLogic : SchedWriteRes<[P5600IssueFPUS]>;
def P5600WriteMSAShortInt : SchedWriteRes<[P5600IssueFPUS]> { let Latency = 2; }
def P5600WriteMoveOtherUnitsToFPU : SchedWriteRes<[P5600IssueFPUS]>;
def P5600WriteMSAOther3 : SchedWriteRes<[P5600IssueFPUS]> { let Latency = 3; }
def P5600WriteMSALongInt : SchedWriteRes<[P5600IssueFPUS]> { let Latency = 5; }
```
- EN: Defines TableGen record `P5600WriteFPUSqrtS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteFPUSqrtS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 292-305
```tablegen
// vshf.[bhwd], binsl.[bhwd], binsr.[bhwd], insert.[bhwd], sld?.[bhwd],
// bset.[bhwd], bclr.[bhwd], bneg.[bhwd], bsel_v, bseli_b
def : InstRW<[P5600WriteMSAShortInt], (instregex "^VSHF_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(BINSL|BINSLI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(BINSR|BINSRI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^INSERT_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(SLD|SLDI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(BSET|BSETI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(BCLR|BCLRI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(BNEG|BNEGI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(BSEL_V|BSELI_B)$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^BMN*Z.*$")>;
def : InstRW<[P5600WriteMSAShortInt],
             (instregex "^BSEL_(H|W|D|FW|FD)_PSEUDO$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 307-309
```tablegen
// pcnt.[bhwd], sat_s.[bhwd], sat_u.bhwd]
def : InstRW<[P5600WriteMSAOther3], (instregex "^PCNT_[BHWD]$")>;
def : InstRW<[P5600WriteMSAOther3], (instregex "^SAT_(S|U)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 311-313
```tablegen
// bnz.[bhwdv], cfcmsa, ctcmsa
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(BNZ|BZ)_[BHWDV]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^C(F|T)CMSA$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 315-319
```tablegen
// FPUS is also used in moves from floating point and MSA registers to general
// purpose registers.
def P5600WriteMoveFPUSToOtherUnits : SchedWriteRes<[P5600IssueFPUS]> {
  let Latency = 0;
}
```
- EN: Defines TableGen record `P5600WriteMoveFPUSToOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteMoveFPUSToOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 321-323
```tablegen
// FPUL is also used in moves from floating point and MSA registers to general
// purpose registers.
def P5600WriteMoveFPULToOtherUnits : SchedWriteRes<[P5600IssueFPUL]>;
```
- EN: Defines TableGen record `P5600WriteMoveFPULToOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteMoveFPULToOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 325-334
```tablegen
// Short Pipe
// ----------
//
// abs.[ds], abs.ps, bc1[tf]l?, mov[tf].[ds], mov[tf], mov.[ds], [cm][ft]c1,
// m[ft]hc1, neg.[ds], neg.ps, nor.v, nori.b, or.v, ori.b, xor.v, xori.b,
// sdxc1, sdc1, st.[bhwd], swc1, swxc1
def : InstRW<[P5600WriteFPUS], (instrs FABS_S, FABS_D32, FABS_D64, MOVF_D32,
                                MOVF_D64, MOVF_S, MOVT_D32, MOVT_D64,
                                MOVT_S, FMOV_D32, FMOV_D64, FMOV_S, FNEG_S,
                                FNEG_D32, FNEG_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 336-346
```tablegen
// adds_a.[bhwd], adds_[asu].[bhwd], addvi?.[bhwd], asub_[us].[bhwd],
// aver?_[us].[bhwd], shf.[bhw], fill[bhwd], splat?.[bhwd]
def : InstRW<[P5600WriteMSAShortInt], (instregex "^ADD_A_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^ADDS_[ASU]_[BHWD]$")>;
// TODO: ADDVI_[BHW] might be 1 cycle latency rather than 2. Need to confirm it.
def : InstRW<[P5600WriteMSAShortInt], (instregex "^ADDVI?_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^ASUB_[US].[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^AVER?_[US].[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^SHF_[BHW]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^FILL_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^(SPLAT|SPLATI)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 348-356
```tablegen
// and.v, andi.b, move.v, ldi.[bhwd]
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^MOVE_V$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^LDI_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(AND|OR|[XN]OR)_V$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(AND|OR|[XN]OR)I_B$")>;
def : InstRW<[P5600WriteMSAShortLogic],
             (instregex "^(AND|OR|[XN]OR)_V_[DHW]_PSEUDO$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^FILL_F(D|W)_PSEUDO$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^INSERT_F(D|W)_PSEUDO$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 358-359
```tablegen
// fexp2_w, fexp2_d
def : InstRW<[P5600WriteFPUS], (instregex "^FEXP2_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 361-378
```tablegen
// compare, converts, round to int, floating point truncate.
def : InstRW<[P5600WriteFPUS], (instregex "^(CLT|CLTI)_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^(CLE|CLEI)_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^(CEQ|CEQI)_[BHWD]$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_UN_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_UEQ_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_EQ_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_LT_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_ULT_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_LE_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^CMP_ULE_(S|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FS(AF|EQ|LT|LE|NE|OR)_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FSUEQ_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FSULE_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FSULT_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FSUNE_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FSUN_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCAF_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 379-396
```tablegen
def : InstRW<[P5600WriteFPUS], (instregex "^FCEQ_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCLE_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCLT_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCNE_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCOR_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCUEQ_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCULE_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCULT_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCUNE_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FCUN_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FABS_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FFINT_(U|S)_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FFQL_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FFQR_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FTINT_(U|S)_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FRINT_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FTQ_(H|W)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FTRUNC_(U|S)_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 398-401
```tablegen
// fexdo.[hw], fexupl.[wd], fexupr.[wd]
def : InstRW<[P5600WriteFPUS], (instregex "^FEXDO_(H|W)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FEXUPL_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FEXUPR_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 403-409
```tablegen
// fclass.[wd], fmax.[wd], fmax_a.[wd], fmin.[wd], fmin_a.[wd], flog2.[wd]
def : InstRW<[P5600WriteFPUS], (instregex "^FCLASS_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FMAX_A_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FMAX_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FMIN_A_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FMIN_(W|D)$")>;
def : InstRW<[P5600WriteFPUS], (instregex "^FLOG2_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 411-414
```tablegen
// interleave right/left, interleave even/odd, insert
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(ILVR|ILVL)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(ILVEV|ILVOD)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^INSVE_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 416-421
```tablegen
// subs_?.[bhwd], subsus_?.[bhwd], subsuu_?.[bhwd], subvi.[bhwd], subv.[bhwd],
def : InstRW<[P5600WriteMSAShortInt], (instregex "^SUBS_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^SUBSUS_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^SUBSUU_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^SUBVI_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortInt], (instregex "^SUBV_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 423-425
```tablegen
// mod_[su].[bhwd], div_[su].[bhwd]
def : InstRW<[P5600WriteFPUDivI], (instregex "^MOD_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteFPUDivI], (instregex "^DIV_(S|U)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 427-443
```tablegen
// hadd_[su].[bhwd], hsub_[su].[bhwd], max_[sua].[bhwd], min_[sua].[bhwd],
// maxi_[su].[bhwd], mini_[su].[bhwd], sra?.[bhwd], srar?.[bhwd], srlr.[bhwd],
// sll?.[bhwd], pckev.[bhwd], pckod.[bhwd], nloc.[bhwd], nlzc.[bhwd],
// insve.[bhwd]
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^HADD_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^HSUB_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(MAX|MIN)_S_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(MAX|MIN)_U_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(MAX|MIN)_A_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(MAXI|MINI)_(S|U)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(SRA|SRAI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(SRL|SRLI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(SRAR|SRARI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(SRLR|SRLRI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(SLL|SLLI)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(PCKEV|PCKOD)_[BHWD]$")>;
def : InstRW<[P5600WriteMSAShortLogic], (instregex "^(NLOC|NLZC)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 445-461
```tablegen
// Long Pipe
// ----------
//
// add.[ds], add.ps, cvt.d.[sw], cvt.s.[dw], cvt.w.[sd], cvt.[sw].ps,
// cvt.ps.[sw], cvt.s.(pl|pu), c.<cc>.[ds], c.<cc>.ps, mul.[ds], mul.ps,
// pl[lu].ps, sub.[ds], sub.ps, trunc.w.[ds], trunc.w.ps
def : InstRW<[P5600WriteFPUL],
             (instrs FADD_D32, FADD_D64, FADD_PS64, FADD_S, FMUL_D32, FMUL_D64,
              FMUL_PS64, FMUL_S, FSUB_D32, FSUB_D64, FSUB_PS64, FSUB_S)>;
def : InstRW<[P5600WriteFPUL], (instregex "^TRUNC_(L|W)_(S|D32|D64)$")>;
def : InstRW<[P5600WriteFPUL],
             (instregex "^CVT_(S|D32|D64|L|W)_(S|D32|D64|L|W)$")>;
def : InstRW<[P5600WriteFPUL], (instrs CVT_PS_S64, CVT_S_PL64, CVT_S_PU64)>;
def : InstRW<[P5600WriteFPUL], (instregex "^C_[A-Z]+_(S|D32|D64)$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^FCMP_(S32|D32|D64)$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^PseudoCVT_(S|D32|D64)_(L|W)$")>;
def : InstRW<[P5600WriteFPUL], (instrs PLL_PS64, PLU_PS64, PUL_PS64, PUU_PS64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 463-465
```tablegen
// div.[ds], div.ps
def : InstRW<[P5600WriteFPUDivS], (instrs FDIV_S)>;
def : InstRW<[P5600WriteFPUDivD], (instrs FDIV_D32, FDIV_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 467-469
```tablegen
// sqrt.[ds], sqrt.ps
def : InstRW<[P5600WriteFPUSqrtS], (instrs FSQRT_S)>;
def : InstRW<[P5600WriteFPUSqrtD], (instrs FSQRT_D32, FSQRT_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 471-473
```tablegen
// frcp.[wd], frsqrt.[wd]
def : InstRW<[P5600WriteFPURsqrtD], (instregex "^FRCP_(W|D)$")>;
def : InstRW<[P5600WriteFPURsqrtD], (instregex "^FRSQRT_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 475-477
```tablegen
def : InstRW<[P5600WriteFPURsqrtD], (instrs RECIP_D32, RECIP_D64, RSQRT_D32,
                                     RSQRT_D64)>;
def : InstRW<[P5600WriteFPURsqrtS], (instrs RECIP_S, RSQRT_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 479-489
```tablegen
// fmadd.[wd], fmsubb.[wd], fdiv.[wd], fsqrt.[wd], fmul.[wd], fadd.[wd],
// fsub.[wd]
def : InstRW<[P5600WriteFPUL_MADDSUB], (instregex "^FMADD_(W|D)$")>;
def : InstRW<[P5600WriteFPUL_MADDSUB], (instregex "^FMSUB_(W|D)$")>;
def : InstRW<[P5600WriteFPUDivS], (instregex "^FDIV_W$")>;
def : InstRW<[P5600WriteFPUDivD], (instregex "^FDIV_D$")>;
def : InstRW<[P5600WriteFPUSqrtS], (instregex "^FSQRT_W$")>;
def : InstRW<[P5600WriteFPUSqrtD], (instregex "^FSQRT_D$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^FMUL_(W|D)$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^FADD_(W|D)$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^FSUB_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 491-498
```tablegen
// dpadd_?.[bhwd], dpsub_?.[bhwd], dotp_?.[bhwd], msubv.[bhwd], maddv.[bhwd]
// mulv.[bhwd].
def : InstRW<[P5600WriteMSALongInt], (instregex "^DPADD_(S|U)_[HWD]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^DPSUB_(S|U)_[HWD]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^DOTP_(S|U)_[HWD]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MSUBV_[BHWD]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MADDV_[BHWD]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MULV_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 500-505
```tablegen
def : InstRW<[P5600WriteMSALongInt], (instregex "^MADDR_Q_[HW]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MADD_Q_[HW]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MSUBR_Q_[HW]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MSUB_Q_[HW]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MULR_Q_[HW]$")>;
def : InstRW<[P5600WriteMSALongInt], (instregex "^MUL_Q_[HW]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 507-511
```tablegen
// madd.[ds], msub.[ds], nmadd.[ds], nmsub.[ds],
// Operand 0 is read on cycle 5. All other operands are read on operand 0.
def : InstRW<[SchedReadAdvance<5>, P5600WriteFPUL_MADDSUB],
             (instrs MADD_D32, MADD_D64, MADD_S, MSUB_D32, MSUB_D64, MSUB_S,
              NMADD_D32, NMADD_D64, NMADD_S, NMSUB_D32, NMSUB_D64, NMSUB_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 513-515
```tablegen
// madd.ps, msub.ps, nmadd.ps, nmsub.ps
// Operand 0 and 1 are read on cycle 5. All others are read on operand 0.
// (none of these instructions exist in the backend yet)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 517-521
```tablegen
// Load Pipe
// ---------
//
// This is typically used in conjunction with the load pipeline under the AGQ
// All the instructions are in the 'Tricky Instructions' section.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 523-525
```tablegen
def P5600WriteLoadOtherUnitsToFPU : SchedWriteRes<[P5600IssueFPULoad]> {
  let Latency = 4;
}
```
- EN: Defines TableGen record `P5600WriteLoadOtherUnitsToFPU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteLoadOtherUnitsToFPU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 527-531
```tablegen
// Tricky Instructions
// ===================
//
// These instructions are split across multiple uops (in different pipelines)
// that must cooperate to complete the operation
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 533-537
```tablegen
// FIXME: This isn't quite right since the implementation of WriteSequence
//        current aggregates the resources and ignores the exact cycle they are
//        used.
def P5600WriteMoveGPRToFPU : WriteSequence<[P5600WriteMoveGPRToOtherUnits,
                                            P5600WriteMoveOtherUnitsToFPU]>;
```
- EN: Defines TableGen record `P5600WriteMoveGPRToFPU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteMoveGPRToFPU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 539-543
```tablegen
// FIXME: This isn't quite right since the implementation of WriteSequence
//        current aggregates the resources and ignores the exact cycle they are
//        used.
def P5600WriteMoveFPUToGPR : WriteSequence<[P5600WriteMoveFPUSToOtherUnits,
                                            P5600WriteGPRFromBypass]>;
```
- EN: Defines TableGen record `P5600WriteMoveFPUToGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteMoveFPUToGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 545-549
```tablegen
// FIXME: This isn't quite right since the implementation of WriteSequence
//        current aggregates the resources and ignores the exact cycle they are
//        used.
def P5600WriteStoreFPUS : WriteSequence<[P5600WriteMoveFPUSToOtherUnits,
                                         P5600WriteStoreFromOtherUnits]>;
```
- EN: Defines TableGen record `P5600WriteStoreFPUS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteStoreFPUS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 551-555
```tablegen
// FIXME: This isn't quite right since the implementation of WriteSequence
//        current aggregates the resources and ignores the exact cycle they are
//        used.
def P5600WriteStoreFPUL : WriteSequence<[P5600WriteMoveFPULToOtherUnits,
                                         P5600WriteStoreFromOtherUnits]>;
```
- EN: Defines TableGen record `P5600WriteStoreFPUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteStoreFPUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 557-561
```tablegen
// FIXME: This isn't quite right since the implementation of WriteSequence
//        current aggregates the resources and ignores the exact cycle they are
//        used.
def P5600WriteLoadFPU : WriteSequence<[P5600WriteLoadToOtherUnits,
                                       P5600WriteLoadOtherUnitsToFPU]>;
```
- EN: Defines TableGen record `P5600WriteLoadFPU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600WriteLoadFPU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 563-566
```tablegen
// ctc1, mtc1, mthc1
def : InstRW<[P5600WriteMoveGPRToFPU], (instrs CTC1, MTC1, MTC1_D64, MTHC1_D32,
                                        MTHC1_D64, BuildPairF64,
                                        BuildPairF64_64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 568-570
```tablegen
// copy.[su]_[bhwd]
def : InstRW<[P5600WriteMoveFPUToGPR], (instregex "^COPY_U_[BHW]$")>;
def : InstRW<[P5600WriteMoveFPUToGPR], (instregex "^COPY_S_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 572-576
```tablegen
// bc1[ft], cfc1, mfc1, mfhc1, movf, movt
def : InstRW<[P5600WriteMoveFPUToGPR], (instrs BC1F, BC1FL, BC1T, BC1TL, CFC1,
                                        MFC1, MFC1_D64, MFHC1_D32, MFHC1_D64,
                                        MOVF_I, MOVT_I, ExtractElementF64,
                                        ExtractElementF64_64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 578-582
```tablegen
// swc1, swxc1, st.[bhwd]
def : InstRW<[P5600WriteStoreFPUS], (instrs SDC1, SDC164, SDXC1, SDXC164,
                                     SWC1, SWXC1, SUXC1, SUXC164)>;
def : InstRW<[P5600WriteStoreFPUS], (instregex "^ST_[BHWD]$")>;
def : InstRW<[P5600WriteStoreFPUS], (instrs ST_F16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 584-586
```tablegen
// movn.[ds], movz.[ds]
def : InstRW<[P5600WriteStoreFPUL], (instrs MOVN_I_D32, MOVN_I_D64, MOVN_I_S,
                                     MOVZ_I_D32, MOVZ_I_D64, MOVZ_I_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 588-592
```tablegen
// l[dw]x?c1, ld.[bhwd]
def : InstRW<[P5600WriteLoadFPU], (instrs LDC1, LDC164, LDXC1, LDXC164,
                                   LWC1, LWXC1, LUXC1, LUXC164)>;
def : InstRW<[P5600WriteLoadFPU], (instregex "LD_[BHWD]")>;
def : InstRW<[P5600WriteLoadFPU], (instrs LD_F16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 594-609
```tablegen
// Unsupported Instructions
// ========================
//
// The following instruction classes are never valid on P5600.
//   II_DADDIU, II_DADDU, II_DMFC1, II_DMTC1, II_DMULT, II_DMULTU, II_DROTR,
//   II_DROTR32, II_DROTRV, II_DDIV, II_DSLL, II_DSLL32, II_DSLLV, II_DSRA,
//   II_DSRA32, II_DSRAV, II_DSRL, II_DSRL32, II_DSRLV, II_DSUBU, II_DDIVU,
//   II_JALRC, II_LD, II_LD[LR], II_RESTORE, II_SAVE, II_SD, II_SDC1, II_SD[LR]
//
// The following instructions are never valid on P5600.
//   addq.ph, repl.ph, repl.qb, subq.ph, subu_s.qb
//
// Guesswork
// =========
//
// This section is largely temporary guesswork.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 611-615
```tablegen
// ceil.[lw].[ds], floor.[lw].[ds]
// Reason behind guess: trunc.[lw].ds and the various cvt's are in FPUL
def : InstRW<[P5600WriteFPUL], (instregex "^CEIL_(L|W)_(S|D32|D64)$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^FLOOR_(L|W)_(S|D32|D64)$")>;
def : InstRW<[P5600WriteFPUL], (instregex "^ROUND_(L|W)_(S|D32|D64)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 617-620
```tablegen
// rotrv
// Reason behind guess: rotr is in the same category and the two register forms
//                      generally follow the immediate forms in this category
def : InstRW<[P5600WriteEitherALU], (instrs ROTRV)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 622-622
```tablegen
// Atomic instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 624-629
```tablegen
// FIXME: Define `WriteAtomic` in the MipsSchedule.td and
// attach it to the Atomic2OpsPostRA, AtomicCmpSwapPostRA, ...
// classes. Then just define resources for the `WriteAtomic` in each
// machine models.
def P5600Atomic : ProcResource<1> { let BufferSize = 1; }
def P5600WriteAtomic : SchedWriteRes<[P5600Atomic]> { let Latency = 2; }
```
- EN: Defines TableGen record `P5600Atomic` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `P5600Atomic`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 631-638
```tablegen
def : InstRW<[P5600WriteAtomic],
    (instregex "^ATOMIC_SWAP_I(8|16|32|64)_POSTRA$")>;
def : InstRW<[P5600WriteAtomic],
    (instregex "^ATOMIC_CMP_SWAP_I(8|16|32|64)_POSTRA$")>;
def : InstRW<[P5600WriteAtomic],
    (instregex "^ATOMIC_LOAD_(ADD|SUB|AND|OR|XOR|NAND|MIN|MAX|UMIN|UMAX)"
               "_I(8|16|32|64)_POSTRA$")>;
}
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

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
