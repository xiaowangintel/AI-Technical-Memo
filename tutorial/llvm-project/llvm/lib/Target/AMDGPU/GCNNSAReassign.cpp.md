# GCNNSAReassign.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNNSAReassign.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNNSAReassign for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNNSAReassign 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===-- GCNNSAReassign.cpp - Reassign registers in NSA instructions -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// \brief Try to reassign registers on GFX10+ from non-sequential to sequential
/// in NSA image instructions. Later SIShrinkInstructions pass will replace NSA
/// with sequential versions where possible.
///
//===----------------------------------------------------------------------===//

#include "GCNNSAReassign.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "SIMachineFunctionInfo.h"
#include "SIRegisterInfo.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/LiveRegMatrix.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/VirtRegMap.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-nsa-reassign"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 32-65: Declares class GCNNSAReassignImpl
```cpp
STATISTIC(NumNSAInstructions,
          "Number of NSA instructions with non-sequential address found");
STATISTIC(NumNSAConverted,
          "Number of NSA instructions changed to sequential");

namespace {
class GCNNSAReassignImpl {
public:
  GCNNSAReassignImpl(VirtRegMap *VM, LiveRegMatrix *LM, LiveIntervals *LS)
      : VRM(VM), LRM(LM), LIS(LS) {}

  bool run(MachineFunction &MF);

private:
  enum NSA_Status {
    NOT_NSA,        // Not an NSA instruction
    FIXED,          // NSA which we cannot modify
    NON_CONTIGUOUS, // NSA with non-sequential address which we can try
                    // to optimize.
    CONTIGUOUS      // NSA with all sequential address registers
  };

  const GCNSubtarget *ST;

  const MachineRegisterInfo *MRI;

  const SIRegisterInfo *TRI;

  VirtRegMap *VRM;

  LiveRegMatrix *LRM;

  LiveIntervals *LIS;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNNSAReassignImpl`, `NSA_Status`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNNSAReassignImpl`, `NSA_Status`。

### Lines 66-98: Declares class GCNNSAReassignLegacy
```cpp
  unsigned MaxNumVGPRs;

  const MCPhysReg *CSRegs;

  NSA_Status CheckNSA(const MachineInstr &MI, bool Fast = false) const;

  bool tryAssignRegisters(SmallVectorImpl<LiveInterval *> &Intervals,
                          unsigned StartReg) const;

  bool canAssign(unsigned StartReg, unsigned NumRegs) const;

  bool scavengeRegs(SmallVectorImpl<LiveInterval *> &Intervals) const;
};

class GCNNSAReassignLegacy : public MachineFunctionPass {
public:
  static char ID;

  GCNNSAReassignLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override { return "GCN NSA Reassign"; };

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.addRequired<VirtRegMapWrapperLegacy>();
    AU.addRequired<LiveRegMatrixWrapperLegacy>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNNSAReassignLegacy`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNNSAReassignLegacy`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 99-130: Registers LLVM passes
```cpp
} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(GCNNSAReassignLegacy, DEBUG_TYPE, "GCN NSA Reassign",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(VirtRegMapWrapperLegacy)
INITIALIZE_PASS_DEPENDENCY(LiveRegMatrixWrapperLegacy)
INITIALIZE_PASS_END(GCNNSAReassignLegacy, DEBUG_TYPE, "GCN NSA Reassign", false,
                    false)

char GCNNSAReassignLegacy::ID = 0;

char &llvm::GCNNSAReassignID = GCNNSAReassignLegacy::ID;

bool GCNNSAReassignImpl::tryAssignRegisters(
    SmallVectorImpl<LiveInterval *> &Intervals, unsigned StartReg) const {
  unsigned NumRegs = Intervals.size();

  for (unsigned N = 0; N < NumRegs; ++N)
    if (VRM->hasPhys(Intervals[N]->reg()))
      LRM->unassign(*Intervals[N]);

  for (unsigned N = 0; N < NumRegs; ++N)
    if (LRM->checkInterference(*Intervals[N], MCRegister::from(StartReg + N)))
      return false;

  for (unsigned N = 0; N < NumRegs; ++N)
    LRM->assign(*Intervals[N], MCRegister::from(StartReg + N));

  return true;
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `GCNNSAReassignImpl::tryAssignRegisters`, `MCRegister::from`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`GCNNSAReassignImpl::tryAssignRegisters`, `MCRegister::from`。

### Lines 131-164: Implements GCNNSAReassignImpl::canAssign
```cpp
bool GCNNSAReassignImpl::canAssign(unsigned StartReg, unsigned NumRegs) const {
  for (unsigned N = 0; N < NumRegs; ++N) {
    unsigned Reg = StartReg + N;
    if (!MRI->isAllocatable(Reg))
      return false;

    for (unsigned I = 0; CSRegs[I]; ++I)
      if (TRI->isSubRegisterEq(Reg, CSRegs[I]) &&
          !LRM->isPhysRegUsed(CSRegs[I]))
      return false;
  }

  return true;
}

bool GCNNSAReassignImpl::scavengeRegs(
    SmallVectorImpl<LiveInterval *> &Intervals) const {
  unsigned NumRegs = Intervals.size();

  if (NumRegs > MaxNumVGPRs)
    return false;
  unsigned MaxReg = MaxNumVGPRs - NumRegs + AMDGPU::VGPR0;

  for (unsigned Reg = AMDGPU::VGPR0; Reg <= MaxReg; ++Reg) {
    if (!canAssign(Reg, NumRegs))
      continue;

    if (tryAssignRegisters(Intervals, Reg))
      return true;
  }

  return false;
}

```
**EN:** This section contains concrete logic for GCNNSAReassignImpl::canAssign. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNNSAReassignImpl::canAssign`, `GCNNSAReassignImpl::scavengeRegs`.
**CN:** 本节包含与 GCNNSAReassignImpl::canAssign 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNNSAReassignImpl::canAssign`, `GCNNSAReassignImpl::scavengeRegs`。

### Lines 165-195: Implements GCNNSAReassignImpl::CheckNSA
```cpp
GCNNSAReassignImpl::NSA_Status
GCNNSAReassignImpl::CheckNSA(const MachineInstr &MI, bool Fast) const {
  const AMDGPU::MIMGInfo *Info = AMDGPU::getMIMGInfo(MI.getOpcode());
  if (!Info)
    return NSA_Status::NOT_NSA;

  switch (Info->MIMGEncoding) {
  case AMDGPU::MIMGEncGfx10NSA:
  case AMDGPU::MIMGEncGfx11NSA:
    break;
  default:
    return NSA_Status::NOT_NSA;
  }

  int VAddr0Idx =
    AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::vaddr0);

  unsigned VgprBase = 0;
  bool NSA = false;
  for (unsigned I = 0; I < Info->VAddrOperands; ++I) {
    const MachineOperand &Op = MI.getOperand(VAddr0Idx + I);
    Register Reg = Op.getReg();
    if (Reg.isPhysical() || !VRM->isAssignedReg(Reg))
      return NSA_Status::FIXED;

    Register PhysReg = VRM->getPhys(Reg);

    if (!Fast) {
      if (!PhysReg)
        return NSA_Status::FIXED;

```
**EN:** This section contains concrete logic for GCNNSAReassignImpl::CheckNSA. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNNSAReassignImpl::CheckNSA`, `AMDGPU::getMIMGInfo`, `AMDGPU::getNamedOperandIdx`.
**CN:** 本节包含与 GCNNSAReassignImpl::CheckNSA 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNNSAReassignImpl::CheckNSA`, `AMDGPU::getMIMGInfo`, `AMDGPU::getNamedOperandIdx`。

### Lines 196-228: Conditional logic and checks
```cpp
      // TODO: address the below limitation to handle GFX11 BVH instructions
      // Bail if address is not a VGPR32. That should be possible to extend the
      // optimization to work with subregs of a wider register tuples, but the
      // logic to find free registers will be much more complicated with much
      // less chances for success. That seems reasonable to assume that in most
      // cases a tuple is used because a vector variable contains different
      // parts of an address and it is either already consecutive or cannot
      // be reassigned if not. If needed it is better to rely on register
      // coalescer to process such address tuples.
      if (TRI->getRegSizeInBits(*MRI->getRegClass(Reg)) != 32 || Op.getSubReg())
        return NSA_Status::FIXED;

      // InlineSpiller does not call LRM::assign() after an LI split leaving
      // it in an inconsistent state, so we cannot call LRM::unassign().
      // See llvm bug #48911.
      // Skip reassign if a register has originated from such split.
      // FIXME: Remove the workaround when bug #48911 is fixed.
      if (VRM->getPreSplitReg(Reg))
        return NSA_Status::FIXED;

      const MachineInstr *Def = MRI->getUniqueVRegDef(Reg);

      if (Def && Def->isCopy() && Def->getOperand(1).getReg() == PhysReg)
        return NSA_Status::FIXED;

      for (auto U : MRI->use_nodbg_operands(Reg)) {
        if (U.isImplicit())
          return NSA_Status::FIXED;
        const MachineInstr *UseInst = U.getParent();
        if (UseInst->isCopy() && UseInst->getOperand(0).getReg() == PhysReg)
          return NSA_Status::FIXED;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `LRM::assign`, `LRM::unassign`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`LRM::assign`, `LRM::unassign`。

### Lines 229-256: Conditional logic and checks
```cpp
      if (!LIS->hasInterval(Reg))
        return NSA_Status::FIXED;
    }

    if (I == 0)
      VgprBase = PhysReg;
    else if (VgprBase + I != PhysReg)
      NSA = true;
  }

  return NSA ? NSA_Status::NON_CONTIGUOUS : NSA_Status::CONTIGUOUS;
}

bool GCNNSAReassignImpl::run(MachineFunction &MF) {
  ST = &MF.getSubtarget<GCNSubtarget>();
  if (!ST->hasNSAEncoding() || !ST->hasNonNSAEncoding())
    return false;

  MRI = &MF.getRegInfo();
  TRI = ST->getRegisterInfo();

  const SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  MaxNumVGPRs = ST->getMaxNumVGPRs(MF);
  MaxNumVGPRs = std::min(
      ST->getMaxNumVGPRs(MFI->getOccupancy(), MFI->getDynamicVGPRBlockSize()),
      MaxNumVGPRs);
  CSRegs = MRI->getCalleeSavedRegs();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNNSAReassignImpl::run`, `std::min`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNNSAReassignImpl::run`, `std::min`。

### Lines 257-287: Type declarations and aliases
```cpp
  using Candidate = std::pair<const MachineInstr*, bool>;
  SmallVector<Candidate, 32> Candidates;
  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineInstr &MI : MBB) {
      switch (CheckNSA(MI)) {
      default:
        continue;
      case NSA_Status::CONTIGUOUS:
        Candidates.push_back(std::pair(&MI, true));
        break;
      case NSA_Status::NON_CONTIGUOUS:
        Candidates.push_back(std::pair(&MI, false));
        ++NumNSAInstructions;
        break;
      }
    }
  }

  bool Changed = false;
  for (auto &C : Candidates) {
    if (C.second)
      continue;

    const MachineInstr *MI = C.first;
    if (CheckNSA(*MI, true) == NSA_Status::CONTIGUOUS) {
      // Already happen to be fixed.
      C.second = true;
      ++NumNSAConverted;
      continue;
    }

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `std::pair`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`std::pair`。

### Lines 288-319: Conditional logic and checks
```cpp
    const AMDGPU::MIMGInfo *Info = AMDGPU::getMIMGInfo(MI->getOpcode());
    int VAddr0Idx =
      AMDGPU::getNamedOperandIdx(MI->getOpcode(), AMDGPU::OpName::vaddr0);

    SmallVector<LiveInterval *, 16> Intervals;
    SmallVector<MCRegister, 16> OrigRegs;
    SlotIndex MinInd, MaxInd;
    for (unsigned I = 0; I < Info->VAddrOperands; ++I) {
      const MachineOperand &Op = MI->getOperand(VAddr0Idx + I);
      Register Reg = Op.getReg();
      LiveInterval *LI = &LIS->getInterval(Reg);
      if (llvm::is_contained(Intervals, LI)) {
        // Same register used, unable to make sequential
        Intervals.clear();
        break;
      }
      Intervals.push_back(LI);
      OrigRegs.push_back(VRM->getPhys(Reg));
      if (LI->empty()) {
        // The address input is undef, so it doesn't contribute to the relevant
        // range. Seed a reasonable index range if required.
        if (I == 0)
          MinInd = MaxInd = LIS->getInstructionIndex(*MI);
        continue;
      }
      MinInd = I != 0 ? std::min(MinInd, LI->beginIndex()) : LI->beginIndex();
      MaxInd = I != 0 ? std::max(MaxInd, LI->endIndex()) : LI->endIndex();
    }

    if (Intervals.empty())
      continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getMIMGInfo`, `AMDGPU::getNamedOperandIdx`, `llvm::is_contained`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getMIMGInfo`, `AMDGPU::getNamedOperandIdx`, `llvm::is_contained`。

### Lines 320-353: Conditional logic and checks
```cpp
    LLVM_DEBUG(dbgs() << "Attempting to reassign NSA: " << *MI
                      << "\tOriginal allocation:\t";
               for (auto *LI
                    : Intervals) dbgs()
               << " " << llvm::printReg((VRM->getPhys(LI->reg())), TRI);
               dbgs() << '\n');

    bool Success = scavengeRegs(Intervals);
    if (!Success) {
      LLVM_DEBUG(dbgs() << "\tCannot reallocate.\n");
      if (VRM->hasPhys(Intervals.back()->reg())) // Did not change allocation.
        continue;
    } else {
      // Check we did not make it worse for other instructions.
      auto *I =
          std::lower_bound(Candidates.begin(), &C, MinInd,
                           [this](const Candidate &C, SlotIndex I) {
                             return LIS->getInstructionIndex(*C.first) < I;
                           });
      for (auto *E = Candidates.end();
           Success && I != E && LIS->getInstructionIndex(*I->first) < MaxInd;
           ++I) {
        if (I->second && CheckNSA(*I->first, true) < NSA_Status::CONTIGUOUS) {
          Success = false;
          LLVM_DEBUG(dbgs() << "\tNSA conversion conflict with " << *I->first);
        }
      }
    }

    if (!Success) {
      for (unsigned I = 0; I < Info->VAddrOperands; ++I)
        if (VRM->hasPhys(Intervals[I]->reg()))
          LRM->unassign(*Intervals[I]);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::printReg`, `std::lower_bound`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::printReg`, `std::lower_bound`。

### Lines 354-382: Result computation and returns
```cpp
      for (unsigned I = 0; I < Info->VAddrOperands; ++I)
        LRM->assign(*Intervals[I], OrigRegs[I]);

      continue;
    }

    C.second = true;
    ++NumNSAConverted;
    LLVM_DEBUG(
        dbgs() << "\tNew allocation:\t\t ["
               << llvm::printReg((VRM->getPhys(Intervals.front()->reg())), TRI)
               << " : "
               << llvm::printReg((VRM->getPhys(Intervals.back()->reg())), TRI)
               << "]\n");
    Changed = true;
  }

  return Changed;
}

bool GCNNSAReassignLegacy::runOnMachineFunction(MachineFunction &MF) {
  auto *VRM = &getAnalysis<VirtRegMapWrapperLegacy>().getVRM();
  auto *LRM = &getAnalysis<LiveRegMatrixWrapperLegacy>().getLRM();
  auto *LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();

  GCNNSAReassignImpl Impl(VRM, LRM, LIS);
  return Impl.run(MF);
}

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `llvm::printReg`, `GCNNSAReassignLegacy::runOnMachineFunction`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`llvm::printReg`, `GCNNSAReassignLegacy::runOnMachineFunction`。

### Lines 383-393: Implements GCNNSAReassignPass::run
```cpp
PreservedAnalyses
GCNNSAReassignPass::run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM) {
  auto &VRM = MFAM.getResult<VirtRegMapAnalysis>(MF);
  auto &LRM = MFAM.getResult<LiveRegMatrixAnalysis>(MF);
  auto &LIS = MFAM.getResult<LiveIntervalsAnalysis>(MF);

  GCNNSAReassignImpl Impl(&VRM, &LRM, &LIS);
  Impl.run(MF);
  return PreservedAnalyses::all();
}
```
**EN:** This section contains concrete logic for GCNNSAReassignPass::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNNSAReassignPass::run`, `PreservedAnalyses::all`.
**CN:** 本节包含与 GCNNSAReassignPass::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNNSAReassignPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNNSAReassignImpl`, `NSA_Status`, `GCNNSAReassignLegacy`, `MachineFunctionPass::getAnalysisUsage`, `GCNNSAReassignImpl::tryAssignRegisters`, `MCRegister::from`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNNSAReassign.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"SIMachineFunctionInfo.h"`
- `"SIRegisterInfo.h"`
- `"llvm/ADT/Statistic.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/LiveRegMatrix.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/VirtRegMap.h"`
- `"llvm/InitializePasses.h"`
