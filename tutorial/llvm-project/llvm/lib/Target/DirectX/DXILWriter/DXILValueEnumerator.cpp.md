# DXILValueEnumerator.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILWriter/DXILValueEnumerator.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the ValueEnumerator class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===- ValueEnumerator.cpp - Number values and types for bitcode writer ---===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the ValueEnumerator class.
10: // Forked from lib/Bitcode/Writer
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "DXILValueEnumerator.h"
15: #include "DirectXIRPasses/DXILDebugInfo.h"
16: #include "llvm/ADT/SmallVector.h"
17: #include "llvm/Config/llvm-config.h"
18: #include "llvm/IR/Argument.h"
19: #include "llvm/IR/BasicBlock.h"
20: #include "llvm/IR/Constant.h"
21: #include "llvm/IR/DebugInfoMetadata.h"
22: #include "llvm/IR/DerivedTypes.h"
23: #include "llvm/IR/Function.h"
24: #include "llvm/IR/GlobalAlias.h"
25: #include "llvm/IR/GlobalIFunc.h"
26: #include "llvm/IR/GlobalObject.h"
27: #include "llvm/IR/GlobalValue.h"
28: #include "llvm/IR/GlobalVariable.h"
29: #include "llvm/IR/Instruction.h"
30: #include "llvm/IR/Instructions.h"
31: #include "llvm/IR/Metadata.h"
32: #include "llvm/IR/Module.h"
33: #include "llvm/IR/Operator.h"
34: #include "llvm/IR/Type.h"
35: #include "llvm/IR/TypedPointerType.h"
36: #include "llvm/IR/Use.h"
37: #include "llvm/IR/User.h"
38: #include "llvm/IR/Value.h"
39: #include "llvm/IR/ValueSymbolTable.h"
40: #include "llvm/Support/Casting.h"
41: #include "llvm/Support/Compiler.h"
42: #include "llvm/Support/Debug.h"
43: #include "llvm/Support/MathExtras.h"
44: #include "llvm/Support/raw_ostream.h"
45: #include <algorithm>
46: #include <cstddef>
47: #include <iterator>
48: #include <tuple>
49:
50: using namespace llvm;
51: using namespace llvm::dxil;
52:
53: namespace {
54:
55: struct OrderMap {
56:   DenseMap<const Value *, std::pair<unsigned, bool>> IDs;
57:   unsigned LastGlobalConstantID = 0;
58:   unsigned LastGlobalValueID = 0;
59:
60:   OrderMap() = default;
61:
62:   bool isGlobalConstant(unsigned ID) const {
63:     return ID <= LastGlobalConstantID;
64:   }
65:
66:   bool isGlobalValue(unsigned ID) const {
67:     return ID <= LastGlobalValueID && !isGlobalConstant(ID);
68:   }
69:
70:   unsigned size() const { return IDs.size(); }
71:   std::pair<unsigned, bool> &operator[](const Value *V) { return IDs[V]; }
72:
73:   std::pair<unsigned, bool> lookup(const Value *V) const {
74:     return IDs.lookup(V);
75:   }
76:
77:   void index(const Value *V) {
78:     // Explicitly sequence get-size and insert-value operations to avoid UB.
79:     unsigned ID = IDs.size() + 1;
80:     IDs[V].first = ID;
81:   }
82: };
83:
84: } // end anonymous namespace
85:
86: static void orderValue(const Value *V, OrderMap &OM) {
87:   if (OM.lookup(V).first)
88:     return;
89:
90:   if (const Constant *C = dyn_cast<Constant>(V)) {
```
- EN: This range defines or declares important types such as OrderMap, isGlobalConstant, isGlobalValue, size, shaping the data model used by DXILValueEnumerator.cpp.
- CN: 这一段定义或声明了 OrderMap、isGlobalConstant、isGlobalValue、size 等关键类型，构成 DXILValueEnumerator.cpp 使用的数据模型。

### Lines 91-180
```cpp
 91:     if (C->getNumOperands() && !isa<GlobalValue>(C)) {
 92:       for (const Value *Op : C->operands())
 93:         if (!isa<BasicBlock>(Op) && !isa<GlobalValue>(Op))
 94:           orderValue(Op, OM);
 95:       if (auto *CE = dyn_cast<ConstantExpr>(C))
 96:         if (CE->getOpcode() == Instruction::ShuffleVector)
 97:           orderValue(CE->getShuffleMaskForBitcode(), OM);
 98:     }
 99:   }
100:
101:   // Note: we cannot cache this lookup above, since inserting into the map
102:   // changes the map's size, and thus affects the other IDs.
103:   OM.index(V);
104: }
105:
106: static OrderMap orderModule(const Module &M) {
107:   // This needs to match the order used by ValueEnumerator::ValueEnumerator()
108:   // and ValueEnumerator::incorporateFunction().
109:   OrderMap OM;
110:
111:   // In the reader, initializers of GlobalValues are set *after* all the
112:   // globals have been read.  Rather than awkwardly modeling this behaviour
113:   // directly in predictValueUseListOrderImpl(), just assign IDs to
114:   // initializers of GlobalValues before GlobalValues themselves to model this
115:   // implicitly.
116:   for (const GlobalVariable &G : M.globals())
117:     if (G.hasInitializer())
118:       if (!isa<GlobalValue>(G.getInitializer()))
119:         orderValue(G.getInitializer(), OM);
120:   for (const GlobalAlias &A : M.aliases())
121:     if (!isa<GlobalValue>(A.getAliasee()))
122:       orderValue(A.getAliasee(), OM);
123:   for (const GlobalIFunc &I : M.ifuncs())
124:     if (!isa<GlobalValue>(I.getResolver()))
125:       orderValue(I.getResolver(), OM);
126:   for (const Function &F : M) {
127:     for (const Use &U : F.operands())
128:       if (!isa<GlobalValue>(U.get()))
129:         orderValue(U.get(), OM);
130:   }
131:
132:   // As constants used in metadata operands are emitted as module-level
133:   // constants, we must order them before other operands. Also, we must order
134:   // these before global values, as these will be read before setting the
135:   // global values' initializers. The latter matters for constants which have
136:   // uses towards other constants that are used as initializers.
137:   auto orderConstantValue = [&OM](const Value *V) {
138:     if ((isa<Constant>(V) && !isa<GlobalValue>(V)) || isa<InlineAsm>(V))
139:       orderValue(V, OM);
140:   };
141:   for (const Function &F : M) {
142:     if (F.isDeclaration())
143:       continue;
144:     for (const BasicBlock &BB : F)
145:       for (const Instruction &I : BB)
146:         for (const Value *V : I.operands()) {
147:           if (const auto *MAV = dyn_cast<MetadataAsValue>(V)) {
148:             if (const auto *VAM =
149:                     dyn_cast<ValueAsMetadata>(MAV->getMetadata())) {
150:               orderConstantValue(VAM->getValue());
151:             } else if (const auto *AL =
152:                            dyn_cast<DIArgList>(MAV->getMetadata())) {
153:               for (const auto *VAM : AL->getArgs())
154:                 orderConstantValue(VAM->getValue());
155:             }
156:           }
157:         }
158:   }
159:   OM.LastGlobalConstantID = OM.size();
160:
161:   // Initializers of GlobalValues are processed in
162:   // BitcodeReader::ResolveGlobalAndAliasInits().  Match the order there rather
163:   // than ValueEnumerator, and match the code in predictValueUseListOrderImpl()
164:   // by giving IDs in reverse order.
165:   //
166:   // Since GlobalValues never reference each other directly (just through
167:   // initializers), their relative IDs only matter for determining order of
168:   // uses in their initializers.
169:   for (const Function &F : M)
170:     orderValue(&F, OM);
171:   for (const GlobalAlias &A : M.aliases())
172:     orderValue(&A, OM);
173:   for (const GlobalIFunc &I : M.ifuncs())
174:     orderValue(&I, OM);
175:   for (const GlobalVariable &G : M.globals())
176:     orderValue(&G, OM);
177:   OM.LastGlobalValueID = OM.size();
178:
179:   for (const Function &F : M) {
180:     if (F.isDeclaration())
```
- EN: This range implements operational logic in helpers such as orderValue, index, orderModule, getMetadata, translating backend policy into executable code.
- CN: 这一段实现了 orderValue、index、orderModule、getMetadata 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:       continue;
182:     // Here we need to match the union of ValueEnumerator::incorporateFunction()
183:     // and WriteFunction().  Basic blocks are implicitly declared before
184:     // anything else (by declaring their size).
185:     for (const BasicBlock &BB : F)
186:       orderValue(&BB, OM);
187:     for (const Argument &A : F.args())
188:       orderValue(&A, OM);
189:     for (const BasicBlock &BB : F)
190:       for (const Instruction &I : BB) {
191:         for (const Value *Op : I.operands())
192:           if ((isa<Constant>(*Op) && !isa<GlobalValue>(*Op)) ||
193:               isa<InlineAsm>(*Op))
194:             orderValue(Op, OM);
195:         if (auto *SVI = dyn_cast<ShuffleVectorInst>(&I))
196:           orderValue(SVI->getShuffleMaskForBitcode(), OM);
197:         if (auto *SI = dyn_cast<SwitchInst>(&I)) {
198:           for (const auto &Case : SI->cases())
199:             orderValue(Case.getCaseValue(), OM);
200:         }
201:       }
202:     for (const BasicBlock &BB : F)
203:       for (const Instruction &I : BB)
204:         orderValue(&I, OM);
205:   }
206:   return OM;
207: }
208:
209: static void predictValueUseListOrderImpl(const Value *V, const Function *F,
210:                                          unsigned ID, const OrderMap &OM,
211:                                          UseListOrderStack &Stack) {
212:   // Predict use-list order for this one.
213:   using Entry = std::pair<const Use *, unsigned>;
214:   SmallVector<Entry, 64> List;
215:   for (const Use &U : V->uses())
216:     // Check if this user will be serialized.
217:     if (OM.lookup(U.getUser()).first)
218:       List.push_back(std::make_pair(&U, List.size()));
219:
220:   if (List.size() < 2)
221:     // We may have lost some users.
222:     return;
223:
224:   bool IsGlobalValue = OM.isGlobalValue(ID);
225:   llvm::sort(List, [&](const Entry &L, const Entry &R) {
226:     const Use *LU = L.first;
227:     const Use *RU = R.first;
228:     if (LU == RU)
229:       return false;
230:
231:     auto LID = OM.lookup(LU->getUser()).first;
232:     auto RID = OM.lookup(RU->getUser()).first;
233:
234:     // Global values are processed in reverse order.
235:     //
236:     // Moreover, initializers of GlobalValues are set *after* all the globals
237:     // have been read (despite having earlier IDs).  Rather than awkwardly
238:     // modeling this behaviour here, orderModule() has assigned IDs to
239:     // initializers of GlobalValues before GlobalValues themselves.
240:     if (OM.isGlobalValue(LID) && OM.isGlobalValue(RID)) {
241:       if (LID == RID)
242:         return LU->getOperandNo() > RU->getOperandNo();
243:       return LID < RID;
244:     }
245:
246:     // If ID is 4, then expect: 7 6 5 1 2 3.
247:     if (LID < RID) {
248:       if (RID <= ID)
249:         if (!IsGlobalValue) // GlobalValue uses don't get reversed.
250:           return true;
251:       return false;
252:     }
253:     if (RID < LID) {
254:       if (LID <= ID)
255:         if (!IsGlobalValue) // GlobalValue uses don't get reversed.
256:           return false;
257:       return true;
258:     }
259:
260:     // LID and RID are equal, so we have different operands of the same user.
261:     // Assume operands are added in order for all instructions.
262:     if (LID <= ID)
263:       if (!IsGlobalValue) // GlobalValue uses don't get reversed.
264:         return LU->getOperandNo() < RU->getOperandNo();
265:     return LU->getOperandNo() > RU->getOperandNo();
266:   });
267:
268:   if (llvm::is_sorted(List, llvm::less_second()))
269:     // Order is already correct.
270:     return;
```
- EN: This range implements operational logic in helpers such as orderValue, push_back, isGlobalValue, llvm::sort, translating backend policy into executable code.
- CN: 这一段实现了 orderValue、push_back、isGlobalValue、llvm::sort 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:
272:   // Store the shuffle.
273:   Stack.emplace_back(V, F, List.size());
274:   assert(List.size() == Stack.back().Shuffle.size() && "Wrong size");
275:   for (size_t I = 0, E = List.size(); I != E; ++I)
276:     Stack.back().Shuffle[I] = List[I].second;
277: }
278:
279: static void predictValueUseListOrder(const Value *V, const Function *F,
280:                                      OrderMap &OM, UseListOrderStack &Stack) {
281:   auto &IDPair = OM[V];
282:   assert(IDPair.first && "Unmapped value");
283:   if (IDPair.second)
284:     // Already predicted.
285:     return;
286:
287:   // Do the actual prediction.
288:   IDPair.second = true;
289:   if (!V->use_empty() && std::next(V->use_begin()) != V->use_end())
290:     predictValueUseListOrderImpl(V, F, IDPair.first, OM, Stack);
291:
292:   // Recursive descent into constants.
293:   if (const Constant *C = dyn_cast<Constant>(V)) {
294:     if (C->getNumOperands()) { // Visit GlobalValues.
295:       for (const Value *Op : C->operands())
296:         if (isa<Constant>(Op)) // Visit GlobalValues.
297:           predictValueUseListOrder(Op, F, OM, Stack);
298:       if (auto *CE = dyn_cast<ConstantExpr>(C))
299:         if (CE->getOpcode() == Instruction::ShuffleVector)
300:           predictValueUseListOrder(CE->getShuffleMaskForBitcode(), F, OM,
301:                                    Stack);
302:     }
303:   }
304: }
305:
306: static UseListOrderStack predictUseListOrder(const Module &M) {
307:   OrderMap OM = orderModule(M);
308:
309:   // Use-list orders need to be serialized after all the users have been added
310:   // to a value, or else the shuffles will be incomplete.  Store them per
311:   // function in a stack.
312:   //
313:   // Aside from function order, the order of values doesn't matter much here.
314:   UseListOrderStack Stack;
315:
316:   // We want to visit the functions backward now so we can list function-local
317:   // constants in the last Function they're used in.  Module-level constants
318:   // have already been visited above.
319:   for (const Function &F : llvm::reverse(M)) {
320:     if (F.isDeclaration())
321:       continue;
322:     for (const BasicBlock &BB : F)
323:       predictValueUseListOrder(&BB, &F, OM, Stack);
324:     for (const Argument &A : F.args())
325:       predictValueUseListOrder(&A, &F, OM, Stack);
326:     for (const BasicBlock &BB : F)
327:       for (const Instruction &I : BB) {
328:         for (const Value *Op : I.operands())
329:           if (isa<Constant>(*Op) || isa<InlineAsm>(*Op)) // Visit GlobalValues.
330:             predictValueUseListOrder(Op, &F, OM, Stack);
331:         if (auto *SVI = dyn_cast<ShuffleVectorInst>(&I))
332:           predictValueUseListOrder(SVI->getShuffleMaskForBitcode(), &F, OM,
333:                                    Stack);
334:       }
335:     for (const BasicBlock &BB : F)
336:       for (const Instruction &I : BB)
337:         predictValueUseListOrder(&I, &F, OM, Stack);
338:   }
339:
340:   // Visit globals last, since the module-level use-list block will be seen
341:   // before the function bodies are processed.
342:   for (const GlobalVariable &G : M.globals())
343:     predictValueUseListOrder(&G, nullptr, OM, Stack);
344:   for (const Function &F : M)
345:     predictValueUseListOrder(&F, nullptr, OM, Stack);
346:   for (const GlobalAlias &A : M.aliases())
347:     predictValueUseListOrder(&A, nullptr, OM, Stack);
348:   for (const GlobalIFunc &I : M.ifuncs())
349:     predictValueUseListOrder(&I, nullptr, OM, Stack);
350:   for (const GlobalVariable &G : M.globals())
351:     if (G.hasInitializer())
352:       predictValueUseListOrder(G.getInitializer(), nullptr, OM, Stack);
353:   for (const GlobalAlias &A : M.aliases())
354:     predictValueUseListOrder(A.getAliasee(), nullptr, OM, Stack);
355:   for (const GlobalIFunc &I : M.ifuncs())
356:     predictValueUseListOrder(I.getResolver(), nullptr, OM, Stack);
357:   for (const Function &F : M) {
358:     for (const Use &U : F.operands())
359:       predictValueUseListOrder(U.get(), nullptr, OM, Stack);
360:   }
```
- EN: This range implements operational logic in helpers such as emplace_back, assert, predictValueUseListOrderImpl, predictValueUseListOrder, translating backend policy into executable code.
- CN: 这一段实现了 emplace_back、assert、predictValueUseListOrderImpl、predictValueUseListOrder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:
362:   return Stack;
363: }
364:
365: ValueEnumerator::ValueEnumerator(const Module &M, Type *PrefixType,
366:                                  const DXILDebugInfoMap &DebugInfo)
367:     : DebugInfo(DebugInfo) {
368:   EnumerateType(PrefixType);
369:   
370:   UseListOrders = predictUseListOrder(M);
371:
372:   // Enumerate the global variables.
373:   for (const GlobalVariable &GV : M.globals()) {
374:     EnumerateValue(&GV);
375:     EnumerateType(GV.getValueType());
376:   }
377:
378:   // Enumerate the functions.
379:   for (const Function &F : M) {
380:     EnumerateValue(&F);
381:     EnumerateType(F.getFunctionType());
382:     EnumerateType(
383:         TypedPointerType::get(F.getFunctionType(), F.getAddressSpace()));
384:     EnumerateAttributes(F.getAttributes());
385:   }
386:
387:   // Enumerate the aliases.
388:   for (const GlobalAlias &GA : M.aliases()) {
389:     EnumerateValue(&GA);
390:     EnumerateType(GA.getValueType());
391:   }
392:
393:   // Enumerate the ifuncs.
394:   for (const GlobalIFunc &GIF : M.ifuncs()) {
395:     EnumerateValue(&GIF);
396:     EnumerateType(GIF.getValueType());
397:   }
398:
399:   // Enumerate the global variable initializers and attributes.
400:   for (const GlobalVariable &GV : M.globals()) {
401:     if (GV.hasInitializer())
402:       EnumerateValue(GV.getInitializer());
403:     EnumerateType(
404:         TypedPointerType::get(GV.getValueType(), GV.getAddressSpace()));
405:     if (GV.hasAttributes())
406:       EnumerateAttributes(GV.getAttributesAsList(AttributeList::FunctionIndex));
407:   }
408:
409:   // Enumerate the aliasees.
410:   for (const GlobalAlias &GA : M.aliases())
411:     EnumerateValue(GA.getAliasee());
412:
413:   // Enumerate the ifunc resolvers.
414:   for (const GlobalIFunc &GIF : M.ifuncs())
415:     EnumerateValue(GIF.getResolver());
416:
417:   // Enumerate any optional Function data.
418:   for (const Function &F : M)
419:     for (const Use &U : F.operands())
420:       EnumerateValue(U.get());
421:
422:   // Enumerate the metadata type.
423:   //
424:   // TODO: Move this to ValueEnumerator::EnumerateOperandType() once bitcode
425:   // only encodes the metadata type when it's used as a value.
426:   EnumerateType(Type::getMetadataTy(M.getContext()));
427:
428:   // Insert constants and metadata that are named at module level into the slot
429:   // pool so that the module symbol table can refer to them...
430:   EnumerateValueSymbolTable(M.getValueSymbolTable());
431:   EnumerateNamedMetadata(M);
432:
433:   SmallVector<std::pair<unsigned, MDNode *>, 8> MDs;
434:   for (const GlobalVariable &GV : M.globals()) {
435:     MDs.clear();
436:     GV.getAllMetadata(MDs);
437:     for (const auto &I : MDs)
438:       // FIXME: Pass GV to EnumerateMetadata and arrange for the bitcode writer
439:       // to write metadata to the global variable's own metadata block
440:       // (PR28134).
441:       EnumerateMetadata(nullptr, I.second);
442:   }
443:
444:   // Enumerate types used by function bodies and argument lists.
445:   for (const Function &F : M) {
446:     for (const Argument &A : F.args())
447:       EnumerateType(A.getType());
448:
449:     // Enumerate metadata attached to this function.
450:     MDs.clear();
```
- EN: This range implements operational logic in helpers such as DebugInfo, EnumerateType, predictUseListOrder, EnumerateValue, translating backend policy into executable code.
- CN: 这一段实现了 DebugInfo、EnumerateType、predictUseListOrder、EnumerateValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:     F.getAllMetadata(MDs);
452:     for (const auto &I : MDs)
453:       EnumerateMetadata(F.isDeclaration() ? nullptr : &F, I.second);
454:
455:     for (const BasicBlock &BB : F)
456:       for (const Instruction &I : BB) {
457:         for (const Use &Op : I.operands()) {
458:           auto *MD = dyn_cast<MetadataAsValue>(&Op);
459:           if (!MD) {
460:             EnumerateOperandType(Op);
461:             continue;
462:           }
463:
464:           // Local metadata is enumerated during function-incorporation, but
465:           // any ConstantAsMetadata arguments in a DIArgList should be examined
466:           // now.
467:           if (isa<LocalAsMetadata>(MD->getMetadata()))
468:             continue;
469:           if (auto *AL = dyn_cast<DIArgList>(MD->getMetadata())) {
470:             for (auto *VAM : AL->getArgs())
471:               if (isa<ConstantAsMetadata>(VAM))
472:                 EnumerateMetadata(&F, VAM);
473:             continue;
474:           }
475:
476:           EnumerateMetadata(&F, MD->getMetadata());
477:         }
478:         if (auto *SVI = dyn_cast<ShuffleVectorInst>(&I))
479:           EnumerateType(SVI->getShuffleMaskForBitcode()->getType());
480:         if (auto *GEP = dyn_cast<GetElementPtrInst>(&I))
481:           EnumerateType(GEP->getSourceElementType());
482:         if (auto *AI = dyn_cast<AllocaInst>(&I))
483:           EnumerateType(AI->getAllocatedType());
484:         EnumerateType(I.getType());
485:         if (const auto *Call = dyn_cast<CallBase>(&I)) {
486:           EnumerateAttributes(Call->getAttributes());
487:           EnumerateType(Call->getFunctionType());
488:         }
489:
490:         // Enumerate metadata attached with this instruction.
491:         MDs.clear();
492:         I.getAllMetadataOtherThanDebugLoc(MDs);
493:         for (unsigned i = 0, e = MDs.size(); i != e; ++i)
494:           EnumerateMetadata(&F, MDs[i].second);
495:
496:         // Don't enumerate the location directly -- it has a special record
497:         // type -- but enumerate its operands.
498:         if (DILocation *L = I.getDebugLoc())
499:           for (const Metadata *Op : L->operands())
500:             EnumerateMetadata(&F, Op);
501:       }
502:   }
503:
504:   // Organize metadata ordering.
505:   organizeMetadata();
506: }
507:
508: unsigned ValueEnumerator::getInstructionID(const Instruction *Inst) const {
509:   InstructionMapType::const_iterator I = InstructionMap.find(Inst);
510:   assert(I != InstructionMap.end() && "Instruction is not mapped!");
511:   return I->second;
512: }
513:
514: unsigned ValueEnumerator::getComdatID(const Comdat *C) const {
515:   unsigned ComdatID = Comdats.idFor(C);
516:   assert(ComdatID && "Comdat not found!");
517:   return ComdatID;
518: }
519:
520: void ValueEnumerator::setInstructionID(const Instruction *I) {
521:   InstructionMap[I] = InstructionCount++;
522: }
523:
524: unsigned ValueEnumerator::getValueID(const Value *V) const {
525:   if (auto *MD = dyn_cast<MetadataAsValue>(V))
526:     return getMetadataID(MD->getMetadata());
527:
528:   ValueMapType::const_iterator I = ValueMap.find(V);
529:   assert(I != ValueMap.end() && "Value not in slotcalculator!");
530:   return I->second - 1;
531: }
532:
533: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
534: LLVM_DUMP_METHOD void ValueEnumerator::dump() const {
535:   print(dbgs(), ValueMap, "Default");
536:   dbgs() << '\n';
537:   print(dbgs(), MetadataMap, "MetaData");
538:   dbgs() << '\n';
539: }
540: #endif
```
- EN: This range implements operational logic in helpers such as getAllMetadata, EnumerateMetadata, EnumerateOperandType, EnumerateType, translating backend policy into executable code.
- CN: 这一段实现了 getAllMetadata、EnumerateMetadata、EnumerateOperandType、EnumerateType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:
542: void ValueEnumerator::print(raw_ostream &OS, const ValueMapType &Map,
543:                             const char *Name) const {
544:   OS << "Map Name: " << Name << "\n";
545:   OS << "Size: " << Map.size() << "\n";
546:   for (const auto &I : Map) {
547:     const Value *V = I.first;
548:     if (V->hasName())
549:       OS << "Value: " << V->getName() << '\n';
550:     else
551:       OS << "Value: [null]\n";
552:     V->print(OS);
553:     OS << '\n';
554:
555:     if (V->hasUseList()) {
556:       OS << " Uses(" << V->getNumUses() << "):";
557:       for (const Use &U : V->uses()) {
558:         if (&U != &*V->use_begin())
559:           OS << ",";
560:         if (U->hasName())
561:           OS << " " << U->getName();
562:         else
563:           OS << " [null]";
564:       }
565:       OS << '\n';
566:     }
567:
568:     OS << '\n';
569:   }
570: }
571:
572: void ValueEnumerator::print(raw_ostream &OS, const MetadataMapType &Map,
573:                             const char *Name) const {
574:   OS << "Map Name: " << Name << "\n";
575:   OS << "Size: " << Map.size() << "\n";
576:   for (const auto &I : Map) {
577:     const Metadata *MD = I.first;
578:     OS << "Metadata: slot = " << I.second.ID << "\n";
579:     OS << "Metadata: function = " << I.second.F << "\n";
580:     MD->print(OS);
581:     OS << "\n";
582:   }
583: }
584:
585: /// EnumerateValueSymbolTable - Insert all of the values in the specified symbol
586: /// table into the values table.
587: void ValueEnumerator::EnumerateValueSymbolTable(const ValueSymbolTable &VST) {
588:   for (ValueSymbolTable::const_iterator VI = VST.begin(), VE = VST.end();
589:        VI != VE; ++VI)
590:     EnumerateValue(VI->getValue());
591: }
592:
593: /// Insert all of the values referenced by named metadata in the specified
594: /// module.
595: void ValueEnumerator::EnumerateNamedMetadata(const Module &M) {
596:   for (const auto &I : M.named_metadata())
597:     EnumerateNamedMDNode(&I);
598: }
599:
600: void ValueEnumerator::EnumerateNamedMDNode(const NamedMDNode *MD) {
601:   for (unsigned i = 0, e = MD->getNumOperands(); i != e; ++i)
602:     EnumerateMetadata(nullptr, MD->getOperand(i));
603: }
604:
605: unsigned ValueEnumerator::getMetadataFunctionID(const Function *F) const {
606:   return F ? getValueID(F) + 1 : 0;
607: }
608:
609: void ValueEnumerator::EnumerateMetadata(const Function *F, const Metadata *MD) {
610:   EnumerateMetadata(getMetadataFunctionID(F), MD);
611: }
612:
613: void ValueEnumerator::EnumerateFunctionLocalMetadata(
614:     const Function &F, const LocalAsMetadata *Local) {
615:   EnumerateFunctionLocalMetadata(getMetadataFunctionID(&F), Local);
616: }
617:
618: void ValueEnumerator::EnumerateFunctionLocalListMetadata(
619:     const Function &F, const DIArgList *ArgList) {
620:   EnumerateFunctionLocalListMetadata(getMetadataFunctionID(&F), ArgList);
621: }
622:
623: void ValueEnumerator::dropFunctionFromMetadata(
624:     MetadataMapType::value_type &FirstMD) {
625:   SmallVector<const MDNode *, 64> Worklist;
626:   auto push = [&Worklist](MetadataMapType::value_type &MD) {
627:     auto &Entry = MD.second;
628:
629:     // Nothing to do if this metadata isn't tagged.
630:     if (!Entry.F)
```
- EN: This range implements operational logic in helpers such as print, getName, ValueEnumerator::EnumerateValueSymbolTable, EnumerateValue, translating backend policy into executable code.
- CN: 这一段实现了 print、getName、ValueEnumerator::EnumerateValueSymbolTable、EnumerateValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:       return;
632:
633:     // Drop the function tag.
634:     Entry.F = 0;
635:
636:     // If this is has an ID and is an MDNode, then its operands have entries as
637:     // well.  We need to drop the function from them too.
638:     if (Entry.ID)
639:       if (auto *N = dyn_cast<MDNode>(MD.first))
640:         Worklist.push_back(N);
641:   };
642:   push(FirstMD);
643:   while (!Worklist.empty())
644:     for (const Metadata *Op : Worklist.pop_back_val()->operands()) {
645:       if (!Op)
646:         continue;
647:       auto MD = MetadataMap.find(Op);
648:       if (MD != MetadataMap.end())
649:         push(*MD);
650:     }
651: }
652:
653: void ValueEnumerator::EnumerateMetadata(unsigned F, const Metadata *MD) {
654:   MD = getDXILMetadata(MD);
655:
656:   // It's vital for reader efficiency that uniqued subgraphs are done in
657:   // post-order; it's expensive when their operands have forward references.
658:   // If a distinct node is referenced from a uniqued node, it'll be delayed
659:   // until the uniqued subgraph has been completely traversed.
660:   SmallVector<const MDNode *, 32> DelayedDistinctNodes;
661:
662:   // Start by enumerating MD, and then work through its transitive operands in
663:   // post-order.  This requires a depth-first search.
664:   SmallVector<std::pair<const MDNode *, MDNode::op_iterator>, 32> Worklist;
665:   if (const MDNode *N = enumerateMetadataImpl(F, MD))
666:     Worklist.push_back(std::make_pair(N, N->op_begin()));
667:
668:   while (!Worklist.empty()) {
669:     const MDNode *N = Worklist.back().first;
670:
671:     // Enumerate operands until we hit a new node.  We need to traverse these
672:     // nodes' operands before visiting the rest of N's operands.
673:     MDNode::op_iterator I = std::find_if(
674:         Worklist.back().second, N->op_end(),
675:         [&](const Metadata *MD) { return enumerateMetadataImpl(F, MD); });
676:     if (I != N->op_end()) {
677:       auto *Op = cast<MDNode>(getDXILMetadata(*I));
678:       Worklist.back().second = ++I;
679:
680:       // Delay traversing Op if it's a distinct node and N is uniqued.
681:       if (Op->isDistinct() && !N->isDistinct())
682:         DelayedDistinctNodes.push_back(Op);
683:       else
684:         Worklist.push_back(std::make_pair(Op, Op->op_begin()));
685:       continue;
686:     }
687:
688:     if (const Metadata *ExtraMD = DebugInfo.MDExtra.lookup(N)) {
689:       if (enumerateMetadataImpl(F, ExtraMD)) {
690:         if (const auto *ExtraN = dyn_cast<MDNode>(ExtraMD)) {
691:           Worklist.push_back(std::make_pair(ExtraN, ExtraN->op_begin()));
692:           continue;
693:         }
694:       }
695:     }
696:
697:     // All the operands have been visited.  Now assign an ID.
698:     Worklist.pop_back();
699:     MDs.push_back(N);
700:     MetadataMap[N].ID = MDs.size();
701:
702:     // Flush out any delayed distinct nodes; these are all the distinct nodes
703:     // that are leaves in last uniqued subgraph.
704:     if (Worklist.empty() || Worklist.back().first->isDistinct()) {
705:       for (const MDNode *N : DelayedDistinctNodes)
706:         Worklist.push_back(std::make_pair(N, N->op_begin()));
707:       DelayedDistinctNodes.clear();
708:     }
709:   }
710: }
711:
712: const MDNode *ValueEnumerator::enumerateMetadataImpl(unsigned F,
713:                                                      const Metadata *MD) {
714:   MD = getDXILMetadata(MD);
715:
716:   if (!MD)
717:     return nullptr;
718:
719:   assert(
720:       (isa<MDNode>(MD) || isa<MDString>(MD) || isa<ConstantAsMetadata>(MD)) &&
```
- EN: This range implements operational logic in helpers such as push_back, push, find, ValueEnumerator::EnumerateMetadata, translating backend policy into executable code.
- CN: 这一段实现了 push_back、push、find、ValueEnumerator::EnumerateMetadata 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:       "Invalid metadata kind");
722:
723:   auto Insertion = MetadataMap.insert(std::make_pair(MD, MDIndex(F)));
724:   MDIndex &Entry = Insertion.first->second;
725:   if (!Insertion.second) {
726:     // Already mapped.  If F doesn't match the function tag, drop it.
727:     if (Entry.hasDifferentFunction(F))
728:       dropFunctionFromMetadata(*Insertion.first);
729:     return nullptr;
730:   }
731:
732:   // Don't assign IDs to metadata nodes.
733:   if (auto *N = dyn_cast<MDNode>(MD))
734:     return N;
735:
736:   // Save the metadata.
737:   MDs.push_back(MD);
738:   Entry.ID = MDs.size();
739:
740:   // Enumerate the constant, if any.
741:   if (auto *C = dyn_cast<ConstantAsMetadata>(MD))
742:     EnumerateValue(C->getValue());
743:
744:   return nullptr;
745: }
746:
747: /// EnumerateFunctionLocalMetadata - Incorporate function-local metadata
748: /// information reachable from the metadata.
749: void ValueEnumerator::EnumerateFunctionLocalMetadata(
750:     unsigned F, const LocalAsMetadata *Local) {
751:   assert(F && "Expected a function");
752:
753:   // Check to see if it's already in!
754:   MDIndex &Index = MetadataMap[Local];
755:   if (Index.ID) {
756:     assert(Index.F == F && "Expected the same function");
757:     return;
758:   }
759:
760:   MDs.push_back(Local);
761:   Index.F = F;
762:   Index.ID = MDs.size();
763:
764:   EnumerateValue(Local->getValue());
765: }
766:
767: /// EnumerateFunctionLocalListMetadata - Incorporate function-local metadata
768: /// information reachable from the metadata.
769: void ValueEnumerator::EnumerateFunctionLocalListMetadata(
770:     unsigned F, const DIArgList *ArgList) {
771:   assert(F && "Expected a function");
772:
773:   // Check to see if it's already in!
774:   MDIndex &Index = MetadataMap[ArgList];
775:   if (Index.ID) {
776:     assert(Index.F == F && "Expected the same function");
777:     return;
778:   }
779:
780:   for (ValueAsMetadata *VAM : ArgList->getArgs()) {
781:     if (isa<LocalAsMetadata>(VAM)) {
782:       assert(MetadataMap.count(VAM) &&
783:              "LocalAsMetadata should be enumerated before DIArgList");
784:       assert(MetadataMap[VAM].F == F &&
785:              "Expected LocalAsMetadata in the same function");
786:     } else {
787:       assert(isa<ConstantAsMetadata>(VAM) &&
788:              "Expected LocalAsMetadata or ConstantAsMetadata");
789:       assert(ValueMap.count(VAM->getValue()) &&
790:              "Constant should be enumerated beforeDIArgList");
791:       EnumerateMetadata(F, VAM);
792:     }
793:   }
794:
795:   MDs.push_back(ArgList);
796:   Index.F = F;
797:   Index.ID = MDs.size();
798: }
799:
800: static unsigned getMetadataTypeOrder(const Metadata *MD) {
801:   // Strings are emitted in bulk and must come first.
802:   if (isa<MDString>(MD))
803:     return 0;
804:
805:   // ConstantAsMetadata doesn't reference anything.  We may as well shuffle it
806:   // to the front since we can detect it.
807:   auto *N = dyn_cast<MDNode>(MD);
808:   if (!N)
809:     return 1;
810:
```
- EN: This range implements operational logic in helpers such as insert, dropFunctionFromMetadata, push_back, size, translating backend policy into executable code.
- CN: 这一段实现了 insert、dropFunctionFromMetadata、push_back、size 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:   // The reader is fast forward references for distinct node operands, but slow
812:   // when uniqued operands are unresolved.
813:   return N->isDistinct() ? 2 : 3;
814: }
815:
816: void ValueEnumerator::organizeMetadata() {
817:   assert(MetadataMap.size() == MDs.size() &&
818:          "Metadata map and vector out of sync");
819:
820:   if (MDs.empty())
821:     return;
822:
823:   // Copy out the index information from MetadataMap in order to choose a new
824:   // order.
825:   SmallVector<MDIndex, 64> Order;
826:   Order.reserve(MetadataMap.size());
827:   for (const Metadata *MD : MDs)
828:     Order.push_back(MetadataMap.lookup(MD));
829:
830:   // Partition:
831:   //   - by function, then
832:   //   - by isa<MDString>
833:   // and then sort by the original/current ID.  Since the IDs are guaranteed to
834:   // be unique, the result of llvm::sort will be deterministic.  There's no need
835:   // for std::stable_sort.
836:   llvm::sort(Order, [this](MDIndex LHS, MDIndex RHS) {
837:     return std::make_tuple(LHS.F, getMetadataTypeOrder(LHS.get(MDs)), LHS.ID) <
838:            std::make_tuple(RHS.F, getMetadataTypeOrder(RHS.get(MDs)), RHS.ID);
839:   });
840:
841:   // Rebuild MDs, index the metadata ranges for each function in FunctionMDs,
842:   // and fix up MetadataMap.
843:   std::vector<const Metadata *> OldMDs;
844:   MDs.swap(OldMDs);
845:   MDs.reserve(OldMDs.size());
846:   for (unsigned I = 0, E = Order.size(); I != E && !Order[I].F; ++I) {
847:     auto *MD = Order[I].get(OldMDs);
848:     MDs.push_back(MD);
849:     MetadataMap[MD].ID = I + 1;
850:     if (isa<MDString>(MD))
851:       ++NumMDStrings;
852:   }
853:
854:   // Return early if there's nothing for the functions.
855:   if (MDs.size() == Order.size())
856:     return;
857:
858:   // Build the function metadata ranges.
859:   MDRange R;
860:   FunctionMDs.reserve(OldMDs.size());
861:   unsigned PrevF = 0;
862:   for (unsigned I = MDs.size(), E = Order.size(), ID = MDs.size(); I != E;
863:        ++I) {
864:     unsigned F = Order[I].F;
865:     if (!PrevF) {
866:       PrevF = F;
867:     } else if (PrevF != F) {
868:       R.Last = FunctionMDs.size();
869:       std::swap(R, FunctionMDInfo[PrevF]);
870:       R.First = FunctionMDs.size();
871:
872:       ID = MDs.size();
873:       PrevF = F;
874:     }
875:
876:     auto *MD = Order[I].get(OldMDs);
877:     FunctionMDs.push_back(MD);
878:     MetadataMap[MD].ID = ++ID;
879:     if (isa<MDString>(MD))
880:       ++R.NumStrings;
881:   }
882:   R.Last = FunctionMDs.size();
883:   FunctionMDInfo[PrevF] = R;
884: }
885:
886: const Metadata *ValueEnumerator::getDXILMetadata(const Metadata *M) const {
887:   if (const Metadata *Replace = DebugInfo.MDReplace.lookup(M))
888:     return Replace;
889:   return M;
890: }
891:
892: void ValueEnumerator::incorporateFunctionMetadata(const Function &F) {
893:   NumModuleMDs = MDs.size();
894:
895:   auto R = FunctionMDInfo.lookup(getValueID(&F) + 1);
896:   NumMDStrings = R.NumStrings;
897:   MDs.insert(MDs.end(), FunctionMDs.begin() + R.First,
898:              FunctionMDs.begin() + R.Last);
899: }
900:
```
- EN: This range implements operational logic in helpers such as ValueEnumerator::organizeMetadata, reserve, push_back, llvm::sort, translating backend policy into executable code.
- CN: 这一段实现了 ValueEnumerator::organizeMetadata、reserve、push_back、llvm::sort 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901: void ValueEnumerator::EnumerateValue(const Value *V) {
902:   assert(!V->getType()->isVoidTy() && "Can't insert void values!");
903:   assert(!isa<MetadataAsValue>(V) && "EnumerateValue doesn't handle Metadata!");
904:
905:   // Check to see if it's already in!
906:   unsigned &ValueID = ValueMap[V];
907:   if (ValueID) {
908:     // Increment use count.
909:     Values[ValueID - 1].second++;
910:     return;
911:   }
912:
913:   if (auto *GO = dyn_cast<GlobalObject>(V))
914:     if (const Comdat *C = GO->getComdat())
915:       Comdats.insert(C);
916:
917:   // Enumerate the type of this value.
918:   EnumerateType(V->getType());
919:
920:   if (const Constant *C = dyn_cast<Constant>(V)) {
921:     if (isa<GlobalValue>(C)) {
922:       // Initializers for globals are handled explicitly elsewhere.
923:     } else if (C->getNumOperands()) {
924:       // If a constant has operands, enumerate them.  This makes sure that if a
925:       // constant has uses (for example an array of const ints), that they are
926:       // inserted also.
927:
928:       // We prefer to enumerate them with values before we enumerate the user
929:       // itself.  This makes it more likely that we can avoid forward references
930:       // in the reader.  We know that there can be no cycles in the constants
931:       // graph that don't go through a global variable.
932:       for (User::const_op_iterator I = C->op_begin(), E = C->op_end(); I != E;
933:            ++I)
934:         if (!isa<BasicBlock>(*I)) // Don't enumerate BB operand to BlockAddress.
935:           EnumerateValue(*I);
936:       if (auto *CE = dyn_cast<ConstantExpr>(C)) {
937:         if (CE->getOpcode() == Instruction::ShuffleVector)
938:           EnumerateValue(CE->getShuffleMaskForBitcode());
939:         if (auto *GEP = dyn_cast<GEPOperator>(CE))
940:           EnumerateType(GEP->getSourceElementType());
941:       }
942:
943:       // Finally, add the value.  Doing this could make the ValueID reference be
944:       // dangling, don't reuse it.
945:       Values.push_back(std::make_pair(V, 1U));
946:       ValueMap[V] = Values.size();
947:       return;
948:     }
949:   }
950:
951:   // Add the value.
952:   Values.push_back(std::make_pair(V, 1U));
953:   ValueID = Values.size();
954: }
955:
956: void ValueEnumerator::EnumerateType(Type *Ty) {
957:   unsigned *TypeID = &TypeMap[Ty];
958:
959:   // We've already seen this type.
960:   if (*TypeID)
961:     return;
962:
963:   // If it is a non-anonymous struct, mark the type as being visited so that we
964:   // don't recursively visit it.  This is safe because we allow forward
965:   // references of these in the bitcode reader.
966:   if (StructType *STy = dyn_cast<StructType>(Ty))
967:     if (!STy->isLiteral())
968:       *TypeID = ~0U;
969:
970:   // Enumerate all of the subtypes before we enumerate this type.  This ensures
971:   // that the type will be enumerated in an order that can be directly built.
972:   for (Type *SubTy : Ty->subtypes())
973:     EnumerateType(SubTy);
974:
975:   // Refresh the TypeID pointer in case the table rehashed.
976:   TypeID = &TypeMap[Ty];
977:
978:   // Check to see if we got the pointer another way.  This can happen when
979:   // enumerating recursive types that hit the base case deeper than they start.
980:   //
981:   // If this is actually a struct that we are treating as forward ref'able,
982:   // then emit the definition now that all of its contents are available.
983:   if (*TypeID && *TypeID != ~0U)
984:     return;
985:
986:   // Add this type now that its contents are all happily enumerated.
987:   Types.push_back(Ty);
988:
989:   *TypeID = Types.size();
990: }
```
- EN: This range implements operational logic in helpers such as ValueEnumerator::EnumerateValue, assert, insert, EnumerateType, translating backend policy into executable code.
- CN: 这一段实现了 ValueEnumerator::EnumerateValue、assert、insert、EnumerateType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:
 992: // Enumerate the types for the specified value.  If the value is a constant,
 993: // walk through it, enumerating the types of the constant.
 994: void ValueEnumerator::EnumerateOperandType(const Value *V) {
 995:   EnumerateType(V->getType());
 996:
 997:   assert(!isa<MetadataAsValue>(V) && "Unexpected metadata operand");
 998:
 999:   const Constant *C = dyn_cast<Constant>(V);
1000:   if (!C)
1001:     return;
1002:
1003:   // If this constant is already enumerated, ignore it, we know its type must
1004:   // be enumerated.
1005:   if (ValueMap.count(C))
1006:     return;
1007:
1008:   // This constant may have operands, make sure to enumerate the types in
1009:   // them.
1010:   for (const Value *Op : C->operands()) {
1011:     // Don't enumerate basic blocks here, this happens as operands to
1012:     // blockaddress.
1013:     if (isa<BasicBlock>(Op))
1014:       continue;
1015:
1016:     EnumerateOperandType(Op);
1017:   }
1018:   if (auto *CE = dyn_cast<ConstantExpr>(C)) {
1019:     if (CE->getOpcode() == Instruction::ShuffleVector)
1020:       EnumerateOperandType(CE->getShuffleMaskForBitcode());
1021:     if (CE->getOpcode() == Instruction::GetElementPtr)
1022:       EnumerateType(cast<GEPOperator>(CE)->getSourceElementType());
1023:   }
1024: }
1025:
1026: void ValueEnumerator::EnumerateAttributes(AttributeList PAL) {
1027:   if (PAL.isEmpty())
1028:     return; // null is always 0.
1029:
1030:   // Do a lookup.
1031:   unsigned &Entry = AttributeListMap[PAL];
1032:   if (Entry == 0) {
1033:     // Never saw this before, add it.
1034:     AttributeLists.push_back(PAL);
1035:     Entry = AttributeLists.size();
1036:   }
1037:
1038:   // Do lookups for all attribute groups.
1039:   for (unsigned i : PAL.indexes()) {
1040:     AttributeSet AS = PAL.getAttributes(i);
1041:     if (!AS.hasAttributes())
1042:       continue;
1043:     IndexAndAttrSet Pair = {i, AS};
1044:     unsigned &Entry = AttributeGroupMap[Pair];
1045:     if (Entry == 0) {
1046:       AttributeGroups.push_back(Pair);
1047:       Entry = AttributeGroups.size();
1048:
1049:       for (Attribute Attr : AS) {
1050:         if (Attr.isTypeAttribute())
1051:           EnumerateType(Attr.getValueAsType());
1052:       }
1053:     }
1054:   }
1055: }
1056:
1057: void ValueEnumerator::incorporateFunction(const Function &F) {
1058:   InstructionCount = 0;
1059:   NumModuleValues = Values.size();
1060:
1061:   // Add global metadata to the function block.  This doesn't include
1062:   // LocalAsMetadata.
1063:   incorporateFunctionMetadata(F);
1064:
1065:   // Adding function arguments to the value table.
1066:   for (const auto &I : F.args()) {
1067:     EnumerateValue(&I);
1068:     if (I.hasAttribute(Attribute::ByVal))
1069:       EnumerateType(I.getParamByValType());
1070:     else if (I.hasAttribute(Attribute::StructRet))
1071:       EnumerateType(I.getParamStructRetType());
1072:     else if (I.hasAttribute(Attribute::ByRef))
1073:       EnumerateType(I.getParamByRefType());
1074:   }
1075:   FirstFuncConstantID = Values.size();
1076:
1077:   // Add all function-level constants to the value table.
1078:   for (const BasicBlock &BB : F) {
1079:     for (const Instruction &I : BB) {
1080:       for (const Use &OI : I.operands()) {
```
- EN: This range implements operational logic in helpers such as ValueEnumerator::EnumerateOperandType, EnumerateType, assert, EnumerateOperandType, translating backend policy into executable code.
- CN: 这一段实现了 ValueEnumerator::EnumerateOperandType、EnumerateType、assert、EnumerateOperandType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:         if ((isa<Constant>(OI) && !isa<GlobalValue>(OI)) || isa<InlineAsm>(OI))
1082:           EnumerateValue(OI);
1083:       }
1084:       if (auto *SVI = dyn_cast<ShuffleVectorInst>(&I))
1085:         EnumerateValue(SVI->getShuffleMaskForBitcode());
1086:       if (auto *SI = dyn_cast<SwitchInst>(&I)) {
1087:         for (const auto &Case : SI->cases())
1088:           EnumerateValue(Case.getCaseValue());
1089:       }
1090:     }
1091:     BasicBlocks.push_back(&BB);
1092:     ValueMap[&BB] = BasicBlocks.size();
1093:   }
1094:
1095:   // Add the function's parameter attributes so they are available for use in
1096:   // the function's instruction.
1097:   EnumerateAttributes(F.getAttributes());
1098:
1099:   FirstInstID = Values.size();
1100:
1101:   SmallVector<LocalAsMetadata *, 8> FnLocalMDVector;
1102:   SmallVector<DIArgList *, 8> ArgListMDVector;
1103:   // Add all of the instructions.
1104:   for (const BasicBlock &BB : F) {
1105:     for (const Instruction &I : BB) {
1106:       for (const Use &OI : I.operands()) {
1107:         if (auto *MD = dyn_cast<MetadataAsValue>(&OI)) {
1108:           if (auto *Local = dyn_cast<LocalAsMetadata>(MD->getMetadata())) {
1109:             // Enumerate metadata after the instructions they might refer to.
1110:             FnLocalMDVector.push_back(Local);
1111:           } else if (auto *ArgList = dyn_cast<DIArgList>(MD->getMetadata())) {
1112:             ArgListMDVector.push_back(ArgList);
1113:             for (ValueAsMetadata *VMD : ArgList->getArgs()) {
1114:               if (auto *Local = dyn_cast<LocalAsMetadata>(VMD)) {
1115:                 // Enumerate metadata after the instructions they might refer
1116:                 // to.
1117:                 FnLocalMDVector.push_back(Local);
1118:               }
1119:             }
1120:           }
1121:         }
1122:       }
1123:
1124:       if (!I.getType()->isVoidTy())
1125:         EnumerateValue(&I);
1126:     }
1127:   }
1128:
1129:   // Add all of the function-local metadata.
1130:   for (unsigned i = 0, e = FnLocalMDVector.size(); i != e; ++i) {
1131:     // At this point, every local values have been incorporated, we shouldn't
1132:     // have a metadata operand that references a value that hasn't been seen.
1133:     assert(ValueMap.count(FnLocalMDVector[i]->getValue()) &&
1134:            "Missing value for metadata operand");
1135:     EnumerateFunctionLocalMetadata(F, FnLocalMDVector[i]);
1136:   }
1137:   // DIArgList entries must come after function-local metadata, as it is not
1138:   // possible to forward-reference them.
1139:   for (const DIArgList *ArgList : ArgListMDVector)
1140:     EnumerateFunctionLocalListMetadata(F, ArgList);
1141: }
1142:
1143: void ValueEnumerator::purgeFunction() {
1144:   /// Remove purged values from the ValueMap.
1145:   for (unsigned i = NumModuleValues, e = Values.size(); i != e; ++i)
1146:     ValueMap.erase(Values[i].first);
1147:   for (unsigned i = NumModuleMDs, e = MDs.size(); i != e; ++i)
1148:     MetadataMap.erase(MDs[i]);
1149:   for (const BasicBlock *BB : BasicBlocks)
1150:     ValueMap.erase(BB);
1151:
1152:   Values.resize(NumModuleValues);
1153:   MDs.resize(NumModuleMDs);
1154:   BasicBlocks.clear();
1155:   NumMDStrings = 0;
1156: }
1157:
1158: static void IncorporateFunctionInfoGlobalBBIDs(
1159:     const Function *F, DenseMap<const BasicBlock *, unsigned> &IDMap) {
1160:   unsigned Counter = 0;
1161:   for (const BasicBlock &BB : *F)
1162:     IDMap[&BB] = ++Counter;
1163: }
1164:
1165: /// getGlobalBasicBlockID - This returns the function-specific ID for the
1166: /// specified basic block.  This is relatively expensive information, so it
1167: /// should only be used by rare constructs such as address-of-label.
1168: unsigned ValueEnumerator::getGlobalBasicBlockID(const BasicBlock *BB) const {
1169:   unsigned &Idx = GlobalBasicBlockIDs[BB];
1170:   if (Idx != 0)
```
- EN: This range implements operational logic in helpers such as EnumerateValue, push_back, size, EnumerateAttributes, translating backend policy into executable code.
- CN: 这一段实现了 EnumerateValue、push_back、size、EnumerateAttributes 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1179
```cpp
1171:     return Idx - 1;
1172:
1173:   IncorporateFunctionInfoGlobalBBIDs(BB->getParent(), GlobalBasicBlockIDs);
1174:   return getGlobalBasicBlockID(BB);
1175: }
1176:
1177: uint64_t ValueEnumerator::computeBitsRequiredForTypeIndices() const {
1178:   return Log2_32_Ceil(getTypes().size() + 1);
1179: }
```
- EN: This range implements operational logic in helpers such as IncorporateFunctionInfoGlobalBBIDs, getGlobalBasicBlockID, ValueEnumerator::computeBitsRequiredForTypeIndices, Log2_32_Ceil, translating backend policy into executable code.
- CN: 这一段实现了 IncorporateFunctionInfoGlobalBBIDs、getGlobalBasicBlockID、ValueEnumerator::computeBitsRequiredForTypeIndices、Log2_32_Ceil 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include OrderMap, isGlobalConstant, isGlobalValue, size, lookup, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 OrderMap, isGlobalConstant, isGlobalValue, size, lookup，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILValueEnumerator.h`
  - `DirectXIRPasses/DXILDebugInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/Config/llvm-config.h`
  - `llvm/IR/Argument.h`
  - `llvm/IR/BasicBlock.h`
  - `llvm/IR/Constant.h`
  - `llvm/IR/DebugInfoMetadata.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/GlobalAlias.h`
  - `llvm/IR/GlobalIFunc.h`
  - `llvm/IR/GlobalObject.h`
  - `llvm/IR/GlobalValue.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Metadata.h`
- System/standard headers / 系统或标准头文件:
  - `algorithm`
  - `cstddef`
  - `iterator`
  - `tuple`
