# CSKYInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrInfo.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the CSKY implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYInstrInfo.h - CSKY Instruction Information --------*- C++ -*---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the CSKY implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_CSKY_CSKYINSTRINFO_H
  14: #define LLVM_LIB_TARGET_CSKY_CSKYINSTRINFO_H
  15: 
  16: #include "MCTargetDesc/CSKYMCTargetDesc.h"
  17: #include "llvm/CodeGen/TargetInstrInfo.h"
  18: 
  19: #define GET_INSTRINFO_HEADER
  20: #include "CSKYGenInstrInfo.inc"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: 
  22: namespace llvm {
  23: 
  24: class CSKYRegisterInfo;
  25: class CSKYSubtarget;
  26: 
  27: class CSKYInstrInfo : public CSKYGenInstrInfo {
  28:   bool v2sf;
  29:   bool v2df;
  30:   bool v3sf;
  31:   bool v3df;
  32: 
  33: protected:
  34:   const CSKYSubtarget &STI;
  35: 
  36: public:
  37:   CSKYInstrInfo(const CSKYSubtarget &STI, const CSKYRegisterInfo &RI);
  38: 
  39:   Register isLoadFromStackSlot(const MachineInstr &MI,
  40:                                int &FrameIndex) const override;
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as CSKYRegisterInfo, CSKYSubtarget, CSKYInstrInfo, which organize the target-specific behavior exposed by the file. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 CSKYRegisterInfo, CSKYSubtarget, CSKYInstrInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:   Register isStoreToStackSlot(const MachineInstr &MI,
  42:                               int &FrameIndex) const override;
  43: 
  44:   void storeRegToStackSlot(
  45:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register SrcReg,
  46:       bool IsKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
  47:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  48: 
  49:   void loadRegFromStackSlot(
  50:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register DestReg,
  51:       int FrameIndex, const TargetRegisterClass *RC, Register VReg,
  52:       unsigned SubReg = 0,
  53:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  54: 
  55:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
  56:                    const DebugLoc &DL, Register DestReg, Register SrcReg,
  57:                    bool KillSrc, bool RenamableDest = false,
  58:                    bool RenamableSrc = false) const override;
  59: 
  60:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
  62:                         const DebugLoc &DL,
  63:                         int *BytesAdded = nullptr) const override;
  64: 
  65:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
  66:                      MachineBasicBlock *&FBB,
  67:                      SmallVectorImpl<MachineOperand> &Cond,
  68:                      bool AllowModify = false) const override;
  69: 
  70:   unsigned removeBranch(MachineBasicBlock &MBB,
  71:                         int *BytesRemoved = nullptr) const override;
  72: 
  73:   bool
  74:   reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;
  75: 
  76:   MachineBasicBlock *getBranchDestBlock(const MachineInstr &MI) const override;
  77: 
  78:   unsigned getInstSizeInBytes(const MachineInstr &MI) const override;
  79: 
  80:   Register getGlobalBaseReg(MachineFunction &MF) const;
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-90

```cpp
  81: 
  82:   // Materializes the given integer Val into DstReg.
  83:   Register movImm(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  84:                   const DebugLoc &DL, uint64_t Val,
  85:                   MachineInstr::MIFlag Flag = MachineInstr::NoFlags) const;
  86: };
  87: 
  88: } // namespace llvm
  89: 
  90: #endif // LLVM_LIB_TARGET_CSKY_CSKYINSTRINFO_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/CSKYMCTargetDesc.h`, `llvm/CodeGen/TargetInstrInfo.h`, `CSKYGenInstrInfo.inc`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `CSKYGenInstrInfo.inc`
- Local companions / 本地配套文件: `CSKYInstrInfo.cpp`, `CSKYInstrInfo.td`
