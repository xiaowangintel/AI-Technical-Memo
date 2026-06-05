# ARMCallingConv.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMCallingConv.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the custom routines for the ARM Calling Convention that aren't done by tablegen, and includes the table generated implementations.
- 用途 (CN): 实现 ARM 后端中的 `ARMCallingConv`，重点处理调用约定规则与 ABI 降级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//=== ARMCallingConv.cpp - ARM Custom CC Routines ---------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the custom routines for the ARM Calling Convention that
// aren't done by tablegen, and includes the table generated implementations.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-17
```cpp
#include "ARMCallingConv.h"
#include "ARM.h"
#include "ARMSubtarget.h"
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 19-23
```cpp
// APCS f64 is in register pairs, possibly split to stack
static bool f64AssignAPCS(unsigned ValNo, MVT ValVT, MVT LocVT,
                          CCValAssign::LocInfo LocInfo,
                          CCState &State, bool CanFail) {
  static const MCPhysReg RegList[] = { ARM::R0, ARM::R1, ARM::R2, ARM::R3 };
```
- EN: Implements `f64AssignAPCS`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `f64AssignAPCS`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 25-31
```cpp
  // Try to get the first register.
  if (MCRegister Reg = State.AllocateReg(RegList))
    State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
  else {
    // For the 2nd half of a v2f64, do not fail.
    if (CanFail)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 33-37
```cpp
    // Put the whole thing on the stack.
    State.addLoc(CCValAssign::getCustomMem(
        ValNo, ValVT, State.AllocateStack(8, Align(4)), LocVT, LocInfo));
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 39-46
```cpp
  // Try to get the second register.
  if (MCRegister Reg = State.AllocateReg(RegList))
    State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
  else
    State.addLoc(CCValAssign::getCustomMem(
        ValNo, ValVT, State.AllocateStack(4, Align(4)), LocVT, LocInfo));
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 48-58
```cpp
static bool CC_ARM_APCS_Custom_f64(unsigned ValNo, MVT ValVT, MVT LocVT,
                                   CCValAssign::LocInfo LocInfo,
                                   ISD::ArgFlagsTy ArgFlags,
                                   CCState &State) {
  if (!f64AssignAPCS(ValNo, ValVT, LocVT, LocInfo, State, true))
    return false;
  if (LocVT == MVT::v2f64 &&
      !f64AssignAPCS(ValNo, ValVT, LocVT, LocInfo, State, false))
    return false;
  return true;  // we handled it
}
```
- EN: Implements `CC_ARM_APCS_Custom_f64`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_ARM_APCS_Custom_f64`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 60-67
```cpp
// AAPCS f64 is in aligned register pairs
static bool f64AssignAAPCS(unsigned ValNo, MVT ValVT, MVT LocVT,
                           CCValAssign::LocInfo LocInfo,
                           CCState &State, bool CanFail) {
  static const MCPhysReg HiRegList[] = { ARM::R0, ARM::R2 };
  static const MCPhysReg LoRegList[] = { ARM::R1, ARM::R3 };
  static const MCPhysReg ShadowRegList[] = { ARM::R0, ARM::R1 };
  static const MCPhysReg GPRArgRegs[] = { ARM::R0, ARM::R1, ARM::R2, ARM::R3 };
```
- EN: Implements `f64AssignAAPCS`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `f64AssignAAPCS`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 69-70
```cpp
  MCRegister Reg = State.AllocateReg(HiRegList, ShadowRegList);
  if (!Reg) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 72-74
```cpp
    // If we had R3 unallocated only, now we still must to waste it.
    Reg = State.AllocateReg(GPRArgRegs);
    assert((!Reg || Reg == ARM::R3) && "Wrong GPRs usage for f64");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 76-78
```cpp
    // For the 2nd half of a v2f64, do not just fail.
    if (CanFail)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 80-84
```cpp
    // Put the whole thing on the stack.
    State.addLoc(CCValAssign::getCustomMem(
        ValNo, ValVT, State.AllocateStack(8, Align(8)), LocVT, LocInfo));
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 86-89
```cpp
  unsigned i;
  for (i = 0; i < 2; ++i)
    if (HiRegList[i] == Reg)
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 91-93
```cpp
  MCRegister T = State.AllocateReg(LoRegList[i]);
  (void)T;
  assert(T == LoRegList[i] && "Could not allocate register");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-99
```cpp
  State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
  State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, LoRegList[i],
                                         LocVT, LocInfo));
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 101-111
```cpp
static bool CC_ARM_AAPCS_Custom_f64(unsigned ValNo, MVT ValVT, MVT LocVT,
                                    CCValAssign::LocInfo LocInfo,
                                    ISD::ArgFlagsTy ArgFlags,
                                    CCState &State) {
  if (!f64AssignAAPCS(ValNo, ValVT, LocVT, LocInfo, State, true))
    return false;
  if (LocVT == MVT::v2f64 &&
      !f64AssignAAPCS(ValNo, ValVT, LocVT, LocInfo, State, false))
    return false;
  return true;  // we handled it
}
```
- EN: Implements `CC_ARM_AAPCS_Custom_f64`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_ARM_AAPCS_Custom_f64`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 113-116
```cpp
static bool f64RetAssign(unsigned ValNo, MVT ValVT, MVT LocVT,
                         CCValAssign::LocInfo LocInfo, CCState &State) {
  static const MCPhysReg HiRegList[] = { ARM::R0, ARM::R2 };
  static const MCPhysReg LoRegList[] = { ARM::R1, ARM::R3 };
```
- EN: Implements `f64RetAssign`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `f64RetAssign`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 118-120
```cpp
  MCRegister Reg = State.AllocateReg(HiRegList, LoRegList);
  if (!Reg)
    return false; // we didn't handle it
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 122-125
```cpp
  unsigned i;
  for (i = 0; i < 2; ++i)
    if (HiRegList[i] == Reg)
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 127-131
```cpp
  State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
  State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, LoRegList[i],
                                         LocVT, LocInfo));
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 133-142
```cpp
static bool RetCC_ARM_APCS_Custom_f64(unsigned ValNo, MVT ValVT, MVT LocVT,
                                      CCValAssign::LocInfo LocInfo,
                                      ISD::ArgFlagsTy ArgFlags,
                                      CCState &State) {
  if (!f64RetAssign(ValNo, ValVT, LocVT, LocInfo, State))
    return false;
  if (LocVT == MVT::v2f64 && !f64RetAssign(ValNo, ValVT, LocVT, LocInfo, State))
    return false;
  return true;  // we handled it
}
```
- EN: Implements `RetCC_ARM_APCS_Custom_f64`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RetCC_ARM_APCS_Custom_f64`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 144-150
```cpp
static bool RetCC_ARM_AAPCS_Custom_f64(unsigned ValNo, MVT ValVT, MVT LocVT,
                                       CCValAssign::LocInfo LocInfo,
                                       ISD::ArgFlagsTy ArgFlags,
                                       CCState &State) {
  return RetCC_ARM_APCS_Custom_f64(ValNo, ValVT, LocVT, LocInfo, ArgFlags,
                                   State);
}
```
- EN: Implements `RetCC_ARM_AAPCS_Custom_f64`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RetCC_ARM_AAPCS_Custom_f64`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-152
```cpp
static const MCPhysReg RRegList[] = { ARM::R0,  ARM::R1,  ARM::R2,  ARM::R3 };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-160
```cpp
static const MCPhysReg SRegList[] = { ARM::S0,  ARM::S1,  ARM::S2,  ARM::S3,
                                      ARM::S4,  ARM::S5,  ARM::S6,  ARM::S7,
                                      ARM::S8,  ARM::S9,  ARM::S10, ARM::S11,
                                      ARM::S12, ARM::S13, ARM::S14,  ARM::S15 };
static const MCPhysReg DRegList[] = { ARM::D0, ARM::D1, ARM::D2, ARM::D3,
                                      ARM::D4, ARM::D5, ARM::D6, ARM::D7 };
static const MCPhysReg QRegList[] = { ARM::Q0, ARM::Q1, ARM::Q2, ARM::Q3 };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-173
```cpp
// Allocate part of an AAPCS HFA or HVA. We assume that each member of the HA
// has InConsecutiveRegs set, and that the last member also has
// InConsecutiveRegsLast set. We must process all members of the HA before
// we can allocate it, as we need to know the total number of registers that
// will be needed in order to (attempt to) allocate a contiguous block.
static bool CC_ARM_AAPCS_Custom_Aggregate(unsigned ValNo, MVT ValVT,
                                          MVT LocVT,
                                          CCValAssign::LocInfo LocInfo,
                                          ISD::ArgFlagsTy ArgFlags,
                                          CCState &State) {
  SmallVectorImpl<CCValAssign> &PendingMembers = State.getPendingLocs();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 175-177
```cpp
  // AAPCS HFAs must have 1-4 elements, all of the same type
  if (PendingMembers.size() > 0)
    assert(PendingMembers[0].getLocVT() == LocVT);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 179-184
```cpp
  // Add the argument to the list to be allocated once we know the size of the
  // aggregate. Store the type's required alignment as extra info for later: in
  // the [N x i64] case all trace has been removed by the time we actually get
  // to do allocation.
  PendingMembers.push_back(CCValAssign::getPending(
      ValNo, ValVT, LocVT, LocInfo, ArgFlags.getNonZeroOrigAlign().value()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-187
```cpp
  if (!ArgFlags.isInConsecutiveRegsLast())
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 189-196
```cpp
  const MachineFunction &MF = State.getMachineFunction();
  // Try to allocate a contiguous block of registers, each of the correct
  // size to hold one member.
  auto &DL = MF.getDataLayout();
  const MaybeAlign StackAlign = DL.getStackAlignment();
  assert(StackAlign && "data layout string is missing stack alignment");
  const Align FirstMemberAlign(PendingMembers[0].getExtraInfo());
  Align Alignment = std::min(FirstMemberAlign, *StackAlign);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 198-202
```cpp
  ArrayRef<MCPhysReg> RegList;
  switch (LocVT.SimpleTy) {
  case MVT::i32: {
    RegList = RRegList;
    unsigned RegIdx = State.getFirstUnallocated(RegList);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 204-208
```cpp
    // First consume all registers that would give an unaligned object. Whether
    // we go on stack or in regs, no-one will be using them in future.
    unsigned RegAlign = alignTo(Alignment.value(), 4) / 4;
    while (RegIdx % RegAlign != 0 && RegIdx < RegList.size())
      State.AllocateReg(RegList[RegIdx++]);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 210-224
```cpp
    break;
  }
  case MVT::f16:
  case MVT::bf16:
  case MVT::f32:
    RegList = SRegList;
    break;
  case MVT::v4f16:
  case MVT::v4bf16:
  case MVT::f64:
    RegList = DRegList;
    break;
  case MVT::v8f16:
  case MVT::v8bf16:
  case MVT::v2f64:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 225-230
```cpp
    RegList = QRegList;
    break;
  default:
    llvm_unreachable("Unexpected member type for block aggregate");
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-241
```cpp
  ArrayRef<MCPhysReg> RegResult =
      State.AllocateRegBlock(RegList, PendingMembers.size());
  if (!RegResult.empty()) {
    for (const auto &[PendingMember, Reg] : zip(PendingMembers, RegResult)) {
      PendingMember.convertToReg(Reg);
      State.addLoc(PendingMember);
    }
    PendingMembers.clear();
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 243-253
```cpp
  // Register allocation failed, we'll be needing the stack
  unsigned Size = LocVT.getSizeInBits() / 8;
  if (LocVT == MVT::i32 && State.getStackSize() == 0) {
    // If nothing else has used the stack until this point, a non-HFA aggregate
    // can be split between regs and stack.
    unsigned RegIdx = State.getFirstUnallocated(RegList);
    for (auto &It : PendingMembers) {
      if (RegIdx >= RegList.size())
        It.convertToMem(State.AllocateStack(Size, Align(Size)));
      else
        It.convertToReg(State.AllocateReg(RegList[RegIdx++]));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 255-259
```cpp
      State.addLoc(It);
    }
    PendingMembers.clear();
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 261-262
```cpp
  if (LocVT != MVT::i32)
    RegList = SRegList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 264-266
```cpp
  // Mark all regs as unavailable (AAPCS rule C.2.vfp for VFP, C.6 for core)
  for (auto Reg : RegList)
    State.AllocateReg(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 268-270
```cpp
  // Clamp the alignment between 4 and 8.
  if (MF.getTarget().getTargetTriple().isTargetAEABI())
    Alignment = ArgFlags.getNonZeroMemAlign() <= 4 ? Align(4) : Align(8);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 272-279
```cpp
  // After the first item has been allocated, the rest are packed as tightly as
  // possible. (E.g. an incoming i64 would have starting Align of 8, but we'll
  // be allocating a bunch of i32 slots).
  for (auto &It : PendingMembers) {
    It.convertToMem(State.AllocateStack(Size, Alignment));
    State.addLoc(It);
    Alignment = Align(1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 281-282
```cpp
  // All pending members have now been allocated
  PendingMembers.clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 284-286
```cpp
  // This will be allocated by the last member of the aggregate
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 288-297
```cpp
static bool CustomAssignInRegList(unsigned ValNo, MVT ValVT, MVT LocVT,
                                  CCValAssign::LocInfo LocInfo, CCState &State,
                                  ArrayRef<MCPhysReg> RegList) {
  MCRegister Reg = State.AllocateReg(RegList);
  if (Reg) {
    State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
    return true;
  }
  return false;
}
```
- EN: Implements `CustomAssignInRegList`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CustomAssignInRegList`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 299-306
```cpp
static bool CC_ARM_AAPCS_Custom_f16(unsigned ValNo, MVT ValVT, MVT LocVT,
                                    CCValAssign::LocInfo LocInfo,
                                    ISD::ArgFlagsTy ArgFlags, CCState &State) {
  // f16 and bf16 arguments are extended to i32 and assigned to a register in
  // [r0, r3].
  return CustomAssignInRegList(ValNo, ValVT, MVT::i32, LocInfo, State,
                               RRegList);
}
```
- EN: Implements `CC_ARM_AAPCS_Custom_f16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_ARM_AAPCS_Custom_f16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 308-316
```cpp
static bool CC_ARM_AAPCS_VFP_Custom_f16(unsigned ValNo, MVT ValVT, MVT LocVT,
                                        CCValAssign::LocInfo LocInfo,
                                        ISD::ArgFlagsTy ArgFlags,
                                        CCState &State) {
  // f16 and bf16 arguments are extended to f32 and assigned to a register in
  // [s0, s15].
  return CustomAssignInRegList(ValNo, ValVT, MVT::f32, LocInfo, State,
                               SRegList);
}
```
- EN: Implements `CC_ARM_AAPCS_VFP_Custom_f16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CC_ARM_AAPCS_VFP_Custom_f16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 318-330
```cpp
static bool CC_ARM_AAPCS_Common_Custom_f16_Stack(unsigned ValNo, MVT ValVT,
                                                 MVT LocVT,
                                                 CCValAssign::LocInfo LocInfo,
                                                 ISD::ArgFlagsTy ArgFlags,
                                                 CCState &State) {
  // f16 and bf16 (if not passed in a register) are assigned to a 32-bit stack
  // slot, with the most-significant 16 bits unspecified. The 32-bit slot is
  // important to make sure that the byte ordering is correct for big endian
  // targets.
  State.addLoc(CCValAssign::getCustomMem(
      ValNo, ValVT, State.AllocateStack(4, Align(4)), MVT::i32, LocInfo));
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 332-333
```cpp
// Include the table generated calling convention implementations.
#include "ARMGenCallingConv.inc"
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: calling-convention rules and ABI lowering.
  - CN: 核心职责：调用约定规则与 ABI 降级。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMCallingConv.h`, `ARM.h`, `ARMSubtarget.h`, `ARMGenCallingConv.inc`.
  - CN: 后端本地头文件：`ARMCallingConv.h`, `ARM.h`, `ARMSubtarget.h`, `ARMGenCallingConv.inc`。
- EN: LLVM infrastructure headers: `(none)`.
  - CN: LLVM 基础设施头文件：`(none)`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
