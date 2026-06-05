# AArch64PostCoalescerPass.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64PostCoalescerPass.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers AArch64 Post Coalescer pass. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Documented code section
```cpp
//===- AArch64PostCoalescerPass.cpp - AArch64 Post Coalescer pass ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//===----------------------------------------------------------------------===//

#include "AArch64.h"
#include "AArch64MachineFunctionInfo.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "aarch64-post-coalescer"

namespace {

bool runAArch64PostCoalescer(MachineFunction &MF, LiveIntervals &LIS) {
  AArch64FunctionInfo *FuncInfo = MF.getInfo<AArch64FunctionInfo>();
  if (!FuncInfo->hasStreamingModeChanges())
    return false;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 28-53: Core AArch64 backend logic
```cpp

  MachineRegisterInfo &MRI = MF.getRegInfo();
  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : make_early_inc_range(MBB)) {
      switch (MI.getOpcode()) {
      default:
        break;
      case AArch64::COALESCER_BARRIER_FPR16:
      case AArch64::COALESCER_BARRIER_FPR32:
      case AArch64::COALESCER_BARRIER_FPR64:
      case AArch64::COALESCER_BARRIER_FPR128: {
        Register Src = MI.getOperand(1).getReg();
        Register Dst = MI.getOperand(0).getReg();
        if (Src != Dst)
          MRI.replaceRegWith(Dst, Src);

        if (MI.getOperand(1).isUndef())
          for (MachineOperand &MO : MRI.use_operands(Dst))
            MO.setIsUndef();

        // MI must be erased from the basic block before recalculating the live
        // interval.
        LIS.RemoveMachineInstrFromMaps(MI);
        MI.eraseFromParent();
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 54-77: Core AArch64 backend logic
```cpp

        LIS.removeInterval(Src);
        LIS.createAndComputeVirtRegInterval(Src);

        Changed = true;
        break;
      }
      }
    }
  }

  return Changed;
}

struct AArch64PostCoalescerLegacy : public MachineFunctionPass {
  static char ID;

  AArch64PostCoalescerLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "AArch64 Post Coalescer pass";
  }
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 78-104: Function getAnalysisUsage
```cpp

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.addPreserved<LiveIntervalsWrapperPass>();
    AU.addPreserved<SlotIndexesWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

char AArch64PostCoalescerLegacy::ID = 0;

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(AArch64PostCoalescerLegacy, "aarch64-post-coalescer",
                      "AArch64 Post Coalescer Pass", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_END(AArch64PostCoalescerLegacy, "aarch64-post-coalescer",
                    "AArch64 Post Coalescer Pass", false, false)

bool AArch64PostCoalescerLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  auto &LIS = getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  return runAArch64PostCoalescer(MF, LIS);
}
```
**EN:** This block implements getAnalysisUsage, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getAnalysisUsage，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 105-122: Function AArch64PostCoalescerPass::run
```cpp

PreservedAnalyses
AArch64PostCoalescerPass::run(MachineFunction &MF,
                              MachineFunctionAnalysisManager &MFAM) {
  auto &LIS = MFAM.getResult<LiveIntervalsAnalysis>(MF);
  const bool Changed = runAArch64PostCoalescer(MF, LIS);
  if (!Changed)
    return PreservedAnalyses::all();
  PreservedAnalyses PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  PA.preserve<LiveIntervalsAnalysis>();
  PA.preserve<SlotIndexesAnalysis>();
  return PA;
}

FunctionPass *llvm::createAArch64PostCoalescerPass() {
  return new AArch64PostCoalescerLegacy();
}
```
**EN:** This block implements AArch64PostCoalescerPass::run, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64PostCoalescerPass::run，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h, AArch64MachineFunctionInfo.h **CN:** 目标本地依赖：AArch64.h, AArch64MachineFunctionInfo.h
- **EN:** Core LLVM interfaces: llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/InitializePasses.h **CN:** 核心 LLVM 接口：llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/InitializePasses.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
