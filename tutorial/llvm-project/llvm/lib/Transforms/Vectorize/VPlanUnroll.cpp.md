# VPlanUnroll.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanUnroll.cpp` | `llvm/lib/Transforms/Vectorize/VPlanUnroll.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vPlan unroller within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanUnroll 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-89

```cpp
//===-- VPlanUnroll.cpp - VPlan unroller ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements explicit unrolling for VPlans.
///
//===----------------------------------------------------------------------===//

#include "VPRecipeBuilder.h"
#include "VPlan.h"
#include "VPlanAnalysis.h"
#include "VPlanCFG.h"
#include "VPlanHelpers.h"
#include "VPlanPatternMatch.h"
#include "VPlanTransforms.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

public:
  UnrollState(VPlan &Plan, unsigned UF) : Plan(Plan), UF(UF), TypeInfo(Plan) {}

  void unrollBlock(VPBlockBase *VPB);

  VPValue *getValueForPart(VPValue *V, unsigned Part) {
    if (Part == 0 || isa<VPIRValue, VPSymbolicValue, VPRegionValue>(V))
      return V;
    assert((VPV2Parts.contains(V) && VPV2Parts[V].size() >= Part) &&
           "accessed value does not exist");
    return VPV2Parts[V][Part - 1];
  }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include UnrollState, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 UnrollState，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 90-176

```cpp
  /// Given a single original recipe \p OrigR (of part zero), and its copy \p
  /// CopyR for part \p Part, map every VPValue defined by \p OrigR to its
  /// corresponding VPValue defined by \p CopyR.
  void addRecipeForPart(VPRecipeBase *OrigR, VPRecipeBase *CopyR,
                        unsigned Part) {
    for (const auto &[Idx, VPV] : enumerate(OrigR->definedValues())) {
      const auto &[V, _] = VPV2Parts.try_emplace(VPV);
      assert(V->second.size() == Part - 1 && "earlier parts not set");
      V->second.push_back(CopyR->getVPValue(Idx));
    }
  }

  /// Given a uniform recipe \p R, add it for all parts.
  void addUniformForAllParts(VPSingleDefRecipe *R) {
    const auto &[V, Inserted] = VPV2Parts.try_emplace(R);
    assert(Inserted && "uniform value already added");
    for (unsigned Part = 0; Part != UF; ++Part)
      V->second.push_back(R);
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (const auto &[PartIVPBB, Part0VPBB] :
         zip(VPBlockUtils::blocksAs<VPBasicBlock>(PartI),
             VPBlockUtils::blocksAs<VPBasicBlock>(Part0))) {
      for (const auto &[PartIR, Part0R] : zip(*PartIVPBB, *Part0VPBB)) {
        remapOperands(&PartIR, Part);
        if (auto *Steps = dyn_cast<VPScalarIVStepsRecipe>(&PartIR))
          addStartIndexForScalarSteps(Steps, Part, Plan, TypeInfo);

        addRecipeForPart(&Part0R, &PartIR, Part);
      }
    }
  }
}

```
- EN: Core entities appearing here include addUniformForAllParts, remapOperand, remapOperands, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addUniformForAllParts, remapOperand, remapOperands，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 177-261

```cpp
void UnrollState::unrollWidenInductionByUF(
    VPWidenInductionRecipe *IV, VPBasicBlock::iterator InsertPtForPhi) {
  VPBasicBlock *PH = cast<VPBasicBlock>(
      IV->getParent()->getEnclosingLoopRegion()->getSinglePredecessor());
  Type *IVTy = TypeInfo.inferScalarType(IV);
  auto &ID = IV->getInductionDescriptor();
  FastMathFlags FMF;
  VPIRFlags::WrapFlagsTy WrapFlags(false, false);
  if (auto *IntOrFPInd = dyn_cast<VPWidenIntOrFpInductionRecipe>(IV)) {
    if (IntOrFPInd->hasFastMathFlags())
      FMF = IntOrFPInd->getFastMathFlags();
    if (IntOrFPInd->hasNoWrapFlags())
      WrapFlags = IntOrFPInd->getNoWrapFlags();
  }

  VPValue *ScalarStep = IV->getStepValue();
  VPBuilder Builder(PH);
  Type *VectorStepTy =
      IVTy->isPointerTy() ? TypeInfo.inferScalarType(ScalarStep) : IVTy;
  VPInstruction *VectorStep = Builder.createNaryOp(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

void UnrollState::unrollHeaderPHIByUF(VPHeaderPHIRecipe *R,
                                      VPBasicBlock::iterator InsertPtForPhi) {
  // First-order recurrences pass a single vector or scalar through their header
  // phis, irrespective of interleaving.
  if (isa<VPFirstOrderRecurrencePHIRecipe>(R))
    return;

  // Generate step vectors for each unrolled part.
  if (auto *IV = dyn_cast<VPWidenInductionRecipe>(R)) {
    unrollWidenInductionByUF(IV, InsertPtForPhi);
    return;
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 262-336

```cpp
  auto *RdxPhi = dyn_cast<VPReductionPHIRecipe>(R);
  if (RdxPhi && RdxPhi->isOrdered())
    return;

  auto InsertPt = std::next(R->getIterator());
  for (unsigned Part = 1; Part != UF; ++Part) {
    VPRecipeBase *Copy = R->clone();
    Copy->insertBefore(*R->getParent(), InsertPt);
    addRecipeForPart(R, Copy, Part);
    if (RdxPhi) {
      // If the start value is a ReductionStartVector, use the identity value
      // (second operand) for unrolled parts. If the scaling factor is > 1,
      // create a new ReductionStartVector with the scale factor and both
      // operands set to the identity value.
      if (auto *VPI = dyn_cast<VPInstruction>(RdxPhi->getStartValue())) {
        assert(VPI->getOpcode() == VPInstruction::ReductionStartVector &&
               "unexpected start VPInstruction");
        if (Part != 1)
          continue;
        VPValue *StartV;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  // Unroll non-uniform recipes.
  auto InsertPt = std::next(R.getIterator());
  VPBasicBlock &VPBB = *R.getParent();
  for (unsigned Part = 1; Part != UF; ++Part) {
    VPRecipeBase *Copy = R.clone();
    Copy->insertBefore(VPBB, InsertPt);
    addRecipeForPart(&R, Copy, Part);

    // Phi operands are updated once all other recipes have been unrolled.
    if (isa<VPWidenPHIRecipe>(Copy))
      continue;

```
- EN: Core entities appearing here include unrollRecipeByUF, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 unrollRecipeByUF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 337-424

```cpp
    VPValue *Op;
    if (match(&R, m_VPInstruction<VPInstruction::FirstOrderRecurrenceSplice>(
                      m_VPValue(), m_VPValue(Op)))) {
      Copy->setOperand(0, getValueForPart(Op, Part - 1));
      Copy->setOperand(1, getValueForPart(Op, Part));
      continue;
    }
    if (auto *VPR = dyn_cast<VPVectorPointerRecipe>(&R)) {
      VPBuilder Builder(VPR);
      const DataLayout &DL = Plan.getDataLayout();
      Type *IndexTy = DL.getIndexType(TypeInfo.inferScalarType(VPR));
      Type *VFTy = Plan.getVF().getType();
      VPValue *VF = Builder.createScalarZExtOrTrunc(
          &Plan.getVF(), IndexTy, VFTy, DebugLoc::getUnknown());
      // VFxUF does not wrap, so VF * Part also cannot wrap.
      VPValue *VFxPart = Builder.createOverflowingOp(
          Instruction::Mul, {VF, Plan.getConstantInt(IndexTy, Part)},
          {true, true});
      Copy->setOperand(0, VPR->getOperand(0));
      Copy->addOperand(VFxPart);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    ReversePostOrderTraversal<VPBlockShallowTraversalWrapper<VPBlockBase *>>
        RPOT(VPR->getEntry());
    for (VPBlockBase *VPB : RPOT)
      unrollBlock(VPB);
    return;
  }

  // VPB is a VPBasicBlock; unroll it, i.e., unroll its recipes.
  auto *VPBB = cast<VPBasicBlock>(VPB);
  auto InsertPtForPhi = VPBB->getFirstNonPhi();
  for (VPRecipeBase &R : make_early_inc_range(*VPBB)) {
    if (ToSkip.contains(&R) || isa<VPIRInstruction>(&R))
      continue;

```
- EN: Core entities appearing here include m_VPValue, unrollBlock, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_VPValue, unrollBlock，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 425-509

```cpp
    // Add all VPValues for all parts to AnyOf, FirstActiveLaneMask and
    // ComputeReductionResult which combine all parts to compute the final
    // value.
    VPValue *Op1;
    if (match(&R, m_VPInstruction<VPInstruction::AnyOf>(m_VPValue(Op1))) ||
        match(&R, m_FirstActiveLane(m_VPValue(Op1))) ||
        match(&R, m_LastActiveLane(m_VPValue(Op1))) ||
        match(&R, m_ComputeReductionResult(m_VPValue(Op1)))) {
      addUniformForAllParts(cast<VPInstruction>(&R));
      for (unsigned Part = 1; Part != UF; ++Part)
        R.addOperand(getValueForPart(Op1, Part));
      continue;
    }
    VPValue *Op0;
    if (match(&R, m_ExtractLane(m_VPValue(Op0), m_VPValue(Op1)))) {
      addUniformForAllParts(cast<VPInstruction>(&R));
      for (unsigned Part = 1; Part != UF; ++Part)
        R.addOperand(getValueForPart(Op1, Part));
      continue;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    auto Iter = vp_depth_first_deep(Plan.getEntry());
    // Remove recipes that are redundant after unrolling.
    for (VPBasicBlock *VPBB : VPBlockUtils::blocksOnly<VPBasicBlock>(Iter)) {
      for (VPRecipeBase &R : make_early_inc_range(*VPBB)) {
        auto *VPI = dyn_cast<VPInstruction>(&R);
        if (VPI &&
            VPI->getOpcode() == VPInstruction::CanonicalIVIncrementForPart &&
            VPI->getOperand(1) == &Plan.getVF()) {
          VPI->replaceAllUsesWith(VPI->getOperand(0));
          VPI->eraseFromParent();
        }
      }
    }

```
- EN: Core entities appearing here include match, m_VPValue, unrollByUF, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match, m_VPValue, unrollByUF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 510-602

```cpp
    Type *TCTy = VPTypeAnalysis(Plan).inferScalarType(Plan.getTripCount());
    Plan.getUF().replaceAllUsesWith(Plan.getConstantInt(TCTy, UF));
  });
  if (UF == 1) {
    return;
  }

  UnrollState Unroller(Plan, UF);

  // Iterate over all blocks in the plan starting from Entry, and unroll
  // recipes inside them. This includes the vector preheader and middle blocks,
  // which may set up or post-process per-part values.
  ReversePostOrderTraversal<VPBlockShallowTraversalWrapper<VPBlockBase *>> RPOT(
      Plan.getEntry());
  for (VPBlockBase *VPB : RPOT)
    Unroller.unrollBlock(VPB);

  unsigned Part = 1;
  // Remap operands of cloned header phis to update backedge values. The header
  // phis cloned during unrolling are just after the header phi for part 0.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// Def2LaneDefs to look up scalar definitions for operands of \DefR.
static VPValue *
cloneForLane(VPlan &Plan, VPBuilder &Builder, Type *IdxTy,
             VPSingleDefRecipe *DefR, VPLane Lane,
             const DenseMap<VPValue *, SmallVector<VPValue *>> &Def2LaneDefs) {
  assert((isa<VPInstruction, VPReplicateRecipe, VPScalarIVStepsRecipe>(DefR)) &&
         "DefR must be a VPReplicateRecipe, VPInstruction or "
         "VPScalarIVStepsRecipe");
  VPValue *Op;
  if (match(DefR, m_VPInstruction<VPInstruction::Unpack>(m_VPValue(Op)))) {
    auto LaneDefs = Def2LaneDefs.find(Op);
    if (LaneDefs != Def2LaneDefs.end())
      return LaneDefs->second[Lane.getKnownLane()];

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 603-687

```cpp
    VPValue *Idx = Plan.getConstantInt(IdxTy, Lane.getKnownLane());
    return Builder.createNaryOp(Instruction::ExtractElement, {Op, Idx});
  }

  // Collect the operands at Lane, creating extracts as needed.
  SmallVector<VPValue *> NewOps;
  for (VPValue *Op : DefR->operands()) {
    // If Op is a definition that has been unrolled, directly use the clone for
    // the corresponding lane.
    auto LaneDefs = Def2LaneDefs.find(Op);
    if (LaneDefs != Def2LaneDefs.end()) {
      NewOps.push_back(LaneDefs->second[Lane.getKnownLane()]);
      continue;
    }
    if (Lane.getKind() == VPLane::Kind::ScalableLast) {
      // Look through mandatory Unpack.
      [[maybe_unused]] bool Matched =
          match(Op, m_VPInstruction<VPInstruction::Unpack>(m_VPValue(Op)));
      assert(Matched && "original op must have been Unpack");
      auto *ExtractPart =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                                       VPBlockBase *Entry,
                                                       ElementCount VF) {
  VPValue *Idx0 = Plan.getZero(IdxTy);
  VPTypeAnalysis TypeInfo(Plan);
  for (VPBlockBase *VPB : vp_depth_first_shallow(Entry)) {
    for (VPRecipeBase &OldR : make_early_inc_range(cast<VPBasicBlock>(*VPB))) {
      assert(
          !isa<VPWidenPHIRecipe>(&OldR) &&
          !match(&OldR,
                 m_CombineOr(
                     m_InsertElement(m_VPValue(), m_VPValue(), m_VPValue()),
                     m_ExtractElement(m_VPValue(), m_VPValue()))) &&
          "must not contain wide phis, inserts or extracts before conversion");

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 688-783

```cpp
      VPBuilder Builder(&OldR);
      DebugLoc OldDL = OldR.getDebugLoc();
      // For scalar VF, operands are already scalar; no extraction needed.
      if (!VF.isScalar()) {
        for (const auto &[I, Op] : enumerate(OldR.operands())) {
          // Skip operands that don't need extraction: values defined in the
          // same block (already scalar), or values that are already single
          // scalars.
          // TODO: Support isSingleScalar for VPScalarIVStepsRecipe.
          auto *DefR = Op->getDefiningRecipe();
          if ((isa_and_present<VPScalarIVStepsRecipe>(DefR) &&
               DefR->getParent() == VPB) ||
              vputils::isSingleScalar(Op))
            continue;

          // Extract lane zero from values defined outside the region.
          VPValue *Extract = Builder.createNaryOp(Instruction::ExtractElement,
                                                  {Op, Idx0}, OldDL);
          OldR.setOperand(I, Extract);
        }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        assert(vputils::onlyFirstLaneUsed(OldPhi) &&
               "VPPhis expected to have only first lane used");
        auto *BVUser = dyn_cast_or_null<VPInstruction>(OldPhi->getSingleUser());
        if (BVUser && match(BVUser, m_CombineOr(m_BuildVector(),
                                                m_BuildStructVector()))) {
          assert(BVUser->getOperand(0) == OldPhi &&
                 "Unexpected first operand of build vector user");
          BVUser->setOperand(Lane, NewPhi);
        }
      }
    }
  }
}

```
- EN: Core entities appearing here include vp_depth_first_shallow, zip_equal, m_BuildStructVector, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 vp_depth_first_shallow, zip_equal, m_BuildStructVector，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 784-864

```cpp
/// Dissolve a single replicate region by replicating its blocks for each lane
/// of \p VF. The region is disconnected, its blocks are reparented, cloned for
/// each lane, and reconnected in sequence.
static void dissolveReplicateRegion(VPRegionBlock *Region, ElementCount VF,
                                    VPlan &Plan, Type *IdxTy) {
  auto *FirstLaneEntry = cast<VPBasicBlock>(Region->getEntry());
  auto *FirstLaneExiting = cast<VPBasicBlock>(Region->getExiting());

  // Disconnect and dissolve the region.
  VPBlockBase *Predecessor = Region->getSinglePredecessor();
  assert(Predecessor && "Replicate region must have a single predecessor");
  auto *Successor = cast<VPBasicBlock>(Region->getSingleSuccessor());
  VPBlockUtils::disconnectBlocks(Predecessor, Region);
  VPBlockUtils::disconnectBlocks(Region, Successor);

  VPRegionBlock *ParentRegion = Region->getParent();
  for (VPBlockBase *VPB : vp_depth_first_shallow(FirstLaneEntry))
    VPB->setParent(ParentRegion);

  // Process the original blocks for lane 0: converting their recipes to
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    processLaneForReplicateRegion(Plan, IdxTy, Lane,
                                  cast<VPBasicBlock>(FirstLaneEntry),
                                  cast<VPBasicBlock>(CurrentLaneEntry));
    VPBlockUtils::connectBlocks(CurrentLaneExiting, NextLaneEntry);
    NextLaneEntry = CurrentLaneEntry;
  }

  // Connect Predecessor to FirstLaneEntry, and FirstLaneRegionExit to
  // NextLaneEntry which is the second lane region entry. The latter is
  // done last so that earlier clonings from FirstLaneEntry stop at
  // FirstLaneExiting.
  VPBlockUtils::connectBlocks(Predecessor, FirstLaneEntry);
  VPBlockUtils::connectBlocks(FirstLaneExiting, NextLaneEntry);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 865-950

```cpp
  // Fold BuildVector fed by scalar phis into VPWidenPHIRecipes with
  // InsertElement per lane.
  // TODO: check if this folding should be dropped.
  for (VPInstruction *BV : BuildVectors) {
    assert(BV->getNumOperands() == NumLanes &&
           "BuildVector must have one operand per lane");
    for (const auto &[Idx, Op] : enumerate(BV->operands())) {
      auto *ScalarPhi = cast<VPPhi>(Op);
      auto DL = ScalarPhi->getDebugLoc();
      auto *PredOp = cast<VPSingleDefRecipe>(ScalarPhi->getOperand(1));
      VPValue *Poison = ScalarPhi->getOperand(0);
      VPValue *PrevVal = Idx == 0 ? Poison : BV->getOperand(Idx - 1);
      auto Builder = VPBuilder::getToInsertAfter(PredOp->getDefiningRecipe());
      auto *Insert = Builder.createNaryOp(
          Instruction::InsertElement,
          {PrevVal, PredOp, Plan.getConstantInt(64, Idx)}, DL);
      Builder.setInsertPoint(ScalarPhi);
      auto *NewPhi = Builder.createWidenPhi({PrevVal, Insert}, DL);
      ScalarPhi->replaceAllUsesWith(NewPhi);
      ScalarPhi->eraseFromParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  DenseMap<VPValue *, SmallVector<VPValue *>> Def2LaneDefs;
  // The removal of current recipes being replaced by new ones needs to be
  // delayed after Def2LaneDefs is no longer in use.
  SmallVector<VPRecipeBase *> ToRemove;
  for (VPBasicBlock *VPBB : VPBBsToUnroll) {
    for (VPRecipeBase &R : make_early_inc_range(*VPBB)) {
      if (!isa<VPInstruction, VPReplicateRecipe, VPScalarIVStepsRecipe>(&R) ||
          (isa<VPReplicateRecipe>(&R) &&
           cast<VPReplicateRecipe>(&R)->isSingleScalar()) ||
          (isa<VPInstruction>(&R) &&
           !cast<VPInstruction>(&R)->doesGeneratePerAllLanes() &&
           cast<VPInstruction>(&R)->getOpcode() != VPInstruction::Unpack))
        continue;

```
- EN: Core entities appearing here include vp_depth_first_shallow, replicateByVF, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 vp_depth_first_shallow, replicateByVF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 951-994

```cpp
      auto *DefR = cast<VPSingleDefRecipe>(&R);
      VPBuilder Builder(DefR);
      if (DefR->getNumUsers() == 0) {
        // Create single-scalar version of DefR for all lanes.
        for (unsigned I = 0; I != VF.getKnownMinValue(); ++I)
          cloneForLane(Plan, Builder, IdxTy, DefR, VPLane(I), Def2LaneDefs);
        DefR->eraseFromParent();
        continue;
      }
      /// Create single-scalar version of DefR for all lanes.
      SmallVector<VPValue *> LaneDefs;
      for (unsigned I = 0; I != VF.getKnownMinValue(); ++I)
        LaneDefs.push_back(
            cloneForLane(Plan, Builder, IdxTy, DefR, VPLane(I), Def2LaneDefs));

      Def2LaneDefs[DefR] = LaneDefs;
      /// Users that only demand the first lane can use the definition for lane
      /// 0.
      DefR->replaceUsesWithIf(LaneDefs[0], [DefR](VPUser &U, unsigned) {
        return U.usesFirstLaneOnly(DefR);
      });

      // Update each build vector user that currently has DefR as its only
      // operand, to have all LaneDefs as its operands.
      for (VPUser *U : to_vector(DefR->users())) {
        auto *VPI = dyn_cast<VPInstruction>(U);
        if (!VPI || (VPI->getOpcode() != VPInstruction::BuildVector &&
                     VPI->getOpcode() != VPInstruction::BuildStructVector))
          continue;
        assert(VPI->getNumOperands() == 1 &&
               "Build(Struct)Vector must have a single operand before "
               "replicating by VF");
        VPI->setOperand(0, LaneDefs[0]);
        for (VPValue *LaneDef : drop_begin(LaneDefs))
          VPI->addOperand(LaneDef);
      }
      ToRemove.push_back(DefR);
    }
  }
  for (auto *R : reverse(ToRemove))
    R->eraseFromParent();

  replicateReplicateRegionsByVF(Plan, VF, IdxTy);
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `UnrollState, addUniformForAllParts, remapOperand, remapOperands, unrollReplicateRegionByUF, unrollRecipeByUF, m_VPValue, unrollBlock` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`UnrollState, addUniformForAllParts, remapOperand, remapOperands, unrollReplicateRegionByUF, unrollRecipeByUF, m_VPValue, unrollBlock` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/IVDescriptors.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/IVDescriptors.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/Intrinsics.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/Intrinsics.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPRecipeBuilder.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h`, `VPlanUtils.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPRecipeBuilder.h`, `VPlan.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h`, `VPlanUtils.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
