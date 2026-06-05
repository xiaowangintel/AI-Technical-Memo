# DXILFlattenArrays.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILFlattenArrays.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a pass to flatten arrays for the DirectX Backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILFlattenArrays.cpp - Flattens DXIL Arrays-----------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===---------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains a pass to flatten arrays for the DirectX Backend.
10: ///
11: //===----------------------------------------------------------------------===//
12:
13: #include "DXILFlattenArrays.h"
14: #include "DirectX.h"
15: #include "llvm/ADT/PostOrderIterator.h"
16: #include "llvm/ADT/STLExtras.h"
17: #include "llvm/IR/BasicBlock.h"
18: #include "llvm/IR/DerivedTypes.h"
19: #include "llvm/IR/IRBuilder.h"
20: #include "llvm/IR/InstVisitor.h"
21: #include "llvm/IR/ReplaceConstant.h"
22: #include "llvm/Support/Casting.h"
23: #include "llvm/Support/MathExtras.h"
24: #include "llvm/Transforms/Utils/Local.h"
25: #include <cassert>
26: #include <cstddef>
27: #include <cstdint>
28: #include <utility>
29:
30: #define DEBUG_TYPE "dxil-flatten-arrays"
31:
32: using namespace llvm;
33: namespace {
34:
35: class DXILFlattenArraysLegacy : public ModulePass {
36:
37: public:
38:   bool runOnModule(Module &M) override;
39:   DXILFlattenArraysLegacy() : ModulePass(ID) {}
40:
41:   static char ID; // Pass identification.
42: };
43:
44: struct GEPInfo {
45:   ArrayType *RootFlattenedArrayType;
46:   Value *RootPointerOperand;
47:   SmallMapVector<Value *, APInt, 4> VariableOffsets;
48:   APInt ConstantOffset;
49: };
50:
51: class DXILFlattenArraysVisitor
52:     : public InstVisitor<DXILFlattenArraysVisitor, bool> {
53: public:
54:   DXILFlattenArraysVisitor(
55:       SmallDenseMap<GlobalVariable *, GlobalVariable *> &GlobalMap)
56:       : GlobalMap(GlobalMap) {}
57:   bool visit(Function &F);
58:   // InstVisitor methods.  They return true if the instruction was scalarized,
59:   // false if nothing changed.
60:   bool visitGetElementPtrInst(GetElementPtrInst &GEPI);
```
- EN: This range defines or declares important types such as DXILFlattenArraysLegacy, runOnModule, GEPInfo, DXILFlattenArraysVisitor, shaping the data model used by DXILFlattenArrays.cpp.
- CN: 这一段定义或声明了 DXILFlattenArraysLegacy、runOnModule、GEPInfo、DXILFlattenArraysVisitor 等关键类型，构成 DXILFlattenArrays.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:   bool visitAllocaInst(AllocaInst &AI);
 62:   bool visitInstruction(Instruction &I) { return false; }
 63:   bool visitSelectInst(SelectInst &SI) { return false; }
 64:   bool visitICmpInst(ICmpInst &ICI) { return false; }
 65:   bool visitFCmpInst(FCmpInst &FCI) { return false; }
 66:   bool visitUnaryOperator(UnaryOperator &UO) { return false; }
 67:   bool visitBinaryOperator(BinaryOperator &BO) { return false; }
 68:   bool visitCastInst(CastInst &CI) { return false; }
 69:   bool visitBitCastInst(BitCastInst &BCI) { return false; }
 70:   bool visitInsertElementInst(InsertElementInst &IEI) { return false; }
 71:   bool visitExtractElementInst(ExtractElementInst &EEI) { return false; }
 72:   bool visitShuffleVectorInst(ShuffleVectorInst &SVI) { return false; }
 73:   bool visitPHINode(PHINode &PHI) { return false; }
 74:   bool visitLoadInst(LoadInst &LI);
 75:   bool visitStoreInst(StoreInst &SI);
 76:   bool visitCallInst(CallInst &ICI) { return false; }
 77:   bool visitFreezeInst(FreezeInst &FI) { return false; }
 78:   static bool isMultiDimensionalArray(Type *T);
 79:   static std::pair<unsigned, Type *> getElementCountAndType(Type *ArrayTy);
 80:
 81: private:
 82:   SmallVector<WeakTrackingVH> PotentiallyDeadInstrs;
 83:   SmallDenseMap<GEPOperator *, GEPInfo> GEPChainInfoMap;
 84:   SmallDenseMap<GlobalVariable *, GlobalVariable *> &GlobalMap;
 85:   bool finish();
 86:   ConstantInt *genConstFlattenIndices(ArrayRef<Value *> Indices,
 87:                                       ArrayRef<uint64_t> Dims,
 88:                                       IRBuilder<> &Builder);
 89:   Value *genInstructionFlattenIndices(ArrayRef<Value *> Indices,
 90:                                       ArrayRef<uint64_t> Dims,
 91:                                       IRBuilder<> &Builder);
 92: };
 93: } // namespace
 94:
 95: bool DXILFlattenArraysVisitor::finish() {
 96:   GEPChainInfoMap.clear();
 97:   RecursivelyDeleteTriviallyDeadInstructionsPermissive(PotentiallyDeadInstrs);
 98:   return true;
 99: }
100:
101: bool DXILFlattenArraysVisitor::isMultiDimensionalArray(Type *T) {
102:   if (ArrayType *ArrType = dyn_cast<ArrayType>(T))
103:     return isa<ArrayType>(ArrType->getElementType());
104:   return false;
105: }
106:
107: std::pair<unsigned, Type *>
108: DXILFlattenArraysVisitor::getElementCountAndType(Type *ArrayTy) {
109:   unsigned TotalElements = 1;
110:   Type *CurrArrayTy = ArrayTy;
111:   while (auto *InnerArrayTy = dyn_cast<ArrayType>(CurrArrayTy)) {
112:     TotalElements *= InnerArrayTy->getNumElements();
113:     CurrArrayTy = InnerArrayTy->getElementType();
114:   }
115:   return std::make_pair(TotalElements, CurrArrayTy);
116: }
117:
118: ConstantInt *DXILFlattenArraysVisitor::genConstFlattenIndices(
119:     ArrayRef<Value *> Indices, ArrayRef<uint64_t> Dims, IRBuilder<> &Builder) {
120:   assert(Indices.size() == Dims.size() &&
```
- EN: This range implements operational logic in helpers such as visitAllocaInst, visitInstruction, visitSelectInst, visitICmpInst, translating backend policy into executable code.
- CN: 这一段实现了 visitAllocaInst、visitInstruction、visitSelectInst、visitICmpInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:          "Indicies and dimmensions should be the same");
122:   unsigned FlatIndex = 0;
123:   unsigned Multiplier = 1;
124:
125:   for (int I = Indices.size() - 1; I >= 0; --I) {
126:     unsigned DimSize = Dims[I];
127:     ConstantInt *CIndex = dyn_cast<ConstantInt>(Indices[I]);
128:     assert(CIndex && "This function expects all indicies to be ConstantInt");
129:     FlatIndex += CIndex->getZExtValue() * Multiplier;
130:     Multiplier *= DimSize;
131:   }
132:   return Builder.getInt32(FlatIndex);
133: }
134:
135: Value *DXILFlattenArraysVisitor::genInstructionFlattenIndices(
136:     ArrayRef<Value *> Indices, ArrayRef<uint64_t> Dims, IRBuilder<> &Builder) {
137:   if (Indices.size() == 1)
138:     return Indices[0];
139:
140:   Value *FlatIndex = Builder.getInt32(0);
141:   unsigned Multiplier = 1;
142:
143:   for (int I = Indices.size() - 1; I >= 0; --I) {
144:     unsigned DimSize = Dims[I];
145:     Value *VMultiplier = Builder.getInt32(Multiplier);
146:     Value *ScaledIndex = Builder.CreateMul(Indices[I], VMultiplier);
147:     FlatIndex = Builder.CreateAdd(FlatIndex, ScaledIndex);
148:     Multiplier *= DimSize;
149:   }
150:   return FlatIndex;
151: }
152:
153: bool DXILFlattenArraysVisitor::visitLoadInst(LoadInst &LI) {
154:   unsigned NumOperands = LI.getNumOperands();
155:   for (unsigned I = 0; I < NumOperands; ++I) {
156:     Value *CurrOpperand = LI.getOperand(I);
157:     ConstantExpr *CE = dyn_cast<ConstantExpr>(CurrOpperand);
158:     if (CE && CE->getOpcode() == Instruction::GetElementPtr) {
159:       GetElementPtrInst *OldGEP =
160:           cast<GetElementPtrInst>(CE->getAsInstruction());
161:       OldGEP->insertBefore(LI.getIterator());
162:
163:       IRBuilder<> Builder(&LI);
164:       LoadInst *NewLoad =
165:           Builder.CreateLoad(LI.getType(), OldGEP, LI.getName());
166:       NewLoad->setAlignment(LI.getAlign());
167:       LI.replaceAllUsesWith(NewLoad);
168:       LI.eraseFromParent();
169:       visitGetElementPtrInst(*OldGEP);
170:       return true;
171:     }
172:   }
173:   return false;
174: }
175:
176: bool DXILFlattenArraysVisitor::visitStoreInst(StoreInst &SI) {
177:   unsigned NumOperands = SI.getNumOperands();
178:   for (unsigned I = 0; I < NumOperands; ++I) {
179:     Value *CurrOpperand = SI.getOperand(I);
180:     ConstantExpr *CE = dyn_cast<ConstantExpr>(CurrOpperand);
```
- EN: This range implements operational logic in helpers such as assert, getInt32, CreateMul, CreateAdd, translating backend policy into executable code.
- CN: 这一段实现了 assert、getInt32、CreateMul、CreateAdd 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:     if (CE && CE->getOpcode() == Instruction::GetElementPtr) {
182:       GetElementPtrInst *OldGEP =
183:           cast<GetElementPtrInst>(CE->getAsInstruction());
184:       OldGEP->insertBefore(SI.getIterator());
185:
186:       IRBuilder<> Builder(&SI);
187:       StoreInst *NewStore = Builder.CreateStore(SI.getValueOperand(), OldGEP);
188:       NewStore->setAlignment(SI.getAlign());
189:       SI.replaceAllUsesWith(NewStore);
190:       SI.eraseFromParent();
191:       visitGetElementPtrInst(*OldGEP);
192:       return true;
193:     }
194:   }
195:   return false;
196: }
197:
198: bool DXILFlattenArraysVisitor::visitAllocaInst(AllocaInst &AI) {
199:   if (!isMultiDimensionalArray(AI.getAllocatedType()))
200:     return false;
201:
202:   ArrayType *ArrType = cast<ArrayType>(AI.getAllocatedType());
203:   IRBuilder<> Builder(&AI);
204:   auto [TotalElements, BaseType] = getElementCountAndType(ArrType);
205:
206:   ArrayType *FattenedArrayType = ArrayType::get(BaseType, TotalElements);
207:   AllocaInst *FlatAlloca =
208:       Builder.CreateAlloca(FattenedArrayType, nullptr, AI.getName() + ".1dim");
209:   FlatAlloca->setAlignment(AI.getAlign());
210:   AI.replaceAllUsesWith(FlatAlloca);
211:   AI.eraseFromParent();
212:   return true;
213: }
214:
215: bool DXILFlattenArraysVisitor::visitGetElementPtrInst(GetElementPtrInst &GEP) {
216:   // Do not visit GEPs more than once
217:   if (GEPChainInfoMap.contains(cast<GEPOperator>(&GEP)))
218:     return false;
219:
220:   Value *PtrOperand = GEP.getPointerOperand();
221:   // It shouldn't(?) be possible for the pointer operand of a GEP to be a PHI
222:   // node unless HLSL has pointers. If this assumption is incorrect or HLSL gets
223:   // pointer types, then the handling of this case can be implemented later.
224:   assert(!isa<PHINode>(PtrOperand) &&
225:          "Pointer operand of GEP should not be a PHI Node");
226:
227:   // Replace a GEP ConstantExpr pointer operand with a GEP instruction so that
228:   // it can be visited
229:   if (auto *PtrOpGEPCE = dyn_cast<ConstantExpr>(PtrOperand);
230:       PtrOpGEPCE && PtrOpGEPCE->getOpcode() == Instruction::GetElementPtr) {
231:     GetElementPtrInst *OldGEPI =
232:         cast<GetElementPtrInst>(PtrOpGEPCE->getAsInstruction());
233:     OldGEPI->insertBefore(GEP.getIterator());
234:
235:     IRBuilder<> Builder(&GEP);
236:     SmallVector<Value *> Indices(GEP.indices());
237:     Value *NewGEP =
238:         Builder.CreateGEP(GEP.getSourceElementType(), OldGEPI, Indices,
239:                           GEP.getName(), GEP.getNoWrapFlags());
240:     assert(isa<GetElementPtrInst>(NewGEP) &&
```
- EN: This range implements operational logic in helpers such as getAsInstruction, insertBefore, Builder, CreateStore, translating backend policy into executable code.
- CN: 这一段实现了 getAsInstruction、insertBefore、Builder、CreateStore 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:            "Expected newly-created GEP to be an instruction");
242:     GetElementPtrInst *NewGEPI = cast<GetElementPtrInst>(NewGEP);
243:
244:     GEP.replaceAllUsesWith(NewGEPI);
245:     GEP.eraseFromParent();
246:     visitGetElementPtrInst(*OldGEPI);
247:     visitGetElementPtrInst(*NewGEPI);
248:     return true;
249:   }
250:
251:   // Construct GEPInfo for this GEP
252:   GEPInfo Info;
253:
254:   // Obtain the variable and constant byte offsets computed by this GEP
255:   const DataLayout &DL = GEP.getDataLayout();
256:   unsigned BitWidth = DL.getIndexTypeSizeInBits(GEP.getType());
257:   Info.ConstantOffset = {BitWidth, 0};
258:   [[maybe_unused]] bool Success = GEP.collectOffset(
259:       DL, BitWidth, Info.VariableOffsets, Info.ConstantOffset);
260:   assert(Success && "Failed to collect offsets for GEP");
261:
262:   // If there is a parent GEP, inherit the root array type and pointer, and
263:   // merge the byte offsets. Otherwise, this GEP is itself the root of a GEP
264:   // chain and we need to deterine the root array type
265:   if (auto *PtrOpGEP = dyn_cast<GEPOperator>(PtrOperand)) {
266:
267:     // If the parent GEP was not processed, then we do not want to process its
268:     // descendants. This can happen if the GEP chain is for an unsupported type
269:     // such as a struct -- we do not flatten structs nor GEP chains for structs
270:     if (!GEPChainInfoMap.contains(PtrOpGEP))
271:       return false;
272:
273:     GEPInfo &PGEPInfo = GEPChainInfoMap[PtrOpGEP];
274:     Info.RootFlattenedArrayType = PGEPInfo.RootFlattenedArrayType;
275:     Info.RootPointerOperand = PGEPInfo.RootPointerOperand;
276:     for (auto &VariableOffset : PGEPInfo.VariableOffsets)
277:       Info.VariableOffsets.insert(VariableOffset);
278:     Info.ConstantOffset += PGEPInfo.ConstantOffset;
279:   } else {
280:     Info.RootPointerOperand = PtrOperand;
281:
282:     // We should try to determine the type of the root from the pointer rather
283:     // than the GEP's source element type because this could be a scalar GEP
284:     // into an array-typed pointer from an Alloca or Global Variable.
285:     Type *RootTy = GEP.getSourceElementType();
286:     if (auto *GlobalVar = dyn_cast<GlobalVariable>(PtrOperand)) {
287:       if (GlobalMap.contains(GlobalVar))
288:         GlobalVar = GlobalMap[GlobalVar];
289:       Info.RootPointerOperand = GlobalVar;
290:       RootTy = GlobalVar->getValueType();
291:     } else if (auto *Alloca = dyn_cast<AllocaInst>(PtrOperand))
292:       RootTy = Alloca->getAllocatedType();
293:     assert(!isMultiDimensionalArray(RootTy) &&
294:            "Expected root array type to be flattened");
295:
296:     // If the root type is not an array, we don't need to do any flattening
297:     if (!isa<ArrayType>(RootTy))
298:       return false;
299:
300:     Info.RootFlattenedArrayType = cast<ArrayType>(RootTy);
```
- EN: This range implements operational logic in helpers such as replaceAllUsesWith, eraseFromParent, visitGetElementPtrInst, getDataLayout, translating backend policy into executable code.
- CN: 这一段实现了 replaceAllUsesWith、eraseFromParent、visitGetElementPtrInst、getDataLayout 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:   }
302:
303:   // GEPs without users or GEPs with non-GEP users should be replaced such that
304:   // the chain of GEPs they are a part of are collapsed to a single GEP into a
305:   // flattened array.
306:   bool ReplaceThisGEP = GEP.users().empty();
307:   for (Value *User : GEP.users())
308:     if (!isa<GetElementPtrInst>(User))
309:       ReplaceThisGEP = true;
310:
311:   if (ReplaceThisGEP) {
312:     unsigned BytesPerElem =
313:         DL.getTypeAllocSize(Info.RootFlattenedArrayType->getArrayElementType());
314:     assert(isPowerOf2_32(BytesPerElem) &&
315:            "Bytes per element should be a power of 2");
316:
317:     // Compute the 32-bit index for this flattened GEP from the constant and
318:     // variable byte offsets in the GEPInfo
319:     IRBuilder<> Builder(&GEP);
320:     Value *ZeroIndex = Builder.getInt32(0);
321:     uint64_t ConstantOffset =
322:         Info.ConstantOffset.udiv(BytesPerElem).getZExtValue();
323:     assert(ConstantOffset < UINT32_MAX &&
324:            "Constant byte offset for flat GEP index must fit within 32 bits");
325:     Value *FlattenedIndex = Builder.getInt32(ConstantOffset);
326:     for (auto [VarIndex, Multiplier] : Info.VariableOffsets) {
327:       assert(Multiplier.getActiveBits() <= 32 &&
328:              "The multiplier for a flat GEP index must fit within 32 bits");
329:       assert(VarIndex->getType()->isIntegerTy(32) &&
330:              "Expected i32-typed GEP indices");
331:       Value *VI;
332:       if (Multiplier.getZExtValue() % BytesPerElem != 0) {
333:         // This can happen, e.g., with i8 GEPs. To handle this we just divide
334:         // by BytesPerElem using an instruction after multiplying VarIndex by
335:         // Multiplier.
336:         VI = Builder.CreateMul(VarIndex,
337:                                Builder.getInt32(Multiplier.getZExtValue()));
338:         VI = Builder.CreateLShr(VI, Builder.getInt32(Log2_32(BytesPerElem)));
339:       } else
340:         VI = Builder.CreateMul(
341:             VarIndex,
342:             Builder.getInt32(Multiplier.getZExtValue() / BytesPerElem));
343:       FlattenedIndex = Builder.CreateAdd(FlattenedIndex, VI);
344:     }
345:
346:     // Construct a new GEP for the flattened array to replace the current GEP
347:     Value *NewGEP = Builder.CreateGEP(
348:         Info.RootFlattenedArrayType, Info.RootPointerOperand,
349:         {ZeroIndex, FlattenedIndex}, GEP.getName(), GEP.getNoWrapFlags());
350:
351:     // If the pointer operand is a global variable and all indices are 0,
352:     // IRBuilder::CreateGEP will return the global variable instead of creating
353:     // a GEP instruction or GEP ConstantExpr. In this case we have to create and
354:     // insert our own GEP instruction.
355:     if (!isa<GEPOperator>(NewGEP))
356:       NewGEP = GetElementPtrInst::Create(
357:           Info.RootFlattenedArrayType, Info.RootPointerOperand,
358:           {ZeroIndex, FlattenedIndex}, GEP.getNoWrapFlags(), GEP.getName(),
359:           Builder.GetInsertPoint());
360:
```
- EN: This range implements operational logic in helpers such as users, getTypeAllocSize, Builder, getInt32, translating backend policy into executable code.
- CN: 这一段实现了 users、getTypeAllocSize、Builder、getInt32 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:     // Replace the current GEP with the new GEP. Store GEPInfo into the map
362:     // for later use in case this GEP was not the end of the chain
363:     GEPChainInfoMap.insert({cast<GEPOperator>(NewGEP), std::move(Info)});
364:     GEP.replaceAllUsesWith(NewGEP);
365:     GEP.eraseFromParent();
366:     return true;
367:   }
368:
369:   // This GEP is potentially dead at the end of the pass since it may not have
370:   // any users anymore after GEP chains have been collapsed. We retain store
371:   // GEPInfo for GEPs down the chain to use to compute their indices.
372:   GEPChainInfoMap.insert({cast<GEPOperator>(&GEP), std::move(Info)});
373:   PotentiallyDeadInstrs.emplace_back(&GEP);
374:   return false;
375: }
376:
377: bool DXILFlattenArraysVisitor::visit(Function &F) {
378:   bool MadeChange = false;
379:   ReversePostOrderTraversal<Function *> RPOT(&F);
380:   for (BasicBlock *BB : make_early_inc_range(RPOT)) {
381:     for (Instruction &I : make_early_inc_range(*BB))
382:       MadeChange |= InstVisitor::visit(I);
383:   }
384:   finish();
385:   return MadeChange;
386: }
387:
388: static void collectElements(Constant *Init,
389:                             SmallVectorImpl<Constant *> &Elements) {
390:   // Base case: If Init is not an array, add it directly to the vector.
391:   auto *ArrayTy = dyn_cast<ArrayType>(Init->getType());
392:   if (!ArrayTy) {
393:     Elements.push_back(Init);
394:     return;
395:   }
396:   unsigned ArrSize = ArrayTy->getNumElements();
397:   if (isa<ConstantAggregateZero>(Init)) {
398:     for (unsigned I = 0; I < ArrSize; ++I)
399:       Elements.push_back(Constant::getNullValue(ArrayTy->getElementType()));
400:     return;
401:   }
402:
403:   // Recursive case: Process each element in the array.
404:   if (auto *ArrayConstant = dyn_cast<ConstantArray>(Init)) {
405:     for (unsigned I = 0; I < ArrayConstant->getNumOperands(); ++I) {
406:       collectElements(ArrayConstant->getOperand(I), Elements);
407:     }
408:   } else if (auto *DataArrayConstant = dyn_cast<ConstantDataArray>(Init)) {
409:     for (unsigned I = 0; I < DataArrayConstant->getNumElements(); ++I) {
410:       collectElements(DataArrayConstant->getElementAsConstant(I), Elements);
411:     }
412:   } else {
413:     llvm_unreachable(
414:         "Expected a ConstantArray or ConstantDataArray for array initializer!");
415:   }
416: }
417:
418: static Constant *transformInitializer(Constant *Init, Type *OrigType,
419:                                       ArrayType *FlattenedType,
420:                                       LLVMContext &Ctx) {
```
- EN: This range implements operational logic in helpers such as insert, replaceAllUsesWith, eraseFromParent, emplace_back, translating backend policy into executable code.
- CN: 这一段实现了 insert、replaceAllUsesWith、eraseFromParent、emplace_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:   // Handle ConstantAggregateZero (zero-initialized constants)
422:   if (isa<ConstantAggregateZero>(Init))
423:     return ConstantAggregateZero::get(FlattenedType);
424:
425:   // Handle UndefValue (undefined constants)
426:   if (isa<UndefValue>(Init))
427:     return UndefValue::get(FlattenedType);
428:
429:   if (!isa<ArrayType>(OrigType))
430:     return Init;
431:
432:   SmallVector<Constant *> FlattenedElements;
433:   collectElements(Init, FlattenedElements);
434:   assert(FlattenedType->getNumElements() == FlattenedElements.size() &&
435:          "The number of collected elements should match the FlattenedType");
436:   return ConstantArray::get(FlattenedType, FlattenedElements);
437: }
438:
439: static void flattenGlobalArrays(
440:     Module &M, SmallDenseMap<GlobalVariable *, GlobalVariable *> &GlobalMap) {
441:   LLVMContext &Ctx = M.getContext();
442:   for (GlobalVariable &G : M.globals()) {
443:     Type *OrigType = G.getValueType();
444:     if (!DXILFlattenArraysVisitor::isMultiDimensionalArray(OrigType))
445:       continue;
446:
447:     ArrayType *ArrType = cast<ArrayType>(OrigType);
448:     auto [TotalElements, BaseType] =
449:         DXILFlattenArraysVisitor::getElementCountAndType(ArrType);
450:     ArrayType *FattenedArrayType = ArrayType::get(BaseType, TotalElements);
451:
452:     // Create a new global variable with the updated type
453:     // Note: Initializer is set via transformInitializer
454:     GlobalVariable *NewGlobal =
455:         new GlobalVariable(M, FattenedArrayType, G.isConstant(), G.getLinkage(),
456:                            /*Initializer=*/nullptr, G.getName() + ".1dim", &G,
457:                            G.getThreadLocalMode(), G.getAddressSpace(),
458:                            G.isExternallyInitialized());
459:
460:     // Copy relevant attributes
461:     NewGlobal->setUnnamedAddr(G.getUnnamedAddr());
462:     if (G.getAlignment() > 0) {
463:       NewGlobal->setAlignment(G.getAlign());
464:     }
465:
466:     if (G.hasInitializer()) {
467:       Constant *Init = G.getInitializer();
468:       Constant *NewInit =
469:           transformInitializer(Init, OrigType, FattenedArrayType, Ctx);
470:       NewGlobal->setInitializer(NewInit);
471:     }
472:     GlobalMap[&G] = NewGlobal;
473:   }
474: }
475:
476: static bool flattenArrays(Module &M) {
477:   bool MadeChange = false;
478:   SmallDenseMap<GlobalVariable *, GlobalVariable *> GlobalMap;
479:   flattenGlobalArrays(M, GlobalMap);
480:   DXILFlattenArraysVisitor Impl(GlobalMap);
```
- EN: This range implements operational logic in helpers such as ConstantAggregateZero::get, UndefValue::get, collectElements, ConstantArray::get, translating backend policy into executable code.
- CN: 这一段实现了 ConstantAggregateZero::get、UndefValue::get、collectElements、ConstantArray::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-515
```cpp
481:   for (auto &F : make_early_inc_range(M.functions())) {
482:     if (F.isDeclaration())
483:       continue;
484:     MadeChange |= Impl.visit(F);
485:   }
486:   for (auto &[Old, New] : GlobalMap) {
487:     Old->replaceAllUsesWith(New);
488:     Old->eraseFromParent();
489:     MadeChange = true;
490:   }
491:   return MadeChange;
492: }
493:
494: PreservedAnalyses DXILFlattenArrays::run(Module &M, ModuleAnalysisManager &) {
495:   bool MadeChanges = flattenArrays(M);
496:   if (!MadeChanges)
497:     return PreservedAnalyses::all();
498:   PreservedAnalyses PA;
499:   return PA;
500: }
501:
502: bool DXILFlattenArraysLegacy::runOnModule(Module &M) {
503:   return flattenArrays(M);
504: }
505:
506: char DXILFlattenArraysLegacy::ID = 0;
507:
508: INITIALIZE_PASS_BEGIN(DXILFlattenArraysLegacy, DEBUG_TYPE,
509:                       "DXIL Array Flattener", false, false)
510: INITIALIZE_PASS_END(DXILFlattenArraysLegacy, DEBUG_TYPE, "DXIL Array Flattener",
511:                     false, false)
512:
513: ModulePass *llvm::createDXILFlattenArraysLegacyPass() {
514:   return new DXILFlattenArraysLegacy();
515: }
```
- EN: This range implements operational logic in helpers such as visit, replaceAllUsesWith, eraseFromParent, DXILFlattenArrays::run, translating backend policy into executable code.
- CN: 这一段实现了 visit、replaceAllUsesWith、eraseFromParent、DXILFlattenArrays::run 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include DXILFlattenArraysLegacy, runOnModule, GEPInfo, DXILFlattenArraysVisitor, GlobalMap, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DXILFlattenArraysLegacy, runOnModule, GEPInfo, DXILFlattenArraysVisitor, GlobalMap，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILFlattenArrays.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/PostOrderIterator.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/IR/BasicBlock.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstVisitor.h`
  - `llvm/IR/ReplaceConstant.h`
  - `llvm/Support/Casting.h`
  - `llvm/Support/MathExtras.h`
  - `llvm/Transforms/Utils/Local.h`
- System/standard headers / 系统或标准头文件:
  - `cassert`
  - `cstddef`
  - `cstdint`
  - `utility`
