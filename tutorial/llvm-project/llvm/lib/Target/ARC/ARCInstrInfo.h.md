# ARCInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCInstrInfo.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the ARC implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCInstrInfo.h - ARC Instruction Information -------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the ARC implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_ARC_ARCINSTRINFO_H
  14: #define LLVM_LIB_TARGET_ARC_ARCINSTRINFO_H
  15: 
  16: #include "ARCRegisterInfo.h"
  17: #include "llvm/CodeGen/TargetInstrInfo.h"
  18: 
  19: #define GET_INSTRINFO_HEADER
  20: #include "ARCGenInstrInfo.inc"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: 
  22: namespace llvm {
  23: 
  24: class ARCSubtarget;
  25: 
  26: class ARCInstrInfo : public ARCGenInstrInfo {
  27:   const ARCRegisterInfo RI;
  28:   virtual void anchor();
  29: 
  30: public:
  31:   ARCInstrInfo(const ARCSubtarget &);
  32: 
  33:   const ARCRegisterInfo &getRegisterInfo() const { return RI; }
  34: 
  35:   /// If the specified machine instruction is a direct
  36:   /// load from a stack slot, return the virtual or physical register number of
  37:   /// the destination along with the FrameIndex of the loaded stack slot.  If
  38:   /// not, return 0.  This predicate must return 0 if the instruction has
  39:   /// any side effects other than loading from the stack slot.
  40:   Register isLoadFromStackSlot(const MachineInstr &MI,
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as ARCSubtarget, ARCInstrInfo, which organize the target-specific behavior exposed by the file. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 ARCSubtarget, ARCInstrInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:                                int &FrameIndex) const override;
  42: 
  43:   /// If the specified machine instruction is a direct
  44:   /// store to a stack slot, return the virtual or physical register number of
  45:   /// the source reg along with the FrameIndex of the loaded stack slot.  If
  46:   /// not, return 0.  This predicate must return 0 if the instruction has
  47:   /// any side effects other than storing to the stack slot.
  48:   Register isStoreToStackSlot(const MachineInstr &MI,
  49:                               int &FrameIndex) const override;
  50: 
  51:   unsigned getInstSizeInBytes(const MachineInstr &MI) const override;
  52: 
  53:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
  54:                      MachineBasicBlock *&FBB,
  55:                      SmallVectorImpl<MachineOperand> &Cond,
  56:                      bool AllowModify) const override;
  57: 
  58:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
  59:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
  60:                         const DebugLoc &,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:                         int *BytesAdded = nullptr) const override;
  62: 
  63:   unsigned removeBranch(MachineBasicBlock &MBB,
  64:                         int *BytesRemoved = nullptr) const override;
  65: 
  66:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
  67:                    const DebugLoc &, Register DestReg, Register SrcReg,
  68:                    bool KillSrc, bool RenamableDest = false,
  69:                    bool RenamableSrc = false) const override;
  70: 
  71:   void storeRegToStackSlot(
  72:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register SrcReg,
  73:       bool IsKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
  74:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  75: 
  76:   void loadRegFromStackSlot(
  77:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register DestReg,
  78:       int FrameIndex, const TargetRegisterClass *RC, Register VReg,
  79:       unsigned subReg = 0,
  80:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81: 
  82:   bool
  83:   reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;
  84: 
  85: 
  86:   bool isPostIncrement(const MachineInstr &MI) const override;
  87: 
  88:   // ARC-specific
  89:   bool isPreIncrement(const MachineInstr &MI) const;
  90: 
  91:   virtual bool getBaseAndOffsetPosition(const MachineInstr &MI,
  92:                                         unsigned &BasePos,
  93:                                         unsigned &OffsetPos) const override;
  94: 
  95:   // Emit code before MBBI to load immediate value into physical register Reg.
  96:   // Returns an iterator to the new instruction.
  97:   MachineBasicBlock::iterator loadImmediate(MachineBasicBlock &MBB,
  98:                                             MachineBasicBlock::iterator MI,
  99:                                             unsigned Reg, uint64_t Value) const;
 100: };
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-104

```cpp
 101: 
 102: } // end namespace llvm
 103: 
 104: #endif // LLVM_LIB_TARGET_ARC_ARCINSTRINFO_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCRegisterInfo.h`, `llvm/CodeGen/TargetInstrInfo.h`, `ARCGenInstrInfo.inc`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `ARCGenInstrInfo.inc`
- Local companions / 本地配套文件: `ARCInstrInfo.cpp`, `ARCInstrInfo.td`
