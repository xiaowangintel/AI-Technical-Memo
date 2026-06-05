# MVETailPredUtils.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MVETailPredUtils.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains utility functions for low overhead and tail predicated loops, shared between the ARMLowOverheadLoops pass and anywhere else that needs them.
- 用途 (CN): 声明 ARM 后端中的 `MVETailPredUtils`，并提供与接口与目标相关声明相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//===-- MVETailPredUtils.h - Tail predication utility functions -*- C++-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains utility functions for low overhead and tail predicated
// loops, shared between the ARMLowOverheadLoops pass and anywhere else that
// needs them.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 15-16
```cpp
#ifndef LLVM_LIB_TARGET_ARM_MVETAILPREDUTILS_H
#define LLVM_LIB_TARGET_ARM_MVETAILPREDUTILS_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 18-21
```cpp
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 25-39
```cpp
static inline unsigned VCTPOpcodeToLSTP(unsigned Opcode, bool IsDoLoop) {
  switch (Opcode) {
  default:
    llvm_unreachable("unhandled vctp opcode");
    break;
  case ARM::MVE_VCTP8:
    return IsDoLoop ? ARM::MVE_DLSTP_8 : ARM::MVE_WLSTP_8;
  case ARM::MVE_VCTP16:
    return IsDoLoop ? ARM::MVE_DLSTP_16 : ARM::MVE_WLSTP_16;
  case ARM::MVE_VCTP32:
    return IsDoLoop ? ARM::MVE_DLSTP_32 : ARM::MVE_WLSTP_32;
  case ARM::MVE_VCTP64:
    return IsDoLoop ? ARM::MVE_DLSTP_64 : ARM::MVE_WLSTP_64;
  }
  return 0;
```
- EN: Implements `VCTPOpcodeToLSTP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `VCTPOpcodeToLSTP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 40-40
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 42-56
```cpp
static inline unsigned getTailPredVectorWidth(unsigned Opcode) {
  switch (Opcode) {
  default:
    llvm_unreachable("unhandled vctp opcode");
  case ARM::MVE_VCTP8:
    return 16;
  case ARM::MVE_VCTP16:
    return 8;
  case ARM::MVE_VCTP32:
    return 4;
  case ARM::MVE_VCTP64:
    return 2;
  }
  return 0;
}
```
- EN: Implements `getTailPredVectorWidth`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getTailPredVectorWidth`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 58-69
```cpp
static inline bool isVCTP(const MachineInstr *MI) {
  switch (MI->getOpcode()) {
  default:
    break;
  case ARM::MVE_VCTP8:
  case ARM::MVE_VCTP16:
  case ARM::MVE_VCTP32:
  case ARM::MVE_VCTP64:
    return true;
  }
  return false;
}
```
- EN: Implements `isVCTP`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isVCTP`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-74
```cpp
static inline bool isDoLoopStart(const MachineInstr &MI) {
  return MI.getOpcode() == ARM::t2DoLoopStart ||
         MI.getOpcode() == ARM::t2DoLoopStartTP;
}
```
- EN: Implements `isDoLoopStart`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isDoLoopStart`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 76-80
```cpp
static inline bool isWhileLoopStart(const MachineInstr &MI) {
  return MI.getOpcode() == ARM::t2WhileLoopStart ||
         MI.getOpcode() == ARM::t2WhileLoopStartLR ||
         MI.getOpcode() == ARM::t2WhileLoopStartTP;
}
```
- EN: Implements `isWhileLoopStart`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isWhileLoopStart`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-84
```cpp
static inline bool isLoopStart(const MachineInstr &MI) {
  return isDoLoopStart(MI) || isWhileLoopStart(MI);
}
```
- EN: Implements `isLoopStart`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLoopStart`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-91
```cpp
// Return the TargetBB stored in a t2WhileLoopStartLR/t2WhileLoopStartTP.
inline MachineBasicBlock *getWhileLoopStartTargetBB(const MachineInstr &MI) {
  assert(isWhileLoopStart(MI) && "Expected WhileLoopStart!");
  unsigned Op = MI.getOpcode() == ARM::t2WhileLoopStartTP ? 3 : 2;
  return MI.getOperand(Op).getMBB();
}
```
- EN: Implements `getWhileLoopStartTargetBB`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getWhileLoopStartTargetBB`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-104
```cpp
// WhileLoopStart holds the exit block, so produce a subs Op0, Op1, 0 and then a
// beq that branches to the exit branch.
// If UseCmp is true, this will create a t2CMP instead of a t2SUBri, meaning the
// value of LR into the loop will not be setup. This is used if the LR setup is
// done via another means (via a t2DoLoopStart, for example).
inline void RevertWhileLoopStartLR(MachineInstr *MI, const TargetInstrInfo *TII,
                                   unsigned BrOpc = ARM::t2Bcc,
                                   bool UseCmp = false) {
  MachineBasicBlock *MBB = MI->getParent();
  assert((MI->getOpcode() == ARM::t2WhileLoopStartLR ||
          MI->getOpcode() == ARM::t2WhileLoopStartTP) &&
         "Only expected a t2WhileLoopStartLR/TP in RevertWhileLoopStartLR!");
```
- EN: Implements `RevertWhileLoopStartLR`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RevertWhileLoopStartLR`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-120
```cpp
  // Subs/Cmp
  if (UseCmp) {
    MachineInstrBuilder MIB =
        BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(ARM::t2CMPri));
    MIB.add(MI->getOperand(1));
    MIB.addImm(0);
    MIB.addImm(ARMCC::AL);
    MIB.addReg(ARM::NoRegister);
  } else {
    MachineInstrBuilder MIB =
        BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(ARM::t2SUBri));
    MIB.add(MI->getOperand(0));
    MIB.add(MI->getOperand(1));
    MIB.addImm(0);
    MIB.addImm(ARMCC::AL);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 121-123
```cpp
    MIB.addReg(ARM::NoRegister);
    MIB.addReg(ARM::CPSR, RegState::Define);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-130
```cpp
  // Branch
  MachineInstrBuilder MIB =
      BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(BrOpc));
  MIB.addMBB(getWhileLoopStartTargetBB(*MI)); // branch target
  MIB.addImm(ARMCC::EQ);                      // condition code
  MIB.addReg(ARM::CPSR);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-133
```cpp
  MI->eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 135-140
```cpp
inline void RevertDoLoopStart(MachineInstr *MI, const TargetInstrInfo *TII) {
  MachineBasicBlock *MBB = MI->getParent();
  BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(ARM::tMOVr))
      .add(MI->getOperand(0))
      .add(MI->getOperand(1))
      .add(predOps(ARMCC::AL));
```
- EN: Implements `RevertDoLoopStart`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RevertDoLoopStart`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 142-143
```cpp
  MI->eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 145-147
```cpp
inline void RevertLoopDec(MachineInstr *MI, const TargetInstrInfo *TII,
                          bool SetFlags = false) {
  MachineBasicBlock *MBB = MI->getParent();
```
- EN: Implements `RevertLoopDec`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RevertLoopDec`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 149-155
```cpp
  MachineInstrBuilder MIB =
      BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(ARM::t2SUBri));
  MIB.add(MI->getOperand(0));
  MIB.add(MI->getOperand(1));
  MIB.add(MI->getOperand(2));
  MIB.addImm(ARMCC::AL);
  MIB.addReg(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 157-161
```cpp
  if (SetFlags) {
    MIB.addReg(ARM::CPSR);
    MIB->getOperand(5).setIsDef(true);
  } else
    MIB.addReg(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 163-164
```cpp
  MI->eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 166-169
```cpp
// Generate a subs, or sub and cmp, and a branch instead of an LE.
inline void RevertLoopEnd(MachineInstr *MI, const TargetInstrInfo *TII,
                          unsigned BrOpc = ARM::t2Bcc, bool SkipCmp = false) {
  MachineBasicBlock *MBB = MI->getParent();
```
- EN: Implements `RevertLoopEnd`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RevertLoopEnd`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 171-179
```cpp
  // Create cmp
  if (!SkipCmp) {
    MachineInstrBuilder MIB =
        BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(ARM::t2CMPri));
    MIB.add(MI->getOperand(0));
    MIB.addImm(0);
    MIB.addImm(ARMCC::AL);
    MIB.addReg(ARM::NoRegister);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 181-188
```cpp
  // Create bne
  MachineInstrBuilder MIB =
      BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(BrOpc));
  MIB.add(MI->getOperand(1)); // branch target
  MIB.addImm(ARMCC::NE);      // condition code
  MIB.addReg(ARM::CPSR);
  MI->eraseFromParent();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 190-190
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 192-192
```cpp
#endif // LLVM_LIB_TARGET_ARM_MVETAILPREDUTILS_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: interfaces and target-specific declarations.
  - CN: 核心职责：接口与目标相关声明。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。

## Dependencies / 依赖关系

- EN: Backend-local headers: `(none)`.
  - CN: 后端本地头文件：`(none)`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/TargetInstrInfo.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/TargetInstrInfo.h`。
