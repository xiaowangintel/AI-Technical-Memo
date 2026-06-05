# XtensaInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaInstrInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//===- XtensaInstrInfo.cpp - Xtensa Instruction Information ---------------===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Xtensa implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 14-27
```cpp

#include "XtensaInstrInfo.h"
#include "XtensaConstantPoolValue.h"
#include "XtensaMachineFunctionInfo.h"
#include "XtensaTargetMachine.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/RegisterScavenging.h"

#define GET_INSTRINFO_CTOR_DTOR
#include "XtensaGenInstrInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaInstrInfo.h`, `XtensaConstantPoolValue.h`, `XtensaMachineFunctionInfo.h`, `XtensaTargetMachine.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaInstrInfo.h`, `XtensaConstantPoolValue.h`, `XtensaMachineFunctionInfo.h`, `XtensaTargetMachine.h`。

### Lines 28-43
```cpp
using namespace llvm;

static const MachineInstrBuilder &
addFrameReference(const MachineInstrBuilder &MIB, int FI) {
  MachineInstr *MI = MIB;
  MachineFunction &MF = *MI->getParent()->getParent();
  MachineFrameInfo &MFFrame = MF.getFrameInfo();
  const MCInstrDesc &MCID = MI->getDesc();
  MachineMemOperand::Flags Flags = MachineMemOperand::MONone;
  if (MCID.mayLoad())
    Flags |= MachineMemOperand::MOLoad;
  if (MCID.mayStore())
    Flags |= MachineMemOperand::MOStore;
  int64_t Offset = 0;
  Align Alignment = MFFrame.getObjectAlign(FI);

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 44-66
```cpp
  MachineMemOperand *MMO =
      MF.getMachineMemOperand(MachinePointerInfo::getFixedStack(MF, FI, Offset),
                              Flags, MFFrame.getObjectSize(FI), Alignment);
  return MIB.addFrameIndex(FI).addImm(Offset).addMemOperand(MMO);
}

XtensaInstrInfo::XtensaInstrInfo(const XtensaSubtarget &STI)
    : XtensaGenInstrInfo(STI, RI, Xtensa::ADJCALLSTACKDOWN,
                         Xtensa::ADJCALLSTACKUP),
      RI(STI), STI(STI) {}

Register XtensaInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
                                              int &FrameIndex) const {
  if (MI.getOpcode() == Xtensa::L32I) {
    if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
        MI.getOperand(2).getImm() == 0) {
      FrameIndex = MI.getOperand(1).getIndex();
      return MI.getOperand(0).getReg();
    }
  }
  return Register();
}

```
- **EN**: Implements logic around `getMachineMemOperand`, `getObjectSize`, `addFrameIndex`, `XtensaInstrInfo`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getMachineMemOperand`, `getObjectSize`, `addFrameIndex`, `XtensaInstrInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 67-78
```cpp
Register XtensaInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
                                             int &FrameIndex) const {
  if (MI.getOpcode() == Xtensa::S32I) {
    if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
        MI.getOperand(2).getImm() == 0) {
      FrameIndex = MI.getOperand(1).getIndex();
      return MI.getOperand(0).getReg();
    }
  }
  return Register();
}

```
- **EN**: Implements logic around `isStoreToStackSlot`, `getOperand`, `Register`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isStoreToStackSlot`, `getOperand`, `Register` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 79-90
```cpp
/// Adjust SP by Amount bytes.
void XtensaInstrInfo::adjustStackPtr(MCRegister SP, int64_t Amount,
                                     MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator I) const {
  DebugLoc DL = I != MBB.end() ? I->getDebugLoc() : DebugLoc();

  if (Amount == 0)
    return;

  MachineRegisterInfo &RegInfo = MBB.getParent()->getRegInfo();
  const TargetRegisterClass *RC = &Xtensa::ARRegClass;

```
- **EN**: Implements logic around `adjustStackPtr`, `end`, `getParent`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `adjustStackPtr`, `end`, `getParent` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 91-103
```cpp
  // create virtual reg to store immediate
  MCRegister Reg = RegInfo.createVirtualRegister(RC);

  if (isInt<8>(Amount)) { // addi sp, sp, amount
    BuildMI(MBB, I, DL, get(Xtensa::ADDI), Reg).addReg(SP).addImm(Amount);
  } else { // Expand immediate that doesn't fit in 8-bit.
    MCRegister Reg1;
    loadImmediate(MBB, I, &Reg1, Amount);
    BuildMI(MBB, I, DL, get(Xtensa::ADD), Reg)
        .addReg(SP)
        .addReg(Reg1, RegState::Kill);
  }

```
- **EN**: Implements logic around `createVirtualRegister`, `BuildMI`, `loadImmediate`, `addReg`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createVirtualRegister`, `BuildMI`, `loadImmediate`, `addReg` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 104-119
```cpp
  if (STI.isWindowedABI()) {
    BuildMI(MBB, I, DL, get(Xtensa::MOVSP), SP).addReg(Reg, RegState::Kill);
  } else {
    BuildMI(MBB, I, DL, get(Xtensa::OR), SP)
        .addReg(Reg, RegState::Kill)
        .addReg(Reg, RegState::Kill);
  }
}

void XtensaInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator MBBI,
                                  const DebugLoc &DL, Register DestReg,
                                  Register SrcReg, bool KillSrc,
                                  bool RenamableDest, bool RenamableSrc) const {
  unsigned Opcode;

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `copyPhysReg`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `copyPhysReg` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 120-140
```cpp
  // The MOV instruction is not present in core ISA for AR registers,
  // so use OR instruction.
  if (Xtensa::ARRegClass.contains(DestReg, SrcReg)) {
    BuildMI(MBB, MBBI, DL, get(Xtensa::OR), DestReg)
        .addReg(SrcReg, getKillRegState(KillSrc))
        .addReg(SrcReg, getKillRegState(KillSrc));
    return;
  }

  if (STI.hasSingleFloat() && Xtensa::FPRRegClass.contains(SrcReg) &&
      Xtensa::FPRRegClass.contains(DestReg))
    Opcode = Xtensa::MOV_S;
  else if (STI.hasSingleFloat() && Xtensa::FPRRegClass.contains(SrcReg) &&
           Xtensa::ARRegClass.contains(DestReg))
    Opcode = Xtensa::RFR;
  else if (STI.hasSingleFloat() && Xtensa::ARRegClass.contains(SrcReg) &&
           Xtensa::FPRRegClass.contains(DestReg))
    Opcode = Xtensa::WFR;
  else
    report_fatal_error("Impossible reg-to-reg copy");

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `contains`, `report_fatal_error`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `contains`, `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 141-157
```cpp
  BuildMI(MBB, MBBI, DL, get(Opcode), DestReg)
      .addReg(SrcReg, getKillRegState(KillSrc));
}

void XtensaInstrInfo::storeRegToStackSlot(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
    bool isKill, int FrameIdx, const TargetRegisterClass *RC,

    Register VReg, MachineInstr::MIFlag Flags) const {
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
  unsigned LoadOpcode, StoreOpcode;
  getLoadStoreOpcodes(RC, LoadOpcode, StoreOpcode, FrameIdx);
  MachineInstrBuilder MIB = BuildMI(MBB, MBBI, DL, get(StoreOpcode))
                                .addReg(SrcReg, getKillRegState(isKill));
  addFrameReference(MIB, FrameIdx);
}

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `storeRegToStackSlot`, `end`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `storeRegToStackSlot`, `end`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 158-169
```cpp
void XtensaInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
                                           MachineBasicBlock::iterator MBBI,
                                           Register DestReg, int FrameIdx,
                                           const TargetRegisterClass *RC,
                                           Register VReg, unsigned SubReg,
                                           MachineInstr::MIFlag Flags) const {
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
  unsigned LoadOpcode, StoreOpcode;
  getLoadStoreOpcodes(RC, LoadOpcode, StoreOpcode, FrameIdx);
  addFrameReference(BuildMI(MBB, MBBI, DL, get(LoadOpcode), DestReg), FrameIdx);
}

```
- **EN**: Implements logic around `loadRegFromStackSlot`, `end`, `getLoadStoreOpcodes`, `addFrameReference`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot`, `end`, `getLoadStoreOpcodes`, `addFrameReference` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 170-184
```cpp
void XtensaInstrInfo::getLoadStoreOpcodes(const TargetRegisterClass *RC,
                                          unsigned &LoadOpcode,
                                          unsigned &StoreOpcode,
                                          int64_t offset) const {
  if (RC == &Xtensa::ARRegClass) {
    LoadOpcode = Xtensa::L32I;
    StoreOpcode = Xtensa::S32I;
  } else if (RC == &Xtensa::FPRRegClass) {
    LoadOpcode = Xtensa::LSI;
    StoreOpcode = Xtensa::SSI;
  } else {
    llvm_unreachable("Unsupported regclass to load or store");
  }
}

```
- **EN**: Implements logic around `getLoadStoreOpcodes`, `llvm_unreachable`; this block applies conditional target rules.
- **CN**: 围绕 `getLoadStoreOpcodes`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 185-199
```cpp
void XtensaInstrInfo::loadImmediate(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator MBBI,
                                    MCRegister *Reg, int64_t Value) const {
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
  MachineRegisterInfo &RegInfo = MBB.getParent()->getRegInfo();
  const TargetRegisterClass *RC = &Xtensa::ARRegClass;

  // create virtual reg to store immediate
  *Reg = RegInfo.createVirtualRegister(RC);
  if (Value >= -2048 && Value <= 2047) {
    BuildMI(MBB, MBBI, DL, get(Xtensa::MOVI), *Reg).addImm(Value);
  } else if (Value >= -32768 && Value <= 32767) {
    int Low = Value & 0xFF;
    int High = Value & ~0xFF;

```
- **EN**: Implements logic around `loadImmediate`, `end`, `getParent`, `createVirtualRegister`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadImmediate`, `end`, `getParent`, `createVirtualRegister`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 200-218
```cpp
    BuildMI(MBB, MBBI, DL, get(Xtensa::MOVI), *Reg).addImm(Low);
    BuildMI(MBB, MBBI, DL, get(Xtensa::ADDMI), *Reg).addReg(*Reg).addImm(High);
  } else if (Value >= -4294967296LL && Value <= 4294967295LL) {
    // 32 bit arbitrary constant
    MachineConstantPool *MCP = MBB.getParent()->getConstantPool();
    uint64_t UVal = ((uint64_t)Value) & 0xFFFFFFFFLL;
    const Constant *CVal = ConstantInt::get(
        Type::getInt32Ty(MBB.getParent()->getFunction().getContext()), UVal,
        false);
    unsigned Idx = MCP->getConstantPoolIndex(CVal, Align(2U));
    //	MCSymbol MSym
    BuildMI(MBB, MBBI, DL, get(Xtensa::L32R), *Reg).addConstantPoolIndex(Idx);
  } else {
    // use L32R to let assembler load immediate best
    // TODO replace to L32R
    report_fatal_error("Unsupported load immediate value");
  }
}

```
- **EN**: Implements logic around `BuildMI`, `getParent`, `get`, `getInt32Ty`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `getParent`, `get`, `getInt32Ty`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 219-232
```cpp
unsigned XtensaInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
  switch (MI.getOpcode()) {
  case TargetOpcode::INLINEASM: { // Inline Asm: Variable size.
    const MachineFunction *MF = MI.getParent()->getParent();
    const char *AsmStr = MI.getOperand(0).getSymbolName();
    return getInlineAsmLength(AsmStr, MF->getTarget().getMCAsmInfo());
  }
  case TargetOpcode::BUNDLE:
    return getInstBundleSize(MI);
  default:
    return MI.getDesc().getSize();
  }
}

```
- **EN**: Implements logic around `getInstSizeInBytes`, `getParent`, `getOperand`, `getInlineAsmLength`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getInstSizeInBytes`, `getParent`, `getOperand`, `getInlineAsmLength`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 233-256
```cpp
bool XtensaInstrInfo::reverseBranchCondition(
    SmallVectorImpl<MachineOperand> &Cond) const {
  assert(Cond.size() <= 4 && "Invalid branch condition!");

  switch (Cond[0].getImm()) {
  case Xtensa::BEQ:
    Cond[0].setImm(Xtensa::BNE);
    return false;
  case Xtensa::BNE:
    Cond[0].setImm(Xtensa::BEQ);
    return false;
  case Xtensa::BLT:
    Cond[0].setImm(Xtensa::BGE);
    return false;
  case Xtensa::BGE:
    Cond[0].setImm(Xtensa::BLT);
    return false;
  case Xtensa::BLTU:
    Cond[0].setImm(Xtensa::BGEU);
    return false;
  case Xtensa::BGEU:
    Cond[0].setImm(Xtensa::BLTU);
    return false;
  case Xtensa::BEQI:
```
- **EN**: Implements logic around `reverseBranchCondition`, `assert`, `setImm`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `reverseBranchCondition`, `assert`, `setImm` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 257-280
```cpp
    Cond[0].setImm(Xtensa::BNEI);
    return false;
  case Xtensa::BNEI:
    Cond[0].setImm(Xtensa::BEQI);
    return false;
  case Xtensa::BGEI:
    Cond[0].setImm(Xtensa::BLTI);
    return false;
  case Xtensa::BLTI:
    Cond[0].setImm(Xtensa::BGEI);
    return false;
  case Xtensa::BGEUI:
    Cond[0].setImm(Xtensa::BLTUI);
    return false;
  case Xtensa::BLTUI:
    Cond[0].setImm(Xtensa::BGEUI);
    return false;
  case Xtensa::BEQZ:
    Cond[0].setImm(Xtensa::BNEZ);
    return false;
  case Xtensa::BNEZ:
    Cond[0].setImm(Xtensa::BEQZ);
    return false;
  case Xtensa::BLTZ:
```
- **EN**: Implements logic around `setImm`; this block returns target-specific results.
- **CN**: 围绕 `setImm` 实现具体逻辑；这一段返回目标相关结果。

### Lines 281-296
```cpp
    Cond[0].setImm(Xtensa::BGEZ);
    return false;
  case Xtensa::BGEZ:
    Cond[0].setImm(Xtensa::BLTZ);
    return false;
  case Xtensa::BF:
    Cond[0].setImm(Xtensa::BT);
    return false;
  case Xtensa::BT:
    Cond[0].setImm(Xtensa::BF);
    return false;
  default:
    report_fatal_error("Invalid branch condition!");
  }
}

```
- **EN**: Implements logic around `setImm`, `report_fatal_error`; this block returns target-specific results.
- **CN**: 围绕 `setImm`, `report_fatal_error` 实现具体逻辑；这一段返回目标相关结果。

### Lines 297-320
```cpp
MachineBasicBlock *
XtensaInstrInfo::getBranchDestBlock(const MachineInstr &MI) const {
  unsigned OpCode = MI.getOpcode();
  switch (OpCode) {
  case Xtensa::BR_JT:
  case Xtensa::JX:
    return nullptr;
  case Xtensa::J:
    return MI.getOperand(0).getMBB();
  case Xtensa::BEQ:
  case Xtensa::BNE:
  case Xtensa::BLT:
  case Xtensa::BLTU:
  case Xtensa::BGE:
  case Xtensa::BGEU:
    return MI.getOperand(2).getMBB();
  case Xtensa::BEQI:
  case Xtensa::BNEI:
  case Xtensa::BLTI:
  case Xtensa::BLTUI:
  case Xtensa::BGEI:
  case Xtensa::BGEUI:
    return MI.getOperand(2).getMBB();
  case Xtensa::BEQZ:
```
- **EN**: Implements logic around `getBranchDestBlock`, `getOpcode`, `getOperand`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getBranchDestBlock`, `getOpcode`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 321-332
```cpp
  case Xtensa::BNEZ:
  case Xtensa::BLTZ:
  case Xtensa::BGEZ:
    return MI.getOperand(1).getMBB();
  case Xtensa::BT:
  case Xtensa::BF:
    return MI.getOperand(1).getMBB();
  default:
    llvm_unreachable("Unknown branch opcode");
  }
}

```
- **EN**: Implements logic around `getOperand`, `llvm_unreachable`; this block returns target-specific results.
- **CN**: 围绕 `getOperand`, `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果。

### Lines 333-356
```cpp
bool XtensaInstrInfo::isBranchOffsetInRange(unsigned BranchOp,
                                            int64_t BrOffset) const {
  switch (BranchOp) {
  case Xtensa::J:
    BrOffset -= 4;
    return isIntN(18, BrOffset);
  case Xtensa::JX:
    return true;
  case Xtensa::BR_JT:
    return true;
  case Xtensa::BEQ:
  case Xtensa::BNE:
  case Xtensa::BLT:
  case Xtensa::BLTU:
  case Xtensa::BGE:
  case Xtensa::BGEU:
  case Xtensa::BEQI:
  case Xtensa::BNEI:
  case Xtensa::BLTI:
  case Xtensa::BLTUI:
  case Xtensa::BGEI:
  case Xtensa::BGEUI:
    BrOffset -= 4;
    return isIntN(8, BrOffset);
```
- **EN**: Implements logic around `isBranchOffsetInRange`, `isIntN`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `isBranchOffsetInRange`, `isIntN` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 357-371
```cpp
  case Xtensa::BEQZ:
  case Xtensa::BNEZ:
  case Xtensa::BLTZ:
  case Xtensa::BGEZ:
    BrOffset -= 4;
    return isIntN(12, BrOffset);
  case Xtensa::BT:
  case Xtensa::BF:
    BrOffset -= 4;
    return isIntN(8, BrOffset);
  default:
    llvm_unreachable("Unknown branch opcode");
  }
}

```
- **EN**: Implements logic around `isIntN`, `llvm_unreachable`; this block returns target-specific results.
- **CN**: 围绕 `isIntN`, `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果。

### Lines 372-386
```cpp
bool XtensaInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
                                    MachineBasicBlock *&TBB,
                                    MachineBasicBlock *&FBB,
                                    SmallVectorImpl<MachineOperand> &Cond,
                                    bool AllowModify = false) const {
  // Most of the code and comments here are boilerplate.

  // Start from the bottom of the block and work up, examining the
  // terminator instructions.
  MachineBasicBlock::iterator I = MBB.end();
  while (I != MBB.begin()) {
    --I;
    if (I->isDebugValue())
      continue;

```
- **EN**: Implements logic around `analyzeBranch`, `end`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `analyzeBranch`, `end` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 387-399
```cpp
    // Working from the bottom, when we see a non-terminator instruction, we're
    // done.
    if (!isUnpredicatedTerminator(*I))
      break;

    // A terminator that isn't a branch can't easily be handled by this
    // analysis.
    SmallVector<MachineOperand, 4> ThisCond;
    ThisCond.push_back(MachineOperand::CreateImm(0));
    const MachineOperand *ThisTarget;
    if (!isBranch(I, ThisCond, ThisTarget))
      return true;

```
- **EN**: Implements logic around `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 400-414
```cpp
    // Can't handle indirect branches.
    if (!ThisTarget->isMBB())
      return true;

    if (ThisCond[0].getImm() == Xtensa::J) {
      // Handle unconditional branches.
      if (!AllowModify) {
        TBB = ThisTarget->getMBB();
        continue;
      }

      // If the block has any instructions after a JMP, delete them.
      while (std::next(I) != MBB.end())
        std::next(I)->eraseFromParent();

```
- **EN**: Implements logic around `getMBB`, `next`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getMBB`, `next` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 415-429
```cpp
      Cond.clear();
      FBB = 0;

      // TBB is used to indicate the unconditinal destination.
      TBB = ThisTarget->getMBB();
      continue;
    }

    // Working from the bottom, handle the first conditional branch.
    if (Cond.empty()) {
      // FIXME: add X86-style branch swap
      FBB = TBB;
      TBB = ThisTarget->getMBB();
      Cond.push_back(MachineOperand::CreateImm(ThisCond[0].getImm()));

```
- **EN**: Implements logic around `clear`, `getMBB`, `push_back`; this block applies conditional target rules.
- **CN**: 围绕 `clear`, `getMBB`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 430-445
```cpp
      // push remaining operands
      for (unsigned int i = 0; i < (I->getNumExplicitOperands() - 1); i++)
        Cond.push_back(I->getOperand(i));

      continue;
    }

    // Handle subsequent conditional branches.
    assert(Cond.size() <= 4);
    assert(TBB);

    // Only handle the case where all conditional branches branch to the same
    // destination.
    if (TBB != ThisTarget->getMBB())
      return true;

```
- **EN**: Implements logic around `push_back`, `assert`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 446-462
```cpp
    // If the conditions are the same, we can leave them alone.
    unsigned OldCond = Cond[0].getImm();
    if (OldCond == ThisCond[0].getImm())
      continue;
  }

  return false;
}

unsigned XtensaInstrInfo::removeBranch(MachineBasicBlock &MBB,
                                       int *BytesRemoved) const {
  // Most of the code and comments here are boilerplate.
  MachineBasicBlock::iterator I = MBB.end();
  unsigned Count = 0;
  if (BytesRemoved)
    *BytesRemoved = 0;

```
- **EN**: Implements logic around `getImm`, `removeBranch`, `end`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getImm`, `removeBranch`, `end` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 463-481
```cpp
  while (I != MBB.begin()) {
    --I;
    SmallVector<MachineOperand, 4> Cond;
    Cond.push_back(MachineOperand::CreateImm(0));
    const MachineOperand *Target;
    if (!isBranch(I, Cond, Target))
      break;
    if (!Target->isMBB())
      break;
    // Remove the branch.
    if (BytesRemoved)
      *BytesRemoved += getInstSizeInBytes(*I);
    I->eraseFromParent();
    I = MBB.end();
    ++Count;
  }
  return Count;
}

```
- **EN**: Implements logic around `push_back`, `getInstSizeInBytes`, `eraseFromParent`, `end`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `getInstSizeInBytes`, `eraseFromParent`, `end` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 482-503
```cpp
unsigned XtensaInstrInfo::insertBranch(
    MachineBasicBlock &MBB, MachineBasicBlock *TBB, MachineBasicBlock *FBB,
    ArrayRef<MachineOperand> Cond, const DebugLoc &DL, int *BytesAdded) const {
  unsigned Count = 0;
  if (BytesAdded)
    *BytesAdded = 0;
  if (FBB) {
    // Need to build two branches then
    // one to branch to TBB on Cond
    // and a second one immediately after to unconditionally jump to FBB
    Count = insertBranchAtInst(MBB, MBB.end(), TBB, Cond, DL, BytesAdded);
    auto &MI = *BuildMI(&MBB, DL, get(Xtensa::J)).addMBB(FBB);
    Count++;
    if (BytesAdded)
      *BytesAdded += getInstSizeInBytes(MI);
    return Count;
  }
  // This function inserts the branch at the end of the MBB
  Count += insertBranchAtInst(MBB, MBB.end(), TBB, Cond, DL, BytesAdded);
  return Count;
}

```
- **EN**: Implements logic around `insertBranch`, `insertBranchAtInst`, `BuildMI`, `getInstSizeInBytes`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertBranch`, `insertBranchAtInst`, `BuildMI`, `getInstSizeInBytes` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 504-519
```cpp
void XtensaInstrInfo::insertIndirectBranch(MachineBasicBlock &MBB,
                                           MachineBasicBlock &DestBB,
                                           MachineBasicBlock &RestoreBB,
                                           const DebugLoc &DL, int64_t BrOffset,
                                           RegScavenger *RS) const {
  assert(RS && "RegScavenger required for long branching");
  assert(MBB.empty() &&
         "new block should be inserted for expanding unconditional branch");
  assert(MBB.pred_size() == 1);

  MachineFunction *MF = MBB.getParent();
  MachineRegisterInfo &MRI = MF->getRegInfo();
  MachineConstantPool *ConstantPool = MF->getConstantPool();
  auto *XtensaFI = MF->getInfo<XtensaMachineFunctionInfo>();
  MachineBasicBlock *JumpToMBB = &DestBB;

```
- **EN**: Implements logic around `insertIndirectBranch`, `assert`, `getParent`, `getRegInfo`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertIndirectBranch`, `assert`, `getParent`, `getRegInfo`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 520-531
```cpp
  if (!isInt<32>(BrOffset))
    report_fatal_error(
        "Branch offsets outside of the signed 32-bit range not supported");

  Register ScratchReg = MRI.createVirtualRegister(&Xtensa::ARRegClass);
  auto II = MBB.end();

  // Create l32r without last operand. We will add this operand later when
  // JumpToMMB will be calculated and placed to the ConstantPool.
  MachineInstr &L32R = *BuildMI(MBB, II, DL, get(Xtensa::L32R), ScratchReg);
  BuildMI(MBB, II, DL, get(Xtensa::JX)).addReg(ScratchReg, RegState::Kill);

```
- **EN**: Implements logic around `report_fatal_error`, `createVirtualRegister`, `end`, `BuildMI`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `report_fatal_error`, `createVirtualRegister`, `end`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 532-543
```cpp
  RS->enterBasicBlockEnd(MBB);
  Register ScavRegister =
      RS->scavengeRegisterBackwards(Xtensa::ARRegClass, L32R.getIterator(),
                                    /*RestoreAfter=*/false, /*SpAdj=*/0,
                                    /*AllowSpill=*/false);
  if (ScavRegister != Xtensa::NoRegister)
    RS->setRegUsed(ScavRegister);
  else {
    // The case when there is no scavenged register needs special handling.
    // Pick A8 because it doesn't make a difference
    ScavRegister = Xtensa::A12;

```
- **EN**: Implements logic around `enterBasicBlockEnd`, `scavengeRegisterBackwards`, `setRegUsed`; this block applies conditional target rules.
- **CN**: 围绕 `enterBasicBlockEnd`, `scavengeRegisterBackwards`, `setRegUsed` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 544-558
```cpp
    int FrameIndex = XtensaFI->getBranchRelaxationScratchFrameIndex();
    if (FrameIndex == -1)
      report_fatal_error(
          "Unable to properly handle scavenged register for indirect jump, "
          "function code size is significantly larger than estimated");

    storeRegToStackSlot(MBB, L32R, ScavRegister, /*IsKill=*/true, FrameIndex,
                        &Xtensa::ARRegClass, Register());
    RI.eliminateFrameIndex(std::prev(L32R.getIterator()),
                           /*SpAdj=*/0, /*FIOperandNum=*/1);

    loadRegFromStackSlot(RestoreBB, RestoreBB.end(), ScavRegister, FrameIndex,
                         &Xtensa::ARRegClass, Register());
    RI.eliminateFrameIndex(RestoreBB.back(),
                           /*SpAdj=*/0, /*FIOperandNum=*/1);
```
- **EN**: Implements logic around `getBranchRelaxationScratchFrameIndex`, `report_fatal_error`, `storeRegToStackSlot`, `Register`, ...; this block applies conditional target rules.
- **CN**: 围绕 `getBranchRelaxationScratchFrameIndex`, `report_fatal_error`, `storeRegToStackSlot`, `Register`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 559-572
```cpp
    JumpToMBB = &RestoreBB;
  }

  unsigned LabelId = XtensaFI->createCPLabelId();

  XtensaConstantPoolValue *C = XtensaConstantPoolMBB::Create(
      MF->getFunction().getContext(), JumpToMBB, LabelId);
  unsigned Idx = ConstantPool->getConstantPoolIndex(C, Align(4));
  L32R.addOperand(MachineOperand::CreateCPI(Idx, 0));

  MRI.replaceRegWith(ScratchReg, ScavRegister);
  MRI.clearVirtRegs();
}

```
- **EN**: Implements logic around `createCPLabelId`, `Create`, `getFunction`, `getConstantPoolIndex`, ....
- **CN**: 围绕 `createCPLabelId`, `Create`, `getFunction`, `getConstantPoolIndex`, ... 实现具体逻辑。

### Lines 573-586
```cpp
unsigned XtensaInstrInfo::insertConstBranchAtInst(
    MachineBasicBlock &MBB, MachineInstr *I, int64_t offset,
    ArrayRef<MachineOperand> Cond, DebugLoc DL, int *BytesAdded) const {
  assert(Cond.size() <= 4 &&
         "Xtensa branch conditions have less than four components!");

  if (Cond.empty() || (Cond[0].getImm() == Xtensa::J)) {
    // Unconditional branch
    MachineInstr *MI = BuildMI(MBB, I, DL, get(Xtensa::J)).addImm(offset);
    if (BytesAdded && MI)
      *BytesAdded += getInstSizeInBytes(*MI);
    return 1;
  }

```
- **EN**: Implements logic around `insertConstBranchAtInst`, `assert`, `BuildMI`, `getInstSizeInBytes`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertConstBranchAtInst`, `assert`, `BuildMI`, `getInstSizeInBytes` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 587-610
```cpp
  unsigned Count = 0;
  unsigned BR_C = Cond[0].getImm();
  MachineInstr *MI = nullptr;
  switch (BR_C) {
  case Xtensa::BEQ:
  case Xtensa::BNE:
  case Xtensa::BLT:
  case Xtensa::BLTU:
  case Xtensa::BGE:
  case Xtensa::BGEU:
    MI = BuildMI(MBB, I, DL, get(BR_C))
             .addImm(offset)
             .addReg(Cond[1].getReg())
             .addReg(Cond[2].getReg());
    break;
  case Xtensa::BEQI:
  case Xtensa::BNEI:
  case Xtensa::BLTI:
  case Xtensa::BLTUI:
  case Xtensa::BGEI:
  case Xtensa::BGEUI:
    MI = BuildMI(MBB, I, DL, get(BR_C))
             .addImm(offset)
             .addReg(Cond[1].getReg())
```
- **EN**: Implements logic around `getImm`, `BuildMI`, `addImm`, `addReg`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getImm`, `BuildMI`, `addImm`, `addReg` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 611-631
```cpp
             .addImm(Cond[2].getImm());
    break;
  case Xtensa::BEQZ:
  case Xtensa::BNEZ:
  case Xtensa::BLTZ:
  case Xtensa::BGEZ:
    MI = BuildMI(MBB, I, DL, get(BR_C)).addImm(offset).addReg(Cond[1].getReg());
    break;
  case Xtensa::BT:
  case Xtensa::BF:
    MI = BuildMI(MBB, I, DL, get(BR_C)).addImm(offset).addReg(Cond[1].getReg());
    break;
  default:
    llvm_unreachable("Invalid branch type!");
  }
  if (BytesAdded && MI)
    *BytesAdded += getInstSizeInBytes(*MI);
  ++Count;
  return Count;
}

```
- **EN**: Implements logic around `addImm`, `BuildMI`, `llvm_unreachable`, `getInstSizeInBytes`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addImm`, `BuildMI`, `llvm_unreachable`, `getInstSizeInBytes` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 632-650
```cpp
unsigned XtensaInstrInfo::insertBranchAtInst(MachineBasicBlock &MBB,
                                             MachineBasicBlock::iterator I,
                                             MachineBasicBlock *TBB,
                                             ArrayRef<MachineOperand> Cond,
                                             const DebugLoc &DL,
                                             int *BytesAdded) const {
  // Shouldn't be a fall through.
  assert(TBB && "InsertBranch must not be told to insert a fallthrough");
  assert(Cond.size() <= 4 &&
         "Xtensa branch conditions have less than four components!");

  if (Cond.empty() || (Cond[0].getImm() == Xtensa::J)) {
    // Unconditional branch
    MachineInstr *MI = BuildMI(MBB, I, DL, get(Xtensa::J)).addMBB(TBB);
    if (BytesAdded && MI)
      *BytesAdded += getInstSizeInBytes(*MI);
    return 1;
  }

```
- **EN**: Implements logic around `insertBranchAtInst`, `assert`, `BuildMI`, `getInstSizeInBytes`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertBranchAtInst`, `assert`, `BuildMI`, `getInstSizeInBytes` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 651-674
```cpp
  unsigned Count = 0;
  unsigned BR_C = Cond[0].getImm();
  MachineInstr *MI = nullptr;
  switch (BR_C) {
  case Xtensa::BEQ:
  case Xtensa::BNE:
  case Xtensa::BLT:
  case Xtensa::BLTU:
  case Xtensa::BGE:
  case Xtensa::BGEU:
    MI = BuildMI(MBB, I, DL, get(BR_C))
             .addReg(Cond[1].getReg())
             .addReg(Cond[2].getReg())
             .addMBB(TBB);
    break;
  case Xtensa::BEQI:
  case Xtensa::BNEI:
  case Xtensa::BLTI:
  case Xtensa::BLTUI:
  case Xtensa::BGEI:
  case Xtensa::BGEUI:
    MI = BuildMI(MBB, I, DL, get(BR_C))
             .addReg(Cond[1].getReg())
             .addImm(Cond[2].getImm())
```
- **EN**: Implements logic around `getImm`, `BuildMI`, `addReg`, `addMBB`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getImm`, `BuildMI`, `addReg`, `addMBB`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 675-695
```cpp
             .addMBB(TBB);
    break;
  case Xtensa::BEQZ:
  case Xtensa::BNEZ:
  case Xtensa::BLTZ:
  case Xtensa::BGEZ:
    MI = BuildMI(MBB, I, DL, get(BR_C)).addReg(Cond[1].getReg()).addMBB(TBB);
    break;
  case Xtensa::BT:
  case Xtensa::BF:
    MI = BuildMI(MBB, I, DL, get(BR_C)).addReg(Cond[1].getReg()).addMBB(TBB);
    break;
  default:
    report_fatal_error("Invalid branch type!");
  }
  if (BytesAdded && MI)
    *BytesAdded += getInstSizeInBytes(*MI);
  ++Count;
  return Count;
}

```
- **EN**: Implements logic around `addMBB`, `BuildMI`, `report_fatal_error`, `getInstSizeInBytes`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addMBB`, `BuildMI`, `report_fatal_error`, `getInstSizeInBytes` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 696-716
```cpp
bool XtensaInstrInfo::isBranch(const MachineBasicBlock::iterator &MI,
                               SmallVectorImpl<MachineOperand> &Cond,
                               const MachineOperand *&Target) const {
  unsigned OpCode = MI->getOpcode();
  switch (OpCode) {
  case Xtensa::J:
  case Xtensa::JX:
  case Xtensa::BR_JT:
    Cond[0].setImm(OpCode);
    Target = &MI->getOperand(0);
    return true;
  case Xtensa::BEQ:
  case Xtensa::BNE:
  case Xtensa::BLT:
  case Xtensa::BLTU:
  case Xtensa::BGE:
  case Xtensa::BGEU:
    Cond[0].setImm(OpCode);
    Target = &MI->getOperand(2);
    return true;

```
- **EN**: Implements logic around `isBranch`, `getOpcode`, `setImm`, `getOperand`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isBranch`, `getOpcode`, `setImm`, `getOperand` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 717-734
```cpp
  case Xtensa::BEQI:
  case Xtensa::BNEI:
  case Xtensa::BLTI:
  case Xtensa::BLTUI:
  case Xtensa::BGEI:
  case Xtensa::BGEUI:
    Cond[0].setImm(OpCode);
    Target = &MI->getOperand(2);
    return true;

  case Xtensa::BEQZ:
  case Xtensa::BNEZ:
  case Xtensa::BLTZ:
  case Xtensa::BGEZ:
    Cond[0].setImm(OpCode);
    Target = &MI->getOperand(1);
    return true;

```
- **EN**: Implements logic around `setImm`, `getOperand`; this block returns target-specific results.
- **CN**: 围绕 `setImm`, `getOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 735-745
```cpp
  case Xtensa::BT:
  case Xtensa::BF:
    Cond[0].setImm(OpCode);
    Target = &MI->getOperand(1);
    return true;

  default:
    assert(!MI->getDesc().isBranch() && "Unknown branch opcode");
    return false;
  }
}
```
- **EN**: Implements logic around `setImm`, `getOperand`, `assert`; this block returns target-specific results.
- **CN**: 围绕 `setImm`, `getOperand`, `assert` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaInstrInfo.h`, `XtensaConstantPoolValue.h`, `XtensaMachineFunctionInfo.h`, `XtensaTargetMachine.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/RegisterScavenging.h`, `XtensaGenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_INSTRINFO_CTOR_DTOR`
