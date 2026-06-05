# SPIRVGlobalRegistry.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVGlobalRegistry.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the implementation of the SPIRVGlobalRegistry class, which is used to maintain rich type information required for SPIR-V even after lowering from LLVM IR to GMIR.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===-- SPIRVGlobalRegistry.cpp - SPIR-V Global Registry --------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the implementation of the SPIRVGlobalRegistry class,
10: // which is used to maintain rich type information required for SPIR-V even
11: // after lowering from LLVM IR to GMIR. It can convert an llvm::Type into
12: // an OpTypeXXX instruction, and map it to a virtual register. Also it builds
13: // and supports consistency of constants and global variables.
14: //
15: //===----------------------------------------------------------------------===//
16:
17: #include "SPIRVGlobalRegistry.h"
18: #include "SPIRV.h"
19: #include "SPIRVBuiltins.h"
20: #include "SPIRVSubtarget.h"
21: #include "SPIRVUtils.h"
22: #include "llvm/ADT/APInt.h"
23: #include "llvm/IR/Constants.h"
24: #include "llvm/IR/DiagnosticInfo.h"
25: #include "llvm/IR/Function.h"
26: #include "llvm/IR/IntrinsicInst.h"
27: #include "llvm/IR/Intrinsics.h"
28: #include "llvm/IR/IntrinsicsSPIRV.h"
29: #include "llvm/IR/Type.h"
30: #include "llvm/Support/Casting.h"
31: #include "llvm/Support/MathExtras.h"
32: #include <cassert>
33: #include <functional>
34:
35: using namespace llvm;
36:
37: static bool allowEmitFakeUse(const Value *Arg) {
38:   if (isSpvIntrinsic(Arg))
39:     return false;
40:   if (isa<AtomicCmpXchgInst, InsertValueInst, UndefValue>(Arg))
41:     return false;
42:   if (const auto *LI = dyn_cast<LoadInst>(Arg))
43:     if (LI->getType()->isAggregateType())
44:       return false;
45:   return true;
46: }
47:
48: static unsigned typeToAddressSpace(const Type *Ty) {
49:   if (auto PType = dyn_cast<TypedPointerType>(Ty))
50:     return PType->getAddressSpace();
51:   if (auto PType = dyn_cast<PointerType>(Ty))
52:     return PType->getAddressSpace();
53:   if (auto *ExtTy = dyn_cast<TargetExtType>(Ty);
54:       ExtTy && isTypedPointerWrapper(ExtTy))
55:     return ExtTy->getIntParameter(0);
56:   reportFatalInternalError("Unable to convert LLVM type to SPIRVType");
57: }
58:
59: static bool
60: storageClassRequiresExplictLayout(SPIRV::StorageClass::StorageClass SC) {
61:   switch (SC) {
62:   case SPIRV::StorageClass::Uniform:
63:   case SPIRV::StorageClass::PushConstant:
64:   case SPIRV::StorageClass::StorageBuffer:
65:   case SPIRV::StorageClass::PhysicalStorageBufferEXT:
66:     return true;
67:   case SPIRV::StorageClass::UniformConstant:
68:   case SPIRV::StorageClass::Input:
69:   case SPIRV::StorageClass::Output:
70:   case SPIRV::StorageClass::Workgroup:
71:   case SPIRV::StorageClass::CrossWorkgroup:
72:   case SPIRV::StorageClass::Private:
73:   case SPIRV::StorageClass::Function:
74:   case SPIRV::StorageClass::Generic:
75:   case SPIRV::StorageClass::AtomicCounter:
76:   case SPIRV::StorageClass::Image:
77:   case SPIRV::StorageClass::CallableDataNV:
78:   case SPIRV::StorageClass::IncomingCallableDataNV:
79:   case SPIRV::StorageClass::RayPayloadNV:
80:   case SPIRV::StorageClass::HitAttributeNV:
81:   case SPIRV::StorageClass::IncomingRayPayloadNV:
82:   case SPIRV::StorageClass::ShaderRecordBufferNV:
83:   case SPIRV::StorageClass::CodeSectionINTEL:
84:   case SPIRV::StorageClass::DeviceOnlyINTEL:
85:   case SPIRV::StorageClass::HostOnlyINTEL:
86:     return false;
87:   }
88:   llvm_unreachable("Unknown SPIRV::StorageClass enum");
89: }
90:
```
- EN: This range defines or declares important types such as allowEmitFakeUse, typeToAddressSpace, getAddressSpace, isTypedPointerWrapper, shaping the data model used by SPIRVGlobalRegistry.cpp.
- CN: 这一段定义或声明了 allowEmitFakeUse、typeToAddressSpace、getAddressSpace、isTypedPointerWrapper 等关键类型，构成 SPIRVGlobalRegistry.cpp 使用的数据模型。

### Lines 91-180
```cpp
 91: SPIRVGlobalRegistry::SPIRVGlobalRegistry(DataLayout DL)
 92:     : DL(DL), Bound(0), CurMF(nullptr) {}
 93:
 94: SPIRVTypeInst
 95: SPIRVGlobalRegistry::assignIntTypeToVReg(unsigned BitWidth, Register VReg,
 96:                                          MachineInstr &I,
 97:                                          const SPIRVInstrInfo &TII) {
 98:   SPIRVTypeInst SpirvType = getOrCreateSPIRVIntegerType(BitWidth, I, TII);
 99:   assignSPIRVTypeToVReg(SpirvType, VReg, *CurMF);
100:   return SpirvType;
101: }
102:
103: SPIRVTypeInst
104: SPIRVGlobalRegistry::assignFloatTypeToVReg(unsigned BitWidth, Register VReg,
105:                                            MachineInstr &I,
106:                                            const SPIRVInstrInfo &TII) {
107:   SPIRVTypeInst SpirvType = getOrCreateSPIRVFloatType(BitWidth, I, TII);
108:   assignSPIRVTypeToVReg(SpirvType, VReg, *CurMF);
109:   return SpirvType;
110: }
111:
112: SPIRVTypeInst SPIRVGlobalRegistry::assignVectTypeToVReg(
113:     SPIRVTypeInst BaseType, unsigned NumElements, Register VReg,
114:     MachineInstr &I, const SPIRVInstrInfo &TII) {
115:   SPIRVTypeInst SpirvType =
116:       getOrCreateSPIRVVectorType(BaseType, NumElements, I, TII);
117:   assignSPIRVTypeToVReg(SpirvType, VReg, *CurMF);
118:   return SpirvType;
119: }
120:
121: SPIRVTypeInst SPIRVGlobalRegistry::assignTypeToVReg(
122:     const Type *Type, Register VReg, MachineIRBuilder &MIRBuilder,
123:     SPIRV::AccessQualifier::AccessQualifier AccessQual, bool EmitIR) {
124:   SPIRVTypeInst SpirvType =
125:       getOrCreateSPIRVType(Type, MIRBuilder, AccessQual, EmitIR);
126:   assignSPIRVTypeToVReg(SpirvType, VReg, MIRBuilder.getMF());
127:   return SpirvType;
128: }
129:
130: void SPIRVGlobalRegistry::assignSPIRVTypeToVReg(SPIRVTypeInst SpirvType,
131:                                                 Register VReg,
132:                                                 const MachineFunction &MF) {
133:   VRegToTypeMap[&MF][VReg] = SpirvType;
134: }
135:
136: static Register createTypeVReg(MachineRegisterInfo &MRI) {
137:   auto Res = MRI.createGenericVirtualRegister(LLT::scalar(64));
138:   MRI.setRegClass(Res, &SPIRV::TYPERegClass);
139:   return Res;
140: }
141:
142: inline Register createTypeVReg(MachineIRBuilder &MIRBuilder) {
143:   return createTypeVReg(MIRBuilder.getMF().getRegInfo());
144: }
145:
146: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeBool(MachineIRBuilder &MIRBuilder) {
147:   return createConstOrTypeAtFunctionEntry(
148:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
149:         return MIRBuilder.buildInstr(SPIRV::OpTypeBool)
150:             .addDef(createTypeVReg(MIRBuilder));
151:       });
152: }
153:
154: unsigned SPIRVGlobalRegistry::adjustOpTypeIntWidth(unsigned Width) const {
155:   const SPIRVSubtarget &ST = cast<SPIRVSubtarget>(CurMF->getSubtarget());
156:   if (ST.canUseExtension(
157:           SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers) ||
158:       (Width == 4 && ST.canUseExtension(SPIRV::Extension::SPV_INTEL_int4)))
159:     return Width;
160:   if (Width <= 8)
161:     return 8;
162:   else if (Width <= 16)
163:     return 16;
164:   else if (Width <= 32)
165:     return 32;
166:   else if (Width <= 64)
167:     return 64;
168:   else if (Width <= 128)
169:     return 128;
170:   reportFatalUsageError("Unsupported Integer width!");
171: }
172:
173: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeInt(unsigned Width,
174:                                                 MachineIRBuilder &MIRBuilder,
175:                                                 bool IsSigned) {
176:   Width = adjustOpTypeIntWidth(Width);
177:   const SPIRVSubtarget &ST =
178:       cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget());
179:   return createConstOrTypeAtFunctionEntry(MIRBuilder, [&](MachineIRBuilder
180:                                                               &MIRBuilder) {
```
- EN: This range implements operational logic in helpers such as SPIRVGlobalRegistry::SPIRVGlobalRegistry, DL, getOrCreateSPIRVIntegerType, assignSPIRVTypeToVReg, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVGlobalRegistry::SPIRVGlobalRegistry、DL、getOrCreateSPIRVIntegerType、assignSPIRVTypeToVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:     if (Width == 4 && ST.canUseExtension(SPIRV::Extension::SPV_INTEL_int4)) {
182:       MIRBuilder.buildInstr(SPIRV::OpExtension)
183:           .addImm(SPIRV::Extension::SPV_INTEL_int4);
184:       MIRBuilder.buildInstr(SPIRV::OpCapability)
185:           .addImm(SPIRV::Capability::Int4TypeINTEL);
186:     } else if ((!isPowerOf2_32(Width) || Width < 8) &&
187:                ST.canUseExtension(
188:                    SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers)) {
189:       MIRBuilder.buildInstr(SPIRV::OpExtension)
190:           .addImm(SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers);
191:       MIRBuilder.buildInstr(SPIRV::OpCapability)
192:           .addImm(SPIRV::Capability::ArbitraryPrecisionIntegersALTERA);
193:     }
194:     return MIRBuilder.buildInstr(SPIRV::OpTypeInt)
195:         .addDef(createTypeVReg(MIRBuilder))
196:         .addImm(Width)
197:         .addImm(IsSigned ? 1 : 0);
198:   });
199: }
200:
201: SPIRVTypeInst
202: SPIRVGlobalRegistry::getOpTypeFloat(uint32_t Width,
203:                                     MachineIRBuilder &MIRBuilder) {
204:   return createConstOrTypeAtFunctionEntry(MIRBuilder, [&](MachineIRBuilder
205:                                                               &MIRBuilder) {
206:     return MIRBuilder.buildInstr(SPIRV::OpTypeFloat)
207:         .addDef(createTypeVReg(MIRBuilder))
208:         .addImm(Width);
209:   });
210: }
211:
212: SPIRVTypeInst
213: SPIRVGlobalRegistry::getOpTypeFloat(uint32_t Width,
214:                                     MachineIRBuilder &MIRBuilder,
215:                                     SPIRV::FPEncoding::FPEncoding FPEncode) {
216:   return createConstOrTypeAtFunctionEntry(MIRBuilder, [&](MachineIRBuilder
217:                                                               &MIRBuilder) {
218:     return MIRBuilder.buildInstr(SPIRV::OpTypeFloat)
219:         .addDef(createTypeVReg(MIRBuilder))
220:         .addImm(Width)
221:         .addImm(FPEncode);
222:   });
223: }
224:
225: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeVoid(MachineIRBuilder &MIRBuilder) {
226:   return createConstOrTypeAtFunctionEntry(
227:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
228:         return MIRBuilder.buildInstr(SPIRV::OpTypeVoid)
229:             .addDef(createTypeVReg(MIRBuilder));
230:       });
231: }
232:
233: void SPIRVGlobalRegistry::invalidateMachineInstr(MachineInstr *MI) {
234:   // Other maps that may hold MachineInstr*:
235:   // - VRegToTypeMap: We cannot remove the definitions of `MI` from
236:   // VRegToTypeMap because some calls to invalidateMachineInstr are replacing MI
237:   // with another instruction defining the same register. We expect that if MI
238:   // is a type instruction, and it is still referenced in VRegToTypeMap, then
239:   // those registers are dead or the VRegToTypeMap is out-of-date. We do not
240:   // expect passes to ask for the SPIR-V type of a dead register. If the
241:   // VRegToTypeMap is out-of-date already, then there was an error before. We
242:   // cannot add an assert to verify this because the VRegToTypeMap can be
243:   // out-of-date.
244:   // - FunctionToInstr & FunctionToInstrRev: At this point, we should not be
245:   // deleting functions. No need to update.
246:   // - AliasInstMDMap: Would require a linear search, and the Intel Alias
247:   // instruction are not instructions instruction selection will be able to
248:   // remove.
249:
250:   const SPIRVSubtarget &ST = MI->getMF()->getSubtarget<SPIRVSubtarget>();
251:   [[maybe_unused]] const SPIRVInstrInfo *TII = ST.getInstrInfo();
252:   assert(!TII->isAliasingInstr(*MI) &&
253:          "Cannot invalidate aliasing instructions.");
254:   assert(MI->getOpcode() != SPIRV::OpFunction &&
255:          "Cannot invalidate OpFunction.");
256:
257:   if (MI->getOpcode() == SPIRV::OpFunctionCall) {
258:     if (const auto *F = dyn_cast<Function>(MI->getOperand(2).getGlobal())) {
259:       auto It = ForwardCalls.find(F);
260:       if (It != ForwardCalls.end()) {
261:         It->second.erase(MI);
262:         if (It->second.empty())
263:           ForwardCalls.erase(It);
264:       }
265:     }
266:   }
267:
268:   const MachineFunction *MF = MI->getMF();
269:   auto It = LastInsertedTypeMap.find(MF);
270:   if (It != LastInsertedTypeMap.end() && It->second == MI)
```
- EN: This range implements operational logic in helpers such as buildInstr, addImm, addDef, SPIRVGlobalRegistry::getOpTypeVoid, translating backend policy into executable code.
- CN: 这一段实现了 buildInstr、addImm、addDef、SPIRVGlobalRegistry::getOpTypeVoid 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:     LastInsertedTypeMap.erase(MF);
272:   // remove from the duplicate tracker to avoid incorrect reuse
273:   erase(MI);
274: }
275:
276: const MachineInstr *SPIRVGlobalRegistry::createConstOrTypeAtFunctionEntry(
277:     MachineIRBuilder &MIRBuilder,
278:     std::function<MachineInstr *(MachineIRBuilder &)> Op) {
279:   auto oldInsertPoint = MIRBuilder.getInsertPt();
280:   MachineBasicBlock *OldMBB = &MIRBuilder.getMBB();
281:   MachineBasicBlock *NewMBB = &*MIRBuilder.getMF().begin();
282:
283:   auto LastInsertedType = LastInsertedTypeMap.find(CurMF);
284:   if (LastInsertedType != LastInsertedTypeMap.end()) {
285:     auto It = LastInsertedType->second->getIterator();
286:     // It might happen that this instruction was removed from the first MBB,
287:     // hence the Parent's check.
288:     MachineBasicBlock::iterator InsertAt;
289:     if (It->getParent() != NewMBB)
290:       InsertAt = oldInsertPoint->getParent() == NewMBB
291:                      ? oldInsertPoint
292:                      : getInsertPtValidEnd(NewMBB);
293:     else if (It->getNextNode())
294:       InsertAt = It->getNextNode()->getIterator();
295:     else
296:       InsertAt = getInsertPtValidEnd(NewMBB);
297:     MIRBuilder.setInsertPt(*NewMBB, InsertAt);
298:   } else {
299:     MIRBuilder.setInsertPt(*NewMBB, NewMBB->begin());
300:     auto Result = LastInsertedTypeMap.try_emplace(CurMF, nullptr);
301:     assert(Result.second);
302:     LastInsertedType = Result.first;
303:   }
304:
305:   MachineInstr *ConstOrType = Op(MIRBuilder);
306:   // We expect all users of this function to insert definitions at the insertion
307:   // point set above that is always the first MBB.
308:   assert(ConstOrType->getParent() == NewMBB);
309:   LastInsertedType->second = ConstOrType;
310:   // Advance past any continued instructions so that the next type/constant
311:   // is inserted after the full group, preserving required adjacency.
312:   while (auto *Next = LastInsertedType->second->getNextNode()) {
313:     unsigned Opc = Next->getOpcode();
314:     if (Opc == SPIRV::OpTypeStructContinuedINTEL ||
315:         Opc == SPIRV::OpConstantCompositeContinuedINTEL ||
316:         Opc == SPIRV::OpSpecConstantCompositeContinuedINTEL ||
317:         Opc == SPIRV::OpCompositeConstructContinuedINTEL)
318:       LastInsertedType->second = Next;
319:     else
320:       break;
321:   }
322:
323:   MIRBuilder.setInsertPt(*OldMBB, oldInsertPoint);
324:   return ConstOrType;
325: }
326:
327: SPIRVTypeInst
328: SPIRVGlobalRegistry::getOpTypeVector(uint32_t NumElems, SPIRVTypeInst ElemType,
329:                                      MachineIRBuilder &MIRBuilder) {
330:   auto EleOpc = ElemType->getOpcode();
331:   assert(NumElems >= 2 && "SPIR-V OpTypeVector requires at least 2 components");
332:
333:   if (EleOpc == SPIRV::OpTypePointer) {
334:     if (!cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget())
335:              .canUseExtension(
336:                  SPIRV::Extension::SPV_INTEL_masked_gather_scatter)) {
337:       const Function &F = MIRBuilder.getMF().getFunction();
338:       F.getContext().diagnose(DiagnosticInfoUnsupported(
339:           F,
340:           "Vector of pointers requires SPV_INTEL_masked_gather_scatter "
341:           "extension",
342:           DebugLoc(), DS_Error));
343:     }
344:   } else {
345:     assert((EleOpc == SPIRV::OpTypeInt || EleOpc == SPIRV::OpTypeFloat ||
346:             EleOpc == SPIRV::OpTypeBool) &&
347:            "Invalid vector element type");
348:   }
349:
350:   return createConstOrTypeAtFunctionEntry(
351:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
352:         return MIRBuilder.buildInstr(SPIRV::OpTypeVector)
353:             .addDef(createTypeVReg(MIRBuilder))
354:             .addUse(getSPIRVTypeID(ElemType))
355:             .addImm(NumElems);
356:       });
357: }
358:
359: Register SPIRVGlobalRegistry::getOrCreateConstFP(APFloat Val, MachineInstr &I,
360:                                                  SPIRVTypeInst SpvType,
```
- EN: This range implements operational logic in helpers such as erase, getInsertPt, getMBB, getMF, translating backend policy into executable code.
- CN: 这一段实现了 erase、getInsertPt、getMBB、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:                                                  const SPIRVInstrInfo &TII,
362:                                                  bool ZeroAsNull) {
363:   LLVMContext &Ctx = CurMF->getFunction().getContext();
364:   auto *const CF = ConstantFP::get(Ctx, Val);
365:   const MachineInstr *MI = findMI(CF, CurMF);
366:   if (MI && (MI->getOpcode() == SPIRV::OpConstantNull ||
367:              MI->getOpcode() == SPIRV::OpConstantF))
368:     return MI->getOperand(0).getReg();
369:   return createConstFP(CF, I, SpvType, TII, ZeroAsNull);
370: }
371:
372: Register SPIRVGlobalRegistry::createConstFP(const ConstantFP *CF,
373:                                             MachineInstr &I,
374:                                             SPIRVTypeInst SpvType,
375:                                             const SPIRVInstrInfo &TII,
376:                                             bool ZeroAsNull) {
377:   unsigned BitWidth = getScalarOrVectorBitWidth(SpvType);
378:   LLT LLTy = LLT::scalar(BitWidth);
379:   Register Res = CurMF->getRegInfo().createGenericVirtualRegister(LLTy);
380:   CurMF->getRegInfo().setRegClass(Res, &SPIRV::fIDRegClass);
381:   assignSPIRVTypeToVReg(SpvType, Res, *CurMF);
382:
383:   MachineInstr *DepMI =
384:       const_cast<MachineInstr *>(static_cast<const MachineInstr *>(SpvType));
385:   MachineIRBuilder MIRBuilder(*DepMI->getParent(), DepMI->getIterator());
386:   const MachineInstr *Const = createConstOrTypeAtFunctionEntry(
387:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
388:         MachineInstrBuilder MIB;
389:         // In OpenCL OpConstantNull - Scalar floating point: +0.0 (all bits 0)
390:         if (CF->getValue().isPosZero() && ZeroAsNull) {
391:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantNull)
392:                     .addDef(Res)
393:                     .addUse(getSPIRVTypeID(SpvType));
394:         } else {
395:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantF)
396:                     .addDef(Res)
397:                     .addUse(getSPIRVTypeID(SpvType));
398:           addNumImm(APInt(BitWidth,
399:                           CF->getValueAPF().bitcastToAPInt().getZExtValue()),
400:                     MIB);
401:         }
402:         const auto &ST = CurMF->getSubtarget();
403:         constrainSelectedInstRegOperands(*MIB, *ST.getInstrInfo(),
404:                                          *ST.getRegisterInfo(),
405:                                          *ST.getRegBankInfo());
406:         return MIB;
407:       });
408:   add(CF, Const);
409:   return Res;
410: }
411:
412: Register SPIRVGlobalRegistry::getOrCreateConstInt(uint64_t Val, MachineInstr &I,
413:                                                   SPIRVTypeInst SpvType,
414:                                                   const SPIRVInstrInfo &TII,
415:                                                   bool ZeroAsNull) {
416:   return getOrCreateConstInt(APInt(getScalarOrVectorBitWidth(SpvType), Val), I,
417:                              SpvType, TII, ZeroAsNull);
418: }
419:
420: Register SPIRVGlobalRegistry::getOrCreateConstInt(const APInt &Val,
421:                                                   MachineInstr &I,
422:                                                   SPIRVTypeInst SpvType,
423:                                                   const SPIRVInstrInfo &TII,
424:                                                   bool ZeroAsNull) {
425:   auto *const CI = ConstantInt::get(
426:       cast<IntegerType>(getTypeForSPIRVType(SpvType))->getContext(), Val);
427:   const MachineInstr *MI = findMI(CI, CurMF);
428:   if (MI && (MI->getOpcode() == SPIRV::OpConstantNull ||
429:              MI->getOpcode() == SPIRV::OpConstantI))
430:     return MI->getOperand(0).getReg();
431:   return createConstInt(CI, I, SpvType, TII, ZeroAsNull);
432: }
433:
434: Register SPIRVGlobalRegistry::createConstInt(const ConstantInt *CI,
435:                                              MachineInstr &I,
436:                                              SPIRVTypeInst SpvType,
437:                                              const SPIRVInstrInfo &TII,
438:                                              bool ZeroAsNull) {
439:   unsigned BitWidth = getScalarOrVectorBitWidth(SpvType);
440:   LLT LLTy = LLT::scalar(BitWidth);
441:   Register Res = CurMF->getRegInfo().createGenericVirtualRegister(LLTy);
442:   CurMF->getRegInfo().setRegClass(Res, &SPIRV::iIDRegClass);
443:   assignIntTypeToVReg(BitWidth, Res, I, TII);
444:
445:   MachineInstr *DepMI =
446:       const_cast<MachineInstr *>(static_cast<const MachineInstr *>(SpvType));
447:   MachineIRBuilder MIRBuilder(*DepMI->getParent(), DepMI->getIterator());
448:   const MachineInstr *Const = createConstOrTypeAtFunctionEntry(
449:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
450:         MachineInstrBuilder MIB;
```
- EN: This range implements operational logic in helpers such as getFunction, ConstantFP::get, findMI, getOpcode, translating backend policy into executable code.
- CN: 这一段实现了 getFunction、ConstantFP::get、findMI、getOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:         if (BitWidth == 1) {
452:           MIB = MIRBuilder
453:                     .buildInstr(CI->isZero() ? SPIRV::OpConstantFalse
454:                                              : SPIRV::OpConstantTrue)
455:                     .addDef(Res)
456:                     .addUse(getSPIRVTypeID(SpvType));
457:         } else if (!CI->isZero() || !ZeroAsNull) {
458:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantI)
459:                     .addDef(Res)
460:                     .addUse(getSPIRVTypeID(SpvType));
461:           addNumImm(CI->getValue(), MIB);
462:         } else {
463:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantNull)
464:                     .addDef(Res)
465:                     .addUse(getSPIRVTypeID(SpvType));
466:         }
467:         const auto &ST = CurMF->getSubtarget();
468:         constrainSelectedInstRegOperands(*MIB, *ST.getInstrInfo(),
469:                                          *ST.getRegisterInfo(),
470:                                          *ST.getRegBankInfo());
471:         return MIB;
472:       });
473:   add(CI, Const);
474:   return Res;
475: }
476:
477: Register SPIRVGlobalRegistry::buildConstantInt(uint64_t Val,
478:                                                MachineIRBuilder &MIRBuilder,
479:                                                SPIRVTypeInst SpvType,
480:                                                bool EmitIR, bool ZeroAsNull) {
481:   assert(SpvType);
482:   auto &MF = MIRBuilder.getMF();
483:   const IntegerType *Ty = cast<IntegerType>(getTypeForSPIRVType(SpvType));
484:   // TODO: Avoid implicit trunc?
485:   // See https://github.com/llvm/llvm-project/issues/112510.
486:   auto *const CI = ConstantInt::get(const_cast<IntegerType *>(Ty), Val,
487:                                     /*IsSigned=*/false, /*ImplicitTrunc=*/true);
488:   Register Res = find(CI, &MF);
489:   if (Res.isValid())
490:     return Res;
491:
492:   unsigned BitWidth = getScalarOrVectorBitWidth(SpvType);
493:   LLT LLTy = LLT::scalar(BitWidth);
494:   MachineRegisterInfo &MRI = MF.getRegInfo();
495:   Res = MRI.createGenericVirtualRegister(LLTy);
496:   MRI.setRegClass(Res, &SPIRV::iIDRegClass);
497:   assignTypeToVReg(Ty, Res, MIRBuilder, SPIRV::AccessQualifier::ReadWrite,
498:                    EmitIR);
499:
500:   const MachineInstr *Const = createConstOrTypeAtFunctionEntry(
501:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
502:         if (EmitIR)
503:           return MIRBuilder.buildConstant(Res, *CI);
504:         Register SpvTypeReg = getSPIRVTypeID(SpvType);
505:         MachineInstrBuilder MIB;
506:         if (Val || !ZeroAsNull) {
507:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantI)
508:                     .addDef(Res)
509:                     .addUse(SpvTypeReg);
510:           addNumImm(APInt(BitWidth, Val), MIB);
511:         } else {
512:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantNull)
513:                     .addDef(Res)
514:                     .addUse(SpvTypeReg);
515:         }
516:         const auto &Subtarget = CurMF->getSubtarget();
517:         constrainSelectedInstRegOperands(*MIB, *Subtarget.getInstrInfo(),
518:                                          *Subtarget.getRegisterInfo(),
519:                                          *Subtarget.getRegBankInfo());
520:         return MIB;
521:       });
522:   add(CI, Const);
523:   return Res;
524: }
525:
526: Register SPIRVGlobalRegistry::buildConstantFP(APFloat Val,
527:                                               MachineIRBuilder &MIRBuilder,
528:                                               SPIRVTypeInst SpvType) {
529:   auto &MF = MIRBuilder.getMF();
530:   LLVMContext &Ctx = MF.getFunction().getContext();
531:   if (!SpvType)
532:     SpvType = getOrCreateSPIRVType(Type::getFloatTy(Ctx), MIRBuilder,
533:                                    SPIRV::AccessQualifier::ReadWrite, true);
534:   auto *const CF = ConstantFP::get(Ctx, Val);
535:   Register Res = find(CF, &MF);
536:   if (Res.isValid())
537:     return Res;
538:
539:   LLT LLTy = LLT::scalar(getScalarOrVectorBitWidth(SpvType));
540:   Res = MF.getRegInfo().createGenericVirtualRegister(LLTy);
```
- EN: This range implements operational logic in helpers such as addDef, addUse, buildInstr, addNumImm, translating backend policy into executable code.
- CN: 这一段实现了 addDef、addUse、buildInstr、addNumImm 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:   MF.getRegInfo().setRegClass(Res, &SPIRV::fIDRegClass);
542:   assignSPIRVTypeToVReg(SpvType, Res, MF);
543:
544:   const MachineInstr *Const = createConstOrTypeAtFunctionEntry(
545:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
546:         MachineInstrBuilder MIB;
547:         MIB = MIRBuilder.buildInstr(SPIRV::OpConstantF)
548:                   .addDef(Res)
549:                   .addUse(getSPIRVTypeID(SpvType));
550:         addNumImm(CF->getValueAPF().bitcastToAPInt(), MIB);
551:         return MIB;
552:       });
553:   add(CF, Const);
554:   return Res;
555: }
556:
557: Register SPIRVGlobalRegistry::getOrCreateBaseRegister(
558:     Constant *Val, MachineInstr &I, SPIRVTypeInst SpvType,
559:     const SPIRVInstrInfo &TII, unsigned BitWidth, bool ZeroAsNull) {
560:   SPIRVTypeInst Type = SpvType;
561:   if (SpvType->getOpcode() == SPIRV::OpTypeVector ||
562:       SpvType->getOpcode() == SPIRV::OpTypeArray) {
563:     auto EleTypeReg = SpvType->getOperand(1).getReg();
564:     Type = getSPIRVTypeForVReg(EleTypeReg);
565:   }
566:   if (Type->getOpcode() == SPIRV::OpTypeFloat) {
567:     SPIRVTypeInst SpvBaseType = getOrCreateSPIRVFloatType(BitWidth, I, TII);
568:     return getOrCreateConstFP(cast<ConstantFP>(Val)->getValue(), I, SpvBaseType,
569:                               TII, ZeroAsNull);
570:   }
571:   assert(Type->getOpcode() == SPIRV::OpTypeInt);
572:   SPIRVTypeInst SpvBaseType = getOrCreateSPIRVIntegerType(BitWidth, I, TII);
573:   return getOrCreateConstInt(Val->getUniqueInteger(), I, SpvBaseType, TII,
574:                              ZeroAsNull);
575: }
576:
577: Register SPIRVGlobalRegistry::getOrCreateCompositeOrNull(
578:     Constant *Val, MachineInstr &I, SPIRVTypeInst SpvType,
579:     const SPIRVInstrInfo &TII, Constant *CA, unsigned BitWidth,
580:     unsigned ElemCnt, bool ZeroAsNull) {
581:   if (Register R = find(CA, CurMF); R.isValid())
582:     return R;
583:
584:   bool IsNull = Val->isNullValue() && ZeroAsNull;
585:   Register ElemReg;
586:   if (!IsNull)
587:     ElemReg =
588:         getOrCreateBaseRegister(Val, I, SpvType, TII, BitWidth, ZeroAsNull);
589:
590:   LLT LLTy = LLT::scalar(64);
591:   Register Res = CurMF->getRegInfo().createGenericVirtualRegister(LLTy);
592:   CurMF->getRegInfo().setRegClass(Res, getRegClass(SpvType));
593:   assignSPIRVTypeToVReg(SpvType, Res, *CurMF);
594:
595:   MachineInstr *DepMI =
596:       const_cast<MachineInstr *>(static_cast<const MachineInstr *>(SpvType));
597:   MachineIRBuilder MIRBuilder(*DepMI->getParent(), DepMI->getIterator());
598:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
599:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
600:         MachineInstrBuilder MIB;
601:         if (!IsNull) {
602:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantComposite)
603:                     .addDef(Res)
604:                     .addUse(getSPIRVTypeID(SpvType));
605:           for (unsigned i = 0; i < ElemCnt; ++i)
606:             MIB.addUse(ElemReg);
607:         } else {
608:           MIB = MIRBuilder.buildInstr(SPIRV::OpConstantNull)
609:                     .addDef(Res)
610:                     .addUse(getSPIRVTypeID(SpvType));
611:         }
612:         const auto &Subtarget = CurMF->getSubtarget();
613:         constrainSelectedInstRegOperands(*MIB, *Subtarget.getInstrInfo(),
614:                                          *Subtarget.getRegisterInfo(),
615:                                          *Subtarget.getRegBankInfo());
616:         return MIB;
617:       });
618:   add(CA, NewMI);
619:   return Res;
620: }
621:
622: Register SPIRVGlobalRegistry::getOrCreateConstVector(uint64_t Val,
623:                                                      MachineInstr &I,
624:                                                      SPIRVTypeInst SpvType,
625:                                                      const SPIRVInstrInfo &TII,
626:                                                      bool ZeroAsNull) {
627:   return getOrCreateConstVector(APInt(getScalarOrVectorBitWidth(SpvType), Val),
628:                                 I, SpvType, TII, ZeroAsNull);
629: }
630:
```
- EN: This range implements operational logic in helpers such as getRegInfo, assignSPIRVTypeToVReg, buildInstr, addDef, translating backend policy into executable code.
- CN: 这一段实现了 getRegInfo、assignSPIRVTypeToVReg、buildInstr、addDef 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631: Register SPIRVGlobalRegistry::getOrCreateConstVector(const APInt &Val,
632:                                                      MachineInstr &I,
633:                                                      SPIRVTypeInst SpvType,
634:                                                      const SPIRVInstrInfo &TII,
635:                                                      bool ZeroAsNull) {
636:   const Type *LLVMTy = getTypeForSPIRVType(SpvType);
637:   assert(LLVMTy->isVectorTy() &&
638:          "Expected vector type for constant vector creation");
639:   const FixedVectorType *LLVMVecTy = cast<FixedVectorType>(LLVMTy);
640:   Type *LLVMBaseTy = LLVMVecTy->getElementType();
641:   assert(LLVMBaseTy->isIntegerTy() &&
642:          "Expected integer element type for APInt constant vector");
643:   auto *ConstVal = cast<ConstantInt>(ConstantInt::get(LLVMBaseTy, Val));
644:   auto *ConstVec =
645:       ConstantVector::getSplat(LLVMVecTy->getElementCount(), ConstVal);
646:   unsigned BW = getScalarOrVectorBitWidth(SpvType);
647:   return getOrCreateCompositeOrNull(ConstVal, I, SpvType, TII, ConstVec, BW,
648:                                     getScalarOrVectorComponentCount(SpvType),
649:                                     ZeroAsNull);
650: }
651:
652: Register SPIRVGlobalRegistry::getOrCreateConstVector(APFloat Val,
653:                                                      MachineInstr &I,
654:                                                      SPIRVTypeInst SpvType,
655:                                                      const SPIRVInstrInfo &TII,
656:                                                      bool ZeroAsNull) {
657:   const Type *LLVMTy = getTypeForSPIRVType(SpvType);
658:   assert(LLVMTy->isVectorTy());
659:   const FixedVectorType *LLVMVecTy = cast<FixedVectorType>(LLVMTy);
660:   Type *LLVMBaseTy = LLVMVecTy->getElementType();
661:   assert(LLVMBaseTy->isFloatingPointTy());
662:   auto *ConstVal = ConstantFP::get(LLVMBaseTy, Val);
663:   auto *ConstVec =
664:       ConstantVector::getSplat(LLVMVecTy->getElementCount(), ConstVal);
665:   unsigned BW = getScalarOrVectorBitWidth(SpvType);
666:   return getOrCreateCompositeOrNull(ConstVal, I, SpvType, TII, ConstVec, BW,
667:                                     getScalarOrVectorComponentCount(SpvType),
668:                                     ZeroAsNull);
669: }
670:
671: Register SPIRVGlobalRegistry::getOrCreateConstIntArray(
672:     uint64_t Val, size_t Num, MachineInstr &I, SPIRVTypeInst SpvType,
673:     const SPIRVInstrInfo &TII) {
674:   const Type *LLVMTy = getTypeForSPIRVType(SpvType);
675:   assert(LLVMTy->isArrayTy());
676:   const ArrayType *LLVMArrTy = cast<ArrayType>(LLVMTy);
677:   Type *LLVMBaseTy = LLVMArrTy->getElementType();
678:   Constant *CI = ConstantInt::get(LLVMBaseTy, Val);
679:   SPIRVTypeInst SpvBaseTy =
680:       getSPIRVTypeForVReg(SpvType->getOperand(1).getReg());
681:   unsigned BW = getScalarOrVectorBitWidth(SpvBaseTy);
682:   // The following is reasonably unique key that is better that [Val]. The naive
683:   // alternative would be something along the lines of:
684:   //   SmallVector<Constant *> NumCI(Num, CI);
685:   //   Constant *UniqueKey =
686:   //     ConstantArray::get(const_cast<ArrayType*>(LLVMArrTy), NumCI);
687:   // that would be a truly unique but dangerous key, because it could lead to
688:   // the creation of constants of arbitrary length (that is, the parameter of
689:   // memset) which were missing in the original module.
690:   Type *I64Ty = Type::getInt64Ty(LLVMBaseTy->getContext());
691:   Constant *UniqueKey = ConstantStruct::getAnon(
692:       {PoisonValue::get(const_cast<ArrayType *>(LLVMArrTy)),
693:        ConstantInt::get(LLVMBaseTy, Val), ConstantInt::get(I64Ty, Num)});
694:   return getOrCreateCompositeOrNull(CI, I, SpvType, TII, UniqueKey, BW,
695:                                     LLVMArrTy->getNumElements());
696: }
697:
698: Register SPIRVGlobalRegistry::getOrCreateIntCompositeOrNull(
699:     uint64_t Val, MachineIRBuilder &MIRBuilder, SPIRVTypeInst SpvType,
700:     bool EmitIR, Constant *CA, unsigned BitWidth, unsigned ElemCnt) {
701:   if (Register R = find(CA, CurMF); R.isValid())
702:     return R;
703:
704:   Register ElemReg;
705:   if (Val || EmitIR) {
706:     SPIRVTypeInst SpvBaseType =
707:         getOrCreateSPIRVIntegerType(BitWidth, MIRBuilder);
708:     ElemReg = buildConstantInt(Val, MIRBuilder, SpvBaseType, EmitIR);
709:   }
710:   LLT LLTy = EmitIR ? LLT::fixed_vector(ElemCnt, BitWidth) : LLT::scalar(64);
711:   Register Res = CurMF->getRegInfo().createGenericVirtualRegister(LLTy);
712:   CurMF->getRegInfo().setRegClass(Res, &SPIRV::iIDRegClass);
713:   assignSPIRVTypeToVReg(SpvType, Res, *CurMF);
714:
715:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
716:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
717:         if (EmitIR)
718:           return MIRBuilder.buildSplatBuildVector(Res, ElemReg);
719:
720:         if (Val) {
```
- EN: This range implements operational logic in helpers such as getTypeForSPIRVType, getElementType, ConstantInt::get, ConstantVector::getSplat, translating backend policy into executable code.
- CN: 这一段实现了 getTypeForSPIRVType、getElementType、ConstantInt::get、ConstantVector::getSplat 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:           auto MIB = MIRBuilder.buildInstr(SPIRV::OpConstantComposite)
722:                          .addDef(Res)
723:                          .addUse(getSPIRVTypeID(SpvType));
724:           for (unsigned i = 0; i < ElemCnt; ++i)
725:             MIB.addUse(ElemReg);
726:           return MIB;
727:         }
728:
729:         return MIRBuilder.buildInstr(SPIRV::OpConstantNull)
730:             .addDef(Res)
731:             .addUse(getSPIRVTypeID(SpvType));
732:       });
733:   add(CA, NewMI);
734:   return Res;
735: }
736:
737: Register SPIRVGlobalRegistry::getOrCreateConsIntVector(
738:     uint64_t Val, MachineIRBuilder &MIRBuilder, SPIRVTypeInst SpvType,
739:     bool EmitIR) {
740:   const Type *LLVMTy = getTypeForSPIRVType(SpvType);
741:   assert(LLVMTy->isVectorTy());
742:   const FixedVectorType *LLVMVecTy = cast<FixedVectorType>(LLVMTy);
743:   Type *LLVMBaseTy = LLVMVecTy->getElementType();
744:   const auto ConstInt = ConstantInt::get(LLVMBaseTy, Val);
745:   auto ConstVec =
746:       ConstantVector::getSplat(LLVMVecTy->getElementCount(), ConstInt);
747:   unsigned BW = getScalarOrVectorBitWidth(SpvType);
748:   return getOrCreateIntCompositeOrNull(
749:       Val, MIRBuilder, SpvType, EmitIR, ConstVec, BW,
750:       getScalarOrVectorComponentCount(SpvType));
751: }
752:
753: Register
754: SPIRVGlobalRegistry::getOrCreateConstNullPtr(MachineIRBuilder &MIRBuilder,
755:                                              SPIRVTypeInst SpvType) {
756:   const Type *Ty = getTypeForSPIRVType(SpvType);
757:   unsigned AddressSpace = typeToAddressSpace(Ty);
758:   Type *ElemTy = ::getPointeeType(Ty);
759:   assert(ElemTy);
760:   const Constant *CP = ConstantTargetNone::get(
761:       dyn_cast<TargetExtType>(getTypedPointerWrapper(ElemTy, AddressSpace)));
762:   Register Res = find(CP, CurMF);
763:   if (Res.isValid())
764:     return Res;
765:
766:   LLT LLTy = LLT::pointer(AddressSpace, getPointerSize());
767:   Res = CurMF->getRegInfo().createGenericVirtualRegister(LLTy);
768:   CurMF->getRegInfo().setRegClass(Res, &SPIRV::pIDRegClass);
769:   assignSPIRVTypeToVReg(SpvType, Res, *CurMF);
770:
771:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
772:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
773:         return MIRBuilder.buildInstr(SPIRV::OpConstantNull)
774:             .addDef(Res)
775:             .addUse(getSPIRVTypeID(SpvType));
776:       });
777:   add(CP, NewMI);
778:   return Res;
779: }
780:
781: Register
782: SPIRVGlobalRegistry::buildConstantSampler(Register ResReg, unsigned AddrMode,
783:                                           unsigned Param, unsigned FilerMode,
784:                                           MachineIRBuilder &MIRBuilder) {
785:   auto Sampler =
786:       ResReg.isValid()
787:           ? ResReg
788:           : MIRBuilder.getMRI()->createVirtualRegister(&SPIRV::iIDRegClass);
789:   SPIRVTypeInst TypeSampler = getOrCreateOpTypeSampler(MIRBuilder);
790:   Register TypeSamplerReg = getSPIRVTypeID(TypeSampler);
791:   // We cannot use createOpType() logic here, because of the
792:   // GlobalISel/IRTranslator.cpp check for a tail call that expects that
793:   // MIRBuilder.getInsertPt() has a previous instruction. If this constant is
794:   // inserted as a result of "__translate_sampler_initializer()" this would
795:   // break this IRTranslator assumption.
796:   MIRBuilder.buildInstr(SPIRV::OpConstantSampler)
797:       .addDef(Sampler)
798:       .addUse(TypeSamplerReg)
799:       .addImm(AddrMode)
800:       .addImm(Param)
801:       .addImm(FilerMode);
802:   return Sampler;
803: }
804:
805: Register SPIRVGlobalRegistry::buildGlobalVariable(
806:     Register ResVReg, SPIRVTypeInst BaseType, StringRef Name,
807:     const GlobalValue *GV, SPIRV::StorageClass::StorageClass Storage,
808:     const MachineInstr *Init, bool IsConst,
809:     const std::optional<SPIRV::LinkageType::LinkageType> &LinkageType,
810:     MachineIRBuilder &MIRBuilder, bool IsInstSelector) {
```
- EN: This range implements operational logic in helpers such as buildInstr, addDef, addUse, add, translating backend policy into executable code.
- CN: 这一段实现了 buildInstr、addDef、addUse、add 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:   const GlobalVariable *GVar = nullptr;
812:   if (GV) {
813:     GVar = cast<const GlobalVariable>(GV);
814:   } else {
815:     // If GV is not passed explicitly, use the name to find or construct
816:     // the global variable.
817:     Module *M = MIRBuilder.getMF().getFunction().getParent();
818:     GVar = M->getGlobalVariable(Name);
819:     if (GVar == nullptr) {
820:       const Type *Ty = getTypeForSPIRVType(BaseType); // TODO: check type.
821:       // Module takes ownership of the global var.
822:       GVar = new GlobalVariable(*M, const_cast<Type *>(Ty), false,
823:                                 GlobalValue::ExternalLinkage, nullptr,
824:                                 Twine(Name));
825:     }
826:     GV = GVar;
827:   }
828:
829:   const MachineFunction *MF = &MIRBuilder.getMF();
830:   Register Reg = find(GVar, MF);
831:   if (Reg.isValid()) {
832:     if (Reg != ResVReg)
833:       MIRBuilder.buildCopy(ResVReg, Reg);
834:     return ResVReg;
835:   }
836:
837:   // Emit the OpVariable into the entry block to ensure the def dominates
838:   // all uses across all MBBs.
839:   MachineBasicBlock &EntryBB = MIRBuilder.getMF().front();
840:   MachineIRBuilder GVBuilder(MIRBuilder.getState());
841:   if (&GVBuilder.getMBB() != &EntryBB)
842:     GVBuilder.setInsertPt(EntryBB, EntryBB.getFirstTerminator());
843:
844:   auto MIB = GVBuilder.buildInstr(SPIRV::OpVariable)
845:                  .addDef(ResVReg)
846:                  .addUse(getSPIRVTypeID(BaseType))
847:                  .addImm(static_cast<uint32_t>(Storage));
848:   if (Init)
849:     MIB.addUse(Init->getOperand(0).getReg());
850:   // ISel may introduce a new register on this step, so we need to add it to
851:   // DT and correct its type avoiding fails on the next stage.
852:   if (IsInstSelector) {
853:     const auto &Subtarget = CurMF->getSubtarget();
854:     constrainSelectedInstRegOperands(*MIB, *Subtarget.getInstrInfo(),
855:                                      *Subtarget.getRegisterInfo(),
856:                                      *Subtarget.getRegBankInfo());
857:   }
858:   add(GVar, MIB);
859:
860:   Reg = MIB->getOperand(0).getReg();
861:   addGlobalObject(GVar, MF, Reg);
862:
863:   // Set to Reg the same type as ResVReg has.
864:   auto MRI = MIRBuilder.getMRI();
865:   if (Reg != ResVReg) {
866:     LLT RegLLTy =
867:         LLT::pointer(MRI->getType(ResVReg).getAddressSpace(), getPointerSize());
868:     MRI->setType(Reg, RegLLTy);
869:     assignSPIRVTypeToVReg(BaseType, Reg, MIRBuilder.getMF());
870:   } else {
871:     // Our knowledge about the type may be updated.
872:     // If that's the case, we need to update a type
873:     // associated with the register.
874:     SPIRVTypeInst DefType = getSPIRVTypeForVReg(ResVReg);
875:     if (!DefType || DefType != SPIRVTypeInst(BaseType))
876:       assignSPIRVTypeToVReg(BaseType, Reg, MIRBuilder.getMF());
877:   }
878:
879:   // If it's a global variable with name, output OpName for it.
880:   if (GVar && GVar->hasName())
881:     buildOpName(Reg, GVar->getName(), MIRBuilder);
882:
883:   // Output decorations for the GV.
884:   // TODO: maybe move to GenerateDecorations pass.
885:   const SPIRVSubtarget &ST =
886:       cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget());
887:   if (IsConst && !ST.isShader())
888:     buildOpDecorate(Reg, MIRBuilder, SPIRV::Decoration::Constant, {});
889:
890:   if (GVar && GVar->getAlign().valueOrOne().value() != 1 && !ST.isShader()) {
891:     unsigned Alignment = (unsigned)GVar->getAlign().valueOrOne().value();
892:     buildOpDecorate(Reg, MIRBuilder, SPIRV::Decoration::Alignment, {Alignment});
893:   }
894:
895:   if (LinkageType)
896:     buildOpDecorate(Reg, MIRBuilder, SPIRV::Decoration::LinkageAttributes,
897:                     {static_cast<uint32_t>(*LinkageType)}, Name);
898:
899:   SPIRV::BuiltIn::BuiltIn BuiltInId;
900:   if (getSpirvBuiltInIdByName(Name, BuiltInId))
```
- EN: This range implements operational logic in helpers such as getMF, getGlobalVariable, getTypeForSPIRVType, Twine, translating backend policy into executable code.
- CN: 这一段实现了 getMF、getGlobalVariable、getTypeForSPIRVType、Twine 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:     buildOpDecorate(Reg, MIRBuilder, SPIRV::Decoration::BuiltIn,
902:                     {static_cast<uint32_t>(BuiltInId)});
903:
904:   // If it's a global variable with "spirv.Decorations" metadata node
905:   // recognize it as a SPIR-V friendly LLVM IR and parse "spirv.Decorations"
906:   // arguments.
907:   MDNode *GVarMD = nullptr;
908:   if (GVar && (GVarMD = GVar->getMetadata("spirv.Decorations")) != nullptr)
909:     buildOpSpirvDecorations(Reg, MIRBuilder, GVarMD, ST);
910:
911:   return Reg;
912: }
913:
914: // Returns a name based on the Type. Notes that this does not look at
915: // decorations, and will return the same string for two types that are the same
916: // except for decorations.
917: Register SPIRVGlobalRegistry::getOrCreateGlobalVariableWithBinding(
918:     SPIRVTypeInst VarType, uint32_t Set, uint32_t Binding, StringRef Name,
919:     MachineIRBuilder &MIRBuilder) {
920:   Register VarReg =
921:       MIRBuilder.getMRI()->createVirtualRegister(&SPIRV::iIDRegClass);
922:
923:   buildGlobalVariable(VarReg, VarType, Name, nullptr,
924:                       getPointerStorageClass(VarType), nullptr, false,
925:                       std::nullopt, MIRBuilder, false);
926:
927:   buildOpDecorate(VarReg, MIRBuilder, SPIRV::Decoration::DescriptorSet, {Set});
928:   buildOpDecorate(VarReg, MIRBuilder, SPIRV::Decoration::Binding, {Binding});
929:   return VarReg;
930: }
931:
932: // TODO: Double check the calls to getOpTypeArray to make sure that `ElemType`
933: // is explicitly laid out when required.
934: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeArray(uint32_t NumElems,
935:                                                   SPIRVTypeInst ElemType,
936:                                                   MachineIRBuilder &MIRBuilder,
937:                                                   bool ExplicitLayoutRequired,
938:                                                   bool EmitIR) {
939:   assert((ElemType->getOpcode() != SPIRV::OpTypeVoid) &&
940:          "Invalid array element type");
941:   SPIRVTypeInst SpvTypeInt32 = getOrCreateSPIRVIntegerType(32, MIRBuilder);
942:   SPIRVTypeInst ArrayType = nullptr;
943:   const SPIRVSubtarget &ST =
944:       cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget());
945:   if (NumElems != 0) {
946:     Register NumElementsVReg =
947:         buildConstantInt(NumElems, MIRBuilder, SpvTypeInt32, EmitIR);
948:     ArrayType = createConstOrTypeAtFunctionEntry(
949:         MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
950:           return MIRBuilder.buildInstr(SPIRV::OpTypeArray)
951:               .addDef(createTypeVReg(MIRBuilder))
952:               .addUse(getSPIRVTypeID(ElemType))
953:               .addUse(NumElementsVReg);
954:         });
955:   } else if (ST.getTargetTriple().getVendor() == Triple::VendorType::AMD) {
956:     // We set the array size to the token UINT64_MAX value, which is generally
957:     // illegal (the maximum legal size is 61-bits) for the foreseeable future.
958:     SPIRVTypeInst SpvTypeInt64 = getOrCreateSPIRVIntegerType(64, MIRBuilder);
959:     Register NumElementsVReg =
960:         buildConstantInt(UINT64_MAX, MIRBuilder, SpvTypeInt64, EmitIR);
961:     ArrayType = createConstOrTypeAtFunctionEntry(
962:         MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
963:           return MIRBuilder.buildInstr(SPIRV::OpTypeArray)
964:               .addDef(createTypeVReg(MIRBuilder))
965:               .addUse(getSPIRVTypeID(ElemType))
966:               .addUse(NumElementsVReg);
967:         });
968:   } else {
969:     if (!ST.isShader()) {
970:       llvm::reportFatalUsageError(
971:           "Runtime arrays are not allowed in non-shader "
972:           "SPIR-V modules");
973:       return nullptr;
974:     }
975:     ArrayType = createConstOrTypeAtFunctionEntry(
976:         MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
977:           return MIRBuilder.buildInstr(SPIRV::OpTypeRuntimeArray)
978:               .addDef(createTypeVReg(MIRBuilder))
979:               .addUse(getSPIRVTypeID(ElemType));
980:         });
981:   }
982:
983:   if (ExplicitLayoutRequired && !isResourceType(ElemType)) {
984:     Type *ET = const_cast<Type *>(getTypeForSPIRVType(ElemType));
985:     addArrayStrideDecorations(ArrayType->defs().begin()->getReg(), ET,
986:                               MIRBuilder);
987:   }
988:
989:   return ArrayType;
990: }
```
- EN: This range implements operational logic in helpers such as buildOpSpirvDecorations, getMRI, buildOpDecorate, getOrCreateSPIRVIntegerType, translating backend policy into executable code.
- CN: 这一段实现了 buildOpSpirvDecorations、getMRI、buildOpDecorate、getOrCreateSPIRVIntegerType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:
 992: SPIRVTypeInst
 993: SPIRVGlobalRegistry::getOpTypeOpaque(const StructType *Ty,
 994:                                      MachineIRBuilder &MIRBuilder) {
 995:   assert(Ty->hasName());
 996:   const StringRef Name = Ty->hasName() ? Ty->getName() : "";
 997:   Register ResVReg = createTypeVReg(MIRBuilder);
 998:   return createConstOrTypeAtFunctionEntry(
 999:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1000:         auto MIB = MIRBuilder.buildInstr(SPIRV::OpTypeOpaque).addDef(ResVReg);
1001:         addStringImm(Name, MIB);
1002:         buildOpName(ResVReg, Name, MIRBuilder);
1003:         return MIB;
1004:       });
1005: }
1006:
1007: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeStruct(
1008:     const StructType *Ty, MachineIRBuilder &MIRBuilder,
1009:     SPIRV::AccessQualifier::AccessQualifier AccQual,
1010:     StructOffsetDecorator Decorator, bool EmitIR) {
1011:   Type *OriginalElementType = nullptr;
1012:   uint64_t TotalSize = 0;
1013:   if (matchPeeledArrayPattern(Ty, OriginalElementType, TotalSize)) {
1014:     SPIRVTypeInst ElementSPIRVType = findSPIRVType(
1015:         OriginalElementType, MIRBuilder, AccQual,
1016:         /* ExplicitLayoutRequired= */ Decorator != nullptr, EmitIR);
1017:     return getOpTypeArray(TotalSize, ElementSPIRVType, MIRBuilder,
1018:                           /*ExplicitLayoutRequired=*/Decorator != nullptr,
1019:                           EmitIR);
1020:   }
1021:
1022:   const SPIRVSubtarget &ST =
1023:       cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget());
1024:   SmallVector<Register, 4> FieldTypes;
1025:   constexpr unsigned MaxWordCount = UINT16_MAX;
1026:   const size_t NumElements = Ty->getNumElements();
1027:
1028:   size_t MaxNumElements = MaxWordCount - 2;
1029:   size_t SPIRVStructNumElements = NumElements;
1030:   if (NumElements > MaxNumElements) {
1031:     // Do adjustments for continued instructions.
1032:     SPIRVStructNumElements = MaxNumElements;
1033:     MaxNumElements = MaxWordCount - 1;
1034:   }
1035:
1036:   for (const auto &Elem : Ty->elements()) {
1037:     SPIRVTypeInst ElemTy = findSPIRVType(
1038:         toTypedPointer(Elem), MIRBuilder, AccQual,
1039:         /* ExplicitLayoutRequired= */ Decorator != nullptr, EmitIR);
1040:     assert(ElemTy && ElemTy->getOpcode() != SPIRV::OpTypeVoid &&
1041:            "Invalid struct element type");
1042:     FieldTypes.push_back(getSPIRVTypeID(ElemTy));
1043:   }
1044:   Register ResVReg = createTypeVReg(MIRBuilder);
1045:   if (Ty->hasName())
1046:     buildOpName(ResVReg, Ty->getName(), MIRBuilder);
1047:   if (Ty->isPacked() && !ST.isShader())
1048:     buildOpDecorate(ResVReg, MIRBuilder, SPIRV::Decoration::CPacked, {});
1049:
1050:   SPIRVTypeInst SPVType = createConstOrTypeAtFunctionEntry(
1051:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1052:         auto MIBStruct =
1053:             MIRBuilder.buildInstr(SPIRV::OpTypeStruct).addDef(ResVReg);
1054:         for (size_t I = 0; I < SPIRVStructNumElements; ++I)
1055:           MIBStruct.addUse(FieldTypes[I]);
1056:         for (size_t I = SPIRVStructNumElements; I < NumElements;
1057:              I += MaxNumElements) {
1058:           auto MIBCont =
1059:               MIRBuilder.buildInstr(SPIRV::OpTypeStructContinuedINTEL);
1060:           for (size_t J = I; J < std::min(I + MaxNumElements, NumElements); ++J)
1061:             MIBCont.addUse(FieldTypes[J]);
1062:         }
1063:         return MIBStruct;
1064:       });
1065:
1066:   if (Decorator)
1067:     Decorator(SPVType->defs().begin()->getReg());
1068:
1069:   return SPVType;
1070: }
1071:
1072: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSpecialType(
1073:     const Type *Ty, MachineIRBuilder &MIRBuilder,
1074:     SPIRV::AccessQualifier::AccessQualifier AccQual) {
1075:   assert(isSpecialOpaqueType(Ty) && "Not a special opaque builtin type");
1076:   return SPIRV::lowerBuiltinType(Ty, AccQual, MIRBuilder, this);
1077: }
1078:
1079: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypePointer(
1080:     SPIRV::StorageClass::StorageClass SC, SPIRVTypeInst ElemType,
```
- EN: This range defines or declares important types such as assert, createTypeVReg, buildInstr, addStringImm, shaping the data model used by SPIRVGlobalRegistry.cpp.
- CN: 这一段定义或声明了 assert、createTypeVReg、buildInstr、addStringImm 等关键类型，构成 SPIRVGlobalRegistry.cpp 使用的数据模型。

### Lines 1081-1170
```cpp
1081:     MachineIRBuilder &MIRBuilder, Register Reg) {
1082:   if (!Reg.isValid())
1083:     Reg = createTypeVReg(MIRBuilder);
1084:
1085:   return createConstOrTypeAtFunctionEntry(MIRBuilder, [&](MachineIRBuilder
1086:                                                               &MIRBuilder) {
1087:     return MIRBuilder.buildInstr(SPIRV::OpTypePointer)
1088:         .addDef(Reg)
1089:         .addImm(static_cast<uint32_t>(SC))
1090:         .addUse(getSPIRVTypeID(ElemType));
1091:   });
1092: }
1093:
1094: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeForwardPointer(
1095:     SPIRV::StorageClass::StorageClass SC, MachineIRBuilder &MIRBuilder) {
1096:   return createConstOrTypeAtFunctionEntry(MIRBuilder, [&](MachineIRBuilder
1097:                                                               &MIRBuilder) {
1098:     return MIRBuilder.buildInstr(SPIRV::OpTypeForwardPointer)
1099:         .addUse(createTypeVReg(MIRBuilder))
1100:         .addImm(static_cast<uint32_t>(SC));
1101:   });
1102: }
1103:
1104: SPIRVTypeInst SPIRVGlobalRegistry::getOpTypeFunction(
1105:     const FunctionType *Ty, SPIRVTypeInst RetType,
1106:     const SmallVectorImpl<SPIRVTypeInst> &ArgTypes,
1107:     MachineIRBuilder &MIRBuilder) {
1108:   const SPIRVSubtarget *ST =
1109:       static_cast<const SPIRVSubtarget *>(&MIRBuilder.getMF().getSubtarget());
1110:   if (Ty->isVarArg() && ST->isShader()) {
1111:     Function &Fn = MIRBuilder.getMF().getFunction();
1112:     Ty->getContext().diagnose(DiagnosticInfoUnsupported(
1113:         Fn, "SPIR-V shaders do not support variadic functions",
1114:         MIRBuilder.getDebugLoc()));
1115:   }
1116:   return createConstOrTypeAtFunctionEntry(MIRBuilder, [&](MachineIRBuilder
1117:                                                               &MIRBuilder) {
1118:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpTypeFunction)
1119:                    .addDef(createTypeVReg(MIRBuilder))
1120:                    .addUse(getSPIRVTypeID(RetType));
1121:     for (auto &ArgType : ArgTypes)
1122:       MIB.addUse(getSPIRVTypeID(ArgType));
1123:     return MIB;
1124:   });
1125: }
1126:
1127: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypeFunctionWithArgs(
1128:     const Type *Ty, SPIRVTypeInst RetType,
1129:     const SmallVectorImpl<SPIRVTypeInst> &ArgTypes,
1130:     MachineIRBuilder &MIRBuilder) {
1131:   if (const MachineInstr *MI = findMI(Ty, false, &MIRBuilder.getMF()))
1132:     return MI;
1133:   const MachineInstr *NewMI =
1134:       getOpTypeFunction(cast<FunctionType>(Ty), RetType, ArgTypes, MIRBuilder);
1135:   add(Ty, false, NewMI);
1136:   return finishCreatingSPIRVType(Ty, NewMI);
1137: }
1138:
1139: SPIRVTypeInst SPIRVGlobalRegistry::findSPIRVType(
1140:     const Type *Ty, MachineIRBuilder &MIRBuilder,
1141:     SPIRV::AccessQualifier::AccessQualifier AccQual,
1142:     bool ExplicitLayoutRequired, bool EmitIR) {
1143:   // Treat <1 x T> as T.
1144:   if (auto *FVT = dyn_cast<FixedVectorType>(Ty);
1145:       FVT && FVT->getNumElements() == 1)
1146:     return findSPIRVType(FVT->getElementType(), MIRBuilder, AccQual,
1147:                          ExplicitLayoutRequired, EmitIR);
1148:   Ty = adjustIntTypeByWidth(Ty);
1149:   // TODO: findMI needs to know if a layout is required.
1150:   if (const MachineInstr *MI =
1151:           findMI(Ty, ExplicitLayoutRequired, &MIRBuilder.getMF()))
1152:     return MI;
1153:   if (auto It = ForwardPointerTypes.find(Ty); It != ForwardPointerTypes.end())
1154:     return It->second;
1155:   return restOfCreateSPIRVType(Ty, MIRBuilder, AccQual, ExplicitLayoutRequired,
1156:                                EmitIR);
1157: }
1158:
1159: Register SPIRVGlobalRegistry::getSPIRVTypeID(SPIRVTypeInst SpirvType) const {
1160:   assert(SpirvType && "Attempting to get type id for nullptr type.");
1161:   if (SpirvType->getOpcode() == SPIRV::OpTypeForwardPointer ||
1162:       SpirvType->getOpcode() == SPIRV::OpTypeStructContinuedINTEL)
1163:     return SpirvType->uses().begin()->getReg();
1164:   return SpirvType->defs().begin()->getReg();
1165: }
1166:
1167: // We need to use a new LLVM integer type if there is a mismatch between
1168: // number of bits in LLVM and SPIRV integer types to let DuplicateTracker
1169: // ensure uniqueness of a SPIRV type by the corresponding LLVM type. Without
1170: // such an adjustment SPIRVGlobalRegistry::getOpTypeInt() could create the
```
- EN: This range implements operational logic in helpers such as createTypeVReg, buildInstr, addDef, addImm, translating backend policy into executable code.
- CN: 这一段实现了 createTypeVReg、buildInstr、addDef、addImm 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1260
```cpp
1171: // same "OpTypeInt 8" type for a series of LLVM integer types with number of
1172: // bits less than 8. This would lead to duplicate type definitions
1173: // eventually due to the method that DuplicateTracker utilizes to reason
1174: // about uniqueness of type records.
1175: const Type *SPIRVGlobalRegistry::adjustIntTypeByWidth(const Type *Ty) const {
1176:   if (auto IType = dyn_cast<IntegerType>(Ty)) {
1177:     unsigned SrcBitWidth = IType->getBitWidth();
1178:     if (SrcBitWidth > 1) {
1179:       unsigned BitWidth = adjustOpTypeIntWidth(SrcBitWidth);
1180:       // Maybe change source LLVM type to keep DuplicateTracker consistent.
1181:       if (SrcBitWidth != BitWidth)
1182:         Ty = IntegerType::get(Ty->getContext(), BitWidth);
1183:     }
1184:   }
1185:   return Ty;
1186: }
1187:
1188: SPIRVTypeInst SPIRVGlobalRegistry::createSPIRVType(
1189:     const Type *Ty, MachineIRBuilder &MIRBuilder,
1190:     SPIRV::AccessQualifier::AccessQualifier AccQual,
1191:     bool ExplicitLayoutRequired, bool EmitIR) {
1192:   if (isSpecialOpaqueType(Ty))
1193:     return getOrCreateSpecialType(Ty, MIRBuilder, AccQual);
1194:
1195:   if (const MachineInstr *MI =
1196:           findMI(Ty, ExplicitLayoutRequired, &MIRBuilder.getMF()))
1197:     return MI;
1198:
1199:   if (auto IType = dyn_cast<IntegerType>(Ty)) {
1200:     const unsigned Width = IType->getBitWidth();
1201:     return Width == 1 ? getOpTypeBool(MIRBuilder)
1202:                       : getOpTypeInt(Width, MIRBuilder, false);
1203:   }
1204:   if (Ty->isFloatingPointTy()) {
1205:     if (Ty->isBFloatTy()) {
1206:       return getOpTypeFloat(Ty->getPrimitiveSizeInBits(), MIRBuilder,
1207:                             SPIRV::FPEncoding::BFloat16KHR);
1208:     } else {
1209:       return getOpTypeFloat(Ty->getPrimitiveSizeInBits(), MIRBuilder);
1210:     }
1211:   }
1212:   if (Ty->isVoidTy())
1213:     return getOpTypeVoid(MIRBuilder);
1214:   if (Ty->isVectorTy()) {
1215:     SPIRVTypeInst El =
1216:         findSPIRVType(cast<FixedVectorType>(Ty)->getElementType(), MIRBuilder,
1217:                       AccQual, ExplicitLayoutRequired, EmitIR);
1218:     return getOpTypeVector(cast<FixedVectorType>(Ty)->getNumElements(), El,
1219:                            MIRBuilder);
1220:   }
1221:   if (Ty->isArrayTy()) {
1222:     SPIRVTypeInst El = findSPIRVType(Ty->getArrayElementType(), MIRBuilder,
1223:                                      AccQual, ExplicitLayoutRequired, EmitIR);
1224:     return getOpTypeArray(Ty->getArrayNumElements(), El, MIRBuilder,
1225:                           ExplicitLayoutRequired, EmitIR);
1226:   }
1227:   if (auto SType = dyn_cast<StructType>(Ty)) {
1228:     if (SType->isOpaque())
1229:       return getOpTypeOpaque(SType, MIRBuilder);
1230:
1231:     StructOffsetDecorator Decorator = nullptr;
1232:     if (ExplicitLayoutRequired) {
1233:       Decorator = [&MIRBuilder, SType, this](Register Reg) {
1234:         addStructOffsetDecorations(Reg, const_cast<StructType *>(SType),
1235:                                    MIRBuilder);
1236:       };
1237:     }
1238:     return getOpTypeStruct(SType, MIRBuilder, AccQual, std::move(Decorator),
1239:                            EmitIR);
1240:   }
1241:   if (auto FType = dyn_cast<FunctionType>(Ty)) {
1242:     SPIRVTypeInst RetTy =
1243:         findSPIRVType(FType->getReturnType(), MIRBuilder, AccQual,
1244:                       ExplicitLayoutRequired, EmitIR);
1245:     SmallVector<SPIRVTypeInst, 4> ParamTypes;
1246:     for (const auto &ParamTy : FType->params())
1247:       ParamTypes.push_back(findSPIRVType(ParamTy, MIRBuilder, AccQual,
1248:                                          ExplicitLayoutRequired, EmitIR));
1249:     return getOpTypeFunction(FType, RetTy, ParamTypes, MIRBuilder);
1250:   }
1251:
1252:   unsigned AddrSpace = typeToAddressSpace(Ty);
1253:
1254:   // Get access to information about available extensions
1255:   const SPIRVSubtarget *ST =
1256:       static_cast<const SPIRVSubtarget *>(&MIRBuilder.getMF().getSubtarget());
1257:   auto SC = addressSpaceToStorageClass(AddrSpace, *ST);
1258:
1259:   SPIRVTypeInst SpvElementType = nullptr;
1260:   Type *ElemTy = ::getPointeeType(Ty);
```
- EN: This range implements operational logic in helpers such as SPIRVGlobalRegistry::adjustIntTypeByWidth, getBitWidth, adjustOpTypeIntWidth, IntegerType::get, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVGlobalRegistry::adjustIntTypeByWidth、getBitWidth、adjustOpTypeIntWidth、IntegerType::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1261-1350
```cpp
1261:   if (ElemTy && isa<FunctionType>(ElemTy) &&
1262:       !ST->canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers))
1263:     ElemTy = nullptr;
1264:   if (ElemTy)
1265:     SpvElementType = getOrCreateSPIRVType(ElemTy, MIRBuilder, AccQual, EmitIR);
1266:   else
1267:     SpvElementType = getOrCreateSPIRVIntegerType(8, MIRBuilder);
1268:
1269:   if (!ElemTy) {
1270:     ElemTy = Type::getInt8Ty(MIRBuilder.getContext());
1271:   }
1272:
1273:   // If we have forward pointer associated with this type, use its register
1274:   // operand to create OpTypePointer.
1275:   if (auto It = ForwardPointerTypes.find(Ty); It != ForwardPointerTypes.end()) {
1276:     Register Reg = getSPIRVTypeID(It->second);
1277:     // TODO: what does getOpTypePointer do?
1278:     return getOpTypePointer(SC, SpvElementType, MIRBuilder, Reg);
1279:   }
1280:
1281:   return getOrCreateSPIRVPointerType(ElemTy, MIRBuilder, SC);
1282: }
1283:
1284: SPIRVTypeInst SPIRVGlobalRegistry::restOfCreateSPIRVType(
1285:     const Type *Ty, MachineIRBuilder &MIRBuilder,
1286:     SPIRV::AccessQualifier::AccessQualifier AccessQual,
1287:     bool ExplicitLayoutRequired, bool EmitIR) {
1288:   // TODO: Could this create a problem if one requires an explicit layout, and
1289:   // the next time it does not?
1290:   if (TypesInProcessing.count(Ty) && !isPointerTyOrWrapper(Ty))
1291:     return nullptr;
1292:   TypesInProcessing.insert(Ty);
1293:   SPIRVTypeInst SpirvType = createSPIRVType(Ty, MIRBuilder, AccessQual,
1294:                                             ExplicitLayoutRequired, EmitIR);
1295:   TypesInProcessing.erase(Ty);
1296:   VRegToTypeMap[&MIRBuilder.getMF()][getSPIRVTypeID(SpirvType)] = SpirvType;
1297:
1298:   // TODO: We could end up with two SPIR-V types pointing to the same llvm type.
1299:   // Is that a problem?
1300:   SPIRVToLLVMType[SpirvType] = unifyPtrType(Ty);
1301:
1302:   if (SpirvType->getOpcode() == SPIRV::OpTypeForwardPointer ||
1303:       findMI(Ty, false, &MIRBuilder.getMF()) || isSpecialOpaqueType(Ty))
1304:     return SpirvType;
1305:
1306:   if (auto *ExtTy = dyn_cast<TargetExtType>(Ty);
1307:       ExtTy && isTypedPointerWrapper(ExtTy))
1308:     add(ExtTy->getTypeParameter(0), ExtTy->getIntParameter(0), SpirvType);
1309:   else if (!isPointerTy(Ty))
1310:     add(Ty, ExplicitLayoutRequired, SpirvType);
1311:   else if (isTypedPointerTy(Ty))
1312:     add(cast<TypedPointerType>(Ty)->getElementType(),
1313:         getPointerAddressSpace(Ty), SpirvType);
1314:   else
1315:     add(Type::getInt8Ty(MIRBuilder.getMF().getFunction().getContext()),
1316:         getPointerAddressSpace(Ty), SpirvType);
1317:   return SpirvType;
1318: }
1319:
1320: SPIRVTypeInst
1321: SPIRVGlobalRegistry::getSPIRVTypeForVReg(Register VReg,
1322:                                          const MachineFunction *MF) const {
1323:   auto t = VRegToTypeMap.find(MF ? MF : CurMF);
1324:   if (t != VRegToTypeMap.end()) {
1325:     auto tt = t->second.find(VReg);
1326:     if (tt != t->second.end())
1327:       return tt->second;
1328:   }
1329:   return nullptr;
1330: }
1331:
1332: SPIRVTypeInst SPIRVGlobalRegistry::getResultType(Register VReg,
1333:                                                  MachineFunction *MF) {
1334:   if (!MF)
1335:     MF = CurMF;
1336:   MachineInstr *Instr = getVRegDef(MF->getRegInfo(), VReg);
1337:   return getSPIRVTypeForVReg(Instr->getOperand(1).getReg(), MF);
1338: }
1339:
1340: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVType(
1341:     const Type *Ty, MachineIRBuilder &MIRBuilder,
1342:     SPIRV::AccessQualifier::AccessQualifier AccessQual,
1343:     bool ExplicitLayoutRequired, bool EmitIR) {
1344:   // SPIR-V doesn't support single-element vectors. Treat <1 x T> as T.
1345:   if (auto *FVT = dyn_cast<FixedVectorType>(Ty);
1346:       FVT && FVT->getNumElements() == 1)
1347:     return getOrCreateSPIRVType(FVT->getElementType(), MIRBuilder, AccessQual,
1348:                                 ExplicitLayoutRequired, EmitIR);
1349:   const MachineFunction *MF = &MIRBuilder.getMF();
1350:   Register Reg;
```
- EN: This range implements operational logic in helpers such as canUseExtension, getOrCreateSPIRVType, getOrCreateSPIRVIntegerType, Type::getInt8Ty, translating backend policy into executable code.
- CN: 这一段实现了 canUseExtension、getOrCreateSPIRVType、getOrCreateSPIRVIntegerType、Type::getInt8Ty 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1351-1440
```cpp
1351:   if (auto *ExtTy = dyn_cast<TargetExtType>(Ty);
1352:       ExtTy && isTypedPointerWrapper(ExtTy))
1353:     Reg = find(ExtTy->getTypeParameter(0), ExtTy->getIntParameter(0), MF);
1354:   else if (!isPointerTy(Ty))
1355:     Reg = find(Ty = adjustIntTypeByWidth(Ty), ExplicitLayoutRequired, MF);
1356:   else if (isTypedPointerTy(Ty))
1357:     Reg = find(cast<TypedPointerType>(Ty)->getElementType(),
1358:                getPointerAddressSpace(Ty), MF);
1359:   else
1360:     Reg = find(Type::getInt8Ty(MIRBuilder.getMF().getFunction().getContext()),
1361:                getPointerAddressSpace(Ty), MF);
1362:   if (Reg.isValid() && !isSpecialOpaqueType(Ty))
1363:     return getSPIRVTypeForVReg(Reg);
1364:
1365:   TypesInProcessing.clear();
1366:   SPIRVTypeInst STy = restOfCreateSPIRVType(Ty, MIRBuilder, AccessQual,
1367:                                             ExplicitLayoutRequired, EmitIR);
1368:   // Create normal pointer types for the corresponding OpTypeForwardPointers.
1369:   for (auto &CU : ForwardPointerTypes) {
1370:     // Pointer type themselves do not require an explicit layout. The types
1371:     // they pointer to might, but that is taken care of when creating the type.
1372:     bool PtrNeedsLayout = false;
1373:     const Type *Ty2 = CU.first;
1374:     SPIRVTypeInst STy2 = CU.second;
1375:     if ((Reg = find(Ty2, PtrNeedsLayout, MF)).isValid())
1376:       STy2 = getSPIRVTypeForVReg(Reg);
1377:     else
1378:       STy2 = restOfCreateSPIRVType(Ty2, MIRBuilder, AccessQual, PtrNeedsLayout,
1379:                                    EmitIR);
1380:     if (Ty == Ty2)
1381:       STy = STy2;
1382:   }
1383:   ForwardPointerTypes.clear();
1384:   return STy;
1385: }
1386:
1387: bool SPIRVGlobalRegistry::isScalarOfType(Register VReg,
1388:                                          unsigned TypeOpcode) const {
1389:   SPIRVTypeInst Type = getSPIRVTypeForVReg(VReg);
1390:   assert(Type && "isScalarOfType VReg has no type assigned");
1391:   return Type->getOpcode() == TypeOpcode;
1392: }
1393:
1394: bool SPIRVGlobalRegistry::isScalarOrVectorOfType(Register VReg,
1395:                                                  unsigned TypeOpcode) const {
1396:   SPIRVTypeInst Type = getSPIRVTypeForVReg(VReg);
1397:   assert(Type && "isScalarOrVectorOfType VReg has no type assigned");
1398:   if (Type->getOpcode() == TypeOpcode)
1399:     return true;
1400:   if (Type->getOpcode() == SPIRV::OpTypeVector) {
1401:     Register ScalarTypeVReg = Type->getOperand(1).getReg();
1402:     SPIRVTypeInst ScalarType = getSPIRVTypeForVReg(ScalarTypeVReg);
1403:     return ScalarType->getOpcode() == TypeOpcode;
1404:   }
1405:   return false;
1406: }
1407:
1408: bool SPIRVGlobalRegistry::isResourceType(SPIRVTypeInst Type) const {
1409:   switch (Type->getOpcode()) {
1410:   case SPIRV::OpTypeImage:
1411:   case SPIRV::OpTypeSampler:
1412:   case SPIRV::OpTypeSampledImage:
1413:     return true;
1414:   case SPIRV::OpTypeStruct:
1415:     return hasBlockDecoration(Type);
1416:   default:
1417:     return false;
1418:   }
1419:   return false;
1420: }
1421: unsigned
1422: SPIRVGlobalRegistry::getScalarOrVectorComponentCount(Register VReg) const {
1423:   return getScalarOrVectorComponentCount(getSPIRVTypeForVReg(VReg));
1424: }
1425:
1426: unsigned
1427: SPIRVGlobalRegistry::getScalarOrVectorComponentCount(SPIRVTypeInst Type) const {
1428:   if (!Type)
1429:     return 0;
1430:   return Type->getOpcode() == SPIRV::OpTypeVector
1431:              ? static_cast<unsigned>(Type->getOperand(2).getImm())
1432:              : 1;
1433: }
1434:
1435: SPIRVTypeInst
1436: SPIRVGlobalRegistry::getScalarOrVectorComponentType(SPIRVTypeInst Type) const {
1437:   if (!Type)
1438:     return nullptr;
1439:   Register ScalarReg = Type->getOpcode() == SPIRV::OpTypeVector
1440:                            ? Type->getOperand(1).getReg()
```
- EN: This range implements operational logic in helpers such as isTypedPointerWrapper, find, getPointerAddressSpace, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 isTypedPointerWrapper、find、getPointerAddressSpace、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1441-1530
```cpp
1441:                            : Type->getOperand(0).getReg();
1442:   SPIRVTypeInst ScalarType = getSPIRVTypeForVReg(ScalarReg);
1443:   assert(isScalarOrVectorOfType(Type->getOperand(0).getReg(),
1444:                                 ScalarType->getOpcode()));
1445:   return ScalarType;
1446: }
1447:
1448: unsigned
1449: SPIRVGlobalRegistry::getScalarOrVectorBitWidth(SPIRVTypeInst Type) const {
1450:   assert(Type && "Invalid Type pointer");
1451:   SPIRVTypeInst ScalarType = getScalarOrVectorComponentType(Type);
1452:   if (ScalarType->getOpcode() == SPIRV::OpTypeInt ||
1453:       ScalarType->getOpcode() == SPIRV::OpTypeFloat)
1454:     return ScalarType->getOperand(1).getImm();
1455:   if (ScalarType->getOpcode() == SPIRV::OpTypeBool)
1456:     return 1;
1457:   llvm_unreachable("Attempting to get bit width of non-integer/float type.");
1458: }
1459:
1460: unsigned SPIRVGlobalRegistry::getNumScalarOrVectorTotalBitWidth(
1461:     SPIRVTypeInst Type) const {
1462:   assert(Type && "Invalid Type pointer");
1463:   unsigned NumElements = getScalarOrVectorComponentCount(Type);
1464:   SPIRVTypeInst ScalarType = getScalarOrVectorComponentType(Type);
1465:   return ScalarType->getOpcode() == SPIRV::OpTypeInt ||
1466:                  ScalarType->getOpcode() == SPIRV::OpTypeFloat
1467:              ? NumElements * ScalarType->getOperand(1).getImm()
1468:              : 0;
1469: }
1470:
1471: SPIRVTypeInst
1472: SPIRVGlobalRegistry::retrieveScalarOrVectorIntType(SPIRVTypeInst Type) const {
1473:   SPIRVTypeInst ScalarType = getScalarOrVectorComponentType(Type);
1474:   return ScalarType && ScalarType->getOpcode() == SPIRV::OpTypeInt ? ScalarType
1475:                                                                    : nullptr;
1476: }
1477:
1478: bool SPIRVGlobalRegistry::isScalarOrVectorSigned(SPIRVTypeInst Type) const {
1479:   SPIRVTypeInst IntType = retrieveScalarOrVectorIntType(Type);
1480:   return IntType && IntType->getOperand(2).getImm() != 0;
1481: }
1482:
1483: SPIRVTypeInst SPIRVGlobalRegistry::getPointeeType(SPIRVTypeInst PtrType) {
1484:   return PtrType && PtrType->getOpcode() == SPIRV::OpTypePointer
1485:              ? getSPIRVTypeForVReg(PtrType->getOperand(2).getReg())
1486:              : nullptr;
1487: }
1488:
1489: unsigned SPIRVGlobalRegistry::getPointeeTypeOp(Register PtrReg) {
1490:   SPIRVTypeInst ElemType = getPointeeType(getSPIRVTypeForVReg(PtrReg));
1491:   return ElemType ? ElemType->getOpcode() : 0;
1492: }
1493:
1494: bool SPIRVGlobalRegistry::isBitcastCompatible(SPIRVTypeInst Type1,
1495:                                               SPIRVTypeInst Type2) const {
1496:   if (!Type1 || !Type2)
1497:     return false;
1498:   auto Op1 = Type1->getOpcode(), Op2 = Type2->getOpcode();
1499:   // Ignore difference between <1.5 and >=1.5 protocol versions:
1500:   // it's valid if either Result Type or Operand is a pointer, and the other
1501:   // is a pointer, an integer scalar, or an integer vector.
1502:   if (Op1 == SPIRV::OpTypePointer &&
1503:       (Op2 == SPIRV::OpTypePointer || retrieveScalarOrVectorIntType(Type2)))
1504:     return true;
1505:   if (Op2 == SPIRV::OpTypePointer &&
1506:       (Op1 == SPIRV::OpTypePointer || retrieveScalarOrVectorIntType(Type1)))
1507:     return true;
1508:   unsigned Bits1 = getNumScalarOrVectorTotalBitWidth(Type1),
1509:            Bits2 = getNumScalarOrVectorTotalBitWidth(Type2);
1510:   return Bits1 > 0 && Bits1 == Bits2;
1511: }
1512:
1513: SPIRV::StorageClass::StorageClass
1514: SPIRVGlobalRegistry::getPointerStorageClass(Register VReg) const {
1515:   SPIRVTypeInst Type = getSPIRVTypeForVReg(VReg);
1516:   assert(Type && Type->getOpcode() == SPIRV::OpTypePointer &&
1517:          Type->getOperand(1).isImm() && "Pointer type is expected");
1518:   return getPointerStorageClass(Type);
1519: }
1520:
1521: SPIRV::StorageClass::StorageClass
1522: SPIRVGlobalRegistry::getPointerStorageClass(SPIRVTypeInst Type) const {
1523:   return static_cast<SPIRV::StorageClass::StorageClass>(
1524:       Type->getOperand(1).getImm());
1525: }
1526:
1527: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateVulkanBufferType(
1528:     MachineIRBuilder &MIRBuilder, Type *ElemType,
1529:     SPIRV::StorageClass::StorageClass SC, bool IsWritable, bool EmitIr) {
1530:   auto Key = SPIRV::irhandle_vkbuffer(ElemType, SC, IsWritable);
```
- EN: This range implements operational logic in helpers such as getOperand, getSPIRVTypeForVReg, getOpcode, SPIRVGlobalRegistry::getScalarOrVectorBitWidth, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getSPIRVTypeForVReg、getOpcode、SPIRVGlobalRegistry::getScalarOrVectorBitWidth 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1531-1620
```cpp
1531:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1532:     return MI;
1533:
1534:   bool ExplicitLayoutRequired = storageClassRequiresExplictLayout(SC);
1535:   // We need to get the SPIR-V type for the element here, so we can add the
1536:   // decoration to it.
1537:   auto *T = StructType::create(ElemType);
1538:   SPIRVTypeInst BlockType =
1539:       getOrCreateSPIRVType(T, MIRBuilder, SPIRV::AccessQualifier::None,
1540:                            ExplicitLayoutRequired, EmitIr);
1541:
1542:   buildOpDecorate(BlockType->defs().begin()->getReg(), MIRBuilder,
1543:                   SPIRV::Decoration::Block, {});
1544:
1545:   if (!IsWritable) {
1546:     buildOpMemberDecorate(BlockType->defs().begin()->getReg(), MIRBuilder,
1547:                           SPIRV::Decoration::NonWritable, 0, {});
1548:   }
1549:
1550:   SPIRVTypeInst R =
1551:       getOrCreateSPIRVPointerTypeInternal(BlockType, MIRBuilder, SC);
1552:   add(Key, R);
1553:   return R;
1554: }
1555:
1556: SPIRVTypeInst
1557: SPIRVGlobalRegistry::getOrCreatePaddingType(MachineIRBuilder &MIRBuilder) {
1558:   auto Key = SPIRV::irhandle_padding();
1559:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1560:     return MI;
1561:   auto *T = Type::getInt8Ty(MIRBuilder.getContext());
1562:   SPIRVTypeInst R = getOrCreateSPIRVIntegerType(8, MIRBuilder);
1563:   finishCreatingSPIRVType(T, R);
1564:   add(Key, R);
1565:   return R;
1566: }
1567:
1568: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateVulkanPushConstantType(
1569:     MachineIRBuilder &MIRBuilder, Type *T) {
1570:   const auto SC = SPIRV::StorageClass::PushConstant;
1571:
1572:   auto Key = SPIRV::irhandle_vkbuffer(T, SC, /* IsWritable= */ false);
1573:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1574:     return MI;
1575:
1576:   // We need to get the SPIR-V type for the element here, so we can add the
1577:   // decoration to it.
1578:   SPIRVTypeInst BlockType = getOrCreateSPIRVType(
1579:       T, MIRBuilder, SPIRV::AccessQualifier::None,
1580:       /* ExplicitLayoutRequired= */ true, /* EmitIr= */ false);
1581:
1582:   buildOpDecorate(BlockType->defs().begin()->getReg(), MIRBuilder,
1583:                   SPIRV::Decoration::Block, {});
1584:   SPIRVTypeInst R = BlockType;
1585:   add(Key, R);
1586:   return R;
1587: }
1588:
1589: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateLayoutType(
1590:     MachineIRBuilder &MIRBuilder, const TargetExtType *T, bool EmitIr) {
1591:   auto Key = SPIRV::handle(T);
1592:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1593:     return MI;
1594:
1595:   StructType *ST = cast<StructType>(T->getTypeParameter(0));
1596:   ArrayRef<uint32_t> Offsets = T->int_params().slice(1);
1597:   assert(ST->getNumElements() == Offsets.size());
1598:
1599:   StructOffsetDecorator Decorator = [&MIRBuilder, &Offsets](Register Reg) {
1600:     for (uint32_t I = 0; I < Offsets.size(); ++I) {
1601:       buildOpMemberDecorate(Reg, MIRBuilder, SPIRV::Decoration::Offset, I,
1602:                             {Offsets[I]});
1603:     }
1604:   };
1605:
1606:   // We need a new OpTypeStruct instruction because decorations will be
1607:   // different from a struct with an explicit layout created from a different
1608:   // entry point.
1609:   SPIRVTypeInst SPIRVStructType =
1610:       getOpTypeStruct(ST, MIRBuilder, SPIRV::AccessQualifier::None,
1611:                       std::move(Decorator), EmitIr);
1612:   add(Key, SPIRVStructType);
1613:   return SPIRVStructType;
1614: }
1615:
1616: SPIRVTypeInst SPIRVGlobalRegistry::getImageType(
1617:     const TargetExtType *ExtensionType,
1618:     const SPIRV::AccessQualifier::AccessQualifier Qualifier,
1619:     MachineIRBuilder &MIRBuilder) {
1620:   assert(ExtensionType->getNumTypeParameters() == 1 &&
```
- EN: This range implements operational logic in helpers such as storageClassRequiresExplictLayout, StructType::create, getOrCreateSPIRVPointerTypeInternal, add, translating backend policy into executable code.
- CN: 这一段实现了 storageClassRequiresExplictLayout、StructType::create、getOrCreateSPIRVPointerTypeInternal、add 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1621-1710
```cpp
1621:          "SPIR-V image builtin type must have sampled type parameter!");
1622:   const SPIRVTypeInst SampledType =
1623:       getOrCreateSPIRVType(ExtensionType->getTypeParameter(0), MIRBuilder,
1624:                            SPIRV::AccessQualifier::ReadWrite, true);
1625:   assert((ExtensionType->getNumIntParameters() == 7 ||
1626:           ExtensionType->getNumIntParameters() == 6) &&
1627:          "Invalid number of parameters for SPIR-V image builtin!");
1628:
1629:   SPIRV::AccessQualifier::AccessQualifier accessQualifier =
1630:       SPIRV::AccessQualifier::None;
1631:   if (ExtensionType->getNumIntParameters() == 7) {
1632:     accessQualifier = Qualifier == SPIRV::AccessQualifier::WriteOnly
1633:                           ? SPIRV::AccessQualifier::WriteOnly
1634:                           : SPIRV::AccessQualifier::AccessQualifier(
1635:                                 ExtensionType->getIntParameter(6));
1636:   }
1637:
1638:   // Create or get an existing type from GlobalRegistry.
1639:   SPIRVTypeInst R = getOrCreateOpTypeImage(
1640:       MIRBuilder, SampledType,
1641:       SPIRV::Dim::Dim(ExtensionType->getIntParameter(0)),
1642:       ExtensionType->getIntParameter(1), ExtensionType->getIntParameter(2),
1643:       ExtensionType->getIntParameter(3), ExtensionType->getIntParameter(4),
1644:       SPIRV::ImageFormat::ImageFormat(ExtensionType->getIntParameter(5)),
1645:       accessQualifier);
1646:   SPIRVToLLVMType[R] = ExtensionType;
1647:   return R;
1648: }
1649:
1650: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypeImage(
1651:     MachineIRBuilder &MIRBuilder, SPIRVTypeInst SampledType,
1652:     SPIRV::Dim::Dim Dim, uint32_t Depth, uint32_t Arrayed,
1653:     uint32_t Multisampled, uint32_t Sampled,
1654:     SPIRV::ImageFormat::ImageFormat ImageFormat,
1655:     SPIRV::AccessQualifier::AccessQualifier AccessQual) {
1656:   auto Key = SPIRV::irhandle_image(SPIRVToLLVMType.lookup(SampledType), Dim,
1657:                                    Depth, Arrayed, Multisampled, Sampled,
1658:                                    ImageFormat, AccessQual);
1659:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1660:     return MI;
1661:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1662:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1663:         auto MIB =
1664:             MIRBuilder.buildInstr(SPIRV::OpTypeImage)
1665:                 .addDef(createTypeVReg(MIRBuilder))
1666:                 .addUse(getSPIRVTypeID(SampledType))
1667:                 .addImm(Dim)
1668:                 .addImm(Depth)   // Depth (whether or not it is a Depth image).
1669:                 .addImm(Arrayed) // Arrayed.
1670:                 .addImm(Multisampled) // Multisampled (0 = only single-sample).
1671:                 .addImm(Sampled)      // Sampled (0 = usage known at runtime).
1672:                 .addImm(ImageFormat);
1673:         if (AccessQual != SPIRV::AccessQualifier::None)
1674:           MIB.addImm(AccessQual);
1675:         return MIB;
1676:       });
1677:   add(Key, NewMI);
1678:   return NewMI;
1679: }
1680:
1681: SPIRVTypeInst
1682: SPIRVGlobalRegistry::getOrCreateOpTypeSampler(MachineIRBuilder &MIRBuilder) {
1683:   auto Key = SPIRV::irhandle_sampler();
1684:   const MachineFunction *MF = &MIRBuilder.getMF();
1685:   if (const MachineInstr *MI = findMI(Key, MF))
1686:     return MI;
1687:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1688:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1689:         return MIRBuilder.buildInstr(SPIRV::OpTypeSampler)
1690:             .addDef(createTypeVReg(MIRBuilder));
1691:       });
1692:   add(Key, NewMI);
1693:   return NewMI;
1694: }
1695:
1696: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypePipe(
1697:     MachineIRBuilder &MIRBuilder,
1698:     SPIRV::AccessQualifier::AccessQualifier AccessQual) {
1699:   auto Key = SPIRV::irhandle_pipe(AccessQual);
1700:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1701:     return MI;
1702:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1703:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1704:         return MIRBuilder.buildInstr(SPIRV::OpTypePipe)
1705:             .addDef(createTypeVReg(MIRBuilder))
1706:             .addImm(AccessQual);
1707:       });
1708:   add(Key, NewMI);
1709:   return NewMI;
1710: }
```
- EN: This range implements operational logic in helpers such as getIntParameter, buildInstr, addDef, addUse, translating backend policy into executable code.
- CN: 这一段实现了 getIntParameter、buildInstr、addDef、addUse 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1711-1800
```cpp
1711:
1712: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypeDeviceEvent(
1713:     MachineIRBuilder &MIRBuilder) {
1714:   auto Key = SPIRV::irhandle_event();
1715:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1716:     return MI;
1717:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1718:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1719:         return MIRBuilder.buildInstr(SPIRV::OpTypeDeviceEvent)
1720:             .addDef(createTypeVReg(MIRBuilder));
1721:       });
1722:   add(Key, NewMI);
1723:   return NewMI;
1724: }
1725:
1726: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypeSampledImage(
1727:     SPIRVTypeInst ImageType, MachineIRBuilder &MIRBuilder) {
1728:   auto Key = SPIRV::irhandle_sampled_image(
1729:       SPIRVToLLVMType.lookup(MIRBuilder.getMF().getRegInfo().getVRegDef(
1730:           ImageType->getOperand(1).getReg())),
1731:       ImageType);
1732:   if (const MachineInstr *MI = findMI(Key, &MIRBuilder.getMF()))
1733:     return MI;
1734:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1735:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1736:         return MIRBuilder.buildInstr(SPIRV::OpTypeSampledImage)
1737:             .addDef(createTypeVReg(MIRBuilder))
1738:             .addUse(getSPIRVTypeID(ImageType));
1739:       });
1740:   add(Key, NewMI);
1741:   return NewMI;
1742: }
1743:
1744: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypeCoopMatr(
1745:     MachineIRBuilder &MIRBuilder, const TargetExtType *ExtensionType,
1746:     SPIRVTypeInst ElemType, uint32_t Scope, uint32_t Rows, uint32_t Columns,
1747:     uint32_t Use, bool EmitIR) {
1748:   if (const MachineInstr *MI =
1749:           findMI(ExtensionType, false, &MIRBuilder.getMF()))
1750:     return MI;
1751:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1752:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1753:         SPIRVTypeInst SpvTypeInt32 =
1754:             getOrCreateSPIRVIntegerType(32, MIRBuilder);
1755:         const Type *ET = getTypeForSPIRVType(ElemType);
1756:         if (ET->isIntegerTy() && ET->getIntegerBitWidth() == 4 &&
1757:             cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget())
1758:                 .canUseExtension(SPIRV::Extension::SPV_INTEL_int4)) {
1759:           MIRBuilder.buildInstr(SPIRV::OpCapability)
1760:               .addImm(SPIRV::Capability::Int4CooperativeMatrixINTEL);
1761:         }
1762:         return MIRBuilder.buildInstr(SPIRV::OpTypeCooperativeMatrixKHR)
1763:             .addDef(createTypeVReg(MIRBuilder))
1764:             .addUse(getSPIRVTypeID(ElemType))
1765:             .addUse(buildConstantInt(Scope, MIRBuilder, SpvTypeInt32, EmitIR))
1766:             .addUse(buildConstantInt(Rows, MIRBuilder, SpvTypeInt32, EmitIR))
1767:             .addUse(buildConstantInt(Columns, MIRBuilder, SpvTypeInt32, EmitIR))
1768:             .addUse(buildConstantInt(Use, MIRBuilder, SpvTypeInt32, EmitIR));
1769:       });
1770:   add(ExtensionType, false, NewMI);
1771:   return NewMI;
1772: }
1773:
1774: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateOpTypeByOpcode(
1775:     const Type *Ty, MachineIRBuilder &MIRBuilder, unsigned Opcode) {
1776:   if (const MachineInstr *MI = findMI(Ty, false, &MIRBuilder.getMF()))
1777:     return MI;
1778:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1779:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1780:         return MIRBuilder.buildInstr(Opcode).addDef(createTypeVReg(MIRBuilder));
1781:       });
1782:   add(Ty, false, NewMI);
1783:   return NewMI;
1784: }
1785:
1786: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateUnknownType(
1787:     const Type *Ty, MachineIRBuilder &MIRBuilder, unsigned Opcode,
1788:     const ArrayRef<MCOperand> Operands) {
1789:   if (const MachineInstr *MI = findMI(Ty, false, &MIRBuilder.getMF()))
1790:     return MI;
1791:   Register ResVReg = createTypeVReg(MIRBuilder);
1792:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1793:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1794:         MachineInstrBuilder MIB = MIRBuilder.buildInstr(SPIRV::UNKNOWN_type)
1795:                                       .addDef(ResVReg)
1796:                                       .addImm(Opcode);
1797:         for (MCOperand Operand : Operands) {
1798:           if (Operand.isReg()) {
1799:             MIB.addUse(Operand.getReg());
1800:           } else if (Operand.isImm()) {
```
- EN: This range implements operational logic in helpers such as SPIRV::irhandle_event, buildInstr, addDef, add, translating backend policy into executable code.
- CN: 这一段实现了 SPIRV::irhandle_event、buildInstr、addDef、add 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1801-1890
```cpp
1801:             MIB.addImm(Operand.getImm());
1802:           }
1803:         }
1804:         return MIB;
1805:       });
1806:   add(Ty, false, NewMI);
1807:   return NewMI;
1808: }
1809:
1810: // Returns nullptr if unable to recognize SPIRV type name
1811: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVTypeByName(
1812:     StringRef TypeStr, MachineIRBuilder &MIRBuilder, bool EmitIR,
1813:     SPIRV::StorageClass::StorageClass SC,
1814:     SPIRV::AccessQualifier::AccessQualifier AQ) {
1815:   unsigned VecElts = 0;
1816:   auto &Ctx = MIRBuilder.getMF().getFunction().getContext();
1817:
1818:   // Parse strings representing either a SPIR-V or OpenCL builtin type.
1819:   if (hasBuiltinTypePrefix(TypeStr))
1820:     return getOrCreateSPIRVType(SPIRV::parseBuiltinTypeNameToTargetExtType(
1821:                                     TypeStr.str(), MIRBuilder.getContext()),
1822:                                 MIRBuilder, AQ, false, true);
1823:
1824:   // Parse type name in either "typeN" or "type vector[N]" format, where
1825:   // N is the number of elements of the vector.
1826:   Type *Ty;
1827:
1828:   Ty = parseBasicTypeName(TypeStr, Ctx);
1829:   if (!Ty)
1830:     // Unable to recognize SPIRV type name
1831:     return nullptr;
1832:
1833:   SPIRVTypeInst SpirvTy = getOrCreateSPIRVType(Ty, MIRBuilder, AQ, false, true);
1834:
1835:   // Handle "type*" or  "type* vector[N]".
1836:   if (TypeStr.consume_front("*"))
1837:     SpirvTy = getOrCreateSPIRVPointerType(Ty, MIRBuilder, SC);
1838:
1839:   // Handle "typeN*" or  "type vector[N]*".
1840:   bool IsPtrToVec = TypeStr.consume_back("*");
1841:
1842:   if (TypeStr.consume_front(" vector[")) {
1843:     TypeStr = TypeStr.substr(0, TypeStr.find(']'));
1844:   }
1845:   TypeStr.getAsInteger(10, VecElts);
1846:   if (VecElts > 0)
1847:     SpirvTy = getOrCreateSPIRVVectorType(SpirvTy, VecElts, MIRBuilder, EmitIR);
1848:
1849:   if (IsPtrToVec)
1850:     SpirvTy = getOrCreateSPIRVPointerType(SpirvTy, MIRBuilder, SC);
1851:
1852:   return SpirvTy;
1853: }
1854:
1855: SPIRVTypeInst
1856: SPIRVGlobalRegistry::getOrCreateSPIRVIntegerType(unsigned BitWidth,
1857:                                                  MachineIRBuilder &MIRBuilder) {
1858:   return getOrCreateSPIRVType(
1859:       IntegerType::get(MIRBuilder.getMF().getFunction().getContext(), BitWidth),
1860:       MIRBuilder, SPIRV::AccessQualifier::ReadWrite, false, true);
1861: }
1862:
1863: SPIRVTypeInst
1864: SPIRVGlobalRegistry::finishCreatingSPIRVType(const Type *LLVMTy,
1865:                                              SPIRVTypeInst SpirvType) {
1866:   assert(CurMF == SpirvType->getMF());
1867:   VRegToTypeMap[CurMF][getSPIRVTypeID(SpirvType)] = SpirvType;
1868:   SPIRVToLLVMType[SpirvType] = unifyPtrType(LLVMTy);
1869:   return SpirvType;
1870: }
1871:
1872: SPIRVTypeInst
1873: SPIRVGlobalRegistry::getOrCreateSPIRVType(unsigned BitWidth, MachineInstr &I,
1874:                                           const SPIRVInstrInfo &TII,
1875:                                           unsigned SPIRVOPcode, Type *Ty) {
1876:   if (const MachineInstr *MI = findMI(Ty, false, CurMF))
1877:     return MI;
1878:   MachineBasicBlock &DepMBB = I.getMF()->front();
1879:   MachineIRBuilder MIRBuilder(DepMBB, DepMBB.getFirstNonPHI());
1880:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1881:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1882:         auto NewTypeMI = BuildMI(MIRBuilder.getMBB(), *MIRBuilder.getInsertPt(),
1883:                                  MIRBuilder.getDL(), TII.get(SPIRVOPcode))
1884:                              .addDef(createTypeVReg(CurMF->getRegInfo()))
1885:                              .addImm(BitWidth);
1886:         // Don't add Encoding to FP type
1887:         if (!Ty->isFloatTy()) {
1888:           return NewTypeMI.addImm(0);
1889:         } else {
1890:           return NewTypeMI;
```
- EN: This range implements operational logic in helpers such as addImm, add, getMF, parseBasicTypeName, translating backend policy into executable code.
- CN: 这一段实现了 addImm、add、getMF、parseBasicTypeName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1891-1980
```cpp
1891:         }
1892:       });
1893:   add(Ty, false, NewMI);
1894:   return finishCreatingSPIRVType(Ty, NewMI);
1895: }
1896:
1897: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVIntegerType(
1898:     unsigned BitWidth, MachineInstr &I, const SPIRVInstrInfo &TII) {
1899:   // Maybe adjust bit width to keep DuplicateTracker consistent. Without
1900:   // such an adjustment SPIRVGlobalRegistry::getOpTypeInt() could create, for
1901:   // example, the same "OpTypeInt 8" type for a series of LLVM integer types
1902:   // with number of bits less than 8, causing duplicate type definitions.
1903:   if (BitWidth > 1)
1904:     BitWidth = adjustOpTypeIntWidth(BitWidth);
1905:   Type *LLVMTy = IntegerType::get(CurMF->getFunction().getContext(), BitWidth);
1906:   return getOrCreateSPIRVType(BitWidth, I, TII, SPIRV::OpTypeInt, LLVMTy);
1907: }
1908:
1909: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVFloatType(
1910:     unsigned BitWidth, MachineInstr &I, const SPIRVInstrInfo &TII) {
1911:   LLVMContext &Ctx = CurMF->getFunction().getContext();
1912:   Type *LLVMTy;
1913:   switch (BitWidth) {
1914:   case 16:
1915:     LLVMTy = Type::getHalfTy(Ctx);
1916:     break;
1917:   case 32:
1918:     LLVMTy = Type::getFloatTy(Ctx);
1919:     break;
1920:   case 64:
1921:     LLVMTy = Type::getDoubleTy(Ctx);
1922:     break;
1923:   default:
1924:     llvm_unreachable("Bit width is of unexpected size.");
1925:   }
1926:   return getOrCreateSPIRVType(BitWidth, I, TII, SPIRV::OpTypeFloat, LLVMTy);
1927: }
1928:
1929: SPIRVTypeInst
1930: SPIRVGlobalRegistry::getOrCreateSPIRVBoolType(MachineIRBuilder &MIRBuilder,
1931:                                               bool EmitIR) {
1932:   return getOrCreateSPIRVType(
1933:       IntegerType::get(MIRBuilder.getMF().getFunction().getContext(), 1),
1934:       MIRBuilder, SPIRV::AccessQualifier::ReadWrite, false, EmitIR);
1935: }
1936:
1937: SPIRVTypeInst
1938: SPIRVGlobalRegistry::getOrCreateSPIRVBoolType(MachineInstr &I,
1939:                                               const SPIRVInstrInfo &TII) {
1940:   Type *Ty = IntegerType::get(CurMF->getFunction().getContext(), 1);
1941:   if (const MachineInstr *MI = findMI(Ty, false, CurMF))
1942:     return MI;
1943:   MachineBasicBlock &DepMBB = I.getMF()->front();
1944:   MachineIRBuilder MIRBuilder(DepMBB, DepMBB.getFirstNonPHI());
1945:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1946:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1947:         return BuildMI(MIRBuilder.getMBB(), *MIRBuilder.getInsertPt(),
1948:                        MIRBuilder.getDL(), TII.get(SPIRV::OpTypeBool))
1949:             .addDef(createTypeVReg(CurMF->getRegInfo()));
1950:       });
1951:   add(Ty, false, NewMI);
1952:   return finishCreatingSPIRVType(Ty, NewMI);
1953: }
1954:
1955: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVVectorType(
1956:     SPIRVTypeInst BaseType, unsigned NumElements, MachineIRBuilder &MIRBuilder,
1957:     bool EmitIR) {
1958:   return getOrCreateSPIRVType(
1959:       FixedVectorType::get(const_cast<Type *>(getTypeForSPIRVType(BaseType)),
1960:                            NumElements),
1961:       MIRBuilder, SPIRV::AccessQualifier::ReadWrite, false, EmitIR);
1962: }
1963:
1964: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVVectorType(
1965:     SPIRVTypeInst BaseType, unsigned NumElements, MachineInstr &I,
1966:     const SPIRVInstrInfo &TII) {
1967:   // At this point of time all 1-element vectors are resolved. Add assertion
1968:   // to fire if anything changes.
1969:   assert(NumElements >= 2 && "SPIR-V vectors must have at least 2 components");
1970:   Type *Ty = FixedVectorType::get(
1971:       const_cast<Type *>(getTypeForSPIRVType(BaseType)), NumElements);
1972:   if (const MachineInstr *MI = findMI(Ty, false, CurMF))
1973:     return MI;
1974:   MachineInstr *DepMI =
1975:       const_cast<MachineInstr *>(static_cast<const MachineInstr *>(BaseType));
1976:   MachineIRBuilder MIRBuilder(*DepMI->getParent(), DepMI->getIterator());
1977:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
1978:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
1979:         return BuildMI(MIRBuilder.getMBB(), *MIRBuilder.getInsertPt(),
1980:                        MIRBuilder.getDL(), TII.get(SPIRV::OpTypeVector))
```
- EN: This range implements operational logic in helpers such as add, finishCreatingSPIRVType, adjustOpTypeIntWidth, IntegerType::get, translating backend policy into executable code.
- CN: 这一段实现了 add、finishCreatingSPIRVType、adjustOpTypeIntWidth、IntegerType::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1981-2070
```cpp
1981:             .addDef(createTypeVReg(CurMF->getRegInfo()))
1982:             .addUse(getSPIRVTypeID(BaseType))
1983:             .addImm(NumElements);
1984:       });
1985:   add(Ty, false, NewMI);
1986:   return finishCreatingSPIRVType(Ty, NewMI);
1987: }
1988:
1989: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVPointerType(
1990:     const Type *BaseType, MachineInstr &I,
1991:     SPIRV::StorageClass::StorageClass SC) {
1992:   MachineIRBuilder MIRBuilder(I);
1993:   return getOrCreateSPIRVPointerType(BaseType, MIRBuilder, SC);
1994: }
1995:
1996: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVPointerType(
1997:     const Type *BaseType, MachineIRBuilder &MIRBuilder,
1998:     SPIRV::StorageClass::StorageClass SC) {
1999:   // TODO: Need to check if EmitIr should always be true.
2000:   SPIRVTypeInst SpirvBaseType = getOrCreateSPIRVType(
2001:       BaseType, MIRBuilder, SPIRV::AccessQualifier::ReadWrite,
2002:       storageClassRequiresExplictLayout(SC), true);
2003:   assert(SpirvBaseType);
2004:   return getOrCreateSPIRVPointerTypeInternal(SpirvBaseType, MIRBuilder, SC);
2005: }
2006:
2007: SPIRVTypeInst SPIRVGlobalRegistry::changePointerStorageClass(
2008:     SPIRVTypeInst PtrType, SPIRV::StorageClass::StorageClass SC,
2009:     MachineInstr &I) {
2010:   [[maybe_unused]] SPIRV::StorageClass::StorageClass OldSC =
2011:       getPointerStorageClass(PtrType);
2012:   assert(storageClassRequiresExplictLayout(OldSC) ==
2013:          storageClassRequiresExplictLayout(SC));
2014:
2015:   SPIRVTypeInst PointeeType = getPointeeType(PtrType);
2016:   MachineIRBuilder MIRBuilder(I);
2017:   return getOrCreateSPIRVPointerTypeInternal(PointeeType, MIRBuilder, SC);
2018: }
2019:
2020: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVPointerType(
2021:     SPIRVTypeInst BaseType, MachineIRBuilder &MIRBuilder,
2022:     SPIRV::StorageClass::StorageClass SC) {
2023:   const Type *LLVMType = getTypeForSPIRVType(BaseType);
2024:   assert(!storageClassRequiresExplictLayout(SC));
2025:   SPIRVTypeInst R = getOrCreateSPIRVPointerType(LLVMType, MIRBuilder, SC);
2026:   assert(
2027:       getPointeeType(R) == BaseType &&
2028:       "The base type was not correctly laid out for the given storage class.");
2029:   return R;
2030: }
2031:
2032: SPIRVTypeInst SPIRVGlobalRegistry::getOrCreateSPIRVPointerTypeInternal(
2033:     SPIRVTypeInst BaseType, MachineIRBuilder &MIRBuilder,
2034:     SPIRV::StorageClass::StorageClass SC) {
2035:   const Type *PointerElementType = getTypeForSPIRVType(BaseType);
2036:   unsigned AddressSpace = storageClassToAddressSpace(SC);
2037:   if (const MachineInstr *MI = findMI(PointerElementType, AddressSpace, CurMF))
2038:     return MI;
2039:   Type *Ty = TypedPointerType::get(const_cast<Type *>(PointerElementType),
2040:                                    AddressSpace);
2041:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
2042:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
2043:         return BuildMI(MIRBuilder.getMBB(), MIRBuilder.getInsertPt(),
2044:                        MIRBuilder.getDebugLoc(),
2045:                        MIRBuilder.getTII().get(SPIRV::OpTypePointer))
2046:             .addDef(createTypeVReg(CurMF->getRegInfo()))
2047:             .addImm(static_cast<uint32_t>(SC))
2048:             .addUse(getSPIRVTypeID(BaseType));
2049:       });
2050:   add(PointerElementType, AddressSpace, NewMI);
2051:   return finishCreatingSPIRVType(Ty, NewMI);
2052: }
2053:
2054: Register SPIRVGlobalRegistry::getOrCreateUndef(MachineInstr &I,
2055:                                                SPIRVTypeInst SpvType,
2056:                                                const SPIRVInstrInfo &TII) {
2057:   UndefValue *UV =
2058:       UndefValue::get(const_cast<Type *>(getTypeForSPIRVType(SpvType)));
2059:   Register Res = find(UV, CurMF);
2060:   if (Res.isValid())
2061:     return Res;
2062:
2063:   LLT LLTy = LLT::scalar(64);
2064:   Res = CurMF->getRegInfo().createGenericVirtualRegister(LLTy);
2065:   CurMF->getRegInfo().setRegClass(Res, &SPIRV::iIDRegClass);
2066:   assignSPIRVTypeToVReg(SpvType, Res, *CurMF);
2067:
2068:   MachineInstr *DepMI =
2069:       const_cast<MachineInstr *>(static_cast<const MachineInstr *>(SpvType));
2070:   MachineIRBuilder MIRBuilder(*DepMI->getParent(), DepMI->getIterator());
```
- EN: This range defines or declares important types such as addDef, addUse, addImm, add, shaping the data model used by SPIRVGlobalRegistry.cpp.
- CN: 这一段定义或声明了 addDef、addUse、addImm、add 等关键类型，构成 SPIRVGlobalRegistry.cpp 使用的数据模型。

### Lines 2071-2160
```cpp
2071:   const MachineInstr *NewMI = createConstOrTypeAtFunctionEntry(
2072:       MIRBuilder, [&](MachineIRBuilder &MIRBuilder) {
2073:         auto MIB = BuildMI(MIRBuilder.getMBB(), *MIRBuilder.getInsertPt(),
2074:                            MIRBuilder.getDL(), TII.get(SPIRV::OpUndef))
2075:                        .addDef(Res)
2076:                        .addUse(getSPIRVTypeID(SpvType));
2077:         const auto &ST = CurMF->getSubtarget();
2078:         constrainSelectedInstRegOperands(*MIB, *ST.getInstrInfo(),
2079:                                          *ST.getRegisterInfo(),
2080:                                          *ST.getRegBankInfo());
2081:         return MIB;
2082:       });
2083:   add(UV, NewMI);
2084:   return Res;
2085: }
2086:
2087: const TargetRegisterClass *
2088: SPIRVGlobalRegistry::getRegClass(SPIRVTypeInst SpvType) const {
2089:   unsigned Opcode = SpvType->getOpcode();
2090:   switch (Opcode) {
2091:   case SPIRV::OpTypeFloat:
2092:     return &SPIRV::fIDRegClass;
2093:   case SPIRV::OpTypePointer:
2094:     return &SPIRV::pIDRegClass;
2095:   case SPIRV::OpTypeVector: {
2096:     SPIRVTypeInst ElemType = getScalarOrVectorComponentType(SpvType);
2097:     unsigned ElemOpcode = ElemType ? ElemType->getOpcode() : 0;
2098:     if (ElemOpcode == SPIRV::OpTypeFloat)
2099:       return &SPIRV::vfIDRegClass;
2100:     if (ElemOpcode == SPIRV::OpTypePointer)
2101:       return &SPIRV::vpIDRegClass;
2102:     return &SPIRV::viIDRegClass;
2103:   }
2104:   }
2105:   return &SPIRV::iIDRegClass;
2106: }
2107:
2108: inline unsigned getAS(SPIRVTypeInst SpvType) {
2109:   return storageClassToAddressSpace(
2110:       static_cast<SPIRV::StorageClass::StorageClass>(
2111:           SpvType->getOperand(1).getImm()));
2112: }
2113:
2114: LLT SPIRVGlobalRegistry::getRegType(SPIRVTypeInst SpvType) const {
2115:   unsigned Opcode = SpvType ? SpvType->getOpcode() : 0;
2116:   switch (Opcode) {
2117:   case SPIRV::OpTypeInt:
2118:   case SPIRV::OpTypeFloat:
2119:   case SPIRV::OpTypeBool:
2120:     return LLT::scalar(getScalarOrVectorBitWidth(SpvType));
2121:   case SPIRV::OpTypePointer:
2122:     return LLT::pointer(getAS(SpvType), getPointerSize());
2123:   case SPIRV::OpTypeVector: {
2124:     SPIRVTypeInst ElemType = getScalarOrVectorComponentType(SpvType);
2125:     LLT ET;
2126:     switch (ElemType ? ElemType->getOpcode() : 0) {
2127:     case SPIRV::OpTypePointer:
2128:       ET = LLT::pointer(getAS(ElemType), getPointerSize());
2129:       break;
2130:     case SPIRV::OpTypeInt:
2131:     case SPIRV::OpTypeFloat:
2132:     case SPIRV::OpTypeBool:
2133:       ET = LLT::scalar(getScalarOrVectorBitWidth(ElemType));
2134:       break;
2135:     default:
2136:       ET = LLT::scalar(64);
2137:     }
2138:     return LLT::fixed_vector(getScalarOrVectorComponentCount(SpvType), ET);
2139:   }
2140:   }
2141:   return LLT::scalar(64);
2142: }
2143:
2144: // Aliasing list MD contains several scope MD nodes whithin it. Each scope MD
2145: // has a selfreference and an extra MD node for aliasing domain and also it
2146: // can contain an optional string operand. Domain MD contains a self-reference
2147: // with an optional string operand. Here we unfold the list, creating SPIR-V
2148: // aliasing instructions.
2149: // TODO: add support for an optional string operand.
2150: MachineInstr *SPIRVGlobalRegistry::getOrAddMemAliasingINTELInst(
2151:     MachineIRBuilder &MIRBuilder, const MDNode *AliasingListMD) {
2152:   if (AliasingListMD->getNumOperands() == 0)
2153:     return nullptr;
2154:   if (auto L = AliasInstMDMap.find(AliasingListMD); L != AliasInstMDMap.end())
2155:     return L->second;
2156:
2157:   SmallVector<MachineInstr *> ScopeList;
2158:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2159:   for (const MDOperand &MDListOp : AliasingListMD->operands()) {
2160:     if (MDNode *ScopeMD = dyn_cast<MDNode>(MDListOp)) {
```
- EN: This range implements operational logic in helpers such as getDL, addDef, addUse, getSubtarget, translating backend policy into executable code.
- CN: 这一段实现了 getDL、addDef、addUse、getSubtarget 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2161-2250
```cpp
2161:       if (ScopeMD->getNumOperands() < 2)
2162:         return nullptr;
2163:       MDNode *DomainMD = dyn_cast<MDNode>(ScopeMD->getOperand(1));
2164:       if (!DomainMD)
2165:         return nullptr;
2166:       auto *Domain = [&] {
2167:         auto D = AliasInstMDMap.find(DomainMD);
2168:         if (D != AliasInstMDMap.end())
2169:           return D->second;
2170:         const Register Ret = MRI->createVirtualRegister(&SPIRV::IDRegClass);
2171:         auto MIB =
2172:             MIRBuilder.buildInstr(SPIRV::OpAliasDomainDeclINTEL).addDef(Ret);
2173:         return MIB.getInstr();
2174:       }();
2175:       AliasInstMDMap.insert(std::make_pair(DomainMD, Domain));
2176:       auto *Scope = [&] {
2177:         auto S = AliasInstMDMap.find(ScopeMD);
2178:         if (S != AliasInstMDMap.end())
2179:           return S->second;
2180:         const Register Ret = MRI->createVirtualRegister(&SPIRV::IDRegClass);
2181:         auto MIB = MIRBuilder.buildInstr(SPIRV::OpAliasScopeDeclINTEL)
2182:                        .addDef(Ret)
2183:                        .addUse(Domain->getOperand(0).getReg());
2184:         return MIB.getInstr();
2185:       }();
2186:       AliasInstMDMap.insert(std::make_pair(ScopeMD, Scope));
2187:       ScopeList.push_back(Scope);
2188:     }
2189:   }
2190:
2191:   const Register Ret = MRI->createVirtualRegister(&SPIRV::IDRegClass);
2192:   auto MIB =
2193:       MIRBuilder.buildInstr(SPIRV::OpAliasScopeListDeclINTEL).addDef(Ret);
2194:   for (auto *Scope : ScopeList)
2195:     MIB.addUse(Scope->getOperand(0).getReg());
2196:   auto List = MIB.getInstr();
2197:   AliasInstMDMap.insert(std::make_pair(AliasingListMD, List));
2198:   return List;
2199: }
2200:
2201: void SPIRVGlobalRegistry::buildMemAliasingOpDecorate(
2202:     Register Reg, MachineIRBuilder &MIRBuilder, uint32_t Dec,
2203:     const MDNode *AliasingListMD) {
2204:   MachineInstr *AliasList =
2205:       getOrAddMemAliasingINTELInst(MIRBuilder, AliasingListMD);
2206:   if (!AliasList)
2207:     return;
2208:   MIRBuilder.buildInstr(SPIRV::OpDecorateId)
2209:       .addUse(Reg)
2210:       .addImm(Dec)
2211:       .addUse(AliasList->getOperand(0).getReg());
2212: }
2213: void SPIRVGlobalRegistry::replaceAllUsesWith(Value *Old, Value *New,
2214:                                              bool DeleteOld) {
2215:   Old->replaceAllUsesWith(New);
2216:   updateIfExistDeducedElementType(Old, New, DeleteOld);
2217:   updateIfExistAssignPtrTypeInstr(Old, New, DeleteOld);
2218: }
2219:
2220: void SPIRVGlobalRegistry::buildAssignType(IRBuilder<> &B, Type *Ty,
2221:                                           Value *Arg) {
2222:   Value *OfType = getNormalizedPoisonValue(Ty);
2223:   CallInst *AssignCI = nullptr;
2224:   if (Arg->getType()->isAggregateType() && Ty->isAggregateType() &&
2225:       allowEmitFakeUse(Arg)) {
2226:     LLVMContext &Ctx = Arg->getContext();
2227:     SmallVector<Metadata *, 2> ArgMDs{
2228:         MDNode::get(Ctx, ValueAsMetadata::getConstant(OfType)),
2229:         MDString::get(Ctx, Arg->getName())};
2230:     B.CreateIntrinsic(Intrinsic::spv_value_md,
2231:                       {MetadataAsValue::get(Ctx, MDTuple::get(Ctx, ArgMDs))});
2232:     AssignCI = B.CreateIntrinsic(Intrinsic::fake_use, {Arg});
2233:   } else {
2234:     AssignCI = buildIntrWithMD(Intrinsic::spv_assign_type, {Arg->getType()},
2235:                                OfType, Arg, {}, B);
2236:   }
2237:   addAssignPtrTypeInstr(Arg, AssignCI);
2238: }
2239:
2240: void SPIRVGlobalRegistry::buildAssignPtr(IRBuilder<> &B, Type *ElemTy,
2241:                                          Value *Arg) {
2242:   Value *OfType = PoisonValue::get(ElemTy);
2243:   CallInst *AssignPtrTyCI = findAssignPtrTypeInstr(Arg);
2244:   Function *CurrF =
2245:       B.GetInsertBlock() ? B.GetInsertBlock()->getParent() : nullptr;
2246:   if (AssignPtrTyCI == nullptr ||
2247:       AssignPtrTyCI->getParent()->getParent() != CurrF) {
2248:     AssignPtrTyCI = buildIntrWithMD(
2249:         Intrinsic::spv_assign_ptr_type, {Arg->getType()}, OfType, Arg,
2250:         {B.getInt32(getPointerAddressSpace(Arg->getType()))}, B);
```
- EN: This range implements operational logic in helpers such as getOperand, find, createVirtualRegister, buildInstr, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、find、createVirtualRegister、buildInstr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2251-2299
```cpp
2251:     addDeducedElementType(AssignPtrTyCI, ElemTy);
2252:     addDeducedElementType(Arg, ElemTy);
2253:     addAssignPtrTypeInstr(Arg, AssignPtrTyCI);
2254:   } else {
2255:     updateAssignType(AssignPtrTyCI, Arg, OfType);
2256:   }
2257: }
2258:
2259: void SPIRVGlobalRegistry::updateAssignType(CallInst *AssignCI, Value *Arg,
2260:                                            Value *OfType) {
2261:   AssignCI->setArgOperand(1, buildMD(OfType));
2262:   if (cast<IntrinsicInst>(AssignCI)->getIntrinsicID() !=
2263:       Intrinsic::spv_assign_ptr_type)
2264:     return;
2265:
2266:   // update association with the pointee type
2267:   Type *ElemTy = OfType->getType();
2268:   addDeducedElementType(AssignCI, ElemTy);
2269:   addDeducedElementType(Arg, ElemTy);
2270: }
2271:
2272: void SPIRVGlobalRegistry::addStructOffsetDecorations(
2273:     Register Reg, StructType *Ty, MachineIRBuilder &MIRBuilder) {
2274:   ArrayRef<TypeSize> Offsets = DL.getStructLayout(Ty)->getMemberOffsets();
2275:   for (uint32_t I = 0; I < Ty->getNumElements(); ++I) {
2276:     buildOpMemberDecorate(Reg, MIRBuilder, SPIRV::Decoration::Offset, I,
2277:                           {static_cast<uint32_t>(Offsets[I])});
2278:   }
2279: }
2280:
2281: void SPIRVGlobalRegistry::addArrayStrideDecorations(
2282:     Register Reg, Type *ElementType, MachineIRBuilder &MIRBuilder) {
2283:   uint32_t SizeInBytes = DL.getTypeSizeInBits(ElementType) / 8;
2284:   buildOpDecorate(Reg, MIRBuilder, SPIRV::Decoration::ArrayStride,
2285:                   {SizeInBytes});
2286: }
2287:
2288: bool SPIRVGlobalRegistry::hasBlockDecoration(SPIRVTypeInst Type) const {
2289:   Register Def = getSPIRVTypeID(Type);
2290:   for (const MachineInstr &Use :
2291:        Type->getMF()->getRegInfo().use_instructions(Def)) {
2292:     if (Use.getOpcode() != SPIRV::OpDecorate)
2293:       continue;
2294:
2295:     if (Use.getOperand(1).getImm() == SPIRV::Decoration::Block)
2296:       return true;
2297:   }
2298:   return false;
2299: }
```
- EN: This range implements operational logic in helpers such as addDeducedElementType, addAssignPtrTypeInstr, updateAssignType, setArgOperand, translating backend policy into executable code.
- CN: 这一段实现了 addDeducedElementType、addAssignPtrTypeInstr、updateAssignType、setArgOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include allowEmitFakeUse, typeToAddressSpace, getAddressSpace, isTypedPointerWrapper, getIntParameter, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 allowEmitFakeUse, typeToAddressSpace, getAddressSpace, isTypedPointerWrapper, getIntParameter，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVGlobalRegistry.h`
  - `SPIRV.h`
  - `SPIRVBuiltins.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APInt.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/Type.h`
  - `llvm/Support/Casting.h`
  - `llvm/Support/MathExtras.h`
- System/standard headers / 系统或标准头文件:
  - `cassert`
  - `functional`
