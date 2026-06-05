# XtensaMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaMachineFunctionInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Stores per-machine-function backend state used during lowering and instruction selection.
  - **CN**: 保存 lowering 和指令选择阶段使用的每个 MachineFunction 的后端状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```cpp
//==- XtensaMachineFunctionInfo.h - Xtensa machine function info --*- C++ -*-=//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 9-13
```cpp
//===----------------------------------------------------------------------===//
//
// This file declares Xtensa-specific per-machine-function information.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 14-17
```cpp

#ifndef LLVM_LIB_TARGET_XTENSA_XTENSAMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_XTENSA_XTENSAMACHINEFUNCTIONINFO_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 18-21
```cpp
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/Target/TargetMachine.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/Target/TargetMachine.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/Target/TargetMachine.h`。

### Lines 22-25
```cpp
namespace llvm {

class XtensaMachineFunctionInfo : public MachineFunctionInfo {
  /// FrameIndex of the spill slot for the scratch register in BranchRelaxation.
```
- **EN**: Introduces declarations for `llvm`, `XtensaMachineFunctionInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `XtensaMachineFunctionInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 26-32
```cpp
  int BranchRelaxationScratchFrameIndex = -1;
  unsigned VarArgsFirstGPR;
  int VarArgsOnStackFrameIndex;
  int VarArgsInRegsFrameIndex;
  bool SaveFrameRegister = false;
  unsigned CPLabelId = 0;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 33-38
```cpp
public:
  explicit XtensaMachineFunctionInfo(const Function &F,
                                     const TargetSubtargetInfo *STI)
      : VarArgsFirstGPR(0), VarArgsOnStackFrameIndex(0),
        VarArgsInRegsFrameIndex(0) {}

```
- **EN**: Implements logic around `XtensaMachineFunctionInfo`, `VarArgsFirstGPR`, `VarArgsInRegsFrameIndex`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `XtensaMachineFunctionInfo`, `VarArgsFirstGPR`, `VarArgsInRegsFrameIndex` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 39-45
```cpp
  int getBranchRelaxationScratchFrameIndex() const {
    return BranchRelaxationScratchFrameIndex;
  }
  void setBranchRelaxationScratchFrameIndex(int Index) {
    BranchRelaxationScratchFrameIndex = Index;
  }

```
- **EN**: Implements logic around `getBranchRelaxationScratchFrameIndex`, `setBranchRelaxationScratchFrameIndex`; this block returns target-specific results.
- **CN**: 围绕 `getBranchRelaxationScratchFrameIndex`, `setBranchRelaxationScratchFrameIndex` 实现具体逻辑；这一段返回目标相关结果。

### Lines 46-51
```cpp
  unsigned getVarArgsFirstGPR() const { return VarArgsFirstGPR; }
  void setVarArgsFirstGPR(unsigned GPR) { VarArgsFirstGPR = GPR; }

  int getVarArgsOnStackFrameIndex() const { return VarArgsOnStackFrameIndex; }
  void setVarArgsOnStackFrameIndex(int FI) { VarArgsOnStackFrameIndex = FI; }

```
- **EN**: Implements logic around `getVarArgsFirstGPR`, `setVarArgsFirstGPR`, `getVarArgsOnStackFrameIndex`, `setVarArgsOnStackFrameIndex`; this block returns target-specific results.
- **CN**: 围绕 `getVarArgsFirstGPR`, `setVarArgsFirstGPR`, `getVarArgsOnStackFrameIndex`, `setVarArgsOnStackFrameIndex` 实现具体逻辑；这一段返回目标相关结果。

### Lines 52-55
```cpp
  // Get and set the frame index of the first stack vararg.
  int getVarArgsInRegsFrameIndex() const { return VarArgsInRegsFrameIndex; }
  void setVarArgsInRegsFrameIndex(int FI) { VarArgsInRegsFrameIndex = FI; }

```
- **EN**: Implements logic around `getVarArgsInRegsFrameIndex`, `setVarArgsInRegsFrameIndex`; this block returns target-specific results.
- **CN**: 围绕 `getVarArgsInRegsFrameIndex`, `setVarArgsInRegsFrameIndex` 实现具体逻辑；这一段返回目标相关结果。

### Lines 56-61
```cpp
  bool isSaveFrameRegister() const { return SaveFrameRegister; }
  void setSaveFrameRegister() { SaveFrameRegister = true; }

  unsigned createCPLabelId() { return CPLabelId++; }
};

```
- **EN**: Implements logic around `isSaveFrameRegister`, `setSaveFrameRegister`, `createCPLabelId`; this block returns target-specific results.
- **CN**: 围绕 `isSaveFrameRegister`, `setSaveFrameRegister`, `createCPLabelId` 实现具体逻辑；这一段返回目标相关结果。

### Lines 62-64
```cpp
} // namespace llvm

#endif /* LLVM_LIB_TARGET_XTENSA_XTENSAMACHINEFUNCTIONINFO_H */
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/Target/TargetMachine.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_XTENSA_XTENSAMACHINEFUNCTIONINFO_H`
