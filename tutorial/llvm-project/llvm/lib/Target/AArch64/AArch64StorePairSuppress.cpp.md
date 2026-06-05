# AArch64StorePairSuppress.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64StorePairSuppress.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements AArch64 backend logic for the AArch64 backend. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Documented code section
```cpp
//===--- AArch64StorePairSuppress.cpp --- Suppress store pair formation ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass identifies floating point stores that should not be combined into
// store pairs. Later we may do the same for floating point loads.
// ===---------------------------------------------------------------------===//

#include "AArch64InstrInfo.h"
#include "AArch64Subtarget.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineTraceMetrics.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetSchedule.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

#define DEBUG_TYPE "aarch64-stp-suppress"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 27-50: Preprocessor configuration
```cpp

#define STPSUPPRESS_PASS_NAME "AArch64 Store Pair Suppression"

namespace {
class AArch64StorePairSuppress : public MachineFunctionPass {
  const AArch64InstrInfo *TII;
  const TargetRegisterInfo *TRI;
  const MachineRegisterInfo *MRI;
  TargetSchedModel SchedModel;
  MachineTraceMetrics *Traces;
  MachineTraceMetrics::Ensemble *MinInstr;

public:
  static char ID;
  AArch64StorePairSuppress() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return STPSUPPRESS_PASS_NAME; }

  bool runOnMachineFunction(MachineFunction &F) override;

private:
  bool shouldAddSTPToBlock(const MachineBasicBlock *BB);

  bool isNarrowFPStore(const MachineInstr &MI);
```
**EN:** This block defines compile-time guards or macros that shape the remainder of the translation unit.  
**CN:** 该代码块定义编译期开关或宏，影响后续整个翻译单元的行为。
### Lines 51-77: Function getAnalysisUsage
```cpp

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineTraceMetricsWrapperPass>();
    AU.addPreserved<MachineTraceMetricsWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
char AArch64StorePairSuppress::ID = 0;
} // anonymous

INITIALIZE_PASS(AArch64StorePairSuppress, "aarch64-stp-suppress",
                STPSUPPRESS_PASS_NAME, false, false)

FunctionPass *llvm::createAArch64StorePairSuppressPass() {
  return new AArch64StorePairSuppress();
}

/// Return true if an STP can be added to this block without increasing the
/// critical resource height. STP is good to form in Ld/St limited blocks and
/// bad to form in float-point limited blocks. This is true independent of the
/// critical path. If the critical path is longer than the resource height, the
/// extra vector ops can limit physreg renaming. Otherwise, it could simply
/// oversaturate the vector units.
bool AArch64StorePairSuppress::shouldAddSTPToBlock(const MachineBasicBlock *BB) {
  if (!MinInstr)
    MinInstr = Traces->getEnsemble(MachineTraceStrategy::TS_MinInstrCount);
```
**EN:** This block implements getAnalysisUsage, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getAnalysisUsage，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 78-105: Class for
```cpp

  MachineTraceMetrics::Trace BBTrace = MinInstr->getTrace(BB);
  unsigned ResLength = BBTrace.getResourceLength();

  // Get the machine model's scheduling class for STPDi and STRDui.
  // Bypass TargetSchedule's SchedClass resolution since we only have an opcode.
  unsigned SCIdx = TII->get(AArch64::STPDi).getSchedClass();
  const MCSchedClassDesc *PairSCDesc =
      SchedModel.getMCSchedModel()->getSchedClassDesc(SCIdx);

  unsigned SCIdx2 = TII->get(AArch64::STRDui).getSchedClass();
  const MCSchedClassDesc *SingleSCDesc =
      SchedModel.getMCSchedModel()->getSchedClassDesc(SCIdx2);

  // If a subtarget does not define resources for STPDi, bail here.
  if (PairSCDesc->isValid() && !PairSCDesc->isVariant() &&
      SingleSCDesc->isValid() && !SingleSCDesc->isVariant()) {
    // Compute the new critical resource length after replacing 2 separate
    // STRDui with one STPDi.
    unsigned ResLenWithSTP =
        BBTrace.getResourceLength({}, PairSCDesc, {SingleSCDesc, SingleSCDesc});
    if (ResLenWithSTP > ResLength) {
      LLVM_DEBUG(dbgs() << "  Suppress STP in BB: " << BB->getNumber()
                        << " resources " << ResLength << " -> " << ResLenWithSTP
                        << "\n");
      return false;
    }
  }
```
**EN:** This block defines for, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 for，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 106-130: Function getMatchingPairOpcode
```cpp
  return true;
}

/// Return true if this is a floating-point store smaller than the V reg. On
/// cyclone, these require a vector shuffle before storing a pair.
/// Ideally we would call getMatchingPairOpcode() and have the machine model
/// tell us if it's profitable with no cpu knowledge here.
///
/// FIXME: We plan to develop a decent Target abstraction for simple loads and
/// stores. Until then use a nasty switch similar to AArch64LoadStoreOptimizer.
bool AArch64StorePairSuppress::isNarrowFPStore(const MachineInstr &MI) {
  switch (MI.getOpcode()) {
  default:
    return false;
  case AArch64::STRSui:
  case AArch64::STRDui:
  case AArch64::STURSi:
  case AArch64::STURDi:
    return true;
  }
}

bool AArch64StorePairSuppress::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()) || MF.getFunction().hasOptSize())
    return false;
```
**EN:** This block implements getMatchingPairOpcode, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getMatchingPairOpcode，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 131-148: Core AArch64 backend logic
```cpp

  const AArch64Subtarget &ST = MF.getSubtarget<AArch64Subtarget>();
  if (!ST.enableStorePairSuppress())
    return false;

  TII = ST.getInstrInfo();
  TRI = ST.getRegisterInfo();
  MRI = &MF.getRegInfo();
  SchedModel.init(&ST);
  Traces = &getAnalysis<MachineTraceMetricsWrapperPass>().getMTM();
  MinInstr = nullptr;

  LLVM_DEBUG(dbgs() << "*** " << getPassName() << ": " << MF.getName() << '\n');

  if (!SchedModel.hasInstrSchedModel()) {
    LLVM_DEBUG(dbgs() << "  Skipping pass: no machine model present.\n");
    return false;
  }
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 149-176: Documented code section
```cpp

  // Check for a sequence of stores to the same base address. We don't need to
  // precisely determine whether a store pair can be formed. But we do want to
  // filter out most situations where we can't form store pairs to avoid
  // computing trace metrics in those cases.
  for (auto &MBB : MF) {
    bool SuppressSTP = false;
    unsigned PrevBaseReg = 0;
    for (auto &MI : MBB) {
      if (!isNarrowFPStore(MI))
        continue;
      const MachineOperand *BaseOp;
      int64_t Offset;
      bool OffsetIsScalable;
      if (TII->getMemOperandWithOffset(MI, BaseOp, Offset, OffsetIsScalable,
                                       TRI) &&
          BaseOp->isReg()) {
        Register BaseReg = BaseOp->getReg();
        if (PrevBaseReg == BaseReg) {
          // If this block can take STPs, skip ahead to the next block.
          if (!SuppressSTP && shouldAddSTPToBlock(MI.getParent()))
            break;
          // Otherwise, continue unpairing the stores in this block.
          LLVM_DEBUG(dbgs() << "Unpairing store " << MI << "\n");
          SuppressSTP = true;
          TII->suppressLdStPair(MI);
        }
        PrevBaseReg = BaseReg;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 177-184: Core AArch64 backend logic
```cpp
      } else
        PrevBaseReg = 0;
    }
  }
  // This pass just sets some internal MachineMemOperand flags. It can't really
  // invalidate anything.
  return false;
}
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64InstrInfo.h, AArch64Subtarget.h **CN:** 目标本地依赖：AArch64InstrInfo.h, AArch64Subtarget.h
- **EN:** Core LLVM interfaces: llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineTraceMetrics.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetSchedule.h, llvm/Support/Debug.h, llvm/Support/raw_ostream.h **CN:** 核心 LLVM 接口：llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineTraceMetrics.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetSchedule.h, llvm/Support/Debug.h, llvm/Support/raw_ostream.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
