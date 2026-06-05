# SPIRVStructurizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVStructurizer.cpp`
- Repository: `llvm-project`
- Purpose (EN): SPIRVStructurizer support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===-- SPIRVStructurizer.cpp ----------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //===----------------------------------------------------------------------===//
10:
11: #include "Analysis/SPIRVConvergenceRegionAnalysis.h"
12: #include "SPIRV.h"
13: #include "SPIRVStructurizerWrapper.h"
14: #include "SPIRVSubtarget.h"
15: #include "SPIRVUtils.h"
16: #include "llvm/ADT/DenseMap.h"
17: #include "llvm/ADT/SmallPtrSet.h"
18: #include "llvm/Analysis/LoopInfo.h"
19: #include "llvm/IR/CFG.h"
20: #include "llvm/IR/Dominators.h"
21: #include "llvm/IR/IRBuilder.h"
22: #include "llvm/IR/IntrinsicInst.h"
23: #include "llvm/IR/Intrinsics.h"
24: #include "llvm/IR/IntrinsicsSPIRV.h"
25: #include "llvm/IR/LegacyPassManager.h"
26: #include "llvm/InitializePasses.h"
27: #include "llvm/Transforms/Utils.h"
28: #include "llvm/Transforms/Utils/Cloning.h"
29: #include "llvm/Transforms/Utils/LoopSimplify.h"
30: #include "llvm/Transforms/Utils/LowerMemIntrinsics.h"
31: #include <stack>
32: #include <unordered_set>
33:
34: using namespace llvm;
35: using namespace SPIRV;
36:
37: using BlockSet = std::unordered_set<BasicBlock *>;
38: using Edge = std::pair<BasicBlock *, BasicBlock *>;
39:
40: // Helper function to do a partial order visit from the block |Start|, calling
41: // |Op| on each visited node.
42: static void partialOrderVisit(BasicBlock &Start,
43:                               std::function<bool(BasicBlock *)> Op) {
44:   PartialOrderingVisitor V(*Start.getParent());
45:   V.partialOrderVisit(Start, std::move(Op));
46: }
47:
48: // Returns the exact convergence region in the tree defined by `Node` for which
49: // `BB` is the header, nullptr otherwise.
50: static const ConvergenceRegion *
51: getRegionForHeader(const ConvergenceRegion *Node, BasicBlock *BB) {
52:   if (Node->Entry == BB)
53:     return Node;
54:
55:   for (auto *Child : Node->Children) {
56:     const auto *CR = getRegionForHeader(Child, BB);
57:     if (CR != nullptr)
58:       return CR;
59:   }
60:   return nullptr;
61: }
62:
63: // Returns the single BasicBlock exiting the convergence region `CR`,
64: // nullptr if no such exit exists.
65: static BasicBlock *getExitFor(const ConvergenceRegion *CR) {
66:   std::unordered_set<BasicBlock *> ExitTargets;
67:   for (BasicBlock *Exit : CR->Exits) {
68:     for (BasicBlock *Successor : successors(Exit)) {
69:       if (CR->Blocks.count(Successor) == 0)
70:         ExitTargets.insert(Successor);
71:     }
72:   }
73:
74:   assert(ExitTargets.size() <= 1);
75:   if (ExitTargets.size() == 0)
76:     return nullptr;
77:
78:   return *ExitTargets.begin();
79: }
80:
81: // Returns the merge block designated by I if I is a merge instruction, nullptr
82: // otherwise.
83: static BasicBlock *getDesignatedMergeBlock(Instruction *I) {
84:   IntrinsicInst *II = dyn_cast_or_null<IntrinsicInst>(I);
85:   if (II == nullptr)
86:     return nullptr;
87:
88:   if (II->getIntrinsicID() != Intrinsic::spv_loop_merge &&
89:       II->getIntrinsicID() != Intrinsic::spv_selection_merge)
90:     return nullptr;
```
- EN: This range implements operational logic in helpers such as bool, V, partialOrderVisit, getRegionForHeader, translating backend policy into executable code.
- CN: 这一段实现了 bool、V、partialOrderVisit、getRegionForHeader 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 91-180
```cpp
 91:
 92:   BlockAddress *BA = cast<BlockAddress>(II->getOperand(0));
 93:   return BA->getBasicBlock();
 94: }
 95:
 96: // Returns the continue block designated by I if I is an OpLoopMerge, nullptr
 97: // otherwise.
 98: static BasicBlock *getDesignatedContinueBlock(Instruction *I) {
 99:   IntrinsicInst *II = dyn_cast_or_null<IntrinsicInst>(I);
100:   if (II == nullptr)
101:     return nullptr;
102:
103:   if (II->getIntrinsicID() != Intrinsic::spv_loop_merge)
104:     return nullptr;
105:
106:   BlockAddress *BA = cast<BlockAddress>(II->getOperand(1));
107:   return BA->getBasicBlock();
108: }
109:
110: // Returns true if Header has one merge instruction which designated Merge as
111: // merge block.
112: static bool isDefinedAsSelectionMergeBy(BasicBlock &Header, BasicBlock &Merge) {
113:   for (auto &I : Header) {
114:     BasicBlock *MB = getDesignatedMergeBlock(&I);
115:     if (MB == &Merge)
116:       return true;
117:   }
118:   return false;
119: }
120:
121: // Returns true if the BB has one OpLoopMerge instruction.
122: static bool hasLoopMergeInstruction(BasicBlock &BB) {
123:   for (auto &I : BB)
124:     if (getDesignatedContinueBlock(&I))
125:       return true;
126:   return false;
127: }
128:
129: // Returns true is I is an OpSelectionMerge or OpLoopMerge instruction, false
130: // otherwise.
131: static bool isMergeInstruction(Instruction *I) {
132:   return getDesignatedMergeBlock(I) != nullptr;
133: }
134:
135: // Returns all blocks in F having at least one OpLoopMerge or OpSelectionMerge
136: // instruction.
137: static SmallPtrSet<BasicBlock *, 2> getHeaderBlocks(Function &F) {
138:   SmallPtrSet<BasicBlock *, 2> Output;
139:   for (BasicBlock &BB : F) {
140:     for (Instruction &I : BB) {
141:       if (getDesignatedMergeBlock(&I) != nullptr)
142:         Output.insert(&BB);
143:     }
144:   }
145:   return Output;
146: }
147:
148: // Returns all basic blocks in |F| referenced by at least 1
149: // OpSelectionMerge/OpLoopMerge instruction.
150: static SmallPtrSet<BasicBlock *, 2> getMergeBlocks(Function &F) {
151:   SmallPtrSet<BasicBlock *, 2> Output;
152:   for (BasicBlock &BB : F) {
153:     for (Instruction &I : BB) {
154:       BasicBlock *MB = getDesignatedMergeBlock(&I);
155:       if (MB != nullptr)
156:         Output.insert(MB);
157:     }
158:   }
159:   return Output;
160: }
161:
162: // Return all the merge instructions contained in BB.
163: // Note: the SPIR-V spec doesn't allow a single BB to contain more than 1 merge
164: // instruction, but this can happen while we structurize the CFG.
165: static std::vector<Instruction *> getMergeInstructions(BasicBlock &BB) {
166:   std::vector<Instruction *> Output;
167:   for (Instruction &I : BB)
168:     if (isMergeInstruction(&I))
169:       Output.push_back(&I);
170:   return Output;
171: }
172:
173: // Returns all basic blocks in |F| referenced as continue target by at least 1
174: // OpLoopMerge instruction.
175: static SmallPtrSet<BasicBlock *, 2> getContinueBlocks(Function &F) {
176:   SmallPtrSet<BasicBlock *, 2> Output;
177:   for (BasicBlock &BB : F) {
178:     for (Instruction &I : BB) {
179:       BasicBlock *MB = getDesignatedContinueBlock(&I);
180:       if (MB != nullptr)
```
- EN: This range implements operational logic in helpers such as getOperand, getBasicBlock, getDesignatedContinueBlock, isDefinedAsSelectionMergeBy, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getBasicBlock、getDesignatedContinueBlock、isDefinedAsSelectionMergeBy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:         Output.insert(MB);
182:     }
183:   }
184:   return Output;
185: }
186:
187: // Do a preorder traversal of the CFG starting from the BB |Start|.
188: // point. Calls |op| on each basic block encountered during the traversal.
189: static void visit(BasicBlock &Start, std::function<bool(BasicBlock *)> op) {
190:   std::stack<BasicBlock *> ToVisit;
191:   SmallPtrSet<BasicBlock *, 8> Seen;
192:
193:   ToVisit.push(&Start);
194:   Seen.insert(ToVisit.top());
195:   while (ToVisit.size() != 0) {
196:     BasicBlock *BB = ToVisit.top();
197:     ToVisit.pop();
198:
199:     if (!op(BB))
200:       continue;
201:
202:     for (auto Succ : successors(BB)) {
203:       if (Seen.contains(Succ))
204:         continue;
205:       ToVisit.push(Succ);
206:       Seen.insert(Succ);
207:     }
208:   }
209: }
210:
211: // Replaces the conditional and unconditional branch targets of |BB| by
212: // |NewTarget| if the target was |OldTarget|. This function also makes sure the
213: // associated merge instruction gets updated accordingly.
214: static void replaceIfBranchTargets(BasicBlock *BB, BasicBlock *OldTarget,
215:                                    BasicBlock *NewTarget) {
216:   auto *BI = cast<CondBrInst>(BB->getTerminator());
217:
218:   // 1. Replace all matching successors.
219:   for (size_t i = 0; i < BI->getNumSuccessors(); i++) {
220:     if (BI->getSuccessor(i) == OldTarget)
221:       BI->setSuccessor(i, NewTarget);
222:   }
223:
224:   // Branch had 2 successors, maybe now both are the same?
225:   if (BI->getSuccessor(0) != BI->getSuccessor(1))
226:     return;
227:
228:   // Note: we may end up here because the original IR had such branches.
229:   // This means Target is not necessarily equal to NewTarget.
230:   IRBuilder<> Builder(BB);
231:   Builder.SetInsertPoint(BI);
232:   Builder.CreateBr(BI->getSuccessor(0));
233:   BI->eraseFromParent();
234:
235:   // The branch was the only instruction, nothing else to do.
236:   if (BB->size() == 1)
237:     return;
238:
239:   // Otherwise, we need to check: was there an OpSelectionMerge before this
240:   // branch? If we removed the OpBranchConditional, we must also remove the
241:   // OpSelectionMerge. This is not valid for OpLoopMerge:
242:   IntrinsicInst *II =
243:       dyn_cast<IntrinsicInst>(BB->getTerminator()->getPrevNode());
244:   if (!II || II->getIntrinsicID() != Intrinsic::spv_selection_merge)
245:     return;
246:
247:   Constant *C = cast<Constant>(II->getOperand(0));
248:   II->eraseFromParent();
249:   if (!C->isConstantUsed())
250:     C->destroyConstant();
251: }
252:
253: // Replaces the target of branch instruction in |BB| with |NewTarget| if it
254: // was |OldTarget|. This function also fixes the associated merge instruction.
255: // Note: this function does not simplify branching instructions, it only updates
256: // targets. See also: simplifyBranches.
257: static void replaceBranchTargets(BasicBlock *BB, BasicBlock *OldTarget,
258:                                  BasicBlock *NewTarget) {
259:   auto *T = BB->getTerminator();
260:   if (isa<ReturnInst>(T))
261:     return;
262:   if (auto *BI = dyn_cast<UncondBrInst>(T)) {
263:     if (BI->getSuccessor() == OldTarget)
264:       BI->setSuccessor(NewTarget);
265:     return;
266:   }
267:
268:   if (isa<CondBrInst>(T))
269:     return replaceIfBranchTargets(BB, OldTarget, NewTarget);
270:
```
- EN: This range implements operational logic in helpers such as insert, visit, push, top, translating backend policy into executable code.
- CN: 这一段实现了 insert、visit、push、top 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:   if (auto *SI = dyn_cast<SwitchInst>(T)) {
272:     for (size_t i = 0; i < SI->getNumSuccessors(); i++) {
273:       if (SI->getSuccessor(i) == OldTarget)
274:         SI->setSuccessor(i, NewTarget);
275:     }
276:     return;
277:   }
278:
279:   assert(false && "Unhandled terminator type.");
280: }
281:
282: namespace {
283: // Given a reducible CFG, produces a structurized CFG in the SPIR-V sense,
284: // adding merge instructions when required.
285: class SPIRVStructurizer : public FunctionPass {
286:   struct DivergentConstruct;
287:   // Represents a list of condition/loops/switch constructs.
288:   // See SPIR-V 2.11.2. Structured Control-flow Constructs for the list of
289:   // constructs.
290:   using ConstructList = std::vector<std::unique_ptr<DivergentConstruct>>;
291:
292:   // Represents a divergent construct in the SPIR-V sense.
293:   // Such constructs are represented by a header (entry), a merge block (exit),
294:   // and possibly a continue block (back-edge). A construct can contain other
295:   // constructs, but their boundaries do not cross.
296:   struct DivergentConstruct {
297:     BasicBlock *Header = nullptr;
298:     BasicBlock *Merge = nullptr;
299:     BasicBlock *Continue = nullptr;
300:
301:     DivergentConstruct *Parent = nullptr;
302:     ConstructList Children;
303:   };
304:
305:   // An helper class to clean the construct boundaries.
306:   // It is used to gather the list of blocks that should belong to each
307:   // divergent construct, and possibly modify CFG edges when exits would cross
308:   // the boundary of multiple constructs.
309:   struct Splitter {
310:     Function &F;
311:     LoopInfo &LI;
312:     DomTreeBuilder::BBDomTree DT;
313:     DomTreeBuilder::BBPostDomTree PDT;
314:
315:     Splitter(Function &F, LoopInfo &LI) : F(F), LI(LI) { invalidate(); }
316:
317:     void invalidate() {
318:       PDT.recalculate(F);
319:       DT.recalculate(F);
320:     }
321:
322:     // Returns the list of blocks that belong to a SPIR-V loop construct,
323:     // including the continue construct.
324:     std::vector<BasicBlock *> getLoopConstructBlocks(BasicBlock *Header,
325:                                                      BasicBlock *Merge) {
326:       assert(DT.dominates(Header, Merge));
327:       std::vector<BasicBlock *> Output;
328:       partialOrderVisit(*Header, [&](BasicBlock *BB) {
329:         if (BB == Merge)
330:           return false;
331:         if (DT.dominates(Merge, BB) || !DT.dominates(Header, BB))
332:           return false;
333:         Output.push_back(BB);
334:         return true;
335:       });
336:       return Output;
337:     }
338:
339:     // Returns the list of blocks that belong to a SPIR-V selection construct.
340:     std::vector<BasicBlock *>
341:     getSelectionConstructBlocks(DivergentConstruct *Node) {
342:       assert(DT.dominates(Node->Header, Node->Merge));
343:       BlockSet OutsideBlocks;
344:       OutsideBlocks.insert(Node->Merge);
345:
346:       for (DivergentConstruct *It = Node->Parent; It != nullptr;
347:            It = It->Parent) {
348:         OutsideBlocks.insert(It->Merge);
349:         if (It->Continue)
350:           OutsideBlocks.insert(It->Continue);
351:       }
352:
353:       std::vector<BasicBlock *> Output;
354:       partialOrderVisit(*Node->Header, [&](BasicBlock *BB) {
355:         if (OutsideBlocks.count(BB) != 0)
356:           return false;
357:         if (DT.dominates(Node->Merge, BB) || !DT.dominates(Node->Header, BB))
358:           return false;
359:         Output.push_back(BB);
360:         return true;
```
- EN: This range defines or declares important types such as getNumSuccessors, setSuccessor, assert, SPIRVStructurizer, shaping the data model used by SPIRVStructurizer.cpp.
- CN: 这一段定义或声明了 getNumSuccessors、setSuccessor、assert、SPIRVStructurizer 等关键类型，构成 SPIRVStructurizer.cpp 使用的数据模型。

### Lines 361-450
```cpp
361:       });
362:       return Output;
363:     }
364:
365:     // Returns the list of blocks that belong to a SPIR-V switch construct.
366:     std::vector<BasicBlock *> getSwitchConstructBlocks(BasicBlock *Header,
367:                                                        BasicBlock *Merge) {
368:       assert(DT.dominates(Header, Merge));
369:
370:       std::vector<BasicBlock *> Output;
371:       partialOrderVisit(*Header, [&](BasicBlock *BB) {
372:         // the blocks structurally dominated by a switch header,
373:         if (!DT.dominates(Header, BB))
374:           return false;
375:         // excluding blocks structurally dominated by the switch header’s merge
376:         // block.
377:         if (DT.dominates(Merge, BB) || BB == Merge)
378:           return false;
379:         Output.push_back(BB);
380:         return true;
381:       });
382:       return Output;
383:     }
384:
385:     // Returns the list of blocks that belong to a SPIR-V case construct.
386:     std::vector<BasicBlock *> getCaseConstructBlocks(BasicBlock *Target,
387:                                                      BasicBlock *Merge) {
388:       assert(DT.dominates(Target, Merge));
389:
390:       std::vector<BasicBlock *> Output;
391:       partialOrderVisit(*Target, [&](BasicBlock *BB) {
392:         // the blocks structurally dominated by an OpSwitch Target or Default
393:         // block
394:         if (!DT.dominates(Target, BB))
395:           return false;
396:         // excluding the blocks structurally dominated by the OpSwitch
397:         // construct’s corresponding merge block.
398:         if (DT.dominates(Merge, BB) || BB == Merge)
399:           return false;
400:         Output.push_back(BB);
401:         return true;
402:       });
403:       return Output;
404:     }
405:
406:     // Splits the given edges by recreating proxy nodes so that the destination
407:     // has unique incoming edges from this region.
408:     //
409:     // clang-format off
410:     //
411:     // In SPIR-V, constructs must have a single exit/merge.
412:     // Given nodes A and B in the construct, a node C outside, and the following edges.
413:     //  A -> C
414:     //  B -> C
415:     //
416:     // In such cases, we must create a new exit node D, that belong to the construct to make is viable:
417:     // A -> D -> C
418:     // B -> D -> C
419:     //
420:     // This is fine (assuming C has no PHI nodes), but requires handling the merge instruction here.
421:     // By adding a proxy node, we create a regular divergent shape which can easily be regularized later on.
422:     // A -> D -> D1 -> C
423:     // B -> D -> D2 -> C
424:     //
425:     // A, B, D belongs to the construct. D is the exit. D1 and D2 are empty.
426:     //
427:     // clang-format on
428:     std::vector<Edge>
429:     createAliasBlocksForComplexEdges(std::vector<Edge> Edges) {
430:       std::unordered_set<BasicBlock *> Seen;
431:       std::vector<Edge> Output;
432:       Output.reserve(Edges.size());
433:
434:       for (auto &[Src, Dst] : Edges) {
435:         auto [Iterator, Inserted] = Seen.insert(Src);
436:         if (!Inserted) {
437:           // Src already a source node. Cannot have 2 edges from A to B.
438:           // Creating alias source block.
439:           BasicBlock *NewSrc = BasicBlock::Create(
440:               F.getContext(), Src->getName() + ".new.src", &F);
441:           replaceBranchTargets(Src, Dst, NewSrc);
442:           IRBuilder<> Builder(NewSrc);
443:           Builder.CreateBr(Dst);
444:           Src = NewSrc;
445:         }
446:
447:         Output.emplace_back(Src, Dst);
448:       }
449:
450:       return Output;
```
- EN: This range implements operational logic in helpers such as assert, partialOrderVisit, push_back, createAliasBlocksForComplexEdges, translating backend policy into executable code.
- CN: 这一段实现了 assert、partialOrderVisit、push_back、createAliasBlocksForComplexEdges 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:     }
452:
453:     AllocaInst *CreateVariable(Function &F, Type *Type,
454:                                BasicBlock::iterator Position) {
455:       const DataLayout &DL = F.getDataLayout();
456:       return new AllocaInst(Type, DL.getAllocaAddrSpace(), nullptr, "reg",
457:                             Position);
458:     }
459:
460:     // Given a construct defined by |Header|, and a list of exiting edges
461:     // |Edges|, creates a new single exit node, fixing up those edges.
462:     BasicBlock *createSingleExitNode(BasicBlock *Header,
463:                                      std::vector<Edge> &Edges) {
464:
465:       std::vector<Edge> FixedEdges = createAliasBlocksForComplexEdges(Edges);
466:
467:       std::vector<BasicBlock *> Dsts;
468:       std::unordered_map<BasicBlock *, ConstantInt *> DstToIndex;
469:       auto NewExit = BasicBlock::Create(F.getContext(),
470:                                         Header->getName() + ".new.exit", &F);
471:       IRBuilder<> ExitBuilder(NewExit);
472:       for (auto &[Src, Dst] : FixedEdges) {
473:         if (DstToIndex.count(Dst) != 0)
474:           continue;
475:         DstToIndex.emplace(Dst, ExitBuilder.getInt32(DstToIndex.size()));
476:         Dsts.push_back(Dst);
477:       }
478:
479:       if (Dsts.size() == 1) {
480:         for (auto &[Src, Dst] : FixedEdges) {
481:           replaceBranchTargets(Src, Dst, NewExit);
482:         }
483:         ExitBuilder.CreateBr(Dsts[0]);
484:         return NewExit;
485:       }
486:
487:       AllocaInst *Variable = CreateVariable(F, ExitBuilder.getInt32Ty(),
488:                                             F.begin()->getFirstInsertionPt());
489:       for (auto &[Src, Dst] : FixedEdges) {
490:         IRBuilder<> B2(Src);
491:         B2.SetInsertPoint(Src->getFirstInsertionPt());
492:         B2.CreateStore(DstToIndex[Dst], Variable);
493:         replaceBranchTargets(Src, Dst, NewExit);
494:       }
495:
496:       Value *Load = ExitBuilder.CreateLoad(ExitBuilder.getInt32Ty(), Variable);
497:
498:       // If we can avoid an OpSwitch, generate an OpBranch. Reason is some
499:       // OpBranch are allowed to exist without a new OpSelectionMerge if one of
500:       // the branch is the parent's merge node, while OpSwitches are not.
501:       if (Dsts.size() == 2) {
502:         Value *Condition =
503:             ExitBuilder.CreateCmp(CmpInst::ICMP_EQ, DstToIndex[Dsts[0]], Load);
504:         ExitBuilder.CreateCondBr(Condition, Dsts[0], Dsts[1]);
505:         return NewExit;
506:       }
507:
508:       SwitchInst *Sw = ExitBuilder.CreateSwitch(Load, Dsts[0], Dsts.size() - 1);
509:       for (BasicBlock *BB : drop_begin(Dsts))
510:         Sw->addCase(DstToIndex[BB], BB);
511:       return NewExit;
512:     }
513:   };
514:
515:   /// Create a value in BB set to the value associated with the branch the block
516:   /// terminator will take.
517:   Value *createExitVariable(
518:       BasicBlock *BB,
519:       const DenseMap<BasicBlock *, ConstantInt *> &TargetToValue) {
520:     auto *T = BB->getTerminator();
521:     if (isa<ReturnInst>(T))
522:       return nullptr;
523:     if (auto *BI = dyn_cast<UncondBrInst>(T))
524:       return TargetToValue.lookup(BI->getSuccessor());
525:
526:     IRBuilder<> Builder(BB);
527:     Builder.SetInsertPoint(T);
528:
529:     if (auto *BI = dyn_cast<CondBrInst>(T)) {
530:       Value *LHS = TargetToValue.lookup(BI->getSuccessor(0));
531:       Value *RHS = TargetToValue.lookup(BI->getSuccessor(1));
532:
533:       if (LHS == nullptr || RHS == nullptr)
534:         return LHS == nullptr ? RHS : LHS;
535:       return Builder.CreateSelect(BI->getCondition(), LHS, RHS);
536:     }
537:
538:     // TODO: add support for switch cases.
539:     llvm_unreachable("Unhandled terminator type.");
540:   }
```
- EN: This range implements operational logic in helpers such as getDataLayout, createAliasBlocksForComplexEdges, getName, ExitBuilder, translating backend policy into executable code.
- CN: 这一段实现了 getDataLayout、createAliasBlocksForComplexEdges、getName、ExitBuilder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:
542:   // Creates a new basic block in F with a single OpUnreachable instruction.
543:   BasicBlock *CreateUnreachable(Function &F) {
544:     BasicBlock *BB = BasicBlock::Create(F.getContext(), "unreachable", &F);
545:     IRBuilder<> Builder(BB);
546:     Builder.CreateUnreachable();
547:     return BB;
548:   }
549:
550:   // Add OpLoopMerge instruction on cycles.
551:   bool addMergeForLoops(Function &F) {
552:     LoopInfo &LI = getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
553:     auto *TopLevelRegion =
554:         getAnalysis<SPIRVConvergenceRegionAnalysisWrapperPass>()
555:             .getRegionInfo()
556:             .getTopLevelRegion();
557:
558:     bool Modified = false;
559:     for (auto &BB : F) {
560:       // Not a loop header. Ignoring for now.
561:       if (!LI.isLoopHeader(&BB))
562:         continue;
563:       auto *L = LI.getLoopFor(&BB);
564:
565:       // This loop header is not the entrance of a convergence region. Ignoring
566:       // this block.
567:       auto *CR = getRegionForHeader(TopLevelRegion, &BB);
568:       if (CR == nullptr)
569:         continue;
570:
571:       IRBuilder<> Builder(&BB);
572:
573:       auto *Merge = getExitFor(CR);
574:       // We are indeed in a loop, but there are no exits (infinite loop).
575:       // This could be caused by a bad shader, but also could be an artifact
576:       // from an earlier optimization. It is not always clear if structurally
577:       // reachable means runtime reachable, so we cannot error-out. What we must
578:       // do however is to make is legal on the SPIR-V point of view, hence
579:       // adding an unreachable merge block.
580:       if (Merge == nullptr) {
581:         UncondBrInst *Br = cast<UncondBrInst>(BB.getTerminator());
582:         Merge = CreateUnreachable(F);
583:         Builder.SetInsertPoint(Br);
584:         Builder.CreateCondBr(Builder.getFalse(), Merge, Br->getSuccessor(0));
585:         Br->eraseFromParent();
586:       }
587:
588:       auto *Continue = L->getLoopLatch();
589:
590:       Builder.SetInsertPoint(BB.getTerminator());
591:       auto MergeAddress = BlockAddress::get(Merge->getParent(), Merge);
592:       auto ContinueAddress = BlockAddress::get(Continue->getParent(), Continue);
593:       SmallVector<Value *, 2> Args = {MergeAddress, ContinueAddress};
594:       SmallVector<unsigned, 1> LoopControlImms =
595:           getSpirvLoopControlOperandsFromLoopMetadata(L);
596:       for (unsigned Imm : LoopControlImms)
597:         Args.emplace_back(ConstantInt::get(Builder.getInt32Ty(), Imm));
598:       Builder.CreateIntrinsic(Intrinsic::spv_loop_merge, {Args});
599:       Modified = true;
600:     }
601:
602:     return Modified;
603:   }
604:
605:   // Adds an OpSelectionMerge to the immediate dominator or each node with an
606:   // in-degree of 2 or more which is not already the merge target of an
607:   // OpLoopMerge/OpSelectionMerge.
608:   bool addMergeForNodesWithMultiplePredecessors(Function &F) {
609:     DomTreeBuilder::BBDomTree DT;
610:     DT.recalculate(F);
611:
612:     bool Modified = false;
613:     for (auto &BB : F) {
614:       if (pred_size(&BB) <= 1)
615:         continue;
616:
617:       if (hasLoopMergeInstruction(BB) && pred_size(&BB) <= 2)
618:         continue;
619:
620:       assert(DT.getNode(&BB)->getIDom());
621:       BasicBlock *Header = DT.getNode(&BB)->getIDom()->getBlock();
622:
623:       if (isDefinedAsSelectionMergeBy(*Header, BB))
624:         continue;
625:
626:       IRBuilder<> Builder(Header);
627:       Builder.SetInsertPoint(Header->getTerminator());
628:
629:       auto MergeAddress = BlockAddress::get(BB.getParent(), &BB);
630:       createOpSelectMerge(&Builder, MergeAddress);
```
- EN: This range implements operational logic in helpers such as CreateUnreachable, BasicBlock::Create, Builder, addMergeForLoops, translating backend policy into executable code.
- CN: 这一段实现了 CreateUnreachable、BasicBlock::Create、Builder、addMergeForLoops 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:
632:       Modified = true;
633:     }
634:
635:     return Modified;
636:   }
637:
638:   // When a block has multiple OpSelectionMerge/OpLoopMerge instructions, sorts
639:   // them to put the "largest" first. A merge instruction is defined as larger
640:   // than another when its target merge block post-dominates the other target's
641:   // merge block. (This ordering should match the nesting ordering of the source
642:   // HLSL).
643:   bool sortSelectionMerge(Function &F, BasicBlock &Block) {
644:     std::vector<Instruction *> MergeInstructions;
645:     for (Instruction &I : Block)
646:       if (isMergeInstruction(&I))
647:         MergeInstructions.push_back(&I);
648:
649:     if (MergeInstructions.size() <= 1)
650:       return false;
651:
652:     Instruction *InsertionPoint = *MergeInstructions.begin();
653:
654:     PartialOrderingVisitor Visitor(F);
655:     std::sort(MergeInstructions.begin(), MergeInstructions.end(),
656:               [&Visitor](Instruction *Left, Instruction *Right) {
657:                 if (Left == Right)
658:                   return false;
659:                 BasicBlock *RightMerge = getDesignatedMergeBlock(Right);
660:                 BasicBlock *LeftMerge = getDesignatedMergeBlock(Left);
661:                 return !Visitor.compare(RightMerge, LeftMerge);
662:               });
663:
664:     for (Instruction *I : MergeInstructions) {
665:       I->moveBefore(InsertionPoint->getIterator());
666:       InsertionPoint = I;
667:     }
668:
669:     return true;
670:   }
671:
672:   // Sorts selection merge headers in |F|.
673:   // A is sorted before B if the merge block designated by B is an ancestor of
674:   // the one designated by A.
675:   bool sortSelectionMergeHeaders(Function &F) {
676:     bool Modified = false;
677:     for (BasicBlock &BB : F) {
678:       Modified |= sortSelectionMerge(F, BB);
679:     }
680:     return Modified;
681:   }
682:
683:   // Split basic blocks containing multiple OpLoopMerge/OpSelectionMerge
684:   // instructions so each basic block contains only a single merge instruction.
685:   bool splitBlocksWithMultipleHeaders(Function &F) {
686:     std::stack<BasicBlock *> Work;
687:     for (auto &BB : F) {
688:       std::vector<Instruction *> MergeInstructions = getMergeInstructions(BB);
689:       if (MergeInstructions.size() <= 1)
690:         continue;
691:       Work.push(&BB);
692:     }
693:
694:     const bool Modified = Work.size() > 0;
695:     while (Work.size() > 0) {
696:       BasicBlock *Header = Work.top();
697:       Work.pop();
698:
699:       std::vector<Instruction *> MergeInstructions =
700:           getMergeInstructions(*Header);
701:       for (unsigned i = 1; i < MergeInstructions.size(); i++) {
702:         BasicBlock *NewBlock =
703:             Header->splitBasicBlock(MergeInstructions[i], "new.header");
704:
705:         if (getDesignatedContinueBlock(MergeInstructions[0]) == nullptr) {
706:           BasicBlock *Unreachable = CreateUnreachable(F);
707:
708:           Instruction *Term = Header->getTerminator();
709:           IRBuilder<> Builder(Header);
710:           Builder.SetInsertPoint(Term);
711:           Builder.CreateCondBr(Builder.getTrue(), NewBlock, Unreachable);
712:           Term->eraseFromParent();
713:         }
714:
715:         Header = NewBlock;
716:       }
717:     }
718:
719:     return Modified;
720:   }
```
- EN: This range implements operational logic in helpers such as sortSelectionMerge, push_back, begin, Visitor, translating backend policy into executable code.
- CN: 这一段实现了 sortSelectionMerge、push_back、begin、Visitor 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:
722:   // Adds an OpSelectionMerge to each block with an out-degree >= 2 which
723:   // doesn't already have an OpSelectionMerge.
724:   bool addMergeForDivergentBlocks(Function &F) {
725:     DomTreeBuilder::BBPostDomTree PDT;
726:     PDT.recalculate(F);
727:     bool Modified = false;
728:
729:     auto MergeBlocks = getMergeBlocks(F);
730:     auto ContinueBlocks = getContinueBlocks(F);
731:
732:     for (auto &BB : F) {
733:       if (getMergeInstructions(BB).size() != 0)
734:         continue;
735:
736:       std::vector<BasicBlock *> Candidates;
737:       for (BasicBlock *Successor : successors(&BB)) {
738:         if (MergeBlocks.contains(Successor))
739:           continue;
740:         if (ContinueBlocks.contains(Successor))
741:           continue;
742:         Candidates.push_back(Successor);
743:       }
744:
745:       if (Candidates.size() <= 1)
746:         continue;
747:
748:       Modified = true;
749:       BasicBlock *Merge = Candidates[0];
750:
751:       auto MergeAddress = BlockAddress::get(Merge->getParent(), Merge);
752:       IRBuilder<> Builder(&BB);
753:       Builder.SetInsertPoint(BB.getTerminator());
754:       createOpSelectMerge(&Builder, MergeAddress);
755:     }
756:
757:     return Modified;
758:   }
759:
760:   // Gather all the exit nodes for the construct header by |Header| and
761:   // containing the blocks |Construct|.
762:   std::vector<Edge> getExitsFrom(const BlockSet &Construct,
763:                                  BasicBlock &Header) {
764:     std::vector<Edge> Output;
765:     visit(Header, [&](BasicBlock *Item) {
766:       if (Construct.count(Item) == 0)
767:         return false;
768:
769:       for (BasicBlock *Successor : successors(Item)) {
770:         if (Construct.count(Successor) == 0)
771:           Output.emplace_back(Item, Successor);
772:       }
773:       return true;
774:     });
775:
776:     return Output;
777:   }
778:
779:   // Build a divergent construct tree searching from |BB|.
780:   // If |Parent| is not null, this tree is attached to the parent's tree.
781:   void constructDivergentConstruct(BlockSet &Visited, Splitter &S,
782:                                    BasicBlock *BB, DivergentConstruct *Parent) {
783:     if (Visited.count(BB) != 0)
784:       return;
785:     Visited.insert(BB);
786:
787:     auto MIS = getMergeInstructions(*BB);
788:     if (MIS.size() == 0) {
789:       for (BasicBlock *Successor : successors(BB))
790:         constructDivergentConstruct(Visited, S, Successor, Parent);
791:       return;
792:     }
793:
794:     assert(MIS.size() == 1);
795:     Instruction *MI = MIS[0];
796:
797:     BasicBlock *Merge = getDesignatedMergeBlock(MI);
798:     BasicBlock *Continue = getDesignatedContinueBlock(MI);
799:
800:     auto Output = std::make_unique<DivergentConstruct>();
801:     Output->Header = BB;
802:     Output->Merge = Merge;
803:     Output->Continue = Continue;
804:     Output->Parent = Parent;
805:
806:     constructDivergentConstruct(Visited, S, Merge, Parent);
807:     if (Continue)
808:       constructDivergentConstruct(Visited, S, Continue, Output.get());
809:
810:     for (BasicBlock *Successor : successors(BB))
```
- EN: This range implements operational logic in helpers such as addMergeForDivergentBlocks, recalculate, getMergeBlocks, getContinueBlocks, translating backend policy into executable code.
- CN: 这一段实现了 addMergeForDivergentBlocks、recalculate、getMergeBlocks、getContinueBlocks 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:       constructDivergentConstruct(Visited, S, Successor, Output.get());
812:
813:     if (Parent)
814:       Parent->Children.emplace_back(std::move(Output));
815:   }
816:
817:   // Returns the blocks belonging to the divergent construct |Node|.
818:   BlockSet getConstructBlocks(Splitter &S, DivergentConstruct *Node) {
819:     assert(Node->Header && Node->Merge);
820:
821:     if (Node->Continue) {
822:       auto LoopBlocks = S.getLoopConstructBlocks(Node->Header, Node->Merge);
823:       return BlockSet(LoopBlocks.begin(), LoopBlocks.end());
824:     }
825:
826:     auto SelectionBlocks = S.getSelectionConstructBlocks(Node);
827:     return BlockSet(SelectionBlocks.begin(), SelectionBlocks.end());
828:   }
829:
830:   // Fixup the construct |Node| to respect a set of rules defined by the SPIR-V
831:   // spec.
832:   bool fixupConstruct(Splitter &S, DivergentConstruct *Node) {
833:     bool Modified = false;
834:     for (auto &Child : Node->Children)
835:       Modified |= fixupConstruct(S, Child.get());
836:
837:     // This construct is the root construct. Does not represent any real
838:     // construct, just a way to access the first level of the forest.
839:     if (Node->Parent == nullptr)
840:       return Modified;
841:
842:     // This node's parent is the root. Meaning this is a top-level construct.
843:     // There can be multiple exists, but all are guaranteed to exit at most 1
844:     // construct since we are at first level.
845:     if (Node->Parent->Header == nullptr)
846:       return Modified;
847:
848:     // Health check for the structure.
849:     assert(Node->Header && Node->Merge);
850:     assert(Node->Parent->Header && Node->Parent->Merge);
851:
852:     BlockSet ConstructBlocks = getConstructBlocks(S, Node);
853:     auto Edges = getExitsFrom(ConstructBlocks, *Node->Header);
854:
855:     //  No edges exiting the construct.
856:     if (Edges.size() < 1)
857:       return Modified;
858:
859:     bool HasBadEdge = Node->Merge == Node->Parent->Merge ||
860:                       Node->Merge == Node->Parent->Continue;
861:     // BasicBlock *Target = Edges[0].second;
862:     for (auto &[Src, Dst] : Edges) {
863:       // - Breaking from a selection construct: S is a selection construct, S is
864:       // the innermost structured
865:       //   control-flow construct containing A, and B is the merge block for S
866:       // - Breaking from the innermost loop: S is the innermost loop construct
867:       // containing A,
868:       //   and B is the merge block for S
869:       if (Node->Merge == Dst)
870:         continue;
871:
872:       // Entering the innermost loop’s continue construct: S is the innermost
873:       // loop construct containing A, and B is the continue target for S
874:       if (Node->Continue == Dst)
875:         continue;
876:
877:       // TODO: what about cases branching to another case in the switch? Seems
878:       // to work, but need to double check.
879:       HasBadEdge = true;
880:     }
881:
882:     if (!HasBadEdge)
883:       return Modified;
884:
885:     // Create a single exit node gathering all exit edges.
886:     BasicBlock *NewExit = S.createSingleExitNode(Node->Header, Edges);
887:
888:     // Fixup this construct's merge node to point to the new exit.
889:     // Note: this algorithm fixes inner-most divergence construct first. So
890:     // recursive structures sharing a single merge node are fixed from the
891:     // inside toward the outside.
892:     auto MergeInstructions = getMergeInstructions(*Node->Header);
893:     assert(MergeInstructions.size() == 1);
894:     Instruction *I = MergeInstructions[0];
895:     BlockAddress *BA = cast<BlockAddress>(I->getOperand(0));
896:     if (BA->getBasicBlock() == Node->Merge) {
897:       auto MergeAddress = BlockAddress::get(NewExit->getParent(), NewExit);
898:       I->setOperand(0, MergeAddress);
899:     }
900:
```
- EN: This range implements operational logic in helpers such as constructDivergentConstruct, emplace_back, getConstructBlocks, assert, translating backend policy into executable code.
- CN: 这一段实现了 constructDivergentConstruct、emplace_back、getConstructBlocks、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:     // Clean up of the possible dangling BockAddr operands to prevent MIR
902:     // comments about "address of removed block taken".
903:     if (!BA->isConstantUsed())
904:       BA->destroyConstant();
905:
906:     Node->Merge = NewExit;
907:     // Regenerate the dom trees.
908:     S.invalidate();
909:     return true;
910:   }
911:
912:   bool splitCriticalEdges(Function &F) {
913:     LoopInfo &LI = getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
914:     Splitter S(F, LI);
915:
916:     DivergentConstruct Root;
917:     BlockSet Visited;
918:     constructDivergentConstruct(Visited, S, &*F.begin(), &Root);
919:     return fixupConstruct(S, &Root);
920:   }
921:
922:   // Simplify branches when possible:
923:   //  - if the 2 sides of a conditional branch are the same, transforms it to an
924:   //  unconditional branch.
925:   //  - if a switch has only 2 distinct successors, converts it to a conditional
926:   //  branch.
927:   bool simplifyBranches(Function &F) {
928:     bool Modified = false;
929:
930:     for (BasicBlock &BB : F) {
931:       SwitchInst *SI = dyn_cast<SwitchInst>(BB.getTerminator());
932:       if (!SI)
933:         continue;
934:       if (SI->getNumCases() > 1)
935:         continue;
936:
937:       Modified = true;
938:       IRBuilder<> Builder(&BB);
939:       Builder.SetInsertPoint(SI);
940:
941:       if (SI->getNumCases() == 0) {
942:         Builder.CreateBr(SI->getDefaultDest());
943:       } else {
944:         Value *Condition =
945:             Builder.CreateCmp(CmpInst::ICMP_EQ, SI->getCondition(),
946:                               SI->case_begin()->getCaseValue());
947:         Builder.CreateCondBr(Condition, SI->case_begin()->getCaseSuccessor(),
948:                              SI->getDefaultDest());
949:       }
950:       SI->eraseFromParent();
951:     }
952:
953:     return Modified;
954:   }
955:
956:   // Makes sure every case target in |F| is unique. If 2 cases branch to the
957:   // same basic block, one of the targets is updated so it jumps to a new basic
958:   // block ending with a single unconditional branch to the original target.
959:   bool splitSwitchCases(Function &F) {
960:     bool Modified = false;
961:
962:     for (BasicBlock &BB : F) {
963:       SwitchInst *SI = dyn_cast<SwitchInst>(BB.getTerminator());
964:       if (!SI)
965:         continue;
966:
967:       BlockSet Seen;
968:       Seen.insert(SI->getDefaultDest());
969:
970:       auto It = SI->case_begin();
971:       while (It != SI->case_end()) {
972:         BasicBlock *Target = It->getCaseSuccessor();
973:         if (Seen.count(Target) == 0) {
974:           Seen.insert(Target);
975:           ++It;
976:           continue;
977:         }
978:
979:         Modified = true;
980:         BasicBlock *NewTarget =
981:             BasicBlock::Create(F.getContext(), "new.sw.case", &F);
982:         IRBuilder<> Builder(NewTarget);
983:         Builder.CreateBr(Target);
984:         SI->addCase(It->getCaseValue(), NewTarget);
985:         It = SI->removeCase(It);
986:       }
987:     }
988:
989:     return Modified;
990:   }
```
- EN: This range implements operational logic in helpers such as destroyConstant, invalidate, splitCriticalEdges, getLoopInfo, translating backend policy into executable code.
- CN: 这一段实现了 destroyConstant、invalidate、splitCriticalEdges、getLoopInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:
 992:   // Removes blocks not contributing to any structured CFG. This assumes there
 993:   // is no PHI nodes.
 994:   bool removeUselessBlocks(Function &F) {
 995:     std::vector<BasicBlock *> ToRemove;
 996:
 997:     auto MergeBlocks = getMergeBlocks(F);
 998:     auto ContinueBlocks = getContinueBlocks(F);
 999:
1000:     for (BasicBlock &BB : F) {
1001:       if (BB.size() != 1)
1002:         continue;
1003:
1004:       if (isa<ReturnInst>(BB.getTerminator()))
1005:         continue;
1006:
1007:       if (MergeBlocks.count(&BB) != 0 || ContinueBlocks.count(&BB) != 0)
1008:         continue;
1009:
1010:       if (BB.getUniqueSuccessor() == nullptr)
1011:         continue;
1012:
1013:       BasicBlock *Successor = BB.getUniqueSuccessor();
1014:       std::vector<BasicBlock *> Predecessors(predecessors(&BB).begin(),
1015:                                              predecessors(&BB).end());
1016:       for (BasicBlock *Predecessor : Predecessors)
1017:         replaceBranchTargets(Predecessor, &BB, Successor);
1018:       ToRemove.push_back(&BB);
1019:     }
1020:
1021:     for (BasicBlock *BB : ToRemove)
1022:       BB->eraseFromParent();
1023:
1024:     return ToRemove.size() != 0;
1025:   }
1026:
1027:   bool addHeaderToRemainingDivergentDAG(Function &F) {
1028:     bool Modified = false;
1029:
1030:     auto MergeBlocks = getMergeBlocks(F);
1031:     auto ContinueBlocks = getContinueBlocks(F);
1032:     auto HeaderBlocks = getHeaderBlocks(F);
1033:
1034:     DomTreeBuilder::BBDomTree DT;
1035:     DomTreeBuilder::BBPostDomTree PDT;
1036:     PDT.recalculate(F);
1037:     DT.recalculate(F);
1038:
1039:     for (BasicBlock &BB : F) {
1040:       if (HeaderBlocks.count(&BB) != 0)
1041:         continue;
1042:       if (succ_size(&BB) < 2)
1043:         continue;
1044:
1045:       size_t CandidateEdges = 0;
1046:       for (BasicBlock *Successor : successors(&BB)) {
1047:         if (MergeBlocks.count(Successor) != 0 ||
1048:             ContinueBlocks.count(Successor) != 0)
1049:           continue;
1050:         if (HeaderBlocks.count(Successor) != 0)
1051:           continue;
1052:         CandidateEdges += 1;
1053:       }
1054:
1055:       if (CandidateEdges <= 1)
1056:         continue;
1057:
1058:       BasicBlock *Header = &BB;
1059:       BasicBlock *Merge = PDT.getNode(&BB)->getIDom()->getBlock();
1060:
1061:       bool HasBadBlock = false;
1062:       visit(*Header, [&](const BasicBlock *Node) {
1063:         if (DT.dominates(Header, Node))
1064:           return false;
1065:         if (PDT.dominates(Merge, Node))
1066:           return false;
1067:         if (Node == Header || Node == Merge)
1068:           return true;
1069:
1070:         HasBadBlock |= MergeBlocks.count(Node) != 0 ||
1071:                        ContinueBlocks.count(Node) != 0 ||
1072:                        HeaderBlocks.count(Node) != 0;
1073:         return !HasBadBlock;
1074:       });
1075:
1076:       if (HasBadBlock)
1077:         continue;
1078:
1079:       Modified = true;
1080:
```
- EN: This range implements operational logic in helpers such as removeUselessBlocks, getMergeBlocks, getContinueBlocks, getUniqueSuccessor, translating backend policy into executable code.
- CN: 这一段实现了 removeUselessBlocks、getMergeBlocks、getContinueBlocks、getUniqueSuccessor 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:       if (Merge == nullptr) {
1082:         Merge = *successors(Header).begin();
1083:         IRBuilder<> Builder(Header);
1084:         Builder.SetInsertPoint(Header->getTerminator());
1085:
1086:         auto MergeAddress = BlockAddress::get(Merge->getParent(), Merge);
1087:         createOpSelectMerge(&Builder, MergeAddress);
1088:         continue;
1089:       }
1090:
1091:       Instruction *SplitInstruction = Merge->getTerminator();
1092:       if (isMergeInstruction(SplitInstruction->getPrevNode()))
1093:         SplitInstruction = SplitInstruction->getPrevNode();
1094:       BasicBlock *NewMerge =
1095:           Merge->splitBasicBlockBefore(SplitInstruction, "new.merge");
1096:
1097:       IRBuilder<> Builder(Header);
1098:       Builder.SetInsertPoint(Header->getTerminator());
1099:
1100:       auto MergeAddress = BlockAddress::get(NewMerge->getParent(), NewMerge);
1101:       createOpSelectMerge(&Builder, MergeAddress);
1102:     }
1103:
1104:     return Modified;
1105:   }
1106:
1107: public:
1108:   static char ID;
1109:
1110:   SPIRVStructurizer() : FunctionPass(ID) {}
1111:
1112:   bool runOnFunction(Function &F) override {
1113:     bool Modified = false;
1114:
1115:     // In LLVM, Switches are allowed to have several cases branching to the same
1116:     // basic block. This is allowed in SPIR-V, but can make structurizing SPIR-V
1117:     // harder, so first remove edge cases.
1118:     Modified |= splitSwitchCases(F);
1119:
1120:     // LLVM allows conditional branches to have both side jumping to the same
1121:     // block. It also allows switched to have a single default, or just one
1122:     // case. Cleaning this up now.
1123:     Modified |= simplifyBranches(F);
1124:
1125:     // At this state, we should have a reducible CFG with cycles.
1126:     // STEP 1: Adding OpLoopMerge instructions to loop headers.
1127:     Modified |= addMergeForLoops(F);
1128:
1129:     // STEP 2: adding OpSelectionMerge to each node with an in-degree >= 2.
1130:     Modified |= addMergeForNodesWithMultiplePredecessors(F);
1131:
1132:     // STEP 3:
1133:     // Sort selection merge, the largest construct goes first.
1134:     // This simplifies the next step.
1135:     Modified |= sortSelectionMergeHeaders(F);
1136:
1137:     // STEP 4: As this stage, we can have a single basic block with multiple
1138:     // OpLoopMerge/OpSelectionMerge instructions. Splitting this block so each
1139:     // BB has a single merge instruction.
1140:     Modified |= splitBlocksWithMultipleHeaders(F);
1141:
1142:     // STEP 5: In the previous steps, we added merge blocks the loops and
1143:     // natural merge blocks (in-degree >= 2). What remains are conditions with
1144:     // an exiting branch (return, unreachable). In such case, we must start from
1145:     // the header, and add headers to divergent construct with no headers.
1146:     Modified |= addMergeForDivergentBlocks(F);
1147:
1148:     // STEP 6: At this stage, we have several divergent construct defines by a
1149:     // header and a merge block. But their boundaries have no constraints: a
1150:     // construct exit could be outside of the parents' construct exit. Such
1151:     // edges are called critical edges. What we need is to split those edges
1152:     // into several parts. Each part exiting the parent's construct by its merge
1153:     // block.
1154:     Modified |= splitCriticalEdges(F);
1155:
1156:     // STEP 7: The previous steps possibly created a lot of "proxy" blocks.
1157:     // Blocks with a single unconditional branch, used to create a valid
1158:     // divergent construct tree. Some nodes are still requires (e.g: nodes
1159:     // allowing a valid exit through the parent's merge block). But some are
1160:     // left-overs of past transformations, and could cause actual validation
1161:     // issues. E.g: the SPIR-V spec allows a construct to break to the parents
1162:     // loop construct without an OpSelectionMerge, but this requires a straight
1163:     // jump. If a proxy block lies between the conditional branch and the
1164:     // parent's merge, the CFG is not valid.
1165:     Modified |= removeUselessBlocks(F);
1166:
1167:     // STEP 8: Final fix-up steps: our tree boundaries are correct, but some
1168:     // blocks are branching with no header. Those are often simple conditional
1169:     // branches with 1 or 2 returning edges. Adding a header for those.
1170:     Modified |= addHeaderToRemainingDivergentDAG(F);
```
- EN: This range implements operational logic in helpers such as successors, Builder, SetInsertPoint, BlockAddress::get, translating backend policy into executable code.
- CN: 这一段实现了 successors、Builder、SetInsertPoint、BlockAddress::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1235
```cpp
1171:
1172:     // STEP 9: sort basic blocks to match both the LLVM & SPIR-V requirements.
1173:     Modified |= sortBlocks(F);
1174:
1175:     return Modified;
1176:   }
1177:
1178:   void getAnalysisUsage(AnalysisUsage &AU) const override {
1179:     AU.addRequired<DominatorTreeWrapperPass>();
1180:     AU.addRequired<LoopInfoWrapperPass>();
1181:     AU.addRequired<SPIRVConvergenceRegionAnalysisWrapperPass>();
1182:
1183:     AU.addPreserved<SPIRVConvergenceRegionAnalysisWrapperPass>();
1184:     FunctionPass::getAnalysisUsage(AU);
1185:   }
1186:
1187:   void createOpSelectMerge(IRBuilder<> *Builder, BlockAddress *MergeAddress) {
1188:     Instruction *BBTerminatorInst = Builder->GetInsertBlock()->getTerminator();
1189:
1190:     MDNode *MDNode = BBTerminatorInst->getMetadata("hlsl.controlflow.hint");
1191:
1192:     ConstantInt *BranchHint = ConstantInt::get(Builder->getInt32Ty(), 0);
1193:
1194:     if (MDNode) {
1195:       assert(MDNode->getNumOperands() == 2 &&
1196:              "invalid metadata hlsl.controlflow.hint");
1197:       BranchHint = mdconst::extract<ConstantInt>(MDNode->getOperand(1));
1198:     }
1199:
1200:     SmallVector<Value *, 2> Args = {MergeAddress, BranchHint};
1201:
1202:     Builder->CreateIntrinsic(Intrinsic::spv_selection_merge,
1203:                              {MergeAddress->getType()}, Args);
1204:   }
1205: };
1206: } // anonymous namespace
1207:
1208: char SPIRVStructurizer::ID = 0;
1209:
1210: INITIALIZE_PASS_BEGIN(SPIRVStructurizer, "spirv-structurizer",
1211:                       "structurize SPIRV", false, false)
1212: INITIALIZE_PASS_DEPENDENCY(LoopSimplify)
1213: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
1214: INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
1215: INITIALIZE_PASS_DEPENDENCY(SPIRVConvergenceRegionAnalysisWrapperPass)
1216:
1217: INITIALIZE_PASS_END(SPIRVStructurizer, "spirv-structurizer",
1218:                     "structurize SPIRV", false, false)
1219:
1220: FunctionPass *llvm::createSPIRVStructurizerPass() {
1221:   return new SPIRVStructurizer();
1222: }
1223:
1224: PreservedAnalyses SPIRVStructurizerWrapper::run(Function &F,
1225:                                                 FunctionAnalysisManager &AF) {
1226:
1227:   auto FPM = legacy::FunctionPassManager(F.getParent());
1228:   FPM.add(createSPIRVStructurizerPass());
1229:
1230:   if (!FPM.run(F))
1231:     return PreservedAnalyses::all();
1232:   PreservedAnalyses PA;
1233:   PA.preserveSet<CFGAnalyses>();
1234:   return PA;
1235: }
```
- EN: This range implements operational logic in helpers such as sortBlocks, getAnalysisUsage, FunctionPass::getAnalysisUsage, createOpSelectMerge, translating backend policy into executable code.
- CN: 这一段实现了 sortBlocks、getAnalysisUsage、FunctionPass::getAnalysisUsage、createOpSelectMerge 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include bool, V, partialOrderVisit, getRegionForHeader, getExitFor, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 bool, V, partialOrderVisit, getRegionForHeader, getExitFor，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `Analysis/SPIRVConvergenceRegionAnalysis.h`
  - `SPIRV.h`
  - `SPIRVStructurizerWrapper.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SmallPtrSet.h`
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/IR/CFG.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/LegacyPassManager.h`
  - `llvm/InitializePasses.h`
  - `llvm/Transforms/Utils.h`
  - `llvm/Transforms/Utils/Cloning.h`
  - `llvm/Transforms/Utils/LoopSimplify.h`
  - `llvm/Transforms/Utils/LowerMemIntrinsics.h`
- System/standard headers / 系统或标准头文件:
  - `stack`
  - `unordered_set`
