# ARMHazardRecognizer.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMHazardRecognizer.h`
- Repository: `llvm-project`
- Purpose (EN): This file defines hazard recognizers for scheduling ARM functions.
- 用途 (CN): 声明 ARM 后端中的 `ARMHazardRecognizer`，并提供与流水线冒险识别与调度约束相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMHazardRecognizer.h - ARM Hazard Recognizers ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines hazard recognizers for scheduling ARM functions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMHAZARDRECOGNIZER_H
#define LLVM_LIB_TARGET_ARM_ARMHAZARDRECOGNIZER_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-21
```cpp
#include "ARMBaseInstrInfo.h"
#include "llvm/ADT/BitmaskEnum.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/ScheduleHazardRecognizer.h"
#include "llvm/Support/DataTypes.h"
#include <initializer_list>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 25-28
```cpp
class DataLayout;
class MachineFunction;
class MachineInstr;
class ScheduleDAG;
```
- EN: Declares `DataLayout`, packaging target-specific state and APIs around `ARMHazardRecognizer`.
- CN: 这里声明 `DataLayout`，把与 `ARMHazardRecognizer` 相关的目标特定状态和 API 组织在一起。

### Lines 30-33
```cpp
// Hazards related to FP MLx instructions
class ARMHazardRecognizerFPMLx : public ScheduleHazardRecognizer {
  MachineInstr *LastMI = nullptr;
  unsigned FpMLxStalls = 0;
```
- EN: Declares `ARMHazardRecognizerFPMLx`, packaging target-specific state and APIs around `ARMHazardRecognizer`.
- CN: 这里声明 `ARMHazardRecognizerFPMLx`，把与 `ARMHazardRecognizer` 相关的目标特定状态和 API 组织在一起。

### Lines 35-36
```cpp
public:
  ARMHazardRecognizerFPMLx() { MaxLookAhead = 1; }
```
- EN: Implements `ARMHazardRecognizerFPMLx`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMHazardRecognizerFPMLx`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-43
```cpp
  HazardType getHazardType(SUnit *SU, int Stalls) override;
  void Reset() override;
  void EmitInstruction(SUnit *SU) override;
  void AdvanceCycle() override;
  void RecedeCycle() override;
};
```
- EN: Declares `getHazardType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getHazardType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 45-51
```cpp
// Hazards related to bank conflicts
class ARMBankConflictHazardRecognizer : public ScheduleHazardRecognizer {
  SmallVector<MachineInstr *, 8> Accesses;
  const MachineFunction &MF;
  const DataLayout &DL;
  int64_t DataMask;
  bool AssumeITCMBankConflict;
```
- EN: Declares `ARMBankConflictHazardRecognizer`, packaging target-specific state and APIs around `ARMHazardRecognizer`.
- CN: 这里声明 `ARMBankConflictHazardRecognizer`，把与 `ARMHazardRecognizer` 相关的目标特定状态和 API 组织在一起。

### Lines 53-60
```cpp
public:
  ARMBankConflictHazardRecognizer(const ScheduleDAG *DAG, int64_t DDM,
                                  bool ABC);
  HazardType getHazardType(SUnit *SU, int Stalls) override;
  void Reset() override;
  void EmitInstruction(SUnit *SU) override;
  void AdvanceCycle() override;
  void RecedeCycle() override;
```
- EN: Declares `ARMBankConflictHazardRecognizer`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMBankConflictHazardRecognizer`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-64
```cpp
private:
  inline HazardType CheckOffsets(unsigned O0, unsigned O1);
};
```
- EN: Declares `CheckOffsets`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CheckOffsets`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 66-66
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 68-68
```cpp
#endif
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: pipeline hazard recognition and scheduling constraints.
  - CN: 核心职责：流水线冒险识别与调度约束。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMBaseInstrInfo.h`.
  - CN: 后端本地头文件：`ARMBaseInstrInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/ScheduleHazardRecognizer.h`, `llvm/Support/DataTypes.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitmaskEnum.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/ScheduleHazardRecognizer.h`, `llvm/Support/DataTypes.h`。
- EN: Standard/system headers: `initializer_list`.
  - CN: 标准库/系统头文件：`initializer_list`。
