# FlattenCFG.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/FlattenCFG.cpp` | `llvm/lib/Transforms/Utils/FlattenCFG.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements ===- FlatternCFG.cpp - Code to perform CFG flattening === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 FlattenCFG 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-46

```cpp
//===- FlatternCFG.cpp - Code to perform CFG flattening -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Reduce conditional branches in CFG.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Value.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include <cassert>

using namespace llvm;

#define DEBUG_TYPE "flatten-cfg"

namespace {

class FlattenCFGOpt {
  AliasAnalysis *AA;

  /// Use parallel-and or parallel-or to generate conditions for
  /// conditional branches.
  bool FlattenParallelAndOr(BasicBlock *BB, IRBuilder<> &Builder);

  /// If \param BB is the merge block of an if-region, attempt to merge
  /// the if-region with an adjacent if-region upstream if two if-regions
  /// contain identical instructions.
  bool MergeIfRegion(BasicBlock *BB, IRBuilder<> &Builder);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 47-90

```cpp
  /// Compare a pair of blocks: \p Block1 and \p Block2, which
  /// are from two if-regions, where \p Head2 is the entry block of the 2nd
  /// if-region.  \returns true if \p Block1 and \p Block2 contain identical
  /// instructions, and have no memory reference alias with \p Head2.
  /// This is used as a legality check for merging if-regions.
  bool CompareIfRegionBlock(BasicBlock *Block1, BasicBlock *Block2,
                            BasicBlock *Head2);

public:
  FlattenCFGOpt(AliasAnalysis *AA) : AA(AA) {}

  bool run(BasicBlock *BB);
};

} // end anonymous namespace

/// If \param [in] BB has more than one predecessor that is a conditional
/// branch, attempt to use parallel and/or for the branch condition. \returns
/// true on success.
///
/// Before:
///   ......
///   %cmp10 = fcmp une float %tmp1, %tmp2
///   br i1 %cmp10, label %if.then, label %lor.rhs
///
/// lor.rhs:
///   ......
///   %cmp11 = fcmp une float %tmp3, %tmp4
///   br i1 %cmp11, label %if.then, label %ifend
///
/// if.end:  // the merge block
///   ......
///
/// if.then: // has two predecessors, both of them contains conditional branch.
///   ......
///   br label %if.end;
///
/// After:
///  ......
///  %cmp10 = fcmp une float %tmp1, %tmp2
///  ......
///  %cmp11 = fcmp une float %tmp3, %tmp4
///  %cmp12 = or i1 %cmp10, %cmp11    // parallel-or mode.
///  br i1 %cmp12, label %if.then, label %ifend
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis 等分析结果。

### Lines 91-140

```cpp
///
///  if.end:
///    ......
///
///  if.then:
///    ......
///    br label %if.end;
///
///  Current implementation handles two cases.
///  Case 1: BB is on the else-path.
///
///          BB1
///        /     |
///       BB2    |
///      /   \   |
///     BB3   \  |     where, BB1, BB2 contain conditional branches.
///      \    |  /     BB3 contains unconditional branch.
///       \   | /      BB4 corresponds to BB which is also the merge.
///  BB => BB4
///
///
///  Corresponding source code:
///
///  if (a == b && c == d)
///    statement; // BB3
///
///  Case 2: BB is on the then-path.
///
///             BB1
///          /      |
///         |      BB2
///         \    /    |  where BB1, BB2 contain conditional branches.
///  BB =>   BB3      |  BB3 contains unconditiona branch and corresponds
///           \     /    to BB.  BB4 is the merge.
///             BB4
///
///  Corresponding source code:
///
///  if (a == b || c == d)
///    statement;  // BB3
///
///  In both cases, BB is the common successor of conditional branches.
///  In Case 1, BB (BB4) has an unconditional branch (BB3) as
///  its predecessor.  In Case 2, BB (BB3) only has conditional branches
///  as its predecessors.
bool FlattenCFGOpt::FlattenParallelAndOr(BasicBlock *BB, IRBuilder<> &Builder) {
  PHINode *PHI = dyn_cast<PHINode>(BB->begin());
  if (PHI)
    return false; // For simplicity, avoid cases containing PHI nodes.

```
- EN: Core entities appearing here include FlattenParallelAndOr, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 FlattenParallelAndOr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 141-181

```cpp
  BasicBlock *LastCondBlock = nullptr;
  BasicBlock *FirstCondBlock = nullptr;
  BasicBlock *UnCondBlock = nullptr;
  int Idx = -1;

  // Check predecessors of \param BB.
  SmallPtrSet<BasicBlock *, 16> Preds(llvm::from_range, predecessors(BB));
  for (BasicBlock *Pred : Preds) {
    BasicBlock *PP = Pred->getSinglePredecessor();

    if (isa<UncondBrInst>(Pred->getTerminator())) {
      // Case 1: Pred (BB3) is an unconditional block, it should
      // have a single predecessor (BB2) that is also a predecessor
      // of \param BB (BB4) and should not have address-taken.
      // There should exist only one such unconditional
      // branch among the predecessors.
      if (UnCondBlock || !PP || !Preds.contains(PP) ||
          Pred->hasAddressTaken())
        return false;

      UnCondBlock = Pred;
      continue;
    }

    // Only conditional branches are allowed beyond this point.
    CondBrInst *PBI = dyn_cast<CondBrInst>(Pred->getTerminator());
    if (!PBI)
      return false;

    // Condition's unique use should be the branch instruction.
    Value *PC = PBI->getCondition();
    if (!PC || !PC->hasOneUse())
      return false;

    if (PP && Preds.count(PP)) {
      // These are internal condition blocks to be merged from, e.g.,
      // BB2 in both cases.
      // Should not be address-taken.
      if (Pred->hasAddressTaken())
        return false;

```
- EN: This region continues the FlattenCFG implementation with local helper logic centered on BasicBlock, LastCondBlock, FirstCondBlock, UnCondBlock.
- CN: 这一段延续了 FlattenCFG 的主体实现，围绕 BasicBlock, LastCondBlock, FirstCondBlock, UnCondBlock 等局部辅助逻辑展开。

### Lines 182-229

```cpp
      // Instructions in the internal condition blocks should be safe
      // to hoist up.
      for (BasicBlock::iterator BI = Pred->begin(), BE = PBI->getIterator();
           BI != BE;) {
        Instruction *CI = &*BI++;
        if (isa<PHINode>(CI) || !isSafeToSpeculativelyExecute(CI))
          return false;
      }
    } else {
      // This is the condition block to be merged into, e.g. BB1 in
      // both cases.
      if (FirstCondBlock)
        return false;
      FirstCondBlock = Pred;
    }

    // Find whether BB is uniformly on the true (or false) path
    // for all of its predecessors.
    BasicBlock *PS1 = PBI->getSuccessor(0);
    BasicBlock *PS2 = PBI->getSuccessor(1);
    BasicBlock *PS = (PS1 == BB) ? PS2 : PS1;
    int CIdx = (PS1 == BB) ? 0 : 1;

    if (Idx == -1)
      Idx = CIdx;
    else if (CIdx != Idx)
      return false;

    // PS is the successor which is not BB. Check successors to identify
    // the last conditional branch.
    if (!Preds.contains(PS)) {
      // Case 2.
      LastCondBlock = Pred;
    } else if (isa<UncondBrInst>(PS->getTerminator())) {
      // Case 1: PS(BB3) should be an unconditional branch.
      LastCondBlock = Pred;
    }
  }

  if (!FirstCondBlock || !LastCondBlock || (FirstCondBlock == LastCondBlock))
    return false;

  Instruction *TBB = LastCondBlock->getTerminator();
  BasicBlock *PS1 = TBB->getSuccessor(0);
  BasicBlock *PS2 = TBB->getSuccessor(1);
  UncondBrInst *PBI1 = dyn_cast<UncondBrInst>(PS1->getTerminator());
  UncondBrInst *PBI2 = dyn_cast<UncondBrInst>(PS2->getTerminator());

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 230-273

```cpp
  // If PS1 does not jump into PS2, but PS2 jumps into PS1,
  // attempt branch inversion.
  if (!PBI1 || (PS1->getTerminator()->getSuccessor(0) != PS2)) {
    // Check whether PS2 jumps into PS1.
    if (!PBI2 || (PS2->getTerminator()->getSuccessor(0) != PS1))
      return false;

    // Do branch inversion.
    BasicBlock *CurrBlock = LastCondBlock;
    bool EverChanged = false;
    for (; CurrBlock != FirstCondBlock;
         CurrBlock = CurrBlock->getSinglePredecessor()) {
      auto *BI = cast<CondBrInst>(CurrBlock->getTerminator());
      auto *CI = dyn_cast<CmpInst>(BI->getCondition());
      if (!CI)
        continue;

      CmpInst::Predicate Predicate = CI->getPredicate();
      // Canonicalize icmp_ne -> icmp_eq, fcmp_one -> fcmp_oeq
      if ((Predicate == CmpInst::ICMP_NE) || (Predicate == CmpInst::FCMP_ONE)) {
        CI->setPredicate(ICmpInst::getInversePredicate(Predicate));
        BI->swapSuccessors();
        EverChanged = true;
      }
    }
    return EverChanged;
  }

  // PS1 must have a conditional branch.
  if (!PBI1)
    return false;

  // PS2 should not contain PHI node.
  PHI = dyn_cast<PHINode>(PS2->begin());
  if (PHI)
    return false;

  // Do the transformation.
  BasicBlock *CB;
  CondBrInst *PBI = cast<CondBrInst>(FirstCondBlock->getTerminator());
  bool Iteration = true;
  IRBuilder<>::InsertPointGuard Guard(Builder);
  Value *PC = PBI->getCondition();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 274-321

```cpp
  do {
    CB = PBI->getSuccessor(1 - Idx);
    // Delete the conditional branch.
    FirstCondBlock->back().eraseFromParent();
    FirstCondBlock->splice(FirstCondBlock->end(), CB);
    PBI = cast<CondBrInst>(FirstCondBlock->getTerminator());
    Value *CC = PBI->getCondition();
    // Merge conditions.
    Builder.SetInsertPoint(PBI);
    Value *NC;
    if (Idx == 0)
      // Case 2, use parallel or.
      NC = Builder.CreateOr(PC, CC);
    else
      // Case 1, use parallel and.
      NC = Builder.CreateAnd(PC, CC);

    PBI->replaceUsesOfWith(CC, NC);
    PC = NC;
    if (CB == LastCondBlock)
      Iteration = false;
    // Remove internal conditional branches.
    CB->dropAllReferences();
    // make CB unreachable and let downstream to delete the block.
    new UnreachableInst(CB->getContext(), CB);
  } while (Iteration);

  LLVM_DEBUG(dbgs() << "Use parallel and/or in:\n" << *FirstCondBlock);
  return true;
}

/// Compare blocks from two if-regions, where \param Head2 is the entry of the
/// 2nd if-region. \param Block1 is a block in the 1st if-region to compare.
/// \param Block2 is a block in the 2nd if-region to compare.  \returns true if
/// Block1 and Block2 have identical instructions and do not have
/// memory reference alias with Head2.
bool FlattenCFGOpt::CompareIfRegionBlock(BasicBlock *Block1, BasicBlock *Block2,
                                         BasicBlock *Head2) {
  Instruction *PTI2 = Head2->getTerminator();
  Instruction *PBI2 = &Head2->front();

  // Check whether instructions in Block1 and Block2 are identical
  // and do not alias with instructions in Head2.
  BasicBlock::iterator iter1 = Block1->begin();
  BasicBlock::iterator end1 = Block1->getTerminator()->getIterator();
  BasicBlock::iterator iter2 = Block2->begin();
  BasicBlock::iterator end2 = Block2->getTerminator()->getIterator();

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 322-361

```cpp
  while (true) {
    if (iter1 == end1) {
      if (iter2 != end2)
        return false;
      break;
    }

    if (!iter1->isIdenticalTo(&*iter2))
      return false;

    // Illegal to remove instructions with side effects except
    // non-volatile stores.
    if (iter1->mayHaveSideEffects()) {
      Instruction *CurI = &*iter1;
      StoreInst *SI = dyn_cast<StoreInst>(CurI);
      if (!SI || SI->isVolatile())
        return false;
    }

    // For simplicity and speed, data dependency check can be
    // avoided if read from memory doesn't exist.
    if (iter1->mayReadFromMemory())
      return false;

    if (iter1->mayWriteToMemory()) {
      for (BasicBlock::iterator BI(PBI2), BE(PTI2); BI != BE; ++BI) {
        if (BI->mayReadFromMemory() || BI->mayWriteToMemory()) {
          // Check whether iter1 and BI may access the same memory location.
          if (!AA || AA->getModRefInfo(&*iter1, &*BI) != ModRefInfo::NoModRef)
            return false;
        }
      }
    }
    ++iter1;
    ++iter2;
  }

  return true;
}

```
- EN: This region continues the FlattenCFG implementation with local helper logic centered on Illegal, Instruction, CurI, StoreInst.
- CN: 这一段延续了 FlattenCFG 的主体实现，围绕 Illegal, Instruction, CurI, StoreInst 等局部辅助逻辑展开。

### Lines 362-411

```cpp
/// Check whether \param BB is the merge block of a if-region.  If yes, check
/// whether there exists an adjacent if-region upstream, the two if-regions
/// contain identical instructions and can be legally merged.  \returns true if
/// the two if-regions are merged.
///
/// From:
/// if (a)
///   statement;
/// if (b)
///   statement;
///
/// To:
/// if (a || b)
///   statement;
///
///
/// And from:
/// if (a)
///   ;
/// else
///   statement;
/// if (b)
///   ;
/// else
///   statement;
///
/// To:
/// if (a && b)
///   ;
/// else
///   statement;
///
/// We always take the form of the first if-region. This means that if the
/// statement in the first if-region, is in the "then-path", while in the second
/// if-region it is in the "else-path", then we convert the second to the first
/// form, by inverting the condition and the branch successors. The same
/// approach goes for the opposite case.
bool FlattenCFGOpt::MergeIfRegion(BasicBlock *BB, IRBuilder<> &Builder) {
  // We cannot merge the if-region if the merge point has phi nodes.
  if (isa<PHINode>(BB->front()))
    return false;

  BasicBlock *IfTrue2, *IfFalse2;
  CondBrInst *DomBI2 = GetIfCondition(BB, IfTrue2, IfFalse2);
  if (!DomBI2)
    return false;
  Instruction *CInst2 = dyn_cast<Instruction>(DomBI2->getCondition());
  if (!CInst2)
    return false;

```
- EN: Core entities appearing here include MergeIfRegion, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 MergeIfRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 412-457

```cpp
  BasicBlock *SecondEntryBlock = CInst2->getParent();
  if (SecondEntryBlock->hasAddressTaken())
    return false;

  BasicBlock *IfTrue1, *IfFalse1;
  CondBrInst *DomBI1 = GetIfCondition(SecondEntryBlock, IfTrue1, IfFalse1);
  if (!DomBI1)
    return false;
  Instruction *CInst1 = dyn_cast<Instruction>(DomBI1->getCondition());
  if (!CInst1)
    return false;

  BasicBlock *FirstEntryBlock = CInst1->getParent();
  // Don't die trying to process degenerate/unreachable code.
  if (FirstEntryBlock == SecondEntryBlock)
    return false;

  // Either then-path or else-path should be empty.
  bool InvertCond2 = false;
  BinaryOperator::BinaryOps CombineOp;
  if (IfFalse1 == FirstEntryBlock) {
    // The else-path is empty, so we must use "or" operation to combine the
    // conditions.
    CombineOp = BinaryOperator::Or;
    if (IfFalse2 != SecondEntryBlock) {
      if (IfTrue2 != SecondEntryBlock)
        return false;

      InvertCond2 = true;
      std::swap(IfTrue2, IfFalse2);
    }

    if (!CompareIfRegionBlock(IfTrue1, IfTrue2, SecondEntryBlock))
      return false;
  } else if (IfTrue1 == FirstEntryBlock) {
    // The then-path is empty, so we must use "and" operation to combine the
    // conditions.
    CombineOp = BinaryOperator::And;
    if (IfTrue2 != SecondEntryBlock) {
      if (IfFalse2 != SecondEntryBlock)
        return false;

      InvertCond2 = true;
      std::swap(IfTrue2, IfFalse2);
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 458-508

```cpp
    if (!CompareIfRegionBlock(IfFalse1, IfFalse2, SecondEntryBlock))
      return false;
  } else
    return false;

  Instruction *PTI2 = SecondEntryBlock->getTerminator();
  Instruction *PBI2 = &SecondEntryBlock->front();

  // Check whether \param SecondEntryBlock has side-effect and is safe to
  // speculate.
  for (BasicBlock::iterator BI(PBI2), BE(PTI2); BI != BE; ++BI) {
    Instruction *CI = &*BI;
    if (isa<PHINode>(CI) || CI->mayHaveSideEffects() ||
        !isSafeToSpeculativelyExecute(CI))
      return false;
  }

  // Merge \param SecondEntryBlock into \param FirstEntryBlock.
  FirstEntryBlock->back().eraseFromParent();
  FirstEntryBlock->splice(FirstEntryBlock->end(), SecondEntryBlock);
  CondBrInst *PBI = cast<CondBrInst>(FirstEntryBlock->getTerminator());
  assert(PBI->getCondition() == CInst2);
  BasicBlock *SaveInsertBB = Builder.GetInsertBlock();
  BasicBlock::iterator SaveInsertPt = Builder.GetInsertPoint();
  Builder.SetInsertPoint(PBI);
  if (InvertCond2) {
    InvertBranch(PBI, Builder);
  }
  Value *NC = Builder.CreateBinOp(CombineOp, CInst1, PBI->getCondition());
  PBI->replaceUsesOfWith(PBI->getCondition(), NC);
  Builder.SetInsertPoint(SaveInsertBB, SaveInsertPt);

  // Remove IfTrue1
  if (IfTrue1 != FirstEntryBlock) {
    IfTrue1->dropAllReferences();
    IfTrue1->eraseFromParent();
  }

  // Remove IfFalse1
  if (IfFalse1 != FirstEntryBlock) {
    IfFalse1->dropAllReferences();
    IfFalse1->eraseFromParent();
  }

  // Remove \param SecondEntryBlock
  SecondEntryBlock->dropAllReferences();
  SecondEntryBlock->eraseFromParent();
  LLVM_DEBUG(dbgs() << "If conditions merged into:\n" << *FirstEntryBlock);
  return true;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 509-525

```cpp
bool FlattenCFGOpt::run(BasicBlock *BB) {
  assert(BB && BB->getParent() && "Block not embedded in function!");
  assert(BB->getTerminator() && "Degenerate basic block encountered!");

  IRBuilder<> Builder(BB);

  if (FlattenParallelAndOr(BB, Builder) || MergeIfRegion(BB, Builder))
    return true;
  return false;
}

/// FlattenCFG - This function is used to flatten a CFG.  For
/// example, it uses parallel-and and parallel-or mode to collapse
/// if-conditions and merge if-regions with identical statements.
bool llvm::FlattenCFG(BasicBlock *BB, AAResults *AA) {
  return FlattenCFGOpt(AA).run(BB);
}
```
- EN: Core entities appearing here include run, FlattenCFG, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run, FlattenCFG，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `FlattenCFGOpt, FlattenParallelAndOr, MergeIfRegion, run, FlattenCFG` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`FlattenCFGOpt, FlattenParallelAndOr, MergeIfRegion, run, FlattenCFG` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AliasAnalysis, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AliasAnalysis, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallPtrSet.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/SmallPtrSet.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AliasAnalysis`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AliasAnalysis`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
