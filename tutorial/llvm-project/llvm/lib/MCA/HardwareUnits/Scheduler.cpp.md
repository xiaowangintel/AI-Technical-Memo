# Scheduler.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/MCA/HardwareUnits/Scheduler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: A scheduler for processor resource units and processor resource groups.
  - **CN**: 实现 llvm-mca 使用的调度资源与硬件单元模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===--------------------- Scheduler.cpp ------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// A scheduler for processor resource units and processor resource groups.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-21
```cpp

#include "llvm/MCA/HardwareUnits/Scheduler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

namespace llvm {
namespace mca {

#define DEBUG_TYPE "llvm-mca"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/MCA/HardwareUnits/Scheduler.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/MCA/HardwareUnits/Scheduler.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`。

### Lines 22-39
```cpp
void Scheduler::initializeStrategy(std::unique_ptr<SchedulerStrategy> S) {
  // Ensure we have a valid (non-null) strategy object.
  Strategy = S ? std::move(S) : std::make_unique<DefaultSchedulerStrategy>();
}

// Anchor the vtable of SchedulerStrategy and DefaultSchedulerStrategy.
SchedulerStrategy::~SchedulerStrategy() = default;
DefaultSchedulerStrategy::~DefaultSchedulerStrategy() = default;

#ifndef NDEBUG
void Scheduler::dump() const {
  dbgs() << "[SCHEDULER]: WaitSet size is: " << WaitSet.size() << '\n';
  dbgs() << "[SCHEDULER]: ReadySet size is: " << ReadySet.size() << '\n';
  dbgs() << "[SCHEDULER]: IssuedSet size is: " << IssuedSet.size() << '\n';
  Resources->dump();
}
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 40-53
```cpp
Scheduler::Status Scheduler::isAvailable(const InstRef &IR) {
  ResourceStateEvent RSE =
      Resources->canBeDispatched(IR.getInstruction()->getUsedBuffers());
  HadTokenStall = RSE != RS_BUFFER_AVAILABLE;

  switch (RSE) {
  case ResourceStateEvent::RS_BUFFER_UNAVAILABLE:
    return Scheduler::SC_BUFFERS_FULL;
  case ResourceStateEvent::RS_RESERVED:
    return Scheduler::SC_DISPATCH_GROUP_STALL;
  case ResourceStateEvent::RS_BUFFER_AVAILABLE:
    break;
  }

```
- **EN**: Implements logic around `isAvailable`, `canBeDispatched`; this block uses `switch`-style dispatch; models machine-level execution behavior.
- **CN**: 围绕 `isAvailable`, `canBeDispatched` 实现具体逻辑；该代码块使用 `switch` 风格分派，并建模机器级执行行为。

### Lines 54-66
```cpp
  // Give lower priority to LSUnit stall events.
  LSUnit::Status LSS = LSU.isAvailable(IR);
  HadTokenStall = LSS != LSUnit::LSU_AVAILABLE;

  switch (LSS) {
  case LSUnit::LSU_LQUEUE_FULL:
    return Scheduler::SC_LOAD_QUEUE_FULL;
  case LSUnit::LSU_SQUEUE_FULL:
    return Scheduler::SC_STORE_QUEUE_FULL;
  case LSUnit::LSU_AVAILABLE:
    return Scheduler::SC_AVAILABLE;
  }

```
- **EN**: Implements logic around `isAvailable`; this block uses `switch`-style dispatch; models machine-level execution behavior.
- **CN**: 围绕 `isAvailable` 实现具体逻辑；该代码块使用 `switch` 风格分派，并建模机器级执行行为。

### Lines 67-79
```cpp
  llvm_unreachable("Don't know how to process this LSU state result!");
}

void Scheduler::issueInstructionImpl(
    InstRef &IR,
    SmallVectorImpl<std::pair<ResourceRef, ReleaseAtCycles>> &UsedResources) {
  Instruction *IS = IR.getInstruction();
  const InstrDesc &D = IS->getDesc();

  // Issue the instruction and collect all the consumed resources
  // into a vector. That vector is then used to notify the listener.
  Resources->issueInstruction(D, UsedResources);

```
- **EN**: Implements logic around `llvm_unreachable`, `issueInstructionImpl`, `getInstruction`, `getDesc`, and 1 more symbols; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `llvm_unreachable`, `issueInstructionImpl`, `getInstruction`, `getDesc`, and 1 more symbols 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 80-92
```cpp
  // Notify the instruction that it started executing.
  // This updates the internal state of each write.
  IS->execute(IR.getSourceIndex());

  IS->computeCriticalRegDep();

  if (IS->isMemOp()) {
    LSU.onInstructionIssued(IR);
    const CriticalDependency &MemDep =
        LSU.getCriticalPredecessor(IS->getLSUTokenID());
    IS->setCriticalMemDep(MemDep);
  }

```
- **EN**: Implements logic around `execute`, `computeCriticalRegDep`, `isMemOp`, `onInstructionIssued`, and 2 more symbols; this block emits or serializes data to an external representation; models machine-level execution behavior.
- **CN**: 围绕 `execute`, `computeCriticalRegDep`, `isMemOp`, `onInstructionIssued`, and 2 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并建模机器级执行行为。

### Lines 93-108
```cpp
  if (IS->isExecuting())
    IssuedSet.emplace_back(IR);
  else if (IS->isExecuted())
    LSU.onInstructionExecuted(IR);
}

// Release the buffered resources and issue the instruction.
void Scheduler::issueInstruction(
    InstRef &IR,
    SmallVectorImpl<std::pair<ResourceRef, ReleaseAtCycles>> &UsedResources,
    SmallVectorImpl<InstRef> &PendingInstructions,
    SmallVectorImpl<InstRef> &ReadyInstructions) {
  const Instruction &Inst = *IR.getInstruction();
  bool HasDependentUsers = Inst.hasDependentUsers();
  HasDependentUsers |= Inst.isMemOp() && LSU.hasDependentUsers(IR);

```
- **EN**: Implements logic around `isExecuting`, `emplace_back`, `isExecuted`, `onInstructionExecuted`, and 4 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `isExecuting`, `emplace_back`, `isExecuted`, `onInstructionExecuted`, and 4 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 109-119
```cpp
  Resources->releaseBuffers(Inst.getUsedBuffers());
  issueInstructionImpl(IR, UsedResources);
  // Instructions that have been issued during this cycle might have unblocked
  // other dependent instructions. Dependent instructions may be issued during
  // this same cycle if operands have ReadAdvance entries.  Promote those
  // instructions to the ReadySet and notify the caller that those are ready.
  if (HasDependentUsers)
    if (promoteToPendingSet(PendingInstructions))
      promoteToReadySet(ReadyInstructions);
}

```
- **EN**: Implements logic around `releaseBuffers`, `issueInstructionImpl`, `promoteToPendingSet`, `promoteToReadySet`; this block models machine-level execution behavior.
- **CN**: 围绕 `releaseBuffers`, `issueInstructionImpl`, `promoteToPendingSet`, `promoteToReadySet` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 120-139
```cpp
bool Scheduler::promoteToReadySet(SmallVectorImpl<InstRef> &Ready) {
  // Scan the set of waiting instructions and promote them to the
  // ready set if operands are all ready.
  unsigned PromotedElements = 0;
  for (auto I = PendingSet.begin(), E = PendingSet.end(); I != E;) {
    InstRef &IR = *I;
    if (!IR)
      break;

    // Check if there are unsolved register dependencies.
    Instruction &IS = *IR.getInstruction();
    if (!IS.isReady() && !IS.updatePending()) {
      ++I;
      continue;
    }
    // Check if there are unsolved memory dependencies.
    if (IS.isMemOp() && !LSU.isReady(IR)) {
      ++I;
      continue;
    }
```
- **EN**: Implements logic around `promoteToReadySet`, `begin`, `getInstruction`, `isReady`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `promoteToReadySet`, `begin`, `getInstruction`, `isReady`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 140-151
```cpp

    LLVM_DEBUG(dbgs() << "[SCHEDULER]: Instruction #" << IR
                      << " promoted to the READY set.\n");

    Ready.emplace_back(IR);
    ReadySet.emplace_back(IR);

    IR.invalidate();
    ++PromotedElements;
    std::iter_swap(I, E - PromotedElements);
  }

```
- **EN**: Implements logic around `dbgs`, `emplace_back`, `invalidate`, `iter_swap`; this block models machine-level execution behavior.
- **CN**: 围绕 `dbgs`, `emplace_back`, `invalidate`, `iter_swap` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 152-164
```cpp
  PendingSet.resize(PendingSet.size() - PromotedElements);
  return PromotedElements;
}

bool Scheduler::promoteToPendingSet(SmallVectorImpl<InstRef> &Pending) {
  // Scan the set of waiting instructions and promote them to the
  // pending set if operands are all ready.
  unsigned RemovedElements = 0;
  for (auto I = WaitSet.begin(), E = WaitSet.end(); I != E;) {
    InstRef &IR = *I;
    if (!IR)
      break;

```
- **EN**: Implements logic around `resize`, `promoteToPendingSet`, `begin`; this block models machine-level execution behavior.
- **CN**: 围绕 `resize`, `promoteToPendingSet`, `begin` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 165-177
```cpp
    // Check if this instruction is now ready. In case, force
    // a transition in state using method 'updateDispatched()'.
    Instruction &IS = *IR.getInstruction();
    if (IS.isDispatched() && !IS.updateDispatched()) {
      ++I;
      continue;
    }

    if (IS.isMemOp() && LSU.isWaiting(IR)) {
      ++I;
      continue;
    }

```
- **EN**: Implements logic around `getInstruction`, `isDispatched`, `isMemOp`; this block models machine-level execution behavior.
- **CN**: 围绕 `getInstruction`, `isDispatched`, `isMemOp` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 178-188
```cpp
    LLVM_DEBUG(dbgs() << "[SCHEDULER]: Instruction #" << IR
                      << " promoted to the PENDING set.\n");

    Pending.emplace_back(IR);
    PendingSet.emplace_back(IR);

    IR.invalidate();
    ++RemovedElements;
    std::iter_swap(I, E - RemovedElements);
  }

```
- **EN**: Implements logic around `dbgs`, `emplace_back`, `invalidate`, `iter_swap`; this block models machine-level execution behavior.
- **CN**: 围绕 `dbgs`, `emplace_back`, `invalidate`, `iter_swap` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 189-208
```cpp
  WaitSet.resize(WaitSet.size() - RemovedElements);
  return RemovedElements;
}

InstRef Scheduler::select() {
  unsigned QueueIndex = ReadySet.size();
  for (unsigned I = 0, E = ReadySet.size(); I != E; ++I) {
    InstRef &IR = ReadySet[I];
    if (QueueIndex == ReadySet.size() ||
        Strategy->compare(IR, ReadySet[QueueIndex])) {
      Instruction &IS = *IR.getInstruction();
      uint64_t BusyResourceMask = Resources->checkAvailability(IS.getDesc());
      if (BusyResourceMask)
        IS.setCriticalResourceMask(BusyResourceMask);
      BusyResourceUnits |= BusyResourceMask;
      if (!BusyResourceMask)
        QueueIndex = I;
    }
  }

```
- **EN**: Implements logic around `resize`, `select`, `size`, `compare`, and 3 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `resize`, `select`, `size`, `compare`, and 3 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 209-218
```cpp
  if (QueueIndex == ReadySet.size())
    return InstRef();

  // We found an instruction to issue.
  InstRef IR = ReadySet[QueueIndex];
  std::swap(ReadySet[QueueIndex], ReadySet[ReadySet.size() - 1]);
  ReadySet.pop_back();
  return IR;
}

```
- **EN**: Implements logic around `size`, `InstRef`, `swap`, `pop_back`; this block models machine-level execution behavior.
- **CN**: 围绕 `size`, `InstRef`, `swap`, `pop_back` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 219-232
```cpp
void Scheduler::updateIssuedSet(SmallVectorImpl<InstRef> &Executed) {
  unsigned RemovedElements = 0;
  for (auto I = IssuedSet.begin(), E = IssuedSet.end(); I != E;) {
    InstRef &IR = *I;
    if (!IR)
      break;
    Instruction &IS = *IR.getInstruction();
    if (!IS.isExecuted()) {
      LLVM_DEBUG(dbgs() << "[SCHEDULER]: Instruction #" << IR
                        << " is still executing.\n");
      ++I;
      continue;
    }

```
- **EN**: Implements logic around `updateIssuedSet`, `begin`, `getInstruction`, `isExecuted`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `updateIssuedSet`, `begin`, `getInstruction`, `isExecuted`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 233-243
```cpp
    // Instruction IR has completed execution.
    LSU.onInstructionExecuted(IR);
    Executed.emplace_back(IR);
    ++RemovedElements;
    IR.invalidate();
    std::iter_swap(I, E - RemovedElements);
  }

  IssuedSet.resize(IssuedSet.size() - RemovedElements);
}

```
- **EN**: Implements logic around `onInstructionExecuted`, `emplace_back`, `invalidate`, `iter_swap`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `onInstructionExecuted`, `emplace_back`, `invalidate`, `iter_swap`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 244-256
```cpp
uint64_t Scheduler::analyzeResourcePressure(SmallVectorImpl<InstRef> &Insts) {
  llvm::append_range(Insts, ReadySet);
  return BusyResourceUnits;
}

void Scheduler::analyzeDataDependencies(SmallVectorImpl<InstRef> &RegDeps,
                                        SmallVectorImpl<InstRef> &MemDeps) {
  const auto EndIt = PendingSet.end() - NumDispatchedToThePendingSet;
  for (const InstRef &IR : make_range(PendingSet.begin(), EndIt)) {
    const Instruction &IS = *IR.getInstruction();
    if (Resources->checkAvailability(IS.getDesc()))
      continue;

```
- **EN**: Implements logic around `analyzeResourcePressure`, `append_range`, `analyzeDataDependencies`, `end`, and 3 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `analyzeResourcePressure`, `append_range`, `analyzeDataDependencies`, `end`, and 3 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 257-270
```cpp
    if (IS.isMemOp() && LSU.isPending(IR))
      MemDeps.emplace_back(IR);

    if (IS.isPending())
      RegDeps.emplace_back(IR);
  }
}

void Scheduler::cycleEvent(SmallVectorImpl<ResourceRef> &Freed,
                           SmallVectorImpl<InstRef> &Executed,
                           SmallVectorImpl<InstRef> &Pending,
                           SmallVectorImpl<InstRef> &Ready) {
  LSU.cycleEvent();

```
- **EN**: Implements logic around `isMemOp`, `emplace_back`, `isPending`, `cycleEvent`; this block models machine-level execution behavior.
- **CN**: 围绕 `isMemOp`, `emplace_back`, `isPending`, `cycleEvent` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 271-280
```cpp
  // Release consumed resources.
  Resources->cycleEvent(Freed);

  for (InstRef &IR : IssuedSet)
    IR.getInstruction()->cycleEvent();
  updateIssuedSet(Executed);

  for (InstRef &IR : PendingSet)
    IR.getInstruction()->cycleEvent();

```
- **EN**: Implements logic around `cycleEvent`, `getInstruction`, `updateIssuedSet`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `cycleEvent`, `getInstruction`, `updateIssuedSet` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 281-290
```cpp
  for (InstRef &IR : WaitSet)
    IR.getInstruction()->cycleEvent();

  promoteToPendingSet(Pending);
  promoteToReadySet(Ready);

  NumDispatchedToThePendingSet = 0;
  BusyResourceUnits = 0;
}

```
- **EN**: Implements logic around `getInstruction`, `promoteToPendingSet`, `promoteToReadySet`; this block models machine-level execution behavior.
- **CN**: 围绕 `getInstruction`, `promoteToPendingSet`, `promoteToReadySet` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 291-300
```cpp
bool Scheduler::mustIssueImmediately(const InstRef &IR) const {
  const InstrDesc &Desc = IR.getInstruction()->getDesc();
  if (Desc.isZeroLatency())
    return true;
  // Instructions that use an in-order dispatch/issue processor resource must be
  // issued immediately to the pipeline(s). Any other in-order buffered
  // resources (i.e. BufferSize=1) is consumed.
  return Desc.MustIssueImmediately;
}

```
- **EN**: Implements logic around `mustIssueImmediately`, `getInstruction`, `isZeroLatency`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `mustIssueImmediately`, `getInstruction`, `isZeroLatency` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 301-314
```cpp
bool Scheduler::dispatch(InstRef &IR) {
  Instruction &IS = *IR.getInstruction();
  Resources->reserveBuffers(IS.getUsedBuffers());

  // If necessary, reserve queue entries in the load-store unit (LSU).
  if (IS.isMemOp())
    IS.setLSUTokenID(LSU.dispatch(IR));

  if (IS.isDispatched() || (IS.isMemOp() && LSU.isWaiting(IR))) {
    LLVM_DEBUG(dbgs() << "[SCHEDULER] Adding #" << IR << " to the WaitSet\n");
    WaitSet.push_back(IR);
    return false;
  }

```
- **EN**: Implements logic around `dispatch`, `getInstruction`, `reserveBuffers`, `isMemOp`, and 4 more symbols; this block manipulates DWARF/debug-info concepts; models machine-level execution behavior.
- **CN**: 围绕 `dispatch`, `getInstruction`, `reserveBuffers`, `isMemOp`, and 4 more symbols 实现具体逻辑；该代码块处理 DWARF/调试信息概念，并建模机器级执行行为。

### Lines 315-334
```cpp
  if (IS.isPending() || (IS.isMemOp() && LSU.isPending(IR))) {
    LLVM_DEBUG(dbgs() << "[SCHEDULER] Adding #" << IR
                      << " to the PendingSet\n");
    PendingSet.push_back(IR);
    ++NumDispatchedToThePendingSet;
    return false;
  }

  assert(IS.isReady() && (!IS.isMemOp() || LSU.isReady(IR)) &&
         "Unexpected internal state found!");
  // Don't add a zero-latency instruction to the Ready queue.
  // A zero-latency instruction doesn't consume any scheduler resources. That is
  // because it doesn't need to be executed, and it is often removed at register
  // renaming stage. For example, register-register moves are often optimized at
  // register renaming stage by simply updating register aliases. On some
  // targets, zero-idiom instructions (for example: a xor that clears the value
  // of a register) are treated specially, and are often eliminated at register
  // renaming stage.
  if (!mustIssueImmediately(IR)) {
    LLVM_DEBUG(dbgs() << "[SCHEDULER] Adding #" << IR << " to the ReadySet\n");
```
- **EN**: Implements logic around `isPending`, `dbgs`, `push_back`, `assert`, and 1 more symbols; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `isPending`, `dbgs`, `push_back`, `assert`, and 1 more symbols 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 335-342
```cpp
    ReadySet.push_back(IR);
  }

  return true;
}

} // namespace mca
} // namespace llvm
```
- **EN**: Introduces declarations for `mca`, `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mca`, `llvm` 等声明，建立本文件后续使用的类型或命名空间。

## Key Concepts / 关键概念

- **Machine-code analysis / 机器码分析**:
  - **EN**: Models pipeline behavior and instruction scheduling for llvm-mca.
  - **CN**: 为 llvm-mca 建模流水线行为与指令调度。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/MCA/HardwareUnits/Scheduler.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (2), llvm-mca simulation interfaces / llvm-mca 模拟接口 (1)
