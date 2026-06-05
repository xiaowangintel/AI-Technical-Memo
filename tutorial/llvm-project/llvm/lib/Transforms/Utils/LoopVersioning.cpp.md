# LoopVersioning.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopVersioning.cpp` | `llvm/lib/Transforms/Utils/LoopVersioning.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements utility to version a loop within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopVersioning 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-34

```cpp
//===- LoopVersioning.cpp - Utility to version a loop ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines a utility class to perform loop versioning.  The versioned
// loop speculates that otherwise may-aliasing memory accesses don't overlap and
// emits checks to prove this.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LoopVersioning.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/InstSimplifyFolder.h"
#include "llvm/Analysis/LoopAccessAnalysis.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/PassManager.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Cloning.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"

using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, LoopInfo, ScalarEvolution, TargetLibraryInfo.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, LoopInfo, ScalarEvolution, TargetLibraryInfo 等分析结果。

### Lines 35-71

```cpp
#define DEBUG_TYPE "loop-versioning"

static cl::opt<bool>
    AnnotateNoAlias("loop-version-annotate-no-alias", cl::init(true),
                    cl::Hidden,
                    cl::desc("Add no-alias annotation for instructions that "
                             "are disambiguated by memchecks"));

LoopVersioning::LoopVersioning(const LoopAccessInfo &LAI,
                               ArrayRef<RuntimePointerCheck> Checks, Loop *L,
                               LoopInfo *LI, DominatorTree *DT,
                               ScalarEvolution *SE)
    : VersionedLoop(L), AliasChecks(Checks), Preds(LAI.getPSE().getPredicate()),
      LAI(LAI), LI(LI), DT(DT), SE(SE) {}

void LoopVersioning::versionLoop(
    const SmallVectorImpl<Instruction *> &DefsUsedOutside) {
  assert(VersionedLoop->getUniqueExitBlock() && "No single exit block");
  assert(VersionedLoop->isLoopSimplifyForm() &&
         "Loop is not in loop-simplify form");

  Value *MemRuntimeCheck;
  Value *SCEVRuntimeCheck;
  Value *RuntimeCheck = nullptr;

  // Add the memcheck in the original preheader (this is empty initially).
  BasicBlock *RuntimeCheckBB = VersionedLoop->getLoopPreheader();
  const auto &RtPtrChecking = *LAI.getRuntimePointerChecking();

  SCEVExpander Exp2(*RtPtrChecking.getSE(), "induction");
  MemRuntimeCheck = addRuntimeChecks(RuntimeCheckBB->getTerminator(),
                                     VersionedLoop, AliasChecks, Exp2);

  SCEVExpander Exp(*SE, "scev.check");
  SCEVRuntimeCheck =
      Exp.expandCodeForPredicate(&Preds, RuntimeCheckBB->getTerminator());

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopAccessInfo, LoopInfo, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopAccessInfo, LoopInfo, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 72-106

```cpp
  IRBuilder<InstSimplifyFolder> Builder(
      RuntimeCheckBB->getContext(),
      InstSimplifyFolder(RuntimeCheckBB->getDataLayout()));
  if (MemRuntimeCheck && SCEVRuntimeCheck) {
    Builder.SetInsertPoint(RuntimeCheckBB->getTerminator());
    RuntimeCheck =
        Builder.CreateOr(MemRuntimeCheck, SCEVRuntimeCheck, "lver.safe");
  } else
    RuntimeCheck = MemRuntimeCheck ? MemRuntimeCheck : SCEVRuntimeCheck;

  Exp.eraseDeadInstructions(SCEVRuntimeCheck);

  assert(RuntimeCheck && "called even though we don't need "
                         "any runtime checks");

  // Rename the block to make the IR more readable.
  RuntimeCheckBB->setName(VersionedLoop->getHeader()->getName() +
                          ".lver.check");

  // Create empty preheader for the loop (and after cloning for the
  // non-versioned loop).
  BasicBlock *PH =
      SplitBlock(RuntimeCheckBB, RuntimeCheckBB->getTerminator(), DT, LI,
                 nullptr, VersionedLoop->getHeader()->getName() + ".ph");

  // Clone the loop including the preheader.
  //
  // FIXME: This does not currently preserve SimplifyLoop because the exit
  // block is a join between the two loops.
  SmallVector<BasicBlock *, 8> NonVersionedLoopBlocks;
  NonVersionedLoop =
      cloneLoopWithPreheader(PH, RuntimeCheckBB, VersionedLoop, VMap,
                             ".lver.orig", LI, DT, NonVersionedLoopBlocks);
  remapInstructionsInBlocks(NonVersionedLoopBlocks, VMap);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 107-137

```cpp
  // Insert the conditional branch based on the result of the memchecks.
  Instruction *OrigTerm = RuntimeCheckBB->getTerminator();
  Builder.SetInsertPoint(OrigTerm);
  auto *BI =
      Builder.CreateCondBr(RuntimeCheck, NonVersionedLoop->getLoopPreheader(),
                           VersionedLoop->getLoopPreheader());
  // We don't know what the probability of executing the versioned vs the
  // unversioned variants is.
  setExplicitlyUnknownBranchWeightsIfProfiled(*BI, DEBUG_TYPE);
  OrigTerm->eraseFromParent();

  // The loops merge in the original exit block.  This is now dominated by the
  // memchecking block.
  DT->changeImmediateDominator(VersionedLoop->getExitBlock(), RuntimeCheckBB);

  // Adds the necessary PHI nodes for the versioned loops based on the
  // loop-defined values used outside of the loop.
  addPHINodes(DefsUsedOutside);
  formDedicatedExitBlocks(NonVersionedLoop, DT, LI, nullptr, true);
  formDedicatedExitBlocks(VersionedLoop, DT, LI, nullptr, true);
  assert(NonVersionedLoop->isLoopSimplifyForm() &&
         VersionedLoop->isLoopSimplifyForm() &&
         "The versioned loops should be in simplify form.");
}

void LoopVersioning::addPHINodes(
    const SmallVectorImpl<Instruction *> &DefsUsedOutside) {
  BasicBlock *PHIBlock = VersionedLoop->getExitBlock();
  assert(PHIBlock && "No single successor to loop exit block");
  PHINode *PN;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 138-173

```cpp
  // First add a single-operand PHI for each DefsUsedOutside if one does not
  // exists yet.
  for (auto *Inst : DefsUsedOutside) {
    // See if we have a single-operand PHI with the value defined by the
    // original loop.
    for (auto I = PHIBlock->begin(); (PN = dyn_cast<PHINode>(I)); ++I) {
      if (PN->getIncomingValue(0) == Inst) {
        SE->forgetLcssaPhiWithNewPredecessor(VersionedLoop, PN);
        break;
      }
    }
    // If not create it.
    if (!PN) {
      PN = PHINode::Create(Inst->getType(), 2, Inst->getName() + ".lver");
      PN->insertBefore(PHIBlock->begin());
      SmallVector<User*, 8> UsersToUpdate;
      for (User *U : Inst->users())
        if (!VersionedLoop->contains(cast<Instruction>(U)->getParent()))
          UsersToUpdate.push_back(U);
      for (User *U : UsersToUpdate)
        U->replaceUsesOfWith(Inst, PN);
      PN->addIncoming(Inst, VersionedLoop->getExitingBlock());
    }
  }

  // Then for each PHI add the operand for the edge from the cloned loop.
  for (auto I = PHIBlock->begin(); (PN = dyn_cast<PHINode>(I)); ++I) {
    assert(PN->getNumOperands() == 1 &&
           "Exit block should only have on predecessor");

    // If the definition was cloned used that otherwise use the same value.
    Value *ClonedValue = PN->getIncomingValue(0);
    auto Mapped = VMap.find(ClonedValue);
    if (Mapped != VMap.end())
      ClonedValue = Mapped->second;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 174-208

```cpp
    PN->addIncoming(ClonedValue, NonVersionedLoop->getExitingBlock());
  }
}

void LoopVersioning::prepareNoAliasMetadata() {
  // We need to turn the no-alias relation between pointer checking groups into
  // no-aliasing annotations between instructions.
  //
  // We accomplish this by mapping each pointer checking group (a set of
  // pointers memchecked together) to an alias scope and then also mapping each
  // group to the list of scopes it can't alias.

  const RuntimePointerChecking *RtPtrChecking = LAI.getRuntimePointerChecking();
  LLVMContext &Context = VersionedLoop->getHeader()->getContext();

  // First allocate an aliasing scope for each pointer checking group.
  //
  // While traversing through the checking groups in the loop, also create a
  // reverse map from pointers to the pointer checking group they were assigned
  // to.
  MDBuilder MDB(Context);
  MDNode *Domain = MDB.createAnonymousAliasScopeDomain("LVerDomain");

  for (const auto &Group : RtPtrChecking->CheckingGroups) {
    GroupToScope[&Group] = MDB.createAnonymousAliasScope(Domain);

    for (unsigned PtrIdx : Group.Members)
      PtrToGroup[RtPtrChecking->getPointerInfo(PtrIdx).PointerValue] = &Group;
  }

  // Go through the checks and for each pointer group, collect the scopes for
  // each non-aliasing pointer group.
  DenseMap<const RuntimeCheckingPtrGroup *, SmallVector<Metadata *, 4>>
      GroupToNonAliasingScopes;

```
- EN: Core entities appearing here include prepareNoAliasMetadata, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 prepareNoAliasMetadata，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 209-241

```cpp
  for (const auto &Check : AliasChecks)
    GroupToNonAliasingScopes[Check.first].push_back(GroupToScope[Check.second]);

  // Finally, transform the above to actually map to scope list which is what
  // the metadata uses.

  for (const auto &Pair : GroupToNonAliasingScopes)
    GroupToNonAliasingScopeList[Pair.first] = MDNode::get(Context, Pair.second);
}

void LoopVersioning::annotateLoopWithNoAlias() {
  if (!AnnotateNoAlias)
    return;

  // First prepare the maps.
  prepareNoAliasMetadata();

  // Add the scope and no-alias metadata to the instructions.
  for (Instruction *I : LAI.getDepChecker().getMemoryInstructions()) {
    annotateInstWithNoAlias(I);
  }
}

std::pair<MDNode *, MDNode *>
LoopVersioning::getNoAliasMetadataFor(const Instruction *OrigInst) const {
  if (!AnnotateNoAlias)
    return {nullptr, nullptr};

  LLVMContext &Context = VersionedLoop->getHeader()->getContext();
  const Value *Ptr = isa<LoadInst>(OrigInst)
                         ? cast<LoadInst>(OrigInst)->getPointerOperand()
                         : cast<StoreInst>(OrigInst)->getPointerOperand();

```
- EN: Core entities appearing here include annotateLoopWithNoAlias, getNoAliasMetadataFor, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 annotateLoopWithNoAlias, getNoAliasMetadataFor，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 242-278

```cpp
  MDNode *AliasScope = nullptr;
  MDNode *NoAlias = nullptr;
  // Find the group for the pointer and then add the scope metadata.
  auto Group = PtrToGroup.find(Ptr);
  if (Group != PtrToGroup.end()) {
    AliasScope = MDNode::concatenate(
        OrigInst->getMetadata(LLVMContext::MD_alias_scope),
        MDNode::get(Context, GroupToScope.lookup(Group->second)));

    // Add the no-alias metadata.
    auto NonAliasingScopeList = GroupToNonAliasingScopeList.find(Group->second);
    if (NonAliasingScopeList != GroupToNonAliasingScopeList.end())
      NoAlias =
          MDNode::concatenate(OrigInst->getMetadata(LLVMContext::MD_noalias),
                              NonAliasingScopeList->second);
  }
  return {AliasScope, NoAlias};
}

void LoopVersioning::annotateInstWithNoAlias(Instruction *VersionedInst,
                                             const Instruction *OrigInst) {
  const auto &[AliasScopeMD, NoAliasMD] = getNoAliasMetadataFor(OrigInst);
  if (AliasScopeMD)
    VersionedInst->setMetadata(LLVMContext::MD_alias_scope, AliasScopeMD);

  if (NoAliasMD)
    VersionedInst->setMetadata(LLVMContext::MD_noalias, NoAliasMD);
}

namespace {
bool runImpl(LoopInfo *LI, LoopAccessInfoManager &LAIs, DominatorTree *DT,
             ScalarEvolution *SE) {
  // Build up a worklist of inner-loops to version. This is necessary as the
  // act of versioning a loop creates new loops and can invalidate iterators
  // across the loops.
  SmallVector<Loop *, 8> Worklist;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopAccessInfo, LoopAccessInfoManager, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopAccessInfo, LoopAccessInfoManager, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 279-310

```cpp
  for (Loop *TopLevelLoop : *LI)
    for (Loop *L : depth_first(TopLevelLoop))
      // We only handle inner-most loops.
      if (L->isInnermost())
        Worklist.push_back(L);

  // Now walk the identified inner loops.
  bool Changed = false;
  for (Loop *L : Worklist) {
    if (!L->isLoopSimplifyForm() || !L->isRotatedForm() ||
        !L->getExitingBlock())
      continue;
    const LoopAccessInfo &LAI = LAIs.getInfo(*L);
    if (!LAI.hasConvergentOp() &&
        (LAI.getNumRuntimePointerChecks() ||
         !LAI.getPSE().getPredicate().isAlwaysTrue())) {
      if (!L->isLCSSAForm(*DT))
       formLCSSARecursively(*L, *DT, LI, SE);

      LoopVersioning LVer(LAI, LAI.getRuntimePointerChecking()->getChecks(), L,
                          LI, DT, SE);
      LVer.versionLoop();
      LVer.annotateLoopWithNoAlias();
      Changed = true;
      LAIs.clear();
    }
  }

  return Changed;
}
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopAccessInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopAccessInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 311-321

```cpp
PreservedAnalyses LoopVersioningPass::run(Function &F,
                                          FunctionAnalysisManager &AM) {
  auto &SE = AM.getResult<ScalarEvolutionAnalysis>(F);
  auto &LI = AM.getResult<LoopAnalysis>(F);
  LoopAccessInfoManager &LAIs = AM.getResult<LoopAccessAnalysis>(F);
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);

  if (runImpl(&LI, LAIs, &DT, &SE))
    return PreservedAnalyses::none();
  return PreservedAnalyses::all();
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopAccessInfo, LoopAccessInfoManager, ScalarEvolution.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopAccessInfo, LoopAccessInfoManager, ScalarEvolution 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `prepareNoAliasMetadata, annotateLoopWithNoAlias, getNoAliasMetadataFor` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`prepareNoAliasMetadata, annotateLoopWithNoAlias, getNoAliasMetadataFor` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AliasAnalysis, DataLayout, DominatorTree, LoopAccessInfo, LoopAccessInfoManager, LoopInfo, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AliasAnalysis, DataLayout, DominatorTree, LoopAccessInfo, LoopAccessInfoManager, LoopInfo, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `AnnotateNoAlias` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `AnnotateNoAlias` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/InstSimplifyFolder.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetLibraryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/InstSimplifyFolder.h`, `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetLibraryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PassManager.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PassManager.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/LoopVersioning.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/ADT/ArrayRef.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/ADT/ArrayRef.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AliasAnalysis`, `DataLayout`, `DominatorTree`, `LoopAccessInfo`, `LoopAccessInfoManager`, `LoopInfo`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AliasAnalysis`, `DataLayout`, `DominatorTree`, `LoopAccessInfo`, `LoopAccessInfoManager`, `LoopInfo`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
