# LoongArchFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchFrameLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements stack frame lowering and prologue/epilogue logic for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责实现栈帧降低以及序言/尾声逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //=- LoongArchFrameLowering.h - TargetFrameLowering for LoongArch -*- C++ -*--//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This class implements LoongArch-specific bits of TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #ifndef LLVM_LIB_TARGET_LOONGARCH_LOONGARCHFRAMELOWERING_H
  14: #define LLVM_LIB_TARGET_LOONGARCH_LOONGARCHFRAMELOWERING_H
  15: 
  16: #include "llvm/CodeGen/TargetFrameLowering.h"
  17: 
  18: namespace llvm {
  19: class LoongArchSubtarget;
  20: 
  21: class LoongArchFrameLowering : public TargetFrameLowering {
  22:   const LoongArchSubtarget &STI;
  23: 
  24: public:
```
- **EN**: It imports dependencies such as `TargetFrameLowering.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchSubtarget`, `LoongArchFrameLowering`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `TargetFrameLowering.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchSubtarget`, `LoongArchFrameLowering` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 25-36 / 第 25-36 行
```cpp
  25:   explicit LoongArchFrameLowering(const LoongArchSubtarget &STI)
  26:       : TargetFrameLowering(StackGrowsDown,
  27:                             /*StackAlignment=*/Align(16),
  28:                             /*LocalAreaOffset=*/0),
  29:         STI(STI) {}
  30: 
  31:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  32:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  33: 
  34:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
  35:                             RegScavenger *RS) const override;
  36: 
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering`.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37:   void processFunctionBeforeFrameFinalized(MachineFunction &MF,
  38:                                            RegScavenger *RS) const override;
  39: 
  40:   bool hasReservedCallFrame(const MachineFunction &MF) const override;
  41:   MachineBasicBlock::iterator
  42:   eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
  43:                                 MachineBasicBlock::iterator MI) const override;
  44:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
  45:                                  MachineBasicBlock::iterator MI,
  46:                                  ArrayRef<CalleeSavedInfo> CSI,
  47:                                  const TargetRegisterInfo *TRI) const override;
  48: 
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 49-60 / 第 49-60 行
```cpp
  49:   StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
  50:                                      Register &FrameReg) const override;
  51: 
  52:   bool hasBP(const MachineFunction &MF) const;
  53: 
  54:   uint64_t getFirstSPAdjustAmount(const MachineFunction &MF) const;
  55: 
  56:   bool enableShrinkWrapping(const MachineFunction &MF) const override;
  57: 
  58: protected:
  59:   bool hasFPImpl(const MachineFunction &MF) const override;
  60: 
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 61-68 / 第 61-68 行
```cpp
  61: private:
  62:   void determineFrameLayout(MachineFunction &MF) const;
  63:   void adjustReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  64:                  const DebugLoc &DL, Register DestReg, Register SrcReg,
  65:                  int64_t Val, MachineInstr::MIFlag Flag) const;
  66: };
  67: } // end namespace llvm
  68: #endif // LLVM_LIB_TARGET_LOONGARCH_LOONGARCHFRAMELOWERING_H
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `llvm/CodeGen/TargetFrameLowering.h`
