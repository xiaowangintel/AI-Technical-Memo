# MSP430MachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430MachineFunctionInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Stores per-machine-function backend state used during lowering and instruction selection.
  - **CN**: 保存 lowering 和指令选择阶段使用的每个 MachineFunction 的后端状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//=== MSP430MachineFunctionInfo.h - MSP430 machine function info -*- C++ -*-==//
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
// This file declares MSP430-specific per-machine-function information.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 12-15
```cpp

#ifndef LLVM_LIB_TARGET_MSP430_MSP430MACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_MSP430_MSP430MACHINEFUNCTIONINFO_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 16-19
```cpp
#include "llvm/CodeGen/MachineFunction.h"

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/MachineFunction.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/MachineFunction.h`。

### Lines 20-24
```cpp
/// MSP430MachineFunctionInfo - This class is derived from MachineFunction and
/// contains private MSP430 target-specific information for each MachineFunction.
class MSP430MachineFunctionInfo : public MachineFunctionInfo {
  virtual void anchor();

```
- **EN**: Introduces declarations for `is`, `MSP430MachineFunctionInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `is`, `MSP430MachineFunctionInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 25-28
```cpp
  /// CalleeSavedFrameSize - Size of the callee-saved register portion of the
  /// stack frame in bytes.
  unsigned CalleeSavedFrameSize = 0;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 29-32
```cpp
  /// ReturnAddrIndex - FrameIndex for return slot.
  int ReturnAddrIndex = 0;

  /// VarArgsFrameIndex - FrameIndex for start of varargs area.
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 33-36
```cpp
  int VarArgsFrameIndex = 0;

  /// SRetReturnReg - Some subtargets require that sret lowering includes
  /// returning the value of the returned struct in a register. This field
```
- **EN**: Introduces declarations for `in`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `in` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 37-42
```cpp
  /// holds the virtual register into which the sret argument is passed.
  Register SRetReturnReg;

public:
  MSP430MachineFunctionInfo() = default;

```
- **EN**: Implements logic around `MSP430MachineFunctionInfo`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `MSP430MachineFunctionInfo` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 43-50
```cpp
  MSP430MachineFunctionInfo(const Function &F, const TargetSubtargetInfo *STI)
      : CalleeSavedFrameSize(0), ReturnAddrIndex(0), SRetReturnReg(0) {}

  MachineFunctionInfo *
  clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
        const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
      const override;

```
- **EN**: Implements logic around `MSP430MachineFunctionInfo`, `CalleeSavedFrameSize`, `clone`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `MSP430MachineFunctionInfo`, `CalleeSavedFrameSize`, `clone` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 51-56
```cpp
  unsigned getCalleeSavedFrameSize() const { return CalleeSavedFrameSize; }
  void setCalleeSavedFrameSize(unsigned bytes) { CalleeSavedFrameSize = bytes; }

  Register getSRetReturnReg() const { return SRetReturnReg; }
  void setSRetReturnReg(Register Reg) { SRetReturnReg = Reg; }

```
- **EN**: Implements logic around `getCalleeSavedFrameSize`, `setCalleeSavedFrameSize`, `getSRetReturnReg`, `setSRetReturnReg`; this block returns target-specific results.
- **CN**: 围绕 `getCalleeSavedFrameSize`, `setCalleeSavedFrameSize`, `getSRetReturnReg`, `setSRetReturnReg` 实现具体逻辑；这一段返回目标相关结果。

### Lines 57-63
```cpp
  int getRAIndex() const { return ReturnAddrIndex; }
  void setRAIndex(int Index) { ReturnAddrIndex = Index; }

  int getVarArgsFrameIndex() const { return VarArgsFrameIndex;}
  void setVarArgsFrameIndex(int Index) { VarArgsFrameIndex = Index; }
};

```
- **EN**: Implements logic around `getRAIndex`, `setRAIndex`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex`; this block returns target-specific results.
- **CN**: 围绕 `getRAIndex`, `setRAIndex`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex` 实现具体逻辑；这一段返回目标相关结果。

### Lines 64-66
```cpp
} // End llvm namespace

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
- **Generated macros / 生成宏**: `GET_MSP430_MSP430MACHINEFUNCTIONINFO_H`
