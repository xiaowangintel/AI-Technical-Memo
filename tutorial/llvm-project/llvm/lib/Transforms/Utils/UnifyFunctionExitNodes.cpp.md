# UnifyFunctionExitNodes.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/UnifyFunctionExitNodes.cpp` | `llvm/lib/Transforms/Utils/UnifyFunctionExitNodes.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements make all functions have a single exit within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 UnifyFunctionExitNodes 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- UnifyFunctionExitNodes.cpp - Make all functions have a single exit -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass is used to ensure that functions have at most one return and one
// unreachable instruction in them.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 14-29

```cpp
#include "llvm/Transforms/Utils/UnifyFunctionExitNodes.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Type.h"
using namespace llvm;

namespace {

bool unifyUnreachableBlocks(Function &F) {
  std::vector<BasicBlock *> UnreachableBlocks;

  for (BasicBlock &I : F)
    if (isa<UnreachableInst>(I.getTerminator()))
      UnreachableBlocks.push_back(&I);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include unifyUnreachableBlocks, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 unifyUnreachableBlocks，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 30-44

```cpp
  if (UnreachableBlocks.size() <= 1)
    return false;

  BasicBlock *UnreachableBlock =
      BasicBlock::Create(F.getContext(), "UnifiedUnreachableBlock", &F);
  new UnreachableInst(F.getContext(), UnreachableBlock);

  for (BasicBlock *BB : UnreachableBlocks) {
    BB->back().eraseFromParent(); // Remove the unreachable inst.
    UncondBrInst::Create(UnreachableBlock, BB);
  }

  return true;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 45-60

```cpp
bool unifyReturnBlocks(Function &F) {
  std::vector<BasicBlock *> ReturningBlocks;

  for (BasicBlock &I : F)
    if (isa<ReturnInst>(I.getTerminator()))
      ReturningBlocks.push_back(&I);

  if (ReturningBlocks.size() <= 1)
    return false;

  // Insert a new basic block into the function, add PHI nodes (if the function
  // returns values), and convert all of the return instructions into
  // unconditional branches.
  BasicBlock *NewRetBlock = BasicBlock::Create(F.getContext(),
                                               "UnifiedReturnBlock", &F);

```
- EN: Core entities appearing here include unifyReturnBlocks, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 unifyReturnBlocks，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 61-71

```cpp
  PHINode *PN = nullptr;
  if (F.getReturnType()->isVoidTy()) {
    ReturnInst::Create(F.getContext(), nullptr, NewRetBlock);
  } else {
    // If the function doesn't return void... add a PHI node to the block...
    PN = PHINode::Create(F.getReturnType(), ReturningBlocks.size(),
                         "UnifiedRetVal");
    PN->insertInto(NewRetBlock, NewRetBlock->end());
    ReturnInst::Create(F.getContext(), PN, NewRetBlock);
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 72-87

```cpp
  // Loop over all of the blocks, replacing the return instruction with an
  // unconditional branch.
  for (BasicBlock *BB : ReturningBlocks) {
    // Add an incoming element to the PHI node for every return instruction that
    // is merging into this new block...
    if (PN)
      PN->addIncoming(BB->getTerminator()->getOperand(0), BB);

    BB->back().eraseFromParent(); // Remove the return insn
    UncondBrInst::Create(NewRetBlock, BB);
  }

  return true;
}
} // namespace

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 88-94

```cpp
PreservedAnalyses UnifyFunctionExitNodesPass::run(Function &F,
                                                  FunctionAnalysisManager &AM) {
  bool Changed = false;
  Changed |= unifyUnreachableBlocks(F);
  Changed |= unifyReturnBlocks(F);
  return Changed ? PreservedAnalyses() : PreservedAnalyses::all();
}
```
- EN: This region continues the UnifyFunctionExitNodes implementation with local helper logic centered on PreservedAnalyses, UnifyFunctionExitNodesPass, Function, FunctionAnalysisManager.
- CN: 这一段延续了 UnifyFunctionExitNodes 的主体实现，围绕 PreservedAnalyses, UnifyFunctionExitNodesPass, Function, FunctionAnalysisManager 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `unifyUnreachableBlocks, unifyReturnBlocks` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`unifyUnreachableBlocks, unifyReturnBlocks` 构成该文件对外 API 与主要实现挂钩。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/UnifyFunctionExitNodes.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/UnifyFunctionExitNodes.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
