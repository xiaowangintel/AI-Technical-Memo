# SILowerWWMCopies.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SILowerWWMCopies.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SILowerWWMCopies for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SILowerWWMCopies 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: File banner, license, and overview
```cpp
//===-- SILowerWWMCopies.cpp - Lower Copies after regalloc ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Lowering the WWM_COPY instructions for various register classes.
/// AMDGPU target generates WWM_COPY instruction to differentiate WWM
/// copy from COPY. This pass generates the necessary exec mask manipulation
/// instructions to replicate 'Whole Wave Mode' and lowers WWM_COPY back to
/// COPY.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 18-39: Header dependencies and setup
```cpp
#include "SILowerWWMCopies.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/VirtRegMap.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "si-lower-wwm-copies"

namespace {

class SILowerWWMCopies {
public:
  SILowerWWMCopies(LiveIntervals *LIS, SlotIndexes *SI, VirtRegMap *VRM)
      : LIS(LIS), Indexes(SI), VRM(VRM) {}
  bool run(MachineFunction &MF);

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `SILowerWWMCopies`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`SILowerWWMCopies`。

### Lines 40-61: Declares class SILowerWWMCopiesLegacy
```cpp
private:
  bool isSCCLiveAtMI(const MachineInstr &MI);
  void addToWWMSpills(MachineFunction &MF, Register Reg);

  LiveIntervals *LIS;
  SlotIndexes *Indexes;
  VirtRegMap *VRM;
  const SIRegisterInfo *TRI;
  const MachineRegisterInfo *MRI;
  SIMachineFunctionInfo *MFI;
};

class SILowerWWMCopiesLegacy : public MachineFunctionPass {
public:
  static char ID;

  SILowerWWMCopiesLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override { return "SI Lower WWM Copies"; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SILowerWWMCopiesLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SILowerWWMCopiesLegacy`。

### Lines 62-83: Registers LLVM passes
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addUsedIfAvailable<LiveIntervalsWrapperPass>();
    AU.addUsedIfAvailable<SlotIndexesWrapperPass>();
    AU.addUsedIfAvailable<VirtRegMapWrapperLegacy>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(SILowerWWMCopiesLegacy, DEBUG_TYPE, "SI Lower WWM Copies",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(VirtRegMapWrapperLegacy)
INITIALIZE_PASS_END(SILowerWWMCopiesLegacy, DEBUG_TYPE, "SI Lower WWM Copies",
                    false, false)

char SILowerWWMCopiesLegacy::ID = 0;

char &llvm::SILowerWWMCopiesLegacyID = SILowerWWMCopiesLegacy::ID;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`。

### Lines 84-105: Implements SILowerWWMCopies::isSCCLiveAtMI
```cpp
bool SILowerWWMCopies::isSCCLiveAtMI(const MachineInstr &MI) {
  // We can't determine the liveness info if LIS isn't available. Early return
  // in that case and always assume SCC is live.
  if (!LIS)
    return true;

  LiveRange &LR =
      LIS->getRegUnit(*MCRegUnitIterator(MCRegister::from(AMDGPU::SCC), TRI));
  SlotIndex Idx = LIS->getInstructionIndex(MI);
  return LR.liveAt(Idx);
}

// If \p Reg is assigned with a physical VGPR, add the latter into wwm-spills
// for preserving its entire lanes at function prolog/epilog.
void SILowerWWMCopies::addToWWMSpills(MachineFunction &MF, Register Reg) {
  if (Reg.isPhysical())
    return;

  // FIXME: VRM may be null here.
  MCRegister PhysReg = VRM->getPhys(Reg);
  assert(PhysReg && "should have allocated a physical register");

```
**EN:** This section contains concrete logic for SILowerWWMCopies::isSCCLiveAtMI. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SILowerWWMCopies::isSCCLiveAtMI`, `MCRegister::from`, `SILowerWWMCopies::addToWWMSpills`.
**CN:** 本节包含与 SILowerWWMCopies::isSCCLiveAtMI 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SILowerWWMCopies::isSCCLiveAtMI`, `MCRegister::from`, `SILowerWWMCopies::addToWWMSpills`。

### Lines 106-129: Declares allocateWWMSpill
```cpp
  MFI->allocateWWMSpill(MF, PhysReg);
}

bool SILowerWWMCopiesLegacy::runOnMachineFunction(MachineFunction &MF) {
  auto *LISWrapper = getAnalysisIfAvailable<LiveIntervalsWrapperPass>();
  auto *LIS = LISWrapper ? &LISWrapper->getLIS() : nullptr;

  auto *SIWrapper = getAnalysisIfAvailable<SlotIndexesWrapperPass>();
  auto *Indexes = SIWrapper ? &SIWrapper->getSI() : nullptr;

  auto *VRMWrapper = getAnalysisIfAvailable<VirtRegMapWrapperLegacy>();
  auto *VRM = VRMWrapper ? &VRMWrapper->getVRM() : nullptr;

  SILowerWWMCopies Impl(LIS, Indexes, VRM);
  return Impl.run(MF);
}

PreservedAnalyses
SILowerWWMCopiesPass::run(MachineFunction &MF,
                          MachineFunctionAnalysisManager &MFAM) {
  auto *LIS = MFAM.getCachedResult<LiveIntervalsAnalysis>(MF);
  auto *Indexes = MFAM.getCachedResult<SlotIndexesAnalysis>(MF);
  auto *VRM = MFAM.getCachedResult<VirtRegMapAnalysis>(MF);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SILowerWWMCopiesLegacy::runOnMachineFunction`, `SILowerWWMCopiesPass::run`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SILowerWWMCopiesLegacy::runOnMachineFunction`, `SILowerWWMCopiesPass::run`。

### Lines 130-151: Declares Impl
```cpp
  SILowerWWMCopies Impl(LIS, Indexes, VRM);
  Impl.run(MF);
  return PreservedAnalyses::all();
}

bool SILowerWWMCopies::run(MachineFunction &MF) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const SIInstrInfo *TII = ST.getInstrInfo();

  MFI = MF.getInfo<SIMachineFunctionInfo>();
  TRI = ST.getRegisterInfo();
  MRI = &MF.getRegInfo();

  if (!MFI->hasVRegFlags())
    return false;

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      if (MI.getOpcode() != AMDGPU::WWM_COPY)
        continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `PreservedAnalyses::all`, `SILowerWWMCopies::run`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`PreservedAnalyses::all`, `SILowerWWMCopies::run`。

### Lines 152-173: Result computation and returns
```cpp
      // TODO: Club adjacent WWM ops between same exec save/restore
      assert(TII->isVGPRCopy(MI));

      // For WWM vector copies, manipulate the exec mask around the copy
      // instruction.
      const DebugLoc &DL = MI.getDebugLoc();
      MachineBasicBlock::iterator InsertPt = MI.getIterator();
      Register RegForExecCopy = MFI->getSGPRForEXECCopy();
      TII->insertScratchExecCopy(MF, MBB, InsertPt, DL, RegForExecCopy,
                                 isSCCLiveAtMI(MI), Indexes);
      TII->restoreExec(MF, MBB, ++InsertPt, DL, RegForExecCopy, Indexes);
      addToWWMSpills(MF, MI.getOperand(0).getReg());
      LLVM_DEBUG(dbgs() << "WWM copy manipulation for " << MI);

      // Lower WWM_COPY back to COPY
      MI.setDesc(TII->get(AMDGPU::COPY));
      Changed |= true;
    }
  }

  return Changed;
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SILowerWWMCopies`, `SILowerWWMCopiesLegacy`, `MachineFunctionPass::getAnalysisUsage`, `SILowerWWMCopies::isSCCLiveAtMI`, `MCRegister::from`, `SILowerWWMCopies::addToWWMSpills`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SILowerWWMCopies.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/VirtRegMap.h"`
- `"llvm/InitializePasses.h"`
