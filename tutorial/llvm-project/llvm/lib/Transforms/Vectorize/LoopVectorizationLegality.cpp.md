# LoopVectorizationLegality.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/LoopVectorizationLegality.cpp` | `llvm/lib/Transforms/Vectorize/LoopVectorizationLegality.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Loop Vectorization Legality within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 LoopVectorizationLegality 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-140

```cpp
//===- LoopVectorizationLegality.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides loop vectorization legality analysis. Original code
// resided in LoopVectorize.cpp for a long time.
//
// At this point, it is implemented as a utility class, not as an analysis
// pass. It should be easy to create an analysis pass around it if there
// is a need (but D45420 needs to happen first).
//

#include "llvm/Transforms/Vectorize/LoopVectorizationLegality.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/Loads.h"
#include "llvm/Analysis/LoopInfo.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (VectorizerParams::isInterleaveForced())
    Interleave.Value = VectorizerParams::VectorizationInterleave;

  // If the metadata doesn't explicitly specify whether to enable scalable
  // vectorization, then decide based on the following criteria (increasing
  // level of priority):
  //  - Target default
  //  - Metadata width
  //  - Force option (always overrides)
  if ((LoopVectorizeHints::ScalableForceKind)Scalable.Value == SK_Unspecified) {
    if (TTI)
      Scalable.Value = TTI->enableScalableVectorization() ? SK_PreferScalable
                                                          : SK_FixedWidthOnly;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 141-273

```cpp
    if (Width.Value)
      // If the width is set, but the metadata says nothing about the scalable
      // property, then assume it concerns only a fixed-width UserVF.
      // If width is not set, the flag takes precedence.
      Scalable.Value = SK_FixedWidthOnly;
  }

  // If the flag is set to force any use of scalable vectors, override the loop
  // hints.
  if (ForceScalableVectorization.getValue() !=
      LoopVectorizeHints::SK_Unspecified)
    Scalable.Value = ForceScalableVectorization.getValue();

  // Scalable vectorization is disabled if no preference is specified.
  if ((LoopVectorizeHints::ScalableForceKind)Scalable.Value == SK_Unspecified)
    Scalable.Value = SK_FixedWidthOnly;

  if (IsVectorized.Value != 1)
    // If the vectorization width and interleaving count are both 1 then
    // consider the loop to have been already vectorized because there's
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (getForce() == LoopVectorizeHints::FK_Undefined && getWidth().isZero())
    return LV_NAME;
  return OptimizationRemarkAnalysis::AlwaysPrint;
}

bool LoopVectorizeHints::allowReordering() const {
  // Allow the vectorizer to change the order of operations if enabling
  // loop hints are provided
  ElementCount EC = getWidth();
  return HintsAllowReordering &&
         (getForce() == LoopVectorizeHints::FK_Enabled ||
          EC.getKnownMinValue() > 1);
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include setAlreadyVectorized, emitRemarkWithHints, allowReordering, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 setAlreadyVectorized, emitRemarkWithHints, allowReordering，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 274-408

```cpp
void LoopVectorizeHints::getHintsFromMetadata() {
  MDNode *LoopID = TheLoop->getLoopID();
  if (!LoopID)
    return;

  // First operand should refer to the loop id itself.
  assert(LoopID->getNumOperands() > 0 && "requires at least one operand");
  assert(LoopID->getOperand(0) == LoopID && "invalid loop id");

  for (const MDOperand &MDO : llvm::drop_begin(LoopID->operands())) {
    const MDString *S = nullptr;
    SmallVector<Metadata *, 4> Args;

    // The expected hint is either a MDString or a MDNode with the first
    // operand a MDString.
    if (const MDNode *MD = dyn_cast<MDNode>(MDO)) {
      if (!MD || MD->getNumOperands() == 0)
        continue;
      S = dyn_cast<MDString>(MD->getOperand(0));
      for (unsigned Idx = 1; Idx < MD->getNumOperands(); ++Idx)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return true;
}

// Return true if \p Lp and all its nested loops are uniform with regard to \p
// OuterLp.
static bool isUniformLoopNest(Loop *Lp, Loop *OuterLp) {
  if (!isUniformLoop(Lp, OuterLp))
    return false;

  // Check if nested loops are uniform.
  for (Loop *SubLp : *Lp)
    if (!isUniformLoopNest(SubLp, OuterLp))
      return false;

```
- EN: Core entities appearing here include getHintsFromMetadata, setHint, isUniformLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopNest. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getHintsFromMetadata, setHint, isUniformLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopNest 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 409-544

```cpp
  return true;
}

static IntegerType *getInductionIntegerTy(const DataLayout &DL, Type *Ty) {
  assert(Ty->isIntOrPtrTy() && "Expected integer or pointer type");

  if (Ty->isPointerTy())
    return DL.getIntPtrType(Ty->getContext(), Ty->getPointerAddressSpace());

  // It is possible that char's or short's overflow when we ask for the loop's
  // trip count, work around this by changing the type size.
  if (Ty->getScalarSizeInBits() < 32)
    return Type::getInt32Ty(Ty->getContext());

  return cast<IntegerType>(Ty);
}

static IntegerType *getWiderInductionTy(const DataLayout &DL, Type *Ty0,
                                        Type *Ty1) {
  IntegerType *TyA = getInductionIntegerTy(DL, Ty0);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    const SCEV *NewStep =
        SE.getMulExpr(Step, SE.getConstant(Ty, StepMultiplier));
    const SCEV *ScaledOffset = SE.getMulExpr(Step, SE.getConstant(Ty, Offset));
    const SCEV *NewStart =
        SE.getAddExpr(Expr->getStart(), SCEVUse(ScaledOffset));
    return SE.getAddRecExpr(NewStart, NewStep, TheLoop, SCEV::FlagAnyWrap);
  }

  const SCEV *visit(const SCEV *S) {
    if (CannotAnalyze || SE.isLoopInvariant(S, TheLoop))
      return S;
    return SCEVRewriteVisitor<SCEVAddRecForUniformityRewriter>::visit(S);
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include isInvariant, SCEVAddRecForUniformityRewriter, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 isInvariant, SCEVAddRecForUniformityRewriter，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution, SCEV 等分析结果。

### Lines 545-680

```cpp
  const SCEV *visitUnknown(const SCEVUnknown *S) {
    if (SE.isLoopInvariant(S, TheLoop))
      return S;
    // The value could vary across iterations.
    CannotAnalyze = true;
    return S;
  }

  const SCEV *visitCouldNotCompute(const SCEVCouldNotCompute *S) {
    // Could not analyze the expression.
    CannotAnalyze = true;
    return S;
  }

  static const SCEV *rewrite(const SCEV *S, ScalarEvolution &SE,
                             unsigned StepMultiplier, unsigned Offset,
                             Loop *TheLoop) {
    /// Bail out if the expression does not contain an UDiv expression.
    /// Uniform values which are not loop invariant require operations to strip
    /// out the lowest bits. For now just look for UDivs and use it to avoid
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Check whether inner loops are uniform. At this point, we only support
  // simple outer loops scenarios with uniform nested loops.
  if (!isUniformLoopNest(TheLoop /*loop nest*/,
                         TheLoop /*context outer loop*/)) {
    reportVectorizationFailure("Outer loop contains divergent loops",
        "loop control flow is not understood by vectorizer",
        "CFGNotUnderstood", ORE, TheLoop);
    if (DoExtraAnalysis)
      Result = false;
    else
      return false;
  }

```
- EN: Core entities appearing here include isUniform, canVectorizeOuterLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopNest, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isUniform, canVectorizeOuterLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopNest, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 681-830

```cpp
  // Check whether we are able to set up outer loop induction.
  if (!setupOuterLoopInductions()) {
    reportVectorizationFailure("Unsupported outer loop Phi(s)",
                               "UnsupportedPhi", ORE, TheLoop);
    if (DoExtraAnalysis)
      Result = false;
    else
      return false;
  }

  return Result;
}

void LoopVectorizationLegality::addInductionPhi(
    PHINode *Phi, const InductionDescriptor &ID,
    SmallPtrSetImpl<Value *> &AllowedExit) {
  Inductions[Phi] = ID;

  // In case this induction also comes with casts that we know we can ignore
  // in the vectorized loop body, record them here. All casts could be recorded
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          "loop induction variable could not be identified",
          "NoInductionVariable", ORE, TheLoop);
      return false;
    }
    if (!WidestIndTy) {
      reportVectorizationFailure(
          "Did not find one integer induction var",
          "integer loop induction variable could not be identified",
          "NoIntegerInductionVariable", ORE, TheLoop);
      return false;
    }
    LLVM_DEBUG(dbgs() << "LV: Did not find one integer induction var.\n");
  }

```
- EN: Core entities appearing here include setupOuterLoopInductions, isTLIScalarize, canVectorizeInstrs, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, SCEV, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 setupOuterLoopInductions, isTLIScalarize, canVectorizeInstrs，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, SCEV, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 831-963

```cpp
  // Now we know the widest induction type, check if our found induction
  // is the same size. If it's not, unset it here and InnerLoopVectorizer
  // will create another.
  if (PrimaryInduction && WidestIndTy != PrimaryInduction->getType())
    PrimaryInduction = nullptr;

  return Result;
}

bool LoopVectorizationLegality::canVectorizeInstr(Instruction &I) {
  BasicBlock *BB = I.getParent();
  BasicBlock *Header = TheLoop->getHeader();

  if (auto *Phi = dyn_cast<PHINode>(&I)) {
    Type *PhiTy = Phi->getType();
    // Check that this PHI type is allowed.
    if (!PhiTy->isIntegerTy() && !PhiTy->isFloatingPointTy() &&
        !PhiTy->isPointerTy()) {
      reportVectorizationFailure(
          "Found a non-int non-pointer PHI",
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //   * Have a vector version available.
  auto *CI = dyn_cast<CallInst>(&I);

  if (CI && !getVectorIntrinsicIDForCall(CI, TLI) &&
      !(CI->getCalledFunction() && TLI &&
        (!VFDatabase::getMappings(*CI).empty() || isTLIScalarize(*TLI, *CI)))) {
    // If the call is a recognized math libary call, it is likely that
    // we can vectorize it given loosened floating-point constraints.
    LibFunc Func;
    bool IsMathLibCall =
        TLI && CI->getCalledFunction() && CI->getType()->isFloatingPointTy() &&
        TLI->getLibFunc(CI->getCalledFunction()->getName(), Func) &&
        TLI->hasOptimizedCodeGen(Func);

```
- EN: Core entities appearing here include canVectorizeInstr, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canVectorizeInstr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 964-1097

```cpp
    if (IsMathLibCall) {
      // TODO: Ideally, we should not use clang-specific language here,
      // but it's hard to provide meaningful yet generic advice.
      // Also, should this be guarded by allowExtraAnalysis() and/or be part
      // of the returned info from isFunctionVectorizable()?
      reportVectorizationFailure(
          "Found a non-intrinsic callsite",
          "library call cannot be vectorized. "
          "Try compiling with -fno-math-errno, -ffast-math, "
          "or similar flags",
          "CantVectorizeLibcall", ORE, TheLoop, CI);
    } else {
      reportVectorizationFailure("Found a non-intrinsic callsite",
                                 "call instruction cannot be vectorized",
                                 "CantVectorizeLibcall", ORE, TheLoop, CI);
    }
    return false;
  }

  // Some intrinsics have scalar arguments and should be same in order for
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // same as outside the loop. Allowing the exit means reusing the SCEV
    // outside the loop.
    if (PSE.getPredicate().isAlwaysTrue()) {
      AllowedExit.insert(&I);
      return true;
    }
    reportVectorizationFailure("Value cannot be used outside the loop",
                               "ValueUsedOutsideLoop", ORE, TheLoop, &I);
    return false;
  }

  return true;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1098-1234

```cpp
/// Find histogram operations that match high-level code in loops:
/// \code
/// buckets[indices[i]]+=step;
/// \endcode
///
/// It matches a pattern starting from \p HSt, which Stores to the 'buckets'
/// array the computed histogram. It uses a BinOp to sum all counts, storing
/// them using a loop-variant index Load from the 'indices' input array.
///
/// On successful matches it updates the STATISTIC 'HistogramsDetected',
/// regardless of hardware support. When there is support, it additionally
/// stores the BinOp/Load pairs in \p HistogramCounts, as well the pointers
/// used to update histogram in \p HistogramPtrs.
static bool findHistogram(LoadInst *LI, StoreInst *HSt, Loop *TheLoop,
                          const PredicatedScalarEvolution &PSE,
                          SmallVectorImpl<HistogramInfo> &Histograms) {

  // Store value must come from a Binary Operation.
  Instruction *HPtrInstr = nullptr;
  BinaryOperator *HBinOp = nullptr;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  LLVM_DEBUG(dbgs() << "LV: Checking for a histogram on: " << *SI << "\n");
  return findHistogram(LI, SI, TheLoop, LAI->getPSE(), Histograms);
}

bool LoopVectorizationLegality::canVectorizeMemory() {
  LAI = &LAIs.getInfo(*TheLoop);
  const OptimizationRemarkAnalysis *LAR = LAI->getReport();
  if (LAR) {
    ORE->emit([&]() {
      return OptimizationRemarkAnalysis(Hints->vectorizeAnalysisPassName(),
                                        "loop not vectorized: ", *LAR);
    });
  }

```
- EN: Core entities appearing here include canVectorizeIndirectUnsafeDependences, canVectorizeMemory, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canVectorizeIndirectUnsafeDependences, canVectorizeMemory，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1235-1371

```cpp
  if (!LAI->canVectorizeMemory()) {
    if (hasUncountableExitWithSideEffects()) {
      reportVectorizationFailure(
          "Cannot vectorize unsafe dependencies in uncountable exit loop with "
          "side effects",
          "CantVectorizeUnsafeDependencyForEELoopWithSideEffects", ORE,
          TheLoop);
      return false;
    }

    return canVectorizeIndirectUnsafeDependences();
  }

  if (LAI->hasLoadStoreDependenceInvolvingLoopInvariantAddress()) {
    reportVectorizationFailure("We don't allow storing to uniform addresses",
                               "write to a loop invariant address could not "
                               "be vectorized",
                               "CantVectorizeStoreToLoopInvariantAddress", ORE,
                               TheLoop);
    return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // reduction operations in-loop.
  return (all_of(getReductionVars(), [&](auto &Reduction) -> bool {
    const RecurrenceDescriptor &RdxDesc = Reduction.second;
    return !RdxDesc.hasExactFPMath() || RdxDesc.isOrdered();
  }));
}

bool LoopVectorizationLegality::isInvariantStoreOfReduction(StoreInst *SI) {
  return any_of(getReductionVars(), [&](auto &Reduction) -> bool {
    const RecurrenceDescriptor &RdxDesc = Reduction.second;
    return RdxDesc.IntermediateStore == SI;
  });
}

```
- EN: Core entities appearing here include erase_if, any_of, isInvariantStoreOfReduction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 erase_if, any_of, isInvariantStoreOfReduction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1372-1508

```cpp
bool LoopVectorizationLegality::isInvariantAddressOfReduction(Value *V) {
  return any_of(getReductionVars(), [&](auto &Reduction) -> bool {
    const RecurrenceDescriptor &RdxDesc = Reduction.second;
    if (!RdxDesc.IntermediateStore)
      return false;

    ScalarEvolution *SE = PSE.getSE();
    Value *InvariantAddress = RdxDesc.IntermediateStore->getPointerOperand();
    return V == InvariantAddress ||
           SE->getSCEV(V) == SE->getSCEV(InvariantAddress);
  });
}

bool LoopVectorizationLegality::isInductionPhi(const Value *V) const {
  Value *In0 = const_cast<Value *>(V);
  PHINode *PN = dyn_cast_or_null<PHINode>(In0);
  if (!PN)
    return false;

  return Inductions.count(PN);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    reportVectorizationFailure("If-conversion is disabled",
                               "IfConversionDisabled", ORE, TheLoop);
    return false;
  }

  assert(TheLoop->getNumBlocks() > 1 && "Single block loops are vectorizable");

  // A list of pointers which are known to be dereferenceable within scope of
  // the loop body for each iteration of the loop which executes.  That is,
  // the memory pointed to can be dereferenced (with the access size implied by
  // the value's type) unconditionally within the loop header without
  // introducing a new fault.
  SmallPtrSet<Value *, 8> SafePointers;

```
- EN: Core entities appearing here include isInvariantAddressOfReduction, isInductionPhi, getIntOrFpInductionDescriptor, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopAccessInfo, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isInvariantAddressOfReduction, isInductionPhi, getIntOrFpInductionDescriptor，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopAccessInfo, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1509-1650

```cpp
  // Collect safe addresses.
  for (BasicBlock *BB : TheLoop->blocks()) {
    if (!blockNeedsPredication(BB)) {
      for (Instruction &I : *BB)
        if (auto *Ptr = getLoadStorePointerOperand(&I))
          SafePointers.insert(Ptr);
      continue;
    }

    // For a block which requires predication, a address may be safe to access
    // in the loop w/o predication if we can prove dereferenceability facts
    // sufficient to ensure it'll never fault within the loop. For the moment,
    // we restrict this to loads; stores are more complicated due to
    // concurrency restrictions.
    ScalarEvolution &SE = *PSE.getSE();
    SmallVector<const SCEVPredicate *, 4> Predicates;
    for (Instruction &I : *BB) {
      LoadInst *LI = dyn_cast<LoadInst>(&I);

      // Make sure we can execute all computations feeding into Ptr in the loop
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      return false;
  }

  // We must have a single backedge.
  if (Lp->getNumBackEdges() != 1) {
    reportVectorizationFailure("The loop must have a single backedge",
        "loop control flow is not understood by vectorizer",
        "CFGNotUnderstood", ORE, TheLoop);
    if (DoExtraAnalysis)
      Result = false;
    else
      return false;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopNest, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopNest, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1651-1787

```cpp
  // The latch must be terminated by a branch.
  BasicBlock *Latch = Lp->getLoopLatch();
  if (Latch && !isa<UncondBrInst, CondBrInst>(Latch->getTerminator())) {
    reportVectorizationFailure(
        "The loop latch terminator is not a UncondBrInst/CondBrInst",
        "loop control flow is not understood by vectorizer", "CFGNotUnderstood",
        ORE, TheLoop);
    if (DoExtraAnalysis)
      Result = false;
    else
      return false;
  }

  return Result;
}

bool LoopVectorizationLegality::canVectorizeLoopNestCFG(
    Loop *Lp, bool UseVPlanNativePath) {
  // Store the result and return it at the end instead of exiting early, in case
  // allowExtraAnalysis is used to report multiple reasons for not vectorizing.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (I.mayWriteToMemory()) {
        if (isa<StoreInst>(&I) && cast<StoreInst>(&I)->isSimple()) {
          HasSideEffects = true;
          continue;
        }

        // We don't support complex writes to memory.
        reportVectorizationFailure(
            "Complex writes to memory unsupported in early exit loops",
            "Cannot vectorize early exit loop with complex writes to memory",
            "WritesInEarlyExitLoop", ORE, TheLoop);
        return false;
      }

```
- EN: Core entities appearing here include isVectorizableEarlyExitLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopNest, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isVectorizableEarlyExitLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopNest, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1788-1923

```cpp
      if (!IsSafeOperation(&I)) {
        reportVectorizationFailure("Early exit loop contains operations that "
                                   "cannot be speculatively executed",
                                   "UnsafeOperationsEarlyExitLoop", ORE,
                                   TheLoop);
        return false;
      }
    }

  SmallVector<LoadInst *, 4> NonDerefLoads;
  // TODO: Handle loops that may fault.
  if (!HasSideEffects) {
    // Read-only loop.
    Predicates.clear();
    if (!isReadOnlyLoop(TheLoop, PSE.getSE(), DT, AC, NonDerefLoads,
                        &Predicates)) {
      reportVectorizationFailure(
          "Loop may fault", "Cannot vectorize non-read-only early exit loop",
          "NonReadOnlyEarlyExitLoop", ORE, TheLoop);
      return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          AliasResult AR = AA->alias(Ptr, SI->getPointerOperand());
          if (AR == AliasResult::NoAlias)
            continue;
        }

        reportVectorizationFailure(
            "Cannot determine whether critical uncountable exit load address "
            "does not alias with a memory write",
            "CantVectorizeAliasWithCriticalUncountableExitLoad", ORE, TheLoop);
        return false;
      }
    }
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include m_Value, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 m_Value，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 1924-2065

```cpp
  return true;
}

bool LoopVectorizationLegality::canVectorize(bool UseVPlanNativePath) {
  // Store the result and return it at the end instead of exiting early, in case
  // allowExtraAnalysis is used to report multiple reasons for not vectorizing.
  bool Result = true;

  bool DoExtraAnalysis = ORE->allowExtraAnalysis(DEBUG_TYPE);
  // Check whether the loop-related control flow in the loop nest is expected by
  // vectorizer.
  if (!canVectorizeLoopNestCFG(TheLoop, UseVPlanNativePath)) {
    if (DoExtraAnalysis) {
      LLVM_DEBUG(dbgs() << "LV: legality check failed: loop nest");
      Result = false;
    } else {
      return false;
    }
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

bool LoopVectorizationLegality::canFoldTailByMasking() const {
  // The only loops we can vectorize without a scalar epilogue, are loops with
  // a bottom-test and a single exiting block. We'd have to handle the fact
  // that not every instruction executes on the last iteration.  This will
  // require a lane mask which varies through the vector loop body.  (TODO)
  if (TheLoop->getExitingBlock() != TheLoop->getLoopLatch()) {
    LLVM_DEBUG(
        dbgs()
        << "LV: Cannot fold tail by masking. Requires a singe latch exit\n");
    return false;
  }

```
- EN: Core entities appearing here include canVectorize, canFoldTailByMasking, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopNest, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canVectorize, canFoldTailByMasking，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopNest, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2066-2100

```cpp
  LLVM_DEBUG(dbgs() << "LV: checking if tail can be folded by masking.\n");

  // The list of pointers that we can safely read and write to remains empty.
  SmallPtrSet<Value *, 8> SafePointers;

  // Check all blocks for predication, including those that ordinarily do not
  // need predication such as the header block.
  SmallPtrSet<const Instruction *, 8> TmpMaskedOp;
  for (BasicBlock *BB : TheLoop->blocks()) {
    if (!blockCanBePredicated(BB, SafePointers, TmpMaskedOp)) {
      LLVM_DEBUG(dbgs() << "LV: Cannot fold tail by masking.\n");
      return false;
    }
  }

  LLVM_DEBUG(dbgs() << "LV: can fold tail by masking.\n");

  return true;
}

void LoopVectorizationLegality::prepareToFoldTailByMasking() {
  // The list of pointers that we can safely read and write to remains empty.
  SmallPtrSet<Value *, 8> SafePointers;

  // Mark all blocks for predication, including those that ordinarily do not
  // need predication such as the header block, and collect instructions needing
  // predication in TailFoldedMaskedOp.
  for (BasicBlock *BB : TheLoop->blocks()) {
    [[maybe_unused]] bool R =
        blockCanBePredicated(BB, SafePointers, TailFoldedMaskedOp);
    assert(R && "Must be able to predicate block when tail-folding.");
  }
}

} // namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include prepareToFoldTailByMasking, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 prepareToFoldTailByMasking，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `SCEVAddRecForUniformityRewriter, validate, TheLoop, setAlreadyVectorized, emitRemarkWithHints, allowReordering, getHintsFromMetadata, setHint` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SCEVAddRecForUniformityRewriter, validate, TheLoop, setAlreadyVectorized, emitRemarkWithHints, allowReordering, getHintsFromMetadata, setHint` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AliasAnalysis, DataLayout, LoopAccessInfo, LoopInfo, LoopNest, OptimizationRemarkEmitter, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AliasAnalysis, DataLayout, LoopAccessInfo, LoopInfo, LoopNest, OptimizationRemarkEmitter, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnableIfConversion, AllowStridedPointerIVs, HintsAllowReordering, VectorizeSCEVCheckThreshold, PragmaVectorizeSCEVCheckThreshold` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnableIfConversion, AllowStridedPointerIVs, HintsAllowReordering, VectorizeSCEVCheckThreshold, PragmaVectorizeSCEVCheckThreshold` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MustExecute.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MustExecute.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h`, `llvm/Transforms/Utils/SizeOpts.h`, `llvm/Transforms/Vectorize/LoopVectorize.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h`, `llvm/Transforms/Utils/SizeOpts.h`, `llvm/Transforms/Vectorize/LoopVectorize.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `AliasAnalysis`, `DataLayout`, `LoopAccessInfo`, `LoopInfo`, `LoopNest`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AliasAnalysis`, `DataLayout`, `LoopAccessInfo`, `LoopInfo`, `LoopNest`, `OptimizationRemarkEmitter`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
