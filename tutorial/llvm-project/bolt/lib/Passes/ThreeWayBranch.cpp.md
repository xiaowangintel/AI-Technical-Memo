# ThreeWayBranch.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/ThreeWayBranch.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/ThreeWayBranch.cpp This file implements the ThreeWayBranch class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/ThreeWayBranch.cpp This file implements the ThreeWayBranch class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/ThreeWayBranch.cpp -------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the ThreeWayBranch class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-28

```cpp
#include "bolt/Passes/ThreeWayBranch.h"

using namespace llvm;

namespace llvm {
namespace bolt {

bool ThreeWayBranch::shouldRunOnFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  for (const BinaryBasicBlock &BB : Function)
    for (const MCInst &Inst : BB)
      if (BC.MIB->isPacked(Inst))
        return false;
  return true;
}
```

- EN: Pulls in 1 header(s) from local project dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `shouldRunOnFunction`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 1 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `shouldRunOnFunction`。这里的注释记录了设计假设、不变量或使用说明。

### Lines 29-46

```cpp
void ThreeWayBranch::runOnFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  MCContext *Ctx = BC.Ctx.get();
  // New blocks will be added and layout will change,
  // so make a copy here to iterate over the original layout
  BinaryFunction::BasicBlockOrderType BlockLayout(
      Function.getLayout().block_begin(), Function.getLayout().block_end());
  for (BinaryBasicBlock *BB : BlockLayout) {
    // The block must be hot
    if (BB->getExecutionCount() == 0 ||
        BB->getExecutionCount() == BinaryBasicBlock::COUNT_NO_PROFILE)
      continue;
    // with two successors
    if (BB->succ_size() != 2)
      continue;
    // no jump table
    if (BB->hasJumpTable())
      continue;
```

- EN: Declares or implements routines including `runOnFunction`, `getExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunction`, `getExecutionCount`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `getExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunction`, `getExecutionCount`。

### Lines 47-56

```cpp

    BinaryBasicBlock *FalseSucc = BB->getConditionalSuccessor(false);
    BinaryBasicBlock *TrueSucc = BB->getConditionalSuccessor(true);

    // One of BB's successors must have only one instruction that is a
    // conditional jump
    if ((FalseSucc->succ_size() != 2 || FalseSucc->size() != 1) &&
        (TrueSucc->succ_size() != 2 || TrueSucc->size() != 1))
      continue;
```

- EN: Declares or implements routines including `getConditionalSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getConditionalSuccessor`.
- CN: 这里声明或实现函数，例如 `getConditionalSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getConditionalSuccessor`。

### Lines 57-64

```cpp
    // SecondBranch has the second conditional jump
    BinaryBasicBlock *SecondBranch = FalseSucc;
    BinaryBasicBlock *FirstEndpoint = TrueSucc;
    if (FalseSucc->succ_size() != 2) {
      SecondBranch = TrueSucc;
      FirstEndpoint = FalseSucc;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 65-74

```cpp
    BinaryBasicBlock *SecondEndpoint =
        SecondBranch->getConditionalSuccessor(false);
    BinaryBasicBlock *ThirdEndpoint =
        SecondBranch->getConditionalSuccessor(true);

    // Make sure we can modify the jump in SecondBranch without disturbing any
    // other paths
    if (SecondBranch->pred_size() != 1)
      continue;
```

- EN: Declares or implements routines including `getConditionalSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getConditionalSuccessor`.
- CN: 这里声明或实现函数，例如 `getConditionalSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getConditionalSuccessor`。

### Lines 75-92

```cpp
    // Get Jump Instructions
    MCInst *FirstJump = BB->getLastNonPseudoInstr();
    MCInst *SecondJump = SecondBranch->getLastNonPseudoInstr();

    // Get condition codes
    unsigned FirstCC = BC.MIB->getCondCode(*FirstJump);
    if (SecondBranch != FalseSucc)
      FirstCC = BC.MIB->getInvertedCondCode(FirstCC);
    // ThirdCC = ThirdCond && !FirstCC = !(!ThirdCond ||
    // !(!FirstCC)) = !(!ThirdCond || FirstCC)
    unsigned ThirdCC =
        BC.MIB->getInvertedCondCode(BC.MIB->getCondCodesLogicalOr(
            BC.MIB->getInvertedCondCode(BC.MIB->getCondCode(*SecondJump)),
            FirstCC));
    // SecondCC = !ThirdCond && !FirstCC = !(!(!ThirdCond) ||
    // !(!FirstCC)) = !(ThirdCond || FirstCC)
    unsigned SecondCC =
        BC.MIB->getInvertedCondCode(BC.MIB->getCondCodesLogicalOr(
```

- EN: Declares or implements routines including `getLastNonPseudoInstr`, `getCondCode`, `getInvertedCondCode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLastNonPseudoInstr`, `getCondCode`, `getInvertedCondCode`.
- CN: 这里声明或实现函数，例如 `getLastNonPseudoInstr`, `getCondCode`, `getInvertedCondCode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLastNonPseudoInstr`, `getCondCode`, `getInvertedCondCode`。

### Lines 93-103

```cpp
            BC.MIB->getCondCode(*SecondJump), FirstCC));

    if (!BC.MIB->isValidCondCode(FirstCC) ||
        !BC.MIB->isValidCondCode(ThirdCC) || !BC.MIB->isValidCondCode(SecondCC))
      continue;

    std::vector<std::pair<BinaryBasicBlock *, unsigned>> Blocks;
    Blocks.push_back(std::make_pair(FirstEndpoint, FirstCC));
    Blocks.push_back(std::make_pair(SecondEndpoint, SecondCC));
    Blocks.push_back(std::make_pair(ThirdEndpoint, ThirdCC));
```

- EN: Declares or implements routines including `getCondCode`, `isValidCondCode`. Notable symbols here include `getCondCode`, `isValidCondCode`.
- CN: 这里声明或实现函数，例如 `getCondCode`, `isValidCondCode`。这里较值得关注的符号包括 `getCondCode`, `isValidCondCode`。

### Lines 104-113

```cpp
    llvm::sort(Blocks, [&](const std::pair<BinaryBasicBlock *, unsigned> A,
                           const std::pair<BinaryBasicBlock *, unsigned> B) {
      return A.first->getExecutionCount() < B.first->getExecutionCount();
    });

    uint64_t NewSecondBranchCount = Blocks[1].first->getExecutionCount() +
                                    Blocks[0].first->getExecutionCount();
    bool SecondBranchBigger =
        NewSecondBranchCount > Blocks[2].first->getExecutionCount();
```

- EN: Declares or implements routines including `getExecutionCount`. Notable symbols here include `getExecutionCount`.
- CN: 这里声明或实现函数，例如 `getExecutionCount`。这里较值得关注的符号包括 `getExecutionCount`。

### Lines 114-122

```cpp
    BB->removeAllSuccessors();
    if (SecondBranchBigger) {
      BB->addSuccessor(Blocks[2].first, Blocks[2].first->getExecutionCount());
      BB->addSuccessor(SecondBranch, NewSecondBranchCount);
    } else {
      BB->addSuccessor(SecondBranch, NewSecondBranchCount);
      BB->addSuccessor(Blocks[2].first, Blocks[2].first->getExecutionCount());
    }
```

- EN: Declares or implements routines including `removeAllSuccessors`, `addSuccessor`. Notable symbols here include `removeAllSuccessors`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `removeAllSuccessors`, `addSuccessor`。这里较值得关注的符号包括 `removeAllSuccessors`, `addSuccessor`。

### Lines 123-131

```cpp
    // Remove and add so there is no duplicate successors
    SecondBranch->removeAllSuccessors();
    SecondBranch->addSuccessor(Blocks[0].first,
                               Blocks[0].first->getExecutionCount());
    SecondBranch->addSuccessor(Blocks[1].first,
                               Blocks[1].first->getExecutionCount());

    SecondBranch->setExecutionCount(NewSecondBranchCount);
```

- EN: Declares or implements routines including `removeAllSuccessors`, `getExecutionCount`, `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `removeAllSuccessors`, `getExecutionCount`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `removeAllSuccessors`, `getExecutionCount`, `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `removeAllSuccessors`, `getExecutionCount`, `setExecutionCount`。

### Lines 132-140

```cpp
    // Replace the branch condition to fallthrough for the most common block
    if (SecondBranchBigger)
      BC.MIB->replaceBranchCondition(*FirstJump, Blocks[2].first->getLabel(),
                                     Ctx, Blocks[2].second);
    else
      BC.MIB->replaceBranchCondition(
          *FirstJump, SecondBranch->getLabel(), Ctx,
          BC.MIB->getInvertedCondCode(Blocks[2].second));
```

- EN: Declares or implements routines including `replaceBranchCondition`, `getInvertedCondCode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `replaceBranchCondition`, `getInvertedCondCode`.
- CN: 这里声明或实现函数，例如 `replaceBranchCondition`, `getInvertedCondCode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `replaceBranchCondition`, `getInvertedCondCode`。

### Lines 141-149

```cpp
    // Replace the branch condition to fallthrough for the second most common
    // block
    BC.MIB->replaceBranchCondition(*SecondJump, Blocks[0].first->getLabel(),
                                   Ctx, Blocks[0].second);

    ++BranchesAltered;
  }
}
```

- EN: Declares or implements routines including `replaceBranchCondition`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `replaceBranchCondition`.
- CN: 这里声明或实现函数，例如 `replaceBranchCondition`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `replaceBranchCondition`。

### Lines 150-162

```cpp
Error ThreeWayBranch::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is supported only on X86\n";
    exit(1);
  }

  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    if (!shouldRunOnFunction(Function))
      continue;
    runOnFunction(Function);
  }
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `exit`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `exit`, `runOnFunction`。

### Lines 163-169

```cpp
  BC.outs() << "BOLT-INFO: number of three way branches order changed: "
            << BranchesAltered << "\n";
  return Error::success();
}

} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `shouldRunOnFunction`: function or method entry point / 函数或方法入口
- `runOnFunction`: function or method entry point / 函数或方法入口
- `getExecutionCount`: function or method entry point / 函数或方法入口
- `getConditionalSuccessor`: function or method entry point / 函数或方法入口
- `getLastNonPseudoInstr`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/ThreeWayBranch.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
