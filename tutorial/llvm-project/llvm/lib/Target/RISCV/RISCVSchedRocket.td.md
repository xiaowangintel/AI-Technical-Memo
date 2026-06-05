# RISCVSchedRocket.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedRocket.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//==- RISCVSchedRocket.td - Rocket Scheduling Definitions ----*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// ===---------------------------------------------------------------------===//
// The following definitions describe the simpler per-operand machine model.
// This works with MachineScheduler. See MCSchedule.h for details.
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-24: TableGen record RocketModel / TableGen 记录 RocketModel
```tablegen
// Rocket machine model for scheduling and other instruction cost heuristics.
def RocketModel : SchedMachineModel {
  let MicroOpBufferSize = 0; // Rocket is in-order.
  let IssueWidth = 1;        // 1 micro-op is dispatched per cycle.
  let LoadLatency = 3;
  let MispredictPenalty = 3;
  let CompleteModel = false;
  let UnsupportedFeatures = [HasStdExtZbkb, HasStdExtZbkc, HasStdExtZbkx,
                             HasStdExtZcmt, HasStdExtZknd, HasStdExtZkne,
                             HasStdExtZknh, HasStdExtZksed, HasStdExtZksh,
                             HasStdExtZkr, HasVInstructions, HasVInstructionsI64];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 25-37: TableGen record RocketUnitALU / TableGen 记录 RocketUnitALU
```tablegen

//===----------------------------------------------------------------------===//
// Define each kind of processor resource and number available.

// Modeling each pipeline as a ProcResource using the BufferSize = 0 since
// Rocket is in-order.

let BufferSize = 0 in {
def RocketUnitALU        : ProcResource<1>; // Int ALU
def RocketUnitIMul       : ProcResource<1>; // Int Multiply
def RocketUnitMem        : ProcResource<1>; // Load/Store
def RocketUnitB          : ProcResource<1>; // Branch
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 38-47: TableGen record RocketUnitFPALU / TableGen 记录 RocketUnitFPALU
```tablegen
def RocketUnitFPALU      : ProcResource<1>; // FP ALU
}

let BufferSize = 1 in {
def RocketUnitIDiv       : ProcResource<1>; // Int Division
def RocketUnitFPDivSqrt  : ProcResource<1>; // FP Divide/Sqrt
}

//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 48-62: TableGen record definition / TableGen 记录定义
```tablegen
let SchedModel = RocketModel in {

// Branching
def : WriteRes<WriteJmp, [RocketUnitB]>;
def : WriteRes<WriteJal, [RocketUnitB]>;
def : WriteRes<WriteJalr, [RocketUnitB]>;

// Integer arithmetic and logic
def : WriteRes<WriteIALU32, [RocketUnitALU]>;
def : WriteRes<WriteIALU, [RocketUnitALU]>;
def : WriteRes<WriteShiftImm32, [RocketUnitALU]>;
def : WriteRes<WriteShiftImm, [RocketUnitALU]>;
def : WriteRes<WriteShiftReg32, [RocketUnitALU]>;
def : WriteRes<WriteShiftReg, [RocketUnitALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 63-74: TableGen record definition / TableGen 记录定义
```tablegen
// Integer multiplication
let Latency = 4 in {
def : WriteRes<WriteIMul, [RocketUnitIMul]>;
def : WriteRes<WriteIMul32, [RocketUnitIMul]>;
}

// Integer division
// Worst case latency is used.
def : WriteRes<WriteIDiv32, [RocketUnitIDiv]> {
  let Latency = 34;
  let ReleaseAtCycles = [34];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 75-84: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteIDiv, [RocketUnitIDiv]> {
  let Latency = 33;
  let ReleaseAtCycles = [33];
}

// Integer remainder
def : WriteRes<WriteIRem32, [RocketUnitIDiv]> {
  let Latency = 34;
  let ReleaseAtCycles = [34];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 85-97: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteIRem, [RocketUnitIDiv]> {
  let Latency = 33;
  let ReleaseAtCycles = [33];
}

// Memory
def : WriteRes<WriteSTB, [RocketUnitMem]>;
def : WriteRes<WriteSTH, [RocketUnitMem]>;
def : WriteRes<WriteSTW, [RocketUnitMem]>;
def : WriteRes<WriteSTD, [RocketUnitMem]>;
def : WriteRes<WriteFST32, [RocketUnitMem]>;
def : WriteRes<WriteFST64, [RocketUnitMem]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 98-108: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 3 in {
def : WriteRes<WriteLDB, [RocketUnitMem]>;
def : WriteRes<WriteLDH, [RocketUnitMem]>;
}

let Latency = 2 in {
def : WriteRes<WriteLDW, [RocketUnitMem]>;
def : WriteRes<WriteLDD, [RocketUnitMem]>;
def : WriteRes<WriteFLD32, [RocketUnitMem]>;
def : WriteRes<WriteFLD64, [RocketUnitMem]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 109-119: TableGen record definition / TableGen 记录定义
```tablegen
// Atomic memory
def : WriteRes<WriteAtomicW, [RocketUnitMem]>;
def : WriteRes<WriteAtomicD, [RocketUnitMem]>;

def : WriteRes<WriteAtomicLDW, [RocketUnitMem]>;
def : WriteRes<WriteAtomicLDD, [RocketUnitMem]>;
}

def : WriteRes<WriteAtomicSTW, [RocketUnitMem]>;
def : WriteRes<WriteAtomicSTD, [RocketUnitMem]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 120-132: TableGen record definition / TableGen 记录定义
```tablegen
// Single precision.
let Latency = 4 in {
def : WriteRes<WriteFAdd32, [RocketUnitFPALU]>;
def : WriteRes<WriteFSGNJ32, [RocketUnitFPALU]>;
def : WriteRes<WriteFMinMax32, [RocketUnitFPALU]>;
}

// Double precision
let Latency = 6 in {
def : WriteRes<WriteFAdd64, [RocketUnitFPALU]>;
def : WriteRes<WriteFSGNJ64, [RocketUnitFPALU]>;
def : WriteRes<WriteFMinMax64, [RocketUnitFPALU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 133-146: TableGen record definition / TableGen 记录定义
```tablegen

// Conversions
let Latency = 2 in {
def : WriteRes<WriteFCvtI32ToF32, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtI32ToF64, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtI64ToF32, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtI64ToF64, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtF32ToI32, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtF32ToI64, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtF64ToI32, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtF64ToI64, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtF32ToF64, [RocketUnitFPALU]>;
def : WriteRes<WriteFCvtF64ToF32, [RocketUnitFPALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 147-156: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFClass32, [RocketUnitFPALU]>;
def : WriteRes<WriteFClass64, [RocketUnitFPALU]>;
def : WriteRes<WriteFCmp32, [RocketUnitFPALU]>;
def : WriteRes<WriteFCmp64, [RocketUnitFPALU]>;
def : WriteRes<WriteFMovF32ToI32, [RocketUnitFPALU]>;
def : WriteRes<WriteFMovI32ToF32, [RocketUnitFPALU]>;
def : WriteRes<WriteFMovF64ToI64, [RocketUnitFPALU]>;
def : WriteRes<WriteFMovI64ToF64, [RocketUnitFPALU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 157-166: TableGen record definition / TableGen 记录定义
```tablegen
// FP multiplication
let Latency = 5 in {
def : WriteRes<WriteFMul32, [RocketUnitFPALU]>;
def : WriteRes<WriteFMA32, [RocketUnitFPALU]>;
}

let Latency = 7 in {
def : WriteRes<WriteFMul64, [RocketUnitFPALU]>;
def : WriteRes<WriteFMA64, [RocketUnitFPALU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 167-180: TableGen record definition / TableGen 记录定义
```tablegen

// FP division
// FP division unit on Rocket is not pipelined, so set resource cycles to latency.
let Latency = 20, ReleaseAtCycles = [20] in {
def : WriteRes<WriteFDiv32, [RocketUnitFPDivSqrt]>;
def : WriteRes<WriteFDiv64, [RocketUnitFPDivSqrt]>;
}

// FP square root unit on Rocket is not pipelined, so set resource cycles to latency.
def : WriteRes<WriteFSqrt32, [RocketUnitFPDivSqrt]> { let Latency = 20;
                                                      let ReleaseAtCycles = [20]; }
def : WriteRes<WriteFSqrt64, [RocketUnitFPDivSqrt]> { let Latency = 25;
                                                      let ReleaseAtCycles = [25]; }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 181-200: TableGen record definition / TableGen 记录定义
```tablegen
// Others
def : WriteRes<WriteCSR, []>;
def : WriteRes<WriteNop, []>;

def : InstRW<[WriteIALU], (instrs COPY)>;

//===----------------------------------------------------------------------===//
// Bypass and advance
def : ReadAdvance<ReadJmp, 0>;
def : ReadAdvance<ReadJalr, 0>;
def : ReadAdvance<ReadCSR, 0>;
def : ReadAdvance<ReadStoreData, 0>;
def : ReadAdvance<ReadMemBase, 0>;
def : ReadAdvance<ReadIALU, 0>;
def : ReadAdvance<ReadIALU32, 0>;
def : ReadAdvance<ReadShiftImm, 0>;
def : ReadAdvance<ReadShiftImm32, 0>;
def : ReadAdvance<ReadShiftReg, 0>;
def : ReadAdvance<ReadShiftReg32, 0>;
def : ReadAdvance<ReadIDiv, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 201-220: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadIDiv32, 0>;
def : ReadAdvance<ReadIRem, 0>;
def : ReadAdvance<ReadIRem32, 0>;
def : ReadAdvance<ReadIMul, 0>;
def : ReadAdvance<ReadIMul32, 0>;
def : ReadAdvance<ReadAtomicWA, 0>;
def : ReadAdvance<ReadAtomicWD, 0>;
def : ReadAdvance<ReadAtomicDA, 0>;
def : ReadAdvance<ReadAtomicDD, 0>;
def : ReadAdvance<ReadAtomicLDW, 0>;
def : ReadAdvance<ReadAtomicLDD, 0>;
def : ReadAdvance<ReadAtomicSTW, 0>;
def : ReadAdvance<ReadAtomicSTD, 0>;
def : ReadAdvance<ReadFStoreData, 0>;
def : ReadAdvance<ReadFMemBase, 0>;
def : ReadAdvance<ReadFAdd32, 0>;
def : ReadAdvance<ReadFAdd64, 0>;
def : ReadAdvance<ReadFMul32, 0>;
def : ReadAdvance<ReadFMul64, 0>;
def : ReadAdvance<ReadFMA32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 221-240: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMA32Addend, 0>;
def : ReadAdvance<ReadFMA64, 0>;
def : ReadAdvance<ReadFMA64Addend, 0>;
def : ReadAdvance<ReadFDiv32, 0>;
def : ReadAdvance<ReadFDiv64, 0>;
def : ReadAdvance<ReadFSqrt32, 0>;
def : ReadAdvance<ReadFSqrt64, 0>;
def : ReadAdvance<ReadFCmp32, 0>;
def : ReadAdvance<ReadFCmp64, 0>;
def : ReadAdvance<ReadFSGNJ32, 0>;
def : ReadAdvance<ReadFSGNJ64, 0>;
def : ReadAdvance<ReadFMinMax32, 0>;
def : ReadAdvance<ReadFMinMax64, 0>;
def : ReadAdvance<ReadFCvtF32ToI32, 0>;
def : ReadAdvance<ReadFCvtF32ToI64, 0>;
def : ReadAdvance<ReadFCvtF64ToI32, 0>;
def : ReadAdvance<ReadFCvtF64ToI64, 0>;
def : ReadAdvance<ReadFCvtI32ToF32, 0>;
def : ReadAdvance<ReadFCvtI32ToF64, 0>;
def : ReadAdvance<ReadFCvtI64ToF32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 241-250: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFCvtI64ToF64, 0>;
def : ReadAdvance<ReadFCvtF32ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF32, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFClass64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 251-267: Bulk record instantiation / 批量记录实例化
```tablegen
//===----------------------------------------------------------------------===//
// Unsupported extensions
defm : UnsupportedSchedQ;
defm : UnsupportedSchedV;
defm : UnsupportedSchedZabha;
defm : UnsupportedSchedZba;
defm : UnsupportedSchedZbb;
defm : UnsupportedSchedZbc;
defm : UnsupportedSchedZbs;
defm : UnsupportedSchedZbkb;
defm : UnsupportedSchedZbkx;
defm : UnsupportedSchedZfa;
defm : UnsupportedSchedZfhmin;
defm : UnsupportedSchedSFB;
defm : UnsupportedSchedZvk;
defm : UnsupportedSchedXsf;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
