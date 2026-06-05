# SPIRVInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVInstrInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the SPIR-V implementation of the TargetInstrInfo class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVInstrInfo.cpp - SPIR-V Instruction Information ------*- C++-*-===//
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
13: #include "SPIRVInstrInfo.h"
14: #include "SPIRV.h"
15: #include "SPIRVSubtarget.h"
16: #include "llvm/ADT/SmallVector.h"
17: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
18: #include "llvm/CodeGen/MachineBasicBlock.h"
19: #include "llvm/IR/DebugLoc.h"
20:
21: #define GET_INSTRINFO_CTOR_DTOR
22: #include "SPIRVGenInstrInfo.inc"
23:
24: using namespace llvm;
25:
26: SPIRVInstrInfo::SPIRVInstrInfo(const SPIRVSubtarget &STI)
27:     : SPIRVGenInstrInfo(STI, RI) {}
28:
29: bool SPIRVInstrInfo::isConstantInstr(const MachineInstr &MI) const {
30:   switch (MI.getOpcode()) {
31:   case SPIRV::OpConstantTrue:
32:   case SPIRV::OpConstantFalse:
33:   case SPIRV::OpConstantI:
34:   case SPIRV::OpConstantF:
35:   case SPIRV::OpConstantComposite:
36:   case SPIRV::OpConstantCompositeContinuedINTEL:
37:   case SPIRV::OpConstantSampler:
38:   case SPIRV::OpConstantNull:
39:   case SPIRV::OpSpecConstantTrue:
40:   case SPIRV::OpSpecConstantFalse:
41:   case SPIRV::OpSpecConstant:
42:   case SPIRV::OpSpecConstantComposite:
43:   case SPIRV::OpSpecConstantCompositeContinuedINTEL:
44:   case SPIRV::OpSpecConstantOp:
45:   case SPIRV::OpUndef:
46:   case SPIRV::OpConstantFunctionPointerINTEL:
47:     return true;
48:   default:
49:     return false;
50:   }
51: }
52:
53: bool SPIRVInstrInfo::isSpecConstantInstr(const MachineInstr &MI) const {
54:   switch (MI.getOpcode()) {
55:   case SPIRV::OpSpecConstantTrue:
56:   case SPIRV::OpSpecConstantFalse:
57:   case SPIRV::OpSpecConstant:
58:   case SPIRV::OpSpecConstantComposite:
59:   case SPIRV::OpSpecConstantCompositeContinuedINTEL:
60:   case SPIRV::OpSpecConstantOp:
```
- EN: This range implements operational logic in helpers such as SPIRVInstrInfo::SPIRVInstrInfo, SPIRVGenInstrInfo, SPIRVInstrInfo::isConstantInstr, SPIRVInstrInfo::isSpecConstantInstr, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVInstrInfo::SPIRVInstrInfo、SPIRVGenInstrInfo、SPIRVInstrInfo::isConstantInstr、SPIRVInstrInfo::isSpecConstantInstr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:     return true;
 62:   default:
 63:     return false;
 64:   }
 65: }
 66:
 67: bool SPIRVInstrInfo::isInlineAsmDefInstr(const MachineInstr &MI) const {
 68:   switch (MI.getOpcode()) {
 69:   case SPIRV::OpAsmTargetINTEL:
 70:   case SPIRV::OpAsmINTEL:
 71:     return true;
 72:   default:
 73:     return false;
 74:   }
 75: }
 76:
 77: bool SPIRVInstrInfo::isTypeDeclInstr(const MachineInstr &MI) const {
 78:   auto &MRI = MI.getMF()->getRegInfo();
 79:   if (MI.getNumDefs() >= 1 && MI.getOperand(0).isReg()) {
 80:     auto DefRegClass = MRI.getRegClassOrNull(MI.getOperand(0).getReg());
 81:     return DefRegClass && DefRegClass->getID() == SPIRV::TYPERegClass.getID();
 82:   } else {
 83:     return MI.getOpcode() == SPIRV::OpTypeForwardPointer ||
 84:            MI.getOpcode() == SPIRV::OpTypeStructContinuedINTEL;
 85:   }
 86: }
 87:
 88: bool SPIRVInstrInfo::isDecorationInstr(const MachineInstr &MI) const {
 89:   switch (MI.getOpcode()) {
 90:   case SPIRV::OpDecorate:
 91:   case SPIRV::OpDecorateId:
 92:   case SPIRV::OpDecorateString:
 93:   case SPIRV::OpMemberDecorate:
 94:   case SPIRV::OpMemberDecorateString:
 95:     return true;
 96:   default:
 97:     return false;
 98:   }
 99: }
100:
101: bool SPIRVInstrInfo::isAliasingInstr(const MachineInstr &MI) const {
102:   switch (MI.getOpcode()) {
103:   case SPIRV::OpAliasDomainDeclINTEL:
104:   case SPIRV::OpAliasScopeDeclINTEL:
105:   case SPIRV::OpAliasScopeListDeclINTEL:
106:     return true;
107:   default:
108:     return false;
109:   }
110: }
111:
112: bool SPIRVInstrInfo::isHeaderInstr(const MachineInstr &MI) const {
113:   switch (MI.getOpcode()) {
114:   case SPIRV::OpCapability:
115:   case SPIRV::OpExtension:
116:   case SPIRV::OpExtInstImport:
117:   case SPIRV::OpMemoryModel:
118:   case SPIRV::OpEntryPoint:
119:   case SPIRV::OpExecutionMode:
120:   case SPIRV::OpExecutionModeId:
```
- EN: This range implements operational logic in helpers such as SPIRVInstrInfo::isInlineAsmDefInstr, SPIRVInstrInfo::isTypeDeclInstr, getMF, getRegClassOrNull, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVInstrInfo::isInlineAsmDefInstr、SPIRVInstrInfo::isTypeDeclInstr、getMF、getRegClassOrNull 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   case SPIRV::OpString:
122:   case SPIRV::OpSourceExtension:
123:   case SPIRV::OpSource:
124:   case SPIRV::OpSourceContinued:
125:   case SPIRV::OpName:
126:   case SPIRV::OpMemberName:
127:   case SPIRV::OpModuleProcessed:
128:     return true;
129:   default:
130:     return isTypeDeclInstr(MI) || isConstantInstr(MI) ||
131:            isDecorationInstr(MI) || isAliasingInstr(MI);
132:   }
133: }
134:
135: bool SPIRVInstrInfo::canUseFastMathFlags(const MachineInstr &MI,
136:                                          bool KHRFloatControls2) const {
137:   switch (MI.getOpcode()) {
138:   case SPIRV::OpFAddS:
139:   case SPIRV::OpFSubS:
140:   case SPIRV::OpFMulS:
141:   case SPIRV::OpFDivS:
142:   case SPIRV::OpFRemS:
143:   case SPIRV::OpFAddV:
144:   case SPIRV::OpFSubV:
145:   case SPIRV::OpFMulV:
146:   case SPIRV::OpFDivV:
147:   case SPIRV::OpFRemV:
148:   case SPIRV::OpFMod:
149:     return true;
150:   case SPIRV::OpFNegateV:
151:   case SPIRV::OpFNegate:
152:   case SPIRV::OpOrdered:
153:   case SPIRV::OpUnordered:
154:   case SPIRV::OpFOrdEqual:
155:   case SPIRV::OpFOrdNotEqual:
156:   case SPIRV::OpFOrdLessThan:
157:   case SPIRV::OpFOrdLessThanEqual:
158:   case SPIRV::OpFOrdGreaterThan:
159:   case SPIRV::OpFOrdGreaterThanEqual:
160:   case SPIRV::OpFUnordEqual:
161:   case SPIRV::OpFUnordNotEqual:
162:   case SPIRV::OpFUnordLessThan:
163:   case SPIRV::OpFUnordLessThanEqual:
164:   case SPIRV::OpFUnordGreaterThan:
165:   case SPIRV::OpFUnordGreaterThanEqual:
166:   case SPIRV::OpExtInst:
167:     return KHRFloatControls2 ? true : false;
168:   default:
169:     return false;
170:   }
171: }
172:
173: bool SPIRVInstrInfo::canUseNSW(const MachineInstr &MI) const {
174:   switch (MI.getOpcode()) {
175:   case SPIRV::OpIAddS:
176:   case SPIRV::OpIAddV:
177:   case SPIRV::OpISubS:
178:   case SPIRV::OpISubV:
179:   case SPIRV::OpIMulS:
180:   case SPIRV::OpIMulV:
```
- EN: This range implements operational logic in helpers such as isDecorationInstr, SPIRVInstrInfo::canUseNSW, translating backend policy into executable code.
- CN: 这一段实现了 isDecorationInstr、SPIRVInstrInfo::canUseNSW 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   case SPIRV::OpShiftLeftLogicalS:
182:   case SPIRV::OpShiftLeftLogicalV:
183:   case SPIRV::OpSNegate:
184:     return true;
185:   default:
186:     return false;
187:   }
188: }
189:
190: bool SPIRVInstrInfo::canUseNUW(const MachineInstr &MI) const {
191:   switch (MI.getOpcode()) {
192:   case SPIRV::OpIAddS:
193:   case SPIRV::OpIAddV:
194:   case SPIRV::OpISubS:
195:   case SPIRV::OpISubV:
196:   case SPIRV::OpIMulS:
197:   case SPIRV::OpIMulV:
198:     return true;
199:   default:
200:     return false;
201:   }
202: }
203:
204: // Analyze the branching code at the end of MBB, returning
205: // true if it cannot be understood (e.g. it's a switch dispatch or isn't
206: // implemented for a target).  Upon success, this returns false and returns
207: // with the following information in various cases:
208: //
209: // 1. If this block ends with no branches (it just falls through to its succ)
210: //    just return false, leaving TBB/FBB null.
211: // 2. If this block ends with only an unconditional branch, it sets TBB to be
212: //    the destination block.
213: // 3. If this block ends with a conditional branch and it falls through to a
214: //    successor block, it sets TBB to be the branch destination block and a
215: //    list of operands that evaluate the condition. These operands can be
216: //    passed to other TargetInstrInfo methods to create new branches.
217: // 4. If this block ends with a conditional branch followed by an
218: //    unconditional branch, it returns the 'true' destination in TBB, the
219: //    'false' destination in FBB, and a list of operands that evaluate the
220: //    condition.  These operands can be passed to other TargetInstrInfo
221: //    methods to create new branches.
222: //
223: // Note that removeBranch and insertBranch must be implemented to support
224: // cases where this method returns success.
225: //
226: // If AllowModify is true, then this routine is allowed to modify the basic
227: // block (e.g. delete instructions after the unconditional branch).
228: //
229: // The CFG information in MBB.Predecessors and MBB.Successors must be valid
230: // before calling this function.
231: bool SPIRVInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
232:                                    MachineBasicBlock *&TBB,
233:                                    MachineBasicBlock *&FBB,
234:                                    SmallVectorImpl<MachineOperand> &Cond,
235:                                    bool AllowModify) const {
236:   // We do not allow to restructure blocks by results of analyzeBranch(),
237:   // because it may potentially break structured control flow and anyway
238:   // doubtedly may be useful in SPIRV, including such reasons as, e.g.:
239:   // 1) there is no way to encode `if (Cond) then Stmt` logic, only full
240:   // if-then-else is supported by OpBranchConditional, so if we supported
```
- EN: This range implements operational logic in helpers such as SPIRVInstrInfo::canUseNUW, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVInstrInfo::canUseNUW 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:   // splitting of blocks ending with OpBranchConditional MachineBasicBlock.cpp
242:   // would expect successfull implementation of calls to insertBranch() setting
243:   // FBB to null that is not feasible; 2) it's not possible to delete
244:   // instructions after the unconditional branch, because this instruction must
245:   // be the last instruction in a block.
246:   return true;
247: }
248:
249: // Remove the branching code at the end of the specific MBB.
250: // This is only invoked in cases where analyzeBranch returns success. It
251: // returns the number of instructions that were removed.
252: // If \p BytesRemoved is non-null, report the change in code size from the
253: // removed instructions.
254: unsigned SPIRVInstrInfo::removeBranch(MachineBasicBlock &MBB,
255:                                       int * /*BytesRemoved*/) const {
256:   MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
257:   if (I == MBB.end())
258:     return 0;
259:
260:   if (I->getOpcode() == SPIRV::OpBranch) {
261:     I->eraseFromParent();
262:     return 1;
263:   }
264:   return 0;
265: }
266:
267: // Insert branch code into the end of the specified MachineBasicBlock. The
268: // operands to this method are the same as those returned by analyzeBranch.
269: // This is only invoked in cases where analyzeBranch returns success. It
270: // returns the number of instructions inserted. If \p BytesAdded is non-null,
271: // report the change in code size from the added instructions.
272: //
273: // It is also invoked by tail merging to add unconditional branches in
274: // cases where analyzeBranch doesn't apply because there was no original
275: // branch to analyze.  At least this much must be implemented, else tail
276: // merging needs to be disabled.
277: //
278: // The CFG information in MBB.Predecessors and MBB.Successors must be valid
279: // before calling this function.
280: unsigned SPIRVInstrInfo::insertBranch(MachineBasicBlock &MBB,
281:                                       MachineBasicBlock *TBB,
282:                                       MachineBasicBlock *FBB,
283:                                       ArrayRef<MachineOperand> Cond,
284:                                       const DebugLoc &DL,
285:                                       int * /*BytesAdded*/) const {
286:   if (!TBB)
287:     return 0;
288:   BuildMI(&MBB, DL, get(SPIRV::OpBranch)).addMBB(TBB);
289:   return 1;
290: }
291:
292: void SPIRVInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
293:                                  MachineBasicBlock::iterator I,
294:                                  const DebugLoc &DL, Register DestReg,
295:                                  Register SrcReg, bool KillSrc,
296:                                  bool RenamableDest, bool RenamableSrc) const {
297:   // Actually we don't need this COPY instruction. However if we do nothing with
298:   // it, post RA pseudo instrs expansion just removes it and we get the code
299:   // with undef registers. Therefore, we need to replace all uses of dst with
300:   // the src register. COPY instr itself will be safely removed later.
```
- EN: This range implements operational logic in helpers such as getLastNonDebugInstr, eraseFromParent, BuildMI, translating backend policy into executable code.
- CN: 这一段实现了 getLastNonDebugInstr、eraseFromParent、BuildMI 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-308
```cpp
301:   assert(I->isCopy() && "Copy instruction is expected");
302:   auto DstOp = I->getOperand(0);
303:   auto SrcOp = I->getOperand(1);
304:   assert(DstOp.isReg() && SrcOp.isReg() &&
305:          "Register operands are expected in COPY");
306:   auto &MRI = I->getMF()->getRegInfo();
307:   MRI.replaceRegWith(DstOp.getReg(), SrcOp.getReg());
308: }
```
- EN: This range implements operational logic in helpers such as assert, getOperand, getMF, replaceRegWith, translating backend policy into executable code.
- CN: 这一段实现了 assert、getOperand、getMF、replaceRegWith 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Instruction info files centralize opcode semantics, scheduling hooks, and target-specific machine properties.
  - CN: 指令信息文件集中描述操作码语义、调度钩子以及目标相关机器属性。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVInstrInfo::SPIRVInstrInfo, SPIRVGenInstrInfo, SPIRVInstrInfo::isConstantInstr, SPIRVInstrInfo::isSpecConstantInstr, SPIRVInstrInfo::isInlineAsmDefInstr, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVInstrInfo::SPIRVInstrInfo, SPIRVGenInstrInfo, SPIRVInstrInfo::isConstantInstr, SPIRVInstrInfo::isSpecConstantInstr, SPIRVInstrInfo::isInlineAsmDefInstr，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVInstrInfo.h`
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
  - `llvm/CodeGen/MachineBasicBlock.h`
  - `llvm/IR/DebugLoc.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenInstrInfo.inc`
