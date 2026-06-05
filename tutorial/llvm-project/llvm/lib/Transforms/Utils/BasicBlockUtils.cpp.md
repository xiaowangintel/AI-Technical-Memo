# BasicBlockUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/BasicBlockUtils.cpp` | `llvm/lib/Transforms/Utils/BasicBlockUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements basicBlock Utilities within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 BasicBlockUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-121

```cpp
//===- BasicBlockUtils.cpp - BasicBlock Utilities --------------------------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This family of functions perform manipulations on basic blocks, and
// instructions contained within basic blocks.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Analysis/CFG.h"
#include "llvm/Analysis/DomTreeUpdater.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  SmallPtrSet<BasicBlock *, 4> UniqueEHRetBlocksToDelete;
  for (auto *BB : BBs) {
    auto NonFirstPhiIt = BB->getFirstNonPHIIt();
    if (NonFirstPhiIt != BB->end()) {
      Instruction &I = *NonFirstPhiIt;
      // Exception handling funclets need to be explicitly addressed.
      // These funclets must begin with cleanuppad or catchpad and end with
      // cleanupred or catchret. The return instructions can be in different
      // basic blocks than the pad instruction. If we would only delete the
      // first block, the we would have possible cleanupret and catchret
      // instructions with poison arguments, which wouldn't be valid.
      if (isa<FuncletPadInst>(I)) {
        UniqueEHRetBlocksToDelete.clear();

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 122-242

```cpp
        for (User *User : I.users()) {
          Instruction *ReturnInstr = dyn_cast<Instruction>(User);
          // If we have a cleanupret or catchret block, replace it with just an
          // unreachable. The other alternative, that may use a catchpad is a
          // catchswitch. That does not need special handling for now.
          if (isa<CatchReturnInst>(ReturnInstr) ||
              isa<CleanupReturnInst>(ReturnInstr)) {
            BasicBlock *ReturnInstrBB = ReturnInstr->getParent();
            UniqueEHRetBlocksToDelete.insert(ReturnInstrBB);
          }
        }

        for (BasicBlock *EHRetBB : UniqueEHRetBlocksToDelete)
          emptyAndDetachBlock(EHRetBB, Updates, KeepOneInputPHIs);
      }
    }

    UniqueEHRetBlocksToDelete.clear();

    // Detaching and emptying the current basic block.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                     LoopInfo *LI, MemorySSAUpdater *MSSAU,
                                     MemoryDependenceResults *MemDep,
                                     bool PredecessorWithTwoSuccessors,
                                     DominatorTree *DT) {
  if (BB->hasAddressTaken())
    return false;

  // Can't merge if there are multiple predecessors, or no predecessors.
  BasicBlock *PredBB = BB->getUniquePredecessor();
  if (!PredBB) return false;

  // Don't break self-loops.
  if (PredBB == BB) return false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemoryDependenceResults. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemoryDependenceResults 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 243-363

```cpp
  // Don't break unwinding instructions or terminators with other side-effects.
  Instruction *PTI = PredBB->getTerminator();
  if (PTI->isSpecialTerminator() || PTI->mayHaveSideEffects())
    return false;

  // Can't merge if there are multiple distinct successors.
  if (!PredecessorWithTwoSuccessors && PredBB->getUniqueSuccessor() != BB)
    return false;

  // Currently only allow PredBB to have two predecessors, one being BB.
  // Update BI to branch to BB's only successor instead of BB.
  CondBrInst *PredBB_BI;
  BasicBlock *NewSucc = nullptr;
  unsigned FallThruPath;
  if (PredecessorWithTwoSuccessors) {
    if (!(PredBB_BI = dyn_cast<CondBrInst>(PTI)))
      return false;
    UncondBrInst *BB_JmpI = dyn_cast<UncondBrInst>(BB->getTerminator());
    if (!BB_JmpI)
      return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Add unreachable to now empty BB.
    new UnreachableInst(BB->getContext(), BB);

    // Update branch in the predecessor.
    PredBB_BI->setSuccessor(FallThruPath, NewSucc);
  } else {
    // Delete the unconditional branch from the predecessor.
    PredBB->back().eraseFromParent();

    // Move terminator instruction.
    BB->back().moveBeforePreserving(*PredBB, PredBB->end());
    // Add unreachable to now empty BB.
    new UnreachableInst(BB->getContext(), BB);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 364-475

```cpp
    // Terminator may be a memory accessing instruction too.
    if (MSSAU)
      if (MemoryUseOrDef *MUD = cast_or_null<MemoryUseOrDef>(
              MSSAU->getMemorySSA()->getMemoryAccess(PredBB->getTerminator())))
        MSSAU->moveToPlace(MUD, PredBB, MemorySSA::End);
  }

  // Inherit predecessors name if it exists.
  if (!PredBB->hasName())
    PredBB->takeName(BB);

  if (LI)
    LI->removeBlock(BB);

  if (MemDep)
    MemDep->invalidateCachedPredecessors();

  if (DTU)
    DTU->applyUpdates(Updates);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      ToBeRemoved.push_back(&DVR);
    }
    // Sequence with consecutive dbg.value instrs ended. Clear the map to
    // restart identifying redundant instructions if case we find another
    // dbg.value sequence.
    VariableSet.clear();
  }

  for (auto &DVR : ToBeRemoved)
    DVR->eraseFromParent();

  return !ToBeRemoved.empty();
}

```
- EN: Core entities appearing here include removeRedundantDbgInstrsUsingBackwardScan, reverse, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 removeRedundantDbgInstrsUsingBackwardScan, reverse，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 476-604

```cpp
/// Remove redundant dbg.value instructions using a forward scan. This can
/// remove a dbg.value instruction that is redundant due to indicating that a
/// variable has the same value as already being indicated by an earlier
/// dbg.value.
///
/// ForwardScan strategy:
/// ---------------------
/// Given two identical dbg.value instructions, separated by a block of
/// instructions that isn't describing the same variable, like this
///
///   dbg.value X1, "x", FragmentX1  (**)
///   <block of instructions, none being "dbg.value ..., "x", ...">
///   dbg.value X1, "x", FragmentX1  (*)
///
/// then the instruction marked with (*) can be removed. Variable "x" is already
/// described as being mapped to the SSA value X1.
///
/// Possible improvements:
/// - Keep track of non-overlapping fragments.
static bool removeRedundantDbgInstrsUsingForwardScan(BasicBlock *BB) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //   (1) dbg.value V1, "x", DIExpression()
  //       ...
  //   (2) dbg.value V2, "x", DIExpression()
  //   (3) dbg.value V1, "x", DIExpression()
  //
  // The backward scan will remove (2), it is made obsolete by (3). After
  // getting (2) out of the way, the foward scan will remove (3) since "x"
  // already is described as having the value V1 at (1).
  MadeChanges |= removeRedundantDbgInstrsUsingBackwardScan(BB);
  if (BB->isEntryBlock() &&
      isAssignmentTrackingEnabled(*BB->getParent()->getParent()))
    MadeChanges |= removeUndefDbgAssignsFromEntryBlock(BB);
  MadeChanges |= removeRedundantDbgInstrsUsingForwardScan(BB);

```
- EN: Core entities appearing here include removeRedundantDbgInstrsUsingForwardScan, make_early_inc_range, removeUndefDbgAssignsFromEntryBlock, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 removeRedundantDbgInstrsUsingForwardScan, make_early_inc_range, removeUndefDbgAssignsFromEntryBlock，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 605-724

```cpp
  if (MadeChanges)
    LLVM_DEBUG(dbgs() << "Removed redundant dbg instrs from: "
                      << BB->getName() << "\n");
  return MadeChanges;
}

void llvm::ReplaceInstWithValue(BasicBlock::iterator &BI, Value *V) {
  Instruction &I = *BI;
  // Replaces all of the uses of the instruction with uses of the value
  I.replaceAllUsesWith(V);

  // Make sure to propagate a name if there is one already.
  if (I.hasName() && !V->hasName())
    V->takeName(&I);

  // Delete the unnecessary instruction now...
  BI = BI->eraseFromParent();
}

void llvm::ReplaceInstWithInst(BasicBlock *BB, BasicBlock::iterator &BI,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  T *LC;
  if constexpr (std::is_same_v<TI, CycleInfo>)
    LC = LCI->getSmallestCommonCycle(CallBrBlock, Succ);
  else
    LC = LCI->getSmallestCommonLoop(CallBrBlock, Succ);
  if (!LC)
    return false;

  if constexpr (std::is_same_v<TI, CycleInfo>)
    LCI->addBlockToCycle(CallBrTarget, LC);
  else
    LC->addBasicBlockToLoop(CallBrTarget, *LCI);

```
- EN: Core entities appearing here include ReplaceInstWithValue, IsBlockFollowedByDeoptOrUnreachable, ReplaceInstWithInst, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ReplaceInstWithValue, IsBlockFollowedByDeoptOrUnreachable, ReplaceInstWithInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 725-851

```cpp
  return true;
}

BasicBlock *llvm::SplitCallBrEdge(BasicBlock *CallBrBlock, BasicBlock *Succ,
                                  unsigned SuccIdx, DomTreeUpdater *DTU,
                                  CycleInfo *CI, LoopInfo *LI,
                                  bool *UpdatedLI) {
  CallBrInst *CallBr = dyn_cast<CallBrInst>(CallBrBlock->getTerminator());
  assert(CallBr && "expected callbr terminator");
  assert(SuccIdx < CallBr->getNumSuccessors() &&
         Succ == CallBr->getSuccessor(SuccIdx) && "invalid successor index");

  // Create a new block between callbr and the specified successor.
  // splitBlockBefore cannot be re-used here since it cannot split if the split
  // point is a PHI node (because BasicBlock::splitBasicBlockBefore cannot
  // handle that). But we don't need to rewire every part of a potential PHI
  // node. We only care about the edge between CallBrBlock and the original
  // successor.
  BasicBlock *CallBrTarget =
      BasicBlock::Create(CallBrBlock->getContext(),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          break;
        }
        LoopPreds.push_back(P);
      }
      // Loop-simplify form can be preserved, if we can split all in-loop
      // predecessors.
      if (any_of(LoopPreds, [](BasicBlock *Pred) {
            return isa<IndirectBrInst>(Pred->getTerminator());
          })) {
        return nullptr;
      }
    }
  }

```
- EN: Core entities appearing here include setUnwindEdgeTo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 setUnwindEdgeTo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 852-970

```cpp
  auto *NewBB =
      BasicBlock::Create(BB->getContext(), BBName, BB->getParent(), Succ);
  setUnwindEdgeTo(BB->getTerminator(), NewBB);
  updatePhiNodes(Succ, BB, NewBB, LandingPadReplacement);

  if (LandingPadReplacement) {
    auto *NewLP = OriginalPad->clone();
    auto *Terminator = UncondBrInst::Create(Succ, NewBB);
    NewLP->insertBefore(Terminator->getIterator());
    LandingPadReplacement->addIncoming(NewLP, NewBB);
  } else {
    Value *ParentPad = nullptr;
    if (auto *FuncletPad = dyn_cast<FuncletPadInst>(PadInst))
      ParentPad = FuncletPad->getParentPad();
    else if (auto *CatchSwitch = dyn_cast<CatchSwitchInst>(PadInst))
      ParentPad = CatchSwitch->getParentPad();
    else if (auto *CleanupPad = dyn_cast<CleanupPadInst>(PadInst))
      ParentPad = CleanupPad->getParentPad();
    else if (auto *LandingPad = dyn_cast<LandingPadInst>(PadInst))
      ParentPad = LandingPad->getParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
         "SplitBB has non-PHI nodes!");

  // For each PHI in the destination block.
  for (PHINode &PN : DestBB->phis()) {
    int Idx = PN.getBasicBlockIndex(SplitBB);
    assert(Idx >= 0 && "Invalid Block Index");
    Value *V = PN.getIncomingValue(Idx);

    // If the input is a PHI which already satisfies LCSSA, don't create
    // a new one.
    if (const PHINode *VP = dyn_cast<PHINode>(V))
      if (VP->getParent() == SplitBB)
        continue;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 971-1107

```cpp
    // Otherwise a new PHI is needed. Create one and populate it.
    PHINode *NewPN = PHINode::Create(PN.getType(), Preds.size(), "split");
    BasicBlock::iterator InsertPos =
        SplitBB->isLandingPad() ? SplitBB->begin()
                                : SplitBB->getTerminator()->getIterator();
    NewPN->insertBefore(InsertPos);
    for (BasicBlock *BB : Preds)
      NewPN->addIncoming(V, BB);

    // Update the original PHI.
    PN.setIncomingValue(Idx, NewPN);
  }
}

unsigned
llvm::SplitAllCriticalEdges(Function &F,
                            const CriticalEdgeSplittingOptions &Options) {
  unsigned NumBroken = 0;
  for (BasicBlock &BB : F) {
    Instruction *TI = BB.getTerminator();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          Updates.push_back({DominatorTree::Delete, Pred, OldBB});
        }
      DTU->applyUpdates(Updates);
    }
  } else if (DT) {
    if (OldBB == DT->getRootNode()->getBlock()) {
      assert(NewBB->isEntryBlock());
      DT->setNewRoot(NewBB);
    } else {
      // Split block expects NewBB to have a non-empty set of predecessors.
      DT->splitBlock(NewBB);
    }
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1108-1235

```cpp
  // Update MemoryPhis after split if MemorySSA is available
  if (MSSAU)
    MSSAU->wireOldPredecessorsToNewImmediatePredecessor(OldBB, NewBB, Preds);

  // The rest of the logic is only relevant for updating the loop structures.
  if (!LI)
    return;

  if (DTU && DTU->hasDomTree())
    DT = &DTU->getDomTree();
  assert(DT && "DT should be available to update LoopInfo!");
  Loop *L = LI->getLoopFor(OldBB);

  // If we need to preserve loop analyses, collect some information about how
  // this split will affect loops.
  bool IsLoopEntry = !!L;
  bool SplitMakesNewLoopHeader = false;
  for (BasicBlock *Pred : Preds) {
    // Preds that are not reachable from entry should not be used to identify if
    // OldBB is a loop entry or if SplitMakesNewLoopHeader. Unreachable blocks
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (!HasLoopExit) {
      InVal = PN->getIncomingValueForBlock(Preds[0]);
      for (unsigned i = 0, e = PN->getNumIncomingValues(); i != e; ++i) {
        if (!PredSet.count(PN->getIncomingBlock(i)))
          continue;
        if (!InVal)
          InVal = PN->getIncomingValue(i);
        else if (InVal != PN->getIncomingValue(i)) {
          InVal = nullptr;
          break;
        }
      }
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1236-1366

```cpp
    if (InVal) {
      // If all incoming values for the new PHI would be the same, just don't
      // make a new PHI.  Instead, just remove the incoming values from the old
      // PHI.
      PN->removeIncomingValueIf(
          [&](unsigned Idx) {
            return PredSet.contains(PN->getIncomingBlock(Idx));
          },
          /* DeletePHIIfEmpty */ false);

      // Add an incoming value to the PHI node in the loop for the preheader
      // edge.
      PN->addIncoming(InVal, NewBB);
      continue;
    }

    // If the values coming into the block are not the same, we need a new
    // PHI.
    // Create the new PHI node, insert it into NewBB at the end of the block
    PHINode *NewPHI =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  if (OldLatch) {
    BasicBlock *NewLatch = L->getLoopLatch();
    if (NewLatch != OldLatch) {
      MDNode *MD = OldLatch->getTerminator()->getMetadata(LLVMContext::MD_loop);
      NewLatch->getTerminator()->setMetadata(LLVMContext::MD_loop, MD);
      // It's still possible that OldLatch is the latch of another inner loop,
      // in which case we do not remove the metadata.
      Loop *IL = LI->getLoopFor(OldLatch);
      if (IL && IL->getLoopLatch() != OldLatch)
        OldLatch->getTerminator()->setMetadata(LLVMContext::MD_loop, nullptr);
    }
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1367-1489

```cpp
  return NewBB;
}

BasicBlock *llvm::SplitBlockPredecessors(BasicBlock *BB,
                                         ArrayRef<BasicBlock *> Preds,
                                         const char *Suffix, DominatorTree *DT,
                                         LoopInfo *LI, MemorySSAUpdater *MSSAU,
                                         bool PreserveLCSSA) {
  return SplitBlockPredecessorsImpl(BB, Preds, Suffix, /*DTU=*/nullptr, DT, LI,
                                    MSSAU, PreserveLCSSA);
}
BasicBlock *llvm::SplitBlockPredecessors(BasicBlock *BB,
                                         ArrayRef<BasicBlock *> Preds,
                                         const char *Suffix,
                                         DomTreeUpdater *DTU, LoopInfo *LI,
                                         MemorySSAUpdater *MSSAU,
                                         bool PreserveLCSSA) {
  return SplitBlockPredecessorsImpl(BB, Preds, Suffix, DTU,
                                    /*DT=*/nullptr, LI, MSSAU, PreserveLCSSA);
}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      PHINode *PN = PHINode::Create(LPad->getType(), 2, "lpad.phi", LPad->getIterator());
      PN->addIncoming(Clone1, NewBB1);
      PN->addIncoming(Clone2, NewBB2);
      LPad->replaceAllUsesWith(PN);
    }
    LPad->eraseFromParent();
  } else {
    // There is no second clone. Just replace the landing pad with the first
    // clone.
    LPad->replaceAllUsesWith(Clone1);
    LPad->eraseFromParent();
  }
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1490-1609

```cpp
void llvm::SplitLandingPadPredecessors(BasicBlock *OrigBB,
                                       ArrayRef<BasicBlock *> Preds,
                                       const char *Suffix1, const char *Suffix2,
                                       SmallVectorImpl<BasicBlock *> &NewBBs,
                                       DomTreeUpdater *DTU, LoopInfo *LI,
                                       MemorySSAUpdater *MSSAU,
                                       bool PreserveLCSSA) {
  return SplitLandingPadPredecessorsImpl(OrigBB, Preds, Suffix1, Suffix2,
                                         NewBBs, DTU, /*DT=*/nullptr, LI, MSSAU,
                                         PreserveLCSSA);
}

ReturnInst *llvm::FoldReturnIntoUncondBranch(ReturnInst *RI, BasicBlock *BB,
                                             BasicBlock *Pred,
                                             DomTreeUpdater *DTU) {
  Instruction *UncondBranch = Pred->getTerminator();
  // Clone the return and add it to the end of the predecessor.
  Instruction *NewRet = RI->clone();
  NewRet->insertInto(Pred, Pred->end());

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  *ThenTerm = ThenBlock->getTerminator();
  *ElseTerm = ElseBlock->getTerminator();
}

void llvm::SplitBlockAndInsertIfThenElse(
    Value *Cond, BasicBlock::iterator SplitBefore, BasicBlock **ThenBlock,
    BasicBlock **ElseBlock, bool UnreachableThen, bool UnreachableElse,
    MDNode *BranchWeights, DomTreeUpdater *DTU, LoopInfo *LI) {
  assert((ThenBlock || ElseBlock) &&
         "At least one branch block must be created");
  assert((!UnreachableThen || !UnreachableElse) &&
         "Split block tail must be reachable");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1610-1732

```cpp
  SmallVector<DominatorTree::UpdateType, 8> Updates;
  SmallPtrSet<BasicBlock *, 8> UniqueOrigSuccessors;
  BasicBlock *Head = SplitBefore->getParent();
  if (DTU) {
    UniqueOrigSuccessors.insert_range(successors(Head));
    Updates.reserve(4 + 2 * UniqueOrigSuccessors.size());
  }

  LLVMContext &C = Head->getContext();
  BasicBlock *Tail = Head->splitBasicBlock(SplitBefore);
  BasicBlock *TrueBlock = Tail;
  BasicBlock *FalseBlock = Tail;
  bool ThenToTailEdge = false;
  bool ElseToTailEdge = false;

  // Encapsulate the logic around creation/insertion/etc of a new block.
  auto handleBlock = [&](BasicBlock **PBB, bool Unreachable, BasicBlock *&BB,
                         bool &ToTailEdge) {
    if (PBB == nullptr)
      return; // Do not create/insert a block.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      SplitBlockAndInsertSimpleForLoop(NumElements, InsertBefore);

    IRB.SetInsertPoint(BodyIP);
    Func(IRB, Index);
    return;
  }

  unsigned Num = EC.getFixedValue();
  for (unsigned Idx = 0; Idx < Num; ++Idx) {
    IRB.SetInsertPoint(InsertBefore);
    Func(IRB, ConstantInt::get(IndexTy, Idx));
  }
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1733-1858

```cpp
void llvm::SplitBlockAndInsertForEachLane(
    Value *EVL, BasicBlock::iterator InsertBefore,
    std::function<void(IRBuilderBase &, Value *)> Func) {

  IRBuilder<> IRB(InsertBefore->getParent(), InsertBefore);
  Type *Ty = EVL->getType();

  if (!isa<ConstantInt>(EVL)) {
    auto [BodyIP, Index] = SplitBlockAndInsertSimpleForLoop(EVL, InsertBefore);
    IRB.SetInsertPoint(BodyIP);
    Func(IRB, Index);
    return;
  }

  unsigned Num = cast<ConstantInt>(EVL)->getZExtValue();
  for (unsigned Idx = 0; Idx < Num; ++Idx) {
    IRB.SetInsertPoint(InsertBefore);
    Func(IRB, ConstantInt::get(Ty, Idx));
  }
}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  return BI;
}

void llvm::InvertBranch(CondBrInst *PBI, IRBuilderBase &Builder) {
  Value *NewCond = PBI->getCondition();
  // If this is a "cmp" instruction, only used for branching (and nowhere
  // else), then we can simply invert the predicate.
  if (NewCond->hasOneUse() && isa<CmpInst>(NewCond)) {
    CmpInst *CI = cast<CmpInst>(NewCond);
    CI->setPredicate(CI->getInversePredicate());
  } else
    NewCond = Builder.CreateNot(NewCond, NewCond->getName() + ".not");

```
- EN: Core entities appearing here include InvertBranch, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 InvertBranch，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1859-1880

```cpp
  PBI->setCondition(NewCond);
  PBI->swapSuccessors();
}

bool llvm::hasOnlySimpleTerminator(const Function &F) {
  for (auto &BB : F) {
    auto *Term = BB.getTerminator();
    if (!isa<ReturnInst, UnreachableInst, UncondBrInst, CondBrInst>(Term))
      return false;
  }
  return true;
}

Printable llvm::printBasicBlock(const BasicBlock *BB) {
  return Printable([BB](raw_ostream &OS) {
    if (!BB) {
      OS << "<nullptr>";
      return;
    }
    BB->printAsOperand(OS);
  });
}
```
- EN: Core entities appearing here include hasOnlySimpleTerminator, printBasicBlock, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 hasOnlySimpleTerminator, printBasicBlock，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `HasLoopOrEntryConvergenceToken, removeRedundantDbgInstrsUsingBackwardScan, reverse, removeRedundantDbgInstrsUsingForwardScan, make_early_inc_range, removeUndefDbgAssignsFromEntryBlock, RemoveRedundantDbgInstrs, ReplaceInstWithValue` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`HasLoopOrEntryConvergenceToken, removeRedundantDbgInstrsUsingBackwardScan, reverse, removeRedundantDbgInstrsUsingForwardScan, make_early_inc_range, removeUndefDbgAssignsFromEntryBlock, RemoveRedundantDbgInstrs, ReplaceInstWithValue` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, MemoryDependenceResults, MemorySSA, MemorySSAUpdater` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, MemoryDependenceResults, MemorySSA, MemorySSAUpdater` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `MaxDeoptOrUnreachableSuccessorCheckDepth` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `MaxDeoptOrUnreachableSuccessorCheckDepth` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/CFG.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemoryDependenceAnalysis.h`, `llvm/Analysis/MemorySSAUpdater.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/CFG.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemoryDependenceAnalysis.h`, `llvm/Analysis/MemorySSAUpdater.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/CycleInfo.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/CycleInfo.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstdint`, `string`, `utility`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstdint`, `string`, `utility`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `MemoryDependenceResults`, `MemorySSA`, `MemorySSAUpdater` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `MemoryDependenceResults`, `MemorySSA`, `MemorySSAUpdater` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
