# SIFormMemoryClauses.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIFormMemoryClauses.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIFormMemoryClauses for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIFormMemoryClauses 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- SIFormMemoryClauses.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass extends the live ranges of registers used as pointers in
/// sequences of adjacent SMEM and VMEM instructions if XNACK is enabled. A
/// load that would overwrite a pointer would require breaking the soft clause.
/// Artificially extend the live ranges of the pointer operands by adding
/// implicit-def early-clobber operands throughout the soft clause.
///
//===----------------------------------------------------------------------===//

#include "SIFormMemoryClauses.h"
#include "AMDGPU.h"
#include "GCNRegPressure.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "si-form-memory-clauses"

// Clauses longer then 15 instructions would overflow one of the counters
// and stall. They can stall even earlier if there are outstanding counters.
static cl::opt<unsigned>
MaxClause("amdgpu-max-memory-clause", cl::Hidden, cl::init(15),
          cl::desc("Maximum length of a memory clause, instructions"));

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `cl::init`, `cl::desc`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`cl::init`, `cl::desc`。

### Lines 35-67: Declares class SIFormMemoryClausesImpl
```cpp
class SIFormMemoryClausesImpl {
  using RegUse = DenseMap<unsigned, std::pair<RegState, LaneBitmask>>;

  bool canBundle(const MachineInstr &MI, const RegUse &Defs,
                 const RegUse &Uses) const;
  bool checkPressure(const MachineInstr &MI, GCNDownwardRPTracker &RPT);
  void collectRegUses(const MachineInstr &MI, RegUse &Defs, RegUse &Uses) const;
  bool processRegUses(const MachineInstr &MI, RegUse &Defs, RegUse &Uses,
                      GCNDownwardRPTracker &RPT);

  const GCNSubtarget *ST;
  const SIRegisterInfo *TRI;
  const MachineRegisterInfo *MRI;
  SIMachineFunctionInfo *MFI;
  LiveIntervals *LIS;

  unsigned LastRecordedOccupancy;
  unsigned MaxVGPRs;
  unsigned MaxSGPRs;

public:
  SIFormMemoryClausesImpl(LiveIntervals *LS) : LIS(LS) {}
  bool run(MachineFunction &MF);
};

class SIFormMemoryClausesLegacy : public MachineFunctionPass {
public:
  static char ID;

  SIFormMemoryClausesLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIFormMemoryClausesImpl`, `SIFormMemoryClausesLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIFormMemoryClausesImpl`, `SIFormMemoryClausesLegacy`。

### Lines 68-98: Registers LLVM passes
```cpp
  StringRef getPassName() const override {
    return "SI Form memory clauses";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  MachineFunctionProperties getClearedProperties() const override {
    return MachineFunctionProperties().setIsSSA();
  }
};

} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(SIFormMemoryClausesLegacy, DEBUG_TYPE,
                      "SI Form memory clauses", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_END(SIFormMemoryClausesLegacy, DEBUG_TYPE,
                    "SI Form memory clauses", false, false)

char SIFormMemoryClausesLegacy::ID = 0;

char &llvm::SIFormMemoryClausesID = SIFormMemoryClausesLegacy::ID;

FunctionPass *llvm::createSIFormMemoryClausesLegacyPass() {
  return new SIFormMemoryClausesLegacy();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIFormMemoryClausesLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `llvm::createSIFormMemoryClausesLegacyPass`。

### Lines 99-132: Defines isVMEMClauseInst
```cpp
static bool isVMEMClauseInst(const MachineInstr &MI) {
  return SIInstrInfo::isVMEM(MI);
}

static bool isSMEMClauseInst(const MachineInstr &MI) {
  return SIInstrInfo::isSMRD(MI);
}

// There no sense to create store clauses, they do not define anything,
// thus there is nothing to set early-clobber.
static bool isValidClauseInst(const MachineInstr &MI, bool IsVMEMClause) {
  assert(!MI.isDebugInstr() && "debug instructions should not reach here");
  if (MI.isBundled())
    return false;
  if (!MI.mayLoad() || MI.mayStore())
    return false;
  if (SIInstrInfo::isAtomic(MI))
    return false;
  if (IsVMEMClause && !isVMEMClauseInst(MI))
    return false;
  if (!IsVMEMClause && !isSMEMClauseInst(MI))
    return false;
  // If this is a load instruction where the result has been coalesced with an operand, then we cannot clause it.
  for (const MachineOperand &ResMO : MI.defs()) {
    Register ResReg = ResMO.getReg();
    for (const MachineOperand &MO : MI.all_uses()) {
      if (MO.getReg() == ResReg)
        return false;
    }
    break; // Only check the first def.
  }
  return true;
}

```
**EN:** This section contains concrete logic for isVMEMClauseInst. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIInstrInfo::isVMEM`, `SIInstrInfo::isSMRD`, `SIInstrInfo::isAtomic`.
**CN:** 本节包含与 isVMEMClauseInst 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIInstrInfo::isVMEM`, `SIInstrInfo::isSMRD`, `SIInstrInfo::isAtomic`。

### Lines 133-166: Defines getMopState
```cpp
static RegState getMopState(const MachineOperand &MO) {
  RegState S = {};
  if (MO.isImplicit())
    S |= RegState::Implicit;
  if (MO.isDead())
    S |= RegState::Dead;
  if (MO.isUndef())
    S |= RegState::Undef;
  if (MO.isKill())
    S |= RegState::Kill;
  if (MO.isEarlyClobber())
    S |= RegState::EarlyClobber;
  if (MO.getReg().isPhysical() && MO.isRenamable())
    S |= RegState::Renamable;
  return S;
}

// Returns false if there is a use of a def already in the map.
// In this case we must break the clause.
bool SIFormMemoryClausesImpl::canBundle(const MachineInstr &MI,
                                        const RegUse &Defs,
                                        const RegUse &Uses) const {
  // Check interference with defs.
  for (const MachineOperand &MO : MI.operands()) {
    // TODO: Prologue/Epilogue Insertion pass does not process bundled
    //       instructions.
    if (MO.isFI())
      return false;

    if (!MO.isReg())
      continue;

    Register Reg = MO.getReg();

```
**EN:** This section contains concrete logic for getMopState. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIFormMemoryClausesImpl::canBundle`.
**CN:** 本节包含与 getMopState 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIFormMemoryClausesImpl::canBundle`。

### Lines 167-186: Conditional logic and checks
```cpp
    // If it is tied we will need to write same register as we read.
    if (MO.isTied())
      return false;

    const RegUse &Map = MO.isDef() ? Uses : Defs;
    auto Conflict = Map.find(Reg);
    if (Conflict == Map.end())
      continue;

    if (Reg.isPhysical())
      return false;

    LaneBitmask Mask = TRI->getSubRegIndexLaneMask(MO.getSubReg());
    if ((Conflict->second.second & Mask).any())
      return false;
  }

  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 187-219: Implements SIFormMemoryClausesImpl::checkPressure
```cpp
// Since all defs in the clause are early clobber we can run out of registers.
// Function returns false if pressure would hit the limit if instruction is
// bundled into a memory clause.
bool SIFormMemoryClausesImpl::checkPressure(const MachineInstr &MI,
                                            GCNDownwardRPTracker &RPT) {
  // NB: skip advanceBeforeNext() call. Since all defs will be marked
  // early-clobber they will all stay alive at least to the end of the
  // clause. Therefor we should not decrease pressure even if load
  // pointer becomes dead and could otherwise be reused for destination.
  RPT.advanceToNext();
  GCNRegPressure MaxPressure = RPT.moveMaxPressure();
  unsigned Occupancy = MaxPressure.getOccupancy(
      *ST,
      MI.getMF()->getInfo<SIMachineFunctionInfo>()->getDynamicVGPRBlockSize());

  // Don't push over half the register budget. We don't want to introduce
  // spilling just to form a soft clause.
  //
  // FIXME: This pressure check is fundamentally broken. First, this is checking
  // the global pressure, not the pressure at this specific point in the
  // program. Second, it's not accounting for the increased liveness of the use
  // operands due to the early clobber we will introduce. Third, the pressure
  // tracking does not account for the alignment requirements for SGPRs, or the
  // fragmentation of registers the allocator will need to satisfy.
  if (Occupancy >= MFI->getMinAllowedOccupancy() &&
      MaxPressure.getVGPRNum(ST->hasGFX90AInsts()) <= MaxVGPRs / 2 &&
      MaxPressure.getSGPRNum() <= MaxSGPRs / 2) {
    LastRecordedOccupancy = Occupancy;
    return true;
  }
  return false;
}

```
**EN:** This section contains concrete logic for SIFormMemoryClausesImpl::checkPressure. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIFormMemoryClausesImpl::checkPressure`.
**CN:** 本节包含与 SIFormMemoryClausesImpl::checkPressure 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIFormMemoryClausesImpl::checkPressure`。

### Lines 220-252: Implements SIFormMemoryClausesImpl::collectRegUses
```cpp
// Collect register defs and uses along with their lane masks and states.
void SIFormMemoryClausesImpl::collectRegUses(const MachineInstr &MI,
                                             RegUse &Defs, RegUse &Uses) const {
  for (const MachineOperand &MO : MI.operands()) {
    if (!MO.isReg())
      continue;
    Register Reg = MO.getReg();
    if (!Reg)
      continue;

    LaneBitmask Mask = Reg.isVirtual()
                           ? TRI->getSubRegIndexLaneMask(MO.getSubReg())
                           : LaneBitmask::getAll();
    RegUse &Map = MO.isDef() ? Defs : Uses;

    RegState State = getMopState(MO);
    auto [Loc, Inserted] = Map.try_emplace(Reg, State, Mask);
    if (!Inserted) {
      Loc->second.first |= State;
      Loc->second.second |= Mask;
    }
  }
}

// Check register def/use conflicts, occupancy limits and collect def/use maps.
// Return true if instruction can be bundled with previous. If it cannot
// def/use maps are not updated.
bool SIFormMemoryClausesImpl::processRegUses(const MachineInstr &MI,
                                             RegUse &Defs, RegUse &Uses,
                                             GCNDownwardRPTracker &RPT) {
  if (!canBundle(MI, Defs, Uses))
    return false;

```
**EN:** This section contains concrete logic for SIFormMemoryClausesImpl::collectRegUses. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIFormMemoryClausesImpl::collectRegUses`, `LaneBitmask::getAll`, `SIFormMemoryClausesImpl::processRegUses`.
**CN:** 本节包含与 SIFormMemoryClausesImpl::collectRegUses 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIFormMemoryClausesImpl::collectRegUses`, `LaneBitmask::getAll`, `SIFormMemoryClausesImpl::processRegUses`。

### Lines 253-286: Declares collectRegUses
```cpp
  if (!checkPressure(MI, RPT))
    return false;

  collectRegUses(MI, Defs, Uses);
  return true;
}

bool SIFormMemoryClausesImpl::run(MachineFunction &MF) {
  ST = &MF.getSubtarget<GCNSubtarget>();
  if (!ST->isXNACKEnabled())
    return false;

  const SIInstrInfo *TII = ST->getInstrInfo();
  TRI = ST->getRegisterInfo();
  MRI = &MF.getRegInfo();
  MFI = MF.getInfo<SIMachineFunctionInfo>();
  SlotIndexes *Ind = LIS->getSlotIndexes();
  bool Changed = false;

  MaxVGPRs = TRI->getAllocatableSet(MF, &AMDGPU::VGPR_32RegClass).count();
  MaxSGPRs = TRI->getAllocatableSet(MF, &AMDGPU::SGPR_32RegClass).count();
  unsigned FuncMaxClause = MF.getFunction().getFnAttributeAsParsedInteger(
      "amdgpu-max-memory-clause", MaxClause);

  for (MachineBasicBlock &MBB : MF) {
    GCNDownwardRPTracker RPT(*LIS);
    MachineBasicBlock::instr_iterator Next;
    for (auto I = MBB.instr_begin(), E = MBB.instr_end(); I != E; I = Next) {
      MachineInstr &MI = *I;
      Next = std::next(I);

      if (MI.isMetaInstruction())
        continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SIFormMemoryClausesImpl::run`, `std::next`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SIFormMemoryClausesImpl::run`, `std::next`。

### Lines 287-315: Conditional logic and checks
```cpp
      bool IsVMEM = isVMEMClauseInst(MI);

      if (!isValidClauseInst(MI, IsVMEM))
        continue;

      if (!RPT.getNext().isValid())
        RPT.reset(MI, MBB.end());
      else { // Advance the state to the current MI.
        RPT.advance(MachineBasicBlock::const_iterator(MI));
        RPT.advanceBeforeNext();
      }

      const GCNRPTracker::LiveRegSet LiveRegsCopy(RPT.getLiveRegs());
      RegUse Defs, Uses;
      if (!processRegUses(MI, Defs, Uses, RPT)) {
        RPT.reset(MI, MBB.end(), &LiveRegsCopy);
        continue;
      }

      MachineBasicBlock::iterator LastClauseInst = Next;
      unsigned Length = 1;
      for ( ; Next != E && Length < FuncMaxClause; ++Next) {
        // Debug instructions should not change the kill insertion.
        if (Next->isMetaInstruction())
          continue;

        if (!isValidClauseInst(*Next, IsVMEM))
          break;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MachineBasicBlock::const_iterator`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MachineBasicBlock::const_iterator`。

### Lines 316-348: Conditional logic and checks
```cpp
        // A load from pointer which was loaded inside the same bundle is an
        // impossible clause because we will need to write and read the same
        // register inside. In this case processRegUses will return false.
        if (!processRegUses(*Next, Defs, Uses, RPT))
          break;

        LastClauseInst = Next;
        ++Length;
      }
      if (Length < 2) {
        RPT.reset(MI, MBB.end(), &LiveRegsCopy);
        continue;
      }

      Changed = true;
      MFI->limitOccupancy(LastRecordedOccupancy);

      assert(!LastClauseInst->isMetaInstruction());

      SlotIndex ClauseLiveInIdx = LIS->getInstructionIndex(MI);
      SlotIndex ClauseLiveOutIdx =
          LIS->getInstructionIndex(*LastClauseInst).getNextIndex();

      // Track the last inserted kill.
      MachineInstrBuilder Kill;

      // Insert one kill per register, with operands covering all necessary
      // subregisters.
      for (auto &&R : Uses) {
        Register Reg = R.first;
        if (Reg.isPhysical())
          continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 349-380: Conditional logic and checks
```cpp
        // Collect the register operands we should extend the live ranges of.
        SmallVector<std::tuple<RegState, unsigned>> KillOps;
        const LiveInterval &LI = LIS->getInterval(R.first);

        if (!LI.hasSubRanges()) {
          if (!LI.liveAt(ClauseLiveOutIdx)) {
            KillOps.emplace_back(R.second.first | RegState::Kill,
                                 AMDGPU::NoSubRegister);
          }
        } else {
          LaneBitmask KilledMask;
          for (const LiveInterval::SubRange &SR : LI.subranges()) {
            if (SR.liveAt(ClauseLiveInIdx) && !SR.liveAt(ClauseLiveOutIdx))
              KilledMask |= SR.LaneMask;
          }

          if (KilledMask.none())
            continue;

          SmallVector<unsigned> KilledIndexes;
          bool Success = TRI->getCoveringSubRegIndexes(
              MRI->getRegClass(Reg), KilledMask, KilledIndexes);
          (void)Success;
          assert(Success && "Failed to find subregister mask to cover lanes");
          for (unsigned SubReg : KilledIndexes) {
            KillOps.emplace_back(R.second.first | RegState::Kill, SubReg);
          }
        }

        if (KillOps.empty())
          continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 381-407: Conditional logic and checks
```cpp
        // We only want to extend the live ranges of used registers. If they
        // already have existing uses beyond the bundle, we don't need the kill.
        //
        // It's possible all of the use registers were already live past the
        // bundle.
        Kill = BuildMI(*MI.getParent(), std::next(LastClauseInst),
                       DebugLoc(), TII->get(AMDGPU::KILL));
        for (auto &Op : KillOps)
          Kill.addUse(Reg, std::get<0>(Op), std::get<1>(Op));
        Ind->insertMachineInstrInMaps(*Kill);
      }

      // Restore the state after processing the end of the bundle.
      RPT.reset(MI, MBB.end(), &LiveRegsCopy);

      if (!Kill)
        continue;

      for (auto &&R : Defs) {
        Register Reg = R.first;
        Uses.erase(Reg);
        if (Reg.isPhysical())
          continue;
        LIS->removeInterval(Reg);
        LIS->createAndComputeVirtRegInterval(Reg);
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 408-435: Conditional logic and checks
```cpp
      for (auto &&R : Uses) {
        Register Reg = R.first;
        if (Reg.isPhysical())
          continue;
        LIS->removeInterval(Reg);
        LIS->createAndComputeVirtRegInterval(Reg);
      }
    }
  }

  return Changed;
}

bool SIFormMemoryClausesLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  LiveIntervals *LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  return SIFormMemoryClausesImpl(LIS).run(MF);
}

PreservedAnalyses
SIFormMemoryClausesPass::run(MachineFunction &MF,
                             MachineFunctionAnalysisManager &MFAM) {
  LiveIntervals &LIS = MFAM.getResult<LiveIntervalsAnalysis>(MF);
  SIFormMemoryClausesImpl(&LIS).run(MF);
  return PreservedAnalyses::all();
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIFormMemoryClausesLegacy::runOnMachineFunction`, `SIFormMemoryClausesPass::run`, `PreservedAnalyses::all`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIFormMemoryClausesLegacy::runOnMachineFunction`, `SIFormMemoryClausesPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIFormMemoryClausesImpl`, `SIFormMemoryClausesLegacy`, `cl::init`, `cl::desc`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIFormMemoryClausesLegacyPass`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIFormMemoryClauses.h"`
- `"AMDGPU.h"`
- `"GCNRegPressure.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/InitializePasses.h"`
