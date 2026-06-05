# SimplifyIndVar.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SimplifyIndVar.cpp` | `llvm/lib/Transforms/Utils/SimplifyIndVar.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements induction variable simplification within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SimplifyIndVar 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-151

```cpp
//===-- SimplifyIndVar.cpp - Induction variable simplification ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements induction variable simplification. It does
// not define any actual pass or policy, but provides a single function to
// simplify a loop's induction variables based on ScalarEvolution.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SimplifyIndVar.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Dominators.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  const unsigned OperIdx = 0;
  const SCEV *FoldedExpr = nullptr;
  bool MustDropExactFlag = false;
  switch (UseInst->getOpcode()) {
  default:
    return nullptr;
  case Instruction::UDiv:
  case Instruction::LShr:
    // We're only interested in the case where we know something about
    // the numerator and have a constant denominator.
    if (IVOperand != UseInst->getOperand(OperIdx) ||
        !isa<ConstantInt>(UseInst->getOperand(1)))
      return nullptr;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 152-308

```cpp
    // Attempt to fold a binary operator with constant operand.
    // e.g. ((I + 1) >> 2) => I >> 2
    if (!isa<BinaryOperator>(IVOperand)
        || !isa<ConstantInt>(IVOperand->getOperand(1)))
      return nullptr;

    IVSrc = IVOperand->getOperand(0);
    // IVSrc must be the (SCEVable) IV, since the other operand is const.
    assert(SE->isSCEVable(IVSrc->getType()) && "Expect SCEVable IV operand");

    ConstantInt *D = cast<ConstantInt>(UseInst->getOperand(1));
    if (UseInst->getOpcode() == Instruction::LShr) {
      // Get a constant for the divisor. See createSCEV.
      uint32_t BitWidth = cast<IntegerType>(UseInst->getType())->getBitWidth();
      if (D->getValue().uge(BitWidth))
        return nullptr;

      D = ConstantInt::get(UseInst->getContext(),
                           APInt::getOneBitSet(BitWidth, D->getZExtValue()));
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // the unsigned variant (for signed compares) hoping that it will open
    // the doors for other optimizations.  Note that we cannot rely on Pred
    // here unless we check if we have swapped it.
    assert(ICmp->getPredicate() == OriginalPred && "Predicate changed?");
    LLVM_DEBUG(dbgs() << "INDVARS: Marking comparison samesign: " << *ICmp
                      << '\n');
    ICmp->setPredicate(ICmpInst::getUnsignedPredicate(OriginalPred));
    ICmp->setSameSign();
    NumSameSign++;
    Changed = true;
    return;
  }
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 309-458

```cpp
bool SimplifyIndvar::eliminateSDiv(BinaryOperator *SDiv) {
  // Get the SCEVs for the ICmp operands.
  const SCEV *N = SE->getSCEV(SDiv->getOperand(0));
  const SCEV *D = SE->getSCEV(SDiv->getOperand(1));

  // Simplify unnecessary loops away.
  const Loop *L = LI->getLoopFor(SDiv->getParent());
  N = SE->getSCEVAtScope(N, L);
  D = SE->getSCEVAtScope(D, L);

  // Replace sdiv by udiv if both of the operands are non-negative
  if (SE->isKnownNonNegative(N) && SE->isKnownNonNegative(D)) {
    auto *UDiv = BinaryOperator::Create(
        BinaryOperator::UDiv, SDiv->getOperand(0), SDiv->getOperand(1),
        SDiv->getName() + ".udiv", SDiv->getIterator());
    UDiv->setIsExact(SDiv->isExact());
    SDiv->replaceAllUsesWith(UDiv);
    UDiv->setDebugLoc(SDiv->getDebugLoc());
    LLVM_DEBUG(dbgs() << "INDVARS: Simplified sdiv: " << *SDiv << '\n');
    ++NumSimplifiedSDiv;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  for (auto *U : WO->users()) {
    if (auto *EVI = dyn_cast<ExtractValueInst>(U)) {
      if (EVI->getIndices()[0] == 1)
        EVI->replaceAllUsesWith(ConstantInt::getFalse(WO->getContext()));
      else {
        assert(EVI->getIndices()[0] == 0 && "Only two possibilities!");
        EVI->replaceAllUsesWith(NewResult);
        NewResult->setDebugLoc(EVI->getDebugLoc());
      }
      ToDelete.push_back(EVI);
    }
  }

```
- EN: Core entities appearing here include eliminateSDiv, replaceSRemWithURem, replaceRemWithNumerator, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 eliminateSDiv, replaceSRemWithURem, replaceRemWithNumerator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 459-602

```cpp
  for (auto *EVI : ToDelete)
    EVI->eraseFromParent();

  if (WO->use_empty())
    WO->eraseFromParent();

  Changed = true;
  return true;
}

bool SimplifyIndvar::eliminateSaturatingIntrinsic(SaturatingInst *SI) {
  const SCEV *LHS = SE->getSCEV(SI->getLHS());
  const SCEV *RHS = SE->getSCEV(SI->getRHS());
  if (!SE->willNotOverflow(SI->getBinaryOp(), SI->isSigned(), LHS, RHS))
    return false;

  BinaryOperator *BO = BinaryOperator::Create(
      SI->getBinaryOp(), SI->getLHS(), SI->getRHS(), SI->getName(), SI->getIterator());
  if (SI->isSigned())
    BO->setHasNoSignedWrap();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    bool Changed;
    L->makeLoopInvariant(Ext, Changed);
    (void)Changed;
    auto *NewCmp = Builder.CreateICmp(Pred, IV, Ext);
    ICI->replaceAllUsesWith(NewCmp);
    DeadInsts.emplace_back(ICI);
  }

  // Trunc no longer needed.
  TI->replaceAllUsesWith(PoisonValue::get(TI->getType()));
  DeadInsts.emplace_back(TI);
  return true;
}

```
- EN: Core entities appearing here include eliminateSaturatingIntrinsic, eliminateTrunc, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 eliminateSaturatingIntrinsic, eliminateTrunc，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 603-746

```cpp
/// Eliminate an operation that consumes a simple IV and has no observable
/// side-effect given the range of IV values.  IVOperand is guaranteed SCEVable,
/// but UseInst may not be.
bool SimplifyIndvar::eliminateIVUser(Instruction *UseInst,
                                     Instruction *IVOperand) {
  if (ICmpInst *ICmp = dyn_cast<ICmpInst>(UseInst)) {
    eliminateIVComparison(ICmp, IVOperand);
    return true;
  }
  if (BinaryOperator *Bin = dyn_cast<BinaryOperator>(UseInst)) {
    bool IsSRem = Bin->getOpcode() == Instruction::SRem;
    if (IsSRem || Bin->getOpcode() == Instruction::URem) {
      simplifyIVRemainder(Bin, IVOperand, IsSRem);
      return true;
    }

    if (Bin->getOpcode() == Instruction::SDiv)
      return eliminateSDiv(Bin);
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        }
      } else
        Conv = IVOperand;

      CI->replaceAllUsesWith(Conv);
      DeadInsts.push_back(CI);
      LLVM_DEBUG(dbgs() << "INDVARS: Replace IV user: " << *CI
                        << " with: " << *Conv << '\n');

      ++NumFoldedUser;
      Changed = true;
    }
  }

```
- EN: Core entities appearing here include replaceIVUserWithLoopInvariant, replaceFloatIVWithIntegerIV, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 replaceIVUserWithLoopInvariant, replaceFloatIVWithIntegerIV，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 747-899

```cpp
  return Changed;
}

/// Eliminate any operation that SCEV can prove is an identity function.
bool SimplifyIndvar::eliminateIdentitySCEV(Instruction *UseInst,
                                           Instruction *IVOperand) {
  if (!SE->isSCEVable(UseInst->getType()) ||
      UseInst->getType() != IVOperand->getType())
    return false;

  const SCEV *UseSCEV = SE->getSCEV(UseInst);
  if (UseSCEV != SE->getSCEV(IVOperand))
    return false;

  // getSCEV(X) == getSCEV(Y) does not guarantee that X and Y are related in the
  // dominator tree, even if X is an operand to Y.  For instance, in
  //
  //     %iv = phi i32 {0,+,1}
  //     br %cond, label %left, label %merge
  //
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    SimpleIVUsers.push_back(std::make_pair(UI, Def));
  }
}

/// Return true if this instruction generates a simple SCEV
/// expression in terms of that IV.
///
/// This is similar to IVUsers' isInteresting() but processes each instruction
/// non-recursively when the operand is already known to be a simpleIVUser.
///
static bool isSimpleIVUser(Instruction *I, const Loop *L, ScalarEvolution *SE) {
  if (!SE->isSCEVable(I->getType()))
    return false;

```
- EN: Core entities appearing here include m_LShr, isSimpleIVUser, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_LShr, isSimpleIVUser，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 900-1050

```cpp
  // Get the symbolic expression for this instruction.
  const SCEV *S = SE->getSCEV(I);

  // Only consider affine recurrences.
  const SCEVAddRecExpr *AR = dyn_cast<SCEVAddRecExpr>(S);
  if (AR && AR->getLoop() == L)
    return true;

  return false;
}

/// Iteratively perform simplification on a worklist of users
/// of the specified induction variable. Each successive simplification may push
/// more users which may themselves be candidates for simplification.
///
/// This algorithm does not require IVUsers analysis. Instead, it simplifies
/// instructions in-place during analysis. Rather than rewriting induction
/// variables bottom-up from their users, it transforms a chain of IVUsers
/// top-down, updating the IR only when it encounters a clear optimization
/// opportunity.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                     SmallVectorImpl<WeakTrackingVH> &Dead) {
  SCEVExpander Rewriter(*SE, "indvars");
#if LLVM_ENABLE_ABI_BREAKING_CHECKS
  Rewriter.setDebugType(DEBUG_TYPE);
#endif
  bool Changed = false;
  for (BasicBlock::iterator I = L->getHeader()->begin(); isa<PHINode>(I); ++I) {
    const auto &[C, _] =
        simplifyUsersOfIV(cast<PHINode>(I), SE, DT, LI, TTI, Dead, Rewriter);
    Changed |= C;
  }
  return Changed;
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include simplifyUsers, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, IVUsers, LoopInfo, ScalarEvolution.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 simplifyUsers，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, IVUsers, LoopInfo, ScalarEvolution 等分析结果。

### Lines 1051-1197

```cpp
} // namespace llvm

namespace {
//===----------------------------------------------------------------------===//
// Widen Induction Variables - Extend the width of an IV to cover its
// widest uses.
//===----------------------------------------------------------------------===//

class WidenIV {
  // Parameters
  PHINode *OrigPhi;
  Type *WideType;

  // Context
  LoopInfo        *LI;
  Loop            *L;
  ScalarEvolution *SE;
  DominatorTree   *DT;

  // Does the module have any calls to the llvm.experimental.guard intrinsic
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
} // namespace

/// Determine the insertion point for this user. By default, insert immediately
/// before the user. SCEVExpander or LICM will hoist loop invariants out of the
/// loop. For PHI nodes, there may be multiple uses, so compute the nearest
/// common dominator for the incoming blocks. A nullptr can be returned if no
/// viable location is found: it may happen if User is a PHI and Def only comes
/// to this PHI from unreachable blocks.
static Instruction *getInsertPointForUses(Instruction *User, Value *Def,
                                          DominatorTree *DT, LoopInfo *LI) {
  PHINode *PHI = dyn_cast<PHINode>(User);
  if (!PHI)
    return User;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include WidenIV, ExtendKind, updatePostIncRangeInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, IVUsers, LoopInfo, ScalarEvolution.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 WidenIV, ExtendKind, updatePostIncRangeInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, IVUsers, LoopInfo, ScalarEvolution 等分析结果。

### Lines 1198-1345

```cpp
  Instruction *InsertPt = nullptr;
  for (unsigned i = 0, e = PHI->getNumIncomingValues(); i != e; ++i) {
    if (PHI->getIncomingValue(i) != Def)
      continue;

    BasicBlock *InsertBB = PHI->getIncomingBlock(i);

    if (!DT->isReachableFromEntry(InsertBB))
      continue;

    if (!InsertPt) {
      InsertPt = InsertBB->getTerminator();
      continue;
    }
    InsertBB = DT->findNearestCommonDominator(InsertPt->getParent(), InsertBB);
    InsertPt = InsertBB->getTerminator();
  }

  // If we have skipped all inputs, it means that Def only comes to Phi from
  // unreachable blocks.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // and check using SCEV if any of them are correct.

  // Returns true if extending NonIVNarrowDef according to `SignExt` is a
  // correct solution to X.
  auto GuessNonIVOperand = [&](bool SignExt) {
    const SCEV *WideLHS;
    const SCEV *WideRHS;

    auto GetExtend = [this, SignExt](const SCEV *S, Type *Ty) {
      if (SignExt)
        return SE->getSignExtendExpr(S, Ty);
      return SE->getZeroExtendExpr(S, Ty);
    };

```
- EN: Core entities appearing here include DeadInsts, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 DeadInsts，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1346-1492

```cpp
    if (IVOpIdx == 0) {
      WideLHS = SE->getSCEV(WideDef);
      const SCEV *NarrowRHS = SE->getSCEV(NarrowUse->getOperand(1));
      WideRHS = GetExtend(NarrowRHS, WideType);
    } else {
      const SCEV *NarrowLHS = SE->getSCEV(NarrowUse->getOperand(0));
      WideLHS = GetExtend(NarrowLHS, WideType);
      WideRHS = SE->getSCEV(WideDef);
    }

    // WideUse is "WideDef `op.wide` X" as described in the comment.
    const SCEV *WideUse =
      getSCEVByOpCode(WideLHS, WideRHS, NarrowUse->getOpcode());

    return WideUse == WideAR;
  };

  bool SignExtend = getExtendKind(NarrowDef) == ExtendKind::Sign;
  if (!GuessNonIVOperand(SignExtend)) {
    SignExtend = !SignExtend;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return std::nullopt;
}

/// No-wrap operations can transfer sign extension of their result to their
/// operands. Generate the SCEV value for the widened operation without
/// actually modifying the IR yet. If the expression after extending the
/// operands is an AddRec for this loop, return the AddRec and the kind of
/// extension used.
WidenIV::WidenedRecTy
WidenIV::getExtendedOperandRecurrence(WidenIV::NarrowIVDefUse DU) {
  auto Op = matchBinaryOp(DU.NarrowUse);
  if (!Op)
    return {nullptr, ExtendKind::Unknown};

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include getExtendKind, BinaryOp, Operands, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 getExtendKind, BinaryOp, Operands，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 1493-1637

```cpp
  assert((Op->Opcode == Instruction::Add || Op->Opcode == Instruction::Sub ||
          Op->Opcode == Instruction::Mul) &&
         "Unexpected opcode");

  // One operand (NarrowDef) has already been extended to WideDef. Now determine
  // if extending the other will lead to a recurrence.
  const unsigned ExtendOperIdx = Op->Operands[0] == DU.NarrowDef ? 1 : 0;
  assert(Op->Operands[1 - ExtendOperIdx] == DU.NarrowDef && "bad DU");

  ExtendKind ExtKind = getExtendKind(DU.NarrowDef);
  if (!(ExtKind == ExtendKind::Sign && Op->IsNSW) &&
      !(ExtKind == ExtendKind::Zero && Op->IsNUW)) {
    ExtKind = ExtendKind::Unknown;

    // For a non-negative NarrowDef, we can choose either type of
    // extension.  We want to use the current extend kind if legal
    // (see above), and we only hit this code if we need to check
    // the opposite case.
    if (DU.NeverNegative) {
      if (Op->IsNSW) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //    and %narrow is always non-negative.  Then
  //
  //      (A) == icmp slt i32 sext(%narrow), sext(%val)
  //          == icmp slt i32 zext(%narrow), sext(%val)
  bool IsSigned = getExtendKind(DU.NarrowDef) == ExtendKind::Sign;
  bool CmpPreferredSign = Cmp->hasSameSign() ? IsSigned : Cmp->isSigned();
  if (!DU.NeverNegative && IsSigned != CmpPreferredSign)
    return false;

  Value *Op = Cmp->getOperand(Cmp->getOperand(0) == DU.NarrowDef ? 1 : 0);
  unsigned CastWidth = SE->getTypeSizeInBits(Op->getType());
  unsigned IVWidth = SE->getTypeSizeInBits(WideType);
  assert(CastWidth <= IVWidth && "Unexpected width while widening compare.");

```
- EN: Core entities appearing here include getWideRecurrence, truncateIVUse, widenLoopCompare, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getWideRecurrence, truncateIVUse, widenLoopCompare，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1638-1783

```cpp
  // Widen the compare instruction.
  DU.NarrowUse->replaceUsesOfWith(DU.NarrowDef, DU.WideDef);

  // Widen the other operand of the compare, if necessary.
  if (CastWidth < IVWidth) {
    // If the narrow IV is always non-negative and the other operand is sext,
    // widen using sext so we can combine them. This works for all non-signed
    // comparison predicates.
    if (DU.NeverNegative && isa<SExtInst>(Op) && !Cmp->isSigned())
      CmpPreferredSign = true;

    Value *ExtOp = createExtendInst(Op, WideType, CmpPreferredSign, Cmp);
    DU.NarrowUse->replaceUsesOfWith(Op, ExtOp);
  }
  return true;
}

// The widenIVUse avoids generating trunc by evaluating the use as AddRec, this
// will not work when:
//    1) SCEV traces back to an instruction inside the loop that SCEV can not
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                               SE->getNegativeSCEV(RHS), CtxI);
    if (!ProvedSubNUW)
      return false;
    // In fact, our 'add' is 'sub nuw'. We will need to widen the 2nd operand as
    // neg(zext(neg(op))), which is basically sext(op).
    AnotherOpExtKind = ExtendKind::Sign;
  }

  // Verifying that Defining operand is an AddRec
  const SCEV *Op1 = SE->getSCEV(WideDef);
  const SCEVAddRecExpr *AddRecOp1 = dyn_cast<SCEVAddRecExpr>(Op1);
  if (!AddRecOp1 || AddRecOp1->getLoop() != L)
    return false;

```
- EN: Core entities appearing here include widenWithVariantUse, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 widenWithVariantUse，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1784-1936

```cpp
  LLVM_DEBUG(dbgs() << "Cloning arithmetic IVUser: " << *NarrowUse << "\n");

  // Generating a widening use instruction.
  Value *LHS =
      (NarrowUse->getOperand(0) == NarrowDef)
          ? WideDef
          : createExtendInst(NarrowUse->getOperand(0), WideType,
                             AnotherOpExtKind == ExtendKind::Sign, NarrowUse);
  Value *RHS =
      (NarrowUse->getOperand(1) == NarrowDef)
          ? WideDef
          : createExtendInst(NarrowUse->getOperand(1), WideType,
                             AnotherOpExtKind == ExtendKind::Sign, NarrowUse);

  auto *NarrowBO = cast<BinaryOperator>(NarrowUse);
  auto *WideBO = BinaryOperator::Create(NarrowBO->getOpcode(), LHS, RHS,
                                        NarrowBO->getName());
  IRBuilder<> Builder(NarrowUse);
  Builder.Insert(WideBO);
  WideBO->copyIRFlags(NarrowBO);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }
    if (NewDef != DU.NarrowUse) {
      LLVM_DEBUG(dbgs() << "INDVARS: eliminating " << *DU.NarrowUse
                        << " replaced by " << *DU.WideDef << "\n");
      ++NumElimExt;
      DU.NarrowUse->replaceAllUsesWith(NewDef);
      DeadInsts.emplace_back(DU.NarrowUse);
    }
    // Now that the extend is gone, we want to expose it's uses for potential
    // further simplification. We don't need to directly inform SimplifyIVUsers
    // of the new users, because their parent IV will be processed later as a
    // new loop phi. If we preserved IVUsers analysis, we would also want to
    // push the uses of WideDef here.

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1937-2092

```cpp
    // No further widening is needed. The deceased [sz]ext had done it for us.
    return nullptr;
  }

  auto tryAddRecExpansion = [&]() -> Instruction* {
    // Does this user itself evaluate to a recurrence after widening?
    WidenedRecTy WideAddRec = getExtendedOperandRecurrence(DU);
    if (!WideAddRec.first)
      WideAddRec = getWideRecurrence(DU);
    assert((WideAddRec.first == nullptr) ==
           (WideAddRec.second == ExtendKind::Unknown));
    if (!WideAddRec.first)
      return nullptr;

    auto CanUseWideInc = [&]() {
      if (!WideInc)
        return false;
      // Reuse the IV increment that SCEVExpander created. Recompute flags,
      // unless the flags for both increments agree and it is safe to use the
      // ones from the original inc. In that case, the new use of the wide
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Iterate over IV uses (including transitive ones) looking for IV increments
  // of the form 'add nsw %iv, <const>'. For each increment and each use of
  // the increment calculate control-dependent range information basing on
  // dominating conditions inside of the loop (e.g. a range check inside of the
  // loop). Calculated ranges are stored in PostIncRangeInfos map.
  //
  // Control-dependent range information is later used to prove that a narrow
  // definition is not negative (see pushNarrowIVUsers). It's difficult to do
  // this on demand because when pushNarrowIVUsers needs this information some
  // of the dominating conditions might be already widened.
  if (UsePostIncrementRanges)
    calculatePostIncRanges(OrigPhi);

```
- EN: Core entities appearing here include pushNarrowIVUsers, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 pushNarrowIVUsers，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2093-2237

```cpp
  // The rewriter provides a value for the desired IV expression. This may
  // either find an existing phi or materialize a new one. Either way, we
  // expect a well-formed cyclic phi-with-increments. i.e. any operand not part
  // of the phi-SCC dominates the loop entry.
  Instruction *InsertPt = &*L->getHeader()->getFirstInsertionPt();
  Value *ExpandInst = Rewriter.expandCodeFor(AddRec, WideType, InsertPt);
  // If the wide phi is not a phi node, for example a cast node, like bitcast,
  // inttoptr, ptrtoint, just skip for now.
  if (!(WidePhi = dyn_cast<PHINode>(ExpandInst))) {
    // if the cast node is an inserted instruction without any user, we should
    // remove it to make sure the pass don't touch the function as we can not
    // wide the phi.
    if (ExpandInst->use_empty() &&
        Rewriter.isInsertedInstruction(cast<Instruction>(ExpandInst)))
      DeadInsts.emplace_back(ExpandInst);
    return nullptr;
  }

  // Remembering the WideIV increment generated by SCEVExpander allows
  // widenIVUse to reuse it when widening the narrow IV's increment. We don't
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (!DT->isReachableFromEntry(NarrowUserBB))
    return;

  for (auto *DTB = (*DT)[NarrowUserBB]->getIDom();
       L->contains(DTB->getBlock());
       DTB = DTB->getIDom()) {
    auto *BB = DTB->getBlock();
    auto *TI = BB->getTerminator();
    UpdateRangeFromGuards(TI);

    auto *BI = dyn_cast<CondBrInst>(TI);
    if (!BI)
      continue;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as IVUsers, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 IVUsers, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2238-2287

```cpp
    auto *TrueSuccessor = BI->getSuccessor(0);
    auto *FalseSuccessor = BI->getSuccessor(1);

    if (DT->dominates(BasicBlockEdge(BB, TrueSuccessor), NarrowUserBB))
      UpdateRangeFromCondition(BI->getCondition(), /*TrueDest=*/true);

    if (DT->dominates(BasicBlockEdge(BB, FalseSuccessor), NarrowUserBB))
      UpdateRangeFromCondition(BI->getCondition(), /*TrueDest=*/false);
  }
}

/// Calculates PostIncRangeInfos map for the given IV
void WidenIV::calculatePostIncRanges(PHINode *OrigPhi) {
  SmallPtrSet<Instruction *, 16> Visited;
  SmallVector<Instruction *, 6> Worklist;
  Worklist.push_back(OrigPhi);
  Visited.insert(OrigPhi);

  while (!Worklist.empty()) {
    Instruction *NarrowDef = Worklist.pop_back_val();

    for (Use &U : NarrowDef->uses()) {
      auto *NarrowUser = cast<Instruction>(U.getUser());

      // Don't go looking outside the current loop.
      auto *NarrowUserLoop = (*LI)[NarrowUser->getParent()];
      if (!NarrowUserLoop || !L->contains(NarrowUserLoop))
        continue;

      if (!Visited.insert(NarrowUser).second)
        continue;

      Worklist.push_back(NarrowUser);

      calculatePostIncRange(NarrowDef, NarrowUser);
    }
  }
}

PHINode *llvm::createWideIV(const WideIVInfo &WI,
    LoopInfo *LI, ScalarEvolution *SE, SCEVExpander &Rewriter,
    DominatorTree *DT, SmallVectorImpl<WeakTrackingVH> &DeadInsts,
    unsigned &NumElimExt, unsigned &NumWidened,
    bool HasGuards, bool UsePostIncrementRanges) {
  WidenIV Widener(WI, LI, SE, DT, DeadInsts, HasGuards, UsePostIncrementRanges);
  PHINode *WidePHI = Widener.createWideIV(Rewriter);
  NumElimExt = Widener.getNumElimExt();
  NumWidened = Widener.getNumWidened();
  return WidePHI;
}
```
- EN: Core entities appearing here include calculatePostIncRanges, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution, SCEV.
- CN: 此处出现的核心实体包括 calculatePostIncRanges，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution, SCEV 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `SimplifyIndvar, WidenIV, ExtendKind, NarrowIVDefUse, BinaryOp, DeadInsts, eliminateSDiv, replaceSRemWithURem` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SimplifyIndvar, WidenIV, ExtendKind, NarrowIVDefUse, BinaryOp, DeadInsts, eliminateSDiv, replaceSRemWithURem` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, IVUsers, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, IVUsers, LoopInfo, ScalarEvolution, SCEV, TargetTransformInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/SimplifyIndVar.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/SimplifyIndVar.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `IVUsers`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `IVUsers`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
