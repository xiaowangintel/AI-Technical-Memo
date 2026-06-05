# PPCScheduleE5500.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCScheduleE5500.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCScheduleE500mc.td - e5500 Scheduling Defs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCScheduleE5500.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCScheduleE500mc.td - e5500 Scheduling Defs -------*- tablegen -*-===//
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
// This file defines the itinerary class data for the Freescale e5500 64-bit
// Power processor.
//
// All information is derived from the "e5500 Core Reference Manual",
// Freescale Document Number e5500RM, Rev. 1, 03/2012.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines the itinerary class data for the Freescale e5500 64-bit".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines the itinerary class data for the Freescale e5500 64-bit”。

### Lines 15-21

```tablegen
//===----------------------------------------------------------------------===//
// Relevant functional units in the Freescale e5500 core
// (These are the same as for the e500mc)
//
//  * Decode & Dispatch
//    Can dispatch up to 2 instructions per clock cycle to either the GPR Issue
//    queues (GIQx), FP Issue Queue (FIQ), or Branch issue queue (BIQ).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relevant functional units in the Freescale e5500 core".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relevant functional units in the Freescale e5500 core”。

### Lines 22-28

```tablegen
def E5500_DIS0 : FuncUnit;
def E5500_DIS1 : FuncUnit;

//  * Execute
//    6 pipelined execution units: SFX0, SFX1, BU, FPU, LSU, CFX.
//    The CFX has a bypass path, allowing non-divide instructions to execute
//    while a divide instruction is being executed.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "* Execute". Notable symbols in this range include `E5500_DIS0`, `E5500_DIS1`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“* Execute”。 该区间中较显眼的符号包括 `E5500_DIS0`, `E5500_DIS1`。

### Lines 29-35

```tablegen
def E5500_SFX0  : FuncUnit; // Simple unit 0
def E5500_SFX1  : FuncUnit; // Simple unit 1
def E5500_BU    : FuncUnit; // Branch unit
def E5500_CFX_DivBypass
                : FuncUnit; // CFX divide bypass path
def E5500_CFX_0 : FuncUnit; // CFX pipeline stage 0
```
- **EN**: Adds declarative TableGen records such as `E5500_SFX0`, `E5500_SFX1`, `E5500_BU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `E5500_SFX0`, `E5500_SFX1`, `E5500_BU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 36-42

```tablegen
def E5500_CFX_1 : FuncUnit; // CFX pipeline stage 1

def E5500_LSU_0 : FuncUnit; // LSU pipeline
def E5500_FPU_0 : FuncUnit; // FPU pipeline

def E5500_GPR_Bypass : Bypass;
def E5500_FPR_Bypass : Bypass;
```
- **EN**: Adds declarative TableGen records such as `E5500_CFX_1`, `E5500_LSU_0`, `E5500_FPU_0` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `E5500_CFX_1`, `E5500_LSU_0`, `E5500_FPU_0`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 43-80

```tablegen
def E5500_CR_Bypass  : Bypass;

def PPCE5500Itineraries : ProcessorItineraries<
  [E5500_DIS0, E5500_DIS1, E5500_SFX0, E5500_SFX1, E5500_BU,
   E5500_CFX_DivBypass, E5500_CFX_0, E5500_CFX_1,
   E5500_LSU_0, E5500_FPU_0],
  [E5500_CR_Bypass, E5500_GPR_Bypass, E5500_FPR_Bypass], [
  InstrItinData<IIC_IntSimple,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1]>],
                                 [5, 2, 2], // Latency = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntGeneral,  [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1]>],
                                 [5, 2, 2], // Latency = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntISEL,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1]>],
                                 [5, 2, 2, 2], // Latency = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass,
                                  E5500_CR_Bypass]>,
  InstrItinData<IIC_IntCompare,  [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1]>],
                                 [6, 2, 2], // Latency = 1 or 2
                                 [E5500_CR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntDivD,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0], 0>,
                                  InstrStage<26, [E5500_CFX_DivBypass]>],
                                 [30, 2, 2], // Latency= 4..26, Repeat rate= 4..26
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntDivW,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0], 0>,
                                  InstrStage<16, [E5500_CFX_DivBypass]>],
                                 [20, 2, 2], // Latency= 4..16, Repeat rate= 4..16
```
- **EN**: Adds declarative TableGen records such as `E5500_CR_Bypass`, `PPCE5500Itineraries` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `E5500_CR_Bypass`, `PPCE5500Itineraries`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 81-118

```tablegen
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntMFFS,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_FPU_0]>],
                                 [11], // Latency = 7, Repeat rate = 1
                                 [E5500_FPR_Bypass]>,
  InstrItinData<IIC_IntMTFSB0,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<7, [E5500_FPU_0]>],
                                 [11, 2, 2], // Latency = 7, Repeat rate = 7
                                 [NoBypass, NoBypass, NoBypass]>,
  InstrItinData<IIC_IntMulHD,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0], 0>,
                                  InstrStage<2, [E5500_CFX_1]>],
                                 [9, 2, 2], // Latency = 4..7, Repeat rate = 2..4
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHW,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0], 0>,
                                  InstrStage<1, [E5500_CFX_1]>],
                                 [8, 2, 2], // Latency = 4, Repeat rate = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHWU,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0], 0>,
                                  InstrStage<1, [E5500_CFX_1]>],
                                 [8, 2, 2], // Latency = 4, Repeat rate = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntMulLI,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0], 0>,
                                  InstrStage<2, [E5500_CFX_1]>],
                                 [8, 2, 2], // Latency = 4 or 5, Repeat = 2
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntRotate,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1]>],
                                 [5, 2, 2], // Latency = 1
                                 [E5500_GPR_Bypass,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 119-156

```tablegen
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntRotateD,  [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<2, [E5500_SFX0, E5500_SFX1]>],
                                 [6, 2, 2], // Latency = 2, Repeat rate = 2
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntRotateDI, [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1]>],
                                 [5, 2, 2], // Latency = 1, Repeat rate = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntShift,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<2, [E5500_SFX0, E5500_SFX1]>],
                                 [6, 2, 2], // Latency = 2, Repeat rate = 2
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_IntTrapW,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<2, [E5500_SFX0]>],
                                 [6, 2], // Latency = 2, Repeat rate = 2
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_BrB,         [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_BU]>],
                                 [5, 2], // Latency = 1
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_BrCR,        [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_BU]>],
                                 [5, 2, 2], // Latency = 1
                                 [E5500_CR_Bypass,
                                  E5500_CR_Bypass, E5500_CR_Bypass]>,
  InstrItinData<IIC_BrMCR,       [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_BU]>],
                                 [5, 2], // Latency = 1
                                 [E5500_CR_Bypass, E5500_CR_Bypass]>,
  InstrItinData<IIC_BrMCRX,      [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0]>],
                                 [5, 2, 2], // Latency = 1
                                 [E5500_CR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBA,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 157-194

```tablegen
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBF,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBI,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoad,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoadUpd, [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLoadUpdX,[InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLD,      [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLDARX,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<3, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 3
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLDU,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 195-232

```tablegen
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLDUX,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStStore,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStICBI,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTFD,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTFDU,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLFD,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [8, 2, 2], // Latency = 4, Repeat rate = 1
                                 [E5500_FPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLFDU,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 233-270

```tablegen
                                 [8, 2, 2], // Latency = 4, Repeat rate = 1
                                 [E5500_FPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLFDUX,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [8, 2, 2], // Latency = 4, Repeat rate = 1
                                 [E5500_FPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLHA,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAU,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLHAUX,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLMW,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<4, [E5500_LSU_0]>],
                                 [8, 2], // Latency = r+3, Repeat rate = r+3
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStLWARX,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<3, [E5500_LSU_0]>],
                                 [7, 2, 2], // Latency = 3, Repeat rate = 3
                                 [E5500_GPR_Bypass,
                                  E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTD,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 271-308

```tablegen
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTDCX,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTU,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStSTUX,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_SFX0, E5500_SFX1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStSTWCX,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>],
                                 [7, 2], // Latency = 3, Repeat rate = 1
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_LdStSync,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0]>]>,
  InstrItinData<IIC_SprMTMSR,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<2, [E5500_CFX_0]>],
                                 [6, 2], // Latency = 2, Repeat rate = 4
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_SprTLBSYNC,  [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_LSU_0], 0>]>,
  InstrItinData<IIC_SprMFCR,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<5, [E5500_CFX_0]>],
                                 [9, 2], // Latency = 5, Repeat rate = 5
                                 [E5500_GPR_Bypass, E5500_CR_Bypass]>,
  InstrItinData<IIC_SprMFCRF,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<5, [E5500_CFX_0]>],
                                 [9, 2], // Latency = 5, Repeat rate = 5
                                 [E5500_GPR_Bypass, E5500_CR_Bypass]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 309-346

```tablegen
  InstrItinData<IIC_SprMFPMR,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<4, [E5500_CFX_0]>],
                                 [8, 2], // Latency = 4, Repeat rate = 4
                                 [E5500_GPR_Bypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_SprMFSPR,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0]>],
                                 [5], // Latency = 1, Repeat rate = 1
                                 [E5500_GPR_Bypass]>,
  InstrItinData<IIC_SprMTPMR,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0]>],
                                 [5], // Latency = 1, Repeat rate = 1
                                 [E5500_GPR_Bypass]>,
  InstrItinData<IIC_SprMFTB,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<4, [E5500_CFX_0]>],
                                 [8, 2], // Latency = 4, Repeat rate = 4
                                 [NoBypass, E5500_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSPR,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_CFX_0]>],
                                 [5], // Latency = 1, Repeat rate = 1
                                 [E5500_GPR_Bypass]>,
  InstrItinData<IIC_FPGeneral,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_FPU_0]>],
                                 [11, 2, 2], // Latency = 7, Repeat rate = 1
                                 [E5500_FPR_Bypass,
                                  E5500_FPR_Bypass, E5500_FPR_Bypass]>,
  InstrItinData<IIC_FPAddSub,    [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_FPU_0]>],
                                 [11, 2, 2], // Latency = 7, Repeat rate = 1
                                 [E5500_FPR_Bypass,
                                  E5500_FPR_Bypass, E5500_FPR_Bypass]>,
  InstrItinData<IIC_FPCompare,   [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_FPU_0]>],
                                 [11, 2, 2], // Latency = 7, Repeat rate = 1
                                 [E5500_CR_Bypass,
                                  E5500_FPR_Bypass, E5500_FPR_Bypass]>,
  InstrItinData<IIC_FPDivD,      [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<31, [E5500_FPU_0]>],
                                 [39, 2, 2], // Latency = 35, Repeat rate = 31
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 347-368

```tablegen
                                 [E5500_FPR_Bypass,
                                  E5500_FPR_Bypass, E5500_FPR_Bypass]>,
  InstrItinData<IIC_FPDivS,      [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<16, [E5500_FPU_0]>],
                                 [24, 2, 2], // Latency = 20, Repeat rate = 16
                                 [E5500_FPR_Bypass,
                                  E5500_FPR_Bypass, E5500_FPR_Bypass]>,
  InstrItinData<IIC_FPFused,     [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<1, [E5500_FPU_0]>],
                                 [11, 2, 2, 2], // Latency = 7, Repeat rate = 1
                                 [E5500_FPR_Bypass,
                                  E5500_FPR_Bypass, E5500_FPR_Bypass,
                                  E5500_FPR_Bypass]>,
  InstrItinData<IIC_FPRes,       [InstrStage<1, [E5500_DIS0, E5500_DIS1], 0>,
                                  InstrStage<2, [E5500_FPU_0]>],
                                 [12, 2], // Latency = 8, Repeat rate = 2
                                 [E5500_FPR_Bypass, E5500_FPR_Bypass]>
]>;

// ===---------------------------------------------------------------------===//
// e5500 machine model for scheduling and other instruction cost heuristics.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 369-376

```tablegen
def PPCE5500Model : SchedMachineModel {
  let IssueWidth = 2;  // 2 micro-ops are dispatched per cycle.
  let LoadLatency = 6; // Optimistic load latency assuming bypass.
                       // This is overriden by OperandCycles if the
                       // Itineraries are queried instead.

  let CompleteModel = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is overriden by OperandCycles if the". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is overriden by OperandCycles if the”。 这一段包含调度或处理器模型元数据。

### Lines 377-378

```tablegen
  let Itineraries = PPCE5500Itineraries;
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
