# SPIRVMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/MCTargetDesc/SPIRVMCCodeEmitter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the SPIRVMCCodeEmitter class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVMCCodeEmitter.cpp - Emit SPIR-V machine code -------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the SPIRVMCCodeEmitter class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "MCTargetDesc/SPIRVMCTargetDesc.h"
14: #include "llvm/MC/MCCodeEmitter.h"
15: #include "llvm/MC/MCFixup.h"
16: #include "llvm/MC/MCInst.h"
17: #include "llvm/MC/MCInstrInfo.h"
18: #include "llvm/MC/MCSubtargetInfo.h"
19: #include "llvm/Support/EndianStream.h"
20:
21: using namespace llvm;
22:
23: #define DEBUG_TYPE "spirv-mccodeemitter"
24:
25: namespace {
26:
27: class SPIRVMCCodeEmitter : public MCCodeEmitter {
28:   const MCInstrInfo &MCII;
29:
30: public:
31:   SPIRVMCCodeEmitter(const MCInstrInfo &mcii) : MCII(mcii) {}
32:   SPIRVMCCodeEmitter(const SPIRVMCCodeEmitter &) = delete;
33:   void operator=(const SPIRVMCCodeEmitter &) = delete;
34:   ~SPIRVMCCodeEmitter() override = default;
35:
36:   // getBinaryCodeForInstr - TableGen'erated function for getting the
37:   // binary encoding for an instruction.
38:   uint64_t getBinaryCodeForInstr(const MCInst &MI,
39:                                  SmallVectorImpl<MCFixup> &Fixups,
40:                                  const MCSubtargetInfo &STI) const;
```
- EN: This range defines or declares important types such as SPIRVMCCodeEmitter, shaping the data model used by SPIRVMCCodeEmitter.cpp.
- CN: 这一段定义或声明了 SPIRVMCCodeEmitter 等关键类型，构成 SPIRVMCCodeEmitter.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:
42:   void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
43:                          SmallVectorImpl<MCFixup> &Fixups,
44:                          const MCSubtargetInfo &STI) const override;
45:   void encodeUnknownType(const MCInst &MI, SmallVectorImpl<char> &CB,
46:                          SmallVectorImpl<MCFixup> &Fixups,
47:                          const MCSubtargetInfo &STI) const;
48: };
49:
50: } // end anonymous namespace
51:
52: MCCodeEmitter *llvm::createSPIRVMCCodeEmitter(const MCInstrInfo &MCII,
53:                                               MCContext &Ctx) {
54:   return new SPIRVMCCodeEmitter(MCII);
55: }
56:
57: using EndianWriter = support::endian::Writer;
58:
59: // Check if the instruction has a type argument for operand 1, and defines an ID
60: // output register in operand 0. If so, we need to swap operands 0 and 1 so the
61: // type comes first in the output, despide coming second in the MCInst.
62: static bool hasType(const MCInst &MI, const MCInstrInfo &MII) {
63:   const MCInstrDesc &MCDesc = MII.get(MI.getOpcode());
64:   // If we define an output, and have at least one other argument.
65:   if (MCDesc.getNumDefs() == 1 && MCDesc.getNumOperands() >= 2) {
66:     // Check if we define an ID, and take a type as operand 1.
67:     return MCDesc.operands()[0].RegClass >= 0 &&
68:            MCDesc.operands()[1].RegClass >= 0 &&
69:            MCDesc.operands()[0].RegClass != SPIRV::TYPERegClassID &&
70:            MCDesc.operands()[1].RegClass == SPIRV::TYPERegClassID;
71:   }
72:   return false;
73: }
74:
75: static void emitOperand(const MCOperand &Op, SmallVectorImpl<char> &CB) {
76:   if (Op.isReg()) {
77:     // Emit the id index starting at 1 (0 is an invalid index).
78:     support::endian::write<uint32_t>(
79:         CB, SPIRV::getIDFromRegister(Op.getReg().id()) + 1,
80:         llvm::endianness::little);
```
- EN: This range implements operational logic in helpers such as SPIRVMCCodeEmitter, hasType, get, emitOperand, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVMCCodeEmitter、hasType、get、emitOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   } else if (Op.isImm()) {
 82:     support::endian::write(CB, static_cast<uint32_t>(Op.getImm()),
 83:                            llvm::endianness::little);
 84:   } else {
 85:     llvm_unreachable("Unexpected operand type in VReg");
 86:   }
 87: }
 88:
 89: // Emit the type in operand 1 before the ID in operand 0 it defines, and all
 90: // remaining operands in the order they come naturally.
 91: static void emitTypedInstrOperands(const MCInst &MI,
 92:                                    SmallVectorImpl<char> &CB) {
 93:   unsigned NumOps = MI.getNumOperands();
 94:   emitOperand(MI.getOperand(1), CB);
 95:   emitOperand(MI.getOperand(0), CB);
 96:   for (unsigned i = 2; i < NumOps; ++i)
 97:     emitOperand(MI.getOperand(i), CB);
 98: }
 99:
100: // Emit operands in the order they come naturally.
101: static void emitUntypedInstrOperands(const MCInst &MI,
102:                                      SmallVectorImpl<char> &CB) {
103:   for (const auto &Op : MI)
104:     emitOperand(Op, CB);
105: }
106:
107: void SPIRVMCCodeEmitter::encodeUnknownType(const MCInst &MI,
108:                                            SmallVectorImpl<char> &CB,
109:                                            SmallVectorImpl<MCFixup> &Fixups,
110:                                            const MCSubtargetInfo &STI) const {
111:   // Encode the first 32 SPIR-V bits with the number of args and the opcode.
112:   const uint64_t OpCode = MI.getOperand(1).getImm();
113:   const uint32_t NumWords = MI.getNumOperands();
114:   const uint32_t FirstWord = (0xFFFF & NumWords) << 16 | (0xFFFF & OpCode);
115:
116:   // encoding: <opcode+len> <result type> [<operand0> <operand1> ...]
117:   support::endian::write(CB, FirstWord, llvm::endianness::little);
118:
119:   emitOperand(MI.getOperand(0), CB);
120:   for (unsigned i = 2; i < NumWords; ++i)
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, getNumOperands, emitOperand, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、getNumOperands、emitOperand、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-146
```cpp
121:     emitOperand(MI.getOperand(i), CB);
122: }
123:
124: void SPIRVMCCodeEmitter::encodeInstruction(const MCInst &MI,
125:                                            SmallVectorImpl<char> &CB,
126:                                            SmallVectorImpl<MCFixup> &Fixups,
127:                                            const MCSubtargetInfo &STI) const {
128:   if (MI.getOpcode() == SPIRV::UNKNOWN_type) {
129:     encodeUnknownType(MI, CB, Fixups, STI);
130:     return;
131:   }
132:
133:   // Encode the first 32 SPIR-V bytes with the number of args and the opcode.
134:   const uint64_t OpCode = getBinaryCodeForInstr(MI, Fixups, STI);
135:   const uint32_t NumWords = MI.getNumOperands() + 1;
136:   const uint32_t FirstWord = (NumWords << 16) | OpCode;
137:   support::endian::write(CB, FirstWord, llvm::endianness::little);
138:
139:   // Emit the instruction arguments (emitting the output type first if present).
140:   if (hasType(MI, MCII))
141:     emitTypedInstrOperands(MI, CB);
142:   else
143:     emitUntypedInstrOperands(MI, CB);
144: }
145:
146: #include "SPIRVGenMCCodeEmitter.inc"
```
- EN: This range implements operational logic in helpers such as emitOperand, encodeUnknownType, getBinaryCodeForInstr, support::endian::write, translating backend policy into executable code.
- CN: 这一段实现了 emitOperand、encodeUnknownType、getBinaryCodeForInstr、support::endian::write 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVMCCodeEmitter, hasType, get, emitOperand, llvm_unreachable, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVMCCodeEmitter, hasType, get, emitOperand, llvm_unreachable，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVMCTargetDesc.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/MC/MCCodeEmitter.h`
  - `llvm/MC/MCFixup.h`
  - `llvm/MC/MCInst.h`
  - `llvm/MC/MCInstrInfo.h`
  - `llvm/MC/MCSubtargetInfo.h`
  - `llvm/Support/EndianStream.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenMCCodeEmitter.inc`
