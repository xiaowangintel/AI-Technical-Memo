# WebAssemblySubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblySubtarget.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the WebAssembly-specific subclass of TargetSubtarget.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblySubtarget.cpp`，主要负责 WebAssembly 后端的子目标特性建模。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblySubtarget.cpp - WebAssembly Subtarget Information ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Subtarget feature gating influences the behavior here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 子目标特性裁剪会影响这里的行为。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the WebAssembly-specific subclass of
/// TargetSubtarget.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 子目标特性裁剪会影响这里的行为。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblySubtarget.h"
#include "GISel/WebAssemblyCallLowering.h"
#include "GISel/WebAssemblyLegalizerInfo.h"
#include "GISel/WebAssemblyRegisterBankInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 20-27

```cpp
#include "WebAssembly.h"
#include "WebAssemblyInstrInfo.h"
#include "WebAssemblyTargetMachine.h"
#include "llvm/MC/TargetRegistry.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-subtarget"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 28-65

```cpp
#define GET_SUBTARGETINFO_CTOR
#define GET_SUBTARGETINFO_TARGET_DESC
#include "WebAssemblyGenSubtargetInfo.inc"

WebAssemblySubtarget &
WebAssemblySubtarget::initializeSubtargetDependencies(StringRef CPU,
                                                      StringRef FS) {
  // Determine default and user-specified characteristics
  LLVM_DEBUG(llvm::dbgs() << "initializeSubtargetDependencies\n");

  if (CPU.empty())
    CPU = "generic";

  ParseSubtargetFeatures(CPU, /*TuneCPU*/ CPU, FS);

  FeatureBitset Bits = getFeatureBits();

  // bulk-memory implies bulk-memory-opt
  if (HasBulkMemory) {
    HasBulkMemoryOpt = true;
    Bits.set(WebAssembly::FeatureBulkMemoryOpt);
  }

  // gc implies reference-types
  if (HasGC) {
    HasReferenceTypes = true;
  }

  // reference-types implies call-indirect-overlong
  if (HasReferenceTypes) {
    HasCallIndirectOverlong = true;
    Bits.set(WebAssembly::FeatureCallIndirectOverlong);
  }

  // In case we changed any bits, update `MCSubtargetInfo`'s `FeatureBitset`.
  setFeatureBits(Bits);

  return *this;
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 66-103

```cpp
}

WebAssemblySubtarget::WebAssemblySubtarget(const Triple &TT,
                                           const std::string &CPU,
                                           const std::string &FS,
                                           const TargetMachine &TM)
    : WebAssemblyGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS),
      TargetTriple(TT), InstrInfo(initializeSubtargetDependencies(CPU, FS)),
      TLInfo(TM, *this) {
  CallLoweringInfo.reset(new WebAssemblyCallLowering(*getTargetLowering()));
  Legalizer.reset(new WebAssemblyLegalizerInfo(*this));
  auto *RBI = new WebAssemblyRegisterBankInfo(*getRegisterInfo());
  RegBankInfo.reset(RBI);

  InstSelector.reset(createWebAssemblyInstructionSelector(
      *static_cast<const WebAssemblyTargetMachine *>(&TM), *this, *RBI));
}

bool WebAssemblySubtarget::enableAtomicExpand() const {
  // If atomics are disabled, atomic ops are lowered instead of expanded
  return hasAtomics();
}

bool WebAssemblySubtarget::enableMachineScheduler() const {
  // Disable the MachineScheduler for now. Even with ShouldTrackPressure set and
  // enableMachineSchedDefaultSched overridden, it appears to have an overall
  // negative effect for the kinds of register optimizations we're doing.
  return false;
}

bool WebAssemblySubtarget::useAA() const { return true; }

const CallLowering *WebAssemblySubtarget::getCallLowering() const {
  return CallLoweringInfo.get();
}

InstructionSelector *WebAssemblySubtarget::getInstructionSelector() const {
  return InstSelector.get();
```
- **EN**: Implements helper routine(s) `WebAssemblySubtarget`, `WebAssemblyGenSubtargetInfo`, `TargetTriple` for this portion of the WebAssembly backend subtarget feature modeling. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分子目标特性建模所需的辅助例程 `WebAssemblySubtarget`, `WebAssemblyGenSubtargetInfo`, `TargetTriple`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 104-112

```cpp
}

const LegalizerInfo *WebAssemblySubtarget::getLegalizerInfo() const {
  return Legalizer.get();
}

const RegisterBankInfo *WebAssemblySubtarget::getRegBankInfo() const {
  return RegBankInfo.get();
}
```
- **EN**: Implements helper routine(s) `getLegalizerInfo`, `get`, `getRegBankInfo` for this portion of the WebAssembly backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分子目标特性建模所需的辅助例程 `getLegalizerInfo`, `get`, `getRegBankInfo`。 子目标特性裁剪会影响这里的行为。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Subtarget feature modeling / 子目标特性建模
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Type or operation legalization / 类型或操作合法化
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblySubtarget.h`
- `GISel/WebAssemblyCallLowering.h`
- `GISel/WebAssemblyLegalizerInfo.h`
- `GISel/WebAssemblyRegisterBankInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyInstrInfo.h`
- `WebAssemblyTargetMachine.h`
- `llvm/MC/TargetRegistry.h`
- `WebAssemblyGenSubtargetInfo.inc`

### Important Collaborators / 重要协作组件

- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
