# Scheduler.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Scheduler.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Scheduler.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Scheduler within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 Scheduler 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-41

```cpp
//===- Scheduler.cpp ------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h"

namespace llvm::sandboxir {

// TODO: Check if we can cache top/bottom to reduce compile-time.
DGNode *SchedBundle::getTop() const {
  DGNode *TopN = Nodes.front();
  for (auto *N : drop_begin(Nodes)) {
    if (N->getInstruction()->comesBefore(TopN->getInstruction()))
      TopN = N;
  }
  return TopN;
}

DGNode *SchedBundle::getBot() const {
  DGNode *BotN = Nodes.front();
  for (auto *N : drop_begin(Nodes)) {
    if (BotN->getInstruction()->comesBefore(N->getInstruction()))
      BotN = N;
  }
  return BotN;
}

void SchedBundle::cluster(BasicBlock::iterator Where) {
  for (auto *N : Nodes) {
    auto *I = N->getInstruction();
    if (I->getIterator() == Where)
      ++Where; // Try to maintain bundle order.
    I->moveBefore(*Where.getNodeParent(), Where);
  }
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include cluster, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 cluster，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 42-77

```cpp
#ifndef NDEBUG
void SchedBundle::dump(raw_ostream &OS) const {
  for (auto *N : Nodes)
    OS << *N;
}

void SchedBundle::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

#ifndef NDEBUG
void ReadyListContainer::dump(raw_ostream &OS) const {
  auto ListCopy = List;
  while (!ListCopy.empty()) {
    OS << *ListCopy.top() << "\n";
    ListCopy.pop();
  }
}

void ReadyListContainer::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

void Scheduler::scheduleAndUpdateReadyList(SchedBundle &Bndl) {
  // Find where we should schedule the instructions.
  assert(ScheduleTopItOpt && "Should have been set by now!");
  auto Where = *ScheduleTopItOpt;
  // Move all instructions in `Bndl` to `Where`.
  Bndl.cluster(Where);
  // Update the last scheduled bundle.
  ScheduleTopItOpt = Bndl.getTop()->getInstruction()->getIterator();
  // Set nodes as "scheduled" and decrement the UnsceduledSuccs counter of all
```
- EN: Core entities appearing here include dump, scheduleAndUpdateReadyList, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 dump, scheduleAndUpdateReadyList，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 78-113

```cpp
  // dependency predecessors.
  for (DGNode *N : Bndl) {
    for (auto *DepN : N->preds(DAG)) {
      DepN->decrUnscheduledSuccs();
      if (DepN->ready() && !DepN->scheduled())
        ReadyList.insert(DepN);
    }
    N->setScheduled();
  }
}

void Scheduler::notifyCreateInstr(Instruction *I) {
  // The DAG notifier should have run by now.
  auto *N = DAG.getNode(I);
  // If there is no DAG node for `I` it means that this is out of scope for the
  // DAG and as such out of scope for the scheduler too, so nothing to do.
  if (N == nullptr)
    return;
  // If the instruction is inserted below the top-of-schedule then we mark it as
  // "scheduled".
  bool IsScheduled = ScheduleTopItOpt &&
                     *ScheduleTopItOpt != I->getParent()->end() &&
                     (*ScheduleTopItOpt.value()).comesBefore(I);
  if (IsScheduled)
    N->setScheduled();
  // If the new instruction is above the top of schedule we need to remove its
  // dependency predecessors from the ready list and increment their
  // `UnscheduledSuccs` counters.
  if (!IsScheduled) {
    for (auto *PredN : N->preds(DAG)) {
      ReadyList.remove(PredN);
      PredN->incrUnscheduledSuccs();
    }
  }
}

```
- EN: Core entities appearing here include notifyCreateInstr, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 notifyCreateInstr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 114-149

```cpp
SchedBundle *Scheduler::createBundle(ArrayRef<Instruction *> Instrs) {
  SchedBundle::ContainerTy Nodes;
  Nodes.reserve(Instrs.size());
  for (auto *I : Instrs)
    Nodes.push_back(DAG.getNode(I));
  auto BndlPtr = std::make_unique<SchedBundle>(std::move(Nodes));
  auto *Bndl = BndlPtr.get();
  Bndls[Bndl] = std::move(BndlPtr);
  return Bndl;
}

void Scheduler::eraseBundle(SchedBundle *SB) { Bndls.erase(SB); }

bool Scheduler::tryScheduleUntil(ArrayRef<Instruction *> Instrs) {
  // Create a bundle for Instrs. If it turns out the schedule is infeasible we
  // will dismantle it.
  auto *InstrsSB = createBundle(Instrs);
  // Keep scheduling ready nodes until we either run out of ready nodes (i.e.,
  // ReadyList is empty), or all nodes that correspond to `Instrs` (the nodes of
  // which are collected in DeferredNodes) are all ready to schedule.
  SmallVector<DGNode *> Retry;
  bool KeepScheduling = true;
  while (KeepScheduling) {
    enum class TryScheduleRes {
      Success,  ///> We successfully scheduled the bundle.
      Failure,  ///> We failed to schedule the bundle.
      Finished, ///> We successfully scheduled the bundle and it is the last
                /// bundle to be scheduled.
    };
    /// TryScheduleNode() attempts to schedule all DAG nodes in the bundle that
    /// ReadyN is in. If it's not in a bundle it will create a singleton bundle
    /// and will try to schedule it.
    auto TryScheduleBndl = [this, InstrsSB](DGNode *ReadyN) -> TryScheduleRes {
      auto *SB = ReadyN->getSchedBundle();
      if (SB == nullptr) {
        // If ReadyN does not belong to a bundle, create a singleton bundle
```
- EN: Core entities appearing here include tryScheduleUntil, TryScheduleRes, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 tryScheduleUntil, TryScheduleRes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 150-185

```cpp
        // and schedule it.
        auto *SingletonSB = createBundle({ReadyN->getInstruction()});
        scheduleAndUpdateReadyList(*SingletonSB);
        return TryScheduleRes::Success;
      }
      if (SB->ready()) {
        // Remove the rest of the bundle from the ready list.
        // TODO: Perhaps change the Scheduler + ReadyList to operate on
        // SchedBundles instead of DGNodes.
        for (auto *N : *SB) {
          if (N != ReadyN)
            ReadyList.remove(N);
        }
        // If all nodes in the bundle are ready.
        scheduleAndUpdateReadyList(*SB);
        if (SB == InstrsSB)
          // We just scheduled InstrsSB bundle, so we are done scheduling.
          return TryScheduleRes::Finished;
        return TryScheduleRes::Success;
      }
      return TryScheduleRes::Failure;
    };
    while (!ReadyList.empty()) {
      auto *ReadyN = ReadyList.pop();
      auto Res = TryScheduleBndl(ReadyN);
      switch (Res) {
      case TryScheduleRes::Success:
        // We successfully scheduled ReadyN, keep scheduling.
        continue;
      case TryScheduleRes::Failure:
        // We failed to schedule ReadyN, defer it to later and keep scheduling
        // other ready instructions.
        Retry.push_back(ReadyN);
        continue;
      case TryScheduleRes::Finished:
        // We successfully scheduled the instruction bundle, so we are done.
```
- EN: This region continues the Scheduler implementation with local helper logic centered on SingletonSB, ReadyN, TryScheduleRes, Success.
- CN: 这一段延续了 Scheduler 的主体实现，围绕 SingletonSB, ReadyN, TryScheduleRes, Success 等局部辅助逻辑展开。

### Lines 186-224

```cpp
        return true;
      }
      llvm_unreachable("Unhandled TrySchedule() result");
    }
    // Try to schedule nodes from the Retry list.
    KeepScheduling = false;
    for (auto *N : make_early_inc_range(Retry)) {
      auto Res = TryScheduleBndl(N);
      if (Res == TryScheduleRes::Success) {
        Retry.erase(find(Retry, N));
        KeepScheduling = true;
      }
    }
  }

  eraseBundle(InstrsSB);
  return false;
}

Scheduler::BndlSchedState
Scheduler::getBndlSchedState(ArrayRef<Instruction *> Instrs) const {
  assert(!Instrs.empty() && "Expected non-empty bundle");
  auto *N0 = DAG.getNode(Instrs[0]);
  auto *SB0 = N0 != nullptr ? N0->getSchedBundle() : nullptr;
  bool AllUnscheduled = SB0 == nullptr;
  bool FullyScheduled = SB0 != nullptr && !SB0->isSingleton();
  for (auto *I : drop_begin(Instrs)) {
    auto *N = DAG.getNode(I);
    auto *SB = N != nullptr ? N->getSchedBundle() : nullptr;
    if (SB != nullptr) {
      // We found a scheduled instr, so there is now way all are unscheduled.
      AllUnscheduled = false;
      if (SB->isSingleton()) {
        // We found an instruction in a temporarily scheduled singleton. There
        // is no way that all instructions are scheduled in the same bundle.
        FullyScheduled = false;
      }
    }

```
- EN: Core entities appearing here include getBndlSchedState, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getBndlSchedState，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 225-260

```cpp
    if (SB != SB0) {
      // Either one of SB, SB0 is null, or they are in different bundles, so
      // Instrs are definitely not in the same vector bundle.
      FullyScheduled = false;
      // One of SB, SB0 are in a vector bundle and they differ.
      if ((SB != nullptr && !SB->isSingleton()) ||
          (SB0 != nullptr && !SB0->isSingleton()))
        return BndlSchedState::AlreadyScheduled;
    }
  }
  return AllUnscheduled   ? BndlSchedState::NoneScheduled
         : FullyScheduled ? BndlSchedState::FullyScheduled
                          : BndlSchedState::TemporarilyScheduled;
}

void Scheduler::trimSchedule(ArrayRef<Instruction *> Instrs) {
  //                                |  Legend: N: DGNode
  //  N <- DAGInterval.top()        |          B: SchedBundle
  //  N                             |          *: Contains instruction in Instrs
  //  B <- TopI (Top of schedule)   +-------------------------------------------
  //  B
  //  B *
  //  B
  //  B * <- LowestI (Lowest in Instrs)
  //  B
  //  N
  //  N
  //  N <- DAGInterval.bottom()
  //
  Instruction *TopI = &*ScheduleTopItOpt.value();
  Instruction *LowestI = VecUtils::getLowest(Instrs);
  // Destroy the singleton schedule bundles from LowestI all the way to the top.
  for (auto *I = LowestI, *E = TopI->getPrevNode(); I != E;
       I = I->getPrevNode()) {
    auto *N = DAG.getNode(I);
    if (N == nullptr)
```
- EN: Core entities appearing here include trimSchedule, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 trimSchedule，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 261-296

```cpp
      continue;
    auto *SB = N->getSchedBundle();
    if (SB->isSingleton())
      eraseBundle(SB);
  }
  // The DAG Nodes contain state like the number of UnscheduledSuccs and the
  // Scheduled flag. We need to reset their state. We need to do this for all
  // nodes from LowestI to the top of the schedule. DAG Nodes that are above the
  // top of schedule that depend on nodes that got reset need to have their
  // UnscheduledSuccs adjusted.
  Interval<Instruction> ResetIntvl(TopI, LowestI);
  for (Instruction &I : ResetIntvl) {
    auto *N = DAG.getNode(&I);
    N->resetScheduleState();
    // Recompute UnscheduledSuccs for nodes not only in ResetIntvl but even for
    // nodes above the top of schedule.
    for (auto *PredN : N->preds(DAG))
      PredN->incrUnscheduledSuccs();
  }
  // Refill the ready list by visiting all nodes from the top of DAG to LowestI.
  ReadyList.clear();
  Interval<Instruction> RefillIntvl(DAG.getInterval().top(), LowestI);
  for (Instruction &I : RefillIntvl) {
    auto *N = DAG.getNode(&I);
    if (N->ready())
      ReadyList.insert(N);
  }
}

bool Scheduler::trySchedule(ArrayRef<Instruction *> Instrs) {
  assert(all_of(drop_begin(Instrs),
                [Instrs](Instruction *I) {
                  return I->getParent() == (*Instrs.begin())->getParent();
                }) &&
         "Instrs not in the same BB, should have been rejected by Legality!");
  // TODO: For now don't cross BBs.
```
- EN: Core entities appearing here include trySchedule, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 trySchedule，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 297-332

```cpp
  if (!DAG.getInterval().empty()) {
    auto *BB = DAG.getInterval().top()->getParent();
    if (any_of(Instrs, [BB](auto *I) { return I->getParent() != BB; }))
      return false;
  }
  if (ScheduledBB == nullptr)
    ScheduledBB = Instrs[0]->getParent();
  // We don't support crossing BBs for now.
  if (any_of(Instrs,
             [this](Instruction *I) { return I->getParent() != ScheduledBB; }))
    return false;
  auto SchedState = getBndlSchedState(Instrs);
  switch (SchedState) {
  case BndlSchedState::FullyScheduled:
    // Nothing to do.
    return true;
  case BndlSchedState::AlreadyScheduled:
    // Instructions are part of a different vector schedule, so we can't
    // schedule \p Instrs in the same bundle (without destroying the existing
    // schedule).
    return false;
  case BndlSchedState::TemporarilyScheduled:
    // If one or more instrs are already scheduled we need to destroy the
    // top-most part of the schedule that includes the instrs in the bundle and
    // re-schedule.
    DAG.extend(Instrs);
    trimSchedule(Instrs);
    ScheduleTopItOpt = std::next(VecUtils::getLowest(Instrs)->getIterator());
    return tryScheduleUntil(Instrs);
  case BndlSchedState::NoneScheduled: {
    // TODO: Set the window of the DAG that we are interested in.
    if (!ScheduleTopItOpt)
      // We start scheduling at the bottom instr of Instrs.
      ScheduleTopItOpt = std::next(VecUtils::getLowest(Instrs)->getIterator());
    // Extend the DAG to include Instrs.
    Interval<Instruction> Extension = DAG.extend(Instrs);
```
- EN: This region continues the Scheduler implementation with local helper logic centered on DAG, Instrs, ScheduledBB, BBs.
- CN: 这一段延续了 Scheduler 的主体实现，围绕 DAG, Instrs, ScheduledBB, BBs 等局部辅助逻辑展开。

### Lines 333-360

```cpp
    // Add nodes to ready list.
    for (auto &I : Extension) {
      auto *N = DAG.getNode(&I);
      if (N->ready())
        ReadyList.insert(N);
    }
    // Try schedule all nodes until we can schedule Instrs back-to-back.
    return tryScheduleUntil(Instrs);
  }
  }
  llvm_unreachable("Unhandled BndlSchedState enum");
}

#ifndef NDEBUG
void Scheduler::dump(raw_ostream &OS) const {
  OS << "ReadyList:\n";
  ReadyList.dump(OS);
  OS << "Top of schedule: ";
  if (ScheduleTopItOpt)
    OS << **ScheduleTopItOpt;
  else
    OS << "Empty";
  OS << "\n";
}
void Scheduler::dump() const { dump(dbgs()); }
#endif // NDEBUG

} // namespace llvm::sandboxir
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include dump, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 dump，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `TryScheduleRes, cluster, dump, scheduleAndUpdateReadyList, notifyCreateInstr, tryScheduleUntil, getBndlSchedState, trimSchedule` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`TryScheduleRes, cluster, dump, scheduleAndUpdateReadyList, notifyCreateInstr, tryScheduleUntil, getBndlSchedState, trimSchedule` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Vectorize/SandboxVectorizer/Scheduler.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
