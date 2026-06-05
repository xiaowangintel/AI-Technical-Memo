# XCoreMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreMachineFunctionInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Stores per-machine-function backend state used during lowering and instruction selection.
  - **CN**: 保存 lowering 和指令选择阶段使用的每个 MachineFunction 的后端状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- XCoreMachineFunctionInfo.h - XCore machine function info -*- C++ -*-===//
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
// This file declares XCore-specific per-machine-function information.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp
#ifndef LLVM_LIB_TARGET_XCORE_XCOREMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_XCORE_XCOREMACHINEFUNCTIONINFO_H

#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include <cassert>
#include <utility>
#include <vector>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `cassert`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `cassert`。

### Lines 23-32
```cpp
namespace llvm {

/// XCoreFunctionInfo - This class is derived from MachineFunction private
/// XCore target-specific information for each MachineFunction.
class XCoreFunctionInfo : public MachineFunctionInfo {
  bool LRSpillSlotSet = false;
  int LRSpillSlot;
  bool FPSpillSlotSet = false;
  int FPSpillSlot;
  bool EHSpillSlotSet = false;
```
- **EN**: Introduces declarations for `llvm`, `is`, `XCoreFunctionInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `is`, `XCoreFunctionInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 33-40
```cpp
  int EHSpillSlot[2];
  unsigned ReturnStackOffset;
  bool ReturnStackOffsetSet = false;
  int VarArgsFrameIndex = 0;
  mutable int CachedEStackSize = -1;
  std::vector<std::pair<MachineBasicBlock::iterator, CalleeSavedInfo>>
  SpillLabels;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 41-45
```cpp
  virtual void anchor();

public:
  XCoreFunctionInfo() = default;

```
- **EN**: Implements logic around `anchor`, `XCoreFunctionInfo`.
- **CN**: 围绕 `anchor`, `XCoreFunctionInfo` 实现具体逻辑。

### Lines 46-53
```cpp
  explicit XCoreFunctionInfo(const Function &F,
                             const TargetSubtargetInfo *STI) {}

  MachineFunctionInfo *
  clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
        const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
      const override;

```
- **EN**: Implements logic around `XCoreFunctionInfo`, `clone`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `XCoreFunctionInfo`, `clone` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 54-58
```cpp
  ~XCoreFunctionInfo() override = default;

  void setVarArgsFrameIndex(int off) { VarArgsFrameIndex = off; }
  int getVarArgsFrameIndex() const { return VarArgsFrameIndex; }

```
- **EN**: Implements logic around `~XCoreFunctionInfo`, `setVarArgsFrameIndex`, `getVarArgsFrameIndex`; this block returns target-specific results.
- **CN**: 围绕 `~XCoreFunctionInfo`, `setVarArgsFrameIndex`, `getVarArgsFrameIndex` 实现具体逻辑；这一段返回目标相关结果。

### Lines 59-65
```cpp
  int createLRSpillSlot(MachineFunction &MF);
  bool hasLRSpillSlot() { return LRSpillSlotSet; }
  int getLRSpillSlot() const {
    assert(LRSpillSlotSet && "LR Spill slot not set");
    return LRSpillSlot;
  }

```
- **EN**: Implements logic around `createLRSpillSlot`, `hasLRSpillSlot`, `getLRSpillSlot`, `assert`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createLRSpillSlot`, `hasLRSpillSlot`, `getLRSpillSlot`, `assert` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 66-72
```cpp
  int createFPSpillSlot(MachineFunction &MF);
  bool hasFPSpillSlot() { return FPSpillSlotSet; }
  int getFPSpillSlot() const {
    assert(FPSpillSlotSet && "FP Spill slot not set");
    return FPSpillSlot;
  }

```
- **EN**: Implements logic around `createFPSpillSlot`, `hasFPSpillSlot`, `getFPSpillSlot`, `assert`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createFPSpillSlot`, `hasFPSpillSlot`, `getFPSpillSlot`, `assert` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 73-79
```cpp
  const int* createEHSpillSlot(MachineFunction &MF);
  bool hasEHSpillSlot() { return EHSpillSlotSet; }
  const int* getEHSpillSlot() const {
    assert(EHSpillSlotSet && "EH Spill slot not set");
    return EHSpillSlot;
  }

```
- **EN**: Implements logic around `createEHSpillSlot`, `hasEHSpillSlot`, `getEHSpillSlot`, `assert`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createEHSpillSlot`, `hasEHSpillSlot`, `getEHSpillSlot`, `assert` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 80-85
```cpp
  void setReturnStackOffset(unsigned value) {
    assert(!ReturnStackOffsetSet && "Return stack offset set twice");
    ReturnStackOffset = value;
    ReturnStackOffsetSet = true;
  }

```
- **EN**: Implements logic around `setReturnStackOffset`, `assert`; this block returns target-specific results.
- **CN**: 围绕 `setReturnStackOffset`, `assert` 实现具体逻辑；这一段返回目标相关结果。

### Lines 86-90
```cpp
  unsigned getReturnStackOffset() const {
    assert(ReturnStackOffsetSet && "Return stack offset not set");
    return ReturnStackOffset;
  }

```
- **EN**: Implements logic around `getReturnStackOffset`, `assert`; this block returns target-specific results.
- **CN**: 围绕 `getReturnStackOffset`, `assert` 实现具体逻辑；这一段返回目标相关结果。

### Lines 91-98
```cpp
  bool isLargeFrame(const MachineFunction &MF) const;

  std::vector<std::pair<MachineBasicBlock::iterator, CalleeSavedInfo>> &
  getSpillLabels() {
    return SpillLabels;
  }
};

```
- **EN**: Implements logic around `isLargeFrame`, `getSpillLabels`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLargeFrame`, `getSpillLabels` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 99-101
```cpp
} // end namespace llvm

#endif // LLVM_LIB_TARGET_XCORE_XCOREMACHINEFUNCTIONINFO_H
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `cassert`, `utility`, `vector`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_XCORE_XCOREMACHINEFUNCTIONINFO_H`
