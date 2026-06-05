# SPIRVPrepareFunctions.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVPrepareFunctions.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass modifies function signatures containing aggregate arguments and or return value before IRTranslator.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVPrepareFunctions.cpp - modify function signatures --*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass modifies function signatures containing aggregate arguments
10: // and/or return value before IRTranslator. Information about the original
11: // signatures is stored in metadata. It is used during call lowering to
12: // restore correct SPIR-V types of function arguments and return values.
13: // This pass also substitutes some llvm intrinsic calls with calls to newly
14: // generated functions (as the Khronos LLVM/SPIR-V Translator does).
15: //
16: // NOTE: this pass is a module-level one due to the necessity to modify
17: // GVs/functions.
18: //
19: //===----------------------------------------------------------------------===//
20:
21: #include "SPIRVPrepareFunctions.h"
22: #include "SPIRV.h"
23: #include "SPIRVBuiltins.h"
24: #include "SPIRVSubtarget.h"
25: #include "SPIRVTargetMachine.h"
26: #include "SPIRVUtils.h"
27: #include "llvm/ADT/StringExtras.h"
28: #include "llvm/Analysis/TargetTransformInfo.h"
29: #include "llvm/Analysis/ValueTracking.h"
30: #include "llvm/CodeGen/IntrinsicLowering.h"
31: #include "llvm/IR/IRBuilder.h"
32: #include "llvm/IR/InstIterator.h"
33: #include "llvm/IR/Instructions.h"
34: #include "llvm/IR/IntrinsicInst.h"
35: #include "llvm/IR/Intrinsics.h"
36: #include "llvm/IR/IntrinsicsSPIRV.h"
37: #include "llvm/Transforms/Utils/Cloning.h"
38: #include "llvm/Transforms/Utils/Local.h"
39: #include "llvm/Transforms/Utils/LowerMemIntrinsics.h"
40: #include <regex>
41:
42: using namespace llvm;
43:
44: namespace {
45:
46: class SPIRVPrepareFunctionsImpl {
47:   const SPIRVTargetMachine &TM;
48:   bool substituteIntrinsicCalls(Function *F);
49:   bool substituteAbortKHRCalls(Function *F);
50:   bool terminateBlocksAfterTrap(Module &M, Intrinsic::ID IID);
51:   Function *removeAggregateTypesFromSignature(Function *F);
52:   bool removeAggregateTypesFromCalls(Function *F);
53:
54: public:
55:   SPIRVPrepareFunctionsImpl(const SPIRVTargetMachine &TM) : TM(TM) {}
56:   bool runOnModule(Module &M);
57: };
58:
59: class SPIRVPrepareFunctionsLegacy : public ModulePass {
60:   const SPIRVTargetMachine &TM;
```
- EN: This range defines or declares important types such as SPIRVPrepareFunctionsImpl, substituteIntrinsicCalls, substituteAbortKHRCalls, terminateBlocksAfterTrap, shaping the data model used by SPIRVPrepareFunctions.cpp.
- CN: 这一段定义或声明了 SPIRVPrepareFunctionsImpl、substituteIntrinsicCalls、substituteAbortKHRCalls、terminateBlocksAfterTrap 等关键类型，构成 SPIRVPrepareFunctions.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:
 62: public:
 63:   static char ID;
 64:   SPIRVPrepareFunctionsLegacy(const SPIRVTargetMachine &TM)
 65:       : ModulePass(ID), TM(TM) {}
 66:
 67:   bool runOnModule(Module &M) override {
 68:     return SPIRVPrepareFunctionsImpl(TM).runOnModule(M);
 69:   }
 70:
 71:   StringRef getPassName() const override { return "SPIRV prepare functions"; }
 72: };
 73:
 74: static cl::list<std::string> SPVAllowUnknownIntrinsics(
 75:     "spv-allow-unknown-intrinsics", cl::CommaSeparated,
 76:     cl::desc("Emit unknown intrinsics as calls to external functions. A "
 77:              "comma-separated input list of intrinsic prefixes must be "
 78:              "provided, and only intrinsics carrying a listed prefix get "
 79:              "emitted as described."),
 80:     cl::value_desc("intrinsic_prefix_0,intrinsic_prefix_1"), cl::ValueOptional);
 81: } // namespace
 82:
 83: char SPIRVPrepareFunctionsLegacy::ID = 0;
 84:
 85: INITIALIZE_PASS(SPIRVPrepareFunctionsLegacy, "spirv-prepare-functions",
 86:                 "SPIRV prepare functions", false, false)
 87:
 88: static std::string lowerLLVMIntrinsicName(IntrinsicInst *II) {
 89:   Function *IntrinsicFunc = II->getCalledFunction();
 90:   assert(IntrinsicFunc && "Missing function");
 91:   std::string FuncName = IntrinsicFunc->getName().str();
 92:   llvm::replace(FuncName, '.', '_');
 93:   FuncName = "spirv." + FuncName;
 94:   return FuncName;
 95: }
 96:
 97: static Function *getOrCreateFunction(Module *M, Type *RetTy,
 98:                                      ArrayRef<Type *> ArgTypes,
 99:                                      StringRef Name) {
100:   FunctionType *FT = FunctionType::get(RetTy, ArgTypes, false);
101:   Function *F = M->getFunction(Name);
102:   if (F && F->getFunctionType() == FT)
103:     return F;
104:   Function *NewF = Function::Create(FT, GlobalValue::ExternalLinkage, Name, M);
105:   if (F)
106:     NewF->setDSOLocal(F->isDSOLocal());
107:   NewF->setCallingConv(CallingConv::SPIR_FUNC);
108:   return NewF;
109: }
110:
111: static bool lowerIntrinsicToFunction(IntrinsicInst *Intrinsic,
112:                                      const TargetTransformInfo &TTI) {
113:   // For @llvm.memset.* intrinsic cases with constant value and length arguments
114:   // are emulated via "storing" a constant array to the destination. For other
115:   // cases we wrap the intrinsic in @spirv.llvm_memset_* function and expand the
116:   // intrinsic to a loop via expandMemSetAsLoop().
117:   if (auto *MSI = dyn_cast<MemSetInst>(Intrinsic))
118:     if (isa<Constant>(MSI->getValue()) && isa<ConstantInt>(MSI->getLength()))
119:       return false; // It is handled later using OpCopyMemorySized.
120:
```
- EN: This range implements operational logic in helpers such as SPIRVPrepareFunctionsLegacy, ModulePass, runOnModule, SPIRVPrepareFunctionsImpl, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVPrepareFunctionsLegacy、ModulePass、runOnModule、SPIRVPrepareFunctionsImpl 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   Module *M = Intrinsic->getModule();
122:   std::string FuncName = lowerLLVMIntrinsicName(Intrinsic);
123:   if (Intrinsic->isVolatile())
124:     FuncName += ".volatile";
125:   // Redirect @llvm.intrinsic.* call to @spirv.llvm_intrinsic_*
126:   Function *F = M->getFunction(FuncName);
127:   if (F) {
128:     Intrinsic->setCalledFunction(F);
129:     return true;
130:   }
131:   // TODO copy arguments attributes: nocapture writeonly.
132:   FunctionCallee FC =
133:       M->getOrInsertFunction(FuncName, Intrinsic->getFunctionType());
134:   auto IntrinsicID = Intrinsic->getIntrinsicID();
135:   Intrinsic->setCalledFunction(FC);
136:
137:   F = dyn_cast<Function>(FC.getCallee());
138:   assert(F && "Callee must be a function");
139:
140:   switch (IntrinsicID) {
141:   case Intrinsic::memset: {
142:     auto *MSI = static_cast<MemSetInst *>(Intrinsic);
143:     Argument *Dest = F->getArg(0);
144:     Argument *Val = F->getArg(1);
145:     Argument *Len = F->getArg(2);
146:     Argument *IsVolatile = F->getArg(3);
147:     Dest->setName("dest");
148:     Val->setName("val");
149:     Len->setName("len");
150:     IsVolatile->setName("isvolatile");
151:     BasicBlock *EntryBB = BasicBlock::Create(M->getContext(), "entry", F);
152:     IRBuilder<> IRB(EntryBB);
153:     auto *MemSet = IRB.CreateMemSet(Dest, Val, Len, MSI->getDestAlign(),
154:                                     MSI->isVolatile());
155:     IRB.CreateRetVoid();
156:     expandMemSetAsLoop(cast<MemSetInst>(MemSet), TTI);
157:     MemSet->eraseFromParent();
158:     break;
159:   }
160:   case Intrinsic::bswap: {
161:     BasicBlock *EntryBB = BasicBlock::Create(M->getContext(), "entry", F);
162:     IRBuilder<> IRB(EntryBB);
163:     auto *BSwap = IRB.CreateIntrinsic(Intrinsic::bswap, Intrinsic->getType(),
164:                                       F->getArg(0));
165:     IRB.CreateRet(BSwap);
166:     IntrinsicLowering IL(M->getDataLayout());
167:     IL.LowerIntrinsicCall(BSwap);
168:     break;
169:   }
170:   default:
171:     break;
172:   }
173:   return true;
174: }
175:
176: static std::string getAnnotation(Value *AnnoVal, Value *OptAnnoVal) {
177:   if (auto *Ref = dyn_cast_or_null<GetElementPtrInst>(AnnoVal))
178:     AnnoVal = Ref->getOperand(0);
179:   if (auto *Ref = dyn_cast_or_null<BitCastInst>(OptAnnoVal))
180:     OptAnnoVal = Ref->getOperand(0);
```
- EN: This range implements operational logic in helpers such as getModule, lowerLLVMIntrinsicName, getFunction, setCalledFunction, translating backend policy into executable code.
- CN: 这一段实现了 getModule、lowerLLVMIntrinsicName、getFunction、setCalledFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:
182:   std::string Anno;
183:   if (auto *C = dyn_cast_or_null<Constant>(AnnoVal)) {
184:     StringRef Str;
185:     if (getConstantStringInfo(C, Str))
186:       Anno = Str;
187:   }
188:   // handle optional annotation parameter in a way that Khronos Translator do
189:   // (collect integers wrapped in a struct)
190:   if (auto *C = dyn_cast_or_null<Constant>(OptAnnoVal);
191:       C && C->getNumOperands()) {
192:     Value *MaybeStruct = C->getOperand(0);
193:     if (auto *Struct = dyn_cast<ConstantStruct>(MaybeStruct)) {
194:       for (unsigned I = 0, E = Struct->getNumOperands(); I != E; ++I) {
195:         if (auto *CInt = dyn_cast<ConstantInt>(Struct->getOperand(I)))
196:           Anno += (I == 0 ? ": " : ", ") +
197:                   std::to_string(CInt->getType()->getIntegerBitWidth() == 1
198:                                      ? CInt->getZExtValue()
199:                                      : CInt->getSExtValue());
200:       }
201:     } else if (auto *Struct = dyn_cast<ConstantAggregateZero>(MaybeStruct)) {
202:       // { i32 i32 ... } zeroinitializer
203:       for (unsigned I = 0, E = Struct->getType()->getStructNumElements();
204:            I != E; ++I)
205:         Anno += I == 0 ? ": 0" : ", 0";
206:     }
207:   }
208:   return Anno;
209: }
210:
211: static SmallVector<Metadata *> parseAnnotation(Value *I,
212:                                                const std::string &Anno,
213:                                                LLVMContext &Ctx,
214:                                                Type *Int32Ty) {
215:   // Try to parse the annotation string according to the following rules:
216:   // annotation := ({kind} | {kind:value,value,...})+
217:   // kind := number
218:   // value := number | string
219:   static const std::regex R(
220:       "\\{(\\d+)(?:[:,](\\d+|\"[^\"]*\")(?:,(\\d+|\"[^\"]*\"))*)?\\}");
221:   SmallVector<Metadata *> MDs;
222:   int Pos = 0;
223:   for (std::sregex_iterator
224:            It = std::sregex_iterator(Anno.begin(), Anno.end(), R),
225:            ItEnd = std::sregex_iterator();
226:        It != ItEnd; ++It) {
227:     if (It->position() != Pos)
228:       return SmallVector<Metadata *>{};
229:     Pos = It->position() + It->length();
230:     std::smatch Match = *It;
231:     SmallVector<Metadata *> MDsItem;
232:     for (std::size_t i = 1; i < Match.size(); ++i) {
233:       std::ssub_match SMatch = Match[i];
234:       std::string Item = SMatch.str();
235:       if (Item.length() == 0)
236:         break;
237:       if (Item[0] == '"') {
238:         Item = Item.substr(1, Item.length() - 2);
239:         // Acceptable format of the string snippet is:
240:         static const std::regex RStr("^(\\d+)(?:,(\\d+))*$");
```
- EN: This range implements operational logic in helpers such as getNumOperands, getOperand, getZExtValue, getSExtValue, translating backend policy into executable code.
- CN: 这一段实现了 getNumOperands、getOperand、getZExtValue、getSExtValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:         if (std::smatch MatchStr; std::regex_match(Item, MatchStr, RStr)) {
242:           for (std::size_t SubIdx = 1; SubIdx < MatchStr.size(); ++SubIdx)
243:             if (std::string SubStr = MatchStr[SubIdx].str(); SubStr.length())
244:               MDsItem.push_back(ConstantAsMetadata::get(
245:                   ConstantInt::get(Int32Ty, std::stoi(SubStr))));
246:         } else {
247:           MDsItem.push_back(MDString::get(Ctx, Item));
248:         }
249:       } else if (int32_t Num; llvm::to_integer(StringRef(Item), Num, 10)) {
250:         MDsItem.push_back(
251:             ConstantAsMetadata::get(ConstantInt::get(Int32Ty, Num)));
252:       } else {
253:         MDsItem.push_back(MDString::get(Ctx, Item));
254:       }
255:     }
256:     if (MDsItem.size() == 0)
257:       return SmallVector<Metadata *>{};
258:     MDs.push_back(MDNode::get(Ctx, MDsItem));
259:   }
260:   return Pos == static_cast<int>(Anno.length()) ? std::move(MDs)
261:                                                 : SmallVector<Metadata *>{};
262: }
263:
264: static void lowerPtrAnnotation(IntrinsicInst *II) {
265:   LLVMContext &Ctx = II->getContext();
266:   Type *Int32Ty = Type::getInt32Ty(Ctx);
267:
268:   // Retrieve an annotation string from arguments.
269:   Value *PtrArg = nullptr;
270:   if (auto *BI = dyn_cast<BitCastInst>(II->getArgOperand(0)))
271:     PtrArg = BI->getOperand(0);
272:   else
273:     PtrArg = II->getOperand(0);
274:   std::string Anno =
275:       getAnnotation(II->getArgOperand(1),
276:                     4 < II->arg_size() ? II->getArgOperand(4) : nullptr);
277:
278:   // Parse the annotation.
279:   SmallVector<Metadata *> MDs = parseAnnotation(II, Anno, Ctx, Int32Ty);
280:
281:   // If the annotation string is not parsed successfully we don't know the
282:   // format used and output it as a general UserSemantic decoration.
283:   // Otherwise MDs is a Metadata tuple (a decoration list) in the format
284:   // expected by `spirv.Decorations`.
285:   if (MDs.size() == 0) {
286:     auto UserSemantic = ConstantAsMetadata::get(ConstantInt::get(
287:         Int32Ty, static_cast<uint32_t>(SPIRV::Decoration::UserSemantic)));
288:     MDs.push_back(MDNode::get(Ctx, {UserSemantic, MDString::get(Ctx, Anno)}));
289:   }
290:
291:   // Build the internal intrinsic function.
292:   IRBuilder<> IRB(II->getParent());
293:   IRB.SetInsertPoint(II);
294:   IRB.CreateIntrinsic(
295:       Intrinsic::spv_assign_decoration, {PtrArg->getType()},
296:       {PtrArg, MetadataAsValue::get(Ctx, MDNode::get(Ctx, MDs))});
297:   II->replaceAllUsesWith(II->getOperand(0));
298: }
299:
300: static void lowerFunnelShifts(IntrinsicInst *FSHIntrinsic) {
```
- EN: This range implements operational logic in helpers such as std::regex_match, size, ConstantInt::get, push_back, translating backend policy into executable code.
- CN: 这一段实现了 std::regex_match、size、ConstantInt::get、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:   // Get a separate function - otherwise, we'd have to rework the CFG of the
302:   // current one. Then simply replace the intrinsic uses with a call to the new
303:   // function.
304:   // Generate LLVM IR for  i* @spirv.llvm_fsh?_i* (i* %a, i* %b, i* %c)
305:   Module *M = FSHIntrinsic->getModule();
306:   FunctionType *FSHFuncTy = FSHIntrinsic->getFunctionType();
307:   Type *FSHRetTy = FSHFuncTy->getReturnType();
308:   const std::string FuncName = lowerLLVMIntrinsicName(FSHIntrinsic);
309:   Function *FSHFunc =
310:       getOrCreateFunction(M, FSHRetTy, FSHFuncTy->params(), FuncName);
311:
312:   if (!FSHFunc->empty()) {
313:     FSHIntrinsic->setCalledFunction(FSHFunc);
314:     return;
315:   }
316:   BasicBlock *RotateBB = BasicBlock::Create(M->getContext(), "rotate", FSHFunc);
317:   IRBuilder<> IRB(RotateBB);
318:   Type *Ty = FSHFunc->getReturnType();
319:   // Build the actual funnel shift rotate logic.
320:   // In the comments, "int" is used interchangeably with "vector of int
321:   // elements".
322:   FixedVectorType *VectorTy = dyn_cast<FixedVectorType>(Ty);
323:   Type *IntTy = VectorTy ? VectorTy->getElementType() : Ty;
324:   unsigned BitWidth = IntTy->getIntegerBitWidth();
325:   ConstantInt *BitWidthConstant = IRB.getInt({BitWidth, BitWidth});
326:   Value *BitWidthForInsts =
327:       VectorTy
328:           ? IRB.CreateVectorSplat(VectorTy->getNumElements(), BitWidthConstant)
329:           : BitWidthConstant;
330:   Value *RotateModVal =
331:       IRB.CreateURem(/*Rotate*/ FSHFunc->getArg(2), BitWidthForInsts);
332:   Value *FirstShift = nullptr, *SecShift = nullptr;
333:   if (FSHIntrinsic->getIntrinsicID() == Intrinsic::fshr) {
334:     // Shift the less significant number right, the "rotate" number of bits
335:     // will be 0-filled on the left as a result of this regular shift.
336:     FirstShift = IRB.CreateLShr(FSHFunc->getArg(1), RotateModVal);
337:   } else {
338:     // Shift the more significant number left, the "rotate" number of bits
339:     // will be 0-filled on the right as a result of this regular shift.
340:     FirstShift = IRB.CreateShl(FSHFunc->getArg(0), RotateModVal);
341:   }
342:   // We want the "rotate" number of the more significant int's LSBs (MSBs) to
343:   // occupy the leftmost (rightmost) "0 space" left by the previous operation.
344:   // Therefore, subtract the "rotate" number from the integer bitsize...
345:   Value *SubRotateVal = IRB.CreateSub(BitWidthForInsts, RotateModVal);
346:   if (FSHIntrinsic->getIntrinsicID() == Intrinsic::fshr) {
347:     // ...and left-shift the more significant int by this number, zero-filling
348:     // the LSBs.
349:     SecShift = IRB.CreateShl(FSHFunc->getArg(0), SubRotateVal);
350:   } else {
351:     // ...and right-shift the less significant int by this number, zero-filling
352:     // the MSBs.
353:     SecShift = IRB.CreateLShr(FSHFunc->getArg(1), SubRotateVal);
354:   }
355:   // A simple binary addition of the shifted ints yields the final result.
356:   IRB.CreateRet(IRB.CreateOr(FirstShift, SecShift));
357:
358:   FSHIntrinsic->setCalledFunction(FSHFunc);
359: }
360:
```
- EN: This range implements operational logic in helpers such as getModule, getFunctionType, getReturnType, lowerLLVMIntrinsicName, translating backend policy into executable code.
- CN: 这一段实现了 getModule、getFunctionType、getReturnType、lowerLLVMIntrinsicName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361: static void lowerConstrainedFPCmpIntrinsic(
362:     ConstrainedFPCmpIntrinsic *ConstrainedCmpIntrinsic,
363:     SmallVector<Instruction *> &EraseFromParent) {
364:   if (!ConstrainedCmpIntrinsic)
365:     return;
366:   // Extract the floating-point values being compared
367:   Value *LHS = ConstrainedCmpIntrinsic->getArgOperand(0);
368:   Value *RHS = ConstrainedCmpIntrinsic->getArgOperand(1);
369:   FCmpInst::Predicate Pred = ConstrainedCmpIntrinsic->getPredicate();
370:   IRBuilder<> Builder(ConstrainedCmpIntrinsic);
371:   Value *FCmp = Builder.CreateFCmp(Pred, LHS, RHS);
372:   ConstrainedCmpIntrinsic->replaceAllUsesWith(FCmp);
373:   EraseFromParent.push_back(dyn_cast<Instruction>(ConstrainedCmpIntrinsic));
374: }
375:
376: static void lowerExpectAssume(IntrinsicInst *II) {
377:   // If we cannot use the SPV_KHR_expect_assume extension, then we need to
378:   // ignore the intrinsic and move on. It should be removed later on by LLVM.
379:   // Otherwise we should lower the intrinsic to the corresponding SPIR-V
380:   // instruction.
381:   // For @llvm.assume we have OpAssumeTrueKHR.
382:   // For @llvm.expect we have OpExpectKHR.
383:   //
384:   // We need to lower this into a builtin and then the builtin into a SPIR-V
385:   // instruction.
386:   if (II->getIntrinsicID() == Intrinsic::assume) {
387:     Function *F = Intrinsic::getOrInsertDeclaration(
388:         II->getModule(), Intrinsic::SPVIntrinsics::spv_assume);
389:     II->setCalledFunction(F);
390:   } else if (II->getIntrinsicID() == Intrinsic::expect) {
391:     Function *F = Intrinsic::getOrInsertDeclaration(
392:         II->getModule(), Intrinsic::SPVIntrinsics::spv_expect,
393:         {II->getOperand(0)->getType()});
394:     II->setCalledFunction(F);
395:   } else {
396:     llvm_unreachable("Unknown intrinsic");
397:   }
398: }
399:
400: static bool toSpvLifetimeIntrinsic(IntrinsicInst *II, Intrinsic::ID NewID) {
401:   auto *LifetimeArg0 = II->getArgOperand(0);
402:
403:   // If the lifetime argument is a poison value, the intrinsic has no effect.
404:   if (isa<PoisonValue>(LifetimeArg0)) {
405:     II->eraseFromParent();
406:     return true;
407:   }
408:
409:   IRBuilder<> Builder(II);
410:   auto *Alloca = cast<AllocaInst>(LifetimeArg0);
411:   std::optional<TypeSize> Size =
412:       Alloca->getAllocationSize(Alloca->getDataLayout());
413:   Value *SizeVal = Builder.getInt64(Size ? *Size : -1);
414:   Builder.CreateIntrinsic(NewID, Alloca->getType(), {SizeVal, LifetimeArg0});
415:   II->eraseFromParent();
416:   return true;
417: }
418:
419: static void
420: lowerConstrainedFmuladd(IntrinsicInst *II,
```
- EN: This range implements operational logic in helpers such as getArgOperand, getPredicate, Builder, CreateFCmp, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、getPredicate、Builder、CreateFCmp 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:                         SmallVector<Instruction *> &EraseFromParent) {
422:   auto *FPI = cast<ConstrainedFPIntrinsic>(II);
423:   Value *A = FPI->getArgOperand(0);
424:   Value *Mul = FPI->getArgOperand(1);
425:   Value *Add = FPI->getArgOperand(2);
426:   IRBuilder<> Builder(II->getParent());
427:   Builder.SetInsertPoint(II);
428:   std::optional<RoundingMode> Rounding = FPI->getRoundingMode();
429:   Value *Product = Builder.CreateFMul(A, Mul, II->getName() + ".mul");
430:   Value *Result = Builder.CreateConstrainedFPBinOp(
431:       Intrinsic::experimental_constrained_fadd, Product, Add, {},
432:       II->getName() + ".add", nullptr, Rounding);
433:   II->replaceAllUsesWith(Result);
434:   EraseFromParent.push_back(II);
435: }
436:
437: // Substitutes calls to LLVM intrinsics with either calls to SPIR-V intrinsics
438: // or calls to proper generated functions. Returns True if F was modified.
439: bool SPIRVPrepareFunctionsImpl::substituteIntrinsicCalls(Function *F) {
440:   bool Changed = false;
441:   const SPIRVSubtarget &STI = TM.getSubtarget<SPIRVSubtarget>(*F);
442:   SmallVector<Instruction *> EraseFromParent;
443:   const TargetTransformInfo &TTI = TM.getTargetTransformInfo(*F);
444:   for (BasicBlock &BB : *F) {
445:     for (Instruction &I : make_early_inc_range(BB)) {
446:       auto Call = dyn_cast<CallInst>(&I);
447:       if (!Call)
448:         continue;
449:       Function *CF = Call->getCalledFunction();
450:       if (!CF || !CF->isIntrinsic())
451:         continue;
452:       auto *II = cast<IntrinsicInst>(Call);
453:       if (Intrinsic::isTargetIntrinsic(II->getIntrinsicID()) &&
454:           II->getCalledOperand()->getName().starts_with("llvm.spv"))
455:         continue;
456:       switch (II->getIntrinsicID()) {
457:       case Intrinsic::memset:
458:       case Intrinsic::bswap:
459:         Changed |= lowerIntrinsicToFunction(II, TTI);
460:         break;
461:       case Intrinsic::fshl:
462:       case Intrinsic::fshr:
463:         lowerFunnelShifts(II);
464:         Changed = true;
465:         break;
466:       case Intrinsic::assume:
467:       case Intrinsic::expect:
468:         if (STI.canUseExtension(SPIRV::Extension::SPV_KHR_expect_assume))
469:           lowerExpectAssume(II);
470:         Changed = true;
471:         break;
472:       case Intrinsic::lifetime_start:
473:         if (!STI.isShader()) {
474:           Changed |= toSpvLifetimeIntrinsic(
475:               II, Intrinsic::SPVIntrinsics::spv_lifetime_start);
476:         } else {
477:           II->eraseFromParent();
478:           Changed = true;
479:         }
480:         break;
```
- EN: This range implements operational logic in helpers such as getArgOperand, Builder, SetInsertPoint, getRoundingMode, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、Builder、SetInsertPoint、getRoundingMode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:       case Intrinsic::lifetime_end:
482:         if (!STI.isShader()) {
483:           Changed |= toSpvLifetimeIntrinsic(
484:               II, Intrinsic::SPVIntrinsics::spv_lifetime_end);
485:         } else {
486:           II->eraseFromParent();
487:           Changed = true;
488:         }
489:         break;
490:       case Intrinsic::ptr_annotation:
491:         lowerPtrAnnotation(II);
492:         Changed = true;
493:         break;
494:       case Intrinsic::experimental_constrained_fmuladd:
495:         lowerConstrainedFmuladd(II, EraseFromParent);
496:         Changed = true;
497:         break;
498:       case Intrinsic::experimental_constrained_fcmp:
499:       case Intrinsic::experimental_constrained_fcmps:
500:         lowerConstrainedFPCmpIntrinsic(dyn_cast<ConstrainedFPCmpIntrinsic>(II),
501:                                        EraseFromParent);
502:         Changed = true;
503:         break;
504:       default:
505:         if (TM.getTargetTriple().getVendor() == Triple::AMD ||
506:             any_of(SPVAllowUnknownIntrinsics, [II](auto &&Prefix) {
507:               if (Prefix.empty())
508:                 return false;
509:               return II->getCalledFunction()->getName().starts_with(Prefix);
510:             }))
511:           Changed |= lowerIntrinsicToFunction(II, TTI);
512:         break;
513:       }
514:     }
515:   }
516:   for (auto *I : EraseFromParent)
517:     I->eraseFromParent();
518:   return Changed;
519: }
520:
521: static void
522: addFunctionTypeMutation(NamedMDNode *NMD,
523:                         SmallVector<std::pair<int, Type *>> ChangedTys,
524:                         StringRef Name, StringRef AsmConstraints = "") {
525:
526:   LLVMContext &Ctx = NMD->getParent()->getContext();
527:   Type *I32Ty = IntegerType::getInt32Ty(Ctx);
528:
529:   SmallVector<Metadata *> MDArgs;
530:   MDArgs.push_back(MDString::get(Ctx, Name));
531:   transform(ChangedTys, std::back_inserter(MDArgs), [=, &Ctx](auto &&CTy) {
532:     return MDNode::get(
533:         Ctx, {ConstantAsMetadata::get(ConstantInt::get(I32Ty, CTy.first, true)),
534:               ValueAsMetadata::get(Constant::getNullValue(CTy.second))});
535:   });
536:   if (!AsmConstraints.empty())
537:     MDArgs.push_back(MDNode::get(Ctx, MDString::get(Ctx, AsmConstraints)));
538:   NMD->addOperand(MDNode::get(Ctx, MDArgs));
539: }
540:
```
- EN: This range implements operational logic in helpers such as eraseFromParent, lowerPtrAnnotation, lowerConstrainedFmuladd, any_of, translating backend policy into executable code.
- CN: 这一段实现了 eraseFromParent、lowerPtrAnnotation、lowerConstrainedFmuladd、any_of 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541: // Returns F if aggregate argument/return types are not present or cloned F
542: // function with the types replaced by i32 types. The change in types is
543: // noted in 'spv.cloned_funcs' metadata for later restoration.
544: Function *
545: SPIRVPrepareFunctionsImpl::removeAggregateTypesFromSignature(Function *F) {
546:   bool IsRetAggr = F->getReturnType()->isAggregateType();
547:   // Allow intrinsics with aggregate return/argument types to reach GlobalISel.
548:   // Renaming/mutating the signature of an intrinsic would desync its name from
549:   // its argument types and break the IR verifier.
550:   if (F->isIntrinsic())
551:     return F;
552:
553:   IRBuilder<> B(F->getContext());
554:
555:   bool HasAggrArg = llvm::any_of(F->args(), [](Argument &Arg) {
556:     return Arg.getType()->isAggregateType();
557:   });
558:   bool DoClone = IsRetAggr || HasAggrArg;
559:   if (!DoClone)
560:     return F;
561:   SmallVector<std::pair<int, Type *>, 4> ChangedTypes;
562:   Type *RetType = IsRetAggr ? B.getInt32Ty() : F->getReturnType();
563:   if (IsRetAggr)
564:     ChangedTypes.push_back(std::pair<int, Type *>(-1, F->getReturnType()));
565:   SmallVector<Type *, 4> ArgTypes;
566:   for (const auto &Arg : F->args()) {
567:     if (Arg.getType()->isAggregateType()) {
568:       ArgTypes.push_back(B.getInt32Ty());
569:       ChangedTypes.push_back(
570:           std::pair<int, Type *>(Arg.getArgNo(), Arg.getType()));
571:     } else
572:       ArgTypes.push_back(Arg.getType());
573:   }
574:   FunctionType *NewFTy =
575:       FunctionType::get(RetType, ArgTypes, F->getFunctionType()->isVarArg());
576:   Function *NewF =
577:       Function::Create(NewFTy, F->getLinkage(), F->getAddressSpace(),
578:                        F->getName(), F->getParent());
579:
580:   ValueToValueMapTy VMap;
581:   auto NewFArgIt = NewF->arg_begin();
582:   for (auto &Arg : F->args()) {
583:     StringRef ArgName = Arg.getName();
584:     NewFArgIt->setName(ArgName);
585:     VMap[&Arg] = &(*NewFArgIt++);
586:   }
587:   SmallVector<ReturnInst *, 8> Returns;
588:
589:   CloneFunctionInto(NewF, F, VMap, CloneFunctionChangeType::LocalChangesOnly,
590:                     Returns);
591:   NewF->takeName(F);
592:
593:   addFunctionTypeMutation(
594:       NewF->getParent()->getOrInsertNamedMetadata("spv.cloned_funcs"),
595:       std::move(ChangedTypes), NewF->getName());
596:
597:   for (auto *U : make_early_inc_range(F->users())) {
598:     if (CallInst *CI;
599:         (CI = dyn_cast<CallInst>(U)) && CI->getCalledFunction() == F)
600:       CI->mutateFunctionType(NewF->getFunctionType());
```
- EN: This range implements operational logic in helpers such as SPIRVPrepareFunctionsImpl::removeAggregateTypesFromSignature, getReturnType, B, llvm::any_of, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVPrepareFunctionsImpl::removeAggregateTypesFromSignature、getReturnType、B、llvm::any_of 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-660
```cpp
601:     if (auto *C = dyn_cast<Constant>(U))
602:       C->handleOperandChange(F, NewF);
603:     else
604:       U->replaceUsesOfWith(F, NewF);
605:   }
606:
607:   // register the mutation
608:   if (RetType != F->getReturnType())
609:     TM.getSubtarget<SPIRVSubtarget>(*F).getSPIRVGlobalRegistry()->addMutated(
610:         NewF, F->getReturnType());
611:   return NewF;
612: }
613:
614: // Returns true iff `F`'s name resolves (after OpenCL/SPIR-V demangling and
615: // builtin-name lookup) to the SPIR-V friendly built-in `__spirv_AbortKHR`.
616: static bool isAbortKHRBuiltin(const Function &F) {
617:   if (F.isIntrinsic())
618:     return false;
619:   StringRef Name = F.getName();
620:   // Quick reject: the mangled or unmangled name must contain the substring.
621:   if (!Name.contains("__spirv_AbortKHR"))
622:     return false;
623:   std::string Demangled = getOclOrSpirvBuiltinDemangledName(Name);
624:   if (Demangled.empty())
625:     return false;
626:   return SPIRV::lookupBuiltinNameHelper(Demangled) == "__spirv_AbortKHR";
627: }
628:
629: // Rewrites a single call to `__spirv_AbortKHR` into a call to the
630: // `llvm.spv.abort` target intrinsic, then re-terminates the block with
631: // `unreachable`. OpAbortKHR is itself a SPIR-V function-termination
632: // instruction and must be the last instruction in its block, so any trailing
633: // stores/lifetime intrinsics/`ret` emitted by the OpenCL ABI are dropped.
634: // `changeToUnreachable` cleans up any successor PHI predecessor entries.
635: static void rewriteAbortKHRCall(CallInst *CI) {
636:   IRBuilder<> B(CI);
637:   Value *Msg = CI->getArgOperand(0);
638:   // The OpenCL C ABI may pass aggregate arguments by pointer (byval). In that
639:   // case load the underlying value so that OpAbortKHR receives the composite
640:   // itself, as required by the SPV_KHR_abort spec ("Message Type must be a
641:   // concrete type").
642:   if (CI->isByValArgument(0)) {
643:     Type *AggTy = CI->getParamByValType(0);
644:     Msg = B.CreateLoad(AggTy, Msg);
645:   }
646:   B.CreateIntrinsic(Intrinsic::spv_abort, {Msg->getType()}, {Msg});
647:   changeToUnreachable(CI);
648: }
649:
650: // Replace OpenCL/SPIR-V style calls to `__spirv_AbortKHR(message)` (i.e.
651: // calls to `F` when `F` is the `__spirv_AbortKHR` built-in) with calls to the
652: // `llvm.spv.abort` target intrinsic.
653: bool SPIRVPrepareFunctionsImpl::substituteAbortKHRCalls(Function *F) {
654:   if (!isAbortKHRBuiltin(*F))
655:     return false;
656:
657:   SmallVector<CallInst *> Calls;
658:   for (User *U : F->users()) {
659:     auto *CI = dyn_cast<CallInst>(U);
660:     if (!CI || CI->getCalledFunction() != F)
```
- EN: This range implements operational logic in helpers such as handleOperandChange, replaceUsesOfWith, getReturnType, isAbortKHRBuiltin, translating backend policy into executable code.
- CN: 这一段实现了 handleOperandChange、replaceUsesOfWith、getReturnType、isAbortKHRBuiltin 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 661-720
```cpp
661:       continue;
662:     if (CI->arg_size() != 1)
663:       continue;
664:     Calls.push_back(CI);
665:   }
666:
667:   for (CallInst *CI : Calls)
668:     rewriteAbortKHRCall(CI);
669:
670:   return !Calls.empty();
671: }
672:
673: // When the SPV_KHR_abort extension is enabled, `llvm.trap` and
674: // `llvm.ubsantrap` are lowered to `OpAbortKHR` during instruction selection.
675: // `OpAbortKHR` is itself a SPIR-V block terminator, so any instructions that
676: // follow the trap call within the same basic block (e.g. `ret`, lifetime
677: // markers) would produce SPIR-V ops after `OpAbortKHR` and break validation.
678: // Terminate the block right after each call to the trap intrinsics by replacing
679: // the next instruction with `unreachable`.
680: bool SPIRVPrepareFunctionsImpl::terminateBlocksAfterTrap(Module &M,
681:                                                          Intrinsic::ID IID) {
682:   assert((IID == Intrinsic::trap || IID == Intrinsic::ubsantrap) &&
683:          "Expected trap intrinsic ID");
684:
685:   Function *F = Intrinsic::getDeclarationIfExists(&M, IID);
686:   if (!F)
687:     return false;
688:
689:   // If the target doesn't support SPV_KHR_abort, we won't be able to lower
690:   // the trap intrinsic to OpAbortKHR, so we can skip the block-terminating
691:   // transformation.
692:   const auto &ST = TM.getSubtarget<SPIRVSubtarget>(*F);
693:   if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_abort))
694:     return false;
695:
696:   SmallVector<CallInst *> Calls;
697:   for (User *U : F->users()) {
698:     auto *CI = dyn_cast<CallInst>(U);
699:     if (!CI || CI->getCalledFunction() != F)
700:       continue;
701:     Calls.push_back(CI);
702:   }
703:
704:   bool Changed = false;
705:   for (CallInst *CI : Calls) {
706:     Instruction *Next = CI->getNextNode();
707:     if (!Next || isa<UnreachableInst>(Next))
708:       continue;
709:     changeToUnreachable(Next);
710:     Changed = true;
711:   }
712:   return Changed;
713: }
714:
715: static std::string fixMultiOutputConstraintString(StringRef Constraints) {
716:   // We should only have one =r return for the made up ASM type.
717:   SmallVector<StringRef> Tmp;
718:   SplitString(Constraints, Tmp, ",");
719:   std::string SafeConstraints("=r,");
720:   for (unsigned I = 0u; I != Tmp.size() - 1; ++I) {
```
- EN: This range implements operational logic in helpers such as push_back, rewriteAbortKHRCall, empty, Intrinsic::getDeclarationIfExists, translating backend policy into executable code.
- CN: 这一段实现了 push_back、rewriteAbortKHRCall、empty、Intrinsic::getDeclarationIfExists 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-780
```cpp
721:     if (Tmp[I].starts_with('=') && (Tmp[I][1] == '&' || isalnum(Tmp[I][1])))
722:       continue;
723:     SafeConstraints.append(Tmp[I]).append({','});
724:   }
725:   SafeConstraints.append(Tmp.back());
726:
727:   return SafeConstraints;
728: }
729:
730: // Mutates indirect and inline ASM callsites iff aggregate argument/return types
731: // are present with the types replaced by i32 types. The change in types is
732: // noted in 'spv.mutated_callsites' metadata for later restoration. For ASM we
733: // also have to mutate the constraint string as IRTranslator tries to handle
734: // multiple outputs and expects an aggregate return type in their presence.
735: bool SPIRVPrepareFunctionsImpl::removeAggregateTypesFromCalls(Function *F) {
736:   if (F->isDeclaration() || F->isIntrinsic())
737:     return false;
738:
739:   SmallVector<std::pair<CallBase *, FunctionType *>> Calls;
740:   for (auto &&I : instructions(F)) {
741:     if (auto *CB = dyn_cast<CallBase>(&I)) {
742:       if (!CB->getCalledOperand() || CB->getCalledFunction())
743:         continue;
744:       if (CB->getType()->isAggregateType() ||
745:           any_of(CB->args(),
746:                  [](auto &&Arg) { return Arg->getType()->isAggregateType(); }))
747:         Calls.emplace_back(CB, nullptr);
748:     }
749:   }
750:
751:   if (Calls.empty())
752:     return false;
753:
754:   IRBuilder<> B(F->getContext());
755:
756:   for (auto &&[CB, NewFnTy] : Calls) {
757:     SmallVector<std::pair<int, Type *>> ChangedTypes;
758:     SmallVector<Type *> NewArgTypes;
759:
760:     Type *RetTy = CB->getType();
761:     if (RetTy->isAggregateType()) {
762:       ChangedTypes.emplace_back(-1, RetTy);
763:       RetTy = B.getInt32Ty();
764:     }
765:
766:     for (auto &&Arg : CB->args()) {
767:       if (Arg->getType()->isAggregateType()) {
768:         NewArgTypes.push_back(B.getInt32Ty());
769:         ChangedTypes.emplace_back(Arg.getOperandNo(), Arg->getType());
770:       } else {
771:         NewArgTypes.push_back(Arg->getType());
772:       }
773:     }
774:     NewFnTy = FunctionType::get(RetTy, NewArgTypes,
775:                                 CB->getFunctionType()->isVarArg());
776:
777:     if (!CB->hasName())
778:       CB->setName("spv.mutated_callsite." + F->getName());
779:     else
780:       CB->setName("spv.named_mutated_callsite." + F->getName() + "." +
```
- EN: This range implements operational logic in helpers such as append, SPIRVPrepareFunctionsImpl::removeAggregateTypesFromCalls, getType, emplace_back, translating backend policy into executable code.
- CN: 这一段实现了 append、SPIRVPrepareFunctionsImpl::removeAggregateTypesFromCalls、getType、emplace_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 781-840
```cpp
781:                   CB->getName());
782:
783:     std::string Constraints;
784:     if (auto *ASM = dyn_cast<InlineAsm>(CB->getCalledOperand())) {
785:       Constraints = ASM->getConstraintString();
786:
787:       CB->setCalledOperand(InlineAsm::get(
788:           NewFnTy, ASM->getAsmString(),
789:           fixMultiOutputConstraintString(Constraints), ASM->hasSideEffects(),
790:           ASM->isAlignStack(), ASM->getDialect(), ASM->canThrow()));
791:     }
792:
793:     addFunctionTypeMutation(
794:         F->getParent()->getOrInsertNamedMetadata("spv.mutated_callsites"),
795:         std::move(ChangedTypes), CB->getName(), Constraints);
796:   }
797:
798:   for (auto &&[CB, NewFTy] : Calls) {
799:     if (NewFTy->getReturnType() != CB->getType())
800:       TM.getSubtarget<SPIRVSubtarget>(*F).getSPIRVGlobalRegistry()->addMutated(
801:           CB, CB->getType());
802:     CB->mutateFunctionType(NewFTy);
803:   }
804:
805:   return true;
806: }
807:
808: bool SPIRVPrepareFunctionsImpl::runOnModule(Module &M) {
809:   // Resolve the SPIR-V environment from module content before any
810:   // function-level processing. This must happen before legalization so that
811:   // isShader()/isKernel() return correct values.
812:   const_cast<SPIRVTargetMachine &>(TM)
813:       .getMutableSubtargetImpl()
814:       ->resolveEnvFromModule(M);
815:
816:   bool Changed = false;
817:   if (M.functions().empty()) {
818:     // If there are no functions, insert a service
819:     // function so that the global/constant tracking intrinsics
820:     // will be created. Without these intrinsics the generated SPIR-V
821:     // will be empty. The service function itself is not emitted.
822:     Function *SF = getOrCreateBackendServiceFunction(M);
823:     BasicBlock *BB = BasicBlock::Create(M.getContext(), "entry", SF);
824:     IRBuilder<> IRB(BB);
825:     IRB.CreateRetVoid();
826:     Changed = true;
827:   }
828:
829:   Changed |= terminateBlocksAfterTrap(M, Intrinsic::trap);
830:   Changed |= terminateBlocksAfterTrap(M, Intrinsic::ubsantrap);
831:
832:   for (Function &F : M) {
833:     Changed |= substituteAbortKHRCalls(&F);
834:     Changed |= substituteIntrinsicCalls(&F);
835:     Changed |= sortBlocks(F);
836:     Changed |= removeAggregateTypesFromCalls(&F);
837:   }
838:
839:   std::vector<Function *> FuncsWorklist;
840:   for (auto &F : M)
```
- EN: This range implements operational logic in helpers such as getName, getConstraintString, isAlignStack, std::move, translating backend policy into executable code.
- CN: 这一段实现了 getName、getConstraintString、isAlignStack、std::move 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 841-864
```cpp
841:     FuncsWorklist.push_back(&F);
842:
843:   for (auto *F : FuncsWorklist) {
844:     Function *NewF = removeAggregateTypesFromSignature(F);
845:
846:     if (NewF != F) {
847:       F->eraseFromParent();
848:       Changed = true;
849:     }
850:   }
851:   return Changed;
852: }
853:
854: PreservedAnalyses SPIRVPrepareFunctions::run(Module &M,
855:                                              ModuleAnalysisManager &AM) {
856:   return SPIRVPrepareFunctionsImpl(TM).runOnModule(M)
857:              ? PreservedAnalyses::none()
858:              : PreservedAnalyses::all();
859: }
860:
861: ModulePass *
862: llvm::createSPIRVPrepareFunctionsPass(const SPIRVTargetMachine &TM) {
863:   return new SPIRVPrepareFunctionsLegacy(TM);
864: }
```
- EN: This range implements operational logic in helpers such as push_back, removeAggregateTypesFromSignature, eraseFromParent, SPIRVPrepareFunctionsImpl, translating backend policy into executable code.
- CN: 这一段实现了 push_back、removeAggregateTypesFromSignature、eraseFromParent、SPIRVPrepareFunctionsImpl 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVPrepareFunctionsImpl, substituteIntrinsicCalls, substituteAbortKHRCalls, terminateBlocksAfterTrap, removeAggregateTypesFromSignature, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVPrepareFunctionsImpl, substituteIntrinsicCalls, substituteAbortKHRCalls, terminateBlocksAfterTrap, removeAggregateTypesFromSignature，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVPrepareFunctions.h`
  - `SPIRV.h`
  - `SPIRVBuiltins.h`
  - `SPIRVSubtarget.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/StringExtras.h`
  - `llvm/Analysis/TargetTransformInfo.h`
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/CodeGen/IntrinsicLowering.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/Transforms/Utils/Cloning.h`
  - `llvm/Transforms/Utils/Local.h`
  - `llvm/Transforms/Utils/LowerMemIntrinsics.h`
- System/standard headers / 系统或标准头文件:
  - `regex`
