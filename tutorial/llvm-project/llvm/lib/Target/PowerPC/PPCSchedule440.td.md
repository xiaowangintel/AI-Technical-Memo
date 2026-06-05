# PPCSchedule440.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCSchedule440.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCSchedule440.td - PPC 440 Scheduling Definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCSchedule440.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCSchedule440.td - PPC 440 Scheduling Definitions -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段包含调度或处理器模型元数据。

### Lines 7-24

```tablegen
//===----------------------------------------------------------------------===//

// Primary reference:
// PowerPC 440x6 Embedded Processor Core User's Manual.
// IBM (as updated in) 2010.

// The basic PPC 440 does not include a floating-point unit; the pipeline
// timings here are constructed to match the FP2 unit shipped with the
// PPC-440- and PPC-450-based Blue Gene (L and P) supercomputers.
// References:
// S. Chatterjee, et al. Design and exploitation of a high-performance
// SIMD floating-point unit for Blue Gene/L.
// IBM J. Res. & Dev. 49 (2/3) March/May 2005.
// also:
// Carlos Sosa and Brant Knudson. IBM System Blue Gene Solution:
// Blue Gene/P Application Development.
// IBM (as updated in) 2009.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Primary reference:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Primary reference:”。

### Lines 25-32

```tablegen
//===----------------------------------------------------------------------===//
// Functional units on the PowerPC 440/450 chip sets
//
def P440_DISS1  : FuncUnit; // Issue unit 1
def P440_DISS2  : FuncUnit; // Issue unit 2
def P440_LRACC  : FuncUnit; // Register access and dispatch for
                            // the simple integer (J-pipe) and
                            // load/store (L-pipe) pipelines
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Functional units on the PowerPC 440/450 chip sets". Notable symbols in this range include `P440_DISS1`, `P440_DISS2`, `P440_LRACC`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Functional units on the PowerPC 440/450 chip sets”。 该区间中较显眼的符号包括 `P440_DISS1`, `P440_DISS2`, `P440_LRACC`。

### Lines 33-39

```tablegen
def P440_IRACC  : FuncUnit; // Register access and dispatch for
                            // the complex integer (I-pipe) pipeline
def P440_FRACC  : FuncUnit; // Register access and dispatch for
                            // the floating-point execution (F-pipe) pipeline
def P440_IEXE1  : FuncUnit; // Execution stage 1 for the I pipeline
def P440_IEXE2  : FuncUnit; // Execution stage 2 for the I pipeline
def P440_IWB    : FuncUnit; // Write-back unit for the I pipeline
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "the complex integer (I-pipe) pipeline". Notable symbols in this range include `P440_IRACC`, `P440_FRACC`, `P440_IEXE1`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“the complex integer (I-pipe) pipeline”。 该区间中较显眼的符号包括 `P440_IRACC`, `P440_FRACC`, `P440_IEXE1`。

### Lines 40-46

```tablegen
def P440_JEXE1  : FuncUnit; // Execution stage 1 for the J pipeline
def P440_JEXE2  : FuncUnit; // Execution stage 2 for the J pipeline
def P440_JWB    : FuncUnit; // Write-back unit for the J pipeline
def P440_AGEN   : FuncUnit; // Address generation for the L pipeline
def P440_CRD    : FuncUnit; // D-cache access for the L pipeline
def P440_LWB    : FuncUnit; // Write-back unit for the L pipeline
def P440_FEXE1  : FuncUnit; // Execution stage 1 for the F pipeline
```
- **EN**: Adds declarative TableGen records such as `P440_JEXE1`, `P440_JEXE2`, `P440_JWB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `P440_JEXE1`, `P440_JEXE2`, `P440_JWB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 47-53

```tablegen
def P440_FEXE2  : FuncUnit; // Execution stage 2 for the F pipeline
def P440_FEXE3  : FuncUnit; // Execution stage 3 for the F pipeline
def P440_FEXE4  : FuncUnit; // Execution stage 4 for the F pipeline
def P440_FEXE5  : FuncUnit; // Execution stage 5 for the F pipeline
def P440_FEXE6  : FuncUnit; // Execution stage 6 for the F pipeline
def P440_FWB    : FuncUnit; // Write-back unit for the F pipeline
```
- **EN**: Adds declarative TableGen records such as `P440_FEXE2`, `P440_FEXE3`, `P440_FEXE4` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `P440_FEXE2`, `P440_FEXE3`, `P440_FEXE4`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 54-91

```tablegen
def P440_LWARX_Hold : FuncUnit; // This is a pseudo-unit which is used
                                // to make sure that no lwarx/stwcx.
                                // instructions are issued while another
                                // lwarx/stwcx. is in the L pipe.

def P440_GPR_Bypass : Bypass; // The bypass for general-purpose regs.
def P440_FPR_Bypass : Bypass; // The bypass for floating-point regs.

// Notes:
// Instructions are held in the FRACC, LRACC and IRACC pipeline
// stages until their source operands become ready. Exceptions:
//  - Store instructions will hold in the AGEN stage
//  - The integer multiply-accumulate instruction will hold in
//    the IEXE1 stage
//
// For most I-pipe operations, the result is available at the end of
// the IEXE1 stage. Operations such as multiply and divide must
// continue to execute in IEXE2 and IWB. Divide resides in IWB for
// 33 cycles (multiply also calculates its result in IWB). For all
// J-pipe instructions, the result is available
// at the end of the JEXE1 stage. Loads have a 3-cycle latency
// (data is not available until after the LWB stage).
//
// The L1 cache hit latency is four cycles for floating point loads
// and three cycles for integer loads.
//
// The stwcx. instruction requires both the LRACC and the IRACC
// dispatch stages. It must be issued from DISS0.
//
// All lwarx/stwcx. instructions hold in LRACC if another
// uncommitted lwarx/stwcx. is in AGEN, CRD, or LWB.
//
// msync (a.k.a. sync) and mbar will hold in LWB until all load/store
// resources are empty. AGEN and CRD are held empty until the msync/mbar
// commits.
//
// Most floating-point instructions, computational and move,
// have a 5-cycle latency. Divide takes longer (30 cycles). Instructions that
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "to make sure that no lwarx/stwcx.". Notable symbols in this range include `P440_LWARX_Hold`, `P440_GPR_Bypass`, `P440_FPR_Bypass`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“to make sure that no lwarx/stwcx.”。 该区间中较显眼的符号包括 `P440_LWARX_Hold`, `P440_GPR_Bypass`, `P440_FPR_Bypass`。

### Lines 92-97

```tablegen
// update the CR take 2 cycles. Stores take 3 cycles and, as mentioned above,
// loads take 4 cycles (for L1 hit).

//
// This file defines the itinerary class data for the PPC 440 processor.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "update the CR take 2 cycles. Stores take 3 cycles and, as mentioned above,".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“update the CR take 2 cycles. Stores take 3 cycles and, as mentioned above,”。

### Lines 98-135

```tablegen
//===----------------------------------------------------------------------===//


def PPC440Itineraries : ProcessorItineraries<
  [P440_DISS1, P440_DISS2, P440_FRACC, P440_IRACC, P440_IEXE1, P440_IEXE2,
   P440_IWB, P440_LRACC, P440_JEXE1, P440_JEXE2, P440_JWB, P440_AGEN, P440_CRD,
   P440_LWB, P440_FEXE1, P440_FEXE2, P440_FEXE3, P440_FEXE4, P440_FEXE5,
   P440_FEXE6, P440_FWB, P440_LWARX_Hold],
  [P440_GPR_Bypass, P440_FPR_Bypass], [
  InstrItinData<IIC_IntSimple,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC, P440_LRACC]>,
                                 InstrStage<1, [P440_IEXE1, P440_JEXE1]>,
                                 InstrStage<1, [P440_IEXE2, P440_JEXE2]>,
                                 InstrStage<1, [P440_IWB, P440_JWB]>],
                                [2, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntGeneral, [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC, P440_LRACC]>,
                                 InstrStage<1, [P440_IEXE1, P440_JEXE1]>,
                                 InstrStage<1, [P440_IEXE2, P440_JEXE2]>,
                                 InstrStage<1, [P440_IWB, P440_JWB]>],
                                [2, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntISEL,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC, P440_LRACC]>,
                                 InstrStage<1, [P440_IEXE1, P440_JEXE1]>,
                                 InstrStage<1, [P440_IEXE2, P440_JEXE2]>,
                                 InstrStage<1, [P440_IWB, P440_JWB]>],
                                [2, 0, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass, NoBypass]>,
  InstrItinData<IIC_IntCompare, [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC, P440_LRACC]>,
                                 InstrStage<1, [P440_IEXE1, P440_JEXE1]>,
                                 InstrStage<1, [P440_IEXE2, P440_JEXE2]>,
                                 InstrStage<1, [P440_IWB, P440_JWB]>],
```
- **EN**: Adds declarative TableGen records such as `PPC440Itineraries` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPC440Itineraries`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 136-173

```tablegen
                                [2, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntDivW,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<33, [P440_IWB]>],
                                [36, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntMFFS,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [3, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntMTFSB0,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [3, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHW,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntMulHWU,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0, 0],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 174-211

```tablegen
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntMulLI,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntRotate,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC, P440_LRACC]>,
                                 InstrStage<1, [P440_IEXE1, P440_JEXE1]>,
                                 InstrStage<1, [P440_IEXE2, P440_JEXE2]>,
                                 InstrStage<1, [P440_IWB, P440_JWB]>],
                                [2, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntShift,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC, P440_LRACC]>,
                                 InstrStage<1, [P440_IEXE1, P440_JEXE1]>,
                                 InstrStage<1, [P440_IEXE2, P440_JEXE2]>,
                                 InstrStage<1, [P440_IWB, P440_JWB]>],
                                [2, 0, 0],
                                [P440_GPR_Bypass,
                                 P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_IntTrapW,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [2, 0],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_BrB,        [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0],
                                [NoBypass, P440_GPR_Bypass]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 212-249

```tablegen
  InstrItinData<IIC_BrCR,       [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_BrMCR,      [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_BrMCRX,     [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0, 0],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBA,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBF,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStDCBI,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 250-287

```tablegen
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoad,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [5, 1, 1],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoadUpd,[InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [5, 2, 1, 1],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLoadUpdX,[InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [5, 2, 1, 1],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStStore,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [1, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStICBI,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 288-325

```tablegen
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTFD,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [1, 1, 1],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTFDU,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [2, 1, 1, 1],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLFD,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [5, 1, 1],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLFDU,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [5, 2, 1, 1],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLFDUX,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [5, 2, 1, 1],
                                [NoBypass, P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHA,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 326-363

```tablegen
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAU,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLHAUX,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLMW,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStLWARX,  [InstrStage<1, [P440_DISS1]>,
                                 InstrStage<1, [P440_IRACC], 0>,
                                 InstrStage<4, [P440_LWARX_Hold], 0>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTD,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 364-401

```tablegen
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTU,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [2, 1, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTUX,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<2, [P440_LWB]>],
                                [2, 1, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTDCX,  [InstrStage<1, [P440_DISS1]>,
                                 InstrStage<1, [P440_IRACC], 0>,
                                 InstrStage<4, [P440_LWARX_Hold], 0>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSTWCX,  [InstrStage<1, [P440_DISS1]>,
                                 InstrStage<1, [P440_IRACC], 0>,
                                 InstrStage<4, [P440_LWARX_Hold], 0>,
                                 InstrStage<1, [P440_LRACC]>,
                                 InstrStage<1, [P440_AGEN]>,
                                 InstrStage<1, [P440_CRD]>,
                                 InstrStage<1, [P440_LWB]>],
                                [4, 1, 1],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_LdStSync,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_LRACC]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 402-439

```tablegen
                                 InstrStage<3, [P440_AGEN], 1>,
                                 InstrStage<2, [P440_CRD],  1>,
                                 InstrStage<1, [P440_LWB]>]>,
  InstrItinData<IIC_SprISYNC,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC], 0>,
                                 InstrStage<1, [P440_LRACC], 0>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_FEXE1], 0>,
                                 InstrStage<1, [P440_AGEN],  0>,
                                 InstrStage<1, [P440_JEXE1], 0>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_FEXE2], 0>,
                                 InstrStage<1, [P440_CRD],   0>,
                                 InstrStage<1, [P440_JEXE2], 0>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<6, [P440_FEXE3], 0>,
                                 InstrStage<6, [P440_LWB],   0>,
                                 InstrStage<6, [P440_JWB],   0>,
                                 InstrStage<6, [P440_IWB]>]>,
  InstrItinData<IIC_SprMFSR,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [2, 0],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMTMSR,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [2, 0],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSR,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<3, [P440_IWB]>],
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 440-477

```tablegen
                                [5, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprTLBSYNC, [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>]>,
  InstrItinData<IIC_SprMFCR,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMFMSR,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [3, 0],
                                [P440_GPR_Bypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMFSPR,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<3, [P440_IWB]>],
                                [6, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMFTB,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<3, [P440_IWB]>],
                                [6, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSPR,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 478-515

```tablegen
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<3, [P440_IWB]>],
                                [6, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprMTSRIN,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<3, [P440_IWB]>],
                                [6, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprRFI,     [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_SprSC,      [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_IRACC]>,
                                 InstrStage<1, [P440_IEXE1]>,
                                 InstrStage<1, [P440_IEXE2]>,
                                 InstrStage<1, [P440_IWB]>],
                                [4, 0],
                                [NoBypass, P440_GPR_Bypass]>,
  InstrItinData<IIC_FPGeneral,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<1, [P440_FWB]>],
                                [6, 0, 0],
                                [P440_FPR_Bypass,
                                 P440_FPR_Bypass, P440_FPR_Bypass]>,
  InstrItinData<IIC_FPAddSub,   [InstrStage<1, [P440_DISS1, P440_DISS2]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 516-553

```tablegen
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<1, [P440_FWB]>],
                                [6, 0, 0],
                                [P440_FPR_Bypass,
                                 P440_FPR_Bypass, P440_FPR_Bypass]>,
  InstrItinData<IIC_FPCompare,  [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<1, [P440_FWB]>],
                                [6, 0, 0],
                                [P440_FPR_Bypass, P440_FPR_Bypass,
                                 P440_FPR_Bypass]>,
  InstrItinData<IIC_FPDivD,     [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<25, [P440_FWB]>],
                                [31, 0, 0],
                                [NoBypass, P440_FPR_Bypass, P440_FPR_Bypass]>,
  InstrItinData<IIC_FPDivS,     [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 554-589

```tablegen
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<13, [P440_FWB]>],
                                [19, 0, 0],
                                [NoBypass, P440_FPR_Bypass, P440_FPR_Bypass]>,
  InstrItinData<IIC_FPFused,    [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<1, [P440_FWB]>],
                                [6, 0, 0, 0],
                                [P440_FPR_Bypass,
                                 P440_FPR_Bypass, P440_FPR_Bypass,
                                 P440_FPR_Bypass]>,
  InstrItinData<IIC_FPRes,      [InstrStage<1, [P440_DISS1, P440_DISS2]>,
                                 InstrStage<1, [P440_FRACC]>,
                                 InstrStage<1, [P440_FEXE1]>,
                                 InstrStage<1, [P440_FEXE2]>,
                                 InstrStage<1, [P440_FEXE3]>,
                                 InstrStage<1, [P440_FEXE4]>,
                                 InstrStage<1, [P440_FEXE5]>,
                                 InstrStage<1, [P440_FEXE6]>,
                                 InstrStage<1, [P440_FWB]>],
                                [6, 0],
                                [P440_FPR_Bypass, P440_FPR_Bypass]>
]>;

// ===---------------------------------------------------------------------===//
// PPC440 machine model for scheduling and other instruction cost heuristics.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 590-597

```tablegen
def PPC440Model : SchedMachineModel {
  let IssueWidth = 2;  // 2 instructions are dispatched per cycle.
  let LoadLatency = 5; // Optimistic load latency assuming bypass.
                       // This is overriden by OperandCycles if the
                       // Itineraries are queried instead.

  let CompleteModel = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is overriden by OperandCycles if the". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is overriden by OperandCycles if the”。 这一段包含调度或处理器模型元数据。

### Lines 598-600

```tablegen
  let Itineraries = PPC440Itineraries;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
