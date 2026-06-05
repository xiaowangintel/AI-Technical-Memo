# LoopInversionPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/LoopInversionPass.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/LoopInversionPass.cpp This file implements the LoopInversionPass class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/LoopInversionPass.cpp This file implements the LoopInversionPass class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/LoopInversionPass.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the LoopInversionPass class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Passes/LoopInversionPass.h"
#include "bolt/Core/ParallelUtilities.h"

using namespace llvm;

namespace opts {
extern cl::OptionCategory BoltCategory;
```

- EN: Pulls in 2 header(s) from local project dependencies needed by this range. Works inside namespace scope `llvm`, `opts` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `opts`.
- CN: 这里引入 2 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `opts`。

### Lines 21-28

```cpp
extern cl::opt<bolt::ReorderBasicBlocks::LayoutType> ReorderBlocks;

static cl::opt<bool> LoopReorder(
    "loop-inversion-opt",
    cl::desc("reorder unconditional jump instructions in loops optimization"),
    cl::init(true), cl::cat(BoltCategory), cl::ReallyHidden);
} // namespace opts
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`, `opts`。

### Lines 29-36

```cpp
namespace llvm {
namespace bolt {

bool LoopInversionPass::runOnFunction(BinaryFunction &BF) {
  bool IsChanged = false;
  if (BF.getLayout().block_size() < 3 || !BF.hasValidProfile())
    return false;
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `runOnFunction`. Notable symbols here include `runOnFunction`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `runOnFunction`。这里较值得关注的符号包括 `runOnFunction`, `llvm`, `bolt`。

### Lines 37-54

```cpp
  BF.getLayout().updateLayoutIndices();
  for (BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    if (BB->succ_size() != 1 || BB->pred_size() != 1)
      continue;

    BinaryBasicBlock *SuccBB = *BB->succ_begin();
    BinaryBasicBlock *PredBB = *BB->pred_begin();
    const unsigned BBIndex = BB->getLayoutIndex();
    const unsigned SuccBBIndex = SuccBB->getLayoutIndex();
    if (SuccBB == PredBB && BB != SuccBB && BBIndex != 0 && SuccBBIndex != 0 &&
        SuccBB->succ_size() == 2 &&
        BB->getFragmentNum() == SuccBB->getFragmentNum()) {
      // Get the second successor (after loop BB)
      BinaryBasicBlock *SecondSucc = nullptr;
      for (BinaryBasicBlock *Succ : SuccBB->successors()) {
        if (Succ != &*BB) {
          SecondSucc = Succ;
          break;
```

- EN: Declares or implements routines including `succ_begin`, `pred_begin`, `getLayoutIndex`, `succ_size`, `getFragmentNum`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `succ_begin`, `pred_begin`, `getLayoutIndex`, `succ_size`, `getFragmentNum`.
- CN: 这里声明或实现函数，例如 `succ_begin`, `pred_begin`, `getLayoutIndex`, `succ_size`, `getFragmentNum`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `succ_begin`, `pred_begin`, `getLayoutIndex`, `succ_size`, `getFragmentNum`。

### Lines 55-68

```cpp
        }
      }

      assert(SecondSucc != nullptr && "Unable to find a second BB successor");
      const uint64_t LoopCount = SuccBB->getBranchInfo(*BB).Count;
      const uint64_t ExitCount = SuccBB->getBranchInfo(*SecondSucc).Count;

      if (LoopCount < ExitCount) {
        if (BBIndex > SuccBBIndex)
          continue;
      } else if (BBIndex < SuccBBIndex) {
        continue;
      }
```

- EN: Declares or implements routines including `assert`, `getBranchInfo`, `if`. Notable symbols here include `assert`, `getBranchInfo`, `if`.
- CN: 这里声明或实现函数，例如 `assert`, `getBranchInfo`, `if`。这里较值得关注的符号包括 `assert`, `getBranchInfo`, `if`。

### Lines 69-83

```cpp
      IsChanged = true;
      BB->setLayoutIndex(SuccBBIndex);
      SuccBB->setLayoutIndex(BBIndex);
    }
  }

  if (IsChanged) {
    BinaryFunction::BasicBlockOrderType NewOrder(BF.getLayout().block_begin(),
                                                 BF.getLayout().block_end());
    llvm::sort(NewOrder, [&](BinaryBasicBlock *BB1, BinaryBasicBlock *BB2) {
      return BB1->getLayoutIndex() < BB2->getLayoutIndex();
    });
    BF.getLayout().update(NewOrder);
  }
```

- EN: Declares or implements routines including `setLayoutIndex`, `NewOrder`, `sort`. Notable symbols here include `setLayoutIndex`, `NewOrder`, `sort`.
- CN: 这里声明或实现函数，例如 `setLayoutIndex`, `NewOrder`, `sort`。这里较值得关注的符号包括 `setLayoutIndex`, `NewOrder`, `sort`。

### Lines 84-92

```cpp
  return IsChanged;
}

Error LoopInversionPass::runOnFunctions(BinaryContext &BC) {
  std::atomic<uint64_t> ModifiedFuncCount{0};
  if (opts::ReorderBlocks == ReorderBasicBlocks::LT_NONE ||
      opts::LoopReorder == false)
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 93-101

```cpp
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    if (runOnFunction(BF))
      ++ModifiedFuncCount;
  };

  ParallelUtilities::PredicateTy SkipFunc = [&](const BinaryFunction &BF) {
    return !shouldOptimize(BF);
  };
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 102-110

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_TRIVIAL, WorkFun, SkipFunc,
      "LoopInversionPass");

  BC.outs() << "BOLT-INFO: " << ModifiedFuncCount
            << " Functions were reordered by LoopInversionPass\n";
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 111-112

```cpp
} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `runOnFunction`: function or method entry point / 函数或方法入口
- `succ_begin`: function or method entry point / 函数或方法入口
- `pred_begin`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/LoopInversionPass.h`, `bolt/Core/ParallelUtilities.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
