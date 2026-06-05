# DXILOpLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILOpLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILOpLowering support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===- DXILOpLowering.cpp - Lowering to DXIL operations -------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILOpLowering.h"
10: #include "DXILConstants.h"
11: #include "DXILOpBuilder.h"
12: #include "DXILRootSignature.h"
13: #include "DXILShaderFlags.h"
14: #include "DirectX.h"
15: #include "llvm/ADT/SmallVector.h"
16: #include "llvm/Analysis/DXILMetadataAnalysis.h"
17: #include "llvm/Analysis/DXILResource.h"
18: #include "llvm/CodeGen/Passes.h"
19: #include "llvm/IR/Constant.h"
20: #include "llvm/IR/DiagnosticInfo.h"
21: #include "llvm/IR/IRBuilder.h"
22: #include "llvm/IR/Instruction.h"
23: #include "llvm/IR/Instructions.h"
24: #include "llvm/IR/Intrinsics.h"
25: #include "llvm/IR/IntrinsicsDirectX.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/IR/PassManager.h"
28: #include "llvm/IR/Use.h"
29: #include "llvm/InitializePasses.h"
30: #include "llvm/Pass.h"
31: #include "llvm/Support/ErrorHandling.h"
32: #include "llvm/Support/FormatVariadic.h"
33:
34: #define DEBUG_TYPE "dxil-op-lower"
35:
36: using namespace llvm;
37: using namespace llvm::dxil;
38:
39: namespace {
40: class OpLowerer {
41:   Module &M;
42:   DXILOpBuilder OpBuilder;
43:   DXILResourceMap &DRM;
44:   DXILResourceTypeMap &DRTM;
45:   const ModuleMetadataInfo &MMDI;
46:   SmallVector<CallInst *> CleanupCasts;
47:   Function *CleanupNURI = nullptr;
48:
49: public:
50:   OpLowerer(Module &M, DXILResourceMap &DRM, DXILResourceTypeMap &DRTM,
51:             const ModuleMetadataInfo &MMDI)
52:       : M(M), OpBuilder(M), DRM(DRM), DRTM(DRTM), MMDI(MMDI) {}
53:
54:   /// Replace every call to \c F using \c ReplaceCall, and then erase \c F. If
55:   /// there is an error replacing a call, we emit a diagnostic and return true.
56:   [[nodiscard]] bool
57:   replaceFunction(Function &F,
58:                   llvm::function_ref<Error(CallInst *CI)> ReplaceCall) {
59:     for (User *U : make_early_inc_range(F.users())) {
60:       CallInst *CI = dyn_cast<CallInst>(U);
61:       if (!CI)
62:         continue;
63:
64:       if (Error E = ReplaceCall(CI)) {
65:         std::string Message(toString(std::move(E)));
66:         M.getContext().diagnose(DiagnosticInfoUnsupported(
67:             *CI->getFunction(), Message, CI->getDebugLoc()));
68:
69:         return true;
70:       }
71:     }
72:     if (F.user_empty())
73:       F.eraseFromParent();
74:     return false;
75:   }
76:
77:   struct IntrinArgSelect {
78:     enum class Type {
79: #define DXIL_OP_INTRINSIC_ARG_SELECT_TYPE(name) name,
80: #include "DXILOperation.inc"
81:     };
82:     Type Type;
83:     int Value;
84:   };
85:
86:   /// Replaces uses of a struct with uses of an equivalent named struct.
87:   ///
88:   /// DXIL operations that return structs give them well known names, so we need
89:   /// to update uses when we switch from an LLVM intrinsic to an op.
90:   Error replaceNamedStructUses(CallInst *Intrin, CallInst *DXILOp) {
```
- EN: This range defines or declares important types such as OpLowerer, M, Error, Message, shaping the data model used by DXILOpLowering.cpp.
- CN: 这一段定义或声明了 OpLowerer、M、Error、Message 等关键类型，构成 DXILOpLowering.cpp 使用的数据模型。

### Lines 91-180
```cpp
 91:     auto *IntrinTy = cast<StructType>(Intrin->getType());
 92:     auto *DXILOpTy = cast<StructType>(DXILOp->getType());
 93:     if (!IntrinTy->isLayoutIdentical(DXILOpTy))
 94:       return make_error<StringError>(
 95:           "Type mismatch between intrinsic and DXIL op",
 96:           inconvertibleErrorCode());
 97:
 98:     for (Use &U : make_early_inc_range(Intrin->uses()))
 99:       if (auto *EVI = dyn_cast<ExtractValueInst>(U.getUser()))
100:         EVI->setOperand(0, DXILOp);
101:       else if (auto *IVI = dyn_cast<InsertValueInst>(U.getUser()))
102:         IVI->setOperand(0, DXILOp);
103:       else
104:         return make_error<StringError>("DXIL ops that return structs may only "
105:                                        "be used by insert- and extractvalue",
106:                                        inconvertibleErrorCode());
107:     return Error::success();
108:   }
109:
110:   bool isFast(FastMathFlags Flags) {
111:     // HLSL Fast Math doesn't enable AllowContract flag; This can be
112:     // removed when we enable it in the future.
113:     return Flags.allowReassoc() && Flags.noNaNs() && Flags.noInfs() &&
114:            Flags.noSignedZeros() && Flags.allowReciprocal() &&
115:            Flags.approxFunc();
116:   }
117:
118:   void setDxPrecise(CallInst *CI) {
119:     const StringRef Key = "dx.precise";
120:     Module *M = CI->getModule();
121:
122:     LLVMContext &Ctx = M->getContext();
123:     MDNode *One =
124:         llvm::MDNode::get(Ctx, ConstantAsMetadata::get(ConstantInt::get(
125:                                    llvm::Type::getInt32Ty(Ctx), 1)));
126:
127:     CI->setMetadata(Key, One);
128:   }
129:
130:   [[nodiscard]] bool
131:   replaceFunctionWithOp(Function &F, dxil::OpCode DXILOp,
132:                         ArrayRef<IntrinArgSelect> ArgSelects) {
133:     return replaceFunction(F, [&](CallInst *CI) -> Error {
134:       OpBuilder.getIRB().SetInsertPoint(CI);
135:       SmallVector<Value *> Args;
136:       if (ArgSelects.size()) {
137:         for (const IntrinArgSelect &A : ArgSelects) {
138:           switch (A.Type) {
139:           case IntrinArgSelect::Type::Index:
140:             Args.push_back(CI->getArgOperand(A.Value));
141:             break;
142:           case IntrinArgSelect::Type::I8:
143:             Args.push_back(OpBuilder.getIRB().getInt8((uint8_t)A.Value));
144:             break;
145:           case IntrinArgSelect::Type::I32:
146:             Args.push_back(OpBuilder.getIRB().getInt32(A.Value));
147:             break;
148:           }
149:         }
150:       } else {
151:         Args.append(CI->arg_begin(), CI->arg_end());
152:       }
153:
154:       Expected<CallInst *> OpCall =
155:           OpBuilder.tryCreateOp(DXILOp, Args, CI->getName(), F.getReturnType());
156:       if (Error E = OpCall.takeError())
157:         return E;
158:
159:       if (isa<FPMathOperator>(CI) &&
160:           !isFast(cast<FPMathOperator>(CI)->getFastMathFlags()))
161:         setDxPrecise(*OpCall);
162:
163:       if (isa<StructType>(CI->getType())) {
164:         if (Error E = replaceNamedStructUses(CI, *OpCall))
165:           return E;
166:       } else
167:         CI->replaceAllUsesWith(*OpCall);
168:
169:       CI->eraseFromParent();
170:       return Error::success();
171:     });
172:   }
173:
174:   /// Create a cast between a `target("dx")` type and `dx.types.Handle`, which
175:   /// is intended to be removed by the end of lowering. This is used to allow
176:   /// lowering of ops which need to change their return or argument types in a
177:   /// piecemeal way - we can add the casts in to avoid updating all of the uses
178:   /// or defs, and by the end all of the casts will be redundant.
179:   Value *createTmpHandleCast(Value *V, Type *Ty) {
180:     CallInst *Cast = OpBuilder.getIRB().CreateIntrinsic(
```
- EN: This range implements operational logic in helpers such as getType, inconvertibleErrorCode, setOperand, Error::success, translating backend policy into executable code.
- CN: 这一段实现了 getType、inconvertibleErrorCode、setOperand、Error::success 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:         Intrinsic::dx_resource_casthandle, {Ty, V->getType()}, {V});
182:     CleanupCasts.push_back(Cast);
183:     return Cast;
184:   }
185:
186:   void cleanupHandleCasts() {
187:     SmallVector<CallInst *> ToRemove;
188:     SmallVector<Function *> CastFns;
189:
190:     for (CallInst *Cast : CleanupCasts) {
191:       // These casts were only put in to ease the move from `target("dx")` types
192:       // to `dx.types.Handle in a piecemeal way. At this point, all of the
193:       // non-cast uses should now be `dx.types.Handle`, and remaining casts
194:       // should all form pairs to and from the now unused `target("dx")` type.
195:       CastFns.push_back(Cast->getCalledFunction());
196:
197:       // If the cast is not to `dx.types.Handle`, it should be the first part of
198:       // the pair. Keep track so we can remove it once it has no more uses.
199:       if (Cast->getType() != OpBuilder.getHandleType()) {
200:         ToRemove.push_back(Cast);
201:         continue;
202:       }
203:       // Otherwise, we're the second handle in a pair. Forward the arguments and
204:       // remove the (second) cast.
205:       CallInst *Def = cast<CallInst>(Cast->getOperand(0));
206:       assert(Def->getIntrinsicID() == Intrinsic::dx_resource_casthandle &&
207:              "Unbalanced pair of temporary handle casts");
208:       Cast->replaceAllUsesWith(Def->getOperand(0));
209:       Cast->eraseFromParent();
210:     }
211:     for (CallInst *Cast : ToRemove) {
212:       assert(Cast->user_empty() && "Temporary handle cast still has users");
213:       Cast->eraseFromParent();
214:     }
215:
216:     // Deduplicate the cast functions so that we only erase each one once.
217:     llvm::sort(CastFns);
218:     CastFns.erase(llvm::unique(CastFns), CastFns.end());
219:     for (Function *F : CastFns)
220:       F->eraseFromParent();
221:
222:     CleanupCasts.clear();
223:   }
224:
225:   void cleanupNonUniformResourceIndexCalls() {
226:     // Replace all NonUniformResourceIndex calls with their argument.
227:     if (!CleanupNURI)
228:       return;
229:     for (User *U : make_early_inc_range(CleanupNURI->users())) {
230:       CallInst *CI = dyn_cast<CallInst>(U);
231:       if (!CI)
232:         continue;
233:       CI->replaceAllUsesWith(CI->getArgOperand(0));
234:       CI->eraseFromParent();
235:     }
236:     CleanupNURI->eraseFromParent();
237:     CleanupNURI = nullptr;
238:   }
239:
240:   // Remove the resource global associated with the handleFromBinding call
241:   // instruction and their uses as they aren't needed anymore.
242:   // TODO: We should verify that all the globals get removed.
243:   // It's expected we'll need a custom pass in the future that will eliminate
244:   // the need for this here.
245:   void removeResourceGlobals(CallInst *CI) {
246:     for (User *User : make_early_inc_range(CI->users())) {
247:       if (StoreInst *Store = dyn_cast<StoreInst>(User)) {
248:         Value *V = Store->getOperand(1);
249:         Store->eraseFromParent();
250:         if (GlobalVariable *GV = dyn_cast<GlobalVariable>(V))
251:           if (GV->use_empty()) {
252:             GV->removeDeadConstantUsers();
253:             GV->eraseFromParent();
254:           }
255:       }
256:     }
257:   }
258:
259:   void replaceHandleFromBindingCall(CallInst *CI, Value *Replacement) {
260:     assert(CI->getCalledFunction()->getIntrinsicID() ==
261:            Intrinsic::dx_resource_handlefrombinding);
262:
263:     removeResourceGlobals(CI);
264:
265:     auto *NameGlobal = dyn_cast<llvm::GlobalVariable>(CI->getArgOperand(4));
266:
267:     CI->replaceAllUsesWith(Replacement);
268:     CI->eraseFromParent();
269:
270:     if (NameGlobal && NameGlobal->use_empty())
```
- EN: This range implements operational logic in helpers such as getType, push_back, cleanupHandleCasts, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getType、push_back、cleanupHandleCasts、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:       NameGlobal->removeFromParent();
272:   }
273:
274:   bool hasNonUniformIndex(Value *IndexOp) {
275:     if (isa<llvm::Constant>(IndexOp))
276:       return false;
277:
278:     SmallVector<Value *, 16> Worklist;
279:     SmallPtrSet<Value *, 16> Visited;
280:     Worklist.push_back(IndexOp);
281:
282:     while (!Worklist.empty()) {
283:       Value *V = Worklist.pop_back_val();
284:
285:       if (isa<llvm::Constant>(V))
286:         continue;
287:
288:       if (!Visited.insert(V).second)
289:         continue;
290:
291:       if (auto *CI = dyn_cast<CallInst>(V))
292:         if (CI->getIntrinsicID() == Intrinsic::dx_resource_nonuniformindex)
293:           return true;
294:
295:       // If it's a PHI node, check ALL incoming values —
296:       // taint from ANY predecessor counts
297:       if (auto *Phi = dyn_cast<PHINode>(V)) {
298:         for (Value *Incoming : Phi->incoming_values())
299:           Worklist.push_back(Incoming);
300:         continue;
301:       }
302:
303:       if (auto *Inst = dyn_cast<Instruction>(V))
304:         if (Inst->getNumOperands() > 0 && !Inst->isTerminator())
305:           for (Value *Op : Inst->operands())
306:             Worklist.push_back(Op);
307:     }
308:     return false;
309:   }
310:
311:   Error validateRawBufferElementIndex(Value *Resource, Value *ElementIndex) {
312:     bool IsStructured =
313:         cast<RawBufferExtType>(Resource->getType())->isStructured();
314:     bool IsPoison = isa<PoisonValue>(ElementIndex);
315:
316:     if (IsStructured && IsPoison)
317:       return make_error<StringError>(
318:           "Element index of structured buffer may not be poison",
319:           inconvertibleErrorCode());
320:
321:     if (!IsStructured && !IsPoison)
322:       return make_error<StringError>(
323:           "Element index of raw buffer must be poison",
324:           inconvertibleErrorCode());
325:
326:     return Error::success();
327:   }
328:
329:   [[nodiscard]] bool lowerToCreateHandle(Function &F) {
330:     IRBuilder<> &IRB = OpBuilder.getIRB();
331:     Type *Int8Ty = IRB.getInt8Ty();
332:     Type *Int32Ty = IRB.getInt32Ty();
333:     Type *Int1Ty = IRB.getInt1Ty();
334:
335:     return replaceFunction(F, [&](CallInst *CI) -> Error {
336:       IRB.SetInsertPoint(CI);
337:
338:       auto *It = DRM.find(CI);
339:       assert(It != DRM.end() && "Resource not in map?");
340:       dxil::ResourceInfo &RI = *It;
341:
342:       const auto &Binding = RI.getBinding();
343:       dxil::ResourceClass RC = DRTM[RI.getHandleTy()].getResourceClass();
344:
345:       Value *IndexOp = CI->getArgOperand(3);
346:       if (Binding.LowerBound != 0)
347:         IndexOp = IRB.CreateAdd(IndexOp,
348:                                 ConstantInt::get(Int32Ty, Binding.LowerBound));
349:
350:       bool HasNonUniformIndex =
351:           (Binding.Size == 1) ? false : hasNonUniformIndex(IndexOp);
352:       std::array<Value *, 4> Args{
353:           ConstantInt::get(Int8Ty, llvm::to_underlying(RC)),
354:           ConstantInt::get(Int32Ty, Binding.RecordID), IndexOp,
355:           ConstantInt::get(Int1Ty, HasNonUniformIndex)};
356:       Expected<CallInst *> OpCall =
357:           OpBuilder.tryCreateOp(OpCode::CreateHandle, Args, CI->getName());
358:       if (Error E = OpCall.takeError())
359:         return E;
360:
```
- EN: This range implements operational logic in helpers such as removeFromParent, hasNonUniformIndex, push_back, pop_back_val, translating backend policy into executable code.
- CN: 这一段实现了 removeFromParent、hasNonUniformIndex、push_back、pop_back_val 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:       Value *Cast = createTmpHandleCast(*OpCall, CI->getType());
362:       replaceHandleFromBindingCall(CI, Cast);
363:       return Error::success();
364:     });
365:   }
366:
367:   [[nodiscard]] bool lowerToBindAndAnnotateHandle(Function &F) {
368:     IRBuilder<> &IRB = OpBuilder.getIRB();
369:     Type *Int32Ty = IRB.getInt32Ty();
370:     Type *Int1Ty = IRB.getInt1Ty();
371:
372:     return replaceFunction(F, [&](CallInst *CI) -> Error {
373:       IRB.SetInsertPoint(CI);
374:
375:       auto *It = DRM.find(CI);
376:       assert(It != DRM.end() && "Resource not in map?");
377:       dxil::ResourceInfo &RI = *It;
378:
379:       const auto &Binding = RI.getBinding();
380:       dxil::ResourceTypeInfo &RTI = DRTM[RI.getHandleTy()];
381:       dxil::ResourceClass RC = RTI.getResourceClass();
382:
383:       Value *IndexOp = CI->getArgOperand(3);
384:       if (Binding.LowerBound != 0)
385:         IndexOp = IRB.CreateAdd(IndexOp,
386:                                 ConstantInt::get(Int32Ty, Binding.LowerBound));
387:
388:       std::pair<uint32_t, uint32_t> Props =
389:           RI.getAnnotateProps(*F.getParent(), RTI);
390:
391:       // For `CreateHandleFromBinding` we need the upper bound rather than the
392:       // size, so we need to be careful about the difference for "unbounded".
393:       uint32_t UpperBound = Binding.Size == 0
394:                                 ? std::numeric_limits<uint32_t>::max()
395:                                 : Binding.LowerBound + Binding.Size - 1;
396:       Constant *ResBind = OpBuilder.getResBind(Binding.LowerBound, UpperBound,
397:                                                Binding.Space, RC);
398:       bool NonUniformIndex =
399:           (Binding.Size == 1) ? false : hasNonUniformIndex(IndexOp);
400:       Constant *NonUniformOp = ConstantInt::get(Int1Ty, NonUniformIndex);
401:       std::array<Value *, 3> BindArgs{ResBind, IndexOp, NonUniformOp};
402:       Expected<CallInst *> OpBind = OpBuilder.tryCreateOp(
403:           OpCode::CreateHandleFromBinding, BindArgs, CI->getName());
404:       if (Error E = OpBind.takeError())
405:         return E;
406:
407:       std::array<Value *, 2> AnnotateArgs{
408:           *OpBind, OpBuilder.getResProps(Props.first, Props.second)};
409:       Expected<CallInst *> OpAnnotate = OpBuilder.tryCreateOp(
410:           OpCode::AnnotateHandle, AnnotateArgs,
411:           CI->hasName() ? CI->getName() + "_annot" : Twine());
412:       if (Error E = OpAnnotate.takeError())
413:         return E;
414:
415:       Value *Cast = createTmpHandleCast(*OpAnnotate, CI->getType());
416:       replaceHandleFromBindingCall(CI, Cast);
417:       return Error::success();
418:     });
419:   }
420:
421:   /// Lower `dx.resource.handlefrombinding` intrinsics depending on the shader
422:   /// model and taking into account binding information from
423:   /// DXILResourceAnalysis.
424:   bool lowerHandleFromBinding(Function &F) {
425:     if (MMDI.DXILVersion < VersionTuple(1, 6))
426:       return lowerToCreateHandle(F);
427:     return lowerToBindAndAnnotateHandle(F);
428:   }
429:
430:   /// Replace uses of \c Intrin with the values in the `dx.ResRet` of \c Op.
431:   /// Since we expect to be post-scalarization, make an effort to avoid vectors.
432:   Error replaceResRetUses(CallInst *Intrin, CallInst *Op, bool HasCheckBit) {
433:     IRBuilder<> &IRB = OpBuilder.getIRB();
434:
435:     Instruction *OldResult = Intrin;
436:     Type *OldTy = Intrin->getType();
437:
438:     if (HasCheckBit) {
439:       auto *ST = cast<StructType>(OldTy);
440:
441:       Value *CheckOp = nullptr;
442:       Type *Int32Ty = IRB.getInt32Ty();
443:       for (Use &U : make_early_inc_range(OldResult->uses())) {
444:         if (auto *EVI = dyn_cast<ExtractValueInst>(U.getUser())) {
445:           ArrayRef<unsigned> Indices = EVI->getIndices();
446:           assert(Indices.size() == 1);
447:           // We're only interested in uses of the check bit for now.
448:           if (Indices[0] != 1)
449:             continue;
450:           if (!CheckOp) {
```
- EN: This range implements operational logic in helpers such as createTmpHandleCast, replaceHandleFromBindingCall, Error::success, lowerToBindAndAnnotateHandle, translating backend policy into executable code.
- CN: 这一段实现了 createTmpHandleCast、replaceHandleFromBindingCall、Error::success、lowerToBindAndAnnotateHandle 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:             Value *NewEVI = IRB.CreateExtractValue(Op, 4);
452:             Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
453:                 OpCode::CheckAccessFullyMapped, {NewEVI},
454:                 OldResult->hasName() ? OldResult->getName() + "_check"
455:                                      : Twine(),
456:                 Int32Ty);
457:             if (Error E = OpCall.takeError())
458:               return E;
459:             CheckOp = *OpCall;
460:           }
461:           EVI->replaceAllUsesWith(CheckOp);
462:           EVI->eraseFromParent();
463:         }
464:       }
465:
466:       if (OldResult->use_empty()) {
467:         // Only the check bit was used, so we're done here.
468:         OldResult->eraseFromParent();
469:         return Error::success();
470:       }
471:
472:       assert(OldResult->hasOneUse() &&
473:              isa<ExtractValueInst>(*OldResult->user_begin()) &&
474:              "Expected only use to be extract of first element");
475:       OldResult = cast<Instruction>(*OldResult->user_begin());
476:       OldTy = ST->getElementType(0);
477:     }
478:
479:     // For scalars, we just extract the first element.
480:     if (!isa<FixedVectorType>(OldTy)) {
481:       Value *EVI = IRB.CreateExtractValue(Op, 0);
482:       OldResult->replaceAllUsesWith(EVI);
483:       OldResult->eraseFromParent();
484:       if (OldResult != Intrin) {
485:         assert(Intrin->use_empty() && "Intrinsic still has uses?");
486:         Intrin->eraseFromParent();
487:       }
488:       return Error::success();
489:     }
490:
491:     std::array<Value *, 4> Extracts = {};
492:     SmallVector<ExtractElementInst *> DynamicAccesses;
493:
494:     // The users of the operation should all be scalarized, so we attempt to
495:     // replace the extractelements with extractvalues directly.
496:     for (Use &U : make_early_inc_range(OldResult->uses())) {
497:       if (auto *EEI = dyn_cast<ExtractElementInst>(U.getUser())) {
498:         if (auto *IndexOp = dyn_cast<ConstantInt>(EEI->getIndexOperand())) {
499:           size_t IndexVal = IndexOp->getZExtValue();
500:           assert(IndexVal < 4 && "Index into buffer load out of range");
501:           if (!Extracts[IndexVal])
502:             Extracts[IndexVal] = IRB.CreateExtractValue(Op, IndexVal);
503:           EEI->replaceAllUsesWith(Extracts[IndexVal]);
504:           EEI->eraseFromParent();
505:         } else {
506:           DynamicAccesses.push_back(EEI);
507:         }
508:       }
509:     }
510:
511:     const auto *VecTy = cast<FixedVectorType>(OldTy);
512:     const unsigned N = VecTy->getNumElements();
513:
514:     // If there's a dynamic access we need to round trip through stack memory so
515:     // that we don't leave vectors around.
516:     if (!DynamicAccesses.empty()) {
517:       Type *Int32Ty = IRB.getInt32Ty();
518:       Constant *Zero = ConstantInt::get(Int32Ty, 0);
519:
520:       Type *ElTy = VecTy->getElementType();
521:       Type *ArrayTy = ArrayType::get(ElTy, N);
522:       Value *Alloca = IRB.CreateAlloca(ArrayTy);
523:
524:       for (int I = 0, E = N; I != E; ++I) {
525:         if (!Extracts[I])
526:           Extracts[I] = IRB.CreateExtractValue(Op, I);
527:         Value *GEP = IRB.CreateInBoundsGEP(
528:             ArrayTy, Alloca, {Zero, ConstantInt::get(Int32Ty, I)});
529:         IRB.CreateStore(Extracts[I], GEP);
530:       }
531:
532:       for (ExtractElementInst *EEI : DynamicAccesses) {
533:         Value *GEP = IRB.CreateInBoundsGEP(ArrayTy, Alloca,
534:                                            {Zero, EEI->getIndexOperand()});
535:         Value *Load = IRB.CreateLoad(ElTy, GEP);
536:         EEI->replaceAllUsesWith(Load);
537:         EEI->eraseFromParent();
538:       }
539:     }
540:
```
- EN: This range implements operational logic in helpers such as CreateExtractValue, replaceAllUsesWith, eraseFromParent, Error::success, translating backend policy into executable code.
- CN: 这一段实现了 CreateExtractValue、replaceAllUsesWith、eraseFromParent、Error::success 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:     // If we still have uses, then we're not fully scalarized and need to
542:     // recreate the vector. This should only happen for things like exported
543:     // functions from libraries.
544:     if (!OldResult->use_empty()) {
545:       for (int I = 0, E = N; I != E; ++I)
546:         if (!Extracts[I])
547:           Extracts[I] = IRB.CreateExtractValue(Op, I);
548:
549:       Value *Vec = PoisonValue::get(OldTy);
550:       for (int I = 0, E = N; I != E; ++I)
551:         Vec = IRB.CreateInsertElement(Vec, Extracts[I], I);
552:       OldResult->replaceAllUsesWith(Vec);
553:     }
554:
555:     OldResult->eraseFromParent();
556:     if (OldResult != Intrin) {
557:       assert(Intrin->use_empty() && "Intrinsic still has uses?");
558:       Intrin->eraseFromParent();
559:     }
560:
561:     return Error::success();
562:   }
563:
564:   [[nodiscard]] bool lowerTypedBufferLoad(Function &F, bool HasCheckBit) {
565:     IRBuilder<> &IRB = OpBuilder.getIRB();
566:     Type *Int32Ty = IRB.getInt32Ty();
567:
568:     return replaceFunction(F, [&](CallInst *CI) -> Error {
569:       IRB.SetInsertPoint(CI);
570:
571:       Value *Handle =
572:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
573:       Value *Index0 = CI->getArgOperand(1);
574:       Value *Index1 = UndefValue::get(Int32Ty);
575:
576:       Type *OldTy = CI->getType();
577:       if (HasCheckBit)
578:         OldTy = cast<StructType>(OldTy)->getElementType(0);
579:       Type *NewRetTy = OpBuilder.getResRetType(OldTy->getScalarType());
580:
581:       std::array<Value *, 3> Args{Handle, Index0, Index1};
582:       Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
583:           OpCode::BufferLoad, Args, CI->getName(), NewRetTy);
584:       if (Error E = OpCall.takeError())
585:         return E;
586:       if (Error E = replaceResRetUses(CI, *OpCall, HasCheckBit))
587:         return E;
588:
589:       return Error::success();
590:     });
591:   }
592:
593:   // Copies `Src` into `Args` starting at `ArgIdx`. If `Src` is a vector, its
594:   // elements are extracted and stored in consecutive slots; otherwise `Src`
595:   // is stored directly. At most `MaxElements` elements are expected.
596:   static void extractElementsIntoArgs(IRBuilder<> &IRB,
597:                                       MutableArrayRef<Value *> Args,
598:                                       unsigned ArgIdx, Value *Src,
599:                                       unsigned MaxElements) {
600:     Type *Ty = Src->getType();
601:     if (auto *VecTy = dyn_cast<FixedVectorType>(Ty)) {
602:       unsigned Count = VecTy->getNumElements();
603:       assert(Count <= MaxElements && "Expected at most 3 elements in vector");
604:       for (unsigned I = 0; I < Count; ++I)
605:         Args[ArgIdx + I] = IRB.CreateExtractElement(Src, uint64_t(I));
606:     } else {
607:       Args[ArgIdx] = Src;
608:     }
609:   }
610:
611:   [[nodiscard]] bool lowerTextureLoad(Function &F) {
612:     IRBuilder<> &IRB = OpBuilder.getIRB();
613:     Type *Int32Ty = IRB.getInt32Ty();
614:
615:     return replaceFunction(F, [&](CallInst *CI) -> Error {
616:       IRB.SetInsertPoint(CI);
617:
618:       Value *Handle =
619:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
620:       Value *Coords = CI->getArgOperand(1);
621:       Value *MipLevel = CI->getArgOperand(2);
622:       Value *Offsets = CI->getArgOperand(3);
623:
624:       Type *OldTy = CI->getType();
625:       Type *NewRetTy = OpBuilder.getResRetType(OldTy->getScalarType());
626:
627:       Value *Undef = UndefValue::get(Int32Ty);
628:       std::array<Value *, 8> Args{Handle, MipLevel, Undef, Undef,
629:                                   Undef,  Undef,    Undef, Undef};
630:
```
- EN: This range implements operational logic in helpers such as CreateExtractValue, PoisonValue::get, CreateInsertElement, replaceAllUsesWith, translating backend policy into executable code.
- CN: 这一段实现了 CreateExtractValue、PoisonValue::get、CreateInsertElement、replaceAllUsesWith 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:       // Copy coordinates and offsets into Args.
632:       extractElementsIntoArgs(IRB, Args, 2, Coords, 3);
633:       if (auto *C = dyn_cast<Constant>(Offsets); !C || !C->isNullValue())
634:         extractElementsIntoArgs(IRB, Args, 5, Offsets, 3);
635:
636:       Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
637:           OpCode::TextureLoad, Args, CI->getName(), NewRetTy);
638:       if (Error E = OpCall.takeError())
639:         return E;
640:       if (Error E = replaceResRetUses(CI, *OpCall, /*HasCheckBit=*/false))
641:         return E;
642:
643:       return Error::success();
644:     });
645:   }
646:
647:   [[nodiscard]] bool lowerRawBufferLoad(Function &F) {
648:     const DataLayout &DL = F.getDataLayout();
649:     IRBuilder<> &IRB = OpBuilder.getIRB();
650:     Type *Int8Ty = IRB.getInt8Ty();
651:     Type *Int32Ty = IRB.getInt32Ty();
652:
653:     return replaceFunction(F, [&](CallInst *CI) -> Error {
654:       IRB.SetInsertPoint(CI);
655:
656:       Type *OldTy = cast<StructType>(CI->getType())->getElementType(0);
657:       Type *ScalarTy = OldTy->getScalarType();
658:       Type *NewRetTy = OpBuilder.getResRetType(ScalarTy);
659:
660:       Value *Handle =
661:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
662:       Value *Index0 = CI->getArgOperand(1);
663:       Value *Index1 = CI->getArgOperand(2);
664:       uint64_t NumElements =
665:           DL.getTypeSizeInBits(OldTy) / DL.getTypeSizeInBits(ScalarTy);
666:       Value *Mask = ConstantInt::get(Int8Ty, ~(~0U << NumElements));
667:       Value *Align =
668:           ConstantInt::get(Int32Ty, DL.getPrefTypeAlign(ScalarTy).value());
669:
670:       if (Error E = validateRawBufferElementIndex(CI->getOperand(0), Index1))
671:         return E;
672:       if (isa<PoisonValue>(Index1))
673:         Index1 = UndefValue::get(Index1->getType());
674:
675:       Expected<CallInst *> OpCall =
676:           MMDI.DXILVersion >= VersionTuple(1, 2)
677:               ? OpBuilder.tryCreateOp(OpCode::RawBufferLoad,
678:                                       {Handle, Index0, Index1, Mask, Align},
679:                                       CI->getName(), NewRetTy)
680:               : OpBuilder.tryCreateOp(OpCode::BufferLoad,
681:                                       {Handle, Index0, Index1}, CI->getName(),
682:                                       NewRetTy);
683:       if (Error E = OpCall.takeError())
684:         return E;
685:       if (Error E = replaceResRetUses(CI, *OpCall, /*HasCheckBit=*/true))
686:         return E;
687:
688:       return Error::success();
689:     });
690:   }
691:
692:   [[nodiscard]] bool lowerCBufferLoad(Function &F) {
693:     IRBuilder<> &IRB = OpBuilder.getIRB();
694:
695:     return replaceFunction(F, [&](CallInst *CI) -> Error {
696:       IRB.SetInsertPoint(CI);
697:
698:       Type *OldTy = cast<StructType>(CI->getType())->getElementType(0);
699:       Type *ScalarTy = OldTy->getScalarType();
700:       Type *NewRetTy = OpBuilder.getCBufRetType(ScalarTy);
701:
702:       Value *Handle =
703:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
704:       Value *Index = CI->getArgOperand(1);
705:
706:       Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
707:           OpCode::CBufferLoadLegacy, {Handle, Index}, CI->getName(), NewRetTy);
708:       if (Error E = OpCall.takeError())
709:         return E;
710:       if (Error E = replaceNamedStructUses(CI, *OpCall))
711:         return E;
712:
713:       CI->eraseFromParent();
714:       return Error::success();
715:     });
716:   }
717:
718:   [[nodiscard]] bool lowerUpdateCounter(Function &F) {
719:     IRBuilder<> &IRB = OpBuilder.getIRB();
720:     Type *Int32Ty = IRB.getInt32Ty();
```
- EN: This range implements operational logic in helpers such as extractElementsIntoArgs, getName, Error::success, lowerRawBufferLoad, translating backend policy into executable code.
- CN: 这一段实现了 extractElementsIntoArgs、getName、Error::success、lowerRawBufferLoad 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:
722:     return replaceFunction(F, [&](CallInst *CI) -> Error {
723:       IRB.SetInsertPoint(CI);
724:       Value *Handle =
725:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
726:       Value *Op1 = CI->getArgOperand(1);
727:
728:       std::array<Value *, 2> Args{Handle, Op1};
729:
730:       Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
731:           OpCode::UpdateCounter, Args, CI->getName(), Int32Ty);
732:
733:       if (Error E = OpCall.takeError())
734:         return E;
735:
736:       CI->replaceAllUsesWith(*OpCall);
737:       CI->eraseFromParent();
738:       return Error::success();
739:     });
740:   }
741:
742:   [[nodiscard]] bool lowerGetDimensionsX(Function &F) {
743:     IRBuilder<> &IRB = OpBuilder.getIRB();
744:     Type *Int32Ty = IRB.getInt32Ty();
745:
746:     return replaceFunction(F, [&](CallInst *CI) -> Error {
747:       IRB.SetInsertPoint(CI);
748:       Value *Handle =
749:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
750:       Value *Undef = UndefValue::get(Int32Ty);
751:
752:       Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
753:           OpCode::GetDimensions, {Handle, Undef}, CI->getName(), Int32Ty);
754:       if (Error E = OpCall.takeError())
755:         return E;
756:       Value *Dim = IRB.CreateExtractValue(*OpCall, 0);
757:
758:       CI->replaceAllUsesWith(Dim);
759:       CI->eraseFromParent();
760:       return Error::success();
761:     });
762:   }
763:
764:   [[nodiscard]] bool lowerGetPointer(Function &F) {
765:     // These should have already been handled in DXILResourceAccess, so we can
766:     // just clean up the dead prototype.
767:     assert(F.user_empty() && "getpointer operations should have been removed");
768:     F.eraseFromParent();
769:     return false;
770:   }
771:
772:   [[nodiscard]] bool lowerBufferStore(Function &F, bool IsRaw) {
773:     const DataLayout &DL = F.getDataLayout();
774:     IRBuilder<> &IRB = OpBuilder.getIRB();
775:     Type *Int8Ty = IRB.getInt8Ty();
776:     Type *Int32Ty = IRB.getInt32Ty();
777:
778:     return replaceFunction(F, [&](CallInst *CI) -> Error {
779:       IRB.SetInsertPoint(CI);
780:
781:       Value *Handle =
782:           createTmpHandleCast(CI->getArgOperand(0), OpBuilder.getHandleType());
783:       Value *Index0 = CI->getArgOperand(1);
784:       Value *Index1 = IsRaw ? CI->getArgOperand(2) : UndefValue::get(Int32Ty);
785:
786:       if (IsRaw) {
787:         if (Error E = validateRawBufferElementIndex(CI->getOperand(0), Index1))
788:           return E;
789:         if (isa<PoisonValue>(Index1))
790:           Index1 = UndefValue::get(Index1->getType());
791:       }
792:
793:       Value *Data = CI->getArgOperand(IsRaw ? 3 : 2);
794:       Type *DataTy = Data->getType();
795:       Type *ScalarTy = DataTy->getScalarType();
796:
797:       uint64_t NumElements =
798:           DL.getTypeSizeInBits(DataTy) / DL.getTypeSizeInBits(ScalarTy);
799:       Value *Mask =
800:           ConstantInt::get(Int8Ty, IsRaw ? ~(~0U << NumElements) : 15U);
801:
802:       // TODO: check that we only have vector or scalar...
803:       if (NumElements > 4)
804:         return make_error<StringError>(
805:             "Buffer store data must have at most 4 elements",
806:             inconvertibleErrorCode());
807:
808:       std::array<Value *, 4> DataElements{nullptr, nullptr, nullptr, nullptr};
809:       if (DataTy == ScalarTy)
810:         DataElements[0] = Data;
```
- EN: This range implements operational logic in helpers such as SetInsertPoint, createTmpHandleCast, getArgOperand, getName, translating backend policy into executable code.
- CN: 这一段实现了 SetInsertPoint、createTmpHandleCast、getArgOperand、getName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:       else {
812:         // Since we're post-scalarizer, if we see a vector here it's likely
813:         // constructed solely for the argument of the store. Just use the scalar
814:         // values from before they're inserted into the temporary.
815:         auto *IEI = dyn_cast<InsertElementInst>(Data);
816:         while (IEI) {
817:           auto *IndexOp = dyn_cast<ConstantInt>(IEI->getOperand(2));
818:           if (!IndexOp)
819:             break;
820:           size_t IndexVal = IndexOp->getZExtValue();
821:           assert(IndexVal < 4 && "Too many elements for buffer store");
822:           DataElements[IndexVal] = IEI->getOperand(1);
823:           IEI = dyn_cast<InsertElementInst>(IEI->getOperand(0));
824:         }
825:       }
826:
827:       // If for some reason we weren't able to forward the arguments from the
828:       // scalarizer artifact, then we may need to actually extract elements from
829:       // the vector.
830:       for (int I = 0, E = NumElements; I < E; ++I)
831:         if (DataElements[I] == nullptr)
832:           DataElements[I] =
833:               IRB.CreateExtractElement(Data, ConstantInt::get(Int32Ty, I));
834:
835:       // For any elements beyond the length of the vector, we should fill it up
836:       // with undef - however, for typed buffers we repeat the first element to
837:       // match DXC.
838:       for (int I = NumElements, E = 4; I < E; ++I)
839:         if (DataElements[I] == nullptr)
840:           DataElements[I] = IsRaw ? UndefValue::get(ScalarTy) : DataElements[0];
841:
842:       dxil::OpCode Op = OpCode::BufferStore;
843:       SmallVector<Value *, 9> Args{
844:           Handle,          Index0,          Index1,          DataElements[0],
845:           DataElements[1], DataElements[2], DataElements[3], Mask};
846:       if (IsRaw && MMDI.DXILVersion >= VersionTuple(1, 2)) {
847:         Op = OpCode::RawBufferStore;
848:         // RawBufferStore requires the alignment
849:         Args.push_back(
850:             ConstantInt::get(Int32Ty, DL.getPrefTypeAlign(ScalarTy).value()));
851:       }
852:       Expected<CallInst *> OpCall =
853:           OpBuilder.tryCreateOp(Op, Args, CI->getName());
854:       if (Error E = OpCall.takeError())
855:         return E;
856:
857:       CI->eraseFromParent();
858:       // Clean up any leftover `insertelement`s
859:       auto *IEI = dyn_cast<InsertElementInst>(Data);
860:       while (IEI && IEI->use_empty()) {
861:         InsertElementInst *Tmp = IEI;
862:         IEI = dyn_cast<InsertElementInst>(IEI->getOperand(0));
863:         Tmp->eraseFromParent();
864:       }
865:
866:       return Error::success();
867:     });
868:   }
869:
870:   [[nodiscard]] bool lowerCtpopToCountBits(Function &F) {
871:     IRBuilder<> &IRB = OpBuilder.getIRB();
872:     Type *Int32Ty = IRB.getInt32Ty();
873:
874:     return replaceFunction(F, [&](CallInst *CI) -> Error {
875:       IRB.SetInsertPoint(CI);
876:       SmallVector<Value *> Args;
877:       Args.append(CI->arg_begin(), CI->arg_end());
878:
879:       Type *RetTy = Int32Ty;
880:       Type *FRT = F.getReturnType();
881:       if (const auto *VT = dyn_cast<VectorType>(FRT))
882:         RetTy = VectorType::get(RetTy, VT);
883:
884:       Expected<CallInst *> OpCall = OpBuilder.tryCreateOp(
885:           dxil::OpCode::CountBits, Args, CI->getName(), RetTy);
886:       if (Error E = OpCall.takeError())
887:         return E;
888:
889:       // If the result type is 32 bits we can do a direct replacement.
890:       if (FRT->isIntOrIntVectorTy(32)) {
891:         CI->replaceAllUsesWith(*OpCall);
892:         CI->eraseFromParent();
893:         return Error::success();
894:       }
895:
896:       unsigned CastOp;
897:       unsigned CastOp2;
898:       if (FRT->isIntOrIntVectorTy(16)) {
899:         CastOp = Instruction::ZExt;
900:         CastOp2 = Instruction::SExt;
```
- EN: This range implements operational logic in helpers such as getOperand, getZExtValue, assert, CreateExtractElement, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getZExtValue、assert、CreateExtractElement 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:       } else { // must be 64 bits
902:         assert(FRT->isIntOrIntVectorTy(64) &&
903:                "Currently only lowering 16, 32, or 64 bit ctpop to CountBits \
904:                 is supported.");
905:         CastOp = Instruction::Trunc;
906:         CastOp2 = Instruction::Trunc;
907:       }
908:
909:       // It is correct to replace the ctpop with the dxil op and
910:       // remove all casts to i32
911:       bool NeedsCast = false;
912:       for (User *User : make_early_inc_range(CI->users())) {
913:         Instruction *I = dyn_cast<Instruction>(User);
914:         if (I && (I->getOpcode() == CastOp || I->getOpcode() == CastOp2) &&
915:             I->getType() == RetTy) {
916:           I->replaceAllUsesWith(*OpCall);
917:           I->eraseFromParent();
918:         } else
919:           NeedsCast = true;
920:       }
921:
922:       // It is correct to replace a ctpop with the dxil op and
923:       // a cast from i32 to the return type of the ctpop
924:       // the cast is emitted here if there is a non-cast to i32
925:       // instr which uses the ctpop
926:       if (NeedsCast) {
927:         Value *Cast =
928:             IRB.CreateZExtOrTrunc(*OpCall, F.getReturnType(), "ctpop.cast");
929:         CI->replaceAllUsesWith(Cast);
930:       }
931:
932:       CI->eraseFromParent();
933:       return Error::success();
934:     });
935:   }
936:
937:   [[nodiscard]] bool lowerLifetimeIntrinsic(Function &F) {
938:     IRBuilder<> &IRB = OpBuilder.getIRB();
939:     return replaceFunction(F, [&](CallInst *CI) -> Error {
940:       IRB.SetInsertPoint(CI);
941:       Value *Ptr = CI->getArgOperand(0);
942:       assert(Ptr->getType()->isPointerTy() &&
943:              "Expected operand of lifetime intrinsic to be a pointer");
944:
945:       auto ZeroOrUndef = [&](Type *Ty) {
946:         return MMDI.ValidatorVersion < VersionTuple(1, 6)
947:                    ? Constant::getNullValue(Ty)
948:                    : UndefValue::get(Ty);
949:       };
950:
951:       Value *Val = nullptr;
952:       if (auto *GV = dyn_cast<GlobalVariable>(Ptr)) {
953:         if (GV->hasInitializer() || GV->isExternallyInitialized())
954:           return Error::success();
955:         Val = ZeroOrUndef(GV->getValueType());
956:       } else if (auto *AI = dyn_cast<AllocaInst>(Ptr))
957:         Val = ZeroOrUndef(AI->getAllocatedType());
958:
959:       assert(Val && "Expected operand of lifetime intrinsic to be a global "
960:                     "variable or alloca instruction");
961:       IRB.CreateStore(Val, Ptr, false);
962:
963:       CI->eraseFromParent();
964:       return Error::success();
965:     });
966:   }
967:
968:   [[nodiscard]] bool lowerIsFPClass(Function &F) {
969:     IRBuilder<> &IRB = OpBuilder.getIRB();
970:     Type *RetTy = IRB.getInt1Ty();
971:
972:     return replaceFunction(F, [&](CallInst *CI) -> Error {
973:       IRB.SetInsertPoint(CI);
974:       SmallVector<Value *> Args;
975:       Value *Fl = CI->getArgOperand(0);
976:       Args.push_back(Fl);
977:
978:       dxil::OpCode OpCode;
979:       Value *T = CI->getArgOperand(1);
980:       auto *TCI = dyn_cast<ConstantInt>(T);
981:       switch (TCI->getZExtValue()) {
982:       case FPClassTest::fcInf:
983:         OpCode = dxil::OpCode::IsInf;
984:         break;
985:       case FPClassTest::fcNan:
986:         OpCode = dxil::OpCode::IsNaN;
987:         break;
988:       case FPClassTest::fcNormal:
989:         OpCode = dxil::OpCode::IsNormal;
990:         break;
```
- EN: This range implements operational logic in helpers such as getType, replaceAllUsesWith, eraseFromParent, CreateZExtOrTrunc, translating backend policy into executable code.
- CN: 这一段实现了 getType、replaceAllUsesWith、eraseFromParent、CreateZExtOrTrunc 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:       case FPClassTest::fcFinite:
 992:         OpCode = dxil::OpCode::IsFinite;
 993:         break;
 994:       default:
 995:         SmallString<128> Msg =
 996:             formatv("Unsupported FPClassTest {0} for DXIL Op Lowering",
 997:                     TCI->getZExtValue());
 998:         return make_error<StringError>(Msg, inconvertibleErrorCode());
 999:       }
1000:
1001:       Expected<CallInst *> OpCall =
1002:           OpBuilder.tryCreateOp(OpCode, Args, CI->getName(), RetTy);
1003:       if (Error E = OpCall.takeError())
1004:         return E;
1005:
1006:       CI->replaceAllUsesWith(*OpCall);
1007:       CI->eraseFromParent();
1008:       return Error::success();
1009:     });
1010:   }
1011:
1012:   bool lowerIntrinsics() {
1013:     bool Updated = false;
1014:     bool HasErrors = false;
1015:
1016:     for (Function &F : make_early_inc_range(M.functions())) {
1017:       if (!F.isDeclaration())
1018:         continue;
1019:       Intrinsic::ID ID = F.getIntrinsicID();
1020:       switch (ID) {
1021:       // NOTE: Skip dx_resource_casthandle here. They are
1022:       // resolved after this loop in cleanupHandleCasts.
1023:       case Intrinsic::dx_resource_casthandle:
1024:       // NOTE: llvm.dbg.value is supported as is in DXIL.
1025:       case Intrinsic::dbg_value:
1026:       case Intrinsic::not_intrinsic:
1027:         if (F.use_empty())
1028:           F.eraseFromParent();
1029:         continue;
1030:       default:
1031:         if (F.use_empty())
1032:           F.eraseFromParent();
1033:         else {
1034:           SmallString<128> Msg = formatv(
1035:               "Unsupported intrinsic {0} for DXIL lowering", F.getName());
1036:           M.getContext().emitError(Msg);
1037:           HasErrors |= true;
1038:         }
1039:         break;
1040:
1041: #define DXIL_OP_INTRINSIC(OpCode, Intrin, ...)                                 \
1042:   case Intrin:                                                                 \
1043:     HasErrors |= replaceFunctionWithOp(                                        \
1044:         F, OpCode, ArrayRef<IntrinArgSelect>{__VA_ARGS__});                    \
1045:     break;
1046: #include "DXILOperation.inc"
1047:       case Intrinsic::dx_resource_handlefrombinding:
1048:         HasErrors |= lowerHandleFromBinding(F);
1049:         break;
1050:       case Intrinsic::dx_resource_getpointer:
1051:         HasErrors |= lowerGetPointer(F);
1052:         break;
1053:       case Intrinsic::dx_resource_nonuniformindex:
1054:         assert(!CleanupNURI &&
1055:                "overloaded llvm.dx.resource.nonuniformindex intrinsics?");
1056:         CleanupNURI = &F;
1057:         break;
1058:       case Intrinsic::dx_resource_load_typedbuffer:
1059:         HasErrors |= lowerTypedBufferLoad(F, /*HasCheckBit=*/true);
1060:         break;
1061:       case Intrinsic::dx_resource_load_level:
1062:         HasErrors |= lowerTextureLoad(F);
1063:         break;
1064:       case Intrinsic::dx_resource_store_typedbuffer:
1065:         HasErrors |= lowerBufferStore(F, /*IsRaw=*/false);
1066:         break;
1067:       case Intrinsic::dx_resource_load_rawbuffer:
1068:         HasErrors |= lowerRawBufferLoad(F);
1069:         break;
1070:       case Intrinsic::dx_resource_store_rawbuffer:
1071:         HasErrors |= lowerBufferStore(F, /*IsRaw=*/true);
1072:         break;
1073:       case Intrinsic::dx_resource_load_cbufferrow_2:
1074:       case Intrinsic::dx_resource_load_cbufferrow_4:
1075:       case Intrinsic::dx_resource_load_cbufferrow_8:
1076:         HasErrors |= lowerCBufferLoad(F);
1077:         break;
1078:       case Intrinsic::dx_resource_updatecounter:
1079:         HasErrors |= lowerUpdateCounter(F);
1080:         break;
```
- EN: This range implements operational logic in helpers such as getZExtValue, inconvertibleErrorCode, tryCreateOp, replaceAllUsesWith, translating backend policy into executable code.
- CN: 这一段实现了 getZExtValue、inconvertibleErrorCode、tryCreateOp、replaceAllUsesWith 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1169
```cpp
1081:       case Intrinsic::dx_resource_getdimensions_x:
1082:         HasErrors |= lowerGetDimensionsX(F);
1083:         break;
1084:       case Intrinsic::ctpop:
1085:         HasErrors |= lowerCtpopToCountBits(F);
1086:         break;
1087:       case Intrinsic::lifetime_start:
1088:       case Intrinsic::lifetime_end:
1089:         if (F.use_empty())
1090:           F.eraseFromParent();
1091:         else {
1092:           if (MMDI.DXILVersion < VersionTuple(1, 6))
1093:             HasErrors |= lowerLifetimeIntrinsic(F);
1094:           else
1095:             continue;
1096:         }
1097:         break;
1098:       case Intrinsic::is_fpclass:
1099:         HasErrors |= lowerIsFPClass(F);
1100:         break;
1101:       }
1102:       Updated = true;
1103:     }
1104:     if (Updated && !HasErrors) {
1105:       cleanupHandleCasts();
1106:       cleanupNonUniformResourceIndexCalls();
1107:     }
1108:
1109:     return Updated;
1110:   }
1111: };
1112: } // namespace
1113:
1114: PreservedAnalyses DXILOpLowering::run(Module &M, ModuleAnalysisManager &MAM) {
1115:   DXILResourceMap &DRM = MAM.getResult<DXILResourceAnalysis>(M);
1116:   DXILResourceTypeMap &DRTM = MAM.getResult<DXILResourceTypeAnalysis>(M);
1117:   const ModuleMetadataInfo MMDI = MAM.getResult<DXILMetadataAnalysis>(M);
1118:
1119:   const bool MadeChanges = OpLowerer(M, DRM, DRTM, MMDI).lowerIntrinsics();
1120:   if (!MadeChanges)
1121:     return PreservedAnalyses::all();
1122:   PreservedAnalyses PA;
1123:   PA.preserve<DXILResourceAnalysis>();
1124:   PA.preserve<DXILMetadataAnalysis>();
1125:   PA.preserve<ShaderFlagsAnalysis>();
1126:   PA.preserve<RootSignatureAnalysis>();
1127:   return PA;
1128: }
1129:
1130: namespace {
1131: class DXILOpLoweringLegacy : public ModulePass {
1132: public:
1133:   bool runOnModule(Module &M) override {
1134:     DXILResourceMap &DRM =
1135:         getAnalysis<DXILResourceWrapperPass>().getResourceMap();
1136:     DXILResourceTypeMap &DRTM =
1137:         getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
1138:     const ModuleMetadataInfo MMDI =
1139:         getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
1140:
1141:     return OpLowerer(M, DRM, DRTM, MMDI).lowerIntrinsics();
1142:   }
1143:   StringRef getPassName() const override { return "DXIL Op Lowering"; }
1144:   DXILOpLoweringLegacy() : ModulePass(ID) {}
1145:
1146:   static char ID; // Pass identification.
1147:   void getAnalysisUsage(llvm::AnalysisUsage &AU) const override {
1148:     AU.addRequired<DXILResourceTypeWrapperPass>();
1149:     AU.addRequired<DXILResourceWrapperPass>();
1150:     AU.addRequired<DXILMetadataAnalysisWrapperPass>();
1151:     AU.addPreserved<DXILResourceWrapperPass>();
1152:     AU.addPreserved<DXILMetadataAnalysisWrapperPass>();
1153:     AU.addPreserved<ShaderFlagsAnalysisWrapper>();
1154:     AU.addPreserved<RootSignatureAnalysisWrapper>();
1155:   }
1156: };
1157: char DXILOpLoweringLegacy::ID = 0;
1158: } // end anonymous namespace
1159:
1160: INITIALIZE_PASS_BEGIN(DXILOpLoweringLegacy, DEBUG_TYPE, "DXIL Op Lowering",
1161:                       false, false)
1162: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
1163: INITIALIZE_PASS_DEPENDENCY(DXILResourceWrapperPass)
1164: INITIALIZE_PASS_END(DXILOpLoweringLegacy, DEBUG_TYPE, "DXIL Op Lowering", false,
1165:                     false)
1166:
1167: ModulePass *llvm::createDXILOpLoweringLegacyPass() {
1168:   return new DXILOpLoweringLegacy();
1169: }
```
- EN: This range defines or declares important types such as lowerGetDimensionsX, lowerCtpopToCountBits, eraseFromParent, lowerLifetimeIntrinsic, shaping the data model used by DXILOpLowering.cpp.
- CN: 这一段定义或声明了 lowerGetDimensionsX、lowerCtpopToCountBits、eraseFromParent、lowerLifetimeIntrinsic 等关键类型，构成 DXILOpLowering.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include OpLowerer, M, Error, Message, eraseFromParent, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 OpLowerer, M, Error, Message, eraseFromParent，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILOpLowering.h`
  - `DXILConstants.h`
  - `DXILOpBuilder.h`
  - `DXILRootSignature.h`
  - `DXILShaderFlags.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/IR/Constant.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/PassManager.h`
  - `llvm/IR/Use.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
- System/standard headers / 系统或标准头文件:
  - `DXILOperation.inc`
  - `DXILOperation.inc`
