# LoopUnroll.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopUnroll.cpp` | `llvm/lib/Transforms/Utils/LoopUnroll.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements === UnrollLoop.cpp - Loop unrolling utilities === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopUnroll 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-112

```cpp
//===-- UnrollLoop.cpp - Loop unrolling utilities -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements some loop unrolling utilities. It does not define any
// actual pass or policy, but provides a single function to perform loop
// unrolling.
//
// The process of unrolling can produce extraneous basic blocks linked with
// unconditional branches.  This will be corrected in the future.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/MapVector.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    cl::init(false)
#endif
                    );

static cl::opt<bool>
UnrollVerifyLoopInfo("unroll-verify-loopinfo", cl::Hidden,
                    cl::desc("Verify loopinfo after unrolling"),
#ifdef EXPENSIVE_CHECKS
    cl::init(true)
#else
    cl::init(false)
#endif
                    );

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 113-223

```cpp
static cl::opt<bool> UnrollAddParallelReductions(
    "unroll-add-parallel-reductions", cl::init(false), cl::Hidden,
    cl::desc("Allow unrolling to add parallel reduction phis."));

/// Check if unrolling created a situation where we need to insert phi nodes to
/// preserve LCSSA form.
/// \param Blocks is a vector of basic blocks representing unrolled loop.
/// \param L is the outer loop.
/// It's possible that some of the blocks are in L, and some are not. In this
/// case, if there is a use is outside L, and definition is inside L, we need to
/// insert a phi-node, otherwise LCSSA will be broken.
/// The function is just a helper function for llvm::UnrollLoop that returns
/// true if this situation occurs, indicating that LCSSA needs to be fixed.
static bool needToInsertPhisForLCSSA(Loop *L,
                                     const std::vector<BasicBlock *> &Blocks,
                                     LoopInfo *LI) {
  for (BasicBlock *BB : Blocks) {
    if (LI->getLoopFor(BB) == L)
      continue;
    for (Instruction &I : *BB) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (isa<ConstantInt>(PN.getIncomingValueForBlock(PreHeader)))
      return true;
  }
  return false;
}

struct LoadValue {
  Instruction *DefI = nullptr;
  unsigned Generation = 0;
  LoadValue() = default;
  LoadValue(Instruction *Inst, unsigned Generation)
      : DefI(Inst), Generation(Generation) {}
};

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include isEpilogProfitable, LoadValue, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 isEpilogProfitable, LoadValue，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。

### Lines 224-341

```cpp
class StackNode {
  ScopedHashTable<const SCEV *, LoadValue>::ScopeTy LoadScope;
  unsigned CurrentGeneration;
  unsigned ChildGeneration;
  DomTreeNode *Node;
  DomTreeNode::const_iterator ChildIter;
  DomTreeNode::const_iterator EndIter;
  bool Processed = false;

public:
  StackNode(ScopedHashTable<const SCEV *, LoadValue> &AvailableLoads,
            unsigned cg, DomTreeNode *N, DomTreeNode::const_iterator Child,
            DomTreeNode::const_iterator End)
      : LoadScope(AvailableLoads), CurrentGeneration(cg), ChildGeneration(cg),
        Node(N), ChildIter(Child), EndIter(End) {}
  // Accessors.
  unsigned currentGeneration() const { return CurrentGeneration; }
  unsigned childGeneration() const { return ChildGeneration; }
  void childGeneration(unsigned generation) { ChildGeneration = generation; }
  DomTreeNode *node() { return Node; }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      DomTreeNode *Child = NodeToProcess->nextChild();
      if (!L->contains(Child->getBlock()))
        continue;
      NodesToProcess.emplace_back(
          new StackNode(AvailableLoads, NodeToProcess->childGeneration(), Child,
                        Child->begin(), Child->end()));
    } else {
      // It has been processed, and there are no more children to process,
      // so delete it and pop it off the stack.
      NodesToProcess.pop_back();
    }
  }
}

```
- EN: Core entities appearing here include StackNode, getMatchingValue, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 StackNode, getMatchingValue，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 342-443

```cpp
/// Perform some cleanup and simplifications on loops after unrolling. It is
/// useful to simplify the IV's in the new loop, as well as do a quick
/// simplify/dce pass of the instructions.
void llvm::simplifyLoopAfterUnroll(Loop *L, bool SimplifyIVs, LoopInfo *LI,
                                   ScalarEvolution *SE, DominatorTree *DT,
                                   AssumptionCache *AC,
                                   const TargetTransformInfo *TTI,
                                   ArrayRef<BasicBlock *> Blocks,
                                   AAResults *AA) {
  using namespace llvm::PatternMatch;

  // Simplify any new induction variables in the partially unrolled loop.
  if (SE && SimplifyIVs) {
    SmallVector<WeakTrackingVH, 16> DeadInsts;
    simplifyLoopIVs(L, SE, DT, LI, TTI, DeadInsts);

    // Aggressively clean up dead instructions that simplifyLoopIVs already
    // identified. Any remaining should be cleaned up below.
    while (!DeadInsts.empty()) {
      Value *V = DeadInsts.pop_back_val();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Check for uncontrolled convergent operations.
  for (auto &BB : L->blocks()) {
    for (auto &I : *BB) {
      if (isa<ConvergenceControlInst>(I))
        return true;
      if (auto *CB = dyn_cast<CallBase>(&I))
        if (CB->isConvergent())
          return CB->getConvergenceControlToken();
    }
  }
  return true;
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include loadCSE, canHaveUnrollRemainder, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DataLayout, DominatorTree.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 loadCSE, canHaveUnrollRemainder，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DataLayout, DominatorTree 等分析结果。

### Lines 444-560

```cpp
// If LoopUnroll has proven OriginalLoopProb is incorrect for some iterations
// of the original loop, adjust latch probabilities in the unrolled loop to
// maintain the original total frequency of the original loop body.
//
// OriginalLoopProb is practical but imprecise
// -------------------------------------------
//
// The latch branch weights that LLVM originally adds to a loop encode one latch
// probability, OriginalLoopProb, applied uniformly across the loop's infinite
// set of theoretically possible iterations.  While this uniform latch
// probability serves as a practical statistic summarizing the trip counts
// observed during profiling, it is imprecise.  Specifically, unless it is zero,
// it is impossible for it to be the actual probability observed at every
// individual iteration.  To see why, consider that the only way to actually
// observe at run time that the latch probability remains non-zero is to profile
// at least one loop execution that has an infinite number of iterations.  I do
// not know how to profile an infinite number of loop iterations, and most loops
// I work with are always finite.
//
// LoopUnroll proves OriginalLoopProb is incorrect
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // course, OriginalLoopProb could be based on inaccurate profile data, but
  // there is nothing we can do about that here.
  if (CondLatches.empty())
    return;

  // If the original latch probability is 1, the original frequency is infinity.
  // Leaving all remaining probabilities set to 1 might or might not get us
  // there (e.g., a completely unrolled loop cannot be infinite), but it is the
  // closest we can come.
  assert(!OriginalLoopProb.isUnknown() &&
         "Expected to have loop probability to fix");
  if (OriginalLoopProb.isOne())
    return;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 561-663

```cpp
  // FreqDesired is the frequency implied by the original loop probability.
  double FreqDesired = 1 / (1 - OriginalLoopProb.toDouble());

  // Get the probability at CondLatches[I].
  auto GetProb = [&](unsigned I) {
    CondBrInst *B = cast<CondBrInst>(CondLatches[I]->getTerminator());
    bool FirstTargetIsNext = B->getSuccessor(0) == CondLatchNexts[I];
    return getBranchProbability(B, FirstTargetIsNext).toDouble();
  };

  // Set the probability at CondLatches[I] to Prob.
  auto SetProb = [&](unsigned I, double Prob) {
    CondBrInst *B = cast<CondBrInst>(CondLatches[I]->getTerminator());
    bool FirstTargetIsNext = B->getSuccessor(0) == CondLatchNexts[I];
    setBranchProbability(B, BranchProbability::getBranchProbability(Prob),
                         FirstTargetIsNext);
  };

  // Set all probabilities in CondLatches to Prob.
  auto SetAllProbs = [&](double Prob) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // - FreqBefore or FreqAfter is the total frequency accumulated before or
  //   after CondLatches[I], respectively, while the probability at
  //   CondLatches[I] is treated as 1.
  //
  // If ComputeIdx == 0, then ComputeProb will set those values for I == 0 and
  // ignore the current values.  If ComputeIdx > 0, then it expects those values
  // to already be set for I == ComputeIdx - 1, and it will set them for I ==
  // ComputeIdx.
  auto AdjustProb = [&](unsigned ComputeIdx, double &ProbBefore,
                        double &ProbAfter, double &FreqBefore,
                        double &FreqAfter) {
    assert(ComputeIdx < CondLatches.size() &&
           "Expected valid CondLatches index");

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 664-775

```cpp
    // Compute or update ProbBefore, ProbAfter, FreqBefore, and FreqAfter.
    auto ComputeAfter = [&]() {
      ProbAfter = 1;
      FreqAfter = IterCounts[ComputeIdx + 1];
      for (unsigned I = ComputeIdx + 1, E = CondLatches.size(); I < E; ++I) {
        double Prob = GetProb(I);
        ProbAfter *= Prob;
        // After Prob == 0, ProbAfter and FreqAfter won't change, so save time.
        if (Prob == 0)
          break;
        FreqAfter += IterCounts[I + 1] * ProbAfter;
      }
    };
    if (ComputeIdx == 0) {
      ProbBefore = 1;
      FreqBefore = IterCounts[0];
      ComputeAfter();
    } else {
      // Rather than iterating all of CondLatches again, we fix up the
      // previously computed values.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        });
      }
    } else {
      assert(FreqBefore > 0 &&
             "Expected at least one iteration before first latch");
      // In this equation, if we replace the left-hand side with FreqDesired and
      // then solve for ProbCompute, we get the ProbCompute formula above.
      FreqCompute = (FreqBefore + FreqAfter * ProbCompute) /
                    (1 - ProbReachingBackedge * ProbCompute);
    }
    assert(FreqCompute > 0 && "Expected valid frequency");
    return FreqCompute;
  };

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 776-886

```cpp
  // Determine and set branch weights.
  if (CondLatches.size() == 1) {
    SetAllProbs(ComputeProbForLinear());
  } else if (CondLatches.size() == 2) {
    SetAllProbs(ComputeProbForQuadratic());
  } else {
    // The polynomial is too complex for a simple formula, so the quick and
    // dirty fix has been selected.  Adjust probabilities starting from the
    // first latch, which has the most influence on the total frequency, so
    // starting there should minimize the number of latches that have to be
    // visited.  We do have to iterate because the first latch alone might not
    // be enough.  For example, we might need to set all probabilities to 1 if
    // the frequency is the unroll factor.
    double ProbBefore = -1, ProbAfter = -1; // Inits expected to be unused.
    double FreqBefore = -1, FreqAfter = -1; // Inits expected to be unused.
    for (unsigned I = 0; I != CondLatches.size(); ++I) {
      double Freq = AdjustProb(I, ProbBefore, ProbAfter, FreqBefore, FreqAfter);
      if (fabs(Freq - FreqDesired) < FreqPrec)
        break;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // changed.
  BasicBlock *Preheader = L->getLoopPreheader();
  BasicBlock *Header = L->getHeader();
  BasicBlock *LatchBlock = L->getLoopLatch();
  SmallVector<BasicBlock *, 4> ExitBlocks;
  L->getExitBlocks(ExitBlocks);
  std::vector<BasicBlock *> OriginalLoopBlocks = L->getBlocks();

  const unsigned MaxTripCount = SE->getSmallConstantMaxTripCount(L);
  const bool MaxOrZero = SE->isBackedgeTakenCountMaxOrZero(L);
  std::optional<unsigned> OriginalTripCount =
      llvm::getLoopEstimatedTripCount(L);
  BranchProbability OriginalLoopProb = llvm::getLoopProbability(L);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 887-993

```cpp
  // Effectively "DCE" unrolled iterations that are beyond the max tripcount
  // and will never be executed.
  if (MaxTripCount && ULO.Count > MaxTripCount)
    ULO.Count = MaxTripCount;

  struct ExitInfo {
    unsigned TripCount;
    unsigned TripMultiple;
    unsigned BreakoutTrip;
    bool ExitOnTrue;
    BasicBlock *FirstExitingBlock = nullptr;
    SmallVector<BasicBlock *> ExitingBlocks;
  };
  MapVector<BasicBlock *, ExitInfo> ExitInfos;
  SmallVector<BasicBlock *, 4> ExitingBlocks;
  L->getExitingBlocks(ExitingBlocks);
  for (auto *ExitingBlock : ExitingBlocks) {
    // The folding code is not prepared to deal with non-branch instructions
    // right now.
    auto *BI = dyn_cast<CondBrInst>(ExitingBlock->getTerminator());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      ULO.Runtime = false;
    else {
      LLVM_DEBUG(dbgs() << "Won't unroll; remainder loop could not be "
                           "generated when assuming runtime trip count\n");
      return LoopUnrollResult::Unmodified;
    }
  }

  using namespace ore;

  // Determine whether this loop originated from the vectorizer so we can
  // produce more informative remarks.
  StringRef LoopKind = getLoopVectorizeKindPrefix(L);

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ExitInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ExitInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 994-1106

```cpp
  // Report the unrolling decision.
  if (CompletelyUnroll) {
    LLVM_DEBUG(dbgs() << "COMPLETELY UNROLLING loop %" << Header->getName()
                      << " with trip count " << ULO.Count << "!\n");
    if (ORE)
      ORE->emit([&]() {
        return OptimizationRemark(DEBUG_TYPE, "FullyUnrolled", L->getStartLoc(),
                                  L->getHeader())
               << "completely unrolled " + LoopKind.str() + "loop with "
               << NV("UnrollCount", ULO.Count) << " iterations";
      });
  } else {
    LLVM_DEBUG({
      dbgs() << "UNROLLING loop %" << Header->getName() << " by " << ULO.Count;
      if (ULO.Runtime) {
        dbgs() << " with run-time trip count";
        if (ULO.UnrollRemainder)
          dbgs() << " (remainder unrolled)";
      }
      dbgs() << "!\n";
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  std::vector<BasicBlock *> Latches;
  Headers.push_back(Header);
  Latches.push_back(LatchBlock);

  // The current on-the-fly SSA update requires blocks to be processed in
  // reverse postorder so that LastValueMap contains the correct value at each
  // exit.
  LoopBlocksDFS DFS(L);
  DFS.perform(LI);

  // Stash the DFS iterators before adding blocks to the loop.
  LoopBlocksDFS::RPOIterator BlockBegin = DFS.beginRPO();
  LoopBlocksDFS::RPOIterator BlockEnd = DFS.endRPO();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1107-1221

```cpp
  std::vector<BasicBlock*> UnrolledLoopBlocks = L->getBlocks();

  // Loop Unrolling might create new loops. While we do preserve LoopInfo, we
  // might break loop-simplified form for these loops (as they, e.g., would
  // share the same exit blocks). We'll keep track of loops for which we can
  // break this so that later we can re-simplify them.
  SmallSetVector<Loop *, 4> LoopsToSimplify;
  LoopsToSimplify.insert_range(*L);

  // When a FSDiscriminator is enabled, we don't need to add the multiply
  // factors to the discriminators.
  if (Header->getParent()->shouldEmitDebugInfoForProfiling() &&
      !EnableFSDiscriminator)
    for (BasicBlock *BB : L->getBlocks())
      for (Instruction &I : *BB)
        if (!I.isDebugOrPseudoInst())
          if (const DILocation *DIL = I.getDebugLoc()) {
            auto NewDIL = DIL->cloneByMultiplyingDuplicationFactor(ULO.Count);
            if (NewDIL)
              I.setDebugLoc(*NewDIL);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // Remap source location atom instance. Do this now, rather than
      // when we remap instructions, because remap is called once we've
      // cloned all blocks (all the clones would get the same atom
      // number).
      if (!VMap.AtomMap.empty())
        for (Instruction &I : *New)
          RemapSourceAtom(&I, VMap);

      // Update our running map of newest clones
      LastValueMap[*BB] = New;
      for (ValueToValueMapTy::iterator VI = VMap.begin(), VE = VMap.end();
           VI != VE; ++VI)
        LastValueMap[VI->first] = VI->second;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1222-1340

```cpp
      // Add phi entries for newly created values to all exit blocks.
      for (BasicBlock *Succ : successors(*BB)) {
        if (L->contains(Succ))
          continue;
        for (PHINode &PHI : Succ->phis()) {
          Value *Incoming = PHI.getIncomingValueForBlock(*BB);
          ValueToValueMapTy::iterator It = LastValueMap.find(Incoming);
          if (It != LastValueMap.end())
            Incoming = It->second;
          PHI.addIncoming(Incoming, New);
          SE->forgetLcssaPhiWithNewPredecessor(L, &PHI);
        }
      }
      // Keep track of new headers and latches as we create them, so that
      // we can insert the proper branches later.
      if (*BB == Header)
        Headers.push_back(New);
      if (*BB == LatchBlock)
        Latches.push_back(New);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        auto *ChildBB = ChildDomNode->getBlock();
        if (!L->contains(ChildBB))
          ChildrenToUpdate.push_back(ChildBB);
      }
      // The new idom of the block will be the nearest common dominator
      // of all copies of the previous idom. This is equivalent to the
      // nearest common dominator of the previous idom and the first latch,
      // which dominates all copies of the previous idom.
      BasicBlock *NewIDom = DT->findNearestCommonDominator(BB, LatchBlock);
      for (auto *ChildBB : ChildrenToUpdate)
        DT->changeImmediateDominator(ChildBB, NewIDom);
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1341-1456

```cpp
  assert(!UnrollVerifyDomtree ||
         DT->verify(DominatorTree::VerificationLevel::Fast));

  SmallVector<DominatorTree::UpdateType> DTUpdates;
  auto SetDest = [&](BasicBlock *Src, bool WillExit, bool ExitOnTrue) {
    auto *Term = cast<CondBrInst>(Src->getTerminator());
    const unsigned Idx = ExitOnTrue ^ WillExit;
    BasicBlock *Dest = Term->getSuccessor(Idx);
    BasicBlock *DeadSucc = Term->getSuccessor(1-Idx);

    // Remove predecessors from all non-Dest successors.
    DeadSucc->removePredecessor(Src, /* KeepOneInputPHIs */ true);

    // Replace the conditional branch with an unconditional one.
    auto *BI = UncondBrInst::Create(Dest, Term->getIterator());
    BI->setDebugLoc(Term->getDebugLoc());
    Term->eraseFromParent();

    DTUpdates.emplace_back(DominatorTree::Delete, Src, DeadSucc);
  };
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // no need to update the DT inside the unrolled loop.
    DTUToUse = nullptr;
    auto &[OriginalExit, Info] = *ExitInfos.begin();
    if (!Info.FirstExitingBlock)
      Info.FirstExitingBlock = Info.ExitingBlocks.back();
    for (auto *C : to_vector(DT->getNode(OriginalExit)->children())) {
      if (L->contains(C->getBlock()))
        continue;
      C->setIDom(DT->getNode(Info.FirstExitingBlock));
    }
  } else {
    DTU.applyUpdates(DTUpdates);
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1457-1566

```cpp
  // When completely unrolling, the last latch becomes unreachable.
  if (!LatchIsExiting && CompletelyUnroll) {
    // There is no need to update the DT here, because there must be a unique
    // latch. Hence if the latch is not exiting it must directly branch back to
    // the original loop header and does not dominate any nodes.
    assert(LatchBlock->getSingleSuccessor() && "Loop with multiple latches?");
    changeToUnreachable(Latches.back()->getTerminator(), PreserveLCSSA);
  }

  // After merging adjacent blocks in Latches below:
  // - CondLatches will list the blocks from Latches that are still terminated
  //   with conditional branches.
  // - For 1 <= I < CondLatches.size(), IterCounts[I] will store the number of
  //   the original loop iterations through which control flows from
  //   CondLatches[I-1] to CondLatches[I].
  // - For I == 0 or I == CondLatches.size(), IterCounts[I] will store the
  //   number of the original loop iterations through which control can flow
  //   before CondLatches.front() or after CondLatches.back(), respectively,
  //   without taking the unrolled loop's backedge, if any.
  // - CondLatchNexts[I] will store the CondLatches[I] branch target for the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  assert(!UnrollVerifyDomtree ||
         DT->verify(DominatorTree::VerificationLevel::Fast));

  Loop *OuterL = L->getParentLoop();
  std::vector<BasicBlock *> Blocks;
  // Update LoopInfo if the loop is completely removed.
  if (CompletelyUnroll) {
    Blocks = L->getBlocks();
    LI->erase(L);
    // We shouldn't try to use `L` anymore.
    L = nullptr;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1567-1679

```cpp
  // At this point, the code is well formed.  We now simplify the unrolled loop,
  // doing constant propagation and dead code elimination as we go.
  simplifyLoopAfterUnroll(
      L, !CompletelyUnroll && ULO.Count > 1, LI, SE, DT, AC, TTI,
      CompletelyUnroll ? ArrayRef<BasicBlock *>(Blocks) : L->getBlocks(), AA);

  NumCompletelyUnrolled += CompletelyUnroll;
  ++NumUnrolled;

  if (!CompletelyUnroll) {
    // Update metadata for the loop's branch weights and estimated trip count:
    // - If ULO.Runtime, UnrollRuntimeLoopRemainder sets the guard branch
    //   weights, latch branch weights, and estimated trip count of the
    //   remainder loop it creates.  It also sets the branch weights for the
    //   unrolled loop guard it creates.  The branch weights for the unrolled
    //   loop latch are adjusted below.  FIXME: Handle prologue loops.
    // - Otherwise, if unrolled loop iteration latches become unconditional,
    //   branch weights are adjusted by the fixProbContradiction call above.
    // - Otherwise, the original loop's branch weights are correct for the
    //   unrolled loop, so do not adjust them.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  return CompletelyUnroll ? LoopUnrollResult::FullyUnrolled
                          : LoopUnrollResult::PartiallyUnrolled;
}

/// Given an llvm.loop loop id metadata node, returns the loop hint metadata
/// node with the given name (for example, "llvm.loop.unroll.count"). If no
/// such metadata node exists, then nullptr is returned.
MDNode *llvm::GetUnrollMetadata(MDNode *LoopID, StringRef Name) {
  // First operand should refer to the loop id itself.
  assert(LoopID->getNumOperands() > 0 && "requires at least one operand");
  assert(LoopID->getOperand(0) == LoopID && "invalid loop id");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1680-1744

```cpp
  for (const MDOperand &MDO : llvm::drop_begin(LoopID->operands())) {
    MDNode *MD = dyn_cast<MDNode>(MDO);
    if (!MD)
      continue;

    MDString *S = dyn_cast<MDString>(MD->getOperand(0));
    if (!S)
      continue;

    if (Name == S->getString())
      return MD;
  }
  return nullptr;
}

// Returns the loop hint metadata node with the given name (for example,
// "llvm.loop.unroll.count").  If no such metadata node exists, then nullptr is
// returned.
MDNode *llvm::getUnrollMetadataForLoop(const Loop *L, StringRef Name) {
  if (MDNode *LoopID = L->getLoopID())
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (any_of(cast<Instruction>(Phi.getIncomingValueForBlock(L->getLoopLatch()))
                 ->operands(),
             IsaPred<Constant>))
    return std::nullopt;

  BasicBlock *Latch = L->getLoopLatch();
  if (!Latch ||
      !is_contained(
          cast<Instruction>(Phi.getIncomingValueForBlock(Latch))->operands(),
          &Phi))
    return std::nullopt;

  return RdxDesc;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DemandedBits, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DemandedBits, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `DataLayout, Value, LoadValue, StackNode, ExitInfo, isEpilogProfitable, getMatchingValue, loadCSE` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`DataLayout, Value, LoadValue, StackNode, ExitInfo, isEpilogProfitable, getMatchingValue, loadCSE` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AliasAnalysis, AssumptionCache, DataLayout, DemandedBits, DominatorTree, DomTreeUpdater, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AliasAnalysis, AssumptionCache, DataLayout, DemandedBits, DominatorTree, DomTreeUpdater, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `UnrollRuntimeEpilog, UnrollVerifyDomtree, UnrollVerifyLoopInfo, UnrollAddParallelReductions` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `UnrollRuntimeEpilog, UnrollVerifyDomtree, UnrollVerifyLoopInfo, UnrollAddParallelReductions` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/GenericDomTree.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopedHashTable.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/GenericDomTree.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopedHashTable.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `assert.h`, `cmath`, `numeric`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`assert.h`, `cmath`, `numeric`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DemandedBits`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DemandedBits`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
