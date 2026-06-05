# ARMMachineFunctionInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMMachineFunctionInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMMachineFunctionInfo` for the ARM backend, focusing on machine-function specific state.
- 用途 (CN): 实现 ARM 后端中的 `ARMMachineFunctionInfo`，重点处理机器函数特定状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- ARMMachineFunctionInfo.cpp - ARM machine function info ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-10
```cpp
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 12-12
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 14-14
```cpp
void ARMFunctionInfo::anchor() {}
```
- EN: Implements `ARMFunctionInfo::anchor`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFunctionInfo::anchor`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 16-17
```cpp
yaml::ARMFunctionInfo::ARMFunctionInfo(const llvm::ARMFunctionInfo &MFI)
    : LRSpilled(MFI.isLRSpilled()) {}
```
- EN: Implements `yaml::ARMFunctionInfo::ARMFunctionInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `yaml::ARMFunctionInfo::ARMFunctionInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 19-21
```cpp
void yaml::ARMFunctionInfo::mappingImpl(yaml::IO &YamlIO) {
  MappingTraits<ARMFunctionInfo>::mapping(YamlIO, *this);
}
```
- EN: Implements `yaml::ARMFunctionInfo::mappingImpl`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `yaml::ARMFunctionInfo::mappingImpl`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 23-26
```cpp
void ARMFunctionInfo::initializeBaseYamlFields(
    const yaml::ARMFunctionInfo &YamlMFI) {
  LRSpilled = YamlMFI.LRSpilled;
}
```
- EN: Implements `ARMFunctionInfo::initializeBaseYamlFields`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFunctionInfo::initializeBaseYamlFields`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 28-31
```cpp
static bool GetBranchTargetEnforcement(const Function &F,
                                       const ARMSubtarget *Subtarget) {
  if (!Subtarget->isMClass() || !Subtarget->hasV7Ops())
    return false;
```
- EN: Implements `GetBranchTargetEnforcement`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `GetBranchTargetEnforcement`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 33-34
```cpp
  return F.hasFnAttribute("branch-target-enforcement");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 36-41
```cpp
// The pair returns values for the ARMFunctionInfo members
// SignReturnAddress and SignReturnAddressAll respectively.
static std::pair<bool, bool> GetSignReturnAddress(const Function &F) {
  if (!F.hasFnAttribute("sign-return-address")) {
    return {false, false};
  }
```
- EN: Implements `GetSignReturnAddress`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `GetSignReturnAddress`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-45
```cpp
  StringRef Scope = F.getFnAttribute("sign-return-address").getValueAsString();
  if (Scope == "none")
    return {false, false};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 47-48
```cpp
  if (Scope == "all")
    return {true, true};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 50-52
```cpp
  assert(Scope == "non-leaf");
  return {true, false};
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 54-62
```cpp
ARMFunctionInfo::ARMFunctionInfo(const Function &F,
                                 const ARMSubtarget *Subtarget)
    : isThumb(Subtarget->isThumb()), hasThumb2(Subtarget->hasThumb2()),
      IsCmseNSEntry(F.hasFnAttribute("cmse_nonsecure_entry")),
      IsCmseNSCall(F.hasFnAttribute("cmse_nonsecure_call")),
      BranchTargetEnforcement(GetBranchTargetEnforcement(F, Subtarget)) {
  if (Subtarget->isMClass() && Subtarget->hasV7Ops())
    std::tie(SignReturnAddress, SignReturnAddressAll) = GetSignReturnAddress(F);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 64-69
```cpp
MachineFunctionInfo *
ARMFunctionInfo::clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
                       const DenseMap<MachineBasicBlock *, MachineBasicBlock *>
                           &Src2DstMBB) const {
  return DestMF.cloneInfo<ARMFunctionInfo>(*this);
}
```
- EN: Implements `ARMFunctionInfo::clone`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMFunctionInfo::clone`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: machine-function specific state.
  - CN: 核心职责：机器函数特定状态。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ARMMachineFunctionInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `(none)`.
  - CN: LLVM 基础设施头文件：`(none)`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
