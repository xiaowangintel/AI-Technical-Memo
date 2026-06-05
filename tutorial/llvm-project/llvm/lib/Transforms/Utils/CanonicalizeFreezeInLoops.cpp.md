# CanonicalizeFreezeInLoops.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CanonicalizeFreezeInLoops.cpp` | `llvm/lib/Transforms/Utils/CanonicalizeFreezeInLoops.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements ==- CanonicalizeFreezeInLoops - Canonicalize freezes in a loop === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CanonicalizeFreezeInLoops 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
//==- CanonicalizeFreezeInLoops - Canonicalize freezes in a loop-*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass canonicalizes freeze instructions in a loop by pushing them out to
// the preheader.
//
//   loop:
//     i = phi init, i.next
//     i.next = add nsw i, 1
//     i.next.fr = freeze i.next // push this out of this loop
//     use(i.next.fr)
//     br i1 (i.next <= N), loop, exit
//   =>
//     init.fr = freeze init
//   loop:
//     i = phi init.fr, i.next
//     i.next = add i, 1         // nsw is dropped here
//     use(i.next)
//     br i1 (i.next <= N), loop, exit
//
// Removing freezes from these chains help scalar evolution successfully analyze
// expressions.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 31-58

```cpp
#include "llvm/Transforms/Utils/CanonicalizeFreezeInLoops.h"
#include "llvm/ADT/DenseMapInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/Analysis/IVDescriptors.h"
#include "llvm/Analysis/LoopAnalysisManager.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/LoopPass.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Dominators.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Utils.h"

using namespace llvm;

#define DEBUG_TYPE "canon-freeze"

namespace {

class CanonicalizeFreezeInLoops : public LoopPass {
public:
  static char ID;

  CanonicalizeFreezeInLoops();

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 59-86

```cpp
private:
  bool runOnLoop(Loop *L, LPPassManager &LPM) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;
};

class CanonicalizeFreezeInLoopsImpl {
  Loop *L;
  ScalarEvolution &SE;
  DominatorTree &DT;

  // Can freeze instruction be pushed into operands of I?
  // In order to do this, I should not create a poison after I's flags are
  // stripped.
  bool canHandleInst(const Instruction *I) {
    auto Opc = I->getOpcode();
    // If add/sub/mul, drop nsw/nuw flags.
    return Opc == Instruction::Add || Opc == Instruction::Sub ||
           Opc == Instruction::Mul;
  }

  void InsertFreezeAndForgetFromSCEV(Use &U);

public:
  CanonicalizeFreezeInLoopsImpl(Loop *L, ScalarEvolution &SE, DominatorTree &DT)
      : L(L), SE(SE), DT(DT) {}
  bool run();
};

```
- EN: Core entities appearing here include CanonicalizeFreezeInLoopsImpl, canHandleInst, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution, SCEV.
- CN: 此处出现的核心实体包括 CanonicalizeFreezeInLoopsImpl, canHandleInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution, SCEV 等分析结果。

### Lines 87-118

```cpp
struct FrozenIndPHIInfo {
  // A freeze instruction that uses an induction phi
  FreezeInst *FI = nullptr;
  // The induction phi, step instruction, the operand idx of StepInst which is
  // a step value
  PHINode *PHI;
  BinaryOperator *StepInst;
  unsigned StepValIdx = 0;

  FrozenIndPHIInfo(PHINode *PHI, BinaryOperator *StepInst)
      : PHI(PHI), StepInst(StepInst) {}

  bool operator==(const FrozenIndPHIInfo &Other) { return FI == Other.FI; }
};

} // namespace

template <> struct llvm::DenseMapInfo<FrozenIndPHIInfo> {
  static inline FrozenIndPHIInfo getEmptyKey() {
    return FrozenIndPHIInfo(DenseMapInfo<PHINode *>::getEmptyKey(),
                            DenseMapInfo<BinaryOperator *>::getEmptyKey());
  }

  static inline FrozenIndPHIInfo getTombstoneKey() {
    return FrozenIndPHIInfo(DenseMapInfo<PHINode *>::getTombstoneKey(),
                            DenseMapInfo<BinaryOperator *>::getTombstoneKey());
  }

  static unsigned getHashValue(const FrozenIndPHIInfo &Val) {
    return DenseMapInfo<FreezeInst *>::getHashValue(Val.FI);
  };

```
- EN: Core entities appearing here include FrozenIndPHIInfo, getEmptyKey, getTombstoneKey, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 FrozenIndPHIInfo, getEmptyKey, getTombstoneKey，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 119-146

```cpp
  static bool isEqual(const FrozenIndPHIInfo &LHS,
                      const FrozenIndPHIInfo &RHS) {
    return LHS.FI == RHS.FI;
  };
};

// Given U = (value, user), replace value with freeze(value), and let
// SCEV forget user. The inserted freeze is placed in the preheader.
void CanonicalizeFreezeInLoopsImpl::InsertFreezeAndForgetFromSCEV(Use &U) {
  auto *PH = L->getLoopPreheader();

  auto *UserI = cast<Instruction>(U.getUser());
  auto *ValueToFr = U.get();
  assert(L->contains(UserI->getParent()) &&
         "Should not process an instruction that isn't inside the loop");
  if (isGuaranteedNotToBeUndefOrPoison(ValueToFr, nullptr, UserI, &DT))
    return;

  LLVM_DEBUG(dbgs() << "canonfr: inserting freeze:\n");
  LLVM_DEBUG(dbgs() << "\tUser: " << *U.getUser() << "\n");
  LLVM_DEBUG(dbgs() << "\tOperand: " << *U.get() << "\n");

  U.set(new FreezeInst(ValueToFr, ValueToFr->getName() + ".frozen",
                       PH->getTerminator()->getIterator()));

  SE.forgetValue(UserI);
}

```
- EN: Core entities appearing here include InsertFreezeAndForgetFromSCEV, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 InsertFreezeAndForgetFromSCEV，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 147-176

```cpp
bool CanonicalizeFreezeInLoopsImpl::run() {
  // The loop should be in LoopSimplify form.
  if (!L->isLoopSimplifyForm())
    return false;

  SmallSetVector<FrozenIndPHIInfo, 4> Candidates;

  for (auto &PHI : L->getHeader()->phis()) {
    InductionDescriptor ID;
    if (!InductionDescriptor::isInductionPHI(&PHI, L, &SE, ID))
      continue;

    LLVM_DEBUG(dbgs() << "canonfr: PHI: " << PHI << "\n");
    FrozenIndPHIInfo Info(&PHI, ID.getInductionBinOp());
    if (!Info.StepInst || !canHandleInst(Info.StepInst)) {
      // The stepping instruction has unknown form.
      // Ignore this PHI.
      continue;
    }

    Info.StepValIdx = Info.StepInst->getOperand(0) == &PHI;
    Value *StepV = Info.StepInst->getOperand(Info.StepValIdx);
    if (auto *StepI = dyn_cast<Instruction>(StepV)) {
      if (L->contains(StepI->getParent())) {
        // The step value is inside the loop. Freezing step value will introduce
        // another freeze into the loop, so skip this PHI.
        continue;
      }
    }

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 177-206

```cpp
    auto Visit = [&](User *U) {
      if (auto *FI = dyn_cast<FreezeInst>(U)) {
        LLVM_DEBUG(dbgs() << "canonfr: found: " << *FI << "\n");
        Info.FI = FI;
        Candidates.insert(Info);
      }
    };
    for_each(PHI.users(), Visit);
    for_each(Info.StepInst->users(), Visit);
  }

  if (Candidates.empty())
    return false;

  SmallPtrSet<PHINode *, 8> ProcessedPHIs;
  for (const auto &Info : Candidates) {
    PHINode *PHI = Info.PHI;
    if (!ProcessedPHIs.insert(Info.PHI).second)
      continue;

    BinaryOperator *StepI = Info.StepInst;
    assert(StepI && "Step instruction should have been found");

    // Drop flags from the step instruction.
    if (!isGuaranteedNotToBeUndefOrPoison(StepI, nullptr, StepI, &DT)) {
      LLVM_DEBUG(dbgs() << "canonfr: drop flags: " << *StepI << "\n");
      StepI->dropPoisonGeneratingFlags();
      SE.forgetValue(StepI);
    }

```
- EN: This region continues the CanonicalizeFreezeInLoops implementation with local helper logic centered on Visit, User, FreezeInst, LLVM_DEBUG.
- CN: 这一段延续了 CanonicalizeFreezeInLoops 的主体实现，围绕 Visit, User, FreezeInst, LLVM_DEBUG 等局部辅助逻辑展开。

### Lines 207-234

```cpp
    InsertFreezeAndForgetFromSCEV(StepI->getOperandUse(Info.StepValIdx));

    unsigned OperandIdx =
        PHI->getOperandNumForIncomingValue(PHI->getIncomingValue(0) == StepI);
    InsertFreezeAndForgetFromSCEV(PHI->getOperandUse(OperandIdx));
  }

  // Finally, remove the old freeze instructions.
  for (const auto &Item : Candidates) {
    auto *FI = Item.FI;
    LLVM_DEBUG(dbgs() << "canonfr: removing " << *FI << "\n");
    SE.forgetValue(FI);
    FI->replaceAllUsesWith(FI->getOperand(0));
    FI->eraseFromParent();
  }

  return true;
}

CanonicalizeFreezeInLoops::CanonicalizeFreezeInLoops() : LoopPass(ID) {
  initializeCanonicalizeFreezeInLoopsPass(*PassRegistry::getPassRegistry());
}

void CanonicalizeFreezeInLoops::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addPreservedID(LoopSimplifyID);
  AU.addRequired<LoopInfoWrapperPass>();
  AU.addPreserved<LoopInfoWrapperPass>();
  AU.addRequiredID(LoopSimplifyID);
```
- EN: Core entities appearing here include CanonicalizeFreezeInLoops, getAnalysisUsage, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 CanonicalizeFreezeInLoops, getAnalysisUsage，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 235-259

```cpp
  AU.addRequired<ScalarEvolutionWrapperPass>();
  AU.addPreserved<ScalarEvolutionWrapperPass>();
  AU.addRequired<DominatorTreeWrapperPass>();
  AU.addPreserved<DominatorTreeWrapperPass>();
}

bool CanonicalizeFreezeInLoops::runOnLoop(Loop *L, LPPassManager &) {
  if (skipLoop(L))
    return false;

  auto &SE = getAnalysis<ScalarEvolutionWrapperPass>().getSE();
  auto &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  return CanonicalizeFreezeInLoopsImpl(L, SE, DT).run();
}

PreservedAnalyses
CanonicalizeFreezeInLoopsPass::run(Loop &L, LoopAnalysisManager &AM,
                                   LoopStandardAnalysisResults &AR,
                                   LPMUpdater &U) {
  if (!CanonicalizeFreezeInLoopsImpl(&L, AR.SE, AR.DT).run())
    return PreservedAnalyses::all();

  return getLoopPassPreservedAnalyses();
}

```
- EN: Core entities appearing here include runOnLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution.
- CN: 此处出现的核心实体包括 runOnLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution 等分析结果。

### Lines 260-272

```cpp
INITIALIZE_PASS_BEGIN(CanonicalizeFreezeInLoops, "canon-freeze",
                      "Canonicalize Freeze Instructions in Loops", false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LoopSimplify)
INITIALIZE_PASS_END(CanonicalizeFreezeInLoops, "canon-freeze",
                    "Canonicalize Freeze Instructions in Loops", false, false)

Pass *llvm::createCanonicalizeFreezeInLoopsPass() {
  return new CanonicalizeFreezeInLoops();
}

char CanonicalizeFreezeInLoops::ID = 0;
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `CanonicalizeFreezeInLoops, CanonicalizeFreezeInLoopsImpl, FrozenIndPHIInfo, canHandleInst, getEmptyKey, getTombstoneKey, getHashValue, InsertFreezeAndForgetFromSCEV` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`CanonicalizeFreezeInLoops, CanonicalizeFreezeInLoopsImpl, FrozenIndPHIInfo, canHandleInst, getEmptyKey, getTombstoneKey, getHashValue, InsertFreezeAndForgetFromSCEV` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, LoopInfo, ScalarEvolution, SCEV, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, LoopInfo, ScalarEvolution, SCEV, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/LoopAnalysisManager.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/LoopAnalysisManager.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/Transforms/Utils/CanonicalizeFreezeInLoops.h`, `llvm/Transforms/Utils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/Transforms/Utils/CanonicalizeFreezeInLoops.h`, `llvm/Transforms/Utils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
