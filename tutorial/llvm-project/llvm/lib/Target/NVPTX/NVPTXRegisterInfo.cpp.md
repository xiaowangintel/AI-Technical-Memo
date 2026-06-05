# NVPTXRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXRegisterInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the NVPTX implementation of the TargetRegisterInfo class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVPTXRegisterInfo.cpp - NVPTX Register Information -----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the NVPTX implementation of the TargetRegisterInfo class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXRegisterInfo.h"
14: #include "MCTargetDesc/NVPTXInstPrinter.h"
15: #include "NVPTX.h"
16: #include "NVPTXTargetMachine.h"
17: #include "llvm/ADT/BitVector.h"
18: #include "llvm/CodeGen/MachineFrameInfo.h"
19: #include "llvm/CodeGen/MachineFunction.h"
20: #include "llvm/CodeGen/TargetInstrInfo.h"
21:
22: using namespace llvm;
23:
24: #define DEBUG_TYPE "nvptx-reg-info"
25:
26: namespace llvm {
27: StringRef getNVPTXRegClassName(TargetRegisterClass const *RC) {
28:   if (RC == &NVPTX::B128RegClass)
29:     return ".b128";
30:   if (RC == &NVPTX::B64RegClass)
31:     // We use untyped (.b) integer registers here as NVCC does.
32:     // Correctness of generated code does not depend on register type,
33:     // but using .s/.u registers runs into ptxas bug that prevents
34:     // assembly of otherwise valid PTX into SASS. Despite PTX ISA
35:     // specifying only argument size for fp16 instructions, ptxas does
36:     // not allow using .s16 or .u16 arguments for .fp16
37:     // instructions. At the same time it allows using .s32/.u32
38:     // arguments for .fp16v2 instructions:
39:     //
40:     //   .reg .b16 rb16
```
- EN: This range implements operational logic in helpers such as getNVPTXRegClassName, translating backend policy into executable code.
- CN: 这一段实现了 getNVPTXRegClassName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:     //   .reg .s16 rs16
42:     //   add.f16 rb16,rb16,rb16; // OK
43:     //   add.f16 rs16,rs16,rs16; // Arguments mismatch for instruction 'add'
44:     // but:
45:     //   .reg .b32 rb32
46:     //   .reg .s32 rs32
47:     //   add.f16v2 rb32,rb32,rb32; // OK
48:     //   add.f16v2 rs32,rs32,rs32; // OK
49:     return ".b64";
50:   if (RC == &NVPTX::B32RegClass)
51:     return ".b32";
52:   if (RC == &NVPTX::B16RegClass)
53:     return ".b16";
54:   if (RC == &NVPTX::B1RegClass)
55:     return ".pred";
56:   if (RC == &NVPTX::SpecialRegsRegClass)
57:     return "!Special!";
58:   return "INTERNAL";
59: }
60:
61: StringRef getNVPTXRegClassStr(TargetRegisterClass const *RC) {
62:   if (RC == &NVPTX::B128RegClass)
63:     return "%rq";
64:   if (RC == &NVPTX::B64RegClass)
65:     return "%rd";
66:   if (RC == &NVPTX::B32RegClass)
67:     return "%r";
68:   if (RC == &NVPTX::B16RegClass)
69:     return "%rs";
70:   if (RC == &NVPTX::B1RegClass)
71:     return "%p";
72:   if (RC == &NVPTX::SpecialRegsRegClass)
73:     return "!Special!";
74:   return "INTERNAL";
75: }
76: } // namespace llvm
77:
78: NVPTXRegisterInfo::NVPTXRegisterInfo()
79:     : NVPTXGenRegisterInfo(0), StrPool(StrAlloc) {}
80:
```
- EN: This range implements operational logic in helpers such as getNVPTXRegClassStr, NVPTXRegisterInfo::NVPTXRegisterInfo, NVPTXGenRegisterInfo, translating backend policy into executable code.
- CN: 这一段实现了 getNVPTXRegClassStr、NVPTXRegisterInfo::NVPTXRegisterInfo、NVPTXGenRegisterInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81: #define GET_REGINFO_TARGET_DESC
 82: #include "NVPTXGenRegisterInfo.inc"
 83:
 84: /// NVPTX Callee Saved Registers
 85: const MCPhysReg *
 86: NVPTXRegisterInfo::getCalleeSavedRegs(const MachineFunction *) const {
 87:   static const MCPhysReg CalleeSavedRegs[] = { 0 };
 88:   return CalleeSavedRegs;
 89: }
 90:
 91: BitVector NVPTXRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
 92:   BitVector Reserved(getNumRegs());
 93:   for (unsigned Reg = NVPTX::ENVREG0; Reg <= NVPTX::ENVREG31; ++Reg) {
 94:     markSuperRegs(Reserved, Reg);
 95:   }
 96:   markSuperRegs(Reserved, NVPTX::VRFrame32);
 97:   markSuperRegs(Reserved, NVPTX::VRFrameLocal32);
 98:   markSuperRegs(Reserved, NVPTX::VRFrame64);
 99:   markSuperRegs(Reserved, NVPTX::VRFrameLocal64);
100:   markSuperRegs(Reserved, NVPTX::VRDepot);
101:   return Reserved;
102: }
103:
104: bool NVPTXRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
105:                                             int SPAdj, unsigned FIOperandNum,
106:                                             RegScavenger *) const {
107:   assert(SPAdj == 0 && "Unexpected");
108:
109:   MachineInstr &MI = *II;
110:   if (MI.isLifetimeMarker()) {
111:     MI.eraseFromParent();
112:     return true;
113:   }
114:
115:   const int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
116:
117:   const MachineFunction &MF = *MI.getParent()->getParent();
118:   const int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex) +
119:                      MI.getOperand(FIOperandNum + 1).getImm();
120:
```
- EN: This range implements operational logic in helpers such as NVPTXRegisterInfo::getCalleeSavedRegs, NVPTXRegisterInfo::getReservedRegs, Reserved, markSuperRegs, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXRegisterInfo::getCalleeSavedRegs、NVPTXRegisterInfo::getReservedRegs、Reserved、markSuperRegs 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:   // Using I0 as the frame pointer
122:   MI.getOperand(FIOperandNum).ChangeToRegister(getFrameRegister(MF), false);
123:   MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
124:   return false;
125: }
126:
127: Register NVPTXRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
128:   const NVPTXTargetMachine &TM =
129:       static_cast<const NVPTXTargetMachine &>(MF.getTarget());
130:   return TM.is64Bit() ? NVPTX::VRFrame64 : NVPTX::VRFrame32;
131: }
132:
133: Register
134: NVPTXRegisterInfo::getFrameLocalRegister(const MachineFunction &MF) const {
135:   const NVPTXTargetMachine &TM =
136:       static_cast<const NVPTXTargetMachine &>(MF.getTarget());
137:   return TM.is64Bit() ? NVPTX::VRFrameLocal64 : NVPTX::VRFrameLocal32;
138: }
139:
140: void NVPTXRegisterInfo::clearDebugRegisterMap() const {
141:   debugRegisterMap.clear();
142: }
143:
144: static uint64_t encodeRegisterForDwarf(StringRef RegisterName) {
145:   if (RegisterName.size() > 8)
146:     // The name is more than 8 characters long, and so won't fit into 64 bits.
147:     return 0;
148:
149:   // Encode the name string into a DWARF register number using cuda-gdb's
150:   // encoding.  See cuda_check_dwarf2_reg_ptx_virtual_register in cuda-tdep.c,
151:   // https://github.com/NVIDIA/cuda-gdb/blob/e5cf3bddae520ffb326f95b4d98ce5c7474b828b/gdb/cuda/cuda-tdep.c#L353
152:   // IE the bytes of the string are concatenated in reverse into a single
153:   // number, which is stored in ULEB128, but in practice must be no more than 8
154:   // bytes (excluding null terminator, which is not included).
155:   uint64_t result = 0;
156:   for (unsigned char c : RegisterName)
157:     result = (result << 8) | c;
158:   return result;
159: }
160:
```
- EN: This range implements operational logic in helpers such as getOperand, NVPTXRegisterInfo::getFrameRegister, getTarget, NVPTXRegisterInfo::getFrameLocalRegister, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、NVPTXRegisterInfo::getFrameRegister、getTarget、NVPTXRegisterInfo::getFrameLocalRegister 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-186
```cpp
161: void NVPTXRegisterInfo::addToDebugRegisterMap(
162:     uint64_t preEncodedVirtualRegister, StringRef RegisterName) const {
163:   uint64_t mapped = encodeRegisterForDwarf(RegisterName);
164:   if (mapped == 0)
165:     return;
166:   debugRegisterMap.insert({preEncodedVirtualRegister, mapped});
167: }
168:
169: int64_t NVPTXRegisterInfo::getDwarfRegNum(MCRegister RegNum, bool isEH) const {
170:   StringRef Name = NVPTXInstPrinter::getRegisterName(RegNum.id());
171:   // In NVPTXFrameLowering.cpp, we do arrange for %Depot to be accessible from
172:   // %SP. Using the %Depot register doesn't provide any debug info in
173:   // cuda-gdb, but switching it to %SP does.
174:   if (RegNum.id() == NVPTX::VRDepot)
175:     Name = "%SP";
176:   return encodeRegisterForDwarf(Name);
177: }
178:
179: int64_t NVPTXRegisterInfo::getDwarfRegNumForVirtReg(Register RegNum,
180:                                                     bool isEH) const {
181:   assert(RegNum.isVirtual());
182:   uint64_t lookup = debugRegisterMap.lookup(RegNum.id());
183:   if (lookup)
184:     return lookup;
185:   return -1;
186: }
```
- EN: This range implements operational logic in helpers such as encodeRegisterForDwarf, insert, NVPTXRegisterInfo::getDwarfRegNum, NVPTXInstPrinter::getRegisterName, translating backend policy into executable code.
- CN: 这一段实现了 encodeRegisterForDwarf、insert、NVPTXRegisterInfo::getDwarfRegNum、NVPTXInstPrinter::getRegisterName 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Register information tracks physical/register-bank constraints and target-specific register properties.
  - CN: 寄存器信息负责描述物理寄存器或寄存器银行约束，以及目标特定属性。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include getNVPTXRegClassName, getNVPTXRegClassStr, NVPTXRegisterInfo::NVPTXRegisterInfo, NVPTXGenRegisterInfo, NVPTXRegisterInfo::getCalleeSavedRegs, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getNVPTXRegClassName, getNVPTXRegClassStr, NVPTXRegisterInfo::NVPTXRegisterInfo, NVPTXGenRegisterInfo, NVPTXRegisterInfo::getCalleeSavedRegs，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXRegisterInfo.h`
  - `MCTargetDesc/NVPTXInstPrinter.h`
  - `NVPTX.h`
  - `NVPTXTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/BitVector.h`
  - `llvm/CodeGen/MachineFrameInfo.h`
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/TargetInstrInfo.h`
- System/standard headers / 系统或标准头文件:
  - `NVPTXGenRegisterInfo.inc`
