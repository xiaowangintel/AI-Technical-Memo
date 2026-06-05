# SparcFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcFrameLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements stack frame layout plus prologue/epilogue emission for the backend.
  - **CN**: 实现该后端的栈帧布局以及序言/尾声生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcFrameLowering.h - Define frame lowering for Sparc --*- C++ -*-===//
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
//
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 12-15
```cpp

#ifndef LLVM_LIB_TARGET_SPARC_SPARCFRAMELOWERING_H
#define LLVM_LIB_TARGET_SPARC_SPARCFRAMELOWERING_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 16-19
```cpp
#include "Sparc.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/Support/TypeSize.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `Sparc.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `Sparc.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`。

### Lines 20-26
```cpp
namespace llvm {

class SparcSubtarget;
class SparcFrameLowering : public TargetFrameLowering {
public:
  explicit SparcFrameLowering(const SparcSubtarget &ST);

```
- **EN**: Introduces declarations for `llvm`, `SparcSubtarget`, `SparcFrameLowering`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `SparcSubtarget`, `SparcFrameLowering` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 27-31
```cpp
  /// emitProlog/emitEpilog - These methods insert prolog and epilog code into
  /// the function.
  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;

```
- **EN**: Implements logic around `emitPrologue`, `emitEpilogue`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitPrologue`, `emitEpilogue` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 32-36
```cpp
  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF,
                                MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator I) const override;

```
- **EN**: Implements logic around `eliminateCallFramePseudoInstr`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateCallFramePseudoInstr` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 37-40
```cpp
  bool hasReservedCallFrame(const MachineFunction &MF) const override;
  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS = nullptr) const override;

```
- **EN**: Implements logic around `hasReservedCallFrame`, `determineCalleeSaves`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasReservedCallFrame`, `determineCalleeSaves` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 41-44
```cpp
  StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
                                     Register &FrameReg) const override;

  /// targetHandlesStackFrameRounding - Returns true if the target is
```
- **EN**: Implements logic around `getFrameIndexReference`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameIndexReference` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 45-48
```cpp
  /// responsible for rounding up the stack frame (probably at emitPrologue
  /// time).
  bool targetHandlesStackFrameRounding() const override { return true; }

```
- **EN**: Implements logic around `targetHandlesStackFrameRounding`; this block returns target-specific results.
- **CN**: 围绕 `targetHandlesStackFrameRounding` 实现具体逻辑；这一段返回目标相关结果。

### Lines 49-55
```cpp
protected:
  bool hasFPImpl(const MachineFunction &MF) const override;

private:
  // Remap input registers to output registers for leaf procedure.
  void remapRegsForLeafProc(MachineFunction &MF) const;

```
- **EN**: Implements logic around `hasFPImpl`, `remapRegsForLeafProc`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasFPImpl`, `remapRegsForLeafProc` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 56-59
```cpp
  // Returns true if MF is a leaf procedure.
  bool isLeafProc(MachineFunction &MF) const;


```
- **EN**: Implements logic around `isLeafProc`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLeafProc` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 60-65
```cpp
  // Emits code for adjusting SP in function prologue/epilogue.
  void emitSPAdjustment(MachineFunction &MF,
                        MachineBasicBlock &MBB,
                        MachineBasicBlock::iterator MBBI,
                        int NumBytes, unsigned ADDrr, unsigned ADDri) const;

```
- **EN**: Implements logic around `emitSPAdjustment`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitSPAdjustment` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 66-69
```cpp
};

} // End llvm namespace

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 70-70
```cpp
#endif
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Stack frame management / 栈帧管理**:
  - **EN**: Controls prologue/epilogue emission and frame layout
  - **CN**: 控制序言尾声生成与栈帧布局
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `Sparc.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
- **Generated macros / 生成宏**: `GET_SPARC_SPARCFRAMELOWERING_H`
