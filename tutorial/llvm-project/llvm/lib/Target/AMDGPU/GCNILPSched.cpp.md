# GCNILPSched.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNILPSched.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNILPSched for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNILPSched 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===---------------------------- GCNILPSched.cpp - -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/ScheduleDAG.h"

using namespace llvm;

#define DEBUG_TYPE "machine-scheduler"

namespace {

class GCNILPScheduler {
  struct Candidate : ilist_node<Candidate> {
    SUnit *SU;

    Candidate(SUnit *SU_)
      : SU(SU_) {}
  };

  SpecificBumpPtrAllocator<Candidate> Alloc;
  using Queue = simple_ilist<Candidate>;
  Queue PendingQueue;
  Queue AvailQueue;
  unsigned CurQueueId = 0;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `GCNILPScheduler`, `Candidate`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`GCNILPScheduler`, `Candidate`。

### Lines 35-62: Declares getNodePriority
```cpp
  std::vector<unsigned> SUNumbers;

  /// CurCycle - The current scheduler state corresponds to this cycle.
  unsigned CurCycle = 0;

  unsigned getNodePriority(const SUnit *SU) const;

  const SUnit *pickBest(const SUnit *left, const SUnit *right);
  Candidate* pickCandidate();

  void releasePending();
  void advanceToCycle(unsigned NextCycle);
  void releasePredecessors(const SUnit* SU);

public:
  std::vector<const SUnit*> schedule(ArrayRef<const SUnit*> TopRoots,
                                     const ScheduleDAG &DAG);
};
} // namespace

/// CalcNodeSethiUllmanNumber - Compute Sethi Ullman number.
/// Smaller number is the higher priority.
static unsigned
CalcNodeSethiUllmanNumber(const SUnit *SU, std::vector<unsigned> &SUNumbers) {
  unsigned &SethiUllmanNumber = SUNumbers[SU->NodeNum];
  if (SethiUllmanNumber != 0)
    return SethiUllmanNumber;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 63-95: Conditional logic and checks
```cpp
  unsigned Extra = 0;
  for (const SDep &Pred : SU->Preds) {
    if (Pred.isCtrl()) continue;  // ignore chain preds
    SUnit *PredSU = Pred.getSUnit();
    unsigned PredSethiUllman = CalcNodeSethiUllmanNumber(PredSU, SUNumbers);
    if (PredSethiUllman > SethiUllmanNumber) {
      SethiUllmanNumber = PredSethiUllman;
      Extra = 0;
    }
    else if (PredSethiUllman == SethiUllmanNumber)
      ++Extra;
  }

  SethiUllmanNumber += Extra;

  if (SethiUllmanNumber == 0)
    SethiUllmanNumber = 1;

  return SethiUllmanNumber;
}

// Lower priority means schedule further down. For bottom-up scheduling, lower
// priority SUs are scheduled before higher priority SUs.
unsigned GCNILPScheduler::getNodePriority(const SUnit *SU) const {
  assert(SU->NodeNum < SUNumbers.size());
  if (SU->NumSuccs == 0 && SU->NumPreds != 0)
    // If SU does not have a register use, i.e. it doesn't produce a value
    // that would be consumed (e.g. store), then it terminates a chain of
    // computation.  Give it a large SethiUllman number so it will be
    // scheduled right before its predecessors that it doesn't lengthen
    // their live ranges.
    return 0xffff;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNILPScheduler::getNodePriority`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNILPScheduler::getNodePriority`。

### Lines 96-129: Defines closestSucc
```cpp
  if (SU->NumPreds == 0 && SU->NumSuccs != 0)
    // If SU does not have a register def, schedule it close to its uses
    // because it does not lengthen any live ranges.
    return 0;

  return SUNumbers[SU->NodeNum];
}

/// closestSucc - Returns the scheduled cycle of the successor which is
/// closest to the current cycle.
static unsigned closestSucc(const SUnit *SU) {
  unsigned MaxHeight = 0;
  for (const SDep &Succ : SU->Succs) {
    if (Succ.isCtrl()) continue;  // ignore chain succs
    unsigned Height = Succ.getSUnit()->getHeight();
    // If there are bunch of CopyToRegs stacked up, they should be considered
    // to be at the same position.
    if (Height > MaxHeight)
      MaxHeight = Height;
  }
  return MaxHeight;
}

/// calcMaxScratches - Returns an cost estimate of the worse case requirement
/// for scratch registers, i.e. number of data dependencies.
static unsigned calcMaxScratches(const SUnit *SU) {
  unsigned Scratches = 0;
  for (const SDep &Pred : SU->Preds) {
    if (Pred.isCtrl()) continue;  // ignore chain preds
    Scratches++;
  }
  return Scratches;
}

```
**EN:** This section contains concrete logic for closestSucc. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 closestSucc 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 130-161: Defines BUCompareLatency
```cpp
// Return -1 if left has higher priority, 1 if right has higher priority.
// Return 0 if latency-based priority is equivalent.
static int BUCompareLatency(const SUnit *left, const SUnit *right) {
  // Scheduling an instruction that uses a VReg whose postincrement has not yet
  // been scheduled will induce a copy. Model this as an extra cycle of latency.
  int LHeight = (int)left->getHeight();
  int RHeight = (int)right->getHeight();

  // If either node is scheduling for latency, sort them by height/depth
  // and latency.

  // If neither instruction stalls (!LStall && !RStall) and HazardRecognizer
  // is enabled, grouping instructions by cycle, then its height is already
  // covered so only its depth matters. We also reach this point if both stall
  // but have the same height.
  if (LHeight != RHeight)
    return LHeight > RHeight ? 1 : -1;

  int LDepth = left->getDepth();
  int RDepth = right->getDepth();
  if (LDepth != RDepth) {
    LLVM_DEBUG(dbgs() << "  Comparing latency of SU (" << left->NodeNum
                      << ") depth " << LDepth << " vs SU (" << right->NodeNum
                      << ") depth " << RDepth << "\n");
    return LDepth < RDepth ? 1 : -1;
  }
  if (left->Latency != right->Latency)
    return left->Latency > right->Latency ? 1 : -1;

  return 0;
}

```
**EN:** This section contains concrete logic for BUCompareLatency. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 BUCompareLatency 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 162-191: Implements GCNILPScheduler::pickBest
```cpp
const SUnit *GCNILPScheduler::pickBest(const SUnit *left, const SUnit *right)
{
  // TODO: add register pressure lowering checks

  bool const DisableSchedCriticalPath = false;
  int MaxReorderWindow = 6;
  if (!DisableSchedCriticalPath) {
    int spread = (int)left->getDepth() - (int)right->getDepth();
    if (std::abs(spread) > MaxReorderWindow) {
      LLVM_DEBUG(dbgs() << "Depth of SU(" << left->NodeNum << "): "
                        << left->getDepth() << " != SU(" << right->NodeNum
                        << "): " << right->getDepth() << "\n");
      return left->getDepth() < right->getDepth() ? right : left;
    }
  }

  bool const DisableSchedHeight = false;
  if (!DisableSchedHeight && left->getHeight() != right->getHeight()) {
    int spread = (int)left->getHeight() - (int)right->getHeight();
    if (std::abs(spread) > MaxReorderWindow)
      return left->getHeight() > right->getHeight() ? right : left;
  }

  // Prioritize by Sethi-Ulmann number and push CopyToReg nodes down.
  unsigned LPriority = getNodePriority(left);
  unsigned RPriority = getNodePriority(right);

  if (LPriority != RPriority)
    return LPriority > RPriority ? right : left;

```
**EN:** This section contains concrete logic for GCNILPScheduler::pickBest. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNILPScheduler::pickBest`, `std::abs`.
**CN:** 本节包含与 GCNILPScheduler::pickBest 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNILPScheduler::pickBest`, `std::abs`。

### Lines 192-219: Conditional logic and checks
```cpp
  // Try schedule def + use closer when Sethi-Ullman numbers are the same.
  // e.g.
  // t1 = op t2, c1
  // t3 = op t4, c2
  //
  // and the following instructions are both ready.
  // t2 = op c3
  // t4 = op c4
  //
  // Then schedule t2 = op first.
  // i.e.
  // t4 = op c4
  // t2 = op c3
  // t1 = op t2, c1
  // t3 = op t4, c2
  //
  // This creates more short live intervals.
  unsigned LDist = closestSucc(left);
  unsigned RDist = closestSucc(right);
  if (LDist != RDist)
    return LDist < RDist ? right : left;

  // How many registers becomes live when the node is scheduled.
  unsigned LScratch = calcMaxScratches(left);
  unsigned RScratch = calcMaxScratches(right);
  if (LScratch != RScratch)
    return LScratch > RScratch ? right : left;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 220-251: Conditional logic and checks
```cpp
  bool const DisableSchedCycles = false;
  if (!DisableSchedCycles) {
    int result = BUCompareLatency(left, right);
    if (result != 0)
      return result > 0 ? right : left;
    return left;
  }
  if (left->getHeight() != right->getHeight())
    return (left->getHeight() > right->getHeight()) ? right : left;

  if (left->getDepth() != right->getDepth())
    return (left->getDepth() < right->getDepth()) ? right : left;

  assert(left->NodeQueueId && right->NodeQueueId &&
        "NodeQueueId cannot be zero");
  return (left->NodeQueueId > right->NodeQueueId) ? right : left;
}

GCNILPScheduler::Candidate* GCNILPScheduler::pickCandidate() {
  if (AvailQueue.empty())
    return nullptr;
  auto Best = AvailQueue.begin();
  for (auto I = std::next(AvailQueue.begin()), E = AvailQueue.end(); I != E; ++I) {
    const auto *NewBestSU = pickBest(Best->SU, I->SU);
    if (NewBestSU != Best->SU) {
      assert(NewBestSU == I->SU);
      Best = I;
    }
  }
  return &*Best;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNILPScheduler::pickCandidate`, `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNILPScheduler::pickCandidate`, `std::next`。

### Lines 252-281: Implements GCNILPScheduler::releasePending
```cpp
void GCNILPScheduler::releasePending() {
  // Check to see if any of the pending instructions are ready to issue.  If
  // so, add them to the available queue.
  for(auto I = PendingQueue.begin(), E = PendingQueue.end(); I != E;) {
    auto &C = *I++;
    if (C.SU->getHeight() <= CurCycle) {
      PendingQueue.remove(C);
      AvailQueue.push_back(C);
      C.SU->NodeQueueId = CurQueueId++;
    }
  }
}

/// Move the scheduler state forward by the specified number of Cycles.
void GCNILPScheduler::advanceToCycle(unsigned NextCycle) {
  if (NextCycle <= CurCycle)
    return;
  CurCycle = NextCycle;
  releasePending();
}

void GCNILPScheduler::releasePredecessors(const SUnit* SU) {
  for (const auto &PredEdge : SU->Preds) {
    auto *PredSU = PredEdge.getSUnit();
    if (PredEdge.isWeak())
      continue;
    assert(PredSU->isBoundaryNode() || PredSU->NumSuccsLeft > 0);

    PredSU->setHeightToAtLeast(SU->getHeight() + PredEdge.getLatency());

```
**EN:** This section contains concrete logic for GCNILPScheduler::releasePending. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNILPScheduler::releasePending`, `GCNILPScheduler::advanceToCycle`, `GCNILPScheduler::releasePredecessors`.
**CN:** 本节包含与 GCNILPScheduler::releasePending 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNILPScheduler::releasePending`, `GCNILPScheduler::advanceToCycle`, `GCNILPScheduler::releasePredecessors`。

### Lines 282-309: Implements GCNILPScheduler::schedule
```cpp
    if (!PredSU->isBoundaryNode() && --PredSU->NumSuccsLeft == 0)
      PendingQueue.push_front(*new (Alloc.Allocate()) Candidate(PredSU));
  }
}

std::vector<const SUnit*>
GCNILPScheduler::schedule(ArrayRef<const SUnit*> BotRoots,
                          const ScheduleDAG &DAG) {
  auto &SUnits = const_cast<ScheduleDAG&>(DAG).SUnits;

  std::vector<SUnit> SUSavedCopy;
  SUSavedCopy.resize(SUnits.size());

  // we cannot save only those fields we touch: some of them are private
  // so save units verbatim: this assumes SUnit should have value semantics
  for (const SUnit &SU : SUnits)
    SUSavedCopy[SU.NodeNum] = SU;

  SUNumbers.assign(SUnits.size(), 0);
  for (const SUnit &SU : SUnits)
    CalcNodeSethiUllmanNumber(&SU, SUNumbers);

  for (const auto *SU : BotRoots) {
    AvailQueue.push_back(
      *new (Alloc.Allocate()) Candidate(const_cast<SUnit*>(SU)));
  }
  releasePredecessors(&DAG.ExitSU);

```
**EN:** This section contains concrete logic for GCNILPScheduler::schedule. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNILPScheduler::schedule`.
**CN:** 本节包含与 GCNILPScheduler::schedule 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNILPScheduler::schedule`。

### Lines 310-338: Declares reserve
```cpp
  std::vector<const SUnit*> Schedule;
  Schedule.reserve(SUnits.size());
  while (true) {
    if (AvailQueue.empty() && !PendingQueue.empty()) {
      auto *EarliestSU =
          llvm::min_element(PendingQueue, [=](const Candidate &C1,
                                              const Candidate &C2) {
            return C1.SU->getHeight() < C2.SU->getHeight();
          })->SU;
      advanceToCycle(std::max(CurCycle + 1, EarliestSU->getHeight()));
    }
    if (AvailQueue.empty())
      break;

    LLVM_DEBUG(dbgs() << "\n=== Picking candidate\n"
                         "Ready queue:";
               for (auto &C
                    : AvailQueue) dbgs()
               << ' ' << C.SU->NodeNum;
               dbgs() << '\n';);

    auto *C = pickCandidate();
    assert(C);
    AvailQueue.remove(*C);
    auto *SU = C->SU;
    LLVM_DEBUG(dbgs() << "Selected "; DAG.dumpNode(*SU));

    advanceToCycle(SU->getHeight());

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `llvm::min_element`, `std::max`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`llvm::min_element`, `std::max`。

### Lines 339-360: Declares assert
```cpp
    releasePredecessors(SU);
    Schedule.push_back(SU);
    SU->isScheduled = true;
  }
  assert(SUnits.size() == Schedule.size());

  std::reverse(Schedule.begin(), Schedule.end());

  // restore units
  for (auto &SU : SUnits)
    SU = SUSavedCopy[SU.NodeNum];

  return Schedule;
}

namespace llvm {
std::vector<const SUnit*> makeGCNILPScheduler(ArrayRef<const SUnit*> BotRoots,
                                              const ScheduleDAG &DAG) {
  GCNILPScheduler S;
  return S.schedule(BotRoots, DAG);
}
} // namespace llvm
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::reverse`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::reverse`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNILPScheduler`, `Candidate`, `GCNILPScheduler::getNodePriority`, `GCNILPScheduler::pickBest`, `std::abs`, `GCNILPScheduler::pickCandidate`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; lowering / 降低; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/CodeGen/ScheduleDAG.h"`
