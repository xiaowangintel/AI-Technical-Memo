# ARMScheduleM85.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleM85.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines the machine model for the ARM Cortex-M85 processor.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleM85`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```tablegen
//=- ARMScheduleM85.td - ARM Cortex-M85 Scheduling Definitions -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the machine model for the ARM Cortex-M85 processor.
//
// All timing is referred to EX2.  Thus, operands which are needed at EX1 are
// stated to have a ReadAdvance of -1.  The FP/MVE pipe actually begins at EX3
// but is described as if it were in EX2 to avoid having unnaturally long latencies
// with delayed inputs on every instruction.  Instead, whenever an FP instruction
// must access a GP register or a non-FP instruction (which includes loads/stores)
// must access an FP register, the operand timing is adjusted:
//     FP accessing GPR:     read one cycle later, write one cycle later
//                           NOTE: absolute spec timing already includes this if
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 19-21
```tablegen
//                                 referenced to EX2
//     non-FP accessing FPR: read one cycle earlier, write one cycle earlier
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 23-30
```tablegen
def CortexM85Model : SchedMachineModel {
  let IssueWidth = 2;        // Dual issue for most instructions.
  let MicroOpBufferSize = 0; // M85 is in-order.
  let LoadLatency = 2;       // Best case for load-use case.
  let MispredictPenalty = 4; // Mispredict cost for forward branches is 7,
                             // but 4 works better
  let CompleteModel = 0;
}
```
- EN: Defines TableGen record `CortexM85Model` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CortexM85Model`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 32-32
```tablegen
let SchedModel = CortexM85Model in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 34-37
```tablegen
//===--------------------------------------------------------------------===//
// CortexM85 has two ALU, two LOAD, two STORE, a MAC, a BRANCH and two VFP
// pipes (with three units).  There are three shifters available: one per
// stage.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 39-56
```tablegen
def M85UnitLoadL  : ProcResource<1> { let BufferSize = 0; }
def M85UnitLoadH  : ProcResource<1> { let BufferSize = 0; }
def M85UnitLoad   : ProcResGroup<[M85UnitLoadL,M85UnitLoadH]> { let BufferSize = 0; }
def M85UnitStoreL : ProcResource<1> { let BufferSize = 0; }
def M85UnitStoreH : ProcResource<1> { let BufferSize = 0; }
def M85UnitStore  : ProcResGroup<[M85UnitStoreL,M85UnitStoreH]> { let BufferSize = 0; }
def M85UnitALU    : ProcResource<2> { let BufferSize = 0; }
def M85UnitShift1 : ProcResource<1> { let BufferSize = 0; }
def M85UnitShift2 : ProcResource<1> { let BufferSize = 0; }
def M85UnitMAC    : ProcResource<1> { let BufferSize = 0; }
def M85UnitBranch : ProcResource<1> { let BufferSize = 0; }
def M85UnitVFPAL  : ProcResource<1> { let BufferSize = 0; }
def M85UnitVFPAH  : ProcResource<1> { let BufferSize = 0; }
def M85UnitVFPA   : ProcResGroup<[M85UnitVFPAL,M85UnitVFPAH]> { let BufferSize = 0; }
def M85UnitVFPBL  : ProcResource<1> { let BufferSize = 0; }
def M85UnitVFPBH  : ProcResource<1> { let BufferSize = 0; }
def M85UnitVFPB   : ProcResGroup<[M85UnitVFPBL,M85UnitVFPBH]> { let BufferSize = 0; }
def M85UnitVFPCL  : ProcResource<1> { let BufferSize = 0; }
```
- EN: Defines TableGen record `M85UnitLoadL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85UnitLoadL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 57-65
```tablegen
def M85UnitVFPCH  : ProcResource<1> { let BufferSize = 0; }
def M85UnitVFPC   : ProcResGroup<[M85UnitVFPCL,M85UnitVFPCH]> { let BufferSize = 0; }
def M85UnitVFPD   : ProcResource<1> { let BufferSize = 0; }
def M85UnitVPortL : ProcResource<1> { let BufferSize = 0; }
def M85UnitVPortH : ProcResource<1> { let BufferSize = 0; }
def M85UnitVPort  : ProcResGroup<[M85UnitVPortL,M85UnitVPortH]> { let BufferSize = 0; }
def M85UnitSIMD   : ProcResource<1> { let BufferSize = 0; }
def M85UnitLShift : ProcResource<1> { let BufferSize = 0; }
def M85UnitDiv    : ProcResource<1> { let BufferSize = 0; }
```
- EN: Defines TableGen record `M85UnitVFPCH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85UnitVFPCH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 67-67
```tablegen
def M85UnitSlot0 : ProcResource<1> { let BufferSize = 0; }
```
- EN: Defines TableGen record `M85UnitSlot0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85UnitSlot0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 69-70
```tablegen
//===---------------------------------------------------------------------===//
// Subtarget-specific SchedWrite types with map ProcResources and set latency.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 72-72
```tablegen
def : WriteRes<WriteALU, [M85UnitALU]> { let Latency = 1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 74-79
```tablegen
// Basic ALU with shifts.
let Latency = 1 in {
  def : WriteRes<WriteALUsi,  [M85UnitALU, M85UnitShift1]>;
  def : WriteRes<WriteALUsr,  [M85UnitALU, M85UnitShift1]>;
  def : WriteRes<WriteALUSsr, [M85UnitALU, M85UnitShift1]>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 81-84
```tablegen
// Compares.
def : WriteRes<WriteCMP,   [M85UnitALU]> { let Latency = 1; }
def : WriteRes<WriteCMPsi, [M85UnitALU, M85UnitShift1]> { let Latency = 2; }
def : WriteRes<WriteCMPsr, [M85UnitALU, M85UnitShift1]> { let Latency = 2; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 86-92
```tablegen
// Multiplies.
let Latency = 2 in {
  def : WriteRes<WriteMUL16,   [M85UnitMAC]>;
  def : WriteRes<WriteMUL32,   [M85UnitMAC]>;
  def : WriteRes<WriteMUL64Lo, [M85UnitMAC]>;
  def : WriteRes<WriteMUL64Hi, []> { let NumMicroOps = 0; }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 94-100
```tablegen
// Multiply-accumulates.
let Latency = 2 in {
def : WriteRes<WriteMAC16,   [M85UnitMAC]>;
def : WriteRes<WriteMAC32,   [M85UnitMAC]>;
def : WriteRes<WriteMAC64Lo, [M85UnitMAC]>;
def : WriteRes<WriteMAC64Hi, []> { let NumMicroOps = 0; }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 102-105
```tablegen
// Divisions.
def : WriteRes<WriteDIV, [M85UnitDiv]> {
  let Latency = 7;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 107-112
```tablegen
// Loads/Stores.
def : WriteRes<WriteLd,    [M85UnitLoad]> { let Latency = 1; }
def : WriteRes<WritePreLd, [M85UnitLoad]> { let Latency = 2; }
def : WriteRes<WriteST,    [M85UnitStore]> { let Latency = 2; }
def M85WriteLdWide : SchedWriteRes<[M85UnitLoadL, M85UnitLoadH]> { let Latency = 1; }
def M85WriteStWide : SchedWriteRes<[M85UnitStoreL, M85UnitStoreH]> { let Latency = 2; }
```
- EN: Defines TableGen record `M85WriteLdWide` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85WriteLdWide`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 114-117
```tablegen
// Branches.
def : WriteRes<WriteBr,    [M85UnitBranch]> { let Latency = 2; }
def : WriteRes<WriteBrL,   [M85UnitBranch]> { let Latency = 2; }
def : WriteRes<WriteBrTbl, [M85UnitBranch]> { let Latency = 2; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 119-120
```tablegen
// Noop.
def : WriteRes<WriteNoop, []> { let Latency = 0; let NumMicroOps = 0; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 122-130
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for floating-point instructions
//
// Floating point conversions.
def : WriteRes<WriteFPCVT, [M85UnitVFPB, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 2;
}
def : WriteRes<WriteFPMOV, [M85UnitVPort, M85UnitSlot0]> { let Latency = 1; }
def M85WriteFPMOV64 : SchedWriteRes<[M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> { let Latency = 1; }
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 132-138
```tablegen
// ALU operations (32/64-bit).  These go down the FP pipeline.
def : WriteRes<WriteFPALU32, [M85UnitVFPA, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 2;
}
def : WriteRes<WriteFPALU64, [M85UnitVFPAL, M85UnitVFPAH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
  let Latency = 6;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 140-146
```tablegen
// Multiplication
def : WriteRes<WriteFPMUL32, [M85UnitVFPB, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 3;
}
def : WriteRes<WriteFPMUL64, [M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
  let Latency = 8;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 148-154
```tablegen
// Multiply-accumulate.  FPMAC goes down the FP Pipeline.
def : WriteRes<WriteFPMAC32, [M85UnitVFPB, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 5;
}
def : WriteRes<WriteFPMAC64, [M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
  let Latency = 14;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 156-162
```tablegen
// Division.   Effective scheduling latency is 3, though real latency is larger
def : WriteRes<WriteFPDIV32, [M85UnitVFPB, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 14;
}
def : WriteRes<WriteFPDIV64, [M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
  let Latency = 29;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 164-170
```tablegen
// Square-root.  Effective scheduling latency is 3, though real latency is larger
def : WriteRes<WriteFPSQRT32, [M85UnitVFPB, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 14;
}
def : WriteRes<WriteFPSQRT64, [M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
  let Latency = 29;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 172-175
```tablegen
let NumMicroOps = 0 in {
  def M85SingleIssue : SchedWriteRes<[]> { let SingleIssue = 1; }
  def M85Slot0Only   : SchedWriteRes<[M85UnitSlot0]> { }
}
```
- EN: Defines TableGen record `M85SingleIssue` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85SingleIssue`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 177-194
```tablegen
// What pipeline stage operands need to be ready for depending on
// where they come from.
def : ReadAdvance<ReadALUsr, 0>;
def : ReadAdvance<ReadMUL, 0>;
def : ReadAdvance<ReadMAC, 1>;
def : ReadAdvance<ReadALU, 0>;
def : ReadAdvance<ReadFPMUL, 0>;
def : ReadAdvance<ReadFPMAC, 3>;
def M85Read_ISSm1 : SchedReadAdvance<-2>;    // operands needed at ISS
def M85Read_ISS : SchedReadAdvance<-1>;    // operands needed at EX1
def M85Read_EX1 : SchedReadAdvance<0>;     // operands needed at EX2
def M85Read_EX2 : SchedReadAdvance<1>;    // operands needed at EX3
def M85Read_EX3 : SchedReadAdvance<2>;    // operands needed at EX4
def M85Read_EX4 : SchedReadAdvance<3>;    // operands needed at EX5
def M85Write1   : SchedWriteRes<[]> {
  let Latency = 1;
  let NumMicroOps = 0;
}
```
- EN: Defines TableGen record `M85Read_ISSm1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85Read_ISSm1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 195-199
```tablegen
def M85Write2   : SchedWriteRes<[]> {
  let Latency = 2;
  let NumMicroOps = 0;
}
def M85WriteShift2   : SchedWriteRes<[M85UnitALU, M85UnitShift2]> {}
```
- EN: Defines TableGen record `M85Write2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85Write2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 201-208
```tablegen
// Non general purpose instructions may not be dual issued. These
// use both issue units.
def M85NonGeneralPurpose : SchedWriteRes<[]> {
  // Assume that these will go down the main ALU pipeline.
  // In reality, many look likely to stall the whole pipeline.
  let Latency = 3;
  let SingleIssue = 1;
}
```
- EN: Defines TableGen record `M85NonGeneralPurpose` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85NonGeneralPurpose`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 210-214
```tablegen
// List the non general purpose instructions.
def : InstRW<[M85NonGeneralPurpose],
                (instregex "t2MRS", "tSVC", "tBKPT", "t2MSR", "t2DMB", "t2DSB",
                           "t2ISB", "t2HVC", "t2SMC", "t2UDF", "ERET", "tHINT",
                           "t2HINT", "t2CLREX", "t2CLRM", "BUNDLE")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 216-221
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

### Lines 223-236
```tablegen
let NumMicroOps = 0 in {
  def M85BaseUpdate : SchedWriteRes<[]> {
    // Update is bypassable out of EX1
    let Latency = 0;
  }
  def M85MVERBaseUpdate : SchedWriteRes<[]> { let Latency = 1; }
  // Q register base update is available in EX3 to bypass into EX2/ISS.
  //  Latency=2 matches what we want for ISS, Latency=1 for EX2.  Going
  //  with 2, as base update into another load/store is most likely.  Could
  //  change later in an override.
  def M85MVEQBaseUpdate : SchedWriteRes<[]> { let Latency = 2; }
  def M85LoadLatency1 : SchedWriteRes<[]> { let Latency = 1; }
}
def M85SlowLoad : SchedWriteRes<[M85UnitLoad]> { let Latency = 2; }
```
- EN: Defines TableGen record `M85BaseUpdate` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85BaseUpdate`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-239
```tablegen
// Byte and half-word loads should have greater latency than other loads.
// So should load exclusive?
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 241-251
```tablegen
def : InstRW<[M85SlowLoad],
               (instregex "t2LDR(B|H|SB|SH)pc")>;
def : InstRW<[M85SlowLoad, M85Read_ISS],
               (instregex "t2LDR(B|H|SB|SH)T", "t2LDR(B|H|SB|SH)i",
                          "tLDRspi", "tLDR(B|H)i")>;
def : InstRW<[M85SlowLoad, M85Read_ISS, M85Read_ISS],
               (instregex "t2LDR(B|H|SB|SH)s")>;
def : InstRW<[M85SlowLoad, M85Read_ISS, M85Read_ISS],
               (instregex "tLDR(B|H)r", "tLDR(SB|SH)")>;
def : InstRW<[M85SlowLoad, M85BaseUpdate, M85Read_ISS],
               (instregex "t2LDR(B|H|SB|SH)_(POST|PRE)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 253-263
```tablegen
// Exclusive/acquire/release loads/stores cannot be dual-issued
def : InstRW<[WriteLd, M85SingleIssue, M85Read_ISS],
               (instregex "t2LDREX$", "t2LDA(EX)?$")>;
def : InstRW<[M85WriteLdWide, M85LoadLatency1, M85SingleIssue, M85Read_ISS],
               (instregex "t2LDAEXD$")>;
def : InstRW<[M85SlowLoad, M85SingleIssue, M85Read_ISS],
               (instregex "t2LDREX(B|H)", "t2LDA(EX)?(B|H)$")>;
def : InstRW<[WriteST, M85SingleIssue, M85Read_EX2, M85Read_ISS],
               (instregex "t2STREX(B|H)?$", "t2STL(EX)?(B|H)?$")>;
def : InstRW<[M85WriteStWide, M85SingleIssue, M85Read_EX2, M85Read_EX2, M85Read_ISS],
               (instregex "t2STLEXD$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 265-265
```tablegen
// Load/store multiples end issue groups.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 267-274
```tablegen
def : InstRW<[M85WriteLdWide, M85SingleIssue, M85Read_ISS],
               (instregex "(t|t2)LDM(DB|IA)$")>;
def : InstRW<[M85WriteStWide, M85SingleIssue, M85Read_ISS],
               (instregex "(t|t2)STM(DB|IA)$")>;
def : InstRW<[M85BaseUpdate, M85WriteLdWide, M85SingleIssue, M85Read_ISS],
               (instregex "(t|t2)LDM(DB|IA)_UPD$", "tPOP")>;
def : InstRW<[M85BaseUpdate, M85WriteStWide, M85SingleIssue, M85Read_ISS],
               (instregex "(t|t2)STM(DB|IA)_UPD$", "tPUSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 276-276
```tablegen
// Load/store doubles
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 278-286
```tablegen
def : InstRW<[M85BaseUpdate, M85WriteStWide,
              M85Read_EX2, M85Read_EX2, M85Read_ISS],
               (instregex "t2STRD_(PRE|POST)")>;
def : InstRW<[M85WriteStWide, M85Read_EX2, M85Read_EX2, M85Read_ISS],
               (instregex "t2STRDi")>;
def : InstRW<[M85WriteLdWide, M85LoadLatency1, M85BaseUpdate, M85Read_ISS],
               (instregex "t2LDRD_(PRE|POST)")>;
def : InstRW<[M85WriteLdWide, M85LoadLatency1, M85Read_ISS],
               (instregex "t2LDRDi")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 288-296
```tablegen
// Word load / preload
def : InstRW<[WriteLd],
               (instregex "t2LDRpc", "t2PL[DI]pci", "tLDRpci")>;
def : InstRW<[WriteLd, M85Read_ISS],
               (instregex "t2LDR(i|T)", "t2PL[DI](W)?i", "tLDRi")>;
def : InstRW<[WriteLd, M85Read_ISS, M85Read_ISS],
               (instregex "t2LDRs", "t2PL[DI](w)?s", "tLDRr")>;
def : InstRW<[WriteLd, M85BaseUpdate, M85Read_ISS],
               (instregex "t2LDR_(POST|PRE)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 298-304
```tablegen
// Stores
def : InstRW<[M85BaseUpdate, WriteST, M85Read_EX2, M85Read_ISS],
               (instregex "t2STR(B|H)?_(POST|PRE)")>;
def : InstRW<[WriteST, M85Read_EX2, M85Read_ISS, M85Read_ISS],
               (instregex "t2STR(B|H)?s$", "tSTR(B|H)?r$")>;
def : InstRW<[WriteST, M85Read_EX2, M85Read_ISS],
               (instregex "t2STR(B|H)?(i|T)", "tSTR(B|H)?i$", "tSTRspi")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 306-306
```tablegen
// TBB/TBH - single-issue only
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 308-308
```tablegen
def M85TableLoad : SchedWriteRes<[M85UnitLoad]> { let SingleIssue = 1; }
```
- EN: Defines TableGen record `M85TableLoad` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85TableLoad`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 310-311
```tablegen
def : InstRW<[M85TableLoad, M85Read_ISS, M85Read_ISS],
                (instregex "t2TB")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 313-316
```tablegen
// VFP/MVE loads and stores
//   Note: timing for VLDR/VSTR special has not been broken out
//   Note 2: see notes at top of file for the reason load latency is 1 and
//           store data is in EX3.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 318-335
```tablegen
def M85LoadSP  : SchedWriteRes<[M85UnitLoad, M85UnitVPort]>;
def M85LoadDP  : SchedWriteRes<[M85UnitLoadL, M85UnitLoadH,
                                M85UnitVPortL, M85UnitVPortH]>;
def M85LoadSys  : SchedWriteRes<[M85UnitLoad, M85UnitVPort,
                                 M85UnitVFPA, M85UnitVFPB, M85UnitVFPC, M85UnitVFPD]> {
  let Latency = 4;
}
def M85StoreSP : SchedWriteRes<[M85UnitStore, M85UnitVPort]>;
def M85StoreDP : SchedWriteRes<[M85UnitStoreL, M85UnitStoreH,
                                M85UnitVPortL, M85UnitVPortH]>;
def M85StoreSys : SchedWriteRes<[M85UnitStore, M85UnitVPort,
                                 M85UnitVFPA, M85UnitVFPB, M85UnitVFPC, M85UnitVFPD]>;
let ReleaseAtCycles = [2,2,1,1], EndGroup = 1 in {
  def M85LoadMVE  : SchedWriteRes<[M85UnitLoadL, M85UnitLoadH,
                                   M85UnitVPortL, M85UnitVPortH]>;
  def M85LoadMVELate  : SchedWriteRes<[M85UnitLoadL, M85UnitLoadH,
                                       M85UnitVPortL, M85UnitVPortH]> {
    let Latency = 4; // 3 cycles later
```
- EN: Defines TableGen record `M85LoadSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85LoadSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 336-339
```tablegen
  }
  def M85StoreMVE : SchedWriteRes<[M85UnitStoreL, M85UnitStoreH,
                                   M85UnitVPortL, M85UnitVPortH]>;
}
```
- EN: Defines TableGen record `M85StoreMVE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85StoreMVE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 341-346
```tablegen
def : InstRW<[M85LoadSP, M85Read_ISS], (instregex "VLDR(S|H)$")>;
def : InstRW<[M85LoadSys, M85Read_ISS], (instregex "VLDR_")>;
def : InstRW<[M85LoadDP, M85Read_ISS], (instregex "VLDRD$")>;
def : InstRW<[M85StoreSP, M85Read_EX3, M85Read_ISS], (instregex "VSTR(S|H)$")>;
def : InstRW<[M85StoreSys, M85Read_EX1, M85Read_ISS], (instregex "VSTR_")>;
def : InstRW<[M85StoreDP, M85Read_EX3, M85Read_ISS], (instregex "VSTRD$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 348-361
```tablegen
def : InstRW<[M85LoadMVELate, M85Read_ISS],
               (instregex "MVE_VLD[24]._[0-9]+$")>;
def : InstRW<[M85LoadMVELate, M85MVERBaseUpdate, M85Read_ISS],
               (instregex "MVE_VLD[24].*wb")>;
def : InstRW<[M85LoadMVE, M85Read_ISS],
               (instregex "MVE_VLDR.*(8|16|32|64)$")>;
def : InstRW<[M85LoadMVE, M85SingleIssue, M85Read_ISS, M85Read_ISS],
               (instregex "MVE_VLDR.*(_rq|_rq|_rq_u)$")>;
def : InstRW<[M85LoadMVE, M85SingleIssue, M85Read_ISS],
               (instregex "MVE_VLDR.*_qi$")>;
def : InstRW<[M85MVERBaseUpdate, M85LoadMVE, M85Read_ISS],
               (instregex "MVE_VLDR.*(_post|[^i]_pre)$")>;
def : InstRW<[M85MVEQBaseUpdate, M85SingleIssue, M85LoadMVE, M85Read_ISS],
               (instregex "MVE_VLDR.*(qi_pre)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 363-377
```tablegen
def : InstRW<[M85StoreMVE, M85Read_EX3, M85Read_ISS],
               (instregex "MVE_VST[24]._[0-9]+$")>;
def : InstRW<[M85StoreMVE, M85Read_EX3, M85MVERBaseUpdate, M85Read_ISS],
               (instregex "MVE_VST[24].*wb")>;
def : InstRW<[M85StoreMVE, M85Read_EX3, M85Read_ISS],
               (instregex "MVE_VSTR.*(8|16|32|64)$")>;
def : InstRW<[M85StoreMVE, M85SingleIssue, M85Read_EX3, M85Read_ISS, M85Read_ISS],
               (instregex "MVE_VSTR.*(_rq|_rq|_rq_u)$")>;
def : InstRW<[M85StoreMVE, M85SingleIssue, M85Read_EX3, M85Read_ISS],
               (instregex "MVE_VSTR.*_qi$")>;
def : InstRW<[M85MVERBaseUpdate, M85StoreMVE, M85Read_EX3, M85Read_ISS],
               (instregex "MVE_VSTR.*(_post|[^i]_pre)$")>;
def : InstRW<[M85MVEQBaseUpdate, M85SingleIssue, M85StoreMVE,
              M85Read_EX3, M85Read_ISS],
               (instregex "MVE_VSTR.*(qi_pre)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 379-379
```tablegen
// Load/store multiples end issue groups.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 381-389
```tablegen
def : InstRW<[M85WriteLdWide, M85SingleIssue, M85Read_ISS],
               (instregex "VLDM(S|D|Q)(DB|IA)$")>;
def : InstRW<[M85WriteStWide, M85SingleIssue, M85Read_ISS, M85Read_EX3],
               (instregex "VSTM(S|D|Q)(DB|IA)$")>;
def : InstRW<[M85BaseUpdate, M85WriteLdWide, M85SingleIssue, M85Read_ISS],
               (instregex "VLDM(S|D|Q)(DB|IA)_UPD$", "VLLDM")>;
def : InstRW<[M85BaseUpdate, M85WriteStWide, M85SingleIssue,
              M85Read_ISS, M85Read_EX3],
               (instregex "VSTM(S|D|Q)(DB|IA)_UPD$", "VLSTM")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 391-393
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for ALU
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 395-396
```tablegen
// Non-small shifted ALU operands are read a cycle early; small LSLs
// aren't, as they don't require the shifter.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 398-400
```tablegen
def M85NonsmallShiftWrite : SchedWriteRes<[M85UnitALU,M85UnitShift1]> {
  let Latency = 1;
}
```
- EN: Defines TableGen record `M85NonsmallShiftWrite` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85NonsmallShiftWrite`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 402-409
```tablegen
def M85WriteALUsi : SchedWriteVariant<[
  SchedVar<NoSchedPred, [M85NonsmallShiftWrite]>
]>;
def M85Ex1ReadNoFastBypass : SchedReadAdvance<-1,
                                   [WriteLd, M85WriteLdWide, M85LoadLatency1]>;
def M85ReadALUsi : SchedReadVariant<[
  SchedVar<NoSchedPred, [M85Read_ISS]>
]>;
```
- EN: Defines TableGen record `M85WriteALUsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85WriteALUsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 411-415
```tablegen
def : InstRW<[M85WriteALUsi, M85Read_EX1, M85ReadALUsi],
               (instregex "t2(ADC|ADDS|BIC|EOR|ORN|ORR|RSBS|RSB|SBC|"
                          "SUBS|CMP|CMN|TEQ|TST)rs$")>;
def : InstRW<[M85WriteALUsi, M85ReadALUsi],
               (instregex "t2MVNs")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 417-424
```tablegen
// CortexM85 treats LSL #0 as needing a shifter. In practice the throughput
// seems to reliably be 2 when run on a cyclemodel, so we don't require a
// shift resource.
def : InstRW<[M85WriteALUsi, M85Read_EX1, M85ReadALUsi],
               (instregex "t2(ADC|ADDS|BIC|EOR|ORN|ORR|RSBS|RSB|SBC|"
                          "SUBS|CMP|CMN|TEQ|TST)rr$")>;
def : InstRW<[M85WriteALUsi, M85ReadALUsi],
               (instregex "t2MVNr")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 426-434
```tablegen
// Shift instructions: most pure shifts (i.e. MOV w/ shift) will use whichever
// shifter is free, thus it is possible to dual-issue them freely with anything
// else.  As a result, they are not modeled as needing a shifter.
// RRX is odd because it must use the EX2 shifter, so it cannot dual-issue with
// itself.
//
// Note that pure shifts which use the EX1 shifter would need their operands
// a cycle earlier.  However, they are only forced to use the EX1 shifter
// when issuing against an RRX instructions, which should be rare.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 436-439
```tablegen
def : InstRW<[M85WriteShift2],
               (instregex "t2RRX$")>;
def : InstRW<[WriteALU],
             (instregex "(t|t2)(LSL|LSR|ASR|ROR|SBFX|UBFX)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 441-441
```tablegen
// Instructions that use the shifter, but have normal timing
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 443-443
```tablegen
def : InstRW<[WriteALUsi,M85Slot0Only], (instregex "t2(BFC|BFI)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 445-445
```tablegen
// Stack pointer add/sub happens in EX1 with checks in EX2
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 447-447
```tablegen
def M85WritesToSPPred : MCSchedPredicate<CheckRegOperand<0, SP>>;
```
- EN: Defines TableGen record `M85WritesToSPPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85WritesToSPPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 449-456
```tablegen
def M85ReadForSP : SchedReadVariant<[
  SchedVar<M85WritesToSPPred, [M85Read_ISS]>,
  SchedVar<NoSchedPred, [M85Read_EX1]>
]>;
def M85ReadForSPShift : SchedReadVariant<[
  SchedVar<M85WritesToSPPred, [M85Read_ISS]>,
  SchedVar<NoSchedPred, [M85Read_ISS]>
]>;
```
- EN: Defines TableGen record `M85ReadForSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85ReadForSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 458-465
```tablegen
def : InstRW<[WriteALU, M85Read_ISS],
               (instregex "tADDspi", "tSUBspi")>;
def : InstRW<[WriteALU, M85ReadForSP],
               (instregex "t2(ADD|SUB)ri", "t2MOVr", "tMOVr")>;
def : InstRW<[WriteALU, M85ReadForSP, M85ReadForSP],
               (instregex "tADDrSP", "tADDspr", "tADDhirr")>;
def : InstRW<[M85WriteALUsi, M85ReadForSP, M85ReadForSPShift],
               (instregex "t2(ADD|SUB)rs")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 467-467
```tablegen
def : InstRW<[WriteALU, M85Slot0Only], (instregex "t2CLZ")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 469-469
```tablegen
// MAC operations that don't have SchedRW set
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 471-471
```tablegen
def : InstRW<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC], (instregex "t2SML[AS]D")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 473-476
```tablegen
// Divides are special because they stall for their latency, and so look like
// two cycles as far as scheduling opportunities go.  By putting M85Write2
// first, we make the operand latency 2, but keep the instruction latency 7.
// Divide operands are read early.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 478-479
```tablegen
def : InstRW<[M85Write2, WriteDIV, M85Read_ISS, M85Read_ISS, WriteALU],
               (instregex "t2(S|U)DIV")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 481-481
```tablegen
// DSP extension operations
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 483-500
```tablegen
def M85WriteSIMD1   : SchedWriteRes<[M85UnitSIMD, M85UnitALU, M85UnitSlot0]> {
  let Latency = 1;
}
def M85WriteSIMD2   : SchedWriteRes<[M85UnitSIMD, M85UnitALU, M85UnitSlot0]> {
  let Latency = 2;
}
def M85WriteShSIMD0 : SchedWriteRes<[M85UnitSIMD, M85UnitALU,
                                       M85UnitShift1, M85UnitSlot0]> {
  let Latency = 0; // Finishes at EX1
}
def M85WriteShSIMD1 : SchedWriteRes<[M85UnitSIMD, M85UnitALU,
                                       M85UnitShift1, M85UnitSlot0]> {
  let Latency = 1;
}
def M85WriteShSIMD2 : SchedWriteRes<[M85UnitSIMD, M85UnitALU,
                                     M85UnitShift1, M85UnitSlot0]> {
  let Latency = 2;
}
```
- EN: Defines TableGen record `M85WriteSIMD1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85WriteSIMD1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 502-518
```tablegen
def : InstRW<[M85WriteShSIMD2, M85Read_ISS],
               (instregex "t2(S|U)SAT")>;
def : InstRW<[M85WriteSIMD1, ReadALU],
               (instregex "(t|t2)(S|U)XT(B|H)")>;
def : InstRW<[M85WriteSIMD1, ReadALU, ReadALU],
               (instregex "t2(S|SH|U|UH)(ADD16|ADD8|ASX|SAX|SUB16|SUB8)",
                          "t2SEL")>;
def : InstRW<[M85WriteSIMD2, ReadALU, ReadALU],
               (instregex "t2(Q|UQ)(ADD|ASX|SAX|SUB)", "t2USAD8")>;
def : InstRW<[M85WriteShSIMD2, M85Read_ISS, M85Read_ISS],
               (instregex "t2QD(ADD|SUB)")>;
def : InstRW<[M85WriteShSIMD0, M85Read_ISS],
               (instregex "t2(RBIT|REV)", "tREV")>;
def : InstRW<[M85WriteShSIMD1, ReadALU, M85Read_ISS],
               (instregex "t2PKH(BT|TB)", "t2(S|U)XTA")>;
def : InstRW<[M85WriteSIMD2, ReadALU, ReadALU, M85Read_EX2],
               (instregex "t2USADA8")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 520-521
```tablegen
// MSR/MRS
def : InstRW<[M85NonGeneralPurpose], (instregex "MSR", "MRS")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 523-523
```tablegen
// 64-bit shift operations in EX3
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 525-528
```tablegen
def M85WriteLShift : SchedWriteRes<[M85UnitLShift, M85UnitALU]> {
  let Latency = 2;
}
def M85WriteLat2  : SchedWriteRes<[]>  { let Latency = 2; let NumMicroOps = 0; }
```
- EN: Defines TableGen record `M85WriteLShift` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85WriteLShift`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 530-538
```tablegen
def : InstRW<[M85WriteLShift, M85WriteLat2, M85Read_EX2, M85Read_EX2],
               (instregex "MVE_(ASRLi|LSLLi|LSRL|SQSHLL|SRSHRL|UQSHLL|URSHRL)$")>;
def : InstRW<[M85WriteLShift, M85WriteLat2,
              M85Read_EX2, M85Read_EX2, M85Read_EX2],
               (instregex "MVE_(ASRLr|LSLLr|SQRSHRL|UQRSHLL)$")>;
def : InstRW<[M85WriteLShift, M85Read_EX2, M85Read_EX2],
               (instregex "MVE_(SQRSHR|UQRSHL)$")>;
def : InstRW<[M85WriteLShift, M85Read_EX2],
               (instregex "MVE_(SQSHL|SRSHR|UQSHL|URSHR)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 540-540
```tablegen
// Loop control/branch future instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 542-542
```tablegen
def M85LE   : SchedWriteRes<[]> { let NumMicroOps = 0; let Latency = -2; }
```
- EN: Defines TableGen record `M85LE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85LE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 544-544
```tablegen
def : InstRW<[WriteALU], (instregex "t2BF(_|Lr|i|Li|r)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 546-551
```tablegen
def : InstRW<[WriteALU], (instregex "MVE_LCTP")>;
def : InstRW<[WriteALU],
               (instregex "t2DLS", "t2WLS", "MVE_DLSTP", "MVE_WLSTP")>;
def : InstRW<[M85LE], (instregex "t2LE$")>;
def : InstRW<[M85LE, M85Read_ISSm1],
               (instregex "t2LEUpdate", "MVE_LETP")>;  // LE is executed at ISS
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 553-553
```tablegen
// Conditional selects
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 555-556
```tablegen
def : InstRW<[M85WriteLShift, M85Read_EX2, M85Read_EX2, M85Read_EX2],
              (instregex "t2(CSEL|CSINC|CSINV|CSNEG)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 558-559
```tablegen
//===---------------------------------------------------------------------===//
// Sched definitions for FP and MVE operations
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 561-566
```tablegen
let NumMicroOps = 0 in {
  def M85OverrideVFPLat5 : SchedWriteRes<[]> { let Latency = 5; }
  def M85OverrideVFPLat4 : SchedWriteRes<[]> { let Latency = 4; }
  def M85OverrideVFPLat3 : SchedWriteRes<[]> { let Latency = 3; }
  def M85OverrideVFPLat2 : SchedWriteRes<[]> { let Latency = 2; }
}
```
- EN: Defines TableGen record `M85OverrideVFPLat5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85OverrideVFPLat5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 568-582
```tablegen
let Latency = 1 in {
   def M85GroupALat1S : SchedWriteRes<[M85UnitVFPA, M85UnitVPort, M85UnitSlot0]>;
   def M85GroupBLat1S : SchedWriteRes<[M85UnitVFPB, M85UnitVPort, M85UnitSlot0]>;
   def M85GroupCLat1S : SchedWriteRes<[M85UnitVFPC, M85UnitVPort, M85UnitSlot0]>;
   def M85GroupALat1D : SchedWriteRes<[M85UnitVFPAL, M85UnitVFPAH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GroupBLat1D : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GroupCLat1D : SchedWriteRes<[M85UnitVFPCL, M85UnitVFPCH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GroupABLat1S : SchedWriteRes<[M85UnitVPort, M85UnitSlot0]>;
}
let Latency = 2 in {
   def M85GroupBLat2S : SchedWriteRes<[M85UnitVFPB, M85UnitVPort, M85UnitSlot0]>;
   def M85GroupBLat2D : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GroupABLat2S : SchedWriteRes<[M85UnitVPort, M85UnitSlot0]>;
   def M85GroupABLat2D : SchedWriteRes<[M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
}
```
- EN: Defines TableGen record `M85GroupALat1S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85GroupALat1S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 584-598
```tablegen
// Instructions which are missing default schedules
def : InstRW<[M85GroupALat1S],  (instregex "V(FP_VMAXNM|FP_VMINNM)(H|S)$")>;
def : InstRW<[M85GroupALat1D],  (instregex "V(FP_VMAXNM|FP_VMINNM)D$")>;
def : InstRW<[M85GroupCLat1S],  (instregex "VCMPE?Z?(H|S)$")>;
def : InstRW<[M85GroupCLat1D],  (instregex "VCMPE?Z?D$")>;
def : InstRW<[M85GroupBLat2S],
               (instregex "VCVT(A|M|N|P|R|X|Z)(S|U)(H|S)",
                          "VRINT(A|M|N|P|R|X|Z)(H|S)")>;
def : InstRW<[M85GroupBLat2D],
               (instregex "VCVT(B|T)(DH|HD)", "VCVT(A|M|N|P|R|X|Z)(S|U)D",
                          "V.*TOD", "VTO.*D", "VCVTDS", "VCVTSD",
                          "VRINT(A|M|N|P|R|X|Z)D")>;
def : InstRW<[M85GroupABLat1S], (instregex "VINSH")>;
def : InstRW<[M85GroupBLat1S],  (instregex "V(ABS|NEG)(H|S)$")>;
def : InstRW<[M85GroupBLat1D],  (instregex "V(ABS|NEG)D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 600-606
```tablegen
// VMRS/VMSR
let SingleIssue = 1 in {
  def M85VMRSEarly : SchedWriteRes<[M85UnitVPort]> { let Latency = 2;}
  def M85VMRSLate  : SchedWriteRes<[M85UnitVPort]> { let Latency = 4; }
  def M85VMSREarly : SchedWriteRes<[M85UnitVPort]> { let Latency = 1; }
  def M85VMSRLate  : SchedWriteRes<[M85UnitVPort]> { let Latency = 3; }
}
```
- EN: Defines TableGen record `M85VMRSEarly` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85VMRSEarly`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 608-610
```tablegen
def M85FPSCRFlagPred : MCSchedPredicate<
                           CheckAll<[CheckIsRegOperand<0>,
                                     CheckRegOperand<0, PC>]>>;
```
- EN: Defines TableGen record `M85FPSCRFlagPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85FPSCRFlagPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 612-615
```tablegen
def M85VMRSFPSCR : SchedWriteVariant<[
  SchedVar<M85FPSCRFlagPred, [M85VMRSEarly]>,
  SchedVar<NoSchedPred, [M85VMRSLate]>
]>;
```
- EN: Defines TableGen record `M85VMRSFPSCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85VMRSFPSCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 617-624
```tablegen
def : InstRW<[M85VMSREarly, M85Read_EX2],
               (instregex "VMSR$", "VMSR_FPSCR_NZCVQC", "VMSR_P0", "VMSR_VPR")>;
def : InstRW<[M85VMRSEarly], (instregex "VMRS_P0", "VMRS_VPR", "FMSTAT")>;
def : InstRW<[M85VMRSLate], (instregex "VMRS_FPSCR_NZCVQC")>;
def : InstRW<[M85VMRSFPSCR], (instregex "VMRS$")>;
// Not matching properly
//def : InstRW<[M85VMSRLate, M85Read_EX2], (instregex "VMSR_FPCTX(NS|S)")>;
//def : InstRW<[M85VMRSLate], (instregex "VMRS_FPCTX(NS|S)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 626-630
```tablegen
// VSEL cannot bypass in its implied $cpsr operand; model as earlier read
def : InstRW<[M85GroupBLat1S, ReadALU, ReadALU, M85Read_ISS],
               (instregex "VSEL.*(S|H)$")>;
def : InstRW<[M85GroupBLat1D, ReadALU, ReadALU, M85Read_ISS],
               (instregex "VSEL.*D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 632-648
```tablegen
// VMOV
def : InstRW<[WriteFPMOV],
               (instregex "VMOV(H|S)$", "FCONST(H|S)")>;
def : InstRW<[WriteFPMOV, M85Read_EX2],
               (instregex "VMOVHR$", "VMOVSR$")>;
def : InstRW<[M85GroupABLat2S],
               (instregex "VMOVRH$", "VMOVRS$")>;
def : InstRW<[M85WriteFPMOV64],
               (instregex "VMOVD$")>;
def : InstRW<[M85WriteFPMOV64],
               (instregex "FCONSTD")>;
def : InstRW<[M85WriteFPMOV64, M85Read_EX2, M85Read_EX2],
               (instregex "VMOVDRR")>;
def : InstRW<[M85WriteFPMOV64, M85Write1, M85Read_EX2, M85Read_EX2],
               (instregex "VMOVSRR")>;
def : InstRW<[M85GroupABLat2D, M85Write2],
               (instregex "VMOV(RRD|RRS)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 650-654
```tablegen
// These shouldn't even exist, but Cortex-m55 defines them, so here they are.
def : InstRW<[WriteFPMOV, M85Read_EX2],
               (instregex "VGETLNi32$")>;
def : InstRW<[M85GroupABLat2S],
               (instregex "VSETLNi32")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 656-656
```tablegen
// Larger-latency overrides
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 658-668
```tablegen
def M85FPDIV16 : SchedWriteRes<[M85UnitVFPB, M85UnitVPort, M85UnitSlot0]> {
  let Latency = 8;
}
def : InstRW<[M85OverrideVFPLat2, M85FPDIV16], (instregex "VDIVH")>;
def : InstRW<[M85OverrideVFPLat2, WriteFPDIV32],   (instregex "VDIVS")>;
def : InstRW<[M85OverrideVFPLat2, WriteFPDIV64],   (instregex "VDIVD")>;
def : InstRW<[M85OverrideVFPLat2, M85FPDIV16], (instregex "VSQRTH")>;
def : InstRW<[M85OverrideVFPLat2, WriteFPSQRT32],  (instregex "VSQRTS")>;
def : InstRW<[M85OverrideVFPLat2, WriteFPSQRT64],  (instregex "VSQRTD")>;
def : InstRW<[M85OverrideVFPLat3, WriteFPMUL64],   (instregex "V(MUL|NMUL)D")>;
def : InstRW<[M85OverrideVFPLat2, WriteFPALU64],   (instregex "V(ADD|SUB)D")>;
```
- EN: Defines TableGen record `M85FPDIV16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85FPDIV16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 670-672
```tablegen
// Multiply-accumulate.  Chained SP timing is correct; rest need overrides
// Double-precision chained MAC should also be seen as having latency of 5,
// as stalls stall everything.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 674-675
```tablegen
def : InstRW<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL],
               (instregex "VN?ML(A|S)H")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 677-679
```tablegen
def : InstRW<[M85OverrideVFPLat5, WriteFPMAC64,
              ReadFPMUL, ReadFPMUL, ReadFPMUL],
               (instregex "VN?ML(A|S)D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 681-681
```tablegen
// Single-precision fused MACs look like latency 4 with advance of 2.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 683-683
```tablegen
def M85ReadFPMAC2   : SchedReadAdvance<2>;
```
- EN: Defines TableGen record `M85ReadFPMAC2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85ReadFPMAC2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 685-687
```tablegen
def : InstRW<[M85OverrideVFPLat4, WriteFPMAC32,
              M85ReadFPMAC2, ReadFPMUL, ReadFPMUL],
               (instregex "VF(N)?M(A|S)(H|S)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 689-689
```tablegen
// Double-precision fused MAC looks like latency 4.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 691-693
```tablegen
def : InstRW<[M85OverrideVFPLat4, WriteFPMAC64,
              ReadFPMUL, ReadFPMUL, ReadFPMUL],
               (instregex "VF(N)?M(A|S)D$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 695-701
```tablegen
// MVE beatwise instructions
// NOTE: Q-register timing for the 2nd beat is off by a cycle and needs
//       DAG overrides to correctly set latencies.
// NOTE2: MVE integer MAC->MAC accumulate latencies are set as if the
//        accumulate value arrives from an unmatching MAC instruction;
//        matching ones are handled via DAG mutation.  These are marked as
//        "limited accumulate bypass"
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 703-720
```tablegen
let Latency = 4, EndGroup = 1 in {
   def M85GrpALat2MveR : SchedWriteRes<[M85UnitVFPAL, M85UnitVFPAH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85GrpABLat2MveR : SchedWriteRes<[M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GrpBLat2MveR : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85Lat2MveR : SchedWriteRes<[]> { let NumMicroOps = 0; }
   def M85GrpBLat4Mve : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
}
let Latency = 3, EndGroup = 1 in {
   def M85GrpBLat3Mve : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85GrpBLat1MveR : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
```
- EN: Defines TableGen record `M85GrpALat2MveR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85GrpALat2MveR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 721-738
```tablegen
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85Lat1MveR : SchedWriteRes<[]> { let NumMicroOps = 0; }
}
let Latency = 2, EndGroup = 1 in {
   def M85GrpALat2Mve : SchedWriteRes<[M85UnitVFPAL, M85UnitVFPAH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85GrpABLat2Mve : SchedWriteRes<[M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GrpBLat2Mve : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85Lat2Mve : SchedWriteRes<[]> { let NumMicroOps = 0; }
}
let Latency = 1, EndGroup = 1 in {
   def M85GrpALat1Mve : SchedWriteRes<[M85UnitVFPAL, M85UnitVFPAH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
```
- EN: Defines TableGen record `M85Lat1MveR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85Lat1MveR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 739-749
```tablegen
   def M85GrpABLat1Mve : SchedWriteRes<[M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]>;
   def M85GrpBLat1Mve : SchedWriteRes<[M85UnitVFPBL, M85UnitVFPBH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85GrpCLat1Mve : SchedWriteRes<[M85UnitVFPCL, M85UnitVFPCH, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,2,1,1,1];
   }
   def M85GrpDLat1Mve : SchedWriteRes<[M85UnitVFPD, M85UnitVPortL, M85UnitVPortH, M85UnitSlot0]> {
     let ReleaseAtCycles = [2,1,1,1];
   }
}
```
- EN: Defines TableGen record `M85GrpABLat1Mve` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M85GrpABLat1Mve`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 751-752
```tablegen
def : InstRW<[M85GrpABLat1Mve, M85Read_EX1, M85Read_EX2, M85Read_EX2],
                (instregex "MVE_VMOV_q_rr")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 754-755
```tablegen
def : InstRW<[M85GrpABLat1Mve, M85Read_EX2],
                (instregex "MVE_VMOV_to_lane_(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 757-769
```tablegen
def : InstRW<[M85GrpABLat1Mve],
                (instregex "MVE_VAND$",
                           "MVE_VBIC$", "MVE_VBICimm",
                           "MVE_VCLSs(8|16|32)",
                           "MVE_VCLZs(8|16|32)",
                           "MVE_VEOR",
                           "MVE_VMOVimmf32", "MVE_VMOVimmi(8|16|32|64)",
                           "MVE_VMVN$", "MVE_VMVNimmi(16|32)",
                           "MVE_VORN$",
                           "MVE_VORR$", "MVE_VORRimm", "MQPRCopy",
                           "MVE_VPSEL",
                           "MVE_VREV(16|32|64)_(8|16|32)"
                           )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 771-772
```tablegen
def : InstRW<[M85GrpABLat2MveR, M85Lat2MveR],
                (instregex "MVE_VMOV_rr_q")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 774-775
```tablegen
def : InstRW<[M85GrpABLat2MveR],
                (instregex "MVE_VMOV_from_lane_(32|u8|s8|u16|s16)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 777-779
```tablegen
def : InstRW<[M85GrpALat1Mve, M85Lat1MveR,
              M85Read_EX1, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VADC$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 781-782
```tablegen
def : InstRW<[M85GrpALat1Mve, M85Lat1MveR],
                (instregex "MVE_VADCI")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 784-793
```tablegen
def : InstRW<[M85GrpALat1Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VADD_qr_i(8|16|32)",
                           "MVE_VBRSR(16|32|8)",
                           "MVE_VHADD_qr_[su](8|16|32)",
                           "MVE_VHSUB_qr_[su](8|16|32)",
                           "MVE_VQADD_qr_[su](8|16|32)",
                           "MVE_VQSUB_qr_[su](8|16|32)",
                           "MVE_VSHL_qr[su](8|16|32)",
                           "MVE_VSUB_qr_i(8|16|32)"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 795-812
```tablegen
def : InstRW<[M85GrpALat1Mve],
                (instregex "MVE_VABD(s|u)(8|16|32)",
                           "MVE_VABS(s|u)(8|16|32)",
                           "MVE_V(MAX|MIN)A?[us](8|16|32)",
                           "MVE_VADDi(8|16|32)",
                           "MVE_VCADDi(8|16|32)",
                           "MVE_VHCADDs(8|16|32)",
                           "MVE_VHSUB[su](8|16|32)",
                           "MVE_VMOVL[su](8|16)[tb]h",
                           "MVE_VMOVNi(16|32)[tb]h",
                           "MVE_VMULL[BT]?[p](8|16|32)(bh|th)?",
                           "MVE_VNEGs(8|16|32)",
                           "MVE_VQABSs(8|16|32)",
                           "MVE_VQADD[su](8|16|32)",
                           "MVE_VQNEGs(8|16|32)",
                           "MVE_VQSUB[su](8|16|32)",
                           "MVE_VR?HADD[su](8|16|32)",
                           "MVE_VSBC$", "MVE_VSBCI",
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 813-822
```tablegen
                           "MVE_VSHL_by_vec[su](8|16|32)",
                           "MVE_VSHL_immi(8|16|32)",
                           "MVE_VSHLL_imm[su](8|16)[bt]h",
                           "MVE_VSHLL_lw[su](8|16)[bt]h",
                           "MVE_VSHRNi(16|32)[bt]h",
                           "MVE_VSHR_imm[su](8|16|32)",
                           "MVE_VSLIimm[su]?(8|16|32)",
                           "MVE_VSRIimm[su]?(8|16|32)",
                           "MVE_VSUBi(8|16|32)"
                 )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 824-825
```tablegen
def : InstRW<[M85GrpALat2Mve, M85Lat2MveR, M85Read_EX2, M85Read_EX2],
                (instregex "MVE_V(D|I)WDUPu(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 827-828
```tablegen
def : InstRW<[M85GrpALat2Mve, M85Lat2MveR, M85Read_EX2],
                (instregex "MVE_V(D|I)DUPu(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 830-834
```tablegen
def : InstRW<[M85GrpALat2Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_V(Q|R|QR)SHL_qr[su](8|16|32)",
                           "MVE_VADD_qr_f(16|32)",
                           "MVE_VSUB_qr_f(16|32)"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 836-837
```tablegen
def : InstRW<[M85GrpALat1Mve, M85Read_EX2],
                (instregex "MVE_VDUP(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 839-843
```tablegen
def : InstRW<[M85GrpBLat1Mve],
                (instregex "MVE_VABSf(16|32)",
                           "MVE_V(MAX|MIN)NMA?f(16|32)",
                           "MVE_VNEGf(16|32)"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 845-846
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Lat2MveR, M85Read_EX3, M85Read_EX3],
                (instregex "MVE_VADDLV[us]32acc")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 848-849
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Lat2MveR],
                (instregex "MVE_VADDLV[us]32no_acc")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 851-853
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Read_EX3],
                (instregex "MVE_VADDV[us](8|16|32)acc"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 855-858
```tablegen
def : InstRW<[M85GrpALat2MveR, M85Read_EX3],
                (instregex "MVE_V(MAX|MIN)A?V[us](8|16|32)",
                           "MVE_VABAV(s|u)(8|16|32)"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 860-861
```tablegen
def : InstRW<[M85GrpALat2MveR],
                (instregex "MVE_VADDV[us](8|16|32)no_acc")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 863-875
```tablegen
def : InstRW<[M85GrpALat2Mve],
                (instregex "MVE_V(Q|R|QR)SHL_by_vec[su](8|16|32)",
                           "MVE_VABDf(16|32)",
                           "MVE_VADDf(16|32)",
                           "MVE_VCADDf(16|32)",
                           "MVE_VQMOVU?N[su](8|16|32)[tb]h",
                           "MVE_VQR?SHL(U_)?imm[su](8|16|32)",
                           "MVE_VQR?SHRN[bt]h[su](16|32)",
                           "MVE_VQR?SHRUNs(16|32)[bt]h",
                           "MVE_VRSHR_imm[su](8|16|32)",
                           "MVE_VRSHRNi(16|32)[bt]h",
                           "MVE_VSUBf(16|32)"
                 )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 877-878
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Read_EX2],
                (instregex "MVE_V(MAX|MIN)NMA?Vf(16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 880-881
```tablegen
def : InstRW<[M85GrpBLat2Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VMUL_qr_i(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 883-884
```tablegen
def : InstRW<[M85GrpBLat2Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VQDMULL_qr_s(16|32)[tb]h")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 886-887
```tablegen
def : InstRW<[M85GrpBLat2Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VQR?DMULH_qr_s(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 889-891
```tablegen
def : InstRW<[M85GrpBLat2Mve, M85Read_EX1, M85Read_EX1, M85Read_EX3],
                // limited accumulate bypass
                (instregex "MVE_VMLAS?_qr_i(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 893-895
```tablegen
def : InstRW<[M85GrpBLat2Mve, M85Read_EX1, M85Read_EX1, M85Read_EX2],
                // limited accumulate bypass
                (instregex "MVE_VQR?DMLAS?H_qrs(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 897-899
```tablegen
def : InstRW<[M85GrpBLat2Mve],
                // limited accumulate bypass
                (instregex "MVE_VQR?DML[AS]DHX?s(8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 901-902
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Lat2MveR, M85Read_EX3, M85Read_EX3],
                (instregex "MVE_VR?ML[AS]LDAVH?ax?[su](8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 904-905
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Lat2MveR],
                (instregex "MVE_VR?ML[AS]LDAVH?x?[su](8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 907-908
```tablegen
def : InstRW<[M85GrpBLat2MveR, M85Read_EX3],
                (instregex "MVE_VML[AS]DAVax?[su](8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 910-911
```tablegen
def : InstRW<[M85GrpBLat2MveR],
                (instregex "MVE_VML[AS]DAVx?[su](8|16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 913-923
```tablegen
def : InstRW<[M85GrpBLat2Mve],
                (instregex "MVE_VCVTf16(u|s)16", "MVE_VCVTf32(u|s)32",
                           "MVE_VCVT(u|s)16f16", "MVE_VCVT(u|s)32f32",
                           "MVE_VCVTf16f32", "MVE_VCVTf32f16",
                           "MVE_VMULL[BT]?[su](8|16|32)(bh|th)?",
                           "MVE_VMUL(t1)*i(8|16|32)",
                           "MVE_VQDMULLs(16|32)[tb]h",
                           "MVE_VQR?DMULHi(8|16|32)",
                           "MVE_VR?MULH[su](8|16|32)",
                           "MVE_VRINTf(16|32)"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 925-926
```tablegen
def : InstRW<[M85GrpBLat3Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VMUL_qr_f(16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 928-931
```tablegen
def : InstRW<[M85GrpBLat3Mve],
                (instregex "MVE_VCMULf(16|32)",
                           "MVE_VMULf(16|32)"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 933-936
```tablegen
def : InstRW<[M85GrpBLat4Mve, M85Read_EX3, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VFMA_qr_Sf(16|32)", // VFMAS
                           "MVE_VFMA_qr_f(16|32)" // VFMA
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 938-939
```tablegen
def : InstRW<[M85GrpBLat4Mve, M85Read_EX3],
                (instregex "MVE_VCMLAf(16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 941-942
```tablegen
def : InstRW<[M85GrpBLat4Mve, M85Read_EX3],
                (instregex "MVE_VFM(A|S)f(16|32)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 944-945
```tablegen
def : InstRW<[M85GrpCLat1Mve, M85Read_EX1, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VPTv(4|8)f(16|32)r")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 947-948
```tablegen
def : InstRW<[M85GrpCLat1Mve, M85Read_EX1, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VPTv(4|8|16)(i|s|u)(8|16|32)r")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 950-951
```tablegen
def : InstRW<[M85GrpCLat1Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VCMP[isu](8|16|32)r$", "MVE_VCMPf(16|32)r$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 953-954
```tablegen
def : InstRW<[M85GrpDLat1Mve, M85Read_EX2],
                (instregex "MVE_VCTP(8|16|32|64)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 956-960
```tablegen
def : InstRW<[M85GrpCLat1Mve],
                (instregex "MVE_VCMPf(16|32)$", "MVE_VCMP[isu](8|16|32)$",
                           "MVE_VPTv(4|8)f(16|32)$",
                           "MVE_VPTv(4|8|16)(i|s|u)(8|16|32)$"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 962-965
```tablegen
def : InstRW<[M85GrpDLat1Mve],
                (instregex "MVE_VPNOT",
                           "MVE_VPST"
                )>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 967-968
```tablegen
def : InstRW<[M85Lat2MveR, M85GrpALat2Mve, M85Read_EX1, M85Read_EX2],
                (instregex "MVE_VSHLC")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 970-970
```tablegen
// VFP instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 972-979
```tablegen
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

### Lines 981-981
```tablegen
}  // SchedModel = CortexCortexM85Model
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
