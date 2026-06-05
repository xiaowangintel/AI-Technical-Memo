# HexagonMachineScheduler.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonMachineScheduler.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon machine scheduling policy and dependency handling.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonMachineScheduler.cpp - MI Scheduler for Hexagon -------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // MachineScheduler schedules machine instructions after phi elimination. It
    10: // preserves LiveIntervals so it can be invoked before register allocation.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "HexagonMachineScheduler.h"
    15: #include "HexagonInstrInfo.h"
    16: #include "HexagonSubtarget.h"
    17: #include "llvm/CodeGen/MachineScheduler.h"
    18: #include "llvm/CodeGen/ScheduleDAG.h"
    19: #include "llvm/CodeGen/VLIWMachineScheduler.h"
    20: 
    21: using namespace llvm;
    22: 
    23: #define DEBUG_TYPE "machine-scheduler"
    24: 
    25: /// Return true if there is a dependence between SUd and SUu.
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as HexagonMachineScheduler.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/CodeGen/MachineScheduler.h, ... (6 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 HexagonMachineScheduler.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/CodeGen/MachineScheduler.h, ... (6 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。

### Lines 26-50 / 第 26-50 行

```cpp
    26: bool HexagonVLIWResourceModel::hasDependence(const SUnit *SUd,
    27:                                              const SUnit *SUu) {
    28:   const auto *QII = static_cast<const HexagonInstrInfo *>(TII);
    29: 
    30:   // Enable .cur formation.
    31:   if (QII->mayBeCurLoad(*SUd->getInstr()))
    32:     return false;
    33: 
    34:   if (QII->canExecuteInBundle(*SUd->getInstr(), *SUu->getInstr()))
    35:     return false;
    36: 
    37:   return VLIWResourceModel::hasDependence(SUd, SUu);
    38: }
    39: 
    40: VLIWResourceModel *HexagonConvergingVLIWScheduler::createVLIWResourceModel(
    41:     const TargetSubtargetInfo &STI, const TargetSchedModel *SchedModel) const {
    42:   return new HexagonVLIWResourceModel(STI, SchedModel);
    43: }
    44: 
    45: int HexagonConvergingVLIWScheduler::SchedulingCost(ReadyQueue &Q, SUnit *SU,
    46:                                                    SchedCandidate &Candidate,
    47:                                                    RegPressureDelta &Delta,
    48:                                                    bool verbose) {
    49:   int ResCount =
    50:       ConvergingVLIWScheduler::SchedulingCost(Q, SU, Candidate, Delta, verbose);
```
- EN: It declares or implements routines such as HexagonVLIWResourceModel::hasDependence, VLIWResourceModel::hasDependence, HexagonConvergingVLIWScheduler::createVLIWResourceModel, HexagonVLIWResourceModel, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVLIWResourceModel, HexagonInstrInfo, HexagonConvergingVLIWScheduler, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonVLIWResourceModel::hasDependence, VLIWResourceModel::hasDependence, HexagonConvergingVLIWScheduler::createVLIWResourceModel, HexagonVLIWResourceModel, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVLIWResourceModel, HexagonInstrInfo, HexagonConvergingVLIWScheduler，说明了它与同级后端组件的连接关系。

### Lines 51-70 / 第 51-70 行

```cpp
    51: 
    52:   if (!SU || SU->isScheduled)
    53:     return ResCount;
    54: 
    55:   auto &QST = DAG->MF.getSubtarget<HexagonSubtarget>();
    56:   auto &QII = *QST.getInstrInfo();
    57:   if (SU->isInstr() && QII.mayBeCurLoad(*SU->getInstr())) {
    58:     if (Q.getID() == TopQID &&
    59:         Top.ResourceModel->isResourceAvailable(SU, true)) {
    60:       ResCount += PriorityTwo;
    61:       LLVM_DEBUG(if (verbose) dbgs() << "C|");
    62:     } else if (Q.getID() == BotQID &&
    63:                Bot.ResourceModel->isResourceAvailable(SU, false)) {
    64:       ResCount += PriorityTwo;
    65:       LLVM_DEBUG(if (verbose) dbgs() << "C|");
    66:     }
    67:   }
    68: 
    69:   return ResCount;
    70: }
```
- EN: It declares or implements routines such as getSubtarget<HexagonSubtarget>, getInstrInfo, LLVM_DEBUG, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget<HexagonSubtarget>, getInstrInfo, LLVM_DEBUG 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction scheduling models / 指令调度模型
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonMachineScheduler.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/CodeGen/MachineScheduler.h, llvm/CodeGen/ScheduleDAG.h, llvm/CodeGen/VLIWMachineScheduler.h`
- Hexagon symbols / Hexagon 符号: `HexagonMachineScheduler, HexagonInstrInfo, HexagonSubtarget, HexagonVLIWResourceModel, HexagonConvergingVLIWScheduler`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
