# SPIRVISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVISelLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the SPIRVTargetLowering class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- SPIRVISelLowering.cpp - SPIR-V DAG Lowering Impl ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the SPIRVTargetLowering class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "SPIRVISelLowering.h"
14: #include "SPIRV.h"
15: #include "SPIRVInstrInfo.h"
16: #include "SPIRVRegisterBankInfo.h"
17: #include "SPIRVRegisterInfo.h"
18: #include "SPIRVSubtarget.h"
19: #include "llvm/CodeGen/MachineInstrBuilder.h"
20: #include "llvm/CodeGen/MachineRegisterInfo.h"
21: #include "llvm/CodeGen/TargetLowering.h"
22: #include "llvm/IR/Instructions.h"
23: #include "llvm/IR/IntrinsicsSPIRV.h"
24:
25: #define DEBUG_TYPE "spirv-lower"
26:
27: using namespace llvm;
28:
29: SPIRVTargetLowering::SPIRVTargetLowering(const TargetMachine &TM,
30:                                          const SPIRVSubtarget &ST)
31:     : TargetLowering(TM, ST), STI(ST) {
32:   // Even with SPV_ALTERA_arbitrary_precision_integers enabled, atomic sizes are
33:   // limited by atomicrmw xchg operation, which only supports operand up to 64
34:   // bits wide, as defined in SPIR-V legalizer. Currently, spirv-val doesn't
35:   // consider 128-bit OpTypeInt as valid either.
36:   setMaxAtomicSizeInBitsSupported(64);
37:   setMinCmpXchgSizeInBits(8);
38: }
39:
40: // Returns true of the types logically match, as defined in
41: // https://registry.khronos.org/SPIR-V/specs/unified1/SPIRV.html#OpCopyLogical.
42: static bool typesLogicallyMatch(const SPIRVTypeInst Ty1,
43:                                 const SPIRVTypeInst Ty2,
44:                                 SPIRVGlobalRegistry &GR) {
45:   if (Ty1->getOpcode() != Ty2->getOpcode())
46:     return false;
47:
48:   if (Ty1->getNumOperands() != Ty2->getNumOperands())
49:     return false;
50:
51:   if (Ty1->getOpcode() == SPIRV::OpTypeArray) {
52:     // Array must have the same size.
53:     if (Ty1->getOperand(2).getReg() != Ty2->getOperand(2).getReg())
54:       return false;
55:
56:     SPIRVTypeInst ElemType1 =
57:         GR.getSPIRVTypeForVReg(Ty1->getOperand(1).getReg());
58:     SPIRVTypeInst ElemType2 =
59:         GR.getSPIRVTypeForVReg(Ty2->getOperand(1).getReg());
60:     return ElemType1 == ElemType2 ||
```
- EN: This range implements operational logic in helpers such as TargetLowering, setMaxAtomicSizeInBitsSupported, setMinCmpXchgSizeInBits, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 TargetLowering、setMaxAtomicSizeInBitsSupported、setMinCmpXchgSizeInBits、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:            typesLogicallyMatch(ElemType1, ElemType2, GR);
 62:   }
 63:
 64:   if (Ty1->getOpcode() == SPIRV::OpTypeStruct) {
 65:     for (unsigned I = 1; I < Ty1->getNumOperands(); I++) {
 66:       SPIRVTypeInst ElemType1 =
 67:           GR.getSPIRVTypeForVReg(Ty1->getOperand(I).getReg());
 68:       SPIRVTypeInst ElemType2 =
 69:           GR.getSPIRVTypeForVReg(Ty2->getOperand(I).getReg());
 70:       if (ElemType1 != ElemType2 &&
 71:           !typesLogicallyMatch(ElemType1, ElemType2, GR))
 72:         return false;
 73:     }
 74:     return true;
 75:   }
 76:   return false;
 77: }
 78:
 79: unsigned SPIRVTargetLowering::getNumRegistersForCallingConv(
 80:     LLVMContext &Context, CallingConv::ID CC, EVT VT) const {
 81:   // This code avoids CallLowering fail inside getVectorTypeBreakdown
 82:   // on v3i1 arguments. Maybe we need to return 1 for all types.
 83:   // TODO: remove it once this case is supported by the default implementation.
 84:   if (VT.isVector() && VT.getVectorNumElements() == 3 &&
 85:       (VT.getVectorElementType() == MVT::i1 ||
 86:        VT.getVectorElementType() == MVT::i8))
 87:     return 1;
 88:   if (!VT.isVector() && VT.isInteger() && VT.getSizeInBits() <= 64)
 89:     return 1;
 90:   return getNumRegisters(Context, VT);
 91: }
 92:
 93: MVT SPIRVTargetLowering::getRegisterTypeForCallingConv(LLVMContext &Context,
 94:                                                        CallingConv::ID CC,
 95:                                                        EVT VT) const {
 96:   // This code avoids CallLowering fail inside getVectorTypeBreakdown
 97:   // on v3i1 arguments. Maybe we need to return i32 for all types.
 98:   // TODO: remove it once this case is supported by the default implementation.
 99:   if (VT.isVector() && VT.getVectorNumElements() == 3) {
100:     if (VT.getVectorElementType() == MVT::i1)
101:       return MVT::v4i1;
102:     else if (VT.getVectorElementType() == MVT::i8)
103:       return MVT::v4i8;
104:   }
105:   return getRegisterType(Context, VT);
106: }
107:
108: void SPIRVTargetLowering::getTgtMemIntrinsic(
109:     SmallVectorImpl<IntrinsicInfo> &Infos, const CallBase &I,
110:     MachineFunction &MF, unsigned Intrinsic) const {
111:   IntrinsicInfo Info;
112:
113:   unsigned AlignIdx = 0;
114:   unsigned OrderingIdx = 0;
115:   unsigned FlagsIdx;
116:
117:   switch (Intrinsic) {
118:   case Intrinsic::spv_load:
119:     FlagsIdx = 1;
120:     AlignIdx = 2;
```
- EN: This range implements operational logic in helpers such as typesLogicallyMatch, getNumOperands, getSPIRVTypeForVReg, getVectorElementType, translating backend policy into executable code.
- CN: 这一段实现了 typesLogicallyMatch、getNumOperands、getSPIRVTypeForVReg、getVectorElementType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:     break;
122:   case Intrinsic::spv_store:
123:     FlagsIdx = 2;
124:     AlignIdx = 3;
125:     break;
126:   case Intrinsic::spv_atomic_load:
127:     FlagsIdx = 1;
128:     OrderingIdx = 2;
129:     break;
130:   case Intrinsic::spv_atomic_store:
131:     FlagsIdx = 2;
132:     OrderingIdx = 3;
133:     break;
134:   default:
135:     return;
136:   }
137:
138:   Info.flags = static_cast<MachineMemOperand::Flags>(
139:       cast<ConstantInt>(I.getOperand(FlagsIdx))->getZExtValue());
140:   Info.memVT = MVT::i64;
141:   // TODO: take into account opaque pointers (don't use getElementType).
142:   // MVT::getVT(PtrTy->getElementType());
143:
144:   if (AlignIdx) {
145:     auto *AlignOp = cast<ConstantInt>(I.getOperand(AlignIdx));
146:     Info.align = Align(AlignOp->getZExtValue());
147:   }
148:
149:   if (OrderingIdx) {
150:     Info.order = static_cast<AtomicOrdering>(
151:         cast<ConstantInt>(I.getOperand(OrderingIdx))->getZExtValue());
152:   }
153:   Infos.push_back(Info);
154: }
155:
156: TargetLowering::ConstraintType
157: SPIRVTargetLowering::getConstraintType(StringRef Constraint) const {
158:   // SPIR-V represents inline assembly via OpAsmINTEL where constraints are
159:   // passed through as literals defined by client API. Return C_RegisterClass
160:   // for non-memory constraints since SPIR-V does not distinguish between
161:   // register, immediate, or memory operands at this level. We do have to return
162:   // C_Memory for memory constraints as otherwise IRTranslator gets confused
163:   // trying to allocate registers for them.
164:   if (Constraint == "m")
165:     return C_Memory;
166:   return C_RegisterClass;
167: }
168:
169: std::pair<unsigned, const TargetRegisterClass *>
170: SPIRVTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
171:                                                   StringRef Constraint,
172:                                                   MVT VT) const {
173:   const TargetRegisterClass *RC = nullptr;
174:   if (Constraint.starts_with("{"))
175:     return std::make_pair(0u, RC);
176:
177:   if (VT.isFloatingPoint())
178:     RC = VT.isVector() ? &SPIRV::vfIDRegClass : &SPIRV::fIDRegClass;
179:   else if (VT.isInteger())
180:     RC = VT.isVector() ? &SPIRV::viIDRegClass : &SPIRV::iIDRegClass;
```
- EN: This range implements operational logic in helpers such as getOperand, Align, push_back, SPIRVTargetLowering::getConstraintType, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、Align、push_back、SPIRVTargetLowering::getConstraintType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   else
182:     RC = &SPIRV::iIDRegClass;
183:
184:   return std::make_pair(0u, RC);
185: }
186:
187: inline Register getTypeReg(MachineRegisterInfo *MRI, Register OpReg) {
188:   const MachineInstr *Inst = MRI->getVRegDef(OpReg);
189:   return Inst && Inst->getOpcode() == SPIRV::OpFunctionParameter
190:              ? Inst->getOperand(1).getReg()
191:              : OpReg;
192: }
193:
194: static void doInsertBitcast(const SPIRVSubtarget &STI, MachineRegisterInfo *MRI,
195:                             SPIRVGlobalRegistry &GR, MachineInstr &I,
196:                             Register OpReg, unsigned OpIdx,
197:                             SPIRVTypeInst NewPtrType) {
198:   MachineIRBuilder MIB(I);
199:   Register NewReg = createVirtualRegister(NewPtrType, &GR, MRI, MIB.getMF());
200:   MIB.buildInstr(SPIRV::OpBitcast)
201:       .addDef(NewReg)
202:       .addUse(GR.getSPIRVTypeID(NewPtrType))
203:       .addUse(OpReg)
204:       .constrainAllUses(*STI.getInstrInfo(), *STI.getRegisterInfo(),
205:                         *STI.getRegBankInfo());
206:   I.getOperand(OpIdx).setReg(NewReg);
207: }
208:
209: static SPIRVTypeInst createNewPtrType(SPIRVGlobalRegistry &GR, MachineInstr &I,
210:                                       SPIRVTypeInst OpType, bool ReuseType,
211:                                       SPIRVTypeInst ResType,
212:                                       const Type *ResTy) {
213:   SPIRV::StorageClass::StorageClass SC =
214:       static_cast<SPIRV::StorageClass::StorageClass>(
215:           OpType->getOperand(1).getImm());
216:   MachineIRBuilder MIB(I);
217:   SPIRVTypeInst NewBaseType =
218:       ReuseType ? ResType
219:                 : GR.getOrCreateSPIRVType(
220:                       ResTy, MIB, SPIRV::AccessQualifier::ReadWrite, false);
221:   return GR.getOrCreateSPIRVPointerType(NewBaseType, MIB, SC);
222: }
223:
224: // Insert a bitcast before the instruction to keep SPIR-V code valid
225: // when there is a type mismatch between results and operand types.
226: static void validatePtrTypes(const SPIRVSubtarget &STI,
227:                              MachineRegisterInfo *MRI, SPIRVGlobalRegistry &GR,
228:                              MachineInstr &I, unsigned OpIdx,
229:                              SPIRVTypeInst ResType,
230:                              const Type *ResTy = nullptr) {
231:   // Get operand type
232:   MachineFunction *MF = I.getParent()->getParent();
233:   Register OpReg = I.getOperand(OpIdx).getReg();
234:   Register OpTypeReg = getTypeReg(MRI, OpReg);
235:   const MachineInstr *OpType = GR.getSPIRVTypeForVReg(OpTypeReg, MF);
236:   if (!ResType || !OpType || OpType->getOpcode() != SPIRV::OpTypePointer)
237:     return;
238:   // Get operand's pointee type
239:   Register ElemTypeReg = OpType->getOperand(2).getReg();
240:   SPIRVTypeInst ElemType = GR.getSPIRVTypeForVReg(ElemTypeReg, MF);
```
- EN: This range implements operational logic in helpers such as std::make_pair, getTypeReg, getVRegDef, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 std::make_pair、getTypeReg、getVRegDef、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:   if (!ElemType)
242:     return;
243:   // Check if we need a bitcast to make a statement valid
244:   bool IsSameMF = MF == ResType->getParent()->getParent();
245:   bool IsEqualTypes = IsSameMF ? ElemType == ResType
246:                                : GR.getTypeForSPIRVType(ElemType) == ResTy;
247:   if (IsEqualTypes)
248:     return;
249:   // There is a type mismatch between results and operand types
250:   // and we insert a bitcast before the instruction to keep SPIR-V code valid
251:   SPIRVTypeInst NewPtrType =
252:       createNewPtrType(GR, I, OpType, IsSameMF, ResType, ResTy);
253:   if (!GR.isBitcastCompatible(NewPtrType, OpType))
254:     report_fatal_error(
255:         "insert validation bitcast: incompatible result and operand types");
256:   doInsertBitcast(STI, MRI, GR, I, OpReg, OpIdx, NewPtrType);
257: }
258:
259: // Insert a bitcast before OpGroupWaitEvents if the last argument is a pointer
260: // that doesn't point to OpTypeEvent.
261: static void validateGroupWaitEventsPtr(const SPIRVSubtarget &STI,
262:                                        MachineRegisterInfo *MRI,
263:                                        SPIRVGlobalRegistry &GR,
264:                                        MachineInstr &I) {
265:   constexpr unsigned OpIdx = 2;
266:   MachineFunction *MF = I.getParent()->getParent();
267:   Register OpReg = I.getOperand(OpIdx).getReg();
268:   Register OpTypeReg = getTypeReg(MRI, OpReg);
269:   SPIRVTypeInst OpType = GR.getSPIRVTypeForVReg(OpTypeReg, MF);
270:   if (!OpType || OpType->getOpcode() != SPIRV::OpTypePointer)
271:     return;
272:   SPIRVTypeInst ElemType =
273:       GR.getSPIRVTypeForVReg(OpType->getOperand(2).getReg());
274:   if (!ElemType || ElemType->getOpcode() == SPIRV::OpTypeEvent)
275:     return;
276:   // Insert a bitcast before the instruction to keep SPIR-V code valid.
277:   LLVMContext &Context = MF->getFunction().getContext();
278:   SPIRVTypeInst NewPtrType =
279:       createNewPtrType(GR, I, OpType, false, nullptr,
280:                        TargetExtType::get(Context, "spirv.Event"));
281:   doInsertBitcast(STI, MRI, GR, I, OpReg, OpIdx, NewPtrType);
282: }
283:
284: static void validateLifetimeStart(const SPIRVSubtarget &STI,
285:                                   MachineRegisterInfo *MRI,
286:                                   SPIRVGlobalRegistry &GR, MachineInstr &I) {
287:   Register PtrReg = I.getOperand(0).getReg();
288:   MachineFunction *MF = I.getParent()->getParent();
289:   Register PtrTypeReg = getTypeReg(MRI, PtrReg);
290:   SPIRVTypeInst PtrType = GR.getSPIRVTypeForVReg(PtrTypeReg, MF);
291:   SPIRVTypeInst PonteeElemType = PtrType ? GR.getPointeeType(PtrType) : nullptr;
292:   if (!PonteeElemType || PonteeElemType->getOpcode() == SPIRV::OpTypeVoid ||
293:       (PonteeElemType->getOpcode() == SPIRV::OpTypeInt &&
294:        PonteeElemType->getOperand(1).getImm() == 8))
295:     return;
296:   // To keep the code valid a bitcast must be inserted
297:   SPIRV::StorageClass::StorageClass SC =
298:       static_cast<SPIRV::StorageClass::StorageClass>(
299:           PtrType->getOperand(1).getImm());
300:   MachineIRBuilder MIB(I);
```
- EN: This range implements operational logic in helpers such as getParent, createNewPtrType, doInsertBitcast, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getParent、createNewPtrType、doInsertBitcast、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:   LLVMContext &Context = MF->getFunction().getContext();
302:   SPIRVTypeInst NewPtrType =
303:       GR.getOrCreateSPIRVPointerType(IntegerType::getInt8Ty(Context), MIB, SC);
304:   doInsertBitcast(STI, MRI, GR, I, PtrReg, 0, NewPtrType);
305: }
306:
307: static void validatePtrUnwrapStructField(const SPIRVSubtarget &STI,
308:                                          MachineRegisterInfo *MRI,
309:                                          SPIRVGlobalRegistry &GR,
310:                                          MachineInstr &I, unsigned OpIdx) {
311:   MachineFunction *MF = I.getParent()->getParent();
312:   Register OpReg = I.getOperand(OpIdx).getReg();
313:   Register OpTypeReg = getTypeReg(MRI, OpReg);
314:   SPIRVTypeInst OpType = GR.getSPIRVTypeForVReg(OpTypeReg, MF);
315:   if (!OpType || OpType->getOpcode() != SPIRV::OpTypePointer)
316:     return;
317:   SPIRVTypeInst ElemType =
318:       GR.getSPIRVTypeForVReg(OpType->getOperand(2).getReg());
319:   if (!ElemType || ElemType->getOpcode() != SPIRV::OpTypeStruct ||
320:       ElemType->getNumOperands() != 2)
321:     return;
322:   // It's a structure-wrapper around another type with a single member field.
323:   SPIRVTypeInst MemberType =
324:       GR.getSPIRVTypeForVReg(ElemType->getOperand(1).getReg());
325:   if (!MemberType)
326:     return;
327:   unsigned MemberTypeOp = MemberType->getOpcode();
328:   if (MemberTypeOp != SPIRV::OpTypeVector && MemberTypeOp != SPIRV::OpTypeInt &&
329:       MemberTypeOp != SPIRV::OpTypeFloat && MemberTypeOp != SPIRV::OpTypeBool)
330:     return;
331:   // It's a structure-wrapper around a valid type. Insert a bitcast before the
332:   // instruction to keep SPIR-V code valid.
333:   SPIRV::StorageClass::StorageClass SC =
334:       static_cast<SPIRV::StorageClass::StorageClass>(
335:           OpType->getOperand(1).getImm());
336:   MachineIRBuilder MIB(I);
337:   SPIRVTypeInst NewPtrType =
338:       GR.getOrCreateSPIRVPointerType(MemberType, MIB, SC);
339:   doInsertBitcast(STI, MRI, GR, I, OpReg, OpIdx, NewPtrType);
340: }
341:
342: // Insert a bitcast before the function call instruction to keep SPIR-V code
343: // valid when there is a type mismatch between actual and expected types of an
344: // argument:
345: // %formal = OpFunctionParameter %formal_type
346: // ...
347: // %res = OpFunctionCall %ty %fun %actual ...
348: // implies that %actual is of %formal_type, and in case of opaque pointers.
349: // We may need to insert a bitcast to ensure this.
350: void validateFunCallMachineDef(const SPIRVSubtarget &STI,
351:                                MachineRegisterInfo *DefMRI,
352:                                MachineRegisterInfo *CallMRI,
353:                                SPIRVGlobalRegistry &GR, MachineInstr &FunCall,
354:                                MachineInstr *FunDef) {
355:   if (FunDef->getOpcode() != SPIRV::OpFunction)
356:     return;
357:   unsigned OpIdx = 3;
358:   for (FunDef = FunDef->getNextNode();
359:        FunDef && FunDef->getOpcode() == SPIRV::OpFunctionParameter &&
360:        OpIdx < FunCall.getNumOperands();
```
- EN: This range implements operational logic in helpers such as getFunction, getOrCreateSPIRVPointerType, doInsertBitcast, getParent, translating backend policy into executable code.
- CN: 这一段实现了 getFunction、getOrCreateSPIRVPointerType、doInsertBitcast、getParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:        FunDef = FunDef->getNextNode(), OpIdx++) {
362:     SPIRVTypeInst DefPtrType =
363:         DefMRI->getVRegDef(FunDef->getOperand(1).getReg());
364:     SPIRVTypeInst DefElemType =
365:         DefPtrType && DefPtrType->getOpcode() == SPIRV::OpTypePointer
366:             ? GR.getSPIRVTypeForVReg(DefPtrType->getOperand(2).getReg(),
367:                                      DefPtrType->getParent()->getParent())
368:             : nullptr;
369:     if (DefElemType) {
370:       const Type *DefElemTy = GR.getTypeForSPIRVType(DefElemType);
371:       // validatePtrTypes() works in the context if the call site
372:       // When we process historical records about forward calls
373:       // we need to switch context to the (forward) call site and
374:       // then restore it back to the current machine function.
375:       MachineFunction *CurMF =
376:           GR.setCurrentFunc(*FunCall.getParent()->getParent());
377:       validatePtrTypes(STI, CallMRI, GR, FunCall, OpIdx, DefElemType,
378:                        DefElemTy);
379:       GR.setCurrentFunc(*CurMF);
380:     }
381:   }
382: }
383:
384: // Ensure there is no mismatch between actual and expected arg types: calls
385: // with a processed definition. Return Function pointer if it's a forward
386: // call (ahead of definition), and nullptr otherwise.
387: const Function *validateFunCall(const SPIRVSubtarget &STI,
388:                                 MachineRegisterInfo *CallMRI,
389:                                 SPIRVGlobalRegistry &GR,
390:                                 MachineInstr &FunCall) {
391:   const GlobalValue *GV = FunCall.getOperand(2).getGlobal();
392:   const Function *F = dyn_cast<Function>(GV);
393:   MachineInstr *FunDef =
394:       const_cast<MachineInstr *>(GR.getFunctionDefinition(F));
395:   if (!FunDef)
396:     return F;
397:   MachineRegisterInfo *DefMRI = &FunDef->getParent()->getParent()->getRegInfo();
398:   validateFunCallMachineDef(STI, DefMRI, CallMRI, GR, FunCall, FunDef);
399:   return nullptr;
400: }
401:
402: // Ensure there is no mismatch between actual and expected arg types: calls
403: // ahead of a processed definition.
404: void validateForwardCalls(const SPIRVSubtarget &STI,
405:                           MachineRegisterInfo *DefMRI, SPIRVGlobalRegistry &GR,
406:                           MachineInstr &FunDef) {
407:   const Function *F = GR.getFunctionByDefinition(&FunDef);
408:   if (SmallPtrSet<MachineInstr *, 8> *FwdCalls = GR.getForwardCalls(F))
409:     for (MachineInstr *FunCall : *FwdCalls) {
410:       MachineRegisterInfo *CallMRI =
411:           &FunCall->getParent()->getParent()->getRegInfo();
412:       validateFunCallMachineDef(STI, DefMRI, CallMRI, GR, *FunCall, &FunDef);
413:     }
414: }
415:
416: // Validation of an access chain.
417: void validateAccessChain(const SPIRVSubtarget &STI, MachineRegisterInfo *MRI,
418:                          SPIRVGlobalRegistry &GR, MachineInstr &I) {
419:   SPIRVTypeInst BaseTypeInst = GR.getSPIRVTypeForVReg(I.getOperand(0).getReg());
420:   if (BaseTypeInst && BaseTypeInst->getOpcode() == SPIRV::OpTypePointer) {
```
- EN: This range implements operational logic in helpers such as getNextNode, getVRegDef, getParent, getTypeForSPIRVType, translating backend policy into executable code.
- CN: 这一段实现了 getNextNode、getVRegDef、getParent、getTypeForSPIRVType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:     SPIRVTypeInst BaseElemType =
422:         GR.getSPIRVTypeForVReg(BaseTypeInst->getOperand(2).getReg());
423:     validatePtrTypes(STI, MRI, GR, I, 2, BaseElemType);
424:   }
425: }
426:
427: // TODO: the logic of inserting additional bitcast's is to be moved
428: // to pre-IRTranslation passes eventually
429: void SPIRVTargetLowering::finalizeLowering(MachineFunction &MF) const {
430:   // finalizeLowering() is called twice (see GlobalISel/InstructionSelect.cpp)
431:   // We'd like to avoid the needless second processing pass.
432:   if (MF.getRegInfo().reservedRegsFrozen())
433:     return;
434:
435:   MachineRegisterInfo *MRI = &MF.getRegInfo();
436:   SPIRVGlobalRegistry &GR = *STI.getSPIRVGlobalRegistry();
437:   GR.setCurrentFunc(MF);
438:   for (MachineFunction::iterator I = MF.begin(), E = MF.end(); I != E; ++I) {
439:     MachineBasicBlock *MBB = &*I;
440:     for (MachineBasicBlock::iterator MBBI = MBB->begin(), MBBE = MBB->end();
441:          MBBI != MBBE;) {
442:       MachineInstr &MI = *MBBI++;
443:       switch (MI.getOpcode()) {
444:       case SPIRV::OpAtomicLoad:
445:       case SPIRV::OpAtomicExchange:
446:       case SPIRV::OpAtomicCompareExchange:
447:       case SPIRV::OpAtomicCompareExchangeWeak:
448:       case SPIRV::OpAtomicIIncrement:
449:       case SPIRV::OpAtomicIDecrement:
450:       case SPIRV::OpAtomicIAdd:
451:       case SPIRV::OpAtomicISub:
452:       case SPIRV::OpAtomicSMin:
453:       case SPIRV::OpAtomicUMin:
454:       case SPIRV::OpAtomicSMax:
455:       case SPIRV::OpAtomicUMax:
456:       case SPIRV::OpAtomicAnd:
457:       case SPIRV::OpAtomicOr:
458:       case SPIRV::OpAtomicXor:
459:         // for the above listed instructions
460:         // OpAtomicXXX <ResType>, ptr %Op, ...
461:         // implies that %Op is a pointer to <ResType>
462:       case SPIRV::OpLoad:
463:         // OpLoad <ResType>, ptr %Op implies that %Op is a pointer to <ResType>
464:         if (enforcePtrTypeCompatibility(MI, 2, 0))
465:           break;
466:
467:         validatePtrTypes(STI, MRI, GR, MI, 2,
468:                          GR.getSPIRVTypeForVReg(MI.getOperand(0).getReg()));
469:         break;
470:       case SPIRV::OpAtomicStore:
471:         // OpAtomicStore ptr %Op, <Scope>, <Mem>, <Obj>
472:         // implies that %Op points to the <Obj>'s type
473:         validatePtrTypes(STI, MRI, GR, MI, 0,
474:                          GR.getSPIRVTypeForVReg(MI.getOperand(3).getReg()));
475:         break;
476:       case SPIRV::OpStore:
477:         // OpStore ptr %Op, <Obj> implies that %Op points to the <Obj>'s type
478:         validatePtrTypes(STI, MRI, GR, MI, 0,
479:                          GR.getSPIRVTypeForVReg(MI.getOperand(1).getReg()));
480:         break;
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeForVReg, validatePtrTypes, SPIRVTargetLowering::finalizeLowering, getRegInfo, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeForVReg、validatePtrTypes、SPIRVTargetLowering::finalizeLowering、getRegInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:       case SPIRV::OpPtrCastToGeneric:
482:       case SPIRV::OpGenericCastToPtr:
483:       case SPIRV::OpGenericCastToPtrExplicit:
484:         validateAccessChain(STI, MRI, GR, MI);
485:         break;
486:       case SPIRV::OpPtrAccessChain:
487:       case SPIRV::OpInBoundsPtrAccessChain:
488:         if (MI.getNumOperands() == 4)
489:           validateAccessChain(STI, MRI, GR, MI);
490:         break;
491:
492:       case SPIRV::OpFunctionCall:
493:         // ensure there is no mismatch between actual and expected arg types:
494:         // calls with a processed definition
495:         if (MI.getNumOperands() > 3)
496:           if (const Function *F = validateFunCall(STI, MRI, GR, MI))
497:             GR.addForwardCall(F, &MI);
498:         break;
499:       case SPIRV::OpFunction:
500:         // ensure there is no mismatch between actual and expected arg types:
501:         // calls ahead of a processed definition
502:         validateForwardCalls(STI, MRI, GR, MI);
503:         break;
504:
505:       // ensure that LLVM IR add/sub instructions result in logical SPIR-V
506:       // instructions when applied to bool type
507:       case SPIRV::OpIAddS:
508:       case SPIRV::OpIAddV:
509:       case SPIRV::OpISubS:
510:       case SPIRV::OpISubV:
511:         if (GR.isScalarOrVectorOfType(MI.getOperand(1).getReg(),
512:                                       SPIRV::OpTypeBool))
513:           MI.setDesc(STI.getInstrInfo()->get(SPIRV::OpLogicalNotEqual));
514:         break;
515:
516:       // ensure that LLVM IR bitwise instructions result in logical SPIR-V
517:       // instructions when applied to bool type
518:       case SPIRV::OpBitwiseOrS:
519:       case SPIRV::OpBitwiseOrV:
520:         if (GR.isScalarOrVectorOfType(MI.getOperand(1).getReg(),
521:                                       SPIRV::OpTypeBool))
522:           MI.setDesc(STI.getInstrInfo()->get(SPIRV::OpLogicalOr));
523:         break;
524:       case SPIRV::OpBitwiseAndS:
525:       case SPIRV::OpBitwiseAndV:
526:         if (GR.isScalarOrVectorOfType(MI.getOperand(1).getReg(),
527:                                       SPIRV::OpTypeBool))
528:           MI.setDesc(STI.getInstrInfo()->get(SPIRV::OpLogicalAnd));
529:         break;
530:       case SPIRV::OpBitwiseXorS:
531:       case SPIRV::OpBitwiseXorV:
532:         if (GR.isScalarOrVectorOfType(MI.getOperand(1).getReg(),
533:                                       SPIRV::OpTypeBool))
534:           MI.setDesc(STI.getInstrInfo()->get(SPIRV::OpLogicalNotEqual));
535:         break;
536:       case SPIRV::OpLifetimeStart:
537:       case SPIRV::OpLifetimeStop:
538:         if (MI.getOperand(1).getImm() > 0)
539:           validateLifetimeStart(STI, MRI, GR, MI);
540:         break;
```
- EN: This range implements operational logic in helpers such as validateAccessChain, addForwardCall, validateForwardCalls, setDesc, translating backend policy into executable code.
- CN: 这一段实现了 validateAccessChain、addForwardCall、validateForwardCalls、setDesc 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:       case SPIRV::OpGroupAsyncCopy:
542:         validatePtrUnwrapStructField(STI, MRI, GR, MI, 3);
543:         validatePtrUnwrapStructField(STI, MRI, GR, MI, 4);
544:         break;
545:       case SPIRV::OpGroupWaitEvents:
546:         // OpGroupWaitEvents ..., ..., <pointer to OpTypeEvent>
547:         validateGroupWaitEventsPtr(STI, MRI, GR, MI);
548:         break;
549:       case SPIRV::OpConstantI: {
550:         SPIRVTypeInst Type = GR.getSPIRVTypeForVReg(MI.getOperand(1).getReg());
551:         if (Type->getOpcode() != SPIRV::OpTypeInt && MI.getOperand(2).isImm() &&
552:             MI.getOperand(2).getImm() == 0) {
553:           // Validate the null constant of a target extension type
554:           MI.setDesc(STI.getInstrInfo()->get(SPIRV::OpConstantNull));
555:           for (unsigned i = MI.getNumOperands() - 1; i > 1; --i)
556:             MI.removeOperand(i);
557:         }
558:       } break;
559:       case SPIRV::OpExtInst: {
560:         // prefetch
561:         if (!MI.getOperand(2).isImm() || !MI.getOperand(3).isImm() ||
562:             MI.getOperand(2).getImm() != SPIRV::InstructionSet::OpenCL_std)
563:           continue;
564:         switch (MI.getOperand(3).getImm()) {
565:         case SPIRV::OpenCLExtInst::frexp:
566:         case SPIRV::OpenCLExtInst::lgamma_r:
567:         case SPIRV::OpenCLExtInst::remquo: {
568:           // The last operand must be of a pointer to i32 or vector of i32
569:           // values.
570:           MachineIRBuilder MIB(MI);
571:           SPIRVTypeInst Int32Type = GR.getOrCreateSPIRVIntegerType(32, MIB);
572:           SPIRVTypeInst RetType = MRI->getVRegDef(MI.getOperand(1).getReg());
573:           assert(RetType && "Expected return type");
574:           validatePtrTypes(
575:               STI, MRI, GR, MI, MI.getNumOperands() - 1,
576:               RetType->getOpcode() != SPIRV::OpTypeVector
577:                   ? Int32Type
578:                   : GR.getOrCreateSPIRVVectorType(
579:                         Int32Type, GR.getScalarOrVectorComponentCount(RetType),
580:                         MIB, false));
581:         } break;
582:         case SPIRV::OpenCLExtInst::fract:
583:         case SPIRV::OpenCLExtInst::modf:
584:         case SPIRV::OpenCLExtInst::sincos:
585:           // The last operand must be of a pointer to the base type represented
586:           // by the previous operand.
587:           assert(MI.getOperand(MI.getNumOperands() - 2).isReg() &&
588:                  "Expected v-reg");
589:           validatePtrTypes(
590:               STI, MRI, GR, MI, MI.getNumOperands() - 1,
591:               GR.getSPIRVTypeForVReg(
592:                   MI.getOperand(MI.getNumOperands() - 2).getReg()));
593:           break;
594:         case SPIRV::OpenCLExtInst::prefetch:
595:           // Expected `ptr` type is a pointer to float, integer or vector, but
596:           // the pontee value can be wrapped into a struct.
597:           assert(MI.getOperand(MI.getNumOperands() - 2).isReg() &&
598:                  "Expected v-reg");
599:           validatePtrUnwrapStructField(STI, MRI, GR, MI,
600:                                        MI.getNumOperands() - 2);
```
- EN: This range implements operational logic in helpers such as validatePtrUnwrapStructField, validateGroupWaitEventsPtr, getSPIRVTypeForVReg, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 validatePtrUnwrapStructField、validateGroupWaitEventsPtr、getSPIRVTypeForVReg、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-660
```cpp
601:           break;
602:         }
603:       } break;
604:       }
605:     }
606:   }
607:   TargetLowering::finalizeLowering(MF);
608: }
609:
610: // Modifies either operand PtrOpIdx or OpIdx so that the pointee type of
611: // PtrOpIdx matches the type for operand OpIdx. Returns true if they already
612: // match or if the instruction was modified to make them match.
613: bool SPIRVTargetLowering::enforcePtrTypeCompatibility(
614:     MachineInstr &I, unsigned int PtrOpIdx, unsigned int OpIdx) const {
615:   SPIRVGlobalRegistry &GR = *STI.getSPIRVGlobalRegistry();
616:   SPIRVTypeInst PtrType = GR.getResultType(I.getOperand(PtrOpIdx).getReg());
617:   SPIRVTypeInst PointeeType = GR.getPointeeType(PtrType);
618:   SPIRVTypeInst OpType = GR.getResultType(I.getOperand(OpIdx).getReg());
619:
620:   if (PointeeType == OpType)
621:     return true;
622:
623:   if (typesLogicallyMatch(PointeeType, OpType, GR)) {
624:     // Apply OpCopyLogical to OpIdx.
625:     if (I.getOperand(OpIdx).isDef() &&
626:         insertLogicalCopyOnResult(I, PointeeType)) {
627:       return true;
628:     }
629:
630:     llvm_unreachable("Unable to add OpCopyLogical yet.");
631:     return false;
632:   }
633:
634:   return false;
635: }
636:
637: bool SPIRVTargetLowering::insertLogicalCopyOnResult(
638:     MachineInstr &I, SPIRVTypeInst NewResultType) const {
639:   MachineRegisterInfo *MRI = &I.getMF()->getRegInfo();
640:   SPIRVGlobalRegistry &GR = *STI.getSPIRVGlobalRegistry();
641:
642:   Register NewResultReg =
643:       createVirtualRegister(NewResultType, &GR, MRI, *I.getMF());
644:   Register NewTypeReg = GR.getSPIRVTypeID(NewResultType);
645:
646:   assert(llvm::size(I.defs()) == 1 && "Expected only one def");
647:   MachineOperand &OldResult = *I.defs().begin();
648:   Register OldResultReg = OldResult.getReg();
649:   MachineOperand &OldType = *I.uses().begin();
650:   Register OldTypeReg = OldType.getReg();
651:
652:   OldResult.setReg(NewResultReg);
653:   OldType.setReg(NewTypeReg);
654:
655:   MachineIRBuilder MIB(*I.getNextNode());
656:   MIB.buildInstr(SPIRV::OpCopyLogical)
657:       .addDef(OldResultReg)
658:       .addUse(OldTypeReg)
659:       .addUse(NewResultReg)
660:       .constrainAllUses(*STI.getInstrInfo(), *STI.getRegisterInfo(),
```
- EN: This range implements operational logic in helpers such as TargetLowering::finalizeLowering, getSPIRVGlobalRegistry, getResultType, getPointeeType, translating backend policy into executable code.
- CN: 这一段实现了 TargetLowering::finalizeLowering、getSPIRVGlobalRegistry、getResultType、getPointeeType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 661-686
```cpp
661:                         *STI.getRegBankInfo());
662:   return true;
663: }
664:
665: TargetLowering::AtomicExpansionKind
666: SPIRVTargetLowering::shouldExpandAtomicRMWInIR(const AtomicRMWInst *RMW) const {
667:   switch (RMW->getOperation()) {
668:   case AtomicRMWInst::FAdd:
669:   case AtomicRMWInst::FSub:
670:   case AtomicRMWInst::FMin:
671:   case AtomicRMWInst::FMax:
672:     return AtomicExpansionKind::None;
673:   case AtomicRMWInst::UIncWrap:
674:   case AtomicRMWInst::UDecWrap:
675:     return AtomicExpansionKind::CmpXChg;
676:   default:
677:     return TargetLowering::shouldExpandAtomicRMWInIR(RMW);
678:   }
679: }
680:
681: TargetLowering::AtomicExpansionKind
682: SPIRVTargetLowering::shouldCastAtomicRMWIInIR(AtomicRMWInst *RMWI) const {
683:   // TODO: Pointer operand should be cast to integer in atomicrmw xchg, since
684:   // SPIR-V only supports atomic exchange for integer and floating-point types.
685:   return AtomicExpansionKind::None;
686: }
```
- EN: This range implements operational logic in helpers such as SPIRVTargetLowering::shouldExpandAtomicRMWInIR, TargetLowering::shouldExpandAtomicRMWInIR, SPIRVTargetLowering::shouldCastAtomicRMWIInIR, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVTargetLowering::shouldExpandAtomicRMWInIR、TargetLowering::shouldExpandAtomicRMWInIR、SPIRVTargetLowering::shouldCastAtomicRMWIInIR 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Instruction selection maps generic LLVM operations onto target-specific machine instructions.
  - CN: 指令选择负责把通用 LLVM 操作映射为目标相关的机器指令。
- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include TargetLowering, setMaxAtomicSizeInBitsSupported, setMinCmpXchgSizeInBits, getSPIRVTypeForVReg, typesLogicallyMatch, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 TargetLowering, setMaxAtomicSizeInBitsSupported, setMinCmpXchgSizeInBits, getSPIRVTypeForVReg, typesLogicallyMatch，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVISelLowering.h`
  - `SPIRV.h`
  - `SPIRVInstrInfo.h`
  - `SPIRVRegisterBankInfo.h`
  - `SPIRVRegisterInfo.h`
  - `SPIRVSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineInstrBuilder.h`
  - `llvm/CodeGen/MachineRegisterInfo.h`
  - `llvm/CodeGen/TargetLowering.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
