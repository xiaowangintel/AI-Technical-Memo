# InlineFunction.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/InlineFunction.cpp` | `llvm/lib/Transforms/Utils/InlineFunction.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements code to perform function inlining within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 InlineFunction 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-229

```cpp
//===- InlineFunction.cpp - Code to perform function inlining -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements inlining of a function into a call site, resolving
// parameters and the return value as appropriate.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/iterator_range.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    PHINode *InnerPHI = PHINode::Create(OuterPHI->getType(), PHICapacity,
                                        OuterPHI->getName() + ".lpad-body");
    InnerPHI->insertBefore(InsertPoint);
    OuterPHI->replaceAllUsesWith(InnerPHI);
    InnerPHI->addIncoming(OuterPHI, OuterResumeDest);
  }

  // Create a PHI for the exception values.
  InnerEHValuesPHI =
      PHINode::Create(CallerLPad->getType(), PHICapacity, "eh.lpad-body");
  InnerEHValuesPHI->insertBefore(InsertPoint);
  CallerLPad->replaceAllUsesWith(InnerEHValuesPHI);
  InnerEHValuesPHI->addIncoming(CallerLPad, OuterResumeDest);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 230-476

```cpp
  // All done.
  return InnerResumeDest;
}

/// Forward the 'resume' instruction to the caller's landing pad block.
/// When the landing pad block has only one predecessor, this is a simple
/// branch. When there is more than one predecessor, we need to split the
/// landing pad block after the landingpad instruction and jump to there.
void LandingPadInliningInfo::forwardResume(
    ResumeInst *RI, SmallPtrSetImpl<LandingPadInst *> &InlinedLPads) {
  BasicBlock *Dest = getInnerResumeDest();
  BasicBlock *Src = RI->getParent();

  auto *BI = UncondBrInst::Create(Dest, Src);
  BI->setDebugLoc(RI->getDebugLoc());

  // Update the PHIs in the destination. They were inserted in an order which
  // makes this work.
  addIncomingPHIValuesForInto(Src, Dest);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (AncestorMemo == MemoMap.end()) {
      UnwindDestToken = getUnwindDestTokenHelper(AncestorPad, MemoMap);
    } else {
      UnwindDestToken = AncestorMemo->second;
    }
    if (UnwindDestToken)
      break;
    LastUselessPad = AncestorPad;
    MemoMap[LastUselessPad] = nullptr;
#ifndef NDEBUG
    TempMemos.insert(LastUselessPad);
#endif
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 477-709

```cpp
  // We know that getUnwindDestTokenHelper was called on LastUselessPad and
  // returned nullptr (and likewise for EHPad and any of its ancestors up to
  // LastUselessPad), so LastUselessPad has no information from below.  Since
  // getUnwindDestTokenHelper must investigate all downward paths through
  // no-information nodes to prove that a node has no information like this,
  // and since any time it finds information it records it in the MemoMap for
  // not just the immediately-containing funclet but also any ancestors also
  // exited, it must be the case that, walking downward from LastUselessPad,
  // visiting just those nodes which have not been mapped to an unwind dest
  // by getUnwindDestTokenHelper (the nullptr TempMemos notwithstanding, since
  // they are just used to keep getUnwindDestTokenHelper from repeating work),
  // any node visited must have been exhaustively searched with no information
  // for it found.
  SmallVector<Instruction *, 8> Worklist(1, LastUselessPad);
  while (!Worklist.empty()) {
    Instruction *UselessPad = Worklist.pop_back_val();
    auto Memo = MemoMap.find(UselessPad);
    if (Memo != MemoMap.end() && Memo->second) {
      // Here the name 'UselessPad' is a bit of a misnomer, because we've found
      // that it is a funclet that does have information about unwinding to
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    UnwindDestPHIValues.push_back(PHI.getIncomingValueForBlock(InvokeBB));
  }

  // Add incoming-PHI values to the unwind destination block for the given basic
  // block, using the values for the original invoke's source block.
  auto UpdatePHINodes = [&](BasicBlock *Src) {
    BasicBlock::iterator I = UnwindDest->begin();
    for (Value *V : UnwindDestPHIValues) {
      PHINode *PHI = cast<PHINode>(I);
      PHI->addIncoming(V, Src);
      ++I;
    }
  };

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 710-939

```cpp
  // This connects all the instructions which 'unwind to caller' to the invoke
  // destination.
  UnwindDestMemoTy FuncletUnwindMap;
  for (Function::iterator BB = FirstNewBlock->getIterator(), E = Caller->end();
       BB != E; ++BB) {
    if (auto *CRI = dyn_cast<CleanupReturnInst>(BB->getTerminator())) {
      if (CRI->unwindsToCaller()) {
        auto *CleanupPad = CRI->getCleanupPad();
        CleanupReturnInst::Create(CleanupPad, UnwindDest, CRI->getIterator());
        CRI->eraseFromParent();
        UpdatePHINodes(&*BB);
        // Finding a cleanupret with an unwind destination would confuse
        // subsequent calls to getUnwindDestToken, so map the cleanuppad
        // to short-circuit any such calls and recognize this as an "unwind
        // to caller" cleanup.
        assert(!FuncletUnwindMap.count(CleanupPad) ||
               isa<ConstantTokenNone>(FuncletUnwindMap[CleanupPad]));
        FuncletUnwindMap[CleanupPad] =
            ConstantTokenNone::get(Caller->getContext());
      }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (!OrigCall || !ClonedCall)
      continue;
    // If the inlined callsite did not have any callsite metadata, then it isn't
    // involved in any profiled call contexts, and we can remove any memprof
    // metadata on the cloned call.
    if (!CallsiteMD) {
      removeMemProfMetadata(ClonedCall);
      removeCallsiteMetadata(ClonedCall);
      continue;
    }
    propagateMemProfHelper(OrigCall, ClonedCall, CallsiteMD, ORE);
  }
}

```
- EN: Core entities appearing here include removeMemProfMetadata, removeCallsiteMetadata, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 removeMemProfMetadata, removeCallsiteMetadata，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 940-1167

```cpp
/// When inlining a call site that has !llvm.mem.parallel_loop_access,
/// !llvm.access.group, !alias.scope or !noalias metadata, that metadata should
/// be propagated to all memory-accessing cloned instructions.
static void PropagateCallSiteMetadata(CallBase &CB, Function::iterator FStart,
                                      Function::iterator FEnd) {
  MDNode *MemParallelLoopAccess =
      CB.getMetadata(LLVMContext::MD_mem_parallel_loop_access);
  MDNode *AccessGroup = CB.getMetadata(LLVMContext::MD_access_group);
  MDNode *AliasScope = CB.getMetadata(LLVMContext::MD_alias_scope);
  MDNode *NoAlias = CB.getMetadata(LLVMContext::MD_noalias);
  if (!MemParallelLoopAccess && !AccessGroup && !AliasScope && !NoAlias)
    return;

  for (BasicBlock &BB : make_range(FStart, FEnd)) {
    for (Instruction &I : BB) {
      // This metadata is only relevant for instructions that access memory.
      if (!I.mayReadOrWriteMemory())
        continue;

      if (MemParallelLoopAccess) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }
  }
}

/// If the inlined function has noalias arguments,
/// then add new alias scopes for each noalias argument, tag the mapped noalias
/// parameters with noalias metadata specifying the new scope, and tag all
/// non-derived loads, stores and memory intrinsics with the new alias scopes.
static void AddAliasScopeMetadata(CallBase &CB, ValueToValueMapTy &VMap,
                                  const DataLayout &DL, AAResults *CalleeAAR,
                                  ClonedCodeInfo &InlinedFunctionInfo) {
  if (!EnableNoAliasConversion)
    return;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ScopedAliasMetadataDeepCloner, addRecursiveMetadataUses, clone, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ScopedAliasMetadataDeepCloner, addRecursiveMetadataUses, clone，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。

### Lines 1168-1395

```cpp
  const Function *CalledFunc = CB.getCalledFunction();
  SmallVector<const Argument *, 4> NoAliasArgs;

  for (const Argument &Arg : CalledFunc->args())
    if (CB.paramHasAttr(Arg.getArgNo(), Attribute::NoAlias) && !Arg.use_empty())
      NoAliasArgs.push_back(&Arg);

  if (NoAliasArgs.empty())
    return;

  // To do a good job, if a noalias variable is captured, we need to know if
  // the capture point dominates the particular use we're considering.
  DominatorTree DT;
  DT.recalculate(const_cast<Function&>(*CalledFunc));

  // noalias indicates that pointer values based on the argument do not alias
  // pointer values which are not based on it. So we add a new "scope" for each
  // noalias function argument. Accesses using pointers based on that argument
  // become part of that alias scope, accesses using pointers not based on that
  // argument are tagged as noalias with that scope.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // (because there is some pointer of unknown origin involved and the
      // other access might also depend on this pointer). We also cannot add
      // scopes to arbitrary functions unless we know they don't access any
      // non-parameter pointer-values.
      bool CanAddScopes = !UsesAliasingPtr;
      if (CanAddScopes && IsFuncCall)
        CanAddScopes = IsArgMemOnlyCall;

      if (CanAddScopes)
        for (const Argument *A : NoAliasArgs) {
          if (ObjSet.count(A))
            Scopes.push_back(NewScopes[A]);
        }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1396-1617

```cpp
      if (!Scopes.empty())
        NI->setMetadata(
            LLVMContext::MD_alias_scope,
            MDNode::concatenate(NI->getMetadata(LLVMContext::MD_alias_scope),
                                MDNode::get(CalledFunc->getContext(), Scopes)));
    }
  }
}

static bool MayContainThrowingOrExitingCallAfterCB(CallBase *Begin,
                                                   ReturnInst *End) {

  assert(Begin->getParent() == End->getParent() &&
         "Expected to be in same basic block!");
  auto BeginIt = Begin->getIterator();
  assert(BeginIt != End->getIterator() && "Non-empty BB has empty iterator");
  return !llvm::isGuaranteedToTransferExecutionToSuccessor(
      ++BeginIt, End->getIterator(), InlinerAttributeWindow + 1);
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto &Context = CalledFunction->getContext();

  for (auto &BB : *CalledFunction) {
    auto *RI = dyn_cast<ReturnInst>(BB.getTerminator());
    if (!RI || !isa<CallBase>(RI->getOperand(0)))
      continue;
    auto *RetVal = cast<CallBase>(RI->getOperand(0));
    // Check that the cloned RetVal exists and is a call, otherwise we cannot
    // add the attributes on the cloned RetVal. Simplification during inlining
    // could have transformed the cloned instruction.
    auto *NewRetVal = dyn_cast_or_null<CallBase>(VMap.lookup(RetVal));
    if (!NewRetVal)
      continue;

```
- EN: Core entities appearing here include IdentifyValidUBGeneratingAttributes, IdentifyValidPoisonGeneratingAttributes, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 IdentifyValidUBGeneratingAttributes, IdentifyValidPoisonGeneratingAttributes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1618-1843

```cpp
    // The RetVal might have be simplified during the inlining
    // process which can make propagation incorrect.
    if (InlinedFunctionInfo.isSimplified(RetVal, NewRetVal))
      continue;
    // Backward propagation of attributes to the returned value may be incorrect
    // if it is control flow dependent.
    // Consider:
    // @callee {
    //  %rv = call @foo()
    //  %rv2 = call @bar()
    //  if (%rv2 != null)
    //    return %rv2
    //  if (%rv == null)
    //    exit()
    //  return %rv
    // }
    // caller() {
    //   %val = call nonnull @callee()
    // }
    // Here we cannot add the nonnull attribute on either foo or bar. So, we
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Alignment = std::max(Alignment, *ByValAlignment);

  AllocaInst *NewAlloca =
      new AllocaInst(ByValType, Arg->getType()->getPointerAddressSpace(),
                     nullptr, Alignment, Arg->getName());
  NewAlloca->setDebugLoc(DebugLoc::getCompilerGenerated());
  NewAlloca->insertBefore(Caller->begin()->begin());
  IFI.StaticAllocas.push_back(NewAlloca);

  // Uses of the argument in the function should use our new alloca
  // instead.
  return NewAlloca;
}

```
- EN: Core entities appearing here include AddAlignmentAssumptions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 AddAlignmentAssumptions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1844-2073

```cpp
// Check whether this Value is used by a lifetime intrinsic.
static bool isUsedByLifetimeMarker(Value *V) {
  for (User *U : V->users())
    if (isa<LifetimeIntrinsic>(U))
      return true;
  return false;
}

// Check whether the given alloca already has
// lifetime.start or lifetime.end intrinsics.
static bool hasLifetimeMarkers(AllocaInst *AI) {
  Type *Ty = AI->getType();
  Type *Int8PtrTy =
      PointerType::get(Ty->getContext(), Ty->getPointerAddressSpace());
  if (Ty == Int8PtrTy)
    return isUsedByLifetimeMarker(AI);

  // Do a scan to find all the casts to i8*.
  for (User *U : AI->users()) {
    if (U->getType() != Int8PtrTy) continue;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// otherwise a function inlined more than once into the same function
/// will cause DIAssignID to be shared by many instructions.
static void fixupAssignments(Function::iterator Start, Function::iterator End) {
  DenseMap<DIAssignID *, DIAssignID *> Map;
  // Loop over all the inlined instructions. If we find a DIAssignID
  // attachment or use, replace it with a new version.
  for (auto BBI = Start; BBI != End; ++BBI) {
    for (Instruction &I : *BBI)
      at::remapAssignID(Map, I);
  }
}
#undef DEBUG_TYPE
#define DEBUG_TYPE "inline-function"

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include isUsedByLifetimeMarker, hasLifetimeMarkers, allocaWouldBeStaticInEntry, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, LoopInfo.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 isUsedByLifetimeMarker, hasLifetimeMarkers, allocaWouldBeStaticInEntry，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, LoopInfo 等分析结果。

### Lines 2074-2304

```cpp
/// Update the block frequencies of the caller after a callee has been inlined.
///
/// Each block cloned into the caller has its block frequency scaled by the
/// ratio of CallSiteFreq/CalleeEntryFreq. This ensures that the cloned copy of
/// callee's entry block gets the same frequency as the callsite block and the
/// relative frequencies of all cloned blocks remain the same after cloning.
static void updateCallerBFI(BasicBlock *CallSiteBlock,
                            const ValueToValueMapTy &VMap,
                            BlockFrequencyInfo *CallerBFI,
                            BlockFrequencyInfo *CalleeBFI,
                            const BasicBlock &CalleeEntryBlock) {
  SmallPtrSet<BasicBlock *, 16> ClonedBBs;
  for (auto Entry : VMap) {
    if (!isa<BasicBlock>(Entry.first) || !Entry.second)
      continue;
    auto *OrigBB = cast<BasicBlock>(Entry.first);
    auto *ClonedBB = cast<BasicBlock>(Entry.second);
    BlockFrequency Freq = CalleeBFI->getBlockFreq(OrigBB);
    if (!ClonedBBs.insert(ClonedBB).second) {
      // Multiple blocks in the callee might get mapped to one cloned block in
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// it's not worth updating those.
static std::pair<std::vector<int64_t>, std::vector<int64_t>>
remapIndices(Function &Caller, BasicBlock *StartBB,
             PGOContextualProfile &CtxProf, uint32_t CalleeCounters,
             uint32_t CalleeCallsites) {
  // We'll allocate a new ID to imported callsite counters and callsites. We're
  // using -1 to indicate a counter we delete. Most likely the entry ID, for
  // example, will be deleted - we don't want 2 IDs in the same BB, and the
  // entry would have been cloned in the callsite's old BB.
  std::vector<int64_t> CalleeCounterMap;
  std::vector<int64_t> CalleeCallsiteMap;
  CalleeCounterMap.resize(CalleeCounters, -1);
  CalleeCallsiteMap.resize(CalleeCallsites, -1);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as BlockFrequencyInfo, ProfileSummaryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 BlockFrequencyInfo, ProfileSummaryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2305-2527

```cpp
  auto RewriteInstrIfNeeded = [&](InstrProfIncrementInst &Ins) -> bool {
    if (Ins.getNameValue() == &Caller)
      return false;
    const auto OldID = static_cast<uint32_t>(Ins.getIndex()->getZExtValue());
    if (CalleeCounterMap[OldID] == -1)
      CalleeCounterMap[OldID] = CtxProf.allocateNextCounterIndex(Caller);
    const auto NewID = static_cast<uint32_t>(CalleeCounterMap[OldID]);

    Ins.setNameValue(&Caller);
    Ins.setIndex(NewID);
    return true;
  };

  auto RewriteCallsiteInsIfNeeded = [&](InstrProfCallsite &Ins) -> bool {
    if (Ins.getNameValue() == &Caller)
      return false;
    const auto OldID = static_cast<uint32_t>(Ins.getIndex()->getZExtValue());
    if (CalleeCallsiteMap[OldID] == -1)
      CalleeCallsiteMap[OldID] = CtxProf.allocateNextCallsiteIndex(Caller);
    const auto NewID = static_cast<uint32_t>(CalleeCallsiteMap[OldID]);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return Ret;
}

llvm::InlineResult llvm::CanInlineCallSite(const CallBase &CB,
                                           InlineFunctionInfo &IFI) {
  assert(CB.getParent() && CB.getFunction() && "Instruction not in function!");

  // FIXME: we don't inline callbr yet.
  if (isa<CallBrInst>(CB))
    return InlineResult::failure("We don't inline callbr yet.");

  // If IFI has any state in it, zap it before we fill it in.
  IFI.reset();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2528-2756

```cpp
  Function *CalledFunc = CB.getCalledFunction();
  if (!CalledFunc ||               // Can't inline external function or indirect
      CalledFunc->isDeclaration()) // call!
    return InlineResult::failure("external or indirect");

  // Don't inline if we've already inlined this callee through this call site
  // before to prevent infinite inlining through mutually recursive functions.
  if (MDNode *InlineHistory = CB.getMetadata(LLVMContext::MD_inline_history)) {
    for (const auto &Op : InlineHistory->operands()) {
      if (auto *MD = dyn_cast_or_null<ValueAsMetadata>(Op)) {
        if (MD->getValue() == CalledFunc) {
          return InlineResult::failure("inline history");
        }
      }
    }
  }

  // The inliner does not know how to inline through calls with operand bundles
  // in general ...
  if (CB.hasOperandBundles()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // the function, so that we do not get in trouble when inlining caller ==
    // callee.
    ScopedAliasMetadataDeepCloner SAMetadataCloner(CB.getCalledFunction());

    auto &DL = Caller->getDataLayout();

    // Calculate the vector of arguments to pass into the function cloner, which
    // matches up the formal to the actual argument values.
    auto AI = CB.arg_begin();
    unsigned ArgNo = 0;
    for (Function::arg_iterator I = CalledFunc->arg_begin(),
         E = CalledFunc->arg_end(); I != E; ++I, ++AI, ++ArgNo) {
      Value *ActualArg = *AI;

```
- EN: Core entities appearing here include getConvergenceEntry, ByValInit, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout, OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getConvergenceEntry, ByValInit，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout, OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2757-2981

```cpp
      // When byval arguments actually inlined, we need to make the copy implied
      // by them explicit.  However, we don't do this if the callee is readonly
      // or readnone, because the copy would be unneeded: the callee doesn't
      // modify the struct.
      if (CB.isByValArgument(ArgNo)) {
        ActualArg = HandleByValArgument(CB.getParamByValType(ArgNo), ActualArg,
                                        &CB, CalledFunc, IFI,
                                        CalledFunc->getParamAlign(ArgNo));
        if (ActualArg != *AI)
          ByValInits.push_back({ActualArg, (Value *)*AI,
                                CB.getParamAlign(ArgNo),
                                CB.getParamByValType(ArgNo)});
      }

      VMap[&*I] = ActualArg;
    }

    // TODO: Remove this when users have been updated to the assume bundles.
    // Add alignment assumptions if necessary. We do this before the inlined
    // instructions are actually cloned into the caller so that we can easily
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

      // Transfer all of the allocas over in a block.  Using splice means
      // that the instructions aren't removed from the symbol table, then
      // reinserted.
      I.setTailBit(true);
      Caller->getEntryBlock().splice(InsertPoint, &*FirstNewBlock,
                                     AI->getIterator(), I);
    }
  }

  // If the call to the callee cannot throw, set the 'nounwind' flag on any
  // calls that we inline.
  bool MarkNoUnwind = CB.doesNotThrow();

```
- EN: Core entities appearing here include allocaWouldBeStaticInEntry, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 allocaWouldBeStaticInEntry，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2982-3210

```cpp
  SmallVector<Value*,4> VarArgsToForward;
  SmallVector<AttributeSet, 4> VarArgsAttrs;
  for (unsigned i = CalledFunc->getFunctionType()->getNumParams();
       i < CB.arg_size(); i++) {
    VarArgsToForward.push_back(CB.getArgOperand(i));
    VarArgsAttrs.push_back(CB.getAttributes().getParamAttrs(i));
  }

  bool InlinedMustTailCalls = false, InlinedDeoptimizeCalls = false;
  if (InlinedFunctionInfo.ContainsCalls) {
    CallInst::TailCallKind CallSiteTailKind = CallInst::TCK_None;
    if (CallInst *CI = dyn_cast<CallInst>(&CB))
      CallSiteTailKind = CI->getTailCallKind();

    // For inlining purposes, the "notail" marker is the same as no marker.
    if (CallSiteTailKind == CallInst::TCK_NoTail)
      CallSiteTailKind = CallInst::TCK_None;

    for (Function::iterator BB = FirstNewBlock, E = Caller->end(); BB != E;
         ++BB) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          NormalReturns.push_back(RI);
          continue;
        }

        // The calling convention on the deoptimize call itself may be bogus,
        // since the code we're inlining may have undefined behavior (and may
        // never actually execute at runtime); but all
        // @llvm.experimental.deoptimize declarations have to have the same
        // calling convention in a well-formed module.
        auto CallingConv = DeoptCall->getCalledFunction()->getCallingConv();
        NewDeoptIntrinsic->setCallingConv(CallingConv);
        auto *CurBB = RI->getParent();
        RI->eraseFromParent();

```
- EN: Core entities appearing here include erase_if, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 erase_if，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3211-3436

```cpp
        SmallVector<Value *, 4> CallArgs(DeoptCall->args());

        SmallVector<OperandBundleDef, 1> OpBundles;
        DeoptCall->getOperandBundlesAsDefs(OpBundles);
        auto DeoptAttributes = DeoptCall->getAttributes();
        DeoptCall->eraseFromParent();
        assert(!OpBundles.empty() &&
               "Expected at least the deopt operand bundle");

        IRBuilder<> Builder(CurBB);
        CallInst *NewDeoptCall =
            Builder.CreateCall(NewDeoptIntrinsic, CallArgs, OpBundles);
        NewDeoptCall->setCallingConv(CallingConv);
        NewDeoptCall->setAttributes(DeoptAttributes);
        if (NewDeoptCall->getType()->isVoidTy())
          Builder.CreateRetVoid();
        else
          Builder.CreateRet(NewDeoptCall);
        // Since the ret type is changed, remove the incompatible attributes.
        NewDeoptCall->removeRetAttrs(AttributeFuncs::typeIncompatible(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // PHI node as their operand.
      CB.replaceAllUsesWith(PHI);
    }

    // Loop over all of the return instructions adding entries to the PHI node
    // as appropriate.
    if (PHI) {
      for (ReturnInst *RI : Returns) {
        assert(RI->getReturnValue()->getType() == PHI->getType() &&
               "Ret value not consistent in function!");
        PHI->addIncoming(RI->getReturnValue(), RI->getParent());
      }
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3437-3538

```cpp
    // Add a branch to the merge points and remove return instructions.
    DebugLoc Loc;
    for (ReturnInst *RI : Returns) {
      UncondBrInst *BI = UncondBrInst::Create(AfterCallBB, RI->getIterator());
      Loc = RI->getDebugLoc();
      BI->setDebugLoc(Loc);
      RI->eraseFromParent();
    }
    // We need to set the debug location to *somewhere* inside the
    // inlined function. The line number may be nonsensical, but the
    // instruction will at least be associated with the right
    // function.
    if (CreatedBranchToNormalDest)
      CreatedBranchToNormalDest->setDebugLoc(Loc);
  } else if (!Returns.empty()) {
    // Otherwise, if there is exactly one return value, just replace anything
    // using the return value of the call with the computed value.
    if (!CB.use_empty()) {
      if (&CB == Returns[0]->getReturnValue())
        CB.replaceAllUsesWith(PoisonValue::get(CB.getType()));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

llvm::InlineResult llvm::InlineFunction(
    CallBase &CB, InlineFunctionInfo &IFI, bool MergeAttributes,
    AAResults *CalleeAAR, bool InsertLifetime, bool TrackInlineHistory,
    Function *ForwardVarArgsTo, OptimizationRemarkEmitter *ORE) {
  llvm::InlineResult Result = CanInlineCallSite(CB, IFI);
  if (Result.isSuccess()) {
    InlineFunctionImpl(CB, IFI, MergeAttributes, CalleeAAR, InsertLifetime,
                       TrackInlineHistory, ForwardVarArgsTo, ORE);
  }

  return Result;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, AssumptionCache, DataLayout, OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, AssumptionCache, DataLayout, OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LandingPadInliningInfo, ScopedAliasMetadataDeepCloner, ByValInit, OuterResumeDest, addIncomingPHIValuesFor, addIncomingPHIValuesForInto, removeMemProfMetadata, removeCallsiteMetadata` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LandingPadInliningInfo, ScopedAliasMetadataDeepCloner, ByValInit, OuterResumeDest, addIncomingPHIValuesFor, addIncomingPHIValuesForInto, removeMemProfMetadata, removeCallsiteMetadata` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AliasAnalysis, AssumptionCache, BlockFrequencyInfo, CallGraph, DataLayout, DominatorTree, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AliasAnalysis, AssumptionCache, BlockFrequencyInfo, CallGraph, DataLayout, DominatorTree, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnableNoAliasConversion, UseNoAliasIntrinsic, PreserveAlignmentAssumptions, InlinerAttributeWindow` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnableNoAliasConversion, UseNoAliasIntrinsic, PreserveAlignmentAssumptions, InlinerAttributeWindow` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/CallGraph.h`, `llvm/Analysis/CaptureTracking.h`, `llvm/Analysis/CtxProfAnalysis.h`, `llvm/Analysis/IndirectCallVisitor.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryProfileInfo.h`, `llvm/Analysis/ObjCARCAnalysisUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/CallGraph.h`, `llvm/Analysis/CaptureTracking.h`, `llvm/Analysis/CtxProfAnalysis.h`, `llvm/Analysis/IndirectCallVisitor.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryProfileInfo.h`, `llvm/Analysis/ObjCARCAnalysisUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/AttributeMask.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/AttributeMask.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DebugLoc.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/iterator_range.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/iterator_range.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `deque`, `iterator`, `optional`, `string`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `deque`, `iterator`, `optional`, `string`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AliasAnalysis`, `AssumptionCache`, `BlockFrequencyInfo`, `CallGraph`, `DataLayout`, `DominatorTree`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AliasAnalysis`, `AssumptionCache`, `BlockFrequencyInfo`, `CallGraph`, `DataLayout`, `DominatorTree`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
