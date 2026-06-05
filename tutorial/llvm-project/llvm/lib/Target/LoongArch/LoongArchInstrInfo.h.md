# LoongArchInstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchInstrInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target instruction information for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责定义或实现目标指令信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- LoongArchInstrInfo.h - LoongArch Instruction Information ---*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the LoongArch implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_LOONGARCH_LOONGARCHINSTRINFO_H
  14: #define LLVM_LIB_TARGET_LOONGARCH_LOONGARCHINSTRINFO_H
  15: 
  16: #include "LoongArchRegisterInfo.h"
  17: #include "llvm/CodeGen/TargetInstrInfo.h"
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchRegisterInfo.h`, `TargetInstrInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchRegisterInfo.h`, `TargetInstrInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #define GET_INSTRINFO_HEADER
  20: #include "LoongArchGenInstrInfo.inc"
  21: 
  22: namespace llvm {
  23: 
  24: class LoongArchSubtarget;
  25: 
  26: class LoongArchInstrInfo : public LoongArchGenInstrInfo {
  27:   const LoongArchRegisterInfo RegInfo;
  28: 
  29: public:
  30:   explicit LoongArchInstrInfo(const LoongArchSubtarget &STI);
  31: 
  32:   const LoongArchRegisterInfo &getRegisterInfo() const { return RegInfo; }
  33: 
  34:   MCInst getNop() const override;
  35: 
  36:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
```
- **EN**: It imports dependencies such as `LoongArchGenInstrInfo.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchSubtarget`, `LoongArchInstrInfo`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchGenInstrInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchSubtarget`, `LoongArchInstrInfo` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:                    const DebugLoc &DL, Register DstReg, Register SrcReg,
  38:                    bool KillSrc, bool RenamableDest = false,
  39:                    bool RenamableSrc = false) const override;
  40: 
  41:   void storeRegToStackSlot(
  42:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
  43:       bool IsKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
  44:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  45:   void loadRegFromStackSlot(
  46:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register DstReg,
  47:       int FrameIndex, const TargetRegisterClass *RC, Register VReg,
  48:       unsigned SubReg = 0,
  49:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
  50: 
  51:   Register isLoadFromStackSlot(const MachineInstr &MI,
  52:                                int &FrameIndex) const override;
  53:   Register isLoadFromStackSlot(const MachineInstr &MI, int &FrameIndex,
  54:                                TypeSize &MemBytes) const override;
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   Register isStoreToStackSlot(const MachineInstr &MI,
  56:                               int &FrameIndex) const override;
  57:   Register isStoreToStackSlot(const MachineInstr &MI, int &FrameIndex,
  58:                               TypeSize &MemBytes) const override;
  59: 
  60:   // Materializes the given integer Val into DstReg.
  61:   void movImm(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  62:               const DebugLoc &DL, Register DstReg, uint64_t Val,
  63:               MachineInstr::MIFlag Flag = MachineInstr::NoFlags) const;
  64: 
  65:   unsigned getInstSizeInBytes(const MachineInstr &MI) const override;
  66: 
  67:   bool isAsCheapAsAMove(const MachineInstr &MI) const override;
  68: 
  69:   MachineBasicBlock *getBranchDestBlock(const MachineInstr &MI) const override;
  70: 
  71:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
  72:                      MachineBasicBlock *&FBB,
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:                      SmallVectorImpl<MachineOperand> &Cond,
  74:                      bool AllowModify) const override;
  75: 
  76:   bool isBranchOffsetInRange(unsigned BranchOpc,
  77:                              int64_t BrOffset) const override;
  78: 
  79:   bool isSafeToMove(const MachineInstr &MI, const MachineBasicBlock *MBB,
  80:                     const MachineFunction &MF) const override;
  81: 
  82:   bool isSchedulingBoundary(const MachineInstr &MI,
  83:                             const MachineBasicBlock *MBB,
  84:                             const MachineFunction &MF) const override;
  85: 
  86:   unsigned removeBranch(MachineBasicBlock &MBB,
  87:                         int *BytesRemoved = nullptr) const override;
  88: 
  89:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
  90:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:                         const DebugLoc &dl,
  92:                         int *BytesAdded = nullptr) const override;
  93: 
  94:   void insertIndirectBranch(MachineBasicBlock &MBB,
  95:                             MachineBasicBlock &NewDestBB,
  96:                             MachineBasicBlock &RestoreBB, const DebugLoc &DL,
  97:                             int64_t BrOffset, RegScavenger *RS) const override;
  98: 
  99:   bool
 100:   reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;
 101: 
 102:   std::pair<unsigned, unsigned>
 103:   decomposeMachineOperandsTargetFlags(unsigned TF) const override;
 104: 
 105:   ArrayRef<std::pair<unsigned, const char *>>
 106:   getSerializableDirectMachineOperandTargetFlags() const override;
 107: 
 108:   ArrayRef<std::pair<unsigned, const char *>>
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   getSerializableBitmaskMachineOperandTargetFlags() const override;
 110: 
 111:   bool canFoldIntoAddrMode(const MachineInstr &MemI, Register Reg,
 112:                            const MachineInstr &AddrI,
 113:                            ExtAddrMode &AM) const override;
 114:   MachineInstr *emitLdStWithAddr(MachineInstr &MemI,
 115:                                  const ExtAddrMode &AM) const override;
 116: 
 117: protected:
 118:   const LoongArchSubtarget &STI;
 119: };
 120: 
 121: namespace LoongArch {
 122: 
 123: // Returns true if this is the sext.w pattern, addi.w rd, rs, 0.
 124: bool isSEXT_W(const MachineInstr &MI);
 125: 
 126: // Mask assignments for floating-point.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-140 / 第 127-140 行
```cpp
 127: static constexpr unsigned FClassMaskSignalingNaN = 0x001;
 128: static constexpr unsigned FClassMaskQuietNaN = 0x002;
 129: static constexpr unsigned FClassMaskNegativeInfinity = 0x004;
 130: static constexpr unsigned FClassMaskNegativeNormal = 0x008;
 131: static constexpr unsigned FClassMaskNegativeSubnormal = 0x010;
 132: static constexpr unsigned FClassMaskNegativeZero = 0x020;
 133: static constexpr unsigned FClassMaskPositiveInfinity = 0x040;
 134: static constexpr unsigned FClassMaskPositiveNormal = 0x080;
 135: static constexpr unsigned FClassMaskPositiveSubnormal = 0x100;
 136: static constexpr unsigned FClassMaskPositiveZero = 0x200;
 137: } // namespace LoongArch
 138: 
 139: } // end namespace llvm
 140: #endif // LLVM_LIB_TARGET_LOONGARCH_LOONGARCHINSTRINFO_H
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchRegisterInfo.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `LoongArchGenInstrInfo.inc`
