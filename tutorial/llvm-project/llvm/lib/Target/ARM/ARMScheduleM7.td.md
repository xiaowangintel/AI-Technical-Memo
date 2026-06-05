# ARMScheduleM7.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleM7.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines the SchedRead/Write data for the ARM Cortex-M7 processor.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleM7`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//=- ARMScheduleM7.td - ARM Cortex-M7 Scheduling Definitions -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the SchedRead/Write data for the ARM Cortex-M7 processor.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-20
```tablegen
def CortexM7Model : SchedMachineModel {
  let IssueWidth = 2;        // Dual issue for most instructions.
  let MicroOpBufferSize = 0; // The Cortex-M7 is in-order.
  let LoadLatency = 2;       // Best case for load-use case.
  let MispredictPenalty = 4; // Mispredict cost for forward branches is 6,
                             // but 4 works better
  let CompleteModel = 0;
}
```
- EN: Defines TableGen record `CortexM7Model` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CortexM7Model`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 22-22
```tablegen
let SchedModel = CortexM7Model in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 24-36
```tablegen
//===--------------------------------------------------------------------===//
// The Cortex-M7 has two ALU, two LOAD, a STORE, a MAC, a BRANCH and a VFP
// pipe. The stages relevant to scheduling are as follows:
//
//   EX1: address generation  shifts
//   EX2: fast load data      ALUs                  FP operation
//   EX3: slow load data      integer writeback     FP operation
//   EX4: store data                                FP writeback
//
// There are shifters in both EX1 and EX2, and some instructions can be
// flexibly allocated between them.  EX2 is used as the "zero" point
// for scheduling, so simple ALU operations executing in EX2 will have
// ReadAdvance<0> (the default) for their source operands and Latency = 1.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 38-51
```tablegen
def M7UnitLoadL  : ProcResource<1> { let BufferSize = 0; }
def M7UnitLoadH  : ProcResource<1> { let BufferSize = 0; }
def M7UnitLoad   : ProcResGroup<[M7UnitLoadL,M7UnitLoadH]> { let BufferSize = 0; }
def M7UnitStore  : ProcResource<1> { let BufferSize = 0; }
def M7UnitALU    : ProcResource<2>;
def M7UnitShift1 : ProcResource<1> { let BufferSize = 0; }
def M7UnitShift2 : ProcResource<1> { let BufferSize = 0; }
def M7UnitMAC    : ProcResource<1> { let BufferSize = 0; }
def M7UnitBranch : ProcResource<1> { let BufferSize = 0; }
def M7UnitVFP    : ProcResource<1> { let BufferSize = 0; }
def M7UnitVPortL : ProcResource<1> { let BufferSize = 0; }
def M7UnitVPortH : ProcResource<1> { let BufferSize = 0; }
def M7UnitVPort  : ProcResGroup<[M7UnitVPortL,M7UnitVPortH]> { let BufferSize = 0; }
def M7UnitSIMD   : ProcResource<1> { let BufferSize = 0; }
```
- EN: Defines TableGen record `M7UnitLoadL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7UnitLoadL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 53-54
```tablegen
//===---------------------------------------------------------------------===//
// Subtarget-specific SchedWrite types with map ProcResources and set latency.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 56-56
```tablegen
def : WriteRes<WriteALU, [M7UnitALU]> { let Latency = 1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 58-63
```tablegen
// Basic ALU with shifts.
let Latency = 1 in {
  def : WriteRes<WriteALUsi,  [M7UnitALU, M7UnitShift1]>;
  def : WriteRes<WriteALUsr,  [M7UnitALU, M7UnitShift1]>;
  def : WriteRes<WriteALUSsr, [M7UnitALU, M7UnitShift1]>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 65-68
```tablegen
// Compares.
def : WriteRes<WriteCMP,   [M7UnitALU]> { let Latency = 1; }
def : WriteRes<WriteCMPsi, [M7UnitALU, M7UnitShift1]> { let Latency = 2; }
def : WriteRes<WriteCMPsr, [M7UnitALU, M7UnitShift1]> { let Latency = 2; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 70-76
```tablegen
// Multiplies.
let Latency = 2 in {
  def : WriteRes<WriteMUL16,   [M7UnitMAC]>;
  def : WriteRes<WriteMUL32,   [M7UnitMAC]>;
  def : WriteRes<WriteMUL64Lo, [M7UnitMAC]>;
  def : WriteRes<WriteMUL64Hi, []> { let NumMicroOps = 0; }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 78-84
```tablegen
// Multiply-accumulates.
let Latency = 2 in {
  def : WriteRes<WriteMAC16,   [M7UnitMAC]>;
  def : WriteRes<WriteMAC32,   [M7UnitMAC]>;
  def : WriteRes<WriteMAC64Lo, [M7UnitMAC]> { let Latency = 2; }
  def : WriteRes<WriteMAC64Hi, []> { let NumMicroOps = 0; }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 86-91
```tablegen
// Divisions.
// These cannot be dual-issued with any instructions.
def : WriteRes<WriteDIV, [M7UnitALU]> {
  let Latency = 7;
  let SingleIssue = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 93-96
```tablegen
// Loads/Stores.
def : WriteRes<WriteLd,    [M7UnitLoad]> { let Latency = 1; }
def : WriteRes<WritePreLd, [M7UnitLoad]> { let Latency = 2; }
def : WriteRes<WriteST,    [M7UnitStore]> { let Latency = 2; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 98-101
```tablegen
// Branches.
def : WriteRes<WriteBr,    [M7UnitBranch]> { let Latency = 2; }
def : WriteRes<WriteBrL,   [M7UnitBranch]> { let Latency = 2; }
def : WriteRes<WriteBrTbl, [M7UnitBranch]> { let Latency = 2; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 103-104
```tablegen
// Noop.
def : WriteRes<WriteNoop, []> { let Latency = 0; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 106-114
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for floating-point instructions
//
// Floating point conversions.
def : WriteRes<WriteFPCVT, [M7UnitVFP, M7UnitVPort]> { let Latency = 3; }
def : WriteRes<WriteFPMOV, [M7UnitVPort]>            { let Latency = 3; }
def M7WriteFPMOV64 : SchedWriteRes<[M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 3;
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 116-122
```tablegen
// The FP pipeline has a latency of 3 cycles.
// ALU operations (32/64-bit).  These go down the FP pipeline.
def : WriteRes<WriteFPALU32, [M7UnitVFP, M7UnitVPort]>  { let Latency = 3; }
def : WriteRes<WriteFPALU64, [M7UnitVFP, M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 4;
  let BeginGroup = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 124-129
```tablegen
// Multiplication
def : WriteRes<WriteFPMUL32, [M7UnitVFP, M7UnitVPort]> { let Latency = 3; }
def : WriteRes<WriteFPMUL64, [M7UnitVFP, M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 7;
  let BeginGroup = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 131-136
```tablegen
// Multiply-accumulate.  FPMAC goes down the FP Pipeline.
def : WriteRes<WriteFPMAC32, [M7UnitVFP, M7UnitVPort]> { let Latency = 6; }
def : WriteRes<WriteFPMAC64, [M7UnitVFP, M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 11;
  let BeginGroup = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 138-143
```tablegen
// Division.   Effective scheduling latency is 3, though real latency is larger
def : WriteRes<WriteFPDIV32, [M7UnitVFP, M7UnitVPort]>  { let Latency = 16; }
def : WriteRes<WriteFPDIV64, [M7UnitVFP, M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 30;
  let BeginGroup = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 145-150
```tablegen
// Square-root.  Effective scheduling latency is 3; real latency is larger
def : WriteRes<WriteFPSQRT32, [M7UnitVFP, M7UnitVPort]> { let Latency = 16; }
def : WriteRes<WriteFPSQRT64, [M7UnitVFP, M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 30;
  let BeginGroup = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 152-152
```tablegen
def M7WriteShift2   : SchedWriteRes<[M7UnitALU, M7UnitShift2]> {}
```
- EN: Defines TableGen record `M7WriteShift2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7WriteShift2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 154-162
```tablegen
// Not used for M7, but needing definitions anyway
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

### Lines 164-171
```tablegen
def M7SingleIssue : SchedWriteRes<[]> {
  let SingleIssue = 1;
  let NumMicroOps = 0;
}
def M7Slot0Only   : SchedWriteRes<[]> {
  let BeginGroup = 1;
  let NumMicroOps = 0;
}
```
- EN: Defines TableGen record `M7SingleIssue` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7SingleIssue`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 173-183
```tablegen
// What pipeline stage operands need to be ready for depending on
// where they come from.
def : ReadAdvance<ReadALUsr, 0>;
def : ReadAdvance<ReadMUL, 0>;
def : ReadAdvance<ReadMAC, 1>;
def : ReadAdvance<ReadALU, 0>;
def : ReadAdvance<ReadFPMUL, 0>;
def : ReadAdvance<ReadFPMAC, 3>;
def M7Read_ISS : SchedReadAdvance<-1>;     // operands needed at EX1
def M7Read_EX2   : SchedReadAdvance<1>;    // operands needed at EX3
def M7Read_EX3   : SchedReadAdvance<2>;    // operands needed at EX4
```
- EN: Defines TableGen record `M7Read_ISS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7Read_ISS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 185-192
```tablegen
// Non general purpose instructions may not be dual issued. These
// use both issue units.
def M7NonGeneralPurpose : SchedWriteRes<[]> {
  // Assume that these will go down the main ALU pipeline.
  // In reality, many look likely to stall the whole pipeline.
  let Latency = 3;
  let SingleIssue = 1;
}
```
- EN: Defines TableGen record `M7NonGeneralPurpose` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7NonGeneralPurpose`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 194-198
```tablegen
// List the non general purpose instructions.
def : InstRW<[M7NonGeneralPurpose], (instregex "t2MRS", "tSVC", "tBKPT",
                                     "t2MSR", "t2DMB", "t2DSB", "t2ISB",
                                     "t2HVC", "t2SMC", "t2UDF", "ERET",
                                     "tHINT", "t2HINT", "t2CLREX", "BUNDLE")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 200-205
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for load/store
//
// Mark whether the loads/stores must be single-issue
// Address operands are needed earlier
// Data operands are needed later
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 207-215
```tablegen
def M7BaseUpdate : SchedWriteRes<[]> {
    let Latency = 0; // Update is bypassable out of EX1
    let NumMicroOps = 0;
}
def M7LoadLatency1 : SchedWriteRes<[]> {
    let Latency = 1;
    let NumMicroOps = 0;
}
def M7SlowLoad : SchedWriteRes<[M7UnitLoad]>            { let Latency = 2; }
```
- EN: Defines TableGen record `M7BaseUpdate` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7BaseUpdate`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 217-218
```tablegen
// Byte and half-word loads should have greater latency than other loads.
// So should load exclusive.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 220-228
```tablegen
def : InstRW<[M7SlowLoad],
      (instregex "t2LDR(B|H|SB|SH)pc")>;
def : InstRW<[M7SlowLoad, M7Read_ISS],
      (instregex "t2LDR(B|H|SB|SH)T", "t2LDR(B|H|SB|SH)i",
                 "tLDR(B|H)i")>;
def : InstRW<[M7SlowLoad, M7Read_ISS, M7Read_ISS],
      (instregex "t2LDR(B|H|SB|SH)s", "tLDR(B|H)r", "tLDR(SB|SH)")>;
def : InstRW<[M7SlowLoad, M7BaseUpdate, M7Read_ISS],
      (instregex "t2LDR(B|H|SB|SH)_(POST|PRE)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 230-236
```tablegen
// Exclusive loads/stores cannot be dual-issued
def : InstRW<[WriteLd, M7Slot0Only, M7Read_ISS],
      (instregex "t2LDREX$")>;
def : InstRW<[M7SlowLoad, M7Slot0Only, M7Read_ISS],
      (instregex "t2LDREX(B|H)")>;
def : InstRW<[WriteST, M7SingleIssue, M7Read_EX2, M7Read_ISS],
      (instregex "t2STREX(B|H)?$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 238-240
```tablegen
// Load/store multiples cannot be dual-issued.  Note that default scheduling
// occurs around read/write times of individual registers in the list; read
// time for STM cannot be overridden because it is a variadic source operand.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 242-249
```tablegen
def : InstRW<[WriteLd, M7SingleIssue, M7Read_ISS],
      (instregex "(t|t2)LDM(DB|IA)$")>;
def : InstRW<[WriteST, M7SingleIssue, M7Read_ISS],
      (instregex "(t|t2)STM(DB|IA)$")>;
def : InstRW<[M7BaseUpdate, WriteLd, M7SingleIssue, M7Read_ISS],
      (instregex "(t|t2)LDM(DB|IA)_UPD$", "tPOP")>;
def : InstRW<[M7BaseUpdate, WriteST, M7SingleIssue, M7Read_ISS],
      (instregex "(t|t2)STM(DB|IA)_UPD$", "tPUSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 251-251
```tablegen
// Load/store doubles cannot be dual-issued.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 253-261
```tablegen
def : InstRW<[M7BaseUpdate, WriteST, M7SingleIssue,
              M7Read_EX2, M7Read_EX2, M7Read_ISS],
      (instregex "t2STRD_(PRE|POST)")>;
def : InstRW<[WriteST, M7SingleIssue, M7Read_EX2, M7Read_EX2, M7Read_ISS],
      (instregex "t2STRDi")>;
def : InstRW<[WriteLd, M7LoadLatency1, M7SingleIssue, M7BaseUpdate, M7Read_ISS],
      (instregex "t2LDRD_(PRE|POST)")>;
def : InstRW<[WriteLd, M7LoadLatency1, M7SingleIssue, M7Read_ISS],
      (instregex "t2LDRDi")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 263-271
```tablegen
// Word load / preload
def : InstRW<[WriteLd],
      (instregex "t2LDRpc", "t2PL[DI]pci", "tLDRpci")>;
def : InstRW<[WriteLd, M7Read_ISS],
      (instregex "t2LDR(i|T)", "t2PL[DI](W)?i", "tLDRi", "tLDRspi")>;
def : InstRW<[WriteLd, M7Read_ISS, M7Read_ISS],
      (instregex "t2LDRs", "t2PL[DI](w)?s", "tLDRr")>;
def : InstRW<[WriteLd, M7BaseUpdate, M7Read_ISS],
      (instregex "t2LDR_(POST|PRE)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 273-279
```tablegen
// Stores
def : InstRW<[M7BaseUpdate, WriteST, M7Read_EX2, M7Read_ISS],
      (instregex "t2STR(B|H)?_(POST|PRE)")>;
def : InstRW<[WriteST, M7Read_EX2, M7Read_ISS, M7Read_ISS],
      (instregex "t2STR(B|H)?s$", "tSTR(B|H)?r$")>;
def : InstRW<[WriteST, M7Read_EX2, M7Read_ISS],
      (instregex "t2STR(B|H)?(i|T)", "tSTR(B|H)?i$", "tSTRspi")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 281-281
```tablegen
// TBB/TBH - single-issue only; takes two cycles to issue
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 283-286
```tablegen
def M7TableLoad : SchedWriteRes<[M7UnitLoad]> {
  let NumMicroOps = 2;
  let SingleIssue = 1;
}
```
- EN: Defines TableGen record `M7TableLoad` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7TableLoad`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 288-288
```tablegen
def : InstRW<[M7TableLoad, M7Read_ISS, M7Read_ISS], (instregex "t2TB")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 290-290
```tablegen
// VFP loads and stores
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 292-300
```tablegen
def M7LoadSP  : SchedWriteRes<[M7UnitLoad, M7UnitVPort]> { let Latency = 1; }
def M7LoadDP  : SchedWriteRes<[M7UnitLoadL, M7UnitLoadH, M7UnitVPortL, M7UnitVPortH]> {
  let Latency = 2;
  let SingleIssue = 1;
}
def M7StoreSP : SchedWriteRes<[M7UnitStore, M7UnitVPort]>;
def M7StoreDP : SchedWriteRes<[M7UnitStore, M7UnitVPortL, M7UnitVPortH]> {
  let SingleIssue = 1;
}
```
- EN: Defines TableGen record `M7LoadSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7LoadSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 302-305
```tablegen
def : InstRW<[M7LoadSP, M7Read_ISS],                 (instregex "VLDR(S|H)$")>;
def : InstRW<[M7LoadDP, M7Read_ISS],                 (instregex "VLDRD$")>;
def : InstRW<[M7StoreSP, M7Read_EX3, M7Read_ISS],    (instregex "VSTR(S|H)$")>;
def : InstRW<[M7StoreDP, M7Read_EX3, M7Read_ISS],    (instregex "VSTRD$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 307-307
```tablegen
// Load/store multiples cannot be dual-issued.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 309-316
```tablegen
def : InstRW<[WriteLd, M7SingleIssue, M7Read_ISS],
      (instregex "VLDM(S|D|Q)(DB|IA)$")>;
def : InstRW<[WriteST, M7SingleIssue, M7Read_ISS],
      (instregex "VSTM(S|D|Q)(DB|IA)$")>;
def : InstRW<[M7BaseUpdate, WriteLd, M7SingleIssue, M7Read_ISS],
      (instregex "VLDM(S|D|Q)(DB|IA)_UPD$")>;
def : InstRW<[M7BaseUpdate, WriteST, M7SingleIssue, M7Read_ISS],
      (instregex "VSTM(S|D|Q)(DB|IA)_UPD$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 318-320
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for ALU
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 322-323
```tablegen
// Shifted ALU operands are read a cycle early.
def M7Ex1ReadNoFastBypass : SchedReadAdvance<-1, [WriteLd, M7LoadLatency1]>;
```
- EN: Defines TableGen record `M7Ex1ReadNoFastBypass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7Ex1ReadNoFastBypass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 325-330
```tablegen
def : InstRW<[WriteALUsi, M7Ex1ReadNoFastBypass, M7Read_ISS],
             (instregex "t2(ADC|ADDS|ADD|BIC|EOR|ORN|ORR|RSBS|RSB|SBC|SUBS)rs$",
                        "t2(SUB|CMP|CMN|TEQ|TST)rs$",
                        "t2(A|L)SRs1$")>;
def : InstRW<[WriteALUsi, M7Read_ISS],
             (instregex "t2MVNs")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 332-335
```tablegen
// Treat pure shift operations (except for RRX) as if they used the EX1
// shifter but have timing as if they used the EX2 shifter as they usually
// can choose the EX2 shifter when needed.  Will miss a few dual-issue cases,
// but the results prove to be better than trying to get them exact.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 337-338
```tablegen
def : InstRW<[M7WriteShift2, M7Read_ISS], (instregex "t2RRX$")>;
def : InstRW<[WriteALUsi], (instregex "(t|t2)(LSL|LSR|ASR|ROR)r", "tROR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 340-340
```tablegen
// Instructions that use the shifter, but have normal timing.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 342-342
```tablegen
def : InstRW<[WriteALUsi,M7Slot0Only], (instregex "t2(BFC|BFI)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 344-344
```tablegen
// Instructions which are slot zero only but otherwise normal.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 346-346
```tablegen
def : InstRW<[WriteALU, M7Slot0Only], (instregex "t2CLZ")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 348-348
```tablegen
// MAC operations that don't have SchedRW set.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 350-350
```tablegen
def : InstRW<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC], (instregex "t2SML[AS]D")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 352-354
```tablegen
// Divides are special because they stall for their latency, and so look like a
// single-cycle as far as scheduling opportunities go.  By putting WriteALU
// first, we make the operand latency 1, but keep the instruction latency 7.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 356-356
```tablegen
def : InstRW<[WriteALU, WriteDIV], (instregex "t2(S|U)DIV")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 358-358
```tablegen
// DSP extension operations
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 360-377
```tablegen
def M7WriteSIMD1   : SchedWriteRes<[M7UnitSIMD, M7UnitALU]> {
  let Latency = 1;
  let BeginGroup = 1;
}
def M7WriteSIMD2   : SchedWriteRes<[M7UnitSIMD, M7UnitALU]> {
  let Latency = 2;
  let BeginGroup = 1;
}
def M7WriteShSIMD1 : SchedWriteRes<[M7UnitSIMD, M7UnitALU, M7UnitShift1]> {
  let Latency = 1;
  let BeginGroup = 1;
}
def M7WriteShSIMD0 : SchedWriteRes<[M7UnitSIMD, M7UnitALU, M7UnitShift1]> {
  let Latency = 0;      // Bypassable out of EX1
  let BeginGroup = 1;
}
def M7WriteShSIMD2 : SchedWriteRes<[M7UnitSIMD, M7UnitALU, M7UnitShift1]> {
  let Latency = 2;
```
- EN: Defines TableGen record `M7WriteSIMD1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7WriteSIMD1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 378-379
```tablegen
  let BeginGroup = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 381-398
```tablegen
def : InstRW<[M7WriteShSIMD2, M7Read_ISS],
             (instregex "t2(S|U)SAT")>;
def : InstRW<[M7WriteSIMD1, ReadALU],
             (instregex "(t|t2)(S|U)XT(B|H)")>;
def : InstRW<[M7WriteSIMD1, ReadALU, ReadALU],
             (instregex "t2(S|SH|U|UH)(ADD16|ADD8|ASX|SAX|SUB16|SUB8)",
                        "t2SEL")>;
def : InstRW<[M7WriteSIMD2, ReadALU, ReadALU],
             (instregex "t2(Q|UQ)(ADD|ASX|SAX|SUB)", "t2USAD8")>;
def : InstRW<[M7WriteShSIMD2, M7Read_ISS, M7Read_ISS],
             (instregex "t2QD(ADD|SUB)")>;
def : InstRW<[M7WriteShSIMD0, M7Read_ISS],
             (instregex "t2(RBIT|REV)", "tREV")>;
def : InstRW<[M7WriteShSIMD1, M7Read_ISS],
             (instregex "t2(SBFX|UBFX)")>;
def : InstRW<[M7WriteShSIMD1, ReadALU, M7Read_ISS],
             (instregex "t2PKH(BT|TB)", "t2(S|U)XTA")>;
def : InstRW<[M7WriteSIMD2, ReadALU, ReadALU, M7Read_EX2],
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 399-399
```tablegen
             (instregex "t2USADA8")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 401-402
```tablegen
// MSR/MRS
def : InstRW<[M7NonGeneralPurpose], (instregex "MSR", "MRS")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 404-406
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for FP operations
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 408-417
```tablegen
// Effective scheduling latency is really 3 for nearly all FP operations,
// even if their true latency is higher.
def M7WriteVFPLatOverride : SchedWriteRes<[]> {
  let Latency = 3;
  let NumMicroOps = 0;
}
def M7WriteVFPExtraVPort  : SchedWriteRes<[M7UnitVPort]> {
  let Latency = 3;
  let NumMicroOps = 0;
}
```
- EN: Defines TableGen record `M7WriteVFPLatOverride` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7WriteVFPLatOverride`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 419-423
```tablegen
// Instructions which are missing default schedules.
def : InstRW<[WriteFPALU32],
             (instregex "V(ABS|CVT.*|NEG|FP_VMAX.*|FP_VMIN.*|RINT.*)S$")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPALU64],
             (instregex "V(ABS|CVT.*|NEG|FP_VMAX.*|FP_VMIN.*|RINT.*)D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 425-432
```tablegen
// VCMP
def M7WriteVCMPS : SchedWriteRes<[M7UnitVFP, M7UnitVPort]> { let Latency = 0; }
def M7WriteVCMPD : SchedWriteRes<[M7UnitVFP, M7UnitVPort, M7UnitVPort]> {
  let Latency = 0;
  let BeginGroup = 1;
}
def : InstRW<[M7WriteVCMPS], (instregex "VCMPS$")>;
def : InstRW<[M7WriteVCMPD], (instregex "VCMPD$")>;
```
- EN: Defines TableGen record `M7WriteVCMPS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7WriteVCMPS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 434-438
```tablegen
    // VMRS/VMSR
def M7VMRS : SchedWriteRes<[M7UnitVFP, M7UnitVPort]> { let SingleIssue = 1; }
def M7VMSR : SchedWriteRes<[M7UnitVFP, M7UnitVPort]> { let SingleIssue = 1; }
def : InstRW<[M7VMRS], (instregex "FMSTAT")>;
def : InstRW<[M7VMSR], (instregex "VMSR")>;
```
- EN: Defines TableGen record `M7VMRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7VMRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 440-445
```tablegen
// VSEL cannot bypass in its implied $cpsr operand; model as earlier read
def : InstRW<[WriteFPALU32, M7Slot0Only, ReadALU, ReadALU, M7Read_ISS],
             (instregex "VSEL.*S$")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPALU64, M7Slot0Only,
              ReadALU, ReadALU, M7Read_ISS],
             (instregex "VSEL.*D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 447-455
```tablegen
// VMOV
def : InstRW<[WriteFPMOV],
             (instregex "VMOV(H|S)$", "FCONST(H|S)")>;
def : InstRW<[WriteFPMOV, M7WriteVFPExtraVPort, M7Slot0Only],
             (instregex "VMOVD$")>;
def : InstRW<[WriteFPMOV, M7WriteVFPExtraVPort, M7Slot0Only],
             (instregex "FCONSTD")>;
def : InstRW<[WriteFPMOV, M7WriteVFPExtraVPort, M7SingleIssue],
             (instregex "VMOV(DRR|RRD|RRS|SRR)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 457-457
```tablegen
// Larger-latency overrides.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 459-466
```tablegen
def : InstRW<[M7WriteVFPLatOverride, WriteFPDIV32],  (instregex "VDIVS")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPDIV64],  (instregex "VDIVD")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPSQRT32], (instregex "VSQRTS")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPSQRT64], (instregex "VSQRTD")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPMUL64],
             (instregex "V(MUL|NMUL)D")>;
def : InstRW<[M7WriteVFPLatOverride, WriteFPALU64],
             (instregex "V(ADD|SUB)D")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 468-470
```tablegen
// Multiply-accumulate.  Chained SP timing is correct; rest need overrides
// Double-precision chained MAC stalls the pipeline behind it for 3 cycles,
// making it appear to have 3 cycle latency for scheduling.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 472-474
```tablegen
def : InstRW<[M7WriteVFPLatOverride, WriteFPMAC64,
              ReadFPMAC, ReadFPMUL, ReadFPMUL],
             (instregex "V(N)?ML(A|S)D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 476-476
```tablegen
// Single-precision fused MACs look like latency 5 with advance of 2.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 478-482
```tablegen
def M7WriteVFPLatOverride5 : SchedWriteRes<[]> {
  let Latency = 5;
  let NumMicroOps = 0;
}
def M7ReadFPMAC2   : SchedReadAdvance<2>;
```
- EN: Defines TableGen record `M7WriteVFPLatOverride5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M7WriteVFPLatOverride5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 484-486
```tablegen
def : InstRW<[M7WriteVFPLatOverride5, WriteFPMAC32,
              M7ReadFPMAC2, ReadFPMUL, ReadFPMUL],
             (instregex "VF(N)?M(A|S)S$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 488-489
```tablegen
// Double-precision fused MAC stalls the pipeline behind it for 2 cycles, making
// it appear to have 3 cycle latency for scheduling.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 491-493
```tablegen
def : InstRW<[M7WriteVFPLatOverride, WriteFPMAC64,
              ReadFPMAC, ReadFPMUL, ReadFPMUL],
             (instregex "VF(N)?M(A|S)D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 495-495
```tablegen
}  // SchedModel = CortexM7Model
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
