# LowerConstantIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/LowerConstantIntrinsics.cpp` | `llvm/lib/Transforms/Scalar/LowerConstantIntrinsics.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements lower constant intrinsic calls within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 LowerConstantIntrinsics 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- LowerConstantIntrinsics.cpp - Lower constant intrinsic calls -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass lowers all remaining 'objectsize' 'is.constant' intrinsic calls
// and provides constant propagation and basic CFG cleanup on the result.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/LowerConstantIntrinsics.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/Statistic.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 18-35

```cpp
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/Analysis/InstructionSimplify.h"
#include "llvm/Analysis/MemoryBuiltins.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/Pass.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/Local.h"
#include <optional>

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, TargetLibraryInfo.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, TargetLibraryInfo 等分析结果。

### Lines 36-52

```cpp
using namespace llvm;
using namespace llvm::PatternMatch;

#define DEBUG_TYPE "lower-is-constant-intrinsic"

STATISTIC(IsConstantIntrinsicsHandled,
          "Number of 'is.constant' intrinsic calls handled");
STATISTIC(ObjectSizeIntrinsicsHandled,
          "Number of 'objectsize' intrinsic calls handled");

static Value *lowerIsConstantIntrinsic(IntrinsicInst *II) {
  if (auto *C = dyn_cast<Constant>(II->getOperand(0)))
    if (C->isManifestConstant())
      return ConstantInt::getTrue(II->getType());
  return ConstantInt::getFalse(II->getType());
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 53-70

```cpp
static bool replaceConditionalBranchesOnConstant(Instruction *II,
                                                 Value *NewValue,
                                                 DomTreeUpdater *DTU) {
  bool HasDeadBlocks = false;
  SmallSetVector<Instruction *, 8> UnsimplifiedUsers;
  replaceAndRecursivelySimplify(II, NewValue, nullptr, nullptr, nullptr,
                                &UnsimplifiedUsers);
  // UnsimplifiedUsers can contain PHI nodes that may be removed when
  // replacing the branch instructions, so use a value handle worklist
  // to handle those possibly removed instructions.
  SmallVector<WeakVH, 8> Worklist(UnsimplifiedUsers.begin(),
                                  UnsimplifiedUsers.end());

  for (auto &VH : Worklist) {
    CondBrInst *BI = dyn_cast_or_null<CondBrInst>(VH);
    if (!BI)
      continue;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。

### Lines 71-89

```cpp
    BasicBlock *Target, *Other;
    if (match(BI->getOperand(0), m_Zero())) {
      Target = BI->getSuccessor(1);
      Other = BI->getSuccessor(0);
    } else if (match(BI->getOperand(0), m_One())) {
      Target = BI->getSuccessor(0);
      Other = BI->getSuccessor(1);
    } else {
      Target = nullptr;
      Other = nullptr;
    }
    if (Target && Target != Other) {
      BasicBlock *Source = BI->getParent();
      Other->removePredecessor(Source);

      Instruction *NewBI = UncondBrInst::Create(Target, Source);
      NewBI->setDebugLoc(BI->getDebugLoc());
      BI->eraseFromParent();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 90-108

```cpp
      if (DTU)
        DTU->applyUpdates({{DominatorTree::Delete, Source, Other}});
      if (pred_empty(Other))
        HasDeadBlocks = true;
    }
  }
  return HasDeadBlocks;
}

bool llvm::lowerConstantIntrinsics(Function &F, const TargetLibraryInfo &TLI,
                                   DominatorTree *DT) {
  std::optional<DomTreeUpdater> DTU;
  if (DT)
    DTU.emplace(DT, DomTreeUpdater::UpdateStrategy::Lazy);

  bool HasDeadBlocks = false;
  const auto &DL = F.getDataLayout();
  SmallVector<WeakTrackingVH, 8> Worklist;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 109-125

```cpp
  ReversePostOrderTraversal<Function *> RPOT(&F);
  for (BasicBlock *BB : RPOT) {
    for (Instruction &I: *BB) {
      IntrinsicInst *II = dyn_cast<IntrinsicInst>(&I);
      if (!II)
        continue;
      switch (II->getIntrinsicID()) {
      default:
        break;
      case Intrinsic::is_constant:
      case Intrinsic::objectsize:
        Worklist.push_back(WeakTrackingVH(&I));
        break;
      }
    }
  }
  for (WeakTrackingVH &VH: Worklist) {
```
- EN: This region continues the LowerConstantIntrinsics implementation with local helper logic centered on ReversePostOrderTraversal, Function, RPOT, BasicBlock.
- CN: 这一段延续了 LowerConstantIntrinsics 的主体实现，围绕 ReversePostOrderTraversal, Function, RPOT, BasicBlock 等局部辅助逻辑展开。

### Lines 126-142

```cpp
    // Items on the worklist can be mutated by earlier recursive replaces.
    // This can remove the intrinsic as dead (VH == null), but also replace
    // the intrinsic in place.
    if (!VH)
      continue;
    IntrinsicInst *II = dyn_cast<IntrinsicInst>(&*VH);
    if (!II)
      continue;
    Value *NewValue;
    switch (II->getIntrinsicID()) {
    default:
      continue;
    case Intrinsic::is_constant:
      NewValue = lowerIsConstantIntrinsic(II);
      LLVM_DEBUG(dbgs() << "Folding " << *II << " to " << *NewValue << "\n");
      IsConstantIntrinsicsHandled++;
      break;
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 143-156

```cpp
    case Intrinsic::objectsize:
      NewValue = lowerObjectSizeCall(II, DL, &TLI, true);
      LLVM_DEBUG(dbgs() << "Folding " << *II << " to " << *NewValue << "\n");
      ObjectSizeIntrinsicsHandled++;
      break;
    }
    HasDeadBlocks |= replaceConditionalBranchesOnConstant(
        II, NewValue, DTU ? &*DTU : nullptr);
  }
  if (HasDeadBlocks)
    removeUnreachableBlocks(F, DTU ? &*DTU : nullptr);
  return !Worklist.empty();
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 157-167

```cpp
PreservedAnalyses
LowerConstantIntrinsicsPass::run(Function &F, FunctionAnalysisManager &AM) {
  if (lowerConstantIntrinsics(F, AM.getResult<TargetLibraryAnalysis>(F),
                              AM.getCachedResult<DominatorTreeAnalysis>(F))) {
    PreservedAnalyses PA;
    PA.preserve<DominatorTreeAnalysis>();
    return PA;
  }

  return PreservedAnalyses::all();
}
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, DomTreeUpdater, TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/TargetLibraryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/TargetLibraryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Scalar/LowerConstantIntrinsics.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/Local.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Scalar/LowerConstantIntrinsics.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/Local.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/Statistic.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/Statistic.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `DomTreeUpdater`, `TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
