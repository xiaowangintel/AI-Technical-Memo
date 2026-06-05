# LoopVectorizationPlanner.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/LoopVectorizationPlanner.h` | `llvm/lib/Transforms/Vectorize/LoopVectorizationPlanner.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This header defines the public interface for planner for LoopVectorization within LLVM's vectorization and VPlan infrastructure layer. | 该头文件定义了对外接口 LLVM 向量化与 VPlan 基础设施层中的 LoopVectorizationPlanner 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-81

```cpp
//===- LoopVectorizationPlanner.h - Planner for LoopVectorization ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file provides a LoopVectorizationPlanner class.
/// InnerLoopVectorizer vectorizes loops which contain only one basic
/// LoopVectorizationPlanner - drives the vectorization process after having
/// passed Legality checks.
/// The planner builds and optimizes the Vectorization Plans which record the
/// decisions how to vectorize the given loop. In particular, represent the
/// control-flow of the vectorized version, the replication of instructions that
/// are to be scalarized, and interleave access groups.
///
/// Also provides a VPlan-based builder utility analogous to IRBuilder.
/// It provides an instruction-level API for generating VPInstructions while
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                             DebugLoc DL = {});

/// VPlan-based builder utility analogous to IRBuilder.
class VPBuilder {
  VPBasicBlock *BB = nullptr;
  VPBasicBlock::iterator InsertPt = VPBasicBlock::iterator();

  /// Insert \p VPI in BB at InsertPt if BB is set.
  template <typename T> T *tryInsertInstruction(T *R) {
    if (BB)
      BB->insert(R, InsertPt);
    return R;
  }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The header guard ensures the declarations are safe to include from multiple compilation units. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 头文件保护宏保证这些声明可以被多个编译单元安全地重复包含。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 82-164

```cpp
  VPInstruction *createInstruction(unsigned Opcode,
                                   ArrayRef<VPValue *> Operands,
                                   const VPIRMetadata &MD, DebugLoc DL,
                                   const Twine &Name = "") {
    return tryInsertInstruction(
        new VPInstruction(Opcode, Operands, {}, MD, DL, Name));
  }

public:
  VPBuilder() = default;
  VPBuilder(VPBasicBlock *InsertBB) { setInsertPoint(InsertBB); }
  VPBuilder(VPRecipeBase *InsertPt) { setInsertPoint(InsertPt); }
  VPBuilder(VPBasicBlock *TheBB, VPBasicBlock::iterator IP) {
    setInsertPoint(TheBB, IP);
  }

  /// Clear the insertion point: created instructions will not be inserted into
  /// a block.
  void clearInsertionPoint() {
    BB = nullptr;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// This specifies that created instructions should be inserted at the
  /// specified point.
  void setInsertPoint(VPBasicBlock *TheBB, VPBasicBlock::iterator IP) {
    BB = TheBB;
    InsertPt = IP;
  }

  /// This specifies that created instructions should be inserted at the
  /// specified point.
  void setInsertPoint(VPRecipeBase *IP) {
    BB = IP->getParent();
    InsertPt = IP->getIterator();
  }

```
- EN: Core entities appearing here include VPBuilder, clearInsertionPoint, getToInsertAfter, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 VPBuilder, clearInsertionPoint, getToInsertAfter，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 165-247

```cpp
  /// Insert \p R at the current insertion point. Returns \p R unchanged.
  template <typename T> [[maybe_unused]] T *insert(T *R) {
    BB->insert(R, InsertPt);
    return R;
  }

  /// Create an N-ary operation with \p Opcode, \p Operands and set \p Inst as
  /// its underlying Instruction.
  VPInstruction *createNaryOp(unsigned Opcode, ArrayRef<VPValue *> Operands,
                              Instruction *Inst = nullptr,
                              const VPIRFlags &Flags = {},
                              const VPIRMetadata &MD = {},
                              DebugLoc DL = DebugLoc::getUnknown(),
                              const Twine &Name = "") {
    VPInstruction *NewVPInst = tryInsertInstruction(
        new VPInstruction(Opcode, Operands, Flags, MD, DL, Name));
    NewVPInst->setUnderlyingValue(Inst);
    return NewVPInst;
  }
  VPInstruction *createNaryOp(unsigned Opcode, ArrayRef<VPValue *> Operands,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                           const Twine &Name = "") {
    return createInstruction(Instruction::BinaryOps::And, {LHS, RHS}, {}, DL,
                             Name);
  }

  VPInstruction *createOr(VPValue *LHS, VPValue *RHS,
                          DebugLoc DL = DebugLoc::getUnknown(),
                          const Twine &Name = "") {

    return tryInsertInstruction(new VPInstruction(
        Instruction::BinaryOps::Or, {LHS, RHS},
        VPRecipeWithIRFlags::DisjointFlagsTy(false), {}, DL, Name));
  }

```
- EN: Core entities appearing here include t, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 t，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 248-329

```cpp
  VPInstruction *
  createAdd(VPValue *LHS, VPValue *RHS, DebugLoc DL = DebugLoc::getUnknown(),
            const Twine &Name = "",
            VPRecipeWithIRFlags::WrapFlagsTy WrapFlags = {false, false}) {
    return createOverflowingOp(Instruction::Add, {LHS, RHS}, WrapFlags, DL,
                               Name);
  }

  VPInstruction *
  createSub(VPValue *LHS, VPValue *RHS, DebugLoc DL = DebugLoc::getUnknown(),
            const Twine &Name = "",
            VPRecipeWithIRFlags::WrapFlagsTy WrapFlags = {false, false}) {
    return createOverflowingOp(Instruction::Sub, {LHS, RHS}, WrapFlags, DL,
                               Name);
  }

  VPInstruction *createLogicalAnd(VPValue *LHS, VPValue *RHS,
                                  DebugLoc DL = DebugLoc::getUnknown(),
                                  const Twine &Name = "") {
    return createNaryOp(VPInstruction::LogicalAnd, {LHS, RHS}, DL, Name);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                              const Twine &Name = "") {
    return tryInsertInstruction(
        new VPInstruction(VPInstruction::PtrAdd, {Ptr, Offset},
                          GEPNoWrapFlags::none(), {}, DL, Name));
  }

  VPInstruction *createNoWrapPtrAdd(VPValue *Ptr, VPValue *Offset,
                                    GEPNoWrapFlags GEPFlags,
                                    DebugLoc DL = DebugLoc::getUnknown(),
                                    const Twine &Name = "") {
    return tryInsertInstruction(new VPInstruction(
        VPInstruction::PtrAdd, {Ptr, Offset}, GEPFlags, {}, DL, Name));
  }

```
- EN: The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 330-417

```cpp
  VPInstruction *createWidePtrAdd(VPValue *Ptr, VPValue *Offset,
                                  DebugLoc DL = DebugLoc::getUnknown(),
                                  const Twine &Name = "") {
    return tryInsertInstruction(
        new VPInstruction(VPInstruction::WidePtrAdd, {Ptr, Offset},
                          GEPNoWrapFlags::none(), {}, DL, Name));
  }

  VPPhi *createScalarPhi(ArrayRef<VPValue *> IncomingValues,
                         DebugLoc DL = DebugLoc::getUnknown(),
                         const Twine &Name = "", const VPIRFlags &Flags = {}) {
    return tryInsertInstruction(new VPPhi(IncomingValues, Flags, DL, Name));
  }

  VPWidenPHIRecipe *createWidenPhi(ArrayRef<VPValue *> IncomingValues,
                                   DebugLoc DL = DebugLoc::getUnknown(),
                                   const Twine &Name = "") {
    return tryInsertInstruction(new VPWidenPHIRecipe(IncomingValues, DL, Name));
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return createScalarCast(CastOp, Op, ResultTy, DL);
  }

  VPValue *createScalarSExtOrTrunc(VPValue *Op, Type *ResultTy, Type *SrcTy,
                                   DebugLoc DL) {
    if (ResultTy == SrcTy)
      return Op;
    Instruction::CastOps CastOp =
        ResultTy->getScalarSizeInBits() < SrcTy->getScalarSizeInBits()
            ? Instruction::Trunc
            : Instruction::SExt;
    return createScalarCast(CastOp, Op, ResultTy, DL);
  }

```
- EN: This region continues the LoopVectorizationPlanner implementation with local helper logic centered on VPInstruction, VPValue, Ptr, Offset.
- CN: 这一段延续了 LoopVectorizationPlanner 的主体实现，围绕 VPInstruction, VPValue, Ptr, Offset 等局部辅助逻辑展开。

### Lines 418-496

```cpp
  VPWidenCastRecipe *createWidenCast(Instruction::CastOps Opcode, VPValue *Op,
                                     Type *ResultTy) {
    return tryInsertInstruction(new VPWidenCastRecipe(
        Opcode, Op, ResultTy, nullptr, VPIRFlags::getDefaultFlags(Opcode)));
  }

  VPScalarIVStepsRecipe *
  createScalarIVSteps(Instruction::BinaryOps InductionOpcode,
                      FPMathOperator *FPBinOp, VPValue *IV, VPValue *Step,
                      VPValue *VF, DebugLoc DL) {
    return tryInsertInstruction(new VPScalarIVStepsRecipe(
        IV, Step, VF, InductionOpcode,
        FPBinOp ? FPBinOp->getFastMathFlags() : FastMathFlags(), DL));
  }

  VPExpandSCEVRecipe *createExpandSCEV(const SCEV *Expr) {
    return tryInsertInstruction(new VPExpandSCEVRecipe(Expr));
  }

  //===--------------------------------------------------------------------===//
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// Width 1 means no vectorization, cost 0 means uncomputed cost.
  static VectorizationFactor Disabled() {
    return {ElementCount::getFixed(1), 0, 0};
  }

  bool operator==(const VectorizationFactor &rhs) const {
    return Width == rhs.Width && Cost == rhs.Cost;
  }

  bool operator!=(const VectorizationFactor &rhs) const {
    return !(*this == rhs);
  }
};

```
- EN: Core entities appearing here include InsertPointGuard, VectorizationFactor, Disabled, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as SCEV, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 InsertPointGuard, VectorizationFactor, Disabled，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 SCEV 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 497-575

```cpp
/// A class that represents two vectorization factors (initialized with 0 by
/// default). One for fixed-width vectorization and one for scalable
/// vectorization. This can be used by the vectorizer to choose from a range of
/// fixed and/or scalable VFs in order to find the most cost-effective VF to
/// vectorize with.
struct FixedScalableVFPair {
  ElementCount FixedVF;
  ElementCount ScalableVF;

  FixedScalableVFPair()
      : FixedVF(ElementCount::getFixed(0)),
        ScalableVF(ElementCount::getScalable(0)) {}
  FixedScalableVFPair(const ElementCount &Max) : FixedScalableVFPair() {
    *(Max.isScalable() ? &ScalableVF : &FixedVF) = Max;
  }
  FixedScalableVFPair(const ElementCount &FixedVF,
                      const ElementCount &ScalableVF)
      : FixedVF(FixedVF), ScalableVF(ScalableVF) {
    assert(!FixedVF.isScalable() && ScalableVF.isScalable() &&
           "Invalid scalable properties");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  void initializeVScaleForTuning();

  const TargetTransformInfo &TTI;
  const LoopVectorizationLegality *Legal;
  const Loop *TheLoop;
  const Function &F;
  PredicatedScalarEvolution &PSE;
  DemandedBits *DB;
  OptimizationRemarkEmitter *ORE;
  const LoopVectorizeHints *Hints;

  /// Cached result of isScalableVectorizationAllowed.
  std::optional<bool> IsScalableVectorizationAllowed;

```
- EN: Core entities appearing here include FixedScalableVFPair, FixedVF, VFSelectionContext, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as DemandedBits, OptimizationRemarkEmitter, ScalarEvolution, TargetTransformInfo, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 FixedScalableVFPair, FixedVF, VFSelectionContext，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 DemandedBits, OptimizationRemarkEmitter, ScalarEvolution, TargetTransformInfo 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 576-656

```cpp
  /// Used to store the value of vscale used for tuning the cost model. It is
  /// initialized during object construction.
  std::optional<unsigned> VScaleForTuning;

  /// The highest VF possible for this loop, without using MaxBandwidth.
  FixedScalableVFPair MaxPermissibleVFWithoutMaxBW;

  /// All element types found in the loop.
  SmallPtrSet<Type *, 16> ElementTypesInLoop;

  /// PHINodes of the reductions that should be expanded in-loop. Set by
  /// collectInLoopReductions.
  SmallPtrSet<PHINode *, 4> InLoopReductions;

  /// A Map of inloop reduction operations and their immediate chain operand.
  /// FIXME: This can be removed once reductions can be costed correctly in
  /// VPlan. This was added to allow quick lookup of the inloop operations.
  /// Set by collectInLoopReductions.
  DenseMap<Instruction *, Instruction *> InLoopReductionImmediateChains;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// that need to be vectorized. We ignore values that remain scalar such as
  /// 64 bit loop indices.
  std::pair<unsigned, unsigned> getSmallestAndWidestTypes() const;

  /// \return An upper bound for the vectorization factors for both
  /// fixed and scalable vectorization, where the minimum-known number of
  /// elements is a power-of-2 larger than zero. If scalable vectorization is
  /// disabled or unsupported, then the scalable part will be equal to
  /// ElementCount::getScalable(0). Also sets MaxSafeElements.
  FixedScalableVFPair computeFeasibleMaxVF(unsigned MaxTripCount,
                                           ElementCount UserVF, unsigned UserIC,
                                           bool FoldTailByMasking,
                                           bool RequiresScalarEpilogue);

```
- EN: Core entities appearing here include OptForSize, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as DemandedBits, OptimizationRemarkEmitter, ScalarEvolution, TargetTransformInfo, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 OptForSize，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 DemandedBits, OptimizationRemarkEmitter, ScalarEvolution, TargetTransformInfo 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 657-736

```cpp
  /// Return maximum safe number of elements to be processed per vector
  /// iteration, which do not prevent store-load forwarding and are safe with
  /// regard to the memory dependencies. Required for EVL-based VPlans to
  /// correctly calculate AVL (application vector length) as min(remaining AVL,
  /// MaxSafeElements). Set by computeFeasibleMaxVF.
  /// TODO: need to consider adjusting cost model to use this value as a
  /// vectorization factor for EVL-based vectorization.
  std::optional<unsigned> getMaxSafeElements() const { return MaxSafeElements; }

  /// Returns true if we should use strict in-order reductions for the given
  /// RdxDesc. This is true if the -enable-strict-reductions flag is passed,
  /// the IsOrdered flag of RdxDesc is set and we do not allow reordering
  /// of FP operations.
  bool useOrderedReductions(const RecurrenceDescriptor &RdxDesc) const;

  /// Returns true if the target machine supports masked loads or stores
  /// for \p I's data type and alignment. The caller must ensure the access is
  /// consecutive or part of an interleave group.
  bool isLegalMaskedLoadOrStore(Instruction *I, ElementCount VF) const;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// Legality checks.
class LoopVectorizationPlanner {
  /// The loop that we evaluate.
  Loop *OrigLoop;

  /// Loop Info analysis.
  LoopInfo *LI;

  /// The dominator tree.
  DominatorTree *DT;

  /// Target Library Info.
  const TargetLibraryInfo *TLI;

```
- EN: Core entities appearing here include isInLoopReduction, LoopVectorizationPlanner, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as DominatorTree, LoopInfo, TargetLibraryInfo, showing what later implementations must consult to reason about safety or cost.
- CN: 此处出现的核心实体包括 isInLoopReduction, LoopVectorizationPlanner，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 DominatorTree, LoopInfo, TargetLibraryInfo 等分析，表明后续实现需要依赖它们来判断安全性或代价。

### Lines 737-812

```cpp
  /// Target Transform Info.
  const TargetTransformInfo &TTI;

  /// The legality analysis.
  LoopVectorizationLegality *Legal;

  /// The profitability analysis.
  LoopVectorizationCostModel &CM;

  /// VF selection state independent of cost-modeling decisions.
  VFSelectionContext &Config;

  /// The interleaved access analysis.
  InterleavedAccessInfo &IAI;

  PredicatedScalarEvolution &PSE;

  const LoopVectorizeHints &Hints;

  OptimizationRemarkEmitter *ORE;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  VPlan &getPlanFor(ElementCount VF) const;

  /// Compute and return the most profitable vectorization factor and the
  /// corresponding best VPlan. Also collect all profitable VFs in
  /// ProfitableVFs.
  std::pair<VectorizationFactor, VPlan *> computeBestVF();

  /// \return The desired interleave count.
  /// If interleave count has been specified by metadata it will be returned.
  /// Otherwise, the interleave count is computed and returned. VF and LoopCost
  /// are the selected vectorization factor and the cost of the selected VF.
  unsigned selectInterleaveCount(VPlan &Plan, ElementCount VF,
                                 InstructionCost LoopCost);

```
- EN: The declarations explicitly reference analyses such as DominatorTree, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 这些声明显式涉及 DominatorTree, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 813-895

```cpp
  /// Generate the IR code for the vectorized loop captured in VPlan \p BestPlan
  /// according to the best selected \p VF and  \p UF.
  ///
  /// TODO: \p EpilogueVecKind should be removed once the re-use issue has been
  /// fixed.
  ///
  /// Returns a mapping of SCEVs to their expanded IR values.
  /// Note that this is a temporary workaround needed due to the current
  /// epilogue handling.
  enum class EpilogueVectorizationKind {
    None,     ///< Not part of epilogue vectorization.
    MainLoop, ///< Vectorizing the main loop of epilogue vectorization.
    Epilogue  ///< Vectorizing the epilogue loop.
  };
  DenseMap<const SCEV *, Value *>
  executePlan(ElementCount VF, unsigned UF, VPlan &BestPlan,
              InnerLoopVectorizer &LB, DominatorTree *DT,
              EpilogueVectorizationKind EpilogueVecKind =
                  EpilogueVectorizationKind::None);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      unsigned OrigLoopInvocationWeight, unsigned EstimatedVFxUF,
      bool DisableRuntimeUnroll);

private:
  /// Build a VPlan using VPRecipes according to the information gathered by
  /// Legal and VPlan-based analysis. For outer loops, performs basic recipe
  /// conversion only. For inner loops, \p Range's largest included VF is
  /// restricted to the maximum VF the returned VPlan is valid for. If no VPlan
  /// can be built for the input range, set the largest included VF to the
  /// maximum VF for which no plan could be built. Each VPlan is built starting
  /// from a copy of \p InitialPlan, which is a plain CFG VPlan wrapping the
  /// original scalar loop.
  VPlanPtr tryToBuildVPlan(VPlanPtr InitialPlan, VFRange &Range);

```
- EN: Core entities appearing here include EpilogueVectorizationKind, hasPlanWithVF, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as DominatorTree, OptimizationRemarkEmitter, SCEV, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 EpilogueVectorizationKind, hasPlanWithVF，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 DominatorTree, OptimizationRemarkEmitter, SCEV 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 896-929

```cpp
  /// Build VPlans for power-of-2 VF's between \p MinVF and \p MaxVF inclusive,
  /// according to the information gathered by Legal when it checked if it is
  /// legal to vectorize the loop.
  void buildVPlans(ElementCount MinVF, ElementCount MaxVF);

  /// Add ComputeReductionResult recipes to the middle block to compute the
  /// final reduction results. Add Select recipes to the latch block when
  /// folding tail, to feed ComputeReductionResult with the last or penultimate
  /// iteration values according to the header mask.
  void addReductionResultComputation(VPlanPtr &Plan,
                                     VPRecipeBuilder &RecipeBuilder,
                                     ElementCount MinVF);

  /// Returns true if the per-lane cost of VectorizationFactor A is lower than
  /// that of B.
  bool isMoreProfitable(const VectorizationFactor &A,
                        const VectorizationFactor &B, bool HasTail,
                        bool IsEpilogue = false) const;

  /// Returns true if the per-lane cost of VectorizationFactor A is lower than
  /// that of B in the context of vectorizing a loop with known \p MaxTripCount.
  bool isMoreProfitable(const VectorizationFactor &A,
                        const VectorizationFactor &B,
                        const unsigned MaxTripCount, bool HasTail,
                        bool IsEpilogue = false) const;

  /// Determines if we have the infrastructure to vectorize the loop and its
  /// epilogue, assuming the main loop is vectorized by \p MainPlan.
  bool isCandidateForEpilogueVectorization(VPlan &MainPlan) const;
};

} // namespace llvm

#endif // LLVM_TRANSFORMS_VECTORIZE_LOOPVECTORIZATIONPLANNER_H
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `GeneratedRTChecks, LoopInfo, DominatorTree, LoopVectorizationLegality, LoopVectorizationCostModel, PredicatedScalarEvolution, LoopVectorizeHints, RecurrenceDescriptor` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`GeneratedRTChecks, LoopInfo, DominatorTree, LoopVectorizationLegality, LoopVectorizationCostModel, PredicatedScalarEvolution, LoopVectorizeHints, RecurrenceDescriptor` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DemandedBits, DominatorTree, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, SCEV, TargetLibraryInfo, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DemandedBits, DominatorTree, LoopInfo, OptimizationRemarkEmitter, ScalarEvolution, SCEV, TargetLibraryInfo, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Header role: it mostly exposes declarations, contracts, inline helpers, and reusable utilities rather than driving a standalone transform by itself.
  - CN: 头文件角色：它主要暴露声明、约定、内联辅助函数和可复用工具，而不是独立执行某个完整变换。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: Utility headers: `llvm/Support/InstructionCost.h`, `llvm/ADT/SmallSet.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/InstructionCost.h`, `llvm/ADT/SmallSet.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlan.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlan.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DemandedBits`, `DominatorTree`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DemandedBits`, `DominatorTree`, `LoopInfo`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
