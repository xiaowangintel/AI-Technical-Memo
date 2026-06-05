# CallGraphUpdater.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CallGraphUpdater.cpp` | `llvm/lib/Transforms/Utils/CallGraphUpdater.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements a (lazy) call graph update helper within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CallGraphUpdater 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- CallGraphUpdater.cpp - A (lazy) call graph update helper -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// This file provides interfaces used to manipulate a call graph, regardless
/// if it is a "old style" CallGraph or an "new style" LazyCallGraph.
///
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/CallGraphUpdater.h"
#include "llvm/IR/Constants.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 19-33

```cpp
using namespace llvm;

bool CallGraphUpdater::finalize() {
  if (!DeadFunctionsInComdats.empty()) {
    filterDeadComdatFunctions(DeadFunctionsInComdats);
    DeadFunctions.append(DeadFunctionsInComdats.begin(),
                         DeadFunctionsInComdats.end());
  }

  // This is the code path for the new lazy call graph and for the case were
  // no call graph was provided.
  for (Function *DeadFn : DeadFunctions) {
    DeadFn->removeDeadConstantUsers();
    // If the function is used by metadata, we don't want it to be replaced with
    // poison in the metadata, so we replace it with nullptr in the metadata
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include finalize, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 finalize，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 34-49

```cpp
    // before RAUW'ing the non-metadata uses below.
    if (DeadFn->isUsedByMetadata())
      ValueAsMetadata::handleDeletion(DeadFn);
    DeadFn->replaceNonMetadataUsesWith(PoisonValue::get(DeadFn->getType()));

    if (LCG && !ReplacedFunctions.count(DeadFn)) {
      // Taken mostly from the inliner:
      LazyCallGraph::Node &N = LCG->get(*DeadFn);
      auto *DeadSCC = LCG->lookupSCC(N);
      assert(DeadSCC && DeadSCC->size() == 1 &&
             &DeadSCC->begin()->getFunction() == DeadFn);

      FAM->clear(*DeadFn, DeadFn->getName());
      AM->clear(*DeadSCC, DeadSCC->getName());
      LCG->markDeadFunction(*DeadFn);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 50-68

```cpp
      // Mark the relevant parts of the call graph as invalid so we don't
      // visit them.
      UR->InvalidatedSCCs.insert(LCG->lookupSCC(N));
      UR->DeadFunctions.push_back(DeadFn);
    } else {
      // The CGSCC infrastructure batch deletes functions at the end of the
      // call graph walk, so only erase the function if we're not using that
      // infrastructure.
      // The function is now really dead and de-attached from everything.
      DeadFn->eraseFromParent();
    }
  }

  bool Changed = !DeadFunctions.empty();
  DeadFunctionsInComdats.clear();
  DeadFunctions.clear();
  return Changed;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 69-82

```cpp
void CallGraphUpdater::reanalyzeFunction(Function &Fn) {
  if (LCG) {
    LazyCallGraph::Node &N = LCG->get(Fn);
    LazyCallGraph::SCC *C = LCG->lookupSCC(N);
    updateCGAndAnalysisManagerForCGSCCPass(*LCG, *C, N, *AM, *UR, *FAM);
  }
}

void CallGraphUpdater::registerOutlinedFunction(Function &OriginalFn,
                                                Function &NewFn) {
  if (LCG)
    LCG->addSplitFunction(OriginalFn, NewFn);
}

```
- EN: Core entities appearing here include reanalyzeFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 此处出现的核心实体包括 reanalyzeFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 83-94

```cpp
void CallGraphUpdater::removeFunction(Function &DeadFn) {
  DeadFn.deleteBody();
  DeadFn.setLinkage(GlobalValue::ExternalLinkage);
  if (DeadFn.hasComdat())
    DeadFunctionsInComdats.push_back(&DeadFn);
  else
    DeadFunctions.push_back(&DeadFn);

  if (FAM)
    FAM->clear(DeadFn, DeadFn.getName());
}

```
- EN: Core entities appearing here include removeFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 此处出现的核心实体包括 removeFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 95-104

```cpp
void CallGraphUpdater::replaceFunctionWith(Function &OldFn, Function &NewFn) {
  OldFn.removeDeadConstantUsers();
  ReplacedFunctions.insert(&OldFn);
  if (LCG) {
    // Directly substitute the functions in the call graph.
    LazyCallGraph::Node &OldLCGN = LCG->get(OldFn);
    SCC->getOuterRefSCC().replaceNodeFunction(OldLCGN, NewFn);
  }
  removeFunction(OldFn);
}
```
- EN: Core entities appearing here include replaceFunctionWith, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 此处出现的核心实体包括 replaceFunctionWith，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `finalize, reanalyzeFunction, removeFunction, replaceFunctionWith` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`finalize, reanalyzeFunction, removeFunction, replaceFunctionWith` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `CallGraph` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `CallGraph` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/Transforms/Utils/CallGraphUpdater.h`, `llvm/Transforms/Utils/ModuleUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/Transforms/Utils/CallGraphUpdater.h`, `llvm/Transforms/Utils/ModuleUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `CallGraph` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`CallGraph` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
