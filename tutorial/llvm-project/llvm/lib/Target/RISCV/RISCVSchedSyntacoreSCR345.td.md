# RISCVSchedSyntacoreSCR345.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedSyntacoreSCR345.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Commentary and design intent / 注释与设计意图
```tablegen
//==- RISCVSchedSyntacoreSCR345.td - SCR3/4/5 Sched Defs -----*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//

// This file covers scheduling models for Syntacore SCR3, SCR4 and SCR5
// processors.
// Configurations:
// * SCR3 rv32imc and rv64imac, overview https://syntacore.com/products/scr3
// * SCR4 rv32imfdc and rv64imafdc, overview https://syntacore.com/products/scr4
// * SCR5 rv32imafdc and rv64imafdc, overview
//   https://syntacore.com/products/scr5
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 19-36: Reusable TableGen multiclass SyntacoreSchedModel / 可复用的 TableGen 多类 SyntacoreSchedModel
```tablegen
// SCR3-5 are single-issue in-order processors
class SyntacoreSchedModel : SchedMachineModel {
  let MicroOpBufferSize = 0;
  let IssueWidth = 1;
  let MispredictPenalty = 3;
  let CompleteModel = 0;
  let UnsupportedFeatures = [HasStdExtZbkb, HasStdExtZbkc, HasStdExtZbkx,
                             HasStdExtZknd, HasStdExtZkne, HasStdExtZknh,
                             HasStdExtZksed, HasStdExtZksh, HasStdExtZkr,
                             HasVInstructions];
}

// Branching
multiclass SCR_Branching<ProcResourceKind BRU> {
  def : WriteRes<WriteJmp, [BRU]>;
  def : WriteRes<WriteJal, [BRU]>;
  def : WriteRes<WriteJalr, [BRU]>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 37-53: Reusable TableGen multiclass SCR_IntALU / 可复用的 TableGen 多类 SCR_IntALU
```tablegen

// Single-cycle integer arithmetic and logic
multiclass SCR_IntALU<ProcResourceKind ALU> {
  def : WriteRes<WriteIALU, [ALU]>;
  def : WriteRes<WriteIALU32, [ALU]>;
  def : WriteRes<WriteShiftImm, [ALU]>;
  def : WriteRes<WriteShiftImm32, [ALU]>;
  def : WriteRes<WriteShiftReg, [ALU]>;
  def : WriteRes<WriteShiftReg32, [ALU]>;
}

// Integer multiplication
multiclass SCR_IntMul<ProcResourceKind MUL> {
  let Latency = 2 in {
    def : WriteRes<WriteIMul, [MUL]>;
    def : WriteRes<WriteIMul32, [MUL]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 54-77: Reusable TableGen multiclass SCR_IntDiv / 可复用的 TableGen 多类 SCR_IntDiv
```tablegen
}

// Integer division
multiclass SCR_IntDiv<ProcResourceKind DIV, int DivLatency> {
  let Latency = DivLatency, ReleaseAtCycles = [DivLatency] in {
    def : WriteRes<WriteIDiv, [DIV]>;
    def : WriteRes<WriteIDiv32, [DIV]>;
    def : WriteRes<WriteIRem, [DIV]>;
    def : WriteRes<WriteIRem32, [DIV]>;
  }
}

// Load/store instructions
multiclass SCR_BasicMemory<ProcResourceKind LSU, int LoadStoreLatency> {
  let Latency = LoadStoreLatency in {
    def : WriteRes<WriteSTB, [LSU]>;
    def : WriteRes<WriteSTH, [LSU]>;
    def : WriteRes<WriteSTW, [LSU]>;
    def : WriteRes<WriteSTD, [LSU]>;
    def : WriteRes<WriteLDB, [LSU]>;
    def : WriteRes<WriteLDH, [LSU]>;
    def : WriteRes<WriteLDW, [LSU]>;
    def : WriteRes<WriteLDD, [LSU]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 78-99: Reusable TableGen multiclass SCR_FPMemory / 可复用的 TableGen 多类 SCR_FPMemory
```tablegen
}

// Floating-point load/store instructions
multiclass SCR_FPMemory<ProcResourceKind LSU, int FPLoadStoreLatency> {
  let Latency = FPLoadStoreLatency in {
    def : WriteRes<WriteFST32, [LSU]>;
    def : WriteRes<WriteFST64, [LSU]>;
    def : WriteRes<WriteFLD32, [LSU]>;
    def : WriteRes<WriteFLD64, [LSU]>;
  }
}

// Atomic memory
multiclass SCR_AtomicMemory<ProcResourceKind LSU> {
  let Latency = 20 in {
    def : WriteRes<WriteAtomicLDW, [LSU]>;
    def : WriteRes<WriteAtomicLDD, [LSU]>;
    def : WriteRes<WriteAtomicW, [LSU]>;
    def : WriteRes<WriteAtomicD, [LSU]>;
    def : WriteRes<WriteAtomicSTW, [LSU]>;
    def : WriteRes<WriteAtomicSTD, [LSU]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 100-115: Reusable TableGen multiclass SCR_FPU / 可复用的 TableGen 多类 SCR_FPU
```tablegen
}

// Floating-point unit (without division and SQRT)
multiclass SCR_FPU<ProcResourceKind FPU> {
  // Single and double-precision computational instructions
  def : WriteRes<WriteFAdd32, [FPU]> { let Latency = 3; }
  def : WriteRes<WriteFAdd64, [FPU]> { let Latency = 3; }
  def : WriteRes<WriteFMul32, [FPU]> { let Latency = 4; }
  def : WriteRes<WriteFMul64, [FPU]> { let Latency = 4; }
  def : WriteRes<WriteFMA32, [FPU]> { let Latency = 4; }
  def : WriteRes<WriteFMA64, [FPU]> { let Latency = 4; }
  def : WriteRes<WriteFSGNJ32, [FPU]> { let Latency = 2; }
  def : WriteRes<WriteFSGNJ64, [FPU]> { let Latency = 2; }
  def : WriteRes<WriteFMinMax32, [FPU]> { let Latency = 2; }
  def : WriteRes<WriteFMinMax64, [FPU]> { let Latency = 2; }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 116-131: TableGen record definition / TableGen 记录定义
```tablegen
  // Conversion and move instructions
  let Latency = 3 in {
    def : WriteRes<WriteFCvtI32ToF32, [FPU]>;
    def : WriteRes<WriteFCvtI32ToF64, [FPU]>;
    def : WriteRes<WriteFCvtI64ToF32, [FPU]>;
    def : WriteRes<WriteFCvtI64ToF64, [FPU]>;
    def : WriteRes<WriteFCvtF32ToF64, [FPU]>;
    def : WriteRes<WriteFCvtF64ToF32, [FPU]>;
  }

  let Latency = 2 in {
    def : WriteRes<WriteFCvtF32ToI32, [FPU]>;
    def : WriteRes<WriteFCvtF64ToI32, [FPU]>;
    def : WriteRes<WriteFCvtF32ToI64, [FPU]>;
    def : WriteRes<WriteFCvtF64ToI64, [FPU]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 132-149: TableGen record definition / TableGen 记录定义
```tablegen

  let Latency = 2 in {
    def : WriteRes<WriteFMovI32ToF32, [FPU]>;
    def : WriteRes<WriteFMovF32ToI32, [FPU]>;
    def : WriteRes<WriteFMovI64ToF64, [FPU]>;
    def : WriteRes<WriteFMovF64ToI64, [FPU]>;
  }

  let Latency = 2 in {
    def : WriteRes<WriteFClass32, [FPU]>;
    def : WriteRes<WriteFClass64, [FPU]>;
  }

  // Comparisons
  let Latency = 2 in {
    def : WriteRes<WriteFCmp32, [FPU]>;
    def : WriteRes<WriteFCmp64, [FPU]>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 150-166: Reusable TableGen multiclass SCR_FDU / 可复用的 TableGen 多类 SCR_FDU
```tablegen
}

// FP division and SQRT is not pipelined
multiclass SCR_FDU<ProcResourceKind FDU> {
  def : WriteRes<WriteFDiv32, [FDU]> {
    let Latency = 10;
    let ReleaseAtCycles = [8];
  }
  def : WriteRes<WriteFDiv64, [FDU]> {
    let Latency = 17;
    let ReleaseAtCycles = [15];
  }

  def : WriteRes<WriteFSqrt32, [FDU]> {
    let Latency = 19;
    let ReleaseAtCycles = [19];
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 167-180: Reusable TableGen multiclass SCR_Other / 可复用的 TableGen 多类 SCR_Other
```tablegen
  def : WriteRes<WriteFSqrt64, [FDU]> {
    let Latency = 33;
    let ReleaseAtCycles = [33];
  }
}

// Others
multiclass SCR_Other {
  def : WriteRes<WriteCSR, []>;
  def : WriteRes<WriteNop, []>;

  def : InstRW<[WriteIALU], (instrs COPY)>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 181-195: Reusable TableGen multiclass SCR_Unsupported / 可复用的 TableGen 多类 SCR_Unsupported
```tablegen
// Unsupported scheduling classes for SCR3-5.
multiclass SCR_Unsupported :
  UnsupportedSchedSFB,
  UnsupportedSchedV,
  UnsupportedSchedZabha,
  UnsupportedSchedZba,
  UnsupportedSchedZbb,
  UnsupportedSchedZbc,
  UnsupportedSchedZbs,
  UnsupportedSchedZbkb,
  UnsupportedSchedZbkx,
  UnsupportedSchedZfa,
  UnsupportedSchedZvk,
  UnsupportedSchedXsf;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 196-223: Reusable TableGen multiclass SCR3_Unsupported / 可复用的 TableGen 多类 SCR3_Unsupported
```tablegen
multiclass SCR3_Unsupported :
  SCR_Unsupported,
  UnsupportedSchedF;

multiclass SCR4_SCR5_Unsupported :
  SCR_Unsupported,
  UnsupportedSchedQ,
  UnsupportedSchedZfhmin;

// Bypasses (none)
multiclass SCR_NoReadAdvances {
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
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 224-251: Reusable TableGen multiclass SCR4_SCR5_NoReadAdvances / 可复用的 TableGen 多类 SCR4_SCR5_NoReadAdvances
```tablegen
  def : ReadAdvance<ReadAtomicWA, 0>;
  def : ReadAdvance<ReadAtomicWD, 0>;
  def : ReadAdvance<ReadAtomicDA, 0>;
  def : ReadAdvance<ReadAtomicDD, 0>;
  def : ReadAdvance<ReadAtomicLDW, 0>;
  def : ReadAdvance<ReadAtomicLDD, 0>;
  def : ReadAdvance<ReadAtomicSTW, 0>;
  def : ReadAdvance<ReadAtomicSTD, 0>;
}

// Floating-point bypasses (none)
multiclass SCR4_SCR5_NoReadAdvances : SCR_NoReadAdvances {
  def : ReadAdvance<ReadFStoreData, 0>;
  def : ReadAdvance<ReadFMemBase, 0>;
  def : ReadAdvance<ReadFAdd32, 0>;
  def : ReadAdvance<ReadFAdd64, 0>;
  def : ReadAdvance<ReadFMul32, 0>;
  def : ReadAdvance<ReadFMul64, 0>;
  def : ReadAdvance<ReadFMA32, 0>;
  def : ReadAdvance<ReadFMA32Addend, 0>;
  def : ReadAdvance<ReadFMA64, 0>;
  def : ReadAdvance<ReadFMA64Addend, 0>;
  def : ReadAdvance<ReadFDiv32, 0>;
  def : ReadAdvance<ReadFDiv64, 0>;
  def : ReadAdvance<ReadFSqrt32, 0>;
  def : ReadAdvance<ReadFSqrt64, 0>;
  def : ReadAdvance<ReadFCmp32, 0>;
  def : ReadAdvance<ReadFCmp64, 0>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 252-272: TableGen record definition / TableGen 记录定义
```tablegen
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
  def : ReadAdvance<ReadFMovF32ToI32, 0>;
  def : ReadAdvance<ReadFMovI32ToF32, 0>;
  def : ReadAdvance<ReadFMovF64ToI64, 0>;
  def : ReadAdvance<ReadFMovI64ToF64, 0>;
  def : ReadAdvance<ReadFClass32, 0>;
  def : ReadAdvance<ReadFClass64, 0>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 273-288: TableGen record SyntacoreSCR3RV32Model / TableGen 记录 SyntacoreSCR3RV32Model
```tablegen

//===----------------------------------------------------------------------===//
// SCR3 scheduling model definition

def SyntacoreSCR3RV32Model : SyntacoreSchedModel {
  let LoadLatency = 2;
}

let SchedModel = SyntacoreSCR3RV32Model in {
  let BufferSize = 0 in {
    def SCR3RV32_ALU : ProcResource<1>;
    def SCR3RV32_MUL : ProcResource<1>;
    def SCR3RV32_DIV : ProcResource<1>;
    def SCR3RV32_LSU : ProcResource<1>;
    def SCR3RV32_CFU : ProcResource<1>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 289-304: Bulk record instantiation SyntacoreSCR3RV64Model / 批量记录实例化 SyntacoreSCR3RV64Model
```tablegen

  defm : SCR_Branching<SCR3RV32_CFU>;
  defm : SCR_IntALU<SCR3RV32_ALU>;
  defm : SCR_IntMul<SCR3RV32_MUL>;
  defm : SCR_IntDiv<SCR3RV32_DIV, /* div latency = */ 8>;
  defm : SCR_BasicMemory<SCR3RV32_LSU, /* load & store latency = */ 2>;
  defm : SCR_AtomicMemory<SCR3RV32_LSU>;
  defm : SCR_Other;

  defm : SCR3_Unsupported;
  defm : SCR_NoReadAdvances;
}

def SyntacoreSCR3RV64Model : SyntacoreSchedModel {
  let LoadLatency = 2;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 305-322: Bulk record instantiation SCR3RV64_ALU / 批量记录实例化 SCR3RV64_ALU
```tablegen

let SchedModel = SyntacoreSCR3RV64Model in {
  let BufferSize = 0 in {
    def SCR3RV64_ALU : ProcResource<1>;
    def SCR3RV64_MUL : ProcResource<1>;
    def SCR3RV64_DIV : ProcResource<1>;
    def SCR3RV64_LSU : ProcResource<1>;
    def SCR3RV64_CFU : ProcResource<1>;
  }

  defm : SCR_Branching<SCR3RV64_CFU>;
  defm : SCR_IntALU<SCR3RV64_ALU>;
  defm : SCR_IntMul<SCR3RV64_MUL>;
  defm : SCR_IntDiv<SCR3RV64_DIV, /* div latency = */ 11>;
  defm : SCR_BasicMemory<SCR3RV64_LSU, /* load & store latency = */ 2>;
  defm : SCR_AtomicMemory<SCR3RV64_LSU>;
  defm : SCR_Other;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 323-343: Bulk record instantiation SyntacoreSCR4RV32Model / 批量记录实例化 SyntacoreSCR4RV32Model
```tablegen
  defm : SCR3_Unsupported;
  defm : SCR_NoReadAdvances;
}

//===----------------------------------------------------------------------===//
// SCR4 scheduling model definition

def SyntacoreSCR4RV32Model : SyntacoreSchedModel {
  let LoadLatency = 2;
}

let SchedModel = SyntacoreSCR4RV32Model in {
  let BufferSize = 0 in {
    def SCR4RV32_ALU : ProcResource<1>;
    def SCR4RV32_MUL : ProcResource<1>;
    def SCR4RV32_DIV : ProcResource<1>;
    def SCR4RV32_LSU : ProcResource<1>;
    def SCR4RV32_CFU : ProcResource<1>;
    def SCR4RV32_FPU : ProcResource<1>;
    def SCR4RV32_FDU : ProcResource<1>; // FP div and sqrt resource
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 344-358: Bulk record instantiation / 批量记录实例化
```tablegen

  defm : SCR_Branching<SCR4RV32_CFU>;
  defm : SCR_IntALU<SCR4RV32_ALU>;
  defm : SCR_IntMul<SCR4RV32_MUL>;
  defm : SCR_IntDiv<SCR4RV32_DIV, /* div latency = */ 8>;
  defm : SCR_BasicMemory<SCR4RV32_LSU, /* load & store latency = */ 2>;
  defm : SCR_FPMemory<SCR4RV32_LSU, /* load & store latency = */ 2>;
  defm : SCR_AtomicMemory<SCR4RV32_LSU>;
  defm : SCR_FPU<SCR4RV32_FPU>;
  defm : SCR_FDU<SCR4RV32_FDU>;
  defm : SCR_Other;

  defm : SCR4_SCR5_Unsupported;
  defm : SCR4_SCR5_NoReadAdvances;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 359-373: TableGen record SyntacoreSCR4RV64Model / TableGen 记录 SyntacoreSCR4RV64Model
```tablegen

def SyntacoreSCR4RV64Model : SyntacoreSchedModel {
  let LoadLatency = 2;
}

let SchedModel = SyntacoreSCR4RV64Model in {
  let BufferSize = 0 in {
    def SCR4RV64_ALU : ProcResource<1>;
    def SCR4RV64_MUL : ProcResource<1>;
    def SCR4RV64_DIV : ProcResource<1>;
    def SCR4RV64_LSU : ProcResource<1>;
    def SCR4RV64_CFU : ProcResource<1>;
    def SCR4RV64_FPU : ProcResource<1>;
    def SCR4RV64_FDU : ProcResource<1>; // FP div and sqrt resource
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 374-388: Bulk record instantiation / 批量记录实例化
```tablegen

  defm : SCR_Branching<SCR4RV64_CFU>;
  defm : SCR_IntALU<SCR4RV64_ALU>;
  defm : SCR_IntMul<SCR4RV64_MUL>;
  defm : SCR_IntDiv<SCR4RV64_DIV, /* div latency = */ 11>;
  defm : SCR_BasicMemory<SCR4RV64_LSU, /* load & store latency = */ 2>;
  defm : SCR_FPMemory<SCR4RV64_LSU, /* load & store latency = */ 2>;
  defm : SCR_AtomicMemory<SCR4RV64_LSU>;
  defm : SCR_FPU<SCR4RV64_FPU>;
  defm : SCR_FDU<SCR4RV64_FDU>;
  defm : SCR_Other;

  defm : SCR4_SCR5_Unsupported;
  defm : SCR4_SCR5_NoReadAdvances;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 389-406: TableGen record SyntacoreSCR5RV32Model / TableGen 记录 SyntacoreSCR5RV32Model
```tablegen

//===----------------------------------------------------------------------===//
// SCR5 scheduling model definition

def SyntacoreSCR5RV32Model : SyntacoreSchedModel {
  let LoadLatency = 3;
}

let SchedModel = SyntacoreSCR5RV32Model in {
  let BufferSize = 0 in {
    def SCR5RV32_ALU : ProcResource<1>;
    def SCR5RV32_MUL : ProcResource<1>;
    def SCR5RV32_DIV : ProcResource<1>;
    def SCR5RV32_LSU : ProcResource<1>;
    def SCR5RV32_CFU : ProcResource<1>;
    def SCR5RV32_FPU : ProcResource<1>;
    def SCR5RV32_FDU : ProcResource<1>; // FP div and sqrt resource
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 407-421: Bulk record instantiation / 批量记录实例化
```tablegen

  defm : SCR_Branching<SCR5RV32_CFU>;
  defm : SCR_IntALU<SCR5RV32_ALU>;
  defm : SCR_IntMul<SCR5RV32_MUL>;
  defm : SCR_IntDiv<SCR5RV32_DIV, /* div latency = */ 8>;
  defm : SCR_BasicMemory<SCR5RV32_LSU, /* load & store latency = */ 3>;
  defm : SCR_FPMemory<SCR5RV32_LSU, /* load & store latency = */ 3>;
  defm : SCR_AtomicMemory<SCR5RV32_LSU>;
  defm : SCR_FPU<SCR5RV32_FPU>;
  defm : SCR_FDU<SCR5RV32_FDU>;
  defm : SCR_Other;

  defm : SCR4_SCR5_Unsupported;
  defm : SCR4_SCR5_NoReadAdvances;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 422-436: TableGen record SyntacoreSCR5RV64Model / TableGen 记录 SyntacoreSCR5RV64Model
```tablegen

def SyntacoreSCR5RV64Model : SyntacoreSchedModel {
  let LoadLatency = 3;
}

let SchedModel = SyntacoreSCR5RV64Model in {
  let BufferSize = 0 in {
    def SCR5RV64_ALU : ProcResource<1>;
    def SCR5RV64_MUL : ProcResource<1>;
    def SCR5RV64_DIV : ProcResource<1>;
    def SCR5RV64_LSU : ProcResource<1>;
    def SCR5RV64_CFU : ProcResource<1>;
    def SCR5RV64_FPU : ProcResource<1>;
    def SCR5RV64_FDU : ProcResource<1>; // FP div and sqrt resource
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 437-451: Bulk record instantiation / 批量记录实例化
```tablegen

  defm : SCR_Branching<SCR5RV64_CFU>;
  defm : SCR_IntALU<SCR5RV64_ALU>;
  defm : SCR_IntMul<SCR5RV64_MUL>;
  defm : SCR_IntDiv<SCR5RV64_DIV, /* div latency = */ 11>;
  defm : SCR_BasicMemory<SCR5RV64_LSU, /* load & store latency = */ 3>;
  defm : SCR_FPMemory<SCR5RV64_LSU, /* load & store latency = */ 3>;
  defm : SCR_AtomicMemory<SCR5RV64_LSU>;
  defm : SCR_FPU<SCR5RV64_FPU>;
  defm : SCR_FDU<SCR5RV64_FDU>;
  defm : SCR_Other;

  defm : SCR4_SCR5_Unsupported;
  defm : SCR4_SCR5_NoReadAdvances;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
