# PPCScheduleE500.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCScheduleE500.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCScheduleE500.td - e500 Scheduling Defs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCScheduleE500.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCScheduleE500.td - e500 Scheduling Defs ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段包含调度或处理器模型元数据。

### Lines 7-14

```tablegen
//===----------------------------------------------------------------------===//
//
// This file defines the itinerary class data for the Freescale e500 32-bit
// Power processor.
//
// All information is derived from the "e500 Core Reference Manual",
// Freescale Document Number E500MCRM, Rev. 1, 03/2012.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines the itinerary class data for the Freescale e500 32-bit".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines the itinerary class data for the Freescale e500 32-bit”。

### Lines 15-21

```tablegen
//===----------------------------------------------------------------------===//
// Relevant functional units in the Freescale e500 core:
//
//  * Decode & Dispatch
//    Can dispatch up to 2 instructions per clock cycle to either the GPR Issue
//    queues (GIQx) or Branch issue queue (BIQ).
def E500_DIS0 : FuncUnit; // Dispatch stage - insn 1
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relevant functional units in the Freescale e500 core:". Notable symbols in this range include `E500_DIS0`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relevant functional units in the Freescale e500 core:”。 该区间中较显眼的符号包括 `E500_DIS0`。

### Lines 22-28

```tablegen
def E500_DIS1 : FuncUnit; // Dispatch stage - insn 2

//  * Execute
//    6 pipelined execution units: SU0, SU1, BU, LSU, MU.
//    Some instructions can only execute in SU0 but not SU1.
def E500_SU0  : FuncUnit; // Simple unit 0
def E500_SU1  : FuncUnit; // Simple unit 1
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "* Execute". Notable symbols in this range include `E500_DIS1`, `E500_SU0`, `E500_SU1`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“* Execute”。 该区间中较显眼的符号包括 `E500_DIS1`, `E500_SU0`, `E500_SU1`。

### Lines 29-36

```tablegen
def E500_BU    : FuncUnit; // Branch unit
def E500_MU    : FuncUnit; // MU pipeline
def E500_LSU_0 : FuncUnit; // LSU pipeline

def E500_GPR_Bypass : Bypass;
def E500_CR_Bypass  : Bypass;
def E500_DivBypass  : Bypass;
```
- **EN**: Adds declarative TableGen records such as `E500_BU`, `E500_MU`, `E500_LSU_0` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `E500_BU`, `E500_MU`, `E500_LSU_0`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 37-74

```tablegen
def PPCE500Itineraries : ProcessorItineraries<
  [E500_DIS0, E500_DIS1, E500_SU0, E500_SU1, E500_BU,
   E500_MU, E500_LSU_0],
  [E500_CR_Bypass, E500_GPR_Bypass, E500_DivBypass], [
  InstrItinData<IIC_IntSimple,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntGeneral,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntISEL,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1, 1, 1], // Latency = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass,
                                  E500_CR_Bypass]>,
  InstrItinData<IIC_IntCompare,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [5, 1, 1], // Latency = 1 or 2
                                 [E500_CR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntDivW,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_MU], 0>,
                                  InstrStage<14, [E500_MU]>],
                                 [17, 1, 1], // Latency=4..35, Repeat= 4..35
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHW,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_MU]>],
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHWU,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_MU]>],
```
- **EN**: Adds declarative TableGen records such as `PPCE500Itineraries` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCE500Itineraries`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 75-112

```tablegen
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntMulLI,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_MU]>],
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntRotate,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntShift,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_IntTrapW,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<2, [E500_SU0]>],
                                 [5, 1], // Latency = 2, Repeat rate = 2
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_BrB,         [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_BU]>],
                                 [4, 1], // Latency = 1
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_BrCR,        [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_BU]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500_CR_Bypass,
                                  E500_CR_Bypass, E500_CR_Bypass]>,
  InstrItinData<IIC_BrMCR,       [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_BU]>],
                                 [4, 1], // Latency = 1
                                 [E500_CR_Bypass, E500_CR_Bypass]>,
  InstrItinData<IIC_BrMCRX,      [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1, 1], // Latency = 1
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 113-150

```tablegen
                                 [E500_CR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBA,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3, Repeat rate = 1
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBF,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBI,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoad,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoadUpd, [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLoadUpdX,[InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStStore,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTU,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500_GPR_Bypass],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 151-188

```tablegen
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStSTUX,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStICBI,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHA,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAU,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAUX,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLMW,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [7, 1], // Latency = r+3
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLWARX,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<3, [E500_LSU_0]>],
                                 [6, 1, 1], // Latency = 3, Repeat rate = 3
                                 [E500_GPR_Bypass,
                                  E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTWCX,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500_GPR_Bypass]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 189-226

```tablegen
  InstrItinData<IIC_LdStSync,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0]>]>,
  InstrItinData<IIC_SprMFSR,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_SU0]>],
                                 [7, 1],
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMTMSR,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<2, [E500_SU0, E500_SU1]>],
                                 [5, 1], // Latency = 2, Repeat rate = 4
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSR,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0]>],
                                 [5, 1],
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprTLBSYNC,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_LSU_0], 0>]>,
  InstrItinData<IIC_SprMFCR,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<5, [E500_SU0]>],
                                 [8, 1],
                                 [E500_GPR_Bypass, E500_CR_Bypass]>,
  InstrItinData<IIC_SprMFCRF,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<5, [E500_SU0]>],
                                 [8, 1],
                                 [E500_GPR_Bypass, E500_CR_Bypass]>,
  InstrItinData<IIC_SprMFPMR,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_SU0]>],
                                 [7, 1], // Latency = 4, Repeat rate = 4
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMFMSR,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_SU0]>],
                                 [7, 1], // Latency = 4, Repeat rate = 4
                                 [E500_GPR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMFSPR,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1], // Latency = 1, Repeat rate = 1
                                 [E500_GPR_Bypass, E500_CR_Bypass]>,
  InstrItinData<IIC_SprMTPMR,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 227-264

```tablegen
                                 [4, 1], // Latency = 1, Repeat rate = 1
                                 [E500_CR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMFTB,     [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_SU0]>],
                                 [7, 1], // Latency = 4, Repeat rate = 4
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSPR,    [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0, E500_SU1]>],
                                 [4, 1], // Latency = 1, Repeat rate = 1
                                 [E500_CR_Bypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSRIN,   [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0]>],
                                 [4, 1],
                                 [NoBypass, E500_GPR_Bypass]>,
  InstrItinData<IIC_FPDGeneral,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<6, [E500_MU]>],
                                 [9, 1, 1],  // Latency = 6, Repeat rate = 1
                                 [NoBypass]>,
  InstrItinData<IIC_FPSGeneral,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_MU]>],
                                 [7, 1, 1],  // Latency = 4, Repeat rate = 1
                                 [NoBypass]>,
  InstrItinData<IIC_FPDivD,      [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<32, [E500_MU]>],
                                 [35, 1, 1], // Latency = 32, Repeat rate = 32
                                 [E500_DivBypass]>,
  InstrItinData<IIC_FPDivS,      [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<29, [E500_MU]>],
                                 [32, 1, 1], // Latency = 29, Repeat rate = 29
                                 [E500_DivBypass]>,
  InstrItinData<IIC_VecGeneral,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<1, [E500_SU0]>],
                                 [4, 1, 1], // Latency = 1, Repeat rate = 1
                                 [NoBypass]>,
  InstrItinData<IIC_VecComplex,  [InstrStage<1, [E500_DIS0, E500_DIS1], 0>,
                                  InstrStage<4, [E500_MU]>],
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [NoBypass]>
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 265-271

```tablegen
]>;

// ===---------------------------------------------------------------------===//
// e500 machine model for scheduling and other instruction cost heuristics.

def PPCE500Model : SchedMachineModel {
  let IssueWidth = 2;  // 2 micro-ops are dispatched per cycle.
```
- **EN**: Adds declarative TableGen records such as `PPCE500Model` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCE500Model`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 272-279

```tablegen
  let LoadLatency = 5; // Optimistic load latency assuming bypass.
                       // This is overriden by OperandCycles if the
                       // Itineraries are queried instead.

  let CompleteModel = 0;

  let Itineraries = PPCE500Itineraries;
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is overriden by OperandCycles if the". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is overriden by OperandCycles if the”。 这一段包含调度或处理器模型元数据。

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
