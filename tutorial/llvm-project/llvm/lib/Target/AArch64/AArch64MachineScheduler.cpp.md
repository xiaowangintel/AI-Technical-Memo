# AArch64MachineScheduler.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64MachineScheduler.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers MI Scheduler for AArch64. / 该文件实现 AArch64 后端中的调度模型。
## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Documented code section
```cpp
//===- AArch64MachineScheduler.cpp - MI Scheduler for AArch64 -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AArch64MachineScheduler.h"
#include "AArch64InstrInfo.h"
#include "AArch64Subtarget.h"
#include "MCTargetDesc/AArch64MCTargetDesc.h"

using namespace llvm;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 15-30: Function needReorderStoreMI
```cpp

static bool needReorderStoreMI(const MachineInstr *MI) {
  if (!MI)
    return false;

  switch (MI->getOpcode()) {
  default:
    return false;
  case AArch64::STURQi:
  case AArch64::STRQui:
    if (!MI->getMF()->getSubtarget<AArch64Subtarget>().isStoreAddressAscend())
      return false;
    [[fallthrough]];
  case AArch64::STPQi:
    return AArch64InstrInfo::getLdStOffsetOp(*MI).isImm();
  }
```
**EN:** This block implements needReorderStoreMI, advancing the file's scheduling models flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 needReorderStoreMI，通过 AArch64 专用的决策与数据处理推进本文件的调度模型流程。
### Lines 31-43: Function mayOverlapWrite
```cpp

  return false;
}

// Return true if two stores with same base address may overlap writes
static bool mayOverlapWrite(const MachineInstr &MI0, const MachineInstr &MI1,
                            int64_t &Off0, int64_t &Off1) {
  const MachineOperand &Base0 = AArch64InstrInfo::getLdStBaseOp(MI0);
  const MachineOperand &Base1 = AArch64InstrInfo::getLdStBaseOp(MI1);

  // May overlapping writes if two store instructions without same base
  if (!Base0.isIdenticalTo(Base1))
    return true;
```
**EN:** This block implements mayOverlapWrite, advancing the file's scheduling models flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 mayOverlapWrite，通过 AArch64 专用的决策与数据处理推进本文件的调度模型流程。
### Lines 44-59: Core AArch64 backend logic
```cpp

  int StoreSize0 = AArch64InstrInfo::getMemScale(MI0);
  int StoreSize1 = AArch64InstrInfo::getMemScale(MI1);
  Off0 = AArch64InstrInfo::hasUnscaledLdStOffset(MI0.getOpcode())
             ? AArch64InstrInfo::getLdStOffsetOp(MI0).getImm()
             : AArch64InstrInfo::getLdStOffsetOp(MI0).getImm() * StoreSize0;
  Off1 = AArch64InstrInfo::hasUnscaledLdStOffset(MI1.getOpcode())
             ? AArch64InstrInfo::getLdStOffsetOp(MI1).getImm()
             : AArch64InstrInfo::getLdStOffsetOp(MI1).getImm() * StoreSize1;

  const MachineInstr &MI = (Off0 < Off1) ? MI0 : MI1;
  int Multiples = AArch64InstrInfo::isPairedLdSt(MI) ? 2 : 1;
  int StoreSize = AArch64InstrInfo::getMemScale(MI) * Multiples;

  return llabs(Off0 - Off1) < StoreSize;
}
```
**EN:** This block continues the file's main scheduling models logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调度模型主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 60-70: Function AArch64PostRASchedStrategy::tryCandidate
```cpp

bool AArch64PostRASchedStrategy::tryCandidate(SchedCandidate &Cand,
                                              SchedCandidate &TryCand) {
  bool OriginalResult = PostGenericScheduler::tryCandidate(Cand, TryCand);

  if (Cand.isValid()) {
    MachineInstr *Instr0 = TryCand.SU->getInstr();
    MachineInstr *Instr1 = Cand.SU->getInstr();

    if (!needReorderStoreMI(Instr0) || !needReorderStoreMI(Instr1))
      return OriginalResult;
```
**EN:** This block implements AArch64PostRASchedStrategy::tryCandidate, advancing the file's scheduling models flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64PostRASchedStrategy::tryCandidate，通过 AArch64 专用的决策与数据处理推进本文件的调度模型流程。
### Lines 71-82: Core AArch64 backend logic
```cpp

    int64_t Off0, Off1;
    // With the same base address and non-overlapping writes.
    if (!mayOverlapWrite(*Instr0, *Instr1, Off0, Off1)) {
      TryCand.Reason = NodeOrder;
      // Order them by ascending offsets.
      return Off0 < Off1;
    }
  }

  return OriginalResult;
}
```
**EN:** This block continues the file's main scheduling models logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调度模型主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Per-core scheduling itineraries and latency models **CN:** 面向具体内核的调度行程与时延模型
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64MachineScheduler.h, AArch64InstrInfo.h, AArch64Subtarget.h, MCTargetDesc/AArch64MCTargetDesc.h **CN:** 目标本地依赖：AArch64MachineScheduler.h, AArch64InstrInfo.h, AArch64Subtarget.h, MCTargetDesc/AArch64MCTargetDesc.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for scheduling models. **CN:** 与周边负责调度模型的 AArch64 后端组件紧密协作。
