# SIMachineScheduler.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIMachineScheduler.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for SIMachineScheduler in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 SIMachineScheduler 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File banner, includes, and setup
```cpp
//===-- SIMachineScheduler.h - SI Scheduler Interface -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// SI Machine Scheduler interface
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_SIMACHINESCHEDULER_H
#define LLVM_LIB_TARGET_AMDGPU_SIMACHINESCHEDULER_H

#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/RegisterPressure.h"
#include "llvm/CodeGen/ScheduleDAG.h"
#include <cstdint>
#include <set>
#include <vector>

namespace llvm {

class SIInstrInfo;
class SIRegisterInfo;
class SIScheduleDAGMI;
class SIScheduleBlockCreator;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIInstrInfo`, `SIRegisterInfo`, `SIScheduleDAGMI`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIInstrInfo`, `SIRegisterInfo`, `SIScheduleDAGMI`。

### Lines 31-61: Declares struct SISchedulerCandidate
```cpp
enum SIScheduleCandReason {
  NoCand,
  RegUsage,
  Latency,
  Successor,
  Depth,
  NodeOrder
};

struct SISchedulerCandidate {
  // The reason for this candidate.
  SIScheduleCandReason Reason = NoCand;

  // Set of reasons that apply to multiple candidates.
  uint32_t RepeatReasonSet = 0;

  SISchedulerCandidate() = default;

  bool isRepeat(SIScheduleCandReason R) { return RepeatReasonSet & (1 << R); }
  void setRepeat(SIScheduleCandReason R) { RepeatReasonSet |= (1 << R); }
};

enum SIScheduleBlockLinkKind {
  NoData,
  Data
};

class SIScheduleBlock {
  SIScheduleDAGMI *DAG;
  SIScheduleBlockCreator *BC;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIScheduleCandReason`, `SISchedulerCandidate`, `SIScheduleBlockLinkKind`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIScheduleCandReason`, `SISchedulerCandidate`, `SIScheduleBlockLinkKind`。

### Lines 62-94: Implementation details and local logic
```cpp
  std::vector<SUnit*> SUnits;
  std::map<unsigned, unsigned> NodeNum2Index;
  std::vector<SUnit*> TopReadySUs;
  std::vector<SUnit*> ScheduledSUnits;

  /// The top of the unscheduled zone.
  IntervalPressure TopPressure;
  RegPressureTracker TopRPTracker;

  // Pressure: number of said class of registers needed to
  // store the live virtual and real registers.
  // We do care only of SGPR32 and VGPR32 and do track only virtual registers.
  // Pressure of additional registers required inside the block.
  std::vector<unsigned> InternalAdditionalPressure;
  // Pressure of input and output registers
  std::vector<unsigned> LiveInPressure;
  std::vector<unsigned> LiveOutPressure;
  // Registers required by the block, and outputs.
  // We do track only virtual registers.
  // Note that some registers are not 32 bits,
  // and thus the pressure is not equal
  // to the number of live registers.
  std::set<Register> LiveInRegs;
  std::set<Register> LiveOutRegs;

  bool Scheduled = false;
  bool HighLatencyBlock = false;

  std::vector<unsigned> HasLowLatencyNonWaitedParent;

  // Unique ID, the index of the Block in the SIScheduleDAGMI Blocks table.
  unsigned ID;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 95-125: Defines SIScheduleBlock
```cpp
  std::vector<SIScheduleBlock*> Preds;  // All blocks predecessors.
  // All blocks successors, and the kind of link
  std::vector<std::pair<SIScheduleBlock*, SIScheduleBlockLinkKind>> Succs;
  unsigned NumHighLatencySuccessors = 0;

public:
  SIScheduleBlock(SIScheduleDAGMI *DAG, SIScheduleBlockCreator *BC,
                  unsigned ID):
    DAG(DAG), BC(BC), TopRPTracker(TopPressure), ID(ID) {}

  ~SIScheduleBlock() = default;

  unsigned getID() const { return ID; }

  /// Functions for Block construction.
  void addUnit(SUnit *SU);

  // When all SUs have been added.
  void finalizeUnits();

  // Add block pred, which has instruction predecessor of SU.
  void addPred(SIScheduleBlock *Pred);
  void addSucc(SIScheduleBlock *Succ, SIScheduleBlockLinkKind Kind);

  const std::vector<SIScheduleBlock*>& getPreds() const { return Preds; }
  ArrayRef<std::pair<SIScheduleBlock*, SIScheduleBlockLinkKind>>
    getSuccs() const { return Succs; }

  unsigned Height = 0;  // Maximum topdown path length to block without outputs
  unsigned Depth = 0;   // Maximum bottomup path length to block without inputs

```
**EN:** This section contains concrete logic for SIScheduleBlock. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 SIScheduleBlock 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 126-159: Defines getNumHighLatencySuccessors
```cpp
  unsigned getNumHighLatencySuccessors() const {
    return NumHighLatencySuccessors;
  }

  bool isHighLatencyBlock() { return HighLatencyBlock; }

  // This is approximative.
  // Ideally should take into accounts some instructions (rcp, etc)
  // are 4 times slower.
  int getCost() { return SUnits.size(); }

  // The block Predecessors and Successors must be all registered
  // before fastSchedule().
  // Fast schedule with no particular requirement.
  void fastSchedule();

  std::vector<SUnit*> getScheduledUnits() { return ScheduledSUnits; }

  // Complete schedule that will try to minimize reg pressure and
  // low latencies, and will fill liveins and liveouts.
  // Needs all MIs to be grouped between BeginBlock and EndBlock.
  // The MIs can be moved after the scheduling,
  // it is just used to allow correct track of live registers.
  void schedule(MachineBasicBlock::iterator BeginBlock,
                MachineBasicBlock::iterator EndBlock);

  bool isScheduled() { return Scheduled; }

  // Needs the block to be scheduled inside
  // TODO: find a way to compute it.
  std::vector<unsigned> &getInternalAdditionalRegUsage() {
    return InternalAdditionalPressure;
  }

```
**EN:** This section contains concrete logic for getNumHighLatencySuccessors. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getNumHighLatencySuccessors 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 160-192: Declares struct SISchedCandidate
```cpp
  std::set<Register> &getInRegs() { return LiveInRegs; }
  std::set<Register> &getOutRegs() { return LiveOutRegs; }

  void printDebug(bool Full);

private:
  struct SISchedCandidate : SISchedulerCandidate {
    // The best SUnit candidate.
    SUnit *SU = nullptr;

    unsigned SGPRUsage;
    unsigned VGPRUsage;
    bool IsLowLatency;
    unsigned LowLatencyOffset;
    bool HasLowLatencyNonWaitedParent;

    SISchedCandidate() = default;

    bool isValid() const { return SU; }

    // Copy the status of another candidate without changing policy.
    void setBest(SISchedCandidate &Best) {
      assert(Best.Reason != NoCand && "uninitialized Sched candidate");
      SU = Best.SU;
      Reason = Best.Reason;
      SGPRUsage = Best.SGPRUsage;
      VGPRUsage = Best.VGPRUsage;
      IsLowLatency = Best.IsLowLatency;
      LowLatencyOffset = Best.LowLatencyOffset;
      HasLowLatencyNonWaitedParent = Best.HasLowLatencyNonWaitedParent;
    }
  };

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SISchedCandidate`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SISchedCandidate`。

### Lines 193-221: Declares struct SIScheduleBlocks
```cpp
  void undoSchedule();

  void undoReleaseSucc(SUnit *SU, SDep *SuccEdge);
  void releaseSucc(SUnit *SU, SDep *SuccEdge);
  // InOrOutBlock: restrict to links pointing inside the block (true),
  // or restrict to links pointing outside the block (false).
  void releaseSuccessors(SUnit *SU, bool InOrOutBlock);

  void nodeScheduled(SUnit *SU);
  void tryCandidateTopDown(SISchedCandidate &Cand, SISchedCandidate &TryCand);
  void tryCandidateBottomUp(SISchedCandidate &Cand, SISchedCandidate &TryCand);
  SUnit* pickNode();
  void traceCandidate(const SISchedCandidate &Cand);
  void initRegPressure(MachineBasicBlock::iterator BeginBlock,
                       MachineBasicBlock::iterator EndBlock);
};

struct SIScheduleBlocks {
  std::vector<SIScheduleBlock*> Blocks;
  std::vector<int> TopDownIndex2Block;
  std::vector<int> TopDownBlock2Index;
};

enum SISchedulerBlockCreatorVariant {
  LatenciesAlone,
  LatenciesGrouped,
  LatenciesAlonePlusConsecutive
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIScheduleBlocks`, `SISchedulerBlockCreatorVariant`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIScheduleBlocks`, `SISchedulerBlockCreatorVariant`。

### Lines 222-253: Declares class SIScheduleBlockCreator
```cpp
class SIScheduleBlockCreator {
  SIScheduleDAGMI *DAG;
  // unique_ptr handles freeing memory for us.
  std::vector<std::unique_ptr<SIScheduleBlock>> BlockPtrs;
  std::map<SISchedulerBlockCreatorVariant,
           SIScheduleBlocks> Blocks;
  std::vector<SIScheduleBlock*> CurrentBlocks;
  std::vector<int> Node2CurrentBlock;

  // Topological sort
  // Maps topological index to the node number.
  std::vector<int> TopDownIndex2Block;
  std::vector<int> TopDownBlock2Index;
  std::vector<int> BottomUpIndex2Block;

  // 0 -> Color not given.
  // 1 to SUnits.size() -> Reserved group (you should only add elements to them).
  // Above -> Other groups.
  int NextReservedID;
  int NextNonReservedID;
  std::vector<int> CurrentColoring;
  std::vector<int> CurrentTopDownReservedDependencyColoring;
  std::vector<int> CurrentBottomUpReservedDependencyColoring;

public:
  SIScheduleBlockCreator(SIScheduleDAGMI *DAG);

  SIScheduleBlocks
  getBlocks(SISchedulerBlockCreatorVariant BlockVariant);

  bool isSUInBlock(SUnit *SU, unsigned ID);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIScheduleBlockCreator`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIScheduleBlockCreator`。

### Lines 254-286: Declares colorHighLatenciesAlone
```cpp
private:
  // Give a Reserved color to every high latency.
  void colorHighLatenciesAlone();

  // Create groups of high latencies with a Reserved color.
  void colorHighLatenciesGroups();

  // Compute coloring for topdown and bottom traversals with
  // different colors depending on dependencies on Reserved colors.
  void colorComputeReservedDependencies();

  // Give color to all non-colored SUs according to Reserved groups dependencies.
  void colorAccordingToReservedDependencies();

  // Divides Blocks having no bottom up or top down dependencies on Reserved groups.
  // The new colors are computed according to the dependencies on the other blocks
  // formed with colorAccordingToReservedDependencies.
  void colorEndsAccordingToDependencies();

  // Cut groups into groups with SUs in consecutive order (except for Reserved groups).
  void colorForceConsecutiveOrderInGroup();

  // Merge Constant loads that have all their users into another group to the group.
  // (TODO: else if all their users depend on the same group, put them there)
  void colorMergeConstantLoadsNextGroup();

  // Merge SUs that have all their users into another group to the group
  void colorMergeIfPossibleNextGroup();

  // Merge SUs that have all their users into another group to the group,
  // but only for Reserved groups.
  void colorMergeIfPossibleNextGroupOnlyForReserved();

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 287-317: Declares colorMergeIfPossibleSmallGroupsToNextGroup
```cpp
  // Merge SUs that have all their users into another group to the group,
  // but only if the group is no more than a few SUs.
  void colorMergeIfPossibleSmallGroupsToNextGroup();

  // Divides Blocks with important size.
  // Idea of implementation: attribute new colors depending on topdown and
  // bottom up links to other blocks.
  void cutHugeBlocks();

  // Put in one group all instructions with no users in this scheduling region
  // (we'd want these groups be at the end).
  void regroupNoUserInstructions();

  // Give Reserved color to export instructions
  void colorExports();

  void createBlocksForVariant(SISchedulerBlockCreatorVariant BlockVariant);

  void topologicalSort();

  void scheduleInsideBlocks();

  void fillStats();
};

enum SISchedulerBlockSchedulerVariant {
  BlockLatencyRegUsage,
  BlockRegUsageLatency,
  BlockRegUsage
};

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SISchedulerBlockSchedulerVariant`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SISchedulerBlockSchedulerVariant`。

### Lines 318-350: Declares class SIScheduleBlockScheduler
```cpp
class SIScheduleBlockScheduler {
  SIScheduleDAGMI *DAG;
  SISchedulerBlockSchedulerVariant Variant;
  std::vector<SIScheduleBlock*> Blocks;

  std::vector<std::map<Register, unsigned>> LiveOutRegsNumUsages;
  std::set<Register> LiveRegs;
  // Num of schedulable unscheduled blocks reading the register.
  std::map<Register, unsigned> LiveRegsConsumers;

  std::vector<unsigned> LastPosHighLatencyParentScheduled;
  int LastPosWaitedHighLatency;

  std::vector<SIScheduleBlock*> BlocksScheduled;
  unsigned NumBlockScheduled;
  std::vector<SIScheduleBlock*> ReadyBlocks;

  unsigned VregCurrentUsage;
  unsigned SregCurrentUsage;

  // Currently is only approximation.
  unsigned maxVregUsage;
  unsigned maxSregUsage;

  std::vector<unsigned> BlockNumPredsLeft;
  std::vector<unsigned> BlockNumSuccsLeft;

public:
  SIScheduleBlockScheduler(SIScheduleDAGMI *DAG,
                           SISchedulerBlockSchedulerVariant Variant,
                           SIScheduleBlocks BlocksStruct);
  ~SIScheduleBlockScheduler() = default;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIScheduleBlockScheduler`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIScheduleBlockScheduler`。

### Lines 351-371: Declares struct SIBlockSchedCandidate
```cpp
  std::vector<SIScheduleBlock*> getBlocks() { return BlocksScheduled; }

  unsigned getVGPRUsage() { return maxVregUsage; }
  unsigned getSGPRUsage() { return maxSregUsage; }

private:
  struct SIBlockSchedCandidate : SISchedulerCandidate {
    // The best Block candidate.
    SIScheduleBlock *Block = nullptr;

    bool IsHighLatency;
    int VGPRUsageDiff;
    unsigned NumSuccessors;
    unsigned NumHighLatencySuccessors;
    unsigned LastPosHighLatParentScheduled;
    unsigned Height;

    SIBlockSchedCandidate() = default;

    bool isValid() const { return Block; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIBlockSchedCandidate`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIBlockSchedCandidate`。

### Lines 372-404: Implementation details and local logic
```cpp
    // Copy the status of another candidate without changing policy.
    void setBest(SIBlockSchedCandidate &Best) {
      assert(Best.Reason != NoCand && "uninitialized Sched candidate");
      Block = Best.Block;
      Reason = Best.Reason;
      IsHighLatency = Best.IsHighLatency;
      VGPRUsageDiff = Best.VGPRUsageDiff;
      NumSuccessors = Best.NumSuccessors;
      NumHighLatencySuccessors = Best.NumHighLatencySuccessors;
      LastPosHighLatParentScheduled = Best.LastPosHighLatParentScheduled;
      Height = Best.Height;
    }
  };

  bool tryCandidateLatency(SIBlockSchedCandidate &Cand,
                           SIBlockSchedCandidate &TryCand);
  bool tryCandidateRegUsage(SIBlockSchedCandidate &Cand,
                            SIBlockSchedCandidate &TryCand);
  SIScheduleBlock *pickBlock();

  void addLiveRegs(std::set<VirtRegOrUnit> &Regs);
  void decreaseLiveRegs(SIScheduleBlock *Block, std::set<Register> &Regs);
  void releaseBlockSuccs(SIScheduleBlock *Parent);
  void blockScheduled(SIScheduleBlock *Block);

  // Check register pressure change
  // by scheduling a block with these LiveIn and LiveOut.
  std::vector<int> checkRegUsageImpact(std::set<Register> &InRegs,
                                       std::set<Register> &OutRegs);

  void schedule();
};

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 405-437: Declares struct SIScheduleBlockResult
```cpp
struct SIScheduleBlockResult {
  std::vector<unsigned> SUs;
  unsigned MaxSGPRUsage;
  unsigned MaxVGPRUsage;
};

class SIScheduler {
  SIScheduleDAGMI *DAG;
  SIScheduleBlockCreator BlockCreator;

public:
  SIScheduler(SIScheduleDAGMI *DAG) : DAG(DAG), BlockCreator(DAG) {}

  ~SIScheduler() = default;

  struct SIScheduleBlockResult
  scheduleVariant(SISchedulerBlockCreatorVariant BlockVariant,
                  SISchedulerBlockSchedulerVariant ScheduleVariant);
};

class SIScheduleDAGMI final : public ScheduleDAGMILive {
  const SIInstrInfo *SITII;
  const SIRegisterInfo *SITRI;

  std::vector<SUnit> SUnitsLinksBackup;

  // For moveLowLatencies. After all Scheduling variants are tested.
  std::vector<unsigned> ScheduledSUnits;
  std::vector<unsigned> ScheduledSUnitsInv;

public:
  SIScheduleDAGMI(MachineSchedContext *C);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIScheduleBlockResult`, `SIScheduler`, `SIScheduleDAGMI`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIScheduleBlockResult`, `SIScheduler`, `SIScheduleDAGMI`。

### Lines 438-464: Declares ~SIScheduleDAGMI
```cpp
  ~SIScheduleDAGMI() override;

  // Entry point for the schedule.
  void schedule() override;

  // To init Block's RPTracker.
  void initRPTracker(RegPressureTracker &RPTracker) {
    RPTracker.init(&MF, RegClassInfo, LIS, BB, RegionBegin, false, false);
  }

  MachineBasicBlock *getBB() { return BB; }
  MachineBasicBlock::iterator getCurrentTop() { return CurrentTop; }
  MachineBasicBlock::iterator getCurrentBottom() { return CurrentBottom; }
  LiveIntervals *getLIS() { return LIS; }
  MachineRegisterInfo *getMRI() { return &MRI; }
  const TargetRegisterInfo *getTRI() { return TRI; }
  ScheduleDAGTopologicalSort *GetTopo() { return &Topo; }
  SUnit &getEntrySU() { return EntrySU; }
  SUnit& getExitSU() { return ExitSU; }

  void restoreSULinksLeft();

  template<typename _Iterator> void fillVgprSgprCost(_Iterator First,
                                                     _Iterator End,
                                                     unsigned &VgprUsage,
                                                     unsigned &SgprUsage);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 465-498: Defines getInRegs
```cpp
  std::set<VirtRegOrUnit> getInRegs() {
    std::set<VirtRegOrUnit> InRegs;
    for (const auto &RegMaskPair : RPTracker.getPressure().LiveInRegs) {
      InRegs.insert(RegMaskPair.VRegOrUnit);
    }
    return InRegs;
  }

  std::set<VirtRegOrUnit> getOutRegs() {
    std::set<VirtRegOrUnit> OutRegs;
    for (const auto &RegMaskPair : RPTracker.getPressure().LiveOutRegs) {
      OutRegs.insert(RegMaskPair.VRegOrUnit);
    }
    return OutRegs;
  };

private:
  void topologicalSort();
  // After scheduling is done, improve low latency placements.
  void moveLowLatencies();

public:
  // Some stats for scheduling inside blocks.
  std::vector<unsigned> IsLowLatencySU;
  std::vector<unsigned> LowLatencyOffset;
  std::vector<unsigned> IsHighLatencySU;
  // Topological sort
  // Maps topological index to the node number.
  std::vector<int> TopDownIndex2SU;
  std::vector<int> BottomUpIndex2SU;
};

} // end namespace llvm

```
**EN:** This section contains concrete logic for getInRegs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getInRegs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 499-499: Preprocessor guards and macros
```cpp
#endif // LLVM_LIB_TARGET_AMDGPU_SIMACHINESCHEDULER_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `SIInstrInfo`, `SIRegisterInfo`, `SIScheduleDAGMI`, `SIScheduleBlockCreator`, `SIScheduleCandReason`, `SISchedulerCandidate`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; lowering / 降低; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/CodeGen/MachineScheduler.h"`
- `"llvm/CodeGen/RegisterPressure.h"`
- `"llvm/CodeGen/ScheduleDAG.h"`
- `<cstdint>`
- `<set>`
- `<vector>`
