# Thumb2InstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/Thumb2InstrInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Thumb-2 implementation of the TargetInstrInfo class.
- 用途 (CN): 实现 ARM 后端中的 `Thumb2InstrInfo`，重点处理指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Thumb2InstrInfo.cpp - Thumb-2 Instruction Information --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Thumb-2 implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "Thumb2InstrInfo.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Module.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-34
```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstBuilder.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetMachine.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 36-36
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 38-41
```cpp
static cl::opt<bool>
OldT2IfCvt("old-thumb2-ifcvt", cl::Hidden,
           cl::desc("Use old-style Thumb2 if-conversion heuristics"),
           cl::init(false));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-46
```cpp
static cl::opt<bool>
PreferNoCSEL("prefer-no-csel", cl::Hidden,
             cl::desc("Prefer predicated Move to CSEL"),
             cl::init(false));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 48-49
```cpp
Thumb2InstrInfo::Thumb2InstrInfo(const ARMSubtarget &STI)
    : ARMBaseInstrInfo(STI, RI), RI(STI) {}
```
- EN: Implements `Thumb2InstrInfo::Thumb2InstrInfo`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::Thumb2InstrInfo`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 51-54
```cpp
/// Return the noop instruction to use for a noop.
MCInst Thumb2InstrInfo::getNop() const {
  return MCInstBuilder(ARM::tHINT).addImm(0).addImm(ARMCC::AL).addReg(0);
}
```
- EN: Implements `Thumb2InstrInfo::getNop`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::getNop`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-59
```cpp
unsigned Thumb2InstrInfo::getUnindexedOpcode(unsigned Opc) const {
  // FIXME
  return 0;
}
```
- EN: Implements `Thumb2InstrInfo::getUnindexedOpcode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::getUnindexedOpcode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 61-69
```cpp
void
Thumb2InstrInfo::ReplaceTailWithBranchTo(MachineBasicBlock::iterator Tail,
                                         MachineBasicBlock *NewDest) const {
  MachineBasicBlock *MBB = Tail->getParent();
  ARMFunctionInfo *AFI = MBB->getParent()->getInfo<ARMFunctionInfo>();
  if (!AFI->hasITBlocks() || Tail->isBranch()) {
    TargetInstrInfo::ReplaceTailWithBranchTo(Tail, NewDest);
    return;
  }
```
- EN: Implements `Thumb2InstrInfo::ReplaceTailWithBranchTo`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::ReplaceTailWithBranchTo`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-78
```cpp
  // If the first instruction of Tail is predicated, we may have to update
  // the IT instruction.
  Register PredReg;
  ARMCC::CondCodes CC = getInstrPredicate(*Tail, PredReg);
  MachineBasicBlock::iterator MBBI = Tail;
  if (CC != ARMCC::AL)
    // Expecting at least the t2IT instruction before it.
    --MBBI;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 80-81
```cpp
  // Actually replace the tail.
  TargetInstrInfo::ReplaceTailWithBranchTo(Tail, NewDest);
```
- EN: Declares `TargetInstrInfo::ReplaceTailWithBranchTo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `TargetInstrInfo::ReplaceTailWithBranchTo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-97
```cpp
  // Fix up IT.
  if (CC != ARMCC::AL) {
    MachineBasicBlock::iterator E = MBB->begin();
    unsigned Count = 4; // At most 4 instructions in an IT block.
    while (Count && MBBI != E) {
      if (MBBI->isDebugInstr()) {
        --MBBI;
        continue;
      }
      if (MBBI->getOpcode() == ARM::t2IT) {
        unsigned Mask = MBBI->getOperand(1).getImm();
        if (Count == 4)
          MBBI->eraseFromParent();
        else {
          unsigned MaskOn = 1 << Count;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 98-105
```cpp
          unsigned MaskOff = ~(MaskOn - 1);
          MBBI->getOperand(1).setImm((Mask & MaskOff) | MaskOn);
        }
        return;
      }
      --MBBI;
      --Count;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 107-110
```cpp
    // Ctrl flow can reach here if branch folding is run before IT block
    // formation pass.
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 112-119
```cpp
bool
Thumb2InstrInfo::isLegalToSplitMBBAt(MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator MBBI) const {
  while (MBBI->isDebugInstr()) {
    ++MBBI;
    if (MBBI == MBB.end())
      return false;
  }
```
- EN: Implements `Thumb2InstrInfo::isLegalToSplitMBBAt`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::isLegalToSplitMBBAt`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-123
```cpp
  Register PredReg;
  return getITInstrPredicate(*MBBI, PredReg) == ARMCC::AL;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-134
```cpp
MachineInstr *
Thumb2InstrInfo::optimizeSelect(MachineInstr &MI,
                                SmallPtrSetImpl<MachineInstr *> &SeenMIs,
                                bool PreferFalse) const {
  // Try to use the base optimizeSelect, which uses canFoldIntoMOVCC to fold the
  // MOVCC into another instruction. If that fails on 8.1-M fall back to using a
  // CSEL.
  MachineInstr *RV = ARMBaseInstrInfo::optimizeSelect(MI, SeenMIs, PreferFalse);
  if (!RV && getSubtarget().hasV8_1MMainlineOps() && !PreferNoCSEL) {
    Register DestReg = MI.getOperand(0).getReg();
```
- EN: Implements `Thumb2InstrInfo::optimizeSelect`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::optimizeSelect`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 136-137
```cpp
    if (!DestReg.isVirtual())
      return nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 139-148
```cpp
    MachineInstrBuilder NewMI = BuildMI(*MI.getParent(), MI, MI.getDebugLoc(),
                                        get(ARM::t2CSEL), DestReg)
                                    .add(MI.getOperand(2))
                                    .add(MI.getOperand(1))
                                    .add(MI.getOperand(3));
    SeenMIs.insert(NewMI);
    return NewMI;
  }
  return RV;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 150-157
```cpp
void Thumb2InstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator I,
                                  const DebugLoc &DL, Register DestReg,
                                  Register SrcReg, bool KillSrc,
                                  bool RenamableDest, bool RenamableSrc) const {
  // Handle SPR, DPR, and QPR copies.
  if (!ARM::GPRRegClass.contains(DestReg, SrcReg))
    return ARMBaseInstrInfo::copyPhysReg(MBB, I, DL, DestReg, SrcReg, KillSrc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 159-162
```cpp
  BuildMI(MBB, I, DL, get(ARM::tMOVr), DestReg)
      .addReg(SrcReg, getKillRegState(KillSrc))
      .add(predOps(ARMCC::AL));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-171
```cpp
void Thumb2InstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator I,
                                          Register SrcReg, bool isKill, int FI,
                                          const TargetRegisterClass *RC,
                                          Register VReg,
                                          MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end()) DL = I->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 173-177
```cpp
  MachineFunction &MF = *MBB.getParent();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MachineMemOperand *MMO = MF.getMachineMemOperand(
      MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOStore,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 179-187
```cpp
  if (ARM::GPRRegClass.hasSubClassEq(RC)) {
    BuildMI(MBB, I, DL, get(ARM::t2STRi12))
        .addReg(SrcReg, getKillRegState(isKill))
        .addFrameIndex(FI)
        .addImm(0)
        .addMemOperand(MMO)
        .add(predOps(ARMCC::AL));
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 189-196
```cpp
  if (ARM::GPRPairRegClass.hasSubClassEq(RC)) {
    // Thumb2 STRD expects its dest-registers to be in rGPR. Not a problem for
    // gsub_0, but needs an extra constraint for gsub_1 (which could be sp
    // otherwise).
    if (SrcReg.isVirtual()) {
      MachineRegisterInfo *MRI = &MF.getRegInfo();
      MRI->constrainRegClass(SrcReg, &ARM::GPRPairnospRegClass);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 198-203
```cpp
    MachineInstrBuilder MIB = BuildMI(MBB, I, DL, get(ARM::t2STRDi8));
    AddDReg(MIB, SrcReg, ARM::gsub_0, getKillRegState(isKill));
    AddDReg(MIB, SrcReg, ARM::gsub_1, {});
    MIB.addFrameIndex(FI).addImm(0).addMemOperand(MMO).add(predOps(ARMCC::AL));
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 205-207
```cpp
  ARMBaseInstrInfo::storeRegToStackSlot(MBB, I, SrcReg, isKill, FI, RC,
                                        Register());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-221
```cpp
void Thumb2InstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                           MachineBasicBlock::iterator I,
                                           Register DestReg, int FI,
                                           const TargetRegisterClass *RC,
                                           Register VReg, unsigned SubReg,
                                           MachineInstr::MIFlag Flags) const {
  MachineFunction &MF = *MBB.getParent();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MachineMemOperand *MMO = MF.getMachineMemOperand(
      MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOLoad,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
  DebugLoc DL;
  if (I != MBB.end()) DL = I->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 223-230
```cpp
  if (ARM::GPRRegClass.hasSubClassEq(RC)) {
    BuildMI(MBB, I, DL, get(ARM::t2LDRi12), DestReg)
        .addFrameIndex(FI)
        .addImm(0)
        .addMemOperand(MMO)
        .add(predOps(ARMCC::AL));
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 232-239
```cpp
  if (ARM::GPRPairRegClass.hasSubClassEq(RC)) {
    // Thumb2 LDRD expects its dest-registers to be in rGPR. Not a problem for
    // gsub_0, but needs an extra constraint for gsub_1 (which could be sp
    // otherwise).
    if (DestReg.isVirtual()) {
      MachineRegisterInfo *MRI = &MF.getRegInfo();
      MRI->constrainRegClass(DestReg, &ARM::GPRPairnospRegClass);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 241-244
```cpp
    MachineInstrBuilder MIB = BuildMI(MBB, I, DL, get(ARM::t2LDRDi8));
    AddDReg(MIB, DestReg, ARM::gsub_0, RegState::DefineNoRead);
    AddDReg(MIB, DestReg, ARM::gsub_1, RegState::DefineNoRead);
    MIB.addFrameIndex(FI).addImm(0).addMemOperand(MMO).add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 246-249
```cpp
    if (DestReg.isPhysical())
      MIB.addReg(DestReg, RegState::ImplicitDefine);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 251-252
```cpp
  ARMBaseInstrInfo::loadRegFromStackSlot(MBB, I, DestReg, FI, RC, Register());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 254-257
```cpp
void Thumb2InstrInfo::expandLoadStackGuard(
    MachineBasicBlock::iterator MI) const {
  MachineFunction &MF = *MI->getParent()->getParent();
  Module &M = *MF.getFunction().getParent();
```
- EN: Implements `Thumb2InstrInfo::expandLoadStackGuard`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::expandLoadStackGuard`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 259-262
```cpp
  if (M.getStackProtectorGuard() == "tls") {
    expandLoadStackGuardBase(MI, ARM::t2MRC, ARM::t2LDRi12);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 264-277
```cpp
  const auto *GV = cast<GlobalValue>((*MI->memoperands_begin())->getValue());
  const ARMSubtarget &Subtarget = MF.getSubtarget<ARMSubtarget>();
  bool IsPIC = MF.getTarget().isPositionIndependent();
  if (Subtarget.isTargetELF() && !GV->isDSOLocal())
    expandLoadStackGuardBase(MI, ARM::t2LDRLIT_ga_pcrel, ARM::t2LDRi12);
  else if (!Subtarget.useMovt())
    expandLoadStackGuardBase(
        MI, IsPIC ? ARM::t2LDRLIT_ga_pcrel : ARM::tLDRLIT_ga_abs,
        ARM::t2LDRi12);
  else if (IsPIC)
    expandLoadStackGuardBase(MI, ARM::t2MOV_ga_pcrel, ARM::t2LDRi12);
  else
    expandLoadStackGuardBase(MI, ARM::t2MOVi32imm, ARM::t2LDRi12);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-293
```cpp
MachineInstr *Thumb2InstrInfo::commuteInstructionImpl(MachineInstr &MI,
                                                      bool NewMI,
                                                      unsigned OpIdx1,
                                                      unsigned OpIdx2) const {
  switch (MI.getOpcode()) {
  case ARM::MVE_VMAXNMAf16:
  case ARM::MVE_VMAXNMAf32:
  case ARM::MVE_VMINNMAf16:
  case ARM::MVE_VMINNMAf32:
    // Don't allow predicated instructions to be commuted.
    if (getVPTInstrPredicate(MI) != ARMVCC::None)
      return nullptr;
  }
  return ARMBaseInstrInfo::commuteInstructionImpl(MI, NewMI, OpIdx1, OpIdx2);
}
```
- EN: Implements `Thumb2InstrInfo::commuteInstructionImpl`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::commuteInstructionImpl`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 295-309
```cpp
bool Thumb2InstrInfo::isSchedulingBoundary(const MachineInstr &MI,
                                           const MachineBasicBlock *MBB,
                                           const MachineFunction &MF) const {
  // BTI clearing instructions shall not take part in scheduling regions as
  // they must stay in their intended place. Although PAC isn't BTI clearing,
  // it can be transformed into PACBTI after the pre-RA Machine Scheduling
  // has taken place, so its movement must also be restricted.
  switch (MI.getOpcode()) {
  case ARM::t2BTI:
  case ARM::t2PAC:
  case ARM::t2PACBTI:
  case ARM::t2SG:
    return true;
  default:
    break;
```
- EN: Implements `Thumb2InstrInfo::isSchedulingBoundary`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb2InstrInfo::isSchedulingBoundary`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 310-312
```cpp
  }
  return ARMBaseInstrInfo::isSchedulingBoundary(MI, MBB, MF);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 314-326
```cpp
void llvm::emitT2RegPlusImmediate(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator &MBBI,
                                  const DebugLoc &dl, Register DestReg,
                                  Register BaseReg, int NumBytes,
                                  ARMCC::CondCodes Pred, Register PredReg,
                                  const ARMBaseInstrInfo &TII,
                                  unsigned MIFlags) {
  if (NumBytes == 0 && DestReg != BaseReg) {
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), DestReg)
      .addReg(BaseReg, RegState::Kill)
      .addImm((unsigned)Pred).addReg(PredReg).setMIFlags(MIFlags);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 328-329
```cpp
  bool isSub = NumBytes < 0;
  if (isSub) NumBytes = -NumBytes;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 331-345
```cpp
  // If profitable, use a movw or movt to materialize the offset.
  // FIXME: Use the scavenger to grab a scratch register.
  if (DestReg != ARM::SP && DestReg != BaseReg &&
      NumBytes >= 4096 &&
      ARM_AM::getT2SOImmVal(NumBytes) == -1) {
    bool Fits = false;
    if (NumBytes < 65536) {
      // Use a movw to materialize the 16-bit constant.
      BuildMI(MBB, MBBI, dl, TII.get(ARM::t2MOVi16), DestReg)
        .addImm(NumBytes)
        .addImm((unsigned)Pred).addReg(PredReg).setMIFlags(MIFlags);
      Fits = true;
    } else if ((NumBytes & 0xffff) == 0) {
      // Use a movt to materialize the 32-bit constant.
      BuildMI(MBB, MBBI, dl, TII.get(ARM::t2MOVTi16), DestReg)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 346-350
```cpp
        .addReg(DestReg)
        .addImm(NumBytes >> 16)
        .addImm((unsigned)Pred).addReg(PredReg).setMIFlags(MIFlags);
      Fits = true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 352-366
```cpp
    if (Fits) {
      if (isSub) {
        BuildMI(MBB, MBBI, dl, TII.get(ARM::t2SUBrr), DestReg)
            .addReg(BaseReg)
            .addReg(DestReg, RegState::Kill)
            .add(predOps(Pred, PredReg))
            .add(condCodeOp())
            .setMIFlags(MIFlags);
      } else {
        // Here we know that DestReg is not SP but we do not
        // know anything about BaseReg. t2ADDrr is an invalid
        // instruction is SP is used as the second argument, but
        // is fine if SP is the first argument. To be sure we
        // do not generate invalid encoding, put BaseReg first.
        BuildMI(MBB, MBBI, dl, TII.get(ARM::t2ADDrr), DestReg)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 367-375
```cpp
            .addReg(BaseReg)
            .addReg(DestReg, RegState::Kill)
            .add(predOps(Pred, PredReg))
            .add(condCodeOp())
            .setMIFlags(MIFlags);
      }
      return;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 377-388
```cpp
  while (NumBytes) {
    unsigned ThisVal = NumBytes;
    unsigned Opc = 0;
    if (DestReg == ARM::SP && BaseReg != ARM::SP) {
      // mov sp, rn. Note t2MOVr cannot be used.
      BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), DestReg)
          .addReg(BaseReg)
          .setMIFlags(MIFlags)
          .add(predOps(ARMCC::AL));
      BaseReg = ARM::SP;
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 390-391
```cpp
    assert((DestReg != ARM::SP || BaseReg == ARM::SP) &&
           "Writing to SP, from other register.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 393-407
```cpp
    // Try to use T1, as it smaller
    if ((DestReg == ARM::SP) && (ThisVal < ((1 << 7) - 1) * 4)) {
      assert((ThisVal & 3) == 0 && "Stack update is not multiple of 4?");
      Opc = isSub ? ARM::tSUBspi : ARM::tADDspi;
      BuildMI(MBB, MBBI, dl, TII.get(Opc), DestReg)
          .addReg(BaseReg)
          .addImm(ThisVal / 4)
          .setMIFlags(MIFlags)
          .add(predOps(ARMCC::AL));
      break;
    }
    bool HasCCOut = true;
    int ImmIsT2SO = ARM_AM::getT2SOImmVal(ThisVal);
    bool ToSP = DestReg == ARM::SP;
    unsigned t2SUB = ToSP ? ARM::t2SUBspImm : ARM::t2SUBri;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 408-422
```cpp
    unsigned t2ADD = ToSP ? ARM::t2ADDspImm : ARM::t2ADDri;
    unsigned t2SUBi12 = ToSP ? ARM::t2SUBspImm12 : ARM::t2SUBri12;
    unsigned t2ADDi12 = ToSP ? ARM::t2ADDspImm12 : ARM::t2ADDri12;
    Opc = isSub ? t2SUB : t2ADD;
    // Prefer T2: sub rd, rn, so_imm | sub sp, sp, so_imm
    if (ImmIsT2SO != -1) {
      NumBytes = 0;
    } else if (ThisVal < 4096) {
      // Prefer T3 if can make it in a single go: subw rd, rn, imm12 | subw sp,
      // sp, imm12
      Opc = isSub ? t2SUBi12 : t2ADDi12;
      HasCCOut = false;
      NumBytes = 0;
    } else {
      // Use one T2 instruction to reduce NumBytes
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 423-429
```cpp
      // FIXME: Move this to ARMAddressingModes.h?
      unsigned RotAmt = llvm::countl_zero(ThisVal);
      ThisVal = ThisVal & llvm::rotr<uint32_t>(0xff000000U, RotAmt);
      NumBytes &= ~ThisVal;
      assert(ARM_AM::getT2SOImmVal(ThisVal) != -1 &&
             "Bit extraction didn't work?");
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 431-438
```cpp
    // Build the new ADD / SUB.
    MachineInstrBuilder MIB = BuildMI(MBB, MBBI, dl, TII.get(Opc), DestReg)
                                  .addReg(BaseReg, RegState::Kill)
                                  .addImm(ThisVal)
                                  .add(predOps(ARMCC::AL))
                                  .setMIFlags(MIFlags);
    if (HasCCOut)
      MIB.add(condCodeOp());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 440-442
```cpp
    BaseReg = DestReg;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 444-458
```cpp
static unsigned
negativeOffsetOpcode(unsigned opcode)
{
  switch (opcode) {
  case ARM::t2LDRi12:   return ARM::t2LDRi8;
  case ARM::t2LDRHi12:  return ARM::t2LDRHi8;
  case ARM::t2LDRBi12:  return ARM::t2LDRBi8;
  case ARM::t2LDRSHi12: return ARM::t2LDRSHi8;
  case ARM::t2LDRSBi12: return ARM::t2LDRSBi8;
  case ARM::t2STRi12:   return ARM::t2STRi8;
  case ARM::t2STRBi12:  return ARM::t2STRBi8;
  case ARM::t2STRHi12:  return ARM::t2STRHi8;
  case ARM::t2PLDi12:   return ARM::t2PLDi8;
  case ARM::t2PLDWi12:  return ARM::t2PLDWi8;
  case ARM::t2PLIi12:   return ARM::t2PLIi8;
```
- EN: Implements `negativeOffsetOpcode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `negativeOffsetOpcode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 460-471
```cpp
  case ARM::t2LDRi8:
  case ARM::t2LDRHi8:
  case ARM::t2LDRBi8:
  case ARM::t2LDRSHi8:
  case ARM::t2LDRSBi8:
  case ARM::t2STRi8:
  case ARM::t2STRBi8:
  case ARM::t2STRHi8:
  case ARM::t2PLDi8:
  case ARM::t2PLDWi8:
  case ARM::t2PLIi8:
    return opcode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 473-476
```cpp
  default:
    llvm_unreachable("unknown thumb2 opcode.");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 478-492
```cpp
static unsigned
positiveOffsetOpcode(unsigned opcode)
{
  switch (opcode) {
  case ARM::t2LDRi8:   return ARM::t2LDRi12;
  case ARM::t2LDRHi8:  return ARM::t2LDRHi12;
  case ARM::t2LDRBi8:  return ARM::t2LDRBi12;
  case ARM::t2LDRSHi8: return ARM::t2LDRSHi12;
  case ARM::t2LDRSBi8: return ARM::t2LDRSBi12;
  case ARM::t2STRi8:   return ARM::t2STRi12;
  case ARM::t2STRBi8:  return ARM::t2STRBi12;
  case ARM::t2STRHi8:  return ARM::t2STRHi12;
  case ARM::t2PLDi8:   return ARM::t2PLDi12;
  case ARM::t2PLDWi8:  return ARM::t2PLDWi12;
  case ARM::t2PLIi8:   return ARM::t2PLIi12;
```
- EN: Implements `positiveOffsetOpcode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `positiveOffsetOpcode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 494-505
```cpp
  case ARM::t2LDRi12:
  case ARM::t2LDRHi12:
  case ARM::t2LDRBi12:
  case ARM::t2LDRSHi12:
  case ARM::t2LDRSBi12:
  case ARM::t2STRi12:
  case ARM::t2STRBi12:
  case ARM::t2STRHi12:
  case ARM::t2PLDi12:
  case ARM::t2PLDWi12:
  case ARM::t2PLIi12:
    return opcode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 507-510
```cpp
  default:
    llvm_unreachable("unknown thumb2 opcode.");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 512-526
```cpp
static unsigned
immediateOffsetOpcode(unsigned opcode)
{
  switch (opcode) {
  case ARM::t2LDRs:   return ARM::t2LDRi12;
  case ARM::t2LDRHs:  return ARM::t2LDRHi12;
  case ARM::t2LDRBs:  return ARM::t2LDRBi12;
  case ARM::t2LDRSHs: return ARM::t2LDRSHi12;
  case ARM::t2LDRSBs: return ARM::t2LDRSBi12;
  case ARM::t2STRs:   return ARM::t2STRi12;
  case ARM::t2STRBs:  return ARM::t2STRBi12;
  case ARM::t2STRHs:  return ARM::t2STRHi12;
  case ARM::t2PLDs:   return ARM::t2PLDi12;
  case ARM::t2PLDWs:  return ARM::t2PLDWi12;
  case ARM::t2PLIs:   return ARM::t2PLIi12;
```
- EN: Implements `immediateOffsetOpcode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `immediateOffsetOpcode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 528-542
```cpp
  case ARM::t2LDRi12:
  case ARM::t2LDRHi12:
  case ARM::t2LDRBi12:
  case ARM::t2LDRSHi12:
  case ARM::t2LDRSBi12:
  case ARM::t2STRi12:
  case ARM::t2STRBi12:
  case ARM::t2STRHi12:
  case ARM::t2PLDi12:
  case ARM::t2PLDWi12:
  case ARM::t2PLIi12:
  case ARM::t2LDRi8:
  case ARM::t2LDRHi8:
  case ARM::t2LDRBi8:
  case ARM::t2LDRSHi8:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 543-550
```cpp
  case ARM::t2LDRSBi8:
  case ARM::t2STRi8:
  case ARM::t2STRBi8:
  case ARM::t2STRHi8:
  case ARM::t2PLDi8:
  case ARM::t2PLDWi8:
  case ARM::t2PLIi8:
    return opcode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 552-555
```cpp
  default:
    llvm_unreachable("unknown thumb2 opcode.");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 557-564
```cpp
bool llvm::rewriteT2FrameIndex(MachineInstr &MI, unsigned FrameRegIdx,
                               Register FrameReg, int &Offset,
                               const ARMBaseInstrInfo &TII,
                               const TargetRegisterInfo *TRI) {
  unsigned Opcode = MI.getOpcode();
  const MCInstrDesc &Desc = MI.getDesc();
  unsigned AddrMode = (Desc.TSFlags & ARMII::AddrModeMask);
  bool isSub = false;
```
- EN: Implements `llvm::rewriteT2FrameIndex`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::rewriteT2FrameIndex`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 566-567
```cpp
  MachineFunction &MF = *MI.getParent()->getParent();
  const TargetRegisterClass *RegClass = TII.getRegClass(Desc, FrameRegIdx);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 569-571
```cpp
  // Memory operands in inline assembly always use AddrModeT2_i12.
  if (Opcode == ARM::INLINEASM || Opcode == ARM::INLINEASM_BR)
    AddrMode = ARMII::AddrModeT2_i12; // FIXME. mode for thumb2?
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 573-575
```cpp
  const bool IsSP = Opcode == ARM::t2ADDspImm12 || Opcode == ARM::t2ADDspImm;
  if (IsSP || Opcode == ARM::t2ADDri || Opcode == ARM::t2ADDri12) {
    Offset += MI.getOperand(FrameRegIdx+1).getImm();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 577-589
```cpp
    Register PredReg;
    if (Offset == 0 && getInstrPredicate(MI, PredReg) == ARMCC::AL &&
        !MI.definesRegister(ARM::CPSR, /*TRI=*/nullptr)) {
      // Turn it into a move.
      MI.setDesc(TII.get(ARM::tMOVr));
      MI.getOperand(FrameRegIdx).ChangeToRegister(FrameReg, false);
      // Remove offset and remaining explicit predicate operands.
      do MI.removeOperand(FrameRegIdx+1);
      while (MI.getNumOperands() > FrameRegIdx+1);
      MachineInstrBuilder MIB(*MI.getParent()->getParent(), &MI);
      MIB.add(predOps(ARMCC::AL));
      return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 591-591
```cpp
    bool HasCCOut = (Opcode != ARM::t2ADDspImm12 && Opcode != ARM::t2ADDri12);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 593-599
```cpp
    if (Offset < 0) {
      Offset = -Offset;
      isSub = true;
      MI.setDesc(IsSP ? TII.get(ARM::t2SUBspImm) : TII.get(ARM::t2SUBri));
    } else {
      MI.setDesc(IsSP ? TII.get(ARM::t2ADDspImm) : TII.get(ARM::t2ADDri));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 601-615
```cpp
    // Common case: small offset, fits into instruction.
    if (ARM_AM::getT2SOImmVal(Offset) != -1) {
      MI.getOperand(FrameRegIdx).ChangeToRegister(FrameReg, false);
      MI.getOperand(FrameRegIdx+1).ChangeToImmediate(Offset);
      // Add cc_out operand if the original instruction did not have one.
      if (!HasCCOut)
        MI.addOperand(MachineOperand::CreateReg(0, false));
      Offset = 0;
      return true;
    }
    // Another common case: imm12.
    if (Offset < 4096 &&
        (!HasCCOut || MI.getOperand(MI.getNumOperands()-1).getReg() == 0)) {
      unsigned NewOpc = isSub ? IsSP ? ARM::t2SUBspImm12 : ARM::t2SUBri12
                              : IsSP ? ARM::t2ADDspImm12 : ARM::t2ADDri12;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 616-624
```cpp
      MI.setDesc(TII.get(NewOpc));
      MI.getOperand(FrameRegIdx).ChangeToRegister(FrameReg, false);
      MI.getOperand(FrameRegIdx+1).ChangeToImmediate(Offset);
      // Remove the cc_out operand.
      if (HasCCOut)
        MI.removeOperand(MI.getNumOperands()-1);
      Offset = 0;
      return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 626-629
```cpp
    // Otherwise, extract 8 adjacent bits from the immediate into this
    // t2ADDri/t2SUBri.
    unsigned RotAmt = llvm::countl_zero<unsigned>(Offset);
    unsigned ThisImmVal = Offset & llvm::rotr<uint32_t>(0xff000000U, RotAmt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 631-632
```cpp
    // We will handle these bits from offset, clear them.
    Offset &= ~ThisImmVal;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 634-643
```cpp
    assert(ARM_AM::getT2SOImmVal(ThisImmVal) != -1 &&
           "Bit extraction didn't work?");
    MI.getOperand(FrameRegIdx+1).ChangeToImmediate(ThisImmVal);
    // Add cc_out operand if the original instruction did not have one.
    if (!HasCCOut)
      MI.addOperand(MachineOperand::CreateReg(0, false));
  } else {
    // AddrMode4 and AddrMode6 cannot handle any offset.
    if (AddrMode == ARMII::AddrMode4 || AddrMode == ARMII::AddrMode6)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 645-653
```cpp
    // AddrModeT2_so cannot handle any offset. If there is no offset
    // register then we change to an immediate version.
    unsigned NewOpc = Opcode;
    if (AddrMode == ARMII::AddrModeT2_so) {
      Register OffsetReg = MI.getOperand(FrameRegIdx + 1).getReg();
      if (OffsetReg != 0) {
        MI.getOperand(FrameRegIdx).ChangeToRegister(FrameReg, false);
        return Offset == 0;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 655-659
```cpp
      MI.removeOperand(FrameRegIdx+1);
      MI.getOperand(FrameRegIdx+1).ChangeToImmediate(0);
      NewOpc = immediateOffsetOpcode(Opcode);
      AddrMode = ARMII::AddrModeT2_i12;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 661-675
```cpp
    unsigned NumBits = 0;
    unsigned Scale = 1;
    if (AddrMode == ARMII::AddrModeT2_i8neg ||
        AddrMode == ARMII::AddrModeT2_i12) {
      // i8 supports only negative, and i12 supports only positive, so
      // based on Offset sign convert Opcode to the appropriate
      // instruction
      Offset += MI.getOperand(FrameRegIdx+1).getImm();
      if (Offset < 0) {
        NewOpc = negativeOffsetOpcode(Opcode);
        NumBits = 8;
        isSub = true;
        Offset = -Offset;
      } else {
        NewOpc = positiveOffsetOpcode(Opcode);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 676-690
```cpp
        NumBits = 12;
      }
    } else if (AddrMode == ARMII::AddrMode5) {
      // VFP address mode.
      const MachineOperand &OffOp = MI.getOperand(FrameRegIdx+1);
      int InstrOffs = ARM_AM::getAM5Offset(OffOp.getImm());
      if (ARM_AM::getAM5Op(OffOp.getImm()) == ARM_AM::sub)
        InstrOffs *= -1;
      NumBits = 8;
      Scale = 4;
      Offset += InstrOffs * 4;
      assert((Offset & (Scale-1)) == 0 && "Can't encode this offset!");
      if (Offset < 0) {
        Offset = -Offset;
        isSub = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 691-705
```cpp
      }
    } else if (AddrMode == ARMII::AddrMode5FP16) {
      // VFP address mode.
      const MachineOperand &OffOp = MI.getOperand(FrameRegIdx+1);
      int InstrOffs = ARM_AM::getAM5FP16Offset(OffOp.getImm());
      if (ARM_AM::getAM5FP16Op(OffOp.getImm()) == ARM_AM::sub)
        InstrOffs *= -1;
      NumBits = 8;
      Scale = 2;
      Offset += InstrOffs * 2;
      assert((Offset & (Scale-1)) == 0 && "Can't encode this offset!");
      if (Offset < 0) {
        Offset = -Offset;
        isSub = true;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 706-720
```cpp
    } else if (AddrMode == ARMII::AddrModeT2_i7s4 ||
               AddrMode == ARMII::AddrModeT2_i7s2 ||
               AddrMode == ARMII::AddrModeT2_i7) {
      Offset += MI.getOperand(FrameRegIdx + 1).getImm();
      unsigned OffsetMask;
      switch (AddrMode) {
      case ARMII::AddrModeT2_i7s4: NumBits = 9; OffsetMask = 0x3; break;
      case ARMII::AddrModeT2_i7s2: NumBits = 8; OffsetMask = 0x1; break;
      default:                     NumBits = 7; OffsetMask = 0x0; break;
      }
      // MCInst operand expects already scaled value.
      Scale = 1;
      assert((Offset & OffsetMask) == 0 && "Can't encode this offset!");
      (void)OffsetMask; // squash unused-variable warning at -NDEBUG
    } else if (AddrMode == ARMII::AddrModeT2_i8s4) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 721-733
```cpp
      Offset += MI.getOperand(FrameRegIdx + 1).getImm();
      NumBits = 8 + 2;
      // MCInst operand expects already scaled value.
      Scale = 1;
      assert((Offset & 3) == 0 && "Can't encode this offset!");
    } else if (AddrMode == ARMII::AddrModeT2_ldrex) {
      Offset += MI.getOperand(FrameRegIdx + 1).getImm() * 4;
      NumBits = 8; // 8 bits scaled by 4
      Scale = 4;
      assert((Offset & 3) == 0 && "Can't encode this offset!");
    } else {
      llvm_unreachable("Unsupported addressing mode!");
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 735-736
```cpp
    if (NewOpc != Opcode)
      MI.setDesc(TII.get(NewOpc));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 738-738
```cpp
    MachineOperand &ImmOp = MI.getOperand(FrameRegIdx+1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 740-753
```cpp
    // Attempt to fold address computation
    // Common case: small offset, fits into instruction. We need to make sure
    // the register class is correct too, for instructions like the MVE
    // VLDRH.32, which only accepts low tGPR registers.
    int ImmedOffset = Offset / Scale;
    unsigned Mask = (1 << NumBits) - 1;
    if ((unsigned)Offset <= Mask * Scale &&
        (FrameReg.isVirtual() || RegClass->contains(FrameReg))) {
      if (FrameReg.isVirtual()) {
        // Make sure the register class for the virtual register is correct
        MachineRegisterInfo *MRI = &MF.getRegInfo();
        if (!MRI->constrainRegClass(FrameReg, RegClass))
          llvm_unreachable("Unable to constrain virtual register class.");
      }
```
- EN: Declares `is`, packaging target-specific state and APIs around `Thumb2InstrInfo`.
- CN: 这里声明 `is`，把与 `Thumb2InstrInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 755-767
```cpp
      // Replace the FrameIndex with fp/sp
      MI.getOperand(FrameRegIdx).ChangeToRegister(FrameReg, false);
      if (isSub) {
        if (AddrMode == ARMII::AddrMode5 || AddrMode == ARMII::AddrMode5FP16)
          // FIXME: Not consistent.
          ImmedOffset |= 1 << NumBits;
        else
          ImmedOffset = -ImmedOffset;
      }
      ImmOp.ChangeToImmediate(ImmedOffset);
      Offset = 0;
      return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 769-783
```cpp
    // Otherwise, offset doesn't fit. Pull in what we can to simplify
    ImmedOffset = ImmedOffset & Mask;
    if (isSub) {
      if (AddrMode == ARMII::AddrMode5 || AddrMode == ARMII::AddrMode5FP16)
        // FIXME: Not consistent.
        ImmedOffset |= 1 << NumBits;
      else {
        ImmedOffset = -ImmedOffset;
        if (ImmedOffset == 0)
          // Change the opcode back if the encoded offset is zero.
          MI.setDesc(TII.get(positiveOffsetOpcode(NewOpc)));
      }
    }
    ImmOp.ChangeToImmediate(ImmedOffset);
    Offset &= ~(Mask*Scale);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 784-784
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 786-788
```cpp
  Offset = (isSub) ? -Offset : Offset;
  return Offset == 0 && (FrameReg.isVirtual() || RegClass->contains(FrameReg));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 790-796
```cpp
ARMCC::CondCodes llvm::getITInstrPredicate(const MachineInstr &MI,
                                           Register &PredReg) {
  unsigned Opc = MI.getOpcode();
  if (Opc == ARM::tBcc || Opc == ARM::t2Bcc)
    return ARMCC::AL;
  return getInstrPredicate(MI, PredReg);
}
```
- EN: Implements `llvm::getITInstrPredicate`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::getITInstrPredicate`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 798-799
```cpp
int llvm::findFirstVPTPredOperandIdx(const MachineInstr &MI) {
  const MCInstrDesc &MCID = MI.getDesc();
```
- EN: Implements `llvm::findFirstVPTPredOperandIdx`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::findFirstVPTPredOperandIdx`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 801-803
```cpp
  for (unsigned i = 0, e = MCID.getNumOperands(); i != e; ++i)
    if (ARM::isVpred(MCID.operands()[i].OperandType))
      return i;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 805-806
```cpp
  return -1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 808-809
```cpp
int llvm::findVPTInactiveOperandIdx(const MachineInstr &MI) {
  const MCInstrDesc &MCID = MI.getDesc();
```
- EN: Implements `llvm::findVPTInactiveOperandIdx`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::findVPTInactiveOperandIdx`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 811-813
```cpp
  for (unsigned i = 0, e = MCID.getNumOperands(); i != e; ++i)
    if (MCID.operands()[i].OperandType == ARM::OPERAND_VPRED_R)
      return i + ARM::SUBOP_vpred_r_inactive;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 815-816
```cpp
  return -1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 818-824
```cpp
ARMVCC::VPTCodes llvm::getVPTInstrPredicate(const MachineInstr &MI,
                                            Register &PredReg) {
  int PIdx = findFirstVPTPredOperandIdx(MI);
  if (PIdx == -1) {
    PredReg = 0;
    return ARMVCC::None;
  }
```
- EN: Implements `llvm::getVPTInstrPredicate`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::getVPTInstrPredicate`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 826-828
```cpp
  PredReg = MI.getOperand(PIdx+1).getReg();
  return (ARMVCC::VPTCodes)MI.getOperand(PIdx).getImm();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 830-831
```cpp
void llvm::recomputeVPTBlockMask(MachineInstr &Instr) {
  assert(isVPTOpcode(Instr.getOpcode()) && "Not a VPST or VPT Instruction!");
```
- EN: Implements `llvm::recomputeVPTBlockMask`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::recomputeVPTBlockMask`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 833-834
```cpp
  MachineOperand &MaskOp = Instr.getOperand(0);
  assert(MaskOp.isImm() && "Operand 0 is not the block mask of the VPT/VPST?!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 836-837
```cpp
  MachineBasicBlock::iterator Iter = ++Instr.getIterator(),
                              End = Instr.getParent()->end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 839-840
```cpp
  while (Iter != End && Iter->isDebugInstr())
    ++Iter;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 842-848
```cpp
  // Verify that the instruction after the VPT/VPST is predicated (it should
  // be), and skip it.
  assert(Iter != End && "Expected some instructions in any VPT block");
  assert(
      getVPTInstrPredicate(*Iter) == ARMVCC::Then &&
      "VPT/VPST should be followed by an instruction with a 'then' predicate!");
  ++Iter;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 850-862
```cpp
  // Iterate over the predicated instructions, updating the BlockMask as we go.
  ARM::PredBlockMask BlockMask = ARM::PredBlockMask::T;
  while (Iter != End) {
    if (Iter->isDebugInstr()) {
      ++Iter;
      continue;
    }
    ARMVCC::VPTCodes Pred = getVPTInstrPredicate(*Iter);
    if (Pred == ARMVCC::None)
      break;
    BlockMask = expandPredBlockMask(BlockMask, Pred);
    ++Iter;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 864-866
```cpp
  // Rewrite the BlockMask.
  MaskOp.setImm((int64_t)(BlockMask));
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

- EN: Backend-local headers: `Thumb2InstrInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`.
  - CN: 后端本地头文件：`Thumb2InstrInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h` ... (+9 more).
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h` ... (+9 more)。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
