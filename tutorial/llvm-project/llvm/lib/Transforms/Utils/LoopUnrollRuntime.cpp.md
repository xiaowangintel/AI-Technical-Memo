# LoopUnrollRuntime.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopUnrollRuntime.cpp` | `llvm/lib/Transforms/Utils/LoopUnrollRuntime.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements === UnrollLoopRuntime.cpp - Runtime Loop unrolling utilities === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopUnrollRuntime 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-104

```cpp
//===-- UnrollLoopRuntime.cpp - Runtime Loop unrolling utilities ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements some loop unrolling utilities for loops with run-time
// trip counts.  See LoopUnroll.cpp for unrolling loops with compile-time
// trip counts.
//
// The functions in this file are used to generate extra code when the
// run-time trip count modulo the unroll factor is not 0.  When this is the
// case, we need to generate code to execute these 'left over' iterations.
//
// The current strategy generates an if-then-else sequence prior to the
// unrolled loop to execute the 'left over' iterations before or after the
// unrolled loop.
//
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Preheader
  //  PrologHeader
  //  ...
  //  PrologLatch
  //  PrologExit
  //   NewPreheader
  //    Header
  //    ...
  //    Latch
  //      LatchExit
  BasicBlock *Latch = L->getLoopLatch();
  assert(Latch && "Loop must have a latch");
  BasicBlock *PrologLatch = cast<BasicBlock>(VMap[Latch]);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 105-198

```cpp
  // Create a PHI node for each outgoing value from the original loop
  // (which means it is an outgoing value from the prolog code too).
  // The new PHI node is inserted in the prolog end basic block.
  // The new PHI node value is added as an operand of a PHI node in either
  // the loop header or the loop exit block.
  for (BasicBlock *Succ : successors(Latch)) {
    for (PHINode &PN : Succ->phis()) {
      // Add a new PHI node to the prolog end block and add the
      // appropriate incoming values.
      // TODO: This code assumes that the PrologExit (or the LatchExit block for
      // prolog loop) contains only one predecessor from the loop, i.e. the
      // PrologLatch. When supporting multiple-exiting block loops, we can have
      // two or more blocks that have the LatchExit as the target in the
      // original loop.
      PHINode *NewPN = PHINode::Create(PN.getType(), 2, PN.getName() + ".unr");
      NewPN->insertBefore(PrologExit->getFirstNonPHIIt());
      // Adding a value to the new PHI node from the original loop preheader.
      // This is the value that skips all the prolog code.
      if (L->contains(&PN)) {
        // Succ is loop header.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Assume loop is nearly always entered.
    MDBuilder MDB(B.getContext());
    BranchWeights = MDB.createBranchWeights(UnrolledLoopHeaderWeights);
  }
  B.CreateCondBr(BrLoopExit, OriginalLoopLatchExit, NewPreHeader,
                 BranchWeights);
  InsertPt->eraseFromParent();
  if (DT) {
    auto *NewDom = DT->findNearestCommonDominator(OriginalLoopLatchExit,
                                                  PrologExit);
    DT->changeImmediateDominator(OriginalLoopLatchExit, NewDom);
  }
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 199-300

```cpp
/// Assume, due to our position in the remainder loop or its guard, anywhere
/// from 0 to \p N more iterations can possibly execute.  Among such cases in
/// the original loop (with loop probability \p OriginalLoopProb), what is the
/// probability of executing at least one more iteration?
static BranchProbability
probOfNextInRemainder(BranchProbability OriginalLoopProb, unsigned N) {
  // OriginalLoopProb == 1 would produce a division by zero in the calculation
  // below.  The problem is that case indicates an always infinite loop, but a
  // remainder loop cannot be calculated at run time if the original loop is
  // infinite as infinity % UnrollCount is undefined.  We then choose
  // probabilities indicating that all remainder loop iterations will always
  // execute.
  //
  // Currently, the remainder loop here is an epilogue, which cannot be reached
  // if the original loop is infinite, so the aforementioned choice is
  // arbitrary.
  //
  // FIXME: Branch weights still need to be fixed in the case of prologues
  // (issue #135812).  In that case, the aforementioned choice seems reasonable
  // for the goal of maintaining the original loop's block frequencies.  That
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    //   PN = PHI [I, Latch]
    // ...
    // Exit:
    //   EpilogPN = PHI [PN, EpilogPreHeader], [X, Exit2], [Y, Exit2.epil]
    //
    // Exits from non-latch blocks point to the original exit block and the
    // epilogue edges have already been added.
    //
    // There is EpilogPreHeader incoming block instead of NewExit as
    // NewExit was split 1 more time to get EpilogPreHeader.
    assert(PN.hasOneUse() && "The phi should have 1 use");
    PHINode *EpilogPN = cast<PHINode>(PN.use_begin()->getUser());
    assert(EpilogPN->getParent() == Exit && "EpilogPN should be in Exit block");

```
- EN: Core entities appearing here include probOfNextInRemainder, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 probOfNextInRemainder，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 301-398

```cpp
    Value *V = PN.getIncomingValueForBlock(Latch);
    Instruction *I = dyn_cast<Instruction>(V);
    if (I && L->contains(I))
      // If value comes from an instruction in the loop add VMap value.
      V = VMap.lookup(I);
    // For the instruction out of the loop, constant or undefined value
    // insert value itself.
    EpilogPN->addIncoming(V, EpilogLatch);

    assert(EpilogPN->getBasicBlockIndex(EpilogPreHeader) >= 0 &&
          "EpilogPN should have EpilogPreHeader incoming block");
    // Change EpilogPreHeader incoming block to NewExit.
    EpilogPN->setIncomingBlock(EpilogPN->getBasicBlockIndex(EpilogPreHeader),
                               NewExit);
    // Now PHIs should look like:
    // NewExit:
    //   PN = PHI [I, Latch]
    // ...
    // Exit:
    //   EpilogPN = PHI [PN, NewExit], [VMap[I], EpilogLatch]
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  InsertPt->eraseFromParent();
  if (DT) {
    auto *NewDom = DT->findNearestCommonDominator(Exit, NewExit);
    DT->changeImmediateDominator(Exit, NewDom);
  }

  // In EpilogPreHeader, assume extra iters is non-zero.
  IRBuilder<> B2(EpilogPreHeader, EpilogPreHeader->getFirstNonPHIIt());
  Value *ModIsNotNull = B2.CreateIsNotNull(ModVal, "lcmp.mod");
  AssumeInst *AI = cast<AssumeInst>(B2.CreateAssumption(ModIsNotNull));
  AC.registerAssumption(AI);
}

```
- EN: Core entities appearing here include hasBranchWeightMD, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasBranchWeightMD，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 399-494

```cpp
/// Create a clone of the blocks in a loop and connect them together. A new
/// loop will be created including all cloned blocks, and the iterator of the
/// new loop switched to count NewIter down to 0.
/// The cloned blocks should be inserted between InsertTop and InsertBot.
/// InsertTop should be new preheader, InsertBot new loop exit.
/// Returns the new cloned loop that is created.
static Loop *CloneLoopBlocks(Loop *L, Value *NewIter,
                             const bool UseEpilogRemainder,
                             const bool UnrollRemainder, BasicBlock *InsertTop,
                             BasicBlock *InsertBot, BasicBlock *Preheader,
                             std::vector<BasicBlock *> &NewBlocks,
                             LoopBlocksDFS &LoopBlocks, ValueToValueMapTy &VMap,
                             DominatorTree *DT, LoopInfo *LI, unsigned Count,
                             std::optional<unsigned> OriginalTripCount,
                             BranchProbability OriginalLoopProb) {
  StringRef suffix = UseEpilogRemainder ? "epil" : "prol";
  BasicBlock *Header = L->getHeader();
  BasicBlock *Latch = L->getLoopLatch();
  Function *F = Header->getParent();
  LoopBlocksDFS::RPOIterator BlockBegin = LoopBlocks.beginRPO();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          ExitWeight = 1;
          BackEdgeWeight = 0;
        }
        MDBuilder MDB(Builder.getContext());
        BranchWeights = MDB.createBranchWeights(BackEdgeWeight, ExitWeight);
      }
      CondBrInst *RemainderLoopLatch =
          Builder.CreateCondBr(IdxCmp, FirstLoopBB, InsertBot, BranchWeights);
      if (!OriginalLoopProb.isUnknown() && UseEpilogRemainder) {
        // Compute the total frequency of the original loop body from the
        // remainder iterations.  Once we've reached them, the first of them
        // always executes, so its frequency and probability are 1.
        double FreqRemIters = 1;
        if (Count > 2) {
```
- EN: Core entities appearing here include hasBranchWeightMD, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasBranchWeightMD，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 495-593

```cpp
          BranchProbability ProbReaching = BranchProbability::getOne();
          for (unsigned N = Count - 2; N >= 1; --N) {
            ProbReaching *= probOfNextInRemainder(OriginalLoopProb, N);
            FreqRemIters += ProbReaching.toDouble();
          }
        }
        // Solve for the loop probability that would produce that frequency.
        // Sum(i=0..inf)(Prob^i) = 1/(1-Prob) = FreqRemIters.
        BranchProbability Prob =
            BranchProbability::getBranchProbability(1 - 1 / FreqRemIters);
        setBranchProbability(RemainderLoopLatch, Prob, /*ForFirstTarget=*/true);
      }
      NewIdx->addIncoming(Zero, InsertTop);
      NewIdx->addIncoming(IdxNext, NewBB);
      LatchBR->eraseFromParent();
    }
  }

  // Change the incoming values to the ones defined in the preheader or
  // cloned loop.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    BasicBlock *LatchBB = L->getLoopLatch();
    assert(LatchBB && "Expected loop to have a latch");
    BasicBlock *NonLatchExitingBlock =
        (ExitingBlocks[0] == LatchBB) ? ExitingBlocks[1] : ExitingBlocks[0];
    auto BranchProb =
        llvm::getBranchProbability(NonLatchExitingBlock, OtherExits[0]);
    // If BranchProbability could not be extracted (returns unknown), then
    // don't return and do the check for deopt block.
    if (!BranchProb.isUnknown()) {
      auto Threshold = TTI->getPredictableBranchThreshold().getCompl();
      return BranchProb < Threshold;
    }
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 594-691

```cpp
  // We know that deoptimize blocks are rarely taken, which also implies the
  // branch leading to the deoptimize block is highly unlikely.
  return OtherExits[0]->getPostdominatingDeoptimizeCall();
  // TODO: These can be fine-tuned further to consider code size or deopt states
  // that are captured by the deoptimize exit block.
  // Also, we can extend this to support more cases, if we actually
  // know of kinds of multiexit loops that would benefit from unrolling.
}

/// Calculate ModVal = (BECount + 1) % Count on the abstract integer domain
/// accounting for the possibility of unsigned overflow in the 2s complement
/// domain. Preconditions:
/// 1) TripCount = BECount + 1 (allowing overflow)
/// 2) Log2(Count) <= BitWidth(BECount)
static Value *CreateTripRemainder(IRBuilder<> &B, Value *BECount,
                                  Value *TripCount, unsigned Count) {
  // Note that TripCount is BECount + 1.
  if (isPowerOf2_32(Count))
    // If the expression is zero, then either:
    //  1. There are no iterations to be run in the prolog/epilog loop.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    unsigned SCEVExpansionBudget, bool RuntimeUnrollMultiExit,
    Loop **ResultLoop, std::optional<unsigned> OriginalTripCount,
    BranchProbability OriginalLoopProb) {
  LLVM_DEBUG(dbgs() << "Trying runtime unrolling on Loop: \n");
  LLVM_DEBUG(L->dump());
  LLVM_DEBUG(UseEpilogRemainder ? dbgs() << "Using epilog remainder.\n"
                                : dbgs() << "Using prolog remainder.\n");

  // Make sure the loop is in canonical form.
  if (!L->isLoopSimplifyForm()) {
    LLVM_DEBUG(dbgs() << "Not in simplify form!\n");
    return false;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 692-790

```cpp
  // Guaranteed by LoopSimplifyForm.
  BasicBlock *Latch = L->getLoopLatch();
  BasicBlock *Header = L->getHeader();

  CondBrInst *LatchBR = dyn_cast<CondBrInst>(Latch->getTerminator());

  if (!LatchBR) {
    // The loop-rotate pass can be helpful to avoid this in many cases.
    LLVM_DEBUG(
        dbgs()
        << "Loop latch not terminated by a conditional branch.\n");
    return false;
  }

  unsigned ExitIndex = LatchBR->getSuccessor(0) == Header ? 1 : 0;
  BasicBlock *LatchExit = LatchBR->getSuccessor(ExitIndex);

  if (L->contains(LatchExit)) {
    // Cloning the loop basic blocks (`CloneLoopBlocks`) requires that one of the
    // targets of the Latch be an exit block out of the loop.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                   PreHeaderBR)) {
    LLVM_DEBUG(dbgs() << "High cost for expanding trip count scev!\n");
    return false;
  }

  // This constraint lets us deal with an overflowing trip count easily; see the
  // comment on ModVal below.
  if (Log2_32(Count) > BEWidth) {
    LLVM_DEBUG(
        dbgs()
        << "Count failed constraint on overflow trip count calculation.\n");
    return false;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 791-888

```cpp
  // Loop structure is the following:
  //
  // PreHeader
  //   Header
  //   ...
  //   Latch
  // LatchExit

  BasicBlock *NewPreHeader;
  BasicBlock *NewExit = nullptr;
  BasicBlock *PrologExit = nullptr;
  BasicBlock *EpilogPreHeader = nullptr;
  BasicBlock *PrologPreHeader = nullptr;

  if (UseEpilogRemainder) {
    // If epilog remainder
    // Split PreHeader to insert a branch around loop for unrolling.
    NewPreHeader = SplitBlock(PreHeader, PreHeader->getTerminator(), DT, LI);
    NewPreHeader->setName(PreHeader->getName() + ".new");
    // Split LatchExit to create phi nodes from branch above.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if ((!OtherExits.empty() || !SE->loopHasNoAbnormalExits(L)) &&
      !isGuaranteedNotToBeUndefOrPoison(TripCount, AC, PreHeaderBR, DT)) {
    TripCount = B.CreateFreeze(TripCount);
    BECount =
        B.CreateAdd(TripCount, Constant::getAllOnesValue(TripCount->getType()));
  } else {
    // If we don't need to freeze, use SCEVExpander for BECount as well, to
    // allow slightly better value reuse.
    BECount =
        Expander.expandCodeFor(BECountSC, BECountSC->getType(), PreHeaderBR);
  }

  Value * const ModVal = CreateTripRemainder(B, BECount, TripCount, Count);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 889-986

```cpp
  Value *BranchVal =
      UseEpilogRemainder ? B.CreateICmpULT(BECount,
                                           ConstantInt::get(BECount->getType(),
                                                            Count - 1)) :
                           B.CreateIsNotNull(ModVal, "lcmp.mod");
  BasicBlock *RemainderLoop =
      UseEpilogRemainder ? EpilogPreHeader : PrologPreHeader;
  BasicBlock *UnrollingLoop = UseEpilogRemainder ? NewPreHeader : PrologExit;
  // Branch to either remainder (extra iterations) loop or unrolling loop.
  MDNode *BranchWeights = nullptr;
  if ((OriginalLoopProb.isUnknown() || !UseEpilogRemainder) &&
      hasBranchWeightMD(*Latch->getTerminator())) {
    // Assume loop is nearly always entered.
    MDBuilder MDB(B.getContext());
    BranchWeights = MDB.createBranchWeights(EpilogHeaderWeights);
  }
  CondBrInst *UnrollingLoopGuard =
      B.CreateCondBr(BranchVal, RemainderLoop, UnrollingLoop, BranchWeights);
  if (!OriginalLoopProb.isUnknown() && UseEpilogRemainder) {
    // The original loop's first iteration always happens.  Compute the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
       if (Instruction *I = dyn_cast<Instruction>(V))
         if (L->contains(I))
           V = VMap.lookup(I);
       PN.addIncoming(V, cast<BasicBlock>(VMap[PredBB]));
     }
   }
#if defined(EXPENSIVE_CHECKS) && !defined(NDEBUG)
    for (BasicBlock *SuccBB : successors(BB)) {
      assert(!(llvm::is_contained(OtherExits, SuccBB) || SuccBB == LatchExit) &&
             "Breaks the definition of dedicated exits!");
    }
#endif
  }

```
- EN: Core entities appearing here include hasBranchWeightMD, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasBranchWeightMD，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 987-1083

```cpp
  // Update the immediate dominator of the exit blocks and blocks that are
  // reachable from the exit blocks. This is needed because we now have paths
  // from both the original loop and the remainder code reaching the exit
  // blocks. While the IDom of these exit blocks were from the original loop,
  // now the IDom is the preheader (which decides whether the original loop or
  // remainder code should run) unless the block still has just the original
  // predecessor (such as NewExit in the case of an epilog remainder).
  if (DT && !L->getExitingBlock()) {
    SmallVector<BasicBlock *, 16> ChildrenToUpdate;
    // NB! We have to examine the dom children of all loop blocks, not just
    // those which are the IDom of the exit blocks. This is because blocks
    // reachable from the exit blocks can have their IDom as the nearest common
    // dominator of the exit blocks.
    for (auto *BB : L->blocks()) {
      auto *DomNodeBB = DT->getNode(BB);
      for (auto *DomChild : DomNodeBB->children()) {
        auto *DomChildBB = DomChild->getBlock();
        if (!L->contains(LI->getLoopFor(DomChildBB)) &&
            DomChildBB->getUniquePredecessor() != BB)
          ChildrenToUpdate.push_back(DomChildBB);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  // If this loop is nested, then the loop unroller changes the code in the any
  // of its parent loops, so the Scalar Evolution pass needs to be run again.
  SE->forgetTopmostLoop(L);

  // Verify that the Dom Tree and Loop Info are correct.
#if defined(EXPENSIVE_CHECKS) && !defined(NDEBUG)
  if (DT) {
    assert(DT->verify(DominatorTree::VerificationLevel::Full));
    LI->verify(*DT);
  }
#endif

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1084-1151

```cpp
  // For unroll factor 2 remainder loop will have 1 iteration.
  if (Count == 2 && DT && LI && SE) {
    // TODO: This code could probably be pulled out into a helper function
    // (e.g. breakLoopBackedgeAndSimplify) and reused in loop-deletion.
    BasicBlock *RemainderLatch = remainderLoop->getLoopLatch();
    assert(RemainderLatch);
    SmallVector<BasicBlock *> RemainderBlocks(remainderLoop->getBlocks());
    breakLoopBackedge(remainderLoop, *DT, *SE, *LI, nullptr);
    remainderLoop = nullptr;

    // Simplify loop values after breaking the backedge
    const DataLayout &DL = L->getHeader()->getDataLayout();
    SmallVector<WeakTrackingVH, 16> DeadInsts;
    for (BasicBlock *BB : RemainderBlocks) {
      for (Instruction &Inst : llvm::make_early_inc_range(*BB)) {
        if (Value *V = simplifyInstruction(&Inst, {DL, nullptr, DT, AC}))
          if (LI->replacementPreservesLCSSAForm(&Inst, V))
            Inst.replaceAllUsesWith(V);
        if (isInstructionTriviallyDead(&Inst))
          DeadInsts.emplace_back(&Inst);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    ULO.AllowExpensiveTripCount = false;
    ULO.UnrollRemainder = false;
    ULO.ForgetAllSCEV = ForgetAllSCEV;
    assert(!getLoopConvergenceHeart(L) &&
           "A loop with a convergence heart does not allow runtime unrolling.");
    UnrollResult = UnrollLoop(remainderLoop, ULO, LI, SE, DT, AC, TTI,
                              /*ORE*/ nullptr, PreserveLCSSA);
  }

  if (ResultLoop && UnrollResult != LoopUnrollResult::FullyUnrolled)
    *ResultLoop = remainderLoop;
  NumRuntimeUnrolled++;
  return true;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `probOfNextInRemainder, hasBranchWeightMD` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`probOfNextInRemainder, hasBranchWeightMD` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `UnrollRuntimeMultiExit, UnrollRuntimeOtherExitPredictable` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `UnrollRuntimeMultiExit, UnrollRuntimeOtherExitPredictable` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h`, `llvm/Transforms/Utils/UnrollLoop.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h`, `llvm/Transforms/Utils/UnrollLoop.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
