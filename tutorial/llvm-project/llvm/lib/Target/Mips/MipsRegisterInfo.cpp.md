# MipsRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsRegisterInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the MIPS implementation of the TargetRegisterInfo class.
- 用途 (CN): 实现 Mips 后端中的 `MipsRegisterInfo`，重点处理寄存器定义、分配约束以及寄存器工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- MipsRegisterInfo.cpp - MIPS Register Information -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the MIPS implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MipsRegisterInfo.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "Mips.h"
#include "MipsMachineFunction.h"
#include "MipsSubtarget.h"
#include "MipsTargetMachine.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/Function.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-31
```cpp
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 33-33
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 35-35
```cpp
#define DEBUG_TYPE "mips-reg-info"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 37-38
```cpp
#define GET_REGINFO_TARGET_DESC
#include "MipsGenRegisterInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 40-43
```cpp
MipsRegisterInfo::MipsRegisterInfo(const MipsSubtarget &STI)
    : MipsGenRegisterInfo(Mips::RA), ArePtrs64bit(STI.getABI().ArePtrs64bit()) {
  MIPS_MC::initLLVMToCVRegMapping(this);
}
```
- EN: Implements `MipsRegisterInfo::MipsRegisterInfo`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::MipsRegisterInfo`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 45-45
```cpp
unsigned MipsRegisterInfo::getPICCallReg() { return Mips::T9; }
```
- EN: Implements `MipsRegisterInfo::getPICCallReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::getPICCallReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-51
```cpp
const TargetRegisterClass *
MipsRegisterInfo::getPointerRegClass(unsigned Kind) const {
  assert(Kind == 0 && "this should only be used for default case");
  return ArePtrs64bit ? &Mips::GPR64RegClass : &Mips::GPR32RegClass;
}
```
- EN: Implements `MipsRegisterInfo::getPointerRegClass`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::getPointerRegClass`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 53-67
```cpp
unsigned
MipsRegisterInfo::getRegPressureLimit(const TargetRegisterClass *RC,
                                      MachineFunction &MF) const {
  switch (RC->getID()) {
  default:
    return 0;
  case Mips::GPR32RegClassID:
  case Mips::GPR64RegClassID:
  case Mips::DSPRRegClassID: {
    const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
    return 28 - TFI->hasFP(MF);
  }
  case Mips::FGR32RegClassID:
    return 32;
  case Mips::AFGR64RegClassID:
```
- EN: Implements `MipsRegisterInfo::getRegPressureLimit`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::getRegPressureLimit`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 68-72
```cpp
    return 16;
  case Mips::FGR64RegClassID:
    return 32;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 74-76
```cpp
//===----------------------------------------------------------------------===//
// Callee Saved Registers methods
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 78-90
```cpp
/// Mips Callee Saved Registers
const MCPhysReg *
MipsRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  const MipsSubtarget &Subtarget = MF->getSubtarget<MipsSubtarget>();
  const Function &F = MF->getFunction();
  if (F.hasFnAttribute("interrupt")) {
    if (Subtarget.hasMips64())
      return Subtarget.hasMips64r6() ? CSR_Interrupt_64R6_SaveList
                                     : CSR_Interrupt_64_SaveList;
    else
      return Subtarget.hasMips32r6() ? CSR_Interrupt_32R6_SaveList
                                     : CSR_Interrupt_32_SaveList;
  }
```
- EN: Implements `MipsRegisterInfo::getCalleeSavedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::getCalleeSavedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 92-95
```cpp
  // N64 ABI
  if (Subtarget.isABI_N64()) {
    if (Subtarget.isSingleFloat())
      return CSR_N64_SingleFloat_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 97-98
```cpp
    return CSR_N64_SaveList;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 100-103
```cpp
  // N32 ABI
  if (Subtarget.isABI_N32()) {
    if (Subtarget.isSingleFloat())
      return CSR_N32_SingleFloat_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 105-106
```cpp
    return CSR_N32_SaveList;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 108-110
```cpp
  // O32 ABI
  if (Subtarget.isSingleFloat())
    return CSR_O32_SingleFloat_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 112-113
```cpp
  if (Subtarget.isFP64bit())
    return CSR_O32_FP64_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 115-116
```cpp
  if (Subtarget.isFPXX())
    return CSR_O32_FPXX_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 118-119
```cpp
  return CSR_O32_SaveList;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-128
```cpp
const uint32_t *
MipsRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
                                       CallingConv::ID) const {
  const MipsSubtarget &Subtarget = MF.getSubtarget<MipsSubtarget>();
  // N64 ABI
  if (Subtarget.isABI_N64()) {
    if (Subtarget.isSingleFloat())
      return CSR_N64_SingleFloat_RegMask;
```
- EN: Implements `MipsRegisterInfo::getCallPreservedMask`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::getCallPreservedMask`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 130-131
```cpp
    return CSR_N64_RegMask;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 133-136
```cpp
  // N32 ABI
  if (Subtarget.isABI_N32()) {
    if (Subtarget.isSingleFloat())
      return CSR_N32_SingleFloat_RegMask;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 138-139
```cpp
    return CSR_N32_RegMask;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-143
```cpp
  // O32 ABI
  if (Subtarget.isSingleFloat())
    return CSR_O32_SingleFloat_RegMask;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 145-146
```cpp
  if (Subtarget.isFP64bit())
    return CSR_O32_FP64_RegMask;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 148-149
```cpp
  if (Subtarget.isFPXX())
    return CSR_O32_FPXX_RegMask;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 151-152
```cpp
  return CSR_O32_RegMask;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-156
```cpp
const uint32_t *MipsRegisterInfo::getMips16RetHelperMask() {
  return CSR_Mips16RetHelper_RegMask;
}
```
- EN: Implements `MipsRegisterInfo::getMips16RetHelperMask`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::getMips16RetHelperMask`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 158-162
```cpp
BitVector MipsRegisterInfo::
getReservedRegs(const MachineFunction &MF) const {
  static const MCPhysReg ReservedGPR32[] = {
    Mips::ZERO, Mips::K0, Mips::K1, Mips::SP
  };
```
- EN: Implements `getReservedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getReservedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 164-166
```cpp
  static const MCPhysReg ReservedGPR64[] = {
    Mips::ZERO_64, Mips::K0_64, Mips::K1_64, Mips::SP_64
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 168-169
```cpp
  BitVector Reserved(getNumRegs());
  const MipsSubtarget &Subtarget = MF.getSubtarget<MipsSubtarget>();
```
- EN: Declares `Reserved`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Reserved`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 171-172
```cpp
  for (MCPhysReg R : ReservedGPR32)
    Reserved.set(R);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 174-175
```cpp
  for (MCPhysReg R : ReservedGPR64)
    Reserved.set(R);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 177-181
```cpp
  // For mno-abicalls, GP is a program invariant!
  if (!Subtarget.isABICalls()) {
    Reserved.set(Mips::GP);
    Reserved.set(Mips::GP_64);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 183-197
```cpp
  if (Subtarget.isFP64bit()) {
    // Reserve all registers in AFGR64.
    for (MCPhysReg Reg : Mips::AFGR64RegClass)
      Reserved.set(Reg);
  } else {
    // Reserve all registers in FGR64.
    for (MCPhysReg Reg : Mips::FGR64RegClass)
      Reserved.set(Reg);
  }
  // Reserve FP if this function should have a dedicated frame pointer register.
  if (Subtarget.getFrameLowering()->hasFP(MF)) {
    if (Subtarget.inMips16Mode())
      Reserved.set(Mips::S0);
    else {
      Reserved.set(Mips::FP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 198-198
```cpp
      Reserved.set(Mips::FP_64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 200-210
```cpp
      // Reserve the base register if we need to both realign the stack and
      // allocate variable-sized objects at runtime. This should test the
      // same conditions as MipsFrameLowering::hasBP().
      if (hasStackRealignment(MF) && MF.getFrameInfo().hasVarSizedObjects()) {
        Reserved.set(Mips::S7);
        Reserved.set(Mips::S7_64);
      }
    }
  }
  // Reserve fp control and status register
  Reserved.set(Mips::FCR31);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 212-214
```cpp
  // Reserve hardware registers.
  Reserved.set(Mips::HWR29);
  Reserved.set(Mips::HWR2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 216-221
```cpp
  // Reserve DSP control register.
  Reserved.set(Mips::DSPPos);
  Reserved.set(Mips::DSPSCount);
  Reserved.set(Mips::DSPCarry);
  Reserved.set(Mips::DSPEFI);
  Reserved.set(Mips::DSPOutFlag);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 223-225
```cpp
  // Reserve MSA control registers.
  for (MCPhysReg Reg : Mips::MSACtrlRegClass)
    Reserved.set(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-236
```cpp
  // Reserve RA if in mips16 mode.
  if (Subtarget.inMips16Mode()) {
    const MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
    Reserved.set(Mips::RA);
    Reserved.set(Mips::RA_64);
    Reserved.set(Mips::T0);
    Reserved.set(Mips::T1);
    if (MF.getFunction().hasFnAttribute("saveS2") || MipsFI->hasSaveS2())
      Reserved.set(Mips::S2);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 238-242
```cpp
  // Reserve GP if small section is used.
  if (Subtarget.useSmallSection()) {
    Reserved.set(Mips::GP);
    Reserved.set(Mips::GP_64);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 244-245
```cpp
  return Reserved;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 247-254
```cpp
// FrameIndex represent objects inside a abstract stack.
// We must replace FrameIndex with an stack/frame pointer
// direct reference.
bool MipsRegisterInfo::
eliminateFrameIndex(MachineBasicBlock::iterator II, int SPAdj,
                    unsigned FIOperandNum, RegScavenger *RS) const {
  MachineInstr &MI = *II;
  MachineFunction &MF = *MI.getParent()->getParent();
```
- EN: Implements `eliminateFrameIndex`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `eliminateFrameIndex`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 256-258
```cpp
  LLVM_DEBUG(errs() << "\nFunction : " << MF.getName() << "\n";
             errs() << "<--------->\n"
                    << MI);
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 260-262
```cpp
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
  uint64_t stackSize = MF.getFrameInfo().getStackSize();
  int64_t spOffset = MF.getFrameInfo().getObjectOffset(FrameIndex);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 264-269
```cpp
  LLVM_DEBUG(errs() << "FrameIndex : " << FrameIndex << "\n"
                    << "spOffset   : " << spOffset << "\n"
                    << "stackSize  : " << stackSize << "\n"
                    << "alignment  : "
                    << DebugStr(MF.getFrameInfo().getObjectAlign(FrameIndex))
                    << "\n");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 271-273
```cpp
  eliminateFI(MI, FIOperandNum, FrameIndex, stackSize, spOffset);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 275-280
```cpp
Register MipsRegisterInfo::
getFrameRegister(const MachineFunction &MF) const {
  const MipsSubtarget &Subtarget = MF.getSubtarget<MipsSubtarget>();
  const TargetFrameLowering *TFI = Subtarget.getFrameLowering();
  bool IsN64 =
      static_cast<const MipsTargetMachine &>(MF.getTarget()).getABI().IsN64();
```
- EN: Implements `getFrameRegister`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFrameRegister`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 282-287
```cpp
  if (Subtarget.inMips16Mode())
    return TFI->hasFP(MF) ? Mips::S0 : Mips::SP;
  else
    return TFI->hasFP(MF) ? (IsN64 ? Mips::FP_64 : Mips::FP) :
                            (IsN64 ? Mips::SP_64 : Mips::SP);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 289-298
```cpp
bool MipsRegisterInfo::canRealignStack(const MachineFunction &MF) const {
  // Avoid realigning functions that explicitly do not want to be realigned.
  // Normally, we should report an error when a function should be dynamically
  // realigned but also has the attribute no-realign-stack. Unfortunately,
  // with this attribute, MachineFrameInfo clamps each new object's alignment
  // to that of the stack's alignment as specified by the ABI. As a result,
  // the information of whether we have objects with larger alignment
  // requirement than the stack's alignment is already lost at this point.
  if (!TargetRegisterInfo::canRealignStack(MF))
    return false;
```
- EN: Implements `MipsRegisterInfo::canRealignStack`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsRegisterInfo::canRealignStack`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 300-302
```cpp
  const MipsSubtarget &Subtarget = MF.getSubtarget<MipsSubtarget>();
  unsigned FP = Subtarget.isGP32bit() ? Mips::FP : Mips::FP_64;
  unsigned BP = Subtarget.isGP32bit() ? Mips::S7 : Mips::S7_64;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 304-306
```cpp
  // Support dynamic stack realignment for all targets except Mips16.
  if (Subtarget.inMips16Mode())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 308-311
```cpp
  // We can't perform dynamic stack realignment if we can't reserve the
  // frame pointer register.
  if (!MF.getRegInfo().canReserveReg(FP))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 313-316
```cpp
  // We can realign the stack if we know the maximum call frame size and we
  // don't have variable sized objects.
  if (Subtarget.getFrameLowering()->hasReservedCallFrame(MF))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 318-321
```cpp
  // We have to reserve the base pointer register in the presence of variable
  // sized objects.
  return MF.getRegInfo().canReserveReg(BP);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: register definitions, allocation constraints, and register utilities.
  - CN: 核心职责：寄存器定义、分配约束以及寄存器工具。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsRegisterInfo.h`, `MCTargetDesc/MipsABIInfo.h`, `Mips.h`, `MipsMachineFunction.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`, `MipsGenRegisterInfo.inc`.
  - CN: 后端本地头文件：`MipsRegisterInfo.h`, `MCTargetDesc/MipsABIInfo.h`, `Mips.h`, `MipsMachineFunction.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`, `MipsGenRegisterInfo.inc`。
- EN: LLVM infrastructure headers: `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h` ... (+4 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h` ... (+4 more)。
- EN: Standard/system headers: `cstdint`.
  - CN: 标准库/系统头文件：`cstdint`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
