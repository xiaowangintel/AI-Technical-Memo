# NVPTXLowerArgs.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXLowerArgs.cpp`
- Repository: `llvm-project`
- Purpose (EN): Arguments to kernel and device functions are passed via param space, which imposes certain restrictions: http: docs.nvidia.com cuda parallel-thread-execution #state-spaces.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- NVPTXLowerArgs.cpp - Lower arguments ------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //
10: // Arguments to kernel and device functions are passed via param space,
11: // which imposes certain restrictions:
12: // http://docs.nvidia.com/cuda/parallel-thread-execution/#state-spaces
13: //
14: // Kernel parameters are read-only and accessible only via ld.param
15: // instruction, directly or via a pointer.
16: //
17: // Device function parameters are directly accessible via
18: // ld.param/st.param, but taking the address of one returns a pointer
19: // to a copy created in local space which *can't* be used with
20: // ld.param/st.param.
21: //
22: // Copying a byval struct into local memory in IR allows us to enforce
23: // the param space restrictions, gives the rest of IR a pointer w/o
24: // param space restrictions, and gives us an opportunity to eliminate
25: // the copy.
26: //
27: // Pointer arguments to kernel functions need more work to be lowered:
28: //
29: // 1. Convert non-byval pointer arguments of CUDA kernels to pointers in the
30: //    global address space. This allows later optimizations to emit
31: //    ld.global.*/st.global.* for accessing these pointer arguments. For
32: //    example,
33: //
34: //    define void @foo(float* %input) {
35: //      %v = load float, float* %input, align 4
36: //      ...
37: //    }
38: //
39: //    becomes
40: //
41: //    define void @foo(float* %input) {
42: //      %input2 = addrspacecast float* %input to float addrspace(1)*
43: //      %input3 = addrspacecast float addrspace(1)* %input2 to float*
44: //      %v = load float, float* %input3, align 4
45: //      ...
46: //    }
47: //
48: //    Later, NVPTXInferAddressSpaces will optimize it to
49: //
50: //    define void @foo(float* %input) {
51: //      %input2 = addrspacecast float* %input to float addrspace(1)*
52: //      %v = load float, float addrspace(1)* %input2, align 4
53: //      ...
54: //    }
55: //
56: // 2. Convert byval kernel parameters to pointers in the param address space
57: //    (so that NVPTX emits ld/st.param).  Convert pointers *within* a byval
58: //    kernel parameter to pointers in the global address space. This allows
59: //    NVPTX to emit ld/st.global.
60: //
```
- EN: This range contains banner comments or file-level notes that explain the role of the file and establish context for the backend component.
- CN: 这一段包含文件头注释或说明文字，用来交代该后端组件的职责与上下文。

### Lines 61-120
```cpp
 61: //    struct S {
 62: //      int *x;
 63: //      int *y;
 64: //    };
 65: //    __global__ void foo(S s) {
 66: //      int *b = s.y;
 67: //      // use b
 68: //    }
 69: //
 70: //    "b" points to the global address space. In the IR level,
 71: //
 72: //    define void @foo(ptr byval %input) {
 73: //      %b_ptr = getelementptr {ptr, ptr}, ptr %input, i64 0, i32 1
 74: //      %b = load ptr, ptr %b_ptr
 75: //      ; use %b
 76: //    }
 77: //
 78: //    becomes
 79: //
 80: //    define void @foo({i32*, i32*}* byval %input) {
 81: //      %b_param = addrspacecat ptr %input to ptr addrspace(101)
 82: //      %b_ptr = getelementptr {ptr, ptr}, ptr addrspace(101) %b_param, i64 0, i32 1
 83: //      %b = load ptr, ptr addrspace(101) %b_ptr
 84: //      %b_global = addrspacecast ptr %b to ptr addrspace(1)
 85: //      ; use %b_generic
 86: //    }
 87: //
 88: //    Create a local copy of kernel byval parameters used in a way that *might* mutate
 89: //    the parameter, by storing it in an alloca. Mutations to "grid_constant" parameters
 90: //    are undefined behaviour, and don't require local copies.
 91: //
 92: //    define void @foo(ptr byval(%struct.s) align 4 %input) {
 93: //       store i32 42, ptr %input
 94: //       ret void
 95: //    }
 96: //
 97: //    becomes
 98: //
 99: //    define void @foo(ptr byval(%struct.s) align 4 %input) #1 {
100: //      %input1 = alloca %struct.s, align 4
101: //      %input2 = addrspacecast ptr %input to ptr addrspace(101)
102: //      %input3 = load %struct.s, ptr addrspace(101) %input2, align 4
103: //      store %struct.s %input3, ptr %input1, align 4
104: //      store i32 42, ptr %input1, align 4
105: //      ret void
106: //    }
107: //
108: //    If %input were passed to a device function, or written to memory,
109: //    conservatively assume that %input gets mutated, and create a local copy.
110: //
111: //    Convert param pointers to grid_constant byval kernel parameters that are
112: //    passed into calls (device functions, intrinsics, inline asm), or otherwise
113: //    "escape" (into stores/ptrtoints) to the generic address space, using the
114: //    `nvvm.ptr.param.to.gen` intrinsic, so that NVPTX emits cvta.param
115: //    (available for sm70+)
116: //
117: //    define void @foo(ptr byval(%struct.s) %input) {
118: //      ; %input is a grid_constant
119: //      %call = call i32 @escape(ptr %input)
120: //      ret void
```
- EN: This range contains banner comments or file-level notes that explain the role of the file and establish context for the backend component.
- CN: 这一段包含文件头注释或说明文字，用来交代该后端组件的职责与上下文。

### Lines 121-180
```cpp
121: //    }
122: //
123: //    becomes
124: //
125: //    define void @foo(ptr byval(%struct.s) %input) {
126: //      %input1 = addrspacecast ptr %input to ptr addrspace(101)
127: //      ; the following intrinsic converts pointer to generic. We don't use an addrspacecast
128: //      ; to prevent generic -> param -> generic from getting cancelled out
129: //      %input1.gen = call ptr @llvm.nvvm.ptr.param.to.gen.p0.p101(ptr addrspace(101) %input1)
130: //      %call = call i32 @escape(ptr %input1.gen)
131: //      ret void
132: //    }
133: //
134: // TODO: merge this pass with NVPTXInferAddressSpaces so that other passes don't
135: // cancel the addrspacecast pair this pass emits.
136: //===----------------------------------------------------------------------===//
137:
138: #include "NVPTX.h"
139: #include "NVPTXTargetMachine.h"
140: #include "NVPTXUtilities.h"
141: #include "NVVMProperties.h"
142: #include "llvm/ADT/STLExtras.h"
143: #include "llvm/ADT/SmallVectorExtras.h"
144: #include "llvm/Analysis/PtrUseVisitor.h"
145: #include "llvm/CodeGen/TargetPassConfig.h"
146: #include "llvm/IR/Attributes.h"
147: #include "llvm/IR/Function.h"
148: #include "llvm/IR/IRBuilder.h"
149: #include "llvm/IR/Instructions.h"
150: #include "llvm/IR/IntrinsicInst.h"
151: #include "llvm/IR/IntrinsicsNVPTX.h"
152: #include "llvm/IR/Type.h"
153: #include "llvm/InitializePasses.h"
154: #include "llvm/Pass.h"
155: #include "llvm/Support/Debug.h"
156: #include "llvm/Support/ErrorHandling.h"
157: #include "llvm/Support/NVPTXAddrSpace.h"
158: #include "llvm/Support/NVVMAttributes.h"
159:
160: #define DEBUG_TYPE "nvptx-lower-args"
161:
162: using namespace llvm;
163: using namespace NVPTXAS;
164:
165: namespace {
166: class NVPTXLowerArgsLegacyPass : public FunctionPass {
167:   bool runOnFunction(Function &F) override;
168:
169: public:
170:   static char ID; // Pass identification, replacement for typeid
171:   NVPTXLowerArgsLegacyPass() : FunctionPass(ID) {}
172:   StringRef getPassName() const override {
173:     return "Lower pointer arguments of CUDA kernels";
174:   }
175:   void getAnalysisUsage(AnalysisUsage &AU) const override {
176:     AU.addRequired<TargetPassConfig>();
177:   }
178: };
179: } // namespace
180:
```
- EN: This range defines or declares important types such as NVPTXLowerArgsLegacyPass, runOnFunction, getPassName, getAnalysisUsage, shaping the data model used by NVPTXLowerArgs.cpp.
- CN: 这一段定义或声明了 NVPTXLowerArgsLegacyPass、runOnFunction、getPassName、getAnalysisUsage 等关键类型，构成 NVPTXLowerArgs.cpp 使用的数据模型。

### Lines 181-240
```cpp
181: char NVPTXLowerArgsLegacyPass::ID = 1;
182:
183: INITIALIZE_PASS_BEGIN(NVPTXLowerArgsLegacyPass, "nvptx-lower-args",
184:                       "Lower arguments (NVPTX)", false, false)
185: INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
186: INITIALIZE_PASS_END(NVPTXLowerArgsLegacyPass, "nvptx-lower-args",
187:                     "Lower arguments (NVPTX)", false, false)
188:
189: // =============================================================================
190: // If the function had a byval struct ptr arg, say foo(ptr byval(%struct.x) %d),
191: // and we can't guarantee that the only accesses are loads,
192: // then add the following instructions to the first basic block:
193: //
194: // %temp = alloca %struct.x, align 8
195: // %tempd = addrspacecast ptr %d to ptr addrspace(101)
196: // %tv = load %struct.x, ptr addrspace(101) %tempd
197: // store %struct.x %tv, ptr %temp, align 8
198: //
199: // The above code allocates some space in the stack and copies the incoming
200: // struct from param space to local space.
201: // Then replace all occurrences of %d by %temp.
202: //
203: // In case we know that all users are GEPs or Loads, replace them with the same
204: // ones in parameter AS, so we can access them using ld.param.
205: // =============================================================================
206:
207: /// Recursively convert the users of a param to the param address space.
208: static void convertToParamAS(ArrayRef<Use *> OldUses, Value *Param) {
209:   struct IP {
210:     Use *OldUse;
211:     Value *NewParam;
212:   };
213:
214:   const auto CloneInstInParamAS = [](const IP &I) -> Value * {
215:     auto *OldInst = cast<Instruction>(I.OldUse->getUser());
216:     if (auto *LI = dyn_cast<LoadInst>(OldInst)) {
217:       LI->setOperand(0, I.NewParam);
218:       return LI;
219:     }
220:     if (auto *GEP = dyn_cast<GetElementPtrInst>(OldInst)) {
221:       SmallVector<Value *, 4> Indices(GEP->indices());
222:       auto *NewGEP = GetElementPtrInst::Create(
223:           GEP->getSourceElementType(), I.NewParam, Indices, GEP->getName(),
224:           GEP->getIterator());
225:       NewGEP->setNoWrapFlags(GEP->getNoWrapFlags());
226:       return NewGEP;
227:     }
228:     if (auto *BC = dyn_cast<BitCastInst>(OldInst)) {
229:       auto *NewBCType =
230:           PointerType::get(BC->getContext(), ADDRESS_SPACE_ENTRY_PARAM);
231:       return BitCastInst::Create(BC->getOpcode(), I.NewParam, NewBCType,
232:                                  BC->getName(), BC->getIterator());
233:     }
234:     if (auto *ASC = dyn_cast<AddrSpaceCastInst>(OldInst)) {
235:       assert(ASC->getDestAddressSpace() == ADDRESS_SPACE_ENTRY_PARAM);
236:       (void)ASC;
237:       // Just pass through the argument, the old ASC is no longer needed.
238:       return I.NewParam;
239:     }
240:     if (auto *MI = dyn_cast<MemTransferInst>(OldInst)) {
```
- EN: This range defines or declares important types such as arguments, INITIALIZE_PASS_DEPENDENCY, convertToParamAS, IP, shaping the data model used by NVPTXLowerArgs.cpp.
- CN: 这一段定义或声明了 arguments、INITIALIZE_PASS_DEPENDENCY、convertToParamAS、IP 等关键类型，构成 NVPTXLowerArgs.cpp 使用的数据模型。

### Lines 241-300
```cpp
241:       if (MI->getRawSource() == I.OldUse->get()) {
242:         // convert to memcpy/memmove from param space.
243:         IRBuilder<> Builder(OldInst);
244:         Intrinsic::ID ID = MI->getIntrinsicID();
245:
246:         CallInst *B = Builder.CreateMemTransferInst(
247:             ID, MI->getRawDest(), MI->getDestAlign(), I.NewParam,
248:             MI->getSourceAlign(), MI->getLength(), MI->isVolatile());
249:         for (unsigned I : {0, 1})
250:           if (uint64_t Bytes = MI->getParamDereferenceableBytes(I))
251:             B->addDereferenceableParamAttr(I, Bytes);
252:         return B;
253:       }
254:     }
255:
256:     llvm_unreachable("Unsupported instruction");
257:   };
258:
259:   auto ItemsToConvert =
260:       map_to_vector(OldUses, [=](Use *U) -> IP { return {U, Param}; });
261:   SmallVector<Instruction *> InstructionsToDelete;
262:
263:   while (!ItemsToConvert.empty()) {
264:     IP I = ItemsToConvert.pop_back_val();
265:     Value *NewInst = CloneInstInParamAS(I);
266:     Instruction *OldInst = cast<Instruction>(I.OldUse->getUser());
267:
268:     if (NewInst && NewInst != OldInst) {
269:       // We've created a new instruction. Queue users of the old instruction to
270:       // be converted and the instruction itself to be deleted. We can't delete
271:       // the old instruction yet, because it's still in use by a load somewhere.
272:       for (Use &U : OldInst->uses())
273:         ItemsToConvert.push_back({&U, NewInst});
274:
275:       InstructionsToDelete.push_back(OldInst);
276:     }
277:   }
278:
279:   // Now we know that all argument loads are using addresses in parameter space
280:   // and we can finally remove the old instructions in generic AS. Instructions
281:   // scheduled for removal should be processed in reverse order so the ones
282:   // closest to the load are deleted first. Otherwise they may still be in use.
283:   // E.g if we have Value = Load(BitCast(GEP(arg))), InstructionsToDelete will
284:   // have {GEP,BitCast}. GEP can't be deleted first, because it's still used by
285:   // the BitCast.
286:   for (Instruction *I : llvm::reverse(InstructionsToDelete))
287:     I->eraseFromParent();
288: }
289:
290: // Create a call to the nvvm_internal_addrspace_wrap intrinsic and set the
291: // alignment of the return value based on the alignment of the argument.
292: static CallInst *createNVVMInternalAddrspaceWrap(IRBuilder<> &IRB,
293:                                                  Argument &Arg) {
294:   CallInst *ArgInParam = IRB.CreateIntrinsic(
295:       Intrinsic::nvvm_internal_addrspace_wrap,
296:       {IRB.getPtrTy(ADDRESS_SPACE_ENTRY_PARAM), Arg.getType()}, &Arg, {},
297:       Arg.getName() + ".param");
298:
299:   if (MaybeAlign ParamAlign = Arg.getParamAlign())
300:     ArgInParam->addRetAttr(
```
- EN: This range implements operational logic in helpers such as Builder, getIntrinsicID, getSourceAlign, addDereferenceableParamAttr, translating backend policy into executable code.
- CN: 这一段实现了 Builder、getIntrinsicID、getSourceAlign、addDereferenceableParamAttr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:         Attribute::getWithAlignment(ArgInParam->getContext(), *ParamAlign));
302:
303:   Arg.addAttr(Attribute::get(Arg.getContext(), NVVMAttr::GridConstant));
304:   Arg.addAttr(Attribute::ReadOnly);
305:
306:   return ArgInParam;
307: }
308:
309: namespace {
310: struct ArgUseChecker : PtrUseVisitor<ArgUseChecker> {
311:   using Base = PtrUseVisitor<ArgUseChecker>;
312:   // Set of phi/select instructions using the Arg
313:   SmallPtrSet<Instruction *, 4> Conditionals;
314:
315:   ArgUseChecker(const DataLayout &DL) : PtrUseVisitor(DL) {}
316:
317:   PtrInfo visitArgPtr(Argument &A) {
318:     assert(A.getType()->isPointerTy());
319:     IntegerType *IntIdxTy = cast<IntegerType>(DL.getIndexType(A.getType()));
320:     IsOffsetKnown = false;
321:     Offset = APInt(IntIdxTy->getBitWidth(), 0);
322:     PI.reset();
323:
324:     LLVM_DEBUG(dbgs() << "Checking Argument " << A << "\n");
325:     // Enqueue the uses of this pointer.
326:     enqueueUsers(A);
327:
328:     // Visit all the uses off the worklist until it is empty.
329:     // Note that unlike PtrUseVisitor we intentionally do not track offsets.
330:     // We're only interested in how we use the pointer.
331:     while (!(Worklist.empty() || PI.isAborted())) {
332:       UseToVisit ToVisit = Worklist.pop_back_val();
333:       U = ToVisit.UseAndIsOffsetKnown.getPointer();
334:       Instruction *I = cast<Instruction>(U->getUser());
335:       LLVM_DEBUG(dbgs() << "Processing " << *I << "\n");
336:       Base::visit(I);
337:     }
338:     if (PI.isEscaped())
339:       LLVM_DEBUG(dbgs() << "Argument pointer escaped: " << *PI.getEscapingInst()
340:                         << "\n");
341:     else if (PI.isAborted())
342:       LLVM_DEBUG(dbgs() << "Pointer use needs a copy: " << *PI.getAbortingInst()
343:                         << "\n");
344:     LLVM_DEBUG(dbgs() << "Traversed " << Conditionals.size()
345:                       << " conditionals\n");
346:     return PI;
347:   }
348:
349:   void visitStoreInst(StoreInst &SI) {
350:     // Storing the pointer escapes it.
351:     if (U->get() == SI.getValueOperand())
352:       return PI.setEscapedAndAborted(&SI);
353:
354:     PI.setAborted(&SI);
355:   }
356:
357:   void visitAddrSpaceCastInst(AddrSpaceCastInst &ASC) {
358:     // ASC to param space are no-ops and do not need a copy
359:     if (ASC.getDestAddressSpace() != ADDRESS_SPACE_ENTRY_PARAM)
360:       return PI.setEscapedAndAborted(&ASC);
```
- EN: This range defines or declares important types such as Attribute::getWithAlignment, addAttr, ArgUseChecker, visitArgPtr, shaping the data model used by NVPTXLowerArgs.cpp.
- CN: 这一段定义或声明了 Attribute::getWithAlignment、addAttr、ArgUseChecker、visitArgPtr 等关键类型，构成 NVPTXLowerArgs.cpp 使用的数据模型。

### Lines 361-420
```cpp
361:     Base::visitAddrSpaceCastInst(ASC);
362:   }
363:
364:   void visitPtrToIntInst(PtrToIntInst &I) { Base::visitPtrToIntInst(I); }
365:
366:   void visitPHINodeOrSelectInst(Instruction &I) {
367:     assert(isa<PHINode>(I) || isa<SelectInst>(I));
368:     enqueueUsers(I);
369:     Conditionals.insert(&I);
370:   }
371:   // PHI and select just pass through the pointers.
372:   void visitPHINode(PHINode &PN) { visitPHINodeOrSelectInst(PN); }
373:   void visitSelectInst(SelectInst &SI) { visitPHINodeOrSelectInst(SI); }
374:
375:   // memcpy/memmove are OK when the pointer is source. We can convert them to
376:   // AS-specific memcpy.
377:   void visitMemTransferInst(MemTransferInst &II) {
378:     if (*U == II.getRawDest())
379:       PI.setAborted(&II);
380:   }
381:
382:   void visitMemSetInst(MemSetInst &II) { PI.setAborted(&II); }
383: }; // struct ArgUseChecker
384:
385: void copyByValParam(Function &F, Argument &Arg) {
386:   LLVM_DEBUG(dbgs() << "Creating a local copy of " << Arg << "\n");
387:   Type *ByValType = Arg.getParamByValType();
388:   const DataLayout &DL = F.getDataLayout();
389:   IRBuilder<> IRB(&F.getEntryBlock().front());
390:   AllocaInst *AllocA = IRB.CreateAlloca(ByValType, nullptr, Arg.getName());
391:   // Set the alignment to alignment of the byval parameter. This is because,
392:   // later load/stores assume that alignment, and we are going to replace
393:   // the use of the byval parameter with this alloca instruction.
394:   AllocA->setAlignment(
395:       Arg.getParamAlign().value_or(DL.getPrefTypeAlign(ByValType)));
396:   Arg.replaceAllUsesWith(AllocA);
397:
398:   Value *ArgInParamAS = createNVVMInternalAddrspaceWrap(IRB, Arg);
399:
400:   // Be sure to propagate alignment to this load; LLVM doesn't know that NVPTX
401:   // addrspacecast preserves alignment.  Since params are constant, this load
402:   // is definitely not volatile.
403:   const auto ArgSize = *AllocA->getAllocationSize(DL);
404:   IRB.CreateMemCpy(AllocA, AllocA->getAlign(), ArgInParamAS, AllocA->getAlign(),
405:                    ArgSize);
406: }
407: } // namespace
408:
409: static bool argIsProcessed(Argument *Arg) {
410:   if (Arg->use_empty())
411:     return true;
412:
413:   // If the argument is already wrapped, it was processed by this pass before.
414:   if (Arg->hasOneUse())
415:     if (const auto *II = dyn_cast<IntrinsicInst>(*Arg->user_begin()))
416:       if (II->getIntrinsicID() == Intrinsic::nvvm_internal_addrspace_wrap)
417:         return true;
418:
419:   return false;
420: }
```
- EN: This range defines or declares important types such as Base::visitAddrSpaceCastInst, visitPtrToIntInst, visitPHINodeOrSelectInst, assert, shaping the data model used by NVPTXLowerArgs.cpp.
- CN: 这一段定义或声明了 Base::visitAddrSpaceCastInst、visitPtrToIntInst、visitPHINodeOrSelectInst、assert 等关键类型，构成 NVPTXLowerArgs.cpp 使用的数据模型。

### Lines 421-480
```cpp
421:
422: static void lowerKernelByValParam(Argument *Arg, Function &F,
423:                                   const bool HasCvtaParam) {
424:   assert(isKernelFunction(F));
425:
426:   const DataLayout &DL = F.getDataLayout();
427:   IRBuilder<> IRB(&F.getEntryBlock().front());
428:
429:   if (argIsProcessed(Arg))
430:     return;
431:
432:   // (1) First check the easy case, if were able to trace through all the uses
433:   // and we can convert them all to param AS, then we'll do this.
434:   ArgUseChecker AUC(DL);
435:   ArgUseChecker::PtrInfo PI = AUC.visitArgPtr(*Arg);
436:   const bool ArgUseIsReadOnly = !(PI.isEscaped() || PI.isAborted());
437:   if (ArgUseIsReadOnly && AUC.Conditionals.empty()) {
438:     // Convert all loads and intermediate operations to use parameter AS and
439:     // skip creation of a local copy of the argument.
440:     SmallVector<Use *, 16> UsesToUpdate(llvm::make_pointer_range(Arg->uses()));
441:     Value *ArgInParamAS = createNVVMInternalAddrspaceWrap(IRB, *Arg);
442:     for (Use *U : UsesToUpdate)
443:       convertToParamAS(U, ArgInParamAS);
444:     return;
445:   }
446:
447:   // (2) If the argument is grid constant, we get to use the pointer directly.
448:   if (HasCvtaParam && (ArgUseIsReadOnly || isParamGridConstant(*Arg))) {
449:     LLVM_DEBUG(dbgs() << "Using non-copy pointer to " << *Arg << "\n");
450:
451:     // Cast argument to param address space. Because the backend will emit the
452:     // argument already in the param address space, we need to use the noop
453:     // intrinsic, this had the added benefit of preventing other optimizations
454:     // from folding away this pair of addrspacecasts.
455:     Instruction *ArgInParamAS = createNVVMInternalAddrspaceWrap(IRB, *Arg);
456:
457:     // Cast param address to generic address space.
458:     Value *GenericArg = IRB.CreateAddrSpaceCast(
459:         ArgInParamAS, IRB.getPtrTy(ADDRESS_SPACE_GENERIC),
460:         Arg->getName() + ".gen");
461:
462:     Arg->replaceAllUsesWith(GenericArg);
463:
464:     // Do not replace Arg in the cast to param space
465:     ArgInParamAS->setOperand(0, Arg);
466:     return;
467:   }
468:
469:   // (3) Otherwise we have to create a copy of the argument in local memory.
470:   copyByValParam(F, *Arg);
471: }
472:
473: // =============================================================================
474: // Main function for this pass.
475: // =============================================================================
476: static bool processFunction(Function &F, NVPTXTargetMachine &TM) {
477:   if (!isKernelFunction(F))
478:     return false;
479:
480:   const NVPTXSubtarget *ST = TM.getSubtargetImpl(F);
```
- EN: This range implements operational logic in helpers such as assert, getDataLayout, IRB, AUC, translating backend policy into executable code.
- CN: 这一段实现了 assert、getDataLayout、IRB、AUC 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-528
```cpp
481:   const bool HasCvtaParam = ST->hasCvtaParam();
482:
483:   LLVM_DEBUG(dbgs() << "Lowering kernel args of " << F.getName() << "\n");
484:   bool Changed = false;
485:   for (Argument &Arg : F.args())
486:     if (Arg.hasByValAttr()) {
487:       lowerKernelByValParam(&Arg, F, HasCvtaParam);
488:       Changed = true;
489:     }
490:
491:   return Changed;
492: }
493:
494: bool NVPTXLowerArgsLegacyPass::runOnFunction(Function &F) {
495:   auto &TM = getAnalysis<TargetPassConfig>().getTM<NVPTXTargetMachine>();
496:   return processFunction(F, TM);
497: }
498:
499: FunctionPass *llvm::createNVPTXLowerArgsPass() {
500:   return new NVPTXLowerArgsLegacyPass();
501: }
502:
503: static bool copyFunctionByValArgs(Function &F) {
504:   LLVM_DEBUG(dbgs() << "Creating a copy of byval args of " << F.getName()
505:                     << "\n");
506:   bool Changed = false;
507:   if (isKernelFunction(F)) {
508:     for (Argument &Arg : F.args())
509:       if (Arg.hasByValAttr() && !isParamGridConstant(Arg)) {
510:         copyByValParam(F, Arg);
511:         Changed = true;
512:       }
513:   }
514:   return Changed;
515: }
516:
517: PreservedAnalyses NVPTXCopyByValArgsPass::run(Function &F,
518:                                               FunctionAnalysisManager &AM) {
519:   return copyFunctionByValArgs(F) ? PreservedAnalyses::none()
520:                                   : PreservedAnalyses::all();
521: }
522:
523: PreservedAnalyses NVPTXLowerArgsPass::run(Function &F,
524:                                           FunctionAnalysisManager &AM) {
525:   auto &NTM = static_cast<NVPTXTargetMachine &>(TM);
526:   bool Changed = processFunction(F, NTM);
527:   return Changed ? PreservedAnalyses::none() : PreservedAnalyses::all();
528: }
```
- EN: This range implements operational logic in helpers such as hasCvtaParam, LLVM_DEBUG, lowerKernelByValParam, NVPTXLowerArgsLegacyPass::runOnFunction, translating backend policy into executable code.
- CN: 这一段实现了 hasCvtaParam、LLVM_DEBUG、lowerKernelByValParam、NVPTXLowerArgsLegacyPass::runOnFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXLowerArgsLegacyPass, runOnFunction, getPassName, getAnalysisUsage, arguments, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXLowerArgsLegacyPass, runOnFunction, getPassName, getAnalysisUsage, arguments，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVPTXTargetMachine.h`
  - `NVPTXUtilities.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/SmallVectorExtras.h`
  - `llvm/Analysis/PtrUseVisitor.h`
  - `llvm/CodeGen/TargetPassConfig.h`
  - `llvm/IR/Attributes.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/IR/Type.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/Debug.h`
  - `llvm/Support/ErrorHandling.h`
  - `llvm/Support/NVPTXAddrSpace.h`
