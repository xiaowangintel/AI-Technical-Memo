# XCoreMachineFunctionInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreMachineFunctionInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Stores per-machine-function backend state used during lowering and instruction selection.
  - **CN**: 保存 lowering 和指令选择阶段使用的每个 MachineFunction 的后端状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XCoreMachineFunctionInfo.cpp - XCore machine function info --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-13
```cpp

#include "XCoreMachineFunctionInfo.h"
#include "XCoreInstrInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/Function.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreMachineFunctionInfo.h`, `XCoreInstrInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/Function.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreMachineFunctionInfo.h`, `XCoreInstrInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/Function.h`。

### Lines 14-23
```cpp
using namespace llvm;

void XCoreFunctionInfo::anchor() { }

MachineFunctionInfo *XCoreFunctionInfo::clone(
    BumpPtrAllocator &Allocator, MachineFunction &DestMF,
    const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
    const {
  return DestMF.cloneInfo<XCoreFunctionInfo>(*this);
}
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 24-33
```cpp

bool XCoreFunctionInfo::isLargeFrame(const MachineFunction &MF) const {
  if (CachedEStackSize == -1) {
    CachedEStackSize = MF.getFrameInfo().estimateStackSize(MF);
  }
  // isLargeFrame() is used when deciding if spill slots should be added to
  // allow eliminateFrameIndex() to scavenge registers.
  // This is only required when there is no FP and offsets are greater than
  // ~256KB (~64Kwords). Thus only for code run on the emulator!
  //
```
- **EN**: Implements logic around `isLargeFrame`, `getFrameInfo`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLargeFrame`, `getFrameInfo` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 34-40
```cpp
  // The arbitrary value of 0xf000 allows frames of up to ~240KB before spill
  // slots are added for the use of eliminateFrameIndex() register scavenging.
  // For frames less than 240KB, it is assumed that there will be less than
  // 16KB of function arguments.
  return CachedEStackSize > 0xf000;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 41-50
```cpp
int XCoreFunctionInfo::createLRSpillSlot(MachineFunction &MF) {
  if (LRSpillSlotSet) {
    return LRSpillSlot;
  }
  const TargetRegisterClass &RC = XCore::GRRegsRegClass;
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  if (! MF.getFunction().isVarArg()) {
    // A fixed offset of 0 allows us to save / restore LR using entsp / retsp.
    LRSpillSlot = MFI.CreateFixedObject(TRI.getSpillSize(RC), 0, true);
```
- **EN**: Implements logic around `createLRSpillSlot`, `getSubtarget`, `getFrameInfo`, `CreateFixedObject`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createLRSpillSlot`, `getSubtarget`, `getFrameInfo`, `CreateFixedObject` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 51-58
```cpp
  } else {
    LRSpillSlot = MFI.CreateStackObject(TRI.getSpillSize(RC),
                                        TRI.getSpillAlign(RC), true);
  }
  LRSpillSlotSet = true;
  return LRSpillSlot;
}

```
- **EN**: Implements logic around `CreateStackObject`, `getSpillAlign`; this block returns target-specific results.
- **CN**: 围绕 `CreateStackObject`, `getSpillAlign` 实现具体逻辑；这一段返回目标相关结果。

### Lines 59-68
```cpp
int XCoreFunctionInfo::createFPSpillSlot(MachineFunction &MF) {
  if (FPSpillSlotSet) {
    return FPSpillSlot;
  }
  const TargetRegisterClass &RC = XCore::GRRegsRegClass;
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  FPSpillSlot =
      MFI.CreateStackObject(TRI.getSpillSize(RC), TRI.getSpillAlign(RC), true);
  FPSpillSlotSet = true;
```
- **EN**: Implements logic around `createFPSpillSlot`, `getSubtarget`, `getFrameInfo`, `CreateStackObject`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createFPSpillSlot`, `getSubtarget`, `getFrameInfo`, `CreateStackObject` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 69-78
```cpp
  return FPSpillSlot;
}

const int* XCoreFunctionInfo::createEHSpillSlot(MachineFunction &MF) {
  if (EHSpillSlotSet) {
    return EHSpillSlot;
  }
  const TargetRegisterClass &RC = XCore::GRRegsRegClass;
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
  MachineFrameInfo &MFI = MF.getFrameInfo();
```
- **EN**: Implements logic around `createEHSpillSlot`, `getSubtarget`, `getFrameInfo`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createEHSpillSlot`, `getSubtarget`, `getFrameInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 79-86
```cpp
  unsigned Size = TRI.getSpillSize(RC);
  Align Alignment = TRI.getSpillAlign(RC);
  EHSpillSlot[0] = MFI.CreateStackObject(Size, Alignment, true);
  EHSpillSlot[1] = MFI.CreateStackObject(Size, Alignment, true);
  EHSpillSlotSet = true;
  return EHSpillSlot;
}

```
- **EN**: Implements logic around `getSpillSize`, `getSpillAlign`, `CreateStackObject`; this block returns target-specific results.
- **CN**: 围绕 `getSpillSize`, `getSpillAlign`, `CreateStackObject` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCoreMachineFunctionInfo.h`, `XCoreInstrInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/Function.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR
