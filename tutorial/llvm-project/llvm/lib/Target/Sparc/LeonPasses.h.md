# LeonPasses.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/LeonPasses.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines SPARC/LEON-specific optimization or cleanup passes and their registration glue.
  - **CN**: 定义 SPARC/LEON 专用优化或清理 Pass 及其注册胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------- LeonPasses.h - Define passes specific to LEON ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-14
```cpp
//
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_SPARC_LEON_PASSES_H
#define LLVM_LIB_TARGET_SPARC_LEON_PASSES_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 15-19
```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"

namespace llvm {
class SparcSubtarget;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/MachineFunctionPass.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/MachineFunctionPass.h`。

### Lines 20-25
```cpp
class LLVM_LIBRARY_VISIBILITY LEONMachineFunctionPass
    : public MachineFunctionPass {
protected:
  const SparcSubtarget *Subtarget = nullptr;
  const int LAST_OPERAND = -1;

```
- **EN**: Introduces declarations for `LLVM_LIBRARY_VISIBILITY`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVM_LIBRARY_VISIBILITY` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 26-32
```cpp
  // this vector holds free registers that we allocate in groups for some of the
  // LEON passes
  std::vector<int> UsedRegisters;

protected:
  LEONMachineFunctionPass(char &ID);

```
- **EN**: Implements logic around `LEONMachineFunctionPass`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LEONMachineFunctionPass` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 33-39
```cpp
  void clearUsedRegisterList() { UsedRegisters.clear(); }

  void markRegisterUsed(int registerIndex) {
    UsedRegisters.push_back(registerIndex);
  }
};

```
- **EN**: Implements logic around `clearUsedRegisterList`, `markRegisterUsed`, `push_back`.
- **CN**: 围绕 `clearUsedRegisterList`, `markRegisterUsed`, `push_back` 实现具体逻辑。

### Lines 40-45
```cpp
class LLVM_LIBRARY_VISIBILITY ErrataWorkaround : public MachineFunctionPass {
protected:
  const SparcSubtarget *ST;
  const TargetInstrInfo *TII;
  const TargetRegisterInfo *TRI;

```
- **EN**: Introduces declarations for `LLVM_LIBRARY_VISIBILITY`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVM_LIBRARY_VISIBILITY` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 46-52
```cpp
  bool checkSeqTN0009A(MachineBasicBlock::iterator I);
  bool checkSeqTN0009B(MachineBasicBlock::iterator I);
  bool checkSeqTN0010First(MachineBasicBlock &MBB);
  bool checkSeqTN0010(MachineBasicBlock::iterator I);
  bool checkSeqTN0012(MachineBasicBlock::iterator I);
  bool checkSeqTN0013(MachineBasicBlock::iterator I);

```
- **EN**: Implements logic around `checkSeqTN0009A`, `checkSeqTN0009B`, `checkSeqTN0010First`, `checkSeqTN0010`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `checkSeqTN0009A`, `checkSeqTN0009B`, `checkSeqTN0010First`, `checkSeqTN0010`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 53-57
```cpp
  bool moveNext(MachineBasicBlock::iterator &I);
  bool isFloat(MachineBasicBlock::iterator I);
  bool isDivSqrt(MachineBasicBlock::iterator I);
  void insertNop(MachineBasicBlock::iterator I);

```
- **EN**: Implements logic around `moveNext`, `isFloat`, `isDivSqrt`, `insertNop`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `moveNext`, `isFloat`, `isDivSqrt`, `insertNop` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 58-63
```cpp
public:
  static char ID;

  ErrataWorkaround();
  bool runOnMachineFunction(MachineFunction &MF) override;

```
- **EN**: Implements logic around `ErrataWorkaround`, `runOnMachineFunction`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `ErrataWorkaround`, `runOnMachineFunction` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 64-70
```cpp
  StringRef getPassName() const override { return "Errata workaround pass"; };
};

class LLVM_LIBRARY_VISIBILITY InsertNOPLoad : public LEONMachineFunctionPass {
public:
  static char ID;

```
- **EN**: Introduces declarations for `LLVM_LIBRARY_VISIBILITY`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVM_LIBRARY_VISIBILITY` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 71-80
```cpp
  InsertNOPLoad();
  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "InsertNOPLoad: Erratum Fix LBR35: insert a NOP instruction after "
           "every single-cycle load instruction when the next instruction is "
           "another load/store instruction";
  }
};

```
- **EN**: Implements logic around `InsertNOPLoad`, `runOnMachineFunction`, `getPassName`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `InsertNOPLoad`, `runOnMachineFunction`, `getPassName` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 81-85
```cpp
class LLVM_LIBRARY_VISIBILITY DetectRoundChange
    : public LEONMachineFunctionPass {
public:
  static char ID;

```
- **EN**: Introduces declarations for `LLVM_LIBRARY_VISIBILITY`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVM_LIBRARY_VISIBILITY` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 86-94
```cpp
  DetectRoundChange();
  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "DetectRoundChange: Leon erratum detection: detect any rounding "
           "mode change request: use only the round-to-nearest rounding mode";
  }
};

```
- **EN**: Implements logic around `DetectRoundChange`, `runOnMachineFunction`, `getPassName`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `DetectRoundChange`, `runOnMachineFunction`, `getPassName` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 95-101
```cpp
class LLVM_LIBRARY_VISIBILITY FixAllFDIVSQRT : public LEONMachineFunctionPass {
public:
  static char ID;

  FixAllFDIVSQRT();
  bool runOnMachineFunction(MachineFunction &MF) override;

```
- **EN**: Introduces declarations for `LLVM_LIBRARY_VISIBILITY`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVM_LIBRARY_VISIBILITY` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 102-108
```cpp
  StringRef getPassName() const override {
    return "FixAllFDIVSQRT: Erratum Fix LBR34: fix FDIVS/FDIVD/FSQRTS/FSQRTD "
           "instructions with NOPs and floating-point store";
  }
};
} // namespace llvm

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 109-109
```cpp
#endif // LLVM_LIB_TARGET_SPARC_LEON_PASSES_H
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/CodeGen/MachineFunctionPass.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_SPARC_LEON_PASSES_H`
