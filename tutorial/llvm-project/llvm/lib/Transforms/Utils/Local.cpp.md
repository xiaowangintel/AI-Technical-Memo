# Local.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/Local.cpp` | `llvm/lib/Transforms/Utils/Local.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements functions to perform local transformations within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 Local 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-256

```cpp
//===- Local.cpp - Functions to perform local transformations -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This family of functions perform various local transformations to the
// program.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/Local.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseMapInfo.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/Hashing.h"
#include "llvm/ADT/STLExtras.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        It = SI->removeCase(It);
        End = SI->case_end();

        // Removing this case may have made the condition constant. In that
        // case, update CI and restart iteration through the cases.
        if (auto *NewCI = dyn_cast<ConstantInt>(SI->getCondition())) {
          CI = NewCI;
          It = SI->case_begin();
        }

        Changed = true;
        continue;
      }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 257-512

```cpp
      // Otherwise, check to see if the switch only branches to one destination.
      // We do this by reseting "TheOnlyDest" to null when we find two non-equal
      // destinations.
      if (It->getCaseSuccessor() != TheOnlyDest)
        TheOnlyDest = nullptr;

      // Increment this iterator as we haven't removed the case.
      ++It;
    }

    if (CI && !TheOnlyDest) {
      // Branching on a constant, but not any of the cases, go to the default
      // successor.
      TheOnlyDest = SI->getDefaultDest();
    }

    // If we found a single destination that we can fold the switch into, do so
    // now.
    if (TheOnlyDest) {
      // Insert the new branch.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        isAssumeWithEmptyBundle(cast<AssumeInst>(*II))) {
      if (ConstantInt *Cond = dyn_cast<ConstantInt>(II->getArgOperand(0)))
        return !Cond->isZero();

      return false;
    }

    if (auto *FPI = dyn_cast<ConstrainedFPIntrinsic>(I)) {
      std::optional<fp::ExceptionBehavior> ExBehavior =
          FPI->getExceptionBehavior();
      return *ExBehavior != fp::ebStrict;
    }
  }

```
- EN: Core entities appearing here include isAssumeWithEmptyBundle, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isAssumeWithEmptyBundle，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 513-774

```cpp
  if (auto *Call = dyn_cast<CallBase>(I)) {
    if (Value *FreedOp = getFreedOperand(Call, TLI))
      if (Constant *C = dyn_cast<Constant>(FreedOp))
        return C->isNullValue() || isa<UndefValue>(C);
    if (isMathLibCallNoop(Call, TLI))
      return true;
  }

  // Non-volatile atomic loads from constants can be removed.
  if (auto *LI = dyn_cast<LoadInst>(I))
    if (auto *GV = dyn_cast<GlobalVariable>(
            LI->getPointerOperand()->stripPointerCasts()))
      if (!LI->isVolatile() && GV->isConstant())
        return true;

  return false;
}

/// RecursivelyDeleteTriviallyDeadInstructions - If the specified value is a
/// trivially dead instruction, delete it.  If that makes any of its operands
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
//

void llvm::MergeBasicBlockIntoOnlyPred(BasicBlock *DestBB,
                                       DomTreeUpdater *DTU) {

  // If BB has single-entry PHI nodes, fold them.
  while (PHINode *PN = dyn_cast<PHINode>(DestBB->begin())) {
    Value *NewVal = PN->getIncomingValue(0);
    // Replace self referencing PHI with poison, it must be dead.
    if (NewVal == PN) NewVal = PoisonValue::get(PN->getType());
    PN->replaceAllUsesWith(NewVal);
    PN->eraseFromParent();
  }

```
- EN: Core entities appearing here include replaceDbgUsesWithUndef, areAllUsesEqual, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DomTreeUpdater, MemorySSA, MemorySSAUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 replaceDbgUsesWithUndef, areAllUsesEqual，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DomTreeUpdater, MemorySSA, MemorySSAUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 775-1033

```cpp
  BasicBlock *PredBB = DestBB->getSinglePredecessor();
  assert(PredBB && "Block doesn't have a single predecessor!");

  bool ReplaceEntryBB = PredBB->isEntryBlock();

  // DTU updates: Collect all the edges that enter
  // PredBB. These dominator edges will be redirected to DestBB.
  SmallVector<DominatorTree::UpdateType, 32> Updates;

  if (DTU) {
    // To avoid processing the same predecessor more than once.
    SmallPtrSet<BasicBlock *, 2> SeenPreds;
    Updates.reserve(Updates.size() + 2 * pred_size(PredBB) + 1);
    for (BasicBlock *PredOfPredBB : predecessors(PredBB))
      // This predecessor of PredBB may already have DestBB as a successor.
      if (PredOfPredBB != PredBB)
        if (SeenPreds.insert(PredOfPredBB).second)
          Updates.push_back({DominatorTree::Insert, PredOfPredBB, DestBB});
    SeenPreds.clear();
    for (BasicBlock *PredOfPredBB : predecessors(PredBB))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// redirected.
static bool
CanRedirectPredsOfEmptyBBToSucc(BasicBlock *BB, BasicBlock *Succ,
                                const SmallPtrSetImpl<BasicBlock *> &BBPreds,
                                BasicBlock *&CommonPred) {

  // There must be phis in BB, otherwise BB will be merged into Succ directly
  if (BB->phis().empty() || Succ->phis().empty())
    return false;

  // BB must have predecessors not shared that can be redirected to Succ
  if (!BB->hasNPredecessorsOrMore(2))
    return false;

```
- EN: Core entities appearing here include CanMergeValues, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 CanMergeValues，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1034-1291

```cpp
  if (any_of(BBPreds, [](const BasicBlock *Pred) {
        return isa<IndirectBrInst>(Pred->getTerminator());
      }))
    return false;

  // Get the single common predecessor of both BB and Succ. Return false
  // when there are more than one common predecessors.
  for (BasicBlock *SuccPred : predecessors(Succ)) {
    if (BBPreds.count(SuccPred)) {
      if (CommonPred)
        return false;
      CommonPred = SuccPred;
    }
  }

  return true;
}

/// Check whether removing \p BB will make the phis in its \p Succ have too
/// many incoming entries. This function does not check whether \p BB is
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // |    for.body <---- (md2)
  // |_______|  |______|
  if (Instruction *TI = BB->getTerminatorOrNull())
    if (TI->hasNonDebugLocLoopMetadata())
      for (BasicBlock *Pred : predecessors(BB))
        if (Instruction *PredTI = Pred->getTerminatorOrNull())
          if (PredTI->hasNonDebugLocLoopMetadata())
            return false;

  if (BBKillable)
    LLVM_DEBUG(dbgs() << "Killing Trivial BB: \n" << *BB);
  else if (BBPhisMergeable)
    LLVM_DEBUG(dbgs() << "Merge Phis in Trivial BB: \n" << *BB);

```
- EN: Core entities appearing here include introduceTooManyPhiEntries, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 introduceTooManyPhiEntries，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1292-1554

```cpp
  SmallVector<DominatorTree::UpdateType, 32> Updates;

  if (DTU) {
    // To avoid processing the same predecessor more than once.
    SmallPtrSet<BasicBlock *, 8> SeenPreds;
    // All predecessors of BB (except the common predecessor) will be moved to
    // Succ.
    Updates.reserve(Updates.size() + 2 * pred_size(BB) + 1);
    SmallPtrSet<BasicBlock *, 16> SuccPreds(llvm::from_range,
                                            predecessors(Succ));
    for (auto *PredOfBB : predecessors(BB)) {
      // Do not modify those common predecessors of BB and Succ
      if (!SuccPreds.contains(PredOfBB))
        if (SeenPreds.insert(PredOfBB).second)
          Updates.push_back({DominatorTree::Insert, PredOfBB, Succ});
    }

    SeenPreds.clear();

    for (auto *PredOfBB : predecessors(BB))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // doesn't.
    Align CurrentAlign = AI->getAlign();
    if (PrefAlign <= CurrentAlign)
      return CurrentAlign;

    // If the preferred alignment is greater than the natural stack alignment
    // then don't round up. This avoids dynamic stack realignment.
    MaybeAlign StackAlign = DL.getStackAlignment();
    if (StackAlign && PrefAlign > *StackAlign)
      return CurrentAlign;
    AI->setAlignment(PrefAlign);
    return PrefAlign;
  }

```
- EN: Core entities appearing here include PHIDenseMapInfo, isSentinel, getHashValueImpl, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PHIDenseMapInfo, isSentinel, getHashValueImpl，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1555-1825

```cpp
  if (auto *GV = dyn_cast<GlobalVariable>(V)) {
    // TODO: as above, this shouldn't be necessary.
    Align CurrentAlign = GV->getPointerAlignment(DL);
    if (PrefAlign <= CurrentAlign)
      return CurrentAlign;

    // If there is a large requested alignment and we can, bump up the alignment
    // of the global.  If the memory we set aside for the global may not be the
    // memory used by the final program then it is impossible for us to reliably
    // enforce the preferred alignment.
    if (!GV->canIncreaseAlignment())
      return CurrentAlign;

    if (GV->isThreadLocal()) {
      unsigned MaxTLSAlign = GV->getParent()->getMaxTLSAlignment() / CHAR_BIT;
      if (MaxTLSAlign && PrefAlign > Align(MaxTLSAlign))
        PrefAlign = Align(MaxTLSAlign);
    }

    GV->setAlignment(PrefAlign);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  DIBuilder DIB(*F.getParent(), /*AllowUnresolved*/ false);
  SmallVector<DbgDeclareInst *, 4> Dbgs;
  SmallVector<DbgVariableRecord *> DVRs;
  for (auto &FI : F) {
    for (Instruction &BI : FI) {
      if (auto *DDI = dyn_cast<DbgDeclareInst>(&BI))
        Dbgs.push_back(DDI);
      for (DbgVariableRecord &DVR : filterDbgVars(BI.getDbgRecordRange())) {
        if (DVR.getType() == DbgVariableRecord::LocationType::Declare)
          DVRs.push_back(&DVR);
      }
    }
  }

```
- EN: Core entities appearing here include valueCoversEntireFragment, isArray, isStructure, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 valueCoversEntireFragment, isArray, isStructure，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1826-2103

```cpp
  if (Dbgs.empty() && DVRs.empty())
    return Changed;

  auto LowerOne = [&](DbgVariableRecord *DDI) {
    AllocaInst *AI =
        dyn_cast_or_null<AllocaInst>(DDI->getVariableLocationOp(0));
    // If this is an alloca for a scalar variable, insert a dbg.value
    // at each load and store to the alloca and erase the dbg.declare.
    // The dbg.values allow tracking a variable even if it is not
    // stored on the stack, while the dbg.declare can only describe
    // the stack slot (and at a lexical-scope granularity). Later
    // passes will attempt to elide the stack slot.
    if (!AI || isArray(AI) || isStructure(AI))
      return;

    // A volatile load/store means that the alloca can't be elided anyway.
    if (llvm::any_of(AI->users(), [](User *U) -> bool {
          if (LoadInst *LI = dyn_cast<LoadInst>(U))
            return LI->isVolatile();
          if (StoreInst *SI = dyn_cast<StoreInst>(U))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      unsigned LocNo = std::distance(DVRLocation.begin(), LocItr);
      uint64_t CurrentLocOps = SalvagedExpr->getNumLocationOperands();
      Op0 = salvageDebugInfoImpl(I, CurrentLocOps, Ops, AdditionalValues);
      if (!Op0)
        break;
      SalvagedExpr =
          DIExpression::appendOpsToArg(SalvagedExpr, Ops, LocNo, StackValue);
      LocItr = std::find(++LocItr, DVRLocation.end(), &I);
    }
    // salvageDebugInfoImpl should fail on examining the first element of
    // DbgUsers, or none of them.
    if (!Op0)
      break;

```
- EN: Core entities appearing here include salvageDebugInfo, s, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 salvageDebugInfo, s，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2104-2363

```cpp
    SalvagedExpr = SalvagedExpr->foldConstantMath();
    DVR->replaceVariableLocationOp(&I, Op0);
    bool IsValidSalvageExpr =
        SalvagedExpr->getNumElements() <= MaxExpressionSize;
    if (AdditionalValues.empty() && IsValidSalvageExpr) {
      DVR->setExpression(SalvagedExpr);
    } else if (DVR->getType() != DbgVariableRecord::LocationType::Declare &&
               IsValidSalvageExpr &&
               DVR->getNumVariableLocationOps() + AdditionalValues.size() <=
                   MaxDebugArgs) {
      DVR->addVariableLocationOps(AdditionalValues, SalvagedExpr);
    } else {
      // Do not salvage using DIArgList for dbg.addr/dbg.declare, as it is
      // currently only valid for stack value expressions.
      // Also do not salvage if the resulting DIArgList would contain an
      // unreasonably large number of values.
      DVR->setKillLocation();
    }
    LLVM_DEBUG(dbgs() << "SALVAGE: " << DVR << '\n');
    Salvaged = true;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return false;

  // Prevent use-before-def of To.
  bool Changed = false;

  SmallPtrSet<DbgVariableRecord *, 1> UndefOrSalvageDVR;
  if (isa<Instruction>(&To)) {
    bool DomPointAfterFrom = From.getNextNode() == &DomPoint;

    // DbgVariableRecord implementation of the above.
    for (auto *DVR : DPUsers) {
      Instruction *MarkedInstr = DVR->getMarker()->MarkedInstr;
      Instruction *NextNonDebug = MarkedInstr;

```
- EN: Core entities appearing here include getDwarfOpForBinOp, getDwarfOpForIcmpPred, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree.
- CN: 此处出现的核心实体包括 getDwarfOpForBinOp, getDwarfOpForIcmpPred，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。

### Lines 2364-2622

```cpp
      // It's common to see a debug user between From and DomPoint. Move it
      // after DomPoint to preserve the variable update without any reordering.
      if (DomPointAfterFrom && NextNonDebug == &DomPoint) {
        LLVM_DEBUG(dbgs() << "MOVE:  " << *DVR << '\n');
        DVR->removeFromParent();
        DomPoint.getParent()->insertDbgRecordAfter(DVR, &DomPoint);
        Changed = true;

      // Users which otherwise aren't dominated by the replacement value must
      // be salvaged or deleted.
      } else if (!DT.dominates(&DomPoint, MarkedInstr)) {
        UndefOrSalvageDVR.insert(DVR);
      }
    }
  }

  // Update debug users without use-before-def risk.
  for (auto *DVR : DPUsers) {
    if (UndefOrSalvageDVR.count(DVR))
      continue;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  BasicBlock *BB = II->getParent();
  BasicBlock *UnwindDestBB = II->getUnwindDest();
  UnwindDestBB->removePredecessor(BB);
  II->eraseFromParent();
  if (DTU)
    DTU->applyUpdates({{DominatorTree::Delete, BB, UnwindDestBB}});
  return NewCall;
}

BasicBlock *llvm::changeToInvokeAndSplitBasicBlock(CallInst *CI,
                                                   BasicBlock *UnwindEdge,
                                                   DomTreeUpdater *DTU) {
  BasicBlock *BB = CI->getParent();

```
- EN: Core entities appearing here include removeAllNonTerminatorAndEHPadInstructions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, DomTreeUpdater, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 removeAllNonTerminatorAndEHPadInstructions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, DomTreeUpdater, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2623-2881

```cpp
  // Convert this function call into an invoke instruction.  First, split the
  // basic block.
  BasicBlock *Split = SplitBlock(BB, CI, DTU, /*LI=*/nullptr, /*MSSAU*/ nullptr,
                                 CI->getName() + ".noexc");

  // Delete the unconditional branch inserted by SplitBlock
  BB->back().eraseFromParent();

  // Create the new invoke instruction.
  SmallVector<Value *, 8> InvokeArgs(CI->args());
  SmallVector<OperandBundleDef, 1> OpBundles;

  CI->getOperandBundlesAsDefs(OpBundles);

  // Note: we're round tripping operand bundles through memory here, and that
  // can potentially be avoided with a cleverer API design that we do not have
  // as of this time.

  InvokeInst *II =
      InvokeInst::Create(CI->getFunctionType(), CI->getCalledOperand(), Split,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  Instruction *NewTI;
  BasicBlock *UnwindDest;

  if (auto *CRI = dyn_cast<CleanupReturnInst>(TI)) {
    NewTI = CleanupReturnInst::Create(CRI->getCleanupPad(), nullptr, CRI->getIterator());
    UnwindDest = CRI->getUnwindDest();
  } else if (auto *CatchSwitch = dyn_cast<CatchSwitchInst>(TI)) {
    auto *NewCatchSwitch = CatchSwitchInst::Create(
        CatchSwitch->getParentPad(), nullptr, CatchSwitch->getNumHandlers(),
        CatchSwitch->getName(), CatchSwitch->getIterator());
    for (BasicBlock *PadBB : CatchSwitch->handlers())
      NewCatchSwitch->addHandler(PadBB);

```
- EN: Core entities appearing here include CatchPadDenseMapInfo, getHashValue, isEqual, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph, DominatorTree, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 CatchPadDenseMapInfo, getHashValue, isEqual，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph, DominatorTree, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2882-3152

```cpp
    NewTI = NewCatchSwitch;
    UnwindDest = CatchSwitch->getUnwindDest();
  } else {
    llvm_unreachable("Could not find unwind successor");
  }

  NewTI->takeName(TI);
  NewTI->setDebugLoc(TI->getDebugLoc());
  UnwindDest->removePredecessor(BB);
  TI->replaceAllUsesWith(NewTI);
  TI->eraseFromParent();
  if (DTU)
    DTU->applyUpdates({{DominatorTree::Delete, BB, UnwindDest}});
  return NewTI;
}

/// removeUnreachableBlocks - Remove blocks that are not reachable, even
/// if they are in a dead cycle.  Return true if a change was made, false
/// otherwise.
bool llvm::removeUnreachableBlocks(Function &F, DomTreeUpdater *DTU,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    case LLVMContext::MD_fpmath:
    case LLVMContext::MD_tbaa_struct:
    case LLVMContext::MD_invariant_load:
    case LLVMContext::MD_alias_scope:
    case LLVMContext::MD_noalias:
    case LLVMContext::MD_nontemporal:
    case LLVMContext::MD_mem_parallel_loop_access:
    case LLVMContext::MD_access_group:
    case LLVMContext::MD_noundef:
    case LLVMContext::MD_noalias_addrspace:
      // All of these directly apply.
      Dest.setMetadata(ID, N);
      break;

```
- EN: Core entities appearing here include combineAAMetadata, copyMetadataForLoad, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, DomTreeUpdater, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 combineAAMetadata, copyMetadataForLoad，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, DomTreeUpdater, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3153-3418

```cpp
    case LLVMContext::MD_nonnull:
      copyNonnullMetadata(Source, N, Dest);
      break;

    case LLVMContext::MD_align:
    case LLVMContext::MD_dereferenceable:
    case LLVMContext::MD_dereferenceable_or_null:
      // These only directly apply if the new type is also a pointer.
      if (NewType->isPointerTy())
        Dest.setMetadata(ID, N);
      break;

    case LLVMContext::MD_range:
      copyRangeMetadata(DL, Source, N, Dest);
      break;

    case LLVMContext::MD_nofpclass:
      // This only applies if the floating-point type interpretation. This
      // should handle degenerate cases like casting between a scalar and single
      // element vector.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // - Remove their debug intrinsic instructions.
  // - Set their debug locations to the values from the insertion point.
  //
  // As per PR39141 (comment #8), the more fundamental reason why the dbg.values
  // need to be deleted, is because there will not be any instructions with a
  // DILocation in either branch left after performing the transformation. We
  // can only insert a dbg.value after the two branches are joined again.
  //
  // See PR38762, PR39243 for more details.
  //
  // TODO: Extend llvm.dbg.value to take more than one SSA Value (PR39141) to
  // encode predicated DIExpressions that yield different results on different
  // code paths.

```
- EN: Core entities appearing here include patchReplacementInstruction, replaceNonLocalUsesWith, dropDebugUsers, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 patchReplacementInstruction, replaceNonLocalUsesWith, dropDebugUsers，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3419-3680

```cpp
  for (BasicBlock::iterator II = BB->begin(), IE = BB->end(); II != IE;) {
    Instruction *I = &*II;
    I->dropUBImplyingAttrsAndMetadata();
    if (I->isUsedByMetadata())
      dropDebugUsers(*I);
    // RemoveDIs: drop debug-info too as the following code does.
    I->dropDbgRecords();
    if (I->isDebugOrPseudoInst()) {
      // Remove DbgInfo and pseudo probe Intrinsics.
      II = I->eraseFromParent();
      continue;
    }
    I->setDebugLoc(InsertPt->getDebugLoc());
    ++II;
  }
  DomBlock->splice(InsertPt->getIterator(), BB, BB->begin(),
                   BB->getTerminator()->getIterator());
}

DIExpression *llvm::getExpressionForConstant(DIBuilder &DIB, const Constant &C,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      for (unsigned BitIdx = 0; BitIdx < NarrowBitWidth; ++BitIdx)
        Result->Provenance[BitIdx] = Res->Provenance[BitIdx];
      for (unsigned BitIdx = NarrowBitWidth; BitIdx < BitWidth; ++BitIdx)
        Result->Provenance[BitIdx] = BitPart::Unset;
      return Result;
    }

    // If this is a truncate instruction, extract the lower bits.
    if (match(V, m_Trunc(m_Value(X)))) {
      const auto &Res = collectBitParts(X, MatchBSwaps, MatchBitReversals, BPS,
                                        Depth + 1, FoundRoot);
      if (!Res)
        return Result;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include remapDebugVariable, BitPart, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 remapDebugVariable, BitPart，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3681-3943

```cpp
      Result = BitPart(Res->Provider, BitWidth);
      for (unsigned BitIdx = 0; BitIdx < BitWidth; ++BitIdx)
        Result->Provenance[BitIdx] = Res->Provenance[BitIdx];
      return Result;
    }

    // BITREVERSE - most likely due to us previous matching a partial
    // bitreverse.
    if (match(V, m_BitReverse(m_Value(X)))) {
      const auto &Res = collectBitParts(X, MatchBSwaps, MatchBitReversals, BPS,
                                        Depth + 1, FoundRoot);
      if (!Res)
        return Result;

      Result = BitPart(Res->Provider, BitWidth);
      for (unsigned BitIdx = 0; BitIdx < BitWidth; ++BitIdx)
        Result->Provenance[(BitWidth - 1) - BitIdx] = Res->Provenance[BitIdx];
      return Result;
    }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Constant bundle operands may need to retain their constant-ness for
    // correctness.
    if (CB.isBundleOperand(OpIdx))
      return false;

    if (OpIdx < CB.arg_size()) {
      // Some variadic intrinsics require constants in the variadic arguments,
      // which currently aren't markable as immarg.
      if (isa<IntrinsicInst>(CB) &&
          OpIdx >= CB.getFunctionType()->getNumParams()) {
        // This is known to be OK for stackmap.
        return CB.getIntrinsicID() == Intrinsic::experimental_stackmap;
      }

```
- EN: Core entities appearing here include match, canReplaceOperandWithVariable, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match, canReplaceOperandWithVariable，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3944-4074

```cpp
      // gcroot is a special case, since it requires a constant argument which
      // isn't also required to be a simple ConstantInt.
      if (CB.getIntrinsicID() == Intrinsic::gcroot)
        return false;

      // Some intrinsic operands are required to be immediates.
      return !CB.paramHasAttr(OpIdx, Attribute::ImmArg);
    }

    // It is never allowed to replace the call argument to an intrinsic, but it
    // may be possible for a call.
    return !isa<IntrinsicInst>(CB);
  }
  case Instruction::ShuffleVector:
    // Shufflevector masks are constant.
    return OpIdx != 2;
  case Instruction::Switch:
  case Instruction::ExtractValue:
    // All operands apart from the first are constant.
    return OpIdx == 0;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

void OverflowTracking::applyFlags(Instruction &I) {
  I.clearSubclassOptionalData();
  if (I.getOpcode() == Instruction::Add ||
      (I.getOpcode() == Instruction::Mul && AllKnownNonZero)) {
    if (HasNUW)
      I.setHasNoUnsignedWrap();
    if (HasNSW && (AllKnownNonNegative || HasNUW))
      I.setHasNoSignedWrap();
  }
  if (auto *DisjointOp = dyn_cast<PossiblyDisjointInst>(&I))
    DisjointOp->setIsDisjoint(IsDisjoint);
}
```
- EN: Core entities appearing here include inferAttributesFromOthers, mergeFlags, applyFlags, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 inferAttributesFromOthers, mergeFlags, applyFlags，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `PHIDenseMapInfo, CatchPadDenseMapInfo, BitPart, isAssumeWithEmptyBundle, replaceDbgUsesWithUndef, areAllUsesEqual, CanMergeValues, introduceTooManyPhiEntries` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PHIDenseMapInfo, CatchPadDenseMapInfo, BitPart, isAssumeWithEmptyBundle, replaceDbgUsesWithUndef, areAllUsesEqual, CanMergeValues, introduceTooManyPhiEntries` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, CallGraph, DataLayout, DominatorTree, DomTreeUpdater, MemorySSA, MemorySSAUpdater, TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, CallGraph, DataLayout, DominatorTree, DomTreeUpdater, MemorySSA, MemorySSAUpdater, TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `PHICSEDebugHash, PHICSENumPHISmallSize, MaxPhiEntriesIncreaseAfterRemovingEmptyBlock` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `PHICSEDebugHash, PHICSENumPHISmallSize, MaxPhiEntriesIncreaseAfterRemovingEmptyBlock` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumeBundleQueries.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/MemorySSAUpdater.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumeBundleQueries.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/MemorySSAUpdater.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/DIBuilder.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/DIBuilder.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/KnownBits.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseMapInfo.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/KnownBits.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseMapInfo.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `iterator`, `map`, `optional`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `iterator`, `map`, `optional`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `CallGraph`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `MemorySSA`, `MemorySSAUpdater`, `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `CallGraph`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `MemorySSA`, `MemorySSAUpdater`, `TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
