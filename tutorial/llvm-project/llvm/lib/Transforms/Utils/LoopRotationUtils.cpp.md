# LoopRotationUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopRotationUtils.cpp` | `llvm/lib/Transforms/Utils/LoopRotationUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Loop Rotation Utils within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopRotationUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-85

```cpp
//===----------------- LoopRotationUtils.cpp -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides utilities to convert a loop into a loop with bottom test.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LoopRotationUtils.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/AssumptionCache.h"
#include "llvm/Analysis/CodeMetrics.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/InstructionSimplify.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/MemorySSA.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
             const SimplifyQuery &SQ, bool RotationOnly, bool IsUtilMode,
             bool PrepareForLTO, bool CheckExitCount)
      : MaxHeaderSize(MaxHeaderSize), LI(LI), TTI(TTI), AC(AC), DT(DT), SE(SE),
        MSSAU(MSSAU), SQ(SQ), RotationOnly(RotationOnly),
        IsUtilMode(IsUtilMode), PrepareForLTO(PrepareForLTO),
        CheckExitCount(CheckExitCount) {}
  bool processLoop(Loop *L);

private:
  bool rotateLoop(Loop *L, bool SimplifiedLatch);
  bool simplifyLoopLatch(Loop *L);
};
} // end anonymous namespace

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 86-166

```cpp
/// Insert (K, V) pair into the ValueToValueMap, and verify the key did not
/// previously exist in the map, and the value was inserted.
static void InsertNewValueIntoMap(ValueToValueMapTy &VM, Value *K, Value *V) {
  bool Inserted = VM.insert({K, V}).second;
  assert(Inserted);
  (void)Inserted;
}
/// RewriteUsesOfClonedInstructions - We just cloned the instructions from the
/// old header into the preheader.  If there were uses of the values produced by
/// these instruction that were outside of the loop, we have to insert PHI nodes
/// to merge the two values.  Do this now.
static void RewriteUsesOfClonedInstructions(BasicBlock *OrigHeader,
                                            BasicBlock *OrigPreheader,
                                            ValueToValueMapTy &ValueMap,
                                            ScalarEvolution *SE,
                                SmallVectorImpl<PHINode*> *InsertedPHIs) {
  // Remove PHI node entries that are no longer live.
  BasicBlock::iterator I, E = OrigHeader->end();
  for (I = OrigHeader->begin(); PHINode *PN = dyn_cast<PHINode>(I); ++I)
    PN->removeIncomingValue(OrigPreheader);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }

    // Replace MetadataAsValue(ValueAsMetadata(OrigHeaderVal)) uses in debug
    // intrinsics.
    SmallVector<DbgVariableRecord *, 1> DbgVariableRecords;
    llvm::findDbgValues(OrigHeaderVal, DbgVariableRecords);

    for (DbgVariableRecord *DVR : DbgVariableRecords) {
      // The original users in the OrigHeader are already using the original
      // definitions.
      BasicBlock *UserBB = DVR->getMarker()->getParent();
      if (UserBB == OrigHeader)
        continue;

```
- EN: Core entities appearing here include InsertNewValueIntoMap, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 InsertNewValueIntoMap，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 167-257

```cpp
      // Users in the OrigPreHeader need to use the value to which the
      // original definitions are mapped and anything else can be handled by
      // the SSAUpdater. To avoid adding PHINodes, check if the value is
      // available in UserBB, if not substitute poison.
      Value *NewVal;
      if (UserBB == OrigPreheader)
        NewVal = OrigPreHeaderVal;
      else if (SSA.HasValueForBlock(UserBB))
        NewVal = SSA.GetValueInMiddleOfBlock(UserBB);
      else
        NewVal = PoisonValue::get(OrigHeaderVal->getType());
      DVR->replaceVariableLocationOp(OrigHeaderVal, NewVal);
    }
  }
}

// Assuming both header and latch are exiting, check if rotating is profitable:
// either a header phi becomes dead, or rotating makes the latch exit count
// computable (enabling downstream optimizations like unrolling/vectorization).
static bool profitableToRotateLoopExitingLatch(Loop *L, ScalarEvolution *SE) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //           |   |            |   Br i1 ... |
  //           -----            |   |      |  |
  //                          x0| x1|   y1 |  |
  //                            V   V      ----
  //                            Exit
  //
  // The following must hold:
  //  -  x == x0 + x1        # counts to "exit" must stay the same.
  //  - y0 == x - x0 == x1   # how often loop was entered at all.
  //  - y1 == y - y0         # How often loop was repeated (after first iter.).
  //
  // We cannot generally deduce how often we had a zero-trip count loop so we
  // have to make a guess for how to distribute x among the new x0 and x1.

```
- EN: Core entities appearing here include profitableToRotateLoopExitingLatch, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 profitableToRotateLoopExitingLatch，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 258-338

```cpp
  uint32_t ExitWeight0;    // aka x0
  uint32_t ExitWeight1;    // aka x1
  uint32_t EnterWeight;    // aka y0
  uint32_t LoopBackWeight; // aka y1
  if (OrigLoopExitWeight > 0 && OrigLoopBackedgeWeight > 0) {
    ExitWeight0 = 0;
    if (HasConditionalPreHeader) {
      // Here we cannot know how many 0-trip count loops we have, so we guess:
      if (OrigLoopBackedgeWeight >= OrigLoopExitWeight) {
        // If the loop count is bigger than the exit count then we set
        // probabilities as if 0-trip count nearly never happens.
        ExitWeight0 = ZeroTripCountWeights[0];
        // Scale up counts if necessary so we can match `ZeroTripCountWeights`
        // for the `ExitWeight0`:`ExitWeight1` (aka `x0`:`x1` ratio`) ratio.
        while (OrigLoopExitWeight < ZeroTripCountWeights[1] + ExitWeight0) {
          // ... but don't overflow.
          uint32_t const HighBit = uint32_t{1} << (sizeof(uint32_t) * 8 - 1);
          if ((OrigLoopBackedgeWeight & HighBit) != 0 ||
              (OrigLoopExitWeight & HighBit) != 0)
            break;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  const uint32_t LoopBIWeights[] = {
      SuccsSwapped ? LoopBackWeight : ExitWeight1,
      SuccsSwapped ? ExitWeight1 : LoopBackWeight,
  };
  setBranchWeights(LoopBI, LoopBIWeights, /*IsExpected=*/false);
  if (HasConditionalPreHeader) {
    const uint32_t PreHeaderBIWeights[] = {
        SuccsSwapped ? EnterWeight : ExitWeight0,
        SuccsSwapped ? ExitWeight0 : EnterWeight,
    };
    setBranchWeights(PreHeaderBI, PreHeaderBIWeights, /*IsExpected=*/false);
  }
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 339-421

```cpp
/// Rotate loop LP. Return true if the loop is rotated.
///
/// \param SimplifiedLatch is true if the latch was just folded into the final
/// loop exit. In this case we may want to rotate even though the new latch is
/// now an exiting branch. This rotation would have happened had the latch not
/// been simplified. However, if SimplifiedLatch is false, then we avoid
/// rotating loops in which the latch exits to avoid excessive or endless
/// rotation. LoopRotate should be repeatable and converge to a canonical
/// form. This property is satisfied because simplifying the loop latch can only
/// happen once across multiple invocations of the LoopRotate pass.
bool LoopRotate::rotateLoop(Loop *L, bool SimplifiedLatch) {
  // If the loop has only one block then there is not much to rotate.
  if (L->getBlocks().size() == 1)
    return false;

  bool Rotated = false;
  BasicBlock *OrigHeader = L->getHeader();
  BasicBlock *OrigLatch = L->getLoopLatch();

  CondBrInst *BI = dyn_cast<CondBrInst>(OrigHeader->getTerminator());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                        << Metrics.NumInsts
                        << " instructions, which is more than the threshold ("
                        << MaxHeaderSize << " instructions): ";
                 L->dump());
      ++NumNotRotatedDueToHeaderSize;
      return Rotated;
    }

    // When preparing for LTO, avoid rotating loops with calls that could be
    // inlined during the LTO stage.
    if (PrepareForLTO && Metrics.NumInlineCandidates > 0)
      return Rotated;
  }

```
- EN: Core entities appearing here include rotateLoop, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 rotateLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 422-505

```cpp
  // Now, this loop is suitable for rotation.
  BasicBlock *OrigPreheader = L->getLoopPreheader();

  // If the loop could not be converted to canonical form, it must have an
  // indirectbr in it, just give up.
  if (!OrigPreheader || !L->hasDedicatedExits())
    return Rotated;

  // Anything ScalarEvolution may know about this loop or the PHI nodes
  // in its header will soon be invalidated. We should also invalidate
  // all outer loops because insertion and deletion of blocks that happens
  // during the rotation may violate invariants related to backedge taken
  // infos in them.
  if (SE) {
    SE->forgetTopmostLoop(L);
    // We may hoist some instructions out of loop. In case if they were cached
    // as "loop variant" or "loop computable", these caches must be dropped.
    // We also may fold basic blocks, so cached block dispositions also need
    // to be dropped.
    SE->forgetBlockAndLoopDispositions();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Build DbgVariableRecord hashes for DbgVariableRecords attached to the
  // terminator.
  for (const DbgVariableRecord &DVR :
       filterDbgVars(OrigPreheader->getTerminator()->getDbgRecordRange()))
    DbgRecords.insert(makeHash(&DVR));

  // Remember the local noalias scope declarations in the header. After the
  // rotation, they must be duplicated and the scope must be cloned. This
  // avoids unwanted interaction across iterations.
  SmallVector<NoAliasScopeDeclInst *, 6> NoAliasDeclInstructions;
  for (Instruction &I : *OrigHeader)
    if (auto *Decl = dyn_cast<NoAliasScopeDeclInst>(&I))
      NoAliasDeclInstructions.push_back(Decl);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as MemorySSA, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 MemorySSA, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 506-593

```cpp
  Module *M = OrigHeader->getModule();

  // Track the next DbgRecord to clone. If we have a sequence where an
  // instruction is hoisted instead of being cloned:
  //    DbgRecord blah
  //    %foo = add i32 0, 0
  //    DbgRecord xyzzy
  //    %bar = call i32 @foobar()
  // where %foo is hoisted, then the DbgRecord "blah" will be seen twice, once
  // attached to %foo, then when %foo his hoisted it will "fall down" onto the
  // function call:
  //    DbgRecord blah
  //    DbgRecord xyzzy
  //    %bar = call i32 @foobar()
  // causing it to appear attached to the call too.
  //
  // To avoid this, cloneDebugInfoFrom takes an optional "start cloning from
  // here" position to account for this behaviour. We point it at any
  // DbgRecords on the next instruction, here labelled xyzzy, before we hoist
  // %foo. Later, we only only clone DbgRecords from that position (xyzzy)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    ++NumInstrsDuplicated;

    if (!NextDbgInsts.empty()) {
      auto Range = C->cloneDebugInfoFrom(Inst, NextDbgInsts.begin());
      RemapDbgRecordRange(M, Range, ValueMap,
                          RF_NoModuleLevelChanges | RF_IgnoreMissingLocals);
      NextDbgInsts = DbgMarker::getEmptyDbgRecordRange();
      // Erase anything we've seen before.
      for (DbgVariableRecord &DVR : make_early_inc_range(filterDbgVars(Range)))
        if (DbgRecords.count(makeHash(&DVR)))
          DVR.eraseFromParent();
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 594-684

```cpp
    // Eagerly remap the operands of the instruction.
    RemapInstruction(C, ValueMap,
                     RF_NoModuleLevelChanges | RF_IgnoreMissingLocals);

    // With the operands remapped, see if the instruction constant folds or is
    // otherwise simplifyable.  This commonly occurs because the entry from PHI
    // nodes allows icmps and other instructions to fold.
    Value *V = simplifyInstruction(C, SQ);
    if (V && LI->replacementPreservesLCSSAForm(C, V)) {
      // If so, then delete the temporary instruction and stick the folded value
      // in the map.
      InsertNewValueIntoMap(ValueMap, Inst, V);
      if (!C->mayHaveSideEffects()) {
        C->eraseFromParent();
        C = nullptr;
      }
    } else {
      InsertNewValueIntoMap(ValueMap, Inst, C);
    }
    if (C) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

      // Keep the compile time impact low by only adapting the inserted block
      // of instructions in the OrigPreHeader. This might result in slightly
      // more aliasing between these instructions and those that were already
      // present, but it will be much faster when the original PreHeader is
      // large.
      LLVM_DEBUG(dbgs() << "  Updating part of OrigPreheader scopes\n");
      auto *FirstDecl =
          cast<Instruction>(ValueMap[*NoAliasDeclInstructions.begin()]);
      auto *LastInst = &OrigPreheader->back();
      cloneAndAdaptNoAliasScopes(NoAliasDeclScopes, FirstDecl, LastInst,
                                 Context, "pre.rot");
      LLVM_DEBUG(OrigPreheader->dump());

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 685-772

```cpp
      LLVM_DEBUG(dbgs() << "  Updated NewHeader:\n");
      LLVM_DEBUG(NewHeader->dump());
    }
  }

  // Along with all the other instructions, we just cloned OrigHeader's
  // terminator into OrigPreHeader. Fix up the PHI nodes in each of OrigHeader's
  // successors by duplicating their incoming values for OrigHeader.
  for (BasicBlock *SuccBB : successors(OrigHeader))
    for (BasicBlock::iterator BI = SuccBB->begin();
         PHINode *PN = dyn_cast<PHINode>(BI); ++BI)
      PN->addIncoming(PN->getIncomingValueForBlock(OrigHeader), OrigPreheader);

  // Now that OrigPreHeader has a clone of OrigHeader's terminator, remove
  // OrigPreHeader's old terminator (the original branch into the loop), and
  // remove the corresponding incoming values from the PHI nodes in OrigHeader.
  LoopEntryBranch->eraseFromParent();
  OrigPreheader->flushTerminatorDbgRecords();

  // Update MemorySSA before the rewrite call below changes the 1:1
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  updateBranchWeights(*PHBI, *BI, HasConditionalPreHeader, BISuccsSwapped);

  if (HasConditionalPreHeader) {
    // The conditional branch can't be folded, handle the general case.
    // Split edges as necessary to preserve LoopSimplify form.

    // Right now OrigPreHeader has two successors, NewHeader and ExitBlock, and
    // thus is not a preheader anymore.
    // Split the edge to form a real preheader.
    BasicBlock *NewPH = SplitCriticalEdge(
        OrigPreheader, NewHeader,
        CriticalEdgeSplittingOptions(DT, LI, MSSAU).setPreserveLCSSA());
    NewPH->setName(NewHeader->getName() + ".lr.ph");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 773-851

```cpp
    // Preserve canonical loop form, which means that 'Exit' should have only
    // one predecessor. Note that Exit could be an exit block for multiple
    // nested loops, causing both of the edges to now be critical and need to
    // be split.
    SmallVector<BasicBlock *, 4> ExitPreds(predecessors(Exit));
    bool SplitLatchEdge = false;
    for (BasicBlock *ExitPred : ExitPreds) {
      // We only need to split loop exit edges.
      Loop *PredLoop = LI->getLoopFor(ExitPred);
      if (!PredLoop || PredLoop->contains(Exit) ||
          isa<IndirectBrInst>(ExitPred->getTerminator()))
        continue;
      SplitLatchEdge |= L->getLoopLatch() == ExitPred;
      BasicBlock *ExitSplit = SplitCriticalEdge(
          ExitPred, Exit,
          CriticalEdgeSplittingOptions(DT, LI, MSSAU).setPreserveLCSSA());
      ExitSplit->moveBefore(Exit);
    }
    assert(SplitLatchEdge &&
           "Despite splitting all preds, failed to split latch exit?");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// conversions.
static bool shouldSpeculateInstrs(BasicBlock::iterator Begin,
                                  BasicBlock::iterator End, Loop *L) {
  bool seenIncrement = false;
  bool MultiExitLoop = false;

  if (!L->getExitingBlock())
    MultiExitLoop = true;

  for (BasicBlock::iterator I = Begin; I != End; ++I) {

    if (!isSafeToSpeculativelyExecute(&*I))
      return false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 852-934

```cpp
    switch (I->getOpcode()) {
    default:
      return false;
    case Instruction::GetElementPtr:
      // GEPs are cheap if all indices are constant.
      if (!cast<GEPOperator>(I)->hasAllConstantIndices())
        return false;
      // fall-thru to increment case
      [[fallthrough]];
    case Instruction::Add:
    case Instruction::Sub:
    case Instruction::And:
    case Instruction::Or:
    case Instruction::Xor:
    case Instruction::Shl:
    case Instruction::LShr:
    case Instruction::AShr: {
      Value *IVOpnd =
          !isa<Constant>(I->getOperand(0))
              ? I->getOperand(0)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  if (!isa<UncondBrInst, CondBrInst>(LastExit->getTerminator()))
    return false;

  if (!shouldSpeculateInstrs(Latch->begin(), Jmp->getIterator(), L))
    return false;

  LLVM_DEBUG(dbgs() << "Folding loop latch " << Latch->getName() << " into "
                    << LastExit->getName() << "\n");

  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Eager);
  MergeBlockIntoPredecessor(Latch, &DTU, LI, MSSAU, nullptr,
                            /*PredecessorWithTwoSuccessors=*/true);

```
- EN: Core entities appearing here include simplifyLoopLatch, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 simplifyLoopLatch，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 935-983

```cpp
    if (SE) {
      // Merging blocks may remove blocks reference in the block disposition cache. Clear the cache.
      SE->forgetBlockAndLoopDispositions();
    }

  if (MSSAU && VerifyMemorySSA)
    MSSAU->getMemorySSA()->verifyMemorySSA();

  return true;
}

/// Rotate \c L, and return true if any modification was made.
bool LoopRotate::processLoop(Loop *L) {
  // Save the loop metadata.
  MDNode *LoopMD = L->getLoopID();

  bool SimplifiedLatch = false;

  // Simplify the loop latch before attempting to rotate the header
  // upward. Rotation may not be needed if the loop tail can be folded into the
  // loop exit.
  if (!RotationOnly)
    SimplifiedLatch = simplifyLoopLatch(L);

  bool MadeChange = rotateLoop(L, SimplifiedLatch);
  assert((!MadeChange || L->isLoopExiting(L->getLoopLatch())) &&
         "Loop latch should be exiting after loop-rotate.");

  // Restore the loop metadata.
  // NB! We presume LoopRotation DOESN'T ADD its own metadata.
  if ((MadeChange || SimplifiedLatch) && LoopMD)
    L->setLoopID(LoopMD);

  return MadeChange || SimplifiedLatch;
}


/// The utility to convert a loop into a loop with bottom test.
bool llvm::LoopRotation(Loop *L, LoopInfo *LI, const TargetTransformInfo *TTI,
                        AssumptionCache *AC, DominatorTree *DT,
                        ScalarEvolution *SE, MemorySSAUpdater *MSSAU,
                        const SimplifyQuery &SQ, bool RotationOnly = true,
                        unsigned Threshold = unsigned(-1),
                        bool IsUtilMode = true, bool PrepareForLTO,
                        bool CheckExitCount) {
  LoopRotate LR(Threshold, LI, TTI, AC, DT, SE, MSSAU, SQ, RotationOnly,
                IsUtilMode, PrepareForLTO, CheckExitCount);
  return LR.processLoop(L);
}
```
- EN: Core entities appearing here include processLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 processLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LoopRotate, InsertNewValueIntoMap, profitableToRotateLoopExitingLatch, rotateLoop, simplifyLoopLatch, processLoop` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LoopRotate, InsertNewValueIntoMap, profitableToRotateLoopExitingLatch, rotateLoop, simplifyLoopLatch, processLoop` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA, MemorySSAUpdater, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA, MemorySSAUpdater, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CodeMetrics.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CodeMetrics.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/CFG.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LoopRotationUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/SSAUpdater.h`, `llvm/Transforms/Utils/ValueMapper.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/CFG.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LoopRotationUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/SSAUpdater.h`, `llvm/Transforms/Utils/ValueMapper.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `MemorySSA`, `MemorySSAUpdater`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `MemorySSA`, `MemorySSAUpdater`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
