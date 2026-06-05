# StripGCRelocates.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/StripGCRelocates.cpp` | `llvm/lib/Transforms/Utils/StripGCRelocates.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements remove gc.relocates inserted by RewriteStatePoints within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 StripGCRelocates 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

```cpp
//===- StripGCRelocates.cpp - Remove gc.relocates inserted by RewriteStatePoints===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a little utility pass that removes the gc.relocates inserted by
// RewriteStatepointsForGC. Note that the generated IR is incorrect,
// but this is useful as a single pass in itself, for analysis of IR, without
// the GC.relocates. The statepoint and gc.result intrinsics would still be
// present.
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 16-29

```cpp
#include "llvm/Transforms/Utils/StripGCRelocates.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Statepoint.h"

using namespace llvm;

static bool stripGCRelocates(Function &F) {
  // Nothing to do for declarations.
  if (F.isDeclaration())
    return false;
  SmallVector<GCRelocateInst *, 20> GCRelocates;
  // TODO: We currently do not handle gc.relocates that are in landing pads,
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include stripGCRelocates, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 stripGCRelocates，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 30-46

```cpp
  // i.e. not bound to a single statepoint token.
  for (Instruction &I : instructions(F)) {
    if (auto *GCR = dyn_cast<GCRelocateInst>(&I))
      if (isa<GCStatepointInst>(GCR->getOperand(0)))
        GCRelocates.push_back(GCR);
  }
  // All gc.relocates are bound to a single statepoint token. The order of
  // visiting gc.relocates for deletion does not matter.
  for (GCRelocateInst *GCRel : GCRelocates) {
    Value *OrigPtr = GCRel->getDerivedPtr();
    Value *ReplaceGCRel = OrigPtr;

    // All gc_relocates are i8 addrspace(1)* typed, we need a bitcast from i8
    // addrspace(1)* to the type of the OrigPtr, if the are not the same.
    if (GCRel->getType() != OrigPtr->getType())
      ReplaceGCRel = new BitCastInst(OrigPtr, GCRel->getType(), "cast", GCRel->getIterator());

```
- EN: This region continues the StripGCRelocates implementation with local helper logic centered on Instruction, GCR, GCRelocateInst, GCStatepointInst.
- CN: 这一段延续了 StripGCRelocates 的主体实现，围绕 Instruction, GCR, GCRelocateInst, GCStatepointInst 等局部辅助逻辑展开。

### Lines 47-60

```cpp
    // Replace all uses of gc.relocate and delete the gc.relocate
    // There maybe unncessary bitcasts back to the OrigPtr type, an instcombine
    // pass would clear this up.
    GCRel->replaceAllUsesWith(ReplaceGCRel);
    GCRel->eraseFromParent();
  }
  return !GCRelocates.empty();
}

PreservedAnalyses StripGCRelocates::run(Function &F,
                                        FunctionAnalysisManager &AM) {
  if (!stripGCRelocates(F))
    return PreservedAnalyses::all();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 61-66

```cpp
  // Removing gc.relocate preserves the CFG, but most other analysis probably
  // need to re-run.
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
- EN: This region continues the StripGCRelocates implementation with local helper logic centered on Removing, CFG, PreservedAnalyses, CFGAnalyses.
- CN: 这一段延续了 StripGCRelocates 的主体实现，围绕 Removing, CFG, PreservedAnalyses, CFGAnalyses 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `stripGCRelocates` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`stripGCRelocates` 构成该文件对外 API 与主要实现挂钩。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Statepoint.h`, `llvm/Transforms/Utils/StripGCRelocates.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Statepoint.h`, `llvm/Transforms/Utils/StripGCRelocates.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
