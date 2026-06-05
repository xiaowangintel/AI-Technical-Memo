# SPIRVLegalizePointerCast.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVLegalizePointerCast.cpp`
- Repository: `llvm-project`
- Purpose (EN): The LLVM IR has multiple legal patterns we cannot lower to Logical SPIR-V.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVLegalizePointerCast.cpp ----------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The LLVM IR has multiple legal patterns we cannot lower to Logical SPIR-V.
10: // This pass modifies such loads to have an IR we can directly lower to valid
11: // logical SPIR-V.
12: // OpenCL can avoid this because they rely on ptrcast, which is not supported
13: // by logical SPIR-V.
14: //
15: // This pass relies on the assign_ptr_type intrinsic to deduce the type of the
16: // pointed values, must replace all occurences of `ptrcast`. This is why
17: // unhandled cases are reported as unreachable: we MUST cover all cases.
18: //
19: // 1. Loading the first element of an array
20: //
21: //        %array = [10 x i32]
22: //        %value = load i32, ptr %array
23: //
24: //    LLVM can skip the GEP instruction, and only request loading the first 4
25: //    bytes. In logical SPIR-V, we need an OpAccessChain to access the first
26: //    element. This pass will add a getelementptr instruction before the load.
27: //
28: //
29: // 2. Implicit downcast from load
30: //
31: //        %1 = getelementptr <4 x i32>, ptr %vec4, i64 0
32: //        %2 = load <3 x i32>, ptr %1
33: //
34: //    The pointer in the GEP instruction is only used for offset computations,
35: //    but it doesn't NEED to match the pointed type. OpAccessChain however
36: //    requires this. Also, LLVM loads define the bitwidth of the load, not the
37: //    pointer. In this example, we can guess %vec4 is a vec4 thanks to the GEP
38: //    instruction basetype, but we only want to load the first 3 elements, hence
39: //    do a partial load. In logical SPIR-V, this is not legal. What we must do
40: //    is load the full vector (basetype), extract 3 elements, and recombine them
41: //    to form a 3-element vector.
42: //
43: //===----------------------------------------------------------------------===//
44:
45: #include "SPIRVLegalizePointerCast.h"
46: #include "SPIRV.h"
47: #include "SPIRVSubtarget.h"
48: #include "SPIRVTargetMachine.h"
49: #include "SPIRVUtils.h"
50: #include "llvm/IR/IRBuilder.h"
51: #include "llvm/IR/IntrinsicInst.h"
52: #include "llvm/IR/Intrinsics.h"
53: #include "llvm/IR/IntrinsicsSPIRV.h"
54: #include "llvm/Transforms/Utils/Cloning.h"
55: #include "llvm/Transforms/Utils/LowerMemIntrinsics.h"
56:
57: using namespace llvm;
58:
59: namespace {
60: class SPIRVLegalizePointerCastImpl {
```
- EN: This range defines or declares important types such as SPIRVLegalizePointerCastImpl, shaping the data model used by SPIRVLegalizePointerCast.cpp.
- CN: 这一段定义或声明了 SPIRVLegalizePointerCastImpl 等关键类型，构成 SPIRVLegalizePointerCast.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:
 62:   // Builds the `spv_assign_type` assigning |Ty| to |Value| at the current
 63:   // builder position.
 64:   void buildAssignType(IRBuilder<> &B, Type *Ty, Value *Arg) {
 65:     Value *OfType = PoisonValue::get(Ty);
 66:     CallInst *AssignCI = buildIntrWithMD(Intrinsic::spv_assign_type,
 67:                                          {Arg->getType()}, OfType, Arg, {}, B);
 68:     GR->addAssignPtrTypeInstr(Arg, AssignCI);
 69:   }
 70:
 71:   static FixedVectorType *makeVectorFromTotalBits(Type *ElemTy,
 72:                                                   TypeSize TotalBits) {
 73:     unsigned ElemBits = ElemTy->getScalarSizeInBits();
 74:     assert(ElemBits && TotalBits % ElemBits == 0 &&
 75:            "TotalBits must be divisible by element bit size");
 76:     return FixedVectorType::get(ElemTy, TotalBits / ElemBits);
 77:   }
 78:
 79:   Value *resizeVectorBitsWithShuffle(IRBuilder<> &B, Value *V,
 80:                                      FixedVectorType *DstTy) {
 81:     auto *SrcTy = cast<FixedVectorType>(V->getType());
 82:     assert(SrcTy->getElementType() == DstTy->getElementType() &&
 83:            "shuffle resize expects identical element types");
 84:
 85:     const unsigned NumNeeded = DstTy->getNumElements();
 86:     const unsigned NumSource = SrcTy->getNumElements();
 87:
 88:     SmallVector<int> Mask(NumNeeded);
 89:     for (unsigned I = 0; I < NumNeeded; ++I)
 90:       Mask[I] = (I < NumSource) ? static_cast<int>(I) : -1;
 91:
 92:     Value *Resized = B.CreateShuffleVector(V, V, Mask);
 93:     buildAssignType(B, DstTy, Resized);
 94:     return Resized;
 95:   }
 96:
 97:   // Loads parts of the vector of type |SourceType| from the pointer |Source|
 98:   // and create a new vector of type |TargetType|. |TargetType| must be a vector
 99:   // type.
100:   // Returns the loaded value.
101:   Value *loadVectorFromVector(IRBuilder<> &B, FixedVectorType *SourceType,
102:                               FixedVectorType *TargetType, Value *Source) {
103:     LoadInst *NewLoad = B.CreateLoad(SourceType, Source);
104:     buildAssignType(B, SourceType, NewLoad);
105:     Value *AssignValue = NewLoad;
106:     if (TargetType->getElementType() != SourceType->getElementType()) {
107:       const DataLayout &DL = B.GetInsertBlock()->getModule()->getDataLayout();
108:       TypeSize TargetTypeSize = DL.getTypeSizeInBits(TargetType);
109:       TypeSize SourceTypeSize = DL.getTypeSizeInBits(SourceType);
110:
111:       Value *BitcastSrcVal = NewLoad;
112:       FixedVectorType *BitcastSrcTy =
113:           cast<FixedVectorType>(BitcastSrcVal->getType());
114:       FixedVectorType *BitcastDstTy = TargetType;
115:
116:       if (TargetTypeSize != SourceTypeSize) {
117:         unsigned TargetElemBits =
118:             TargetType->getElementType()->getScalarSizeInBits();
119:         if (SourceTypeSize % TargetElemBits == 0) {
120:           // No Resize needed. Same total bits as source, but use target element
```
- EN: This range implements operational logic in helpers such as buildAssignType, PoisonValue::get, getType, addAssignPtrTypeInstr, translating backend policy into executable code.
- CN: 这一段实现了 buildAssignType、PoisonValue::get、getType、addAssignPtrTypeInstr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:           // type.
122:           BitcastDstTy = makeVectorFromTotalBits(TargetType->getElementType(),
123:                                                  SourceTypeSize);
124:         } else {
125:           // Resize source to target total bitwidth using source element type.
126:           BitcastSrcTy = makeVectorFromTotalBits(SourceType->getElementType(),
127:                                                  TargetTypeSize);
128:           BitcastSrcVal = resizeVectorBitsWithShuffle(B, NewLoad, BitcastSrcTy);
129:         }
130:       }
131:       AssignValue =
132:           B.CreateIntrinsic(Intrinsic::spv_bitcast,
133:                             {BitcastDstTy, BitcastSrcTy}, {BitcastSrcVal});
134:       buildAssignType(B, BitcastDstTy, AssignValue);
135:       if (BitcastDstTy == TargetType)
136:         return AssignValue;
137:     }
138:
139:     assert(TargetType->getNumElements() < SourceType->getNumElements());
140:     SmallVector<int> Mask(/* Size= */ TargetType->getNumElements());
141:     for (unsigned I = 0; I < TargetType->getNumElements(); ++I)
142:       Mask[I] = I;
143:     Value *Output = B.CreateShuffleVector(AssignValue, AssignValue, Mask);
144:     buildAssignType(B, TargetType, Output);
145:     return Output;
146:   }
147:
148:   // Returns true if |FromTy| has a memory layout compatible with loading or
149:   // storing |ToTy|.
150:   bool isCompatibleMemoryLayout(Type *ToTy, Type *FromTy) {
151:     if (ToTy == FromTy)
152:       return true;
153:     auto *SVT = dyn_cast<FixedVectorType>(FromTy);
154:     auto *DVT = dyn_cast<FixedVectorType>(ToTy);
155:     if (SVT && DVT)
156:       return true;
157:     auto *SAT = dyn_cast<ArrayType>(FromTy);
158:     if (SAT && DVT) {
159:       if (SAT->getElementType() == DVT->getElementType())
160:         return true;
161:       if (auto *MAT = dyn_cast<FixedVectorType>(SAT->getElementType()))
162:         if (MAT->getElementType() == DVT->getElementType())
163:           return true;
164:     }
165:     return false;
166:   }
167:
168:   // Traverses the aggregate type to find the first sub-type that matches
169:   // the TargetElemType's memory layout, optionally emitting a GEP intrinsic.
170:   std::optional<std::pair<Value *, Type *>>
171:   getPointerToFirstCompatibleType(IRBuilder<> &B, Value *BasePtr,
172:                                   Type *PointerType, Type *TargetElemType,
173:                                   bool IsInBounds) {
174:     Type *CurrentTy = GR->findDeducedElementType(BasePtr);
175:     assert(CurrentTy && "Could not deduce aggregate type");
176:     SmallVector<Value *, 8> Args{/* isInBounds= */ B.getInt1(IsInBounds),
177:                                  BasePtr};
178:     Args.push_back(B.getInt32(0)); // Pointer offset
179:
180:     while (!isCompatibleMemoryLayout(TargetElemType, CurrentTy)) {
```
- EN: This range implements operational logic in helpers such as resizeVectorBitsWithShuffle, buildAssignType, assert, Mask, translating backend policy into executable code.
- CN: 这一段实现了 resizeVectorBitsWithShuffle、buildAssignType、assert、Mask 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:       if (auto *ST = dyn_cast<StructType>(CurrentTy)) {
182:         if (ST->getNumElements() == 0)
183:           return std::nullopt;
184:         CurrentTy = ST->getTypeAtIndex(0u);
185:       } else if (auto *AT = dyn_cast<ArrayType>(CurrentTy)) {
186:         CurrentTy = AT->getElementType();
187:       } else if (auto *VT = dyn_cast<FixedVectorType>(CurrentTy)) {
188:         CurrentTy = VT->getElementType();
189:       } else {
190:         return std::nullopt;
191:       }
192:       Args.push_back(B.getInt32(0));
193:     }
194:
195:     Value *GEP = BasePtr;
196:     if (Args.size() > 3) {
197:       std::array<Type *, 2> Types = {PointerType, BasePtr->getType()};
198:       GEP = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
199:       GR->buildAssignPtr(B, CurrentTy, GEP);
200:     }
201:
202:     return std::make_pair(GEP, CurrentTy);
203:   }
204:
205:   // Builds a legalized load from a pointer, drilling down through
206:   // memory layouts to find a compatible type. Load flags will be
207:   // copied from |BadLoad|, which should be the load being legalized.
208:   Value *buildLegalizedLoad(IRBuilder<> &B, Type *ElementType, Value *Source,
209:                             LoadInst *BadLoad) {
210:     auto ResultOpt = getPointerToFirstCompatibleType(
211:         B, Source, BadLoad->getPointerOperandType(), ElementType, false);
212:     assert(ResultOpt && "Failed to load from aggregate: "
213:                         "Could not find compatible memory layout.");
214:     auto [GEP, CurrentTy] = *ResultOpt;
215:
216:     auto *SAT = dyn_cast<ArrayType>(CurrentTy);
217:     auto *SVT = dyn_cast<FixedVectorType>(CurrentTy);
218:     auto *DVT = dyn_cast<FixedVectorType>(ElementType);
219:     auto *MAT =
220:         SAT ? dyn_cast<FixedVectorType>(SAT->getElementType()) : nullptr;
221:
222:     if (ElementType == CurrentTy) {
223:       LoadInst *LI = B.CreateLoad(ElementType, GEP);
224:       LI->setAlignment(BadLoad->getAlign());
225:       buildAssignType(B, ElementType, LI);
226:       return LI;
227:     }
228:     if (SVT && DVT)
229:       return loadVectorFromVector(B, SVT, DVT, GEP);
230:     if (SAT && DVT && SAT->getElementType() == DVT->getElementType())
231:       return loadVectorFromArray(B, DVT, GEP);
232:     if (MAT && DVT && MAT->getElementType() == DVT->getElementType())
233:       return loadVectorFromMatrixArray(B, DVT, GEP, MAT);
234:
235:     llvm_unreachable("Failed to load from aggregate.");
236:   }
237:   Value *
238:   buildVectorFromLoadedElements(IRBuilder<> &B, FixedVectorType *TargetType,
239:                                 SmallVector<Value *, 4> &LoadedElements) {
240:     // Build the vector from the loaded elements.
```
- EN: This range implements operational logic in helpers such as getTypeAtIndex, getElementType, push_back, CreateIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 getTypeAtIndex、getElementType、push_back、CreateIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:     Value *NewVector = PoisonValue::get(TargetType);
242:     buildAssignType(B, TargetType, NewVector);
243:
244:     for (unsigned I = 0, E = TargetType->getNumElements(); I < E; ++I) {
245:       Value *Index = B.getInt32(I);
246:       SmallVector<Type *, 4> Types = {TargetType, TargetType,
247:                                       TargetType->getElementType(),
248:                                       Index->getType()};
249:       SmallVector<Value *> Args = {NewVector, LoadedElements[I], Index};
250:       NewVector = B.CreateIntrinsic(Intrinsic::spv_insertelt, {Types}, {Args});
251:       buildAssignType(B, TargetType, NewVector);
252:     }
253:     return NewVector;
254:   }
255:
256:   // Loads elements from a matrix with an array of vector memory layout and
257:   // constructs a vector.
258:   Value *loadVectorFromMatrixArray(IRBuilder<> &B, FixedVectorType *TargetType,
259:                                    Value *Source,
260:                                    FixedVectorType *ArrElemVecTy) {
261:     Type *TargetElemTy = TargetType->getElementType();
262:     unsigned ScalarsPerArrayElement = ArrElemVecTy->getNumElements();
263:     // Load each element of the array.
264:     SmallVector<Value *, 4> LoadedElements;
265:     std::array<Type *, 2> Types = {Source->getType(), Source->getType()};
266:     for (unsigned I = 0, E = TargetType->getNumElements(); I < E; ++I) {
267:       unsigned ArrayIndex = I / ScalarsPerArrayElement;
268:       unsigned ElementIndexInArrayElem = I % ScalarsPerArrayElement;
269:       // Create a GEP to access the i-th element of the array.
270:       std::array<Value *, 4> Args = {
271:           B.getInt1(/*Inbounds=*/false), Source, B.getInt32(0),
272:           ConstantInt::get(B.getInt32Ty(), ArrayIndex)};
273:       auto *ElementPtr = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
274:       GR->buildAssignPtr(B, ArrElemVecTy, ElementPtr);
275:       Value *LoadVec = B.CreateLoad(ArrElemVecTy, ElementPtr);
276:       buildAssignType(B, ArrElemVecTy, LoadVec);
277:       LoadedElements.push_back(makeExtractElement(B, TargetElemTy, LoadVec,
278:                                                   ElementIndexInArrayElem));
279:     }
280:     return buildVectorFromLoadedElements(B, TargetType, LoadedElements);
281:   }
282:   // Loads elements from an array and constructs a vector.
283:   Value *loadVectorFromArray(IRBuilder<> &B, FixedVectorType *TargetType,
284:                              Value *Source) {
285:     // Load each element of the array.
286:     SmallVector<Value *, 4> LoadedElements;
287:     std::array<Type *, 2> Types = {Source->getType(), Source->getType()};
288:     for (unsigned I = 0, E = TargetType->getNumElements(); I < E; ++I) {
289:       // Create a GEP to access the i-th element of the array.
290:       std::array<Value *, 4> Args = {B.getInt1(/*Inbounds=*/false), Source,
291:                                      B.getInt32(0),
292:                                      ConstantInt::get(B.getInt32Ty(), I)};
293:       auto *ElementPtr = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
294:       GR->buildAssignPtr(B, TargetType->getElementType(), ElementPtr);
295:
296:       // Load the value from the element pointer.
297:       Value *Load = B.CreateLoad(TargetType->getElementType(), ElementPtr);
298:       buildAssignType(B, TargetType->getElementType(), Load);
299:       LoadedElements.push_back(Load);
300:     }
```
- EN: This range implements operational logic in helpers such as PoisonValue::get, buildAssignType, getInt32, CreateIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 PoisonValue::get、buildAssignType、getInt32、CreateIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:     return buildVectorFromLoadedElements(B, TargetType, LoadedElements);
302:   }
303:
304:   // Stores elements from a vector into a matrix (an array of vectors).
305:   void storeMatrixArrayFromVector(IRBuilder<> &B, Value *SrcVector,
306:                                   Value *DstArrayPtr, ArrayType *ArrTy,
307:                                   Align Alignment) {
308:     auto *SrcVecTy = cast<FixedVectorType>(SrcVector->getType());
309:     auto *ArrElemVecTy = cast<FixedVectorType>(ArrTy->getElementType());
310:     Type *ElemTy = ArrElemVecTy->getElementType();
311:     unsigned ScalarsPerArrayElement = ArrElemVecTy->getNumElements();
312:     unsigned SrcNumElements = SrcVecTy->getNumElements();
313:     assert(
314:         SrcNumElements % ScalarsPerArrayElement == 0 &&
315:         "Source vector size must be a multiple of array element vector size");
316:
317:     std::array<Type *, 2> Types = {DstArrayPtr->getType(),
318:                                    DstArrayPtr->getType()};
319:
320:     for (unsigned I = 0; I < SrcNumElements; I += ScalarsPerArrayElement) {
321:       unsigned ArrayIndex = I / ScalarsPerArrayElement;
322:       // Create a GEP to access the array element.
323:       std::array<Value *, 4> Args = {
324:           B.getInt1(/*Inbounds=*/false), DstArrayPtr, B.getInt32(0),
325:           ConstantInt::get(B.getInt32Ty(), ArrayIndex)};
326:       auto *ElementPtr = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
327:       GR->buildAssignPtr(B, ArrElemVecTy, ElementPtr);
328:
329:       // Extract scalar elements from the source vector for this array slot.
330:       SmallVector<Value *, 4> Elements;
331:       for (unsigned J = 0; J < ScalarsPerArrayElement; ++J)
332:         Elements.push_back(makeExtractElement(B, ElemTy, SrcVector, I + J));
333:
334:       // Build a vector from the extracted elements and store it.
335:       Value *Vec = buildVectorFromLoadedElements(B, ArrElemVecTy, Elements);
336:       StoreInst *SI = B.CreateStore(Vec, ElementPtr);
337:       SI->setAlignment(Alignment);
338:     }
339:   }
340:
341:   // Stores elements from a vector into an array.
342:   void storeArrayFromVector(IRBuilder<> &B, Value *SrcVector,
343:                             Value *DstArrayPtr, ArrayType *ArrTy,
344:                             Align Alignment) {
345:     auto *VecTy = cast<FixedVectorType>(SrcVector->getType());
346:     Type *ElemTy = ArrTy->getElementType();
347:
348:     // Ensure the element types of the array and vector are the same.
349:     assert(VecTy->getElementType() == ElemTy &&
350:            "Element types of array and vector must be the same.");
351:     std::array<Type *, 2> Types = {DstArrayPtr->getType(),
352:                                    DstArrayPtr->getType()};
353:
354:     for (unsigned I = 0, E = VecTy->getNumElements(); I < E; ++I) {
355:       // Create a GEP to access the i-th element of the array.
356:       std::array<Value *, 4> Args = {B.getInt1(/*Inbounds=*/false), DstArrayPtr,
357:                                      B.getInt32(0),
358:                                      ConstantInt::get(B.getInt32Ty(), I)};
359:       auto *ElementPtr = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
360:       GR->buildAssignPtr(B, ElemTy, ElementPtr);
```
- EN: This range implements operational logic in helpers such as buildVectorFromLoadedElements, getType, getElementType, getNumElements, translating backend policy into executable code.
- CN: 这一段实现了 buildVectorFromLoadedElements、getType、getElementType、getNumElements 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:
362:       // Extract the element from the vector and store it.
363:       Value *Element = makeExtractElement(B, ElemTy, SrcVector, I);
364:       StoreInst *SI = B.CreateStore(Element, ElementPtr);
365:       SI->setAlignment(Alignment);
366:     }
367:   }
368:
369:   // Replaces the load instruction to get rid of the ptrcast used as source
370:   // operand.
371:   void transformLoad(IRBuilder<> &B, LoadInst *LI, Value *CastedOperand,
372:                      Value *OriginalOperand) {
373:     Type *ToTy = GR->findDeducedElementType(CastedOperand);
374:     B.SetInsertPoint(LI);
375:
376:     Value *Output = buildLegalizedLoad(B, ToTy, OriginalOperand, LI);
377:
378:     GR->replaceAllUsesWith(LI, Output, /* DeleteOld= */ true);
379:     DeadInstructions.push_back(LI);
380:   }
381:
382:   // Creates an spv_insertelt instruction (equivalent to llvm's insertelement).
383:   Value *makeInsertElement(IRBuilder<> &B, Value *Vector, Value *Element,
384:                            unsigned Index) {
385:     Type *Int32Ty = Type::getInt32Ty(B.getContext());
386:     SmallVector<Type *, 4> Types = {Vector->getType(), Vector->getType(),
387:                                     Element->getType(), Int32Ty};
388:     SmallVector<Value *> Args = {Vector, Element, B.getInt32(Index)};
389:     Instruction *NewI =
390:         B.CreateIntrinsic(Intrinsic::spv_insertelt, {Types}, {Args});
391:     buildAssignType(B, Vector->getType(), NewI);
392:     return NewI;
393:   }
394:
395:   // Creates an spv_extractelt instruction (equivalent to llvm's
396:   // extractelement).
397:   Value *makeExtractElement(IRBuilder<> &B, Type *ElementType, Value *Vector,
398:                             unsigned Index) {
399:     Type *Int32Ty = Type::getInt32Ty(B.getContext());
400:     SmallVector<Type *, 3> Types = {ElementType, Vector->getType(), Int32Ty};
401:     SmallVector<Value *> Args = {Vector, B.getInt32(Index)};
402:     Instruction *NewI =
403:         B.CreateIntrinsic(Intrinsic::spv_extractelt, {Types}, {Args});
404:     buildAssignType(B, ElementType, NewI);
405:     return NewI;
406:   }
407:
408:   // Stores the given Src vector operand into the Dst vector, adjusting the size
409:   // if required.
410:   Value *storeVectorFromVector(IRBuilder<> &B, Value *Src, Value *Dst,
411:                                Align Alignment) {
412:     FixedVectorType *SrcType = cast<FixedVectorType>(Src->getType());
413:     FixedVectorType *DstType =
414:         cast<FixedVectorType>(GR->findDeducedElementType(Dst));
415:     auto dstNumElements = DstType->getNumElements();
416:     auto srcNumElements = SrcType->getNumElements();
417:
418:     // if the element type differs, it is a bitcast.
419:     if (DstType->getElementType() != SrcType->getElementType()) {
420:       // Support bitcast between vectors of different sizes only if
```
- EN: This range implements operational logic in helpers such as makeExtractElement, CreateStore, setAlignment, findDeducedElementType, translating backend policy into executable code.
- CN: 这一段实现了 makeExtractElement、CreateStore、setAlignment、findDeducedElementType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:       // the total bitwidth is the same.
422:       [[maybe_unused]] auto dstBitWidth =
423:           DstType->getElementType()->getScalarSizeInBits() * dstNumElements;
424:       [[maybe_unused]] auto srcBitWidth =
425:           SrcType->getElementType()->getScalarSizeInBits() * srcNumElements;
426:       assert(dstBitWidth == srcBitWidth &&
427:              "Unsupported bitcast between vectors of different sizes.");
428:
429:       Src =
430:           B.CreateIntrinsic(Intrinsic::spv_bitcast, {DstType, SrcType}, {Src});
431:       buildAssignType(B, DstType, Src);
432:       SrcType = DstType;
433:
434:       StoreInst *SI = B.CreateStore(Src, Dst);
435:       SI->setAlignment(Alignment);
436:       return SI;
437:     }
438:
439:     assert(DstType->getNumElements() >= SrcType->getNumElements());
440:     LoadInst *LI = B.CreateLoad(DstType, Dst);
441:     LI->setAlignment(Alignment);
442:     Value *OldValues = LI;
443:     buildAssignType(B, OldValues->getType(), OldValues);
444:     Value *NewValues = Src;
445:
446:     for (unsigned I = 0; I < SrcType->getNumElements(); ++I) {
447:       Value *Element =
448:           makeExtractElement(B, SrcType->getElementType(), NewValues, I);
449:       OldValues = makeInsertElement(B, OldValues, Element, I);
450:     }
451:
452:     StoreInst *SI = B.CreateStore(OldValues, Dst);
453:     SI->setAlignment(Alignment);
454:     return SI;
455:   }
456:
457:   // Builds a legalized store to a pointer, drilling down through
458:   // memory layouts to find a compatible type.
459:   void buildLegalizedStore(IRBuilder<> &B, Value *Src, Value *Dst,
460:                            Align Alignment) {
461:     auto ResultOpt = getPointerToFirstCompatibleType(B, Dst, Dst->getType(),
462:                                                      Src->getType(), true);
463:     assert(ResultOpt && "Failed to store to aggregate: "
464:                         "Could not find compatible memory layout.");
465:     auto [GEP, CurrentTy] = *ResultOpt;
466:
467:     auto *DAT = dyn_cast<ArrayType>(CurrentTy);
468:     auto *DVT = dyn_cast<FixedVectorType>(CurrentTy);
469:     auto *SVT = dyn_cast<FixedVectorType>(Src->getType());
470:     auto *DMAT =
471:         DAT ? dyn_cast<FixedVectorType>(DAT->getElementType()) : nullptr;
472:
473:     if (Src->getType() == CurrentTy) {
474:       StoreInst *SI = B.CreateStore(Src, GEP);
475:       SI->setAlignment(Alignment);
476:       return;
477:     }
478:     if (DVT && SVT) {
479:       storeVectorFromVector(B, Src, GEP, Alignment);
480:       return;
```
- EN: This range implements operational logic in helpers such as CreateIntrinsic, buildAssignType, CreateStore, setAlignment, translating backend policy into executable code.
- CN: 这一段实现了 CreateIntrinsic、buildAssignType、CreateStore、setAlignment 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:     }
482:     if (DAT && SVT && SVT->getElementType() == DAT->getElementType()) {
483:       storeArrayFromVector(B, Src, GEP, DAT, Alignment);
484:       return;
485:     }
486:     if (DMAT && SVT && DMAT->getElementType() == SVT->getElementType()) {
487:       storeMatrixArrayFromVector(B, Src, GEP, DAT, Alignment);
488:       return;
489:     }
490:
491:     llvm_unreachable("Failed to store to aggregate.");
492:   }
493:
494:   // Transforms a store instruction (or SPV intrinsic) using a ptrcast as
495:   // operand into a valid logical SPIR-V store with no ptrcast.
496:   void transformStore(IRBuilder<> &B, Instruction *BadStore, Value *Src,
497:                       Value *Dst, Align Alignment) {
498:     B.SetInsertPoint(BadStore);
499:     buildLegalizedStore(B, Src, Dst, Alignment);
500:     DeadInstructions.push_back(BadStore);
501:   }
502:
503:   void legalizePointerCast(IntrinsicInst *II) {
504:     Value *CastedOperand = II;
505:     Value *OriginalOperand = II->getOperand(0);
506:
507:     IRBuilder<> B(II->getContext());
508:     std::vector<Value *> Users;
509:     for (Use &U : II->uses())
510:       Users.push_back(U.getUser());
511:
512:     for (Value *User : Users) {
513:       if (LoadInst *LI = dyn_cast<LoadInst>(User)) {
514:         transformLoad(B, LI, CastedOperand, OriginalOperand);
515:         continue;
516:       }
517:
518:       if (StoreInst *SI = dyn_cast<StoreInst>(User)) {
519:         transformStore(B, SI, SI->getValueOperand(), OriginalOperand,
520:                        SI->getAlign());
521:         continue;
522:       }
523:
524:       if (IntrinsicInst *Intrin = dyn_cast<IntrinsicInst>(User)) {
525:         if (Intrin->getIntrinsicID() == Intrinsic::spv_assign_ptr_type) {
526:           DeadInstructions.push_back(Intrin);
527:           continue;
528:         }
529:
530:         if (Intrin->getIntrinsicID() == Intrinsic::spv_gep) {
531:           GR->replaceAllUsesWith(CastedOperand, OriginalOperand,
532:                                  /* DeleteOld= */ false);
533:           continue;
534:         }
535:
536:         if (Intrin->getIntrinsicID() == Intrinsic::spv_store) {
537:           Align Alignment;
538:           if (ConstantInt *C = dyn_cast<ConstantInt>(Intrin->getOperand(3)))
539:             Alignment = Align(C->getZExtValue());
540:           transformStore(B, Intrin, Intrin->getArgOperand(0), OriginalOperand,
```
- EN: This range implements operational logic in helpers such as storeArrayFromVector, storeMatrixArrayFromVector, llvm_unreachable, SetInsertPoint, translating backend policy into executable code.
- CN: 这一段实现了 storeArrayFromVector、storeMatrixArrayFromVector、llvm_unreachable、SetInsertPoint 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:                          Alignment);
542:           continue;
543:         }
544:       }
545:
546:       llvm_unreachable("Unsupported ptrcast user. Please fix.");
547:     }
548:
549:     DeadInstructions.push_back(II);
550:   }
551:
552: public:
553:   SPIRVLegalizePointerCastImpl(const SPIRVTargetMachine &TM) : TM(TM) {}
554:
555:   bool run(Function &F) {
556:     const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(F);
557:     GR = ST.getSPIRVGlobalRegistry();
558:     DeadInstructions.clear();
559:
560:     std::vector<IntrinsicInst *> WorkList;
561:     for (auto &BB : F) {
562:       for (auto &I : BB) {
563:         auto *II = dyn_cast<IntrinsicInst>(&I);
564:         if (II && II->getIntrinsicID() == Intrinsic::spv_ptrcast)
565:           WorkList.push_back(II);
566:       }
567:     }
568:
569:     for (IntrinsicInst *II : WorkList)
570:       legalizePointerCast(II);
571:
572:     for (Instruction *I : DeadInstructions)
573:       I->eraseFromParent();
574:
575:     return DeadInstructions.size() != 0;
576:   }
577:
578: private:
579:   const SPIRVTargetMachine &TM;
580:   SPIRVGlobalRegistry *GR = nullptr;
581:   std::vector<Instruction *> DeadInstructions;
582: };
583:
584: class SPIRVLegalizePointerCastLegacy : public FunctionPass {
585: public:
586:   static char ID;
587:   SPIRVLegalizePointerCastLegacy(const SPIRVTargetMachine &TM)
588:       : FunctionPass(ID), TM(TM) {}
589:
590:   bool runOnFunction(Function &F) override {
591:     return SPIRVLegalizePointerCastImpl(TM).run(F);
592:   }
593:
594: private:
595:   const SPIRVTargetMachine &TM;
596: };
597: } // namespace
598:
599: PreservedAnalyses SPIRVLegalizePointerCast::run(Function &F,
600:                                                 FunctionAnalysisManager &AM) {
```
- EN: This range defines or declares important types such as llvm_unreachable, push_back, SPIRVLegalizePointerCastImpl, run, shaping the data model used by SPIRVLegalizePointerCast.cpp.
- CN: 这一段定义或声明了 llvm_unreachable、push_back、SPIRVLegalizePointerCastImpl、run 等关键类型，构成 SPIRVLegalizePointerCast.cpp 使用的数据模型。

### Lines 601-611
```cpp
601:   return SPIRVLegalizePointerCastImpl(TM).run(F) ? PreservedAnalyses::none()
602:                                                  : PreservedAnalyses::all();
603: }
604:
605: char SPIRVLegalizePointerCastLegacy::ID = 0;
606: INITIALIZE_PASS(SPIRVLegalizePointerCastLegacy, "spirv-legalize-pointer-cast",
607:                 "SPIRV legalize pointer cast pass", false, false)
608:
609: FunctionPass *llvm::createSPIRVLegalizePointerCastPass(SPIRVTargetMachine *TM) {
610:   return new SPIRVLegalizePointerCastLegacy(*TM);
611: }
```
- EN: This range implements operational logic in helpers such as SPIRVLegalizePointerCastImpl, PreservedAnalyses::all, llvm::createSPIRVLegalizePointerCastPass, SPIRVLegalizePointerCastLegacy, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVLegalizePointerCastImpl、PreservedAnalyses::all、llvm::createSPIRVLegalizePointerCastPass、SPIRVLegalizePointerCastLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVLegalizePointerCastImpl, buildAssignType, PoisonValue::get, getType, addAssignPtrTypeInstr, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVLegalizePointerCastImpl, buildAssignType, PoisonValue::get, getType, addAssignPtrTypeInstr，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVLegalizePointerCast.h`
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/Transforms/Utils/Cloning.h`
  - `llvm/Transforms/Utils/LowerMemIntrinsics.h`
