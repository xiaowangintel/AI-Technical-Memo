# PPCScheduleE500mc.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCScheduleE500mc.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCScheduleE500mc.td - e500mc Scheduling Defs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCScheduleE500mc.td`，主要负责 PowerPC 后端的该后端的 MC 层支持。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCScheduleE500mc.td - e500mc Scheduling Defs ------*- tablegen -*-===//
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
// This file defines the itinerary class data for the Freescale e500mc 32-bit
// Power processor.
//
// All information is derived from the "e500mc Core Reference Manual",
// Freescale Document Number E500MCRM, Rev. 1, 03/2012.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines the itinerary class data for the Freescale e500mc 32-bit".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines the itinerary class data for the Freescale e500mc 32-bit”。

### Lines 15-21

```tablegen
//===----------------------------------------------------------------------===//
// Relevant functional units in the Freescale e500mc core:
//
//  * Decode & Dispatch
//    Can dispatch up to 2 instructions per clock cycle to either the GPR Issue
//    queues (GIQx), FP Issue Queue (FIQ), or Branch issue queue (BIQ).
def E500mc_DIS0 : FuncUnit; // Dispatch stage - insn 1
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relevant functional units in the Freescale e500mc core:". Notable symbols in this range include `E500mc_DIS0`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relevant functional units in the Freescale e500mc core:”。 该区间中较显眼的符号包括 `E500mc_DIS0`。

### Lines 22-28

```tablegen
def E500mc_DIS1 : FuncUnit; // Dispatch stage - insn 2

//  * Execute
//    6 pipelined execution units: SFX0, SFX1, BU, FPU, LSU, CFX.
//    Some instructions can only execute in SFX0 but not SFX1.
//    The CFX has a bypass path, allowing non-divide instructions to execute
//    while a divide instruction is executed.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "* Execute". Notable symbols in this range include `E500mc_DIS1`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“* Execute”。 该区间中较显眼的符号包括 `E500mc_DIS1`。

### Lines 29-35

```tablegen
def E500mc_SFX0  : FuncUnit; // Simple unit 0
def E500mc_SFX1  : FuncUnit; // Simple unit 1
def E500mc_BU    : FuncUnit; // Branch unit
def E500mc_CFX_DivBypass
               : FuncUnit; // CFX divide bypass path
def E500mc_CFX_0 : FuncUnit; // CFX pipeline
def E500mc_LSU_0 : FuncUnit; // LSU pipeline
```
- **EN**: Adds declarative TableGen records such as `E500mc_SFX0`, `E500mc_SFX1`, `E500mc_BU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `E500mc_SFX0`, `E500mc_SFX1`, `E500mc_BU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 36-73

```tablegen
def E500mc_FPU_0 : FuncUnit; // FPU pipeline

def E500mc_GPR_Bypass : Bypass;
def E500mc_FPR_Bypass : Bypass;
def E500mc_CR_Bypass  : Bypass;

def PPCE500mcItineraries : ProcessorItineraries<
  [E500mc_DIS0, E500mc_DIS1, E500mc_SFX0, E500mc_SFX1, E500mc_BU, E500mc_CFX_DivBypass,
   E500mc_CFX_0, E500mc_LSU_0, E500mc_FPU_0],
  [E500mc_CR_Bypass, E500mc_GPR_Bypass, E500mc_FPR_Bypass], [
  InstrItinData<IIC_IntSimple,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntGeneral,  [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntISEL,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1, 1, 1], // Latency = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass,
                                  E500mc_CR_Bypass]>,
  InstrItinData<IIC_IntCompare,  [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [5, 1, 1], // Latency = 1 or 2
                                 [E500mc_CR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntDivW,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_CFX_0], 0>,
                                  InstrStage<14, [E500mc_CFX_DivBypass]>],
                                 [17, 1, 1], // Latency=4..35, Repeat= 4..35
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntMFFS,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
```
- **EN**: Adds declarative TableGen records such as `E500mc_FPU_0`, `E500mc_GPR_Bypass`, `E500mc_FPR_Bypass` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `E500mc_FPU_0`, `E500mc_GPR_Bypass`, `E500mc_FPR_Bypass`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 74-111

```tablegen
                                  InstrStage<8, [E500mc_FPU_0]>],
                                 [11], // Latency = 8
                                 [E500mc_FPR_Bypass]>,
  InstrItinData<IIC_IntMTFSB0,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<8, [E500mc_FPU_0]>],
                                 [11, 1, 1], // Latency = 8
                                 [NoBypass, NoBypass, NoBypass]>,
  InstrItinData<IIC_IntMulHW,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_CFX_0]>],
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHWU,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_CFX_0]>],
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntMulLI,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_CFX_0]>],
                                 [7, 1, 1], // Latency = 4, Repeat rate = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntRotate,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntShift,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_IntTrapW,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<2, [E500mc_SFX0]>],
                                 [5, 1], // Latency = 2, Repeat rate = 2
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_BrB,         [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_BU]>],
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 112-149

```tablegen
                                 [4, 1], // Latency = 1
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_BrCR,        [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_BU]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500mc_CR_Bypass,
                                  E500mc_CR_Bypass, E500mc_CR_Bypass]>,
  InstrItinData<IIC_BrMCR,       [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_BU]>],
                                 [4, 1], // Latency = 1
                                 [E500mc_CR_Bypass, E500mc_CR_Bypass]>,
  InstrItinData<IIC_BrMCRX,      [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1, 1], // Latency = 1
                                 [E500mc_CR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBA,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3, Repeat rate = 1
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBF,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBI,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoad,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoadUpd, [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLoadUpdX,[InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 150-187

```tablegen
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStStore,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTU,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStSTUX,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStICBI,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTFD,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1, 1], // Latency = 3
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTFDU,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1, 1], // Latency = 3
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLFD,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 188-225

```tablegen
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [7, 1, 1], // Latency = 4
                                 [E500mc_FPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLFDU,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [7, 1, 1], // Latency = 4
                                 [E500mc_FPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLFDUX,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [7, 1, 1], // Latency = 4
                                 [E500mc_FPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass],
                                 2>, // 2 micro-ops
  InstrItinData<IIC_LdStLHA,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAU,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAUX,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLMW,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [7, 1], // Latency = r+3
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStLWARX,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<3, [E500mc_LSU_0]>],
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 226-263

```tablegen
                                 [6, 1, 1], // Latency = 3, Repeat rate = 3
                                 [E500mc_GPR_Bypass,
                                  E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTWCX,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>],
                                 [6, 1], // Latency = 3
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_LdStSync,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0]>]>,
  InstrItinData<IIC_SprMFSR,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<4, [E500mc_SFX0]>],
                                 [7, 1],
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMTMSR,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<2, [E500mc_SFX0, E500mc_SFX1]>],
                                 [5, 1], // Latency = 2, Repeat rate = 4
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSR,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0]>],
                                 [5, 1],
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprTLBSYNC,  [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_LSU_0], 0>]>,
  InstrItinData<IIC_SprMFCR,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<5, [E500mc_SFX0]>],
                                 [8, 1],
                                 [E500mc_GPR_Bypass, E500mc_CR_Bypass]>,
  InstrItinData<IIC_SprMFCRF,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<5, [E500mc_SFX0]>],
                                 [8, 1],
                                 [E500mc_GPR_Bypass, E500mc_CR_Bypass]>,
  InstrItinData<IIC_SprMFPMR,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<4, [E500mc_SFX0]>],
                                 [7, 1], // Latency = 4, Repeat rate = 4
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMFMSR,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<4, [E500mc_SFX0]>],
                                 [7, 1], // Latency = 4, Repeat rate = 4
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 264-301

```tablegen
                                 [E500mc_GPR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMFSPR,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1], // Latency = 1, Repeat rate = 1
                                 [E500mc_GPR_Bypass, E500mc_CR_Bypass]>,
  InstrItinData<IIC_SprMTPMR,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0]>],
                                 [4, 1], // Latency = 1, Repeat rate = 1
                                 [E500mc_CR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMFTB,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<4, [E500mc_SFX0]>],
                                 [7, 1], // Latency = 4, Repeat rate = 4
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSPR,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0, E500mc_SFX1]>],
                                 [4, 1], // Latency = 1, Repeat rate = 1
                                 [E500mc_CR_Bypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSRIN,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<1, [E500mc_SFX0]>],
                                 [4, 1],
                                 [NoBypass, E500mc_GPR_Bypass]>,
  InstrItinData<IIC_FPGeneral,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<2, [E500mc_FPU_0]>],
                                 [11, 1, 1], // Latency = 8, Repeat rate = 2
                                 [E500mc_FPR_Bypass,
                                  E500mc_FPR_Bypass, E500mc_FPR_Bypass]>,
  InstrItinData<IIC_FPAddSub,    [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<4, [E500mc_FPU_0]>],
                                 [13, 1, 1], // Latency = 10, Repeat rate = 4
                                 [E500mc_FPR_Bypass,
                                  E500mc_FPR_Bypass, E500mc_FPR_Bypass]>,
  InstrItinData<IIC_FPCompare,   [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<2, [E500mc_FPU_0]>],
                                 [11, 1, 1], // Latency = 8, Repeat rate = 2
                                 [E500mc_CR_Bypass,
                                  E500mc_FPR_Bypass, E500mc_FPR_Bypass]>,
  InstrItinData<IIC_FPDivD,      [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<68, [E500mc_FPU_0]>],
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 302-324

```tablegen
                                 [71, 1, 1], // Latency = 68, Repeat rate = 68
                                 [E500mc_FPR_Bypass,
                                  E500mc_FPR_Bypass, E500mc_FPR_Bypass]>,
  InstrItinData<IIC_FPDivS,      [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<38, [E500mc_FPU_0]>],
                                 [41, 1, 1], // Latency = 38, Repeat rate = 38
                                 [E500mc_FPR_Bypass,
                                  E500mc_FPR_Bypass, E500mc_FPR_Bypass]>,
  InstrItinData<IIC_FPFused,     [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<4, [E500mc_FPU_0]>],
                                 [13, 1, 1, 1], // Latency = 10, Repeat rate = 4
                                 [E500mc_FPR_Bypass,
                                  E500mc_FPR_Bypass, E500mc_FPR_Bypass,
                                  E500mc_FPR_Bypass]>,
  InstrItinData<IIC_FPRes,       [InstrStage<1, [E500mc_DIS0, E500mc_DIS1], 0>,
                                  InstrStage<38, [E500mc_FPU_0]>],
                                 [41, 1], // Latency = 38, Repeat rate = 38
                                 [E500mc_FPR_Bypass, E500mc_FPR_Bypass]>
]>;

// ===---------------------------------------------------------------------===//
// e500mc machine model for scheduling and other instruction cost heuristics.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 325-332

```tablegen
def PPCE500mcModel : SchedMachineModel {
  let IssueWidth = 2;  // 2 micro-ops are dispatched per cycle.
  let LoadLatency = 5; // Optimistic load latency assuming bypass.
                       // This is overriden by OperandCycles if the
                       // Itineraries are queried instead.

  let CompleteModel = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is overriden by OperandCycles if the". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is overriden by OperandCycles if the”。 这一段包含调度或处理器模型元数据。

### Lines 333-334

```tablegen
  let Itineraries = PPCE500mcItineraries;
}
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
