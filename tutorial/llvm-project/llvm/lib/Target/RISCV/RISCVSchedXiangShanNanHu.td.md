# RISCVSchedXiangShanNanHu.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedXiangShanNanHu.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Commentary and design intent / 注释与设计意图
```tablegen
//==- RISCVSchedXiangShanNanHu.td - XS-NanHu Scheduling Defs -*- tablegen -*-=//
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

### Lines 11-28: Commentary and design intent / 注释与设计意图
```tablegen
// XiangShan is a high-performance open-source RISC-V processor developed by
// the Institute of Computing Technology (ICT), Chinese Academy of Sciences.
// Source: https://github.com/OpenXiangShan/XiangShan
// Documentation: https://github.com/OpenXiangShan/XiangShan-doc

// XiangShan-NanHu is the second generation of XiangShan processor series.
// Overview: https://xiangshan-doc.readthedocs.io/zh-cn/latest/integration/overview/

def XiangShanNanHuModel : SchedMachineModel {
  let MicroOpBufferSize = 256;
  let LoopMicroOpBufferSize = 48;  // Instruction queue size
  let IssueWidth = 6;  // 6-way decode and dispatch
  let LoadLatency = 4;
  let MispredictPenalty = 11; // Based on estimate of pipeline depth.
  let CompleteModel = 0;
  let UnsupportedFeatures = [HasStdExtZcmt, HasStdExtZkr, HasVInstructions,
                             HasVInstructionsI64];
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 29-40: TableGen record XS2ALU / TableGen 记录 XS2ALU
```tablegen

let SchedModel = XiangShanNanHuModel in {

// The reservation stations are distributed and grouped as 32-entry or 16-entry smaller ones.
let BufferSize = 16 in {
  def XS2ALU : ProcResource<4>;
  def XS2MDU : ProcResource<2>;
  def XS2MISC : ProcResource<1>;

  def XS2FMAC : ProcResource<4>;
  def XS2FMISC : ProcResource<2>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 41-50: TableGen record XS2LD / TableGen 记录 XS2LD
```tablegen
  // Load/Store queues are ignored.
  def XS2LD : ProcResource<2>;
  def XS2ST : ProcResource<2>;
}

// Branching
def : WriteRes<WriteJmp, [XS2MISC]>;
def : WriteRes<WriteJal, [XS2MISC]>;
def : WriteRes<WriteJalr, [XS2MISC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 51-60: TableGen record definition / TableGen 记录定义
```tablegen
// Integer arithmetic and logic
let Latency = 1 in {
def : WriteRes<WriteIALU, [XS2ALU]>;
def : WriteRes<WriteIALU32, [XS2ALU]>;
def : WriteRes<WriteShiftImm, [XS2ALU]>;
def : WriteRes<WriteShiftImm32, [XS2ALU]>;
def : WriteRes<WriteShiftReg, [XS2ALU]>;
def : WriteRes<WriteShiftReg32, [XS2ALU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 61-74: TableGen record definition / TableGen 记录定义
```tablegen
// Integer multiplication
let Latency = 3 in {
def : WriteRes<WriteIMul, [XS2MDU]>;
def : WriteRes<WriteIMul32, [XS2MDU]>;
}

// Integer division/remainder
// SRT16 algorithm
let Latency = 20, ReleaseAtCycles = [20] in {
def : WriteRes<WriteIDiv32, [XS2MDU]>;
def : WriteRes<WriteIDiv, [XS2MDU]>;
def : WriteRes<WriteIRem32, [XS2MDU]>;
def : WriteRes<WriteIRem, [XS2MDU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 75-90: TableGen record definition / TableGen 记录定义
```tablegen

// Zb*
let Latency = 1 in {
// Zba
def : WriteRes<WriteSHXADD, [XS2ALU]>;
def : WriteRes<WriteSHXADD32, [XS2ALU]>;

// Zbb
def : WriteRes<WriteRotateImm, [XS2ALU]>;
def : WriteRes<WriteRotateImm32, [XS2ALU]>;
def : WriteRes<WriteRotateReg, [XS2ALU]>;
def : WriteRes<WriteRotateReg32, [XS2ALU]>;
def : WriteRes<WriteORCB, [XS2ALU]>;
def : WriteRes<WriteIMinMax, [XS2ALU]>;
def : WriteRes<WriteREV8, [XS2ALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 91-102: TableGen record definition / TableGen 记录定义
```tablegen
// Zbkb
def : WriteRes<WriteBREV8, [XS2ALU]>;
def : WriteRes<WritePACK, [XS2ALU]>;
def : WriteRes<WritePACK32, [XS2ALU]>;
def : WriteRes<WriteZIP, [XS2ALU]>;

// Zbs
def : WriteRes<WriteSingleBit, [XS2ALU]>;
def : WriteRes<WriteSingleBitImm, [XS2ALU]>;
def : WriteRes<WriteBEXT, [XS2ALU]>;
def : WriteRes<WriteBEXTI, [XS2ALU]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 103-112: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 3 in {
// Zbb
def : WriteRes<WriteCLZ, [XS2MDU]>;
def : WriteRes<WriteCLZ32, [XS2MDU]>;
def : WriteRes<WriteCTZ, [XS2MDU]>;
def : WriteRes<WriteCTZ32, [XS2MDU]>;
def : WriteRes<WriteCPOP, [XS2MDU]>;
def : WriteRes<WriteCPOP32, [XS2MDU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 113-129: TableGen record definition / TableGen 记录定义
```tablegen
// Zbkc
def : WriteRes<WriteCLMUL, [XS2MDU]>;

// Zbkx
def : WriteRes<WriteXPERM, [XS2MDU]>;
}

// Memory
def : WriteRes<WriteSTB, [XS2ST]>;
def : WriteRes<WriteSTH, [XS2ST]>;
def : WriteRes<WriteSTW, [XS2ST]>;
def : WriteRes<WriteSTD, [XS2ST]>;
def : WriteRes<WriteFST32, [XS2ST]>;
def : WriteRes<WriteFST64, [XS2ST]>;
def : WriteRes<WriteAtomicSTW, [XS2ST]>;
def : WriteRes<WriteAtomicSTD, [XS2ST]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 130-140: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 5 in {
def : WriteRes<WriteLDB, [XS2LD]>;
def : WriteRes<WriteLDH, [XS2LD]>;
def : WriteRes<WriteLDW, [XS2LD]>;
def : WriteRes<WriteLDD, [XS2LD]>;

def : WriteRes<WriteAtomicW, [XS2LD]>;
def : WriteRes<WriteAtomicD, [XS2LD]>;
def : WriteRes<WriteAtomicLDW, [XS2LD]>;
def : WriteRes<WriteAtomicLDD, [XS2LD]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 141-155: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFLD32, [XS2LD]>;
def : WriteRes<WriteFLD64, [XS2LD]>;
}

// XiangShan-NanHu uses FuDian FPU instead of Berkeley HardFloat.
// Documentation: https://github.com/OpenXiangShan/fudian

let Latency = 3 in {
def : WriteRes<WriteFAdd32, [XS2FMAC]>;
def : WriteRes<WriteFSGNJ32, [XS2FMAC]>;
def : WriteRes<WriteFMinMax32, [XS2FMAC]>;
def : WriteRes<WriteFAdd64, [XS2FMAC]>;
def : WriteRes<WriteFSGNJ64, [XS2FMAC]>;
def : WriteRes<WriteFMinMax64, [XS2FMAC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 156-166: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFCvtI32ToF32, [XS2FMAC]>;
def : WriteRes<WriteFCvtI32ToF64, [XS2FMAC]>;
def : WriteRes<WriteFCvtI64ToF32, [XS2FMAC]>;
def : WriteRes<WriteFCvtI64ToF64, [XS2FMAC]>;
def : WriteRes<WriteFCvtF32ToI32, [XS2FMAC]>;
def : WriteRes<WriteFCvtF32ToI64, [XS2FMAC]>;
def : WriteRes<WriteFCvtF64ToI32, [XS2FMAC]>;
def : WriteRes<WriteFCvtF64ToI64, [XS2FMAC]>;
def : WriteRes<WriteFCvtF32ToF64, [XS2FMAC]>;
def : WriteRes<WriteFCvtF64ToF32, [XS2FMAC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 167-176: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFClass32, [XS2FMAC]>;
def : WriteRes<WriteFClass64, [XS2FMAC]>;
def : WriteRes<WriteFCmp32, [XS2FMAC]>;
def : WriteRes<WriteFCmp64, [XS2FMAC]>;
def : WriteRes<WriteFMovF32ToI32, [XS2FMAC]>;
def : WriteRes<WriteFMovI32ToF32, [XS2FMAC]>;
def : WriteRes<WriteFMovF64ToI64, [XS2FMAC]>;
def : WriteRes<WriteFMovI64ToF64, [XS2FMAC]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 177-186: TableGen record definition / TableGen 记录定义
```tablegen
// FP multiplication
let Latency = 3 in {
def : WriteRes<WriteFMul32, [XS2FMAC]>;
def : WriteRes<WriteFMul64, [XS2FMAC]>;
}

let Latency = 5 in {
def : WriteRes<WriteFMA32, [XS2FMAC]>;
def : WriteRes<WriteFMA64, [XS2FMAC]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 187-198: TableGen record definition / TableGen 记录定义
```tablegen

// FP division
def : WriteRes<WriteFDiv32, [XS2FMISC]> {
    let Latency = 11;
}
def : WriteRes<WriteFDiv64, [XS2FMISC]> {
    let Latency = 18;
}

def : WriteRes<WriteFSqrt32, [XS2FMISC]> {
    let Latency = 17;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 199-208: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFSqrt64, [XS2FMISC]> {
    let Latency = 31;
}

// Others
def : WriteRes<WriteCSR, [XS2MISC]>;
def : WriteRes<WriteNop, []>;

def : InstRW<[WriteIALU], (instrs COPY)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 209-228: TableGen class XS2LoadToALUBypass<SchedRead / TableGen 类 XS2LoadToALUBypass<SchedRead
```tablegen
// Bypass and advance

class XS2LoadToALUBypass<SchedRead read>
    : ReadAdvance<read, 1, [WriteLDB, WriteLDH, WriteLDW, WriteLDD, WriteAtomicW, WriteAtomicD, WriteAtomicLDW, WriteAtomicLDD]>;

def : ReadAdvance<ReadJmp, 0>;
def : ReadAdvance<ReadJalr, 0>;
def : ReadAdvance<ReadCSR, 0>;
def : ReadAdvance<ReadStoreData, 0>;
def : ReadAdvance<ReadMemBase, 0>;
def : XS2LoadToALUBypass<ReadIALU>;
def : XS2LoadToALUBypass<ReadIALU32>;
def : XS2LoadToALUBypass<ReadShiftImm>;
def : XS2LoadToALUBypass<ReadShiftImm32>;
def : XS2LoadToALUBypass<ReadShiftReg>;
def : XS2LoadToALUBypass<ReadShiftReg32>;
def : ReadAdvance<ReadIDiv, 0>;
def : ReadAdvance<ReadIDiv32, 0>;
def : ReadAdvance<ReadIRem, 0>;
def : ReadAdvance<ReadIRem32, 0>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 229-248: TableGen record definition / TableGen 记录定义
```tablegen
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
def : ReadAdvance<ReadFMA32Addend, 2>; // Cascade FMA
def : ReadAdvance<ReadFMA64, 0>;
def : ReadAdvance<ReadFMA64Addend, 2>; // Cascade FMA
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 249-268: TableGen record definition / TableGen 记录定义
```tablegen
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
def : ReadAdvance<ReadFCvtI64ToF64, 0>;
def : ReadAdvance<ReadFCvtF32ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 269-288: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFClass64, 0>;

// Zb*
// Zba
def : XS2LoadToALUBypass<ReadSHXADD>;
def : XS2LoadToALUBypass<ReadSHXADD32>;
// Zbb
def : XS2LoadToALUBypass<ReadRotateImm>;
def : XS2LoadToALUBypass<ReadRotateImm32>;
def : XS2LoadToALUBypass<ReadRotateReg>;
def : XS2LoadToALUBypass<ReadRotateReg32>;
def : ReadAdvance<ReadCLZ, 0>;
def : ReadAdvance<ReadCLZ32, 0>;
def : ReadAdvance<ReadCTZ, 0>;
def : ReadAdvance<ReadCTZ32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 289-306: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadCPOP, 0>;
def : ReadAdvance<ReadCPOP32, 0>;
def : XS2LoadToALUBypass<ReadORCB>;
def : XS2LoadToALUBypass<ReadIMinMax>;
def : XS2LoadToALUBypass<ReadREV8>;
// Zbkc
def : ReadAdvance<ReadCLMUL, 0>;
// Zbs
def : XS2LoadToALUBypass<ReadSingleBit>;
def : XS2LoadToALUBypass<ReadSingleBitImm>;
// Zbkb
def : XS2LoadToALUBypass<ReadBREV8>;
def : XS2LoadToALUBypass<ReadPACK>;
def : XS2LoadToALUBypass<ReadPACK32>;
def : XS2LoadToALUBypass<ReadZIP>;
// Zbkx
def : ReadAdvance<ReadXPERM, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 307-317: Bulk record instantiation / 批量记录实例化
```tablegen
//===----------------------------------------------------------------------===//
// Unsupported extensions
defm : UnsupportedSchedQ;
defm : UnsupportedSchedV;
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
