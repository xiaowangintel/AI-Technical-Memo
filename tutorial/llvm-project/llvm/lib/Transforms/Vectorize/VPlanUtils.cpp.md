# VPlanUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanUtils.cpp` | `llvm/lib/Transforms/Vectorize/VPlanUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vPlan-related utilities within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-73

```cpp
//===- VPlanUtils.cpp - VPlan-related utilities ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "VPlanUtils.h"
#include "VPlanAnalysis.h"
#include "VPlanCFG.h"
#include "VPlanDominatorTree.h"
#include "VPlanPatternMatch.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Analysis/MemoryLocation.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"
#include "llvm/Analysis/ScalarEvolutionPatternMatch.h"

using namespace llvm;
using namespace llvm::VPlanPatternMatch;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
            cast<VPWidenIntOrFpInductionRecipe>(A)->isCanonical());
  };

  VPValue *A, *B;

  auto m_CanonicalScalarIVSteps = m_ScalarIVSteps(
      m_CombineOr(m_CanonicalIV(),
                  m_DerivedIV(m_ZeroInt(), m_CanonicalIV(), m_One())),
      m_One(), m_Specific(&Plan.getVF()));

  if (match(V, m_ActiveLaneMask(m_VPValue(A), m_VPValue(B), m_One())))
    return B == Plan.getTripCount() &&
           (match(A, m_CanonicalScalarIVSteps) || IsWideCanonicalIV(A));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include onlyFirstLaneUsed, onlyFirstPartUsed, onlyScalarValuesUsed, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 onlyFirstLaneUsed, onlyFirstPartUsed, onlyScalarValuesUsed，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 74-141

```cpp
  // For scalar plans, the header mask uses the scalar steps.
  if (match(V, m_ICmp(m_CanonicalScalarIVSteps,
                      m_Specific(Plan.getBackedgeTakenCount())))) {
    assert(Plan.hasScalarVFOnly() &&
           "Non-scalar VF using scalar IV steps for header mask?");
    return true;
  }

  auto MaskMatch = m_ICmp(m_VPValue(A), m_VPValue(B));
  return (match(V, m_CombineOr(MaskMatch, m_Reverse(MaskMatch)))) &&
         IsWideCanonicalIV(A) && B == Plan.getBackedgeTakenCount();
}

/// Returns true if \p R propagates poison from any operand to its result.
static bool propagatesPoisonFromRecipeOp(const VPRecipeBase *R) {
  return TypeSwitch<const VPRecipeBase *, bool>(R)
      .Case<VPWidenGEPRecipe, VPWidenCastRecipe>(
          [](const VPRecipeBase *) { return true; })
      .Case([](const VPReplicateRecipe *Rep) {
        // GEP and casts propagate poison from all operands.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // Check if poison propagates through this recipe to any of its users.
      auto *R = cast<VPRecipeBase>(U);
      for (const VPValue *Op : R->operands()) {
        if (Op == Current && propagatesPoisonFromRecipeOp(R)) {
          Worklist.push_back(R->getVPSingleValue());
          break;
        }
      }
    }
  }

  return false;
}

```
- EN: Core entities appearing here include m_Specific, propagatesPoisonFromRecipeOp, poisonGuaranteesUB, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 m_Specific, propagatesPoisonFromRecipeOp, poisonGuaranteesUB，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 142-212

```cpp
GEPNoWrapFlags vputils::getGEPFlagsForPtr(VPValue *Ptr) {
  // Like IR stripPointerCasts, look through GEPs with all-zero indices and
  // casts to find a root GEP VPInstruction.
  while (auto *PtrVPI = dyn_cast<VPInstruction>(Ptr)) {
    unsigned Opcode = PtrVPI->getOpcode();
    if (Opcode == Instruction::GetElementPtr) {
      if (any_of(drop_begin(PtrVPI->operands()),
                 [](VPValue *Op) { return !match(Op, m_ZeroInt()); }))
        return PtrVPI->getGEPNoWrapFlags();
      Ptr = PtrVPI->getOperand(0);
      continue;
    }
    if (Opcode != Instruction::BitCast && Opcode != Instruction::AddrSpaceCast)
      break;
    Ptr = PtrVPI->getOperand(0);
  }
  return GEPNoWrapFlags::none();
}

const SCEV *vputils::getSCEVExprForVPValue(const VPValue *V,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    });
  if (match(V, m_Sub(m_VPValue(LHSVal), m_VPValue(RHSVal))))
    return CreateSCEV({LHSVal, RHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getMinusSCEV(Ops[0], Ops[1], SCEV::FlagAnyWrap, 0);
    });
  if (match(V, m_Not(m_VPValue(LHSVal)))) {
    // not X = xor X, -1 = -1 - X
    return CreateSCEV({LHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getMinusSCEV(SE.getMinusOne(Ops[0]->getType()), Ops[0]);
    });
  }
  if (match(V, m_Mul(m_VPValue(LHSVal), m_VPValue(RHSVal))))
    return CreateSCEV({LHSVal, RHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getMulExpr(Ops[0], Ops[1], SCEV::FlagAnyWrap, 0);
```
- EN: Core entities appearing here include getGEPFlagsForPtr, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getGEPFlagsForPtr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 213-288

```cpp
    });
  if (match(V,
            m_Binary<Instruction::UDiv>(m_VPValue(LHSVal), m_VPValue(RHSVal))))
    return CreateSCEV({LHSVal, RHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getUDivExpr(Ops[0], Ops[1]);
    });
  // Handle AND with constant mask: x & (2^n - 1) can be represented as x % 2^n.
  const APInt *Mask;
  if (match(V, m_c_BinaryAnd(m_VPValue(LHSVal), m_APInt(Mask))) &&
      (*Mask + 1).isPowerOf2())
    return CreateSCEV({LHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getURemExpr(Ops[0], SE.getConstant(*Mask + 1));
    });
  if (match(V, m_Trunc(m_VPValue(LHSVal)))) {
    const VPlan *Plan = V->getDefiningRecipe()->getParent()->getPlan();
    Type *DestTy = VPTypeAnalysis(*Plan).inferScalarType(V);
    return CreateSCEV({LHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getTruncateExpr(Ops[0], DestTy);
    });
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    });
  if (match(V,
            m_Intrinsic<Intrinsic::smin>(m_VPValue(LHSVal), m_VPValue(RHSVal))))
    return CreateSCEV({LHSVal, RHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getSMinExpr(Ops[0], Ops[1]);
    });
  if (match(V, m_Intrinsic<Intrinsic::abs>(m_VPValue(LHSVal), m_VPValue())))
    return CreateSCEV({LHSVal}, [&](ArrayRef<SCEVUse> Ops) {
      // is_int_min_poison is local to this intrinsic: poison on INT_MIN is
      // not proof that the input is never INT_MIN, nor that poison reaches
      // UB. Do not translate it to SCEV's global IsNSW flag.
      return SE.getAbsExpr(Ops[0], /*IsNSW=*/false);
    });

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 289-362

```cpp
  ArrayRef<VPValue *> Ops;
  Type *SourceElementType;
  if (match(V, m_GetElementPtr(SourceElementType, Ops))) {
    const SCEV *GEPExpr = CreateSCEV(Ops, [&](ArrayRef<SCEVUse> Ops) {
      return SE.getGEPExpr(Ops.front(), Ops.drop_front(), SourceElementType);
    });
    return PSE.getPredicatedSCEV(GEPExpr);
  }

  // TODO: Support constructing SCEVs for more recipes as needed.
  const VPRecipeBase *DefR = V->getDefiningRecipe();
  const SCEV *Expr =
      TypeSwitch<const VPRecipeBase *, const SCEV *>(DefR)
          .Case([](const VPExpandSCEVRecipe *R) { return R->getSCEV(); })
          .Case([&SE, &PSE, L](const VPWidenIntOrFpInductionRecipe *R) {
            const SCEV *Step = getSCEVExprForVPValue(R->getStepValue(), PSE, L);
            if (!L || isa<SCEVCouldNotCompute>(Step))
              return SE.getCouldNotCompute();
            const SCEV *Start =
                getSCEVExprForVPValue(R->getStartValue(), PSE, L);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

bool vputils::isAddressSCEVForCost(const SCEV *Addr, ScalarEvolution &SE,
                                   const Loop *L) {
  // If address is an SCEVAddExpr, we require that all operands must be either
  // be invariant or a (possibly sign-extend) affine AddRec.
  if (auto *PtrAdd = dyn_cast<SCEVAddExpr>(Addr)) {
    return all_of(PtrAdd->operands(), [&SE, L](const SCEV *Op) {
      return SE.isLoopInvariant(Op, L) ||
             match(Op, m_scev_SExt(m_scev_AffineAddRec(m_SCEV(), m_SCEV()))) ||
             match(Op, m_scev_AffineAddRec(m_SCEV(), m_SCEV()));
    });
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 363-442

```cpp
  // Otherwise, check if address is loop invariant or an affine add recurrence.
  return SE.isLoopInvariant(Addr, L) ||
         match(Addr, m_scev_AffineAddRec(m_SCEV(), m_SCEV()));
}

/// Returns true if \p Opcode preserves uniformity, i.e., if all operands are
/// uniform, the result will also be uniform.
static bool preservesUniformity(unsigned Opcode) {
  if (Instruction::isBinaryOp(Opcode) || Instruction::isCast(Opcode))
    return true;
  switch (Opcode) {
  case Instruction::Freeze:
  case Instruction::GetElementPtr:
  case Instruction::ICmp:
  case Instruction::FCmp:
  case Instruction::Select:
  case VPInstruction::Not:
  case VPInstruction::Broadcast:
  case VPInstruction::MaskedCond:
  case VPInstruction::PtrAdd:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return true;

  const VPRecipeBase *R = V->getDefiningRecipe();
  const VPBasicBlock *VPBB = R ? R->getParent() : nullptr;
  const VPlan *Plan = VPBB ? VPBB->getPlan() : nullptr;
  if (VPBB) {
    if ((VPBB == Plan->getVectorPreheader() || VPBB == Plan->getEntry())) {
      if (match(V->getDefiningRecipe(),
                m_VPInstruction<VPInstruction::CanonicalIVIncrementForPart>()))
        return false;
      return all_of(R->operands(), isUniformAcrossVFsAndUFs);
    }
  }

```
- EN: Core entities appearing here include preservesUniformity, isSingleScalar, isUniformAcrossVFsAndUFs, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 preservesUniformity, isSingleScalar, isUniformAcrossVFsAndUFs，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 443-511

```cpp
  return TypeSwitch<const VPRecipeBase *, bool>(R)
      .Case([](const VPDerivedIVRecipe *R) { return true; })
      .Case([](const VPReplicateRecipe *R) {
        // Be conservative about side-effects, except for the
        // known-side-effecting assumes and stores, which we know will be
        // uniform.
        return R->isSingleScalar() &&
               (!R->mayHaveSideEffects() ||
                isa<AssumeInst, StoreInst>(R->getUnderlyingInstr())) &&
               all_of(R->operands(), isUniformAcrossVFsAndUFs);
      })
      .Case([](const VPWidenRecipe *R) {
        return preservesUniformity(R->getOpcode()) &&
               all_of(R->operands(), isUniformAcrossVFsAndUFs);
      })
      .Case([](const VPInstruction *VPI) {
        return (VPI->isSingleScalar() || VPI->isVectorToScalar() ||
                preservesUniformity(VPI->getOpcode())) &&
               all_of(VPI->operands(), isUniformAcrossVFsAndUFs);
      })
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Assumes don't alias anything or throw; as long as they're guaranteed to
  // execute, they're safe to hoist. They should however not be sunk, as it
  // would destroy information.
  if (match(&R, m_Intrinsic<Intrinsic::assume>()))
    return Sinking;
  // TODO: Relax checks in the future, e.g. we could also hoist reads, if their
  // memory location is not modified in the vector loop.
  if (R.mayHaveSideEffects() || R.mayReadFromMemory() || R.isPhi())
    return true;
  // Allocas cannot be hoisted.
  auto *RepR = dyn_cast<VPReplicateRecipe>(&R);
  return RepR && RepR->getOpcode() == Instruction::Alloca;
}

```
- EN: Core entities appearing here include getVFScaleFactor, cannotHoistOrSinkRecipe, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getVFScaleFactor, cannotHoistOrSinkRecipe，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 512-587

```cpp
std::optional<VPValue *>
vputils::getRecipesForUncountableExit(SmallVectorImpl<VPInstruction *> &Recipes,
                                      SmallVectorImpl<VPInstruction *> &GEPs,
                                      VPBasicBlock *LatchVPBB) {
  // Given a plain CFG VPlan loop with countable latch exiting block
  // \p LatchVPBB, we're looking to match the recipes contributing to the
  // uncountable exit condition comparison (here, vp<%4>) back to either
  // live-ins or the address nodes for the load used as part of the uncountable
  // exit comparison so that we can either move them within the loop, or copy
  // them to the preheader depending on the chosen method for dealing with
  // stores in uncountable exit loops.
  //
  // Currently, the address of the load is restricted to a GEP with 2 operands
  // and a live-in base address. This constraint may be relaxed later.
  //
  // VPlan ' for UF>=1' {
  // Live-in vp<%0> = VF * UF
  // Live-in vp<%1> = vector-trip-count
  // Live-in ir<20> = original trip-count
  //
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Worklist.push_back(UncountableCondition);
  while (!Worklist.empty()) {
    VPValue *V = Worklist.pop_back_val();

    // Any value defined outside the loop does not need to be copied.
    if (V->isDefinedOutsideLoopRegions())
      continue;

    // FIXME: Remove the single user restriction; it's here because we're
    //        starting with the simplest set of loops we can, and multiple
    //        users means needing to add PHI nodes in the transform.
    if (V->getNumUsers() > 1)
      return std::nullopt;

```
- EN: This region continues the VPlanUtils implementation with local helper logic centered on VPValue, SmallVectorImpl, VPInstruction, Recipes.
- CN: 这一段延续了 VPlanUtils 的主体实现，围绕 VPValue, SmallVectorImpl, VPInstruction, Recipes 等局部辅助逻辑展开。

### Lines 588-659

```cpp
    VPValue *Op1, *Op2;
    // Walk back through recipes until we find at least one load from memory.
    if (match(V, m_ICmp(m_VPValue(Op1), m_VPValue(Op2)))) {
      Worklist.push_back(Op1);
      Worklist.push_back(Op2);
      Recipes.push_back(cast<VPInstruction>(V->getDefiningRecipe()));
    } else if (match(V, m_VPInstruction<Instruction::Load>(m_VPValue(Op1)))) {
      VPRecipeBase *GepR = Op1->getDefiningRecipe();
      // Only matching base + single offset term for now.
      if (GepR->getNumOperands() != 2)
        return std::nullopt;
      // Matching a GEP with a loop-invariant base ptr.
      if (!match(GepR, m_VPInstruction<Instruction::GetElementPtr>(
                           m_LiveIn(), m_VPValue())))
        return std::nullopt;
      Recipes.push_back(cast<VPInstruction>(V->getDefiningRecipe()));
      Recipes.push_back(cast<VPInstruction>(GepR));
      GEPs.push_back(cast<VPInstruction>(GepR));
    } else if (match(V, m_VPInstruction<VPInstruction::MaskedCond>(
                            m_VPValue(Op1)))) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (VPUser *U : Wide->users()) {
      auto *VPI = dyn_cast<VPInstruction>(U);
      if (!VPI || !vputils::isHeaderMask(VPI, Plan))
        continue;

      assert(VPI->getOperand(0) == Wide &&
             "WidenCanonicalIV must be the first operand of the compare");
      assert(!HeaderMask && "Multiple header masks found?");
      HeaderMask = VPI;
    }
  }
  return HeaderMask;
}

```
- EN: Core entities appearing here include m_VPValue, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_VPValue，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 660-733

```cpp
SmallVector<VPBasicBlock *>
VPBlockUtils::blocksInSingleSuccessorChainBetween(VPBasicBlock *FirstBB,
                                                  VPBasicBlock *LastBB) {
  assert(FirstBB->getParent() == LastBB->getParent() &&
         "FirstBB and LastBB from different regions");
#ifndef NDEBUG
  bool InSingleSuccChain = false;
  for (VPBlockBase *Succ = FirstBB; Succ; Succ = Succ->getSingleSuccessor())
    InSingleSuccChain |= (Succ == LastBB);
  assert(InSingleSuccChain &&
         "LastBB unreachable from FirstBB in single-successor chain");
#endif
  auto Blocks = to_vector(
      VPBlockUtils::blocksOnly<VPBasicBlock>(vp_depth_first_deep(FirstBB)));
  auto *LastIt = find(Blocks, LastBB);
  assert(LastIt != Blocks.end() &&
         "LastBB unreachable from FirstBB in depth-first traversal");
  Blocks.erase(std::next(LastIt), Blocks.end());
  return Blocks;
}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

VPInstruction *vputils::findCanonicalIVIncrement(VPlan &Plan) {
  VPRegionBlock *LoopRegion = Plan.getVectorLoopRegion();
  VPRegionValue *CanIV = LoopRegion->getCanonicalIV();
  assert(CanIV && "Expected loop region to have a canonical IV");

  VPSymbolicValue &VFxUF = Plan.getVFxUF();

  // Check if \p Step matches the expected increment step, accounting for
  // materialization of VFxUF and UF.
  auto IsIncrementStep = [&](VPValue *Step) -> bool {
    if (!VFxUF.isMaterialized())
      return Step == &VFxUF;

```
- EN: Core entities appearing here include getMemoryLocation, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 getMemoryLocation，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 734-810

```cpp
    VPSymbolicValue &UF = Plan.getUF();
    if (!UF.isMaterialized())
      return Step == &UF;

    unsigned ConcreteUF = Plan.getConcreteUF();
    // Fixed VF: step is just the concrete UF.
    if (match(Step, m_SpecificInt(ConcreteUF)))
      return true;

    // Scalable VF: step involves VScale.
    if (ConcreteUF == 1)
      return match(Step, m_VPInstruction<VPInstruction::VScale>());
    if (match(Step, m_c_Mul(m_SpecificInt(ConcreteUF),
                            m_VPInstruction<VPInstruction::VScale>())))
      return true;
    // mul(VScale, ConcreteUF) may have been simplified to
    // shl(VScale, log2(ConcreteUF)) when ConcreteUF is a power of 2.
    return isPowerOf2_32(ConcreteUF) &&
           match(Step, m_Binary<Instruction::Shl>(
                           m_VPInstruction<VPInstruction::VScale>(),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  while (!WorkList.empty()) {
    const VPValue *Cur = WorkList.pop_back_val();
    if (!Seen.insert(Cur).second)
      continue;

    auto *Blend = dyn_cast<VPBlendRecipe>(Cur);
    // Skip blends that use V only through a compare by checking if any incoming
    // value was already visited.
    if (Blend && none_of(seq<unsigned>(0, Blend->getNumIncomingValues()),
                         [&](unsigned I) {
                           return Seen.contains(Blend->getIncomingValue(I));
                         }))
      continue;

```
- EN: Core entities appearing here include IsIncrementStep, isUsedByLoadStoreAddress, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 IsIncrementStep, isUsedByLoadStoreAddress，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 811-842

```cpp
    for (VPUser *U : Cur->users()) {
      if (auto *InterleaveR = dyn_cast<VPInterleaveBase>(U))
        if (InterleaveR->getAddr() == Cur)
          return true;
      if (auto *RepR = dyn_cast<VPReplicateRecipe>(U)) {
        if (RepR->getOpcode() == Instruction::Load &&
            RepR->getOperand(0) == Cur)
          return true;
        if (RepR->getOpcode() == Instruction::Store &&
            RepR->getOperand(1) == Cur)
          return true;
      }
      if (auto *MemR = dyn_cast<VPWidenMemoryRecipe>(cast<VPRecipeBase>(U))) {
        if (MemR->getAddr() == Cur && MemR->isConsecutive())
          return true;
      }
    }

    // The legacy cost model only supports scalarization loads/stores with phi
    // addresses, if the phi is directly used as load/store address. Don't
    // traverse further for Blends.
    if (Blend)
      continue;

    // Only traverse further through users that also define a value (and can
    // thus have their own users walked).
    for (VPUser *U : Cur->users())
      if (auto *SDR = dyn_cast<VPSingleDefRecipe>(U))
        WorkList.push_back(SDR);
  }
  return false;
}
```
- EN: This region continues the VPlanUtils implementation with local helper logic centered on VPUser, Cur, InterleaveR, VPInterleaveBase.
- CN: 这一段延续了 VPlanUtils 的主体实现，围绕 VPUser, Cur, InterleaveR, VPInterleaveBase 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `onlyFirstLaneUsed, onlyFirstPartUsed, onlyScalarValuesUsed, isHeaderMask, m_Specific, propagatesPoisonFromRecipeOp, poisonGuaranteesUB, getGEPFlagsForPtr` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`onlyFirstLaneUsed, onlyFirstPartUsed, onlyScalarValuesUsed, isHeaderMask, m_Specific, propagatesPoisonFromRecipeOp, poisonGuaranteesUB, getGEPFlagsForPtr` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: Utility headers: `llvm/ADT/TypeSwitch.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/TypeSwitch.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlanUtils.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanPatternMatch.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlanUtils.h`, `VPlanAnalysis.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanPatternMatch.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
