# ARMCallLowering.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMCallLowering.h`
- Repository: `llvm-project`
- Purpose (EN): Declares `ARMCallLowering` for the ARM backend and exposes interfaces for GlobalISel call lowering.
- 用途 (CN): 声明 ARM 后端中的 `ARMCallLowering`，并提供与GlobalISel 调用降级相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- llvm/lib/Target/ARM/ARMCallLowering.h - Call lowering ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file describes how to lower LLVM calls to machine code calls.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-15
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMCALLLOWERING_H
#define LLVM_LIB_TARGET_ARM_ARMCALLLOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 17-21
```cpp
#include "llvm/ADT/ArrayRef.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/IR/CallingConv.h"
#include <cstdint>
#include <functional>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 25-28
```cpp
class ARMTargetLowering;
class MachineInstrBuilder;
class MachineIRBuilder;
class Value;
```
- EN: Declares `ARMTargetLowering`, packaging target-specific state and APIs around `ARMCallLowering`.
- CN: 这里声明 `ARMTargetLowering`，把与 `ARMCallLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 30-32
```cpp
class ARMCallLowering : public CallLowering {
public:
  ARMCallLowering(const ARMTargetLowering &TLI);
```
- EN: Declares `ARMCallLowering`, packaging target-specific state and APIs around `ARMCallLowering`.
- CN: 这里声明 `ARMCallLowering`，把与 `ARMCallLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 34-36
```cpp
  bool lowerReturn(MachineIRBuilder &MIRBuilder, const Value *Val,
                   ArrayRef<Register> VRegs,
                   FunctionLoweringInfo &FLI) const override;
```
- EN: Declares `lowerReturn`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerReturn`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-40
```cpp
  bool lowerFormalArguments(MachineIRBuilder &MIRBuilder, const Function &F,
                            ArrayRef<ArrayRef<Register>> VRegs,
                            FunctionLoweringInfo &FLI) const override;
```
- EN: Declares `lowerFormalArguments`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerFormalArguments`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-43
```cpp
  bool lowerCall(MachineIRBuilder &MIRBuilder,
                 CallLoweringInfo &Info) const override;
```
- EN: Declares `lowerCall`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerCall`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 45-45
```cpp
  bool enableBigEndian() const override;
```
- EN: Declares `enableBigEndian`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enableBigEndian`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-51
```cpp
private:
  bool lowerReturnVal(MachineIRBuilder &MIRBuilder, const Value *Val,
                      ArrayRef<Register> VRegs,
                      MachineInstrBuilder &Ret) const;
};
```
- EN: Declares `lowerReturnVal`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerReturnVal`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 53-53
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 55-55
```cpp
#endif // LLVM_LIB_TARGET_ARM_ARMCALLLOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: GlobalISel call lowering.
  - CN: 核心职责：GlobalISel 调用降级。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。

## Dependencies / 依赖关系

- EN: Backend-local headers: `(none)`.
  - CN: 后端本地头文件：`(none)`。
- EN: LLVM infrastructure headers: `llvm/ADT/ArrayRef.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/IR/CallingConv.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/ArrayRef.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/IR/CallingConv.h`。
- EN: Standard/system headers: `cstdint`, `functional`.
  - CN: 标准库/系统头文件：`cstdint`, `functional`。
