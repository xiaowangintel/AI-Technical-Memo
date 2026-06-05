# NaryReassociate.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/NaryReassociate.cpp` | `llvm/lib/Transforms/Scalar/NaryReassociate.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements reassociate n-ary expressions within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 NaryReassociate 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-56

```cpp
//===- NaryReassociate.cpp - Reassociate n-ary expressions ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass reassociates n-ary add expressions and eliminates the redundancy
// exposed by the reassociation.
//
// A motivating example:
//
//   void foo(int a, int b) {
//     bar(a + b);
//     bar((a + 2) + b);
//   }
//
// An ideal compiler should reassociate (a + 2) + b to (a + b) + 2 and simplify
// the above code to
//
//   int t = a + b;
//   bar(t);
//   bar(t + 2);
//
// However, the Reassociate pass is unable to do that because it processes each
// instruction individually and believes (a + 2) + b is the best form according
// to its rank system.
//
// To address this limitation, NaryReassociate reassociates an expression in a
// form that reuses existing instructions. As a result, NaryReassociate can
// reassociate (a + 2) + b in the example to (a + b) + 2 because it detects that
// (a + b) is computed before.
//
// NaryReassociate works as follows. For every instruction in the form of (a +
// b) + c, it checks whether a + c or b + c is already computed by a dominating
// instruction. If so, it then reassociates (a + b) + c into (a + c) + b or (b +
// c) + a and removes the redundancy accordingly. To efficiently look up whether
// an expression is computed before, we store each instruction seen and its SCEV
// into an SCEV-to-instruction map.
//
// Although the algorithm pattern-matches only ternary additions, it
// automatically handles many >3-ary expressions by walking through the function
// in the depth-first order. For example, given
//
//   (a + c) + d
//   ((a + b) + c) + d
//
// NaryReassociate first rewrites (a + b) + c to (a + c) + b, and then rewrites
// ((a + c) + b) + d into ((a + c) + d) + b.
//
// Finally, the above dominator-based algorithm may need to be run multiple
// iterations before emitting optimal code. One source of this need is that we
// only split an operand when it is used only once. The above algorithm can
// eliminate an instruction and decrease the usage count of its operands. As a
// result, an instruction that previously had multiple uses may become a
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 57-113

```cpp
// single-use instruction and thus eligible for split consideration. For
// example,
//
//   ac = a + c
//   ab = a + b
//   abc = ab + c
//   ab2 = ab + b
//   ab2c = ab2 + c
//
// In the first iteration, we cannot reassociate abc to ac+b because ab is used
// twice. However, we can reassociate ab2c to abc+b in the first iteration. As a
// result, ab2 becomes dead and ab will be used only once in the second
// iteration.
//
// Limitations and TODO items:
//
// 1) We only considers n-ary adds and muls for now. This should be extended
// and generalized.
//
//===----------------------------------------------------------------------===//
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
#include "llvm/IR/PatternMatch.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Value.h"
#include "llvm/IR/ValueHandle.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"
#include <cassert>
#include <cstdint>

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, ScalarEvolution, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, ScalarEvolution, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 114-172

```cpp
using namespace llvm;
using namespace PatternMatch;

#define DEBUG_TYPE "nary-reassociate"

namespace {

class NaryReassociateLegacyPass : public FunctionPass {
public:
  static char ID;

  NaryReassociateLegacyPass() : FunctionPass(ID) {
    initializeNaryReassociateLegacyPassPass(*PassRegistry::getPassRegistry());
  }

  bool doInitialization(Module &M) override {
    return false;
  }

  bool runOnFunction(Function &F) override;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
INITIALIZE_PASS_DEPENDENCY(TargetLibraryInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_END(NaryReassociateLegacyPass, "nary-reassociate",
                    "Nary reassociation", false, false)

FunctionPass *llvm::createNaryReassociatePass() {
  return new NaryReassociateLegacyPass();
}

bool NaryReassociateLegacyPass::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include NaryReassociateLegacyPass, runOnFunction, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 NaryReassociateLegacyPass, runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 173-232

```cpp
  auto *AC = &getAnalysis<AssumptionCacheTracker>().getAssumptionCache(F);
  auto *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  auto *SE = &getAnalysis<ScalarEvolutionWrapperPass>().getSE();
  auto *TLI = &getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(F);
  auto *TTI = &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);

  return Impl.runImpl(F, AC, DT, SE, TLI, TTI);
}

PreservedAnalyses NaryReassociatePass::run(Function &F,
                                           FunctionAnalysisManager &AM) {
  auto *AC = &AM.getResult<AssumptionAnalysis>(F);
  auto *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  auto *SE = &AM.getResult<ScalarEvolutionAnalysis>(F);
  auto *TLI = &AM.getResult<TargetLibraryAnalysis>(F);
  auto *TTI = &AM.getResult<TargetIRAnalysis>(F);

  if (!runImpl(F, AC, DT, SE, TLI, TTI))
    return PreservedAnalyses::all();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  bool Changed = false;
  SeenExprs.clear();
  // Process the basic blocks in a depth first traversal of the dominator
  // tree. This order ensures that all bases of a candidate are in Candidates
  // when we process it.
  SmallVector<WeakTrackingVH, 16> DeadInsts;
  for (const auto Node : depth_first(DT)) {
    BasicBlock *BB = Node->getBlock();
    for (Instruction &OrigI : *BB) {
      SCEVUse OrigSCEV = nullptr;
      if (Instruction *NewI = tryReassociate(&OrigI, OrigSCEV)) {
        Changed = true;
        OrigI.replaceAllUsesWith(NewI);

```
- EN: Core entities appearing here include doOneIteration, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 doOneIteration，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 233-294

```cpp
        // Add 'OrigI' to the list of dead instructions.
        DeadInsts.push_back(WeakTrackingVH(&OrigI));
        // Add the rewritten instruction to SeenExprs; the original
        // instruction is deleted.
        SCEVUse NewSCEV = SE->getSCEV(NewI);
        SeenExprs[NewSCEV].push_back(WeakTrackingVH(NewI));

        // Ideally, NewSCEV should equal OldSCEV because tryReassociate(I)
        // is equivalent to I. However, ScalarEvolution::getSCEV may
        // weaken nsw causing NewSCEV not to equal OldSCEV. For example,
        // suppose we reassociate
        //   I = &a[sext(i +nsw j)] // assuming sizeof(a[0]) = 4
        // to
        //   NewI = &a[sext(i)] + sext(j).
        //
        // ScalarEvolution computes
        //   getSCEV(I)    = a + 4 * sext(i + j)
        //   getSCEV(newI) = a + 4 * sext(i) + 4 * sext(j)
        // which are different SCEVs.
        //
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      MaxMin_match<ICmpInst, match_bind<Value>, match_bind<Value>, PredT>(
          m_Value(LHS), m_Value(RHS));
  if (match(I, MinMaxMatcher)) {
    OrigSCEV = SE->getSCEV(I);
    if (auto *NewMinMax = dyn_cast_or_null<Instruction>(
            tryReassociateMinOrMax(I, MinMaxMatcher, LHS, RHS)))
      return NewMinMax;
    if (auto *NewMinMax = dyn_cast_or_null<Instruction>(
            tryReassociateMinOrMax(I, MinMaxMatcher, RHS, LHS)))
      return NewMinMax;
  }
  return nullptr;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 295-351

```cpp
Instruction *NaryReassociatePass::tryReassociate(Instruction *I,
                                                 SCEVUse &OrigSCEV) {

  if (!SE->isSCEVable(I->getType()))
    return nullptr;

  switch (I->getOpcode()) {
  case Instruction::Add:
  case Instruction::Mul:
    OrigSCEV = SE->getSCEV(I);
    return tryReassociateBinaryOp(cast<BinaryOperator>(I));
  case Instruction::GetElementPtr:
    OrigSCEV = SE->getSCEV(I);
    return tryReassociateGEP(cast<GetElementPtrInst>(I));
  default:
    break;
  }

  // Try to match signed/unsigned Min/Max.
  Instruction *ResI = nullptr;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return nullptr;

  gep_type_iterator GTI = gep_type_begin(*GEP);
  for (unsigned I = 1, E = GEP->getNumOperands(); I != E; ++I, ++GTI) {
    if (GTI.isSequential()) {
      if (auto *NewGEP = tryReassociateGEPAtIndex(GEP, I - 1,
                                                  GTI.getIndexedType())) {
        return NewGEP;
      }
    }
  }
  return nullptr;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV, TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV, TargetTransformInfo 等分析结果。

### Lines 352-407

```cpp
bool NaryReassociatePass::requiresSignExtension(Value *Index,
                                                GetElementPtrInst *GEP) {
  unsigned IndexSizeInBits =
      DL->getIndexSizeInBits(GEP->getType()->getPointerAddressSpace());
  return cast<IntegerType>(Index->getType())->getBitWidth() < IndexSizeInBits;
}

GetElementPtrInst *
NaryReassociatePass::tryReassociateGEPAtIndex(GetElementPtrInst *GEP,
                                              unsigned I, Type *IndexedType) {
  SimplifyQuery SQ(*DL, DT, AC, GEP);
  Value *IndexToSplit = GEP->getOperand(I + 1);
  if (SExtInst *SExt = dyn_cast<SExtInst>(IndexToSplit)) {
    IndexToSplit = SExt->getOperand(0);
  } else if (ZExtInst *ZExt = dyn_cast<ZExtInst>(IndexToSplit)) {
    // zext can be treated as sext if the source is non-negative.
    if (isKnownNonNegative(ZExt->getOperand(0), SQ))
      IndexToSplit = ZExt->getOperand(0);
  }

  if (AddOperator *AO = dyn_cast<AddOperator>(IndexToSplit)) {
    // If the I-th index needs sext and the underlying add is not equipped with
    // nsw, we cannot split the add because
    //   sext(LHS + RHS) != sext(LHS) + sext(RHS).
    if (requiresSignExtension(IndexToSplit, GEP) &&
        computeOverflowForSignedAdd(AO, SQ) != OverflowResult::NeverOverflows)
      return nullptr;

    Value *LHS = AO->getOperand(0), *RHS = AO->getOperand(1);
    // IndexToSplit = LHS + RHS.
    if (auto *NewGEP = tryReassociateGEPAtIndex(GEP, I, LHS, RHS, IndexedType))
      return NewGEP;
    // Symmetrically, try IndexToSplit = RHS + LHS.
    if (LHS != RHS) {
      if (auto *NewGEP =
              tryReassociateGEPAtIndex(GEP, I, RHS, LHS, IndexedType))
        return NewGEP;
    }
  }
  return nullptr;
}

GetElementPtrInst *
NaryReassociatePass::tryReassociateGEPAtIndex(GetElementPtrInst *GEP,
                                              unsigned I, Value *LHS,
                                              Value *RHS, Type *IndexedType) {
  // Look for GEP's closest dominator that has the same SCEV as GEP except that
  // the I-th index is replaced with LHS.
  SmallVector<SCEVUse, 4> IndexExprs;
  for (Use &Index : GEP->indices())
    IndexExprs.push_back(SE->getSCEV(Index));
  // Replace the I-th index with LHS.
  IndexExprs[I] = SE->getSCEV(LHS);
  Type *GEPArgType = SE->getEffectiveSCEVType(GEP->getOperand(I)->getType());
  Type *LHSType = SE->getEffectiveSCEVType(LHS->getType());
  size_t LHSSize = DL->getTypeSizeInBits(LHSType).getFixedValue();
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 408-462

```cpp
  size_t GEPArgSize = DL->getTypeSizeInBits(GEPArgType).getFixedValue();
  if (isKnownNonNegative(LHS, SimplifyQuery(*DL, DT, AC, GEP)) &&
      LHSSize < GEPArgSize) {
    // Zero-extend LHS if it is non-negative. InstCombine canonicalizes sext to
    // zext if the source operand is proved non-negative. We should do that
    // consistently so that CandidateExpr more likely appears before. See
    // @reassociate_gep_assume for an example of this canonicalization.
    IndexExprs[I] = SE->getZeroExtendExpr(IndexExprs[I], GEPArgType);
  }
  SCEVUse CandidateExpr = SE->getGEPExpr(cast<GEPOperator>(GEP), IndexExprs);

  Value *Candidate = findClosestMatchingDominator(CandidateExpr, GEP);
  if (Candidate == nullptr)
    return nullptr;

  IRBuilder<> Builder(GEP);
  // Candidate should have the same pointer type as GEP.
  assert(Candidate->getType() == GEP->getType());

  // NewGEP = (char *)Candidate + RHS * sizeof(IndexedType)
  uint64_t IndexedSize = DL->getTypeAllocSize(IndexedType);
  Type *ElementType = GEP->getResultElementType();
  uint64_t ElementSize = DL->getTypeAllocSize(ElementType);
  // Another less rare case: because I is not necessarily the last index of the
  // GEP, the size of the type at the I-th index (IndexedSize) is not
  // necessarily divisible by ElementSize. For example,
  //
  // #pragma pack(1)
  // struct S {
  //   int a[3];
  //   int64 b[8];
  // };
  // #pragma pack()
  //
  // sizeof(S) = 100 is indivisible by sizeof(int64) = 8.
  //
  // TODO: bail out on this case for now. We could emit uglygep.
  if (IndexedSize % ElementSize != 0)
    return nullptr;

  // NewGEP = &Candidate[RHS * (sizeof(IndexedType) / sizeof(Candidate[0])));
  Type *PtrIdxTy = DL->getIndexType(GEP->getType());
  if (RHS->getType() != PtrIdxTy)
    RHS = Builder.CreateSExtOrTrunc(RHS, PtrIdxTy);
  if (IndexedSize != ElementSize) {
    RHS = Builder.CreateMul(
        RHS, ConstantInt::get(PtrIdxTy, IndexedSize / ElementSize));
  }
  GetElementPtrInst *NewGEP = cast<GetElementPtrInst>(
      Builder.CreateGEP(GEP->getResultElementType(), Candidate, RHS));
  NewGEP->setIsInBounds(GEP->isInBounds());
  NewGEP->takeName(GEP);
  return NewGEP;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 463-523

```cpp
Instruction *NaryReassociatePass::tryReassociateBinaryOp(BinaryOperator *I) {
  Value *LHS = I->getOperand(0), *RHS = I->getOperand(1);
  // There is no need to reassociate 0.
  if (SE->getSCEV(I)->isZero())
    return nullptr;
  if (auto *NewI = tryReassociateBinaryOp(LHS, RHS, I))
    return NewI;
  if (auto *NewI = tryReassociateBinaryOp(RHS, LHS, I))
    return NewI;
  return nullptr;
}

Instruction *NaryReassociatePass::tryReassociateBinaryOp(Value *LHS, Value *RHS,
                                                         BinaryOperator *I) {
  Value *A = nullptr, *B = nullptr;
  // To be conservative, we reassociate I only when it is the only user of (A op
  // B).
  if (LHS->hasOneUse() && matchTernaryOp(I, LHS, A, B)) {
    // I = (A op B) op RHS
    //   = (A op RHS) op B or (B op RHS) op A
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case Instruction::Add:
    NewI = BinaryOperator::CreateAdd(LHS, RHS, "", I->getIterator());
    break;
  case Instruction::Mul:
    NewI = BinaryOperator::CreateMul(LHS, RHS, "", I->getIterator());
    break;
  default:
    llvm_unreachable("Unexpected instruction.");
  }
  NewI->setDebugLoc(I->getDebugLoc());
  NewI->takeName(I);
  return NewI;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 524-585

```cpp
bool NaryReassociatePass::matchTernaryOp(BinaryOperator *I, Value *V,
                                         Value *&Op1, Value *&Op2) {
  switch (I->getOpcode()) {
  case Instruction::Add:
    return match(V, m_Add(m_Value(Op1), m_Value(Op2)));
  case Instruction::Mul:
    return match(V, m_Mul(m_Value(Op1), m_Value(Op2)));
  default:
    llvm_unreachable("Unexpected instruction.");
  }
  return false;
}

SCEVUse NaryReassociatePass::getBinarySCEV(BinaryOperator *I, SCEVUse LHS,
                                           SCEVUse RHS) {
  switch (I->getOpcode()) {
  case Instruction::Add:
    return SE->getAddExpr(LHS, RHS);
  case Instruction::Mul:
    return SE->getMulExpr(LHS, RHS);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      SmallVector<Instruction *> DropPoisonGeneratingInsts;
      if (!SE->canReuseInstruction(CandidateExpr, CandidateInstruction,
                                   DropPoisonGeneratingInsts))
        continue;

      for (Instruction *I : DropPoisonGeneratingInsts)
        I->dropPoisonGeneratingAnnotations();

      return CandidateInstruction;
    }
  }
  return nullptr;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 586-642

```cpp
template <typename MaxMinT> static SCEVTypes convertToSCEVype(MaxMinT &MM) {
  if (std::is_same_v<smax_pred_ty, typename MaxMinT::PredType>)
    return scSMaxExpr;
  else if (std::is_same_v<umax_pred_ty, typename MaxMinT::PredType>)
    return scUMaxExpr;
  else if (std::is_same_v<smin_pred_ty, typename MaxMinT::PredType>)
    return scSMinExpr;
  else if (std::is_same_v<umin_pred_ty, typename MaxMinT::PredType>)
    return scUMinExpr;

  llvm_unreachable("Can't convert MinMax pattern to SCEV type");
  return scUnknown;
}

// Parameters:
//  I - instruction matched by MaxMinMatch matcher
//  MaxMinMatch - min/max idiom matcher
//  LHS - first operand of I
//  RHS - second operand of I
template <typename MaxMinT>
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Instruction *R1MinMax = findClosestMatchingDominator(R1Expr, I);

    if (!R1MinMax)
      return nullptr;

    LLVM_DEBUG(dbgs() << "NARY: Found common sub-expr: " << *R1MinMax << "\n");

    SmallVector<SCEVUse, 2> Ops2{SE->getUnknown(C), SE->getUnknown(R1MinMax)};
    SCEVUse R2Expr = SE->getMinMaxExpr(SCEVType, Ops2);

    SCEVExpander Expander(*SE, "nary-reassociate");
    Value *NewMinMax = Expander.expandCodeFor(R2Expr, I->getType(), I);
    NewMinMax->setName(Twine(I->getName()).concat(".nary"));

```
- EN: Core entities appearing here include e, any_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 e, any_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 643-665

```cpp
    LLVM_DEBUG(dbgs() << "NARY: Deleting:  " << *I << "\n"
                      << "NARY: Inserting: " << *NewMinMax << "\n");
    return NewMinMax;
  };

  SCEVUse AExpr = SE->getSCEV(A);
  SCEVUse BExpr = SE->getSCEV(B);
  SCEVUse RHSExpr = SE->getSCEV(RHS);

  if (BExpr != RHSExpr) {
    // Try (A op RHS) op B
    if (auto *NewMinMax = tryCombination(A, AExpr, RHS, RHSExpr, B, BExpr))
      return NewMinMax;
  }

  if (AExpr != RHSExpr) {
    // Try (RHS op B) op A
    if (auto *NewMinMax = tryCombination(RHS, RHSExpr, B, BExpr, A, AExpr))
      return NewMinMax;
  }

  return nullptr;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `NaryReassociateLegacyPass, runOnFunction, doOneIteration, e, any_of` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`NaryReassociateLegacyPass, runOnFunction, doOneIteration, e, any_of` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, ScalarEvolution, SCEV, TargetLibraryInfo, TargetTransformInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, ScalarEvolution, SCEV, TargetLibraryInfo, TargetTransformInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/ErrorHandling.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/ErrorHandling.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstdint` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstdint` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo`, `TargetTransformInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo`, `TargetTransformInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
