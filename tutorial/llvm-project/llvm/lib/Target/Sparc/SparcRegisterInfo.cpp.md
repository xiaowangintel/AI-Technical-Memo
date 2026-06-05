# SparcRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcRegisterInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific register information, reserved registers, spills, and frame-index handling.
  - **CN**: 实现目标相关的寄存器信息、保留寄存器、溢出处理以及 frame index 处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcRegisterInfo.cpp - SPARC Register Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-21
```cpp
//
// This file contains the SPARC implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//

#include "SparcRegisterInfo.h"
#include "Sparc.h"
#include "SparcSubtarget.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/IR/Type.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `SparcRegisterInfo.h`, `Sparc.h`, `SparcSubtarget.h`, `llvm/ADT/BitVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcRegisterInfo.h`, `Sparc.h`, `SparcSubtarget.h`, `llvm/ADT/BitVector.h`。

### Lines 22-29
```cpp
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;

#define GET_REGINFO_TARGET_DESC
#include "SparcGenRegisterInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `SparcGenRegisterInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `SparcGenRegisterInfo.inc`。

### Lines 30-36
```cpp
static cl::opt<bool>
ReserveAppRegisters("sparc-reserve-app-registers", cl::Hidden, cl::init(false),
                    cl::desc("Reserve application registers (%g2-%g4)"));

SparcRegisterInfo::SparcRegisterInfo(const SparcSubtarget &STI)
    : SparcGenRegisterInfo(SP::O7), Is64Bit(STI.is64Bit()) {}

```
- **EN**: Implements logic around `ReserveAppRegisters`, `desc`, `SparcRegisterInfo`, `SparcGenRegisterInfo`.
- **CN**: 围绕 `ReserveAppRegisters`, `desc`, `SparcRegisterInfo`, `SparcGenRegisterInfo` 实现具体逻辑。

### Lines 37-47
```cpp
const MCPhysReg*
SparcRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  return CSR_SaveList;
}

const uint32_t *
SparcRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
                                        CallingConv::ID CC) const {
  return CSR_RegMask;
}

```
- **EN**: Implements logic around `getCalleeSavedRegs`, `getCallPreservedMask`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getCalleeSavedRegs`, `getCallPreservedMask` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 48-58
```cpp
const uint32_t*
SparcRegisterInfo::getRTCallPreservedMask(CallingConv::ID CC) const {
  return RTCSR_RegMask;
}

BitVector SparcRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  BitVector Reserved(getNumRegs());
  const SparcSubtarget &Subtarget = MF.getSubtarget<SparcSubtarget>();
  // FIXME: G1 reserved for now for large imm generation by frame code.
  Reserved.set(SP::G1);

```
- **EN**: Implements logic around `getRTCallPreservedMask`, `getReservedRegs`, `Reserved`, `getSubtarget<SparcSubtarget>`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getRTCallPreservedMask`, `getReservedRegs`, `Reserved`, `getSubtarget<SparcSubtarget>`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 59-68
```cpp
  // G1-G4 can be used in applications.
  if (ReserveAppRegisters) {
    Reserved.set(SP::G2);
    Reserved.set(SP::G3);
    Reserved.set(SP::G4);
  }
  // G5 is not reserved in 64 bit mode.
  if (!Subtarget.is64Bit())
    Reserved.set(SP::G5);

```
- **EN**: Implements logic around `set`; this block applies conditional target rules.
- **CN**: 围绕 `set` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 69-75
```cpp
  Reserved.set(SP::O6);
  Reserved.set(SP::I6);
  Reserved.set(SP::I7);
  Reserved.set(SP::G0);
  Reserved.set(SP::G6);
  Reserved.set(SP::G7);

```
- **EN**: Implements logic around `set`.
- **CN**: 围绕 `set` 实现具体逻辑。

### Lines 76-83
```cpp
  // Also reserve the register pair aliases covering the above
  // registers, with the same conditions.
  Reserved.set(SP::G0_G1);
  if (ReserveAppRegisters)
    Reserved.set(SP::G2_G3);
  if (ReserveAppRegisters || !Subtarget.is64Bit())
    Reserved.set(SP::G4_G5);

```
- **EN**: Implements logic around `set`; this block applies conditional target rules.
- **CN**: 围绕 `set` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 84-95
```cpp
  Reserved.set(SP::O6_O7);
  Reserved.set(SP::I6_I7);
  Reserved.set(SP::G6_G7);

  // Unaliased double registers are not available in non-V9 targets.
  if (!Subtarget.isV9()) {
    for (unsigned n = 0; n != 16; ++n) {
      for (MCRegAliasIterator AI(SP::D16 + n, this, true); AI.isValid(); ++AI)
        Reserved.set(*AI);
    }
  }

```
- **EN**: Implements logic around `set`; this block applies conditional target rules.
- **CN**: 围绕 `set` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 96-105
```cpp
  // Reserve ASR1-ASR31
  for (unsigned n = 0; n < 31; n++)
    Reserved.set(SP::ASR1 + n);

  for (TargetRegisterClass::iterator i = SP::IntRegsRegClass.begin();
       i != SP::IntRegsRegClass.end(); ++i) {
    if (MF.getSubtarget<SparcSubtarget>().isRegisterReserved(*i))
      markSuperRegs(Reserved, *i);
  }

```
- **EN**: Implements logic around `set`, `end`, `markSuperRegs`; this block applies conditional target rules.
- **CN**: 围绕 `set`, `end`, `markSuperRegs` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 106-114
```cpp
  assert(checkAllSuperRegsMarked(Reserved));
  return Reserved;
}

bool SparcRegisterInfo::isReservedReg(const MachineFunction &MF,
                                      MCRegister Reg) const {
  return getReservedRegs(MF)[Reg];
}

```
- **EN**: Implements logic around `assert`, `isReservedReg`, `getReservedRegs`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `isReservedReg`, `getReservedRegs` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 115-128
```cpp
const TargetRegisterClass *
SparcRegisterInfo::getPointerRegClass(unsigned Kind) const {
  assert(Kind == 0 && "this should only be used for default cases");
  return Is64Bit ? &SP::I64RegsRegClass : &SP::IntRegsRegClass;
}

static void replaceFI(MachineFunction &MF, MachineBasicBlock::iterator II,
                      MachineInstr &MI, const DebugLoc &dl,
                      unsigned FIOperandNum, int Offset, unsigned FramePtr) {
  // Replace frame index with a frame pointer reference.
  if (Offset >= -4096 && Offset <= 4095) {
    // If the offset is small enough to fit in the immediate field, directly
    // encode it.
    MI.getOperand(FIOperandNum).ChangeToRegister(FramePtr, false);
```
- **EN**: Implements logic around `getPointerRegClass`, `assert`, `replaceFI`, `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getPointerRegClass`, `assert`, `replaceFI`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 129-142
```cpp
    MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
    return;
  }

  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();

  // FIXME: it would be better to scavenge a register here instead of
  // reserving G1 all of the time.
  if (Offset >= 0) {
    // Emit nonnegaive immediates with sethi + or.
    // sethi %hi(Offset), %g1
    // add %g1, %fp, %g1
    // Insert G1+%lo(offset) into the user.
    BuildMI(*MI.getParent(), II, dl, TII.get(SP::SETHIi), SP::G1)
```
- **EN**: Implements logic around `getOperand`, `getSubtarget`, `BuildMI`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `getSubtarget`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 143-154
```cpp
      .addImm(HI22(Offset));


    // Emit G1 = G1 + I6
    BuildMI(*MI.getParent(), II, dl, TII.get(SP::ADDrr), SP::G1).addReg(SP::G1)
      .addReg(FramePtr);
    // Insert: G1+%lo(offset) into the user.
    MI.getOperand(FIOperandNum).ChangeToRegister(SP::G1, false);
    MI.getOperand(FIOperandNum + 1).ChangeToImmediate(LO10(Offset));
    return;
  }

```
- **EN**: Implements logic around `addImm`, `BuildMI`, `addReg`, `getOperand`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addImm`, `BuildMI`, `addReg`, `getOperand` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 155-164
```cpp
  // Emit Negative numbers with sethi + xor
  // sethi %hix(Offset), %g1
  // xor  %g1, %lox(offset), %g1
  // add %g1, %fp, %g1
  // Insert: G1 + 0 into the user.
  BuildMI(*MI.getParent(), II, dl, TII.get(SP::SETHIi), SP::G1)
    .addImm(HIX22(Offset));
  BuildMI(*MI.getParent(), II, dl, TII.get(SP::XORri), SP::G1)
    .addReg(SP::G1).addImm(LOX10(Offset));

```
- **EN**: Implements logic around `BuildMI`, `addImm`, `addReg`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addImm`, `addReg` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 165-171
```cpp
  BuildMI(*MI.getParent(), II, dl, TII.get(SP::ADDrr), SP::G1).addReg(SP::G1)
    .addReg(FramePtr);
  // Insert: G1+%lo(offset) into the user.
  MI.getOperand(FIOperandNum).ChangeToRegister(SP::G1, false);
  MI.getOperand(FIOperandNum + 1).ChangeToImmediate(0);
}

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `getOperand`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `getOperand` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 172-178
```cpp

bool
SparcRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                       int SPAdj, unsigned FIOperandNum,
                                       RegScavenger *RS) const {
  assert(SPAdj == 0 && "Unexpected");

```
- **EN**: Implements logic around `eliminateFrameIndex`, `assert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateFrameIndex`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 179-185
```cpp
  MachineInstr &MI = *II;
  DebugLoc dl = MI.getDebugLoc();
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
  MachineFunction &MF = *MI.getParent()->getParent();
  const SparcSubtarget &Subtarget = MF.getSubtarget<SparcSubtarget>();
  const SparcFrameLowering *TFI = getFrameLowering(MF);

```
- **EN**: Implements logic around `getDebugLoc`, `getOperand`, `getParent`, `getSubtarget<SparcSubtarget>`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getDebugLoc`, `getOperand`, `getParent`, `getSubtarget<SparcSubtarget>`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 186-199
```cpp
  Register FrameReg;
  int Offset;
  Offset = TFI->getFrameIndexReference(MF, FrameIndex, FrameReg).getFixed();

  Offset += MI.getOperand(FIOperandNum + 1).getImm();

  if (!Subtarget.isV9() || !Subtarget.hasHardQuad()) {
    if (MI.getOpcode() == SP::STQFri) {
      const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
      Register SrcReg = MI.getOperand(2).getReg();
      Register SrcEvenReg = getSubReg(SrcReg, SP::sub_even64);
      Register SrcOddReg = getSubReg(SrcReg, SP::sub_odd64);
      MachineInstr *StMI =
        BuildMI(*MI.getParent(), II, dl, TII.get(SP::STDFri))
```
- **EN**: Implements logic around `getFrameIndexReference`, `getOperand`, `getInstrInfo`, `getSubReg`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameIndexReference`, `getOperand`, `getInstrInfo`, `getSubReg`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 200-213
```cpp
        .addReg(FrameReg).addImm(0).addReg(SrcEvenReg);
      replaceFI(MF, *StMI, *StMI, dl, 0, Offset, FrameReg);
      MI.setDesc(TII.get(SP::STDFri));
      MI.getOperand(2).setReg(SrcOddReg);
      Offset += 8;
    } else if (MI.getOpcode() == SP::LDQFri) {
      const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
      Register DestReg = MI.getOperand(0).getReg();
      Register DestEvenReg = getSubReg(DestReg, SP::sub_even64);
      Register DestOddReg = getSubReg(DestReg, SP::sub_odd64);
      MachineInstr *LdMI =
        BuildMI(*MI.getParent(), II, dl, TII.get(SP::LDDFri), DestEvenReg)
        .addReg(FrameReg).addImm(0);
      replaceFI(MF, *LdMI, *LdMI, dl, 1, Offset, FrameReg);
```
- **EN**: Implements logic around `addReg`, `replaceFI`, `setDesc`, `getOperand`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addReg`, `replaceFI`, `setDesc`, `getOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 214-220
```cpp

      MI.setDesc(TII.get(SP::LDDFri));
      MI.getOperand(0).setReg(DestOddReg);
      Offset += 8;
    }
  }

```
- **EN**: Implements logic around `setDesc`, `getOperand`.
- **CN**: 围绕 `setDesc`, `getOperand` 实现具体逻辑。

### Lines 221-228
```cpp
  replaceFI(MF, II, MI, dl, FIOperandNum, Offset, FrameReg);
  // replaceFI never removes II
  return false;
}

Register SparcRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  return SP::I6;
}
```
- **EN**: Implements logic around `replaceFI`, `getFrameRegister`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `replaceFI`, `getFrameRegister` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Register model / 寄存器模型**:
  - **EN**: Describes registers, classes, and allocation-facing details
  - **CN**: 描述寄存器、寄存器类以及面向分配器的细节
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcRegisterInfo.h`, `Sparc.h`, `SparcSubtarget.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/IR/Type.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `SparcGenRegisterInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
- **Generated macros / 生成宏**: `GET_REGINFO_TARGET_DESC`
