# CSKYFrameLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYFrameLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements stack frame layout, prologue/epilogue emission, and related calling-sequence details.
- 目的（中文）: 实现栈帧布局、函数序言/尾声生成以及相关调用序列细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYFrameLowering.h - Define frame lowering for CSKY -*- C++ -*--===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This class implements CSKY-specific bits of TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_CSKY_CSKYFRAMELOWERING_H
  14: #define LLVM_LIB_TARGET_CSKY_CSKYFRAMELOWERING_H
  15: 
  16: #include "llvm/CodeGen/TargetFrameLowering.h"
  17: 
  18: namespace llvm {
  19: class CSKYSubtarget;
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as implements, CSKYSubtarget, which organize the target-specific behavior exposed by the file. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 implements, CSKYSubtarget 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 21-40

```cpp
  21: class CSKYFrameLowering : public TargetFrameLowering {
  22:   const CSKYSubtarget &STI;
  23: 
  24:   void determineFrameLayout(MachineFunction &MF) const;
  25:   void adjustReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  26:                  const DebugLoc &DL, Register DestReg, Register SrcReg,
  27:                  int64_t Val, MachineInstr::MIFlag Flag) const;
  28: 
  29: public:
  30:   explicit CSKYFrameLowering(const CSKYSubtarget &STI)
  31:       : TargetFrameLowering(StackGrowsDown,
  32:                             /*StackAlignment=*/Align(4),
  33:                             /*LocalAreaOffset=*/0),
  34:         STI(STI) {}
  35: 
  36:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  37:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  38: 
  39:   StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
  40:                                      Register &FrameReg) const override;
```

- EN: This chunk introduces interfaces or data structures such as CSKYFrameLowering, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CSKYFrameLowering contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一段引入了 CSKYFrameLowering 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CSKYFrameLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 41-60

```cpp
  41: 
  42:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
  43:                             RegScavenger *RS) const override;
  44: 
  45:   bool assignCalleeSavedSpillSlots(
  46:       MachineFunction &MF, const TargetRegisterInfo *TRI,
  47:       std::vector<CalleeSavedInfo> &CSI) const override {
  48: 
  49:     std::reverse(CSI.begin(), CSI.end());
  50: 
  51:     return false;
  52:   }
  53: 
  54:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
  55:                                  MachineBasicBlock::iterator MI,
  56:                                  ArrayRef<CalleeSavedInfo> CSI,
  57:                                  const TargetRegisterInfo *TRI) const override;
  58:   bool
  59:   restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
  60:                               MachineBasicBlock::iterator MI,
```

- EN: At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 61-76

```cpp
  61:                               MutableArrayRef<CalleeSavedInfo> CSI,
  62:                               const TargetRegisterInfo *TRI) const override;
  63: 
  64:   bool hasBP(const MachineFunction &MF) const;
  65: 
  66:   bool hasReservedCallFrame(const MachineFunction &MF) const override;
  67: 
  68:   MachineBasicBlock::iterator
  69:   eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
  70:                                 MachineBasicBlock::iterator MI) const override;
  71: 
  72: protected:
  73:   bool hasFPImpl(const MachineFunction &MF) const override;
  74: };
  75: } // namespace llvm
  76: #endif
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

## Key Concepts / 关键概念

- Prologue and epilogue emission / 序言与尾声生成
- Stack object layout / 栈对象布局
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/CodeGen/TargetFrameLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `CSKYFrameLowering.cpp`
