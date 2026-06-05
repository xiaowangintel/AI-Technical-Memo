# MSP430InstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430InstrInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MSP430InstrInfo.cpp - MSP430 Instruction Information --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the MSP430 implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-21
```cpp

#include "MSP430InstrInfo.h"
#include "MSP430.h"
#include "MSP430Subtarget.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430InstrInfo.h`, `MSP430.h`, `MSP430Subtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430InstrInfo.h`, `MSP430.h`, `MSP430Subtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`。

### Lines 22-32
```cpp
#define GET_INSTRINFO_CTOR_DTOR
#include "MSP430GenInstrInfo.inc"

// Pin the vtable to this file.
void MSP430InstrInfo::anchor() {}

MSP430InstrInfo::MSP430InstrInfo(const MSP430Subtarget &STI)
    : MSP430GenInstrInfo(STI, RI, MSP430::ADJCALLSTACKDOWN,
                         MSP430::ADJCALLSTACKUP),
      RI() {}

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenInstrInfo.inc`。

### Lines 33-41
```cpp
void MSP430InstrInfo::storeRegToStackSlot(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register SrcReg,
    bool isKill, int FrameIdx, const TargetRegisterClass *RC, Register VReg,
    MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (MI != MBB.end()) DL = MI->getDebugLoc();
  MachineFunction &MF = *MBB.getParent();
  MachineFrameInfo &MFI = MF.getFrameInfo();

```
- **EN**: Implements logic around `storeRegToStackSlot`, `getParent`, `getFrameInfo`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `storeRegToStackSlot`, `getParent`, `getFrameInfo` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 42-58
```cpp
  MachineMemOperand *MMO = MF.getMachineMemOperand(
      MachinePointerInfo::getFixedStack(MF, FrameIdx),
      MachineMemOperand::MOStore, MFI.getObjectSize(FrameIdx),
      MFI.getObjectAlign(FrameIdx));

  if (RC == &MSP430::GR16RegClass)
    BuildMI(MBB, MI, DL, get(MSP430::MOV16mr))
      .addFrameIndex(FrameIdx).addImm(0)
      .addReg(SrcReg, getKillRegState(isKill)).addMemOperand(MMO);
  else if (RC == &MSP430::GR8RegClass)
    BuildMI(MBB, MI, DL, get(MSP430::MOV8mr))
      .addFrameIndex(FrameIdx).addImm(0)
      .addReg(SrcReg, getKillRegState(isKill)).addMemOperand(MMO);
  else
    llvm_unreachable("Cannot store this register to stack slot!");
}

```
- **EN**: Implements logic around `getMachineMemOperand`, `getFixedStack`, `getObjectSize`, `getObjectAlign`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getMachineMemOperand`, `getFixedStack`, `getObjectSize`, `getObjectAlign`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 59-69
```cpp
void MSP430InstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                           MachineBasicBlock::iterator MI,
                                           Register DestReg, int FrameIdx,
                                           const TargetRegisterClass *RC,
                                           Register VReg, unsigned SubReg,
                                           MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (MI != MBB.end()) DL = MI->getDebugLoc();
  MachineFunction &MF = *MBB.getParent();
  MachineFrameInfo &MFI = MF.getFrameInfo();

```
- **EN**: Implements logic around `loadRegFromStackSlot`, `getParent`, `getFrameInfo`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot`, `getParent`, `getFrameInfo` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 70-86
```cpp
  MachineMemOperand *MMO = MF.getMachineMemOperand(
      MachinePointerInfo::getFixedStack(MF, FrameIdx),
      MachineMemOperand::MOLoad, MFI.getObjectSize(FrameIdx),
      MFI.getObjectAlign(FrameIdx));

  if (RC == &MSP430::GR16RegClass)
    BuildMI(MBB, MI, DL, get(MSP430::MOV16rm))
      .addReg(DestReg, getDefRegState(true)).addFrameIndex(FrameIdx)
      .addImm(0).addMemOperand(MMO);
  else if (RC == &MSP430::GR8RegClass)
    BuildMI(MBB, MI, DL, get(MSP430::MOV8rm))
      .addReg(DestReg, getDefRegState(true)).addFrameIndex(FrameIdx)
      .addImm(0).addMemOperand(MMO);
  else
    llvm_unreachable("Cannot store this register to stack slot!");
}

```
- **EN**: Implements logic around `getMachineMemOperand`, `getFixedStack`, `getObjectSize`, `getObjectAlign`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getMachineMemOperand`, `getFixedStack`, `getObjectSize`, `getObjectAlign`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 87-99
```cpp
void MSP430InstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator I,
                                  const DebugLoc &DL, Register DestReg,
                                  Register SrcReg, bool KillSrc,
                                  bool RenamableDest, bool RenamableSrc) const {
  unsigned Opc;
  if (MSP430::GR16RegClass.contains(DestReg, SrcReg))
    Opc = MSP430::MOV16rr;
  else if (MSP430::GR8RegClass.contains(DestReg, SrcReg))
    Opc = MSP430::MOV8rr;
  else
    llvm_unreachable("Impossible reg-to-reg copy");

```
- **EN**: Implements logic around `copyPhysReg`, `llvm_unreachable`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `copyPhysReg`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 100-110
```cpp
  BuildMI(MBB, I, DL, get(Opc), DestReg)
    .addReg(SrcReg, getKillRegState(KillSrc));
}

unsigned MSP430InstrInfo::removeBranch(MachineBasicBlock &MBB,
                                       int *BytesRemoved) const {
  assert(!BytesRemoved && "code size not handled");

  MachineBasicBlock::iterator I = MBB.end();
  unsigned Count = 0;

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `removeBranch`, `assert`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `removeBranch`, `assert`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 111-126
```cpp
  while (I != MBB.begin()) {
    --I;
    if (I->isDebugInstr())
      continue;
    if (I->getOpcode() != MSP430::JMP &&
        I->getOpcode() != MSP430::JCC &&
        I->getOpcode() != MSP430::Bi &&
        I->getOpcode() != MSP430::Br &&
        I->getOpcode() != MSP430::Bm)
      break;
    // Remove the branch.
    I->eraseFromParent();
    I = MBB.end();
    ++Count;
  }

```
- **EN**: Implements logic around `getOpcode`, `eraseFromParent`, `end`; this block applies conditional target rules.
- **CN**: 围绕 `getOpcode`, `eraseFromParent`, `end` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 127-135
```cpp
  return Count;
}

bool MSP430InstrInfo::
reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const {
  assert(Cond.size() == 1 && "Invalid Xbranch condition!");

  MSP430CC::CondCodes CC = static_cast<MSP430CC::CondCodes>(Cond[0].getImm());

```
- **EN**: Implements logic around `reverseBranchCondition`, `assert`, `CondCodes>`; this block returns target-specific results.
- **CN**: 围绕 `reverseBranchCondition`, `assert`, `CondCodes>` 实现具体逻辑；这一段返回目标相关结果。

### Lines 136-153
```cpp
  switch (CC) {
  default: llvm_unreachable("Invalid branch condition!");
  case MSP430CC::COND_E:
    CC = MSP430CC::COND_NE;
    break;
  case MSP430CC::COND_NE:
    CC = MSP430CC::COND_E;
    break;
  case MSP430CC::COND_L:
    CC = MSP430CC::COND_GE;
    break;
  case MSP430CC::COND_GE:
    CC = MSP430CC::COND_L;
    break;
  case MSP430CC::COND_HS:
    CC = MSP430CC::COND_LO;
    break;
  case MSP430CC::COND_LO:
```
- **EN**: Implements logic around `llvm_unreachable`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 154-171
```cpp
    CC = MSP430CC::COND_HS;
    break;
  }

  Cond[0].setImm(CC);
  return false;
}

bool MSP430InstrInfo::analyzeBranch(MachineBasicBlock &MBB,
                                    MachineBasicBlock *&TBB,
                                    MachineBasicBlock *&FBB,
                                    SmallVectorImpl<MachineOperand> &Cond,
                                    bool AllowModify) const {
  // Start from the bottom of the block and work up, examining the
  // terminator instructions.
  MachineBasicBlock::iterator I = MBB.end();
  while (I != MBB.begin()) {
    --I;
```
- **EN**: Implements logic around `setImm`, `analyzeBranch`, `end`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `setImm`, `analyzeBranch`, `end` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 172-184
```cpp
    if (I->isDebugInstr())
      continue;

    // Working from the bottom, when we see a non-terminator
    // instruction, we're done.
    if (!isUnpredicatedTerminator(*I))
      break;

    // A terminator that isn't a branch can't easily be handled
    // by this analysis.
    if (!I->isBranch())
      return true;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 185-196
```cpp
    // Cannot handle indirect branches.
    if (I->getOpcode() == MSP430::Br ||
        I->getOpcode() == MSP430::Bm)
      return true;

    // Handle unconditional branches.
    if (I->getOpcode() == MSP430::JMP || I->getOpcode() == MSP430::Bi) {
      if (!AllowModify) {
        TBB = I->getOperand(0).getMBB();
        continue;
      }

```
- **EN**: Implements logic around `getOpcode`, `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOpcode`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 197-209
```cpp
      // If the block has any instructions after a JMP, delete them.
      MBB.erase(std::next(I), MBB.end());
      Cond.clear();
      FBB = nullptr;

      // Delete the JMP if it's equivalent to a fall-through.
      if (MBB.isLayoutSuccessor(I->getOperand(0).getMBB())) {
        TBB = nullptr;
        I->eraseFromParent();
        I = MBB.end();
        continue;
      }

```
- **EN**: Implements logic around `erase`, `clear`, `eraseFromParent`, `end`; this block applies conditional target rules.
- **CN**: 围绕 `erase`, `clear`, `eraseFromParent`, `end` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 210-221
```cpp
      // TBB is used to indicate the unconditinal destination.
      TBB = I->getOperand(0).getMBB();
      continue;
    }

    // Handle conditional branches.
    assert(I->getOpcode() == MSP430::JCC && "Invalid conditional branch");
    MSP430CC::CondCodes BranchCode =
      static_cast<MSP430CC::CondCodes>(I->getOperand(1).getImm());
    if (BranchCode == MSP430CC::COND_INVALID)
      return true;  // Can't handle weird stuff.

```
- **EN**: Implements logic around `getOperand`, `assert`, `CondCodes>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOperand`, `assert`, `CondCodes>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 222-234
```cpp
    // Working from the bottom, handle the first conditional branch.
    if (Cond.empty()) {
      FBB = TBB;
      TBB = I->getOperand(0).getMBB();
      Cond.push_back(MachineOperand::CreateImm(BranchCode));
      continue;
    }

    // Handle subsequent conditional branches. Only handle the case where all
    // conditional branches branch to the same destination.
    assert(Cond.size() == 1);
    assert(TBB);

```
- **EN**: Implements logic around `getOperand`, `push_back`, `assert`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand`, `push_back`, `assert` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 235-244
```cpp
    // Only handle the case where all conditional branches branch to
    // the same destination.
    if (TBB != I->getOperand(0).getMBB())
      return true;

    MSP430CC::CondCodes OldBranchCode = (MSP430CC::CondCodes)Cond[0].getImm();
    // If the conditions are the same, we can leave them alone.
    if (OldBranchCode == BranchCode)
      continue;

```
- **EN**: Implements logic around `getImm`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 245-262
```cpp
    return true;
  }

  return false;
}

unsigned MSP430InstrInfo::insertBranch(MachineBasicBlock &MBB,
                                       MachineBasicBlock *TBB,
                                       MachineBasicBlock *FBB,
                                       ArrayRef<MachineOperand> Cond,
                                       const DebugLoc &DL,
                                       int *BytesAdded) const {
  // Shouldn't be a fall through.
  assert(TBB && "insertBranch must not be told to insert a fallthrough");
  assert((Cond.size() == 1 || Cond.size() == 0) &&
         "MSP430 branch conditions have one component!");
  assert(!BytesAdded && "code size not handled");

```
- **EN**: Implements logic around `insertBranch`, `assert`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertBranch`, `assert` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 263-274
```cpp
  if (Cond.empty()) {
    // Unconditional branch?
    assert(!FBB && "Unconditional branch with multiple successors!");
    BuildMI(&MBB, DL, get(MSP430::JMP)).addMBB(TBB);
    return 1;
  }

  // Conditional branch.
  unsigned Count = 0;
  BuildMI(&MBB, DL, get(MSP430::JCC)).addMBB(TBB).addImm(Cond[0].getImm());
  ++Count;

```
- **EN**: Implements logic around `assert`, `BuildMI`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 275-283
```cpp
  if (FBB) {
    // Two-way Conditional branch. Insert the second branch.
    BuildMI(&MBB, DL, get(MSP430::JMP)).addMBB(FBB);
    ++Count;
  }
  return Count;
}

/// GetInstSize - Return the number of bytes of code the specified
```
- **EN**: Implements logic around `BuildMI`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 284-301
```cpp
/// instruction may be.  This returns the maximum number of bytes.
///
unsigned MSP430InstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
  const MCInstrDesc &Desc = MI.getDesc();

  switch (Desc.getOpcode()) {
  case TargetOpcode::CFI_INSTRUCTION:
  case TargetOpcode::EH_LABEL:
  case TargetOpcode::IMPLICIT_DEF:
  case TargetOpcode::KILL:
  case TargetOpcode::DBG_VALUE:
    return 0;
  case TargetOpcode::INLINEASM:
  case TargetOpcode::INLINEASM_BR: {
    const MachineFunction *MF = MI.getParent()->getParent();
    const TargetInstrInfo &TII = *MF->getSubtarget().getInstrInfo();
    return TII.getInlineAsmLength(MI.getOperand(0).getSymbolName(),
                                  MF->getTarget().getMCAsmInfo());
```
- **EN**: Implements logic around `getInstSizeInBytes`, `getDesc`, `getParent`, `getSubtarget`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `getInstSizeInBytes`, `getDesc`, `getParent`, `getSubtarget`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 302-308
```cpp
  }
  case TargetOpcode::BUNDLE:
    return getInstBundleSize(MI);
  }

  return Desc.getSize();
}
```
- **EN**: Implements logic around `getInstBundleSize`, `getSize`; this block returns target-specific results.
- **CN**: 围绕 `getInstBundleSize`, `getSize` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MSP430InstrInfo.h`, `MSP430.h`, `MSP430Subtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/Support/ErrorHandling.h`, `MSP430GenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
- **Generated macros / 生成宏**: `GET_INSTRINFO_CTOR_DTOR`
