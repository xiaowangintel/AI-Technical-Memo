# Mips16InstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips16InstrInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Mips16 implementation of the TargetInstrInfo class.
- 用途 (CN): 实现 Mips 后端中的 `Mips16InstrInfo`，重点处理指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- Mips16InstrInfo.cpp - Mips16 Instruction Information ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Mips16 implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "Mips16InstrInfo.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-33
```cpp
#include <cctype>
#include <cstdint>
#include <cstdlib>
#include <cstring>
#include <iterator>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 35-35
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 37-37
```cpp
#define DEBUG_TYPE "mips16-instrinfo"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 39-40
```cpp
Mips16InstrInfo::Mips16InstrInfo(const MipsSubtarget &STI)
    : MipsInstrInfo(STI, RI, Mips::Bimm16), RI(STI) {}
```
- EN: Implements `Mips16InstrInfo::Mips16InstrInfo`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::Mips16InstrInfo`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-50
```cpp
/// isLoadFromStackSlot - If the specified machine instruction is a direct
/// load from a stack slot, return the virtual or physical register number of
/// the destination along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than loading from the stack slot.
Register Mips16InstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
                                              int &FrameIndex) const {
  return 0;
}
```
- EN: Implements `Mips16InstrInfo::isLoadFromStackSlot`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::isLoadFromStackSlot`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-60
```cpp
/// isStoreToStackSlot - If the specified machine instruction is a direct
/// store to a stack slot, return the virtual or physical register number of
/// the source reg along with the FrameIndex of the loaded stack slot.  If
/// not, return 0.  This predicate must return 0 if the instruction has
/// any side effects other than storing to the stack slot.
Register Mips16InstrInfo::isStoreToStackSlot(const MachineInstr &MI,
                                             int &FrameIndex) const {
  return 0;
}
```
- EN: Implements `Mips16InstrInfo::isStoreToStackSlot`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::isStoreToStackSlot`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-67
```cpp
void Mips16InstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator I,
                                  const DebugLoc &DL, Register DestReg,
                                  Register SrcReg, bool KillSrc,
                                  bool RenamableDest, bool RenamableSrc) const {
  unsigned Opc = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 69-80
```cpp
  if (Mips::CPU16RegsRegClass.contains(DestReg) &&
      Mips::GPR32RegClass.contains(SrcReg))
    Opc = Mips::MoveR3216;
  else if (Mips::GPR32RegClass.contains(DestReg) &&
           Mips::CPU16RegsRegClass.contains(SrcReg))
    Opc = Mips::Move32R16;
  else if ((SrcReg == Mips::HI0) &&
           (Mips::CPU16RegsRegClass.contains(DestReg)))
    Opc = Mips::Mfhi16, SrcReg = 0;
  else if ((SrcReg == Mips::LO0) &&
           (Mips::CPU16RegsRegClass.contains(DestReg)))
    Opc = Mips::Mflo16, SrcReg = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 82-82
```cpp
  assert(Opc && "Cannot copy registers");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-84
```cpp
  MachineInstrBuilder MIB = BuildMI(MBB, I, DL, get(Opc));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 86-87
```cpp
  if (DestReg)
    MIB.addReg(DestReg, RegState::Define);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 89-91
```cpp
  if (SrcReg)
    MIB.addReg(SrcReg, getKillRegState(KillSrc));
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 93-98
```cpp
std::optional<DestSourcePair>
Mips16InstrInfo::isCopyInstrImpl(const MachineInstr &MI) const {
  if (MI.isMoveReg())
    return DestSourcePair{MI.getOperand(0), MI.getOperand(1)};
  return std::nullopt;
}
```
- EN: Implements `Mips16InstrInfo::isCopyInstrImpl`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::isCopyInstrImpl`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-114
```cpp
void Mips16InstrInfo::storeRegToStack(MachineBasicBlock &MBB,
                                      MachineBasicBlock::iterator I,
                                      Register SrcReg, bool isKill, int FI,
                                      const TargetRegisterClass *RC,
                                      int64_t Offset,
                                      MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end()) DL = I->getDebugLoc();
  MachineMemOperand *MMO = GetMemOperand(MBB, FI, MachineMemOperand::MOStore);
  unsigned Opc = 0;
  if (Mips::CPU16RegsRegClass.hasSubClassEq(RC))
    Opc = Mips::SwRxSpImmX16;
  assert(Opc && "Register class not handled!");
  BuildMI(MBB, I, DL, get(Opc)).addReg(SrcReg, getKillRegState(isKill)).
      addFrameIndex(FI).addImm(Offset)
```
- EN: Declares `not`, packaging target-specific state and APIs around `Mips16InstrInfo`.
- CN: 这里声明 `not`，把与 `Mips16InstrInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 115-116
```cpp
      .addMemOperand(MMO);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 118-127
```cpp
void Mips16InstrInfo::loadRegFromStack(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator I,
                                       Register DestReg, int FI,
                                       const TargetRegisterClass *RC,
                                       int64_t Offset,
                                       MachineInstr::MIFlag Flags) const {
  DebugLoc DL;
  if (I != MBB.end()) DL = I->getDebugLoc();
  MachineMemOperand *MMO = GetMemOperand(MBB, FI, MachineMemOperand::MOLoad);
  unsigned Opc = 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 129-134
```cpp
  if (Mips::CPU16RegsRegClass.hasSubClassEq(RC))
    Opc = Mips::LwRxSpImmX16;
  assert(Opc && "Register class not handled!");
  BuildMI(MBB, I, DL, get(Opc), DestReg).addFrameIndex(FI).addImm(Offset)
    .addMemOperand(MMO);
}
```
- EN: Declares `not`, packaging target-specific state and APIs around `Mips16InstrInfo`.
- CN: 这里声明 `not`，把与 `Mips16InstrInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 136-144
```cpp
bool Mips16InstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
  MachineBasicBlock &MBB = *MI.getParent();
  switch (MI.getDesc().getOpcode()) {
  default:
    return false;
  case Mips::RetRA16:
    ExpandRetRA16(MBB, MI, Mips::JrcRa16);
    break;
  }
```
- EN: Implements `Mips16InstrInfo::expandPostRAPseudo`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::expandPostRAPseudo`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-148
```cpp
  MBB.erase(MI.getIterator());
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 150-164
```cpp
/// GetOppositeBranchOpc - Return the inverse of the specified
/// opcode, e.g. turning BEQ to BNE.
unsigned Mips16InstrInfo::getOppositeBranchOpc(unsigned Opc) const {
  switch (Opc) {
  case Mips::BeqzRxImmX16: return Mips::BnezRxImmX16;
  case Mips::BnezRxImmX16: return Mips::BeqzRxImmX16;
  case Mips::BeqzRxImm16: return Mips::BnezRxImm16;
  case Mips::BnezRxImm16: return Mips::BeqzRxImm16;
  case Mips::BteqzT8CmpX16: return Mips::BtnezT8CmpX16;
  case Mips::BteqzT8SltX16: return Mips::BtnezT8SltX16;
  case Mips::BteqzT8SltiX16: return Mips::BtnezT8SltiX16;
  case Mips::Btnez16: return Mips::Bteqz16;
  case Mips::BtnezX16: return Mips::BteqzX16;
  case Mips::BtnezT8CmpiX16: return Mips::BteqzT8CmpiX16;
  case Mips::BtnezT8SltuX16: return Mips::BteqzT8SltuX16;
```
- EN: Implements `Mips16InstrInfo::getOppositeBranchOpc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::getOppositeBranchOpc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-176
```cpp
  case Mips::BtnezT8SltiuX16: return Mips::BteqzT8SltiuX16;
  case Mips::Bteqz16: return Mips::Btnez16;
  case Mips::BteqzX16: return Mips::BtnezX16;
  case Mips::BteqzT8CmpiX16: return Mips::BtnezT8CmpiX16;
  case Mips::BteqzT8SltuX16: return Mips::BtnezT8SltuX16;
  case Mips::BteqzT8SltiuX16: return Mips::BtnezT8SltiuX16;
  case Mips::BtnezT8CmpX16: return Mips::BteqzT8CmpX16;
  case Mips::BtnezT8SltX16: return Mips::BteqzT8SltX16;
  case Mips::BtnezT8SltiX16: return Mips::BteqzT8SltiX16;
  }
  llvm_unreachable("Illegal opcode!");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 178-192
```cpp
static void addSaveRestoreRegs(MachineInstrBuilder &MIB,
                               ArrayRef<CalleeSavedInfo> CSI,
                               RegState Flags = {}) {
  for (unsigned i = 0, e = CSI.size(); i != e; ++i) {
    // Add the callee-saved register as live-in. Do not add if the register is
    // RA and return address is taken, because it has already been added in
    // method MipsTargetLowering::lowerRETURNADDR.
    // It's killed at the spill, unless the register is RA and return address
    // is taken.
    Register Reg = CSI[e-i-1].getReg();
    switch (Reg) {
    case Mips::RA:
    case Mips::S0:
    case Mips::S1:
      MIB.addReg(Reg, Flags);
```
- EN: Implements `addSaveRestoreRegs`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addSaveRestoreRegs`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-197
```cpp
      break;
    case Mips::S2:
      break;
    default:
      llvm_unreachable("unexpected mips16 callee saved register");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 199-201
```cpp
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 203-217
```cpp
// Adjust SP by FrameSize bytes. Save RA, S0, S1
void Mips16InstrInfo::makeFrame(unsigned SP, int64_t FrameSize,
                                MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator I) const {
  DebugLoc DL;
  MachineFunction &MF = *MBB.getParent();
  MachineFrameInfo &MFI    = MF.getFrameInfo();
  const BitVector Reserved = RI.getReservedRegs(MF);
  bool SaveS2 = Reserved[Mips::S2];
  MachineInstrBuilder MIB;
  unsigned Opc = ((FrameSize <= 128) && !SaveS2)? Mips::Save16:Mips::SaveX16;
  MIB = BuildMI(MBB, I, DL, get(Opc));
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
  addSaveRestoreRegs(MIB, CSI);
  if (SaveS2)
```
- EN: Implements `Mips16InstrInfo::makeFrame`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::makeFrame`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 218-231
```cpp
    MIB.addReg(Mips::S2);
  if (isUInt<11>(FrameSize))
    MIB.addImm(FrameSize);
  else {
    int Base = 2040; // should create template function like isUInt that
                     // returns largest possible n bit unsigned integer
    int64_t Remainder = FrameSize - Base;
    MIB.addImm(Base);
    if (isInt<16>(-Remainder))
      BuildAddiuSpImm(MBB, I, -Remainder);
    else
      adjustStackPtrBig(SP, -Remainder, MBB, I, Mips::V0, Mips::V1);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 233-244
```cpp
// Adjust SP by FrameSize bytes. Restore RA, S0, S1
void Mips16InstrInfo::restoreFrame(unsigned SP, int64_t FrameSize,
                                   MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator I) const {
  DebugLoc DL = I != MBB.end() ? I->getDebugLoc() : DebugLoc();
  MachineFunction *MF = MBB.getParent();
  MachineFrameInfo &MFI    = MF->getFrameInfo();
  const BitVector Reserved = RI.getReservedRegs(*MF);
  bool SaveS2 = Reserved[Mips::S2];
  MachineInstrBuilder MIB;
  unsigned Opc = ((FrameSize <= 128) && !SaveS2)?
    Mips::Restore16:Mips::RestoreX16;
```
- EN: Implements `Mips16InstrInfo::restoreFrame`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::restoreFrame`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-250
```cpp
  if (!isUInt<11>(FrameSize)) {
    unsigned Base = 2040;
    int64_t Remainder = FrameSize - Base;
    FrameSize = Base; // should create template function like isUInt that
                     // returns largest possible n bit unsigned integer
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 252-263
```cpp
    if (isInt<16>(Remainder))
      BuildAddiuSpImm(MBB, I, Remainder);
    else
      adjustStackPtrBig(SP, Remainder, MBB, I, Mips::A0, Mips::A1);
  }
  MIB = BuildMI(MBB, I, DL, get(Opc));
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
  addSaveRestoreRegs(MIB, CSI, RegState::Define);
  if (SaveS2)
    MIB.addReg(Mips::S2, RegState::Define);
  MIB.addImm(FrameSize);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 265-279
```cpp
// Adjust SP by Amount bytes where bytes can be up to 32bit number.
// This can only be called at times that we know that there is at least one free
// register.
// This is clearly safe at prologue and epilogue.
void Mips16InstrInfo::adjustStackPtrBig(unsigned SP, int64_t Amount,
                                        MachineBasicBlock &MBB,
                                        MachineBasicBlock::iterator I,
                                        unsigned Reg1, unsigned Reg2) const {
  DebugLoc DL;
  //
  // li reg1, constant
  // move reg2, sp
  // add reg1, reg1, reg2
  // move sp, reg1
  //
```
- EN: Implements `Mips16InstrInfo::adjustStackPtrBig`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::adjustStackPtrBig`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 280-291
```cpp
  //
  MachineInstrBuilder MIB1 = BuildMI(MBB, I, DL, get(Mips::LwConstant32), Reg1);
  MIB1.addImm(Amount).addImm(-1);
  MachineInstrBuilder MIB2 = BuildMI(MBB, I, DL, get(Mips::MoveR3216), Reg2);
  MIB2.addReg(Mips::SP, RegState::Kill);
  MachineInstrBuilder MIB3 = BuildMI(MBB, I, DL, get(Mips::AdduRxRyRz16), Reg1);
  MIB3.addReg(Reg1);
  MIB3.addReg(Reg2, RegState::Kill);
  MachineInstrBuilder MIB4 = BuildMI(MBB, I, DL, get(Mips::Move32R16),
                                                     Mips::SP);
  MIB4.addReg(Reg1, RegState::Kill);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 293-297
```cpp
void Mips16InstrInfo::adjustStackPtrBigUnrestricted(
    unsigned SP, int64_t Amount, MachineBasicBlock &MBB,
    MachineBasicBlock::iterator I) const {
   llvm_unreachable("adjust stack pointer amount exceeded");
}
```
- EN: Implements `Mips16InstrInfo::adjustStackPtrBigUnrestricted`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::adjustStackPtrBigUnrestricted`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 299-304
```cpp
/// Adjust SP by Amount bytes.
void Mips16InstrInfo::adjustStackPtr(unsigned SP, int64_t Amount,
                                     MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator I) const {
  if (Amount == 0)
    return;
```
- EN: Implements `Mips16InstrInfo::adjustStackPtr`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::adjustStackPtr`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 306-310
```cpp
  if (isInt<16>(Amount))  // need to change to addiu sp, ....and isInt<16>
    BuildAddiuSpImm(MBB, I, Amount);
  else
    adjustStackPtrBigUnrestricted(SP, Amount, MBB, I);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 312-326
```cpp
/// This function generates the sequence of instructions needed to get the
/// result of adding register REG and immediate IMM.
unsigned Mips16InstrInfo::loadImmediate(unsigned FrameReg, int64_t Imm,
                                        MachineBasicBlock &MBB,
                                        MachineBasicBlock::iterator II,
                                        const DebugLoc &DL,
                                        unsigned &NewImm) const {
  //
  // given original instruction is:
  // Instr rx, T[offset] where offset is too big.
  //
  // lo = offset & 0xFFFF
  // hi = ((offset >> 16) + (lo >> 15)) & 0xFFFF;
  //
  // let T = temporary register
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 327-335
```cpp
  // li T, hi
  // shl T, 16
  // add T, Rx, T
  //
  RegScavenger rs;
  int32_t lo = Imm & 0xFFFF;
  NewImm = lo;
  int Reg =0;
  int SpReg = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 337-351
```cpp
  rs.enterBasicBlockEnd(MBB);
  rs.backward(std::next(II));
  //
  // We need to know which registers can be used, in the case where there
  // are not enough free registers. We exclude all registers that
  // are used in the instruction that we are helping.
  //  // Consider all allocatable registers in the register class initially
  BitVector Candidates =
      RI.getAllocatableSet
      (*II->getParent()->getParent(), &Mips::CPU16RegsRegClass);
  // Exclude all the registers being used by the instruction.
  for (MachineOperand &MO : II->operands()) {
    if (MO.isReg() && MO.getReg() != 0 && !MO.isDef() &&
        !MO.getReg().isVirtual())
      Candidates.reset(MO.getReg());
```
- EN: Declares `initially`, packaging target-specific state and APIs around `Mips16InstrInfo`.
- CN: 这里声明 `initially`，把与 `Mips16InstrInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 352-352
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 354-368
```cpp
  // If the same register was used and defined in an instruction, then
  // it will not be in the list of candidates.
  //
  // we need to analyze the instruction that we are helping.
  // we need to know if it defines register x but register x is not
  // present as an operand of the instruction. this tells
  // whether the register is live before the instruction. if it's not
  // then we don't need to save it in case there are no free registers.
  int DefReg = 0;
  for (MachineOperand &MO : II->operands()) {
    if (MO.isReg() && MO.isDef()) {
      DefReg = MO.getReg();
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 370-377
```cpp
  BitVector Available = rs.getRegsAvailable(&Mips::CPU16RegsRegClass);
  Available &= Candidates;
  //
  // we use T0 for the first register, if we need to save something away.
  // we use T1 for the second register, if we need to save something away.
  //
  unsigned FirstRegSaved =0, SecondRegSaved=0;
  unsigned FirstRegSavedTo = 0, SecondRegSavedTo = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 379-379
```cpp
  Reg = Available.find_first();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 381-395
```cpp
  if (Reg == -1) {
    Reg = Candidates.find_first();
    Candidates.reset(Reg);
    if (DefReg != Reg) {
      FirstRegSaved = Reg;
      FirstRegSavedTo = Mips::T0;
      copyPhysReg(MBB, II, DL, FirstRegSavedTo, FirstRegSaved, true);
    }
  }
  else
    Available.reset(Reg);
  BuildMI(MBB, II, DL, get(Mips::LwConstant32), Reg).addImm(Imm).addImm(-1);
  NewImm = 0;
  if (FrameReg == Mips::SP) {
    SpReg = Available.find_first();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 396-410
```cpp
    if (SpReg == -1) {
      SpReg = Candidates.find_first();
      // Candidates.reset(SpReg); // not really needed
      if (DefReg!= SpReg) {
        SecondRegSaved = SpReg;
        SecondRegSavedTo = Mips::T1;
      }
      if (SecondRegSaved)
        copyPhysReg(MBB, II, DL, SecondRegSavedTo, SecondRegSaved, true);
    } else {
      Available.reset(SpReg);
    }
    copyPhysReg(MBB, II, DL, SpReg, Mips::SP, false);
    BuildMI(MBB, II, DL, get(Mips::AdduRxRyRz16), Reg)
        .addReg(SpReg, RegState::Kill)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 411-424
```cpp
        .addReg(Reg);
  }
  else
    BuildMI(MBB, II, DL, get(Mips::  AdduRxRyRz16), Reg).addReg(FrameReg)
      .addReg(Reg, RegState::Kill);
  if (FirstRegSaved || SecondRegSaved) {
    II = std::next(II);
    if (FirstRegSaved)
      copyPhysReg(MBB, II, DL, FirstRegSaved, FirstRegSavedTo, true);
    if (SecondRegSaved)
      copyPhysReg(MBB, II, DL, SecondRegSaved, SecondRegSavedTo, true);
  }
  return Reg;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 426-439
```cpp
unsigned Mips16InstrInfo::getAnalyzableBrOpc(unsigned Opc) const {
  return (Opc == Mips::BeqzRxImmX16   || Opc == Mips::BimmX16  ||
          Opc == Mips::Bimm16  ||
          Opc == Mips::Bteqz16        || Opc == Mips::Btnez16 ||
          Opc == Mips::BeqzRxImm16    || Opc == Mips::BnezRxImm16   ||
          Opc == Mips::BnezRxImmX16   || Opc == Mips::BteqzX16 ||
          Opc == Mips::BteqzT8CmpX16  || Opc == Mips::BteqzT8CmpiX16 ||
          Opc == Mips::BteqzT8SltX16  || Opc == Mips::BteqzT8SltuX16  ||
          Opc == Mips::BteqzT8SltiX16 || Opc == Mips::BteqzT8SltiuX16 ||
          Opc == Mips::BtnezX16       || Opc == Mips::BtnezT8CmpX16 ||
          Opc == Mips::BtnezT8CmpiX16 || Opc == Mips::BtnezT8SltX16 ||
          Opc == Mips::BtnezT8SltuX16 || Opc == Mips::BtnezT8SltiX16 ||
          Opc == Mips::BtnezT8SltiuX16 ) ? Opc : 0;
}
```
- EN: Implements `Mips16InstrInfo::getAnalyzableBrOpc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::getAnalyzableBrOpc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 441-445
```cpp
void Mips16InstrInfo::ExpandRetRA16(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator I,
                                  unsigned Opc) const {
  BuildMI(MBB, I, I->getDebugLoc(), get(Opc));
}
```
- EN: Implements `Mips16InstrInfo::ExpandRetRA16`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::ExpandRetRA16`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 447-452
```cpp
const MCInstrDesc &Mips16InstrInfo::AddiuSpImm(int64_t Imm) const {
  if (validSpImm8(Imm))
    return get(Mips::AddiuSpImm16);
  else
    return get(Mips::AddiuSpImmX16);
}
```
- EN: Implements `Mips16InstrInfo::AddiuSpImm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::AddiuSpImm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 454-458
```cpp
void Mips16InstrInfo::BuildAddiuSpImm
  (MachineBasicBlock &MBB, MachineBasicBlock::iterator I, int64_t Imm) const {
  DebugLoc DL;
  BuildMI(MBB, I, DL, AddiuSpImm(Imm)).addImm(Imm);
}
```
- EN: Implements `Mips16InstrInfo::BuildAddiuSpImm`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::BuildAddiuSpImm`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 460-462
```cpp
const MipsInstrInfo *llvm::createMips16InstrInfo(const MipsSubtarget &STI) {
  return new Mips16InstrInfo(STI);
}
```
- EN: Implements `llvm::createMips16InstrInfo`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMips16InstrInfo`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 464-478
```cpp
bool Mips16InstrInfo::validImmediate(unsigned Opcode, unsigned Reg,
                                     int64_t Amount) {
  switch (Opcode) {
  case Mips::LbRxRyOffMemX16:
  case Mips::LbuRxRyOffMemX16:
  case Mips::LhRxRyOffMemX16:
  case Mips::LhuRxRyOffMemX16:
  case Mips::SbRxRyOffMemX16:
  case Mips::ShRxRyOffMemX16:
  case Mips::LwRxRyOffMemX16:
  case Mips::SwRxRyOffMemX16:
  case Mips::SwRxSpImmX16:
  case Mips::LwRxSpImmX16:
    return isInt<16>(Amount);
  case Mips::AddiuRxRyOffMemX16:
```
- EN: Implements `Mips16InstrInfo::validImmediate`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips16InstrInfo::validImmediate`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 479-484
```cpp
    if ((Reg == Mips::PC) || (Reg == Mips::SP))
      return isInt<16>(Amount);
    return isInt<15>(Amount);
  }
  llvm_unreachable("unexpected Opcode in validImmediate");
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

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

- EN: Backend-local headers: `Mips16InstrInfo.h`.
  - CN: 后端本地头文件：`Mips16InstrInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/MachineOperand.h` ... (+5 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/CodeGen/MachineOperand.h` ... (+5 more)。
- EN: Standard/system headers: `cassert`, `cctype`, `cstdint`, `cstdlib`, `cstring`, `iterator`, `vector`.
  - CN: 标准库/系统头文件：`cassert`, `cctype`, `cstdint`, `cstdlib`, `cstring`, `iterator`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
