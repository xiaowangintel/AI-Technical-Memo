# GuardUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/GuardUtils.cpp` | `llvm/lib/Transforms/Utils/GuardUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements utils for work with guards within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 GuardUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

```cpp
//===-- GuardUtils.cpp - Utils for work with guards -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// Utils that are used to perform transformations related to guards and their
// conditions.
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 12-24

```cpp
#include "llvm/Transforms/Utils/GuardUtils.h"
#include "llvm/Analysis/GuardUtils.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

using namespace llvm;
using namespace llvm::PatternMatch;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 25-38

```cpp
static cl::opt<uint32_t> PredicatePassBranchWeight(
    "guards-predicate-pass-branch-weight", cl::Hidden, cl::init(1 << 20),
    cl::desc("The probability of a guard failing is assumed to be the "
             "reciprocal of this value (default = 1 << 20)"));

void llvm::makeGuardControlFlowExplicit(Function *DeoptIntrinsic,
                                        CallInst *Guard, bool UseWC) {
  OperandBundleDef DeoptOB(*Guard->getOperandBundle(LLVMContext::OB_deopt));
  SmallVector<Value *, 4> Args(drop_begin(Guard->args()));

  auto *CheckBB = Guard->getParent();
  auto *DeoptBlockTerm =
      SplitBlockAndInsertIfThen(Guard->getArgOperand(0), Guard, true);

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 39-54

```cpp
  auto *CheckBI = cast<CondBrInst>(CheckBB->getTerminator());

  // SplitBlockAndInsertIfThen inserts control flow that branches to
  // DeoptBlockTerm if the condition is true.  We want the opposite.
  CheckBI->swapSuccessors();

  CheckBI->getSuccessor(0)->setName("guarded");
  CheckBI->getSuccessor(1)->setName("deopt");

  if (auto *MD = Guard->getMetadata(LLVMContext::MD_make_implicit))
    CheckBI->setMetadata(LLVMContext::MD_make_implicit, MD);

  MDBuilder MDB(Guard->getContext());
  CheckBI->setMetadata(LLVMContext::MD_prof,
                       MDB.createBranchWeights(PredicatePassBranchWeight, 1));

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 55-67

```cpp
  IRBuilder<> B(DeoptBlockTerm);
  auto *DeoptCall = B.CreateCall(DeoptIntrinsic, Args, {DeoptOB}, "");

  if (DeoptIntrinsic->getReturnType()->isVoidTy()) {
    B.CreateRetVoid();
  } else {
    DeoptCall->setName("deoptcall");
    B.CreateRet(DeoptCall);
  }

  DeoptCall->setCallingConv(Guard->getCallingConv());
  DeoptBlockTerm->eraseFromParent();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 68-83

```cpp
  if (UseWC) {
    // We want the guard to be expressed as explicit control flow, but still be
    // widenable. For that, we add Widenable Condition intrinsic call to the
    // guard's condition.
    IRBuilder<> B(CheckBI);
    auto *WC = B.CreateIntrinsic(Intrinsic::experimental_widenable_condition,
                                 {}, nullptr, "widenable_cond");
    CheckBI->setCondition(B.CreateAnd(CheckBI->getCondition(), WC,
                                      "exiplicit_guard_cond"));
    assert(isWidenableBranch(CheckBI) && "Branch must be widenable.");
  }
}

void llvm::widenWidenableBranch(CondBrInst *WidenableBR, Value *NewCond) {
  assert(isWidenableBranch(WidenableBR) && "precondition");

```
- EN: Core entities appearing here include widenWidenableBranch, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 widenWidenableBranch，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 84-96

```cpp
  // The tempting trivially option is to produce something like this:
  // br (and oldcond, newcond) where oldcond is assumed to contain a widenable
  // condition, but that doesn't match the pattern parseWidenableBranch expects
  // so we have to be more sophisticated.

  Use *C, *WC;
  BasicBlock *IfTrueBB, *IfFalseBB;
  parseWidenableBranch(WidenableBR, C, WC, IfTrueBB, IfFalseBB);
  if (!C) {
    // br (wc()), ... form
    IRBuilder<> B(WidenableBR);
    WidenableBR->setCondition(B.CreateAnd(NewCond, WC->get()));
  } else {
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 97-109

```cpp
    // br (wc & C), ... form
    IRBuilder<> B(WidenableBR);
    C->set(B.CreateAnd(NewCond, C->get()));
    Instruction *WCAnd = cast<Instruction>(WidenableBR->getCondition());
    // Condition is only guaranteed to dominate branch
    WCAnd->moveBefore(WidenableBR->getIterator());
  }
  assert(isWidenableBranch(WidenableBR) && "preserve widenabiliy");
}

void llvm::setWidenableBranchCond(CondBrInst *WidenableBR, Value *NewCond) {
  assert(isWidenableBranch(WidenableBR) && "precondition");

```
- EN: Core entities appearing here include setWidenableBranchCond, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 setWidenableBranchCond，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 110-122

```cpp
  Use *C, *WC;
  BasicBlock *IfTrueBB, *IfFalseBB;
  parseWidenableBranch(WidenableBR, C, WC, IfTrueBB, IfFalseBB);
  if (!C) {
    // br (wc()), ... form
    IRBuilder<> B(WidenableBR);
    WidenableBR->setCondition(B.CreateAnd(NewCond, WC->get()));
  } else {
    // br (wc & C), ... form
    Instruction *WCAnd = cast<Instruction>(WidenableBR->getCondition());
    // Condition is only guaranteed to dominate branch
    WCAnd->moveBefore(WidenableBR->getIterator());
    C->set(NewCond);
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 123-125

```cpp
  }
  assert(isWidenableBranch(WidenableBR) && "preserve widenabiliy");
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `widenWidenableBranch, setWidenableBranchCond` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`widenWidenableBranch, setWidenableBranchCond` 构成该文件对外 API 与主要实现挂钩。
- EN: Tuning surface: command-line knobs such as `PredicatePassBranchWeight` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `PredicatePassBranchWeight` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/GuardUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/GuardUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/GuardUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/GuardUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
