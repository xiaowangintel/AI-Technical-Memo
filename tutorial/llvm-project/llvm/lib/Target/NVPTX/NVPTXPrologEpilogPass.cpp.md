# NVPTXPrologEpilogPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXPrologEpilogPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file is a copy of the generic LLVM PrologEpilogInserter pass, modified to remove unneeded functionality and to handle virtual registers.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXPrologEpilogPass.cpp - NVPTX prolog/epilog inserter ----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file is a copy of the generic LLVM PrologEpilogInserter pass, modified
10: // to remove unneeded functionality and to handle virtual registers. Most code
11: // here is a copy of PrologEpilogInserter.cpp.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #include "NVPTX.h"
16: #include "llvm/CodeGen/MachineFrameInfo.h"
17: #include "llvm/CodeGen/MachineFunction.h"
18: #include "llvm/CodeGen/MachineFunctionPass.h"
19: #include "llvm/CodeGen/TargetFrameLowering.h"
20: #include "llvm/CodeGen/TargetRegisterInfo.h"
21: #include "llvm/CodeGen/TargetSubtargetInfo.h"
22: #include "llvm/IR/DebugInfoMetadata.h"
23: #include "llvm/Pass.h"
24: #include "llvm/Support/Debug.h"
25: #include "llvm/Support/raw_ostream.h"
26:
27: using namespace llvm;
28:
29: #define DEBUG_TYPE "nvptx-prolog-epilog"
30:
31: namespace {
32: class NVPTXPrologEpilogPass : public MachineFunctionPass {
33: public:
34:   static char ID;
35:   NVPTXPrologEpilogPass() : MachineFunctionPass(ID) {}
36:
37:   bool runOnMachineFunction(MachineFunction &MF) override;
38:
39:   StringRef getPassName() const override { return "NVPTX Prolog Epilog Pass"; }
40:
```
- EN: This range defines or declares important types such as NVPTXPrologEpilogPass, runOnMachineFunction, getPassName, shaping the data model used by NVPTXPrologEpilogPass.cpp.
- CN: 这一段定义或声明了 NVPTXPrologEpilogPass、runOnMachineFunction、getPassName 等关键类型，构成 NVPTXPrologEpilogPass.cpp 使用的数据模型。

### Lines 41-80
```cpp
41: private:
42:   void calculateFrameObjectOffsets(MachineFunction &Fn);
43: };
44: } // end anonymous namespace
45:
46: MachineFunctionPass *llvm::createNVPTXPrologEpilogPass() {
47:   return new NVPTXPrologEpilogPass();
48: }
49:
50: char NVPTXPrologEpilogPass::ID = 0;
51:
52: INITIALIZE_PASS(NVPTXPrologEpilogPass, DEBUG_TYPE,
53:                 "NVPTX Prologue/Epilogue Insertion", false, false)
54:
55: static bool replaceFrameIndexDebugInstr(MachineFunction &MF, MachineInstr &MI,
56:                                         unsigned OpIdx) {
57:   const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
58:   const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
59:   if (MI.isDebugValue()) {
60:
61:     MachineOperand &Op = MI.getOperand(OpIdx);
62:     assert(MI.isDebugOperand(&Op) &&
63:            "Frame indices can only appear as a debug operand in a DBG_VALUE*"
64:            " machine instruction");
65:     Register Reg;
66:     unsigned FrameIdx = Op.getIndex();
67:
68:     StackOffset Offset = TFI->getFrameIndexReference(MF, FrameIdx, Reg);
69:     Op.ChangeToRegister(Reg, false /*isDef*/);
70:
71:     const DIExpression *DIExpr = MI.getDebugExpression();
72:     if (MI.isNonListDebugValue()) {
73:       DIExpr = TRI.prependOffsetExpression(MI.getDebugExpression(),
74:                                            DIExpression::ApplyOffset, Offset);
75:     } else {
76:       // The debug operand at DebugOpIndex was a frame index at offset
77:       // `Offset`; now the operand has been replaced with the frame
78:       // register, we must add Offset with `register x, plus Offset`.
79:       unsigned DebugOpIndex = MI.getDebugOperandIndex(&Op);
80:       SmallVector<uint64_t, 3> Ops;
```
- EN: This range implements operational logic in helpers such as calculateFrameObjectOffsets, llvm::createNVPTXPrologEpilogPass, NVPTXPrologEpilogPass, getSubtarget, translating backend policy into executable code.
- CN: 这一段实现了 calculateFrameObjectOffsets、llvm::createNVPTXPrologEpilogPass、NVPTXPrologEpilogPass、getSubtarget 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:       TRI.getOffsetOpcodes(Offset, Ops);
 82:       DIExpr = DIExpression::appendOpsToArg(DIExpr, Ops, DebugOpIndex);
 83:     }
 84:     MI.getDebugExpressionOp().setMetadata(DIExpr);
 85:     return true;
 86:   }
 87:   return false;
 88: }
 89:
 90: bool NVPTXPrologEpilogPass::runOnMachineFunction(MachineFunction &MF) {
 91:   const TargetSubtargetInfo &STI = MF.getSubtarget();
 92:   const TargetFrameLowering &TFI = *STI.getFrameLowering();
 93:   const TargetRegisterInfo &TRI = *STI.getRegisterInfo();
 94:   bool Modified = false;
 95:
 96:   calculateFrameObjectOffsets(MF);
 97:
 98:   for (MachineBasicBlock &BB : MF) {
 99:     for (MachineBasicBlock::iterator I = BB.end(); I != BB.begin();) {
100:       MachineInstr &MI = *std::prev(I);
101:
102:       bool RemovedMI = false;
103:       for (const auto &[Idx, Op] : enumerate(MI.operands())) {
104:         if (!Op.isFI())
105:           continue;
106:
107:         if (replaceFrameIndexDebugInstr(MF, MI, Idx))
108:           continue;
109:
110:         // Eliminate this FrameIndex operand.
111:         RemovedMI = TRI.eliminateFrameIndex(MI, 0, Idx, nullptr);
112:         Modified = true;
113:         if (RemovedMI)
114:           break;
115:       }
116:
117:       if (!RemovedMI)
118:         --I;
119:     }
120:   }
```
- EN: This range implements operational logic in helpers such as getOffsetOpcodes, DIExpression::appendOpsToArg, getDebugExpressionOp, NVPTXPrologEpilogPass::runOnMachineFunction, translating backend policy into executable code.
- CN: 这一段实现了 getOffsetOpcodes、DIExpression::appendOpsToArg、getDebugExpressionOp、NVPTXPrologEpilogPass::runOnMachineFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:
122:   // Add function prolog/epilog
123:   TFI.emitPrologue(MF, MF.front());
124:
125:   for (MachineFunction::iterator I = MF.begin(), E = MF.end(); I != E; ++I) {
126:     // If last instruction is a return instruction, add an epilogue
127:     if (I->isReturnBlock())
128:       TFI.emitEpilogue(MF, *I);
129:   }
130:
131:   return Modified;
132: }
133:
134: /// AdjustStackOffset - Helper function used to adjust the stack frame offset.
135: static inline void AdjustStackOffset(MachineFrameInfo &MFI, int FrameIdx,
136:                                      bool StackGrowsDown, int64_t &Offset,
137:                                      Align &MaxAlign) {
138:   // If the stack grows down, add the object size to find the lowest address.
139:   if (StackGrowsDown)
140:     Offset += MFI.getObjectSize(FrameIdx);
141:
142:   Align Alignment = MFI.getObjectAlign(FrameIdx);
143:
144:   // If the alignment of this object is greater than that of the stack, then
145:   // increase the stack alignment to match.
146:   MaxAlign = std::max(MaxAlign, Alignment);
147:
148:   // Adjust to alignment boundary.
149:   Offset = alignTo(Offset, Alignment);
150:
151:   if (StackGrowsDown) {
152:     LLVM_DEBUG(dbgs() << "alloc FI(" << FrameIdx << ") at SP[" << -Offset
153:                       << "]\n");
154:     MFI.setObjectOffset(FrameIdx, -Offset); // Set the computed offset
155:   } else {
156:     LLVM_DEBUG(dbgs() << "alloc FI(" << FrameIdx << ") at SP[" << Offset
157:                       << "]\n");
158:     MFI.setObjectOffset(FrameIdx, Offset);
159:     Offset += MFI.getObjectSize(FrameIdx);
160:   }
```
- EN: This range implements operational logic in helpers such as emitPrologue, emitEpilogue, getObjectSize, getObjectAlign, translating backend policy into executable code.
- CN: 这一段实现了 emitPrologue、emitEpilogue、getObjectSize、getObjectAlign 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161: }
162:
163: void
164: NVPTXPrologEpilogPass::calculateFrameObjectOffsets(MachineFunction &Fn) {
165:   const TargetFrameLowering &TFI = *Fn.getSubtarget().getFrameLowering();
166:   const TargetRegisterInfo *RegInfo = Fn.getSubtarget().getRegisterInfo();
167:
168:   bool StackGrowsDown =
169:     TFI.getStackGrowthDirection() == TargetFrameLowering::StackGrowsDown;
170:
171:   // Loop over all of the stack objects, assigning sequential addresses...
172:   MachineFrameInfo &MFI = Fn.getFrameInfo();
173:
174:   // Start at the beginning of the local area.
175:   // The Offset is the distance from the stack top in the direction
176:   // of stack growth -- so it's always nonnegative.
177:   int LocalAreaOffset = TFI.getOffsetOfLocalArea();
178:   if (StackGrowsDown)
179:     LocalAreaOffset = -LocalAreaOffset;
180:   assert(LocalAreaOffset >= 0
181:          && "Local area offset should be in direction of stack growth");
182:   int64_t Offset = LocalAreaOffset;
183:
184:   // If there are fixed sized objects that are preallocated in the local area,
185:   // non-fixed objects can't be allocated right at the start of local area.
186:   // We currently don't support filling in holes in between fixed sized
187:   // objects, so we adjust 'Offset' to point to the end of last fixed sized
188:   // preallocated object.
189:   for (int i = MFI.getObjectIndexBegin(); i != 0; ++i) {
190:     int64_t FixedOff;
191:     if (StackGrowsDown) {
192:       // The maximum distance from the stack pointer is at lower address of
193:       // the object -- which is given by offset. For down growing stack
194:       // the offset is negative, so we negate the offset to get the distance.
195:       FixedOff = -MFI.getObjectOffset(i);
196:     } else {
197:       // The maximum distance from the start pointer is at the upper
198:       // address of the object.
199:       FixedOff = MFI.getObjectOffset(i) + MFI.getObjectSize(i);
200:     }
```
- EN: This range implements operational logic in helpers such as NVPTXPrologEpilogPass::calculateFrameObjectOffsets, getSubtarget, getFrameInfo, getOffsetOfLocalArea, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXPrologEpilogPass::calculateFrameObjectOffsets、getSubtarget、getFrameInfo、getOffsetOfLocalArea 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:     if (FixedOff > Offset) Offset = FixedOff;
202:   }
203:
204:   // NOTE: We do not have a call stack
205:
206:   Align MaxAlign = MFI.getMaxAlign();
207:
208:   // No scavenger
209:
210:   // FIXME: Once this is working, then enable flag will change to a target
211:   // check for whether the frame is large enough to want to use virtual
212:   // frame index registers. Functions which don't want/need this optimization
213:   // will continue to use the existing code path.
214:   if (MFI.getUseLocalStackAllocationBlock()) {
215:     Align Alignment = MFI.getLocalFrameMaxAlign();
216:
217:     // Adjust to alignment boundary.
218:     Offset = alignTo(Offset, Alignment);
219:
220:     LLVM_DEBUG(dbgs() << "Local frame base offset: " << Offset << "\n");
221:
222:     // Resolve offsets for objects in the local block.
223:     for (unsigned i = 0, e = MFI.getLocalFrameObjectCount(); i != e; ++i) {
224:       std::pair<int, int64_t> Entry = MFI.getLocalFrameObjectMap(i);
225:       int64_t FIOffset = (StackGrowsDown ? -Offset : Offset) + Entry.second;
226:       LLVM_DEBUG(dbgs() << "alloc FI(" << Entry.first << ") at SP[" << FIOffset
227:                         << "]\n");
228:       MFI.setObjectOffset(Entry.first, FIOffset);
229:     }
230:     // Allocate the local block
231:     Offset += MFI.getLocalFrameSize();
232:
233:     MaxAlign = std::max(Alignment, MaxAlign);
234:   }
235:
236:   // No stack protector
237:
238:   // Then assign frame offsets to stack objects that are not used to spill
239:   // callee saved registers.
240:   for (unsigned i = 0, e = MFI.getObjectIndexEnd(); i != e; ++i) {
```
- EN: This range implements operational logic in helpers such as getMaxAlign, getLocalFrameMaxAlign, alignTo, LLVM_DEBUG, translating backend policy into executable code.
- CN: 这一段实现了 getMaxAlign、getLocalFrameMaxAlign、alignTo、LLVM_DEBUG 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-279
```cpp
241:     if (MFI.isObjectPreAllocated(i) &&
242:         MFI.getUseLocalStackAllocationBlock())
243:       continue;
244:     if (MFI.isDeadObjectIndex(i))
245:       continue;
246:
247:     AdjustStackOffset(MFI, i, StackGrowsDown, Offset, MaxAlign);
248:   }
249:
250:   // No scavenger
251:
252:   if (!TFI.targetHandlesStackFrameRounding()) {
253:     // If we have reserved argument space for call sites in the function
254:     // immediately on entry to the current function, count it as part of the
255:     // overall stack size.
256:     if (MFI.adjustsStack() && TFI.hasReservedCallFrame(Fn))
257:       Offset += MFI.getMaxCallFrameSize();
258:
259:     // Round up the size to a multiple of the alignment.  If the function has
260:     // any calls or alloca's, align to the target's StackAlignment value to
261:     // ensure that the callee's frame or the alloca data is suitably aligned;
262:     // otherwise, for leaf functions, align to the TransientStackAlignment
263:     // value.
264:     Align StackAlign;
265:     if (MFI.adjustsStack() || MFI.hasVarSizedObjects() ||
266:         (RegInfo->hasStackRealignment(Fn) && MFI.getObjectIndexEnd() != 0))
267:       StackAlign = TFI.getStackAlign();
268:     else
269:       StackAlign = TFI.getTransientStackAlign();
270:
271:     // If the frame pointer is eliminated, all frame offsets will be relative to
272:     // SP not FP. Align to MaxAlign so this works.
273:     Offset = alignTo(Offset, std::max(StackAlign, MaxAlign));
274:   }
275:
276:   // Update frame info to pretend that this is part of the stack...
277:   int64_t StackSize = Offset - LocalAreaOffset;
278:   MFI.setStackSize(StackSize);
279: }
```
- EN: This range implements operational logic in helpers such as getUseLocalStackAllocationBlock, AdjustStackOffset, getMaxCallFrameSize, hasStackRealignment, translating backend policy into executable code.
- CN: 这一段实现了 getUseLocalStackAllocationBlock、AdjustStackOffset、getMaxCallFrameSize、hasStackRealignment 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXPrologEpilogPass, runOnMachineFunction, getPassName, calculateFrameObjectOffsets, llvm::createNVPTXPrologEpilogPass, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXPrologEpilogPass, runOnMachineFunction, getPassName, calculateFrameObjectOffsets, llvm::createNVPTXPrologEpilogPass，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineFrameInfo.h`
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/MachineFunctionPass.h`
  - `llvm/CodeGen/TargetFrameLowering.h`
  - `llvm/CodeGen/TargetRegisterInfo.h`
  - `llvm/CodeGen/TargetSubtargetInfo.h`
  - `llvm/IR/DebugInfoMetadata.h`
  - `llvm/Pass.h`
  - `llvm/Support/Debug.h`
  - `llvm/Support/raw_ostream.h`
