# XtensaRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaRegisterInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific register information, reserved registers, spills, and frame-index handling.
  - **CN**: 实现目标相关的寄存器信息、保留寄存器、溢出处理以及 frame index 处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- XtensaRegisterInfo.cpp - Xtensa Register Information ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file contains the Xtensa implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp
#include "XtensaRegisterInfo.h"
#include "MCTargetDesc/XtensaMCTargetDesc.h"
#include "XtensaInstrInfo.h"
#include "XtensaSubtarget.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaRegisterInfo.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `XtensaInstrInfo.h`, `XtensaSubtarget.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaRegisterInfo.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `XtensaInstrInfo.h`, `XtensaSubtarget.h`。

### Lines 23-29
```cpp
#include "llvm/Support/raw_ostream.h"

#define DEBUG_TYPE "xtensa-reg-info"

#define GET_REGINFO_TARGET_DESC
#include "XtensaGenRegisterInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/raw_ostream.h`, `XtensaGenRegisterInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/raw_ostream.h`, `XtensaGenRegisterInfo.inc`。

### Lines 30-34
```cpp
using namespace llvm;

XtensaRegisterInfo::XtensaRegisterInfo(const XtensaSubtarget &STI)
    : XtensaGenRegisterInfo(Xtensa::A0), Subtarget(STI) {}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 35-40
```cpp
const uint16_t *
XtensaRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  return Subtarget.isWindowedABI() ? CSRW8_Xtensa_SaveList
                                   : CSR_Xtensa_SaveList;
}

```
- **EN**: Implements logic around `getCalleeSavedRegs`, `isWindowedABI`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getCalleeSavedRegs`, `isWindowedABI` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 41-46
```cpp
const uint32_t *
XtensaRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
                                         CallingConv::ID) const {
  return Subtarget.isWindowedABI() ? CSRW8_Xtensa_RegMask : CSR_Xtensa_RegMask;
}

```
- **EN**: Implements logic around `getCallPreservedMask`, `isWindowedABI`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getCallPreservedMask`, `isWindowedABI` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 47-56
```cpp
BitVector XtensaRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  BitVector Reserved(getNumRegs());
  const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();

  Reserved.set(Xtensa::A0);
  if (TFI->hasFP(MF)) {
    // Reserve frame pointer.
    Reserved.set(getFrameRegister(MF));
  }
  if (Subtarget.hasTHREADPTR()) {
```
- **EN**: Implements logic around `getReservedRegs`, `Reserved`, `getSubtarget`, `set`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getReservedRegs`, `Reserved`, `getSubtarget`, `set` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 57-64
```cpp
    // Reserve frame pointer.
    Reserved.set(Xtensa::THREADPTR);
  }
  // Reserve stack pointer.
  Reserved.set(Xtensa::SP);
  return Reserved;
}

```
- **EN**: Implements logic around `set`; this block returns target-specific results.
- **CN**: 围绕 `set` 实现具体逻辑；这一段返回目标相关结果。

### Lines 65-74
```cpp
bool XtensaRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                             int SPAdj, unsigned FIOperandNum,
                                             RegScavenger *RS) const {
  MachineInstr &MI = *II;
  MachineFunction &MF = *MI.getParent()->getParent();
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
  uint64_t StackSize = MF.getFrameInfo().getStackSize();
  int64_t SPOffset = MF.getFrameInfo().getObjectOffset(FrameIndex);
  MachineFrameInfo &MFI = MF.getFrameInfo();
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
```
- **EN**: Implements logic around `eliminateFrameIndex`, `getParent`, `getOperand`, `getFrameInfo`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateFrameIndex`, `getParent`, `getOperand`, `getFrameInfo`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 75-84
```cpp
  int MinCSFI = 0;
  int MaxCSFI = -1;

  if (CSI.size()) {
    MinCSFI = CSI[0].getFrameIdx();
    MaxCSFI = CSI[CSI.size() - 1].getFrameIdx();
  }
  // The following stack frame objects are always referenced relative to $sp:
  //  1. Outgoing arguments.
  //  2. Pointer to dynamically allocated stack space.
```
- **EN**: Implements logic around `getFrameIdx`, `size`; this block applies conditional target rules.
- **CN**: 围绕 `getFrameIdx`, `size` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 85-94
```cpp
  //  3. Locations for callee-saved registers.
  //  4. Locations for eh data registers.
  // Everything else is referenced relative to whatever register
  // getFrameRegister() returns.
  MCRegister FrameReg;
  if ((FrameIndex >= MinCSFI && FrameIndex <= MaxCSFI))
    FrameReg = Xtensa::SP;
  else
    FrameReg = getFrameRegister(MF);

```
- **EN**: Implements logic around `getFrameRegister`; this block applies conditional target rules.
- **CN**: 围绕 `getFrameRegister` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 95-104
```cpp
  // Calculate final offset.
  // - There is no need to change the offset if the frame object is one of the
  //   following: an outgoing argument, pointer to a dynamically allocated
  //   stack space or a $gp restore location,
  // - If the frame object is any of the following, its offset must be adjusted
  //   by adding the size of the stack:
  //   incoming argument, callee-saved register location or local variable.
  bool IsKill = false;
  int64_t Offset =
      SPOffset + (int64_t)StackSize + MI.getOperand(FIOperandNum + 1).getImm();
```
- **EN**: Implements logic around `getOperand`.
- **CN**: 围绕 `getOperand` 实现具体逻辑。

### Lines 105-114
```cpp

  bool Valid = Xtensa::isValidAddrOffsetForOpcode(MI.getOpcode(), Offset);

  // If MI is not a debug value, make sure Offset fits in the 16-bit immediate
  // field.
  if (!MI.isDebugValue() && !Valid) {
    MachineBasicBlock &MBB = *MI.getParent();
    DebugLoc DL = II->getDebugLoc();
    unsigned ADD = Xtensa::ADD;
    MCRegister Reg;
```
- **EN**: Implements logic around `isValidAddrOffsetForOpcode`, `getParent`, `getDebugLoc`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isValidAddrOffsetForOpcode`, `getParent`, `getDebugLoc` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 115-122
```cpp
    const XtensaInstrInfo &TII = *static_cast<const XtensaInstrInfo *>(
        MBB.getParent()->getSubtarget().getInstrInfo());

    TII.loadImmediate(MBB, II, &Reg, Offset);
    BuildMI(MBB, II, DL, TII.get(ADD), Reg)
        .addReg(FrameReg)
        .addReg(Reg, RegState::Kill);

```
- **EN**: Implements logic around `getParent`, `loadImmediate`, `BuildMI`, `addReg`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getParent`, `loadImmediate`, `BuildMI`, `addReg` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 123-127
```cpp
    FrameReg = Reg;
    Offset = 0;
    IsKill = true;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 128-133
```cpp
  MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, false, false, IsKill);
  MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);

  return false;
}

```
- **EN**: Implements logic around `getOperand`; this block returns target-specific results.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 134-138
```cpp
Register XtensaRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
  return TFI->hasFP(MF) ? (Subtarget.isWindowedABI() ? Xtensa::A7 : Xtensa::A15)
                        : Xtensa::SP;
}
```
- **EN**: Implements logic around `getFrameRegister`, `getSubtarget`, `hasFP`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameRegister`, `getSubtarget`, `hasFP` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Register model / 寄存器模型**:
  - **EN**: Describes registers, classes, and allocation-facing details
  - **CN**: 描述寄存器、寄存器类以及面向分配器的细节
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaRegisterInfo.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `XtensaInstrInfo.h`, `XtensaSubtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `XtensaGenRegisterInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
- **Generated macros / 生成宏**: `GET_REGINFO_TARGET_DESC`
