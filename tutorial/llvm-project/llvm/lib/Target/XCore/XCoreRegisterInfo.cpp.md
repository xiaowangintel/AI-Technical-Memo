# XCoreRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreRegisterInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific register information, reserved registers, spills, and frame-index handling.
  - **CN**: 实现目标相关的寄存器信息、保留寄存器、溢出处理以及 frame index 处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- XCoreRegisterInfo.cpp - XCore Register Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the XCore implementation of the MRegisterInfo class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-29
```cpp

#include "XCoreRegisterInfo.h"
#include "XCoreInstrInfo.h"
#include "XCoreSubtarget.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Type.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Target/TargetOptions.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreRegisterInfo.h`, `XCoreInstrInfo.h`, `XCoreSubtarget.h`, `llvm/ADT/BitVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreRegisterInfo.h`, `XCoreInstrInfo.h`, `XCoreSubtarget.h`, `llvm/ADT/BitVector.h`。

### Lines 30-41
```cpp

using namespace llvm;

#define DEBUG_TYPE "xcore-reg-info"

#define GET_REGINFO_TARGET_DESC
#include "XCoreGenRegisterInfo.inc"

XCoreRegisterInfo::XCoreRegisterInfo()
  : XCoreGenRegisterInfo(XCore::LR) {
}

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreGenRegisterInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreGenRegisterInfo.inc`。

### Lines 42-50
```cpp
// helper functions
static inline bool isImmUs(unsigned val) {
  return val <= 11;
}

static inline bool isImmU6(unsigned val) {
  return val < (1 << 6);
}

```
- **EN**: Implements logic around `isImmUs`, `isImmU6`; this block returns target-specific results.
- **CN**: 围绕 `isImmUs`, `isImmU6` 实现具体逻辑；这一段返回目标相关结果。

### Lines 51-62
```cpp
static inline bool isImmU16(unsigned val) {
  return val < (1 << 16);
}


static void InsertFPImmInst(MachineBasicBlock::iterator II,
                            const XCoreInstrInfo &TII,
                            unsigned Reg, unsigned FrameReg, int Offset ) {
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  DebugLoc dl = MI.getDebugLoc();

```
- **EN**: Implements logic around `isImmU16`, `InsertFPImmInst`, `getParent`, `getDebugLoc`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isImmU16`, `InsertFPImmInst`, `getParent`, `getDebugLoc` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 63-80
```cpp
  switch (MI.getOpcode()) {
  case XCore::LDWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::LDW_2rus), Reg)
          .addReg(FrameReg)
          .addImm(Offset)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::STWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::STW_2rus))
          .addReg(Reg, getKillRegState(MI.getOperand(0).isKill()))
          .addReg(FrameReg)
          .addImm(Offset)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::LDAWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::LDAWF_l2rus), Reg)
          .addReg(FrameReg)
          .addImm(Offset);
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`, `addMemOperand`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm`, `addMemOperand` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 81-98
```cpp
    break;
  default:
    llvm_unreachable("Unexpected Opcode");
  }
}

static void InsertFPConstInst(MachineBasicBlock::iterator II,
                              const XCoreInstrInfo &TII,
                              unsigned Reg, unsigned FrameReg,
                              int Offset, RegScavenger *RS ) {
  assert(RS && "requiresRegisterScavenging failed");
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  DebugLoc dl = MI.getDebugLoc();
  Register ScratchOffset =
      RS->scavengeRegisterBackwards(XCore::GRRegsRegClass, II, false, 0);
  RS->setRegUsed(ScratchOffset);
  TII.loadImmediate(MBB, II, ScratchOffset, Offset);
```
- **EN**: Implements logic around `llvm_unreachable`, `InsertFPConstInst`, `assert`, `getParent`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `llvm_unreachable`, `InsertFPConstInst`, `assert`, `getParent`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 99-116
```cpp

  switch (MI.getOpcode()) {
  case XCore::LDWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::LDW_3r), Reg)
          .addReg(FrameReg)
          .addReg(ScratchOffset, RegState::Kill)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::STWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::STW_l3r))
          .addReg(Reg, getKillRegState(MI.getOperand(0).isKill()))
          .addReg(FrameReg)
          .addReg(ScratchOffset, RegState::Kill)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::LDAWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::LDAWF_l3r), Reg)
          .addReg(FrameReg)
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addMemOperand`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addMemOperand` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 117-131
```cpp
          .addReg(ScratchOffset, RegState::Kill);
    break;
  default:
    llvm_unreachable("Unexpected Opcode");
  }
}

static void InsertSPImmInst(MachineBasicBlock::iterator II,
                            const XCoreInstrInfo &TII,
                            unsigned Reg, int Offset) {
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  DebugLoc dl = MI.getDebugLoc();
  bool isU6 = isImmU6(Offset);

```
- **EN**: Implements logic around `addReg`, `llvm_unreachable`, `InsertSPImmInst`, `getParent`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addReg`, `llvm_unreachable`, `InsertSPImmInst`, `getParent`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 132-149
```cpp
  switch (MI.getOpcode()) {
  int NewOpcode;
  case XCore::LDWFI:
    NewOpcode = (isU6) ? XCore::LDWSP_ru6 : XCore::LDWSP_lru6;
    BuildMI(MBB, II, dl, TII.get(NewOpcode), Reg)
          .addImm(Offset)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::STWFI:
    NewOpcode = (isU6) ? XCore::STWSP_ru6 : XCore::STWSP_lru6;
    BuildMI(MBB, II, dl, TII.get(NewOpcode))
          .addReg(Reg, getKillRegState(MI.getOperand(0).isKill()))
          .addImm(Offset)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::LDAWFI:
    NewOpcode = (isU6) ? XCore::LDAWSP_ru6 : XCore::LDAWSP_lru6;
    BuildMI(MBB, II, dl, TII.get(NewOpcode), Reg)
```
- **EN**: Implements logic around `BuildMI`, `addImm`, `addMemOperand`, `addReg`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addImm`, `addMemOperand`, `addReg` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 150-165
```cpp
          .addImm(Offset);
    break;
  default:
    llvm_unreachable("Unexpected Opcode");
  }
}

static void InsertSPConstInst(MachineBasicBlock::iterator II,
                                const XCoreInstrInfo &TII,
                                unsigned Reg, int Offset, RegScavenger *RS ) {
  assert(RS && "requiresRegisterScavenging failed");
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  DebugLoc dl = MI.getDebugLoc();
  unsigned OpCode = MI.getOpcode();

```
- **EN**: Implements logic around `addImm`, `llvm_unreachable`, `InsertSPConstInst`, `assert`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addImm`, `llvm_unreachable`, `InsertSPConstInst`, `assert`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 166-178
```cpp
  unsigned ScratchBase;
  if (OpCode==XCore::STWFI) {
    ScratchBase =
        RS->scavengeRegisterBackwards(XCore::GRRegsRegClass, II, false, 0);
    RS->setRegUsed(ScratchBase);
  } else
    ScratchBase = Reg;
  BuildMI(MBB, II, dl, TII.get(XCore::LDAWSP_ru6), ScratchBase).addImm(0);
  Register ScratchOffset =
      RS->scavengeRegisterBackwards(XCore::GRRegsRegClass, II, false, 0);
  RS->setRegUsed(ScratchOffset);
  TII.loadImmediate(MBB, II, ScratchOffset, Offset);

```
- **EN**: Implements logic around `scavengeRegisterBackwards`, `setRegUsed`, `BuildMI`, `loadImmediate`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `scavengeRegisterBackwards`, `setRegUsed`, `BuildMI`, `loadImmediate` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 179-196
```cpp
  switch (OpCode) {
  case XCore::LDWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::LDW_3r), Reg)
          .addReg(ScratchBase, RegState::Kill)
          .addReg(ScratchOffset, RegState::Kill)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::STWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::STW_l3r))
          .addReg(Reg, getKillRegState(MI.getOperand(0).isKill()))
          .addReg(ScratchBase, RegState::Kill)
          .addReg(ScratchOffset, RegState::Kill)
          .addMemOperand(*MI.memoperands_begin());
    break;
  case XCore::LDAWFI:
    BuildMI(MBB, II, dl, TII.get(XCore::LDAWF_l3r), Reg)
          .addReg(ScratchBase, RegState::Kill)
          .addReg(ScratchOffset, RegState::Kill);
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addMemOperand`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addMemOperand` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 197-206
```cpp
    break;
  default:
    llvm_unreachable("Unexpected Opcode");
  }
}

bool XCoreRegisterInfo::needsFrameMoves(const MachineFunction &MF) {
  return MF.needsFrameMoves();
}

```
- **EN**: Implements logic around `llvm_unreachable`, `needsFrameMoves`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `llvm_unreachable`, `needsFrameMoves` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 207-224
```cpp
const MCPhysReg *
XCoreRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  // The callee saved registers LR & FP are explicitly handled during
  // emitPrologue & emitEpilogue and related functions.
  static const MCPhysReg CalleeSavedRegs[] = {
    XCore::R4, XCore::R5, XCore::R6, XCore::R7,
    XCore::R8, XCore::R9, XCore::R10,
    0
  };
  static const MCPhysReg CalleeSavedRegsFP[] = {
    XCore::R4, XCore::R5, XCore::R6, XCore::R7,
    XCore::R8, XCore::R9,
    0
  };
  const XCoreFrameLowering *TFI = getFrameLowering(*MF);
  if (TFI->hasFP(*MF))
    return CalleeSavedRegsFP;
  return CalleeSavedRegs;
```
- **EN**: Implements logic around `getCalleeSavedRegs`, `getFrameLowering`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getCalleeSavedRegs`, `getFrameLowering` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 225-240
```cpp
}

BitVector XCoreRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  BitVector Reserved(getNumRegs());
  const XCoreFrameLowering *TFI = getFrameLowering(MF);

  Reserved.set(XCore::CP);
  Reserved.set(XCore::DP);
  Reserved.set(XCore::SP);
  Reserved.set(XCore::LR);
  if (TFI->hasFP(MF)) {
    Reserved.set(XCore::R10);
  }
  return Reserved;
}

```
- **EN**: Implements logic around `getReservedRegs`, `Reserved`, `getFrameLowering`, `set`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getReservedRegs`, `Reserved`, `getFrameLowering`, `set` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 241-250
```cpp
bool
XCoreRegisterInfo::requiresRegisterScavenging(const MachineFunction &MF) const {
  return true;
}

bool
XCoreRegisterInfo::useFPForScavengingIndex(const MachineFunction &MF) const {
  return false;
}

```
- **EN**: Implements logic around `requiresRegisterScavenging`, `useFPForScavengingIndex`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `requiresRegisterScavenging`, `useFPForScavengingIndex` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 251-259
```cpp
bool
XCoreRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                       int SPAdj, unsigned FIOperandNum,
                                       RegScavenger *RS) const {
  assert(SPAdj == 0 && "Unexpected");
  MachineInstr &MI = *II;
  MachineOperand &FrameOp = MI.getOperand(FIOperandNum);
  int FrameIndex = FrameOp.getIndex();

```
- **EN**: Implements logic around `eliminateFrameIndex`, `assert`, `getOperand`, `getIndex`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateFrameIndex`, `assert`, `getOperand`, `getIndex` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 260-276
```cpp
  MachineFunction &MF = *MI.getParent()->getParent();
  const XCoreInstrInfo &TII =
      *static_cast<const XCoreInstrInfo *>(MF.getSubtarget().getInstrInfo());

  const XCoreFrameLowering *TFI = getFrameLowering(MF);
  int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex);
  int StackSize = MF.getFrameInfo().getStackSize();

  #ifndef NDEBUG
  LLVM_DEBUG(errs() << "\nFunction         : " << MF.getName() << "\n");
  LLVM_DEBUG(errs() << "<--------->\n");
  LLVM_DEBUG(MI.print(errs()));
  LLVM_DEBUG(errs() << "FrameIndex         : " << FrameIndex << "\n");
  LLVM_DEBUG(errs() << "FrameOffset        : " << Offset << "\n");
  LLVM_DEBUG(errs() << "StackSize          : " << StackSize << "\n");
#endif

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 277-287
```cpp
  Offset += StackSize;

  Register FrameReg = getFrameRegister(MF);

  // Special handling of DBG_VALUE instructions.
  if (MI.isDebugValue()) {
    MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, false /*isDef*/);
    MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
    return false;
  }

```
- **EN**: Implements logic around `getFrameRegister`, `getOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getFrameRegister`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 288-296
```cpp
  // fold constant into offset.
  Offset += MI.getOperand(FIOperandNum + 1).getImm();
  MI.getOperand(FIOperandNum + 1).ChangeToImmediate(0);

  assert(Offset%4 == 0 && "Misaligned stack offset");
  LLVM_DEBUG(errs() << "Offset             : " << Offset << "\n"
                    << "<--------->\n");
  Offset/=4;

```
- **EN**: Implements logic around `getOperand`, `assert`.
- **CN**: 围绕 `getOperand`, `assert` 实现具体逻辑。

### Lines 297-314
```cpp
  Register Reg = MI.getOperand(0).getReg();
  assert(XCore::GRRegsRegClass.contains(Reg) && "Unexpected register operand");

  if (TFI->hasFP(MF)) {
    if (isImmUs(Offset))
      InsertFPImmInst(II, TII, Reg, FrameReg, Offset);
    else
      InsertFPConstInst(II, TII, Reg, FrameReg, Offset, RS);
  } else {
    if (isImmU16(Offset))
      InsertSPImmInst(II, TII, Reg, Offset);
    else
      InsertSPConstInst(II, TII, Reg, Offset, RS);
  }
  // Erase old instruction.
  MachineBasicBlock &MBB = *MI.getParent();
  MBB.erase(II);
  return true;
```
- **EN**: Implements logic around `getOperand`, `assert`, `InsertFPImmInst`, `InsertFPConstInst`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `assert`, `InsertFPImmInst`, `InsertFPConstInst`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 315-322
```cpp
}


Register XCoreRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  const XCoreFrameLowering *TFI = getFrameLowering(MF);

  return TFI->hasFP(MF) ? XCore::R10 : XCore::SP;
}
```
- **EN**: Implements logic around `getFrameRegister`, `getFrameLowering`, `hasFP`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameRegister`, `getFrameLowering`, `hasFP` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Register model / 寄存器模型**:
  - **EN**: Describes registers, classes, and allocation-facing details
  - **CN**: 描述寄存器、寄存器类以及面向分配器的细节
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCoreRegisterInfo.h`, `XCoreInstrInfo.h`, `XCoreSubtarget.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/CodeGen/RegisterScavenging.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/IR/Function.h`, `llvm/IR/Type.h` ... (+6 more)
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
- **Generated macros / 生成宏**: `GET_REGINFO_TARGET_DESC`
