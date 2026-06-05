# LoopUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopUtils.cpp` | `llvm/lib/Transforms/Utils/LoopUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements loop Utility functions within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-166

```cpp
//===-- LoopUtils.cpp - Loop Utility functions -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines common loop utility functions.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/PriorityWorklist.h"
#include "llvm/ADT/ScopeExit.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/AliasAnalysis.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // We must also preserve LoopSimplify and LCSSA. We locally access their IDs
  // here because users shouldn't directly get them from this header.
  extern char &LoopSimplifyID;
  extern char &LCSSAID;
  AU.addRequiredID(LoopSimplifyID);
  AU.addPreservedID(LoopSimplifyID);
  AU.addRequiredID(LCSSAID);
  AU.addPreservedID(LCSSAID);
  // This is used in the LPPassManager to perform LCSSA verification on passes
  // which preserve lcssa form
  AU.addRequired<LCSSAVerificationPass>();
  AU.addPreserved<LCSSAVerificationPass>();

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 167-328

```cpp
  // Loop passes are designed to run inside of a loop pass manager which means
  // that any function analyses they require must be required by the first loop
  // pass in the manager (so that it is computed before the loop pass manager
  // runs) and preserved by all loop pasess in the manager. To make this
  // reasonably robust, the set needed for most loop passes is maintained here.
  // If your loop pass requires an analysis not listed here, you will need to
  // carefully audit the loop pass manager nesting structure that results.
  AU.addRequired<AAResultsWrapperPass>();
  AU.addPreserved<AAResultsWrapperPass>();
  AU.addPreserved<BasicAAWrapperPass>();
  AU.addPreserved<GlobalsAAWrapperPass>();
  AU.addPreserved<SCEVAAWrapperPass>();
  AU.addRequired<ScalarEvolutionWrapperPass>();
  AU.addPreserved<ScalarEvolutionWrapperPass>();
  // FIXME: When all loop passes preserve MemorySSA, it can be required and
  // preserved here instead of the individual handling in each pass.
}

/// Manually defined generic "LoopPass" dependency initialization. This is used
/// to initialize the exact set of passes from above in \c
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  bool HasAnyFollowup = false;
  for (StringRef OptionName : FollowupOptions) {
    MDNode *FollowupNode = findOptionMDForLoopID(OrigLoopID, OptionName);
    if (!FollowupNode)
      continue;

    HasAnyFollowup = true;
    for (const MDOperand &Option : drop_begin(FollowupNode->operands())) {
      MDs.push_back(Option.get());
      Changed = true;
    }
  }

```
- EN: Core entities appearing here include initializeLoopPassPass, getOptionalElementCountLoopAttribute, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 initializeLoopPassPass, getOptionalElementCountLoopAttribute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 329-485

```cpp
  // Attributes of the followup loop not specified explicity, so signal to the
  // transformation pass to add suitable attributes.
  if (!AlwaysNew && !HasAnyFollowup)
    return std::nullopt;

  // If no attributes were added or remove, the previous loop Id can be reused.
  if (!AlwaysNew && !Changed)
    return OrigLoopID;

  // No attributes is equivalent to having no !llvm.loop metadata at all.
  if (MDs.size() == 1)
    return nullptr;

  // Build the new loop ID.
  MDTuple *FollowupLoopID = MDNode::get(OrigLoopID->getContext(), MDs);
  FollowupLoopID->replaceOperandWith(0, FollowupLoopID);
  return FollowupLoopID;
}

bool llvm::hasDisableAllTransformsHint(const Loop *L) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (CurLoop->contains(BB))
      Worklist.push_back(DTN->getBlock());
  };

  AddRegionToWorklist(N);

  for (size_t I = 0; I < Worklist.size(); I++) {
    for (DomTreeNode *Child : DT->getNode(Worklist[I])->children())
      AddRegionToWorklist(Child);
  }

  return Worklist;
}

```
- EN: Core entities appearing here include hasDisableAllTransformsHint, hasDisableLICMTransformsHint, getLoopVectorizeKindPrefix, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasDisableAllTransformsHint, hasDisableLICMTransformsHint, getLoopVectorizeKindPrefix，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 486-648

```cpp
bool llvm::isAlmostDeadIV(PHINode *PN, BasicBlock *LatchBlock, Value *Cond) {
  int LatchIdx = PN->getBasicBlockIndex(LatchBlock);
  assert(LatchIdx != -1 && "LatchBlock is not a case in this PHINode");
  Value *IncV = PN->getIncomingValue(LatchIdx);

  for (User *U : PN->users())
    if (U != Cond && U != IncV) return false;

  for (User *U : IncV->users())
    if (U != Cond && U != PN) return false;
  return true;
}


void llvm::deleteDeadLoop(Loop *L, DominatorTree *DT, ScalarEvolution *SE,
                          LoopInfo *LI, MemorySSA *MSSA) {
  assert((!DT || L->isLCSSAForm(*DT)) && "Expected LCSSA!");
  auto *Preheader = L->getLoopPreheader();
  assert(Preheader && "Preheader should exist!");

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (Instruction &I : *Block) {
      auto *Poison = PoisonValue::get(I.getType());
      for (Use &U : llvm::make_early_inc_range(I.uses())) {
        if (auto *Usr = dyn_cast<Instruction>(U.getUser()))
          if (L->contains(Usr->getParent()))
            continue;
        // If we have a DT then we can check that uses outside a loop only in
        // unreachable block.
        if (DT)
          assert(!DT->isReachableFromEntry(U) &&
                 "Unexpected user in reachable block");
        U.set(Poison);
      }

```
- EN: Core entities appearing here include isAlmostDeadIV, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isAlmostDeadIV，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 649-807

```cpp
      if (ExitBlock) {
        // For one of each variable encountered, preserve a debug record (set
        // to Poison) and transfer it to the loop exit. This terminates any
        // variable locations that were set during the loop.
        for (DbgVariableRecord &DVR :
             llvm::make_early_inc_range(filterDbgVars(I.getDbgRecordRange()))) {
          DebugVariable Key(DVR.getVariable(), DVR.getExpression(),
                            DVR.getDebugLoc().get());
          if (!DeadDebugSet.insert(Key).second)
            continue;
          // Unlinks the DVR from it's container, for later insertion.
          DVR.removeFromParent();
          DeadDbgVariableRecords.push_back(&DVR);
        }
      }
    }

  if (ExitBlock) {
    // After the loop has been deleted all the values defined and modified
    // inside the loop are going to be unavailable. Values computed in the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // loop which might have a had a block removed.
  if (OutermostLoop != L)
    formLCSSARecursively(*OutermostLoop, DT, &LI, &SE);
}


/// Checks if \p L has an exiting latch branch.  There may also be other
/// exiting blocks.  Returns branch instruction terminating the loop
/// latch if above check is successful, nullptr otherwise.
static CondBrInst *getExpectedExitLoopLatchBranch(Loop *L) {
  BasicBlock *Latch = L->getLoopLatch();
  if (!Latch)
    return nullptr;

```
- EN: Core entities appearing here include make_early_inc_range, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 make_early_inc_range，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 808-966

```cpp
  CondBrInst *LatchBR = dyn_cast<CondBrInst>(Latch->getTerminator());
  if (!LatchBR || !L->isLoopExiting(Latch))
    return nullptr;

  assert((LatchBR->getSuccessor(0) == L->getHeader() ||
          LatchBR->getSuccessor(1) == L->getHeader()) &&
         "At least one edge out of the latch must go to the header");

  return LatchBR;
}

struct DbgLoop {
  const Loop *L;
  explicit DbgLoop(const Loop *L) : L(L) {}
};

#ifndef NDEBUG
static inline raw_ostream &operator<<(raw_ostream &OS, DbgLoop D) {
  OS << "function ";
  D.L->getHeader()->getParent()->printAsOperand(OS, /*PrintType=*/false);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

bool llvm::setLoopEstimatedTripCount(
    Loop *L, unsigned EstimatedTripCount,
    std::optional<unsigned> EstimatedloopInvocationWeight) {
  // If EstimatedLoopInvocationWeight, we do not support this loop if
  // getExpectedExitLoopLatchBranch returns nullptr.
  //
  // FIXME: See comments in getLoopEstimatedTripCount for why this is required
  // here regardless of EstimatedLoopInvocationWeight.
  CondBrInst *LatchBranch = getExpectedExitLoopLatchBranch(L);
  if (!LatchBranch)
    return false;

```
- EN: Core entities appearing here include DbgLoop, estimateLoopTripCount, getOptionalIntLoopAttribute, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 DbgLoop, estimateLoopTripCount, getOptionalIntLoopAttribute，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 967-1143

```cpp
  // Set the metadata.
  addStringMetadataToLoop(L, LLVMLoopEstimatedTripCount, EstimatedTripCount);

  // At the moment, we currently support changing the estimated trip count in
  // the latch branch's branch weights only.  We could extend this API to
  // manipulate estimated trip counts for any exit.
  //
  // TODO: Eventually, once all passes have migrated away from setting branch
  // weights to indicate estimated trip counts, we will not set branch weights
  // here at all.
  if (!EstimatedloopInvocationWeight)
    return true;

  // Calculate taken and exit weights.
  unsigned LatchExitWeight = ProfcheckDisableMetadataFixes ? 0 : 1;
  unsigned BackedgeTakenWeight = 0;

  if (EstimatedTripCount != 0) {
    LatchExitWeight = *EstimatedloopInvocationWeight;
    BackedgeTakenWeight = (EstimatedTripCount - 1) * LatchExitWeight;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case RecurKind::FMin:
  case RecurKind::FMinNum:
    return Intrinsic::vector_reduce_fmin;
  case RecurKind::FMaximum:
    return Intrinsic::vector_reduce_fmaximum;
  case RecurKind::FMinimum:
    return Intrinsic::vector_reduce_fminimum;
  case RecurKind::FMaximumNum:
    return Intrinsic::vector_reduce_fmax;
  case RecurKind::FMinimumNum:
    return Intrinsic::vector_reduce_fmin;
  }
}

```
- EN: Core entities appearing here include getLoopProbability, setLoopProbability, getBranchProbability, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 此处出现的核心实体包括 getLoopProbability, setLoopProbability, getBranchProbability，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 1144-1300

```cpp
Intrinsic::ID llvm::getMinMaxReductionIntrinsicID(Intrinsic::ID IID) {
  switch (IID) {
  default:
    llvm_unreachable("Unexpected intrinsic id");
  case Intrinsic::umin:
    return Intrinsic::vector_reduce_umin;
  case Intrinsic::umax:
    return Intrinsic::vector_reduce_umax;
  case Intrinsic::smin:
    return Intrinsic::vector_reduce_smin;
  case Intrinsic::smax:
    return Intrinsic::vector_reduce_smax;
  }
}

// This is the inverse to getReductionForBinop
unsigned llvm::getArithmeticReductionInstruction(Intrinsic::ID RdxID) {
  switch (RdxID) {
  case Intrinsic::vector_reduce_fadd:
    return Instruction::FAdd;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case RecurKind::SMin:
    return CmpInst::ICMP_SLT;
  case RecurKind::SMax:
    return CmpInst::ICMP_SGT;
  case RecurKind::FMin:
    return CmpInst::FCMP_OLT;
  case RecurKind::FMax:
    return CmpInst::FCMP_OGT;
  // We do not add FMinimum/FMaximum recurrence kind here since there is no
  // equivalent predicate which compares signed zeroes according to the
  // semantics of the intrinsics (llvm.minimum/maximum).
  }
}

```
- EN: Core entities appearing here include getMinMaxReductionIntrinsicID, getArithmeticReductionInstruction, getReductionForBinop, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getMinMaxReductionIntrinsicID, getArithmeticReductionInstruction, getReductionForBinop，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1301-1462

```cpp
Value *llvm::createMinMaxOp(IRBuilderBase &Builder, RecurKind RK, Value *Left,
                            Value *Right) {
  Type *Ty = Left->getType();
  if (Ty->isIntOrIntVectorTy() ||
      (RK == RecurKind::FMinNum || RK == RecurKind::FMaxNum ||
       RK == RecurKind::FMinimum || RK == RecurKind::FMaximum ||
       RK == RecurKind::FMinimumNum || RK == RecurKind::FMaximumNum)) {
    Intrinsic::ID Id = getMinMaxReductionIntrinsicOp(RK);
    return Builder.CreateIntrinsic(Ty, Id, {Left, Right}, nullptr,
                                   "rdx.minmax");
  }
  CmpInst::Predicate Pred = getMinMaxReductionPredicate(RK);
  Value *Cmp = Builder.CreateCmp(Pred, Left, Right, "rdx.minmax.cmp");
  Value *Select = Builder.CreateSelect(Cmp, Left, Right, "rdx.minmax.select");
  return Select;
}

// Helper to generate an ordered reduction.
Value *llvm::getOrderedReduction(IRBuilderBase &Builder, Value *Acc, Value *Src,
                                 unsigned Op, RecurKind RdxKind) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (unsigned i = VF; i != 1; i >>= 1) {
      // Move the upper half of the vector to the lower half.
      for (unsigned j = 0; j != i / 2; ++j)
        ShuffleMask[j] = i / 2 + j;

      // Fill the rest of the mask with undef.
      std::fill(&ShuffleMask[i / 2], ShuffleMask.end(), -1);
      BuildShuffledOp(ShuffleMask, TmpVec);
    }
  }
  // The result is in the first element of the vector.
  return Builder.CreateExtractElement(TmpVec, Builder.getInt32(0));
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, LoopInfo, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, LoopInfo, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1463-1624

```cpp
Value *llvm::createAnyOfReduction(IRBuilderBase &Builder, Value *Src,
                                  Value *InitVal, PHINode *OrigPhi) {
  Value *NewVal = nullptr;

  // First use the original phi to determine the new value we're trying to
  // select from in the loop.
  SelectInst *SI = nullptr;
  for (auto *U : OrigPhi->users()) {
    if ((SI = dyn_cast<SelectInst>(U)))
      break;
  }
  assert(SI && "One user of the original phi should be a select");

  if (SI->getTrueValue() == OrigPhi)
    NewVal = SI->getFalseValue();
  else {
    assert(SI->getFalseValue() == OrigPhi &&
           "At least one input to the select should be the original Phi");
    NewVal = SI->getTrueValue();
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  assert((Kind == RecurKind::FAdd || Kind == RecurKind::FMulAdd) &&
         "Unexpected reduction kind");
  assert(Src->getType()->isVectorTy() && "Expected a vector type");
  assert(!Start->getType()->isVectorTy() && "Expected a scalar type");

  Intrinsic::ID Id = getReductionIntrinsicID(RecurKind::FAdd);
  auto VPID = VPIntrinsic::getForIntrinsic(Id);
  assert(VPReductionIntrinsic::isVPReduction(VPID) &&
         "No VPIntrinsic for this reduction");
  auto *EltTy = cast<VectorType>(Src->getType())->getElementType();
  Value *Ops[] = {Start, Src, Mask, EVL};
  return Builder.CreateIntrinsic(EltTy, VPID, Ops);
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1625-1782

```cpp
void llvm::propagateIRFlags(Value *I, ArrayRef<Value *> VL, Value *OpValue,
                            bool IncludeWrapFlags) {
  auto *VecOp = dyn_cast<Instruction>(I);
  if (!VecOp)
    return;
  auto *Intersection = (OpValue == nullptr) ? dyn_cast<Instruction>(VL[0])
                                            : dyn_cast<Instruction>(OpValue);
  if (!Intersection)
    return;
  const unsigned Opcode = Intersection->getOpcode();
  VecOp->copyIRFlags(Intersection, IncludeWrapFlags);
  for (auto *V : VL) {
    auto *Instr = dyn_cast<Instruction>(V);
    if (!Instr)
      continue;
    if (OpValue == nullptr || Opcode == Instr->getOpcode())
      VecOp->andIRFlags(V);
  }
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (Phi.PN == P && (Phi.PN)->getIncomingValue(i) == Incoming) {
        found = true;
        break;
      }
    }

    Instruction *I;
    if (!found && (I = dyn_cast<Instruction>(Incoming)))
      if (!L->hasLoopInvariantOperands(I))
        return false;

    ++BI;
  }

```
- EN: Core entities appearing here include hasHardUserWithinLoop, RewritePhi, canLoopBeDeleted, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 此处出现的核心实体包括 hasHardUserWithinLoop, RewritePhi, canLoopBeDeleted，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 1783-1941

```cpp
  for (auto *BB : L->blocks())
    if (llvm::any_of(*BB, [](Instruction &I) {
          return I.mayHaveSideEffects();
        }))
      return false;

  return true;
}

/// Checks if it is safe to call InductionDescriptor::isInductionPHI for \p Phi,
/// and returns true if this Phi is an induction phi in the loop. When
/// isInductionPHI returns true, \p ID will be also be set by isInductionPHI.
static bool checkIsIndPhi(PHINode *Phi, Loop *L, ScalarEvolution *SE,
                          InductionDescriptor &ID) {
  if (!Phi)
    return false;
  if (!L->getLoopPreheader())
    return false;
  if (Phi->getParent() != L->getHeader())
    return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        if (ReplaceExitValue != AlwaysRepl && !isa<SCEVConstant>(ExitValue) &&
            !isa<SCEVUnknown>(ExitValue) && hasHardUserWithinLoop(L, Inst))
          continue;

        // Check if expansions of this SCEV would count as being high cost.
        bool HighCost = Rewriter.isHighCostExpansion(
            ExitValue, L, SCEVCheapExpansionBudget, TTI, Inst);

        // Note that we must not perform expansions until after
        // we query *all* the costs, because if we perform temporary expansion
        // inbetween, one that we might not intend to keep, said expansion
        // *may* affect cost calculation of the next SCEV's we'll query,
        // and next SCEV may errneously get smaller cost.

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution, SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution, SCEV 等分析结果。

### Lines 1942-2103

```cpp
        // Collect all the candidate PHINodes to be rewritten.
        Instruction *InsertPt =
          (isa<PHINode>(Inst) || isa<LandingPadInst>(Inst)) ?
          &*Inst->getParent()->getFirstInsertionPt() : Inst;
        RewritePhiSet.emplace_back(PN, i, ExitValue, InsertPt, HighCost);
      }
    }
  }

  // TODO: evaluate whether it is beneficial to change how we calculate
  // high-cost: if we have SCEV 'A' which we know we will expand, should we
  // calculate the cost of other SCEV's after expanding SCEV 'A', thus
  // potentially giving cost bonus to those other SCEV's?

  bool LoopCanBeDel = canLoopBeDeleted(L, RewritePhiSet);
  int NumReplaced = 0;

  // Transformation.
  for (const RewritePhi &Phi : RewritePhiSet) {
    PHINode *PN = Phi.PN;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
struct PointerBounds {
  TrackingVH<Value> Start;
  TrackingVH<Value> End;
  Value *StrideToCheck;
};

/// Expand code for the lower and upper bound of the pointer group \p CG
/// in \p TheLoop.  \return the values for the bounds.
static PointerBounds expandBounds(const RuntimeCheckingPtrGroup *CG,
                                  Loop *TheLoop, Instruction *Loc,
                                  SCEVExpander &Exp, bool HoistRuntimeChecks) {
  LLVMContext &Ctx = Loc->getContext();
  Type *PtrArithTy = PointerType::get(Ctx, CG->AddressSpace);

```
- EN: Core entities appearing here include PointerBounds, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PointerBounds，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2104-2262

```cpp
  Value *Start = nullptr, *End = nullptr;
  LLVM_DEBUG(dbgs() << "LAA: Adding RT check for range:\n");
  const SCEV *Low = CG->Low, *High = CG->High, *Stride = nullptr;

  // If the Low and High values are themselves loop-variant, then we may want
  // to expand the range to include those covered by the outer loop as well.
  // There is a trade-off here with the advantage being that creating checks
  // using the expanded range permits the runtime memory checks to be hoisted
  // out of the outer loop. This reduces the cost of entering the inner loop,
  // which can be significant for low trip counts. The disadvantage is that
  // there is a chance we may now never enter the vectorized inner loop,
  // whereas using a restricted range check could have allowed us to enter at
  // least once. This is why the behaviour is not currently the default and is
  // controlled by the parameter 'HoistRuntimeChecks'.
  if (HoistRuntimeChecks && TheLoop->getParentLoop() &&
      isa<SCEVAddRecExpr>(High) && isa<SCEVAddRecExpr>(Low)) {
    auto *HighAR = cast<SCEVAddRecExpr>(High);
    auto *LowAR = cast<SCEVAddRecExpr>(Low);
    const Loop *OuterLoop = TheLoop->getParentLoop();
    ScalarEvolution &SE = *Exp.getSE();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
struct SCEVPtrToAddrRewriter : SCEVRewriteVisitor<SCEVPtrToAddrRewriter> {
  const DataLayout &DL;
  SCEVPtrToAddrRewriter(ScalarEvolution &SE, const DataLayout &DL)
      : SCEVRewriteVisitor(SE), DL(DL) {}

  const SCEV *visitPtrToIntExpr(const SCEVPtrToIntExpr *E) {
    const SCEV *Op = visit(E->getOperand());
    if (E->getType() == DL.getAddressType(E->getOperand()->getType()))
      return SE.getPtrToAddrExpr(Op);
    return Op == E->getOperand() ? E : SE.getPtrToIntExpr(Op, E->getType());
  }
};
} // namespace

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include SCEVPtrToAddrRewriter, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution, SCEV.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 SCEVPtrToAddrRewriter，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution, SCEV 等分析结果。

### Lines 2263-2422

```cpp
Value *llvm::addDiffRuntimeChecks(
    Instruction *Loc, ArrayRef<PointerDiffInfo> Checks, SCEVExpander &Expander,
    function_ref<Value *(IRBuilderBase &, unsigned)> GetVF, unsigned IC) {

  LLVMContext &Ctx = Loc->getContext();
  IRBuilder ChkBuilder(Ctx, InstSimplifyFolder(Loc->getDataLayout()));
  ChkBuilder.SetInsertPoint(Loc);
  // Our instructions might fold to a constant.
  Value *MemoryRuntimeCheck = nullptr;

  auto &SE = *Expander.getSE();
  const DataLayout &DL = Loc->getDataLayout();
  SCEVPtrToAddrRewriter Rewriter(SE, DL);
  // Map to keep track of created compares, The key is the pair of operands for
  // the compare, to allow detecting and re-using redundant compares.
  DenseMap<std::pair<Value *, Value *>, Value *> SeenCompares;
  for (const auto &[SrcStart, SinkStart, AccessSize, NeedsFreeze] : Checks) {
    Type *Ty = SinkStart->getType();
    // Compute VF * IC * AccessSize.
    auto *VFTimesICTimesSize =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    while (!AccessesToCheck.empty()) {
      MemoryAccess *Current = AccessesToCheck.pop_back_val();
      auto SeenI = SeenAccesses.insert(Current);
      if (!SeenI.second || !Seen.contains(Current->getBlock()))
        continue;

      // Bail out if exceeded the threshold.
      if (SeenAccesses.size() >= MSSAThreshold)
        return {};

      // MemoryUse are read-only accesses.
      if (isa<MemoryUse>(Current))
        continue;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout, MemorySSA, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout, MemorySSA, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2423-2486

```cpp
      // For a MemoryDef, check if is aliases any of the location feeding
      // the original condition.
      if (auto *CurrentDef = dyn_cast<MemoryDef>(Current)) {
        if (any_of(AccessedLocs, [&AA, CurrentDef](MemoryLocation &Loc) {
              return isModSet(
                  AA.getModRefInfo(CurrentDef->getMemoryInst(), Loc));
            }))
          return {};
      }

      for (Use &U : Current->uses())
        AccessesToCheck.push_back(cast<MemoryAccess>(U.getUser()));
    }

    // We could also allow loops with known trip counts without mustprogress,
    // but ScalarEvolution may not be available.
    Info.PathIsNoop &= isMustProgress(&L);

    // If the path is considered a no-op so far, check if it reaches a
    // single exit block without any phis. This ensures no values from the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  if (auto Info = HasNoClobbersOnPath(TI->getSuccessor(0), L.getHeader(),
                                      AccessesToCheck)) {
    Info->KnownValue = ConstantInt::getTrue(TI->getContext());
    return Info;
  }
  if (auto Info = HasNoClobbersOnPath(TI->getSuccessor(1), L.getHeader(),
                                      AccessesToCheck)) {
    Info->KnownValue = ConstantInt::getFalse(TI->getContext());
    return Info;
  }

  return {};
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `DbgLoop, RewritePhi, PointerBounds, SCEVPtrToAddrRewriter, findDefsUsedOutsideOfLoop, getLoopAnalysisUsage, initializeLoopPassPass, getOptionalElementCountLoopAttribute` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`DbgLoop, RewritePhi, PointerBounds, SCEVPtrToAddrRewriter, findDefsUsedOutsideOfLoop, getLoopAnalysisUsage, initializeLoopPassPass, getOptionalElementCountLoopAttribute` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA, MemorySSAUpdater` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, MemorySSA, MemorySSAUpdater` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/BasicAliasAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstSimplifyFolder.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/BasicAliasAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstSimplifyFolder.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DIBuilder.h`, `llvm/IR/Dominators.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/ValueHandle.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DIBuilder.h`, `llvm/IR/Dominators.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/ValueHandle.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/PriorityWorklist.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/PriorityWorklist.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AAResults`, `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `MemorySSA`, `MemorySSAUpdater` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `MemorySSA`, `MemorySSAUpdater` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
