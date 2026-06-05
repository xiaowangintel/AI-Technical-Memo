# ARMBaseRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMBaseRegisterInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the base ARM implementation of TargetRegisterInfo class.
- 用途 (CN): 实现 ARM 后端中的 `ARMBaseRegisterInfo`，重点处理共享寄存器定义与寄存器管理辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMBaseRegisterInfo.cpp - ARM Register Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the base ARM implementation of TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "ARMBaseRegisterInfo.h"
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMFrameLowering.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-42
```cpp
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/VirtRegMap.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Type.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/Support/Debug.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-48
```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Target/TargetOptions.h"
#include <cassert>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 50-50
```cpp
#define DEBUG_TYPE "arm-register-info"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 52-53
```cpp
#define GET_REGINFO_TARGET_DESC
#include "ARMGenRegisterInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 55-55
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 57-60
```cpp
ARMBaseRegisterInfo::ARMBaseRegisterInfo()
    : ARMGenRegisterInfo(ARM::LR, 0, 0, ARM::PC) {
  ARM_MC::initLLVMToCVRegMapping(this);
}
```
- EN: Implements `ARMBaseRegisterInfo::ARMBaseRegisterInfo`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::ARMBaseRegisterInfo`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-67
```cpp
const MCPhysReg*
ARMBaseRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  const ARMSubtarget &STI = MF->getSubtarget<ARMSubtarget>();
  ARMSubtarget::PushPopSplitVariation PushPopSplit =
      STI.getPushPopSplitVariation(*MF);
  const Function &F = MF->getFunction();
```
- EN: Implements `ARMBaseRegisterInfo::getCalleeSavedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getCalleeSavedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 69-82
```cpp
  if (F.getCallingConv() == CallingConv::GHC) {
    // GHC set of callee saved regs is empty as all those regs are
    // used for passing STG regs around
    return CSR_NoRegs_SaveList;
  } else if (PushPopSplit == ARMSubtarget::SplitR11WindowsSEH) {
    return CSR_Win_SplitFP_SaveList;
  } else if (F.getCallingConv() == CallingConv::CFGuard_Check) {
    return CSR_Win_AAPCS_CFGuard_Check_SaveList;
  } else if (F.getCallingConv() == CallingConv::SwiftTail) {
    return STI.isTargetDarwin() ? CSR_iOS_SwiftTail_SaveList
                                : (PushPopSplit == ARMSubtarget::SplitR7
                                       ? CSR_ATPCS_SplitPush_SwiftTail_SaveList
                                       : CSR_AAPCS_SwiftTail_SaveList);
  } else if (F.hasFnAttribute("interrupt")) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 84-87
```cpp
    // Don't save the floating point registers if target does not have floating
    // point registers.
    if (STI.hasFPRegs() && F.hasFnAttribute("save-fp")) {
      bool HasNEON = STI.hasNEON();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 89-100
```cpp
      if (STI.isMClass()) {
        assert(!HasNEON && "NEON is only for Cortex-R/A");
        return PushPopSplit == ARMSubtarget::SplitR7
                   ? CSR_ATPCS_SplitPush_FP_SaveList
                   : CSR_AAPCS_FP_SaveList;
      }
      if (F.getFnAttribute("interrupt").getValueAsString() == "FIQ") {
        return HasNEON ? CSR_FIQ_FP_NEON_SaveList : CSR_FIQ_FP_SaveList;
      }
      return HasNEON ? CSR_GenericInt_FP_NEON_SaveList
                     : CSR_GenericInt_FP_SaveList;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 102-116
```cpp
    if (STI.isMClass()) {
      // M-class CPUs have hardware which saves the registers needed to allow a
      // function conforming to the AAPCS to function as a handler.
      return PushPopSplit == ARMSubtarget::SplitR7
                 ? CSR_ATPCS_SplitPush_SaveList
                 : CSR_AAPCS_SaveList;
    } else if (F.getFnAttribute("interrupt").getValueAsString() == "FIQ") {
      // Fast interrupt mode gives the handler a private copy of R8-R14, so less
      // need to be saved to restore user-mode state.
      return CSR_FIQ_SaveList;
    } else {
      // Generally only R13-R14 (i.e. SP, LR) are automatically preserved by
      // exception handling.
      return CSR_GenericInt_SaveList;
    }
```
- EN: Declares `CPUs`, packaging target-specific state and APIs around `ARMBaseRegisterInfo`.
- CN: 这里声明 `CPUs`，把与 `ARMBaseRegisterInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 117-117
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 119-122
```cpp
  if (STI.getTargetLowering()->supportSwiftError() &&
      F.getAttributes().hasAttrSomewhere(Attribute::SwiftError)) {
    if (STI.isTargetDarwin())
      return CSR_iOS_SwiftError_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 124-127
```cpp
    return PushPopSplit == ARMSubtarget::SplitR7
               ? CSR_ATPCS_SplitPush_SwiftError_SaveList
               : CSR_AAPCS_SwiftError_SaveList;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 129-132
```cpp
  if (STI.isTargetDarwin() && F.getCallingConv() == CallingConv::CXX_FAST_TLS)
    return MF->getInfo<ARMFunctionInfo>()->isSplitCSR()
               ? CSR_iOS_CXX_TLS_PE_SaveList
               : CSR_iOS_CXX_TLS_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 134-135
```cpp
  if (STI.isTargetDarwin())
    return CSR_iOS_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 137-139
```cpp
  if (PushPopSplit == ARMSubtarget::SplitR7)
    return STI.createAAPCSFrameChain() ? CSR_AAPCS_SplitPush_R7_SaveList
                                       : CSR_ATPCS_SplitPush_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 141-142
```cpp
  if (PushPopSplit == ARMSubtarget::SplitR11AAPCSSignRA)
    return CSR_AAPCS_SplitPush_R11_SaveList;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 144-145
```cpp
  return CSR_AAPCS_SaveList;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 147-154
```cpp
const MCPhysReg *ARMBaseRegisterInfo::getCalleeSavedRegsViaCopy(
    const MachineFunction *MF) const {
  assert(MF && "Invalid MachineFunction pointer.");
  if (MF->getFunction().getCallingConv() == CallingConv::CXX_FAST_TLS &&
      MF->getInfo<ARMFunctionInfo>()->isSplitCSR())
    return CSR_iOS_CXX_TLS_ViaCopy_SaveList;
  return nullptr;
}
```
- EN: Implements `ARMBaseRegisterInfo::getCalleeSavedRegsViaCopy`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getCalleeSavedRegsViaCopy`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 156-170
```cpp
const uint32_t *
ARMBaseRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
                                          CallingConv::ID CC) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  if (CC == CallingConv::GHC)
    // This is academic because all GHC calls are (supposed to be) tail calls
    return CSR_NoRegs_RegMask;
  if (CC == CallingConv::CFGuard_Check)
    return CSR_Win_AAPCS_CFGuard_Check_RegMask;
  if (CC == CallingConv::SwiftTail) {
    return STI.isTargetDarwin() ? CSR_iOS_SwiftTail_RegMask
                                : CSR_AAPCS_SwiftTail_RegMask;
  }
  if (STI.getTargetLowering()->supportSwiftError() &&
      MF.getFunction().getAttributes().hasAttrSomewhere(Attribute::SwiftError))
```
- EN: Implements `ARMBaseRegisterInfo::getCallPreservedMask`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getCallPreservedMask`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 171-172
```cpp
    return STI.isTargetDarwin() ? CSR_iOS_SwiftError_RegMask
                                : CSR_AAPCS_SwiftError_RegMask;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 174-177
```cpp
  if (STI.isTargetDarwin() && CC == CallingConv::CXX_FAST_TLS)
    return CSR_iOS_CXX_TLS_RegMask;
  return STI.isTargetDarwin() ? CSR_iOS_RegMask : CSR_AAPCS_RegMask;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 179-182
```cpp
const uint32_t*
ARMBaseRegisterInfo::getNoPreservedMask() const {
  return CSR_NoRegs_RegMask;
}
```
- EN: Implements `ARMBaseRegisterInfo::getNoPreservedMask`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getNoPreservedMask`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-189
```cpp
const uint32_t *
ARMBaseRegisterInfo::getTLSCallPreservedMask(const MachineFunction &MF) const {
  assert(MF.getSubtarget<ARMSubtarget>().isTargetDarwin() &&
         "only know about special TLS call on Darwin");
  return CSR_iOS_TLSCall_RegMask;
}
```
- EN: Implements `ARMBaseRegisterInfo::getTLSCallPreservedMask`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getTLSCallPreservedMask`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-198
```cpp
const uint32_t *
ARMBaseRegisterInfo::getSjLjDispatchPreservedMask(const MachineFunction &MF) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  if (!STI.useSoftFloat() && STI.hasVFP2Base() && !STI.isThumb1Only())
    return CSR_NoRegs_RegMask;
  else
    return CSR_FPRegs_RegMask;
}
```
- EN: Implements `ARMBaseRegisterInfo::getSjLjDispatchPreservedMask`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getSjLjDispatchPreservedMask`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 200-214
```cpp
const uint32_t *
ARMBaseRegisterInfo::getThisReturnPreservedMask(const MachineFunction &MF,
                                                CallingConv::ID CC) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  // This should return a register mask that is the same as that returned by
  // getCallPreservedMask but that additionally preserves the register used for
  // the first i32 argument (which must also be the register used to return a
  // single i32 return value)
  //
  // In case that the calling convention does not use the same register for
  // both or otherwise does not want to enable this optimization, the function
  // should return NULL
  if (CC == CallingConv::GHC)
    // This is academic because all GHC calls are (supposed to be) tail calls
    return nullptr;
```
- EN: Implements `ARMBaseRegisterInfo::getThisReturnPreservedMask`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getThisReturnPreservedMask`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 215-217
```cpp
  return STI.isTargetDarwin() ? CSR_iOS_ThisReturn_RegMask
                              : CSR_AAPCS_ThisReturn_RegMask;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-223
```cpp
ArrayRef<MCPhysReg> ARMBaseRegisterInfo::getIntraCallClobberedRegs(
    const MachineFunction *MF) const {
  static const MCPhysReg IntraCallClobberedRegs[] = {ARM::R12};
  return ArrayRef<MCPhysReg>(IntraCallClobberedRegs);
}
```
- EN: Implements `ARMBaseRegisterInfo::getIntraCallClobberedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getIntraCallClobberedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 225-228
```cpp
BitVector ARMBaseRegisterInfo::
getReservedRegs(const MachineFunction &MF) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
```
- EN: Implements `getReservedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getReservedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 230-244
```cpp
  // FIXME: avoid re-calculating this every time.
  BitVector Reserved(getNumRegs());
  markSuperRegs(Reserved, ARM::SP);
  markSuperRegs(Reserved, ARM::PC);
  markSuperRegs(Reserved, ARM::FPSCR);
  markSuperRegs(Reserved, ARM::FPSCR_RM);
  markSuperRegs(Reserved, ARM::APSR_NZCV);
  if (TFI->isFPReserved(MF))
    markSuperRegs(Reserved, STI.getFramePointerReg());
  if (hasBasePointer(MF))
    markSuperRegs(Reserved, BasePtr);
  // Some targets reserve R9.
  if (STI.isR9Reserved())
    markSuperRegs(Reserved, ARM::R9);
  // Reserve D16-D31 if the subtarget doesn't support them.
```
- EN: Declares `Reserved`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Reserved`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 245-256
```cpp
  if (!STI.hasD32()) {
    static_assert(ARM::D31 == ARM::D16 + 15, "Register list not consecutive!");
    for (unsigned R = 0; R < 16; ++R)
      markSuperRegs(Reserved, ARM::D16 + R);
  }
  const TargetRegisterClass &RC = ARM::GPRPairRegClass;
  for (unsigned Reg : RC)
    for (MCPhysReg S : subregs(Reg))
      if (Reserved.test(S))
        markSuperRegs(Reserved, Reg);
  // For v8.1m architecture
  markSuperRegs(Reserved, ARM::ZR);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 258-260
```cpp
  assert(checkAllSuperRegsMarked(Reserved));
  return Reserved;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 262-265
```cpp
bool ARMBaseRegisterInfo::
isAsmClobberable(const MachineFunction &MF, MCRegister PhysReg) const {
  return !getReservedRegs(MF).test(PhysReg);
}
```
- EN: Implements `isAsmClobberable`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isAsmClobberable`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 267-270
```cpp
bool ARMBaseRegisterInfo::isInlineAsmReadOnlyReg(const MachineFunction &MF,
                                                 MCRegister PhysReg) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
```
- EN: Implements `ARMBaseRegisterInfo::isInlineAsmReadOnlyReg`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::isInlineAsmReadOnlyReg`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 272-280
```cpp
  BitVector Reserved(getNumRegs());
  markSuperRegs(Reserved, ARM::PC);
  if (TFI->isFPReserved(MF))
    markSuperRegs(Reserved, STI.getFramePointerReg());
  if (hasBasePointer(MF))
    markSuperRegs(Reserved, BasePtr);
  assert(checkAllSuperRegsMarked(Reserved));
  return Reserved.test(PhysReg.id());
}
```
- EN: Declares `Reserved`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Reserved`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 282-296
```cpp
const TargetRegisterClass *
ARMBaseRegisterInfo::getLargestLegalSuperClass(const TargetRegisterClass *RC,
                                               const MachineFunction &MF) const {
  unsigned SuperID = RC->getID();
  auto I = RC->superclasses().begin();
  auto E = RC->superclasses().end();
  do {
    switch (SuperID) {
    case ARM::GPRRegClassID:
    case ARM::SPRRegClassID:
    case ARM::DPRRegClassID:
    case ARM::GPRPairRegClassID:
      return getRegClass(SuperID);
    case ARM::QPRRegClassID:
    case ARM::QQPRRegClassID:
```
- EN: Implements `ARMBaseRegisterInfo::getLargestLegalSuperClass`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getLargestLegalSuperClass`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 297-311
```cpp
    case ARM::QQQQPRRegClassID:
      if (MF.getSubtarget<ARMSubtarget>().hasNEON())
        return getRegClass(SuperID);
      break;
    case ARM::MQPRRegClassID:
    case ARM::MQQPRRegClassID:
    case ARM::MQQQQPRRegClassID:
      if (MF.getSubtarget<ARMSubtarget>().hasMVEIntegerOps())
        return getRegClass(SuperID);
      break;
    }
    SuperID = (I != E) ? *I++ : ~0U;
  } while (SuperID != ~0U);
  return RC;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 313-316
```cpp
const TargetRegisterClass *
ARMBaseRegisterInfo::getPointerRegClass(unsigned Kind) const {
  return &ARM::GPRRegClass;
}
```
- EN: Implements `ARMBaseRegisterInfo::getPointerRegClass`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getPointerRegClass`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 318-325
```cpp
const TargetRegisterClass *
ARMBaseRegisterInfo::getCrossCopyRegClass(const TargetRegisterClass *RC) const {
  if (RC == &ARM::CCRRegClass)
    return &ARM::rGPRRegClass;  // Can't copy CCR registers.
  if (RC == &ARM::cl_FPSCR_NZCVRegClass)
    return &ARM::rGPRRegClass;
  return RC;
}
```
- EN: Implements `ARMBaseRegisterInfo::getCrossCopyRegClass`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getCrossCopyRegClass`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 327-331
```cpp
unsigned
ARMBaseRegisterInfo::getRegPressureLimit(const TargetRegisterClass *RC,
                                         MachineFunction &MF) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
```
- EN: Implements `ARMBaseRegisterInfo::getRegPressureLimit`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getRegPressureLimit`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 333-347
```cpp
  switch (RC->getID()) {
  default:
    return 0;
  case ARM::tGPRRegClassID: {
    // hasFP ends up calling getMaxCallFrameComputed() which may not be
    // available when getPressureLimit() is called as part of
    // ScheduleDAGRRList.
    bool HasFP = MF.getFrameInfo().isMaxCallFrameSizeComputed()
                 ? TFI->hasFP(MF) : true;
    return 5 - HasFP;
  }
  case ARM::GPRRegClassID: {
    bool HasFP = MF.getFrameInfo().isMaxCallFrameSizeComputed()
                 ? TFI->hasFP(MF) : true;
    return 10 - HasFP - (STI.isR9Reserved() ? 1 : 0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 348-353
```cpp
  }
  case ARM::SPRRegClassID:  // Currently not used as 'rep' register class.
  case ARM::DPRRegClassID:
    return 32 - 10;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 355-362
```cpp
// Get the other register in a GPRPair.
static MCRegister getPairedGPR(MCRegister Reg, bool Odd,
                               const MCRegisterInfo *RI) {
  for (MCPhysReg Super : RI->superregs(Reg))
    if (ARM::GPRPairRegClass.contains(Super))
      return RI->getSubReg(Super, Odd ? ARM::gsub_1 : ARM::gsub_0);
  return MCRegister();
}
```
- EN: Implements `getPairedGPR`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPairedGPR`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 364-370
```cpp
// Resolve the RegPairEven / RegPairOdd register allocator hints.
bool ARMBaseRegisterInfo::getRegAllocationHints(
    Register VirtReg, ArrayRef<MCPhysReg> Order,
    SmallVectorImpl<MCPhysReg> &Hints, const MachineFunction &MF,
    const VirtRegMap *VRM, const LiveRegMatrix *Matrix) const {
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  std::pair<unsigned, Register> Hint = MRI.getRegAllocationHint(VirtReg);
```
- EN: Implements `ARMBaseRegisterInfo::getRegAllocationHints`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getRegAllocationHints`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 372-386
```cpp
  unsigned Odd;
  switch (Hint.first) {
  case ARMRI::RegPairEven:
    Odd = 0;
    break;
  case ARMRI::RegPairOdd:
    Odd = 1;
    break;
  case ARMRI::RegLR:
    TargetRegisterInfo::getRegAllocationHints(VirtReg, Order, Hints, MF, VRM);
    if (MRI.getRegClass(VirtReg)->contains(ARM::LR))
      Hints.push_back(ARM::LR);
    return false;
  default:
    return TargetRegisterInfo::getRegAllocationHints(VirtReg, Order, Hints, MF, VRM);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 387-387
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 389-394
```cpp
  // This register should preferably be even (Odd == 0) or odd (Odd == 1).
  // Check if the other part of the pair has already been assigned, and provide
  // the paired register as the first hint.
  Register Paired = Hint.second;
  if (!Paired)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 396-401
```cpp
  Register PairedPhys;
  if (Paired.isPhysical()) {
    PairedPhys = Paired;
  } else if (VRM && VRM->hasPhys(Paired)) {
    PairedPhys = getPairedGPR(VRM->getPhys(Paired), Odd, this);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 403-405
```cpp
  // First prefer the paired physreg.
  if (PairedPhys && is_contained(Order, PairedPhys))
    Hints.push_back(PairedPhys);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 407-418
```cpp
  // Then prefer even or odd registers.
  for (MCPhysReg Reg : Order) {
    if (Reg == PairedPhys || (getEncodingValue(Reg) & 1) != Odd)
      continue;
    // Don't provide hints that are paired to a reserved register.
    MCRegister Paired = getPairedGPR(Reg, !Odd, this);
    if (!Paired || MRI.isReserved(Paired))
      continue;
    Hints.push_back(Reg);
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 420-434
```cpp
void ARMBaseRegisterInfo::updateRegAllocHint(Register Reg, Register NewReg,
                                             MachineFunction &MF) const {
  MachineRegisterInfo *MRI = &MF.getRegInfo();
  std::pair<unsigned, Register> Hint = MRI->getRegAllocationHint(Reg);
  if ((Hint.first == ARMRI::RegPairOdd || Hint.first == ARMRI::RegPairEven) &&
      Hint.second.isVirtual()) {
    // If 'Reg' is one of the even / odd register pair and it's now changed
    // (e.g. coalesced) into a different register. The other register of the
    // pair allocation hint must be updated to reflect the relationship
    // change.
    Register OtherReg = Hint.second;
    Hint = MRI->getRegAllocationHint(OtherReg);
    // Make sure the pair has not already divorced.
    if (Hint.second == Reg) {
      MRI->setRegAllocationHint(OtherReg, Hint.first, NewReg);
```
- EN: Implements `ARMBaseRegisterInfo::updateRegAllocHint`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::updateRegAllocHint`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 435-443
```cpp
      if (NewReg.isVirtual())
        MRI->setRegAllocationHint(NewReg,
                                  Hint.first == ARMRI::RegPairOdd
                                      ? ARMRI::RegPairEven
                                      : ARMRI::RegPairOdd,
                                  OtherReg);
    }
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 445-448
```cpp
bool ARMBaseRegisterInfo::hasBasePointer(const MachineFunction &MF) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  const ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
```
- EN: Implements `ARMBaseRegisterInfo::hasBasePointer`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::hasBasePointer`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 450-454
```cpp
  // If we have stack realignment and VLAs, we have no pointer to use to
  // access the stack. If we have stack realignment, and a large call frame,
  // we have no place to allocate the emergency spill slot.
  if (hasStackRealignment(MF) && !TFI->hasReservedCallFrame(MF))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 456-470
```cpp
  // Thumb has trouble with negative offsets from the FP. Thumb2 has a limited
  // negative range for ldr/str (255), and Thumb1 is positive offsets only.
  //
  // It's going to be better to use the SP or Base Pointer instead. When there
  // are variable sized objects, we can't reference off of the SP, so we
  // reserve a Base Pointer.
  //
  // For Thumb2, estimate whether a negative offset from the frame pointer
  // will be sufficient to reach the whole stack frame. If a function has a
  // smallish frame, it's less likely to have lots of spills and callee saved
  // space, so it's all more likely to be within range of the frame pointer.
  // If it's wrong, the scavenger will still enable access to work, it just
  // won't be optimal.  (We should always be able to reach the emergency
  // spill slot from the frame pointer.)
  if (AFI->isThumb2Function() && MFI.hasVarSizedObjects() &&
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 471-480
```cpp
      MFI.getLocalFrameSize() >= 128)
    return true;
  // For Thumb1, if sp moves, nothing is in range, so force a base pointer.
  // This is necessary for correctness in cases where we need an emergency
  // spill slot. (In Thumb1, we can't use a negative offset from the frame
  // pointer.)
  if (AFI->isThumb1OnlyFunction() && !TFI->hasReservedCallFrame(MF))
    return true;
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 482-496
```cpp
bool ARMBaseRegisterInfo::canRealignStack(const MachineFunction &MF) const {
  const MachineRegisterInfo *MRI = &MF.getRegInfo();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  // We can't realign the stack if:
  // 1. Dynamic stack realignment is explicitly disabled,
  // 2. There are VLAs in the function and the base pointer is disabled.
  if (!TargetRegisterInfo::canRealignStack(MF))
    return false;
  // Stack realignment requires a frame pointer.  If we already started
  // register allocation with frame pointer elimination, it is too late now.
  if (!MRI->canReserveReg(STI.getFramePointerReg()))
    return false;
  // We may also need a base pointer if there are dynamic allocas or stack
  // pointer adjustments around calls.
```
- EN: Implements `ARMBaseRegisterInfo::canRealignStack`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::canRealignStack`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 497-502
```cpp
  if (TFI->hasReservedCallFrame(MF))
    return true;
  // A base pointer is required and allowed.  Check that it isn't too late to
  // reserve it.
  return MRI->canReserveReg(BasePtr);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 504-511
```cpp
bool ARMBaseRegisterInfo::
cannotEliminateFrame(const MachineFunction &MF) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  if (MF.getTarget().Options.DisableFramePointerElim(MF) && MFI.adjustsStack())
    return true;
  return MFI.hasVarSizedObjects() || MFI.isFrameAddressTaken() ||
         hasStackRealignment(MF);
}
```
- EN: Implements `cannotEliminateFrame`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `cannotEliminateFrame`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 513-516
```cpp
Register
ARMBaseRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  const ARMSubtarget &STI = MF.getSubtarget<ARMSubtarget>();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
```
- EN: Implements `ARMBaseRegisterInfo::getFrameRegister`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::getFrameRegister`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 518-521
```cpp
  if (TFI->hasFP(MF))
    return STI.getFramePointerReg();
  return ARM::SP;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 523-534
```cpp
/// emitLoadConstPool - Emits a load from constpool to materialize the
/// specified immediate.
void ARMBaseRegisterInfo::emitLoadConstPool(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
    const DebugLoc &dl, Register DestReg, unsigned SubIdx, int Val,
    ARMCC::CondCodes Pred, Register PredReg, unsigned MIFlags) const {
  MachineFunction &MF = *MBB.getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  MachineConstantPool *ConstantPool = MF.getConstantPool();
  const Constant *C =
        ConstantInt::get(Type::getInt32Ty(MF.getFunction().getContext()), Val);
  unsigned Idx = ConstantPool->getConstantPoolIndex(C, Align(4));
```
- EN: Implements `ARMBaseRegisterInfo::emitLoadConstPool`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::emitLoadConstPool`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 536-542
```cpp
  BuildMI(MBB, MBBI, dl, TII.get(ARM::LDRcp))
      .addReg(DestReg, getDefRegState(true), SubIdx)
      .addConstantPoolIndex(Idx)
      .addImm(0)
      .add(predOps(Pred, PredReg))
      .setMIFlags(MIFlags);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 544-547
```cpp
bool ARMBaseRegisterInfo::
requiresRegisterScavenging(const MachineFunction &MF) const {
  return true;
}
```
- EN: Implements `requiresRegisterScavenging`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `requiresRegisterScavenging`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 549-552
```cpp
bool ARMBaseRegisterInfo::
requiresFrameIndexScavenging(const MachineFunction &MF) const {
  return true;
}
```
- EN: Implements `requiresFrameIndexScavenging`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `requiresFrameIndexScavenging`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 554-557
```cpp
bool ARMBaseRegisterInfo::
requiresVirtualBaseRegisters(const MachineFunction &MF) const {
  return true;
}
```
- EN: Implements `requiresVirtualBaseRegisters`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `requiresVirtualBaseRegisters`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 559-573
```cpp
int64_t ARMBaseRegisterInfo::
getFrameIndexInstrOffset(const MachineInstr *MI, int Idx) const {
  const MCInstrDesc &Desc = MI->getDesc();
  unsigned AddrMode = (Desc.TSFlags & ARMII::AddrModeMask);
  int64_t InstrOffs = 0;
  int Scale = 1;
  unsigned ImmIdx = 0;
  switch (AddrMode) {
  case ARMII::AddrModeT2_i8:
  case ARMII::AddrModeT2_i8neg:
  case ARMII::AddrModeT2_i8pos:
  case ARMII::AddrModeT2_i12:
  case ARMII::AddrMode_i12:
    InstrOffs = MI->getOperand(Idx+1).getImm();
    Scale = 1;
```
- EN: Implements `getFrameIndexInstrOffset`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFrameIndexInstrOffset`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 574-588
```cpp
    break;
  case ARMII::AddrMode5: {
    // VFP address mode.
    const MachineOperand &OffOp = MI->getOperand(Idx+1);
    InstrOffs = ARM_AM::getAM5Offset(OffOp.getImm());
    if (ARM_AM::getAM5Op(OffOp.getImm()) == ARM_AM::sub)
      InstrOffs = -InstrOffs;
    Scale = 4;
    break;
  }
  case ARMII::AddrMode2:
    ImmIdx = Idx+2;
    InstrOffs = ARM_AM::getAM2Offset(MI->getOperand(ImmIdx).getImm());
    if (ARM_AM::getAM2Op(MI->getOperand(ImmIdx).getImm()) == ARM_AM::sub)
      InstrOffs = -InstrOffs;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 589-603
```cpp
    break;
  case ARMII::AddrMode3:
    ImmIdx = Idx+2;
    InstrOffs = ARM_AM::getAM3Offset(MI->getOperand(ImmIdx).getImm());
    if (ARM_AM::getAM3Op(MI->getOperand(ImmIdx).getImm()) == ARM_AM::sub)
      InstrOffs = -InstrOffs;
    break;
  case ARMII::AddrModeT1_s:
    ImmIdx = Idx+1;
    InstrOffs = MI->getOperand(ImmIdx).getImm();
    Scale = 4;
    break;
  default:
    llvm_unreachable("Unsupported addressing mode!");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 605-606
```cpp
  return InstrOffs * Scale;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 608-616
```cpp
/// needsFrameBaseReg - Returns true if the instruction's frame index
/// reference would be better served by a base register other than FP
/// or SP. Used by LocalStackFrameAllocation to determine which frame index
/// references it should create new base registers for.
bool ARMBaseRegisterInfo::
needsFrameBaseReg(MachineInstr *MI, int64_t Offset) const {
  for (unsigned i = 0; !MI->getOperand(i).isFI(); ++i) {
    assert(i < MI->getNumOperands() &&"Instr doesn't have FrameIndex operand!");
  }
```
- EN: Implements `needsFrameBaseReg`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `needsFrameBaseReg`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 618-623
```cpp
  // It's the load/store FI references that cause issues, as it can be difficult
  // to materialize the offset if it won't fit in the literal field. Estimate
  // based on the size of the local frame and some conservative assumptions
  // about the rest of the stack frame (note, this is pre-regalloc, so
  // we don't know everything for certain yet) whether this offset is likely
  // to be out of range of the immediate. Return true if so.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 625-639
```cpp
  // We only generate virtual base registers for loads and stores, so
  // return false for everything else.
  unsigned Opc = MI->getOpcode();
  switch (Opc) {
  case ARM::LDRi12: case ARM::LDRH: case ARM::LDRBi12:
  case ARM::STRi12: case ARM::STRH: case ARM::STRBi12:
  case ARM::t2LDRi12: case ARM::t2LDRi8:
  case ARM::t2STRi12: case ARM::t2STRi8:
  case ARM::VLDRS: case ARM::VLDRD:
  case ARM::VSTRS: case ARM::VSTRD:
  case ARM::tSTRspi: case ARM::tLDRspi:
    break;
  default:
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 641-649
```cpp
  // Without a virtual base register, if the function has variable sized
  // objects, all fixed-size local references will be via the frame pointer,
  // Approximate the offset and see if it's legal for the instruction.
  // Note that the incoming offset is based on the SP value at function entry,
  // so it'll be negative.
  MachineFunction &MF = *MI->getParent()->getParent();
  const ARMFrameLowering *TFI = getFrameLowering(MF);
  MachineFrameInfo &MFI = MF.getFrameInfo();
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 651-665
```cpp
  // Estimate an offset from the frame pointer.
  // Conservatively assume all callee-saved registers get pushed. R4-R6
  // will be earlier than the FP, so we ignore those.
  // R7, LR
  int64_t FPOffset = Offset - 8;
  // ARM and Thumb2 functions also need to consider R8-R11 and D8-D15
  if (!AFI->isThumbFunction() || !AFI->isThumb1OnlyFunction())
    FPOffset -= 80;
  // Estimate an offset from the stack pointer.
  // The incoming offset is relating to the SP at the start of the function,
  // but when we access the local it'll be relative to the SP after local
  // allocation, so adjust our SP-relative offset by that allocation size.
  Offset += MFI.getLocalFrameSize();
  // Assume that we'll have at least some spill slots allocated.
  // FIXME: This is a total SWAG number. We should run some statistics
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 666-667
```cpp
  //        and pick a real one.
  Offset += 128; // 128 bytes of spill slots
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 669-683
```cpp
  // If there's a frame pointer and the addressing mode allows it, try using it.
  // The FP is only available if there is no dynamic realignment. We
  // don't know for sure yet whether we'll need that, so we guess based
  // on whether there are any local variables that would trigger it.
  if (TFI->hasFP(MF) &&
      !((MFI.getLocalFrameMaxAlign() > TFI->getStackAlign()) &&
        canRealignStack(MF))) {
    if (isFrameOffsetLegal(MI, getFrameRegister(MF), FPOffset))
      return false;
  }
  // If we can reference via the stack pointer, try that.
  // FIXME: This (and the code that resolves the references) can be improved
  //        to only disallow SP relative references in the live range of
  //        the VLA(s). In practice, it's unclear how much difference that
  //        would make, but it may be worth doing.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 684-685
```cpp
  if (!MFI.hasVarSizedObjects() && isFrameOffsetLegal(MI, ARM::SP, Offset))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 687-689
```cpp
  // The offset likely isn't legal, we want to allocate a virtual base register.
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 691-699
```cpp
/// materializeFrameBaseRegister - Insert defining instruction(s) for BaseReg to
/// be a pointer to FrameIdx at the beginning of the basic block.
Register
ARMBaseRegisterInfo::materializeFrameBaseRegister(MachineBasicBlock *MBB,
                                                  int FrameIdx,
                                                  int64_t Offset) const {
  ARMFunctionInfo *AFI = MBB->getParent()->getInfo<ARMFunctionInfo>();
  unsigned ADDriOpc = !AFI->isThumbFunction() ? ARM::ADDri :
    (AFI->isThumb1OnlyFunction() ? ARM::tADDframe : ARM::t2ADDri);
```
- EN: Implements `ARMBaseRegisterInfo::materializeFrameBaseRegister`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::materializeFrameBaseRegister`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 701-704
```cpp
  MachineBasicBlock::iterator Ins = MBB->begin();
  DebugLoc DL;                  // Defaults to "unknown"
  if (Ins != MBB->end())
    DL = Ins->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 706-711
```cpp
  const MachineFunction &MF = *MBB->getParent();
  MachineRegisterInfo &MRI = MBB->getParent()->getRegInfo();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  const MCInstrDesc &MCID = TII.get(ADDriOpc);
  Register BaseReg = MRI.createVirtualRegister(&ARM::GPRRegClass);
  MRI.constrainRegClass(BaseReg, TII.getRegClass(MCID, 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-714
```cpp
  MachineInstrBuilder MIB = BuildMI(*MBB, Ins, DL, MCID, BaseReg)
    .addFrameIndex(FrameIdx).addImm(Offset);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 716-717
```cpp
  if (!AFI->isThumb1OnlyFunction())
    MIB.add(predOps(ARMCC::AL)).add(condCodeOp());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 719-720
```cpp
  return BaseReg;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 722-730
```cpp
void ARMBaseRegisterInfo::resolveFrameIndex(MachineInstr &MI, Register BaseReg,
                                            int64_t Offset) const {
  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  const ARMBaseInstrInfo &TII =
      *static_cast<const ARMBaseInstrInfo *>(MF.getSubtarget().getInstrInfo());
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  int Off = Offset; // ARM doesn't need the general 64-bit offsets
  unsigned i = 0;
```
- EN: Implements `ARMBaseRegisterInfo::resolveFrameIndex`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::resolveFrameIndex`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 732-733
```cpp
  assert(!AFI->isThumb1OnlyFunction() &&
         "This resolveFrameIndex does not support Thumb1!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 735-748
```cpp
  while (!MI.getOperand(i).isFI()) {
    ++i;
    assert(i < MI.getNumOperands() && "Instr doesn't have FrameIndex operand!");
  }
  bool Done = false;
  if (!AFI->isThumbFunction())
    Done = rewriteARMFrameIndex(MI, i, BaseReg, Off, TII);
  else {
    assert(AFI->isThumb2Function());
    Done = rewriteT2FrameIndex(MI, i, BaseReg, Off, TII, this);
  }
  assert(Done && "Unable to resolve frame index!");
  (void)Done;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 750-757
```cpp
bool ARMBaseRegisterInfo::isFrameOffsetLegal(const MachineInstr *MI,
                                             Register BaseReg,
                                             int64_t Offset) const {
  const MCInstrDesc &Desc = MI->getDesc();
  unsigned AddrMode = (Desc.TSFlags & ARMII::AddrModeMask);
  unsigned i = 0;
  for (; !MI->getOperand(i).isFI(); ++i)
    assert(i+1 < MI->getNumOperands() && "Instr doesn't have FrameIndex operand!");
```
- EN: Implements `ARMBaseRegisterInfo::isFrameOffsetLegal`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::isFrameOffsetLegal`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 759-761
```cpp
  // AddrMode4 and AddrMode6 cannot handle any offset.
  if (AddrMode == ARMII::AddrMode4 || AddrMode == ARMII::AddrMode6)
    return Offset == 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 763-777
```cpp
  unsigned NumBits = 0;
  unsigned Scale = 1;
  bool isSigned = true;
  switch (AddrMode) {
  case ARMII::AddrModeT2_i8:
  case ARMII::AddrModeT2_i8pos:
  case ARMII::AddrModeT2_i8neg:
  case ARMII::AddrModeT2_i12:
    // i8 supports only negative, and i12 supports only positive, so
    // based on Offset sign, consider the appropriate instruction
    Scale = 1;
    if (Offset < 0) {
      NumBits = 8;
      Offset = -Offset;
    } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 778-792
```cpp
      NumBits = 12;
    }
    break;
  case ARMII::AddrMode5:
    // VFP address mode.
    NumBits = 8;
    Scale = 4;
    break;
  case ARMII::AddrMode_i12:
  case ARMII::AddrMode2:
    NumBits = 12;
    break;
  case ARMII::AddrMode3:
    NumBits = 8;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 793-800
```cpp
  case ARMII::AddrModeT1_s:
    NumBits = (BaseReg == ARM::SP ? 8 : 5);
    Scale = 4;
    isSigned = false;
    break;
  default:
    llvm_unreachable("Unsupported addressing mode!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 802-806
```cpp
  Offset += getFrameIndexInstrOffset(MI, i);
  // Make sure the offset is encodable for instructions that scale the
  // immediate.
  if ((Offset & (Scale-1)) != 0)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 808-809
```cpp
  if (isSigned && Offset < 0)
    Offset = -Offset;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 811-813
```cpp
  unsigned Mask = (1 << NumBits) - 1;
  if ((unsigned)Offset <= Mask * Scale)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 815-816
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 818-832
```cpp
bool
ARMBaseRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                         int SPAdj, unsigned FIOperandNum,
                                         RegScavenger *RS) const {
  MachineInstr &MI = *II;
  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  const ARMBaseInstrInfo &TII =
      *static_cast<const ARMBaseInstrInfo *>(MF.getSubtarget().getInstrInfo());
  const ARMFrameLowering *TFI = getFrameLowering(MF);
  ARMFunctionInfo *AFI = MF.getInfo<ARMFunctionInfo>();
  assert(!AFI->isThumb1OnlyFunction() &&
         "This eliminateFrameIndex does not support Thumb1!");
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
  Register FrameReg;
```
- EN: Implements `ARMBaseRegisterInfo::eliminateFrameIndex`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseRegisterInfo::eliminateFrameIndex`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 834-834
```cpp
  int Offset = TFI->ResolveFrameIndexReference(MF, FrameIndex, FrameReg, SPAdj);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 836-849
```cpp
  // PEI::scavengeFrameVirtualRegs() cannot accurately track SPAdj because the
  // call frame setup/destroy instructions have already been eliminated.  That
  // means the stack pointer cannot be used to access the emergency spill slot
  // when !hasReservedCallFrame().
#ifndef NDEBUG
  if (RS && FrameReg == ARM::SP && RS->isScavengingFrameIndex(FrameIndex)){
    assert(TFI->hasReservedCallFrame(MF) &&
           "Cannot use SP to access the emergency spill slot in "
           "functions without a reserved call frame");
    assert(!MF.getFrameInfo().hasVarSizedObjects() &&
           "Cannot use SP to access the emergency spill slot in "
           "functions with variable sized frame objects");
  }
#endif // NDEBUG
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 851-851
```cpp
  assert(!MI.isDebugValue() && "DBG_VALUEs should be handled in target-independent code");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 853-862
```cpp
  // Modify MI as necessary to handle as much of 'Offset' as possible
  bool Done = false;
  if (!AFI->isThumbFunction())
    Done = rewriteARMFrameIndex(MI, FIOperandNum, FrameReg, Offset, TII);
  else {
    assert(AFI->isThumb2Function());
    Done = rewriteT2FrameIndex(MI, FIOperandNum, FrameReg, Offset, TII, this);
  }
  if (Done)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 864-875
```cpp
  // If we get here, the immediate doesn't fit into the instruction.  We folded
  // as much as possible above, handle the rest, providing a register that is
  // SP+LargeImm.
  assert(
      (Offset ||
       (MI.getDesc().TSFlags & ARMII::AddrModeMask) == ARMII::AddrMode4 ||
       (MI.getDesc().TSFlags & ARMII::AddrModeMask) == ARMII::AddrMode6 ||
       (MI.getDesc().TSFlags & ARMII::AddrModeMask) == ARMII::AddrModeT2_i7 ||
       (MI.getDesc().TSFlags & ARMII::AddrModeMask) == ARMII::AddrModeT2_i7s2 ||
       (MI.getDesc().TSFlags & ARMII::AddrModeMask) ==
           ARMII::AddrModeT2_i7s4) &&
      "This code isn't needed if offset already handled!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 877-881
```cpp
  unsigned ScratchReg = 0;
  int PIdx = MI.findFirstPredOperandIdx();
  ARMCC::CondCodes Pred = (PIdx == -1)
    ? ARMCC::AL : (ARMCC::CondCodes)MI.getOperand(PIdx).getImm();
  Register PredReg = (PIdx == -1) ? Register() : MI.getOperand(PIdx+1).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 883-884
```cpp
  const MCInstrDesc &MCID = MI.getDesc();
  const TargetRegisterClass *RegClass = TII.getRegClass(MCID, FIOperandNum);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 886-900
```cpp
  if (Offset == 0 && (FrameReg.isVirtual() || RegClass->contains(FrameReg)))
    // Must be addrmode4/6.
    MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, false, false, false);
  else {
    ScratchReg = MF.getRegInfo().createVirtualRegister(RegClass);
    if (!AFI->isThumbFunction())
      emitARMRegPlusImmediate(MBB, II, MI.getDebugLoc(), ScratchReg, FrameReg,
                              Offset, Pred, PredReg, TII);
    else {
      assert(AFI->isThumb2Function());
      emitT2RegPlusImmediate(MBB, II, MI.getDebugLoc(), ScratchReg, FrameReg,
                             Offset, Pred, PredReg, TII);
    }
    // Update the original instruction to use the scratch register.
    MI.getOperand(FIOperandNum).ChangeToRegister(ScratchReg, false, false,true);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 901-903
```cpp
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 905-919
```cpp
bool ARMBaseRegisterInfo::shouldCoalesce(MachineInstr *MI,
                                  const TargetRegisterClass *SrcRC,
                                  unsigned SubReg,
                                  const TargetRegisterClass *DstRC,
                                  unsigned DstSubReg,
                                  const TargetRegisterClass *NewRC,
                                  LiveIntervals &LIS) const {
  auto MBB = MI->getParent();
  auto MF = MBB->getParent();
  const MachineRegisterInfo &MRI = MF->getRegInfo();
  // If not copying into a sub-register this should be ok because we shouldn't
  // need to split the reg.
  if (!DstSubReg)
    return true;
  // Small registers don't frequently cause a problem, so we can coalesce them.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 920-922
```cpp
  if (getRegSizeInBits(*NewRC) < 256 && getRegSizeInBits(*DstRC) < 256 &&
      getRegSizeInBits(*SrcRC) < 256)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 924-935
```cpp
  auto NewRCWeight =
              MRI.getTargetRegisterInfo()->getRegClassWeight(NewRC);
  auto SrcRCWeight =
              MRI.getTargetRegisterInfo()->getRegClassWeight(SrcRC);
  auto DstRCWeight =
              MRI.getTargetRegisterInfo()->getRegClassWeight(DstRC);
  // If the source register class is more expensive than the destination, the
  // coalescing is probably profitable.
  if (SrcRCWeight.RegWeight > NewRCWeight.RegWeight)
    return true;
  if (DstRCWeight.RegWeight > NewRCWeight.RegWeight)
    return true;
```
- EN: Declares `is`, packaging target-specific state and APIs around `ARMBaseRegisterInfo`.
- CN: 这里声明 `is`，把与 `ARMBaseRegisterInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 937-942
```cpp
  // If the register allocator isn't constrained, we can always allow coalescing
  // unfortunately we don't know yet if we will be constrained.
  // The goal of this heuristic is to restrict how many expensive registers
  // we allow to coalesce in a given basic block.
  auto AFI = MF->getInfo<ARMFunctionInfo>();
  auto It = AFI->getCoalescedWeight(MBB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 944-947
```cpp
  LLVM_DEBUG(dbgs() << "\tARM::shouldCoalesce - Coalesced Weight: "
                    << It->second << "\n");
  LLVM_DEBUG(dbgs() << "\tARM::shouldCoalesce - Reg Weight: "
                    << NewRCWeight.RegWeight << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 949-962
```cpp
  // This number is the largest round number that which meets the criteria:
  //  (1) addresses PR18825
  //  (2) generates better code in some test cases (like vldm-shed-a9.ll)
  //  (3) Doesn't regress any test cases (in-tree, test-suite, and SPEC)
  // In practice the SizeMultiplier will only factor in for straight line code
  // that uses a lot of NEON vectors, which isn't terribly common.
  unsigned SizeMultiplier = MBB->size()/100;
  SizeMultiplier = SizeMultiplier ? SizeMultiplier : 1;
  if (It->second < NewRCWeight.WeightLimit * SizeMultiplier) {
    It->second += NewRCWeight.RegWeight;
    return true;
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

## Key Concepts / 关键概念

- EN: Primary role: shared register definitions and register-management helpers.
  - CN: 核心职责：共享寄存器定义与寄存器管理辅助逻辑。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMBaseRegisterInfo.h`, `ARM.h`, `ARMBaseInstrInfo.h`, `ARMFrameLowering.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h` ... (+1 more).
  - CN: 后端本地头文件：`ARMBaseRegisterInfo.h`, `ARM.h`, `ARMBaseInstrInfo.h`, `ARMFrameLowering.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h` ... (+1 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/BitVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h` ... (+18 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h` ... (+18 more)。
- EN: Standard/system headers: `cassert`, `utility`.
  - CN: 标准库/系统头文件：`cassert`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
