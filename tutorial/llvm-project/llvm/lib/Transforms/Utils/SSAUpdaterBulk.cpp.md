# SSAUpdaterBulk.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SSAUpdaterBulk.cpp` | `llvm/lib/Transforms/Utils/SSAUpdaterBulk.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements unstructured SSA Update Tool within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SSAUpdaterBulk 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-37

```cpp
//===- SSAUpdaterBulk.cpp - Unstructured SSA Update Tool ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the SSAUpdaterBulk class.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SSAUpdaterBulk.h"
#include "llvm/Analysis/InstructionSimplify.h"
#include "llvm/Analysis/IteratedDominanceFrontier.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Use.h"
#include "llvm/IR/Value.h"

using namespace llvm;

#define DEBUG_TYPE "ssaupdaterbulk"

/// Helper function for finding a block which should have a value for the given
/// user. For PHI-nodes this block is the corresponding predecessor, for other
/// instructions it's their parent block.
static BasicBlock *getUserBB(Use *U) {
  auto *User = cast<Instruction>(U->getUser());

  if (auto *UserPN = dyn_cast<PHINode>(User))
    return UserPN->getIncomingBlock(*U);
  else
    return User->getParent();
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 38-67

```cpp
/// Add a new variable to the SSA rewriter. This needs to be called before
/// AddAvailableValue or AddUse calls.
unsigned SSAUpdaterBulk::AddVariable(StringRef Name, Type *Ty) {
  unsigned Var = Rewrites.size();
  LLVM_DEBUG(dbgs() << "SSAUpdater: Var=" << Var << ": initialized with Ty = "
                    << *Ty << ", Name = " << Name << "\n");
  RewriteInfo RI(Name, Ty);
  Rewrites.push_back(RI);
  return Var;
}

/// Indicate that a rewritten value is available in the specified block with the
/// specified value.
void SSAUpdaterBulk::AddAvailableValue(unsigned Var, BasicBlock *BB, Value *V) {
  assert(Var < Rewrites.size() && "Variable not found!");
  LLVM_DEBUG(dbgs() << "SSAUpdater: Var=" << Var
                    << ": added new available value " << *V << " in "
                    << BB->getName() << "\n");
  Rewrites[Var].Defines.emplace_back(BB, V);
}

/// Record a use of the symbolic value. This use will be updated with a
/// rewritten value when RewriteAllUses is called.
void SSAUpdaterBulk::AddUse(unsigned Var, Use *U) {
  assert(Var < Rewrites.size() && "Variable not found!");
  LLVM_DEBUG(dbgs() << "SSAUpdater: Var=" << Var << ": added a use" << *U->get()
                    << " in " << getUserBB(U)->getName() << "\n");
  Rewrites[Var].Uses.push_back(U);
}

```
- EN: Core entities appearing here include AddVariable, AddAvailableValue, AddUse, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 AddVariable, AddAvailableValue, AddUse，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 68-104

```cpp
/// Given sets of UsingBlocks and DefBlocks, compute the set of LiveInBlocks.
/// This is basically a subgraph limited by DefBlocks and UsingBlocks.
static void
ComputeLiveInBlocks(const SmallPtrSetImpl<BasicBlock *> &UsingBlocks,
                    const SmallPtrSetImpl<BasicBlock *> &DefBlocks,
                    SmallPtrSetImpl<BasicBlock *> &LiveInBlocks,
                    PredIteratorCache &PredCache) {
  // To determine liveness, we must iterate through the predecessors of blocks
  // where the def is live.  Blocks are added to the worklist if we need to
  // check their predecessors.  Start with all the using blocks.
  SmallVector<BasicBlock *, 64> LiveInBlockWorklist(UsingBlocks.begin(),
                                                    UsingBlocks.end());

  // Now that we have a set of blocks where the phi is live-in, recursively add
  // their predecessors until we find the full region the value is live.
  while (!LiveInBlockWorklist.empty()) {
    BasicBlock *BB = LiveInBlockWorklist.pop_back_val();

    // The block really is live in here, insert it into the set.  If already in
    // the set, then it has already been processed.
    if (!LiveInBlocks.insert(BB).second)
      continue;

    // Since the value is live into BB, it is either defined in a predecessor or
    // live into it to.  Add the preds to the worklist unless they are a
    // defining block.
    for (BasicBlock *P : PredCache.get(BB)) {
      // The value is not live into a predecessor if it defines the value.
      if (DefBlocks.count(P))
        continue;

      // Otherwise it is, add to the worklist.
      LiveInBlockWorklist.push_back(P);
    }
  }
}

```
- EN: This region continues the SSAUpdaterBulk implementation with local helper logic centered on Given, UsingBlocks, DefBlocks, LiveInBlocks.
- CN: 这一段延续了 SSAUpdaterBulk 的主体实现，围绕 Given, UsingBlocks, DefBlocks, LiveInBlocks 等局部辅助逻辑展开。

### Lines 105-140

```cpp
struct BBValueInfo {
  Value *LiveInValue = nullptr;
  Value *LiveOutValue = nullptr;
};

/// Perform all the necessary updates, including new PHI-nodes insertion and the
/// requested uses update.
void SSAUpdaterBulk::RewriteAllUses(DominatorTree *DT,
                                    SmallVectorImpl<PHINode *> *InsertedPHIs) {
  DenseMap<BasicBlock *, BBValueInfo> BBInfos;
  for (RewriteInfo &R : Rewrites) {
    BBInfos.clear();

    // Compute locations for new phi-nodes.
    // For that we need to initialize DefBlocks from definitions in R.Defines,
    // UsingBlocks from uses in R.Uses, then compute LiveInBlocks, and then use
    // this set for computing iterated dominance frontier (IDF).
    // The IDF blocks are the blocks where we need to insert new phi-nodes.
    ForwardIDFCalculator IDF(*DT);
    LLVM_DEBUG(dbgs() << "SSAUpdater: rewriting " << R.Uses.size()
                      << " use(s)\n");

    SmallPtrSet<BasicBlock *, 2> DefBlocks(llvm::from_range,
                                           llvm::make_first_range(R.Defines));
    IDF.setDefiningBlocks(DefBlocks);

    SmallPtrSet<BasicBlock *, 2> UsingBlocks;
    for (Use *U : R.Uses)
      UsingBlocks.insert(getUserBB(U));

    SmallVector<BasicBlock *, 32> IDFBlocks;
    SmallPtrSet<BasicBlock *, 32> LiveInBlocks;
    ComputeLiveInBlocks(UsingBlocks, DefBlocks, LiveInBlocks, PredCache);
    IDF.setLiveInBlocks(LiveInBlocks);
    IDF.calculate(IDFBlocks);

```
- EN: Core entities appearing here include BBValueInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 BBValueInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 141-178

```cpp
    // Reserve sufficient buckets to prevent map growth. [1]
    BBInfos.reserve(LiveInBlocks.size() + DefBlocks.size());

    for (auto [BB, V] : R.Defines)
      BBInfos[BB].LiveOutValue = V;

    // We've computed IDF, now insert new phi-nodes there.
    for (BasicBlock *FrontierBB : IDFBlocks) {
      IRBuilder<> B(FrontierBB, FrontierBB->begin());
      PHINode *PN = B.CreatePHI(R.Ty, 0, R.Name);
      BBInfos[FrontierBB].LiveInValue = PN;
      if (InsertedPHIs)
        InsertedPHIs->push_back(PN);
    }

    // IsLiveOut indicates whether we are computing live-out values (true) or
    // live-in values (false).
    auto ComputeValue = [&](BasicBlock *BB, bool IsLiveOut) -> Value * {
      BBValueInfo *BBInfo = &BBInfos[BB];

      if (IsLiveOut && BBInfo->LiveOutValue)
        return BBInfo->LiveOutValue;

      if (BBInfo->LiveInValue)
        return BBInfo->LiveInValue;

      SmallVector<BBValueInfo *, 4> Stack = {BBInfo};
      Value *V = nullptr;

      while (DT->isReachableFromEntry(BB) && !PredCache.get(BB).empty() &&
             (BB = DT->getNode(BB)->getIDom()->getBlock())) {
        BBInfo = &BBInfos[BB];

        if (BBInfo->LiveOutValue) {
          V = BBInfo->LiveOutValue;
          break;
        }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 179-210

```cpp
        if (BBInfo->LiveInValue) {
          V = BBInfo->LiveInValue;
          break;
        }

        Stack.emplace_back(BBInfo);
      }

      if (!V)
        V = UndefValue::get(R.Ty);

      for (BBValueInfo *BBInfo : Stack)
        // Loop above can insert new entries into the BBInfos map: assume the
        // map shouldn't grow due to [1] and BBInfo references are valid.
        BBInfo->LiveInValue = V;

      return V;
    };

    // Fill in arguments of the inserted PHIs.
    for (BasicBlock *BB : IDFBlocks) {
      auto *PHI = cast<PHINode>(&BB->front());
      for (BasicBlock *Pred : PredCache.get(BB))
        PHI->addIncoming(ComputeValue(Pred, /*IsLiveOut=*/true), Pred);
    }

    // Rewrite actual uses with the inserted definitions.
    SmallPtrSet<Use *, 4> ProcessedUses;
    for (Use *U : R.Uses) {
      if (!ProcessedUses.insert(U).second)
        continue;

```
- EN: This region continues the SSAUpdaterBulk implementation with local helper logic centered on BBInfo, LiveInValue, Stack, UndefValue.
- CN: 这一段延续了 SSAUpdaterBulk 的主体实现，围绕 BBInfo, LiveInValue, Stack, UndefValue 等局部辅助逻辑展开。

### Lines 211-241

```cpp
      auto *User = cast<Instruction>(U->getUser());
      BasicBlock *BB = getUserBB(U);
      Value *V = ComputeValue(BB, /*IsLiveOut=*/BB != User->getParent());
      Value *OldVal = U->get();
      assert(OldVal && "Invalid use!");
      // Notify that users of the existing value that it is being replaced.
      if (OldVal != V && OldVal->hasValueHandle())
        ValueHandleBase::ValueIsRAUWd(OldVal, V);
      LLVM_DEBUG(dbgs() << "SSAUpdater: replacing " << *OldVal << " with " << *V
                        << "\n");
      U->set(V);
    }
  }
}

// Perform a single pass of simplification over the worklist of PHIs.
// This should be called after RewriteAllUses() because simplifying PHIs
// immediately after creation would require updating all references to those
// PHIs in the BBValueInfo structures, which would necessitate additional
// reference tracking overhead.
static void simplifyPass(MutableArrayRef<PHINode *> Worklist,
                         const DataLayout &DL) {
  for (PHINode *&PHI : Worklist) {
    if (Value *Simplified = simplifyInstruction(PHI, DL)) {
      PHI->replaceAllUsesWith(Simplified);
      PHI->eraseFromParent();
      PHI = nullptr; // Mark as removed.
    }
  }
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 242-275

```cpp
#ifndef NDEBUG // Should this be under EXPENSIVE_CHECKS?
// New PHI nodes should not reference one another but they may reference
// themselves or existing PHI nodes, and existing PHI nodes may reference new
// PHI nodes.
static bool
PHIAreRefEachOther(const iterator_range<BasicBlock::phi_iterator> NewPHIs) {
  SmallPtrSet<PHINode *, 8> NewPHISet;
  for (PHINode &PN : NewPHIs)
    NewPHISet.insert(&PN);
  for (PHINode &PHI : NewPHIs) {
    for (Value *V : PHI.incoming_values()) {
      PHINode *IncPHI = dyn_cast<PHINode>(V);
      if (IncPHI && IncPHI != &PHI && NewPHISet.contains(IncPHI))
        return true;
    }
  }
  return false;
}
#endif

static bool replaceIfIdentical(PHINode &PHI, PHINode &ReplPHI) {
  if (!PHI.isIdenticalToWhenDefined(&ReplPHI))
    return false;
  PHI.replaceAllUsesWith(&ReplPHI);
  PHI.eraseFromParent();
  return true;
}

namespace llvm {

bool EliminateNewDuplicatePHINodes(BasicBlock *BB,
                                   BasicBlock::phi_iterator FirstExistingPN) {
  assert(!PHIAreRefEachOther(make_range(BB->phis().begin(), FirstExistingPN)));

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PHIAreRefEachOther, replaceIfIdentical, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PHIAreRefEachOther, replaceIfIdentical，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 276-312

```cpp
  // Deduplicate new PHIs first to reduce the number of comparisons on the
  // following new -> existing pass.
  bool Changed = false;
  for (auto I = BB->phis().begin(); I != FirstExistingPN; ++I) {
    for (auto J = std::next(I); J != FirstExistingPN;) {
      Changed |= replaceIfIdentical(*J++, *I);
    }
  }

  // Iterate over existing PHIs and replace identical new PHIs.
  for (PHINode &ExistingPHI : make_range(FirstExistingPN, BB->phis().end())) {
    auto I = BB->phis().begin();
    assert(I != FirstExistingPN); // Should be at least one new PHI.
    do {
      Changed |= replaceIfIdentical(*I++, ExistingPHI);
    } while (I != FirstExistingPN);
    if (BB->phis().begin() == FirstExistingPN)
      return Changed;
  }
  return Changed;
}

} // end namespace llvm

static void deduplicatePass(ArrayRef<PHINode *> Worklist) {
  SmallDenseMap<BasicBlock *, unsigned> BBs;
  for (PHINode *PHI : Worklist) {
    if (PHI)
      ++BBs[PHI->getParent()];
  }

  for (auto [BB, NumNewPHIs] : BBs) {
    auto FirstExistingPN = std::next(BB->phis().begin(), NumNewPHIs);
    EliminateNewDuplicatePHINodes(BB, FirstExistingPN);
  }
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include deduplicatePass, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 deduplicatePass，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 313-321

```cpp
void SSAUpdaterBulk::RewriteAndOptimizeAllUses(DominatorTree &DT) {
  SmallVector<PHINode *, 4> PHIs;
  RewriteAllUses(&DT, &PHIs);
  if (PHIs.empty())
    return;

  simplifyPass(PHIs, PHIs.front()->getParent()->getDataLayout());
  deduplicatePass(PHIs);
}
```
- EN: Core entities appearing here include RewriteAndOptimizeAllUses, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 RewriteAndOptimizeAllUses，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `BBValueInfo, AddVariable, AddAvailableValue, AddUse, PHIAreRefEachOther, replaceIfIdentical, deduplicatePass, RewriteAndOptimizeAllUses` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`BBValueInfo, AddVariable, AddAvailableValue, AddUse, PHIAreRefEachOther, replaceIfIdentical, deduplicatePass, RewriteAndOptimizeAllUses` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/IteratedDominanceFrontier.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/IteratedDominanceFrontier.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Use.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/SSAUpdaterBulk.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Use.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/SSAUpdaterBulk.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
