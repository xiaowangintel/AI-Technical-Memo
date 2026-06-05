# LoadStoreVec.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Passes/LoadStoreVec.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Passes/LoadStoreVec.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vectorizer pass short load-store chains within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 LoadStoreVec 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-19

```cpp
//===- LoadStoreVec.cpp - Vectorizer pass short load-store chains ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/Passes/LoadStoreVec.h"
#include "llvm/SandboxIR/Module.h"
#include "llvm/SandboxIR/Region.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/Legality.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h"

namespace llvm {

namespace sandboxir {

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 20-37

```cpp
std::optional<Type *> LoadStoreVec::canVectorize(ArrayRef<Instruction *> Bndl,
                                                 Scheduler &Sched) {
  // Check if in the same BB.
  if (LegalityAnalysis::differentBlock(Bndl))
    return std::nullopt;

  // Check if instructions repeat.
  if (!LegalityAnalysis::areUnique(Bndl))
    return std::nullopt;

  // TODO: This is target-dependent.
  // Don't mix integer with floating point.
  bool IsFloat = false;
  bool IsInteger = false;
  for ([[maybe_unused]] auto *I : Bndl) {
    if (Utils::getExpectedType(I)->getScalarType()->isFloatingPointTy())
      IsFloat = true;
    else
```
- EN: This region continues the LoadStoreVec implementation with local helper logic centered on Type, LoadStoreVec, ArrayRef, Instruction.
- CN: 这一段延续了 LoadStoreVec 的主体实现，围绕 Type, LoadStoreVec, ArrayRef, Instruction 等局部辅助逻辑展开。

### Lines 38-55

```cpp
      IsInteger = true;
  }
  if (IsFloat && IsInteger)
    return std::nullopt;

  // Check scheduling.
  if (!Sched.trySchedule(Bndl))
    return std::nullopt;

  return VecUtils::getCombinedVectorTypeFor(Bndl, *DL);
}

void LoadStoreVec::tryEraseDeadInstrs(ArrayRef<Instruction *> Stores,
                                      ArrayRef<Value *> Operands) {
  SmallPtrSet<Instruction *, 8> DeadCandidates;
  for (auto *SI : Stores) {
    if (auto *PtrI =
            dyn_cast<Instruction>(cast<StoreInst>(SI)->getPointerOperand()))
```
- EN: This region continues the LoadStoreVec implementation with local helper logic centered on IsInteger, IsFloat, Check, Sched.
- CN: 这一段延续了 LoadStoreVec 的主体实现，围绕 IsInteger, IsFloat, Check, Sched 等局部辅助逻辑展开。

### Lines 56-71

```cpp
      DeadCandidates.insert(PtrI);
    SI->eraseFromParent();
  }
  for (auto *Op : Operands) {
    if (auto *LI = dyn_cast<LoadInst>(Op)) {
      if (auto *PtrI =
              dyn_cast<Instruction>(cast<LoadInst>(LI)->getPointerOperand()))
        DeadCandidates.insert(PtrI);
      cast<LoadInst>(LI)->eraseFromParent();
    }
  }
  for (auto *PtrI : DeadCandidates)
    if (!PtrI->hasNUsesOrMore(1))
      PtrI->eraseFromParent();
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 72-89

```cpp
bool LoadStoreVec::runOnRegion(Region &Rgn, const Analyses &A) {
  SmallVector<Instruction *, 8> Bndl(Rgn.getAux().begin(), Rgn.getAux().end());
  if (Bndl.size() < 2)
    return false;
  Function &F = *Bndl[0]->getParent()->getParent();
  DL = &F.getParent()->getDataLayout();
  auto &Ctx = F.getContext();
  Scheduler Sched(A.getAA(), Ctx);
  if (!VecUtils::areConsecutive<StoreInst, Instruction>(
          Bndl, A.getScalarEvolution(), *DL))
    return false;
  if (!canVectorize(Bndl, Sched))
    return false;

  SmallVector<Value *, 4> Operands;
  Operands.reserve(Bndl.size());
  for (auto *I : Bndl) {
    auto *Op = cast<StoreInst>(I)->getValueOperand();
```
- EN: Core entities appearing here include runOnRegion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution.
- CN: 此处出现的核心实体包括 runOnRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution 等分析结果。

### Lines 90-111

```cpp
    Operands.push_back(Op);
  }
  BasicBlock *BB = Bndl[0]->getParent();
  // TODO: For now we only support load operands.
  // TODO: For now we don't cross BBs.
  // TODO: For now don't vectorize if the loads have external uses.
  bool AllLoads = all_of(Operands, [BB](Value *V) {
    auto *LI = dyn_cast<LoadInst>(V);
    if (LI == nullptr)
      return false;
    // TODO: For now we don't cross BBs.
    if (LI->getParent() != BB)
      return false;
    if (LI->hasNUsesOrMore(2))
      return false;
    return true;
  });
  bool AllConstants =
      all_of(Operands, [](Value *V) { return isa<Constant>(V); });
  if (!AllLoads && !AllConstants)
    return false;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 112-126

```cpp
  Value *VecOp = nullptr;
  if (AllLoads) {
    // TODO: Try to avoid the extra copy to an instruction vector.
    SmallVector<Instruction *, 8> Loads;
    Loads.reserve(Operands.size());
    for (Value *Op : Operands)
      Loads.push_back(cast<Instruction>(Op));

    bool Consecutive = VecUtils::areConsecutive<LoadInst, Instruction>(
        Loads, A.getScalarEvolution(), *DL);
    if (!Consecutive)
      return false;
    if (!canVectorize(Loads, Sched))
      return false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution 等分析结果。

### Lines 127-144

```cpp
    // Generate vector load.
    Type *Ty = VecUtils::getCombinedVectorTypeFor(Bndl, *DL);
    Value *LdPtr = cast<LoadInst>(Loads[0])->getPointerOperand();
    // TODO: Compute alignment.
    Align LdAlign(1);
    auto LdWhereIt = std::next(VecUtils::getLowest(Loads)->getIterator());
    VecOp = LoadInst::create(Ty, LdPtr, LdAlign, LdWhereIt, Ctx, "VecIinitL");
  } else if (AllConstants) {
    SmallVector<Constant *, 8> Constants;
    Constants.reserve(Operands.size());
    for (Value *Op : Operands) {
      auto *COp = cast<Constant>(Op);
      if (auto *AggrCOp = dyn_cast<ConstantAggregate>(COp)) {
        // If the operand is a constant aggregate, then append all its elements.
        for (Value *Elm : AggrCOp->operands())
          Constants.push_back(cast<Constant>(Elm));
      } else if (auto *SeqCOp = dyn_cast<ConstantDataSequential>(COp)) {
        for (auto ElmIdx : seq<unsigned>(SeqCOp->getNumElements()))
```
- EN: This region continues the LoadStoreVec implementation with local helper logic centered on Generate, Type, VecUtils, Bndl.
- CN: 这一段延续了 LoadStoreVec 的主体实现，围绕 Generate, Type, VecUtils, Bndl 等局部辅助逻辑展开。

### Lines 145-164

```cpp
          Constants.push_back(SeqCOp->getElementAsConstant(ElmIdx));
      } else if (auto *Zero = dyn_cast<ConstantAggregateZero>(COp)) {
        auto *ZeroElm = Zero->getSequentialElement();
        for ([[maybe_unused]] auto Cnt :
             seq<unsigned>(Zero->getElementCount().getFixedValue()))
          Constants.push_back(ZeroElm);
      } else {
        Constants.push_back(COp);
      }
    }
    VecOp = ConstantVector::get(Constants);
  }

  // Generate vector store.
  Value *StPtr = cast<StoreInst>(Bndl[0])->getPointerOperand();
  // TODO: Compute alignment.
  Align StAlign(1);
  auto StWhereIt = std::next(VecUtils::getLowest(Bndl)->getIterator());
  StoreInst::create(VecOp, StPtr, StAlign, StWhereIt, Ctx);

```
- EN: This region continues the LoadStoreVec implementation with local helper logic centered on Constants, SeqCOp, ElmIdx, Zero.
- CN: 这一段延续了 LoadStoreVec 的主体实现，围绕 Constants, SeqCOp, ElmIdx, Zero 等局部辅助逻辑展开。

### Lines 165-171

```cpp
  tryEraseDeadInstrs(Bndl, Operands);
  return true;
}

} // namespace sandboxir

} // namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `runOnRegion` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`runOnRegion` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, ScalarEvolution` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, ScalarEvolution` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Vectorize/SandboxVectorizer/Passes/LoadStoreVec.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Legality.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Vectorize/SandboxVectorizer/Passes/LoadStoreVec.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Legality.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/SandboxIR/Module.h`, `llvm/SandboxIR/Region.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/SandboxIR/Module.h`, `llvm/SandboxIR/Region.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DataLayout`, `ScalarEvolution` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `ScalarEvolution` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
