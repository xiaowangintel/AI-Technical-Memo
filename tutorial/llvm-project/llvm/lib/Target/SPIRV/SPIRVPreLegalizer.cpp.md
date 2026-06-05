# SPIRVPreLegalizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVPreLegalizer.cpp`
- Repository: `llvm-project`
- Purpose (EN): The pass prepares IR for legalization: it assigns SPIR-V types to registers and removes intrinsics which holded these types during IR translation.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===-- SPIRVPreLegalizer.cpp - prepare IR for legalization -----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The pass prepares IR for legalization: it assigns SPIR-V types to registers
10: // and removes intrinsics which holded these types during IR translation.
11: // Also it processes constants and registers them in GR to avoid duplication.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #include "SPIRV.h"
16: #include "SPIRVSubtarget.h"
17: #include "SPIRVUtils.h"
18: #include "llvm/ADT/PostOrderIterator.h"
19: #include "llvm/CodeGen/GlobalISel/CSEInfo.h"
20: #include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
21: #include "llvm/IR/Attributes.h"
22: #include "llvm/IR/Constants.h"
23: #include "llvm/IR/IntrinsicsSPIRV.h"
24:
25: #define DEBUG_TYPE "spirv-prelegalizer"
26:
27: using namespace llvm;
28:
29: namespace {
30: class SPIRVPreLegalizer : public MachineFunctionPass {
31: public:
32:   static char ID;
33:   SPIRVPreLegalizer() : MachineFunctionPass(ID) {}
34:   bool runOnMachineFunction(MachineFunction &MF) override;
35:   void getAnalysisUsage(AnalysisUsage &AU) const override;
36: };
37: } // namespace
38:
39: void SPIRVPreLegalizer::getAnalysisUsage(AnalysisUsage &AU) const {
40:   AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
41:   MachineFunctionPass::getAnalysisUsage(AU);
42: }
43:
44: static inline void invalidateAndEraseMI(SPIRVGlobalRegistry *GR,
45:                                         MachineInstr *MI) {
46:   GR->invalidateMachineInstr(MI);
47:   MI->eraseFromParent();
48: }
49:
50: static void
51: addConstantsToTrack(MachineFunction &MF, SPIRVGlobalRegistry *GR,
52:                     const SPIRVSubtarget &STI,
53:                     DenseMap<MachineInstr *, Type *> &TargetExtConstTypes) {
54:   MachineRegisterInfo &MRI = MF.getRegInfo();
55:   DenseMap<MachineInstr *, Register> RegsAlreadyAddedToDT;
56:   SmallVector<MachineInstr *, 10> ToErase, ToEraseComposites;
57:   for (MachineBasicBlock &MBB : MF) {
58:     for (MachineInstr &MI : MBB) {
59:       if (!isSpvIntrinsic(MI, Intrinsic::spv_track_constant))
60:         continue;
61:       ToErase.push_back(&MI);
62:       Register SrcReg = MI.getOperand(2).getReg();
63:       auto *Const =
64:           cast<Constant>(cast<ConstantAsMetadata>(
65:                              MI.getOperand(3).getMetadata()->getOperand(0))
66:                              ->getValue());
67:       if (auto *GV = dyn_cast<GlobalValue>(Const)) {
68:         Register Reg = GR->find(GV, &MF);
69:         if (!Reg.isValid()) {
70:           GR->add(GV, MRI.getVRegDef(SrcReg));
71:           GR->addGlobalObject(GV, &MF, SrcReg);
72:         } else
73:           RegsAlreadyAddedToDT[&MI] = Reg;
74:       } else {
75:         Register Reg = GR->find(Const, &MF);
76:         if (!Reg.isValid()) {
77:           if (auto *ConstVec = dyn_cast<ConstantDataVector>(Const)) {
78:             auto *BuildVec = MRI.getVRegDef(SrcReg);
79:             assert(BuildVec &&
80:                    BuildVec->getOpcode() == TargetOpcode::G_BUILD_VECTOR);
81:             GR->add(Const, BuildVec);
82:             for (unsigned i = 0; i < ConstVec->getNumElements(); ++i) {
83:               // Ensure that OpConstantComposite reuses a constant when it's
84:               // already created and available in the same machine function.
85:               Constant *ElemConst = ConstVec->getElementAsConstant(i);
86:               Register ElemReg = GR->find(ElemConst, &MF);
87:               if (!ElemReg.isValid())
88:                 GR->add(ElemConst,
89:                         MRI.getVRegDef(BuildVec->getOperand(1 + i).getReg()));
90:               else
```
- EN: This range defines or declares important types such as SPIRVPreLegalizer, runOnMachineFunction, getAnalysisUsage, SPIRVPreLegalizer::getAnalysisUsage, shaping the data model used by SPIRVPreLegalizer.cpp.
- CN: 这一段定义或声明了 SPIRVPreLegalizer、runOnMachineFunction、getAnalysisUsage、SPIRVPreLegalizer::getAnalysisUsage 等关键类型，构成 SPIRVPreLegalizer.cpp 使用的数据模型。

### Lines 91-180
```cpp
 91:                 BuildVec->getOperand(1 + i).setReg(ElemReg);
 92:             }
 93:           }
 94:           if (Const->getType()->isTargetExtTy()) {
 95:             // remember association so that we can restore it when assign types
 96:             MachineInstr *SrcMI = MRI.getVRegDef(SrcReg);
 97:             if (SrcMI)
 98:               GR->add(Const, SrcMI);
 99:             if (SrcMI && (SrcMI->getOpcode() == TargetOpcode::G_CONSTANT ||
100:                           SrcMI->getOpcode() == TargetOpcode::G_IMPLICIT_DEF))
101:               TargetExtConstTypes[SrcMI] = Const->getType();
102:             if (Const->isNullValue()) {
103:               MachineBasicBlock &DepMBB = MF.front();
104:               MachineIRBuilder MIB(DepMBB, DepMBB.getFirstNonPHI());
105:               SPIRVTypeInst ExtType = GR->getOrCreateSPIRVType(
106:                   Const->getType(), MIB, SPIRV::AccessQualifier::ReadWrite,
107:                   true);
108:               assert(SrcMI && "Expected source instruction to be valid");
109:               SrcMI->setDesc(STI.getInstrInfo()->get(SPIRV::OpConstantNull));
110:               SrcMI->addOperand(MachineOperand::CreateReg(
111:                   GR->getSPIRVTypeID(ExtType), false));
112:             }
113:           }
114:         } else {
115:           RegsAlreadyAddedToDT[&MI] = Reg;
116:           // This MI is unused and will be removed. If the MI uses
117:           // const_composite, it will be unused and should be removed too.
118:           assert(MI.getOperand(2).isReg() && "Reg operand is expected");
119:           MachineInstr *SrcMI = MRI.getVRegDef(MI.getOperand(2).getReg());
120:           if (SrcMI && isSpvIntrinsic(*SrcMI, Intrinsic::spv_const_composite))
121:             ToEraseComposites.push_back(SrcMI);
122:         }
123:       }
124:     }
125:   }
126:   for (MachineInstr *MI : ToErase) {
127:     Register Reg = MI->getOperand(2).getReg();
128:     auto It = RegsAlreadyAddedToDT.find(MI);
129:     if (It != RegsAlreadyAddedToDT.end())
130:       Reg = It->second;
131:     auto *RC = MRI.getRegClassOrNull(MI->getOperand(0).getReg());
132:     if (!MRI.getRegClassOrNull(Reg) && RC)
133:       MRI.setRegClass(Reg, RC);
134:     MRI.replaceRegWith(MI->getOperand(0).getReg(), Reg);
135:     invalidateAndEraseMI(GR, MI);
136:   }
137:   for (MachineInstr *MI : ToEraseComposites)
138:     invalidateAndEraseMI(GR, MI);
139: }
140:
141: static void foldConstantsIntoIntrinsics(MachineFunction &MF,
142:                                         SPIRVGlobalRegistry *GR,
143:                                         MachineIRBuilder MIB) {
144:   SmallVector<MachineInstr *, 64> ToErase;
145:   for (MachineBasicBlock &MBB : MF) {
146:     for (MachineInstr &MI : MBB) {
147:       if (!isSpvIntrinsic(MI, Intrinsic::spv_assign_name))
148:         continue;
149:       const MDNode *MD = MI.getOperand(2).getMetadata();
150:       StringRef ValueName = cast<MDString>(MD->getOperand(0))->getString();
151:       if (ValueName.size() > 0) {
152:         MIB.setInsertPt(*MI.getParent(), MI);
153:         buildOpName(MI.getOperand(1).getReg(), ValueName, MIB);
154:       }
155:       ToErase.push_back(&MI);
156:     }
157:     for (MachineInstr *MI : ToErase)
158:       invalidateAndEraseMI(GR, MI);
159:     ToErase.clear();
160:   }
161: }
162:
163: static MachineInstr *findAssignTypeInstr(Register Reg,
164:                                          MachineRegisterInfo *MRI) {
165:   for (MachineRegisterInfo::use_instr_iterator I = MRI->use_instr_begin(Reg),
166:                                                IE = MRI->use_instr_end();
167:        I != IE; ++I) {
168:     MachineInstr *UseMI = &*I;
169:     if ((isSpvIntrinsic(*UseMI, Intrinsic::spv_assign_ptr_type) ||
170:          isSpvIntrinsic(*UseMI, Intrinsic::spv_assign_type)) &&
171:         UseMI->getOperand(1).getReg() == Reg)
172:       return UseMI;
173:   }
174:   return nullptr;
175: }
176:
177: static void buildOpBitcast(SPIRVGlobalRegistry *GR, MachineIRBuilder &MIB,
178:                            Register ResVReg, Register OpReg) {
179:   SPIRVTypeInst ResType = GR->getSPIRVTypeForVReg(ResVReg);
180:   SPIRVTypeInst OpType = GR->getSPIRVTypeForVReg(OpReg);
```
- EN: This range implements operational logic in helpers such as getOperand, getVRegDef, add, getOpcode, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getVRegDef、add、getOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:   assert(ResType && OpType && "Operand types are expected");
182:   if (!GR->isBitcastCompatible(ResType, OpType))
183:     report_fatal_error("incompatible result and operand types in a bitcast");
184:   MachineRegisterInfo *MRI = MIB.getMRI();
185:   if (!MRI->getRegClassOrNull(ResVReg))
186:     MRI->setRegClass(ResVReg, GR->getRegClass(ResType));
187:   if (ResType == OpType)
188:     MIB.buildInstr(TargetOpcode::COPY).addDef(ResVReg).addUse(OpReg);
189:   else
190:     MIB.buildInstr(SPIRV::OpBitcast)
191:         .addDef(ResVReg)
192:         .addUse(GR->getSPIRVTypeID(ResType))
193:         .addUse(OpReg);
194: }
195:
196: // We lower G_BITCAST to OpBitcast here to avoid a MachineVerifier error.
197: // The verifier checks if the source and destination LLTs of a G_BITCAST are
198: // different, but this check is too strict for SPIR-V's typed pointers, which
199: // may have the same LLT but different SPIRV type (e.g. pointers to different
200: // pointee types). By lowering to OpBitcast here, we bypass the verifier's
201: // check. See discussion in https://github.com/llvm/llvm-project/pull/110270
202: // for more context.
203: //
204: // We also handle the llvm.spv.bitcast intrinsic here. If the source and
205: // destination SPIR-V types are the same, we lower it to a COPY to enable
206: // further optimizations like copy propagation.
207: static void lowerBitcasts(MachineFunction &MF, SPIRVGlobalRegistry *GR,
208:                           MachineIRBuilder MIB) {
209:   SmallVector<MachineInstr *, 16> ToErase;
210:   for (MachineBasicBlock &MBB : MF) {
211:     for (MachineInstr &MI : MBB) {
212:       if (isSpvIntrinsic(MI, Intrinsic::spv_bitcast)) {
213:         Register DstReg = MI.getOperand(0).getReg();
214:         Register SrcReg = MI.getOperand(2).getReg();
215:         SPIRVTypeInst DstType = GR->getSPIRVTypeForVReg(DstReg);
216:         assert(
217:             DstType &&
218:             "Expected destination SPIR-V type to have been assigned already.");
219:         SPIRVTypeInst SrcType = GR->getSPIRVTypeForVReg(SrcReg);
220:         assert(SrcType &&
221:                "Expected source SPIR-V type to have been assigned already.");
222:         if (DstType == SrcType) {
223:           MIB.setInsertPt(*MI.getParent(), MI);
224:           MIB.buildCopy(DstReg, SrcReg);
225:           ToErase.push_back(&MI);
226:           continue;
227:         }
228:       }
229:
230:       if (MI.getOpcode() != TargetOpcode::G_BITCAST)
231:         continue;
232:
233:       MIB.setInsertPt(*MI.getParent(), MI);
234:       buildOpBitcast(GR, MIB, MI.getOperand(0).getReg(),
235:                      MI.getOperand(1).getReg());
236:       ToErase.push_back(&MI);
237:     }
238:   }
239:   for (MachineInstr *MI : ToErase)
240:     invalidateAndEraseMI(GR, MI);
241: }
242:
243: static void insertBitcasts(MachineFunction &MF, SPIRVGlobalRegistry *GR,
244:                            MachineIRBuilder MIB) {
245:   // Get access to information about available extensions
246:   const SPIRVSubtarget *ST =
247:       static_cast<const SPIRVSubtarget *>(&MIB.getMF().getSubtarget());
248:   SmallVector<MachineInstr *, 10> ToErase;
249:   for (MachineBasicBlock &MBB : MF) {
250:     for (MachineInstr &MI : MBB) {
251:       if (!isSpvIntrinsic(MI, Intrinsic::spv_ptrcast))
252:         continue;
253:       assert(MI.getOperand(2).isReg());
254:       MIB.setInsertPt(*MI.getParent(), MI);
255:       ToErase.push_back(&MI);
256:       Register Def = MI.getOperand(0).getReg();
257:       Register Source = MI.getOperand(2).getReg();
258:       Type *ElemTy = getMDOperandAsType(MI.getOperand(3).getMetadata(), 0);
259:       auto SC =
260:           isa<FunctionType>(ElemTy)
261:               ? SPIRV::StorageClass::CodeSectionINTEL
262:               : addressSpaceToStorageClass(MI.getOperand(4).getImm(), *ST);
263:       SPIRVTypeInst AssignedPtrType =
264:           GR->getOrCreateSPIRVPointerType(ElemTy, MI, SC);
265:
266:       // If the ptrcast would be redundant, replace all uses with the source
267:       // register.
268:       MachineRegisterInfo *MRI = MIB.getMRI();
269:       if (GR->getSPIRVTypeForVReg(Source) == AssignedPtrType) {
270:         // Erase Def's assign type instruction if we are going to replace Def.
```
- EN: This range implements operational logic in helpers such as assert, report_fatal_error, getMRI, setRegClass, translating backend policy into executable code.
- CN: 这一段实现了 assert、report_fatal_error、getMRI、setRegClass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:         if (MachineInstr *AssignMI = findAssignTypeInstr(Def, MRI))
272:           ToErase.push_back(AssignMI);
273:         MRI->replaceRegWith(Def, Source);
274:       } else {
275:         if (!GR->getSPIRVTypeForVReg(Def, &MF))
276:           GR->assignSPIRVTypeToVReg(AssignedPtrType, Def, MF);
277:         MIB.buildBitcast(Def, Source);
278:       }
279:     }
280:   }
281:   for (MachineInstr *MI : ToErase)
282:     invalidateAndEraseMI(GR, MI);
283: }
284:
285: // Translating GV, IRTranslator sometimes generates following IR:
286: //   %1 = G_GLOBAL_VALUE
287: //   %2 = COPY %1
288: //   %3 = G_ADDRSPACE_CAST %2
289: //
290: // or
291: //
292: //  %1 = G_ZEXT %2
293: //  G_MEMCPY ... %2 ...
294: //
295: // New registers have no SPIRV type and no register class info.
296: //
297: // Set SPIRV type for GV, propagate it from GV to other instructions,
298: // also set register classes.
299: static SPIRVTypeInst propagateSPIRVType(MachineInstr *MI,
300:                                         SPIRVGlobalRegistry *GR,
301:                                         MachineRegisterInfo &MRI,
302:                                         MachineIRBuilder &MIB) {
303:   SPIRVTypeInst SpvType = nullptr;
304:   assert(MI && "Machine instr is expected");
305:   if (MI->getOperand(0).isReg()) {
306:     Register Reg = MI->getOperand(0).getReg();
307:     SpvType = GR->getSPIRVTypeForVReg(Reg);
308:     if (!SpvType) {
309:       switch (MI->getOpcode()) {
310:       case TargetOpcode::G_FCONSTANT:
311:       case TargetOpcode::G_CONSTANT: {
312:         MIB.setInsertPt(*MI->getParent(), MI);
313:         Type *Ty = MI->getOperand(1).getCImm()->getType();
314:         SpvType = GR->getOrCreateSPIRVType(
315:             Ty, MIB, SPIRV::AccessQualifier::ReadWrite, true);
316:         break;
317:       }
318:       case TargetOpcode::G_GLOBAL_VALUE: {
319:         MIB.setInsertPt(*MI->getParent(), MI);
320:         const GlobalValue *Global = MI->getOperand(1).getGlobal();
321:         Type *ElementTy = toTypedPointer(GR->getDeducedGlobalValueType(Global));
322:         auto *Ty = TypedPointerType::get(ElementTy,
323:                                          Global->getType()->getAddressSpace());
324:         SpvType = GR->getOrCreateSPIRVType(
325:             Ty, MIB, SPIRV::AccessQualifier::ReadWrite, true);
326:         break;
327:       }
328:       case TargetOpcode::G_ANYEXT:
329:       case TargetOpcode::G_SEXT:
330:       case TargetOpcode::G_ZEXT: {
331:         if (MI->getOperand(1).isReg()) {
332:           if (MachineInstr *DefInstr =
333:                   MRI.getVRegDef(MI->getOperand(1).getReg())) {
334:             if (SPIRVTypeInst Def =
335:                     propagateSPIRVType(DefInstr, GR, MRI, MIB)) {
336:               unsigned CurrentBW = GR->getScalarOrVectorBitWidth(Def);
337:               unsigned ExpectedBW =
338:                   std::max(MRI.getType(Reg).getScalarSizeInBits(), CurrentBW);
339:               unsigned NumElements = GR->getScalarOrVectorComponentCount(Def);
340:               SpvType = GR->getOrCreateSPIRVIntegerType(ExpectedBW, MIB);
341:               if (NumElements > 1)
342:                 SpvType = GR->getOrCreateSPIRVVectorType(SpvType, NumElements,
343:                                                          MIB, true);
344:             }
345:           }
346:         }
347:         break;
348:       }
349:       case TargetOpcode::G_PTRTOINT:
350:         SpvType = GR->getOrCreateSPIRVIntegerType(
351:             MRI.getType(Reg).getScalarSizeInBits(), MIB);
352:         break;
353:       case TargetOpcode::G_TRUNC:
354:       case TargetOpcode::G_ADDRSPACE_CAST:
355:       case TargetOpcode::G_PTR_ADD:
356:       case TargetOpcode::COPY: {
357:         MachineOperand &Op = MI->getOperand(1);
358:         MachineInstr *Def = Op.isReg() ? MRI.getVRegDef(Op.getReg()) : nullptr;
359:         if (Def)
360:           SpvType = propagateSPIRVType(Def, GR, MRI, MIB);
```
- EN: This range implements operational logic in helpers such as push_back, replaceRegWith, assignSPIRVTypeToVReg, buildBitcast, translating backend policy into executable code.
- CN: 这一段实现了 push_back、replaceRegWith、assignSPIRVTypeToVReg、buildBitcast 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:         break;
362:       }
363:       default:
364:         break;
365:       }
366:       if (SpvType) {
367:         // check if the address space needs correction
368:         LLT RegType = MRI.getType(Reg);
369:         if (SpvType->getOpcode() == SPIRV::OpTypePointer &&
370:             RegType.isPointer() &&
371:             storageClassToAddressSpace(GR->getPointerStorageClass(SpvType)) !=
372:                 RegType.getAddressSpace()) {
373:           const SPIRVSubtarget &ST =
374:               MI->getParent()->getParent()->getSubtarget<SPIRVSubtarget>();
375:           auto TSC = addressSpaceToStorageClass(RegType.getAddressSpace(), ST);
376:           SpvType = GR->changePointerStorageClass(SpvType, TSC, *MI);
377:         }
378:         GR->assignSPIRVTypeToVReg(SpvType, Reg, MIB.getMF());
379:       }
380:       if (!MRI.getRegClassOrNull(Reg))
381:         MRI.setRegClass(Reg, SpvType ? GR->getRegClass(SpvType)
382:                                      : &SPIRV::iIDRegClass);
383:     }
384:   }
385:   return SpvType;
386: }
387:
388: // To support current approach and limitations wrt. bit width here we widen a
389: // scalar register with a bit width greater than 1 to valid sizes and cap it to
390: // 128 width.
391: static unsigned widenBitWidthToNextPow2(unsigned BitWidth) {
392:   if (BitWidth == 1)
393:     return 1; // No need to widen 1-bit values
394:   return std::min(std::max(1u << Log2_32_Ceil(BitWidth), 8u), 128u);
395: }
396:
397: static void widenScalarType(Register Reg, MachineRegisterInfo &MRI) {
398:   LLT RegType = MRI.getType(Reg);
399:   if (!RegType.isScalar())
400:     return;
401:   unsigned CurrentWidth = RegType.getScalarSizeInBits();
402:   unsigned NewWidth = widenBitWidthToNextPow2(CurrentWidth);
403:   if (NewWidth != CurrentWidth)
404:     MRI.setType(Reg, LLT::scalar(NewWidth));
405: }
406:
407: static void widenCImmType(MachineOperand &MOP) {
408:   const ConstantInt *CImmVal = MOP.getCImm();
409:   unsigned CurrentWidth = CImmVal->getBitWidth();
410:   unsigned NewWidth = widenBitWidthToNextPow2(CurrentWidth);
411:   if (NewWidth != CurrentWidth) {
412:     // Replace the immediate value with the widened version
413:     MOP.setCImm(ConstantInt::get(CImmVal->getType()->getContext(),
414:                                  CImmVal->getValue().zextOrTrunc(NewWidth)));
415:   }
416: }
417:
418: static void setInsertPtAfterDef(MachineIRBuilder &MIB, MachineInstr *Def) {
419:   MachineBasicBlock &MBB = *Def->getParent();
420:   MachineBasicBlock::iterator DefIt =
421:       Def->getNextNode() ? Def->getNextNode()->getIterator() : MBB.end();
422:   // Skip all the PHI and debug instructions.
423:   while (DefIt != MBB.end() &&
424:          (DefIt->isPHI() || DefIt->isDebugOrPseudoInstr()))
425:     DefIt = std::next(DefIt);
426:   MIB.setInsertPt(MBB, DefIt);
427: }
428:
429: namespace llvm {
430: void updateRegType(Register Reg, Type *Ty, SPIRVTypeInst SpvType,
431:                    SPIRVGlobalRegistry *GR, MachineIRBuilder &MIB,
432:                    MachineRegisterInfo &MRI) {
433:   assert((Ty || SpvType) && "Either LLVM or SPIRV type is expected.");
434:   MachineInstr *Def = MRI.getVRegDef(Reg);
435:   setInsertPtAfterDef(MIB, Def);
436:   if (!SpvType)
437:     SpvType = GR->getOrCreateSPIRVType(Ty, MIB,
438:                                        SPIRV::AccessQualifier::ReadWrite, true);
439:   if (!MRI.getRegClassOrNull(Reg))
440:     MRI.setRegClass(Reg, GR->getRegClass(SpvType));
441:   if (!MRI.getType(Reg).isValid())
442:     MRI.setType(Reg, GR->getRegType(SpvType));
443:   GR->assignSPIRVTypeToVReg(SpvType, Reg, MIB.getMF());
444: }
445:
446: void processInstr(MachineInstr &MI, MachineIRBuilder &MIB,
447:                   MachineRegisterInfo &MRI, SPIRVGlobalRegistry *GR,
448:                   SPIRVTypeInst KnownResType) {
449:   MIB.setInsertPt(*MI.getParent(), MI.getIterator());
450:   for (auto &Op : MI.operands()) {
```
- EN: This range implements operational logic in helpers such as getType, getAddressSpace, getParent, addressSpaceToStorageClass, translating backend policy into executable code.
- CN: 这一段实现了 getType、getAddressSpace、getParent、addressSpaceToStorageClass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:     if (!Op.isReg() || Op.isDef())
452:       continue;
453:     Register OpReg = Op.getReg();
454:     SPIRVTypeInst SpvType = GR->getSPIRVTypeForVReg(OpReg);
455:     if (!SpvType && KnownResType) {
456:       SpvType = KnownResType;
457:       GR->assignSPIRVTypeToVReg(KnownResType, OpReg, *MI.getMF());
458:     }
459:     assert(SpvType);
460:     if (!MRI.getRegClassOrNull(OpReg))
461:       MRI.setRegClass(OpReg, GR->getRegClass(SpvType));
462:     if (!MRI.getType(OpReg).isValid())
463:       MRI.setType(OpReg, GR->getRegType(SpvType));
464:   }
465: }
466: } // namespace llvm
467:
468: static void
469: generateAssignInstrs(MachineFunction &MF, SPIRVGlobalRegistry *GR,
470:                      MachineIRBuilder MIB,
471:                      DenseMap<MachineInstr *, Type *> &TargetExtConstTypes) {
472:   // Get access to information about available extensions
473:   const SPIRVSubtarget *ST =
474:       static_cast<const SPIRVSubtarget *>(&MIB.getMF().getSubtarget());
475:
476:   MachineRegisterInfo &MRI = MF.getRegInfo();
477:   SmallVector<MachineInstr *, 10> ToErase;
478:   DenseMap<MachineInstr *, Register> RegsAlreadyAddedToDT;
479:
480:   bool IsExtendedInts =
481:       ST->canUseExtension(
482:           SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers) ||
483:       ST->canUseExtension(SPIRV::Extension::SPV_KHR_bit_instructions) ||
484:       ST->canUseExtension(SPIRV::Extension::SPV_INTEL_int4);
485:
486:   if (!IsExtendedInts) {
487:     // Without arbitrary precision integer extensions, SPIR-V only supports
488:     // integer widths of 8, 16, 32, 64. Non-standard widths (e.g., i24, i40)
489:     // must be widened to the next power of two.
490:     //
491:     // G_TRUNC requires special handling because its semantics depend on the
492:     // original destination width. For example:
493:     //   %dst:s24 = G_TRUNC %src:s64
494:     // After widening s24 to s32, we cannot simply do:
495:     //   %dst:s32 = G_TRUNC %src:s64
496:     // because this would keep 32 bits instead of 24. Instead, we insert a
497:     // G_AND to mask the value to the original width:
498:     //   %mask:s64 = G_CONSTANT 0xFFFFFF      ; 24-bit mask
499:     //   %masked:s64 = G_AND %src:s64, %mask
500:     //   %dst:s32 = G_TRUNC %masked:s64
501:     // If src and dst widen to the same size, G_TRUNC is replaced entirely:
502:     //   %mask:s64 = G_CONSTANT 0xFFFFFFFFFF  ; 40-bit mask
503:     //   %dst:s64 = G_AND %src:s64, %mask
504:     SmallVector<MachineInstr *, 8> TruncToRemove;
505:     for (MachineBasicBlock &MBB : MF) {
506:       for (MachineInstr &MI : MBB) {
507:         unsigned MIOp = MI.getOpcode();
508:         if (MIOp != TargetOpcode::G_TRUNC)
509:           continue;
510:         assert(MI.getNumOperands() == 2);
511:         assert(MI.getOperand(0).isReg());
512:         assert(MI.getOperand(1).isReg());
513:
514:         Register DstReg = MI.getOperand(0).getReg();
515:         Register SrcReg = MI.getOperand(1).getReg();
516:
517:         // TODO: handle vector types.
518:         if (!MRI.getType(DstReg).isScalar()) {
519:           assert(!MRI.getType(SrcReg).isScalar());
520:           continue;
521:         }
522:
523:         unsigned OriginalDstWidth = MRI.getType(DstReg).getScalarSizeInBits();
524:         unsigned OriginalSrcWidth = MRI.getType(SrcReg).getScalarSizeInBits();
525:
526:         unsigned NewDstWidth = widenBitWidthToNextPow2(OriginalDstWidth);
527:         unsigned NewSrcWidth = widenBitWidthToNextPow2(OriginalSrcWidth);
528:
529:         // No Dst width change means no truncation semantics change.
530:         if (OriginalDstWidth == NewDstWidth)
531:           continue;
532:
533:         MRI.setType(SrcReg, LLT::scalar(NewSrcWidth));
534:         MRI.setType(DstReg, LLT::scalar(NewDstWidth));
535:
536:         MIB.setInsertPt(MBB, MI.getIterator());
537:         APInt Mask = APInt::getLowBitsSet(NewSrcWidth, OriginalDstWidth);
538:         auto MaskReg = MIB.buildConstant(LLT::scalar(NewSrcWidth), Mask);
539:         Register MaskedReg =
540:             MRI.createGenericVirtualRegister(LLT::scalar(NewSrcWidth));
```
- EN: This range implements operational logic in helpers such as getReg, getSPIRVTypeForVReg, assignSPIRVTypeToVReg, assert, translating backend policy into executable code.
- CN: 这一段实现了 getReg、getSPIRVTypeForVReg、assignSPIRVTypeToVReg、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:         MIB.buildAnd(MaskedReg, SrcReg, MaskReg);
542:
543:         if (NewSrcWidth == NewDstWidth) {
544:           MRI.replaceRegWith(DstReg, MaskedReg);
545:           TruncToRemove.push_back(&MI);
546:         } else {
547:           MI.getOperand(1).setReg(MaskedReg);
548:         }
549:       }
550:     }
551:     for (MachineInstr *MI : TruncToRemove)
552:       MI->eraseFromParent();
553:   }
554:
555:   for (MachineBasicBlock *MBB : post_order(&MF)) {
556:     if (MBB->empty())
557:       continue;
558:
559:     bool ReachedBegin = false;
560:     for (auto MII = std::prev(MBB->end()), Begin = MBB->begin();
561:          !ReachedBegin;) {
562:       MachineInstr &MI = *MII;
563:       unsigned MIOp = MI.getOpcode();
564:
565:       if (!IsExtendedInts) {
566:         // validate bit width of scalar registers and constant immediates
567:         for (auto &MOP : MI.operands()) {
568:           if (MOP.isReg())
569:             widenScalarType(MOP.getReg(), MRI);
570:           else if (MOP.isCImm())
571:             widenCImmType(MOP);
572:         }
573:       }
574:
575:       if (isSpvIntrinsic(MI, Intrinsic::spv_assign_ptr_type)) {
576:         Register Reg = MI.getOperand(1).getReg();
577:         MIB.setInsertPt(*MI.getParent(), MI.getIterator());
578:         Type *ElementTy = getMDOperandAsType(MI.getOperand(2).getMetadata(), 0);
579:         SPIRVTypeInst AssignedPtrType = GR->getOrCreateSPIRVPointerType(
580:             ElementTy, MI,
581:             addressSpaceToStorageClass(MI.getOperand(3).getImm(), *ST));
582:         MachineInstr *Def = MRI.getVRegDef(Reg);
583:         assert(Def && "Expecting an instruction that defines the register");
584:         // G_GLOBAL_VALUE already has type info.
585:         if (Def->getOpcode() != TargetOpcode::G_GLOBAL_VALUE)
586:           updateRegType(Reg, nullptr, AssignedPtrType, GR, MIB,
587:                         MF.getRegInfo());
588:         ToErase.push_back(&MI);
589:       } else if (isSpvIntrinsic(MI, Intrinsic::spv_assign_type)) {
590:         Register Reg = MI.getOperand(1).getReg();
591:         Type *Ty = getMDOperandAsType(MI.getOperand(2).getMetadata(), 0);
592:         MachineInstr *Def = MRI.getVRegDef(Reg);
593:         assert(Def && "Expecting an instruction that defines the register");
594:         // G_GLOBAL_VALUE already has type info.
595:         if (Def->getOpcode() != TargetOpcode::G_GLOBAL_VALUE)
596:           updateRegType(Reg, Ty, nullptr, GR, MIB, MF.getRegInfo());
597:         ToErase.push_back(&MI);
598:       } else if (MIOp == TargetOpcode::FAKE_USE && MI.getNumOperands() > 0) {
599:         MachineInstr *MdMI = MI.getPrevNode();
600:         if (MdMI && isSpvIntrinsic(*MdMI, Intrinsic::spv_value_md)) {
601:           // It's an internal service info from before IRTranslator passes.
602:           MachineInstr *Def = getVRegDef(MRI, MI.getOperand(0).getReg());
603:           for (unsigned I = 1, E = MI.getNumOperands(); I != E && Def; ++I)
604:             if (getVRegDef(MRI, MI.getOperand(I).getReg()) != Def)
605:               Def = nullptr;
606:           if (Def) {
607:             const MDNode *MD = MdMI->getOperand(1).getMetadata();
608:             StringRef ValueName =
609:                 cast<MDString>(MD->getOperand(1))->getString();
610:             const MDNode *TypeMD = cast<MDNode>(MD->getOperand(0));
611:             Type *ValueTy = getMDOperandAsType(TypeMD, 0);
612:             GR->addValueAttrs(Def, std::make_pair(ValueTy, ValueName.str()));
613:           }
614:           ToErase.push_back(MdMI);
615:         }
616:         ToErase.push_back(&MI);
617:       } else if (MIOp == TargetOpcode::G_CONSTANT ||
618:                  MIOp == TargetOpcode::G_FCONSTANT ||
619:                  MIOp == TargetOpcode::G_BUILD_VECTOR) {
620:         // %rc = G_CONSTANT ty Val
621:         // Ensure %rc has a valid SPIR-V type assigned in the Global Registry.
622:         Register Reg = MI.getOperand(0).getReg();
623:         bool NeedAssignType = !GR->getSPIRVTypeForVReg(Reg);
624:         Type *Ty = nullptr;
625:         if (MIOp == TargetOpcode::G_CONSTANT) {
626:           auto TargetExtIt = TargetExtConstTypes.find(&MI);
627:           Ty = TargetExtIt == TargetExtConstTypes.end()
628:                    ? MI.getOperand(1).getCImm()->getType()
629:                    : TargetExtIt->second;
630:           const ConstantInt *OpCI = MI.getOperand(1).getCImm();
```
- EN: This range implements operational logic in helpers such as buildAnd, replaceRegWith, push_back, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 buildAnd、replaceRegWith、push_back、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:           // TODO: we may wish to analyze here if OpCI is zero and LLT RegType =
632:           // MRI.getType(Reg); RegType.isPointer() is true, so that we observe
633:           // at this point not i64/i32 constant but null pointer in the
634:           // corresponding address space of RegType.getAddressSpace(). This may
635:           // help to successfully validate the case when a OpConstantComposite's
636:           // constituent has type that does not match Result Type of
637:           // OpConstantComposite (see, for example,
638:           // pointers/PtrCast-null-in-OpSpecConstantOp.ll).
639:           Register PrimaryReg = GR->find(OpCI, &MF);
640:           if (!PrimaryReg.isValid()) {
641:             GR->add(OpCI, &MI);
642:           } else if (PrimaryReg != Reg &&
643:                      MRI.getType(Reg) == MRI.getType(PrimaryReg)) {
644:             auto *RCReg = MRI.getRegClassOrNull(Reg);
645:             auto *RCPrimary = MRI.getRegClassOrNull(PrimaryReg);
646:             if (!RCReg || RCPrimary == RCReg) {
647:               RegsAlreadyAddedToDT[&MI] = PrimaryReg;
648:               ToErase.push_back(&MI);
649:               NeedAssignType = false;
650:             }
651:           }
652:         } else if (MIOp == TargetOpcode::G_FCONSTANT) {
653:           Ty = MI.getOperand(1).getFPImm()->getType();
654:         } else {
655:           assert(MIOp == TargetOpcode::G_BUILD_VECTOR);
656:           Type *ElemTy = nullptr;
657:           MachineInstr *ElemMI = MRI.getVRegDef(MI.getOperand(1).getReg());
658:           assert(ElemMI);
659:
660:           if (ElemMI->getOpcode() == TargetOpcode::G_CONSTANT) {
661:             ElemTy = ElemMI->getOperand(1).getCImm()->getType();
662:           } else if (ElemMI->getOpcode() == TargetOpcode::G_FCONSTANT) {
663:             ElemTy = ElemMI->getOperand(1).getFPImm()->getType();
664:           } else {
665:             if (SPIRVTypeInst ElemSpvType =
666:                     GR->getSPIRVTypeForVReg(MI.getOperand(1).getReg(), &MF))
667:               ElemTy = const_cast<Type *>(GR->getTypeForSPIRVType(ElemSpvType));
668:           }
669:           if (ElemTy)
670:             Ty = VectorType::get(
671:                 ElemTy, MI.getNumExplicitOperands() - MI.getNumExplicitDefs(),
672:                 false);
673:           else
674:             NeedAssignType = false;
675:         }
676:         if (NeedAssignType)
677:           updateRegType(Reg, Ty, nullptr, GR, MIB, MRI);
678:       } else if (MIOp == TargetOpcode::G_GLOBAL_VALUE) {
679:         propagateSPIRVType(&MI, GR, MRI, MIB);
680:       }
681:
682:       if (MII == Begin)
683:         ReachedBegin = true;
684:       else
685:         --MII;
686:     }
687:   }
688:   for (MachineInstr *MI : ToErase) {
689:     auto It = RegsAlreadyAddedToDT.find(MI);
690:     if (It != RegsAlreadyAddedToDT.end())
691:       MRI.replaceRegWith(MI->getOperand(0).getReg(), It->second);
692:     invalidateAndEraseMI(GR, MI);
693:   }
694:
695:   // Address the case when IRTranslator introduces instructions with new
696:   // registers without associated SPIRV type.
697:   for (MachineBasicBlock &MBB : MF) {
698:     for (MachineInstr &MI : MBB) {
699:       switch (MI.getOpcode()) {
700:       case TargetOpcode::G_TRUNC:
701:       case TargetOpcode::G_ANYEXT:
702:       case TargetOpcode::G_SEXT:
703:       case TargetOpcode::G_ZEXT:
704:       case TargetOpcode::G_PTRTOINT:
705:       case TargetOpcode::COPY:
706:       case TargetOpcode::G_ADDRSPACE_CAST:
707:         propagateSPIRVType(&MI, GR, MRI, MIB);
708:         break;
709:       }
710:     }
711:   }
712: }
713:
714: static void processInstrsWithTypeFolding(MachineFunction &MF,
715:                                          SPIRVGlobalRegistry *GR,
716:                                          MachineIRBuilder MIB) {
717:   MachineRegisterInfo &MRI = MF.getRegInfo();
718:   for (MachineBasicBlock &MBB : MF)
719:     for (MachineInstr &MI : MBB)
720:       if (isTypeFoldingSupported(MI.getOpcode()))
```
- EN: This range implements operational logic in helpers such as find, add, getType, getRegClassOrNull, translating backend policy into executable code.
- CN: 这一段实现了 find、add、getType、getRegClassOrNull 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:         processInstr(MI, MIB, MRI, GR, nullptr);
722: }
723:
724: static Register
725: collectInlineAsmInstrOperands(MachineInstr *MI,
726:                               SmallVector<unsigned, 4> *Ops = nullptr) {
727:   Register DefReg;
728:   unsigned StartOp = InlineAsm::MIOp_FirstOperand,
729:            AsmDescOp = InlineAsm::MIOp_FirstOperand;
730:   for (unsigned Idx = StartOp, MISz = MI->getNumOperands(); Idx != MISz;
731:        ++Idx) {
732:     const MachineOperand &MO = MI->getOperand(Idx);
733:     if (MO.isMetadata())
734:       continue;
735:     if (Idx == AsmDescOp && MO.isImm()) {
736:       // compute the index of the next operand descriptor
737:       const InlineAsm::Flag F(MO.getImm());
738:       AsmDescOp += 1 + F.getNumOperandRegisters();
739:       continue;
740:     }
741:     if (MO.isReg() && MO.isDef()) {
742:       if (!Ops)
743:         return MO.getReg();
744:       DefReg = MO.getReg();
745:     } else if (Ops) {
746:       Ops->push_back(Idx);
747:     }
748:   }
749:   return DefReg;
750: }
751:
752: static void
753: insertInlineAsmProcess(MachineFunction &MF, SPIRVGlobalRegistry *GR,
754:                        const SPIRVSubtarget &ST, MachineIRBuilder MIRBuilder,
755:                        const SmallVector<MachineInstr *> &ToProcess) {
756:   MachineRegisterInfo &MRI = MF.getRegInfo();
757:   Register AsmTargetReg;
758:   for (unsigned i = 0, Sz = ToProcess.size(); i + 1 < Sz; i += 2) {
759:     MachineInstr *I1 = ToProcess[i], *I2 = ToProcess[i + 1];
760:     assert(isSpvIntrinsic(*I1, Intrinsic::spv_inline_asm) && I2->isInlineAsm());
761:     MIRBuilder.setInsertPt(*I2->getParent(), *I2);
762:
763:     if (!AsmTargetReg.isValid()) {
764:       // define vendor specific assembly target or dialect
765:       AsmTargetReg = MRI.createGenericVirtualRegister(LLT::scalar(32));
766:       MRI.setRegClass(AsmTargetReg, &SPIRV::iIDRegClass);
767:       auto AsmTargetMIB =
768:           MIRBuilder.buildInstr(SPIRV::OpAsmTargetINTEL).addDef(AsmTargetReg);
769:       addStringImm(ST.getTargetTripleAsStr(), AsmTargetMIB);
770:       GR->add(AsmTargetMIB.getInstr(), AsmTargetMIB);
771:     }
772:
773:     // create types
774:     const MDNode *IAMD = I1->getOperand(1).getMetadata();
775:     FunctionType *FTy = cast<FunctionType>(getMDOperandAsType(IAMD, 0));
776:     SmallVector<SPIRVTypeInst, 4> ArgTypes;
777:     for (const auto &ArgTy : FTy->params())
778:       ArgTypes.push_back(GR->getOrCreateSPIRVType(
779:           ArgTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true));
780:     SPIRVTypeInst RetType =
781:         GR->getOrCreateSPIRVType(FTy->getReturnType(), MIRBuilder,
782:                                  SPIRV::AccessQualifier::ReadWrite, true);
783:     SPIRVTypeInst FuncType = GR->getOrCreateOpTypeFunctionWithArgs(
784:         FTy, RetType, ArgTypes, MIRBuilder);
785:
786:     // define vendor specific assembly instructions string
787:     Register AsmReg = MRI.createGenericVirtualRegister(LLT::scalar(32));
788:     MRI.setRegClass(AsmReg, &SPIRV::iIDRegClass);
789:     auto AsmMIB = MIRBuilder.buildInstr(SPIRV::OpAsmINTEL)
790:                       .addDef(AsmReg)
791:                       .addUse(GR->getSPIRVTypeID(RetType))
792:                       .addUse(GR->getSPIRVTypeID(FuncType))
793:                       .addUse(AsmTargetReg);
794:     // inline asm string:
795:     addStringImm(I2->getOperand(InlineAsm::MIOp_AsmString).getSymbolName(),
796:                  AsmMIB);
797:     // inline asm constraint string:
798:     addStringImm(cast<MDString>(I1->getOperand(2).getMetadata()->getOperand(0))
799:                      ->getString(),
800:                  AsmMIB);
801:     GR->add(AsmMIB.getInstr(), AsmMIB);
802:
803:     // calls the inline assembly instruction
804:     unsigned ExtraInfo = I2->getOperand(InlineAsm::MIOp_ExtraInfo).getImm();
805:     if (ExtraInfo & InlineAsm::Extra_HasSideEffects)
806:       MIRBuilder.buildInstr(SPIRV::OpDecorate)
807:           .addUse(AsmReg)
808:           .addImm(static_cast<uint32_t>(SPIRV::Decoration::SideEffectsINTEL));
809:
810:     Register DefReg = collectInlineAsmInstrOperands(I2);
```
- EN: This range implements operational logic in helpers such as processInstr, getOperand, F, getNumOperandRegisters, translating backend policy into executable code.
- CN: 这一段实现了 processInstr、getOperand、F、getNumOperandRegisters 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:     if (!DefReg.isValid()) {
812:       DefReg = MRI.createGenericVirtualRegister(LLT::scalar(32));
813:       MRI.setRegClass(DefReg, &SPIRV::iIDRegClass);
814:       SPIRVTypeInst VoidType = GR->getOrCreateSPIRVType(
815:           Type::getVoidTy(MF.getFunction().getContext()), MIRBuilder,
816:           SPIRV::AccessQualifier::ReadWrite, true);
817:       GR->assignSPIRVTypeToVReg(VoidType, DefReg, MF);
818:     }
819:
820:     auto AsmCall = MIRBuilder.buildInstr(SPIRV::OpAsmCallINTEL)
821:                        .addDef(DefReg)
822:                        .addUse(GR->getSPIRVTypeID(RetType))
823:                        .addUse(AsmReg);
824:     for (unsigned IntrIdx = 3; IntrIdx < I1->getNumOperands(); ++IntrIdx)
825:       AsmCall.addUse(I1->getOperand(IntrIdx).getReg());
826:
827:     // IRTranslator gets a bit confused when lowering inline ASM with outputs
828:     // and inserts a spurious COPY & TRUNC as registers are assumed to be i64;
829:     // we have to clean that up here to prevent erroneous trunc casts either on
830:     // a struct (for multiple outputs) or same width integers to get lowered
831:     // into SPIR-V
832:     if (MRI.hasOneUse(DefReg)) {
833:       MachineInstr &CopyMI = *MRI.use_instr_begin(DefReg);
834:       if (CopyMI.getOpcode() == TargetOpcode::COPY) {
835:         Register CopyDst = CopyMI.getOperand(0).getReg();
836:         if (MRI.hasOneUse(CopyDst)) {
837:           MachineInstr &TruncMI = *MRI.use_instr_begin(CopyDst);
838:           if (TruncMI.getOpcode() == TargetOpcode::G_TRUNC) {
839:             MRI.setType(DefReg, GR->getRegType(RetType));
840:             Register TruncReg = TruncMI.defs().begin()->getReg();
841:             MRI.replaceRegWith(TruncReg, DefReg);
842:             invalidateAndEraseMI(GR, &TruncMI);
843:             invalidateAndEraseMI(GR, &CopyMI);
844:           }
845:         }
846:       }
847:     }
848:   }
849:   for (MachineInstr *MI : ToProcess)
850:     invalidateAndEraseMI(GR, MI);
851: }
852:
853: static void insertInlineAsm(MachineFunction &MF, SPIRVGlobalRegistry *GR,
854:                             const SPIRVSubtarget &ST,
855:                             MachineIRBuilder MIRBuilder) {
856:   SmallVector<MachineInstr *> ToProcess;
857:   for (MachineBasicBlock &MBB : MF) {
858:     for (MachineInstr &MI : MBB) {
859:       if (isSpvIntrinsic(MI, Intrinsic::spv_inline_asm) ||
860:           MI.getOpcode() == TargetOpcode::INLINEASM)
861:         ToProcess.push_back(&MI);
862:     }
863:   }
864:   if (ToProcess.size() == 0)
865:     return;
866:
867:   if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_inline_assembly))
868:     report_fatal_error("Inline assembly instructions require the "
869:                        "following SPIR-V extension: SPV_INTEL_inline_assembly",
870:                        false);
871:
872:   insertInlineAsmProcess(MF, GR, ST, MIRBuilder, ToProcess);
873: }
874:
875: static uint32_t convertFloatToSPIRVWord(float F) {
876:   union {
877:     float F;
878:     uint32_t Spir;
879:   } FPMaxError;
880:   FPMaxError.F = F;
881:   return FPMaxError.Spir;
882: }
883:
884: static void insertSpirvDecorations(MachineFunction &MF, SPIRVGlobalRegistry *GR,
885:                                    MachineIRBuilder MIB) {
886:   const SPIRVSubtarget &ST = cast<SPIRVSubtarget>(MIB.getMF().getSubtarget());
887:   SmallVector<MachineInstr *, 10> ToErase;
888:   for (MachineBasicBlock &MBB : MF) {
889:     for (MachineInstr &MI : MBB) {
890:       if (!isSpvIntrinsic(MI, Intrinsic::spv_assign_decoration) &&
891:           !isSpvIntrinsic(MI, Intrinsic::spv_assign_aliasing_decoration) &&
892:           !isSpvIntrinsic(MI, Intrinsic::spv_assign_fpmaxerror_decoration))
893:         continue;
894:       MIB.setInsertPt(*MI.getParent(), MI.getNextNode());
895:       if (isSpvIntrinsic(MI, Intrinsic::spv_assign_decoration)) {
896:         buildOpSpirvDecorations(MI.getOperand(1).getReg(), MIB,
897:                                 MI.getOperand(2).getMetadata(), ST);
898:       } else if (isSpvIntrinsic(MI,
899:                                 Intrinsic::spv_assign_fpmaxerror_decoration)) {
900:         ConstantFP *OpV = mdconst::dyn_extract<ConstantFP>(
```
- EN: This range implements operational logic in helpers such as createGenericVirtualRegister, setRegClass, assignSPIRVTypeToVReg, buildInstr, translating backend policy into executable code.
- CN: 这一段实现了 createGenericVirtualRegister、setRegClass、assignSPIRVTypeToVReg、buildInstr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:             MI.getOperand(2).getMetadata()->getOperand(0));
902:         uint32_t OpValue =
903:             convertFloatToSPIRVWord(OpV->getValueAPF().convertToFloat());
904:
905:         buildOpDecorate(MI.getOperand(1).getReg(), MIB,
906:                         SPIRV::Decoration::FPMaxErrorDecorationINTEL,
907:                         {OpValue});
908:       } else {
909:         GR->buildMemAliasingOpDecorate(MI.getOperand(1).getReg(), MIB,
910:                                        MI.getOperand(2).getImm(),
911:                                        MI.getOperand(3).getMetadata());
912:       }
913:
914:       ToErase.push_back(&MI);
915:     }
916:   }
917:   for (MachineInstr *MI : ToErase)
918:     invalidateAndEraseMI(GR, MI);
919: }
920:
921: // LLVM allows the switches to use registers as cases, while SPIR-V required
922: // those to be immediate values. This function replaces such operands with the
923: // equivalent immediate constant.
924: static void processSwitchesConstants(MachineFunction &MF,
925:                                      SPIRVGlobalRegistry *GR,
926:                                      MachineIRBuilder MIB) {
927:   MachineRegisterInfo &MRI = MF.getRegInfo();
928:   for (MachineBasicBlock &MBB : MF) {
929:     for (MachineInstr &MI : MBB) {
930:       if (!isSpvIntrinsic(MI, Intrinsic::spv_switch))
931:         continue;
932:
933:       SmallVector<MachineOperand, 8> NewOperands;
934:       NewOperands.push_back(MI.getOperand(0)); // Opcode
935:       NewOperands.push_back(MI.getOperand(1)); // Condition
936:       NewOperands.push_back(MI.getOperand(2)); // Default
937:       for (unsigned i = 3; i < MI.getNumOperands(); i += 2) {
938:         Register Reg = MI.getOperand(i).getReg();
939:         MachineInstr *ConstInstr = getDefInstrMaybeConstant(Reg, &MRI);
940:         NewOperands.push_back(
941:             MachineOperand::CreateCImm(ConstInstr->getOperand(1).getCImm()));
942:
943:         NewOperands.push_back(MI.getOperand(i + 1));
944:       }
945:
946:       assert(MI.getNumOperands() == NewOperands.size());
947:       while (MI.getNumOperands() > 0)
948:         MI.removeOperand(0);
949:       for (auto &MO : NewOperands)
950:         MI.addOperand(MO);
951:     }
952:   }
953: }
954:
955: // Some instructions are used during CodeGen but should never be emitted.
956: // Cleaning up those.
957: static void cleanupHelperInstructions(MachineFunction &MF,
958:                                       SPIRVGlobalRegistry *GR) {
959:   SmallVector<MachineInstr *, 8> ToEraseMI;
960:   for (MachineBasicBlock &MBB : MF) {
961:     for (MachineInstr &MI : MBB) {
962:       if (isSpvIntrinsic(MI, Intrinsic::spv_track_constant) ||
963:           MI.getOpcode() == TargetOpcode::G_BRINDIRECT)
964:         ToEraseMI.push_back(&MI);
965:     }
966:   }
967:
968:   for (MachineInstr *MI : ToEraseMI)
969:     invalidateAndEraseMI(GR, MI);
970: }
971:
972: // Find all usages of G_BLOCK_ADDR in our intrinsics and replace those
973: // operands/registers by the actual MBB it references.
974: static void processBlockAddr(MachineFunction &MF, SPIRVGlobalRegistry *GR,
975:                              MachineIRBuilder MIB) {
976:   // Gather the reverse-mapping BB -> MBB.
977:   DenseMap<const BasicBlock *, MachineBasicBlock *> BB2MBB;
978:   for (MachineBasicBlock &MBB : MF)
979:     BB2MBB[MBB.getBasicBlock()] = &MBB;
980:
981:   // Gather instructions requiring patching. For now, only those can use
982:   // G_BLOCK_ADDR.
983:   SmallVector<MachineInstr *, 8> InstructionsToPatch;
984:   for (MachineBasicBlock &MBB : MF) {
985:     for (MachineInstr &MI : MBB) {
986:       if (isSpvIntrinsic(MI, Intrinsic::spv_switch) ||
987:           isSpvIntrinsic(MI, Intrinsic::spv_loop_merge) ||
988:           isSpvIntrinsic(MI, Intrinsic::spv_selection_merge))
989:         InstructionsToPatch.push_back(&MI);
990:     }
```
- EN: This range implements operational logic in helpers such as getOperand, convertFloatToSPIRVWord, push_back, invalidateAndEraseMI, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、convertFloatToSPIRVWord、push_back、invalidateAndEraseMI 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:   }
 992:
 993:   // For each instruction to fix, we replace all the G_BLOCK_ADDR operands by
 994:   // the actual MBB it references. Once those references have been updated, we
 995:   // can cleanup remaining G_BLOCK_ADDR references.
 996:   SmallPtrSet<MachineBasicBlock *, 8> ClearAddressTaken;
 997:   SmallPtrSet<MachineInstr *, 8> ToEraseMI;
 998:   MachineRegisterInfo &MRI = MF.getRegInfo();
 999:   for (MachineInstr *MI : InstructionsToPatch) {
1000:     SmallVector<MachineOperand, 8> NewOps;
1001:     for (unsigned i = 0; i < MI->getNumOperands(); ++i) {
1002:       // The operand is not a register, keep as-is.
1003:       if (!MI->getOperand(i).isReg()) {
1004:         NewOps.push_back(MI->getOperand(i));
1005:         continue;
1006:       }
1007:
1008:       Register Reg = MI->getOperand(i).getReg();
1009:       MachineInstr *BuildMBB = MRI.getVRegDef(Reg);
1010:       // The register is not the result of G_BLOCK_ADDR, keep as-is.
1011:       if (!BuildMBB || BuildMBB->getOpcode() != TargetOpcode::G_BLOCK_ADDR) {
1012:         NewOps.push_back(MI->getOperand(i));
1013:         continue;
1014:       }
1015:
1016:       assert(BuildMBB && BuildMBB->getOpcode() == TargetOpcode::G_BLOCK_ADDR &&
1017:              BuildMBB->getOperand(1).isBlockAddress() &&
1018:              BuildMBB->getOperand(1).getBlockAddress());
1019:       BasicBlock *BB =
1020:           BuildMBB->getOperand(1).getBlockAddress()->getBasicBlock();
1021:       auto It = BB2MBB.find(BB);
1022:       if (It == BB2MBB.end())
1023:         report_fatal_error("cannot find a machine basic block by a basic block "
1024:                            "in a switch statement");
1025:       MachineBasicBlock *ReferencedBlock = It->second;
1026:       NewOps.push_back(MachineOperand::CreateMBB(ReferencedBlock));
1027:
1028:       ClearAddressTaken.insert(ReferencedBlock);
1029:       ToEraseMI.insert(BuildMBB);
1030:     }
1031:
1032:     // Replace the operands.
1033:     assert(MI->getNumOperands() == NewOps.size());
1034:     while (MI->getNumOperands() > 0)
1035:       MI->removeOperand(0);
1036:     for (auto &MO : NewOps)
1037:       MI->addOperand(MO);
1038:
1039:     if (MachineInstr *Next = MI->getNextNode()) {
1040:       if (isSpvIntrinsic(*Next, Intrinsic::spv_track_constant)) {
1041:         ToEraseMI.insert(Next);
1042:         Next = MI->getNextNode();
1043:       }
1044:       if (Next && Next->getOpcode() == TargetOpcode::G_BRINDIRECT)
1045:         ToEraseMI.insert(Next);
1046:     }
1047:   }
1048:
1049:   // BlockAddress operands were used to keep information between passes,
1050:   // let's undo the "address taken" status to reflect that Succ doesn't
1051:   // actually correspond to an IR-level basic block.
1052:   for (MachineBasicBlock *Succ : ClearAddressTaken)
1053:     Succ->setAddressTakenIRBlock(nullptr);
1054:
1055:   // If we just delete G_BLOCK_ADDR instructions with BlockAddress operands,
1056:   // this leaves their BasicBlock counterparts in a "address taken" status. This
1057:   // would make AsmPrinter to generate a series of unneeded labels of a "Address
1058:   // of block that was removed by CodeGen" kind. Let's first ensure that we
1059:   // don't have a dangling BlockAddress constants by zapping the BlockAddress
1060:   // nodes, and only after that proceed with erasing G_BLOCK_ADDR instructions.
1061:   Constant *Replacement =
1062:       ConstantInt::get(Type::getInt32Ty(MF.getFunction().getContext()), 1);
1063:   for (MachineInstr *BlockAddrI : ToEraseMI) {
1064:     if (BlockAddrI->getOpcode() == TargetOpcode::G_BLOCK_ADDR) {
1065:       BlockAddress *BA = const_cast<BlockAddress *>(
1066:           BlockAddrI->getOperand(1).getBlockAddress());
1067:       BA->replaceAllUsesWith(
1068:           ConstantExpr::getIntToPtr(Replacement, BA->getType()));
1069:       BA->destroyConstant();
1070:     }
1071:     invalidateAndEraseMI(GR, BlockAddrI);
1072:   }
1073: }
1074:
1075: static bool isImplicitFallthrough(MachineBasicBlock &MBB) {
1076:   if (MBB.empty())
1077:     return MBB.getNextNode() != nullptr;
1078:
1079:   // Branching SPIR-V intrinsics are not detected by this generic method.
1080:   // Thus, we can only trust negative result.
```
- EN: This range implements operational logic in helpers such as getRegInfo, getNumOperands, push_back, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getRegInfo、getNumOperands、push_back、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1142
```cpp
1081:   if (!MBB.canFallThrough())
1082:     return false;
1083:
1084:   // Otherwise, we must manually check if we have a SPIR-V intrinsic which
1085:   // prevent an implicit fallthrough.
1086:   for (MachineBasicBlock::reverse_iterator It = MBB.rbegin(), E = MBB.rend();
1087:        It != E; ++It) {
1088:     if (isSpvIntrinsic(*It, Intrinsic::spv_switch))
1089:       return false;
1090:   }
1091:   return true;
1092: }
1093:
1094: static void removeImplicitFallthroughs(MachineFunction &MF,
1095:                                        MachineIRBuilder MIB) {
1096:   // It is valid for MachineBasicBlocks to not finish with a branch instruction.
1097:   // In such cases, they will simply fallthrough their immediate successor.
1098:   for (MachineBasicBlock &MBB : MF) {
1099:     if (!isImplicitFallthrough(MBB))
1100:       continue;
1101:
1102:     assert(MBB.succ_size() == 1);
1103:     MIB.setInsertPt(MBB, MBB.end());
1104:     MIB.buildBr(**MBB.successors().begin());
1105:   }
1106: }
1107:
1108: bool SPIRVPreLegalizer::runOnMachineFunction(MachineFunction &MF) {
1109:   // Initialize the type registry.
1110:   const SPIRVSubtarget &ST = MF.getSubtarget<SPIRVSubtarget>();
1111:   SPIRVGlobalRegistry *GR = ST.getSPIRVGlobalRegistry();
1112:   GR->setCurrentFunc(MF);
1113:   MachineIRBuilder MIB(MF);
1114:   // a registry of target extension constants
1115:   DenseMap<MachineInstr *, Type *> TargetExtConstTypes;
1116:   // to keep record of tracked constants
1117:   addConstantsToTrack(MF, GR, ST, TargetExtConstTypes);
1118:   foldConstantsIntoIntrinsics(MF, GR, MIB);
1119:   insertBitcasts(MF, GR, MIB);
1120:   generateAssignInstrs(MF, GR, MIB, TargetExtConstTypes);
1121:
1122:   processSwitchesConstants(MF, GR, MIB);
1123:   processBlockAddr(MF, GR, MIB);
1124:   cleanupHelperInstructions(MF, GR);
1125:
1126:   processInstrsWithTypeFolding(MF, GR, MIB);
1127:   removeImplicitFallthroughs(MF, MIB);
1128:   insertSpirvDecorations(MF, GR, MIB);
1129:   insertInlineAsm(MF, GR, ST, MIB);
1130:   lowerBitcasts(MF, GR, MIB);
1131:
1132:   return true;
1133: }
1134:
1135: INITIALIZE_PASS(SPIRVPreLegalizer, DEBUG_TYPE, "SPIRV pre legalizer", false,
1136:                 false)
1137:
1138: char SPIRVPreLegalizer::ID = 0;
1139:
1140: FunctionPass *llvm::createSPIRVPreLegalizerPass() {
1141:   return new SPIRVPreLegalizer();
1142: }
```
- EN: This range implements operational logic in helpers such as assert, setInsertPt, buildBr, SPIRVPreLegalizer::runOnMachineFunction, translating backend policy into executable code.
- CN: 这一段实现了 assert、setInsertPt、buildBr、SPIRVPreLegalizer::runOnMachineFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Legalization logic rewrites unsupported operations into forms that the target can handle.
  - CN: 合法化逻辑会把目标不支持的操作重写为可处理的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVPreLegalizer, runOnMachineFunction, getAnalysisUsage, SPIRVPreLegalizer::getAnalysisUsage, MachineFunctionPass::getAnalysisUsage, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVPreLegalizer, runOnMachineFunction, getAnalysisUsage, SPIRVPreLegalizer::getAnalysisUsage, MachineFunctionPass::getAnalysisUsage，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/PostOrderIterator.h`
  - `llvm/CodeGen/GlobalISel/CSEInfo.h`
  - `llvm/CodeGen/GlobalISel/GISelValueTracking.h`
  - `llvm/IR/Attributes.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
