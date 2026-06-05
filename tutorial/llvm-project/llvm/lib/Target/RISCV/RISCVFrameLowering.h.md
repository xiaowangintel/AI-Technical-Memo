# RISCVFrameLowering.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVFrameLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for stack frame layout, prologue/epilogue emission, and callee-save handling for RISC-V. / 声明RISC-V 的栈帧布局、序言/尾声生成与被调者保存寄存器处理所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVFrameLowering.h - Define frame lowering for RISC-V -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class implements RISC-V specific bits of TargetFrameLowering class.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-21: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#ifndef LLVM_LIB_TARGET_RISCV_RISCVFRAMELOWERING_H
#define LLVM_LIB_TARGET_RISCV_RISCVFRAMELOWERING_H

#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/Support/TypeSize.h"

namespace llvm {
class RISCVSubtarget;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 22-31: Type declaration for RISCVFrameLowering / RISCVFrameLowering 的类型声明
```cpp
class RISCVFrameLowering : public TargetFrameLowering {
public:
  explicit RISCVFrameLowering(const RISCVSubtarget &STI);

  int getInitialCFAOffset(const MachineFunction &MF) const override;
  Register getInitialCFARegister(const MachineFunction &MF) const override;

  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 32-39: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  uint64_t getStackSizeWithRVVPadding(const MachineFunction &MF) const;

  StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
                                     Register &FrameReg) const override;

  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 40-49: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void processFunctionBeforeFrameFinalized(MachineFunction &MF,
                                           RegScavenger *RS) const override;

  bool hasBP(const MachineFunction &MF) const;

  bool hasReservedCallFrame(const MachineFunction &MF) const override;
  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MI) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 50-63: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool
  assignCalleeSavedSpillSlots(MachineFunction &MF,
                              const TargetRegisterInfo *TRI,
                              std::vector<CalleeSavedInfo> &CSI) const override;
  bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MI,
                                 ArrayRef<CalleeSavedInfo> CSI,
                                 const TargetRegisterInfo *TRI) const override;
  bool
  restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator MI,
                              MutableArrayRef<CalleeSavedInfo> CSI,
                              const TargetRegisterInfo *TRI) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 64-71: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Get the first stack adjustment amount for SplitSPAdjust.
  // Return 0 if we don't want to split the SP adjustment in prologue and
  // epilogue.
  uint64_t getFirstSPAdjustAmount(const MachineFunction &MF) const;

  bool canUseAsPrologue(const MachineBasicBlock &MBB) const override;
  bool canUseAsEpilogue(const MachineBasicBlock &MBB) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 72-81: Function implementation: isStackIdSafeForLocalArea / 函数实现：isStackIdSafeForLocalArea
```cpp
  bool enableShrinkWrapping(const MachineFunction &MF) const override;

  bool isSupportedStackID(TargetStackID::Value ID) const override;
  TargetStackID::Value getStackIDForScalableVectors() const override;

  bool isStackIdSafeForLocalArea(unsigned StackId) const override {
    // We don't support putting RISC-V Vector objects into the pre-allocated
    // local frame block at the moment.
    return StackId != TargetStackID::ScalableVector;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 82-91: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  void allocateStack(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                     MachineFunction &MF, uint64_t Offset,
                     uint64_t RealStackSize, bool EmitCFI, bool NeedProbe,
                     uint64_t ProbeSize, bool DynAllocation,
                     MachineInstr::MIFlag Flag) const;

protected:
  const RISCVSubtarget &STI;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 92-107: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool hasFPImpl(const MachineFunction &MF) const override;

private:
  void determineFrameLayout(MachineFunction &MF) const;
  void emitCalleeSavedRVVPrologCFI(MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator MI,
                                   bool HasFP) const;
  void emitCalleeSavedRVVEpilogCFI(MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator MI) const;
  template <typename Emitter>
  void emitCFIForCSI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                     const SmallVector<CalleeSavedInfo, 8> &CSI) const;
  void deallocateStack(MachineFunction &MF, MachineBasicBlock &MBB,
                       MachineBasicBlock::iterator MBBI, const DebugLoc &DL,
                       uint64_t &StackSize, int64_t CFAOffset) const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 108-118: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  std::pair<int64_t, Align>
  assignRVVStackObjectOffsets(MachineFunction &MF) const;
  // Replace a StackProbe stub (if any) with the actual probe code inline
  void inlineStackProbe(MachineFunction &MF,
                        MachineBasicBlock &PrologueMBB) const override;
  void allocateAndProbeStackForRVV(MachineFunction &MF, MachineBasicBlock &MBB,
                                   MachineBasicBlock::iterator MBBI,
                                   const DebugLoc &DL, int64_t Amount,
                                   MachineInstr::MIFlag Flag, bool EmitCFI,
                                   bool DynAllocation) const;
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 119-120: Header guard and interface framing / 头文件保护与接口框架
```cpp
} // namespace llvm
#endif
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **Stack frame lowering** / **栈帧下降**

## Dependencies / 依赖关系
- `llvm/CodeGen/TargetFrameLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/TypeSize.h` — Directly referenced by this file. / 该文件直接引用的依赖。
