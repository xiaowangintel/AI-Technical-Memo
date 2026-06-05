# GCNIterativeScheduler.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNIterativeScheduler.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for GCNIterativeScheduler in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 GCNIterativeScheduler 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===- GCNIterativeScheduler.h - GCN Scheduler ------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines the class GCNIterativeScheduler, which uses an iterative
/// approach to find a best schedule for GCN architecture. It basically makes
/// use of various lightweight schedules, scores them, chooses best one based on
/// their scores, and finally implements the chosen one.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_GCNITERATIVESCHEDULER_H
#define LLVM_LIB_TARGET_AMDGPU_GCNITERATIVESCHEDULER_H

#include "GCNRegPressure.h"
#include "llvm/CodeGen/MachineScheduler.h"

namespace llvm {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 25-44: Declares class MachineInstr
```cpp
class MachineInstr;
class SUnit;
class raw_ostream;

class GCNIterativeScheduler : public ScheduleDAGMILive {
  using BaseClass = ScheduleDAGMILive;

public:
  enum StrategyKind {
    SCHEDULE_MINREGONLY,
    SCHEDULE_MINREGFORCED,
    SCHEDULE_LEGACYMAXOCCUPANCY,
    SCHEDULE_ILP
  };

  GCNIterativeScheduler(MachineSchedContext *C,
                        StrategyKind S);

  void schedule() override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MachineInstr`, `SUnit`, `raw_ostream`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MachineInstr`, `SUnit`, `raw_ostream`。

### Lines 45-68: Declares struct TentativeSchedule
```cpp
  void enterRegion(MachineBasicBlock *BB,
                   MachineBasicBlock::iterator Begin,
                   MachineBasicBlock::iterator End,
                   unsigned RegionInstrs) override;

  void finalizeSchedule() override;

protected:
  using ScheduleRef = ArrayRef<const SUnit *>;

  struct TentativeSchedule {
    std::vector<MachineInstr *> Schedule;
    GCNRegPressure MaxPressure;
  };

  struct Region {
    // Fields except for BestSchedule are supposed to reflect current IR state
    // `const` fields are to emphasize they shouldn't change for any schedule.
    MachineBasicBlock::iterator Begin;
    // End is either a boundary instruction or end of basic block
    const MachineBasicBlock::iterator End;
    const unsigned NumRegionInstrs;
    GCNRegPressure MaxPressure;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `TentativeSchedule`, `Region`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`TentativeSchedule`, `Region`。

### Lines 69-91: Declares class BuildDAG
```cpp
    // best schedule for the region so far (not scheduled yet)
    std::unique_ptr<TentativeSchedule> BestSchedule;
  };

  SpecificBumpPtrAllocator<Region> Alloc;
  std::vector<Region*> Regions;

  MachineSchedContext *Context;
  const StrategyKind Strategy;
  mutable GCNUpwardRPTracker UPTracker;

  std::vector<std::unique_ptr<ScheduleDAGMutation>> SavedMutations;

  class BuildDAG;
  class OverrideLegacyStrategy;

  template <typename Range>
  GCNRegPressure getSchedulePressure(const Region &R,
                                     Range &&Schedule) const;

  GCNRegPressure getRegionPressure(MachineBasicBlock::iterator Begin,
                                   MachineBasicBlock::iterator End) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `BuildDAG`, `OverrideLegacyStrategy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`BuildDAG`, `OverrideLegacyStrategy`。

### Lines 92-113: Defines getRegionPressure
```cpp
  GCNRegPressure getRegionPressure(const Region &R) const {
    return getRegionPressure(R.Begin, R.End);
  }

  void swapIGLPMutations(const Region &R, bool IsReentry);
  void setBestSchedule(Region &R,
                       ScheduleRef Schedule,
                       const GCNRegPressure &MaxRP = GCNRegPressure());

  void scheduleBest(Region &R);

  std::vector<MachineInstr*> detachSchedule(ScheduleRef Schedule) const;

  void sortRegionsByPressure(unsigned TargetOcc);

  template <typename Range>
  void scheduleRegion(Region &R, Range &&Schedule,
                      const GCNRegPressure &MaxRP = GCNRegPressure());

  unsigned tryMaximizeOccupancy(unsigned TargetOcc =
                                std::numeric_limits<unsigned>::max());

```
**EN:** This section contains concrete logic for getRegionPressure. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getRegionPressure 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 114-129: Preprocessor guards and macros
```cpp
  void scheduleLegacyMaxOccupancy(bool TryMaximizeOccupancy = true);
  void scheduleMinReg(bool force = false);
  void scheduleILP(bool TryMaximizeOccupancy = true);

  void printRegions(raw_ostream &OS) const;
  void printSchedResult(raw_ostream &OS,
                        const Region *R,
                        const GCNRegPressure &RP) const;
  void printSchedRP(raw_ostream &OS,
                    const GCNRegPressure &Before,
                    const GCNRegPressure &After) const;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_GCNITERATIVESCHEDULER_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MachineInstr`, `SUnit`, `raw_ostream`, `GCNIterativeScheduler`, `StrategyKind`, `TentativeSchedule`
- **Main themes / 核心主题**: instruction semantics / 指令语义; scheduling / 调度; SelectionDAG processing / SelectionDAG 处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNRegPressure.h"`
- `"llvm/CodeGen/MachineScheduler.h"`
