# PPCScheduleA2.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCScheduleA2.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCScheduleA2.td - PPC A2 Scheduling Definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCScheduleA2.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===- PPCScheduleA2.td - PPC A2 Scheduling Definitions --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段包含调度或处理器模型元数据。

### Lines 7-12

```tablegen
//===----------------------------------------------------------------------===//

// Primary reference:
// A2 Processor User's Manual.
// IBM (as updated in) 2010.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Primary reference:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Primary reference:”。

### Lines 13-21

```tablegen
//===----------------------------------------------------------------------===//
// Functional units on the PowerPC A2 chip sets
//
def A2_XU     : FuncUnit; // A2_XU pipeline
def A2_FU     : FuncUnit; // FI pipeline

//
// This file defines the itinerary class data for the PPC A2 processor.
//
```
- **EN**: Adds declarative TableGen records such as `A2_XU`, `A2_FU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `A2_XU`, `A2_FU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 22-59

```tablegen
//===----------------------------------------------------------------------===//


def PPCA2Itineraries : ProcessorItineraries<
  [A2_XU, A2_FU], [], [
  InstrItinData<IIC_IntSimple,   [InstrStage<1, [A2_XU]>],
                                 [1, 0, 0]>,
  InstrItinData<IIC_IntGeneral,  [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0]>,
  InstrItinData<IIC_IntISEL,     [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0, 0]>,
  InstrItinData<IIC_IntCompare,  [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0]>,
  InstrItinData<IIC_IntDivW,     [InstrStage<1, [A2_XU]>],
                                 [39, 0, 0]>,
  InstrItinData<IIC_IntDivD,     [InstrStage<1, [A2_XU]>],
                                 [71, 0, 0]>,
  InstrItinData<IIC_IntMulHW,    [InstrStage<1, [A2_XU]>],
                                 [5, 0, 0]>,
  InstrItinData<IIC_IntMulHWU,   [InstrStage<1, [A2_XU]>],
                                 [5, 0, 0]>,
  InstrItinData<IIC_IntMulLI,    [InstrStage<1, [A2_XU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_IntRotate,   [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0]>,
  InstrItinData<IIC_IntRotateD,  [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0]>,
  InstrItinData<IIC_IntRotateDI, [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0]>,
  InstrItinData<IIC_IntShift,    [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0]>,
  InstrItinData<IIC_IntTrapW,    [InstrStage<1, [A2_XU]>],
                                 [2, 0]>,
  InstrItinData<IIC_IntTrapD,    [InstrStage<1, [A2_XU]>],
                                 [2, 0]>,
  InstrItinData<IIC_BrB,         [InstrStage<1, [A2_XU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_BrCR,        [InstrStage<1, [A2_XU]>],
```
- **EN**: Adds declarative TableGen records such as `PPCA2Itineraries` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCA2Itineraries`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 60-97

```tablegen
                                 [1, 0, 0]>,
  InstrItinData<IIC_BrMCR,       [InstrStage<1, [A2_XU]>],
                                 [5, 0, 0]>,
  InstrItinData<IIC_BrMCRX,      [InstrStage<1, [A2_XU]>],
                                 [1, 0, 0]>,
  InstrItinData<IIC_LdStDCBA,    [InstrStage<1, [A2_XU]>],
                                 [1, 0, 0]>,
  InstrItinData<IIC_LdStDCBF,    [InstrStage<1, [A2_XU]>],
                                 [1, 0, 0]>,
  InstrItinData<IIC_LdStDCBI,    [InstrStage<1, [A2_XU]>],
                                 [1, 0, 0]>,
  InstrItinData<IIC_LdStLoad,    [InstrStage<1, [A2_XU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_LdStLoadUpd, [InstrStage<1, [A2_XU]>],
                                 [6, 8, 0, 0]>,
  InstrItinData<IIC_LdStLoadUpdX,[InstrStage<1, [A2_XU]>],
                                 [6, 8, 0, 0]>,
  InstrItinData<IIC_LdStLDU,     [InstrStage<1, [A2_XU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_LdStLDUX,    [InstrStage<1, [A2_XU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_LdStStore,   [InstrStage<1, [A2_XU]>],
                                 [0, 0, 0]>,
  InstrItinData<IIC_LdStICBI,    [InstrStage<1, [A2_XU]>],
                                 [16, 0, 0]>,
  InstrItinData<IIC_LdStSTFD,    [InstrStage<1, [A2_XU]>],
                                 [0, 0, 0]>,
  InstrItinData<IIC_LdStSTFDU,   [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0, 0]>,
  InstrItinData<IIC_LdStLFD,     [InstrStage<1, [A2_XU]>],
                                 [7, 0, 0]>,
  InstrItinData<IIC_LdStLFDU,    [InstrStage<1, [A2_XU]>],
                                 [7, 9, 0, 0]>,
  InstrItinData<IIC_LdStLFDUX,   [InstrStage<1, [A2_XU]>],
                                 [7, 9, 0, 0]>,
  InstrItinData<IIC_LdStLHA,     [InstrStage<1, [A2_XU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_LdStLHAU,    [InstrStage<1, [A2_XU]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 98-135

```tablegen
                                 [6, 8, 0, 0]>,
  InstrItinData<IIC_LdStLHAUX,   [InstrStage<1, [A2_XU]>],
                                 [6, 8, 0, 0]>,
  InstrItinData<IIC_LdStLWARX,   [InstrStage<1, [A2_XU]>],
                                 [82, 0, 0]>, // L2 latency
  InstrItinData<IIC_LdStSTD,     [InstrStage<1, [A2_XU]>],
                                 [0, 0, 0]>,
  InstrItinData<IIC_LdStSTU,     [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0, 0]>,
  InstrItinData<IIC_LdStSTUX,    [InstrStage<1, [A2_XU]>],
                                 [2, 0, 0, 0]>,
  InstrItinData<IIC_LdStSTDCX,   [InstrStage<1, [A2_XU]>],
                                 [82, 0, 0]>, // L2 latency
  InstrItinData<IIC_LdStSTWCX,   [InstrStage<1, [A2_XU]>],
                                 [82, 0, 0]>, // L2 latency
  InstrItinData<IIC_LdStSync,    [InstrStage<1, [A2_XU]>],
                                 [6]>,
  InstrItinData<IIC_SprISYNC,    [InstrStage<1, [A2_XU]>],
                                 [16]>,
  InstrItinData<IIC_SprMTMSR,    [InstrStage<1, [A2_XU]>],
                                 [16, 0]>,
  InstrItinData<IIC_SprMFCR,     [InstrStage<1, [A2_XU]>],
                                 [6, 0]>,
  InstrItinData<IIC_SprMFCRF,    [InstrStage<1, [A2_XU]>],
                                 [1, 0]>,
  InstrItinData<IIC_SprMFMSR,    [InstrStage<1, [A2_XU]>],
                                 [4, 0]>,
  InstrItinData<IIC_SprMFSPR,    [InstrStage<1, [A2_XU]>],
                                 [6, 0]>,
  InstrItinData<IIC_SprMFTB,     [InstrStage<1, [A2_XU]>],
                                 [4, 0]>,
  InstrItinData<IIC_SprMTSPR,    [InstrStage<1, [A2_XU]>],
                                 [6, 0]>,
  InstrItinData<IIC_SprRFI,      [InstrStage<1, [A2_XU]>],
                                 [16]>,
  InstrItinData<IIC_SprSC,       [InstrStage<1, [A2_XU]>],
                                 [16]>,
  InstrItinData<IIC_FPGeneral,   [InstrStage<1, [A2_FU]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 136-157

```tablegen
                                 [6, 0, 0]>,
  InstrItinData<IIC_FPAddSub,    [InstrStage<1, [A2_FU]>],
                                 [6, 0, 0]>,
  InstrItinData<IIC_FPCompare,   [InstrStage<1, [A2_FU]>],
                                 [5, 0, 0]>,
  InstrItinData<IIC_FPDivD,      [InstrStage<1, [A2_FU]>],
                                 [72, 0, 0]>,
  InstrItinData<IIC_FPDivS,      [InstrStage<1, [A2_FU]>],
                                 [59, 0, 0]>,
  InstrItinData<IIC_FPSqrtD,     [InstrStage<1, [A2_FU]>],
                                 [69, 0, 0]>,
  InstrItinData<IIC_FPSqrtS,     [InstrStage<1, [A2_FU]>],
                                 [65, 0, 0]>,
  InstrItinData<IIC_FPFused,     [InstrStage<1, [A2_FU]>],
                                 [6, 0, 0, 0]>,
  InstrItinData<IIC_FPRes,       [InstrStage<1, [A2_FU]>],
                                 [6, 0]>
]>;

// ===---------------------------------------------------------------------===//
// A2 machine model for scheduling and other instruction cost heuristics.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 158-164

```tablegen
def PPCA2Model : SchedMachineModel {
  let IssueWidth = 1;  // 1 instruction is dispatched per cycle.
  let LoadLatency = 6; // Optimistic load latency assuming bypass.
                       // This is overriden by OperandCycles if the
                       // Itineraries are queried instead.
  let MispredictPenalty = 13;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is overriden by OperandCycles if the". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is overriden by OperandCycles if the”。 这一段包含调度或处理器模型元数据。

### Lines 165-169

```tablegen
  let CompleteModel = 0;

  let Itineraries = PPCA2Itineraries;
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
