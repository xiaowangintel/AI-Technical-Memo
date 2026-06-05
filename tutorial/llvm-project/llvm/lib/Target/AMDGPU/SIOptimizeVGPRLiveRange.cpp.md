# SIOptimizeVGPRLiveRange.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIOptimizeVGPRLiveRange.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIOptimizeVGPRLiveRange for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIOptimizeVGPRLiveRange 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, license, and overview
```cpp
//===--------------------- SIOptimizeVGPRLiveRange.cpp  -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass tries to remove unnecessary VGPR live ranges in divergent if-else
/// structures and waterfall loops.
///
/// When we do structurization, we usually transform an if-else into two
/// successive if-then (with a flow block to do predicate inversion). Consider a
/// simple case after structurization: A divergent value %a was defined before
/// if-else and used in both THEN (use in THEN is optional) and ELSE part:
///    bb.if:
///      %a = ...
///      ...
///    bb.then:
///      ... = op %a
///      ... // %a can be dead here
///    bb.flow:
///      ...
///    bb.else:
///      ... = %a
///      ...
///    bb.endif
///
///  As register allocator has no idea of the thread-control-flow, it will just
///  assume %a would be alive in the whole range of bb.then because of a later
///  use in bb.else. On AMDGPU architecture, the VGPR is accessed with respect
///  to exec mask. For this if-else case, the lanes active in bb.then will be
///  inactive in bb.else, and vice-versa. So we are safe to say that %a was dead
```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 35-68: Implementation details and local logic
```cpp
///  after the last use in bb.then until the end of the block. The reason is
///  the instructions in bb.then will only overwrite lanes that will never be
///  accessed in bb.else.
///
///  This pass aims to tell register allocator that %a is in-fact dead,
///  through inserting a phi-node in bb.flow saying that %a is undef when coming
///  from bb.then, and then replace the uses in the bb.else with the result of
///  newly inserted phi.
///
///  Two key conditions must be met to ensure correctness:
///  1.) The def-point should be in the same loop-level as if-else-endif to make
///      sure the second loop iteration still get correct data.
///  2.) There should be no further uses after the IF-ELSE region.
///
///
/// Waterfall loops get inserted around instructions that use divergent values
/// but can only be executed with a uniform value. For example an indirect call
/// to a divergent address:
///    bb.start:
///      %a = ...
///      %fun = ...
///      ...
///    bb.loop:
///      call %fun (%a)
///      ... // %a can be dead here
///      loop %bb.loop
///
///  The loop block is executed multiple times, but it is run exactly once for
///  each active lane. Similar to the if-else case, the register allocator
///  assumes that %a is live throughout the loop as it is used again in the next
///  iteration. If %a is a VGPR that is unused after the loop, it does not need
///  to be live after its last use in the loop block. By inserting a phi-node at
///  the start of bb.loop that is undef when coming from bb.loop, the register
///  allocation knows that the value of %a does not need to be preserved through
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 69-100: Header dependencies and setup
```cpp
///  iterations of the loop.
///
//
//===----------------------------------------------------------------------===//

#include "SIOptimizeVGPRLiveRange.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/LiveVariables.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "si-opt-vgpr-liverange"

namespace {

class SIOptimizeVGPRLiveRange {
private:
  const SIRegisterInfo *TRI = nullptr;
  const SIInstrInfo *TII = nullptr;
  LiveVariables *LV = nullptr;
  MachineDominatorTree *MDT = nullptr;
  const MachineLoopInfo *Loops = nullptr;
  MachineRegisterInfo *MRI = nullptr;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `SIOptimizeVGPRLiveRange`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`SIOptimizeVGPRLiveRange`。

### Lines 101-130: Defines SIOptimizeVGPRLiveRange
```cpp
public:
  SIOptimizeVGPRLiveRange(LiveVariables *LV, MachineDominatorTree *MDT,
                          MachineLoopInfo *Loops)
      : LV(LV), MDT(MDT), Loops(Loops) {}
  bool run(MachineFunction &MF);

  MachineBasicBlock *getElseTarget(MachineBasicBlock *MBB) const;

  void collectElseRegionBlocks(MachineBasicBlock *Flow,
                               MachineBasicBlock *Endif,
                               SmallSetVector<MachineBasicBlock *, 16> &) const;

  void
  collectCandidateRegisters(MachineBasicBlock *If, MachineBasicBlock *Flow,
                            MachineBasicBlock *Endif,
                            SmallSetVector<MachineBasicBlock *, 16> &ElseBlocks,
                            SmallVectorImpl<Register> &CandidateRegs) const;

  void collectWaterfallCandidateRegisters(
      MachineBasicBlock *LoopHeader, MachineBasicBlock *LoopEnd,
      SmallSetVector<Register, 16> &CandidateRegs,
      SmallSetVector<MachineBasicBlock *, 2> &Blocks,
      SmallVectorImpl<MachineInstr *> &Instructions) const;

  void findNonPHIUsesInBlock(Register Reg, MachineBasicBlock *MBB,
                             SmallVectorImpl<MachineInstr *> &Uses) const;

  void updateLiveRangeInThenRegion(Register Reg, MachineBasicBlock *If,
                                   MachineBasicBlock *Flow) const;

```
**EN:** This section contains concrete logic for SIOptimizeVGPRLiveRange. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 SIOptimizeVGPRLiveRange 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 131-158: Declares class SIOptimizeVGPRLiveRangeLegacy
```cpp
  void updateLiveRangeInElseRegion(
      Register Reg, Register NewReg, MachineBasicBlock *Flow,
      MachineBasicBlock *Endif,
      SmallSetVector<MachineBasicBlock *, 16> &ElseBlocks) const;

  void
  optimizeLiveRange(Register Reg, MachineBasicBlock *If,
                    MachineBasicBlock *Flow, MachineBasicBlock *Endif,
                    SmallSetVector<MachineBasicBlock *, 16> &ElseBlocks) const;

  void optimizeWaterfallLiveRange(
      Register Reg, MachineBasicBlock *LoopHeader,
      SmallSetVector<MachineBasicBlock *, 2> &LoopBlocks,
      SmallVectorImpl<MachineInstr *> &Instructions) const;
};

class SIOptimizeVGPRLiveRangeLegacy : public MachineFunctionPass {
public:
  static char ID;

  SIOptimizeVGPRLiveRangeLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "SI Optimize VGPR LiveRange";
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIOptimizeVGPRLiveRangeLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIOptimizeVGPRLiveRangeLegacy`。

### Lines 159-191: Defines getAnalysisUsage
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<LiveVariablesWrapperPass>();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachineLoopInfoWrapperPass>();
    AU.addPreserved<LiveVariablesWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setIsSSA();
  }

  MachineFunctionProperties getClearedProperties() const override {
    return MachineFunctionProperties().setNoPHIs();
  }
};

} // end anonymous namespace

// Check whether the MBB is a else flow block and get the branching target which
// is the Endif block
MachineBasicBlock *
SIOptimizeVGPRLiveRange::getElseTarget(MachineBasicBlock *MBB) const {
  for (auto &BR : MBB->terminators()) {
    if (BR.getOpcode() == AMDGPU::SI_ELSE)
      return BR.getOperand(2).getMBB();
  }
  return nullptr;
}

```
**EN:** This section contains concrete logic for getAnalysisUsage. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `SIOptimizeVGPRLiveRange::getElseTarget`.
**CN:** 本节包含与 getAnalysisUsage 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `SIOptimizeVGPRLiveRange::getElseTarget`。

### Lines 192-218: Implements SIOptimizeVGPRLiveRange::collectElseRegionBlocks
```cpp
void SIOptimizeVGPRLiveRange::collectElseRegionBlocks(
    MachineBasicBlock *Flow, MachineBasicBlock *Endif,
    SmallSetVector<MachineBasicBlock *, 16> &Blocks) const {
  assert(Flow != Endif);

  MachineBasicBlock *MBB = Endif;
  unsigned Cur = 0;
  while (MBB) {
    for (auto *Pred : MBB->predecessors()) {
      if (Pred != Flow)
        Blocks.insert(Pred);
    }

    if (Cur < Blocks.size())
      MBB = Blocks[Cur++];
    else
      MBB = nullptr;
  }

  LLVM_DEBUG({
    dbgs() << "Found Else blocks: ";
    for (auto *MBB : Blocks)
      dbgs() << printMBBReference(*MBB) << ' ';
    dbgs() << '\n';
  });
}

```
**EN:** This section contains concrete logic for SIOptimizeVGPRLiveRange::collectElseRegionBlocks. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIOptimizeVGPRLiveRange::collectElseRegionBlocks`.
**CN:** 本节包含与 SIOptimizeVGPRLiveRange::collectElseRegionBlocks 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIOptimizeVGPRLiveRange::collectElseRegionBlocks`。

### Lines 219-251: Implements SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock
```cpp
/// Find the instructions(excluding phi) in \p MBB that uses the \p Reg.
void SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock(
    Register Reg, MachineBasicBlock *MBB,
    SmallVectorImpl<MachineInstr *> &Uses) const {
  for (auto &UseMI : MRI->use_nodbg_instructions(Reg)) {
    if (UseMI.getParent() == MBB && !UseMI.isPHI())
      Uses.push_back(&UseMI);
  }
}

/// Collect the killed registers in the ELSE region which are not alive through
/// the whole THEN region.
void SIOptimizeVGPRLiveRange::collectCandidateRegisters(
    MachineBasicBlock *If, MachineBasicBlock *Flow, MachineBasicBlock *Endif,
    SmallSetVector<MachineBasicBlock *, 16> &ElseBlocks,
    SmallVectorImpl<Register> &CandidateRegs) const {

  SmallSet<Register, 8> KillsInElse;

  for (auto *Else : ElseBlocks) {
    for (auto &MI : Else->instrs()) {
      if (MI.isDebugInstr())
        continue;

      for (auto &MO : MI.operands()) {
        if (!MO.isReg() || !MO.getReg() || MO.isDef())
          continue;

        Register MOReg = MO.getReg();
        // We can only optimize AGPR/VGPR virtual register
        if (MOReg.isPhysical() || !TRI->isVectorRegister(*MRI, MOReg))
          continue;

```
**EN:** This section contains concrete logic for SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock`, `SIOptimizeVGPRLiveRange::collectCandidateRegisters`.
**CN:** 本节包含与 SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock`, `SIOptimizeVGPRLiveRange::collectCandidateRegisters`。

### Lines 252-284: Conditional logic and checks
```cpp
        if (MO.readsReg()) {
          LiveVariables::VarInfo &VI = LV->getVarInfo(MOReg);
          const MachineBasicBlock *DefMBB = MRI->getVRegDef(MOReg)->getParent();
          // Make sure two conditions are met:
          // a.) the value is defined before/in the IF block
          // b.) should be defined in the same loop-level.
          if ((VI.AliveBlocks.test(If->getNumber()) || DefMBB == If) &&
              Loops->getLoopFor(DefMBB) == Loops->getLoopFor(If)) {
            // Check if the register is live into the endif block. If not,
            // consider it killed in the else region.
            LiveVariables::VarInfo &VI = LV->getVarInfo(MOReg);
            if (!VI.isLiveIn(*Endif, MOReg, *MRI)) {
              KillsInElse.insert(MOReg);
            } else {
              LLVM_DEBUG(dbgs() << "Excluding " << printReg(MOReg, TRI)
                                << " as Live in Endif\n");
            }
          }
        }
      }
    }
  }

  // Check the phis in the Endif, looking for value coming from the ELSE
  // region. Make sure the phi-use is the last use.
  for (auto &MI : Endif->phis()) {
    for (unsigned Idx = 1; Idx < MI.getNumOperands(); Idx += 2) {
      auto &MO = MI.getOperand(Idx);
      auto *Pred = MI.getOperand(Idx + 1).getMBB();
      if (Pred == Flow)
        continue;
      assert(ElseBlocks.contains(Pred) && "Should be from Else region\n");

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 285-308: Conditional logic and checks
```cpp
      if (!MO.isReg() || !MO.getReg() || MO.isUndef())
        continue;

      Register Reg = MO.getReg();
      if (Reg.isPhysical() || !TRI->isVectorRegister(*MRI, Reg))
        continue;

      LiveVariables::VarInfo &VI = LV->getVarInfo(Reg);

      if (VI.isLiveIn(*Endif, Reg, *MRI)) {
        LLVM_DEBUG(dbgs() << "Excluding " << printReg(Reg, TRI)
                          << " as Live in Endif\n");
        continue;
      }
      // Make sure two conditions are met:
      // a.) the value is defined before/in the IF block
      // b.) should be defined in the same loop-level.
      const MachineBasicBlock *DefMBB = MRI->getVRegDef(Reg)->getParent();
      if ((VI.AliveBlocks.test(If->getNumber()) || DefMBB == If) &&
          Loops->getLoopFor(DefMBB) == Loops->getLoopFor(If))
        KillsInElse.insert(Reg);
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 309-336: Conditional logic and checks
```cpp
  auto IsLiveThroughThen = [&](Register Reg) {
    for (auto I = MRI->use_nodbg_begin(Reg), E = MRI->use_nodbg_end(); I != E;
         ++I) {
      if (!I->readsReg())
        continue;
      auto *UseMI = I->getParent();
      auto *UseMBB = UseMI->getParent();
      if (UseMBB == Flow || UseMBB == Endif) {
        if (!UseMI->isPHI())
          return true;

        auto *IncomingMBB = UseMI->getOperand(I.getOperandNo() + 1).getMBB();
        // The register is live through the path If->Flow or Flow->Endif.
        // we should not optimize for such cases.
        if ((UseMBB == Flow && IncomingMBB != If) ||
            (UseMBB == Endif && IncomingMBB == Flow))
          return true;
      }
    }
    return false;
  };

  for (auto Reg : KillsInElse) {
    if (!IsLiveThroughThen(Reg))
      CandidateRegs.push_back(Reg);
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 337-368: Implements SIOptimizeVGPRLiveRange::collectWaterfallCandidateRegisters
```cpp
/// Collect the registers used in the waterfall loop block that are defined
/// before.
void SIOptimizeVGPRLiveRange::collectWaterfallCandidateRegisters(
    MachineBasicBlock *LoopHeader, MachineBasicBlock *LoopEnd,
    SmallSetVector<Register, 16> &CandidateRegs,
    SmallSetVector<MachineBasicBlock *, 2> &Blocks,
    SmallVectorImpl<MachineInstr *> &Instructions) const {

  // Collect loop instructions, potentially spanning multiple blocks
  auto *MBB = LoopHeader;
  for (;;) {
    Blocks.insert(MBB);
    for (auto &MI : *MBB) {
      if (MI.isDebugInstr())
        continue;
      Instructions.push_back(&MI);
    }
    if (MBB == LoopEnd)
      break;

    if ((MBB != LoopHeader && MBB->pred_size() != 1) ||
        (MBB == LoopHeader && MBB->pred_size() != 2) || MBB->succ_size() != 1) {
      LLVM_DEBUG(dbgs() << "Unexpected edges in CFG, ignoring loop\n");
      return;
    }

    MBB = *MBB->succ_begin();
  }

  for (auto *I : Instructions) {
    auto &MI = *I;

```
**EN:** This section contains concrete logic for SIOptimizeVGPRLiveRange::collectWaterfallCandidateRegisters. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIOptimizeVGPRLiveRange::collectWaterfallCandidateRegisters`.
**CN:** 本节包含与 SIOptimizeVGPRLiveRange::collectWaterfallCandidateRegisters 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIOptimizeVGPRLiveRange::collectWaterfallCandidateRegisters`。

### Lines 369-402: Conditional logic and checks
```cpp
    for (auto &MO : MI.all_uses()) {
      if (!MO.getReg())
        continue;

      Register MOReg = MO.getReg();
      // We can only optimize AGPR/VGPR virtual register
      if (MOReg.isPhysical() || !TRI->isVectorRegister(*MRI, MOReg))
        continue;

      if (MO.readsReg()) {
        MachineBasicBlock *DefMBB = MRI->getVRegDef(MOReg)->getParent();
        // Make sure the value is defined before the LOOP block
        if (!Blocks.contains(DefMBB) && !CandidateRegs.contains(MOReg)) {
          // If the variable is used after the loop, the register coalescer will
          // merge the newly created register and remove the phi node again.
          // Just do nothing in that case.
          LiveVariables::VarInfo &OldVarInfo = LV->getVarInfo(MOReg);
          bool IsUsed = false;
          for (auto *Succ : LoopEnd->successors()) {
            if (!Blocks.contains(Succ) &&
                OldVarInfo.isLiveIn(*Succ, MOReg, *MRI)) {
              IsUsed = true;
              break;
            }
          }
          if (!IsUsed) {
            LLVM_DEBUG(dbgs() << "Found candidate reg: "
                              << printReg(MOReg, TRI, 0, MRI) << '\n');
            CandidateRegs.insert(MOReg);
          } else {
            LLVM_DEBUG(dbgs() << "Reg is used after loop, ignoring: "
                              << printReg(MOReg, TRI, 0, MRI) << '\n');
          }
        }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 403-433: Implements SIOptimizeVGPRLiveRange::updateLiveRangeInThenRegion
```cpp
      }
    }
  }
}

// Re-calculate the liveness of \p Reg in the THEN-region
void SIOptimizeVGPRLiveRange::updateLiveRangeInThenRegion(
    Register Reg, MachineBasicBlock *If, MachineBasicBlock *Flow) const {
  SetVector<MachineBasicBlock *> Blocks;
  SmallVector<MachineBasicBlock *> WorkList({If});

  // Collect all successors until we see the flow block, where we should
  // reconverge.
  while (!WorkList.empty()) {
    auto *MBB = WorkList.pop_back_val();
    for (auto *Succ : MBB->successors()) {
      if (Succ != Flow && Blocks.insert(Succ))
        WorkList.push_back(Succ);
    }
  }

  LiveVariables::VarInfo &OldVarInfo = LV->getVarInfo(Reg);
  for (MachineBasicBlock *MBB : Blocks) {
    // Clear Live bit, as we will recalculate afterwards
    LLVM_DEBUG(dbgs() << "Clear AliveBlock " << printMBBReference(*MBB)
                      << '\n');
    OldVarInfo.AliveBlocks.reset(MBB->getNumber());
  }

  SmallPtrSet<MachineBasicBlock *, 4> PHIIncoming;

```
**EN:** This section contains concrete logic for SIOptimizeVGPRLiveRange::updateLiveRangeInThenRegion. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIOptimizeVGPRLiveRange::updateLiveRangeInThenRegion`.
**CN:** 本节包含与 SIOptimizeVGPRLiveRange::updateLiveRangeInThenRegion 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIOptimizeVGPRLiveRange::updateLiveRangeInThenRegion`。

### Lines 434-462: Conditional logic and checks
```cpp
  // Get the blocks the Reg should be alive through
  for (auto I = MRI->use_nodbg_begin(Reg), E = MRI->use_nodbg_end(); I != E;
       ++I) {
    auto *UseMI = I->getParent();
    if (UseMI->isPHI() && I->readsReg()) {
      if (Blocks.contains(UseMI->getParent()))
        PHIIncoming.insert(UseMI->getOperand(I.getOperandNo() + 1).getMBB());
    }
  }

  for (MachineBasicBlock *MBB : Blocks) {
    SmallVector<MachineInstr *> Uses;
    // PHI instructions has been processed before.
    findNonPHIUsesInBlock(Reg, MBB, Uses);

    if (Uses.size() == 1) {
      LLVM_DEBUG(dbgs() << "Found one Non-PHI use in "
                        << printMBBReference(*MBB) << '\n');
      LV->HandleVirtRegUse(Reg, MBB, *(*Uses.begin()));
    } else if (Uses.size() > 1) {
      // Process the instructions in-order
      LLVM_DEBUG(dbgs() << "Found " << Uses.size() << " Non-PHI uses in "
                        << printMBBReference(*MBB) << '\n');
      for (MachineInstr &MI : *MBB) {
        if (llvm::is_contained(Uses, &MI))
          LV->HandleVirtRegUse(Reg, MBB, MI);
      }
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::is_contained`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::is_contained`。

### Lines 463-493: Conditional logic and checks
```cpp
    // Mark Reg alive through the block if this is a PHI incoming block
    if (PHIIncoming.contains(MBB))
      LV->MarkVirtRegAliveInBlock(OldVarInfo, MRI->getVRegDef(Reg)->getParent(),
                                  MBB);
  }

  // Set the isKilled flag if we get new Kills in the THEN region.
  for (auto *MI : OldVarInfo.Kills) {
    if (Blocks.contains(MI->getParent()))
      MI->addRegisterKilled(Reg, TRI);
  }
}

void SIOptimizeVGPRLiveRange::updateLiveRangeInElseRegion(
    Register Reg, Register NewReg, MachineBasicBlock *Flow,
    MachineBasicBlock *Endif,
    SmallSetVector<MachineBasicBlock *, 16> &ElseBlocks) const {
  LiveVariables::VarInfo &NewVarInfo = LV->getVarInfo(NewReg);
  LiveVariables::VarInfo &OldVarInfo = LV->getVarInfo(Reg);

  // Transfer aliveBlocks from Reg to NewReg
  for (auto *MBB : ElseBlocks) {
    unsigned BBNum = MBB->getNumber();
    if (OldVarInfo.AliveBlocks.test(BBNum)) {
      NewVarInfo.AliveBlocks.set(BBNum);
      LLVM_DEBUG(dbgs() << "Removing AliveBlock " << printMBBReference(*MBB)
                        << '\n');
      OldVarInfo.AliveBlocks.reset(BBNum);
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIOptimizeVGPRLiveRange::updateLiveRangeInElseRegion`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIOptimizeVGPRLiveRange::updateLiveRangeInElseRegion`。

### Lines 494-524: Declares begin
```cpp
  // Transfer the possible Kills in ElseBlocks from Reg to NewReg
  auto I = OldVarInfo.Kills.begin();
  while (I != OldVarInfo.Kills.end()) {
    if (ElseBlocks.contains((*I)->getParent())) {
      NewVarInfo.Kills.push_back(*I);
      I = OldVarInfo.Kills.erase(I);
    } else {
      ++I;
    }
  }
}

void SIOptimizeVGPRLiveRange::optimizeLiveRange(
    Register Reg, MachineBasicBlock *If, MachineBasicBlock *Flow,
    MachineBasicBlock *Endif,
    SmallSetVector<MachineBasicBlock *, 16> &ElseBlocks) const {
  // Insert a new PHI, marking the value from the THEN region being
  // undef.
  LLVM_DEBUG(dbgs() << "Optimizing " << printReg(Reg, TRI) << '\n');
  const auto *RC = MRI->getRegClass(Reg);
  Register NewReg = MRI->createVirtualRegister(RC);
  Register UndefReg = MRI->createVirtualRegister(RC);
  MachineInstrBuilder PHI = BuildMI(*Flow, Flow->getFirstNonPHI(), DebugLoc(),
                                    TII->get(TargetOpcode::PHI), NewReg);
  for (auto *Pred : Flow->predecessors()) {
    if (Pred == If)
      PHI.addReg(Reg).addMBB(Pred);
    else
      PHI.addReg(UndefReg, RegState::Undef).addMBB(Pred);
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SIOptimizeVGPRLiveRange::optimizeLiveRange`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SIOptimizeVGPRLiveRange::optimizeLiveRange`。

### Lines 525-558: Type declarations and aliases
```cpp
  // Replace all uses in the ELSE region or the PHIs in ENDIF block
  // Use early increment range because setReg() will update the linked list.
  for (auto &O : make_early_inc_range(MRI->use_operands(Reg))) {
    auto *UseMI = O.getParent();
    auto *UseBlock = UseMI->getParent();
    // Replace uses in Endif block
    if (UseBlock == Endif) {
      if (UseMI->isPHI())
        O.setReg(NewReg);
      else if (UseMI->isDebugInstr())
        continue;
      else {
        // DetectDeadLanes may mark register uses as undef without removing
        // them, in which case a non-phi instruction using the original register
        // may exist in the Endif block even though the register is not live
        // into it.
        assert(!O.readsReg());
      }
      continue;
    }

    // Replace uses in Else region
    if (ElseBlocks.contains(UseBlock))
      O.setReg(NewReg);
  }

  // The optimized Reg is not alive through Flow blocks anymore.
  LiveVariables::VarInfo &OldVarInfo = LV->getVarInfo(Reg);
  OldVarInfo.AliveBlocks.reset(Flow->getNumber());

  updateLiveRangeInElseRegion(Reg, NewReg, Flow, Endif, ElseBlocks);
  updateLiveRangeInThenRegion(Reg, If, Flow);
}

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。

### Lines 559-591: Implements SIOptimizeVGPRLiveRange::optimizeWaterfallLiveRange
```cpp
void SIOptimizeVGPRLiveRange::optimizeWaterfallLiveRange(
    Register Reg, MachineBasicBlock *LoopHeader,
    SmallSetVector<MachineBasicBlock *, 2> &Blocks,
    SmallVectorImpl<MachineInstr *> &Instructions) const {
  // Insert a new PHI, marking the value from the last loop iteration undef.
  LLVM_DEBUG(dbgs() << "Optimizing " << printReg(Reg, TRI) << '\n');
  const auto *RC = MRI->getRegClass(Reg);
  Register NewReg = MRI->createVirtualRegister(RC);
  Register UndefReg = MRI->createVirtualRegister(RC);

  // Replace all uses in the LOOP region
  // Use early increment range because setReg() will update the linked list.
  for (auto &O : make_early_inc_range(MRI->use_operands(Reg))) {
    auto *UseMI = O.getParent();
    auto *UseBlock = UseMI->getParent();
    // Replace uses in Loop blocks
    if (Blocks.contains(UseBlock))
      O.setReg(NewReg);
  }

  MachineInstrBuilder PHI =
      BuildMI(*LoopHeader, LoopHeader->getFirstNonPHI(), DebugLoc(),
              TII->get(TargetOpcode::PHI), NewReg);
  for (auto *Pred : LoopHeader->predecessors()) {
    if (Blocks.contains(Pred))
      PHI.addReg(UndefReg, RegState::Undef).addMBB(Pred);
    else
      PHI.addReg(Reg).addMBB(Pred);
  }

  LiveVariables::VarInfo &NewVarInfo = LV->getVarInfo(NewReg);
  LiveVariables::VarInfo &OldVarInfo = LV->getVarInfo(Reg);

```
**EN:** This section contains concrete logic for SIOptimizeVGPRLiveRange::optimizeWaterfallLiveRange. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIOptimizeVGPRLiveRange::optimizeWaterfallLiveRange`.
**CN:** 本节包含与 SIOptimizeVGPRLiveRange::optimizeWaterfallLiveRange 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIOptimizeVGPRLiveRange::optimizeWaterfallLiveRange`。

### Lines 592-625: Declares assert
```cpp
  // Find last use and mark as kill
  MachineInstr *Kill = nullptr;
  for (auto *MI : reverse(Instructions)) {
    if (MI->readsRegister(NewReg, TRI)) {
      MI->addRegisterKilled(NewReg, TRI);
      NewVarInfo.Kills.push_back(MI);
      Kill = MI;
      break;
    }
  }
  assert(Kill && "Failed to find last usage of register in loop");

  MachineBasicBlock *KillBlock = Kill->getParent();
  bool PostKillBlock = false;
  for (auto *Block : Blocks) {
    auto BBNum = Block->getNumber();

    // collectWaterfallCandidateRegisters only collects registers that are dead
    // after the loop. So we know that the old reg is no longer live throughout
    // the waterfall loop.
    OldVarInfo.AliveBlocks.reset(BBNum);

    // The new register is live up to (and including) the block that kills it.
    PostKillBlock |= (Block == KillBlock);
    if (PostKillBlock) {
      NewVarInfo.AliveBlocks.reset(BBNum);
    } else if (Block != LoopHeader) {
      NewVarInfo.AliveBlocks.set(BBNum);
    }
  }
}

char SIOptimizeVGPRLiveRangeLegacy::ID = 0;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 626-658: Registers LLVM passes
```cpp
INITIALIZE_PASS_BEGIN(SIOptimizeVGPRLiveRangeLegacy, DEBUG_TYPE,
                      "SI Optimize VGPR LiveRange", false, false)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LiveVariablesWrapperPass)
INITIALIZE_PASS_END(SIOptimizeVGPRLiveRangeLegacy, DEBUG_TYPE,
                    "SI Optimize VGPR LiveRange", false, false)

char &llvm::SIOptimizeVGPRLiveRangeLegacyID = SIOptimizeVGPRLiveRangeLegacy::ID;

FunctionPass *llvm::createSIOptimizeVGPRLiveRangeLegacyPass() {
  return new SIOptimizeVGPRLiveRangeLegacy();
}

bool SIOptimizeVGPRLiveRangeLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  LiveVariables *LV = &getAnalysis<LiveVariablesWrapperPass>().getLV();
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  MachineLoopInfo *Loops = &getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  return SIOptimizeVGPRLiveRange(LV, MDT, Loops).run(MF);
}

PreservedAnalyses
SIOptimizeVGPRLiveRangePass::run(MachineFunction &MF,
                                 MachineFunctionAnalysisManager &MFAM) {
  MFPropsModifier _(*this, MF);
  LiveVariables *LV = &MFAM.getResult<LiveVariablesAnalysis>(MF);
  MachineDominatorTree *MDT = &MFAM.getResult<MachineDominatorTreeAnalysis>(MF);
  MachineLoopInfo *Loops = &MFAM.getResult<MachineLoopAnalysis>(MF);

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createSIOptimizeVGPRLiveRangeLegacyPass`, `SIOptimizeVGPRLiveRangeLegacy::runOnMachineFunction`, `SIOptimizeVGPRLiveRangePass::run`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createSIOptimizeVGPRLiveRangeLegacyPass`, `SIOptimizeVGPRLiveRangeLegacy::runOnMachineFunction`, `SIOptimizeVGPRLiveRangePass::run`。

### Lines 659-689: Declares SIOptimizeVGPRLiveRange
```cpp
  bool Changed = SIOptimizeVGPRLiveRange(LV, MDT, Loops).run(MF);
  if (!Changed)
    return PreservedAnalyses::all();

  auto PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserve<LiveVariablesAnalysis>();
  PA.preserve<DominatorTreeAnalysis>();
  PA.preserve<MachineLoopAnalysis>();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

bool SIOptimizeVGPRLiveRange::run(MachineFunction &MF) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  TII = ST.getInstrInfo();
  TRI = &TII->getRegisterInfo();
  MRI = &MF.getRegInfo();

  bool MadeChange = false;

  // TODO: we need to think about the order of visiting the blocks to get
  // optimal result for nesting if-else cases.
  for (MachineBasicBlock &MBB : MF) {
    for (auto &MI : MBB.terminators()) {
      // Detect the if-else blocks
      if (MI.getOpcode() == AMDGPU::SI_IF) {
        MachineBasicBlock *IfTarget = MI.getOperand(2).getMBB();
        auto *Endif = getElseTarget(IfTarget);
        if (!Endif)
          continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `PreservedAnalyses::all`, `SIOptimizeVGPRLiveRange::run`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`PreservedAnalyses::all`, `SIOptimizeVGPRLiveRange::run`。

### Lines 690-722: Conditional logic and checks
```cpp
        // Skip unexpected control flow.
        if (!MDT->dominates(&MBB, IfTarget) || !MDT->dominates(IfTarget, Endif))
          continue;

        SmallSetVector<MachineBasicBlock *, 16> ElseBlocks;
        SmallVector<Register> CandidateRegs;

        LLVM_DEBUG(dbgs() << "Checking IF-ELSE-ENDIF: "
                          << printMBBReference(MBB) << ' '
                          << printMBBReference(*IfTarget) << ' '
                          << printMBBReference(*Endif) << '\n');

        // Collect all the blocks in the ELSE region
        collectElseRegionBlocks(IfTarget, Endif, ElseBlocks);

        // Collect the registers can be optimized
        collectCandidateRegisters(&MBB, IfTarget, Endif, ElseBlocks,
                                  CandidateRegs);
        MadeChange |= !CandidateRegs.empty();
        // Now we are safe to optimize.
        for (auto Reg : CandidateRegs)
          optimizeLiveRange(Reg, &MBB, IfTarget, Endif, ElseBlocks);
      } else if (MI.getOpcode() == AMDGPU::SI_WATERFALL_LOOP) {
        auto *LoopHeader = MI.getOperand(0).getMBB();
        auto *LoopEnd = &MBB;

        LLVM_DEBUG(dbgs() << "Checking Waterfall loop: "
                          << printMBBReference(*LoopHeader) << '\n');

        SmallSetVector<Register, 16> CandidateRegs;
        SmallVector<MachineInstr *, 16> Instructions;
        SmallSetVector<MachineBasicBlock *, 2> Blocks;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 723-734: Result computation and returns
```cpp
        collectWaterfallCandidateRegisters(LoopHeader, LoopEnd, CandidateRegs,
                                           Blocks, Instructions);
        MadeChange |= !CandidateRegs.empty();
        // Now we are safe to optimize.
        for (auto Reg : CandidateRegs)
          optimizeWaterfallLiveRange(Reg, LoopHeader, Blocks, Instructions);
      }
    }
  }

  return MadeChange;
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIOptimizeVGPRLiveRange`, `SIOptimizeVGPRLiveRangeLegacy`, `MachineFunctionPass::getAnalysisUsage`, `SIOptimizeVGPRLiveRange::getElseTarget`, `SIOptimizeVGPRLiveRange::collectElseRegionBlocks`, `SIOptimizeVGPRLiveRange::findNonPHIUsesInBlock`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; predicates and constraints / 谓词与约束
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIOptimizeVGPRLiveRange.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/LiveVariables.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/TargetRegisterInfo.h"`
- `"llvm/IR/Dominators.h"`
- `"llvm/InitializePasses.h"`
