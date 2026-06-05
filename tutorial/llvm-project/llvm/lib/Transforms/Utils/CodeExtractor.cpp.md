# CodeExtractor.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CodeExtractor.cpp` | `llvm/lib/Transforms/Utils/CodeExtractor.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements pull code region into a new function within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CodeExtractor 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-139

```cpp
//===- CodeExtractor.cpp - Pull code region into a new function -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the interface to tear out a code region, such as an
// individual loop or a parallel section, into a new function, replacing it with
// a call to the new function.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/CodeExtractor.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallPtrSet.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }

    // All catch handlers of a catchswitch instruction as well as the unwind
    // destination must be in the subgraph.
    if (const auto *CSI = dyn_cast<CatchSwitchInst>(I)) {
      if (auto *UBB = CSI->getUnwindDest())
        if (!Result.count(UBB))
          return false;
      for (const auto *HBB : CSI->handlers())
        if (!Result.count(const_cast<BasicBlock*>(HBB)))
          return false;
      continue;
    }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 140-284

```cpp
    // Make sure that entire catch handler is within subgraph. It is sufficient
    // to check that catch return's block is in the list.
    if (const auto *CPI = dyn_cast<CatchPadInst>(I)) {
      for (const auto *U : CPI->users())
        if (const auto *CRI = dyn_cast<CatchReturnInst>(U))
          if (!Result.count(const_cast<BasicBlock*>(CRI->getParent())))
            return false;
      continue;
    }

    // And do similar checks for cleanup handler - the entire handler must be
    // in subgraph which is going to be extracted. For cleanup return should
    // additionally check that the unwind destination is also in the subgraph.
    if (const auto *CPI = dyn_cast<CleanupPadInst>(I)) {
      for (const auto *U : CPI->users())
        if (const auto *CRI = dyn_cast<CleanupReturnInst>(U))
          if (!Result.count(const_cast<BasicBlock*>(CRI->getParent())))
            return false;
      continue;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      DeallocationBlocks(DeallocationBlocks), AllowVarArgs(AllowVarArgs),
      Blocks(buildExtractionBlockSet(BBs, DT, AllowVarArgs, AllowAlloca)),
      Suffix(Suffix), ArgsInZeroAddressSpace(ArgsInZeroAddressSpace),
      VoidReturnWithSingleOutput(VoidReturnWithSingleOutput) {}

/// definedInRegion - Return true if the specified value is defined in the
/// extracted region.
static bool definedInRegion(const SetVector<BasicBlock *> &Blocks, Value *V) {
  if (Instruction *I = dyn_cast<Instruction>(V))
    if (Blocks.count(I->getParent()))
      return true;
  return false;
}

```
- EN: Core entities appearing here include isAlignmentPreservedForAddrCast, definedInRegion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, BlockFrequencyInfo, BranchProbabilityInfo, DominatorTree.
- CN: 此处出现的核心实体包括 isAlignmentPreservedForAddrCast, definedInRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, BlockFrequencyInfo, BranchProbabilityInfo, DominatorTree 等分析结果。

### Lines 285-433

```cpp
/// definedInCaller - Return true if the specified value is defined in the
/// function being code extracted, but not in the region being extracted.
/// These values must be passed in as live-ins to the function.
static bool definedInCaller(const SetVector<BasicBlock *> &Blocks, Value *V) {
  if (isa<Argument>(V)) return true;
  if (Instruction *I = dyn_cast<Instruction>(V))
    if (!Blocks.count(I->getParent()))
      return true;
  return false;
}

static BasicBlock *getCommonExitBlock(const SetVector<BasicBlock *> &Blocks) {
  BasicBlock *CommonExitBlock = nullptr;
  auto hasNonCommonExitSucc = [&](BasicBlock *Block) {
    for (auto *Succ : successors(Block)) {
      // Internal edges, ok.
      if (Blocks.count(Succ))
        continue;
      if (!CommonExitBlock) {
        CommonExitBlock = Succ;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (!Phi)
        break;
      if (!FirstPhi) {
        FirstPhi = Phi;
        break;
      }
    }
    return FirstPhi;
  };
  // If there are any phi nodes, the single pred either exists or has already
  // be created before code extraction.
  assert(!getFirstPHI(CommonExitBlock) && "Phi not expected");
#endif

```
- EN: Core entities appearing here include definedInCaller, CodeExtractorAnalysisCache, findSideEffectInfoForBlock, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 definedInCaller, CodeExtractorAnalysisCache, findSideEffectInfoForBlock，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 434-571

```cpp
  BasicBlock *NewExitBlock =
      CommonExitBlock->splitBasicBlock(CommonExitBlock->getFirstNonPHIIt());

  for (BasicBlock *Pred :
       llvm::make_early_inc_range(predecessors(CommonExitBlock))) {
    if (Blocks.count(Pred))
      continue;
    Pred->getTerminator()->replaceUsesOfWith(CommonExitBlock, NewExitBlock);
  }
  // Now add the old exit block to the outline region.
  Blocks.insert(CommonExitBlock);
  return CommonExitBlock;
}

Instruction *CodeExtractor::allocateVar(IRBuilder<>::InsertPoint AllocaIP,
                                        Type *VarType, const Twine &Name,
                                        AddrSpaceCastInst **CastedAlloc) {
  const DataLayout &DL = AllocaIP.getBlock()->getModule()->getDataLayout();
  Instruction *Alloca = new AllocaInst(VarType, DL.getAllocaAddrSpace(),
                                       nullptr, Name, AllocaIP.getPoint());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      LLVM_DEBUG(dbgs() << "Sinking alloca: " << *AI << "\n");
      SinkCands.insert(AI);
      continue;
    }

    // Find bitcasts in the outlined region that have lifetime marker users
    // outside that region. Replace the lifetime marker use with an
    // outside region bitcast to avoid unnecessary alloca/reload instructions
    // and extra lifetime markers.
    SmallVector<Instruction *, 2> LifetimeBitcastUsers;
    for (User *U : AI->users()) {
      if (!definedInRegion(Blocks, U))
        continue;

```
- EN: Core entities appearing here include make_early_inc_range, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 make_early_inc_range，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 572-709

```cpp
      if (U->stripInBoundsConstantOffsets() != AI)
        continue;

      Instruction *Bitcast = cast<Instruction>(U);
      for (User *BU : Bitcast->users()) {
        auto *IntrInst = dyn_cast<LifetimeIntrinsic>(BU);
        if (!IntrInst)
          continue;

        if (definedInRegion(Blocks, IntrInst))
          continue;

        LLVM_DEBUG(dbgs() << "Replace use of extracted region bitcast"
                          << *Bitcast << " in out-of-region lifetime marker "
                          << *IntrInst << "\n");
        LifetimeBitcastUsers.push_back(IntrInst);
      }
    }

    for (Instruction *I : LifetimeBitcastUsers) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        if (!SinkCands.count(V) &&
            (definedInCaller(Blocks, V) ||
             (CollectGlobalInputs && llvm::isa<llvm::GlobalVariable>(V))))
          Inputs.insert(V);
      }

      for (User *U : II.users())
        if (!definedInRegion(Blocks, U)) {
          Outputs.insert(&II);
          break;
        }
    }
  }

```
- EN: Core entities appearing here include isEligible, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isEligible，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 710-855

```cpp
  // Reset stale state from any prior call in HotColdSplitting; the CFG may
  // have changed since.
  FuncRetVal = nullptr;
  if (!VoidReturnWithSingleOutput && !AggregateArgs && Outputs.size() == 1 &&
      getCommonExitBlock(Blocks)) {
    FuncRetVal = Outputs[0];
    Outputs.clear();
  }
}

/// severSplitPHINodesOfEntry - If a PHI node has multiple inputs from outside
/// of the region, we need to split the entry block of the region so that the
/// PHI node is easier to deal with.
void CodeExtractor::severSplitPHINodesOfEntry(BasicBlock *&Header) {
  unsigned NumPredsFromRegion = 0;
  unsigned NumPredsOutsideRegion = 0;

  if (Header != &Header->getParent()->getEntryBlock()) {
    PHINode *PN = dyn_cast<PHINode>(Header->begin());
    if (!PN) return;  // No PHI nodes.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

void CodeExtractor::splitReturnBlocks() {
  for (BasicBlock *Block : Blocks)
    if (ReturnInst *RI = dyn_cast<ReturnInst>(Block->getTerminator())) {
      BasicBlock *New =
          Block->splitBasicBlock(RI->getIterator(), Block->getName() + ".ret");
      if (DT) {
        // Old dominates New. New node dominates all other nodes dominated
        // by Old.
        DomTreeNode *OldNode = DT->getNode(Block);
        SmallVector<DomTreeNode *, 8> Children(OldNode->begin(),
                                               OldNode->end());

```
- EN: Core entities appearing here include getCommonExitBlock, severSplitPHINodesOfEntry, severSplitPHINodesOfExits, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getCommonExitBlock, severSplitPHINodesOfEntry, severSplitPHINodesOfExits，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 856-991

```cpp
        DomTreeNode *NewNode = DT->addNewBlock(New, Block);

        for (DomTreeNode *I : Children)
          DT->changeImmediateDominator(I, NewNode);
      }
    }
}

Function *CodeExtractor::constructFunctionDeclaration(
    const ValueSet &inputs, const ValueSet &outputs, BlockFrequency EntryFreq,
    const Twine &Name, ValueSet &StructValues, StructType *&StructTy) {
  LLVM_DEBUG(dbgs() << "inputs: " << inputs.size() << "\n");
  LLVM_DEBUG(dbgs() << "outputs: " << outputs.size() << "\n");

  Function *oldFunction = Blocks.front()->getParent();
  Module *M = Blocks.front()->getModule();

  // Assemble the function's parameter lists.
  std::vector<Type *> ParamTy;
  std::vector<Type *> AggParamTy;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      case Attribute::NoRecurse:
      case Attribute::InlineHint:
      case Attribute::MinSize:
      case Attribute::NoCallback:
      case Attribute::NoDuplicate:
      case Attribute::NoFree:
      case Attribute::NoImplicitFloat:
      case Attribute::NoInline:
      case Attribute::NoOutline:
      case Attribute::NonLazyBind:
      case Attribute::NoRedZone:
      case Attribute::NoUnwind:
      case Attribute::NoSanitizeBounds:
      case Attribute::NoSanitizeCoverage:
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 992-1135

```cpp
      case Attribute::NullPointerIsValid:
      case Attribute::OptimizeForDebugging:
      case Attribute::OptForFuzzing:
      case Attribute::OptimizeNone:
      case Attribute::OptimizeForSize:
      case Attribute::SafeStack:
      case Attribute::ShadowCallStack:
      case Attribute::SanitizeAddress:
      case Attribute::SanitizeMemory:
      case Attribute::SanitizeNumericalStability:
      case Attribute::SanitizeThread:
      case Attribute::SanitizeType:
      case Attribute::SanitizeHWAddress:
      case Attribute::SanitizeMemTag:
      case Attribute::SanitizeRealtime:
      case Attribute::SanitizeRealtimeBlocking:
      case Attribute::SanitizeAllocToken:
      case Attribute::SpeculativeLoadHardening:
      case Attribute::StackProtect:
      case Attribute::StackProtectReq:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// region, and insert the referenced memory into \p LifetimesStart.
///
/// The extraction region is defined by a set of blocks (\p Blocks), and a set
/// of allocas which will be moved from the caller function into the extracted
/// function (\p SunkAllocas).
static void eraseLifetimeMarkersOnInputs(const SetVector<BasicBlock *> &Blocks,
                                         const SetVector<Value *> &SunkAllocas,
                                         SetVector<Value *> &LifetimesStart) {
  for (BasicBlock *BB : Blocks) {
    for (Instruction &I : llvm::make_early_inc_range(*BB)) {
      auto *II = dyn_cast<LifetimeIntrinsic>(&I);
      if (!II)
        continue;

```
- EN: Core entities appearing here include any_of, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 any_of，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1136-1280

```cpp
      // Get the memory operand of the lifetime marker. If the underlying
      // object is a sunk alloca, or is otherwise defined in the extraction
      // region, the lifetime marker must not be erased.
      Value *Mem = II->getOperand(0);
      if (SunkAllocas.count(Mem) || definedInRegion(Blocks, Mem))
        continue;

      if (II->getIntrinsicID() == Intrinsic::lifetime_start)
        LifetimesStart.insert(Mem);
      II->eraseFromParent();
    }
  }
}

/// Insert lifetime start/end markers surrounding the call to the new function
/// for objects defined in the caller.
static void insertLifetimeMarkersSurroundingCall(
    Module *M, ArrayRef<Value *> LifetimesStart, ArrayRef<Value *> LifetimesEnd,
    CallInst *TheCall) {
  Instruction *Term = TheCall->getParent()->getTerminator();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// 1. If a debug record points to a value that has been replaced, update the
///    record to use the new value.
/// 2. If an Input value that has been replaced was used as a location of a
///    debug record in the Parent function, then materealize a similar record in
///    the new function.
/// 3. Point line locations and debug intrinsics to the new subprogram scope
/// 4. Remove intrinsics which point to values outside of the new function.
static void fixupDebugInfoPostExtraction(Function &OldFunc, Function &NewFunc,
                                         CallInst &TheCall,
                                         const SetVector<Value *> &Inputs,
                                         ArrayRef<Value *> NewValues) {
  DISubprogram *OldSP = OldFunc.getSubprogram();
  LLVMContext &Ctx = OldFunc.getContext();

```
- EN: Core entities appearing here include moveCodeToFunction, eraseDebugIntrinsicsWithNonLocalRefs, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as BlockFrequencyInfo, BranchProbabilityInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 moveCodeToFunction, eraseDebugIntrinsicsWithNonLocalRefs，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 BlockFrequencyInfo, BranchProbabilityInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1281-1420

```cpp
  if (!OldSP) {
    // Erase any debug info the new function contains.
    stripDebugInfo(NewFunc);
    // Make sure the old function doesn't contain any non-local metadata refs.
    eraseDebugIntrinsicsWithNonLocalRefs(NewFunc);
    return;
  }

  // Create a subprogram for the new function. Leave out a description of the
  // function arguments, as the parameters don't correspond to anything at the
  // source level.
  assert(OldSP->getUnit() && "Missing compile unit for subprogram");
  DIBuilder DIB(*OldFunc.getParent(), /*AllowUnresolved=*/false,
                OldSP->getUnit());
  auto SPType = DIB.createSubroutineType(DIB.getOrCreateTypeArray({}));
  DISubprogram::DISPFlags SPFlags = DISubprogram::SPFlagDefinition |
                                    DISubprogram::SPFlagOptimized |
                                    DISubprogram::SPFlagLocalToUnit;
  auto NewSP = DIB.createFunction(
      OldSP->getUnit(), NewFunc.getName(), NewFunc.getName(), OldSP->getFile(),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // inlined, point the intrinsic to a fresh variable within the new
      // function.
      if (!DVR.getDebugLoc().getInlinedAt())
        DVR.setVariable(GetUpdatedDIVariable(DVR.getVariable()));
    }
  };

  for (Instruction &I : instructions(NewFunc))
    UpdateDbgRecordsOnInst(I);

  for (auto *DVR : DVRsToDelete)
    DVR->getMarker()->MarkedInstr->dropOneDbgRecord(DVR);
  DIB.finalizeSubprogram(NewSP);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1421-1557

```cpp
  // Fix up the scope information attached to the line locations and the
  // debug assignment metadata in the new function.
  DenseMap<DIAssignID *, DIAssignID *> AssignmentIDMap;
  for (Instruction &I : instructions(NewFunc)) {
    if (const DebugLoc &DL = I.getDebugLoc())
      I.setDebugLoc(
          DebugLoc::replaceInlinedAtSubprogram(DL, *NewSP, Ctx, Cache));
    for (DbgRecord &DR : I.getDbgRecordRange())
      DR.setDebugLoc(DebugLoc::replaceInlinedAtSubprogram(DR.getDebugLoc(),
                                                          *NewSP, Ctx, Cache));

    // Loop info metadata may contain line locations. Fix them up.
    auto updateLoopInfoLoc = [&Ctx, &Cache, NewSP](Metadata *MD) -> Metadata * {
      if (auto *Loc = dyn_cast_or_null<DILocation>(MD))
        return DebugLoc::replaceInlinedAtSubprogram(Loc, *NewSP, Ctx, Cache);
      return MD;
    };
    updateLoopMetadataDebugLocations(I, updateLoopInfoLoc);
    at::remapAssignID(AssignmentIDMap, I);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      StructValues, StructTy);
  SmallVector<Value *> NewValues;

  emitFunctionBody(inputs, outputs, StructValues, newFunction, StructTy, header,
                   SinkingCands, NewValues);

  std::vector<Value *> Reloads;
  CallInst *TheCall = emitReplacerCall(
      inputs, outputs, StructValues, newFunction, StructTy, oldFunction, ReplIP,
      EntryFreq, LifetimesStart.getArrayRef(), Reloads);

  insertReplacerCall(oldFunction, header, TheCall, outputs, Reloads,
                     ExitWeights);

```
- EN: Core entities appearing here include extractCodeRegion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 extractCodeRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1558-1701

```cpp
  fixupDebugInfoPostExtraction(*oldFunction, *newFunction, *TheCall, inputs,
                               NewValues);

  LLVM_DEBUG(llvm::dbgs() << "After extractCodeRegion - newFunction:\n");
  LLVM_DEBUG(newFunction->dump());
  LLVM_DEBUG(llvm::dbgs() << "After extractCodeRegion - oldFunction:\n");
  LLVM_DEBUG(oldFunction->dump());
  LLVM_DEBUG(if (AC && verifyAssumptionCache(*oldFunction, *newFunction, AC))
                 report_fatal_error("Stale Asumption cache for old Function!"));
  return newFunction;
}

void CodeExtractor::normalizeCFGForExtraction(BasicBlock *&header) {
  // If we have any return instructions in the region, split those blocks so
  // that the return is not in the region.
  splitReturnBlocks();

  // If we have to split PHI nodes of the entry or exit blocks, do so now.
  severSplitPHINodesOfEntry(header);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          AlignmentValue = inputs[i]->getPointerAlignment(DL).value();
        MDBuilder MDB(header->getContext());
        LoadGEP->setMetadata(
            LLVMContext::MD_align,
            MDNode::get(
                header->getContext(),
                MDB.createConstant(ConstantInt::get(
                    Type::getInt64Ty(header->getContext()), AlignmentValue))));
      }
      RewriteVal = LoadGEP;
      ++aggIdx;
    } else
      RewriteVal = &*ScalarAI++;

```
- EN: Core entities appearing here include normalizeCFGForExtraction, computeExtractedFuncRetVals, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 normalizeCFGForExtraction, computeExtractedFuncRetVals，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1702-1841

```cpp
    NewValues.push_back(RewriteVal);
  }

  moveCodeToFunction(newFunction);

  for (unsigned i = 0, e = inputs.size(); i != e; ++i) {
    Value *RewriteVal = NewValues[i];

    std::vector<User *> Users(inputs[i]->user_begin(), inputs[i]->user_end());
    for (User *use : Users)
      if (Instruction *inst = dyn_cast<Instruction>(use))
        if (Blocks.count(inst->getParent()))
          inst->replaceUsesOfWith(inputs[i], RewriteVal);
  }

  // Since there may be multiple exits from the original region, make the new
  // function return an unsigned, switch on that number.  This loop iterates
  // over all of the blocks in the extracted region, updating any terminator
  // instructions in the to-be-extracted region that branch to blocks that are
  // not in the region to be extracted.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      Idx[1] = ConstantInt::get(Type::getInt32Ty(Context), AggIdx);
      GetElementPtrInst *GEP = GetElementPtrInst::Create(
          StructArgTy, AggArg, Idx, "gep_" + Output->getName(), InsertPt);
      new StoreInst(Output, GEP, InsertPt);
      ++AggIdx;
    } else {
      assert(ScalarAI != newFunction->arg_end() &&
             "Number of scalar output arguments should match "
             "the number of defined values");
      new StoreInst(Output, &*ScalarAI, InsertPt);
      ++ScalarAI;
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1842-1985

```cpp
  if (ExtractedFuncRetVals.empty()) {
    // Mark the new function `noreturn` if applicable. Terminators which resume
    // exception propagation are treated as returning instructions. This is to
    // avoid inserting traps after calls to outlined functions which unwind.
    if (none_of(Blocks, [](const BasicBlock *BB) {
          const Instruction *Term = BB->getTerminator();
          return isa<ReturnInst>(Term) || isa<ResumeInst>(Term);
        }))
      newFunction->setDoesNotReturn();
  }
}

CallInst *CodeExtractor::emitReplacerCall(
    const ValueSet &inputs, const ValueSet &outputs,
    const ValueSet &StructValues, Function *newFunction,
    StructType *StructArgTy, Function *oldFunction, BasicBlock *ReplIP,
    BlockFrequency EntryFreq, ArrayRef<Value *> LifetimesStart,
    std::vector<Value *> &Reloads) {
  LLVMContext &Context = oldFunction->getContext();
  Module *M = oldFunction->getParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    LoadInst *load =
        new LoadInst(outputs[i]->getType(), Output,
                     outputs[i]->getName() + ".reload", codeReplacer);
    Reloads.push_back(load);
  }

  // Now we can emit a switch statement using the call as a value.
  SwitchInst *TheSwitch =
      SwitchInst::Create(Constant::getNullValue(Type::getInt16Ty(Context)),
                         codeReplacer, 0, codeReplacer);
  for (auto P : enumerate(ExtractedFuncRetVals)) {
    BasicBlock *OldTarget = P.value();
    size_t SuccNum = P.index();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1986-2126

```cpp
    TheSwitch->addCase(ConstantInt::get(Type::getInt16Ty(Context), SuccNum),
                       OldTarget);
  }

  // Now that we've done the deed, simplify the switch instruction.
  Type *OldFnRetTy = TheSwitch->getParent()->getParent()->getReturnType();
  switch (ExtractedFuncRetVals.size()) {
  case 0:
    // There are no successors (the block containing the switch itself), which
    // means that previously this was the last part of the function, and hence
    // this should be rewritten as a `ret` or `unreachable`.
    if (newFunction->doesNotReturn()) {
      // If fn is no return, end with an unreachable terminator.
      (void)new UnreachableInst(Context, TheSwitch->getIterator());
    } else if (OldFnRetTy->isVoidTy()) {
      // We have no return value.
      ReturnInst::Create(Context, nullptr,
                         TheSwitch->getIterator()); // Return void
    } else if (OldFnRetTy == TheSwitch->getCondition()->getType()) {
      // return what we have
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                 "PHI has two incompatbile incoming values from codeRepl");
      }
    }

  for (unsigned i = 0, e = outputs.size(); i != e; ++i) {
    Value *load = Reloads[i];
    std::vector<User *> Users(outputs[i]->user_begin(), outputs[i]->user_end());
    for (User *U : Users) {
      Instruction *inst = cast<Instruction>(U);
      if (inst->getParent()->getParent() == oldFunction)
        inst->replaceUsesOfWith(outputs[i], load);
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2127-2168

```cpp
  if (FuncRetVal)
    FuncRetVal->replaceUsesWithIf(ReplacerCall, [&](Use &U) {
      return cast<Instruction>(U.getUser())->getFunction() == oldFunction;
    });

  // Update the branch weights for the exit block.
  if (BFI && ExtractedFuncRetVals.size() > 1)
    calculateNewCallTerminatorWeights(codeReplacer, ExitWeights, BPI);
}

bool CodeExtractor::verifyAssumptionCache(const Function &OldFunc,
                                          const Function &NewFunc,
                                          AssumptionCache *AC) {
  for (auto AssumeVH : AC->assumptions()) {
    auto *I = dyn_cast_or_null<CallInst>(AssumeVH);
    if (!I)
      continue;

    // There shouldn't be any llvm.assume intrinsics in the new function.
    if (I->getFunction() != &OldFunc)
      return true;

    // There shouldn't be any stale affected values in the assumption cache
    // that were previously in the old function, but that have now been moved
    // to the new function.
    for (auto AffectedValVH : AC->assumptionsFor(I->getOperand(0))) {
      auto *AffectedCI = dyn_cast_or_null<CallInst>(AffectedValVH);
      if (!AffectedCI)
        continue;
      if (AffectedCI->getFunction() != &OldFunc)
        return true;
      auto *AssumedInst = cast<Instruction>(AffectedCI->getOperand(0));
      if (AssumedInst->getFunction() != &OldFunc)
        return true;
    }
  }
  return false;
}

void CodeExtractor::excludeArgFromAggregate(Value *Arg) {
  ExcludeArgsFromAggregate.insert(Arg);
}
```
- EN: Core entities appearing here include excludeArgFromAggregate, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache.
- CN: 此处出现的核心实体包括 excludeArgFromAggregate，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `isAlignmentPreservedForAddrCast, definedInRegion, definedInCaller, CodeExtractorAnalysisCache, findSideEffectInfoForBlock, findOrCreateBlockForHoisting, make_early_inc_range, isEligible` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`isAlignmentPreservedForAddrCast, definedInRegion, definedInCaller, CodeExtractorAnalysisCache, findSideEffectInfoForBlock, findOrCreateBlockForHoisting, make_early_inc_range, isEligible` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, BlockFrequencyInfo, BranchProbabilityInfo, DataLayout, DominatorTree, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, BlockFrequencyInfo, BranchProbabilityInfo, DataLayout, DominatorTree, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `AggregateArgsOpt` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `AggregateArgsOpt` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/BlockFrequencyInfoImpl.h`, `llvm/Analysis/BranchProbabilityInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/BlockFrequencyInfoImpl.h`, `llvm/Analysis/BranchProbabilityInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/Attributes.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DIBuilder.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/Attributes.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DIBuilder.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/BlockFrequency.h`, `llvm/Support/BranchProbability.h`, `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/BlockFrequency.h`, `llvm/Support/BranchProbability.h`, `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstdint`, `iterator`, `map`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstdint`, `iterator`, `map`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `BlockFrequencyInfo`, `BranchProbabilityInfo`, `DataLayout`, `DominatorTree`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `BlockFrequencyInfo`, `BranchProbabilityInfo`, `DataLayout`, `DominatorTree`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
