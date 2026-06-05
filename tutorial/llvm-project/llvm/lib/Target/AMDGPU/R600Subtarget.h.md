# R600Subtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600Subtarget.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for R600Subtarget in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 R600Subtarget 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: File banner, includes, and setup
```cpp
//=====-- R600Subtarget.h - Define Subtarget for AMDGPU R600 ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//==-----------------------------------------------------------------------===//
//
/// \file
/// AMDGPU R600 specific subclass of TargetSubtarget.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_R600SUBTARGET_H
#define LLVM_LIB_TARGET_AMDGPU_R600SUBTARGET_H

#include "AMDGPUSubtarget.h"
#include "R600FrameLowering.h"
#include "R600ISelLowering.h"
#include "R600InstrInfo.h"
#include "Utils/AMDGPUBaseInfo.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 23-43: Header dependencies and setup
```cpp
#define GET_SUBTARGETINFO_HEADER
#include "R600GenSubtargetInfo.inc"

namespace llvm {

class R600Subtarget final : public R600GenSubtargetInfo,
                            public AMDGPUSubtarget {

#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool ATTRIBUTE = DEFAULT;
#include "R600GenSubtargetInfo.inc"

private:
  R600InstrInfo InstrInfo;
  R600FrameLowering FrameLowering;
  short TexVTXClauseSize = 0;
  Generation Gen = R600;
  R600TargetLowering TLInfo;
  InstrItineraryData InstrItins;
  std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `R600Subtarget`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`R600Subtarget`。

### Lines 44-67: Defines R600Subtarget
```cpp
public:
  R600Subtarget(const Triple &TT, StringRef CPU, StringRef FS,
                const TargetMachine &TM);

  ~R600Subtarget() override;

  const R600InstrInfo *getInstrInfo() const override { return &InstrInfo; }

  const R600FrameLowering *getFrameLowering() const override {
    return &FrameLowering;
  }

  const R600TargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }

  const R600RegisterInfo *getRegisterInfo() const override {
    return &InstrInfo.getRegisterInfo();
  }

  const InstrItineraryData *getInstrItineraryData() const override {
    return &InstrItins;
  }

```
**EN:** This section contains concrete logic for R600Subtarget. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 R600Subtarget 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 68-88: Declares getSelectionDAGInfo
```cpp
  const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;

  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);

  Generation getGeneration() const {
    return Gen;
  }

  Align getStackAlignment() const { return Align(4); }

  R600Subtarget &initializeSubtargetDependencies(const Triple &TT,
                                                 StringRef GPU, StringRef FS);

  bool hasBFE() const {
    return (getGeneration() >= EVERGREEN);
  }

  bool hasBFI() const {
    return (getGeneration() >= EVERGREEN);
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 89-109: Defines hasBCNT
```cpp
  bool hasBCNT(unsigned Size) const {
    if (Size == 32)
      return (getGeneration() >= EVERGREEN);

    return false;
  }

  bool hasBORROW() const {
    return (getGeneration() >= EVERGREEN);
  }

  bool hasCARRY() const {
    return (getGeneration() >= EVERGREEN);
  }

  bool hasCaymanISA() const { return HasCaymanISA; }

  bool hasFFBL() const {
    return (getGeneration() >= EVERGREEN);
  }

```
**EN:** This section contains concrete logic for hasBCNT. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 hasBCNT 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 110-133: Defines hasFFBH
```cpp
  bool hasFFBH() const {
    return (getGeneration() >= EVERGREEN);
  }

  bool hasFMA() const override { return HasFMA; }

  bool hasMadMacF32Insts() const override { return HasMadMacF32Insts; }

  bool hasFP64() const override { return HasFP64; }

  bool hasCFALUBug() const { return HasCFALUBug; }

  bool hasVertexCache() const { return HasVertexCache; }

  short getTexVTXClauseSize() const { return TexVTXClauseSize; }

  bool enableMachineScheduler() const override {
    return true;
  }

  bool enableSubRegLiveness() const override {
    return true;
  }

```
**EN:** This section contains concrete logic for hasFFBH. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 hasFFBH 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 134-156: Defines getMaxWorkGroupsPerCU
```cpp
  /// \returns Maximum number of work groups per compute unit supported by the
  /// subtarget and limited by given \p FlatWorkGroupSize.
  unsigned getMaxWorkGroupsPerCU(unsigned FlatWorkGroupSize) const override {
    return AMDGPU::IsaInfo::getMaxWorkGroupsPerCU(this, FlatWorkGroupSize);
  }

  /// \returns Minimum flat work group size supported by the subtarget.
  unsigned getMinFlatWorkGroupSize() const override {
    return AMDGPU::IsaInfo::getMinFlatWorkGroupSize(this);
  }

  /// \returns Maximum flat work group size supported by the subtarget.
  unsigned getMaxFlatWorkGroupSize() const override {
    return AMDGPU::IsaInfo::getMaxFlatWorkGroupSize();
  }

  /// \returns Number of waves per execution unit required to support the given
  /// \p FlatWorkGroupSize.
  unsigned
  getWavesPerEUForWorkGroup(unsigned FlatWorkGroupSize) const override {
    return AMDGPU::IsaInfo::getWavesPerEUForWorkGroup(this, FlatWorkGroupSize);
  }

```
**EN:** This section contains concrete logic for getMaxWorkGroupsPerCU. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `IsaInfo::getMaxWorkGroupsPerCU`, `IsaInfo::getMinFlatWorkGroupSize`, `IsaInfo::getMaxFlatWorkGroupSize`.
**CN:** 本节包含与 getMaxWorkGroupsPerCU 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`IsaInfo::getMaxWorkGroupsPerCU`, `IsaInfo::getMinFlatWorkGroupSize`, `IsaInfo::getMaxFlatWorkGroupSize`。

### Lines 157-172: Preprocessor guards and macros
```cpp
  /// \returns Minimum number of waves per execution unit supported by the
  /// subtarget.
  unsigned getMinWavesPerEU() const override {
    return AMDGPU::IsaInfo::getMinWavesPerEU(this);
  }

  bool requiresDisjointEarlyClobberAndUndef() const override {
    // AMDGPU doesn't care if early-clobber and undef operands are allocated
    // to the same register.
    return false;
  }
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_R600SUBTARGET_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `IsaInfo::getMinWavesPerEU`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`IsaInfo::getMinWavesPerEU`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `R600Subtarget`, `IsaInfo::getMaxWorkGroupsPerCU`, `IsaInfo::getMinFlatWorkGroupSize`, `IsaInfo::getMaxFlatWorkGroupSize`, `IsaInfo::getWavesPerEUForWorkGroup`, `IsaInfo::getMinWavesPerEU`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUSubtarget.h"`
- `"R600FrameLowering.h"`
- `"R600ISelLowering.h"`
- `"R600InstrInfo.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"R600GenSubtargetInfo.inc"`
