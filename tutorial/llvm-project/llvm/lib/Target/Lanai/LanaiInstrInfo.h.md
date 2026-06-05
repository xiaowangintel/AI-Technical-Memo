# LanaiInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiInstrInfo.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the Lanai implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- LanaiInstrInfo.h - Lanai Instruction Information ---------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the Lanai implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_LANAI_LANAIINSTRINFO_H
  14: #define LLVM_LIB_TARGET_LANAI_LANAIINSTRINFO_H
  15: 
  16: #include "LanaiRegisterInfo.h"
  17: #include "MCTargetDesc/LanaiMCTargetDesc.h"
  18: #include "llvm/CodeGen/TargetInstrInfo.h"
  19: 
  20: #define GET_INSTRINFO_HEADER
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: #include "LanaiGenInstrInfo.inc"
  22: 
  23: namespace llvm {
  24: 
  25: class LanaiSubtarget;
  26: 
  27: class LanaiInstrInfo : public LanaiGenInstrInfo {
  28:   const LanaiRegisterInfo RegisterInfo;
  29: 
  30: public:
  31:   LanaiInstrInfo(const LanaiSubtarget &STI);
  32: 
  33:   // getRegisterInfo - TargetInstrInfo is a superset of MRegister info.  As
  34:   // such, whenever a client has an instance of instruction info, it should
  35:   // always be able to get register info as well (through this method).
  36:   virtual const LanaiRegisterInfo &getRegisterInfo() const {
  37:     return RegisterInfo;
  38:   }
  39: 
  40:   bool areMemAccessesTriviallyDisjoint(const MachineInstr &MIa,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as LanaiSubtarget, LanaiInstrInfo, which organize the target-specific behavior exposed by the file. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 LanaiSubtarget, LanaiInstrInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:                                        const MachineInstr &MIb) const override;
  42: 
  43:   Register isLoadFromStackSlot(const MachineInstr &MI,
  44:                                int &FrameIndex) const override;
  45: 
  46:   Register isLoadFromStackSlotPostFE(const MachineInstr &MI,
  47:                                      int &FrameIndex) const override;
  48: 
  49:   Register isStoreToStackSlot(const MachineInstr &MI,
  50:                               int &FrameIndex) const override;
  51: 
  52:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator Position,
  53:                    const DebugLoc &DL, Register DestinationRegister,
  54:                    Register SourceRegister, bool KillSource,
  55:                    bool RenamableDest = false,
  56:                    bool RenamableSrc = false) const override;
  57: 
  58:   void storeRegToStackSlot(
  59:       MachineBasicBlock &MBB, MachineBasicBlock::iterator Position,
  60:       Register SourceRegister, bool IsKill, int FrameIndex,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:       const TargetRegisterClass *RegisterClass, Register VReg,
  62:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  63: 
  64:   void loadRegFromStackSlot(
  65:       MachineBasicBlock &MBB, MachineBasicBlock::iterator Position,
  66:       Register DestinationRegister, int FrameIndex,
  67:       const TargetRegisterClass *RegisterClass, Register VReg,
  68:       unsigned SubReg = 0,
  69:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  70: 
  71:   bool expandPostRAPseudo(MachineInstr &MI) const override;
  72: 
  73:   bool getMemOperandsWithOffsetWidth(
  74:       const MachineInstr &LdSt,
  75:       SmallVectorImpl<const MachineOperand *> &BaseOps, int64_t &Offset,
  76:       bool &OffsetIsScalable, LocationSize &Width,
  77:       const TargetRegisterInfo *TRI) const override;
  78: 
  79:   bool getMemOperandWithOffsetWidth(const MachineInstr &LdSt,
  80:                                     const MachineOperand *&BaseOp,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81:                                     int64_t &Offset, LocationSize &Width,
  82:                                     const TargetRegisterInfo *TRI) const;
  83: 
  84:   std::pair<unsigned, unsigned>
  85:   decomposeMachineOperandsTargetFlags(unsigned TF) const override;
  86: 
  87:   ArrayRef<std::pair<unsigned, const char *>>
  88:   getSerializableDirectMachineOperandTargetFlags() const override;
  89: 
  90:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TrueBlock,
  91:                      MachineBasicBlock *&FalseBlock,
  92:                      SmallVectorImpl<MachineOperand> &Condition,
  93:                      bool AllowModify) const override;
  94: 
  95:   unsigned removeBranch(MachineBasicBlock &MBB,
  96:                         int *BytesRemoved = nullptr) const override;
  97: 
  98:   // For a comparison instruction, return the source registers in SrcReg and
  99:   // SrcReg2 if having two register operands, and the value it compares against
 100:   // in CmpValue. Return true if the comparison instruction can be analyzed.
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-120

```cpp
 101:   bool analyzeCompare(const MachineInstr &MI, Register &SrcReg,
 102:                       Register &SrcReg2, int64_t &CmpMask,
 103:                       int64_t &CmpValue) const override;
 104: 
 105:   // See if the comparison instruction can be converted into something more
 106:   // efficient. E.g., on Lanai register-register instructions can set the flag
 107:   // register, obviating the need for a separate compare.
 108:   bool optimizeCompareInstr(MachineInstr &CmpInstr, Register SrcReg,
 109:                             Register SrcReg2, int64_t CmpMask, int64_t CmpValue,
 110:                             const MachineRegisterInfo *MRI) const override;
 111: 
 112:   // Given a select instruction that was understood by analyzeSelect and
 113:   // returned Optimizable = true, attempt to optimize MI by merging it with one
 114:   // of its operands. Returns NULL on failure.
 115:   //
 116:   // When successful, returns the new select instruction. The client is
 117:   // responsible for deleting MI.
 118:   //
 119:   // If both sides of the select can be optimized, the TrueOp is modifed.
 120:   // PreferFalse is not used.
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 121-140

```cpp
 121:   MachineInstr *optimizeSelect(MachineInstr &MI,
 122:                                SmallPtrSetImpl<MachineInstr *> &SeenMIs,
 123:                                bool PreferFalse) const override;
 124: 
 125:   bool reverseBranchCondition(
 126:       SmallVectorImpl<MachineOperand> &Condition) const override;
 127: 
 128:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TrueBlock,
 129:                         MachineBasicBlock *FalseBlock,
 130:                         ArrayRef<MachineOperand> Condition,
 131:                         const DebugLoc &DL,
 132:                         int *BytesAdded = nullptr) const override;
 133: };
 134: 
 135: static inline bool isSPLSOpcode(unsigned Opcode) {
 136:   switch (Opcode) {
 137:   case Lanai::LDBs_RI:
 138:   case Lanai::LDBz_RI:
 139:   case Lanai::LDHs_RI:
 140:   case Lanai::LDHz_RI:
```

- EN: Function bodies or method definitions such as isSPLSOpcode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isSPLSOpcode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 141-160

```cpp
 141:   case Lanai::STB_RI:
 142:   case Lanai::STH_RI:
 143:     return true;
 144:   default:
 145:     return false;
 146:   }
 147: }
 148: 
 149: static inline bool isRMOpcode(unsigned Opcode) {
 150:   switch (Opcode) {
 151:   case Lanai::LDW_RI:
 152:   case Lanai::SW_RI:
 153:     return true;
 154:   default:
 155:     return false;
 156:   }
 157: }
 158: 
 159: static inline bool isRRMOpcode(unsigned Opcode) {
 160:   switch (Opcode) {
```

- EN: Function bodies or method definitions such as isRMOpcode, isRRMOpcode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isRMOpcode, isRRMOpcode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 161-178

```cpp
 161:   case Lanai::LDBs_RR:
 162:   case Lanai::LDBz_RR:
 163:   case Lanai::LDHs_RR:
 164:   case Lanai::LDHz_RR:
 165:   case Lanai::LDWz_RR:
 166:   case Lanai::LDW_RR:
 167:   case Lanai::STB_RR:
 168:   case Lanai::STH_RR:
 169:   case Lanai::SW_RR:
 170:     return true;
 171:   default:
 172:     return false;
 173:   }
 174: }
 175: 
 176: } // namespace llvm
 177: 
 178: #endif // LLVM_LIB_TARGET_LANAI_LANAIINSTRINFO_H
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
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiRegisterInfo.h`, `MCTargetDesc/LanaiMCTargetDesc.h`, `llvm/CodeGen/TargetInstrInfo.h`, `LanaiGenInstrInfo.inc`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `LanaiGenInstrInfo.inc`
- Local companions / 本地配套文件: `LanaiInstrInfo.cpp`, `LanaiInstrInfo.td`
