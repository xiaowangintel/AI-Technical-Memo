# LoopPeel.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopPeel.cpp` | `llvm/lib/Transforms/Utils/LoopPeel.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Loop Peel within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopPeel 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-92

```cpp
//===- LoopPeel.cpp -------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Loop Peeling Utilities.
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LoopPeel.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/Loads.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/LoopIterator.h"
#include "llvm/Analysis/ScalarEvolution.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    "unroll-force-peel-count", cl::init(0), cl::Hidden,
    cl::desc("Force a peel count regardless of profiling information."));

static cl::opt<bool> DisableAdvancedPeeling(
    "disable-advanced-peeling", cl::init(false), cl::Hidden,
    cl::desc(
        "Disable advance peeling. Issues for convergent targets (D134803)."));

static cl::opt<bool> EnablePeelingForIV(
    "enable-peeling-for-iv", cl::init(false), cl::Hidden,
    cl::desc("Enable peeling to convert Phi nodes into IVs"));

static const char *PeeledCountMetaData = "llvm.loop.peeled.count";

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 93-183

```cpp
extern cl::opt<bool> ProfcheckDisableMetadataFixes;
} // namespace llvm

// Check whether we are capable of peeling this loop.
bool llvm::canPeel(const Loop *L) {
  // Make sure the loop is in simplified form
  if (!L->isLoopSimplifyForm())
    return false;
  if (!DisableAdvancedPeeling)
    return true;

  SmallVector<BasicBlock *, 4> Exits;
  L->getUniqueNonLatchExitBlocks(Exits);
  // The latch must either be the only exiting block or all non-latch exit
  // blocks have either a deopt or unreachable terminator or compose a chain of
  // blocks where the last one is either deopt or unreachable terminated. Both
  // deopt and unreachable terminators are a strong indication they are not
  // taken. Note that this is a profitability check, not a legality check. Also
  // note that LoopPeeling currently can only update the branch weights of latch
  // blocks and branch weights to blocks with deopt or unreachable do not need
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// iteration). For example, consider the following function.
//
//   #define N 100
//   void f(int a[], int b[]) {
//     int im = N - 1;
//     for (int i = 0; i < N; i++) {
//       a[i] = b[i] + b[im];
//       im = i;
//     }
//   }
//
// The IR of the loop will look something like the following.
//
//   %i = phi i32 [ 0, %entry ], [ %i.next, %for.body ]
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include canPeel, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 canPeel，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 184-276

```cpp
//   %im = phi i32 [ 99, %entry ], [ %i, %for.body ]
//   ...
//   %i.next = add nuw nsw i32 %i, 1
//   ...
//
// In this case, %im becomes a loop-induction variable by peeling 1 iteration,
// because %i is a loop-induction one. The peeling count can be determined by
// the same algorithm with loop-invariant case. Such peeling is profitable for
// loop-vectorization.
//
// The PhiAnalyzer class calculates how many times a loop should be
// peeled based on the above analysis of the phi nodes in the loop while
// respecting the maximum specified.
class PhiAnalyzer {
public:
  PhiAnalyzer(const Loop &L, unsigned MaxIterations, bool PeelForIV);

  // Calculate the sufficient minimum number of iterations of the loop to peel
  // such that phi instructions become determined (subject to allowable limits)
  std::optional<unsigned> calculateIterationsToPeel();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (Latch == nullptr)
    return false;

  Value *Cur = Phi->getIncomingValueForBlock(Latch);
  SmallPtrSet<Value *, 4> Visited;
  bool VisitBinOp = false;

  // Starting from the incoming value of the Phi, we follow the use-def chain.
  // We consider Phi to be an IV if we can reach it again by traversing only
  // add, sub, or cast instructions.
  while (true) {
    if (Cur == Phi)
      break;

```
- EN: Core entities appearing here include PhiAnalyzer, PeelCounterType, addOne, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PhiAnalyzer, PeelCounterType, addOne，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 277-369

```cpp
    // Avoid infinite loop.
    if (!Visited.insert(Cur).second)
      return false;

    auto *I = dyn_cast<Instruction>(Cur);
    if (!I || !L.contains(I))
      return false;

    if (auto *Cast = dyn_cast<CastInst>(I)) {
      Cur = Cast->getOperand(0);
    } else if (auto *BinOp = dyn_cast<BinaryOperator>(I)) {
      if (BinOp->getOpcode() != Instruction::Add &&
          BinOp->getOpcode() != Instruction::Sub)
        return false;
      if (!isa<ConstantInt>(BinOp->getOperand(1)))
        return false;

      VisitBinOp = true;
      Cur = BinOp->getOperand(0);
    } else {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                makeZero(PeelCounterType::Invariant));
  if (const PHINode *Phi = dyn_cast<PHINode>(&V)) {
    if (Phi->getParent() != L.getHeader()) {
      // Phi is not in header block so Unknown.
      assert(IterationsToInvarianceOrInduction[&V] == Unknown &&
             "unexpected value saved");
      return Unknown;
    }

    // If Phi is an induction, register it as a starting point.
    if (PeelForIV && isInductionPHI(Phi))
      return (IterationsToInvarianceOrInduction[&V] =
                  makeZero(PeelCounterType::Induction));

```
- EN: Core entities appearing here include calculate, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 calculate，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 370-457

```cpp
    // We need to analyze the input from the back edge and add 1.
    Value *Input = Phi->getIncomingValueForBlock(L.getLoopLatch());
    PeelCounter Iterations = calculate(*Input);
    assert(IterationsToInvarianceOrInduction[Input] == Iterations &&
           "unexpected value saved");
    return (IterationsToInvarianceOrInduction[Phi] = addOne(Iterations));
  }
  if (const Instruction *I = dyn_cast<Instruction>(&V)) {
    if (isa<CmpInst>(I) || I->isBinaryOp()) {
      // Binary instructions get the max of the operands.
      PeelCounter LHS = calculate(*I->getOperand(0));
      if (LHS == Unknown)
        return Unknown;
      PeelCounter RHS = calculate(*I->getOperand(1));
      if (RHS == Unknown)
        return Unknown;
      return (IterationsToInvarianceOrInduction[I] =
                  mergeTwoCounters(*I, *LHS, *RHS));
    }
    if (I->isCast())
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // collect the set of instructions controlled by such loads. Only peel if an
  // exit condition uses (transitively) such a load.
  BasicBlock *Header = L.getHeader();
  BasicBlock *Latch = L.getLoopLatch();
  SmallPtrSet<Value *, 8> LoadUsers;
  const DataLayout &DL = L.getHeader()->getDataLayout();
  for (BasicBlock *BB : L.blocks()) {
    for (Instruction &I : *BB) {
      // Calls that only access inaccessible memory can never alias with loads.
      if (I.mayWriteToMemory() &&
          !(isa<CallBase>(I) &&
            cast<CallBase>(I).onlyAccessesInaccessibleMemory()))
        return 0;

```
- EN: Core entities appearing here include calculateIterationsToPeel, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 calculateIterationsToPeel，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 458-555

```cpp
      if (LoadUsers.contains(&I))
        LoadUsers.insert_range(I.users());
      // Do not look for reads in the header; they can already be hoisted
      // without peeling.
      if (BB == Header)
        continue;
      if (auto *LI = dyn_cast<LoadInst>(&I)) {
        Value *Ptr = LI->getPointerOperand();
        if (DT.dominates(BB, Latch) && L.isLoopInvariant(Ptr) &&
            !isDereferenceablePointer(Ptr, LI->getType(), DL, LI, AC, &DT))
          LoadUsers.insert_range(I.users());
      }
    }
  }
  SmallVector<BasicBlock *> ExitingBlocks;
  L.getExitingBlocks(ExitingBlocks);
  if (any_of(ExitingBlocks, [&LoadUsers](BasicBlock *Exiting) {
        return LoadUsers.contains(Exiting->getTerminator());
      }))
    return 1;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
//
//  for (i = 0; i < n; i++)
//    if (i < 2)
//      ..
//    else
//      ..
//   }
static std::pair<unsigned, unsigned>
countToEliminateCompares(Loop &L, unsigned MaxPeelCount, ScalarEvolution &SE,
                         const TargetTransformInfo &TTI) {
  assert(L.isLoopSimplifyForm() && "Loop needs to be in loop simplify form");
  unsigned DesiredPeelCount = 0;
  unsigned DesiredPeelCountLast = 0;

```
- EN: Core entities appearing here include canPeelLastIteration, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canPeelLastIteration，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 556-657

```cpp
  // Do not peel the entire loop.
  const SCEV *BE = SE.getConstantMaxBackedgeTakenCount(&L);
  if (const SCEVConstant *SC = dyn_cast<SCEVConstant>(BE))
    MaxPeelCount =
        std::min((unsigned)SC->getAPInt().getLimitedValue() - 1, MaxPeelCount);

  // Increase PeelCount while (IterVal Pred BoundSCEV) condition is satisfied;
  // return true if inversed condition become known before reaching the
  // MaxPeelCount limit.
  auto PeelWhilePredicateIsKnown =
      [&](unsigned &PeelCount, const SCEV *&IterVal, const SCEV *BoundSCEV,
          const SCEV *Step, ICmpInst::Predicate Pred) {
        while (PeelCount < MaxPeelCount &&
               SE.isKnownPredicate(Pred, IterVal, BoundSCEV)) {
          IterVal = SE.getAddExpr(IterVal, Step);
          ++PeelCount;
        }
        return SE.isKnownPredicate(ICmpInst::getInversePredicate(Pred), IterVal,
                                   BoundSCEV);
      };
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // However, for equality comparisons, that isn't always sufficient to
    // eliminate the comparsion in loop body, we may need to peel one more
    // iteration. See if that makes !Pred become unknown again.
    const SCEV *NextIterVal = SE.getAddExpr(IterVal, Step);
    if (ICmpInst::isEquality(Pred) &&
        !SE.isKnownPredicate(ICmpInst::getInversePredicate(Pred), NextIterVal,
                             RightSCEV) &&
        !SE.isKnownPredicate(Pred, IterVal, RightSCEV) &&
        SE.isKnownPredicate(Pred, NextIterVal, RightSCEV)) {
      if (NewPeelCount >= MaxPeelCount)
        return; // Need to peel one more iteration, but can't. Give up.
      ++NewPeelCount; // Great!
    }

```
- EN: Core entities appearing here include match, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 658-750

```cpp
    DesiredPeelCount = std::max(DesiredPeelCount, NewPeelCount);
    DesiredPeelCountLast = std::max(DesiredPeelCountLast, NewPeelCount);
  };

  auto ComputePeelCountMinMax = [&](MinMaxIntrinsic *MinMax) {
    if (!MinMax->getType()->isIntegerTy())
      return;
    Value *LHS = MinMax->getLHS(), *RHS = MinMax->getRHS();
    const SCEV *BoundSCEV, *IterSCEV;
    if (L.isLoopInvariant(LHS)) {
      BoundSCEV = SE.getSCEV(LHS);
      IterSCEV = SE.getSCEV(RHS);
    } else if (L.isLoopInvariant(RHS)) {
      BoundSCEV = SE.getSCEV(RHS);
      IterSCEV = SE.getSCEV(LHS);
    } else
      return;
    const auto *AddRec = dyn_cast<SCEVAddRecExpr>(IterSCEV);
    // For simplicity, we support only affine recurrences.
    if (!AddRec || !AddRec->isAffine() || AddRec->getLoop() != &L)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (!LatchBR || !L->isLoopExiting(Latch))
    return true;

  assert((LatchBR->getSuccessor(0) == L->getHeader() ||
          LatchBR->getSuccessor(1) == L->getHeader()) &&
         "At least one edge out of the latch must go to the header");

  SmallVector<BasicBlock *, 4> ExitBlocks;
  L->getUniqueNonLatchExitBlocks(ExitBlocks);
  return any_of(ExitBlocks, [](const BasicBlock *EB) {
      return !EB->getTerminatingDeoptimizeCall();
    });
}

```
- EN: Core entities appearing here include violatesLegacyMultiExitLoopCheck, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 violatesLegacyMultiExitLoopCheck，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 751-856

```cpp

// Return the number of iterations we want to peel off.
void llvm::computePeelCount(Loop *L, unsigned LoopSize,
                            TargetTransformInfo::PeelingPreferences &PP,
                            unsigned TripCount, DominatorTree &DT,
                            ScalarEvolution &SE, const TargetTransformInfo &TTI,
                            AssumptionCache *AC, unsigned Threshold) {
  assert(LoopSize > 0 && "Zero loop size is not allowed!");
  // Save the PP.PeelCount value set by the target in
  // TTI.getPeelingPreferences or by the flag -unroll-peel-count.
  unsigned TargetPeelCount = PP.PeelCount;
  PP.PeelCount = 0;
  PP.PeelLast = false;
  if (!canPeel(L))
    return;

  // Only try to peel innermost loops by default.
  // The constraint can be relaxed by the target in TTI.getPeelingPreferences
  // or by the flag -unroll-allow-loop-nests-peeling.
  if (!PP.AllowLoopNestsPeeling && !L->isInnermost())
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        std::min(CountToEliminateCmpsLast, MaxPeelCount);
    // Consider max peel count limitation.
    assert(DesiredPeelCountLast > 0 && "Wrong loop size estimation?");
    if (DesiredPeelCountLast + AlreadyPeeled <= UnrollPeelMaxCount) {
      LLVM_DEBUG(dbgs() << "Peel " << DesiredPeelCount
                        << " iteration(s) to turn"
                        << " some Phis into invariants.\n");
      PP.PeelCount = DesiredPeelCountLast;
      PP.PeelProfiledIterations = false;
      PP.PeelLast = true;
      return;
    }
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopNest, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopNest, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 857-950

```cpp
  // Bail if we know the statically calculated trip count.
  // In this case we rather prefer partial unrolling.
  if (TripCount)
    return;

  // Do not apply profile base peeling if it is disabled.
  if (!PP.PeelProfiledIterations)
    return;
  // If we don't know the trip count, but have reason to believe the average
  // trip count is low, peeling should be beneficial, since we will usually
  // hit the peeled section.
  // We only do this in the presence of profile information, since otherwise
  // our estimates of the trip count are not reliable enough.
  if (L->getHeader()->getParent()->hasProfileData()) {
    if (violatesLegacyMultiExitLoopCheck(L))
      return;
    std::optional<unsigned> EstimatedTripCount = getLoopEstimatedTripCount(L);
    if (!EstimatedTripCount)
      return;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    VMap[*BB] = NewBB;

    // If dominator tree is available, insert nodes to represent cloned blocks.
    if (DT) {
      if (Header == *BB)
        DT->addNewBlock(NewBB, InsertTop);
      else {
        DomTreeNode *IDom = DT->getNode(*BB)->getIDom();
        // VMap must contain entry for IDom, as the iteration order is RPO.
        DT->addNewBlock(NewBB, cast<BasicBlock>(VMap[IDom->getBlock()]));
      }
    }
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 951-1053

```cpp
  {
    // Identify what other metadata depends on the cloned version. After
    // cloning, replace the metadata with the corrected version for both
    // memory instructions and noalias intrinsics.
    std::string Ext = (Twine("Peel") + Twine(IterNumber)).str();
    cloneAndAdaptNoAliasScopes(LoopLocalNoAliasDeclScopes, NewBlocks,
                               Header->getContext(), Ext);
  }

  // Recursively create the new Loop objects for nested loops, if any,
  // to preserve LoopInfo.
  for (Loop *ChildLoop : *L) {
    cloneLoop(ChildLoop, ParentLoop, VMap, LI, nullptr);
  }

  // Hook-up the control flow for the newly inserted blocks.
  // The new header is hooked up directly to the "top", which is either
  // the original loop preheader (for the first iteration) or the previous
  // iteration's exiting block (for every other iteration)
  InsertTop->getTerminator()->setSuccessor(0, cast<BasicBlock>(VMap[Header]));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Fix up the outgoing values - we need to add a value for the iteration
  // we've just created. Note that this must happen *after* the incoming
  // values are adjusted, since the value going out of the latch may also be
  // a value coming into the header.
  for (auto Edge : ExitEdges)
    for (PHINode &PHI : Edge.second->phis()) {
      Value *LatchVal = PHI.getIncomingValueForBlock(Edge.first);
      Instruction *LatchInst = dyn_cast<Instruction>(LatchVal);
      if (LatchInst && L->contains(LatchInst))
        LatchVal = VMap[LatchVal];
      PHI.addIncoming(LatchVal, cast<BasicBlock>(VMap[Edge.first]));
      SE.forgetLcssaPhiWithNewPredecessor(L, &PHI);
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1054-1145

```cpp
  // LastValueMap is updated with the values for the current loop
  // which are used the next time this function is called.
  for (auto KV : VMap)
    LVMap[KV.first] = KV.second;
}

TargetTransformInfo::PeelingPreferences
llvm::gatherPeelingPreferences(Loop *L, ScalarEvolution &SE,
                               const TargetTransformInfo &TTI,
                               std::optional<bool> UserAllowPeeling,
                               std::optional<bool> UserAllowProfileBasedPeeling,
                               bool UnrollingSpecficValues) {
  TargetTransformInfo::PeelingPreferences PP;

  // Set the default values.
  PP.PeelCount = 0;
  PP.AllowPeeling = true;
  PP.AllowLoopNestsPeeling = false;
  PP.PeelLast = false;
  PP.PeelProfiledIterations = true;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (auto *ChildDomNode : BBDomNode->children()) {
      auto *ChildBB = ChildDomNode->getBlock();
      if (!L->contains(ChildBB))
        ChildrenToUpdate.push_back(ChildBB);
    }
    // The new idom of the block will be the nearest common dominator
    // of all copies of the previous idom. This is equivalent to the
    // nearest common dominator of the previous idom and the first latch,
    // which dominates all copies of the previous idom.
    BasicBlock *NewIDom = DT.findNearestCommonDominator(BB, Latch);
    for (auto *ChildBB : ChildrenToUpdate)
      NonLoopBlocksIDom[ChildBB] = NewIDom;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, LoopInfo, LoopNest. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, LoopInfo, LoopNest 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1146-1249

```cpp
  Function *F = Header->getParent();

  // Set up all the necessary basic blocks.
  BasicBlock *InsertTop;
  BasicBlock *InsertBot;
  BasicBlock *NewPreHeader = nullptr;
  DenseMap<Instruction *, Value *> ExitValues;
  if (PeelLast) {
    // It is convenient to split the single exit block from the latch the
    // into 3 parts - two blocks to anchor the peeled copy of the loop body,
    // and a new final  exit block.

    // Peeling the last iteration transforms.
    //
    // PreHeader:
    // ...
    // Header:
    //   LoopBody
    //   If (cond) goto Header
    // Exit:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        // The probability that the new guard skips the loop to execute just one
        // iteration is the original loop's probability of exiting at the latch
        // after any iteration. That should maintain the original loop body
        // frequency. Upon arriving at the loop, due to the guard, the
        // probability of reaching iteration i of the new loop is the
        // probability of reaching iteration i+1 of the original loop. The
        // probability of reaching the peeled iteration is 1, which is the
        // probability of reaching iteration 0 of the original loop.
        if (L->getExitBlock() == OrigLatchBr->getSuccessor(0))
          std::swap(Weights[0], Weights[1]);
        setBranchWeights(*BI, Weights, /*IsExpected=*/false);
      }
      PreHeaderBR->eraseFromParent();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1250-1351

```cpp
      // PreHeader now dominates InsertTop.
      DT.changeImmediateDominator(InsertTop, PreHeader);
    }
  } else {
    // It is convenient to split the preheader into 3 parts - two blocks to
    // anchor the peeled copy of the loop body, and a new preheader for the
    // "real" loop.

    // Peeling the first iteration transforms.
    //
    // PreHeader:
    // ...
    // Header:
    //   LoopBody
    //   If (cond) goto Header
    // Exit:
    //
    // into
    //
    // InsertTop:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        auto *Cmp =
            cast<ICmpInst>(L->getLoopLatch()->getTerminator()->getOperand(0));
        IRBuilder B(Cmp);
        Cmp->setOperand(
            1, B.CreateSub(Cmp->getOperand(1),
                           ConstantInt::get(Cmp->getOperand(1)->getType(), 1)));
      } else {
        // Update IDoms of the blocks reachable through exits.
        for (auto BBIDom : NonLoopBlocksIDom)
          DT.changeImmediateDominator(BBIDom.first,
                                      cast<BasicBlock>(LVMap[BBIDom.second]));
      }
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1352-1442

```cpp
#ifdef EXPENSIVE_CHECKS
    assert(DT.verify(DominatorTree::VerificationLevel::Fast));
#endif

    // Remove Loop metadata from the latch branch instruction
    // because it is not the Loop's latch branch anymore.
    auto *LatchTermCopy = cast<Instruction>(VMap[LatchTerm]);
    LatchTermCopy->setMetadata(LLVMContext::MD_loop, nullptr);

    InsertTop = InsertBot;
    InsertBot = SplitBlock(InsertBot, InsertBot->getTerminator(), &DT, LI);
    InsertBot->setName(Header->getName() + ".peel.next");

    F->splice(InsertTop->getIterator(), F, NewBlocks[0]->getIterator(),
              F->end());
  }

  if (PeelLast) {
    // Now adjust users of the original exit values by replacing them with the
    // exit value from the peeled iteration and remove them.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    L = ParentLoop;

  // We modified the loop, update SE.
  SE->forgetTopmostLoop(L);
  SE->forgetBlockAndLoopDispositions();

#ifdef EXPENSIVE_CHECKS
  // Finally DomtTree must be correct.
  assert(DT.verify(DominatorTree::VerificationLevel::Fast));
#endif

  // FIXME: Incrementally update loop-simplify
  simplifyLoop(L, &DT, LI, SE, AC, nullptr, PreserveLCSSA);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1443-1445

```cpp
  NumPeeled++;
  NumPeeledEnd += PeelLast;
}
```
- EN: This region continues the LoopPeel implementation with local helper logic centered on NumPeeled, NumPeeledEnd, PeelLast.
- CN: 这一段延续了 LoopPeel 的主体实现，围绕 NumPeeled, NumPeeledEnd, PeelLast 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `PhiAnalyzer, PeelCounterType, canPeel, addOne, makeZero, L, isInductionPHI, calculate` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PhiAnalyzer, PeelCounterType, canPeel, addOne, makeZero, L, isInductionPHI, calculate` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, LoopInfo, LoopNest, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, LoopInfo, LoopNest, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `UnrollPeelCount, UnrollAllowPeeling, UnrollAllowLoopNestsPeeling, UnrollPeelMaxCount, UnrollForcePeelCount` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `UnrollPeelCount, UnrollAllowPeeling, UnrollAllowLoopNestsPeeling, UnrollPeelMaxCount, UnrollForcePeelCount` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/Loads.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/Loads.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LoopPeel.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LoopPeel.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CheckedArithmetic.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CheckedArithmetic.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `LoopInfo`, `LoopNest`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `LoopInfo`, `LoopNest`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
