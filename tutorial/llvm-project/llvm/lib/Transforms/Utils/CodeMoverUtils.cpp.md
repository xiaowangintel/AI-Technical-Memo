# CodeMoverUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CodeMoverUtils.cpp` | `llvm/lib/Transforms/Utils/CodeMoverUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements codeMover Utilities within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CodeMoverUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
//===- CodeMoverUtils.cpp - CodeMover Utilities ----------------------------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This family of functions perform movements on basic blocks, and instructions
// contained within a function.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/CodeMoverUtils.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/DependenceAnalysis.h"
#include "llvm/Analysis/PostDominators.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Dominators.h"

using namespace llvm;

#define DEBUG_TYPE "codemover-utils"

STATISTIC(HasDependences,
          "Cannot move across instructions that has memory dependences");
STATISTIC(MayThrowException, "Cannot move across instructions that may throw");
STATISTIC(NotMovedPHINode, "Movement of PHINodes are not supported");
STATISTIC(NotMovedTerminator, "Movement of Terminator are not supported");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 31-57

```cpp
static bool domTreeLevelBefore(DominatorTree *DT, const Instruction *InstA,
                               const Instruction *InstB) {
  // Use ordered basic block in case the 2 instructions are in the same
  // block.
  if (InstA->getParent() == InstB->getParent())
    return InstA->comesBefore(InstB);

  DomTreeNode *DA = DT->getNode(InstA->getParent());
  DomTreeNode *DB = DT->getNode(InstB->getParent());
  return DA->getLevel() < DB->getLevel();
}

static bool reportInvalidCandidate(const Instruction &I,
                                   llvm::Statistic &Stat) {
  ++Stat;
  LLVM_DEBUG(dbgs() << "Unable to move instruction: " << I << ". "
                    << Stat.getDesc());
  return false;
}

/// Collect all instructions in between \p StartInst and \p EndInst, and store
/// them in \p InBetweenInsts.
static void
collectInstructionsInBetween(Instruction &StartInst, const Instruction &EndInst,
                             SmallPtrSetImpl<Instruction *> &InBetweenInsts) {
  assert(InBetweenInsts.empty() && "Expecting InBetweenInsts to be empty");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 58-85

```cpp
  /// Get the next instructions of \p I, and push them to \p WorkList.
  auto getNextInsts = [](Instruction &I,
                         SmallPtrSetImpl<Instruction *> &WorkList) {
    if (Instruction *NextInst = I.getNextNode())
      WorkList.insert(NextInst);
    else {
      assert(I.isTerminator() && "Expecting a terminator instruction");
      for (BasicBlock *Succ : successors(&I))
        WorkList.insert(&Succ->front());
    }
  };

  SmallPtrSet<Instruction *, 10> WorkList;
  getNextInsts(StartInst, WorkList);
  while (!WorkList.empty()) {
    Instruction *CurInst = *WorkList.begin();
    WorkList.erase(CurInst);

    if (CurInst == &EndInst)
      continue;

    if (!InBetweenInsts.insert(CurInst).second)
      continue;

    getNextInsts(*CurInst, WorkList);
  }
}

```
- EN: This region continues the CodeMoverUtils implementation with local helper logic centered on Get, WorkList, Instruction, SmallPtrSetImpl.
- CN: 这一段延续了 CodeMoverUtils 的主体实现，围绕 Get, WorkList, Instruction, SmallPtrSetImpl 等局部辅助逻辑展开。

### Lines 86-112

```cpp
bool llvm::isSafeToMoveBefore(Instruction &I, Instruction &InsertPoint,
                              DominatorTree &DT, const PostDominatorTree *PDT,
                              DependenceInfo *DI, bool CheckForEntireBlock) {
  // Skip tests when we don't have PDT or DI
  if (!PDT || !DI)
    return false;

  // Cannot move itself before itself.
  if (&I == &InsertPoint)
    return false;

  // Not moved.
  if (I.getNextNode() == &InsertPoint)
    return true;

  if (isa<PHINode>(I) || isa<PHINode>(InsertPoint))
    return reportInvalidCandidate(I, NotMovedPHINode);

  if (I.isTerminator())
    return reportInvalidCandidate(I, NotMovedTerminator);

  if (isReachedBefore(&I, &InsertPoint, &DT, PDT))
    for (const Use &U : I.uses())
      if (auto *UserInst = dyn_cast<Instruction>(U.getUser())) {
        // If InsertPoint is in a BB that comes after I, then we cannot move if
        // I is used in the terminator of the current BB.
        if (I.getParent() == InsertPoint.getParent() &&
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, PostDominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, PostDominatorTree 等分析结果。

### Lines 113-138

```cpp
            UserInst == I.getParent()->getTerminator())
          return false;
        if (UserInst != &InsertPoint && !DT.dominates(&InsertPoint, U)) {
          // If UserInst is an instruction that appears later in the same BB as
          // I, then it is okay to move since I will still be available when
          // UserInst is executed.
          if (CheckForEntireBlock && I.getParent() == UserInst->getParent() &&
              DT.dominates(&I, UserInst))
            continue;
          return false;
        }
      }
  if (isReachedBefore(&InsertPoint, &I, &DT, PDT))
    for (const Value *Op : I.operands())
      if (auto *OpInst = dyn_cast<Instruction>(Op)) {
        if (&InsertPoint == OpInst)
          return false;
        // If OpInst is an instruction that appears earlier in the same BB as
        // I, then it is okay to move since OpInst will still be available.
        if (CheckForEntireBlock && I.getParent() == OpInst->getParent() &&
            DT.dominates(OpInst, &I))
          continue;
        if (!DT.dominates(OpInst, &InsertPoint))
          return false;
      }

```
- EN: This region continues the CodeMoverUtils implementation with local helper logic centered on UserInst, InsertPoint, CheckForEntireBlock, PDT.
- CN: 这一段延续了 CodeMoverUtils 的主体实现，围绕 UserInst, InsertPoint, CheckForEntireBlock, PDT 等局部辅助逻辑展开。

### Lines 139-167

```cpp
  DT.updateDFSNumbers();
  const bool MoveForward = domTreeLevelBefore(&DT, &I, &InsertPoint);
  Instruction &StartInst = (MoveForward ? I : InsertPoint);
  Instruction &EndInst = (MoveForward ? InsertPoint : I);
  SmallPtrSet<Instruction *, 10> InstsToCheck;
  collectInstructionsInBetween(StartInst, EndInst, InstsToCheck);
  if (!MoveForward)
    InstsToCheck.insert(&InsertPoint);

  // Check if there exists instructions which may throw, may synchonize, or may
  // never return, from I to InsertPoint.
  if (!isSafeToSpeculativelyExecute(&I))
    if (llvm::any_of(InstsToCheck, [](Instruction *I) {
          if (I->mayThrow())
            return true;

          const CallBase *CB = dyn_cast<CallBase>(I);
          if (!CB)
            return false;
          if (!CB->hasFnAttr(Attribute::WillReturn))
            return true;
          if (!CB->hasFnAttr(Attribute::NoSync))
            return true;

          return false;
        })) {
      return reportInvalidCandidate(I, MayThrowException);
    }

```
- EN: This region continues the CodeMoverUtils implementation with local helper logic centered on MoveForward, InsertPoint, Instruction, StartInst.
- CN: 这一段延续了 CodeMoverUtils 的主体实现，围绕 MoveForward, InsertPoint, Instruction, StartInst 等局部辅助逻辑展开。

### Lines 168-193

```cpp
  // Check if I has any output/flow/anti dependences with instructions from \p
  // StartInst to \p EndInst.
  if (llvm::any_of(InstsToCheck, [&DI, &I](Instruction *CurInst) {
        auto DepResult = DI->depends(&I, CurInst);
        if (DepResult && (DepResult->isOutput() || DepResult->isFlow() ||
                          DepResult->isAnti()))
          return true;
        return false;
      }))
    return reportInvalidCandidate(I, HasDependences);

  return true;
}

bool llvm::isSafeToMoveBefore(BasicBlock &BB, Instruction &InsertPoint,
                              DominatorTree &DT, const PostDominatorTree *PDT,
                              DependenceInfo *DI) {
  return llvm::all_of(BB, [&](Instruction &I) {
    if (BB.getTerminator() == &I)
      return true;

    return isSafeToMoveBefore(I, InsertPoint, DT, PDT, DI,
                              /*CheckForEntireBlock=*/true);
  });
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, PostDominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, PostDominatorTree 等分析结果。

### Lines 194-220

```cpp
void llvm::moveInstructionsToTheBeginning(BasicBlock &FromBB, BasicBlock &ToBB,
                                          DominatorTree &DT,
                                          const PostDominatorTree &PDT,
                                          DependenceInfo &DI,
                                          ScalarEvolution &SE) {
  for (Instruction &I :
       llvm::make_early_inc_range(llvm::drop_begin(llvm::reverse(FromBB)))) {
    BasicBlock::iterator MovePos = ToBB.getFirstNonPHIOrDbg();

    if (isSafeToMoveBefore(I, *MovePos, DT, &PDT, &DI)) {
      // Update SCEV to ensure it remains valid throughout the function.
      SE.forgetValue(&I);
      I.moveBeforePreserving(MovePos);
    }
  }
}

void llvm::moveInstructionsToTheEnd(BasicBlock &FromBB, BasicBlock &ToBB,
                                    DominatorTree &DT,
                                    const PostDominatorTree &PDT,
                                    DependenceInfo &DI, ScalarEvolution &SE) {
  Instruction *MovePos = ToBB.getTerminator();
  while (FromBB.size() > 1) {
    Instruction &I = FromBB.front();
    if (isSafeToMoveBefore(I, *MovePos, DT, &PDT, &DI)) {
      // Update SCEV to ensure it remains valid throughout the function.
      SE.forgetValue(&I);
```
- EN: Core entities appearing here include make_early_inc_range, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, PostDominatorTree, ScalarEvolution, SCEV.
- CN: 此处出现的核心实体包括 make_early_inc_range，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, PostDominatorTree, ScalarEvolution, SCEV 等分析结果。

### Lines 221-246

```cpp
      I.moveBeforePreserving(MovePos->getIterator());
    }
  }
}

bool llvm::nonStrictlyPostDominate(const BasicBlock *ThisBlock,
                                   const BasicBlock *OtherBlock,
                                   const DominatorTree *DT,
                                   const PostDominatorTree *PDT) {
  const BasicBlock *CommonDominator =
      DT->findNearestCommonDominator(ThisBlock, OtherBlock);
  if (CommonDominator == nullptr)
    return false;

  /// Recursively check the predecessors of \p ThisBlock up to
  /// their common dominator, and see if any of them post-dominates
  /// \p OtherBlock.
  SmallVector<const BasicBlock *, 8> WorkList;
  SmallPtrSet<const BasicBlock *, 8> Visited;
  WorkList.push_back(ThisBlock);
  while (!WorkList.empty()) {
    const BasicBlock *CurBlock = WorkList.pop_back_val();
    Visited.insert(CurBlock);
    if (PDT->dominates(CurBlock, OtherBlock))
      return true;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, PostDominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, PostDominatorTree 等分析结果。

### Lines 247-265

```cpp
    for (const auto *Pred : predecessors(CurBlock)) {
      if (Pred == CommonDominator || Visited.count(Pred))
        continue;
      WorkList.push_back(Pred);
    }
  }
  return false;
}

bool llvm::isReachedBefore(const Instruction *I0, const Instruction *I1,
                           const DominatorTree *DT,
                           const PostDominatorTree *PDT) {
  const BasicBlock *BB0 = I0->getParent();
  const BasicBlock *BB1 = I1->getParent();
  if (BB0 == BB1)
    return DT->dominates(I0, I1);

  return nonStrictlyPostDominate(BB1, BB0, DT, PDT);
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, PostDominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, PostDominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `make_early_inc_range` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`make_early_inc_range` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, PostDominatorTree, ScalarEvolution, SCEV, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, PostDominatorTree, ScalarEvolution, SCEV, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DependenceAnalysis.h`, `llvm/Analysis/PostDominators.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DependenceAnalysis.h`, `llvm/Analysis/PostDominators.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/Transforms/Utils/CodeMoverUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/Transforms/Utils/CodeMoverUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `PostDominatorTree`, `ScalarEvolution`, `SCEV`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `PostDominatorTree`, `ScalarEvolution`, `SCEV`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
