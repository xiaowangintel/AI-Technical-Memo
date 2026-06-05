# AMDGPUGlobalISelDivergenceLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUGlobalISelDivergenceLowering.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUGlobalISelDivergenceLowering for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUGlobalISelDivergenceLowering 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===-- AMDGPUGlobalISelDivergenceLowering.cpp ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// GlobalISel pass that selects divergent i1 phis as lane mask phis.
/// Lane mask merging uses same algorithm as SDAG in SILowerI1Copies.
/// Handles all cases of temporal divergence.
/// For divergent non-phi i1 and uniform i1 uses outside of the cycle this pass
/// currently depends on LCSSA to insert phis with one incoming.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUGlobalISelUtils.h"
#include "SILowerI1Copies.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineUniformityAnalysis.h"
#include "llvm/InitializePasses.h"

#define DEBUG_TYPE "amdgpu-global-isel-divergence-lowering"

using namespace llvm;

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 32-64: Declares class AMDGPUGlobalISelDivergenceLowering
```cpp
class AMDGPUGlobalISelDivergenceLowering : public MachineFunctionPass {
public:
  static char ID;

public:
  AMDGPUGlobalISelDivergenceLowering() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "AMDGPU GlobalISel divergence lowering";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachinePostDominatorTreeWrapperPass>();
    AU.addRequired<MachineUniformityAnalysisPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

class DivergenceLoweringHelper : public AMDGPU::PhiLoweringHelper {
public:
  DivergenceLoweringHelper(MachineFunction *MF, MachineDominatorTree *DT,
                           MachinePostDominatorTree *PDT,
                           MachineUniformityInfo *MUI);

private:
  MachineUniformityInfo *MUI = nullptr;
  MachineIRBuilder B;
  Register buildRegCopyToLaneMask(Register Reg);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUGlobalISelDivergenceLowering`, `DivergenceLoweringHelper`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUGlobalISelDivergenceLowering`, `DivergenceLoweringHelper`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 65-98: Declares markAsLaneMask
```cpp
public:
  void markAsLaneMask(Register DstReg) const override;
  void getCandidatesForLowering(
      SmallVectorImpl<MachineInstr *> &Vreg1Phis) const override;
  void collectIncomingValuesFromPhi(
      const MachineInstr *MI,
      SmallVectorImpl<AMDGPU::Incoming> &Incomings) const override;
  void replaceDstReg(Register NewReg, Register OldReg,
                     MachineBasicBlock *MBB) override;
  void buildMergeLaneMasks(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator I, const DebugLoc &DL,
                           Register DstReg, Register PrevReg,
                           Register CurReg) override;
  void constrainAsLaneMask(AMDGPU::Incoming &In) override;

  bool lowerTemporalDivergence();
  bool lowerTemporalDivergenceI1();
};

DivergenceLoweringHelper::DivergenceLoweringHelper(
    MachineFunction *MF, MachineDominatorTree *DT,
    MachinePostDominatorTree *PDT, MachineUniformityInfo *MUI)
    : PhiLoweringHelper(MF, DT, PDT), MUI(MUI), B(*MF) {}

// _(s1) -> SReg_32/64(s1)
void DivergenceLoweringHelper::markAsLaneMask(Register DstReg) const {
  assert(MRI->getType(DstReg) == LLT::scalar(1));

  if (MRI->getRegClassOrNull(DstReg)) {
    if (MRI->constrainRegClass(DstReg, ST->getBoolRC()))
      return;
    llvm_unreachable("Failed to constrain register class");
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `DivergenceLoweringHelper::DivergenceLoweringHelper`, `DivergenceLoweringHelper::markAsLaneMask`, `LLT::scalar`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`DivergenceLoweringHelper::DivergenceLoweringHelper`, `DivergenceLoweringHelper::markAsLaneMask`, `LLT::scalar`。

### Lines 99-128: Declares setRegClass
```cpp
  MRI->setRegClass(DstReg, ST->getBoolRC());
}

void DivergenceLoweringHelper::getCandidatesForLowering(
    SmallVectorImpl<MachineInstr *> &Vreg1Phis) const {
  LLT S1 = LLT::scalar(1);

  // Add divergent i1 G_PHIs to the list. Only consider G_PHI instructions,
  // not PHI instructions that may have been created by earlier lowering stages
  // (e.g., lowerTemporalDivergenceI1).
  for (MachineBasicBlock &MBB : *MF) {
    for (MachineInstr &MI : MBB.phis()) {
      if (MI.getOpcode() != TargetOpcode::G_PHI)
        continue;
      Register Dst = MI.getOperand(0).getReg();
      if (MRI->getType(Dst) == S1 && MUI->isDivergentAtDef(Dst))
        Vreg1Phis.push_back(&MI);
    }
  }
}

void DivergenceLoweringHelper::collectIncomingValuesFromPhi(
    const MachineInstr *MI,
    SmallVectorImpl<AMDGPU::Incoming> &Incomings) const {
  for (unsigned i = 1; i < MI->getNumOperands(); i += 2) {
    Incomings.emplace_back(MI->getOperand(i).getReg(),
                           MI->getOperand(i + 1).getMBB(), Register());
  }
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `DivergenceLoweringHelper::getCandidatesForLowering`, `LLT::scalar`, `DivergenceLoweringHelper::collectIncomingValuesFromPhi`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`DivergenceLoweringHelper::getCandidatesForLowering`, `LLT::scalar`, `DivergenceLoweringHelper::collectIncomingValuesFromPhi`。

### Lines 129-162: Implements DivergenceLoweringHelper::replaceDstReg
```cpp
void DivergenceLoweringHelper::replaceDstReg(Register NewReg, Register OldReg,
                                             MachineBasicBlock *MBB) {
  BuildMI(*MBB, MBB->getFirstNonPHI(), {}, TII->get(AMDGPU::COPY), OldReg)
      .addReg(NewReg);
}

// Copy Reg to new lane mask register, insert a copy after instruction that
// defines Reg while skipping phis if needed.
Register DivergenceLoweringHelper::buildRegCopyToLaneMask(Register Reg) {
  Register LaneMask = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);
  MachineInstr *Instr = MRI->getVRegDef(Reg);
  MachineBasicBlock *MBB = Instr->getParent();
  B.setInsertPt(*MBB, MBB->SkipPHIsAndLabels(std::next(Instr->getIterator())));
  B.buildCopy(LaneMask, Reg);
  return LaneMask;
}

// bb.previous
//   %PrevReg = ...
//
// bb.current
//   %CurReg = ...
//
//   %DstReg - not defined
//
// -> (wave32 example, new registers have sreg_32 reg class and S1 LLT)
//
// bb.previous
//   %PrevReg = ...
//   %PrevRegCopy:sreg_32(s1) = COPY %PrevReg
//
// bb.current
//   %CurReg = ...
//   %CurRegCopy:sreg_32(s1) = COPY %CurReg
```
**EN:** This section contains concrete logic for DivergenceLoweringHelper::replaceDstReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DivergenceLoweringHelper::replaceDstReg`, `DivergenceLoweringHelper::buildRegCopyToLaneMask`, `AMDGPU::createLaneMaskReg`.
**CN:** 本节包含与 DivergenceLoweringHelper::replaceDstReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DivergenceLoweringHelper::replaceDstReg`, `DivergenceLoweringHelper::buildRegCopyToLaneMask`, `AMDGPU::createLaneMaskReg`。

### Lines 163-196: Implements DivergenceLoweringHelper::buildMergeLaneMasks
```cpp
//   ...
//   %PrevMaskedReg:sreg_32(s1) = ANDN2 %PrevRegCopy, ExecReg - active lanes 0
//   %CurMaskedReg:sreg_32(s1)  = AND %ExecReg, CurRegCopy - inactive lanes to 0
//   %DstReg:sreg_32(s1)        = OR %PrevMaskedReg, CurMaskedReg
//
// DstReg = for active lanes rewrite bit in PrevReg with bit from CurReg
void DivergenceLoweringHelper::buildMergeLaneMasks(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator I, const DebugLoc &DL,
    Register DstReg, Register PrevReg, Register CurReg) {
  // DstReg = (PrevReg & !EXEC) | (CurReg & EXEC)
  // TODO: check if inputs are constants or results of a compare.

  Register PrevRegCopy = buildRegCopyToLaneMask(PrevReg);
  Register CurRegCopy = buildRegCopyToLaneMask(CurReg);
  Register PrevMaskedReg = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);
  Register CurMaskedReg = AMDGPU::createLaneMaskReg(MRI, LaneMaskRegAttrs);

  B.setInsertPt(MBB, I);
  B.buildInstr(LMC->AndN2Opc, {PrevMaskedReg}, {PrevRegCopy, LMC->ExecReg});
  B.buildInstr(LMC->AndOpc, {CurMaskedReg}, {LMC->ExecReg, CurRegCopy});
  B.buildInstr(LMC->OrOpc, {DstReg}, {PrevMaskedReg, CurMaskedReg});
}

// GlobalISel has to constrain S1 incoming taken as-is with lane mask register
// class. Insert a copy of Incoming.Reg to new lane mask inside Incoming.Block,
// Incoming.Reg becomes that new lane mask.
void DivergenceLoweringHelper::constrainAsLaneMask(AMDGPU::Incoming &In) {
  B.setInsertPt(*In.Block, In.Block->getFirstTerminator());

  auto Copy = B.buildCopy(LLT::scalar(1), In.Reg);
  MRI->setRegClass(Copy.getReg(0), ST->getBoolRC());
  In.Reg = Copy.getReg(0);
}

```
**EN:** This section contains concrete logic for DivergenceLoweringHelper::buildMergeLaneMasks. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DivergenceLoweringHelper::buildMergeLaneMasks`, `AMDGPU::createLaneMaskReg`, `DivergenceLoweringHelper::constrainAsLaneMask`.
**CN:** 本节包含与 DivergenceLoweringHelper::buildMergeLaneMasks 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DivergenceLoweringHelper::buildMergeLaneMasks`, `AMDGPU::createLaneMaskReg`, `DivergenceLoweringHelper::constrainAsLaneMask`。

### Lines 197-227: Defines replaceUsesOfRegInInstWith
```cpp
void replaceUsesOfRegInInstWith(Register Reg, MachineInstr *Inst,
                                Register NewReg) {
  for (MachineOperand &Op : Inst->operands()) {
    if (Op.isReg() && Op.getReg() == Reg)
      Op.setReg(NewReg);
  }
}

bool DivergenceLoweringHelper::lowerTemporalDivergence() {
  AMDGPU::IntrinsicLaneMaskAnalyzer ILMA(*MF);
  DenseMap<Register, Register> TDCache;

  for (auto [Reg, UseInst, _] : MUI->getTemporalDivergenceList()) {
    if (MRI->getType(Reg) == LLT::scalar(1) || MUI->isDivergentAtDef(Reg) ||
        ILMA.isS32S64LaneMask(Reg))
      continue;

    Register CachedTDCopy = TDCache.lookup(Reg);
    if (CachedTDCopy) {
      replaceUsesOfRegInInstWith(Reg, UseInst, CachedTDCopy);
      continue;
    }

    MachineInstr *Inst = MRI->getVRegDef(Reg);
    MachineBasicBlock *MBB = Inst->getParent();
    B.setInsertPt(*MBB, MBB->SkipPHIsAndLabels(std::next(Inst->getIterator())));

    Register VgprReg = MRI->createGenericVirtualRegister(MRI->getType(Reg));
    B.buildInstr(AMDGPU::COPY, {VgprReg}, {Reg})
        .addUse(LMC->ExecReg, RegState::Implicit);

```
**EN:** This section contains concrete logic for replaceUsesOfRegInInstWith. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DivergenceLoweringHelper::lowerTemporalDivergence`, `LLT::scalar`, `std::next`.
**CN:** 本节包含与 replaceUsesOfRegInInstWith 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DivergenceLoweringHelper::lowerTemporalDivergence`, `LLT::scalar`, `std::next`。

### Lines 228-261: Implements DivergenceLoweringHelper::lowerTemporalDivergenceI1
```cpp
    replaceUsesOfRegInInstWith(Reg, UseInst, VgprReg);
    TDCache[Reg] = VgprReg;
  }
  return false;
}

bool DivergenceLoweringHelper::lowerTemporalDivergenceI1() {
  MachineRegisterInfo::VRegAttrs BoolS1 = {ST->getBoolRC(), LLT::scalar(1)};
  initializeLaneMaskRegisterAttributes(BoolS1);
  MachineSSAUpdater SSAUpdater(*MF);

  // In case of use outside muliple nested cycles or muliple uses we only need
  // to merge lane mask across largest relevant cycle.
  SmallDenseMap<Register, std::pair<const MachineCycle *, Register>> LRCCache;
  for (auto [Reg, UseInst, LRC] : MUI->getTemporalDivergenceList()) {
    if (MRI->getType(Reg) != LLT::scalar(1))
      continue;

    auto [LRCCacheIter, RegNotCached] = LRCCache.try_emplace(Reg);
    auto &CycleMergedMask = LRCCacheIter->getSecond();
    const MachineCycle *&CachedLRC = CycleMergedMask.first;
    if (RegNotCached || LRC->contains(CachedLRC)) {
      CachedLRC = LRC;
    }
  }

  for (auto &LRCCacheEntry : LRCCache) {
    Register Reg = LRCCacheEntry.first;
    auto &CycleMergedMask = LRCCacheEntry.getSecond();
    const MachineCycle *Cycle = CycleMergedMask.first;

    Register MergedMask = MRI->createVirtualRegister(BoolS1);
    SSAUpdater.Initialize(MergedMask);

```
**EN:** This section contains concrete logic for DivergenceLoweringHelper::lowerTemporalDivergenceI1. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DivergenceLoweringHelper::lowerTemporalDivergenceI1`, `LLT::scalar`.
**CN:** 本节包含与 DivergenceLoweringHelper::lowerTemporalDivergenceI1 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DivergenceLoweringHelper::lowerTemporalDivergenceI1`, `LLT::scalar`。

### Lines 262-292: Conditional logic and checks
```cpp
    MachineBasicBlock *MBB = MRI->getVRegDef(Reg)->getParent();
    SSAUpdater.AddAvailableValue(MBB, MergedMask);

    for (auto Entry : Cycle->getEntries()) {
      for (MachineBasicBlock *Pred : Entry->predecessors()) {
        if (!Cycle->contains(Pred)) {
          B.setInsertPt(*Pred, Pred->getFirstTerminator());
          auto ImplDef = B.buildInstr(AMDGPU::IMPLICIT_DEF, {BoolS1}, {});
          SSAUpdater.AddAvailableValue(Pred, ImplDef.getReg(0));
        }
      }
    }

    buildMergeLaneMasks(*MBB, MBB->getFirstTerminator(), {}, MergedMask,
                        SSAUpdater.GetValueInMiddleOfBlock(MBB), Reg);

    CycleMergedMask.second = MergedMask;
  }

  for (auto [Reg, UseInst, Cycle] : MUI->getTemporalDivergenceList()) {
    if (MRI->getType(Reg) != LLT::scalar(1))
      continue;

    replaceUsesOfRegInInstWith(Reg, UseInst, LRCCache.lookup(Reg).second);
  }

  return false;
}

} // End anonymous namespace.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `LLT::scalar`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`LLT::scalar`。

### Lines 293-320: Registers LLVM passes
```cpp
INITIALIZE_PASS_BEGIN(AMDGPUGlobalISelDivergenceLowering, DEBUG_TYPE,
                      "AMDGPU GlobalISel divergence lowering", false, false)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachinePostDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineUniformityAnalysisPass)
INITIALIZE_PASS_END(AMDGPUGlobalISelDivergenceLowering, DEBUG_TYPE,
                    "AMDGPU GlobalISel divergence lowering", false, false)

char AMDGPUGlobalISelDivergenceLowering::ID = 0;

char &llvm::AMDGPUGlobalISelDivergenceLoweringID =
    AMDGPUGlobalISelDivergenceLowering::ID;

FunctionPass *llvm::createAMDGPUGlobalISelDivergenceLoweringPass() {
  return new AMDGPUGlobalISelDivergenceLowering();
}

bool AMDGPUGlobalISelDivergenceLowering::runOnMachineFunction(
    MachineFunction &MF) {
  MachineDominatorTree &DT =
      getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  MachinePostDominatorTree &PDT =
      getAnalysis<MachinePostDominatorTreeWrapperPass>().getPostDomTree();
  MachineUniformityInfo &MUI =
      getAnalysis<MachineUniformityAnalysisPass>().getUniformityInfo();

  DivergenceLoweringHelper Helper(&MF, &DT, &PDT, &MUI);

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUGlobalISelDivergenceLoweringPass`, `AMDGPUGlobalISelDivergenceLowering::runOnMachineFunction`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUGlobalISelDivergenceLoweringPass`, `AMDGPUGlobalISelDivergenceLowering::runOnMachineFunction`。

### Lines 321-338: Declares lowerTemporalDivergence
```cpp
  bool Changed = false;
  // Temporal divergence lowering needs to inspect list of instructions used
  // outside cycle with divergent exit provided by uniformity analysis. Uniform
  // instructions from the list require lowering, no instruction is deleted.
  // Thus it needs to be run before lowerPhis that deletes phis that require
  // lowering and replaces them with new instructions.

  // Non-i1 temporal divergence lowering.
  Changed |= Helper.lowerTemporalDivergence();
  // This covers both uniform and divergent i1s. Lane masks are in sgpr and need
  // to be updated in each iteration.
  Changed |= Helper.lowerTemporalDivergenceI1();
  // Temporal divergence lowering of divergent i1 phi used outside of the cycle
  // could also be handled by lowerPhis but we do it in lowerTempDivergenceI1
  // since in some case lowerPhis does unnecessary lane mask merging.
  Changed |= Helper.lowerPhis();
  return Changed;
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUGlobalISelDivergenceLowering`, `DivergenceLoweringHelper`, `MachineFunctionPass::getAnalysisUsage`, `DivergenceLoweringHelper::DivergenceLoweringHelper`, `DivergenceLoweringHelper::markAsLaneMask`, `LLT::scalar`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUGlobalISelUtils.h"`
- `"SILowerI1Copies.h"`
- `"llvm/CodeGen/GlobalISel/MachineIRBuilder.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachineUniformityAnalysis.h"`
- `"llvm/InitializePasses.h"`
