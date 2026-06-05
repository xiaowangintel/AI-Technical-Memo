# Thumb1InstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/Thumb1InstrInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Thumb-1 implementation of the TargetInstrInfo class.
- 用途 (CN): 实现 ARM 后端中的 `Thumb1InstrInfo`，重点处理指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- Thumb1InstrInfo.cpp - Thumb-1 Instruction Information -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Thumb-1 implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-22
```cpp
#include "Thumb1InstrInfo.h"
#include "ARMSubtarget.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/LiveRegUnits.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstBuilder.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-24
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 26-27
```cpp
Thumb1InstrInfo::Thumb1InstrInfo(const ARMSubtarget &STI)
    : ARMBaseInstrInfo(STI, RI), RI(STI) {}
```
- EN: Implements `Thumb1InstrInfo::Thumb1InstrInfo`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1InstrInfo::Thumb1InstrInfo`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 29-36
```cpp
/// Return the noop instruction to use for a noop.
MCInst Thumb1InstrInfo::getNop() const {
  return MCInstBuilder(ARM::tMOVr)
      .addReg(ARM::R8)
      .addReg(ARM::R8)
      .addImm(ARMCC::AL)
      .addReg(0);
}
```
- EN: Implements `Thumb1InstrInfo::getNop`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1InstrInfo::getNop`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-40
```cpp
unsigned Thumb1InstrInfo::getUnindexedOpcode(unsigned Opc) const {
  return 0;
}
```
- EN: Implements `Thumb1InstrInfo::getUnindexedOpcode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1InstrInfo::getUnindexedOpcode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-49
```cpp
void Thumb1InstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator I,
                                  const DebugLoc &DL, Register DestReg,
                                  Register SrcReg, bool KillSrc,
                                  bool RenamableDest, bool RenamableSrc) const {
  // Need to check the arch.
  MachineFunction &MF = *MBB.getParent();
  const ARMSubtarget &st = MF.getSubtarget<ARMSubtarget>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 51-52
```cpp
  assert(ARM::GPRRegClass.contains(DestReg, SrcReg) &&
         "Thumb1 can only copy GPR registers");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 54-62
```cpp
  if (st.hasV6Ops() || ARM::hGPRRegClass.contains(SrcReg) ||
      !ARM::tGPRRegClass.contains(DestReg))
    BuildMI(MBB, I, DL, get(ARM::tMOVr), DestReg)
        .addReg(SrcReg, getKillRegState(KillSrc))
        .add(predOps(ARMCC::AL));
  else {
    const TargetRegisterInfo *RegInfo = st.getRegisterInfo();
    LiveRegUnits UsedRegs(*RegInfo);
    UsedRegs.addLiveOuts(MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 64-68
```cpp
    auto InstUpToI = MBB.end();
    while (InstUpToI != I)
      // The pre-decrement is on purpose here.
      // We want to have the liveness right before I.
      UsedRegs.stepBackward(*--InstUpToI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 70-75
```cpp
    if (UsedRegs.available(ARM::CPSR)) {
      BuildMI(MBB, I, DL, get(ARM::tMOVSr), DestReg)
          .addReg(SrcReg, getKillRegState(KillSrc))
          ->addRegisterDead(ARM::CPSR, RegInfo);
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 77-80
```cpp
    // Use high register to move source to destination
    // if movs is not an option.
    BitVector Allocatable = RegInfo->getAllocatableSet(
        MF, RegInfo->getRegClass(ARM::hGPRRegClassID));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 82-93
```cpp
    Register TmpReg = ARM::NoRegister;
    // Prefer R12 as it is known to not be preserved anyway
    if (UsedRegs.available(ARM::R12) && Allocatable.test(ARM::R12)) {
      TmpReg = ARM::R12;
    } else {
      for (Register Reg : Allocatable.set_bits()) {
        if (UsedRegs.available(Reg)) {
          TmpReg = Reg;
          break;
        }
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 95-103
```cpp
    if (TmpReg) {
      BuildMI(MBB, I, DL, get(ARM::tMOVr), TmpReg)
          .addReg(SrcReg, getKillRegState(KillSrc))
          .add(predOps(ARMCC::AL));
      BuildMI(MBB, I, DL, get(ARM::tMOVr), DestReg)
          .addReg(TmpReg, getKillRegState(true))
          .add(predOps(ARMCC::AL));
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 105-113
```cpp
    // 'MOV lo, lo' is unpredictable on < v6, so use the stack to do it
    BuildMI(MBB, I, DL, get(ARM::tPUSH))
        .add(predOps(ARMCC::AL))
        .addReg(SrcReg, getKillRegState(KillSrc));
    BuildMI(MBB, I, DL, get(ARM::tPOP))
        .add(predOps(ARMCC::AL))
        .addReg(DestReg, getDefRegState(true));
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 115-123
```cpp
void Thumb1InstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator I,
                                          Register SrcReg, bool isKill, int FI,
                                          const TargetRegisterClass *RC,
                                          Register VReg,
                                          MachineInstr::MIFlag Flags) const {
  assert((RC == &ARM::tGPRRegClass ||
          (SrcReg.isPhysical() && isARMLowRegister(SrcReg))) &&
         "Unknown regclass!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-128
```cpp
  if (RC == &ARM::tGPRRegClass ||
      (SrcReg.isPhysical() && isARMLowRegister(SrcReg))) {
    DebugLoc DL;
    if (I != MBB.end()) DL = I->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 130-142
```cpp
    MachineFunction &MF = *MBB.getParent();
    MachineFrameInfo &MFI = MF.getFrameInfo();
    MachineMemOperand *MMO = MF.getMachineMemOperand(
        MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOStore,
        MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
    BuildMI(MBB, I, DL, get(ARM::tSTRspi))
        .addReg(SrcReg, getKillRegState(isKill))
        .addFrameIndex(FI)
        .addImm(0)
        .addMemOperand(MMO)
        .add(predOps(ARMCC::AL));
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 144-152
```cpp
void Thumb1InstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                           MachineBasicBlock::iterator I,
                                           Register DestReg, int FI,
                                           const TargetRegisterClass *RC,
                                           Register VReg, unsigned SubReg,
                                           MachineInstr::MIFlag Flags) const {
  assert((RC->hasSuperClassEq(&ARM::tGPRRegClass) ||
          (DestReg.isPhysical() && isARMLowRegister(DestReg))) &&
         "Unknown regclass!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-157
```cpp
  if (RC->hasSuperClassEq(&ARM::tGPRRegClass) ||
      (DestReg.isPhysical() && isARMLowRegister(DestReg))) {
    DebugLoc DL;
    if (I != MBB.end()) DL = I->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 159-170
```cpp
    MachineFunction &MF = *MBB.getParent();
    MachineFrameInfo &MFI = MF.getFrameInfo();
    MachineMemOperand *MMO = MF.getMachineMemOperand(
        MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOLoad,
        MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
    BuildMI(MBB, I, DL, get(ARM::tLDRspi), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addMemOperand(MMO)
        .add(predOps(ARMCC::AL));
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 172-176
```cpp
void Thumb1InstrInfo::expandLoadStackGuard(
    MachineBasicBlock::iterator MI) const {
  MachineFunction &MF = *MI->getParent()->getParent();
  const ARMSubtarget &ST = MF.getSubtarget<ARMSubtarget>();
  const auto *GV = cast<GlobalValue>((*MI->memoperands_begin())->getValue());
```
- EN: Implements `Thumb1InstrInfo::expandLoadStackGuard`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1InstrInfo::expandLoadStackGuard`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 178-179
```cpp
  assert(MF.getFunction().getParent()->getStackProtectorGuard() != "tls" &&
         "TLS stack protector not supported for Thumb1 targets");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 181-191
```cpp
  unsigned Instr;
  if (!GV->isDSOLocal())
    Instr = ARM::tLDRLIT_ga_pcrel;
  else if (ST.genExecuteOnly() && ST.hasV8MBaselineOps())
    Instr = ARM::t2MOVi32imm;
  else if (ST.genExecuteOnly())
    Instr = ARM::tMOVi32imm;
  else
    Instr = ARM::tLDRLIT_ga_abs;
  expandLoadStackGuardBase(MI, Instr, ARM::tLDRi);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 193-201
```cpp
bool Thumb1InstrInfo::canCopyGluedNodeDuringSchedule(SDNode *N) const {
  // In Thumb1 the scheduler may need to schedule a cross-copy between GPRS and CPSR
  // but this is not always possible there, so allow the Scheduler to clone tADCS and tSBCS
  // even if they have glue.
  // FIXME. Actually implement the cross-copy where it is possible (post v6)
  // because these copies entail more spilling.
  unsigned Opcode = N->getMachineOpcode();
  if (Opcode == ARM::tADCS || Opcode == ARM::tSBCS)
    return true;
```
- EN: Implements `Thumb1InstrInfo::canCopyGluedNodeDuringSchedule`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1InstrInfo::canCopyGluedNodeDuringSchedule`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 203-204
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `Thumb1InstrInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`Thumb1InstrInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/BitVector.h`, `llvm/CodeGen/LiveRegUnits.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/IR/Module.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstBuilder.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitVector.h`, `llvm/CodeGen/LiveRegUnits.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/IR/Module.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstBuilder.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
