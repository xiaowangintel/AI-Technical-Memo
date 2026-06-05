# ScalarEvolutionExpander.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/ScalarEvolutionExpander.cpp` | `llvm/lib/Transforms/Utils/ScalarEvolutionExpander.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements scalar Evolution Analysis within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 ScalarEvolutionExpander 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-161

```cpp
//===- ScalarEvolutionExpander.cpp - Scalar Evolution Analysis ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the implementation of the scalar evolution expander,
// which is used to generate the code corresponding to a given scalar evolution
// expression.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/ScopeExit.h"
#include "llvm/Analysis/InstructionSimplify.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ScalarEvolutionPatternMatch.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  return Ret;
}

BasicBlock::iterator
SCEVExpander::findInsertPointAfter(Instruction *I,
                                   Instruction *MustDominate) const {
  BasicBlock::iterator IP = ++I->getIterator();
  if (auto *II = dyn_cast<InvokeInst>(I))
    IP = II->getNormalDest()->begin();

  while (isa<PHINode>(IP))
    ++IP;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 162-322

```cpp
  if (isa<FuncletPadInst>(IP) || isa<LandingPadInst>(IP)) {
    ++IP;
  } else if (isa<CatchSwitchInst>(IP)) {
    IP = MustDominate->getParent()->getFirstInsertionPt();
  } else {
    assert(!IP->isEHPad() && "unexpected eh pad!");
  }

  // Adjust insert point to be after instructions inserted by the expander, so
  // we can re-use already inserted instructions. Avoid skipping past the
  // original \p MustDominate, in case it is an inserted instruction.
  while (isInsertedInstruction(&*IP) && &*IP != MustDominate)
    ++IP;

  return IP;
}

void SCEVExpander::eraseDeadInstructions(Value *Root) {
  SmallVector<Value *> WorkList;
  SmallPtrSet<Value *, 8> DeletedValues;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        }
        // Conservatively, do not use any instruction which has any of exact
        // flags installed.
        if (isa<PossiblyExactOperator>(I) && I->isExact())
          return true;
        return false;
      };
      if (IP->getOpcode() == (unsigned)Opcode && IP->getOperand(0) == LHS &&
          IP->getOperand(1) == RHS && !canGenerateIncompatiblePoison(&*IP))
        return &*IP;
      if (IP == BlockBegin) break;
    }
  }

```
- EN: Core entities appearing here include eraseDeadInstructions, GetOptimalInsertionPointForCastOf, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 eraseDeadInstructions, GetOptimalInsertionPointForCastOf，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 323-498

```cpp
  // Save the original insertion point so we can restore it when we're done.
  DebugLoc Loc = Builder.GetInsertPoint()->getDebugLoc();
  SCEVInsertPointGuard Guard(Builder, this);

  if (IsSafeToHoist) {
    // Move the insertion point out of as many loops as we can.
    while (const Loop *L = SE.LI.getLoopFor(Builder.GetInsertBlock())) {
      if (!L->isLoopInvariant(LHS) || !L->isLoopInvariant(RHS)) break;
      BasicBlock *Preheader = L->getLoopPreheader();
      if (!Preheader) break;

      // Ok, move up a level.
      Builder.SetInsertPoint(Preheader->getTerminator());
    }
  }

  // If we haven't found this binop, insert it.
  Builder.SetCurrentDebugLocation(Loc);
  bool IsNUW = any(Flags & SCEV::FlagNUW);
  bool IsNSW = any(Flags & SCEV::FlagNSW);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
  case scUnknown: {
    const SCEVUnknown *U = cast<SCEVUnknown>(S);
    if (const Instruction *I = dyn_cast<Instruction>(U->getValue()))
      return Pair.first->second = SE.LI.getLoopFor(I->getParent());
    // A non-instruction has no relevant loops.
    return nullptr;
  }
  case scCouldNotCompute:
    llvm_unreachable("Attempt to use a SCEVCouldNotCompute object!");
  }
  llvm_unreachable("Unexpected SCEV type!");
}

```
- EN: Core entities appearing here include all_of, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DominatorTree, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 all_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DominatorTree, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 499-662

```cpp
namespace {

/// LoopCompare - Compare loops by PickMostRelevantLoop.
class LoopCompare {
  DominatorTree &DT;
public:
  explicit LoopCompare(DominatorTree &dt) : DT(dt) {}

  bool operator()(std::pair<const Loop *, const SCEV *> LHS,
                  std::pair<const Loop *, const SCEV *> RHS) const {
    // Keep pointer operands sorted at the end.
    if (LHS.second->getType()->isPointerTy() !=
        RHS.second->getType()->isPointerTy())
      return LHS.second->getType()->isPointerTy();

    // Compare loops with PickMostRelevantLoop.
    if (LHS.first != RHS.first)
      return PickMostRelevantLoop(LHS.first, RHS.first, DT) != LHS.first;

    // If one operand is a non-constant negative and the other is not,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      P = InsertBinop(Instruction::Mul, P, P, SCEV::FlagAnyWrap,
                      /*IsSafeToHoist*/ true);
      if (Exponent & BinExp)
        Result = Result ? InsertBinop(Instruction::Mul, Result, P,
                                      SCEV::FlagAnyWrap,
                                      /*IsSafeToHoist*/ true)
                        : P;
    }

    I = E;
    assert(Result && "Nothing was expanded?");
    return Result;
  };

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include LoopCompare, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 LoopCompare，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, SCEV 等分析结果。

### Lines 663-822

```cpp
  while (I != OpsAndLoops.end()) {
    if (!Prod) {
      // This is the first operand. Just expand it.
      Prod = ExpandOpBinPowN();
    } else if (I->second->isAllOnesValue()) {
      // Instead of doing a multiply by negative one, just do a negate.
      Prod = InsertBinop(Instruction::Sub, Constant::getNullValue(Ty), Prod,
                         SCEV::FlagAnyWrap, /*IsSafeToHoist*/ true);
      ++I;
    } else {
      // A simple mul.
      Value *W = ExpandOpBinPowN();
      // Canonicalize a constant to the RHS.
      if (isa<Constant>(Prod)) std::swap(Prod, W);
      const APInt *RHS;
      if (match(W, m_Power2(RHS))) {
        // Canonicalize Prod*(1<<C) to Prod<<C.
        assert(!Ty->isVectorTy() && "vector types are not SCEVable");
        auto NWFlags = S.getNoWrapFlags();
        // clear nsw flag if shl will produce poison value.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// 'I' is being moved.  If this fixup is not done and 'I' is moved to a
/// different block, the inconsistent insert point (with a mismatched
/// Instruction and Block) can lead to an instruction being inserted in a block
/// other than its parent.
void SCEVExpander::fixupInsertPoints(Instruction *I) {
  BasicBlock::iterator It(*I);
  BasicBlock::iterator NewInsertPt = std::next(It);
  if (Builder.GetInsertPoint() == It)
    Builder.SetInsertPoint(&*NewInsertPt);
  for (auto *InsertPtGuard : InsertPointGuards)
    if (InsertPtGuard->GetInsertPoint() == It)
      InsertPtGuard->SetInsertPoint(NewInsertPt);
}

```
- EN: Core entities appearing here include fixupInsertPoints, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 fixupInsertPoints，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 823-984

```cpp
/// hoistStep - Attempt to hoist a simple IV increment above InsertPos to make
/// it available to other uses in this loop. Recursively hoist any operands,
/// until we reach a value that dominates InsertPos.
bool SCEVExpander::hoistIVInc(Instruction *IncV, Instruction *InsertPos,
                              bool RecomputePoisonFlags) {
  auto FixupPoisonFlags = [this](Instruction *I) {
    // Drop flags that are potentially inferred from old context and infer flags
    // in new context.
    rememberFlags(I);
    I->dropPoisonGeneratingFlags();
    if (auto *OBO = dyn_cast<OverflowingBinaryOperator>(I))
      if (auto Flags = SE.getStrengthenedNoWrapFlagsFromBinOp(OBO)) {
        auto *BO = cast<BinaryOperator>(I);
        BO->setHasNoUnsignedWrap(
            ScalarEvolution::maskFlags(*Flags, SCEV::FlagNUW) == SCEV::FlagNUW);
        BO->setHasNoSignedWrap(
            ScalarEvolution::maskFlags(*Flags, SCEV::FlagNSW) == SCEV::FlagNSW);
      }
  };

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
static bool IsIncrementNUW(ScalarEvolution &SE, const SCEVAddRecExpr *AR) {
  if (!isa<IntegerType>(AR->getType()))
    return false;

  unsigned BitWidth = cast<IntegerType>(AR->getType())->getBitWidth();
  Type *WideTy = IntegerType::get(AR->getType()->getContext(), BitWidth * 2);
  const SCEV *Step = AR->getStepRecurrence(SE);
  const SCEV *OpAfterExtend = SE.getAddExpr(SE.getZeroExtendExpr(Step, WideTy),
                                            SE.getZeroExtendExpr(AR, WideTy));
  const SCEV *ExtendAfterOp =
    SE.getZeroExtendExpr(SE.getAddExpr(AR, Step), WideTy);
  return ExtendAfterOp == OpAfterExtend;
}

```
- EN: Core entities appearing here include IsIncrementNSW, IsIncrementNUW, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 IsIncrementNSW, IsIncrementNUW，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 985-1146

```cpp
/// getAddRecExprPHILiterally - Helper for expandAddRecExprLiterally. Expand
/// the base addrec, which is the addrec without any non-loop-dominating
/// values, and return the PHI.
PHINode *
SCEVExpander::getAddRecExprPHILiterally(const SCEVAddRecExpr *Normalized,
                                        const Loop *L, Type *&TruncTy,
                                        bool &InvertStep) {
  assert((!IVIncInsertLoop || IVIncInsertPos) &&
         "Uninitialized insert position");

  // Reuse a previously-inserted PHI, if present.
  BasicBlock *LatchBlock = L->getLoopLatch();
  if (LatchBlock) {
    PHINode *AddRecPhiMatch = nullptr;
    Instruction *IncV = nullptr;
    TruncTy = nullptr;
    InvertStep = false;

    // Only try partially matching scevs that need truncation and/or
    // step-inversion if we know this loop is outside the current loop.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Add a start value.
    if (!L->contains(Pred)) {
      PN->addIncoming(StartV, Pred);
      continue;
    }

    // Create a step value and add it to the PHI.
    // If IVIncInsertLoop is non-null and equal to the addrec's loop, insert the
    // instructions at IVIncInsertPos.
    Instruction *InsertPos = L == IVIncInsertLoop ?
      IVIncInsertPos : Pred->getTerminator();
    Builder.SetInsertPoint(InsertPos);
    Value *IncV = expandIVInc(PN, StepV, L, useSubtract);

```
- EN: Core entities appearing here include canBeCheaplyTransformed, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canBeCheaplyTransformed，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1147-1316

```cpp
    if (isa<OverflowingBinaryOperator>(IncV)) {
      if (IncrementIsNUW)
        cast<BinaryOperator>(IncV)->setHasNoUnsignedWrap();
      if (IncrementIsNSW)
        cast<BinaryOperator>(IncV)->setHasNoSignedWrap();
    }
    PN->addIncoming(IncV, Pred);
  }

  // After expanding subexpressions, restore the PostIncLoops set so the caller
  // can ensure that IVIncrement dominates the current uses.
  PostIncLoops = SavedPostIncLoops;

  // Remember this PHI, even in post-inc mode. LSR SCEV-based salvaging is most
  // effective when we are able to use an IV inserted here, so record it.
  InsertedValues.insert(PN);
  InsertedIVs.push_back(PN);
  return PN;
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      continue;

    assert(Diff->getType()->isIntegerTy() &&
           "difference must be of integer type");
    Value *DiffV = expand(Diff);
    Value *BaseV = fixupLCSSAFormFor(&PN);
    if (PhiTy->isPointerTy()) {
      if (STy->isPointerTy())
        return Builder.CreatePtrAdd(BaseV, DiffV);
      BaseV = Builder.CreatePtrToAddr(BaseV);
    }
    return Builder.CreateAdd(BaseV, DiffV);
  }

```
- EN: Core entities appearing here include expandAddRecExprLiterally, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 expandAddRecExprLiterally，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1317-1481

```cpp
  return nullptr;
}

Value *SCEVExpander::visitAddRecExpr(SCEVUseT<const SCEVAddRecExpr *> S) {
  // In canonical mode we compute the addrec as an expression of a canonical IV
  // using evaluateAtIteration and expand the resulting SCEV expression. This
  // way we avoid introducing new IVs to carry on the computation of the addrec
  // throughout the loop.
  //
  // For nested addrecs evaluateAtIteration might need a canonical IV of a
  // type wider than the addrec itself. Emitting a canonical IV of the
  // proper type might produce non-legal types, for example expanding an i64
  // {0,+,2,+,1} addrec would need an i65 canonical IV. To avoid this just fall
  // back to non-canonical mode for nested addrecs.
  if (!CanonicalMode || (S->getNumOperands() > 2))
    return expandAddRecExprLiterally(S);

  Type *Ty = SE.getEffectiveSCEVType(S->getType());
  const Loop *L = S->getLoop();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    BasicBlock::iterator BIP = Builder.GetInsertPoint();
    for (User *U : V->users()) {
      auto *CI = dyn_cast<CastInst>(U);
      if (CI && CI->getType() == Ty &&
          (CI->getOpcode() == CastInst::PtrToAddr ||
           CI->getOpcode() == CastInst::PtrToInt) &&
          &*BIP != CI && SE.DT.dominates(CI, &*BIP))
        return CI;
    }
  }
  return ReuseOrCreateCast(V, Ty, CastInst::PtrToAddr,
                           GetOptimalInsertionPointForCastOf(V));
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1482-1660

```cpp
Value *SCEVExpander::visitPtrToIntExpr(SCEVUseT<const SCEVPtrToIntExpr *> S) {
  Value *V = expand(S->getOperand());
  return ReuseOrCreateCast(V, S->getType(), CastInst::PtrToInt,
                           GetOptimalInsertionPointForCastOf(V));
}

Value *SCEVExpander::visitTruncateExpr(SCEVUseT<const SCEVTruncateExpr *> S) {
  Value *V = expand(S->getOperand());
  return Builder.CreateTrunc(V, S->getType());
}

Value *
SCEVExpander::visitZeroExtendExpr(SCEVUseT<const SCEVZeroExtendExpr *> S) {
  Value *V = expand(S->getOperand());
  return Builder.CreateZExt(V, S->getType(), "",
                            SE.isKnownNonNegative(S->getOperand()));
}

Value *
SCEVExpander::visitSignExtendExpr(SCEVUseT<const SCEVSignExtendExpr *> S) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          InsertPt = Preheader->getTerminator()->getIterator();
        } else {
          // LSR sets the insertion point for AddRec start/step values to the
          // block start to simplify value reuse, even though it's an invalid
          // position. SCEVExpander must correct for this in all cases.
          InsertPt = L->getHeader()->getFirstInsertionPt();
        }
      } else {
        // If the SCEV is computable at this level, insert it into the header
        // after the PHIs (and after any other instructions that we've inserted
        // there) so that it is guaranteed to dominate any user inside the loop.
        if (L && SE.hasComputableLoopEvolution(S, L) && !PostIncLoops.count(L))
          InsertPt = L->getHeader()->getFirstInsertionPt();

```
- EN: Core entities appearing here include visitZeroExtendExpr, visitSignExtendExpr, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitZeroExtendExpr, visitSignExtendExpr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1661-1833

```cpp
        while (InsertPt != Builder.GetInsertPoint() &&
               (isInsertedInstruction(&*InsertPt))) {
          InsertPt = std::next(InsertPt);
        }
        break;
      }
    }
  }

  // Check to see if we already expanded this here.
  auto I = InsertedExpressions.find(std::make_pair(S, &*InsertPt));
  if (I != InsertedExpressions.end())
    return I->second;

  SCEVInsertPointGuard Guard(Builder, this);
  Builder.SetInsertPoint(InsertPt->getParent(), InsertPt);

  // Expand the expression into instructions.
  SmallVector<Instruction *> DropPoisonGeneratingInsts;
  Value *V = FindValueInExprValueMap(S, &*InsertPt, DropPoisonGeneratingInsts);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// replaceCongruentIVs - Check for congruent phis in this loop header and
/// replace them with their most canonical representative. Return the number of
/// phis eliminated.
///
/// This does not depend on any SCEVExpander state but should be used in
/// the same context that SCEVExpander is used.
unsigned
SCEVExpander::replaceCongruentIVs(Loop *L, const DominatorTree *DT,
                                  SmallVectorImpl<WeakTrackingVH> &DeadInsts,
                                  const TargetTransformInfo *TTI) {
  // Find integer phis in order of increasing width.
  SmallVector<PHINode *, 8> Phis(
      llvm::make_pointer_range(L->getHeader()->phis()));

```
- EN: Core entities appearing here include rememberInstruction, rememberFlags, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 rememberInstruction, rememberFlags，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1834-2001

```cpp
  if (TTI)
    // Use stable_sort to preserve order of equivalent PHIs, so the order
    // of the sorted Phis is the same from run to run on the same loop.
    llvm::stable_sort(Phis, [](Value *LHS, Value *RHS) {
      // Put pointers at the back and make sure pointer < pointer = false.
      if (!LHS->getType()->isIntegerTy() || !RHS->getType()->isIntegerTy())
        return RHS->getType()->isIntegerTy() && !LHS->getType()->isIntegerTy();
      return RHS->getType()->getPrimitiveSizeInBits().getFixedValue() <
             LHS->getType()->getPrimitiveSizeInBits().getFixedValue();
    });

  unsigned NumElim = 0;
  DenseMap<const SCEV *, PHINode *> ExprToIVMap;
  // Process phis from wide to narrow. Map wide phis to their truncation
  // so narrow phis can reuse them.
  for (PHINode *Phi : Phis) {
    auto SimplifyPHINode = [&](PHINode *PN) -> Value * {
      if (Value *V = simplifyInstruction(PN, {DL, &SE.TLI, &SE.DT, &SE.AC}))
        return V;
      if (!SE.isSCEVable(PN->getType()))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Operations.emplace_back(Opcode, MinIdx, MaxIdx);
    return NumRequired *
      TTI.getArithmeticInstrCost(Opcode, S->getType(), CostKind);
  };

  auto CmpSelCost = [&](unsigned Opcode, unsigned NumRequired, unsigned MinIdx,
                        unsigned MaxIdx) -> InstructionCost {
    Operations.emplace_back(Opcode, MinIdx, MaxIdx);
    Type *OpType = S->getType();
    return NumRequired * TTI.getCmpSelInstrCost(
                             Opcode, OpType, CmpInst::makeCmpResultType(OpType),
                             CmpInst::BAD_ICMP_PREDICATE, CostKind);
  };

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include stable_sort, OperationIndices, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV, TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 stable_sort, OperationIndices，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV, TargetTransformInfo 等分析结果。

### Lines 2002-2162

```cpp
  switch (S->getSCEVType()) {
  case scCouldNotCompute:
    llvm_unreachable("Attempt to use a SCEVCouldNotCompute object!");
  case scUnknown:
  case scConstant:
  case scVScale:
    return 0;
  case scPtrToAddr:
    Cost = CastCost(Instruction::PtrToAddr);
    break;
  case scPtrToInt:
    Cost = CastCost(Instruction::PtrToInt);
    break;
  case scTruncate:
    Cost = CastCost(Instruction::Trunc);
    break;
  case scZeroExtend:
    Cost = CastCost(Instruction::ZExt);
    break;
  case scSignExtend:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case scPtrToAddr:
  case scPtrToInt:
  case scZeroExtend:
  case scSignExtend: {
    Cost +=
        costAndCollectOperands<SCEVCastExpr>(WorkItem, TTI, CostKind, Worklist);
    return false; // Will answer upon next entry into this function.
  }
  case scUDivExpr: {
    // UDivExpr is very likely a UDiv that ScalarEvolution's HowFarToZero or
    // HowManyLessThans produced to compute a precise expression, rather than a
    // UDiv from the user's code. If we can't find a UDiv in the code with some
    // simple searching, we need to account for it's cost.

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV, TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。

### Lines 2163-2331

```cpp
    // At the beginning of this function we already tried to find existing
    // value for plain 'S'. Now try to lookup 'S + 1' since it is common
    // pattern involving division. This is just a simple search heuristic.
    if (hasRelatedExistingExpansion(
            SE.getAddExpr(S, SE.getConstant(S->getType(), 1)), &At, L))
      return false; // Consider it to be free.

    Cost +=
        costAndCollectOperands<SCEVUDivExpr>(WorkItem, TTI, CostKind, Worklist);
    return false; // Will answer upon next entry into this function.
  }
  case scAddExpr:
  case scMulExpr:
  case scUMaxExpr:
  case scSMaxExpr:
  case scUMinExpr:
  case scSMinExpr:
  case scSequentialUMinExpr: {
    assert(cast<SCEVNAryExpr>(S)->getNumOperands() > 1 &&
           "Nary expr should have more than 1 operand.");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  };
  Value *EndCheck = ComputeEndCheck();

  // If the backedge taken count type is larger than the AR type,
  // check that we don't drop any bits by truncating it. If we are
  // dropping bits, then we have overflow (unless the step is zero).
  if (SrcBits > DstBits) {
    auto MaxVal = APInt::getMaxValue(DstBits).zext(SrcBits);
    auto *BackedgeCheck =
        Builder.CreateICmp(ICmpInst::ICMP_UGT, TripCountVal,
                           ConstantInt::get(Loc->getContext(), MaxVal));
    BackedgeCheck = Builder.CreateAnd(
        BackedgeCheck, Builder.CreateICmp(ICmpInst::ICMP_NE, StepValue, Zero));

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2332-2499

```cpp
    EndCheck = Builder.CreateOr(EndCheck, BackedgeCheck);
  }

  return EndCheck;
}

Value *SCEVExpander::expandWrapPredicate(const SCEVWrapPredicate *Pred,
                                         Instruction *IP) {
  const auto *A = cast<SCEVAddRecExpr>(Pred->getExpr());
  Value *NSSWCheck = nullptr, *NUSWCheck = nullptr;

  // Add a check for NUSW
  if (Pred->getFlags() & SCEVWrapPredicate::IncrementNUSW)
    NUSWCheck = generateOverflowCheck(A, IP, false);

  // Add a check for NSSW
  if (Pred->getFlags() & SCEVWrapPredicate::IncrementNSSW)
    NSSWCheck = generateOverflowCheck(A, IP, true);

  if (NUSWCheck && NSSWCheck)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // but for the moment, we paper over the problem by handling two common and
  // cheap to check cases.
  if (SE.properlyDominates(S, InsertionPoint->getParent()))
    return true;
  if (SE.dominates(S, InsertionPoint->getParent())) {
    if (InsertionPoint->getParent()->getTerminator() == InsertionPoint)
      return true;
    if (const SCEVUnknown *U = dyn_cast<SCEVUnknown>(S))
      if (llvm::is_contained(InsertionPoint->operand_values(), U->getValue()))
        return true;
  }
  return false;
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include SCEVFindUnsafe, follow, isSafeToExpand, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 SCEVFindUnsafe, follow, isSafeToExpand，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, ScalarEvolution, SCEV 等分析结果。

### Lines 2500-2533

```cpp
void SCEVExpanderCleaner::cleanup() {
  // Result is used, nothing to remove.
  if (ResultUsed)
    return;

  // Restore original poison flags.
  for (auto [I, Flags] : Expander.OrigFlags)
    Flags.apply(I);

  auto InsertedInstructions = Expander.getAllInsertedInstructions();
#ifndef NDEBUG
  SmallPtrSet<Instruction *, 8> InsertedSet(llvm::from_range,
                                            InsertedInstructions);
  (void)InsertedSet;
#endif
  // Remove sets with value handles.
  Expander.clear();

  // Remove all inserted instructions.
  for (Instruction *I : reverse(InsertedInstructions)) {
#ifndef NDEBUG
    assert(all_of(I->users(),
                  [&InsertedSet](Value *U) {
                    return InsertedSet.contains(cast<Instruction>(U));
                  }) &&
           "removed instruction should only be used by instructions inserted "
           "during expansion");
#endif
    assert(!I->getType()->isVoidTy() &&
           "inserted instruction should have non-void types");
    I->replaceAllUsesWith(PoisonValue::get(I->getType()));
    I->eraseFromParent();
  }
}
```
- EN: Core entities appearing here include cleanup, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 cleanup，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LoopCompare, OperationIndices, SCEVFindUnsafe, PoisonFlags, apply, eraseDeadInstructions, GetOptimalInsertionPointForCastOf, all_of` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LoopCompare, OperationIndices, SCEVFindUnsafe, PoisonFlags, apply, eraseDeadInstructions, GetOptimalInsertionPointForCastOf, all_of` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AliasAnalysis, DataLayout, DominatorTree, IVUsers, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AliasAnalysis, DataLayout, DominatorTree, IVUsers, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolutionPatternMatch.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DataLayout.h`, `llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/LoopUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DataLayout.h`, `llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/LoopUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AliasAnalysis`, `DataLayout`, `DominatorTree`, `IVUsers`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AliasAnalysis`, `DataLayout`, `DominatorTree`, `IVUsers`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
