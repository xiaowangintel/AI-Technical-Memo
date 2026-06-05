# SIOptimizeExecMaskingPreRA.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIOptimizeExecMaskingPreRA.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIOptimizeExecMaskingPreRA for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIOptimizeExecMaskingPreRA 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: File banner, includes, and setup
```cpp
//===-- SIOptimizeExecMaskingPreRA.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass performs exec mask handling peephole optimizations which needs
/// to be done before register allocation to reduce register pressure.
///
//===----------------------------------------------------------------------===//

#include "SIOptimizeExecMaskingPreRA.h"
#include "AMDGPU.h"
#include "AMDGPULaneMaskUtils.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "si-optimize-exec-masking-pre-ra"

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 30-60: Declares class SIOptimizeExecMaskingPreRA
```cpp
class SIOptimizeExecMaskingPreRA {
private:
  const GCNSubtarget &ST;
  const SIRegisterInfo *TRI;
  const SIInstrInfo *TII;
  MachineRegisterInfo *MRI;
  LiveIntervals *LIS;
  const AMDGPU::LaneMaskConstants &LMC;

  MCRegister CondReg;
  MCRegister ExecReg;

  bool optimizeVcndVcmpPair(MachineBasicBlock &MBB);
  bool optimizeElseBranch(MachineBasicBlock &MBB);

public:
  SIOptimizeExecMaskingPreRA(MachineFunction &MF, LiveIntervals *LIS)
      : ST(MF.getSubtarget<GCNSubtarget>()), TRI(ST.getRegisterInfo()),
        TII(ST.getInstrInfo()), MRI(&MF.getRegInfo()), LIS(LIS),
        LMC(AMDGPU::LaneMaskConstants::get(ST)) {}
  bool run(MachineFunction &MF);
};

class SIOptimizeExecMaskingPreRALegacy : public MachineFunctionPass {
public:
  static char ID;

  SIOptimizeExecMaskingPreRALegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIOptimizeExecMaskingPreRA`, `SIOptimizeExecMaskingPreRALegacy`, `LaneMaskConstants::get`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIOptimizeExecMaskingPreRA`, `SIOptimizeExecMaskingPreRALegacy`, `LaneMaskConstants::get`。

### Lines 61-87: Registers LLVM passes
```cpp
  StringRef getPassName() const override {
    return "SI optimize exec mask operations pre-RA";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(SIOptimizeExecMaskingPreRALegacy, DEBUG_TYPE,
                      "SI optimize exec mask operations pre-RA", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_END(SIOptimizeExecMaskingPreRALegacy, DEBUG_TYPE,
                    "SI optimize exec mask operations pre-RA", false, false)

char SIOptimizeExecMaskingPreRALegacy::ID = 0;

char &llvm::SIOptimizeExecMaskingPreRAID = SIOptimizeExecMaskingPreRALegacy::ID;

FunctionPass *llvm::createSIOptimizeExecMaskingPreRAPass() {
  return new SIOptimizeExecMaskingPreRALegacy();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIOptimizeExecMaskingPreRAPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `llvm::createSIOptimizeExecMaskingPreRAPass`。

### Lines 88-113: Defines isDefBetween
```cpp
// See if there is a def between \p AndIdx and \p SelIdx that needs to live
// beyond \p AndIdx.
static bool isDefBetween(const LiveRange &LR, SlotIndex AndIdx,
                         SlotIndex SelIdx) {
  LiveQueryResult AndLRQ = LR.Query(AndIdx);
  return (!AndLRQ.isKill() && AndLRQ.valueIn() != LR.Query(SelIdx).valueOut());
}

// FIXME: Why do we bother trying to handle physical registers here?
static bool isDefBetween(const SIRegisterInfo &TRI,
                         LiveIntervals *LIS, Register Reg,
                         const MachineInstr &Sel, const MachineInstr &And) {
  SlotIndex AndIdx = LIS->getInstructionIndex(And).getRegSlot();
  SlotIndex SelIdx = LIS->getInstructionIndex(Sel).getRegSlot();

  if (Reg.isVirtual())
    return isDefBetween(LIS->getInterval(Reg), AndIdx, SelIdx);

  for (MCRegUnit Unit : TRI.regunits(Reg.asMCReg())) {
    if (isDefBetween(LIS->getRegUnit(Unit), AndIdx, SelIdx))
      return true;
  }

  return false;
}

```
**EN:** This section contains concrete logic for isDefBetween. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isDefBetween 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 114-143: Conditional logic and checks
```cpp
// Optimize sequence
//    %sel = V_CNDMASK_B32_e64 0, 1, %cc
//    %cmp = V_CMP_NE_U32 1, %sel
//    $vcc = S_AND_B64 $exec, %cmp
//    S_CBRANCH_VCC[N]Z
// =>
//    $vcc = S_ANDN2_B64 $exec, %cc
//    S_CBRANCH_VCC[N]Z
//
// It is the negation pattern inserted by DAGCombiner::visitBRCOND() in the
// rebuildSetCC(). We start with S_CBRANCH to avoid exhaustive search, but
// only 3 first instructions are really needed. S_AND_B64 with exec is a
// required part of the pattern since V_CNDMASK_B32 writes zeroes for inactive
// lanes.
//
// Returns true on success.
bool SIOptimizeExecMaskingPreRA::optimizeVcndVcmpPair(MachineBasicBlock &MBB) {
  auto I = llvm::find_if(MBB.terminators(), [](const MachineInstr &MI) {
                           unsigned Opc = MI.getOpcode();
                           return Opc == AMDGPU::S_CBRANCH_VCCZ ||
                                  Opc == AMDGPU::S_CBRANCH_VCCNZ; });
  if (I == MBB.terminators().end())
    return false;

  auto *And =
      TRI->findReachingDef(CondReg, AMDGPU::NoSubRegister, *I, *MRI, LIS);
  if (!And || And->getOpcode() != LMC.AndOpc || !And->getOperand(1).isReg() ||
      !And->getOperand(2).isReg())
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `DAGCombiner::visitBRCOND`, `SIOptimizeExecMaskingPreRA::optimizeVcndVcmpPair`, `llvm::find_if`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`DAGCombiner::visitBRCOND`, `SIOptimizeExecMaskingPreRA::optimizeVcndVcmpPair`, `llvm::find_if`。

### Lines 144-175: Declares getOperand
```cpp
  MachineOperand *AndCC = &And->getOperand(1);
  Register CmpReg = AndCC->getReg();
  unsigned CmpSubReg = AndCC->getSubReg();
  if (CmpReg == Register(ExecReg)) {
    AndCC = &And->getOperand(2);
    CmpReg = AndCC->getReg();
    CmpSubReg = AndCC->getSubReg();
  } else if (And->getOperand(2).getReg() != Register(ExecReg)) {
    return false;
  }

  auto *Cmp = TRI->findReachingDef(CmpReg, CmpSubReg, *And, *MRI, LIS);
  if (!Cmp || !(Cmp->getOpcode() == AMDGPU::V_CMP_NE_U32_e32 ||
                Cmp->getOpcode() == AMDGPU::V_CMP_NE_U32_e64) ||
      Cmp->getParent() != And->getParent())
    return false;

  MachineOperand *Op1 = TII->getNamedOperand(*Cmp, AMDGPU::OpName::src0);
  MachineOperand *Op2 = TII->getNamedOperand(*Cmp, AMDGPU::OpName::src1);
  if (Op1->isImm() && Op2->isReg())
    std::swap(Op1, Op2);
  if (!Op1->isReg() || !Op2->isImm() || Op2->getImm() != 1)
    return false;

  Register SelReg = Op1->getReg();
  if (SelReg.isPhysical())
    return false;

  auto *Sel = TRI->findReachingDef(SelReg, Op1->getSubReg(), *Cmp, *MRI, LIS);
  if (!Sel || Sel->getOpcode() != AMDGPU::V_CNDMASK_B32_e64)
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::swap`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::swap`。

### Lines 176-207: Declares getNamedOperand
```cpp
  if (TII->hasModifiersSet(*Sel, AMDGPU::OpName::src0_modifiers) ||
      TII->hasModifiersSet(*Sel, AMDGPU::OpName::src1_modifiers))
    return false;

  Op1 = TII->getNamedOperand(*Sel, AMDGPU::OpName::src0);
  Op2 = TII->getNamedOperand(*Sel, AMDGPU::OpName::src1);
  MachineOperand *CC = TII->getNamedOperand(*Sel, AMDGPU::OpName::src2);
  if (!Op1->isImm() || !Op2->isImm() || !CC->isReg() ||
      Op1->getImm() != 0 || Op2->getImm() != 1)
    return false;

  Register CCReg = CC->getReg();

  // If there was a def between the select and the and, we would need to move it
  // to fold this.
  if (isDefBetween(*TRI, LIS, CCReg, *Sel, *And))
    return false;

  // Cannot safely mirror live intervals with PHI nodes, so check for these
  // before optimization.
  SlotIndex SelIdx = LIS->getInstructionIndex(*Sel);
  LiveInterval *SelLI = &LIS->getInterval(SelReg);
  if (llvm::any_of(SelLI->vnis(),
                    [](const VNInfo *VNI) {
                      return VNI->isPHIDef();
                    }))
    return false;

  // TODO: Guard against implicit def operands?
  LLVM_DEBUG(dbgs() << "Folding sequence:\n\t" << *Sel << '\t' << *Cmp << '\t'
                    << *And);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `llvm::any_of`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`llvm::any_of`。

### Lines 208-236: Declares getOperand
```cpp
  MachineInstr *Andn2 =
      BuildMI(MBB, *And, And->getDebugLoc(), TII->get(LMC.AndN2Opc),
              And->getOperand(0).getReg())
          .addReg(ExecReg)
          .addReg(CCReg, getUndefRegState(CC->isUndef()), CC->getSubReg());
  MachineOperand &AndSCC = And->getOperand(3);
  assert(AndSCC.getReg() == AMDGPU::SCC);
  MachineOperand &Andn2SCC = Andn2->getOperand(3);
  assert(Andn2SCC.getReg() == AMDGPU::SCC);
  Andn2SCC.setIsDead(AndSCC.isDead());

  SlotIndex AndIdx = LIS->ReplaceMachineInstrInMaps(*And, *Andn2);
  And->eraseFromParent();

  LLVM_DEBUG(dbgs() << "=>\n\t" << *Andn2 << '\n');

  // Update live intervals for CCReg before potentially removing CmpReg/SelReg,
  // and their associated liveness information.
  SlotIndex CmpIdx = LIS->getInstructionIndex(*Cmp);
  if (CCReg.isVirtual()) {
    LiveInterval &CCLI = LIS->getInterval(CCReg);
    auto CCQ = CCLI.Query(SelIdx.getRegSlot());
    if (CCQ.valueIn()) {
      LIS->removeInterval(CCReg);
      LIS->createAndComputeVirtRegInterval(CCReg);
    }
  } else
    LIS->removeAllRegUnitsForPhysReg(CCReg);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 237-259: Declares isVirtual
```cpp
  // Try to remove compare. Cmp value should not used in between of cmp
  // and s_and_b64 if VCC or just unused if any other register.
  LiveInterval *CmpLI = CmpReg.isVirtual() ? &LIS->getInterval(CmpReg) : nullptr;
  if ((CmpLI && CmpLI->Query(AndIdx.getRegSlot()).isKill()) ||
      (CmpReg == Register(CondReg) &&
       std::none_of(std::next(Cmp->getIterator()), Andn2->getIterator(),
                    [&](const MachineInstr &MI) {
                      return MI.readsRegister(CondReg, TRI);
                    }))) {
    LLVM_DEBUG(dbgs() << "Erasing: " << *Cmp << '\n');
    if (CmpLI)
      LIS->removeVRegDefAt(*CmpLI, CmpIdx.getRegSlot());
    LIS->RemoveMachineInstrFromMaps(*Cmp);
    Cmp->eraseFromParent();

    // Try to remove v_cndmask_b32.
    // Kill status must be checked before shrinking the live range.
    bool IsKill = SelLI->Query(CmpIdx.getRegSlot()).isKill();
    LIS->shrinkToUses(SelLI);
    bool IsDead = SelLI->Query(SelIdx.getRegSlot()).isDeadDef();
    if (MRI->use_nodbg_empty(SelReg) && (IsKill || IsDead)) {
      LLVM_DEBUG(dbgs() << "Erasing: " << *Sel << '\n');

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::none_of`, `std::next`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::none_of`, `std::next`。

### Lines 260-292: Conditional logic and checks
```cpp
      LIS->removeVRegDefAt(*SelLI, SelIdx.getRegSlot());
      LIS->RemoveMachineInstrFromMaps(*Sel);
      bool ShrinkSel = Sel->getOperand(0).readsReg();
      Sel->eraseFromParent();
      if (ShrinkSel) {
        // The result of the V_CNDMASK was a subreg def which counted as a read
        // from the other parts of the reg. Shrink their live ranges.
        LIS->shrinkToUses(SelLI);
      }
    }
  }

  return true;
}

// Optimize sequence
//    %dst = S_OR_SAVEEXEC %src
//    ... instructions not modifying exec ...
//    %tmp = S_AND $exec, %dst
//    $exec = S_XOR_term $exec, %tmp
// =>
//    %dst = S_OR_SAVEEXEC %src
//    ... instructions not modifying exec ...
//    $exec = S_XOR_term $exec, %dst
//
// Clean up potentially unnecessary code added for safety during
// control flow lowering.
//
// Return whether any changes were made to MBB.
bool SIOptimizeExecMaskingPreRA::optimizeElseBranch(MachineBasicBlock &MBB) {
  if (MBB.empty())
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIOptimizeExecMaskingPreRA::optimizeElseBranch`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIOptimizeExecMaskingPreRA::optimizeElseBranch`。

### Lines 293-323: Declares begin
```cpp
  // Check this is an else block.
  auto First = MBB.begin();
  MachineInstr &SaveExecMI = *First;
  if (SaveExecMI.getOpcode() != LMC.OrSaveExecOpc)
    return false;

  auto I = llvm::find_if(MBB.terminators(), [this](const MachineInstr &MI) {
    return MI.getOpcode() == LMC.XorTermOpc;
  });
  if (I == MBB.terminators().end())
    return false;

  MachineInstr &XorTermMI = *I;
  if (XorTermMI.getOperand(1).getReg() != Register(ExecReg))
    return false;

  Register SavedExecReg = SaveExecMI.getOperand(0).getReg();
  Register DstReg = XorTermMI.getOperand(2).getReg();

  // Find potentially unnecessary S_AND
  MachineInstr *AndExecMI = nullptr;
  I--;
  while (I != First && !AndExecMI) {
    if (I->getOpcode() == LMC.AndOpc && I->getOperand(0).getReg() == DstReg &&
        I->getOperand(1).getReg() == Register(ExecReg))
      AndExecMI = &*I;
    I--;
  }
  if (!AndExecMI)
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `llvm::find_if`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`llvm::find_if`。

### Lines 324-357: Declares getInstructionIndex
```cpp
  // Check for exec modifying instructions.
  // Note: exec defs do not create live ranges beyond the
  // instruction so isDefBetween cannot be used.
  // Instead just check that the def segments are adjacent.
  SlotIndex StartIdx = LIS->getInstructionIndex(SaveExecMI);
  SlotIndex EndIdx = LIS->getInstructionIndex(*AndExecMI);
  for (MCRegUnit Unit : TRI->regunits(ExecReg)) {
    LiveRange &RegUnit = LIS->getRegUnit(Unit);
    if (RegUnit.find(StartIdx) != std::prev(RegUnit.find(EndIdx)))
      return false;
  }

  // Remove unnecessary S_AND
  LIS->removeInterval(SavedExecReg);
  LIS->removeInterval(DstReg);

  SaveExecMI.getOperand(0).setReg(DstReg);

  LIS->RemoveMachineInstrFromMaps(*AndExecMI);
  AndExecMI->eraseFromParent();

  LIS->createAndComputeVirtRegInterval(DstReg);

  return true;
}

PreservedAnalyses
SIOptimizeExecMaskingPreRAPass::run(MachineFunction &MF,
                                    MachineFunctionAnalysisManager &MFAM) {
  auto &LIS = MFAM.getResult<LiveIntervalsAnalysis>(MF);
  SIOptimizeExecMaskingPreRA(MF, &LIS).run(MF);
  return PreservedAnalyses::all();
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::prev`, `SIOptimizeExecMaskingPreRAPass::run`, `PreservedAnalyses::all`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::prev`, `SIOptimizeExecMaskingPreRAPass::run`, `PreservedAnalyses::all`。

### Lines 358-387: Implements SIOptimizeExecMaskingPreRALegacy::runOnMachineFunction
```cpp
bool SIOptimizeExecMaskingPreRALegacy::runOnMachineFunction(
    MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  auto *LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  return SIOptimizeExecMaskingPreRA(MF, LIS).run(MF);
}

bool SIOptimizeExecMaskingPreRA::run(MachineFunction &MF) {
  CondReg = MCRegister::from(LMC.VccReg);
  ExecReg = MCRegister::from(LMC.ExecReg);

  DenseSet<Register> RecalcRegs({AMDGPU::EXEC_LO, AMDGPU::EXEC_HI});
  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {

    if (optimizeElseBranch(MBB)) {
      RecalcRegs.insert(AMDGPU::SCC);
      Changed = true;
    }

    if (optimizeVcndVcmpPair(MBB)) {
      RecalcRegs.insert(AMDGPU::VCC_LO);
      RecalcRegs.insert(AMDGPU::VCC_HI);
      RecalcRegs.insert(AMDGPU::SCC);
      Changed = true;
    }

```
**EN:** This section contains concrete logic for SIOptimizeExecMaskingPreRALegacy::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIOptimizeExecMaskingPreRALegacy::runOnMachineFunction`, `SIOptimizeExecMaskingPreRA::run`, `MCRegister::from`.
**CN:** 本节包含与 SIOptimizeExecMaskingPreRALegacy::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIOptimizeExecMaskingPreRALegacy::runOnMachineFunction`, `SIOptimizeExecMaskingPreRA::run`, `MCRegister::from`。

### Lines 388-418: Conditional logic and checks
```cpp
    // Try to remove unneeded instructions before s_endpgm.
    if (MBB.succ_empty()) {
      if (MBB.empty())
        continue;

      // Skip this if the endpgm has any implicit uses, otherwise we would need
      // to be careful to update / remove them.
      // S_ENDPGM always has a single imm operand that is not used other than to
      // end up in the encoding
      MachineInstr &Term = MBB.back();
      if (Term.getOpcode() != AMDGPU::S_ENDPGM || Term.getNumOperands() != 1)
        continue;

      SmallVector<MachineBasicBlock*, 4> Blocks({&MBB});

      while (!Blocks.empty()) {
        auto *CurBB = Blocks.pop_back_val();
        auto I = CurBB->rbegin(), E = CurBB->rend();
        if (I != E) {
          if (I->isUnconditionalBranch() || I->getOpcode() == AMDGPU::S_ENDPGM)
            ++I;
          else if (I->isBranch())
            continue;
        }

        while (I != E) {
          if (I->isDebugInstr()) {
            I = std::next(I);
            continue;
          }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 419-450: Conditional logic and checks
```cpp
          if (I->mayStore() || I->isBarrier() || I->isCall() ||
              I->hasUnmodeledSideEffects() || I->hasOrderedMemoryRef())
            break;

          LLVM_DEBUG(dbgs()
                     << "Removing no effect instruction: " << *I << '\n');

          for (auto &Op : I->operands()) {
            if (Op.isReg())
              RecalcRegs.insert(Op.getReg());
          }

          auto Next = std::next(I);
          LIS->RemoveMachineInstrFromMaps(*I);
          I->eraseFromParent();
          I = Next;

          Changed = true;
        }

        if (I != E)
          continue;

        // Try to ascend predecessors.
        for (auto *Pred : CurBB->predecessors()) {
          if (Pred->succ_size() == 1)
            Blocks.push_back(Pred);
        }
      }
      continue;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 451-484: Conditional logic and checks
```cpp
    // If the only user of a logical operation is move to exec, fold it now
    // to prevent forming of saveexec. I.e.:
    //
    //    %0:sreg_64 = COPY $exec
    //    %1:sreg_64 = S_AND_B64 %0:sreg_64, %2:sreg_64
    // =>
    //    %1 = S_AND_B64 $exec, %2:sreg_64
    unsigned ScanThreshold = 10;
    for (auto I = MBB.rbegin(), E = MBB.rend(); I != E
         && ScanThreshold--; ++I) {
      // Continue scanning if this is not a full exec copy
      if (!(I->isFullCopy() && I->getOperand(1).getReg() == Register(ExecReg)))
        continue;

      Register SavedExec = I->getOperand(0).getReg();
      if (SavedExec.isVirtual() && MRI->hasOneNonDBGUse(SavedExec)) {
        MachineInstr *SingleExecUser = &*MRI->use_instr_nodbg_begin(SavedExec);
        int Idx = SingleExecUser->findRegisterUseOperandIdx(SavedExec,
                                                            /*TRI=*/nullptr);
        assert(Idx != -1);
        if (SingleExecUser->getParent() == I->getParent() &&
            !SingleExecUser->getOperand(Idx).isImplicit() &&
            static_cast<unsigned>(Idx) <
                SingleExecUser->getDesc().getNumOperands() &&
            TII->isOperandLegal(*SingleExecUser, Idx, &I->getOperand(1))) {
          LLVM_DEBUG(dbgs() << "Redundant EXEC COPY: " << *I << '\n');
          LIS->RemoveMachineInstrFromMaps(*I);
          I->eraseFromParent();
          MRI->replaceRegWith(SavedExec, ExecReg);
          LIS->removeInterval(SavedExec);
          Changed = true;
        }
      }
      break;
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 485-501: Conditional logic and checks
```cpp
    }
  }

  if (Changed) {
    for (auto Reg : RecalcRegs) {
      if (Reg.isVirtual()) {
        LIS->removeInterval(Reg);
        if (!MRI->reg_empty(Reg))
          LIS->createAndComputeVirtRegInterval(Reg);
      } else {
        LIS->removeAllRegUnitsForPhysReg(Reg);
      }
    }
  }

  return Changed;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIOptimizeExecMaskingPreRA`, `SIOptimizeExecMaskingPreRALegacy`, `LaneMaskConstants::get`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIOptimizeExecMaskingPreRAPass`, `DAGCombiner::visitBRCOND`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIOptimizeExecMaskingPreRA.h"`
- `"AMDGPU.h"`
- `"AMDGPULaneMaskUtils.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/InitializePasses.h"`
