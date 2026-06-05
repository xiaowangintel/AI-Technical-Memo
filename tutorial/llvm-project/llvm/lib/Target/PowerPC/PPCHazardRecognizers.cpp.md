# PPCHazardRecognizers.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCHazardRecognizers.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCHazardRecognizers.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCHazardRecognizers.cpp - PowerPC Hazard Recognizer Impls --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file implements hazard recognizers for scheduling on PowerPC processors.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file implements hazard recognizers for scheduling on PowerPC processors.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file implements hazard recognizers for scheduling on PowerPC processors.”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "PPCHazardRecognizers.h"
#include "PPCInstrInfo.h"
#include "PPCSubtarget.h"
#include "llvm/CodeGen/ScheduleDAG.h"
#include "llvm/Support/Debug.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 18-55

```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "pre-RA-sched"

bool PPCDispatchGroupSBHazardRecognizer::isLoadAfterStore(SUnit *SU) {
  // FIXME: Move this.
  if (isBCTRAfterSet(SU))
    return true;

  const MCInstrDesc *MCID = DAG->getInstrDesc(SU);
  if (!MCID)
    return false;

  if (!MCID->mayLoad())
    return false;

  // SU is a load; for any predecessors in this dispatch group, that are stores,
  // and with which we have an ordering dependency, return true.
  for (unsigned i = 0, ie = (unsigned) SU->Preds.size(); i != ie; ++i) {
    const MCInstrDesc *PredMCID = DAG->getInstrDesc(SU->Preds[i].getSUnit());
    if (!PredMCID || !PredMCID->mayStore())
      continue;

    if (!SU->Preds[i].isNormalMemory() && !SU->Preds[i].isBarrier())
      continue;

    for (unsigned j = 0, je = CurGroup.size(); j != je; ++j)
      if (SU->Preds[i].getSUnit() == CurGroup[j])
        return true;
  }

  return false;
}

bool PPCDispatchGroupSBHazardRecognizer::isBCTRAfterSet(SUnit *SU) {
  const MCInstrDesc *MCID = DAG->getInstrDesc(SU);
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 56-80

```cpp
  if (!MCID)
    return false;

  if (!MCID->isBranch())
    return false;

  // SU is a branch; for any predecessors in this dispatch group, with which we
  // have a data dependence and set the counter register, return true.
  for (unsigned i = 0, ie = (unsigned) SU->Preds.size(); i != ie; ++i) {
    const MCInstrDesc *PredMCID = DAG->getInstrDesc(SU->Preds[i].getSUnit());
    if (!PredMCID || PredMCID->getSchedClass() != PPC::Sched::IIC_SprMTSPR)
      continue;

    if (SU->Preds[i].isCtrl())
      continue;

    for (unsigned j = 0, je = CurGroup.size(); j != je; ++j)
      if (SU->Preds[i].getSUnit() == CurGroup[j])
        return true;
  }

  return false;
}

// FIXME: Remove this when we don't need this:
```
- **EN**: Implements helper routine(s) `isBranch`, `size`, `getInstrDesc` for this portion of the PowerPC backend backend implementation logic. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isBranch`, `size`, `getInstrDesc`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 81-118

```cpp
namespace llvm {
namespace PPC {
extern int32_t getNonRecordFormOpcode(uint32_t);
}
} // namespace llvm

// FIXME: A lot of code in PPCDispatchGroupSBHazardRecognizer is P7 specific.

bool PPCDispatchGroupSBHazardRecognizer::mustComeFirst(const MCInstrDesc *MCID,
                                                       unsigned &NSlots) const {
  // FIXME: Indirectly, this information is contained in the itinerary, and
  // we should derive it from there instead of separately specifying it
  // here.
  unsigned IIC = MCID->getSchedClass();
  switch (IIC) {
  default:
    NSlots = 1;
    break;
  case PPC::Sched::IIC_IntDivW:
  case PPC::Sched::IIC_IntDivD:
  case PPC::Sched::IIC_LdStLoadUpd:
  case PPC::Sched::IIC_LdStLDU:
  case PPC::Sched::IIC_LdStLFDU:
  case PPC::Sched::IIC_LdStLFDUX:
  case PPC::Sched::IIC_LdStLHA:
  case PPC::Sched::IIC_LdStLHAU:
  case PPC::Sched::IIC_LdStLWA:
  case PPC::Sched::IIC_LdStSTU:
  case PPC::Sched::IIC_LdStSTFDU:
    NSlots = 2;
    break;
  case PPC::Sched::IIC_LdStLoadUpdX:
  case PPC::Sched::IIC_LdStLDUX:
  case PPC::Sched::IIC_LdStLHAUX:
  case PPC::Sched::IIC_LdStLWARX:
  case PPC::Sched::IIC_LdStLDARX:
  case PPC::Sched::IIC_LdStSTUX:
  case PPC::Sched::IIC_LdStSTDCX:
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. The logic interacts with LLVM's MC layer.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 119-156

```cpp
  case PPC::Sched::IIC_LdStSTWCX:
  case PPC::Sched::IIC_BrMCRX: // mtcr
  // FIXME: Add sync/isync (here and in the itinerary).
    NSlots = 4;
    break;
  }

  // FIXME: record-form instructions need a different itinerary class.
  if (NSlots == 1 && PPC::getNonRecordFormOpcode(MCID->getOpcode()) != -1)
    NSlots = 2;

  switch (IIC) {
  default:
    // All multi-slot instructions must come first.
    return NSlots > 1;
  case PPC::Sched::IIC_BrCR: // cr logicals
  case PPC::Sched::IIC_SprMFCR:
  case PPC::Sched::IIC_SprMFCRF:
  case PPC::Sched::IIC_SprMTSPR:
    return true;
  }
}

ScheduleHazardRecognizer::HazardType
PPCDispatchGroupSBHazardRecognizer::getHazardType(SUnit *SU, int Stalls) {
  if (Stalls == 0 && isLoadAfterStore(SU))
    return NoopHazard;

  return ScoreboardHazardRecognizer::getHazardType(SU, Stalls);
}

bool PPCDispatchGroupSBHazardRecognizer::ShouldPreferAnother(SUnit *SU) const {
  const MCInstrDesc *MCID = DAG->getInstrDesc(SU);
  unsigned NSlots;
  if (MCID && mustComeFirst(MCID, NSlots) && CurSlots)
    return true;

  return ScoreboardHazardRecognizer::ShouldPreferAnother(SU);
```
- **EN**: Implements helper routine(s) `isync`, `getNonRecordFormOpcode`, `getOpcode` for this portion of the PowerPC backend backend implementation logic. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isync`, `getNonRecordFormOpcode`, `getOpcode`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 157-194

```cpp
}

unsigned PPCDispatchGroupSBHazardRecognizer::PreEmitNoops(SUnit *SU) {
  // We only need to fill out a maximum of 5 slots here: The 6th slot could
  // only be a second branch, and otherwise the next instruction will start a
  // new group.
  if (isLoadAfterStore(SU) && CurSlots < 6) {
    unsigned Directive =
        DAG->MF.getSubtarget<PPCSubtarget>().getCPUDirective();
    // If we're using a special group-terminating nop, then we need only one.
    // FIXME: the same for P9 as previous gen until POWER9 scheduling is ready
    if (Directive == PPC::DIR_PWR6 || Directive == PPC::DIR_PWR7 ||
        Directive == PPC::DIR_PWR8 || Directive == PPC::DIR_PWR9)
      return 1;

    return 5 - CurSlots;
  }

  return ScoreboardHazardRecognizer::PreEmitNoops(SU);
}

void PPCDispatchGroupSBHazardRecognizer::EmitInstruction(SUnit *SU) {
  const MCInstrDesc *MCID = DAG->getInstrDesc(SU);
  if (MCID) {
    if (CurSlots == 5 || (MCID->isBranch() && CurBranches == 1)) {
      CurGroup.clear();
      CurSlots = CurBranches = 0;
    } else {
      LLVM_DEBUG(dbgs() << "**** Adding to dispatch group: ");
      LLVM_DEBUG(DAG->dumpNode(*SU));

      unsigned NSlots;
      bool MustBeFirst = mustComeFirst(MCID, NSlots);

      // If this instruction must come first, but does not, then it starts a
      // new group.
      if (MustBeFirst && CurSlots) {
        CurSlots = CurBranches = 0;
```
- **EN**: Implements helper routine(s) `PreEmitNoops`, `isLoadAfterStore`, `getCPUDirective` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `PreEmitNoops`, `isLoadAfterStore`, `getCPUDirective`。 子目标特性裁剪会影响这里的行为。

### Lines 195-232

```cpp
        CurGroup.clear();
      }

      CurSlots += NSlots;
      CurGroup.push_back(SU);

      if (MCID->isBranch())
        ++CurBranches;
    }
  }

  return ScoreboardHazardRecognizer::EmitInstruction(SU);
}

void PPCDispatchGroupSBHazardRecognizer::AdvanceCycle() {
  return ScoreboardHazardRecognizer::AdvanceCycle();
}

void PPCDispatchGroupSBHazardRecognizer::RecedeCycle() {
  llvm_unreachable("Bottom-up scheduling not supported");
}

void PPCDispatchGroupSBHazardRecognizer::Reset() {
  CurGroup.clear();
  CurSlots = CurBranches = 0;
  return ScoreboardHazardRecognizer::Reset();
}

void PPCDispatchGroupSBHazardRecognizer::EmitNoop() {
  unsigned Directive =
      DAG->MF.getSubtarget<PPCSubtarget>().getCPUDirective();
  // If the group has now filled all of its slots, or if we're using a special
  // group-terminating nop, the group is complete.
  // FIXME: the same for P9 as previous gen until POWER9 scheduling is ready
  if (Directive == PPC::DIR_PWR6 || Directive == PPC::DIR_PWR7 ||
      Directive == PPC::DIR_PWR8 || Directive == PPC::DIR_PWR9 ||
      CurSlots == 6) {
    CurGroup.clear();
```
- **EN**: Implements helper routine(s) `clear`, `push_back`, `isBranch` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `clear`, `push_back`, `isBranch`。 子目标特性裁剪会影响这里的行为。

### Lines 233-239

```cpp
    CurSlots = CurBranches = 0;
  } else {
    CurGroup.push_back(nullptr);
    ++CurSlots;
  }
}
```
- **EN**: Implements helper routine(s) `push_back` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `push_back`。

### Lines 240-277

```cpp
//===----------------------------------------------------------------------===//
// PowerPC 970 Hazard Recognizer
//
// This models the dispatch group formation of the PPC970 processor.  Dispatch
// groups are bundles of up to five instructions that can contain various mixes
// of instructions.  The PPC970 can dispatch a peak of 4 non-branch and one
// branch instruction per-cycle.
//
// There are a number of restrictions to dispatch group formation: some
// instructions can only be issued in the first slot of a dispatch group, & some
// instructions fill an entire dispatch group.  Additionally, only branches can
// issue in the 5th (last) slot.
//
// Finally, there are a number of "structural" hazards on the PPC970.  These
// conditions cause large performance penalties due to misprediction, recovery,
// and replay logic that has to happen.  These cases include setting a CTR and
// branching through it in the same dispatch group, and storing to an address,
// then loading from the same address within a dispatch group.  To avoid these
// conditions, we insert no-op instructions when appropriate.
//
// FIXME: This is missing some significant cases:
//   1. Modeling of microcoded instructions.
//   2. Handling of serialized operations.
//   3. Handling of the esoteric cases in "Resource-based Instruction Grouping".
//

PPCHazardRecognizer970::PPCHazardRecognizer970(const ScheduleDAG &DAG)
    : DAG(DAG) {
  EndDispatchGroup();
}

void PPCHazardRecognizer970::EndDispatchGroup() {
  LLVM_DEBUG(errs() << "=== Start of dispatch group\n");
  NumIssued = 0;

  // Structural hazard info.
  HasCTRSet = false;
  NumStores = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC 970 Hazard Recognizer". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC 970 Hazard Recognizer”。 这一段包含调度或处理器模型元数据。

### Lines 278-315

```cpp
}


PPCII::PPC970_Unit
PPCHazardRecognizer970::GetInstrType(unsigned Opcode,
                                     bool &isFirst, bool &isSingle,
                                     bool &isCracked,
                                     bool &isLoad, bool &isStore) {
  const MCInstrDesc &MCID = DAG.TII->get(Opcode);

  isLoad  = MCID.mayLoad();
  isStore = MCID.mayStore();

  uint64_t TSFlags = MCID.TSFlags;

  isFirst   = TSFlags & PPCII::PPC970_First;
  isSingle  = TSFlags & PPCII::PPC970_Single;
  isCracked = TSFlags & PPCII::PPC970_Cracked;
  return (PPCII::PPC970_Unit)(TSFlags & PPCII::PPC970_Mask);
}

/// isLoadOfStoredAddress - If we have a load from the previously stored pointer
/// as indicated by StorePtr1/StorePtr2/StoreSize, return true.
bool PPCHazardRecognizer970::
isLoadOfStoredAddress(uint64_t LoadSize, int64_t LoadOffset,
  const Value *LoadValue) const {
  for (unsigned i = 0, e = NumStores; i != e; ++i) {
    // Handle exact and commuted addresses.
    if (LoadValue == StoreValue[i] && LoadOffset == StoreOffset[i])
      return true;

    // Okay, we don't have an exact match, if this is an indexed offset, see if
    // we have overlap (which happens during fp->int conversion for example).
    if (StoreValue[i] == LoadValue) {
      // Okay the base pointers match, so we have [c1+r] vs [c2+r].  Check
      // to see if the load and store actually overlap.
      if (StoreOffset[i] < LoadOffset) {
        if (int64_t(StoreOffset[i]+StoreSize[i]) > LoadOffset) return true;
```
- **EN**: Implements helper routine(s) `GetInstrType`, `get`, `mayLoad` for this portion of the PowerPC backend backend implementation logic. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `GetInstrType`, `get`, `mayLoad`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 316-353

```cpp
      } else {
        if (int64_t(LoadOffset+LoadSize) > StoreOffset[i]) return true;
      }
    }
  }
  return false;
}

/// getHazardType - We return hazard for any non-branch instruction that would
/// terminate the dispatch group.  We turn NoopHazard for any
/// instructions that wouldn't terminate the dispatch group that would cause a
/// pipeline flush.
ScheduleHazardRecognizer::HazardType PPCHazardRecognizer970::
getHazardType(SUnit *SU, int Stalls) {
  assert(Stalls == 0 && "PPC hazards don't support scoreboard lookahead");

  MachineInstr *MI = SU->getInstr();

  if (MI->isDebugInstr())
    return NoHazard;

  unsigned Opcode = MI->getOpcode();
  bool isFirst, isSingle, isCracked, isLoad, isStore;
  PPCII::PPC970_Unit InstrType =
    GetInstrType(Opcode, isFirst, isSingle, isCracked,
                 isLoad, isStore);
  if (InstrType == PPCII::PPC970_Pseudo) return NoHazard;

  // We can only issue a PPC970_First/PPC970_Single instruction (such as
  // crand/mtspr/etc) if this is the first cycle of the dispatch group.
  if (NumIssued != 0 && (isFirst || isSingle))
    return Hazard;

  // If this instruction is cracked into two ops by the decoder, we know that
  // it is not a branch and that it cannot issue if 3 other instructions are
  // already in the dispatch group.
  if (isCracked && NumIssued > 2)
    return Hazard;
```
- **EN**: Implements helper routine(s) `int64_t`, `getHazardType`, `getInstr` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `int64_t`, `getHazardType`, `getInstr`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 354-391

```cpp

  switch (InstrType) {
  default: llvm_unreachable("Unknown instruction type!");
  case PPCII::PPC970_FXU:
  case PPCII::PPC970_LSU:
  case PPCII::PPC970_FPU:
  case PPCII::PPC970_VALU:
  case PPCII::PPC970_VPERM:
    // We can only issue a branch as the last instruction in a group.
    if (NumIssued == 4) return Hazard;
    break;
  case PPCII::PPC970_CRU:
    // We can only issue a CR instruction in the first two slots.
    if (NumIssued >= 2) return Hazard;
    break;
  case PPCII::PPC970_BRU:
    break;
  }

  // Do not allow MTCTR and BCTRL to be in the same dispatch group.
  if (HasCTRSet && Opcode == PPC::BCTRL)
    return NoopHazard;

  // If this is a load following a store, make sure it's not to the same or
  // overlapping address.
  if (isLoad && NumStores && !MI->memoperands_empty()) {
    MachineMemOperand *MO = *MI->memoperands_begin();
    if (MO->getSize().hasValue() &&
        isLoadOfStoredAddress(MO->getSize().getValue(), MO->getOffset(),
                              MO->getValue()))
      return NoopHazard;
  }

  return NoHazard;
}

void PPCHazardRecognizer970::EmitInstruction(SUnit *SU) {
  MachineInstr *MI = SU->getInstr();
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `memoperands_empty`, `memoperands_begin` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `llvm_unreachable`, `memoperands_empty`, `memoperands_begin`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 392-429

```cpp

  if (MI->isDebugInstr())
    return;

  unsigned Opcode = MI->getOpcode();
  bool isFirst, isSingle, isCracked, isLoad, isStore;
  PPCII::PPC970_Unit InstrType =
    GetInstrType(Opcode, isFirst, isSingle, isCracked,
                 isLoad, isStore);
  if (InstrType == PPCII::PPC970_Pseudo) return;

  // Update structural hazard information.
  if (Opcode == PPC::MTCTR || Opcode == PPC::MTCTR8) HasCTRSet = true;

  // Track the address stored to.
  if (isStore && NumStores < 4 && !MI->memoperands_empty() &&
      (*MI->memoperands_begin())->getSize().hasValue()) {
    MachineMemOperand *MO = *MI->memoperands_begin();
    StoreSize[NumStores] = MO->getSize().getValue();
    StoreOffset[NumStores] = MO->getOffset();
    StoreValue[NumStores] = MO->getValue();
    ++NumStores;
  }

  if (InstrType == PPCII::PPC970_BRU || isSingle)
    NumIssued = 4;  // Terminate a d-group.
  ++NumIssued;

  // If this instruction is cracked into two ops by the decoder, remember that
  // we issued two pieces.
  if (isCracked)
    ++NumIssued;

  if (NumIssued == 5)
    EndDispatchGroup();
}

void PPCHazardRecognizer970::AdvanceCycle() {
```
- **EN**: Implements helper routine(s) `isDebugInstr`, `getOpcode`, `GetInstrType` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isDebugInstr`, `getOpcode`, `GetInstrType`。

### Lines 430-439

```cpp
  assert(NumIssued < 5 && "Illegal dispatch group!");
  ++NumIssued;
  if (NumIssued == 5)
    EndDispatchGroup();
}

void PPCHazardRecognizer970::Reset() {
  EndDispatchGroup();
}
```
- **EN**: Implements helper routine(s) `EndDispatchGroup`, `Reset` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `EndDispatchGroup`, `Reset`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCHazardRecognizers.h`
- `PPCInstrInfo.h`
- `PPCSubtarget.h`
- `llvm/CodeGen/ScheduleDAG.h`
- `llvm/Support/Debug.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
