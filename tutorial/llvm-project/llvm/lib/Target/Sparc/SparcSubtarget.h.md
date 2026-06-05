# SparcSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcSubtarget.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Captures CPU features and per-function subtarget configuration.
  - **CN**: 封装 CPU 特性以及按函数区分的子目标配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcSubtarget.h - Define Subtarget for the SPARC -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file declares the SPARC specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp
#ifndef LLVM_LIB_TARGET_SPARC_SPARCSUBTARGET_H
#define LLVM_LIB_TARGET_SPARC_SPARCSUBTARGET_H

#include "MCTargetDesc/SparcMCTargetDesc.h"
#include "SparcFrameLowering.h"
#include "SparcISelLowering.h"
#include "SparcInstrInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/Support/ErrorHandling.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/SparcMCTargetDesc.h`, `SparcFrameLowering.h`, `SparcISelLowering.h`, `SparcInstrInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/SparcMCTargetDesc.h`, `SparcFrameLowering.h`, `SparcISelLowering.h`, `SparcInstrInfo.h`。

### Lines 23-27
```cpp
#include "llvm/TargetParser/Triple.h"

#define GET_SUBTARGETINFO_HEADER
#include "SparcGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/TargetParser/Triple.h`, `SparcGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/TargetParser/Triple.h`, `SparcGenSubtargetInfo.inc`。

### Lines 28-35
```cpp
namespace llvm {
class StringRef;

class SparcSubtarget : public SparcGenSubtargetInfo {
  // ReserveRegister[i] - Register #i is not available as a general purpose
  // register.
  BitVector ReserveRegister;

```
- **EN**: Introduces declarations for `llvm`, `StringRef`, `SparcSubtarget`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `StringRef`, `SparcSubtarget` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 36-41
```cpp
  virtual void anchor();

#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool ATTRIBUTE = DEFAULT;
#include "SparcGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenSubtargetInfo.inc`。

### Lines 42-46
```cpp
  SparcInstrInfo InstrInfo;
  SparcTargetLowering TLInfo;
  std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
  SparcFrameLowering FrameLowering;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 47-52
```cpp
public:
  SparcSubtarget(const StringRef &CPU, const StringRef &TuneCPU,
                 const StringRef &FS, const TargetMachine &TM);

  ~SparcSubtarget() override;

```
- **EN**: Implements logic around `SparcSubtarget`, `~SparcSubtarget`.
- **CN**: 围绕 `SparcSubtarget`, `~SparcSubtarget` 实现具体逻辑。

### Lines 53-62
```cpp
  const SparcInstrInfo *getInstrInfo() const override { return &InstrInfo; }
  const TargetFrameLowering *getFrameLowering() const override {
    return &FrameLowering;
  }
  const SparcRegisterInfo *getRegisterInfo() const override {
    return &InstrInfo.getRegisterInfo();
  }
  const SparcTargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }
```
- **EN**: Implements logic around `getInstrInfo`, `getFrameLowering`, `getRegisterInfo`, `getTargetLowering`; this block returns target-specific results.
- **CN**: 围绕 `getInstrInfo`, `getFrameLowering`, `getRegisterInfo`, `getTargetLowering` 实现具体逻辑；这一段返回目标相关结果。

### Lines 63-67
```cpp

  const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;

  void initLibcallLoweringInfo(LibcallLoweringInfo &Info) const override;

```
- **EN**: Implements logic around `getSelectionDAGInfo`, `initLibcallLoweringInfo`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSelectionDAGInfo`, `initLibcallLoweringInfo` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 68-73
```cpp
  bool enableMachineScheduler() const override;

#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool GETTER() const { return ATTRIBUTE; }
#include "SparcGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenSubtargetInfo.inc`。

### Lines 74-80
```cpp
  /// ParseSubtargetFeatures - Parses features string setting specified
  /// subtarget options.  Definition of function is auto generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
  SparcSubtarget &initializeSubtargetDependencies(StringRef CPU,
                                                  StringRef TuneCPU,
                                                  StringRef FS);

```
- **EN**: Implements logic around `ParseSubtargetFeatures`, `initializeSubtargetDependencies`.
- **CN**: 围绕 `ParseSubtargetFeatures`, `initializeSubtargetDependencies` 实现具体逻辑。

### Lines 81-86
```cpp
  /// The 64-bit ABI uses biased stack and frame pointers, so the stack frame
  /// of the current function is the area from [%sp+BIAS] to [%fp+BIAS].
  int64_t getStackPointerBias() const {
    return is64Bit() ? 2047 : 0;
  }

```
- **EN**: Implements logic around `getStackPointerBias`, `is64Bit`; this block returns target-specific results.
- **CN**: 围绕 `getStackPointerBias`, `is64Bit` 实现具体逻辑；这一段返回目标相关结果。

### Lines 87-91
```cpp
  bool isRegisterReserved(MCPhysReg PhysReg) const {
    return ReserveRegister[PhysReg];
  }

  /// Given a actual stack size as determined by FrameInfo, this function
```
- **EN**: Implements logic around `isRegisterReserved`; this block returns target-specific results.
- **CN**: 围绕 `isRegisterReserved` 实现具体逻辑；这一段返回目标相关结果。

### Lines 92-96
```cpp
  /// returns adjusted framesize which includes space for register window
  /// spills and arguments.
  int getAdjustedFrameSize(int stackSize) const;
};

```
- **EN**: Implements logic around `getAdjustedFrameSize`.
- **CN**: 围绕 `getAdjustedFrameSize` 实现具体逻辑。

### Lines 97-99
```cpp
} // end namespace llvm

#endif
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Subtarget features / 子目标特性**:
  - **EN**: Tracks CPU variants and feature bits
  - **CN**: 跟踪 CPU 变体与特性位
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/SparcMCTargetDesc.h`, `SparcFrameLowering.h`, `SparcISelLowering.h`, `SparcInstrInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/DataLayout.h`, `llvm/Support/ErrorHandling.h`, `llvm/TargetParser/Triple.h`, `SparcGenSubtargetInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
- **Generated macros / 生成宏**: `GET_SPARC_SPARCSUBTARGET_H`, `GET_SUBTARGETINFO_HEADER`, `GET_SUBTARGETINFO_MACRO`
