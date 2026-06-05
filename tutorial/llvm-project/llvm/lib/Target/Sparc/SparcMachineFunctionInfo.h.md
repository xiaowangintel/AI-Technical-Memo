# SparcMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcMachineFunctionInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Stores per-machine-function backend state used during lowering and instruction selection.
  - **CN**: 保存 lowering 和指令选择阶段使用的每个 MachineFunction 的后端状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- SparcMachineFunctionInfo.h - Sparc Machine Function Info -*- C++ -*-===//
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
// This file declares  Sparc specific per-machine-function information.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 12-16
```cpp
#ifndef LLVM_LIB_TARGET_SPARC_SPARCMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_SPARC_SPARCMACHINEFUNCTIONINFO_H

#include "llvm/CodeGen/MachineFunction.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/MachineFunction.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/MachineFunction.h`。

### Lines 17-23
```cpp
namespace llvm {

  class SparcMachineFunctionInfo : public MachineFunctionInfo {
    virtual void anchor();
  private:
    Register GlobalBaseReg;

```
- **EN**: Introduces declarations for `llvm`, `SparcMachineFunctionInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `SparcMachineFunctionInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 24-27
```cpp
    /// VarArgsFrameOffset - Frame offset to start of varargs area.
    int VarArgsFrameOffset;

    /// SRetReturnReg - Holds the virtual register into which the sret
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 28-31
```cpp
    /// argument is passed.
    Register SRetReturnReg;

    /// IsLeafProc - True if the function is a leaf procedure.
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 32-39
```cpp
    bool IsLeafProc;
  public:
    SparcMachineFunctionInfo()
      : GlobalBaseReg(0), VarArgsFrameOffset(0), SRetReturnReg(0),
        IsLeafProc(false) {}
    SparcMachineFunctionInfo(const Function &F, const TargetSubtargetInfo *STI)
        : GlobalBaseReg(0), VarArgsFrameOffset(0), SRetReturnReg(0),
          IsLeafProc(false) {}
```
- **EN**: Implements logic around `SparcMachineFunctionInfo`, `GlobalBaseReg`, `IsLeafProc`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `SparcMachineFunctionInfo`, `GlobalBaseReg`, `IsLeafProc` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 40-45
```cpp

    MachineFunctionInfo *
    clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
          const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
        const override;

```
- **EN**: Implements logic around `clone`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `clone` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 46-51
```cpp
    Register getGlobalBaseReg() const { return GlobalBaseReg; }
    void setGlobalBaseReg(Register Reg) { GlobalBaseReg = Reg; }

    int getVarArgsFrameOffset() const { return VarArgsFrameOffset; }
    void setVarArgsFrameOffset(int Offset) { VarArgsFrameOffset = Offset; }

```
- **EN**: Implements logic around `getGlobalBaseReg`, `setGlobalBaseReg`, `getVarArgsFrameOffset`, `setVarArgsFrameOffset`; this block returns target-specific results.
- **CN**: 围绕 `getGlobalBaseReg`, `setGlobalBaseReg`, `getVarArgsFrameOffset`, `setVarArgsFrameOffset` 实现具体逻辑；这一段返回目标相关结果。

### Lines 52-59
```cpp
    Register getSRetReturnReg() const { return SRetReturnReg; }
    void setSRetReturnReg(Register Reg) { SRetReturnReg = Reg; }

    void setLeafProc(bool rhs) { IsLeafProc = rhs; }
    bool isLeafProc() const { return IsLeafProc; }
  };
}

```
- **EN**: Implements logic around `getSRetReturnReg`, `setSRetReturnReg`, `setLeafProc`, `isLeafProc`; this block returns target-specific results.
- **CN**: 围绕 `getSRetReturnReg`, `setSRetReturnReg`, `setLeafProc`, `isLeafProc` 实现具体逻辑；这一段返回目标相关结果。

### Lines 60-60
```cpp
#endif
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/CodeGen/MachineFunction.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_SPARC_SPARCMACHINEFUNCTIONINFO_H`
