# SPIRVISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVISelLowering.h`
- Repository: `llvm-project`
- Purpose (EN): This file defines the interfaces that SPIR-V uses to lower LLVM code into a selection DAG.
- 目的（中文）: 该文件实现目标相关的指令选择逻辑，把 LLVM 或机器 IR 映射到后端可接受的表示。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVISelLowering.h - SPIR-V DAG Lowering Interface -----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file defines the interfaces that SPIR-V uses to lower LLVM code into a
10: // selection DAG.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVISELLOWERING_H
15: #define LLVM_LIB_TARGET_SPIRV_SPIRVISELLOWERING_H
16:
17: #include "SPIRVGlobalRegistry.h"
18: #include "llvm/CodeGen/TargetLowering.h"
19:
20: namespace llvm {
21: class SPIRVSubtarget;
22:
23: class SPIRVTargetLowering : public TargetLowering {
24:   const SPIRVSubtarget &STI;
25:
26: public:
27:   explicit SPIRVTargetLowering(const TargetMachine &TM,
28:                                const SPIRVSubtarget &ST);
29:
30:   // Stop IRTranslator breaking up FMA instrs to preserve types information.
31:   bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
32:                                   EVT) const override {
33:     return true;
34:   }
35:
36:   // prevent creation of jump tables
37:   bool areJTsAllowed(const Function *) const override { return false; }
38:
39:   // This is to prevent sexts of non-i64 vector indices which are generated
40:   // within general IRTranslator hence type generation for it is omitted.
```
- EN: This range defines or declares important types such as SPIRVSubtarget, SPIRVTargetLowering, areJTsAllowed, shaping the data model used by SPIRVISelLowering.h.
- CN: 这一段定义或声明了 SPIRVSubtarget、SPIRVTargetLowering、areJTsAllowed 等关键类型，构成 SPIRVISelLowering.h 使用的数据模型。

### Lines 41-80
```cpp
41:   unsigned getVectorIdxWidth(const DataLayout &DL) const override { return 32; }
42:   unsigned getNumRegistersForCallingConv(LLVMContext &Context,
43:                                          CallingConv::ID CC,
44:                                          EVT VT) const override;
45:   MVT getRegisterTypeForCallingConv(LLVMContext &Context, CallingConv::ID CC,
46:                                     EVT VT) const override;
47:   void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &Infos,
48:                           const CallBase &I, MachineFunction &MF,
49:                           unsigned Intrinsic) const override;
50:
51:   ConstraintType getConstraintType(StringRef Constraint) const override;
52:
53:   std::pair<unsigned, const TargetRegisterClass *>
54:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
55:                                StringRef Constraint, MVT VT) const override;
56:   unsigned
57:   getNumRegisters(LLVMContext &Context, EVT VT,
58:                   std::optional<MVT> RegisterVT = std::nullopt) const override {
59:     return 1;
60:   }
61:
62:   // Call the default implementation and finalize target lowering by inserting
63:   // extra instructions required to preserve validity of SPIR-V code imposed by
64:   // the standard.
65:   void finalizeLowering(MachineFunction &MF) const override;
66:
67:   MVT getPreferredSwitchConditionType(LLVMContext &Context,
68:                                       EVT ConditionVT) const override {
69:     return ConditionVT.getSimpleVT();
70:   }
71:
72:   bool enforcePtrTypeCompatibility(MachineInstr &I, unsigned PtrOpIdx,
73:                                    unsigned OpIdx) const;
74:   bool insertLogicalCopyOnResult(MachineInstr &I,
75:                                  SPIRVTypeInst NewResultType) const;
76:
77:   AtomicExpansionKind
78:   shouldExpandAtomicRMWInIR(const AtomicRMWInst *RMW) const override;
79:   AtomicExpansionKind
80:   shouldCastAtomicRMWIInIR(AtomicRMWInst *RMWI) const override;
```
- EN: This range declares interfaces or inline helpers such as getVectorIdxWidth, getConstraintType, finalizeLowering, getSimpleVT, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getVectorIdxWidth、getConstraintType、finalizeLowering、getSimpleVT 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-88
```cpp
81:
82:   bool shouldIssueAtomicLoadForAtomicEmulationLoop() const override {
83:     return false;
84:   }
85: };
86: } // namespace llvm
87:
88: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVISELLOWERING_H
```
- EN: This range declares interfaces or inline helpers such as shouldIssueAtomicLoadForAtomicEmulationLoop, defining how other backend pieces interact with this header.
- CN: 这一段声明了 shouldIssueAtomicLoadForAtomicEmulationLoop 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: Instruction selection maps generic LLVM operations onto target-specific machine instructions.
  - CN: 指令选择负责把通用 LLVM 操作映射为目标相关的机器指令。
- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVSubtarget, SPIRVTargetLowering, areJTsAllowed, getVectorIdxWidth, getConstraintType, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVSubtarget, SPIRVTargetLowering, areJTsAllowed, getVectorIdxWidth, getConstraintType，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVGlobalRegistry.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/TargetLowering.h`
