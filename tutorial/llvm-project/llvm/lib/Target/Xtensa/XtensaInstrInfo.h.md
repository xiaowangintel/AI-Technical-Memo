# XtensaInstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaInstrInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
//===-- XtensaInstrInfo.h - Xtensa Instruction Information ------*- C++ -*-===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-14
```cpp
//
// This file contains the Xtensa implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 15-22
```cpp
#ifndef LLVM_LIB_TARGET_XTENSA_XTENSAINSTRINFO_H
#define LLVM_LIB_TARGET_XTENSA_XTENSAINSTRINFO_H

#include "Xtensa.h"
#include "XtensaRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `Xtensa.h`, `XtensaRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `Xtensa.h`, `XtensaRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`。

### Lines 23-28
```cpp
#define GET_INSTRINFO_HEADER

#include "XtensaGenInstrInfo.inc"

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenInstrInfo.inc`。

### Lines 29-34
```cpp
class XtensaTargetMachine;
class XtensaSubtarget;
class XtensaInstrInfo : public XtensaGenInstrInfo {
  const XtensaRegisterInfo RI;
  const XtensaSubtarget &STI;

```
- **EN**: Introduces declarations for `XtensaTargetMachine`, `XtensaSubtarget`, `XtensaInstrInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaTargetMachine`, `XtensaSubtarget`, `XtensaInstrInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 35-40
```cpp
public:
  XtensaInstrInfo(const XtensaSubtarget &STI);

  void adjustStackPtr(MCRegister SP, int64_t Amount, MachineBasicBlock &MBB,
                      MachineBasicBlock::iterator I) const;

```
- **EN**: Implements logic around `XtensaInstrInfo`, `adjustStackPtr`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `XtensaInstrInfo`, `adjustStackPtr` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 41-45
```cpp
  unsigned getInstSizeInBytes(const MachineInstr &MI) const override;

  // Return the XtensaRegisterInfo, which this class owns.
  const XtensaRegisterInfo &getRegisterInfo() const { return RI; }

```
- **EN**: Introduces declarations for `owns`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `owns` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 46-51
```cpp
  Register isLoadFromStackSlot(const MachineInstr &MI,
                               int &FrameIndex) const override;

  Register isStoreToStackSlot(const MachineInstr &MI,
                              int &FrameIndex) const override;

```
- **EN**: Implements logic around `isLoadFromStackSlot`, `isStoreToStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLoadFromStackSlot`, `isStoreToStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 52-56
```cpp
  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                   const DebugLoc &DL, Register DestReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;

```
- **EN**: Implements logic around `copyPhysReg`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `copyPhysReg` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 57-61
```cpp
  void storeRegToStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
      bool isKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

```
- **EN**: Implements logic around `storeRegToStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `storeRegToStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 62-67
```cpp
  void loadRegFromStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
      Register DestReg, int FrameIdx, const TargetRegisterClass *RC,
      Register VReg, unsigned SubReg = 0,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

```
- **EN**: Implements logic around `loadRegFromStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 68-76
```cpp
  // Get the load and store opcodes for a given register class and offset.
  void getLoadStoreOpcodes(const TargetRegisterClass *RC, unsigned &LoadOpcode,
                           unsigned &StoreOpcode, int64_t offset) const;

  // Emit code before MBBI in MI to move immediate value Value into
  // physical register Reg.
  void loadImmediate(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                     MCRegister *Reg, int64_t Value) const;

```
- **EN**: Introduces declarations for `and`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `and` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 77-81
```cpp
  bool
  reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;

  MachineBasicBlock *getBranchDestBlock(const MachineInstr &MI) const override;

```
- **EN**: Implements logic around `reverseBranchCondition`, `getBranchDestBlock`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `reverseBranchCondition`, `getBranchDestBlock` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 82-89
```cpp
  bool isBranchOffsetInRange(unsigned BranchOpc,
                             int64_t BrOffset) const override;

  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify) const override;

```
- **EN**: Implements logic around `isBranchOffsetInRange`, `analyzeBranch`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isBranchOffsetInRange`, `analyzeBranch` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 90-97
```cpp
  unsigned removeBranch(MachineBasicBlock &MBB,
                        int *BytesRemoved = nullptr) const override;

  unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
                        MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
                        const DebugLoc &DL,
                        int *BytesAdded = nullptr) const override;

```
- **EN**: Implements logic around `removeBranch`, `insertBranch`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `removeBranch`, `insertBranch` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 98-102
```cpp
  void insertIndirectBranch(MachineBasicBlock &MBB, MachineBasicBlock &DestBB,
                            MachineBasicBlock &RestoreBB, const DebugLoc &DL,
                            int64_t BrOffset = 0,
                            RegScavenger *RS = nullptr) const override;

```
- **EN**: Implements logic around `insertIndirectBranch`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertIndirectBranch` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 103-108
```cpp
  unsigned insertBranchAtInst(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator I,
                              MachineBasicBlock *TBB,
                              ArrayRef<MachineOperand> Cond, const DebugLoc &DL,
                              int *BytesAdded) const;

```
- **EN**: Implements logic around `insertBranchAtInst`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertBranchAtInst` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 109-113
```cpp
  unsigned insertConstBranchAtInst(MachineBasicBlock &MBB, MachineInstr *I,
                                   int64_t offset,
                                   ArrayRef<MachineOperand> Cond, DebugLoc DL,
                                   int *BytesAdded) const;

```
- **EN**: Implements logic around `insertConstBranchAtInst`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insertConstBranchAtInst` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 114-122
```cpp
  // Return true if MI is a conditional or unconditional branch.
  // When returning true, set Cond to the mask of condition-code
  // values on which the instruction will branch, and set Target
  // to the operand that contains the branch target.  This target
  // can be a register or a basic block.
  bool isBranch(const MachineBasicBlock::iterator &MI,
                SmallVectorImpl<MachineOperand> &Cond,
                const MachineOperand *&Target) const;

```
- **EN**: Implements logic around `isBranch`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isBranch` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 123-127
```cpp
  const XtensaSubtarget &getSubtarget() const { return STI; }
};
} // end namespace llvm

#endif /* LLVM_LIB_TARGET_XTENSA_XTENSAINSTRINFO_H */
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `Xtensa.h`, `XtensaRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `XtensaGenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_INSTRINFO_HEADER`, `GET_XTENSA_XTENSAINSTRINFO_H`
