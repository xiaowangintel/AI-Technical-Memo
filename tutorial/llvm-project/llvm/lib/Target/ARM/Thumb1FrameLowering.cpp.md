# Thumb1FrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/Thumb1FrameLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Thumb1 implementation of TargetFrameLowering class.
- 用途 (CN): 实现 ARM 后端中的 `Thumb1FrameLowering`，重点处理栈帧布局以及序言/结语生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Thumb1FrameLowering.cpp - Thumb1 Frame Information -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Thumb1 implementation of TargetFrameLowering class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "Thumb1FrameLowering.h"
#include "ARMBaseInstrInfo.h"
#include "ARMBaseRegisterInfo.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "Thumb1InstrInfo.h"
#include "ThumbRegisterInfo.h"
#include "Utils/ARMBaseInfo.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/CFIInstBuilder.h"
#include "llvm/CodeGen/LivePhysRegs.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-41
```cpp
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include <cassert>
#include <iterator>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-43
```cpp
#define DEBUG_TYPE "arm-frame-lowering"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 45-45
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 47-48
```cpp
Thumb1FrameLowering::Thumb1FrameLowering(const ARMSubtarget &sti)
    : ARMFrameLowering(sti) {}
```
- EN: Implements `Thumb1FrameLowering::Thumb1FrameLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::Thumb1FrameLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-58
```cpp
bool Thumb1FrameLowering::hasReservedCallFrame(const MachineFunction &MF) const{
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  unsigned CFSize = MFI.getMaxCallFrameSize();
  // It's not always a good idea to include the call frame as part of the
  // stack frame. ARM (especially Thumb) has small immediate offset to
  // address the stack frame. So a large call frame can cause poor codegen
  // and may even makes it impossible to scavenge a register.
  if (CFSize >= ((1 << 8) - 1) * 4 / 2) // Half of imm8 * 4
    return false;
```
- EN: Implements `Thumb1FrameLowering::hasReservedCallFrame`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::hasReservedCallFrame`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 60-61
```cpp
  return !MFI.hasVarSizedObjects();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 63-77
```cpp
static void
emitPrologueEpilogueSPUpdate(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator &MBBI,
                             const TargetInstrInfo &TII, const DebugLoc &dl,
                             const ThumbRegisterInfo &MRI, int NumBytes,
                             unsigned ScratchReg, unsigned MIFlags) {
  // If it would take more than three instructions to adjust the stack pointer
  // using tADDspi/tSUBspi, load an immediate instead.
  if (std::abs(NumBytes) > 508 * 3) {
    // We use a different codepath here from the normal
    // emitThumbRegPlusImmediate so we don't have to deal with register
    // scavenging. (Scavenging could try to use the emergency spill slot
    // before we've actually finished setting up the stack.)
    if (ScratchReg == ARM::NoRegister)
      report_fatal_error("Failed to emit Thumb1 stack adjustment");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 78-92
```cpp
    MachineFunction &MF = *MBB.getParent();
    const ARMSubtarget &ST = MF.getSubtarget<ARMSubtarget>();
    if (ST.genExecuteOnly()) {
      unsigned XOInstr = ST.useMovt() ? ARM::t2MOVi32imm : ARM::tMOVi32imm;
      BuildMI(MBB, MBBI, dl, TII.get(XOInstr), ScratchReg)
          .addImm(NumBytes).setMIFlags(MIFlags);
    } else {
      MRI.emitLoadConstPool(MBB, MBBI, dl, ScratchReg, 0, NumBytes, ARMCC::AL,
                            0, MIFlags);
    }
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tADDhirr), ARM::SP)
        .addReg(ARM::SP)
        .addReg(ScratchReg, RegState::Kill)
        .add(predOps(ARMCC::AL))
        .setMIFlags(MIFlags);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 93-98
```cpp
    return;
  }
  // FIXME: This is assuming the heuristics in emitThumbRegPlusImmediate
  // won't change.
  emitThumbRegPlusImmediate(MBB, MBBI, dl, ARM::SP, ARM::SP, NumBytes, TII,
                            MRI, MIFlags);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 100-100
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 102-109
```cpp
static void emitCallSPUpdate(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator &MBBI,
                             const TargetInstrInfo &TII, const DebugLoc &dl,
                             const ThumbRegisterInfo &MRI, int NumBytes,
                             unsigned MIFlags = MachineInstr::NoFlags) {
  emitThumbRegPlusImmediate(MBB, MBBI, dl, ARM::SP, ARM::SP, NumBytes, TII,
                            MRI, MIFlags);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 112-126
```cpp
MachineBasicBlock::iterator Thumb1FrameLowering::
eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator I) const {
  const Thumb1InstrInfo &TII =
      *static_cast<const Thumb1InstrInfo *>(STI.getInstrInfo());
  const ThumbRegisterInfo *RegInfo =
      static_cast<const ThumbRegisterInfo *>(STI.getRegisterInfo());
  if (!hasReservedCallFrame(MF)) {
    // If we have alloca, convert as follows:
    // ADJCALLSTACKDOWN -> sub, sp, sp, amount
    // ADJCALLSTACKUP   -> add, sp, sp, amount
    MachineInstr &Old = *I;
    DebugLoc dl = Old.getDebugLoc();
    unsigned Amount = TII.getFrameSize(Old);
    if (Amount != 0) {
```
- EN: Implements `eliminateCallFramePseudoInstr`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `eliminateCallFramePseudoInstr`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 127-130
```cpp
      // We need to keep the stack aligned properly.  To do this, we round the
      // amount of space needed for the outgoing arguments up to the next
      // alignment boundary.
      Amount = alignTo(Amount, getStackAlign());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-143
```cpp
      // Replace the pseudo instruction with a new instruction...
      unsigned Opc = Old.getOpcode();
      if (Opc == ARM::ADJCALLSTACKDOWN || Opc == ARM::tADJCALLSTACKDOWN) {
        emitCallSPUpdate(MBB, I, TII, dl, *RegInfo, -Amount);
      } else {
        assert(Opc == ARM::ADJCALLSTACKUP || Opc == ARM::tADJCALLSTACKUP);
        emitCallSPUpdate(MBB, I, TII, dl, *RegInfo, Amount);
      }
    }
  }
  return MBB.erase(I);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 145-153
```cpp
void Thumb1FrameLowering::emitPrologue(MachineFunction &MF,
                                       MachineBasicBlock &MBB) const {
  MachineBasicBlock::iterator MBBI = MBB.begin();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  const ThumbRegisterInfo *RegInfo =
      static_cast<const ThumbRegisterInfo *>(STI.getRegisterInfo());
  const Thumb1InstrInfo &TII =
      *static_cast<const Thumb1InstrInfo *>(STI.getInstrInfo());
```
- EN: Implements `Thumb1FrameLowering::emitPrologue`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::emitPrologue`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 155-161
```cpp
  unsigned ArgRegsSaveSize = AFI->getArgRegsSaveSize();
  unsigned NumBytes = MFI.getStackSize();
  assert(NumBytes >= ArgRegsSaveSize &&
         "ArgRegsSaveSize is included in NumBytes");
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
  assert(STI.getPushPopSplitVariation(MF) == ARMSubtarget::SplitR7 &&
         "Must use R7 spilt for Thumb1");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-165
```cpp
  // Debug location must be unknown since the first debug location is used
  // to determine the end of the prologue.
  DebugLoc dl;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 167-169
```cpp
  Register FramePtr = RegInfo->getFrameRegister(MF);
  Register BasePtr = RegInfo->getBaseRegister();
  int CFAOffset = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 171-173
```cpp
  // Thumb add/sub sp, imm8 instructions implicitly multiply the offset by 4.
  NumBytes = (NumBytes + 3) & ~3;
  MFI.setStackSize(NumBytes);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 175-179
```cpp
  // Determine the sizes of each callee-save spill areas and record which frame
  // belongs to which callee-save spill areas.
  unsigned FRSize = 0, GPRCS1Size = 0, GPRCS2Size = 0, DPRCSSize = 0;
  int FramePtrSpillFI = 0;
  CFIInstBuilder CFIBuilder(MBB, MBBI, MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 181-186
```cpp
  if (ArgRegsSaveSize) {
    emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo, -ArgRegsSaveSize,
                                 ARM::NoRegister, MachineInstr::FrameSetup);
    CFAOffset += ArgRegsSaveSize;
    CFIBuilder.buildDefCFAOffset(CFAOffset);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 188-197
```cpp
  if (!AFI->hasStackFrame()) {
    if (NumBytes - ArgRegsSaveSize != 0) {
      emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo,
                                   -(NumBytes - ArgRegsSaveSize),
                                   ARM::NoRegister, MachineInstr::FrameSetup);
      CFAOffset += NumBytes - ArgRegsSaveSize;
      CFIBuilder.buildDefCFAOffset(CFAOffset);
    }
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 199-199
```cpp
  bool HasFrameRecordArea = hasFP(MF) && ARM::hGPRRegClass.contains(FramePtr);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 201-215
```cpp
  for (const CalleeSavedInfo &I : CSI) {
    MCRegister Reg = I.getReg();
    int FI = I.getFrameIdx();
    if (Reg == FramePtr.asMCReg())
      FramePtrSpillFI = FI;
    switch (Reg) {
    case ARM::R11:
      if (HasFrameRecordArea) {
        FRSize += 4;
        break;
      }
      [[fallthrough]];
    case ARM::R8:
    case ARM::R9:
    case ARM::R10:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 216-230
```cpp
      GPRCS2Size += 4;
      break;
    case ARM::LR:
      if (HasFrameRecordArea) {
        FRSize += 4;
        break;
      }
      [[fallthrough]];
    case ARM::R4:
    case ARM::R5:
    case ARM::R6:
    case ARM::R7:
      GPRCS1Size += 4;
      break;
    default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 231-233
```cpp
      DPRCSSize += 8;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 235-243
```cpp
  MachineBasicBlock::iterator FRPush, GPRCS1Push, GPRCS2Push;
  if (HasFrameRecordArea) {
    // Skip Frame Record setup:
    //   push {lr}
    //   mov lr, r11
    //   push {lr}
    std::advance(MBBI, 2);
    FRPush = MBBI++;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 245-248
```cpp
  if (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tPUSH) {
    GPRCS1Push = MBBI;
    ++MBBI;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 250-264
```cpp
  // Find last push instruction for GPRCS2 - spilling of high registers
  // (r8-r11) could consist of multiple tPUSH and tMOVr instructions.
  while (true) {
    MachineBasicBlock::iterator OldMBBI = MBBI;
    // Skip a run of tMOVr instructions
    while (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tMOVr &&
           MBBI->getFlag(MachineInstr::FrameSetup))
      MBBI++;
    if (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tPUSH &&
        MBBI->getFlag(MachineInstr::FrameSetup)) {
      GPRCS2Push = MBBI;
      MBBI++;
    } else {
      // We have reached an instruction which is not a push, so the previous
      // run of tMOVr instructions (which may have been empty) was not part of
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 265-269
```cpp
      // the prologue. Reset MBBI back to the last PUSH of the prologue.
      MBBI = OldMBBI;
      break;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 271-283
```cpp
  // Skip past this code sequence, which is emitted to restore the LR if it is
  // live-in and clobbered by the frame record setup code:
  //   ldr rX, [sp, #Y]
  //   mov lr, rX
  if (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tLDRspi &&
      MBBI->getFlag(MachineInstr::FrameSetup)) {
    ++MBBI;
    if (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tMOVr &&
        MBBI->getOperand(0).getReg() == ARM::LR &&
        MBBI->getFlag(MachineInstr::FrameSetup)) {
      ++MBBI;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 285-299
```cpp
  // Determine starting offsets of spill areas.
  unsigned DPRCSOffset = NumBytes - ArgRegsSaveSize -
                         (FRSize + GPRCS1Size + GPRCS2Size + DPRCSSize);
  unsigned GPRCS2Offset = DPRCSOffset + DPRCSSize;
  unsigned GPRCS1Offset = GPRCS2Offset + GPRCS2Size;
  bool HasFP = hasFP(MF);
  if (HasFP)
    AFI->setFramePtrSpillOffset(MFI.getObjectOffset(FramePtrSpillFI) +
                                NumBytes);
  if (HasFrameRecordArea)
    AFI->setFrameRecordSavedAreaSize(FRSize);
  AFI->setGPRCalleeSavedArea1Offset(GPRCS1Offset);
  AFI->setGPRCalleeSavedArea2Offset(GPRCS2Offset);
  AFI->setDPRCalleeSavedArea1Offset(DPRCSOffset);
  NumBytes = DPRCSOffset;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 301-309
```cpp
  int FramePtrOffsetInBlock = 0;
  unsigned adjustedGPRCS1Size = GPRCS1Size;
  if (GPRCS1Size > 0 && GPRCS2Size == 0 &&
      tryFoldSPUpdateIntoPushPop(STI, MF, &*(GPRCS1Push), NumBytes)) {
    FramePtrOffsetInBlock = NumBytes;
    adjustedGPRCS1Size += NumBytes;
    NumBytes = 0;
  }
  CFAOffset += adjustedGPRCS1Size;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 311-325
```cpp
  // Adjust FP so it point to the stack slot that contains the previous FP.
  if (HasFP) {
    MachineBasicBlock::iterator AfterPush =
        HasFrameRecordArea ? std::next(FRPush) : std::next(GPRCS1Push);
    if (HasFrameRecordArea) {
      // We have just finished pushing the previous FP into the stack,
      // so simply capture the SP value as the new Frame Pointer.
      BuildMI(MBB, AfterPush, dl, TII.get(ARM::tMOVr), FramePtr)
          .addReg(ARM::SP)
          .setMIFlags(MachineInstr::FrameSetup)
          .add(predOps(ARMCC::AL));
    } else {
      FramePtrOffsetInBlock +=
          MFI.getObjectOffset(FramePtrSpillFI) + GPRCS1Size + ArgRegsSaveSize;
      BuildMI(MBB, AfterPush, dl, TII.get(ARM::tADDrSPi), FramePtr)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 326-330
```cpp
          .addReg(ARM::SP)
          .addImm(FramePtrOffsetInBlock / 4)
          .setMIFlags(MachineInstr::FrameSetup)
          .add(predOps(ARMCC::AL));
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 332-341
```cpp
    CFIBuilder.setInsertPoint(AfterPush);
    if (FramePtrOffsetInBlock)
      CFIBuilder.buildDefCFA(FramePtr, CFAOffset - FramePtrOffsetInBlock);
    else
      CFIBuilder.buildDefCFARegister(FramePtr);
    if (NumBytes > 508)
      // If offset is > 508 then sp cannot be adjusted in a single instruction,
      // try restoring from fp instead.
      AFI->setShouldRestoreSPFromFP(true);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 343-357
```cpp
  // Emit call frame information for the callee-saved low registers.
  if (GPRCS1Size > 0) {
    CFIBuilder.setInsertPoint(std::next(GPRCS1Push));
    if (adjustedGPRCS1Size)
      CFIBuilder.buildDefCFAOffset(CFAOffset);
    for (const CalleeSavedInfo &I : CSI) {
      switch (I.getReg()) {
      case ARM::R8:
      case ARM::R9:
      case ARM::R10:
      case ARM::R11:
      case ARM::R12:
        break;
      case ARM::R0:
      case ARM::R1:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 358-370
```cpp
      case ARM::R2:
      case ARM::R3:
      case ARM::R4:
      case ARM::R5:
      case ARM::R6:
      case ARM::R7:
      case ARM::LR:
        CFIBuilder.buildOffset(I.getReg(),
                               MFI.getObjectOffset(I.getFrameIdx()));
        break;
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 372-386
```cpp
  // Emit call frame information for the callee-saved high registers.
  if (GPRCS2Size > 0) {
    CFIBuilder.setInsertPoint(std::next(GPRCS2Push));
    for (auto &I : CSI) {
      switch (I.getReg()) {
      case ARM::R8:
      case ARM::R9:
      case ARM::R10:
      case ARM::R11:
      case ARM::R12:
        CFIBuilder.buildOffset(I.getReg(),
                               MFI.getObjectOffset(I.getFrameIdx()));
        break;
      default:
        break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 387-389
```cpp
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 391-405
```cpp
  if (NumBytes) {
    // Insert it after all the callee-save spills.
    //
    // For a large stack frame, we might need a scratch register to store
    // the size of the frame.  We know all callee-save registers are free
    // at this point in the prologue, so pick one.
    unsigned ScratchRegister = ARM::NoRegister;
    for (auto &I : CSI) {
      MCRegister Reg = I.getReg();
      if (isARMLowRegister(Reg) && !(HasFP && Reg == FramePtr.asMCReg())) {
        ScratchRegister = Reg;
        break;
      }
    }
    emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo, -NumBytes,
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 406-411
```cpp
                                 ScratchRegister, MachineInstr::FrameSetup);
    if (!HasFP) {
      CFAOffset += NumBytes;
      CFIBuilder.buildDefCFAOffset(CFAOffset);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 413-415
```cpp
  if (STI.isTargetELF() && HasFP)
    MFI.setOffsetAdjustment(MFI.getOffsetAdjustment() -
                            AFI->getFramePtrSpillOffset());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 417-419
```cpp
  AFI->setGPRCalleeSavedArea1Size(GPRCS1Size);
  AFI->setGPRCalleeSavedArea2Size(GPRCS2Size);
  AFI->setDPRCalleeSavedArea1Size(DPRCSSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 421-431
```cpp
  if (RegInfo->hasStackRealignment(MF)) {
    const unsigned NrBitsToZero = Log2(MFI.getMaxAlign());
    // Emit the following sequence, using R4 as a temporary, since we cannot use
    // SP as a source or destination register for the shifts:
    // mov  r4, sp
    // lsrs r4, r4, #NrBitsToZero
    // lsls r4, r4, #NrBitsToZero
    // mov  sp, r4
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), ARM::R4)
      .addReg(ARM::SP, RegState::Kill)
      .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 433-437
```cpp
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tLSRri), ARM::R4)
      .addDef(ARM::CPSR)
      .addReg(ARM::R4, RegState::Kill)
      .addImm(NrBitsToZero)
      .add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 439-443
```cpp
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tLSLri), ARM::R4)
      .addDef(ARM::CPSR)
      .addReg(ARM::R4, RegState::Kill)
      .addImm(NrBitsToZero)
      .add(predOps(ARMCC::AL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 445-447
```cpp
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), ARM::SP)
      .addReg(ARM::R4, RegState::Kill)
      .add(predOps(ARMCC::AL));
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 449-450
```cpp
    AFI->setShouldRestoreSPFromFP(true);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 452-459
```cpp
  // If we need a base pointer, set it up here. It's whatever the value
  // of the stack pointer is at this point. Any variable size objects
  // will be allocated after this, so we can still use the base pointer
  // to reference locals.
  if (RegInfo->hasBasePointer(MF))
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), BasePtr)
        .addReg(ARM::SP)
        .add(predOps(ARMCC::AL));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 461-465
```cpp
  // If the frame has variable sized objects then the epilogue must restore
  // the sp from fp. We can assume there's an FP here since hasFP already
  // checks for hasVarSizedObjects.
  if (MFI.hasVarSizedObjects())
    AFI->setShouldRestoreSPFromFP(true);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 467-470
```cpp
  // In some cases, virtual registers have been introduced, e.g. by uses of
  // emitThumbRegPlusImmInReg.
  MF.getProperties().resetNoVRegs();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 472-481
```cpp
void Thumb1FrameLowering::emitEpilogue(MachineFunction &MF,
                                   MachineBasicBlock &MBB) const {
  MachineBasicBlock::iterator MBBI = MBB.getFirstTerminator();
  DebugLoc dl = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  const ThumbRegisterInfo *RegInfo =
      static_cast<const ThumbRegisterInfo *>(STI.getRegisterInfo());
  const Thumb1InstrInfo &TII =
      *static_cast<const Thumb1InstrInfo *>(STI.getInstrInfo());
```
- EN: Implements `Thumb1FrameLowering::emitEpilogue`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::emitEpilogue`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 483-487
```cpp
  unsigned ArgRegsSaveSize = AFI->getArgRegsSaveSize();
  int NumBytes = (int)MFI.getStackSize();
  assert((unsigned)NumBytes >= ArgRegsSaveSize &&
         "ArgRegsSaveSize is included in NumBytes");
  Register FramePtr = RegInfo->getFrameRegister(MF);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 489-502
```cpp
  if (!AFI->hasStackFrame()) {
    if (NumBytes - ArgRegsSaveSize != 0)
      emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo,
                                   NumBytes - ArgRegsSaveSize, ARM::NoRegister,
                                   MachineInstr::FrameDestroy);
  } else {
    // Unwind MBBI to point to first LDR / VLDRD.
    if (MBBI != MBB.begin()) {
      do
        --MBBI;
      while (MBBI != MBB.begin() && MBBI->getFlag(MachineInstr::FrameDestroy));
      if (!MBBI->getFlag(MachineInstr::FrameDestroy))
        ++MBBI;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 504-508
```cpp
    // Move SP to start of FP callee save spill area.
    NumBytes -=
        (AFI->getFrameRecordSavedAreaSize() +
         AFI->getGPRCalleeSavedArea1Size() + AFI->getGPRCalleeSavedArea2Size() +
         AFI->getDPRCalleeSavedArea1Size() + ArgRegsSaveSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 510-520
```cpp
    // We are likely to need a scratch register and we know all callee-save
    // registers are free at this point in the epilogue, so pick one.
    unsigned ScratchRegister = ARM::NoRegister;
    bool HasFP = hasFP(MF);
    for (auto &I : MFI.getCalleeSavedInfo()) {
      MCRegister Reg = I.getReg();
      if (isARMLowRegister(Reg) && !(HasFP && Reg == FramePtr.asMCReg())) {
        ScratchRegister = Reg;
        break;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 522-536
```cpp
    if (AFI->shouldRestoreSPFromFP()) {
      NumBytes = AFI->getFramePtrSpillOffset() - NumBytes;
      // Reset SP based on frame pointer only if the stack frame extends beyond
      // frame pointer stack slot, the target is ELF and the function has FP, or
      // the target uses var sized objects.
      if (NumBytes) {
        assert(ScratchRegister != ARM::NoRegister &&
               "No scratch register to restore SP from FP!");
        emitThumbRegPlusImmediate(MBB, MBBI, dl, ScratchRegister, FramePtr, -NumBytes,
                                  TII, *RegInfo, MachineInstr::FrameDestroy);
        BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), ARM::SP)
            .addReg(ScratchRegister)
            .add(predOps(ARMCC::AL))
            .setMIFlag(MachineInstr::FrameDestroy);
      } else
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 537-551
```cpp
        BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr), ARM::SP)
            .addReg(FramePtr)
            .add(predOps(ARMCC::AL))
            .setMIFlag(MachineInstr::FrameDestroy);
    } else {
      if (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tBX_RET &&
          &MBB.front() != &*MBBI && std::prev(MBBI)->getOpcode() == ARM::tPOP) {
        MachineBasicBlock::iterator PMBBI = std::prev(MBBI);
        if (!tryFoldSPUpdateIntoPushPop(STI, MF, &*PMBBI, NumBytes))
          emitPrologueEpilogueSPUpdate(MBB, PMBBI, TII, dl, *RegInfo, NumBytes,
                                       ScratchRegister, MachineInstr::FrameDestroy);
      } else if (!tryFoldSPUpdateIntoPushPop(STI, MF, &*MBBI, NumBytes))
        emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo, NumBytes,
                                     ScratchRegister, MachineInstr::FrameDestroy);
    }
```
- EN: Declares `BuildMI`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 552-552
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 554-559
```cpp
  if (needPopSpecialFixUp(MF)) {
    bool Done = emitPopSpecialFixUp(MBB, /* DoIt */ true);
    (void)Done;
    assert(Done && "Emission of the special fixup failed!?");
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 561-563
```cpp
bool Thumb1FrameLowering::canUseAsEpilogue(const MachineBasicBlock &MBB) const {
  if (!needPopSpecialFixUp(*MBB.getParent()))
    return true;
```
- EN: Implements `Thumb1FrameLowering::canUseAsEpilogue`, a query/helper routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::canUseAsEpilogue`，它是一个围绕机器基本块展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 565-567
```cpp
  MachineBasicBlock *TmpMBB = const_cast<MachineBasicBlock *>(&MBB);
  return emitPopSpecialFixUp(*TmpMBB, /* DoIt */ false);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 569-573
```cpp
bool Thumb1FrameLowering::needPopSpecialFixUp(const MachineFunction &MF) const {
  ARMFunctionInfo *AFI =
      const_cast<MachineFunction *>(&MF)->getInfo<ARMFunctionInfo>();
  if (AFI->getArgRegsSaveSize())
    return true;
```
- EN: Implements `Thumb1FrameLowering::needPopSpecialFixUp`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::needPopSpecialFixUp`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 575-578
```cpp
  // LR cannot be encoded with Thumb1, i.e., it requires a special fix-up.
  for (const CalleeSavedInfo &CSI : MF.getFrameInfo().getCalleeSavedInfo())
    if (CSI.getReg() == ARM::LR)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 580-581
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 583-597
```cpp
static void findTemporariesForLR(const BitVector &GPRsNoLRSP,
                                 const BitVector &PopFriendly,
                                 const LiveRegUnits &UsedRegs, unsigned &PopReg,
                                 unsigned &TmpReg, MachineRegisterInfo &MRI) {
  PopReg = TmpReg = 0;
  for (auto Reg : GPRsNoLRSP.set_bits()) {
    if (UsedRegs.available(Reg)) {
      // Remember the first pop-friendly register and exit.
      if (PopFriendly.test(Reg)) {
        PopReg = Reg;
        TmpReg = 0;
        break;
      }
      // Otherwise, remember that the register will be available to
      // save a pop-friendly register.
```
- EN: Implements `findTemporariesForLR`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `findTemporariesForLR`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 598-601
```cpp
      TmpReg = Reg;
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 603-610
```cpp
bool Thumb1FrameLowering::emitPopSpecialFixUp(MachineBasicBlock &MBB,
                                              bool DoIt) const {
  MachineFunction &MF = *MBB.getParent();
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  unsigned ArgRegsSaveSize = AFI->getArgRegsSaveSize();
  const TargetInstrInfo &TII = *STI.getInstrInfo();
  const ThumbRegisterInfo *RegInfo =
      static_cast<const ThumbRegisterInfo *>(STI.getRegisterInfo());
```
- EN: Implements `Thumb1FrameLowering::emitPopSpecialFixUp`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::emitPopSpecialFixUp`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 612-626
```cpp
  // If MBBI is a return instruction, or is a tPOP followed by a return
  // instruction in the successor BB, we may be able to directly restore
  // LR in the PC.
  // This is only possible with v5T ops (v4T can't change the Thumb bit via
  // a POP PC instruction), and only if we do not need to emit any SP update.
  // Otherwise, we need a temporary register to pop the value
  // and copy that value into LR.
  auto MBBI = MBB.getFirstTerminator();
  bool CanRestoreDirectly = STI.hasV5TOps() && !ArgRegsSaveSize;
  if (CanRestoreDirectly) {
    if (MBBI != MBB.end() && MBBI->getOpcode() != ARM::tB)
      CanRestoreDirectly = (MBBI->getOpcode() == ARM::tBX_RET ||
                            MBBI->getOpcode() == ARM::tPOP_RET);
    else {
      auto MBBI_prev = MBBI;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 627-635
```cpp
      MBBI_prev--;
      assert(MBBI_prev->getOpcode() == ARM::tPOP);
      assert(MBB.succ_size() == 1);
      if ((*MBB.succ_begin())->begin()->getOpcode() == ARM::tBX_RET)
        MBBI = MBBI_prev; // Replace the final tPOP with a tPOP_RET.
      else
        CanRestoreDirectly = false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 637-651
```cpp
  if (CanRestoreDirectly) {
    if (!DoIt || MBBI->getOpcode() == ARM::tPOP_RET)
      return true;
    MachineInstrBuilder MIB =
        BuildMI(MBB, MBBI, MBBI->getDebugLoc(), TII.get(ARM::tPOP_RET))
            .add(predOps(ARMCC::AL))
            .setMIFlag(MachineInstr::FrameDestroy);
    // Copy implicit ops and popped registers, if any.
    for (auto MO: MBBI->operands())
      if (MO.isReg() && (MO.isImplicit() || MO.isDef()))
        MIB.add(MO);
    MIB.addReg(ARM::PC, RegState::Define);
    // Erase the old instruction (tBX_RET or tPOP).
    MBB.erase(MBBI);
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 652-652
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 654-665
```cpp
  // Look for a temporary register to use.
  // First, compute the liveness information.
  const TargetRegisterInfo &TRI = *STI.getRegisterInfo();
  LiveRegUnits UsedRegs(TRI);
  UsedRegs.addLiveOuts(MBB);
  // The semantic of pristines changed recently and now,
  // the callee-saved registers that are touched in the function
  // are not part of the pristines set anymore.
  // Add those callee-saved now.
  const MCPhysReg *CSRegs = TRI.getCalleeSavedRegs(&MF);
  for (unsigned i = 0; CSRegs[i]; ++i)
    UsedRegs.addReg(CSRegs[i]);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 667-675
```cpp
  DebugLoc dl = DebugLoc();
  if (MBBI != MBB.end()) {
    dl = MBBI->getDebugLoc();
    auto InstUpToMBBI = MBB.end();
    while (InstUpToMBBI != MBBI)
      // The pre-decrement is on purpose here.
      // We want to have the liveness right before MBBI.
      UsedRegs.stepBackward(*--InstUpToMBBI);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 677-682
```cpp
  // Look for a register that can be directly use in the POP.
  unsigned PopReg = 0;
  // And some temporary register, just in case.
  unsigned TemporaryReg = 0;
  BitVector PopFriendly =
      TRI.getAllocatableSet(MF, TRI.getRegClass(ARM::tGPRRegClassID));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 684-694
```cpp
  assert(PopFriendly.any() && "No allocatable pop-friendly register?!");
  // Rebuild the GPRs from the high registers because they are removed
  // form the GPR reg class for thumb1.
  BitVector GPRsNoLRSP =
      TRI.getAllocatableSet(MF, TRI.getRegClass(ARM::hGPRRegClassID));
  GPRsNoLRSP |= PopFriendly;
  GPRsNoLRSP.reset(ARM::LR);
  GPRsNoLRSP.reset(ARM::SP);
  GPRsNoLRSP.reset(ARM::PC);
  findTemporariesForLR(GPRsNoLRSP, PopFriendly, UsedRegs, PopReg, TemporaryReg,
                       MF.getRegInfo());
```
- EN: Declares `for`, packaging target-specific state and APIs around `Thumb1FrameLowering`.
- CN: 这里声明 `for`，把与 `Thumb1FrameLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 696-710
```cpp
  // If we couldn't find a pop-friendly register, try restoring LR before
  // popping the other callee-saved registers, so we could use one of them as a
  // temporary.
  bool UseLDRSP = false;
  if (!PopReg && MBBI != MBB.begin()) {
    auto PrevMBBI = MBBI;
    PrevMBBI--;
    if (PrevMBBI->getOpcode() == ARM::tPOP) {
      UsedRegs.stepBackward(*PrevMBBI);
      findTemporariesForLR(GPRsNoLRSP, PopFriendly, UsedRegs, PopReg,
                           TemporaryReg, MF.getRegInfo());
      if (PopReg) {
        MBBI = PrevMBBI;
        UseLDRSP = true;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 711-712
```cpp
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 714-715
```cpp
  if (!DoIt && !PopReg && !TemporaryReg)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 717-717
```cpp
  assert((PopReg || TemporaryReg) && "Cannot get LR");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 719-733
```cpp
  if (UseLDRSP) {
    assert(PopReg && "Do not know how to get LR");
    // Load the LR via LDR tmp, [SP, #off]
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tLDRspi))
      .addReg(PopReg, RegState::Define)
      .addReg(ARM::SP)
      .addImm(MBBI->getNumExplicitOperands() - 2)
      .add(predOps(ARMCC::AL))
      .setMIFlag(MachineInstr::FrameDestroy);
    // Move from the temporary register to the LR.
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr))
      .addReg(ARM::LR, RegState::Define)
      .addReg(PopReg, RegState::Kill)
      .add(predOps(ARMCC::AL))
      .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 734-741
```cpp
    // Advance past the pop instruction.
    MBBI++;
    // Increment the SP.
    emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo,
                                 ArgRegsSaveSize + 4, ARM::NoRegister,
                                 MachineInstr::FrameDestroy);
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 743-751
```cpp
  if (TemporaryReg) {
    assert(!PopReg && "Unnecessary MOV is about to be inserted");
    PopReg = PopFriendly.find_first();
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr))
        .addReg(TemporaryReg, RegState::Define)
        .addReg(PopReg, RegState::Kill)
        .add(predOps(ARMCC::AL))
        .setMIFlag(MachineInstr::FrameDestroy);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 753-767
```cpp
  if (MBBI != MBB.end() && MBBI->getOpcode() == ARM::tPOP_RET) {
    // We couldn't use the direct restoration above, so
    // perform the opposite conversion: tPOP_RET to tPOP.
    MachineInstrBuilder MIB =
        BuildMI(MBB, MBBI, MBBI->getDebugLoc(), TII.get(ARM::tPOP))
            .add(predOps(ARMCC::AL))
            .setMIFlag(MachineInstr::FrameDestroy);
    bool Popped = false;
    for (auto MO: MBBI->operands())
      if (MO.isReg() && (MO.isImplicit() || MO.isDef()) &&
          MO.getReg() != ARM::PC) {
        MIB.add(MO);
        if (!MO.isImplicit())
          Popped = true;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 768-776
```cpp
    // Is there anything left to pop?
    if (!Popped)
      MBB.erase(MIB.getInstr());
    // Erase the old instruction.
    MBB.erase(MBBI);
    MBBI = BuildMI(MBB, MBB.end(), dl, TII.get(ARM::tBX_RET))
               .add(predOps(ARMCC::AL))
               .setMIFlag(MachineInstr::FrameDestroy);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 778-782
```cpp
  assert(PopReg && "Do not know how to get LR");
  BuildMI(MBB, MBBI, dl, TII.get(ARM::tPOP))
      .add(predOps(ARMCC::AL))
      .addReg(PopReg, RegState::Define)
      .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 784-785
```cpp
  emitPrologueEpilogueSPUpdate(MBB, MBBI, TII, dl, *RegInfo, ArgRegsSaveSize,
                               ARM::NoRegister, MachineInstr::FrameDestroy);
```
- EN: Declares `emitPrologueEpilogueSPUpdate`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitPrologueEpilogueSPUpdate`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 787-791
```cpp
  BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr))
      .addReg(ARM::LR, RegState::Define)
      .addReg(PopReg, RegState::Kill)
      .add(predOps(ARMCC::AL))
      .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 793-798
```cpp
  if (TemporaryReg)
    BuildMI(MBB, MBBI, dl, TII.get(ARM::tMOVr))
        .addReg(PopReg, RegState::Define)
        .addReg(TemporaryReg, RegState::Kill)
        .add(predOps(ARMCC::AL))
        .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 800-801
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 803-809
```cpp
static const SmallVector<Register> OrderedLowRegs = {ARM::R4, ARM::R5, ARM::R6,
                                                     ARM::R7, ARM::LR};
static const SmallVector<Register> OrderedHighRegs = {ARM::R8, ARM::R9,
                                                      ARM::R10, ARM::R11};
static const SmallVector<Register> OrderedCopyRegs = {
    ARM::R0, ARM::R1, ARM::R2, ARM::R3, ARM::R4,
    ARM::R5, ARM::R6, ARM::R7, ARM::LR};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 811-823
```cpp
static void splitLowAndHighRegs(const std::set<Register> &Regs,
                                std::set<Register> &LowRegs,
                                std::set<Register> &HighRegs) {
  for (Register Reg : Regs) {
    if (ARM::tGPRRegClass.contains(Reg) || Reg == ARM::LR) {
      LowRegs.insert(Reg);
    } else if (ARM::hGPRRegClass.contains(Reg) && Reg != ARM::LR) {
      HighRegs.insert(Reg);
    } else {
      llvm_unreachable("callee-saved register of unexpected class");
    }
  }
}
```
- EN: Implements `splitLowAndHighRegs`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `splitLowAndHighRegs`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 825-830
```cpp
template <typename It>
It getNextOrderedReg(It OrderedStartIt, It OrderedEndIt,
                     const std::set<Register> &RegSet) {
  return std::find_if(OrderedStartIt, OrderedEndIt,
                      [&](Register Reg) { return RegSet.count(Reg); });
}
```
- EN: Implements `std::find_if`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `std::find_if`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 832-840
```cpp
static void pushRegsToStack(MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator MI,
                            const TargetInstrInfo &TII,
                            const std::set<Register> &RegsToSave,
                            const std::set<Register> &CopyRegs,
                            bool &UsedLRAsTemp) {
  MachineFunction &MF = *MBB.getParent();
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  DebugLoc DL;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 842-843
```cpp
  std::set<Register> LowRegs, HighRegs;
  splitLowAndHighRegs(RegsToSave, LowRegs, HighRegs);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 845-853
```cpp
  // Push low regs first
  if (!LowRegs.empty()) {
    MachineInstrBuilder MIB =
        BuildMI(MBB, MI, DL, TII.get(ARM::tPUSH)).add(predOps(ARMCC::AL));
    for (unsigned Reg : OrderedLowRegs) {
      if (LowRegs.count(Reg)) {
        bool isKill = !MRI.isLiveIn(Reg);
        if (isKill && !MRI.isReserved(Reg))
          MBB.addLiveIn(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 855-859
```cpp
        MIB.addReg(Reg, getKillRegState(isKill));
      }
    }
    MIB.setMIFlags(MachineInstr::FrameSetup);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 861-865
```cpp
  // Now push the high registers
  // There are no store instructions that can access high registers directly,
  // so we have to move them to low registers, and push them.
  // This might take multiple pushes, as it is possible for there to
  // be fewer low registers available than high registers which need saving.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 867-874
```cpp
  // Find the first register to save.
  // Registers must be processed in reverse order so that in case we need to use
  // multiple PUSH instructions, the order of the registers on the stack still
  // matches the unwind info. They need to be switched back to ascending order
  // before adding to the PUSH instruction.
  auto HiRegToSave = getNextOrderedReg(OrderedHighRegs.rbegin(),
                                       OrderedHighRegs.rend(),
                                       HighRegs);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 876-880
```cpp
  while (HiRegToSave != OrderedHighRegs.rend()) {
    // Find the first low register to use.
    auto CopyRegIt = getNextOrderedReg(OrderedCopyRegs.rbegin(),
                                       OrderedCopyRegs.rend(),
                                       CopyRegs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 882-885
```cpp
    // Create the PUSH, but don't insert it yet (the MOVs need to come first).
    MachineInstrBuilder PushMIB = BuildMI(MF, DL, TII.get(ARM::tPUSH))
                                      .add(predOps(ARMCC::AL))
                                      .setMIFlags(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 887-895
```cpp
    SmallVector<unsigned, 4> RegsToPush;
    while (HiRegToSave != OrderedHighRegs.rend() &&
           CopyRegIt != OrderedCopyRegs.rend()) {
      if (HighRegs.count(*HiRegToSave)) {
        bool isKill = !MRI.isLiveIn(*HiRegToSave);
        if (isKill && !MRI.isReserved(*HiRegToSave))
          MBB.addLiveIn(*HiRegToSave);
        if (*CopyRegIt == ARM::LR)
          UsedLRAsTemp = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 897-902
```cpp
        // Emit a MOV from the high reg to the low reg.
        BuildMI(MBB, MI, DL, TII.get(ARM::tMOVr))
            .addReg(*CopyRegIt, RegState::Define)
            .addReg(*HiRegToSave, getKillRegState(isKill))
            .add(predOps(ARMCC::AL))
            .setMIFlags(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 904-905
```cpp
        // Record the register that must be added to the PUSH.
        RegsToPush.push_back(*CopyRegIt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 907-914
```cpp
        CopyRegIt = getNextOrderedReg(std::next(CopyRegIt),
                                      OrderedCopyRegs.rend(),
                                      CopyRegs);
        HiRegToSave = getNextOrderedReg(std::next(HiRegToSave),
                                        OrderedHighRegs.rend(),
                                        HighRegs);
      }
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 916-918
```cpp
    // Add the low registers to the PUSH, in ascending order.
    for (unsigned Reg : llvm::reverse(RegsToPush))
      PushMIB.addReg(Reg, RegState::Kill);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 920-923
```cpp
    // Insert the PUSH instruction after the MOVs.
    MBB.insert(MI, PushMIB);
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 925-932
```cpp
static void popRegsFromStack(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator &MI,
                             const TargetInstrInfo &TII,
                             const std::set<Register> &RegsToRestore,
                             const std::set<Register> &AvailableCopyRegs,
                             bool IsVarArg, bool HasV5Ops) {
  if (RegsToRestore.empty())
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 934-936
```cpp
  MachineFunction &MF = *MBB.getParent();
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  DebugLoc DL = MI != MBB.end() ? MI->getDebugLoc() : DebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 938-939
```cpp
  std::set<Register> LowRegs, HighRegs;
  splitLowAndHighRegs(RegsToRestore, LowRegs, HighRegs);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 941-945
```cpp
  // Pop the high registers first
  // There are no store instructions that can access high registers directly,
  // so we have to pop into low registers and them move to  the high registers.
  // This might take multiple pops, as it is possible for there to
  // be fewer low registers available than high registers which need restoring.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 947-950
```cpp
  // Find the first register to restore.
  auto HiRegToRestore = getNextOrderedReg(OrderedHighRegs.begin(),
                                          OrderedHighRegs.end(),
                                          HighRegs);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 952-964
```cpp
  std::set<Register> CopyRegs = AvailableCopyRegs;
  Register LowScratchReg;
  if (!HighRegs.empty() && CopyRegs.empty()) {
    // No copy regs are available to pop high regs. Let's make use of a return
    // register and the scratch register (IP/R12) to copy things around.
    LowScratchReg = ARM::R0;
    BuildMI(MBB, MI, DL, TII.get(ARM::tMOVr))
        .addReg(ARM::R12, RegState::Define)
        .addReg(LowScratchReg, RegState::Kill)
        .add(predOps(ARMCC::AL))
        .setMIFlag(MachineInstr::FrameDestroy);
    CopyRegs.insert(LowScratchReg);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 966-971
```cpp
  while (HiRegToRestore != OrderedHighRegs.end()) {
    assert(!CopyRegs.empty());
    // Find the first low register to use.
    auto CopyReg = getNextOrderedReg(OrderedCopyRegs.begin(),
                                     OrderedCopyRegs.end(),
                                     CopyRegs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 973-976
```cpp
    // Create the POP instruction.
    MachineInstrBuilder PopMIB = BuildMI(MBB, MI, DL, TII.get(ARM::tPOP))
                                     .add(predOps(ARMCC::AL))
                                     .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 978-981
```cpp
    while (HiRegToRestore != OrderedHighRegs.end() &&
           CopyReg != OrderedCopyRegs.end()) {
      // Add the low register to the POP.
      PopMIB.addReg(*CopyReg, RegState::Define);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 983-988
```cpp
      // Create the MOV from low to high register.
      BuildMI(MBB, MI, DL, TII.get(ARM::tMOVr))
          .addReg(*HiRegToRestore, RegState::Define)
          .addReg(*CopyReg, RegState::Kill)
          .add(predOps(ARMCC::AL))
          .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 990-997
```cpp
      CopyReg = getNextOrderedReg(std::next(CopyReg),
                                  OrderedCopyRegs.end(),
                                  CopyRegs);
      HiRegToRestore = getNextOrderedReg(std::next(HiRegToRestore),
                                         OrderedHighRegs.end(),
                                         HighRegs);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 999-1006
```cpp
  // Restore low register used as scratch if necessary
  if (LowScratchReg.isValid()) {
    BuildMI(MBB, MI, DL, TII.get(ARM::tMOVr))
        .addReg(LowScratchReg, RegState::Define)
        .addReg(ARM::R12, RegState::Kill)
        .add(predOps(ARMCC::AL))
        .setMIFlag(MachineInstr::FrameDestroy);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1008-1012
```cpp
  // Now pop the low registers
  if (!LowRegs.empty()) {
    MachineInstrBuilder MIB = BuildMI(MF, DL, TII.get(ARM::tPOP))
                                  .add(predOps(ARMCC::AL))
                                  .setMIFlag(MachineInstr::FrameDestroy);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1014-1017
```cpp
    bool NeedsPop = false;
    for (Register Reg : OrderedLowRegs) {
      if (!LowRegs.count(Reg))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1019-1033
```cpp
      if (Reg == ARM::LR) {
        if (!MBB.succ_empty() || MI->getOpcode() == ARM::TCRETURNdi ||
            MI->getOpcode() == ARM::TCRETURNri ||
            MI->getOpcode() == ARM::TCRETURNrinotr12)
          // LR may only be popped into PC, as part of return sequence.
          // If this isn't the return sequence, we'll need emitPopSpecialFixUp
          // to restore LR the hard way.
          // FIXME: if we don't pass any stack arguments it would be actually
          // advantageous *and* correct to do the conversion to an ordinary call
          // instruction here.
          continue;
        // Special epilogue for vararg functions. See emitEpilogue
        if (IsVarArg)
          continue;
        // ARMv4T requires BX, see emitEpilogue
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1034-1035
```cpp
        if (!HasV5Ops)
          continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1037-1039
```cpp
        // CMSE entry functions must return via BXNS, see emitEpilogue.
        if (AFI->isCmseNSEntryFunction())
          continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1041-1050
```cpp
        // Pop LR into PC.
        Reg = ARM::PC;
        (*MIB).setDesc(TII.get(ARM::tPOP_RET));
        if (MI != MBB.end())
          MIB.copyImplicitOps(*MI);
        MI = MBB.erase(MI);
      }
      MIB.addReg(Reg, getDefRegState(true));
      NeedsPop = true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1052-1058
```cpp
    // It's illegal to emit pop instruction without operands.
    if (NeedsPop)
      MBB.insert(MI, &*MIB);
    else
      MF.deleteMachineInstr(MIB);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1060-1064
```cpp
bool Thumb1FrameLowering::spillCalleeSavedRegisters(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
    ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
  if (CSI.empty())
    return false;
```
- EN: Implements `Thumb1FrameLowering::spillCalleeSavedRegisters`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::spillCalleeSavedRegisters`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1066-1070
```cpp
  const TargetInstrInfo &TII = *STI.getInstrInfo();
  MachineFunction &MF = *MBB.getParent();
  const ARMBaseRegisterInfo *RegInfo = static_cast<const ARMBaseRegisterInfo *>(
      MF.getSubtarget().getRegisterInfo());
  Register FPReg = RegInfo->getFrameRegister(MF);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1072-1076
```cpp
  // In case FP is a high reg, we need a separate push sequence to generate
  // a correct Frame Record
  bool NeedsFrameRecordPush = hasFP(MF) && ARM::hGPRRegClass.contains(FPReg);
  bool LRLiveIn = MF.getRegInfo().isLiveIn(ARM::LR);
  bool UsedLRAsTemp = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1078-1086
```cpp
  std::set<Register> FrameRecord;
  std::set<Register> SpilledGPRs;
  for (const CalleeSavedInfo &I : CSI) {
    MCRegister Reg = I.getReg();
    if (NeedsFrameRecordPush && (Reg == FPReg.asMCReg() || Reg == ARM::LR))
      FrameRecord.insert(Reg);
    else
      SpilledGPRs.insert(Reg);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1088-1101
```cpp
  // Determine intermediate registers which can be used for pushing the frame
  // record:
  // - Unused argument registers
  // - LR: This is possible because the first PUSH will save it on the stack,
  //       so it is free to be used as a temporary for the second. However, it
  //       is possible for LR to be live-in to the function, in which case we
  //       will need to restore it later in the prologue, so we only use this
  //       if there are no free argument registers.
  std::set<Register> FrameRecordCopyRegs;
  for (unsigned ArgReg : {ARM::R0, ARM::R1, ARM::R2, ARM::R3})
    if (!MF.getRegInfo().isLiveIn(ArgReg))
      FrameRecordCopyRegs.insert(ArgReg);
  if (FrameRecordCopyRegs.empty())
    FrameRecordCopyRegs.insert(ARM::LR);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1103-1103
```cpp
  pushRegsToStack(MBB, MI, TII, FrameRecord, FrameRecordCopyRegs, UsedLRAsTemp);
```
- EN: Declares `pushRegsToStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `pushRegsToStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1105-1115
```cpp
  // Determine intermediate registers which can be used for pushing high regs:
  // - Spilled low regs
  // - Unused argument registers
  std::set<Register> CopyRegs;
  for (Register Reg : SpilledGPRs)
    if ((ARM::tGPRRegClass.contains(Reg) || Reg == ARM::LR) &&
        !MF.getRegInfo().isLiveIn(Reg) && !(hasFP(MF) && Reg == FPReg))
      CopyRegs.insert(Reg);
  for (unsigned ArgReg : {ARM::R0, ARM::R1, ARM::R2, ARM::R3})
    if (!MF.getRegInfo().isLiveIn(ArgReg))
      CopyRegs.insert(ArgReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1117-1117
```cpp
  pushRegsToStack(MBB, MI, TII, SpilledGPRs, CopyRegs, UsedLRAsTemp);
```
- EN: Declares `pushRegsToStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `pushRegsToStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1119-1131
```cpp
  // If the push sequence used LR as a temporary, and LR is live-in (for
  // example because it is used by the llvm.returnaddress intrinsic), then we
  // need to reload it from the stack. Thumb1 does not have a load instruction
  // which can use LR, so we need to load into a temporary low register and
  // copy to LR.
  if (LRLiveIn && UsedLRAsTemp) {
    auto CopyRegIt = getNextOrderedReg(OrderedCopyRegs.rbegin(),
                                       OrderedCopyRegs.rend(), CopyRegs);
    assert(CopyRegIt != OrderedCopyRegs.rend());
    unsigned NumRegsPushed = FrameRecord.size() + SpilledGPRs.size();
    LLVM_DEBUG(
        dbgs() << "LR is live-in but clobbered in prologue, restoring via "
               << RegInfo->getName(*CopyRegIt) << "\n");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1133-1137
```cpp
    BuildMI(MBB, MI, DebugLoc(), TII.get(ARM::tLDRspi), *CopyRegIt)
        .addReg(ARM::SP)
        .addImm(NumRegsPushed - 1)
        .add(predOps(ARMCC::AL))
        .setMIFlags(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1139-1143
```cpp
    BuildMI(MBB, MI, DebugLoc(), TII.get(ARM::tMOVr), ARM::LR)
        .addReg(*CopyRegIt)
        .add(predOps(ARMCC::AL))
        .setMIFlags(MachineInstr::FrameSetup);
  }
```
- EN: Declares `BuildMI`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1145-1146
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1148-1152
```cpp
bool Thumb1FrameLowering::restoreCalleeSavedRegisters(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
    MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
  if (CSI.empty())
    return false;
```
- EN: Implements `Thumb1FrameLowering::restoreCalleeSavedRegisters`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Thumb1FrameLowering::restoreCalleeSavedRegisters`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1154-1160
```cpp
  MachineFunction &MF = *MBB.getParent();
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  const TargetInstrInfo &TII = *STI.getInstrInfo();
  const ARMBaseRegisterInfo *RegInfo = static_cast<const ARMBaseRegisterInfo *>(
      MF.getSubtarget().getRegisterInfo());
  bool IsVarArg = AFI->getArgRegsSaveSize() > 0;
  Register FPReg = RegInfo->getFrameRegister(MF);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1162-1164
```cpp
  // In case FP is a high reg, we need a separate pop sequence to generate
  // a correct Frame Record
  bool NeedsFrameRecordPop = hasFP(MF) && ARM::hGPRRegClass.contains(FPReg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1166-1173
```cpp
  std::set<Register> FrameRecord;
  std::set<Register> SpilledGPRs;
  for (CalleeSavedInfo &I : CSI) {
    MCRegister Reg = I.getReg();
    if (NeedsFrameRecordPop && (Reg == FPReg.asMCReg() || Reg == ARM::LR))
      FrameRecord.insert(Reg);
    else
      SpilledGPRs.insert(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1175-1177
```cpp
    if (Reg == ARM::LR)
      I.setRestored(false);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1179-1193
```cpp
  // Determine intermediate registers which can be used for popping high regs:
  // - Spilled low regs
  // - Unused return registers
  std::set<Register> CopyRegs;
  std::set<Register> UnusedReturnRegs;
  for (Register Reg : SpilledGPRs)
    if ((ARM::tGPRRegClass.contains(Reg)) && !(hasFP(MF) && Reg == FPReg))
      CopyRegs.insert(Reg);
  auto Terminator = MBB.getFirstTerminator();
  if (Terminator != MBB.end() && Terminator->getOpcode() == ARM::tBX_RET) {
    UnusedReturnRegs.insert(ARM::R0);
    UnusedReturnRegs.insert(ARM::R1);
    UnusedReturnRegs.insert(ARM::R2);
    UnusedReturnRegs.insert(ARM::R3);
    for (auto Op : Terminator->implicit_operands()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1194-1198
```cpp
      if (Op.isReg())
        UnusedReturnRegs.erase(Op.getReg());
    }
  }
  CopyRegs.insert(UnusedReturnRegs.begin(), UnusedReturnRegs.end());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1200-1202
```cpp
  // First pop regular spilled regs.
  popRegsFromStack(MBB, MI, TII, SpilledGPRs, CopyRegs, IsVarArg,
                   STI.hasV5TOps());
```
- EN: Declares `popRegsFromStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `popRegsFromStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1204-1207
```cpp
  // LR may only be popped into pc, as part of a return sequence.
  // Check that no other pop instructions are inserted after that.
  assert((!SpilledGPRs.count(ARM::LR) || FrameRecord.empty()) &&
         "Can't insert pop after return sequence");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1209-1212
```cpp
  // Now pop Frame Record regs.
  // Only unused return registers can be used as copy regs at this point.
  popRegsFromStack(MBB, MI, TII, FrameRecord, UnusedReturnRegs, IsVarArg,
                   STI.hasV5TOps());
```
- EN: Declares `popRegsFromStack`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `popRegsFromStack`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1214-1215
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: stack-frame layout and prologue/epilogue lowering.
  - CN: 核心职责：栈帧布局以及序言/结语生成。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `Thumb1FrameLowering.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `Thumb1InstrInfo.h`, `ThumbRegisterInfo.h`, `Utils/ARMBaseInfo.h`.
  - CN: 后端本地头文件：`Thumb1FrameLowering.h`, `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `Thumb1InstrInfo.h`, `ThumbRegisterInfo.h`, `Utils/ARMBaseInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/BitVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/CFIInstBuilder.h`, `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h` ... (+10 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/CFIInstBuilder.h`, `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h` ... (+10 more)。
- EN: Standard/system headers: `cassert`, `iterator`, `vector`.
  - CN: 标准库/系统头文件：`cassert`, `iterator`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
