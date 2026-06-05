# RISCVMachineScheduler.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVMachineScheduler.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V machine scheduling strategies and mutations. / 实现RISC-V 机器调度策略与变换。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===- RISCVMachineScheduler.cpp - MI Scheduler for RISC-V ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "RISCVMachineScheduler.h"
#include "llvm/CodeGen/ScheduleDAG.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 12-22: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-prera-sched-strategy"

RISCV::VSETVLIInfo
RISCVPreRAMachineSchedStrategy::getVSETVLIInfo(const MachineInstr *MI) const {
  unsigned TSFlags = MI->getDesc().TSFlags;
  if (!RISCVII::hasSEWOp(TSFlags))
    return RISCV::VSETVLIInfo();
  return VIA.computeInfoForInstr(*MI);
}
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 23-37: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVPreRAMachineSchedStrategy::tryVSETVLIInfo(
    const RISCV::VSETVLIInfo &TryInfo, const RISCV::VSETVLIInfo &CandInfo,
    SchedCandidate &TryCand, SchedCandidate &Cand, CandReason Reason) const {
  // Do not compare the vsetvli info changes between top and bottom
  // boundary.
  if (Cand.AtTop != TryCand.AtTop)
    return false;

  auto IsCompatible = [&](const RISCV::VSETVLIInfo &FirstInfo,
                          const RISCV::VSETVLIInfo &SecondInfo) {
    return FirstInfo.isValid() && SecondInfo.isValid() &&
           FirstInfo.isCompatible(RISCV::DemandedFields::all(), SecondInfo,
                                  Context->LIS);
  };
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 38-47: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Try Cand first.
  // We prefer the top node as it is straightforward from the perspective of
  // vsetvli dataflow.
  if (Cand.AtTop && IsCompatible(CandInfo, TopInfo))
    return true;

  if (!Cand.AtTop && IsCompatible(CandInfo, BottomInfo))
    return true;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 48-57: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Then try TryCand.
  if (TryCand.AtTop && IsCompatible(TryInfo, TopInfo)) {
    TryCand.Reason = Reason;
    return true;
  }

  if (!TryCand.AtTop && IsCompatible(TryInfo, BottomInfo)) {
    TryCand.Reason = Reason;
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 58-73: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return false;
}

bool RISCVPreRAMachineSchedStrategy::tryCandidate(SchedCandidate &Cand,
                                                  SchedCandidate &TryCand,
                                                  SchedBoundary *Zone) const {
  //-------------------------------------------------------------------------//
  // Below is copied from `GenericScheduler::tryCandidate`.
  // FIXME: Is there a way to not replicate this?
  //-------------------------------------------------------------------------//
  // Initialize the candidate if needed.
  if (!Cand.isValid()) {
    TryCand.Reason = FirstValid;
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 74-85: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Bias PhysReg Defs and copies to their uses and defined respectively.
  if (tryGreater(biasPhysReg(TryCand.SU, TryCand.AtTop),
                 biasPhysReg(Cand.SU, Cand.AtTop), TryCand, Cand, PhysReg))
    return TryCand.Reason != NoCand;

  // Avoid exceeding the target's limit.
  if (DAG->isTrackingPressure() &&
      tryPressure(TryCand.RPDelta.Excess, Cand.RPDelta.Excess, TryCand, Cand,
                  RegExcess, TRI, DAG->MF))
    return TryCand.Reason != NoCand;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 86-105: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
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
  if (SameBoundary) {
    // For loops that are acyclic path limited, aggressively schedule for
    // latency. Within an single cycle, whenever CurrMOps > 0, allow normal
    // heuristics to take precedence.
    if (Rem.IsAcyclicLatencyLimited && !Zone->getCurrMOps() &&
        tryLatency(TryCand, Cand, *Zone))
      return TryCand.Reason != NoCand;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 106-124: Definitions and supporting logic / 定义与支撑逻辑
```cpp
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
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 125-134: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
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
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 135-152: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
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
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 153-163: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    // Avoid serializing long latency dependence chains.
    // For acyclic path limited loops, latency was already checked above.
    if (!RegionPolicy.DisableLatencyHeuristic && TryCand.Policy.ReduceLatency &&
        !Rem.IsAcyclicLatencyLimited && tryLatency(TryCand, Cand, *Zone))
      return TryCand.Reason != NoCand;

    // Fall through to original instruction order.
    if ((Zone->isTop() && TryCand.SU->NodeNum < Cand.SU->NodeNum) ||
        (!Zone->isTop() && TryCand.SU->NodeNum > Cand.SU->NodeNum))
      TryCand.Reason = NodeOrder;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 164-173: Commentary and design intent / 注释与设计意图
```cpp

  //-------------------------------------------------------------------------//
  // Below is RISC-V specific scheduling heuristics.
  //-------------------------------------------------------------------------//

  // Add RISC-V specific heuristic only when TryCand isn't selected or
  // selected as node order.
  if (TryCand.Reason != NodeOrder && TryCand.Reason != NoCand)
    return true;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 174-183: Enumeration and symbolic state / 枚举与符号状态
```cpp
  // TODO: We should not use `CandReason::Cluster` here, but is there a
  // mechanism to extend this enum?
  if (ST->enableVsetvliSchedHeuristic() &&
      tryVSETVLIInfo(getVSETVLIInfo(TryCand.SU->getInstr()),
                     getVSETVLIInfo(Cand.SU->getInstr()), TryCand, Cand,
                     Cluster))
    return TryCand.Reason != NoCand;

  return TryCand.Reason != NoCand;
}
```
**EN:** This block defines named constants or state encodings that make later target logic more explicit and less error-prone.

**CN:** 该区段定义具名常量或状态编码，使后续目标相关逻辑更清晰且更不易出错。

### Lines 184-193: Function implementation: RISCVPreRAMachineSchedStrategy::enterMBB / 函数实现：RISCVPreRAMachineSchedStrategy::enterMBB
```cpp

void RISCVPreRAMachineSchedStrategy::enterMBB(MachineBasicBlock *MBB) {
  TopInfo = RISCV::VSETVLIInfo();
  BottomInfo = RISCV::VSETVLIInfo();
}

void RISCVPreRAMachineSchedStrategy::leaveMBB() {
  TopInfo = RISCV::VSETVLIInfo();
  BottomInfo = RISCV::VSETVLIInfo();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 194-213: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVPreRAMachineSchedStrategy::schedNode(SUnit *SU, bool IsTopNode) {
  GenericScheduler::schedNode(SU, IsTopNode);
  if (ST->enableVsetvliSchedHeuristic()) {
    MachineInstr *MI = SU->getInstr();
    const RISCV::VSETVLIInfo &Info = getVSETVLIInfo(MI);
    if (Info.isValid()) {
      if (IsTopNode)
        TopInfo = Info;
      else
        BottomInfo = Info;
      LLVM_DEBUG({
        dbgs() << "Previous scheduled Unit: \n";
        dbgs() << "  IsTop: " << IsTopNode << "\n";
        dbgs() << "  SU(" << SU->NodeNum << ") - ";
        MI->dump();
        dbgs() << "  \n";
        Info.dump();
        dbgs() << "  \n";
      });
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 214-216: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    }
  }
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**

## Dependencies / 依赖关系
- `RISCVMachineScheduler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/ScheduleDAG.h` — Directly referenced by this file. / 该文件直接引用的依赖。
