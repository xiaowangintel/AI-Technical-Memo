# FixIrreducible.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/FixIrreducible.cpp` | `llvm/lib/Transforms/Utils/FixIrreducible.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements convert irreducible control-flow into loops within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 FixIrreducible 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-39

```cpp
//===- FixIrreducible.cpp - Convert irreducible control-flow into loops ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// INPUT CFG: The blocks H and B form an irreducible cycle with two headers.
//
//                        Entry
//                       /     \
//                      v       v
//                      H ----> B
//                      ^      /|
//                       `----' |
//                              v
//                             Exit
//
// OUTPUT CFG: Converted to a natural loop with a new header N.
//
//                        Entry
//                          |
//                          v
//                          N <---.
//                         / \     \
//                        /   \     |
//                       v     v    /
//                       H --> B --'
//                             |
//                             v
//                            Exit
//
// To convert an irreducible cycle C to a natural loop L:
//
// 1. Add a new node N to C.
// 2. Redirect all external incoming edges through N.
// 3. Redirect all edges incident on header H through N.
//
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。

### Lines 40-78

```cpp
// This is sufficient to ensure that:
//
// a. Every closed path in C also exists in L, with the modification that any
//    path passing through H now passes through N before reaching H.
// b. Every external path incident on any entry of C is now incident on N and
//    then redirected to the entry.
//
// Thus, L is a strongly connected component dominated by N, and hence L is a
// natural loop with header N.
//
// When an irreducible cycle C with header H is transformed into a loop, the
// following invariants hold:
//
// 1. No new subcycles are "discovered" in the set (C-H). The only internal
//    edges that are redirected by the transform are incident on H. Any subcycle
//    S in (C-H), already existed prior to this transform, and is already in the
//    list of children for this cycle C.
//
// 2. Subcycles of C are not modified by the transform. For some subcycle S of
//    C, edges incident on the entries of S are either internal to C, or they
//    are now redirected through N, which is outside of S. So the list of
//    entries to S does not change. Since the transform only adds a block
//    outside S, and redirects edges that are not internal to S, the list of
//    blocks in S does not change.
//
// 3. Similarly, any natural loop L included in C is not affected, with one
//    exception: L is "destroyed" by the transform iff its header is H. The
//    backedges of such a loop are now redirected to N instead, and hence the
//    body of this loop gets merged into the new loop with header N.
//
// The actual transformation is handled by the ControlFlowHub, which redirects
// specified control flow edges through a set of guard blocks. This also moves
// every PHINode in an outgoing block to the hub. Since the hub dominates all
// the outgoing blocks, each such PHINode continues to dominate its uses. Since
// every header in an SCC has at least two predecessors, every value used in the
// header (or later) but defined in a predecessor (or earlier) is represented by
// a PHINode in a header. Hence the above handling of PHINodes is sufficient and
// no further processing is required to restore SSA.
//
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 79-117

```cpp
// Limitation: The pass cannot handle switch statements and indirect
//             branches. Both must be lowered to plain branches first.
//
// CallBr support: CallBr is handled as a more general branch instruction which
// can have multiple successors. The pass redirects the edges to intermediate
// target blocks that unconditionally branch to the original callbr target
// blocks. This allows the control flow hub to know to which of the original
// target blocks to jump to.
// Example input CFG:
//                        Entry (callbr)
//                       /     \
//                      v       v
//                      H ----> B
//                      ^      /|
//                       `----' |
//                              v
//                             Exit
//
// becomes:
//                        Entry (callbr)
//                       /     \
//                      v       v
//                 target.H   target.B
//                      |       |
//                      v       v
//                      H ----> B
//                      ^      /|
//                       `----' |
//                              v
//                             Exit
//
// Note
// OUTPUT CFG: Converted to a natural loop with a new header N.
//
//                        Entry (callbr)
//                       /     \
//                      v       v
//                 target.H   target.B
//                      \       /
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 118-159

```cpp
//                       \     /
//                        v   v
//                          N <---.
//                         / \     \
//                        /   \     |
//                       v     v    /
//                       H --> B --'
//                             |
//                             v
//                            Exit
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/FixIrreducible.h"
#include "llvm/Analysis/CycleAnalysis.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/ControlFlowUtils.h"

#define DEBUG_TYPE "fix-irreducible"

using namespace llvm;

namespace {
struct FixIrreducible : public FunctionPass {
  static char ID;
  FixIrreducible() : FunctionPass(ID) {
    initializeFixIrreduciblePass(*PassRegistry::getPassRegistry());
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<DominatorTreeWrapperPass>();
    AU.addRequired<CycleInfoWrapperPass>();
    AU.addPreserved<DominatorTreeWrapperPass>();
    AU.addPreserved<CycleInfoWrapperPass>();
    AU.addPreserved<LoopInfoWrapperPass>();
  }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 160-198

```cpp
  bool runOnFunction(Function &F) override;
};
} // namespace

char FixIrreducible::ID = 0;

FunctionPass *llvm::createFixIrreduciblePass() { return new FixIrreducible(); }

INITIALIZE_PASS_BEGIN(FixIrreducible, "fix-irreducible",
                      "Convert irreducible control-flow into natural loops",
                      false /* Only looks at CFG */, false /* Analysis Pass */)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
INITIALIZE_PASS_END(FixIrreducible, "fix-irreducible",
                    "Convert irreducible control-flow into natural loops",
                    false /* Only looks at CFG */, false /* Analysis Pass */)

// When a new loop is created, existing children of the parent loop may now be
// fully inside the new loop. Reconnect these as children of the new loop.
static void reconnectChildLoops(LoopInfo &LI, Loop *ParentLoop, Loop *NewLoop,
                                BasicBlock *OldHeader) {
  auto &CandidateLoops = ParentLoop ? ParentLoop->getSubLoopsVector()
                                    : LI.getTopLevelLoopsVector();
  // Any candidate is a child iff its header is owned by the new loop. Move all
  // the children to a new vector.
  auto FirstChild = llvm::partition(CandidateLoops, [&](Loop *L) {
    return NewLoop == L || !NewLoop->contains(L->getHeader());
  });
  SmallVector<Loop *, 8> ChildLoops(FirstChild, CandidateLoops.end());
  CandidateLoops.erase(FirstChild, CandidateLoops.end());

  for (Loop *Child : ChildLoops) {
    LLVM_DEBUG(dbgs() << "child loop: " << Child->getHeader()->getName()
                      << "\n");
    // A child loop whose header was the old cycle header gets destroyed since
    // its backedges are removed.
    if (Child->getHeader() == OldHeader) {
      for (auto *BB : Child->blocks()) {
        if (LI.getLoopFor(BB) != Child)
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。

### Lines 199-238

```cpp
          continue;
        LI.changeLoopFor(BB, NewLoop);
        LLVM_DEBUG(dbgs() << "moved block from child: " << BB->getName()
                          << "\n");
      }
      std::vector<Loop *> GrandChildLoops;
      std::swap(GrandChildLoops, Child->getSubLoopsVector());
      for (auto *GrandChildLoop : GrandChildLoops) {
        GrandChildLoop->setParentLoop(nullptr);
        NewLoop->addChildLoop(GrandChildLoop);
      }
      LI.destroy(Child);
      LLVM_DEBUG(dbgs() << "subsumed child loop (common header)\n");
      continue;
    }

    Child->setParentLoop(nullptr);
    NewLoop->addChildLoop(Child);
    LLVM_DEBUG(dbgs() << "added child loop to new loop\n");
  }
}

static void updateLoopInfo(LoopInfo &LI, Cycle &C,
                           ArrayRef<BasicBlock *> GuardBlocks) {
  // The parent loop is a natural loop L mapped to the cycle header H as long as
  // H is not also the header of L. In the latter case, L is destroyed and we
  // seek its parent instead.
  BasicBlock *CycleHeader = C.getHeader();
  Loop *ParentLoop = LI.getLoopFor(CycleHeader);
  if (ParentLoop && ParentLoop->getHeader() == CycleHeader)
    ParentLoop = ParentLoop->getParentLoop();

  // Create a new loop from the now-transformed cycle
  auto *NewLoop = LI.AllocateLoop();
  if (ParentLoop) {
    ParentLoop->addChildLoop(NewLoop);
  } else {
    LI.addTopLevelLoop(NewLoop);
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 239-280

```cpp
  // Add the guard blocks to the new loop. The first guard block is
  // the head of all the backedges, and it is the first to be inserted
  // in the loop. This ensures that it is recognized as the
  // header. Since the new loop is already in LoopInfo, the new blocks
  // are also propagated up the chain of parent loops.
  for (auto *G : GuardBlocks) {
    LLVM_DEBUG(dbgs() << "added guard block to loop: " << G->getName() << "\n");
    NewLoop->addBasicBlockToLoop(G, LI);
  }

  for (auto *BB : C.blocks()) {
    NewLoop->addBlockEntry(BB);
    if (LI.getLoopFor(BB) == ParentLoop) {
      LLVM_DEBUG(dbgs() << "moved block from parent: " << BB->getName()
                        << "\n");
      LI.changeLoopFor(BB, NewLoop);
    } else {
      LLVM_DEBUG(dbgs() << "added block from child: " << BB->getName() << "\n");
    }
  }
  LLVM_DEBUG(dbgs() << "header for new loop: "
                    << NewLoop->getHeader()->getName() << "\n");

  reconnectChildLoops(LI, ParentLoop, NewLoop, C.getHeader());

  LLVM_DEBUG(dbgs() << "Verify new loop.\n"; NewLoop->print(dbgs()));
  NewLoop->verifyLoop();
  if (ParentLoop) {
    LLVM_DEBUG(dbgs() << "Verify parent loop.\n"; ParentLoop->print(dbgs()));
    ParentLoop->verifyLoop();
  }
}

// Given a set of blocks and headers in an irreducible SCC, convert it into a
// natural loop. Also insert this new loop at its appropriate place in the
// hierarchy of loops.
static bool fixIrreducible(Cycle &C, CycleInfo &CI, DominatorTree &DT,
                           LoopInfo *LI) {
  if (C.isReducible())
    return false;
  LLVM_DEBUG(dbgs() << "Processing cycle:\n" << CI.print(&C) << "\n";);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 281-319

```cpp
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Eager);
  ControlFlowHub CHub;
  SetVector<BasicBlock *> Predecessors;

  // Redirect internal edges incident on the header.
  BasicBlock *Header = C.getHeader();
  for (BasicBlock *P : predecessors(Header)) {
    if (C.contains(P))
      Predecessors.insert(P);
  }

  for (BasicBlock *P : Predecessors) {
    if (isa<UncondBrInst>(P->getTerminator())) {
      assert(P->getTerminator()->getSuccessor(0) == Header);
      CHub.addBranch(P, Header);

      LLVM_DEBUG(dbgs() << "Added internal branch: " << printBasicBlock(P)
                        << " -> " << printBasicBlock(Header) << '\n');
    } else if (CondBrInst *Branch = dyn_cast<CondBrInst>(P->getTerminator())) {
      // Exactly one of the two successors is the header.
      BasicBlock *Succ0 = Branch->getSuccessor(0) == Header ? Header : nullptr;
      BasicBlock *Succ1 = Succ0 ? nullptr : Header;
      assert(Succ0 || Branch->getSuccessor(1) == Header);
      assert(Succ0 || Succ1);
      CHub.addBranch(P, Succ0, Succ1);

      LLVM_DEBUG(dbgs() << "Added internal branch: " << printBasicBlock(P)
                        << " -> " << printBasicBlock(Succ0)
                        << (Succ0 && Succ1 ? " " : "") << printBasicBlock(Succ1)
                        << '\n');
    } else if (CallBrInst *CallBr = dyn_cast<CallBrInst>(P->getTerminator())) {
      for (unsigned I = 0; I < CallBr->getNumSuccessors(); ++I) {
        BasicBlock *Succ = CallBr->getSuccessor(I);
        if (Succ != Header)
          continue;
        BasicBlock *NewSucc = SplitCallBrEdge(P, Succ, I, &DTU, &CI, LI);
        CHub.addBranch(NewSucc, Succ);
        LLVM_DEBUG(dbgs() << "Added internal branch: "
                          << printBasicBlock(NewSucc) << " -> "
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。

### Lines 320-358

```cpp
                          << printBasicBlock(Succ) << '\n');
      }
    } else {
      llvm_unreachable("unsupported block terminator");
    }
  }

  // Redirect external incoming edges. This includes the edges on the header.
  Predecessors.clear();
  for (BasicBlock *E : C.entries()) {
    for (BasicBlock *P : predecessors(E)) {
      if (!C.contains(P))
        Predecessors.insert(P);
    }
  }

  for (BasicBlock *P : Predecessors) {
    if (UncondBrInst *Branch = dyn_cast<UncondBrInst>(P->getTerminator())) {
      BasicBlock *Succ0 = Branch->getSuccessor();
      Succ0 = C.contains(Succ0) ? Succ0 : nullptr;
      CHub.addBranch(P, Succ0);

      LLVM_DEBUG(dbgs() << "Added external branch: " << printBasicBlock(P)
                        << " -> " << printBasicBlock(Succ0) << '\n');
    } else if (CondBrInst *Branch = dyn_cast<CondBrInst>(P->getTerminator())) {
      BasicBlock *Succ0 = Branch->getSuccessor(0);
      Succ0 = C.contains(Succ0) ? Succ0 : nullptr;
      BasicBlock *Succ1 = Branch->getSuccessor(1);
      Succ1 = C.contains(Succ1) ? Succ1 : nullptr;
      CHub.addBranch(P, Succ0, Succ1);

      LLVM_DEBUG(dbgs() << "Added external branch: " << printBasicBlock(P)
                        << " -> " << printBasicBlock(Succ0)
                        << (Succ0 && Succ1 ? " " : "") << printBasicBlock(Succ1)
                        << '\n');
    } else if (CallBrInst *CallBr = dyn_cast<CallBrInst>(P->getTerminator())) {
      for (unsigned I = 0; I < CallBr->getNumSuccessors(); ++I) {
        BasicBlock *Succ = CallBr->getSuccessor(I);
        if (!C.contains(Succ))
```
- EN: This region continues the FixIrreducible implementation with local helper logic centered on Succ, Redirect, This, Predecessors.
- CN: 这一段延续了 FixIrreducible 的主体实现，围绕 Succ, Redirect, This, Predecessors 等局部辅助逻辑展开。

### Lines 359-402

```cpp
          continue;
        BasicBlock *NewSucc = SplitCallBrEdge(P, Succ, I, &DTU, &CI, LI);
        CHub.addBranch(NewSucc, Succ);
        LLVM_DEBUG(dbgs() << "Added external branch: "
                          << printBasicBlock(NewSucc) << " -> "
                          << printBasicBlock(Succ) << '\n');
      }
    } else {
      llvm_unreachable("unsupported block terminator");
    }
  }

  // Redirect all the backedges through a "hub" consisting of a series
  // of guard blocks that manage the flow of control from the
  // predecessors to the headers.
  SmallVector<BasicBlock *> GuardBlocks;

  // Minor optimization: The cycle entries are discovered in an order that is
  // the opposite of the order in which these blocks appear as branch targets.
  // This results in a lot of condition inversions in the control flow out of
  // the new ControlFlowHub, which can be mitigated if the orders match. So we
  // reverse the entries when adding them to the hub.
  SetVector<BasicBlock *> Entries;
  Entries.insert(C.entry_rbegin(), C.entry_rend());

  CHub.finalize(&DTU, GuardBlocks, "irr");
#if defined(EXPENSIVE_CHECKS)
  assert(DT.verify(DominatorTree::VerificationLevel::Full));
#else
  assert(DT.verify(DominatorTree::VerificationLevel::Fast));
#endif

  // If we are updating LoopInfo, do that now before modifying the cycle. This
  // ensures that the first guard block is the header of a new natural loop.
  if (LI)
    updateLoopInfo(*LI, C, GuardBlocks);

  for (auto *G : GuardBlocks) {
    LLVM_DEBUG(dbgs() << "added guard block to cycle: " << G->getName()
                      << "\n");
    CI.addBlockToCycle(G, &C);
  }
  C.setSingleEntry(GuardBlocks[0]);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 403-443

```cpp
  C.verifyCycle();
  if (Cycle *Parent = C.getParentCycle())
    Parent->verifyCycle();

  LLVM_DEBUG(dbgs() << "Finished one cycle:\n"; CI.print(dbgs()););
  return true;
}

static bool FixIrreducibleImpl(Function &F, CycleInfo &CI, DominatorTree &DT,
                               LoopInfo *LI) {
  LLVM_DEBUG(dbgs() << "===== Fix irreducible control-flow in function: "
                    << F.getName() << "\n");

  bool Changed = false;
  for (Cycle *TopCycle : CI.toplevel_cycles()) {
    for (Cycle *C : depth_first(TopCycle)) {
      Changed |= fixIrreducible(*C, CI, DT, LI);
    }
  }

  if (!Changed)
    return false;

#if defined(EXPENSIVE_CHECKS)
  CI.verify();
  if (LI) {
    LI->verify(DT);
  }
#endif // EXPENSIVE_CHECKS

  return true;
}

bool FixIrreducible::runOnFunction(Function &F) {
  auto *LIWP = getAnalysisIfAvailable<LoopInfoWrapperPass>();
  LoopInfo *LI = LIWP ? &LIWP->getLoopInfo() : nullptr;
  auto &CI = getAnalysis<CycleInfoWrapperPass>().getResult();
  auto &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  return FixIrreducibleImpl(F, CI, DT, LI);
}

```
- EN: Core entities appearing here include runOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo.
- CN: 此处出现的核心实体包括 runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。

### Lines 444-458

```cpp
PreservedAnalyses FixIrreduciblePass::run(Function &F,
                                          FunctionAnalysisManager &AM) {
  auto *LI = AM.getCachedResult<LoopAnalysis>(F);
  auto &CI = AM.getResult<CycleAnalysis>(F);
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);

  if (!FixIrreducibleImpl(F, CI, DT, LI))
    return PreservedAnalyses::all();

  PreservedAnalyses PA;
  PA.preserve<LoopAnalysis>();
  PA.preserve<CycleAnalysis>();
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `FixIrreducible, runOnFunction` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`FixIrreducible, runOnFunction` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, DomTreeUpdater, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, DomTreeUpdater, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/CycleAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/CycleAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/Transforms/Utils/FixIrreducible.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/ControlFlowUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Utils/FixIrreducible.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/ControlFlowUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `DomTreeUpdater`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `DomTreeUpdater`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
