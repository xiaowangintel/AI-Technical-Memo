# MipsMachineFunction.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsMachineFunction.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsMachineFunction` for the Mips backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 Mips 后端中的 `MipsMachineFunction`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MipsMachineFunctionInfo.cpp - Private data used for Mips ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-18
```cpp
#include "MipsMachineFunction.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "MipsSubtarget.h"
#include "MipsTargetMachine.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/PseudoSourceValue.h"
#include "llvm/CodeGen/PseudoSourceValueManager.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/Support/CommandLine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 20-20
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 22-24
```cpp
static cl::opt<bool>
FixGlobalBaseReg("mips-fix-global-base-reg", cl::Hidden, cl::init(true),
                 cl::desc("Always use $gp as the global base register."));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 26-31
```cpp
MachineFunctionInfo *
MipsFunctionInfo::clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
                        const DenseMap<MachineBasicBlock *, MachineBasicBlock *>
                            &Src2DstMBB) const {
  return DestMF.cloneInfo<MipsFunctionInfo>(*this);
}
```
- EN: Implements `MipsFunctionInfo::clone`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::clone`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 33-33
```cpp
MipsFunctionInfo::~MipsFunctionInfo() = default;
```
- EN: Declares `MipsFunctionInfo::~MipsFunctionInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsFunctionInfo::~MipsFunctionInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 35-37
```cpp
bool MipsFunctionInfo::globalBaseRegSet() const {
  return GlobalBaseReg;
}
```
- EN: Implements `MipsFunctionInfo::globalBaseRegSet`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::globalBaseRegSet`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 39-41
```cpp
static const TargetRegisterClass &getGlobalBaseRegClass(MachineFunction &MF) {
  auto &STI = MF.getSubtarget<MipsSubtarget>();
  auto &TM = static_cast<const MipsTargetMachine &>(MF.getTarget());
```
- EN: Implements `getGlobalBaseRegClass`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getGlobalBaseRegClass`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-44
```cpp
  if (STI.inMips16Mode())
    return Mips::CPU16RegsRegClass;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 46-47
```cpp
  if (STI.inMicroMipsMode())
    return Mips::GPRMM16RegClass;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 49-50
```cpp
  if (TM.getABI().IsN64())
    return Mips::GPR64RegClass;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 52-53
```cpp
  return Mips::GPR32RegClass;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 55-60
```cpp
Register MipsFunctionInfo::getGlobalBaseReg(MachineFunction &MF) {
  if (!GlobalBaseReg)
    GlobalBaseReg =
        MF.getRegInfo().createVirtualRegister(&getGlobalBaseRegClass(MF));
  return GlobalBaseReg;
}
```
- EN: Implements `MipsFunctionInfo::getGlobalBaseReg`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::getGlobalBaseReg`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-68
```cpp
Register MipsFunctionInfo::getGlobalBaseRegForGlobalISel(MachineFunction &MF) {
  if (!GlobalBaseReg) {
    getGlobalBaseReg(MF);
    initGlobalBaseReg(MF);
  }
  return GlobalBaseReg;
}
```
- EN: Implements `MipsFunctionInfo::getGlobalBaseRegForGlobalISel`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::getGlobalBaseRegForGlobalISel`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 70-72
```cpp
void MipsFunctionInfo::initGlobalBaseReg(MachineFunction &MF) {
  if (!GlobalBaseReg)
    return;
```
- EN: Implements `MipsFunctionInfo::initGlobalBaseReg`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::initGlobalBaseReg`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 74-82
```cpp
  MachineBasicBlock &MBB = MF.front();
  MachineBasicBlock::iterator I = MBB.begin();
  MachineRegisterInfo &RegInfo = MF.getRegInfo();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  DebugLoc DL;
  const TargetRegisterClass *RC;
  const MipsABIInfo &ABI =
      static_cast<const MipsTargetMachine &>(MF.getTarget()).getABI();
  RC = (ABI.IsN64()) ? &Mips::GPR64RegClass : &Mips::GPR32RegClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-85
```cpp
  Register V0 = RegInfo.createVirtualRegister(RC);
  Register V1 = RegInfo.createVirtualRegister(RC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 87-89
```cpp
  if (ABI.IsN64()) {
    MF.getRegInfo().addLiveIn(Mips::T9_64);
    MBB.addLiveIn(Mips::T9_64);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 91-102
```cpp
    // lui $v0, %hi(%neg(%gp_rel(fname)))
    // daddu $v1, $v0, $t9
    // daddiu $globalbasereg, $v1, %lo(%neg(%gp_rel(fname)))
    const GlobalValue *FName = &MF.getFunction();
    BuildMI(MBB, I, DL, TII.get(Mips::LUi64), V0)
        .addGlobalAddress(FName, 0, MipsII::MO_GPOFF_HI);
    BuildMI(MBB, I, DL, TII.get(Mips::DADDu), V1).addReg(V0)
        .addReg(Mips::T9_64);
    BuildMI(MBB, I, DL, TII.get(Mips::DADDiu), GlobalBaseReg).addReg(V1)
        .addGlobalAddress(FName, 0, MipsII::MO_GPOFF_LO);
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-114
```cpp
  if (!MF.getTarget().isPositionIndependent()) {
    // Set global register to __gnu_local_gp.
    //
    // lui   $v0, %hi(__gnu_local_gp)
    // addiu $globalbasereg, $v0, %lo(__gnu_local_gp)
    BuildMI(MBB, I, DL, TII.get(Mips::LUi), V0)
        .addExternalSymbol("__gnu_local_gp", MipsII::MO_ABS_HI);
    BuildMI(MBB, I, DL, TII.get(Mips::ADDiu), GlobalBaseReg).addReg(V0)
        .addExternalSymbol("__gnu_local_gp", MipsII::MO_ABS_LO);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 116-117
```cpp
  MF.getRegInfo().addLiveIn(Mips::T9);
  MBB.addLiveIn(Mips::T9);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 119-130
```cpp
  if (ABI.IsN32()) {
    // lui $v0, %hi(%neg(%gp_rel(fname)))
    // addu $v1, $v0, $t9
    // addiu $globalbasereg, $v1, %lo(%neg(%gp_rel(fname)))
    const GlobalValue *FName = &MF.getFunction();
    BuildMI(MBB, I, DL, TII.get(Mips::LUi), V0)
        .addGlobalAddress(FName, 0, MipsII::MO_GPOFF_HI);
    BuildMI(MBB, I, DL, TII.get(Mips::ADDu), V1).addReg(V0).addReg(Mips::T9);
    BuildMI(MBB, I, DL, TII.get(Mips::ADDiu), GlobalBaseReg).addReg(V1)
        .addGlobalAddress(FName, 0, MipsII::MO_GPOFF_LO);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 132-132
```cpp
  assert(ABI.IsO32());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 134-148
```cpp
  // For O32 ABI, the following instruction sequence is emitted to initialize
  // the global base register:
  //
  //  0. lui   $2, %hi(_gp_disp)
  //  1. addiu $2, $2, %lo(_gp_disp)
  //  2. addu  $globalbasereg, $2, $t9
  //
  // We emit only the last instruction here.
  //
  // GNU linker requires that the first two instructions appear at the beginning
  // of a function and no instructions be inserted before or between them.
  // The two instructions are emitted during lowering to MC layer in order to
  // avoid any reordering.
  //
  // Register $2 (Mips::V0) is added to the list of live-in registers to ensure
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 149-155
```cpp
  // the value instruction 1 (addiu) defines is valid when instruction 2 (addu)
  // reads it.
  MF.getRegInfo().addLiveIn(Mips::V0);
  MBB.addLiveIn(Mips::V0);
  BuildMI(MBB, I, DL, TII.get(Mips::ADDu), GlobalBaseReg)
      .addReg(Mips::V0).addReg(Mips::T9);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 157-163
```cpp
void MipsFunctionInfo::createEhDataRegsFI(MachineFunction &MF) {
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
  for (int &I : EhDataRegFI) {
    const TargetRegisterClass &RC =
        static_cast<const MipsTargetMachine &>(MF.getTarget()).getABI().IsN64()
            ? Mips::GPR64RegClass
            : Mips::GPR32RegClass;
```
- EN: Implements `MipsFunctionInfo::createEhDataRegsFI`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::createEhDataRegsFI`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-168
```cpp
    I = MF.getFrameInfo().CreateStackObject(TRI.getSpillSize(RC),
                                            TRI.getSpillAlign(RC), false);
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 170-176
```cpp
void MipsFunctionInfo::createISRRegFI(MachineFunction &MF) {
  // ISRs require spill slots for Status & ErrorPC Coprocessor 0 registers.
  // The current implementation only supports Mips32r2+ not Mips64rX. Status
  // is always 32 bits, ErrorPC is 32 or 64 bits dependent on architecture,
  // however Mips32r2+ is the supported architecture.
  const TargetRegisterClass &RC = Mips::GPR32RegClass;
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
```
- EN: Implements `MipsFunctionInfo::createISRRegFI`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::createISRRegFI`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 178-181
```cpp
  for (int &I : ISRDataRegFI)
    I = MF.getFrameInfo().CreateStackObject(TRI.getSpillSize(RC),
                                            TRI.getSpillAlign(RC), false);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 183-186
```cpp
bool MipsFunctionInfo::isEhDataRegFI(int FI) const {
  return CallsEhReturn && (FI == EhDataRegFI[0] || FI == EhDataRegFI[1]
                        || FI == EhDataRegFI[2] || FI == EhDataRegFI[3]);
}
```
- EN: Implements `MipsFunctionInfo::isEhDataRegFI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::isEhDataRegFI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 188-194
```cpp
bool MipsFunctionInfo::isISRRegFI(int FI) const {
  return IsISR && (FI == ISRDataRegFI[0] || FI == ISRDataRegFI[1]);
}
MachinePointerInfo MipsFunctionInfo::callPtrInfo(MachineFunction &MF,
                                                 const char *ES) {
  return MachinePointerInfo(MF.getPSVManager().getExternalSymbolCallEntry(ES));
}
```
- EN: Implements `MipsFunctionInfo::isISRRegFI`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::isISRRegFI`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-199
```cpp
MachinePointerInfo MipsFunctionInfo::callPtrInfo(MachineFunction &MF,
                                                 const GlobalValue *GV) {
  return MachinePointerInfo(MF.getPSVManager().getGlobalValueCallEntry(GV));
}
```
- EN: Implements `MipsFunctionInfo::callPtrInfo`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::callPtrInfo`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 201-209
```cpp
int MipsFunctionInfo::getMoveF64ViaSpillFI(MachineFunction &MF,
                                           const TargetRegisterClass *RC) {
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
  if (MoveF64ViaSpillFI == -1) {
    MoveF64ViaSpillFI = MF.getFrameInfo().CreateStackObject(
        TRI.getSpillSize(*RC), TRI.getSpillAlign(*RC), false);
  }
  return MoveF64ViaSpillFI;
}
```
- EN: Implements `MipsFunctionInfo::getMoveF64ViaSpillFI`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::getMoveF64ViaSpillFI`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 211-211
```cpp
void MipsFunctionInfo::anchor() {}
```
- EN: Implements `MipsFunctionInfo::anchor`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsFunctionInfo::anchor`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsMachineFunction.h`, `MCTargetDesc/MipsABIInfo.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`MipsMachineFunction.h`, `MCTargetDesc/MipsABIInfo.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/PseudoSourceValue.h`, `llvm/CodeGen/PseudoSourceValueManager.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/Support/CommandLine.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/PseudoSourceValue.h`, `llvm/CodeGen/PseudoSourceValueManager.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/Support/CommandLine.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
