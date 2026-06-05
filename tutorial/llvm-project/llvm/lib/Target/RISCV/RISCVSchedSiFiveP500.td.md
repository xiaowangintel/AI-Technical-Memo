# RISCVSchedSiFiveP500.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedSiFiveP500.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Commentary and design intent / 注释与设计意图
```tablegen
//==- RISCVSchedSiFiveP500.td - SiFiveP500 Scheduling Defs ---*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 11-25: TableGen record SiFiveP500Model / TableGen 记录 SiFiveP500Model
```tablegen
def SiFiveP500Model : SchedMachineModel {
  let IssueWidth = 3;         // 3 micro-ops are dispatched per cycle.
  let MicroOpBufferSize = 96; // Max micro-ops that can be buffered.
  let LoadLatency = 4;        // Cycles for loads to access the cache.
  let MispredictPenalty = 9;  // Extra cycles for a mispredicted branch.
  let CompleteModel = false;
}

// The SiFiveP500 microarchitecure has 7 pipelines:
// Three pipelines for integer operations.
// Two pipelines for FPU operations.
// One pipeline for Load operations.
// One pipeline for Store operations.
let SchedModel = SiFiveP500Model in {
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 26-40: TableGen record SiFiveP500IEXQ0 / TableGen 记录 SiFiveP500IEXQ0
```tablegen
def SiFiveP500IEXQ0       : ProcResource<1>;
def SiFiveP500IEXQ1       : ProcResource<1>;
def SiFiveP500IEXQ2       : ProcResource<1>;
def SiFiveP500FEXQ0       : ProcResource<1>;
def SiFiveP500FEXQ1       : ProcResource<1>;
def SiFiveP500Load        : ProcResource<1>;
def SiFiveP500Store       : ProcResource<1>;

def SiFiveP500IntArith    : ProcResGroup<[SiFiveP500IEXQ0, SiFiveP500IEXQ1, SiFiveP500IEXQ2]>;
defvar SiFiveP500Branch   = SiFiveP500IEXQ0;
defvar SiFiveP500SYS      = SiFiveP500IEXQ1;
defvar SiFiveP500CMOV     = SiFiveP500IEXQ1;
defvar SiFiveP500MulI2F   = SiFiveP500IEXQ2;
def SiFiveP500Div         : ProcResource<1>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 41-57: TableGen record SiFiveP500FloatArith / TableGen 记录 SiFiveP500FloatArith
```tablegen
def SiFiveP500FloatArith  : ProcResGroup<[SiFiveP500FEXQ0, SiFiveP500FEXQ1]>;
defvar SiFiveP500F2I      = SiFiveP500FEXQ0;
def SiFiveP500FloatDiv    : ProcResource<1>;

let Latency = 1 in {
// Integer arithmetic and logic
def : WriteRes<WriteIALU, [SiFiveP500IntArith]>;
def : WriteRes<WriteIALU32, [SiFiveP500IntArith]>;
def : WriteRes<WriteShiftImm, [SiFiveP500IntArith]>;
def : WriteRes<WriteShiftImm32, [SiFiveP500IntArith]>;
def : WriteRes<WriteShiftReg, [SiFiveP500IntArith]>;
def : WriteRes<WriteShiftReg32, [SiFiveP500IntArith]>;
// Branching
def : WriteRes<WriteJmp, [SiFiveP500Branch]>;
def : WriteRes<WriteJal, [SiFiveP500Branch]>;
def : WriteRes<WriteJalr, [SiFiveP500Branch]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 58-73: TableGen record P500WriteCMOV / TableGen 记录 P500WriteCMOV
```tablegen

// CMOV
def P500WriteCMOV : SchedWriteRes<[SiFiveP500Branch, SiFiveP500CMOV]> {
  let Latency = 2;
  let NumMicroOps = 2;
}
def : InstRW<[P500WriteCMOV], (instrs PseudoCCMOVGPRNoX0)>;

let Latency = 3 in {
// Integer multiplication
def : WriteRes<WriteIMul, [SiFiveP500MulI2F]>;
def : WriteRes<WriteIMul32, [SiFiveP500MulI2F]>;
// cpop[w] look exactly like multiply.
def : WriteRes<WriteCPOP, [SiFiveP500MulI2F]>;
def : WriteRes<WriteCPOP32, [SiFiveP500MulI2F]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 74-83: TableGen record definition / TableGen 记录定义
```tablegen

// Integer division
def : WriteRes<WriteIDiv, [SiFiveP500MulI2F, SiFiveP500Div]> {
  let Latency = 35;
  let ReleaseAtCycles = [1, 34];
}
def : WriteRes<WriteIDiv32, [SiFiveP500MulI2F, SiFiveP500Div]> {
  let Latency = 20;
  let ReleaseAtCycles = [1, 19];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 84-93: TableGen record definition / TableGen 记录定义
```tablegen

// Integer remainder
def : WriteRes<WriteIRem, [SiFiveP500MulI2F, SiFiveP500Div]> {
  let Latency = 35;
  let ReleaseAtCycles = [1, 34];
}
def : WriteRes<WriteIRem32, [SiFiveP500MulI2F, SiFiveP500Div]> {
  let Latency = 20;
  let ReleaseAtCycles = [1, 19];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 94-106: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 1 in {
// Bitmanip
def : WriteRes<WriteRotateImm, [SiFiveP500IntArith]>;
def : WriteRes<WriteRotateImm32, [SiFiveP500IntArith]>;
def : WriteRes<WriteRotateReg, [SiFiveP500IntArith]>;
def : WriteRes<WriteRotateReg32, [SiFiveP500IntArith]>;

def : WriteRes<WriteCLZ, [SiFiveP500IntArith]>;
def : WriteRes<WriteCLZ32, [SiFiveP500IntArith]>;
def : WriteRes<WriteCTZ, [SiFiveP500IntArith]>;
def : WriteRes<WriteCTZ32, [SiFiveP500IntArith]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 107-125: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteORCB, [SiFiveP500IntArith]>;
def : WriteRes<WriteIMinMax, [SiFiveP500IntArith]>;

def : WriteRes<WriteREV8, [SiFiveP500IntArith]>;

def : WriteRes<WriteSHXADD, [SiFiveP500IntArith]>;
def : WriteRes<WriteSHXADD32, [SiFiveP500IntArith]>;
}

// Memory
let Latency = 1 in {
def : WriteRes<WriteSTB, [SiFiveP500Store]>;
def : WriteRes<WriteSTH, [SiFiveP500Store]>;
def : WriteRes<WriteSTW, [SiFiveP500Store]>;
def : WriteRes<WriteSTD, [SiFiveP500Store]>;
def : WriteRes<WriteFST16, [SiFiveP500Store]>;
def : WriteRes<WriteFST32, [SiFiveP500Store]>;
def : WriteRes<WriteFST64, [SiFiveP500Store]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 126-139: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 4 in {
def : WriteRes<WriteLDB, [SiFiveP500Load]>;
def : WriteRes<WriteLDH, [SiFiveP500Load]>;
}
let Latency = 4 in {
def : WriteRes<WriteLDW, [SiFiveP500Load]>;
def : WriteRes<WriteLDD, [SiFiveP500Load]>;
}

let Latency = 5 in {
def : WriteRes<WriteFLD16, [SiFiveP500Load]>;
def : WriteRes<WriteFLD32, [SiFiveP500Load]>;
def : WriteRes<WriteFLD64, [SiFiveP500Load]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 140-149: TableGen record definition / TableGen 记录定义
```tablegen

// Atomic memory
let Latency = 3 in {
def : WriteRes<WriteAtomicSTW, [SiFiveP500Store]>;
def : WriteRes<WriteAtomicSTD, [SiFiveP500Store]>;
def : WriteRes<WriteAtomicW, [SiFiveP500Load]>;
def : WriteRes<WriteAtomicD, [SiFiveP500Load]>;
def : WriteRes<WriteAtomicLDW, [SiFiveP500Load]>;
def : WriteRes<WriteAtomicLDD, [SiFiveP500Load]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 150-160: TableGen record definition / TableGen 记录定义
```tablegen

// Floating point
let Latency = 4 in {
def : WriteRes<WriteFAdd16, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFAdd32, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFAdd64, [SiFiveP500FloatArith]>;

def : WriteRes<WriteFMul16, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFMul32, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFMul64, [SiFiveP500FloatArith]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 161-170: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFMA16, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFMA32, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFMA64, [SiFiveP500FloatArith]>;
}

let Latency = 2 in {
def : WriteRes<WriteFSGNJ16, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFSGNJ32, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFSGNJ64, [SiFiveP500FloatArith]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 171-180: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFMinMax16, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFMinMax32, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFMinMax64, [SiFiveP500FloatArith]>;
}

// Half precision.
def : WriteRes<WriteFDiv16, [SiFiveP500FEXQ1, SiFiveP500FloatDiv]> {
  let Latency = 19;
  let ReleaseAtCycles = [1, 18];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 181-190: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFSqrt16, [SiFiveP500FEXQ1, SiFiveP500FloatDiv]> {
  let Latency = 18;
  let ReleaseAtCycles = [1, 17];
}

// Single precision.
def : WriteRes<WriteFDiv32, [SiFiveP500FEXQ1, SiFiveP500FloatDiv]> {
  let Latency = 19;
  let ReleaseAtCycles = [1, 18];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 191-200: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFSqrt32, [SiFiveP500FEXQ1, SiFiveP500FloatDiv]> {
  let Latency = 18;
  let ReleaseAtCycles = [1, 17];
}

// Double precision
def : WriteRes<WriteFDiv64, [SiFiveP500FEXQ1, SiFiveP500FloatDiv]> {
  let Latency = 33;
  let ReleaseAtCycles = [1, 32];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 201-220: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFSqrt64, [SiFiveP500FEXQ1, SiFiveP500FloatDiv]> {
  let Latency = 33;
  let ReleaseAtCycles = [1, 32];
}

// Conversions
let Latency = 2 in {
def : WriteRes<WriteFCvtI32ToF16, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFCvtI32ToF32, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFCvtI32ToF64, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFCvtI64ToF16, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFCvtI64ToF32, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFCvtI64ToF64, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFCvtF16ToI32, [SiFiveP500F2I]>;
def : WriteRes<WriteFCvtF16ToI64, [SiFiveP500F2I]>;
def : WriteRes<WriteFCvtF16ToF32, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFCvtF16ToF64, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFCvtF32ToI32, [SiFiveP500F2I]>;
def : WriteRes<WriteFCvtF32ToI64, [SiFiveP500F2I]>;
def : WriteRes<WriteFCvtF32ToF16, [SiFiveP500FloatArith]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 221-239: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFCvtF32ToF64, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFCvtF64ToI32, [SiFiveP500F2I]>;
def : WriteRes<WriteFCvtF64ToI64, [SiFiveP500F2I]>;
def : WriteRes<WriteFCvtF64ToF16, [SiFiveP500FloatArith]>;
def : WriteRes<WriteFCvtF64ToF32, [SiFiveP500FloatArith]>;

def : WriteRes<WriteFClass16, [SiFiveP500F2I]>;
def : WriteRes<WriteFClass32, [SiFiveP500F2I]>;
def : WriteRes<WriteFClass64, [SiFiveP500F2I]>;
def : WriteRes<WriteFCmp16, [SiFiveP500F2I]>;
def : WriteRes<WriteFCmp32, [SiFiveP500F2I]>;
def : WriteRes<WriteFCmp64, [SiFiveP500F2I]>;
def : WriteRes<WriteFMovI16ToF16, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFMovF16ToI16, [SiFiveP500F2I]>;
def : WriteRes<WriteFMovI32ToF32, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFMovF32ToI32, [SiFiveP500F2I]>;
def : WriteRes<WriteFMovI64ToF64, [SiFiveP500MulI2F]>;
def : WriteRes<WriteFMovF64ToI64, [SiFiveP500F2I]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 240-259: TableGen record definition / TableGen 记录定义
```tablegen

// Others
def : WriteRes<WriteCSR, [SiFiveP500SYS]>;
def : WriteRes<WriteNop, []>;

// FIXME: This could be better modeled by looking at the regclasses of the operands.
def : InstRW<[WriteIALU, ReadIALU], (instrs COPY)>;

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
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 260-279: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadShiftReg32, 0>;
def : ReadAdvance<ReadIDiv, 0>;
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
def : ReadAdvance<ReadFAdd16, 0>;
def : ReadAdvance<ReadFAdd32, 0>;
def : ReadAdvance<ReadFAdd64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 280-299: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMul16, 0>;
def : ReadAdvance<ReadFMA16, 0>;
def : ReadAdvance<ReadFMA16Addend, 0>;
def : ReadAdvance<ReadFMul32, 0>;
def : ReadAdvance<ReadFMA32, 0>;
def : ReadAdvance<ReadFMA32Addend, 0>;
def : ReadAdvance<ReadFMul64, 0>;
def : ReadAdvance<ReadFMA64, 0>;
def : ReadAdvance<ReadFMA64Addend, 0>;
def : ReadAdvance<ReadFDiv16, 0>;
def : ReadAdvance<ReadFDiv32, 0>;
def : ReadAdvance<ReadFDiv64, 0>;
def : ReadAdvance<ReadFSqrt16, 0>;
def : ReadAdvance<ReadFSqrt32, 0>;
def : ReadAdvance<ReadFSqrt64, 0>;
def : ReadAdvance<ReadFCmp16, 0>;
def : ReadAdvance<ReadFCmp32, 0>;
def : ReadAdvance<ReadFCmp64, 0>;
def : ReadAdvance<ReadFSGNJ16, 0>;
def : ReadAdvance<ReadFSGNJ32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 300-319: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFSGNJ64, 0>;
def : ReadAdvance<ReadFMinMax16, 0>;
def : ReadAdvance<ReadFMinMax32, 0>;
def : ReadAdvance<ReadFMinMax64, 0>;
def : ReadAdvance<ReadFCvtF16ToI32, 0>;
def : ReadAdvance<ReadFCvtF16ToI64, 0>;
def : ReadAdvance<ReadFCvtF32ToI32, 0>;
def : ReadAdvance<ReadFCvtF32ToI64, 0>;
def : ReadAdvance<ReadFCvtF64ToI32, 0>;
def : ReadAdvance<ReadFCvtF64ToI64, 0>;
def : ReadAdvance<ReadFCvtI32ToF16, 0>;
def : ReadAdvance<ReadFCvtI32ToF32, 0>;
def : ReadAdvance<ReadFCvtI32ToF64, 0>;
def : ReadAdvance<ReadFCvtI64ToF16, 0>;
def : ReadAdvance<ReadFCvtI64ToF32, 0>;
def : ReadAdvance<ReadFCvtI64ToF64, 0>;
def : ReadAdvance<ReadFCvtF32ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF32, 0>;
def : ReadAdvance<ReadFCvtF16ToF32, 0>;
def : ReadAdvance<ReadFCvtF32ToF16, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 320-331: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFCvtF16ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF16, 0>;
def : ReadAdvance<ReadFMovF16ToI16, 0>;
def : ReadAdvance<ReadFMovI16ToF16, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFClass16, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFClass64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 332-348: TableGen record definition / TableGen 记录定义
```tablegen
// Bitmanip
def : ReadAdvance<ReadRotateImm, 0>;
def : ReadAdvance<ReadRotateImm32, 0>;
def : ReadAdvance<ReadRotateReg, 0>;
def : ReadAdvance<ReadRotateReg32, 0>;
def : ReadAdvance<ReadCLZ, 0>;
def : ReadAdvance<ReadCLZ32, 0>;
def : ReadAdvance<ReadCTZ, 0>;
def : ReadAdvance<ReadCTZ32, 0>;
def : ReadAdvance<ReadCPOP, 0>;
def : ReadAdvance<ReadCPOP32, 0>;
def : ReadAdvance<ReadORCB, 0>;
def : ReadAdvance<ReadIMinMax, 0>;
def : ReadAdvance<ReadREV8, 0>;
def : ReadAdvance<ReadSHXADD, 0>;
def : ReadAdvance<ReadSHXADD32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 349-362: Bulk record instantiation / 批量记录实例化
```tablegen
//===----------------------------------------------------------------------===//
// Unsupported extensions
defm : UnsupportedSchedQ;
defm : UnsupportedSchedV;
defm : UnsupportedSchedZabha;
defm : UnsupportedSchedZbc;
defm : UnsupportedSchedZbs;
defm : UnsupportedSchedZbkb;
defm : UnsupportedSchedZbkx;
defm : UnsupportedSchedSFB;
defm : UnsupportedSchedZfa;
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
