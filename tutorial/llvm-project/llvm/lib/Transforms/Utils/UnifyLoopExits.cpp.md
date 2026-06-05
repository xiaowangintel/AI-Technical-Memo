# UnifyLoopExits.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/UnifyLoopExits.cpp` | `llvm/lib/Transforms/Utils/UnifyLoopExits.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements redirect exiting edges to one block within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 UnifyLoopExits 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-34

```cpp
//===- UnifyLoopExits.cpp - Redirect exiting edges to one block -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// For each natural loop with multiple exit blocks, this pass creates a new
// block N such that all exiting blocks now branch to N, and then control flow
// is redistributed to all the original exit blocks.
//
// Limitation: This assumes that all terminators in the CFG are direct branches
//             (the "br" instruction). The presence of any other control flow
//             such as indirectbr or switch will cause an assert.
//             The callbr terminator is supported by creating intermediate
//             target blocks that unconditionally branch to the original target
//             blocks. These intermediate target blocks can then be redirected
//             through the ControlFlowHub as usual.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/UnifyLoopExits.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/ControlFlowUtils.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, LoopInfo.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, LoopInfo 等分析结果。

### Lines 35-67

```cpp
#define DEBUG_TYPE "unify-loop-exits"

using namespace llvm;

static cl::opt<unsigned> MaxBooleansInControlFlowHub(
    "max-booleans-in-control-flow-hub", cl::init(32), cl::Hidden,
    cl::desc("Set the maximum number of outgoing blocks for using a boolean "
             "value to record the exiting block in the ControlFlowHub."));

namespace {
struct UnifyLoopExitsLegacyPass : public FunctionPass {
  static char ID;
  UnifyLoopExitsLegacyPass() : FunctionPass(ID) {
    initializeUnifyLoopExitsLegacyPassPass(*PassRegistry::getPassRegistry());
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LoopInfoWrapperPass>();
    AU.addRequired<DominatorTreeWrapperPass>();
    AU.addPreserved<LoopInfoWrapperPass>();
    AU.addPreserved<DominatorTreeWrapperPass>();
  }

  bool runOnFunction(Function &F) override;
};
} // namespace

char UnifyLoopExitsLegacyPass::ID = 0;

FunctionPass *llvm::createUnifyLoopExitsPass() {
  return new UnifyLoopExitsLegacyPass();
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include UnifyLoopExitsLegacyPass, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 UnifyLoopExitsLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 68-97

```cpp
INITIALIZE_PASS_BEGIN(UnifyLoopExitsLegacyPass, "unify-loop-exits",
                      "Fixup each natural loop to have a single exit block",
                      false /* Only looks at CFG */, false /* Analysis Pass */)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
INITIALIZE_PASS_END(UnifyLoopExitsLegacyPass, "unify-loop-exits",
                    "Fixup each natural loop to have a single exit block",
                    false /* Only looks at CFG */, false /* Analysis Pass */)

// The current transform introduces new control flow paths which may break the
// SSA requirement that every def must dominate all its uses. For example,
// consider a value D defined inside the loop that is used by some instruction
// U outside the loop. It follows that D dominates U, since the original
// program has valid SSA form. After merging the exits, all paths from D to U
// now flow through the unified exit block. In addition, there may be other
// paths that do not pass through D, but now reach the unified exit
// block. Thus, D no longer dominates U.
//
// Restore the dominance by creating a phi for each such D at the new unified
// loop exit. But when doing this, ignore any uses U that are in the new unified
// loop exit, since those were introduced specially when the block was created.
//
// The use of SSAUpdater seems like overkill for this operation. The location
// for creating the new PHI is well-known, and also the set of incoming blocks
// to the new PHI.
static void restoreSSA(const DominatorTree &DT, const Loop *L,
                       SmallVectorImpl<BasicBlock *> &Incoming,
                       BasicBlock *LoopExitBlock) {
  using InstVector = SmallVector<Instruction *, 8>;
  using IIMap = MapVector<Instruction *, InstVector>;
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。

### Lines 98-127

```cpp
  IIMap ExternalUsers;
  for (auto *BB : L->blocks()) {
    for (auto &I : *BB) {
      for (auto &U : I.uses()) {
        auto UserInst = cast<Instruction>(U.getUser());
        auto UserBlock = UserInst->getParent();
        if (UserBlock == LoopExitBlock)
          continue;
        if (L->contains(UserBlock))
          continue;
        LLVM_DEBUG(dbgs() << "added ext use for " << I.getName() << "("
                          << BB->getName() << ")"
                          << ": " << UserInst->getName() << "("
                          << UserBlock->getName() << ")"
                          << "\n");
        ExternalUsers[&I].push_back(UserInst);
      }
    }
  }

  for (const auto &II : ExternalUsers) {
    // For each Def used outside the loop, create NewPhi in
    // LoopExitBlock. NewPhi receives Def only along exiting blocks that
    // dominate it, while the remaining values are undefined since those paths
    // didn't exist in the original CFG.
    auto Def = II.first;
    LLVM_DEBUG(dbgs() << "externally used: " << Def->getName() << "\n");
    auto NewPhi =
        PHINode::Create(Def->getType(), Incoming.size(),
                        Def->getName() + ".moved", LoopExitBlock->begin());
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 128-160

```cpp
    for (auto *In : Incoming) {
      LLVM_DEBUG(dbgs() << "predecessor " << In->getName() << ": ");
      if (Def->getParent() == In || DT.dominates(Def, In)) {
        LLVM_DEBUG(dbgs() << "dominated\n");
        NewPhi->addIncoming(Def, In);
      } else {
        LLVM_DEBUG(dbgs() << "not dominated\n");
        NewPhi->addIncoming(PoisonValue::get(Def->getType()), In);
      }
    }

    LLVM_DEBUG(dbgs() << "external users:");
    for (auto *U : II.second) {
      LLVM_DEBUG(dbgs() << " " << U->getName());
      U->replaceUsesOfWith(Def, NewPhi);
    }
    LLVM_DEBUG(dbgs() << "\n");
  }
}

static bool unifyLoopExits(DominatorTree &DT, LoopInfo &LI, Loop *L) {
  // To unify the loop exits, we need a list of the exiting blocks as
  // well as exit blocks. The functions for locating these lists both
  // traverse the entire loop body. It is more efficient to first
  // locate the exiting blocks and then examine their successors to
  // locate the exit blocks.
  SmallVector<BasicBlock *, 8> ExitingBlocks;
  L->getExitingBlocks(ExitingBlocks);

  // No exit blocks, so nothing to do. Just return.
  if (ExitingBlocks.empty())
    return false;

```
- EN: Core entities appearing here include unifyLoopExits, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo.
- CN: 此处出现的核心实体包括 unifyLoopExits，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。

### Lines 161-190

```cpp
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Eager);
  SmallVector<BasicBlock *, 8> CallBrTargetBlocksToFix;

  // Redirect exiting edges through a control flow hub.
  ControlFlowHub CHub;
  bool Changed = false;

  for (unsigned I = 0; I < ExitingBlocks.size(); ++I) {
    BasicBlock *BB = ExitingBlocks[I];
    if (UncondBrInst *Branch = dyn_cast<UncondBrInst>(BB->getTerminator())) {
      BasicBlock *Succ0 = Branch->getSuccessor(0);
      Succ0 = L->contains(Succ0) ? nullptr : Succ0;
      CHub.addBranch(BB, Succ0);

      LLVM_DEBUG(dbgs() << "Added extiting branch: " << printBasicBlock(BB)
                        << " -> " << printBasicBlock(Succ0) << '\n');
    } else if (CondBrInst *Branch = dyn_cast<CondBrInst>(BB->getTerminator())) {
      BasicBlock *Succ0 = Branch->getSuccessor(0);
      Succ0 = L->contains(Succ0) ? nullptr : Succ0;

      BasicBlock *Succ1 = Branch->getSuccessor(1);
      Succ1 = L->contains(Succ1) ? nullptr : Succ1;
      CHub.addBranch(BB, Succ0, Succ1);

      LLVM_DEBUG(dbgs() << "Added extiting branch: " << printBasicBlock(BB)
                        << " -> " << printBasicBlock(Succ0)
                        << (Succ0 && Succ1 ? " " : "") << printBasicBlock(Succ1)
                        << '\n');
    } else if (CallBrInst *CallBr = dyn_cast<CallBrInst>(BB->getTerminator())) {
      for (unsigned J = 0; J < CallBr->getNumSuccessors(); ++J) {
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。

### Lines 191-220

```cpp
        BasicBlock *Succ = CallBr->getSuccessor(J);
        if (L->contains(Succ))
          continue;
        bool UpdatedLI = false;
        BasicBlock *NewSucc =
            SplitCallBrEdge(BB, Succ, J, &DTU, nullptr, &LI, &UpdatedLI);
        // SplitCallBrEdge modifies the CFG because it creates an intermediate
        // block. So we need to set the changed flag no matter what the
        // ControlFlowHub is going to do later.
        Changed = true;
        // Even if CallBr and Succ do not have a common parent loop, we need to
        // add the new target block to the parent loop of the current loop.
        if (!UpdatedLI)
          CallBrTargetBlocksToFix.push_back(NewSucc);
        // ExitingBlocks is later used to restore SSA, so we need to make sure
        // that the blocks used for phi nodes in the guard blocks match the
        // predecessors of the guard blocks, which, in the case of callbr, are
        // the new intermediate target blocks instead of the callbr blocks
        // themselves.
        ExitingBlocks[I] = NewSucc;
        CHub.addBranch(NewSucc, Succ);
        LLVM_DEBUG(dbgs() << "Added exiting branch: "
                          << printBasicBlock(NewSucc) << " -> "
                          << printBasicBlock(Succ) << '\n');
      }
    } else {
      llvm_unreachable("unsupported block terminator");
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 221-250

```cpp
  SmallVector<BasicBlock *, 8> GuardBlocks;
  BasicBlock *LoopExitBlock;
  bool ChangedCFG;
  std::tie(LoopExitBlock, ChangedCFG) = CHub.finalize(
      &DTU, GuardBlocks, "loop.exit", MaxBooleansInControlFlowHub.getValue());
  ChangedCFG |= Changed;
  if (!ChangedCFG)
    return false;

  restoreSSA(DT, L, ExitingBlocks, LoopExitBlock);

#if defined(EXPENSIVE_CHECKS)
  assert(DT.verify(DominatorTree::VerificationLevel::Full));
#else
  assert(DT.verify(DominatorTree::VerificationLevel::Fast));
#endif // EXPENSIVE_CHECKS
  L->verifyLoop();

  // The guard blocks were created outside the loop, so they need to become
  // members of the parent loop.
  // Same goes for the callbr target blocks.  Although we try to add them to the
  // smallest common parent loop of the callbr block and the corresponding
  // original target block, there might not have been such a loop, in which case
  // the newly created callbr target blocks are not part of any loop. For nested
  // loops, this might result in them leading to a loop with multiple entry
  // points.
  if (auto *ParentLoop = L->getParentLoop()) {
    for (auto *G : GuardBlocks) {
      ParentLoop->addBasicBlockToLoop(G, LI);
    }
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 251-283

```cpp
    for (auto *C : CallBrTargetBlocksToFix) {
      ParentLoop->addBasicBlockToLoop(C, LI);
    }
    ParentLoop->verifyLoop();
  }

#if defined(EXPENSIVE_CHECKS)
  LI.verify(DT);
#endif // EXPENSIVE_CHECKS

  return true;
}

static bool runImpl(LoopInfo &LI, DominatorTree &DT) {

  bool Changed = false;
  auto Loops = LI.getLoopsInPreorder();
  for (auto *L : Loops) {
    LLVM_DEBUG(dbgs() << "Processing loop:\n"; L->print(dbgs()));
    Changed |= unifyLoopExits(DT, LI, L);
  }
  return Changed;
}

bool UnifyLoopExitsLegacyPass::runOnFunction(Function &F) {
  LLVM_DEBUG(dbgs() << "===== Unifying loop exits in function " << F.getName()
                    << "\n");
  auto &LI = getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
  auto &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();

  return runImpl(LI, DT);
}

```
- EN: Core entities appearing here include runImpl, runOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo.
- CN: 此处出现的核心实体包括 runImpl, runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。

### Lines 284-300

```cpp
namespace llvm {

PreservedAnalyses UnifyLoopExitsPass::run(Function &F,
                                          FunctionAnalysisManager &AM) {
  LLVM_DEBUG(dbgs() << "===== Unifying loop exits in function " << F.getName()
                    << "\n");
  auto &LI = AM.getResult<LoopAnalysis>(F);
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);

  if (!runImpl(LI, DT))
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserve<LoopAnalysis>();
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
} // namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `UnifyLoopExitsLegacyPass, unifyLoopExits, runImpl, runOnFunction` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`UnifyLoopExitsLegacyPass, unifyLoopExits, runImpl, runOnFunction` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, DomTreeUpdater, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, DomTreeUpdater, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `MaxBooleansInControlFlowHub` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `MaxBooleansInControlFlowHub` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/Transforms/Utils/UnifyLoopExits.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/ControlFlowUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/Transforms/Utils/UnifyLoopExits.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/ControlFlowUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/ADT/MapVector.h`, `llvm/InitializePasses.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/ADT/MapVector.h`, `llvm/InitializePasses.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `DomTreeUpdater`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `DomTreeUpdater`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
