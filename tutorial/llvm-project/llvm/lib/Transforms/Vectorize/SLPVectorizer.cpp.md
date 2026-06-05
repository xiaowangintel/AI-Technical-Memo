# SLPVectorizer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SLPVectorizer.cpp` | `llvm/lib/Transforms/Vectorize/SLPVectorizer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements a bottom up SLP Vectorizer within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 SLPVectorizer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-1341

```cpp
//===- SLPVectorizer.cpp - A bottom up SLP Vectorizer ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass implements the Bottom Up SLP vectorizer. It detects consecutive
// stores that can be put together into vector-stores. Next, it attempts to
// construct vectorizable tree using the use-def chains. If a profitable tree
// was found, the SLP vectorizer performs vectorization on the tree.
//
// The pass is inspired by the work described in the paper:
//  "Loop-Aware SLP in GCC" by Ira Rosen, Dorit Nuzman, Ayal Zaks.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SLPVectorizer.h"
#include "llvm/ADT/DenseMap.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// The vectorized result would be:
  /// intermediated_0 = add <4 x i32> <a, b, c, d>, <e, f, g, h>
  /// intermediated_1 = sub <4 x i32> <a, b, c, d>, <e, f, g, h>
  /// result = shufflevector <4 x i32> intermediated_0,
  ///                        <4 x i32> intermediated_1,
  ///                        <4 x i32> <i32 0, i32 5, i32 2, i32 7>
  /// Since shufflevector is used in the final result, when calculating the cost
  /// (getEntryCost), we must account for the usage of shufflevector in
  /// GetVectorCost.
  Instruction *MainOp = nullptr;
  Instruction *AltOp = nullptr;
  /// Wether the instruction state represents copyable instructions.
  bool HasCopyables = false;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 1342-2660

```cpp
public:
  Instruction *getMainOp() const {
    assert(valid() && "InstructionsState is invalid.");
    return MainOp;
  }

  Instruction *getAltOp() const {
    assert(valid() && "InstructionsState is invalid.");
    return AltOp;
  }

  /// The main/alternate opcodes for the list of instructions.
  unsigned getOpcode() const { return getMainOp()->getOpcode(); }

  unsigned getAltOpcode() const { return getAltOp()->getOpcode(); }

  /// Some of the instructions in the list have alternate opcodes.
  bool isAltShuffle() const { return getMainOp() != getAltOp(); }

  /// Checks if the instruction matches either the main or alternate opcode.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
              });
            };
            return AllUsersVectorized(V1) && AllUsersVectorized(V2);
          };
          // A broadcast of a load can be cheaper on some targets.
          if (R.TTI->isLegalBroadcastLoad(V1->getType(),
                                          ElementCount::getFixed(NumLanes)) &&
              ((int)V1->getNumUses() == NumLanes ||
               AllUsersAreInternal(V1, V2)))
            return LookAheadHeuristics::ScoreSplatLoads;
        }
        return LookAheadHeuristics::ScoreSplat;
      }

```
- EN: Core entities appearing here include isShiftOp, isBitwiseLogicOp, isMulDivLikeOp, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DataLayout, DemandedBits. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isShiftOp, isBitwiseLogicOp, isMulDivLikeOp，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DataLayout, DemandedBits 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2661-3978

```cpp
      auto CheckSameEntryOrFail = [&]() {
        if (ArrayRef<TreeEntry *> TEs1 = R.getTreeEntries(V1); !TEs1.empty()) {
          SmallPtrSet<TreeEntry *, 4> Set(llvm::from_range, TEs1);
          if (ArrayRef<TreeEntry *> TEs2 = R.getTreeEntries(V2);
              !TEs2.empty() &&
              any_of(TEs2, [&](TreeEntry *E) { return Set.contains(E); }))
            return LookAheadHeuristics::ScoreSplatLoads;
        }
        return LookAheadHeuristics::ScoreFail;
      };

      auto *LI1 = dyn_cast<LoadInst>(V1);
      auto *LI2 = dyn_cast<LoadInst>(V2);
      if (LI1 && LI2) {
        if (LI1->getParent() != LI2->getParent() || !LI1->isSimple() ||
            !LI2->isSimple())
          return CheckSameEntryOrFail();

        std::optional<int64_t> Dist = getPointersDiff(
            LI1->getType(), LI1->getPointerOperand(), LI2->getType(),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// per-entry operand invariance. Returns 1 when loop-aware cost modeling
  /// is disabled or \p L is null.
  uint64_t getLoopNestScale(const Loop *L);

  /// \returns a refined execution scale for a gather/buildvector tree entry
  /// \p TE. The scale is computed as the average of per-lane execution
  /// scales: each lane's scale is the loop-nest scale of the loop that
  /// contains the lane's defining instruction (or 1 if the lane is a
  /// constant / loop-invariant non-instruction value). This models the
  /// LICM hoisting that optimizeGatherSequence() performs after vectorization
  /// for inserts with loop-invariant operands. Falls back to the whole-entry
  /// scale when per-lane information is unavailable or the feature is off.
  uint64_t getGatherNodeEffectiveScale(const TreeEntry &TE);

```
- EN: Core entities appearing here include m_Undef, all_of, VLOperands, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, LoopNest, ScalarEvolution, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_Undef, all_of, VLOperands，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, LoopNest, ScalarEvolution, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3979-5295

```cpp
  /// Get the loop nest for the given loop \p L.
  ArrayRef<const Loop *> getLoopNest(const Loop *L);

  /// \returns the cost of the vectorizable entry.
  InstructionCost getEntryCost(const TreeEntry *E,
                               ArrayRef<Value *> VectorizedVals,
                               SmallPtrSetImpl<Value *> &CheckedExtracts);

  /// Estimates spill/reload cost from vector register pressure for \p E at the
  /// point of emitting its vector result type \p FinalVecTy. \p ScalarTy is the
  /// scalar/slot type used to widen into \p VecTy/\p FinalVecTy and may itself
  /// be a FixedVectorType in ReVec mode or an adjusted type due to MinBWs.
  InstructionCost getVectorSpillReloadCost(const TreeEntry *E, Type *ScalarTy,
                                           VectorType *VecTy,
                                           VectorType *FinalVecTy,
                                           TTI::TargetCostKind CostKind) const;

  /// This is the recursive part of buildTree.
  void buildTreeRec(ArrayRef<Value *> Roots, unsigned Depth, const EdgeInfo &EI,
                    unsigned InterleaveFactor = 0);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    /// The number of dependencies. Constitutes of the number of users of the
    /// instruction plus the number of dependent memory instructions (if any).
    /// This value is calculated on demand.
    /// If InvalidDeps, the number of dependencies is not calculated yet.
    int Dependencies = InvalidDeps;

    /// The number of dependencies minus the number of dependencies of scheduled
    /// instructions. As soon as this is zero, the instruction/bundle gets ready
    /// for scheduling.
    /// Note that this is negative as long as Dependencies is not calculated.
    int UnscheduledDeps = InvalidDeps;
  };

```
- EN: Core entities appearing here include TreeEntry, getCommonMask, getSplitMask, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AliasAnalysis, LoopNest, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 TreeEntry, getCommonMask, getSplitMask，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AliasAnalysis, LoopNest, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 5296-6609

```cpp
#ifndef NDEBUG
  friend inline raw_ostream &operator<<(raw_ostream &OS,
                                        const BoUpSLP::ScheduleData &SD) {
    SD.dump(OS);
    return OS;
  }
#endif

  class ScheduleBundle final : public ScheduleEntity {
    /// The schedule data for the instructions in the bundle.
    SmallVector<ScheduleEntity *> Bundle;
    /// True if this bundle is valid.
    bool IsValid = true;
    /// The TreeEntry that this instruction corresponds to.
    TreeEntry *TE = nullptr;
    ScheduleBundle(bool IsValid)
        : ScheduleEntity(Kind::ScheduleBundle), IsValid(IsValid) {}

  public:
    ScheduleBundle() : ScheduleEntity(Kind::ScheduleBundle) {}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

/// Reorders the given \p Reuses mask according to the given \p Mask. \p Reuses
/// contains original mask for the scalars reused in the node. Procedure
/// transform this mask in accordance with the given \p Mask.
static void reorderReuses(SmallVectorImpl<int> &Reuses, ArrayRef<int> Mask) {
  assert(!Mask.empty() && Reuses.size() == Mask.size() &&
         "Expected non-empty mask.");
  SmallVector<int> Prev(Reuses.begin(), Reuses.end());
  Prev.swap(Reuses);
  for (unsigned I = 0, E = Prev.size(); I < E; ++I)
    if (Mask[I] != PoisonMaskElem)
      Reuses[Mask[I]] = Prev[I];
}

```
- EN: Core entities appearing here include ScheduleBundle, classof, verify, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DemandedBits, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ScheduleBundle, classof, verify，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DemandedBits, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 6610-8027

```cpp
/// Reorders the given \p Order according to the given \p Mask. \p Order - is
/// the original order of the scalars. Procedure transforms the provided order
/// in accordance with the given \p Mask. If the resulting \p Order is just an
/// identity order, \p Order is cleared.
static void reorderOrder(SmallVectorImpl<unsigned> &Order, ArrayRef<int> Mask,
                         bool BottomOrder = false) {
  assert(!Mask.empty() && "Expected non-empty mask.");
  unsigned Sz = Mask.size();
  if (BottomOrder) {
    SmallVector<unsigned> PrevOrder;
    if (Order.empty()) {
      PrevOrder.resize(Sz);
      std::iota(PrevOrder.begin(), PrevOrder.end(), 0);
    } else {
      PrevOrder.swap(Order);
    }
    Order.assign(Sz, Sz);
    for (unsigned I = 0; I < Sz; ++I)
      if (Mask[I] != PoisonMaskElem)
        Order[I] = PrevOrder[Mask[I]];
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        return (!GEP && doesNotNeedToBeScheduled(P)) ||
               (GEP && GEP->getNumOperands() == 2 &&
                isa<Constant, Instruction>(GEP->getOperand(1)));
      })) {
    // Check if potential masked gather can be represented as series
    // of loads + insertsubvectors.
    // If masked gather cost is higher - better to vectorize, so
    // consider it as a gather node. It will be better estimated
    // later.
    if (!TryRecursiveCheck || !CheckForShuffledLoads(CommonAlignment, BestVF,
                                                     ProfitableGatherPointers))
      return LoadsState::ScatterVectorize;
  }

```
- EN: Core entities appearing here include TransformMaskToOrder, computeCommonAlignment, isReverseOrder, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 TransformMaskToOrder, computeCommonAlignment, isReverseOrder，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 8028-9514

```cpp
  return LoadsState::Gather;
}

static bool clusterSortPtrAccesses(ArrayRef<Value *> VL,
                                   ArrayRef<BasicBlock *> BBs, Type *ElemTy,
                                   const DataLayout &DL, ScalarEvolution &SE,
                                   SmallVectorImpl<unsigned> &SortedIndices) {
  assert(
      all_of(VL, [](const Value *V) { return V->getType()->isPointerTy(); }) &&
      "Expected list of pointer operands.");
  // Map from bases to a vector of (Ptr, Offset, OrigIdx), which we insert each
  // Ptr into, sort and return the sorted indices with values next to one
  // another.
  SmallMapVector<
      std::pair<BasicBlock *, Value *>,
      SmallVector<SmallVector<std::tuple<Value *, int64_t, unsigned>>>, 8>
      Bases;
  Bases
      .try_emplace(std::make_pair(
          BBs.front(), getUnderlyingObject(VL.front(), RecursionMaxDepth)))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          // the graph.
          Queue.push(Data.first);
        }
      } else {
        reorderOrder(Data.first->ReorderIndices, Mask);
      }
    }
  }
  // If the reordering is unnecessary, just remove the reorder.
  if (IgnoreReorder && !VectorizableTree.front()->ReorderIndices.empty() &&
      VectorizableTree.front()->ReuseShuffleIndices.empty())
    VectorizableTree.front()->ReorderIndices.clear();
}

```
- EN: Core entities appearing here include all_of, stable_sort, findPartiallyOrderedLoads, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of, stable_sort, findPartiallyOrderedLoads，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 9515-10844

```cpp
Instruction *BoUpSLP::getRootEntryInstruction(const TreeEntry &Entry) const {
  if (Entry.hasState() &&
      (Entry.getOpcode() == Instruction::Store ||
       Entry.getOpcode() == Instruction::Load) &&
      Entry.State == TreeEntry::StridedVectorize &&
      !Entry.ReorderIndices.empty() && isReverseOrder(Entry.ReorderIndices))
    return dyn_cast<Instruction>(Entry.Scalars[Entry.ReorderIndices.front()]);
  return dyn_cast<Instruction>(Entry.Scalars.front());
}

void BoUpSLP::buildExternalUses(
    const ExtraValueToDebugLocsMap &ExternallyUsedValues) {
  const size_t NumVectScalars = ScalarToTreeEntries.size() + 1;
  DenseMap<Value *, unsigned> ScalarToExtUses;
  // Collect the values that we need to extract from the tree.
  for (auto &TEPtr : VectorizableTree) {
    TreeEntry *Entry = TEPtr.get();

    // No need to handle users of gathered values.
    if (Entry->isGather() || Entry->State == TreeEntry::SplitVectorize ||
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      return TreeEntry::NeedToGather;
    return TreeEntry::Vectorize;
  case Instruction::GetElementPtr: {
    // We don't combine GEPs with complicated (nested) indexing.
    for (Value *V : VL) {
      auto *I = dyn_cast<GetElementPtrInst>(V);
      if (!I)
        continue;
      if (I->getNumOperands() != 2) {
        LLVM_DEBUG(dbgs() << "SLP: not-vectorizable GEP (nested indexes).\n");
        return TreeEntry::NeedToGather;
      }
    }

```
- EN: Core entities appearing here include any_of, all_of, collectUserStores, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, LoopNest, ScalarEvolution, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 any_of, all_of, collectUserStores，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, LoopNest, ScalarEvolution, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 10845-12238

```cpp
    // We can't combine several GEPs into one vector if they operate on
    // different types.
    Type *Ty0 = cast<GEPOperator>(VL0)->getSourceElementType();
    for (Value *V : VL) {
      auto *GEP = dyn_cast<GEPOperator>(V);
      if (!GEP)
        continue;
      Type *CurTy = GEP->getSourceElementType();
      if (Ty0 != CurTy) {
        LLVM_DEBUG(dbgs() << "SLP: not-vectorizable GEP (different types).\n");
        return TreeEntry::NeedToGather;
      }
    }

    // We don't combine GEPs with non-constant indexes.
    Type *Ty1 = VL0->getOperand(1)->getType();
    for (Value *V : VL) {
      auto *I = dyn_cast<GetElementPtrInst>(V);
      if (!I)
        continue;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
      // First operand not a constant or splat? Last attempt - check for
      // potential vectorization.
      InstructionsCompatibilityAnalysis Analysis(DT, DL, TTI, TLI);
      InstructionsState OpS = Analysis.buildInstructionsState(Ops, R);
      if (!OpS || (OpS.getOpcode() == Instruction::PHI && !allSameBlock(Ops)))
        return false;
      unsigned CopyableNum =
          count_if(Ops, [&](Value *V) { return OpS.isCopyableElement(V); });
      return CopyableNum <= VL.size() / 2;
    };
    if (!CheckOperand(Operands.front()))
      return OrigS;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PHIHandler, buildOperands, getMainAltOpsNoStateVL, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, TargetLibraryInfo, TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PHIHandler, buildOperands, getMainAltOpsNoStateVL，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, TargetLibraryInfo, TargetTransformInfo 等分析结果。

### Lines 12239-13552

```cpp
    return S;
  }

  SmallVector<BoUpSLP::ValueList> buildOperands(const InstructionsState &S,
                                                ArrayRef<Value *> VL) {
    assert(S && "Invalid state!");
    SmallVector<BoUpSLP::ValueList> Operands;
    if (S.areInstructionsWithCopyableElements()) {
      MainOp = S.getMainOp();
      MainOpcode = S.getOpcode();
      const bool IsCommutative =
          isCommutative(MainOp) && MainOp->getNumOperands() == 2;
      Operands.assign(MainOp->getNumOperands(),
                      BoUpSLP::ValueList(VL.size(), nullptr));
      // Populate operands for every lane.
      for (auto [Idx, V] : enumerate(VL)) {
        SmallVector<Value *> OperandsForValue = getOperands(S, V);
        for (auto [OperandIdx, Operand] : enumerate(OperandsForValue))
          Operands[OperandIdx][Idx] = Operand;
      }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                   const TargetLibraryInfo &TLI) {
  if (auto *MainCI = dyn_cast<CmpInst>(MainOp)) {
    auto *AltCI = cast<CmpInst>(AltOp);
    CmpInst::Predicate MainP = MainCI->getPredicate();
    [[maybe_unused]] CmpInst::Predicate AltP = AltCI->getPredicate();
    assert(MainP != AltP && "Expected different main/alternate predicates.");
    auto *CI = cast<CmpInst>(I);
    if (isCmpSameOrSwapped(MainCI, CI, TLI))
      return false;
    if (isCmpSameOrSwapped(AltCI, CI, TLI))
      return true;
    CmpInst::Predicate P = CI->getPredicate();
    CmpInst::Predicate SwappedP = CmpInst::getSwappedPredicate(P);

```
- EN: Core entities appearing here include PairInfo, all_of, getSameValuesTreeEntry, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DemandedBits, DominatorTree, LoopNest. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PairInfo, all_of, getSameValuesTreeEntry，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DemandedBits, DominatorTree, LoopNest 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 13553-14970

```cpp
    assert((MainP == P || AltP == P || MainP == SwappedP || AltP == SwappedP) &&
           "CmpInst expected to match either main or alternate predicate or "
           "their swap.");
    return MainP != P && MainP != SwappedP;
  }
  return InstructionsState(MainOp, AltOp).getMatchingMainOpOrAltOp(I) == AltOp;
}

TTI::OperandValueInfo BoUpSLP::getOperandInfo(ArrayRef<Value *> Ops) const {
  assert(!Ops.empty());
  const auto *Op0 = Ops.front();

  const bool IsConstant = all_of(Ops, [](Value *V) {
    // TODO: We should allow undef elements here
    return isConstant(V) && !isa<UndefValue>(V);
  });
  const bool IsUniform = all_of(Ops, [=](Value *V) {
    // TODO: We should allow undef elements here
    return V == Op0;
  });
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        auto IsInterleaveMask = [&, &TTI = *TTI](ArrayRef<int> Mask) {
          auto *BaseSI = cast<StoreInst>(E.Scalars.front());
          assert(Mask.size() > 1 && "Expected mask greater than 1 element.");
          if (Mask.size() < 4)
            return 0u;
          for (unsigned Factor : seq<unsigned>(2, Mask.size() / 2 + 1)) {
            if (ShuffleVectorInst::isInterleaveMask(
                    Mask, Factor, VecTy->getElementCount().getFixedValue()) &&
                TTI.isLegalInterleavedAccessType(
                    VecTy, Factor, BaseSI->getAlign(),
                    BaseSI->getPointerAddressSpace()))
              return Factor;
          }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include getOperandInfo, BaseShuffleAnalysis, getVF, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, TargetLibraryInfo, TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 getOperandInfo, BaseShuffleAnalysis, getVF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, TargetLibraryInfo, TargetTransformInfo 等分析结果。

### Lines 14971-16312

```cpp
          return 0u;
        };
        SmallVector<int> Mask(E.ReorderIndices.begin(), E.ReorderIndices.end());
        unsigned InterleaveFactor = IsInterleaveMask(Mask);
        if (InterleaveFactor != 0)
          E.setInterleave(InterleaveFactor);
      }
      break;
    }
    case Instruction::Select: {
      if (E.State != TreeEntry::Vectorize)
        break;
      auto [MinMaxID, SelectOnly] = canConvertToMinOrMaxIntrinsic(E.Scalars);
      if (MinMaxID != Intrinsic::not_intrinsic) {
        // This node is a minmax node.
        E.CombinedOp = TreeEntry::MinMax;
        TreeEntry *CondEntry = getOperandEntry(&E, 0);
        if (SelectOnly && CondEntry->UserTreeIndex &&
            CondEntry->State == TreeEntry::Vectorize) {
          // The condition node is part of the combined minmax node.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Sum = SaturatingAdd(Sum, LaneScale, &Overflow);
    if (Overflow)
      return BaseScale;
  }
  if (N == 0)
    return BaseScale;
  // Ceil-divide so we never round the effective scale down below 1.
  uint64_t Numerator = SaturatingAdd(Sum, uint64_t(N - 1), &Overflow);
  if (Overflow)
    return BaseScale;
  uint64_t Avg = Numerator / N;
  return std::clamp<uint64_t>(Avg, 1, BaseScale);
}

```
- EN: Core entities appearing here include all_of, BoUpSLP, getBuildVectorCost, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, LoopNest, ScalarEvolution, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of, BoUpSLP, getBuildVectorCost，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, LoopNest, ScalarEvolution, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 16313-17661

```cpp
InstructionCost
BoUpSLP::getVectorSpillReloadCost(const TreeEntry *E, Type *ScalarTy,
                                  VectorType *VecTy, VectorType *FinalVecTy,
                                  TTI::TargetCostKind CostKind) const {
  InstructionCost SpillsReloads = 0;

  // Estimate vector register pressure per target register class: operand
  // vectors plus the result. The same vector operand is counted once via
  // CountedOpEntries deduplication. PHIs take the max operand pressure across
  // incoming slots (only one predecessor is live at a time) plus the result.
  // All-constant operand bundles are skipped.
  if (!E->hasState() || E->getOpcode() == Instruction::Store ||
      E->getOpcode() == Instruction::ExtractElement ||
      E->getOpcode() == Instruction::ExtractValue ||
      E->getOpcode() == Instruction::Freeze ||
      (E->getOpcode() == Instruction::Load &&
       E->State != TreeEntry::ScatterVectorize))
    return SpillsReloads;

  const bool IsPHI =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                    *TTI, TargetTransformInfo::SK_PermuteSingleSrc, VecTy,
                    calculateShufflevectorMask(E->Scalars));
            }
            return TTI::TCC_Free;
          });
    return GetCostDiff(GetScalarCost, GetVectorCost);
  }
  case Instruction::Freeze:
    return CommonCost;
  default:
    llvm_unreachable("Unknown instruction");
  }
}

```
- EN: Core entities appearing here include all_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 17662-18990

```cpp
bool BoUpSLP::isFullyVectorizableTinyTree(bool ForReduction) const {
  LLVM_DEBUG(dbgs() << "SLP: Check whether the tree with height "
                    << VectorizableTree.size() << " is fully vectorizable .\n");

  auto &&AreVectorizableGathers = [this](const TreeEntry *TE, unsigned Limit) {
    SmallVector<int> Mask;
    return TE->isGather() &&
           !any_of(TE->Scalars,
                   [this](Value *V) { return EphValues.contains(V); }) &&
           (allConstant(TE->Scalars) || isSplat(TE->Scalars) ||
            TE->Scalars.size() < Limit ||
            (((TE->hasState() &&
               TE->getOpcode() == Instruction::ExtractElement) ||
              all_of(TE->Scalars, IsaPred<ExtractElementInst, UndefValue>)) &&
             isFixedVectorShuffle(TE->Scalars, Mask, AC)) ||
            (TE->hasState() && TE->getOpcode() == Instruction::Load &&
             !TE->isAltShuffle()) ||
            any_of(TE->Scalars, IsaPred<LoadInst>));
  };

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    bool HasNonPowerOf2 = any_of(Nodes, [&](unsigned Idx) {
      return !has_single_bit(VectorizableTree[Idx]->Scalars.size());
    });
    InstructionCost TrimMargin = HasNonPowerOf2 ? 1 : 0;
    if (TotalSubtreeCost > GatherCost + TrimMargin ||
        IsEqualCostAltShuffleToTrim()) {
      PreferTrimmedTree |= TotalSubtreeCost == GatherCost;
      // If the remaining tree is just a buildvector - exit, it will cause
      // endless attempts to vectorize.
      if (VectorizableTree.front()->hasState() &&
          VectorizableTree.front()->getOpcode() == Instruction::InsertElement &&
          TE->Idx == 1)
        return InstructionCost::getInvalid();

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include isFullyVectorizableTinyTree, isTreeTinyAndNotFullyVectorizable, all_of, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 isFullyVectorizableTinyTree, isTreeTinyAndNotFullyVectorizable, all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 18991-20325

```cpp
      LLVM_DEBUG(dbgs() << "SLP: Trimming unprofitable subtree at node "
                        << TE->Idx << " with cost "
                        << std::get<0>(Worklist.top().second)
                        << " and gather cost " << GatherCost << ".\n");
      if (TE->UserTreeIndex) {
        TransformedToGatherNodes.try_emplace(TE, GatherCost);
        NodesCosts.erase(TE);
      } else {
        DeletedNodes.insert(TE);
        TransformedToGatherNodes.erase(TE);
        NodesCosts.erase(TE);
      }
      for (unsigned Idx : Nodes) {
        TreeEntry &ChildTE = *VectorizableTree[Idx];
        DeletedNodes.insert(&ChildTE);
        TransformedToGatherNodes.erase(&ChildTE);
        NodesCosts.erase(&ChildTE);
      }
      Changed = true;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // source vectors.
      if (Idx == UsedTEs.size()) {
        // If the number of input vectors is greater than 2 - not a permutation,
        // fallback to the regular gather.
        // TODO: support multiple reshuffled nodes.
        if (UsedTEs.size() == 2)
          continue;
        UsedTEs.push_back(SavedVToTEs);
        Idx = UsedTEs.size() - 1;
      }
      UsedValuesEntry.try_emplace(V, Idx);
    }
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include none_of, all_of, count_if, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 none_of, all_of, count_if，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 20326-21806

```cpp
  if (UsedTEs.empty()) {
    Entries.clear();
    return std::nullopt;
  }

  unsigned VF = 0;
  if (UsedTEs.size() == 1) {
    // Keep the order to avoid non-determinism.
    SmallVector<const TreeEntry *> FirstEntries(UsedTEs.front().begin(),
                                                UsedTEs.front().end());
    sort(FirstEntries, [](const TreeEntry *TE1, const TreeEntry *TE2) {
      return TE1->Idx < TE2->Idx;
    });
    // Try to find the perfect match in another gather node at first.
    auto *It = find_if(FirstEntries, [=](const TreeEntry *EntryPtr) {
      return EntryPtr->isSame(VL) || EntryPtr->isSame(TE->Scalars);
    });
    if (It != FirstEntries.end() &&
        (IsReusedNodeFound || (*It)->getVectorFactor() == VL.size() ||
         ((*It)->getVectorFactor() == TE->Scalars.size() &&
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        for (auto [I1, I2] : zip(SVMask, CommonMask)) {
          if (I2 != PoisonMaskElem) {
            assert(I1 == PoisonMaskElem && "Expected unused subvectors mask");
            I1 = I2 + CommonMask.size();
          }
        }
        Value *InsertVec =
            CreateSubVectors(PoisonValue::get(Vec->getType()), CommonMask);
        Vec = createShuffle(InsertVec, Vec, SVMask);
        transformMaskAfterShuffle(CommonMask, SVMask);
      }
      InVectors.front() = Vec;
    }

```
- EN: Core entities appearing here include sort, none_of, setInsertPointAfterBundle, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 sort, none_of, setInsertPointAfterBundle，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 21807-23123

```cpp
    if (!ExtMask.empty()) {
      if (CommonMask.empty()) {
        CommonMask.assign(ExtMask.begin(), ExtMask.end());
      } else {
        SmallVector<int> NewMask(ExtMask.size(), PoisonMaskElem);
        for (int I = 0, Sz = ExtMask.size(); I < Sz; ++I) {
          if (ExtMask[I] == PoisonMaskElem)
            continue;
          NewMask[I] = CommonMask[ExtMask[I]];
        }
        CommonMask.swap(NewMask);
      }
    }
    if (CommonMask.empty()) {
      assert(InVectors.size() == 1 && "Expected only one vector with no mask");
      return InVectors.front();
    }
    if (InVectors.size() == 2)
      return createShuffle(InVectors.front(), InVectors.back(), CommonMask);
    return createShuffle(InVectors.front(), nullptr, CommonMask);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }

      V = FinalShuffle(V, E);

      E->VectorizedValue = V;
      ++NumVectorInstructions;

      return V;
    }
    case Instruction::Load: {
      // Loads are inserted at the head of the tree because we don't want to
      // sink them all the way down past store instructions.
      setInsertPointAfterBundle(E);

```
- EN: Core entities appearing here include ~ShuffleInstructionBuilder, isIdentityMask, equal, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ~ShuffleInstructionBuilder, isIdentityMask, equal，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 23124-24440

```cpp
      LoadInst *LI = cast<LoadInst>(VL0);
      Instruction *NewLI;
      FixedVectorType *StridedLoadTy = nullptr;
      Value *PO = LI->getPointerOperand();
      if (E->State == TreeEntry::Vectorize) {
        NewLI = Builder.CreateAlignedLoad(VecTy, PO, LI->getAlign());
      } else if (E->State == TreeEntry::CompressVectorize) {
        auto [CompressMask, LoadVecTy, InterleaveFactor, IsMasked] =
            CompressEntryToData.at(E);
        Align CommonAlignment = LI->getAlign();
        if (IsMasked) {
          unsigned VF = getNumElements(LoadVecTy);
          SmallVector<Constant *> MaskValues(
              VF / getNumElements(LI->getType()),
              ConstantInt::getFalse(VecTy->getContext()));
          for (int I : CompressMask)
            MaskValues[I] = ConstantInt::getTrue(VecTy->getContext());
          if (auto *VecTy = dyn_cast<FixedVectorType>(LI->getType())) {
            assert(SLPReVec && "Only supported by REVEC.");
            MaskValues = replicateMask(MaskValues, VecTy->getNumElements());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

void BoUpSLP::optimizeGatherSequence() {
  LLVM_DEBUG(dbgs() << "SLP: Optimizing " << GatherShuffleExtractSeq.size()
                    << " gather sequences instructions.\n");
  // LICM InsertElementInst sequences.
  for (Instruction *I : GatherShuffleExtractSeq) {
    if (isDeleted(I))
      continue;

    // Check if this block is inside a loop.
    Loop *L = LI->getLoopFor(I->getParent());
    if (!L)
      continue;

```
- EN: Core entities appearing here include transform, any_of, optimizeGatherSequence, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 transform, any_of, optimizeGatherSequence，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 24441-25758

```cpp
    // Check if it has a preheader.
    BasicBlock *PreHeader = L->getLoopPreheader();
    if (!PreHeader)
      continue;

    // If the vector or the element that we insert into it are
    // instructions that are defined in this basic block then we can't
    // hoist this instruction.
    if (any_of(I->operands(), [L](Value *V) {
          auto *OpI = dyn_cast<Instruction>(V);
          return OpI && L->contains(OpI);
        }))
      continue;

    // We can hoist this instruction. Move it to the pre-header.
    I->moveBefore(PreHeader->getTerminator()->getIterator());
    CSEBlocks.insert(PreHeader);
  }

  // Make a list of all reachable blocks in our CSE queue.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *FirstNonBool = nullptr;
  while (!Worklist.empty()) {
    auto [I, Parent, Level] = Worklist.pop_back_val();

    // We should only be looking at scalar instructions here. If the current
    // instruction has a vector type, skip.
    auto *Ty = I->getType();
    if (isa<VectorType>(Ty))
      continue;
    if (Ty != Builder.getInt1Ty() && !FirstNonBool)
      FirstNonBool = I;
    if (Level > RecursionMaxDepth)
      continue;

```
- EN: Core entities appearing here include sort, any_of, all_of, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 sort, any_of, all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 25759-27076

```cpp
    // If the current instruction is a load, update MaxWidth to reflect the
    // width of the loaded value.
    if (isa<LoadInst, ExtractElementInst, ExtractValueInst>(I))
      Width = std::max<unsigned>(Width, DL->getTypeSizeInBits(Ty));

    // Otherwise, we need to visit the operands of the instruction. We only
    // handle the interesting cases from buildTree here. If an operand is an
    // instruction we haven't yet visited and from the same basic block as the
    // user or the use is a PHI node, we add it to the worklist.
    else if (isa<PHINode, CastInst, GetElementPtrInst, CmpInst, SelectInst,
                 BinaryOperator, UnaryOperator>(I)) {
      for (Use &U : I->operands()) {
        if (auto *J = dyn_cast<Instruction>(U.get()))
          if (Visited.insert(J).second &&
              (isa<PHINode>(I) || J->getParent() == Parent)) {
            Worklist.emplace_back(J, J->getParent(), Level + 1);
            continue;
          }
        if (!FirstNonBool && U.get()->getType() != Builder.getInt1Ty())
          FirstNonBool = U.get();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// Return the index of the first vectorized store after \p StartIdx
unsigned StoreChainContext::getFirstVecStoreAfter(unsigned StartIdx) const {
  return std::distance(
      RangeSizes.begin(),
      find_if(RangeSizes.drop_front(StartIdx),
              [this](const SizePair &P) { return this->isVectorized(P); }));
}

// Return true if all stores have been vectorized
bool StoreChainContext::allVectorized() const {
  return all_of(RangeSizes,
                [this](const SizePair &P) { return this->isVectorized(P); });
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include get, any_of, computeMinimumValueSizes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AAManager, AssumptionCache, DataLayout.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 get, any_of, computeMinimumValueSizes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AAManager, AssumptionCache, DataLayout 等分析结果。

### Lines 27077-28419

```cpp
// Return true if all elements in the given range match \p TreeSize
bool StoreChainContext::isFirstSizeSameRange(unsigned StartIdx, unsigned Length,
                                             unsigned TreeSize) const {
  return all_of(RangeSizes.slice(StartIdx, Length),
                [TreeSize, this](const SizePair &P) {
                  return firstSizeSame(TreeSize, P);
                });
}

// Return true if the \p TreeSize is profitable for all elements in the range
bool StoreChainContext::allOfRangeProfitable(unsigned StartIdx, unsigned Length,
                                             unsigned TreeSize) const {
  return all_of(RangeSizes.slice(StartIdx, Length),
                [TreeSize, this](const SizePair &P) {
                  return isVFProfitable(TreeSize, P);
                });
}

// Update the live (first) range sizes from the cached values (second)
void StoreChainContext::updateRangeSizesFromCache() {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        ReducedVals.back().append(Data.rbegin(), Data.rend());
      }
    }
    // Post optimize reduced values to get better reduction sequences and sort
    // them by size.
    optimizeReducedVals(R, DT, DL, TTI, TLI);
    // Sort the reduced values by number of same/alternate opcode and/or pointer
    // operand.
    stable_sort(ReducedVals, [](ArrayRef<Value *> P1, ArrayRef<Value *> P2) {
      return P1.size() > P2.size();
    });
    return true;
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include updateRangeSizesFromCache, updateCandidateVFs, getCurrentVF, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, ScalarEvolution, TargetLibraryInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 updateRangeSizesFromCache, updateCandidateVFs, getCurrentVF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, ScalarEvolution, TargetLibraryInfo 等分析结果。

### Lines 28420-29746

```cpp
  /// Attempt to vectorize the tree found by matchAssociativeReduction.
  Value *tryToReduce(BoUpSLP &V, const DataLayout &DL, TargetTransformInfo *TTI,
                     const TargetLibraryInfo &TLI, AssumptionCache *AC,
                     DominatorTree &DT) {
    constexpr unsigned RegMaxNumber = 4;
    const unsigned RedValsMaxNumber =
        (RK == ReductionOrdering::Ordered &&
         ReductionRoot->getType()->isIntOrIntVectorTy())
            ? 48
            : 128;
    // If there are a sufficient number of reduction values, reduce
    // to a nearby power-of-2. We can safely generate oversized
    // vectors and rely on the backend to split them to legal sizes.
    if (unsigned NumReducedVals =
            accumulate(ReducedVals, 0,
                       [](unsigned Num, ArrayRef<Value *> Vals) -> unsigned {
                         if (!isGoodForReduction(Vals))
                           return Num;
                         return Num + Vals.size();
                       });
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    case RecurKind::FMulAdd:
    case RecurKind::AnyOf:
    case RecurKind::FindIV:
    case RecurKind::FindLast:
    case RecurKind::FMaxNum:
    case RecurKind::FMinNum:
    case RecurKind::FMaximumNum:
    case RecurKind::FMinimumNum:
    case RecurKind::None:
      llvm_unreachable("Unexpected reduction kind for repeated scalar.");
    }
    return nullptr;
  }

```
- EN: Core entities appearing here include any_of, enumerate, isGuaranteedNotToBePoison, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 any_of, enumerate, isGuaranteedNotToBePoison，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 29747-31078

```cpp
  /// Emits actual operation for the scalar identity values, found during
  /// horizontal reduction analysis.
  Value *
  emitReusedOps(Value *VectorizedValue, IRBuilderBase &Builder, BoUpSLP &R,
                const SmallMapVector<Value *, unsigned, 16> &SameValuesCounter,
                ArrayRef<Value *> TrackedToOrig) {
    assert(IsSupportedHorRdxIdentityOp &&
           "The optimization of matched scalar identity horizontal reductions "
           "must be supported.");
    ArrayRef<Value *> VL = R.getRootNodeScalars();
    auto *VTy = cast<FixedVectorType>(VectorizedValue->getType());
    if (VTy->getElementType() != VL.front()->getType()) {
      VectorizedValue = Builder.CreateIntCast(
          VectorizedValue,
          getWidenedType(VL.front()->getType(), VTy->getNumElements()),
          R.isSignedMinBitwidthRootNode());
    }
    switch (RdxKind) {
    case RecurKind::Add: {
      // root = mul prev_root, <1, 1, n, 1>
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      while (!Nodes.empty()) {
        auto *PHI = cast<PHINode>(Nodes.pop_back_val());
        if (!Visited.insert(PHI).second)
          continue;
        for (Value *V : PHI->incoming_values()) {
          if (auto *PHI1 = dyn_cast<PHINode>((V))) {
            Nodes.push_back(PHI1);
            continue;
          }
          Opcodes.emplace_back(V);
        }
      }
    }

```
- EN: Core entities appearing here include getRdxKind, getAggregateSize, matchRdxBop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, TargetLibraryInfo, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getRdxKind, getAggregateSize, matchRdxBop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, TargetLibraryInfo, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 31079-31510

```cpp
    HaveVectorizedPhiNodes = tryToVectorizeSequence<Value>(
        Incoming, PHICompare, AreCompatiblePHIs,
        [this, &R](ArrayRef<Value *> Candidates, bool MaxVFOnly) {
          return tryToVectorizeList(Candidates, R, MaxVFOnly);
        },
        /*MaxVFOnly=*/true, R);
    Changed |= HaveVectorizedPhiNodes;
    if (HaveVectorizedPhiNodes && any_of(PHIToOpcodes, [&](const auto &P) {
          auto *PHI = dyn_cast<PHINode>(P.first);
          return !PHI || R.isDeleted(PHI);
        }))
      PHIToOpcodes.clear();
    VisitedInstrs.insert_range(Incoming);
  } while (HaveVectorizedPhiNodes);

  VisitedInstrs.clear();

  InstSetVector PostProcessInserts;
  SmallSetVector<CmpInst *, 8> PostProcessCmps;
  // Non-vectorizable root instructions other than stores: calls
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Reverse stores to do bottom-to-top analysis. This is important if the
    // values are stores to the same addresses several times, in this case need
    // to follow the stores order (reversed to meet the memory dependecies).
    SmallVector<StoreInst *> ReversedStores(Pair.second.rbegin(),
                                            Pair.second.rend());
    Changed |= tryToVectorizeSequence<StoreInst>(
        ReversedStores, StoreSorter, AreCompatibleStores,
        [&](ArrayRef<StoreInst *> Candidates, bool) {
          return vectorizeStores(Candidates, R, Attempted);
        },
        /*MaxVFOnly=*/false, R);
  }
  return Changed;
}
```
- EN: Core entities appearing here include VectorizeInsertsAndCmps, vectorizeGEPIndices, vectorizeStoreChains, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 VectorizeInsertsAndCmps, vectorizeGEPIndices, vectorizeStoreChains，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `UseMask, ShuffleMode, BinOpSameOpcodeHelper, InterchangeableInfo, InstructionsState, slpvectorizer, TreeEntry, ScheduleEntity` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`UseMask, ShuffleMode, BinOpSameOpcodeHelper, InterchangeableInfo, InstructionsState, slpvectorizer, TreeEntry, ScheduleEntity` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AAManager, AliasAnalysis, AssumptionCache, DataLayout, DemandedBits, DominatorTree, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AAManager, AliasAnalysis, AssumptionCache, DataLayout, DemandedBits, DominatorTree, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `RunSLPVectorization, SLPReVec, SLPCostThreshold, ShouldVectorizeHor, ShouldStartVectorizeHorAtStore` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `RunSLPVectorization, SLPReVec, SLPCostThreshold, ShouldVectorizeHor, ShouldStartVectorizeHorAtStore` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CodeMetrics.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/DemandedBits.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CodeMetrics.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/DemandedBits.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/DOTGraphTraits.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/GraphWriter.h`, `llvm/Support/InstructionCost.h`, `llvm/Support/KnownBits.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/DOTGraphTraits.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/GraphWriter.h`, `llvm/Support/InstructionCost.h`, `llvm/Support/KnownBits.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `iterator`, `map`, `memory`, `optional`, `set` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `iterator`, `map`, `memory`, `optional`, `set` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DemandedBits`, `DominatorTree`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DemandedBits`, `DominatorTree`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
