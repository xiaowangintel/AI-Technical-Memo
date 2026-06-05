# ARCFrameLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCFrameLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements stack frame layout, prologue/epilogue emission, and related calling-sequence details.
- 目的（中文）: 实现栈帧布局、函数序言/尾声生成以及相关调用序列细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCFrameLowering.h - Define frame lowering for ARC -------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This class implements the ARC specific frame lowering.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_ARC_ARCFRAMELOWERING_H
  14: #define LLVM_LIB_TARGET_ARC_ARCFRAMELOWERING_H
  15: 
  16: #include "ARC.h"
  17: #include "llvm/CodeGen/MachineBasicBlock.h"
  18: #include "llvm/CodeGen/MachineFrameInfo.h"
  19: #include "llvm/CodeGen/TargetFrameLowering.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. This chunk introduces interfaces or data structures such as implements, which organize the target-specific behavior exposed by the file. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 这一段引入了 implements 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 21-40

```cpp
  21: namespace llvm {
  22: 
  23: class MachineFunction;
  24: class ARCSubtarget;
  25: class ARCInstrInfo;
  26: 
  27: class ARCFrameLowering : public TargetFrameLowering {
  28: public:
  29:   ARCFrameLowering(const ARCSubtarget &st)
  30:       : TargetFrameLowering(TargetFrameLowering::StackGrowsDown, Align(4), 0),
  31:         ST(st) {}
  32: 
  33:   /// Insert Prologue into the function.
  34:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  35: 
  36:   /// Insert Epilogue into the function.
  37:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  38: 
  39:   /// Add explicit callee save registers.
  40:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as MachineFunction, ARCSubtarget, ARCInstrInfo, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as ARCFrameLowering contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 MachineFunction, ARCSubtarget, ARCInstrInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 ARCFrameLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 41-60

```cpp
  41:                             RegScavenger *RS) const override;
  42: 
  43:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
  44:                                  MachineBasicBlock::iterator MI,
  45:                                  ArrayRef<CalleeSavedInfo> CSI,
  46:                                  const TargetRegisterInfo *TRI) const override;
  47: 
  48:   bool
  49:   restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
  50:                               MachineBasicBlock::iterator MI,
  51:                               MutableArrayRef<CalleeSavedInfo> CSI,
  52:                               const TargetRegisterInfo *TRI) const override;
  53: 
  54:   void processFunctionBeforeFrameFinalized(MachineFunction &MF,
  55:                                            RegScavenger *RS) const override;
  56: 
  57:   MachineBasicBlock::iterator
  58:   eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
  59:                                 MachineBasicBlock::iterator I) const override;
  60: 
```

- EN: At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 61-78

```cpp
  61:   bool assignCalleeSavedSpillSlots(
  62:       llvm::MachineFunction &, const llvm::TargetRegisterInfo *,
  63:       std::vector<llvm::CalleeSavedInfo> &) const override;
  64: 
  65: protected:
  66:   bool hasFPImpl(const MachineFunction &MF) const override;
  67: 
  68: private:
  69:   void adjustStackToMatchRecords(MachineBasicBlock &MBB,
  70:                                  MachineBasicBlock::iterator MI,
  71:                                  bool allocate) const;
  72: 
  73:   const ARCSubtarget &ST;
  74: };
  75: 
  76: } // end namespace llvm
  77: 
  78: #endif // LLVM_LIB_TARGET_ARC_ARCFRAMELOWERING_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

## Key Concepts / 关键概念

- Prologue and epilogue emission / 序言与尾声生成
- Stack object layout / 栈对象布局
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARC.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/TargetFrameLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `ARCFrameLowering.cpp`
