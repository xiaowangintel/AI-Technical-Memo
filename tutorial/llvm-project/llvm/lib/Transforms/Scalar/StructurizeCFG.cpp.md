# StructurizeCFG.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/StructurizeCFG.cpp` | `llvm/lib/Transforms/Scalar/StructurizeCFG.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Structurize CFG within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 StructurizeCFG 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-96

```cpp
//===- StructurizeCFG.cpp -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/StructurizeCFG.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/EquivalenceClasses.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/SCCIterator.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/InstructionSimplify.h"
#include "llvm/Analysis/RegionInfo.h"
#include "llvm/Analysis/RegionIterator.h"
#include "llvm/Analysis/RegionPass.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

using BBSet = SmallPtrSet<BasicBlock *, 8>;

using PhiMap = MapVector<PHINode *, BBValueVector>;
using BB2BBVecMap = MapVector<BasicBlock *, BBVector>;

using BBPhiMap = DenseMap<BasicBlock *, PhiMap>;

using MaybeCondBranchWeights = std::optional<class CondBranchWeights>;

class CondBranchWeights {
  uint32_t TrueWeight;
  uint32_t FalseWeight;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 97-193

```cpp
  CondBranchWeights(uint32_t T, uint32_t F) : TrueWeight(T), FalseWeight(F) {}

public:
  static MaybeCondBranchWeights tryParse(const CondBrInst &Br) {
    uint64_t T, F;
    if (!extractBranchWeights(Br, T, F))
      return std::nullopt;

    return CondBranchWeights(T, F);
  }

  static void setMetadata(CondBrInst &Br,
                          const MaybeCondBranchWeights &Weights) {
    if (!Weights)
      return;
    uint32_t Arr[] = {Weights->TrueWeight, Weights->FalseWeight};
    setBranchWeights(Br, Arr, false);
  }

  CondBranchWeights invert() const {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return children(N).end();
  }
};

/// Finds the nearest common dominator of a set of BasicBlocks.
///
/// For every BB you add to the set, you can specify whether we "remember" the
/// block.  When you get the common dominator, you can also ask whether it's one
/// of the blocks we remembered.
class NearestCommonDominator {
  DominatorTree *DT;
  BasicBlock *Result = nullptr;
  bool ResultIsRemembered = false;

```
- EN: Core entities appearing here include tryParse, invert, PredInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 tryParse, invert, PredInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 194-291

```cpp
  /// Add BB to the resulting dominator.
  void addBlock(BasicBlock *BB, bool Remember) {
    if (!Result) {
      Result = BB;
      ResultIsRemembered = Remember;
      return;
    }

    BasicBlock *NewResult = DT->findNearestCommonDominator(Result, BB);
    if (NewResult != Result)
      ResultIsRemembered = false;
    if (NewResult == BB)
      ResultIsRemembered |= Remember;
    Result = NewResult;
  }

public:
  explicit NearestCommonDominator(DominatorTree *DomTree) : DT(DomTree) {}

  void addBlock(BasicBlock *BB) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  ConstantInt *BoolTrue;
  ConstantInt *BoolFalse;
  Value *BoolPoison;
  const TargetTransformInfo *TTI;
  Function *Func;
  Region *ParentRegion;

  UniformityInfo *UA = nullptr;
  DominatorTree *DT;

  SmallVector<RegionNode *, 8> Order;
  BBSet Visited;
  BBSet FlowSet;

```
- EN: Core entities appearing here include addBlock, addAndRememberBlock, StructurizeCFG, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo.
- CN: 此处出现的核心实体包括 addBlock, addAndRememberBlock, StructurizeCFG，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。

### Lines 292-386

```cpp
  SmallVector<WeakVH, 8> AffectedPhis;
  BBPhiMap DeletedPhis;
  BB2BBVecMap AddedPhis;

  PredMap Predicates;
  BranchVector Conditions;

  BB2BBMap Loops;
  PredMap LoopPreds;
  BranchVector LoopConds;

  Val2BBMap HoistedValues;

  RegionNode *PrevNode;

  void hoistZeroCostElseBlockPhiValues(BasicBlock *ElseBB, BasicBlock *ThenBB);

  bool isHoistableInstruction(Instruction *I, BasicBlock *BB,
                              BasicBlock *HoistTo);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

class StructurizeCFGLegacyPass : public RegionPass {
  bool SkipUniformRegions;

public:
  static char ID;

  explicit StructurizeCFGLegacyPass(bool SkipUniformRegions_ = false)
      : RegionPass(ID), SkipUniformRegions(SkipUniformRegions_) {
    if (ForceSkipUniformRegions.getNumOccurrences())
      SkipUniformRegions = ForceSkipUniformRegions.getValue();
    initializeStructurizeCFGLegacyPassPass(*PassRegistry::getPassRegistry());
  }

```
- EN: Core entities appearing here include StructurizeCFGLegacyPass, RegionPass, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 StructurizeCFGLegacyPass, RegionPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 387-476

```cpp
  bool runOnRegion(Region *R, RGPassManager &RGM) override {
    StructurizeCFG SCFG;
    SCFG.init(R);
    if (SkipUniformRegions) {
      UniformityInfo &UA =
          getAnalysis<UniformityInfoWrapperPass>().getUniformityInfo();
      if (SCFG.makeUniformRegion(R, UA))
        return false;
    }
    Function *F = R->getEntry()->getParent();
    const TargetTransformInfo *TTI =
        &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(*F);
    DominatorTree *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
    return SCFG.run(R, DT, TTI);
  }

  StringRef getPassName() const override { return "Structurize control flow"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    if (SkipUniformRegions)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// (via the path Then → Flow → Else), leading to unnecessary VGPR copies.
///
/// This function examines phi nodes whose incoming values are zero-cost
/// instructions in the Else block. It identifies such values that can be safely
/// hoisted and moves them to the nearest common dominator of Then and Else
/// blocks. A follow-up function after setting PhiNodes assigns the hoisted
/// value to poison phi nodes along the if→flow edge, aiding register coalescing
/// and minimizing unnecessary live ranges.
void StructurizeCFG::hoistZeroCostElseBlockPhiValues(BasicBlock *ElseBB,
                                                     BasicBlock *ThenBB) {

  BasicBlock *ElseSucc = ElseBB->getSingleSuccessor();
  BasicBlock *CommonDominator = DT->findNearestCommonDominator(ElseBB, ThenBB);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 477-572

```cpp
  if (!ElseSucc || !CommonDominator)
    return;
  // Only hoist in a simple if-else: ThenBB must branch directly to ElseSucc
  // and ElseSucc must have exactly 2 predecessors (ThenBB and ElseBB).
  // simplifyHoistedPhis assumes this exact shape; with additional predecessors
  // the hoisted value leaks into unrelated control-flow paths.
  if (ThenBB->getSingleSuccessor() != ElseSucc ||
      !ElseSucc->hasNPredecessors(2))
    return;
  Instruction *Term = CommonDominator->getTerminator();
  for (PHINode &Phi : ElseSucc->phis()) {
    Value *ElseVal = Phi.getIncomingValueForBlock(ElseBB);
    auto *Inst = dyn_cast<Instruction>(ElseVal);
    if (!Inst || !isHoistableInstruction(Inst, ElseBB, CommonDominator))
      continue;
    Inst->removeFromParent();
    Inst->insertInto(CommonDominator, Term->getIterator());
    HoistedValues[Inst] = CommonDominator;
  }
}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (Visited.count(Exit))
      Loops[Exit] = N->getEntry();

  } else {
    // Test for successors as back edge
    // TODO: support other terminators other than branches.
    BasicBlock *BB = N->getNodeAs<BasicBlock>();
    if (isa<UncondBrInst, CondBrInst>(BB->getTerminator()))
      for (BasicBlock *Succ : successors(BB))
        if (Visited.count(Succ))
          Loops[Succ] = BB;
  }
}

```
- EN: Core entities appearing here include orderNodes, analyzeLoops, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 orderNodes, analyzeLoops，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 573-668

```cpp
/// Build the condition for one edge
PredInfo StructurizeCFG::buildCondition(CondBrInst *Term, unsigned Idx,
                                        bool Invert) {
  Value *Cond = Term->getCondition();
  auto Weights = CondBranchWeights::tryParse(*Term);
  if (Idx != (unsigned)Invert) {
    Cond = invertCondition(Cond);
    if (Weights)
      Weights = Weights->invert();
  }
  return {Cond, Weights};
}

/// Analyze the predecessors of each block and build up predicates
void StructurizeCFG::gatherPredicates(RegionNode *N) {
  RegionInfo *RI = ParentRegion->getRegionInfo();
  BasicBlock *BB = N->getEntry();
  BBPredicates &Pred = Predicates[BB];
  BBPredicates &LPred = LoopPreds[BB];

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                      << (RN->isSubRegion() ? "SubRegion with entry: " : "")
                      << RN->getEntry()->getName() << "\n");

    // Analyze all the conditions leading to a node
    gatherPredicates(RN);

    // Remember that we've seen this node
    Visited.insert(RN->getEntry());

    // Find the last back edges
    analyzeLoops(RN);
  }
}

```
- EN: Core entities appearing here include gatherPredicates, collectInfos, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 gatherPredicates, collectInfos，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 669-756

```cpp
/// Insert the missing branch conditions
void StructurizeCFG::insertConditions(bool Loops, SSAUpdaterBulk &PhiInserter) {
  BranchVector &Conds = Loops ? LoopConds : Conditions;
  Value *Default = Loops ? BoolTrue : BoolFalse;

  for (CondBrInst *Term : Conds) {
    BasicBlock *Parent = Term->getParent();
    BasicBlock *SuccTrue = Term->getSuccessor(0);
    BasicBlock *SuccFalse = Term->getSuccessor(1);

    unsigned Variable = PhiInserter.AddVariable("", Boolean);
    PhiInserter.AddAvailableValue(Variable, Loops ? SuccFalse : Parent,
                                  Default);

    BBPredicates &Preds = Loops ? LoopPreds[SuccFalse] : Predicates[SuccTrue];

    NearestCommonDominator Dominator(DT);
    Dominator.addBlock(Parent);

    PredInfo ParentInfo{nullptr, std::nullopt};
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
    }
  }
}

/// Add a dummy PHI value as soon as we knew the new predecessor
void StructurizeCFG::addPhiValues(BasicBlock *From, BasicBlock *To) {
  for (PHINode &Phi : To->phis()) {
    Value *Poison = PoisonValue::get(Phi.getType());
    Phi.addIncoming(Poison, From);
  }
  AddedPhis[To].push_back(From);
}

```
- EN: Core entities appearing here include insertConditions, simplifyConditions, delPhiValues, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 insertConditions, simplifyConditions, delPhiValues，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 757-854

```cpp
/// When we are reconstructing a PHI inside \p PHIBlock with incoming values
/// from predecessors \p Incomings, we have a chance to mark the available value
/// from some blocks as undefined. The function will find out all such blocks
/// and return in \p UndefBlks.
void StructurizeCFG::findUndefBlocks(
    BasicBlock *PHIBlock, const SmallPtrSet<BasicBlock *, 8> &Incomings,
    SmallVector<BasicBlock *> &UndefBlks) const {
  //  We may get a post-structured CFG like below:
  //
  //  | P1
  //  |/
  //  F1
  //  |\
  //  | N
  //  |/
  //  F2
  //  |\
  //  | P2
  //  |/
  //  F3
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Mergeable.insert({BB, V});
  }

  // Update the incoming value of leaderA.
  IncomingA.assign(Mergeable.begin(), Mergeable.end());
  PhiClasses.unionSets(ItA, ItB);
}

/// Add the real PHI value as soon as everything is set up
void StructurizeCFG::setPhiValues() {
  SmallVector<PHINode *, 8> InsertedPhis;
  SSAUpdater Updater(&InsertedPhis);
  DenseMap<BasicBlock *, SmallVector<BasicBlock *>> UndefBlksMap;

```
- EN: Core entities appearing here include setPhiValues, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 setPhiValues，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 855-952

```cpp
  // Find phi nodes that have compatible incoming values (either they have
  // the same value for the same block or only one phi node has an incoming
  // value, see example below). We only search again the phi's that are
  // referenced by another phi, which is the case we care about.
  //
  // For example (-- means no incoming value):
  // phi1 : BB1:phi2   BB2:v  BB3:--
  // phi2:  BB1:--     BB2:v  BB3:w
  //
  // Then we can merge these incoming values and let phi1, phi2 use the
  // same set of incoming values:
  //
  // phi1&phi2: BB1:phi2  BB2:v  BB3:w
  //
  // By doing this, phi1 and phi2 would share more intermediate phi nodes.
  // This would help reduce the number of phi nodes during SSA reconstruction
  // and ultimately result in fewer COPY instructions.
  //
  // This should be correct, because if a phi node does not have incoming
  // value from certain block, this means the block is not the predecessor
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }

      for (auto UB : UndefBlks) {
        // If this undef block is dominated by any predecessor(before
        // structurization) of reconstructed PHI with constant incoming value,
        // don't mark the available value as undefined. Setting undef to such
        // block will stop us from getting optimal phi insertion.
        if (any_of(ConstantPreds,
                   [&](BasicBlock *CP) { return DT->dominates(CP, UB); }))
          continue;
        // Maybe already get a value through sharing with other phi nodes.
        if (Updater.HasValueForBlock(UB))
          continue;

```
- EN: This region continues the StructurizeCFG implementation with local helper logic centered on Find, For, BB1, BB2.
- CN: 这一段延续了 StructurizeCFG 的主体实现，围绕 Find, For, BB1, BB2 等局部辅助逻辑展开。

### Lines 953-1052

```cpp
        Updater.AddAvailableValue(UB, Poison);
      }

      for (BasicBlock *FI : From)
        Phi->setIncomingValueForBlock(FI, Updater.GetValueAtEndOfBlock(FI));
      AffectedPhis.push_back(Phi);
    }
  }

  AffectedPhis.append(InsertedPhis.begin(), InsertedPhis.end());
}

/// Updates PHI nodes after hoisted zero cost instructions by replacing poison
/// entries on Flow nodes with the appropriate hoisted values
void StructurizeCFG::simplifyHoistedPhis() {
  for (WeakVH VH : AffectedPhis) {
    PHINode *Phi = dyn_cast_or_null<PHINode>(VH);
    if (!Phi || Phi->getNumIncomingValues() != 2)
      continue;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// Let node exit(s) point to NewExit
void StructurizeCFG::changeExit(RegionNode *Node, BasicBlock *NewExit,
                                bool IncludeDominator) {
  if (Node->isSubRegion()) {
    Region *SubRegion = Node->getNodeAs<Region>();
    BasicBlock *OldExit = SubRegion->getExit();
    BasicBlock *Dominator = nullptr;

    // Find all the edges from the sub region to the exit.
    // We use make_early_inc_range here because we modify BB's terminator.
    for (BasicBlock *BB : llvm::make_early_inc_range(predecessors(OldExit))) {
      if (!SubRegion->contains(BB))
        continue;

```
- EN: Core entities appearing here include simplifyHoistedPhis, simplifyAffectedPhis, killTerminator, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 simplifyHoistedPhis, simplifyAffectedPhis, killTerminator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1053-1147

```cpp
      // Modify the edges to point to the new exit
      delPhiValues(BB, OldExit);
      BB->getTerminator()->replaceUsesOfWith(OldExit, NewExit);
      addPhiValues(BB, NewExit);

      // Find the new dominator (if requested)
      if (IncludeDominator) {
        if (!Dominator)
          Dominator = BB;
        else
          Dominator = DT->findNearestCommonDominator(Dominator, BB);
      }
    }

    // Change the dominator (if requested)
    if (Dominator)
      DT->changeImmediateDominator(NewExit, Dominator);

    // Update the region info
    SubRegion->replaceExit(NewExit);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

/// Does BB dominate all the predicates of Node?
bool StructurizeCFG::dominatesPredicates(BasicBlock *BB, RegionNode *Node) {
  BBPredicates &Preds = Predicates[Node->getEntry()];
  return llvm::all_of(Preds, [&](std::pair<BasicBlock *, PredInfo> Pred) {
    return DT->dominates(BB, Pred.first);
  });
}

/// Can we predict that this node will always be called?
bool StructurizeCFG::isPredictableTrue(RegionNode *Node) {
  BBPredicates &Preds = Predicates[Node->getEntry()];
  bool Dominated = false;

```
- EN: Core entities appearing here include needPrefix, setPrevNode, dominatesPredicates, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 needPrefix, setPrevNode, dominatesPredicates，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1148-1245

```cpp
  // Regionentry is always true
  if (!PrevNode)
    return true;

  for (auto [BB, PI] : Preds) {
    if (PI.Pred != BoolTrue)
      return false;

    if (!Dominated && DT->dominates(BB, PrevNode->getEntry()))
      Dominated = true;
  }

  // TODO: The dominator check is too strict
  return Dominated;
}

/// Take one node from the order vector and wire it up
void StructurizeCFG::wireFlow(bool ExitUseAllowed,
                              BasicBlock *LoopEnd) {
  RegionNode *Node = Order.pop_back_val();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

/// After this function control flow looks like it should be, but
/// branches and PHI nodes only have undefined conditions.
void StructurizeCFG::createFlow() {
  BasicBlock *Exit = ParentRegion->getExit();
  bool EntryDominatesExit = DT->dominates(ParentRegion->getEntry(), Exit);

  AffectedPhis.clear();
  DeletedPhis.clear();
  AddedPhis.clear();
  Conditions.clear();
  LoopConds.clear();

```
- EN: Core entities appearing here include dominatesPredicates, createFlow, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 dominatesPredicates, createFlow，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1246-1348

```cpp
  PrevNode = nullptr;
  Visited.clear();

  while (!Order.empty()) {
    handleLoops(EntryDominatesExit, nullptr);
  }

  if (PrevNode)
    changeExit(PrevNode, Exit, EntryDominatesExit);
  else
    assert(EntryDominatesExit);
}

/// Handle a rare case where the disintegrated nodes instructions
/// no longer dominate all their uses. Not sure if this is really necessary
void StructurizeCFG::rebuildSSA() {
  SSAUpdater Updater;
  for (BasicBlock *BB : ParentRegion->blocks())
    for (Instruction &I : *BB) {
      bool Initialized = false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        }
      }
    }
  }

  // Our region is uniform if:
  // 1. All conditional branches that are direct children are uniform (checked
  // above).
  // 2. And either:
  //   a. All sub-regions are uniform.
  //   b. There is one or less conditional branches among the direct children.
  return SubRegionsAreUniform || (ConditionalDirectChildren <= 1);
}

```
- EN: Core entities appearing here include rebuildSSA, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 rebuildSSA，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1349-1445

```cpp
void StructurizeCFG::init(Region *R) {
  LLVMContext &Context = R->getEntry()->getContext();

  Boolean = Type::getInt1Ty(Context);
  BoolTrue = ConstantInt::getTrue(Context);
  BoolFalse = ConstantInt::getFalse(Context);
  BoolPoison = PoisonValue::get(Boolean);

  this->UA = nullptr;
}

bool StructurizeCFG::makeUniformRegion(Region *R, UniformityInfo &UA) {
  if (R->isTopLevelRegion())
    return false;

  this->UA = &UA;

  // TODO: We could probably be smarter here with how we handle sub-regions.
  // We currently rely on the fact that metadata is set by earlier invocations
  // of the pass on sub-regions, and that this metadata doesn't get lost --
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Predicates.clear();
  Conditions.clear();
  Loops.clear();
  LoopPreds.clear();
  LoopConds.clear();
  FlowSet.clear();

  return true;
}

Pass *llvm::createStructurizeCFGPass(bool SkipUniformRegions) {
  return new StructurizeCFGLegacyPass(SkipUniformRegions);
}

```
- EN: Core entities appearing here include init, makeUniformRegion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 init, makeUniformRegion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1446-1498

```cpp
static void addRegionIntoQueue(Region &R, std::vector<Region *> &Regions) {
  Regions.push_back(&R);
  for (const auto &E : R)
    addRegionIntoQueue(*E, Regions);
}

StructurizeCFGPass::StructurizeCFGPass(bool SkipUniformRegions_)
    : SkipUniformRegions(SkipUniformRegions_) {
  if (ForceSkipUniformRegions.getNumOccurrences())
    SkipUniformRegions = ForceSkipUniformRegions.getValue();
}

void StructurizeCFGPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<StructurizeCFGPass> *>(this)->printPipeline(
      OS, MapClassName2PassName);
  if (SkipUniformRegions)
    OS << "<skip-uniform-regions>";
}

PreservedAnalyses StructurizeCFGPass::run(Function &F,
                                          FunctionAnalysisManager &AM) {

  bool Changed = false;
  DominatorTree *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  auto &RI = AM.getResult<RegionInfoAnalysis>(F);
  TargetTransformInfo *TTI = &AM.getResult<TargetIRAnalysis>(F);
  UniformityInfo *UI = nullptr;
  if (SkipUniformRegions)
    UI = &AM.getResult<UniformityInfoAnalysis>(F);

  std::vector<Region *> Regions;
  addRegionIntoQueue(*RI.getTopLevelRegion(), Regions);
  while (!Regions.empty()) {
    Region *R = Regions.back();
    Regions.pop_back();

    StructurizeCFG SCFG;
    SCFG.init(R);

    if (SkipUniformRegions && SCFG.makeUniformRegion(R, *UI)) {
      Changed = true; // May have added metadata.
      continue;
    }

    Changed |= SCFG.run(R, DT, TTI);
  }
  if (!Changed)
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
```
- EN: Core entities appearing here include addRegionIntoQueue, SkipUniformRegions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo.
- CN: 此处出现的核心实体包括 addRegionIntoQueue, SkipUniformRegions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `CondBranchWeights, PredInfo, SubGraphTraits, WrappedSuccIterator, NearestCommonDominator, StructurizeCFG, StructurizeCFGLegacyPass, tryParse` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`CondBranchWeights, PredInfo, SubGraphTraits, WrappedSuccIterator, NearestCommonDominator, StructurizeCFG, StructurizeCFGLegacyPass, tryParse` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `ForceSkipUniformRegions, RelaxedUniformRegions` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `ForceSkipUniformRegions, RelaxedUniformRegions` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Analysis/RegionIterator.h`, `llvm/Analysis/RegionPass.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/UniformityAnalysis.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Analysis/RegionIterator.h`, `llvm/Analysis/RegionPass.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/UniformityAnalysis.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Metadata.h`, `llvm/IR/PassManager.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Metadata.h`, `llvm/IR/PassManager.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/EquivalenceClasses.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/SCCIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/EquivalenceClasses.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/SCCIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
