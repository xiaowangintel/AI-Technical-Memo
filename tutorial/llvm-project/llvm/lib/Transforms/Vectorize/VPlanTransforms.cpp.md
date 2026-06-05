# VPlanTransforms.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanTransforms.cpp` | `llvm/lib/Transforms/Vectorize/VPlanTransforms.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements utility VPlan to VPlan transforms within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanTransforms 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-351

```cpp
//===-- VPlanTransforms.cpp - Utility VPlan to VPlan transforms -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements a set of utility VPlan to VPlan transformations.
///
//===----------------------------------------------------------------------===//

#include "VPlanTransforms.h"
#include "VPRecipeBuilder.h"
#include "VPlan.h"
#include "VPlanAnalysis.h"
#include "VPlanCFG.h"
#include "VPlanDominatorTree.h"
#include "VPlanHelpers.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    // All recipe users of SinkCandidate must be in the same block SinkTo or all
    // users outside of SinkTo must only use the first lane of SinkCandidate. In
    // the latter case, we need to duplicate SinkCandidate.
    auto UsersOutsideSinkTo =
        make_filter_range(SinkCandidate->users(), [SinkTo](VPUser *U) {
          return cast<VPRecipeBase>(U)->getParent() != SinkTo;
        });
    if (any_of(UsersOutsideSinkTo, [SinkCandidate](VPUser *U) {
          return !U->usesFirstLaneOnly(SinkCandidate);
        }))
      continue;
    bool NeedsDuplicating = !UsersOutsideSinkTo.empty();

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include make_early_inc_range, SinkStoreInfo, isNoAliasViaDistance, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 make_early_inc_range, SinkStoreInfo, isNoAliasViaDistance，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 352-700

```cpp
    if (NeedsDuplicating) {
      if (ScalarVFOnly)
        continue;
      VPSingleDefRecipe *Clone;
      if (auto *SinkCandidateRepR =
              dyn_cast<VPReplicateRecipe>(SinkCandidate)) {
        // TODO: Handle converting to uniform recipes as separate transform,
        // then cloning should be sufficient here.
        Instruction *I = SinkCandidate->getUnderlyingInstr();
        Clone = new VPReplicateRecipe(I, SinkCandidate->operands(), true,
                                      nullptr /*Mask*/, *SinkCandidateRepR,
                                      *SinkCandidateRepR);
        // TODO: add ".cloned" suffix to name of Clone's VPValue.
      } else {
        Clone = SinkCandidate->clone();
      }

      Clone->insertBefore(SinkCandidate);
      SinkCandidate->replaceUsesWithIf(Clone, [SinkTo](VPUser &U, unsigned) {
        return cast<VPRecipeBase>(&U)->getParent() != SinkTo;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (ResultTy != StepTy) {
    assert(StepTy->getScalarSizeInBits() > ResultTy->getScalarSizeInBits() &&
           "Not truncating.");
    assert(StepTy->isIntegerTy() && "Truncation requires an integer type");
    auto *VecPreheader =
        cast<VPBasicBlock>(HeaderVPBB->getSingleHierarchicalPredecessor());
    VPBuilder::InsertPointGuard Guard(Builder);
    Builder.setInsertPoint(VecPreheader);
    Step = Builder.createScalarCast(Instruction::Trunc, Step, ResultTy, DL);
  }
  return Builder.createScalarIVSteps(InductionOpcode, FPBinOp, BaseIV, Step,
                                     &Plan.getVF(), DL);
}

```
- EN: Core entities appearing here include mergeReplicateRegionsIntoSuccessors, vp_depth_first_deep, addReplicateRegions, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 mergeReplicateRegionsIntoSuccessors, vp_depth_first_deep, addReplicateRegions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 701-1041

```cpp
void VPlanTransforms::replaceWideCanonicalIVWithWideIV(
    VPlan &Plan, ScalarEvolution &SE, const TargetTransformInfo &TTI,
    TargetTransformInfo::TargetCostKind CostKind, ElementCount VF, unsigned UF,
    const SmallPtrSetImpl<const Value *> &ValuesToIgnore) {
  VPRegionBlock *LoopRegion = Plan.getVectorLoopRegion();
  if (!LoopRegion)
    return;

  auto *WideCanIV = vputils::findUserOf<VPWidenCanonicalIVRecipe>(
      LoopRegion->getCanonicalIV());
  if (!WideCanIV)
    return;

  Type *CanIVTy = LoopRegion->getCanonicalIVType();

  // Replace the wide canonical IV with a scalar-iv-steps over the canonical
  // IV.
  if (Plan.hasScalarVFOnly() || vputils::onlyFirstLaneUsed(WideCanIV)) {
    VPBuilder Builder(WideCanIV);
    WideCanIV->replaceAllUsesWith(createScalarIVSteps(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Calculate the final index.
  VPRegionBlock *LoopRegion = Plan.getVectorLoopRegion();
  auto *CanonicalIV = LoopRegion->getCanonicalIV();
  Type *CanonicalIVType = LoopRegion->getCanonicalIVType();
  auto *ExtractR = cast<VPInstruction>(Op);
  VPBuilder B(ExtractR);

  DebugLoc DL = ExtractR->getDebugLoc();
  VPValue *FirstActiveLane = B.createFirstActiveLane(Mask, DL);
  Type *FirstActiveLaneType = TypeInfo.inferScalarType(FirstActiveLane);
  FirstActiveLane = B.createScalarZExtOrTrunc(FirstActiveLane, CanonicalIVType,
                                              FirstActiveLaneType, DL);
  VPValue *EndValue = B.createAdd(CanonicalIV, FirstActiveLane, DL);

```
- EN: Core entities appearing here include isDeadRecipe, removeDeadRecipes, collectUsersRecursively, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isDeadRecipe, removeDeadRecipes, collectUsersRecursively，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1042-1400

```cpp
  // `getOptimizableIVOf()` always returns the pre-incremented IV, so if it
  // changed it means the exit is using the incremented value, so we need to
  // add the step.
  if (Incoming != WideIV) {
    VPValue *One = Plan.getConstantInt(CanonicalIVType, 1);
    EndValue = B.createAdd(EndValue, One, DL);
  }

  if (!WideIntOrFp || !WideIntOrFp->isCanonical()) {
    const InductionDescriptor &ID = WideIV->getInductionDescriptor();
    VPIRValue *Start = WideIV->getStartValue();
    VPValue *Step = WideIV->getStepValue();
    EndValue = B.createDerivedIV(
        ID.getKind(), dyn_cast_or_null<FPMathOperator>(ID.getInductionBinOp()),
        Start, EndValue, Step);
  }

  return EndValue;
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }
#ifndef NDEBUG
    // Verify that the cached type info is for both A and its users is still
    // accurate by comparing it to freshly computed types.
    VPTypeAnalysis TypeInfo2(*Plan);
    assert(TypeInfo.inferScalarType(A) == TypeInfo2.inferScalarType(A));
    for (VPUser *U : A->users()) {
      auto *R = cast<VPRecipeBase>(U);
      for (VPValue *VPV : R->definedValues())
        assert(TypeInfo.inferScalarType(VPV) == TypeInfo2.inferScalarType(VPV));
    }
#endif
  }

```
- EN: Core entities appearing here include removeRedundantExpandSCEVRecipes, make_early_inc_range, recursivelyDeleteDeadRecipes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 removeRedundantExpandSCEVRecipes, make_early_inc_range, recursivelyDeleteDeadRecipes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1401-1748

```cpp
  // Simplify (X && Y) | (X && !Y) -> X.
  // TODO: Split up into simpler, modular combines: (X && Y) | (X && Z) into X
  // && (Y | Z) and (X | !X) into true. This requires queuing newly created
  // recipes to be visited during simplification.
  VPValue *X, *Y, *Z;
  if (match(Def,
            m_c_BinaryOr(m_LogicalAnd(m_VPValue(X), m_VPValue(Y)),
                         m_LogicalAnd(m_Deferred(X), m_Not(m_Deferred(Y)))))) {
    Def->replaceAllUsesWith(X);
    Def->eraseFromParent();
    return;
  }

  // x | AllOnes -> AllOnes
  if (match(Def, m_c_BinaryOr(m_VPValue(X), m_AllOnes())))
    return Def->replaceAllUsesWith(
        Plan->getAllOnesValue(TypeInfo.inferScalarType(Def)));

  // x | 0 -> x
  if (match(Def, m_c_BinaryOr(m_VPValue(X), m_ZeroInt())))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // If Phi has a second user (besides IVInc's defining recipe), it must
      // be Inc = Phi + Y for the fold to apply.
      auto *Inc = dyn_cast_or_null<VPSingleDefRecipe>(
          vputils::findUserOf(Phi, m_Add(m_Specific(Phi), m_Specific(Y))));
      if (Phi->getNumUsers() == 1 || (Phi->getNumUsers() == 2 && Inc)) {
        Def->replaceAllUsesWith(IVInc);
        if (Inc)
          Inc->replaceAllUsesWith(Phi);
        Phi->setOperand(0, Y);
        return;
      }
    }
  }

```
- EN: Core entities appearing here include m_LogicalAnd, match, m_Select, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_LogicalAnd, match, m_Select，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1749-2089

```cpp
  // Simplify unrolled VectorPointer without offset, or with zero offset, to
  // just the pointer operand.
  if (auto *VPR = dyn_cast<VPVectorPointerRecipe>(Def))
    if (!VPR->getOffset() || match(VPR->getOffset(), m_ZeroInt()))
      return VPR->replaceAllUsesWith(VPR->getOperand(0));

  // VPScalarIVSteps after unrolling can be replaced by their start value, if
  // the start index is zero and only the first lane 0 is demanded.
  if (auto *Steps = dyn_cast<VPScalarIVStepsRecipe>(Def)) {
    if (!Steps->getStartIndex() && vputils::onlyFirstLaneUsed(Steps)) {
      Steps->replaceAllUsesWith(Steps->getOperand(0));
      return;
    }
  }
  // Simplify redundant ReductionStartVector recipes after unrolling.
  VPValue *StartV;
  if (match(Def, m_VPInstruction<VPInstruction::ReductionStartVector>(
                     m_VPValue(StartV), m_VPValue(), m_VPValue()))) {
    Def->replaceUsesWithIf(StartV, [](const VPUser &U, unsigned Idx) {
      auto *PhiR = dyn_cast<VPReductionPHIRecipe>(&U);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (!WideIV || !WideIV->isCanonical() ||
        WideIV->hasMoreThanOneUniqueUser() ||
        NewIVTy == WideIV->getScalarType())
      continue;

    // Currently only handle cases where the single user is a header-mask
    // comparison with the backedge-taken-count.
    VPUser *SingleUser = WideIV->getSingleUser();
    if (!SingleUser ||
        !match(SingleUser,
               m_ICmp(m_Specific(WideIV),
                      m_Broadcast(m_Specific(Plan.getBackedgeTakenCount())))))
      continue;

```
- EN: Core entities appearing here include m_VPValue, isSingleScalar, simplifyRecipes, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_VPValue, isSingleScalar, simplifyRecipes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2090-2463

```cpp
    // Update IV operands and comparison bound to use new narrower type.
    auto *NewStart = Plan.getZero(NewIVTy);
    WideIV->setStartValue(NewStart);
    auto *NewStep = Plan.getConstantInt(NewIVTy, 1);
    WideIV->setStepValue(NewStep);

    auto *NewBTC = new VPWidenCastRecipe(
        Instruction::Trunc, Plan.getOrCreateBackedgeTakenCount(), NewIVTy,
        nullptr, VPIRFlags::getDefaultFlags(Instruction::Trunc));
    Plan.getVectorPreheader()->appendRecipe(NewBTC);
    auto *Cmp = cast<VPInstruction>(WideIV->getSingleUser());
    Cmp->setOperand(1, NewBTC);

    MadeChange = true;
  }

  return MadeChange;
}

/// Return true if \p Cond is known to be true for given \p BestVF and \p
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // the same parent.
    const VPRegionBlock *RegionL = L->getRegion();
    const VPRegionBlock *RegionR = R->getRegion();
    if (((RegionL && RegionL->isReplicator()) ||
         (RegionR && RegionR->isReplicator())) &&
        L->getParent() != R->getParent())
      return false;
    const VPlan *Plan = L->getParent()->getPlan();
    VPTypeAnalysis TypeInfo(*Plan);
    return TypeInfo.inferScalarType(L) == TypeInfo.inferScalarType(R);
  }
};
} // end anonymous namespace

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include m_VPValue, match, vp_depth_first_deep, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 m_VPValue, match, vp_depth_first_deep，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution, SCEV 等分析结果。

### Lines 2464-2812

```cpp
/// Perform a common-subexpression-elimination of VPSingleDefRecipes on the \p
/// Plan.
void VPlanTransforms::cse(VPlan &Plan) {
  VPDominatorTree VPDT(Plan);
  DenseMap<VPSingleDefRecipe *, VPSingleDefRecipe *, VPCSEDenseMapInfo> CSEMap;

  ReversePostOrderTraversal<VPBlockDeepTraversalWrapper<VPBlockBase *>> RPOT(
      Plan.getEntry());
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksOnly<VPBasicBlock>(RPOT)) {
    for (VPRecipeBase &R : *VPBB) {
      auto *Def = dyn_cast<VPSingleDefRecipe>(&R);
      if (!Def || !VPCSEDenseMapInfo::canHandle(Def))
        continue;
      if (VPSingleDefRecipe *V = CSEMap.lookup(Def)) {
        // V must dominate Def for a valid replacement.
        if (!VPDT.dominates(V->getParent(), VPBB))
          continue;
        // Only keep flags present on both V and Def.
        if (auto *RFlags = dyn_cast<VPRecipeWithIRFlags>(V))
          RFlags->intersectFlags(*cast<VPRecipeWithIRFlags>(Def));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
addVPLaneMaskPhiAndUpdateExitBranch(VPlan &Plan) {
  VPRegionBlock *TopRegion = Plan.getVectorLoopRegion();
  VPBasicBlock *EB = TopRegion->getExitingBasicBlock();
  VPValue *StartV = Plan.getZero(TopRegion->getCanonicalIVType());
  auto *CanonicalIVIncrement = TopRegion->getOrCreateCanonicalIVIncrement();
  // TODO: Check if dropping the flags is needed.
  TopRegion->clearCanonicalIVNUW(CanonicalIVIncrement);
  DebugLoc DL = CanonicalIVIncrement->getDebugLoc();
  // We can't use StartV directly in the ActiveLaneMask VPInstruction, since
  // we have to take unrolling into account. Each part needs to start at
  //   Part * VF
  auto *VecPreheader = Plan.getVectorPreheader();
  VPBuilder Builder(VecPreheader);

```
- EN: Core entities appearing here include cse, licm, vp_depth_first_shallow, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 cse, licm, vp_depth_first_shallow，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2813-3165

```cpp
  // Create the ActiveLaneMask instruction using the correct start values.
  VPValue *TC = Plan.getTripCount();
  VPValue *VF = &Plan.getVF();

  auto *EntryIncrement = Builder.createOverflowingOp(
      VPInstruction::CanonicalIVIncrementForPart, {StartV, VF}, {false, false},
      DL, "index.part.next");

  // Create the active lane mask instruction in the VPlan preheader.
  VPValue *ALMMultiplier =
      Plan.getConstantInt(TopRegion->getCanonicalIVType(), 1);
  auto *EntryALM = Builder.createNaryOp(VPInstruction::ActiveLaneMask,
                                        {EntryIncrement, TC, ALMMultiplier}, DL,
                                        "active.lane.mask.entry");

  // Now create the ActiveLaneMaskPhi recipe in the main loop using the
  // preheader ActiveLaneMask instruction.
  auto *LaneMaskPhi =
      new VPActiveLaneMaskPHIRecipe(EntryALM, DebugLoc::getUnknown());
  auto *HeaderVPBB = TopRegion->getEntryBasicBlock();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          continue;
        VPValue *Imm = Plan.getOrAddLiveIn(
            ConstantInt::getSigned(Type::getInt32Ty(Plan.getContext()), -1));
        VPWidenIntrinsicRecipe *VPSplice = new VPWidenIntrinsicRecipe(
            Intrinsic::experimental_vp_splice,
            {V1, V2, Imm, Plan.getTrue(), PrevEVL, &EVL},
            TypeInfo.inferScalarType(R.getVPSingleValue()), {}, {},
            R.getDebugLoc());
        VPSplice->insertBefore(&R);
        R.getVPSingleValue()->replaceAllUsesWith(VPSplice);
      }
    }
  }

```
- EN: Core entities appearing here include h, getVPDivRemIntrinsic, match, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 h, getVPDivRemIntrinsic, match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3166-3510

```cpp
  VPValue *HeaderMask = vputils::findHeaderMask(Plan);
  if (!HeaderMask)
    return;

  // Ensure that any reduction that uses a select to mask off tail lanes does so
  // in the vector loop, not the middle block, since EVL tail folding can have
  // tail elements in the penultimate iteration.
  assert(all_of(*Plan.getMiddleBlock(), [&Plan, HeaderMask](VPRecipeBase &R) {
    if (match(&R, m_ComputeReductionResult(m_Select(m_Specific(HeaderMask),
                                                    m_VPValue(), m_VPValue()))))
      return R.getOperand(0)->getDefiningRecipe()->getRegion() ==
             Plan.getVectorLoopRegion();
    return true;
  }));

  // Replace header masks with a mask equivalent to predicating by EVL:
  //
  // icmp ule widen-canonical-iv backedge-taken-count
  // ->
  // icmp ult step-vector, EVL
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          RecWithFlags->replaceAllUsesWith(New);
          RecWithFlags->eraseFromParent();
          CurRec = New;
        } else
          RecWithFlags->dropPoisonGeneratingFlags();
      } else {
        Instruction *Instr = dyn_cast_or_null<Instruction>(
            CurRec->getVPSingleValue()->getUnderlyingValue());
        (void)Instr;
        assert((!Instr || !Instr->hasPoisonGeneratingFlags()) &&
               "found instruction with poison generating flags not covered by "
               "VPRecipeWithIRFlags");
      }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include convertToVariableLengthStep, m_c_Add, convertEVLExitCond, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 convertToVariableLengthStep, m_c_Add, convertEVLExitCond，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 3511-3878

```cpp
      // Add new definitions to the worklist.
      for (VPValue *Operand : CurRec->operands())
        if (VPRecipeBase *OpDef = Operand->getDefiningRecipe())
          Worklist.push_back(OpDef);
    }
  });

  // We want to exclude the tail folding case, as we don't need to drop flags
  // for operations computing the first lane in this case: the first lane of the
  // header mask must always be true.
  auto IsNotHeaderMask = [&Plan](VPValue *Mask) {
    return Mask && !vputils::isHeaderMask(Mask, Plan);
  };

  // Traverse all the recipes in the VPlan and collect the poison-generating
  // recipes in the backward slice starting at the address of a VPWidenRecipe or
  // VPInterleaveRecipe.
  auto Iter =
      vp_depth_first_shallow(Plan.getVectorLoopRegion()->getEntryBasicBlock());
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksOnly<VPBasicBlock>(Iter)) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
           (FPBinOp->getOpcode() == Instruction::FAdd ||
            FPBinOp->getOpcode() == Instruction::FSub) &&
           "Original BinOp should be defined for FP induction");
    FastMathFlags FMF = FPBinOp->getFastMathFlags();
    VPValue *FMul = Builder.createNaryOp(Instruction::FMul, {Step, Index}, FMF);
    return R->replaceAllUsesWith(
        Builder.createNaryOp(FPBinOp->getOpcode(), {Start, FMul}, FMF));
  }
  case InductionDescriptor::IK_NoInduction:
    return;
  }
  llvm_unreachable("Unhandled induction kind");
}

```
- EN: Core entities appearing here include expandVPDerivedIV, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandVPDerivedIV，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3879-4266

```cpp
void VPlanTransforms::dissolveLoopRegions(VPlan &Plan) {
  // Replace loop regions with explicity CFG.
  SmallVector<VPRegionBlock *> LoopRegions;
  for (VPRegionBlock *R : VPBlockUtils::blocksOnly<VPRegionBlock>(
           vp_depth_first_deep(Plan.getEntry()))) {
    if (!R->isReplicator())
      LoopRegions.push_back(R);
  }
  for (VPRegionBlock *R : LoopRegions)
    R->dissolveToCFGLoop();
}

void VPlanTransforms::expandBranchOnTwoConds(VPlan &Plan) {
  SmallVector<VPInstruction *> WorkList;
  // The transform runs after dissolving loop regions, so all VPBasicBlocks
  // terminated with BranchOnTwoConds are reached via a shallow traversal.
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksAs<VPBasicBlock>(
           vp_depth_first_shallow(Plan.getEntry()))) {
    if (!VPBB->empty() && match(&VPBB->back(), m_BranchOnTwoConds()))
      WorkList.push_back(cast<VPInstruction>(&VPBB->back()));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      auto *ExitIRI = cast<VPIRPhi>(&R);
      VPValue *IncomingVal =
          ExitIRI->getIncomingValueForBlock(EarlyExitingVPBB);
      VPValue *NewIncoming = IncomingVal;
      if (!isa<VPIRValue>(IncomingVal)) {
        VPBuilder EarlyExitBuilder(VectorEarlyExitVPBB);
        NewIncoming = EarlyExitBuilder.createNaryOp(
            VPInstruction::ExtractLane, {FirstActiveLane, IncomingVal},
            DebugLoc::getUnknown(), "early.exit.value");
      }
      ExitIRI->removeIncomingValueFor(EarlyExitingVPBB);
      ExitIRI->addOperand(NewIncoming);
    }

```
- EN: Core entities appearing here include dissolveLoopRegions, vp_depth_first_deep, expandBranchOnTwoConds, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 dissolveLoopRegions, vp_depth_first_deep, expandBranchOnTwoConds，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4267-4605

```cpp
    EarlyExitingVPBB->getTerminator()->eraseFromParent();
    VPBlockUtils::disconnectBlocks(EarlyExitingVPBB, EarlyExitVPBB);
    VPBlockUtils::connectBlocks(VectorEarlyExitVPBB, EarlyExitVPBB);
  }

  // Chain through exits: for each exit, check if its condition is true at
  // the first active lane. If so, take that exit; otherwise, try the next.
  // The last exit needs no check since it must be taken if all others fail.
  //
  // For 3 exits (cond.0, cond.1, cond.2), this creates:
  //
  // latch:
  //   ...
  //   EMIT vp<%combined> = logical-or vp<%cond.0>, vp<%cond.1>, vp<%cond.2>
  //   ...
  //
  // vector.early.exit.check:
  //   EMIT vp<%first.lane> = first-active-lane vp<%combined>
  //   EMIT vp<%at.cond.0> = extract-lane vp<%first.lane>, vp<%cond.0>
  //   EMIT branch-on-cond vp<%at.cond.0>
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (Plan.hasScalarVFOnly())
    return;

#ifndef NDEBUG
  VPDominatorTree VPDT(Plan);
#endif

  SmallVector<VPValue *> VPValues;
  if (VPValue *BTC = Plan.getBackedgeTakenCount())
    VPValues.push_back(BTC);
  append_range(VPValues, Plan.getLiveIns());
  for (VPRecipeBase &R : *Plan.getEntry())
    append_range(VPValues, R.definedValues());

```
- EN: Core entities appearing here include IsMulAccValidAndClampRange, vp_depth_first_deep, materializeBroadcasts, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 IsMulAccValidAndClampRange, vp_depth_first_deep, materializeBroadcasts，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4606-4947

```cpp
  auto *VectorPreheader = Plan.getVectorPreheader();
  for (VPValue *VPV : VPValues) {
    if (vputils::onlyScalarValuesUsed(VPV) ||
        (isa<VPIRValue>(VPV) && isa<Constant>(VPV->getLiveInIRValue())))
      continue;

    // Add explicit broadcast at the insert point that dominates all users.
    VPBasicBlock *HoistBlock = VectorPreheader;
    VPBasicBlock::iterator HoistPoint = VectorPreheader->end();
    for (VPUser *User : VPV->users()) {
      if (User->usesScalars(VPV))
        continue;
      if (cast<VPRecipeBase>(User)->getParent() == VectorPreheader)
        HoistPoint = HoistBlock->begin();
      else
        assert(VPDT.dominates(VectorPreheader,
                              cast<VPRecipeBase>(User)->getParent()) &&
               "All users must be in the vector preheader or dominated by it");
    }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  for (VPBasicBlock *VPBB : VPBBsInsideLoopRegion) {
    for (VPRecipeBase &R : make_early_inc_range(*VPBB)) {
      if (isa<VPReplicateRecipe, VPInstruction, VPScalarIVStepsRecipe,
              VPDerivedIVRecipe>(&R))
        continue;
      for (VPValue *Def : R.definedValues()) {
        // Skip recipes that are single-scalar or only have their first lane
        // used.
        // TODO: The Defs skipped here may or may not be vector values.
        // Introduce Unpacks, and remove them later, if they are guaranteed to
        // produce scalar values.
        if (vputils::isSingleScalar(Def) || vputils::onlyFirstLaneUsed(Def))
          continue;

```
- EN: Core entities appearing here include getCommonMetadata, findRecipeWithMinAlign, materializePacksAndUnpacks, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getCommonMetadata, findRecipeWithMinAlign, materializePacksAndUnpacks，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4948-5290

```cpp
        // At the moment, we create unpacks only for scalar users outside
        // replicate regions. Recipes inside replicate regions still extract the
        // required lanes implicitly.
        // TODO: Remove once replicate regions are unrolled completely.
        auto IsCandidateUnpackUser = [Def](VPUser *U) {
          VPRegionBlock *ParentRegion = cast<VPRecipeBase>(U)->getRegion();
          return U->usesScalars(Def) &&
                 (!ParentRegion || !ParentRegion->isReplicator());
        };
        if (none_of(Def->users(), IsCandidateUnpackUser))
          continue;

        auto *Unpack = new VPInstruction(VPInstruction::Unpack, {Def});
        if (R.isPhi())
          Unpack->insertBefore(*VPBB, VPBB->getFirstNonPhi());
        else
          Unpack->insertAfter(&R);
        Def->replaceUsesWithIf(Unpack,
                               [&IsCandidateUnpackUser](VPUser &U, unsigned) {
                                 return IsCandidateUnpackUser(&U);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto *WideLoad = cast<VPWidenLoadRecipe>(R);
  VPValue *PtrOp = WideLoad->getAddr();
  if (auto *VecPtr = dyn_cast<VPVectorPointerRecipe>(PtrOp))
    PtrOp = VecPtr->getOperand(0);
  // Narrow wide load to uniform scalar load, as transformed VPlan will only
  // process one original iteration.
  auto *N = new VPReplicateRecipe(&WideLoad->getIngredient(), {PtrOp},
                                  /*IsUniform*/ true,
                                  /*Mask*/ nullptr, {}, *WideLoad);
  N->insertBefore(WideLoad);
  NarrowedOps.insert(N);
  return N;
}

```
- EN: Core entities appearing here include expandSCEVs, canNarrowOps, isAlreadyNarrow, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandSCEVs, canNarrowOps, isAlreadyNarrow，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 5291-5629

```cpp
std::unique_ptr<VPlan>
VPlanTransforms::narrowInterleaveGroups(VPlan &Plan,
                                        const TargetTransformInfo &TTI) {
  VPRegionBlock *VectorLoop = Plan.getVectorLoopRegion();

  if (!VectorLoop)
    return nullptr;

  // Only handle single-block loops for now.
  if (VectorLoop->getEntryBasicBlock() != VectorLoop->getExitingBasicBlock())
    return nullptr;

  // Skip plans when we may not be able to properly narrow.
  VPBasicBlock *Exiting = VectorLoop->getExitingBasicBlock();
  if (!match(&Exiting->back(), m_BranchOnCount()))
    return nullptr;

  assert(match(&Exiting->back(),
               m_BranchOnCount(m_Add(m_VPValue(), m_Specific(&Plan.getVFxUF())),
                               m_Specific(&Plan.getVectorTripCount()))) &&
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      Instruction::isIntDivRem(BinOp->getOpcode()))
    return nullptr;

  VPValue *WidenIVCandidate = BinOp->getOperand(0);
  VPValue *InvariantCandidate = BinOp->getOperand(1);
  if (!isa<VPWidenIntOrFpInductionRecipe>(WidenIVCandidate))
    std::swap(WidenIVCandidate, InvariantCandidate);

  if (!InvariantCandidate->isDefinedOutsideLoopRegions())
    return nullptr;

  return dyn_cast<VPWidenIntOrFpInductionRecipe>(WidenIVCandidate);
}

```
- EN: Core entities appearing here include all_of, make_range, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of, make_range，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 5630-5970

```cpp
/// Create a scalar version of \p BinOp, with its \p WidenIV operand replaced
/// by \p ScalarIV, and place it after \p ScalarIV's defining recipe.
static VPValue *cloneBinOpForScalarIV(VPWidenRecipe *BinOp, VPValue *ScalarIV,
                                      VPWidenIntOrFpInductionRecipe *WidenIV) {
  assert(Instruction::isBinaryOp(BinOp->getOpcode()) &&
         BinOp->getNumOperands() == 2 && "BinOp must have 2 operands");
  auto *ClonedOp = BinOp->clone();
  if (ClonedOp->getOperand(0) == WidenIV) {
    ClonedOp->setOperand(0, ScalarIV);
  } else {
    assert(ClonedOp->getOperand(1) == WidenIV && "one operand must be WideIV");
    ClonedOp->setOperand(1, ScalarIV);
  }
  ClonedOp->insertAfter(ScalarIV->getDefiningRecipe());
  return ClonedOp;
}

void VPlanTransforms::optimizeFindIVReductions(VPlan &Plan,
                                               PredicatedScalarEvolution &PSE,
                                               Loop &L) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        MulLHS->getOpcode() != MulRHS->getOpcode())
      return Op;
    VPBuilder Builder(Mul);
    Mul->setOperand(0, Builder.createWidenCast(MulLHS->getOpcode(),
                                               MulLHS->getOperand(0),
                                               Ext->getResultType()));
    Mul->setOperand(1, MulLHS == MulRHS
                           ? Mul->getOperand(0)
                           : Builder.createWidenCast(MulRHS->getOpcode(),
                                                     MulRHS->getOperand(0),
                                                     Ext->getResultType()));
    return Mul;
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include make_early_inc_range, m_scev_AffineAddRec, CheckSentinel, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 make_early_inc_range, m_scev_AffineAddRec, CheckSentinel，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 5971-6311

```cpp
  return Op;
}

static VPExpressionRecipe *
createPartialReductionExpression(VPReductionRecipe *Red) {
  VPValue *VecOp = Red->getVecOp();

  // reduce.[f]add(ext(op))
  //  -> VPExpressionRecipe(op, red)
  if (match(VecOp, m_WidenAnyExtend(m_VPValue())))
    return new VPExpressionRecipe(cast<VPWidenCastRecipe>(VecOp), Red);

  // reduce.[f]add([f]mul(ext(a), ext(b)))
  //  -> VPExpressionRecipe(a, b, mul, red)
  if (match(VecOp, m_FMul(m_FPExt(m_VPValue()), m_FPExt(m_VPValue()))) ||
      match(VecOp,
            m_Mul(m_ZExtOrSExt(m_VPValue()), m_ZExtOrSExt(m_VPValue())))) {
    auto *Mul = cast<VPWidenRecipe>(VecOp);
    auto *ExtA = cast<VPWidenCastRecipe>(Mul->getOperand(0));
    auto *ExtB = cast<VPWidenCastRecipe>(Mul->getOperand(1));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  RecurKind RK = RedPhiR->getRecurrenceKind();
  Type *PhiType = TypeInfo.inferScalarType(RedPhiR);
  TypeSize PHISize = PhiType->getPrimitiveSizeInBits();

  // Work backwards from the ExitValue examining each reduction operation.
  VPValue *CurrentValue = ExitValue;
  while (CurrentValue != RedPhiR) {
    auto *UpdateR = dyn_cast<VPWidenRecipe>(CurrentValue);
    if (!UpdateR || !Instruction::isBinaryOp(UpdateR->getOpcode()))
      return std::nullopt;

    VPValue *Op = UpdateR->getOperand(1);
    VPValue *PrevValue = UpdateR->getOperand(0);

```
- EN: Core entities appearing here include createPartialReductionExpression, m_Mul, m_ZExtOrSExt, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createPartialReductionExpression, m_Mul, m_ZExtOrSExt，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 6312-6655

```cpp
    // Find the extended operand. The other operand (PrevValue) is the next link
    // in the reduction chain.
    std::optional<ExtendedReductionOperand> ExtendedOp =
        matchExtendedReductionOperand(UpdateR, Op, TypeInfo);
    if (!ExtendedOp) {
      ExtendedOp = matchExtendedReductionOperand(UpdateR, PrevValue, TypeInfo);
      if (!ExtendedOp)
        return std::nullopt;
      std::swap(Op, PrevValue);
    }

    Type *ExtSrcType = ExtendedOp->ExtendA.SrcType;
    TypeSize ExtSrcSize = ExtSrcType->getPrimitiveSizeInBits();
    if (!PHISize.hasKnownScalarFactor(ExtSrcSize))
      return std::nullopt;

    // Check if a partial reduction chain is supported by the target (i.e. does
    // not have an invalid cost) for the given VF range. Clamps the range and
    // returns true if feasible for any VF.
    VPPartialReductionChain Link(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Pseudo intrinsics (assume, lifetime, ...) are always scalarized.
  if (ID && VPCostContext::isFreeScalarIntrinsic(ID))
    return CallWideningDecision::KindTy::Scalarize;

  InstructionCost ScalarCost =
      VPReplicateRecipe::computeCallCost(CalledFn, ResultTy, Ops,
                                         /*IsSingleScalar=*/false, VF, CostCtx);

  Function *VecFunc = findVectorVariant(CI, Ops, VF, MaskRequired, CostCtx.PSE,
                                        CostCtx.L, CostCtx.Types);
  InstructionCost VecCallCost = InstructionCost::getInvalid();
  if (VecFunc)
    VecCallCost = VPWidenCallRecipe::computeCallCost(VecFunc, CostCtx);

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include makeScalarizationDecisions, CallWideningDecision, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 makeScalarizationDecisions, CallWideningDecision，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 6656-6741

```cpp
  // Prefer the intrinsic if it is at least as cheap as scalarizing and any
  // available vector variant.
  if (ID) {
    InstructionCost IntrinsicCost =
        VPWidenIntrinsicRecipe::computeCallCost(ID, Ops, VPI, VF, CostCtx);
    if (IntrinsicCost.isValid() && ScalarCost >= IntrinsicCost &&
        (!VecFunc || VecCallCost >= IntrinsicCost))
      return CallWideningDecision::KindTy::Intrinsic;
  }

  // Otherwise, use a vector library variant when it beats scalarizing.
  if (VecFunc && ScalarCost >= VecCallCost)
    return {CallWideningDecision::KindTy::VectorVariant, VecFunc};

  return CallWideningDecision::KindTy::Scalarize;
}

void VPlanTransforms::makeCallWideningDecisions(VPlan &Plan, VFRange &Range,
                                                VPRecipeBuilder &RecipeBuilder,
                                                VPCostContext &CostCtx) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                        return true;
                      auto Legacy = CostCtx.getLegacyCallKind(CI, VF);
                      return !Legacy || *Legacy == Decision.Kind;
                    }) &&
             "VPlan call widening decision must match legacy decision");

      Replacement->insertBefore(VPI);
      VPI->replaceAllUsesWith(Replacement);
      ToErase.push_back(VPI);
    }
  }
  for (VPInstruction *VPI : ToErase)
    VPI->eraseFromParent();
}
```
- EN: Core entities appearing here include vp_depth_first_shallow, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 vp_depth_first_shallow，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `SinkStoreInfo, VPCSEDenseMapInfo, EarlyExitInfo, ReductionExtend, ExtendedReductionOperand, VPPartialReductionChain, CallWideningDecision, make_early_inc_range` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SinkStoreInfo, VPCSEDenseMapInfo, EarlyExitInfo, ReductionExtend, ExtendedReductionOperand, VPPartialReductionChain, CallWideningDecision, make_early_inc_range` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, LoopInfo, ScalarEvolution, SCEV, TargetLibraryInfo, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, LoopInfo, ScalarEvolution, SCEV, TargetLibraryInfo, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/InstSimplifyFolder.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h`, `llvm/Analysis/ScopedNoAliasAA.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/InstSimplifyFolder.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h`, `llvm/Analysis/ScopedNoAliasAA.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Intrinsics.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Metadata.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Intrinsics.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Metadata.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/TypeSize.h`, `llvm/ADT/APInt.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetOperations.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/TypeSwitch.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/TypeSize.h`, `llvm/ADT/APInt.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetOperations.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/TypeSwitch.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlanTransforms.h`, `VPRecipeBuilder.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlanTransforms.h`, `VPRecipeBuilder.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
