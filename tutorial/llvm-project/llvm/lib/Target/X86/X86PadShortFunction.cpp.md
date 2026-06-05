# X86PadShortFunction.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86PadShortFunction.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-------- X86PadShortFunction.cpp - pad short functions -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the pass which will pad short functions to prevent
// a stall if a function returns before the return address is ready. This
// is needed for some Intel Atom processors.
//
//===----------------------------------------------------------------------===//


#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/CodeGen/LazyMachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineSizeOpts.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetSchedule.h"
#include "llvm/IR/Function.h"

using namespace llvm;

```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#define DEBUG_TYPE "x86-pad-short-functions"

STATISTIC(NumBBsPadded, "Number of basic blocks padded");

namespace {
  struct VisitedBBInfo {
    // HasReturn - Whether the BB contains a return instruction
    bool HasReturn = false;

    // Cycles - Number of cycles until return if HasReturn is true, otherwise
    // number of cycles until end of the BB
    unsigned int Cycles = 0;

    VisitedBBInfo() = default;
    VisitedBBInfo(bool HasReturn, unsigned int Cycles)
      : HasReturn(HasReturn), Cycles(Cycles) {}
  };

  struct PadShortFunc : public MachineFunctionPass {
    static char ID;
    PadShortFunc() : MachineFunctionPass(ID) {}

    bool runOnMachineFunction(MachineFunction &MF) override;

    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.addRequired<ProfileSummaryInfoWrapperPass>();
      AU.addRequired<LazyMachineBlockFrequencyInfoPass>();
      AU.addPreserved<LazyMachineBlockFrequencyInfoPass>();
      MachineFunctionPass::getAnalysisUsage(AU);
    }
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include VisitedBBInfo, PadShortFunc. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 VisitedBBInfo, PadShortFunc。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp

    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }

    StringRef getPassName() const override {
      return "X86 Atom pad short functions";
    }

  private:
    void findReturns(MachineBasicBlock *MBB,
                     unsigned int Cycles = 0);

    bool cyclesUntilReturn(MachineBasicBlock *MBB,
                           unsigned int &Cycles);

    void addPadding(MachineBasicBlock *MBB,
                    MachineBasicBlock::iterator &MBBI,
                    unsigned int NOOPsToAdd);

    const unsigned int Threshold = 4;

    // ReturnBBs - Maps basic blocks that return to the minimum number of
    // cycles until the return, starting from the entry block.
    DenseMap<MachineBasicBlock*, unsigned int> ReturnBBs;

    // VisitedBBs - Cache of previously visited BBs.
    DenseMap<MachineBasicBlock*, VisitedBBInfo> VisitedBBs;

    TargetSchedModel TSM;
```
**EN:** This section implements supporting logic and local data handling for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  };

  char PadShortFunc::ID = 0;
}

FunctionPass *llvm::createX86PadShortFunctions() {
  return new PadShortFunc();
}

/// runOnMachineFunction - Loop over all of the basic blocks, inserting
/// NOOP instructions before early exits.
bool PadShortFunc::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "Start X86PadShortFunctionPass\n";);
  if (skipFunction(MF.getFunction()))
    return false;

  if (MF.getFunction().hasOptSize())
    return false;

  if (!MF.getSubtarget<X86Subtarget>().padShortFunctions())
    return false;

  TSM.init(&MF.getSubtarget());

  auto *PSI =
      &getAnalysis<ProfileSummaryInfoWrapperPass>().getPSI();
  auto *MBFI = (PSI && PSI->hasProfileSummary()) ?
               &getAnalysis<LazyMachineBlockFrequencyInfoPass>().getBFI() :
               nullptr;

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  // Search through basic blocks and mark the ones that have early returns
  ReturnBBs.clear();
  VisitedBBs.clear();
  findReturns(&MF.front());

  bool MadeChange = false;

  // Pad the identified basic blocks with NOOPs
  for (const auto &ReturnBB : ReturnBBs) {
    MachineBasicBlock *MBB = ReturnBB.first;
    unsigned Cycles = ReturnBB.second;

    if (llvm::shouldOptimizeForSize(MBB, PSI, MBFI))
      continue;

    if (Cycles < Threshold) {
      // BB ends in a return. Skip over any DBG_VALUE instructions
      // trailing the terminator.
      assert(MBB->size() > 0 &&
             "Basic block should contain at least a RET but is empty");
      MachineBasicBlock::iterator ReturnLoc = --MBB->end();

      while (ReturnLoc->isDebugInstr())
        --ReturnLoc;
      assert(ReturnLoc->isReturn() && !ReturnLoc->isCall() &&
             "Basic block does not end with RET");

      addPadding(MBB, ReturnLoc, Threshold - Cycles);
      NumBBsPadded++;
      MadeChange = true;
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    }
  }
  LLVM_DEBUG(dbgs() << "End X86PadShortFunctionPass\n";);
  return MadeChange;
}

/// findReturn - Starting at MBB, follow control flow and add all
/// basic blocks that contain a return to ReturnBBs.
void PadShortFunc::findReturns(MachineBasicBlock *MBB, unsigned int Cycles) {
  // If this BB has a return, note how many cycles it takes to get there.
  bool hasReturn = cyclesUntilReturn(MBB, Cycles);
  if (Cycles >= Threshold)
    return;

  if (hasReturn) {
    unsigned int &NumCycles = ReturnBBs[MBB];
    NumCycles = std::max(NumCycles, Cycles);
    return;
  }

  // Follow branches in BB and look for returns
  for (MachineBasicBlock *Succ : MBB->successors())
    if (Succ != MBB)
      findReturns(Succ, Cycles);
}

/// cyclesUntilReturn - return true if the MBB has a return instruction,
/// and return false otherwise.
/// Cycles will be incremented by the number of cycles taken to reach the
/// return or the end of the BB, whichever occurs first.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
bool PadShortFunc::cyclesUntilReturn(MachineBasicBlock *MBB,
                                     unsigned int &Cycles) {
  // Return cached result if BB was previously visited
  auto [It, Inserted] = VisitedBBs.try_emplace(MBB);
  if (!Inserted) {
    VisitedBBInfo BBInfo = It->second;
    Cycles += BBInfo.Cycles;
    return BBInfo.HasReturn;
  }

  unsigned int CyclesToEnd = 0;

  for (MachineInstr &MI : *MBB) {
    // Mark basic blocks with a return instruction. Calls to other
    // functions do not count because the called function will be padded,
    // if necessary.
    if (MI.isReturn() && !MI.isCall()) {
      It->second = VisitedBBInfo(true, CyclesToEnd);
      Cycles += CyclesToEnd;
      return true;
    }

    CyclesToEnd += TSM.computeInstrLatency(&MI);
  }

  It->second = VisitedBBInfo(false, CyclesToEnd);
  Cycles += CyclesToEnd;
  return false;
}

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. Key symbols include PadShortFunc::cyclesUntilReturn. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。关键符号包括 PadShortFunc::cyclesUntilReturn。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-221: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
/// addPadding - Add the given number of NOOP instructions to the function
/// just prior to the return at MBBI
void PadShortFunc::addPadding(MachineBasicBlock *MBB,
                              MachineBasicBlock::iterator &MBBI,
                              unsigned int NOOPsToAdd) {
  const DebugLoc &DL = MBBI->getDebugLoc();
  unsigned IssueWidth = TSM.getIssueWidth();

  for (unsigned i = 0, e = IssueWidth * NOOPsToAdd; i != e; ++i)
    BuildMI(*MBB, MBBI, DL, TSM.getInstrInfo()->get(X86::NOOP));
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: VisitedBBInfo, PadShortFunc. / 重要符号：VisitedBBInfo, PadShortFunc。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/Statistic.h, llvm/Analysis/ProfileSummaryInfo.h, llvm/CodeGen/LazyMachineBlockFrequencyInfo.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineSizeOpts.h, llvm/CodeGen/Passes.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/Statistic.h, llvm/Analysis/ProfileSummaryInfo.h, llvm/CodeGen/LazyMachineBlockFrequencyInfo.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineSizeOpts.h, llvm/CodeGen/Passes.h。
- Additional include dependencies: 2 more headers. / 额外包含依赖：还有 2 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
