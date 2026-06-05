# MSP430FrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430FrameLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements stack frame layout plus prologue/epilogue emission for the backend.
  - **CN**: 实现该后端的栈帧布局以及序言/尾声生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MSP430FrameLowering.cpp - MSP430 Frame Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the MSP430 implementation of TargetFrameLowering class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-23
```cpp

#include "MSP430FrameLowering.h"
#include "MSP430InstrInfo.h"
#include "MSP430MachineFunctionInfo.h"
#include "MSP430Subtarget.h"
#include "llvm/CodeGen/CFIInstBuilder.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/Target/TargetOptions.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430FrameLowering.h`, `MSP430InstrInfo.h`, `MSP430MachineFunctionInfo.h`, `MSP430Subtarget.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430FrameLowering.h`, `MSP430InstrInfo.h`, `MSP430MachineFunctionInfo.h`, `MSP430Subtarget.h`。

### Lines 24-33
```cpp
using namespace llvm;

MSP430FrameLowering::MSP430FrameLowering(const MSP430Subtarget &STI)
    : TargetFrameLowering(TargetFrameLowering::StackGrowsDown, Align(2), -2,
                          Align(2)),
      STI(STI), TII(*STI.getInstrInfo()), TRI(STI.getRegisterInfo()) {}

bool MSP430FrameLowering::hasFPImpl(const MachineFunction &MF) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-42
```cpp
  return (MF.getTarget().Options.DisableFramePointerElim(MF) ||
          MF.getFrameInfo().hasVarSizedObjects() ||
          MFI.isFrameAddressTaken());
}

bool MSP430FrameLowering::hasReservedCallFrame(const MachineFunction &MF) const {
  return !MF.getFrameInfo().hasVarSizedObjects();
}

```
- **EN**: Implements logic around `getFrameInfo`, `isFrameAddressTaken`, `hasReservedCallFrame`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameInfo`, `isFrameAddressTaken`, `hasReservedCallFrame` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 43-54
```cpp
void MSP430FrameLowering::BuildCFI(MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator MBBI,
                                   const DebugLoc &DL,
                                   const MCCFIInstruction &CFIInst,
                                   MachineInstr::MIFlag Flag) const {
  MachineFunction &MF = *MBB.getParent();
  unsigned CFIIndex = MF.addFrameInst(CFIInst);
  BuildMI(MBB, MBBI, DL, TII.get(TargetOpcode::CFI_INSTRUCTION))
      .addCFIIndex(CFIIndex)
      .setMIFlag(Flag);
}

```
- **EN**: Implements logic around `BuildCFI`, `getParent`, `addFrameInst`, `BuildMI`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildCFI`, `getParent`, `addFrameInst`, `BuildMI`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 55-64
```cpp
void MSP430FrameLowering::emitCalleeSavedFrameMoves(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
    const DebugLoc &DL, bool IsPrologue) const {
  MachineFunction &MF = *MBB.getParent();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  const MCRegisterInfo *MRI = MF.getContext().getRegisterInfo();

  // Add callee saved registers to move list.
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();

```
- **EN**: Implements logic around `emitCalleeSavedFrameMoves`, `getParent`, `getFrameInfo`, `getContext`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitCalleeSavedFrameMoves`, `getParent`, `getFrameInfo`, `getContext`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 65-80
```cpp
  // Calculate offsets.
  for (const CalleeSavedInfo &I : CSI) {
    int64_t Offset = MFI.getObjectOffset(I.getFrameIdx());
    MCRegister Reg = I.getReg();
    unsigned DwarfReg = MRI->getDwarfRegNum(Reg, true);

    if (IsPrologue) {
      BuildCFI(MBB, MBBI, DL,
               MCCFIInstruction::createOffset(nullptr, DwarfReg, Offset));
    } else {
      BuildCFI(MBB, MBBI, DL,
               MCCFIInstruction::createRestore(nullptr, DwarfReg));
    }
  }
}

```
- **EN**: Implements logic around `getObjectOffset`, `getReg`, `getDwarfRegNum`, `BuildCFI`, ...; this block applies conditional target rules.
- **CN**: 围绕 `getObjectOffset`, `getReg`, `getDwarfRegNum`, `BuildCFI`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 81-91
```cpp
void MSP430FrameLowering::emitPrologue(MachineFunction &MF,
                                       MachineBasicBlock &MBB) const {
  assert(&MF.front() == &MBB && "Shrink-wrapping not yet supported");
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MSP430MachineFunctionInfo *MSP430FI = MF.getInfo<MSP430MachineFunctionInfo>();
  const MSP430InstrInfo &TII =
      *static_cast<const MSP430InstrInfo *>(MF.getSubtarget().getInstrInfo());

  MachineBasicBlock::iterator MBBI = MBB.begin();
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();

```
- **EN**: Implements logic around `emitPrologue`, `assert`, `getFrameInfo`, `getInfo<MSP430MachineFunctionInfo>`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitPrologue`, `assert`, `getFrameInfo`, `getInfo<MSP430MachineFunctionInfo>`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 92-101
```cpp
  // Get the number of bytes to allocate from the FrameInfo.
  uint64_t StackSize = MFI.getStackSize();
  int stackGrowth = -2;

  uint64_t NumBytes = 0;
  if (hasFP(MF)) {
    // Calculate required stack adjustment
    uint64_t FrameSize = StackSize - 2;
    NumBytes = FrameSize - MSP430FI->getCalleeSavedFrameSize();

```
- **EN**: Implements logic around `getStackSize`, `getCalleeSavedFrameSize`; this block applies conditional target rules.
- **CN**: 围绕 `getStackSize`, `getCalleeSavedFrameSize` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 102-111
```cpp
    // Get the offset of the stack slot for the EBP register... which is
    // guaranteed to be the last slot by processFunctionBeforeFrameFinalized.
    // Update the frame offset adjustment.
    MFI.setOffsetAdjustment(-NumBytes);

    // Save FP into the appropriate stack slot...
    BuildMI(MBB, MBBI, DL, TII.get(MSP430::PUSH16r))
        .addReg(MSP430::R4, RegState::Kill)
        .setMIFlag(MachineInstr::FrameSetup);

```
- **EN**: Implements logic around `setOffsetAdjustment`, `BuildMI`, `addReg`, `setMIFlag`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `setOffsetAdjustment`, `BuildMI`, `addReg`, `setMIFlag` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 112-124
```cpp
    // Mark the place where FP was saved.
    // Define the current CFA rule to use the provided offset.
    BuildCFI(MBB, MBBI, DL,
             MCCFIInstruction::cfiDefCfaOffset(nullptr, -2 * stackGrowth),
             MachineInstr::FrameSetup);

    // Change the rule for the FramePtr to be an "offset" rule.
    unsigned DwarfFramePtr = TRI->getDwarfRegNum(MSP430::R4, true);
    BuildCFI(
        MBB, MBBI, DL,
        MCCFIInstruction::createOffset(nullptr, DwarfFramePtr, 2 * stackGrowth),
        MachineInstr::FrameSetup);

```
- **EN**: Implements logic around `BuildCFI`, `cfiDefCfaOffset`, `getDwarfRegNum`, `createOffset`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildCFI`, `cfiDefCfaOffset`, `getDwarfRegNum`, `createOffset` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 125-135
```cpp
    // Update FP with the new base value...
    BuildMI(MBB, MBBI, DL, TII.get(MSP430::MOV16rr), MSP430::R4)
        .addReg(MSP430::SP)
        .setMIFlag(MachineInstr::FrameSetup);

    // Mark effective beginning of when frame pointer becomes valid.
    // Define the current CFA to use the FP register.
    BuildCFI(MBB, MBBI, DL,
             MCCFIInstruction::createDefCfaRegister(nullptr, DwarfFramePtr),
             MachineInstr::FrameSetup);

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `setMIFlag`, `BuildCFI`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `setMIFlag`, `BuildCFI`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 136-147
```cpp
    // Mark the FramePtr as live-in in every block except the entry.
    for (MachineBasicBlock &MBBJ : llvm::drop_begin(MF))
      MBBJ.addLiveIn(MSP430::R4);
  } else
    NumBytes = StackSize - MSP430FI->getCalleeSavedFrameSize();

  // Skip the callee-saved push instructions.
  int StackOffset = 2 * stackGrowth;
  while (MBBI != MBB.end() && MBBI->getFlag(MachineInstr::FrameSetup) &&
         (MBBI->getOpcode() == MSP430::PUSH16r)) {
    ++MBBI;

```
- **EN**: Implements logic around `addLiveIn`, `getCalleeSavedFrameSize`, `getOpcode`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addLiveIn`, `getCalleeSavedFrameSize`, `getOpcode` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 148-158
```cpp
    if (!hasFP(MF)) {
      // Mark callee-saved push instruction.
      // Define the current CFA rule to use the provided offset.
      assert(StackSize && "Expected stack frame");
      BuildCFI(MBB, MBBI, DL,
               MCCFIInstruction::cfiDefCfaOffset(nullptr, -StackOffset),
               MachineInstr::FrameSetup);
      StackOffset += stackGrowth;
    }
  }

```
- **EN**: Implements logic around `assert`, `BuildCFI`, `cfiDefCfaOffset`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `BuildCFI`, `cfiDefCfaOffset` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 159-169
```cpp
  if (MBBI != MBB.end())
    DL = MBBI->getDebugLoc();

  if (NumBytes) { // adjust stack pointer: SP -= numbytes
    // If there is an SUB16ri of SP immediately before this instruction, merge
    // the two.
    //NumBytes -= mergeSPUpdates(MBB, MBBI, true);
    // If there is an ADD16ri or SUB16ri of SP immediately after this
    // instruction, merge the two instructions.
    // mergeSPUpdatesDown(MBB, MBBI, &NumBytes);

```
- **EN**: Implements logic around `getDebugLoc`; this block applies conditional target rules.
- **CN**: 围绕 `getDebugLoc` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 170-187
```cpp
    if (NumBytes) {
      MachineInstr *MI =
          BuildMI(MBB, MBBI, DL, TII.get(MSP430::SUB16ri), MSP430::SP)
              .addReg(MSP430::SP)
              .addImm(NumBytes)
              .setMIFlag(MachineInstr::FrameSetup);
      // The SRW implicit def is dead.
      MI->getOperand(3).setIsDead();
    }
    if (!hasFP(MF)) {
      // Adjust the previous CFA value if CFA was not redefined by FP
      BuildCFI(
          MBB, MBBI, DL,
          MCCFIInstruction::cfiDefCfaOffset(nullptr, StackSize - stackGrowth),
          MachineInstr::FrameSetup);
    }
  }

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`, `setMIFlag`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm`, `setMIFlag`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 188-197
```cpp
  emitCalleeSavedFrameMoves(MBB, MBBI, DL, true);
}

void MSP430FrameLowering::emitEpilogue(MachineFunction &MF,
                                       MachineBasicBlock &MBB) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  MSP430MachineFunctionInfo *MSP430FI = MF.getInfo<MSP430MachineFunctionInfo>();
  const MSP430InstrInfo &TII =
      *static_cast<const MSP430InstrInfo *>(MF.getSubtarget().getInstrInfo());

```
- **EN**: Implements logic around `emitCalleeSavedFrameMoves`, `emitEpilogue`, `getFrameInfo`, `getInfo<MSP430MachineFunctionInfo>`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitCalleeSavedFrameMoves`, `emitEpilogue`, `getFrameInfo`, `getInfo<MSP430MachineFunctionInfo>`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 198-208
```cpp
  MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
  unsigned RetOpcode = MBBI->getOpcode();
  DebugLoc DL = MBBI->getDebugLoc();

  switch (RetOpcode) {
  case MSP430::RET:
  case MSP430::RETI: break;  // These are ok
  default:
    llvm_unreachable("Can only insert epilog into returning blocks");
  }

```
- **EN**: Implements logic around `getLastNonDebugInstr`, `getOpcode`, `getDebugLoc`, `llvm_unreachable`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getLastNonDebugInstr`, `getOpcode`, `getDebugLoc`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 209-219
```cpp
  // Get the number of bytes to allocate from the FrameInfo
  uint64_t StackSize = MFI.getStackSize();
  unsigned CSSize = MSP430FI->getCalleeSavedFrameSize();
  uint64_t NumBytes = 0;

  MachineBasicBlock::iterator AfterPop = MBBI;
  if (hasFP(MF)) {
    // Calculate required stack adjustment
    uint64_t FrameSize = StackSize - 2;
    NumBytes = FrameSize - CSSize;

```
- **EN**: Implements logic around `getStackSize`, `getCalleeSavedFrameSize`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getStackSize`, `getCalleeSavedFrameSize` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 220-237
```cpp
    // pop FP.
    BuildMI(MBB, MBBI, DL, TII.get(MSP430::POP16r), MSP430::R4)
        .setMIFlag(MachineInstr::FrameDestroy);
    unsigned DwarfStackPtr = TRI->getDwarfRegNum(MSP430::SP, true);
    BuildCFI(MBB, MBBI, DL,
             MCCFIInstruction::cfiDefCfa(nullptr, DwarfStackPtr, 2),
             MachineInstr::FrameDestroy);
    --MBBI;
    if (!MBB.succ_empty() && !MBB.isReturnBlock()) {
      unsigned DwarfFramePtr = TRI->getDwarfRegNum(MSP430::R4, true);
      BuildCFI(MBB, AfterPop, DL,
               MCCFIInstruction::createRestore(nullptr, DwarfFramePtr),
               MachineInstr::FrameDestroy);
      --MBBI;
      --AfterPop;
    }
  } else
    NumBytes = StackSize - CSSize;
```
- **EN**: Implements logic around `BuildMI`, `setMIFlag`, `getDwarfRegNum`, `BuildCFI`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `setMIFlag`, `getDwarfRegNum`, `BuildCFI`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 238-251
```cpp

  // Skip the callee-saved pop instructions.
  MachineBasicBlock::iterator FirstCSPop = MBBI;
  while (MBBI != MBB.begin()) {
    MachineBasicBlock::iterator PI = std::prev(MBBI);
    unsigned Opc = PI->getOpcode();
    if ((Opc != MSP430::POP16r || !PI->getFlag(MachineInstr::FrameDestroy)) &&
        !PI->isTerminator())
      break;
    FirstCSPop = PI;
    --MBBI;
  }
  MBBI = FirstCSPop;

```
- **EN**: Implements logic around `prev`, `getOpcode`, `isTerminator`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `prev`, `getOpcode`, `isTerminator` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 252-269
```cpp
  DL = MBBI->getDebugLoc();

  // If there is an ADD16ri or SUB16ri of SP immediately before this
  // instruction, merge the two instructions.
  //if (NumBytes || MFI.hasVarSizedObjects())
  //  mergeSPUpdatesUp(MBB, MBBI, StackPtr, &NumBytes);

  if (MFI.hasVarSizedObjects()) {
    BuildMI(MBB, MBBI, DL, TII.get(MSP430::MOV16rr), MSP430::SP)
        .addReg(MSP430::R4)
        .setMIFlag(MachineInstr::FrameDestroy);
    if (CSSize) {
      MachineInstr *MI =
          BuildMI(MBB, MBBI, DL, TII.get(MSP430::SUB16ri), MSP430::SP)
              .addReg(MSP430::SP)
              .addImm(CSSize)
              .setMIFlag(MachineInstr::FrameDestroy);
      // The SRW implicit def is dead.
```
- **EN**: Implements logic around `getDebugLoc`, `BuildMI`, `addReg`, `setMIFlag`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getDebugLoc`, `BuildMI`, `addReg`, `setMIFlag`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 270-282
```cpp
      MI->getOperand(3).setIsDead();
    }
  } else {
    // adjust stack pointer back: SP += numbytes
    if (NumBytes) {
      MachineInstr *MI =
          BuildMI(MBB, MBBI, DL, TII.get(MSP430::ADD16ri), MSP430::SP)
              .addReg(MSP430::SP)
              .addImm(NumBytes)
              .setMIFlag(MachineInstr::FrameDestroy);
      // The SRW implicit def is dead.
      MI->getOperand(3).setIsDead();

```
- **EN**: Implements logic around `getOperand`, `BuildMI`, `addReg`, `addImm`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `BuildMI`, `addReg`, `addImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 283-291
```cpp
      if (!hasFP(MF)) {
        // Adjust CFA value if it was defined by SP
        BuildCFI(MBB, MBBI, DL,
                 MCCFIInstruction::cfiDefCfaOffset(nullptr, CSSize + 2),
                 MachineInstr::FrameDestroy);
      }
    }
  }

```
- **EN**: Implements logic around `BuildCFI`, `cfiDefCfaOffset`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildCFI`, `cfiDefCfaOffset` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 292-309
```cpp
  if (!hasFP(MF)) {
    MBBI = FirstCSPop;
    int64_t Offset = -(int64_t)CSSize - 2;
    // Mark callee-saved pop instruction.
    // Define the current CFA rule to use the provided offset.
    while (MBBI != MBB.end()) {
      MachineBasicBlock::iterator PI = MBBI;
      unsigned Opc = PI->getOpcode();
      ++MBBI;
      if (Opc == MSP430::POP16r) {
        Offset += 2;
        BuildCFI(MBB, MBBI, DL,
                 MCCFIInstruction::cfiDefCfaOffset(nullptr, -Offset),
                 MachineInstr::FrameDestroy);
      }
    }
  }
  emitCalleeSavedFrameMoves(MBB, AfterPop, DL, false);
```
- **EN**: Implements logic around `getOpcode`, `BuildCFI`, `cfiDefCfaOffset`, `emitCalleeSavedFrameMoves`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOpcode`, `BuildCFI`, `cfiDefCfaOffset`, `emitCalleeSavedFrameMoves` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 310-318
```cpp
}

// FIXME: Can we eleminate these in favour of generic code?
bool MSP430FrameLowering::spillCalleeSavedRegisters(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
    ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
  if (CSI.empty())
    return false;

```
- **EN**: Implements logic around `spillCalleeSavedRegisters`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `spillCalleeSavedRegisters` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 319-336
```cpp
  DebugLoc DL;
  if (MI != MBB.end()) DL = MI->getDebugLoc();

  MachineFunction &MF = *MBB.getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  MSP430MachineFunctionInfo *MFI = MF.getInfo<MSP430MachineFunctionInfo>();
  MFI->setCalleeSavedFrameSize(CSI.size() * 2);

  for (const CalleeSavedInfo &I : CSI) {
    MCRegister Reg = I.getReg();
    // Add the callee-saved register as live-in. It's killed at the spill.
    MBB.addLiveIn(Reg);
    BuildMI(MBB, MI, DL, TII.get(MSP430::PUSH16r))
        .addReg(Reg, RegState::Kill)
        .setMIFlag(MachineInstr::FrameSetup);
  }
  return true;
}
```
- **EN**: Implements logic around `getParent`, `getSubtarget`, `getInfo<MSP430MachineFunctionInfo>`, `setCalleeSavedFrameSize`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `getSubtarget`, `getInfo<MSP430MachineFunctionInfo>`, `setCalleeSavedFrameSize`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 337-346
```cpp

bool MSP430FrameLowering::restoreCalleeSavedRegisters(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
    MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
  if (CSI.empty())
    return false;

  DebugLoc DL;
  if (MI != MBB.end()) DL = MI->getDebugLoc();

```
- **EN**: Implements logic around `restoreCalleeSavedRegisters`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `restoreCalleeSavedRegisters` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 347-356
```cpp
  MachineFunction &MF = *MBB.getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();

  for (const CalleeSavedInfo &I : llvm::reverse(CSI))
    BuildMI(MBB, MI, DL, TII.get(MSP430::POP16r), I.getReg())
        .setMIFlag(MachineInstr::FrameDestroy);

  return true;
}

```
- **EN**: Implements logic around `getParent`, `getSubtarget`, `BuildMI`, `setMIFlag`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `getSubtarget`, `BuildMI`, `setMIFlag` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 357-374
```cpp
MachineBasicBlock::iterator MSP430FrameLowering::eliminateCallFramePseudoInstr(
    MachineFunction &MF, MachineBasicBlock &MBB,
    MachineBasicBlock::iterator I) const {
  const MSP430InstrInfo &TII =
      *static_cast<const MSP430InstrInfo *>(MF.getSubtarget().getInstrInfo());
  if (!hasReservedCallFrame(MF)) {
    // If the stack pointer can be changed after prologue, turn the
    // adjcallstackup instruction into a 'sub SP, <amt>' and the
    // adjcallstackdown instruction into 'add SP, <amt>'
    // TODO: consider using push / pop instead of sub + store / add
    MachineInstr &Old = *I;
    uint64_t Amount = TII.getFrameSize(Old);
    if (Amount != 0) {
      // We need to keep the stack aligned properly.  To do this, we round the
      // amount of space needed for the outgoing arguments up to the next
      // alignment boundary.
      Amount = alignTo(Amount, getStackAlign());

```
- **EN**: Implements logic around `eliminateCallFramePseudoInstr`, `getSubtarget`, `getFrameSize`, `alignTo`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateCallFramePseudoInstr`, `getSubtarget`, `getFrameSize`, `alignTo` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 375-391
```cpp
      MachineInstr *New = nullptr;
      if (Old.getOpcode() == TII.getCallFrameSetupOpcode()) {
        New =
            BuildMI(MF, Old.getDebugLoc(), TII.get(MSP430::SUB16ri), MSP430::SP)
                .addReg(MSP430::SP)
                .addImm(Amount);
      } else {
        assert(Old.getOpcode() == TII.getCallFrameDestroyOpcode());
        // factor out the amount the callee already popped.
        Amount -= TII.getFramePoppedByCallee(Old);
        if (Amount)
          New = BuildMI(MF, Old.getDebugLoc(), TII.get(MSP430::ADD16ri),
                        MSP430::SP)
                    .addReg(MSP430::SP)
                    .addImm(Amount);
      }

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`, `assert`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 392-409
```cpp
      if (New) {
        // The SRW implicit def is dead.
        New->getOperand(3).setIsDead();

        // Replace the pseudo instruction with a new instruction...
        MBB.insert(I, New);
      }
    }
  } else if (I->getOpcode() == TII.getCallFrameDestroyOpcode()) {
    // If we are performing frame pointer elimination and if the callee pops
    // something off the stack pointer, add it back.
    if (uint64_t CalleeAmt = TII.getFramePoppedByCallee(*I)) {
      MachineInstr &Old = *I;
      MachineInstr *New =
          BuildMI(MF, Old.getDebugLoc(), TII.get(MSP430::SUB16ri), MSP430::SP)
              .addReg(MSP430::SP)
              .addImm(CalleeAmt);
      if (!hasFP(MF)) {
```
- **EN**: Implements logic around `getOperand`, `insert`, `BuildMI`, `addReg`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `insert`, `BuildMI`, `addReg`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 410-419
```cpp
        CFIInstBuilder(MBB, I, MachineInstr::NoFlags)
            .buildAdjustCFAOffset(CalleeAmt);
      }
      // The SRW implicit def is dead.
      New->getOperand(3).setIsDead();

      MBB.insert(I, New);
    }
  }

```
- **EN**: Implements logic around `CFIInstBuilder`, `buildAdjustCFAOffset`, `getOperand`, `insert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `CFIInstBuilder`, `buildAdjustCFAOffset`, `getOperand`, `insert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 420-433
```cpp
  return MBB.erase(I);
}

void
MSP430FrameLowering::processFunctionBeforeFrameFinalized(MachineFunction &MF,
                                                         RegScavenger *) const {
  // Create a frame entry for the FP register that must be saved.
  if (hasFP(MF)) {
    int FrameIdx = MF.getFrameInfo().CreateFixedObject(2, -4, true);
    (void)FrameIdx;
    assert(FrameIdx == MF.getFrameInfo().getObjectIndexBegin() &&
           "Slot for FP register must be last in order to be found!");
  }
}
```
- **EN**: Implements logic around `erase`, `processFunctionBeforeFrameFinalized`, `getFrameInfo`, `assert`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `erase`, `processFunctionBeforeFrameFinalized`, `getFrameInfo`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Stack frame management / 栈帧管理**:
  - **EN**: Controls prologue/epilogue emission and frame layout
  - **CN**: 控制序言尾声生成与栈帧布局
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MSP430FrameLowering.h`, `MSP430InstrInfo.h`, `MSP430MachineFunctionInfo.h`, `MSP430Subtarget.h`, `llvm/CodeGen/CFIInstBuilder.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/Target/TargetOptions.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
