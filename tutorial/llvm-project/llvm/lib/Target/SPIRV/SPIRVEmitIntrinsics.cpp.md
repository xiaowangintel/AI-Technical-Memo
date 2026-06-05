# SPIRVEmitIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVEmitIntrinsics.cpp`
- Repository: `llvm-project`
- Purpose (EN): The pass emits SPIRV intrinsics keeping essential high-level information for the translation of LLVM IR to SPIR-V.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-120
```cpp
  1: //===-- SPIRVEmitIntrinsics.cpp - emit SPIRV intrinsics ---------*- C++ -*-===//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8: //
  9: // The pass emits SPIRV intrinsics keeping essential high-level information for
 10: // the translation of LLVM IR to SPIR-V.
 11: //
 12: //===----------------------------------------------------------------------===//
 13:
 14: #include "SPIRVEmitIntrinsics.h"
 15: #include "SPIRV.h"
 16: #include "SPIRVBuiltins.h"
 17: #include "SPIRVSubtarget.h"
 18: #include "SPIRVTargetMachine.h"
 19: #include "SPIRVUtils.h"
 20: #include "llvm/ADT/DenseSet.h"
 21: #include "llvm/ADT/StringSet.h"
 22: #include "llvm/Analysis/LoopInfo.h"
 23: #include "llvm/IR/Dominators.h"
 24: #include "llvm/IR/IRBuilder.h"
 25: #include "llvm/IR/InstIterator.h"
 26: #include "llvm/IR/InstVisitor.h"
 27: #include "llvm/IR/IntrinsicsSPIRV.h"
 28: #include "llvm/IR/PatternMatch.h"
 29: #include "llvm/IR/TypedPointerType.h"
 30: #include "llvm/IR/Value.h"
 31: #include "llvm/Support/CommandLine.h"
 32: #include "llvm/Transforms/Utils/Local.h"
 33:
 34: #include <cassert>
 35: #include <optional>
 36: #include <queue>
 37: #include <unordered_set>
 38:
 39: // This pass performs the following transformation on LLVM IR level required
 40: // for the following translation to SPIR-V:
 41: // - replaces direct usages of aggregate constants with target-specific
 42: //   intrinsics;
 43: // - replaces aggregates-related instructions (extract/insert, ld/st, etc)
 44: //   with a target-specific intrinsics;
 45: // - emits intrinsics for the global variable initializers since IRTranslator
 46: //   doesn't handle them and it's not very convenient to translate them
 47: //   ourselves;
 48: // - emits intrinsics to keep track of the string names assigned to the values;
 49: // - emits intrinsics to keep track of constants (this is necessary to have an
 50: //   LLVM IR constant after the IRTranslation is completed) for their further
 51: //   deduplication;
 52: // - emits intrinsics to keep track of original LLVM types of the values
 53: //   to be able to emit proper SPIR-V types eventually.
 54: //
 55: // TODO: consider removing spv.track.constant in favor of spv.assign.type.
 56:
 57: using namespace llvm;
 58: using namespace llvm::PatternMatch;
 59:
 60: static cl::opt<bool>
 61:     SpirvEmitOpNames("spirv-emit-op-names",
 62:                      cl::desc("Emit OpName for all instructions"),
 63:                      cl::init(false));
 64:
 65: namespace llvm::SPIRV {
 66: #define GET_BuiltinGroup_DECL
 67: #include "SPIRVGenTables.inc"
 68: } // namespace llvm::SPIRV
 69:
 70: namespace {
 71: // This class keeps track of which functions reference which global variables.
 72: class GlobalVariableUsers {
 73:   template <typename T1, typename T2>
 74:   using OneToManyMapTy = DenseMap<T1, SmallPtrSet<T2, 4>>;
 75:
 76:   OneToManyMapTy<const GlobalVariable *, const Function *> GlobalIsUsedByFun;
 77:
 78:   void collectGlobalUsers(
 79:       const GlobalVariable *GV,
 80:       OneToManyMapTy<const GlobalVariable *, const GlobalVariable *>
 81:           &GlobalIsUsedByGlobal) {
 82:     SmallVector<const Value *> Stack = {GV->user_begin(), GV->user_end()};
 83:     while (!Stack.empty()) {
 84:       const Value *V = Stack.pop_back_val();
 85:
 86:       if (const Instruction *I = dyn_cast<Instruction>(V)) {
 87:         GlobalIsUsedByFun[GV].insert(I->getFunction());
 88:         continue;
 89:       }
 90:
 91:       if (const GlobalVariable *UserGV = dyn_cast<GlobalVariable>(V)) {
 92:         GlobalIsUsedByGlobal[GV].insert(UserGV);
 93:         continue;
 94:       }
 95:
 96:       if (const Constant *C = dyn_cast<Constant>(V))
 97:         Stack.append(C->user_begin(), C->user_end());
 98:     }
 99:   }
100:
101:   bool propagateGlobalToGlobalUsers(
102:       OneToManyMapTy<const GlobalVariable *, const GlobalVariable *>
103:           &GlobalIsUsedByGlobal) {
104:     SmallVector<const GlobalVariable *> OldUsersGlobals;
105:     bool Changed = false;
106:     for (auto &[GV, UserGlobals] : GlobalIsUsedByGlobal) {
107:       OldUsersGlobals.assign(UserGlobals.begin(), UserGlobals.end());
108:       for (const GlobalVariable *UserGV : OldUsersGlobals) {
109:         auto It = GlobalIsUsedByGlobal.find(UserGV);
110:         if (It == GlobalIsUsedByGlobal.end())
111:           continue;
112:         Changed |= set_union(UserGlobals, It->second);
113:       }
114:     }
115:     return Changed;
116:   }
117:
118:   void propagateGlobalToFunctionReferences(
119:       OneToManyMapTy<const GlobalVariable *, const GlobalVariable *>
120:           &GlobalIsUsedByGlobal) {
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 121-240
```cpp
121:     for (auto &[GV, UserGlobals] : GlobalIsUsedByGlobal) {
122:       auto &UserFunctions = GlobalIsUsedByFun[GV];
123:       for (const GlobalVariable *UserGV : UserGlobals) {
124:         auto It = GlobalIsUsedByFun.find(UserGV);
125:         if (It == GlobalIsUsedByFun.end())
126:           continue;
127:         set_union(UserFunctions, It->second);
128:       }
129:     }
130:   }
131:
132: public:
133:   void init(Module &M) {
134:     // Collect which global variables are referenced by which global variables
135:     // and which functions reference each global variables.
136:     OneToManyMapTy<const GlobalVariable *, const GlobalVariable *>
137:         GlobalIsUsedByGlobal;
138:     GlobalIsUsedByFun.clear();
139:     for (GlobalVariable &GV : M.globals())
140:       collectGlobalUsers(&GV, GlobalIsUsedByGlobal);
141:
142:     // Compute indirect references by iterating until a fixed point is reached.
143:     while (propagateGlobalToGlobalUsers(GlobalIsUsedByGlobal))
144:       (void)0;
145:
146:     propagateGlobalToFunctionReferences(GlobalIsUsedByGlobal);
147:   }
148:
149:   using FunctionSetType = typename decltype(GlobalIsUsedByFun)::mapped_type;
150:   const FunctionSetType &
151:   getTransitiveUserFunctions(const GlobalVariable &GV) const {
152:     auto It = GlobalIsUsedByFun.find(&GV);
153:     if (It != GlobalIsUsedByFun.end())
154:       return It->second;
155:
156:     static const FunctionSetType Empty{};
157:     return Empty;
158:   }
159: };
160:
161: static bool isaGEP(const Value *V) {
162:   return isa<StructuredGEPInst>(V) || isa<GetElementPtrInst>(V);
163: }
164:
165: // If Ty is a byte-addressing type, return the multiplier for the offset.
166: // Otherwise return std::nullopt.
167: static std::optional<uint64_t> getByteAddressingMultiplier(Type *Ty) {
168:   if (Ty == IntegerType::getInt8Ty(Ty->getContext())) {
169:     return 1;
170:   }
171:   if (auto *AT = dyn_cast<ArrayType>(Ty)) {
172:     if (AT->getElementType() == IntegerType::getInt8Ty(Ty->getContext())) {
173:       return AT->getNumElements();
174:     }
175:   }
176:   return std::nullopt;
177: }
178:
179: class SPIRVEmitIntrinsics
180:     : public ModulePass,
181:       public InstVisitor<SPIRVEmitIntrinsics, Instruction *> {
182:   const SPIRVTargetMachine &TM;
183:   SPIRVGlobalRegistry *GR = nullptr;
184:   Function *CurrF = nullptr;
185:   bool TrackConstants = true;
186:   bool HaveFunPtrs = false;
187:   DenseMap<Instruction *, Constant *> AggrConsts;
188:   DenseMap<Instruction *, Type *> AggrConstTypes;
189:   DenseSet<Instruction *> AggrStores;
190:   SmallPtrSet<Instruction *, 8> DeletedInstrs;
191:   GlobalVariableUsers GVUsers;
192:   std::unordered_set<Value *> Named;
193:
194:   // map of function declarations to <pointer arg index => element type>
195:   DenseMap<Function *, SmallVector<std::pair<unsigned, Type *>>> FDeclPtrTys;
196:
197:   // a register of Instructions that don't have a complete type definition
198:   bool CanTodoType = true;
199:   unsigned TodoTypeSz = 0;
200:   DenseMap<Value *, bool> TodoType;
201:   void insertTodoType(Value *Op) {
202:     // TODO: add isa<CallInst>(Op) to no-insert
203:     if (CanTodoType && !isaGEP(Op)) {
204:       auto It = TodoType.try_emplace(Op, true);
205:       if (It.second)
206:         ++TodoTypeSz;
207:     }
208:   }
209:   void eraseTodoType(Value *Op) {
210:     auto It = TodoType.find(Op);
211:     if (It != TodoType.end() && It->second) {
212:       It->second = false;
213:       --TodoTypeSz;
214:     }
215:   }
216:   bool isTodoType(Value *Op) {
217:     if (isaGEP(Op))
218:       return false;
219:     auto It = TodoType.find(Op);
220:     return It != TodoType.end() && It->second;
221:   }
222:   // a register of Instructions that were visited by deduceOperandElementType()
223:   // to validate operand types with an instruction
224:   std::unordered_set<Instruction *> TypeValidated;
225:
226:   // well known result types of builtins
227:   enum WellKnownTypes { Event };
228:
229:   // deduce element type of untyped pointers
230:   Type *deduceElementType(Value *I, bool UnknownElemTypeI8);
231:   Type *deduceElementTypeHelper(Value *I, bool UnknownElemTypeI8);
232:   Type *deduceElementTypeHelper(Value *I, std::unordered_set<Value *> &Visited,
233:                                 bool UnknownElemTypeI8,
234:                                 bool IgnoreKnownType = false);
235:   Type *deduceElementTypeByValueDeep(Type *ValueTy, Value *Operand,
236:                                      bool UnknownElemTypeI8);
237:   Type *deduceElementTypeByValueDeep(Type *ValueTy, Value *Operand,
238:                                      std::unordered_set<Value *> &Visited,
239:                                      bool UnknownElemTypeI8);
240:   Type *deduceElementTypeByUsersDeep(Value *Op,
```
- EN: This range defines or declares important types such as find, set_union, init, clear, shaping the data model used by SPIRVEmitIntrinsics.cpp.
- CN: 这一段定义或声明了 find、set_union、init、clear 等关键类型，构成 SPIRVEmitIntrinsics.cpp 使用的数据模型。

### Lines 241-360
```cpp
241:                                      std::unordered_set<Value *> &Visited,
242:                                      bool UnknownElemTypeI8);
243:   void maybeAssignPtrType(Type *&Ty, Value *I, Type *RefTy,
244:                           bool UnknownElemTypeI8);
245:
246:   // deduce nested types of composites
247:   Type *deduceNestedTypeHelper(User *U, bool UnknownElemTypeI8);
248:   Type *deduceNestedTypeHelper(User *U, Type *Ty,
249:                                std::unordered_set<Value *> &Visited,
250:                                bool UnknownElemTypeI8);
251:
252:   // deduce Types of operands of the Instruction if possible
253:   void deduceOperandElementType(Instruction *I,
254:                                 SmallPtrSet<Instruction *, 4> *IncompleteRets,
255:                                 const SmallPtrSet<Value *, 4> *AskOps = nullptr,
256:                                 bool IsPostprocessing = false);
257:
258:   void preprocessCompositeConstants(IRBuilder<> &B);
259:   void preprocessUndefs(IRBuilder<> &B);
260:   void simplifyNullAddrSpaceCasts();
261:
262:   Type *reconstructType(Value *Op, bool UnknownElemTypeI8,
263:                         bool IsPostprocessing);
264:
265:   void replaceMemInstrUses(Instruction *Old, Instruction *New, IRBuilder<> &B);
266:   void processInstrAfterVisit(Instruction *I, IRBuilder<> &B);
267:   bool insertAssignPtrTypeIntrs(Instruction *I, IRBuilder<> &B,
268:                                 bool UnknownElemTypeI8);
269:   void insertAssignTypeIntrs(Instruction *I, IRBuilder<> &B);
270:   void insertAssignPtrTypeTargetExt(TargetExtType *AssignedType, Value *V,
271:                                     IRBuilder<> &B);
272:   void replacePointerOperandWithPtrCast(Instruction *I, Value *Pointer,
273:                                         Type *ExpectedElementType,
274:                                         unsigned OperandToReplace,
275:                                         IRBuilder<> &B);
276:   void insertPtrCastOrAssignTypeInstr(Instruction *I, IRBuilder<> &B);
277:   bool shouldTryToAddMemAliasingDecoration(Instruction *Inst);
278:   void insertSpirvDecorations(Instruction *I, IRBuilder<> &B);
279:   void insertConstantsForFPFastMathDefault(Module &M);
280:   Value *buildSpvUndefComposite(Type *AggrTy, IRBuilder<> &B);
281:   void processGlobalValue(GlobalVariable &GV, IRBuilder<> &B);
282:   void processParamTypes(Function *F, IRBuilder<> &B);
283:   void processParamTypesByFunHeader(Function *F, IRBuilder<> &B);
284:   Type *deduceFunParamElementType(Function *F, unsigned OpIdx);
285:   Type *deduceFunParamElementType(Function *F, unsigned OpIdx,
286:                                   std::unordered_set<Function *> &FVisited);
287:
288:   bool deduceOperandElementTypeCalledFunction(
289:       CallInst *CI, SmallVector<std::pair<Value *, unsigned>> &Ops,
290:       Type *&KnownElemTy, bool &Incomplete);
291:   void deduceOperandElementTypeFunctionPointer(
292:       CallInst *CI, SmallVector<std::pair<Value *, unsigned>> &Ops,
293:       Type *&KnownElemTy, bool IsPostprocessing);
294:   bool deduceOperandElementTypeFunctionRet(
295:       Instruction *I, SmallPtrSet<Instruction *, 4> *IncompleteRets,
296:       const SmallPtrSet<Value *, 4> *AskOps, bool IsPostprocessing,
297:       Type *&KnownElemTy, Value *Op, Function *F);
298:
299:   CallInst *buildSpvPtrcast(Function *F, Value *Op, Type *ElemTy);
300:   void replaceUsesOfWithSpvPtrcast(Value *Op, Type *ElemTy, Instruction *I,
301:                                    DenseMap<Function *, CallInst *> Ptrcasts);
302:   void propagateElemType(Value *Op, Type *ElemTy,
303:                          DenseSet<std::pair<Value *, Value *>> &VisitedSubst);
304:   void
305:   propagateElemTypeRec(Value *Op, Type *PtrElemTy, Type *CastElemTy,
306:                        DenseSet<std::pair<Value *, Value *>> &VisitedSubst);
307:   void propagateElemTypeRec(Value *Op, Type *PtrElemTy, Type *CastElemTy,
308:                             DenseSet<std::pair<Value *, Value *>> &VisitedSubst,
309:                             std::unordered_set<Value *> &Visited,
310:                             DenseMap<Function *, CallInst *> Ptrcasts);
311:
312:   void replaceAllUsesWith(Value *Src, Value *Dest, bool DeleteOld = true);
313:   void replaceAllUsesWithAndErase(IRBuilder<> &B, Instruction *Src,
314:                                   Instruction *Dest, bool DeleteOld = true);
315:
316:   void applyDemangledPtrArgTypes(IRBuilder<> &B);
317:
318:   GetElementPtrInst *simplifyZeroLengthArrayGepInst(GetElementPtrInst *GEP);
319:
320:   bool runOnFunction(Function &F);
321:   bool postprocessTypes(Module &M);
322:   bool processFunctionPointers(Module &M);
323:   void parseFunDeclarations(Module &M);
324:   void useRoundingMode(ConstrainedFPIntrinsic *FPI, IRBuilder<> &B);
325:   bool processMaskedMemIntrinsic(IntrinsicInst &I);
326:   bool convertMaskedMemIntrinsics(Module &M);
327:   void preprocessBoolVectorBitcasts(Function &F);
328:
329:   void emitUnstructuredLoopControls(Function &F, IRBuilder<> &B);
330:
331:   // Tries to walk the type accessed by the given GEP instruction.
332:   // For each nested type access, one of the 2 callbacks is called:
333:   //  - OnLiteralIndexing when the index is a known constant value.
334:   //    Parameters:
335:   //      PointedType: the pointed type resulting of this indexing.
336:   //        If the parent type is an array, this is the index in the array.
337:   //        If the parent type is a struct, this is the field index.
338:   //      Index: index of the element in the parent type.
339:   //  - OnDynamnicIndexing when the index is a non-constant value.
340:   //    This callback is only called when indexing into an array.
341:   //    Parameters:
342:   //      ElementType: the type of the elements stored in the parent array.
343:   //      Offset: the Value* containing the byte offset into the array.
344:   //      Multiplier: a scaling factor for the offset.
345:   // Return true if an error occurred during the walk, false otherwise.
346:   bool walkLogicalAccessChain(
347:       GetElementPtrInst &GEP,
348:       const std::function<void(Type *PointedType, uint64_t Index)>
349:           &OnLiteralIndexing,
350:       const std::function<void(Type *ElementType, Value *Offset,
351:                                uint64_t Multiplier)> &OnDynamicIndexing);
352:
353:   bool walkLogicalAccessChainDynamic(
354:       Type *CurType, Value *Operand, uint64_t Multiplier,
355:       const std::function<void(Type *, uint64_t)> &OnLiteralIndexing,
356:       const std::function<void(Type *, Value *, uint64_t)> &OnDynamicIndexing);
357:
358:   bool walkLogicalAccessChainConstant(
359:       Type *CurType, uint64_t Offset,
360:       const std::function<void(Type *, uint64_t)> &OnLiteralIndexing);
```
- EN: This range implements operational logic in helpers such as deduceNestedTypeHelper, preprocessCompositeConstants, preprocessUndefs, simplifyNullAddrSpaceCasts, translating backend policy into executable code.
- CN: 这一段实现了 deduceNestedTypeHelper、preprocessCompositeConstants、preprocessUndefs、simplifyNullAddrSpaceCasts 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-480
```cpp
361:
362:   // Returns the type accessed using the given GEP instruction by relying
363:   // on the GEP type.
364:   // FIXME: GEP types are not supposed to be used to retrieve the pointed
365:   // type. This must be fixed.
366:   Type *getGEPType(GetElementPtrInst *GEP);
367:
368:   // Returns the type accessed using the given GEP instruction by walking
369:   // the source type using the GEP indices.
370:   // FIXME: without help from the frontend, this method cannot reliably retrieve
371:   // the stored type, nor can robustly determine the depth of the type
372:   // we are accessing.
373:   Type *getGEPTypeLogical(GetElementPtrInst *GEP);
374:
375:   Instruction *buildLogicalAccessChainFromGEP(GetElementPtrInst &GEP);
376:
377: public:
378:   static char ID;
379:   SPIRVEmitIntrinsics(const SPIRVTargetMachine &TM) : ModulePass(ID), TM(TM) {}
380:   Instruction *visitInstruction(Instruction &I) { return &I; }
381:   Instruction *visitSwitchInst(SwitchInst &I);
382:   Instruction *visitGetElementPtrInst(GetElementPtrInst &I);
383:   Instruction *visitIntrinsicInst(IntrinsicInst &I);
384:   Instruction *visitBitCastInst(BitCastInst &I);
385:   Instruction *visitInsertElementInst(InsertElementInst &I);
386:   Instruction *visitExtractElementInst(ExtractElementInst &I);
387:   Instruction *visitInsertValueInst(InsertValueInst &I);
388:   Instruction *visitExtractValueInst(ExtractValueInst &I);
389:   Instruction *visitLoadInst(LoadInst &I);
390:   Instruction *visitStoreInst(StoreInst &I);
391:   Instruction *visitAllocaInst(AllocaInst &I);
392:   Instruction *visitAtomicCmpXchgInst(AtomicCmpXchgInst &I);
393:   Instruction *visitUnreachableInst(UnreachableInst &I);
394:   Instruction *visitCallInst(CallInst &I);
395:
396:   StringRef getPassName() const override { return "SPIRV emit intrinsics"; }
397:
398:   bool runOnModule(Module &M) override;
399:
400:   void getAnalysisUsage(AnalysisUsage &AU) const override {
401:     ModulePass::getAnalysisUsage(AU);
402:   }
403: };
404:
405: bool isConvergenceIntrinsic(const Instruction *I) {
406:   return match(I, m_AnyIntrinsic<Intrinsic::experimental_convergence_entry,
407:                                  Intrinsic::experimental_convergence_loop,
408:                                  Intrinsic::experimental_convergence_anchor>());
409: }
410:
411: bool expectIgnoredInIRTranslation(const Instruction *I) {
412:   return match(I, m_AnyIntrinsic<Intrinsic::invariant_start,
413:                                  Intrinsic::spv_resource_handlefrombinding,
414:                                  Intrinsic::spv_resource_getbasepointer,
415:                                  Intrinsic::spv_resource_getpointer>());
416: }
417:
418: // Returns the source pointer from `I` ignoring intermediate ptrcast.
419: Value *getPointerRoot(Value *I) {
420:   Value *V;
421:   if (match(I, m_Intrinsic<Intrinsic::spv_ptrcast>(m_Value(V))))
422:     return getPointerRoot(V);
423:   return I;
424: }
425:
426: } // namespace
427:
428: char SPIRVEmitIntrinsics::ID = 0;
429:
430: INITIALIZE_PASS(SPIRVEmitIntrinsics, "spirv-emit-intrinsics",
431:                 "SPIRV emit intrinsics", false, false)
432:
433: static inline bool isAssignTypeInstr(const Instruction *I) {
434:   return match(I, m_Intrinsic<Intrinsic::spv_assign_type>());
435: }
436:
437: static bool isMemInstrToReplace(Instruction *I) {
438:   return isa<StoreInst>(I) || isa<LoadInst>(I) || isa<InsertValueInst>(I) ||
439:          isa<ExtractValueInst>(I) || isa<AtomicCmpXchgInst>(I);
440: }
441:
442: static bool isAggrConstForceInt32(const Value *V) {
443:   bool IsAggrZero =
444:       isa<ConstantAggregateZero>(V) && !V->getType()->isVectorTy();
445:   bool IsUndefAggregate = isa<UndefValue>(V) && V->getType()->isAggregateType();
446:   return isa<ConstantArray>(V) || isa<ConstantStruct>(V) ||
447:          isa<ConstantDataArray>(V) || IsAggrZero || IsUndefAggregate;
448: }
449:
450: static void setInsertPointSkippingPhis(IRBuilder<> &B, Instruction *I) {
451:   if (isa<PHINode>(I))
452:     B.SetInsertPoint(I->getParent()->getFirstNonPHIOrDbgOrAlloca());
453:   else
454:     B.SetInsertPoint(I);
455: }
456:
457: static void setInsertPointAfterDef(IRBuilder<> &B, Instruction *I) {
458:   B.SetCurrentDebugLocation(I->getDebugLoc());
459:   if (I->getType()->isVoidTy())
460:     B.SetInsertPoint(I->getNextNode());
461:   else
462:     B.SetInsertPoint(*I->getInsertionPointAfterDef());
463: }
464:
465: static bool requireAssignType(Instruction *I) {
466:   return !match(
467:       I,
468:       m_AnyIntrinsic<Intrinsic::invariant_start, Intrinsic::invariant_end>());
469: }
470:
471: static inline void reportFatalOnTokenType(const Instruction *I) {
472:   if (I->getType()->isTokenTy())
473:     report_fatal_error("A token is encountered but SPIR-V without extensions "
474:                        "does not support token type",
475:                        false);
476: }
477:
478: static void emitAssignName(Instruction *I, IRBuilder<> &B) {
479:   if (!I->hasName() || I->getType()->isAggregateType() ||
480:       expectIgnoredInIRTranslation(I))
```
- EN: This range implements operational logic in helpers such as getGEPType, getGEPTypeLogical, buildLogicalAccessChainFromGEP, SPIRVEmitIntrinsics, translating backend policy into executable code.
- CN: 这一段实现了 getGEPType、getGEPTypeLogical、buildLogicalAccessChainFromGEP、SPIRVEmitIntrinsics 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-600
```cpp
481:     return;
482:
483:   // We want to be conservative when adding the names because they can interfere
484:   // with later optimizations.
485:   bool KeepName = SpirvEmitOpNames;
486:   if (!KeepName) {
487:     if (isa<AllocaInst>(I)) {
488:       KeepName = true;
489:     } else if (auto *CI = dyn_cast<CallBase>(I)) {
490:       Function *F = CI->getCalledFunction();
491:       if (F && F->getName().starts_with("llvm.spv.alloca"))
492:         KeepName = true;
493:     }
494:   }
495:
496:   if (!KeepName)
497:     return;
498:
499:   reportFatalOnTokenType(I);
500:   setInsertPointAfterDef(B, I);
501:   LLVMContext &Ctx = I->getContext();
502:   std::vector<Value *> Args = {
503:       I, MetadataAsValue::get(
504:              Ctx, MDNode::get(Ctx, MDString::get(Ctx, I->getName())))};
505:   B.CreateIntrinsic(Intrinsic::spv_assign_name, {I->getType()}, Args);
506: }
507:
508: void SPIRVEmitIntrinsics::replaceAllUsesWith(Value *Src, Value *Dest,
509:                                              bool DeleteOld) {
510:   GR->replaceAllUsesWith(Src, Dest, DeleteOld);
511:   // Update uncomplete type records if any
512:   if (isTodoType(Src)) {
513:     if (DeleteOld)
514:       eraseTodoType(Src);
515:     insertTodoType(Dest);
516:   }
517: }
518:
519: void SPIRVEmitIntrinsics::replaceAllUsesWithAndErase(IRBuilder<> &B,
520:                                                      Instruction *Src,
521:                                                      Instruction *Dest,
522:                                                      bool DeleteOld) {
523:   replaceAllUsesWith(Src, Dest, DeleteOld);
524:   std::string Name = Src->hasName() ? Src->getName().str() : "";
525:   Src->eraseFromParent();
526:   if (!Name.empty()) {
527:     Dest->setName(Name);
528:     if (Named.insert(Dest).second)
529:       emitAssignName(Dest, B);
530:   }
531: }
532:
533: static bool IsKernelArgInt8(Function *F, StoreInst *SI) {
534:   return SI && F->getCallingConv() == CallingConv::SPIR_KERNEL &&
535:          isPointerTy(SI->getValueOperand()->getType()) &&
536:          isa<Argument>(SI->getValueOperand());
537: }
538:
539: // Maybe restore original function return type.
540: static inline Type *restoreMutatedType(SPIRVGlobalRegistry *GR, Instruction *I,
541:                                        Type *Ty) {
542:   CallInst *CI = dyn_cast<CallInst>(I);
543:   if (!CI || CI->isIndirectCall() || CI->isInlineAsm() ||
544:       !CI->getCalledFunction() || CI->getCalledFunction()->isIntrinsic())
545:     return Ty;
546:   if (Type *OriginalTy = GR->findMutated(CI->getCalledFunction()))
547:     return OriginalTy;
548:   return Ty;
549: }
550:
551: // Reconstruct type with nested element types according to deduced type info.
552: // Return nullptr if no detailed type info is available.
553: Type *SPIRVEmitIntrinsics::reconstructType(Value *Op, bool UnknownElemTypeI8,
554:                                            bool IsPostprocessing) {
555:   Type *Ty = Op->getType();
556:   if (auto *OpI = dyn_cast<Instruction>(Op)) {
557:     Ty = restoreMutatedType(GR, OpI, Ty);
558:     // Aggregate undef/constant values are lowered to spv_undef/
559:     // spv_const_composite intrinsics whose return type is i32; recover the
560:     // original aggregate type so downstream pointee deduction sees it.
561:     if (match(OpI, m_CombineOr(m_Intrinsic<Intrinsic::spv_undef>(),
562:                                m_Intrinsic<Intrinsic::spv_const_composite>())))
563:       if (auto It = AggrConstTypes.find(OpI); It != AggrConstTypes.end())
564:         Ty = It->second;
565:   }
566:   if (!isUntypedPointerTy(Ty))
567:     return Ty;
568:   // try to find the pointee type
569:   if (Type *NestedTy = GR->findDeducedElementType(Op))
570:     return getTypedPointerWrapper(NestedTy, getPointerAddressSpace(Ty));
571:   // not a pointer according to the type info (e.g., Event object)
572:   CallInst *CI = GR->findAssignPtrTypeInstr(Op);
573:   if (CI) {
574:     MetadataAsValue *MD = cast<MetadataAsValue>(CI->getArgOperand(1));
575:     return cast<ConstantAsMetadata>(MD->getMetadata())->getType();
576:   }
577:   if (UnknownElemTypeI8) {
578:     if (!IsPostprocessing)
579:       insertTodoType(Op);
580:     return getTypedPointerWrapper(IntegerType::getInt8Ty(Op->getContext()),
581:                                   getPointerAddressSpace(Ty));
582:   }
583:   return nullptr;
584: }
585:
586: CallInst *SPIRVEmitIntrinsics::buildSpvPtrcast(Function *F, Value *Op,
587:                                                Type *ElemTy) {
588:   IRBuilder<> B(Op->getContext());
589:   if (auto *OpI = dyn_cast<Instruction>(Op)) {
590:     // spv_ptrcast's argument Op denotes an instruction that generates
591:     // a value, and we may use getInsertionPointAfterDef()
592:     setInsertPointAfterDef(B, OpI);
593:   } else if (auto *OpA = dyn_cast<Argument>(Op)) {
594:     B.SetInsertPointPastAllocas(OpA->getParent());
595:     B.SetCurrentDebugLocation(DebugLoc());
596:   } else {
597:     B.SetInsertPoint(F->getEntryBlock().getFirstNonPHIOrDbgOrAlloca());
598:   }
599:   Type *OpTy = Op->getType();
600:   SmallVector<Type *, 2> Types = {OpTy, OpTy};
```
- EN: This range implements operational logic in helpers such as getCalledFunction, reportFatalOnTokenType, setInsertPointAfterDef, getContext, translating backend policy into executable code.
- CN: 这一段实现了 getCalledFunction、reportFatalOnTokenType、setInsertPointAfterDef、getContext 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-720
```cpp
601:   SmallVector<Value *, 2> Args = {Op, buildMD(getNormalizedPoisonValue(ElemTy)),
602:                                   B.getInt32(getPointerAddressSpace(OpTy))};
603:   CallInst *PtrCasted =
604:       B.CreateIntrinsic(Intrinsic::spv_ptrcast, {Types}, Args);
605:   GR->buildAssignPtr(B, ElemTy, PtrCasted);
606:   return PtrCasted;
607: }
608:
609: void SPIRVEmitIntrinsics::replaceUsesOfWithSpvPtrcast(
610:     Value *Op, Type *ElemTy, Instruction *I,
611:     DenseMap<Function *, CallInst *> Ptrcasts) {
612:   Function *F = I->getParent()->getParent();
613:   CallInst *PtrCastedI = nullptr;
614:   auto It = Ptrcasts.find(F);
615:   if (It == Ptrcasts.end()) {
616:     PtrCastedI = buildSpvPtrcast(F, Op, ElemTy);
617:     Ptrcasts[F] = PtrCastedI;
618:   } else {
619:     PtrCastedI = It->second;
620:   }
621:   I->replaceUsesOfWith(Op, PtrCastedI);
622: }
623:
624: void SPIRVEmitIntrinsics::propagateElemType(
625:     Value *Op, Type *ElemTy,
626:     DenseSet<std::pair<Value *, Value *>> &VisitedSubst) {
627:   DenseMap<Function *, CallInst *> Ptrcasts;
628:   SmallVector<User *> Users(Op->users());
629:   for (auto *U : Users) {
630:     if (!isa<Instruction>(U) || isSpvIntrinsic(U))
631:       continue;
632:     if (!VisitedSubst.insert(std::make_pair(U, Op)).second)
633:       continue;
634:     Instruction *UI = dyn_cast<Instruction>(U);
635:     // If the instruction was validated already, we need to keep it valid by
636:     // keeping current Op type.
637:     if (isaGEP(UI) || TypeValidated.find(UI) != TypeValidated.end())
638:       replaceUsesOfWithSpvPtrcast(Op, ElemTy, UI, Ptrcasts);
639:   }
640: }
641:
642: void SPIRVEmitIntrinsics::propagateElemTypeRec(
643:     Value *Op, Type *PtrElemTy, Type *CastElemTy,
644:     DenseSet<std::pair<Value *, Value *>> &VisitedSubst) {
645:   std::unordered_set<Value *> Visited;
646:   DenseMap<Function *, CallInst *> Ptrcasts;
647:   propagateElemTypeRec(Op, PtrElemTy, CastElemTy, VisitedSubst, Visited,
648:                        std::move(Ptrcasts));
649: }
650:
651: void SPIRVEmitIntrinsics::propagateElemTypeRec(
652:     Value *Op, Type *PtrElemTy, Type *CastElemTy,
653:     DenseSet<std::pair<Value *, Value *>> &VisitedSubst,
654:     std::unordered_set<Value *> &Visited,
655:     DenseMap<Function *, CallInst *> Ptrcasts) {
656:   if (!Visited.insert(Op).second)
657:     return;
658:   SmallVector<User *> Users(Op->users());
659:   for (auto *U : Users) {
660:     if (!isa<Instruction>(U) || isSpvIntrinsic(U))
661:       continue;
662:     if (!VisitedSubst.insert(std::make_pair(U, Op)).second)
663:       continue;
664:     Instruction *UI = dyn_cast<Instruction>(U);
665:     // If the instruction was validated already, we need to keep it valid by
666:     // keeping current Op type.
667:     if (isaGEP(UI) || TypeValidated.find(UI) != TypeValidated.end())
668:       replaceUsesOfWithSpvPtrcast(Op, CastElemTy, UI, Ptrcasts);
669:   }
670: }
671:
672: // Set element pointer type to the given value of ValueTy and tries to
673: // specify this type further (recursively) by Operand value, if needed.
674:
675: Type *
676: SPIRVEmitIntrinsics::deduceElementTypeByValueDeep(Type *ValueTy, Value *Operand,
677:                                                   bool UnknownElemTypeI8) {
678:   std::unordered_set<Value *> Visited;
679:   return deduceElementTypeByValueDeep(ValueTy, Operand, Visited,
680:                                       UnknownElemTypeI8);
681: }
682:
683: Type *SPIRVEmitIntrinsics::deduceElementTypeByValueDeep(
684:     Type *ValueTy, Value *Operand, std::unordered_set<Value *> &Visited,
685:     bool UnknownElemTypeI8) {
686:   Type *Ty = ValueTy;
687:   if (Operand) {
688:     if (auto *PtrTy = dyn_cast<PointerType>(Ty)) {
689:       if (Type *NestedTy =
690:               deduceElementTypeHelper(Operand, Visited, UnknownElemTypeI8))
691:         Ty = getTypedPointerWrapper(NestedTy, PtrTy->getAddressSpace());
692:     } else {
693:       Ty = deduceNestedTypeHelper(dyn_cast<User>(Operand), Ty, Visited,
694:                                   UnknownElemTypeI8);
695:     }
696:   }
697:   return Ty;
698: }
699:
700: // Traverse User instructions to deduce an element pointer type of the operand.
701: Type *SPIRVEmitIntrinsics::deduceElementTypeByUsersDeep(
702:     Value *Op, std::unordered_set<Value *> &Visited, bool UnknownElemTypeI8) {
703:   if (!Op || !isPointerTy(Op->getType()) || isa<ConstantPointerNull>(Op) ||
704:       isa<UndefValue>(Op))
705:     return nullptr;
706:
707:   if (auto ElemTy = getPointeeType(Op->getType()))
708:     return ElemTy;
709:
710:   // maybe we already know operand's element type
711:   if (Type *KnownTy = GR->findDeducedElementType(Op))
712:     return KnownTy;
713:
714:   for (User *OpU : Op->users()) {
715:     if (Instruction *Inst = dyn_cast<Instruction>(OpU)) {
716:       if (Type *Ty = deduceElementTypeHelper(Inst, Visited, UnknownElemTypeI8))
717:         return Ty;
718:     }
719:   }
720:   return nullptr;
```
- EN: This range implements operational logic in helpers such as CreateIntrinsic, buildAssignPtr, getParent, find, translating backend policy into executable code.
- CN: 这一段实现了 CreateIntrinsic、buildAssignPtr、getParent、find 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-840
```cpp
721: }
722:
723: // Implements what we know in advance about intrinsics and builtin calls
724: // TODO: consider feasibility of this particular case to be generalized by
725: // encoding knowledge about intrinsics and builtin calls by corresponding
726: // specification rules
727: static Type *getPointeeTypeByCallInst(StringRef DemangledName,
728:                                       Function *CalledF, unsigned OpIdx) {
729:   if ((DemangledName.starts_with("__spirv_ocl_printf(") ||
730:        DemangledName.starts_with("printf(")) &&
731:       OpIdx == 0)
732:     return IntegerType::getInt8Ty(CalledF->getContext());
733:   return nullptr;
734: }
735:
736: // Deduce and return a successfully deduced Type of the Instruction,
737: // or nullptr otherwise.
738: Type *SPIRVEmitIntrinsics::deduceElementTypeHelper(Value *I,
739:                                                    bool UnknownElemTypeI8) {
740:   std::unordered_set<Value *> Visited;
741:   return deduceElementTypeHelper(I, Visited, UnknownElemTypeI8);
742: }
743:
744: void SPIRVEmitIntrinsics::maybeAssignPtrType(Type *&Ty, Value *Op, Type *RefTy,
745:                                              bool UnknownElemTypeI8) {
746:   if (isUntypedPointerTy(RefTy)) {
747:     if (!UnknownElemTypeI8)
748:       return;
749:     insertTodoType(Op);
750:     if (isa<IntToPtrInst>(Op))
751:       return;
752:   }
753:   Ty = RefTy;
754: }
755:
756: bool SPIRVEmitIntrinsics::walkLogicalAccessChainDynamic(
757:     Type *CurType, Value *Operand, uint64_t Multiplier,
758:     const std::function<void(Type *, uint64_t)> &OnLiteralIndexing,
759:     const std::function<void(Type *, Value *, uint64_t)> &OnDynamicIndexing) {
760:   // Dynamic indexing into a struct is not possible.
761:   // We know that we must be accessing the first element
762:   // of the struct if the current type is a struct.
763:   // Try to find the first array type that is at offset 0 in the struct.
764:   while (auto *ST = dyn_cast<StructType>(CurType)) {
765:     if (ST->getNumElements() > 0) {
766:       CurType = ST->getElementType(0);
767:       OnLiteralIndexing(CurType, 0);
768:       continue;
769:     }
770:   }
771:
772:   assert(CurType);
773:   ArrayType *AT = dyn_cast<ArrayType>(CurType);
774:   // Operand is not constant. Either we have an array and accept it, or we
775:   // give up.
776:   if (AT)
777:     OnDynamicIndexing(AT->getElementType(), Operand, Multiplier);
778:   return AT == nullptr;
779: }
780:
781: bool SPIRVEmitIntrinsics::walkLogicalAccessChainConstant(
782:     Type *CurType, uint64_t Offset,
783:     const std::function<void(Type *, uint64_t)> &OnLiteralIndexing) {
784:   auto &DL = CurrF->getDataLayout();
785:
786:   do {
787:     if (ArrayType *AT = dyn_cast<ArrayType>(CurType)) {
788:       uint32_t EltTypeSize = DL.getTypeSizeInBits(AT->getElementType()) / 8;
789:       assert(Offset < AT->getNumElements() * EltTypeSize);
790:       uint64_t Index = Offset / EltTypeSize;
791:       Offset = Offset - (Index * EltTypeSize);
792:       CurType = AT->getElementType();
793:       OnLiteralIndexing(CurType, Index);
794:     } else if (StructType *ST = dyn_cast<StructType>(CurType)) {
795:       uint32_t StructSize = DL.getTypeSizeInBits(ST) / 8;
796:       assert(Offset < StructSize);
797:       (void)StructSize;
798:       const auto &STL = DL.getStructLayout(ST);
799:       unsigned Element = STL->getElementContainingOffset(Offset);
800:       Offset -= STL->getElementOffset(Element);
801:       CurType = ST->getElementType(Element);
802:       OnLiteralIndexing(CurType, Element);
803:     } else if (auto *VT = dyn_cast<FixedVectorType>(CurType)) {
804:       Type *EltTy = VT->getElementType();
805:       TypeSize EltSizeBits = DL.getTypeSizeInBits(EltTy);
806:       assert(EltSizeBits % 8 == 0 &&
807:              "Element type size in bits must be a multiple of 8.");
808:       uint32_t EltTypeSize = EltSizeBits / 8;
809:       assert(Offset < VT->getNumElements() * EltTypeSize);
810:       uint64_t Index = Offset / EltTypeSize;
811:       Offset -= Index * EltTypeSize;
812:       CurType = EltTy;
813:       OnLiteralIndexing(CurType, Index);
814:     } else {
815:       // Unknown composite kind; give up.
816:       return true;
817:     }
818:   } while (Offset > 0);
819:
820:   return false;
821: }
822:
823: bool SPIRVEmitIntrinsics::walkLogicalAccessChain(
824:     GetElementPtrInst &GEP,
825:     const std::function<void(Type *, uint64_t)> &OnLiteralIndexing,
826:     const std::function<void(Type *, Value *, uint64_t)> &OnDynamicIndexing) {
827:   // We only rewrite byte-addressing GEP. Other should be left as-is.
828:   // Valid byte-addressing GEP must always have a single index.
829:   std::optional<uint64_t> MultiplierOpt =
830:       getByteAddressingMultiplier(GEP.getSourceElementType());
831:   assert(MultiplierOpt && "We only rewrite byte-addressing GEP");
832:   uint64_t Multiplier = *MultiplierOpt;
833:   assert(GEP.getNumIndices() == 1);
834:
835:   Value *Src = getPointerRoot(GEP.getPointerOperand());
836:   Type *CurType = deduceElementType(Src, true);
837:
838:   Value *Operand = *GEP.idx_begin();
839:   if (ConstantInt *CI = dyn_cast<ConstantInt>(Operand))
840:     return walkLogicalAccessChainConstant(
```
- EN: This range implements operational logic in helpers such as IntegerType::getInt8Ty, deduceElementTypeHelper, insertTodoType, void, translating backend policy into executable code.
- CN: 这一段实现了 IntegerType::getInt8Ty、deduceElementTypeHelper、insertTodoType、void 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 841-960
```cpp
841:         CurType, CI->getZExtValue() * Multiplier, OnLiteralIndexing);
842:
843:   return walkLogicalAccessChainDynamic(CurType, Operand, Multiplier,
844:                                        OnLiteralIndexing, OnDynamicIndexing);
845: }
846:
847: Instruction *
848: SPIRVEmitIntrinsics::buildLogicalAccessChainFromGEP(GetElementPtrInst &GEP) {
849:   auto &DL = CurrF->getDataLayout();
850:   IRBuilder<> B(GEP.getParent());
851:   B.SetInsertPoint(&GEP);
852:
853:   std::vector<Value *> Indices;
854:   Indices.push_back(ConstantInt::get(
855:       IntegerType::getInt32Ty(CurrF->getContext()), 0, /* Signed= */ false));
856:   walkLogicalAccessChain(
857:       GEP,
858:       [&Indices, &B](Type *EltType, uint64_t Index) {
859:         Indices.push_back(
860:             ConstantInt::get(B.getInt64Ty(), Index, /* Signed= */ false));
861:       },
862:       [&Indices, &B, &DL, this](Type *EltType, Value *Offset,
863:                                 uint64_t Multiplier) {
864:         Value *Index = nullptr;
865:         uint32_t EltTypeSize = DL.getTypeSizeInBits(EltType) / 8;
866:         assert(Multiplier != 0);
867:         if (Multiplier == EltTypeSize) {
868:           Index = Offset;
869:         } else if (EltTypeSize % Multiplier == 0) {
870:           Index =
871:               B.CreateUDiv(Offset, ConstantInt::get(Offset->getType(),
872:                                                     EltTypeSize / Multiplier,
873:                                                     /* Signed= */ false));
874:         } else {
875:           Index = B.CreateMul(Offset,
876:                               ConstantInt::get(Offset->getType(), Multiplier,
877:                                                /* Signed= */ false));
878:           insertAssignTypeIntrs(cast<Instruction>(Index), B);
879:           Index = B.CreateUDiv(Index,
880:                                ConstantInt::get(Offset->getType(), EltTypeSize,
881:                                                 /* Signed= */ false));
882:         }
883:         insertAssignTypeIntrs(cast<Instruction>(Index), B);
884:         Indices.push_back(Index);
885:       });
886:
887:   SmallVector<Type *, 2> Types = {GEP.getType(), GEP.getOperand(0)->getType()};
888:   SmallVector<Value *, 4> Args;
889:   Args.push_back(B.getInt1(GEP.isInBounds()));
890:   Args.push_back(GEP.getOperand(0));
891:   llvm::append_range(Args, Indices);
892:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
893:   replaceAllUsesWithAndErase(B, &GEP, NewI);
894:   return NewI;
895: }
896:
897: Type *SPIRVEmitIntrinsics::getGEPTypeLogical(GetElementPtrInst *GEP) {
898:
899:   Type *CurType = GEP->getResultElementType();
900:
901:   bool Interrupted = walkLogicalAccessChain(
902:       *GEP, [&CurType](Type *EltType, uint64_t Index) { CurType = EltType; },
903:       [&CurType](Type *EltType, Value *Index, uint64_t) { CurType = EltType; });
904:
905:   return Interrupted ? GEP->getResultElementType() : CurType;
906: }
907:
908: Type *SPIRVEmitIntrinsics::getGEPType(GetElementPtrInst *Ref) {
909:   if (getByteAddressingMultiplier(Ref->getSourceElementType()) &&
910:       TM.getSubtargetImpl()->isLogicalSPIRV()) {
911:     return getGEPTypeLogical(Ref);
912:   }
913:
914:   Type *Ty = nullptr;
915:   // TODO: not sure if GetElementPtrInst::getTypeAtIndex() does anything
916:   // useful here
917:   if (isNestedPointer(Ref->getSourceElementType())) {
918:     Ty = Ref->getSourceElementType();
919:     for (Use &U : drop_begin(Ref->indices()))
920:       Ty = GetElementPtrInst::getTypeAtIndex(Ty, U.get());
921:   } else {
922:     Ty = Ref->getResultElementType();
923:   }
924:   return Ty;
925: }
926:
927: Type *SPIRVEmitIntrinsics::deduceElementTypeHelper(
928:     Value *I, std::unordered_set<Value *> &Visited, bool UnknownElemTypeI8,
929:     bool IgnoreKnownType) {
930:   // allow to pass nullptr as an argument
931:   if (!I)
932:     return nullptr;
933:
934:   // maybe already known
935:   if (!IgnoreKnownType)
936:     if (Type *KnownTy = GR->findDeducedElementType(I))
937:       return KnownTy;
938:
939:   // maybe a cycle
940:   if (!Visited.insert(I).second)
941:     return nullptr;
942:
943:   // fallback value in case when we fail to deduce a type
944:   Type *Ty = nullptr;
945:   // look for known basic patterns of type inference
946:   if (auto *Ref = dyn_cast<AllocaInst>(I)) {
947:     maybeAssignPtrType(Ty, I, Ref->getAllocatedType(), UnknownElemTypeI8);
948:   } else if (auto *Ref = dyn_cast<GetElementPtrInst>(I)) {
949:     Ty = getGEPType(Ref);
950:   } else if (auto *SGEP = dyn_cast<StructuredGEPInst>(I)) {
951:     Ty = SGEP->getResultElementType();
952:   } else if (auto *Ref = dyn_cast<LoadInst>(I)) {
953:     Value *Op = Ref->getPointerOperand();
954:     Type *KnownTy = GR->findDeducedElementType(Op);
955:     if (!KnownTy)
956:       KnownTy = Op->getType();
957:     if (Type *ElemTy = getPointeeType(KnownTy))
958:       maybeAssignPtrType(Ty, I, ElemTy, UnknownElemTypeI8);
959:   } else if (auto *Ref = dyn_cast<GlobalValue>(I)) {
960:     if (auto *Fn = dyn_cast<Function>(Ref)) {
```
- EN: This range implements operational logic in helpers such as getZExtValue, SPIRVEmitIntrinsics::buildLogicalAccessChainFromGEP, getDataLayout, B, translating backend policy into executable code.
- CN: 这一段实现了 getZExtValue、SPIRVEmitIntrinsics::buildLogicalAccessChainFromGEP、getDataLayout、B 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 961-1080
```cpp
 961:       Ty = SPIRV::getOriginalFunctionType(*Fn);
 962:       GR->addDeducedElementType(I, Ty);
 963:     } else {
 964:       Ty = deduceElementTypeByValueDeep(
 965:           Ref->getValueType(),
 966:           Ref->getNumOperands() > 0 ? Ref->getOperand(0) : nullptr, Visited,
 967:           UnknownElemTypeI8);
 968:     }
 969:   } else if (auto *Ref = dyn_cast<AddrSpaceCastInst>(I)) {
 970:     Type *RefTy = deduceElementTypeHelper(Ref->getPointerOperand(), Visited,
 971:                                           UnknownElemTypeI8);
 972:     maybeAssignPtrType(Ty, I, RefTy, UnknownElemTypeI8);
 973:   } else if (auto *Ref = dyn_cast<IntToPtrInst>(I)) {
 974:     maybeAssignPtrType(Ty, I, Ref->getDestTy(), UnknownElemTypeI8);
 975:   } else if (auto *Ref = dyn_cast<BitCastInst>(I)) {
 976:     if (Type *Src = Ref->getSrcTy(), *Dest = Ref->getDestTy();
 977:         isPointerTy(Src) && isPointerTy(Dest))
 978:       Ty = deduceElementTypeHelper(Ref->getOperand(0), Visited,
 979:                                    UnknownElemTypeI8);
 980:   } else if (auto *Ref = dyn_cast<AtomicCmpXchgInst>(I)) {
 981:     Value *Op = Ref->getNewValOperand();
 982:     if (isPointerTy(Op->getType()))
 983:       Ty = deduceElementTypeHelper(Op, Visited, UnknownElemTypeI8);
 984:   } else if (auto *Ref = dyn_cast<AtomicRMWInst>(I)) {
 985:     Value *Op = Ref->getValOperand();
 986:     if (isPointerTy(Op->getType()))
 987:       Ty = deduceElementTypeHelper(Op, Visited, UnknownElemTypeI8);
 988:   } else if (auto *Ref = dyn_cast<PHINode>(I)) {
 989:     Type *BestTy = nullptr;
 990:     unsigned MaxN = 1;
 991:     DenseMap<Type *, unsigned> PhiTys;
 992:     for (int i = Ref->getNumIncomingValues() - 1; i >= 0; --i) {
 993:       Ty = deduceElementTypeByUsersDeep(Ref->getIncomingValue(i), Visited,
 994:                                         UnknownElemTypeI8);
 995:       if (!Ty)
 996:         continue;
 997:       auto It = PhiTys.try_emplace(Ty, 1);
 998:       if (!It.second) {
 999:         ++It.first->second;
1000:         if (It.first->second > MaxN) {
1001:           MaxN = It.first->second;
1002:           BestTy = Ty;
1003:         }
1004:       }
1005:     }
1006:     if (BestTy)
1007:       Ty = BestTy;
1008:   } else if (auto *Ref = dyn_cast<SelectInst>(I)) {
1009:     for (Value *Op : {Ref->getTrueValue(), Ref->getFalseValue()}) {
1010:       Ty = deduceElementTypeByUsersDeep(Op, Visited, UnknownElemTypeI8);
1011:       if (Ty)
1012:         break;
1013:     }
1014:   } else if (auto *CI = dyn_cast<CallInst>(I)) {
1015:     static StringMap<unsigned> ResTypeByArg = {
1016:         {"to_global", 0},
1017:         {"to_local", 0},
1018:         {"to_private", 0},
1019:         {"__spirv_GenericCastToPtr_ToGlobal", 0},
1020:         {"__spirv_GenericCastToPtr_ToLocal", 0},
1021:         {"__spirv_GenericCastToPtr_ToPrivate", 0},
1022:         {"__spirv_GenericCastToPtrExplicit_ToGlobal", 0},
1023:         {"__spirv_GenericCastToPtrExplicit_ToLocal", 0},
1024:         {"__spirv_GenericCastToPtrExplicit_ToPrivate", 0}};
1025:     // TODO: maybe improve performance by caching demangled names
1026:
1027:     auto *II = dyn_cast<IntrinsicInst>(I);
1028:     if (II && (II->getIntrinsicID() == Intrinsic::spv_resource_getbasepointer ||
1029:                II->getIntrinsicID() == Intrinsic::spv_resource_getpointer)) {
1030:       auto *HandleType = cast<TargetExtType>(II->getOperand(0)->getType());
1031:       if (HandleType->getTargetExtName() == "spirv.Image" ||
1032:           HandleType->getTargetExtName() == "spirv.SignedImage") {
1033:         for (User *U : II->users()) {
1034:           Ty = cast<Instruction>(U)->getAccessType();
1035:           if (Ty)
1036:             break;
1037:         }
1038:       } else if (HandleType->getTargetExtName() == "spirv.VulkanBuffer") {
1039:         // This call is supposed to index into an array
1040:         Ty = HandleType->getTypeParameter(0);
1041:         if (II->getIntrinsicID() == Intrinsic::spv_resource_getpointer) {
1042:           if (Ty->isArrayTy())
1043:             Ty = Ty->getArrayElementType();
1044:           else {
1045:             assert(Ty && Ty->isStructTy());
1046:             uint32_t Index =
1047:                 cast<ConstantInt>(II->getOperand(1))->getZExtValue();
1048:             Ty = cast<StructType>(Ty)->getElementType(Index);
1049:           }
1050:         }
1051:         Ty = reconstitutePeeledArrayType(Ty);
1052:       } else {
1053:         llvm_unreachable("Unknown handle type for spv_resource_getpointer.");
1054:       }
1055:     } else if (II && II->getIntrinsicID() ==
1056:                          Intrinsic::spv_generic_cast_to_ptr_explicit) {
1057:       Ty = deduceElementTypeHelper(CI->getArgOperand(0), Visited,
1058:                                    UnknownElemTypeI8);
1059:     } else if (Function *CalledF = CI->getCalledFunction()) {
1060:       std::string DemangledName =
1061:           getOclOrSpirvBuiltinDemangledName(CalledF->getName());
1062:       if (DemangledName.length() > 0)
1063:         DemangledName = SPIRV::lookupBuiltinNameHelper(DemangledName);
1064:       auto AsArgIt = ResTypeByArg.find(DemangledName);
1065:       if (AsArgIt != ResTypeByArg.end())
1066:         Ty = deduceElementTypeHelper(CI->getArgOperand(AsArgIt->second),
1067:                                      Visited, UnknownElemTypeI8);
1068:       else if (Type *KnownRetTy = GR->findDeducedElementType(CalledF))
1069:         Ty = KnownRetTy;
1070:     }
1071:   }
1072:
1073:   // remember the found relationship
1074:   if (Ty && !IgnoreKnownType) {
1075:     // specify nested types if needed, otherwise return unchanged
1076:     GR->addDeducedElementType(I, normalizeType(Ty));
1077:   }
1078:
1079:   return Ty;
1080: }
```
- EN: This range implements operational logic in helpers such as SPIRV::getOriginalFunctionType, addDeducedElementType, maybeAssignPtrType, isPointerTy, translating backend policy into executable code.
- CN: 这一段实现了 SPIRV::getOriginalFunctionType、addDeducedElementType、maybeAssignPtrType、isPointerTy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1200
```cpp
1081:
1082: // Re-create a type of the value if it has untyped pointer fields, also nested.
1083: // Return the original value type if no corrections of untyped pointer
1084: // information is found or needed.
1085: Type *SPIRVEmitIntrinsics::deduceNestedTypeHelper(User *U,
1086:                                                   bool UnknownElemTypeI8) {
1087:   std::unordered_set<Value *> Visited;
1088:   return deduceNestedTypeHelper(U, U->getType(), Visited, UnknownElemTypeI8);
1089: }
1090:
1091: Type *SPIRVEmitIntrinsics::deduceNestedTypeHelper(
1092:     User *U, Type *OrigTy, std::unordered_set<Value *> &Visited,
1093:     bool UnknownElemTypeI8) {
1094:   if (!U)
1095:     return OrigTy;
1096:
1097:   // maybe already known
1098:   if (Type *KnownTy = GR->findDeducedCompositeType(U))
1099:     return KnownTy;
1100:
1101:   // maybe a cycle
1102:   if (!Visited.insert(U).second)
1103:     return OrigTy;
1104:
1105:   if (isa<StructType>(OrigTy)) {
1106:     SmallVector<Type *> Tys;
1107:     bool Change = false;
1108:     for (unsigned i = 0; i < U->getNumOperands(); ++i) {
1109:       Value *Op = U->getOperand(i);
1110:       assert(Op && "Operands should not be null.");
1111:       Type *OpTy = Op->getType();
1112:       Type *Ty = OpTy;
1113:       if (auto *PtrTy = dyn_cast<PointerType>(OpTy)) {
1114:         if (Type *NestedTy =
1115:                 deduceElementTypeHelper(Op, Visited, UnknownElemTypeI8))
1116:           Ty = getTypedPointerWrapper(NestedTy, PtrTy->getAddressSpace());
1117:       } else {
1118:         Ty = deduceNestedTypeHelper(dyn_cast<User>(Op), OpTy, Visited,
1119:                                     UnknownElemTypeI8);
1120:       }
1121:       Tys.push_back(Ty);
1122:       Change |= Ty != OpTy;
1123:     }
1124:     if (Change) {
1125:       Type *NewTy = StructType::create(Tys);
1126:       GR->addDeducedCompositeType(U, NewTy);
1127:       return NewTy;
1128:     }
1129:   } else if (auto *ArrTy = dyn_cast<ArrayType>(OrigTy)) {
1130:     if (Value *Op = U->getNumOperands() > 0 ? U->getOperand(0) : nullptr) {
1131:       Type *OpTy = ArrTy->getElementType();
1132:       Type *Ty = OpTy;
1133:       if (auto *PtrTy = dyn_cast<PointerType>(OpTy)) {
1134:         if (Type *NestedTy =
1135:                 deduceElementTypeHelper(Op, Visited, UnknownElemTypeI8))
1136:           Ty = getTypedPointerWrapper(NestedTy, PtrTy->getAddressSpace());
1137:       } else {
1138:         Ty = deduceNestedTypeHelper(dyn_cast<User>(Op), OpTy, Visited,
1139:                                     UnknownElemTypeI8);
1140:       }
1141:       if (Ty != OpTy) {
1142:         Type *NewTy = ArrayType::get(Ty, ArrTy->getNumElements());
1143:         GR->addDeducedCompositeType(U, NewTy);
1144:         return NewTy;
1145:       }
1146:     }
1147:   } else if (auto *VecTy = dyn_cast<VectorType>(OrigTy)) {
1148:     if (Value *Op = U->getNumOperands() > 0 ? U->getOperand(0) : nullptr) {
1149:       Type *OpTy = VecTy->getElementType();
1150:       Type *Ty = OpTy;
1151:       if (auto *PtrTy = dyn_cast<PointerType>(OpTy)) {
1152:         if (Type *NestedTy =
1153:                 deduceElementTypeHelper(Op, Visited, UnknownElemTypeI8))
1154:           Ty = getTypedPointerWrapper(NestedTy, PtrTy->getAddressSpace());
1155:       } else {
1156:         Ty = deduceNestedTypeHelper(dyn_cast<User>(Op), OpTy, Visited,
1157:                                     UnknownElemTypeI8);
1158:       }
1159:       if (Ty != OpTy) {
1160:         Type *NewTy = VectorType::get(Ty, VecTy->getElementCount());
1161:         GR->addDeducedCompositeType(U, normalizeType(NewTy));
1162:         return NewTy;
1163:       }
1164:     }
1165:   }
1166:
1167:   return OrigTy;
1168: }
1169:
1170: Type *SPIRVEmitIntrinsics::deduceElementType(Value *I, bool UnknownElemTypeI8) {
1171:   if (Type *Ty = deduceElementTypeHelper(I, UnknownElemTypeI8))
1172:     return Ty;
1173:   if (!UnknownElemTypeI8)
1174:     return nullptr;
1175:   insertTodoType(I);
1176:   return IntegerType::getInt8Ty(I->getContext());
1177: }
1178:
1179: static inline Type *getAtomicElemTy(SPIRVGlobalRegistry *GR, Instruction *I,
1180:                                     Value *PointerOperand) {
1181:   Type *PointeeTy = GR->findDeducedElementType(PointerOperand);
1182:   if (PointeeTy && !isUntypedPointerTy(PointeeTy))
1183:     return nullptr;
1184:   auto *PtrTy = dyn_cast<PointerType>(I->getType());
1185:   if (!PtrTy)
1186:     return I->getType();
1187:   if (Type *NestedTy = GR->findDeducedElementType(I))
1188:     return getTypedPointerWrapper(NestedTy, PtrTy->getAddressSpace());
1189:   return nullptr;
1190: }
1191:
1192: // Try to deduce element type for a call base. Returns false if this is an
1193: // indirect function invocation, and true otherwise.
1194: bool SPIRVEmitIntrinsics::deduceOperandElementTypeCalledFunction(
1195:     CallInst *CI, SmallVector<std::pair<Value *, unsigned>> &Ops,
1196:     Type *&KnownElemTy, bool &Incomplete) {
1197:   Function *CalledF = CI->getCalledFunction();
1198:   if (!CalledF)
1199:     return false;
1200:   std::string DemangledName =
```
- EN: This range implements operational logic in helpers such as deduceNestedTypeHelper, getNumOperands, getOperand, assert, translating backend policy into executable code.
- CN: 这一段实现了 deduceNestedTypeHelper、getNumOperands、getOperand、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1201-1320
```cpp
1201:       getOclOrSpirvBuiltinDemangledName(CalledF->getName());
1202:   if (DemangledName.length() > 0 &&
1203:       !StringRef(DemangledName).starts_with("llvm.")) {
1204:     const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(*CalledF);
1205:     auto [Grp, Opcode, ExtNo] = SPIRV::mapBuiltinToOpcode(
1206:         DemangledName, ST.getPreferredInstructionSet());
1207:     if (Opcode == SPIRV::OpGroupAsyncCopy) {
1208:       for (unsigned i = 0, PtrCnt = 0; i < CI->arg_size() && PtrCnt < 2; ++i) {
1209:         Value *Op = CI->getArgOperand(i);
1210:         if (!isPointerTy(Op->getType()))
1211:           continue;
1212:         ++PtrCnt;
1213:         if (Type *ElemTy = GR->findDeducedElementType(Op))
1214:           KnownElemTy = ElemTy; // src will rewrite dest if both are defined
1215:         Ops.push_back(std::make_pair(Op, i));
1216:       }
1217:     } else if (Grp == SPIRV::Atomic || Grp == SPIRV::AtomicFloating) {
1218:       if (CI->arg_size() == 0)
1219:         return true;
1220:       Value *Op = CI->getArgOperand(0);
1221:       if (!isPointerTy(Op->getType()))
1222:         return true;
1223:       switch (Opcode) {
1224:       case SPIRV::OpAtomicFAddEXT:
1225:       case SPIRV::OpAtomicFMinEXT:
1226:       case SPIRV::OpAtomicFMaxEXT:
1227:       case SPIRV::OpAtomicLoad:
1228:       case SPIRV::OpAtomicCompareExchangeWeak:
1229:       case SPIRV::OpAtomicCompareExchange:
1230:       case SPIRV::OpAtomicExchange:
1231:       case SPIRV::OpAtomicIAdd:
1232:       case SPIRV::OpAtomicISub:
1233:       case SPIRV::OpAtomicOr:
1234:       case SPIRV::OpAtomicXor:
1235:       case SPIRV::OpAtomicAnd:
1236:       case SPIRV::OpAtomicUMin:
1237:       case SPIRV::OpAtomicUMax:
1238:       case SPIRV::OpAtomicSMin:
1239:       case SPIRV::OpAtomicSMax: {
1240:         KnownElemTy = isPointerTy(CI->getType()) ? getAtomicElemTy(GR, CI, Op)
1241:                                                  : CI->getType();
1242:         if (!KnownElemTy)
1243:           return true;
1244:         Incomplete = isTodoType(Op);
1245:         Ops.push_back(std::make_pair(Op, 0));
1246:       } break;
1247:       case SPIRV::OpAtomicStore: {
1248:         if (CI->arg_size() < 4)
1249:           return true;
1250:         Value *ValOp = CI->getArgOperand(3);
1251:         KnownElemTy = isPointerTy(ValOp->getType())
1252:                           ? getAtomicElemTy(GR, CI, Op)
1253:                           : ValOp->getType();
1254:         if (!KnownElemTy)
1255:           return true;
1256:         Incomplete = isTodoType(Op);
1257:         Ops.push_back(std::make_pair(Op, 0));
1258:       } break;
1259:       }
1260:     }
1261:   }
1262:   return true;
1263: }
1264:
1265: // Try to deduce element type for a function pointer.
1266: void SPIRVEmitIntrinsics::deduceOperandElementTypeFunctionPointer(
1267:     CallInst *CI, SmallVector<std::pair<Value *, unsigned>> &Ops,
1268:     Type *&KnownElemTy, bool IsPostprocessing) {
1269:   Value *Op = CI->getCalledOperand();
1270:   if (!Op || !isPointerTy(Op->getType()))
1271:     return;
1272:   Ops.push_back(std::make_pair(Op, std::numeric_limits<unsigned>::max()));
1273:   FunctionType *FTy = SPIRV::getOriginalFunctionType(*CI);
1274:   bool IsNewFTy = false, IsIncomplete = false;
1275:   SmallVector<Type *, 4> ArgTys;
1276:   for (auto &&[ParmIdx, Arg] : llvm::enumerate(CI->args())) {
1277:     Type *ArgTy = Arg->getType();
1278:     if (ArgTy->isPointerTy()) {
1279:       if (Type *ElemTy = GR->findDeducedElementType(Arg)) {
1280:         IsNewFTy = true;
1281:         ArgTy = getTypedPointerWrapper(ElemTy, getPointerAddressSpace(ArgTy));
1282:         if (isTodoType(Arg))
1283:           IsIncomplete = true;
1284:       } else {
1285:         IsIncomplete = true;
1286:       }
1287:     } else {
1288:       ArgTy = FTy->getFunctionParamType(ParmIdx);
1289:     }
1290:     ArgTys.push_back(ArgTy);
1291:   }
1292:   Type *RetTy = FTy->getReturnType();
1293:   if (CI->getType()->isPointerTy()) {
1294:     if (Type *ElemTy = GR->findDeducedElementType(CI)) {
1295:       IsNewFTy = true;
1296:       RetTy =
1297:           getTypedPointerWrapper(ElemTy, getPointerAddressSpace(CI->getType()));
1298:       if (isTodoType(CI))
1299:         IsIncomplete = true;
1300:     } else {
1301:       IsIncomplete = true;
1302:     }
1303:   }
1304:   if (!IsPostprocessing && IsIncomplete)
1305:     insertTodoType(Op);
1306:   KnownElemTy =
1307:       IsNewFTy ? FunctionType::get(RetTy, ArgTys, FTy->isVarArg()) : FTy;
1308: }
1309:
1310: bool SPIRVEmitIntrinsics::deduceOperandElementTypeFunctionRet(
1311:     Instruction *I, SmallPtrSet<Instruction *, 4> *IncompleteRets,
1312:     const SmallPtrSet<Value *, 4> *AskOps, bool IsPostprocessing,
1313:     Type *&KnownElemTy, Value *Op, Function *F) {
1314:   KnownElemTy = GR->findDeducedElementType(F);
1315:   if (KnownElemTy)
1316:     return false;
1317:   if (Type *OpElemTy = GR->findDeducedElementType(Op)) {
1318:     OpElemTy = normalizeType(OpElemTy);
1319:     GR->addDeducedElementType(F, OpElemTy);
1320:     GR->addReturnType(
```
- EN: This range implements operational logic in helpers such as getOclOrSpirvBuiltinDemangledName, StringRef, getPreferredInstructionSet, getArgOperand, translating backend policy into executable code.
- CN: 这一段实现了 getOclOrSpirvBuiltinDemangledName、StringRef、getPreferredInstructionSet、getArgOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1321-1440
```cpp
1321:         F, TypedPointerType::get(OpElemTy,
1322:                                  getPointerAddressSpace(F->getReturnType())));
1323:     // non-recursive update of types in function uses
1324:     DenseSet<std::pair<Value *, Value *>> VisitedSubst{std::make_pair(I, Op)};
1325:     for (User *U : F->users()) {
1326:       CallInst *CI = dyn_cast<CallInst>(U);
1327:       if (!CI || CI->getCalledFunction() != F)
1328:         continue;
1329:       if (CallInst *AssignCI = GR->findAssignPtrTypeInstr(CI)) {
1330:         if (Type *PrevElemTy = GR->findDeducedElementType(CI)) {
1331:           GR->updateAssignType(AssignCI, CI,
1332:                                getNormalizedPoisonValue(OpElemTy));
1333:           propagateElemType(CI, PrevElemTy, VisitedSubst);
1334:         }
1335:       }
1336:     }
1337:     // Non-recursive update of types in the function uncomplete returns.
1338:     // This may happen just once per a function, the latch is a pair of
1339:     // findDeducedElementType(F) / addDeducedElementType(F, ...).
1340:     // With or without the latch it is a non-recursive call due to
1341:     // IncompleteRets set to nullptr in this call.
1342:     if (IncompleteRets)
1343:       for (Instruction *IncompleteRetI : *IncompleteRets)
1344:         deduceOperandElementType(IncompleteRetI, nullptr, AskOps,
1345:                                  IsPostprocessing);
1346:   } else if (IncompleteRets) {
1347:     IncompleteRets->insert(I);
1348:   }
1349:   TypeValidated.insert(I);
1350:   return true;
1351: }
1352:
1353: // If the Instruction has Pointer operands with unresolved types, this function
1354: // tries to deduce them. If the Instruction has Pointer operands with known
1355: // types which differ from expected, this function tries to insert a bitcast to
1356: // resolve the issue.
1357: void SPIRVEmitIntrinsics::deduceOperandElementType(
1358:     Instruction *I, SmallPtrSet<Instruction *, 4> *IncompleteRets,
1359:     const SmallPtrSet<Value *, 4> *AskOps, bool IsPostprocessing) {
1360:   SmallVector<std::pair<Value *, unsigned>> Ops;
1361:   Type *KnownElemTy = nullptr;
1362:   bool Incomplete = false;
1363:   // look for known basic patterns of type inference
1364:   if (auto *Ref = dyn_cast<PHINode>(I)) {
1365:     if (!isPointerTy(I->getType()) ||
1366:         !(KnownElemTy = GR->findDeducedElementType(I)))
1367:       return;
1368:     Incomplete = isTodoType(I);
1369:     for (unsigned i = 0; i < Ref->getNumIncomingValues(); i++) {
1370:       Value *Op = Ref->getIncomingValue(i);
1371:       if (isPointerTy(Op->getType()))
1372:         Ops.push_back(std::make_pair(Op, i));
1373:     }
1374:   } else if (auto *Ref = dyn_cast<AddrSpaceCastInst>(I)) {
1375:     KnownElemTy = GR->findDeducedElementType(I);
1376:     if (!KnownElemTy)
1377:       return;
1378:     Incomplete = isTodoType(I);
1379:     Ops.push_back(std::make_pair(Ref->getPointerOperand(), 0));
1380:   } else if (auto *Ref = dyn_cast<BitCastInst>(I)) {
1381:     if (!isPointerTy(I->getType()))
1382:       return;
1383:     KnownElemTy = GR->findDeducedElementType(I);
1384:     if (!KnownElemTy)
1385:       return;
1386:     Incomplete = isTodoType(I);
1387:     Ops.push_back(std::make_pair(Ref->getOperand(0), 0));
1388:   } else if (auto *Ref = dyn_cast<GetElementPtrInst>(I)) {
1389:     if (GR->findDeducedElementType(Ref->getPointerOperand()))
1390:       return;
1391:     KnownElemTy = Ref->getSourceElementType();
1392:     Ops.push_back(std::make_pair(Ref->getPointerOperand(),
1393:                                  GetElementPtrInst::getPointerOperandIndex()));
1394:   } else if (auto *Ref = dyn_cast<StructuredGEPInst>(I)) {
1395:     if (GR->findDeducedElementType(Ref->getPointerOperand()))
1396:       return;
1397:     KnownElemTy = Ref->getBaseType();
1398:     Ops.push_back(std::make_pair(Ref->getPointerOperand(),
1399:                                  StructuredGEPInst::getPointerOperandIndex()));
1400:   } else if (auto *Ref = dyn_cast<LoadInst>(I)) {
1401:     KnownElemTy = I->getType();
1402:     if (isUntypedPointerTy(KnownElemTy))
1403:       return;
1404:     Type *PointeeTy = GR->findDeducedElementType(Ref->getPointerOperand());
1405:     if (PointeeTy && !isUntypedPointerTy(PointeeTy))
1406:       return;
1407:     Ops.push_back(std::make_pair(Ref->getPointerOperand(),
1408:                                  LoadInst::getPointerOperandIndex()));
1409:   } else if (auto *Ref = dyn_cast<StoreInst>(I)) {
1410:     if (!(KnownElemTy =
1411:               reconstructType(Ref->getValueOperand(), false, IsPostprocessing)))
1412:       return;
1413:     Type *PointeeTy = GR->findDeducedElementType(Ref->getPointerOperand());
1414:     if (PointeeTy && !isUntypedPointerTy(PointeeTy))
1415:       return;
1416:     Ops.push_back(std::make_pair(Ref->getPointerOperand(),
1417:                                  StoreInst::getPointerOperandIndex()));
1418:   } else if (auto *Ref = dyn_cast<AtomicCmpXchgInst>(I)) {
1419:     KnownElemTy = isPointerTy(I->getType())
1420:                       ? getAtomicElemTy(GR, I, Ref->getPointerOperand())
1421:                       : I->getType();
1422:     if (!KnownElemTy)
1423:       return;
1424:     Incomplete = isTodoType(Ref->getPointerOperand());
1425:     Ops.push_back(std::make_pair(Ref->getPointerOperand(),
1426:                                  AtomicCmpXchgInst::getPointerOperandIndex()));
1427:   } else if (auto *Ref = dyn_cast<AtomicRMWInst>(I)) {
1428:     KnownElemTy = isPointerTy(I->getType())
1429:                       ? getAtomicElemTy(GR, I, Ref->getPointerOperand())
1430:                       : I->getType();
1431:     if (!KnownElemTy)
1432:       return;
1433:     Incomplete = isTodoType(Ref->getPointerOperand());
1434:     Ops.push_back(std::make_pair(Ref->getPointerOperand(),
1435:                                  AtomicRMWInst::getPointerOperandIndex()));
1436:   } else if (auto *Ref = dyn_cast<SelectInst>(I)) {
1437:     if (!isPointerTy(I->getType()) ||
1438:         !(KnownElemTy = GR->findDeducedElementType(I)))
1439:       return;
1440:     Incomplete = isTodoType(I);
```
- EN: This range implements operational logic in helpers such as getPointerAddressSpace, getNormalizedPoisonValue, propagateElemType, insert, translating backend policy into executable code.
- CN: 这一段实现了 getPointerAddressSpace、getNormalizedPoisonValue、propagateElemType、insert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1441-1560
```cpp
1441:     for (unsigned i = 0; i < Ref->getNumOperands(); i++) {
1442:       Value *Op = Ref->getOperand(i);
1443:       if (isPointerTy(Op->getType()))
1444:         Ops.push_back(std::make_pair(Op, i));
1445:     }
1446:   } else if (auto *Ref = dyn_cast<ReturnInst>(I)) {
1447:     if (!isPointerTy(CurrF->getReturnType()))
1448:       return;
1449:     Value *Op = Ref->getReturnValue();
1450:     if (!Op)
1451:       return;
1452:     if (deduceOperandElementTypeFunctionRet(I, IncompleteRets, AskOps,
1453:                                             IsPostprocessing, KnownElemTy, Op,
1454:                                             CurrF))
1455:       return;
1456:     Incomplete = isTodoType(CurrF);
1457:     Ops.push_back(std::make_pair(Op, 0));
1458:   } else if (auto *Ref = dyn_cast<ICmpInst>(I)) {
1459:     if (!isPointerTy(Ref->getOperand(0)->getType()))
1460:       return;
1461:     Value *Op0 = Ref->getOperand(0);
1462:     Value *Op1 = Ref->getOperand(1);
1463:     bool Incomplete0 = isTodoType(Op0);
1464:     bool Incomplete1 = isTodoType(Op1);
1465:     Type *ElemTy1 = GR->findDeducedElementType(Op1);
1466:     Type *ElemTy0 = (Incomplete0 && !Incomplete1 && ElemTy1)
1467:                         ? nullptr
1468:                         : GR->findDeducedElementType(Op0);
1469:     if (ElemTy0) {
1470:       KnownElemTy = ElemTy0;
1471:       Incomplete = Incomplete0;
1472:       Ops.push_back(std::make_pair(Op1, 1));
1473:     } else if (ElemTy1) {
1474:       KnownElemTy = ElemTy1;
1475:       Incomplete = Incomplete1;
1476:       Ops.push_back(std::make_pair(Op0, 0));
1477:     }
1478:   } else if (CallInst *CI = dyn_cast<CallInst>(I)) {
1479:     if (!CI->isIndirectCall())
1480:       deduceOperandElementTypeCalledFunction(CI, Ops, KnownElemTy, Incomplete);
1481:     else if (HaveFunPtrs)
1482:       deduceOperandElementTypeFunctionPointer(CI, Ops, KnownElemTy,
1483:                                               IsPostprocessing);
1484:   }
1485:
1486:   // There is no enough info to deduce types or all is valid.
1487:   if (!KnownElemTy || Ops.size() == 0)
1488:     return;
1489:
1490:   LLVMContext &Ctx = CurrF->getContext();
1491:   IRBuilder<> B(Ctx);
1492:   for (auto &OpIt : Ops) {
1493:     Value *Op = OpIt.first;
1494:     if (AskOps && !AskOps->contains(Op))
1495:       continue;
1496:     Type *AskTy = nullptr;
1497:     CallInst *AskCI = nullptr;
1498:     if (IsPostprocessing && AskOps) {
1499:       AskTy = GR->findDeducedElementType(Op);
1500:       AskCI = GR->findAssignPtrTypeInstr(Op);
1501:       assert(AskTy && AskCI);
1502:     }
1503:     Type *Ty = AskTy ? AskTy : GR->findDeducedElementType(Op);
1504:     if (Ty == KnownElemTy)
1505:       continue;
1506:     Value *OpTyVal = getNormalizedPoisonValue(KnownElemTy);
1507:     Type *OpTy = Op->getType();
1508:     if (Op->hasUseList() &&
1509:         (!Ty || AskTy || isUntypedPointerTy(Ty) || isTodoType(Op))) {
1510:       Type *PrevElemTy = GR->findDeducedElementType(Op);
1511:       GR->addDeducedElementType(Op, normalizeType(KnownElemTy));
1512:       // check if KnownElemTy is complete
1513:       if (!Incomplete)
1514:         eraseTodoType(Op);
1515:       else if (!IsPostprocessing)
1516:         insertTodoType(Op);
1517:       // check if there is existing Intrinsic::spv_assign_ptr_type instruction
1518:       CallInst *AssignCI = AskCI ? AskCI : GR->findAssignPtrTypeInstr(Op);
1519:       if (AssignCI == nullptr) {
1520:         Instruction *User = dyn_cast<Instruction>(Op->use_begin()->get());
1521:         setInsertPointSkippingPhis(B, User ? User->getNextNode() : I);
1522:         CallInst *CI =
1523:             buildIntrWithMD(Intrinsic::spv_assign_ptr_type, {OpTy}, OpTyVal, Op,
1524:                             {B.getInt32(getPointerAddressSpace(OpTy))}, B);
1525:         GR->addAssignPtrTypeInstr(Op, CI);
1526:       } else {
1527:         GR->updateAssignType(AssignCI, Op, OpTyVal);
1528:         DenseSet<std::pair<Value *, Value *>> VisitedSubst{
1529:             std::make_pair(I, Op)};
1530:         propagateElemTypeRec(Op, KnownElemTy, PrevElemTy, VisitedSubst);
1531:       }
1532:     } else {
1533:       eraseTodoType(Op);
1534:       CallInst *PtrCastI =
1535:           buildSpvPtrcast(I->getParent()->getParent(), Op, KnownElemTy);
1536:       if (OpIt.second == std::numeric_limits<unsigned>::max())
1537:         dyn_cast<CallInst>(I)->setCalledOperand(PtrCastI);
1538:       else
1539:         I->setOperand(OpIt.second, PtrCastI);
1540:     }
1541:   }
1542:   TypeValidated.insert(I);
1543: }
1544:
1545: void SPIRVEmitIntrinsics::replaceMemInstrUses(Instruction *Old,
1546:                                               Instruction *New,
1547:                                               IRBuilder<> &B) {
1548:   while (!Old->user_empty()) {
1549:     auto *U = Old->user_back();
1550:     if (isAssignTypeInstr(U)) {
1551:       B.SetInsertPoint(U);
1552:       SmallVector<Value *, 2> Args = {New, U->getOperand(1)};
1553:       CallInst *AssignCI =
1554:           B.CreateIntrinsic(Intrinsic::spv_assign_type, {New->getType()}, Args);
1555:       GR->addAssignPtrTypeInstr(New, AssignCI);
1556:       U->eraseFromParent();
1557:     } else if (isMemInstrToReplace(U) || isa<ReturnInst>(U) ||
1558:                isa<CallInst>(U)) {
1559:       U->replaceUsesOfWith(Old, New);
1560:       // For a `llvm.spv.abort` call whose composite message argument was
```
- EN: This range implements operational logic in helpers such as getNumOperands, getOperand, push_back, getReturnValue, translating backend policy into executable code.
- CN: 这一段实现了 getNumOperands、getOperand、push_back、getReturnValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1561-1680
```cpp
1561:       // rewritten to a value-id (i32), also retarget the call to a matching
1562:       // intrinsic declaration so the IR verifier is satisfied. The SPIR-V
1563:       // type of the value is tracked via the GlobalRegistry, so the selector
1564:       // still emits OpAbortKHR with the original composite type.
1565:       if (auto *CI = dyn_cast<CallInst>(U);
1566:           CI && CI->getIntrinsicID() == Intrinsic::spv_abort) {
1567:         Type *NewArgTy = New->getType();
1568:         Type *ExpectedArgTy = CI->getFunctionType()->getParamType(0);
1569:         if (NewArgTy != ExpectedArgTy) {
1570:           Module *M = CI->getModule();
1571:           Function *NewF = Intrinsic::getOrInsertDeclaration(
1572:               M, Intrinsic::spv_abort, {NewArgTy});
1573:           CI->setCalledFunction(NewF);
1574:         }
1575:       }
1576:     } else if (auto *Phi = dyn_cast<PHINode>(U)) {
1577:       if (Phi->getType() != New->getType()) {
1578:         Phi->mutateType(New->getType());
1579:         Phi->replaceUsesOfWith(Old, New);
1580:         // Convert extractvalue users of the mutated PHI to spv_extractv
1581:         SmallVector<ExtractValueInst *, 4> EVUsers;
1582:         for (User *PhiUser : Phi->users())
1583:           if (auto *EV = dyn_cast<ExtractValueInst>(PhiUser))
1584:             EVUsers.push_back(EV);
1585:         for (ExtractValueInst *EV : EVUsers) {
1586:           B.SetInsertPoint(EV);
1587:           SmallVector<Value *> Args(EV->operand_values());
1588:           for (unsigned Idx : EV->indices())
1589:             Args.push_back(B.getInt32(Idx));
1590:           auto *NewEV =
1591:               B.CreateIntrinsic(Intrinsic::spv_extractv, {EV->getType()}, Args);
1592:           EV->replaceAllUsesWith(NewEV);
1593:           DeletedInstrs.insert(EV);
1594:           EV->eraseFromParent();
1595:         }
1596:       } else {
1597:         Phi->replaceUsesOfWith(Old, New);
1598:       }
1599:     } else {
1600:       llvm_unreachable("illegal aggregate intrinsic user");
1601:     }
1602:   }
1603:   New->copyMetadata(*Old);
1604:   Old->eraseFromParent();
1605: }
1606:
1607: void SPIRVEmitIntrinsics::preprocessUndefs(IRBuilder<> &B) {
1608:   std::queue<Instruction *> Worklist;
1609:   for (auto &I : instructions(CurrF))
1610:     Worklist.push(&I);
1611:
1612:   while (!Worklist.empty()) {
1613:     Instruction *I = Worklist.front();
1614:     bool BPrepared = false;
1615:     Worklist.pop();
1616:
1617:     for (auto &Op : I->operands()) {
1618:       auto *AggrUndef = dyn_cast<UndefValue>(Op);
1619:       if (!AggrUndef || !Op->getType()->isAggregateType())
1620:         continue;
1621:
1622:       if (!BPrepared) {
1623:         setInsertPointSkippingPhis(B, I);
1624:         BPrepared = true;
1625:       }
1626:       auto *IntrUndef = B.CreateIntrinsic(Intrinsic::spv_undef, {});
1627:       Worklist.push(IntrUndef);
1628:       I->replaceUsesOfWith(Op, IntrUndef);
1629:       AggrConsts[IntrUndef] = AggrUndef;
1630:       AggrConstTypes[IntrUndef] = AggrUndef->getType();
1631:     }
1632:   }
1633: }
1634:
1635: // Simplify addrspacecast(null) instructions to ConstantPointerNull of the
1636: // target type. Casting null always yields null, and this avoids SPIR-V
1637: // lowering issues where the null gets typed as an integer instead of a
1638: // pointer.
1639: void SPIRVEmitIntrinsics::simplifyNullAddrSpaceCasts() {
1640:   for (Instruction &I : make_early_inc_range(instructions(CurrF)))
1641:     if (auto *ASC = dyn_cast<AddrSpaceCastInst>(&I))
1642:       if (isa<ConstantPointerNull>(ASC->getPointerOperand())) {
1643:         ASC->replaceAllUsesWith(
1644:             ConstantPointerNull::get(cast<PointerType>(ASC->getType())));
1645:         ASC->eraseFromParent();
1646:       }
1647: }
1648:
1649: void SPIRVEmitIntrinsics::preprocessCompositeConstants(IRBuilder<> &B) {
1650:   std::queue<Instruction *> Worklist;
1651:   for (auto &I : instructions(CurrF))
1652:     Worklist.push(&I);
1653:
1654:   while (!Worklist.empty()) {
1655:     auto *I = Worklist.front();
1656:     bool IsPhi = isa<PHINode>(I), BPrepared = false;
1657:     assert(I);
1658:     bool KeepInst = false;
1659:     for (const auto &Op : I->operands()) {
1660:       Constant *AggrConst = nullptr;
1661:       Type *ResTy = nullptr;
1662:       if (auto *COp = dyn_cast<ConstantVector>(Op)) {
1663:         AggrConst = COp;
1664:         ResTy = COp->getType();
1665:       } else if (auto *COp = dyn_cast<ConstantArray>(Op)) {
1666:         AggrConst = COp;
1667:         ResTy = B.getInt32Ty();
1668:       } else if (auto *COp = dyn_cast<ConstantStruct>(Op)) {
1669:         AggrConst = COp;
1670:         ResTy = B.getInt32Ty();
1671:       } else if (auto *COp = dyn_cast<ConstantDataArray>(Op)) {
1672:         AggrConst = COp;
1673:         ResTy = B.getInt32Ty();
1674:       } else if (auto *COp = dyn_cast<ConstantAggregateZero>(Op)) {
1675:         AggrConst = COp;
1676:         ResTy = Op->getType()->isVectorTy() ? COp->getType() : B.getInt32Ty();
1677:       }
1678:       if (AggrConst) {
1679:         SmallVector<Value *> Args;
1680:         if (auto *COp = dyn_cast<ConstantDataSequential>(Op))
```
- EN: This range implements operational logic in helpers such as getIntrinsicID, getType, getFunctionType, getModule, translating backend policy into executable code.
- CN: 这一段实现了 getIntrinsicID、getType、getFunctionType、getModule 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1681-1800
```cpp
1681:           for (unsigned i = 0; i < COp->getNumElements(); ++i)
1682:             Args.push_back(COp->getElementAsConstant(i));
1683:         else
1684:           for (Value *Op : AggrConst->operands()) {
1685:             // Simplify addrspacecast(null) to null in the target address space
1686:             // so that null pointers get the correct pointer type when lowered.
1687:             if (auto *CE = dyn_cast<ConstantExpr>(Op);
1688:                 CE && CE->getOpcode() == Instruction::AddrSpaceCast &&
1689:                 isa<ConstantPointerNull>(CE->getOperand(0)))
1690:               Op = ConstantPointerNull::get(cast<PointerType>(CE->getType()));
1691:             Args.push_back(Op);
1692:           }
1693:         if (!BPrepared) {
1694:           IsPhi ? B.SetInsertPointPastAllocas(I->getParent()->getParent())
1695:                 : B.SetInsertPoint(I);
1696:           BPrepared = true;
1697:         }
1698:         auto *CI =
1699:             B.CreateIntrinsic(Intrinsic::spv_const_composite, {ResTy}, {Args});
1700:         Worklist.push(CI);
1701:         I->replaceUsesOfWith(Op, CI);
1702:         KeepInst = true;
1703:         AggrConsts[CI] = AggrConst;
1704:         AggrConstTypes[CI] = deduceNestedTypeHelper(AggrConst, false);
1705:       }
1706:     }
1707:     if (!KeepInst)
1708:       Worklist.pop();
1709:   }
1710: }
1711:
1712: static void createDecorationIntrinsic(Instruction *I, MDNode *Node,
1713:                                       IRBuilder<> &B) {
1714:   LLVMContext &Ctx = I->getContext();
1715:   setInsertPointAfterDef(B, I);
1716:   B.CreateIntrinsic(Intrinsic::spv_assign_decoration, {I->getType()},
1717:                     {I, MetadataAsValue::get(Ctx, MDNode::get(Ctx, {Node}))});
1718: }
1719:
1720: static void createRoundingModeDecoration(Instruction *I,
1721:                                          unsigned RoundingModeDeco,
1722:                                          IRBuilder<> &B) {
1723:   LLVMContext &Ctx = I->getContext();
1724:   Type *Int32Ty = Type::getInt32Ty(Ctx);
1725:   MDNode *RoundingModeNode = MDNode::get(
1726:       Ctx,
1727:       {ConstantAsMetadata::get(
1728:            ConstantInt::get(Int32Ty, SPIRV::Decoration::FPRoundingMode)),
1729:        ConstantAsMetadata::get(ConstantInt::get(Int32Ty, RoundingModeDeco))});
1730:   createDecorationIntrinsic(I, RoundingModeNode, B);
1731: }
1732:
1733: static void createSaturatedConversionDecoration(Instruction *I,
1734:                                                 IRBuilder<> &B) {
1735:   LLVMContext &Ctx = I->getContext();
1736:   Type *Int32Ty = Type::getInt32Ty(Ctx);
1737:   MDNode *SaturatedConversionNode =
1738:       MDNode::get(Ctx, {ConstantAsMetadata::get(ConstantInt::get(
1739:                            Int32Ty, SPIRV::Decoration::SaturatedConversion))});
1740:   createDecorationIntrinsic(I, SaturatedConversionNode, B);
1741: }
1742:
1743: static void addSaturatedDecorationToIntrinsic(Instruction *I, IRBuilder<> &B) {
1744:   if (match(I, m_AnyIntrinsic<Intrinsic::fptosi_sat, Intrinsic::fptoui_sat>()))
1745:     createSaturatedConversionDecoration(I, B);
1746: }
1747:
1748: Instruction *SPIRVEmitIntrinsics::visitCallInst(CallInst &Call) {
1749:   if (!Call.isInlineAsm())
1750:     return &Call;
1751:
1752:   LLVMContext &Ctx = CurrF->getContext();
1753:   // TODO: this does not retain elementtype info for memory constraints, which
1754:   //       in turn means that we lower them into pointers to i8, rather than
1755:   //       pointers to elementtype; this can be fixed during reverse translation
1756:   //       but we should correct it here, possibly by tweaking the function
1757:   //       type to take TypedPointerType args.
1758:   Constant *TyC = UndefValue::get(SPIRV::getOriginalFunctionType(Call));
1759:   MDString *ConstraintString =
1760:       MDString::get(Ctx, SPIRV::getOriginalAsmConstraints(Call));
1761:   SmallVector<Value *> Args = {
1762:       buildMD(TyC),
1763:       MetadataAsValue::get(Ctx, MDNode::get(Ctx, ConstraintString))};
1764:   for (unsigned OpIdx = 0; OpIdx < Call.arg_size(); OpIdx++)
1765:     Args.push_back(Call.getArgOperand(OpIdx));
1766:
1767:   IRBuilder<> B(Call.getParent());
1768:   B.SetInsertPoint(&Call);
1769:   B.CreateIntrinsic(Intrinsic::spv_inline_asm, {Args});
1770:   return &Call;
1771: }
1772:
1773: // Use a tip about rounding mode to create a decoration.
1774: void SPIRVEmitIntrinsics::useRoundingMode(ConstrainedFPIntrinsic *FPI,
1775:                                           IRBuilder<> &B) {
1776:   std::optional<RoundingMode> RM = FPI->getRoundingMode();
1777:   if (!RM.has_value())
1778:     return;
1779:   unsigned RoundingModeDeco = std::numeric_limits<unsigned>::max();
1780:   switch (RM.value()) {
1781:   default:
1782:     // ignore unknown rounding modes
1783:     break;
1784:   case RoundingMode::NearestTiesToEven:
1785:     RoundingModeDeco = SPIRV::FPRoundingMode::FPRoundingMode::RTE;
1786:     break;
1787:   case RoundingMode::TowardNegative:
1788:     RoundingModeDeco = SPIRV::FPRoundingMode::FPRoundingMode::RTN;
1789:     break;
1790:   case RoundingMode::TowardPositive:
1791:     RoundingModeDeco = SPIRV::FPRoundingMode::FPRoundingMode::RTP;
1792:     break;
1793:   case RoundingMode::TowardZero:
1794:     RoundingModeDeco = SPIRV::FPRoundingMode::FPRoundingMode::RTZ;
1795:     break;
1796:   case RoundingMode::Dynamic:
1797:   case RoundingMode::NearestTiesToAway:
1798:     // TODO: check if supported
1799:     break;
1800:   }
```
- EN: This range implements operational logic in helpers such as getNumElements, push_back, getOperand, ConstantPointerNull::get, translating backend policy into executable code.
- CN: 这一段实现了 getNumElements、push_back、getOperand、ConstantPointerNull::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1801-1920
```cpp
1801:   if (RoundingModeDeco == std::numeric_limits<unsigned>::max())
1802:     return;
1803:   // Convert the tip about rounding mode into a decoration record.
1804:   createRoundingModeDecoration(FPI, RoundingModeDeco, B);
1805: }
1806:
1807: Instruction *SPIRVEmitIntrinsics::visitSwitchInst(SwitchInst &I) {
1808:   BasicBlock *ParentBB = I.getParent();
1809:   Function *F = ParentBB->getParent();
1810:   IRBuilder<> B(ParentBB);
1811:   B.SetInsertPoint(&I);
1812:   SmallVector<Value *, 4> Args;
1813:   SmallVector<BasicBlock *> BBCases;
1814:   Args.push_back(I.getCondition());
1815:   BBCases.push_back(I.getDefaultDest());
1816:   Args.push_back(BlockAddress::get(F, I.getDefaultDest()));
1817:   for (auto &Case : I.cases()) {
1818:     Args.push_back(Case.getCaseValue());
1819:     BBCases.push_back(Case.getCaseSuccessor());
1820:     Args.push_back(BlockAddress::get(F, Case.getCaseSuccessor()));
1821:   }
1822:   CallInst *NewI = B.CreateIntrinsic(Intrinsic::spv_switch,
1823:                                      {I.getOperand(0)->getType()}, {Args});
1824:   // remove switch to avoid its unneeded and undesirable unwrap into branches
1825:   // and conditions
1826:   replaceAllUsesWith(&I, NewI);
1827:   I.eraseFromParent();
1828:   // insert artificial and temporary instruction to preserve valid CFG,
1829:   // it will be removed after IR translation pass
1830:   B.SetInsertPoint(ParentBB);
1831:   IndirectBrInst *BrI = B.CreateIndirectBr(
1832:       Constant::getNullValue(PointerType::getUnqual(ParentBB->getContext())),
1833:       BBCases.size());
1834:   for (BasicBlock *BBCase : BBCases)
1835:     BrI->addDestination(BBCase);
1836:   return BrI;
1837: }
1838:
1839: static bool isFirstIndexZero(const GetElementPtrInst *GEP) {
1840:   return GEP->getNumIndices() > 0 && match(GEP->getOperand(1), m_Zero());
1841: }
1842:
1843: Instruction *SPIRVEmitIntrinsics::visitIntrinsicInst(IntrinsicInst &I) {
1844:   auto *SGEP = dyn_cast<StructuredGEPInst>(&I);
1845:   if (!SGEP)
1846:     return &I;
1847:
1848:   IRBuilder<> B(I.getParent());
1849:   B.SetInsertPoint(&I);
1850:   SmallVector<Type *, 2> Types = {I.getType(), I.getOperand(0)->getType()};
1851:   SmallVector<Value *, 4> Args;
1852:   Args.push_back(/* inBounds= */ B.getInt1(true));
1853:   Args.push_back(I.getOperand(0));
1854:   Args.push_back(/* zero index */ B.getInt32(0));
1855:   for (unsigned J = 0; J < SGEP->getNumIndices(); ++J)
1856:     Args.push_back(SGEP->getIndexOperand(J));
1857:
1858:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_gep, Types, Args);
1859:   replaceAllUsesWithAndErase(B, &I, NewI);
1860:   return NewI;
1861: }
1862:
1863: Instruction *SPIRVEmitIntrinsics::visitGetElementPtrInst(GetElementPtrInst &I) {
1864:   IRBuilder<> B(I.getParent());
1865:   B.SetInsertPoint(&I);
1866:
1867:   if (TM.getSubtargetImpl()->isLogicalSPIRV() && !isFirstIndexZero(&I)) {
1868:     // Logical SPIR-V cannot use the OpPtrAccessChain instruction. If the first
1869:     // index of the GEP is not 0, then we need to try to adjust it.
1870:     //
1871:     // If the GEP is doing byte addressing, try to rebuild the full access chain
1872:     // from the type of the pointer.
1873:     if (getByteAddressingMultiplier(I.getSourceElementType())) {
1874:       return buildLogicalAccessChainFromGEP(I);
1875:     }
1876:
1877:     // Look for the array-to-pointer decay. If this is the pattern
1878:     // we can adjust the types, and prepend a 0 to the indices.
1879:     Value *PtrOp = I.getPointerOperand();
1880:     Type *SrcElemTy = I.getSourceElementType();
1881:     Type *DeducedPointeeTy = deduceElementType(PtrOp, true);
1882:
1883:     if (auto *ArrTy = dyn_cast<ArrayType>(DeducedPointeeTy)) {
1884:       if (ArrTy->getElementType() == SrcElemTy) {
1885:         SmallVector<Value *> NewIndices;
1886:         Type *FirstIdxType = I.getOperand(1)->getType();
1887:         NewIndices.push_back(ConstantInt::get(FirstIdxType, 0));
1888:         for (Value *Idx : I.indices())
1889:           NewIndices.push_back(Idx);
1890:
1891:         SmallVector<Type *, 2> Types = {I.getType(), I.getPointerOperandType()};
1892:         SmallVector<Value *, 4> Args;
1893:         Args.push_back(B.getInt1(I.isInBounds()));
1894:         Args.push_back(I.getPointerOperand());
1895:         Args.append(NewIndices.begin(), NewIndices.end());
1896:
1897:         auto *NewI = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
1898:         replaceAllUsesWithAndErase(B, &I, NewI);
1899:         return NewI;
1900:       }
1901:     }
1902:   }
1903:
1904:   SmallVector<Type *, 2> Types = {I.getType(), I.getOperand(0)->getType()};
1905:   SmallVector<Value *, 4> Args;
1906:   Args.push_back(B.getInt1(I.isInBounds()));
1907:   llvm::append_range(Args, I.operands());
1908:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_gep, {Types}, {Args});
1909:   replaceAllUsesWithAndErase(B, &I, NewI);
1910:   return NewI;
1911: }
1912:
1913: Instruction *SPIRVEmitIntrinsics::visitBitCastInst(BitCastInst &I) {
1914:   IRBuilder<> B(I.getParent());
1915:   B.SetInsertPoint(&I);
1916:   Value *Source = I.getOperand(0);
1917:
1918:   // SPIR-V, contrary to LLVM 17+ IR, supports bitcasts between pointers of
1919:   // varying element types. In case of IR coming from older versions of LLVM
1920:   // such bitcasts do not provide sufficient information, should be just skipped
```
- EN: This range implements operational logic in helpers such as createRoundingModeDecoration, SPIRVEmitIntrinsics::visitSwitchInst, getParent, B, translating backend policy into executable code.
- CN: 这一段实现了 createRoundingModeDecoration、SPIRVEmitIntrinsics::visitSwitchInst、getParent、B 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1921-2040
```cpp
1921:   // here, and handled in insertPtrCastOrAssignTypeInstr.
1922:   if (isPointerTy(I.getType())) {
1923:     replaceAllUsesWith(&I, Source);
1924:     I.eraseFromParent();
1925:     return nullptr;
1926:   }
1927:
1928:   SmallVector<Type *, 2> Types = {I.getType(), Source->getType()};
1929:   SmallVector<Value *> Args(I.op_begin(), I.op_end());
1930:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_bitcast, {Types}, {Args});
1931:   replaceAllUsesWithAndErase(B, &I, NewI);
1932:   return NewI;
1933: }
1934:
1935: void SPIRVEmitIntrinsics::insertAssignPtrTypeTargetExt(
1936:     TargetExtType *AssignedType, Value *V, IRBuilder<> &B) {
1937:   Type *VTy = V->getType();
1938:
1939:   // A couple of sanity checks.
1940:   assert((isPointerTy(VTy)) && "Expect a pointer type!");
1941:   if (Type *ElemTy = getPointeeType(VTy))
1942:     if (ElemTy != AssignedType)
1943:       report_fatal_error("Unexpected pointer element type!");
1944:
1945:   CallInst *AssignCI = GR->findAssignPtrTypeInstr(V);
1946:   if (!AssignCI) {
1947:     GR->buildAssignType(B, AssignedType, V);
1948:     return;
1949:   }
1950:
1951:   Type *CurrentType =
1952:       dyn_cast<ConstantAsMetadata>(
1953:           cast<MetadataAsValue>(AssignCI->getOperand(1))->getMetadata())
1954:           ->getType();
1955:   if (CurrentType == AssignedType)
1956:     return;
1957:
1958:   // Builtin types cannot be redeclared or casted.
1959:   if (CurrentType->isTargetExtTy())
1960:     report_fatal_error("Type mismatch " + CurrentType->getTargetExtName() +
1961:                            "/" + AssignedType->getTargetExtName() +
1962:                            " for value " + V->getName(),
1963:                        false);
1964:
1965:   // Our previous guess about the type seems to be wrong, let's update
1966:   // inferred type according to a new, more precise type information.
1967:   GR->updateAssignType(AssignCI, V, getNormalizedPoisonValue(AssignedType));
1968: }
1969:
1970: void SPIRVEmitIntrinsics::replacePointerOperandWithPtrCast(
1971:     Instruction *I, Value *Pointer, Type *ExpectedElementType,
1972:     unsigned OperandToReplace, IRBuilder<> &B) {
1973:   TypeValidated.insert(I);
1974:
1975:   // Do not emit spv_ptrcast if Pointer's element type is ExpectedElementType
1976:   Type *PointerElemTy = deduceElementTypeHelper(Pointer, false);
1977:   if (PointerElemTy == ExpectedElementType ||
1978:       isEquivalentTypes(PointerElemTy, ExpectedElementType))
1979:     return;
1980:
1981:   setInsertPointSkippingPhis(B, I);
1982:   Value *ExpectedElementVal = getNormalizedPoisonValue(ExpectedElementType);
1983:   MetadataAsValue *VMD = buildMD(ExpectedElementVal);
1984:   unsigned AddressSpace = getPointerAddressSpace(Pointer->getType());
1985:   bool FirstPtrCastOrAssignPtrType = true;
1986:
1987:   // Do not emit new spv_ptrcast if equivalent one already exists or when
1988:   // spv_assign_ptr_type already targets this pointer with the same element
1989:   // type.
1990:   if (Pointer->hasUseList()) {
1991:     for (auto User : Pointer->users()) {
1992:       auto *II = dyn_cast<IntrinsicInst>(User);
1993:       if (!II ||
1994:           (II->getIntrinsicID() != Intrinsic::spv_assign_ptr_type &&
1995:            II->getIntrinsicID() != Intrinsic::spv_ptrcast) ||
1996:           II->getOperand(0) != Pointer)
1997:         continue;
1998:
1999:       // There is some spv_ptrcast/spv_assign_ptr_type already targeting this
2000:       // pointer.
2001:       FirstPtrCastOrAssignPtrType = false;
2002:       if (II->getOperand(1) != VMD ||
2003:           dyn_cast<ConstantInt>(II->getOperand(2))->getSExtValue() !=
2004:               AddressSpace)
2005:         continue;
2006:
2007:       // The spv_ptrcast/spv_assign_ptr_type targeting this pointer is of the
2008:       // same element type and address space.
2009:       if (II->getIntrinsicID() != Intrinsic::spv_ptrcast)
2010:         return;
2011:
2012:       // This must be a spv_ptrcast, do not emit new if this one has the same BB
2013:       // as I. Otherwise, search for other spv_ptrcast/spv_assign_ptr_type.
2014:       if (II->getParent() != I->getParent())
2015:         continue;
2016:
2017:       I->setOperand(OperandToReplace, II);
2018:       return;
2019:     }
2020:   }
2021:
2022:   if (isa<Instruction>(Pointer) || isa<Argument>(Pointer)) {
2023:     if (FirstPtrCastOrAssignPtrType) {
2024:       // If this would be the first spv_ptrcast, do not emit spv_ptrcast and
2025:       // emit spv_assign_ptr_type instead.
2026:       GR->buildAssignPtr(B, ExpectedElementType, Pointer);
2027:       return;
2028:     } else if (isTodoType(Pointer)) {
2029:       eraseTodoType(Pointer);
2030:       if (!isa<CallInst>(Pointer) && !isaGEP(Pointer) &&
2031:           !isa<AllocaInst>(Pointer)) {
2032:         //  If this wouldn't be the first spv_ptrcast but existing type info is
2033:         //  uncomplete, update spv_assign_ptr_type arguments.
2034:         if (CallInst *AssignCI = GR->findAssignPtrTypeInstr(Pointer)) {
2035:           Type *PrevElemTy = GR->findDeducedElementType(Pointer);
2036:           assert(PrevElemTy);
2037:           DenseSet<std::pair<Value *, Value *>> VisitedSubst{
2038:               std::make_pair(I, Pointer)};
2039:           GR->updateAssignType(AssignCI, Pointer, ExpectedElementVal);
2040:           propagateElemType(Pointer, PrevElemTy, VisitedSubst);
```
- EN: This range implements operational logic in helpers such as replaceAllUsesWith, eraseFromParent, Args, CreateIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 replaceAllUsesWith、eraseFromParent、Args、CreateIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2041-2160
```cpp
2041:         } else {
2042:           GR->buildAssignPtr(B, ExpectedElementType, Pointer);
2043:         }
2044:         return;
2045:       }
2046:     }
2047:   }
2048:
2049:   // Emit spv_ptrcast
2050:   SmallVector<Type *, 2> Types = {Pointer->getType(), Pointer->getType()};
2051:   SmallVector<Value *, 2> Args = {Pointer, VMD, B.getInt32(AddressSpace)};
2052:   auto *PtrCastI = B.CreateIntrinsic(Intrinsic::spv_ptrcast, {Types}, Args);
2053:   I->setOperand(OperandToReplace, PtrCastI);
2054:   // We need to set up a pointee type for the newly created spv_ptrcast.
2055:   GR->buildAssignPtr(B, ExpectedElementType, PtrCastI);
2056: }
2057:
2058: void SPIRVEmitIntrinsics::insertPtrCastOrAssignTypeInstr(Instruction *I,
2059:                                                          IRBuilder<> &B) {
2060:   // Handle basic instructions:
2061:   StoreInst *SI = dyn_cast<StoreInst>(I);
2062:   if (IsKernelArgInt8(CurrF, SI)) {
2063:     replacePointerOperandWithPtrCast(
2064:         I, SI->getValueOperand(), IntegerType::getInt8Ty(CurrF->getContext()),
2065:         0, B);
2066:   }
2067:   if (SI) {
2068:     Value *Op = SI->getValueOperand();
2069:     Value *Pointer = SI->getPointerOperand();
2070:     Type *OpTy = Op->getType();
2071:     if (auto *OpI = dyn_cast<Instruction>(Op))
2072:       OpTy = restoreMutatedType(GR, OpI, OpTy);
2073:     // Aggregate undef/constant values were lowered by preprocessUndefs/
2074:     // preprocessCompositeConstants to spv_undef/spv_const_composite calls
2075:     // returning i32; recover the original aggregate type so the pointee type
2076:     // assigned to the store's pointer operand reflects the value being stored.
2077:     if (match(Op, m_CombineOr(m_Intrinsic<Intrinsic::spv_undef>(),
2078:                               m_Intrinsic<Intrinsic::spv_const_composite>())))
2079:       if (auto It = AggrConstTypes.find(cast<Instruction>(Op));
2080:           It != AggrConstTypes.end())
2081:         OpTy = It->second;
2082:     if (OpTy == Op->getType())
2083:       OpTy = deduceElementTypeByValueDeep(OpTy, Op, false);
2084:     replacePointerOperandWithPtrCast(I, Pointer, OpTy, 1, B);
2085:     return;
2086:   }
2087:   if (LoadInst *LI = dyn_cast<LoadInst>(I)) {
2088:     Value *Pointer = LI->getPointerOperand();
2089:     Type *OpTy = LI->getType();
2090:     if (auto *PtrTy = dyn_cast<PointerType>(OpTy)) {
2091:       if (Type *ElemTy = GR->findDeducedElementType(LI)) {
2092:         OpTy = getTypedPointerWrapper(ElemTy, PtrTy->getAddressSpace());
2093:       } else {
2094:         Type *NewOpTy = OpTy;
2095:         OpTy = deduceElementTypeByValueDeep(OpTy, LI, false);
2096:         if (OpTy == NewOpTy)
2097:           insertTodoType(Pointer);
2098:       }
2099:     }
2100:     replacePointerOperandWithPtrCast(I, Pointer, OpTy, 0, B);
2101:     return;
2102:   }
2103:   if (GetElementPtrInst *GEPI = dyn_cast<GetElementPtrInst>(I)) {
2104:     Value *Pointer = GEPI->getPointerOperand();
2105:     Type *OpTy = nullptr;
2106:
2107:     // Logical SPIR-V is not allowed to use Op*PtrAccessChain instructions. If
2108:     // the first index is 0, then we can trivially lower to OpAccessChain. If
2109:     // not we need to try to rewrite the GEP. We avoid adding a pointer cast at
2110:     // this time, and will rewrite the GEP when visiting it.
2111:     if (TM.getSubtargetImpl()->isLogicalSPIRV() && !isFirstIndexZero(GEPI)) {
2112:       return;
2113:     }
2114:
2115:     // In all cases, fall back to the GEP type if type scavenging failed.
2116:     if (!OpTy)
2117:       OpTy = GEPI->getSourceElementType();
2118:
2119:     replacePointerOperandWithPtrCast(I, Pointer, OpTy, 0, B);
2120:     if (isNestedPointer(OpTy))
2121:       insertTodoType(Pointer);
2122:     return;
2123:   }
2124:
2125:   // TODO: review and merge with existing logics:
2126:   // Handle calls to builtins (non-intrinsics):
2127:   CallInst *CI = dyn_cast<CallInst>(I);
2128:   if (!CI || CI->isIndirectCall() || CI->isInlineAsm() ||
2129:       !CI->getCalledFunction() || CI->getCalledFunction()->isIntrinsic())
2130:     return;
2131:
2132:   // collect information about formal parameter types
2133:   std::string DemangledName =
2134:       getOclOrSpirvBuiltinDemangledName(CI->getCalledFunction()->getName());
2135:   Function *CalledF = CI->getCalledFunction();
2136:   SmallVector<Type *, 4> CalledArgTys;
2137:   bool HaveTypes = false;
2138:   for (unsigned OpIdx = 0; OpIdx < CalledF->arg_size(); ++OpIdx) {
2139:     Argument *CalledArg = CalledF->getArg(OpIdx);
2140:     Type *ArgType = CalledArg->getType();
2141:     if (!isPointerTy(ArgType)) {
2142:       CalledArgTys.push_back(nullptr);
2143:     } else if (Type *ArgTypeElem = getPointeeType(ArgType)) {
2144:       CalledArgTys.push_back(ArgTypeElem);
2145:       HaveTypes = true;
2146:     } else {
2147:       Type *ElemTy = GR->findDeducedElementType(CalledArg);
2148:       if (!ElemTy && hasPointeeTypeAttr(CalledArg))
2149:         ElemTy = getPointeeTypeByAttr(CalledArg);
2150:       if (!ElemTy) {
2151:         ElemTy = getPointeeTypeByCallInst(DemangledName, CalledF, OpIdx);
2152:         if (ElemTy) {
2153:           GR->addDeducedElementType(CalledArg, normalizeType(ElemTy));
2154:         } else {
2155:           for (User *U : CalledArg->users()) {
2156:             if (Instruction *Inst = dyn_cast<Instruction>(U)) {
2157:               if ((ElemTy = deduceElementTypeHelper(Inst, false)) != nullptr)
2158:                 break;
2159:             }
2160:           }
```
- EN: This range implements operational logic in helpers such as buildAssignPtr, CreateIntrinsic, setOperand, getValueOperand, translating backend policy into executable code.
- CN: 这一段实现了 buildAssignPtr、CreateIntrinsic、setOperand、getValueOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2161-2280
```cpp
2161:         }
2162:       }
2163:       HaveTypes |= ElemTy != nullptr;
2164:       CalledArgTys.push_back(ElemTy);
2165:     }
2166:   }
2167:
2168:   if (DemangledName.empty() && !HaveTypes)
2169:     return;
2170:
2171:   for (unsigned OpIdx = 0; OpIdx < CI->arg_size(); OpIdx++) {
2172:     Value *ArgOperand = CI->getArgOperand(OpIdx);
2173:     if (!isPointerTy(ArgOperand->getType()))
2174:       continue;
2175:
2176:     // Constants (nulls/undefs) are handled in insertAssignPtrTypeIntrs()
2177:     if (!isa<Instruction>(ArgOperand) && !isa<Argument>(ArgOperand)) {
2178:       // However, we may have assumptions about the formal argument's type and
2179:       // may have a need to insert a ptr cast for the actual parameter of this
2180:       // call.
2181:       Argument *CalledArg = CalledF->getArg(OpIdx);
2182:       if (!GR->findDeducedElementType(CalledArg))
2183:         continue;
2184:     }
2185:
2186:     Type *ExpectedType =
2187:         OpIdx < CalledArgTys.size() ? CalledArgTys[OpIdx] : nullptr;
2188:     if (!ExpectedType && !DemangledName.empty())
2189:       ExpectedType = SPIRV::parseBuiltinCallArgumentBaseType(
2190:           DemangledName, OpIdx, I->getContext());
2191:     if (!ExpectedType || ExpectedType->isVoidTy())
2192:       continue;
2193:
2194:     if (ExpectedType->isTargetExtTy() &&
2195:         !isTypedPointerWrapper(cast<TargetExtType>(ExpectedType)))
2196:       insertAssignPtrTypeTargetExt(cast<TargetExtType>(ExpectedType),
2197:                                    ArgOperand, B);
2198:     else
2199:       replacePointerOperandWithPtrCast(CI, ArgOperand, ExpectedType, OpIdx, B);
2200:   }
2201: }
2202:
2203: Instruction *SPIRVEmitIntrinsics::visitInsertElementInst(InsertElementInst &I) {
2204:   // If it's a <1 x Type> vector type, don't modify it. It's not a legal vector
2205:   // type in LLT and IRTranslator will replace it by the scalar.
2206:   if (isVector1(I.getType()))
2207:     return &I;
2208:
2209:   SmallVector<Type *, 4> Types = {I.getType(), I.getOperand(0)->getType(),
2210:                                   I.getOperand(1)->getType(),
2211:                                   I.getOperand(2)->getType()};
2212:   IRBuilder<> B(I.getParent());
2213:   B.SetInsertPoint(&I);
2214:   SmallVector<Value *> Args(I.op_begin(), I.op_end());
2215:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_insertelt, {Types}, {Args});
2216:   replaceAllUsesWithAndErase(B, &I, NewI);
2217:   return NewI;
2218: }
2219:
2220: Instruction *
2221: SPIRVEmitIntrinsics::visitExtractElementInst(ExtractElementInst &I) {
2222:   // If it's a <1 x Type> vector type, don't modify it. It's not a legal vector
2223:   // type in LLT and IRTranslator will replace it by the scalar.
2224:   if (isVector1(I.getVectorOperandType()))
2225:     return &I;
2226:
2227:   IRBuilder<> B(I.getParent());
2228:   B.SetInsertPoint(&I);
2229:   SmallVector<Type *, 3> Types = {I.getType(), I.getVectorOperandType(),
2230:                                   I.getIndexOperand()->getType()};
2231:   SmallVector<Value *, 2> Args = {I.getVectorOperand(), I.getIndexOperand()};
2232:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_extractelt, {Types}, {Args});
2233:   replaceAllUsesWithAndErase(B, &I, NewI);
2234:   return NewI;
2235: }
2236:
2237: Instruction *SPIRVEmitIntrinsics::visitInsertValueInst(InsertValueInst &I) {
2238:   IRBuilder<> B(I.getParent());
2239:   B.SetInsertPoint(&I);
2240:   SmallVector<Type *, 1> Types = {I.getInsertedValueOperand()->getType()};
2241:   SmallVector<Value *> Args;
2242:   Value *AggregateOp = I.getAggregateOperand();
2243:   if (isa<UndefValue>(AggregateOp))
2244:     Args.push_back(UndefValue::get(B.getInt32Ty()));
2245:   else
2246:     Args.push_back(AggregateOp);
2247:   Args.push_back(I.getInsertedValueOperand());
2248:   for (auto &Op : I.indices())
2249:     Args.push_back(B.getInt32(Op));
2250:   Instruction *NewI =
2251:       B.CreateIntrinsic(Intrinsic::spv_insertv, {Types}, {Args});
2252:   replaceMemInstrUses(&I, NewI, B);
2253:   return NewI;
2254: }
2255:
2256: Instruction *SPIRVEmitIntrinsics::visitExtractValueInst(ExtractValueInst &I) {
2257:   if (I.getAggregateOperand()->getType()->isAggregateType())
2258:     return &I;
2259:   IRBuilder<> B(I.getParent());
2260:   B.SetInsertPoint(&I);
2261:   SmallVector<Value *> Args(I.operands());
2262:   for (auto &Op : I.indices())
2263:     Args.push_back(B.getInt32(Op));
2264:   auto *NewI =
2265:       B.CreateIntrinsic(Intrinsic::spv_extractv, {I.getType()}, {Args});
2266:   replaceAllUsesWithAndErase(B, &I, NewI);
2267:   return NewI;
2268: }
2269:
2270: Instruction *SPIRVEmitIntrinsics::visitLoadInst(LoadInst &I) {
2271:   if (!I.getType()->isAggregateType())
2272:     return &I;
2273:   IRBuilder<> B(I.getParent());
2274:   B.SetInsertPoint(&I);
2275:   TrackConstants = false;
2276:   const auto *TLI = TM.getSubtargetImpl()->getTargetLowering();
2277:   MachineMemOperand::Flags Flags =
2278:       TLI->getLoadMemOperandFlags(I, CurrF->getDataLayout());
2279:
2280:   unsigned IntrinsicId;
```
- EN: This range implements operational logic in helpers such as push_back, arg_size, getArgOperand, getArg, translating backend policy into executable code.
- CN: 这一段实现了 push_back、arg_size、getArgOperand、getArg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2281-2400
```cpp
2281:   SmallVector<Value *, 4> Args = {I.getPointerOperand(), B.getInt16(Flags)};
2282:   if (!I.isAtomic()) {
2283:     IntrinsicId = Intrinsic::spv_load;
2284:     Args.push_back(B.getInt32(I.getAlign().value()));
2285:   } else {
2286:     IntrinsicId = Intrinsic::spv_atomic_load;
2287:     Args.push_back(B.getInt8(static_cast<uint8_t>(I.getOrdering())));
2288:   }
2289:   CallInst *NewI =
2290:       B.CreateIntrinsic(IntrinsicId, {I.getOperand(0)->getType()}, Args);
2291:
2292:   replaceMemInstrUses(&I, NewI, B);
2293:   return NewI;
2294: }
2295:
2296: Instruction *SPIRVEmitIntrinsics::visitStoreInst(StoreInst &I) {
2297:   if (!AggrStores.contains(&I))
2298:     return &I;
2299:   IRBuilder<> B(I.getParent());
2300:   B.SetInsertPoint(&I);
2301:   TrackConstants = false;
2302:   const auto *TLI = TM.getSubtargetImpl()->getTargetLowering();
2303:   MachineMemOperand::Flags Flags =
2304:       TLI->getStoreMemOperandFlags(I, CurrF->getDataLayout());
2305:   auto *PtrOp = I.getPointerOperand();
2306:
2307:   if (I.getValueOperand()->getType()->isAggregateType()) {
2308:     // It is possible that what used to be an ExtractValueInst has been replaced
2309:     // with a call to the spv_extractv intrinsic, and that said call hasn't
2310:     // had its return type replaced with i32 during the dedicated pass (because
2311:     // it was emitted later); we have to handle this here, because IRTranslator
2312:     // cannot deal with multi-register types at the moment.
2313:     CallBase *CB = dyn_cast<CallBase>(I.getValueOperand());
2314:     assert(CB && CB->getIntrinsicID() == Intrinsic::spv_extractv &&
2315:            "Unexpected argument of aggregate type, should be spv_extractv!");
2316:     CB->mutateType(B.getInt32Ty());
2317:   }
2318:
2319:   unsigned IntrinsicId;
2320:   SmallVector<Value *, 4> Args = {I.getValueOperand(), PtrOp,
2321:                                   B.getInt16(Flags)};
2322:   if (!I.isAtomic()) {
2323:     IntrinsicId = Intrinsic::spv_store;
2324:     Args.push_back(B.getInt32(I.getAlign().value()));
2325:   } else {
2326:     IntrinsicId = Intrinsic::spv_atomic_store;
2327:     Args.push_back(B.getInt8(static_cast<uint8_t>(I.getOrdering())));
2328:   }
2329:   auto *NewI = B.CreateIntrinsic(
2330:       IntrinsicId, {I.getValueOperand()->getType(), PtrOp->getType()}, Args);
2331:   NewI->copyMetadata(I);
2332:   I.eraseFromParent();
2333:   return NewI;
2334: }
2335:
2336: Instruction *SPIRVEmitIntrinsics::visitAllocaInst(AllocaInst &I) {
2337:   Value *ArraySize = nullptr;
2338:   if (I.isArrayAllocation()) {
2339:     const SPIRVSubtarget *STI = TM.getSubtargetImpl(*I.getFunction());
2340:     if (!STI->canUseExtension(
2341:             SPIRV::Extension::SPV_INTEL_variable_length_array))
2342:       report_fatal_error(
2343:           "array allocation: this instruction requires the following "
2344:           "SPIR-V extension: SPV_INTEL_variable_length_array",
2345:           false);
2346:     ArraySize = I.getArraySize();
2347:   }
2348:   IRBuilder<> B(I.getParent());
2349:   B.SetInsertPoint(&I);
2350:   TrackConstants = false;
2351:   Type *PtrTy = I.getType();
2352:   auto *NewI =
2353:       ArraySize
2354:           ? B.CreateIntrinsic(Intrinsic::spv_alloca_array,
2355:                               {PtrTy, ArraySize->getType()},
2356:                               {ArraySize, B.getInt32(I.getAlign().value())})
2357:           : B.CreateIntrinsic(Intrinsic::spv_alloca, {PtrTy},
2358:                               {B.getInt32(I.getAlign().value())});
2359:   replaceAllUsesWithAndErase(B, &I, NewI);
2360:   return NewI;
2361: }
2362:
2363: Instruction *SPIRVEmitIntrinsics::visitAtomicCmpXchgInst(AtomicCmpXchgInst &I) {
2364:   assert(I.getType()->isAggregateType() && "Aggregate result is expected");
2365:   IRBuilder<> B(I.getParent());
2366:   B.SetInsertPoint(&I);
2367:   SmallVector<Value *> Args(I.operands());
2368:   Args.push_back(B.getInt32(
2369:       static_cast<uint32_t>(getMemScope(I.getContext(), I.getSyncScopeID()))));
2370:   // Per SPIR-V spec atomic ops must combine the ordering bits with the
2371:   // storage-class bit.
2372:   const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(*I.getFunction());
2373:   unsigned AS = I.getPointerOperand()->getType()->getPointerAddressSpace();
2374:   uint32_t ScSem = static_cast<uint32_t>(
2375:       getMemSemanticsForStorageClass(addressSpaceToStorageClass(AS, ST)));
2376:   Args.push_back(B.getInt32(
2377:       static_cast<uint32_t>(getMemSemantics(I.getSuccessOrdering())) | ScSem));
2378:   Args.push_back(B.getInt32(
2379:       static_cast<uint32_t>(getMemSemantics(I.getFailureOrdering())) | ScSem));
2380:   auto *NewI = B.CreateIntrinsic(Intrinsic::spv_cmpxchg,
2381:                                  {I.getPointerOperand()->getType()}, {Args});
2382:   replaceMemInstrUses(&I, NewI, B);
2383:   return NewI;
2384: }
2385:
2386: static bool isAbortCall(const Instruction &I, const SPIRVSubtarget &ST) {
2387:   auto *CI = dyn_cast<CallInst>(&I);
2388:   if (!CI)
2389:     return false;
2390:   switch (CI->getIntrinsicID()) {
2391:   case Intrinsic::spv_abort:
2392:     return true;
2393:   case Intrinsic::trap:
2394:   case Intrinsic::ubsantrap:
2395:     // When the extension is enabled, selection lowers these to OpAbortKHR.
2396:     return ST.canUseExtension(SPIRV::Extension::SPV_KHR_abort);
2397:   default:
2398:     return false;
2399:   }
2400: }
```
- EN: This range implements operational logic in helpers such as push_back, CreateIntrinsic, replaceMemInstrUses, SPIRVEmitIntrinsics::visitStoreInst, translating backend policy into executable code.
- CN: 这一段实现了 push_back、CreateIntrinsic、replaceMemInstrUses、SPIRVEmitIntrinsics::visitStoreInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2401-2520
```cpp
2401:
2402: // The OpAbortKHR instruction itself is a block terminator, so we don't need to
2403: // emit an extra OpUnreachable instruction.
2404: static bool precededByAbortIntrinsic(const UnreachableInst &I,
2405:                                      const SPIRVSubtarget &ST) {
2406:   // Find a previous non-debug instruction.
2407:   const Instruction *Prev = I.getPrevNode();
2408:   while (Prev && Prev->isDebugOrPseudoInst())
2409:     Prev = Prev->getPrevNode();
2410:
2411:   if (Prev && isAbortCall(*Prev, ST))
2412:     return true;
2413:
2414:   assert(llvm::none_of(
2415:              *I.getParent(),
2416:              [&ST](const Instruction &II) { return isAbortCall(II, ST); }) &&
2417:          "abort-like call must be the last non-debug instruction before its "
2418:          "block's terminator");
2419:   return false;
2420: }
2421:
2422: Instruction *SPIRVEmitIntrinsics::visitUnreachableInst(UnreachableInst &I) {
2423:   const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(*I.getFunction());
2424:   if (precededByAbortIntrinsic(I, ST))
2425:     return &I;
2426:   IRBuilder<> B(&I);
2427:   B.CreateIntrinsic(Intrinsic::spv_unreachable, {});
2428:   return &I;
2429: }
2430:
2431: static bool
2432: shouldEmitIntrinsicsForGlobalValue(const GlobalVariableUsers &GVUsers,
2433:                                    const GlobalVariable &GV,
2434:                                    const Function *F) {
2435:   // Skip special artificial variables.
2436:   static const StringSet<> ArtificialGlobals{"llvm.global.annotations",
2437:                                              "llvm.compiler.used", "llvm.used"};
2438:
2439:   if (ArtificialGlobals.contains(GV.getName()))
2440:     return false;
2441:
2442:   auto &UserFunctions = GVUsers.getTransitiveUserFunctions(GV);
2443:   if (UserFunctions.contains(F))
2444:     return true;
2445:
2446:   // Do not emit the intrinsics in this function, it's going to be emitted on
2447:   // the functions that reference it.
2448:   if (!UserFunctions.empty())
2449:     return false;
2450:
2451:   // Emit definitions for globals that are not referenced by any function on the
2452:   // first function definition.
2453:   const Module &M = *F->getParent();
2454:   const Function &FirstDefinition = *M.getFunctionDefs().begin();
2455:   return F == &FirstDefinition;
2456: }
2457:
2458: Value *SPIRVEmitIntrinsics::buildSpvUndefComposite(Type *AggrTy,
2459:                                                    IRBuilder<> &B) {
2460:   SmallVector<Value *, 4> Elems;
2461:   if (auto *ArrTy = dyn_cast<ArrayType>(AggrTy)) {
2462:     Type *ElemTy = ArrTy->getElementType();
2463:     auto *UI = B.CreateIntrinsic(Intrinsic::spv_undef, {});
2464:     AggrConsts[UI] = PoisonValue::get(ElemTy);
2465:     AggrConstTypes[UI] = ElemTy;
2466:     Elems.assign(ArrTy->getNumElements(), UI);
2467:   } else {
2468:     auto *StructTy = cast<StructType>(AggrTy);
2469:     DenseMap<Type *, Instruction *> UndefByType;
2470:     for (unsigned I = 0; I < StructTy->getNumElements(); ++I) {
2471:       Type *ElemTy = StructTy->getContainedType(I);
2472:       auto &Entry = UndefByType[ElemTy];
2473:       if (!Entry) {
2474:         Entry = B.CreateIntrinsic(Intrinsic::spv_undef, {});
2475:         AggrConsts[Entry] = PoisonValue::get(ElemTy);
2476:         AggrConstTypes[Entry] = ElemTy;
2477:       }
2478:       Elems.push_back(Entry);
2479:     }
2480:   }
2481:   auto *Composite = B.CreateIntrinsic(Intrinsic::spv_const_composite,
2482:                                       {B.getInt32Ty()}, Elems);
2483:   AggrConsts[Composite] = PoisonValue::get(AggrTy);
2484:   AggrConstTypes[Composite] = AggrTy;
2485:   return Composite;
2486: }
2487:
2488: void SPIRVEmitIntrinsics::processGlobalValue(GlobalVariable &GV,
2489:                                              IRBuilder<> &B) {
2490:
2491:   if (!shouldEmitIntrinsicsForGlobalValue(GVUsers, GV, CurrF))
2492:     return;
2493:
2494:   Constant *Init = nullptr;
2495:   if (hasInitializer(&GV)) {
2496:     // Deduce element type and store results in Global Registry.
2497:     // Result is ignored, because TypedPointerType is not supported
2498:     // by llvm IR general logic.
2499:     deduceElementTypeHelper(&GV, false);
2500:     Init = GV.getInitializer();
2501:     Value *InitOp = Init;
2502:     if (isa<UndefValue>(Init) && Init->getType()->isAggregateType())
2503:       InitOp = buildSpvUndefComposite(Init->getType(), B);
2504:     Type *Ty = isAggrConstForceInt32(Init) ? B.getInt32Ty() : Init->getType();
2505:     Constant *Const = isAggrConstForceInt32(Init) ? B.getInt32(1) : Init;
2506:     auto *InitInst = B.CreateIntrinsic(Intrinsic::spv_init_global,
2507:                                        {GV.getType(), Ty}, {&GV, Const});
2508:     InitInst->setArgOperand(1, InitOp);
2509:   }
2510:   if (!Init && GV.use_empty())
2511:     B.CreateIntrinsic(Intrinsic::spv_unref_global, GV.getType(), &GV);
2512: }
2513:
2514: // Return true, if we can't decide what is the pointee type now and will get
2515: // back to the question later. Return false is spv_assign_ptr_type is not needed
2516: // or can be inserted immediately.
2517: bool SPIRVEmitIntrinsics::insertAssignPtrTypeIntrs(Instruction *I,
2518:                                                    IRBuilder<> &B,
2519:                                                    bool UnknownElemTypeI8) {
2520:   reportFatalOnTokenType(I);
```
- EN: This range implements operational logic in helpers such as getPrevNode, isAbortCall, SPIRVEmitIntrinsics::visitUnreachableInst, getFunction, translating backend policy into executable code.
- CN: 这一段实现了 getPrevNode、isAbortCall、SPIRVEmitIntrinsics::visitUnreachableInst、getFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2521-2640
```cpp
2521:   if (!isPointerTy(I->getType()) || !requireAssignType(I))
2522:     return false;
2523:
2524:   setInsertPointAfterDef(B, I);
2525:   if (Type *ElemTy = deduceElementType(I, UnknownElemTypeI8)) {
2526:     GR->buildAssignPtr(B, ElemTy, I);
2527:     return false;
2528:   }
2529:   return true;
2530: }
2531:
2532: void SPIRVEmitIntrinsics::insertAssignTypeIntrs(Instruction *I,
2533:                                                 IRBuilder<> &B) {
2534:   // TODO: extend the list of functions with known result types
2535:   static StringMap<unsigned> ResTypeWellKnown = {
2536:       {"async_work_group_copy", WellKnownTypes::Event},
2537:       {"async_work_group_strided_copy", WellKnownTypes::Event},
2538:       {"__spirv_GroupAsyncCopy", WellKnownTypes::Event}};
2539:
2540:   reportFatalOnTokenType(I);
2541:
2542:   bool IsKnown = false;
2543:   if (auto *CI = dyn_cast<CallInst>(I)) {
2544:     if (!CI->isIndirectCall() && !CI->isInlineAsm() &&
2545:         CI->getCalledFunction() && !CI->getCalledFunction()->isIntrinsic()) {
2546:       Function *CalledF = CI->getCalledFunction();
2547:       std::string DemangledName =
2548:           getOclOrSpirvBuiltinDemangledName(CalledF->getName());
2549:       FPDecorationId DecorationId = FPDecorationId::NONE;
2550:       if (DemangledName.length() > 0)
2551:         DemangledName =
2552:             SPIRV::lookupBuiltinNameHelper(DemangledName, &DecorationId);
2553:       auto ResIt = ResTypeWellKnown.find(DemangledName);
2554:       if (ResIt != ResTypeWellKnown.end()) {
2555:         IsKnown = true;
2556:         setInsertPointAfterDef(B, I);
2557:         switch (ResIt->second) {
2558:         case WellKnownTypes::Event:
2559:           GR->buildAssignType(
2560:               B, TargetExtType::get(I->getContext(), "spirv.Event"), I);
2561:           break;
2562:         }
2563:       }
2564:       // check if a floating rounding mode or saturation info is present
2565:       switch (DecorationId) {
2566:       default:
2567:         break;
2568:       case FPDecorationId::SAT:
2569:         createSaturatedConversionDecoration(CI, B);
2570:         break;
2571:       case FPDecorationId::RTE:
2572:         createRoundingModeDecoration(
2573:             CI, SPIRV::FPRoundingMode::FPRoundingMode::RTE, B);
2574:         break;
2575:       case FPDecorationId::RTZ:
2576:         createRoundingModeDecoration(
2577:             CI, SPIRV::FPRoundingMode::FPRoundingMode::RTZ, B);
2578:         break;
2579:       case FPDecorationId::RTP:
2580:         createRoundingModeDecoration(
2581:             CI, SPIRV::FPRoundingMode::FPRoundingMode::RTP, B);
2582:         break;
2583:       case FPDecorationId::RTN:
2584:         createRoundingModeDecoration(
2585:             CI, SPIRV::FPRoundingMode::FPRoundingMode::RTN, B);
2586:         break;
2587:       }
2588:     }
2589:   }
2590:
2591:   Type *Ty = I->getType();
2592:   if (!IsKnown && !Ty->isVoidTy() && !isPointerTy(Ty) && requireAssignType(I)) {
2593:     setInsertPointAfterDef(B, I);
2594:     Type *TypeToAssign = Ty;
2595:     if (auto *II = dyn_cast<IntrinsicInst>(I)) {
2596:       if (II->getIntrinsicID() == Intrinsic::spv_const_composite ||
2597:           II->getIntrinsicID() == Intrinsic::spv_undef) {
2598:         auto It = AggrConstTypes.find(II);
2599:         if (It == AggrConstTypes.end())
2600:           report_fatal_error("Unknown composite intrinsic type");
2601:         TypeToAssign = It->second;
2602:       }
2603:     } else if (auto It = AggrConstTypes.find(I); It != AggrConstTypes.end())
2604:       TypeToAssign = It->second;
2605:     TypeToAssign = restoreMutatedType(GR, I, TypeToAssign);
2606:     GR->buildAssignType(B, TypeToAssign, I);
2607:   }
2608:   for (const auto &Op : I->operands()) {
2609:     if (isa<ConstantPointerNull>(Op) || isa<UndefValue>(Op) ||
2610:         // Check GetElementPtrConstantExpr case.
2611:         (isa<ConstantExpr>(Op) &&
2612:          (isa<GEPOperator>(Op) ||
2613:           (cast<ConstantExpr>(Op)->getOpcode() == CastInst::IntToPtr)))) {
2614:       setInsertPointSkippingPhis(B, I);
2615:       Type *OpTy = Op->getType();
2616:       if (isa<UndefValue>(Op) && OpTy->isAggregateType()) {
2617:         CallInst *AssignCI =
2618:             buildIntrWithMD(Intrinsic::spv_assign_type, {B.getInt32Ty()}, Op,
2619:                             UndefValue::get(B.getInt32Ty()), {}, B);
2620:         GR->addAssignPtrTypeInstr(Op, AssignCI);
2621:       } else if (!isa<Instruction>(Op)) {
2622:         Type *OpTy = Op->getType();
2623:         Type *OpTyElem = getPointeeType(OpTy);
2624:         if (OpTyElem) {
2625:           GR->buildAssignPtr(B, OpTyElem, Op);
2626:         } else if (isPointerTy(OpTy)) {
2627:           Type *ElemTy = GR->findDeducedElementType(Op);
2628:           GR->buildAssignPtr(B, ElemTy ? ElemTy : deduceElementType(Op, true),
2629:                              Op);
2630:         } else {
2631:           Value *OpTyVal = Op;
2632:           if (OpTy->isTargetExtTy()) {
2633:             // We need to do this in order to be consistent with how target ext
2634:             // types are handled in `processInstrAfterVisit`
2635:             OpTyVal = getNormalizedPoisonValue(OpTy);
2636:           }
2637:           CallInst *AssignCI =
2638:               buildIntrWithMD(Intrinsic::spv_assign_type, {OpTy},
2639:                               getNormalizedPoisonValue(OpTy), OpTyVal, {}, B);
2640:           GR->addAssignPtrTypeInstr(OpTyVal, AssignCI);
```
- EN: This range implements operational logic in helpers such as setInsertPointAfterDef, buildAssignPtr, reportFatalOnTokenType, getCalledFunction, translating backend policy into executable code.
- CN: 这一段实现了 setInsertPointAfterDef、buildAssignPtr、reportFatalOnTokenType、getCalledFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2641-2760
```cpp
2641:         }
2642:       }
2643:     }
2644:   }
2645: }
2646:
2647: bool SPIRVEmitIntrinsics::shouldTryToAddMemAliasingDecoration(
2648:     Instruction *Inst) {
2649:   const SPIRVSubtarget *STI = TM.getSubtargetImpl(*Inst->getFunction());
2650:   if (!STI->canUseExtension(SPIRV::Extension::SPV_INTEL_memory_access_aliasing))
2651:     return false;
2652:   // Add aliasing decorations to internal load and store intrinsics.
2653:   // Do not attach them to store atomic or load atomic intrinsics / instructions
2654:   // since the extension is inconsistent at the moment (we cannot add the
2655:   // decoration to atomic stores because they do not have an id).
2656:   return match(Inst,
2657:                m_AnyIntrinsic<Intrinsic::spv_load, Intrinsic::spv_store>());
2658: }
2659:
2660: void SPIRVEmitIntrinsics::insertSpirvDecorations(Instruction *I,
2661:                                                  IRBuilder<> &B) {
2662:   if (MDNode *MD = I->getMetadata("spirv.Decorations")) {
2663:     setInsertPointAfterDef(B, I);
2664:     B.CreateIntrinsic(Intrinsic::spv_assign_decoration, {I->getType()},
2665:                       {I, MetadataAsValue::get(I->getContext(), MD)});
2666:   }
2667:   // Lower alias.scope/noalias metadata
2668:   {
2669:     auto processMemAliasingDecoration = [&](unsigned Kind) {
2670:       if (MDNode *AliasListMD = I->getMetadata(Kind)) {
2671:         if (shouldTryToAddMemAliasingDecoration(I)) {
2672:           uint32_t Dec = Kind == LLVMContext::MD_alias_scope
2673:                              ? SPIRV::Decoration::AliasScopeINTEL
2674:                              : SPIRV::Decoration::NoAliasINTEL;
2675:           SmallVector<Value *, 3> Args = {
2676:               I, ConstantInt::get(B.getInt32Ty(), Dec),
2677:               MetadataAsValue::get(I->getContext(), AliasListMD)};
2678:           setInsertPointAfterDef(B, I);
2679:           B.CreateIntrinsic(Intrinsic::spv_assign_aliasing_decoration,
2680:                             {I->getType()}, {Args});
2681:         }
2682:       }
2683:     };
2684:     processMemAliasingDecoration(LLVMContext::MD_alias_scope);
2685:     processMemAliasingDecoration(LLVMContext::MD_noalias);
2686:   }
2687:   // MD_fpmath
2688:   if (MDNode *MD = I->getMetadata(LLVMContext::MD_fpmath)) {
2689:     const SPIRVSubtarget *STI = TM.getSubtargetImpl(*I->getFunction());
2690:     bool AllowFPMaxError =
2691:         STI->canUseExtension(SPIRV::Extension::SPV_INTEL_fp_max_error);
2692:     if (!AllowFPMaxError)
2693:       return;
2694:
2695:     setInsertPointAfterDef(B, I);
2696:     B.CreateIntrinsic(Intrinsic::spv_assign_fpmaxerror_decoration,
2697:                       {I->getType()},
2698:                       {I, MetadataAsValue::get(I->getContext(), MD)});
2699:   }
2700:   if (I->getModule()->getTargetTriple().getVendor() == Triple::AMD &&
2701:       isa<AtomicRMWInst>(I)) {
2702:     // If present, we encode AMDGPU atomic metadata as UserSemantic string
2703:     // decorations, which will be parsed during reverse translation.
2704:     auto &Ctx = B.getContext();
2705:     auto *US = ConstantAsMetadata::get(
2706:         ConstantInt::get(B.getInt32Ty(), SPIRV::Decoration::UserSemantic));
2707:
2708:     SmallVector<Metadata *> MDs;
2709:     if (I->hasMetadata("amdgpu.no.fine.grained.memory"))
2710:       MDs.push_back(MDNode::get(
2711:           Ctx, {US, MDString::get(Ctx, "amdgpu.no.fine.grained.memory")}));
2712:     if (I->hasMetadata("amdgpu.no.remote.memory"))
2713:       MDs.push_back(MDNode::get(
2714:           Ctx, {US, MDString::get(Ctx, "amdgpu.no.remote.memory")}));
2715:     if (I->hasMetadata("amdgpu.ignore.denormal.mode"))
2716:       MDs.push_back(MDNode::get(
2717:           Ctx, {US, MDString::get(Ctx, "amdgpu.ignore.denormal.mode")}));
2718:     if (!MDs.empty())
2719:       B.CreateIntrinsic(Intrinsic::spv_assign_decoration, {I->getType()},
2720:                         {I, MetadataAsValue::get(Ctx, MDNode::get(Ctx, MDs))});
2721:   }
2722: }
2723:
2724: static SPIRV::FPFastMathDefaultInfoVector &getOrCreateFPFastMathDefaultInfoVec(
2725:     const Module &M,
2726:     DenseMap<Function *, SPIRV::FPFastMathDefaultInfoVector>
2727:         &FPFastMathDefaultInfoMap,
2728:     Function *F) {
2729:   auto it = FPFastMathDefaultInfoMap.find(F);
2730:   if (it != FPFastMathDefaultInfoMap.end())
2731:     return it->second;
2732:
2733:   // If the map does not contain the entry, create a new one. Initialize it to
2734:   // contain all 3 elements sorted by bit width of target type: {half, float,
2735:   // double}.
2736:   SPIRV::FPFastMathDefaultInfoVector FPFastMathDefaultInfoVec;
2737:   FPFastMathDefaultInfoVec.emplace_back(Type::getHalfTy(M.getContext()),
2738:                                         SPIRV::FPFastMathMode::None);
2739:   FPFastMathDefaultInfoVec.emplace_back(Type::getFloatTy(M.getContext()),
2740:                                         SPIRV::FPFastMathMode::None);
2741:   FPFastMathDefaultInfoVec.emplace_back(Type::getDoubleTy(M.getContext()),
2742:                                         SPIRV::FPFastMathMode::None);
2743:   return FPFastMathDefaultInfoMap[F] = std::move(FPFastMathDefaultInfoVec);
2744: }
2745:
2746: static SPIRV::FPFastMathDefaultInfo &getFPFastMathDefaultInfo(
2747:     SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec,
2748:     const Type *Ty) {
2749:   size_t BitWidth = Ty->getScalarSizeInBits();
2750:   int Index =
2751:       SPIRV::FPFastMathDefaultInfoVector::computeFPFastMathDefaultInfoVecIndex(
2752:           BitWidth);
2753:   assert(Index >= 0 && Index < 3 &&
2754:          "Expected FPFastMathDefaultInfo for half, float, or double");
2755:   assert(FPFastMathDefaultInfoVec.size() == 3 &&
2756:          "Expected FPFastMathDefaultInfoVec to have exactly 3 elements");
2757:   return FPFastMathDefaultInfoVec[Index];
2758: }
2759:
2760: void SPIRVEmitIntrinsics::insertConstantsForFPFastMathDefault(Module &M) {
```
- EN: This range implements operational logic in helpers such as getSubtargetImpl, setInsertPointAfterDef, MetadataAsValue::get, getType, translating backend policy into executable code.
- CN: 这一段实现了 getSubtargetImpl、setInsertPointAfterDef、MetadataAsValue::get、getType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2761-2880
```cpp
2761:   const SPIRVSubtarget *ST = TM.getSubtargetImpl();
2762:   if (!ST->canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2))
2763:     return;
2764:
2765:   // Store the FPFastMathDefaultInfo in the FPFastMathDefaultInfoMap.
2766:   // We need the entry point (function) as the key, and the target
2767:   // type and flags as the value.
2768:   // We also need to check ContractionOff and SignedZeroInfNanPreserve
2769:   // execution modes, as they are now deprecated and must be replaced
2770:   // with FPFastMathDefaultInfo.
2771:   auto Node = M.getNamedMetadata("spirv.ExecutionMode");
2772:   if (!Node) {
2773:     if (!M.getNamedMetadata("opencl.enable.FP_CONTRACT")) {
2774:       // This requires emitting ContractionOff. However, because
2775:       // ContractionOff is now deprecated, we need to replace it with
2776:       // FPFastMathDefaultInfo with FP Fast Math Mode bitmask set to all 0.
2777:       // We need to create the constant for that.
2778:
2779:       // Create constant instruction with the bitmask flags.
2780:       Constant *InitValue =
2781:           ConstantInt::get(Type::getInt32Ty(M.getContext()), 0);
2782:       // TODO: Reuse constant if there is one already with the required
2783:       // value.
2784:       [[maybe_unused]] GlobalVariable *GV =
2785:           new GlobalVariable(M,                                // Module
2786:                              Type::getInt32Ty(M.getContext()), // Type
2787:                              true,                             // isConstant
2788:                              GlobalValue::InternalLinkage,     // Linkage
2789:                              InitValue                         // Initializer
2790:           );
2791:     }
2792:     return;
2793:   }
2794:
2795:   // The table maps function pointers to their default FP fast math info. It
2796:   // can be assumed that the SmallVector is sorted by the bit width of the
2797:   // type. The first element is the smallest bit width, and the last element
2798:   // is the largest bit width, therefore, we will have {half, float, double}
2799:   // in the order of their bit widths.
2800:   DenseMap<Function *, SPIRV::FPFastMathDefaultInfoVector>
2801:       FPFastMathDefaultInfoMap;
2802:
2803:   for (unsigned i = 0; i < Node->getNumOperands(); i++) {
2804:     MDNode *MDN = cast<MDNode>(Node->getOperand(i));
2805:     assert(MDN->getNumOperands() >= 2 && "Expected at least 2 operands");
2806:     Function *F = cast<Function>(
2807:         cast<ConstantAsMetadata>(MDN->getOperand(0))->getValue());
2808:     const auto EM =
2809:         cast<ConstantInt>(
2810:             cast<ConstantAsMetadata>(MDN->getOperand(1))->getValue())
2811:             ->getZExtValue();
2812:     if (EM == SPIRV::ExecutionMode::FPFastMathDefault) {
2813:       assert(MDN->getNumOperands() == 4 &&
2814:              "Expected 4 operands for FPFastMathDefault");
2815:       const Type *T = cast<ValueAsMetadata>(MDN->getOperand(2))->getType();
2816:       unsigned Flags =
2817:           cast<ConstantInt>(
2818:               cast<ConstantAsMetadata>(MDN->getOperand(3))->getValue())
2819:               ->getZExtValue();
2820:       SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec =
2821:           getOrCreateFPFastMathDefaultInfoVec(M, FPFastMathDefaultInfoMap, F);
2822:       SPIRV::FPFastMathDefaultInfo &Info =
2823:           getFPFastMathDefaultInfo(FPFastMathDefaultInfoVec, T);
2824:       Info.FastMathFlags = Flags;
2825:       Info.FPFastMathDefault = true;
2826:     } else if (EM == SPIRV::ExecutionMode::ContractionOff) {
2827:       assert(MDN->getNumOperands() == 2 &&
2828:              "Expected no operands for ContractionOff");
2829:
2830:       // We need to save this info for every possible FP type, i.e. {half,
2831:       // float, double, fp128}.
2832:       SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec =
2833:           getOrCreateFPFastMathDefaultInfoVec(M, FPFastMathDefaultInfoMap, F);
2834:       for (SPIRV::FPFastMathDefaultInfo &Info : FPFastMathDefaultInfoVec) {
2835:         Info.ContractionOff = true;
2836:       }
2837:     } else if (EM == SPIRV::ExecutionMode::SignedZeroInfNanPreserve) {
2838:       assert(MDN->getNumOperands() == 3 &&
2839:              "Expected 1 operand for SignedZeroInfNanPreserve");
2840:       unsigned TargetWidth =
2841:           cast<ConstantInt>(
2842:               cast<ConstantAsMetadata>(MDN->getOperand(2))->getValue())
2843:               ->getZExtValue();
2844:       // We need to save this info only for the FP type with TargetWidth.
2845:       SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec =
2846:           getOrCreateFPFastMathDefaultInfoVec(M, FPFastMathDefaultInfoMap, F);
2847:       int Index = SPIRV::FPFastMathDefaultInfoVector::
2848:           computeFPFastMathDefaultInfoVecIndex(TargetWidth);
2849:       assert(Index >= 0 && Index < 3 &&
2850:              "Expected FPFastMathDefaultInfo for half, float, or double");
2851:       assert(FPFastMathDefaultInfoVec.size() == 3 &&
2852:              "Expected FPFastMathDefaultInfoVec to have exactly 3 elements");
2853:       FPFastMathDefaultInfoVec[Index].SignedZeroInfNanPreserve = true;
2854:     }
2855:   }
2856:
2857:   std::unordered_map<unsigned, GlobalVariable *> GlobalVars;
2858:   for (auto &[Func, FPFastMathDefaultInfoVec] : FPFastMathDefaultInfoMap) {
2859:     if (FPFastMathDefaultInfoVec.empty())
2860:       continue;
2861:
2862:     for (const SPIRV::FPFastMathDefaultInfo &Info : FPFastMathDefaultInfoVec) {
2863:       assert(Info.Ty && "Expected target type for FPFastMathDefaultInfo");
2864:       // Skip if none of the execution modes was used.
2865:       unsigned Flags = Info.FastMathFlags;
2866:       if (Flags == SPIRV::FPFastMathMode::None && !Info.ContractionOff &&
2867:           !Info.SignedZeroInfNanPreserve && !Info.FPFastMathDefault)
2868:         continue;
2869:
2870:       // Check if flags are compatible.
2871:       if (Info.ContractionOff && (Flags & SPIRV::FPFastMathMode::AllowContract))
2872:         report_fatal_error("Conflicting FPFastMathFlags: ContractionOff "
2873:                            "and AllowContract");
2874:
2875:       if (Info.SignedZeroInfNanPreserve &&
2876:           !(Flags &
2877:             (SPIRV::FPFastMathMode::NotNaN | SPIRV::FPFastMathMode::NotInf |
2878:              SPIRV::FPFastMathMode::NSZ))) {
2879:         if (Info.FPFastMathDefault)
2880:           report_fatal_error("Conflicting FPFastMathFlags: "
```
- EN: This range implements operational logic in helpers such as getSubtargetImpl, getNamedMetadata, ConstantInt::get, getNumOperands, translating backend policy into executable code.
- CN: 这一段实现了 getSubtargetImpl、getNamedMetadata、ConstantInt::get、getNumOperands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2881-3000
```cpp
2881:                              "SignedZeroInfNanPreserve but at least one of "
2882:                              "NotNaN/NotInf/NSZ is enabled.");
2883:       }
2884:
2885:       if ((Flags & SPIRV::FPFastMathMode::AllowTransform) &&
2886:           !((Flags & SPIRV::FPFastMathMode::AllowReassoc) &&
2887:             (Flags & SPIRV::FPFastMathMode::AllowContract))) {
2888:         report_fatal_error("Conflicting FPFastMathFlags: "
2889:                            "AllowTransform requires AllowReassoc and "
2890:                            "AllowContract to be set.");
2891:       }
2892:
2893:       auto it = GlobalVars.find(Flags);
2894:       GlobalVariable *GV = nullptr;
2895:       if (it != GlobalVars.end()) {
2896:         // Reuse existing global variable.
2897:         GV = it->second;
2898:       } else {
2899:         // Create constant instruction with the bitmask flags.
2900:         Constant *InitValue =
2901:             ConstantInt::get(Type::getInt32Ty(M.getContext()), Flags);
2902:         // TODO: Reuse constant if there is one already with the required
2903:         // value.
2904:         GV = new GlobalVariable(M,                                // Module
2905:                                 Type::getInt32Ty(M.getContext()), // Type
2906:                                 true,                             // isConstant
2907:                                 GlobalValue::InternalLinkage,     // Linkage
2908:                                 InitValue                         // Initializer
2909:         );
2910:         GlobalVars[Flags] = GV;
2911:       }
2912:     }
2913:   }
2914: }
2915:
2916: void SPIRVEmitIntrinsics::processInstrAfterVisit(Instruction *I,
2917:                                                  IRBuilder<> &B) {
2918:   auto *II = dyn_cast<IntrinsicInst>(I);
2919:   bool IsConstComposite =
2920:       II && II->getIntrinsicID() == Intrinsic::spv_const_composite;
2921:   if (IsConstComposite && TrackConstants) {
2922:     setInsertPointAfterDef(B, I);
2923:     auto t = AggrConsts.find(I);
2924:     assert(t != AggrConsts.end());
2925:     auto *NewOp =
2926:         buildIntrWithMD(Intrinsic::spv_track_constant,
2927:                         {II->getType(), II->getType()}, t->second, I, {}, B);
2928:     replaceAllUsesWith(I, NewOp, false);
2929:     NewOp->setArgOperand(0, I);
2930:   }
2931:   bool IsPhi = isa<PHINode>(I), BPrepared = false;
2932:   for (const auto &Op : I->operands()) {
2933:     if (isa<PHINode>(I) || isa<SwitchInst>(I) ||
2934:         !(isa<ConstantData>(Op) || isa<ConstantExpr>(Op)))
2935:       continue;
2936:     unsigned OpNo = Op.getOperandNo();
2937:     if (II && ((II->getIntrinsicID() == Intrinsic::spv_gep && OpNo == 0) ||
2938:                (!II->isBundleOperand(OpNo) &&
2939:                 II->paramHasAttr(OpNo, Attribute::ImmArg))))
2940:       continue;
2941:
2942:     if (!BPrepared) {
2943:       IsPhi ? B.SetInsertPointPastAllocas(I->getParent()->getParent())
2944:             : B.SetInsertPoint(I);
2945:       BPrepared = true;
2946:     }
2947:     Type *OpTy = Op->getType();
2948:     Type *OpElemTy = GR->findDeducedElementType(Op);
2949:     Value *NewOp = Op;
2950:     if (OpTy->isTargetExtTy()) {
2951:       // Since this value is replaced by poison, we need to do the same in
2952:       // `insertAssignTypeIntrs`.
2953:       Value *OpTyVal = getNormalizedPoisonValue(OpTy);
2954:       NewOp = buildIntrWithMD(Intrinsic::spv_track_constant,
2955:                               {OpTy, OpTyVal->getType()}, Op, OpTyVal, {}, B);
2956:     }
2957:     if (!IsConstComposite && isPointerTy(OpTy) && OpElemTy != nullptr &&
2958:         OpElemTy != IntegerType::getInt8Ty(I->getContext())) {
2959:       SmallVector<Type *, 2> Types = {OpTy, OpTy};
2960:       SmallVector<Value *, 2> Args = {
2961:           NewOp, buildMD(getNormalizedPoisonValue(OpElemTy)),
2962:           B.getInt32(getPointerAddressSpace(OpTy))};
2963:       CallInst *PtrCasted =
2964:           B.CreateIntrinsic(Intrinsic::spv_ptrcast, {Types}, Args);
2965:       GR->buildAssignPtr(B, OpElemTy, PtrCasted);
2966:       NewOp = PtrCasted;
2967:     }
2968:     if (NewOp != Op)
2969:       I->setOperand(OpNo, NewOp);
2970:   }
2971:   if (Named.insert(I).second)
2972:     emitAssignName(I, B);
2973: }
2974:
2975: Type *SPIRVEmitIntrinsics::deduceFunParamElementType(Function *F,
2976:                                                      unsigned OpIdx) {
2977:   std::unordered_set<Function *> FVisited;
2978:   return deduceFunParamElementType(F, OpIdx, FVisited);
2979: }
2980:
2981: Type *SPIRVEmitIntrinsics::deduceFunParamElementType(
2982:     Function *F, unsigned OpIdx, std::unordered_set<Function *> &FVisited) {
2983:   // maybe a cycle
2984:   if (!FVisited.insert(F).second)
2985:     return nullptr;
2986:
2987:   std::unordered_set<Value *> Visited;
2988:   SmallVector<std::pair<Function *, unsigned>> Lookup;
2989:   // search in function's call sites
2990:   for (User *U : F->users()) {
2991:     CallInst *CI = dyn_cast<CallInst>(U);
2992:     if (!CI || OpIdx >= CI->arg_size())
2993:       continue;
2994:     Value *OpArg = CI->getArgOperand(OpIdx);
2995:     if (!isPointerTy(OpArg->getType()))
2996:       continue;
2997:     // maybe we already know operand's element type
2998:     if (Type *KnownTy = GR->findDeducedElementType(OpArg))
2999:       return KnownTy;
3000:     // try to deduce from the operand itself
```
- EN: This range implements operational logic in helpers such as find, ConstantInt::get, setInsertPointAfterDef, assert, translating backend policy into executable code.
- CN: 这一段实现了 find、ConstantInt::get、setInsertPointAfterDef、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3001-3120
```cpp
3001:     Visited.clear();
3002:     if (Type *Ty = deduceElementTypeHelper(OpArg, Visited, false))
3003:       return Ty;
3004:     // search in actual parameter's users
3005:     for (User *OpU : OpArg->users()) {
3006:       Instruction *Inst = dyn_cast<Instruction>(OpU);
3007:       if (!Inst || Inst == CI)
3008:         continue;
3009:       Visited.clear();
3010:       if (Type *Ty = deduceElementTypeHelper(Inst, Visited, false))
3011:         return Ty;
3012:     }
3013:     // check if it's a formal parameter of the outer function
3014:     if (!CI->getParent() || !CI->getParent()->getParent())
3015:       continue;
3016:     Function *OuterF = CI->getParent()->getParent();
3017:     if (FVisited.find(OuterF) != FVisited.end())
3018:       continue;
3019:     for (unsigned i = 0; i < OuterF->arg_size(); ++i) {
3020:       if (OuterF->getArg(i) == OpArg) {
3021:         Lookup.push_back(std::make_pair(OuterF, i));
3022:         break;
3023:       }
3024:     }
3025:   }
3026:
3027:   // search in function parameters
3028:   for (auto &Pair : Lookup) {
3029:     if (Type *Ty = deduceFunParamElementType(Pair.first, Pair.second, FVisited))
3030:       return Ty;
3031:   }
3032:
3033:   return nullptr;
3034: }
3035:
3036: void SPIRVEmitIntrinsics::processParamTypesByFunHeader(Function *F,
3037:                                                        IRBuilder<> &B) {
3038:   B.SetInsertPointPastAllocas(F);
3039:   for (unsigned OpIdx = 0; OpIdx < F->arg_size(); ++OpIdx) {
3040:     Argument *Arg = F->getArg(OpIdx);
3041:     if (!isUntypedPointerTy(Arg->getType()))
3042:       continue;
3043:     Type *ElemTy = GR->findDeducedElementType(Arg);
3044:     if (ElemTy)
3045:       continue;
3046:     if (hasPointeeTypeAttr(Arg) &&
3047:         (ElemTy = getPointeeTypeByAttr(Arg)) != nullptr) {
3048:       GR->buildAssignPtr(B, ElemTy, Arg);
3049:       continue;
3050:     }
3051:     // search in function's call sites
3052:     for (User *U : F->users()) {
3053:       CallInst *CI = dyn_cast<CallInst>(U);
3054:       if (!CI || OpIdx >= CI->arg_size())
3055:         continue;
3056:       Value *OpArg = CI->getArgOperand(OpIdx);
3057:       if (!isPointerTy(OpArg->getType()))
3058:         continue;
3059:       // maybe we already know operand's element type
3060:       if ((ElemTy = GR->findDeducedElementType(OpArg)) != nullptr)
3061:         break;
3062:     }
3063:     if (ElemTy) {
3064:       GR->buildAssignPtr(B, ElemTy, Arg);
3065:       continue;
3066:     }
3067:     if (HaveFunPtrs) {
3068:       for (User *U : Arg->users()) {
3069:         CallInst *CI = dyn_cast<CallInst>(U);
3070:         if (CI && !isa<IntrinsicInst>(CI) && CI->isIndirectCall() &&
3071:             CI->getCalledOperand() == Arg &&
3072:             CI->getParent()->getParent() == CurrF) {
3073:           SmallVector<std::pair<Value *, unsigned>> Ops;
3074:           deduceOperandElementTypeFunctionPointer(CI, Ops, ElemTy, false);
3075:           if (ElemTy) {
3076:             GR->buildAssignPtr(B, ElemTy, Arg);
3077:             break;
3078:           }
3079:         }
3080:       }
3081:     }
3082:   }
3083: }
3084:
3085: void SPIRVEmitIntrinsics::processParamTypes(Function *F, IRBuilder<> &B) {
3086:   B.SetInsertPointPastAllocas(F);
3087:   for (unsigned OpIdx = 0; OpIdx < F->arg_size(); ++OpIdx) {
3088:     Argument *Arg = F->getArg(OpIdx);
3089:     if (!isUntypedPointerTy(Arg->getType()))
3090:       continue;
3091:     Type *ElemTy = GR->findDeducedElementType(Arg);
3092:     if (!ElemTy && (ElemTy = deduceFunParamElementType(F, OpIdx)) != nullptr) {
3093:       if (CallInst *AssignCI = GR->findAssignPtrTypeInstr(Arg)) {
3094:         DenseSet<std::pair<Value *, Value *>> VisitedSubst;
3095:         GR->updateAssignType(AssignCI, Arg, getNormalizedPoisonValue(ElemTy));
3096:         propagateElemType(Arg, IntegerType::getInt8Ty(F->getContext()),
3097:                           VisitedSubst);
3098:       } else {
3099:         GR->buildAssignPtr(B, ElemTy, Arg);
3100:       }
3101:     }
3102:   }
3103: }
3104:
3105: static FunctionType *getFunctionPointerElemType(Function *F,
3106:                                                 SPIRVGlobalRegistry *GR) {
3107:   FunctionType *FTy = F->getFunctionType();
3108:   bool IsNewFTy = false;
3109:   SmallVector<Type *, 4> ArgTys;
3110:   for (Argument &Arg : F->args()) {
3111:     Type *ArgTy = Arg.getType();
3112:     if (ArgTy->isPointerTy())
3113:       if (Type *ElemTy = GR->findDeducedElementType(&Arg)) {
3114:         IsNewFTy = true;
3115:         ArgTy = getTypedPointerWrapper(ElemTy, getPointerAddressSpace(ArgTy));
3116:       }
3117:     ArgTys.push_back(ArgTy);
3118:   }
3119:   return IsNewFTy
3120:              ? FunctionType::get(FTy->getReturnType(), ArgTys, FTy->isVarArg())
```
- EN: This range implements operational logic in helpers such as clear, getParent, arg_size, push_back, translating backend policy into executable code.
- CN: 这一段实现了 clear、getParent、arg_size、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3121-3240
```cpp
3121:              : FTy;
3122: }
3123:
3124: bool SPIRVEmitIntrinsics::processFunctionPointers(Module &M) {
3125:   SmallVector<Function *> Worklist;
3126:   for (auto &F : M) {
3127:     if (F.isIntrinsic())
3128:       continue;
3129:     if (F.isDeclaration()) {
3130:       for (User *U : F.users()) {
3131:         CallInst *CI = dyn_cast<CallInst>(U);
3132:         if (!CI || CI->getCalledFunction() != &F) {
3133:           Worklist.push_back(&F);
3134:           break;
3135:         }
3136:       }
3137:     } else {
3138:       if (F.user_empty())
3139:         continue;
3140:       Type *FPElemTy = GR->findDeducedElementType(&F);
3141:       if (!FPElemTy)
3142:         FPElemTy = getFunctionPointerElemType(&F, GR);
3143:       for (User *U : F.users()) {
3144:         IntrinsicInst *II = dyn_cast<IntrinsicInst>(U);
3145:         if (!II || II->arg_size() != 3 || II->getOperand(0) != &F)
3146:           continue;
3147:         if (II->getIntrinsicID() == Intrinsic::spv_assign_ptr_type ||
3148:             II->getIntrinsicID() == Intrinsic::spv_ptrcast) {
3149:           GR->updateAssignType(II, &F, getNormalizedPoisonValue(FPElemTy));
3150:           break;
3151:         }
3152:       }
3153:     }
3154:   }
3155:   if (Worklist.empty())
3156:     return false;
3157:
3158:   LLVMContext &Ctx = M.getContext();
3159:   Function *SF = getOrCreateBackendServiceFunction(M);
3160:   BasicBlock *BB = BasicBlock::Create(Ctx, "entry", SF);
3161:   IRBuilder<> IRB(BB);
3162:
3163:   for (Function *F : Worklist) {
3164:     SmallVector<Value *> Args;
3165:     for (const auto &Arg : F->args())
3166:       Args.push_back(getNormalizedPoisonValue(Arg.getType()));
3167:     IRB.CreateCall(F, Args);
3168:   }
3169:   IRB.CreateRetVoid();
3170:
3171:   return true;
3172: }
3173:
3174: // Apply types parsed from demangled function declarations.
3175: void SPIRVEmitIntrinsics::applyDemangledPtrArgTypes(IRBuilder<> &B) {
3176:   DenseMap<Function *, CallInst *> Ptrcasts;
3177:   for (auto It : FDeclPtrTys) {
3178:     Function *F = It.first;
3179:     for (auto *U : F->users()) {
3180:       CallInst *CI = dyn_cast<CallInst>(U);
3181:       if (!CI || CI->getCalledFunction() != F)
3182:         continue;
3183:       unsigned Sz = CI->arg_size();
3184:       for (auto [Idx, ElemTy] : It.second) {
3185:         if (Idx >= Sz)
3186:           continue;
3187:         Value *Param = CI->getArgOperand(Idx);
3188:         if (GR->findDeducedElementType(Param) || isa<GlobalValue>(Param))
3189:           continue;
3190:         if (Argument *Arg = dyn_cast<Argument>(Param)) {
3191:           if (!hasPointeeTypeAttr(Arg)) {
3192:             B.SetInsertPointPastAllocas(Arg->getParent());
3193:             B.SetCurrentDebugLocation(DebugLoc());
3194:             GR->buildAssignPtr(B, ElemTy, Arg);
3195:           }
3196:         } else if (isaGEP(Param)) {
3197:           replaceUsesOfWithSpvPtrcast(Param, normalizeType(ElemTy), CI,
3198:                                       Ptrcasts);
3199:         } else if (isa<Instruction>(Param)) {
3200:           GR->addDeducedElementType(Param, normalizeType(ElemTy));
3201:           // insertAssignTypeIntrs() will complete buildAssignPtr()
3202:         } else {
3203:           B.SetInsertPoint(CI->getParent()
3204:                                ->getParent()
3205:                                ->getEntryBlock()
3206:                                .getFirstNonPHIOrDbgOrAlloca());
3207:           GR->buildAssignPtr(B, ElemTy, Param);
3208:         }
3209:         CallInst *Ref = dyn_cast<CallInst>(Param);
3210:         if (!Ref)
3211:           continue;
3212:         Function *RefF = Ref->getCalledFunction();
3213:         if (!RefF || !isPointerTy(RefF->getReturnType()) ||
3214:             GR->findDeducedElementType(RefF))
3215:           continue;
3216:         ElemTy = normalizeType(ElemTy);
3217:         GR->addDeducedElementType(RefF, ElemTy);
3218:         GR->addReturnType(
3219:             RefF, TypedPointerType::get(
3220:                       ElemTy, getPointerAddressSpace(RefF->getReturnType())));
3221:       }
3222:     }
3223:   }
3224: }
3225:
3226: GetElementPtrInst *
3227: SPIRVEmitIntrinsics::simplifyZeroLengthArrayGepInst(GetElementPtrInst *GEP) {
3228:   // getelementptr [0 x T], P, 0 (zero), I -> getelementptr T, P, I.
3229:   // If type is 0-length array and first index is 0 (zero), drop both the
3230:   // 0-length array type and the first index. This is a common pattern in
3231:   // the IR, e.g. when using a zero-length array as a placeholder for a
3232:   // flexible array such as unbound arrays.
3233:   assert(GEP && "GEP is null");
3234:   Type *SrcTy = GEP->getSourceElementType();
3235:   SmallVector<Value *, 8> Indices(GEP->indices());
3236:   ArrayType *ArrTy = dyn_cast<ArrayType>(SrcTy);
3237:   if (ArrTy && ArrTy->getNumElements() == 0 && match(Indices[0], m_Zero())) {
3238:     Indices.erase(Indices.begin());
3239:     SrcTy = ArrTy->getElementType();
3240:     return GetElementPtrInst::Create(SrcTy, GEP->getPointerOperand(), Indices,
```
- EN: This range implements operational logic in helpers such as SPIRVEmitIntrinsics::processFunctionPointers, push_back, findDeducedElementType, getFunctionPointerElemType, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVEmitIntrinsics::processFunctionPointers、push_back、findDeducedElementType、getFunctionPointerElemType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3241-3360
```cpp
3241:                                      GEP->getNoWrapFlags(), "",
3242:                                      GEP->getIterator());
3243:   }
3244:   return nullptr;
3245: }
3246:
3247: void SPIRVEmitIntrinsics::emitUnstructuredLoopControls(Function &F,
3248:                                                        IRBuilder<> &B) {
3249:   const SPIRVSubtarget *ST = TM.getSubtargetImpl(F);
3250:   // Shaders use SPIRVStructurizer which emits OpLoopMerge via spv_loop_merge.
3251:   if (ST->isShader())
3252:     return;
3253:
3254:   if (ST->canUseExtension(
3255:           SPIRV::Extension::SPV_INTEL_unstructured_loop_controls)) {
3256:     for (BasicBlock &BB : F) {
3257:       Instruction *Term = BB.getTerminator();
3258:       MDNode *LoopMD = Term->getMetadata(LLVMContext::MD_loop);
3259:       if (!LoopMD)
3260:         continue;
3261:
3262:       SmallVector<unsigned, 1> Ops =
3263:           getSpirvLoopControlOperandsFromLoopMetadata(LoopMD);
3264:       unsigned LC = Ops[0];
3265:       if (LC == SPIRV::LoopControl::None)
3266:         continue;
3267:
3268:       // Emit intrinsic: loop control mask + optional parameters.
3269:       B.SetInsertPoint(Term);
3270:       SmallVector<Value *, 4> IntrArgs;
3271:       for (unsigned Op : Ops)
3272:         IntrArgs.push_back(B.getInt32(Op));
3273:       B.CreateIntrinsic(Intrinsic::spv_loop_control_intel, IntrArgs);
3274:     }
3275:     return;
3276:   }
3277:
3278:   // For non-shader targets without the Intel extension, emit OpLoopMerge
3279:   // using spv_loop_merge intrinsics, mirroring the structurizer approach.
3280:   DominatorTree DT(F);
3281:   LoopInfo LI(DT);
3282:   if (LI.empty())
3283:     return;
3284:
3285:   for (Loop *L : LI.getLoopsInPreorder()) {
3286:     BasicBlock *Latch = L->getLoopLatch();
3287:     if (!Latch)
3288:       continue;
3289:     BasicBlock *MergeBlock = L->getUniqueExitBlock();
3290:     if (!MergeBlock)
3291:       continue;
3292:
3293:     // Check for loop unroll metadata on the latch terminator.
3294:     SmallVector<unsigned, 1> LoopControlOps =
3295:         getSpirvLoopControlOperandsFromLoopMetadata(L);
3296:     if (LoopControlOps[0] == SPIRV::LoopControl::None)
3297:       continue;
3298:
3299:     BasicBlock *Header = L->getHeader();
3300:     B.SetInsertPoint(Header->getTerminator());
3301:     auto *MergeAddress = BlockAddress::get(&F, MergeBlock);
3302:     auto *ContinueAddress = BlockAddress::get(&F, Latch);
3303:     SmallVector<Value *, 4> Args = {MergeAddress, ContinueAddress};
3304:     for (unsigned Imm : LoopControlOps)
3305:       Args.emplace_back(B.getInt32(Imm));
3306:     B.CreateIntrinsic(Intrinsic::spv_loop_merge, {Args});
3307:   }
3308: }
3309:
3310: bool SPIRVEmitIntrinsics::runOnFunction(Function &Func) {
3311:   if (Func.isDeclaration())
3312:     return false;
3313:
3314:   const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(Func);
3315:   GR = ST.getSPIRVGlobalRegistry();
3316:
3317:   if (!CurrF)
3318:     HaveFunPtrs =
3319:         ST.canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers);
3320:
3321:   CurrF = &Func;
3322:   IRBuilder<> B(Func.getContext());
3323:   AggrConsts.clear();
3324:   AggrConstTypes.clear();
3325:   AggrStores.clear();
3326:   DeletedInstrs.clear();
3327:
3328:   processParamTypesByFunHeader(CurrF, B);
3329:
3330:   // Fix GEP result types ahead of inference, and simplify if possible.
3331:   // Data structure for dead instructions that were simplified and replaced.
3332:   SmallPtrSet<Instruction *, 4> DeadInsts;
3333:   for (auto &I : instructions(Func)) {
3334:     auto *GEP = dyn_cast<GetElementPtrInst>(&I);
3335:     auto *SGEP = dyn_cast<StructuredGEPInst>(&I);
3336:
3337:     if ((!GEP && !SGEP) || GR->findDeducedElementType(&I))
3338:       continue;
3339:
3340:     if (SGEP) {
3341:       GR->addDeducedElementType(SGEP,
3342:                                 normalizeType(SGEP->getResultElementType()));
3343:       continue;
3344:     }
3345:
3346:     GetElementPtrInst *NewGEP = simplifyZeroLengthArrayGepInst(GEP);
3347:     if (NewGEP) {
3348:       GEP->replaceAllUsesWith(NewGEP);
3349:       DeadInsts.insert(GEP);
3350:       GEP = NewGEP;
3351:     }
3352:     if (Type *GepTy = getGEPType(GEP))
3353:       GR->addDeducedElementType(GEP, normalizeType(GepTy));
3354:   }
3355:   // Remove dead instructions that were simplified and replaced.
3356:   for (auto *I : DeadInsts) {
3357:     assert(I->use_empty() && "Dead instruction should not have any uses left");
3358:     I->eraseFromParent();
3359:   }
3360:
```
- EN: This range implements operational logic in helpers such as getIterator, getSubtargetImpl, getTerminator, getMetadata, translating backend policy into executable code.
- CN: 这一段实现了 getIterator、getSubtargetImpl、getTerminator、getMetadata 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3361-3480
```cpp
3361:   // StoreInst's operand type can be changed during the next
3362:   // transformations, so we need to store it in the set. Also store already
3363:   // transformed types.
3364:   for (auto &I : instructions(Func)) {
3365:     StoreInst *SI = dyn_cast<StoreInst>(&I);
3366:     if (!SI)
3367:       continue;
3368:     Type *ElTy = SI->getValueOperand()->getType();
3369:     if (ElTy->isAggregateType() || ElTy->isVectorTy())
3370:       AggrStores.insert(&I);
3371:   }
3372:
3373:   B.SetInsertPoint(&Func.getEntryBlock(), Func.getEntryBlock().begin());
3374:   for (auto &GV : Func.getParent()->globals())
3375:     processGlobalValue(GV, B);
3376:
3377:   preprocessUndefs(B);
3378:   simplifyNullAddrSpaceCasts();
3379:   preprocessCompositeConstants(B);
3380:
3381:   for (BasicBlock &BB : Func)
3382:     for (PHINode &Phi : BB.phis())
3383:       if (Phi.getType()->isAggregateType()) {
3384:         AggrConstTypes[&Phi] = Phi.getType();
3385:         Phi.mutateType(B.getInt32Ty());
3386:       }
3387:
3388:   preprocessBoolVectorBitcasts(Func);
3389:   SmallVector<Instruction *> Worklist(
3390:       llvm::make_pointer_range(instructions(Func)));
3391:
3392:   applyDemangledPtrArgTypes(B);
3393:
3394:   // Pass forward: use operand to deduce instructions result.
3395:   for (auto &I : Worklist) {
3396:     // Don't emit intrinsincs for convergence intrinsics.
3397:     if (isConvergenceIntrinsic(I))
3398:       continue;
3399:
3400:     bool Postpone = insertAssignPtrTypeIntrs(I, B, false);
3401:     // if Postpone is true, we can't decide on pointee type yet
3402:     insertAssignTypeIntrs(I, B);
3403:     insertPtrCastOrAssignTypeInstr(I, B);
3404:     insertSpirvDecorations(I, B);
3405:     // if instruction requires a pointee type set, let's check if we know it
3406:     // already, and force it to be i8 if not
3407:     if (Postpone && !GR->findAssignPtrTypeInstr(I))
3408:       insertAssignPtrTypeIntrs(I, B, true);
3409:
3410:     if (auto *FPI = dyn_cast<ConstrainedFPIntrinsic>(I))
3411:       useRoundingMode(FPI, B);
3412:   }
3413:
3414:   // Pass backward: use instructions results to specify/update/cast operands
3415:   // where needed.
3416:   SmallPtrSet<Instruction *, 4> IncompleteRets;
3417:   for (auto &I : llvm::reverse(instructions(Func)))
3418:     deduceOperandElementType(&I, &IncompleteRets);
3419:
3420:   // Pass forward for PHIs only, their operands are not preceed the
3421:   // instruction in meaning of `instructions(Func)`.
3422:   for (BasicBlock &BB : Func)
3423:     for (PHINode &Phi : BB.phis())
3424:       if (isPointerTy(Phi.getType()))
3425:         deduceOperandElementType(&Phi, nullptr);
3426:
3427:   for (auto *I : Worklist) {
3428:     if (DeletedInstrs.count(I))
3429:       continue;
3430:     TrackConstants = true;
3431:     if (!I->getType()->isVoidTy() || isa<StoreInst>(I))
3432:       setInsertPointAfterDef(B, I);
3433:     // Visitors return either the original/newly created instruction for
3434:     // further processing, nullptr otherwise.
3435:     I = visit(*I);
3436:     if (!I)
3437:       continue;
3438:
3439:     // Don't emit intrinsics for convergence operations.
3440:     if (isConvergenceIntrinsic(I))
3441:       continue;
3442:
3443:     addSaturatedDecorationToIntrinsic(I, B);
3444:     processInstrAfterVisit(I, B);
3445:   }
3446:
3447:   emitUnstructuredLoopControls(Func, B);
3448:
3449:   return true;
3450: }
3451:
3452: // Try to deduce a better type for pointers to untyped ptr.
3453: bool SPIRVEmitIntrinsics::postprocessTypes(Module &M) {
3454:   if (!GR || TodoTypeSz == 0)
3455:     return false;
3456:
3457:   unsigned SzTodo = TodoTypeSz;
3458:   DenseMap<Value *, SmallPtrSet<Value *, 4>> ToProcess;
3459:   for (auto [Op, Enabled] : TodoType) {
3460:     // TODO: add isa<CallInst>(Op) to continue
3461:     if (!Enabled || isaGEP(Op))
3462:       continue;
3463:     CallInst *AssignCI = GR->findAssignPtrTypeInstr(Op);
3464:     Type *KnownTy = GR->findDeducedElementType(Op);
3465:     if (!KnownTy || !AssignCI)
3466:       continue;
3467:     assert(Op == AssignCI->getArgOperand(0));
3468:     // Try to improve the type deduced after all Functions are processed.
3469:     if (auto *CI = dyn_cast<Instruction>(Op)) {
3470:       CurrF = CI->getParent()->getParent();
3471:       std::unordered_set<Value *> Visited;
3472:       if (Type *ElemTy = deduceElementTypeHelper(Op, Visited, false, true)) {
3473:         if (ElemTy != KnownTy) {
3474:           DenseSet<std::pair<Value *, Value *>> VisitedSubst;
3475:           propagateElemType(CI, ElemTy, VisitedSubst);
3476:           eraseTodoType(Op);
3477:           continue;
3478:         }
3479:       }
3480:     }
```
- EN: This range implements operational logic in helpers such as getValueOperand, insert, SetInsertPoint, processGlobalValue, translating backend policy into executable code.
- CN: 这一段实现了 getValueOperand、insert、SetInsertPoint、processGlobalValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3481-3600
```cpp
3481:
3482:     if (Op->hasUseList()) {
3483:       for (User *U : Op->users()) {
3484:         Instruction *Inst = dyn_cast<Instruction>(U);
3485:         if (Inst && !isa<IntrinsicInst>(Inst))
3486:           ToProcess[Inst].insert(Op);
3487:       }
3488:     }
3489:   }
3490:   if (TodoTypeSz == 0)
3491:     return true;
3492:
3493:   for (auto &F : M) {
3494:     CurrF = &F;
3495:     SmallPtrSet<Instruction *, 4> IncompleteRets;
3496:     for (auto &I : llvm::reverse(instructions(F))) {
3497:       auto It = ToProcess.find(&I);
3498:       if (It == ToProcess.end())
3499:         continue;
3500:       It->second.remove_if([this](Value *V) { return !isTodoType(V); });
3501:       if (It->second.size() == 0)
3502:         continue;
3503:       deduceOperandElementType(&I, &IncompleteRets, &It->second, true);
3504:       if (TodoTypeSz == 0)
3505:         return true;
3506:     }
3507:   }
3508:
3509:   return SzTodo > TodoTypeSz;
3510: }
3511:
3512: // Parse and store argument types of function declarations where needed.
3513: void SPIRVEmitIntrinsics::parseFunDeclarations(Module &M) {
3514:   for (auto &F : M) {
3515:     if (!F.isDeclaration() || F.isIntrinsic())
3516:       continue;
3517:     // get the demangled name
3518:     std::string DemangledName = getOclOrSpirvBuiltinDemangledName(F.getName());
3519:     if (DemangledName.empty())
3520:       continue;
3521:     // allow only OpGroupAsyncCopy use case at the moment
3522:     const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(F);
3523:     auto [Grp, Opcode, ExtNo] = SPIRV::mapBuiltinToOpcode(
3524:         DemangledName, ST.getPreferredInstructionSet());
3525:     if (Opcode != SPIRV::OpGroupAsyncCopy)
3526:       continue;
3527:     // find pointer arguments
3528:     SmallVector<unsigned> Idxs;
3529:     for (unsigned OpIdx = 0; OpIdx < F.arg_size(); ++OpIdx) {
3530:       Argument *Arg = F.getArg(OpIdx);
3531:       if (isPointerTy(Arg->getType()) && !hasPointeeTypeAttr(Arg))
3532:         Idxs.push_back(OpIdx);
3533:     }
3534:     if (!Idxs.size())
3535:       continue;
3536:     // parse function arguments
3537:     LLVMContext &Ctx = F.getContext();
3538:     SmallVector<StringRef, 10> TypeStrs;
3539:     SPIRV::parseBuiltinTypeStr(TypeStrs, DemangledName, Ctx);
3540:     if (!TypeStrs.size())
3541:       continue;
3542:     // find type info for pointer arguments
3543:     for (unsigned Idx : Idxs) {
3544:       if (Idx >= TypeStrs.size())
3545:         continue;
3546:       if (Type *ElemTy =
3547:               SPIRV::parseBuiltinCallArgumentType(TypeStrs[Idx].trim(), Ctx))
3548:         if (TypedPointerType::isValidElementType(ElemTy) &&
3549:             !ElemTy->isTargetExtTy())
3550:           FDeclPtrTys[&F].push_back(std::make_pair(Idx, ElemTy));
3551:     }
3552:   }
3553: }
3554:
3555: bool SPIRVEmitIntrinsics::processMaskedMemIntrinsic(IntrinsicInst &I) {
3556:   const SPIRVSubtarget &ST = TM.getSubtarget<SPIRVSubtarget>(*I.getFunction());
3557:
3558:   if (I.getIntrinsicID() == Intrinsic::masked_gather) {
3559:     if (!ST.canUseExtension(
3560:             SPIRV::Extension::SPV_INTEL_masked_gather_scatter)) {
3561:       I.getContext().emitError(
3562:           &I, "llvm.masked.gather requires SPV_INTEL_masked_gather_scatter "
3563:               "extension");
3564:       // Replace with poison to allow compilation to continue and report error.
3565:       I.replaceAllUsesWith(PoisonValue::get(I.getType()));
3566:       I.eraseFromParent();
3567:       return true;
3568:     }
3569:
3570:     IRBuilder<> B(&I);
3571:
3572:     Value *Ptrs = I.getArgOperand(0);
3573:     Value *Mask = I.getArgOperand(1);
3574:     Value *Passthru = I.getArgOperand(2);
3575:
3576:     // Alignment is stored as a parameter attribute, not as a regular parameter.
3577:     uint32_t Alignment = I.getParamAlign(0).valueOrOne().value();
3578:
3579:     SmallVector<Value *, 4> Args = {Ptrs, B.getInt32(Alignment), Mask,
3580:                                     Passthru};
3581:     SmallVector<Type *, 4> Types = {I.getType(), Ptrs->getType(),
3582:                                     Mask->getType(), Passthru->getType()};
3583:
3584:     auto *NewI = B.CreateIntrinsic(Intrinsic::spv_masked_gather, Types, Args);
3585:     I.replaceAllUsesWith(NewI);
3586:     I.eraseFromParent();
3587:     return true;
3588:   }
3589:
3590:   if (I.getIntrinsicID() == Intrinsic::masked_scatter) {
3591:     if (!ST.canUseExtension(
3592:             SPIRV::Extension::SPV_INTEL_masked_gather_scatter)) {
3593:       I.getContext().emitError(
3594:           &I, "llvm.masked.scatter requires SPV_INTEL_masked_gather_scatter "
3595:               "extension");
3596:       // Erase the intrinsic to allow compilation to continue and report error.
3597:       I.eraseFromParent();
3598:       return true;
3599:     }
3600:
```
- EN: This range implements operational logic in helpers such as insert, find, remove_if, deduceOperandElementType, translating backend policy into executable code.
- CN: 这一段实现了 insert、find、remove_if、deduceOperandElementType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3601-3720
```cpp
3601:     IRBuilder<> B(&I);
3602:
3603:     Value *Values = I.getArgOperand(0);
3604:     Value *Ptrs = I.getArgOperand(1);
3605:     Value *Mask = I.getArgOperand(2);
3606:
3607:     // Alignment is stored as a parameter attribute on the ptrs parameter (arg
3608:     // 1).
3609:     uint32_t Alignment = I.getParamAlign(1).valueOrOne().value();
3610:
3611:     SmallVector<Value *, 4> Args = {Values, Ptrs, B.getInt32(Alignment), Mask};
3612:     SmallVector<Type *, 3> Types = {Values->getType(), Ptrs->getType(),
3613:                                     Mask->getType()};
3614:
3615:     B.CreateIntrinsic(Intrinsic::spv_masked_scatter, Types, Args);
3616:     I.eraseFromParent();
3617:     return true;
3618:   }
3619:
3620:   return false;
3621: }
3622:
3623: // SPIR-V doesn't support bitcasts involving vector boolean type. Decompose such
3624: // bitcasts into element-wise operations before building instructions
3625: // worklist, so new instructions are properly visited and converted to
3626: // SPIR-V intrinsics.
3627: void SPIRVEmitIntrinsics::preprocessBoolVectorBitcasts(Function &F) {
3628:   struct BoolVecBitcast {
3629:     BitCastInst *BC;
3630:     FixedVectorType *BoolVecTy;
3631:     bool SrcIsBoolVec;
3632:   };
3633:
3634:   auto getAsBoolVec = [](Type *Ty) -> FixedVectorType * {
3635:     auto *VTy = dyn_cast<FixedVectorType>(Ty);
3636:     return (VTy && VTy->getElementType()->isIntegerTy(1)) ? VTy : nullptr;
3637:   };
3638:
3639:   SmallVector<BoolVecBitcast, 4> ToReplace;
3640:   for (auto &I : instructions(F)) {
3641:     auto *BC = dyn_cast<BitCastInst>(&I);
3642:     if (!BC)
3643:       continue;
3644:     if (auto *BVTy = getAsBoolVec(BC->getSrcTy()))
3645:       ToReplace.push_back({BC, BVTy, true});
3646:     else if (auto *BVTy = getAsBoolVec(BC->getDestTy()))
3647:       ToReplace.push_back({BC, BVTy, false});
3648:   }
3649:
3650:   for (auto &[BC, BoolVecTy, SrcIsBoolVec] : ToReplace) {
3651:     IRBuilder<> B(BC);
3652:     Value *Src = BC->getOperand(0);
3653:     unsigned BoolVecN = BoolVecTy->getNumElements();
3654:     // Use iN as the scalar intermediate type for the bool vector side.
3655:     Type *IntTy = B.getIntNTy(BoolVecN);
3656:
3657:     // Convert source to scalar integer.
3658:     Value *IntVal;
3659:     if (SrcIsBoolVec) {
3660:       // Extract each bool, zext, shift, and OR.
3661:       IntVal = ConstantInt::get(IntTy, 0);
3662:       for (unsigned I = 0; I < BoolVecN; ++I) {
3663:         Value *Elem = B.CreateExtractElement(Src, B.getInt32(I));
3664:         Value *Ext = B.CreateZExt(Elem, IntTy);
3665:         if (I > 0)
3666:           Ext = B.CreateShl(Ext, ConstantInt::get(IntTy, I));
3667:         IntVal = B.CreateOr(IntVal, Ext);
3668:       }
3669:     } else {
3670:       // Source is a non-bool type. If it's already a scalar integer, use it
3671:       // directly, otherwise bitcast to iN first.
3672:       IntVal = Src;
3673:       if (!Src->getType()->isIntegerTy())
3674:         IntVal = B.CreateBitCast(Src, IntTy);
3675:     }
3676:
3677:     // Convert scalar integer to destination type.
3678:     Value *Result;
3679:     if (!SrcIsBoolVec) {
3680:       // Test each bit with AND + icmp.
3681:       Result = PoisonValue::get(BoolVecTy);
3682:       for (unsigned I = 0; I < BoolVecN; ++I) {
3683:         Value *Mask = ConstantInt::get(IntTy, APInt::getOneBitSet(BoolVecN, I));
3684:         Value *And = B.CreateAnd(IntVal, Mask);
3685:         Value *Cmp = B.CreateICmpNE(And, ConstantInt::get(IntTy, 0));
3686:         Result = B.CreateInsertElement(Result, Cmp, B.getInt32(I));
3687:       }
3688:     } else {
3689:       // Destination is a non-bool type. If it's a scalar integer, use IntVal
3690:       // directly, otherwise bitcast from iN.
3691:       Result = IntVal;
3692:       if (!BC->getDestTy()->isIntegerTy())
3693:         Result = B.CreateBitCast(IntVal, BC->getDestTy());
3694:     }
3695:
3696:     BC->replaceAllUsesWith(Result);
3697:     BC->eraseFromParent();
3698:   }
3699: }
3700:
3701: bool SPIRVEmitIntrinsics::convertMaskedMemIntrinsics(Module &M) {
3702:   bool Changed = false;
3703:
3704:   for (Function &F : make_early_inc_range(M)) {
3705:     if (!F.isIntrinsic())
3706:       continue;
3707:     Intrinsic::ID IID = F.getIntrinsicID();
3708:     if (IID != Intrinsic::masked_gather && IID != Intrinsic::masked_scatter)
3709:       continue;
3710:
3711:     for (User *U : make_early_inc_range(F.users())) {
3712:       if (auto *II = dyn_cast<IntrinsicInst>(U))
3713:         Changed |= processMaskedMemIntrinsic(*II);
3714:     }
3715:
3716:     if (F.use_empty())
3717:       F.eraseFromParent();
3718:   }
3719:
3720:   return Changed;
```
- EN: This range defines or declares important types such as B, getArgOperand, getParamAlign, CreateIntrinsic, shaping the data model used by SPIRVEmitIntrinsics.cpp.
- CN: 这一段定义或声明了 B、getArgOperand、getParamAlign、CreateIntrinsic 等关键类型，构成 SPIRVEmitIntrinsics.cpp 使用的数据模型。

### Lines 3721-3765
```cpp
3721: }
3722:
3723: bool SPIRVEmitIntrinsics::runOnModule(Module &M) {
3724:   bool Changed = false;
3725:
3726:   Changed |= convertMaskedMemIntrinsics(M);
3727:
3728:   parseFunDeclarations(M);
3729:   insertConstantsForFPFastMathDefault(M);
3730:   GVUsers.init(M);
3731:
3732:   TodoType.clear();
3733:   for (auto &F : M)
3734:     Changed |= runOnFunction(F);
3735:
3736:   // Specify function parameters after all functions were processed.
3737:   for (auto &F : M) {
3738:     // check if function parameter types are set
3739:     CurrF = &F;
3740:     if (!F.isDeclaration() && !F.isIntrinsic()) {
3741:       IRBuilder<> B(F.getContext());
3742:       processParamTypes(&F, B);
3743:     }
3744:   }
3745:
3746:   CanTodoType = false;
3747:   Changed |= postprocessTypes(M);
3748:
3749:   if (HaveFunPtrs)
3750:     Changed |= processFunctionPointers(M);
3751:
3752:   return Changed;
3753: }
3754:
3755: PreservedAnalyses
3756: llvm::SPIRVEmitIntrinsicsPass::run(Module &M, ModuleAnalysisManager &AM) {
3757:   SPIRVEmitIntrinsics Legacy(TM);
3758:   if (Legacy.runOnModule(M))
3759:     return PreservedAnalyses::none();
3760:   return PreservedAnalyses::all();
3761: }
3762:
3763: ModulePass *llvm::createSPIRVEmitIntrinsicsPass(const SPIRVTargetMachine &TM) {
3764:   return new SPIRVEmitIntrinsics(TM);
3765: }
```
- EN: This range implements operational logic in helpers such as SPIRVEmitIntrinsics::runOnModule, convertMaskedMemIntrinsics, parseFunDeclarations, insertConstantsForFPFastMathDefault, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVEmitIntrinsics::runOnModule、convertMaskedMemIntrinsics、parseFunDeclarations、insertConstantsForFPFastMathDefault 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Intrinsic handling connects target-specific builtins with LLVM IR or machine-level lowering paths.
  - CN: 内建函数处理负责把目标特定 builtin 与 LLVM IR 或机器级降级路径连接起来。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include cl::init, GlobalVariableUsers, pop_back_val, insert, append, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, GlobalVariableUsers, pop_back_val, insert, append，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVEmitIntrinsics.h`
  - `SPIRV.h`
  - `SPIRVBuiltins.h`
  - `SPIRVSubtarget.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseSet.h`
  - `llvm/ADT/StringSet.h`
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/InstVisitor.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/PatternMatch.h`
  - `llvm/IR/TypedPointerType.h`
  - `llvm/IR/Value.h`
  - `llvm/Support/CommandLine.h`
  - `llvm/Transforms/Utils/Local.h`
- System/standard headers / 系统或标准头文件:
  - `cassert`
  - `optional`
  - `queue`
  - `unordered_set`
  - `SPIRVGenTables.inc`
