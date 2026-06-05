# GCNIterativeScheduler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNIterativeScheduler.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNIterativeScheduler for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNIterativeScheduler 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===- GCNIterativeScheduler.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the class GCNIterativeScheduler.
///
//===----------------------------------------------------------------------===//

#include "GCNIterativeScheduler.h"
#include "AMDGPUIGroupLP.h"
#include "GCNSchedStrategy.h"
#include "SIMachineFunctionInfo.h"

using namespace llvm;

#define DEBUG_TYPE "machine-scheduler"

namespace llvm {

std::vector<const SUnit *> makeMinRegSchedule(ArrayRef<const SUnit *> TopRoots,
                                              const ScheduleDAG &DAG);

std::vector<const SUnit *> makeGCNILPScheduler(ArrayRef<const SUnit *> BotRoots,
                                               const ScheduleDAG &DAG);
} // namespace llvm

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 32-65: Preprocessor guards and macros
```cpp
// shim accessors for different order containers
static inline MachineInstr *getMachineInstr(MachineInstr *MI) {
  return MI;
}
static inline MachineInstr *getMachineInstr(const SUnit *SU) {
  return SU->getInstr();
}
static inline MachineInstr *getMachineInstr(const SUnit &SU) {
  return SU.getInstr();
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_DUMP_METHOD
static void printRegion(raw_ostream &OS,
                        MachineBasicBlock::iterator Begin,
                        MachineBasicBlock::iterator End,
                        const LiveIntervals *LIS,
                        unsigned MaxInstNum =
                          std::numeric_limits<unsigned>::max()) {
  auto *BB = Begin->getParent();
  OS << BB->getParent()->getName() << ":" << printMBBReference(*BB) << ' '
     << BB->getName() << ":\n";
  auto I = Begin;
  MaxInstNum = std::max(MaxInstNum, 1u);
  for (; I != End && MaxInstNum; ++I, --MaxInstNum) {
    if (!I->isDebugInstr() && LIS)
      OS << LIS->getInstructionIndex(*I);
    OS << '\t' << *I;
  }
  if (I != End) {
    OS << "\t...\n";
    I = std::prev(End);
    if (!I->isDebugInstr() && LIS)
      OS << LIS->getInstructionIndex(*I);
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `std::max`, `std::prev`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`std::max`, `std::prev`。

### Lines 66-90: Defines printLivenessInfo
```cpp
    OS << '\t' << *I;
  }
  if (End != BB->end()) { // print boundary inst if present
    OS << "----\n";
    if (LIS) OS << LIS->getInstructionIndex(*End) << '\t';
    OS << *End;
  }
}

LLVM_DUMP_METHOD
static void printLivenessInfo(raw_ostream &OS,
                              MachineBasicBlock::iterator Begin,
                              MachineBasicBlock::iterator End,
                              const LiveIntervals *LIS) {
  auto *const BB = Begin->getParent();
  const auto &MRI = BB->getParent()->getRegInfo();

  const auto LiveIns = getLiveRegsBefore(*Begin, *LIS);
  OS << "LIn RP: " << print(getRegPressure(MRI, LiveIns));

  const auto BottomMI = End == BB->end() ? std::prev(End) : End;
  const auto LiveOuts = getLiveRegsAfter(*BottomMI, *LIS);
  OS << "LOt RP: " << print(getRegPressure(MRI, LiveOuts));
}

```
**EN:** This section contains concrete logic for printLivenessInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::prev`.
**CN:** 本节包含与 printLivenessInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::prev`。

### Lines 91-121: Preprocessor guards and macros
```cpp
LLVM_DUMP_METHOD
void GCNIterativeScheduler::printRegions(raw_ostream &OS) const {
  const auto &ST = MF.getSubtarget<GCNSubtarget>();
  for (auto *const R : Regions) {
    OS << "Region to schedule ";
    printRegion(OS, R->Begin, R->End, LIS, 1);
    printLivenessInfo(OS, R->Begin, R->End, LIS);
    OS << "Max RP: " << print(R->MaxPressure, &ST);
  }
}

LLVM_DUMP_METHOD
void GCNIterativeScheduler::printSchedResult(raw_ostream &OS,
                                             const Region *R,
                                             const GCNRegPressure &RP) const {
  OS << "\nAfter scheduling ";
  printRegion(OS, R->Begin, R->End, LIS);
  printSchedRP(OS, R->MaxPressure, RP);
  OS << '\n';
}

LLVM_DUMP_METHOD
void GCNIterativeScheduler::printSchedRP(raw_ostream &OS,
                                         const GCNRegPressure &Before,
                                         const GCNRegPressure &After) const {
  const auto &ST = MF.getSubtarget<GCNSubtarget>();
  OS << "RP before: " << print(Before, &ST)
     << "RP after:  " << print(After, &ST);
}
#endif

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `GCNIterativeScheduler::printRegions`, `GCNIterativeScheduler::printSchedResult`, `GCNIterativeScheduler::printSchedRP`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`GCNIterativeScheduler::printRegions`, `GCNIterativeScheduler::printSchedResult`, `GCNIterativeScheduler::printSchedRP`。

### Lines 122-146: Declares class GCNIterativeScheduler
```cpp
void GCNIterativeScheduler::swapIGLPMutations(const Region &R, bool IsReentry) {
  bool HasIGLPInstrs = false;
  const SIInstrInfo *SII = static_cast<const SIInstrInfo *>(TII);
  for (MachineBasicBlock::iterator I = R.Begin; I != R.End; I++) {
    if (SII->isIGLPMutationOnly(I->getOpcode())) {
      HasIGLPInstrs = true;
      break;
    }
  }

  if (HasIGLPInstrs) {
    SavedMutations.clear();
    SavedMutations.swap(Mutations);
    auto SchedPhase = IsReentry ? AMDGPU::SchedulingPhase::PreRAReentry
                                : AMDGPU::SchedulingPhase::Initial;

    addMutation(createIGroupLPDAGMutation(SchedPhase));
  }
}

// DAG builder helper
class GCNIterativeScheduler::BuildDAG {
  GCNIterativeScheduler &Sch;
  SmallVector<SUnit *, 8> TopRoots;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNIterativeScheduler`, `GCNIterativeScheduler::swapIGLPMutations`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNIterativeScheduler`, `GCNIterativeScheduler::swapIGLPMutations`。

### Lines 147-180: Declares class GCNIterativeScheduler
```cpp
  SmallVector<SUnit*, 8> BotRoots;
public:
  BuildDAG(const Region &R, GCNIterativeScheduler &_Sch, bool IsReentry = false)
      : Sch(_Sch) {
    auto *BB = R.Begin->getParent();
    Sch.BaseClass::startBlock(BB);
    Sch.BaseClass::enterRegion(BB, R.Begin, R.End, R.NumRegionInstrs);
    Sch.swapIGLPMutations(R, IsReentry);
    Sch.buildSchedGraph(Sch.AA, nullptr, nullptr, nullptr,
                        /*TrackLaneMask*/true);
    Sch.postProcessDAG();
    Sch.Topo.InitDAGTopologicalSorting();
    Sch.findRootsAndBiasEdges(TopRoots, BotRoots);
  }

  ~BuildDAG() {
    Sch.BaseClass::exitRegion();
    Sch.BaseClass::finishBlock();
  }

  ArrayRef<const SUnit *> getTopRoots() const {
    return TopRoots;
  }
  ArrayRef<SUnit*> getBottomRoots() const {
    return BotRoots;
  }
};

class GCNIterativeScheduler::OverrideLegacyStrategy {
  GCNIterativeScheduler &Sch;
  Region &Rgn;
  std::unique_ptr<MachineSchedStrategy> SaveSchedImpl;
  GCNRegPressure SaveMaxRP;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNIterativeScheduler`, `BaseClass::startBlock`, `BaseClass::enterRegion`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNIterativeScheduler`, `BaseClass::startBlock`, `BaseClass::enterRegion`。

### Lines 181-214: Defines OverrideLegacyStrategy
```cpp
public:
  OverrideLegacyStrategy(Region &R,
                         MachineSchedStrategy &OverrideStrategy,
                         GCNIterativeScheduler &_Sch)
    : Sch(_Sch)
    , Rgn(R)
    , SaveSchedImpl(std::move(_Sch.SchedImpl))
    , SaveMaxRP(R.MaxPressure) {
    Sch.SchedImpl.reset(&OverrideStrategy);
    auto *BB = R.Begin->getParent();
    Sch.BaseClass::startBlock(BB);
    Sch.BaseClass::enterRegion(BB, R.Begin, R.End, R.NumRegionInstrs);
  }

  ~OverrideLegacyStrategy() {
    Sch.BaseClass::exitRegion();
    Sch.BaseClass::finishBlock();
    Sch.SchedImpl.release();
    Sch.SchedImpl = std::move(SaveSchedImpl);
  }

  void schedule() {
    assert(Sch.RegionBegin == Rgn.Begin && Sch.RegionEnd == Rgn.End);
    LLVM_DEBUG(dbgs() << "\nScheduling ";
               printRegion(dbgs(), Rgn.Begin, Rgn.End, Sch.LIS, 2));
    Sch.BaseClass::schedule();

    // Unfortunately placeDebugValues incorrectly modifies RegionEnd, restore
    Sch.RegionEnd = Rgn.End;
    //assert(Rgn.End == Sch.RegionEnd);
    Rgn.Begin = Sch.RegionBegin;
    Rgn.MaxPressure.clear();
  }

```
**EN:** This section contains concrete logic for OverrideLegacyStrategy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::move`, `BaseClass::startBlock`, `BaseClass::enterRegion`.
**CN:** 本节包含与 OverrideLegacyStrategy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::move`, `BaseClass::startBlock`, `BaseClass::enterRegion`。

### Lines 215-246: Declares class SchedStrategyStub
```cpp
  void restoreOrder() {
    assert(Sch.RegionBegin == Rgn.Begin && Sch.RegionEnd == Rgn.End);
    // DAG SUnits are stored using original region's order
    // so just use SUnits as the restoring schedule
    Sch.scheduleRegion(Rgn, Sch.SUnits, SaveMaxRP);
  }
};

namespace {

// just a stub to make base class happy
class SchedStrategyStub : public MachineSchedStrategy {
public:
  bool shouldTrackPressure() const override { return false; }
  bool shouldTrackLaneMasks() const override { return false; }
  void initialize(ScheduleDAGMI *DAG) override {}
  SUnit *pickNode(bool &IsTopNode) override { return nullptr; }
  void schedNode(SUnit *SU, bool IsTopNode) override {}
  void releaseTopNode(SUnit *SU) override {}
  void releaseBottomNode(SUnit *SU) override {}
};

} // end anonymous namespace

GCNIterativeScheduler::GCNIterativeScheduler(MachineSchedContext *C,
                                             StrategyKind S)
  : BaseClass(C, std::make_unique<SchedStrategyStub>())
  , Context(C)
  , Strategy(S)
  , UPTracker(*LIS) {
}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SchedStrategyStub`, `GCNIterativeScheduler::GCNIterativeScheduler`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SchedStrategyStub`, `GCNIterativeScheduler::GCNIterativeScheduler`。

### Lines 247-278: Implements GCNIterativeScheduler::getRegionPressure
```cpp
// returns max pressure for a region
GCNRegPressure
GCNIterativeScheduler::getRegionPressure(MachineBasicBlock::iterator Begin,
                                         MachineBasicBlock::iterator End)
  const {
  // For the purpose of pressure tracking bottom inst of the region should
  // be also processed. End is either BB end, BB terminator inst or sched
  // boundary inst.
  auto const BBEnd = Begin->getParent()->end();
  auto const BottomMI = End == BBEnd ? std::prev(End) : End;

  // scheduleRegions walks bottom to top, so its likely we just get next
  // instruction to track
  auto AfterBottomMI = std::next(BottomMI);
  if (AfterBottomMI == BBEnd ||
      &*AfterBottomMI != UPTracker.getLastTrackedMI()) {
    UPTracker.reset(*BottomMI);
  } else {
    assert(UPTracker.isValid());
  }

  for (auto I = BottomMI; I != Begin; --I)
    UPTracker.recede(*I);

  UPTracker.recede(*Begin);

  assert(UPTracker.isValid() ||
         (dbgs() << "Tracked region ",
          printRegion(dbgs(), Begin, End, LIS), false));
  return UPTracker.getMaxPressureAndReset();
}

```
**EN:** This section contains concrete logic for GCNIterativeScheduler::getRegionPressure. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNIterativeScheduler::getRegionPressure`, `std::prev`, `std::next`.
**CN:** 本节包含与 GCNIterativeScheduler::getRegionPressure 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNIterativeScheduler::getRegionPressure`, `std::prev`, `std::next`。

### Lines 279-312: Implements GCNIterativeScheduler::getSchedulePressure
```cpp
// returns max pressure for a tentative schedule
template <typename Range> GCNRegPressure
GCNIterativeScheduler::getSchedulePressure(const Region &R,
                                           Range &&Schedule) const {
  auto const BBEnd = R.Begin->getParent()->end();
  GCNUpwardRPTracker RPTracker(*LIS);
  if (R.End != BBEnd) {
    // R.End points to the boundary instruction but the
    // schedule doesn't include it
    RPTracker.reset(*R.End);
    RPTracker.recede(*R.End);
  } else {
    // R.End doesn't point to the boundary instruction
    RPTracker.reset(*std::prev(BBEnd));
  }
  for (auto I = Schedule.end(), B = Schedule.begin(); I != B;) {
    RPTracker.recede(*getMachineInstr(*--I));
  }
  return RPTracker.getMaxPressureAndReset();
}

void GCNIterativeScheduler::enterRegion(MachineBasicBlock *BB, // overridden
                                        MachineBasicBlock::iterator Begin,
                                        MachineBasicBlock::iterator End,
                                        unsigned NumRegionInstrs) {
  BaseClass::enterRegion(BB, Begin, End, NumRegionInstrs);
  if (NumRegionInstrs > 2) {
    Regions.push_back(
      new (Alloc.Allocate())
      Region { Begin, End, NumRegionInstrs,
               getRegionPressure(Begin, End), nullptr });
  }
}

```
**EN:** This section contains concrete logic for GCNIterativeScheduler::getSchedulePressure. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNIterativeScheduler::getSchedulePressure`, `std::prev`, `GCNIterativeScheduler::enterRegion`.
**CN:** 本节包含与 GCNIterativeScheduler::getSchedulePressure 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNIterativeScheduler::getSchedulePressure`, `std::prev`, `GCNIterativeScheduler::enterRegion`。

### Lines 313-344: Implements GCNIterativeScheduler::schedule
```cpp
void GCNIterativeScheduler::schedule() { // overridden
  // do nothing
  LLVM_DEBUG(printLivenessInfo(dbgs(), RegionBegin, RegionEnd, LIS);
             if (!Regions.empty() && Regions.back()->Begin == RegionBegin) {
               dbgs() << "Max RP: "
                      << print(Regions.back()->MaxPressure,
                               &MF.getSubtarget<GCNSubtarget>());
             } dbgs()
             << '\n';);
}

void GCNIterativeScheduler::finalizeSchedule() { // overridden
  if (Regions.empty())
    return;
  switch (Strategy) {
  case SCHEDULE_MINREGONLY: scheduleMinReg(); break;
  case SCHEDULE_MINREGFORCED: scheduleMinReg(true); break;
  case SCHEDULE_LEGACYMAXOCCUPANCY: scheduleLegacyMaxOccupancy(); break;
  case SCHEDULE_ILP: scheduleILP(false); break;
  }
}

// Detach schedule from SUnits and interleave it with debug values.
// Returned schedule becomes independent of DAG state.
std::vector<MachineInstr*>
GCNIterativeScheduler::detachSchedule(ScheduleRef Schedule) const {
  std::vector<MachineInstr*> Res;
  Res.reserve(Schedule.size() * 2);

  if (FirstDbgValue)
    Res.push_back(FirstDbgValue);

```
**EN:** This section contains concrete logic for GCNIterativeScheduler::schedule. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNIterativeScheduler::schedule`, `GCNIterativeScheduler::finalizeSchedule`, `GCNIterativeScheduler::detachSchedule`.
**CN:** 本节包含与 GCNIterativeScheduler::schedule 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNIterativeScheduler::schedule`, `GCNIterativeScheduler::finalizeSchedule`, `GCNIterativeScheduler::detachSchedule`。

### Lines 345-369: Declares begin
```cpp
  const auto DbgB = DbgValues.begin(), DbgE = DbgValues.end();
  for (const auto *SU : Schedule) {
    Res.push_back(SU->getInstr());
    const auto &D = std::find_if(DbgB, DbgE, [SU](decltype(*DbgB) &P) {
      return P.second == SU->getInstr();
    });
    if (D != DbgE)
      Res.push_back(D->first);
  }
  return Res;
}

void GCNIterativeScheduler::setBestSchedule(Region &R,
                                            ScheduleRef Schedule,
                                            const GCNRegPressure &MaxRP) {
  R.BestSchedule.reset(
    new TentativeSchedule{ detachSchedule(Schedule), MaxRP });
}

void GCNIterativeScheduler::scheduleBest(Region &R) {
  assert(R.BestSchedule.get() && "No schedule specified");
  scheduleRegion(R, R.BestSchedule->Schedule, R.BestSchedule->MaxPressure);
  R.BestSchedule.reset();
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::find_if`, `GCNIterativeScheduler::setBestSchedule`, `GCNIterativeScheduler::scheduleBest`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::find_if`, `GCNIterativeScheduler::setBestSchedule`, `GCNIterativeScheduler::scheduleBest`。

### Lines 370-398: Preprocessor guards and macros
```cpp
// minimal required region scheduler, works for ranges of SUnits*,
// SUnits or MachineIntrs*
template <typename Range>
void GCNIterativeScheduler::scheduleRegion(Region &R, Range &&Schedule,
                                           const GCNRegPressure &MaxRP) {
  assert(RegionBegin == R.Begin && RegionEnd == R.End);
  assert(LIS != nullptr);
#ifndef NDEBUG
  const auto SchedMaxRP = getSchedulePressure(R, Schedule);
#endif
  auto *BB = R.Begin->getParent();
  auto Top = R.Begin;
  for (const auto &I : Schedule) {
    auto MI = getMachineInstr(I);

    MachineBasicBlock::iterator MII = MI->getIterator();
    if (MII != Top) {
      bool NonDebugReordered =
          !MI->isDebugInstr() && skipDebugInstructionsForward(Top, MII) != MII;
      BB->remove(MI);
      BB->insert(Top, MI);
      if (NonDebugReordered)
        LIS->handleMove(*MI, true);
    }
    if (!MI->isDebugInstr()) {
      // Reset read - undef flags and update them later.
      for (auto &Op : MI->all_defs())
        Op.setIsUndef(false);

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `GCNIterativeScheduler::scheduleRegion`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`GCNIterativeScheduler::scheduleRegion`。

### Lines 399-421: Declares getMachineInstr
```cpp
      RegisterOperands RegOpers;
      RegOpers.collect(*MI, *TRI, MRI, /*ShouldTrackLaneMasks*/true,
                                       /*IgnoreDead*/false);
      // Adjust liveness and add missing dead+read-undef flags.
      auto SlotIdx = LIS->getInstructionIndex(*MI).getRegSlot();
      RegOpers.adjustLaneLiveness(*LIS, MRI, SlotIdx, MI);
    }
    Top = std::next(MI->getIterator());
  }
  RegionBegin = getMachineInstr(Schedule.front());

  // Schedule consisting of MachineInstr* is considered 'detached'
  // and already interleaved with debug values
  if (!std::is_same_v<decltype(*Schedule.begin()), MachineInstr*>) {
    placeDebugValues();
    // Unfortunately placeDebugValues incorrectly modifies RegionEnd, restore
    // assert(R.End == RegionEnd);
    RegionEnd = R.End;
  }

  R.Begin = RegionBegin;
  R.MaxPressure = MaxRP;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::next`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::next`。

### Lines 422-445: Preprocessor guards and macros
```cpp
#ifndef NDEBUG
  const auto RegionMaxRP = getRegionPressure(R);
  const auto &ST = MF.getSubtarget<GCNSubtarget>();
#endif
  assert(
      (SchedMaxRP == RegionMaxRP && (MaxRP.empty() || SchedMaxRP == MaxRP)) ||
      (dbgs() << "Max RP mismatch!!!\n"
                 "RP for schedule (calculated): "
              << print(SchedMaxRP, &ST)
              << "RP for schedule (reported): " << print(MaxRP, &ST)
              << "RP after scheduling: " << print(RegionMaxRP, &ST),
       false));
}

// Sort recorded regions by pressure - highest at the front
void GCNIterativeScheduler::sortRegionsByPressure(unsigned TargetOcc) {
  llvm::sort(Regions, [this, TargetOcc](const Region *R1, const Region *R2) {
    return R2->MaxPressure.less(MF, R1->MaxPressure, TargetOcc);
  });
}

///////////////////////////////////////////////////////////////////////////////
// Legacy MaxOccupancy Strategy

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `GCNIterativeScheduler::sortRegionsByPressure`, `llvm::sort`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`GCNIterativeScheduler::sortRegionsByPressure`, `llvm::sort`。

### Lines 446-477: Implements GCNIterativeScheduler::tryMaximizeOccupancy
```cpp
// Tries to increase occupancy applying minreg scheduler for a sequence of
// most demanding regions. Obtained schedules are saved as BestSchedule for a
// region.
// TargetOcc is the best achievable occupancy for a kernel.
// Returns better occupancy on success or current occupancy on fail.
// BestSchedules aren't deleted on fail.
unsigned GCNIterativeScheduler::tryMaximizeOccupancy(unsigned TargetOcc) {
  // TODO: assert Regions are sorted descending by pressure
  const auto &ST = MF.getSubtarget<GCNSubtarget>();
  const unsigned DynamicVGPRBlockSize =
      MF.getInfo<SIMachineFunctionInfo>()->getDynamicVGPRBlockSize();
  const auto Occ =
      Regions.front()->MaxPressure.getOccupancy(ST, DynamicVGPRBlockSize);
  LLVM_DEBUG(dbgs() << "Trying to improve occupancy, target = " << TargetOcc
                    << ", current = " << Occ << '\n');

  auto NewOcc = TargetOcc;
  for (auto *R : Regions) {
    // Always build the DAG to add mutations
    BuildDAG DAG(*R, *this);

    if (R->MaxPressure.getOccupancy(ST, DynamicVGPRBlockSize) >= NewOcc)
      continue;

    LLVM_DEBUG(printRegion(dbgs(), R->Begin, R->End, LIS, 3);
               printLivenessInfo(dbgs(), R->Begin, R->End, LIS));

    const auto MinSchedule = makeMinRegSchedule(DAG.getTopRoots(), *this);
    const auto MaxRP = getSchedulePressure(*R, MinSchedule);
    LLVM_DEBUG(dbgs() << "Occupancy improvement attempt:\n";
               printSchedRP(dbgs(), R->MaxPressure, MaxRP));

```
**EN:** This section contains concrete logic for GCNIterativeScheduler::tryMaximizeOccupancy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNIterativeScheduler::tryMaximizeOccupancy`.
**CN:** 本节包含与 GCNIterativeScheduler::tryMaximizeOccupancy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNIterativeScheduler::tryMaximizeOccupancy`。

### Lines 478-510: Defines LLVM_DEBUG
```cpp
    NewOcc = std::min(NewOcc, MaxRP.getOccupancy(ST, DynamicVGPRBlockSize));
    if (NewOcc <= Occ)
      break;

    setBestSchedule(*R, MinSchedule, MaxRP);
  }
  LLVM_DEBUG(dbgs() << "New occupancy = " << NewOcc
                    << ", prev occupancy = " << Occ << '\n');
  if (NewOcc > Occ) {
    SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
    MFI->increaseOccupancy(MF, NewOcc);
  }

  return std::max(NewOcc, Occ);
}

void GCNIterativeScheduler::scheduleLegacyMaxOccupancy(
  bool TryMaximizeOccupancy) {
  const auto &ST = MF.getSubtarget<GCNSubtarget>();
  SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  auto TgtOcc = MFI->getMinAllowedOccupancy();
  unsigned DynamicVGPRBlockSize = MFI->getDynamicVGPRBlockSize();

  sortRegionsByPressure(TgtOcc);
  auto Occ =
      Regions.front()->MaxPressure.getOccupancy(ST, DynamicVGPRBlockSize);

  bool IsReentry = false;
  if (TryMaximizeOccupancy && Occ < TgtOcc) {
    Occ = tryMaximizeOccupancy(TgtOcc);
    IsReentry = true;
  }

```
**EN:** This section contains concrete logic for LLVM_DEBUG. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::min`, `std::max`, `GCNIterativeScheduler::scheduleLegacyMaxOccupancy`.
**CN:** 本节包含与 LLVM_DEBUG 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::min`, `std::max`, `GCNIterativeScheduler::scheduleLegacyMaxOccupancy`。

### Lines 511-533: Implements std::min
```cpp
  // This is really weird but for some magic scheduling regions twice
  // gives performance improvement
  const int NumPasses = Occ < TgtOcc ? 2 : 1;

  TgtOcc = std::min(Occ, TgtOcc);
  LLVM_DEBUG(dbgs() << "Scheduling using default scheduler, "
                       "target occupancy = "
                    << TgtOcc << '\n');
  GCNMaxOccupancySchedStrategy LStrgy(Context, /*IsLegacyScheduler=*/true);
  unsigned FinalOccupancy = std::min(Occ, MFI->getOccupancy());

  for (int I = 0; I < NumPasses; ++I) {
    // running first pass with TargetOccupancy = 0 mimics previous scheduling
    // approach and is a performance magic
    LStrgy.setTargetOccupancy(I == 0 ? 0 : TgtOcc);
    for (auto *R : Regions) {
      OverrideLegacyStrategy Ovr(*R, LStrgy, *this);
      IsReentry |= I > 0;
      swapIGLPMutations(*R, IsReentry);
      Ovr.schedule();
      const auto RP = getRegionPressure(*R);
      LLVM_DEBUG(printSchedRP(dbgs(), R->MaxPressure, RP));

```
**EN:** This section contains concrete logic for std::min. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::min`.
**CN:** 本节包含与 std::min 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::min`。

### Lines 534-566: Conditional logic and checks
```cpp
      if (RP.getOccupancy(ST, DynamicVGPRBlockSize) < TgtOcc) {
        LLVM_DEBUG(dbgs() << "Didn't fit into target occupancy O" << TgtOcc);
        if (R->BestSchedule.get() && R->BestSchedule->MaxPressure.getOccupancy(
                                         ST, DynamicVGPRBlockSize) >= TgtOcc) {
          LLVM_DEBUG(dbgs() << ", scheduling minimal register\n");
          scheduleBest(*R);
        } else {
          LLVM_DEBUG(dbgs() << ", restoring\n");
          Ovr.restoreOrder();
          assert(R->MaxPressure.getOccupancy(ST, DynamicVGPRBlockSize) >=
                 TgtOcc);
        }
      }
      FinalOccupancy =
          std::min(FinalOccupancy, RP.getOccupancy(ST, DynamicVGPRBlockSize));
    }
  }
  MFI->limitOccupancy(FinalOccupancy);
}

///////////////////////////////////////////////////////////////////////////////
// Minimal Register Strategy

void GCNIterativeScheduler::scheduleMinReg(bool force) {
  const SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  const auto TgtOcc = MFI->getOccupancy();
  sortRegionsByPressure(TgtOcc);

  auto MaxPressure = Regions.front()->MaxPressure;
  for (auto *R : Regions) {
    if (!force && R->MaxPressure.less(MF, MaxPressure, TgtOcc))
      break;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::min`, `GCNIterativeScheduler::scheduleMinReg`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::min`, `GCNIterativeScheduler::scheduleMinReg`。

### Lines 567-599: Conditional logic and checks
```cpp
    BuildDAG DAG(*R, *this);
    const auto MinSchedule = makeMinRegSchedule(DAG.getTopRoots(), *this);

    const auto RP = getSchedulePressure(*R, MinSchedule);
    LLVM_DEBUG(if (R->MaxPressure.less(MF, RP, TgtOcc)) {
      dbgs() << "\nWarning: Pressure becomes worse after minreg!";
      printSchedRP(dbgs(), R->MaxPressure, RP);
    });

    if (!force && MaxPressure.less(MF, RP, TgtOcc))
      break;

    scheduleRegion(*R, MinSchedule, RP);
    LLVM_DEBUG(printSchedResult(dbgs(), R, RP));

    MaxPressure = RP;
  }
}

///////////////////////////////////////////////////////////////////////////////
// ILP scheduler port

void GCNIterativeScheduler::scheduleILP(
  bool TryMaximizeOccupancy) {
  const auto &ST = MF.getSubtarget<GCNSubtarget>();
  SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  auto TgtOcc = MFI->getMinAllowedOccupancy();
  unsigned DynamicVGPRBlockSize = MFI->getDynamicVGPRBlockSize();

  sortRegionsByPressure(TgtOcc);
  auto Occ =
      Regions.front()->MaxPressure.getOccupancy(ST, DynamicVGPRBlockSize);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNIterativeScheduler::scheduleILP`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNIterativeScheduler::scheduleILP`。

### Lines 600-618: Implements std::min
```cpp
  bool IsReentry = false;
  if (TryMaximizeOccupancy && Occ < TgtOcc) {
    Occ = tryMaximizeOccupancy(TgtOcc);
    IsReentry = true;
  }

  TgtOcc = std::min(Occ, TgtOcc);
  LLVM_DEBUG(dbgs() << "Scheduling using default scheduler, "
                       "target occupancy = "
                    << TgtOcc << '\n');

  unsigned FinalOccupancy = std::min(Occ, MFI->getOccupancy());
  for (auto *R : Regions) {
    BuildDAG DAG(*R, *this, IsReentry);
    const auto ILPSchedule = makeGCNILPScheduler(DAG.getBottomRoots(), *this);

    const auto RP = getSchedulePressure(*R, ILPSchedule);
    LLVM_DEBUG(printSchedRP(dbgs(), R->MaxPressure, RP));

```
**EN:** This section contains concrete logic for std::min. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::min`.
**CN:** 本节包含与 std::min 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::min`。

### Lines 619-634: Conditional logic and checks
```cpp
    if (RP.getOccupancy(ST, DynamicVGPRBlockSize) < TgtOcc) {
      LLVM_DEBUG(dbgs() << "Didn't fit into target occupancy O" << TgtOcc);
      if (R->BestSchedule.get() && R->BestSchedule->MaxPressure.getOccupancy(
                                       ST, DynamicVGPRBlockSize) >= TgtOcc) {
        LLVM_DEBUG(dbgs() << ", scheduling minimal register\n");
        scheduleBest(*R);
      }
    } else {
      scheduleRegion(*R, ILPSchedule, RP);
      LLVM_DEBUG(printSchedResult(dbgs(), R, RP));
      FinalOccupancy =
          std::min(FinalOccupancy, RP.getOccupancy(ST, DynamicVGPRBlockSize));
    }
  }
  MFI->limitOccupancy(FinalOccupancy);
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::min`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::min`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNIterativeScheduler`, `SchedStrategyStub`, `std::max`, `std::prev`, `GCNIterativeScheduler::printRegions`, `GCNIterativeScheduler::printSchedResult`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNIterativeScheduler.h"`
- `"AMDGPUIGroupLP.h"`
- `"GCNSchedStrategy.h"`
- `"SIMachineFunctionInfo.h"`
