# MipsScheduleGeneric.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsScheduleGeneric.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the interAptiv processor in a manner of speaking. It describes a hypothetical version of the in-order MIPS32R2 interAptiv with all branches of the MIPS ISAs, ASEs and ISA variants. The itinerary lists are broken down into per ISA lists, so that this file can be used to rapidly develop new schedule models.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsScheduleGeneric`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```tablegen
//=- MipsScheduleGeneric.td - Generic Scheduling Definitions -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the interAptiv processor in a manner of speaking. It
// describes a hypothetical version of the in-order MIPS32R2 interAptiv with all
// branches of the MIPS ISAs, ASEs and ISA variants. The itinerary lists are
// broken down into per ISA lists, so that this file can be used to rapidly
// develop new schedule models.
//
//===----------------------------------------------------------------------===//
def MipsGenericModel : SchedMachineModel {
  int IssueWidth = 1;
  int MicroOpBufferSize = 0;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 20-22
```tablegen
  // These figures assume an L1 hit.
  int LoadLatency = 2;
  int MispredictPenalty = 4;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 24-25
```tablegen
  int HighLatency = 37;
  list<Predicate> UnsupportedFeatures = [];
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 27-28
```tablegen
  let CompleteModel = 1;
  let PostRAScheduler = 1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 30-32
```tablegen
  // FIXME: Remove when all errors have been fixed.
  let FullInstRWOverlapCheck = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 34-34
```tablegen
let SchedModel = MipsGenericModel in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 36-37
```tablegen
// ALU Pipeline
// ============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 39-40
```tablegen
def GenericALU : ProcResource<1> { let BufferSize = 1; }
def GenericIssueALU : ProcResource<1> { let Super = GenericALU; }
```
- EN: Defines TableGen record `GenericALU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericALU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 42-42
```tablegen
def GenericWriteALU : SchedWriteRes<[GenericIssueALU]>;
```
- EN: Defines TableGen record `GenericWriteALU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteALU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 44-51
```tablegen
// add, addi, addiu, addu, and, andi, clo, clz, ext, ins, lui, nor, or, ori,
// rotr, rotrv, seb, seh, sll, sllv, slt, slti, sltiu, sltu, sra, srav, srl,
// srlv, ssnop, sub, subu, wsbh, xor, xori
def : InstRW<[GenericWriteALU], (instrs ADD, ADDi, ADDiu, ADDu, AND, ANDi,
                                 CLO, CLZ, EXT, INS, LEA_ADDiu, LUi, NOP,
                                 NOR, OR, ORi, ROTR, ROTRV, SEB, SEH, SLL,
                                 SLLV, SLT, SLTi, SLTiu, SLTu, SRA, SRAV, SRL,
                                 SRLV, SSNOP, SUB, SUBu, WSBH, XOR, XORi)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 53-53
```tablegen
def : InstRW<[GenericWriteALU], (instrs COPY)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 55-56
```tablegen
// MIPSR6
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 58-61
```tablegen
// addiupc, align, aluipc, aui, auipc, bitswap, clo, clz, lsa, seleqz, selnez
def : InstRW<[GenericWriteALU], (instrs ADDIUPC, ALIGN, ALUIPC, AUI,
                                 AUIPC, BITSWAP, CLO_R6, CLZ_R6, LSA_R6,
                                 SELEQZ, SELNEZ)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 63-64
```tablegen
// MIPS16e
// =======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 66-80
```tablegen
def : InstRW<[GenericWriteALU], (instrs AddiuRxImmX16, AddiuRxRxImm16,
                                 AddiuRxRxImmX16, AddiuRxRyOffMemX16,
                                 AddiuRxPcImmX16, AddiuSpImm16, AddiuSpImmX16,
                                 AdduRxRyRz16, AndRxRxRy16, CmpRxRy16,
                                 CmpiRxImm16, CmpiRxImmX16, LiRxImm16,
                                 LiRxImmX16, LiRxImmAlignX16, Move32R16,
                                 MoveR3216, Mfhi16, Mflo16, NegRxRy16,
                                 NotRxRy16, OrRxRxRy16, SebRx16, SehRx16,
                                 SllX16, SllvRxRy16, SltiRxImm16,
                                 SltiRxImmX16, SltiCCRxImmX16,
                                 SltiuRxImm16, SltiuRxImmX16, SltiuCCRxImmX16,
                                 SltRxRy16, SltCCRxRy16, SltuRxRy16,
                                 SltuRxRyRz16, SltuCCRxRy16, SravRxRy16,
                                 SraX16, SrlvRxRy16, SrlX16, SubuRxRyRz16,
                                 XorRxRxRy16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 82-83
```tablegen
def : InstRW<[GenericWriteALU], (instrs Constant32, LwConstant32,
                                 GotPrologue16, CONSTPOOL_ENTRY)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 85-86
```tablegen
// microMIPS
// =========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 88-99
```tablegen
def : InstRW<[GenericWriteALU], (instrs ADDIUPC_MM, ADDIUR1SP_MM, ADDIUR2_MM,
                                 ADDIUS5_MM, ADDIUSP_MM, ADDU16_MM, ADD_MM,
                                 ADDi_MM, ADDiu_MM, ADDu_MM, AND16_MM,
                                 ANDI16_MM, AND_MM, ANDi_MM, CLO_MM, CLZ_MM,
                                 EXT_MM, INS_MM, LEA_ADDiu_MM, LI16_MM,
                                 LUi_MM, MOVE16_MM, MOVEP_MM, NOR_MM,
                                 NOT16_MM, OR16_MM, OR_MM, ORi_MM, ROTRV_MM,
                                 ROTR_MM, SEB_MM, SEH_MM, SLL16_MM, SLLV_MM,
                                 SLL_MM, SLT_MM, SLTi_MM, SLTiu_MM, SLTu_MM,
                                 SRAV_MM, SRA_MM, SRL16_MM, SRLV_MM, SRL_MM,
                                 SSNOP_MM, SUBU16_MM, SUB_MM, SUBu_MM,
                                 WSBH_MM, XOR16_MM, XOR_MM, XORi_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 101-102
```tablegen
// microMIPS32r6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 104-114
```tablegen
def : InstRW<[GenericWriteALU], (instrs ADDIUPC_MMR6, ADDIU_MMR6, ADDU16_MMR6,
                                 ADDU_MMR6, ADD_MMR6, ALIGN_MMR6, ALUIPC_MMR6,
                                 AND16_MMR6, ANDI16_MMR6, ANDI_MMR6, AND_MMR6,
                                 AUIPC_MMR6, AUI_MMR6, BITSWAP_MMR6, CLO_MMR6,
                                 CLZ_MMR6, EXT_MMR6, INS_MMR6, LI16_MMR6,
                                 LSA_MMR6, LUI_MMR6, MOVE16_MMR6, NOR_MMR6,
                                 NOT16_MMR6, OR16_MMR6, ORI_MMR6, OR_MMR6,
                                 SELEQZ_MMR6, SELNEZ_MMR6, SLL16_MMR6,
                                 SLL_MMR6, SRL16_MMR6, SSNOP_MMR6, SUBU16_MMR6,
                                 SUBU_MMR6, SUB_MMR6, WSBH_MMR6, XOR16_MMR6,
										             XORI_MMR6, XOR_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 116-117
```tablegen
// MIPS64
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 119-122
```tablegen
def : InstRW<[GenericWriteALU], (instrs AND64, ANDi64, DEXT64_32, DSLL64_32,
                                 ORi64, SEB64, SEH64, SLL64_32, SLL64_64,
                                 SLT64, SLTi64, SLTiu64, SLTu64, XOR64,
                                 XORi64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 124-129
```tablegen
def : InstRW<[GenericWriteALU], (instrs DADD, DADDi, DADDiu, DADDu, DCLO,
                                 DCLZ, DEXT, DEXTM, DEXTU, DINS, DINSM, DINSU,
                                 DROTR, DROTR32, DROTRV, DSBH, DSHD, DSLL,
                                 DSLL32, DSLLV, DSRA, DSRA32, DSRAV, DSRL,
                                 DSRL32, DSRLV, DSUB, DSUBu, LEA_ADDiu64,
                                 LUi64, NOR64, OR64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 131-132
```tablegen
// MIPS64R6
// ========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 134-136
```tablegen
def : InstRW<[GenericWriteALU], (instrs DALIGN, DAHI, DATI, DAUI, DCLO_R6,
                                 DCLZ_R6, DBITSWAP, DLSA, DLSA_R6, SELEQZ64,
                                 SELNEZ64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 139-145
```tablegen
def GenericMDU : ProcResource<1> { let BufferSize = 1; }
def GenericIssueMDU : ProcResource<1> { let Super = GenericALU; }
def GenericIssueDIV : ProcResource<1> { let Super = GenericMDU; }
def GenericWriteHILO : SchedWriteRes<[GenericIssueMDU]>;
def GenericWriteALULong : SchedWriteRes<[GenericIssueALU]> { let Latency = 5; }
def GenericWriteMove : SchedWriteRes<[GenericIssueALU]> { let Latency = 2; }
def GenericWriteMul : SchedWriteRes<[GenericIssueMDU]> { let Latency = 4; }
```
- EN: Defines TableGen record `GenericMDU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericMDU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 147-147
```tablegen
def : InstRW<[GenericWriteHILO], (instrs MADD, MADDU, MSUB, MSUBU)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 149-151
```tablegen
def : InstRW<[GenericWriteHILO], (instrs PseudoMADD_MM, PseudoMADDU_MM,
                                  PseudoMSUB_MM, PseudoMSUBU_MM,
                                  PseudoMULT_MM, PseudoMULTu_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 153-154
```tablegen
def : InstRW<[GenericWriteHILO], (instrs PseudoMADD, PseudoMADDU, PseudoMSUB,
                                  PseudoMSUBU, PseudoMULT, PseudoMULTu)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 156-158
```tablegen
def GenericWriteMDUtoGPR : SchedWriteRes<[GenericIssueMDU]> {
  let Latency = 5;
}
```
- EN: Defines TableGen record `GenericWriteMDUtoGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteMDUtoGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 160-169
```tablegen
def GenericWriteDIV : SchedWriteRes<[GenericIssueDIV]> {
  // Estimated worst case
  let Latency = 33;
  let ReleaseAtCycles = [33];
}
def GenericWriteDIVU : SchedWriteRes<[GenericIssueDIV]> {
  // Estimated worst case
  let Latency = 31;
  let ReleaseAtCycles = [31];
}
```
- EN: Defines TableGen record `GenericWriteDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 171-172
```tablegen
// mul
def : InstRW<[GenericWriteMDUtoGPR], (instrs MUL)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 174-175
```tablegen
// mult, multu
def : InstRW<[GenericWriteMul], (instrs MULT, MULTu)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 177-178
```tablegen
// div, sdiv
def : InstRW<[GenericWriteDIV], (instrs PseudoSDIV, SDIV)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 180-180
```tablegen
def : InstRW<[GenericWriteDIVU], (instrs PseudoUDIV, UDIV)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 182-184
```tablegen
// mfhi, mflo, movn, mthi, mtlo, rdwhr
def : InstRW<[GenericWriteALULong], (instrs MFHI, MFLO, PseudoMFHI,
                                     PseudoMFLO)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 186-186
```tablegen
def : InstRW<[GenericWriteALULong], (instrs PseudoMFHI_MM, PseudoMFLO_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 188-189
```tablegen
def : InstRW<[GenericWriteMove], (instrs MTHI, MTLO, RDHWR, PseudoMTLOHI)>;
def : InstRW<[GenericWriteMove], (instrs PseudoMTLOHI_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 191-191
```tablegen
def : InstRW<[GenericWriteALU], (instrs MOVN_I_I, MOVZ_I_I)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 193-194
```tablegen
// MIPSR6
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 196-197
```tablegen
// muh, muhu, mulu, mul
def : InstRW<[GenericWriteMul], (instrs MUH, MUHU, MULU, MUL_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 199-200
```tablegen
// divu, udiv
def : InstRW<[GenericWriteDIV], (instrs MOD, MODU, DIV, DIVU)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 203-204
```tablegen
// MIPS16e
// =======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 206-207
```tablegen
def : InstRW<[GenericWriteHILO], (instrs MultRxRy16, MultuRxRy16,
                                  MultRxRyRz16, MultuRxRyRz16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 209-209
```tablegen
def : InstRW<[GenericWriteDIV], (instrs DivRxRy16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 211-211
```tablegen
def : InstRW<[GenericWriteDIVU], (instrs DivuRxRy16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 213-214
```tablegen
// microMIPS
// =========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 216-217
```tablegen
def : InstRW<[GenericWriteMul], (instrs MULT_MM, MULTu_MM, MADD_MM, MADDU_MM,
                                 MSUB_MM, MSUBU_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 219-219
```tablegen
def : InstRW<[GenericWriteALULong], (instrs MUL_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 221-221
```tablegen
def : InstRW<[GenericWriteDIV], (instrs SDIV_MM, SDIV_MM_Pseudo)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 223-223
```tablegen
def : InstRW<[GenericWriteDIVU], (instrs UDIV_MM, UDIV_MM_Pseudo)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 225-227
```tablegen
def : InstRW<[GenericWriteMove], (instrs MFHI16_MM, MFLO16_MM, MOVF_I_MM,
                                  MOVT_I_MM, MFHI_MM, MFLO_MM, MTHI_MM,
                                  MTLO_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 229-229
```tablegen
def : InstRW<[GenericWriteMove], (instrs RDHWR_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 231-232
```tablegen
// microMIPS32r6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 234-235
```tablegen
def : InstRW<[GenericWriteMul], (instrs MUHU_MMR6, MUH_MMR6, MULU_MMR6,
                                 MUL_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 237-238
```tablegen
def : InstRW<[GenericWriteDIV], (instrs MODU_MMR6, MOD_MMR6, DIVU_MMR6,
                                 DIV_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 240-240
```tablegen
def : InstRW<[GenericWriteMove], (instrs RDHWR_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 242-243
```tablegen
// MIPS64
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 245-246
```tablegen
def : InstRW<[GenericWriteHILO], (instrs DMULU, DMULT, DMULTu, PseudoDMULT,
                                  PseudoDMULTu)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 248-248
```tablegen
def : InstRW<[GenericWriteDIV], (instrs DSDIV, PseudoDSDIV)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 250-250
```tablegen
def : InstRW<[GenericWriteDIVU], (instrs DUDIV, PseudoDUDIV)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 252-253
```tablegen
def : InstRW<[GenericWriteALULong], (instrs MFHI64, MFLO64, PseudoMFHI64,
                                     PseudoMFLO64, PseudoMTLOHI64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 255-255
```tablegen
def : InstRW<[GenericWriteMove], (instrs MTHI64, MTLO64, RDHWR64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 257-259
```tablegen
// mov[zn]
def : InstRW<[GenericWriteALU], (instrs MOVN_I_I64, MOVN_I64_I, MOVN_I64_I64,
                                 MOVZ_I_I64, MOVZ_I64_I, MOVZ_I64_I64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 262-263
```tablegen
// MIPS64R6
// ========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 265-265
```tablegen
def : InstRW<[GenericWriteMDUtoGPR], (instrs DMUH, DMUHU, DMUL_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 267-267
```tablegen
def : InstRW<[GenericWriteDIV], (instrs DDIV, DMOD)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 269-269
```tablegen
def : InstRW<[GenericWriteDIVU], (instrs DDIVU, DMODU)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 271-272
```tablegen
// CTISTD Pipeline
// ---------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 274-274
```tablegen
def GenericIssueCTISTD : ProcResource<1> { let Super = GenericALU; }
```
- EN: Defines TableGen record `GenericIssueCTISTD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericIssueCTISTD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 276-277
```tablegen
def GenericLDST : ProcResource<1> { let BufferSize = 1; }
def GenericIssueLDST : ProcResource<1> { let Super = GenericLDST; }
```
- EN: Defines TableGen record `GenericLDST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericLDST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 279-282
```tablegen
def GenericWriteJump : SchedWriteRes<[GenericIssueCTISTD]>;
def GenericWriteJumpAndLink : SchedWriteRes<[GenericIssueCTISTD]> {
  let Latency = 2;
}
```
- EN: Defines TableGen record `GenericWriteJump` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteJump`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 284-288
```tablegen
// b, beq, beql, bg[et]z, bl[et]z, bne, bnel, j, syscall, jal, bltzal, jalx,
// jalr, jr.hb, jr, jalr.hb, jarlc, jialc
def : InstRW<[GenericWriteJump], (instrs B, BAL, BAL_BR, BEQ, BNE, BGTZ, BGEZ,
                                  BLEZ, BLTZ, BLTZAL, J, JALX, JR, JR_HB, ERET,
                                  ERet, ERETNC, DERET, NAL)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 290-291
```tablegen
def : InstRW<[GenericWriteJump], (instrs BEQL, BNEL, BGEZL, BGTZL, BLEZL,
                                  BLTZL)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 293-296
```tablegen
def : InstRW<[GenericWriteJump], (instrs TAILCALL, TAILCALLREG,
                                  TAILCALLREGHB, PseudoIndirectBranch,
                                  PseudoIndirectHazardBranch, PseudoReturn,
                                  RetRA)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 298-299
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs BGEZAL, JAL, JALR, JALR_HB,
                                         JALRHBPseudo, JALRPseudo)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 301-301
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs BGEZALL, BLTZALL)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 303-303
```tablegen
def GenericWriteTrap : SchedWriteRes<[GenericIssueCTISTD]>;
```
- EN: Defines TableGen record `GenericWriteTrap` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteTrap`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 305-308
```tablegen
def : InstRW<[GenericWriteTrap], (instrs BREAK, SYSCALL, TEQ, TEQI,
                                  TGE, TGEI, TGEIU, TGEU, TNE,
                                  TNEI, TLT, TLTI, TLTU, TTLTIU,
                                  TRAP, SDBBP)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 310-311
```tablegen
// MIPSR6
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 313-316
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs BALC, BEQZALC, BGEZALC,
                                         BGTZALC, BLEZALC, BLTZALC,
                                         BNEZALC,
                                         JIALC)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 318-322
```tablegen
def : InstRW<[GenericWriteJump], (instrs BC, BC2EQZ, BC2NEZ, BEQC, BEQZC, BGEC,
                                  BGEUC, BGEZC, BGTZC, BLEZC, BLTC, BLTUC,
                                  BLTZC, BNEC, BNEZC, BNVC, BOVC, JIC, JR_HB_R6,
                                  SIGRIE, PseudoIndirectBranchR6,
                                  PseudoIndrectHazardBranchR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 324-324
```tablegen
def : InstRW<[GenericWriteJump], (instrs TAILCALLR6REG, TAILCALLHBR6REG)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 326-326
```tablegen
def : InstRW<[GenericWriteTrap], (instrs SDBBP_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 328-329
```tablegen
// MIPS16e
// =======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 331-340
```tablegen
def : InstRW<[GenericWriteJump], (instrs Bimm16, BimmX16, BeqzRxImm16,
                                  BeqzRxImmX16, BnezRxImm16, BnezRxImmX16,
                                  Bteqz16, BteqzX16, BteqzT8CmpX16,
                                  BteqzT8CmpiX16, BteqzT8SltX16,
                                  BteqzT8SltuX16, BteqzT8SltiX16,
                                  BteqzT8SltiuX16, Btnez16, BtnezX16,
                                  BtnezT8CmpX16, BtnezT8CmpiX16,
                                  BtnezT8SltX16, BtnezT8SltuX16,
                                  BtnezT8SltiX16, BtnezT8SltiuX16, JrRa16,
                                  JrcRa16, JrcRx16, RetRA16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 342-342
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs Jal16, JalB16, JumpLinkReg16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 344-344
```tablegen
def : InstRW<[GenericWriteTrap], (instrs Break16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 346-352
```tablegen
def : InstRW<[GenericWriteALULong], (instrs SelBeqZ, SelTBteqZCmp,
                                     SelTBteqZCmpi, SelTBteqZSlt,
                                     SelTBteqZSlti, SelTBteqZSltu,
                                     SelTBteqZSltiu, SelBneZ, SelTBtneZCmp,
                                     SelTBtneZCmpi, SelTBtneZSlt,
                                     SelTBtneZSlti, SelTBtneZSltu,
                                     SelTBtneZSltiu)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 354-355
```tablegen
// microMIPS
// =========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 357-361
```tablegen
def : InstRW<[GenericWriteJump], (instrs B16_MM, BAL_BR_MM, BC1F_MM, BC1T_MM,
                                  BEQZ16_MM, BEQZC_MM, BEQ_MM, BGEZ_MM,
                                  BGTZ_MM, BLEZ_MM, BLTZ_MM, BNEZ16_MM,
                                  BNEZC_MM, BNE_MM, B_MM, DERET_MM, ERET_MM,
                                  JR16_MM, JR_MM, J_MM, B_MM_Pseudo)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 363-366
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs BGEZALS_MM, BGEZAL_MM,
                                         BLTZALS_MM, BLTZAL_MM, JALR16_MM,
                                         JALRS16_MM, JALRS_MM, JALR_MM,
                                         JALS_MM, JALX_MM, JAL_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 368-369
```tablegen
def : InstRW<[GenericWriteJump], (instrs TAILCALLREG_MM, TAILCALL_MM,
                                  PseudoIndirectBranch_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 371-375
```tablegen
def : InstRW<[GenericWriteTrap], (instrs BREAK16_MM, BREAK_MM, SDBBP16_MM,
                                  SDBBP_MM, SYSCALL_MM, TEQI_MM, TEQ_MM,
                                  TGEIU_MM, TGEI_MM, TGEU_MM, TGE_MM, TLTIU_MM,
                                  TLTI_MM, TLTU_MM, TLT_MM, TNEI_MM, TNE_MM,
                                  TRAP_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 377-378
```tablegen
// microMIPS32r6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 380-390
```tablegen
def : InstRW<[GenericWriteJump], (instrs BC16_MMR6, BC1EQZC_MMR6, BC1NEZC_MMR6,
                                  BC2EQZC_MMR6, BC2NEZC_MMR6, BC_MMR6,
                                  BEQC_MMR6, BEQZC16_MMR6, BEQZC_MMR6,
                                  BGEC_MMR6, BGEUC_MMR6, BGEZC_MMR6,
                                  BGTZC_MMR6, BLEZC_MMR6, BLTC_MMR6,
                                  BLTUC_MMR6, BLTZC_MMR6, BNEC_MMR6,
                                  BNEZC16_MMR6, BNEZC_MMR6, BNVC_MMR6,
                                  BOVC_MMR6, DERET_MMR6, ERETNC_MMR6, JAL_MMR6,
                                  ERET_MMR6, JIC_MMR6, JRADDIUSP, JRC16_MM,
                                  JRC16_MMR6, JRCADDIUSP_MMR6, SIGRIE_MMR6,
                                  B_MMR6_Pseudo, PseudoIndirectBranch_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 392-397
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs BALC_MMR6, BEQZALC_MMR6,
                                         BGEZALC_MMR6, BGTZALC_MMR6,
                                         BLEZALC_MMR6, BLTZALC_MMR6,
                                         BNEZALC_MMR6, JALRC16_MMR6,
                                         JALRC_HB_MMR6, JALRC_MMR6,
                                         JIALC_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 399-399
```tablegen
def : InstRW<[GenericWriteJump], (instrs TAILCALLREG_MMR6, TAILCALL_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 401-402
```tablegen
def : InstRW<[GenericWriteTrap], (instrs BREAK16_MMR6, BREAK_MMR6, SDBBP_MMR6,
                                  SDBBP16_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 404-405
```tablegen
// MIPS64
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 407-408
```tablegen
def : InstRW<[GenericWriteJump], (instrs BEQ64, BGEZ64, BGTZ64, BLEZ64,
                                  BLTZ64, BNE64, JR64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 410-411
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs JALR64, JALR64Pseudo,
                                         JALRHB64Pseudo, JALR_HB64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 413-414
```tablegen
def : InstRW<[GenericWriteJump], (instrs JR_HB64, TAILCALLREG64,
                                  TAILCALLREGHB64, PseudoReturn64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 416-417
```tablegen
// MIPS64R6
// ========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 419-423
```tablegen
def : InstRW<[GenericWriteJump], (instrs BEQC64, BEQZC64, BGEC64, BGEUC64,
                                  BGEZC64, BGTZC64, BLEZC64, BLTC64, BLTUC64,
                                  BLTZC64, BNEC64, BNEZC64, JIC64,
                                  PseudoIndirectBranch64,
                                  PseudoIndirectHazardBranch64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 425-425
```tablegen
def : InstRW<[GenericWriteJumpAndLink], (instrs JIALC64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 427-429
```tablegen
def : InstRW<[GenericWriteJump], (instrs JR_HB64_R6, TAILCALL64R6REG,
                                  TAILCALLHB64R6REG, PseudoIndirectBranch64R6,
                                  PseudoIndrectHazardBranch64R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 431-432
```tablegen
// COP0 Pipeline
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 434-434
```tablegen
def GenericCOP0 : ProcResource<1> { let BufferSize = 1; }
```
- EN: Defines TableGen record `GenericCOP0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericCOP0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 436-444
```tablegen
def GenericIssueCOP0 : ProcResource<1> { let Super = GenericCOP0; }
def GenericWriteCOP0TLB : SchedWriteRes<[GenericIssueCOP0]> { let Latency = 4; }
def GenericWriteCOP0 : SchedWriteRes<[GenericIssueCOP0]> { let Latency = 3; }
def GenericReadCOP0 : SchedWriteRes<[GenericIssueCOP0]> { let Latency = 2; }
def GenericReadWritePGPR : SchedWriteRes<[GenericIssueCOP0]>;
def GenericReadWriteCOP0Long : SchedWriteRes<[GenericIssueCOP0]> {
  let Latency = 5;
}
def GenericWriteCOP0Short : SchedWriteRes<[GenericIssueCOP0]>;
```
- EN: Defines TableGen record `GenericIssueCOP0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericIssueCOP0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 446-447
```tablegen
def : InstRW<[GenericWriteCOP0TLB], (instrs TLBP, TLBR, TLBWI, TLBWR)>;
def : InstRW<[GenericWriteCOP0TLB], (instrs TLBINV, TLBINVF)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 449-450
```tablegen
def : InstRW<[GenericReadCOP0], (instrs MFC0)>;
def : InstRW<[GenericWriteCOP0], (instrs MTC0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 452-452
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs EVP, DVP)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 454-454
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs DI, EI)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 456-456
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs EHB, PAUSE, WAIT)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 458-459
```tablegen
// microMIPS
// =========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 461-462
```tablegen
def : InstRW<[GenericWriteCOP0TLB], (instrs TLBP_MM, TLBR_MM, TLBWI_MM,
                                     TLBWR_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 464-464
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs DI_MM, EI_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 466-466
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs EHB_MM, PAUSE_MM, WAIT_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 469-470
```tablegen
// microMIPS32R6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 472-472
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs RDPGPR_MMR6, WRPGPR_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 474-474
```tablegen
def : InstRW<[GenericWriteCOP0TLB], (instrs TLBINV_MMR6, TLBINVF_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 476-477
```tablegen
def : InstRW<[GenericReadCOP0], (instrs MFHC0_MMR6, MFC0_MMR6, MFHC2_MMR6,
                                 MFC2_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 479-480
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs MTHC0_MMR6, MTC0_MMR6, MTHC2_MMR6,
                                  MTC2_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 482-482
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs EVP_MMR6, DVP_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 484-484
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs DI_MMR6, EI_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 486-486
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs EHB_MMR6, PAUSE_MMR6, WAIT_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 488-489
```tablegen
// MIPS64
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 491-491
```tablegen
def : InstRW<[GenericReadCOP0], (instrs DMFC0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 493-493
```tablegen
def : InstRW<[GenericWriteCOP0], (instrs DMTC0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 496-497
```tablegen
def GenericCOP2 : ProcResource<1> { let BufferSize = 1; }
def GenericWriteCOPOther : SchedWriteRes<[GenericCOP2]>;
```
- EN: Defines TableGen record `GenericCOP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericCOP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 499-499
```tablegen
def : InstRW<[GenericWriteCOPOther], (instrs MFC2, MTC2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 501-501
```tablegen
def : InstRW<[GenericWriteCOPOther], (instrs DMFC2, DMTC2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 503-504
```tablegen
// microMIPS32R6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 506-508
```tablegen
// The latency and repeat rate of these instructions are implementation
// dependant.
def : InstRW<[GenericWriteMove], (instrs CFC2_MM, CTC2_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 511-512
```tablegen
// MIPS MT ASE - hasMT
// ====================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 514-515
```tablegen
def : InstRW<[GenericWriteMove], (instrs DMT, DVPE, EMT, EVPE, MFTR,
                                  MTTR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 517-517
```tablegen
def : InstRW<[GenericReadWriteCOP0Long], (instrs YIELD)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 519-519
```tablegen
def : InstRW<[GenericWriteCOP0Short], (instrs FORK)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 521-522
```tablegen
// MIPS Virtualization ASE
// =======================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 524-526
```tablegen
def : InstRW<[GenericWriteCOP0Short], (instrs HYPCALL, TLBGINV, TLBGINVF, TLBGP,
                                       TLBGR, TLBGWI, TLBGWR, MFGC0, MFHGC0,
                                       MTGC0, MTHGC0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 528-529
```tablegen
// MIPS64 Virtualization ASE
// =========================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 531-531
```tablegen
def : InstRW<[GenericWriteCOP0Short], (instrs DMFGC0, DMTGC0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 533-534
```tablegen
// microMIPS virtualization ASE
// ============================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 536-539
```tablegen
def : InstRW<[GenericWriteCOP0Short], (instrs HYPCALL_MM, TLBGINVF_MM,
                                       TLBGINV_MM, TLBGP_MM, TLBGR_MM,
                                       TLBGWI_MM, TLBGWR_MM, MFGC0_MM,
                                       MFHGC0_MM, MTGC0_MM, MTHGC0_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 541-542
```tablegen
// LDST Pipeline
// -------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 544-546
```tablegen
def GenericWriteLoad : SchedWriteRes<[GenericIssueLDST]> {
  let Latency = 2;
}
```
- EN: Defines TableGen record `GenericWriteLoad` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteLoad`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 548-550
```tablegen
def GenericWritePref : SchedWriteRes<[GenericIssueLDST]>;
def GenericWriteSync : SchedWriteRes<[GenericIssueLDST]>;
def GenericWriteCache : SchedWriteRes<[GenericIssueLDST]> { let Latency = 5; }
```
- EN: Defines TableGen record `GenericWritePref` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWritePref`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 552-553
```tablegen
def GenericWriteStore : SchedWriteRes<[GenericIssueLDST]>;
def GenericWriteStoreSC : SchedWriteRes<[GenericIssueLDST]> { let Latency = 2; }
```
- EN: Defines TableGen record `GenericWriteStore` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteStore`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 555-557
```tablegen
def GenericWriteGPRFromBypass : SchedWriteRes<[GenericIssueLDST]> {
  let Latency = 2;
}
```
- EN: Defines TableGen record `GenericWriteGPRFromBypass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteGPRFromBypass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 559-562
```tablegen
def GenericWriteStoreFromOtherUnits : SchedWriteRes<[GenericIssueLDST]>;
def GenericWriteLoadToOtherUnits : SchedWriteRes<[GenericIssueLDST]> {
  let Latency = 0;
}
```
- EN: Defines TableGen record `GenericWriteStoreFromOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteStoreFromOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 564-566
```tablegen
// l[bhw], l[bh]u, ll
def : InstRW<[GenericWriteLoad], (instrs LB, LBu, LH, LHu, LW, LL,
                                  LWC2, LWC3, LDC2, LDC3)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 568-569
```tablegen
// lw[lr]
def : InstRW<[GenericWriteLoad], (instrs LWL, LWR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 571-573
```tablegen
// s[bhw], sc, s[dw]c[23]
def : InstRW<[GenericWriteStore], (instrs SB, SH, SW, SWC2, SWC3,
                                   SDC2, SDC3)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 575-576
```tablegen
// PreMIPSR6 sw[lr]
def : InstRW<[GenericWriteStore], (instrs SWL, SWR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 578-578
```tablegen
def : InstRW<[GenericWriteStoreSC], (instrs SC, SC_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 580-583
```tablegen
// pref
def : InstRW<[GenericWritePref], (instrs PREF)>;
// cache
def : InstRW<[GenericWriteCache], (instrs CACHE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 585-586
```tablegen
// sync
def : InstRW<[GenericWriteSync], (instrs SYNC, SYNCI)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 588-589
```tablegen
// MIPSR6
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 591-591
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LDC2_R6, LL_R6, LWC2_R6, LWPC)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 593-593
```tablegen
def : InstRW<[GenericWriteStore], (instrs SWC2_R6,  SDC2_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 595-595
```tablegen
def : InstRW<[GenericWriteStoreSC], (instrs SC_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 597-597
```tablegen
def : InstRW<[GenericWritePref], (instrs PREF_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 599-599
```tablegen
def : InstRW<[GenericWriteCache], (instrs CACHE_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 601-601
```tablegen
def : InstRW<[GenericWriteSync], (instrs GINVI, GINVT)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 603-604
```tablegen
// MIPS32 EVA
// ==========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 606-607
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LBE, LBuE, LHE, LHuE, LWE,
                                  LLE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 609-609
```tablegen
def : InstRW<[GenericWriteStore], (instrs SBE, SHE, SWE, SCE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 611-611
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LWLE, LWRE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 613-613
```tablegen
def : InstRW<[GenericWriteStore], (instrs SWLE, SWRE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 615-615
```tablegen
def : InstRW<[GenericWritePref], (instrs PREFE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 617-617
```tablegen
def : InstRW<[GenericWriteCache], (instrs CACHEE)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 619-620
```tablegen
// microMIPS EVA ASE - InMicroMipsMode, hasEVA
// ===========================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 622-623
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LBE_MM, LBuE_MM, LHE_MM, LHuE_MM,
                                  LWE_MM, LWLE_MM, LWRE_MM, LLE_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 625-626
```tablegen
def : InstRW<[GenericWriteStore], (instrs SBE_MM, SB_MM, SHE_MM, SWE_MM,
                                   SWLE_MM, SWRE_MM, SCE_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 628-629
```tablegen
def : InstRW<[GenericWritePref], (instrs PREFE_MM)>;
def : InstRW<[GenericWriteCache], (instrs CACHEE_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 632-633
```tablegen
// MIPS16e
// =======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 635-639
```tablegen
def : InstRW<[GenericWriteLoad], (instrs Restore16, RestoreX16,
                                  LbRxRyOffMemX16,
                                  LbuRxRyOffMemX16, LhRxRyOffMemX16,
                                  LhuRxRyOffMemX16, LwRxRyOffMemX16,
                                  LwRxSpImmX16, LwRxPcTcp16, LwRxPcTcpX16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 641-643
```tablegen
def : InstRW<[GenericWriteStore], (instrs Save16, SaveX16, SbRxRyOffMemX16,
                                   ShRxRyOffMemX16, SwRxRyOffMemX16,
                                   SwRxSpImmX16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 645-646
```tablegen
// microMIPS
// =========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 648-651
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LBU16_MM, LB_MM, LBu_MM, LHU16_MM,
                                  LH_MM, LHu_MM, LL_MM, LW16_MM, LWGP_MM,
                                  LWL_MM, LWM16_MM, LWM32_MM, LWP_MM, LWR_MM,
                                  LWSP_MM, LWU_MM, LWXS_MM, LW_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 653-655
```tablegen
def : InstRW<[GenericWriteStore], (instrs SB16_MM, SC_MM, SH16_MM, SH_MM,
                                   SW16_MM, SWL_MM, SWM16_MM, SWM32_MM, SWM_MM,
                                   SWP_MM, SWR_MM, SWSP_MM, SW_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 658-658
```tablegen
def : InstRW<[GenericWritePref], (instrs PREF_MM, PREFX_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 660-660
```tablegen
def : InstRW<[GenericWriteCache], (instrs CACHE_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 662-663
```tablegen
def : InstRW<[GenericWriteSync], (instrs SYNC_MM, SYNCI_MM)>;
def : InstRW<[GenericWriteSync], (instrs GINVI_MMR6, GINVT_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 665-666
```tablegen
// microMIPS32r6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 668-669
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LBU_MMR6, LB_MMR6, LDC2_MMR6, LL_MMR6,
                                  LWM16_MMR6, LWC2_MMR6, LWPC_MMR6, LW_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 671-673
```tablegen
def : InstRW<[GenericWriteStore], (instrs SB16_MMR6, SB_MMR6, SDC2_MMR6,
                                   SH16_MMR6, SH_MMR6, SW16_MMR6, SWC2_MMR6,
                                   SWM16_MMR6, SWSP_MMR6, SW_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 675-675
```tablegen
def : InstRW<[GenericWriteSync], (instrs SYNC_MMR6, SYNCI_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 677-677
```tablegen
def : InstRW<[GenericWritePref], (instrs PREF_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 679-679
```tablegen
def : InstRW<[GenericWriteCache], (instrs CACHE_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 681-682
```tablegen
// MIPS64
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 684-685
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LD, LL64, LLD, LWu, LB64, LBu64,
                                  LH64, LHu64, LW64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 687-688
```tablegen
// l[dw][lr]
def : InstRW<[GenericWriteLoad], (instrs LWL64, LWR64, LDL, LDR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 690-691
```tablegen
def : InstRW<[GenericWriteStore], (instrs SD, SC64, SCD, SB64, SH64, SW64,
                                   SWL64, SWR64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 693-693
```tablegen
def : InstRW<[GenericWriteStore], (instrs SDL, SDR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 695-696
```tablegen
// MIPS64R6
// ========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 698-698
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LWUPC, LDPC)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 700-700
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LLD_R6, LL64_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 702-702
```tablegen
def : InstRW<[GenericWriteStoreSC], (instrs SC64_R6, SCD_R6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 704-705
```tablegen
// MIPSR6 CRC ASE - hasCRC
// =======================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 707-708
```tablegen
def : InstRW<[GenericWriteALU], (instrs CRC32B, CRC32H, CRC32W, CRC32CB,
                                 CRC32CH, CRC32CW)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 710-711
```tablegen
// MIPS64R6 CRC ASE - hasCRC
// -------------------------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 713-713
```tablegen
def : InstRW<[GenericWriteALU], (instrs CRC32D, CRC32CD)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 716-717
```tablegen
// Cavium Networks MIPS (cnMIPS) - Octeon, HasCnMips
// =================================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 719-724
```tablegen
def : InstRW<[GenericWriteALU], (instrs BADDu, BBIT0, BBIT032, BBIT1, BBIT132,
                                 CINS, CINS32, CINS64_32, CINS_i32,
                                 DMFC2_OCTEON, DMTC2_OCTEON, DPOP, EXTS,
                                 EXTS32, MTM0, MTM1, MTM2, MTP0, MTP1, MTP2,
                                 POP, SEQ, SEQi, SNE, SNEi,
                                 V3MULU, VMM0, VMULU)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 726-726
```tablegen
def : InstRW<[GenericWriteMDUtoGPR], (instrs DMUL)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 728-729
```tablegen
// Cavium Networks MIPS (cnMIPSP) - Octeon+, HasCnMipsP
// =================================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 731-731
```tablegen
def : InstRW<[GenericWriteALU], (instrs SAA, SAAD)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 733-734
```tablegen
// FPU Pipelines
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 736-742
```tablegen
def GenericFPQ : ProcResource<1> { let BufferSize = 1; }
def GenericIssueFPUS : ProcResource<1> { let Super = GenericFPQ; }
def GenericIssueFPUL : ProcResource<1> { let Super = GenericFPQ; }
def GenericIssueFPULoad : ProcResource<1> { let Super = GenericFPQ; }
def GenericIssueFPUStore : ProcResource<1> { let Super = GenericFPQ; }
def GenericIssueFPUMove : ProcResource<1> { let Super = GenericFPQ; }
def GenericFPUDivSqrt : ProcResource<1> { let Super = GenericFPQ; }
```
- EN: Defines TableGen record `GenericFPQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericFPQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 744-745
```tablegen
// The floating point compare of the 24k series including interAptiv has a
// listed latency of 1-2. Using the higher latency here.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 747-764
```tablegen
def GenericWriteFPUCmp : SchedWriteRes<[GenericIssueFPUS]> { let Latency = 2; }
def GenericWriteFPUS : SchedWriteRes<[GenericIssueFPUS]> { let Latency = 4; }
def GenericWriteFPUL : SchedWriteRes<[GenericIssueFPUL]> { let Latency = 5; }
def GenericWriteFPUStore : SchedWriteRes<[GenericIssueFPUStore]> { let
  Latency = 1;
}
def GenericWriteFPULoad : SchedWriteRes<[GenericIssueFPULoad]> {
  let Latency = 2;
}
def GenericWriteFPUMoveFP : SchedWriteRes<[GenericIssueFPUMove]> {
  let Latency = 4;
}
def GenericWriteFPUMoveGPRFPU : SchedWriteRes<[GenericIssueFPUMove]> {
  let Latency = 2;
}
def GenericWriteFPUDivS : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 17;
  let ReleaseAtCycles = [ 14 ];
```
- EN: Defines TableGen record `GenericWriteFPUCmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteFPUCmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 765-782
```tablegen
}
def GenericWriteFPUDivD : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 32;
  let ReleaseAtCycles = [ 29 ];
}
def GenericWriteFPURcpS : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 13;
  let ReleaseAtCycles = [ 10 ];
}
def GenericWriteFPURcpD : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 25;
  let ReleaseAtCycles = [ 21 ];
}
def GenericWriteFPURsqrtS : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 17;
  let ReleaseAtCycles = [ 14 ];
}
def GenericWriteFPURsqrtD : SchedWriteRes<[GenericFPUDivSqrt]> {
```
- EN: Defines TableGen record `GenericWriteFPUDivD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteFPUDivD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 783-793
```tablegen
  let Latency = 32;
  let ReleaseAtCycles = [ 29 ];
}
def GenericWriteFPUSqrtS : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 17;
  let ReleaseAtCycles = [ 14 ];
}
def GenericWriteFPUSqrtD : SchedWriteRes<[GenericFPUDivSqrt]> {
  let Latency = 29;
  let ReleaseAtCycles = [ 29 ];
}
```
- EN: Defines TableGen record `GenericWriteFPUSqrtS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteFPUSqrtS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 795-800
```tablegen
// Floating point compare and branch
// ---------------------------------
//
// c.<cc>.[ds], bc1[tf], bc1[tf]l
def : InstRW<[GenericWriteFPUCmp], (instrs FCMP_D32, FCMP_D64, FCMP_S32, BC1F,
                                    BC1T, BC1FL, BC1TL)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 802-802
```tablegen
def : InstRW<[GenericWriteFPUCmp], (instregex "C_[A-Z]+_(S|D32|D64)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 804-808
```tablegen
// Short Pipe
// ----------
//
// abs.[ds], abs.ps, add.[ds], neg.[ds], neg.ps, madd.s, msub.s, nmadd,s
// nmsub.s, sub.[ds], mul.s
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 810-813
```tablegen
def : InstRW<[GenericWriteFPUS], (instrs FABS_S, FABS_D32, FABS_D64, FADD_D32,
                                  FADD_D64, FADD_S, MADD_S, MSUB_S, FMUL_S,
                                  FNEG_S, FNEG_D32, FNEG_D64, NMADD_S, NMSUB_S,
                                  FSUB_S, FSUB_D32, FSUB_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 815-820
```tablegen
// Long Pipe
// ----------
//
// nmadd.d, nmsub.d, mul.[ds], mul.ps, ceil.[wl].[sd], cvt.d.[sw], cvt.s.[dw],
// cvt.w.[sd], cvt.[sw].ps, trunc.w.[ds], trunc.w.ps, floor.[ds],
// round.[lw].[ds], floor.[lw].ds
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 822-839
```tablegen
// madd.d, msub.dm mul.d, mul.ps, nmadd.d, nmsub.d, ceil.[wl].[sd], cvt.d.[sw],
// cvt.s.[dw], cvt.w.[sd], cvt.[sw].ps, round.[lw].[ds], floor.[lw].ds,
// trunc.w.[ds], trunc.w.ps,
def : InstRW<[GenericWriteFPUL], (instrs ADDR_PS64,
                                  CEIL_L_D64, CEIL_L_S, CEIL_W_D32,
                                  CEIL_W_D64, CEIL_W_S, CVT_D32_S, CVT_D32_W,
                                  CVT_D64_L, CVT_D64_S, CVT_D64_W, CVT_L_D64,
                                  CVT_L_S, CVT_S_D32, CVT_S_D64, CVT_S_L,
                                  CVT_S_W, CVT_W_D32, CVT_W_D64, CVT_W_S,
                                  CVT_PS_S64, CVT_S_PL64, CVT_S_PU64,
                                  CVT_PS_PW64, CVT_PW_PS64, FADD_PS64,
                                  FLOOR_L_D64, FLOOR_L_S, FLOOR_W_D32,
                                  FLOOR_W_D64, FLOOR_W_S, FMUL_D32, FMUL_D64,
                                  FMUL_PS64, FSUB_PS64, MADD_D32, MADD_D64,
                                  MSUB_D32, MSUB_D64, MULR_PS64,
                                  NMADD_D32, NMADD_D64, NMSUB_D32, NMSUB_D64,
                                  PLL_PS64, PLU_PS64, PUL_PS64, PUU_PS64,
                                  ROUND_L_D64, ROUND_L_S, ROUND_W_D32,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 840-843
```tablegen
                                  ROUND_W_D64, ROUND_W_S, TRUNC_L_D64,
                                  TRUNC_L_S, TRUNC_W_D32, TRUNC_W_D64,
                                  TRUNC_W_S, PseudoTRUNC_W_D,
                                  PseudoTRUNC_W_D32, PseudoTRUNC_W_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 845-848
```tablegen
// Pseudo convert instruction
def : InstRW<[GenericWriteFPUL], (instrs PseudoCVT_D32_W, PseudoCVT_D64_L,
                                  PseudoCVT_D64_W, PseudoCVT_S_L,
                                  PseudoCVT_S_W)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 850-852
```tablegen
// div.[ds], div.ps
def : InstRW<[GenericWriteFPUDivS], (instrs FDIV_S)>;
def : InstRW<[GenericWriteFPUDivD], (instrs FDIV_D32, FDIV_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 854-856
```tablegen
// sqrt.[ds], sqrt.ps
def : InstRW<[GenericWriteFPUSqrtS], (instrs FSQRT_S)>;
def : InstRW<[GenericWriteFPUSqrtD], (instrs FSQRT_D32, FSQRT_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 858-861
```tablegen
// rsqrt.[ds], recip.[ds]
def : InstRW<[GenericWriteFPURcpS], (instrs RECIP_S, RSQRT_S)>;
def : InstRW<[GenericWriteFPURcpD], (instrs RECIP_D32, RECIP_D64,
                                     RSQRT_D32, RSQRT_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 864-865
```tablegen
// Load Pipe
// ---------
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 867-873
```tablegen
// ctc1, mtc1, mthc1, cfc1, mfc1, mfhc1
def : InstRW<[GenericWriteFPUMoveGPRFPU], (instrs BuildPairF64,
                                           BuildPairF64_64, ExtractElementF64,
                                           ExtractElementF64_64, CFC1, CTC1,
                                           MFC1, MFC1_D64, MFHC1_D32,
                                           MFHC1_D64, MTC1, MTC1_D64,
                                           MTHC1_D32, MTHC1_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 875-877
```tablegen
// swc1, swxc1
def : InstRW<[GenericWriteFPUStore], (instrs SDC1, SDC164, SDXC1, SDXC164,
                                      SUXC1, SUXC164, SWC1, SWXC1)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 879-879
```tablegen
def : InstRW<[GenericWriteFPUMoveFP], (instrs FMOV_D32, FMOV_D64, FMOV_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 882-887
```tablegen
// movn.[ds], movz.[ds]
def : InstRW<[GenericWriteFPUMoveFP], (instrs MOVF_I, MOVF_D32, MOVF_D64,
                                       MOVF_S, MOVT_I, MOVT_D32, MOVT_D64,
                                       MOVT_S, MOVN_I_D32, MOVN_I_D64,
                                       MOVN_I_S, MOVZ_I_D32, MOVZ_I_D64,
                                       MOVZ_I_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 889-891
```tablegen
def : InstRW<[GenericWriteFPUMoveFP], (instrs MOVT_I64, MOVF_I64, MOVZ_I64_S,
                                       MOVN_I64_D64, MOVN_I64_S,
                                       MOVZ_I64_D64)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 893-895
```tablegen
// l[dw]x?c1
def : InstRW<[GenericWriteFPULoad], (instrs LDC1, LDC164, LDXC1, LDXC164,
                                     LUXC1, LUXC164, LWC1, LWXC1)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 897-898
```tablegen
// MIPSR6
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 900-903
```tablegen
// sel(eq|ne).[ds], max.[ds], maxa.[ds], min.[ds], mina.[ds], class.[ds]
def : InstRW<[GenericWriteFPUS], (instrs SELEQZ_S, SELNEZ_S, SELEQZ_D, SELNEZ_D,
                                  MAX_S, MAX_D, MAXA_S, MAXA_D, MIN_S, MIN_D,
                                  MINA_S, MINA_D, CLASS_S, CLASS_D)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 905-905
```tablegen
def : InstRW<[GenericWriteFPUL], (instrs RINT_S, RINT_D)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 907-907
```tablegen
def : InstRW<[GenericWriteFPUCmp], (instrs BC1EQZ, BC1NEZ, SEL_D, SEL_S)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 909-909
```tablegen
def : InstRW<[GenericWriteFPUS], (instrs MADDF_S, MSUBF_S, MADDF_D, MSUBF_D)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 912-913
```tablegen
// microMIPS
// =========
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 915-918
```tablegen
def : InstRW<[GenericWriteFPUMoveFP], (instrs MOVF_D32_MM, MOVF_S_MM,
                                       MOVN_I_D32_MM, MOVN_I_S_MM,
                                       MOVT_D32_MM, MOVT_S_MM, MOVZ_I_D32_MM,
                                       MOVZ_I_S_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 921-930
```tablegen
//  cvt.?.?, ceil.?, floor.?, round.?, trunc.? (n)madd.? (n)msub.?
def : InstRW<[GenericWriteFPUL], (instrs CVT_D32_S_MM, CVT_D32_W_MM,
                                  CVT_D64_S_MM, CVT_D64_W_MM, CVT_L_D64_MM,
                                  CVT_L_S_MM, CVT_S_D32_MM, CVT_S_D64_MM,
                                  CVT_S_W_MM, CVT_W_D32_MM, CVT_W_D64_MM,
                                  CVT_W_S_MM, CEIL_W_MM, CEIL_W_S_MM,
                                  FLOOR_W_MM, FLOOR_W_S_MM, NMADD_S_MM,
                                  NMADD_D32_MM, NMSUB_S_MM, NMSUB_D32_MM,
                                  MADD_S_MM, MADD_D32_MM, ROUND_W_MM,
                                  ROUND_W_S_MM, TRUNC_W_MM, TRUNC_W_S_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 932-936
```tablegen
def : InstRW<[GenericWriteFPUCmp], (instregex "^C_[A-Z]_(S|D32|D64)_MM$")>;
def : InstRW<[GenericWriteFPUCmp], (instregex "^C_[A-Z][A-Z]_(S|D32|D64)_MM$")>;
def : InstRW<[GenericWriteFPUCmp], (instregex "^C_[A-Z][A-Z][A-Z]_(S|D32|D64)_MM$")>;
def : InstRW<[GenericWriteFPUCmp], (instregex "^C_NGLE_(S|D32|D64)_MM$")>;
def : InstRW<[GenericWriteFPUCmp], (instrs FCMP_S32_MM, FCMP_D32_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 938-940
```tablegen
def : InstRW<[GenericWriteFPUS], (instrs MFC1_MM, MFHC1_D32_MM, MFHC1_D64_MM,
                                  MTC1_MM, MTC1_D64_MM,
                                  MTHC1_D32_MM, MTHC1_D64_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 942-948
```tablegen
def : InstRW<[GenericWriteFPUS], (instrs FABS_D32_MM, FABS_D64_MM, FABS_S_MM,
                                  FNEG_D32_MM, FNEG_D64_MM, FNEG_S_MM,
                                  FADD_D32_MM, FADD_D64_MM, FADD_S_MM,
                                  FMOV_D32_MM, FMOV_D64_MM, FMOV_S_MM,
                                  FMUL_D32_MM, FMUL_D64_MM, FMUL_S_MM,
                                  FSUB_D32_MM, FSUB_D64_MM, FSUB_S_MM,
                                  MSUB_S_MM, MSUB_D32_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 950-951
```tablegen
def : InstRW<[GenericWriteFPUDivS], (instrs FDIV_S_MM)>;
def : InstRW<[GenericWriteFPUDivD], (instrs FDIV_D32_MM, FDIV_D64_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 953-954
```tablegen
def : InstRW<[GenericWriteFPUSqrtS], (instrs FSQRT_S_MM)>;
def : InstRW<[GenericWriteFPUSqrtD], (instrs FSQRT_D32_MM, FSQRT_D64_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 956-958
```tablegen
def : InstRW<[GenericWriteFPURcpS], (instrs RECIP_S_MM, RSQRT_S_MM)>;
def : InstRW<[GenericWriteFPURcpD], (instrs RECIP_D32_MM, RECIP_D64_MM,
                                     RSQRT_D32_MM, RSQRT_D64_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 960-961
```tablegen
def : InstRW<[GenericWriteFPUStore], (instrs SDC1_MM_D32, SDC1_MM_D64, SWC1_MM,
                                      SUXC1_MM, SWXC1_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 963-963
```tablegen
def : InstRW<[GenericWriteFPUMoveGPRFPU], (instrs CFC1_MM, CTC1_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 965-966
```tablegen
def : InstRW<[GenericWriteFPULoad], (instrs LDC1_MM_D32, LDC1_MM_D64, LUXC1_MM,
                                     LWC1_MM, LWXC1_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 968-969
```tablegen
// microMIPS32r6
// =============
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 971-971
```tablegen
def : InstRW<[GenericWriteFPUS], (instrs FNEG_S_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 973-977
```tablegen
def : InstRW<[GenericWriteFPUCmp], (instregex "CMP_[A-Z][A-Z]_(S|D)_MMR6")>;
def : InstRW<[GenericWriteFPUCmp],
             (instregex "CMP_[A-Z][A-Z][A-Z]_(S|D)_MMR6")>;
def : InstRW<[GenericWriteFPUCmp],
             (instregex "CMP_[A-Z][A-Z][A-Z][A-Z]_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 979-980
```tablegen
def : InstRW<[GenericWriteFPUL],
             (instregex "CVT_(L|D|S|W)_(L|D|S|L|W)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 982-983
```tablegen
def : InstRW<[GenericWriteFPUL],
             (instregex "TRUNC_(L|W)_(D|S)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 985-986
```tablegen
def : InstRW<[GenericWriteFPUL],
             (instregex "ROUND_(L|W)_(D|S)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 988-989
```tablegen
def : InstRW<[GenericWriteFPUL],
             (instregex "FLOOR_(L|W)_(D|S)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 991-992
```tablegen
def : InstRW<[GenericWriteFPUL],
             (instregex "CEIL_(L|W)_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 994-996
```tablegen
def : InstRW<[GenericWriteFPUS],
             (instrs MFC1_MMR6, MTC1_MMR6, CLASS_S_MMR6, CLASS_D_MMR6,
              FADD_S_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 998-998
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "M(IN|AX)_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1000-1000
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "M(IN|AX)A_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1002-1002
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "SEL(EQ|NE)Z_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1004-1004
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "SEL_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1006-1006
```tablegen
def : InstRW<[GenericWriteFPUL], (instrs RINT_S_MMR6, RINT_D_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1008-1008
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "M(ADD|SUB)F_(S|D)_MMR6")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1010-1011
```tablegen
def : InstRW<[GenericWriteFPUS], (instrs FMOV_S_MMR6, FMUL_S_MMR6,
                                  FSUB_S_MMR6, FMOV_D_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1013-1013
```tablegen
def : InstRW<[GenericWriteFPUL], (instrs FDIV_S_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1015-1015
```tablegen
def : InstRW<[GenericWriteFPUStore], (instrs SDC1_D64_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1017-1017
```tablegen
def : InstRW<[GenericWriteFPULoad], (instrs LDC1_D64_MMR6)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1019-1020
```tablegen
// MIPS64
// ======
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1022-1022
```tablegen
def : InstRW<[GenericWriteFPUMoveGPRFPU], (instrs DMFC1, DMTC1)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1024-1025
```tablegen
// MIPS DSP ASE, HasDSP
// ====================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1027-1027
```tablegen
def : InstRW<[GenericWriteStore], (instrs SWDSP)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1029-1029
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LWDSP)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1031-1031
```tablegen
def : InstRW<[GenericWriteMove], (instrs PseudoMTLOHI_DSP)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1033-1039
```tablegen
def GenericDSP : ProcResource<1> { let BufferSize = 1; }
def GenericDSPShort : SchedWriteRes<[GenericDSP]> { let Latency = 2; }
def GenericDSPLong : SchedWriteRes<[GenericDSP]> { let Latency = 6; }
def GenericDSPBypass : SchedWriteRes<[GenericDSP]> { let Latency = 1; }
def GenericDSPMTHILO : SchedWriteRes<[GenericDSP]> { let Latency = 5; }
def GenericDSPLoad : SchedWriteRes<[GenericDSP]> { let Latency = 4; }
def GenericDSPMTHLIP : SchedWriteRes<[GenericDSP]> { let Latency = 5; }
```
- EN: Defines TableGen record `GenericDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1041-1049
```tablegen
def : InstRW<[GenericDSPLong], (instregex "^EXTRV_RS_W$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTRV_R_W$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTRV_S_H$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTRV_W$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTR_RS_W$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTR_R_W$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTR_S_H$")>;
def : InstRW<[GenericDSPLong], (instregex "^EXTR_W$")>;
def : InstRW<[GenericDSPLong], (instregex "^INSV$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1051-1053
```tablegen
def : InstRW<[GenericDSPMTHLIP], (instregex "^MTHLIP$")>;
def : InstRW<[GenericDSPMTHILO], (instregex "^MTHI_DSP$")>;
def : InstRW<[GenericDSPMTHILO], (instregex "^MTLO_DSP$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1055-1072
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^ABSQ_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^ABSQ_S_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQ_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQ_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQ_S_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDSC$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_S_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDWC$")>;
def : InstRW<[GenericDSPShort], (instregex "^BITREV$")>;
def : InstRW<[GenericDSPShort], (instregex "^BPOSGE32$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGU_EQ_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGU_LE_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGU_LT_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPU_EQ_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPU_LE_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPU_LT_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMP_EQ_PH$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1073-1090
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^CMP_LE_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMP_LT_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQ_SA_L_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQ_S_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAU_H_QBL$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAU_H_QBR$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQ_SA_L_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQ_S_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSU_H_QBL$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSU_H_QBR$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTPDPV$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTPDP$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTPV$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTP$")>;
def : InstRW<[GenericDSPShort], (instregex "^LBUX$")>;
def : InstRW<[GenericDSPShort], (instregex "^LHX$")>;
def : InstRW<[GenericDSPShort], (instregex "^LWX$")>;
def : InstRW<[GenericDSPShort], (instregex "^MADDU_DSP$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1091-1108
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^MADD_DSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_SA_W_PHL$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_SA_W_PHR$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_S_W_PHL$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_S_W_PHR$")>;
def : InstRW<[GenericDSPShort], (instregex "^MFHI_DSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^MFLO_DSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^MODSUB$")>;
def : InstRW<[GenericDSPShort], (instregex "^MSUBU_DSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^MSUB_DSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEQ_S_W_PHL$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEQ_S_W_PHR$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEU_S_PH_QBL$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEU_S_PH_QBR$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_RS_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULSAQ_S_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULTU_DSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULT_DSP$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1109-1126
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^PACKRL_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^PICK_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^PICK_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBLA$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBL$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBRA$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBR$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQ_W_PHL$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQ_W_PHR$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBLA$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBL$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBRA$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBR$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQU_S_QB_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQ_PH_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQ_QB_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQ_RS_PH_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^RADDU_W_QB$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1127-1144
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^RDDSP$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPLV_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPLV_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPL_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPL_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHILOV$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHILO$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_S_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_S_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_R_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_R_W$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1145-1155
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^SHRA_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_R_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_R_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRLV_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRL_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQ_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQ_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQ_S_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_S_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^WRDSP$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1157-1160
```tablegen
def : InstRW<[GenericDSPShort],
             (instregex "^Pseudo(CMP|CMPU)_(EQ|LE|LT)_(PH|QB)$")>;
def : InstRW<[GenericDSPShort],
						 (instregex "^PseudoPICK_(PH|QB)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1162-1163
```tablegen
// MIPS DSP R2 - hasDSP, HasDSPR2, InMicroMips
// ===========================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1165-1182
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^ABSQ_S_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_R_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_R_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDUH_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDUH_R_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^APPEND$")>;
def : InstRW<[GenericDSPShort], (instregex "^BALIGN$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGDU_EQ_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGDU_LE_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGDU_LT_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPA_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQX_SA_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQX_S_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAX_W_PH$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1183-1200
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^DPS_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQX_S_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQX_SA_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSX_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^MUL_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^MUL_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_RS_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_S_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULSA_W_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECR_QB_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECR_SRA_PH_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECR_SRA_R_PH_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^PREPEND$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_R_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_R_QB$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1201-1210
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^SHRL_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRLV_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_R_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_R_W$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_S_PH$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBUH_QB$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBUH_R_QB$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1212-1213
```tablegen
// microMIPS DSP R1 - HasDSP, InMicroMips
// ======================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1215-1215
```tablegen
def : InstRW<[GenericWriteLoad], (instrs LWDSP_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1217-1217
```tablegen
def : InstRW<[GenericWriteStore], (instrs SWDSP_MM)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1219-1236
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^ABSQ_S_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ABSQ_S_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQ_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQ_S_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQ_S_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDSC_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_S_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDWC_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^BITREV_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^BPOSGE32_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGU_EQ_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGU_LE_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGU_LT_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPU_EQ_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPU_LE_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPU_LT_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMP_EQ_PH_MM$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1237-1254
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^CMP_LE_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMP_LT_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQ_SA_L_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQ_S_W_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAU_H_QBL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAU_H_QBR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQ_SA_L_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQ_S_W_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSU_H_QBL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSU_H_QBR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTPDPV_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTPDP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTPV_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTRV_RS_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTRV_R_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTRV_S_H_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTRV_W_MM$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1255-1272
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^EXTR_RS_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTR_R_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTR_S_H_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^EXTR_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^INSV_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^LBUX_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^LHX_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^LWX_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MADDU_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MADD_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_SA_W_PHL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_SA_W_PHR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_S_W_PHL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MAQ_S_W_PHR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MFHI_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MFLO_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MODSUB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MOVEP_MMR6$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1273-1290
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^MOVN_I_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MOVZ_I_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MSUBU_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MSUB_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MTHI_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MTHLIP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MTLO_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEQ_S_W_PHL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEQ_S_W_PHR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEU_S_PH_QBL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULEU_S_PH_QBR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_RS_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULSAQ_S_W_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULTU_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULT_DSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PACKRL_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PICK_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PICK_QB_MM$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1291-1308
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBLA_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBRA_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQU_PH_QBR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQ_W_PHL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEQ_W_PHR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBLA_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBL_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBRA_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECEU_PH_QBR_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQU_S_QB_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQ_PH_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQ_QB_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECRQ_RS_PH_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^RADDU_W_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^RDDSP_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPLV_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPLV_QB_MM$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1309-1326
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^REPL_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^REPL_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHILOV_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHILO_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_S_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLLV_S_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_S_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHLL_S_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_R_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_R_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_R_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_R_W_MM$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1327-1334
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^SHRLV_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRL_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQ_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQ_S_PH_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQ_S_W_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_S_QB_MM$")>;
def : InstRW<[GenericDSPShort], (instregex "^WRDSP_MM$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1337-1338
```tablegen
// microMIPS DSP R2 - hasDSP, HasDSPR2, InMicroMips
// ================================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1340-1357
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^ABSQ_S_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_R_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_R_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDQH_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDUH_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDUH_R_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^ADDU_S_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^APPEND_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^BALIGN_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGDU_EQ_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGDU_LE_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^CMPGDU_LT_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPA_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQX_SA_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAQX_S_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPAX_W_PH_MMR2$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1358-1375
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^DPS_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQX_S_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSQX_SA_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^DPSX_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^MUL_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^MUL_S_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_RS_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_S_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULQ_S_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^MULSA_W_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECR_QB_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECR_SRA_PH_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^PRECR_SRA_R_PH_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^PREPEND_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRA_R_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRAV_R_QB_MMR2$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1376-1385
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^SHRL_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SHRLV_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_R_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBQH_R_W_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBU_S_PH_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBUH_QB_MMR2$")>;
def : InstRW<[GenericDSPShort], (instregex "^SUBUH_R_QB_MMR2$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1387-1388
```tablegen
// microMIPS DSP R3 - hasDSP, hasDSPR2, hasDSPR3, InMicroMips
// ==========================================================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1390-1390
```tablegen
def : InstRW<[GenericDSPShort], (instregex "^BPOSGE32C_MMR3$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1392-1393
```tablegen
// MIPS MSA ASE - hasMSA
// =====================
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1395-1409
```tablegen
def GenericWriteMSAShortLogic : SchedWriteRes<[GenericIssueFPUS]>;
def GenericWriteMSAShortInt : SchedWriteRes<[GenericIssueFPUS]> {
let Latency = 2;
}
def GenericWriteMoveOtherUnitsToFPU : SchedWriteRes<[GenericIssueFPUS]>;
def GenericWriteMSAOther3 : SchedWriteRes<[GenericIssueFPUS]> {
let Latency = 3;
}
def GenericWriteMSALongInt : SchedWriteRes<[GenericIssueFPUS]> {
let Latency = 5;
}
def GenericWriteFPUDivI : SchedWriteRes<[GenericFPQ]> {
  let Latency = 33;
  let ReleaseAtCycles = [ 33 ];
}
```
- EN: Defines TableGen record `GenericWriteMSAShortLogic` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteMSAShortLogic`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1411-1415
```tablegen
// FPUS is also used in moves from floating point and MSA registers to general
// purpose registers.
def GenericWriteMoveFPUSToOtherUnits : SchedWriteRes<[GenericIssueFPUS]> {
  let Latency = 0;
}
```
- EN: Defines TableGen record `GenericWriteMoveFPUSToOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteMoveFPUSToOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1417-1419
```tablegen
// FPUL is also used in moves from floating point and MSA registers to general
// purpose registers.
def GenericWriteMoveFPULToOtherUnits : SchedWriteRes<[GenericIssueFPUL]>;
```
- EN: Defines TableGen record `GenericWriteMoveFPULToOtherUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericWriteMoveFPULToOtherUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1422-1425
```tablegen
// adds_a.[bhwd], adds_[asu].[bhwd], addvi?.[bhwd], asub_[us].[bhwd],
// aver?_[us].[bhwd]
def : InstRW<[GenericWriteMSAShortInt], (instregex "^ADD_A_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^ADDS_[ASU]_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1427-1431
```tablegen
// TODO: ADDVI_[BHW] might be 1 cycle latency rather than 2. Need to confirm it.
// add.[bhwd], addvi.[bhwd], asub_[us].[bhwd], ave.[bhwd], aver.[bhwd]
def : InstRW<[GenericWriteMSAShortInt], (instregex "^ADDVI?_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^ASUB_[US].[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^AVER?_[US].[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1433-1440
```tablegen
// and.v, andi.b, move.v, ldi.[bhwd], xor.v, nor.v, xori.b, nori.b, lsa
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^MOVE_V$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^LDI_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instrs LSA)>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(AND|OR|[XN]OR)_V$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(AND|OR|[XN]OR)I_B$")>;
def : InstRW<[GenericWriteMSAShortLogic],
             (instregex "^(AND|OR|[XN]OR)_V_[DHW]_PSEUDO$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1442-1455
```tablegen
// vshf.[bhwd], binsl.[bhwd], binsr.[bhwd], insert.[bhwd], sld?.[bhwd],
// bset.[bhwd], bclr.[bhwd], bneg.[bhwd], bsel_v, bseli_b
def : InstRW<[GenericWriteMSAShortInt], (instregex "^VSHF_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(BINSL|BINSLI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(BINSR|BINSRI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^INSERT_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(SLD|SLDI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(BSET|BSETI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(BCLR|BCLRI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(BNEG|BNEGI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(BSEL_V|BSELI_B)$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^BMN*Z.*$")>;
def : InstRW<[GenericWriteMSAShortInt],
             (instregex "^BSEL_(H|W|D|FW|FD)_PSEUDO$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1457-1459
```tablegen
// pcnt.[bhwd], sat_s.[bhwd], sat_u.[bhwd]
def : InstRW<[GenericWriteMSAOther3], (instregex "^PCNT_[BHWD]$")>;
def : InstRW<[GenericWriteMSAOther3], (instregex "^SAT_(S|U)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1461-1463
```tablegen
// bnz.[bhwdv], cfcmsa, ctcmsa
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(BNZ|BZ)_[BHWDV]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^C(F|T)CMSA$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1465-1468
```tablegen
// shf.[bhw], fill[bhwd], splat?.[bhwd]
def : InstRW<[GenericWriteMSAShortInt], (instregex "^SHF_[BHW]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^FILL_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^(SPLAT|SPLATI)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1470-1471
```tablegen
// fexp2_w, fexp2_d
def : InstRW<[GenericWriteFPUS], (instregex "^FEXP2_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1473-1490
```tablegen
// compare, converts, round to int, floating point truncate.
def : InstRW<[GenericWriteFPUS], (instregex "^(CLT|CLTI)_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^(CLE|CLEI)_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^(CEQ|CEQI)_[BHWD]$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_UN_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_UEQ_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_EQ_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_LT_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_ULT_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_LE_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_ULE_(S|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_F_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SAF_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SEQ_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SLE_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SLT_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SUEQ_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SULE_(D|S)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1491-1508
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SULT_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^CMP_SUN_(D|S)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FS(AF|EQ|LT|LE|NE|OR)_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FSUEQ_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FSULE_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FSULT_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FSUNE_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FSUN_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCAF_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCEQ_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCLE_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCLT_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCNE_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCOR_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCUEQ_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCULE_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCULT_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FCUNE_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1509-1517
```tablegen
def : InstRW<[GenericWriteFPUS], (instregex "^FCUN_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FABS_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FFINT_(U|S)_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FFQL_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FFQR_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FTINT_(U|S)_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FRINT_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FTQ_(H|W)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FTRUNC_(U|S)_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1519-1522
```tablegen
// fexdo.[hw], fexupl.[wd], fexupr.[wd]
def : InstRW<[GenericWriteFPUS], (instregex "^FEXDO_(H|W)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FEXUPL_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FEXUPR_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1524-1530
```tablegen
// fclass.[wd], fmax.[wd], fmax_a.[wd], fmin.[wd], fmin_a.[wd], flog2.[wd]
def : InstRW<[GenericWriteFPUS], (instregex "^FCLASS_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FMAX_A_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FMAX_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FMIN_A_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FMIN_(W|D)$")>;
def : InstRW<[GenericWriteFPUS], (instregex "^FLOG2_(W|D)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1532-1534
```tablegen
// interleave right/left, interleave even/odd, insert
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(ILVR|ILVL)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(ILVEV|ILVOD)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1536-1541
```tablegen
// subs_?.[bhwd], subsus_?.[bhwd], subsuu_?.[bhwd], subvi.[bhwd], subv.[bhwd],
def : InstRW<[GenericWriteMSAShortInt], (instregex "^SUBS_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^SUBSUS_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^SUBSUU_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^SUBVI_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortInt], (instregex "^SUBV_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1543-1545
```tablegen
// mod_[su].[bhwd], div_[su].[bhwd]
def : InstRW<[GenericWriteFPUDivI], (instregex "^MOD_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteFPUDivI], (instregex "^DIV_(S|U)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1547-1564
```tablegen
// hadd_[su].[bhwd], hsub_[su].[bhwd], max_[sua].[bhwd], min_[sua].[bhwd],
// maxi_[su].[bhwd], mini_[su].[bhwd], sra?.[bhwd], srar?.[bhwd], srlr.[bhwd],
// sll?.[bhwd], pckev.[bhwd], pckod.[bhwd], nloc.[bhwd], nlzc.[bhwd],
// insve.[bhwd]
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^HADD_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^HSUB_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(MAX|MIN)_S_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(MAX|MIN)_U_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(MAX|MIN)_A_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic],
             (instregex "^(MAXI|MINI)_(S|U)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(SRA|SRAI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(SRL|SRLI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(SRAR|SRARI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(SRLR|SRLRI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(SLL|SLLI)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(PCKEV|PCKOD)_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^(NLOC|NLZC)_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1565-1567
```tablegen
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^INSVE_[BHWD]$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^INSERT_F(D|W)_PSEUDO$")>;
def : InstRW<[GenericWriteMSAShortLogic], (instregex "^FILL_F(D|W)_PSEUDO$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1569-1576
```tablegen
// dpadd_?.[bhwd], dpsub_?.[bhwd], dotp_?.[bhwd], msubv.[bhwd], maddv.[bhwd]
// mulv.[bhwd].
def : InstRW<[GenericWriteMSALongInt], (instregex "^DPADD_(S|U)_[HWD]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^DPSUB_(S|U)_[HWD]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^DOTP_(S|U)_[HWD]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MSUBV_[BHWD]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MADDV_[BHWD]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MULV_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1578-1584
```tablegen
// madd?.q.[hw], msub?.q.[hw], mul?.q.[hw]
def : InstRW<[GenericWriteMSALongInt], (instregex "^MADDR_Q_[HW]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MADD_Q_[HW]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MSUBR_Q_[HW]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MSUB_Q_[HW]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MULR_Q_[HW]$")>;
def : InstRW<[GenericWriteMSALongInt], (instregex "^MUL_Q_[HW]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1586-1596
```tablegen
// fadd.[dw], fmadd.[dw], fmul.[dw], frcp.[dw], frsqrt.[dw], fsqrt.[dw]
// fsub.[dw], fdiv.[dw]
def : InstRW<[GenericWriteFPUL], (instregex "^FADD_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FMADD_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FMSUB_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FMUL_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FRCP_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FRSQRT_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FSQRT_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FSUB_[DW]$")>;
def : InstRW<[GenericWriteFPUL], (instregex "^FDIV_[DW]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1598-1600
```tablegen
// copy.[su]_[bhwd]
def : InstRW<[GenericWriteFPUMoveGPRFPU], (instregex "^COPY_U_[BHW]$")>;
def : InstRW<[GenericWriteFPUMoveGPRFPU], (instregex "^COPY_S_[BHWD]$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1602-1605
```tablegen
def : InstRW<[GenericWriteFPUStore], (instregex "^ST_[BHWD]$")>;
def : InstRW<[GenericWriteFPUStore], (instrs ST_F16)>;
def : InstRW<[GenericWriteFPULoad], (instregex "^LD_[BHWD]$")>;
def : InstRW<[GenericWriteFPULoad], (instrs LD_F16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1607-1607
```tablegen
// Atomic instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1609-1614
```tablegen
// FIXME: Define `WriteAtomic` in the MipsSchedule.td and
// attach it to the Atomic2OpsPostRA, AtomicCmpSwapPostRA, ...
// classes. Then just define resources for the `WriteAtomic` in each
// machine models.
def GenericAtomic : ProcResource<1> { let BufferSize = 1; }
def GenericWriteAtomic : SchedWriteRes<[GenericAtomic]> { let Latency = 2; }
```
- EN: Defines TableGen record `GenericAtomic` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenericAtomic`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1616-1623
```tablegen
def : InstRW<[GenericWriteAtomic],
    (instregex "^ATOMIC_SWAP_I(8|16|32|64)_POSTRA$")>;
def : InstRW<[GenericWriteAtomic],
    (instregex "^ATOMIC_CMP_SWAP_I(8|16|32|64)_POSTRA$")>;
def : InstRW<[GenericWriteAtomic],
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
