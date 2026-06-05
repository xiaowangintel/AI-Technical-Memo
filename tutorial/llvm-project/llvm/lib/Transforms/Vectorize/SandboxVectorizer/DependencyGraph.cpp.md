# DependencyGraph.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/DependencyGraph.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/DependencyGraph.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Dependency Graph within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 DependencyGraph 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-67

```cpp
//===- DependencyGraph.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/DependencyGraph.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/SandboxIR/Instruction.h"
#include "llvm/SandboxIR/Utils.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h"

namespace llvm::sandboxir {

User::op_iterator PredIterator::skipBadIt(User::op_iterator OpIt,
                                          User::op_iterator OpItE,
                                          const DependencyGraph &DAG) {
  auto Skip = [&DAG](auto OpIt) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // It's a MemDGNode, so if we are not at the end of the use-def iterator we
  // need to first increment that.
  if (OpIt != OpItE) {
    ++OpIt;
    // Skip operands that are not instructions or are outside the DAG.
    OpIt = PredIterator::skipBadIt(OpIt, OpItE, *DAG);
    return *this;
  }
  // It's a MemDGNode with OpIt == end, so we need to increment MemIt.
  assert(MemIt != cast<MemDGNode>(N)->MemPreds.end() && "Already at end!");
  ++MemIt;
  return *this;
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 68-127

```cpp
bool PredIterator::operator==(const PredIterator &Other) const {
  assert(DAG == Other.DAG && "Iterators of different DAGs!");
  assert(N == Other.N && "Iterators of different nodes!");
  return OpIt == Other.OpIt && MemIt == Other.MemIt;
}

void DGNode::setSchedBundle(SchedBundle &SB) {
  if (this->SB != nullptr)
    this->SB->eraseFromBundle(this);
  this->SB = &SB;
}

DGNode::~DGNode() {
  if (SB == nullptr)
    return;
  SB->eraseFromBundle(this);
}

#ifndef NDEBUG
void DGNode::print(raw_ostream &OS, bool PrintDeps) const {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

MemDGNode *
MemDGNodeIntervalBuilder::getBotMemDGNode(const Interval<Instruction> &Intvl,
                                          const DependencyGraph &DAG) {
  Instruction *I = Intvl.bottom();
  Instruction *AfterI = Intvl.top();
  // Walk up the chain looking for a mem-dep candidate instruction.
  while (!DGNode::isMemDepNodeCandidate(I) && I != AfterI)
    I = I->getPrevNode();
  if (!DGNode::isMemDepNodeCandidate(I))
    return nullptr;
  return cast<MemDGNode>(DAG.getNode(I));
}

```
- EN: Core entities appearing here include setSchedBundle, ~DGNode, print, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 setSchedBundle, ~DGNode, print，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 128-180

```cpp
Interval<MemDGNode>
MemDGNodeIntervalBuilder::make(const Interval<Instruction> &Instrs,
                               DependencyGraph &DAG) {
  if (Instrs.empty())
    return {};
  auto *TopMemN = getTopMemDGNode(Instrs, DAG);
  // If we couldn't find a mem node in range TopN - BotN then it's empty.
  if (TopMemN == nullptr)
    return {};
  auto *BotMemN = getBotMemDGNode(Instrs, DAG);
  assert(BotMemN != nullptr && "TopMemN should be null too!");
  // Now that we have the mem-dep nodes, create and return the range.
  return Interval<MemDGNode>(TopMemN, BotMemN);
}

DependencyGraph::DependencyType
DependencyGraph::getRoughDepType(Instruction *FromI, Instruction *ToI) {
  // TODO: Perhaps compile-time improvement by skipping if neither is mem?
  if (FromI->mayWriteToMemory()) {
    if (ToI->mayReadFromMemory())
      return DependencyType::ReadAfterWrite;
    if (ToI->mayWriteToMemory())
      return DependencyType::WriteAfterWrite;
  } else if (FromI->mayReadFromMemory()) {
    if (ToI->mayWriteToMemory())
      return DependencyType::WriteAfterRead;
  }
  if (isa<sandboxir::PHINode>(FromI) || isa<sandboxir::PHINode>(ToI))
    return DependencyType::Control;
  if (ToI->isTerminator())
    return DependencyType::Control;
  if (DGNode::isStackSaveOrRestoreIntrinsic(FromI) ||
      DGNode::isStackSaveOrRestoreIntrinsic(ToI))
    return DependencyType::Other;
  return DependencyType::None;
}

static bool isOrdered(Instruction *I) {
  auto IsOrdered = [](Instruction *I) {
    if (auto *LI = dyn_cast<LoadInst>(I))
      return !LI->isUnordered();
    if (auto *SI = dyn_cast<StoreInst>(I))
      return !SI->isUnordered();
    if (DGNode::isFenceLike(I))
      return true;
    return false;
  };
  bool Is = IsOrdered(I);
  assert((!Is || DGNode::isMemDepCandidate(I)) &&
         "An ordered instruction must be a MemDepCandidate!");
  return Is;
}

```
- EN: Core entities appearing here include getRoughDepType, isOrdered, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getRoughDepType, isOrdered，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 181-240

```cpp
bool DependencyGraph::alias(Instruction *SrcI, Instruction *DstI,
                            DependencyType DepType) {
  std::optional<MemoryLocation> DstLocOpt =
      Utils::memoryLocationGetOrNone(DstI);
  if (!DstLocOpt)
    return true;
  // Check aliasing.
  assert((SrcI->mayReadFromMemory() || SrcI->mayWriteToMemory()) &&
         "Expected a mem instr");
  // TODO: Check AABudget
  ModRefInfo SrcModRef =
      isOrdered(SrcI)
          ? ModRefInfo::ModRef
          : Utils::aliasAnalysisGetModRefInfo(*BatchAA, SrcI, *DstLocOpt);
  switch (DepType) {
  case DependencyType::ReadAfterWrite:
  case DependencyType::WriteAfterWrite:
    return isModSet(SrcModRef);
  case DependencyType::WriteAfterRead:
    return isRefSet(SrcModRef);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
void DependencyGraph::scanAndAddDeps(MemDGNode &DstN,
                                     const Interval<MemDGNode> &SrcScanRange) {
  assert(isa<MemDGNode>(DstN) &&
         "DstN is the mem dep destination, so it must be mem");
  Instruction *DstI = DstN.getInstruction();
  // Walk up the instruction chain from ScanRange bottom to top, looking for
  // memory instrs that may alias.
  for (MemDGNode &SrcN : reverse(SrcScanRange)) {
    Instruction *SrcI = SrcN.getInstruction();
    if (hasDep(SrcI, DstI))
      DstN.addMemPred(&SrcN);
  }
}

```
- EN: Core entities appearing here include hasDep, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 hasDep，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 241-299

```cpp
void DependencyGraph::setDefUseUnscheduledSuccs(
    const Interval<Instruction> &NewInterval) {
  // +---+
  // |   |  Def
  // |   |   |
  // |   |   v
  // |   |  Use
  // +---+
  // Set the intra-interval counters in NewInterval.
  for (Instruction &I : NewInterval) {
    for (Value *Op : I.operands()) {
      auto *OpI = dyn_cast<Instruction>(Op);
      if (OpI == nullptr)
        continue;
      // TODO: For now don't cross BBs.
      if (OpI->getParent() != I.getParent())
        continue;
      if (!NewInterval.contains(OpI))
        continue;
      auto *OpN = getNode(OpI);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (Value *Op : BotI.operands()) {
      auto *OpI = dyn_cast<Instruction>(Op);
      if (OpI == nullptr)
        continue;
      auto *OpN = getNode(OpI);
      if (OpN == nullptr)
        continue;
      if (!TopInterval.contains(OpI))
        continue;
      OpN->incrUnscheduledSuccs();
    }
  }
}

```
- EN: This region continues the DependencyGraph implementation with local helper logic centered on DependencyGraph, Interval, Instruction, NewInterval.
- CN: 这一段延续了 DependencyGraph 的主体实现，围绕 DependencyGraph, Interval, Instruction, NewInterval 等局部辅助逻辑展开。

### Lines 300-362

```cpp
void DependencyGraph::createNewNodes(const Interval<Instruction> &NewInterval) {
  // Create Nodes only for the new sections of the DAG.
  DGNode *LastN = getOrCreateNode(NewInterval.top());
  MemDGNode *LastMemN = dyn_cast<MemDGNode>(LastN);
  for (Instruction &I : drop_begin(NewInterval)) {
    auto *N = getOrCreateNode(&I);
    // Build the Mem node chain.
    if (auto *MemN = dyn_cast<MemDGNode>(N)) {
      MemN->setPrevNode(LastMemN);
      LastMemN = MemN;
    }
  }
  // Link new MemDGNode chain with the old one, if any.
  if (!DAGInterval.empty()) {
    bool NewIsAbove = NewInterval.comesBefore(DAGInterval);
    const auto &TopInterval = NewIsAbove ? NewInterval : DAGInterval;
    const auto &BotInterval = NewIsAbove ? DAGInterval : NewInterval;
    MemDGNode *LinkTopN =
        MemDGNodeIntervalBuilder::getBotMemDGNode(TopInterval, *this);
    MemDGNode *LinkBotN =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                               MemDGNode *SkipN) const {
  auto *I = N->getInstruction();
  for (auto *PrevI = IncludingN ? I : I->getPrevNode(); PrevI != nullptr;
       PrevI = PrevI->getPrevNode()) {
    auto *PrevN = getNodeOrNull(PrevI);
    if (PrevN == nullptr)
      return nullptr;
    auto *PrevMemN = dyn_cast<MemDGNode>(PrevN);
    if (PrevMemN != nullptr && PrevMemN != SkipN)
      return PrevMemN;
  }
  return nullptr;
}

```
- EN: Core entities appearing here include createNewNodes, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createNewNodes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 363-427

```cpp
MemDGNode *DependencyGraph::getMemDGNodeAfter(DGNode *N, bool IncludingN,
                                              MemDGNode *SkipN) const {
  auto *I = N->getInstruction();
  for (auto *NextI = IncludingN ? I : I->getNextNode(); NextI != nullptr;
       NextI = NextI->getNextNode()) {
    auto *NextN = getNodeOrNull(NextI);
    if (NextN == nullptr)
      return nullptr;
    auto *NextMemN = dyn_cast<MemDGNode>(NextN);
    if (NextMemN != nullptr && NextMemN != SkipN)
      return NextMemN;
  }
  return nullptr;
}

void DependencyGraph::notifyCreateInstr(Instruction *I) {
  if (Ctx->getTracker().getState() == Tracker::TrackerState::Reverting)
    // We don't maintain the DAG while reverting.
    return;
  // Nothing to do if the node is not in the focus range of the DAG.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }
  }
}

void DependencyGraph::notifyMoveInstr(Instruction *I, const BBIterator &To) {
  if (Ctx->getTracker().getState() == Tracker::TrackerState::Reverting)
    // We don't maintain the DAG while reverting.
    return;
  // NOTE: This function runs before `I` moves to its new destination.
  BasicBlock *BB = To.getNodeParent();
  assert(!(To != BB->end() && &*To == I->getNextNode()) &&
         !(To == BB->end() && std::next(I->getIterator()) == BB->end()) &&
         "Should not have been called if destination is same as origin.");

```
- EN: Core entities appearing here include notifyCreateInstr, notifyMoveInstr, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 notifyCreateInstr, notifyMoveInstr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 428-487

```cpp
  // TODO: We can only handle fully internal movements within DAGInterval or at
  // the borders, i.e., right before the top or right after the bottom.
  assert(To.getNodeParent() == I->getParent() &&
         "TODO: We don't support movement across BBs!");
  assert(
      (To == std::next(DAGInterval.bottom()->getIterator()) ||
       (To != BB->end() && std::next(To) == DAGInterval.top()->getIterator()) ||
       (To != BB->end() && DAGInterval.contains(&*To))) &&
      "TODO: To should be either within the DAGInterval or right "
      "before/after it.");

  // Make a copy of the DAGInterval before we update it.
  auto OrigDAGInterval = DAGInterval;

  // Maintain the DAGInterval.
  DAGInterval.notifyMoveInstr(I, To);

  // TODO: Perhaps check if this is legal by checking the dependencies?

  // Update the MemDGNode chain to reflect the instr movement if necessary.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // update the chain.
    DGNode *InsertAfterN = getNode(&*std::prev(To));
    MemN->setPrevNode(
        getMemDGNodeBefore(InsertAfterN, /*IncludingN=*/true, /*SkipN=*/MemN));
  } else {
    // We have a node to insert before, so update the chain.
    DGNode *BeforeToN = getNode(&*To);
    MemN->setPrevNode(
        getMemDGNodeBefore(BeforeToN, /*IncludingN=*/false, /*SkipN=*/MemN));
    MemN->setNextNode(
        getMemDGNodeAfter(BeforeToN, /*IncludingN=*/true, /*SkipN=*/MemN));
  }
}

```
- EN: This region continues the DependencyGraph implementation with local helper logic centered on TODO, DAGInterval, BBs, Make.
- CN: 这一段延续了 DependencyGraph 的主体实现，围绕 TODO, DAGInterval, BBs, Make 等局部辅助逻辑展开。

### Lines 488-545

```cpp
void DependencyGraph::notifyEraseInstr(Instruction *I) {
  if (Ctx->getTracker().getState() == Tracker::TrackerState::Reverting)
    // We don't maintain the DAG while reverting.
    return;
  auto *N = getNode(I);
  if (N == nullptr)
    // Early return if there is no DAG node for `I`.
    return;
  if (auto *MemN = dyn_cast<MemDGNode>(getNode(I))) {
    // Update the MemDGNode chain if this is a memory node.
    auto *PrevMemN = getMemDGNodeBefore(MemN, /*IncludingN=*/false);
    auto *NextMemN = getMemDGNodeAfter(MemN, /*IncludingN=*/false);
    if (PrevMemN != nullptr)
      PrevMemN->NextMemN = NextMemN;
    if (NextMemN != nullptr)
      NextMemN->PrevMemN = PrevMemN;

    // Drop the memory dependencies from both predecessors and successors.
    while (!MemN->memPreds().empty()) {
      auto *PredN = *MemN->memPreds().begin();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //  U.User     U.User
  auto *UserI = dyn_cast_or_null<Instruction>(U.getUser());
  if (UserI == nullptr)
    return;
  auto *UserN = getNode(UserI);
  if (UserN == nullptr)
    return;
  // If UserN is marked as scheduled then we should not update CrrSrcN' or
  // NewSrcN's unscheduled successors.
  if (UserN->scheduled())
    return;
  // Update the UnscheduledSuccs counter for both the current source and
  // NewSrc if needed.
  if (auto *CurrSrcI = dyn_cast<Instruction>(U.get())) {
```
- EN: Core entities appearing here include notifyEraseInstr, notifySetUse, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 notifyEraseInstr, notifySetUse，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 546-603

```cpp
    if (auto *CurrSrcN = getNode(CurrSrcI)) {
      // If CurrSrcN is scheduled there is no point in updating UnscheduleSuccs.
      if (!CurrSrcN->scheduled())
        CurrSrcN->decrUnscheduledSuccs();
    }
  }
  if (auto *NewSrcI = dyn_cast<Instruction>(NewSrc)) {
    if (auto *NewSrcN = getNode(NewSrcI)) {
      // If CurrSrcN is scheduled there is no point in updating UnscheduleSuccs.
      if (!NewSrcN->scheduled())
        NewSrcN->incrUnscheduledSuccs();
    }
  }
}

Interval<Instruction> DependencyGraph::extend(ArrayRef<Instruction *> Instrs) {
  if (Instrs.empty())
    return {};

  Interval<Instruction> InstrsInterval(Instrs);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        scanAndAddDeps(DstN, SrcRange);
      }
    }
  };
  auto MemDAGInterval = MemDGNodeIntervalBuilder::make(DAGInterval, *this);
  if (MemDAGInterval.empty()) {
    FullScan(NewInterval);
  }
  // 2. The new section is below the old section.
  // +---+       -
  // |   |       |
  // |Old| SrcN  |
  // |   |  |    |
  // +---+  |    | SrcRange
```
- EN: Core entities appearing here include extend, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 extend，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 604-664

```cpp
  // +---+  |    |             -
  // |   |  |    |             |
  // |New|  v    |             | DstRange
  // |   | DstN  -             |
  // |   |                     |
  // +---+                     -
  // We are scanning for deps with destination in NewInterval because the deps
  // in DAGInterval have already been computed. We consider sources in the whole
  // range including both NewInterval and DAGInterval until DstN, for each DstN.
  else if (DAGInterval.bottom()->comesBefore(NewInterval.top())) {
    auto DstRange = MemDGNodeIntervalBuilder::make(NewInterval, *this);
    auto SrcRangeFull = MemDAGInterval.getUnionInterval(DstRange);
    for (MemDGNode &DstN : DstRange) {
      auto SrcRange =
          Interval<MemDGNode>(SrcRangeFull.top(), DstN.getPrevNode());
      scanAndAddDeps(DstN, SrcRange);
    }
  }
  // 3. The new section is above the old section.
  else if (NewInterval.bottom()->comesBefore(DAGInterval.top())) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // consider sources from the NewInterval only, because all intra-DAGInterval
    // dependencies have already been created.
    auto DstRangeOld = MemDAGInterval;
    auto SrcRange = MemDGNodeIntervalBuilder::make(NewInterval, *this);
    for (MemDGNode &DstN : DstRangeOld)
      scanAndAddDeps(DstN, SrcRange);
  } else {
    llvm_unreachable("We don't expect extending in both directions!");
  }

  DAGInterval = Union;
  return NewInterval;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 665-686

```cpp
#ifndef NDEBUG
void DependencyGraph::print(raw_ostream &OS) const {
  // InstrToNodeMap is unordered so we need to create an ordered vector.
  SmallVector<DGNode *> Nodes;
  Nodes.reserve(InstrToNodeMap.size());
  for (const auto &Pair : InstrToNodeMap)
    Nodes.push_back(Pair.second.get());
  // Sort them based on which one comes first in the BB.
  sort(Nodes, [](DGNode *N1, DGNode *N2) {
    return N1->getInstruction()->comesBefore(N2->getInstruction());
  });
  for (auto *N : Nodes)
    N->print(OS, /*PrintDeps=*/true);
}

void DependencyGraph::dump() const {
  print(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

} // namespace llvm::sandboxir
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include print, sort, dump, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 print, sort, dump，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `setSchedBundle, ~DGNode, print, getRoughDepType, isOrdered, hasDep, createNewNodes, notifyCreateInstr` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`setSchedBundle, ~DGNode, print, getRoughDepType, isOrdered, hasDep, createNewNodes, notifyCreateInstr` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Vectorize/SandboxVectorizer/DependencyGraph.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Vectorize/SandboxVectorizer/DependencyGraph.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/ArrayRef.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Utils.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/ArrayRef.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Utils.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
