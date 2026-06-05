# VPlanPredicator.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanPredicator.cpp` | `llvm/lib/Transforms/Vectorize/VPlanPredicator.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vPlan predicator within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 VPlanPredicator 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-33

```cpp
//===-- VPlanPredicator.cpp - VPlan predicator ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements predication for VPlans.
///
//===----------------------------------------------------------------------===//

#include "VPRecipeBuilder.h"
#include "VPlan.h"
#include "VPlanCFG.h"
#include "VPlanDominatorTree.h"
#include "VPlanPatternMatch.h"
#include "VPlanTransforms.h"
#include "VPlanUtils.h"
#include "llvm/ADT/PostOrderIterator.h"

using namespace llvm;
using namespace VPlanPatternMatch;

namespace {
class VPPredicator {
  /// Builder to construct recipes to compute masks.
  VPBuilder Builder;

  /// Dominator tree for the VPlan.
  VPDominatorTree VPDT;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include VPPredicator, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 VPPredicator，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 34-62

```cpp
  /// Post-dominator tree for the VPlan.
  VPPostDominatorTree VPPDT;

  /// When we if-convert we need to create edge masks. We have to cache values
  /// so that we don't end up with exponential recursion/IR.
  using EdgeMaskCacheTy =
      DenseMap<std::pair<const VPBasicBlock *, const VPBasicBlock *>,
               VPValue *>;
  using BlockMaskCacheTy = DenseMap<const VPBasicBlock *, VPValue *>;
  EdgeMaskCacheTy EdgeMaskCache;

  BlockMaskCacheTy BlockMaskCache;

  /// Create an edge mask for every destination of cases and/or default.
  void createSwitchEdgeMasks(const VPInstruction *SI);

  /// Computes and return the predicate of the edge between \p Src and \p Dst,
  /// possibly inserting new recipes at \p Dst (using Builder's insertion point)
  VPValue *createEdgeMask(const VPBasicBlock *Src, const VPBasicBlock *Dst);

  /// Record \p Mask as the *entry* mask of \p VPBB, which is expected to not
  /// already have a mask.
  void setBlockInMask(const VPBasicBlock *VPBB, VPValue *Mask) {
    // TODO: Include the masks as operands in the predicated VPlan directly to
    // avoid keeping the map of masks beyond the predication transform.
    assert(!getBlockInMask(VPBB) && "Mask already set");
    BlockMaskCache[VPBB] = Mask;
  }

```
- EN: Core entities appearing here include setBlockInMask, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, PostDominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 setBlockInMask，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, PostDominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 63-96

```cpp
  /// Record \p Mask as the mask of the edge from \p Src to \p Dst. The edge is
  /// expected to not have a mask already.
  VPValue *setEdgeMask(const VPBasicBlock *Src, const VPBasicBlock *Dst,
                       VPValue *Mask) {
    assert(Src != Dst && "Src and Dst must be different");
    assert(!getEdgeMask(Src, Dst) && "Mask already set");
    return EdgeMaskCache[{Src, Dst}] = Mask;
  }

public:
  VPPredicator(VPlan &Plan) : VPDT(Plan), VPPDT(Plan) {}

  /// Returns the *entry* mask for \p VPBB.
  VPValue *getBlockInMask(const VPBasicBlock *VPBB) const {
    return BlockMaskCache.lookup(VPBB);
  }

  /// Returns the precomputed predicate of the edge from \p Src to \p Dst.
  VPValue *getEdgeMask(const VPBasicBlock *Src, const VPBasicBlock *Dst) const {
    return EdgeMaskCache.lookup({Src, Dst});
  }

  /// Compute the predicate of \p VPBB.
  void createBlockInMask(VPBasicBlock *VPBB);

  /// Convert phi recipes in \p VPBB to VPBlendRecipes.
  void convertPhisToBlends(VPBasicBlock *VPBB);
};
} // namespace

VPValue *VPPredicator::createEdgeMask(const VPBasicBlock *Src,
                                      const VPBasicBlock *Dst) {
  assert(is_contained(Dst->getPredecessors(), Src) && "Invalid edge");

```
- EN: This region continues the VPlanPredicator implementation with local helper logic centered on Record, Mask, Src, Dst.
- CN: 这一段延续了 VPlanPredicator 的主体实现，围绕 Record, Mask, Src, Dst 等局部辅助逻辑展开。

### Lines 97-131

```cpp
  // Look for cached value.
  VPValue *EdgeMask = getEdgeMask(Src, Dst);
  if (EdgeMask)
    return EdgeMask;

  VPValue *SrcMask = getBlockInMask(Src);

  // If there's a single successor, there's no terminator recipe.
  if (Src->getNumSuccessors() == 1)
    return setEdgeMask(Src, Dst, SrcMask);

  auto *Term = cast<VPInstruction>(Src->getTerminator());
  if (Term->getOpcode() == Instruction::Switch) {
    createSwitchEdgeMasks(Term);
    return getEdgeMask(Src, Dst);
  }

  assert(Term->getOpcode() == VPInstruction::BranchOnCond &&
         "Unsupported terminator");
  if (Src->getSuccessors()[0] == Src->getSuccessors()[1])
    return setEdgeMask(Src, Dst, SrcMask);

  EdgeMask = Term->getOperand(0);
  assert(EdgeMask && "No Edge Mask found for condition");

  if (Src->getSuccessors()[0] != Dst)
    EdgeMask = Builder.createNot(EdgeMask, Term->getDebugLoc());

  if (SrcMask) { // Otherwise block in-mask is all-one, no need to AND.
    // The bitwise 'And' of SrcMask and EdgeMask introduces new UB if SrcMask
    // is false and EdgeMask is poison. Avoid that by using 'LogicalAnd'
    // instead which generates 'select i1 SrcMask, i1 EdgeMask, i1 false'.
    EdgeMask = Builder.createLogicalAnd(SrcMask, EdgeMask, Term->getDebugLoc());
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 132-165

```cpp
  return setEdgeMask(Src, Dst, EdgeMask);
}

void VPPredicator::createBlockInMask(VPBasicBlock *VPBB) {
  // Start inserting after the block's phis, which be replaced by blends later.
  Builder.setInsertPoint(VPBB, VPBB->getFirstNonPhi());

  // Reuse the mask of the immediate dominator if the VPBB post-dominates the
  // immediate dominator.
  auto *IDom = VPDT.getNode(VPBB)->getIDom();
  assert(IDom && "Block in loop must have immediate dominator");
  auto *IDomBB = cast<VPBasicBlock>(IDom->getBlock());
  if (VPPDT.properlyDominates(VPBB, IDomBB)) {
    setBlockInMask(VPBB, getBlockInMask(IDomBB));
    return;
  }
  // All-one mask is modelled as no-mask following the convention for masked
  // load/store/gather/scatter. Initialize BlockMask to no-mask.
  VPValue *BlockMask = nullptr;
  // This is the block mask. We OR all unique incoming edges.
  for (auto *Predecessor : SetVector<VPBlockBase *>(
           VPBB->getPredecessors().begin(), VPBB->getPredecessors().end())) {
    VPValue *EdgeMask = createEdgeMask(cast<VPBasicBlock>(Predecessor), VPBB);
    if (!EdgeMask) { // Mask of predecessor is all-one so mask of block is
                     // too.
      setBlockInMask(VPBB, EdgeMask);
      return;
    }

    if (!BlockMask) { // BlockMask has its initial nullptr value.
      BlockMask = EdgeMask;
      continue;
    }

```
- EN: Core entities appearing here include createBlockInMask, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createBlockInMask，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 166-197

```cpp
    BlockMask = Builder.createOr(BlockMask, EdgeMask, {});
  }

  setBlockInMask(VPBB, BlockMask);
}

void VPPredicator::createSwitchEdgeMasks(const VPInstruction *SI) {
  const VPBasicBlock *Src = SI->getParent();

  // Create masks where SI is a switch. We create masks for all edges from SI's
  // parent block at the same time. This is more efficient, as we can create and
  // collect compares for all cases once.
  VPValue *Cond = SI->getOperand(0);
  VPBasicBlock *DefaultDst = cast<VPBasicBlock>(Src->getSuccessors()[0]);
  MapVector<VPBasicBlock *, SmallVector<VPValue *>> Dst2Compares;
  for (const auto &[Idx, Succ] : enumerate(drop_begin(Src->getSuccessors()))) {
    VPBasicBlock *Dst = cast<VPBasicBlock>(Succ);
    assert(!getEdgeMask(Src, Dst) && "Edge masks already created");
    //  Cases whose destination is the same as default are redundant and can
    //  be ignored - they will get there anyhow.
    if (Dst == DefaultDst)
      continue;
    auto &Compares = Dst2Compares[Dst];
    VPValue *V = SI->getOperand(Idx + 1);
    Compares.push_back(Builder.createICmp(CmpInst::ICMP_EQ, Cond, V));
  }

  // We need to handle 2 separate cases below for all entries in Dst2Compares,
  // which excludes destinations matching the default destination.
  VPValue *SrcMask = getBlockInMask(Src);
  VPValue *DefaultMask = nullptr;
  for (const auto &[Dst, Conds] : Dst2Compares) {
```
- EN: Core entities appearing here include createSwitchEdgeMasks, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createSwitchEdgeMasks，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 198-226

```cpp
    // 1. Dst is not the default destination. Dst is reached if any of the
    // cases with destination == Dst are taken. Join the conditions for each
    // case whose destination == Dst using an OR.
    VPValue *Mask = Conds[0];
    for (VPValue *V : drop_begin(Conds))
      Mask = Builder.createOr(Mask, V);
    if (SrcMask)
      Mask = Builder.createLogicalAnd(SrcMask, Mask);
    setEdgeMask(Src, Dst, Mask);

    // 2. Create the mask for the default destination, which is reached if
    // none of the cases with destination != default destination are taken.
    // Join the conditions for each case where the destination is != Dst using
    // an OR and negate it.
    DefaultMask = DefaultMask ? Builder.createOr(DefaultMask, Mask) : Mask;
  }

  if (DefaultMask) {
    DefaultMask = Builder.createNot(DefaultMask);
    if (SrcMask)
      DefaultMask = Builder.createLogicalAnd(SrcMask, DefaultMask);
  } else {
    // There are no destinations other than the default destination, so this is
    // an unconditional branch.
    DefaultMask = SrcMask;
  }
  setEdgeMask(Src, DefaultDst, DefaultMask);
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 227-261

```cpp
void VPPredicator::convertPhisToBlends(VPBasicBlock *VPBB) {
  SmallVector<VPPhi *> Phis;
  for (VPRecipeBase &R : VPBB->phis())
    Phis.push_back(cast<VPPhi>(&R));
  for (VPPhi *PhiR : Phis) {
    // The non-header Phi is converted into a Blend recipe below,
    // so we don't have to worry about the insertion order and we can just use
    // the builder. At this point we generate the predication tree. There may
    // be duplications since this is a simple recursive scan, but future
    // optimizations will clean it up.

    auto NotPoison = make_filter_range(PhiR->incoming_values(), [](VPValue *V) {
      return !match(V, m_Poison());
    });
    if (all_equal(NotPoison)) {
      PhiR->replaceAllUsesWith(NotPoison.empty() ? PhiR->getIncomingValue(0)
                                                 : *NotPoison.begin());
      PhiR->eraseFromParent();
      continue;
    }

    SmallVector<VPValue *, 2> OperandsWithMask;
    for (const auto &[InVPV, InVPBB] : PhiR->incoming_values_and_blocks()) {
      OperandsWithMask.push_back(InVPV);
      OperandsWithMask.push_back(createEdgeMask(InVPBB, VPBB));
    }
    PHINode *IRPhi = cast_or_null<PHINode>(PhiR->getUnderlyingValue());
    auto *Blend =
        new VPBlendRecipe(IRPhi, OperandsWithMask, *PhiR, PhiR->getDebugLoc());
    Builder.insert(Blend);
    PhiR->replaceAllUsesWith(Blend);
    PhiR->eraseFromParent();
  }
}

```
- EN: Core entities appearing here include convertPhisToBlends, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 convertPhisToBlends，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 262-294

```cpp
void VPlanTransforms::introduceMasksAndLinearize(VPlan &Plan) {
  // Nested loop regions (outer-loop vectorization) are not supported yet.
  if (Plan.isOuterLoop())
    return;
  VPRegionBlock *LoopRegion = Plan.getVectorLoopRegion();
  // Scan the body of the loop in a topological order to visit each basic block
  // after having visited its predecessor basic blocks.
  VPBasicBlock *Header = LoopRegion->getEntryBasicBlock();
  ReversePostOrderTraversal<VPBlockShallowTraversalWrapper<VPBlockBase *>> RPOT(
      Header);
  VPPredicator Predicator(Plan);
  for (VPBlockBase *VPB : RPOT) {
    // Non-outer regions with VPBBs only are supported at the moment.
    auto *VPBB = cast<VPBasicBlock>(VPB);
    // Introduce the mask for VPBB, which may introduce needed edge masks, and
    // convert all phi recipes of VPBB to blend recipes unless VPBB is the
    // header.
    if (VPBB != Header) {
      Predicator.createBlockInMask(VPBB);
      Predicator.convertPhisToBlends(VPBB);
    }

    VPValue *BlockMask = Predicator.getBlockInMask(VPBB);
    if (!BlockMask)
      continue;

    // Mask all VPInstructions in the block.
    for (VPRecipeBase &R : *VPBB) {
      if (auto *VPI = dyn_cast<VPInstruction>(&R))
        VPI->addMask(BlockMask);
    }
  }

```
- EN: Core entities appearing here include introduceMasksAndLinearize, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 introduceMasksAndLinearize，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 295-311

```cpp
  // Linearize the blocks of the loop into one serial chain.
  VPBlockBase *PrevVPBB = nullptr;
  for (VPBasicBlock *VPBB : VPBlockUtils::blocksOnly<VPBasicBlock>(RPOT)) {
    auto Successors = to_vector(VPBB->getSuccessors());
    if (Successors.size() > 1)
      VPBB->getTerminator()->eraseFromParent();

    // Flatten the CFG in the loop. To do so, first disconnect VPBB from its
    // successors. Then connect VPBB to the previously visited VPBB.
    for (auto *Succ : Successors)
      VPBlockUtils::disconnectBlocks(VPBB, Succ);
    if (PrevVPBB)
      VPBlockUtils::connectBlocks(PrevVPBB, VPBB);

    PrevVPBB = VPBB;
  }
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `VPPredicator, setBlockInMask, createBlockInMask, createSwitchEdgeMasks, convertPhisToBlends, introduceMasksAndLinearize` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`VPPredicator, setBlockInMask, createBlockInMask, createSwitchEdgeMasks, convertPhisToBlends, introduceMasksAndLinearize` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, PostDominatorTree` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, PostDominatorTree` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Utility headers: `llvm/ADT/PostOrderIterator.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/PostOrderIterator.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPRecipeBuilder.h`, `VPlan.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h`, `VPlanUtils.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPRecipeBuilder.h`, `VPlan.h`, `VPlanCFG.h`, `VPlanDominatorTree.h`, `VPlanPatternMatch.h`, `VPlanTransforms.h`, `VPlanUtils.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree`, `PostDominatorTree` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `PostDominatorTree` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
