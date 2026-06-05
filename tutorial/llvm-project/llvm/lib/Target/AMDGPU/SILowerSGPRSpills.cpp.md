# SILowerSGPRSpills.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SILowerSGPRSpills.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SILowerSGPRSpills for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SILowerSGPRSpills 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- SILowerSGPRSPills.cpp ---------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Handle SGPR spills. This pass takes the place of PrologEpilogInserter for all
// SGPR spills, so must insert CSR SGPR spills as well as expand them.
//
// This pass must never create new SGPR virtual registers.
//
// FIXME: Must stop RegScavenger spills in later passes.
//
//===----------------------------------------------------------------------===//

#include "SILowerSGPRSpills.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "SISpillUtils.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineCycleAnalysis.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "si-lower-sgpr-spills"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-64: Declares struct LaneVGPRInsertPt
```cpp
using MBBVector = SmallVector<MachineBasicBlock *, 4>;

namespace {

/// Insertion point for IMPLICIT_DEF: iterator may be MBB::end() and can't be
/// dereferenced so the parent block is stored explicitly.
struct LaneVGPRInsertPt {
  MachineBasicBlock *MBB;
  MachineBasicBlock::iterator It;
};

static LaneVGPRInsertPt insertPt(MachineBasicBlock *MBB,
                                 MachineBasicBlock::iterator It) {
  return {MBB, It};
}

static cl::opt<unsigned> MaxNumVGPRsForWwmAllocation(
    "amdgpu-num-vgprs-for-wwm-alloc",
    cl::desc("Max num VGPRs for whole-wave register allocation."),
    cl::ReallyHidden, cl::init(10));

class SILowerSGPRSpills {
private:
  const SIRegisterInfo *TRI = nullptr;
  const SIInstrInfo *TII = nullptr;
  LiveIntervals *LIS = nullptr;
  SlotIndexes *Indexes = nullptr;
  MachineDominatorTree *MDT = nullptr;
  MachineCycleInfo *MCI = nullptr;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `LaneVGPRInsertPt`, `SILowerSGPRSpills`, `MBB::end`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`LaneVGPRInsertPt`, `SILowerSGPRSpills`, `MBB::end`。

### Lines 65-93: Declares class SILowerSGPRSpillsLegacy
```cpp
  // Save and Restore blocks of the current function. Typically there is a
  // single save block, unless Windows EH funclets are involved.
  MBBVector SaveBlocks;
  MBBVector RestoreBlocks;

  MachineBasicBlock *getCycleDomBB(MachineCycle *C);

public:
  SILowerSGPRSpills(LiveIntervals *LIS, SlotIndexes *Indexes,
                    MachineDominatorTree *MDT, MachineCycleInfo *MCI)
      : LIS(LIS), Indexes(Indexes), MDT(MDT), MCI(MCI) {}
  bool run(MachineFunction &MF);
  void calculateSaveRestoreBlocks(MachineFunction &MF);
  bool spillCalleeSavedRegs(MachineFunction &MF,
                            SmallVectorImpl<int> &CalleeSavedFIs);
  void updateLaneVGPRDomInstr(
      int FI, MachineBasicBlock *MBB, MachineBasicBlock::iterator InsertPt,
      DenseMap<Register, LaneVGPRInsertPt> &LaneVGPRDomInstr);
  void determineRegsForWWMAllocation(MachineFunction &MF, BitVector &RegMask);
};

class SILowerSGPRSpillsLegacy : public MachineFunctionPass {
public:
  static char ID;

  SILowerSGPRSpillsLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SILowerSGPRSpillsLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SILowerSGPRSpillsLegacy`。

### Lines 94-121: Registers LLVM passes
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachineCycleInfoWrapperPass>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  MachineFunctionProperties getClearedProperties() const override {
    // SILowerSGPRSpills introduces new Virtual VGPRs for spilling SGPRs.
    return MachineFunctionProperties().setIsSSA().setNoVRegs();
  }
};

} // end anonymous namespace

char SILowerSGPRSpillsLegacy::ID = 0;

INITIALIZE_PASS_BEGIN(SILowerSGPRSpillsLegacy, DEBUG_TYPE,
                      "SI lower SGPR spill instructions", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(SlotIndexesWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineCycleInfoWrapperPass)
INITIALIZE_PASS_END(SILowerSGPRSpillsLegacy, DEBUG_TYPE,
                    "SI lower SGPR spill instructions", false, false)

char &llvm::SILowerSGPRSpillsLegacyID = SILowerSGPRSpillsLegacy::ID;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`。

### Lines 122-151: Defines isLiveIntoMBB
```cpp
static bool isLiveIntoMBB(MCRegister Reg, MachineBasicBlock &MBB,
                          const TargetRegisterInfo *TRI) {
  for (MCRegAliasIterator R(Reg, TRI, true); R.isValid(); ++R) {
    if (MBB.isLiveIn(*R)) {
      return true;
    }
  }
  return false;
}

/// Insert spill code for the callee-saved registers used in the function.
static void insertCSRSaves(MachineBasicBlock &SaveBlock,
                           ArrayRef<CalleeSavedInfo> CSI, SlotIndexes *Indexes,
                           LiveIntervals *LIS) {
  MachineFunction &MF = *SaveBlock.getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const SIRegisterInfo *RI = ST.getRegisterInfo();

  MachineBasicBlock::iterator I = SaveBlock.begin();
  if (!TFI->spillCalleeSavedRegisters(SaveBlock, I, CSI, RI)) {
    for (const CalleeSavedInfo &CS : CSI) {
      // Insert the spill to the stack frame.
      MCRegister Reg = CS.getReg();

      MachineInstrSpan MIS(I, &SaveBlock);
      const TargetRegisterClass *RC = RI->getMinimalPhysRegClass(
          Reg, Reg == RI->getReturnAddressReg(MF) ? MVT::i64 : MVT::i32);

```
**EN:** This section contains concrete logic for isLiveIntoMBB. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isLiveIntoMBB 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 152-179: Conditional logic and checks
```cpp
      // If this value was already livein, we probably have a direct use of the
      // incoming register value, so don't kill at the spill point. This happens
      // since we pass some special inputs (workgroup IDs) in the callee saved
      // range.
      const bool IsLiveIn = isLiveIntoMBB(Reg, SaveBlock, RI);
      TII.storeRegToStackSlot(SaveBlock, I, Reg, !IsLiveIn, CS.getFrameIdx(),
                              RC, Register());

      if (Indexes) {
        assert(std::distance(MIS.begin(), I) == 1);
        MachineInstr &Inst = *std::prev(I);
        Indexes->insertMachineInstrInMaps(Inst);
      }

      if (LIS)
        LIS->removeAllRegUnitsForPhysReg(Reg);
    }
  } else {
    // TFI doesn't update Indexes and LIS, so we have to do it separately.
    if (Indexes)
      Indexes->repairIndexesInRange(&SaveBlock, SaveBlock.begin(), I);

    if (LIS)
      for (const CalleeSavedInfo &CS : CSI)
        LIS->removeAllRegUnitsForPhysReg(CS.getReg());
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::distance`, `std::prev`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::distance`, `std::prev`。

### Lines 180-205: Defines insertCSRRestores
```cpp
/// Insert restore code for the callee-saved registers used in the function.
static void insertCSRRestores(MachineBasicBlock &RestoreBlock,
                              MutableArrayRef<CalleeSavedInfo> CSI,
                              SlotIndexes *Indexes, LiveIntervals *LIS) {
  MachineFunction &MF = *RestoreBlock.getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
  const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
  // Restore all registers immediately before the return and any
  // terminators that precede it.
  MachineBasicBlock::iterator I = RestoreBlock.getFirstTerminator();
  const MachineBasicBlock::iterator BeforeRestoresI =
      I == RestoreBlock.begin() ? I : std::prev(I);

  // FIXME: Just emit the readlane/writelane directly
  if (!TFI->restoreCalleeSavedRegisters(RestoreBlock, I, CSI, TRI)) {
    for (const CalleeSavedInfo &CI : reverse(CSI)) {
      // Insert in reverse order.  loadRegFromStackSlot can insert
      // multiple instructions.
      TFI->restoreCalleeSavedRegister(RestoreBlock, I, CI, &TII, TRI);

      if (Indexes) {
        MachineInstr &Inst = *std::prev(I);
        Indexes->insertMachineInstrInMaps(Inst);
      }

```
**EN:** This section contains concrete logic for insertCSRRestores. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::prev`.
**CN:** 本节包含与 insertCSRRestores 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::prev`。

### Lines 206-229: Conditional logic and checks
```cpp
      if (LIS)
        LIS->removeAllRegUnitsForPhysReg(CI.getReg());
    }
  } else {
    // TFI doesn't update Indexes and LIS, so we have to do it separately.
    if (Indexes)
      Indexes->repairIndexesInRange(&RestoreBlock, BeforeRestoresI,
                                    RestoreBlock.getFirstTerminator());

    if (LIS)
      for (const CalleeSavedInfo &CS : CSI)
        LIS->removeAllRegUnitsForPhysReg(CS.getReg());
  }
}

/// Compute the sets of entry and return blocks for saving and restoring
/// callee-saved registers, and placing prolog and epilog code.
void SILowerSGPRSpills::calculateSaveRestoreBlocks(MachineFunction &MF) {
  const MachineFrameInfo &MFI = MF.getFrameInfo();

  // Even when we do not change any CSR, we still want to insert the
  // prologue and epilogue of the function.
  // So set the save points for those.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SILowerSGPRSpills::calculateSaveRestoreBlocks`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SILowerSGPRSpills::calculateSaveRestoreBlocks`。

### Lines 230-262: Conditional logic and checks
```cpp
  // Use the points found by shrink-wrapping, if any.
  if (!MFI.getSavePoints().empty()) {
    assert(MFI.getSavePoints().size() == 1 &&
           "Multiple save points not yet supported!");
    const auto &SavePoint = *MFI.getSavePoints().begin();
    SaveBlocks.push_back(SavePoint.first);
    assert(MFI.getRestorePoints().size() == 1 &&
           "Multiple restore points not yet supported!");
    const auto &RestorePoint = *MFI.getRestorePoints().begin();
    MachineBasicBlock *RestoreBlock = RestorePoint.first;
    // If RestoreBlock does not have any successor and is not a return block
    // then the end point is unreachable and we do not need to insert any
    // epilogue.
    if (!RestoreBlock->succ_empty() || RestoreBlock->isReturnBlock())
      RestoreBlocks.push_back(RestoreBlock);
    return;
  }

  // Save refs to entry and return blocks.
  SaveBlocks.push_back(&MF.front());
  for (MachineBasicBlock &MBB : MF) {
    if (MBB.isEHFuncletEntry())
      SaveBlocks.push_back(&MBB);
    if (MBB.isReturnBlock())
      RestoreBlocks.push_back(&MBB);
  }
}

// TODO: To support shrink wrapping, this would need to copy
// PrologEpilogInserter's updateLiveness.
static void updateLiveness(MachineFunction &MF, ArrayRef<CalleeSavedInfo> CSI) {
  MachineBasicBlock &EntryBB = MF.front();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 263-292: Declares sortUniqueLiveIns
```cpp
  for (const CalleeSavedInfo &CSIReg : CSI)
    EntryBB.addLiveIn(CSIReg.getReg());
  EntryBB.sortUniqueLiveIns();
}

bool SILowerSGPRSpills::spillCalleeSavedRegs(
    MachineFunction &MF, SmallVectorImpl<int> &CalleeSavedFIs) {
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const Function &F = MF.getFunction();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const SIFrameLowering *TFI = ST.getFrameLowering();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  RegScavenger *RS = nullptr;

  // Determine which of the registers in the callee save list should be saved.
  BitVector SavedRegs;
  TFI->determineCalleeSavesSGPR(MF, SavedRegs, RS);

  // Add the code to save and restore the callee saved registers.
  if (!F.hasFnAttribute(Attribute::Naked)) {
    // FIXME: This is a lie. The CalleeSavedInfo is incomplete, but this is
    // necessary for verifier liveness checks.
    MFI.setCalleeSavedInfoValid(true);

    std::vector<CalleeSavedInfo> CSI;
    const MCPhysReg *CSRegs = MRI.getCalleeSavedRegs();

    for (unsigned I = 0; CSRegs[I]; ++I) {
      MCRegister Reg = CSRegs[I];

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SILowerSGPRSpills::spillCalleeSavedRegs`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SILowerSGPRSpills::spillCalleeSavedRegs`。

### Lines 293-320: Conditional logic and checks
```cpp
      if (SavedRegs.test(Reg)) {
        const TargetRegisterClass *RC =
          TRI->getMinimalPhysRegClass(Reg, MVT::i32);
        int JunkFI = MFI.CreateStackObject(TRI->getSpillSize(*RC),
                                           TRI->getSpillAlign(*RC), true);

        CSI.emplace_back(Reg, JunkFI);
        CalleeSavedFIs.push_back(JunkFI);
      }
    }

    if (!CSI.empty()) {
      for (MachineBasicBlock *SaveBlock : SaveBlocks)
        insertCSRSaves(*SaveBlock, CSI, Indexes, LIS);

      // Add live ins to save blocks.
      assert(SaveBlocks.size() == 1 && "shrink wrapping not fully implemented");
      updateLiveness(MF, CSI);

      for (MachineBasicBlock *RestoreBlock : RestoreBlocks)
        insertCSRRestores(*RestoreBlock, CSI, Indexes, LIS);
      return true;
    }
  }

  return false;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 321-338: Implements SILowerSGPRSpills::getCycleDomBB
```cpp
MachineBasicBlock *SILowerSGPRSpills::getCycleDomBB(MachineCycle *C) {
  // If the insertion point lands on a cycle entry, move it to a block that
  // dominates all entries.
  if (C->isReducible()) {
    if (auto *IDom = MDT->getNode(C->getHeader())->getIDom())
      return IDom->getBlock();
    llvm_unreachable("Expected cycle to have an IDom.");
    return nullptr;
  }

  const SmallVectorImpl<MachineBasicBlock *> &Entries = C->getEntries();
  assert(!Entries.empty() && "Expected cycle to have at least one entry.");
  MachineBasicBlock *EntryBB = Entries[0];
  for (unsigned I = 1; I < Entries.size(); ++I)
    EntryBB = MDT->findNearestCommonDominator(EntryBB, Entries[I]);
  return EntryBB;
}

```
**EN:** This section contains concrete logic for SILowerSGPRSpills::getCycleDomBB. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SILowerSGPRSpills::getCycleDomBB`.
**CN:** 本节包含与 SILowerSGPRSpills::getCycleDomBB 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SILowerSGPRSpills::getCycleDomBB`。

### Lines 339-356: Implements SILowerSGPRSpills::updateLaneVGPRDomInstr
```cpp
void SILowerSGPRSpills::updateLaneVGPRDomInstr(
    int FI, MachineBasicBlock *MBB, MachineBasicBlock::iterator InsertPt,
    DenseMap<Register, LaneVGPRInsertPt> &LaneVGPRDomInstr) {
  // For the Def of a virtual LaneVGPR to dominate all its uses, we should
  // insert an IMPLICIT_DEF before the dominating spill. Switching to a
  // depth first order doesn't really help since the machine function can be in
  // the unstructured control flow post-SSA. For each virtual register, hence
  // finding the common dominator to get either the dominating spill or a block
  // dominating all spills.
  SIMachineFunctionInfo *FuncInfo =
      MBB->getParent()->getInfo<SIMachineFunctionInfo>();
  ArrayRef<SIRegisterInfo::SpilledReg> VGPRSpills =
      FuncInfo->getSGPRSpillToVirtualVGPRLanes(FI);
  Register PrevLaneVGPR;
  for (auto &Spill : VGPRSpills) {
    if (PrevLaneVGPR == Spill.VGPR)
      continue;

```
**EN:** This section contains concrete logic for SILowerSGPRSpills::updateLaneVGPRDomInstr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SILowerSGPRSpills::updateLaneVGPRDomInstr`.
**CN:** 本节包含与 SILowerSGPRSpills::updateLaneVGPRDomInstr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SILowerSGPRSpills::updateLaneVGPRDomInstr`。

### Lines 357-390: Conditional logic and checks
```cpp
    PrevLaneVGPR = Spill.VGPR;
    auto I = LaneVGPRDomInstr.find(Spill.VGPR);
    if (Spill.Lane == 0 && I == LaneVGPRDomInstr.end()) {
      LaneVGPRDomInstr[Spill.VGPR] = insertPt(MBB, InsertPt);
    } else {
      assert(I != LaneVGPRDomInstr.end());
      LaneVGPRInsertPt Prev = I->second;
      MachineBasicBlock *PrevInsertMBB = Prev.MBB;
      MachineBasicBlock::iterator PrevInsertPt = Prev.It;
      MachineBasicBlock *DomMBB = PrevInsertMBB;
      if (DomMBB == MBB) {
        // The insertion point earlier selected in a predecessor block whose
        // spills are currently being lowered. The earlier InsertPt would be
        // the one just before the block terminator and it should be changed
        // if we insert any new spill in it.
        if (PrevInsertPt == MBB->end() ||
            MDT->dominates(&*InsertPt, &*PrevInsertPt))
          I->second = insertPt(MBB, InsertPt);

        continue;
      }

      // Find the common dominator block between PrevInsertPt and the
      // current spill.
      DomMBB = MDT->findNearestCommonDominator(DomMBB, MBB);

      if (DomMBB == MBB)
        I->second = insertPt(MBB, InsertPt);
      else if (DomMBB != PrevInsertMBB)
        I->second = insertPt(DomMBB, DomMBB->getFirstTerminator());
    }
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 391-420: Implements SILowerSGPRSpills::determineRegsForWWMAllocation
```cpp
void SILowerSGPRSpills::determineRegsForWWMAllocation(MachineFunction &MF,
                                                      BitVector &RegMask) {
  // Determine an optimal number of VGPRs for WWM allocation. The complement
  // list will be available for allocating other VGPR virtual registers.
  SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  BitVector ReservedRegs = TRI->getReservedRegs(MF);
  BitVector NonWwmAllocMask(TRI->getNumRegs());
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();

  // FIXME: MaxNumVGPRsForWwmAllocation might need to be adjusted in the future
  // to have a balanced allocation between WWM values and per-thread vector
  // register operands.
  unsigned NumRegs = MaxNumVGPRsForWwmAllocation;
  NumRegs =
      std::min(static_cast<unsigned>(MFI->getSGPRSpillVGPRs().size()), NumRegs);

  auto [MaxNumVGPRs, MaxNumAGPRs] = ST.getMaxNumVectorRegs(MF.getFunction());
  // Try to use the highest available registers for now. Later after
  // vgpr-regalloc, they can be shifted to the lowest range.
  unsigned I = 0;
  for (unsigned Reg = AMDGPU::VGPR0 + MaxNumVGPRs - 1;
       (I < NumRegs) && (Reg >= AMDGPU::VGPR0); --Reg) {
    if (!ReservedRegs.test(Reg) &&
        !MRI.isPhysRegUsed(Reg, /*SkipRegMaskTest=*/true)) {
      TRI->markSuperRegs(RegMask, Reg);
      ++I;
    }
  }

```
**EN:** This section contains concrete logic for SILowerSGPRSpills::determineRegsForWWMAllocation. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SILowerSGPRSpills::determineRegsForWWMAllocation`, `std::min`.
**CN:** 本节包含与 SILowerSGPRSpills::determineRegsForWWMAllocation 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SILowerSGPRSpills::determineRegsForWWMAllocation`, `std::min`。

### Lines 421-453: Implements SILowerSGPRSpillsLegacy::runOnMachineFunction
```cpp
  if (I != NumRegs) {
    // Reserve an arbitrary register and report the error.
    TRI->markSuperRegs(RegMask, AMDGPU::VGPR0);
    MF.getFunction().getContext().emitError(
        "cannot find enough VGPRs for wwm-regalloc");
  }
}

bool SILowerSGPRSpillsLegacy::runOnMachineFunction(MachineFunction &MF) {
  auto *LISWrapper = getAnalysisIfAvailable<LiveIntervalsWrapperPass>();
  LiveIntervals *LIS = LISWrapper ? &LISWrapper->getLIS() : nullptr;
  auto *SIWrapper = getAnalysisIfAvailable<SlotIndexesWrapperPass>();
  SlotIndexes *Indexes = SIWrapper ? &SIWrapper->getSI() : nullptr;
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  MachineCycleInfo *MCI =
      &getAnalysis<MachineCycleInfoWrapperPass>().getCycleInfo();
  return SILowerSGPRSpills(LIS, Indexes, MDT, MCI).run(MF);
}

bool SILowerSGPRSpills::run(MachineFunction &MF) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  TII = ST.getInstrInfo();
  TRI = &TII->getRegisterInfo();

  assert(SaveBlocks.empty() && RestoreBlocks.empty());

  // First, expose any CSR SGPR spills. This is mostly the same as what PEI
  // does, but somewhat simpler.
  calculateSaveRestoreBlocks(MF);
  SmallVector<int> CalleeSavedFIs;
  bool HasCSRs = spillCalleeSavedRegs(MF, CalleeSavedFIs);

```
**EN:** This section contains concrete logic for SILowerSGPRSpillsLegacy::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SILowerSGPRSpillsLegacy::runOnMachineFunction`, `SILowerSGPRSpills::run`.
**CN:** 本节包含与 SILowerSGPRSpillsLegacy::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SILowerSGPRSpillsLegacy::runOnMachineFunction`, `SILowerSGPRSpills::run`。

### Lines 454-483: Declares getFrameInfo
```cpp
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  SIMachineFunctionInfo *FuncInfo = MF.getInfo<SIMachineFunctionInfo>();

  if (!MFI.hasStackObjects() && !HasCSRs) {
    SaveBlocks.clear();
    RestoreBlocks.clear();
    return false;
  }

  bool MadeChange = false;
  bool SpilledToVirtVGPRLanes = false;

  // TODO: CSR VGPRs will never be spilled to AGPRs. These can probably be
  // handled as SpilledToReg in regular PrologEpilogInserter.
  const bool HasSGPRSpillToVGPR = TRI->spillSGPRToVGPR() &&
                                  (HasCSRs || FuncInfo->hasSpilledSGPRs());
  if (HasSGPRSpillToVGPR) {
    // Process all SGPR spills before frame offsets are finalized. Ideally SGPRs
    // are spilled to VGPRs, in which case we can eliminate the stack usage.
    //
    // This operates under the assumption that only other SGPR spills are users
    // of the frame index.

    // To track the spill frame indices handled in this pass.
    BitVector SpillFIs(MFI.getObjectIndexEnd(), false);

    // To track the IMPLICIT_DEF insertion point for the lane vgprs.
    DenseMap<Register, LaneVGPRInsertPt> LaneVGPRDomInstr;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 484-502: Conditional logic and checks
```cpp
    for (MachineBasicBlock &MBB : MF) {
      for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
        if (!TII->isSGPRSpill(MI))
          continue;

        if (MI.getOperand(0).isUndef()) {
          if (Indexes)
            Indexes->removeMachineInstrFromMaps(MI);
          MI.eraseFromParent();
          continue;
        }

        int FI = TII->getNamedOperand(MI, AMDGPU::OpName::addr)->getIndex();
        assert(MFI.getStackID(FI) == TargetStackID::SGPRSpill);

        bool IsCalleeSaveSGPRSpill = llvm::is_contained(CalleeSavedFIs, FI);
        if (IsCalleeSaveSGPRSpill) {
          // Spill callee-saved SGPRs into physical VGPR lanes.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::make_early_inc_range`, `llvm::is_contained`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::make_early_inc_range`, `llvm::is_contained`。

### Lines 503-534: Conditional logic and checks
```cpp
          // TODO: This is to ensure the CFIs are static for efficient frame
          // unwinding in the debugger. Spilling them into virtual VGPR lanes
          // involve regalloc to allocate the physical VGPRs and that might
          // cause intermediate spill/split of such liveranges for successful
          // allocation. This would result in broken CFI encoding unless the
          // regalloc aware CFI generation to insert new CFIs along with the
          // intermediate spills is implemented. There is no such support
          // currently exist in the LLVM compiler.
          if (FuncInfo->allocateSGPRSpillToVGPRLane(
                  MF, FI, /*SpillToPhysVGPRLane=*/true)) {
            bool Spilled = TRI->eliminateSGPRToVGPRSpillFrameIndex(
                MI, FI, nullptr, Indexes, LIS, true);
            if (!Spilled)
              llvm_unreachable(
                  "failed to spill SGPR to physical VGPR lane when allocated");
          }
        } else {
          MachineInstrSpan MIS(&MI, &MBB);
          if (FuncInfo->allocateSGPRSpillToVGPRLane(MF, FI)) {
            bool Spilled = TRI->eliminateSGPRToVGPRSpillFrameIndex(
                MI, FI, nullptr, Indexes, LIS);
            if (!Spilled)
              llvm_unreachable(
                  "failed to spill SGPR to virtual VGPR lane when allocated");
            SpillFIs.set(FI);
            updateLaneVGPRDomInstr(FI, &MBB, MIS.begin(), LaneVGPRDomInstr);
            SpilledToVirtVGPRLanes = true;
          }
        }
      }
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 535-566: Conditional logic and checks
```cpp
    for (auto Reg : FuncInfo->getSGPRSpillVGPRs()) {
      LaneVGPRInsertPt IP = LaneVGPRDomInstr[Reg];
      if (MachineCycle *C = MCI->getTopLevelParentCycle(IP.MBB)) {
        MachineBasicBlock *AdjMBB = getCycleDomBB(C);
        IP = insertPt(AdjMBB, AdjMBB->getFirstTerminator());
      }
      // Insert the IMPLICIT_DEF at the identified points.
      MachineBasicBlock &Block = *IP.MBB;
      DebugLoc DL = Block.findDebugLoc(IP.It);
      auto MIB = BuildMI(Block, IP.It, DL, TII->get(AMDGPU::IMPLICIT_DEF), Reg);

      // Add WWM flag to the virtual register.
      FuncInfo->setFlag(Reg, AMDGPU::VirtRegFlag::WWM_REG);

      // Set SGPR_SPILL asm printer flag
      MIB->setAsmPrinterFlag(AMDGPU::SGPR_SPILL);
      if (LIS) {
        LIS->InsertMachineInstrInMaps(*MIB);
        LIS->createAndComputeVirtRegInterval(Reg);
      }
    }

    // Determine the registers for WWM allocation and also compute the register
    // mask for non-wwm VGPR allocation.
    if (FuncInfo->getSGPRSpillVGPRs().size()) {
      BitVector WwmRegMask(TRI->getNumRegs());

      determineRegsForWWMAllocation(MF, WwmRegMask);

      BitVector NonWwmRegMask(WwmRegMask);
      NonWwmRegMask.flip().clearBitsNotInMask(TRI->getAllVGPRRegMask());

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 567-599: Conditional logic and checks
```cpp
      // The complement set will be the registers for non-wwm (per-thread) vgpr
      // allocation.
      FuncInfo->updateNonWWMRegMask(NonWwmRegMask);
    }

    for (MachineBasicBlock &MBB : MF)
      clearDebugInfoForSpillFIs(MFI, MBB, SpillFIs);

    // All those frame indices which are dead by now should be removed from the
    // function frame. Otherwise, there is a side effect such as re-mapping of
    // free frame index ids by the later pass(es) like "stack slot coloring"
    // which in turn could mess-up with the book keeping of "frame index to VGPR
    // lane".
    FuncInfo->removeDeadFrameIndices(MFI, /*ResetSGPRSpillStackIDs*/ false);

    MadeChange = true;
  }

  if (SpilledToVirtVGPRLanes) {
    const TargetRegisterClass *RC = TRI->getWaveMaskRegClass();
    // Shift back the reserved SGPR for EXEC copy into the lowest range.
    // This SGPR is reserved to handle the whole-wave spill/copy operations
    // that might get inserted during vgpr regalloc.
    Register UnusedLowSGPR = TRI->findUnusedRegister(MRI, RC, MF);
    if (UnusedLowSGPR && TRI->getHWRegIndex(UnusedLowSGPR) <
                             TRI->getHWRegIndex(FuncInfo->getSGPRForEXECCopy()))
      FuncInfo->setSGPRForEXECCopy(UnusedLowSGPR);
  } else {
    // No SGPR spills to virtual VGPR lanes and hence there won't be any WWM
    // spills/copies. Reset the SGPR reserved for EXEC copy.
    FuncInfo->setSGPRForEXECCopy(AMDGPU::NoRegister);
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 600-616: Declares clear
```cpp
  SaveBlocks.clear();
  RestoreBlocks.clear();

  return MadeChange;
}

PreservedAnalyses
SILowerSGPRSpillsPass::run(MachineFunction &MF,
                           MachineFunctionAnalysisManager &MFAM) {
  MFPropsModifier _(*this, MF);
  auto *LIS = MFAM.getCachedResult<LiveIntervalsAnalysis>(MF);
  auto *Indexes = MFAM.getCachedResult<SlotIndexesAnalysis>(MF);
  MachineDominatorTree *MDT = &MFAM.getResult<MachineDominatorTreeAnalysis>(MF);
  MachineCycleInfo &MCI = MFAM.getResult<MachineCycleAnalysis>(MF);
  SILowerSGPRSpills(LIS, Indexes, MDT, &MCI).run(MF);
  return PreservedAnalyses::all();
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SILowerSGPRSpillsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SILowerSGPRSpillsPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `LaneVGPRInsertPt`, `SILowerSGPRSpills`, `SILowerSGPRSpillsLegacy`, `MBB::end`, `cl::desc`, `cl::init`
- **Main themes / 核心主题**: alias analysis / 别名分析; register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SILowerSGPRSpills.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"SISpillUtils.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/MachineCycleAnalysis.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/MachineFrameInfo.h"`
- `"llvm/CodeGen/RegisterScavenging.h"`
- `"llvm/InitializePasses.h"`
