# PPCSchedule.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCSchedule.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCSchedule.td - PowerPC Scheduling Definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCSchedule.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCSchedule.td - PowerPC Scheduling Definitions ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段包含调度或处理器模型元数据。

### Lines 7-8

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 9-15

```tablegen
//===----------------------------------------------------------------------===//
// Instruction Itinerary classes used for PowerPC
//
def IIC_IntSimple    : InstrItinClass;
def IIC_IntGeneral   : InstrItinClass;
def IIC_IntCompare   : InstrItinClass;
def IIC_IntISEL      : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_IntSimple`, `IIC_IntGeneral`, `IIC_IntCompare` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_IntSimple`, `IIC_IntGeneral`, `IIC_IntCompare`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 16-22

```tablegen
def IIC_IntDivD      : InstrItinClass;
def IIC_IntDivW      : InstrItinClass;
def IIC_IntMFFS      : InstrItinClass;
def IIC_IntMFVSCR    : InstrItinClass;
def IIC_IntMTFSB0    : InstrItinClass;
def IIC_IntMTSRD     : InstrItinClass;
def IIC_IntMulHD     : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_IntDivD`, `IIC_IntDivW`, `IIC_IntMFFS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_IntDivD`, `IIC_IntDivW`, `IIC_IntMFFS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 23-29

```tablegen
def IIC_IntMulHW     : InstrItinClass;
def IIC_IntMulHWU    : InstrItinClass;
def IIC_IntMulLI     : InstrItinClass;
def IIC_IntRFID      : InstrItinClass;
def IIC_IntRotateD   : InstrItinClass;
def IIC_IntRotateDI  : InstrItinClass;
def IIC_IntRotate    : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_IntMulHW`, `IIC_IntMulHWU`, `IIC_IntMulLI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_IntMulHW`, `IIC_IntMulHWU`, `IIC_IntMulLI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 30-36

```tablegen
def IIC_IntShift     : InstrItinClass;
def IIC_IntTrapD     : InstrItinClass;
def IIC_IntTrapW     : InstrItinClass;
def IIC_BrB          : InstrItinClass;
def IIC_BrCR         : InstrItinClass;
def IIC_BrMCR        : InstrItinClass;
def IIC_BrMCRX       : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_IntShift`, `IIC_IntTrapD`, `IIC_IntTrapW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_IntShift`, `IIC_IntTrapD`, `IIC_IntTrapW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 37-43

```tablegen
def IIC_LdStDCBA     : InstrItinClass;
def IIC_LdStDCBF     : InstrItinClass;
def IIC_LdStDCBI     : InstrItinClass;
def IIC_LdStLoad     : InstrItinClass;
def IIC_LdStLoadUpd  : InstrItinClass;
def IIC_LdStLoadUpdX : InstrItinClass;
def IIC_LdStStore    : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_LdStDCBA`, `IIC_LdStDCBF`, `IIC_LdStDCBI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_LdStDCBA`, `IIC_LdStDCBF`, `IIC_LdStDCBI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 44-50

```tablegen
def IIC_LdStDSS      : InstrItinClass;
def IIC_LdStICBI     : InstrItinClass;
def IIC_LdStLD       : InstrItinClass;
def IIC_LdStLDU      : InstrItinClass;
def IIC_LdStLDUX     : InstrItinClass;
def IIC_LdStLDARX    : InstrItinClass;
def IIC_LdStLFD      : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_LdStDSS`, `IIC_LdStICBI`, `IIC_LdStLD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_LdStDSS`, `IIC_LdStICBI`, `IIC_LdStLD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 51-57

```tablegen
def IIC_LdStLFDU     : InstrItinClass;
def IIC_LdStLFDUX    : InstrItinClass;
def IIC_LdStLHA      : InstrItinClass;
def IIC_LdStLHAU     : InstrItinClass;
def IIC_LdStLHAUX    : InstrItinClass;
def IIC_LdStLMW      : InstrItinClass;
def IIC_LdStLQ       : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_LdStLFDU`, `IIC_LdStLFDUX`, `IIC_LdStLHA` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_LdStLFDU`, `IIC_LdStLFDUX`, `IIC_LdStLHA`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 58-64

```tablegen
def IIC_LdStLQARX    : InstrItinClass;
def IIC_LdStLVecX    : InstrItinClass;
def IIC_LdStLWA      : InstrItinClass;
def IIC_LdStLWARX    : InstrItinClass;
def IIC_LdStSLBIA    : InstrItinClass;
def IIC_LdStSLBIE    : InstrItinClass;
def IIC_LdStSTD      : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_LdStLQARX`, `IIC_LdStLVecX`, `IIC_LdStLWA` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_LdStLQARX`, `IIC_LdStLVecX`, `IIC_LdStLWA`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 65-71

```tablegen
def IIC_LdStSTDCX    : InstrItinClass;
def IIC_LdStSTQ      : InstrItinClass;
def IIC_LdStSTQCX    : InstrItinClass;
def IIC_LdStSTU      : InstrItinClass;
def IIC_LdStSTUX     : InstrItinClass;
def IIC_LdStSTFD     : InstrItinClass;
def IIC_LdStSTFDU    : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_LdStSTDCX`, `IIC_LdStSTQ`, `IIC_LdStSTQCX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_LdStSTDCX`, `IIC_LdStSTQ`, `IIC_LdStSTQCX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 72-78

```tablegen
def IIC_LdStSTVEBX   : InstrItinClass;
def IIC_LdStSTWCX    : InstrItinClass;
def IIC_LdStSync     : InstrItinClass;
def IIC_LdStCOPY     : InstrItinClass;
def IIC_LdStPASTE    : InstrItinClass;
def IIC_SprISYNC     : InstrItinClass;
def IIC_SprMFSR      : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_LdStSTVEBX`, `IIC_LdStSTWCX`, `IIC_LdStSync` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_LdStSTVEBX`, `IIC_LdStSTWCX`, `IIC_LdStSync`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 79-85

```tablegen
def IIC_SprMTMSR     : InstrItinClass;
def IIC_SprMTSR      : InstrItinClass;
def IIC_SprTLBSYNC   : InstrItinClass;
def IIC_SprMFCR      : InstrItinClass;
def IIC_SprMFCRF     : InstrItinClass;
def IIC_SprMFMSR     : InstrItinClass;
def IIC_SprMFSPR     : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_SprMTMSR`, `IIC_SprMTSR`, `IIC_SprTLBSYNC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_SprMTMSR`, `IIC_SprMTSR`, `IIC_SprTLBSYNC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 86-92

```tablegen
def IIC_SprMFTB      : InstrItinClass;
def IIC_SprMTSPR     : InstrItinClass;
def IIC_SprMTSRIN    : InstrItinClass;
def IIC_SprRFI       : InstrItinClass;
def IIC_SprSC        : InstrItinClass;
def IIC_FPGeneral    : InstrItinClass;
def IIC_FPDGeneral   : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_SprMFTB`, `IIC_SprMTSPR`, `IIC_SprMTSRIN` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_SprMFTB`, `IIC_SprMTSPR`, `IIC_SprMTSRIN`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 93-99

```tablegen
def IIC_FPSGeneral   : InstrItinClass;
def IIC_FPAddSub     : InstrItinClass;
def IIC_FPCompare    : InstrItinClass;
def IIC_FPDivD       : InstrItinClass;
def IIC_FPDivS       : InstrItinClass;
def IIC_FPFused      : InstrItinClass;
def IIC_FPRes        : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_FPSGeneral`, `IIC_FPAddSub`, `IIC_FPCompare` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_FPSGeneral`, `IIC_FPAddSub`, `IIC_FPCompare`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 100-106

```tablegen
def IIC_FPSqrtD      : InstrItinClass;
def IIC_FPSqrtS      : InstrItinClass;
def IIC_VecGeneral   : InstrItinClass;
def IIC_VecFP        : InstrItinClass;
def IIC_VecFPCompare : InstrItinClass;
def IIC_VecComplex   : InstrItinClass;
def IIC_VecPerm      : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_FPSqrtD`, `IIC_FPSqrtS`, `IIC_VecGeneral` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_FPSqrtD`, `IIC_FPSqrtS`, `IIC_VecGeneral`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 107-113

```tablegen
def IIC_VecFPRound   : InstrItinClass;
def IIC_VecVSL       : InstrItinClass;
def IIC_VecVSR       : InstrItinClass;
def IIC_SprMTMSRD    : InstrItinClass;
def IIC_SprSLIE      : InstrItinClass;
def IIC_SprSLBFEE    : InstrItinClass;
def IIC_SprSLBIE     : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_VecFPRound`, `IIC_VecVSL`, `IIC_VecVSR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_VecFPRound`, `IIC_VecVSL`, `IIC_VecVSR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 114-120

```tablegen
def IIC_SprSLBIEG    : InstrItinClass;
def IIC_SprSLBMTE    : InstrItinClass;
def IIC_SprSLBMFEE   : InstrItinClass;
def IIC_SprSLBMFEV   : InstrItinClass;
def IIC_SprSLBIA     : InstrItinClass;
def IIC_SprSLBSYNC   : InstrItinClass;
def IIC_SprTLBIA     : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_SprSLBIEG`, `IIC_SprSLBMTE`, `IIC_SprSLBMFEE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_SprSLBIEG`, `IIC_SprSLBMTE`, `IIC_SprSLBMFEE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 121-127

```tablegen
def IIC_SprTLBIEL    : InstrItinClass;
def IIC_SprTLBIE     : InstrItinClass;
def IIC_SprABORT     : InstrItinClass;
def IIC_SprMSGSYNC   : InstrItinClass;
def IIC_SprMSGSNDP   : InstrItinClass;
def IIC_SprSTOP      : InstrItinClass;
def IIC_SprMFPMR     : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_SprTLBIEL`, `IIC_SprTLBIE`, `IIC_SprABORT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_SprTLBIEL`, `IIC_SprTLBIE`, `IIC_SprABORT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 128-129

```tablegen
def IIC_SprMTPMR     : InstrItinClass;
```
- **EN**: Adds declarative TableGen records such as `IIC_SprMTPMR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IIC_SprMTPMR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 130-136

```tablegen
//===----------------------------------------------------------------------===//
// Processor instruction itineraries.
include "PPCInstrInfo.td"

include "PPCSchedPredicates.td"
include "PPCScheduleG3.td"
include "PPCSchedule440.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这些声明会进入生成式模式匹配逻辑。

### Lines 137-143

```tablegen
include "PPCScheduleG4.td"
include "PPCScheduleG4Plus.td"
include "PPCScheduleG5.td"
include "PPCScheduleP7.td"
include "PPCScheduleP8.td"
include "PPCScheduleP9.td"
include "PPCScheduleP10.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段包含调度或处理器模型元数据。

### Lines 144-147

```tablegen
include "PPCScheduleA2.td"
include "PPCScheduleE500.td"
include "PPCScheduleE500mc.td"
include "PPCScheduleE5500.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段包含调度或处理器模型元数据。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCInstrInfo.td`
- `PPCSchedPredicates.td`
- `PPCScheduleG3.td`
- `PPCSchedule440.td`
- `PPCScheduleG4.td`
- `PPCScheduleG4Plus.td`
- `PPCScheduleG5.td`
- `PPCScheduleP7.td`
- `PPCScheduleP8.td`
- `PPCScheduleP9.td`
- `PPCScheduleP10.td`
- `PPCScheduleA2.td`
- `PPCScheduleE500.td`
- `PPCScheduleE500mc.td`
- `PPCScheduleE5500.td`

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
