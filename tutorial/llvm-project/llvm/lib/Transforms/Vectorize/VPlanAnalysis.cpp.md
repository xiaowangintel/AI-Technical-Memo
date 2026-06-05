# VPlanAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanAnalysis.cpp` | `llvm/lib/Transforms/Vectorize/VPlanAnalysis.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements various Analyses working on VPlan within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanAnalysis 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-56

```cpp
//===- VPlanAnalysis.cpp - Various Analyses working on VPlan ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "VPlanAnalysis.h"
#include "VPlan.h"
#include "VPlanCFG.h"
#include "VPlanDominatorTree.h"
#include "VPlanHelpers.h"
#include "VPlanPatternMatch.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/PatternMatch.h"

using namespace llvm;
using namespace VPlanPatternMatch;

#define DEBUG_TYPE "vplan"

Type *VPTypeAnalysis::inferScalarTypeForRecipe(const VPBlendRecipe *R) {
  Type *ResTy = inferScalarType(R->getIncomingValue(0));
  for (unsigned I = 1, E = R->getNumIncomingValues(); I != E; ++I) {
    VPValue *Inc = R->getIncomingValue(I);
    assert(inferScalarType(Inc) == ResTy &&
           "different types inferred for different incoming values");
    CachedTypes[Inc] = ResTy;
  }
  return ResTy;
}

Type *VPTypeAnalysis::inferScalarTypeForRecipe(const VPInstruction *R) {
  // Set the result type from the first operand, check if the types for all
  // other operands match and cache them.
  auto SetResultTyFromOp = [this, R]() {
    Type *ResTy = inferScalarType(R->getOperand(0));
    unsigned NumOperands = R->getNumOperandsWithoutMask();
    for (unsigned Op = 1; Op != NumOperands; ++Op) {
      VPValue *OtherV = R->getOperand(Op);
      assert(inferScalarType(OtherV) == ResTy &&
             "different types inferred for different operands");
      CachedTypes[OtherV] = ResTy;
    }
    return ResTy;
  };

  unsigned Opcode = R->getOpcode();
  if (Instruction::isBinaryOp(Opcode) || Instruction::isUnaryOp(Opcode))
    return SetResultTyFromOp();

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 57-110

```cpp
  switch (Opcode) {
  case Instruction::PHI:
    for (VPValue *Op : R->operands()) {
      if (auto *VIR = dyn_cast<VPIRValue>(Op))
        return VIR->getType();
      if (auto *Ty = CachedTypes.lookup(Op))
        return Ty;
    }
  LLVM_FALLTHROUGH;
  case Instruction::ExtractElement:
  case Instruction::InsertElement:
  case Instruction::Freeze:
  case VPInstruction::Broadcast:
  case VPInstruction::ComputeReductionResult:
  case VPInstruction::ExitingIVValue:
  case VPInstruction::ExtractLastLane:
  case VPInstruction::ExtractPenultimateElement:
  case VPInstruction::ExtractLastPart:
  case VPInstruction::ExtractLastActive:
  case VPInstruction::PtrAdd:
  case VPInstruction::WidePtrAdd:
  case VPInstruction::ReductionStartVector:
  case VPInstruction::ResumeForEpilogue:
  case VPInstruction::Reverse:
    return inferScalarType(R->getOperand(0));
  case Instruction::Select: {
    Type *ResTy = inferScalarType(R->getOperand(1));
    VPValue *OtherV = R->getOperand(2);
    assert(inferScalarType(OtherV) == ResTy &&
           "different types inferred for different operands");
    CachedTypes[OtherV] = ResTy;
    return ResTy;
  }
  case Instruction::ICmp:
  case Instruction::FCmp:
  case VPInstruction::ActiveLaneMask:
    assert(inferScalarType(R->getOperand(0)) ==
               inferScalarType(R->getOperand(1)) &&
           "different types inferred for different operands");
    return IntegerType::get(Ctx, 1);
  case VPInstruction::ExplicitVectorLength:
    return Type::getIntNTy(Ctx, 32);
  case VPInstruction::FirstOrderRecurrenceSplice:
  case VPInstruction::Not:
  case VPInstruction::CalculateTripCountMinusVF:
  case VPInstruction::CanonicalIVIncrementForPart:
  case VPInstruction::AnyOf:
  case VPInstruction::BuildStructVector:
  case VPInstruction::BuildVector:
  case VPInstruction::Unpack:
    return SetResultTyFromOp();
  case VPInstruction::ExtractLane:
    return inferScalarType(R->getOperand(1));
  case VPInstruction::FirstActiveLane:
```
- EN: This region continues the VPlanAnalysis implementation with local helper logic centered on Opcode, Instruction, PHI, VPValue.
- CN: 这一段延续了 VPlanAnalysis 的主体实现，围绕 Opcode, Instruction, PHI, VPValue 等局部辅助逻辑展开。

### Lines 111-164

```cpp
  case VPInstruction::LastActiveLane:
    // Assume that the maximum possible number of elements in a vector fits
    // within the index type for the default address space.
    return DL.getIndexType(Ctx, 0);
  case VPInstruction::LogicalAnd:
  case VPInstruction::LogicalOr:
    assert(inferScalarType(R->getOperand(0))->isIntegerTy(1) &&
           inferScalarType(R->getOperand(1))->isIntegerTy(1) &&
           "LogicalAnd/Or operands should be bool");
    return IntegerType::get(Ctx, 1);
  case VPInstruction::MaskedCond:
    assert(inferScalarType(R->getOperand(0))->isIntegerTy(1));
    return IntegerType::get(Ctx, 1);
  case VPInstruction::BranchOnCond:
  case VPInstruction::BranchOnTwoConds:
  case VPInstruction::BranchOnCount:
  case Instruction::Store:
  case Instruction::Switch:
    return Type::getVoidTy(Ctx);
  case Instruction::Load:
    return cast<LoadInst>(R->getUnderlyingValue())->getType();
  case Instruction::Alloca:
    return cast<AllocaInst>(R->getUnderlyingValue())->getType();
  case Instruction::Call: {
    unsigned CallIdx = R->getNumOperandsWithoutMask() - 1;
    return cast<Function>(R->getOperand(CallIdx)->getLiveInIRValue())
        ->getReturnType();
  }
  case Instruction::GetElementPtr:
    return inferScalarType(R->getOperand(0));
  case Instruction::ExtractValue:
    return cast<ExtractValueInst>(R->getUnderlyingValue())->getType();
  default:
    break;
  }
  // Type inference not implemented for opcode.
  LLVM_DEBUG({
    dbgs() << "LV: Found unhandled opcode for: ";
    R->getVPSingleValue()->dump();
  });
  llvm_unreachable("Unhandled opcode!");
}

Type *VPTypeAnalysis::inferScalarTypeForRecipe(const VPWidenRecipe *R) {
  unsigned Opcode = R->getOpcode();
  if (Instruction::isBinaryOp(Opcode) || Instruction::isShift(Opcode) ||
      Instruction::isBitwiseLogicOp(Opcode)) {
    Type *ResTy = inferScalarType(R->getOperand(0));
    assert(ResTy == inferScalarType(R->getOperand(1)) &&
           "types for both operands must match for binary op");
    CachedTypes[R->getOperand(1)] = ResTy;
    return ResTy;
  }

```
- EN: Core entities appearing here include isBitwiseLogicOp, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isBitwiseLogicOp，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 165-220

```cpp
  switch (Opcode) {
  case Instruction::ICmp:
  case Instruction::FCmp:
    return IntegerType::get(Ctx, 1);
  case Instruction::FNeg:
  case Instruction::Freeze:
    return inferScalarType(R->getOperand(0));
  case Instruction::ExtractValue: {
    assert(R->getNumOperands() == 2 && "expected single level extractvalue");
    auto *StructTy = cast<StructType>(inferScalarType(R->getOperand(0)));
    return StructTy->getTypeAtIndex(
        cast<VPConstantInt>(R->getOperand(1))->getZExtValue());
  }
  case Instruction::Select: {
    Type *ResTy = inferScalarType(R->getOperand(1));
    VPValue *OtherV = R->getOperand(2);
    assert(inferScalarType(OtherV) == ResTy &&
           "different types inferred for different operands");
    CachedTypes[OtherV] = ResTy;
    return ResTy;
  }
  default:
    break;
  }

  // Type inference not implemented for opcode.
  LLVM_DEBUG({
    dbgs() << "LV: Found unhandled opcode for: ";
    R->getVPSingleValue()->dump();
  });
  llvm_unreachable("Unhandled opcode!");
}

Type *VPTypeAnalysis::inferScalarTypeForRecipe(const VPWidenCallRecipe *R) {
  auto &CI = *cast<CallInst>(R->getUnderlyingInstr());
  return CI.getType();
}

Type *VPTypeAnalysis::inferScalarTypeForRecipe(const VPWidenMemoryRecipe *R) {
  assert((isa<VPWidenLoadRecipe, VPWidenLoadEVLRecipe>(R->getAsRecipe())) &&
         "Store recipes should not define any values");
  return cast<LoadInst>(&R->getIngredient())->getType();
}

Type *VPTypeAnalysis::inferScalarTypeForRecipe(const VPReplicateRecipe *R) {
  unsigned Opcode = R->getUnderlyingInstr()->getOpcode();

  if (Instruction::isBinaryOp(Opcode) || Instruction::isShift(Opcode) ||
      Instruction::isBitwiseLogicOp(Opcode)) {
    Type *ResTy = inferScalarType(R->getOperand(0));
    assert(ResTy == inferScalarType(R->getOperand(1)) &&
           "inferred types for operands of binary op don't match");
    CachedTypes[R->getOperand(1)] = ResTy;
    return ResTy;
  }

```
- EN: Core entities appearing here include isBitwiseLogicOp, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isBitwiseLogicOp，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 221-277

```cpp
  if (Instruction::isCast(Opcode))
    return R->getUnderlyingInstr()->getType();

  switch (Opcode) {
  case Instruction::Call: {
    unsigned CallIdx = R->getNumOperands() - (R->isPredicated() ? 2 : 1);
    return cast<Function>(R->getOperand(CallIdx)->getLiveInIRValue())
        ->getReturnType();
  }
  case Instruction::Select: {
    Type *ResTy = inferScalarType(R->getOperand(1));
    assert(ResTy == inferScalarType(R->getOperand(2)) &&
           "inferred types for operands of select op don't match");
    CachedTypes[R->getOperand(2)] = ResTy;
    return ResTy;
  }
  case Instruction::ICmp:
  case Instruction::FCmp:
    return IntegerType::get(Ctx, 1);
  case Instruction::Alloca:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

Type *VPTypeAnalysis::inferScalarType(const VPValue *V) {
  if (Type *CachedTy = CachedTypes.lookup(V))
    return CachedTy;

  if (auto *IRV = dyn_cast<VPIRValue>(V))
    return IRV->getType();

  if (auto *SymbolicV = dyn_cast<VPSymbolicValue>(V))
    return SymbolicV->getType();

  if (auto *RegionV = dyn_cast<VPRegionValue>(V))
    return RegionV->getType();

```
- EN: This region continues the VPlanAnalysis implementation with local helper logic centered on Instruction, Opcode, Call, CallIdx.
- CN: 这一段延续了 VPlanAnalysis 的主体实现，围绕 Instruction, Opcode, Call, CallIdx 等局部辅助逻辑展开。

### Lines 278-337

```cpp
  Type *ResultTy =
      TypeSwitch<const VPRecipeBase *, Type *>(V->getDefiningRecipe())
          .Case<VPActiveLaneMaskPHIRecipe, VPFirstOrderRecurrencePHIRecipe,
                VPReductionPHIRecipe, VPWidenPointerInductionRecipe,
                VPCurrentIterationPHIRecipe>([this](const auto *R) {
            // Handle header phi recipes, except VPWidenIntOrFpInduction
            // which needs special handling due it being possibly truncated.
            // TODO: consider inferring/caching type of siblings, e.g.,
            // backedge value, here and in cases below.
            return inferScalarType(R->getStartValue());
          })
          .Case<VPWidenIntOrFpInductionRecipe, VPDerivedIVRecipe>(
              [](const auto *R) { return R->getScalarType(); })
          .Case<VPReductionRecipe, VPPredInstPHIRecipe, VPWidenPHIRecipe,
                VPScalarIVStepsRecipe, VPWidenGEPRecipe, VPVectorPointerRecipe,
                VPVectorEndPointerRecipe, VPWidenCanonicalIVRecipe>(
              [this](const VPRecipeBase *R) {
                return inferScalarType(R->getOperand(0));
              })
          // VPInstructionWithType must be handled before VPInstruction.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    VPlan &Plan, DenseSet<VPRecipeBase *> &EphRecipes) {
  // First, collect seed recipes which are operands of assumes.
  SmallVector<VPRecipeBase *> Worklist;
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksOnly<VPBasicBlock>(
           vp_depth_first_deep(Plan.getVectorLoopRegion()->getEntry()))) {
    for (VPRecipeBase &R : *VPBB) {
      auto *RepR = dyn_cast<VPReplicateRecipe>(&R);
      if (!RepR || !match(RepR, m_Intrinsic<Intrinsic::assume>()))
        continue;
      Worklist.push_back(RepR);
      EphRecipes.insert(RepR);
    }
  }

```
- EN: Core entities appearing here include vp_depth_first_deep, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 vp_depth_first_deep，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 338-391

```cpp
  // Process operands of candidates in worklist and add them to the set of
  // ephemeral recipes, if they don't have side-effects and are only used by
  // other ephemeral recipes.
  while (!Worklist.empty()) {
    VPRecipeBase *Cur = Worklist.pop_back_val();
    for (VPValue *Op : Cur->operands()) {
      auto *OpR = Op->getDefiningRecipe();
      if (!OpR || OpR->mayHaveSideEffects() || EphRecipes.contains(OpR))
        continue;
      if (any_of(Op->users(), [EphRecipes](VPUser *U) {
            auto *UR = dyn_cast<VPRecipeBase>(U);
            return !UR || !EphRecipes.contains(UR);
          }))
        continue;
      EphRecipes.insert(OpR);
      Worklist.push_back(OpR);
    }
  }
}

template void DomTreeBuilder::Calculate<DominatorTreeBase<VPBlockBase, false>>(
    DominatorTreeBase<VPBlockBase, false> &DT);

bool VPDominatorTree::properlyDominates(const VPRecipeBase *A,
                                        const VPRecipeBase *B) {
  if (A == B)
    return false;

  auto LocalComesBefore = [](const VPRecipeBase *A, const VPRecipeBase *B) {
    for (auto &R : *A->getParent()) {
      if (&R == A)
        return true;
      if (&R == B)
        return false;
    }
    llvm_unreachable("recipe not found");
  };
  const VPBlockBase *ParentA = A->getParent();
  const VPBlockBase *ParentB = B->getParent();
  if (ParentA == ParentB)
    return LocalComesBefore(A, B);

  return Base::properlyDominates(ParentA, ParentB);
}

InstructionCost
VPRegisterUsage::spillCost(const TargetTransformInfo &TTI,
                           TargetTransformInfo::TargetCostKind CostKind,
                           unsigned OverrideMaxNumRegs) const {
  InstructionCost Cost;
  for (const auto &[RegClass, MaxUsers] : MaxLocalUsers) {
    unsigned AvailableRegs = OverrideMaxNumRegs > 0
                                 ? OverrideMaxNumRegs
                                 : TTI.getNumberOfRegisters(RegClass);
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 392-443

```cpp
    if (MaxUsers > AvailableRegs) {
      // Assume that for each register used past what's available we get one
      // spill and reload.
      unsigned Spills = MaxUsers - AvailableRegs;
      InstructionCost SpillCost =
          TTI.getRegisterClassSpillCost(RegClass, CostKind) +
          TTI.getRegisterClassReloadCost(RegClass, CostKind);
      InstructionCost TotalCost = Spills * SpillCost;
      LLVM_DEBUG(dbgs() << "LV(REG): Cost of " << TotalCost << " from "
                        << Spills << " spills of "
                        << TTI.getRegisterClassName(RegClass) << "\n");
      Cost += TotalCost;
    }
  }
  return Cost;
}

SmallVector<VPRegisterUsage, 8> llvm::calculateRegisterUsageForPlan(
    VPlan &Plan, ArrayRef<ElementCount> VFs, const TargetTransformInfo &TTI,
    const SmallPtrSetImpl<const Value *> &ValuesToIgnore) {
  // Each 'key' in the map opens a new interval. The values
  // of the map are the index of the 'last seen' usage of the
  // VPValue that is the key.
  using IntervalMap = SmallDenseMap<VPValue *, unsigned, 16>;

  // Maps indices to recipes.
  SmallVector<VPRecipeBase *, 64> Idx2Recipe;
  // Marks the end of each interval.
  IntervalMap EndPoint;
  // Saves the list of VPValues that are used in the loop.
  SmallPtrSet<VPValue *, 8> Ends;
  // Saves the list of values that are used in the loop but are defined outside
  // the loop (not including non-recipe values such as arguments and
  // constants).
  SmallSetVector<VPValue *, 8> LoopInvariants;
  if (Plan.getVectorTripCount().getNumUsers() > 0)
    LoopInvariants.insert(&Plan.getVectorTripCount());

  // We scan the loop in a topological order in order and assign a number to
  // each recipe. We use RPO to ensure that defs are met before their users. We
  // assume that each recipe that has in-loop users starts an interval. We
  // record every time that an in-loop value is used, so we have a list of the
  // first occurences of each recipe and last occurrence of each VPValue.
  VPRegionBlock *LoopRegion = Plan.getVectorLoopRegion();
  ReversePostOrderTraversal<VPBlockDeepTraversalWrapper<VPBlockBase *>> RPOT(
      LoopRegion);
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksOnly<VPBasicBlock>(RPOT)) {
    if (!VPBB->getParent())
      break;
    for (VPRecipeBase &R : *VPBB) {
      Idx2Recipe.push_back(&R);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 444-500

```cpp
      // Save the end location of each USE.
      for (VPValue *U : R.operands()) {
        if (isa<VPRecipeValue>(U)) {
          // Overwrite previous end points.
          EndPoint[U] = Idx2Recipe.size();
          Ends.insert(U);
        } else if (auto *IRV = dyn_cast<VPIRValue>(U)) {
          // Ignore non-recipe values such as arguments, constants, etc.
          // FIXME: Might need some motivation why these values are ignored. If
          // for example an argument is used inside the loop it will increase
          // the register pressure (so shouldn't we add it to LoopInvariants).
          if (!isa<Instruction>(IRV->getValue()))
            continue;
          // This recipe is outside the loop, record it and continue.
          LoopInvariants.insert(U);
        }
        // Other types of VPValue are currently not tracked.
      }
    }
    if (VPBB == LoopRegion->getExiting()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  LLVM_DEBUG(dbgs() << "LV(REG): Calculating max register usage:\n");

  VPTypeAnalysis TypeInfo(Plan);

  const auto &TTICapture = TTI;
  auto GetRegUsage = [&TTICapture](Type *Ty, ElementCount VF) -> unsigned {
    if (Ty->isTokenTy() || !VectorType::isValidElementType(Ty) ||
        (VF.isScalable() &&
         !TTICapture.isElementTypeLegalForScalableVector(Ty)))
      return 0;
    return TTICapture.getRegUsageForType(VectorType::get(Ty, VF));
  };

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 501-552

```cpp
  VPValue *CanIV = LoopRegion->getCanonicalIV();
  // Note: canonical IVs are retained even if they have no users.
  if (CanIV->getNumUsers() != 0)
    OpenIntervals.insert(CanIV);

  // We scan the instructions linearly and record each time that a new interval
  // starts, by placing it in a set. If we find this value in TransposEnds then
  // we remove it from the set. The max register usage is the maximum register
  // usage of the recipes of the set.
  for (unsigned int Idx = 0, Sz = Idx2Recipe.size(); Idx < Sz; ++Idx) {
    VPRecipeBase *R = Idx2Recipe[Idx];

    // Remove all of the VPValues that end at this location.
    VPValueList &List = TransposeEnds[Idx];
    for (VPValue *ToRemove : List)
      OpenIntervals.erase(ToRemove);

    // Ignore recipes that are never used within the loop and do not have side
    // effects.
    if (none_of(R->definedValues(),
                [&Ends](VPValue *Def) { return Ends.count(Def); }) &&
        !R->mayHaveSideEffects())
      continue;

    // Skip recipes for ignored values.
    // TODO: Should mark recipes for ephemeral values that cannot be removed
    // explictly in VPlan.
    if (isa<VPSingleDefRecipe>(R) &&
        ValuesToIgnore.contains(
            cast<VPSingleDefRecipe>(R)->getUnderlyingValue()))
      continue;

    // For each VF find the maximum usage of registers.
    for (unsigned J = 0, E = VFs.size(); J < E; ++J) {
      // Count the number of registers used, per register class, given all open
      // intervals.
      // Note that elements in this SmallMapVector will be default constructed
      // as 0. So we can use "RegUsage[ClassID] += n" in the code below even if
      // there is no previous entry for ClassID.
      SmallMapVector<unsigned, unsigned, 4> RegUsage;

      for (auto *VPV : OpenIntervals) {
        // Skip artificial values or values that weren't present in the original
        // loop.
        // TODO: Remove skipping values that weren't present in the original
        // loop after removing the legacy
        // LoopVectorizationCostModel::calculateRegisterUsage
        if (isa<VPVectorPointerRecipe, VPVectorEndPointerRecipe,
                VPBranchOnMaskRecipe>(VPV) ||
            match(VPV, m_ExtractLastPart(m_VPValue())))
          continue;

```
- EN: This region continues the VPlanAnalysis implementation with local helper logic centered on VPValue, CanIV, LoopRegion, Note.
- CN: 这一段延续了 VPlanAnalysis 的主体实现，围绕 VPValue, CanIV, LoopRegion, Note 等局部辅助逻辑展开。

### Lines 553-608

```cpp
        if (VFs[J].isScalar() ||
            isa<VPRegionValue, VPReplicateRecipe, VPDerivedIVRecipe,
                VPCurrentIterationPHIRecipe, VPScalarIVStepsRecipe>(VPV) ||
            (isa<VPInstruction>(VPV) && vputils::onlyScalarValuesUsed(VPV)) ||
            (isa<VPReductionPHIRecipe>(VPV) &&
             (cast<VPReductionPHIRecipe>(VPV))->isInLoop())) {
          unsigned ClassID =
              TTI.getRegisterClassForType(false, TypeInfo.inferScalarType(VPV));
          // FIXME: The target might use more than one register for the type
          // even in the scalar case.
          RegUsage[ClassID] += 1;
        } else {
          // The output from scaled phis and scaled reductions actually has
          // fewer lanes than the VF.
          unsigned ScaleFactor =
              vputils::getVFScaleFactor(VPV->getDefiningRecipe());
          ElementCount VF = VFs[J];
          if (ScaleFactor > 1) {
            VF = VFs[J].divideCoefficientBy(ScaleFactor);
            LLVM_DEBUG(dbgs() << "LV(REG): Scaled down VF from " << VFs[J]
                              << " to " << VF << " for " << *R << "\n";);
          }

          Type *ScalarTy = TypeInfo.inferScalarType(VPV);
          unsigned ClassID = TTI.getRegisterClassForType(true, ScalarTy);
          RegUsage[ClassID] += GetRegUsage(ScalarTy, VF);
        }
      }

      for (const auto &Pair : RegUsage) {
        auto &Entry = MaxUsages[J][Pair.first];
        Entry = std::max(Entry, Pair.second);
      }
    }

    LLVM_DEBUG(dbgs() << "LV(REG): At #" << Idx << " Interval # "
                      << OpenIntervals.size() << '\n');

    // Add used VPValues defined by the current recipe to the list of open
    // intervals.
    for (VPValue *DefV : R->definedValues())
      if (Ends.contains(DefV))
        OpenIntervals.insert(DefV);
  }

  // We also search for instructions that are defined outside the loop, but are
  // used inside the loop. We need this number separately from the max-interval
  // usage number because when we unroll, loop-invariant values do not take
  // more register.
  VPRegisterUsage RU;
  for (unsigned Idx = 0, End = VFs.size(); Idx < End; ++Idx) {
    // Note that elements in this SmallMapVector will be default constructed
    // as 0. So we can use "Invariant[ClassID] += n" in the code below even if
    // there is no previous entry for ClassID.
    SmallMapVector<unsigned, unsigned, 4> Invariant;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 609-644

```cpp
    for (auto *In : LoopInvariants) {
      // FIXME: The target might use more than one register for the type
      // even in the scalar case.
      bool IsScalar = vputils::onlyScalarValuesUsed(In);

      ElementCount VF = IsScalar ? ElementCount::getFixed(1) : VFs[Idx];
      unsigned ClassID = TTI.getRegisterClassForType(
          VF.isVector(), TypeInfo.inferScalarType(In));
      Invariant[ClassID] += GetRegUsage(TypeInfo.inferScalarType(In), VF);
    }

    LLVM_DEBUG({
      dbgs() << "LV(REG): VF = " << VFs[Idx] << '\n';
      dbgs() << "LV(REG): Found max usage: " << MaxUsages[Idx].size()
             << " item\n";
      for (const auto &pair : MaxUsages[Idx]) {
        dbgs() << "LV(REG): RegisterClass: "
               << TTI.getRegisterClassName(pair.first) << ", " << pair.second
               << " registers\n";
      }
      dbgs() << "LV(REG): Found invariant usage: " << Invariant.size()
             << " item\n";
      for (const auto &pair : Invariant) {
        dbgs() << "LV(REG): RegisterClass: "
               << TTI.getRegisterClassName(pair.first) << ", " << pair.second
               << " registers\n";
      }
    });

    RU.LoopInvariantRegs = Invariant;
    RU.MaxLocalUsers = MaxUsages[Idx];
    RUs[Idx] = RU;
  }

  return RUs;
}
```
- EN: This region continues the VPlanAnalysis implementation with local helper logic centered on LoopInvariants, FIXME, The, IsScalar.
- CN: 这一段延续了 VPlanAnalysis 的主体实现，围绕 LoopInvariants, FIXME, The, IsScalar 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `isBitwiseLogicOp, vp_depth_first_deep` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`isBitwiseLogicOp, vp_depth_first_deep` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Instruction.h`, `llvm/IR/PatternMatch.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Instruction.h`, `llvm/IR/PatternMatch.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/TypeSwitch.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/PostOrderIterator.h`, `llvm/ADT/TypeSwitch.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlanAnalysis.h`, `VPlan.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlanAnalysis.h`, `VPlan.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
