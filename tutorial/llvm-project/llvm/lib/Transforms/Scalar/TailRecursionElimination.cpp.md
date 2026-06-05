# TailRecursionElimination.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/TailRecursionElimination.cpp` | `llvm/lib/Transforms/Scalar/TailRecursionElimination.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements eliminate Tail Calls within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 TailRecursionElimination 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-86

```cpp
//===- TailRecursionElimination.cpp - Eliminate Tail Calls ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file transforms calls of the current function (self recursion) followed
// by a return instruction with a branch to the entry of the function, creating
// a loop.  This pass also implements the following extensions to the basic
// algorithm:
//
//  1. Trivial instructions between the call and return do not prevent the
//     transformation from taking place, though currently the analysis cannot
//     support moving any really useful instructions (only dead ones).
//  2. This pass transforms functions that are prevented from being tail
//     recursive by an associative and commutative expression to use an
//     accumulator variable, thus compiling the typical naive factorial or
//     'fib' implementation into efficient code.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Module.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include <cmath>
using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as BlockFrequencyInfo, DataLayout, DomTreeUpdater, OptimizationRemarkEmitter.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 BlockFrequencyInfo, DataLayout, DomTreeUpdater, OptimizationRemarkEmitter 等分析结果。

### Lines 87-174

```cpp
#define DEBUG_TYPE "tailcallelim"

STATISTIC(NumEliminated, "Number of tail calls removed");
STATISTIC(NumRetDuped,   "Number of return duplicated");
STATISTIC(NumAccumAdded, "Number of accumulators introduced");

static cl::opt<bool> ForceDisableBFI(
    "tre-disable-entrycount-recompute", cl::init(false), cl::Hidden,
    cl::desc("Force disabling recomputing of function entry count, on "
             "successful tail recursion elimination."));

/// Scan the specified function for alloca instructions.
/// If it contains any dynamic allocas, returns false.
static bool canTRE(Function &F) {
  // TODO: We don't do TRE if dynamic allocas are used.
  // Dynamic allocas allocate stack space which should be
  // deallocated before new iteration started. That is
  // currently not implemented.
  return llvm::all_of(instructions(F), [](Instruction &I) {
    auto *AI = dyn_cast<AllocaInst>(&I);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          EscapePoints.insert(I);
        continue;  // Stores have no users to analyze.
      }
      case Instruction::BitCast:
      case Instruction::GetElementPtr:
      case Instruction::PHI:
      case Instruction::Select:
      case Instruction::AddrSpaceCast:
        break;
      default:
        EscapePoints.insert(I);
        break;
      }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include canTRE, AllocaDerivedValueTracker, walk, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 canTRE, AllocaDerivedValueTracker, walk，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 175-265

```cpp
      AddUsesToWorklist(I);
    }
  }

  void callUsesLocalStack(CallBase &CB, bool IsNocapture) {
    // Add it to the list of alloca users.
    AllocaUsers.insert(&CB);

    // If it's nocapture then it can't capture this alloca.
    if (IsNocapture)
      return;

    // If it can write to memory, it can leak the alloca value.
    if (!CB.onlyReadsMemory())
      EscapePoints.insert(&CB);
  }

  SmallPtrSet<Instruction *, 32> AllocaUsers;
  SmallPtrSet<Instruction *, 32> EscapePoints;
};
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

      // Bail out for intrinsic stackrestore call because it can modify
      // unescaped allocas.
      if (auto *II = dyn_cast<IntrinsicInst>(CI))
        if (II->getIntrinsicID() == Intrinsic::stackrestore)
          continue;

      // Special-case operand bundles "clang.arc.attachedcall", "ptrauth", and
      // "kcfi".
      bool IsNoTail = CI->isNoTailCall() ||
                      CI->hasOperandBundlesOtherThan(
                          {LLVMContext::OB_clang_arc_attachedcall,
                           LLVMContext::OB_ptrauth, LLVMContext::OB_kcfi});

```
- EN: Core entities appearing here include callUsesLocalStack, markTails, VisitType, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter.
- CN: 此处出现的核心实体包括 callUsesLocalStack, markTails, VisitType，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。

### Lines 266-353

```cpp
      if (!IsNoTail && CI->doesNotAccessMemory()) {
        // A call to a readnone function whose arguments are all things computed
        // outside this function can be marked tail. Even if you stored the
        // alloca address into a global, a readnone function can't load the
        // global anyhow.
        //
        // Note that this runs whether we know an alloca has escaped or not. If
        // it has, then we can't trust Tracker.AllocaUsers to be accurate.
        bool SafeToTail = true;
        for (auto &Arg : CI->args()) {
          if (isa<Constant>(Arg.getUser()))
            continue;
          if (Argument *A = dyn_cast<Argument>(Arg.getUser()))
            if (!A->hasByValAttr())
              continue;
          SafeToTail = false;
          break;
        }
        if (SafeToTail) {
          using namespace ore;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// Return true if it is safe to move the specified
/// instruction from after the call to before the call, assuming that all
/// instructions between the call and this instruction are movable.
///
static bool canMoveAboveCall(Instruction *I, CallInst *CI, AliasAnalysis *AA) {
  if (const IntrinsicInst *II = dyn_cast<IntrinsicInst>(I))
    if (II->getIntrinsicID() == Intrinsic::lifetime_end)
      return true;

  // FIXME: We can move load/store/call/free instructions above the call if the
  // call does not mod/ref the memory location being processed.
  if (I->mayHaveSideEffects())  // This also handles volatile loads.
    return false;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include canMoveAboveCall, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 canMoveAboveCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis 等分析结果。

### Lines 354-438

```cpp
  if (LoadInst *L = dyn_cast<LoadInst>(I)) {
    // Loads may always be moved above calls without side effects.
    if (CI->mayHaveSideEffects()) {
      // Non-volatile loads may be moved above a call with side effects if it
      // does not write to memory and the load provably won't trap.
      // Writes to memory only matter if they may alias the pointer
      // being loaded from.
      const DataLayout &DL = L->getDataLayout();
      if (isModSet(AA->getModRefInfo(CI, MemoryLocation::get(L))) ||
          !isSafeToLoadUnconditionally(L->getPointerOperand(), L->getType(),
                                       L->getAlign(), DL, L))
        return false;
    }
  }

  // Otherwise, if this is a side-effect free instruction, check to make sure
  // that it does not use the return value of the call.  If it doesn't use the
  // return value of the call, it must only use things that are defined before
  // the call, or movable instructions between the call and the instruction
  // itself.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Vector of select instructions we insereted. These selects use RetKnownPN
  // to either propagate RetPN or select a new return value.
  SmallVector<SelectInst *, 8> RetSelects;

  // The below are shared state needed when performing accumulator recursion.
  // There values should be populated by insertAccumulator the first time we
  // find an elimination that requires an accumulator.

  // PHI node to store our current accumulated value.
  PHINode *AccPN = nullptr;

  // The instruction doing the accumulating.
  Instruction *AccumulatorRecursionInstr = nullptr;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include canTransformAccumulatorRecursion, TailRecursionEliminator, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, BlockFrequencyInfo, DataLayout, DomTreeUpdater.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 canTransformAccumulatorRecursion, TailRecursionEliminator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, BlockFrequencyInfo, DataLayout, DomTreeUpdater 等分析结果。

### Lines 439-533

```cpp
  TailRecursionEliminator(Function &F, const TargetTransformInfo *TTI,
                          AliasAnalysis *AA, OptimizationRemarkEmitter *ORE,
                          DomTreeUpdater &DTU, BlockFrequencyInfo *BFI)
      : F(F), TTI(TTI), AA(AA), ORE(ORE), DTU(DTU), BFI(BFI),
        OrigEntryBBFreq(
            BFI ? BFI->getBlockFreq(&F.getEntryBlock()).getFrequency() : 0U),
        OrigEntryCount(F.getEntryCount() ? F.getEntryCount()->getCount() : 0) {
    if (BFI) {
      // The assert is meant as API documentation for the caller.
      assert((OrigEntryCount != 0 && OrigEntryBBFreq != 0) &&
             "If a BFI was provided, the function should have both an entry "
             "count that is non-zero and an entry basic block with a non-zero "
             "frequency.");
    }
  }

  CallInst *findTRECandidate(BasicBlock *BB);

  void createTailRecurseLoopHeader(CallInst *CI);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return CI;
}

void TailRecursionEliminator::createTailRecurseLoopHeader(CallInst *CI) {
  HeaderBB = &F.getEntryBlock();
  BasicBlock *NewEntry = BasicBlock::Create(F.getContext(), "", &F, HeaderBB);
  NewEntry->takeName(HeaderBB);
  HeaderBB->setName("tailrecurse");
  auto *BI = UncondBrInst::Create(HeaderBB, NewEntry);
  BI->setDebugLoc(DebugLoc::getCompilerGenerated());
  // If the new branch preserves the debug location of CI, it could result in
  // misleading stepping, if CI is located in a conditional branch.
  // So, here we don't give any debug location to the new branch.

```
- EN: Core entities appearing here include OrigEntryCount, createTailRecurseLoopHeader, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, BlockFrequencyInfo, DomTreeUpdater, OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 OrigEntryCount, createTailRecurseLoopHeader，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, BlockFrequencyInfo, DomTreeUpdater, OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 534-624

```cpp
  // Move all fixed sized allocas from HeaderBB to NewEntry.
  for (BasicBlock::iterator OEBI = HeaderBB->begin(), E = HeaderBB->end(),
                            NEBI = NewEntry->begin();
       OEBI != E;)
    if (AllocaInst *AI = dyn_cast<AllocaInst>(OEBI++))
      if (isa<ConstantInt>(AI->getArraySize()))
        AI->moveBefore(NEBI);

  // Now that we have created a new block, which jumps to the entry
  // block, insert a PHI node for each argument of the function.
  // For now, we initialize each PHI to only have the real arguments
  // which are passed in.
  BasicBlock::iterator InsertPos = HeaderBB->begin();
  for (Function::arg_iterator I = F.arg_begin(), E = F.arg_end(); I != E; ++I) {
    PHINode *PN = PHINode::Create(I->getType(), 2, I->getName() + ".tr");
    PN->insertBefore(InsertPos);
    I->replaceAllUsesWith(PN); // Everyone use the PHI node now!
    PN->addIncoming(&*I, NewEntry);
    ArgumentPHIs.push_back(PN);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                                              int OpndIdx) {
  Type *AggTy = CI->getParamByValType(OpndIdx);
  assert(AggTy);
  const DataLayout &DL = F.getDataLayout();

  // Get alignment of byVal operand.
  Align Alignment(CI->getParamAlign(OpndIdx).valueOrOne());

  // Create alloca for temporarily byval operands.
  // Put alloca into the entry block.
  Value *NewAlloca = new AllocaInst(
      AggTy, DL.getAllocaAddrSpace(), nullptr, Alignment,
      CI->getArgOperand(OpndIdx)->getName(), F.getEntryBlock().begin());

```
- EN: Core entities appearing here include insertAccumulator, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 insertAccumulator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 625-716

```cpp
  IRBuilder<> Builder(CI);
  Value *Size = Builder.getInt64(DL.getTypeAllocSize(AggTy));

  // Copy data from byvalue operand into the temporarily variable.
  Builder.CreateMemCpy(NewAlloca, /*DstAlign*/ Alignment,
                       CI->getArgOperand(OpndIdx),
                       /*SrcAlign*/ Alignment, Size);
  CI->setArgOperand(OpndIdx, NewAlloca);
}

// Creates a copy from temporarily variable(keeping value of ByVal argument)
// into the corresponding function argument location.
void TailRecursionEliminator::copyLocalTempOfByValueOperandIntoArguments(
    CallInst *CI, int OpndIdx) {
  Type *AggTy = CI->getParamByValType(OpndIdx);
  assert(AggTy);
  const DataLayout &DL = F.getDataLayout();

  // Get alignment of byVal operand.
  Align Alignment(CI->getParamAlign(OpndIdx).valueOrOne());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  for (unsigned I = 0, E = CI->arg_size(); I != E; ++I) {
    if (CI->isByValArgument(I)) {
      copyLocalTempOfByValueOperandIntoArguments(CI, I);
      // When eliminating a tail call, we modify the values of the arguments.
      // Therefore, if the byval parameter has a readonly attribute, we have to
      // remove it. It is safe because, from the perspective of a caller, the
      // byval parameter is always treated as "readonly," even if the readonly
      // attribute is removed.
      F.removeParamAttr(I, Attribute::ReadOnly);
      ArgumentPHIs[I]->addIncoming(F.getArg(I), BB);
    } else
      ArgumentPHIs[I]->addIncoming(CI->getArgOperand(I), BB);
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include eliminateCall, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 eliminateCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 717-804

```cpp
  if (AccRecInstr) {
    insertAccumulator(AccRecInstr);

    // Rewrite the accumulator recursion instruction so that it does not use
    // the result of the call anymore, instead, use the PHI node we just
    // inserted.
    AccRecInstr->setOperand(AccRecInstr->getOperand(0) != CI, AccPN);
  }

  // Update our return value tracking
  if (RetPN) {
    if (Ret->getReturnValue() == CI || AccRecInstr) {
      // Defer selecting a return value
      RetPN->addIncoming(RetPN, BB);
      RetKnownPN->addIncoming(RetKnownPN, BB);
    } else {
      // We found a return value we want to use, insert a select instruction to
      // select it if we don't already know what our return value will be and
      // store the result in our return value PHI node.
      SelectInst *SI =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // If the PHI Node is a dynamic constant, replace it with the value it is.
    if (Value *PNV = simplifyInstruction(PN, F.getDataLayout())) {
      PN->replaceAllUsesWith(PNV);
      PN->eraseFromParent();
    }
  }

  if (RetPN) {
    if (RetSelects.empty()) {
      // If we didn't insert any select instructions, then we know we didn't
      // store a return value and we can remove the PHI nodes we inserted.
      RetPN->dropAllReferences();
      RetPN->eraseFromParent();

```
- EN: Core entities appearing here include cleanupAndFinalize, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 cleanupAndFinalize，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 805-892

```cpp
      RetKnownPN->dropAllReferences();
      RetKnownPN->eraseFromParent();

      if (AccPN) {
        // We need to insert a copy of our accumulator instruction before any
        // return in the function, and return its result instead.
        Instruction *AccRecInstr = AccumulatorRecursionInstr;
        for (BasicBlock &BB : F) {
          ReturnInst *RI = dyn_cast<ReturnInst>(BB.getTerminator());
          if (!RI)
            continue;

          Instruction *AccRecInstrNew = AccRecInstr->clone();
          AccRecInstrNew->setName("accumulator.ret.tr");
          AccRecInstrNew->setOperand(AccRecInstr->getOperand(0) == AccPN,
                                     RI->getOperand(0));
          AccRecInstrNew->insertBefore(RI->getIterator());
          AccRecInstrNew->dropLocation();
          RI->setOperand(0, AccRecInstrNew);
        }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    // If all predecessors of Succ have been eliminated by
    // FoldReturnIntoUncondBranch, delete it.  It is important to empty it,
    // because the ret instruction in there is still using a value which
    // eliminateCall will attempt to remove.  This block can only contain
    // instructions that can't have uses, therefore it is safe to remove.
    if (pred_empty(Succ))
      DTU.deleteBB(Succ);

    eliminateCall(CI);
    return true;
  } else if (isa<ReturnInst>(TI)) {
    CallInst *CI = findTRECandidate(&BB);

```
- EN: Core entities appearing here include processBlock, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 processBlock，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 893-981

```cpp
    if (CI)
      return eliminateCall(CI);
  }

  return false;
}

bool TailRecursionEliminator::eliminate(Function &F,
                                        const TargetTransformInfo *TTI,
                                        AliasAnalysis *AA,
                                        OptimizationRemarkEmitter *ORE,
                                        DomTreeUpdater &DTU,
                                        BlockFrequencyInfo *BFI) {
  if (F.getFnAttribute("disable-tail-calls").getValueAsBool())
    return false;

  bool MadeChange = false;
  MadeChange |= markTails(F, ORE);

  // If this function is a varargs function, we won't be able to PHI the args
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

char TailCallElim::ID = 0;
INITIALIZE_PASS_BEGIN(TailCallElim, "tailcallelim", "Tail Call Elimination",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(OptimizationRemarkEmitterWrapperPass)
INITIALIZE_PASS_END(TailCallElim, "tailcallelim", "Tail Call Elimination",
                    false, false)

// Public interface to the TailCallElimination pass
FunctionPass *llvm::createTailCallEliminationPass() {
  return new TailCallElim();
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include TailCallElim, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AliasAnalysis, BlockFrequencyInfo, DominatorTree.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 TailCallElim，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AliasAnalysis, BlockFrequencyInfo, DominatorTree 等分析结果。

### Lines 982-1010

```cpp
PreservedAnalyses TailCallElimPass::run(Function &F,
                                        FunctionAnalysisManager &AM) {

  TargetTransformInfo &TTI = AM.getResult<TargetIRAnalysis>(F);
  AliasAnalysis &AA = AM.getResult<AAManager>(F);
  // This must come first. It needs the 2 analyses, meaning, if it came after
  // the lines asking for the cached result, should they be nullptr (which, in
  // the case of the PDT, is likely), updates to the trees would be missed.
  auto *BFI = (!ForceDisableBFI && UpdateFunctionEntryCount &&
               F.getEntryCount().has_value() && F.getEntryCount()->getCount())
                  ? &AM.getResult<BlockFrequencyAnalysis>(F)
                  : nullptr;
  auto &ORE = AM.getResult<OptimizationRemarkEmitterAnalysis>(F);
  auto *DT = AM.getCachedResult<DominatorTreeAnalysis>(F);
  auto *PDT = AM.getCachedResult<PostDominatorTreeAnalysis>(F);
  // There is no noticable performance difference here between Lazy and Eager
  // UpdateStrategy based on some test results. It is feasible to switch the
  // UpdateStrategy to Lazy if we find it profitable later.
  DomTreeUpdater DTU(DT, PDT, DomTreeUpdater::UpdateStrategy::Eager);
  bool Changed =
      TailRecursionEliminator::eliminate(F, &TTI, &AA, &ORE, DTU, BFI);

  if (!Changed)
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  PA.preserve<PostDominatorTreeAnalysis>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, AliasAnalysis, DominatorTree, DomTreeUpdater.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAManager, AliasAnalysis, DominatorTree, DomTreeUpdater 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `AllocaDerivedValueTracker, VisitType, TailRecursionEliminator, TailCallElim, canTRE, walk, callUsesLocalStack, markTails` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`AllocaDerivedValueTracker, VisitType, TailRecursionEliminator, TailCallElim, canTRE, walk, callUsesLocalStack, markTails` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AAManager, AliasAnalysis, BlockFrequencyInfo, DataLayout, DominatorTree, DomTreeUpdater, OptimizationRemarkEmitter` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AAManager, AliasAnalysis, BlockFrequencyInfo, DataLayout, DominatorTree, DomTreeUpdater, OptimizationRemarkEmitter` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `ForceDisableBFI` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `ForceDisableBFI` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/PostDominators.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/PostDominators.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/Statistic.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/Statistic.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AAManager`, `AliasAnalysis`, `BlockFrequencyInfo`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `OptimizationRemarkEmitter` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AAManager`, `AliasAnalysis`, `BlockFrequencyInfo`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `OptimizationRemarkEmitter` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
