# RISCVSchedTTAscalonX.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedTTAscalonX.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//=- RISCVSchedTTAscalonX.td - TT Ascalon X Sched Defs -------*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//

class AscalonIsWorstCaseMX<string mx, list<string> MxList> {
  defvar LLMUL = LargestLMUL<MxList>.r;
  bit c = !eq(mx, LLMUL);
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-35: TableGen class AscalonIsWorstCaseMXSEW<string / TableGen 类 AscalonIsWorstCaseMXSEW<string
```tablegen

class AscalonIsWorstCaseMXSEW<string mx, int sew, list<string> MxList,
                               bit isF = 0> {
  defvar LLMUL = LargestLMUL<MxList>.r;
  defvar SSEW = SmallestSEW<mx, isF>.r;
  bit c = !and(!eq(mx, LLMUL), !eq(sew, SSEW));
}

/// Cycle counts that scale with LMUL with LMUL=1 having the same latency as
/// fractional LMULs
class AscalonGetCyclesLMUL<string mx, int base> {
  int c = !cond(
    !eq(mx, "M1") : base,
    !eq(mx, "M2") : !mul(base, 2),
    !eq(mx, "M4") : !mul(base, 4),
    !eq(mx, "M8") : !mul(base, 8),
    !eq(mx, "MF2") : base,
    !eq(mx, "MF4") : base,
    !eq(mx, "MF8") : base
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 36-49: TableGen class AscalonGetCyclesLMULFractional<string / TableGen 类 AscalonGetCyclesLMULFractional<string
```tablegen

/// Linear LMUL scaling starting from smallest fractional LMUL
class AscalonGetCyclesLMULFractional<string mx, int base> {
  int c = !cond(
    !eq(mx, "MF8") : base,
    !eq(mx, "MF4") : !mul(base, 2),
    !eq(mx, "MF2") : !mul(base, 4),
    !eq(mx, "M1") : !mul(base, 8),
    !eq(mx, "M2") : !mul(base, 16),
    !eq(mx, "M4") : !mul(base, 32),
    !eq(mx, "M8") : !mul(base, 64)
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 50-63: TableGen class AscalonGetCyclesDefault<string / TableGen 类 AscalonGetCyclesDefault<string
```tablegen
class AscalonGetCyclesDefault<string mx> {
  int c = AscalonGetCyclesLMUL<mx, 1>.c;
}

class AscalonGetCyclesNarrowing<string mx> {
  int c = !cond(
    !eq(mx, "M1") : 4,
    !eq(mx, "M2") : 8,
    !eq(mx, "M4") : 16,
    !eq(mx, "MF2") : 2,
    !eq(mx, "MF4") : 1,
    !eq(mx, "MF8") : 1
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 64-85: TableGen class AscalonGetCyclesDivOrSqrt<string / TableGen 类 AscalonGetCyclesDivOrSqrt<string
```tablegen


class AscalonGetCyclesDivOrSqrt<string mx, int sew> {
  int c = !cond(
    !eq(sew, 8) : AscalonGetCyclesLMUL<mx, 7>.c,
    !eq(sew, 16) : AscalonGetCyclesLMUL<mx, 6>.c,
    !eq(sew, 32) : AscalonGetCyclesLMUL<mx, 5>.c,
    !eq(sew, 64) : AscalonGetCyclesLMUL<mx, 8>.c
  );
}

class AscalonGetCyclesVRGatherVV<string mx> {
  int c = !cond(
    !eq(mx, "M1")  : 2,
    !eq(mx, "M2")  : 4,
    !eq(mx, "M4")  : 12,
    !eq(mx, "M8")  : 48,
    !eq(mx, "MF2") : 2,
    !eq(mx, "MF4") : 2,
    !eq(mx, "MF8") : 2
  );
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 86-103: TableGen class AscalonGetCyclesStridedSegmented<string / TableGen 类 AscalonGetCyclesStridedSegmented<string
```tablegen

class AscalonGetCyclesStridedSegmented<string mx, int sew> {
  int c = !cond(
    !eq(sew, 8) : AscalonGetCyclesLMULFractional<mx, 4>.c,
    !eq(sew, 16) : AscalonGetCyclesLMULFractional<mx, 2>.c,
    !eq(sew, 32) : AscalonGetCyclesLMULFractional<mx, 1>.c,
    !eq(sew, 64) : AscalonGetCyclesLMULFractional<mx, 1>.c
  );
}

//===----------------------------------------------------------------------===//

def TTAscalonXModel : SchedMachineModel {
  let IssueWidth        =   8; // 8-way decode and dispatch
  let MicroOpBufferSize = 256; // 256 micro-op re-order buffer
  let LoadLatency       =   4; // Optimistic load latency
  let MispredictPenalty =  14; // Fetch + Decode/Rename/Dispatch + Branch
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 104-117: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let CompleteModel = false;

  // TODO: supported, but haven't added scheduling info yet.
  let UnsupportedFeatures = [HasStdExtZbkb, HasStdExtZbkc, HasStdExtZbkx,
                             HasStdExtZcmt, HasStdExtZknd, HasStdExtZkne,
                             HasStdExtZknh, HasStdExtZksed, HasStdExtZksh,
                             HasStdExtZkr];
}

let SchedModel = TTAscalonXModel in {

//===----------------------------------------------------------------------===//
// Define each kind of processor resource and number available.
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 118-133: TableGen record AscalonLS / TableGen 记录 AscalonLS
```tablegen
let BufferSize = 16 in {
  def AscalonLS : ProcResource<3>;
  def AscalonFXA : ProcResource<1>; // ALU, FP/VEC -> INT, MUL, DIV, CSR
  def AscalonFXB : ProcResource<1>; // ALU, INT -> FP/VEC
  def AscalonFXC : ProcResource<2>; // ALU, BR
  def AscalonFXD : ProcResource<2>; // ALU
  def AscalonFX : ProcResGroup<[AscalonFXA, AscalonFXB, AscalonFXC, AscalonFXD]>;
  // FP
  def AscalonFPA : ProcResource<1>; // Pipe A also handles FP/VEC -> INT
  def AscalonFPB : ProcResource<1>;
  def AscalonFP : ProcResGroup<[AscalonFPA, AscalonFPB]>;
  // Vector
  def AscalonVA : ProcResource<1>;
  def AscalonVB : ProcResource<1>;
  def AscalonV : ProcResGroup<[AscalonVA, AscalonVB]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 134-150: TableGen record definition / TableGen 记录定义
```tablegen


//===----------------------------------------------------------------------===//

// Branching
def : WriteRes<WriteJmp, [AscalonFXC]>;
def : WriteRes<WriteJal, [AscalonFXC]>;
def : WriteRes<WriteJalr, [AscalonFXC]>;

// Integer arithmetic and logic
def : WriteRes<WriteIALU32, [AscalonFX]>;
def : WriteRes<WriteIALU, [AscalonFX]>;
def : WriteRes<WriteShiftImm32, [AscalonFX]>;
def : WriteRes<WriteShiftImm, [AscalonFX]>;
def : WriteRes<WriteShiftReg32, [AscalonFX]>;
def : WriteRes<WriteShiftReg, [AscalonFX]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 151-165: TableGen record definition / TableGen 记录定义
```tablegen
// Integer multiplication
let Latency = 3 in {
def : WriteRes<WriteIMul, [AscalonFXA]>;
def : WriteRes<WriteIMul32, [AscalonFXA]>;
}

// Integer division
// Worst case latency is used.

let Latency = 7, ReleaseAtCycles = [7] in {
  def : WriteRes<WriteIDiv32, [AscalonFXA]>;
  def : WriteRes<WriteIDiv, [AscalonFXA]>;
  def : WriteRes<WriteIRem32, [AscalonFXA]>;
  def : WriteRes<WriteIRem, [AscalonFXA]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 166-180: TableGen record definition / TableGen 记录定义
```tablegen

// Bitmanip
def : WriteRes<WriteRotateImm, [AscalonFX]>;
def : WriteRes<WriteRotateImm32, [AscalonFX]>;
def : WriteRes<WriteRotateReg, [AscalonFX]>;
def : WriteRes<WriteRotateReg32, [AscalonFX]>;

def : WriteRes<WriteCLZ, [AscalonFX]>;
def : WriteRes<WriteCLZ32, [AscalonFX]>;
def : WriteRes<WriteCTZ, [AscalonFX]>;
def : WriteRes<WriteCTZ32, [AscalonFX]>;

def : WriteRes<WriteCPOP, [AscalonFX]>;
def : WriteRes<WriteCPOP32, [AscalonFX]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 181-195: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteORCB, [AscalonFX]>;

def : WriteRes<WriteIMinMax, [AscalonFX]>;

def : WriteRes<WriteREV8, [AscalonFX]>;

def : WriteRes<WriteSHXADD, [AscalonFX]>;
def : WriteRes<WriteSHXADD32, [AscalonFX]>;

// Single-bit instructions
def : WriteRes<WriteSingleBit, [AscalonFX]>;
def : WriteRes<WriteSingleBitImm, [AscalonFX]>;
def : WriteRes<WriteBEXT, [AscalonFX]>;
def : WriteRes<WriteBEXTI, [AscalonFX]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 196-213: TableGen record definition / TableGen 记录定义
```tablegen
// Memory
def : WriteRes<WriteSTB, [AscalonLS]>;
def : WriteRes<WriteSTH, [AscalonLS]>;
def : WriteRes<WriteSTW, [AscalonLS]>;
def : WriteRes<WriteSTD, [AscalonLS]>;
def : WriteRes<WriteFST16, [AscalonLS]>;
def : WriteRes<WriteFST32, [AscalonLS]>;
def : WriteRes<WriteFST64, [AscalonLS]>;

let Latency = 4 in {
def : WriteRes<WriteLDB, [AscalonLS]>;
def : WriteRes<WriteLDH, [AscalonLS]>;
def : WriteRes<WriteLDW, [AscalonLS]>;
def : WriteRes<WriteLDD, [AscalonLS]>;
def : WriteRes<WriteFLD16, [AscalonLS]>;
def : WriteRes<WriteFLD32, [AscalonLS]>;
def : WriteRes<WriteFLD64, [AscalonLS]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 214-233: TableGen record definition / TableGen 记录定义
```tablegen

// Atomic memory
def : WriteRes<WriteAtomicSTW, [AscalonLS]>;
def : WriteRes<WriteAtomicSTD, [AscalonLS]>;

let Latency = 4 in {
def : WriteRes<WriteAtomicW, [AscalonLS]>;
def : WriteRes<WriteAtomicD, [AscalonLS]>;
def : WriteRes<WriteAtomicLDW, [AscalonLS]>;
def : WriteRes<WriteAtomicLDD, [AscalonLS]>;
}

// Half precision.
let Latency = 3 in {
def : WriteRes<WriteFAdd16, [AscalonFP]>;
def : WriteRes<WriteFMul16, [AscalonFP]>;
def : WriteRes<WriteFMA16, [AscalonFP]>;
def : WriteRes<WriteFSGNJ16, [AscalonFP]>;
def : WriteRes<WriteFMinMax16, [AscalonFP]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 234-247: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 7, ReleaseAtCycles = [7] in {
def :  WriteRes<WriteFDiv16, [AscalonFP]>;
def :  WriteRes<WriteFSqrt16, [AscalonFP]>;
}

// Single precision.
let Latency = 3 in {
def : WriteRes<WriteFAdd32, [AscalonFP]>;
def : WriteRes<WriteFMul32, [AscalonFP]>;
def : WriteRes<WriteFMA32, [AscalonFP]>;
def : WriteRes<WriteFSGNJ32, [AscalonFP]>;
def : WriteRes<WriteFMinMax32, [AscalonFP]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 248-261: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 7, ReleaseAtCycles = [7] in {
def :  WriteRes<WriteFDiv32, [AscalonFP]>;
def :  WriteRes<WriteFSqrt32, [AscalonFP]>;
}

// Double precision
let Latency = 3 in {
def : WriteRes<WriteFAdd64, [AscalonFP]>;
def : WriteRes<WriteFMul64, [AscalonFP]>;
def : WriteRes<WriteFMA64, [AscalonFP]>;
def : WriteRes<WriteFSGNJ64, [AscalonFP]>;
def : WriteRes<WriteFMinMax64, [AscalonFP]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 262-287: TableGen record definition / TableGen 记录定义
```tablegen

let Latency = 12, ReleaseAtCycles = [12] in {
def :  WriteRes<WriteFDiv64, [AscalonFP]>;
def :  WriteRes<WriteFSqrt64, [AscalonFP]>;
}

// Conversions
def : WriteRes<WriteFCvtI32ToF16, [AscalonFXB]>;
def : WriteRes<WriteFCvtI32ToF32, [AscalonFXB]>;
def : WriteRes<WriteFCvtI32ToF64, [AscalonFXB]>;
def : WriteRes<WriteFCvtI64ToF16, [AscalonFXB]>;
def : WriteRes<WriteFCvtI64ToF32, [AscalonFXB]>;
def : WriteRes<WriteFCvtI64ToF64, [AscalonFXB]>;
def : WriteRes<WriteFCvtF16ToI32, [AscalonFXA]>;
def : WriteRes<WriteFCvtF16ToI64, [AscalonFXA]>;
def : WriteRes<WriteFCvtF16ToF32, [AscalonFP]>;
def : WriteRes<WriteFCvtF16ToF64, [AscalonFP]>;
def : WriteRes<WriteFCvtF32ToI32, [AscalonFXA]>;
def : WriteRes<WriteFCvtF32ToI64, [AscalonFXA]>;
def : WriteRes<WriteFCvtF32ToF16, [AscalonFP]>;
def : WriteRes<WriteFCvtF32ToF64, [AscalonFP]>;
def : WriteRes<WriteFCvtF64ToI32, [AscalonFXA]>;
def : WriteRes<WriteFCvtF64ToI64, [AscalonFXA]>;
def : WriteRes<WriteFCvtF64ToF16, [AscalonFP]>;
def : WriteRes<WriteFCvtF64ToF32, [AscalonFP]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 288-301: TableGen record definition / TableGen 记录定义
```tablegen
def : WriteRes<WriteFClass16, [AscalonFP]>;
def : WriteRes<WriteFClass32, [AscalonFP]>;
def : WriteRes<WriteFClass64, [AscalonFP]>;
def : WriteRes<WriteFCmp16, [AscalonFP]>;
def : WriteRes<WriteFCmp32, [AscalonFP]>;
def : WriteRes<WriteFCmp64, [AscalonFP]>;

def : WriteRes<WriteFMovI16ToF16, [AscalonFXB]>;
def : WriteRes<WriteFMovF16ToI16, [AscalonFXA]>;
def : WriteRes<WriteFMovI32ToF32, [AscalonFXB]>;
def : WriteRes<WriteFMovF32ToI32, [AscalonFXA]>;
def : WriteRes<WriteFMovI64ToF64, [AscalonFXB]>;
def : WriteRes<WriteFMovF64ToI64, [AscalonFXA]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 302-329: TableGen record definition / TableGen 记录定义
```tablegen
// Others
def : WriteRes<WriteCSR, [AscalonFXA]>;
def : WriteRes<WriteNop, [AscalonFX]>;

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
def : ReadAdvance<ReadIDiv32, 0>;
def : ReadAdvance<ReadIRem, 0>;
def : ReadAdvance<ReadIRem32, 0>;
def : ReadAdvance<ReadIMul, 0>;
def : ReadAdvance<ReadIMul32, 0>;
def : ReadAdvance<ReadAtomicWA, 0>;
def : ReadAdvance<ReadAtomicWD, 0>;
def : ReadAdvance<ReadAtomicDA, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 330-357: TableGen record definition / TableGen 记录定义
```tablegen
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
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 358-385: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFSGNJ16, 0>;
def : ReadAdvance<ReadFSGNJ32, 0>;
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
def : ReadAdvance<ReadFCvtF16ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF16, 0>;
def : ReadAdvance<ReadFMovF16ToI16, 0>;
def : ReadAdvance<ReadFMovI16ToF16, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 386-411: TableGen record definition / TableGen 记录定义
```tablegen
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
def : ReadAdvance<ReadORCB, 0>;
def : ReadAdvance<ReadIMinMax, 0>;
def : ReadAdvance<ReadREV8, 0>;
def : ReadAdvance<ReadSHXADD, 0>;
def : ReadAdvance<ReadSHXADD32, 0>;
// Single-bit instructions
def : ReadAdvance<ReadSingleBit, 0>;
def : ReadAdvance<ReadSingleBitImm, 0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 412-430: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
//===----------------------------------------------------------------------===//
// Vector
def : WriteRes<WriteRdVLENB, [AscalonFXA]>;

// Configuration-Setting Instructions
def : WriteRes<WriteVSETVLI, [AscalonV]>;
def : WriteRes<WriteVSETIVLI, [AscalonV]>;
let Latency = 2 in {
  def : WriteRes<WriteVSETVL, [AscalonV]>;
}

// Vector Loads and Stores
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles in {
    defm "" : LMULWriteResMX<"WriteVLDE",    [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDFF",   [AscalonLS], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 431-450: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  defm "" : LMULWriteResMX<"WriteVSTE",    [AscalonLS], mx, IsWorstCase>;
}

foreach mx = SchedMxList in {
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  defm "" : LMULWriteResMX<"WriteVLDM",    [AscalonLS], mx, IsWorstCase>;
  defm "" : LMULWriteResMX<"WriteVSTM",    [AscalonLS], mx, IsWorstCase>;
}

foreach mx = SchedMxListEEW8 in {
  defvar Cycles = AscalonGetCyclesLMUL<mx, 2>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles in {
    defm "" : LMULWriteResMX<"WriteVLDS8", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDUX8", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDOX8", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTS8",  [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTUX8", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTOX8", [AscalonLS], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 451-474: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
foreach mx = SchedMxListEEW16 in {
  defvar Cycles = AscalonGetCyclesLMUL<mx, 2>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles in {
    defm "" : LMULWriteResMX<"WriteVLDS16", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDUX16", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDOX16", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTS16",  [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTUX16", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTOX16", [AscalonLS], mx, IsWorstCase>;
  }
}
foreach mx = SchedMxListEEW32 in {
  defvar Cycles = AscalonGetCyclesLMUL<mx, 2>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles in {
    defm "" : LMULWriteResMX<"WriteVLDS32", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDUX32", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDOX32", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTS32",  [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTUX32", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTOX32", [AscalonLS], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 475-488: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
foreach mx = SchedMxListEEW64 in {
  defvar Cycles = AscalonGetCyclesLMUL<mx, 2>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles in {
    defm "" : LMULWriteResMX<"WriteVLDS64", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDUX64", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVLDOX64", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTS64",  [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTUX64", [AscalonLS], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSTOX64", [AscalonLS], mx, IsWorstCase>;
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 489-509: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
// VLD*R is not LMUL aware
def : WriteRes<WriteVLD1R,  [AscalonLS]>;
def : WriteRes<WriteVLD2R,  [AscalonLS]>;
def : WriteRes<WriteVLD4R,  [AscalonLS]>;
def : WriteRes<WriteVLD8R,  [AscalonLS]>;
// VST*R is not LMUL aware
def : WriteRes<WriteVST1R,   [AscalonLS]>;
def : WriteRes<WriteVST2R,   [AscalonLS]>;
def : WriteRes<WriteVST4R,   [AscalonLS]>;
def : WriteRes<WriteVST8R,   [AscalonLS]>;

// Segmented Loads and Stores
foreach mx = SchedMxList in {
  foreach eew = [8, 16, 32, 64] in {
    foreach nf=2-8 in {
      defvar Cycles = AscalonGetCyclesDefault<mx>.c;
      defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
      let Latency = Cycles in {
        defm "" : LMULWriteResMX<"WriteVLSEG" # nf # "e" # eew,   [AscalonLS], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVLSEGFF" # nf # "e" # eew, [AscalonLS], mx, IsWorstCase>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 510-527: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      let Latency = 1, AcquireAtCycles = [1], ReleaseAtCycles = [!add(1, Cycles)] in
      defm "" : LMULWriteResMX<"WriteVSSEG" # nf # "e" # eew,   [AscalonLS], mx, IsWorstCase>;
    }
  }
}
foreach mx = SchedMxList in {
  foreach nf=2-8 in {
    foreach eew = [8, 16, 32, 64] in {
      defvar Cycles = AscalonGetCyclesStridedSegmented<mx, eew>.c;
      defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
      let Latency = Cycles in {
        defm "" : LMULWriteResMX<"WriteVLSSEG" # nf # "e" # eew,  [AscalonLS], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVLUXSEG" # nf # "e" # eew, [AscalonLS], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVLOXSEG" # nf # "e" # eew, [AscalonLS], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVSSSEG" # nf # "e" # eew,  [AscalonLS], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVSUXSEG" # nf # "e" # eew, [AscalonLS], mx, IsWorstCase>;
        defm "" : LMULWriteResMX<"WriteVSOXSEG" # nf # "e" # eew, [AscalonLS], mx, IsWorstCase>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 528-547: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    }
  }
}

// Vector Fixed-Point Arithmetic Instructions
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVSALUV",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSALUX",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSALUI",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVAALUV",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVAALUX",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSMulV",   [AscalonFXA, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSMulX",   [AscalonFXA, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSShiftV", [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSShiftX", [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSShiftI", [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 548-563: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
// Narrowing
foreach mx = SchedMxListW in {
  defvar Cycles = AscalonGetCyclesNarrowing<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxListW>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVNClipV",  [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNClipX",  [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNClipI",  [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
}

// Configuration-Setting Instructions
def : ReadAdvance<ReadVSETVLI, 1>;
def : ReadAdvance<ReadVSETVL, 1>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 564-591: Bulk record instantiation / 批量记录实例化
```tablegen
// Vector Loads and Stores
def : ReadAdvance<ReadVLDX, 0>;
def : ReadAdvance<ReadVSTX, 0>;
defm "" : LMULReadAdvance<"ReadVSTEV", 0>;
defm "" : LMULReadAdvance<"ReadVSTM", 0>;
def : ReadAdvance<ReadVLDSX, 0>;
def : ReadAdvance<ReadVSTSX, 0>;
defm "" : LMULReadAdvance<"ReadVSTS8V", 0>;
defm "" : LMULReadAdvance<"ReadVSTS16V", 0>;
defm "" : LMULReadAdvance<"ReadVSTS32V", 0>;
defm "" : LMULReadAdvance<"ReadVSTS64V", 0>;
defm "" : LMULReadAdvance<"ReadVLDUXV", 0>;
defm "" : LMULReadAdvance<"ReadVLDOXV", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX8", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX16", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX32", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX64", 0>;
defm "" : LMULReadAdvance<"ReadVSTUXV", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX8V", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX16V", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX32V", 0>;
defm "" : LMULReadAdvance<"ReadVSTUX64V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX8", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX16", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX32", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX64", 0>;
defm "" : LMULReadAdvance<"ReadVSTOXV", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX8V", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 592-619: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
defm "" : LMULReadAdvance<"ReadVSTOX16V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX32V", 0>;
defm "" : LMULReadAdvance<"ReadVSTOX64V", 0>;
// LMUL Aware
def : ReadAdvance<ReadVST1R, 0>;
def : ReadAdvance<ReadVST2R, 0>;
def : ReadAdvance<ReadVST4R, 0>;
def : ReadAdvance<ReadVST8R, 0>;

// Vector Integer Arithmetic Instructions
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIALUV",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIALUX",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIALUI",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUV",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUX",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUI",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUMV",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUMX",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICALUMI",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVShiftV",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVShiftX",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVShiftI",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMinMaxV",  [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMinMaxX",  [AscalonFX, AscalonV], mx, IsWorstCase>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 620-633: Bulk record instantiation / 批量记录实例化
```tablegen
    defm "" : LMULWriteResMX<"WriteVIMulV",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMulX",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMulAddV",  [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMulAddX",  [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMergeV",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMergeX",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMergeI",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMovV",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMovX",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIMovI",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICmpV",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICmpX",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVICmpI",     [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 634-649: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVExtV",      [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
}
foreach mx = SchedMxList in {
  foreach sew = SchedSEWSet<mx>.val in {
    defvar Cycles = AscalonGetCyclesDivOrSqrt<mx, sew>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVIDivV", [AscalonFX, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVIDivX", [AscalonFX, AscalonV], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 650-665: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
}

// Widening
foreach mx = SchedMxListW in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxListW>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIWALUV",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWALUX",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWALUI",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulV",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulX",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulAddV", [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIWMulAddX", [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 666-693: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
// Narrowing
foreach mx = SchedMxListW in {
  defvar Cycles = AscalonGetCyclesNarrowing<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxListW>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVNShiftV",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNShiftX",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVNShiftI",   [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
}

// Vector Floating-Point Instructions
foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, isF=1>.val in {
    defvar Cycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListF, isF=1>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFALUV",  [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFALUF",  [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulV",  [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulF",  [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulAddV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMulAddF", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRecpV",   [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFCvtIToFV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMinMaxV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFMinMaxF", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 694-709: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
      defm "" : LMULSEWWriteResMXSEW<"WriteVFSgnjV",   [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFSgnjF",   [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
    }
  }
}
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVFCvtFToIV",  [AscalonFPA, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFClassV",    [AscalonFP, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFMergeV",    [AscalonFP, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFMovV",      [AscalonFP, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFCmpV",      [AscalonFP, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFCmpF",      [AscalonFP, AscalonV], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 710-730: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, isF=1>.val in {
    defvar Cycles = AscalonGetCyclesDivOrSqrt<mx, sew>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFSqrtV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFDivV",  [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFDivF",  [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
    }
  }
}

// Widening
foreach mx = SchedMxListW in {
  foreach sew = SchedSEWSet<mx, isF=0, isWidening=1>.val in {
    defvar Cycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListW>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in
    defm "" : LMULSEWWriteResMXSEW<"WriteVFWCvtIToFV", [AscalonFXB, AscalonV], mx, sew, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 731-744: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}
foreach mx = SchedMxListFW in {
  foreach sew = SchedSEWSet<mx, isF=1, isWidening=1>.val in {
    defvar Cycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListFW, isF=1>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWALUV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWALUF", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulF", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulAddV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWMulAddF", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFWCvtFToFV", [AscalonFP, AscalonV], mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 745-758: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxListFW>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in
  defm "" : LMULWriteResMX<"WriteVFWCvtFToIV", [AscalonFPA, AscalonV], mx, IsWorstCase>;
}
// Narrowing
foreach mx = SchedMxListW in {
  defvar Cycles = AscalonGetCyclesNarrowing<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxListW>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVFNCvtFToIV", [AscalonFPA, AscalonV], mx, IsWorstCase>;
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 759-780: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
foreach mx = SchedMxListFW in {
  foreach sew = SchedSEWSet<mx, isF=1, isWidening=1>.val in {
    defvar Cycles = AscalonGetCyclesNarrowing<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListFW, isF=1>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFNCvtIToFV", [AscalonFXB, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFNCvtFToFV", [AscalonFXB, AscalonV], mx, sew, IsWorstCase>;
    }
  }
}

// Vector Reduction Instructions
foreach mx = SchedMxList in {
  foreach sew = SchedSEWSet<mx>.val in {
    defvar Cycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVIRedV_From", [AscalonFX, AscalonV],
                                     mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVIRedMinMaxV_From", [AscalonFX, AscalonV],
                                     mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 781-803: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
  }
}

foreach mx = SchedMxListWRed in {
  foreach sew = SchedSEWSet<mx, 0, 1>.val in {
    defvar Cycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListWRed>.c;
    let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in
    defm "" : LMULSEWWriteResMXSEW<"WriteVIWRedV_From", [AscalonFX, AscalonV],
                                   mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxListF in {
  foreach sew = SchedSEWSet<mx, 1>.val in {
    defvar RedCycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListF, 1>.c;
    let Latency = RedCycles, ReleaseAtCycles = [1, RedCycles] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRedV_From", [AscalonFX, AscalonV],
                                     mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVFRedMinMaxV_From", [AscalonFX, AscalonV],
                                     mx, sew, IsWorstCase>;
    }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 804-822: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
    defvar OrdRedCycles = AscalonGetCyclesLMUL<mx, 18>.c;
    let Latency = OrdRedCycles, ReleaseAtCycles = [1, OrdRedCycles] in
    defm "" : LMULSEWWriteResMXSEW<"WriteVFRedOV_From", [AscalonFX, AscalonV],
                                   mx, sew, IsWorstCase>;
  }
}

foreach mx = SchedMxListFWRed in {
  foreach sew = SchedSEWSet<mx, 1, 1>.val in {
    defvar RedCycles = AscalonGetCyclesDefault<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxListFWRed, 1>.c;
    let Latency = RedCycles, ReleaseAtCycles = [1, RedCycles] in
    defm "" : LMULSEWWriteResMXSEW<"WriteVFWRedV_From", [AscalonFX, AscalonV],
                                   mx, sew, IsWorstCase>;
    defvar OrdRedCycles = AscalonGetCyclesLMUL<mx, 18>.c;
    let Latency = OrdRedCycles, ReleaseAtCycles = [1, OrdRedCycles] in
    defm "" : LMULSEWWriteResMXSEW<"WriteVFWRedOV_From", [AscalonFX, AscalonV],
                                   mx, sew, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 823-841: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}

// Vector Mask Instructions
foreach mx = SchedMxList in {
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  defm "" : LMULWriteResMX<"WriteVMALUV", [AscalonV], mx, IsWorstCase>;
  defm "" : LMULWriteResMX<"WriteVMSFSV", [AscalonV], mx, IsWorstCase>;
  let Latency = 2, ReleaseAtCycles = [1, 2] in {
    defm "" : LMULWriteResMX<"WriteVMPopV", [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVMFFSV", [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
}
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = Cycles, ReleaseAtCycles = [1, Cycles] in {
    defm "" : LMULWriteResMX<"WriteVIotaV", [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVIdxV", [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 842-857: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}

// Vector Permutation Instructions
let Latency = 2, ReleaseAtCycles = [1, 2] in {
  def : WriteRes<WriteVMovSX, [AscalonFX, AscalonV]>;
  def : WriteRes<WriteVMovXS, [AscalonFX, AscalonV]>;
  def : WriteRes<WriteVMovSF, [AscalonFX, AscalonV]>;
  def : WriteRes<WriteVMovFS, [AscalonFX, AscalonV]>;
}
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = !mul(Cycles, 2), ReleaseAtCycles = [Cycles, !mul(Cycles, 2)] in {
    defm "" : LMULWriteResMX<"WriteVRGatherVX",    [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVRGatherVI",    [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 858-871: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
}

foreach mx = SchedMxList in {
  foreach sew = SchedSEWSet<mx>.val in {
    defvar Cycles = AscalonGetCyclesVRGatherVV<mx>.c;
    defvar IsWorstCase = AscalonIsWorstCaseMXSEW<mx, sew, SchedMxList>.c;
    let Latency = !add(Cycles, 3), ReleaseAtCycles = [1, !add(1, Cycles)] in {
      defm "" : LMULSEWWriteResMXSEW<"WriteVRGatherVV", [AscalonFX, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVRGatherEI16VV", [AscalonFX, AscalonV], mx, sew, IsWorstCase>;
      defm "" : LMULSEWWriteResMXSEW<"WriteVCompressV", [AscalonFX, AscalonV], mx, sew, IsWorstCase>;
    }
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 872-893: Bulk record instantiation mx / 批量记录实例化 mx
```tablegen
foreach mx = SchedMxList in {
  defvar Cycles = AscalonGetCyclesDefault<mx>.c;
  defvar IsWorstCase = AscalonIsWorstCaseMX<mx, SchedMxList>.c;
  let Latency = 4, AcquireAtCycles = [0, 1], ReleaseAtCycles = [1, !add(1, Cycles)] in {
    defm "" : LMULWriteResMX<"WriteVSlideUpX",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSlideDownX", [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVSlideI",     [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVISlide1X",   [AscalonFX, AscalonV], mx, IsWorstCase>;
    defm "" : LMULWriteResMX<"WriteVFSlide1F",   [AscalonFX, AscalonV], mx, IsWorstCase>;
  }
}

// Whole vector register move, vmv<N>.v, not LMUL aware
let Latency = 1, ReleaseAtCycles = [1] in
  def : WriteRes<WriteVMov1V,     [AscalonV]>;
let Latency = 2, ReleaseAtCycles = [2] in
  def : WriteRes<WriteVMov2V,     [AscalonV]>;
let Latency = 4, ReleaseAtCycles = [4] in
  def : WriteRes<WriteVMov4V,     [AscalonV]>;
let Latency = 8, ReleaseAtCycles = [8] in
  def : WriteRes<WriteVMov8V,     [AscalonV]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 894-921: Bulk record instantiation / 批量记录实例化
```tablegen
// Vector Integer Arithmetic Instructions
defm : LMULReadAdvance<"ReadVIALUV", 0>;
defm : LMULReadAdvance<"ReadVIALUX", 0>;
defm : LMULReadAdvanceW<"ReadVIWALUV", 0>;
defm : LMULReadAdvanceW<"ReadVIWALUX", 0>;
defm : LMULReadAdvance<"ReadVExtV", 0>;
defm : LMULReadAdvance<"ReadVICALUV", 0>;
defm : LMULReadAdvance<"ReadVICALUX", 0>;
defm : LMULReadAdvance<"ReadVShiftV", 0>;
defm : LMULReadAdvance<"ReadVShiftX", 0>;
defm : LMULReadAdvanceW<"ReadVNShiftV", 0>;
defm : LMULReadAdvanceW<"ReadVNShiftX", 0>;
defm : LMULReadAdvance<"ReadVICmpV", 0>;
defm : LMULReadAdvance<"ReadVICmpX", 0>;
defm : LMULReadAdvance<"ReadVIMinMaxV", 0>;
defm : LMULReadAdvance<"ReadVIMinMaxX", 0>;
defm : LMULReadAdvance<"ReadVIMulV", 0>;
defm : LMULReadAdvance<"ReadVIMulX", 0>;
defm : LMULSEWReadAdvance<"ReadVIDivV", 0>;
defm : LMULSEWReadAdvance<"ReadVIDivX", 0>;
defm : LMULReadAdvanceW<"ReadVIWMulV", 0>;
defm : LMULReadAdvanceW<"ReadVIWMulX", 0>;
defm : LMULReadAdvance<"ReadVIMulAddV", 0>;
defm : LMULReadAdvance<"ReadVIMulAddX", 0>;
defm : LMULReadAdvanceW<"ReadVIWMulAddV", 0>;
defm : LMULReadAdvanceW<"ReadVIWMulAddX", 0>;
defm : LMULReadAdvance<"ReadVIMergeV", 0>;
defm : LMULReadAdvance<"ReadVIMergeX", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 922-936: Bulk record instantiation / 批量记录实例化
```tablegen
defm : LMULReadAdvance<"ReadVIMovV", 0>;
defm : LMULReadAdvance<"ReadVIMovX", 0>;

// Vector Fixed-Point Arithmetic Instructions
defm "" : LMULReadAdvance<"ReadVSALUV", 0>;
defm "" : LMULReadAdvance<"ReadVSALUX", 0>;
defm "" : LMULReadAdvance<"ReadVAALUV", 0>;
defm "" : LMULReadAdvance<"ReadVAALUX", 0>;
defm "" : LMULReadAdvance<"ReadVSMulV", 0>;
defm "" : LMULReadAdvance<"ReadVSMulX", 0>;
defm "" : LMULReadAdvance<"ReadVSShiftV", 0>;
defm "" : LMULReadAdvance<"ReadVSShiftX", 0>;
defm "" : LMULReadAdvanceW<"ReadVNClipV", 0>;
defm "" : LMULReadAdvanceW<"ReadVNClipX", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 937-964: Bulk record instantiation / 批量记录实例化
```tablegen
// Vector Floating-Point Instructions
defm "" : LMULSEWReadAdvanceF<"ReadVFALUV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFALUF", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWALUV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWALUF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFDivV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFDivF", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulAddV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMulAddF", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulAddV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWMulAddF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFSqrtV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFRecpV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMinMaxV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFMinMaxF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFSgnjV", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFSgnjF", 0>;
defm "" : LMULReadAdvance<"ReadVFCmpV", 0>;
defm "" : LMULReadAdvance<"ReadVFCmpF", 0>;
defm "" : LMULReadAdvance<"ReadVFClassV", 0>;
defm "" : LMULReadAdvance<"ReadVFMergeV", 0>;
defm "" : LMULReadAdvance<"ReadVFMergeF", 0>;
defm "" : LMULReadAdvance<"ReadVFMovF", 0>;
defm "" : LMULSEWReadAdvanceF<"ReadVFCvtIToFV", 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 965-986: Bulk record instantiation / 批量记录实例化
```tablegen
defm "" : LMULReadAdvance<"ReadVFCvtFToIV", 0>;
defm "" : LMULSEWReadAdvanceW<"ReadVFWCvtIToFV", 0>;
defm "" : LMULReadAdvanceFW<"ReadVFWCvtFToIV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFWCvtFToFV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFNCvtIToFV", 0>;
defm "" : LMULReadAdvanceW<"ReadVFNCvtFToIV", 0>;
defm "" : LMULSEWReadAdvanceFW<"ReadVFNCvtFToFV", 0>;

// Vector Reduction Instructions
def : ReadAdvance<ReadVIRedV, 0>;
def : ReadAdvance<ReadVIRedV0, 0>;
def : ReadAdvance<ReadVIWRedV, 0>;
def : ReadAdvance<ReadVIWRedV0, 0>;
def : ReadAdvance<ReadVFRedV, 0>;
def : ReadAdvance<ReadVFRedV0, 0>;
def : ReadAdvance<ReadVFRedOV, 0>;
def : ReadAdvance<ReadVFRedOV0, 0>;
def : ReadAdvance<ReadVFWRedV, 0>;
def : ReadAdvance<ReadVFWRedV0, 0>;
def : ReadAdvance<ReadVFWRedOV, 0>;
def : ReadAdvance<ReadVFWRedOV0, 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 987-1014: Bulk record instantiation / 批量记录实例化
```tablegen
// Vector Mask Instructions
defm "" : LMULReadAdvance<"ReadVMALUV", 0>;
defm "" : LMULReadAdvance<"ReadVMPopV", 0>;
defm "" : LMULReadAdvance<"ReadVMFFSV", 0>;
defm "" : LMULReadAdvance<"ReadVMSFSV", 0>;
defm "" : LMULReadAdvance<"ReadVIotaV", 0>;

// Vector Permutation Instructions
def : ReadAdvance<ReadVMovXS, 0>;
def : ReadAdvance<ReadVMovSX_V, 0>;
def : ReadAdvance<ReadVMovSX_X, 0>;
def : ReadAdvance<ReadVMovFS, 0>;
def : ReadAdvance<ReadVMovSF_V, 0>;
def : ReadAdvance<ReadVMovSF_F, 0>;
defm "" : LMULReadAdvance<"ReadVISlideV", 0>;
defm "" : LMULReadAdvance<"ReadVISlideX", 0>;
defm "" : LMULReadAdvance<"ReadVFSlideV", 0>;
defm "" : LMULReadAdvance<"ReadVFSlideF", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherVV_data", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherVV_index", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherEI16VV_data", 0>;
defm "" : LMULSEWReadAdvance<"ReadVRGatherEI16VV_index", 0>;
defm "" : LMULReadAdvance<"ReadVRGatherVX_data", 0>;
defm "" : LMULReadAdvance<"ReadVRGatherVX_index", 0>;
defm "" : LMULReadAdvance<"ReadVRGatherVI_data", 0>;
defm "" : LMULSEWReadAdvance<"ReadVCompressV", 0>;
// LMUL Aware
def : ReadAdvance<ReadVMov1V, 0>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1015-1028: TableGen record mx / TableGen 记录 mx
```tablegen
def : ReadAdvance<ReadVMov2V, 0>;
def : ReadAdvance<ReadVMov4V, 0>;
def : ReadAdvance<ReadVMov8V, 0>;

// Others
def : ReadAdvance<ReadVMask, 0>;
def : ReadAdvance<ReadVPassthru_WorstCase, 0>;
foreach mx = SchedMxList in {
  def : ReadAdvance<!cast<SchedRead>("ReadVPassthru_" # mx), 0>;
  foreach sew = SchedSEWSet<mx>.val in
    def : ReadAdvance<!cast<SchedRead>("ReadVPassthru_" # mx  # "_E" # sew), 0>;
}

//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1029-1039: Bulk record instantiation / 批量记录实例化
```tablegen
// Unsupported extensions
defm : UnsupportedSchedQ;
defm : UnsupportedSchedZabha;
defm : UnsupportedSchedZbc;
defm : UnsupportedSchedZbkb;
defm : UnsupportedSchedZbkx;
defm : UnsupportedSchedZfa;
defm : UnsupportedSchedZvk;
defm : UnsupportedSchedSFB;
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
