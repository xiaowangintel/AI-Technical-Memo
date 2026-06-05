# MoveAutoInit.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/MoveAutoInit.cpp` | `llvm/lib/Transforms/Utils/MoveAutoInit.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements move auto-init inst closer to their use site within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 MoveAutoInit 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-26

```cpp
//===-- MoveAutoInit.cpp - move auto-init inst closer to their use site----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass moves instruction maked as auto-init closer to the basic block that
// use it, eventually removing it from some control path of the function.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/MoveAutoInit.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/MemorySSA.h"
#include "llvm/Analysis/MemorySSAUpdater.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/DebugInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/LoopUtils.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as MemorySSA, MemorySSAUpdater, ValueTracking.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 MemorySSA, MemorySSAUpdater, ValueTracking 等分析结果。

### Lines 27-51

```cpp
using namespace llvm;

#define DEBUG_TYPE "move-auto-init"

STATISTIC(NumMoved, "Number of instructions moved");

static cl::opt<unsigned> MoveAutoInitThreshold(
    "move-auto-init-threshold", cl::Hidden, cl::init(128),
    cl::desc("Maximum instructions to analyze per moved initialization"));

static bool hasAutoInitMetadata(const Instruction &I) {
  return I.hasMetadata(LLVMContext::MD_annotation) &&
         any_of(I.getMetadata(LLVMContext::MD_annotation)->operands(),
                [](const MDOperand &Op) { return Op.equalsStr("auto-init"); });
}

static std::optional<MemoryLocation> writeToAlloca(const Instruction &I) {
  MemoryLocation ML;
  if (auto *MI = dyn_cast<MemIntrinsic>(&I))
    ML = MemoryLocation::getForDest(MI);
  else if (auto *SI = dyn_cast<StoreInst>(&I))
    ML = MemoryLocation::get(SI);
  else
    return std::nullopt;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include hasAutoInitMetadata, writeToAlloca, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 hasAutoInitMetadata, writeToAlloca，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 52-76

```cpp
  if (isa<AllocaInst>(getUnderlyingObject(ML.Ptr)))
    return ML;
  else
    return {};
}

/// Finds a BasicBlock in the CFG where instruction `I` can be moved to while
/// not changing the Memory SSA ordering and being guarded by at least one
/// condition.
static BasicBlock *usersDominator(const MemoryLocation &ML, Instruction *I,
                                  DominatorTree &DT, MemorySSA &MSSA) {
  BasicBlock *CurrentDominator = nullptr;
  MemoryUseOrDef &IMA = *MSSA.getMemoryAccess(I);
  BatchAAResults AA(MSSA.getAA());

  SmallPtrSet<MemoryAccess *, 8> Visited;

  auto AsMemoryAccess = [](User *U) { return cast<MemoryAccess>(U); };
  SmallVector<MemoryAccess *> WorkList(map_range(IMA.users(), AsMemoryAccess));

  while (!WorkList.empty()) {
    MemoryAccess *MA = WorkList.pop_back_val();
    if (!Visited.insert(MA).second)
      continue;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 77-103

```cpp
    if (Visited.size() > MoveAutoInitThreshold)
      return nullptr;

    bool FoundClobberingUser = false;
    if (auto *M = dyn_cast<MemoryUseOrDef>(MA)) {
      Instruction *MI = M->getMemoryInst();

      // If this memory instruction may not clobber `I`, we can skip it.
      // LifetimeEnd is a valid user, but we do not want it in the user
      // dominator.
      if (AA.getModRefInfo(MI, ML) != ModRefInfo::NoModRef &&
          !MI->isLifetimeStartOrEnd() && MI != I) {
        FoundClobberingUser = true;
        CurrentDominator = CurrentDominator
                               ? DT.findNearestCommonDominator(CurrentDominator,
                                                               MI->getParent())
                               : MI->getParent();
      }
    }
    if (!FoundClobberingUser) {
      auto UsersAsMemoryAccesses = map_range(MA->users(), AsMemoryAccess);
      append_range(WorkList, UsersAsMemoryAccesses);
    }
  }
  return CurrentDominator;
}

```
- EN: This region continues the MoveAutoInit implementation with local helper logic centered on Visited, MoveAutoInitThreshold, FoundClobberingUser, MemoryUseOrDef.
- CN: 这一段延续了 MoveAutoInit 的主体实现，围绕 Visited, MoveAutoInitThreshold, FoundClobberingUser, MemoryUseOrDef 等局部辅助逻辑展开。

### Lines 104-128

```cpp
static bool runMoveAutoInit(Function &F, DominatorTree &DT, MemorySSA &MSSA) {
  BasicBlock &EntryBB = F.getEntryBlock();
  SmallVector<std::pair<Instruction *, BasicBlock *>> JobList;

  //
  // Compute movable instructions.
  //
  for (Instruction &I : EntryBB) {
    if (!hasAutoInitMetadata(I))
      continue;

    std::optional<MemoryLocation> ML = writeToAlloca(I);
    if (!ML)
      continue;

    if (I.isVolatile())
      continue;

    BasicBlock *UsersDominator = usersDominator(ML.value(), &I, DT, MSSA);
    if (!UsersDominator)
      continue;

    if (UsersDominator == &EntryBB)
      continue;

```
- EN: Core entities appearing here include runMoveAutoInit, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, MemorySSA.
- CN: 此处出现的核心实体包括 runMoveAutoInit，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, MemorySSA 等分析结果。

### Lines 129-153

```cpp
    // Traverse the CFG to detect cycles `UsersDominator` would be part of.
    SmallPtrSet<BasicBlock *, 8> TransitiveSuccessors;
    SmallVector<BasicBlock *> WorkList(successors(UsersDominator));
    bool HasCycle = false;
    while (!WorkList.empty()) {
      BasicBlock *CurrBB = WorkList.pop_back_val();
      if (CurrBB == UsersDominator)
        // No early exit because we want to compute the full set of transitive
        // successors.
        HasCycle = true;
      for (BasicBlock *Successor : successors(CurrBB)) {
        if (!TransitiveSuccessors.insert(Successor).second)
          continue;
        WorkList.push_back(Successor);
      }
    }

    // Don't insert if that could create multiple execution of I,
    // but we can insert it in the non back-edge predecessors, if it exists.
    if (HasCycle) {
      BasicBlock *UsersDominatorHead = UsersDominator;
      while (BasicBlock *UniquePredecessor =
                 UsersDominatorHead->getUniquePredecessor())
        UsersDominatorHead = UniquePredecessor;

```
- EN: This region continues the MoveAutoInit implementation with local helper logic centered on Traverse, CFG, UsersDominator, SmallPtrSet.
- CN: 这一段延续了 MoveAutoInit 的主体实现，围绕 Traverse, CFG, UsersDominator, SmallPtrSet 等局部辅助逻辑展开。

### Lines 154-180

```cpp
      if (UsersDominatorHead == &EntryBB)
        continue;

      BasicBlock *DominatingPredecessor = nullptr;
      for (BasicBlock *Pred : predecessors(UsersDominatorHead)) {
        // If one of the predecessor of the dominator also transitively is a
        // successor, moving to the dominator would do the inverse of loop
        // hoisting, and we don't want that.
        if (TransitiveSuccessors.count(Pred))
          continue;

        if (!DT.isReachableFromEntry(Pred))
          continue;
        if (!DT.dominates(Pred, UsersDominatorHead))
          continue;
        DominatingPredecessor =
            DominatingPredecessor
                ? DT.findNearestCommonDominator(DominatingPredecessor, Pred)
                : Pred;
      }

      if (!DominatingPredecessor || DominatingPredecessor == &EntryBB)
        continue;

      UsersDominator = DominatingPredecessor;
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 181-202

```cpp
    // CatchSwitchInst blocks can only have one instruction, so they are not
    // good candidates for insertion.
    while (isa<CatchSwitchInst>(UsersDominator->getFirstNonPHIIt())) {
      for (BasicBlock *Pred : predecessors(UsersDominator))
        if (DT.isReachableFromEntry(Pred))
          UsersDominator = DT.findNearestCommonDominator(UsersDominator, Pred);
    }

    // We finally found a place where I can be moved while not introducing extra
    // execution, and guarded by at least one condition.
    if (UsersDominator != &EntryBB)
      JobList.emplace_back(&I, UsersDominator);
  }

  //
  // Perform the actual substitution.
  //
  if (JobList.empty())
    return false;

  MemorySSAUpdater MSSAU(&MSSA);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as MemorySSA, MemorySSAUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 MemorySSA, MemorySSAUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 203-227

```cpp
  // Reverse insertion to respect relative order between instructions:
  // if two instructions are moved from the same BB to the same BB, we insert
  // the second one in the front, then the first on top of it.
  for (auto &Job : reverse(JobList)) {
    Job.first->moveBefore(*Job.second, Job.second->getFirstInsertionPt());
    MSSAU.moveToPlace(MSSA.getMemoryAccess(Job.first), Job.first->getParent(),
                      MemorySSA::InsertionPlace::Beginning);
  }

  if (VerifyMemorySSA)
    MSSA.verifyMemorySSA();

  NumMoved += JobList.size();

  return true;
}

PreservedAnalyses MoveAutoInitPass::run(Function &F,
                                        FunctionAnalysisManager &AM) {

  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
  auto &MSSA = AM.getResult<MemorySSAAnalysis>(F).getMSSA();
  if (!runMoveAutoInit(F, DT, MSSA))
    return PreservedAnalyses::all();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, MemorySSA.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, MemorySSA 等分析结果。

### Lines 228-233

```cpp
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  PA.preserve<MemorySSAAnalysis>();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, MemorySSA.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, MemorySSA 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `hasAutoInitMetadata, writeToAlloca, runMoveAutoInit` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`hasAutoInitMetadata, writeToAlloca, runMoveAutoInit` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, DominatorTree, MemorySSA, MemorySSAUpdater, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, DominatorTree, MemorySSA, MemorySSAUpdater, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `MoveAutoInitThreshold` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `MoveAutoInitThreshold` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DebugInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/Transforms/Utils/MoveAutoInit.h`, `llvm/Transforms/Utils/LoopUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DebugInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/Transforms/Utils/MoveAutoInit.h`, `llvm/Transforms/Utils/LoopUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AAResults`, `DominatorTree`, `MemorySSA`, `MemorySSAUpdater`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `DominatorTree`, `MemorySSA`, `MemorySSAUpdater`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
