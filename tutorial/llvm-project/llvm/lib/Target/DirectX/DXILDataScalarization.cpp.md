# DXILDataScalarization.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILDataScalarization.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILDataScalarization support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILDataScalarization.cpp - Perform DXIL Data Legalization ---------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===---------------------------------------------------------------------===//
 8:
 9: #include "DXILDataScalarization.h"
10: #include "DirectX.h"
11: #include "llvm/ADT/PostOrderIterator.h"
12: #include "llvm/ADT/STLExtras.h"
13: #include "llvm/IR/DerivedTypes.h"
14: #include "llvm/IR/GlobalVariable.h"
15: #include "llvm/IR/IRBuilder.h"
16: #include "llvm/IR/InstVisitor.h"
17: #include "llvm/IR/Instructions.h"
18: #include "llvm/IR/Module.h"
19: #include "llvm/IR/Operator.h"
20: #include "llvm/IR/PassManager.h"
21: #include "llvm/IR/ReplaceConstant.h"
22: #include "llvm/IR/Type.h"
23: #include "llvm/Support/Casting.h"
24: #include "llvm/Transforms/Utils/Cloning.h"
25: #include "llvm/Transforms/Utils/Local.h"
26:
27: #define DEBUG_TYPE "dxil-data-scalarization"
28: static const int MaxVecSize = 4;
29:
30: using namespace llvm;
31:
32: class DXILDataScalarizationLegacy : public ModulePass {
33:
34: public:
35:   bool runOnModule(Module &M) override;
36:   DXILDataScalarizationLegacy() : ModulePass(ID) {}
37:
38:   static char ID; // Pass identification.
39: };
40:
41: static bool findAndReplaceVectors(Module &M);
42:
43: class DataScalarizerVisitor : public InstVisitor<DataScalarizerVisitor, bool> {
44: public:
45:   DataScalarizerVisitor() : GlobalMap() {}
46:   bool visit(Function &F);
47:   // InstVisitor methods.  They return true if the instruction was scalarized,
48:   // false if nothing changed.
49:   bool visitAllocaInst(AllocaInst &AI);
50:   bool visitInstruction(Instruction &I) { return false; }
51:   bool visitSelectInst(SelectInst &SI) { return false; }
52:   bool visitICmpInst(ICmpInst &ICI) { return false; }
53:   bool visitFCmpInst(FCmpInst &FCI) { return false; }
54:   bool visitUnaryOperator(UnaryOperator &UO) { return false; }
55:   bool visitBinaryOperator(BinaryOperator &BO) { return false; }
56:   bool visitGetElementPtrInst(GetElementPtrInst &GEPI);
57:   bool visitCastInst(CastInst &CI) { return false; }
58:   bool visitBitCastInst(BitCastInst &BCI) { return false; }
59:   bool visitInsertElementInst(InsertElementInst &IEI);
60:   bool visitExtractElementInst(ExtractElementInst &EEI);
```
- EN: This range defines or declares important types such as DXILDataScalarizationLegacy, runOnModule, findAndReplaceVectors, DataScalarizerVisitor, shaping the data model used by DXILDataScalarization.cpp.
- CN: 这一段定义或声明了 DXILDataScalarizationLegacy、runOnModule、findAndReplaceVectors、DataScalarizerVisitor 等关键类型，构成 DXILDataScalarization.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:   bool visitShuffleVectorInst(ShuffleVectorInst &SVI) { return false; }
 62:   bool visitPHINode(PHINode &PHI) { return false; }
 63:   bool visitLoadInst(LoadInst &LI);
 64:   bool visitStoreInst(StoreInst &SI);
 65:   bool visitCallInst(CallInst &ICI) { return false; }
 66:   bool visitFreezeInst(FreezeInst &FI) { return false; }
 67:   friend bool findAndReplaceVectors(llvm::Module &M);
 68:
 69: private:
 70:   typedef std::tuple<AllocaInst *, Type *, SmallVector<Value *, 4>>
 71:       AllocaAndGEPs;
 72:   typedef SmallDenseMap<Value *, AllocaAndGEPs>
 73:       VectorToArrayMap; // A map from a vector-typed Value to its corresponding
 74:                         // AllocaInst and GEPs to each element of an array
 75:   VectorToArrayMap VectorAllocaMap;
 76:   AllocaAndGEPs createArrayFromVector(IRBuilder<> &Builder, Value *Vec,
 77:                                       const Twine &Name);
 78:   bool replaceDynamicInsertElementInst(InsertElementInst &IEI);
 79:   bool replaceDynamicExtractElementInst(ExtractElementInst &EEI);
 80:
 81:   GlobalVariable *lookupReplacementGlobal(Value *CurrOperand);
 82:   DenseMap<GlobalVariable *, GlobalVariable *> GlobalMap;
 83: };
 84:
 85: bool DataScalarizerVisitor::visit(Function &F) {
 86:   bool MadeChange = false;
 87:   ReversePostOrderTraversal<Function *> RPOT(&F);
 88:   for (BasicBlock *BB : make_early_inc_range(RPOT)) {
 89:     for (Instruction &I : make_early_inc_range(*BB))
 90:       MadeChange |= InstVisitor::visit(I);
 91:   }
 92:   VectorAllocaMap.clear();
 93:   return MadeChange;
 94: }
 95:
 96: GlobalVariable *
 97: DataScalarizerVisitor::lookupReplacementGlobal(Value *CurrOperand) {
 98:   if (GlobalVariable *OldGlobal = dyn_cast<GlobalVariable>(CurrOperand)) {
 99:     auto It = GlobalMap.find(OldGlobal);
100:     if (It != GlobalMap.end()) {
101:       return It->second; // Found, return the new global
102:     }
103:   }
104:   return nullptr; // Not found
105: }
106:
107: // Helper function to check if a type is a vector or an array of vectors
108: static bool isVectorOrArrayOfVectors(Type *T) {
109:   if (isa<VectorType>(T))
110:     return true;
111:   if (ArrayType *ArrayTy = dyn_cast<ArrayType>(T))
112:     return isVectorOrArrayOfVectors(ArrayTy->getElementType());
113:   return false;
114: }
115:
116: // Recursively creates an array-like version of a given vector type.
117: static Type *equivalentArrayTypeFromVector(Type *T) {
118:   if (auto *VecTy = dyn_cast<VectorType>(T))
119:     return ArrayType::get(VecTy->getElementType(),
120:                           dyn_cast<FixedVectorType>(VecTy)->getNumElements());
```
- EN: This range implements operational logic in helpers such as visitShuffleVectorInst, visitPHINode, visitLoadInst, visitStoreInst, translating backend policy into executable code.
- CN: 这一段实现了 visitShuffleVectorInst、visitPHINode、visitLoadInst、visitStoreInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   if (auto *ArrayTy = dyn_cast<ArrayType>(T)) {
122:     Type *NewElementType =
123:         equivalentArrayTypeFromVector(ArrayTy->getElementType());
124:     return ArrayType::get(NewElementType, ArrayTy->getNumElements());
125:   }
126:   // If it's not a vector or array, return the original type.
127:   return T;
128: }
129:
130: bool DataScalarizerVisitor::visitAllocaInst(AllocaInst &AI) {
131:   Type *AllocatedType = AI.getAllocatedType();
132:   if (!isVectorOrArrayOfVectors(AllocatedType))
133:     return false;
134:
135:   IRBuilder<> Builder(&AI);
136:   Type *NewType = equivalentArrayTypeFromVector(AllocatedType);
137:   AllocaInst *ArrAlloca =
138:       Builder.CreateAlloca(NewType, nullptr, AI.getName() + ".scalarized");
139:   ArrAlloca->setAlignment(AI.getAlign());
140:   AI.replaceAllUsesWith(ArrAlloca);
141:   AI.eraseFromParent();
142:   return true;
143: }
144:
145: bool DataScalarizerVisitor::visitLoadInst(LoadInst &LI) {
146:   Value *PtrOperand = LI.getPointerOperand();
147:   ConstantExpr *CE = dyn_cast<ConstantExpr>(PtrOperand);
148:   if (CE && CE->getOpcode() == Instruction::GetElementPtr) {
149:     GetElementPtrInst *OldGEP = cast<GetElementPtrInst>(CE->getAsInstruction());
150:     OldGEP->insertBefore(LI.getIterator());
151:     IRBuilder<> Builder(&LI);
152:     LoadInst *NewLoad = Builder.CreateLoad(LI.getType(), OldGEP, LI.getName());
153:     NewLoad->setAlignment(LI.getAlign());
154:     LI.replaceAllUsesWith(NewLoad);
155:     LI.eraseFromParent();
156:     visitGetElementPtrInst(*OldGEP);
157:     return true;
158:   }
159:   if (GlobalVariable *NewGlobal = lookupReplacementGlobal(PtrOperand))
160:     LI.setOperand(LI.getPointerOperandIndex(), NewGlobal);
161:   return false;
162: }
163:
164: bool DataScalarizerVisitor::visitStoreInst(StoreInst &SI) {
165:
166:   Value *PtrOperand = SI.getPointerOperand();
167:   ConstantExpr *CE = dyn_cast<ConstantExpr>(PtrOperand);
168:   if (CE && CE->getOpcode() == Instruction::GetElementPtr) {
169:     GetElementPtrInst *OldGEP = cast<GetElementPtrInst>(CE->getAsInstruction());
170:     OldGEP->insertBefore(SI.getIterator());
171:     IRBuilder<> Builder(&SI);
172:     StoreInst *NewStore = Builder.CreateStore(SI.getValueOperand(), OldGEP);
173:     NewStore->setAlignment(SI.getAlign());
174:     SI.replaceAllUsesWith(NewStore);
175:     SI.eraseFromParent();
176:     visitGetElementPtrInst(*OldGEP);
177:     return true;
178:   }
179:   if (GlobalVariable *NewGlobal = lookupReplacementGlobal(PtrOperand))
180:     SI.setOperand(SI.getPointerOperandIndex(), NewGlobal);
```
- EN: This range implements operational logic in helpers such as equivalentArrayTypeFromVector, ArrayType::get, DataScalarizerVisitor::visitAllocaInst, getAllocatedType, translating backend policy into executable code.
- CN: 这一段实现了 equivalentArrayTypeFromVector、ArrayType::get、DataScalarizerVisitor::visitAllocaInst、getAllocatedType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:
182:   return false;
183: }
184:
185: DataScalarizerVisitor::AllocaAndGEPs
186: DataScalarizerVisitor::createArrayFromVector(IRBuilder<> &Builder, Value *Vec,
187:                                              const Twine &Name = "") {
188:   // If there is already an alloca for this vector, return it
189:   if (VectorAllocaMap.contains(Vec))
190:     return VectorAllocaMap[Vec];
191:
192:   auto InsertPoint = Builder.GetInsertPoint();
193:
194:   // Allocate the array to hold the vector elements
195:   Builder.SetInsertPointPastAllocas(Builder.GetInsertBlock()->getParent());
196:   Type *ArrTy = equivalentArrayTypeFromVector(Vec->getType());
197:   AllocaInst *ArrAlloca =
198:       Builder.CreateAlloca(ArrTy, nullptr, Name + ".alloca");
199:   const uint64_t ArrNumElems = ArrTy->getArrayNumElements();
200:
201:   // Create loads and stores to populate the array immediately after the
202:   // original vector's defining instruction if available, else immediately after
203:   // the alloca
204:   if (auto *Instr = dyn_cast<Instruction>(Vec))
205:     Builder.SetInsertPoint(Instr->getNextNode());
206:   SmallVector<Value *, 4> GEPs(ArrNumElems);
207:   for (unsigned I = 0; I < ArrNumElems; ++I) {
208:     Value *EE = Builder.CreateExtractElement(Vec, I, Name + ".extract");
209:     GEPs[I] = Builder.CreateInBoundsGEP(
210:         ArrTy, ArrAlloca, {Builder.getInt32(0), Builder.getInt32(I)},
211:         Name + ".index");
212:     Builder.CreateStore(EE, GEPs[I]);
213:   }
214:
215:   VectorAllocaMap.insert({Vec, {ArrAlloca, ArrTy, GEPs}});
216:   Builder.SetInsertPoint(InsertPoint);
217:   return {ArrAlloca, ArrTy, GEPs};
218: }
219:
220: /// Returns a pair of Value* with the first being a GEP into ArrAlloca using
221: /// indices {0, Index}, and the second Value* being a Load of the GEP
222: static std::pair<Value *, Value *>
223: dynamicallyLoadArray(IRBuilder<> &Builder, AllocaInst *ArrAlloca, Type *ArrTy,
224:                      Value *Index, const Twine &Name = "") {
225:   Value *GEP = Builder.CreateInBoundsGEP(
226:       ArrTy, ArrAlloca, {Builder.getInt32(0), Index}, Name + ".index");
227:   Value *Load =
228:       Builder.CreateLoad(ArrTy->getArrayElementType(), GEP, Name + ".load");
229:   return std::make_pair(GEP, Load);
230: }
231:
232: bool DataScalarizerVisitor::replaceDynamicInsertElementInst(
233:     InsertElementInst &IEI) {
234:   IRBuilder<> Builder(&IEI);
235:
236:   Value *Vec = IEI.getOperand(0);
237:   Value *Val = IEI.getOperand(1);
238:   Value *Index = IEI.getOperand(2);
239:
240:   AllocaAndGEPs ArrAllocaAndGEPs =
```
- EN: This range implements operational logic in helpers such as GetInsertPoint, SetInsertPointPastAllocas, equivalentArrayTypeFromVector, CreateAlloca, translating backend policy into executable code.
- CN: 这一段实现了 GetInsertPoint、SetInsertPointPastAllocas、equivalentArrayTypeFromVector、CreateAlloca 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:       createArrayFromVector(Builder, Vec, IEI.getName());
242:   AllocaInst *ArrAlloca = std::get<0>(ArrAllocaAndGEPs);
243:   Type *ArrTy = std::get<1>(ArrAllocaAndGEPs);
244:   SmallVector<Value *, 4> &ArrGEPs = std::get<2>(ArrAllocaAndGEPs);
245:
246:   auto GEPAndLoad =
247:       dynamicallyLoadArray(Builder, ArrAlloca, ArrTy, Index, IEI.getName());
248:   Value *GEP = GEPAndLoad.first;
249:   Value *Load = GEPAndLoad.second;
250:
251:   Builder.CreateStore(Val, GEP);
252:   Value *NewIEI = PoisonValue::get(Vec->getType());
253:   for (unsigned I = 0; I < ArrTy->getArrayNumElements(); ++I) {
254:     Value *Load = Builder.CreateLoad(ArrTy->getArrayElementType(), ArrGEPs[I],
255:                                      IEI.getName() + ".load");
256:     NewIEI = Builder.CreateInsertElement(NewIEI, Load, Builder.getInt32(I),
257:                                          IEI.getName() + ".insert");
258:   }
259:
260:   // Store back the original value so the Alloca can be reused for subsequent
261:   // insertelement instructions on the same vector
262:   Builder.CreateStore(Load, GEP);
263:
264:   IEI.replaceAllUsesWith(NewIEI);
265:   IEI.eraseFromParent();
266:   return true;
267: }
268:
269: bool DataScalarizerVisitor::visitInsertElementInst(InsertElementInst &IEI) {
270:   // If the index is a constant then we don't need to scalarize it
271:   Value *Index = IEI.getOperand(2);
272:   if (isa<ConstantInt>(Index))
273:     return false;
274:   return replaceDynamicInsertElementInst(IEI);
275: }
276:
277: bool DataScalarizerVisitor::replaceDynamicExtractElementInst(
278:     ExtractElementInst &EEI) {
279:   IRBuilder<> Builder(&EEI);
280:
281:   AllocaAndGEPs ArrAllocaAndGEPs =
282:       createArrayFromVector(Builder, EEI.getVectorOperand(), EEI.getName());
283:   AllocaInst *ArrAlloca = std::get<0>(ArrAllocaAndGEPs);
284:   Type *ArrTy = std::get<1>(ArrAllocaAndGEPs);
285:
286:   auto GEPAndLoad = dynamicallyLoadArray(Builder, ArrAlloca, ArrTy,
287:                                          EEI.getIndexOperand(), EEI.getName());
288:   Value *Load = GEPAndLoad.second;
289:
290:   EEI.replaceAllUsesWith(Load);
291:   EEI.eraseFromParent();
292:   return true;
293: }
294:
295: bool DataScalarizerVisitor::visitExtractElementInst(ExtractElementInst &EEI) {
296:   // If the index is a constant then we don't need to scalarize it
297:   Value *Index = EEI.getIndexOperand();
298:   if (isa<ConstantInt>(Index))
299:     return false;
300:   return replaceDynamicExtractElementInst(EEI);
```
- EN: This range implements operational logic in helpers such as createArrayFromVector, dynamicallyLoadArray, CreateStore, PoisonValue::get, translating backend policy into executable code.
- CN: 这一段实现了 createArrayFromVector、dynamicallyLoadArray、CreateStore、PoisonValue::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301: }
302:
303: bool DataScalarizerVisitor::visitGetElementPtrInst(GetElementPtrInst &GEPI) {
304:   GEPOperator *GOp = cast<GEPOperator>(&GEPI);
305:   Value *PtrOperand = GOp->getPointerOperand();
306:   Type *GEPType = GOp->getSourceElementType();
307:
308:   // Replace a GEP ConstantExpr pointer operand with a GEP instruction so that
309:   // it can be visited
310:   if (auto *PtrOpGEPCE = dyn_cast<ConstantExpr>(PtrOperand);
311:       PtrOpGEPCE && PtrOpGEPCE->getOpcode() == Instruction::GetElementPtr) {
312:     GetElementPtrInst *OldGEPI =
313:         cast<GetElementPtrInst>(PtrOpGEPCE->getAsInstruction());
314:     OldGEPI->insertBefore(GEPI.getIterator());
315:
316:     IRBuilder<> Builder(&GEPI);
317:     SmallVector<Value *> Indices(GEPI.indices());
318:     Value *NewGEP =
319:         Builder.CreateGEP(GEPI.getSourceElementType(), OldGEPI, Indices,
320:                           GEPI.getName(), GEPI.getNoWrapFlags());
321:     assert(isa<GetElementPtrInst>(NewGEP) &&
322:            "Expected newly-created GEP to be an instruction");
323:     GetElementPtrInst *NewGEPI = cast<GetElementPtrInst>(NewGEP);
324:
325:     GEPI.replaceAllUsesWith(NewGEPI);
326:     GEPI.eraseFromParent();
327:     visitGetElementPtrInst(*OldGEPI);
328:     visitGetElementPtrInst(*NewGEPI);
329:     return true;
330:   }
331:
332:   Type *NewGEPType = equivalentArrayTypeFromVector(GEPType);
333:   Value *NewPtrOperand = PtrOperand;
334:   if (GlobalVariable *NewGlobal = lookupReplacementGlobal(PtrOperand))
335:     NewPtrOperand = NewGlobal;
336:
337:   bool NeedsTransform = NewPtrOperand != PtrOperand || NewGEPType != GEPType;
338:   if (!NeedsTransform)
339:     return false;
340:
341:   IRBuilder<> Builder(&GEPI);
342:   SmallVector<Value *, MaxVecSize> Indices(GOp->idx_begin(), GOp->idx_end());
343:   Value *NewGEP = Builder.CreateGEP(NewGEPType, NewPtrOperand, Indices,
344:                                     GOp->getName(), GOp->getNoWrapFlags());
345:
346:   GOp->replaceAllUsesWith(NewGEP);
347:
348:   if (auto *OldGEPI = dyn_cast<GetElementPtrInst>(GOp))
349:     OldGEPI->eraseFromParent();
350:
351:   return true;
352: }
353:
354: static Constant *transformInitializer(Constant *Init, Type *OrigType,
355:                                       Type *NewType, LLVMContext &Ctx) {
356:   // Handle ConstantAggregateZero (zero-initialized constants)
357:   if (isa<ConstantAggregateZero>(Init)) {
358:     return ConstantAggregateZero::get(NewType);
359:   }
360:
```
- EN: This range implements operational logic in helpers such as DataScalarizerVisitor::visitGetElementPtrInst, getPointerOperand, getSourceElementType, getOpcode, translating backend policy into executable code.
- CN: 这一段实现了 DataScalarizerVisitor::visitGetElementPtrInst、getPointerOperand、getSourceElementType、getOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   // Handle UndefValue (undefined constants)
362:   if (isa<UndefValue>(Init)) {
363:     return UndefValue::get(NewType);
364:   }
365:
366:   // Handle vector to array transformation
367:   if (isa<VectorType>(OrigType) && isa<ArrayType>(NewType)) {
368:     // Convert vector initializer to array initializer
369:     SmallVector<Constant *, MaxVecSize> ArrayElements;
370:
371:     unsigned E = cast<FixedVectorType>(OrigType)->getNumElements();
372:     for (unsigned I = 0; I != E; ++I)
373:       if (Constant *Elt = Init->getAggregateElement(I))
374:         ArrayElements.push_back(Elt);
375:
376:     assert(ArrayElements.size() == E &&
377:            "Expected fixed length constant aggregate for vector initializer!");
378:     return ConstantArray::get(cast<ArrayType>(NewType), ArrayElements);
379:   }
380:
381:   // Handle array of vectors transformation
382:   if (auto *ArrayTy = dyn_cast<ArrayType>(OrigType)) {
383:     auto *ArrayInit = dyn_cast<ConstantArray>(Init);
384:     assert(ArrayInit && "Expected a ConstantArray for array initializer!");
385:
386:     SmallVector<Constant *, MaxVecSize> NewArrayElements;
387:     for (unsigned I = 0; I < ArrayTy->getNumElements(); ++I) {
388:       // Recursively transform array elements
389:       Constant *NewElemInit = transformInitializer(
390:           ArrayInit->getOperand(I), ArrayTy->getElementType(),
391:           cast<ArrayType>(NewType)->getElementType(), Ctx);
392:       NewArrayElements.push_back(NewElemInit);
393:     }
394:
395:     return ConstantArray::get(cast<ArrayType>(NewType), NewArrayElements);
396:   }
397:
398:   // If not a vector or array, return the original initializer
399:   return Init;
400: }
401:
402: static bool findAndReplaceVectors(Module &M) {
403:   bool MadeChange = false;
404:   LLVMContext &Ctx = M.getContext();
405:   IRBuilder<> Builder(Ctx);
406:   DataScalarizerVisitor Impl;
407:   for (GlobalVariable &G : M.globals()) {
408:     Type *OrigType = G.getValueType();
409:
410:     Type *NewType = equivalentArrayTypeFromVector(OrigType);
411:     if (OrigType != NewType) {
412:       // Create a new global variable with the updated type
413:       // Note: Initializer is set via transformInitializer
414:       GlobalVariable *NewGlobal = new GlobalVariable(
415:           M, NewType, G.isConstant(), G.getLinkage(),
416:           /*Initializer=*/nullptr, G.getName() + ".scalarized", &G,
417:           G.getThreadLocalMode(), G.getAddressSpace(),
418:           G.isExternallyInitialized());
419:
420:       // Copy relevant attributes
```
- EN: This range implements operational logic in helpers such as UndefValue::get, getNumElements, push_back, ConstantArray::get, translating backend policy into executable code.
- CN: 这一段实现了 UndefValue::get、getNumElements、push_back、ConstantArray::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-474
```cpp
421:       NewGlobal->setUnnamedAddr(G.getUnnamedAddr());
422:       if (G.getAlignment() > 0) {
423:         NewGlobal->setAlignment(G.getAlign());
424:       }
425:
426:       if (G.hasInitializer()) {
427:         Constant *Init = G.getInitializer();
428:         Constant *NewInit = transformInitializer(Init, OrigType, NewType, Ctx);
429:         NewGlobal->setInitializer(NewInit);
430:       }
431:
432:       // Note: we want to do G.replaceAllUsesWith(NewGlobal);, but it assumes
433:       // type equality. Instead we will use the visitor pattern.
434:       Impl.GlobalMap[&G] = NewGlobal;
435:     }
436:   }
437:
438:   for (auto &F : make_early_inc_range(M.functions())) {
439:     if (F.isDeclaration())
440:       continue;
441:     MadeChange |= Impl.visit(F);
442:   }
443:
444:   // Remove the old globals after the iteration
445:   for (auto &[Old, New] : Impl.GlobalMap) {
446:     Old->eraseFromParent();
447:     MadeChange = true;
448:   }
449:   return MadeChange;
450: }
451:
452: PreservedAnalyses DXILDataScalarization::run(Module &M,
453:                                              ModuleAnalysisManager &) {
454:   bool MadeChanges = findAndReplaceVectors(M);
455:   if (!MadeChanges)
456:     return PreservedAnalyses::all();
457:   PreservedAnalyses PA;
458:   return PA;
459: }
460:
461: bool DXILDataScalarizationLegacy::runOnModule(Module &M) {
462:   return findAndReplaceVectors(M);
463: }
464:
465: char DXILDataScalarizationLegacy::ID = 0;
466:
467: INITIALIZE_PASS_BEGIN(DXILDataScalarizationLegacy, DEBUG_TYPE,
468:                       "DXIL Data Scalarization", false, false)
469: INITIALIZE_PASS_END(DXILDataScalarizationLegacy, DEBUG_TYPE,
470:                     "DXIL Data Scalarization", false, false)
471:
472: ModulePass *llvm::createDXILDataScalarizationLegacyPass() {
473:   return new DXILDataScalarizationLegacy();
474: }
```
- EN: This range implements operational logic in helpers such as setUnnamedAddr, setAlignment, getInitializer, transformInitializer, translating backend policy into executable code.
- CN: 这一段实现了 setUnnamedAddr、setAlignment、getInitializer、transformInitializer 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include DXILDataScalarizationLegacy, runOnModule, findAndReplaceVectors, DataScalarizerVisitor, visit, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DXILDataScalarizationLegacy, runOnModule, findAndReplaceVectors, DataScalarizerVisitor, visit，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILDataScalarization.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/PostOrderIterator.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstVisitor.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/Operator.h`
  - `llvm/IR/PassManager.h`
  - `llvm/IR/ReplaceConstant.h`
  - `llvm/IR/Type.h`
  - `llvm/Support/Casting.h`
  - `llvm/Transforms/Utils/Cloning.h`
  - `llvm/Transforms/Utils/Local.h`
