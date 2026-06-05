# SimpleLoopUnswitch.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/SimpleLoopUnswitch.cpp` | `llvm/lib/Transforms/Scalar/SimpleLoopUnswitch.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements /===- SimpleLoopUnswitch.cpp - Hoist loop-invariant control flow === within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 SimpleLoopUnswitch 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-241

```cpp
///===- SimpleLoopUnswitch.cpp - Hoist loop-invariant control flow ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/SimpleLoopUnswitch.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Sequence.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Analysis/AssumptionCache.h"
#include "llvm/Analysis/BlockFrequencyInfo.h"
#include "llvm/Analysis/CFG.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }

      // If not an instruction with the same opcode, nothing we can do.
      Instruction *OpI = dyn_cast<Instruction>(skipTrivialSelect(OpV));

      if (OpI && ((IsRootAnd && match(OpI, m_LogicalAnd())) ||
                  (IsRootOr  && match(OpI, m_LogicalOr())))) {
        // Visit this operand.
        if (Visited.insert(OpI).second)
          Worklist.push_back(OpI);
      }
    }
  } while (!Worklist.empty());

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 242-484

```cpp
  return Invariants;
}

static void replaceLoopInvariantUses(const Loop &L, Value *Invariant,
                                     Constant &Replacement) {
  assert(!isa<Constant>(Invariant) && "Why are we unswitching on a constant?");

  // Replace uses of LIC in the loop with the given constant.
  // We use make_early_inc_range as set invalidates the iterator.
  for (Use &U : llvm::make_early_inc_range(Invariant->uses())) {
    Instruction *UserI = dyn_cast<Instruction>(U.getUser());

    // Replace this use within the loop body.
    if (UserI && L.contains(UserI))
      U.set(&Replacement);
  }
}

/// Check that all the LCSSA PHI nodes in the loop exit block have trivial
/// incoming values along this edge.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                 MemorySSAUpdater *MSSAU, ScalarEvolution *SE) {
  // If the loop is already at the top level, we can't hoist it anywhere.
  Loop *OldParentL = L.getParentLoop();
  if (!OldParentL)
    return;

  SmallVector<BasicBlock *, 4> Exits;
  L.getExitBlocks(Exits);
  Loop *NewParentL = nullptr;
  for (auto *ExitBB : Exits)
    if (Loop *ExitL = LI.getLoopFor(ExitBB))
      if (!NewParentL || NewParentL->contains(ExitL))
        NewParentL = ExitL;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 485-725

```cpp
  if (NewParentL == OldParentL)
    return;

  // The new parent loop (if different) should always contain the old one.
  if (NewParentL)
    assert(NewParentL->contains(OldParentL) &&
           "Can only hoist this loop up the nest!");

  // The preheader will need to move with the body of this loop. However,
  // because it isn't in this loop we also need to update the primary loop map.
  assert(OldParentL == LI.getLoopFor(&Preheader) &&
         "Parent loop of this loop should contain this loop's preheader!");
  LI.changeLoopFor(&Preheader, NewParentL);

  // Remove this loop from its old parent.
  OldParentL->removeChildLoop(&L);

  // Add the loop either to the new parent or as a top-level loop.
  if (NewParentL)
    NewParentL->addChildLoop(&L);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        FreezeLoopUnswitchCond, OldPH->getTerminatorOrNull(), nullptr, DT, BI);
  }

  // Update the dominator tree with the added edge.
  DT.insertEdge(OldPH, UnswitchedBB);

  // After the dominator tree was updated with the added edge, update MemorySSA
  // if available.
  if (MSSAU) {
    SmallVector<CFGUpdate, 1> Updates;
    Updates.push_back({cfg::UpdateKind::Insert, OldPH, UnswitchedBB});
    MSSAU->applyInsertUpdates(Updates, DT);
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, MemorySSA, MemorySSAUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, MemorySSA, MemorySSAUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 726-961

```cpp
  // Finish updating dominator tree and memory ssa for full unswitch.
  if (FullUnswitch) {
    if (MSSAU) {
      Instruction *Term = ParentBB->getTerminator();
      // Remove the cloned branch instruction and create unconditional branch
      // now.
      Instruction *NewBI = UncondBrInst::Create(ContinueBB, ParentBB);
      NewBI->setDebugLoc(Term->getDebugLoc());
      Term->eraseFromParent();
      MSSAU->removeEdge(ParentBB, LoopExitBB);
    }
    DT.deleteEdge(ParentBB, LoopExitBB);
  }

  if (MSSAU && VerifyMemorySSA)
    MSSAU->getMemorySSA()->verifyMemorySSA();

  // Rewrite the relevant PHI nodes.
  if (UnswitchedBB == LoopExitBB)
    rewritePHINodesForUnswitchedExitBlock(*UnswitchedBB, *ParentBB, *OldPH);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      auto *SplitBB =
          SplitBlock(DefaultExitBB, DefaultExitBB->begin(), &DT, &LI, MSSAU);
      rewritePHINodesForExitAndUnswitchedBlocks(*DefaultExitBB, *SplitBB,
                                                *ParentBB, *OldPH,
                                                /*FullUnswitch*/ true);
      DefaultExitBB = SplitExitBBMap[DefaultExitBB] = SplitBB;
    }
  }
  // Note that we must use a reference in the for loop so that we update the
  // container.
  for (auto &ExitCase : reverse(ExitCases)) {
    // Grab a reference to the exit block in the pair so that we can update it.
    BasicBlock *ExitBB = std::get<1>(ExitCase);

```
- EN: Core entities appearing here include all_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, MemorySSA, MemorySSAUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, MemorySSA, MemorySSAUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 962-1199

```cpp
    // If this case is the last edge into the exit block, we can simply reuse it
    // as it will no longer be a loop exit. No mapping necessary.
    if (pred_empty(ExitBB)) {
      // Only rewrite once.
      if (UnswitchedExitBBs.insert(ExitBB).second)
        rewritePHINodesForUnswitchedExitBlock(*ExitBB, *ParentBB, *OldPH);
      continue;
    }

    // Otherwise we need to split the exit block so that we retain an exit
    // block from the loop and a target for the unswitched condition.
    BasicBlock *&SplitExitBB = SplitExitBBMap[ExitBB];
    if (!SplitExitBB) {
      // If this is the first time we see this, do the split and remember it.
      SplitExitBB = SplitBlock(ExitBB, ExitBB->begin(), &DT, &LI, MSSAU);
      rewritePHINodesForExitAndUnswitchedBlocks(*ExitBB, *SplitExitBB,
                                                *ParentBB, *OldPH,
                                                /*FullUnswitch*/ true);
    }
    // Update the case pair to point to the split block.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (isa<CondBrInst>(CurrentBB->getTerminator()))
      return Changed;

    // Follow the newly unconditional branch into its successor.
    CurrentBB = cast<UncondBrInst>(CurrentBB->getTerminator())->getSuccessor();

    // When continuing, if we exit the loop or reach a previous visited block,
    // then we can not reach any trivial condition candidates (unfoldable
    // branch instructions or switch instructions) and no unswitch can happen.
  } while (L.contains(CurrentBB) && Visited.insert(CurrentBB).second);

  return Changed;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, MemorySSA, MemorySSAUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, MemorySSA, MemorySSAUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1200-1441

```cpp
/// Build the cloned blocks for an unswitched copy of the given loop.
///
/// The cloned blocks are inserted before the loop preheader (`LoopPH`) and
/// after the split block (`SplitBB`) that will be used to select between the
/// cloned and original loop.
///
/// This routine handles cloning all of the necessary loop blocks and exit
/// blocks including rewriting their instructions and the relevant PHI nodes.
/// Any loop blocks or exit blocks which are dominated by a different successor
/// than the one for this clone of the loop blocks can be trivially skipped. We
/// use the `DominatingSucc` map to determine whether a block satisfies that
/// property with a simple map lookup.
///
/// It also correctly creates the unconditional branch in the cloned
/// unswitched parent block to only point at the unswitched successor.
///
/// This does not handle most of the necessary updates to `LoopInfo`. Only exit
/// block splitting is correctly reflected in `LoopInfo`, essentially all of
/// the cloned blocks (and their loops) are left without full `LoopInfo`
/// updates. This also doesn't fully update `DominatorTree`. It adds the cloned
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (LI.getLoopFor(BB) == &OrigL)
        LI.changeLoopFor(ClonedBB, &ClonedL);
    }
  };

  // We specially handle the first loop because it may get cloned into
  // a different parent and because we most commonly are cloning leaf loops.
  Loop *ClonedRootL = LI.AllocateLoop();
  if (RootParentL)
    RootParentL->addChildLoop(ClonedRootL);
  else
    LI.addTopLevelLoop(ClonedRootL);
  AddClonedBlocksToLoop(OrigRootL, *ClonedRootL);

```
- EN: Core entities appearing here include prev, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, LoopNest. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 prev，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, LoopNest 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1442-1681

```cpp
  if (OrigRootL.isInnermost())
    return ClonedRootL;

  // If we have a nest, we can quickly clone the entire loop nest using an
  // iterative approach because it is a tree. We keep the cloned parent in the
  // data structure to avoid repeatedly querying through a map to find it.
  SmallVector<std::pair<Loop *, Loop *>, 16> LoopsToClone;
  // Build up the loops to clone in reverse order as we'll clone them from the
  // back.
  for (Loop *ChildL : llvm::reverse(OrigRootL))
    LoopsToClone.push_back({ClonedRootL, ChildL});
  do {
    Loop *ClonedParentL, *L;
    std::tie(ClonedParentL, L) = LoopsToClone.pop_back_val();
    Loop *ClonedL = LI.AllocateLoop();
    ClonedParentL->addChildLoop(ClonedL);
    AddClonedBlocksToLoop(*L, *ClonedL);
    for (Loop *ChildL : llvm::reverse(*L))
      LoopsToClone.push_back({ClonedL, ChildL});
  } while (!LoopsToClone.empty());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        if (!UnloopedBlockSet.erase(PredBB)) {
          assert(
              (BlocksInClonedLoop.count(PredBB) || ExitLoopMap.count(PredBB)) &&
              "Predecessor not mapped to a loop!");
          continue;
        }

        // We just insert into the loop set here. We'll add these blocks to the
        // exit loop after we build up the set in an order that doesn't rely on
        // predecessor order (which in turn relies on use list order).
        bool Inserted = ExitLoopMap.insert({PredBB, ExitL}).second;
        (void)Inserted;
        assert(Inserted && "Should only visit an unlooped block once!");

```
- EN: Core entities appearing here include sort, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo, LoopNest. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 sort，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo, LoopNest 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1682-1925

```cpp
        // And recurse through to its predecessors.
        Worklist.push_back(PredBB);
      }
    } while (!Worklist.empty());
  }

  // Now that the ExitLoopMap gives as  mapping for all the non-looping cloned
  // blocks to their outer loops, walk the cloned blocks and the cloned exits
  // in their original order adding them to the correct loop.

  // We need a stable insertion order. We use the order of the original loop
  // order and map into the correct parent loop.
  for (auto *BB : llvm::concat<BasicBlock *const>(
           ArrayRef(ClonedPH), ClonedLoopBlocks, ClonedExitsInLoops))
    if (Loop *OuterL = ExitLoopMap.lookup(BB))
      OuterL->addBasicBlockToLoop(BB, LI);

#ifndef NDEBUG
  for (auto &BBAndL : ExitLoopMap) {
    auto *BB = BBAndL.first;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          continue;

        // Insert all of the blocks (other than those already present) into
        // the loop set. We expect at least the block that led us to find the
        // inner loop to be in the block set, but we may also have other loop
        // blocks if they were already enqueued as predecessors of some other
        // outer loop block.
        for (auto *InnerBB : InnerL->blocks()) {
          if (InnerBB == BB) {
            assert(LoopBlockSet.count(InnerBB) &&
                   "Block should already be in the set!");
            continue;
          }

```
- EN: Core entities appearing here include erase_if, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, LoopNest, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 erase_if，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, LoopNest, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1926-2164

```cpp
          LoopBlockSet.insert(InnerBB);
        }

        // Add the preheader to the worklist so we will continue past the
        // loop body.
        Worklist.push_back(InnerPH);
        continue;
      }

    // Insert any predecessors that were in the original loop into the new
    // set, and if the insert is successful, add them to the worklist.
    for (auto *Pred : predecessors(BB))
      if (L.contains(Pred) && LoopBlockSet.insert(Pred).second)
        Worklist.push_back(Pred);
  }

  assert(LoopBlockSet.count(Header) && "Cannot fail to add the header!");

  // We've found all the blocks participating in the loop, return our completed
  // set.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    assert(SubLoops.empty() &&
           "Failed to remove all subloops from the original loop!");
    if (Loop *ParentL = L.getParentLoop())
      ParentL->removeChildLoop(llvm::find(*ParentL, &L));
    else
      LI.removeLoop(llvm::find(LI, &L));
    // markLoopAsDeleted for L should be triggered by the caller (it is
    // typically done within postUnswitch).
    if (SE)
      SE->forgetBlockAndLoopDispositions();
    LI.destroy(&L);
    return false;
  }

```
- EN: Core entities appearing here include erase_if, stable_sort, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 erase_if, stable_sort，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2165-2400

```cpp
  return true;
}

/// Helper to visit a dominator subtree, invoking a callable on each node.
///
/// Returning false at any point will stop walking past that node of the tree.
template <typename CallableT>
void visitDomSubTree(DominatorTree &DT, BasicBlock *BB, CallableT Callable) {
  SmallVector<DomTreeNode *, 4> DomWorklist;
  DomWorklist.push_back(DT[BB]);
#ifndef NDEBUG
  SmallPtrSet<DomTreeNode *, 4> Visited;
  Visited.insert(DT[BB]);
#endif
  do {
    DomTreeNode *N = DomWorklist.pop_back_val();

    // Visit this node.
    if (!Callable(N->getBlock()))
      continue;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // The stitching of the branched code back together depends on whether we're
  // doing full unswitching or not with the exception that we always want to
  // nuke the initial terminator placed in the split block.
  SplitBB->getTerminator()->eraseFromParent();
  if (FullUnswitch) {
    // Keep a clone of the terminator for MSSA updates.
    Instruction *NewTI = TI.clone();
    NewTI->insertInto(ParentBB, ParentBB->end());

    // Splice the terminator from the original loop and rewrite its
    // successors.
    TI.moveBefore(*SplitBB, SplitBB->end());
    TI.dropLocation();

```
- EN: Core entities appearing here include visitDomSubTree, all_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitDomSubTree, all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2401-2644

```cpp
    // First wire up the moved terminator to the preheaders.
    if (BI) {
      BasicBlock *ClonedPH = ClonedPHs.begin()->second;
      BI->setSuccessor(ClonedSucc, ClonedPH);
      BI->setSuccessor(1 - ClonedSucc, LoopPH);
      Value *Cond = skipTrivialSelect(BI->getCondition());
      if (InsertFreeze) {
        // We don't give any debug location to the new freeze, because the
        // BI (`dyn_cast<CondBrInst>(TI)`) is an in-loop instruction hoisted
        // out of the loop.
        Cond = new FreezeInst(Cond, Cond->getName() + ".fr", BI->getIterator());
        cast<Instruction>(Cond)->setDebugLoc(DebugLoc::getDropped());
      }
      BI->setCondition(Cond);
      DTUpdates.push_back({DominatorTree::Insert, SplitBB, ClonedPH});
    } else {
      assert(SI && "Must either be a branch or switch!");

      // Walk the cases and directly update their successors.
      assert(SI->getDefaultDest() == RetainedSuccBB &&
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto UpdateLoop = [&](Loop &UpdateL) {
#ifndef NDEBUG
    UpdateL.verifyLoop();
    for (Loop *ChildL : UpdateL) {
      ChildL->verifyLoop();
      assert(ChildL->isRecursivelyLCSSAForm(DT, LI) &&
             "Perturbed a child loop's LCSSA form!");
    }
#endif
    // First build LCSSA for this loop so that we can preserve it when
    // forming dedicated exits. We don't want to perturb some other loop's
    // LCSSA while doing that CFG edit.
    formLCSSA(UpdateL, DT, &LI, SE);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2645-2889

```cpp
    // For loops reached by this loop's original exit blocks we may
    // introduced new, non-dedicated exits. At least try to re-form dedicated
    // exits for these loops. This may fail if they couldn't have dedicated
    // exits to start with.
    formDedicatedExitBlocks(&UpdateL, &DT, &LI, MSSAU, /*PreserveLCSSA*/ true);
  };

  // For non-child cloned loops and hoisted loops, we just need to update LCSSA
  // and we can do it in any order as they don't nest relative to each other.
  //
  // Also check if any of the loops we have updated have become top-level loops
  // as that will necessitate widening the outer loop scope.
  for (Loop *UpdatedL :
       llvm::concat<Loop *>(NonChildClonedLoops, HoistedLoops)) {
    UpdateLoop(*UpdatedL);
    if (UpdatedL->isOutermost())
      OuterExitL = nullptr;
  }
  if (IsStillLoop) {
    UpdateLoop(L);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // another path to the latch remaining that does not allow to eliminate the
  // loop copy on unswitch).
  const BasicBlock *Latch = L.getLoopLatch();
  const BasicBlock *CondBlock = TI.getParent();
  if (DT.dominates(CondBlock, Latch) &&
      (isGuard(&TI) ||
       (TI.isTerminator() &&
        llvm::count_if(successors(&TI), [&L](const BasicBlock *SuccBB) {
          return L.contains(SuccBB);
        }) <= 1))) {
    NumCostMultiplierSkipped++;
    return 1;
  }

```
- EN: Core entities appearing here include count_if, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, DomTreeUpdater, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 count_if，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, DomTreeUpdater, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2890-3127

```cpp
  // Each invariant non-trivial condition, after being unswitched, is supposed
  // to have its own specialized sibling loop (the invariant condition has been
  // hoisted out of the child loop into a newly-cloned loop). When unswitching
  // conditions in nested loops, the basic block size of the outer loop should
  // not be altered. If such a size significantly increases across unswitching
  // invocations, something may be wrong; so adjust the final cost taking this
  // into account.
  auto *ParentL = L.getParentLoop();
  int ParentLoopSizeMultiplier = 1;
  if (ParentL)
    ParentLoopSizeMultiplier =
        std::max<int>(ParentL->getNumBlocks() / UnswitchParentBlocksDiv, 1);

  int SiblingsCount =
      (ParentL ? ParentL->getSubLoopsVector().size() : llvm::size(LI));
  // Count amount of clones that all the candidates might cause during
  // unswitching. Branch/guard/select counts as 1, switch counts as log2 of its
  // cases.
  int UnswitchedClones = 0;
  for (const auto &Candidate : UnswitchCandidates) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  assert(Weights.size() == 2 && "Unexpected profile data!");
  size_t Idx = BI->getSuccessor(0) == TakenSucc ? 0 : 1;
  auto Num = Weights[Idx];
  auto Denom = Weights[0] + Weights[1];
  // Degenerate or overflowed metadata.
  if (Denom == 0 || Num > Denom)
    return false;
  BranchProbability ActualTaken(Num, Denom);
  if (LikelyTaken > ActualTaken)
    return false;
  return true;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, LoopInfo, MemorySSA, MemorySSAUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, LoopInfo, MemorySSA, MemorySSAUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3128-3366

```cpp
/// Materialize pending invariant condition of the given candidate into IR. The
/// injected loop-invariant condition implies the original loop-variant branch
/// condition, so the materialization turns
///
/// loop_block:
///   ...
///   br i1 %variant_cond, label InLoopSucc, label OutOfLoopSucc
///
/// into
///
/// preheader:
///   %invariant_cond = LHS pred RHS
/// ...
/// loop_block:
///   br i1 %invariant_cond, label InLoopSucc, label OriginalCheck
/// OriginalCheck:
///   br i1 %variant_cond, label InLoopSucc, label OutOfLoopSucc
/// ...
static NonTrivialUnswitchCandidate
injectPendingInvariantConditions(NonTrivialUnswitchCandidate Candidate, Loop &L,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
    }

  // Check if there are irreducible CFG cycles in this loop. If so, we cannot
  // easily unswitch non-trivial edges out of the loop. Doing so might turn the
  // irreducible control flow into reducible control flow and introduce new
  // loops "out of thin air". If we ever discover important use cases for doing
  // this, we can add support to loop unswitch, but it is a lot of complexity
  // for what seems little or no real world benefit.
  LoopBlocksRPO RPOT(&L);
  RPOT.perform(&LI);
  if (containsIrreducibleCFG<const BasicBlock *>(RPOT, LI))
    return false;

```
- EN: Core entities appearing here include isSafeForNoNTrivialUnswitching, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isSafeForNoNTrivialUnswitching，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3367-3605

```cpp
  SmallVector<BasicBlock *, 4> ExitBlocks;
  L.getUniqueExitBlocks(ExitBlocks);
  // We cannot unswitch if exit blocks contain a cleanuppad/catchswitch
  // instruction as we don't know how to split those exit blocks.
  // FIXME: We should teach SplitBlock to handle this and remove this
  // restriction.
  for (auto *ExitBB : ExitBlocks) {
    auto It = ExitBB->getFirstNonPHIIt();
    if (isa<CleanupPadInst>(It) || isa<CatchSwitchInst>(It)) {
      LLVM_DEBUG(dbgs() << "Cannot unswitch because of cleanuppad/catchswitch "
                           "in exit block\n");
      return false;
    }
  }

  return true;
}

static NonTrivialUnswitchCandidate findBestNonTrivialUnswitchCandidate(
    ArrayRef<NonTrivialUnswitchCandidate> UnswitchCandidates, const Loop &L,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (*Best.Cost >= UnswitchThreshold) {
    LLVM_DEBUG(dbgs() << "Cannot unswitch, lowest cost found: " << *Best.Cost
                      << "\n");
    return false;
  }

  bool InjectedCondition = false;
  if (Best.hasPendingInjection()) {
    Best = injectPendingInvariantConditions(Best, L, DT, LI, AC, MSSAU);
    InjectedCondition = true;
  }
  assert(!Best.hasPendingInjection() &&
         "All injections should have been done by now!");

```
- EN: Core entities appearing here include all_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3606-3760

```cpp
  if (Best.TI != PartialIVCondBranch)
    PartialIVInfo.InstToDuplicate.clear();

  bool InsertFreeze;
  if (auto *SI = dyn_cast<SelectInst>(Best.TI)) {
    // If the best candidate is a select, turn it into a branch. Select
    // instructions with a poison conditional do not propagate poison, but
    // branching on poison causes UB. Insert a freeze on the select
    // conditional to prevent UB after turning the select into a branch.
    InsertFreeze = !isGuaranteedNotToBeUndefOrPoison(
        SI->getCondition(), &AC, L.getLoopPreheader()->getTerminator(), &DT);
    Best.TI = turnSelectIntoBranch(SI, DT, LI, MSSAU, &AC);
  } else {
    // If the best candidate is a guard, turn it into a branch.
    if (isGuard(Best.TI))
      Best.TI =
          turnGuardIntoBranch(cast<IntrinsicInst>(Best.TI), L, DT, LI, MSSAU);
    InsertFreeze = shouldInsertFreeze(L, *Best.TI, DT, AC);
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    PA.preserve<MemorySSAAnalysis>();
  return PA;
}

void SimpleLoopUnswitchPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<SimpleLoopUnswitchPass> *>(this)->printPipeline(
      OS, MapClassName2PassName);

  OS << '<';
  OS << (NonTrivial ? "" : "no-") << "nontrivial;";
  OS << (Trivial ? "" : "no-") << "trivial";
  OS << '>';
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `CompareDesc, InjectedInvariant, NonTrivialUnswitchCandidate, all_of, prev, sort, erase_if, stable_sort` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`CompareDesc, InjectedInvariant, NonTrivialUnswitchCandidate, all_of, prev, sort, erase_if, stable_sort` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AssumptionCache, BlockFrequencyInfo, DominatorTree, DomTreeUpdater, LoopInfo, LoopNest, MemorySSA` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AssumptionCache, BlockFrequencyInfo, DominatorTree, DomTreeUpdater, LoopInfo, LoopNest, MemorySSA` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnableNonTrivialUnswitch, UnswitchThreshold, EnableUnswitchCostMultiplier, UnswitchSiblingsToplevelDiv, UnswitchParentBlocksDiv` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnableNonTrivialUnswitch, UnswitchThreshold, EnableUnswitchCostMultiplier, UnswitchSiblingsToplevelDiv, UnswitchParentBlocksDiv` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/CFG.h`, `llvm/Analysis/CodeMetrics.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GuardUtils.h`, `llvm/Analysis/LoopAnalysisManager.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/MemorySSA.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/CFG.h`, `llvm/Analysis/CodeMetrics.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GuardUtils.h`, `llvm/Analysis/LoopAnalysisManager.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/MemorySSA.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/GenericDomTree.h`, `llvm/Support/InstructionCost.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Sequence.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/GenericDomTree.h`, `llvm/Support/InstructionCost.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Sequence.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `iterator`, `numeric`, `optional`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `iterator`, `numeric`, `optional`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AssumptionCache`, `BlockFrequencyInfo`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `LoopNest`, `MemorySSA` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AssumptionCache`, `BlockFrequencyInfo`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `LoopNest`, `MemorySSA` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
