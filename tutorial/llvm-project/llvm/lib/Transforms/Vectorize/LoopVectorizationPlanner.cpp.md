# LoopVectorizationPlanner.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/LoopVectorizationPlanner.cpp` | `llvm/lib/Transforms/Vectorize/LoopVectorizationPlanner.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vF selection and planning within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 LoopVectorizationPlanner 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-68

```cpp
//===- LoopVectorizationPlanner.cpp - VF selection and planning -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements VFSelectionContext methods for loop vectorization
/// VF selection, independent of cost-modeling decisions.
///
//===----------------------------------------------------------------------===//

#include "LoopVectorizationPlanner.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/Support/CommandLine.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
             "overriding the targets preference."));

/// Note: This currently only applies to `llvm.masked.load` and
/// `llvm.masked.store`. TODO: Extend this to cover other operations as needed.
static cl::opt<bool> ForceTargetSupportsMaskedMemoryOps(
    "force-target-supports-masked-memory-ops", cl::init(false), cl::Hidden,
    cl::desc("Assume the target supports masked memory operations (used for "
             "testing)."));

static cl::opt<bool> ForceTargetSupportsGatherScatterOps(
    "force-target-supports-gather-scatter-ops", cl::init(false), cl::Hidden,
    cl::desc("Assume the target supports gather/scatter operations (used for "
             "testing)."));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 69-136

```cpp
bool VFSelectionContext::isLegalMaskedLoadOrStore(Instruction *I,
                                                  ElementCount VF) const {
  assert(isa<LoadInst>(I) || isa<StoreInst>(I));
  auto *Ty = getLoadStoreType(I);
  const unsigned AS = getLoadStoreAddressSpace(I);
  const Align Alignment = getLoadStoreAlignment(I);

  return ForceTargetSupportsMaskedMemoryOps ||
         (isa<LoadInst>(I) ? TTI.isLegalMaskedLoad(Ty, Alignment, AS)
                           : TTI.isLegalMaskedStore(Ty, Alignment, AS));
}

bool VFSelectionContext::isLegalGatherOrScatter(Value *V,
                                                ElementCount VF) const {
  bool LI = isa<LoadInst>(V);
  bool SI = isa<StoreInst>(V);
  if (!LI && !SI)
    return false;
  auto *Ty = getLoadStoreType(V);
  Align Align = getLoadStoreAlignment(V);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      VF, VF.isScalable() ? MaxPermissibleVFWithoutMaxBW.ScalableVF
                          : MaxPermissibleVFWithoutMaxBW.FixedVF);
}

ElementCount VFSelectionContext::clampVFByMaxTripCount(
    ElementCount VF, unsigned MaxTripCount, unsigned UserIC,
    bool FoldTailByMasking, bool RequiresScalarEpilogue) const {
  unsigned EstimatedVF = VF.getKnownMinValue();
  if (VF.isScalable() && F.hasFnAttribute(Attribute::VScaleRange)) {
    auto Attr = F.getFnAttribute(Attribute::VScaleRange);
    auto Min = Attr.getVScaleRangeMin();
    EstimatedVF *= Min;
  }

```
- EN: Core entities appearing here include supportsScalableVectors, useMaxBandwidth, shouldConsiderRegPressureForVF, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 此处出现的核心实体包括 supportsScalableVectors, useMaxBandwidth, shouldConsiderRegPressureForVF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 137-207

```cpp
  // When a scalar epilogue is required, at least one iteration of the scalar
  // loop has to execute. Adjust MaxTripCount accordingly to avoid picking a
  // max VF that results in a dead vector loop.
  if (MaxTripCount > 0 && RequiresScalarEpilogue)
    MaxTripCount -= 1;

  // When the user specifies an interleave count, we need to ensure that
  // VF * UserIC <= MaxTripCount to avoid a dead vector loop.
  unsigned IC = UserIC > 0 ? UserIC : 1;
  unsigned EstimatedVFTimesIC = EstimatedVF * IC;

  if (MaxTripCount && MaxTripCount <= EstimatedVFTimesIC &&
      (!FoldTailByMasking || isPowerOf2_32(MaxTripCount))) {
    // If upper bound loop trip count (TC) is known at compile time there is no
    // point in choosing VF greater than TC / IC (as done in the loop below).
    // Select maximum power of two which doesn't exceed TC / IC. If VF is
    // scalable, we only fall back on a fixed VF when the TC is less than or
    // equal to the known number of lanes.
    auto ClampedUpperTripCount = llvm::bit_floor(MaxTripCount / IC);
    if (ClampedUpperTripCount == 0)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    LLVM_DEBUG(dbgs() << "LV: The target has no "
                      << (ComputeScalableMaxVF ? "scalable" : "fixed")
                      << " vector registers.\n");
    return ElementCount::getFixed(1);
  }

  ElementCount MaxVF =
      clampVFByMaxTripCount(MaxVectorElementCount, MaxTripCount, UserIC,
                            FoldTailByMasking, RequiresScalarEpilogue);
  // If the MaxVF was already clamped, there's no point in trying to pick a
  // larger one.
  if (MaxVF != MaxVectorElementCount)
    return MaxVF;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。

### Lines 208-280

```cpp
  if (MaxVF.isScalable())
    MaxPermissibleVFWithoutMaxBW.ScalableVF = MaxVF;
  else
    MaxPermissibleVFWithoutMaxBW.FixedVF = MaxVF;

  if (useMaxBandwidth(ComputeScalableMaxVF)) {
    auto MaxVectorElementCountMaxBW = ElementCount::get(
        llvm::bit_floor(WidestRegister.getKnownMinValue() / SmallestType),
        ComputeScalableMaxVF);
    MaxVF = MinVF(MaxVectorElementCountMaxBW, MaxSafeVF);

    if (ElementCount MinVF =
            TTI.getMinimumVF(SmallestType, ComputeScalableMaxVF)) {
      if (ElementCount::isKnownLT(MaxVF, MinVF)) {
        LLVM_DEBUG(dbgs() << "LV: Overriding calculated MaxVF(" << MaxVF
                          << ") with target's minimum: " << MinVF << '\n');
        MaxVF = MinVF;
      }
    }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // instead of invalidating vectorization for a whole set of VFs based on the
  // MaxVF.

  // Disable scalable vectorization if the loop contains unsupported reductions.
  if (!all_of(Legal->getReductionVars(), [&](const auto &Reduction) -> bool {
        return TTI.isLegalToVectorizeReduction(Reduction.second, MaxScalableVF);
      })) {
    reportVectorizationInfo(
        "Scalable vectorization not supported for the reduction "
        "operations found in this loop.",
        "ScalableVFUnfeasible", ORE, TheLoop);
    return false;
  }

```
- EN: Core entities appearing here include isScalableVectorizationAllowed, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isScalableVectorizationAllowed，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 281-345

```cpp
  // Disable scalable vectorization if the loop contains any instructions
  // with element types not supported for scalable vectors.
  if (any_of(ElementTypesInLoop, [&](Type *Ty) {
        return !Ty->isVoidTy() && !TTI.isElementTypeLegalForScalableVector(Ty);
      })) {
    reportVectorizationInfo("Scalable vectorization is not supported "
                            "for all element types found in this loop.",
                            "ScalableVFUnfeasible", ORE, TheLoop);
    return false;
  }

  if (!Legal->isSafeForAnyVectorWidth() && !getMaxVScale(F, TTI)) {
    reportVectorizationInfo("The target does not provide maximum vscale value "
                            "for safe distance analysis.",
                            "ScalableVFUnfeasible", ORE, TheLoop);
    return false;
  }

  IsScalableVectorizationAllowed = true;
  return true;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // It is computed by MaxVF * sizeOf(type) * 8, where type is taken from
  // the memory accesses that is most restrictive (involved in the smallest
  // dependence distance).
  unsigned MaxSafeElementsPowerOf2 =
      llvm::bit_floor(Legal->getMaxSafeVectorWidthInBits() / WidestType);
  if (!Legal->isSafeForAnyStoreLoadForwardDistances()) {
    unsigned SLDist = Legal->getMaxStoreLoadForwardSafeDistanceInBits();
    MaxSafeElementsPowerOf2 =
        std::min(MaxSafeElementsPowerOf2, SLDist / WidestType);
  }

  auto MaxSafeFixedVF = ElementCount::getFixed(MaxSafeElementsPowerOf2);
  auto MaxSafeScalableVF = getMaxLegalScalableVF(MaxSafeElementsPowerOf2);

```
- EN: Core entities appearing here include getMaxLegalScalableVF, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getMaxLegalScalableVF，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 346-415

```cpp
  if (!Legal->isSafeForAnyVectorWidth())
    MaxSafeElements = MaxSafeElementsPowerOf2;

  LLVM_DEBUG(dbgs() << "LV: The max safe fixed VF is: " << MaxSafeFixedVF
                    << ".\n");
  LLVM_DEBUG(dbgs() << "LV: The max safe scalable VF is: " << MaxSafeScalableVF
                    << ".\n");

  // First analyze the UserVF, fall back if the UserVF should be ignored.
  if (UserVF) {
    auto MaxSafeUserVF =
        UserVF.isScalable() ? MaxSafeScalableVF : MaxSafeFixedVF;

    if (ElementCount::isKnownLE(UserVF, MaxSafeUserVF)) {
      // If `VF=vscale x N` is safe, then so is `VF=N`
      if (UserVF.isScalable())
        return FixedScalableVFPair(
            ElementCount::getFixed(UserVF.getKnownMinValue()), UserVF);

      return UserVF;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      LLVM_DEBUG(dbgs() << "LV: User VF=" << UserVF
                        << " is unsafe. Ignoring scalable UserVF.\n");
      ORE->emit([&]() {
        return OptimizationRemarkAnalysis(DEBUG_TYPE, "VectorizationFactor",
                                          TheLoop->getStartLoc(),
                                          TheLoop->getHeader())
               << "User-specified vectorization factor "
               << ore::NV("UserVectorizationFactor", UserVF)
               << " is unsafe. Ignoring the hint to let the compiler pick a "
                  "more suitable value.";
      });
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 416-480

```cpp
  LLVM_DEBUG(dbgs() << "LV: The Smallest and Widest types: " << SmallestType
                    << " / " << WidestType << " bits.\n");

  FixedScalableVFPair Result(ElementCount::getFixed(1),
                             ElementCount::getScalable(0));
  if (auto MaxVF = getMaximizedVFForTarget(
          MaxTripCount, SmallestType, WidestType, MaxSafeFixedVF, UserIC,
          FoldTailByMasking, RequiresScalarEpilogue))
    Result.FixedVF = MaxVF;

  if (auto MaxVF = getMaximizedVFForTarget(
          MaxTripCount, SmallestType, WidestType, MaxSafeScalableVF, UserIC,
          FoldTailByMasking, RequiresScalarEpilogue))
    if (MaxVF.isScalable()) {
      Result.ScalableVF = MaxVF;
      LLVM_DEBUG(dbgs() << "LV: Found feasible scalable VF = " << MaxVF
                        << "\n");
    }

  return Result;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

void VFSelectionContext::collectElementTypesForWidening(
    const SmallPtrSetImpl<const Value *> *ValuesToIgnore) {
  ElementTypesInLoop.clear();
  // For each block.
  for (BasicBlock *BB : TheLoop->blocks()) {
    // For each instruction in the loop.
    for (Instruction &I : *BB) {
      Type *T = I.getType();

      // Skip ignored values.
      if (ValuesToIgnore && ValuesToIgnore->contains(&I))
        continue;

```
- EN: Core entities appearing here include getSmallestAndWidestTypes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 getSmallestAndWidestTypes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 481-546

```cpp
      // Only examine Loads, Stores and PHINodes.
      if (!isa<LoadInst, StoreInst, PHINode>(I))
        continue;

      // Examine PHI nodes that are reduction variables. Update the type to
      // account for the recurrence type.
      if (auto *PN = dyn_cast<PHINode>(&I)) {
        if (!Legal->isReductionVariable(PN))
          continue;
        const RecurrenceDescriptor &RdxDesc =
            Legal->getRecurrenceDescriptor(PN);
        if (PreferInLoopReductions || useOrderedReductions(RdxDesc) ||
            TTI.preferInLoopReduction(RdxDesc.getRecurrenceKind(),
                                      RdxDesc.getRecurrenceType()))
          continue;
        T = RdxDesc.getRecurrenceType();
      }

      // Examine the stored values.
      if (auto *ST = dyn_cast<StoreInst>(&I))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
bool VFSelectionContext::runtimeChecksRequired() {
  LLVM_DEBUG(dbgs() << "LV: Performing code size checks.\n");

  Loop *L = const_cast<Loop *>(TheLoop);
  if (Legal->getRuntimePointerChecking()->Need) {
    reportVectorizationFailure(
        "Runtime ptr check is required with -Os/-Oz",
        "runtime pointer checks needed. Enable vectorization of this "
        "loop with '#pragma clang loop vectorize(enable)' when "
        "compiling with -Os/-Oz",
        "CantVersionLoopWithOptForSize", ORE, L);
    return true;
  }

```
- EN: Core entities appearing here include initializeVScaleForTuning, runtimeChecksRequired, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 initializeVScaleForTuning, runtimeChecksRequired，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 547-611

```cpp
  if (!PSE.getPredicate().isAlwaysTrue()) {
    reportVectorizationFailure(
        "Runtime SCEV check is required with -Os/-Oz",
        "runtime SCEV checks needed. Enable vectorization of this "
        "loop with '#pragma clang loop vectorize(enable)' when "
        "compiling with -Os/-Oz",
        "CantVersionLoopWithOptForSize", ORE, L);
    return true;
  }

  // FIXME: Avoid specializing for stride==1 instead of bailing out.
  if (!Legal->getLAI()->getSymbolicStrides().empty()) {
    reportVectorizationFailure(
        "Runtime stride check for small trip count",
        "runtime stride == 1 checks needed. Enable vectorization of "
        "this loop without such check by compiling with -Os/-Oz",
        "CantVersionLoopWithOptForSize", ORE, L);
    return true;
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    // If the target would prefer this reduction to happen "in-loop", then we
    // want to record it as such.
    if (!PreferInLoopReductions && !useOrderedReductions(RdxDesc) &&
        !TTI.preferInLoopReduction(Kind, Phi->getType()))
      continue;

    // Check that we can correctly put the reductions into the loop, by
    // finding the chain of operations that leads from the phi to the loop
    // exit value.
    SmallVector<Instruction *, 4> ReductionOperations =
        RdxDesc.getReductionOpChain(Phi, const_cast<Loop *>(TheLoop));
    bool InLoop = !ReductionOperations.empty();

```
- EN: Core entities appearing here include computeMinimalBitwidths, collectInLoopReductions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 computeMinimalBitwidths, collectInLoopReductions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 612-675

```cpp
    if (InLoop) {
      InLoopReductions.insert(Phi);
      // Add the elements to InLoopReductionImmediateChains for cost modelling.
      Instruction *LastChain = Phi;
      for (auto *I : ReductionOperations) {
        InLoopReductionImmediateChains[I] = LastChain;
        LastChain = I;
      }
    }
    LLVM_DEBUG(dbgs() << "LV: Using " << (InLoop ? "inloop" : "out of loop")
                      << " reduction for phi: " << *Phi << "\n");
  }
}

bool LoopVectorizationPlanner::isMoreProfitable(const VectorizationFactor &A,
                                                const VectorizationFactor &B,
                                                const unsigned MaxTripCount,
                                                bool HasTail,
                                                bool IsEpilogue) const {
  InstructionCost CostA = A.Cost;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  auto CmpFn = [PreferScalable](const InstructionCost &LHS,
                                const InstructionCost &RHS) {
    return PreferScalable ? LHS <= RHS : LHS < RHS;
  };

  // To avoid the need for FP division:
  //      (CostA / EstimatedWidthA) < (CostB / EstimatedWidthB)
  // <=>  (CostA * EstimatedWidthB) < (CostB * EstimatedWidthA)
  bool LowerCostWithoutTC =
      CmpFn(CostA * EstimatedWidthB, CostB * EstimatedWidthA);
  if (!MaxTripCount)
    return LowerCostWithoutTC;

```
- EN: This region continues the LoopVectorizationPlanner implementation with local helper logic centered on InLoop, InLoopReductions, Phi, Add.
- CN: 这一段延续了 LoopVectorizationPlanner 的主体实现，围绕 InLoop, InLoopReductions, Phi, Add 等局部辅助逻辑展开。

### Lines 676-740

```cpp
  auto GetCostForTC = [MaxTripCount, HasTail](unsigned VF,
                                              InstructionCost VectorCost,
                                              InstructionCost ScalarCost) {
    // If the trip count is a known (possibly small) constant, the trip count
    // will be rounded up to an integer number of iterations under
    // FoldTailByMasking. The total cost in that case will be
    // VecCost*ceil(TripCount/VF). When not folding the tail, the total
    // cost will be VecCost*floor(TC/VF) + ScalarCost*(TC%VF). There will be
    // some extra overheads, but for the purpose of comparing the costs of
    // different VFs we can use this to compare the total loop-body cost
    // expected after vectorization.
    if (HasTail)
      return VectorCost * (MaxTripCount / VF) +
             ScalarCost * (MaxTripCount % VF);
    return VectorCost * divideCeil(MaxTripCount, VF);
  };

  auto RTCostA = GetCostForTC(EstimatedWidthA, CostA, A.ScalarCost);
  auto RTCostB = GetCostForTC(EstimatedWidthB, CostB, B.ScalarCost);
  bool LowerCostWithTC = CmpFn(RTCostA, RTCostB);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        "vectorization cannot be used because the target does not support "
        "scalable vectors.",
        "ScalableVFUnfeasible", ORE, TheLoop);
    return FixedScalableVFPair::getNone();
  }

  ElementCount VF = UserVF;
  if (VF.isZero()) {
    auto [_, WidestType] = getSmallestAndWidestTypes();

    auto RegKind = TTI.enableScalableVectorization()
                       ? TargetTransformInfo::RGK_ScalableVector
                       : TargetTransformInfo::RGK_FixedWidthVector;

```
- EN: Core entities appearing here include computeVPlanOuterloopVF, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 computeVPlanOuterloopVF，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 741-760

```cpp
    TypeSize RegSize = TTI.getRegisterBitWidth(RegKind);
    unsigned N = RegSize.getKnownMinValue() / WidestType;
    VF = ElementCount::get(N, RegSize.isScalable());
    LLVM_DEBUG(dbgs() << "LV: VPlan computed VF " << VF << ".\n");

    // Make sure we have a VF > 1 for stress testing.
    if (VPlanBuildOuterloopStressTest && VF.isScalar()) {
      LLVM_DEBUG(dbgs() << "LV: VPlan stress testing: "
                        << "overriding computed VF.\n");
      VF = ElementCount::getFixed(4);
    }
  }
  assert(isPowerOf2_32(VF.getKnownMinValue()) &&
         "VF needs to be a power of two");
  if (VF.isScalar())
    return FixedScalableVFPair::getNone();
  LLVM_DEBUG(dbgs() << "LV: Using " << (!UserVF.isZero() ? "user " : "")
                    << "VF " << VF << " to build VPlans.\n");
  return FixedScalableVFPair(VF);
}
```
- EN: This region continues the LoopVectorizationPlanner implementation with local helper logic centered on TypeSize, RegSize, TTI, RegKind.
- CN: 这一段延续了 LoopVectorizationPlanner 的主体实现，围绕 TypeSize, RegSize, TTI, RegKind 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `supportsScalableVectors, useMaxBandwidth, shouldConsiderRegPressureForVF, isScalableVectorizationAllowed, getMaxLegalScalableVF, getSmallestAndWidestTypes, initializeVScaleForTuning, runtimeChecksRequired` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`supportsScalableVectors, useMaxBandwidth, shouldConsiderRegPressureForVF, isScalableVectorizationAllowed, getMaxLegalScalableVF, getSmallestAndWidestTypes, initializeVScaleForTuning, runtimeChecksRequired` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `MaximizeBandwidth, UseWiderVFIfCallVariantsPresent, ConsiderRegPressure, ForceTargetSupportsScalableVectors, ForceTargetSupportsMaskedMemoryOps` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `MaximizeBandwidth, UseWiderVFIfCallVariantsPresent, ConsiderRegPressure, ForceTargetSupportsScalableVectors, ForceTargetSupportsMaskedMemoryOps` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolution.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DiagnosticInfo.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h`, `llvm/Transforms/Vectorize/LoopVectorize.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DiagnosticInfo.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h`, `llvm/Transforms/Vectorize/LoopVectorize.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/MathExtras.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/MathExtras.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `LoopVectorizationPlanner.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`LoopVectorizationPlanner.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
