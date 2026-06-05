# WebAssemblyTargetMachine.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyTargetMachine.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file declares the WebAssembly-specific subclass of TargetMachine.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyTargetMachine.h`，主要负责 WebAssembly 后端的目标机器配置。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
// WebAssemblyTargetMachine.h - Define TargetMachine for WebAssembly -*- C++ -*-
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file declares the WebAssembly-specific subclass of
/// TargetMachine.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYTARGETMACHINE_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYTARGETMACHINE_H

#include "WebAssemblySubtarget.h"
#include "llvm/CodeGen/CodeGenTargetMachineImpl.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 20-32

```cpp
#include <optional>

namespace llvm {

namespace WebAssembly {
// Exception handling / setjmp-longjmp handling command-line options
extern cl::opt<bool> WasmEnableEmEH;   // asm.js-style EH
extern cl::opt<bool> WasmEnableEmSjLj; // asm.js-style SjLJ
extern cl::opt<bool> WasmEnableEH;     // EH using Wasm EH instructions
extern cl::opt<bool> WasmEnableSjLj;   // SjLj using Wasm EH instructions
extern cl::opt<bool> WasmUseLegacyEH;  // Legacy Wasm EH
} // namespace WebAssembly
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 33-58

```cpp
class WebAssemblyTargetMachine final : public CodeGenTargetMachineImpl {
  std::unique_ptr<TargetLoweringObjectFile> TLOF;
  mutable StringMap<std::unique_ptr<WebAssemblySubtarget>> SubtargetMap;
  bool UsesMultivalueABI = false;

public:
  WebAssemblyTargetMachine(const Target &T, const Triple &TT, StringRef CPU,
                           StringRef FS, const TargetOptions &Options,
                           std::optional<Reloc::Model> RM,
                           std::optional<CodeModel::Model> CM,
                           CodeGenOptLevel OL, bool JIT);

  ~WebAssemblyTargetMachine() override;

  const WebAssemblySubtarget *getSubtargetImpl() const;
  const WebAssemblySubtarget *getSubtargetImpl(std::string CPU,
                                               std::string FS) const;
  const WebAssemblySubtarget *
  getSubtargetImpl(const Function &F) const override;

  // Pass Pipeline Configuration
  TargetPassConfig *createPassConfig(PassManagerBase &PM) override;

  TargetLoweringObjectFile *getObjFileLowering() const override {
    return TLOF.get();
  }
```
- **EN**: Declares a backend-facing type `WebAssemblyTargetMachine`, `getSubtargetImpl`, `createPassConfig` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `WebAssemblyTargetMachine`, `getSubtargetImpl`, `createPassConfig`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 59-80

```cpp

  MachineFunctionInfo *
  createMachineFunctionInfo(BumpPtrAllocator &Allocator, const Function &F,
                            const TargetSubtargetInfo *STI) const override;

  TargetTransformInfo getTargetTransformInfo(const Function &F) const override;

  bool usesPhysRegsForValues() const override { return false; }

  yaml::MachineFunctionInfo *createDefaultFuncInfoYAML() const override;
  yaml::MachineFunctionInfo *
  convertFuncInfoToYAML(const MachineFunction &MF) const override;
  bool parseMachineFunctionInfo(const yaml::MachineFunctionInfo &,
                                PerFunctionMIParsingState &PFS,
                                SMDiagnostic &Error,
                                SMRange &SourceRange) const override;

  bool usesMultivalueABI() const { return UsesMultivalueABI; }
};

} // end namespace llvm
```
- **EN**: Implements helper routine(s) `createMachineFunctionInfo`, `getTargetTransformInfo`, `usesPhysRegsForValues` for this portion of the WebAssembly backend target machine configuration. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `createMachineFunctionInfo`, `getTargetTransformInfo`, `usesPhysRegsForValues`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 81-81

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Target machine configuration / 目标机器配置
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblySubtarget.h`
- `llvm/CodeGen/CodeGenTargetMachineImpl.h`
- `optional`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
