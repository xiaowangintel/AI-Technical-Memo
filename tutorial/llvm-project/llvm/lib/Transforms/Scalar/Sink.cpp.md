# Sink.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/Sink.cpp` | `llvm/lib/Transforms/Scalar/Sink.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements code Sinking within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 Sink 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```cpp
//===-- Sink.cpp - Code Sinking -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass moves instructions into successor blocks, when possible, so that
// they aren't executed on paths where their results aren't needed.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/Sink.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Scalar.h"
using namespace llvm;

#define DEBUG_TYPE "sink"

STATISTIC(NumSunk, "Number of instructions sunk");
STATISTIC(NumSinkIter, "Number of sinking iterations");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 30-60

```cpp
static bool isSafeToMove(Instruction *Inst, AliasAnalysis &AA,
                         SmallPtrSetImpl<Instruction *> &Stores) {

  if (Inst->mayWriteToMemory()) {
    Stores.insert(Inst);
    return false;
  }

  // Don't sink static alloca instructions.  CodeGen assumes allocas outside the
  // entry block are dynamically sized stack objects.
  if (AllocaInst *AI = dyn_cast<AllocaInst>(Inst))
    if (AI->isStaticAlloca())
      return false;

  if (LoadInst *L = dyn_cast<LoadInst>(Inst)) {
    MemoryLocation Loc = MemoryLocation::get(L);
    for (Instruction *S : Stores)
      if (isModSet(AA.getModRefInfo(S, Loc)))
        return false;
  }

  if (Inst->isTerminator() || isa<PHINode>(Inst) || Inst->isEHPad() ||
      Inst->mayThrow() || !Inst->willReturn())
    return false;

  if (auto *Call = dyn_cast<CallBase>(Inst)) {
    // Convergent operations cannot be made control-dependent on additional
    // values.
    if (Call->isConvergent())
      return false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 61-90

```cpp
    for (Instruction *S : Stores)
      if (isModSet(AA.getModRefInfo(S, Call)))
        return false;
  }

  return true;
}

/// IsAcceptableTarget - Return true if it is possible to sink the instruction
/// in the specified basic block.
static bool IsAcceptableTarget(Instruction *Inst, BasicBlock *SuccToSinkTo,
                               DominatorTree &DT, LoopInfo &LI) {
  assert(Inst && "Instruction to be sunk is null");
  assert(SuccToSinkTo && "Candidate sink target is null");

  // It's never legal to sink an instruction into an EH-pad block.
  if (SuccToSinkTo->isEHPad())
    return false;

  // If the block has multiple predecessors, this would introduce computation
  // on different code paths.  We could split the critical edge, but for now we
  // just punt.
  // FIXME: Split critical edges if not backedges.
  if (SuccToSinkTo->getUniquePredecessor() != Inst->getParent()) {
    // We cannot sink a load across a critical edge - there may be stores in
    // other code paths.
    if (Inst->mayReadFromMemory() &&
        !Inst->hasMetadata(LLVMContext::MD_invariant_load))
      return false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 91-122

```cpp
    // Don't sink instructions into a loop.
    Loop *succ = LI.getLoopFor(SuccToSinkTo);
    Loop *cur = LI.getLoopFor(Inst->getParent());
    if (succ != nullptr && succ != cur)
      return false;
  }

  return true;
}

/// SinkInstruction - Determine whether it is safe to sink the specified machine
/// instruction out of its current block into a successor.
static bool SinkInstruction(Instruction *Inst,
                            SmallPtrSetImpl<Instruction *> &Stores,
                            DominatorTree &DT, LoopInfo &LI, AAResults &AA) {

  // Check if it's safe to move the instruction.
  if (!isSafeToMove(Inst, AA, Stores))
    return false;

  // FIXME: This should include support for sinking instructions within the
  // block they are currently in to shorten the live ranges.  We often get
  // instructions sunk into the top of a large block, but it would be better to
  // also sink them down before their first use in the block.  This xform has to
  // be careful not to *increase* register pressure though, e.g. sinking
  // "x = y + z" down if it kills y and z would increase the live ranges of y
  // and z and only shrink the live range of x.

  // SuccToSinkTo - This is the successor to sink this instruction to, once we
  // decide.
  BasicBlock *SuccToSinkTo = nullptr;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 123-153

```cpp
  // Find the nearest common dominator of all users as the candidate.
  BasicBlock *BB = Inst->getParent();
  for (Use &U : Inst->uses()) {
    Instruction *UseInst = cast<Instruction>(U.getUser());
    BasicBlock *UseBlock = UseInst->getParent();
    if (PHINode *PN = dyn_cast<PHINode>(UseInst)) {
      // PHI nodes use the operand in the predecessor block, not the block with
      // the PHI.
      unsigned Num = PHINode::getIncomingValueNumForOperand(U.getOperandNo());
      UseBlock = PN->getIncomingBlock(Num);
    }
    // Don't worry about dead users.
    if (!DT.isReachableFromEntry(UseBlock))
      continue;

    if (SuccToSinkTo)
      SuccToSinkTo = DT.findNearestCommonDominator(SuccToSinkTo, UseBlock);
    else
      SuccToSinkTo = UseBlock;
  }

  if (SuccToSinkTo) {
    // The nearest common dominator may be in a parent loop of BB, which may not
    // be beneficial. Find an ancestor.
    while (SuccToSinkTo != BB &&
           !IsAcceptableTarget(Inst, SuccToSinkTo, DT, LI))
      SuccToSinkTo = DT.getNode(SuccToSinkTo)->getIDom()->getBlock();
    if (SuccToSinkTo == BB)
      SuccToSinkTo = nullptr;
  }

```
- EN: This region continues the Sink implementation with local helper logic centered on Find, BasicBlock, Inst, Use.
- CN: 这一段延续了 Sink 的主体实现，围绕 Find, BasicBlock, Inst, Use 等局部辅助逻辑展开。

### Lines 154-181

```cpp
  // If we couldn't find a block to sink to, ignore this instruction.
  if (!SuccToSinkTo)
    return false;

  LLVM_DEBUG(dbgs() << "Sink" << *Inst << " (";
             Inst->getParent()->printAsOperand(dbgs(), false); dbgs() << " -> ";
             SuccToSinkTo->printAsOperand(dbgs(), false); dbgs() << ")\n");

  // The current location of Inst dominates all uses, thus it must dominate
  // SuccToSinkTo, which is on the IDom chain between the nearest common
  // dominator to all uses and the current location.
  assert(DT.dominates(BB, SuccToSinkTo) &&
         "SuccToSinkTo must be dominated by current Inst location!");

  // Move the instruction.
  Inst->moveBefore(SuccToSinkTo->getFirstInsertionPt());
  return true;
}

static bool ProcessBlock(BasicBlock &BB, DominatorTree &DT, LoopInfo &LI,
                         AAResults &AA) {
  // Don't bother sinking code out of unreachable blocks. In addition to being
  // unprofitable, it can also lead to infinite looping, because in an
  // unreachable loop there may be nowhere to stop.
  if (!DT.isReachableFromEntry(&BB)) return false;

  bool MadeChange = false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 182-213

```cpp
  // Walk the basic block bottom-up.  Remember if we saw a store.
  BasicBlock::iterator I = BB.end();
  --I;
  bool ProcessedBegin = false;
  SmallPtrSet<Instruction *, 8> Stores;
  do {
    Instruction *Inst = &*I; // The instruction to sink.

    // Predecrement I (if it's not begin) so that it isn't invalidated by
    // sinking.
    ProcessedBegin = I == BB.begin();
    if (!ProcessedBegin)
      --I;

    if (Inst->isDebugOrPseudoInst())
      continue;

    if (SinkInstruction(Inst, Stores, DT, LI, AA)) {
      ++NumSunk;
      MadeChange = true;
    }

    // If we just processed the first instruction in the block, we're done.
  } while (!ProcessedBegin);

  return MadeChange;
}

static bool iterativelySinkInstructions(Function &F, DominatorTree &DT,
                                        LoopInfo &LI, AAResults &AA) {
  bool MadeChange, EverMadeChange = false;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 214-239

```cpp
  do {
    MadeChange = false;
    LLVM_DEBUG(dbgs() << "Sinking iteration " << NumSinkIter << "\n");
    // Process all basic blocks.
    for (BasicBlock &I : F)
      MadeChange |= ProcessBlock(I, DT, LI, AA);
    EverMadeChange |= MadeChange;
    NumSinkIter++;
  } while (MadeChange);

  return EverMadeChange;
}

PreservedAnalyses SinkingPass::run(Function &F, FunctionAnalysisManager &AM) {
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
  auto &LI = AM.getResult<LoopAnalysis>(F);
  auto &AA = AM.getResult<AAManager>(F);

  if (!iterativelySinkInstructions(F, DT, LI, AA))
    return PreservedAnalyses::all();

  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, DominatorTree.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAManager, DominatorTree 等分析结果。

### Lines 240-267

```cpp
namespace {
  class SinkingLegacyPass : public FunctionPass {
  public:
    static char ID; // Pass identification
    SinkingLegacyPass() : FunctionPass(ID) {
      initializeSinkingLegacyPassPass(*PassRegistry::getPassRegistry());
    }

    bool runOnFunction(Function &F) override {
      auto &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();
      auto &LI = getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
      auto &AA = getAnalysis<AAResultsWrapperPass>().getAAResults();

      return iterativelySinkInstructions(F, DT, LI, AA);
    }

    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.setPreservesCFG();
      FunctionPass::getAnalysisUsage(AU);
      AU.addRequired<AAResultsWrapperPass>();
      AU.addRequired<DominatorTreeWrapperPass>();
      AU.addRequired<LoopInfoWrapperPass>();
      AU.addPreserved<DominatorTreeWrapperPass>();
      AU.addPreserved<LoopInfoWrapperPass>();
    }
  };
} // end anonymous namespace

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include SinkingLegacyPass, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 SinkingLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo 等分析结果。

### Lines 268-275

```cpp
char SinkingLegacyPass::ID = 0;
INITIALIZE_PASS_BEGIN(SinkingLegacyPass, "sink", "Code sinking", false, false)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
INITIALIZE_PASS_END(SinkingLegacyPass, "sink", "Code sinking", false, false)

FunctionPass *llvm::createSinkingPass() { return new SinkingLegacyPass(); }
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `SinkingLegacyPass, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SinkingLegacyPass, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AAManager, AliasAnalysis, DominatorTree, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AAManager, AliasAnalysis, DominatorTree, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/LoopInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/LoopInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/Transforms/Scalar/Sink.h`, `llvm/Transforms/Scalar.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/Transforms/Scalar/Sink.h`, `llvm/Transforms/Scalar.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/Statistic.h`, `llvm/InitializePasses.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/Statistic.h`, `llvm/InitializePasses.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AAResults`, `AAManager`, `AliasAnalysis`, `DominatorTree`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AAManager`, `AliasAnalysis`, `DominatorTree`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
