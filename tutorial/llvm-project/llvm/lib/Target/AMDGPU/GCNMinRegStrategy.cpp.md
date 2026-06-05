# GCNMinRegStrategy.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNMinRegStrategy.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNMinRegStrategy for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNMinRegStrategy 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===- GCNMinRegStrategy.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines and implements the class GCNMinRegScheduler, which
/// implements an experimental, simple scheduler whose main goal is to learn
/// ways about consuming less possible registers for a region.
///
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/ScheduleDAG.h"
using namespace llvm;

#define DEBUG_TYPE "machine-scheduler"

namespace {

class GCNMinRegScheduler {
  struct Candidate : ilist_node<Candidate> {
    const SUnit *SU;
    int Priority;

    Candidate(const SUnit *SU_, int Priority_ = 0)
      : SU(SU_), Priority(Priority_) {}
  };

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `GCNMinRegScheduler`, `Candidate`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`GCNMinRegScheduler`, `Candidate`。

### Lines 32-64: Defines isScheduled
```cpp
  SpecificBumpPtrAllocator<Candidate> Alloc;
  using Queue = simple_ilist<Candidate>;
  Queue RQ; // Ready queue

  std::vector<unsigned> NumPreds;

  bool isScheduled(const SUnit *SU) const {
    assert(!SU->isBoundaryNode());
    return NumPreds[SU->NodeNum] == std::numeric_limits<unsigned>::max();
  }

  void setIsScheduled(const SUnit *SU)  {
    assert(!SU->isBoundaryNode());
    NumPreds[SU->NodeNum] = std::numeric_limits<unsigned>::max();
  }

  unsigned getNumPreds(const SUnit *SU) const {
    assert(!SU->isBoundaryNode());
    assert(NumPreds[SU->NodeNum] != std::numeric_limits<unsigned>::max());
    return NumPreds[SU->NodeNum];
  }

  unsigned decNumPreds(const SUnit *SU) {
    assert(!SU->isBoundaryNode());
    assert(NumPreds[SU->NodeNum] != std::numeric_limits<unsigned>::max());
    return --NumPreds[SU->NodeNum];
  }

  void initNumPreds(const decltype(ScheduleDAG::SUnits) &SUnits);

  int getReadySuccessors(const SUnit *SU) const;
  int getNotReadySuccessors(const SUnit *SU) const;

```
**EN:** This section contains concrete logic for isScheduled. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isScheduled 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 65-85: Declares findMax
```cpp
  template <typename Calc>
  unsigned findMax(unsigned Num, Calc C);

  Candidate* pickCandidate();

  void bumpPredsPriority(const SUnit *SchedSU, int Priority);
  void releaseSuccessors(const SUnit* SU, int Priority);

public:
  std::vector<const SUnit*> schedule(ArrayRef<const SUnit*> TopRoots,
                                     const ScheduleDAG &DAG);
};

} // end anonymous namespace

void GCNMinRegScheduler::initNumPreds(const decltype(ScheduleDAG::SUnits) &SUnits) {
  NumPreds.resize(SUnits.size());
  for (unsigned I = 0; I < SUnits.size(); ++I)
    NumPreds[I] = SUnits[I].NumPredsLeft;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `GCNMinRegScheduler::initNumPreds`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`GCNMinRegScheduler::initNumPreds`。

### Lines 86-112: Implements GCNMinRegScheduler::getReadySuccessors
```cpp
int GCNMinRegScheduler::getReadySuccessors(const SUnit *SU) const {
  unsigned NumSchedSuccs = 0;
  for (auto SDep : SU->Succs) {
    bool wouldBeScheduled = true;
    for (auto PDep : SDep.getSUnit()->Preds) {
      auto *PSU = PDep.getSUnit();
      assert(!PSU->isBoundaryNode());
      if (PSU != SU && !isScheduled(PSU)) {
        wouldBeScheduled = false;
        break;
      }
    }
    NumSchedSuccs += wouldBeScheduled ? 1 : 0;
  }
  return NumSchedSuccs;
}

int GCNMinRegScheduler::getNotReadySuccessors(const SUnit *SU) const {
  return SU->Succs.size() - getReadySuccessors(SU);
}

template <typename Calc>
unsigned GCNMinRegScheduler::findMax(unsigned Num, Calc C) {
  assert(!RQ.empty() && Num <= RQ.size());

  using T = decltype(C(*RQ.begin())) ;

```
**EN:** This section contains concrete logic for GCNMinRegScheduler::getReadySuccessors. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNMinRegScheduler::getReadySuccessors`, `GCNMinRegScheduler::getNotReadySuccessors`, `GCNMinRegScheduler::findMax`.
**CN:** 本节包含与 GCNMinRegScheduler::getReadySuccessors 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNMinRegScheduler::getReadySuccessors`, `GCNMinRegScheduler::getNotReadySuccessors`, `GCNMinRegScheduler::findMax`。

### Lines 113-142: Declares min
```cpp
  T Max = std::numeric_limits<T>::min();
  unsigned NumMax = 0;
  for (auto I = RQ.begin(); Num; --Num) {
    T Cur = C(*I);
    if (Cur >= Max) {
      if (Cur > Max) {
        Max = Cur;
        NumMax = 1;
      } else
        ++NumMax;
      auto &Cand = *I++;
      RQ.remove(Cand);
      RQ.push_front(Cand);
      continue;
    }
    ++I;
  }
  return NumMax;
}

GCNMinRegScheduler::Candidate* GCNMinRegScheduler::pickCandidate() {
  do {
    unsigned Num = RQ.size();
    if (Num == 1) break;

    LLVM_DEBUG(dbgs() << "\nSelecting max priority candidates among " << Num
                      << '\n');
    Num = findMax(Num, [=](const Candidate &C) { return C.Priority; });
    if (Num == 1) break;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `GCNMinRegScheduler::pickCandidate`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`GCNMinRegScheduler::pickCandidate`。

### Lines 143-176: Conditional logic and checks
```cpp
    LLVM_DEBUG(dbgs() << "\nSelecting min non-ready producing candidate among "
                      << Num << '\n');
    Num = findMax(Num, [=](const Candidate &C) {
      const auto *SU = C.SU;
      int Res = getNotReadySuccessors(SU);
      LLVM_DEBUG(dbgs() << "SU(" << SU->NodeNum << ") would left non-ready "
                        << Res << " successors, metric = " << -Res << '\n');
      return -Res;
    });
    if (Num == 1) break;

    LLVM_DEBUG(dbgs() << "\nSelecting most producing candidate among " << Num
                      << '\n');
    Num = findMax(Num, [=](const Candidate &C) {
      const auto *SU = C.SU;
      auto Res = getReadySuccessors(SU);
      LLVM_DEBUG(dbgs() << "SU(" << SU->NodeNum << ") would make ready " << Res
                        << " successors, metric = " << Res << '\n');
      return Res;
    });
    if (Num == 1) break;

    Num = Num ? Num : RQ.size();
    LLVM_DEBUG(
        dbgs()
        << "\nCan't find best candidate, selecting in program order among "
        << Num << '\n');
    Num = findMax(Num, [=](const Candidate &C) { return -(int64_t)C.SU->NodeNum; });
    assert(Num == 1);
  } while (false);

  return &RQ.front();
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 177-210: Implements GCNMinRegScheduler::bumpPredsPriority
```cpp
void GCNMinRegScheduler::bumpPredsPriority(const SUnit *SchedSU, int Priority) {
  SmallPtrSet<const SUnit*, 32> Set;
  for (const auto &S : SchedSU->Succs) {
    if (S.getSUnit()->isBoundaryNode() || isScheduled(S.getSUnit()) ||
        S.getKind() != SDep::Data)
      continue;
    for (const auto &P : S.getSUnit()->Preds) {
      auto *PSU = P.getSUnit();
      assert(!PSU->isBoundaryNode());
      if (PSU != SchedSU && !isScheduled(PSU)) {
        Set.insert(PSU);
      }
    }
  }
  SmallVector<const SUnit*, 32> Worklist(Set.begin(), Set.end());
  while (!Worklist.empty()) {
    const auto *SU = Worklist.pop_back_val();
    assert(!SU->isBoundaryNode());
    for (const auto &P : SU->Preds) {
      if (!P.getSUnit()->isBoundaryNode() && !isScheduled(P.getSUnit()) &&
          Set.insert(P.getSUnit()).second)
        Worklist.push_back(P.getSUnit());
    }
  }
  LLVM_DEBUG(dbgs() << "Make the predecessors of SU(" << SchedSU->NodeNum
                    << ")'s non-ready successors of " << Priority
                    << " priority in ready queue: ");
  for (auto &C : RQ) {
    if (Set.count(C.SU)) {
      C.Priority = Priority;
      LLVM_DEBUG(dbgs() << " SU(" << C.SU->NodeNum << ')');
    }
  }
  LLVM_DEBUG(dbgs() << '\n');
```
**EN:** This section contains concrete logic for GCNMinRegScheduler::bumpPredsPriority. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNMinRegScheduler::bumpPredsPriority`.
**CN:** 本节包含与 GCNMinRegScheduler::bumpPredsPriority 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNMinRegScheduler::bumpPredsPriority`。

### Lines 211-239: Implements GCNMinRegScheduler::releaseSuccessors
```cpp
}

void GCNMinRegScheduler::releaseSuccessors(const SUnit* SU, int Priority) {
  for (const auto &S : SU->Succs) {
    auto *SuccSU = S.getSUnit();
    if (S.isWeak())
      continue;
    assert(SuccSU->isBoundaryNode() || getNumPreds(SuccSU) > 0);
    if (!SuccSU->isBoundaryNode() && decNumPreds(SuccSU) == 0)
      RQ.push_front(*new (Alloc.Allocate()) Candidate(SuccSU, Priority));
  }
}

std::vector<const SUnit*>
GCNMinRegScheduler::schedule(ArrayRef<const SUnit*> TopRoots,
                             const ScheduleDAG &DAG) {
  const auto &SUnits = DAG.SUnits;
  std::vector<const SUnit*> Schedule;
  Schedule.reserve(SUnits.size());

  initNumPreds(SUnits);

  int StepNo = 0;

  for (const auto *SU : TopRoots) {
    RQ.push_back(*new (Alloc.Allocate()) Candidate(SU, StepNo));
  }
  releaseSuccessors(&DAG.EntrySU, StepNo);

```
**EN:** This section contains concrete logic for GCNMinRegScheduler::releaseSuccessors. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNMinRegScheduler::releaseSuccessors`, `GCNMinRegScheduler::schedule`.
**CN:** 本节包含与 GCNMinRegScheduler::releaseSuccessors 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNMinRegScheduler::releaseSuccessors`, `GCNMinRegScheduler::schedule`。

### Lines 240-270: Namespace declarations and scope setup
```cpp
  while (!RQ.empty()) {
    LLVM_DEBUG(dbgs() << "\n=== Picking candidate, Step = " << StepNo
                      << "\n"
                         "Ready queue:";
               for (auto &C
                    : RQ) dbgs()
               << ' ' << C.SU->NodeNum << "(P" << C.Priority << ')';
               dbgs() << '\n';);

    auto *C = pickCandidate();
    assert(C);
    RQ.remove(*C);
    const auto *SU = C->SU;
    LLVM_DEBUG(dbgs() << "Selected "; DAG.dumpNode(*SU));

    releaseSuccessors(SU, StepNo);
    Schedule.push_back(SU);
    setIsScheduled(SU);

    if (getReadySuccessors(SU) == 0)
      bumpPredsPriority(SU, StepNo);

    ++StepNo;
  }
  assert(SUnits.size() == Schedule.size());

  return Schedule;
}

namespace llvm {

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。

### Lines 271-277: Defines makeMinRegSchedule
```cpp
std::vector<const SUnit*> makeMinRegSchedule(ArrayRef<const SUnit*> TopRoots,
                                             const ScheduleDAG &DAG) {
  GCNMinRegScheduler S;
  return S.schedule(TopRoots, DAG);
}

} // end namespace llvm
```
**EN:** This section contains concrete logic for makeMinRegSchedule. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 makeMinRegSchedule 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNMinRegScheduler`, `Candidate`, `GCNMinRegScheduler::initNumPreds`, `GCNMinRegScheduler::getReadySuccessors`, `GCNMinRegScheduler::getNotReadySuccessors`, `GCNMinRegScheduler::findMax`
- **Main themes / 核心主题**: register management / 寄存器管理; scheduling / 调度; SelectionDAG processing / SelectionDAG 处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/CodeGen/ScheduleDAG.h"`
