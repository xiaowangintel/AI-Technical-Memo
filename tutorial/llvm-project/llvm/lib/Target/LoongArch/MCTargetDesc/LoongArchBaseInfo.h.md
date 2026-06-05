# LoongArchBaseInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchBaseInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- LoongArchBaseInfo.h - Top level definitions for LoongArch MC -*- C++ -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains small standalone enum definitions and helper function
  10: // definitions for the LoongArch target useful for the compiler back-end and the
  11: // MC libraries.
  12: //
  13: //===----------------------------------------------------------------------===//
  14: #ifndef LLVM_LIB_TARGET_LOONGARCH_MCTARGETDESC_LOONGARCHBASEINFO_H
  15: #define LLVM_LIB_TARGET_LOONGARCH_MCTARGETDESC_LOONGARCHBASEINFO_H
  16: 
  17: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  18: #include "llvm/ADT/StringRef.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchMCTargetDesc.h`, `StringRef.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchMCTargetDesc.h`, `StringRef.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/ADT/StringSwitch.h"
  20: #include "llvm/CodeGen/MachineOperand.h"
  21: #include "llvm/MC/MCInstrDesc.h"
  22: #include "llvm/TargetParser/SubtargetFeature.h"
  23: 
  24: namespace llvm {
  25: 
  26: // This namespace holds all of the target specific flags that instruction info
  27: // tracks.
  28: namespace LoongArchII {
  29: enum {
  30:   MO_None,
  31:   MO_CALL,
  32:   MO_CALL_PLT,
  33:   MO_PCREL_HI,
  34:   MO_PCREL_LO,
  35:   MO_PCREL64_LO,
  36:   MO_PCREL64_HI,
```
- **EN**: It imports dependencies such as `StringSwitch.h`, `MachineOperand.h`, `MCInstrDesc.h`, `SubtargetFeature.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `StringSwitch.h`, `MachineOperand.h`, `MCInstrDesc.h`, `SubtargetFeature.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   MO_GOT_PC_HI,
  38:   MO_GOT_PC_LO,
  39:   MO_GOT_PC64_LO,
  40:   MO_GOT_PC64_HI,
  41:   MO_LE_HI,
  42:   MO_LE_LO,
  43:   MO_LE64_LO,
  44:   MO_LE64_HI,
  45:   MO_IE_PC_HI,
  46:   MO_IE_PC_LO,
  47:   MO_IE_PC64_LO,
  48:   MO_IE_PC64_HI,
  49:   MO_LD_PC_HI,
  50:   MO_GD_PC_HI,
  51:   MO_CALL30,
  52:   MO_CALL36,
  53:   MO_DESC_PC_HI,
  54:   MO_DESC_PC_LO,
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   MO_DESC64_PC_HI,
  56:   MO_DESC64_PC_LO,
  57:   MO_DESC_LD,
  58:   MO_DESC_CALL,
  59:   MO_LE_HI_R,
  60:   MO_LE_ADD_R,
  61:   MO_LE_LO_R,
  62:   MO_PCADD_HI,
  63:   MO_PCADD_LO,
  64:   MO_GOT_PCADD_HI,
  65:   MO_GOT_PCADD_LO,
  66:   MO_IE_PCADD_HI,
  67:   MO_IE_PCADD_LO,
  68:   MO_LD_PCADD_HI,
  69:   MO_LD_PCADD_LO,
  70:   MO_GD_PCADD_HI,
  71:   MO_GD_PCADD_LO,
  72:   MO_DESC_PCADD_HI,
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   MO_DESC_PCADD_LO,
  74:   // TODO: Add more flags.
  75: 
  76:   // Used to differentiate between target-specific "direct" flags and "bitmask"
  77:   // flags. A machine operand can only have one "direct" flag, but can have
  78:   // multiple "bitmask" flags.
  79:   MO_DIRECT_FLAG_MASK = 0x3f,
  80: 
  81:   MO_RELAX = 0x40
  82: };
  83: 
  84: // Given a MachineOperand that may carry out "bitmask" flags, such as MO_RELAX,
  85: // return LoongArch target-specific "direct" flags.
  86: static inline unsigned getDirectFlags(const MachineOperand &MO) {
  87:   return MO.getTargetFlags() & MO_DIRECT_FLAG_MASK;
  88: }
  89: 
  90: // Add MO_RELAX "bitmask" flag when FeatureRelax is enabled.
```
- **EN**: The range implements or declares functions including `getDirectFlags`.
- **CN**: 这一段实现或声明了 `getDirectFlags` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91: static inline unsigned encodeFlags(unsigned Flags, bool Relax) {
  92:   return Flags | (Relax ? MO_RELAX : 0);
  93: }
  94: 
  95: // \returns true if the given MachineOperand has MO_RELAX "bitmask" flag.
  96: static inline bool hasRelaxFlag(const MachineOperand &MO) {
  97:   return MO.getTargetFlags() & MO_RELAX;
  98: }
  99: 
 100: // Target-specific flags of LAInst.
 101: // All definitions must match LoongArchInstrFormats.td.
 102: enum {
 103:   // Whether the instruction's rd is normally required to differ from rj and
 104:   // rk, in the way the 3-register atomic memory operations behave
 105:   // (Section 2.2.7.1 and 2.2.7.2, LoongArch Reference Manual Volume 1 v1.10;
 106:   // while Section 2.2.7.3 lacked similar description for the AMCAS
 107:   // instructions, at least the INE exception is still signaled on Loongson
 108:   // 3A6000 when its rd == rj).
```
- **EN**: The range implements or declares functions including `encodeFlags`, `hasRelaxFlag`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `encodeFlags`, `hasRelaxFlag` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   //
 110:   // Used for generating diagnostics for assembler input that violate the
 111:   // constraint. As described on the manual, the covered instructions require
 112:   // rd != rj && rd != rk to work as intended.
 113:   IsSubjectToAMORdConstraintShift = 0,
 114:   IsSubjectToAMORdConstraintMask = 1 << IsSubjectToAMORdConstraintShift,
 115: 
 116:   // Whether the instruction belongs to the AMCAS family.
 117:   IsAMCASShift = IsSubjectToAMORdConstraintShift + 1,
 118:   IsAMCASMask = 1 << IsAMCASShift,
 119: };
 120: 
 121: /// \returns true if this instruction's rd is normally required to differ
 122: /// from rj and rk, in the way 3-register atomic memory operations behave.
 123: static inline bool isSubjectToAMORdConstraint(uint64_t TSFlags) {
 124:   return TSFlags & IsSubjectToAMORdConstraintMask;
 125: }
 126: 
```
- **EN**: The range implements or declares functions including `isSubjectToAMORdConstraint`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSubjectToAMORdConstraint` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127: /// \returns true if this instruction belongs to the AMCAS family.
 128: static inline bool isAMCAS(uint64_t TSFlags) { return TSFlags & IsAMCASMask; }
 129: } // end namespace LoongArchII
 130: 
 131: namespace LoongArchABI {
 132: enum ABI {
 133:   ABI_ILP32S,
 134:   ABI_ILP32F,
 135:   ABI_ILP32D,
 136:   ABI_LP64S,
 137:   ABI_LP64F,
 138:   ABI_LP64D,
 139:   ABI_Unknown
 140: };
 141: 
 142: ABI computeTargetABI(const Triple &TT, const FeatureBitset &FeatureBits,
 143:                      StringRef ABIName);
 144: ABI getTargetABI(StringRef ABIName);
```
- **EN**: The range implements or declares functions including `isAMCAS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isAMCAS` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-152 / 第 145-152 行
```cpp
 145: 
 146: // Returns the register used to hold the stack pointer after realignment.
 147: MCRegister getBPReg();
 148: } // end namespace LoongArchABI
 149: 
 150: } // end namespace llvm
 151: 
 152: #endif // LLVM_LIB_TARGET_LOONGARCH_MCTARGETDESC_LOONGARCHBASEINFO_H
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `llvm/ADT/StringRef.h`
- `llvm/ADT/StringSwitch.h`
- `llvm/CodeGen/MachineOperand.h`
- `llvm/MC/MCInstrDesc.h`
- `llvm/TargetParser/SubtargetFeature.h`
