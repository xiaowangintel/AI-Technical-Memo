# MipsCallLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsCallLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsCallLowering` for the Mips backend, focusing on GlobalISel call lowering.
- 用途 (CN): 实现 Mips 后端中的 `MipsCallLowering`，重点处理GlobalISel 调用降级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//===- MipsCallLowering.cpp -------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file implements the lowering of LLVM calls to machine code calls for
/// GlobalISel.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 15-21
```cpp
#include "MipsCallLowering.h"
#include "MipsCCState.h"
#include "MipsMachineFunction.h"
#include "MipsTargetMachine.h"
#include "llvm/CodeGen/Analysis.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 25-26
```cpp
MipsCallLowering::MipsCallLowering(const MipsTargetLowering &TLI)
    : CallLowering(&TLI) {}
```
- EN: Implements `MipsCallLowering::MipsCallLowering`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCallLowering::MipsCallLowering`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 28-30
```cpp
namespace {
class MipsIncomingValueHandler : public CallLowering::IncomingValueHandler {
  const MipsSubtarget &STI;
```
- EN: Declares `MipsIncomingValueHandler`, packaging target-specific state and APIs around `MipsCallLowering`.
- CN: 这里声明 `MipsIncomingValueHandler`，把与 `MipsCallLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 32-36
```cpp
public:
  MipsIncomingValueHandler(MachineIRBuilder &MIRBuilder,
                           MachineRegisterInfo &MRI)
      : IncomingValueHandler(MIRBuilder, MRI),
        STI(MIRBuilder.getMF().getSubtarget<MipsSubtarget>()) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 38-41
```cpp
private:
  void assignValueToReg(Register ValVReg, Register PhysReg,
                        const CCValAssign &VA,
                        ISD::ArgFlagsTy Flags = {}) override;
```
- EN: Implements `assignValueToReg`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `assignValueToReg`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-48
```cpp
  Register getStackAddress(uint64_t Size, int64_t Offset,
                           MachinePointerInfo &MPO,
                           ISD::ArgFlagsTy Flags) override;
  void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
                            const MachinePointerInfo &MPO,
                            const CCValAssign &VA) override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 50-52
```cpp
  unsigned assignCustomValue(CallLowering::ArgInfo &Arg,
                             ArrayRef<CCValAssign> VAs,
                             std::function<void()> *Thunk = nullptr) override;
```
- EN: Declares `assignCustomValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `assignCustomValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 54-58
```cpp
  virtual void markPhysRegUsed(unsigned PhysReg) {
    MIRBuilder.getMRI()->addLiveIn(PhysReg);
    MIRBuilder.getMBB().addLiveIn(PhysReg);
  }
};
```
- EN: Implements `markPhysRegUsed`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `markPhysRegUsed`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 60-64
```cpp
class CallReturnHandler : public MipsIncomingValueHandler {
public:
  CallReturnHandler(MachineIRBuilder &MIRBuilder, MachineRegisterInfo &MRI,
                    MachineInstrBuilder &MIB)
      : MipsIncomingValueHandler(MIRBuilder, MRI), MIB(MIB) {}
```
- EN: Declares `CallReturnHandler`, packaging target-specific state and APIs around `MipsCallLowering`.
- CN: 这里声明 `CallReturnHandler`，把与 `MipsCallLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 66-69
```cpp
private:
  void markPhysRegUsed(unsigned PhysReg) override {
    MIB.addDef(PhysReg, RegState::Implicit);
  }
```
- EN: Implements `markPhysRegUsed`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `markPhysRegUsed`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-72
```cpp
  MachineInstrBuilder &MIB;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 74-74
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 76-82
```cpp
void MipsIncomingValueHandler::assignValueToReg(Register ValVReg,
                                                Register PhysReg,
                                                const CCValAssign &VA,
                                                ISD::ArgFlagsTy Flags) {
  markPhysRegUsed(PhysReg);
  IncomingValueHandler::assignValueToReg(ValVReg, PhysReg, VA);
}
```
- EN: Implements `MipsIncomingValueHandler::assignValueToReg`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsIncomingValueHandler::assignValueToReg`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 84-87
```cpp
Register MipsIncomingValueHandler::getStackAddress(uint64_t Size,
                                                   int64_t Offset,
                                                   MachinePointerInfo &MPO,
                                                   ISD::ArgFlagsTy Flags) {
```
- EN: Implements `MipsIncomingValueHandler::getStackAddress`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsIncomingValueHandler::getStackAddress`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-90
```cpp
  MachineFunction &MF = MIRBuilder.getMF();
  MachineFrameInfo &MFI = MF.getFrameInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 92-94
```cpp
  // FIXME: This should only be immutable for non-byval memory arguments.
  int FI = MFI.CreateFixedObject(Size, Offset, true);
  MPO = MachinePointerInfo::getFixedStack(MIRBuilder.getMF(), FI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 96-97
```cpp
  return MIRBuilder.buildFrameIndex(LLT::pointer(0, 32), FI).getReg(0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 99-106
```cpp
void MipsIncomingValueHandler::assignValueToAddress(
    Register ValVReg, Register Addr, LLT MemTy, const MachinePointerInfo &MPO,
    const CCValAssign &VA) {
  MachineFunction &MF = MIRBuilder.getMF();
  auto MMO = MF.getMachineMemOperand(MPO, MachineMemOperand::MOLoad, MemTy,
                                     inferAlignFromPtrInfo(MF, MPO));
  MIRBuilder.buildLoad(ValVReg, Addr, *MMO);
}
```
- EN: Implements `MipsIncomingValueHandler::assignValueToAddress`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsIncomingValueHandler::assignValueToAddress`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 108-117
```cpp
/// Handle cases when f64 is split into 2 32-bit GPRs. This is a custom
/// assignment because generic code assumes getNumRegistersForCallingConv is
/// accurate. In this case it is not because the type/number are context
/// dependent on other arguments.
unsigned
MipsIncomingValueHandler::assignCustomValue(CallLowering::ArgInfo &Arg,
                                            ArrayRef<CCValAssign> VAs,
                                            std::function<void()> *Thunk) {
  const CCValAssign &VALo = VAs[0];
  const CCValAssign &VAHi = VAs[1];
```
- EN: Implements `MipsIncomingValueHandler::assignCustomValue`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsIncomingValueHandler::assignCustomValue`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 119-121
```cpp
  assert(VALo.getLocVT() == MVT::i32 && VAHi.getLocVT() == MVT::i32 &&
         VALo.getValVT() == MVT::f64 && VAHi.getValVT() == MVT::f64 &&
         "unexpected custom value");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 123-126
```cpp
  auto CopyLo = MIRBuilder.buildCopy(LLT::scalar(32), VALo.getLocReg());
  auto CopyHi = MIRBuilder.buildCopy(LLT::scalar(32), VAHi.getLocReg());
  if (!STI.isLittle())
    std::swap(CopyLo, CopyHi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 128-130
```cpp
  Arg.OrigRegs.assign(Arg.Regs.begin(), Arg.Regs.end());
  Arg.Regs = { CopyLo.getReg(0), CopyHi.getReg(0) };
  MIRBuilder.buildMergeLikeInstr(Arg.OrigRegs[0], {CopyLo, CopyHi});
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-135
```cpp
  markPhysRegUsed(VALo.getLocReg());
  markPhysRegUsed(VAHi.getLocReg());
  return 2;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 137-139
```cpp
namespace {
class MipsOutgoingValueHandler : public CallLowering::OutgoingValueHandler {
  const MipsSubtarget &STI;
```
- EN: Declares `MipsOutgoingValueHandler`, packaging target-specific state and APIs around `MipsCallLowering`.
- CN: 这里声明 `MipsOutgoingValueHandler`，把与 `MipsCallLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 141-145
```cpp
public:
  MipsOutgoingValueHandler(MachineIRBuilder &MIRBuilder,
                           MachineRegisterInfo &MRI, MachineInstrBuilder &MIB)
      : OutgoingValueHandler(MIRBuilder, MRI),
        STI(MIRBuilder.getMF().getSubtarget<MipsSubtarget>()), MIB(MIB) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 147-150
```cpp
private:
  void assignValueToReg(Register ValVReg, Register PhysReg,
                        const CCValAssign &VA,
                        ISD::ArgFlagsTy Flags = {}) override;
```
- EN: Implements `assignValueToReg`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `assignValueToReg`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-154
```cpp
  Register getStackAddress(uint64_t Size, int64_t Offset,
                           MachinePointerInfo &MPO,
                           ISD::ArgFlagsTy Flags) override;
```
- EN: Declares `getStackAddress`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getStackAddress`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 156-161
```cpp
  void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
                            const MachinePointerInfo &MPO,
                            const CCValAssign &VA) override;
  unsigned assignCustomValue(CallLowering::ArgInfo &Arg,
                             ArrayRef<CCValAssign> VAs,
                             std::function<void()> *Thunk) override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-165
```cpp
  MachineInstrBuilder &MIB;
};
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 167-174
```cpp
void MipsOutgoingValueHandler::assignValueToReg(Register ValVReg,
                                                Register PhysReg,
                                                const CCValAssign &VA,
                                                ISD::ArgFlagsTy Flags) {
  Register ExtReg = extendRegister(ValVReg, VA);
  MIRBuilder.buildCopy(PhysReg, ExtReg);
  MIB.addUse(PhysReg, RegState::Implicit);
}
```
- EN: Implements `MipsOutgoingValueHandler::assignValueToReg`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsOutgoingValueHandler::assignValueToReg`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 176-181
```cpp
Register MipsOutgoingValueHandler::getStackAddress(uint64_t Size,
                                                   int64_t Offset,
                                                   MachinePointerInfo &MPO,
                                                   ISD::ArgFlagsTy Flags) {
  MachineFunction &MF = MIRBuilder.getMF();
  MPO = MachinePointerInfo::getStack(MF, Offset);
```
- EN: Implements `MipsOutgoingValueHandler::getStackAddress`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsOutgoingValueHandler::getStackAddress`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 183-185
```cpp
  LLT p0 = LLT::pointer(0, 32);
  LLT s32 = LLT::scalar(32);
  auto SPReg = MIRBuilder.buildCopy(p0, Register(Mips::SP));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 187-190
```cpp
  auto OffsetReg = MIRBuilder.buildConstant(s32, Offset);
  auto AddrReg = MIRBuilder.buildPtrAdd(p0, SPReg, OffsetReg);
  return AddrReg.getReg(0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 192-196
```cpp
void MipsOutgoingValueHandler::assignValueToAddress(
    Register ValVReg, Register Addr, LLT MemTy, const MachinePointerInfo &MPO,
    const CCValAssign &VA) {
  MachineFunction &MF = MIRBuilder.getMF();
  uint64_t LocMemOffset = VA.getLocMemOffset();
```
- EN: Implements `MipsOutgoingValueHandler::assignValueToAddress`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsOutgoingValueHandler::assignValueToAddress`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 198-200
```cpp
  auto MMO = MF.getMachineMemOperand(
      MPO, MachineMemOperand::MOStore, MemTy,
      commonAlignment(STI.getStackAlignment(), LocMemOffset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 202-204
```cpp
  Register ExtReg = extendRegister(ValVReg, VA);
  MIRBuilder.buildStore(ExtReg, Addr, *MMO);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 206-211
```cpp
unsigned
MipsOutgoingValueHandler::assignCustomValue(CallLowering::ArgInfo &Arg,
                                            ArrayRef<CCValAssign> VAs,
                                            std::function<void()> *Thunk) {
  const CCValAssign &VALo = VAs[0];
  const CCValAssign &VAHi = VAs[1];
```
- EN: Implements `MipsOutgoingValueHandler::assignCustomValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsOutgoingValueHandler::assignCustomValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 213-215
```cpp
  assert(VALo.getLocVT() == MVT::i32 && VAHi.getLocVT() == MVT::i32 &&
         VALo.getValVT() == MVT::f64 && VAHi.getValVT() == MVT::f64 &&
         "unexpected custom value");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 217-220
```cpp
  auto Unmerge =
      MIRBuilder.buildUnmerge({LLT::scalar(32), LLT::scalar(32)}, Arg.Regs[0]);
  Register Lo = Unmerge.getReg(0);
  Register Hi = Unmerge.getReg(1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 222-225
```cpp
  Arg.OrigRegs.assign(Arg.Regs.begin(), Arg.Regs.end());
  Arg.Regs = { Lo, Hi };
  if (!STI.isLittle())
    std::swap(Lo, Hi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-239
```cpp
  // If we can return a thunk, just include the register copies. The unmerge can
  // be emitted earlier.
  if (Thunk) {
    *Thunk = [=]() {
      MIRBuilder.buildCopy(VALo.getLocReg(), Lo);
      MIRBuilder.buildCopy(VAHi.getLocReg(), Hi);
    };
    return 2;
  }
  MIRBuilder.buildCopy(VALo.getLocReg(), Lo);
  MIRBuilder.buildCopy(VAHi.getLocReg(), Hi);
  return 2;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 241-249
```cpp
static bool isSupportedArgumentType(Type *T) {
  if (T->isIntegerTy())
    return true;
  if (T->isPointerTy())
    return true;
  if (T->isFloatingPointTy())
    return true;
  return false;
}
```
- EN: Implements `isSupportedArgumentType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSupportedArgumentType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 251-261
```cpp
static bool isSupportedReturnType(Type *T) {
  if (T->isIntegerTy())
    return true;
  if (T->isPointerTy())
    return true;
  if (T->isFloatingPointTy())
    return true;
  if (T->isAggregateType())
    return true;
  return false;
}
```
- EN: Implements `isSupportedReturnType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSupportedReturnType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 263-265
```cpp
bool MipsCallLowering::lowerReturn(MachineIRBuilder &MIRBuilder,
                                   const Value *Val, ArrayRef<Register> VRegs,
                                   FunctionLoweringInfo &FLI) const {
```
- EN: Implements `MipsCallLowering::lowerReturn`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCallLowering::lowerReturn`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 267-267
```cpp
  MachineInstrBuilder Ret = MIRBuilder.buildInstrNoInsert(Mips::RetRA);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 269-270
```cpp
  if (Val != nullptr && !isSupportedReturnType(Val->getType()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 272-276
```cpp
  if (!VRegs.empty()) {
    MachineFunction &MF = MIRBuilder.getMF();
    const Function &F = MF.getFunction();
    const DataLayout &DL = MF.getDataLayout();
    const MipsTargetLowering &TLI = *getTLI<MipsTargetLowering>();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 278-278
```cpp
    SmallVector<ArgInfo, 8> RetInfos;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 280-282
```cpp
    ArgInfo ArgRetInfo(VRegs, *Val, 0);
    setArgFlags(ArgRetInfo, AttributeList::ReturnIndex, DL, F);
    splitToValueTypes(ArgRetInfo, RetInfos, DL, F.getCallingConv());
```
- EN: Declares `ArgRetInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ArgRetInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 284-284
```cpp
    SmallVector<CCValAssign, 16> ArgLocs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 286-287
```cpp
    MipsCCState CCInfo(F.getCallingConv(), F.isVarArg(), MF, ArgLocs,
                       F.getContext());
```
- EN: Declares `CCInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 289-290
```cpp
    MipsOutgoingValueHandler RetHandler(MIRBuilder, MF.getRegInfo(), Ret);
    OutgoingValueAssigner Assigner(TLI.CCAssignFnForReturn());
```
- EN: Declares `RetHandler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `RetHandler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 292-293
```cpp
    if (!determineAssignments(Assigner, RetInfos, CCInfo))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 295-297
```cpp
    if (!handleAssignments(RetHandler, RetInfos, CCInfo, ArgLocs, MIRBuilder))
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 299-301
```cpp
  MIRBuilder.insertInstr(Ret);
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 303-306
```cpp
bool MipsCallLowering::lowerFormalArguments(MachineIRBuilder &MIRBuilder,
                                            const Function &F,
                                            ArrayRef<ArrayRef<Register>> VRegs,
                                            FunctionLoweringInfo &FLI) const {
```
- EN: Implements `MipsCallLowering::lowerFormalArguments`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCallLowering::lowerFormalArguments`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 308-310
```cpp
  // Quick exit if there aren't any args.
  if (F.arg_empty())
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 312-315
```cpp
  for (auto &Arg : F.args()) {
    if (!isSupportedArgumentType(Arg.getType()))
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 317-319
```cpp
  MachineFunction &MF = MIRBuilder.getMF();
  const DataLayout &DL = MF.getDataLayout();
  const MipsTargetLowering &TLI = *getTLI<MipsTargetLowering>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 321-325
```cpp
  SmallVector<ArgInfo, 8> ArgInfos;
  unsigned i = 0;
  for (auto &Arg : F.args()) {
    ArgInfo AInfo(VRegs[i], Arg, i);
    setArgFlags(AInfo, i + AttributeList::FirstArgIndex, DL, F);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 327-329
```cpp
    splitToValueTypes(AInfo, ArgInfos, DL, F.getCallingConv());
    ++i;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 331-333
```cpp
  SmallVector<CCValAssign, 16> ArgLocs;
  MipsCCState CCInfo(F.getCallingConv(), F.isVarArg(), MF, ArgLocs,
                     F.getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 335-339
```cpp
  const MipsTargetMachine &TM =
      static_cast<const MipsTargetMachine &>(MF.getTarget());
  const MipsABIInfo &ABI = TM.getABI();
  CCInfo.AllocateStack(ABI.GetCalleeAllocdArgSizeInBytes(F.getCallingConv()),
                       Align(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 341-343
```cpp
  IncomingValueAssigner Assigner(TLI.CCAssignFnForCall());
  if (!determineAssignments(Assigner, ArgInfos, CCInfo))
    return false;
```
- EN: Declares `Assigner`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Assigner`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 345-347
```cpp
  MipsIncomingValueHandler Handler(MIRBuilder, MF.getRegInfo());
  if (!handleAssignments(Handler, ArgInfos, CCInfo, ArgLocs, MIRBuilder))
    return false;
```
- EN: Declares `Handler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Handler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 349-352
```cpp
  if (F.isVarArg()) {
    ArrayRef<MCPhysReg> ArgRegs =
        ABI.getVarArgRegs(MF.getSubtarget<MipsSubtarget>().isGP64bit());
    unsigned Idx = CCInfo.getFirstUnallocated(ArgRegs);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 354-362
```cpp
    int VaArgOffset;
    unsigned RegSize = 4;
    if (ArgRegs.size() == Idx)
      VaArgOffset = alignTo(CCInfo.getStackSize(), RegSize);
    else {
      VaArgOffset =
          (int)ABI.GetCalleeAllocdArgSizeInBytes(CCInfo.getCallingConv()) -
          (int)(RegSize * (ArgRegs.size() - Idx));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 364-366
```cpp
    MachineFrameInfo &MFI = MF.getFrameInfo();
    int FI = MFI.CreateFixedObject(RegSize, VaArgOffset, true);
    MF.getInfo<MipsFunctionInfo>()->setVarArgsFrameIndex(FI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 368-374
```cpp
    for (unsigned I = Idx; I < ArgRegs.size(); ++I, VaArgOffset += RegSize) {
      MIRBuilder.getMBB().addLiveIn(ArgRegs[I]);
      LLT RegTy = LLT::scalar(RegSize * 8);
      MachineInstrBuilder Copy =
          MIRBuilder.buildCopy(RegTy, Register(ArgRegs[I]));
      FI = MFI.CreateFixedObject(RegSize, VaArgOffset, true);
      MachinePointerInfo MPO = MachinePointerInfo::getFixedStack(MF, FI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 376-382
```cpp
      const LLT PtrTy = LLT::pointer(MPO.getAddrSpace(), 32);
      auto FrameIndex = MIRBuilder.buildFrameIndex(PtrTy, FI);
      MachineMemOperand *MMO = MF.getMachineMemOperand(
          MPO, MachineMemOperand::MOStore, RegTy, Align(RegSize));
      MIRBuilder.buildStore(Copy, FrameIndex, *MMO);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 384-385
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 387-388
```cpp
bool MipsCallLowering::lowerCall(MachineIRBuilder &MIRBuilder,
                                 CallLoweringInfo &Info) const {
```
- EN: Implements `MipsCallLowering::lowerCall`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCallLowering::lowerCall`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 390-391
```cpp
  if (Info.CallConv != CallingConv::C)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 393-400
```cpp
  for (auto &Arg : Info.OrigArgs) {
    if (!isSupportedArgumentType(Arg.Ty))
      return false;
    if (Arg.Flags[0].isByVal())
      return false;
    if (Arg.Flags[0].isSRet() && !Arg.Ty->isPointerTy())
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 402-403
```cpp
  if (!Info.OrigRet.Ty->isVoidTy() && !isSupportedReturnType(Info.OrigRet.Ty))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 405-411
```cpp
  MachineFunction &MF = MIRBuilder.getMF();
  const Function &F = MF.getFunction();
  const DataLayout &DL = MF.getDataLayout();
  const MipsTargetLowering &TLI = *getTLI<MipsTargetLowering>();
  const MipsTargetMachine &TM =
      static_cast<const MipsTargetMachine &>(MF.getTarget());
  const MipsABIInfo &ABI = TM.getABI();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 413-414
```cpp
  MachineInstrBuilder CallSeqStart =
      MIRBuilder.buildInstr(Mips::ADJCALLSTACKDOWN);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 416-417
```cpp
  const bool IsCalleeGlobalPIC =
      Info.Callee.isGlobal() && TM.isPositionIndependent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 419-433
```cpp
  MachineInstrBuilder MIB = MIRBuilder.buildInstrNoInsert(
      Info.Callee.isReg() || IsCalleeGlobalPIC ? Mips::JALRPseudo : Mips::JAL);
  MIB.addDef(Mips::SP, RegState::Implicit);
  if (IsCalleeGlobalPIC) {
    Register CalleeReg =
        MF.getRegInfo().createGenericVirtualRegister(LLT::pointer(0, 32));
    MachineInstr *CalleeGlobalValue =
        MIRBuilder.buildGlobalValue(CalleeReg, Info.Callee.getGlobal());
    if (!Info.Callee.getGlobal()->hasLocalLinkage())
      CalleeGlobalValue->getOperand(1).setTargetFlags(MipsII::MO_GOT_CALL);
    MIB.addUse(CalleeReg);
  } else
    MIB.add(Info.Callee);
  const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
  MIB.addRegMask(TRI->getCallPreservedMask(MF, Info.CallConv));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 435-436
```cpp
  TargetLowering::ArgListTy FuncOrigArgs;
  FuncOrigArgs.reserve(Info.OrigArgs.size());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 438-440
```cpp
  SmallVector<ArgInfo, 8> ArgInfos;
  for (auto &Arg : Info.OrigArgs)
    splitToValueTypes(Arg, ArgInfos, DL, Info.CallConv);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 442-447
```cpp
  SmallVector<CCValAssign, 8> ArgLocs;
  bool IsCalleeVarArg = false;
  if (Info.Callee.isGlobal()) {
    const Function *CF = static_cast<const Function *>(Info.Callee.getGlobal());
    IsCalleeVarArg = CF->isVarArg();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 449-452
```cpp
  // FIXME: Should use MipsCCState::getSpecialCallingConvForCallee, but it
  // depends on looking directly at the call target.
  MipsCCState CCInfo(Info.CallConv, IsCalleeVarArg, MF, ArgLocs,
                     F.getContext());
```
- EN: Declares `CCInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 454-455
```cpp
  CCInfo.AllocateStack(ABI.GetCalleeAllocdArgSizeInBytes(Info.CallConv),
                       Align(1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 457-459
```cpp
  OutgoingValueAssigner Assigner(TLI.CCAssignFnForCall());
  if (!determineAssignments(Assigner, ArgInfos, CCInfo))
    return false;
```
- EN: Declares `Assigner`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Assigner`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 461-463
```cpp
  MipsOutgoingValueHandler ArgHandler(MIRBuilder, MF.getRegInfo(), MIB);
  if (!handleAssignments(ArgHandler, ArgInfos, CCInfo, ArgLocs, MIRBuilder))
    return false;
```
- EN: Declares `ArgHandler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ArgHandler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 465-472
```cpp
  unsigned StackSize = CCInfo.getStackSize();
  unsigned StackAlignment = F.getParent()->getOverrideStackAlignment();
  if (!StackAlignment) {
    const TargetFrameLowering *TFL = MF.getSubtarget().getFrameLowering();
    StackAlignment = TFL->getStackAlignment();
  }
  StackSize = alignTo(StackSize, StackAlignment);
  CallSeqStart.addImm(StackSize).addImm(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 474-485
```cpp
  if (IsCalleeGlobalPIC) {
    MIRBuilder.buildCopy(
      Register(Mips::GP),
      MF.getInfo<MipsFunctionInfo>()->getGlobalBaseRegForGlobalISel(MF));
    MIB.addDef(Mips::GP, RegState::Implicit);
  }
  MIRBuilder.insertInstr(MIB);
  if (MIB->getOpcode() == Mips::JALRPseudo) {
    const MipsSubtarget &STI = MIRBuilder.getMF().getSubtarget<MipsSubtarget>();
    MIB.constrainAllUses(MIRBuilder.getTII(), *STI.getRegisterInfo(),
                         *STI.getRegBankInfo());
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 487-488
```cpp
  if (!Info.OrigRet.Ty->isVoidTy()) {
    ArgInfos.clear();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 490-491
```cpp
    CallLowering::splitToValueTypes(Info.OrigRet, ArgInfos, DL,
                                    F.getCallingConv());
```
- EN: Declares `CallLowering::splitToValueTypes`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CallLowering::splitToValueTypes`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 493-495
```cpp
    SmallVector<CCValAssign, 8> ArgLocs;
    IncomingValueAssigner Assigner(TLI.CCAssignFnForReturn());
    CallReturnHandler RetHandler(MIRBuilder, MF.getRegInfo(), MIB);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 497-498
```cpp
    MipsCCState CCInfo(F.getCallingConv(), F.isVarArg(), MF, ArgLocs,
                       F.getContext());
```
- EN: Declares `CCInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CCInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 500-501
```cpp
    if (!determineAssignments(Assigner, ArgInfos, CCInfo))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 503-505
```cpp
    if (!handleAssignments(RetHandler, ArgInfos, CCInfo, ArgLocs, MIRBuilder))
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 507-507
```cpp
  MIRBuilder.buildInstr(Mips::ADJCALLSTACKUP).addImm(StackSize).addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 509-510
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: GlobalISel call lowering.
  - CN: 核心职责：GlobalISel 调用降级。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsCallLowering.h`, `MipsCCState.h`, `MipsMachineFunction.h`, `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`MipsCallLowering.h`, `MipsCCState.h`, `MipsMachineFunction.h`, `MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/Analysis.h`, `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`, `llvm/CodeGen/MachineFrameInfo.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/Analysis.h`, `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`, `llvm/CodeGen/MachineFrameInfo.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
