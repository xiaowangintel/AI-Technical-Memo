# VEFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEFrameLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements stack frame layout plus prologue/epilogue emission for the backend.
  - **CN**: 实现该后端的栈帧布局以及序言/尾声生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEFrameLowering.h - Define frame lowering for VE --*- C++ -*-===//
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
// This class implements VE-specific bits of TargetFrameLowering class.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-19
```cpp
#ifndef LLVM_LIB_TARGET_VE_VEFRAMELOWERING_H
#define LLVM_LIB_TARGET_VE_VEFRAMELOWERING_H

#include "VE.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/Support/TypeSize.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `VE.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VE.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`。

### Lines 20-26
```cpp
namespace llvm {

class VESubtarget;
class VEFrameLowering : public TargetFrameLowering {
public:
  explicit VEFrameLowering(const VESubtarget &ST);

```
- **EN**: Introduces declarations for `llvm`, `VESubtarget`, `VEFrameLowering`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `VESubtarget`, `VEFrameLowering` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 27-36
```cpp
  /// emitProlog/emitEpilog - These methods insert prolog and epilog code into
  /// the function.
  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitPrologueInsns(MachineFunction &MF, MachineBasicBlock &MBB,
                         MachineBasicBlock::iterator MBBI, uint64_t NumBytes,
                         bool RequireFPUpdate) const;
  void emitEpilogueInsns(MachineFunction &MF, MachineBasicBlock &MBB,
                         MachineBasicBlock::iterator MBBI, uint64_t NumBytes,
                         bool RequireFPUpdate) const;
```
- **EN**: Implements logic around `emitPrologue`, `emitEpilogue`, `emitPrologueInsns`, `emitEpilogueInsns`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitPrologue`, `emitEpilogue`, `emitPrologueInsns`, `emitEpilogueInsns` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 37-41
```cpp

  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator I) const override;

```
- **EN**: Implements logic around `eliminateCallFramePseudoInstr`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateCallFramePseudoInstr` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 42-51
```cpp
  bool hasBP(const MachineFunction &MF) const;
  bool hasGOT(const MachineFunction &MF) const;

  // VE reserves argument space always for call sites in the function
  // immediately on entry of the current function.
  bool hasReservedCallFrame(const MachineFunction &MF) const override {
    return true;
  }
  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS = nullptr) const override;
```
- **EN**: Implements logic around `hasBP`, `hasGOT`, `hasReservedCallFrame`, `determineCalleeSaves`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasBP`, `hasGOT`, `hasReservedCallFrame`, `determineCalleeSaves` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 52-61
```cpp

  StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
                                     Register &FrameReg) const override;

  const SpillSlot *
  getCalleeSavedSpillSlots(unsigned &NumEntries) const override {
    static const SpillSlot Offsets[] = {
        {VE::SX17, 40},  {VE::SX18, 48},  {VE::SX19, 56},  {VE::SX20, 64},
        {VE::SX21, 72},  {VE::SX22, 80},  {VE::SX23, 88},  {VE::SX24, 96},
        {VE::SX25, 104}, {VE::SX26, 112}, {VE::SX27, 120}, {VE::SX28, 128},
```
- **EN**: Implements logic around `getFrameIndexReference`, `getCalleeSavedSpillSlots`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameIndexReference`, `getCalleeSavedSpillSlots` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 62-67
```cpp
        {VE::SX29, 136}, {VE::SX30, 144}, {VE::SX31, 152}, {VE::SX32, 160},
        {VE::SX33, 168}};
    NumEntries = std::size(Offsets);
    return Offsets;
  }

```
- **EN**: Implements logic around `size`; this block returns target-specific results.
- **CN**: 围绕 `size` 实现具体逻辑；这一段返回目标相关结果。

### Lines 68-72
```cpp
protected:
  const VESubtarget &STI;

  bool hasFPImpl(const MachineFunction &MF) const override;

```
- **EN**: Implements logic around `hasFPImpl`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasFPImpl` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 73-81
```cpp
private:
  // Returns true if MF is a leaf procedure.
  bool isLeafProc(MachineFunction &MF) const;

  // Emits code for adjusting SP in function prologue/epilogue.
  void emitSPAdjustment(MachineFunction &MF, MachineBasicBlock &MBB,
                        MachineBasicBlock::iterator MBBI, int64_t NumBytes,
                        MaybeAlign MayAlign = MaybeAlign()) const;

```
- **EN**: Implements logic around `isLeafProc`, `emitSPAdjustment`, `MaybeAlign`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLeafProc`, `emitSPAdjustment`, `MaybeAlign` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 82-86
```cpp
  // Emits code for extending SP in function prologue/epilogue.
  void emitSPExtend(MachineFunction &MF, MachineBasicBlock &MBB,
                    MachineBasicBlock::iterator MBBI) const;
};

```
- **EN**: Implements logic around `emitSPExtend`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitSPExtend` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 87-89
```cpp
} // namespace llvm

#endif
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Stack frame management / 栈帧管理**:
  - **EN**: Controls prologue/epilogue emission and frame layout
  - **CN**: 控制序言尾声生成与栈帧布局
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VE.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
- **Generated macros / 生成宏**: `GET_VE_VEFRAMELOWERING_H`
