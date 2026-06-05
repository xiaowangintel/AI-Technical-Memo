# SPIRVPostLegalizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVPostLegalizer.cpp`
- Repository: `llvm-project`
- Purpose (EN): The pass partially applies pre-legalization logic to new instructions inserted as a result of legalization: assigns SPIR-V types to registers for new instructions.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVPostLegalizer.cpp - amend info after legalization -*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The pass partially applies pre-legalization logic to new instructions
10: // inserted as a result of legalization:
11: // - assigns SPIR-V types to registers for new instructions.
12: // - inserts ASSIGN_TYPE pseudo-instructions required for type folding.
13: //
14: //===----------------------------------------------------------------------===//
15:
16: #include "SPIRV.h"
17: #include "SPIRVSubtarget.h"
18: #include "SPIRVUtils.h"
19: #include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
20: #include "llvm/CodeGen/MachineFrameInfo.h"
21: #include "llvm/IR/IntrinsicsSPIRV.h"
22: #include "llvm/Support/Debug.h"
23: #include <stack>
24:
25: #define DEBUG_TYPE "spirv-postlegalizer"
26:
27: using namespace llvm;
28:
29: namespace {
30: class SPIRVPostLegalizer : public MachineFunctionPass {
31: public:
32:   static char ID;
33:   SPIRVPostLegalizer() : MachineFunctionPass(ID) {}
34:   bool runOnMachineFunction(MachineFunction &MF) override;
35: };
36: } // namespace
37:
38: namespace llvm {
39: //  Defined in SPIRVPreLegalizer.cpp.
40: extern void updateRegType(Register Reg, Type *Ty, SPIRVTypeInst SpirvTy,
41:                           SPIRVGlobalRegistry *GR, MachineIRBuilder &MIB,
42:                           MachineRegisterInfo &MRI);
43: extern void processInstr(MachineInstr &MI, MachineIRBuilder &MIB,
44:                          MachineRegisterInfo &MRI, SPIRVGlobalRegistry *GR,
45:                          SPIRVTypeInst KnownResType);
46: } // namespace llvm
47:
48: static SPIRVTypeInst deduceIntTypeFromResult(Register ResVReg,
49:                                              MachineIRBuilder &MIB,
50:                                              SPIRVGlobalRegistry *GR) {
51:   const LLT &Ty = MIB.getMRI()->getType(ResVReg);
52:   return GR->getOrCreateSPIRVIntegerType(Ty.getScalarSizeInBits(), MIB);
53: }
54:
55: static SPIRVTypeInst deduceTypeFromSingleOperand(MachineInstr *I,
56:                                                  MachineIRBuilder &MIB,
57:                                                  SPIRVGlobalRegistry *GR,
58:                                                  unsigned OpIdx) {
59:   Register OpReg = I->getOperand(OpIdx).getReg();
60:   if (SPIRVTypeInst OpType = GR->getSPIRVTypeForVReg(OpReg)) {
```
- EN: This range defines or declares important types such as SPIRVPostLegalizer, runOnMachineFunction, getMRI, getOrCreateSPIRVIntegerType, shaping the data model used by SPIRVPostLegalizer.cpp.
- CN: 这一段定义或声明了 SPIRVPostLegalizer、runOnMachineFunction、getMRI、getOrCreateSPIRVIntegerType 等关键类型，构成 SPIRVPostLegalizer.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:     if (SPIRVTypeInst CompType = GR->getScalarOrVectorComponentType(OpType)) {
 62:       Register ResVReg = I->getOperand(0).getReg();
 63:       const LLT &ResLLT = MIB.getMRI()->getType(ResVReg);
 64:       if (ResLLT.isVector())
 65:         return GR->getOrCreateSPIRVVectorType(CompType, ResLLT.getNumElements(),
 66:                                               MIB, false);
 67:       return CompType;
 68:     }
 69:   }
 70:   return nullptr;
 71: }
 72:
 73: static SPIRVTypeInst deduceTypeFromOperandRange(MachineInstr *I,
 74:                                                 MachineIRBuilder &MIB,
 75:                                                 SPIRVGlobalRegistry *GR,
 76:                                                 unsigned StartOp,
 77:                                                 unsigned EndOp) {
 78:   SPIRVTypeInst ResType = nullptr;
 79:   for (unsigned i = StartOp; i < EndOp; ++i) {
 80:     if (SPIRVTypeInst Type = deduceTypeFromSingleOperand(I, MIB, GR, i)) {
 81: #ifdef EXPENSIVE_CHECKS
 82:       assert(!ResType || Type == ResType && "Conflicting type from operands.");
 83:       ResType = Type;
 84: #else
 85:       return Type;
 86: #endif
 87:     }
 88:   }
 89:   return ResType;
 90: }
 91:
 92: static SPIRVTypeInst deduceTypeFromResultRegister(MachineInstr *Use,
 93:                                                   Register UseRegister,
 94:                                                   SPIRVGlobalRegistry *GR,
 95:                                                   MachineIRBuilder &MIB) {
 96:   for (const MachineOperand &MO : Use->defs()) {
 97:     if (!MO.isReg())
 98:       continue;
 99:     if (SPIRVTypeInst OpType = GR->getSPIRVTypeForVReg(MO.getReg())) {
100:       if (SPIRVTypeInst CompType = GR->getScalarOrVectorComponentType(OpType)) {
101:         const LLT &ResLLT = MIB.getMRI()->getType(UseRegister);
102:         if (ResLLT.isVector())
103:           return GR->getOrCreateSPIRVVectorType(
104:               CompType, ResLLT.getNumElements(), MIB, false);
105:         return CompType;
106:       }
107:     }
108:   }
109:   return nullptr;
110: }
111:
112: static SPIRVTypeInst
113: deducePointerTypeFromResultRegister(MachineInstr *Use, Register UseRegister,
114:                                     SPIRVGlobalRegistry *GR,
115:                                     MachineIRBuilder &MIB) {
116:   assert(Use->getOpcode() == TargetOpcode::G_LOAD ||
117:          Use->getOpcode() == TargetOpcode::G_STORE);
118:
119:   Register ValueReg = Use->getOperand(0).getReg();
120:   SPIRVTypeInst ValueType = GR->getSPIRVTypeForVReg(ValueReg);
```
- EN: This range implements operational logic in helpers such as getOperand, getMRI, assert, getNumElements, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getMRI、assert、getNumElements 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   if (!ValueType)
122:     return nullptr;
123:
124:   return GR->getOrCreateSPIRVPointerType(ValueType, MIB,
125:                                          SPIRV::StorageClass::Function);
126: }
127:
128: static SPIRVTypeInst deduceTypeFromPointerOperand(MachineInstr *Use,
129:                                                   Register UseRegister,
130:                                                   SPIRVGlobalRegistry *GR,
131:                                                   MachineIRBuilder &MIB) {
132:   assert(Use->getOpcode() == TargetOpcode::G_LOAD ||
133:          Use->getOpcode() == TargetOpcode::G_STORE);
134:
135:   Register PtrReg = Use->getOperand(1).getReg();
136:   SPIRVTypeInst PtrType = GR->getSPIRVTypeForVReg(PtrReg);
137:   if (!PtrType)
138:     return nullptr;
139:
140:   return GR->getPointeeType(PtrType);
141: }
142:
143: static SPIRVTypeInst deduceTypeFromUses(Register Reg, MachineFunction &MF,
144:                                         SPIRVGlobalRegistry *GR,
145:                                         MachineIRBuilder &MIB) {
146:   MachineRegisterInfo &MRI = MF.getRegInfo();
147:   for (MachineInstr &Use : MRI.use_nodbg_instructions(Reg)) {
148:     SPIRVTypeInst ResType = nullptr;
149:     LLVM_DEBUG(dbgs() << "Looking at use " << Use);
150:     switch (Use.getOpcode()) {
151:     case TargetOpcode::G_BUILD_VECTOR:
152:     case TargetOpcode::G_EXTRACT_VECTOR_ELT:
153:     case TargetOpcode::G_UNMERGE_VALUES:
154:     case TargetOpcode::G_ADD:
155:     case TargetOpcode::G_SUB:
156:     case TargetOpcode::G_MUL:
157:     case TargetOpcode::G_SDIV:
158:     case TargetOpcode::G_UDIV:
159:     case TargetOpcode::G_SREM:
160:     case TargetOpcode::G_UREM:
161:     case TargetOpcode::G_FADD:
162:     case TargetOpcode::G_FSUB:
163:     case TargetOpcode::G_FMUL:
164:     case TargetOpcode::G_FDIV:
165:     case TargetOpcode::G_FREM:
166:     case TargetOpcode::G_FMA:
167:     case TargetOpcode::COPY:
168:     case TargetOpcode::G_STRICT_FMA:
169:       ResType = deduceTypeFromResultRegister(&Use, Reg, GR, MIB);
170:       break;
171:     case TargetOpcode::G_LOAD:
172:     case TargetOpcode::G_STORE:
173:       if (Reg == Use.getOperand(1).getReg())
174:         ResType = deducePointerTypeFromResultRegister(&Use, Reg, GR, MIB);
175:       else
176:         ResType = deduceTypeFromPointerOperand(&Use, Reg, GR, MIB);
177:       break;
178:     case TargetOpcode::G_INTRINSIC_W_SIDE_EFFECTS:
179:     case TargetOpcode::G_INTRINSIC: {
180:       auto IntrinsicID = cast<GIntrinsic>(Use).getIntrinsicID();
```
- EN: This range implements operational logic in helpers such as getOpcode, getOperand, getSPIRVTypeForVReg, getPointeeType, translating backend policy into executable code.
- CN: 这一段实现了 getOpcode、getOperand、getSPIRVTypeForVReg、getPointeeType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:       if (IntrinsicID == Intrinsic::spv_insertelt) {
182:         if (Reg == Use.getOperand(2).getReg())
183:           ResType = deduceTypeFromResultRegister(&Use, Reg, GR, MIB);
184:       } else if (IntrinsicID == Intrinsic::spv_extractelt) {
185:         if (Reg == Use.getOperand(2).getReg())
186:           ResType = deduceTypeFromResultRegister(&Use, Reg, GR, MIB);
187:       }
188:       break;
189:     }
190:     }
191:     if (ResType) {
192:       LLVM_DEBUG(dbgs() << "Deduced type from use " << *ResType);
193:       return ResType;
194:     }
195:   }
196:   return nullptr;
197: }
198:
199: static SPIRVTypeInst deduceGEPType(MachineInstr *I, SPIRVGlobalRegistry *GR,
200:                                    MachineIRBuilder &MIB) {
201:   LLVM_DEBUG(dbgs() << "Deducing GEP type for: " << *I);
202:   Register PtrReg = I->getOperand(3).getReg();
203:   SPIRVTypeInst PtrType = GR->getSPIRVTypeForVReg(PtrReg);
204:   if (!PtrType) {
205:     LLVM_DEBUG(dbgs() << "  Could not get type for pointer operand.\n");
206:     return nullptr;
207:   }
208:
209:   SPIRVTypeInst PointeeType = GR->getPointeeType(PtrType);
210:   if (!PointeeType) {
211:     LLVM_DEBUG(dbgs() << "  Could not get pointee type from pointer type.\n");
212:     return nullptr;
213:   }
214:
215:   MachineRegisterInfo *MRI = MIB.getMRI();
216:
217:   // The first index (operand 4) steps over the pointer, so the type doesn't
218:   // change.
219:   for (unsigned i = 5; i < I->getNumOperands(); ++i) {
220:     LLVM_DEBUG(dbgs() << "  Traversing index " << i
221:                       << ", current type: " << *PointeeType);
222:     switch (PointeeType->getOpcode()) {
223:     case SPIRV::OpTypeArray:
224:     case SPIRV::OpTypeRuntimeArray:
225:     case SPIRV::OpTypeVector: {
226:       Register ElemTypeReg = PointeeType->getOperand(1).getReg();
227:       PointeeType = GR->getSPIRVTypeForVReg(ElemTypeReg);
228:       break;
229:     }
230:     case SPIRV::OpTypeStruct: {
231:       MachineOperand &IdxOp = I->getOperand(i);
232:       if (!IdxOp.isReg()) {
233:         LLVM_DEBUG(dbgs() << "  Index is not a register.\n");
234:         return nullptr;
235:       }
236:       MachineInstr *Def = MRI->getVRegDef(IdxOp.getReg());
237:       if (!Def) {
238:         LLVM_DEBUG(
239:             dbgs() << "  Could not find definition for index register.\n");
240:         return nullptr;
```
- EN: This range implements operational logic in helpers such as deduceTypeFromResultRegister, LLVM_DEBUG, getOperand, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 deduceTypeFromResultRegister、LLVM_DEBUG、getOperand、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:       }
242:
243:       uint64_t IndexVal = foldImm(IdxOp, MRI);
244:       if (IndexVal >= PointeeType->getNumOperands() - 1) {
245:         LLVM_DEBUG(dbgs() << "  Struct index out of bounds.\n");
246:         return nullptr;
247:       }
248:
249:       Register MemberTypeReg = PointeeType->getOperand(IndexVal + 1).getReg();
250:       PointeeType = GR->getSPIRVTypeForVReg(MemberTypeReg);
251:       break;
252:     }
253:     default:
254:       LLVM_DEBUG(dbgs() << "  Unknown type opcode for GEP traversal.\n");
255:       return nullptr;
256:     }
257:
258:     if (!PointeeType) {
259:       LLVM_DEBUG(dbgs() << "  Could not resolve next pointee type.\n");
260:       return nullptr;
261:     }
262:   }
263:   LLVM_DEBUG(dbgs() << "  Final pointee type: " << *PointeeType);
264:
265:   SPIRV::StorageClass::StorageClass SC = GR->getPointerStorageClass(PtrType);
266:   SPIRVTypeInst Res = GR->getOrCreateSPIRVPointerType(PointeeType, MIB, SC);
267:   LLVM_DEBUG(dbgs() << "  Deduced GEP type: " << *Res);
268:   return Res;
269: }
270:
271: static SPIRVTypeInst deduceResultTypeFromOperands(MachineInstr *I,
272:                                                   SPIRVGlobalRegistry *GR,
273:                                                   MachineIRBuilder &MIB) {
274:   Register ResVReg = I->getOperand(0).getReg();
275:   switch (I->getOpcode()) {
276:   case TargetOpcode::G_CONSTANT:
277:   case TargetOpcode::G_ANYEXT:
278:   case TargetOpcode::G_SEXT:
279:   case TargetOpcode::G_ZEXT:
280:     return deduceIntTypeFromResult(ResVReg, MIB, GR);
281:   case TargetOpcode::G_BUILD_VECTOR:
282:     return deduceTypeFromOperandRange(I, MIB, GR, 1, I->getNumOperands());
283:   case TargetOpcode::G_SHUFFLE_VECTOR:
284:     return deduceTypeFromOperandRange(I, MIB, GR, 1, 3);
285:   case TargetOpcode::G_INTRINSIC_W_SIDE_EFFECTS:
286:   case TargetOpcode::G_INTRINSIC: {
287:     auto IntrinsicID = cast<GIntrinsic>(I)->getIntrinsicID();
288:     if (IntrinsicID == Intrinsic::spv_gep)
289:       return deduceGEPType(I, GR, MIB);
290:     break;
291:   }
292:   case TargetOpcode::G_LOAD: {
293:     SPIRVTypeInst PtrType = deduceTypeFromSingleOperand(I, MIB, GR, 1);
294:     return PtrType ? GR->getPointeeType(PtrType) : nullptr;
295:   }
296:   case TargetOpcode::G_PHI: {
297:     for (unsigned Idx = 1; Idx < I->getNumOperands(); Idx += 2) {
298:       Register OpReg = I->getOperand(Idx).getReg();
299:       if (SPIRVTypeInst OpType = GR->getSPIRVTypeForVReg(OpReg))
300:         return OpType;
```
- EN: This range implements operational logic in helpers such as foldImm, LLVM_DEBUG, getOperand, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 foldImm、LLVM_DEBUG、getOperand、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:     }
302:     return nullptr;
303:   }
304:   default:
305:     if (I->getNumDefs() == 1 && I->getNumOperands() > 1 &&
306:         I->getOperand(1).isReg())
307:       return deduceTypeFromSingleOperand(I, MIB, GR, 1);
308:   }
309:   return nullptr;
310: }
311:
312: static bool deduceAndAssignTypeForGUnmerge(MachineInstr *I, MachineFunction &MF,
313:                                            SPIRVGlobalRegistry *GR,
314:                                            MachineIRBuilder &MIB) {
315:   MachineRegisterInfo &MRI = MF.getRegInfo();
316:   Register SrcReg = I->getOperand(I->getNumOperands() - 1).getReg();
317:   SPIRVTypeInst ScalarType = nullptr;
318:   if (SPIRVTypeInst DefType = GR->getSPIRVTypeForVReg(SrcReg)) {
319:     assert(DefType->getOpcode() == SPIRV::OpTypeVector);
320:     ScalarType = GR->getScalarOrVectorComponentType(DefType);
321:   }
322:
323:   if (!ScalarType) {
324:     // If we could not deduce the type from the source, try to deduce it from
325:     // the uses of the results.
326:     for (unsigned i = 0; i < I->getNumDefs(); ++i) {
327:       Register DefReg = I->getOperand(i).getReg();
328:       ScalarType = deduceTypeFromUses(DefReg, MF, GR, MIB);
329:       if (ScalarType) {
330:         ScalarType = GR->getScalarOrVectorComponentType(ScalarType);
331:         break;
332:       }
333:     }
334:   }
335:
336:   if (!ScalarType)
337:     return false;
338:
339:   for (unsigned i = 0; i < I->getNumOperands(); ++i) {
340:     Register DefReg = I->getOperand(i).getReg();
341:     if (GR->getSPIRVTypeForVReg(DefReg))
342:       continue;
343:
344:     LLT DefLLT = MRI.getType(DefReg);
345:     SPIRVTypeInst ResType =
346:         DefLLT.isVector()
347:             ? GR->getOrCreateSPIRVVectorType(
348:                   ScalarType, DefLLT.getNumElements(), *I,
349:                   *MF.getSubtarget<SPIRVSubtarget>().getInstrInfo())
350:             : ScalarType;
351:     setRegClassType(DefReg, ResType, GR, &MRI, MF);
352:   }
353:   return true;
354: }
355:
356: static bool deduceAndAssignSpirvType(MachineInstr *I, MachineFunction &MF,
357:                                      SPIRVGlobalRegistry *GR,
358:                                      MachineIRBuilder &MIB) {
359:   LLVM_DEBUG(dbgs() << "\nProcessing instruction: " << *I);
360:   MachineRegisterInfo &MRI = MF.getRegInfo();
```
- EN: This range implements operational logic in helpers such as getOperand, deduceTypeFromSingleOperand, getRegInfo, assert, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、deduceTypeFromSingleOperand、getRegInfo、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   Register ResVReg = I->getOperand(0).getReg();
362:
363:   // G_UNMERGE_VALUES is handled separately because it has multiple definitions,
364:   // unlike the other instructions which have a single result register. The main
365:   // deduction logic is designed for the single-definition case.
366:   if (I->getOpcode() == TargetOpcode::G_UNMERGE_VALUES)
367:     return deduceAndAssignTypeForGUnmerge(I, MF, GR, MIB);
368:
369:   LLVM_DEBUG(dbgs() << "Inferring type from operands\n");
370:   SPIRVTypeInst ResType = deduceResultTypeFromOperands(I, GR, MIB);
371:   if (!ResType) {
372:     LLVM_DEBUG(dbgs() << "Inferring type from uses\n");
373:     ResType = deduceTypeFromUses(ResVReg, MF, GR, MIB);
374:   }
375:
376:   if (!ResType)
377:     return false;
378:
379:   LLVM_DEBUG(dbgs() << "Assigned type to " << *I << ": " << *ResType);
380:   setRegClassType(ResVReg, ResType, GR, &MRI, MF);
381:   return true;
382: }
383:
384: static bool requiresSpirvType(MachineInstr &I, SPIRVGlobalRegistry *GR,
385:                               MachineRegisterInfo &MRI) {
386:   LLVM_DEBUG(dbgs() << "Checking if instruction requires a SPIR-V type: "
387:                     << I;);
388:   if (I.getNumDefs() == 0) {
389:     LLVM_DEBUG(dbgs() << "Instruction does not have a definition.\n");
390:     return false;
391:   }
392:
393:   if (!I.isPreISelOpcode()) {
394:     LLVM_DEBUG(dbgs() << "Instruction is not a generic instruction.\n");
395:     return false;
396:   }
397:
398:   Register ResultRegister = I.defs().begin()->getReg();
399:   if (GR->getSPIRVTypeForVReg(ResultRegister)) {
400:     LLVM_DEBUG(dbgs() << "Instruction already has a SPIR-V type.\n");
401:     if (!MRI.getRegClassOrNull(ResultRegister)) {
402:       LLVM_DEBUG(dbgs() << "Updating the register class.\n");
403:       setRegClassType(ResultRegister, GR->getSPIRVTypeForVReg(ResultRegister),
404:                       GR, &MRI, *GR->CurMF, true);
405:     }
406:     return false;
407:   }
408:
409:   return true;
410: }
411:
412: static void registerSpirvTypeForNewInstructions(MachineFunction &MF,
413:                                                 SPIRVGlobalRegistry *GR) {
414:   MachineRegisterInfo &MRI = MF.getRegInfo();
415:   SmallVector<MachineInstr *, 8> Worklist;
416:   for (MachineBasicBlock &MBB : MF) {
417:     for (MachineInstr &I : MBB) {
418:       if (requiresSpirvType(I, GR, MRI)) {
419:         Worklist.push_back(&I);
420:       }
```
- EN: This range defines or declares important types such as getOperand, deduceAndAssignTypeForGUnmerge, LLVM_DEBUG, deduceResultTypeFromOperands, shaping the data model used by SPIRVPostLegalizer.cpp.
- CN: 这一段定义或声明了 getOperand、deduceAndAssignTypeForGUnmerge、LLVM_DEBUG、deduceResultTypeFromOperands 等关键类型，构成 SPIRVPostLegalizer.cpp 使用的数据模型。

### Lines 421-480
```cpp
421:     }
422:   }
423:
424:   if (Worklist.empty()) {
425:     LLVM_DEBUG(dbgs() << "Initial worklist is empty.\n");
426:     return;
427:   }
428:
429:   LLVM_DEBUG(dbgs() << "Initial worklist:\n";
430:              for (auto *I : Worklist) { I->dump(); });
431:
432:   bool Changed;
433:   do {
434:     Changed = false;
435:     SmallVector<MachineInstr *, 8> NextWorklist;
436:
437:     for (MachineInstr *I : Worklist) {
438:       MachineIRBuilder MIB(*I);
439:       if (deduceAndAssignSpirvType(I, MF, GR, MIB)) {
440:         Changed = true;
441:       } else {
442:         NextWorklist.push_back(I);
443:       }
444:     }
445:     Worklist = std::move(NextWorklist);
446:     LLVM_DEBUG(dbgs() << "Worklist size: " << Worklist.size() << "\n");
447:   } while (Changed);
448:
449:   if (Worklist.empty())
450:     return;
451:
452:   for (auto *I : Worklist) {
453:     MachineIRBuilder MIB(*I);
454:     LLVM_DEBUG(dbgs() << "Assigning default type to results in " << *I);
455:     for (unsigned Idx = 0; Idx < I->getNumDefs(); ++Idx) {
456:       Register ResVReg = I->getOperand(Idx).getReg();
457:       if (GR->getSPIRVTypeForVReg(ResVReg))
458:         continue;
459:       const LLT &ResLLT = MRI.getType(ResVReg);
460:       SPIRVTypeInst ResType = nullptr;
461:       if (ResLLT.isVector()) {
462:         SPIRVTypeInst CompType = GR->getOrCreateSPIRVIntegerType(
463:             ResLLT.getElementType().getSizeInBits(), MIB);
464:         ResType = GR->getOrCreateSPIRVVectorType(
465:             CompType, ResLLT.getNumElements(), MIB, false);
466:       } else {
467:         ResType = GR->getOrCreateSPIRVIntegerType(ResLLT.getSizeInBits(), MIB);
468:       }
469:       setRegClassType(ResVReg, ResType, GR, &MRI, MF, true);
470:     }
471:   }
472: }
473:
474: static bool hasAssignType(Register Reg, MachineRegisterInfo &MRI) {
475:   for (MachineInstr &UseInstr : MRI.use_nodbg_instructions(Reg)) {
476:     if (UseInstr.getOpcode() == SPIRV::ASSIGN_TYPE) {
477:       return true;
478:     }
479:   }
480:   return false;
```
- EN: This range implements operational logic in helpers such as LLVM_DEBUG, MIB, push_back, std::move, translating backend policy into executable code.
- CN: 这一段实现了 LLVM_DEBUG、MIB、push_back、std::move 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481: }
482:
483: static void generateAssignType(MachineInstr &MI, Register ResultRegister,
484:                                SPIRVTypeInst ResultType,
485:                                SPIRVGlobalRegistry *GR,
486:                                MachineRegisterInfo &MRI) {
487:   LLVM_DEBUG(dbgs() << "  Adding ASSIGN_TYPE for ResultRegister: "
488:                     << printReg(ResultRegister, MRI.getTargetRegisterInfo())
489:                     << " with type: " << *ResultType);
490:   MachineIRBuilder MIB(MI);
491:   updateRegType(ResultRegister, nullptr, ResultType, GR, MIB, MRI);
492:
493:   // Tablegen definition assumes SPIRV::ASSIGN_TYPE pseudo-instruction is
494:   // present after each auto-folded instruction to take a type reference
495:   // from.
496:   Register NewReg =
497:       MRI.createGenericVirtualRegister(MRI.getType(ResultRegister));
498:   const auto *RegClass = GR->getRegClass(ResultType);
499:   MRI.setRegClass(NewReg, RegClass);
500:   MRI.setRegClass(ResultRegister, RegClass);
501:
502:   GR->assignSPIRVTypeToVReg(ResultType, ResultRegister, MIB.getMF());
503:   // This is to make it convenient for Legalizer to get the SPIRVType
504:   // when processing the actual MI (i.e. not pseudo one).
505:   GR->assignSPIRVTypeToVReg(ResultType, NewReg, MIB.getMF());
506:   // Copy MIFlags from Def to ASSIGN_TYPE instruction. It's required to
507:   // keep the flags after instruction selection.
508:   const uint32_t Flags = MI.getFlags();
509:   MIB.buildInstr(SPIRV::ASSIGN_TYPE)
510:       .addDef(ResultRegister)
511:       .addUse(NewReg)
512:       .addUse(GR->getSPIRVTypeID(ResultType))
513:       .setMIFlags(Flags);
514:   for (unsigned I = 0, E = MI.getNumDefs(); I != E; ++I) {
515:     MachineOperand &MO = MI.getOperand(I);
516:     if (MO.getReg() == ResultRegister) {
517:       MO.setReg(NewReg);
518:       break;
519:     }
520:   }
521: }
522:
523: static void ensureAssignTypeForTypeFolding(MachineFunction &MF,
524:                                            SPIRVGlobalRegistry *GR) {
525:   LLVM_DEBUG(dbgs() << "Entering ensureAssignTypeForTypeFolding for function "
526:                     << MF.getName() << "\n");
527:   MachineRegisterInfo &MRI = MF.getRegInfo();
528:   for (MachineBasicBlock &MBB : MF) {
529:     for (MachineInstr &MI : MBB) {
530:       if (!isTypeFoldingSupported(MI.getOpcode()))
531:         continue;
532:
533:       LLVM_DEBUG(dbgs() << "Processing instruction: " << MI);
534:
535:       Register ResultRegister = MI.defs().begin()->getReg();
536:       if (hasAssignType(ResultRegister, MRI)) {
537:         LLVM_DEBUG(dbgs() << "  Instruction already has ASSIGN_TYPE\n");
538:         continue;
539:       }
540:
```
- EN: This range implements operational logic in helpers such as printReg, MIB, updateRegType, createGenericVirtualRegister, translating backend policy into executable code.
- CN: 这一段实现了 printReg、MIB、updateRegType、createGenericVirtualRegister 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-564
```cpp
541:       SPIRVTypeInst ResultType = GR->getSPIRVTypeForVReg(ResultRegister);
542:       generateAssignType(MI, ResultRegister, ResultType, GR, MRI);
543:     }
544:   }
545: }
546:
547: bool SPIRVPostLegalizer::runOnMachineFunction(MachineFunction &MF) {
548:   // Initialize the type registry.
549:   const SPIRVSubtarget &ST = MF.getSubtarget<SPIRVSubtarget>();
550:   SPIRVGlobalRegistry *GR = ST.getSPIRVGlobalRegistry();
551:   GR->setCurrentFunc(MF);
552:   registerSpirvTypeForNewInstructions(MF, GR);
553:   ensureAssignTypeForTypeFolding(MF, GR);
554:   return true;
555: }
556:
557: INITIALIZE_PASS(SPIRVPostLegalizer, DEBUG_TYPE, "SPIRV post legalizer", false,
558:                 false)
559:
560: char SPIRVPostLegalizer::ID = 0;
561:
562: FunctionPass *llvm::createSPIRVPostLegalizerPass() {
563:   return new SPIRVPostLegalizer();
564: }
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeForVReg, generateAssignType, SPIRVPostLegalizer::runOnMachineFunction, getSPIRVGlobalRegistry, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeForVReg、generateAssignType、SPIRVPostLegalizer::runOnMachineFunction、getSPIRVGlobalRegistry 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Legalization logic rewrites unsupported operations into forms that the target can handle.
  - CN: 合法化逻辑会把目标不支持的操作重写为可处理的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVPostLegalizer, runOnMachineFunction, getMRI, getOrCreateSPIRVIntegerType, getOperand, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVPostLegalizer, runOnMachineFunction, getMRI, getOrCreateSPIRVIntegerType, getOperand，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`
  - `llvm/CodeGen/MachineFrameInfo.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/Support/Debug.h`
- System/standard headers / 系统或标准头文件:
  - `stack`
