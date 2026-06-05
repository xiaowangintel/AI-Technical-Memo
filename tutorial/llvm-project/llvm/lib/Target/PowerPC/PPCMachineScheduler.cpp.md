# PPCMachineScheduler.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCMachineScheduler.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCMachineScheduler.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- PPCMachineScheduler.cpp - MI Scheduler for PowerPC -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段包含调度或处理器模型元数据。

### Lines 7-44

```cpp
//===----------------------------------------------------------------------===//

#include "PPCMachineScheduler.h"
#include "MCTargetDesc/PPCMCTargetDesc.h"

using namespace llvm;

static cl::opt<bool> 
DisableAddiLoadHeuristic("disable-ppc-sched-addi-load",
                         cl::desc("Disable scheduling addi instruction before" 
                                  "load for ppc"), cl::Hidden);
static cl::opt<bool>
    EnableAddiHeuristic("ppc-postra-bias-addi",
                        cl::desc("Enable scheduling addi instruction as early"
                                 "as possible post ra"),
                        cl::Hidden, cl::init(true));

static bool isADDIInstr(const GenericScheduler::SchedCandidate &Cand) {
  return Cand.SU->getInstr()->getOpcode() == PPC::ADDI ||
         Cand.SU->getInstr()->getOpcode() == PPC::ADDI8;
}

bool PPCPreRASchedStrategy::biasAddiLoadCandidate(SchedCandidate &Cand,
                                                  SchedCandidate &TryCand,
                                                  SchedBoundary &Zone) const {
  if (DisableAddiLoadHeuristic)
    return false;

  SchedCandidate &FirstCand = Zone.isTop() ? TryCand : Cand;
  SchedCandidate &SecondCand = Zone.isTop() ? Cand : TryCand;
  if (isADDIInstr(FirstCand) && SecondCand.SU->getInstr()->mayLoad()) {
    TryCand.Reason = Stall;
    return true;
  }
  if (FirstCand.SU->getInstr()->mayLoad() && isADDIInstr(SecondCand)) {
    TryCand.Reason = NoCand;
    return true;
  }
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段包含调度或处理器模型元数据。

### Lines 45-82

```cpp

  return false;
}

bool PPCPreRASchedStrategy::tryCandidate(SchedCandidate &Cand,
                                         SchedCandidate &TryCand,
                                         SchedBoundary *Zone) const {
  // From GenericScheduler::tryCandidate

  // Initialize the candidate if needed.
  if (!Cand.isValid()) {
    TryCand.Reason = NodeOrder;
    return true;
  }

  // Bias PhysReg Defs and copies to their uses and defined respectively.
  if (tryGreater(biasPhysReg(TryCand.SU, TryCand.AtTop),
                 biasPhysReg(Cand.SU, Cand.AtTop), TryCand, Cand, PhysReg))
    return TryCand.Reason != NoCand;

  // Avoid exceeding the target's limit.
  if (DAG->isTrackingPressure() &&
      tryPressure(TryCand.RPDelta.Excess, Cand.RPDelta.Excess, TryCand, Cand,
                  RegExcess, TRI, DAG->MF))
    return TryCand.Reason != NoCand;

  // Avoid increasing the max critical pressure in the scheduled region.
  if (DAG->isTrackingPressure() &&
      tryPressure(TryCand.RPDelta.CriticalMax, Cand.RPDelta.CriticalMax,
                  TryCand, Cand, RegCritical, TRI, DAG->MF))
    return TryCand.Reason != NoCand;

  // We only compare a subset of features when comparing nodes between
  // Top and Bottom boundary. Some properties are simply incomparable, in many
  // other instances we should only override the other boundary if something
  // is a clear good pick on one boundary. Skip heuristics that are more
  // "tie-breaking" in nature.
  bool SameBoundary = Zone != nullptr;
```
- **EN**: Implements helper routine(s) `tryCandidate`, `isValid`, `tryGreater` for this portion of the PowerPC backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `tryCandidate`, `isValid`, `tryGreater`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 83-120

```cpp
  if (SameBoundary) {
    // For loops that are acyclic path limited, aggressively schedule for
    // latency. Within an single cycle, whenever CurrMOps > 0, allow normal
    // heuristics to take precedence.
    if (Rem.IsAcyclicLatencyLimited && !Zone->getCurrMOps() &&
        tryLatency(TryCand, Cand, *Zone))
      return TryCand.Reason != NoCand;

    // Prioritize instructions that read unbuffered resources by stall cycles.
    if (tryLess(Zone->getLatencyStallCycles(TryCand.SU),
                Zone->getLatencyStallCycles(Cand.SU), TryCand, Cand, Stall))
      return TryCand.Reason != NoCand;
  }

  // Keep clustered nodes together to encourage downstream peephole
  // optimizations which may reduce resource requirements.
  //
  // This is a best effort to set things up for a post-RA pass. Optimizations
  // like generating loads of multiple registers should ideally be done within
  // the scheduler pass by combining the loads during DAG postprocessing.
  unsigned CandZoneCluster = Cand.AtTop ? TopClusterID : BotClusterID;
  unsigned TryCandZoneCluster = TryCand.AtTop ? TopClusterID : BotClusterID;
  bool CandIsClusterSucc =
      isTheSameCluster(CandZoneCluster, Cand.SU->ParentClusterIdx);
  bool TryCandIsClusterSucc =
      isTheSameCluster(TryCandZoneCluster, TryCand.SU->ParentClusterIdx);

  if (tryGreater(TryCandIsClusterSucc, CandIsClusterSucc, TryCand, Cand,
                 Cluster))
    return TryCand.Reason != NoCand;

  if (SameBoundary) {
    // Weak edges are for clustering and other constraints.
    if (tryLess(getWeakLeft(TryCand.SU, TryCand.AtTop),
                getWeakLeft(Cand.SU, Cand.AtTop), TryCand, Cand, Weak))
      return TryCand.Reason != NoCand;
  }
```
- **EN**: Implements helper routine(s) `getCurrMOps`, `tryLatency`, `tryLess` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getCurrMOps`, `tryLatency`, `tryLess`。

### Lines 121-158

```cpp
  // Avoid increasing the max pressure of the entire region.
  if (DAG->isTrackingPressure() &&
      tryPressure(TryCand.RPDelta.CurrentMax, Cand.RPDelta.CurrentMax, TryCand,
                  Cand, RegMax, TRI, DAG->MF))
    return TryCand.Reason != NoCand;

  if (SameBoundary) {
    // Avoid critical resource consumption and balance the schedule.
    TryCand.initResourceDelta(DAG, SchedModel);
    if (tryLess(TryCand.ResDelta.CritResources, Cand.ResDelta.CritResources,
                TryCand, Cand, ResourceReduce))
      return TryCand.Reason != NoCand;
    if (tryGreater(TryCand.ResDelta.DemandedResources,
                   Cand.ResDelta.DemandedResources, TryCand, Cand,
                   ResourceDemand))
      return TryCand.Reason != NoCand;

    // Avoid serializing long latency dependence chains.
    // For acyclic path limited loops, latency was already checked above.
    if (!RegionPolicy.DisableLatencyHeuristic && TryCand.Policy.ReduceLatency &&
        !Rem.IsAcyclicLatencyLimited && tryLatency(TryCand, Cand, *Zone))
      return TryCand.Reason != NoCand;

    // Fall through to original instruction order.
    if ((Zone->isTop() && TryCand.SU->NodeNum < Cand.SU->NodeNum) ||
        (!Zone->isTop() && TryCand.SU->NodeNum > Cand.SU->NodeNum)) {
      TryCand.Reason = NodeOrder;
    }
  }

  // GenericScheduler::tryCandidate end

  // Add powerpc specific heuristic only when TryCand isn't selected or
  // selected as node order.
  if (TryCand.Reason != NodeOrder && TryCand.Reason != NoCand)
    return true;

  // There are some benefits to schedule the ADDI before the load to hide the
```
- **EN**: Implements helper routine(s) `isTrackingPressure`, `tryPressure`, `initResourceDelta` for this portion of the PowerPC backend backend implementation logic. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isTrackingPressure`, `tryPressure`, `initResourceDelta`。 这一段包含调度或处理器模型元数据。

### Lines 159-196

```cpp
  // latency, as RA may create a true dependency between the load and addi.
  if (SameBoundary) {
    if (biasAddiLoadCandidate(Cand, TryCand, *Zone))
      return TryCand.Reason != NoCand;
  }

  return TryCand.Reason != NoCand;
}

bool PPCPostRASchedStrategy::biasAddiCandidate(SchedCandidate &Cand,
                                               SchedCandidate &TryCand) const {
  if (!EnableAddiHeuristic)
    return false;

  if (isADDIInstr(TryCand) && !isADDIInstr(Cand)) {
    TryCand.Reason = Stall;
    return true;
  }
  return false;
}

bool PPCPostRASchedStrategy::tryCandidate(SchedCandidate &Cand,
                                          SchedCandidate &TryCand) {
  // From PostGenericScheduler::tryCandidate

  // Initialize the candidate if needed.
  if (!Cand.isValid()) {
    TryCand.Reason = NodeOrder;
    return true;
  }

  // Prioritize instructions that read unbuffered resources by stall cycles.
  if (tryLess(Top.getLatencyStallCycles(TryCand.SU),
              Top.getLatencyStallCycles(Cand.SU), TryCand, Cand, Stall))
    return TryCand.Reason != NoCand;

  // Keep clustered nodes together.
  unsigned CandZoneCluster = Cand.AtTop ? TopClusterID : BotClusterID;
```
- **EN**: Implements helper routine(s) `biasAddiLoadCandidate`, `biasAddiCandidate`, `isADDIInstr` for this portion of the PowerPC backend backend implementation logic. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `biasAddiLoadCandidate`, `biasAddiCandidate`, `isADDIInstr`。 这一段包含调度或处理器模型元数据。

### Lines 197-234

```cpp
  unsigned TryCandZoneCluster = TryCand.AtTop ? TopClusterID : BotClusterID;
  bool CandIsClusterSucc =
      isTheSameCluster(CandZoneCluster, Cand.SU->ParentClusterIdx);
  bool TryCandIsClusterSucc =
      isTheSameCluster(TryCandZoneCluster, TryCand.SU->ParentClusterIdx);

  if (tryGreater(TryCandIsClusterSucc, CandIsClusterSucc, TryCand, Cand,
                 Cluster))
    return TryCand.Reason != NoCand;

  // Avoid critical resource consumption and balance the schedule.
  if (tryLess(TryCand.ResDelta.CritResources, Cand.ResDelta.CritResources,
              TryCand, Cand, ResourceReduce))
    return TryCand.Reason != NoCand;
  if (tryGreater(TryCand.ResDelta.DemandedResources,
                 Cand.ResDelta.DemandedResources, TryCand, Cand,
                 ResourceDemand))
    return TryCand.Reason != NoCand;

  // Avoid serializing long latency dependence chains.
  if (Cand.Policy.ReduceLatency && tryLatency(TryCand, Cand, Top)) {
    return TryCand.Reason != NoCand;
  }

  // Fall through to original instruction order.
  if (TryCand.SU->NodeNum < Cand.SU->NodeNum)
    TryCand.Reason = NodeOrder;

  // PostGenericScheduler::tryCandidate end

  // Add powerpc post ra specific heuristic only when TryCand isn't selected or
  // selected as node order.
  if (TryCand.Reason != NodeOrder && TryCand.Reason != NoCand)
    return true;

  // There are some benefits to schedule the ADDI as early as possible post ra
  // to avoid stalled by vector instructions which take up all the hw units.
  // And ADDI is usually used to post inc the loop indvar, which matters the
```
- **EN**: Implements helper routine(s) `isTheSameCluster`, `tryGreater`, `tryLess` for this portion of the PowerPC backend backend implementation logic. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isTheSameCluster`, `tryGreater`, `tryLess`。 这一段包含调度或处理器模型元数据。

### Lines 235-261

```cpp
  // performance.
  if (biasAddiCandidate(Cand, TryCand))
    return TryCand.Reason != NoCand;

  return TryCand.Reason != NoCand;
}

void PPCPostRASchedStrategy::enterMBB(MachineBasicBlock *MBB) {
  // Custom PPC PostRA specific behavior here.
  PostGenericScheduler::enterMBB(MBB);
}

void PPCPostRASchedStrategy::leaveMBB() {
  // Custom PPC PostRA specific behavior here.
  PostGenericScheduler::leaveMBB();
}

void PPCPostRASchedStrategy::initialize(ScheduleDAGMI *Dag) {
  // Custom PPC PostRA specific initialization here.
  PostGenericScheduler::initialize(Dag);
}

SUnit *PPCPostRASchedStrategy::pickNode(bool &IsTopNode) {
  // Custom PPC PostRA specific scheduling here.
  return PostGenericScheduler::pickNode(IsTopNode);
}
```
- **EN**: Implements helper routine(s) `biasAddiCandidate`, `enterMBB`, `leaveMBB` for this portion of the PowerPC backend backend implementation logic. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `biasAddiCandidate`, `enterMBB`, `leaveMBB`。 这一段包含调度或处理器模型元数据。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Instruction selection or opcode handling / 指令选择或操作码处理
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCMachineScheduler.h`
- `MCTargetDesc/PPCMCTargetDesc.h`

### Important Collaborators / 重要协作组件

- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
