# SPIRVInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVInstrInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains the SPIR-V implementation of the TargetInstrInfo class.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVInstrInfo.h - SPIR-V Instruction Information -------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the SPIR-V implementation of the TargetInstrInfo class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVINSTRINFO_H
14: #define LLVM_LIB_TARGET_SPIRV_SPIRVINSTRINFO_H
15:
16: #include "SPIRVRegisterInfo.h"
17: #include "llvm/CodeGen/TargetInstrInfo.h"
18:
19: #define GET_INSTRINFO_HEADER
20: #include "SPIRVGenInstrInfo.inc"
21:
22: namespace llvm {
23: class SPIRVSubtarget;
24:
25: class SPIRVInstrInfo : public SPIRVGenInstrInfo {
26:   const SPIRVRegisterInfo RI;
27:
28: public:
29:   explicit SPIRVInstrInfo(const SPIRVSubtarget &STI);
30:
31:   const SPIRVRegisterInfo &getRegisterInfo() const { return RI; }
32:   bool isHeaderInstr(const MachineInstr &MI) const;
33:   bool isConstantInstr(const MachineInstr &MI) const;
34:   bool isSpecConstantInstr(const MachineInstr &MI) const;
35:   bool isInlineAsmDefInstr(const MachineInstr &MI) const;
36:   bool isTypeDeclInstr(const MachineInstr &MI) const;
37:   bool isDecorationInstr(const MachineInstr &MI) const;
38:   bool isAliasingInstr(const MachineInstr &MI) const;
39:   bool canUseFastMathFlags(const MachineInstr &MI,
40:                            bool KHRFloatControls2) const;
```
- EN: This range defines or declares important types such as SPIRVSubtarget, SPIRVInstrInfo, getRegisterInfo, isHeaderInstr, shaping the data model used by SPIRVInstrInfo.h.
- CN: 这一段定义或声明了 SPIRVSubtarget、SPIRVInstrInfo、getRegisterInfo、isHeaderInstr 等关键类型，构成 SPIRVInstrInfo.h 使用的数据模型。

### Lines 41-73
```cpp
41:   bool canUseNSW(const MachineInstr &MI) const;
42:   bool canUseNUW(const MachineInstr &MI) const;
43:
44:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
45:                      MachineBasicBlock *&FBB,
46:                      SmallVectorImpl<MachineOperand> &Cond,
47:                      bool AllowModify = false) const override;
48:
49:   unsigned removeBranch(MachineBasicBlock &MBB,
50:                         int *BytesRemoved = nullptr) const override;
51:
52:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
53:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
54:                         const DebugLoc &DL,
55:                         int *BytesAdded = nullptr) const override;
56:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
57:                    const DebugLoc &DL, Register DestReg, Register SrcReg,
58:                    bool KillSrc, bool RenamableDest = false,
59:                    bool RenamableSrc = false) const override;
60: };
61:
62: namespace SPIRV {
63: enum AsmComments : MachineInstr::AsmPrinterFlagTy {
64:   // It is a half type
65:   ASM_PRINTER_WIDTH16 = MachineInstr::TAsmComments,
66:   // It is a 64 bit type
67:   ASM_PRINTER_WIDTH64 = ASM_PRINTER_WIDTH16 << 1,
68: };
69: } // namespace SPIRV
70:
71: } // namespace llvm
72:
73: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVINSTRINFO_H
```
- EN: This range defines or declares important types such as canUseNSW, canUseNUW, AsmComments, shaping the data model used by SPIRVInstrInfo.h.
- CN: 这一段定义或声明了 canUseNSW、canUseNUW、AsmComments 等关键类型，构成 SPIRVInstrInfo.h 使用的数据模型。

## Key Concepts / 关键概念

- EN: Instruction info files centralize opcode semantics, scheduling hooks, and target-specific machine properties.
  - CN: 指令信息文件集中描述操作码语义、调度钩子以及目标相关机器属性。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVSubtarget, SPIRVInstrInfo, getRegisterInfo, isHeaderInstr, isConstantInstr, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVSubtarget, SPIRVInstrInfo, getRegisterInfo, isHeaderInstr, isConstantInstr，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVRegisterInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/TargetInstrInfo.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenInstrInfo.inc`
