# AArch64SchedKryo.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64SchedKryo.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This file defines the machine model for Qualcomm Kryo to support instruction scheduling and other instruction cost heuristics. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的调度模型。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Documented TableGen section
```tablegen
//==- AArch64SchedKryo.td - Qualcomm Kryo Scheduling Defs ---*- tablegen -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the machine model for Qualcomm Kryo to support
// instruction scheduling and other instruction cost heuristics.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// The issue width is set to five, matching the five issue queues for expanded
// uops. Now, the latency spreadsheet has information based on fragmented uops,
// but these do not actually take up an issue queue.

def KryoModel : SchedMachineModel {
  let IssueWidth        =   5; // 5-wide issue for expanded uops
  let MicroOpBufferSize = 128; // Out-of-order with temporary unified issue buffer
  let LoadLatency       =   4; // Optimistic load latency
  let MispredictPenalty =  14; // Fetch + Decode/Rename/Dispatch + Branch
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 24-39: Scoped TableGen attributes
```tablegen

  // Enable partial & runtime unrolling. The magic number is chosen based on
  // experiments and benchmarking data.
  let LoopMicroOpBufferSize = 16;
  let CompleteModel = 1;

  list<Predicate> UnsupportedFeatures = !listconcat(SVEUnsupported.F,
                                                    PAUnsupported.F,
                                                    SMEUnsupported.F,
                                                    [HasMTE, HasCSSC, HasMOPS, HasMOPS_GO]);
  // FIXME: Remove when all errors have been fixed.
  let FullInstRWOverlapCheck = 0;
}

//===----------------------------------------------------------------------===//
// Define each kind of processor resource and number available on Kryo.
```
**EN:** This let block applies shared attributes to a region of TableGen records so policy changes stay centralized.  
**CN:** 该 let 代码块把共享属性施加到一片 TableGen 记录上，使策略调整保持集中化。
### Lines 40-64: TableGen definition KryoUnitXA
```tablegen

let SchedModel = KryoModel in {
  def KryoUnitXA : ProcResource<1>;                   // Type X(A) micro-ops
  def KryoUnitXB : ProcResource<1>;                   // Type X(B) micro-ops
  def KryoUnitYA : ProcResource<1>;                   // Type Y(A) micro-ops
  def KryoUnitYB : ProcResource<1>;                   // Type Y(B) micro-ops
  def KryoUnitX : ProcResGroup<[KryoUnitXA,          // Type X micro-ops
                                KryoUnitXB]>;
  def KryoUnitY : ProcResGroup<[KryoUnitYA,          // Type Y micro-ops
                                KryoUnitYB]>;
  def KryoUnitXY : ProcResGroup<[KryoUnitXA,         // Type XY micro-ops
                                 KryoUnitXB,
                                 KryoUnitYA,
                                 KryoUnitYB]>;
  def KryoUnitLSA : ProcResource<1>;                  // Type LS(A) micro-ops
  def KryoUnitLSB : ProcResource<1>;                  // Type LS(B) micro-ops
  def KryoUnitLS : ProcResGroup<[KryoUnitLSA,        // Type LS micro-ops
                                 KryoUnitLSB]>;
}

let SchedModel = KryoModel in {

//===----------------------------------------------------------------------===//
// Map the target-defined scheduler read/write resources and latency for
// Kryo.
```
**EN:** This definition materializes KryoUnitXA as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 KryoUnitXA 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 65-91: TableGen definition
```tablegen

def : WriteRes<WriteImm,   [KryoUnitXY]> { let Latency = 1; }
def : WriteRes<WriteI,     [KryoUnitXY]> { let Latency = 1; }
def : WriteRes<WriteISReg, [KryoUnitXY, KryoUnitXY]>
      { let Latency = 2; let NumMicroOps = 2; }
def : WriteRes<WriteIEReg, [KryoUnitXY, KryoUnitXY]>
      { let Latency = 2; let NumMicroOps = 2; }
def : WriteRes<WriteExtr,  [KryoUnitXY, KryoUnitX]>
      { let Latency = 2; let NumMicroOps = 2; }
def : WriteRes<WriteIS,    [KryoUnitXY]> { let Latency = 2; }
def : WriteRes<WriteID32,  [KryoUnitXA, KryoUnitY]>
      { let Latency = 8; let NumMicroOps = 1; } // Fragent -1
def : WriteRes<WriteID64,  [KryoUnitXA, KryoUnitY]>
      { let Latency = 8; let NumMicroOps = 1; } // Fragent -1
def : WriteRes<WriteIM32,  [KryoUnitX]> { let Latency = 5; }
def : WriteRes<WriteIM64,  [KryoUnitX]> { let Latency = 5; }
def : WriteRes<WriteBr,    [KryoUnitXY]> { let Latency = 1; }
def : WriteRes<WriteBrReg, [KryoUnitXY]> { let Latency = 1; }
def : WriteRes<WriteLD,    [KryoUnitLS]> { let Latency = 4; }
def : WriteRes<WriteST,    [KryoUnitLS]> { let Latency = 4; }
def : WriteRes<WriteSTP,   [KryoUnitLS]> { let Latency = 4; }
def : WriteRes<WriteAdr,   [KryoUnitXY]> { let Latency = 6; }
def : WriteRes<WriteLDIdx, [KryoUnitLS]> { let Latency = 4; }
def : WriteRes<WriteSTIdx, [KryoUnitLS]> { let Latency = 4; }
def : WriteRes<WriteF,     [KryoUnitXY, KryoUnitXY]>
      { let Latency = 3; let NumMicroOps = 2; }
def : WriteRes<WriteFCmp,  [KryoUnitXY]> { let Latency = 2; }
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 92-110: TableGen definition
```tablegen
def : WriteRes<WriteFCvt,  [KryoUnitX]> { let Latency = 4; }
def : WriteRes<WriteFCopy, [KryoUnitXY]> { let Latency = 6; }
def : WriteRes<WriteFImm,  [KryoUnitXY]> { let Latency = 6; }
def : WriteRes<WriteFMul,  [KryoUnitX, KryoUnitX]>
      { let Latency = 6; let NumMicroOps = 2; }
def : WriteRes<WriteFDiv,  [KryoUnitXA, KryoUnitY]>
      { let Latency = 12; let NumMicroOps = 2; } // Fragent -1 / NoRSV +1
def : WriteRes<WriteVd,    [KryoUnitXY]> { let Latency = 6; }
def : WriteRes<WriteVq,    [KryoUnitXY]> { let Latency = 6; }
def : WriteRes<WriteVLD,   [KryoUnitLS]> { let Latency = 4; }
def : WriteRes<WriteVST,   [KryoUnitLS]> { let Latency = 4; }

def : WriteRes<WriteSys,     []> { let Latency = 1; }
def : WriteRes<WriteBarrier, []> { let Latency = 1; }
def : WriteRes<WriteHint,    []> { let Latency = 1; }

def : WriteRes<WriteLDHi,    []> { let Latency = 4; }

def : WriteRes<WriteAtomic, []> { let Unsupported = 1; }
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 111-133: TableGen definition
```tablegen

// No forwarding logic is modelled yet.
def : ReadAdvance<ReadI,       0>;
def : ReadAdvance<ReadISReg,   0>;
def : ReadAdvance<ReadIEReg,   0>;
def : ReadAdvance<ReadIM,      0>;
def : ReadAdvance<ReadIMA,     0>;
def : ReadAdvance<ReadID,      0>;
def : ReadAdvance<ReadExtrHi,  0>;
def : ReadAdvance<ReadAdrBase, 0>;
def : ReadAdvance<ReadVLD,     0>;
def : ReadAdvance<ReadST,      0>;


//===----------------------------------------------------------------------===//
// Specialize the coarse model by associating instruction groups with the
// subtarget-defined types. As the modeled is refined, this will override most
// of the above SchedWriteRes and SchedAlias mappings.

// Miscellaneous
// -----------------------------------------------------------------------------

def : InstRW<[WriteI], (instrs COPY)>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 134-141: TableGen includes and shared records
```tablegen


// Detailed Refinedments
// -----------------------------------------------------------------------------
include "AArch64SchedKryoDetails.td"


} // SchedModel = KryoModel
```
**EN:** This TableGen block imports shared records so the file can build larger AArch64 descriptions on top of them.  
**CN:** 该 TableGen 代码块导入共享记录，使本文件能够在其之上继续构建更完整的 AArch64 描述。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Per-core scheduling itineraries and latency models **CN:** 面向具体内核的调度行程与时延模型
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** TableGen includes: AArch64SchedKryoDetails.td **CN:** TableGen 包含文件：AArch64SchedKryoDetails.td
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for scheduling models. **CN:** 与周边负责调度模型的 AArch64 后端组件紧密协作。
