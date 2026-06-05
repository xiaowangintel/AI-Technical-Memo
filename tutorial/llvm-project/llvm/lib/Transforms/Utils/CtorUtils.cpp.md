# CtorUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CtorUtils.cpp` | `llvm/lib/Transforms/Utils/CtorUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements helpers for working with global_ctors within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CtorUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
//===- CtorUtils.cpp - Helpers for working with global_ctors ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines functions that are used to process llvm.global_ctors.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/CtorUtils.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。

### Lines 17-35

```cpp
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <numeric>

#define DEBUG_TYPE "ctor_utils"

using namespace llvm;

/// Given a specified llvm.global_ctors list, remove the listed elements.
static void removeGlobalCtors(GlobalVariable *GCL, const BitVector &CtorsToRemove) {
  // Filter out the initializer elements to remove.
  ConstantArray *OldCA = cast<ConstantArray>(GCL->getInitializer());
  SmallVector<Constant *, 10> CAList;
  for (unsigned I = 0, E = OldCA->getNumOperands(); I < E; ++I)
    if (!CtorsToRemove.test(I))
      CAList.push_back(OldCA->getOperand(I));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 36-53

```cpp
  // Create the new array initializer.
  ArrayType *ATy =
      ArrayType::get(OldCA->getType()->getElementType(), CAList.size());
  Constant *CA = ConstantArray::get(ATy, CAList);

  // If we didn't change the number of elements, don't create a new GV.
  if (CA->getType() == OldCA->getType()) {
    GCL->setInitializer(CA);
    return;
  }

  // Create the new global and insert it next to the existing list.
  GlobalVariable *NGV = new GlobalVariable(
      CA->getType(), GCL->isConstant(), GCL->getLinkage(), CA, "",
      GCL->getThreadLocalMode(), GCL->getAddressSpace());
  GCL->getParent()->insertGlobalVariable(GCL->getIterator(), NGV);
  NGV->takeName(GCL);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 54-69

```cpp
  // Nuke the old list, replacing any uses with the new one.
  if (!GCL->use_empty())
    GCL->replaceAllUsesWith(NGV);

  GCL->eraseFromParent();
}

/// Given a llvm.global_ctors list that we can understand,
/// return a list of the functions and null terminator as a vector.
static std::vector<std::pair<uint32_t, Function *>>
parseGlobalCtors(GlobalVariable *GV) {
  ConstantArray *CA = cast<ConstantArray>(GV->getInitializer());
  std::vector<std::pair<uint32_t, Function *>> Result;
  Result.reserve(CA->getNumOperands());
  for (auto &V : CA->operands()) {
    ConstantStruct *CS = cast<ConstantStruct>(V);
```
- EN: Core entities appearing here include parseGlobalCtors, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 parseGlobalCtors，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 70-86

```cpp
    Result.emplace_back(cast<ConstantInt>(CS->getOperand(0))->getZExtValue(),
                        dyn_cast<Function>(CS->getOperand(1)));
  }
  return Result;
}

/// Find the llvm.global_ctors list.
static GlobalVariable *findGlobalCtors(Module &M) {
  GlobalVariable *GV = M.getGlobalVariable("llvm.global_ctors");
  if (!GV)
    return nullptr;

  // Verify that the initializer is simple enough for us to handle. We are
  // only allowed to optimize the initializer if it is unique.
  if (!GV->hasUniqueInitializer())
    return nullptr;

```
- EN: This region continues the CtorUtils implementation with local helper logic centered on Result, ConstantInt, Function, Find.
- CN: 这一段延续了 CtorUtils 的主体实现，围绕 Result, ConstantInt, Function, Find 等局部辅助逻辑展开。

### Lines 87-99

```cpp
  // If there are no ctors, then the initializer might be null/undef/poison.
  // Ignore anything but an array.
  ConstantArray *CA = dyn_cast<ConstantArray>(GV->getInitializer());
  if (!CA)
    return nullptr;

  for (auto &V : CA->operands()) {
    if (isa<ConstantAggregateZero>(V))
      continue;
    ConstantStruct *CS = cast<ConstantStruct>(V);
    if (isa<ConstantPointerNull>(CS->getOperand(1)))
      continue;

```
- EN: This region continues the CtorUtils implementation with local helper logic centered on Ignore, ConstantArray, ConstantAggregateZero, ConstantStruct.
- CN: 这一段延续了 CtorUtils 的主体实现，围绕 Ignore, ConstantArray, ConstantAggregateZero, ConstantStruct 等局部辅助逻辑展开。

### Lines 100-115

```cpp
    // Can only handle global constructors with no arguments.
    Function *F = dyn_cast<Function>(CS->getOperand(1));
    if (!F || F->arg_size() != 0)
      return nullptr;
  }
  return GV;
}

/// Call "ShouldRemove" for every entry in M's global_ctor list and remove the
/// entries for which it returns true.  Return true if anything changed.
bool llvm::optimizeGlobalCtorsList(
    Module &M, function_ref<bool(uint32_t, Function *)> ShouldRemove) {
  GlobalVariable *GlobalCtors = findGlobalCtors(M);
  if (!GlobalCtors)
    return false;

```
- EN: This region continues the CtorUtils implementation with local helper logic centered on Can, Function, Call, ShouldRemove.
- CN: 这一段延续了 CtorUtils 的主体实现，围绕 Can, Function, Call, ShouldRemove 等局部辅助逻辑展开。

### Lines 116-134

```cpp
  std::vector<std::pair<uint32_t, Function *>> Ctors =
      parseGlobalCtors(GlobalCtors);
  if (Ctors.empty())
    return false;

  bool MadeChange = false;
  // Loop over global ctors, optimizing them when we can.
  BitVector CtorsToRemove(Ctors.size());
  std::vector<size_t> CtorsByPriority(Ctors.size());
  std::iota(CtorsByPriority.begin(), CtorsByPriority.end(), 0);
  stable_sort(CtorsByPriority, [&](size_t LHS, size_t RHS) {
    return Ctors[LHS].first < Ctors[RHS].first;
  });
  for (unsigned CtorIndex : CtorsByPriority) {
    const uint32_t Priority = Ctors[CtorIndex].first;
    Function *F = Ctors[CtorIndex].second;
    if (!F)
      continue;

```
- EN: Core entities appearing here include stable_sort, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 stable_sort，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 135-148

```cpp
    LLVM_DEBUG(dbgs() << "Optimizing Global Constructor: " << *F << "\n");

    // If we can evaluate the ctor at compile time, do.
    if (ShouldRemove(Priority, F)) {
      Ctors[CtorIndex].second = nullptr;
      CtorsToRemove.set(CtorIndex);
      MadeChange = true;
      continue;
    }
  }

  if (!MadeChange)
    return false;

```
- EN: This region continues the CtorUtils implementation with local helper logic centered on LLVM_DEBUG, Optimizing, Global, Constructor.
- CN: 这一段延续了 CtorUtils 的主体实现，围绕 LLVM_DEBUG, Optimizing, Global, Constructor 等局部辅助逻辑展开。

### Lines 149-151

```cpp
  removeGlobalCtors(GlobalCtors, CtorsToRemove);
  return true;
}
```
- EN: This region continues the CtorUtils implementation with local helper logic centered on GlobalCtors, CtorsToRemove.
- CN: 这一段延续了 CtorUtils 的主体实现，围绕 GlobalCtors, CtorsToRemove 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `removeGlobalCtors, parseGlobalCtors, stable_sort` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`removeGlobalCtors, parseGlobalCtors, stable_sort` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/CtorUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/CtorUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/BitVector.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/BitVector.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `numeric` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`numeric` 为 LLVM API 之外的 C++ 语言工具提供支持。
