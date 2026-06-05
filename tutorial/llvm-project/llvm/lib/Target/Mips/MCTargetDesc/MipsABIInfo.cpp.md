# MipsABIInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MCTargetDesc/MipsABIInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsABIInfo` for the Mips backend, focusing on MC-layer target description and encoding support.
- 用途 (CN): 实现 Mips 后端中的 `MipsABIInfo`，重点处理MC 层目标描述与编码支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===---- MipsABIInfo.cpp - Information about MIPS ABI's ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-13
```cpp
#include "MipsABIInfo.h"
#include "Mips.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/Support/CommandLine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 15-15
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 17-26
```cpp
// Note: this option is defined here to be visible from libLLVMMipsAsmParser
//       and libLLVMMipsCodeGen
cl::opt<bool>
EmitJalrReloc("mips-jalr-reloc", cl::Hidden,
              cl::desc("MIPS: Emit R_{MICRO}MIPS_JALR relocation with jalr"),
              cl::init(true));
cl::opt<bool>
    NoZeroDivCheck("mno-check-zero-division", cl::Hidden,
                   cl::desc("MIPS: Don't trap on integer division by zero."),
                   cl::init(false));
```
- EN: Implements `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 28-29
```cpp
namespace {
static const MCPhysReg O32IntRegs[4] = {Mips::A0, Mips::A1, Mips::A2, Mips::A3};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 31-34
```cpp
static const MCPhysReg Mips64IntRegs[8] = {
    Mips::A0_64, Mips::A1_64, Mips::A2_64, Mips::A3_64,
    Mips::T0_64, Mips::T1_64, Mips::T2_64, Mips::T3_64};
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 36-42
```cpp
ArrayRef<MCPhysReg> MipsABIInfo::GetByValArgRegs() const {
  if (IsO32())
    return ArrayRef(O32IntRegs);
  if (IsN32() || IsN64())
    return ArrayRef(Mips64IntRegs);
  llvm_unreachable("Unhandled ABI");
}
```
- EN: Implements `MipsABIInfo::GetByValArgRegs`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetByValArgRegs`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 44-54
```cpp
ArrayRef<MCPhysReg> MipsABIInfo::getVarArgRegs(bool isGP64bit) const {
  if (IsO32()) {
    if (isGP64bit)
      return ArrayRef(Mips64IntRegs);
    else
      return ArrayRef(O32IntRegs);
  }
  if (IsN32() || IsN64())
    return ArrayRef(Mips64IntRegs);
  llvm_unreachable("Unhandled ABI");
}
```
- EN: Implements `MipsABIInfo::getVarArgRegs`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::getVarArgRegs`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-62
```cpp
unsigned MipsABIInfo::GetCalleeAllocdArgSizeInBytes(CallingConv::ID CC) const {
  if (IsO32())
    return CC != CallingConv::Fast ? 16 : 0;
  if (IsN32() || IsN64())
    return 0;
  llvm_unreachable("Unhandled ABI");
}
```
- EN: Implements `MipsABIInfo::GetCalleeAllocdArgSizeInBytes`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetCalleeAllocdArgSizeInBytes`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 64-73
```cpp
MipsABIInfo MipsABIInfo::computeTargetABI(const Triple &TT, StringRef ABIName) {
  if (ABIName.starts_with("o32"))
    return MipsABIInfo::O32();
  if (ABIName.starts_with("n32"))
    return MipsABIInfo::N32();
  if (ABIName.starts_with("n64"))
    return MipsABIInfo::N64();
  if (TT.isABIN32())
    return MipsABIInfo::N32();
  assert(ABIName.empty() && "Unknown ABI option for MIPS");
```
- EN: Implements `MipsABIInfo::computeTargetABI`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::computeTargetABI`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-78
```cpp
  if (TT.isMIPS64())
    return MipsABIInfo::N64();
  return MipsABIInfo::O32();
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 80-82
```cpp
unsigned MipsABIInfo::GetStackPtr() const {
  return ArePtrs64bit() ? Mips::SP_64 : Mips::SP;
}
```
- EN: Implements `MipsABIInfo::GetStackPtr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetStackPtr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 84-86
```cpp
unsigned MipsABIInfo::GetFramePtr() const {
  return ArePtrs64bit() ? Mips::FP_64 : Mips::FP;
}
```
- EN: Implements `MipsABIInfo::GetFramePtr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetFramePtr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 88-90
```cpp
unsigned MipsABIInfo::GetBasePtr() const {
  return ArePtrs64bit() ? Mips::S7_64 : Mips::S7;
}
```
- EN: Implements `MipsABIInfo::GetBasePtr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetBasePtr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 92-94
```cpp
unsigned MipsABIInfo::GetGlobalPtr() const {
  return ArePtrs64bit() ? Mips::GP_64 : Mips::GP;
}
```
- EN: Implements `MipsABIInfo::GetGlobalPtr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetGlobalPtr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 96-98
```cpp
unsigned MipsABIInfo::GetNullPtr() const {
  return ArePtrs64bit() ? Mips::ZERO_64 : Mips::ZERO;
}
```
- EN: Implements `MipsABIInfo::GetNullPtr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetNullPtr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-102
```cpp
unsigned MipsABIInfo::GetZeroReg() const {
  return AreGprs64bit() ? Mips::ZERO_64 : Mips::ZERO;
}
```
- EN: Implements `MipsABIInfo::GetZeroReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetZeroReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 104-106
```cpp
unsigned MipsABIInfo::GetPtrAdduOp() const {
  return ArePtrs64bit() ? Mips::DADDu : Mips::ADDu;
}
```
- EN: Implements `MipsABIInfo::GetPtrAdduOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetPtrAdduOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 108-110
```cpp
unsigned MipsABIInfo::GetPtrAddiuOp() const {
  return ArePtrs64bit() ? Mips::DADDiu : Mips::ADDiu;
}
```
- EN: Implements `MipsABIInfo::GetPtrAddiuOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetPtrAddiuOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 112-114
```cpp
unsigned MipsABIInfo::GetPtrSubuOp() const {
  return ArePtrs64bit() ? Mips::DSUBu : Mips::SUBu;
}
```
- EN: Implements `MipsABIInfo::GetPtrSubuOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetPtrSubuOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 116-118
```cpp
unsigned MipsABIInfo::GetPtrAndOp() const {
  return ArePtrs64bit() ? Mips::AND64 : Mips::AND;
}
```
- EN: Implements `MipsABIInfo::GetPtrAndOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetPtrAndOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 120-122
```cpp
unsigned MipsABIInfo::GetGPRMoveOp() const {
  return ArePtrs64bit() ? Mips::OR64 : Mips::OR;
}
```
- EN: Implements `MipsABIInfo::GetGPRMoveOp`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetGPRMoveOp`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 124-130
```cpp
unsigned MipsABIInfo::GetEhDataReg(unsigned I) const {
  static const unsigned EhDataReg[] = {
    Mips::A0, Mips::A1, Mips::A2, Mips::A3
  };
  static const unsigned EhDataReg64[] = {
    Mips::A0_64, Mips::A1_64, Mips::A2_64, Mips::A3_64
  };
```
- EN: Implements `MipsABIInfo::GetEhDataReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsABIInfo::GetEhDataReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 132-133
```cpp
  return IsN64() ? EhDataReg64[I] : EhDataReg[I];
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsABIInfo.h`, `Mips.h`.
  - CN: 后端本地头文件：`MipsABIInfo.h`, `Mips.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/StringRef.h`, `llvm/MC/MCTargetOptions.h`, `llvm/Support/CommandLine.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringRef.h`, `llvm/MC/MCTargetOptions.h`, `llvm/Support/CommandLine.h`。
