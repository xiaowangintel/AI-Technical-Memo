# AArch64SchedFalkor.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64SchedFalkor.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This file defines the machine model for Qualcomm Falkor to support instruction scheduling and other instruction cost heuristics. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的调度模型。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Documented TableGen section
```tablegen
//==- AArch64SchedFalkor.td - Falkor Scheduling Definitions -*- tablegen -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the machine model for Qualcomm Falkor to support
// instruction scheduling and other instruction cost heuristics.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Define the SchedMachineModel and provide basic properties for coarse grained
// instruction cost model.

def FalkorModel : SchedMachineModel {
  let IssueWidth = 8;          // 8 uops are dispatched per cycle.
  let MicroOpBufferSize = 128; // Out-of-order with temporary unified issue buffer.
  let LoopMicroOpBufferSize = 16;
  let LoadLatency = 3;         // Optimistic load latency.
  let MispredictPenalty = 11;  // Minimum branch misprediction penalty.
  let CompleteModel = 1;
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 25-49: TableGen backend rules
```tablegen

  list<Predicate> UnsupportedFeatures = !listconcat(SVEUnsupported.F,
                                                    PAUnsupported.F,
                                                    SMEUnsupported.F,
                                                    [HasMTE, HasCSSC, HasMOPS, HasMOPS_GO]);
  // FIXME: Remove when all errors have been fixed.
  let FullInstRWOverlapCheck = 0;
}

//===----------------------------------------------------------------------===//
// Define each kind of processor resource and number available on Falkor.

let SchedModel = FalkorModel in {

  def FalkorUnitB    : ProcResource<1>; // Branch
  def FalkorUnitLD   : ProcResource<1>; // Load pipe
  def FalkorUnitSD   : ProcResource<1>; // Store data
  def FalkorUnitST   : ProcResource<1>; // Store pipe
  def FalkorUnitX    : ProcResource<1>; // Complex arithmetic
  def FalkorUnitY    : ProcResource<1>; // Simple arithmetic
  def FalkorUnitZ    : ProcResource<1>; // Simple arithmetic

  def FalkorUnitVSD  : ProcResource<1>; // Vector store data
  def FalkorUnitVX   : ProcResource<1>; // Vector X-pipe
  def FalkorUnitVY   : ProcResource<1>; // Vector Y-pipe
```
**EN:** This TableGen block continues the file's scheduling models work and contributes declarative data for LLVM's code generators.  
**CN:** 该 TableGen 代码块延续本文件的调度模型工作，并为 LLVM 代码生成器提供声明式数据。
### Lines 50-68: TableGen definition FalkorUnitGTOV
```tablegen

  def FalkorUnitGTOV : ProcResource<1>; // Scalar to Vector
  def FalkorUnitVTOG : ProcResource<1>; // Vector to Scalar

  // Define the resource groups.
  def FalkorUnitXY   : ProcResGroup<[FalkorUnitX, FalkorUnitY]>;
  def FalkorUnitXYZ  : ProcResGroup<[FalkorUnitX, FalkorUnitY, FalkorUnitZ]>;
  def FalkorUnitXYZB : ProcResGroup<[FalkorUnitX, FalkorUnitY, FalkorUnitZ,
                                     FalkorUnitB]>;
  def FalkorUnitZB   : ProcResGroup<[FalkorUnitZ, FalkorUnitB]>;
  def FalkorUnitVXVY : ProcResGroup<[FalkorUnitVX, FalkorUnitVY]>;

}

//===----------------------------------------------------------------------===//
// Map the target-defined scheduler read/write resources and latency for
// Falkor.

let SchedModel = FalkorModel in {
```
**EN:** This definition materializes FalkorUnitGTOV as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 FalkorUnitGTOV 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 69-95: TableGen definition
```tablegen

// These WriteRes entries are not used in the Falkor sched model.
def : WriteRes<WriteImm, []>     { let Unsupported = 1; }
def : WriteRes<WriteI, []>       { let Unsupported = 1; }
def : WriteRes<WriteISReg, []>   { let Unsupported = 1; }
def : WriteRes<WriteIEReg, []>   { let Unsupported = 1; }
def : WriteRes<WriteExtr, []>    { let Unsupported = 1; }
def : WriteRes<WriteIS, []>      { let Unsupported = 1; }
def : WriteRes<WriteID32, []>    { let Unsupported = 1; }
def : WriteRes<WriteID64, []>    { let Unsupported = 1; }
def : WriteRes<WriteIM32, []>    { let Unsupported = 1; }
def : WriteRes<WriteIM64, []>    { let Unsupported = 1; }
def : WriteRes<WriteBr, []>      { let Unsupported = 1; }
def : WriteRes<WriteBrReg, []>   { let Unsupported = 1; }
def : WriteRes<WriteLD, []>      { let Unsupported = 1; }
def : WriteRes<WriteST, []>      { let Unsupported = 1; }
def : WriteRes<WriteSTP, []>     { let Unsupported = 1; }
def : WriteRes<WriteAdr, []>     { let Unsupported = 1; }
def : WriteRes<WriteLDIdx, []>   { let Unsupported = 1; }
def : WriteRes<WriteSTIdx, []>   { let Unsupported = 1; }
def : WriteRes<WriteF, []>       { let Unsupported = 1; }
def : WriteRes<WriteFCmp, []>    { let Unsupported = 1; }
def : WriteRes<WriteFCvt, []>    { let Unsupported = 1; }
def : WriteRes<WriteFCopy, []>   { let Unsupported = 1; }
def : WriteRes<WriteFImm, []>    { let Unsupported = 1; }
def : WriteRes<WriteFMul, []>    { let Unsupported = 1; }
def : WriteRes<WriteFDiv, []>    { let Unsupported = 1; }
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 96-122: TableGen definition
```tablegen
def : WriteRes<WriteVd, []>      { let Unsupported = 1; }
def : WriteRes<WriteVq, []>      { let Unsupported = 1; }
def : WriteRes<WriteVLD, []>     { let Unsupported = 1; }
def : WriteRes<WriteVST, []>     { let Unsupported = 1; }
def : WriteRes<WriteSys, []>     { let Unsupported = 1; }
def : WriteRes<WriteBarrier, []> { let Unsupported = 1; }
def : WriteRes<WriteHint, []>    { let Unsupported = 1; }
def : WriteRes<WriteLDHi, []>    { let Unsupported = 1; }
def : WriteRes<WriteAtomic, []>  { let Unsupported = 1; }

// These ReadAdvance entries are not used in the Falkor sched model.
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

// Detailed Refinements
// -----------------------------------------------------------------------------
include "AArch64SchedFalkorDetails.td"

}
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Per-core scheduling itineraries and latency models **CN:** 面向具体内核的调度行程与时延模型
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** TableGen includes: AArch64SchedFalkorDetails.td **CN:** TableGen 包含文件：AArch64SchedFalkorDetails.td
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for scheduling models. **CN:** 与周边负责调度模型的 AArch64 后端组件紧密协作。
