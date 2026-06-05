# AMDGPUCoExecSchedStrategy.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUCoExecSchedStrategy.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUCoExecSchedStrategy in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUCoExecSchedStrategy 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File banner, includes, and setup
```cpp
//===- AMDGPUCoExecSchedStrategy.h - CoExec Scheduling Strategy -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Coexecution-focused scheduling strategy for AMDGPU.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUCOEXECSCHEDSTRATEGY_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUCOEXECSCHEDSTRATEGY_H

#include "GCNSchedStrategy.h"
#include "llvm/CodeGen/MachineScheduler.h"

namespace llvm {

namespace AMDGPU {

//===----------------------------------------------------------------------===//
// Instruction Flavor Classification
//===----------------------------------------------------------------------===//

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 28-61: Defines VALU
```cpp
enum class InstructionFlavor : uint8_t {
  WMMA,            // WMMA/MFMA matrix operations
  SingleCycleVALU, // Single-cycle VALU (not TRANS32, not multi-cycle CVT)
  TRANS,           // Transcendental ops (v_exp, v_log, etc.)
  MultiCycleVALU,  // VALU instructions with repeat rate > 1
  VMEM,            // FLAT/GLOBAL memory operations
  DS,              // LDS/GDS operations
  SALU,            // Scalar ALU
  DMA,             // Tensor DMA operations
  Fence,           // Fences and waits
  Other,           // Everything else
  NUM_FLAVORS
};

inline StringRef getFlavorName(InstructionFlavor F) {
  switch (F) {
  case InstructionFlavor::WMMA:
    return "WMMA";
  case InstructionFlavor::SingleCycleVALU:
    return "VALU(1c)";
  case InstructionFlavor::TRANS:
    return "TRANS";
  case InstructionFlavor::MultiCycleVALU:
    return "VALU(Nc)";
  case InstructionFlavor::VMEM:
    return "VMEM";
  case InstructionFlavor::DS:
    return "DS";
  case InstructionFlavor::SALU:
    return "SALU";
  case InstructionFlavor::DMA:
    return "DMA";
  case InstructionFlavor::Fence:
    return "Fence";
```
**EN:** This section contains concrete logic for VALU. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `InstructionFlavor`.
**CN:** 本节包含与 VALU 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`InstructionFlavor`。

### Lines 62-95: Declares llvm_unreachable
```cpp
  case InstructionFlavor::Other:
    return "Other";
  case InstructionFlavor::NUM_FLAVORS:
    return "???";
  }
  llvm_unreachable("Unknown InstructionFlavor");
}

inline StringRef getFlavorShortName(InstructionFlavor F) {
  switch (F) {
  case InstructionFlavor::WMMA:
    return "W";
  case InstructionFlavor::SingleCycleVALU:
    return "V";
  case InstructionFlavor::TRANS:
    return "T";
  case InstructionFlavor::MultiCycleVALU:
    return "C";
  case InstructionFlavor::VMEM:
    return "M";
  case InstructionFlavor::DS:
    return "D";
  case InstructionFlavor::SALU:
    return "S";
  case InstructionFlavor::DMA:
    return "X";
  case InstructionFlavor::Fence:
    return "F";
  case InstructionFlavor::Other:
    return "O";
  case InstructionFlavor::NUM_FLAVORS:
    return "?";
  }
  llvm_unreachable("Unknown InstructionFlavor");
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 96-121: Defines classifyFlavor
```cpp
}

InstructionFlavor classifyFlavor(const MachineInstr &MI,
                                 const SIInstrInfo &SII);

using FlavorGroup = SmallVector<InstructionFlavor, 4>;

namespace FlavorGroups {
inline FlavorGroup allVALU() {
  return {InstructionFlavor::SingleCycleVALU, InstructionFlavor::TRANS,
          InstructionFlavor::MultiCycleVALU};
}
inline FlavorGroup allMem() {
  return {InstructionFlavor::VMEM, InstructionFlavor::DS,
          InstructionFlavor::DMA};
}
inline FlavorGroup individual(InstructionFlavor F) { return {F}; }
inline FlavorGroup all() {
  FlavorGroup G;
  for (unsigned I = 0;
       I < static_cast<unsigned>(InstructionFlavor::NUM_FLAVORS); ++I)
    G.push_back(static_cast<InstructionFlavor>(I));
  return G;
}
} // namespace FlavorGroups

```
**EN:** This section contains concrete logic for classifyFlavor. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 classifyFlavor 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 122-150: Defines getReasonName
```cpp
/// AMDGPU-specific scheduling decision reasons. These provide more granularity
/// than the generic CandReason enum for debugging purposes.
enum class AMDGPUSchedReason : uint8_t {
  None,
  CritResourceBalance, // tryCriticalResource chose based on resource pressure
  CritResourceDep,     // tryCriticalResourceDependency chose based on enabling
  NUM_REASONS
};

inline StringRef getReasonName(AMDGPUSchedReason R) {
  switch (R) {
  case AMDGPUSchedReason::None:
    return "None";
  case AMDGPUSchedReason::CritResourceBalance:
    return "CritResource";
  case AMDGPUSchedReason::CritResourceDep:
    return "CritResourceDep";
  case AMDGPUSchedReason::NUM_REASONS:
    return "???";
  }
  llvm_unreachable("Unknown AMDGPUSchedReason");
}

} // End namespace AMDGPU

//===----------------------------------------------------------------------===//
// Hardware Unit Information
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for getReasonName. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSchedReason`.
**CN:** 本节包含与 getReasonName 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSchedReason`。

### Lines 151-182: Declares class HardwareUnitInfo
```cpp
/// HardwareUnitInfo is a wrapper class which maps to some real hardware
/// resource. This is used to model hardware resource pressure per region, and
/// guide scheduling heuristics.
class HardwareUnitInfo {
private:
  /// PrioritySUs maintains a list of the SUs we want to prioritize scheduling
  /// for this HardwareUnit. This is used for agreement between
  /// tryCriticalResourceDependency and tryCriticalResource: we schedule the
  /// dependencies for a SU on critical resource, then schedule that same SU on
  /// the critical resource. This agreement results in shorter live ranges and
  /// more regular HardwareUnit access patterns. SUs are prioritized based on
  /// depth for top-down scheduling.
  SmallSetVector<SUnit *, 16> PrioritySUs;
  /// All the SUs in the region that consume this resource
  SmallSetVector<SUnit *, 16> AllSUs;
  /// The total number of busy cycles for this HardwareUnit for a given region.
  unsigned TotalCycles = 0;
  // InstructionFlavor mapping
  AMDGPU::InstructionFlavor Type;
  // Whether or not instructions on this HardwareUnit may produce a window in
  // which instructions in other HardwareUnits can coexecute. For example, WMMA
  // / MFMA instructions may take multiple cycles, which may be overlapped with
  // instructions on other HardwareUnits
  bool ProducesCoexecWindow = false;

public:
  HardwareUnitInfo() {}

  unsigned size() { return AllSUs.size(); }

  unsigned getTotalCycles() { return TotalCycles; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `HardwareUnitInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`HardwareUnitInfo`。

### Lines 183-211: Defines setType
```cpp
  void setType(unsigned TheType) {
    assert(TheType < (unsigned)AMDGPU::InstructionFlavor::NUM_FLAVORS);
    Type = (AMDGPU::InstructionFlavor)(TheType);
  }

  AMDGPU::InstructionFlavor getType() const { return Type; }

  bool producesCoexecWindow() const { return ProducesCoexecWindow; }

  void setProducesCoexecWindow(bool Val) { ProducesCoexecWindow = Val; }

  bool contains(SUnit *SU) const { return AllSUs.contains(SU); }

  /// \returns true if there is a difference in priority between \p SU and \p
  /// Other. If so, \returns the SUnit with higher priority. This
  /// method looks through the PrioritySUs to determine if one SU is more
  /// prioritized than the other. If neither are in the PrioritySUs list, then
  /// neither have priority over each other.
  SUnit *getHigherPriority(SUnit *SU, SUnit *Other) const {
    for (auto *SUOrder : PrioritySUs) {
      if (SUOrder == SU)
        return SU;

      if (SUOrder == Other)
        return Other;
    }
    return nullptr;
  }

```
**EN:** This section contains concrete logic for setType. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 setType 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 212-242: Defines reset
```cpp
  void reset() {
    AllSUs.clear();
    PrioritySUs.clear();
    TotalCycles = 0;
    Type = AMDGPU::InstructionFlavor::Other;
    ProducesCoexecWindow = false;
  }

  /// \returns the next SU in PrioritySUs that is not ready. If \p LookDeep is
  /// set, we will look beyond the PrioritySUs (if all the PrioritySUs are
  /// ready) to AllSUs to attempt to find a target SU. When looking through
  /// AllSUs we sort pick the target SU by minimal depth for top-down
  /// scheduling. getNextTargetSU is useful for determining which SU on this
  /// HardwareUnit we are trying to schedule - this info helps us determine
  /// which dependencies to schedule. LookDeep is useful if the dependencies are
  /// long latency (e.g. memory instructions). If we have many long latency
  /// dependencies, it is beneficial to enable SUs multiple levels ahead.
  SUnit *getNextTargetSU(bool LookDeep = false) const;
  /// Insert the \p SU into the AllSUs and account its \p BlockingCycles into
  /// the TotalCycles. This maintains the list of PrioritySUs.
  void insert(SUnit *SU, unsigned BlockingCycles);
  /// Update the state for \p SU being scheduled by removing it from the AllSus
  /// and reducing its \p BlockingCycles from the TotalCycles. This maintains
  /// the list of PrioritySUS.
  void markScheduled(SUnit *SU, unsigned BlockingCycles);
};

//===----------------------------------------------------------------------===//
// Candidate Heuristics
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for reset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 reset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 243-273: Declares class CandidateHeuristics
```cpp
/// CandidateHeuristics contains state and implementations to facilitate making
/// per instruction scheduling decisions; it contains methods used in
/// tryCandidate to decide which instruction to schedule next.
class CandidateHeuristics {
protected:
  ScheduleDAGMI *DAG;
  const SIInstrInfo *SII;
  const SIRegisterInfo *SRI;
  const TargetSchedModel *SchedModel;
  SmallVector<HardwareUnitInfo, 8> HWUInfo;

  /// Walk over the region and collect total usage per HardwareUnit
  void collectHWUIPressure();

  /// Compute the blocking cycles for the appropriate HardwareUnit given an \p
  /// SU
  unsigned getHWUICyclesForInst(SUnit *SU);

  /// Given a \p Flavor , find the corresponding HardwareUnit. \returns the
  /// mapped HardwareUnit.
  HardwareUnitInfo *getHWUIFromFlavor(AMDGPU::InstructionFlavor Flavor);

public:
  CandidateHeuristics() = default;

  void initialize(ScheduleDAGMI *DAG, const TargetSchedModel *SchedModel,
                  const TargetRegisterInfo *TRI);

  /// Update the state to reflect that \p SU is going to be scheduled.
  void updateForScheduling(SUnit *SU);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `CandidateHeuristics`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`CandidateHeuristics`。

### Lines 274-306: Declares class AMDGPUCoExecSchedStrategy
```cpp
  /// Sort the HWUInfo vector. After sorting, the HardwareUnits that are highest
  /// priority are first. Priority is determined by maximizing coexecution and
  /// keeping the critical HardwareUnit busy.
  void sortHWUIResources();

  /// Check for critical resource consumption. Prefer the candidate that uses
  /// the most prioritized HardwareUnit. If both candidates use the same
  /// HarwareUnit, prefer the candidate with higher priority on that
  /// HardwareUnit.
  bool tryCriticalResource(GenericSchedulerBase::SchedCandidate &TryCand,
                           GenericSchedulerBase::SchedCandidate &Cand,
                           SchedBoundary *Zone) const;

  /// Check for dependencies of instructions that use prioritized HardwareUnits.
  /// Prefer the candidate that is a dependency of an instruction that uses the
  /// most prioritized HardwareUnit. If both candidates enable the same
  /// HardwareUnit, prefer the candidate that enables the higher priority
  /// instruction on that HardwareUnit.
  bool
  tryCriticalResourceDependency(GenericSchedulerBase::SchedCandidate &TryCand,
                                GenericSchedulerBase::SchedCandidate &Cand,
                                SchedBoundary *Zone) const;

  void dumpRegionSummary();
};

class AMDGPUCoExecSchedStrategy final : public GCNSchedStrategy {
protected:
  bool tryEffectiveStall(SchedCandidate &Cand, SchedCandidate &TryCand,
                         SchedBoundary &Zone) const;
  AMDGPU::AMDGPUSchedReason LastAMDGPUReason = AMDGPU::AMDGPUSchedReason::None;
  CandidateHeuristics Heurs;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUCoExecSchedStrategy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUCoExecSchedStrategy`。

### Lines 307-334: Preprocessor guards and macros
```cpp
#ifndef NDEBUG
  void dumpPickSummary(SUnit *SU, bool IsTopNode, SchedCandidate &Cand);
#endif

  bool tryCandidateCoexec(SchedCandidate &Cand, SchedCandidate &TryCand,
                          SchedBoundary *Zone);
  void pickNodeFromQueue(SchedBoundary &Zone, const CandPolicy &ZonePolicy,
                         const RegPressureTracker &RPTracker,
                         SchedCandidate &Cand, bool &PickedPending,
                         bool IsBottomUp);

public:
  AMDGPUCoExecSchedStrategy(const MachineSchedContext *C);

  void initPolicy(MachineBasicBlock::iterator Begin,
                  MachineBasicBlock::iterator End,
                  unsigned NumRegionInstrs) override;
  void initialize(ScheduleDAGMI *DAG) override;
  SUnit *pickNode(bool &IsTopNode) override;
  void schedNode(SUnit *SU, bool IsTopNode) override;
};

ScheduleDAGInstrs *createGCNCoExecMachineScheduler(MachineSchedContext *C);
ScheduleDAGInstrs *createGCNNoopPostMachineScheduler(MachineSchedContext *C);

} // End namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUCOEXECSCHEDSTRATEGY_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `InstructionFlavor`, `AMDGPUSchedReason`, `HardwareUnitInfo`, `CandidateHeuristics`, `AMDGPUCoExecSchedStrategy`
- **Main themes / 核心主题**: instruction semantics / 指令语义; scheduling / 调度; hardware resource modeling / 硬件资源建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNSchedStrategy.h"`
- `"llvm/CodeGen/MachineScheduler.h"`
