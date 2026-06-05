# PPCScheduleG5.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCScheduleG5.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCScheduleG5.td - PPC G5 Scheduling Definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCScheduleG5.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCScheduleG5.td - PPC G5 Scheduling Definitions ---*- tablegen -*-===//
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
// This file defines the itinerary class data for the G5 (970) processor.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines the itinerary class data for the G5 (970) processor.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines the itinerary class data for the G5 (970) processor.”。

### Lines 11-17

```tablegen
//===----------------------------------------------------------------------===//

def G5_BPU    : FuncUnit; // Branch unit
def G5_SLU    : FuncUnit; // Store/load unit
def G5_SRU    : FuncUnit; // special register unit
def G5_IU1    : FuncUnit; // integer unit 1 (simple)
def G5_IU2    : FuncUnit; // integer unit 2 (complex)
```
- **EN**: Adds declarative TableGen records such as `G5_BPU`, `G5_SLU`, `G5_SRU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `G5_BPU`, `G5_SLU`, `G5_SRU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 18-24

```tablegen
def G5_FPU1   : FuncUnit; // floating point unit 1
def G5_FPU2   : FuncUnit; // floating point unit 2
def G5_VPU    : FuncUnit; // vector permutation unit
def G5_VIU1   : FuncUnit; // vector integer unit 1 (simple)
def G5_VIU2   : FuncUnit; // vector integer unit 2 (complex)
def G5_VFPU   : FuncUnit; // vector floating point unit
```
- **EN**: Adds declarative TableGen records such as `G5_FPU1`, `G5_FPU2`, `G5_VPU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `G5_FPU1`, `G5_FPU2`, `G5_VPU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 25-62

```tablegen
def G5Itineraries : ProcessorItineraries<
  [G5_IU1, G5_IU2, G5_SLU, G5_BPU, G5_FPU1, G5_FPU2,
   G5_VFPU, G5_VIU1, G5_VIU2, G5_VPU], [], [
  InstrItinData<IIC_IntSimple   , [InstrStage<2, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntGeneral  , [InstrStage<2, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntCompare  , [InstrStage<3, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntDivD     , [InstrStage<68, [G5_IU1]>]>,
  InstrItinData<IIC_IntDivW     , [InstrStage<36, [G5_IU1]>]>,
  InstrItinData<IIC_IntMFFS     , [InstrStage<6, [G5_IU2]>]>,
  InstrItinData<IIC_IntMFVSCR   , [InstrStage<1, [G5_VFPU]>]>,
  InstrItinData<IIC_IntMTFSB0   , [InstrStage<6, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_IntMulHD    , [InstrStage<7, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntMulHW    , [InstrStage<5, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntMulHWU   , [InstrStage<5, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntMulLI    , [InstrStage<4, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntRFID     , [InstrStage<1, [G5_IU2]>]>,
  InstrItinData<IIC_IntRotateD  , [InstrStage<2, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntRotateDI , [InstrStage<2, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntRotate   , [InstrStage<4, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntShift    , [InstrStage<2, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntTrapD    , [InstrStage<1, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_IntTrapW    , [InstrStage<1, [G5_IU1, G5_IU2]>]>,
  InstrItinData<IIC_BrB         , [InstrStage<1, [G5_BPU]>]>,
  InstrItinData<IIC_BrCR        , [InstrStage<4, [G5_BPU]>]>,
  InstrItinData<IIC_BrMCR       , [InstrStage<2, [G5_BPU]>]>,
  InstrItinData<IIC_BrMCRX      , [InstrStage<3, [G5_BPU]>]>,
  InstrItinData<IIC_LdStDCBF    , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLoad    , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLoadUpd , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLoadUpdX, [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStStore   , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStDSS     , [InstrStage<10, [G5_SLU]>]>,
  InstrItinData<IIC_LdStICBI    , [InstrStage<40, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTFD    , [InstrStage<4, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTFDU   , [InstrStage<4, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLD      , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLDU     , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLDUX    , [InstrStage<3, [G5_SLU]>]>,
```
- **EN**: Adds declarative TableGen records such as `G5Itineraries` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `G5Itineraries`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 63-100

```tablegen
  InstrItinData<IIC_LdStLDARX   , [InstrStage<11, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLFD     , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLFDU    , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLFDUX   , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLHA     , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLHAU    , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLHAUX   , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLMW     , [InstrStage<64, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLVecX   , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLWA     , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStLWARX   , [InstrStage<11, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSLBIA   , [InstrStage<40, [G5_SLU]>]>, // needs work
  InstrItinData<IIC_LdStSLBIE   , [InstrStage<2, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTD     , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTU     , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTUX    , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTDCX   , [InstrStage<11, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTVEBX  , [InstrStage<5, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSTWCX   , [InstrStage<11, [G5_SLU]>]>,
  InstrItinData<IIC_LdStSync    , [InstrStage<35, [G5_SLU]>]>,
  InstrItinData<IIC_SprISYNC    , [InstrStage<40, [G5_SLU]>]>, // needs work
  InstrItinData<IIC_SprMFSR     , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_SprMTMSR    , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_SprMTSR     , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_SprTLBSYNC  , [InstrStage<3, [G5_SLU]>]>,
  InstrItinData<IIC_SprMFCR     , [InstrStage<2, [G5_IU2]>]>,
  InstrItinData<IIC_SprMFCRF    , [InstrStage<2, [G5_IU2]>]>,
  InstrItinData<IIC_SprMFMSR    , [InstrStage<3, [G5_IU2]>]>,
  InstrItinData<IIC_SprMFSPR    , [InstrStage<3, [G5_IU2]>]>,
  InstrItinData<IIC_SprMFTB     , [InstrStage<10, [G5_IU2]>]>,
  InstrItinData<IIC_SprMTSPR    , [InstrStage<8, [G5_IU2]>]>,
  InstrItinData<IIC_SprSC       , [InstrStage<1, [G5_IU2]>]>,
  InstrItinData<IIC_FPGeneral   , [InstrStage<6, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPAddSub    , [InstrStage<6, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPCompare   , [InstrStage<8, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPDivD      , [InstrStage<33, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPDivS      , [InstrStage<33, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPFused     , [InstrStage<6, [G5_FPU1, G5_FPU2]>]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 101-116

```tablegen
  InstrItinData<IIC_FPRes       , [InstrStage<6, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPSqrtD     , [InstrStage<40, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_FPSqrtS     , [InstrStage<40, [G5_FPU1, G5_FPU2]>]>,
  InstrItinData<IIC_VecGeneral  , [InstrStage<2, [G5_VIU1]>]>,
  InstrItinData<IIC_VecFP       , [InstrStage<8, [G5_VFPU]>]>,
  InstrItinData<IIC_VecFPCompare, [InstrStage<2, [G5_VFPU]>]>,
  InstrItinData<IIC_VecComplex  , [InstrStage<5, [G5_VIU2]>]>,
  InstrItinData<IIC_VecPerm     , [InstrStage<3, [G5_VPU]>]>,
  InstrItinData<IIC_VecFPRound  , [InstrStage<8, [G5_VFPU]>]>,
  InstrItinData<IIC_VecVSL      , [InstrStage<2, [G5_VIU1]>]>,
  InstrItinData<IIC_VecVSR      , [InstrStage<3, [G5_VPU]>]>
]>;

// ===---------------------------------------------------------------------===//
// G5 machine model for scheduling and other instruction cost heuristics.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 117-123

```tablegen
def G5Model : SchedMachineModel {
  let IssueWidth = 4;  // 4 (non-branch) instructions are dispatched per cycle.
  let LoadLatency = 3; // Optimistic load latency assuming bypass.
                       // This is overriden by OperandCycles if the
                       // Itineraries are queried instead.
  let MispredictPenalty = 16;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is overriden by OperandCycles if the". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is overriden by OperandCycles if the”。 这一段包含调度或处理器模型元数据。

### Lines 124-128

```tablegen
  let CompleteModel = 0;

  let Itineraries = G5Itineraries;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
