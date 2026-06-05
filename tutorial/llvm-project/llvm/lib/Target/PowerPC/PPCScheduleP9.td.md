# PPCScheduleP9.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCScheduleP9.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCScheduleP9.td - PPC P9 Scheduling Definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCScheduleP9.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCScheduleP9.td - PPC P9 Scheduling Definitions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段包含调度或处理器模型元数据。

### Lines 7-10

```tablegen
//===----------------------------------------------------------------------===//
//
// This file defines the itinerary class data for the POWER9 processor.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines the itinerary class data for the POWER9 processor.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines the itinerary class data for the POWER9 processor.”。

### Lines 11-23

```tablegen
//===----------------------------------------------------------------------===//
def P9Model : SchedMachineModel {
  // The maximum number of instructions to be issued at the same time.
  // While a value of 8 is technically correct since 8 instructions can be
  // fetched from the instruction cache. However, only 6 instructions may be
  // actually dispatched at a time.
  let IssueWidth = 8;

  // Load latency is 4 or 5 cycles depending on the load. This latency assumes
  // that we have a cache hit. For a cache miss the load latency will be more.
  // There are two instructions (lxvl, lxvll) that have a latency of 6 cycles.
  // However it is not worth bumping this value up to 6 when the vast majority
  // of instructions are 4 or 5 cycles.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The maximum number of instructions to be issued at the same time.". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The maximum number of instructions to be issued at the same time.”。 这一段包含调度或处理器模型元数据。

### Lines 24-30

```tablegen
  let LoadLatency = 5;

  // A total of 16 cycles to recover from a branch mispredict.
  let MispredictPenalty = 16;

  // Try to make sure we have at least 10 dispatch groups in a loop.
  // A dispatch group is 6 instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "A total of 16 cycles to recover from a branch mispredict.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“A total of 16 cycles to recover from a branch mispredict.”。

### Lines 31-38

```tablegen
  let LoopMicroOpBufferSize = 60;

  // As iops are dispatched to a slice, they are held in an independent slice
  // issue queue until all register sources and other dependencies have been
  // resolved and they can be issued. Each of four execution slices has an
  // 11-entry iop issue queue.
  let MicroOpBufferSize = 44;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "As iops are dispatched to a slice, they are held in an independent slice".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“As iops are dispatched to a slice, they are held in an independent slice”。

### Lines 39-49

```tablegen
  let CompleteModel = 1;

  // Do not support SPE (Signal Processing Engine) or prefixed instructions on
  // Power 9, or MMA, or paired vector mem ops,  or PC relative mem ops, or
  // instructions introduced after ISA 3.0.
  let UnsupportedFeatures = [HasSPE, PrefixInstrs, MMA,
                             PairedVectorMemops, IsBookE,
                             PCRelativeMemops, IsISA3_1, IsISAFuture,
                             HasFutureVector];
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Do not support SPE (Signal Processing Engine) or prefixed instructions on". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Do not support SPE (Signal Processing Engine) or prefixed instructions on”。 子目标特性裁剪会影响这里的行为。

### Lines 50-56

```tablegen
let SchedModel = P9Model in {

  // ***************** Processor Resources *****************

  // Dispatcher slots:
  // x0, x1, x2, and x3 are the dedicated slice dispatch ports, where each
  // corresponds to one of the four execution slices.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "***************** Processor Resources *****************". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“***************** Processor Resources *****************”。 这一段包含调度或处理器模型元数据。

### Lines 57-65

```tablegen
  def DISPx02 : ProcResource<2>;
  def DISPx13 : ProcResource<2>;
  // The xa and xb ports can be used to send an iop to either of the two slices
  // of the superslice, but are restricted to iops with only two primary sources.
  def DISPxab : ProcResource<2>;
  // b0 and b1 are dedicated dispatch ports into the branch slice.
  def DISPb01 : ProcResource<2>;

  // Any non BR dispatch ports
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The xa and xb ports can be used to send an iop to either of the two slices".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The xa and xb ports can be used to send an iop to either of the two slices”。

### Lines 66-76

```tablegen
  def DISP_NBR
      : ProcResGroup<[ DISPx02, DISPx13, DISPxab]>;
  def DISP_SS : ProcResGroup<[ DISPx02, DISPx13]>;

  // Issue Ports
  // An instruction can go down one of two issue queues.
  // Address Generation (AGEN) mainly for loads and stores.
  // Execution (EXEC) for most other instructions.
  // Some instructions cannot be run on just any issue queue and may require an
  // Even or an Odd queue. The EXECE represents the even queues and the EXECO
  // represents the odd queues.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Issue Ports".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Issue Ports”。

### Lines 77-84

```tablegen
  def IP_AGEN : ProcResource<4>;
  def IP_EXEC : ProcResource<4>;
  def IP_EXECE : ProcResource<2> {
    //Even Exec Ports
    let Super = IP_EXEC;
  }
  def IP_EXECO : ProcResource<2> {
    //Odd Exec Ports
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Even Exec Ports".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Even Exec Ports”。

### Lines 85-92

```tablegen
    let Super = IP_EXEC;
  }

  // Pipeline Groups
  // Four ALU (Fixed Point Arithmetic) units in total. Two even, two Odd.
  def ALU : ProcResource<4>;
  def ALUE : ProcResource<2> {
    //Even ALU pipelines
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pipeline Groups".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pipeline Groups”。

### Lines 93-100

```tablegen
    let Super = ALU;
  }
  def ALUO : ProcResource<2> {
    //Odd ALU pipelines
    let Super = ALU;
  }

  // Two DIV (Fixed Point Divide) units.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Odd ALU pipelines".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Odd ALU pipelines”。

### Lines 101-108

```tablegen
  def DIV : ProcResource<2>;

  // Four DP (Floating Point) units in total. Two even, two Odd.
  def DP : ProcResource<4>;
  def DPE : ProcResource<2> {
    //Even DP pipelines
    let Super = DP;
  }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Four DP (Floating Point) units in total. Two even, two Odd.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Four DP (Floating Point) units in total. Two even, two Odd.”。

### Lines 109-117

```tablegen
  def DPO : ProcResource<2> {
    //Odd DP pipelines
    let Super = DP;
  }

  // Four LS (Load or Store) units.
  def LS : ProcResource<4>;

  // Two PM (Permute) units.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Odd DP pipelines".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Odd DP pipelines”。

### Lines 118-124

```tablegen
  def PM : ProcResource<2>;

  // Only one DFU (Decimal Floating Point and Quad Precision) unit.
  def DFU : ProcResource<1>;

  // Only one Branch unit.
  def BR : ProcResource<1> {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Only one DFU (Decimal Floating Point and Quad Precision) unit.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Only one DFU (Decimal Floating Point and Quad Precision) unit.”。

### Lines 125-137

```tablegen
    let BufferSize = 16;
  }

  // Only one CY (Crypto) unit.
  def CY : ProcResource<1>;

  // ***************** SchedWriteRes Definitions *****************

  // Dispatcher
  // Dispatch Rules: '-' or 'V'
  // Vector ('V') - vector iops (128-bit operand) take only one decode and
  // dispatch slot but are dispatched to both the even and odd slices of a
  // superslice.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Only one CY (Crypto) unit.". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Only one CY (Crypto) unit.”。 这一段包含调度或处理器模型元数据。

### Lines 138-144

```tablegen
  def DISP_1C : SchedWriteRes<[DISP_NBR]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }
  // Dispatch Rules: 'E' 
  // Even slice ('E')- certain operations must be sent only to an even slice.
  // Also consumes odd dispatch slice slot of the same superslice at dispatch
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Dispatch Rules: 'E'". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Dispatch Rules: 'E'”。 这一段包含调度或处理器模型元数据。

### Lines 145-151

```tablegen
  def DISP_EVEN_1C : SchedWriteRes<[ DISPx02, DISPx13 ]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }
  // Dispatch Rules: 'P'
  // Paired ('P') - certain cracked and expanded iops are paired such that they
  // must dispatch together to the same superslice.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Dispatch Rules: 'P'". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Dispatch Rules: 'P'”。 这一段包含调度或处理器模型元数据。

### Lines 152-158

```tablegen
  def DISP_PAIR_1C : SchedWriteRes<[ DISP_SS, DISP_SS]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }
  // Tuple Restricted ('R') - certain iops preclude dispatching more than one
  // operation per slice for the super- slice to which they are dispatched
  def DISP_3SLOTS_1C : SchedWriteRes<[DISPx02, DISPx13, DISPxab]> {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Tuple Restricted ('R') - certain iops preclude dispatching more than one". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Tuple Restricted ('R') - certain iops preclude dispatching more than one”。 这一段包含调度或处理器模型元数据。

### Lines 159-168

```tablegen
    let NumMicroOps = 0;
    let Latency = 1;
  }
  // Each execution and branch slice can receive up to two iops per cycle
  def DISP_BR_1C : SchedWriteRes<[ DISPxab ]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }

  // Issue Ports
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 169-175

```tablegen
  def IP_AGEN_1C : SchedWriteRes<[IP_AGEN]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }

  def IP_EXEC_1C : SchedWriteRes<[IP_EXEC]> {
    let NumMicroOps = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 176-183

```tablegen
    let Latency = 1;
  }

  def IP_EXECE_1C : SchedWriteRes<[IP_EXECE]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 184-194

```tablegen
  def IP_EXECO_1C : SchedWriteRes<[IP_EXECO]> {
    let NumMicroOps = 0;
    let Latency = 1;
  }

  //Pipeline Groups

  // ALU Units
  // An ALU may take either 2 or 3 cycles to complete the operation.
  // However, the ALU unit is only ever busy for 1 cycle at a time and may
  // receive new instructions each cycle.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pipeline Groups". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pipeline Groups”。 这一段包含调度或处理器模型元数据。

### Lines 195-202

```tablegen
  def P9_ALU_2C : SchedWriteRes<[ALU]> {
    let Latency = 2;
  }

  def P9_ALUE_2C : SchedWriteRes<[ALUE]> {
    let Latency = 2;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 203-210

```tablegen
  def P9_ALUO_2C : SchedWriteRes<[ALUO]> {
    let Latency = 2;
  }

  def P9_ALU_3C : SchedWriteRes<[ALU]> {
    let Latency = 3;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 211-221

```tablegen
  def P9_ALUE_3C : SchedWriteRes<[ALUE]> {
    let Latency = 3;
  }

  def P9_ALUO_3C : SchedWriteRes<[ALUO]> {
    let Latency = 3;
  }

  // DIV Unit
  // A DIV unit may take from 5 to 40 cycles to complete.
  // Some DIV operations may keep the unit busy for up to 8 cycles.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DIV Unit". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DIV Unit”。 这一段包含调度或处理器模型元数据。

### Lines 222-229

```tablegen
  def P9_DIV_5C : SchedWriteRes<[DIV]> {
    let Latency = 5;
  }

  def P9_DIV_12C : SchedWriteRes<[DIV]> {
    let Latency = 12;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 230-236

```tablegen
  def P9_DIV_16C_8 : SchedWriteRes<[DIV]> {
    let ReleaseAtCycles = [8];
    let Latency = 16;
  }

  def P9_DIV_24C_8 : SchedWriteRes<[DIV]> {
    let ReleaseAtCycles = [8];
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 237-247

```tablegen
    let Latency = 24;
  }

  def P9_DIV_40C_8 : SchedWriteRes<[DIV]> {
    let ReleaseAtCycles = [8];
    let Latency = 40;
  }

  // DP Unit
  // A DP unit may take from 2 to 36 cycles to complete.
  // Some DP operations keep the unit busy for up to 10 cycles.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DP Unit". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DP Unit”。 这一段包含调度或处理器模型元数据。

### Lines 248-255

```tablegen
  def P9_DP_5C : SchedWriteRes<[DP]> {
    let Latency = 5;
  }

  def P9_DP_7C : SchedWriteRes<[DP]> {
    let Latency = 7;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 256-263

```tablegen
  def P9_DPE_7C : SchedWriteRes<[DPE]> {
    let Latency = 7;
  }

  def P9_DPO_7C : SchedWriteRes<[DPO]> {
    let Latency = 7;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 264-270

```tablegen
  def P9_DP_22C_5 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [5];
    let Latency = 22;
  }

  def P9_DPO_24C_8 : SchedWriteRes<[DPO]> {
    let ReleaseAtCycles = [8];
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 271-278

```tablegen
    let Latency = 24;
  }

  def P9_DPE_24C_8 : SchedWriteRes<[DPE]> {
    let ReleaseAtCycles = [8];
    let Latency = 24;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 279-285

```tablegen
  def P9_DP_26C_5 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [5];
    let Latency = 22;
  }

  def P9_DPE_27C_10 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [10];
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 286-293

```tablegen
    let Latency = 27;
  }

  def P9_DPO_27C_10 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [10];
    let Latency = 27;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 294-300

```tablegen
  def P9_DP_33C_8 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [8];
    let Latency = 33;
  }

  def P9_DPE_33C_8 : SchedWriteRes<[DPE]> {
    let ReleaseAtCycles = [8];
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 301-308

```tablegen
    let Latency = 33;
  }

  def P9_DPO_33C_8 : SchedWriteRes<[DPO]> {
    let ReleaseAtCycles = [8];
    let Latency = 33;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 309-315

```tablegen
  def P9_DP_36C_10 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [10];
    let Latency = 36;
  }

  def P9_DPE_36C_10 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [10];
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 316-325

```tablegen
    let Latency = 36;
  }

  def P9_DPO_36C_10 : SchedWriteRes<[DP]> {
    let ReleaseAtCycles = [10];
    let Latency = 36;
  }

  // PM Unit
  // Three cycle permute operations.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 326-335

```tablegen
  def P9_PM_3C : SchedWriteRes<[PM]> {
    let Latency = 3;
  }

  // Load and Store Units
  // Loads can have 4, 5 or 6 cycles of latency.
  // Stores are listed as having a single cycle of latency. This is not
  // completely accurate since it takes more than 1 cycle to actually store
  // the value. However, since the store does not produce a result it can be
  // considered complete after one cycle.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load and Store Units". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load and Store Units”。 这一段包含调度或处理器模型元数据。

### Lines 336-343

```tablegen
  def P9_LS_1C : SchedWriteRes<[LS]> {
    let Latency = 1;
  }

  def P9_LS_4C : SchedWriteRes<[LS]> {
    let Latency = 4;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 344-355

```tablegen
  def P9_LS_5C : SchedWriteRes<[LS]> {
    let Latency = 5;
  }

  def P9_LS_6C : SchedWriteRes<[LS]> {
    let Latency = 6;
  }

  // DFU Unit
  // Some of the most expensive ops use the DFU.
  // Can take from 12 cycles to 76 cycles to obtain a result.
  // The unit may be busy for up to 62 cycles.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DFU Unit". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DFU Unit”。 这一段包含调度或处理器模型元数据。

### Lines 356-364

```tablegen
  def P9_DFU_12C : SchedWriteRes<[DFU]> {
    let Latency = 12;
  }

  def P9_DFU_23C : SchedWriteRes<[DFU]> {
    let Latency = 23;
    let ReleaseAtCycles = [11];
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 365-371

```tablegen
  def P9_DFU_24C : SchedWriteRes<[DFU]> {
    let Latency = 24;
    let ReleaseAtCycles = [12];
  }

  def P9_DFU_37C : SchedWriteRes<[DFU]> {
    let Latency = 37;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 372-379

```tablegen
    let ReleaseAtCycles = [25];
  }

  def P9_DFU_58C : SchedWriteRes<[DFU]> {
    let Latency = 58;
    let ReleaseAtCycles = [44];
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 380-386

```tablegen
  def P9_DFU_76C : SchedWriteRes<[DFU]> {
    let Latency = 76;
    let ReleaseAtCycles = [62];
  }

  // 2 or 5 cycle latencies for the branch unit.
  def P9_BR_2C : SchedWriteRes<[BR]> {
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这一段包含调度或处理器模型元数据。

### Lines 387-394

```tablegen
    let Latency = 2;
  }

  def P9_BR_5C : SchedWriteRes<[BR]> {
    let Latency = 5;
  }

  // 6 cycle latency for the crypto unit
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这一段包含调度或处理器模型元数据。

### Lines 395-403

```tablegen
  def P9_CY_6C : SchedWriteRes<[CY]> {
    let Latency = 6;
  }

  // ***************** WriteSeq Definitions *****************

  // These are combinations of the resources listed above.
  // The idea is that some cracked instructions cannot be done in parallel and
  // so the latencies for their resources must be added.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "***************** WriteSeq Definitions *****************". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“***************** WriteSeq Definitions *****************”。 这一段包含调度或处理器模型元数据。

### Lines 404-410

```tablegen
  def P9_LoadAndALUOp_6C : WriteSequence<[P9_LS_4C, P9_ALU_2C]>;
  def P9_LoadAndALUOp_7C : WriteSequence<[P9_LS_5C, P9_ALU_2C]>;
  def P9_LoadAndALU2Op_7C : WriteSequence<[P9_LS_4C, P9_ALU_3C]>;
  def P9_LoadAndALU2Op_8C : WriteSequence<[P9_LS_5C, P9_ALU_3C]>;
  def P9_LoadAndPMOp_8C : WriteSequence<[P9_LS_5C, P9_PM_3C]>;
  def P9_IntDivAndALUOp_18C_8 : WriteSequence<[P9_DIV_16C_8, P9_ALU_2C]>;
  def P9_IntDivAndALUOp_26C_8 : WriteSequence<[P9_DIV_24C_8, P9_ALU_2C]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 411-417

```tablegen
  def P9_IntDivAndALUOp_42C_8 : WriteSequence<[P9_DIV_40C_8, P9_ALU_2C]>;
  def P9_StoreAndALUOp_3C : WriteSequence<[P9_LS_1C, P9_ALU_2C]>;
  def P9_ALUOpAndALUOp_4C : WriteSequence<[P9_ALU_2C, P9_ALU_2C]>;
  def P9_ALU2OpAndALU2Op_6C : WriteSequence<[P9_ALU_3C, P9_ALU_3C]>;
  def P9_ALUOpAndALUOpAndALUOp_6C :
    WriteSequence<[P9_ALU_2C, P9_ALU_2C, P9_ALU_2C]>;
  def P9_DPOpAndALUOp_7C : WriteSequence<[P9_DP_5C, P9_ALU_2C]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 418-425

```tablegen
  def P9_DPOpAndALU2Op_10C : WriteSequence<[P9_DP_7C, P9_ALU_3C]>;
  def P9_DPOpAndALU2Op_25C_5 : WriteSequence<[P9_DP_22C_5, P9_ALU_3C]>;
  def P9_DPOpAndALU2Op_29C_5 : WriteSequence<[P9_DP_26C_5, P9_ALU_3C]>;
  def P9_DPOpAndALU2Op_36C_8 : WriteSequence<[P9_DP_33C_8, P9_ALU_3C]>;
  def P9_DPOpAndALU2Op_39C_10 : WriteSequence<[P9_DP_36C_10, P9_ALU_3C]>;
  def P9_BROpAndALUOp_7C : WriteSequence<[P9_BR_5C, P9_ALU_2C]>;

  // Include the resource requirements of individual instructions.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 426-429

```tablegen
  include "P9InstrResources.td"

}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Subtarget features / 子目标特性
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `P9InstrResources.td`

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
