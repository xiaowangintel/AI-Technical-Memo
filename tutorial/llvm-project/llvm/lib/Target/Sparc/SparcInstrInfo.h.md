# SparcInstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstrInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific instruction information, instruction helpers, and scheduling-related behavior.
  - **CN**: 实现目标相关的指令信息、指令辅助逻辑以及与调度相关的行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcInstrInfo.h - Sparc Instruction Information --------*- C++ -*-===//
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
// This file contains the Sparc implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-18
```cpp
#ifndef LLVM_LIB_TARGET_SPARC_SPARCINSTRINFO_H
#define LLVM_LIB_TARGET_SPARC_SPARCINSTRINFO_H

#include "SparcRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`。

### Lines 19-23
```cpp
#define GET_INSTRINFO_HEADER
#include "SparcGenInstrInfo.inc"

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenInstrInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenInstrInfo.inc`。

### Lines 24-28
```cpp
class SparcSubtarget;

/// SPII - This namespace holds all of the target specific flags that
/// instruction info tracks.
///
```
- **EN**: Introduces declarations for `SparcSubtarget`, `holds`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SparcSubtarget`, `holds` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 29-37
```cpp
namespace SPII {
  enum {
    Pseudo = (1<<0),
    Load = (1<<1),
    Store = (1<<2),
    DelaySlot = (1<<3)
  };
}

```
- **EN**: Introduces declarations for `SPII`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SPII` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 38-44
```cpp
class SparcInstrInfo : public SparcGenInstrInfo {
  const SparcRegisterInfo RI;
  const SparcSubtarget& Subtarget;
  virtual void anchor();
public:
  explicit SparcInstrInfo(const SparcSubtarget &ST);

```
- **EN**: Introduces declarations for `SparcInstrInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SparcInstrInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-50
```cpp
  /// getRegisterInfo - TargetInstrInfo is a superset of MRegister info.  As
  /// such, whenever a client has an instance of instruction info, it should
  /// always be able to get register info as well (through this method).
  ///
  const SparcRegisterInfo &getRegisterInfo() const { return RI; }

```
- **EN**: Implements logic around `getRegisterInfo`; this block returns target-specific results.
- **CN**: 围绕 `getRegisterInfo` 实现具体逻辑；这一段返回目标相关结果。

### Lines 51-55
```cpp
  /// isLoadFromStackSlot - If the specified machine instruction is a direct
  /// load from a stack slot, return the virtual or physical register number of
  /// the destination along with the FrameIndex of the loaded stack slot.  If
  /// not, return 0.  This predicate must return 0 if the instruction has
  /// any side effects other than loading from the stack slot.
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 56-60
```cpp
  Register isLoadFromStackSlot(const MachineInstr &MI, int &FrameIndex,
                               TypeSize &MemBytes) const override;

  /// isStoreToStackSlot - If the specified machine instruction is a direct
  /// store to a stack slot, return the virtual or physical register number of
```
- **EN**: Implements logic around `isLoadFromStackSlot`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLoadFromStackSlot` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 61-66
```cpp
  /// the source reg along with the FrameIndex of the loaded stack slot.  If
  /// not, return 0.  This predicate must return 0 if the instruction has
  /// any side effects other than storing to the stack slot.
  Register isStoreToStackSlot(const MachineInstr &MI, int &FrameIndex,
                              TypeSize &MemBytes) const override;

```
- **EN**: Implements logic around `isStoreToStackSlot`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isStoreToStackSlot` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 67-73
```cpp
  MachineBasicBlock *getBranchDestBlock(const MachineInstr &MI) const override;

  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify = false) const override;

```
- **EN**: Implements logic around `getBranchDestBlock`, `analyzeBranch`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getBranchDestBlock`, `analyzeBranch` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 74-81
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

### Lines 82-87
```cpp
  bool
  reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;

  /// Determine if the branch target is in range.
  bool isBranchOffsetInRange(unsigned BranchOpc, int64_t Offset) const override;

```
- **EN**: Implements logic around `reverseBranchCondition`, `isBranchOffsetInRange`.
- **CN**: 围绕 `reverseBranchCondition`, `isBranchOffsetInRange` 实现具体逻辑。

### Lines 88-92
```cpp
  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
                   const DebugLoc &DL, Register DestReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;

```
- **EN**: Implements logic around `copyPhysReg`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `copyPhysReg` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 93-97
```cpp
  void storeRegToStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
      bool isKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

```
- **EN**: Implements logic around `storeRegToStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `storeRegToStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 98-103
```cpp
  void loadRegFromStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
      Register DestReg, int FrameIndex, const TargetRegisterClass *RC,
      Register VReg, unsigned SubReg = 0,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

```
- **EN**: Implements logic around `loadRegFromStackSlot`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `loadRegFromStackSlot` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 104-109
```cpp
  Register getGlobalBaseReg(MachineFunction *MF) const;

  /// GetInstSize - Return the number of bytes of code the specified
  /// instruction may be.  This returns the maximum number of bytes.
  unsigned getInstSizeInBytes(const MachineInstr &MI) const override;

```
- **EN**: Implements logic around `getGlobalBaseReg`, `getInstSizeInBytes`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getGlobalBaseReg`, `getInstSizeInBytes` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 110-117
```cpp
  bool analyzeCompare(const MachineInstr &MI, Register &SrcReg,
                      Register &SrcReg2, int64_t &CmpMask,
                      int64_t &CmpValue) const override;

  bool optimizeCompareInstr(MachineInstr &CmpInstr, Register SrcReg,
                            Register SrcReg2, int64_t CmpMask, int64_t CmpValue,
                            const MachineRegisterInfo *MRI) const override;

```
- **EN**: Implements logic around `analyzeCompare`, `optimizeCompareInstr`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `analyzeCompare`, `optimizeCompareInstr` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 118-123
```cpp
  // Lower pseudo instructions after register allocation.
  bool expandPostRAPseudo(MachineInstr &MI) const override;
};

}

```
- **EN**: Implements logic around `expandPostRAPseudo`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `expandPostRAPseudo` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

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

- **Direct includes / 直接包含**: `SparcRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `SparcGenInstrInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_INSTRINFO_HEADER`, `GET_SPARC_SPARCINSTRINFO_H`
