# NVPTXInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXInstrInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the NVPTX implementation of the TargetInstrInfo class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVPTXInstrInfo.cpp - NVPTX Instruction Information -----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the NVPTX implementation of the TargetInstrInfo class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXInstrInfo.h"
14: #include "NVPTX.h"
15: #include "NVPTXSubtarget.h"
16: #include "llvm/CodeGen/MachineFunction.h"
17: #include "llvm/CodeGen/MachineInstrBuilder.h"
18: #include "llvm/CodeGen/MachineRegisterInfo.h"
19:
20: using namespace llvm;
21:
22: #define GET_INSTRINFO_CTOR_DTOR
23: #include "NVPTXGenInstrInfo.inc"
24:
25: // Pin the vtable to this file.
26: void NVPTXInstrInfo::anchor() {}
27:
28: NVPTXInstrInfo::NVPTXInstrInfo(const NVPTXSubtarget &STI)
29:     : NVPTXGenInstrInfo(STI, RegInfo), RegInfo() {}
30:
31: void NVPTXInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
32:                                  MachineBasicBlock::iterator I,
33:                                  const DebugLoc &DL, Register DestReg,
34:                                  Register SrcReg, bool KillSrc,
35:                                  bool RenamableDest, bool RenamableSrc) const {
36:   const MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
37:   const TargetRegisterClass *DestRC = MRI.getRegClass(DestReg);
38:   const TargetRegisterClass *SrcRC = MRI.getRegClass(SrcReg);
39:
40:   if (DestRC != SrcRC)
```
- EN: This range implements operational logic in helpers such as NVPTXInstrInfo::anchor, NVPTXInstrInfo::NVPTXInstrInfo, NVPTXGenInstrInfo, getParent, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXInstrInfo::anchor、NVPTXInstrInfo::NVPTXInstrInfo、NVPTXGenInstrInfo、getParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:     report_fatal_error("Copy one register into another with a different width");
42:
43:   unsigned Op;
44:   if (DestRC == &NVPTX::B1RegClass)
45:     Op = NVPTX::MOV_B1_r;
46:   else if (DestRC == &NVPTX::B16RegClass)
47:     Op = NVPTX::MOV_B16_r;
48:   else if (DestRC == &NVPTX::B32RegClass)
49:     Op = NVPTX::MOV_B32_r;
50:   else if (DestRC == &NVPTX::B64RegClass)
51:     Op = NVPTX::MOV_B64_r;
52:   else if (DestRC == &NVPTX::B128RegClass)
53:     Op = NVPTX::MOV_B128_r;
54:   else
55:     llvm_unreachable("Bad register copy");
56:
57:   BuildMI(MBB, I, DL, get(Op), DestReg)
58:       .addReg(SrcReg, getKillRegState(KillSrc));
59: }
60:
61: /// analyzeBranch - Analyze the branching code at the end of MBB, returning
62: /// true if it cannot be understood (e.g. it's a switch dispatch or isn't
63: /// implemented for a target).  Upon success, this returns false and returns
64: /// with the following information in various cases:
65: ///
66: /// 1. If this block ends with no branches (it just falls through to its succ)
67: ///    just return false, leaving TBB/FBB null.
68: /// 2. If this block ends with only an unconditional branch, it sets TBB to be
69: ///    the destination block.
70: /// 3. If this block ends with an conditional branch and it falls through to
71: ///    an successor block, it sets TBB to be the branch destination block and a
72: ///    list of operands that evaluate the condition. These
73: ///    operands can be passed to other TargetInstrInfo methods to create new
74: ///    branches.
75: /// 4. If this block ends with an conditional branch and an unconditional
76: ///    block, it returns the 'true' destination in TBB, the 'false' destination
77: ///    in FBB, and a list of operands that evaluate the condition. These
78: ///    operands can be passed to other TargetInstrInfo methods to create new
79: ///    branches.
80: ///
```
- EN: This range implements operational logic in helpers such as report_fatal_error, llvm_unreachable, BuildMI, addReg, translating backend policy into executable code.
- CN: 这一段实现了 report_fatal_error、llvm_unreachable、BuildMI、addReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81: /// Note that removeBranch and insertBranch must be implemented to support
 82: /// cases where this method returns success.
 83: ///
 84: bool NVPTXInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 85:                                    MachineBasicBlock *&TBB,
 86:                                    MachineBasicBlock *&FBB,
 87:                                    SmallVectorImpl<MachineOperand> &Cond,
 88:                                    bool AllowModify) const {
 89:   // If the block has no terminators, it just falls into the block after it.
 90:   MachineBasicBlock::iterator I = MBB.end();
 91:   if (I == MBB.begin() || !isUnpredicatedTerminator(*--I))
 92:     return false;
 93:
 94:   // Get the last instruction in the block.
 95:   MachineInstr &LastInst = *I;
 96:
 97:   // If there is only one terminator instruction, process it.
 98:   if (I == MBB.begin() || !isUnpredicatedTerminator(*--I)) {
 99:     if (LastInst.getOpcode() == NVPTX::GOTO) {
100:       TBB = LastInst.getOperand(0).getMBB();
101:       return false;
102:     } else if (LastInst.getOpcode() == NVPTX::CBranch) {
103:       // Block ends with fall-through condbranch.
104:       TBB = LastInst.getOperand(1).getMBB();
105:       Cond.push_back(LastInst.getOperand(0));
106:       Cond.push_back(LastInst.getOperand(2));
107:       return false;
108:     }
109:     // Otherwise, don't know what this is.
110:     return true;
111:   }
112:
113:   // Get the instruction before it if it's a terminator.
114:   MachineInstr &SecondLastInst = *I;
115:
116:   // If there are three terminators, we don't know what sort of block this is.
117:   if (I != MBB.begin() && isUnpredicatedTerminator(*--I))
118:     return true;
119:
120:   // If the block ends with NVPTX::GOTO and NVPTX:CBranch, handle it.
```
- EN: This range implements operational logic in helpers such as end, getOperand, push_back, translating backend policy into executable code.
- CN: 这一段实现了 end、getOperand、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:   if (SecondLastInst.getOpcode() == NVPTX::CBranch &&
122:       LastInst.getOpcode() == NVPTX::GOTO) {
123:     TBB = SecondLastInst.getOperand(1).getMBB();
124:     Cond.push_back(SecondLastInst.getOperand(0));
125:     Cond.push_back(SecondLastInst.getOperand(2));
126:     FBB = LastInst.getOperand(0).getMBB();
127:     return false;
128:   }
129:
130:   // If the block ends with two NVPTX:GOTOs, handle it.  The second one is not
131:   // executed, so remove it.
132:   if (SecondLastInst.getOpcode() == NVPTX::GOTO &&
133:       LastInst.getOpcode() == NVPTX::GOTO) {
134:     TBB = SecondLastInst.getOperand(0).getMBB();
135:     I = LastInst;
136:     if (AllowModify)
137:       I->eraseFromParent();
138:     return false;
139:   }
140:
141:   // Otherwise, can't handle this.
142:   return true;
143: }
144:
145: unsigned NVPTXInstrInfo::removeBranch(MachineBasicBlock &MBB,
146:                                       int *BytesRemoved) const {
147:   assert(!BytesRemoved && "code size not handled");
148:   MachineBasicBlock::iterator I = MBB.end();
149:   if (I == MBB.begin())
150:     return 0;
151:   --I;
152:   if (I->getOpcode() != NVPTX::GOTO && I->getOpcode() != NVPTX::CBranch)
153:     return 0;
154:
155:   // Remove the branch.
156:   I->eraseFromParent();
157:
158:   I = MBB.end();
159:
160:   if (I == MBB.begin())
```
- EN: This range implements operational logic in helpers such as getOpcode, getOperand, push_back, eraseFromParent, translating backend policy into executable code.
- CN: 这一段实现了 getOpcode、getOperand、push_back、eraseFromParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:     return 1;
162:   --I;
163:   if (I->getOpcode() != NVPTX::CBranch)
164:     return 1;
165:
166:   // Remove the branch.
167:   I->eraseFromParent();
168:   return 2;
169: }
170:
171: unsigned NVPTXInstrInfo::insertBranch(MachineBasicBlock &MBB,
172:                                       MachineBasicBlock *TBB,
173:                                       MachineBasicBlock *FBB,
174:                                       ArrayRef<MachineOperand> Cond,
175:                                       const DebugLoc &DL,
176:                                       int *BytesAdded) const {
177:   assert(!BytesAdded && "code size not handled");
178:
179:   // Shouldn't be a fall through.
180:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
181:   assert((Cond.size() == 2 || Cond.size() == 0) &&
182:          "NVPTX branch conditions have two components!");
183:
184:   // One-way branch.
185:   if (!FBB) {
186:     if (Cond.empty()) // Unconditional branch
187:       BuildMI(&MBB, DL, get(NVPTX::GOTO)).addMBB(TBB);
188:     else // Conditional branch
189:       BuildMI(&MBB, DL, get(NVPTX::CBranch))
190:           .add(Cond[0])
191:           .addMBB(TBB)
192:           .add(Cond[1]);
193:     return 1;
194:   }
195:
196:   // Two-way Conditional Branch.
197:   BuildMI(&MBB, DL, get(NVPTX::CBranch)).add(Cond[0]).addMBB(TBB).add(Cond[1]);
198:   BuildMI(&MBB, DL, get(NVPTX::GOTO)).addMBB(FBB);
199:   return 2;
200: }
```
- EN: This range implements operational logic in helpers such as eraseFromParent, assert, BuildMI, add, translating backend policy into executable code.
- CN: 这一段实现了 eraseFromParent、assert、BuildMI、add 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-207
```cpp
201:
202: bool NVPTXInstrInfo::reverseBranchCondition(
203:     SmallVectorImpl<MachineOperand> &Cond) const {
204:   assert(Cond.size() == 2 && "Invalid NVPTX branch condition!");
205:   Cond[1].setImm(!Cond[1].getImm());
206:   return false;
207: }
```
- EN: This range implements operational logic in helpers such as assert, setImm, translating backend policy into executable code.
- CN: 这一段实现了 assert、setImm 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Instruction info files centralize opcode semantics, scheduling hooks, and target-specific machine properties.
  - CN: 指令信息文件集中描述操作码语义、调度钩子以及目标相关机器属性。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXInstrInfo::anchor, NVPTXInstrInfo::NVPTXInstrInfo, NVPTXGenInstrInfo, getParent, getRegClass, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXInstrInfo::anchor, NVPTXInstrInfo::NVPTXInstrInfo, NVPTXGenInstrInfo, getParent, getRegClass，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXInstrInfo.h`
  - `NVPTX.h`
  - `NVPTXSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/MachineInstrBuilder.h`
  - `llvm/CodeGen/MachineRegisterInfo.h`
- System/standard headers / 系统或标准头文件:
  - `NVPTXGenInstrInfo.inc`
