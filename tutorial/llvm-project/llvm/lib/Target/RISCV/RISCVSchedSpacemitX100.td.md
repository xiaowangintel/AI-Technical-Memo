# RISCVSchedSpacemitX100.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedSpacemitX100.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Commentary and design intent / 注释与设计意图
```tablegen
//- RISCVSchedSpacemitX100.td - Spacemit X100 Scheduling Defs -*- tablegen -*-//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
//
// Scheduler model for the SpacemiT-X100 processor.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 14-23: TableGen record SpacemitX100Model / TableGen 记录 SpacemitX100Model
```tablegen

def SpacemitX100Model : SchedMachineModel {
  let IssueWidth = 4;          // 4 micro-ops are dispatched per cycle.
  let MicroOpBufferSize = 192; // Max micro-ops that can be buffered.
                               // 64 entry ROB. Max 3 micro-ops share one entry.
  let LoadLatency = 3;         // Cycles for loads to access the cache.
  let MispredictPenalty = 9;   // Extra cycles for a mispredicted branch.

  let CompleteModel = 0;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 24-39: TableGen record SMTX100_IQ0 / TableGen 记录 SMTX100_IQ0
```tablegen
  let UnsupportedFeatures = [HasStdExtZknd, HasStdExtZkne, HasStdExtZknh,
                             HasStdExtZksed, HasStdExtZksh, HasStdExtZkr,
                             HasVInstructions];
}

let SchedModel = SpacemitX100Model in {

//===----------------------------------------------------------------------===//
// Define processor resources for Spacemit-X100
let BufferSize = 6 in {
  // IQ0, IQ1, BQ: 12 entry queue, can accept 2 ops and issue 1 op per cycle
  // To model the accept bandwidth, split into 2 sub-queues of 6 entry each
  def SMTX100_IQ0 : ProcResource<2>; //Integer Queue 0
  def SMTX100_IQ1 : ProcResource<2>; //Integer Queue 1
  def SMTX100_BQ  : ProcResource<2>; //Branch Queue
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 40-49: TableGen record SMTX100_LSQ / TableGen 记录 SMTX100_LSQ
```tablegen

let BufferSize = 4 in {
  // LSQ: 16 entry queue, can accept 4 ops and issue 2 op per cycle
  // To model the accept bandwidth, split into 4 sub-queues of 4 entry each
  def SMTX100_LSQ : ProcResource<4>; //Load Store Queue
  // FQ0, FQ1: 8 entry queue, can accept 2 ops and issue 1 op per cycle
  // To model the accept bandwidth, split into 2 sub-queues of 4 entry each
  def SMTX100_FQ0 : ProcResource<2>; //Float Queue 0
  def SMTX100_FQ1 : ProcResource<2>; //Float Queue 1
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 50-61: TableGen record SMTX100_IQ / TableGen 记录 SMTX100_IQ
```tablegen

def SMTX100_IQ : ProcResGroup<[SMTX100_IQ0, SMTX100_IQ1]>;
def SMTX100_FQ : ProcResGroup<[SMTX100_FQ0, SMTX100_FQ1]>;

//===----------------------------------------------------------------------===//

// Branching
let Latency = 2 in {
  def : WriteRes<WriteJmp,  [SMTX100_BQ]>;
  def : WriteRes<WriteJal,  [SMTX100_BQ]>;
  def : WriteRes<WriteJalr, [SMTX100_BQ]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 62-74: TableGen record definition / TableGen 记录定义
```tablegen

// Integer arithmetic and logic
def : WriteRes<WriteIALU32,     [SMTX100_IQ]>;
def : WriteRes<WriteIALU,       [SMTX100_IQ]>;
def : WriteRes<WriteShiftImm32, [SMTX100_IQ]>;
def : WriteRes<WriteShiftImm,   [SMTX100_IQ]>;
def : WriteRes<WriteShiftReg32, [SMTX100_IQ]>;
def : WriteRes<WriteShiftReg,   [SMTX100_IQ]>;

// Integer multiplication
def : WriteRes<WriteIMul32, [SMTX100_IQ1]> { let Latency = 2; }
def : WriteRes<WriteIMul,   [SMTX100_IQ1]> { let Latency = 3; }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 75-85: TableGen record definition / TableGen 记录定义
```tablegen
// Integer division/remainder
// Latency is 4-14, Worst case latency is used
let Latency = 14, ReleaseAtCycles = [14] in {
  def : WriteRes<WriteIDiv32, [SMTX100_IQ0]>;
  def : WriteRes<WriteIRem32, [SMTX100_IQ0]>;
}
// Latency is 4-22, Worst case latency is used
let Latency = 22, ReleaseAtCycles = [22] in {
  def : WriteRes<WriteIDiv, [SMTX100_IQ0]>;
  def : WriteRes<WriteIRem, [SMTX100_IQ0]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 86-97: TableGen record definition / TableGen 记录定义
```tablegen

// Bitmanip
def : WriteRes<WriteRotateImm,   [SMTX100_IQ]>;
def : WriteRes<WriteRotateImm32, [SMTX100_IQ]>;
def : WriteRes<WriteRotateReg,   [SMTX100_IQ]>;
def : WriteRes<WriteRotateReg32, [SMTX100_IQ]>;

def : WriteRes<WriteCLZ,   [SMTX100_IQ]>;
def : WriteRes<WriteCLZ32, [SMTX100_IQ]>;
def : WriteRes<WriteCTZ,   [SMTX100_IQ]>;
def : WriteRes<WriteCTZ32, [SMTX100_IQ]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 98-109: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 2 in {
  def : WriteRes<WriteCPOP,   [SMTX100_IQ]>;
  def : WriteRes<WriteCPOP32, [SMTX100_IQ]>;
}

def : WriteRes<WriteORCB,    [SMTX100_IQ]>;
def : WriteRes<WriteIMinMax, [SMTX100_IQ]>;
def : WriteRes<WriteREV8,    [SMTX100_IQ]>;

def : WriteRes<WriteSHXADD,   [SMTX100_IQ]>;
def : WriteRes<WriteSHXADD32, [SMTX100_IQ]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 110-126: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteCLMUL,  [SMTX100_IQ]> { let Latency = 2; }

// Single-bit instructions
def : WriteRes<WriteSingleBit,    [SMTX100_IQ]>;
def : WriteRes<WriteSingleBitImm, [SMTX100_IQ]>;
def : WriteRes<WriteBEXT,  [SMTX100_IQ]>;
def : WriteRes<WriteBEXTI, [SMTX100_IQ]>;

// Memory/Atomic memory
def : WriteRes<WriteSTB, [SMTX100_LSQ]>;
def : WriteRes<WriteSTH, [SMTX100_LSQ]>;
def : WriteRes<WriteSTW, [SMTX100_LSQ]>;
def : WriteRes<WriteSTD, [SMTX100_LSQ]>;
def : WriteRes<WriteFST16, [SMTX100_LSQ]>;
def : WriteRes<WriteFST32, [SMTX100_LSQ]>;
def : WriteRes<WriteFST64, [SMTX100_LSQ]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 127-138: TableGen record definition / TableGen 记录定义
```tablegen
let Latency = 3 in {
  def : WriteRes<WriteLDB, [SMTX100_LSQ]>;
  def : WriteRes<WriteLDH, [SMTX100_LSQ]>;
  def : WriteRes<WriteLDW, [SMTX100_LSQ]>;
  def : WriteRes<WriteLDD, [SMTX100_LSQ]>;
}

let Latency = 4 in {
  def : WriteRes<WriteFLD16, [SMTX100_LSQ]>;
  def : WriteRes<WriteFLD32, [SMTX100_LSQ]>;
  def : WriteRes<WriteFLD64, [SMTX100_LSQ]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 139-149: TableGen record definition / TableGen 记录定义
```tablegen

// Atomics
// Latency is at least 7, not sure worst case latency now
let Latency = 7 in {
  def : WriteRes<WriteAtomicSTW, [SMTX100_LSQ]>;
  def : WriteRes<WriteAtomicSTD, [SMTX100_LSQ]>;
  def : WriteRes<WriteAtomicLDW, [SMTX100_LSQ]>;
  def : WriteRes<WriteAtomicLDD, [SMTX100_LSQ]>;
  def : WriteRes<WriteAtomicW, [SMTX100_LSQ]>;
  def : WriteRes<WriteAtomicD, [SMTX100_LSQ]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 150-159: TableGen record definition / TableGen 记录定义
```tablegen

// Floating point units Half precision
let Latency = 3 in {
  def : WriteRes<WriteFAdd16, [SMTX100_FQ]>;
  def : WriteRes<WriteFMul16, [SMTX100_FQ]>;
  def : WriteRes<WriteFSGNJ16,   [SMTX100_FQ]>;
  def : WriteRes<WriteFMinMax16, [SMTX100_FQ]>;
}
def : WriteRes<WriteFMA16, [SMTX100_FQ]> { let Latency = 5; }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 160-172: TableGen record definition / TableGen 记录定义
```tablegen

// Latency is 4-12, Worst case latency is used
let Latency = 12, ReleaseAtCycles = [12] in {
  def :  WriteRes<WriteFDiv16,  [SMTX100_FQ0]>;
  def :  WriteRes<WriteFSqrt16, [SMTX100_FQ0]>;
}

// Single precision
let Latency = 3 in {
  def : WriteRes<WriteFAdd32,    [SMTX100_FQ]>;
  def : WriteRes<WriteFSGNJ32,   [SMTX100_FQ]>;
  def : WriteRes<WriteFMinMax32, [SMTX100_FQ]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 173-187: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFMul32, [SMTX100_FQ]> { let Latency = 4; }
def : WriteRes<WriteFMA32,  [SMTX100_FQ]> { let Latency = 5; }

// Latency is 4-12, Worst case latency is used
let Latency = 12, ReleaseAtCycles = [12] in {
  def :  WriteRes<WriteFDiv32,  [SMTX100_FQ0]>;
  def :  WriteRes<WriteFSqrt32, [SMTX100_FQ0]>;
}

// Double precision
let Latency = 3 in {
  def : WriteRes<WriteFAdd64,    [SMTX100_FQ]>;
  def : WriteRes<WriteFSGNJ64,   [SMTX100_FQ]>;
  def : WriteRes<WriteFMinMax64, [SMTX100_FQ]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 188-202: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFMul64, [SMTX100_FQ]> { let Latency = 4; }
def : WriteRes<WriteFMA64,  [SMTX100_FQ]> { let Latency = 4; }

// Latency is 4-20, Worst case latency is used
let Latency = 20, ReleaseAtCycles = [20] in {
  def :  WriteRes<WriteFDiv64,  [SMTX100_FQ0]>;
  def :  WriteRes<WriteFSqrt64, [SMTX100_FQ0]>;
}

// Zfa
let Latency = 3 in {
  def : WriteRes<WriteFRoundF16, [SMTX100_FQ1]>;
  def : WriteRes<WriteFRoundF32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFRoundF64, [SMTX100_FQ1]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 203-222: TableGen record definition / TableGen 记录定义
```tablegen
  def : WriteRes<WriteFLI16, [SMTX100_FQ1]>;
  def : WriteRes<WriteFLI32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFLI64, [SMTX100_FQ1]>;
}

// Conversions
let Latency = 3 in {
  def : WriteRes<WriteFCvtF16ToI32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF32ToI32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF32ToI64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF64ToI64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF64ToI32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF16ToI64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtI32ToF16, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtI32ToF32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtI32ToF64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtI64ToF16, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtI64ToF32, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtI64ToF64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF16ToF32, [SMTX100_FQ1]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 223-232: TableGen record definition / TableGen 记录定义
```tablegen
  def : WriteRes<WriteFCvtF16ToF64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF32ToF16, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF32ToF64, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF64ToF16, [SMTX100_FQ1]>;
  def : WriteRes<WriteFCvtF64ToF32, [SMTX100_FQ1]>;

  def : WriteRes<WriteFClass16, [SMTX100_FQ]>;
  def : WriteRes<WriteFClass32, [SMTX100_FQ]>;
  def : WriteRes<WriteFClass64, [SMTX100_FQ]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 233-243: TableGen record definition / TableGen 记录定义
```tablegen
  def : WriteRes<WriteFCmp16, [SMTX100_FQ]>;
  def : WriteRes<WriteFCmp32, [SMTX100_FQ]>;
  def : WriteRes<WriteFCmp64, [SMTX100_FQ]>;

  def : WriteRes<WriteFMovF16ToI16, [SMTX100_FQ]>;
  def : WriteRes<WriteFMovI16ToF16, [SMTX100_FQ]>;
  def : WriteRes<WriteFMovF32ToI32, [SMTX100_FQ]>;
  def : WriteRes<WriteFMovI32ToF32, [SMTX100_FQ]>;
  def : WriteRes<WriteFMovF64ToI64, [SMTX100_FQ]>;
  def : WriteRes<WriteFMovI64ToF64, [SMTX100_FQ]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 244-263: TableGen record definition / TableGen 记录定义
```tablegen

// Others
def : WriteRes<WriteCSR,   [SMTX100_IQ0]>;
def : WriteRes<WriteNop,   [SMTX100_IQ]>;

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
def : ReadAdvance<ReadIDiv32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 264-283: TableGen record definition / TableGen 记录定义
```tablegen
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
def : ReadAdvance<ReadFMul16, 0>;
def : ReadAdvance<ReadFMA16, 0>;
def : ReadAdvance<ReadFMA16Addend, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 284-303: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMul32, 0>;
def : ReadAdvance<ReadFMul64, 0>;
def : ReadAdvance<ReadFMA32, 0>;
def : ReadAdvance<ReadFMA32Addend, 0>;
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
def : ReadAdvance<ReadFSGNJ64, 0>;
def : ReadAdvance<ReadFMinMax16, 0>;
def : ReadAdvance<ReadFMinMax32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 304-323: TableGen record definition / TableGen 记录定义
```tablegen
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
def : ReadAdvance<ReadFCvtF16ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF16, 0>;
def : ReadAdvance<ReadFMovF16ToI16, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 324-343: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFMovI16ToF16, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFClass16, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFClass64, 0>;

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
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 344-357: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadORCB, 0>;
def : ReadAdvance<ReadIMinMax, 0>;
def : ReadAdvance<ReadREV8, 0>;
def : ReadAdvance<ReadSHXADD, 0>;
def : ReadAdvance<ReadSHXADD32, 0>;
def : ReadAdvance<ReadCLMUL, 0>;
// Single-bit instructions
def : ReadAdvance<ReadSingleBit, 0>;
def : ReadAdvance<ReadSingleBitImm, 0>;
// Zfa
def : ReadAdvance<ReadFRoundF32, 0>;
def : ReadAdvance<ReadFRoundF64, 0>;
def : ReadAdvance<ReadFRoundF16, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 358-369: Bulk record instantiation / 批量记录实例化
```tablegen
//===----------------------------------------------------------------------===//
// Unsupported extensions
defm : UnsupportedSchedQ;
defm : UnsupportedSchedV;
defm : UnsupportedSchedZabha;
defm : UnsupportedSchedZbkb;
defm : UnsupportedSchedZbkx;
defm : UnsupportedSchedZfaWithQ;
defm : UnsupportedSchedZvk;
defm : UnsupportedSchedSFB;
defm : UnsupportedSchedXsf;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 370-370: Whitespace and block separation / 空白与代码分段
```tablegen

```
**EN:** This block mainly separates nearby logic and preserves the source layout for readability.

**CN:** 该区段主要用于分隔相邻逻辑，并保持源码布局的可读性。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
