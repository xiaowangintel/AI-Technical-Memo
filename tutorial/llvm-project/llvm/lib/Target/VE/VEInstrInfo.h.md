# VEInstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEInstrInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEInstrInfo.h - VE Instruction Information --------------*- C++ -*-===//
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
// This file contains the VE implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-18
```cpp
#ifndef LLVM_LIB_TARGET_VE_VEINSTRINFO_H
#define LLVM_LIB_TARGET_VE_VEINSTRINFO_H

#include "VERegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `VERegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VERegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`。

### Lines 19-23
```cpp
#define GET_INSTRINFO_HEADER
#include "VEGenInstrInfo.inc"

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenInstrInfo.inc`。

### Lines 24-28
```cpp
class VESubtarget;

/// VEII - This namespace holds all of the Aurora VE target-specific
/// per-instruction flags.  These must match the corresponding definitions in
/// VEInstrFormats.td.
```
- **EN**: Introduces declarations for `VESubtarget`, `holds`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VESubtarget`, `holds` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 29-33
```cpp
namespace VEII {
enum {
  // Aurora VE Instruction Flags.  These flags describe the characteristics of
  // the Aurora VE instructions for vector handling.

```
- **EN**: Introduces declarations for `VEII`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEII` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-39
```cpp
  /// VE_Vector - This instruction is Vector Instruction.
  VE_Vector = 0x1,

  /// VE_VLInUse - This instruction has a vector register in its operands.
  VE_VLInUse = 0x2,

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 40-45
```cpp
  /// VE_VLMask/Shift - This is a bitmask that selects the index number where
  /// an instruction holds vector length informatio (0 to 6, 7 means undef).n
  VE_VLShift = 2,
  VE_VLMask = 0x07 << VE_VLShift,
};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 46-50
```cpp
#define HAS_VLINDEX(TSF) ((TSF)&VEII::VE_VLInUse)
#define GET_VLINDEX(TSF)                                                       \
  (HAS_VLINDEX(TSF) ? (int)(((TSF)&VEII::VE_VLMask) >> VEII::VE_VLShift) : -1)
} // end namespace VEII

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 51-57
```cpp
class VEInstrInfo : public VEGenInstrInfo {
  const VERegisterInfo RI;
  virtual void anchor();

public:
  explicit VEInstrInfo(const VESubtarget &ST);

```
- **EN**: Introduces declarations for `VEInstrInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEInstrInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 58-63
```cpp
  /// getRegisterInfo - TargetInstrInfo is a superset of MRegister info.  As
  /// such, whenever a client has an instance of instruction info, it should
  /// always be able to get register info as well (through this method).
  ///
  const VERegisterInfo &getRegisterInfo() const { return RI; }

```
- **EN**: Implements logic around `getRegisterInfo`; this block returns target-specific results.
- **CN**: 围绕 `getRegisterInfo` 实现具体逻辑；这一段返回目标相关结果。

### Lines 64-69
```cpp
  /// Branch Analysis & Modification {
  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify = false) const override;

```
- **EN**: Implements logic around `analyzeBranch`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `analyzeBranch` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 70-77
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

### Lines 78-86
```cpp
  bool
  reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;
  /// } Branch Analysis & Modification

  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
                   const DebugLoc &DL, Register DestReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;

```
- **EN**: Implements logic around `reverseBranchCondition`, `copyPhysReg`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `reverseBranchCondition`, `copyPhysReg` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 87-95
```cpp
  /// Stack Spill & Reload {
  Register isLoadFromStackSlot(const MachineInstr &MI,
                               int &FrameIndex) const override;
  Register isStoreToStackSlot(const MachineInstr &MI,
                              int &FrameIndex) const override;
  void storeRegToStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
      bool isKill, int FrameIndex, const TargetRegisterClass *RC,

```
- **EN**: Implements logic around `isLoadFromStackSlot`, `isStoreToStackSlot`, `storeRegToStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLoadFromStackSlot`, `isStoreToStackSlot`, `storeRegToStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 96-104
```cpp
      Register VReg,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

  void loadRegFromStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
      Register DestReg, int FrameIndex, const TargetRegisterClass *RC,
      Register VReg, unsigned SubReg = 0,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  /// } Stack Spill & Reload
```
- **EN**: Implements logic around `loadRegFromStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 105-110
```cpp

  /// Optimization {

  bool foldImmediate(MachineInstr &UseMI, MachineInstr &DefMI, Register Reg,
                     MachineRegisterInfo *MRI) const override;

```
- **EN**: Implements logic around `foldImmediate`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `foldImmediate` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 111-117
```cpp
  /// } Optimization

  Register getGlobalBaseReg(MachineFunction *MF) const;

  // Lower pseudo instructions after register allocation.
  bool expandPostRAPseudo(MachineInstr &MI) const override;

```
- **EN**: Implements logic around `getGlobalBaseReg`, `expandPostRAPseudo`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getGlobalBaseReg`, `expandPostRAPseudo` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 118-123
```cpp
  bool expandExtendStackPseudo(MachineInstr &MI) const;
  bool expandGetStackTopPseudo(MachineInstr &MI) const;
};

} // namespace llvm

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 124-124
```cpp
#endif
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VERegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `VEGenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_INSTRINFO_HEADER`, `GET_VE_VEINSTRINFO_H`, `GET_VLINDEX`
