# ARMSLSHardening.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSLSHardening.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a pass to insert code to mitigate against side channel vulnerabilities that may happen under straight line miss-speculation.
- 用途 (CN): 实现 ARM 后端中的 `ARMSLSHardening`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- ARMSLSHardening.cpp - Harden Straight Line Missspeculation ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass to insert code to mitigate against side channel
// vulnerabilities that may happen under straight line miss-speculation.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-25
```cpp
#include "ARM.h"
#include "ARMInstrInfo.h"
#include "ARMSubtarget.h"
#include "llvm/CodeGen/IndirectThunks.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/DebugLoc.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 27-27
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 29-29
```cpp
#define DEBUG_TYPE "arm-sls-hardening"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 31-31
```cpp
#define ARM_SLS_HARDENING_NAME "ARM sls hardening pass"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 33-33
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 35-38
```cpp
class ARMSLSHardening : public MachineFunctionPass {
public:
  const TargetInstrInfo *TII;
  const ARMSubtarget *ST;
```
- EN: Declares `ARMSLSHardening`, packaging target-specific state and APIs around `ARMSLSHardening`.
- CN: 这里声明 `ARMSLSHardening`，把与 `ARMSLSHardening` 相关的目标特定状态和 API 组织在一起。

### Lines 40-40
```cpp
  static char ID;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 42-44
```cpp
  ARMSLSHardening() : MachineFunctionPass(ID) {
    initializeARMSLSHardeningPass(*PassRegistry::getPassRegistry());
  }
```
- EN: Implements `PassRegistry::getPassRegistry`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `PassRegistry::getPassRegistry`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 46-46
```cpp
  bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 48-48
```cpp
  StringRef getPassName() const override { return ARM_SLS_HARDENING_NAME; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-53
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-61
```cpp
private:
  bool hardenReturnsAndBRs(MachineBasicBlock &MBB) const;
  bool hardenIndirectCalls(MachineBasicBlock &MBB) const;
  MachineBasicBlock &
  ConvertIndirectCallToIndirectJump(MachineBasicBlock &MBB,
                                    MachineBasicBlock::iterator) const;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 63-63
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 65-65
```cpp
char ARMSLSHardening::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 67-68
```cpp
INITIALIZE_PASS(ARMSLSHardening, "arm-sls-hardening",
                ARM_SLS_HARDENING_NAME, false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 70-84
```cpp
static void insertSpeculationBarrier(const ARMSubtarget *ST,
                                     MachineBasicBlock &MBB,
                                     MachineBasicBlock::iterator MBBI,
                                     DebugLoc DL,
                                     bool AlwaysUseISBDSB = false) {
  assert(MBBI != MBB.begin() &&
         "Must not insert SpeculationBarrierEndBB as only instruction in MBB.");
  assert(std::prev(MBBI)->isBarrier() &&
         "SpeculationBarrierEndBB must only follow unconditional control flow "
         "instructions.");
  assert(std::prev(MBBI)->isTerminator() &&
         "SpeculationBarrierEndBB must only follow terminators.");
  const TargetInstrInfo *TII = ST->getInstrInfo();
  assert(ST->hasDataBarrier() || ST->hasSB());
  bool ProduceSB = ST->hasSB() && !AlwaysUseISBDSB;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 85-92
```cpp
  unsigned BarrierOpc =
      ProduceSB ? (ST->isThumb() ? ARM::t2SpeculationBarrierSBEndBB
                                 : ARM::SpeculationBarrierSBEndBB)
                : (ST->isThumb() ? ARM::t2SpeculationBarrierISBDSBEndBB
                                 : ARM::SpeculationBarrierISBDSBEndBB);
  if (MBBI == MBB.end() || !isSpeculationBarrierEndBBOpcode(MBBI->getOpcode()))
    BuildMI(MBB, MBBI, DL, TII->get(BarrierOpc));
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 94-96
```cpp
bool ARMSLSHardening::runOnMachineFunction(MachineFunction &MF) {
  ST = &MF.getSubtarget<ARMSubtarget>();
  TII = MF.getSubtarget().getInstrInfo();
```
- EN: Implements `ARMSLSHardening::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSLSHardening::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 98-102
```cpp
  bool Modified = false;
  for (auto &MBB : MF) {
    Modified |= hardenReturnsAndBRs(MBB);
    Modified |= hardenIndirectCalls(MBB);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 104-105
```cpp
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 107-121
```cpp
bool ARMSLSHardening::hardenReturnsAndBRs(MachineBasicBlock &MBB) const {
  if (!ST->hardenSlsRetBr())
    return false;
  assert(!ST->isThumb1Only());
  bool Modified = false;
  MachineBasicBlock::iterator MBBI = MBB.getFirstTerminator(), E = MBB.end();
  MachineBasicBlock::iterator NextMBBI;
  for (; MBBI != E; MBBI = NextMBBI) {
    MachineInstr &MI = *MBBI;
    NextMBBI = std::next(MBBI);
    if (isIndirectControlFlowNotComingBack(MI)) {
      assert(MI.isTerminator());
      assert(!TII->isPredicated(MI));
      insertSpeculationBarrier(ST, MBB, std::next(MBBI), MI.getDebugLoc());
      Modified = true;
```
- EN: Implements `ARMSLSHardening::hardenReturnsAndBRs`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSLSHardening::hardenReturnsAndBRs`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 122-125
```cpp
    }
  }
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 127-127
```cpp
static const char SLSBLRNamePrefix[] = "__llvm_slsblr_thunk_";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 129-143
```cpp
static const struct ThunkNameRegMode {
  const char* Name;
  Register Reg;
  bool isThumb;
} SLSBLRThunks[] = {
    {"__llvm_slsblr_thunk_arm_r0", ARM::R0, false},
    {"__llvm_slsblr_thunk_arm_r1", ARM::R1, false},
    {"__llvm_slsblr_thunk_arm_r2", ARM::R2, false},
    {"__llvm_slsblr_thunk_arm_r3", ARM::R3, false},
    {"__llvm_slsblr_thunk_arm_r4", ARM::R4, false},
    {"__llvm_slsblr_thunk_arm_r5", ARM::R5, false},
    {"__llvm_slsblr_thunk_arm_r6", ARM::R6, false},
    {"__llvm_slsblr_thunk_arm_r7", ARM::R7, false},
    {"__llvm_slsblr_thunk_arm_r8", ARM::R8, false},
    {"__llvm_slsblr_thunk_arm_r9", ARM::R9, false},
```
- EN: Declares `ThunkNameRegMode`, packaging target-specific state and APIs around `ARMSLSHardening`.
- CN: 这里声明 `ThunkNameRegMode`，把与 `ARMSLSHardening` 相关的目标特定状态和 API 组织在一起。

### Lines 144-158
```cpp
    {"__llvm_slsblr_thunk_arm_r10", ARM::R10, false},
    {"__llvm_slsblr_thunk_arm_r11", ARM::R11, false},
    {"__llvm_slsblr_thunk_arm_sp", ARM::SP, false},
    {"__llvm_slsblr_thunk_arm_pc", ARM::PC, false},
    {"__llvm_slsblr_thunk_thumb_r0", ARM::R0, true},
    {"__llvm_slsblr_thunk_thumb_r1", ARM::R1, true},
    {"__llvm_slsblr_thunk_thumb_r2", ARM::R2, true},
    {"__llvm_slsblr_thunk_thumb_r3", ARM::R3, true},
    {"__llvm_slsblr_thunk_thumb_r4", ARM::R4, true},
    {"__llvm_slsblr_thunk_thumb_r5", ARM::R5, true},
    {"__llvm_slsblr_thunk_thumb_r6", ARM::R6, true},
    {"__llvm_slsblr_thunk_thumb_r7", ARM::R7, true},
    {"__llvm_slsblr_thunk_thumb_r8", ARM::R8, true},
    {"__llvm_slsblr_thunk_thumb_r9", ARM::R9, true},
    {"__llvm_slsblr_thunk_thumb_r10", ARM::R10, true},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 159-162
```cpp
    {"__llvm_slsblr_thunk_thumb_r11", ARM::R11, true},
    {"__llvm_slsblr_thunk_thumb_sp", ARM::SP, true},
    {"__llvm_slsblr_thunk_thumb_pc", ARM::PC, true},
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-166
```cpp
// An enum for tracking whether Arm and Thumb thunks have been inserted into the
// current module so far.
enum ArmInsertedThunks { NoThunk = 0, ArmThunk = 1, ThumbThunk = 2 };
```
- EN: Defines enumeration `for` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `for`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 168-171
```cpp
inline ArmInsertedThunks &operator|=(ArmInsertedThunks &X,
                                     ArmInsertedThunks Y) {
  return X = static_cast<ArmInsertedThunks>(X | Y);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 173-183
```cpp
namespace {
struct SLSBLRThunkInserter
    : ThunkInserter<SLSBLRThunkInserter, ArmInsertedThunks> {
  const char *getThunkPrefix() { return SLSBLRNamePrefix; }
  bool mayUseThunk(const MachineFunction &MF) {
    ComdatThunks &= !MF.getSubtarget<ARMSubtarget>().hardenSlsNoComdat();
    return MF.getSubtarget<ARMSubtarget>().hardenSlsBlr();
  }
  ArmInsertedThunks insertThunks(MachineModuleInfo &MMI, MachineFunction &MF,
                                 ArmInsertedThunks InsertedThunks);
  void populateThunk(MachineFunction &MF);
```
- EN: Declares `SLSBLRThunkInserter`, packaging target-specific state and APIs around `ARMSLSHardening`.
- CN: 这里声明 `SLSBLRThunkInserter`，把与 `ARMSLSHardening` 相关的目标特定状态和 API 组织在一起。

### Lines 185-188
```cpp
private:
  bool ComdatThunks = true;
};
} // namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 190-204
```cpp
ArmInsertedThunks
SLSBLRThunkInserter::insertThunks(MachineModuleInfo &MMI, MachineFunction &MF,
                                  ArmInsertedThunks InsertedThunks) {
  if ((InsertedThunks & ArmThunk &&
       !MF.getSubtarget<ARMSubtarget>().isThumb()) ||
      (InsertedThunks & ThumbThunk &&
       MF.getSubtarget<ARMSubtarget>().isThumb()))
    return NoThunk;
  // FIXME: It probably would be possible to filter which thunks to produce
  // based on which registers are actually used in indirect calls in this
  // function. But would that be a worthwhile optimization?
  const ARMSubtarget *ST = &MF.getSubtarget<ARMSubtarget>();
  for (auto T : SLSBLRThunks)
    if (ST->isThumb() == T.isThumb)
      createThunkFunction(MMI, T.Name, ComdatThunks,
```
- EN: Implements `SLSBLRThunkInserter::insertThunks`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `SLSBLRThunkInserter::insertThunks`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 205-207
```cpp
                          T.isThumb ? "+thumb-mode" : "");
  return ST->isThumb() ? ThumbThunk : ArmThunk;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-219
```cpp
void SLSBLRThunkInserter::populateThunk(MachineFunction &MF) {
  assert(MF.getFunction().hasComdat() == ComdatThunks &&
         "ComdatThunks value changed since MF creation");
  // FIXME: How to better communicate Register number, rather than through
  // name and lookup table?
  assert(MF.getName().starts_with(getThunkPrefix()));
  auto ThunkIt = llvm::find_if(
      SLSBLRThunks, [&MF](auto T) { return T.Name == MF.getName(); });
  assert(ThunkIt != std::end(SLSBLRThunks));
  Register ThunkReg = ThunkIt->Reg;
  bool isThumb = ThunkIt->isThumb;
```
- EN: Implements `SLSBLRThunkInserter::populateThunk`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `SLSBLRThunkInserter::populateThunk`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 221-223
```cpp
  const TargetInstrInfo *TII = MF.getSubtarget<ARMSubtarget>().getInstrInfo();
  MachineBasicBlock *Entry = &MF.front();
  Entry->clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 225-236
```cpp
  //  These thunks need to consist of the following instructions:
  //  __llvm_slsblr_thunk_(arm/thumb)_rN:
  //      bx  rN
  //      barrierInsts
  Entry->addLiveIn(ThunkReg);
  if (isThumb)
    BuildMI(Entry, DebugLoc(), TII->get(ARM::tBX))
        .addReg(ThunkReg)
        .add(predOps(ARMCC::AL));
  else
    BuildMI(Entry, DebugLoc(), TII->get(ARM::BX))
        .addReg(ThunkReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 238-244
```cpp
  // Make sure the thunks do not make use of the SB extension in case there is
  // a function somewhere that will call to it that for some reason disabled
  // the SB extension locally on that function, even though it's enabled for
  // the module otherwise. Therefore set AlwaysUseISBSDB to true.
  insertSpeculationBarrier(&MF.getSubtarget<ARMSubtarget>(), *Entry,
                           Entry->end(), DebugLoc(), true /*AlwaysUseISBDSB*/);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 246-260
```cpp
MachineBasicBlock &ARMSLSHardening::ConvertIndirectCallToIndirectJump(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI) const {
  // Transform an indirect call to an indirect jump as follows:
  // Before:
  //   |-----------------------------|
  //   |      ...                    |
  //   |  instI                      |
  //   |  BLX rN                     |
  //   |  instJ                      |
  //   |      ...                    |
  //   |-----------------------------|
  //
  // After:
  //   |----------   -------------------------|
  //   |      ...                             |
```
- EN: Implements `ARMSLSHardening::ConvertIndirectCallToIndirectJump`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSLSHardening::ConvertIndirectCallToIndirectJump`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 261-275
```cpp
  //   |  instI                               |
  //   |  *call* __llvm_slsblr_thunk_mode_xN  |
  //   |  instJ                               |
  //   |      ...                             |
  //   |--------------------------------------|
  //
  //   __llvm_slsblr_thunk_mode_xN:
  //   |-----------------------------|
  //   |  BX rN                      |
  //   |  barrierInsts               |
  //   |-----------------------------|
  //
  // The __llvm_slsblr_thunk_mode_xN thunks are created by the
  // SLSBLRThunkInserter.
  // This function merely needs to transform an indirect call to a direct call
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 276-290
```cpp
  // to __llvm_slsblr_thunk_xN.
  MachineInstr &IndirectCall = *MBBI;
  assert(isIndirectCall(IndirectCall) && !IndirectCall.isReturn());
  int RegOpIdxOnIndirectCall = -1;
  bool isThumb;
  switch (IndirectCall.getOpcode()) {
  case ARM::BLX:   // !isThumb2
  case ARM::BLX_noip:   // !isThumb2
    isThumb = false;
    RegOpIdxOnIndirectCall = 0;
    break;
  case ARM::tBLXr:      // isThumb2
  case ARM::tBLXr_noip: // isThumb2
    isThumb = true;
    RegOpIdxOnIndirectCall = 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 291-294
```cpp
    break;
  default:
    llvm_unreachable("unhandled Indirect Call");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 296-304
```cpp
  Register Reg = IndirectCall.getOperand(RegOpIdxOnIndirectCall).getReg();
  // Since linkers are allowed to clobber R12 on function calls, the above
  // mitigation only works if the original indirect call instruction was not
  // using R12. Code generation before must make sure that no indirect call
  // using R12 was produced if the mitigation is enabled.
  // Also, the transformation is incorrect if the indirect call uses LR, so
  // also have to avoid that.
  assert(Reg != ARM::R12 && Reg != ARM::LR);
  bool RegIsKilled = IndirectCall.getOperand(RegOpIdxOnIndirectCall).isKill();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 306-306
```cpp
  DebugLoc DL = IndirectCall.getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 308-314
```cpp
  MachineFunction &MF = *MBBI->getMF();
  auto ThunkIt = llvm::find_if(SLSBLRThunks, [Reg, isThumb](auto T) {
    return T.Reg == Reg && T.isThumb == isThumb;
  });
  assert(ThunkIt != std::end(SLSBLRThunks));
  Module *M = MF.getFunction().getParent();
  const GlobalValue *GV = cast<GlobalValue>(M->getNamedValue(ThunkIt->Name));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 316-321
```cpp
  MachineInstr *BL =
      isThumb ? BuildMI(MBB, MBBI, DL, TII->get(ARM::tBL))
                    .addImm(IndirectCall.getOperand(0).getImm())
                    .addReg(IndirectCall.getOperand(1).getReg())
                    .addGlobalAddress(GV)
              : BuildMI(MBB, MBBI, DL, TII->get(ARM::BL)).addGlobalAddress(GV);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 323-337
```cpp
  // Now copy the implicit operands from IndirectCall to BL and copy other
  // necessary info.
  // However, both IndirectCall and BL instructions implicitly use SP and
  // implicitly define LR. Blindly copying implicit operands would result in SP
  // and LR operands to be present multiple times. While this may not be too
  // much of an issue, let's avoid that for cleanliness, by removing those
  // implicit operands from the BL created above before we copy over all
  // implicit operands from the IndirectCall.
  int ImpLROpIdx = -1;
  int ImpSPOpIdx = -1;
  for (unsigned OpIdx = BL->getNumExplicitOperands();
       OpIdx < BL->getNumOperands(); OpIdx++) {
    MachineOperand Op = BL->getOperand(OpIdx);
    if (!Op.isReg())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 338-352
```cpp
    if (Op.getReg() == ARM::LR && Op.isDef())
      ImpLROpIdx = OpIdx;
    if (Op.getReg() == ARM::SP && !Op.isDef())
      ImpSPOpIdx = OpIdx;
  }
  assert(ImpLROpIdx != -1);
  assert(ImpSPOpIdx != -1);
  int FirstOpIdxToRemove = std::max(ImpLROpIdx, ImpSPOpIdx);
  int SecondOpIdxToRemove = std::min(ImpLROpIdx, ImpSPOpIdx);
  BL->removeOperand(FirstOpIdxToRemove);
  BL->removeOperand(SecondOpIdxToRemove);
  // Now copy over the implicit operands from the original IndirectCall
  BL->copyImplicitOps(MF, IndirectCall);
  MF.moveAdditionalCallInfo(&IndirectCall, BL);
  // Also add the register called in the IndirectCall as being used in the
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 353-359
```cpp
  // called thunk.
  BL->addOperand(MachineOperand::CreateReg(Reg, false /*isDef*/, true /*isImp*/,
                                           RegIsKilled /*isKill*/));
  // Remove IndirectCallinstruction
  MBB.erase(MBBI);
  return MBB;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 361-375
```cpp
bool ARMSLSHardening::hardenIndirectCalls(MachineBasicBlock &MBB) const {
  if (!ST->hardenSlsBlr())
    return false;
  bool Modified = false;
  MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  MachineBasicBlock::iterator NextMBBI;
  for (; MBBI != E; MBBI = NextMBBI) {
    MachineInstr &MI = *MBBI;
    NextMBBI = std::next(MBBI);
    // Tail calls are both indirect calls and "returns".
    // They are also indirect jumps, so should be handled by sls-harden-retbr,
    // rather than sls-harden-blr.
    if (isIndirectCall(MI) && !MI.isReturn()) {
      ConvertIndirectCallToIndirectJump(MBB, MBBI);
      Modified = true;
```
- EN: Implements `ARMSLSHardening::hardenIndirectCalls`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSLSHardening::hardenIndirectCalls`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 376-379
```cpp
    }
  }
  return Modified;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 383-385
```cpp
FunctionPass *llvm::createARMSLSHardeningPass() {
  return new ARMSLSHardening();
}
```
- EN: Implements `llvm::createARMSLSHardeningPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMSLSHardeningPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 387-390
```cpp
namespace {
class ARMIndirectThunks : public ThunkInserterPass<SLSBLRThunkInserter> {
public:
  static char ID;
```
- EN: Declares `ARMIndirectThunks`, packaging target-specific state and APIs around `ARMSLSHardening`.
- CN: 这里声明 `ARMIndirectThunks`，把与 `ARMSLSHardening` 相关的目标特定状态和 API 组织在一起。

### Lines 392-392
```cpp
  ARMIndirectThunks() : ThunkInserterPass(ID) {}
```
- EN: Implements `ARMIndirectThunks`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMIndirectThunks`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 394-396
```cpp
  StringRef getPassName() const override { return "ARM Indirect Thunks"; }
};
} // end anonymous namespace
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 398-398
```cpp
char ARMIndirectThunks::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 400-402
```cpp
FunctionPass *llvm::createARMIndirectThunks() {
  return new ARMIndirectThunks();
}
```
- EN: Implements `llvm::createARMIndirectThunks`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMIndirectThunks`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMInstrInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMInstrInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/IndirectThunks.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/IR/DebugLoc.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/IndirectThunks.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/IR/DebugLoc.h`。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
