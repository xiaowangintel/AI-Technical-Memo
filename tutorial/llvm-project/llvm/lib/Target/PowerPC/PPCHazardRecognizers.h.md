# PPCHazardRecognizers.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCHazardRecognizers.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCHazardRecognizers.h - PowerPC Hazard Recognizers.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCHazardRecognizers.h`，主要负责 PowerPC 后端的后端的公共或内部声明。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCHazardRecognizers.h - PowerPC Hazard Recognizers -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file defines hazard recognizers for scheduling on PowerPC processors.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file defines hazard recognizers for scheduling on PowerPC processors.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file defines hazard recognizers for scheduling on PowerPC processors.”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCHAZARDRECOGNIZERS_H
#define LLVM_LIB_TARGET_POWERPC_PPCHAZARDRECOGNIZERS_H

#include "PPCInstrInfo.h"
#include "llvm/CodeGen/ScheduleHazardRecognizer.h"
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段包含调度或处理器模型元数据。

### Lines 18-24

```cpp
#include "llvm/CodeGen/ScoreboardHazardRecognizer.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"

namespace llvm {

/// PPCDispatchGroupSBHazardRecognizer - This class implements a scoreboard-based
/// hazard recognizer for PPC ooo processors with dispatch-group hazards.
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 25-50

```cpp
class PPCDispatchGroupSBHazardRecognizer : public ScoreboardHazardRecognizer {
  const ScheduleDAG *DAG;
  SmallVector<SUnit *, 7> CurGroup;
  unsigned CurSlots, CurBranches;

  bool isLoadAfterStore(SUnit *SU);
  bool isBCTRAfterSet(SUnit *SU);
  bool mustComeFirst(const MCInstrDesc *MCID, unsigned &NSlots) const;

public:
  PPCDispatchGroupSBHazardRecognizer(const InstrItineraryData *ItinData,
                         const ScheduleDAG *DAG_) :
    ScoreboardHazardRecognizer(ItinData, DAG_), DAG(DAG_),
    CurSlots(0), CurBranches(0) {}

  HazardType getHazardType(SUnit *SU, int Stalls) override;
  bool ShouldPreferAnother(SUnit *SU) const override;
  unsigned PreEmitNoops(SUnit *SU) override;
  void EmitInstruction(SUnit *SU) override;
  void AdvanceCycle() override;
  void RecedeCycle() override;
  void Reset() override;
  void EmitNoop() override;
};

/// PPCHazardRecognizer970 - This class defines a finite state automata that
```
- **EN**: Declares a backend-facing type `PPCDispatchGroupSBHazardRecognizer`, `isLoadAfterStore`, `isBCTRAfterSet` and outlines the API or state that nearby code will rely on. The logic interacts with LLVM's MC layer.
- **CN**: 这里声明面向后端的类型 `PPCDispatchGroupSBHazardRecognizer`, `isLoadAfterStore`, `isBCTRAfterSet`，并勾勒出周边代码会依赖的接口或状态。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 51-76

```cpp
/// models the dispatch logic on the PowerPC 970 (aka G5) processor.  This
/// promotes good dispatch group formation and implements noop insertion to
/// avoid structural hazards that cause significant performance penalties (e.g.
/// setting the CTR register then branching through it within a dispatch group),
/// or storing then loading from the same address within a dispatch group.
class PPCHazardRecognizer970 : public ScheduleHazardRecognizer {
  const ScheduleDAG &DAG;

  unsigned NumIssued;  // Number of insts issued, including advanced cycles.

  // Various things that can cause a structural hazard.

  // HasCTRSet - If the CTR register is set in this group, disallow BCTRL.
  bool HasCTRSet;

  // StoredPtr - Keep track of the address of any store.  If we see a load from
  // the same address (or one that aliases it), disallow the store.  We can have
  // up to four stores in one dispatch group, hence we track up to 4.
  //
  // This is null if we haven't seen a store yet.  We keep track of both
  // operands of the store here, since we support [r+r] and [r+i] addressing.
  const Value *StoreValue[4];
  int64_t StoreOffset[4];
  uint64_t StoreSize[4];
  unsigned NumStores;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "models the dispatch logic on the PowerPC 970 (aka G5) processor.  This". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“models the dispatch logic on the PowerPC 970 (aka G5) processor.  This”。 这一段包含调度或处理器模型元数据。

### Lines 77-100

```cpp
public:
  PPCHazardRecognizer970(const ScheduleDAG &DAG);
  HazardType getHazardType(SUnit *SU, int Stalls) override;
  void EmitInstruction(SUnit *SU) override;
  void AdvanceCycle() override;
  void Reset() override;

private:
  /// EndDispatchGroup - Called when we are finishing a new dispatch group.
  ///
  void EndDispatchGroup();

  /// GetInstrType - Classify the specified powerpc opcode according to its
  /// pipeline.
  PPCII::PPC970_Unit GetInstrType(unsigned Opcode,
                                  bool &isFirst, bool &isSingle,bool &isCracked,
                                  bool &isLoad, bool &isStore);

  bool isLoadOfStoredAddress(uint64_t LoadSize, int64_t LoadOffset,
                             const Value *LoadValue) const;
};

} // end namespace llvm
```
- **EN**: Declares function entry points including `PPCHazardRecognizer970`, `getHazardType`, `EmitInstruction` that other backend components call later. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `PPCHazardRecognizer970`, `getHazardType`, `EmitInstruction`。 这一段包含调度或处理器模型元数据。

### Lines 101-102

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Public or internal declarations for the backend / 后端的公共或内部声明
- SelectionDAG lowering / SelectionDAG lowering
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCInstrInfo.h`
- `llvm/CodeGen/ScheduleHazardRecognizer.h`
- `llvm/CodeGen/ScoreboardHazardRecognizer.h`
- `llvm/CodeGen/SelectionDAGNodes.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
