# GCNSchedStrategy.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNSchedStrategy.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for GCNSchedStrategy in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 GCNSchedStrategy 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: File banner, includes, and setup
```cpp
//===-- GCNSchedStrategy.h - GCN Scheduler Strategy -*- C++ -*-------------===//
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

#ifndef LLVM_LIB_TARGET_AMDGPU_GCNSCHEDSTRATEGY_H
#define LLVM_LIB_TARGET_AMDGPU_GCNSCHEDSTRATEGY_H

#include "GCNRegPressure.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/Rematerializer.h"

namespace llvm {

class SIMachineFunctionInfo;
class SIRegisterInfo;
class GCNSubtarget;
class GCNSchedStage;

enum class GCNSchedStageID : unsigned {
  OccInitialSchedule = 0,
  RewriteMFMAForm = 1,
  UnclusteredHighRPReschedule = 2,
  ClusteredLowOccupancyReschedule = 3,
  PreRARematerialize = 4,
  ILPInitialSchedule = 5,
  MemoryClauseInitialSchedule = 6
};

#ifndef NDEBUG
raw_ostream &operator<<(raw_ostream &OS, const GCNSchedStageID &StageID);
#endif

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIMachineFunctionInfo`, `SIRegisterInfo`, `GCNSubtarget`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIMachineFunctionInfo`, `SIRegisterInfo`, `GCNSubtarget`。

### Lines 45-88: Declares class GCNSchedStrategy
```cpp
/// This is a minimal scheduler strategy.  The main difference between this
/// and the GenericScheduler is that GCNSchedStrategy uses different
/// heuristics to determine excess/critical pressure sets.
class GCNSchedStrategy : public GenericScheduler {
protected:
  SUnit *pickNodeBidirectional(bool &IsTopNode, bool &PickedPending);

  void pickNodeFromQueue(SchedBoundary &Zone, const CandPolicy &ZonePolicy,
                         const RegPressureTracker &RPTracker,
                         SchedCandidate &Cand, bool &IsPending,
                         bool IsBottomUp);

  void initCandidate(SchedCandidate &Cand, SUnit *SU, bool AtTop,
                     const RegPressureTracker &RPTracker,
                     const SIRegisterInfo *SRI, unsigned SGPRPressure,
                     unsigned VGPRPressure, bool IsBottomUp);

  /// Estimate how many cycles \p SU must wait due to structural hazards at the
  /// current boundary cycle. Returns zero when no stall is required.
  unsigned getStructuralStallCycles(SchedBoundary &Zone, SUnit *SU) const;

  /// Evaluates instructions in the pending queue using a subset of scheduling
  /// heuristics.
  ///
  /// Instructions that cannot be issued due to hardware constraints are placed
  /// in the pending queue rather than the available queue, making them normally
  /// invisible to scheduling heuristics. However, in certain scenarios (such as
  /// avoiding register spilling), it may be beneficial to consider scheduling
  /// these not-yet-ready instructions.
  bool tryPendingCandidate(SchedCandidate &Cand, SchedCandidate &TryCand,
                           SchedBoundary *Zone) const;

  void printCandidateDecision(const SchedCandidate &Current,
                              const SchedCandidate &Preferred);

  void getRegisterPressures(bool AtTop, const RegPressureTracker &RPTracker,
                            SUnit *SU, std::vector<unsigned> &Pressure,
                            std::vector<unsigned> &MaxPressure,
                            GCNDownwardRPTracker &DownwardTracker,
                            GCNUpwardRPTracker &UpwardTracker,
                            ScheduleDAGMI *DAG, const SIRegisterInfo *SRI);

  std::vector<unsigned> Pressure;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNSchedStrategy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNSchedStrategy`。

### Lines 89-131: Implementation details and local logic
```cpp
  std::vector<unsigned> MaxPressure;

  unsigned SGPRExcessLimit;

  unsigned VGPRExcessLimit;

  unsigned TargetOccupancy;

  MachineFunction *MF;

  // Scheduling stages for this strategy.
  SmallVector<GCNSchedStageID, 4> SchedStages;

  // Pointer to the current SchedStageID.
  SmallVectorImpl<GCNSchedStageID>::iterator CurrentStage = nullptr;

  // GCN RP Tracker for top-down scheduling
  mutable GCNDownwardRPTracker DownwardTracker;

  // GCN RP Tracker for botttom-up scheduling
  mutable GCNUpwardRPTracker UpwardTracker;

  bool UseGCNTrackers = false;

  std::optional<bool> GCNTrackersOverride;

public:
  // schedule() have seen register pressure over the critical limits and had to
  // track register pressure for actual scheduling heuristics.
  bool HasHighPressure;

  // Schedule known to have excess register pressure. Be more conservative in
  // increasing ILP and preserving VGPRs.
  bool KnownExcessRP = false;

  // An error margin is necessary because of poor performance of the generic RP
  // tracker and can be adjusted up for tuning heuristics to try and more
  // aggressively reduce register pressure.
  unsigned ErrorMargin = 3;

  // Bias for SGPR limits under a high register pressure.
  const unsigned HighRPSGPRBias = 7;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 132-172: Declares GCNSchedStrategy
```cpp
  // Bias for VGPR limits under a high register pressure.
  const unsigned HighRPVGPRBias = 7;

  unsigned SGPRCriticalLimit;

  unsigned VGPRCriticalLimit;

  unsigned SGPRLimitBias = 0;

  unsigned VGPRLimitBias = 0;

  GCNSchedStrategy(const MachineSchedContext *C);

  SUnit *pickNode(bool &IsTopNode) override;

  void schedNode(SUnit *SU, bool IsTopNode) override;

  void initialize(ScheduleDAGMI *DAG) override;

  unsigned getTargetOccupancy() { return TargetOccupancy; }

  void setTargetOccupancy(unsigned Occ) { TargetOccupancy = Occ; }

  GCNSchedStageID getCurrentStage();

  // Advances stage. Returns true if there are remaining stages.
  bool advanceStage();

  bool hasNextStage() const;

  bool useGCNTrackers() const {
    return GCNTrackersOverride.value_or(UseGCNTrackers);
  }

  GCNSchedStageID getNextStage() const;

  GCNDownwardRPTracker *getDownwardTracker() { return &DownwardTracker; }

  GCNUpwardRPTracker *getUpwardTracker() { return &UpwardTracker; }
};

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 173-206: Declares class GCNMaxOccupancySchedStrategy
```cpp
/// The goal of this scheduling strategy is to maximize kernel occupancy (i.e.
/// maximum number of waves per simd).
class GCNMaxOccupancySchedStrategy final : public GCNSchedStrategy {
public:
  GCNMaxOccupancySchedStrategy(const MachineSchedContext *C,
                               bool IsLegacyScheduler = false);
};

/// The goal of this scheduling strategy is to maximize ILP for a single wave
/// (i.e. latency hiding).
class GCNMaxILPSchedStrategy final : public GCNSchedStrategy {
protected:
  bool tryCandidate(SchedCandidate &Cand, SchedCandidate &TryCand,
                    SchedBoundary *Zone) const override;

public:
  GCNMaxILPSchedStrategy(const MachineSchedContext *C);
};

/// The goal of this scheduling strategy is to maximize memory clause for a
/// single wave.
class GCNMaxMemoryClauseSchedStrategy final : public GCNSchedStrategy {
protected:
  bool tryCandidate(SchedCandidate &Cand, SchedCandidate &TryCand,
                    SchedBoundary *Zone) const override;

public:
  GCNMaxMemoryClauseSchedStrategy(const MachineSchedContext *C);
};

class ScheduleMetrics {
  unsigned ScheduleLength;
  unsigned BubbleCycles;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNMaxOccupancySchedStrategy`, `GCNMaxILPSchedStrategy`, `GCNMaxMemoryClauseSchedStrategy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNMaxOccupancySchedStrategy`, `GCNMaxILPSchedStrategy`, `GCNMaxMemoryClauseSchedStrategy`。

### Lines 207-246: Declares class GCNScheduleDAGMILive
```cpp
public:
  ScheduleMetrics() = default;
  ScheduleMetrics(unsigned L, unsigned BC)
      : ScheduleLength(L), BubbleCycles(BC) {}
  unsigned getLength() const { return ScheduleLength; }
  unsigned getBubbles() const { return BubbleCycles; }
  unsigned getMetric() const {
    unsigned Metric = (BubbleCycles * ScaleFactor) / ScheduleLength;
    // Metric is zero if the amount of bubbles is less than 1% which is too
    // small. So, return 1.
    return Metric ? Metric : 1;
  }
  static const unsigned ScaleFactor;
};

inline raw_ostream &operator<<(raw_ostream &OS, const ScheduleMetrics &Sm) {
  dbgs() << "\n Schedule Metric (scaled by " << ScheduleMetrics::ScaleFactor
         << " ) is: " << Sm.getMetric() << " [ " << Sm.getBubbles() << "/"
         << Sm.getLength() << " ]\n";
  return OS;
}

class GCNScheduleDAGMILive;
class RegionPressureMap {
  GCNScheduleDAGMILive *DAG;
  // The live in/out pressure as indexed by the first or last MI in the region
  // before scheduling.
  DenseMap<MachineInstr *, GCNRPTracker::LiveRegSet> RegionLiveRegMap;
  // The mapping of RegionIDx to key instruction
  DenseMap<unsigned, MachineInstr *> IdxToInstruction;
  // Whether we are calculating LiveOuts or LiveIns
  bool IsLiveOut;

public:
  RegionPressureMap() = default;
  RegionPressureMap(GCNScheduleDAGMILive *GCNDAG, bool LiveOut)
      : DAG(GCNDAG), IsLiveOut(LiveOut) {}
  // Build the Instr->LiveReg and RegionIdx->Instr maps
  void buildLiveRegMap();

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNScheduleDAGMILive`, `RegionPressureMap`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNScheduleDAGMILive`, `RegionPressureMap`。

### Lines 247-289: Declares class GCNScheduleDAGMILive
```cpp
  // Retrieve the LiveReg for a given RegionIdx
  GCNRPTracker::LiveRegSet &getLiveRegsForRegionIdx(unsigned RegionIdx) {
    assert(IdxToInstruction.contains(RegionIdx));
    MachineInstr *Key = IdxToInstruction[RegionIdx];
    return RegionLiveRegMap[Key];
  }
};

/// A region's boundaries i.e. a pair of instruction bundle iterators. The lower
/// boundary is inclusive, the upper boundary is exclusive.
using RegionBoundaries =
    std::pair<MachineBasicBlock::iterator, MachineBasicBlock::iterator>;

class GCNScheduleDAGMILive final : public ScheduleDAGMILive {
  friend class GCNSchedStage;
  friend class OccInitialScheduleStage;
  friend class RewriteMFMAFormStage;
  friend class UnclusteredHighRPStage;
  friend class ClusteredLowOccStage;
  friend class PreRARematStage;
  friend class ILPInitialScheduleStage;
  friend class RegionPressureMap;

  const GCNSubtarget &ST;

  SIMachineFunctionInfo &MFI;

  // Occupancy target at the beginning of function scheduling cycle.
  unsigned StartingOccupancy;

  // Minimal real occupancy recorder for the function.
  unsigned MinOccupancy;

  // Vector of regions recorder for later rescheduling
  SmallVector<RegionBoundaries, 32> Regions;

  // Record regions with high register pressure.
  BitVector RegionsWithHighRP;

  // Record regions with excess register pressure over the physical register
  // limit. Register pressure in these regions usually will result in spilling.
  BitVector RegionsWithExcessRP;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNScheduleDAGMILive`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNScheduleDAGMILive`。

### Lines 290-331: Result computation and returns
```cpp
  // Regions that have IGLP instructions (SCHED_GROUP_BARRIER or IGLP_OPT).
  BitVector RegionsWithIGLPInstrs;

  // Region live-in cache.
  SmallVector<GCNRPTracker::LiveRegSet, 32> LiveIns;

  // Region pressure cache.
  SmallVector<GCNRegPressure, 32> Pressure;

  // Temporary basic block live-in cache.
  DenseMap<const MachineBasicBlock *, GCNRPTracker::LiveRegSet> MBBLiveIns;

  // The map of the initial first region instruction to region live in registers
  DenseMap<MachineInstr *, GCNRPTracker::LiveRegSet> BBLiveInMap;

  // Calculate the map of the initial first region instruction to region live in
  // registers
  DenseMap<MachineInstr *, GCNRPTracker::LiveRegSet> getRegionLiveInMap() const;

  // Calculate the map of the initial last region instruction to region live out
  // registers
  DenseMap<MachineInstr *, GCNRPTracker::LiveRegSet>
  getRegionLiveOutMap() const;

  // The live out registers per region. These are internally stored as a map of
  // the initial last region instruction to region live out registers, but can
  // be retreived with the regionIdx by calls to getLiveRegsForRegionIdx.
  RegionPressureMap RegionLiveOuts;

  // Return current region pressure.
  GCNRegPressure getRealRegPressure(unsigned RegionIdx) const;

  // Compute and cache live-ins and pressure for all regions in block.
  void computeBlockPressure(unsigned RegionIdx, const MachineBasicBlock *MBB);

  /// Makes the scheduler try to achieve an occupancy of \p TargetOccupancy.
  void setTargetOccupancy(unsigned TargetOccupancy);

  void runSchedStages();

  std::unique_ptr<GCNSchedStage> createSchedStage(GCNSchedStageID SchedStageID);

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

### Lines 332-374: Declares class GCNSchedStage
```cpp
public:
  GCNScheduleDAGMILive(MachineSchedContext *C,
                       std::unique_ptr<MachineSchedStrategy> S);

  void schedule() override;

  void finalizeSchedule() override;
};

// GCNSchedStrategy applies multiple scheduling stages to a function.
class GCNSchedStage {
protected:
  GCNScheduleDAGMILive &DAG;

  GCNSchedStrategy &S;

  MachineFunction &MF;

  SIMachineFunctionInfo &MFI;

  const GCNSubtarget &ST;

  const GCNSchedStageID StageID;

  // The current block being scheduled.
  MachineBasicBlock *CurrentMBB = nullptr;

  // Current region index.
  unsigned RegionIdx = 0;

  // Record the original order of instructions before scheduling.
  std::vector<MachineInstr *> Unsched;

  // RP before scheduling the current region.
  GCNRegPressure PressureBefore;

  // RP after scheduling the current region.
  GCNRegPressure PressureAfter;

  std::vector<std::unique_ptr<ScheduleDAGMutation>> SavedMutations;

  GCNSchedStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNSchedStage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNSchedStage`。

### Lines 375-416: Declares initGCNSchedStage
```cpp
public:
  // Initialize state for a scheduling stage. Returns false if the current stage
  // should be skipped.
  virtual bool initGCNSchedStage();

  // Finalize state after finishing a scheduling pass on the function.
  virtual void finalizeGCNSchedStage();

  // Setup for scheduling a region. Returns false if the current region should
  // be skipped.
  virtual bool initGCNRegion();

  // Finalize state after scheduling a region.
  virtual void finalizeGCNRegion();

  // Track whether a new region is also a new MBB.
  void setupNewBlock();

  // Check result of scheduling.
  void checkScheduling();

  // computes the given schedule virtual execution time in clocks
  ScheduleMetrics getScheduleMetrics(const std::vector<SUnit> &InputSchedule);
  ScheduleMetrics getScheduleMetrics(const GCNScheduleDAGMILive &DAG);
  unsigned computeSUnitReadyCycle(const SUnit &SU, unsigned CurrCycle,
                                  DenseMap<unsigned, unsigned> &ReadyCycles,
                                  const TargetSchedModel &SM);

  // Returns true if scheduling should be reverted.
  virtual bool shouldRevertScheduling(unsigned WavesAfter);

  // Returns true if current region has known excess pressure.
  bool isRegionWithExcessRP() const {
    return DAG.RegionsWithExcessRP[RegionIdx];
  }

  // The region number this stage is currently working on
  unsigned getRegionIdx() { return RegionIdx; }

  // Returns true if the new schedule may result in more spilling.
  bool mayCauseSpilling(unsigned WavesAfter);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 417-456: Declares class OccInitialScheduleStage
```cpp
  /// Sets the schedule of region \p RegionIdx to \p MIOrder. The MIs in \p
  /// MIOrder must be exactly the same as the ones currently existing inside the
  /// region, only in a different order that honors def-use chains.
  void modifyRegionSchedule(unsigned RegionIdx,
                            ArrayRef<MachineInstr *> MIOrder);

  void advanceRegion() { RegionIdx++; }

  virtual ~GCNSchedStage() = default;
};

class OccInitialScheduleStage : public GCNSchedStage {
public:
  bool shouldRevertScheduling(unsigned WavesAfter) override;

  OccInitialScheduleStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG) {}
};

class RewriteMFMAFormStage : public GCNSchedStage {
private:
  // Record regions with excess archvgpr register pressure over the physical
  // register limit. Register pressure in these regions usually will result in
  // spilling.
  BitVector RegionsWithExcessArchVGPR;

  const SIInstrInfo *TII;
  const SIRegisterInfo *SRI;

  /// Do a speculative rewrite and collect copy locations. The speculative
  /// rewrite allows us to calculate the RP of the code after the rewrite, and
  /// the copy locations allow us to calculate the total cost of copies required
  /// for the rewrite. Stores the rewritten instructions in \p RewriteCands ,
  /// the copy locations for uses (of the MFMA result) in \p CopyForUse and the
  /// copy locations for defs (of the MFMA operands) in \p CopyForDef
  bool
  initHeuristics(std::vector<std::pair<MachineInstr *, unsigned>> &RewriteCands,
                 DenseMap<MachineBasicBlock *, std::set<Register>> &CopyForUse,
                 SmallPtrSetImpl<MachineInstr *> &CopyForDef);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `OccInitialScheduleStage`, `RewriteMFMAFormStage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`OccInitialScheduleStage`, `RewriteMFMAFormStage`。

### Lines 457-500: Declares class UnclusteredHighRPStage
```cpp
  /// Calculate the rewrite cost and undo the state change (e.g. rewriting) done
  /// in initHeuristics. Uses \p CopyForUse and \p CopyForDef to calculate copy
  /// costs, and \p RewriteCands to undo rewriting.
  int64_t getRewriteCost(
      const std::vector<std::pair<MachineInstr *, unsigned>> &RewriteCands,
      const DenseMap<MachineBasicBlock *, std::set<Register>> &CopyForUse,
      const SmallPtrSetImpl<MachineInstr *> &CopyForDef);

  /// Do the final rewrite on \p RewriteCands and insert any needed copies.
  bool
  rewrite(const std::vector<std::pair<MachineInstr *, unsigned>> &RewriteCands);

  /// \returns true if this MI is a rewrite candidate.
  bool isRewriteCandidate(MachineInstr *MI) const;

  /// Finds all the reaching defs of \p UseMO and stores the SlotIndexes into \p
  /// DefIdxs
  void findReachingDefs(MachineOperand &UseMO, LiveIntervals *LIS,
                        SmallVectorImpl<SlotIndex> &DefIdxs);

  /// Finds all the reaching uses of \p DefMI and stores the use operands in \p
  /// ReachingUses
  void findReachingUses(MachineInstr *DefMI, LiveIntervals *LIS,
                        SmallVectorImpl<MachineOperand *> &ReachingUses);

public:
  bool initGCNSchedStage() override;

  RewriteMFMAFormStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG) {}
};

class UnclusteredHighRPStage : public GCNSchedStage {
private:
  // Save the initial occupancy before starting this stage.
  unsigned InitialOccupancy;
  // Save the temporary target occupancy before starting this stage.
  unsigned TempTargetOccupancy;
  // Track whether any region was scheduled by this stage.
  bool IsAnyRegionScheduled;

public:
  bool initGCNSchedStage() override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `UnclusteredHighRPStage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`UnclusteredHighRPStage`。

### Lines 501-525: Declares class ClusteredLowOccStage
```cpp
  void finalizeGCNSchedStage() override;

  bool initGCNRegion() override;

  bool shouldRevertScheduling(unsigned WavesAfter) override;

  UnclusteredHighRPStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG) {}
};

// Retry function scheduling if we found resulting occupancy and it is
// lower than used for other scheduling passes. This will give more freedom
// to schedule low register pressure blocks.
class ClusteredLowOccStage : public GCNSchedStage {
public:
  bool initGCNSchedStage() override;

  bool initGCNRegion() override;

  bool shouldRevertScheduling(unsigned WavesAfter) override;

  ClusteredLowOccStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG) {}
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `ClusteredLowOccStage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`ClusteredLowOccStage`。

### Lines 526-564: Declares class PreRARematStage
```cpp
/// Attempts to reduce function spilling or, if there is no spilling, to
/// increase function occupancy by one with respect to register usage by sinking
/// rematerializable instructions to their use. When the stage estimates that
/// reducing spilling or increasing occupancy is possible, it tries to
/// rematerialize as few registers as possible to reduce potential negative
/// effects on function latency.
///
/// The stage only supports rematerializing registers that meet all of the
/// following constraints.
/// 1. The register is virtual and has a single defining instruction.
/// 2. The single defining instruction is either deemed rematerializable by the
///    target-independent logic, or if not, has no non-constant and
///    non-ignorable physical register use.
/// 3  The register has no virtual register use whose live range would be
///    extended by the rematerialization.
/// 4. The register has a single non-debug user in a different region from its
///    defining region.
/// 5. The register is not used by or using another register that is going to be
///    rematerialized.
class PreRARematStage : public GCNSchedStage {
private:
  using RegisterIdx = Rematerializer::RegisterIdx;

  /// A scored rematerialization candidate. Higher scores indicate more
  /// beneficial rematerializations. A null score indicate the rematerialization
  /// is not helpful to reduce RP in target regions.
  struct ScoredRemat {
    /// The register index handle in the rematerializer.
    RegisterIdx RegIdx;
    /// Regions in which the register is live-in/live-out/live anywhere.
    BitVector LiveIn, LiveOut, Live;
    /// Subset of \ref Live regions in which the rematerialization is not
    /// guaranteed to reduce RP (i.e., regions in which the register is not
    /// live-through and unused).
    BitVector UnpredictableRPSave;
    /// Expected register pressure decrease induced by rematerializing this
    /// candidate.
    GCNRegPressure RPSave;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `PreRARematStage`, `ScoredRemat`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`PreRARematStage`, `ScoredRemat`。

### Lines 565-606: Declares struct FreqInfo
```cpp
    /// Execution frequency information required by scoring heuristics.
    /// Frequencies are scaled down if they are high to avoid overflow/underflow
    /// when combining them.
    struct FreqInfo {
      /// Per-region execution frequencies. 0 when unknown.
      SmallVector<uint64_t> Regions;
      /// Minimum and maximum observed frequencies.
      uint64_t MinFreq, MaxFreq;

      FreqInfo(MachineFunction &MF, const GCNScheduleDAGMILive &DAG);

    private:
      static const uint64_t ScaleFactor = 1024;
    };

    /// Initializes the candidate with state-independent characteristics for
    /// rematerializable register with index handle \p RegIdx. This doesn't
    /// update the actual score (call \ref update for this).
    void init(RegisterIdx RegIdx, const FreqInfo &Freq,
              const Rematerializer &Remater, GCNScheduleDAGMILive &DAG);

    /// Rematerializes the candidate using the \p Remater.
    void rematerialize(Rematerializer &Remater) const;

    /// Determines whether this rematerialization may be beneficial in at least
    /// one target region.
    bool maybeBeneficial(const BitVector &TargetRegions,
                         ArrayRef<GCNRPTarget> RPTargets) const;

    /// Rematerializes the candidate and returns the new MI. This removes the
    /// rematerialized register from live-in/out lists in the \p DAG and updates
    /// \p RPTargets in all affected regions. Regions in which RP savings are
    /// not guaranteed are set in \p RecomputeRP.
    MachineInstr *rematerialize(BitVector &RecomputeRP,
                                SmallVectorImpl<GCNRPTarget> &RPTargets,
                                GCNScheduleDAGMILive &DAG) const;

    /// Updates the rematerialization's score w.r.t. the current \p RPTargets.
    /// \p RegionFreq indicates the frequency of each region.
    void update(const BitVector &TargetRegions, ArrayRef<GCNRPTarget> RPTargets,
                const FreqInfo &Freq, bool ReduceSpill);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `FreqInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`FreqInfo`。

### Lines 607-641: Preprocessor guards and macros
```cpp
    /// Returns whether the current score is null, indicating the
    /// rematerialization is useless.
    bool hasNullScore() const { return !RegionImpact; }

    /// Compare score components of non-null scores pair-wise. Scores shouldn't
    /// be null (as defined by \ref hasNullScore).
    bool operator<(const ScoredRemat &O) const {
      assert(!hasNullScore() && "this has null score");
      assert(!O.hasNullScore() && "other has null score");
      if (MaxFreq != O.MaxFreq)
        return MaxFreq < O.MaxFreq;
      if (FreqDiff != O.FreqDiff)
        return FreqDiff < O.FreqDiff;
      if (RegionImpact != O.RegionImpact)
        return RegionImpact < O.RegionImpact;
      // Break ties using register index handles. If the two registers are
      // connected in some dependency DAG of rematerializable registers, this
      // will tend to give a higher score to the register further from the
      // dependency DAG's root. If the two registers are disconnected, this will
      // give a higher score to the register with lower virtual register index.
      // In general, within a region, this should prefer registers defined
      // earlier that have longer live ranges in their defining region (since
      // the registers we consider are always live-out in their defining
      // region).
      return RegIdx > O.RegIdx;
    }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
    Printable print() const;
#endif

  private:
    // The three members below are the scoring components, top to bottom from
    // most important to least important when comparing candidates.

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

### Lines 642-682: Declares struct RollbackSupport
```cpp
    /// Frequency of impacted target region with highest known frequency. This
    /// only matters when the stage is trying to reduce spilling, so it is
    /// always 0 when it is not.
    uint64_t MaxFreq;
    /// Frequency difference between defining and using regions. Negative values
    /// indicate we are rematerializing to higher frequency regions; positive
    /// values indicate the contrary.
    int64_t FreqDiff;
    /// Expected number of target regions impacted by the rematerialization,
    /// scaled by the size of the register being rematerialized.
    unsigned RegionImpact;
  };

  /// Register pressure targets for all regions.
  SmallVector<GCNRPTarget> RPTargets;
  /// Regions which are above the stage's RP target.
  BitVector TargetRegions;
  /// The target occupancy the set is trying to achieve. Empty when the
  /// objective is spilling reduction.
  std::optional<unsigned> TargetOcc;
  /// Achieved occupancy *only* through rematerializations (pre-rescheduling).
  unsigned AchievedOcc;
  /// After successful stage initialization, indicates which regions should be
  /// rescheduled.
  BitVector RescheduleRegions;

  /// Underlying utilities to identify and perform rematerializations.
  Rematerializer Remater;

  struct RollbackSupport {
    struct LiveMapUpdate {
      /// The register index handle in the rematerializer.
      RegisterIdx RegIdx;
      /// Regions in which the original register was live-in or live-out.
      BitVector LiveIn, LiveOut;

      LiveMapUpdate(RegisterIdx RegIdx, const BitVector &LiveIn,
                    const BitVector &LiveOut)
          : RegIdx(RegIdx), LiveIn(LiveIn), LiveOut(LiveOut) {}
    };

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `RollbackSupport`, `LiveMapUpdate`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`RollbackSupport`, `LiveMapUpdate`。

### Lines 683-721: Declares struct RegionSchedRevert
```cpp
    /// Rollback listener.
    Rollbacker Listener;
    /// Registers removed from live-maps along with bitvectors indicationg the
    /// regions in which they were live-ins and live-outs.
    SmallVector<LiveMapUpdate> LiveMapUpdates;

    /// Attaches the rollback listener to the rematerializer.
    RollbackSupport(Rematerializer &Remater) { Remater.addListener(&Listener); }
  };

  /// Rollback support. Maintained through a unique pointer because it is
  /// optional and needs to persist between stage initialization and
  /// finalization.
  std::unique_ptr<RollbackSupport> Rollback;

  /// State of a region pre-re-scheduling but post-rematerializations that we
  /// must keep to be able to revert re-scheduling effects.
  struct RegionSchedRevert {
    /// Region number;
    unsigned RegionIdx;
    /// Original instruction order (both debug and non-debug MIs).
    std::vector<MachineInstr *> OrigMIOrder;
    /// Maximum pressure recorded in the region.
    GCNRegPressure MaxPressure;

    RegionSchedRevert(unsigned RegionIdx, ArrayRef<MachineInstr *> OrigMIOrder,
                      const GCNRegPressure &MaxPressure)
        : RegionIdx(RegionIdx), OrigMIOrder(OrigMIOrder),
          MaxPressure(MaxPressure) {}
  };
  /// After re-scheduling, contains pre-re-scheduling data for all re-scheduled
  /// regions.
  SmallVector<RegionSchedRevert> RegionReverts;
  /// Whether we should revert all re-scheduled regions.
  bool RevertAllRegions = false;

  /// Returns the occupancy the stage is trying to achieve.
  unsigned getStageTargetOccupancy() const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `RegionSchedRevert`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`RegionSchedRevert`。

### Lines 722-760: Declares setObjective
```cpp
  /// Determines the stage's objective (increasing occupancy or reducing
  /// spilling, set in \ref TargetOcc). Defines \ref RPTargets in all regions to
  /// achieve that objective and mark those that don't achieve it in \ref
  /// TargetRegions. Returns whether there is any target region.
  bool setObjective();

  /// In all regions set in \p Regions, saves pressure \p RPSave and clear it as
  /// a target if its RP target has been reached.
  void updateRPTargets(const BitVector &Regions, const GCNRegPressure &RPSave);

  /// Fully recomputes RP from the DAG in \p Regions. Among those regions, sets
  /// again all \ref TargetRegions that were optimistically marked as satisfied
  /// but are actually not, and returns whether there were any such regions.
  bool updateAndVerifyRPTargets(const BitVector &Regions);

  /// Removes register \p Reg from the live-ins of regions set in \p LiveIn and
  /// the live-outs of regions set in \p LiveOut.
  void removeFromLiveMaps(Register Reg, const BitVector &LiveIn,
                          const BitVector &LiveOut);

  /// Adds register \p Reg with mask \p Mask to the live-ins of regions set in
  /// \p LiveIn and the live-outs of regions set in \p LiveOut.
  void addToLiveMaps(Register Reg, LaneBitmask Mask, const BitVector &LiveIn,
                     const BitVector &LiveOut);

  /// If remat alone did not increase occupancy to the target one, rollbacks all
  /// rematerializations and resets live-ins/RP in all regions impacted by the
  /// stage to their pre-stage values.
  void finalizeGCNSchedStage() override;

public:
  bool initGCNSchedStage() override;

  bool initGCNRegion() override;

  void finalizeGCNRegion() override;

  bool shouldRevertScheduling(unsigned WavesAfter) override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 761-804: Declares class ILPInitialScheduleStage
```cpp
  PreRARematStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG), TargetRegions(DAG.Regions.size()),
        RescheduleRegions(DAG.Regions.size()),
        Remater(MF, DAG.Regions, *DAG.LIS) {
    const unsigned NumRegions = DAG.Regions.size();
    RPTargets.reserve(NumRegions);
  }
};

class ILPInitialScheduleStage : public GCNSchedStage {
public:
  bool shouldRevertScheduling(unsigned WavesAfter) override;

  ILPInitialScheduleStage(GCNSchedStageID StageID, GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG) {}
};

class MemoryClauseInitialScheduleStage : public GCNSchedStage {
public:
  bool shouldRevertScheduling(unsigned WavesAfter) override;

  MemoryClauseInitialScheduleStage(GCNSchedStageID StageID,
                                   GCNScheduleDAGMILive &DAG)
      : GCNSchedStage(StageID, DAG) {}
};

class GCNPostScheduleDAGMILive final : public ScheduleDAGMI {
private:
  std::vector<std::unique_ptr<ScheduleDAGMutation>> SavedMutations;

  bool HasIGLPInstrs = false;

public:
  void schedule() override;

  void finalizeSchedule() override;

  GCNPostScheduleDAGMILive(MachineSchedContext *C,
                           std::unique_ptr<MachineSchedStrategy> S,
                           bool RemoveKillFlags);
};

} // End namespace llvm

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `ILPInitialScheduleStage`, `MemoryClauseInitialScheduleStage`, `GCNPostScheduleDAGMILive`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`ILPInitialScheduleStage`, `MemoryClauseInitialScheduleStage`, `GCNPostScheduleDAGMILive`。

### Lines 805-805: Preprocessor guards and macros
```cpp
#endif // LLVM_LIB_TARGET_AMDGPU_GCNSCHEDSTRATEGY_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `SIMachineFunctionInfo`, `SIRegisterInfo`, `GCNSubtarget`, `GCNSchedStage`, `GCNSchedStageID`, `GCNSchedStrategy`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNRegPressure.h"`
- `"llvm/ADT/DenseMap.h"`
- `"llvm/CodeGen/MachineBasicBlock.h"`
- `"llvm/CodeGen/MachineBlockFrequencyInfo.h"`
- `"llvm/CodeGen/MachineInstr.h"`
- `"llvm/CodeGen/MachineScheduler.h"`
- `"llvm/CodeGen/Rematerializer.h"`
