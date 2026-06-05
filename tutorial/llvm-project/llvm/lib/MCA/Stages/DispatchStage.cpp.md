# DispatchStage.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/MCA/Stages/DispatchStage.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements pipeline stages used by llvm-mca simulation.
  - **CN**: 实现 llvm-mca 模拟中的流水级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===--------------------- DispatchStage.cpp --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
/// \file
///
/// This file models the dispatch component of an instruction pipeline.
///
/// The DispatchStage is responsible for updating instruction dependencies
/// and communicating to the simulated instruction scheduler that an instruction
/// is ready to be scheduled for execution.
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-21
```cpp
///
//===----------------------------------------------------------------------===//

#include "llvm/MCA/Stages/DispatchStage.h"
#include "llvm/MCA/HWEventListener.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/MCA/Stages/DispatchStage.h`, `llvm/MCA/HWEventListener.h`, `llvm/Support/Debug.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/MCA/Stages/DispatchStage.h`, `llvm/MCA/HWEventListener.h`, `llvm/Support/Debug.h`。

### Lines 22-35
```cpp
#define DEBUG_TYPE "llvm-mca"

namespace llvm {
namespace mca {

DispatchStage::DispatchStage(const MCSubtargetInfo &Subtarget,
                             const MCRegisterInfo &MRI,
                             unsigned MaxDispatchWidth, RetireControlUnit &R,
                             RegisterFile &F)
    : DispatchWidth(MaxDispatchWidth), AvailableEntries(MaxDispatchWidth),
      CarryOver(0U), STI(Subtarget), RCU(R), PRF(F) {
  if (!DispatchWidth)
    DispatchWidth = Subtarget.getSchedModel().IssueWidth;
}
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 36-44
```cpp

void DispatchStage::notifyInstructionDispatched(const InstRef &IR,
                                                ArrayRef<unsigned> UsedRegs,
                                                unsigned UOps) const {
  LLVM_DEBUG(dbgs() << "[E] Instruction Dispatched: #" << IR << '\n');
  notifyEvent<HWInstructionEvent>(
      HWInstructionDispatchedEvent(IR, UsedRegs, UOps));
}

```
- **EN**: Implements logic around `notifyInstructionDispatched`, `dbgs`, `notifyEvent`, `HWInstructionDispatchedEvent`; this block models machine-level execution behavior.
- **CN**: 围绕 `notifyInstructionDispatched`, `dbgs`, `notifyEvent`, `HWInstructionDispatchedEvent` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 45-57
```cpp
bool DispatchStage::checkPRF(const InstRef &IR) const {
  SmallVector<MCPhysReg, 4> RegDefs;
  for (const WriteState &RegDef : IR.getInstruction()->getDefs())
    RegDefs.emplace_back(RegDef.getRegisterID());

  const unsigned RegisterMask = PRF.isAvailable(RegDefs);
  // A mask with all zeroes means: register files are available.
  if (RegisterMask) {
    notifyEvent<HWStallEvent>(
        HWStallEvent(HWStallEvent::RegisterFileStall, IR));
    return false;
  }

```
- **EN**: Implements logic around `checkPRF`, `getInstruction`, `emplace_back`, `isAvailable`, and 2 more symbols; this block emits or serializes data to an external representation; models machine-level execution behavior.
- **CN**: 围绕 `checkPRF`, `getInstruction`, `emplace_back`, `isAvailable`, and 2 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并建模机器级执行行为。

### Lines 58-69
```cpp
  return true;
}

bool DispatchStage::checkRCU(const InstRef &IR) const {
  const unsigned NumMicroOps = IR.getInstruction()->getNumMicroOps();
  if (RCU.isAvailable(NumMicroOps))
    return true;
  notifyEvent<HWStallEvent>(
      HWStallEvent(HWStallEvent::RetireControlUnitStall, IR));
  return false;
}

```
- **EN**: Implements logic around `checkRCU`, `getInstruction`, `isAvailable`, `notifyEvent`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `checkRCU`, `getInstruction`, `isAvailable`, `notifyEvent`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 70-76
```cpp
bool DispatchStage::canDispatch(const InstRef &IR) const {
  bool CanDispatch = checkRCU(IR);
  CanDispatch &= checkPRF(IR);
  CanDispatch &= checkNextStage(IR);
  return CanDispatch;
}

```
- **EN**: Implements logic around `canDispatch`, `checkRCU`, `checkPRF`, `checkNextStage`; this block models machine-level execution behavior.
- **CN**: 围绕 `canDispatch`, `checkRCU`, `checkPRF`, `checkNextStage` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 77-90
```cpp
Error DispatchStage::dispatch(InstRef IR) {
  assert(!CarryOver && "Cannot dispatch another instruction!");
  Instruction &IS = *IR.getInstruction();
  const unsigned NumMicroOps = IS.getNumMicroOps();
  if (NumMicroOps > DispatchWidth) {
    assert(AvailableEntries == DispatchWidth);
    AvailableEntries = 0;
    CarryOver = NumMicroOps - DispatchWidth;
    CarriedOver = IR;
  } else {
    assert(AvailableEntries >= NumMicroOps);
    AvailableEntries -= NumMicroOps;
  }

```
- **EN**: Implements logic around `dispatch`, `assert`, `getInstruction`, `getNumMicroOps`; this block propagates recoverable errors through LLVM error utilities; models machine-level execution behavior.
- **CN**: 围绕 `dispatch`, `assert`, `getInstruction`, `getNumMicroOps` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并建模机器级执行行为。

### Lines 91-99
```cpp
  // Check if this instructions ends the dispatch group.
  if (IS.getEndGroup())
    AvailableEntries = 0;

  // Check if this is an optimizable reg-reg move or an XCHG-like instruction.
  if (IS.isOptimizableMove())
    if (PRF.tryEliminateMoveOrSwap(IS.getDefs(), IS.getUses()))
      IS.setEliminated();

```
- **EN**: Implements logic around `getEndGroup`, `isOptimizableMove`, `tryEliminateMoveOrSwap`, `setEliminated`; this block models machine-level execution behavior.
- **CN**: 围绕 `getEndGroup`, `isOptimizableMove`, `tryEliminateMoveOrSwap`, `setEliminated` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 100-113
```cpp
  // A dependency-breaking instruction doesn't have to wait on the register
  // input operands, and it is often optimized at register renaming stage.
  // Update RAW dependencies if this instruction is not a dependency-breaking
  // instruction. A dependency-breaking instruction is a zero-latency
  // instruction that doesn't consume hardware resources.
  // An example of dependency-breaking instruction on X86 is a zero-idiom XOR.
  //
  // We also don't update data dependencies for instructions that have been
  // eliminated at register renaming stage.
  if (!IS.isEliminated()) {
    for (ReadState &RS : IS.getUses())
      PRF.addRegisterRead(RS, STI);
  }

```
- **EN**: Implements logic around `isEliminated`, `getUses`, `addRegisterRead`; this block parses or classifies structured input; models machine-level execution behavior.
- **CN**: 围绕 `isEliminated`, `getUses`, `addRegisterRead` 实现具体逻辑；该代码块解析或分类结构化输入，并建模机器级执行行为。

### Lines 114-120
```cpp
  // By default, a dependency-breaking zero-idiom is expected to be optimized
  // at register renaming stage. That means, no physical register is allocated
  // to the instruction.
  SmallVector<unsigned, 4> RegisterFiles(PRF.getNumRegisterFiles());
  for (WriteState &WS : IS.getDefs())
    PRF.addRegisterWrite(WriteRef(IR.getSourceIndex(), &WS), RegisterFiles);

```
- **EN**: Implements logic around `RegisterFiles`, `getDefs`, `addRegisterWrite`; this block emits or serializes data to an external representation; models machine-level execution behavior.
- **CN**: 围绕 `RegisterFiles`, `getDefs`, `addRegisterWrite` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并建模机器级执行行为。

### Lines 121-132
```cpp
  // Reserve entries in the reorder buffer.
  unsigned RCUTokenID = RCU.dispatch(IR);
  // Notify the instruction that it has been dispatched.
  IS.dispatch(RCUTokenID);

  // Notify listeners of the "instruction dispatched" event,
  // and move IR to the next stage.
  notifyInstructionDispatched(IR, RegisterFiles,
                              std::min(DispatchWidth, NumMicroOps));
  return moveToTheNextStage(IR);
}

```
- **EN**: Implements logic around `dispatch`, `notifyInstructionDispatched`, `min`, `moveToTheNextStage`; this block models machine-level execution behavior.
- **CN**: 围绕 `dispatch`, `notifyInstructionDispatched`, `min`, `moveToTheNextStage` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 133-140
```cpp
Error DispatchStage::cycleStart() {
  // The retire stage is responsible for calling method `cycleStart`
  // on the PRF.
  if (!CarryOver) {
    AvailableEntries = DispatchWidth;
    return ErrorSuccess();
  }

```
- **EN**: Implements logic around `cycleStart`, `ErrorSuccess`; this block propagates recoverable errors through LLVM error utilities; models machine-level execution behavior.
- **CN**: 围绕 `cycleStart`, `ErrorSuccess` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并建模机器级执行行为。

### Lines 141-152
```cpp
  AvailableEntries = CarryOver >= DispatchWidth ? 0 : DispatchWidth - CarryOver;
  unsigned DispatchedOpcodes = DispatchWidth - AvailableEntries;
  CarryOver -= DispatchedOpcodes;
  assert(CarriedOver && "Invalid dispatched instruction");

  SmallVector<unsigned, 8> RegisterFiles(PRF.getNumRegisterFiles(), 0U);
  notifyInstructionDispatched(CarriedOver, RegisterFiles, DispatchedOpcodes);
  if (!CarryOver)
    CarriedOver = InstRef();
  return ErrorSuccess();
}

```
- **EN**: Implements logic around `assert`, `RegisterFiles`, `notifyInstructionDispatched`, `InstRef`, and 1 more symbols; this block models machine-level execution behavior.
- **CN**: 围绕 `assert`, `RegisterFiles`, `notifyInstructionDispatched`, `InstRef`, and 1 more symbols 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 153-163
```cpp
bool DispatchStage::isAvailable(const InstRef &IR) const {
  // Conservatively bail out if there are no available dispatch entries.
  if (!AvailableEntries)
    return false;

  const Instruction &Inst = *IR.getInstruction();
  unsigned NumMicroOps = Inst.getNumMicroOps();
  unsigned Required = std::min(NumMicroOps, DispatchWidth);
  if (Required > AvailableEntries)
    return false;

```
- **EN**: Implements logic around `isAvailable`, `getInstruction`, `getNumMicroOps`, `min`; this block models machine-level execution behavior.
- **CN**: 围绕 `isAvailable`, `getInstruction`, `getNumMicroOps`, `min` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 164-172
```cpp
  if (Inst.getBeginGroup() && AvailableEntries != DispatchWidth)
    return false;

  // The dispatch logic doesn't internally buffer instructions.  It only accepts
  // instructions that can be successfully moved to the next stage during this
  // same cycle.
  return canDispatch(IR);
}

```
- **EN**: Implements logic around `getBeginGroup`, `canDispatch`; this block models machine-level execution behavior.
- **CN**: 围绕 `getBeginGroup`, `canDispatch` 实现具体逻辑；该代码块建模机器级执行行为。

### Lines 173-185
```cpp
Error DispatchStage::execute(InstRef &IR) {
  assert(canDispatch(IR) && "Cannot dispatch another instruction!");
  return dispatch(IR);
}

#ifndef NDEBUG
void DispatchStage::dump() const {
  PRF.dump();
  RCU.dump();
}
#endif
} // namespace mca
} // namespace llvm
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

## Key Concepts / 关键概念

- **Machine-code analysis / 机器码分析**:
  - **EN**: Models pipeline behavior and instruction scheduling for llvm-mca.
  - **CN**: 为 llvm-mca 建模流水线行为与指令调度。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/MCA/Stages/DispatchStage.h`, `llvm/MCA/HWEventListener.h`, `llvm/Support/Debug.h`
- **Subsystem categories / 子系统类别**: llvm-mca simulation interfaces / llvm-mca 模拟接口 (2), support-library helpers / Support 库辅助功能 (1)
