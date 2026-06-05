# LSUnit.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/MCA/HardwareUnits/LSUnit.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements machine scheduling resources and hardware unit models for llvm-mca.
  - **CN**: 实现 llvm-mca 使用的调度资源与硬件单元模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------- LSUnit.cpp --------------------------*- C++-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-18
```cpp
/// \file
///
/// A Load-Store Unit for the llvm-mca tool.
///
//===----------------------------------------------------------------------===//

#include "llvm/MCA/HardwareUnits/LSUnit.h"
#include "llvm/MCA/Instruction.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/MCA/HardwareUnits/LSUnit.h`, `llvm/MCA/Instruction.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/MCA/HardwareUnits/LSUnit.h`, `llvm/MCA/Instruction.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`。

### Lines 19-32
```cpp
#define DEBUG_TYPE "llvm-mca"

namespace llvm {
namespace mca {

LSUnitBase::LSUnitBase(const MCSchedModel &SM, unsigned LQ, unsigned SQ,
                       bool AssumeNoAlias)
    : LQSize(LQ), SQSize(SQ), UsedLQEntries(0), UsedSQEntries(0),
      NoAlias(AssumeNoAlias) {
  if (SM.hasExtraProcessorInfo()) {
    const MCExtraProcessorInfo &EPI = SM.getExtraProcessorInfo();
    if (!LQSize && EPI.LoadQueueID) {
      const MCProcResourceDesc &LdQDesc = *SM.getProcResource(EPI.LoadQueueID);
      LQSize = std::max(0, LdQDesc.BufferSize);
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 33-41
```cpp
    }

    if (!SQSize && EPI.StoreQueueID) {
      const MCProcResourceDesc &StQDesc = *SM.getProcResource(EPI.StoreQueueID);
      SQSize = std::max(0, StQDesc.BufferSize);
    }
  }
}

```
- **EN**: Implements logic around `getProcResource`, `max`; this block models machine-level execution behavior.
- **CN**: 围绕 `getProcResource`, `max` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 42-48
```cpp
LSUnitBase::~LSUnitBase() = default;

void LSUnit::cycleEvent() {
  for (const std::pair<unsigned, std::unique_ptr<MemoryGroup>> &G : Groups)
    G.second->cycleEvent();
}

```
- **EN**: Implements logic around `~LSUnitBase`, `cycleEvent`; this block models machine-level execution behavior.
- **CN**: 围绕 `~LSUnitBase`, `cycleEvent` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 49-62
```cpp
#ifndef NDEBUG
void LSUnit::dump() const {
  dbgs() << "[LSUnit] LQ_Size = " << getLoadQueueSize() << '\n';
  dbgs() << "[LSUnit] SQ_Size = " << getStoreQueueSize() << '\n';
  dbgs() << "[LSUnit] NextLQSlotIdx = " << getUsedLQEntries() << '\n';
  dbgs() << "[LSUnit] NextSQSlotIdx = " << getUsedSQEntries() << '\n';
  dbgs() << "\n";
  for (const auto &GroupIt : Groups) {
    const MemoryGroup &Group = *GroupIt.second;
    dbgs() << "[LSUnit] Group (" << GroupIt.first << "): "
           << "[ #Preds = " << Group.getNumPredecessors()
           << ", #GIssued = " << Group.getNumExecutingPredecessors()
           << ", #GExecuted = " << Group.getNumExecutedPredecessors()
           << ", #Inst = " << Group.getNumInstructions()
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 63-74
```cpp
           << ", #IIssued = " << Group.getNumExecuting()
           << ", #IExecuted = " << Group.getNumExecuted() << '\n';
  }
}
#endif

unsigned LSUnit::dispatch(const InstRef &IR) {
  const Instruction &IS = *IR.getInstruction();
  bool IsStoreBarrier = IS.isAStoreBarrier();
  bool IsLoadBarrier = IS.isALoadBarrier();
  assert((IS.getMayLoad() || IS.getMayStore()) && "Not a memory operation!");

```
- **EN**: Implements logic around `getNumExecuting`, `getNumExecuted`, `dispatch`, `getInstruction`, and 3 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `getNumExecuting`, `getNumExecuted`, `dispatch`, `getInstruction`, and 3 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 75-84
```cpp
  if (IS.getMayLoad())
    acquireLQSlot();
  if (IS.getMayStore())
    acquireSQSlot();

  if (IS.getMayStore()) {
    unsigned NewGID = createMemoryGroup();
    MemoryGroup &NewGroup = getGroup(NewGID);
    NewGroup.addInstruction();

```
- **EN**: Implements logic around `getMayLoad`, `acquireLQSlot`, `getMayStore`, `acquireSQSlot`, and 3 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `getMayLoad`, `acquireLQSlot`, `getMayStore`, `acquireSQSlot`, and 3 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 85-94
```cpp
    // A store may not pass a previous load or load barrier.
    unsigned ImmediateLoadDominator =
        std::max(CurrentLoadGroupID, CurrentLoadBarrierGroupID);
    if (ImmediateLoadDominator) {
      MemoryGroup &IDom = getGroup(ImmediateLoadDominator);
      LLVM_DEBUG(dbgs() << "[LSUnit]: GROUP DEP: (" << ImmediateLoadDominator
                        << ") --> (" << NewGID << ")\n");
      IDom.addSuccessor(&NewGroup, !assumeNoAlias());
    }

```
- **EN**: Implements logic around `max`, `getGroup`, `dbgs`, `addSuccessor`; this block models machine-level execution behavior.
- **CN**: 围绕 `max`, `getGroup`, `dbgs`, `addSuccessor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 95-103
```cpp
    // A store may not pass a previous store barrier.
    if (CurrentStoreBarrierGroupID) {
      MemoryGroup &StoreGroup = getGroup(CurrentStoreBarrierGroupID);
      LLVM_DEBUG(dbgs() << "[LSUnit]: GROUP DEP: ("
                        << CurrentStoreBarrierGroupID << ") --> (" << NewGID
                        << ")\n");
      StoreGroup.addSuccessor(&NewGroup, true);
    }

```
- **EN**: Implements logic around `getGroup`, `dbgs`, `addSuccessor`; this block models machine-level execution behavior.
- **CN**: 围绕 `getGroup`, `dbgs`, `addSuccessor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 104-112
```cpp
    // A store may not pass a previous store.
    if (CurrentStoreGroupID &&
        (CurrentStoreGroupID != CurrentStoreBarrierGroupID)) {
      MemoryGroup &StoreGroup = getGroup(CurrentStoreGroupID);
      LLVM_DEBUG(dbgs() << "[LSUnit]: GROUP DEP: (" << CurrentStoreGroupID
                        << ") --> (" << NewGID << ")\n");
      StoreGroup.addSuccessor(&NewGroup, !assumeNoAlias());
    }

```
- **EN**: Implements logic around `getGroup`, `dbgs`, `addSuccessor`; this block models machine-level execution behavior.
- **CN**: 围绕 `getGroup`, `dbgs`, `addSuccessor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 113-122
```cpp
    CurrentStoreGroupID = NewGID;
    if (IsStoreBarrier)
      CurrentStoreBarrierGroupID = NewGID;

    if (IS.getMayLoad()) {
      CurrentLoadGroupID = NewGID;
      if (IsLoadBarrier)
        CurrentLoadBarrierGroupID = NewGID;
    }

```
- **EN**: Implements logic around `getMayLoad`; this block models machine-level execution behavior.
- **CN**: 围绕 `getMayLoad` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 123-130
```cpp
    return NewGID;
  }

  assert(IS.getMayLoad() && "Expected a load!");

  unsigned ImmediateLoadDominator =
      std::max(CurrentLoadGroupID, CurrentLoadBarrierGroupID);

```
- **EN**: Implements logic around `assert`, `max`; this block models machine-level execution behavior.
- **CN**: 围绕 `assert`, `max` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 131-144
```cpp
  // A new load group is created if we are in one of the following situations:
  // 1) This is a load barrier (by construction, a load barrier is always
  //    assigned to a different memory group).
  // 2) There is no load in flight (by construction we always keep loads and
  //    stores into separate memory groups).
  // 3) There is a load barrier in flight. This load depends on it.
  // 4) There is an intervening store between the last load dispatched to the
  //    LSU and this load. We always create a new group even if this load
  //    does not alias the last dispatched store.
  // 5) There is no intervening store and there is an active load group.
  //    However that group has already started execution, so we cannot add
  //    this load to it.
  bool ShouldCreateANewGroup =
      IsLoadBarrier || !ImmediateLoadDominator ||
```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 145-153
```cpp
      CurrentLoadBarrierGroupID == ImmediateLoadDominator ||
      ImmediateLoadDominator <= CurrentStoreGroupID ||
      getGroup(ImmediateLoadDominator).isExecuting();

  if (ShouldCreateANewGroup) {
    unsigned NewGID = createMemoryGroup();
    MemoryGroup &NewGroup = getGroup(NewGID);
    NewGroup.addInstruction();

```
- **EN**: Implements logic around `getGroup`, `createMemoryGroup`, `addInstruction`; this block models machine-level execution behavior.
- **CN**: 围绕 `getGroup`, `createMemoryGroup`, `addInstruction` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 154-162
```cpp
    // A load may not pass a previous store or store barrier
    // unless flag 'NoAlias' is set.
    if (!assumeNoAlias() && CurrentStoreGroupID) {
      MemoryGroup &StoreGroup = getGroup(CurrentStoreGroupID);
      LLVM_DEBUG(dbgs() << "[LSUnit]: GROUP DEP: (" << CurrentStoreGroupID
                        << ") --> (" << NewGID << ")\n");
      StoreGroup.addSuccessor(&NewGroup, true);
    }

```
- **EN**: Implements logic around `assumeNoAlias`, `getGroup`, `dbgs`, `addSuccessor`; this block models machine-level execution behavior.
- **CN**: 围绕 `assumeNoAlias`, `getGroup`, `dbgs`, `addSuccessor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 163-176
```cpp
    // A load barrier may not pass a previous load or load barrier.
    if (IsLoadBarrier) {
      if (ImmediateLoadDominator) {
        MemoryGroup &LoadGroup = getGroup(ImmediateLoadDominator);
        LLVM_DEBUG(dbgs() << "[LSUnit]: GROUP DEP: (" << ImmediateLoadDominator
                          << ") --> (" << NewGID << ")\n");
        LoadGroup.addSuccessor(&NewGroup, true);
      }
    } else {
      // A younger load cannot pass a older load barrier.
      if (CurrentLoadBarrierGroupID) {
        MemoryGroup &LoadGroup = getGroup(CurrentLoadBarrierGroupID);
        LLVM_DEBUG(dbgs() << "[LSUnit]: GROUP DEP: ("
                          << CurrentLoadBarrierGroupID << ") --> (" << NewGID
```
- **EN**: Implements logic around `getGroup`, `dbgs`, `addSuccessor`; this block models machine-level execution behavior.
- **CN**: 围绕 `getGroup`, `dbgs`, `addSuccessor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 177-187
```cpp
                          << ")\n");
        LoadGroup.addSuccessor(&NewGroup, true);
      }
    }

    CurrentLoadGroupID = NewGID;
    if (IsLoadBarrier)
      CurrentLoadBarrierGroupID = NewGID;
    return NewGID;
  }

```
- **EN**: Implements logic around `addSuccessor`; this block models machine-level execution behavior.
- **CN**: 围绕 `addSuccessor` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 188-201
```cpp
  // A load may pass a previous load.
  MemoryGroup &Group = getGroup(CurrentLoadGroupID);
  Group.addInstruction();
  return CurrentLoadGroupID;
}

LSUnit::Status LSUnit::isAvailable(const InstRef &IR) const {
  const Instruction &IS = *IR.getInstruction();
  if (IS.getMayLoad() && isLQFull())
    return LSUnit::LSU_LQUEUE_FULL;
  if (IS.getMayStore() && isSQFull())
    return LSUnit::LSU_SQUEUE_FULL;
  return LSUnit::LSU_AVAILABLE;
}
```
- **EN**: Implements logic around `getGroup`, `addInstruction`, `isAvailable`, `getInstruction`, and 2 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `getGroup`, `addInstruction`, `isAvailable`, `getInstruction`, and 2 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 202-208
```cpp

void LSUnit::onInstructionRetired(const InstRef &IR) {
  const Instruction &IS = *IR.getInstruction();
  bool IsALoad = IS.getMayLoad();
  bool IsAStore = IS.getMayStore();
  assert((IsALoad || IsAStore) && "Expected a memory operation!");

```
- **EN**: Implements logic around `onInstructionRetired`, `getInstruction`, `getMayLoad`, `getMayStore`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `onInstructionRetired`, `getInstruction`, `getMayLoad`, `getMayStore`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 209-221
```cpp
  if (IsALoad) {
    releaseLQSlot();
    LLVM_DEBUG(dbgs() << "[LSUnit]: Instruction idx=" << IR.getSourceIndex()
                      << " has been removed from the load queue.\n");
  }

  if (IsAStore) {
    releaseSQSlot();
    LLVM_DEBUG(dbgs() << "[LSUnit]: Instruction idx=" << IR.getSourceIndex()
                      << " has been removed from the store queue.\n");
  }
}

```
- **EN**: Implements logic around `releaseLQSlot`, `dbgs`, `releaseSQSlot`; this block models machine-level execution behavior.
- **CN**: 围绕 `releaseLQSlot`, `dbgs`, `releaseSQSlot` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 222-233
```cpp
void LSUnit::onInstructionExecuted(const InstRef &IR) {
  const Instruction &IS = *IR.getInstruction();
  if (!IS.isMemOp())
    return;

  unsigned GroupID = IS.getLSUTokenID();
  auto It = Groups.find(GroupID);
  assert(It != Groups.end() && "Instruction not dispatched to the LS unit");
  It->second->onInstructionExecuted(IR);
  if (It->second->isExecuted())
    Groups.erase(It);

```
- **EN**: Implements logic around `onInstructionExecuted`, `getInstruction`, `isMemOp`, `getLSUTokenID`, and 4 more symbols; this block manipulates DWARF/debug-info concepts; models machine-level execution behavior.
- **CN**: 围绕 `onInstructionExecuted`, `getInstruction`, `isMemOp`, `getLSUTokenID`, and 4 more symbols 实现具体逻辑；该代码块处理 DWARF/调试信息概念，并建模机器级执行行为。

### Lines 234-245
```cpp
  if (!isValidGroupID(GroupID)) {
    if (GroupID == CurrentLoadGroupID)
      CurrentLoadGroupID = 0;
    if (GroupID == CurrentStoreGroupID)
      CurrentStoreGroupID = 0;
    if (GroupID == CurrentLoadBarrierGroupID)
      CurrentLoadBarrierGroupID = 0;
    if (GroupID == CurrentStoreBarrierGroupID)
      CurrentStoreBarrierGroupID = 0;
  }
}

```
- **EN**: Implements logic around `isValidGroupID`; this block models machine-level execution behavior.
- **CN**: 围绕 `isValidGroupID` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 246-247
```cpp
} // namespace mca
} // namespace llvm
```
- **EN**: Introduces declarations for `mca`, `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mca`, `llvm` 等声明，建立本文件后续使用的类型或命名空间。

## Key Concepts / 关键概念

- **Machine-code analysis / 机器码分析**:
  - **EN**: Models pipeline behavior and instruction scheduling for llvm-mca.
  - **CN**: 为 llvm-mca 建模流水线行为与指令调度。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/MCA/HardwareUnits/LSUnit.h`, `llvm/MCA/Instruction.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`
- **Subsystem categories / 子系统类别**: llvm-mca simulation interfaces / llvm-mca 模拟接口 (2), support-library helpers / Support 库辅助功能 (2)
