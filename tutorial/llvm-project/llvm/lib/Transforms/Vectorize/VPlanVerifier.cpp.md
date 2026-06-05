# VPlanVerifier.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanVerifier.cpp` | `llvm/lib/Transforms/Vectorize/VPlanVerifier.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements V Plan Verifier within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanVerifier 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-44

```cpp
//===-- VPlanVerifier.cpp -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines the class VPlanVerifier, which contains utility functions
/// to check the consistency and invariants of a VPlan.
///
//===----------------------------------------------------------------------===//

#include "VPlanVerifier.h"
#include "VPlan.h"
#include "VPlanCFG.h"
#include "VPlanDominatorTree.h"
#include "VPlanHelpers.h"
#include "VPlanPatternMatch.h"
#include "VPlanUtils.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/TypeSwitch.h"

#define DEBUG_TYPE "loop-vectorize"

using namespace llvm;
using namespace VPlanPatternMatch;

namespace {
class VPlanVerifier {
  const VPDominatorTree &VPDT;
  VPTypeAnalysis &TypeInfo;

  SmallPtrSet<BasicBlock *, 8> WrappedIRBBs;

  // Verify that phi-like recipes are at the beginning of \p VPBB, with no
  // other recipes in between. Also check that only header blocks contain
  // VPHeaderPHIRecipes.
  bool verifyPhiRecipes(const VPBasicBlock *VPBB);

  /// Verify that \p LastActiveLane's operand is guaranteed to be a prefix-mask.
  bool verifyLastActiveLaneRecipe(const VPInstruction &LastActiveLane) const;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 45-89

```cpp
  bool verifyVPBasicBlock(const VPBasicBlock *VPBB);

  bool verifyBlock(const VPBlockBase *VPB);

  /// Helper function that verifies the CFG invariants of the VPBlockBases
  /// within
  /// \p Region. Checks in this function are generic for VPBlockBases. They are
  /// not specific for VPBasicBlocks or VPRegionBlocks.
  bool verifyBlocksInRegion(const VPRegionBlock *Region);

  /// Verify the CFG invariants of VPRegionBlock \p Region and its nested
  /// VPBlockBases. Do not recurse inside nested VPRegionBlocks.
  bool verifyRegion(const VPRegionBlock *Region);

  /// Verify the CFG invariants of VPRegionBlock \p Region and its nested
  /// VPBlockBases. Recurse inside nested VPRegionBlocks.
  bool verifyRegionRec(const VPRegionBlock *Region);

public:
  VPlanVerifier(VPDominatorTree &VPDT, VPTypeAnalysis &TypeInfo)
      : VPDT(VPDT), TypeInfo(TypeInfo) {}

  bool verify(const VPlan &Plan);
};
} // namespace

bool VPlanVerifier::verifyPhiRecipes(const VPBasicBlock *VPBB) {
  auto RecipeI = VPBB->begin();
  auto End = VPBB->end();
  unsigned NumActiveLaneMaskPhiRecipes = 0;
  bool IsHeaderVPBB = VPBlockUtils::isHeader(VPBB, VPDT);
  while (RecipeI != End && RecipeI->isPhi()) {
    if (isa<VPActiveLaneMaskPHIRecipe>(RecipeI))
      NumActiveLaneMaskPhiRecipes++;

    if (IsHeaderVPBB &&
        !isa<VPHeaderPHIRecipe, VPWidenPHIRecipe, VPPhi>(*RecipeI)) {
      errs() << "Found non-header PHI recipe in header VPBB";
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
      errs() << ": ";
      RecipeI->dump();
#endif
      return false;
    }

```
- EN: Core entities appearing here include verifyPhiRecipes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 verifyPhiRecipes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 90-128

```cpp
    if (!IsHeaderVPBB && isa<VPHeaderPHIRecipe>(*RecipeI)) {
      errs() << "Found header PHI recipe in non-header VPBB";
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
      errs() << ": ";
      RecipeI->dump();
#endif
      return false;
    }

    if (isa<VPCurrentIterationPHIRecipe>(RecipeI) &&
        RecipeI->getIterator() != VPBB->begin()) {
      errs() << "CurrentIteration PHI is not the first recipe\n";
      return false;
    }

    // Check if the recipe operands match the number of predecessors.
    // TODO Extend to other phi-like recipes.
    if (auto *PhiIRI = dyn_cast<VPIRPhi>(&*RecipeI)) {
      if (PhiIRI->getNumOperands() != VPBB->getNumPredecessors()) {
        errs() << "Phi-like recipe with different number of operands and "
                  "predecessors.\n";
        // TODO: Print broken recipe. At the moment printing an ill-formed
        // phi-like recipe may crash.
        return false;
      }
    }

    RecipeI++;
  }

  if (!VPBB->getPlan()->isUnrolled() && NumActiveLaneMaskPhiRecipes > 1) {
    errs() << "There should be no more than one VPActiveLaneMaskPHIRecipe";
    return false;
  }

  while (RecipeI != End) {
    if (RecipeI->isPhi() && !isa<VPBlendRecipe>(&*RecipeI)) {
      errs() << "Found phi-like recipe after non-phi recipe";

```
- EN: This region continues the VPlanVerifier implementation with local helper logic centered on IsHeaderVPBB, VPHeaderPHIRecipe, RecipeI, Found.
- CN: 这一段延续了 VPlanVerifier 的主体实现，围绕 IsHeaderVPBB, VPHeaderPHIRecipe, RecipeI, Found 等局部辅助逻辑展开。

### Lines 129-172

```cpp
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
      errs() << ": ";
      RecipeI->dump();
      errs() << "after\n";
      std::prev(RecipeI)->dump();
#endif
      return false;
    }
    RecipeI++;
  }
  return true;
}

static bool isKnownMonotonic(VPValue *V) {
  VPValue *X, *Y;
  if (match(V, m_Add(m_VPValue(X), m_VPValue(Y))))
    return cast<VPRecipeWithIRFlags>(V)->hasNoUnsignedWrap() &&
           isKnownMonotonic(X) && isKnownMonotonic(Y);
  if (match(V, m_StepVector()))
    return true;
  // Only handle a subset of IVs until we can guarantee there's no overflow.
  if (auto *WidenIV = dyn_cast<VPWidenIntOrFpInductionRecipe>(V))
    return WidenIV->isCanonical() || WidenIV->hasNoUnsignedWrap();
  if (auto *Steps = dyn_cast<VPScalarIVStepsRecipe>(V))
    return match(Steps->getOperand(0),
                 m_CombineOr(
                     m_CanonicalIV(),
                     m_DerivedIV(m_ZeroInt(), m_CanonicalIV(), m_One()))) &&
           match(Steps->getStepValue(), m_One());
  if (isa<VPWidenCanonicalIVRecipe>(V))
    return true;
  return vputils::isUniformAcrossVFsAndUFs(V);
}

bool VPlanVerifier::verifyLastActiveLaneRecipe(
    const VPInstruction &LastActiveLane) const {
  assert(LastActiveLane.getOpcode() == VPInstruction::LastActiveLane &&
         "must be called with VPInstruction::LastActiveLane");

  if (LastActiveLane.getNumOperands() < 1) {
    errs() << "LastActiveLane must have at least one operand\n";
    return false;
  }

```
- EN: Core entities appearing here include isKnownMonotonic, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isKnownMonotonic，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 173-211

```cpp
  const VPlan &Plan = *LastActiveLane.getParent()->getPlan();
  // All operands must be prefix-mask. This means an icmp ult/ule LHS, RHS where
  // the LHS is monotonically increasing and RHS is uniform across VFs and UF.
  for (VPValue *Op : LastActiveLane.operands()) {
    if (vputils::isHeaderMask(Op, Plan))
      continue;

    CmpPredicate Pred;
    VPValue *LHS, *RHS;
    if (match(Op, m_ICmp(Pred, m_VPValue(LHS), m_VPValue(RHS))) &&
        (Pred == CmpInst::ICMP_ULE || Pred == CmpInst::ICMP_ULT) &&
        isKnownMonotonic(LHS) &&
        (vputils::isUniformAcrossVFsAndUFs(RHS) ||
         match(RHS, m_EVL(m_VPValue()))))
      continue;

    errs() << "LastActiveLane operand ";
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
    VPSlotTracker Tracker(&Plan);
    Op->printAsOperand(errs(), Tracker);
#endif
    errs() << " must be prefix mask (a header mask or an "
              "EVL-derived mask currently)\n";
    return false;
  }

  return true;
}

bool VPlanVerifier::verifyVPBasicBlock(const VPBasicBlock *VPBB) {
  if (!verifyPhiRecipes(VPBB))
    return false;

  // Verify that defs in VPBB dominate all their uses.
  DenseMap<const VPRecipeBase *, unsigned> RecipeNumbering;
  unsigned Cnt = 0;
  for (const VPRecipeBase &R : *VPBB)
    RecipeNumbering[&R] = Cnt++;

```
- EN: Core entities appearing here include verifyVPBasicBlock, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 verifyVPBasicBlock，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 212-253

```cpp
  for (const VPRecipeBase &R : *VPBB) {
    if (isa<VPIRInstruction>(&R) && !isa<VPIRBasicBlock>(VPBB)) {
      errs() << "VPIRInstructions ";
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
      R.dump();
      errs() << " ";
#endif
      errs() << "not in a VPIRBasicBlock!\n";
      return false;
    }
    for (const VPValue *V : R.definedValues()) {
      // Verify that we can infer a scalar type for each defined value. With
      // assertions enabled, inferScalarType will perform some consistency
      // checks during type inference.
      if (!TypeInfo.inferScalarType(V)) {
        errs() << "Failed to infer scalar type!\n";
        return false;
      }

      // MaskedCond may be used from blocks it don't dominate; the block will be
      // linearized and it will dominate its users after linearization.
      if (match(&R, m_VPInstruction<VPInstruction::MaskedCond>()))
        continue;

      for (const VPUser *U : V->users()) {
        auto *UI = cast<VPRecipeBase>(U);
        if (isa<VPIRPhi>(UI) &&
            UI->getNumOperands() != UI->getParent()->getNumPredecessors()) {
          errs() << "Phi-like recipe with different number of operands and "
                    "predecessors.\n";
          return false;
        }

        if (auto *Phi = dyn_cast<VPPhiAccessors>(UI)) {
          for (const auto &[IncomingVPV, IncomingVPBB] :
               Phi->incoming_values_and_blocks()) {
            if (IncomingVPV != V)
              continue;

            if (VPDT.dominates(VPBB, IncomingVPBB))
              continue;

```
- EN: This region continues the VPlanVerifier implementation with local helper logic centered on VPRecipeBase, VPBB, VPIRInstruction, VPIRBasicBlock.
- CN: 这一段延续了 VPlanVerifier 的主体实现，围绕 VPRecipeBase, VPBB, VPIRInstruction, VPIRBasicBlock 等局部辅助逻辑展开。

### Lines 254-292

```cpp
            errs() << "Incoming def does not dominate incoming block!\n";
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
            VPSlotTracker Tracker(VPBB->getPlan());
            IncomingVPV->getDefiningRecipe()->print(errs(), "  ", Tracker);
            errs() << "\n  does not dominate " << IncomingVPBB->getName()
                   << " for\n";
            UI->print(errs(), "  ", Tracker);
#endif
            return false;
          }
          continue;
        }
        // TODO: Also verify VPPredInstPHIRecipe.
        if (isa<VPPredInstPHIRecipe>(UI))
          continue;

        // If the user is in the same block, check it comes after R in the
        // block.
        if (UI->getParent() == VPBB) {
          if (RecipeNumbering[UI] >= RecipeNumbering[&R])
            continue;
        } else {
          if (VPDT.dominates(VPBB, UI->getParent()))
            continue;
        }

        // Recipes in blocks with a MaskedCond may be used in exit blocks; the
        // block will be linearized and its recipes will dominate their users
        // after linearization.
        bool BlockHasMaskedCond = any_of(*VPBB, [](const VPRecipeBase &R) {
          return match(&R, m_VPInstruction<VPInstruction::MaskedCond>());
        });
        if (BlockHasMaskedCond &&
            any_of(VPBB->getPlan()->getExitBlocks(), [UI](VPIRBasicBlock *EB) {
              return is_contained(EB->getPredecessors(), UI->getParent());
            })) {
          continue;
        }

```
- EN: Core entities appearing here include any_of, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 any_of，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 293-334

```cpp
        errs() << "Use before def!\n";
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
        VPSlotTracker Tracker(VPBB->getPlan());
        UI->print(errs(), "  ", Tracker);
        errs() << "\n  before\n";
        R.print(errs(), "  ", Tracker);
        errs() << "\n";
#endif
        return false;
      }
    }
    if (const auto *VPI = dyn_cast<VPInstruction>(&R)) {
      switch (VPI->getOpcode()) {
      case VPInstruction::LastActiveLane:
        if (!verifyLastActiveLaneRecipe(*VPI))
          return false;
        break;
      default:
        break;
      }
    }
    if (const auto *ScalarIVSteps = dyn_cast<VPScalarIVStepsRecipe>(&R)) {
      unsigned NumOps = ScalarIVSteps->getNumOperands();
      if (NumOps != 3 && NumOps != 4) {
        errs() << "VPScalarIVStepsRecipe must have 3 or 4 operands\n";
        return false;
      }
    }
  }

  auto *IRBB = dyn_cast<VPIRBasicBlock>(VPBB);
  if (!IRBB)
    return true;

  if (!WrappedIRBBs.insert(IRBB->getIRBasicBlock()).second) {
    errs() << "Same IR basic block used by multiple wrapper blocks!\n";
    return false;
  }

  return true;
}

```
- EN: This region continues the VPlanVerifier implementation with local helper logic centered on Use, NDEBUG, LLVM_ENABLE_DUMP, VPSlotTracker.
- CN: 这一段延续了 VPlanVerifier 的主体实现，围绕 Use, NDEBUG, LLVM_ENABLE_DUMP, VPSlotTracker 等局部辅助逻辑展开。

### Lines 335-381

```cpp
bool VPlanVerifier::verifyBlock(const VPBlockBase *VPB) {
  auto *VPBB = dyn_cast<VPBasicBlock>(VPB);
  // Check block's condition bit.
  if (VPBB && !isa<VPIRBasicBlock>(VPB)) {
    // For plain CFG VPlans, verify header and latch block structure.
    if (!VPBB->getParent()) {
      if (VPBlockUtils::isHeader(VPBB, VPDT)) {
        if (VPB->getNumPredecessors() != 2) {
          errs()
              << "Header block in plain CFG VPlan must have 2 predecessors!\n";
          return false;
        }
        // Predecessor 0 is preheader, predecessor 1 is latch.
        if (!VPBlockUtils::isLatch(VPB->getPredecessors()[1], VPDT)) {
          errs() << "Header's second predecessor must be the latch!\n";
          return false;
        }
      }

      if (VPBlockUtils::isLatch(VPBB, VPDT)) {
        auto BranchTerminator =
            m_CombineOr(m_BranchOnCond(),
                        m_CombineOr(m_BranchOnCount(), m_BranchOnTwoConds()));
        if (!match(VPBB->getTerminator(), BranchTerminator)) {
          errs() << "Latch block must have a branch terminator!\n";
          return false;
        }
        // Successor 0 is middle block, successor 1 is header.
        if (VPBlockUtils::isHeader(VPB->getSuccessors()[0], VPDT)) {
          errs() << "Latch's first successor must not be the header (must be "
                    "middle block)!\n";
          return false;
        }
      }
    } else if (VPB->getNumSuccessors() > 1 ||
               (VPBB->isExiting() && !VPBB->getParent()->isReplicator())) {
      if (!VPBB->getTerminator()) {
        errs() << "Block has multiple successors but doesn't "
                  "have a proper branch recipe!\n";
        return false;
      }
    } else if (VPBB->getTerminator()) {
      errs() << "Unexpected branch recipe!\n";
      return false;
    }
  }

```
- EN: Core entities appearing here include verifyBlock, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 verifyBlock，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 382-426

```cpp
  // Check block's successors.
  const auto &Successors = VPB->getSuccessors();
  for (const VPBlockBase *Succ : Successors) {
    // There must be a bi-directional link between block and successor.
    const auto &SuccPreds = Succ->getPredecessors();
    if (!is_contained(SuccPreds, VPB)) {
      errs() << "Missing predecessor link.\n";
      return false;
    }
  }

  // Check block's predecessors.
  const auto &Predecessors = VPB->getPredecessors();

  for (const VPBlockBase *Pred : Predecessors) {
    // Block and predecessor must be inside the same region.
    if (Pred->getParent() != VPB->getParent()) {
      errs() << "Predecessor is not in the same region.\n";
      return false;
    }

    // There must be a bi-directional link between block and predecessor.
    const auto &PredSuccs = Pred->getSuccessors();
    if (!is_contained(PredSuccs, VPB)) {
      errs() << "Missing successor link.\n";
      return false;
    }
  }
  return !VPBB || verifyVPBasicBlock(VPBB);
}

bool VPlanVerifier::verifyBlocksInRegion(const VPRegionBlock *Region) {
  for (const VPBlockBase *VPB : vp_depth_first_shallow(Region->getEntry())) {
    // Check block's parent.
    if (VPB->getParent() != Region) {
      errs() << "VPBlockBase has wrong parent\n";
      return false;
    }

    if (!verifyBlock(VPB))
      return false;
  }
  return true;
}

```
- EN: Core entities appearing here include verifyBlocksInRegion, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 verifyBlocksInRegion，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 427-471

```cpp
bool VPlanVerifier::verifyRegion(const VPRegionBlock *Region) {
  const VPBlockBase *Entry = Region->getEntry();
  const VPBlockBase *Exiting = Region->getExiting();

  // Entry and Exiting shouldn't have any predecessor/successor, respectively.
  if (Entry->hasPredecessors()) {
    errs() << "region entry block has predecessors\n";
    return false;
  }
  if (Exiting->getNumSuccessors() != 0) {
    errs() << "region exiting block has successors\n";
    return false;
  }

  return verifyBlocksInRegion(Region);
}

bool VPlanVerifier::verifyRegionRec(const VPRegionBlock *Region) {
  // Recurse inside nested regions and check all blocks inside the region.
  return verifyRegion(Region) &&
         all_of(vp_depth_first_shallow(Region->getEntry()),
                [this](const VPBlockBase *VPB) {
                  const auto *SubRegion = dyn_cast<VPRegionBlock>(VPB);
                  return !SubRegion || verifyRegionRec(SubRegion);
                });
}

bool VPlanVerifier::verify(const VPlan &Plan) {
  if (any_of(vp_depth_first_shallow(Plan.getEntry()),
             [this](const VPBlockBase *VPB) { return !verifyBlock(VPB); }))
    return false;

  const VPRegionBlock *TopRegion = Plan.getVectorLoopRegion();
  // TODO: Verify all blocks using vp_depth_first_deep iterators.
  if (!TopRegion)
    return true;

  if (!verifyRegionRec(TopRegion))
    return false;

  if (TopRegion->getParent()) {
    errs() << "VPlan Top Region should have no parent.\n";
    return false;
  }

```
- EN: Core entities appearing here include verifyRegion, verifyRegionRec, verify, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 verifyRegion, verifyRegionRec, verify，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 472-507

```cpp
  const VPBasicBlock *Entry = dyn_cast<VPBasicBlock>(TopRegion->getEntry());
  if (!Entry) {
    errs() << "VPlan entry block is not a VPBasicBlock\n";
    return false;
  }

  const VPBasicBlock *Exiting = dyn_cast<VPBasicBlock>(TopRegion->getExiting());
  if (!Exiting) {
    errs() << "VPlan exiting block is not a VPBasicBlock\n";
    return false;
  }

  if (Exiting->empty()) {
    errs() << "VPlan vector loop exiting block must end with BranchOnCount, "
              "BranchOnCond, or BranchOnTwoConds VPInstruction but is empty\n";
    return false;
  }

  auto *LastInst = dyn_cast<VPInstruction>(std::prev(Exiting->end()));
  if (!match(LastInst, m_CombineOr(m_BranchOnCond(),
                                   m_CombineOr(m_BranchOnCount(),
                                               m_BranchOnTwoConds())))) {
    errs() << "VPlan vector loop exit must end with BranchOnCount, "
              "BranchOnCond, or BranchOnTwoConds VPInstruction\n";
    return false;
  }

  return true;
}

bool llvm::verifyVPlanIsValid(const VPlan &Plan) {
  VPDominatorTree VPDT(const_cast<VPlan &>(Plan));
  VPTypeAnalysis TypeInfo(Plan);
  VPlanVerifier Verifier(VPDT, TypeInfo);
  return Verifier.verify(Plan);
}
```
- EN: Core entities appearing here include m_BranchOnTwoConds, verifyVPlanIsValid, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 m_BranchOnTwoConds, verifyVPlanIsValid，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `VPlanVerifier, verifyPhiRecipes, isKnownMonotonic, verifyVPBasicBlock, any_of, verifyBlock, verifyBlocksInRegion, verifyRegion` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`VPlanVerifier, verifyPhiRecipes, isKnownMonotonic, verifyVPBasicBlock, any_of, verifyBlock, verifyBlocksInRegion, verifyRegion` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Utility headers: `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/TypeSwitch.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SmallPtrSet.h`, `llvm/ADT/TypeSwitch.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlanVerifier.h`, `VPlan.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanUtils.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlanVerifier.h`, `VPlan.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanHelpers.h`, `VPlanPatternMatch.h`, `VPlanUtils.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
