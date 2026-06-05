# XtensaSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaSubtarget.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Captures CPU features and per-function subtarget configuration.
  - **CN**: 封装 CPU 特性以及按函数区分的子目标配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- XtensaSubtarget.cpp - Xtensa Subtarget Information -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-11
```cpp
//
// This file implements the Xtensa specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 12-17
```cpp

#include "XtensaSubtarget.h"
#include "XtensaSelectionDAGInfo.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaSubtarget.h`, `XtensaSelectionDAGInfo.h`, `llvm/IR/GlobalValue.h`, `llvm/Support/Debug.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaSubtarget.h`, `XtensaSelectionDAGInfo.h`, `llvm/IR/GlobalValue.h`, `llvm/Support/Debug.h`。

### Lines 18-23
```cpp
#define DEBUG_TYPE "xtensa-subtarget"

#define GET_SUBTARGETINFO_TARGET_DESC
#define GET_SUBTARGETINFO_CTOR
#include "XtensaGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenSubtargetInfo.inc`。

### Lines 24-31
```cpp
using namespace llvm;

XtensaSubtarget &
XtensaSubtarget::initializeSubtargetDependencies(StringRef CPU, StringRef FS) {
  StringRef CPUName = CPU;
  if (CPUName.empty()) {
    // set default cpu name
    CPUName = "generic";
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 32-38
```cpp
  }

  // Parse features string.
  ParseSubtargetFeatures(CPUName, CPUName, FS);
  return *this;
}

```
- **EN**: Implements logic around `ParseSubtargetFeatures`; this block returns target-specific results.
- **CN**: 围绕 `ParseSubtargetFeatures` 实现具体逻辑；这一段返回目标相关结果。

### Lines 39-46
```cpp
XtensaSubtarget::XtensaSubtarget(const Triple &TT, StringRef CPU, StringRef FS,
                                 const TargetMachine &TM)
    : XtensaGenSubtargetInfo(TT, CPU, /*TuneCPU=*/CPU, FS), TargetTriple(TT),
      InstrInfo(initializeSubtargetDependencies(CPU, FS)), TLInfo(TM, *this),
      FrameLowering(*this) {
  TSInfo = std::make_unique<SelectionDAGTargetInfo>();
}

```
- **EN**: Implements logic around `XtensaSubtarget`, `XtensaGenSubtargetInfo`, `InstrInfo`, `FrameLowering`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `XtensaSubtarget`, `XtensaGenSubtargetInfo`, `InstrInfo`, `FrameLowering`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 47-51
```cpp
XtensaSubtarget::~XtensaSubtarget() = default;

const SelectionDAGTargetInfo *XtensaSubtarget::getSelectionDAGInfo() const {
  return TSInfo.get();
}
```
- **EN**: Implements logic around `~XtensaSubtarget`, `getSelectionDAGInfo`, `get`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `~XtensaSubtarget`, `getSelectionDAGInfo`, `get` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

## Key Concepts / 关键概念

- **Subtarget features / 子目标特性**:
  - **EN**: Tracks CPU variants and feature bits
  - **CN**: 跟踪 CPU 变体与特性位
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaSubtarget.h`, `XtensaSelectionDAGInfo.h`, `llvm/IR/GlobalValue.h`, `llvm/Support/Debug.h`, `XtensaGenSubtargetInfo.inc`
- **LLVM subsystems / LLVM 子系统**: IR, Support
- **Generated macros / 生成宏**: `GET_SUBTARGETINFO_CTOR`, `GET_SUBTARGETINFO_TARGET_DESC`
