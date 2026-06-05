# MSP430RegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430RegisterInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific register information, reserved registers, spills, and frame-index handling.
  - **CN**: 实现目标相关的寄存器信息、保留寄存器、溢出处理以及 frame index 处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MSP430RegisterInfo.cpp - MSP430 Register Information --------------===//
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
// This file contains the MSP430 implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//

#include "MSP430RegisterInfo.h"
#include "MSP430TargetMachine.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/IR/Function.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Target/TargetOptions.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430RegisterInfo.h`, `MSP430TargetMachine.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430RegisterInfo.h`, `MSP430TargetMachine.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`。

### Lines 22-29
```cpp

using namespace llvm;

#define DEBUG_TYPE "msp430-reg-info"

#define GET_REGINFO_TARGET_DESC
#include "MSP430GenRegisterInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenRegisterInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenRegisterInfo.inc`。

### Lines 30-43
```cpp
// FIXME: Provide proper call frame setup / destroy opcodes.
MSP430RegisterInfo::MSP430RegisterInfo()
  : MSP430GenRegisterInfo(MSP430::PC) {}

const MCPhysReg*
MSP430RegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  const MSP430FrameLowering *TFI = getFrameLowering(*MF);
  const Function* F = &MF->getFunction();
  static const MCPhysReg CalleeSavedRegs[] = {
    MSP430::R4, MSP430::R5, MSP430::R6, MSP430::R7,
    MSP430::R8, MSP430::R9, MSP430::R10,
    0
  };
  static const MCPhysReg CalleeSavedRegsFP[] = {
```
- **EN**: Implements logic around `MSP430RegisterInfo`, `MSP430GenRegisterInfo`, `getCalleeSavedRegs`, `getFrameLowering`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `MSP430RegisterInfo`, `MSP430GenRegisterInfo`, `getCalleeSavedRegs`, `getFrameLowering`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 44-57
```cpp
    MSP430::R5, MSP430::R6, MSP430::R7,
    MSP430::R8, MSP430::R9, MSP430::R10,
    0
  };
  static const MCPhysReg CalleeSavedRegsIntr[] = {
    MSP430::R4,  MSP430::R5,  MSP430::R6,  MSP430::R7,
    MSP430::R8,  MSP430::R9,  MSP430::R10, MSP430::R11,
    MSP430::R12, MSP430::R13, MSP430::R14, MSP430::R15,
    0
  };
  static const MCPhysReg CalleeSavedRegsIntrFP[] = {
    MSP430::R5,  MSP430::R6,  MSP430::R7,
    MSP430::R8,  MSP430::R9,  MSP430::R10, MSP430::R11,
    MSP430::R12, MSP430::R13, MSP430::R14, MSP430::R15,
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 58-67
```cpp
    0
  };

  if (TFI->hasFP(*MF))
    return (F->getCallingConv() == CallingConv::MSP430_INTR ?
            CalleeSavedRegsIntrFP : CalleeSavedRegsFP);
  else
    return (F->getCallingConv() == CallingConv::MSP430_INTR ?
            CalleeSavedRegsIntr : CalleeSavedRegs);

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 68-81
```cpp
}

BitVector MSP430RegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  BitVector Reserved(getNumRegs());
  const MSP430FrameLowering *TFI = getFrameLowering(MF);

  // Mark 4 special registers with subregisters as reserved.
  Reserved.set(MSP430::PCB);
  Reserved.set(MSP430::SPB);
  Reserved.set(MSP430::SRB);
  Reserved.set(MSP430::CGB);
  Reserved.set(MSP430::PC);
  Reserved.set(MSP430::SP);
  Reserved.set(MSP430::SR);
```
- **EN**: Implements logic around `getReservedRegs`, `Reserved`, `getFrameLowering`, `set`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getReservedRegs`, `Reserved`, `getFrameLowering`, `set` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 82-89
```cpp
  Reserved.set(MSP430::CG);

  // Mark frame pointer as reserved if needed.
  if (TFI->hasFP(MF)) {
    Reserved.set(MSP430::R4B);
    Reserved.set(MSP430::R4);
  }

```
- **EN**: Implements logic around `set`; this block applies conditional target rules.
- **CN**: 围绕 `set` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 90-97
```cpp
  return Reserved;
}

const TargetRegisterClass *
MSP430RegisterInfo::getPointerRegClass(unsigned Kind) const {
  return &MSP430::GR16RegClass;
}

```
- **EN**: Implements logic around `getPointerRegClass`; this block returns target-specific results.
- **CN**: 围绕 `getPointerRegClass` 实现具体逻辑；这一段返回目标相关结果。

### Lines 98-110
```cpp
bool
MSP430RegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                        int SPAdj, unsigned FIOperandNum,
                                        RegScavenger *RS) const {
  assert(SPAdj == 0 && "Unexpected");

  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  const MSP430FrameLowering *TFI = getFrameLowering(MF);
  DebugLoc dl = MI.getDebugLoc();
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();

```
- **EN**: Implements logic around `eliminateFrameIndex`, `assert`, `getParent`, `getFrameLowering`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateFrameIndex`, `assert`, `getParent`, `getFrameLowering`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 111-121
```cpp
  unsigned BasePtr = (TFI->hasFP(MF) ? MSP430::R4 : MSP430::SP);
  int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex);

  // Skip the saved PC
  Offset += 2;

  if (!TFI->hasFP(MF))
    Offset += MF.getFrameInfo().getStackSize();
  else
    Offset += 2; // Skip the saved FP

```
- **EN**: Implements logic around `hasFP`, `getFrameInfo`; this block applies conditional target rules.
- **CN**: 围绕 `hasFP`, `getFrameInfo` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 122-130
```cpp
  // Fold imm into offset
  Offset += MI.getOperand(FIOperandNum + 1).getImm();

  if (MI.getOpcode() == MSP430::ADDframe) {
    // This is actually "load effective address" of the stack slot
    // instruction. We have only two-address instructions, thus we need to
    // expand it into mov + add
    const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();

```
- **EN**: Implements logic around `getOperand`, `getSubtarget`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand`, `getSubtarget` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 131-139
```cpp
    MI.setDesc(TII.get(MSP430::MOV16rr));
    MI.getOperand(FIOperandNum).ChangeToRegister(BasePtr, false);

    // Remove the now unused Offset operand.
    MI.removeOperand(FIOperandNum + 1);

    if (Offset == 0)
      return false;

```
- **EN**: Implements logic around `setDesc`, `getOperand`, `removeOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `setDesc`, `getOperand`, `removeOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 140-148
```cpp
    // We need to materialize the offset via add instruction.
    Register DstReg = MI.getOperand(0).getReg();
    if (Offset < 0)
      BuildMI(MBB, std::next(II), dl, TII.get(MSP430::SUB16ri), DstReg)
        .addReg(DstReg).addImm(-Offset);
    else
      BuildMI(MBB, std::next(II), dl, TII.get(MSP430::ADD16ri), DstReg)
        .addReg(DstReg).addImm(Offset);

```
- **EN**: Implements logic around `getOperand`, `BuildMI`, `addReg`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `BuildMI`, `addReg` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 149-156
```cpp
    return false;
  }

  MI.getOperand(FIOperandNum).ChangeToRegister(BasePtr, false);
  MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
  return false;
}

```
- **EN**: Implements logic around `getOperand`; this block returns target-specific results.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 157-160
```cpp
Register MSP430RegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  const MSP430FrameLowering *TFI = getFrameLowering(MF);
  return TFI->hasFP(MF) ? MSP430::R4 : MSP430::SP;
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

- **Direct includes / 直接包含**: `MSP430RegisterInfo.h`, `MSP430TargetMachine.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/IR/Function.h`, `llvm/Target/TargetMachine.h`, `llvm/Target/TargetOptions.h`, `MSP430GenRegisterInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR
- **Generated macros / 生成宏**: `GET_REGINFO_TARGET_DESC`
