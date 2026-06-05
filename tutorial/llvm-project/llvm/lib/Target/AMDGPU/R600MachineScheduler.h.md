# R600MachineScheduler.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600MachineScheduler.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for R600MachineScheduler in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 R600MachineScheduler 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===-- R600MachineScheduler.h - R600 Scheduler Interface -*- C++ -*-------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// R600 Machine Scheduler interface
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_R600MACHINESCHEDULER_H
#define LLVM_LIB_TARGET_AMDGPU_R600MACHINESCHEDULER_H

#include "llvm/CodeGen/MachineScheduler.h"
#include <vector>

namespace llvm {

class R600InstrInfo;
struct R600RegisterInfo;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `R600InstrInfo`, `R600RegisterInfo`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`R600InstrInfo`, `R600RegisterInfo`。

### Lines 25-37: Declares class R600SchedStrategy
```cpp
class R600SchedStrategy final : public MachineSchedStrategy {
  const ScheduleDAGMILive *DAG = nullptr;
  const R600InstrInfo *TII = nullptr;
  const R600RegisterInfo *TRI = nullptr;
  MachineRegisterInfo *MRI = nullptr;

  enum InstKind {
    IDAlu,
    IDFetch,
    IDOther,
    IDLast
  };

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600SchedStrategy`, `InstKind`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600SchedStrategy`, `InstKind`。

### Lines 38-61: Type declarations and aliases
```cpp
  enum AluKind {
    AluAny,
    AluT_X,
    AluT_Y,
    AluT_Z,
    AluT_W,
    AluT_XYZW,
    AluPredX,
    AluTrans,
    AluDiscarded, // LLVM Instructions that are going to be eliminated
    AluLast
  };

  std::vector<SUnit *> Available[IDLast], Pending[IDLast];
  std::vector<SUnit *> AvailableAlus[AluLast];
  std::vector<SUnit *> PhysicalRegCopy;

  InstKind CurInstKind;
  int CurEmitted;
  InstKind NextInstKind;

  unsigned AluInstCount;
  unsigned FetchInstCount;

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `AluKind`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`AluKind`。

### Lines 62-79: Declares R600SchedStrategy
```cpp
  int InstKindLimit[IDLast];

  int OccupiedSlotsMask;

public:
  R600SchedStrategy() = default;
  ~R600SchedStrategy() override = default;

  void initialize(ScheduleDAGMI *dag) override;
  SUnit *pickNode(bool &IsTopNode) override;
  void schedNode(SUnit *SU, bool IsTopNode) override;
  void releaseTopNode(SUnit *SU) override;
  void releaseBottomNode(SUnit *SU) override;

private:
  std::vector<MachineInstr *> InstructionsGroupCandidate;
  bool VLIW5;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 80-97: Preprocessor guards and macros
```cpp
  int getInstKind(SUnit *SU);
  bool regBelongsToClass(Register Reg, const TargetRegisterClass *RC) const;
  AluKind getAluKind(SUnit *SU) const;
  void LoadAlu();
  unsigned AvailablesAluCount() const;
  SUnit *AttemptFillSlot (unsigned Slot, bool AnyAlu);
  void PrepareNextSlot();
  SUnit *PopInst(std::vector<SUnit*> &Q, bool AnyALU);

  void AssignSlot(MachineInstr *MI, unsigned Slot);
  SUnit* pickAlu();
  SUnit* pickOther(int QID);
  void MoveUnits(std::vector<SUnit *> &QSrc, std::vector<SUnit *> &QDst);
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_R600MACHINESCHEDULER_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `R600InstrInfo`, `R600RegisterInfo`, `R600SchedStrategy`, `InstKind`, `AluKind`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; scheduling / 调度; SelectionDAG processing / SelectionDAG 处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/CodeGen/MachineScheduler.h"`
- `<vector>`
