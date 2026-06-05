# LowerSwitch.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LowerSwitch.cpp` | `llvm/lib/Transforms/Utils/LowerSwitch.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements eliminate Switch instructions within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LowerSwitch 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-54

```cpp
//===- LowerSwitch.cpp - Eliminate Switch instructions --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The LowerSwitch transformation rewrites switch instructions with a sequence
// of branches, which allows targets to get away with not implementing the
// switch instruction until it is convenient.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LowerSwitch.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/AssumptionCache.h"
#include "llvm/Analysis/LazyValueInfo.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/CFG.h"
#include "llvm/IR/ConstantRange.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/PassManager.h"
#include "llvm/IR/Value.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/KnownBits.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Transforms/Utils.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include <cassert>
#include <iterator>
#include <vector>

using namespace llvm;

#define DEBUG_TYPE "lower-switch"

namespace {

struct IntRange {
  APInt Low, High;
};

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 55-105

```cpp
} // end anonymous namespace

namespace {
// Return true iff R is covered by Ranges.
bool IsInRanges(const IntRange &R, const std::vector<IntRange> &Ranges) {
  // Note: Ranges must be sorted, non-overlapping and non-adjacent.

  // Find the first range whose High field is >= R.High,
  // then check if the Low field is <= R.Low. If so, we
  // have a Range that covers R.
  auto I = llvm::lower_bound(
      Ranges, R, [](IntRange A, IntRange B) { return A.High.slt(B.High); });
  return I != Ranges.end() && I->Low.sle(R.Low);
}

struct CaseRange {
  ConstantInt *Low;
  ConstantInt *High;
  BasicBlock *BB;

  CaseRange(ConstantInt *low, ConstantInt *high, BasicBlock *bb)
      : Low(low), High(high), BB(bb) {}
};

using CaseVector = std::vector<CaseRange>;
using CaseItr = std::vector<CaseRange>::iterator;

/// The comparison function for sorting the switch case values in the vector.
/// WARNING: Case ranges should be disjoint!
struct CaseCmp {
  bool operator()(const CaseRange &C1, const CaseRange &C2) {
    const ConstantInt *CI1 = cast<const ConstantInt>(C1.Low);
    const ConstantInt *CI2 = cast<const ConstantInt>(C2.High);
    return CI1->getValue().slt(CI2->getValue());
  }
};

/// Used for debugging purposes.
LLVM_ATTRIBUTE_USED
raw_ostream &operator<<(raw_ostream &O, const CaseVector &C) {
  O << "[";

  for (CaseVector::const_iterator B = C.begin(), E = C.end(); B != E;) {
    O << "[" << B->Low->getValue() << ", " << B->High->getValue() << "]";
    if (++B != E)
      O << ", ";
  }

  return O << "]";
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include IsInRanges, CaseRange, CaseCmp, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 IsInRanges, CaseRange, CaseCmp，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 106-160

```cpp
/// Update the first occurrence of the "switch statement" BB in the PHI
/// node with the "new" BB. The other occurrences will:
///
/// 1) Be updated by subsequent calls to this function.  Switch statements may
/// have more than one outcoming edge into the same BB if they all have the same
/// value. When the switch statement is converted these incoming edges are now
/// coming from multiple BBs.
/// 2) Removed if subsequent incoming values now share the same case, i.e.,
/// multiple outcome edges are condensed into one. This is necessary to keep the
/// number of phi values equal to the number of branches to SuccBB.
void FixPhis(BasicBlock *SuccBB, BasicBlock *OrigBB, BasicBlock *NewBB,
             const APInt &NumMergedCases) {
  for (auto &I : SuccBB->phis()) {
    PHINode *PN = cast<PHINode>(&I);

    // Only update the first occurrence if NewBB exists.
    unsigned Idx = 0, E = PN->getNumIncomingValues();
    APInt LocalNumMergedCases = NumMergedCases;
    for (; Idx != E && NewBB; ++Idx) {
      if (PN->getIncomingBlock(Idx) == OrigBB) {
        PN->setIncomingBlock(Idx, NewBB);
        break;
      }
    }

    // Skip the updated incoming block so that it will not be removed.
    if (NewBB)
      ++Idx;

    // Remove additional occurrences coming from condensed cases and keep the
    // number of incoming values equal to the number of branches to SuccBB.
    SmallVector<unsigned, 8> Indices;
    for (; LocalNumMergedCases.ugt(0) && Idx < E; ++Idx)
      if (PN->getIncomingBlock(Idx) == OrigBB) {
        Indices.push_back(Idx);
        LocalNumMergedCases -= 1;
      }
    // Remove incoming values in the reverse order to prevent invalidating
    // *successive* index.
    for (unsigned III : llvm::reverse(Indices))
      PN->removeIncomingValue(III);
  }
}

/// Create a new leaf block for the binary lookup tree. It checks if the
/// switch's value == the case's value. If not, then it jumps to the default
/// branch. At this point in the tree, the value can't be another valid case
/// value, so the jump to the "default" branch is warranted.
BasicBlock *NewLeafBlock(CaseRange &Leaf, Value *Val, ConstantInt *LowerBound,
                         ConstantInt *UpperBound, BasicBlock *OrigBlock,
                         BasicBlock *Default) {
  Function *F = OrigBlock->getParent();
  BasicBlock *NewLeaf = BasicBlock::Create(Val->getContext(), "LeafBlock");
  F->insert(++OrigBlock->getIterator(), NewLeaf);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 161-212

```cpp
  // Emit comparison
  ICmpInst *Comp = nullptr;
  if (Leaf.Low == Leaf.High) {
    // Make the seteq instruction...
    Comp =
        new ICmpInst(NewLeaf, ICmpInst::ICMP_EQ, Val, Leaf.Low, "SwitchLeaf");
  } else {
    // Make range comparison
    if (Leaf.Low == LowerBound) {
      // Val >= Min && Val <= Hi --> Val <= Hi
      Comp = new ICmpInst(NewLeaf, ICmpInst::ICMP_SLE, Val, Leaf.High,
                          "SwitchLeaf");
    } else if (Leaf.High == UpperBound) {
      // Val <= Max && Val >= Lo --> Val >= Lo
      Comp = new ICmpInst(NewLeaf, ICmpInst::ICMP_SGE, Val, Leaf.Low,
                          "SwitchLeaf");
    } else if (Leaf.Low->isZero()) {
      // Val >= 0 && Val <= Hi --> Val <=u Hi
      Comp = new ICmpInst(NewLeaf, ICmpInst::ICMP_ULE, Val, Leaf.High,
                          "SwitchLeaf");
    } else {
      // Emit V-Lo <=u Hi-Lo
      Constant *NegLo = ConstantExpr::getNeg(Leaf.Low);
      Instruction *Add = BinaryOperator::CreateAdd(
          Val, NegLo, Val->getName() + ".off", NewLeaf);
      Constant *UpperBound = ConstantExpr::getAdd(NegLo, Leaf.High);
      Comp = new ICmpInst(NewLeaf, ICmpInst::ICMP_ULE, Add, UpperBound,
                          "SwitchLeaf");
    }
  }

  // Make the conditional branch...
  BasicBlock *Succ = Leaf.BB;
  CondBrInst::Create(Comp, Succ, Default, NewLeaf);

  // Update the PHI incoming value/block for the default.
  for (auto &I : Default->phis()) {
    PHINode *PN = cast<PHINode>(&I);
    auto *V = PN->getIncomingValueForBlock(OrigBlock);
    PN->addIncoming(V, NewLeaf);
  }

  // If there were any PHI nodes in this successor, rewrite one entry
  // from OrigBlock to come from NewLeaf.
  for (BasicBlock::iterator I = Succ->begin(); isa<PHINode>(I); ++I) {
    PHINode *PN = cast<PHINode>(I);
    // Remove all but one incoming entries from the cluster
    APInt Range = Leaf.High->getValue() - Leaf.Low->getValue();
    for (APInt j(Range.getBitWidth(), 0, false); j.ult(Range); ++j) {
      PN->removeIncomingValue(OrigBlock);
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 213-268

```cpp
    int BlockIdx = PN->getBasicBlockIndex(OrigBlock);
    assert(BlockIdx != -1 && "Switch didn't go to this successor??");
    PN->setIncomingBlock((unsigned)BlockIdx, NewLeaf);
  }

  return NewLeaf;
}

/// Convert the switch statement into a binary lookup of the case values.
/// The function recursively builds this tree. LowerBound and UpperBound are
/// used to keep track of the bounds for Val that have already been checked by
/// a block emitted by one of the previous calls to switchConvert in the call
/// stack.
BasicBlock *SwitchConvert(CaseItr Begin, CaseItr End, ConstantInt *LowerBound,
                          ConstantInt *UpperBound, Value *Val,
                          BasicBlock *Predecessor, BasicBlock *OrigBlock,
                          BasicBlock *Default,
                          const std::vector<IntRange> &UnreachableRanges) {
  assert(LowerBound && UpperBound && "Bounds must be initialized");
  unsigned Size = End - Begin;

  if (Size == 1) {
    // Check if the Case Range is perfectly squeezed in between
    // already checked Upper and Lower bounds. If it is then we can avoid
    // emitting the code that checks if the value actually falls in the range
    // because the bounds already tell us so.
    if (Begin->Low == LowerBound && Begin->High == UpperBound) {
      APInt NumMergedCases = UpperBound->getValue() - LowerBound->getValue();
      FixPhis(Begin->BB, OrigBlock, Predecessor, NumMergedCases);
      return Begin->BB;
    }
    return NewLeafBlock(*Begin, Val, LowerBound, UpperBound, OrigBlock,
                        Default);
  }

  unsigned Mid = Size / 2;
  std::vector<CaseRange> LHS(Begin, Begin + Mid);
  LLVM_DEBUG(dbgs() << "LHS: " << LHS << "\n");
  std::vector<CaseRange> RHS(Begin + Mid, End);
  LLVM_DEBUG(dbgs() << "RHS: " << RHS << "\n");

  CaseRange &Pivot = *(Begin + Mid);
  LLVM_DEBUG(dbgs() << "Pivot ==> [" << Pivot.Low->getValue() << ", "
                    << Pivot.High->getValue() << "]\n");

  // NewLowerBound here should never be the integer minimal value.
  // This is because it is computed from a case range that is never
  // the smallest, so there is always a case range that has at least
  // a smaller value.
  ConstantInt *NewLowerBound = Pivot.Low;

  // Because NewLowerBound is never the smallest representable integer
  // it is safe here to subtract one.
  ConstantInt *NewUpperBound = ConstantInt::get(NewLowerBound->getContext(),
                                                NewLowerBound->getValue() - 1);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 269-320

```cpp
  if (!UnreachableRanges.empty()) {
    // Check if the gap between LHS's highest and NewLowerBound is unreachable.
    APInt GapLow = LHS.back().High->getValue() + 1;
    APInt GapHigh = NewLowerBound->getValue() - 1;
    IntRange Gap = {GapLow, GapHigh};
    if (GapHigh.sge(GapLow) && IsInRanges(Gap, UnreachableRanges))
      NewUpperBound = LHS.back().High;
  }

  LLVM_DEBUG(dbgs() << "LHS Bounds ==> [" << LowerBound->getValue() << ", "
                    << NewUpperBound->getValue() << "]\n"
                    << "RHS Bounds ==> [" << NewLowerBound->getValue() << ", "
                    << UpperBound->getValue() << "]\n");

  // Create a new node that checks if the value is < pivot. Go to the
  // left branch if it is and right branch if not.
  Function *F = OrigBlock->getParent();
  BasicBlock *NewNode = BasicBlock::Create(Val->getContext(), "NodeBlock");

  ICmpInst *Comp = new ICmpInst(ICmpInst::ICMP_SLT, Val, Pivot.Low, "Pivot");

  BasicBlock *LBranch =
      SwitchConvert(LHS.begin(), LHS.end(), LowerBound, NewUpperBound, Val,
                    NewNode, OrigBlock, Default, UnreachableRanges);
  BasicBlock *RBranch =
      SwitchConvert(RHS.begin(), RHS.end(), NewLowerBound, UpperBound, Val,
                    NewNode, OrigBlock, Default, UnreachableRanges);

  F->insert(++OrigBlock->getIterator(), NewNode);
  Comp->insertInto(NewNode, NewNode->end());

  CondBrInst::Create(Comp, LBranch, RBranch, NewNode);
  return NewNode;
}

/// Transform simple list of \p SI's cases into list of CaseRange's \p Cases.
/// \post \p Cases wouldn't contain references to \p SI's default BB.
/// \returns Number of \p SI's cases that do not reference \p SI's default BB.
unsigned Clusterify(CaseVector &Cases, SwitchInst *SI) {
  unsigned NumSimpleCases = 0;

  // Start with "simple" cases
  for (auto Case : SI->cases()) {
    if (Case.getCaseSuccessor() == SI->getDefaultDest())
      continue;
    Cases.push_back(CaseRange(Case.getCaseValue(), Case.getCaseValue(),
                              Case.getCaseSuccessor()));
    ++NumSimpleCases;
  }

  llvm::sort(Cases, CaseCmp());

```
- EN: Core entities appearing here include Clusterify, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 Clusterify，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 321-377

```cpp
  // Merge case into clusters
  if (Cases.size() >= 2) {
    CaseItr I = Cases.begin();
    for (CaseItr J = std::next(I), E = Cases.end(); J != E; ++J) {
      const APInt &nextValue = J->Low->getValue();
      const APInt &currentValue = I->High->getValue();
      BasicBlock *nextBB = J->BB;
      BasicBlock *currentBB = I->BB;

      // If the two neighboring cases go to the same destination, merge them
      // into a single case.
      assert(nextValue.sgt(currentValue) &&
             "Cases should be strictly ascending");
      if ((nextValue == currentValue + 1) && (currentBB == nextBB)) {
        I->High = J->High;
        // FIXME: Combine branch weights.
      } else if (++I != J) {
        *I = *J;
      }
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // Prepare cases vector.
  CaseVector Cases;
  const unsigned NumSimpleCases = Clusterify(Cases, SI);
  IntegerType *IT = cast<IntegerType>(SI->getCondition()->getType());
  const unsigned BitWidth = IT->getBitWidth();
  // Explicitly use higher precision to prevent unsigned overflow where
  // `UnsignedMax - 0 + 1 == 0`
  APInt UnsignedZero(BitWidth + 1, 0);
  APInt UnsignedMax = APInt::getMaxValue(BitWidth);
  LLVM_DEBUG(dbgs() << "Clusterify finished. Total clusters: " << Cases.size()
                    << ". Total non-default cases: " << NumSimpleCases
                    << "\nCase clusters: " << Cases << "\n");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, LazyValueInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, LazyValueInfo 等分析结果。

### Lines 378-429

```cpp
  // If there is only the default destination, just branch.
  if (Cases.empty()) {
    UncondBrInst::Create(Default, OrigBlock);
    // Remove all the references from Default's PHIs to OrigBlock, but one.
    FixPhis(Default, OrigBlock, OrigBlock, UnsignedMax);
    SI->eraseFromParent();
    return;
  }

  ConstantInt *LowerBound = nullptr;
  ConstantInt *UpperBound = nullptr;
  bool DefaultIsUnreachableFromSwitch = false;

  if (SI->defaultDestUnreachable()) {
    // Make the bounds tightly fitted around the case value range, because we
    // know that the value passed to the switch must be exactly one of the case
    // values.
    LowerBound = Cases.front().Low;
    UpperBound = Cases.back().High;
    DefaultIsUnreachableFromSwitch = true;
  } else {
    // Constraining the range of the value being switched over helps eliminating
    // unreachable BBs and minimizing the number of `add` instructions
    // newLeafBlock ends up emitting. Running CorrelatedValuePropagation after
    // LowerSwitch isn't as good, and also much more expensive in terms of
    // compile time for the following reasons:
    // 1. it processes many kinds of instructions, not just switches;
    // 2. even if limited to icmp instructions only, it will have to process
    //    roughly C icmp's per switch, where C is the number of cases in the
    //    switch, while LowerSwitch only needs to call LVI once per switch.
    const DataLayout &DL = F->getDataLayout();
    KnownBits Known = computeKnownBits(Val, DL, AC, SI);
    // TODO Shouldn't this create a signed range?
    ConstantRange KnownBitsRange =
        ConstantRange::fromKnownBits(Known, /*IsSigned=*/false);
    const ConstantRange LVIRange =
        LVI->getConstantRange(Val, SI, /*UndefAllowed*/ false);
    ConstantRange ValRange = KnownBitsRange.intersectWith(LVIRange);
    // We delegate removal of unreachable non-default cases to other passes. In
    // the unlikely event that some of them survived, we just conservatively
    // maintain the invariant that all the cases lie between the bounds. This
    // may, however, still render the default case effectively unreachable.
    const APInt &Low = Cases.front().Low->getValue();
    const APInt &High = Cases.back().High->getValue();
    APInt Min = APIntOps::smin(ValRange.getSignedMin(), Low);
    APInt Max = APIntOps::smax(ValRange.getSignedMax(), High);

    LowerBound = ConstantInt::get(SI->getContext(), Min);
    UpperBound = ConstantInt::get(SI->getContext(), Max);
    DefaultIsUnreachableFromSwitch = (Min + (NumSimpleCases - 1) == Max);
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 430-486

```cpp
  std::vector<IntRange> UnreachableRanges;

  if (DefaultIsUnreachableFromSwitch) {
    DenseMap<BasicBlock *, APInt> Popularity;
    APInt MaxPop(UnsignedZero);
    BasicBlock *PopSucc = nullptr;

    APInt SignedMax = APInt::getSignedMaxValue(BitWidth);
    APInt SignedMin = APInt::getSignedMinValue(BitWidth);
    IntRange R = {SignedMin, SignedMax};
    UnreachableRanges.push_back(R);
    for (const auto &I : Cases) {
      const APInt &Low = I.Low->getValue();
      const APInt &High = I.High->getValue();

      IntRange &LastRange = UnreachableRanges.back();
      if (LastRange.Low.eq(Low)) {
        // There is nothing left of the previous range.
        UnreachableRanges.pop_back();
      } else {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      assert(I->Low.sle(I->High));
      auto Next = I + 1;
      if (Next != E) {
        assert(Next->Low.sgt(I->High));
      }
    }
#endif

    // As the default block in the switch is unreachable, update the PHI nodes
    // (remove all of the references to the default block) to reflect this.
    const unsigned NumDefaultEdges = SI->getNumCases() + 1 - NumSimpleCases;
    for (unsigned I = 0; I < NumDefaultEdges; ++I)
      Default->removePredecessor(OrigBlock);

```
- EN: This region continues the LowerSwitch implementation with local helper logic centered on IntRange, UnreachableRanges, DefaultIsUnreachableFromSwitch, DenseMap.
- CN: 这一段延续了 LowerSwitch 的主体实现，围绕 IntRange, UnreachableRanges, DefaultIsUnreachableFromSwitch, DenseMap 等局部辅助逻辑展开。

### Lines 487-538

```cpp
    // Use the most popular block as the new default, reducing the number of
    // cases.
    Default = PopSucc;
    llvm::erase_if(Cases,
                   [PopSucc](const CaseRange &R) { return R.BB == PopSucc; });

    // If there are no cases left, just branch.
    if (Cases.empty()) {
      UncondBrInst::Create(Default, OrigBlock);
      SI->eraseFromParent();
      // As all the cases have been replaced with a single branch, only keep
      // one entry in the PHI nodes.
      if (!MaxPop.isZero())
        for (APInt I(UnsignedZero); I.ult(MaxPop - 1); ++I)
          PopSucc->removePredecessor(OrigBlock);
      return;
    }

    // If the condition was a PHI node with the switch block as a predecessor
    // removing predecessors may have caused the condition to be erased.
    // Getting the condition value again here protects against that.
    Val = SI->getCondition();
  }

  BasicBlock *SwitchBlock =
      SwitchConvert(Cases.begin(), Cases.end(), LowerBound, UpperBound, Val,
                    OrigBlock, OrigBlock, Default, UnreachableRanges);

  // We have added incoming values for newly-created predecessors in
  // NewLeafBlock(). The only meaningful work we offload to FixPhis() is to
  // remove the incoming values from OrigBlock. There might be a special case
  // that SwitchBlock is the same as Default, under which the PHIs in Default
  // are fixed inside SwitchConvert().
  if (SwitchBlock != Default)
    FixPhis(Default, OrigBlock, nullptr, UnsignedMax);

  // Branch to our shiny new if-then stuff...
  UncondBrInst::Create(SwitchBlock, OrigBlock);

  // We are now done with the switch instruction, delete it.
  BasicBlock *OldDefault = SI->getDefaultDest();
  SI->eraseFromParent();

  // If the Default block has no more predecessors just add it to DeleteList.
  if (pred_empty(OldDefault))
    DeleteList.insert(OldDefault);
}

bool LowerSwitch(Function &F, LazyValueInfo *LVI, AssumptionCache *AC) {
  bool Changed = false;
  SmallPtrSet<BasicBlock *, 8> DeleteList;

```
- EN: Core entities appearing here include LowerSwitch, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, LazyValueInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 LowerSwitch，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, LazyValueInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 539-590

```cpp
  // We use make_early_inc_range here so that we don't traverse new blocks.
  for (BasicBlock &Cur : llvm::make_early_inc_range(F)) {
    // If the block is a dead Default block that will be deleted later, don't
    // waste time processing it.
    if (DeleteList.count(&Cur))
      continue;

    if (SwitchInst *SI = dyn_cast<SwitchInst>(Cur.getTerminator())) {
      Changed = true;
      ProcessSwitchInst(SI, DeleteList, AC, LVI);
    }
  }

  for (BasicBlock *BB : DeleteList) {
    LVI->eraseBlock(BB);
    DeleteDeadBlock(BB);
  }

  return Changed;
}

/// Replace all SwitchInst instructions with chained branch instructions.
class LowerSwitchLegacyPass : public FunctionPass {
public:
  // Pass identification, replacement for typeid
  static char ID;

  LowerSwitchLegacyPass() : FunctionPass(ID) {
    initializeLowerSwitchLegacyPassPass(*PassRegistry::getPassRegistry());
  }

  bool runOnFunction(Function &F) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LazyValueInfoWrapperPass>();
  }
};

} // end anonymous namespace

char LowerSwitchLegacyPass::ID = 0;

// Publicly exposed interface to pass...
char &llvm::LowerSwitchID = LowerSwitchLegacyPass::ID;

INITIALIZE_PASS_BEGIN(LowerSwitchLegacyPass, "lowerswitch",
                      "Lower SwitchInst's to branches", false, false)
INITIALIZE_PASS_DEPENDENCY(AssumptionCacheTracker)
INITIALIZE_PASS_DEPENDENCY(LazyValueInfoWrapperPass)
INITIALIZE_PASS_END(LowerSwitchLegacyPass, "lowerswitch",
                    "Lower SwitchInst's to branches", false, false)

```
- EN: Core entities appearing here include LowerSwitchLegacyPass, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, LazyValueInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 LowerSwitchLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, LazyValueInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 591-609

```cpp
// createLowerSwitchPass - Interface to this file...
FunctionPass *llvm::createLowerSwitchPass() {
  return new LowerSwitchLegacyPass();
}

bool LowerSwitchLegacyPass::runOnFunction(Function &F) {
  LazyValueInfo *LVI = &getAnalysis<LazyValueInfoWrapperPass>().getLVI();
  auto *ACT = getAnalysisIfAvailable<AssumptionCacheTracker>();
  AssumptionCache *AC = ACT ? &ACT->getAssumptionCache(F) : nullptr;
  return LowerSwitch(F, LVI, AC);
}

PreservedAnalyses LowerSwitchPass::run(Function &F,
                                       FunctionAnalysisManager &AM) {
  LazyValueInfo *LVI = &AM.getResult<LazyValueAnalysis>(F);
  AssumptionCache *AC = AM.getCachedResult<AssumptionAnalysis>(F);
  return LowerSwitch(F, LVI, AC) ? PreservedAnalyses::none()
                                 : PreservedAnalyses::all();
}
```
- EN: Core entities appearing here include runOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, LazyValueInfo.
- CN: 此处出现的核心实体包括 runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, LazyValueInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `IntRange, CaseRange, CaseCmp, LowerSwitchLegacyPass, IsInRanges, operator, Clusterify, LowerSwitch` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`IntRange, CaseRange, CaseCmp, LowerSwitchLegacyPass, IsInRanges, operator, Clusterify, LowerSwitch` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DataLayout, LazyValueInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DataLayout, LazyValueInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/LazyValueInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/LazyValueInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instructions.h`, `llvm/IR/PassManager.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/LowerSwitch.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/ConstantRange.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instructions.h`, `llvm/IR/PassManager.h`, `llvm/IR/Value.h`, `llvm/Transforms/Utils/LowerSwitch.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/KnownBits.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/Support/KnownBits.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `iterator`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `iterator`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DataLayout`, `LazyValueInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DataLayout`, `LazyValueInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
