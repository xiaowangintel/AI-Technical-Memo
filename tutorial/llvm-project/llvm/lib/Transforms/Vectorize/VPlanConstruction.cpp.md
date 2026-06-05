# VPlanConstruction.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanConstruction.cpp` | `llvm/lib/Transforms/Vectorize/VPlanConstruction.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements transforms for initial VPlan construction within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanConstruction 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-150

```cpp
//===-- VPlanConstruction.cpp - Transforms for initial VPlan construction -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements transforms for initial VPlan construction.
///
//===----------------------------------------------------------------------===//

#include "LoopVectorizationPlanner.h"
#include "VPlan.h"
#include "VPlanAnalysis.h"
#include "VPlanCFG.h"
#include "VPlanDominatorTree.h"
#include "VPlanHelpers.h"
#include "VPlanPatternMatch.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

#ifndef NDEBUG
// Return true if \p Val is considered an external definition. An external
// definition is either:
// 1. A Value that is not an Instruction. This will be refined in the future.
// 2. An Instruction that is outside of the IR region represented in VPlan,
// i.e., is not part of the loop nest.
bool PlainCFGBuilder::isExternalDef(Value *Val) {
  // All the Values that are not Instructions are considered external
  // definitions for now.
  Instruction *Inst = dyn_cast<Instruction>(Val);
  if (!Inst)
    return true;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 151-294

```cpp
  // Check whether Instruction definition is in loop body.
  return !TheLoop->contains(Inst);
}
#endif

// Create a new VPValue or retrieve an existing one for the Instruction's
// operand \p IRVal. This function must only be used to create/retrieve VPValues
// for *Instruction's operands* and not to create regular VPInstruction's. For
// the latter, please, look at 'createVPInstructionsForVPBB'.
VPValue *PlainCFGBuilder::getOrCreateVPOperand(Value *IRVal) {
  auto VPValIt = IRDef2VPValue.find(IRVal);
  if (VPValIt != IRDef2VPValue.end())
    // Operand has an associated VPInstruction or VPValue that was previously
    // created.
    return VPValIt->second;

  // Operand doesn't have a previously created VPInstruction/VPValue. This
  // means that operand is:
  //   A) a definition external to VPlan,
  //   B) any other Value without specific representation in VPlan.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
    }

    IRDef2VPValue[Inst] = NewR;
  }
}

// Main interface to build the plain CFG.
std::unique_ptr<VPlan> PlainCFGBuilder::buildPlainCFG() {
  VPIRBasicBlock *Entry = cast<VPIRBasicBlock>(Plan->getEntry());
  BB2VPBB[Entry->getIRBasicBlock()] = Entry;
  for (VPIRBasicBlock *ExitVPBB : Plan->getExitBlocks())
    BB2VPBB[ExitVPBB->getIRBasicBlock()] = ExitVPBB;

```
- EN: Core entities appearing here include buildPlainCFG, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 buildPlainCFG，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 295-449

```cpp
  // 1. Scan the body of the loop in a topological order to visit each basic
  // block after having visited its predecessor basic blocks. Create a VPBB for
  // each BB and link it to its successor and predecessor VPBBs. Note that
  // predecessors must be set in the same order as they are in the incomming IR.
  // Otherwise, there might be problems with existing phi nodes and algorithm
  // based on predecessors traversal.

  // Loop PH needs to be explicitly visited since it's not taken into account by
  // LoopBlocksDFS.
  BasicBlock *ThePreheaderBB = TheLoop->getLoopPreheader();
  assert((ThePreheaderBB->getTerminator()->getNumSuccessors() == 1) &&
         "Unexpected loop preheader");
  for (auto &I : *ThePreheaderBB) {
    if (I.getType()->isVoidTy())
      continue;
    IRDef2VPValue[&I] = Plan->getOrAddLiveIn(&I);
  }

  LoopBlocksRPO RPO(TheLoop);
  RPO.perform(LI);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // canonical IV of the outermost (to be vectorized) loop. Only the outermost
  // header will have a canonical IV. Other, nested loops are assigned a
  // canonical IV of null type and debug location.
  Type *CanIVTy = nullptr;
  DebugLoc DL = DebugLoc::getUnknown();
  auto *OutermostHeaderVPBB = cast<VPBasicBlock>(
      Plan.getEntry()->getSuccessors()[1]->getSingleSuccessor());
  VPPhi *OutermostVPPhi = nullptr;
  if (HeaderVPB == OutermostHeaderVPBB) {
    OutermostVPPhi = cast<VPPhi>(&OutermostHeaderVPBB->front());
    CanIVTy = OutermostVPPhi->getOperand(0)->getLiveInIRValue()->getType();
    DL = OutermostVPPhi->getDebugLoc();
  }

```
- EN: Core entities appearing here include createLoopRegion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createLoopRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 450-606

```cpp
  auto *PreheaderVPBB = HeaderVPB->getPredecessors()[0];
  auto *LatchVPBB = HeaderVPB->getPredecessors()[1];

  VPBlockUtils::disconnectBlocks(PreheaderVPBB, HeaderVPB);
  VPBlockUtils::disconnectBlocks(LatchVPBB, HeaderVPB);

  // Create an empty region first and insert it between PreheaderVPBB and
  // the exit blocks, taking care to preserve the original predecessor &
  // successor order of blocks. Set region entry and exiting after both
  // HeaderVPB and LatchVPBB have been disconnected from their
  // predecessors/successors.
  auto *R = Plan.createLoopRegion(CanIVTy, DL);

  // Transfer latch's successors to the region.
  VPBlockUtils::transferSuccessors(LatchVPBB, R);

  VPBlockUtils::connectBlocks(PreheaderVPBB, R);
  R->setEntry(HeaderVPB);
  R->setExiting(LatchVPBB);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    VPValue *BackedgeVal = VectorPhiR->getOperand(1);
    VPValue *ResumeFromVectorLoop =
        MiddleBuilder.createNaryOp(VPInstruction::ExtractLastPart, BackedgeVal);
    ResumeFromVectorLoop = MiddleBuilder.createNaryOp(
        VPInstruction::ExtractLastLane, ResumeFromVectorLoop);
    // Create scalar resume phi, with the first operand being the incoming value
    // from the middle block and the second operand coming from the entry block.
    auto *ResumePhiR = ScalarPHBuilder.createScalarPhi(
        {ResumeFromVectorLoop, VectorPhiR->getOperand(0)},
        VectorPhiR->getDebugLoc());
    cast<VPIRPhi>(&ScalarPhiR)->addOperand(ResumePhiR);
  }
}

```
- EN: Core entities appearing here include addCanonicalIVRecipes, createExtractsForLiveOuts, zip_equal, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addCanonicalIVRecipes, createExtractsForLiveOuts, zip_equal，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 607-747

```cpp
/// Check \p Plan's live-in and replace them with constants, if they can be
/// simplified via SCEV.
static void simplifyLiveInsWithSCEV(VPlan &Plan,
                                    PredicatedScalarEvolution &PSE) {
  auto GetSimplifiedLiveInViaSCEV = [&](VPValue *VPV) -> VPValue * {
    const SCEV *Expr = vputils::getSCEVExprForVPValue(VPV, PSE);
    if (auto *C = dyn_cast<SCEVConstant>(Expr))
      return Plan.getOrAddLiveIn(C->getValue());
    return nullptr;
  };

  for (VPValue *LiveIn : to_vector(Plan.getLiveIns())) {
    if (VPValue *SimplifiedLiveIn = GetSimplifiedLiveInViaSCEV(LiveIn))
      LiveIn->replaceAllUsesWith(SimplifiedLiveIn);
  }
}

/// To make RUN_VPLAN_PASS print initial VPlan.
static void printAfterInitialConstruction(VPlan &) {}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (vputils::cannotHoistOrSinkRecipe(*SinkCandidate, /*Sinking=*/true))
      return false;

    WorkList.push_back(SinkCandidate);
    return true;
  };

  // Recursively sink users of FOR after Previous.
  WorkList.push_back(FOR);
  for (unsigned I = 0; I != WorkList.size(); ++I) {
    VPRecipeBase *Current = WorkList[I];
    assert(Current->getNumDefinedValues() == 1 &&
           "only recipes with a single defined value expected");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 748-889

```cpp
    for (VPUser *User : Current->getVPSingleValue()->users()) {
      if (!TryToPushSinkCandidate(cast<VPRecipeBase>(User)))
        return false;
    }
  }

  // Keep recipes to sink ordered by dominance so earlier instructions are
  // processed first.
  sort(WorkList, [&VPDT](const VPRecipeBase *A, const VPRecipeBase *B) {
    return VPDT.properlyDominates(A, B);
  });

  for (VPRecipeBase *SinkCandidate : WorkList) {
    if (SinkCandidate == FOR)
      continue;

    SinkCandidate->moveAfter(Previous);
    Previous = SinkCandidate;
  }
  return true;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
               dyn_cast_or_null<VPFirstOrderRecurrencePHIRecipe>(Previous)) {
      assert(PrevPhi->getParent() == FOR->getParent() &&
             "PrevPhi must be in same block as FOR");
      assert(SeenPhis.insert(PrevPhi).second &&
             "PrevPhi must not be visited multiple times");
      Previous = PrevPhi->getBackedgeValue()->getDefiningRecipe();
    }

    assert(Previous && "Previous must be a recipe");
    // Sink FOR users after Previous or hoist Previous before FOR users.
    if (!sinkRecurrenceUsersAfterPrevious(FOR, Previous, VPDT) &&
        !hoistPreviousBeforeFORUsers(FOR, Previous, VPDT))
      return false;

```
- EN: Core entities appearing here include sort, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 sort，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 890-1032

```cpp
    // Create FirstOrderRecurrenceSplice and replace FOR uses.
    VPBasicBlock *InsertBlock = Previous->getParent();
    auto InsertPt = isa<VPHeaderPHIRecipe>(Previous)
                        ? InsertBlock->getFirstNonPhi()
                        : std::next(Previous->getIterator());
    VPBuilder LoopBuilder(InsertBlock, InsertPt);
    auto *RecurSplice =
        LoopBuilder.createNaryOp(VPInstruction::FirstOrderRecurrenceSplice,
                                 {FOR, FOR->getBackedgeValue()});
    FOR->replaceUsesWithIf(RecurSplice, [RecurSplice](VPUser &U, unsigned) {
      return &U != RecurSplice;
    });
  }

  return true;
}

bool VPlanTransforms::createHeaderPhiRecipes(
    VPlan &Plan, PredicatedScalarEvolution &PSE, Loop &OrigLoop,
    const MapVector<PHINode *, InductionDescriptor> &Inductions,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          assert((UserRecipe->getParent() == Plan.getMiddleBlock() ||
                  UserRecipe->getParent() == Plan.getScalarPreheader()) &&
                 "U must be either in the loop region, the middle block or the "
                 "scalar preheader.");
          continue;
        }

        // Stores using instructions will be sunk later.
        if (match(UserRecipe, m_VPInstruction<Instruction::Store>()))
          continue;
        Worklist.insert(UserRecipe);
      }
    }

```
- EN: Core entities appearing here include zip_equal, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 zip_equal，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1033-1179

```cpp
    // Visit operation "Links" along the reduction chain top-down starting from
    // the phi until LoopExitValue. We keep track of the previous item
    // (PreviousLink) to tell which of the two operands of a Link will remain
    // scalar and which will be reduced. For minmax by select(cmp), Link will be
    // the select instructions. Blend recipes of in-loop reduction phi's will
    // get folded to their non-phi operand, as the reduction recipe handles the
    // condition directly.
    VPSingleDefRecipe *PreviousLink = PhiR; // Aka Worklist[0].
    for (VPSingleDefRecipe *CurrentLink : drop_begin(Worklist)) {
      if (auto *Blend = dyn_cast<VPBlendRecipe>(CurrentLink)) {
        assert(Blend->getNumIncomingValues() == 2 &&
               "Blend must have 2 incoming values");
        unsigned PhiRIdx = Blend->getIncomingValue(0) == PhiR ? 0 : 1;
        assert(Blend->getIncomingValue(PhiRIdx) == PhiR &&
               "PhiR must be an operand of the blend");
        Blend->replaceAllUsesWith(Blend->getIncomingValue(1 - PhiRIdx));
        continue;
      }

      if (IsFPRecurrence) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (!VPI || VPI->getOpcode() != Instruction::Load) {
        assert(!R.mayReadFromMemory() && "unexpected recipe reading memory");
        continue;
      }

      // Get the pointer SCEV for dereferenceability checking.
      VPValue *Ptr = VPI->getOperand(0);
      const SCEV *PtrSCEV = vputils::getSCEVExprForVPValue(Ptr, PSE, TheLoop);
      if (isa<SCEVCouldNotCompute>(PtrSCEV)) {
        LLVM_DEBUG(dbgs() << "LV: Not vectorizing: Found non-dereferenceable "
                             "load with SCEVCouldNotCompute pointer\n");
        return false;
      }

```
- EN: Core entities appearing here include match, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1180-1328

```cpp
      // Check dereferenceability using the SCEV-based version.
      Type *LoadTy = VPI->getResultType();
      const SCEV *SizeSCEV =
          SE.getStoreSizeOfExpr(DL.getIndexType(PtrSCEV->getType()), LoadTy);
      auto *Load = cast<LoadInst>(VPI->getUnderlyingValue());
      SmallVector<const SCEVPredicate *> Preds;
      if (isDereferenceableAndAlignedInLoop(PtrSCEV, Load->getAlign(), SizeSCEV,
                                            TheLoop, SE, DT, AC, &Preds))
        continue;

      LLVM_DEBUG(
          dbgs() << "LV: Not vectorizing: Auto-vectorization of loops with "
                    "potentially faulting load is not supported.\n");
      return false;
    }
  }
  return true;
}

bool VPlanTransforms::handleEarlyExits(VPlan &Plan, UncountableExitStyle Style,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  VPBasicBlock *OrigLatch = LoopRegion->getExitingBasicBlock();
  VPValue *IVInc;
  [[maybe_unused]] bool TermBranchOnCount =
      match(OrigLatch->getTerminator(),
            m_BranchOnCount(m_VPValue(IVInc),
                            m_Specific(&Plan.getVectorTripCount())));
  assert(TermBranchOnCount &&
         match(IVInc, m_Add(m_Specific(LoopRegion->getCanonicalIV()),
                            m_Specific(&Plan.getVFxUF()))) &&
         std::next(IVInc->getDefiningRecipe()->getIterator()) ==
             OrigLatch->getTerminator()->getIterator() &&
         "Unexpected canonical iv increment");

```
- EN: Core entities appearing here include addMiddleCheck, createLoopRegions, foldTailByMasking, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addMiddleCheck, createLoopRegions, foldTailByMasking，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1329-1477

```cpp
  // Split the latch at the IV update, and branch to it from the header mask.
  VPBasicBlock *Latch =
      OrigLatch->splitAt(IVInc->getDefiningRecipe()->getIterator());
  Latch->setName("vector.latch");
  VPBlockUtils::connectBlocks(Header, Latch);

  // Collect any values defined in the loop that need a phi. Currently this
  // includes header phi backedges and live-outs extracted in the middle block.
  // TODO: Handle early exits via Plan.getExitBlocks()
  MapVector<VPValue *, SmallVector<VPUser *>> NeedsPhi;
  for (VPRecipeBase &R : Header->phis())
    if (!isa<VPWidenInductionRecipe>(R))
      NeedsPhi[cast<VPHeaderPHIRecipe>(R).getBackedgeValue()].push_back(&R);

  VPValue *V;
  for (VPRecipeBase &R : *Plan.getMiddleBlock())
    if (match(&R, m_ExtractLastPart(m_VPValue(V))))
      NeedsPhi[V].push_back(&R);

  // Insert phis for values coming past the end of the tail.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto GetMinTripCount = [&]() -> const SCEV * {
    // Compute max(MinProfitableTripCount, UF * VF) and return it.
    const SCEV *VFxUF =
        SE.getElementCount(TripCountTy, (VF * UF), SCEV::FlagNUW);
    if (UF * VF.getKnownMinValue() >=
        MinProfitableTripCount.getKnownMinValue()) {
      // TODO: SCEV should be able to simplify test.
      return VFxUF;
    }
    const SCEV *MinProfitableTripCountSCEV =
        SE.getElementCount(TripCountTy, MinProfitableTripCount, SCEV::FlagNUW);
    return SE.getUMaxExpr(MinProfitableTripCountSCEV, VFxUF);
  };

```
- EN: Core entities appearing here include getExpandSCEVInsertPt, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getExpandSCEVInsertPt，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1478-1618

```cpp
  VPBasicBlock *EntryVPBB = Plan.getEntry();
  // Place compare and branch in CheckBlock if given, ExpandSCEVs in Entry.
  VPBasicBlock *CheckVPBB = CheckBlock ? CheckBlock : EntryVPBB;
  VPBuilder Builder(CheckVPBB);
  VPValue *TripCountCheck = Plan.getFalse();
  const SCEV *Step = GetMinTripCount();
  // TripCountCheck = false, folding tail implies positive vector trip
  // count.
  if (!TailFolded) {
    // TODO: Emit unconditional branch to vector preheader instead of
    // conditional branch with known condition.
    TripCount = SE.applyLoopGuards(TripCount, OrigLoop);
    // Check if the trip count is < the step.
    if (SE.isKnownPredicate(CmpPred, TripCount, Step)) {
      // TODO: Ensure step is at most the trip count when determining max VF and
      // UF, w/o tail folding.
      TripCountCheck = Plan.getTrue();
    } else if (!SE.isKnownPredicate(CmpInst::getInversePredicate(CmpPred),
                                    TripCount, Step)) {
      // Generate the minimum iteration check only if we cannot prove the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (isa<VPWidenIntOrFpInductionRecipe>(&R))
      continue;
    auto *Cur = dyn_cast<VPReductionPHIRecipe>(&R);
    if (!Cur) {
      // TODO: Also support fixed-order recurrence phis.
      HasUnsupportedPhi = true;
      continue;
    }
    if (!RecurrenceDescriptor::isFPMinMaxNumRecurrenceKind(
            Cur->getRecurrenceKind())) {
      HasUnsupportedPhi = true;
      continue;
    }

```
- EN: Core entities appearing here include findRecipe, handleMaxMinNumReductions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 findRecipe, handleMaxMinNumReductions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1619-1758

```cpp
    VPValue *MinOrMaxOp = GetMinOrMaxCompareValue(Cur);
    if (!MinOrMaxOp)
      return false;

    MinOrMaxNumReductionsToHandle.emplace_back(Cur, MinOrMaxOp);
  }

  if (MinOrMaxNumReductionsToHandle.empty())
    return true;

  // We won't be able to resume execution in the scalar tail, if there are
  // unsupported header phis or there is no scalar tail at all, due to
  // tail-folding.
  if (HasUnsupportedPhi || !Plan.hasScalarTail())
    return false;

  /// Check if the vector loop of \p Plan can early exit and restart
  /// execution of last vector iteration in the scalar loop. This requires all
  /// recipes up to early exit point be side-effect free as they are
  /// re-executed. Currently we check that the loop is free of any recipe that
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //   data.phi = phi ir<default.val>, vp<new.data>
  //
  //   ...'data' and 'compare' created by existing nodes...
  //
  //   ...new recipes introduced to determine whether to update the reduction
  //      values or keep the current one.
  //   any.active = i1 any-of ir<compare>
  //   new.mask = select vp<any.active>, ir<compare>, vp<mask.phi>
  //   new.data = select vp<any.active>, ir<data>, ir<data.phi>
  //
  // middle.block:
  //   ...extract-last-active replaces compute-reduction-result.
  //   result = extract-last-active vp<new.data>, vp<new.mask>, ir<default.val>

```
- EN: Core entities appearing here include handleFindLastReductions, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 handleFindLastReductions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1759-1901

```cpp
  SmallVector<VPReductionPHIRecipe *, 4> Phis;
  for (VPRecipeBase &Phi :
       Plan.getVectorLoopRegion()->getEntryBasicBlock()->phis()) {
    auto *PhiR = dyn_cast<VPReductionPHIRecipe>(&Phi);
    if (PhiR && RecurrenceDescriptor::isFindLastRecurrenceKind(
                    PhiR->getRecurrenceKind()))
      Phis.push_back(PhiR);
  }

  if (Phis.empty())
    return true;

  VPValue *HeaderMask = vputils::findHeaderMask(Plan);
  for (VPReductionPHIRecipe *PhiR : Phis) {
    // Find the condition for the select/blend.
    VPValue *BackedgeSelect = PhiR->getBackedgeValue();
    VPValue *CondSelect = BackedgeSelect;

    // If there's a header mask, the backedge select will not be the find-last
    // select.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // TODO: Support non (i.e., narrower than) canonical IV types.
  // TODO: Emit remarks for failed transformations.
  if (Ty != VPTypeAnalysis(Plan).inferScalarType(WideIV))
    return false;

  auto *FindIVSelectR = cast<VPSingleDefRecipe>(
      FindLastIVPhiR->getBackedgeValue()->getDefiningRecipe());
  assert(
      match(FindIVSelectR, m_Select(m_VPValue(), m_VPValue(), m_VPValue())) &&
      "backedge value must be a select");
  if (FindIVSelectR->getOperand(1) != WideIV &&
      FindIVSelectR->getOperand(2) != WideIV)
    return false;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1902-2045

```cpp
  // If the original wide IV is not canonical, create a new one. The canonical
  // wide IV is guaranteed to not wrap for all lanes that are active in the
  // vector loop.
  if (!WideIV->isCanonical()) {
    VPIRValue *Zero = Plan.getConstantInt(Ty, 0);
    VPIRValue *One = Plan.getConstantInt(Ty, 1);
    auto *WidenCanIV = new VPWidenIntOrFpInductionRecipe(
        nullptr, Zero, One, WideIV->getVFValue(),
        WideIV->getInductionDescriptor(),
        VPIRFlags::WrapFlagsTy(/*HasNUW=*/true, /*HasNSW=*/false),
        WideIV->getDebugLoc());
    WidenCanIV->insertBefore(WideIV);

    // Update the select to use the wide canonical IV.
    FindIVSelectR->setOperand(FindIVSelectR->getOperand(1) == WideIV ? 1 : 2,
                              WidenCanIV);
  }
  FindLastIVPhiR->setOperand(0, Plan.getOrAddLiveIn(PoisonValue::get(Ty)));

  // The reduction using MinOrMaxPhiR needs adjusting to compute the correct
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        RecurrenceDescriptor::isIntMinMaxRecurrenceKind(RdxKind) &&
        "only min/max recurrences support users outside the reduction chain");

    auto *MinOrMaxOp =
        dyn_cast<VPRecipeWithIRFlags>(MinOrMaxPhiR->getBackedgeValue());
    if (!MinOrMaxOp)
      return false;

    // Check that MinOrMaxOp is a VPWidenIntrinsicRecipe or VPReplicateRecipe
    // with an intrinsic that matches the reduction kind.
    Intrinsic::ID ExpectedIntrinsicID = getMinMaxReductionIntrinsicOp(RdxKind);
    if (!match(MinOrMaxOp, m_Intrinsic(ExpectedIntrinsicID)))
      return false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2046-2183

```cpp
    // MinOrMaxOp must have 2 users: 1) MinOrMaxPhiR and 2)
    // ComputeReductionResult.
    assert(MinOrMaxOp->getNumUsers() == 2 &&
           "MinOrMaxOp must have exactly 2 users");
    VPValue *MinOrMaxOpValue = MinOrMaxOp->getOperand(0);
    if (MinOrMaxOpValue == MinOrMaxPhiR)
      MinOrMaxOpValue = MinOrMaxOp->getOperand(1);

    VPValue *CmpOpA;
    VPValue *CmpOpB;
    CmpPredicate Pred;
    auto *Cmp = dyn_cast_or_null<VPRecipeWithIRFlags>(vputils::findUserOf(
        MinOrMaxPhiR, m_Cmp(Pred, m_VPValue(CmpOpA), m_VPValue(CmpOpB))));
    if (!Cmp || Cmp->getNumUsers() != 1 ||
        (CmpOpA != MinOrMaxOpValue && CmpOpB != MinOrMaxOpValue))
      return false;

    if (MinOrMaxOpValue != CmpOpB)
      Pred = CmpInst::getSwappedPredicate(Pred);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    //     correspond to the lanes matching the min/max reduction result.
    //
    // For example, this transforms
    // vp<%min.result> = compute-reduction-result ir<%min.val.next>
    // vp<%iv.rdx> = compute-reduction-result (smax) vp<%min.idx.next>
    // vp<%cmp> = icmp ne vp<%iv.rdx>, SENTINEL
    // vp<%find.iv.result> = select vp<%cmp>, vp<%iv.rdx>, ir<0>
    //
    // into:
    //
    // vp<min.result> = compute-reduction-result ir<%min.val.next>
    // vp<%final.min.cmp> = icmp eq ir<%min.val.next>, vp<min.result>
    // vp<%final.iv> = select vp<%final.min.cmp>, vp<%min.idx.next>, SENTINEL
    // vp<%iv.rdx> = compute-reduction-result (smax) vp<%final.iv>
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2184-2198

```cpp
    // vp<%cmp> = icmp ne vp<%iv.rdx>, SENTINEL
    // vp<%find.iv.result> = select vp<%cmp>, vp<%iv.rdx>, ir<0>
    //
    VPBuilder B(FindIVRdxResult);
    VPValue *MinOrMaxExiting = MinOrMaxResult->getOperand(0);
    auto *FinalMinOrMaxCmp =
        B.createICmp(CmpInst::ICMP_EQ, MinOrMaxExiting, MinOrMaxResult);
    VPValue *Sentinel = FindIVCmp->getOperand(1);
    VPValue *LastIVExiting = FindIVRdxResult->getOperand(0);
    auto *FinalIVSelect =
        B.createSelect(FinalMinOrMaxCmp, LastIVExiting, Sentinel);
    FindIVRdxResult->setOperand(0, FinalIVSelect);
  }
  return true;
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `PlainCFGBuilder, setVPBBPredsFromBB, isHeaderBB, fixHeaderPhis, isExternalDef, buildPlainCFG, createLoopRegion, addCanonicalIVRecipes` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PlainCFGBuilder, setVPBBPredsFromBB, isHeaderBB, fixHeaderPhis, isExternalDef, buildPlainCFG, createLoopRegion, addCanonicalIVRecipes` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/Loads.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/Loads.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopIterator.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/InstrTypes.h`, `llvm/IR/MDBuilder.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/InstrTypes.h`, `llvm/IR/MDBuilder.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `LoopVectorizationPlanner.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`LoopVectorizationPlanner.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
