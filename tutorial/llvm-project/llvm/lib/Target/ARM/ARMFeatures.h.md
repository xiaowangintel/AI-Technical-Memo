# ARMFeatures.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMFeatures.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains the code shared between ARM CodeGen and ARM MC.
- 用途 (CN): 声明 ARM 后端中的 `ARMFeatures`，并提供与特性标志与能力判定相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMFeatures.h - Checks for ARM instruction features -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the code shared between ARM CodeGen and ARM MC
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMFEATURES_H
#define LLVM_LIB_TARGET_ARM_ARMFEATURES_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-16
```cpp
#include "MCTargetDesc/ARMMCTargetDesc.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 18-18
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 20-21
```cpp
template<typename InstrType> // could be MachineInstr or MCInst
bool IsCPSRDead(const InstrType *Instr);
```
- EN: Declares `IsCPSRDead`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `IsCPSRDead`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 23-37
```cpp
template<typename InstrType> // could be MachineInstr or MCInst
inline bool isV8EligibleForIT(const InstrType *Instr) {
  switch (Instr->getOpcode()) {
  default:
    return false;
  case ARM::tADC:
  case ARM::tADDi3:
  case ARM::tADDi8:
  case ARM::tADDrr:
  case ARM::tAND:
  case ARM::tASRri:
  case ARM::tASRrr:
  case ARM::tBIC:
  case ARM::tEOR:
  case ARM::tLSLri:
```
- EN: Implements `isV8EligibleForIT`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isV8EligibleForIT`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-52
```cpp
  case ARM::tLSLrr:
  case ARM::tLSRri:
  case ARM::tLSRrr:
  case ARM::tMOVi8:
  case ARM::tMUL:
  case ARM::tMVN:
  case ARM::tORR:
  case ARM::tROR:
  case ARM::tRSB:
  case ARM::tSBC:
  case ARM::tSUBi3:
  case ARM::tSUBi8:
  case ARM::tSUBrr:
    // Outside of an IT block, these set CPSR.
    return IsCPSRDead(Instr);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 53-67
```cpp
  case ARM::tADDrSPi:
  case ARM::tCMN:
  case ARM::tCMPi8:
  case ARM::tCMPr:
  case ARM::tLDRBi:
  case ARM::tLDRBr:
  case ARM::tLDRHi:
  case ARM::tLDRHr:
  case ARM::tLDRSB:
  case ARM::tLDRSH:
  case ARM::tLDRi:
  case ARM::tLDRr:
  case ARM::tLDRspi:
  case ARM::tSTRBi:
  case ARM::tSTRBr:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 68-82
```cpp
  case ARM::tSTRHi:
  case ARM::tSTRHr:
  case ARM::tSTRi:
  case ARM::tSTRr:
  case ARM::tSTRspi:
  case ARM::tTST:
    return true;
// there are some "conditionally deprecated" opcodes
  case ARM::tADDspr:
  case ARM::tBLXr:
  case ARM::tBLXr_noip:
    return Instr->getOperand(2).getReg() != ARM::PC;
  // ADD PC, SP and BLX PC were always unpredictable,
  // now on top of it they're deprecated
  case ARM::tADDrSP:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 83-93
```cpp
  case ARM::tBX:
    return Instr->getOperand(0).getReg() != ARM::PC;
  case ARM::tADDhirr:
    return Instr->getOperand(0).getReg() != ARM::PC &&
           Instr->getOperand(2).getReg() != ARM::PC;
  case ARM::tCMPhir:
  case ARM::tMOVr:
    return Instr->getOperand(0).getReg() != ARM::PC &&
           Instr->getOperand(1).getReg() != ARM::PC;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-95
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 97-97
```cpp
#endif
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: feature flags and capability predicates.
  - CN: 核心职责：特性标志与能力判定。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMMCTargetDesc.h`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMMCTargetDesc.h`。
- EN: LLVM infrastructure headers: `(none)`.
  - CN: LLVM 基础设施头文件：`(none)`。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
