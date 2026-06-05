# DemoteRegToStack.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/DemoteRegToStack.cpp` | `llvm/lib/Transforms/Utils/DemoteRegToStack.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements move a virtual register to the stack within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 DemoteRegToStack 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- DemoteRegToStack.cpp - Move a virtual register to the stack --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/DenseMap.h"
#include "llvm/Analysis/CFG.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Local.h"
using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 18-36

```cpp
/// DemoteRegToStack - This function takes a virtual register computed by an
/// Instruction and replaces it with a slot in the stack frame, allocated via
/// alloca.  This allows the CFG to be changed around without fear of
/// invalidating the SSA information for the value.  It returns the pointer to
/// the alloca inserted to create a stack slot for I.
AllocaInst *llvm::DemoteRegToStack(Instruction &I, bool VolatileLoads,
                                   std::optional<BasicBlock::iterator> AllocaPoint) {
  if (I.use_empty()) {
    I.eraseFromParent();
    return nullptr;
  }

  Function *F = I.getParent()->getParent();
  const DataLayout &DL = F->getDataLayout();

  // Create a stack slot to hold the value.
  AllocaInst *Slot;
  if (AllocaPoint) {
    Slot = new AllocaInst(I.getType(), DL.getAllocaAddrSpace(), nullptr,
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 37-55

```cpp
                          I.getName()+".reg2mem", *AllocaPoint);
  } else {
    Slot = new AllocaInst(I.getType(), DL.getAllocaAddrSpace(), nullptr,
                          I.getName() + ".reg2mem", F->getEntryBlock().begin());
  }

  // We cannot demote invoke instructions to the stack if their normal edge
  // is critical. Therefore, split the critical edge and create a basic block
  // into which the store can be inserted.
  if (InvokeInst *II = dyn_cast<InvokeInst>(&I)) {
    if (!II->getNormalDest()->getSinglePredecessor()) {
      unsigned SuccNum = GetSuccessorNumber(II->getParent(), II->getNormalDest());
      assert(isCriticalEdge(II, SuccNum) && "Expected a critical edge!");
      BasicBlock *BB = SplitCriticalEdge(II, SuccNum);
      assert(BB && "Unable to split critical edge.");
      (void)BB;
    }
  } else if (CallBrInst *CBI = dyn_cast<CallBrInst>(&I)) {
    for (unsigned i = 0; i < CBI->getNumSuccessors(); i++) {
```
- EN: This region continues the DemoteRegToStack implementation with local helper logic centered on AllocaPoint, Slot, AllocaInst, Therefore.
- CN: 这一段延续了 DemoteRegToStack 的主体实现，围绕 AllocaPoint, Slot, AllocaInst, Therefore 等局部辅助逻辑展开。

### Lines 56-74

```cpp
      auto *Succ = CBI->getSuccessor(i);
      if (!Succ->getSinglePredecessor()) {
        assert(isCriticalEdge(CBI, i) && "Expected a critical edge!");
        [[maybe_unused]] BasicBlock *BB = SplitCriticalEdge(CBI, i);
        assert(BB && "Unable to split critical edge.");
      }
    }
  }

  // Change all of the users of the instruction to read from the stack slot.
  while (!I.use_empty()) {
    Instruction *U = cast<Instruction>(I.user_back());
    if (PHINode *PN = dyn_cast<PHINode>(U)) {
      // If this is a PHI node, we can't insert a load of the value before the
      // use.  Instead insert the load in the predecessor block corresponding
      // to the incoming value.
      //
      // Note that if there are multiple edges from a basic block to this PHI
      // node that we cannot have multiple loads. The problem is that the
```
- EN: This region continues the DemoteRegToStack implementation with local helper logic centered on Succ, CBI, Expected, BasicBlock.
- CN: 这一段延续了 DemoteRegToStack 的主体实现，围绕 Succ, CBI, Expected, BasicBlock 等局部辅助逻辑展开。

### Lines 75-91

```cpp
      // resulting PHI node will have multiple values (from each load) coming in
      // from the same block, which is illegal SSA form. For this reason, we
      // keep track of and reuse loads we insert.
      DenseMap<BasicBlock*, Value*> Loads;
      for (unsigned i = 0, e = PN->getNumIncomingValues(); i != e; ++i)
        if (PN->getIncomingValue(i) == &I) {
          Value *&V = Loads[PN->getIncomingBlock(i)];
          if (!V) {
            // Insert the load into the predecessor block
            V = new LoadInst(I.getType(), Slot, I.getName() + ".reload",
                             VolatileLoads,
                             PN->getIncomingBlock(i)->getTerminator()->getIterator());
            Loads[PN->getIncomingBlock(i)] = V;
          }
          PN->setIncomingValue(i, V);
        }

```
- EN: This region continues the DemoteRegToStack implementation with local helper logic centered on PHI, SSA, For, DenseMap.
- CN: 这一段延续了 DemoteRegToStack 的主体实现，围绕 PHI, SSA, For, DenseMap 等局部辅助逻辑展开。

### Lines 92-110

```cpp
    } else {
      // If this is a normal instruction, just insert a load.
      Value *V = new LoadInst(I.getType(), Slot, I.getName() + ".reload",
                              VolatileLoads, U->getIterator());
      U->replaceUsesOfWith(&I, V);
    }
  }

  // Insert stores of the computed value into the stack slot. We have to be
  // careful if I is an invoke instruction, because we can't insert the store
  // AFTER the terminator instruction.
  BasicBlock::iterator InsertPt;
  if (!I.isTerminator()) {
    InsertPt = ++I.getIterator();
    // Don't insert before PHI nodes or landingpad instrs.
    for (; isa<PHINode>(InsertPt) || InsertPt->isEHPad(); ++InsertPt)
      if (isa<CatchSwitchInst>(InsertPt))
        break;
    if (isa<CatchSwitchInst>(InsertPt)) {
```
- EN: This region continues the DemoteRegToStack implementation with local helper logic centered on Value, LoadInst, Slot, VolatileLoads.
- CN: 这一段延续了 DemoteRegToStack 的主体实现，围绕 Value, LoadInst, Slot, VolatileLoads 等局部辅助逻辑展开。

### Lines 111-128

```cpp
      for (BasicBlock *Handler : successors(&*InsertPt))
        new StoreInst(&I, Slot, Handler->getFirstInsertionPt());
      return Slot;
    }
  } else if (InvokeInst *II = dyn_cast<InvokeInst>(&I)) {
    InsertPt = II->getNormalDest()->getFirstInsertionPt();
  } else if (CallBrInst *CBI = dyn_cast<CallBrInst>(&I)) {
    for (BasicBlock *Succ : successors(CBI))
      new StoreInst(CBI, Slot, Succ->getFirstInsertionPt());
    return Slot;
  } else {
    llvm_unreachable("Unsupported terminator for Reg2Mem");
  }

  new StoreInst(&I, Slot, InsertPt);
  return Slot;
}

```
- EN: This region continues the DemoteRegToStack implementation with local helper logic centered on BasicBlock, Handler, InsertPt, StoreInst.
- CN: 这一段延续了 DemoteRegToStack 的主体实现，围绕 BasicBlock, Handler, InsertPt, StoreInst 等局部辅助逻辑展开。

### Lines 129-151

```cpp
/// DemotePHIToStack - This function takes a virtual register computed by a PHI
/// node and replaces it with a slot in the stack frame allocated via alloca.
/// The PHI node is deleted. It returns the pointer to the alloca inserted.
AllocaInst *llvm::DemotePHIToStack(PHINode *P, std::optional<BasicBlock::iterator> AllocaPoint) {
  if (P->use_empty()) {
    P->eraseFromParent();
    return nullptr;
  }

  const DataLayout &DL = P->getDataLayout();

  // Create a stack slot to hold the value.
  AllocaInst *Slot;
  if (AllocaPoint) {
    Slot = new AllocaInst(P->getType(), DL.getAllocaAddrSpace(), nullptr,
                          P->getName()+".reg2mem", *AllocaPoint);
  } else {
    Function *F = P->getParent()->getParent();
    Slot = new AllocaInst(P->getType(), DL.getAllocaAddrSpace(), nullptr,
                          P->getName() + ".reg2mem",
                          F->getEntryBlock().begin());
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 152-170

```cpp
  // Iterate over each operand inserting a store in each predecessor.
  for (unsigned i = 0, e = P->getNumIncomingValues(); i < e; ++i) {
    if (InvokeInst *II = dyn_cast<InvokeInst>(P->getIncomingValue(i))) {
      assert(II->getParent() != P->getIncomingBlock(i) &&
             "Invoke edge not supported yet"); (void)II;
    }
    new StoreInst(P->getIncomingValue(i), Slot,
                  P->getIncomingBlock(i)->getTerminator()->getIterator());
  }

  // Insert a load in place of the PHI and replace all uses.
  BasicBlock::iterator InsertPt = P->getIterator();
  // Don't insert before PHI nodes or landingpad instrs.
  for (; isa<PHINode>(InsertPt) || InsertPt->isEHPad(); ++InsertPt)
    if (isa<CatchSwitchInst>(InsertPt))
      break;
  if (isa<CatchSwitchInst>(InsertPt)) {
    // We need a separate load before each actual use of the PHI
    SmallVector<Instruction *, 4> Users;
```
- EN: This region continues the DemoteRegToStack implementation with local helper logic centered on Iterate, InvokeInst, Invoke, StoreInst.
- CN: 这一段延续了 DemoteRegToStack 的主体实现，围绕 Iterate, InvokeInst, Invoke, StoreInst 等局部辅助逻辑展开。

### Lines 171-188

```cpp
    for (User *U : P->users()) {
      Instruction *User = cast<Instruction>(U);
      Users.push_back(User);
    }
    for (Instruction *User : Users) {
      Value *V =
          new LoadInst(P->getType(), Slot, P->getName() + ".reload", User->getIterator());
      User->replaceUsesOfWith(P, V);
    }
  } else {
    Value *V =
        new LoadInst(P->getType(), Slot, P->getName() + ".reload", InsertPt);
    P->replaceAllUsesWith(V);
  }
  // Delete PHI.
  P->eraseFromParent();
  return Slot;
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/CFG.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/CFG.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DataLayout.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DataLayout.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Local.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/DenseMap.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/DenseMap.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
