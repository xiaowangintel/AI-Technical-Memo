# ARMTargetMachine.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMTargetMachine.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares the ARM specific subclass of TargetMachine.
- 用途 (CN): 声明 ARM 后端中的 `ARMTargetMachine`，并提供与目标机器配置与优化流水线集成相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMTargetMachine.h - Define TargetMachine for ARM -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the ARM specific subclass of TargetMachine.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMTARGETMACHINE_H
#define LLVM_LIB_TARGET_ARM_ARMTARGETMACHINE_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-25
```cpp
#include "ARMSubtarget.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/CodeGenTargetMachineImpl.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/ARMTargetParser.h"
#include <memory>
#include <optional>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 27-27
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 29-31
```cpp
class ARMBaseTargetMachine : public CodeGenTargetMachineImpl {
public:
  ARM::ARMABI TargetABI;
```
- EN: Declares `ARMBaseTargetMachine`, packaging target-specific state and APIs around `ARMTargetMachine`.
- CN: 这里声明 `ARMBaseTargetMachine`，把与 `ARMTargetMachine` 相关的目标特定状态和 API 组织在一起。

### Lines 33-36
```cpp
protected:
  std::unique_ptr<TargetLoweringObjectFile> TLOF;
  bool isLittle;
  mutable StringMap<std::unique_ptr<ARMSubtarget>> SubtargetMap;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 38-39
```cpp
  /// Reset internal state.
  void reset() override;
```
- EN: Declares `reset`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `reset`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 41-46
```cpp
public:
  ARMBaseTargetMachine(const Target &T, const Triple &TT, StringRef CPU,
                       StringRef FS, const TargetOptions &Options,
                       std::optional<Reloc::Model> RM,
                       std::optional<CodeModel::Model> CM, CodeGenOptLevel OL);
  ~ARMBaseTargetMachine() override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 48-53
```cpp
  const ARMSubtarget *getSubtargetImpl(const Function &F) const override;
  // DO NOT IMPLEMENT: There is no such thing as a valid default subtarget,
  // subtargets are per-function entities based on the target-specific
  // attributes of each function.
  const ARMSubtarget *getSubtargetImpl() const = delete;
  bool isLittleEndian() const { return isLittle; }
```
- EN: Implements `getSubtargetImpl`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSubtargetImpl`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-55
```cpp
  TargetTransformInfo getTargetTransformInfo(const Function &F) const override;
```
- EN: Declares `getTargetTransformInfo`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetTransformInfo`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-58
```cpp
  // Pass Pipeline Configuration
  TargetPassConfig *createPassConfig(PassManagerBase &PM) override;
```
- EN: Declares `createPassConfig`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createPassConfig`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 60-60
```cpp
  void registerPassBuilderCallbacks(PassBuilder &PB) override;
```
- EN: Declares `registerPassBuilderCallbacks`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `registerPassBuilderCallbacks`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-64
```cpp
  TargetLoweringObjectFile *getObjFileLowering() const override {
    return TLOF.get();
  }
```
- EN: Implements `getObjFileLowering`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getObjFileLowering`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 66-69
```cpp
  bool isAPCS_ABI() const {
    assert(TargetABI != ARM::ARM_ABI_UNKNOWN);
    return TargetABI == ARM::ARM_ABI_APCS;
  }
```
- EN: Implements `isAPCS_ABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isAPCS_ABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-74
```cpp
  bool isAAPCS_ABI() const {
    assert(TargetABI != ARM::ARM_ABI_UNKNOWN);
    return TargetABI == ARM::ARM_ABI_AAPCS || TargetABI == ARM::ARM_ABI_AAPCS16;
  }
```
- EN: Implements `isAAPCS_ABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isAAPCS_ABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 76-79
```cpp
  bool isAAPCS16_ABI() const {
    assert(TargetABI != ARM::ARM_ABI_UNKNOWN);
    return TargetABI == ARM::ARM_ABI_AAPCS16;
  }
```
- EN: Implements `isAAPCS16_ABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isAAPCS16_ABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 81-89
```cpp
  bool isTargetHardFloat() const {
    return TargetTriple.getEnvironment() == Triple::GNUEABIHF ||
           TargetTriple.getEnvironment() == Triple::GNUEABIHFT64 ||
           TargetTriple.getEnvironment() == Triple::MuslEABIHF ||
           TargetTriple.getEnvironment() == Triple::EABIHF ||
           (TargetTriple.isOSBinFormatMachO() &&
            TargetTriple.getSubArch() == Triple::ARMSubArch_v7em) ||
           TargetTriple.isOSWindows() || TargetABI == ARM::ARM_ABI_AAPCS16;
  }
```
- EN: Implements `isTargetHardFloat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetHardFloat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 91-91
```cpp
  bool targetSchedulesPostRAScheduling() const override { return true; };
```
- EN: Implements `targetSchedulesPostRAScheduling`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `targetSchedulesPostRAScheduling`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-95
```cpp
  MachineFunctionInfo *
  createMachineFunctionInfo(BumpPtrAllocator &Allocator, const Function &F,
                            const TargetSubtargetInfo *STI) const override;
```
- EN: Declares `createMachineFunctionInfo`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `createMachineFunctionInfo`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 97-101
```cpp
  /// Returns true if a cast between SrcAS and DestAS is a noop.
  bool isNoopAddrSpaceCast(unsigned SrcAS, unsigned DestAS) const override {
    // Addrspacecasts are always noops.
    return true;
  }
```
- EN: Implements `isNoopAddrSpaceCast`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isNoopAddrSpaceCast`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 103-105
```cpp
  bool isGVIndirectSymbol(const GlobalValue *GV) const {
    if (!shouldAssumeDSOLocal(GV))
      return true;
```
- EN: Implements `isGVIndirectSymbol`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isGVIndirectSymbol`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 107-112
```cpp
    // 32 bit macho has no relocation for a-b if a is undefined, even if b is in
    // the section that is being relocated. This means we have to use o load
    // even for GVs that are known to be local to the dso.
    if (getTargetTriple().isOSBinFormatMachO() && isPositionIndependent() &&
        (GV->isDeclarationForLinker() || GV->hasCommonLinkage()))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 114-115
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 117-128
```cpp
  yaml::MachineFunctionInfo *createDefaultFuncInfoYAML() const override;
  yaml::MachineFunctionInfo *
  convertFuncInfoToYAML(const MachineFunction &MF) const override;
  bool parseMachineFunctionInfo(const yaml::MachineFunctionInfo &,
                                PerFunctionMIParsingState &PFS,
                                SMDiagnostic &Error,
                                SMRange &SourceRange) const override;
  ScheduleDAGInstrs *
  createMachineScheduler(MachineSchedContext *C) const override;
  ScheduleDAGInstrs *
  createPostMachineScheduler(MachineSchedContext *C) const override;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 130-139
```cpp
/// ARM/Thumb little endian target machine.
///
class ARMLETargetMachine : public ARMBaseTargetMachine {
public:
  ARMLETargetMachine(const Target &T, const Triple &TT, StringRef CPU,
                     StringRef FS, const TargetOptions &Options,
                     std::optional<Reloc::Model> RM,
                     std::optional<CodeModel::Model> CM, CodeGenOptLevel OL,
                     bool JIT);
};
```
- EN: Declares `ARMLETargetMachine`, packaging target-specific state and APIs around `ARMTargetMachine`.
- CN: 这里声明 `ARMLETargetMachine`，把与 `ARMTargetMachine` 相关的目标特定状态和 API 组织在一起。

### Lines 141-150
```cpp
/// ARM/Thumb big endian target machine.
///
class ARMBETargetMachine : public ARMBaseTargetMachine {
public:
  ARMBETargetMachine(const Target &T, const Triple &TT, StringRef CPU,
                     StringRef FS, const TargetOptions &Options,
                     std::optional<Reloc::Model> RM,
                     std::optional<CodeModel::Model> CM, CodeGenOptLevel OL,
                     bool JIT);
};
```
- EN: Declares `ARMBETargetMachine`, packaging target-specific state and APIs around `ARMTargetMachine`.
- CN: 这里声明 `ARMBETargetMachine`，把与 `ARMTargetMachine` 相关的目标特定状态和 API 组织在一起。

### Lines 152-152
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 154-154
```cpp
#endif // LLVM_LIB_TARGET_ARM_ARMTARGETMACHINE_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: target-machine configuration and pass-pipeline integration.
  - CN: 核心职责：目标机器配置与优化流水线集成。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/CodeGenTargetMachineImpl.h`, `llvm/Support/CodeGen.h`, `llvm/Target/TargetMachine.h`, `llvm/TargetParser/ARMTargetParser.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/CodeGenTargetMachineImpl.h`, `llvm/Support/CodeGen.h`, `llvm/Target/TargetMachine.h`, `llvm/TargetParser/ARMTargetParser.h`。
- EN: Standard/system headers: `memory`, `optional`.
  - CN: 标准库/系统头文件：`memory`, `optional`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
