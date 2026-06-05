# XtensaSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaSubtarget.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Captures CPU features and per-function subtarget configuration.
  - **CN**: 封装 CPU 特性以及按函数区分的子目标配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XtensaSubtarget.h - Define Subtarget for the Xtensa ----*- C++ -*--===//
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
// This file declares the Xtensa specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp
#ifndef LLVM_LIB_TARGET_XTENSA_XTENSASUBTARGET_H
#define LLVM_LIB_TARGET_XTENSA_XTENSASUBTARGET_H

#include "XtensaFrameLowering.h"
#include "XtensaISelLowering.h"
#include "XtensaInstrInfo.h"
#include "XtensaRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/Target/TargetMachine.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaFrameLowering.h`, `XtensaISelLowering.h`, `XtensaInstrInfo.h`, `XtensaRegisterInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaFrameLowering.h`, `XtensaISelLowering.h`, `XtensaInstrInfo.h`, `XtensaRegisterInfo.h`。

### Lines 23-29
```cpp

#define GET_SUBTARGETINFO_HEADER
#include "XtensaGenSubtargetInfo.inc"

namespace llvm {
class StringRef;

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenSubtargetInfo.inc`。

### Lines 30-36
```cpp
class XtensaSubtarget : public XtensaGenSubtargetInfo {
private:
// Bool members corresponding to the SubtargetFeatures defined in tablegen
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool ATTRIBUTE = DEFAULT;
#include "XtensaGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenSubtargetInfo.inc`。

### Lines 37-42
```cpp
  const Triple &TargetTriple;
  XtensaInstrInfo InstrInfo;
  XtensaTargetLowering TLInfo;
  std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
  XtensaFrameLowering FrameLowering;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 43-48
```cpp
  XtensaSubtarget &initializeSubtargetDependencies(StringRef CPU, StringRef FS);

public:
  XtensaSubtarget(const Triple &TT, StringRef CPU, StringRef FS,
                  const TargetMachine &TM);

```
- **EN**: Implements logic around `initializeSubtargetDependencies`, `XtensaSubtarget`.
- **CN**: 围绕 `initializeSubtargetDependencies`, `XtensaSubtarget` 实现具体逻辑。

### Lines 49-58
```cpp
  ~XtensaSubtarget() override;

  const Triple &getTargetTriple() const { return TargetTriple; }

  const TargetFrameLowering *getFrameLowering() const override {
    return &FrameLowering;
  }
  const XtensaInstrInfo *getInstrInfo() const override { return &InstrInfo; }
  const XtensaRegisterInfo *getRegisterInfo() const override {
    return &InstrInfo.getRegisterInfo();
```
- **EN**: Implements logic around `~XtensaSubtarget`, `getTargetTriple`, `getFrameLowering`, `getInstrInfo`, ...; this block returns target-specific results.
- **CN**: 围绕 `~XtensaSubtarget`, `getTargetTriple`, `getFrameLowering`, `getInstrInfo`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 59-64
```cpp
  }

  const XtensaTargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }

```
- **EN**: Implements logic around `getTargetLowering`; this block returns target-specific results.
- **CN**: 围绕 `getTargetLowering` 实现具体逻辑；这一段返回目标相关结果。

### Lines 65-74
```cpp
  const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;

  bool hasDensity() const { return HasDensity; }
  bool hasMAC16() const { return HasMAC16; }
  bool hasWindowed() const { return HasWindowed; }
  bool hasBoolean() const { return HasBoolean; }
  bool hasLoop() const { return HasLoop; }
  bool hasSEXT() const { return HasSEXT; }
  bool hasCLAMPS() const { return HasCLAMPS; }
  bool hasNSA() const { return HasNSA; }
```
- **EN**: Implements logic around `getSelectionDAGInfo`, `hasDensity`, `hasMAC16`, `hasWindowed`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSelectionDAGInfo`, `hasDensity`, `hasMAC16`, `hasWindowed`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 75-84
```cpp
  bool hasMINMAX() const { return HasMINMAX; }
  bool hasMul16() const { return HasMul16; }
  bool hasMul32() const { return HasMul32; }
  bool hasMul32High() const { return HasMul32High; }
  bool hasDiv32() const { return HasDiv32; }
  bool hasS32C1I() const { return HasS32C1I; }
  bool hasForcedAtomics() const { return HasForcedAtomics; }
  bool hasSingleFloat() const { return HasSingleFloat; }
  bool hasRegionProtection() const { return HasRegionProtection; }
  bool hasRelocatableVector() const { return HasRelocatableVector; }
```
- **EN**: Implements logic around `hasMINMAX`, `hasMul16`, `hasMul32`, `hasMul32High`, ...; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `hasMINMAX`, `hasMul16`, `hasMul32`, `hasMul32High`, ... 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 85-94
```cpp
  bool hasMiscSR() const { return HasMiscSR; }
  bool hasExtendedL32R() const { return HasExtendedL32R; }
  bool hasDataCache() const { return HasDataCache; }
  bool hasHighPriInterrupts() const { return HasHighPriInterrupts; }
  bool hasHighPriInterruptsLevel3() const { return HasHighPriInterruptsLevel3; }
  bool hasHighPriInterruptsLevel4() const { return HasHighPriInterruptsLevel4; }
  bool hasHighPriInterruptsLevel5() const { return HasHighPriInterruptsLevel5; }
  bool hasHighPriInterruptsLevel6() const { return HasHighPriInterruptsLevel6; }
  bool hasHighPriInterruptsLevel7() const { return HasHighPriInterruptsLevel7; }
  bool hasInterrupt() const { return HasInterrupt; }
```
- **EN**: Implements logic around `hasMiscSR`, `hasExtendedL32R`, `hasDataCache`, `hasHighPriInterrupts`, ...; this block returns target-specific results.
- **CN**: 围绕 `hasMiscSR`, `hasExtendedL32R`, `hasDataCache`, `hasHighPriInterrupts`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 95-103
```cpp
  bool hasException() const { return HasException; }
  bool hasTHREADPTR() const { return HasTHREADPTR; }
  bool isWindowedABI() const { return hasWindowed(); }

  // Automatically generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
};
} // end namespace llvm

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 104-104
```cpp
#endif /* LLVM_LIB_TARGET_XTENSA_XTENSASUBTARGET_H */
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Subtarget features / 子目标特性**:
  - **EN**: Tracks CPU variants and feature bits
  - **CN**: 跟踪 CPU 变体与特性位
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaFrameLowering.h`, `XtensaISelLowering.h`, `XtensaInstrInfo.h`, `XtensaRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/DataLayout.h`, `llvm/Target/TargetMachine.h`, `XtensaGenSubtargetInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR
- **Generated macros / 生成宏**: `GET_SUBTARGETINFO_HEADER`, `GET_SUBTARGETINFO_MACRO`, `GET_XTENSA_XTENSASUBTARGET_H`
