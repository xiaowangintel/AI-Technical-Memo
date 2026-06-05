# DXILIntrinsicExpansion.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILIntrinsicExpansion.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains DXIL intrinsic expansions for those that don't have opcodes in DirectX Intermediate Language (DXIL).
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===- DXILIntrinsicExpansion.cpp - Prepare LLVM Module for DXIL encoding--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains DXIL intrinsic expansions for those that don't have
10: //  opcodes in DirectX Intermediate Language (DXIL).
11: //===----------------------------------------------------------------------===//
12:
13: #include "DXILIntrinsicExpansion.h"
14: #include "DirectX.h"
15: #include "llvm/ADT/STLExtras.h"
16: #include "llvm/ADT/SmallVector.h"
17: #include "llvm/CodeGen/Passes.h"
18: #include "llvm/IR/Constants.h"
19: #include "llvm/IR/IRBuilder.h"
20: #include "llvm/IR/InstrTypes.h"
21: #include "llvm/IR/Instruction.h"
22: #include "llvm/IR/Instructions.h"
23: #include "llvm/IR/Intrinsics.h"
24: #include "llvm/IR/IntrinsicsDirectX.h"
25: #include "llvm/IR/MatrixBuilder.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/IR/PassManager.h"
28: #include "llvm/IR/Type.h"
29: #include "llvm/Pass.h"
30: #include "llvm/Support/Casting.h"
31: #include "llvm/Support/ErrorHandling.h"
32: #include "llvm/Support/MathExtras.h"
33:
34: #define DEBUG_TYPE "dxil-intrinsic-expansion"
35:
36: using namespace llvm;
37:
38: class DXILIntrinsicExpansionLegacy : public ModulePass {
39:
40: public:
41:   bool runOnModule(Module &M) override;
42:   DXILIntrinsicExpansionLegacy() : ModulePass(ID) {}
43:
44:   static char ID; // Pass identification.
45: };
46:
47: static bool resourceAccessNeeds64BitExpansion(Module *M, Type *OverloadTy,
48:                                               bool IsRaw) {
49:   if (IsRaw && M->getTargetTriple().getDXILVersion() > VersionTuple(1, 2))
50:     return false;
51:
52:   Type *ScalarTy = OverloadTy->getScalarType();
53:   return ScalarTy->isDoubleTy() || ScalarTy->isIntegerTy(64);
54: }
55:
56: static Value *expand16BitIsInf(CallInst *Orig) {
57:   Module *M = Orig->getModule();
58:   if (M->getTargetTriple().getDXILVersion() >= VersionTuple(1, 9))
59:     return nullptr;
60:
61:   Value *Val = Orig->getOperand(0);
62:   Type *ValTy = Val->getType();
63:   if (!ValTy->getScalarType()->isHalfTy())
64:     return nullptr;
65:
66:   IRBuilder<> Builder(Orig);
67:   Type *IType = Type::getInt16Ty(M->getContext());
68:   Constant *PosInf =
69:       ValTy->isVectorTy()
70:           ? ConstantVector::getSplat(
71:                 ElementCount::getFixed(
72:                     cast<FixedVectorType>(ValTy)->getNumElements()),
73:                 ConstantInt::get(IType, 0x7c00))
74:           : ConstantInt::get(IType, 0x7c00);
75:
76:   Constant *NegInf =
77:       ValTy->isVectorTy()
78:           ? ConstantVector::getSplat(
79:                 ElementCount::getFixed(
80:                     cast<FixedVectorType>(ValTy)->getNumElements()),
81:                 ConstantInt::get(IType, 0xfc00))
82:           : ConstantInt::get(IType, 0xfc00);
83:
84:   Value *IVal = Builder.CreateBitCast(Val, PosInf->getType());
85:   Value *B1 = Builder.CreateICmpEQ(IVal, PosInf);
86:   Value *B2 = Builder.CreateICmpEQ(IVal, NegInf);
87:   Value *B3 = Builder.CreateOr(B1, B2);
88:   return B3;
89: }
90:
```
- EN: This range defines or declares important types such as DXILIntrinsicExpansionLegacy, runOnModule, getScalarType, isDoubleTy, shaping the data model used by DXILIntrinsicExpansion.cpp.
- CN: 这一段定义或声明了 DXILIntrinsicExpansionLegacy、runOnModule、getScalarType、isDoubleTy 等关键类型，构成 DXILIntrinsicExpansion.cpp 使用的数据模型。

### Lines 91-180
```cpp
 91: static Value *expand16BitIsNaN(CallInst *Orig) {
 92:   Module *M = Orig->getModule();
 93:   if (M->getTargetTriple().getDXILVersion() >= VersionTuple(1, 9))
 94:     return nullptr;
 95:
 96:   Value *Val = Orig->getOperand(0);
 97:   Type *ValTy = Val->getType();
 98:   if (!ValTy->getScalarType()->isHalfTy())
 99:     return nullptr;
100:
101:   IRBuilder<> Builder(Orig);
102:   Type *IType = Type::getInt16Ty(M->getContext());
103:
104:   Constant *ExpBitMask =
105:       ValTy->isVectorTy()
106:           ? ConstantVector::getSplat(
107:                 ElementCount::getFixed(
108:                     cast<FixedVectorType>(ValTy)->getNumElements()),
109:                 ConstantInt::get(IType, 0x7c00))
110:           : ConstantInt::get(IType, 0x7c00);
111:   Constant *SigBitMask =
112:       ValTy->isVectorTy()
113:           ? ConstantVector::getSplat(
114:                 ElementCount::getFixed(
115:                     cast<FixedVectorType>(ValTy)->getNumElements()),
116:                 ConstantInt::get(IType, 0x3ff))
117:           : ConstantInt::get(IType, 0x3ff);
118:
119:   Constant *Zero =
120:       ValTy->isVectorTy()
121:           ? ConstantVector::getSplat(
122:                 ElementCount::getFixed(
123:                     cast<FixedVectorType>(ValTy)->getNumElements()),
124:                 ConstantInt::get(IType, 0))
125:           : ConstantInt::get(IType, 0);
126:
127:   Value *IVal = Builder.CreateBitCast(Val, ExpBitMask->getType());
128:   Value *Exp = Builder.CreateAnd(IVal, ExpBitMask);
129:   Value *B1 = Builder.CreateICmpEQ(Exp, ExpBitMask);
130:
131:   Value *Sig = Builder.CreateAnd(IVal, SigBitMask);
132:   Value *B2 = Builder.CreateICmpNE(Sig, Zero);
133:   Value *B3 = Builder.CreateAnd(B1, B2);
134:   return B3;
135: }
136:
137: static Value *expand16BitIsFinite(CallInst *Orig) {
138:   Module *M = Orig->getModule();
139:   if (M->getTargetTriple().getDXILVersion() >= VersionTuple(1, 9))
140:     return nullptr;
141:
142:   Value *Val = Orig->getOperand(0);
143:   Type *ValTy = Val->getType();
144:   if (!ValTy->getScalarType()->isHalfTy())
145:     return nullptr;
146:
147:   IRBuilder<> Builder(Orig);
148:   Type *IType = Type::getInt16Ty(M->getContext());
149:
150:   Constant *ExpBitMask =
151:       ValTy->isVectorTy()
152:           ? ConstantVector::getSplat(
153:                 ElementCount::getFixed(
154:                     cast<FixedVectorType>(ValTy)->getNumElements()),
155:                 ConstantInt::get(IType, 0x7c00))
156:           : ConstantInt::get(IType, 0x7c00);
157:
158:   Value *IVal = Builder.CreateBitCast(Val, ExpBitMask->getType());
159:   Value *Exp = Builder.CreateAnd(IVal, ExpBitMask);
160:   Value *B1 = Builder.CreateICmpNE(Exp, ExpBitMask);
161:   return B1;
162: }
163:
164: static Value *expand16BitIsNormal(CallInst *Orig) {
165:   Module *M = Orig->getModule();
166:   if (M->getTargetTriple().getDXILVersion() >= VersionTuple(1, 9))
167:     return nullptr;
168:
169:   Value *Val = Orig->getOperand(0);
170:   Type *ValTy = Val->getType();
171:   if (!ValTy->getScalarType()->isHalfTy())
172:     return nullptr;
173:
174:   IRBuilder<> Builder(Orig);
175:   Type *IType = Type::getInt16Ty(M->getContext());
176:
177:   Constant *ExpBitMask =
178:       ValTy->isVectorTy()
179:           ? ConstantVector::getSplat(
180:                 ElementCount::getFixed(
```
- EN: This range implements operational logic in helpers such as expand16BitIsNaN, getModule, getOperand, getType, translating backend policy into executable code.
- CN: 这一段实现了 expand16BitIsNaN、getModule、getOperand、getType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:                     cast<FixedVectorType>(ValTy)->getNumElements()),
182:                 ConstantInt::get(IType, 0x7c00))
183:           : ConstantInt::get(IType, 0x7c00);
184:   Constant *Zero =
185:       ValTy->isVectorTy()
186:           ? ConstantVector::getSplat(
187:                 ElementCount::getFixed(
188:                     cast<FixedVectorType>(ValTy)->getNumElements()),
189:                 ConstantInt::get(IType, 0))
190:           : ConstantInt::get(IType, 0);
191:
192:   Value *IVal = Builder.CreateBitCast(Val, ExpBitMask->getType());
193:   Value *Exp = Builder.CreateAnd(IVal, ExpBitMask);
194:   Value *NotAllZeroes = Builder.CreateICmpNE(Exp, Zero);
195:   Value *NotAllOnes = Builder.CreateICmpNE(Exp, ExpBitMask);
196:   Value *B1 = Builder.CreateAnd(NotAllZeroes, NotAllOnes);
197:   return B1;
198: }
199:
200: static bool isIntrinsicExpansion(Function &F) {
201:   switch (F.getIntrinsicID()) {
202:   case Intrinsic::assume:
203:   case Intrinsic::abs:
204:   case Intrinsic::atan2:
205:   case Intrinsic::fshl:
206:   case Intrinsic::fshr:
207:   case Intrinsic::exp:
208:   case Intrinsic::is_fpclass:
209:   case Intrinsic::log:
210:   case Intrinsic::log10:
211:   case Intrinsic::pow:
212:   case Intrinsic::powi:
213:   case Intrinsic::dx_all:
214:   case Intrinsic::dx_any:
215:   case Intrinsic::dx_cross:
216:   case Intrinsic::dx_uclamp:
217:   case Intrinsic::dx_sclamp:
218:   case Intrinsic::dx_nclamp:
219:   case Intrinsic::dx_degrees:
220:   case Intrinsic::dx_isinf:
221:   case Intrinsic::dx_isnan:
222:   case Intrinsic::dx_lerp:
223:   case Intrinsic::dx_normalize:
224:   case Intrinsic::dx_fdot:
225:   case Intrinsic::dx_sdot:
226:   case Intrinsic::dx_udot:
227:   case Intrinsic::dx_sign:
228:   case Intrinsic::dx_step:
229:   case Intrinsic::dx_radians:
230:   case Intrinsic::usub_sat:
231:   case Intrinsic::vector_reduce_add:
232:   case Intrinsic::vector_reduce_fadd:
233:   case Intrinsic::matrix_multiply:
234:   case Intrinsic::matrix_transpose:
235:     return true;
236:   case Intrinsic::dx_resource_load_rawbuffer:
237:     return resourceAccessNeeds64BitExpansion(
238:         F.getParent(), F.getReturnType()->getStructElementType(0),
239:         /*IsRaw*/ true);
240:   case Intrinsic::dx_resource_load_typedbuffer:
241:     return resourceAccessNeeds64BitExpansion(
242:         F.getParent(), F.getReturnType()->getStructElementType(0),
243:         /*IsRaw*/ false);
244:   case Intrinsic::dx_resource_store_rawbuffer:
245:     return resourceAccessNeeds64BitExpansion(
246:         F.getParent(), F.getFunctionType()->getParamType(3), /*IsRaw*/ true);
247:   case Intrinsic::dx_resource_store_typedbuffer:
248:     return resourceAccessNeeds64BitExpansion(
249:         F.getParent(), F.getFunctionType()->getParamType(2), /*IsRaw*/ false);
250:   }
251:   return false;
252: }
253:
254: static Value *expandUsubSat(CallInst *Orig) {
255:   Value *A = Orig->getArgOperand(0);
256:   Value *B = Orig->getArgOperand(1);
257:   Type *Ty = A->getType();
258:
259:   IRBuilder<> Builder(Orig);
260:
261:   Value *Cmp = Builder.CreateICmpULT(A, B, "usub.cmp");
262:   Value *Sub = Builder.CreateSub(A, B, "usub.sub");
263:   Value *Zero = ConstantInt::get(Ty, 0);
264:   return Builder.CreateSelect(Cmp, Zero, Sub, "usub.sat");
265: }
266:
267: static Value *expandVecReduceAdd(CallInst *Orig, Intrinsic::ID IntrinsicId) {
268:   assert(IntrinsicId == Intrinsic::vector_reduce_add ||
269:          IntrinsicId == Intrinsic::vector_reduce_fadd);
270:
```
- EN: This range implements operational logic in helpers such as ConstantInt::get, isVectorTy, CreateBitCast, CreateAnd, translating backend policy into executable code.
- CN: 这一段实现了 ConstantInt::get、isVectorTy、CreateBitCast、CreateAnd 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:   IRBuilder<> Builder(Orig);
272:   bool IsFAdd = (IntrinsicId == Intrinsic::vector_reduce_fadd);
273:
274:   Value *X = Orig->getOperand(IsFAdd ? 1 : 0);
275:   Type *Ty = X->getType();
276:   auto *XVec = dyn_cast<FixedVectorType>(Ty);
277:   unsigned XVecSize = XVec->getNumElements();
278:   Value *Sum = Builder.CreateExtractElement(X, static_cast<uint64_t>(0));
279:
280:   // Handle the initial start value for floating-point addition.
281:   if (IsFAdd) {
282:     Constant *StartValue = dyn_cast<Constant>(Orig->getOperand(0));
283:     if (StartValue && !StartValue->isNullValue())
284:       Sum = Builder.CreateFAdd(Sum, StartValue);
285:   }
286:
287:   // Accumulate the remaining vector elements.
288:   for (unsigned I = 1; I < XVecSize; I++) {
289:     Value *Elt = Builder.CreateExtractElement(X, I);
290:     if (IsFAdd)
291:       Sum = Builder.CreateFAdd(Sum, Elt);
292:     else
293:       Sum = Builder.CreateAdd(Sum, Elt);
294:   }
295:
296:   return Sum;
297: }
298:
299: static Value *expandAbs(CallInst *Orig) {
300:   Value *X = Orig->getOperand(0);
301:   IRBuilder<> Builder(Orig);
302:   Type *Ty = X->getType();
303:   Type *EltTy = Ty->getScalarType();
304:   Constant *Zero = Ty->isVectorTy()
305:                        ? ConstantVector::getSplat(
306:                              ElementCount::getFixed(
307:                                  cast<FixedVectorType>(Ty)->getNumElements()),
308:                              ConstantInt::get(EltTy, 0))
309:                        : ConstantInt::get(EltTy, 0);
310:   auto *V = Builder.CreateSub(Zero, X);
311:   return Builder.CreateIntrinsic(Ty, Intrinsic::smax, {X, V}, nullptr,
312:                                  "dx.max");
313: }
314:
315: static Value *expandCrossIntrinsic(CallInst *Orig) {
316:
317:   VectorType *VT = cast<VectorType>(Orig->getType());
318:   if (cast<FixedVectorType>(VT)->getNumElements() != 3)
319:     reportFatalUsageError("return vector must have exactly 3 elements");
320:
321:   Value *op0 = Orig->getOperand(0);
322:   Value *op1 = Orig->getOperand(1);
323:   IRBuilder<> Builder(Orig);
324:
325:   Value *op0_x = Builder.CreateExtractElement(op0, (uint64_t)0, "x0");
326:   Value *op0_y = Builder.CreateExtractElement(op0, 1, "x1");
327:   Value *op0_z = Builder.CreateExtractElement(op0, 2, "x2");
328:
329:   Value *op1_x = Builder.CreateExtractElement(op1, (uint64_t)0, "y0");
330:   Value *op1_y = Builder.CreateExtractElement(op1, 1, "y1");
331:   Value *op1_z = Builder.CreateExtractElement(op1, 2, "y2");
332:
333:   auto MulSub = [&](Value *x0, Value *y0, Value *x1, Value *y1) -> Value * {
334:     Value *xy = Builder.CreateFMul(x0, y1);
335:     Value *yx = Builder.CreateFMul(y0, x1);
336:     return Builder.CreateFSub(xy, yx, Orig->getName());
337:   };
338:
339:   Value *yz_zy = MulSub(op0_y, op0_z, op1_y, op1_z);
340:   Value *zx_xz = MulSub(op0_z, op0_x, op1_z, op1_x);
341:   Value *xy_yx = MulSub(op0_x, op0_y, op1_x, op1_y);
342:
343:   Value *cross = PoisonValue::get(VT);
344:   cross = Builder.CreateInsertElement(cross, yz_zy, (uint64_t)0);
345:   cross = Builder.CreateInsertElement(cross, zx_xz, 1);
346:   cross = Builder.CreateInsertElement(cross, xy_yx, 2);
347:   return cross;
348: }
349:
350: // Create appropriate DXIL float dot intrinsic for the given A and B operands
351: // The appropriate opcode will be determined by the size of the operands
352: // The dot product is placed in the position indicated by Orig
353: static Value *expandFloatDotIntrinsic(CallInst *Orig, Value *A, Value *B) {
354:   Type *ATy = A->getType();
355:   [[maybe_unused]] Type *BTy = B->getType();
356:   assert(ATy->isVectorTy() && BTy->isVectorTy());
357:
358:   IRBuilder<> Builder(Orig);
359:
360:   auto *AVec = dyn_cast<FixedVectorType>(ATy);
```
- EN: This range implements operational logic in helpers such as Builder, getOperand, getType, getNumElements, translating backend policy into executable code.
- CN: 这一段实现了 Builder、getOperand、getType、getNumElements 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:
362:   assert(ATy->getScalarType()->isFloatingPointTy());
363:
364:   Intrinsic::ID DotIntrinsic = Intrinsic::dx_dot4;
365:   int NumElts = AVec->getNumElements();
366:   switch (NumElts) {
367:   case 2:
368:     DotIntrinsic = Intrinsic::dx_dot2;
369:     break;
370:   case 3:
371:     DotIntrinsic = Intrinsic::dx_dot3;
372:     break;
373:   case 4:
374:     DotIntrinsic = Intrinsic::dx_dot4;
375:     break;
376:   default:
377:     reportFatalUsageError(
378:         "Invalid dot product input vector: length is outside 2-4");
379:     return nullptr;
380:   }
381:
382:   SmallVector<Value *> Args;
383:   for (int I = 0; I < NumElts; ++I)
384:     Args.push_back(Builder.CreateExtractElement(A, Builder.getInt32(I)));
385:   for (int I = 0; I < NumElts; ++I)
386:     Args.push_back(Builder.CreateExtractElement(B, Builder.getInt32(I)));
387:   return Builder.CreateIntrinsic(ATy->getScalarType(), DotIntrinsic, Args,
388:                                  nullptr, "dot");
389: }
390:
391: // Create the appropriate DXIL float dot intrinsic for the operands of Orig
392: // The appropriate opcode will be determined by the size of the operands
393: // The dot product is placed in the position indicated by Orig
394: static Value *expandFloatDotIntrinsic(CallInst *Orig) {
395:   return expandFloatDotIntrinsic(Orig, Orig->getOperand(0),
396:                                  Orig->getOperand(1));
397: }
398:
399: // Expand integer dot product to multiply and add ops
400: static Value *expandIntegerDotIntrinsic(CallInst *Orig,
401:                                         Intrinsic::ID DotIntrinsic) {
402:   assert(DotIntrinsic == Intrinsic::dx_sdot ||
403:          DotIntrinsic == Intrinsic::dx_udot);
404:   Value *A = Orig->getOperand(0);
405:   Value *B = Orig->getOperand(1);
406:   Type *ATy = A->getType();
407:   [[maybe_unused]] Type *BTy = B->getType();
408:   assert(ATy->isVectorTy() && BTy->isVectorTy());
409:
410:   IRBuilder<> Builder(Orig);
411:
412:   auto *AVec = dyn_cast<FixedVectorType>(ATy);
413:
414:   assert(ATy->getScalarType()->isIntegerTy());
415:
416:   Value *Result;
417:   Intrinsic::ID MadIntrinsic = DotIntrinsic == Intrinsic::dx_sdot
418:                                    ? Intrinsic::dx_imad
419:                                    : Intrinsic::dx_umad;
420:   Value *Elt0 = Builder.CreateExtractElement(A, (uint64_t)0);
421:   Value *Elt1 = Builder.CreateExtractElement(B, (uint64_t)0);
422:   Result = Builder.CreateMul(Elt0, Elt1);
423:   for (unsigned I = 1; I < AVec->getNumElements(); I++) {
424:     Elt0 = Builder.CreateExtractElement(A, I);
425:     Elt1 = Builder.CreateExtractElement(B, I);
426:     Result = Builder.CreateIntrinsic(Result->getType(), MadIntrinsic,
427:                                      ArrayRef<Value *>{Elt0, Elt1, Result},
428:                                      nullptr, "dx.mad");
429:   }
430:   return Result;
431: }
432:
433: static Value *expandExpIntrinsic(CallInst *Orig) {
434:   Value *X = Orig->getOperand(0);
435:   IRBuilder<> Builder(Orig);
436:   Type *Ty = X->getType();
437:   Type *EltTy = Ty->getScalarType();
438:   Constant *Log2eConst =
439:       Ty->isVectorTy() ? ConstantVector::getSplat(
440:                              ElementCount::getFixed(
441:                                  cast<FixedVectorType>(Ty)->getNumElements()),
442:                              ConstantFP::get(EltTy, numbers::log2ef))
443:                        : ConstantFP::get(EltTy, numbers::log2ef);
444:   Value *NewX = Builder.CreateFMul(Log2eConst, X);
445:   auto *Exp2Call =
446:       Builder.CreateIntrinsic(Ty, Intrinsic::exp2, {NewX}, nullptr, "dx.exp2");
447:   Exp2Call->setTailCall(Orig->isTailCall());
448:   Exp2Call->setAttributes(Orig->getAttributes());
449:   return Exp2Call;
450: }
```
- EN: This range implements operational logic in helpers such as assert, getNumElements, push_back, expandFloatDotIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 assert、getNumElements、push_back、expandFloatDotIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:
452: static Value *expandIsFPClass(CallInst *Orig) {
453:   Value *T = Orig->getArgOperand(1);
454:   auto *TCI = dyn_cast<ConstantInt>(T);
455:
456:   // These FPClassTest cases have DXIL opcodes, so they will be handled in
457:   // DXIL Op Lowering instead for all non f16 cases.
458:   switch (TCI->getZExtValue()) {
459:   case FPClassTest::fcInf:
460:     return expand16BitIsInf(Orig);
461:   case FPClassTest::fcNan:
462:     return expand16BitIsNaN(Orig);
463:   case FPClassTest::fcNormal:
464:     return expand16BitIsNormal(Orig);
465:   case FPClassTest::fcFinite:
466:     return expand16BitIsFinite(Orig);
467:   }
468:
469:   IRBuilder<> Builder(Orig);
470:
471:   Value *F = Orig->getArgOperand(0);
472:   Type *FTy = F->getType();
473:   unsigned FNumElem = 0; // 0 => F is not a vector
474:
475:   unsigned BitWidth; // Bit width of F or the ElemTy of F
476:   Type *BitCastTy;   // An IntNTy of the same bitwidth as F or ElemTy of F
477:
478:   if (auto *FVecTy = dyn_cast<FixedVectorType>(FTy)) {
479:     Type *ElemTy = FVecTy->getElementType();
480:     FNumElem = FVecTy->getNumElements();
481:     BitWidth = ElemTy->getPrimitiveSizeInBits();
482:     BitCastTy = FixedVectorType::get(Builder.getIntNTy(BitWidth), FNumElem);
483:   } else {
484:     BitWidth = FTy->getPrimitiveSizeInBits();
485:     BitCastTy = Builder.getIntNTy(BitWidth);
486:   }
487:
488:   Value *FBitCast = Builder.CreateBitCast(F, BitCastTy);
489:   switch (TCI->getZExtValue()) {
490:   case FPClassTest::fcNegZero: {
491:     Value *NegZero =
492:         ConstantInt::get(Builder.getIntNTy(BitWidth), 1 << (BitWidth - 1),
493:                          /*IsSigned=*/true);
494:     Value *RetVal;
495:     if (FNumElem) {
496:       Value *NegZeroSplat = Builder.CreateVectorSplat(FNumElem, NegZero);
497:       RetVal =
498:           Builder.CreateICmpEQ(FBitCast, NegZeroSplat, "is.fpclass.negzero");
499:     } else
500:       RetVal = Builder.CreateICmpEQ(FBitCast, NegZero, "is.fpclass.negzero");
501:     return RetVal;
502:   }
503:   default:
504:     reportFatalUsageError("Unsupported FPClassTest");
505:   }
506: }
507:
508: static Value *expandAnyOrAllIntrinsic(CallInst *Orig,
509:                                       Intrinsic::ID IntrinsicId) {
510:   Value *X = Orig->getOperand(0);
511:   IRBuilder<> Builder(Orig);
512:   Type *Ty = X->getType();
513:   Type *EltTy = Ty->getScalarType();
514:
515:   auto ApplyOp = [&Builder](Intrinsic::ID IntrinsicId, Value *Result,
516:                             Value *Elt) {
517:     if (IntrinsicId == Intrinsic::dx_any)
518:       return Builder.CreateOr(Result, Elt);
519:     assert(IntrinsicId == Intrinsic::dx_all);
520:     return Builder.CreateAnd(Result, Elt);
521:   };
522:
523:   Value *Result = nullptr;
524:   if (!Ty->isVectorTy()) {
525:     Result = EltTy->isFloatingPointTy()
526:                  ? Builder.CreateFCmpUNE(X, ConstantFP::get(EltTy, 0))
527:                  : Builder.CreateICmpNE(X, ConstantInt::get(EltTy, 0));
528:   } else {
529:     auto *XVec = dyn_cast<FixedVectorType>(Ty);
530:     Value *Cond =
531:         EltTy->isFloatingPointTy()
532:             ? Builder.CreateFCmpUNE(
533:                   X, ConstantVector::getSplat(
534:                          ElementCount::getFixed(XVec->getNumElements()),
535:                          ConstantFP::get(EltTy, 0)))
536:             : Builder.CreateICmpNE(
537:                   X, ConstantVector::getSplat(
538:                          ElementCount::getFixed(XVec->getNumElements()),
539:                          ConstantInt::get(EltTy, 0)));
540:     Result = Builder.CreateExtractElement(Cond, (uint64_t)0);
```
- EN: This range implements operational logic in helpers such as expandIsFPClass, getArgOperand, expand16BitIsInf, expand16BitIsNaN, translating backend policy into executable code.
- CN: 这一段实现了 expandIsFPClass、getArgOperand、expand16BitIsInf、expand16BitIsNaN 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:     for (unsigned I = 1; I < XVec->getNumElements(); I++) {
542:       Value *Elt = Builder.CreateExtractElement(Cond, I);
543:       Result = ApplyOp(IntrinsicId, Result, Elt);
544:     }
545:   }
546:   return Result;
547: }
548:
549: static Value *expandLerpIntrinsic(CallInst *Orig) {
550:   Value *X = Orig->getOperand(0);
551:   Value *Y = Orig->getOperand(1);
552:   Value *S = Orig->getOperand(2);
553:   IRBuilder<> Builder(Orig);
554:   auto *V = Builder.CreateFSub(Y, X);
555:   V = Builder.CreateFMul(S, V);
556:   return Builder.CreateFAdd(X, V, "dx.lerp");
557: }
558:
559: static Value *expandLogIntrinsic(CallInst *Orig,
560:                                  float LogConstVal = numbers::ln2f) {
561:   Value *X = Orig->getOperand(0);
562:   IRBuilder<> Builder(Orig);
563:   Type *Ty = X->getType();
564:   Type *EltTy = Ty->getScalarType();
565:   Constant *Ln2Const =
566:       Ty->isVectorTy() ? ConstantVector::getSplat(
567:                              ElementCount::getFixed(
568:                                  cast<FixedVectorType>(Ty)->getNumElements()),
569:                              ConstantFP::get(EltTy, LogConstVal))
570:                        : ConstantFP::get(EltTy, LogConstVal);
571:   auto *Log2Call =
572:       Builder.CreateIntrinsic(Ty, Intrinsic::log2, {X}, nullptr, "elt.log2");
573:   Log2Call->setTailCall(Orig->isTailCall());
574:   Log2Call->setAttributes(Orig->getAttributes());
575:   return Builder.CreateFMul(Ln2Const, Log2Call);
576: }
577: static Value *expandLog10Intrinsic(CallInst *Orig) {
578:   return expandLogIntrinsic(Orig, numbers::ln2f / numbers::ln10f);
579: }
580:
581: // Use dot product of vector operand with itself to calculate the length.
582: // Divide the vector by that length to normalize it.
583: static Value *expandNormalizeIntrinsic(CallInst *Orig) {
584:   Value *X = Orig->getOperand(0);
585:   Type *Ty = Orig->getType();
586:   Type *EltTy = Ty->getScalarType();
587:   IRBuilder<> Builder(Orig);
588:
589:   auto *XVec = dyn_cast<FixedVectorType>(Ty);
590:   if (!XVec) {
591:     if (auto *constantFP = dyn_cast<ConstantFP>(X)) {
592:       const APFloat &fpVal = constantFP->getValueAPF();
593:       if (fpVal.isZero())
594:         reportFatalUsageError("Invalid input scalar: length is zero");
595:     }
596:     return Builder.CreateFDiv(X, X);
597:   }
598:
599:   Value *DotProduct = expandFloatDotIntrinsic(Orig, X, X);
600:
601:   // verify that the length is non-zero
602:   // (if the dot product is non-zero, then the length is non-zero)
603:   if (auto *constantFP = dyn_cast<ConstantFP>(DotProduct)) {
604:     const APFloat &fpVal = constantFP->getValueAPF();
605:     if (fpVal.isZero())
606:       reportFatalUsageError("Invalid input vector: length is zero");
607:   }
608:
609:   Value *Multiplicand = Builder.CreateIntrinsic(EltTy, Intrinsic::dx_rsqrt,
610:                                                 ArrayRef<Value *>{DotProduct},
611:                                                 nullptr, "dx.rsqrt");
612:
613:   Value *MultiplicandVec =
614:       Builder.CreateVectorSplat(XVec->getNumElements(), Multiplicand);
615:   return Builder.CreateFMul(X, MultiplicandVec);
616: }
617:
618: static Value *expandAtan2Intrinsic(CallInst *Orig) {
619:   Value *Y = Orig->getOperand(0);
620:   Value *X = Orig->getOperand(1);
621:   Type *Ty = X->getType();
622:   IRBuilder<> Builder(Orig);
623:   Builder.setFastMathFlags(Orig->getFastMathFlags());
624:
625:   Value *Tan = Builder.CreateFDiv(Y, X);
626:
627:   CallInst *Atan =
628:       Builder.CreateIntrinsic(Ty, Intrinsic::atan, {Tan}, nullptr, "Elt.Atan");
629:   Atan->setTailCall(Orig->isTailCall());
630:   Atan->setAttributes(Orig->getAttributes());
```
- EN: This range implements operational logic in helpers such as getNumElements, CreateExtractElement, ApplyOp, expandLerpIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 getNumElements、CreateExtractElement、ApplyOp、expandLerpIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:
632:   // Modify atan result based on https://en.wikipedia.org/wiki/Atan2.
633:   Constant *Pi = ConstantFP::get(Ty, llvm::numbers::pi);
634:   Constant *HalfPi = ConstantFP::get(Ty, llvm::numbers::pi / 2);
635:   Constant *NegHalfPi = ConstantFP::get(Ty, -llvm::numbers::pi / 2);
636:   Constant *Zero = ConstantFP::get(Ty, 0);
637:   Value *AtanAddPi = Builder.CreateFAdd(Atan, Pi);
638:   Value *AtanSubPi = Builder.CreateFSub(Atan, Pi);
639:
640:   // x > 0 -> atan.
641:   Value *Result = Atan;
642:   Value *XLt0 = Builder.CreateFCmpOLT(X, Zero);
643:   Value *XEq0 = Builder.CreateFCmpOEQ(X, Zero);
644:   Value *YGe0 = Builder.CreateFCmpOGE(Y, Zero);
645:   Value *YLt0 = Builder.CreateFCmpOLT(Y, Zero);
646:
647:   // x < 0, y >= 0 -> atan + pi.
648:   Value *XLt0AndYGe0 = Builder.CreateAnd(XLt0, YGe0);
649:   Result = Builder.CreateSelect(XLt0AndYGe0, AtanAddPi, Result);
650:
651:   // x < 0, y < 0 -> atan - pi.
652:   Value *XLt0AndYLt0 = Builder.CreateAnd(XLt0, YLt0);
653:   Result = Builder.CreateSelect(XLt0AndYLt0, AtanSubPi, Result);
654:
655:   // x == 0, y < 0 -> -pi/2
656:   Value *XEq0AndYLt0 = Builder.CreateAnd(XEq0, YLt0);
657:   Result = Builder.CreateSelect(XEq0AndYLt0, NegHalfPi, Result);
658:
659:   // x == 0, y > 0 -> pi/2
660:   Value *XEq0AndYGe0 = Builder.CreateAnd(XEq0, YGe0);
661:   Result = Builder.CreateSelect(XEq0AndYGe0, HalfPi, Result);
662:
663:   return Result;
664: }
665:
666: template <bool LeftFunnel>
667: static Value *expandFunnelShiftIntrinsic(CallInst *Orig) {
668:   Type *Ty = Orig->getType();
669:   Value *A = Orig->getOperand(0);
670:   Value *B = Orig->getOperand(1);
671:   Value *Shift = Orig->getOperand(2);
672:
673:   IRBuilder<> Builder(Orig);
674:
675:   unsigned BitWidth = Ty->getScalarSizeInBits();
676:   assert(llvm::isPowerOf2_32(BitWidth) &&
677:          "Can't use Mask to compute modulo and inverse");
678:
679:   // Note: if (Shift % BitWidth) == 0 then (BitWidth - Shift) == BitWidth,
680:   // shifting by the bitwidth for shl/lshr returns a poisoned result. As such,
681:   // we implement the same formula as LegalizerHelper::lowerFunnelShiftAsShifts.
682:   //
683:   // The funnel shift is expanded like so:
684:   // fshl
685:   //  -> msb_extract((concat(A, B) << (Shift % BitWidth)), BitWidth)
686:   //  -> A << (Shift % BitWidth) | B >> 1 >> (BitWidth - 1 - (Shift % BitWidth))
687:   // fshr
688:   //  -> lsb_extract((concat(A, B) >> (Shift % BitWidth), BitWidth))
689:   //  -> A << 1 << (BitWidth - 1 - (Shift % BitWidth)) | B >> (Shift % BitWidth)
690:
691:   // (BitWidth - 1) -> Mask
692:   Constant *Mask = ConstantInt::get(Ty, Ty->getScalarSizeInBits() - 1);
693:
694:   // Shift % BitWidth
695:   //  -> Shift & (BitWidth - 1)
696:   //  -> Shift & Mask
697:   Value *MaskedShift = Builder.CreateAnd(Shift, Mask);
698:
699:   // (BitWidth - 1) - (Shift % BitWidth)
700:   //  -> ~Shift & (BitWidth - 1)
701:   //  -> ~Shift & Mask
702:   Value *NotShift = Builder.CreateNot(Shift);
703:   Value *InverseShift = Builder.CreateAnd(NotShift, Mask);
704:
705:   Constant *One = ConstantInt::get(Ty, 1);
706:   Value *ShiftedA;
707:   Value *ShiftedB;
708:
709:   if (LeftFunnel) {
710:     ShiftedA = Builder.CreateShl(A, MaskedShift);
711:     Value *ShiftB1 = Builder.CreateLShr(B, One);
712:     ShiftedB = Builder.CreateLShr(ShiftB1, InverseShift);
713:   } else {
714:     Value *ShiftA1 = Builder.CreateShl(A, One);
715:     ShiftedA = Builder.CreateShl(ShiftA1, InverseShift);
716:     ShiftedB = Builder.CreateLShr(B, MaskedShift);
717:   }
718:
719:   Value *Result = Builder.CreateOr(ShiftedA, ShiftedB);
720:   return Result;
```
- EN: This range implements operational logic in helpers such as ConstantFP::get, CreateFAdd, CreateFSub, CreateFCmpOLT, translating backend policy into executable code.
- CN: 这一段实现了 ConstantFP::get、CreateFAdd、CreateFSub、CreateFCmpOLT 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721: }
722:
723: static Value *expandPowIntrinsic(CallInst *Orig, Intrinsic::ID IntrinsicId) {
724:
725:   Value *X = Orig->getOperand(0);
726:   Value *Y = Orig->getOperand(1);
727:   Type *Ty = X->getType();
728:   IRBuilder<> Builder(Orig);
729:
730:   if (IntrinsicId == Intrinsic::powi)
731:     Y = Builder.CreateSIToFP(Y, Ty);
732:
733:   auto *Log2Call =
734:       Builder.CreateIntrinsic(Ty, Intrinsic::log2, {X}, nullptr, "elt.log2");
735:   auto *Mul = Builder.CreateFMul(Log2Call, Y);
736:   auto *Exp2Call =
737:       Builder.CreateIntrinsic(Ty, Intrinsic::exp2, {Mul}, nullptr, "elt.exp2");
738:   Exp2Call->setTailCall(Orig->isTailCall());
739:   Exp2Call->setAttributes(Orig->getAttributes());
740:   return Exp2Call;
741: }
742:
743: static Value *expandStepIntrinsic(CallInst *Orig) {
744:
745:   Value *X = Orig->getOperand(0);
746:   Value *Y = Orig->getOperand(1);
747:   Type *Ty = X->getType();
748:   IRBuilder<> Builder(Orig);
749:
750:   Constant *One = ConstantFP::get(Ty->getScalarType(), 1.0);
751:   Constant *Zero = ConstantFP::get(Ty->getScalarType(), 0.0);
752:   Value *Cond = Builder.CreateFCmpOLT(Y, X);
753:
754:   if (Ty != Ty->getScalarType()) {
755:     auto *XVec = dyn_cast<FixedVectorType>(Ty);
756:     One = ConstantVector::getSplat(
757:         ElementCount::getFixed(XVec->getNumElements()), One);
758:     Zero = ConstantVector::getSplat(
759:         ElementCount::getFixed(XVec->getNumElements()), Zero);
760:   }
761:
762:   return Builder.CreateSelect(Cond, Zero, One);
763: }
764:
765: static Value *expandRadiansIntrinsic(CallInst *Orig) {
766:   Value *X = Orig->getOperand(0);
767:   Type *Ty = X->getType();
768:   IRBuilder<> Builder(Orig);
769:   Value *PiOver180 = ConstantFP::get(Ty, llvm::numbers::pi / 180.0);
770:   return Builder.CreateFMul(X, PiOver180);
771: }
772:
773: static bool expandBufferLoadIntrinsic(CallInst *Orig, bool IsRaw) {
774:   IRBuilder<> Builder(Orig);
775:
776:   Type *BufferTy = Orig->getType()->getStructElementType(0);
777:   Type *ScalarTy = BufferTy->getScalarType();
778:   bool IsDouble = ScalarTy->isDoubleTy();
779:   assert(IsDouble || ScalarTy->isIntegerTy(64) &&
780:                          "Only expand double or int64 scalars or vectors");
781:   bool IsVector = false;
782:   unsigned ExtractNum = 2;
783:   if (auto *VT = dyn_cast<FixedVectorType>(BufferTy)) {
784:     ExtractNum = 2 * VT->getNumElements();
785:     IsVector = true;
786:     assert(IsRaw || ExtractNum == 4 && "TypedBufferLoad vector must be size 2");
787:   }
788:
789:   SmallVector<Value *, 2> Loads;
790:   Value *Result = PoisonValue::get(BufferTy);
791:   unsigned Base = 0;
792:   // If we need to extract more than 4 i32; we need to break it up into
793:   // more than one load. LoadNum tells us how many i32s we are loading in
794:   // each load
795:   while (ExtractNum > 0) {
796:     unsigned LoadNum = std::min(ExtractNum, 4u);
797:     Type *Ty = VectorType::get(Builder.getInt32Ty(), LoadNum, false);
798:
799:     Type *LoadType = StructType::get(Ty, Builder.getInt1Ty());
800:     Intrinsic::ID LoadIntrinsic = Intrinsic::dx_resource_load_typedbuffer;
801:     SmallVector<Value *, 3> Args = {Orig->getOperand(0), Orig->getOperand(1)};
802:     if (IsRaw) {
803:       LoadIntrinsic = Intrinsic::dx_resource_load_rawbuffer;
804:       Value *Tmp = Builder.getInt32(4 * Base * 2);
805:       Args.push_back(Builder.CreateAdd(Orig->getOperand(2), Tmp));
806:     }
807:
808:     CallInst *Load = Builder.CreateIntrinsic(LoadType, LoadIntrinsic, Args);
809:     Loads.push_back(Load);
810:
```
- EN: This range implements operational logic in helpers such as expandPowIntrinsic, getOperand, getType, Builder, translating backend policy into executable code.
- CN: 这一段实现了 expandPowIntrinsic、getOperand、getType、Builder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:     // extract the buffer load's result
812:     Value *Extract = Builder.CreateExtractValue(Load, {0});
813:
814:     SmallVector<Value *> ExtractElements;
815:     for (unsigned I = 0; I < LoadNum; ++I)
816:       ExtractElements.push_back(
817:           Builder.CreateExtractElement(Extract, Builder.getInt32(I)));
818:
819:     // combine into double(s) or int64(s)
820:     for (unsigned I = 0; I < LoadNum; I += 2) {
821:       Value *Combined = nullptr;
822:       if (IsDouble)
823:         // For doubles, use dx_asdouble intrinsic
824:         Combined = Builder.CreateIntrinsic(
825:             Builder.getDoubleTy(), Intrinsic::dx_asdouble,
826:             {ExtractElements[I], ExtractElements[I + 1]});
827:       else {
828:         // For int64, manually combine two int32s
829:         // First, zero-extend both values to i64
830:         Value *Lo =
831:             Builder.CreateZExt(ExtractElements[I], Builder.getInt64Ty());
832:         Value *Hi =
833:             Builder.CreateZExt(ExtractElements[I + 1], Builder.getInt64Ty());
834:         // Shift the high bits left by 32 bits
835:         Value *ShiftedHi = Builder.CreateShl(Hi, Builder.getInt64(32));
836:         // OR the high and low bits together
837:         Combined = Builder.CreateOr(Lo, ShiftedHi);
838:       }
839:
840:       if (IsVector)
841:         Result = Builder.CreateInsertElement(Result, Combined,
842:                                              Builder.getInt32((I / 2) + Base));
843:       else
844:         Result = Combined;
845:     }
846:
847:     ExtractNum -= LoadNum;
848:     Base += LoadNum / 2;
849:   }
850:
851:   Value *CheckBit = nullptr;
852:   for (User *U : make_early_inc_range(Orig->users())) {
853:     // If it's not a ExtractValueInst, we don't know how to
854:     // handle it
855:     auto *EVI = dyn_cast<ExtractValueInst>(U);
856:     if (!EVI)
857:       llvm_unreachable("Unexpected user of typedbufferload");
858:
859:     ArrayRef<unsigned> Indices = EVI->getIndices();
860:     assert(Indices.size() == 1);
861:
862:     if (Indices[0] == 0) {
863:       // Use of the value(s)
864:       EVI->replaceAllUsesWith(Result);
865:     } else {
866:       // Use of the check bit
867:       assert(Indices[0] == 1 && "Unexpected type for typedbufferload");
868:       // Note: This does not always match the historical behaviour of DXC.
869:       // See https://github.com/microsoft/DirectXShaderCompiler/issues/7622
870:       if (!CheckBit) {
871:         SmallVector<Value *, 2> CheckBits;
872:         for (Value *L : Loads)
873:           CheckBits.push_back(Builder.CreateExtractValue(L, {1}));
874:         CheckBit = Builder.CreateAnd(CheckBits);
875:       }
876:       EVI->replaceAllUsesWith(CheckBit);
877:     }
878:     EVI->eraseFromParent();
879:   }
880:   Orig->eraseFromParent();
881:   return true;
882: }
883:
884: static bool expandBufferStoreIntrinsic(CallInst *Orig, bool IsRaw) {
885:   IRBuilder<> Builder(Orig);
886:
887:   unsigned ValIndex = IsRaw ? 3 : 2;
888:   Type *BufferTy = Orig->getFunctionType()->getParamType(ValIndex);
889:   Type *ScalarTy = BufferTy->getScalarType();
890:   bool IsDouble = ScalarTy->isDoubleTy();
891:   assert((IsDouble || ScalarTy->isIntegerTy(64)) &&
892:          "Only expand double or int64 scalars or vectors");
893:
894:   // Determine if we're dealing with a vector or scalar
895:   bool IsVector = false;
896:   unsigned ExtractNum = 2;
897:   unsigned VecLen = 0;
898:   if (auto *VT = dyn_cast<FixedVectorType>(BufferTy)) {
899:     VecLen = VT->getNumElements();
900:     assert(IsRaw || VecLen == 2 && "TypedBufferStore vector must be size 2");
```
- EN: This range implements operational logic in helpers such as CreateExtractValue, CreateExtractElement, CreateZExt, CreateShl, translating backend policy into executable code.
- CN: 这一段实现了 CreateExtractValue、CreateExtractElement、CreateZExt、CreateShl 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:     ExtractNum = VecLen * 2;
902:     IsVector = true;
903:   }
904:
905:   // Create the appropriate vector type for the result
906:   Type *Int32Ty = Builder.getInt32Ty();
907:   Type *ResultTy = VectorType::get(Int32Ty, ExtractNum, false);
908:   Value *Val = PoisonValue::get(ResultTy);
909:
910:   Type *SplitElementTy = Int32Ty;
911:   if (IsVector)
912:     SplitElementTy = VectorType::get(SplitElementTy, VecLen, false);
913:
914:   Value *LowBits = nullptr;
915:   Value *HighBits = nullptr;
916:   // Split the 64-bit values into 32-bit components
917:   if (IsDouble) {
918:     auto *SplitTy = llvm::StructType::get(SplitElementTy, SplitElementTy);
919:     Value *Split = Builder.CreateIntrinsic(SplitTy, Intrinsic::dx_splitdouble,
920:                                            {Orig->getOperand(ValIndex)});
921:     LowBits = Builder.CreateExtractValue(Split, 0);
922:     HighBits = Builder.CreateExtractValue(Split, 1);
923:   } else {
924:     // Handle int64 type(s)
925:     Value *InputVal = Orig->getOperand(ValIndex);
926:     Constant *ShiftAmt = Builder.getInt64(32);
927:     if (IsVector)
928:       ShiftAmt =
929:           ConstantVector::getSplat(ElementCount::getFixed(VecLen), ShiftAmt);
930:
931:     // Split into low and high 32-bit parts
932:     LowBits = Builder.CreateTrunc(InputVal, SplitElementTy);
933:     Value *ShiftedVal = Builder.CreateLShr(InputVal, ShiftAmt);
934:     HighBits = Builder.CreateTrunc(ShiftedVal, SplitElementTy);
935:   }
936:
937:   if (IsVector) {
938:     SmallVector<int, 8> Mask;
939:     for (unsigned I = 0; I < VecLen; ++I) {
940:       Mask.push_back(I);
941:       Mask.push_back(I + VecLen);
942:     }
943:     Val = Builder.CreateShuffleVector(LowBits, HighBits, Mask);
944:   } else {
945:     Val = Builder.CreateInsertElement(Val, LowBits, Builder.getInt32(0));
946:     Val = Builder.CreateInsertElement(Val, HighBits, Builder.getInt32(1));
947:   }
948:
949:   // If we need to extract more than 4 i32; we need to break it up into
950:   // more than one store. StoreNum tells us how many i32s we are storing in
951:   // each store
952:   unsigned Base = 0;
953:   while (ExtractNum > 0) {
954:     unsigned StoreNum = std::min(ExtractNum, 4u);
955:
956:     Intrinsic::ID StoreIntrinsic = Intrinsic::dx_resource_store_typedbuffer;
957:     SmallVector<Value *, 4> Args = {Orig->getOperand(0), Orig->getOperand(1)};
958:     if (IsRaw) {
959:       StoreIntrinsic = Intrinsic::dx_resource_store_rawbuffer;
960:       Value *Tmp = Builder.getInt32(4 * Base);
961:       Args.push_back(Builder.CreateAdd(Orig->getOperand(2), Tmp));
962:     }
963:
964:     SmallVector<int, 4> Mask;
965:     for (unsigned I = 0; I < StoreNum; ++I) {
966:       Mask.push_back(Base + I);
967:     }
968:
969:     Value *SubVal = Val;
970:     if (VecLen > 2)
971:       SubVal = Builder.CreateShuffleVector(Val, Mask);
972:
973:     Args.push_back(SubVal);
974:     // Create the final intrinsic call
975:     Builder.CreateIntrinsic(Builder.getVoidTy(), StoreIntrinsic, Args);
976:
977:     ExtractNum -= StoreNum;
978:     Base += StoreNum;
979:   }
980:   Orig->eraseFromParent();
981:   return true;
982: }
983:
984: static Intrinsic::ID getMaxForClamp(Intrinsic::ID ClampIntrinsic) {
985:   if (ClampIntrinsic == Intrinsic::dx_uclamp)
986:     return Intrinsic::umax;
987:   if (ClampIntrinsic == Intrinsic::dx_sclamp)
988:     return Intrinsic::smax;
989:   assert(ClampIntrinsic == Intrinsic::dx_nclamp);
990:   return Intrinsic::maxnum;
```
- EN: This range implements operational logic in helpers such as getInt32Ty, VectorType::get, PoisonValue::get, llvm::StructType::get, translating backend policy into executable code.
- CN: 这一段实现了 getInt32Ty、VectorType::get、PoisonValue::get、llvm::StructType::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991: }
 992:
 993: static Intrinsic::ID getMinForClamp(Intrinsic::ID ClampIntrinsic) {
 994:   if (ClampIntrinsic == Intrinsic::dx_uclamp)
 995:     return Intrinsic::umin;
 996:   if (ClampIntrinsic == Intrinsic::dx_sclamp)
 997:     return Intrinsic::smin;
 998:   assert(ClampIntrinsic == Intrinsic::dx_nclamp);
 999:   return Intrinsic::minnum;
1000: }
1001:
1002: static Value *expandClampIntrinsic(CallInst *Orig,
1003:                                    Intrinsic::ID ClampIntrinsic) {
1004:   Value *X = Orig->getOperand(0);
1005:   Value *Min = Orig->getOperand(1);
1006:   Value *Max = Orig->getOperand(2);
1007:   Type *Ty = X->getType();
1008:   IRBuilder<> Builder(Orig);
1009:   auto *MaxCall = Builder.CreateIntrinsic(Ty, getMaxForClamp(ClampIntrinsic),
1010:                                           {X, Min}, nullptr, "dx.max");
1011:   return Builder.CreateIntrinsic(Ty, getMinForClamp(ClampIntrinsic),
1012:                                  {MaxCall, Max}, nullptr, "dx.min");
1013: }
1014:
1015: static Value *expandDegreesIntrinsic(CallInst *Orig) {
1016:   Value *X = Orig->getOperand(0);
1017:   Type *Ty = X->getType();
1018:   IRBuilder<> Builder(Orig);
1019:   Value *DegreesRatio = ConstantFP::get(Ty, 180.0 * llvm::numbers::inv_pi);
1020:   return Builder.CreateFMul(X, DegreesRatio);
1021: }
1022:
1023: static Value *expandSignIntrinsic(CallInst *Orig) {
1024:   Value *X = Orig->getOperand(0);
1025:   Type *Ty = X->getType();
1026:   Type *ScalarTy = Ty->getScalarType();
1027:   Type *RetTy = Orig->getType();
1028:   Constant *Zero = Constant::getNullValue(Ty);
1029:
1030:   IRBuilder<> Builder(Orig);
1031:
1032:   Value *GT;
1033:   Value *LT;
1034:   if (ScalarTy->isFloatingPointTy()) {
1035:     GT = Builder.CreateFCmpOLT(Zero, X);
1036:     LT = Builder.CreateFCmpOLT(X, Zero);
1037:   } else {
1038:     assert(ScalarTy->isIntegerTy());
1039:     GT = Builder.CreateICmpSLT(Zero, X);
1040:     LT = Builder.CreateICmpSLT(X, Zero);
1041:   }
1042:
1043:   Value *ZextGT = Builder.CreateZExt(GT, RetTy);
1044:   Value *ZextLT = Builder.CreateZExt(LT, RetTy);
1045:
1046:   return Builder.CreateSub(ZextGT, ZextLT);
1047: }
1048:
1049: // Expand llvm.matrix.multiply by extracting row/column vectors and computing
1050: // dot products.
1051: // Result[r,c] = dot(row_r(LHS), col_c(RHS))
1052: // Element (r,c) is at index c*NumRows + r (column-major).
1053: static Value *expandMatrixMultiply(CallInst *Orig) {
1054:   Value *LHS = Orig->getArgOperand(0);
1055:   Value *RHS = Orig->getArgOperand(1);
1056:   unsigned LHSRows = cast<ConstantInt>(Orig->getArgOperand(2))->getZExtValue();
1057:   unsigned LHSCols = cast<ConstantInt>(Orig->getArgOperand(3))->getZExtValue();
1058:   unsigned RHSCols = cast<ConstantInt>(Orig->getArgOperand(4))->getZExtValue();
1059:
1060:   auto *RetTy = cast<FixedVectorType>(Orig->getType());
1061:   Type *EltTy = RetTy->getElementType();
1062:   bool IsFP = EltTy->isFloatingPointTy();
1063:
1064:   IRBuilder<> Builder(Orig);
1065:
1066:   // Column-major indexing:
1067:   // LHS row R, element K: index = K * LHSRows + R
1068:   // RHS col C, element K: index = C * LHSCols + K
1069:   Value *Result = PoisonValue::get(RetTy);
1070:
1071:   // Extract all scalar elements from LHS and RHS once, then reuse them.
1072:   unsigned LHSSize = LHSRows * LHSCols;
1073:   unsigned RHSSize = LHSCols * RHSCols;
1074:   SmallVector<Value *, 16> LHSElts(LHSSize);
1075:   SmallVector<Value *, 16> RHSElts(RHSSize);
1076:   for (unsigned I = 0; I < LHSSize; ++I)
1077:     LHSElts[I] = Builder.CreateExtractElement(LHS, I);
1078:   for (unsigned I = 0; I < RHSSize; ++I)
1079:     RHSElts[I] = Builder.CreateExtractElement(RHS, I);
1080:
```
- EN: This range implements operational logic in helpers such as getMinForClamp, assert, getOperand, getType, translating backend policy into executable code.
- CN: 这一段实现了 getMinForClamp、assert、getOperand、getType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:   // Choose the appropriate scalar-arg dot intrinsic for floats.
1082:   // K=1 and double types use scalar expansion instead.
1083:   Intrinsic::ID FloatDotID = Intrinsic::not_intrinsic;
1084:   bool UseScalarFP = IsFP && (EltTy->isDoubleTy() || LHSCols == 1);
1085:   if (IsFP && !UseScalarFP) {
1086:     switch (LHSCols) {
1087:     case 2:
1088:       FloatDotID = Intrinsic::dx_dot2;
1089:       break;
1090:     case 3:
1091:       FloatDotID = Intrinsic::dx_dot3;
1092:       break;
1093:     case 4:
1094:       FloatDotID = Intrinsic::dx_dot4;
1095:       break;
1096:     default:
1097:       reportFatalUsageError(
1098:           "Invalid matrix inner dimension for dot product: must be 2-4");
1099:       return nullptr;
1100:     }
1101:   }
1102:
1103:   for (unsigned C = 0; C < RHSCols; ++C) {
1104:     for (unsigned R = 0; R < LHSRows; ++R) {
1105:       // Gather row R from LHS and column C from RHS.
1106:       SmallVector<Value *, 4> RowElts, ColElts;
1107:       for (unsigned K = 0; K < LHSCols; ++K) {
1108:         RowElts.push_back(LHSElts[K * LHSRows + R]);
1109:         ColElts.push_back(RHSElts[C * LHSCols + K]);
1110:       }
1111:
1112:       Value *Dot;
1113:       if (UseScalarFP) {
1114:         // Scalar fmul+fmuladd expansion for double types and K=1.
1115:         Dot = Builder.CreateFMul(RowElts[0], ColElts[0]);
1116:         for (unsigned K = 1; K < LHSCols; ++K)
1117:           Dot = Builder.CreateIntrinsic(EltTy, Intrinsic::fmuladd,
1118:                                         {RowElts[K], ColElts[K], Dot});
1119:       } else if (IsFP) {
1120:         // Emit scalar-arg DXIL dot directly (dx.dot2/dx.dot3/dx.dot4).
1121:         SmallVector<Value *, 8> Args;
1122:         Args.append(RowElts.begin(), RowElts.end());
1123:         Args.append(ColElts.begin(), ColElts.end());
1124:         Dot = Builder.CreateIntrinsic(EltTy, FloatDotID, Args);
1125:       } else {
1126:         // Integer: emit multiply + imad chain.
1127:         Dot = Builder.CreateMul(RowElts[0], ColElts[0]);
1128:         for (unsigned K = 1; K < LHSCols; ++K)
1129:           Dot = Builder.CreateIntrinsic(EltTy, Intrinsic::dx_imad,
1130:                                         {RowElts[K], ColElts[K], Dot});
1131:       }
1132:       unsigned ResIdx = C * LHSRows + R;
1133:       Result = Builder.CreateInsertElement(Result, Dot, ResIdx);
1134:     }
1135:   }
1136:   return Result;
1137: }
1138:
1139: // Expand llvm.matrix.transpose as a shufflevector that permutes elements
1140: // from column-major source to column-major transposed layout.
1141: // Element (r,c) at index c*Rows + r moves to index r*Cols + c.
1142: static Value *expandMatrixTranspose(CallInst *Orig) {
1143:   Value *Mat = Orig->getArgOperand(0);
1144:   unsigned Rows = cast<ConstantInt>(Orig->getArgOperand(1))->getZExtValue();
1145:   unsigned Cols = cast<ConstantInt>(Orig->getArgOperand(2))->getZExtValue();
1146:
1147:   unsigned NumElts = Rows * Cols;
1148:   SmallVector<int, 16> Mask(NumElts);
1149:   for (unsigned I = 0; I < NumElts; ++I)
1150:     Mask[I] = (I % Cols) * Rows + (I / Cols);
1151:
1152:   IRBuilder<> Builder(Orig);
1153:   return Builder.CreateShuffleVector(Mat, Mask);
1154: }
1155:
1156: static bool expandIntrinsic(Function &F, CallInst *Orig) {
1157:   Value *Result = nullptr;
1158:   Intrinsic::ID IntrinsicId = F.getIntrinsicID();
1159:   switch (IntrinsicId) {
1160:   case Intrinsic::abs:
1161:     Result = expandAbs(Orig);
1162:     break;
1163:   case Intrinsic::assume:
1164:     Orig->eraseFromParent();
1165:     return true;
1166:   case Intrinsic::atan2:
1167:     Result = expandAtan2Intrinsic(Orig);
1168:     break;
1169:   case Intrinsic::fshl:
1170:     Result = expandFunnelShiftIntrinsic<true>(Orig);
```
- EN: This range implements operational logic in helpers such as isDoubleTy, push_back, CreateFMul, append, translating backend policy into executable code.
- CN: 这一段实现了 isDoubleTy、push_back、CreateFMul、append 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1260
```cpp
1171:     break;
1172:   case Intrinsic::fshr:
1173:     Result = expandFunnelShiftIntrinsic<false>(Orig);
1174:     break;
1175:   case Intrinsic::exp:
1176:     Result = expandExpIntrinsic(Orig);
1177:     break;
1178:   case Intrinsic::is_fpclass:
1179:     Result = expandIsFPClass(Orig);
1180:     break;
1181:   case Intrinsic::log:
1182:     Result = expandLogIntrinsic(Orig);
1183:     break;
1184:   case Intrinsic::log10:
1185:     Result = expandLog10Intrinsic(Orig);
1186:     break;
1187:   case Intrinsic::pow:
1188:   case Intrinsic::powi:
1189:     Result = expandPowIntrinsic(Orig, IntrinsicId);
1190:     break;
1191:   case Intrinsic::dx_all:
1192:   case Intrinsic::dx_any:
1193:     Result = expandAnyOrAllIntrinsic(Orig, IntrinsicId);
1194:     break;
1195:   case Intrinsic::dx_cross:
1196:     Result = expandCrossIntrinsic(Orig);
1197:     break;
1198:   case Intrinsic::dx_uclamp:
1199:   case Intrinsic::dx_sclamp:
1200:   case Intrinsic::dx_nclamp:
1201:     Result = expandClampIntrinsic(Orig, IntrinsicId);
1202:     break;
1203:   case Intrinsic::dx_degrees:
1204:     Result = expandDegreesIntrinsic(Orig);
1205:     break;
1206:   case Intrinsic::dx_isinf:
1207:     Result = expand16BitIsInf(Orig);
1208:     break;
1209:   case Intrinsic::dx_isnan:
1210:     Result = expand16BitIsNaN(Orig);
1211:     break;
1212:   case Intrinsic::dx_lerp:
1213:     Result = expandLerpIntrinsic(Orig);
1214:     break;
1215:   case Intrinsic::dx_normalize:
1216:     Result = expandNormalizeIntrinsic(Orig);
1217:     break;
1218:   case Intrinsic::dx_fdot:
1219:     Result = expandFloatDotIntrinsic(Orig);
1220:     break;
1221:   case Intrinsic::dx_sdot:
1222:   case Intrinsic::dx_udot:
1223:     Result = expandIntegerDotIntrinsic(Orig, IntrinsicId);
1224:     break;
1225:   case Intrinsic::dx_sign:
1226:     Result = expandSignIntrinsic(Orig);
1227:     break;
1228:   case Intrinsic::dx_step:
1229:     Result = expandStepIntrinsic(Orig);
1230:     break;
1231:   case Intrinsic::dx_radians:
1232:     Result = expandRadiansIntrinsic(Orig);
1233:     break;
1234:   case Intrinsic::dx_resource_load_rawbuffer:
1235:     if (expandBufferLoadIntrinsic(Orig, /*IsRaw*/ true))
1236:       return true;
1237:     break;
1238:   case Intrinsic::dx_resource_store_rawbuffer:
1239:     if (expandBufferStoreIntrinsic(Orig, /*IsRaw*/ true))
1240:       return true;
1241:     break;
1242:   case Intrinsic::dx_resource_load_typedbuffer:
1243:     if (expandBufferLoadIntrinsic(Orig, /*IsRaw*/ false))
1244:       return true;
1245:     break;
1246:   case Intrinsic::dx_resource_store_typedbuffer:
1247:     if (expandBufferStoreIntrinsic(Orig, /*IsRaw*/ false))
1248:       return true;
1249:     break;
1250:   case Intrinsic::usub_sat:
1251:     Result = expandUsubSat(Orig);
1252:     break;
1253:   case Intrinsic::vector_reduce_add:
1254:   case Intrinsic::vector_reduce_fadd:
1255:     Result = expandVecReduceAdd(Orig, IntrinsicId);
1256:     break;
1257:   case Intrinsic::matrix_multiply:
1258:     Result = expandMatrixMultiply(Orig);
1259:     break;
1260:   case Intrinsic::matrix_transpose:
```
- EN: This range implements operational logic in helpers such as expandExpIntrinsic, expandIsFPClass, expandLogIntrinsic, expandLog10Intrinsic, translating backend policy into executable code.
- CN: 这一段实现了 expandExpIntrinsic、expandIsFPClass、expandLogIntrinsic、expandLog10Intrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1261-1309
```cpp
1261:     Result = expandMatrixTranspose(Orig);
1262:     break;
1263:   }
1264:   if (Result) {
1265:     Orig->replaceAllUsesWith(Result);
1266:     Orig->eraseFromParent();
1267:     return true;
1268:   }
1269:   return false;
1270: }
1271:
1272: static bool expansionIntrinsics(Module &M) {
1273:   for (auto &F : make_early_inc_range(M.functions())) {
1274:     if (!isIntrinsicExpansion(F))
1275:       continue;
1276:     bool IntrinsicExpanded = false;
1277:     for (User *U : make_early_inc_range(F.users())) {
1278:       auto *IntrinsicCall = dyn_cast<CallInst>(U);
1279:       if (!IntrinsicCall)
1280:         continue;
1281:       IntrinsicExpanded = expandIntrinsic(F, IntrinsicCall);
1282:     }
1283:     if (F.user_empty() && IntrinsicExpanded)
1284:       F.eraseFromParent();
1285:   }
1286:   return true;
1287: }
1288:
1289: PreservedAnalyses DXILIntrinsicExpansion::run(Module &M,
1290:                                               ModuleAnalysisManager &) {
1291:   if (expansionIntrinsics(M))
1292:     return PreservedAnalyses::none();
1293:   return PreservedAnalyses::all();
1294: }
1295:
1296: bool DXILIntrinsicExpansionLegacy::runOnModule(Module &M) {
1297:   return expansionIntrinsics(M);
1298: }
1299:
1300: char DXILIntrinsicExpansionLegacy::ID = 0;
1301:
1302: INITIALIZE_PASS_BEGIN(DXILIntrinsicExpansionLegacy, DEBUG_TYPE,
1303:                       "DXIL Intrinsic Expansion", false, false)
1304: INITIALIZE_PASS_END(DXILIntrinsicExpansionLegacy, DEBUG_TYPE,
1305:                     "DXIL Intrinsic Expansion", false, false)
1306:
1307: ModulePass *llvm::createDXILIntrinsicExpansionLegacyPass() {
1308:   return new DXILIntrinsicExpansionLegacy();
1309: }
```
- EN: This range implements operational logic in helpers such as expandMatrixTranspose, replaceAllUsesWith, eraseFromParent, expansionIntrinsics, translating backend policy into executable code.
- CN: 这一段实现了 expandMatrixTranspose、replaceAllUsesWith、eraseFromParent、expansionIntrinsics 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Intrinsic handling connects target-specific builtins with LLVM IR or machine-level lowering paths.
  - CN: 内建函数处理负责把目标特定 builtin 与 LLVM IR 或机器级降级路径连接起来。
- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include DXILIntrinsicExpansionLegacy, runOnModule, getScalarType, isDoubleTy, expand16BitIsInf, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DXILIntrinsicExpansionLegacy, runOnModule, getScalarType, isDoubleTy, expand16BitIsInf，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILIntrinsicExpansion.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstrTypes.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/MatrixBuilder.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/PassManager.h`
  - `llvm/IR/Type.h`
  - `llvm/Pass.h`
  - `llvm/Support/Casting.h`
