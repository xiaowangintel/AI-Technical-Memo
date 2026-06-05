# DXILResourceAccess.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILResourceAccess.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILResourceAccess support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILResourceAccess.cpp - Resource access via load/store ------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILResourceAccess.h"
10: #include "DirectX.h"
11: #include "llvm/ADT/SetVector.h"
12: #include "llvm/Analysis/DXILResource.h"
13: #include "llvm/Analysis/VectorUtils.h"
14: #include "llvm/Frontend/HLSL/HLSLResource.h"
15: #include "llvm/IR/BasicBlock.h"
16: #include "llvm/IR/DiagnosticInfo.h"
17: #include "llvm/IR/Dominators.h"
18: #include "llvm/IR/IRBuilder.h"
19: #include "llvm/IR/Instruction.h"
20: #include "llvm/IR/Instructions.h"
21: #include "llvm/IR/IntrinsicInst.h"
22: #include "llvm/IR/Intrinsics.h"
23: #include "llvm/IR/IntrinsicsDirectX.h"
24: #include "llvm/IR/LLVMContext.h"
25: #include "llvm/IR/User.h"
26: #include "llvm/InitializePasses.h"
27: #include "llvm/Support/FormatVariadic.h"
28: #include "llvm/Transforms/Utils/ValueMapper.h"
29:
30: #define DEBUG_TYPE "dxil-resource-access"
31:
32: using namespace llvm;
33:
34: static void diagnoseNonUniqueResourceAccess(Instruction *I,
35:                                             ArrayRef<IntrinsicInst *> Handles) {
36:   LLVMContext &Context = I->getContext();
37:   std::string InstStr;
38:   raw_string_ostream InstOS(InstStr);
39:   I->print(InstOS);
40:   Context.diagnose(
41:       DiagnosticInfoGeneric("At resource access:" + Twine(InstStr), DS_Note));
42:
43:   for (auto *Handle : Handles) {
44:     std::string HandleStr;
45:     raw_string_ostream HandleOS(HandleStr);
46:     Handle->print(HandleOS);
47:     Context.diagnose(DiagnosticInfoGeneric(
48:         "Uses resource handle:" + Twine(HandleStr), DS_Note));
49:   }
50:   Context.diagnose(DiagnosticInfoGeneric(
51:       "Resource access is not guaranteed to map to a unique global resource"));
52: }
53:
54: static Value *traverseGEPOffsets(const DataLayout &DL, IRBuilder<> &Builder,
55:                                  Value *Ptr, uint64_t AccessSize) {
56:   Value *Offset = nullptr;
57:
58:   while (Ptr) {
59:     if (auto *II = dyn_cast<IntrinsicInst>(Ptr)) {
60:       assert(II->getIntrinsicID() == Intrinsic::dx_resource_getpointer &&
```
- EN: This range implements operational logic in helpers such as getContext, InstOS, print, DiagnosticInfoGeneric, translating backend policy into executable code.
- CN: 这一段实现了 getContext、InstOS、print、DiagnosticInfoGeneric 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:              "Resource access through unexpected intrinsic");
 62:       return Offset ? Offset : ConstantInt::get(Builder.getInt32Ty(), 0);
 63:     }
 64:
 65:     auto *GEP = dyn_cast<GetElementPtrInst>(Ptr);
 66:     assert(GEP && "Resource access through unexpected instruction");
 67:
 68:     unsigned NumIndices = GEP->getNumIndices();
 69:     uint64_t IndexScale = DL.getTypeAllocSize(GEP->getSourceElementType());
 70:     APInt ConstantOffset(DL.getIndexTypeSizeInBits(GEP->getType()), 0);
 71:     Value *GEPOffset;
 72:     if (GEP->accumulateConstantOffset(DL, ConstantOffset)) {
 73:       // We have a constant offset (in bytes).
 74:       GEPOffset =
 75:           ConstantInt::get(DL.getIndexType(GEP->getType()), ConstantOffset);
 76:       IndexScale = 1;
 77:     } else if (NumIndices == 1) {
 78:       // If we have a single index we're indexing into a top level array. This
 79:       // generally only happens with cbuffers.
 80:       GEPOffset = *GEP->idx_begin();
 81:     } else if (NumIndices == 2) {
 82:       // If we have two indices, this should be an access through a pointer.
 83:       auto *IndexIt = GEP->idx_begin();
 84:       assert(cast<ConstantInt>(IndexIt)->getZExtValue() == 0 &&
 85:              "GEP is not indexing through pointer");
 86:       GEPOffset = *(++IndexIt);
 87:     } else
 88:       llvm_unreachable("Unhandled GEP structure for resource access");
 89:
 90:     uint64_t ElemSize = AccessSize;
 91:     if (!(IndexScale % ElemSize)) {
 92:       // If our scale is an exact multiple of the access size, adjust the
 93:       // scaling to avoid an unnecessary division.
 94:       IndexScale /= ElemSize;
 95:       ElemSize = 1;
 96:     }
 97:     if (IndexScale != 1)
 98:       GEPOffset = Builder.CreateMul(
 99:           GEPOffset, ConstantInt::get(Builder.getInt32Ty(), IndexScale));
100:     if (ElemSize != 1)
101:       GEPOffset = Builder.CreateUDiv(
102:           GEPOffset, ConstantInt::get(Builder.getInt32Ty(), ElemSize));
103:
104:     Offset = Offset ? Builder.CreateAdd(Offset, GEPOffset) : GEPOffset;
105:     Ptr = GEP->getPointerOperand();
106:   }
107:
108:   llvm_unreachable("GEP of null pointer?");
109: }
110:
111: static void createTypedBufferStore(IntrinsicInst *II, StoreInst *SI,
112:                                    dxil::ResourceTypeInfo &RTI) {
113:   const DataLayout &DL = SI->getDataLayout();
114:   IRBuilder<> Builder(SI);
115:   Type *ContainedType = RTI.getHandleTy()->getTypeParameter(0);
116:   Type *ScalarType = ContainedType->getScalarType();
117:   Type *LoadType = StructType::get(ContainedType, Builder.getInt1Ty());
118:
119:   Value *V = SI->getValueOperand();
120:   if (V->getType() == ContainedType) {
```
- EN: This range implements operational logic in helpers such as ConstantInt::get, assert, getNumIndices, getTypeAllocSize, translating backend policy into executable code.
- CN: 这一段实现了 ConstantInt::get、assert、getNumIndices、getTypeAllocSize 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:     // V is already the right type.
122:     assert(SI->getPointerOperand() == II &&
123:            "Store of whole element has mismatched address to store to");
124:   } else if (V->getType() == ScalarType) {
125:     // We're storing a scalar, so we need to load the current value and only
126:     // replace the relevant part.
127:     auto *Load = Builder.CreateIntrinsic(
128:         LoadType, Intrinsic::dx_resource_load_typedbuffer,
129:         {II->getOperand(0), II->getOperand(1)});
130:     auto *Struct = Builder.CreateExtractValue(Load, {0});
131:
132:     uint64_t AccessSize = DL.getTypeSizeInBits(ScalarType) / 8;
133:     Value *Offset =
134:         traverseGEPOffsets(DL, Builder, SI->getPointerOperand(), AccessSize);
135:     V = Builder.CreateInsertElement(Struct, V, Offset);
136:   } else {
137:     llvm_unreachable("Store to typed resource has invalid type");
138:   }
139:
140:   auto *Inst = Builder.CreateIntrinsic(
141:       Builder.getVoidTy(), Intrinsic::dx_resource_store_typedbuffer,
142:       {II->getOperand(0), II->getOperand(1), V});
143:   SI->replaceAllUsesWith(Inst);
144: }
145:
146: static void emitRawStore(IRBuilder<> &Builder, Value *Buffer, Value *Index,
147:                          Value *Offset, Value *V, dxil::ResourceTypeInfo &RTI) {
148:   // For raw buffer (ie, HLSL's ByteAddressBuffer), we need to fold the access
149:   // entirely into the index.
150:   if (!RTI.isStruct()) {
151:     auto *ConstantOffset = dyn_cast<ConstantInt>(Offset);
152:     if (!ConstantOffset || !ConstantOffset->isZero())
153:       Index = Builder.CreateAdd(Index, Offset);
154:     Offset = llvm::PoisonValue::get(Builder.getInt32Ty());
155:   }
156:
157:   Builder.CreateIntrinsic(Builder.getVoidTy(),
158:                           Intrinsic::dx_resource_store_rawbuffer,
159:                           {Buffer, Index, Offset, V});
160: }
161:
162: static void createRawStores(IntrinsicInst *II, StoreInst *SI,
163:                             dxil::ResourceTypeInfo &RTI) {
164:   const DataLayout &DL = SI->getDataLayout();
165:   IRBuilder<> Builder(SI);
166:
167:   Value *V = SI->getValueOperand();
168:   assert(!V->getType()->isAggregateType() &&
169:          "Resource store should be scalar or vector type");
170:
171:   Value *Index = II->getOperand(1);
172:   // The offset for the rawbuffer load and store ops is always in bytes.
173:   uint64_t AccessSize = 1;
174:   Value *Offset =
175:       traverseGEPOffsets(DL, Builder, SI->getPointerOperand(), AccessSize);
176:
177:   auto *VT = dyn_cast<FixedVectorType>(V->getType());
178:   if (VT && VT->getNumElements() > 4) {
179:     // Split into stores of at most 4 elements.
180:     Type *EltTy = VT->getElementType();
```
- EN: This range implements operational logic in helpers such as getOperand, CreateExtractValue, traverseGEPOffsets, CreateInsertElement, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、CreateExtractValue、traverseGEPOffsets、CreateInsertElement 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:     Value *Stride = ConstantInt::get(Builder.getInt32Ty(),
182:                                      4 * (DL.getTypeSizeInBits(EltTy) / 8));
183:
184:     SmallVector<int, 4> Indices;
185:     for (unsigned int I = 0, N = VT->getNumElements(); I < N; I += 4) {
186:       if (I > 0)
187:         Offset = Builder.CreateAdd(Offset, Stride);
188:
189:       for (unsigned int J = I, E = std::min(N, J + 4); J < E; ++J)
190:         Indices.push_back(J);
191:       Value *Part = Builder.CreateShuffleVector(V, Indices);
192:       emitRawStore(Builder, II->getOperand(0), Index, Offset, Part, RTI);
193:
194:       Indices.clear();
195:     }
196:   } else
197:     emitRawStore(Builder, II->getOperand(0), Index, Offset, V, RTI);
198: }
199:
200: static void createStoreIntrinsic(IntrinsicInst *II, StoreInst *SI,
201:                                  dxil::ResourceTypeInfo &RTI) {
202:   switch (RTI.getResourceKind()) {
203:   case dxil::ResourceKind::TypedBuffer:
204:     return createTypedBufferStore(II, SI, RTI);
205:   case dxil::ResourceKind::RawBuffer:
206:   case dxil::ResourceKind::StructuredBuffer:
207:     return createRawStores(II, SI, RTI);
208:   case dxil::ResourceKind::Texture1D:
209:   case dxil::ResourceKind::Texture2D:
210:   case dxil::ResourceKind::Texture2DMS:
211:   case dxil::ResourceKind::Texture3D:
212:   case dxil::ResourceKind::TextureCube:
213:   case dxil::ResourceKind::Texture1DArray:
214:   case dxil::ResourceKind::Texture2DArray:
215:   case dxil::ResourceKind::Texture2DMSArray:
216:   case dxil::ResourceKind::TextureCubeArray:
217:   case dxil::ResourceKind::FeedbackTexture2D:
218:   case dxil::ResourceKind::FeedbackTexture2DArray:
219:     reportFatalUsageError("DXIL Store not implemented for texture resources");
220:     return;
221:   case dxil::ResourceKind::CBuffer:
222:   case dxil::ResourceKind::Sampler:
223:   case dxil::ResourceKind::TBuffer:
224:   case dxil::ResourceKind::RTAccelerationStructure:
225:   case dxil::ResourceKind::Invalid:
226:   case dxil::ResourceKind::NumEntries:
227:     llvm_unreachable("Invalid resource kind for store");
228:   }
229:   llvm_unreachable("Unhandled case in switch");
230: }
231:
232: static void createTypedBufferLoad(IntrinsicInst *II, LoadInst *LI,
233:                                   dxil::ResourceTypeInfo &RTI) {
234:   const DataLayout &DL = LI->getDataLayout();
235:   IRBuilder<> Builder(LI);
236:   Type *ContainedType = RTI.getHandleTy()->getTypeParameter(0);
237:   Type *LoadType = StructType::get(ContainedType, Builder.getInt1Ty());
238:
239:   Value *V =
240:       Builder.CreateIntrinsic(LoadType, Intrinsic::dx_resource_load_typedbuffer,
```
- EN: This range implements operational logic in helpers such as getTypeSizeInBits, CreateAdd, push_back, CreateShuffleVector, translating backend policy into executable code.
- CN: 这一段实现了 getTypeSizeInBits、CreateAdd、push_back、CreateShuffleVector 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:                               {II->getOperand(0), II->getOperand(1)});
242:   V = Builder.CreateExtractValue(V, {0});
243:
244:   Type *ScalarType = ContainedType->getScalarType();
245:   uint64_t AccessSize = DL.getTypeSizeInBits(ScalarType) / 8;
246:   Value *Offset =
247:       traverseGEPOffsets(DL, Builder, LI->getPointerOperand(), AccessSize);
248:   auto *ConstantOffset = dyn_cast<ConstantInt>(Offset);
249:   if (!ConstantOffset || !ConstantOffset->isZero())
250:     V = Builder.CreateExtractElement(V, Offset);
251:
252:   // If we loaded a <1 x ...> instead of a scalar (presumably to feed a
253:   // shufflevector), then make sure we're maintaining the resulting type.
254:   if (auto *VT = dyn_cast<FixedVectorType>(LI->getType()))
255:     if (VT->getNumElements() == 1 && !isa<FixedVectorType>(V->getType()))
256:       V = Builder.CreateInsertElement(PoisonValue::get(VT), V,
257:                                       Builder.getInt32(0));
258:
259:   LI->replaceAllUsesWith(V);
260: }
261:
262: static void createTextureLoad(IntrinsicInst *II, LoadInst *LI,
263:                               dxil::ResourceTypeInfo &RTI) {
264:   const DataLayout &DL = LI->getDataLayout();
265:   IRBuilder<> Builder(LI);
266:   Type *ContainedType = RTI.getHandleTy()->getTypeParameter(0);
267:
268:   Value *Handle = II->getOperand(0);
269:   Value *Coords = II->getOperand(1);
270:
271:   // For operator[], mip level is 0.
272:   Value *MipLevel = Builder.getInt32(0);
273:
274:   // For operator[], offsets are zero.
275:   Type *CoordTy = Coords->getType();
276:   Type *OffsetTy;
277:   if (auto *VecTy = dyn_cast<FixedVectorType>(CoordTy))
278:     OffsetTy =
279:         FixedVectorType::get(Builder.getInt32Ty(), VecTy->getNumElements());
280:   else
281:     OffsetTy = Builder.getInt32Ty();
282:   Value *Offsets = Constant::getNullValue(OffsetTy);
283:
284:   Value *V =
285:       Builder.CreateIntrinsic(ContainedType, Intrinsic::dx_resource_load_level,
286:                               {Handle, Coords, MipLevel, Offsets});
287:
288:   Type *ScalarType = ContainedType->getScalarType();
289:   uint64_t AccessSize = DL.getTypeSizeInBits(ScalarType) / 8;
290:   Value *Offset =
291:       traverseGEPOffsets(DL, Builder, LI->getPointerOperand(), AccessSize);
292:   auto *ConstantOffset = dyn_cast<ConstantInt>(Offset);
293:   if (!ConstantOffset || !ConstantOffset->isZero())
294:     V = Builder.CreateExtractElement(V, Offset);
295:
296:   // If we loaded a <1 x ...> instead of a scalar (presumably to feed a
297:   // shufflevector), then make sure we're maintaining the resulting type.
298:   if (auto *VT = dyn_cast<FixedVectorType>(LI->getType()))
299:     if (VT->getNumElements() == 1 && !isa<FixedVectorType>(V->getType()))
300:       V = Builder.CreateInsertElement(PoisonValue::get(VT), V,
```
- EN: This range implements operational logic in helpers such as getOperand, CreateExtractValue, getScalarType, traverseGEPOffsets, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、CreateExtractValue、getScalarType、traverseGEPOffsets 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:                                       Builder.getInt32(0));
302:
303:   LI->replaceAllUsesWith(V);
304: }
305:
306: static Value *emitRawLoad(IRBuilder<> &Builder, Type *Ty, Value *Buffer,
307:                           Value *Index, Value *Offset,
308:                           dxil::ResourceTypeInfo &RTI) {
309:   // For raw buffer (ie, HLSL's ByteAddressBuffer), we need to fold the access
310:   // entirely into the index.
311:   if (!RTI.isStruct()) {
312:     auto *ConstantOffset = dyn_cast<ConstantInt>(Offset);
313:     if (!ConstantOffset || !ConstantOffset->isZero())
314:       Index = Builder.CreateAdd(Index, Offset);
315:     Offset = llvm::PoisonValue::get(Builder.getInt32Ty());
316:   }
317:
318:   // The load intrinsic includes the bit for CheckAccessFullyMapped, so we need
319:   // to add that to the return type.
320:   Type *TypeWithCheck = StructType::get(Ty, Builder.getInt1Ty());
321:   Value *V = Builder.CreateIntrinsic(TypeWithCheck,
322:                                      Intrinsic::dx_resource_load_rawbuffer,
323:                                      {Buffer, Index, Offset});
324:   return Builder.CreateExtractValue(V, {0});
325: }
326:
327: static void createRawLoads(IntrinsicInst *II, LoadInst *LI,
328:                            dxil::ResourceTypeInfo &RTI) {
329:   const DataLayout &DL = LI->getDataLayout();
330:   IRBuilder<> Builder(LI);
331:
332:   Value *Index = II->getOperand(1);
333:   // The offset for the rawbuffer load and store ops is always in bytes.
334:   uint64_t AccessSize = 1;
335:   Value *Offset =
336:       traverseGEPOffsets(DL, Builder, LI->getPointerOperand(), AccessSize);
337:
338:   // TODO: We could make this handle aggregates by walking the structure and
339:   // handling each field individually, but we don't ever generate code that
340:   // would hit that so it seems superfluous.
341:   assert(!LI->getType()->isAggregateType() &&
342:          "Resource load should be scalar or vector type");
343:
344:   Value *V;
345:   if (auto *VT = dyn_cast<FixedVectorType>(LI->getType())) {
346:     // Split into loads of at most 4 elements.
347:     Type *EltTy = VT->getElementType();
348:     Value *Stride = ConstantInt::get(Builder.getInt32Ty(),
349:                                      4 * (DL.getTypeSizeInBits(EltTy) / 8));
350:
351:     SmallVector<Value *> Parts;
352:     for (unsigned int I = 0, N = VT->getNumElements(); I < N; I += 4) {
353:       Type *Ty = FixedVectorType::get(EltTy, N - I < 4 ? N - I : 4);
354:       if (I > 0)
355:         Offset = Builder.CreateAdd(Offset, Stride);
356:       Parts.push_back(
357:           emitRawLoad(Builder, Ty, II->getOperand(0), Index, Offset, RTI));
358:     }
359:
360:     V = Parts.size() > 1 ? concatenateVectors(Builder, Parts) : Parts[0];
```
- EN: This range implements operational logic in helpers such as getInt32, replaceAllUsesWith, CreateAdd, llvm::PoisonValue::get, translating backend policy into executable code.
- CN: 这一段实现了 getInt32、replaceAllUsesWith、CreateAdd、llvm::PoisonValue::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   } else
362:     V = emitRawLoad(Builder, LI->getType(), II->getOperand(0), Index, Offset,
363:                     RTI);
364:
365:   LI->replaceAllUsesWith(V);
366: }
367:
368: namespace {
369: /// Helper for building a `load.cbufferrow` intrinsic given a simple type.
370: struct CBufferRowIntrin {
371:   Intrinsic::ID IID;
372:   Type *RetTy;
373:   unsigned int EltSize;
374:   unsigned int NumElts;
375:
376:   CBufferRowIntrin(const DataLayout &DL, Type *Ty) {
377:     assert(Ty == Ty->getScalarType() && "Expected scalar type");
378:
379:     switch (DL.getTypeSizeInBits(Ty)) {
380:     case 16:
381:       IID = Intrinsic::dx_resource_load_cbufferrow_8;
382:       RetTy = StructType::get(Ty, Ty, Ty, Ty, Ty, Ty, Ty, Ty);
383:       EltSize = 2;
384:       NumElts = 8;
385:       break;
386:     case 32:
387:       IID = Intrinsic::dx_resource_load_cbufferrow_4;
388:       RetTy = StructType::get(Ty, Ty, Ty, Ty);
389:       EltSize = 4;
390:       NumElts = 4;
391:       break;
392:     case 64:
393:       IID = Intrinsic::dx_resource_load_cbufferrow_2;
394:       RetTy = StructType::get(Ty, Ty);
395:       EltSize = 8;
396:       NumElts = 2;
397:       break;
398:     default:
399:       llvm_unreachable("Only 16, 32, and 64 bit types supported");
400:     }
401:   }
402: };
403: } // namespace
404:
405: static void createCBufferLoad(IntrinsicInst *II, LoadInst *LI,
406:                               dxil::ResourceTypeInfo &RTI) {
407:   const DataLayout &DL = LI->getDataLayout();
408:
409:   Type *Ty = LI->getType();
410:   assert(!isa<StructType>(Ty) && "Structs not handled yet");
411:   CBufferRowIntrin Intrin(DL, Ty->getScalarType());
412:
413:   StringRef Name = LI->getName();
414:   Value *Handle = II->getOperand(0);
415:
416:   IRBuilder<> Builder(LI);
417:
418:   ConstantInt *GlobalOffset = dyn_cast<ConstantInt>(II->getOperand(1));
419:   assert(GlobalOffset && "CBuffer getpointer index must be constant");
420:
```
- EN: This range defines or declares important types such as replaceAllUsesWith, CBufferRowIntrin, assert, StructType::get, shaping the data model used by DXILResourceAccess.cpp.
- CN: 这一段定义或声明了 replaceAllUsesWith、CBufferRowIntrin、assert、StructType::get 等关键类型，构成 DXILResourceAccess.cpp 使用的数据模型。

### Lines 421-480
```cpp
421:   uint64_t GlobalOffsetVal = GlobalOffset->getZExtValue();
422:   Value *CurrentRow = ConstantInt::get(
423:       Builder.getInt32Ty(), GlobalOffsetVal / hlsl::CBufferRowSizeInBytes);
424:   unsigned int CurrentIndex =
425:       (GlobalOffsetVal % hlsl::CBufferRowSizeInBytes) / Intrin.EltSize;
426:
427:   // Every object in a cbuffer either fits in a row or is aligned to a row. This
428:   // means that only the very last pointer access can point into a row.
429:   auto *LastGEP = dyn_cast<GEPOperator>(LI->getPointerOperand());
430:   if (!LastGEP) {
431:     // If we don't have a GEP at all we're just accessing the resource through
432:     // the result of getpointer directly.
433:     assert(LI->getPointerOperand() == II &&
434:            "Unexpected indirect access to resource without GEP");
435:   } else {
436:     Value *GEPOffset = traverseGEPOffsets(
437:         DL, Builder, LastGEP->getPointerOperand(), hlsl::CBufferRowSizeInBytes);
438:     CurrentRow = Builder.CreateAdd(GEPOffset, CurrentRow);
439:
440:     APInt ConstantOffset(DL.getIndexTypeSizeInBits(LastGEP->getType()), 0);
441:     if (LastGEP->accumulateConstantOffset(DL, ConstantOffset)) {
442:       APInt Remainder(DL.getIndexTypeSizeInBits(LastGEP->getType()),
443:                       hlsl::CBufferRowSizeInBytes);
444:       APInt::udivrem(ConstantOffset, Remainder, ConstantOffset, Remainder);
445:       CurrentRow = Builder.CreateAdd(
446:           CurrentRow, ConstantInt::get(Builder.getInt32Ty(), ConstantOffset));
447:       CurrentIndex += Remainder.udiv(Intrin.EltSize).getZExtValue();
448:     } else {
449:       assert(LastGEP->getNumIndices() == 1 &&
450:              "Last GEP of cbuffer access is not array or struct access");
451:       // We assume a non-constant access will be row-aligned. This is safe
452:       // because arrays and structs are always row aligned, and accesses to
453:       // vector elements will show up as a load of the vector followed by an
454:       // extractelement.
455:       CurrentRow = cast<ConstantInt>(CurrentRow)->isZero()
456:                        ? *LastGEP->idx_begin()
457:                        : Builder.CreateAdd(CurrentRow, *LastGEP->idx_begin());
458:       CurrentIndex = 0;
459:     }
460:   }
461:
462:   auto *CBufLoad = Builder.CreateIntrinsic(
463:       Intrin.RetTy, Intrin.IID, {Handle, CurrentRow}, nullptr, Name + ".load");
464:   auto *Elt =
465:       Builder.CreateExtractValue(CBufLoad, {CurrentIndex++}, Name + ".extract");
466:
467:   // At this point we've loaded the first scalar of our result, but our original
468:   // type may have been a vector.
469:   unsigned int Remaining =
470:       ((DL.getTypeSizeInBits(Ty) / 8) / Intrin.EltSize) - 1;
471:   if (Remaining == 0) {
472:     // We only have a single element, so we're done.
473:     Value *Result = Elt;
474:
475:     // However, if we loaded a <1 x T>, then we need to adjust the type.
476:     if (auto *VT = dyn_cast<FixedVectorType>(Ty)) {
477:       assert(VT->getNumElements() == 1 && "Can't have multiple elements here");
478:       Result = Builder.CreateInsertElement(PoisonValue::get(VT), Result,
479:                                            Builder.getInt32(0), Name);
480:     }
```
- EN: This range defines or declares important types such as getZExtValue, getInt32Ty, getPointerOperand, CreateAdd, shaping the data model used by DXILResourceAccess.cpp.
- CN: 这一段定义或声明了 getZExtValue、getInt32Ty、getPointerOperand、CreateAdd 等关键类型，构成 DXILResourceAccess.cpp 使用的数据模型。

### Lines 481-540
```cpp
481:     LI->replaceAllUsesWith(Result);
482:     return;
483:   }
484:
485:   // Walk each element and extract it, wrapping to new rows as needed.
486:   SmallVector<Value *> Extracts{Elt};
487:   while (Remaining--) {
488:     CurrentIndex %= Intrin.NumElts;
489:
490:     if (CurrentIndex == 0) {
491:       CurrentRow = Builder.CreateAdd(CurrentRow,
492:                                      ConstantInt::get(Builder.getInt32Ty(), 1));
493:       CBufLoad = Builder.CreateIntrinsic(Intrin.RetTy, Intrin.IID,
494:                                          {Handle, CurrentRow}, nullptr,
495:                                          Name + ".load");
496:     }
497:
498:     Extracts.push_back(Builder.CreateExtractValue(CBufLoad, {CurrentIndex++},
499:                                                   Name + ".extract"));
500:   }
501:
502:   // Finally, we build up the original loaded value.
503:   Value *Result = PoisonValue::get(Ty);
504:   for (int I = 0, E = Extracts.size(); I < E; ++I)
505:     Result = Builder.CreateInsertElement(
506:         Result, Extracts[I], Builder.getInt32(I), Name + formatv(".upto{}", I));
507:   LI->replaceAllUsesWith(Result);
508: }
509:
510: static void createLoadIntrinsic(IntrinsicInst *II, LoadInst *LI,
511:                                 dxil::ResourceTypeInfo &RTI) {
512:   switch (RTI.getResourceKind()) {
513:   case dxil::ResourceKind::TypedBuffer:
514:     return createTypedBufferLoad(II, LI, RTI);
515:   case dxil::ResourceKind::RawBuffer:
516:   case dxil::ResourceKind::StructuredBuffer:
517:     return createRawLoads(II, LI, RTI);
518:   case dxil::ResourceKind::CBuffer:
519:     return createCBufferLoad(II, LI, RTI);
520:   case dxil::ResourceKind::Texture1D:
521:   case dxil::ResourceKind::Texture2D:
522:   case dxil::ResourceKind::Texture2DMS:
523:   case dxil::ResourceKind::Texture3D:
524:   case dxil::ResourceKind::TextureCube:
525:   case dxil::ResourceKind::Texture1DArray:
526:   case dxil::ResourceKind::Texture2DArray:
527:   case dxil::ResourceKind::Texture2DMSArray:
528:   case dxil::ResourceKind::TextureCubeArray:
529:     return createTextureLoad(II, LI, RTI);
530:   case dxil::ResourceKind::FeedbackTexture2D:
531:   case dxil::ResourceKind::FeedbackTexture2DArray:
532:   case dxil::ResourceKind::TBuffer:
533:     reportFatalUsageError("Load not yet implemented for resource type");
534:     return;
535:   case dxil::ResourceKind::Sampler:
536:   case dxil::ResourceKind::RTAccelerationStructure:
537:   case dxil::ResourceKind::Invalid:
538:   case dxil::ResourceKind::NumEntries:
539:     llvm_unreachable("Invalid resource kind for load");
540:   }
```
- EN: This range implements operational logic in helpers such as replaceAllUsesWith, ConstantInt::get, PoisonValue::get, getInt32, translating backend policy into executable code.
- CN: 这一段实现了 replaceAllUsesWith、ConstantInt::get、PoisonValue::get、getInt32 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:   llvm_unreachable("Unhandled case in switch");
542: }
543:
544: static Instruction *getStoreLoadPointerOperand(Instruction *AI) {
545:   if (auto *LI = dyn_cast<LoadInst>(AI))
546:     return dyn_cast<Instruction>(LI->getPointerOperand());
547:   if (auto *SI = dyn_cast<StoreInst>(AI))
548:     return dyn_cast<Instruction>(SI->getPointerOperand());
549:
550:   return nullptr;
551: }
552:
553: static const std::array<Intrinsic::ID, 2> HandleIntrins = {
554:     Intrinsic::dx_resource_handlefrombinding,
555:     Intrinsic::dx_resource_handlefromimplicitbinding,
556: };
557:
558: static SmallVector<IntrinsicInst *> collectUsedHandles(Value *Ptr) {
559:   SmallVector<Value *> Worklist = {Ptr};
560:   SmallVector<IntrinsicInst *> Handles;
561:
562:   while (!Worklist.empty()) {
563:     Value *X = Worklist.pop_back_val();
564:
565:     if (!X->getType()->isPointerTy() && !X->getType()->isTargetExtTy())
566:       return {}; // Early exit on store/load into non-resource
567:
568:     if (auto *Phi = dyn_cast<PHINode>(X))
569:       for (Use &V : Phi->incoming_values())
570:         Worklist.push_back(V.get());
571:     else if (auto *Select = dyn_cast<SelectInst>(X))
572:       for (Value *V : {Select->getTrueValue(), Select->getFalseValue()})
573:         Worklist.push_back(V);
574:     else if (auto *II = dyn_cast<IntrinsicInst>(X)) {
575:       Intrinsic::ID IID = II->getIntrinsicID();
576:
577:       if (IID == Intrinsic::dx_resource_getpointer)
578:         Worklist.push_back(II->getArgOperand(/*Handle=*/0));
579:
580:       if (llvm::is_contained(HandleIntrins, IID))
581:         Handles.push_back(II);
582:     }
583:   }
584:
585:   return Handles;
586: }
587:
588: static hlsl::Binding getHandleIntrinsicBinding(IntrinsicInst *Handle,
589:                                                DXILResourceTypeMap &DRTM) {
590:   assert(llvm::is_contained(HandleIntrins, Handle->getIntrinsicID()) &&
591:          "Only expects a Handle as determined from collectUsedHandles.");
592:
593:   auto *HandleTy = cast<TargetExtType>(Handle->getType());
594:   dxil::ResourceClass Class = DRTM[HandleTy].getResourceClass();
595:   uint32_t Space = cast<ConstantInt>(Handle->getArgOperand(0))->getZExtValue();
596:   uint32_t LowerBound =
597:       cast<ConstantInt>(Handle->getArgOperand(1))->getZExtValue();
598:   uint32_t Size = cast<ConstantInt>(Handle->getArgOperand(2))->getZExtValue();
599:   uint32_t UpperBound = Size == UINT32_MAX ? UINT32_MAX : LowerBound + Size - 1;
600:
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, getStoreLoadPointerOperand, getPointerOperand, collectUsedHandles, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、getStoreLoadPointerOperand、getPointerOperand、collectUsedHandles 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-660
```cpp
601:   return hlsl::Binding(Class, Space, LowerBound, UpperBound, nullptr);
602: }
603:
604: namespace {
605: /// Helper for propagating the current handle and ptr indices.
606: struct AccessIndices {
607:   Value *GetPtrIdx;
608:   Value *HandleIdx;
609:
610:   bool hasGetPtrIdx() { return GetPtrIdx != nullptr; }
611:   bool hasHandleIdx() { return HandleIdx != nullptr; }
612: };
613: } // namespace
614:
615: // getAccessIndices traverses up the control flow that a ptr came from and
616: // propagates back the indicies used to access the resource (AccessIndices):
617: //
618: //  - GetPtrIdx is the index of dx.resource.getpointer
619: //  - HandleIdx is the index of dx.resource.handlefrom.*
620: static AccessIndices
621: getAccessIndices(Instruction *I, SmallSetVector<Instruction *, 16> &DeadInsts) {
622:   if (auto *II = dyn_cast<IntrinsicInst>(I)) {
623:     if (llvm::is_contained(HandleIntrins, II->getIntrinsicID())) {
624:       DeadInsts.insert(II);
625:       return {nullptr, II->getArgOperand(/*Index=*/3)};
626:     }
627:
628:     if (II->getIntrinsicID() == Intrinsic::dx_resource_getpointer) {
629:       auto *V = dyn_cast<Instruction>(II->getArgOperand(/*Handle=*/0));
630:       auto AccessIdx = getAccessIndices(V, DeadInsts);
631:       assert(!AccessIdx.hasGetPtrIdx() &&
632:              "Encountered multiple dx.resource.getpointers in ptr chain?");
633:       AccessIdx.GetPtrIdx = II->getArgOperand(1);
634:
635:       DeadInsts.insert(II);
636:       return AccessIdx;
637:     }
638:   }
639:
640:   if (auto *Phi = dyn_cast<PHINode>(I)) {
641:     unsigned NumEdges = Phi->getNumIncomingValues();
642:     assert(NumEdges != 0 && "Malformed Phi Node");
643:
644:     IRBuilder<> Builder(Phi);
645:     PHINode *GetPtrPhi = PHINode::Create(Builder.getInt32Ty(), NumEdges);
646:     PHINode *HandlePhi = PHINode::Create(Builder.getInt32Ty(), NumEdges);
647:
648:     bool HasGetPtr = true;
649:     for (unsigned Idx = 0; Idx < NumEdges; Idx++) {
650:       auto *BB = Phi->getIncomingBlock(Idx);
651:       auto *V = dyn_cast<Instruction>(Phi->getIncomingValue(Idx));
652:       auto AccessIdx = getAccessIndices(V, DeadInsts);
653:       HasGetPtr &= AccessIdx.hasGetPtrIdx();
654:       if (HasGetPtr)
655:         GetPtrPhi->addIncoming(AccessIdx.GetPtrIdx, BB);
656:       HandlePhi->addIncoming(AccessIdx.HandleIdx, BB);
657:     }
658:
659:     if (HasGetPtr)
660:       Builder.Insert(GetPtrPhi);
```
- EN: This range defines or declares important types such as hlsl::Binding, AccessIndices, hasGetPtrIdx, hasHandleIdx, shaping the data model used by DXILResourceAccess.cpp.
- CN: 这一段定义或声明了 hlsl::Binding、AccessIndices、hasGetPtrIdx、hasHandleIdx 等关键类型，构成 DXILResourceAccess.cpp 使用的数据模型。

### Lines 661-720
```cpp
661:     else
662:       GetPtrPhi = nullptr;
663:
664:     Builder.Insert(HandlePhi);
665:
666:     DeadInsts.insert(Phi);
667:     return {GetPtrPhi, HandlePhi};
668:   }
669:
670:   if (auto *Select = dyn_cast<SelectInst>(I)) {
671:     auto *TrueV = dyn_cast<Instruction>(Select->getTrueValue());
672:     auto TrueAccessIdx = getAccessIndices(TrueV, DeadInsts);
673:
674:     auto *FalseV = dyn_cast<Instruction>(Select->getFalseValue());
675:     auto FalseAccessIdx = getAccessIndices(FalseV, DeadInsts);
676:
677:     IRBuilder<> Builder(Select);
678:     Value *GetPtrSelect = nullptr;
679:
680:     if (TrueAccessIdx.hasGetPtrIdx() && FalseAccessIdx.hasGetPtrIdx())
681:       GetPtrSelect =
682:           Builder.CreateSelect(Select->getCondition(), TrueAccessIdx.GetPtrIdx,
683:                                FalseAccessIdx.GetPtrIdx);
684:
685:     auto *HandleSelect =
686:         Builder.CreateSelect(Select->getCondition(), TrueAccessIdx.HandleIdx,
687:                              FalseAccessIdx.HandleIdx);
688:     DeadInsts.insert(Select);
689:     return {GetPtrSelect, HandleSelect};
690:   }
691:
692:   llvm_unreachable("collectUsedHandles should assure this does not occur");
693: }
694:
695: static void
696: replaceHandleWithIndices(Instruction *Ptr, IntrinsicInst *OldHandle,
697:                          SmallSetVector<Instruction *, 16> &DeadInsts) {
698:   auto AccessIdx = getAccessIndices(Ptr, DeadInsts);
699:   assert(AccessIdx.hasGetPtrIdx() && AccessIdx.hasHandleIdx() &&
700:          "Couldn't retrieve indices. This is guaranteed by getAccessIndices");
701:
702:   IRBuilder<> Builder(Ptr);
703:   IntrinsicInst *Handle = cast<IntrinsicInst>(OldHandle->clone());
704:   Handle->setArgOperand(/*Index=*/3, AccessIdx.HandleIdx);
705:   Builder.Insert(Handle);
706:
707:   auto *GetPtr =
708:       Builder.CreateIntrinsic(Ptr->getType(), Intrinsic::dx_resource_getpointer,
709:                               {Handle, AccessIdx.GetPtrIdx});
710:
711:   Ptr->replaceAllUsesWith(GetPtr);
712:   DeadInsts.insert(Ptr);
713: }
714:
715: // Try to legalize dx.resource.handlefrom.*.binding and dx.resource.getpointer
716: // calls with their respective index values and propagate the index values to
717: // be used at resource access.
718: //
719: // If it can't be transformed to be legal then:
720: //
```
- EN: This range implements operational logic in helpers such as Insert, insert, getTrueValue, getAccessIndices, translating backend policy into executable code.
- CN: 这一段实现了 Insert、insert、getTrueValue、getAccessIndices 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-780
```cpp
721: // Reports an error if a resource access is not guaranteed into a unique global
722: // resource.
723: //
724: // Returns true if any changes are made.
725: static bool legalizeResourceHandles(Function &F, DXILResourceTypeMap &DRTM) {
726:   SmallSetVector<Instruction *, 16> DeadInsts;
727:   for (BasicBlock &BB : make_early_inc_range(F)) {
728:     for (Instruction &I : BB) {
729:       if (auto *PtrOp = getStoreLoadPointerOperand(&I)) {
730:         SmallVector<IntrinsicInst *> Handles = collectUsedHandles(PtrOp);
731:         unsigned NumHandles = Handles.size();
732:         if (NumHandles <= 1)
733:           continue; // Legal, no-replacement required
734:
735:         bool SameGlobalBinding = true;
736:         hlsl::Binding B = getHandleIntrinsicBinding(Handles[0], DRTM);
737:         for (unsigned Idx = 1; Idx < NumHandles; Idx++)
738:           SameGlobalBinding &=
739:               (B == getHandleIntrinsicBinding(Handles[Idx], DRTM));
740:
741:         if (!SameGlobalBinding) {
742:           diagnoseNonUniqueResourceAccess(&I, Handles);
743:           continue;
744:         }
745:
746:         replaceHandleWithIndices(PtrOp, Handles[0], DeadInsts);
747:       }
748:     }
749:   }
750:
751:   bool MadeChanges = false;
752:
753:   for (auto *I : llvm::reverse(DeadInsts))
754:     if (I->hasNUses(0)) { // Handle can still be used outside of replaced path
755:       I->eraseFromParent();
756:       MadeChanges = true;
757:     }
758:
759:   return MadeChanges;
760: }
761:
762: static void replaceAccess(IntrinsicInst *II, dxil::ResourceTypeInfo &RTI) {
763:   SmallVector<User *> Worklist;
764:   for (User *U : II->users())
765:     Worklist.push_back(U);
766:
767:   SmallVector<Instruction *> DeadInsts;
768:   while (!Worklist.empty()) {
769:     User *U = Worklist.back();
770:     Worklist.pop_back();
771:
772:     if (auto *GEP = dyn_cast<GetElementPtrInst>(U)) {
773:       for (User *U : GEP->users())
774:         Worklist.push_back(U);
775:       DeadInsts.push_back(GEP);
776:
777:     } else if (auto *SI = dyn_cast<StoreInst>(U)) {
778:       assert(SI->getValueOperand() != II && "Pointer escaped!");
779:       createStoreIntrinsic(II, SI, RTI);
780:       DeadInsts.push_back(SI);
```
- EN: This range implements operational logic in helpers such as legalizeResourceHandles, collectUsedHandles, size, getHandleIntrinsicBinding, translating backend policy into executable code.
- CN: 这一段实现了 legalizeResourceHandles、collectUsedHandles、size、getHandleIntrinsicBinding 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 781-840
```cpp
781:
782:     } else if (auto *LI = dyn_cast<LoadInst>(U)) {
783:       createLoadIntrinsic(II, LI, RTI);
784:       DeadInsts.push_back(LI);
785:     } else
786:       llvm_unreachable("Unhandled instruction - pointer escaped?");
787:   }
788:
789:   // Traverse the now-dead instructions in RPO and remove them.
790:   for (Instruction *Dead : llvm::reverse(DeadInsts))
791:     Dead->eraseFromParent();
792:   II->eraseFromParent();
793: }
794:
795: static bool transformResourcePointers(Function &F, DXILResourceTypeMap &DRTM) {
796:   SmallVector<std::pair<IntrinsicInst *, dxil::ResourceTypeInfo>> Resources;
797:   for (BasicBlock &BB : make_early_inc_range(F))
798:     for (Instruction &I : BB)
799:       if (auto *II = dyn_cast<IntrinsicInst>(&I))
800:         if (II->getIntrinsicID() == Intrinsic::dx_resource_getpointer) {
801:           auto *HandleTy = cast<TargetExtType>(II->getArgOperand(0)->getType());
802:           Resources.emplace_back(II, DRTM[HandleTy]);
803:         }
804:
805:   for (auto &[II, RI] : Resources)
806:     replaceAccess(II, RI);
807:
808:   return !Resources.empty();
809: }
810:
811: PreservedAnalyses DXILResourceAccess::run(Function &F,
812:                                           FunctionAnalysisManager &FAM) {
813:   auto &MAMProxy = FAM.getResult<ModuleAnalysisManagerFunctionProxy>(F);
814:   DXILResourceTypeMap *DRTM =
815:       MAMProxy.getCachedResult<DXILResourceTypeAnalysis>(*F.getParent());
816:   assert(DRTM && "DXILResourceTypeAnalysis must be available");
817:
818:   bool MadeHandleChanges = legalizeResourceHandles(F, *DRTM);
819:   bool MadeResourceChanges = transformResourcePointers(F, *DRTM);
820:   if (!(MadeHandleChanges || MadeResourceChanges))
821:     return PreservedAnalyses::all();
822:
823:   PreservedAnalyses PA;
824:   PA.preserve<DXILResourceTypeAnalysis>();
825:   PA.preserve<DominatorTreeAnalysis>();
826:   return PA;
827: }
828:
829: namespace {
830: class DXILResourceAccessLegacy : public FunctionPass {
831: public:
832:   bool runOnFunction(Function &F) override {
833:     DXILResourceTypeMap &DRTM =
834:         getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
835:     bool MadeHandleChanges = legalizeResourceHandles(F, DRTM);
836:     bool MadeResourceChanges = transformResourcePointers(F, DRTM);
837:     return MadeHandleChanges || MadeResourceChanges;
838:   }
839:   StringRef getPassName() const override { return "DXIL Resource Access"; }
840:   DXILResourceAccessLegacy() : FunctionPass(ID) {}
```
- EN: This range defines or declares important types such as createLoadIntrinsic, push_back, llvm_unreachable, eraseFromParent, shaping the data model used by DXILResourceAccess.cpp.
- CN: 这一段定义或声明了 createLoadIntrinsic、push_back、llvm_unreachable、eraseFromParent 等关键类型，构成 DXILResourceAccess.cpp 使用的数据模型。

### Lines 841-859
```cpp
841:
842:   static char ID; // Pass identification.
843:   void getAnalysisUsage(llvm::AnalysisUsage &AU) const override {
844:     AU.addRequired<DXILResourceTypeWrapperPass>();
845:     AU.addPreserved<DominatorTreeWrapperPass>();
846:   }
847: };
848: char DXILResourceAccessLegacy::ID = 0;
849: } // end anonymous namespace
850:
851: INITIALIZE_PASS_BEGIN(DXILResourceAccessLegacy, DEBUG_TYPE,
852:                       "DXIL Resource Access", false, false)
853: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
854: INITIALIZE_PASS_END(DXILResourceAccessLegacy, DEBUG_TYPE,
855:                     "DXIL Resource Access", false, false)
856:
857: FunctionPass *llvm::createDXILResourceAccessLegacyPass() {
858:   return new DXILResourceAccessLegacy();
859: }
```
- EN: This range implements operational logic in helpers such as getAnalysisUsage, INITIALIZE_PASS_DEPENDENCY, llvm::createDXILResourceAccessLegacyPass, DXILResourceAccessLegacy, translating backend policy into executable code.
- CN: 这一段实现了 getAnalysisUsage、INITIALIZE_PASS_DEPENDENCY、llvm::createDXILResourceAccessLegacyPass、DXILResourceAccessLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include getContext, InstOS, print, DiagnosticInfoGeneric, HandleOS, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getContext, InstOS, print, DiagnosticInfoGeneric, HandleOS，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILResourceAccess.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SetVector.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/Analysis/VectorUtils.h`
  - `llvm/Frontend/HLSL/HLSLResource.h`
  - `llvm/IR/BasicBlock.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/LLVMContext.h`
  - `llvm/IR/User.h`
  - `llvm/InitializePasses.h`
