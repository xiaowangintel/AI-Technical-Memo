# SCCP.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/SCCP.cpp` | `llvm/lib/Transforms/Scalar/SCCP.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements sparse Conditional Constant Propagation within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 SCCP 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- SCCP.cpp - Sparse Conditional Constant Propagation -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements sparse conditional constant propagation and merging:
//
// Specifically, this:
//   * Assumes values are constant unless proven otherwise
//   * Assumes BasicBlocks are dead unless proven otherwise
//   * Proves values to be constant, and replaces them with constants
//   * Proves conditional branches to be unconditional
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 19-32

```cpp
#include "llvm/Transforms/Scalar/SCCP.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/AssumptionCache.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/Analysis/ValueLatticeUtils.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InstrTypes.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DomTreeUpdater, TargetLibraryInfo, ValueTracking.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DomTreeUpdater, TargetLibraryInfo, ValueTracking 等分析结果。

### Lines 33-47

```cpp
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/PassManager.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Value.h"
#include "llvm/Pass.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Transforms/Utils/SCCPSolver.h"

using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 48-64

```cpp
#define DEBUG_TYPE "sccp"

STATISTIC(NumInstRemoved, "Number of instructions removed");
STATISTIC(NumDeadBlocks , "Number of basic blocks unreachable");
STATISTIC(NumInstReplaced,
          "Number of instructions replaced with (simpler) instruction");

// runSCCP() - Run the Sparse Conditional Constant Propagation algorithm,
// and return true if the function was modified.
static bool runSCCP(Function &F, const DataLayout &DL,
                    const TargetLibraryInfo *TLI, DominatorTree &DT,
                    AssumptionCache &AC) {
  LLVM_DEBUG(dbgs() << "SCCP on function '" << F.getName() << "'\n");
  SCCPSolver Solver(
      DL, [TLI](Function &F) -> const TargetLibraryInfo & { return *TLI; },
      F.getContext());

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DataLayout, DominatorTree, TargetLibraryInfo.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DataLayout, DominatorTree, TargetLibraryInfo 等分析结果。

### Lines 65-78

```cpp
  Solver.addPredicateInfo(F, DT, AC);

  // While we don't do any actual inter-procedural analysis, still track
  // return values so we can infer attributes.
  if (canTrackReturnsInterprocedurally(&F))
    Solver.addTrackedFunction(&F);

  // Mark the first block of the function as being executable.
  Solver.markBlockExecutable(&F.front());

  // Initialize arguments based on attributes.
  for (Argument &AI : F.args())
    Solver.trackValueOfArgument(&AI);

```
- EN: This region continues the SCCP implementation with local helper logic centered on Solver, While, Mark, Initialize.
- CN: 这一段延续了 SCCP 的主体实现，围绕 Solver, While, Mark, Initialize 等局部辅助逻辑展开。

### Lines 79-92

```cpp
  // Solve for constants.
  bool ResolvedUndefs = true;
  while (ResolvedUndefs) {
    Solver.solve();
    LLVM_DEBUG(dbgs() << "RESOLVING UNDEFs\n");
    ResolvedUndefs = Solver.resolvedUndefsIn(F);
  }

  bool MadeChanges = false;

  // If we decided that there are basic blocks that are dead in this function,
  // delete their contents now.  Note that we cannot actually delete the blocks,
  // as we cannot modify the CFG of the function.

```
- EN: This region continues the SCCP implementation with local helper logic centered on Solve, ResolvedUndefs, Solver, LLVM_DEBUG.
- CN: 这一段延续了 SCCP 的主体实现，围绕 Solve, ResolvedUndefs, Solver, LLVM_DEBUG 等局部辅助逻辑展开。

### Lines 93-107

```cpp
  SmallPtrSet<Value *, 32> InsertedValues;
  SmallVector<BasicBlock *, 8> BlocksToErase;
  for (BasicBlock &BB : F) {
    if (!Solver.isBlockExecutable(&BB)) {
      LLVM_DEBUG(dbgs() << "  BasicBlock Dead:" << BB);
      ++NumDeadBlocks;
      BlocksToErase.push_back(&BB);
      MadeChanges = true;
      continue;
    }

    MadeChanges |= Solver.simplifyInstsInBlock(BB, InsertedValues,
                                               NumInstRemoved, NumInstReplaced);
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 108-123

```cpp
  // Remove unreachable blocks and non-feasible edges.
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Lazy);
  for (BasicBlock *DeadBB : BlocksToErase)
    NumInstRemoved += changeToUnreachable(&*DeadBB->getFirstNonPHIIt(),
                                          /*PreserveLCSSA=*/false, &DTU);

  BasicBlock *NewUnreachableBB = nullptr;
  for (BasicBlock &BB : F)
    MadeChanges |= Solver.removeNonFeasibleEdges(&BB, DTU, NewUnreachableBB);

  for (BasicBlock *DeadBB : BlocksToErase)
    if (!DeadBB->hasAddressTaken())
      DTU.deleteBB(DeadBB);

  Solver.removeSSACopies(F);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。

### Lines 124-136

```cpp
  Solver.inferReturnAttributes();

  return MadeChanges;
}

PreservedAnalyses SCCPPass::run(Function &F, FunctionAnalysisManager &AM) {
  const DataLayout &DL = F.getDataLayout();
  auto &TLI = AM.getResult<TargetLibraryAnalysis>(F);
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
  auto &AC = AM.getResult<AssumptionAnalysis>(F);
  if (!runSCCP(F, DL, &TLI, DT, AC))
    return PreservedAnalyses::all();

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。

### Lines 137-140

```cpp
  auto PA = PreservedAnalyses();
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/ValueLatticeUtils.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/ValueLatticeUtils.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PassManager.h`, `llvm/IR/Type.h`, `llvm/IR/Value.h`, `llvm/Transforms/Scalar/SCCP.h`, `llvm/Transforms/Scalar.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PassManager.h`, `llvm/IR/Type.h`, `llvm/IR/Value.h`, `llvm/Transforms/Scalar/SCCP.h`, `llvm/Transforms/Scalar.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
