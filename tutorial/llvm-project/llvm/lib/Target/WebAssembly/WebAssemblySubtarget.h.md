# WebAssemblySubtarget.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblySubtarget.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file declares the WebAssembly-specific subclass of TargetSubtarget.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblySubtarget.h`，主要负责 WebAssembly 后端的子目标特性建模。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=- WebAssemblySubtarget.h - Define Subtarget for the WebAssembly -*- C++ -*-//
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
/// This file declares the WebAssembly-specific subclass of
/// TargetSubtarget.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 子目标特性裁剪会影响这里的行为。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYSUBTARGET_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYSUBTARGET_H

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssemblyFrameLowering.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 20-26

```cpp
#include "WebAssemblyISelLowering.h"
#include "WebAssemblyInstrInfo.h"
#include "WebAssemblySelectionDAGInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/RegisterBankInfo.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 27-38

```cpp
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include <string>

#define GET_SUBTARGETINFO_HEADER
#include "WebAssemblyGenSubtargetInfo.inc"

namespace llvm {

// Defined in WebAssemblyGenSubtargetInfo.inc.
extern const SubtargetFeatureKV
    WebAssemblyFeatureKV[WebAssembly::NumSubtargetFeatures];
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 39-64

```cpp
class WebAssemblySubtarget final : public WebAssemblyGenSubtargetInfo {
  enum SIMDEnum {
    NoSIMD,
    SIMD128,
    RelaxedSIMD,
  } SIMDLevel = NoSIMD;

  bool HasAtomics = false;
  bool HasBulkMemory = false;
  bool HasBulkMemoryOpt = false;
  bool HasCallIndirectOverlong = false;
  bool HasCompactImports = false;
  bool HasExceptionHandling = false;
  bool HasExtendedConst = false;
  bool HasFP16 = false;
  bool HasGC = false;
  bool HasMultiMemory = false;
  bool HasMultivalue = false;
  bool HasMutableGlobals = false;
  bool HasNontrappingFPToInt = false;
  bool HasReferenceTypes = false;
  bool HasRelaxedAtomics = false;
  bool HasSignExt = false;
  bool HasTailCall = false;
  bool HasWideArithmetic = false;
```
- **EN**: Declares a backend-facing type `WebAssemblySubtarget` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `WebAssemblySubtarget`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 65-90

```cpp
  /// What processor and OS we're targeting.
  Triple TargetTriple;

  WebAssemblyFrameLowering FrameLowering;
  WebAssemblyInstrInfo InstrInfo;
  WebAssemblySelectionDAGInfo TSInfo;
  WebAssemblyTargetLowering TLInfo;

  std::unique_ptr<CallLowering> CallLoweringInfo;
  std::unique_ptr<InstructionSelector> InstSelector;
  std::unique_ptr<LegalizerInfo> Legalizer;
  std::unique_ptr<RegisterBankInfo> RegBankInfo;

  WebAssemblySubtarget &initializeSubtargetDependencies(StringRef CPU,
                                                        StringRef FS);

public:
  /// This constructor initializes the data members to match that
  /// of the specified triple.
  WebAssemblySubtarget(const Triple &TT, const std::string &CPU,
                       const std::string &FS, const TargetMachine &TM);

  const WebAssemblySelectionDAGInfo *getSelectionDAGInfo() const override {
    return &TSInfo;
  }
  const WebAssemblyFrameLowering *getFrameLowering() const override {
```
- **EN**: Implements helper routine(s) `initializeSubtargetDependencies`, `WebAssemblySubtarget`, `getSelectionDAGInfo` for this portion of the WebAssembly backend subtarget feature modeling. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分子目标特性建模所需的辅助例程 `initializeSubtargetDependencies`, `WebAssemblySubtarget`, `getSelectionDAGInfo`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 91-116

```cpp
    return &FrameLowering;
  }
  const WebAssemblyTargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }
  const WebAssemblyInstrInfo *getInstrInfo() const override {
    return &InstrInfo;
  }
  const WebAssemblyRegisterInfo *getRegisterInfo() const override {
    return &getInstrInfo()->getRegisterInfo();
  }
  const Triple &getTargetTriple() const { return TargetTriple; }
  bool enableAtomicExpand() const override;
  bool enableIndirectBrExpand() const override { return true; }
  bool enableMachineScheduler() const override;
  bool useAA() const override;

  // Predicates used by WebAssemblyInstrInfo.td.
  bool hasAddr64() const { return TargetTriple.isArch64Bit(); }
  bool hasAtomics() const { return HasAtomics; }
  bool hasBulkMemory() const { return HasBulkMemory; }
  bool hasBulkMemoryOpt() const { return HasBulkMemoryOpt; }
  bool hasCallIndirectOverlong() const { return HasCallIndirectOverlong; }
  bool hasCompactImports() const { return HasCompactImports; }
  bool hasExceptionHandling() const { return HasExceptionHandling; }
  bool hasExtendedConst() const { return HasExtendedConst; }
```
- **EN**: Implements helper routine(s) `getTargetLowering`, `getInstrInfo`, `getRegisterInfo` for this portion of the WebAssembly backend subtarget feature modeling. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分子目标特性建模所需的辅助例程 `getTargetLowering`, `getInstrInfo`, `getRegisterInfo`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 117-142

```cpp
  bool hasFP16() const { return HasFP16; }
  bool hasGC() const { return HasGC; }
  bool hasMultiMemory() const { return HasMultiMemory; }
  bool hasMultivalue() const { return HasMultivalue; }
  bool hasMutableGlobals() const { return HasMutableGlobals; }
  bool hasNontrappingFPToInt() const { return HasNontrappingFPToInt; }
  bool hasReferenceTypes() const { return HasReferenceTypes; }
  bool hasRelaxedAtomics() const { return HasRelaxedAtomics; }
  bool hasRelaxedSIMD() const { return SIMDLevel >= RelaxedSIMD; }
  bool hasSignExt() const { return HasSignExt; }
  bool hasSIMD128() const { return SIMDLevel >= SIMD128; }
  bool hasTailCall() const { return HasTailCall; }
  bool hasWideArithmetic() const { return HasWideArithmetic; }

  /// Parses features string setting specified subtarget options. Definition of
  /// function is auto generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);

  const CallLowering *getCallLowering() const override;
  InstructionSelector *getInstructionSelector() const override;
  const LegalizerInfo *getLegalizerInfo() const override;
  const RegisterBankInfo *getRegBankInfo() const override;
};

} // end namespace llvm
```
- **EN**: Implements helper routine(s) `hasFP16`, `hasGC`, `hasMultiMemory` for this portion of the WebAssembly backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分子目标特性建模所需的辅助例程 `hasFP16`, `hasGC`, `hasMultiMemory`。 子目标特性裁剪会影响这里的行为。

### Lines 143-143

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Subtarget feature modeling / 子目标特性建模
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssemblyFrameLowering.h`
- `WebAssemblyISelLowering.h`
- `WebAssemblyInstrInfo.h`
- `WebAssemblySelectionDAGInfo.h`
- `llvm/CodeGen/GlobalISel/CallLowering.h`
- `llvm/CodeGen/GlobalISel/InstructionSelector.h`
- `llvm/CodeGen/GlobalISel/LegalizerInfo.h`
- `llvm/CodeGen/RegisterBankInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `string`
- `WebAssemblyGenSubtargetInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
