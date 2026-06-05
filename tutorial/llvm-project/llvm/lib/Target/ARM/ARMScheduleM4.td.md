# ARMScheduleM4.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleM4.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines the SchedRead/Write data for the ARM Cortex-M4 processor.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleM4`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//==- ARMScheduleM4.td - Cortex-M4 Scheduling Definitions -*- tablegen -*-====//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the SchedRead/Write data for the ARM Cortex-M4 processor.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-18
```tablegen
def CortexM4Model : SchedMachineModel {
  let IssueWidth        = 1; // Only IT can be dual-issued, so assume single-issue
  let MicroOpBufferSize = 0; // In-order
  let LoadLatency       = 2; // Latency when not pipelined, not pc-relative
  let MispredictPenalty = 2; // Best case branch taken cost
  let PostRAScheduler   = 1;
```
- EN: Defines TableGen record `CortexM4Model` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CortexM4Model`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 20-24
```tablegen
  let CompleteModel = 0;
  let UnsupportedFeatures = [IsARM, HasNEON, HasDotProd, HasZCZ, HasMVEInt,
          IsNotMClass, HasDPVFP, HasFPARMv8, HasFullFP16, Has8MSecExt, HasV8,
          HasV8_3a, HasTrustZone, HasDFB, IsWindows];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 27-28
```tablegen
// We model the entire cpu as a single pipeline with a BufferSize = 0 since
// Cortex-M4 is in-order.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 30-30
```tablegen
def M4Unit : ProcResource<1> { let BufferSize = 0; }
```
- EN: Defines TableGen record `M4Unit` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `M4Unit`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 33-33
```tablegen
let SchedModel = CortexM4Model in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 35-35
```tablegen
// Some definitions of latencies we apply to different instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 37-44
```tablegen
class M4UnitL1<SchedWrite write> : WriteRes<write, [M4Unit]> { let Latency = 1; }
class M4UnitL2<SchedWrite write> : WriteRes<write, [M4Unit]> { let Latency = 2; }
class M4UnitL3<SchedWrite write> : WriteRes<write, [M4Unit]> { let Latency = 3; }
class M4UnitL14<SchedWrite write> : WriteRes<write, [M4Unit]> { let Latency = 14; }
def M4UnitL1_wr : SchedWriteRes<[M4Unit]> { let Latency = 1; }
def M4UnitL2_wr : SchedWriteRes<[M4Unit]> { let Latency = 2; }
class M4UnitL1I<dag instr> : InstRW<[M4UnitL1_wr], instr>;
class M4UnitL2I<dag instr> : InstRW<[M4UnitL2_wr], instr>;
```
- EN: Declares reusable TableGen class `M4UnitL1` for `ARMScheduleM4`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMScheduleM4` 声明可复用的 TableGen 类 `M4UnitL1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 47-53
```tablegen
// Loads, MAC's and DIV all get a higher latency of 2
def : M4UnitL2<WriteLd>;
def : M4UnitL2<WriteMAC32>;
def : M4UnitL2<WriteMAC64Hi>;
def : M4UnitL2<WriteMAC64Lo>;
def : M4UnitL2<WriteMAC16>;
def : M4UnitL2<WriteDIV>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 55-56
```tablegen
def : M4UnitL2I<(instregex "(t|t2)LDM")>;
def : M4UnitL2I<(instregex "(t|t2)LDR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 59-59
```tablegen
// Stores we use a latency of 1 as they have no outputs
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 61-62
```tablegen
def : M4UnitL1<WriteST>;
def : M4UnitL1I<(instregex "(t|t2)STM")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 65-65
```tablegen
// Everything else has a Latency of 1
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 67-84
```tablegen
def : M4UnitL1<WriteALU>;
def : M4UnitL1<WriteALUsi>;
def : M4UnitL1<WriteALUsr>;
def : M4UnitL1<WriteALUSsr>;
def : M4UnitL1<WriteBr>;
def : M4UnitL1<WriteBrL>;
def : M4UnitL1<WriteBrTbl>;
def : M4UnitL1<WriteCMPsi>;
def : M4UnitL1<WriteCMPsr>;
def : M4UnitL1<WriteCMP>;
def : M4UnitL1<WriteMUL32>;
def : M4UnitL1<WriteMUL64Hi>;
def : M4UnitL1<WriteMUL64Lo>;
def : M4UnitL1<WriteMUL16>;
def : M4UnitL1<WriteNoop>;
def : M4UnitL1<WritePreLd>;
def : M4UnitL1I<(instregex "(t|t2)MOV")>;
def : M4UnitL1I<(instrs COPY)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 85-88
```tablegen
def : M4UnitL1I<(instregex "t2IT", "t2MSR", "t2MRS")>;
def : M4UnitL1I<(instregex "t2CLREX")>;
def : M4UnitL1I<(instregex "t2SEL", "t2USAD8", "t2SML[AS]",
    "t2(S|Q|SH|U|UQ|UH|QD)(ADD|ASX|SAX|SUB)", "t2USADA8", "(t|t2)REV")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 90-98
```tablegen
// These instructions are not of much interest to scheduling as they will not
// be generated or it is not very useful to schedule them. They are here to make
// the model more complete.
def : M4UnitL1I<(instregex "t2CDP", "t2LDC", "t2MCR", "t2MRC", "t2MRRC", "t2STC")>;
def : M4UnitL1I<(instregex "tCPS", "t2ISB", "t2DSB", "t2DMB", "t2?HINT$")>;
def : M4UnitL1I<(instregex "t2?UDF$", "tBKPT", "t2DBG")>;
def : M4UnitL1I<(instregex "t?2?Int_eh_sjlj_", "tADDframe", "t?ADJCALL")>;
def : M4UnitL1I<(instregex "CMP_SWAP", "JUMPTABLE", "MEMCPY")>;
def : M4UnitL1I<(instregex "VSETLNi32", "VGETLNi32")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 100-103
```tablegen
def : ReadAdvance<ReadALU, 0>;
def : ReadAdvance<ReadALUsr, 0>;
def : ReadAdvance<ReadMUL, 0>;
def : ReadAdvance<ReadMAC, 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 105-106
```tablegen
// Most FP instructions are single-cycle latency, except MAC's, Div's and Sqrt's.
// Loads still take 2 cycles.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 108-125
```tablegen
def : M4UnitL1<WriteFPCVT>;
def : M4UnitL1<WriteFPMOV>;
def : M4UnitL1<WriteFPALU32>;
def : M4UnitL1<WriteFPALU64>;
def : M4UnitL1<WriteFPMUL32>;
def : M4UnitL1<WriteFPMUL64>;
def : M4UnitL2I<(instregex "VLD")>;
def : M4UnitL1I<(instregex "VST")>;
def : M4UnitL3<WriteFPMAC32>;
def : M4UnitL3<WriteFPMAC64>;
def : M4UnitL14<WriteFPDIV32>;
def : M4UnitL14<WriteFPDIV64>;
def : M4UnitL14<WriteFPSQRT32>;
def : M4UnitL14<WriteFPSQRT64>;
def : M4UnitL1<WriteVLD1>;
def : M4UnitL1<WriteVLD2>;
def : M4UnitL1<WriteVLD3>;
def : M4UnitL1<WriteVLD4>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 126-132
```tablegen
def : M4UnitL1<WriteVST1>;
def : M4UnitL1<WriteVST2>;
def : M4UnitL1<WriteVST3>;
def : M4UnitL1<WriteVST4>;
def : M4UnitL1I<(instregex "VMOVS", "FCONSTS", "VCMP", "VNEG", "VABS")>;
def : M4UnitL2I<(instregex "VMOVD")>;
def : M4UnitL1I<(instregex "VMRS", "VMSR", "FMSTAT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 134-135
```tablegen
def : ReadAdvance<ReadFPMUL, 0>;
def : ReadAdvance<ReadFPMAC, 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 137-137
```tablegen
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
