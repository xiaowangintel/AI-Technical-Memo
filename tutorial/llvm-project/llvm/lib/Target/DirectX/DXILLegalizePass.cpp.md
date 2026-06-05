# DXILLegalizePass.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILLegalizePass.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILLegalizePass support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILLegalizePass.cpp - Legalizes llvm IR for DXIL ------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===---------------------------------------------------------------------===//
 8:
 9: #include "DXILLegalizePass.h"
10: #include "DirectX.h"
11: #include "llvm/ADT/APInt.h"
12: #include "llvm/IR/Constants.h"
13: #include "llvm/IR/Function.h"
14: #include "llvm/IR/IRBuilder.h"
15: #include "llvm/IR/InstIterator.h"
16: #include "llvm/IR/Instruction.h"
17: #include "llvm/IR/Instructions.h"
18: #include "llvm/IR/Module.h"
19: #include "llvm/Pass.h"
20: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
21: #include <functional>
22:
23: #define DEBUG_TYPE "dxil-legalize"
24:
25: using namespace llvm;
26:
27: static bool legalizeFreeze(Instruction &I,
28:                            SmallVectorImpl<Instruction *> &ToRemove,
29:                            DenseMap<Value *, Value *>) {
30:   auto *FI = dyn_cast<FreezeInst>(&I);
31:   if (!FI)
32:     return false;
33:
34:   FI->replaceAllUsesWith(FI->getOperand(0));
35:   ToRemove.push_back(FI);
36:   return true;
37: }
38:
39: static bool fixI8UseChain(Instruction &I,
40:                           SmallVectorImpl<Instruction *> &ToRemove,
41:                           DenseMap<Value *, Value *> &ReplacedValues) {
42:
43:   auto ProcessOperands = [&](SmallVector<Value *> &NewOperands) {
44:     Type *InstrType = IntegerType::get(I.getContext(), 32);
45:
46:     for (unsigned OpIdx = 0; OpIdx < I.getNumOperands(); ++OpIdx) {
47:       Value *Op = I.getOperand(OpIdx);
48:       if (ReplacedValues.count(Op) &&
49:           ReplacedValues[Op]->getType()->isIntegerTy())
50:         InstrType = ReplacedValues[Op]->getType();
51:     }
52:
53:     for (unsigned OpIdx = 0; OpIdx < I.getNumOperands(); ++OpIdx) {
54:       Value *Op = I.getOperand(OpIdx);
55:       if (ReplacedValues.count(Op))
56:         NewOperands.push_back(ReplacedValues[Op]);
57:       else if (auto *Imm = dyn_cast<ConstantInt>(Op)) {
58:         APInt Value = Imm->getValue();
59:         unsigned NewBitWidth = InstrType->getIntegerBitWidth();
60:         // Note: options here are sext or sextOrTrunc.
```
- EN: This range implements operational logic in helpers such as replaceAllUsesWith, push_back, IntegerType::get, getNumOperands, translating backend policy into executable code.
- CN: 这一段实现了 replaceAllUsesWith、push_back、IntegerType::get、getNumOperands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:         // Since i8 isn't supported, we assume new values
 62:         // will always have a higher bitness.
 63:         assert(NewBitWidth > Value.getBitWidth() &&
 64:                "Replacement's BitWidth should be larger than Current.");
 65:         APInt NewValue = Value.sext(NewBitWidth);
 66:         NewOperands.push_back(ConstantInt::get(InstrType, NewValue));
 67:       } else {
 68:         assert(!Op->getType()->isIntegerTy(8));
 69:         NewOperands.push_back(Op);
 70:       }
 71:     }
 72:   };
 73:   IRBuilder<> Builder(&I);
 74:   if (auto *Trunc = dyn_cast<TruncInst>(&I)) {
 75:     if (Trunc->getDestTy()->isIntegerTy(8)) {
 76:       ReplacedValues[Trunc] = Trunc->getOperand(0);
 77:       ToRemove.push_back(Trunc);
 78:       return true;
 79:     }
 80:   }
 81:
 82:   if (auto *Store = dyn_cast<StoreInst>(&I)) {
 83:     if (!Store->getValueOperand()->getType()->isIntegerTy(8))
 84:       return false;
 85:     SmallVector<Value *> NewOperands;
 86:     ProcessOperands(NewOperands);
 87:     Value *NewStore = Builder.CreateStore(NewOperands[0], NewOperands[1]);
 88:     ReplacedValues[Store] = NewStore;
 89:     ToRemove.push_back(Store);
 90:     return true;
 91:   }
 92:
 93:   if (auto *Load = dyn_cast<LoadInst>(&I);
 94:       Load && I.getType()->isIntegerTy(8)) {
 95:     SmallVector<Value *> NewOperands;
 96:     ProcessOperands(NewOperands);
 97:     Type *ElementType = NewOperands[0]->getType();
 98:     if (auto *AI = dyn_cast<AllocaInst>(NewOperands[0]))
 99:       ElementType = AI->getAllocatedType();
100:     if (auto *GEP = dyn_cast<GetElementPtrInst>(NewOperands[0])) {
101:       ElementType = GEP->getSourceElementType();
102:     }
103:     if (ElementType->isArrayTy())
104:       ElementType = ElementType->getArrayElementType();
105:     LoadInst *NewLoad = Builder.CreateLoad(ElementType, NewOperands[0]);
106:     ReplacedValues[Load] = NewLoad;
107:     ToRemove.push_back(Load);
108:     return true;
109:   }
110:
111:   if (auto *Load = dyn_cast<LoadInst>(&I);
112:       Load && isa<ConstantExpr>(Load->getPointerOperand())) {
113:     auto *CE = dyn_cast<ConstantExpr>(Load->getPointerOperand());
114:     if (!(CE->getOpcode() == Instruction::GetElementPtr))
115:       return false;
116:     auto *GEP = dyn_cast<GEPOperator>(CE);
117:     if (!GEP->getSourceElementType()->isIntegerTy(8))
118:       return false;
119:
120:     Type *ElementType = Load->getType();
```
- EN: This range implements operational logic in helpers such as sext, push_back, assert, Builder, translating backend policy into executable code.
- CN: 这一段实现了 sext、push_back、assert、Builder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:     ConstantInt *Offset = dyn_cast<ConstantInt>(GEP->getOperand(1));
122:     uint32_t ByteOffset = Offset->getZExtValue();
123:     uint32_t ElemSize = Load->getDataLayout().getTypeAllocSize(ElementType);
124:     uint32_t Index = ByteOffset / ElemSize;
125:
126:     Value *PtrOperand = GEP->getPointerOperand();
127:     Type *GEPType = GEP->getPointerOperandType();
128:
129:     if (auto *GV = dyn_cast<GlobalVariable>(PtrOperand))
130:       GEPType = GV->getValueType();
131:     if (auto *AI = dyn_cast<AllocaInst>(PtrOperand))
132:       GEPType = AI->getAllocatedType();
133:
134:     if (auto *ArrTy = dyn_cast<ArrayType>(GEPType))
135:       GEPType = ArrTy;
136:     else
137:       GEPType = ArrayType::get(ElementType, 1); // its a scalar
138:
139:     Value *NewGEP = Builder.CreateGEP(
140:         GEPType, PtrOperand, {Builder.getInt32(0), Builder.getInt32(Index)},
141:         GEP->getName(), GEP->getNoWrapFlags());
142:
143:     LoadInst *NewLoad = Builder.CreateLoad(ElementType, NewGEP);
144:     ReplacedValues[Load] = NewLoad;
145:     Load->replaceAllUsesWith(NewLoad);
146:     ToRemove.push_back(Load);
147:     return true;
148:   }
149:
150:   if (auto *BO = dyn_cast<BinaryOperator>(&I)) {
151:     if (!I.getType()->isIntegerTy(8))
152:       return false;
153:     SmallVector<Value *> NewOperands;
154:     ProcessOperands(NewOperands);
155:     Value *NewInst =
156:         Builder.CreateBinOp(BO->getOpcode(), NewOperands[0], NewOperands[1]);
157:     if (auto *OBO = dyn_cast<OverflowingBinaryOperator>(&I)) {
158:       auto *NewBO = dyn_cast<BinaryOperator>(NewInst);
159:       if (NewBO && OBO->hasNoSignedWrap())
160:         NewBO->setHasNoSignedWrap();
161:       if (NewBO && OBO->hasNoUnsignedWrap())
162:         NewBO->setHasNoUnsignedWrap();
163:     }
164:     ReplacedValues[BO] = NewInst;
165:     ToRemove.push_back(BO);
166:     return true;
167:   }
168:
169:   if (auto *Sel = dyn_cast<SelectInst>(&I)) {
170:     if (!I.getType()->isIntegerTy(8))
171:       return false;
172:     SmallVector<Value *> NewOperands;
173:     ProcessOperands(NewOperands);
174:     Value *NewInst = Builder.CreateSelect(Sel->getCondition(), NewOperands[1],
175:                                           NewOperands[2]);
176:     ReplacedValues[Sel] = NewInst;
177:     ToRemove.push_back(Sel);
178:     return true;
179:   }
180:
```
- EN: This range implements operational logic in helpers such as getOperand, getZExtValue, getDataLayout, getPointerOperand, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getZExtValue、getDataLayout、getPointerOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   if (auto *Cmp = dyn_cast<CmpInst>(&I)) {
182:     if (!Cmp->getOperand(0)->getType()->isIntegerTy(8))
183:       return false;
184:     SmallVector<Value *> NewOperands;
185:     ProcessOperands(NewOperands);
186:     Value *NewInst =
187:         Builder.CreateCmp(Cmp->getPredicate(), NewOperands[0], NewOperands[1]);
188:     Cmp->replaceAllUsesWith(NewInst);
189:     ReplacedValues[Cmp] = NewInst;
190:     ToRemove.push_back(Cmp);
191:     return true;
192:   }
193:
194:   if (auto *Cast = dyn_cast<CastInst>(&I)) {
195:     if (!Cast->getSrcTy()->isIntegerTy(8))
196:       return false;
197:
198:     ToRemove.push_back(Cast);
199:     auto *Replacement = ReplacedValues[Cast->getOperand(0)];
200:     if (Cast->getType() == Replacement->getType()) {
201:       Cast->replaceAllUsesWith(Replacement);
202:       return true;
203:     }
204:
205:     Value *AdjustedCast = nullptr;
206:     if (Cast->getOpcode() == Instruction::ZExt)
207:       AdjustedCast = Builder.CreateZExtOrTrunc(Replacement, Cast->getType());
208:     if (Cast->getOpcode() == Instruction::SExt)
209:       AdjustedCast = Builder.CreateSExtOrTrunc(Replacement, Cast->getType());
210:
211:     if (AdjustedCast)
212:       Cast->replaceAllUsesWith(AdjustedCast);
213:   }
214:   if (auto *GEP = dyn_cast<GetElementPtrInst>(&I)) {
215:     if (!GEP->getType()->isPointerTy() ||
216:         !GEP->getSourceElementType()->isIntegerTy(8))
217:       return false;
218:
219:     Value *BasePtr = GEP->getPointerOperand();
220:     if (ReplacedValues.count(BasePtr))
221:       BasePtr = ReplacedValues[BasePtr];
222:
223:     Type *ElementType = BasePtr->getType();
224:
225:     if (auto *AI = dyn_cast<AllocaInst>(BasePtr))
226:       ElementType = AI->getAllocatedType();
227:     if (auto *GV = dyn_cast<GlobalVariable>(BasePtr))
228:       ElementType = GV->getValueType();
229:
230:     Type *GEPType = ElementType;
231:     if (auto *ArrTy = dyn_cast<ArrayType>(ElementType))
232:       ElementType = ArrTy->getArrayElementType();
233:     else
234:       GEPType = ArrayType::get(ElementType, 1); // its a scalar
235:
236:     ConstantInt *Offset = dyn_cast<ConstantInt>(GEP->getOperand(1));
237:     // Note: i8 to i32 offset conversion without emitting IR requires constant
238:     // ints. Since offset conversion is common, we can safely assume Offset is
239:     // always a ConstantInt, so no need to have a conditional bail out on
240:     // nullptr, instead assert this is the case.
```
- EN: This range implements operational logic in helpers such as ProcessOperands, CreateCmp, replaceAllUsesWith, push_back, translating backend policy into executable code.
- CN: 这一段实现了 ProcessOperands、CreateCmp、replaceAllUsesWith、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:     assert(Offset && "Offset is expected to be a ConstantInt");
242:     uint32_t ByteOffset = Offset->getZExtValue();
243:     uint32_t ElemSize = GEP->getDataLayout().getTypeAllocSize(ElementType);
244:     assert(ElemSize > 0 && "ElementSize must be set");
245:     uint32_t Index = ByteOffset / ElemSize;
246:     Value *NewGEP = Builder.CreateGEP(
247:         GEPType, BasePtr, {Builder.getInt32(0), Builder.getInt32(Index)},
248:         GEP->getName(), GEP->getNoWrapFlags());
249:     ReplacedValues[GEP] = NewGEP;
250:     GEP->replaceAllUsesWith(NewGEP);
251:     ToRemove.push_back(GEP);
252:     return true;
253:   }
254:   return false;
255: }
256:
257: static bool upcastI8AllocasAndUses(Instruction &I,
258:                                    SmallVectorImpl<Instruction *> &ToRemove,
259:                                    DenseMap<Value *, Value *> &ReplacedValues) {
260:   auto *AI = dyn_cast<AllocaInst>(&I);
261:   if (!AI || !AI->getAllocatedType()->isIntegerTy(8))
262:     return false;
263:
264:   Type *SmallestType = nullptr;
265:
266:   auto ProcessLoad = [&](LoadInst *Load) {
267:     for (User *LU : Load->users()) {
268:       CastInst *Cast = dyn_cast<CastInst>(LU);
269:       if (!Cast)
270:         continue;
271:       Type *Ty = Cast->getType();
272:
273:       if (!SmallestType ||
274:           Ty->getPrimitiveSizeInBits() < SmallestType->getPrimitiveSizeInBits())
275:         SmallestType = Ty;
276:     }
277:   };
278:
279:   for (User *U : AI->users()) {
280:     if (auto *Load = dyn_cast<LoadInst>(U))
281:       ProcessLoad(Load);
282:     else if (auto *GEP = dyn_cast<GetElementPtrInst>(U)) {
283:       for (User *GU : GEP->users()) {
284:         if (auto *Load = dyn_cast<LoadInst>(GU))
285:           ProcessLoad(Load);
286:       }
287:     }
288:   }
289:
290:   if (!SmallestType)
291:     return false; // no valid casts found
292:
293:   // Replace alloca
294:   IRBuilder<> Builder(AI);
295:   auto *NewAlloca = Builder.CreateAlloca(SmallestType);
296:   ReplacedValues[AI] = NewAlloca;
297:   ToRemove.push_back(AI);
298:   return true;
299: }
300:
```
- EN: This range implements operational logic in helpers such as assert, getZExtValue, getDataLayout, getName, translating backend policy into executable code.
- CN: 这一段实现了 assert、getZExtValue、getDataLayout、getName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301: static bool
302: downcastI64toI32InsertExtractElements(Instruction &I,
303:                                       SmallVectorImpl<Instruction *> &ToRemove,
304:                                       DenseMap<Value *, Value *> &) {
305:
306:   if (auto *Extract = dyn_cast<ExtractElementInst>(&I)) {
307:     Value *Idx = Extract->getIndexOperand();
308:     auto *CI = dyn_cast<ConstantInt>(Idx);
309:     if (CI && CI->getBitWidth() == 64) {
310:       IRBuilder<> Builder(Extract);
311:       int64_t IndexValue = CI->getSExtValue();
312:       auto *Idx32 =
313:           ConstantInt::get(Type::getInt32Ty(I.getContext()), IndexValue);
314:       Value *NewExtract = Builder.CreateExtractElement(
315:           Extract->getVectorOperand(), Idx32, Extract->getName());
316:
317:       Extract->replaceAllUsesWith(NewExtract);
318:       ToRemove.push_back(Extract);
319:       return true;
320:     }
321:   }
322:
323:   if (auto *Insert = dyn_cast<InsertElementInst>(&I)) {
324:     Value *Idx = Insert->getOperand(2);
325:     auto *CI = dyn_cast<ConstantInt>(Idx);
326:     if (CI && CI->getBitWidth() == 64) {
327:       int64_t IndexValue = CI->getSExtValue();
328:       auto *Idx32 =
329:           ConstantInt::get(Type::getInt32Ty(I.getContext()), IndexValue);
330:       IRBuilder<> Builder(Insert);
331:       Value *Insert32Index = Builder.CreateInsertElement(
332:           Insert->getOperand(0), Insert->getOperand(1), Idx32,
333:           Insert->getName());
334:
335:       Insert->replaceAllUsesWith(Insert32Index);
336:       ToRemove.push_back(Insert);
337:       return true;
338:     }
339:   }
340:   return false;
341: }
342:
343: static bool updateFnegToFsub(Instruction &I,
344:                              SmallVectorImpl<Instruction *> &ToRemove,
345:                              DenseMap<Value *, Value *> &) {
346:   const Intrinsic::ID ID = I.getOpcode();
347:   if (ID != Instruction::FNeg)
348:     return false;
349:
350:   IRBuilder<> Builder(&I);
351:   Value *In = I.getOperand(0);
352:   Value *Zero = ConstantFP::get(In->getType(), -0.0);
353:   I.replaceAllUsesWith(Builder.CreateFSub(Zero, In));
354:   ToRemove.push_back(&I);
355:   return true;
356: }
357:
358: static bool
359: legalizeGetHighLowi64Bytes(Instruction &I,
360:                            SmallVectorImpl<Instruction *> &ToRemove,
```
- EN: This range implements operational logic in helpers such as getIndexOperand, Builder, getSExtValue, ConstantInt::get, translating backend policy into executable code.
- CN: 这一段实现了 getIndexOperand、Builder、getSExtValue、ConstantInt::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:                            DenseMap<Value *, Value *> &ReplacedValues) {
362:   if (auto *BitCast = dyn_cast<BitCastInst>(&I)) {
363:     if (BitCast->getDestTy() ==
364:             FixedVectorType::get(Type::getInt32Ty(I.getContext()), 2) &&
365:         BitCast->getSrcTy()->isIntegerTy(64)) {
366:       ToRemove.push_back(BitCast);
367:       ReplacedValues[BitCast] = BitCast->getOperand(0);
368:       return true;
369:     }
370:   }
371:
372:   if (auto *Extract = dyn_cast<ExtractElementInst>(&I)) {
373:     if (!dyn_cast<BitCastInst>(Extract->getVectorOperand()))
374:       return false;
375:     auto *VecTy = dyn_cast<FixedVectorType>(Extract->getVectorOperandType());
376:     if (VecTy && VecTy->getElementType()->isIntegerTy(32) &&
377:         VecTy->getNumElements() == 2) {
378:       if (auto *Index = dyn_cast<ConstantInt>(Extract->getIndexOperand())) {
379:         unsigned Idx = Index->getZExtValue();
380:         IRBuilder<> Builder(&I);
381:
382:         auto *Replacement = ReplacedValues[Extract->getVectorOperand()];
383:         assert(Replacement && "The BitCast replacement should have been set "
384:                               "before working on ExtractElementInst.");
385:         if (Idx == 0) {
386:           Value *LowBytes = Builder.CreateTrunc(
387:               Replacement, Type::getInt32Ty(I.getContext()));
388:           ReplacedValues[Extract] = LowBytes;
389:         } else {
390:           assert(Idx == 1);
391:           Value *LogicalShiftRight = Builder.CreateLShr(
392:               Replacement,
393:               ConstantInt::get(
394:                   Replacement->getType(),
395:                   APInt(Replacement->getType()->getIntegerBitWidth(), 32)));
396:           Value *HighBytes = Builder.CreateTrunc(
397:               LogicalShiftRight, Type::getInt32Ty(I.getContext()));
398:           ReplacedValues[Extract] = HighBytes;
399:         }
400:         ToRemove.push_back(Extract);
401:         Extract->replaceAllUsesWith(ReplacedValues[Extract]);
402:         return true;
403:       }
404:     }
405:   }
406:   return false;
407: }
408:
409: static bool
410: resolveUnreachableSwitchDefault(Instruction &I,
411:                                 SmallVectorImpl<Instruction *> &ToRemove,
412:                                 DenseMap<Value *, Value *> &) {
413:   auto *SI = dyn_cast<SwitchInst>(&I);
414:   if (!SI || SI->getNumCases() == 0)
415:     return false;
416:
417:   BasicBlock *DefaultBB = SI->getDefaultDest();
418:
419:   // Check if the default destination ends with an unreachable instruction.
420:   if (DefaultBB->size() == 0 ||
```
- EN: This range implements operational logic in helpers such as getSrcTy, push_back, getOperand, getVectorOperandType, translating backend policy into executable code.
- CN: 这一段实现了 getSrcTy、push_back、getOperand、getVectorOperandType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:       !isa<UnreachableInst>(DefaultBB->getTerminator()))
422:     return false;
423:
424:   // Try to find a common successor of all case destinations. If all case
425:   // blocks unconditionally branch to the same block, that is the common
426:   // successor. This is just a best effort, and is done as the original form of
427:   // the switch statement was likely in this form before being transformed to
428:   // an unreachable branch.
429:   BasicBlock *CommonSuccessor = nullptr;
430:   for (auto &Case : SI->cases()) {
431:     BasicBlock *CaseBB = Case.getCaseSuccessor();
432:     auto *BI = dyn_cast<UncondBrInst>(CaseBB->getTerminator());
433:     if (!BI) {
434:       CommonSuccessor = nullptr;
435:       break;
436:     }
437:     BasicBlock *Succ = BI->getSuccessor(0);
438:     if (!CommonSuccessor)
439:       CommonSuccessor = Succ;
440:     else if (CommonSuccessor != Succ) {
441:       CommonSuccessor = nullptr;
442:       break;
443:     }
444:   }
445:
446:   BasicBlock *NewDefault =
447:       CommonSuccessor ? CommonSuccessor : SI->case_begin()->getCaseSuccessor();
448:
449:   BasicBlock *SwitchBB = SI->getParent();
450:   SI->setDefaultDest(NewDefault);
451:
452:   // Ensure all phi nodes are legal by adding an incoming poison value from the
453:   // unreachable branch.
454:   for (PHINode &Phi : NewDefault->phis())
455:     Phi.addIncoming(PoisonValue::get(Phi.getType()), SwitchBB);
456:
457:   return true;
458: }
459:
460: static bool
461: legalizeScalarLoadStoreOnArrays(Instruction &I,
462:                                 SmallVectorImpl<Instruction *> &ToRemove,
463:                                 DenseMap<Value *, Value *> &) {
464:
465:   Value *PtrOp;
466:   unsigned PtrOpIndex;
467:   [[maybe_unused]] Type *LoadStoreTy;
468:   if (auto *LI = dyn_cast<LoadInst>(&I)) {
469:     PtrOp = LI->getPointerOperand();
470:     PtrOpIndex = LI->getPointerOperandIndex();
471:     LoadStoreTy = LI->getType();
472:   } else if (auto *SI = dyn_cast<StoreInst>(&I)) {
473:     PtrOp = SI->getPointerOperand();
474:     PtrOpIndex = SI->getPointerOperandIndex();
475:     LoadStoreTy = SI->getValueOperand()->getType();
476:   } else
477:     return false;
478:
479:   // If the load/store is not of a single-value type (i.e., scalar or vector)
480:   // then we do not modify it. It shouldn't be a vector either because the
```
- EN: This range implements operational logic in helpers such as getTerminator, getCaseSuccessor, getSuccessor, case_begin, translating backend policy into executable code.
- CN: 这一段实现了 getTerminator、getCaseSuccessor、getSuccessor、case_begin 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:   // dxil-data-scalarization pass is expected to run before this, but it's not
482:   // incorrect to apply this transformation to vector load/stores.
483:   if (!LoadStoreTy->isSingleValueType())
484:     return false;
485:
486:   Type *ArrayTy;
487:   if (auto *GlobalVarPtrOp = dyn_cast<GlobalVariable>(PtrOp))
488:     ArrayTy = GlobalVarPtrOp->getValueType();
489:   else if (auto *AllocaPtrOp = dyn_cast<AllocaInst>(PtrOp))
490:     ArrayTy = AllocaPtrOp->getAllocatedType();
491:   else
492:     return false;
493:
494:   if (!isa<ArrayType>(ArrayTy))
495:     return false;
496:
497:   assert(ArrayTy->getArrayElementType() == LoadStoreTy &&
498:          "Expected array element type to be the same as to the scalar load or "
499:          "store type");
500:
501:   Value *Zero = ConstantInt::get(Type::getInt32Ty(I.getContext()), 0);
502:   Value *GEP = GetElementPtrInst::Create(
503:       ArrayTy, PtrOp, {Zero, Zero}, GEPNoWrapFlags::all(), "", I.getIterator());
504:   I.setOperand(PtrOpIndex, GEP);
505:   return true;
506: }
507:
508: namespace {
509: class DXILLegalizationPipeline {
510:
511: public:
512:   DXILLegalizationPipeline() { initializeLegalizationPipeline(); }
513:
514:   bool runLegalizationPipeline(Function &F) {
515:     bool MadeChange = false;
516:     SmallVector<Instruction *> ToRemove;
517:     DenseMap<Value *, Value *> ReplacedValues;
518:     for (int Stage = 0; Stage < NumStages; ++Stage) {
519:       ToRemove.clear();
520:       ReplacedValues.clear();
521:       for (auto &I : instructions(F)) {
522:         for (auto &LegalizationFn : LegalizationPipeline[Stage])
523:           MadeChange |= LegalizationFn(I, ToRemove, ReplacedValues);
524:       }
525:
526:       for (auto *Inst : reverse(ToRemove))
527:         Inst->eraseFromParent();
528:     }
529:     return MadeChange;
530:   }
531:
532: private:
533:   enum LegalizationStage { Stage1 = 0, Stage2 = 1, NumStages };
534:
535:   using LegalizationFnTy =
536:       std::function<bool(Instruction &, SmallVectorImpl<Instruction *> &,
537:                          DenseMap<Value *, Value *> &)>;
538:
539:   SmallVector<LegalizationFnTy> LegalizationPipeline[NumStages];
540:
```
- EN: This range defines or declares important types such as getValueType, getAllocatedType, ConstantInt::get, GEPNoWrapFlags::all, shaping the data model used by DXILLegalizePass.cpp.
- CN: 这一段定义或声明了 getValueType、getAllocatedType、ConstantInt::get、GEPNoWrapFlags::all 等关键类型，构成 DXILLegalizePass.cpp 使用的数据模型。

### Lines 541-593
```cpp
541:   void initializeLegalizationPipeline() {
542:     LegalizationPipeline[Stage1].push_back(upcastI8AllocasAndUses);
543:     LegalizationPipeline[Stage1].push_back(fixI8UseChain);
544:     LegalizationPipeline[Stage1].push_back(legalizeGetHighLowi64Bytes);
545:     LegalizationPipeline[Stage1].push_back(legalizeFreeze);
546:     LegalizationPipeline[Stage1].push_back(updateFnegToFsub);
547:     // Note: legalizeGetHighLowi64Bytes and
548:     // downcastI64toI32InsertExtractElements both modify extractelement, so they
549:     // must run staggered stages. legalizeGetHighLowi64Bytes runs first b\c it
550:     // removes extractelements, reducing the number that
551:     // downcastI64toI32InsertExtractElements needs to handle.
552:     LegalizationPipeline[Stage2].push_back(
553:         downcastI64toI32InsertExtractElements);
554:     LegalizationPipeline[Stage2].push_back(legalizeScalarLoadStoreOnArrays);
555:     LegalizationPipeline[Stage2].push_back(resolveUnreachableSwitchDefault);
556:   }
557: };
558:
559: class DXILLegalizeLegacy : public FunctionPass {
560:
561: public:
562:   bool runOnFunction(Function &F) override;
563:   DXILLegalizeLegacy() : FunctionPass(ID) {}
564:
565:   static char ID; // Pass identification.
566: };
567: } // namespace
568:
569: PreservedAnalyses DXILLegalizePass::run(Function &F,
570:                                         FunctionAnalysisManager &FAM) {
571:   DXILLegalizationPipeline DXLegalize;
572:   bool MadeChanges = DXLegalize.runLegalizationPipeline(F);
573:   if (!MadeChanges)
574:     return PreservedAnalyses::all();
575:   PreservedAnalyses PA;
576:   return PA;
577: }
578:
579: bool DXILLegalizeLegacy::runOnFunction(Function &F) {
580:   DXILLegalizationPipeline DXLegalize;
581:   return DXLegalize.runLegalizationPipeline(F);
582: }
583:
584: char DXILLegalizeLegacy::ID = 0;
585:
586: INITIALIZE_PASS_BEGIN(DXILLegalizeLegacy, DEBUG_TYPE, "DXIL Legalizer", false,
587:                       false)
588: INITIALIZE_PASS_END(DXILLegalizeLegacy, DEBUG_TYPE, "DXIL Legalizer", false,
589:                     false)
590:
591: FunctionPass *llvm::createDXILLegalizeLegacyPass() {
592:   return new DXILLegalizeLegacy();
593: }
```
- EN: This range defines or declares important types such as initializeLegalizationPipeline, push_back, DXILLegalizeLegacy, runOnFunction, shaping the data model used by DXILLegalizePass.cpp.
- CN: 这一段定义或声明了 initializeLegalizationPipeline、push_back、DXILLegalizeLegacy、runOnFunction 等关键类型，构成 DXILLegalizePass.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include replaceAllUsesWith, push_back, IntegerType::get, getNumOperands, getOperand, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 replaceAllUsesWith, push_back, IntegerType::get, getNumOperands, getOperand，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILLegalizePass.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APInt.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Module.h`
  - `llvm/Pass.h`
  - `llvm/Transforms/Utils/BasicBlockUtils.h`
- System/standard headers / 系统或标准头文件:
  - `functional`
