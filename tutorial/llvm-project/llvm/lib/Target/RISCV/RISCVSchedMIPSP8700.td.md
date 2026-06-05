# RISCVSchedMIPSP8700.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedMIPSP8700.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVSchedMIPSP8700.td - MIPS RISC-V Processor -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// P8700 - a RISC-V processor by MIPS.
// Pipelines:
//   - 2 Integer Arithmetic and Logical Units (ALU and AL2)
//   - Multiply / Divide Unit (MDU)
//   - Branch Unit (CTI)
//   - Load Store Unit (LSU)
//   - Short Floating Point Pipe (FPUS)
//   - Long Floating Point Pipe (FPUL)
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 19-32: TableGen record MIPSP8700Model / TableGen 记录 MIPSP8700Model
```tablegen

def MIPSP8700Model : SchedMachineModel {
  int IssueWidth = 4;
  int MicroOpBufferSize = 96;
  int LoadLatency = 4;
  int MispredictPenalty = 8;
  let CompleteModel = 0;
}

let SchedModel = MIPSP8700Model in {
// Handle ALQ Pipelines.
// It contains 1 ALU Unit only.
def p8700ALQ : ProcResource<1> { let BufferSize = 16; }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 33-43: TableGen record p8700AGQ / TableGen 记录 p8700AGQ
```tablegen
// Handle AGQ Pipelines.
def p8700AGQ : ProcResource<3> { let BufferSize = 16; }
def p8700IssueAL2 : ProcResource<1> { let Super = p8700AGQ; }
def p8700IssueCTI : ProcResource<1> { let Super = p8700AGQ; }
def p8700IssueLSU : ProcResource<1> { let Super = p8700AGQ; }
def p8700WriteEitherALU : ProcResGroup<[p8700ALQ, p8700IssueAL2]>;

// Handle Multiply Divide Pipe.
def p8700GpDiv : ProcResource<1>;
def p8700GpMul : ProcResource<1>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 44-54: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteIALU, [p8700WriteEitherALU]>;
def : WriteRes<WriteIALU32, [p8700WriteEitherALU]>;
def : WriteRes<WriteShiftImm, [p8700WriteEitherALU]>;
def : WriteRes<WriteShiftImm32, [p8700WriteEitherALU]>;
def : WriteRes<WriteShiftReg, [p8700WriteEitherALU]>;
def : WriteRes<WriteShiftReg32, [p8700WriteEitherALU]>;

// Handle zba.
def : WriteRes<WriteSHXADD, [p8700WriteEitherALU]>;
def : WriteRes<WriteSHXADD32, [p8700WriteEitherALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 55-71: TableGen record definition / TableGen 记录定义
```tablegen
// Handle zbb.
let Latency = 2 in {
def : WriteRes<WriteCLZ, [p8700IssueAL2]>;
def : WriteRes<WriteCTZ, [p8700IssueAL2]>;
def : WriteRes<WriteCPOP, [p8700IssueAL2]>;
def : WriteRes<WriteCLZ32, [p8700IssueAL2]>;
def : WriteRes<WriteCTZ32, [p8700IssueAL2]>;
def : WriteRes<WriteCPOP32, [p8700IssueAL2]>;
}
def : WriteRes<WriteRotateReg, [p8700WriteEitherALU]>;
def : WriteRes<WriteRotateImm, [p8700WriteEitherALU]>;
def : WriteRes<WriteRotateReg32, [p8700WriteEitherALU]>;
def : WriteRes<WriteRotateImm32, [p8700WriteEitherALU]>;
def : WriteRes<WriteREV8, [p8700WriteEitherALU]>;
def : WriteRes<WriteORCB, [p8700WriteEitherALU]>;
def : WriteRes<WriteIMinMax, [p8700WriteEitherALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 72-85: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 0 in
def : WriteRes<WriteNop, [p8700WriteEitherALU]>;

let Latency = 4 in {
def : WriteRes<WriteLDB, [p8700IssueLSU]>;
def : WriteRes<WriteLDH, [p8700IssueLSU]>;
def : WriteRes<WriteLDW, [p8700IssueLSU]>;
def : WriteRes<WriteLDD, [p8700IssueLSU]>;

def : WriteRes<WriteAtomicW, [p8700IssueLSU]>;
def : WriteRes<WriteAtomicD, [p8700IssueLSU]>;
def : WriteRes<WriteAtomicLDW, [p8700IssueLSU]>;
def : WriteRes<WriteAtomicLDD, [p8700IssueLSU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 86-97: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 8 in {
def : WriteRes<WriteFLD32, [p8700IssueLSU]>;
def : WriteRes<WriteFLD64, [p8700IssueLSU]>;
}

let Latency = 3 in {
def : WriteRes<WriteSTB, [p8700IssueLSU]>;
def : WriteRes<WriteSTH, [p8700IssueLSU]>;
def : WriteRes<WriteSTW, [p8700IssueLSU]>;
def : WriteRes<WriteSTD, [p8700IssueLSU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 98-110: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteAtomicSTW, [p8700IssueLSU]>;
def : WriteRes<WriteAtomicSTD, [p8700IssueLSU]>;
}

def : WriteRes<WriteFST32, [p8700IssueLSU]>;
def : WriteRes<WriteFST64, [p8700IssueLSU]>;

let Latency = 7 in {
def : WriteRes<WriteFMovI32ToF32, [p8700IssueLSU]>;
def : WriteRes<WriteFMovF32ToI32, [p8700IssueLSU]>;
def : WriteRes<WriteFMovI64ToF64, [p8700IssueLSU]>;
def : WriteRes<WriteFMovF64ToI64, [p8700IssueLSU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 111-122: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 4 in {
def : WriteRes<WriteIMul, [p8700GpMul]>;
def : WriteRes<WriteIMul32, [p8700GpMul]>;
}

let Latency = 7, ReleaseAtCycles = [7] in {
def : WriteRes<WriteIDiv, [p8700GpDiv]>;
def : WriteRes<WriteIDiv32,  [p8700GpDiv]>;
def : WriteRes<WriteIRem, [p8700GpDiv]>;
def : WriteRes<WriteIRem32, [p8700GpDiv]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 123-137: TableGen record p8700FPQ / TableGen 记录 p8700FPQ
```tablegen

def : WriteRes<WriteCSR, [p8700ALQ]>;

// Handle CTI Pipeline.
def : WriteRes<WriteJmp, [p8700IssueCTI]>;
def : WriteRes<WriteJal, [p8700IssueCTI]>;
def : WriteRes<WriteJalr, [p8700IssueCTI]>;

// Handle FPU Pipelines.
def p8700FPQ : ProcResource<3> { let BufferSize = 16; }
def p8700IssueFPUS : ProcResource<1> { let Super = p8700FPQ; }
def p8700IssueFPUL : ProcResource<1> { let Super = p8700FPQ; }
def p8700FpuApu    : ProcResource<1>;
def p8700FpuLong   : ProcResource<1>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 138-149: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 4 in {
def : WriteRes<WriteFCvtI32ToF32, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtI32ToF64, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtI64ToF32, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtI64ToF64, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtF32ToI32, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtF32ToI64, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtF32ToF64, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtF64ToI32, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtF64ToI64, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFCvtF64ToF32, [p8700IssueFPUL, p8700FpuApu]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 150-159: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFAdd32, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFAdd64, [p8700IssueFPUL, p8700FpuApu]>;
}

let Latency = 2 in {
def : WriteRes<WriteFSGNJ32, [p8700IssueFPUS, p8700FpuApu]>;
def : WriteRes<WriteFMinMax32, [p8700IssueFPUS, p8700FpuApu]>;
def : WriteRes<WriteFSGNJ64, [p8700IssueFPUS, p8700FpuApu]>;
def : WriteRes<WriteFMinMax64, [p8700IssueFPUS, p8700FpuApu]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 160-170: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFCmp32, [p8700IssueFPUS, p8700FpuApu]>;
def : WriteRes<WriteFCmp64, [p8700IssueFPUS, p8700FpuApu]>;
}

def : WriteRes<WriteFClass32, [p8700IssueFPUS, p8700FpuApu]>;
def : WriteRes<WriteFClass64, [p8700IssueFPUS, p8700FpuApu]>;

let Latency = 8 in {
def : WriteRes<WriteFMA32, [p8700FpuLong, p8700FpuApu]>;
def : WriteRes<WriteFMA64, [p8700FpuLong, p8700FpuApu]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 171-180: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 5 in {
def : WriteRes<WriteFMul32, [p8700FpuLong, p8700FpuApu]>;
def : WriteRes<WriteFMul64, [p8700FpuLong, p8700FpuApu]>;
}

let Latency = 11, ReleaseAtCycles = [1, 11] in {
def : WriteRes<WriteFDiv32, [p8700FpuLong, p8700FpuApu]>;
def : WriteRes<WriteFSqrt32, [p8700FpuLong, p8700FpuApu]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 181-200: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 17, ReleaseAtCycles = [1, 17] in {
def : WriteRes<WriteFDiv64, [p8700IssueFPUL, p8700FpuApu]>;
def : WriteRes<WriteFSqrt64, [p8700IssueFPUL, p8700FpuApu]>;
}

// Bypass and advance.
def : ReadAdvance<ReadIALU, 0>;
def : ReadAdvance<ReadIALU32, 0>;
def : ReadAdvance<ReadShiftImm, 0>;
def : ReadAdvance<ReadShiftImm32, 0>;
def : ReadAdvance<ReadShiftReg, 0>;
def : ReadAdvance<ReadShiftReg32, 0>;
def : ReadAdvance<ReadSHXADD, 0>;
def : ReadAdvance<ReadSHXADD32, 0>;
def : ReadAdvance<ReadRotateReg, 0>;
def : ReadAdvance<ReadRotateImm, 0>;
def : ReadAdvance<ReadCLZ, 0>;
def : ReadAdvance<ReadCTZ, 0>;
def : ReadAdvance<ReadCPOP, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 201-220: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadRotateReg32, 0>;
def : ReadAdvance<ReadRotateImm32, 0>;
def : ReadAdvance<ReadCLZ32, 0>;
def : ReadAdvance<ReadCTZ32, 0>;
def : ReadAdvance<ReadCPOP32, 0>;
def : ReadAdvance<ReadREV8, 0>;
def : ReadAdvance<ReadORCB, 0>;
def : ReadAdvance<ReadIMul, 0>;
def : ReadAdvance<ReadIMul32, 0>;
def : ReadAdvance<ReadIDiv, 0>;
def : ReadAdvance<ReadIDiv32, 0>;
def : ReadAdvance<ReadJmp, 0>;
def : ReadAdvance<ReadJalr, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
def : ReadAdvance<ReadFSGNJ32, 0>;
def : ReadAdvance<ReadFMinMax32, 0>;
def : ReadAdvance<ReadFSGNJ64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 221-240: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMinMax64, 0>;
def : ReadAdvance<ReadFCmp32, 0>;
def : ReadAdvance<ReadFCmp64, 0>;
def : ReadAdvance<ReadFCvtI32ToF32, 0>;
def : ReadAdvance<ReadFCvtI32ToF64, 0>;
def : ReadAdvance<ReadFCvtI64ToF32, 0>;
def : ReadAdvance<ReadFCvtI64ToF64, 0>;
def : ReadAdvance<ReadFCvtF32ToI32, 0>;
def : ReadAdvance<ReadFCvtF32ToI64, 0>;
def : ReadAdvance<ReadFCvtF32ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToI32, 0>;
def : ReadAdvance<ReadFCvtF64ToI64, 0>;
def : ReadAdvance<ReadFCvtF64ToF32, 0>;
def : ReadAdvance<ReadFAdd32, 0>;
def : ReadAdvance<ReadFAdd64, 0>;
def : ReadAdvance<ReadFMul32, 0>;
def : ReadAdvance<ReadFMul64, 0>;
def : ReadAdvance<ReadFMA32, 0>;
def : ReadAdvance<ReadFMA32Addend, 0>;
def : ReadAdvance<ReadFMA64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 241-260: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMA64Addend, 0>;
def : ReadAdvance<ReadFDiv32, 0>;
def : ReadAdvance<ReadFSqrt32, 0>;
def : ReadAdvance<ReadFDiv64, 0>;
def : ReadAdvance<ReadFSqrt64, 0>;
def : ReadAdvance<ReadAtomicWA, 0>;
def : ReadAdvance<ReadAtomicWD, 0>;
def : ReadAdvance<ReadAtomicDA, 0>;
def : ReadAdvance<ReadAtomicDD, 0>;
def : ReadAdvance<ReadAtomicLDW, 0>;
def : ReadAdvance<ReadAtomicLDD, 0>;
def : ReadAdvance<ReadAtomicSTW, 0>;
def : ReadAdvance<ReadAtomicSTD, 0>;
def : ReadAdvance<ReadFStoreData, 0>;
def : ReadAdvance<ReadCSR, 0>;
def : ReadAdvance<ReadMemBase, 0>;
def : ReadAdvance<ReadStoreData, 0>;
def : ReadAdvance<ReadFMemBase, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFClass64, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 261-278: Bulk record instantiation / 批量记录实例化
```tablegen
def : ReadAdvance<ReadIMinMax, 0>;
def : ReadAdvance<ReadIRem, 0>;
def : ReadAdvance<ReadIRem32, 0>;

// Unsupported extensions.
defm : UnsupportedSchedQ;
defm : UnsupportedSchedV;
defm : UnsupportedSchedZbc;
defm : UnsupportedSchedZbs;
defm : UnsupportedSchedZbkb;
defm : UnsupportedSchedZbkx;
defm : UnsupportedSchedZfa;
defm : UnsupportedSchedZfhmin;
defm : UnsupportedSchedSFB;
defm : UnsupportedSchedZabha;
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
