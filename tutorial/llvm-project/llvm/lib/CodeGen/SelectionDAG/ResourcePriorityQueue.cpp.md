# ResourcePriorityQueue.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/SelectionDAG/ResourcePriorityQueue.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `A DFA-oriented priority queue -*- C++ -*` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“A DFA-oriented priority queue -*- C++ -*”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- ResourcePriorityQueue.cpp - A DFA-oriented priority queue -*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the ResourcePriorityQueue class, which is a
// SchedulingPriorityQueue that prioritizes instructions using DFA state to
// reduce the length of the critical path through the basic block
// on VLIW platforms.
// The scheduler is basically a top-down adaptable list scheduler with DFA
// resource tracking added to the cost function.
// DFA is queried as a state machine to model "packets/bundles" during
// schedule. Currently packets/bundles are discarded at the end of
// scheduling, affecting only order of instructions.
//
//===----------------------------------------------------------------------===//

````
- **L1 EN**: Comment documents: `===- ResourcePriorityQueue.cpp - A DFA-oriented priority queue -*- C++ -…`.
  **L1 CN**: 注释说明：`===- ResourcePriorityQueue.cpp - A DFA-oriented priority queue -*- C++ -…`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L4 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `This file implements the ResourcePriorityQueue class, which is a`.
  **L9 CN**: 注释说明：`This file implements the ResourcePriorityQueue class, which is a`。
- **L10 EN**: Comment documents: `SchedulingPriorityQueue that prioritizes instructions using DFA state to`.
  **L10 CN**: 注释说明：`SchedulingPriorityQueue that prioritizes instructions using DFA state to`。
- **L11 EN**: Comment documents: `reduce the length of the critical path through the basic block`.
  **L11 CN**: 注释说明：`reduce the length of the critical path through the basic block`。
- **L12 EN**: Comment documents: `on VLIW platforms.`.
  **L12 CN**: 注释说明：`on VLIW platforms.`。
- **L13 EN**: Comment documents: `The scheduler is basically a top-down adaptable list scheduler with DFA`.
  **L13 CN**: 注释说明：`The scheduler is basically a top-down adaptable list scheduler with DFA`。
- **L14 EN**: Comment documents: `resource tracking added to the cost function.`.
  **L14 CN**: 注释说明：`resource tracking added to the cost function.`。
- **L15 EN**: Comment documents: `DFA is queried as a state machine to model "packets/bundles" during`.
  **L15 CN**: 注释说明：`DFA is queried as a state machine to model "packets/bundles" during`。
- **L16 EN**: Comment documents: `schedule. Currently packets/bundles are discarded at the end of`.
  **L16 CN**: 注释说明：`schedule. Currently packets/bundles are discarded at the end of`。
- **L17 EN**: Comment documents: `scheduling, affecting only order of instructions.`.
  **L17 CN**: 注释说明：`scheduling, affecting only order of instructions.`。
- **L18 EN**: Continues the surrounding comment block.
  **L18 CN**: 延续周围的注释块。
- **L19 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L19 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L20 EN**: Separates nearby statements for readability.
  **L20 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 21-40

````cpp
#include "llvm/CodeGen/ResourcePriorityQueue.h"
#include "llvm/CodeGen/DFAPacketizer.h"
#include "llvm/CodeGen/SelectionDAGISel.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/Support/CommandLine.h"

using namespace llvm;

#define DEBUG_TYPE "scheduler"

static cl::opt<bool>
    DisableDFASched("disable-dfa-sched", cl::Hidden,
                    cl::desc("Disable use of DFA during scheduling"));

static cl::opt<int> RegPressureThreshold(
    "dfa-sched-reg-pressure-threshold", cl::Hidden, cl::init(5),
````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/ResourcePriorityQueue.h` for ResourcePriorityQueue support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/ResourcePriorityQueue.h`，用于 ResourcePriorityQueue 相关支持。
- **L22 EN**: Includes LLVM header `llvm/CodeGen/DFAPacketizer.h` for DFAPacketizer support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/CodeGen/DFAPacketizer.h`，用于 DFAPacketizer 相关支持。
- **L23 EN**: Includes LLVM header `llvm/CodeGen/SelectionDAGISel.h` for SelectionDAGISel support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/CodeGen/SelectionDAGISel.h`，用于 SelectionDAGISel 相关支持。
- **L24 EN**: Includes LLVM header `llvm/CodeGen/SelectionDAGNodes.h` for SelectionDAGNodes support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/CodeGen/SelectionDAGNodes.h`，用于 SelectionDAGNodes 相关支持。
- **L25 EN**: Includes LLVM header `llvm/CodeGen/TargetInstrInfo.h` for TargetInstrInfo support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetInstrInfo.h`，用于 TargetInstrInfo 相关支持。
- **L26 EN**: Includes LLVM header `llvm/CodeGen/TargetLowering.h` for TargetLowering support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetLowering.h`，用于 TargetLowering 相关支持。
- **L27 EN**: Includes LLVM header `llvm/CodeGen/TargetRegisterInfo.h` for TargetRegisterInfo support.
  **L27 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetRegisterInfo.h`，用于 TargetRegisterInfo 相关支持。
- **L28 EN**: Includes LLVM header `llvm/CodeGen/TargetSubtargetInfo.h` for TargetSubtargetInfo support.
  **L28 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetSubtargetInfo.h`，用于 TargetSubtargetInfo 相关支持。
- **L29 EN**: Includes LLVM header `llvm/Support/CommandLine.h` for CommandLine support.
  **L29 CN**: 引入 LLVM 头文件 `llvm/Support/CommandLine.h`，用于 CommandLine 相关支持。
- **L30 EN**: Separates nearby statements for readability.
  **L30 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L31 EN**: Imports namespace `llvm` into this translation unit.
  **L31 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L32 EN**: Separates nearby statements for readability.
  **L32 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L33 EN**: Defines the LLVM debug channel used by this file.
  **L33 CN**: 定义该文件使用的 LLVM 调试通道。
- **L34 EN**: Separates nearby statements for readability.
  **L34 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L35 EN**: Declares LLVM command-line option `command-line option`.
  **L35 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L36 EN**: Continues logic with `DisableDFASched("disable-dfa-sched", cl::Hidden,`.
  **L36 CN**: 继续处理逻辑：`DisableDFASched("disable-dfa-sched", cl::Hidden,`。
- **L37 EN**: Declares function or method `desc`.
  **L37 CN**: 声明函数或方法 `desc`。
- **L38 EN**: Separates nearby statements for readability.
  **L38 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L39 EN**: Declares LLVM command-line option `command-line option`.
  **L39 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L40 EN**: Provides part of the signature for `init`.
  **L40 CN**: 给出 `init` 的一部分签名。

### Lines 41-60

````cpp
    cl::desc("Track reg pressure and switch priority to in-depth"));

ResourcePriorityQueue::ResourcePriorityQueue(SelectionDAGISel *IS)
    : Picker(this), InstrItins(IS->MF->getSubtarget().getInstrItineraryData()) {
  const TargetSubtargetInfo &STI = IS->MF->getSubtarget();
  TRI = STI.getRegisterInfo();
  TLI = IS->TLI;
  TII = STI.getInstrInfo();
  ResourcesModel.reset(TII->CreateTargetScheduleState(STI));
  // This hard requirement could be relaxed, but for now
  // do not let it proceed.
  assert(ResourcesModel && "Unimplemented CreateTargetScheduleState.");

  unsigned NumRC = TRI->getNumRegClasses();
  RegLimit.resize(NumRC);
  RegPressure.resize(NumRC);
  llvm::fill(RegLimit, 0);
  llvm::fill(RegPressure, 0);
  for (const TargetRegisterClass *RC : TRI->regclasses())
    RegLimit[RC->getID()] = TRI->getRegPressureLimit(RC, *IS->MF);
````
- **L41 EN**: Declares function or method `desc`.
  **L41 CN**: 声明函数或方法 `desc`。
- **L42 EN**: Separates nearby statements for readability.
  **L42 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L43 EN**: Provides part of the signature for `ResourcePriorityQueue`.
  **L43 CN**: 给出 `ResourcePriorityQueue` 的一部分签名。
- **L44 EN**: Begins the definition of `Picker`.
  **L44 CN**: 开始定义 `Picker`。
- **L45 EN**: Assigns or initializes `const TargetSubtargetInfo &STI`.
  **L45 CN**: 对 `const TargetSubtargetInfo &STI` 进行赋值或初始化。
- **L46 EN**: Assigns or initializes `TRI`.
  **L46 CN**: 对 `TRI` 进行赋值或初始化。
- **L47 EN**: Assigns or initializes `TLI`.
  **L47 CN**: 对 `TLI` 进行赋值或初始化。
- **L48 EN**: Assigns or initializes `TII`.
  **L48 CN**: 对 `TII` 进行赋值或初始化。
- **L49 EN**: Executes statement `ResourcesModel.reset(TII->CreateTargetScheduleState(STI));`.
  **L49 CN**: 执行语句 `ResourcesModel.reset(TII->CreateTargetScheduleState(STI));`。
- **L50 EN**: Comment documents: `This hard requirement could be relaxed, but for now`.
  **L50 CN**: 注释说明：`This hard requirement could be relaxed, but for now`。
- **L51 EN**: Comment documents: `do not let it proceed.`.
  **L51 CN**: 注释说明：`do not let it proceed.`。
- **L52 EN**: Checks an invariant in debug builds.
  **L52 CN**: 在调试构建中检查一个不变量。
- **L53 EN**: Separates nearby statements for readability.
  **L53 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L54 EN**: Assigns or initializes `unsigned NumRC`.
  **L54 CN**: 对 `unsigned NumRC` 进行赋值或初始化。
- **L55 EN**: Executes statement `RegLimit.resize(NumRC);`.
  **L55 CN**: 执行语句 `RegLimit.resize(NumRC);`。
- **L56 EN**: Executes statement `RegPressure.resize(NumRC);`.
  **L56 CN**: 执行语句 `RegPressure.resize(NumRC);`。
- **L57 EN**: Declares function or method `fill`.
  **L57 CN**: 声明函数或方法 `fill`。
- **L58 EN**: Declares function or method `fill`.
  **L58 CN**: 声明函数或方法 `fill`。
- **L59 EN**: Starts a loop over a sequence or range.
  **L59 CN**: 开始遍历序列或范围的循环。
- **L60 EN**: Assigns or initializes `RegLimit[RC->getID()]`.
  **L60 CN**: 对 `RegLimit[RC->getID()]` 进行赋值或初始化。

### Lines 61-80

````cpp

  ParallelLiveRanges = 0;
  HorizontalVerticalBalance = 0;
}

ResourcePriorityQueue::~ResourcePriorityQueue() = default;

unsigned
ResourcePriorityQueue::numberRCValPredInSU(SUnit *SU, unsigned RCId) {
  unsigned NumberDeps = 0;
  for (SDep &Pred : SU->Preds) {
    if (Pred.isCtrl())
      continue;

    SUnit *PredSU = Pred.getSUnit();
    const SDNode *ScegN = PredSU->getNode();

    if (!ScegN)
      continue;

````
- **L61 EN**: Separates nearby statements for readability.
  **L61 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L62 EN**: Assigns or initializes `ParallelLiveRanges`.
  **L62 CN**: 对 `ParallelLiveRanges` 进行赋值或初始化。
- **L63 EN**: Assigns or initializes `HorizontalVerticalBalance`.
  **L63 CN**: 对 `HorizontalVerticalBalance` 进行赋值或初始化。
- **L64 EN**: Closes the current scope.
  **L64 CN**: 关闭当前作用域。
- **L65 EN**: Separates nearby statements for readability.
  **L65 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L66 EN**: Declares function or method `~ResourcePriorityQueue`.
  **L66 CN**: 声明函数或方法 `~ResourcePriorityQueue`。
- **L67 EN**: Separates nearby statements for readability.
  **L67 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L68 EN**: Continues logic with `unsigned`.
  **L68 CN**: 继续处理逻辑：`unsigned`。
- **L69 EN**: Begins the definition of `numberRCValPredInSU`.
  **L69 CN**: 开始定义 `numberRCValPredInSU`。
- **L70 EN**: Assigns or initializes `unsigned NumberDeps`.
  **L70 CN**: 对 `unsigned NumberDeps` 进行赋值或初始化。
- **L71 EN**: Starts a loop over a sequence or range.
  **L71 CN**: 开始遍历序列或范围的循环。
- **L72 EN**: Begins a conditional branch.
  **L72 CN**: 开始一个条件分支。
- **L73 EN**: Skips to the next loop iteration.
  **L73 CN**: 跳到下一次循环迭代。
- **L74 EN**: Separates nearby statements for readability.
  **L74 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L75 EN**: Assigns or initializes `SUnit *PredSU`.
  **L75 CN**: 对 `SUnit *PredSU` 进行赋值或初始化。
- **L76 EN**: Assigns or initializes `const SDNode *ScegN`.
  **L76 CN**: 对 `const SDNode *ScegN` 进行赋值或初始化。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Begins a conditional branch.
  **L78 CN**: 开始一个条件分支。
- **L79 EN**: Skips to the next loop iteration.
  **L79 CN**: 跳到下一次循环迭代。
- **L80 EN**: Separates nearby statements for readability.
  **L80 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 81-100

````cpp
    // If value is passed to CopyToReg, it is probably
    // live outside BB.
    switch (ScegN->getOpcode()) {
      default:  break;
      case ISD::TokenFactor:    break;
      case ISD::CopyFromReg:    NumberDeps++;  break;
      case ISD::CopyToReg:      break;
      case ISD::INLINEASM:      break;
      case ISD::INLINEASM_BR:   break;
    }
    if (!ScegN->isMachineOpcode())
      continue;

    for (unsigned i = 0, e = ScegN->getNumValues(); i != e; ++i) {
      MVT VT = ScegN->getSimpleValueType(i);
      if (TLI->isTypeLegal(VT)
          && (TLI->getRegClassFor(VT)->getID() == RCId)) {
        NumberDeps++;
        break;
      }
````
- **L81 EN**: Comment documents: `If value is passed to CopyToReg, it is probably`.
  **L81 CN**: 注释说明：`If value is passed to CopyToReg, it is probably`。
- **L82 EN**: Comment documents: `live outside BB.`.
  **L82 CN**: 注释说明：`live outside BB.`。
- **L83 EN**: Starts a multi-way branch.
  **L83 CN**: 开始一个多路分支。
- **L84 EN**: Handles the default switch case.
  **L84 CN**: 处理 switch 的默认分支。
- **L85 EN**: Handles one switch case.
  **L85 CN**: 处理一个 switch 分支。
- **L86 EN**: Handles one switch case.
  **L86 CN**: 处理一个 switch 分支。
- **L87 EN**: Handles one switch case.
  **L87 CN**: 处理一个 switch 分支。
- **L88 EN**: Handles one switch case.
  **L88 CN**: 处理一个 switch 分支。
- **L89 EN**: Handles one switch case.
  **L89 CN**: 处理一个 switch 分支。
- **L90 EN**: Closes the current scope.
  **L90 CN**: 关闭当前作用域。
- **L91 EN**: Begins a conditional branch.
  **L91 CN**: 开始一个条件分支。
- **L92 EN**: Skips to the next loop iteration.
  **L92 CN**: 跳到下一次循环迭代。
- **L93 EN**: Separates nearby statements for readability.
  **L93 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L94 EN**: Starts a loop over a sequence or range.
  **L94 CN**: 开始遍历序列或范围的循环。
- **L95 EN**: Assigns or initializes `MVT VT`.
  **L95 CN**: 对 `MVT VT` 进行赋值或初始化。
- **L96 EN**: Begins a conditional branch.
  **L96 CN**: 开始一个条件分支。
- **L97 EN**: Starts block `&& (TLI->getRegClassFor(VT)->getID() == RCId))`.
  **L97 CN**: 开始代码块 `&& (TLI->getRegClassFor(VT)->getID() == RCId))`。
- **L98 EN**: Executes statement `NumberDeps++;`.
  **L98 CN**: 执行语句 `NumberDeps++;`。
- **L99 EN**: Breaks out of the current control-flow construct.
  **L99 CN**: 跳出当前控制流结构。
- **L100 EN**: Closes the current scope.
  **L100 CN**: 关闭当前作用域。

### Lines 101-120

````cpp
    }
  }
  return NumberDeps;
}

unsigned ResourcePriorityQueue::numberRCValSuccInSU(SUnit *SU,
                                                    unsigned RCId) {
  unsigned NumberDeps = 0;
  for (const SDep &Succ : SU->Succs) {
    if (Succ.isCtrl())
      continue;

    SUnit *SuccSU = Succ.getSUnit();
    const SDNode *ScegN = SuccSU->getNode();
    if (!ScegN)
      continue;

    // If value is passed to CopyToReg, it is probably
    // live outside BB.
    switch (ScegN->getOpcode()) {
````
- **L101 EN**: Closes the current scope.
  **L101 CN**: 关闭当前作用域。
- **L102 EN**: Closes the current scope.
  **L102 CN**: 关闭当前作用域。
- **L103 EN**: Returns `NumberDeps` to the caller.
  **L103 CN**: 向调用者返回 `NumberDeps`。
- **L104 EN**: Closes the current scope.
  **L104 CN**: 关闭当前作用域。
- **L105 EN**: Separates nearby statements for readability.
  **L105 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L106 EN**: Provides part of the signature for `numberRCValSuccInSU`.
  **L106 CN**: 给出 `numberRCValSuccInSU` 的一部分签名。
- **L107 EN**: Starts block `unsigned RCId)`.
  **L107 CN**: 开始代码块 `unsigned RCId)`。
- **L108 EN**: Assigns or initializes `unsigned NumberDeps`.
  **L108 CN**: 对 `unsigned NumberDeps` 进行赋值或初始化。
- **L109 EN**: Starts a loop over a sequence or range.
  **L109 CN**: 开始遍历序列或范围的循环。
- **L110 EN**: Begins a conditional branch.
  **L110 CN**: 开始一个条件分支。
- **L111 EN**: Skips to the next loop iteration.
  **L111 CN**: 跳到下一次循环迭代。
- **L112 EN**: Separates nearby statements for readability.
  **L112 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L113 EN**: Assigns or initializes `SUnit *SuccSU`.
  **L113 CN**: 对 `SUnit *SuccSU` 进行赋值或初始化。
- **L114 EN**: Assigns or initializes `const SDNode *ScegN`.
  **L114 CN**: 对 `const SDNode *ScegN` 进行赋值或初始化。
- **L115 EN**: Begins a conditional branch.
  **L115 CN**: 开始一个条件分支。
- **L116 EN**: Skips to the next loop iteration.
  **L116 CN**: 跳到下一次循环迭代。
- **L117 EN**: Separates nearby statements for readability.
  **L117 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L118 EN**: Comment documents: `If value is passed to CopyToReg, it is probably`.
  **L118 CN**: 注释说明：`If value is passed to CopyToReg, it is probably`。
- **L119 EN**: Comment documents: `live outside BB.`.
  **L119 CN**: 注释说明：`live outside BB.`。
- **L120 EN**: Starts a multi-way branch.
  **L120 CN**: 开始一个多路分支。

### Lines 121-140

````cpp
      default:  break;
      case ISD::TokenFactor:    break;
      case ISD::CopyFromReg:    break;
      case ISD::CopyToReg:      NumberDeps++;  break;
      case ISD::INLINEASM:      break;
      case ISD::INLINEASM_BR:   break;
    }
    if (!ScegN->isMachineOpcode())
      continue;

    for (unsigned i = 0, e = ScegN->getNumOperands(); i != e; ++i) {
      const SDValue &Op = ScegN->getOperand(i);
      MVT VT = Op.getNode()->getSimpleValueType(Op.getResNo());
      if (TLI->isTypeLegal(VT)
          && (TLI->getRegClassFor(VT)->getID() == RCId)) {
        NumberDeps++;
        break;
      }
    }
  }
````
- **L121 EN**: Handles the default switch case.
  **L121 CN**: 处理 switch 的默认分支。
- **L122 EN**: Handles one switch case.
  **L122 CN**: 处理一个 switch 分支。
- **L123 EN**: Handles one switch case.
  **L123 CN**: 处理一个 switch 分支。
- **L124 EN**: Handles one switch case.
  **L124 CN**: 处理一个 switch 分支。
- **L125 EN**: Handles one switch case.
  **L125 CN**: 处理一个 switch 分支。
- **L126 EN**: Handles one switch case.
  **L126 CN**: 处理一个 switch 分支。
- **L127 EN**: Closes the current scope.
  **L127 CN**: 关闭当前作用域。
- **L128 EN**: Begins a conditional branch.
  **L128 CN**: 开始一个条件分支。
- **L129 EN**: Skips to the next loop iteration.
  **L129 CN**: 跳到下一次循环迭代。
- **L130 EN**: Separates nearby statements for readability.
  **L130 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L131 EN**: Starts a loop over a sequence or range.
  **L131 CN**: 开始遍历序列或范围的循环。
- **L132 EN**: Assigns or initializes `const SDValue &Op`.
  **L132 CN**: 对 `const SDValue &Op` 进行赋值或初始化。
- **L133 EN**: Assigns or initializes `MVT VT`.
  **L133 CN**: 对 `MVT VT` 进行赋值或初始化。
- **L134 EN**: Begins a conditional branch.
  **L134 CN**: 开始一个条件分支。
- **L135 EN**: Starts block `&& (TLI->getRegClassFor(VT)->getID() == RCId))`.
  **L135 CN**: 开始代码块 `&& (TLI->getRegClassFor(VT)->getID() == RCId))`。
- **L136 EN**: Executes statement `NumberDeps++;`.
  **L136 CN**: 执行语句 `NumberDeps++;`。
- **L137 EN**: Breaks out of the current control-flow construct.
  **L137 CN**: 跳出当前控制流结构。
- **L138 EN**: Closes the current scope.
  **L138 CN**: 关闭当前作用域。
- **L139 EN**: Closes the current scope.
  **L139 CN**: 关闭当前作用域。
- **L140 EN**: Closes the current scope.
  **L140 CN**: 关闭当前作用域。

### Lines 141-160

````cpp
  return NumberDeps;
}

static unsigned numberCtrlDepsInSU(SUnit *SU) {
  unsigned NumberDeps = 0;
  for (const SDep &Succ : SU->Succs)
    if (Succ.isCtrl())
      NumberDeps++;

  return NumberDeps;
}

static unsigned numberCtrlPredInSU(SUnit *SU) {
  unsigned NumberDeps = 0;
  for (SDep &Pred : SU->Preds)
    if (Pred.isCtrl())
      NumberDeps++;

  return NumberDeps;
}
````
- **L141 EN**: Returns `NumberDeps` to the caller.
  **L141 CN**: 向调用者返回 `NumberDeps`。
- **L142 EN**: Closes the current scope.
  **L142 CN**: 关闭当前作用域。
- **L143 EN**: Separates nearby statements for readability.
  **L143 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L144 EN**: Begins the definition of `numberCtrlDepsInSU`.
  **L144 CN**: 开始定义 `numberCtrlDepsInSU`。
- **L145 EN**: Assigns or initializes `unsigned NumberDeps`.
  **L145 CN**: 对 `unsigned NumberDeps` 进行赋值或初始化。
- **L146 EN**: Starts a loop over a sequence or range.
  **L146 CN**: 开始遍历序列或范围的循环。
- **L147 EN**: Begins a conditional branch.
  **L147 CN**: 开始一个条件分支。
- **L148 EN**: Executes statement `NumberDeps++;`.
  **L148 CN**: 执行语句 `NumberDeps++;`。
- **L149 EN**: Separates nearby statements for readability.
  **L149 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L150 EN**: Returns `NumberDeps` to the caller.
  **L150 CN**: 向调用者返回 `NumberDeps`。
- **L151 EN**: Closes the current scope.
  **L151 CN**: 关闭当前作用域。
- **L152 EN**: Separates nearby statements for readability.
  **L152 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L153 EN**: Begins the definition of `numberCtrlPredInSU`.
  **L153 CN**: 开始定义 `numberCtrlPredInSU`。
- **L154 EN**: Assigns or initializes `unsigned NumberDeps`.
  **L154 CN**: 对 `unsigned NumberDeps` 进行赋值或初始化。
- **L155 EN**: Starts a loop over a sequence or range.
  **L155 CN**: 开始遍历序列或范围的循环。
- **L156 EN**: Begins a conditional branch.
  **L156 CN**: 开始一个条件分支。
- **L157 EN**: Executes statement `NumberDeps++;`.
  **L157 CN**: 执行语句 `NumberDeps++;`。
- **L158 EN**: Separates nearby statements for readability.
  **L158 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L159 EN**: Returns `NumberDeps` to the caller.
  **L159 CN**: 向调用者返回 `NumberDeps`。
- **L160 EN**: Closes the current scope.
  **L160 CN**: 关闭当前作用域。

### Lines 161-180

````cpp

///
/// Initialize nodes.
///
void ResourcePriorityQueue::initNodes(std::vector<SUnit> &sunits) {
  SUnits = &sunits;
  NumNodesSolelyBlocking.resize(SUnits->size(), 0);

  for (SUnit &SU : *SUnits) {
    initNumRegDefsLeft(&SU);
    SU.NodeQueueId = 0;
  }
}

/// This heuristic is used if DFA scheduling is not desired
/// for some VLIW platform.
bool resource_sort::operator()(const SUnit *LHS, const SUnit *RHS) const {
  // The isScheduleHigh flag allows nodes with wraparound dependencies that
  // cannot easily be modeled as edges with latencies to be scheduled as
  // soon as possible in a top-down schedule.
````
- **L161 EN**: Separates nearby statements for readability.
  **L161 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L162 EN**: Continues the surrounding comment block.
  **L162 CN**: 延续周围的注释块。
- **L163 EN**: Comment documents: `Initialize nodes.`.
  **L163 CN**: 注释说明：`Initialize nodes.`。
- **L164 EN**: Continues the surrounding comment block.
  **L164 CN**: 延续周围的注释块。
- **L165 EN**: Begins the definition of `initNodes`.
  **L165 CN**: 开始定义 `initNodes`。
- **L166 EN**: Assigns or initializes `SUnits`.
  **L166 CN**: 对 `SUnits` 进行赋值或初始化。
- **L167 EN**: Executes statement `NumNodesSolelyBlocking.resize(SUnits->size(), 0);`.
  **L167 CN**: 执行语句 `NumNodesSolelyBlocking.resize(SUnits->size(), 0);`。
- **L168 EN**: Separates nearby statements for readability.
  **L168 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L169 EN**: Starts a loop over a sequence or range.
  **L169 CN**: 开始遍历序列或范围的循环。
- **L170 EN**: Executes statement `initNumRegDefsLeft(&SU);`.
  **L170 CN**: 执行语句 `initNumRegDefsLeft(&SU);`。
- **L171 EN**: Assigns or initializes `SU.NodeQueueId`.
  **L171 CN**: 对 `SU.NodeQueueId` 进行赋值或初始化。
- **L172 EN**: Closes the current scope.
  **L172 CN**: 关闭当前作用域。
- **L173 EN**: Closes the current scope.
  **L173 CN**: 关闭当前作用域。
- **L174 EN**: Separates nearby statements for readability.
  **L174 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L175 EN**: Comment documents: `This heuristic is used if DFA scheduling is not desired`.
  **L175 CN**: 注释说明：`This heuristic is used if DFA scheduling is not desired`。
- **L176 EN**: Comment documents: `for some VLIW platform.`.
  **L176 CN**: 注释说明：`for some VLIW platform.`。
- **L177 EN**: Begins the definition of `operator`.
  **L177 CN**: 开始定义 `operator`。
- **L178 EN**: Comment documents: `The isScheduleHigh flag allows nodes with wraparound dependencies that`.
  **L178 CN**: 注释说明：`The isScheduleHigh flag allows nodes with wraparound dependencies that`。
- **L179 EN**: Comment documents: `cannot easily be modeled as edges with latencies to be scheduled as`.
  **L179 CN**: 注释说明：`cannot easily be modeled as edges with latencies to be scheduled as`。
- **L180 EN**: Comment documents: `soon as possible in a top-down schedule.`.
  **L180 CN**: 注释说明：`soon as possible in a top-down schedule.`。

### Lines 181-200

````cpp
  if (LHS->isScheduleHigh && !RHS->isScheduleHigh)
    return false;

  if (!LHS->isScheduleHigh && RHS->isScheduleHigh)
    return true;

  unsigned LHSNum = LHS->NodeNum;
  unsigned RHSNum = RHS->NodeNum;

  // The most important heuristic is scheduling the critical path.
  unsigned LHSLatency = PQ->getLatency(LHSNum);
  unsigned RHSLatency = PQ->getLatency(RHSNum);
  if (LHSLatency < RHSLatency) return true;
  if (LHSLatency > RHSLatency) return false;

  // After that, if two nodes have identical latencies, look to see if one will
  // unblock more other nodes than the other.
  unsigned LHSBlocked = PQ->getNumSolelyBlockNodes(LHSNum);
  unsigned RHSBlocked = PQ->getNumSolelyBlockNodes(RHSNum);
  if (LHSBlocked < RHSBlocked) return true;
````
- **L181 EN**: Begins a conditional branch.
  **L181 CN**: 开始一个条件分支。
- **L182 EN**: Returns `false` to the caller.
  **L182 CN**: 向调用者返回 `false`。
- **L183 EN**: Separates nearby statements for readability.
  **L183 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L184 EN**: Begins a conditional branch.
  **L184 CN**: 开始一个条件分支。
- **L185 EN**: Returns `true` to the caller.
  **L185 CN**: 向调用者返回 `true`。
- **L186 EN**: Separates nearby statements for readability.
  **L186 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L187 EN**: Assigns or initializes `unsigned LHSNum`.
  **L187 CN**: 对 `unsigned LHSNum` 进行赋值或初始化。
- **L188 EN**: Assigns or initializes `unsigned RHSNum`.
  **L188 CN**: 对 `unsigned RHSNum` 进行赋值或初始化。
- **L189 EN**: Separates nearby statements for readability.
  **L189 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L190 EN**: Comment documents: `The most important heuristic is scheduling the critical path.`.
  **L190 CN**: 注释说明：`The most important heuristic is scheduling the critical path.`。
- **L191 EN**: Assigns or initializes `unsigned LHSLatency`.
  **L191 CN**: 对 `unsigned LHSLatency` 进行赋值或初始化。
- **L192 EN**: Assigns or initializes `unsigned RHSLatency`.
  **L192 CN**: 对 `unsigned RHSLatency` 进行赋值或初始化。
- **L193 EN**: Begins a conditional branch.
  **L193 CN**: 开始一个条件分支。
- **L194 EN**: Begins a conditional branch.
  **L194 CN**: 开始一个条件分支。
- **L195 EN**: Separates nearby statements for readability.
  **L195 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L196 EN**: Comment documents: `After that, if two nodes have identical latencies, look to see if one wi…`.
  **L196 CN**: 注释说明：`After that, if two nodes have identical latencies, look to see if one wi…`。
- **L197 EN**: Comment documents: `unblock more other nodes than the other.`.
  **L197 CN**: 注释说明：`unblock more other nodes than the other.`。
- **L198 EN**: Assigns or initializes `unsigned LHSBlocked`.
  **L198 CN**: 对 `unsigned LHSBlocked` 进行赋值或初始化。
- **L199 EN**: Assigns or initializes `unsigned RHSBlocked`.
  **L199 CN**: 对 `unsigned RHSBlocked` 进行赋值或初始化。
- **L200 EN**: Begins a conditional branch.
  **L200 CN**: 开始一个条件分支。

### Lines 201-220

````cpp
  if (LHSBlocked > RHSBlocked) return false;

  // Finally, just to provide a stable ordering, use the node number as a
  // deciding factor.
  return LHSNum < RHSNum;
}


/// getSingleUnscheduledPred - If there is exactly one unscheduled predecessor
/// of SU, return it, otherwise return null.
SUnit *ResourcePriorityQueue::getSingleUnscheduledPred(SUnit *SU) {
  SUnit *OnlyAvailablePred = nullptr;
  for (const SDep &Pred : SU->Preds) {
    SUnit &PredSU = *Pred.getSUnit();
    if (!PredSU.isScheduled) {
      // We found an available, but not scheduled, predecessor.  If it's the
      // only one we have found, keep track of it... otherwise give up.
      if (OnlyAvailablePred && OnlyAvailablePred != &PredSU)
        return nullptr;
      OnlyAvailablePred = &PredSU;
````
- **L201 EN**: Begins a conditional branch.
  **L201 CN**: 开始一个条件分支。
- **L202 EN**: Separates nearby statements for readability.
  **L202 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L203 EN**: Comment documents: `Finally, just to provide a stable ordering, use the node number as a`.
  **L203 CN**: 注释说明：`Finally, just to provide a stable ordering, use the node number as a`。
- **L204 EN**: Comment documents: `deciding factor.`.
  **L204 CN**: 注释说明：`deciding factor.`。
- **L205 EN**: Returns `LHSNum < RHSNum` to the caller.
  **L205 CN**: 向调用者返回 `LHSNum < RHSNum`。
- **L206 EN**: Closes the current scope.
  **L206 CN**: 关闭当前作用域。
- **L207 EN**: Separates nearby statements for readability.
  **L207 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L208 EN**: Separates nearby statements for readability.
  **L208 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L209 EN**: Comment documents: `getSingleUnscheduledPred - If there is exactly one unscheduled predecess…`.
  **L209 CN**: 注释说明：`getSingleUnscheduledPred - If there is exactly one unscheduled predecess…`。
- **L210 EN**: Comment documents: `of SU, return it, otherwise return null.`.
  **L210 CN**: 注释说明：`of SU, return it, otherwise return null.`。
- **L211 EN**: Begins the definition of `getSingleUnscheduledPred`.
  **L211 CN**: 开始定义 `getSingleUnscheduledPred`。
- **L212 EN**: Assigns or initializes `SUnit *OnlyAvailablePred`.
  **L212 CN**: 对 `SUnit *OnlyAvailablePred` 进行赋值或初始化。
- **L213 EN**: Starts a loop over a sequence or range.
  **L213 CN**: 开始遍历序列或范围的循环。
- **L214 EN**: Assigns or initializes `SUnit &PredSU`.
  **L214 CN**: 对 `SUnit &PredSU` 进行赋值或初始化。
- **L215 EN**: Begins a conditional branch.
  **L215 CN**: 开始一个条件分支。
- **L216 EN**: Comment documents: `We found an available, but not scheduled, predecessor. If it's the`.
  **L216 CN**: 注释说明：`We found an available, but not scheduled, predecessor. If it's the`。
- **L217 EN**: Comment documents: `only one we have found, keep track of it... otherwise give up.`.
  **L217 CN**: 注释说明：`only one we have found, keep track of it... otherwise give up.`。
- **L218 EN**: Begins a conditional branch.
  **L218 CN**: 开始一个条件分支。
- **L219 EN**: Returns `nullptr` to the caller.
  **L219 CN**: 向调用者返回 `nullptr`。
- **L220 EN**: Assigns or initializes `OnlyAvailablePred`.
  **L220 CN**: 对 `OnlyAvailablePred` 进行赋值或初始化。

### Lines 221-240

````cpp
    }
  }
  return OnlyAvailablePred;
}

void ResourcePriorityQueue::push(SUnit *SU) {
  // Look at all of the successors of this node.  Count the number of nodes that
  // this node is the sole unscheduled node for.
  unsigned NumNodesBlocking = 0;
  for (const SDep &Succ : SU->Succs)
    if (getSingleUnscheduledPred(Succ.getSUnit()) == SU)
      ++NumNodesBlocking;

  NumNodesSolelyBlocking[SU->NodeNum] = NumNodesBlocking;
  Queue.push_back(SU);
}

/// Check if scheduling of this SU is possible
/// in the current packet.
bool ResourcePriorityQueue::isResourceAvailable(SUnit *SU) {
````
- **L221 EN**: Closes the current scope.
  **L221 CN**: 关闭当前作用域。
- **L222 EN**: Closes the current scope.
  **L222 CN**: 关闭当前作用域。
- **L223 EN**: Returns `OnlyAvailablePred` to the caller.
  **L223 CN**: 向调用者返回 `OnlyAvailablePred`。
- **L224 EN**: Closes the current scope.
  **L224 CN**: 关闭当前作用域。
- **L225 EN**: Separates nearby statements for readability.
  **L225 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L226 EN**: Begins the definition of `push`.
  **L226 CN**: 开始定义 `push`。
- **L227 EN**: Comment documents: `Look at all of the successors of this node. Count the number of nodes th…`.
  **L227 CN**: 注释说明：`Look at all of the successors of this node. Count the number of nodes th…`。
- **L228 EN**: Comment documents: `this node is the sole unscheduled node for.`.
  **L228 CN**: 注释说明：`this node is the sole unscheduled node for.`。
- **L229 EN**: Assigns or initializes `unsigned NumNodesBlocking`.
  **L229 CN**: 对 `unsigned NumNodesBlocking` 进行赋值或初始化。
- **L230 EN**: Starts a loop over a sequence or range.
  **L230 CN**: 开始遍历序列或范围的循环。
- **L231 EN**: Begins a conditional branch.
  **L231 CN**: 开始一个条件分支。
- **L232 EN**: Executes statement `++NumNodesBlocking;`.
  **L232 CN**: 执行语句 `++NumNodesBlocking;`。
- **L233 EN**: Separates nearby statements for readability.
  **L233 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L234 EN**: Assigns or initializes `NumNodesSolelyBlocking[SU->NodeNum]`.
  **L234 CN**: 对 `NumNodesSolelyBlocking[SU->NodeNum]` 进行赋值或初始化。
- **L235 EN**: Executes statement `Queue.push_back(SU);`.
  **L235 CN**: 执行语句 `Queue.push_back(SU);`。
- **L236 EN**: Closes the current scope.
  **L236 CN**: 关闭当前作用域。
- **L237 EN**: Separates nearby statements for readability.
  **L237 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L238 EN**: Comment documents: `Check if scheduling of this SU is possible`.
  **L238 CN**: 注释说明：`Check if scheduling of this SU is possible`。
- **L239 EN**: Comment documents: `in the current packet.`.
  **L239 CN**: 注释说明：`in the current packet.`。
- **L240 EN**: Begins the definition of `isResourceAvailable`.
  **L240 CN**: 开始定义 `isResourceAvailable`。

### Lines 241-260

````cpp
  if (!SU || !SU->getNode())
    return false;

  // If this is a compound instruction,
  // it is likely to be a call. Do not delay it.
  if (SU->getNode()->getGluedNode())
    return true;

  // First see if the pipeline could receive this instruction
  // in the current cycle.
  if (SU->getNode()->isMachineOpcode())
    switch (SU->getNode()->getMachineOpcode()) {
    default:
      if (!ResourcesModel->canReserveResources(&TII->get(
          SU->getNode()->getMachineOpcode())))
           return false;
      break;
    case TargetOpcode::EXTRACT_SUBREG:
    case TargetOpcode::INSERT_SUBREG:
    case TargetOpcode::SUBREG_TO_REG:
````
- **L241 EN**: Begins a conditional branch.
  **L241 CN**: 开始一个条件分支。
- **L242 EN**: Returns `false` to the caller.
  **L242 CN**: 向调用者返回 `false`。
- **L243 EN**: Separates nearby statements for readability.
  **L243 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L244 EN**: Comment documents: `If this is a compound instruction,`.
  **L244 CN**: 注释说明：`If this is a compound instruction,`。
- **L245 EN**: Comment documents: `it is likely to be a call. Do not delay it.`.
  **L245 CN**: 注释说明：`it is likely to be a call. Do not delay it.`。
- **L246 EN**: Begins a conditional branch.
  **L246 CN**: 开始一个条件分支。
- **L247 EN**: Returns `true` to the caller.
  **L247 CN**: 向调用者返回 `true`。
- **L248 EN**: Separates nearby statements for readability.
  **L248 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L249 EN**: Comment documents: `First see if the pipeline could receive this instruction`.
  **L249 CN**: 注释说明：`First see if the pipeline could receive this instruction`。
- **L250 EN**: Comment documents: `in the current cycle.`.
  **L250 CN**: 注释说明：`in the current cycle.`。
- **L251 EN**: Begins a conditional branch.
  **L251 CN**: 开始一个条件分支。
- **L252 EN**: Starts a multi-way branch.
  **L252 CN**: 开始一个多路分支。
- **L253 EN**: Handles the default switch case.
  **L253 CN**: 处理 switch 的默认分支。
- **L254 EN**: Begins a conditional branch.
  **L254 CN**: 开始一个条件分支。
- **L255 EN**: Continues logic with `SU->getNode()->getMachineOpcode())))`.
  **L255 CN**: 继续处理逻辑：`SU->getNode()->getMachineOpcode())))`。
- **L256 EN**: Returns `false` to the caller.
  **L256 CN**: 向调用者返回 `false`。
- **L257 EN**: Breaks out of the current control-flow construct.
  **L257 CN**: 跳出当前控制流结构。
- **L258 EN**: Handles one switch case.
  **L258 CN**: 处理一个 switch 分支。
- **L259 EN**: Handles one switch case.
  **L259 CN**: 处理一个 switch 分支。
- **L260 EN**: Handles one switch case.
  **L260 CN**: 处理一个 switch 分支。

### Lines 261-280

````cpp
    case TargetOpcode::REG_SEQUENCE:
    case TargetOpcode::IMPLICIT_DEF:
        break;
    }

  // Now see if there are no other dependencies
  // to instructions already in the packet.
  for (const SUnit *S : Packet)
    for (const SDep &Succ : S->Succs) {
      // Since we do not add pseudos to packets, might as well
      // ignore order deps.
      if (Succ.isCtrl())
        continue;

      if (Succ.getSUnit() == SU)
        return false;
    }

  return true;
}
````
- **L261 EN**: Handles one switch case.
  **L261 CN**: 处理一个 switch 分支。
- **L262 EN**: Handles one switch case.
  **L262 CN**: 处理一个 switch 分支。
- **L263 EN**: Breaks out of the current control-flow construct.
  **L263 CN**: 跳出当前控制流结构。
- **L264 EN**: Closes the current scope.
  **L264 CN**: 关闭当前作用域。
- **L265 EN**: Separates nearby statements for readability.
  **L265 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L266 EN**: Comment documents: `Now see if there are no other dependencies`.
  **L266 CN**: 注释说明：`Now see if there are no other dependencies`。
- **L267 EN**: Comment documents: `to instructions already in the packet.`.
  **L267 CN**: 注释说明：`to instructions already in the packet.`。
- **L268 EN**: Starts a loop over a sequence or range.
  **L268 CN**: 开始遍历序列或范围的循环。
- **L269 EN**: Starts a loop over a sequence or range.
  **L269 CN**: 开始遍历序列或范围的循环。
- **L270 EN**: Comment documents: `Since we do not add pseudos to packets, might as well`.
  **L270 CN**: 注释说明：`Since we do not add pseudos to packets, might as well`。
- **L271 EN**: Comment documents: `ignore order deps.`.
  **L271 CN**: 注释说明：`ignore order deps.`。
- **L272 EN**: Begins a conditional branch.
  **L272 CN**: 开始一个条件分支。
- **L273 EN**: Skips to the next loop iteration.
  **L273 CN**: 跳到下一次循环迭代。
- **L274 EN**: Separates nearby statements for readability.
  **L274 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L275 EN**: Begins a conditional branch.
  **L275 CN**: 开始一个条件分支。
- **L276 EN**: Returns `false` to the caller.
  **L276 CN**: 向调用者返回 `false`。
- **L277 EN**: Closes the current scope.
  **L277 CN**: 关闭当前作用域。
- **L278 EN**: Separates nearby statements for readability.
  **L278 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L279 EN**: Returns `true` to the caller.
  **L279 CN**: 向调用者返回 `true`。
- **L280 EN**: Closes the current scope.
  **L280 CN**: 关闭当前作用域。

### Lines 281-300

````cpp

/// Keep track of available resources.
void ResourcePriorityQueue::reserveResources(SUnit *SU) {
  // If this SU does not fit in the packet
  // start a new one.
  if (!isResourceAvailable(SU) || SU->getNode()->getGluedNode()) {
    ResourcesModel->clearResources();
    Packet.clear();
  }

  if (SU->getNode() && SU->getNode()->isMachineOpcode()) {
    switch (SU->getNode()->getMachineOpcode()) {
    default:
      ResourcesModel->reserveResources(&TII->get(
        SU->getNode()->getMachineOpcode()));
      break;
    case TargetOpcode::EXTRACT_SUBREG:
    case TargetOpcode::INSERT_SUBREG:
    case TargetOpcode::SUBREG_TO_REG:
    case TargetOpcode::REG_SEQUENCE:
````
- **L281 EN**: Separates nearby statements for readability.
  **L281 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L282 EN**: Comment documents: `Keep track of available resources.`.
  **L282 CN**: 注释说明：`Keep track of available resources.`。
- **L283 EN**: Begins the definition of `reserveResources`.
  **L283 CN**: 开始定义 `reserveResources`。
- **L284 EN**: Comment documents: `If this SU does not fit in the packet`.
  **L284 CN**: 注释说明：`If this SU does not fit in the packet`。
- **L285 EN**: Comment documents: `start a new one.`.
  **L285 CN**: 注释说明：`start a new one.`。
- **L286 EN**: Begins a conditional branch.
  **L286 CN**: 开始一个条件分支。
- **L287 EN**: Executes statement `ResourcesModel->clearResources();`.
  **L287 CN**: 执行语句 `ResourcesModel->clearResources();`。
- **L288 EN**: Executes statement `Packet.clear();`.
  **L288 CN**: 执行语句 `Packet.clear();`。
- **L289 EN**: Closes the current scope.
  **L289 CN**: 关闭当前作用域。
- **L290 EN**: Separates nearby statements for readability.
  **L290 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L291 EN**: Begins a conditional branch.
  **L291 CN**: 开始一个条件分支。
- **L292 EN**: Starts a multi-way branch.
  **L292 CN**: 开始一个多路分支。
- **L293 EN**: Handles the default switch case.
  **L293 CN**: 处理 switch 的默认分支。
- **L294 EN**: Continues logic with `ResourcesModel->reserveResources(&TII->get(`.
  **L294 CN**: 继续处理逻辑：`ResourcesModel->reserveResources(&TII->get(`。
- **L295 EN**: Executes statement `SU->getNode()->getMachineOpcode()));`.
  **L295 CN**: 执行语句 `SU->getNode()->getMachineOpcode()));`。
- **L296 EN**: Breaks out of the current control-flow construct.
  **L296 CN**: 跳出当前控制流结构。
- **L297 EN**: Handles one switch case.
  **L297 CN**: 处理一个 switch 分支。
- **L298 EN**: Handles one switch case.
  **L298 CN**: 处理一个 switch 分支。
- **L299 EN**: Handles one switch case.
  **L299 CN**: 处理一个 switch 分支。
- **L300 EN**: Handles one switch case.
  **L300 CN**: 处理一个 switch 分支。

### Lines 301-320

````cpp
    case TargetOpcode::IMPLICIT_DEF:
      break;
    }
    Packet.push_back(SU);
  }
  // Forcefully end packet for PseudoOps.
  else {
    ResourcesModel->clearResources();
    Packet.clear();
  }

  // If packet is now full, reset the state so in the next cycle
  // we start fresh.
  if (Packet.size() >= InstrItins->SchedModel.IssueWidth) {
    ResourcesModel->clearResources();
    Packet.clear();
  }
}

int ResourcePriorityQueue::rawRegPressureDelta(SUnit *SU, unsigned RCId) {
````
- **L301 EN**: Handles one switch case.
  **L301 CN**: 处理一个 switch 分支。
- **L302 EN**: Breaks out of the current control-flow construct.
  **L302 CN**: 跳出当前控制流结构。
- **L303 EN**: Closes the current scope.
  **L303 CN**: 关闭当前作用域。
- **L304 EN**: Executes statement `Packet.push_back(SU);`.
  **L304 CN**: 执行语句 `Packet.push_back(SU);`。
- **L305 EN**: Closes the current scope.
  **L305 CN**: 关闭当前作用域。
- **L306 EN**: Comment documents: `Forcefully end packet for PseudoOps.`.
  **L306 CN**: 注释说明：`Forcefully end packet for PseudoOps.`。
- **L307 EN**: Handles the fallback branch.
  **L307 CN**: 处理兜底分支。
- **L308 EN**: Executes statement `ResourcesModel->clearResources();`.
  **L308 CN**: 执行语句 `ResourcesModel->clearResources();`。
- **L309 EN**: Executes statement `Packet.clear();`.
  **L309 CN**: 执行语句 `Packet.clear();`。
- **L310 EN**: Closes the current scope.
  **L310 CN**: 关闭当前作用域。
- **L311 EN**: Separates nearby statements for readability.
  **L311 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L312 EN**: Comment documents: `If packet is now full, reset the state so in the next cycle`.
  **L312 CN**: 注释说明：`If packet is now full, reset the state so in the next cycle`。
- **L313 EN**: Comment documents: `we start fresh.`.
  **L313 CN**: 注释说明：`we start fresh.`。
- **L314 EN**: Begins a conditional branch.
  **L314 CN**: 开始一个条件分支。
- **L315 EN**: Executes statement `ResourcesModel->clearResources();`.
  **L315 CN**: 执行语句 `ResourcesModel->clearResources();`。
- **L316 EN**: Executes statement `Packet.clear();`.
  **L316 CN**: 执行语句 `Packet.clear();`。
- **L317 EN**: Closes the current scope.
  **L317 CN**: 关闭当前作用域。
- **L318 EN**: Closes the current scope.
  **L318 CN**: 关闭当前作用域。
- **L319 EN**: Separates nearby statements for readability.
  **L319 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L320 EN**: Begins the definition of `rawRegPressureDelta`.
  **L320 CN**: 开始定义 `rawRegPressureDelta`。

### Lines 321-340

````cpp
  int RegBalance = 0;

  if (!SU || !SU->getNode() || !SU->getNode()->isMachineOpcode())
    return RegBalance;

  // Gen estimate.
  for (unsigned i = 0, e = SU->getNode()->getNumValues(); i != e; ++i) {
      MVT VT = SU->getNode()->getSimpleValueType(i);
      if (TLI->isTypeLegal(VT)
          && TLI->getRegClassFor(VT)
          && TLI->getRegClassFor(VT)->getID() == RCId)
        RegBalance += numberRCValSuccInSU(SU, RCId);
  }
  // Kill estimate.
  for (unsigned i = 0, e = SU->getNode()->getNumOperands(); i != e; ++i) {
      const SDValue &Op = SU->getNode()->getOperand(i);
      MVT VT = Op.getNode()->getSimpleValueType(Op.getResNo());
      if (isa<ConstantSDNode>(Op.getNode()))
        continue;

````
- **L321 EN**: Assigns or initializes `int RegBalance`.
  **L321 CN**: 对 `int RegBalance` 进行赋值或初始化。
- **L322 EN**: Separates nearby statements for readability.
  **L322 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L323 EN**: Begins a conditional branch.
  **L323 CN**: 开始一个条件分支。
- **L324 EN**: Returns `RegBalance` to the caller.
  **L324 CN**: 向调用者返回 `RegBalance`。
- **L325 EN**: Separates nearby statements for readability.
  **L325 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L326 EN**: Comment documents: `Gen estimate.`.
  **L326 CN**: 注释说明：`Gen estimate.`。
- **L327 EN**: Starts a loop over a sequence or range.
  **L327 CN**: 开始遍历序列或范围的循环。
- **L328 EN**: Assigns or initializes `MVT VT`.
  **L328 CN**: 对 `MVT VT` 进行赋值或初始化。
- **L329 EN**: Begins a conditional branch.
  **L329 CN**: 开始一个条件分支。
- **L330 EN**: Continues logic with `&& TLI->getRegClassFor(VT)`.
  **L330 CN**: 继续处理逻辑：`&& TLI->getRegClassFor(VT)`。
- **L331 EN**: Continues logic with `&& TLI->getRegClassFor(VT)->getID() == RCId)`.
  **L331 CN**: 继续处理逻辑：`&& TLI->getRegClassFor(VT)->getID() == RCId)`。
- **L332 EN**: Assigns or initializes `RegBalance +`.
  **L332 CN**: 对 `RegBalance +` 进行赋值或初始化。
- **L333 EN**: Closes the current scope.
  **L333 CN**: 关闭当前作用域。
- **L334 EN**: Comment documents: `Kill estimate.`.
  **L334 CN**: 注释说明：`Kill estimate.`。
- **L335 EN**: Starts a loop over a sequence or range.
  **L335 CN**: 开始遍历序列或范围的循环。
- **L336 EN**: Assigns or initializes `const SDValue &Op`.
  **L336 CN**: 对 `const SDValue &Op` 进行赋值或初始化。
- **L337 EN**: Assigns or initializes `MVT VT`.
  **L337 CN**: 对 `MVT VT` 进行赋值或初始化。
- **L338 EN**: Begins a conditional branch.
  **L338 CN**: 开始一个条件分支。
- **L339 EN**: Skips to the next loop iteration.
  **L339 CN**: 跳到下一次循环迭代。
- **L340 EN**: Separates nearby statements for readability.
  **L340 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 341-360

````cpp
      if (TLI->isTypeLegal(VT) && TLI->getRegClassFor(VT)
          && TLI->getRegClassFor(VT)->getID() == RCId)
        RegBalance -= numberRCValPredInSU(SU, RCId);
  }
  return RegBalance;
}

/// Estimates change in reg pressure from this SU.
/// It is achieved by trivial tracking of defined
/// and used vregs in dependent instructions.
/// The RawPressure flag makes this function to ignore
/// existing reg file sizes, and report raw def/use
/// balance.
int ResourcePriorityQueue::regPressureDelta(SUnit *SU, bool RawPressure) {
  int RegBalance = 0;

  if (!SU || !SU->getNode() || !SU->getNode()->isMachineOpcode())
    return RegBalance;

  if (RawPressure) {
````
- **L341 EN**: Begins a conditional branch.
  **L341 CN**: 开始一个条件分支。
- **L342 EN**: Continues logic with `&& TLI->getRegClassFor(VT)->getID() == RCId)`.
  **L342 CN**: 继续处理逻辑：`&& TLI->getRegClassFor(VT)->getID() == RCId)`。
- **L343 EN**: Assigns or initializes `RegBalance -`.
  **L343 CN**: 对 `RegBalance -` 进行赋值或初始化。
- **L344 EN**: Closes the current scope.
  **L344 CN**: 关闭当前作用域。
- **L345 EN**: Returns `RegBalance` to the caller.
  **L345 CN**: 向调用者返回 `RegBalance`。
- **L346 EN**: Closes the current scope.
  **L346 CN**: 关闭当前作用域。
- **L347 EN**: Separates nearby statements for readability.
  **L347 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L348 EN**: Comment documents: `Estimates change in reg pressure from this SU.`.
  **L348 CN**: 注释说明：`Estimates change in reg pressure from this SU.`。
- **L349 EN**: Comment documents: `It is achieved by trivial tracking of defined`.
  **L349 CN**: 注释说明：`It is achieved by trivial tracking of defined`。
- **L350 EN**: Comment documents: `and used vregs in dependent instructions.`.
  **L350 CN**: 注释说明：`and used vregs in dependent instructions.`。
- **L351 EN**: Comment documents: `The RawPressure flag makes this function to ignore`.
  **L351 CN**: 注释说明：`The RawPressure flag makes this function to ignore`。
- **L352 EN**: Comment documents: `existing reg file sizes, and report raw def/use`.
  **L352 CN**: 注释说明：`existing reg file sizes, and report raw def/use`。
- **L353 EN**: Comment documents: `balance.`.
  **L353 CN**: 注释说明：`balance.`。
- **L354 EN**: Begins the definition of `regPressureDelta`.
  **L354 CN**: 开始定义 `regPressureDelta`。
- **L355 EN**: Assigns or initializes `int RegBalance`.
  **L355 CN**: 对 `int RegBalance` 进行赋值或初始化。
- **L356 EN**: Separates nearby statements for readability.
  **L356 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L357 EN**: Begins a conditional branch.
  **L357 CN**: 开始一个条件分支。
- **L358 EN**: Returns `RegBalance` to the caller.
  **L358 CN**: 向调用者返回 `RegBalance`。
- **L359 EN**: Separates nearby statements for readability.
  **L359 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L360 EN**: Begins a conditional branch.
  **L360 CN**: 开始一个条件分支。

### Lines 361-380

````cpp
    for (const TargetRegisterClass *RC : TRI->regclasses())
      RegBalance += rawRegPressureDelta(SU, RC->getID());
  }
  else {
    for (const TargetRegisterClass *RC : TRI->regclasses()) {
      if ((RegPressure[RC->getID()] +
           rawRegPressureDelta(SU, RC->getID()) > 0) &&
          (RegPressure[RC->getID()] +
           rawRegPressureDelta(SU, RC->getID())  >= RegLimit[RC->getID()]))
        RegBalance += rawRegPressureDelta(SU, RC->getID());
    }
  }

  return RegBalance;
}

// Constants used to denote relative importance of
// heuristic components for cost computation.
static const unsigned PriorityOne = 200;
static const unsigned PriorityTwo = 50;
````
- **L361 EN**: Starts a loop over a sequence or range.
  **L361 CN**: 开始遍历序列或范围的循环。
- **L362 EN**: Assigns or initializes `RegBalance +`.
  **L362 CN**: 对 `RegBalance +` 进行赋值或初始化。
- **L363 EN**: Closes the current scope.
  **L363 CN**: 关闭当前作用域。
- **L364 EN**: Handles the fallback branch.
  **L364 CN**: 处理兜底分支。
- **L365 EN**: Starts a loop over a sequence or range.
  **L365 CN**: 开始遍历序列或范围的循环。
- **L366 EN**: Begins a conditional branch.
  **L366 CN**: 开始一个条件分支。
- **L367 EN**: Continues logic with `rawRegPressureDelta(SU, RC->getID()) > 0) &&`.
  **L367 CN**: 继续处理逻辑：`rawRegPressureDelta(SU, RC->getID()) > 0) &&`。
- **L368 EN**: Continues logic with `(RegPressure[RC->getID()] +`.
  **L368 CN**: 继续处理逻辑：`(RegPressure[RC->getID()] +`。
- **L369 EN**: Continues logic with `rawRegPressureDelta(SU, RC->getID()) >= RegLimit[RC->getID()]))`.
  **L369 CN**: 继续处理逻辑：`rawRegPressureDelta(SU, RC->getID()) >= RegLimit[RC->getID()]))`。
- **L370 EN**: Assigns or initializes `RegBalance +`.
  **L370 CN**: 对 `RegBalance +` 进行赋值或初始化。
- **L371 EN**: Closes the current scope.
  **L371 CN**: 关闭当前作用域。
- **L372 EN**: Closes the current scope.
  **L372 CN**: 关闭当前作用域。
- **L373 EN**: Separates nearby statements for readability.
  **L373 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L374 EN**: Returns `RegBalance` to the caller.
  **L374 CN**: 向调用者返回 `RegBalance`。
- **L375 EN**: Closes the current scope.
  **L375 CN**: 关闭当前作用域。
- **L376 EN**: Separates nearby statements for readability.
  **L376 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L377 EN**: Comment documents: `Constants used to denote relative importance of`.
  **L377 CN**: 注释说明：`Constants used to denote relative importance of`。
- **L378 EN**: Comment documents: `heuristic components for cost computation.`.
  **L378 CN**: 注释说明：`heuristic components for cost computation.`。
- **L379 EN**: Assigns or initializes `static const unsigned PriorityOne`.
  **L379 CN**: 对 `static const unsigned PriorityOne` 进行赋值或初始化。
- **L380 EN**: Assigns or initializes `static const unsigned PriorityTwo`.
  **L380 CN**: 对 `static const unsigned PriorityTwo` 进行赋值或初始化。

### Lines 381-400

````cpp
static const unsigned PriorityThree = 15;
static const unsigned PriorityFour = 5;
static const unsigned ScaleOne = 20;
static const unsigned ScaleTwo = 10;
static const unsigned ScaleThree = 5;
static const unsigned FactorOne = 2;

/// Returns single number reflecting benefit of scheduling SU
/// in the current cycle.
int ResourcePriorityQueue::SUSchedulingCost(SUnit *SU) {
  // Initial trivial priority.
  int ResCount = 1;

  // Do not waste time on a node that is already scheduled.
  if (SU->isScheduled)
    return ResCount;

  // Forced priority is high.
  if (SU->isScheduleHigh)
    ResCount += PriorityOne;
````
- **L381 EN**: Assigns or initializes `static const unsigned PriorityThree`.
  **L381 CN**: 对 `static const unsigned PriorityThree` 进行赋值或初始化。
- **L382 EN**: Assigns or initializes `static const unsigned PriorityFour`.
  **L382 CN**: 对 `static const unsigned PriorityFour` 进行赋值或初始化。
- **L383 EN**: Assigns or initializes `static const unsigned ScaleOne`.
  **L383 CN**: 对 `static const unsigned ScaleOne` 进行赋值或初始化。
- **L384 EN**: Assigns or initializes `static const unsigned ScaleTwo`.
  **L384 CN**: 对 `static const unsigned ScaleTwo` 进行赋值或初始化。
- **L385 EN**: Assigns or initializes `static const unsigned ScaleThree`.
  **L385 CN**: 对 `static const unsigned ScaleThree` 进行赋值或初始化。
- **L386 EN**: Assigns or initializes `static const unsigned FactorOne`.
  **L386 CN**: 对 `static const unsigned FactorOne` 进行赋值或初始化。
- **L387 EN**: Separates nearby statements for readability.
  **L387 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L388 EN**: Comment documents: `Returns single number reflecting benefit of scheduling SU`.
  **L388 CN**: 注释说明：`Returns single number reflecting benefit of scheduling SU`。
- **L389 EN**: Comment documents: `in the current cycle.`.
  **L389 CN**: 注释说明：`in the current cycle.`。
- **L390 EN**: Begins the definition of `SUSchedulingCost`.
  **L390 CN**: 开始定义 `SUSchedulingCost`。
- **L391 EN**: Comment documents: `Initial trivial priority.`.
  **L391 CN**: 注释说明：`Initial trivial priority.`。
- **L392 EN**: Assigns or initializes `int ResCount`.
  **L392 CN**: 对 `int ResCount` 进行赋值或初始化。
- **L393 EN**: Separates nearby statements for readability.
  **L393 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L394 EN**: Comment documents: `Do not waste time on a node that is already scheduled.`.
  **L394 CN**: 注释说明：`Do not waste time on a node that is already scheduled.`。
- **L395 EN**: Begins a conditional branch.
  **L395 CN**: 开始一个条件分支。
- **L396 EN**: Returns `ResCount` to the caller.
  **L396 CN**: 向调用者返回 `ResCount`。
- **L397 EN**: Separates nearby statements for readability.
  **L397 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L398 EN**: Comment documents: `Forced priority is high.`.
  **L398 CN**: 注释说明：`Forced priority is high.`。
- **L399 EN**: Begins a conditional branch.
  **L399 CN**: 开始一个条件分支。
- **L400 EN**: Assigns or initializes `ResCount +`.
  **L400 CN**: 对 `ResCount +` 进行赋值或初始化。

### Lines 401-420

````cpp

  // Adaptable scheduling
  // A small, but very parallel
  // region, where reg pressure is an issue.
  if (HorizontalVerticalBalance > RegPressureThreshold) {
    // Critical path first
    ResCount += (SU->getHeight() * ScaleTwo);
    // If resources are available for it, multiply the
    // chance of scheduling.
    if (isResourceAvailable(SU))
      ResCount <<= FactorOne;

    // Consider change to reg pressure from scheduling
    // this SU.
    ResCount -= (regPressureDelta(SU,true) * ScaleOne);
  }
  // Default heuristic, greeady and
  // critical path driven.
  else {
    // Critical path first.
````
- **L401 EN**: Separates nearby statements for readability.
  **L401 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L402 EN**: Comment documents: `Adaptable scheduling`.
  **L402 CN**: 注释说明：`Adaptable scheduling`。
- **L403 EN**: Comment documents: `A small, but very parallel`.
  **L403 CN**: 注释说明：`A small, but very parallel`。
- **L404 EN**: Comment documents: `region, where reg pressure is an issue.`.
  **L404 CN**: 注释说明：`region, where reg pressure is an issue.`。
- **L405 EN**: Begins a conditional branch.
  **L405 CN**: 开始一个条件分支。
- **L406 EN**: Comment documents: `Critical path first`.
  **L406 CN**: 注释说明：`Critical path first`。
- **L407 EN**: Assigns or initializes `ResCount +`.
  **L407 CN**: 对 `ResCount +` 进行赋值或初始化。
- **L408 EN**: Comment documents: `If resources are available for it, multiply the`.
  **L408 CN**: 注释说明：`If resources are available for it, multiply the`。
- **L409 EN**: Comment documents: `chance of scheduling.`.
  **L409 CN**: 注释说明：`chance of scheduling.`。
- **L410 EN**: Begins a conditional branch.
  **L410 CN**: 开始一个条件分支。
- **L411 EN**: Assigns or initializes `ResCount <<`.
  **L411 CN**: 对 `ResCount <<` 进行赋值或初始化。
- **L412 EN**: Separates nearby statements for readability.
  **L412 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L413 EN**: Comment documents: `Consider change to reg pressure from scheduling`.
  **L413 CN**: 注释说明：`Consider change to reg pressure from scheduling`。
- **L414 EN**: Comment documents: `this SU.`.
  **L414 CN**: 注释说明：`this SU.`。
- **L415 EN**: Assigns or initializes `ResCount -`.
  **L415 CN**: 对 `ResCount -` 进行赋值或初始化。
- **L416 EN**: Closes the current scope.
  **L416 CN**: 关闭当前作用域。
- **L417 EN**: Comment documents: `Default heuristic, greeady and`.
  **L417 CN**: 注释说明：`Default heuristic, greeady and`。
- **L418 EN**: Comment documents: `critical path driven.`.
  **L418 CN**: 注释说明：`critical path driven.`。
- **L419 EN**: Handles the fallback branch.
  **L419 CN**: 处理兜底分支。
- **L420 EN**: Comment documents: `Critical path first.`.
  **L420 CN**: 注释说明：`Critical path first.`。

### Lines 421-440

````cpp
    ResCount += (SU->getHeight() * ScaleTwo);
    // Now see how many instructions is blocked by this SU.
    ResCount += (NumNodesSolelyBlocking[SU->NodeNum] * ScaleTwo);
    // If resources are available for it, multiply the
    // chance of scheduling.
    if (isResourceAvailable(SU))
      ResCount <<= FactorOne;

    ResCount -= (regPressureDelta(SU) * ScaleTwo);
  }

  // These are platform-specific things.
  // Will need to go into the back end
  // and accessed from here via a hook.
  for (SDNode *N = SU->getNode(); N; N = N->getGluedNode()) {
    if (N->isMachineOpcode()) {
      const MCInstrDesc &TID = TII->get(N->getMachineOpcode());
      if (TID.isCall())
        ResCount += (PriorityTwo + (ScaleThree*N->getNumValues()));
    }
````
- **L421 EN**: Assigns or initializes `ResCount +`.
  **L421 CN**: 对 `ResCount +` 进行赋值或初始化。
- **L422 EN**: Comment documents: `Now see how many instructions is blocked by this SU.`.
  **L422 CN**: 注释说明：`Now see how many instructions is blocked by this SU.`。
- **L423 EN**: Assigns or initializes `ResCount +`.
  **L423 CN**: 对 `ResCount +` 进行赋值或初始化。
- **L424 EN**: Comment documents: `If resources are available for it, multiply the`.
  **L424 CN**: 注释说明：`If resources are available for it, multiply the`。
- **L425 EN**: Comment documents: `chance of scheduling.`.
  **L425 CN**: 注释说明：`chance of scheduling.`。
- **L426 EN**: Begins a conditional branch.
  **L426 CN**: 开始一个条件分支。
- **L427 EN**: Assigns or initializes `ResCount <<`.
  **L427 CN**: 对 `ResCount <<` 进行赋值或初始化。
- **L428 EN**: Separates nearby statements for readability.
  **L428 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L429 EN**: Assigns or initializes `ResCount -`.
  **L429 CN**: 对 `ResCount -` 进行赋值或初始化。
- **L430 EN**: Closes the current scope.
  **L430 CN**: 关闭当前作用域。
- **L431 EN**: Separates nearby statements for readability.
  **L431 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L432 EN**: Comment documents: `These are platform-specific things.`.
  **L432 CN**: 注释说明：`These are platform-specific things.`。
- **L433 EN**: Comment documents: `Will need to go into the back end`.
  **L433 CN**: 注释说明：`Will need to go into the back end`。
- **L434 EN**: Comment documents: `and accessed from here via a hook.`.
  **L434 CN**: 注释说明：`and accessed from here via a hook.`。
- **L435 EN**: Starts a loop over a sequence or range.
  **L435 CN**: 开始遍历序列或范围的循环。
- **L436 EN**: Begins a conditional branch.
  **L436 CN**: 开始一个条件分支。
- **L437 EN**: Assigns or initializes `const MCInstrDesc &TID`.
  **L437 CN**: 对 `const MCInstrDesc &TID` 进行赋值或初始化。
- **L438 EN**: Begins a conditional branch.
  **L438 CN**: 开始一个条件分支。
- **L439 EN**: Assigns or initializes `ResCount +`.
  **L439 CN**: 对 `ResCount +` 进行赋值或初始化。
- **L440 EN**: Closes the current scope.
  **L440 CN**: 关闭当前作用域。

### Lines 441-460

````cpp
    else
      switch (N->getOpcode()) {
      default:  break;
      case ISD::TokenFactor:
      case ISD::CopyFromReg:
      case ISD::CopyToReg:
        ResCount += PriorityFour;
        break;

      case ISD::INLINEASM:
      case ISD::INLINEASM_BR:
        ResCount += PriorityThree;
        break;
      }
  }
  return ResCount;
}


/// Main resource tracking point.
````
- **L441 EN**: Handles the fallback branch.
  **L441 CN**: 处理兜底分支。
- **L442 EN**: Starts a multi-way branch.
  **L442 CN**: 开始一个多路分支。
- **L443 EN**: Handles the default switch case.
  **L443 CN**: 处理 switch 的默认分支。
- **L444 EN**: Handles one switch case.
  **L444 CN**: 处理一个 switch 分支。
- **L445 EN**: Handles one switch case.
  **L445 CN**: 处理一个 switch 分支。
- **L446 EN**: Handles one switch case.
  **L446 CN**: 处理一个 switch 分支。
- **L447 EN**: Assigns or initializes `ResCount +`.
  **L447 CN**: 对 `ResCount +` 进行赋值或初始化。
- **L448 EN**: Breaks out of the current control-flow construct.
  **L448 CN**: 跳出当前控制流结构。
- **L449 EN**: Separates nearby statements for readability.
  **L449 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L450 EN**: Handles one switch case.
  **L450 CN**: 处理一个 switch 分支。
- **L451 EN**: Handles one switch case.
  **L451 CN**: 处理一个 switch 分支。
- **L452 EN**: Assigns or initializes `ResCount +`.
  **L452 CN**: 对 `ResCount +` 进行赋值或初始化。
- **L453 EN**: Breaks out of the current control-flow construct.
  **L453 CN**: 跳出当前控制流结构。
- **L454 EN**: Closes the current scope.
  **L454 CN**: 关闭当前作用域。
- **L455 EN**: Closes the current scope.
  **L455 CN**: 关闭当前作用域。
- **L456 EN**: Returns `ResCount` to the caller.
  **L456 CN**: 向调用者返回 `ResCount`。
- **L457 EN**: Closes the current scope.
  **L457 CN**: 关闭当前作用域。
- **L458 EN**: Separates nearby statements for readability.
  **L458 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L459 EN**: Separates nearby statements for readability.
  **L459 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L460 EN**: Comment documents: `Main resource tracking point.`.
  **L460 CN**: 注释说明：`Main resource tracking point.`。

### Lines 461-480

````cpp
void ResourcePriorityQueue::scheduledNode(SUnit *SU) {
  // Use NULL entry as an event marker to reset
  // the DFA state.
  if (!SU) {
    ResourcesModel->clearResources();
    Packet.clear();
    return;
  }

  const SDNode *ScegN = SU->getNode();
  // Update reg pressure tracking.
  // First update current node.
  if (ScegN->isMachineOpcode()) {
    // Estimate generated regs.
    for (unsigned i = 0, e = ScegN->getNumValues(); i != e; ++i) {
      MVT VT = ScegN->getSimpleValueType(i);

      if (TLI->isTypeLegal(VT)) {
        const TargetRegisterClass *RC = TLI->getRegClassFor(VT);
        if (RC)
````
- **L461 EN**: Begins the definition of `scheduledNode`.
  **L461 CN**: 开始定义 `scheduledNode`。
- **L462 EN**: Comment documents: `Use NULL entry as an event marker to reset`.
  **L462 CN**: 注释说明：`Use NULL entry as an event marker to reset`。
- **L463 EN**: Comment documents: `the DFA state.`.
  **L463 CN**: 注释说明：`the DFA state.`。
- **L464 EN**: Begins a conditional branch.
  **L464 CN**: 开始一个条件分支。
- **L465 EN**: Executes statement `ResourcesModel->clearResources();`.
  **L465 CN**: 执行语句 `ResourcesModel->clearResources();`。
- **L466 EN**: Executes statement `Packet.clear();`.
  **L466 CN**: 执行语句 `Packet.clear();`。
- **L467 EN**: Returns control to the caller.
  **L467 CN**: 将控制流返回给调用者。
- **L468 EN**: Closes the current scope.
  **L468 CN**: 关闭当前作用域。
- **L469 EN**: Separates nearby statements for readability.
  **L469 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L470 EN**: Assigns or initializes `const SDNode *ScegN`.
  **L470 CN**: 对 `const SDNode *ScegN` 进行赋值或初始化。
- **L471 EN**: Comment documents: `Update reg pressure tracking.`.
  **L471 CN**: 注释说明：`Update reg pressure tracking.`。
- **L472 EN**: Comment documents: `First update current node.`.
  **L472 CN**: 注释说明：`First update current node.`。
- **L473 EN**: Begins a conditional branch.
  **L473 CN**: 开始一个条件分支。
- **L474 EN**: Comment documents: `Estimate generated regs.`.
  **L474 CN**: 注释说明：`Estimate generated regs.`。
- **L475 EN**: Starts a loop over a sequence or range.
  **L475 CN**: 开始遍历序列或范围的循环。
- **L476 EN**: Assigns or initializes `MVT VT`.
  **L476 CN**: 对 `MVT VT` 进行赋值或初始化。
- **L477 EN**: Separates nearby statements for readability.
  **L477 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L478 EN**: Begins a conditional branch.
  **L478 CN**: 开始一个条件分支。
- **L479 EN**: Assigns or initializes `const TargetRegisterClass *RC`.
  **L479 CN**: 对 `const TargetRegisterClass *RC` 进行赋值或初始化。
- **L480 EN**: Begins a conditional branch.
  **L480 CN**: 开始一个条件分支。

### Lines 481-500

````cpp
          RegPressure[RC->getID()] += numberRCValSuccInSU(SU, RC->getID());
      }
    }
    // Estimate killed regs.
    for (unsigned i = 0, e = ScegN->getNumOperands(); i != e; ++i) {
      const SDValue &Op = ScegN->getOperand(i);
      MVT VT = Op.getNode()->getSimpleValueType(Op.getResNo());

      if (TLI->isTypeLegal(VT)) {
        const TargetRegisterClass *RC = TLI->getRegClassFor(VT);
        if (RC) {
          if (RegPressure[RC->getID()] >
            (numberRCValPredInSU(SU, RC->getID())))
            RegPressure[RC->getID()] -= numberRCValPredInSU(SU, RC->getID());
          else RegPressure[RC->getID()] = 0;
        }
      }
    }
    for (SDep &Pred : SU->Preds) {
      if (Pred.isCtrl() || (Pred.getSUnit()->NumRegDefsLeft == 0))
````
- **L481 EN**: Assigns or initializes `RegPressure[RC->getID()] +`.
  **L481 CN**: 对 `RegPressure[RC->getID()] +` 进行赋值或初始化。
- **L482 EN**: Closes the current scope.
  **L482 CN**: 关闭当前作用域。
- **L483 EN**: Closes the current scope.
  **L483 CN**: 关闭当前作用域。
- **L484 EN**: Comment documents: `Estimate killed regs.`.
  **L484 CN**: 注释说明：`Estimate killed regs.`。
- **L485 EN**: Starts a loop over a sequence or range.
  **L485 CN**: 开始遍历序列或范围的循环。
- **L486 EN**: Assigns or initializes `const SDValue &Op`.
  **L486 CN**: 对 `const SDValue &Op` 进行赋值或初始化。
- **L487 EN**: Assigns or initializes `MVT VT`.
  **L487 CN**: 对 `MVT VT` 进行赋值或初始化。
- **L488 EN**: Separates nearby statements for readability.
  **L488 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L489 EN**: Begins a conditional branch.
  **L489 CN**: 开始一个条件分支。
- **L490 EN**: Assigns or initializes `const TargetRegisterClass *RC`.
  **L490 CN**: 对 `const TargetRegisterClass *RC` 进行赋值或初始化。
- **L491 EN**: Begins a conditional branch.
  **L491 CN**: 开始一个条件分支。
- **L492 EN**: Begins a conditional branch.
  **L492 CN**: 开始一个条件分支。
- **L493 EN**: Continues logic with `(numberRCValPredInSU(SU, RC->getID())))`.
  **L493 CN**: 继续处理逻辑：`(numberRCValPredInSU(SU, RC->getID())))`。
- **L494 EN**: Assigns or initializes `RegPressure[RC->getID()] -`.
  **L494 CN**: 对 `RegPressure[RC->getID()] -` 进行赋值或初始化。
- **L495 EN**: Handles the fallback branch.
  **L495 CN**: 处理兜底分支。
- **L496 EN**: Closes the current scope.
  **L496 CN**: 关闭当前作用域。
- **L497 EN**: Closes the current scope.
  **L497 CN**: 关闭当前作用域。
- **L498 EN**: Closes the current scope.
  **L498 CN**: 关闭当前作用域。
- **L499 EN**: Starts a loop over a sequence or range.
  **L499 CN**: 开始遍历序列或范围的循环。
- **L500 EN**: Begins a conditional branch.
  **L500 CN**: 开始一个条件分支。

### Lines 501-520

````cpp
        continue;
      --Pred.getSUnit()->NumRegDefsLeft;
    }
  }

  // Reserve resources for this SU.
  reserveResources(SU);

  // Adjust number of parallel live ranges.
  // Heuristic is simple - node with no data successors reduces
  // number of live ranges. All others, increase it.
  unsigned NumberNonControlDeps = 0;

  for (const SDep &Succ : SU->Succs) {
    adjustPriorityOfUnscheduledPreds(Succ.getSUnit());
    if (!Succ.isCtrl())
      NumberNonControlDeps++;
  }

  if (!NumberNonControlDeps) {
````
- **L501 EN**: Skips to the next loop iteration.
  **L501 CN**: 跳到下一次循环迭代。
- **L502 EN**: Executes statement `--Pred.getSUnit()->NumRegDefsLeft;`.
  **L502 CN**: 执行语句 `--Pred.getSUnit()->NumRegDefsLeft;`。
- **L503 EN**: Closes the current scope.
  **L503 CN**: 关闭当前作用域。
- **L504 EN**: Closes the current scope.
  **L504 CN**: 关闭当前作用域。
- **L505 EN**: Separates nearby statements for readability.
  **L505 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L506 EN**: Comment documents: `Reserve resources for this SU.`.
  **L506 CN**: 注释说明：`Reserve resources for this SU.`。
- **L507 EN**: Executes statement `reserveResources(SU);`.
  **L507 CN**: 执行语句 `reserveResources(SU);`。
- **L508 EN**: Separates nearby statements for readability.
  **L508 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L509 EN**: Comment documents: `Adjust number of parallel live ranges.`.
  **L509 CN**: 注释说明：`Adjust number of parallel live ranges.`。
- **L510 EN**: Comment documents: `Heuristic is simple - node with no data successors reduces`.
  **L510 CN**: 注释说明：`Heuristic is simple - node with no data successors reduces`。
- **L511 EN**: Comment documents: `number of live ranges. All others, increase it.`.
  **L511 CN**: 注释说明：`number of live ranges. All others, increase it.`。
- **L512 EN**: Assigns or initializes `unsigned NumberNonControlDeps`.
  **L512 CN**: 对 `unsigned NumberNonControlDeps` 进行赋值或初始化。
- **L513 EN**: Separates nearby statements for readability.
  **L513 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L514 EN**: Starts a loop over a sequence or range.
  **L514 CN**: 开始遍历序列或范围的循环。
- **L515 EN**: Executes statement `adjustPriorityOfUnscheduledPreds(Succ.getSUnit());`.
  **L515 CN**: 执行语句 `adjustPriorityOfUnscheduledPreds(Succ.getSUnit());`。
- **L516 EN**: Begins a conditional branch.
  **L516 CN**: 开始一个条件分支。
- **L517 EN**: Executes statement `NumberNonControlDeps++;`.
  **L517 CN**: 执行语句 `NumberNonControlDeps++;`。
- **L518 EN**: Closes the current scope.
  **L518 CN**: 关闭当前作用域。
- **L519 EN**: Separates nearby statements for readability.
  **L519 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L520 EN**: Begins a conditional branch.
  **L520 CN**: 开始一个条件分支。

### Lines 521-540

````cpp
    if (ParallelLiveRanges >= SU->NumPreds)
      ParallelLiveRanges -= SU->NumPreds;
    else
      ParallelLiveRanges = 0;

  }
  else
    ParallelLiveRanges += SU->NumRegDefsLeft;

  // Track parallel live chains.
  HorizontalVerticalBalance += (SU->Succs.size() - numberCtrlDepsInSU(SU));
  HorizontalVerticalBalance -= (SU->Preds.size() - numberCtrlPredInSU(SU));
}

void ResourcePriorityQueue::initNumRegDefsLeft(SUnit *SU) {
  unsigned  NodeNumDefs = 0;
  for (SDNode *N = SU->getNode(); N; N = N->getGluedNode())
    if (N->isMachineOpcode()) {
      const MCInstrDesc &TID = TII->get(N->getMachineOpcode());
      // No register need be allocated for this.
````
- **L521 EN**: Begins a conditional branch.
  **L521 CN**: 开始一个条件分支。
- **L522 EN**: Assigns or initializes `ParallelLiveRanges -`.
  **L522 CN**: 对 `ParallelLiveRanges -` 进行赋值或初始化。
- **L523 EN**: Handles the fallback branch.
  **L523 CN**: 处理兜底分支。
- **L524 EN**: Assigns or initializes `ParallelLiveRanges`.
  **L524 CN**: 对 `ParallelLiveRanges` 进行赋值或初始化。
- **L525 EN**: Separates nearby statements for readability.
  **L525 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L526 EN**: Closes the current scope.
  **L526 CN**: 关闭当前作用域。
- **L527 EN**: Handles the fallback branch.
  **L527 CN**: 处理兜底分支。
- **L528 EN**: Assigns or initializes `ParallelLiveRanges +`.
  **L528 CN**: 对 `ParallelLiveRanges +` 进行赋值或初始化。
- **L529 EN**: Separates nearby statements for readability.
  **L529 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L530 EN**: Comment documents: `Track parallel live chains.`.
  **L530 CN**: 注释说明：`Track parallel live chains.`。
- **L531 EN**: Assigns or initializes `HorizontalVerticalBalance +`.
  **L531 CN**: 对 `HorizontalVerticalBalance +` 进行赋值或初始化。
- **L532 EN**: Assigns or initializes `HorizontalVerticalBalance -`.
  **L532 CN**: 对 `HorizontalVerticalBalance -` 进行赋值或初始化。
- **L533 EN**: Closes the current scope.
  **L533 CN**: 关闭当前作用域。
- **L534 EN**: Separates nearby statements for readability.
  **L534 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L535 EN**: Begins the definition of `initNumRegDefsLeft`.
  **L535 CN**: 开始定义 `initNumRegDefsLeft`。
- **L536 EN**: Assigns or initializes `unsigned NodeNumDefs`.
  **L536 CN**: 对 `unsigned NodeNumDefs` 进行赋值或初始化。
- **L537 EN**: Starts a loop over a sequence or range.
  **L537 CN**: 开始遍历序列或范围的循环。
- **L538 EN**: Begins a conditional branch.
  **L538 CN**: 开始一个条件分支。
- **L539 EN**: Assigns or initializes `const MCInstrDesc &TID`.
  **L539 CN**: 对 `const MCInstrDesc &TID` 进行赋值或初始化。
- **L540 EN**: Comment documents: `No register need be allocated for this.`.
  **L540 CN**: 注释说明：`No register need be allocated for this.`。

### Lines 541-560

````cpp
      if (N->getMachineOpcode() == TargetOpcode::IMPLICIT_DEF) {
        NodeNumDefs = 0;
        break;
      }
      NodeNumDefs = std::min(N->getNumValues(), TID.getNumDefs());
    }
    else
      switch(N->getOpcode()) {
        default:     break;
        case ISD::CopyFromReg:
          NodeNumDefs++;
          break;
        case ISD::INLINEASM:
        case ISD::INLINEASM_BR:
          NodeNumDefs++;
          break;
      }

  SU->NumRegDefsLeft = NodeNumDefs;
}
````
- **L541 EN**: Begins a conditional branch.
  **L541 CN**: 开始一个条件分支。
- **L542 EN**: Assigns or initializes `NodeNumDefs`.
  **L542 CN**: 对 `NodeNumDefs` 进行赋值或初始化。
- **L543 EN**: Breaks out of the current control-flow construct.
  **L543 CN**: 跳出当前控制流结构。
- **L544 EN**: Closes the current scope.
  **L544 CN**: 关闭当前作用域。
- **L545 EN**: Declares function or method `min`.
  **L545 CN**: 声明函数或方法 `min`。
- **L546 EN**: Closes the current scope.
  **L546 CN**: 关闭当前作用域。
- **L547 EN**: Handles the fallback branch.
  **L547 CN**: 处理兜底分支。
- **L548 EN**: Starts a multi-way branch.
  **L548 CN**: 开始一个多路分支。
- **L549 EN**: Handles the default switch case.
  **L549 CN**: 处理 switch 的默认分支。
- **L550 EN**: Handles one switch case.
  **L550 CN**: 处理一个 switch 分支。
- **L551 EN**: Executes statement `NodeNumDefs++;`.
  **L551 CN**: 执行语句 `NodeNumDefs++;`。
- **L552 EN**: Breaks out of the current control-flow construct.
  **L552 CN**: 跳出当前控制流结构。
- **L553 EN**: Handles one switch case.
  **L553 CN**: 处理一个 switch 分支。
- **L554 EN**: Handles one switch case.
  **L554 CN**: 处理一个 switch 分支。
- **L555 EN**: Executes statement `NodeNumDefs++;`.
  **L555 CN**: 执行语句 `NodeNumDefs++;`。
- **L556 EN**: Breaks out of the current control-flow construct.
  **L556 CN**: 跳出当前控制流结构。
- **L557 EN**: Closes the current scope.
  **L557 CN**: 关闭当前作用域。
- **L558 EN**: Separates nearby statements for readability.
  **L558 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L559 EN**: Assigns or initializes `SU->NumRegDefsLeft`.
  **L559 CN**: 对 `SU->NumRegDefsLeft` 进行赋值或初始化。
- **L560 EN**: Closes the current scope.
  **L560 CN**: 关闭当前作用域。

### Lines 561-580

````cpp

/// adjustPriorityOfUnscheduledPreds - One of the predecessors of SU was just
/// scheduled.  If SU is not itself available, then there is at least one
/// predecessor node that has not been scheduled yet.  If SU has exactly ONE
/// unscheduled predecessor, we want to increase its priority: it getting
/// scheduled will make this node available, so it is better than some other
/// node of the same priority that will not make a node available.
void ResourcePriorityQueue::adjustPriorityOfUnscheduledPreds(SUnit *SU) {
  if (SU->isAvailable) return;  // All preds scheduled.

  SUnit *OnlyAvailablePred = getSingleUnscheduledPred(SU);
  if (!OnlyAvailablePred || !OnlyAvailablePred->isAvailable)
    return;

  // Okay, we found a single predecessor that is available, but not scheduled.
  // Since it is available, it must be in the priority queue.  First remove it.
  remove(OnlyAvailablePred);

  // Reinsert the node into the priority queue, which recomputes its
  // NumNodesSolelyBlocking value.
````
- **L561 EN**: Separates nearby statements for readability.
  **L561 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L562 EN**: Comment documents: `adjustPriorityOfUnscheduledPreds - One of the predecessors of SU was jus…`.
  **L562 CN**: 注释说明：`adjustPriorityOfUnscheduledPreds - One of the predecessors of SU was jus…`。
- **L563 EN**: Comment documents: `scheduled. If SU is not itself available, then there is at least one`.
  **L563 CN**: 注释说明：`scheduled. If SU is not itself available, then there is at least one`。
- **L564 EN**: Comment documents: `predecessor node that has not been scheduled yet. If SU has exactly ONE`.
  **L564 CN**: 注释说明：`predecessor node that has not been scheduled yet. If SU has exactly ONE`。
- **L565 EN**: Comment documents: `unscheduled predecessor, we want to increase its priority: it getting`.
  **L565 CN**: 注释说明：`unscheduled predecessor, we want to increase its priority: it getting`。
- **L566 EN**: Comment documents: `scheduled will make this node available, so it is better than some other`.
  **L566 CN**: 注释说明：`scheduled will make this node available, so it is better than some other`。
- **L567 EN**: Comment documents: `node of the same priority that will not make a node available.`.
  **L567 CN**: 注释说明：`node of the same priority that will not make a node available.`。
- **L568 EN**: Begins the definition of `adjustPriorityOfUnscheduledPreds`.
  **L568 CN**: 开始定义 `adjustPriorityOfUnscheduledPreds`。
- **L569 EN**: Begins a conditional branch.
  **L569 CN**: 开始一个条件分支。
- **L570 EN**: Separates nearby statements for readability.
  **L570 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L571 EN**: Assigns or initializes `SUnit *OnlyAvailablePred`.
  **L571 CN**: 对 `SUnit *OnlyAvailablePred` 进行赋值或初始化。
- **L572 EN**: Begins a conditional branch.
  **L572 CN**: 开始一个条件分支。
- **L573 EN**: Returns control to the caller.
  **L573 CN**: 将控制流返回给调用者。
- **L574 EN**: Separates nearby statements for readability.
  **L574 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L575 EN**: Comment documents: `Okay, we found a single predecessor that is available, but not scheduled…`.
  **L575 CN**: 注释说明：`Okay, we found a single predecessor that is available, but not scheduled…`。
- **L576 EN**: Comment documents: `Since it is available, it must be in the priority queue. First remove it…`.
  **L576 CN**: 注释说明：`Since it is available, it must be in the priority queue. First remove it…`。
- **L577 EN**: Executes statement `remove(OnlyAvailablePred);`.
  **L577 CN**: 执行语句 `remove(OnlyAvailablePred);`。
- **L578 EN**: Separates nearby statements for readability.
  **L578 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L579 EN**: Comment documents: `Reinsert the node into the priority queue, which recomputes its`.
  **L579 CN**: 注释说明：`Reinsert the node into the priority queue, which recomputes its`。
- **L580 EN**: Comment documents: `NumNodesSolelyBlocking value.`.
  **L580 CN**: 注释说明：`NumNodesSolelyBlocking value.`。

### Lines 581-600

````cpp
  push(OnlyAvailablePred);
}


/// Main access point - returns next instructions
/// to be placed in scheduling sequence.
SUnit *ResourcePriorityQueue::pop() {
  if (empty())
    return nullptr;

  std::vector<SUnit *>::iterator Best = Queue.begin();
  if (!DisableDFASched) {
    int BestCost = SUSchedulingCost(*Best);
    for (auto I = std::next(Queue.begin()), E = Queue.end(); I != E; ++I) {

      if (SUSchedulingCost(*I) > BestCost) {
        BestCost = SUSchedulingCost(*I);
        Best = I;
      }
    }
````
- **L581 EN**: Executes statement `push(OnlyAvailablePred);`.
  **L581 CN**: 执行语句 `push(OnlyAvailablePred);`。
- **L582 EN**: Closes the current scope.
  **L582 CN**: 关闭当前作用域。
- **L583 EN**: Separates nearby statements for readability.
  **L583 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L584 EN**: Separates nearby statements for readability.
  **L584 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L585 EN**: Comment documents: `Main access point - returns next instructions`.
  **L585 CN**: 注释说明：`Main access point - returns next instructions`。
- **L586 EN**: Comment documents: `to be placed in scheduling sequence.`.
  **L586 CN**: 注释说明：`to be placed in scheduling sequence.`。
- **L587 EN**: Begins the definition of `pop`.
  **L587 CN**: 开始定义 `pop`。
- **L588 EN**: Begins a conditional branch.
  **L588 CN**: 开始一个条件分支。
- **L589 EN**: Returns `nullptr` to the caller.
  **L589 CN**: 向调用者返回 `nullptr`。
- **L590 EN**: Separates nearby statements for readability.
  **L590 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L591 EN**: Assigns or initializes `std::vector<SUnit *>::iterator Best`.
  **L591 CN**: 对 `std::vector<SUnit *>::iterator Best` 进行赋值或初始化。
- **L592 EN**: Begins a conditional branch.
  **L592 CN**: 开始一个条件分支。
- **L593 EN**: Assigns or initializes `int BestCost`.
  **L593 CN**: 对 `int BestCost` 进行赋值或初始化。
- **L594 EN**: Starts a loop over a sequence or range.
  **L594 CN**: 开始遍历序列或范围的循环。
- **L595 EN**: Separates nearby statements for readability.
  **L595 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L596 EN**: Begins a conditional branch.
  **L596 CN**: 开始一个条件分支。
- **L597 EN**: Assigns or initializes `BestCost`.
  **L597 CN**: 对 `BestCost` 进行赋值或初始化。
- **L598 EN**: Assigns or initializes `Best`.
  **L598 CN**: 对 `Best` 进行赋值或初始化。
- **L599 EN**: Closes the current scope.
  **L599 CN**: 关闭当前作用域。
- **L600 EN**: Closes the current scope.
  **L600 CN**: 关闭当前作用域。

### Lines 601-620

````cpp
  }
  // Use default TD scheduling mechanism.
  else {
    for (auto I = std::next(Queue.begin()), E = Queue.end(); I != E; ++I)
      if (Picker(*Best, *I))
        Best = I;
  }

  SUnit *V = *Best;
  if (Best != std::prev(Queue.end()))
    std::swap(*Best, Queue.back());

  Queue.pop_back();

  return V;
}


void ResourcePriorityQueue::remove(SUnit *SU) {
  assert(!Queue.empty() && "Queue is empty!");
````
- **L601 EN**: Closes the current scope.
  **L601 CN**: 关闭当前作用域。
- **L602 EN**: Comment documents: `Use default TD scheduling mechanism.`.
  **L602 CN**: 注释说明：`Use default TD scheduling mechanism.`。
- **L603 EN**: Handles the fallback branch.
  **L603 CN**: 处理兜底分支。
- **L604 EN**: Starts a loop over a sequence or range.
  **L604 CN**: 开始遍历序列或范围的循环。
- **L605 EN**: Begins a conditional branch.
  **L605 CN**: 开始一个条件分支。
- **L606 EN**: Assigns or initializes `Best`.
  **L606 CN**: 对 `Best` 进行赋值或初始化。
- **L607 EN**: Closes the current scope.
  **L607 CN**: 关闭当前作用域。
- **L608 EN**: Separates nearby statements for readability.
  **L608 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L609 EN**: Assigns or initializes `SUnit *V`.
  **L609 CN**: 对 `SUnit *V` 进行赋值或初始化。
- **L610 EN**: Begins a conditional branch.
  **L610 CN**: 开始一个条件分支。
- **L611 EN**: Declares function or method `swap`.
  **L611 CN**: 声明函数或方法 `swap`。
- **L612 EN**: Separates nearby statements for readability.
  **L612 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L613 EN**: Executes statement `Queue.pop_back();`.
  **L613 CN**: 执行语句 `Queue.pop_back();`。
- **L614 EN**: Separates nearby statements for readability.
  **L614 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L615 EN**: Returns `V` to the caller.
  **L615 CN**: 向调用者返回 `V`。
- **L616 EN**: Closes the current scope.
  **L616 CN**: 关闭当前作用域。
- **L617 EN**: Separates nearby statements for readability.
  **L617 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L618 EN**: Separates nearby statements for readability.
  **L618 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L619 EN**: Begins the definition of `remove`.
  **L619 CN**: 开始定义 `remove`。
- **L620 EN**: Checks an invariant in debug builds.
  **L620 CN**: 在调试构建中检查一个不变量。

### Lines 621-626

````cpp
  std::vector<SUnit *>::iterator I = find(Queue, SU);
  if (I != std::prev(Queue.end()))
    std::swap(*I, Queue.back());

  Queue.pop_back();
}
````
- **L621 EN**: Assigns or initializes `std::vector<SUnit *>::iterator I`.
  **L621 CN**: 对 `std::vector<SUnit *>::iterator I` 进行赋值或初始化。
- **L622 EN**: Begins a conditional branch.
  **L622 CN**: 开始一个条件分支。
- **L623 EN**: Declares function or method `swap`.
  **L623 CN**: 声明函数或方法 `swap`。
- **L624 EN**: Separates nearby statements for readability.
  **L624 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L625 EN**: Executes statement `Queue.pop_back();`.
  **L625 CN**: 执行语句 `Queue.pop_back();`。
- **L626 EN**: Closes the current scope.
  **L626 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Instruction scheduling** / **指令调度**
- **Register management** / **寄存器管理**
- **Register pressure tracking** / **寄存器压力跟踪**
- **SelectionDAG lowering** / **SelectionDAG 降低**
- **Live range updates** / **活跃范围更新**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**
- **Pseudo-instruction expansion** / **伪指令展开**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/ResourcePriorityQueue.h`, `llvm/CodeGen/DFAPacketizer.h`, `llvm/CodeGen/SelectionDAGISel.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/Support/CommandLine.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
