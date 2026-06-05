# MakeGuardsExplicit.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/MakeGuardsExplicit.cpp` | `llvm/lib/Transforms/Scalar/MakeGuardsExplicit.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements turn guard intrinsics into guard branches within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 MakeGuardsExplicit 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

```cpp
//===- MakeGuardsExplicit.cpp - Turn guard intrinsics into guard branches -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass lowers the @llvm.experimental.guard intrinsic to the new form of
// guard represented as widenable explicit branch to the deopt block. The
// difference between this pass and LowerGuardIntrinsic is that after this pass
// the guard represented as intrinsic:
//
//   call void(i1, ...) @llvm.experimental.guard(i1 %old_cond) [ "deopt"() ]
//
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 16-33

```cpp
// transforms to a guard represented as widenable explicit branch:
//
//   %widenable_cond = call i1 @llvm.experimental.widenable.condition()
//   br i1 (%old_cond & %widenable_cond), label %guarded, label %deopt
//
// Here:
//   - The semantics of @llvm.experimental.widenable.condition allows to replace
//     %widenable_cond with the construction (%widenable_cond & %any_other_cond)
//     without loss of correctness;
//   - %guarded is the lower part of old guard intrinsic's parent block split by
//     the intrinsic call;
//   - %deopt is a block containing a sole call to @llvm.experimental.deoptimize
//     intrinsic.
//
// Therefore, this branch preserves the property of widenability.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 34-50

```cpp
#include "llvm/Transforms/Scalar/MakeGuardsExplicit.h"
#include "llvm/Analysis/GuardUtils.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils/GuardUtils.h"

using namespace llvm;

static void turnToExplicitForm(CallInst *Guard, Function *DeoptIntrinsic) {
  // Replace the guard with an explicit branch (just like in GuardWidening).
  BasicBlock *OriginalBB = Guard->getParent();
  (void)OriginalBB;
  makeGuardControlFlowExplicit(DeoptIntrinsic, Guard, true);
  assert(isWidenableBranch(OriginalBB->getTerminator()) && "should hold");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include turnToExplicitForm, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 turnToExplicitForm，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 51-66

```cpp
  Guard->eraseFromParent();
}

static bool explicifyGuards(Function &F) {
  // Check if we can cheaply rule out the possibility of not having any work to
  // do.
  auto *GuardDecl = Intrinsic::getDeclarationIfExists(
      F.getParent(), Intrinsic::experimental_guard);
  if (!GuardDecl || GuardDecl->use_empty())
    return false;

  SmallVector<CallInst *, 8> GuardIntrinsics;
  for (auto &I : instructions(F))
    if (isGuard(&I))
      GuardIntrinsics.push_back(cast<CallInst>(&I));

```
- EN: Core entities appearing here include explicifyGuards, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 explicifyGuards，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 67-79

```cpp
  if (GuardIntrinsics.empty())
    return false;

  auto *DeoptIntrinsic = Intrinsic::getOrInsertDeclaration(
      F.getParent(), Intrinsic::experimental_deoptimize, {F.getReturnType()});
  DeoptIntrinsic->setCallingConv(GuardDecl->getCallingConv());

  for (auto *Guard : GuardIntrinsics)
    turnToExplicitForm(Guard, DeoptIntrinsic);

  return true;
}

```
- EN: This region continues the MakeGuardsExplicit implementation with local helper logic centered on GuardIntrinsics, DeoptIntrinsic, Intrinsic, GuardDecl.
- CN: 这一段延续了 MakeGuardsExplicit 的主体实现，围绕 GuardIntrinsics, DeoptIntrinsic, Intrinsic, GuardDecl 等局部辅助逻辑展开。

### Lines 80-85

```cpp
PreservedAnalyses MakeGuardsExplicitPass::run(Function &F,
                                           FunctionAnalysisManager &) {
  if (explicifyGuards(F))
    return PreservedAnalyses::none();
  return PreservedAnalyses::all();
}
```
- EN: This region continues the MakeGuardsExplicit implementation with local helper logic centered on PreservedAnalyses, MakeGuardsExplicitPass, Function, FunctionAnalysisManager.
- CN: 这一段延续了 MakeGuardsExplicit 的主体实现，围绕 PreservedAnalyses, MakeGuardsExplicitPass, Function, FunctionAnalysisManager 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `turnToExplicitForm, explicifyGuards` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`turnToExplicitForm, explicifyGuards` 构成该文件对外 API 与主要实现挂钩。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/GuardUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/GuardUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/Transforms/Scalar/MakeGuardsExplicit.h`, `llvm/Transforms/Utils/GuardUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/Transforms/Scalar/MakeGuardsExplicit.h`, `llvm/Transforms/Utils/GuardUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
