# MacroFusion.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/MacroFusion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Macro Fusion` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Macro Fusion”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- MacroFusion.cpp - Macro Fusion -------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains the implementation of the DAG scheduling mutation
/// to pair instructions back to back.
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/MacroFusion.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/ScheduleDAG.h"
#include "llvm/CodeGen/ScheduleDAGInstrs.h"
#include "llvm/CodeGen/ScheduleDAGMutation.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
````
- **L1 EN**: Comment documents: `===- MacroFusion.cpp - Macro Fusion ------------------------------------…`.
  **L1 CN**: 注释说明：`===- MacroFusion.cpp - Macro Fusion ------------------------------------…`。
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
- **L9 EN**: Comment documents: `\file This file contains the implementation of the DAG scheduling mutati…`.
  **L9 CN**: 注释说明：`\file This file contains the implementation of the DAG scheduling mutati…`。
- **L10 EN**: Comment documents: `to pair instructions back to back.`.
  **L10 CN**: 注释说明：`to pair instructions back to back.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/MacroFusion.h` for MacroFusion support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MacroFusion.h`，用于 MacroFusion 相关支持。
- **L15 EN**: Includes LLVM header `llvm/ADT/Statistic.h` for Statistic support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/ADT/Statistic.h`，用于 Statistic 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/MachineInstr.h` for MachineInstr support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineInstr.h`，用于 MachineInstr 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/ScheduleDAG.h` for ScheduleDAG support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/ScheduleDAG.h`，用于 ScheduleDAG 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/ScheduleDAGInstrs.h` for ScheduleDAGInstrs support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/ScheduleDAGInstrs.h`，用于 ScheduleDAGInstrs 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/ScheduleDAGMutation.h` for ScheduleDAGMutation support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/ScheduleDAGMutation.h`，用于 ScheduleDAGMutation 相关支持。
- **L20 EN**: Includes LLVM header `llvm/CodeGen/TargetInstrInfo.h` for TargetInstrInfo support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetInstrInfo.h`，用于 TargetInstrInfo 相关支持。

### Lines 21-40

````cpp
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

#define DEBUG_TYPE "machine-scheduler"

STATISTIC(NumFused, "Number of instr pairs fused");
STATISTIC(NumFusionConflicts,
          "Number of conflicts between a fusion pair and an already existing "
          "cluster (either fusion or non-fusion)");

using namespace llvm;

static cl::opt<bool> EnableMacroFusion("misched-fusion", cl::Hidden,
  cl::desc("Enable scheduling for macro fusion."), cl::init(true));

static bool isHazard(const SDep &Dep) {
  return Dep.getKind() == SDep::Anti || Dep.getKind() == SDep::Output;
}

````
- **L21 EN**: Includes LLVM header `llvm/Support/CommandLine.h` for CommandLine support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/Support/CommandLine.h`，用于 CommandLine 相关支持。
- **L22 EN**: Includes LLVM header `llvm/Support/Debug.h` for Debug support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/Support/Debug.h`，用于 Debug 相关支持。
- **L23 EN**: Includes LLVM header `llvm/Support/raw_ostream.h` for raw_ostream support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/Support/raw_ostream.h`，用于 raw_ostream 相关支持。
- **L24 EN**: Separates nearby statements for readability.
  **L24 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L25 EN**: Defines the LLVM debug channel used by this file.
  **L25 CN**: 定义该文件使用的 LLVM 调试通道。
- **L26 EN**: Separates nearby statements for readability.
  **L26 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L27 EN**: Registers a pass statistic counter.
  **L27 CN**: 注册一个 pass 统计计数器。
- **L28 EN**: Registers a pass statistic counter.
  **L28 CN**: 注册一个 pass 统计计数器。
- **L29 EN**: Continues logic with `"Number of conflicts between a fusion pair and an already existing "`.
  **L29 CN**: 继续处理逻辑：`"Number of conflicts between a fusion pair and an already existing "`。
- **L30 EN**: Executes statement `"cluster (either fusion or non-fusion)");`.
  **L30 CN**: 执行语句 `"cluster (either fusion or non-fusion)");`。
- **L31 EN**: Separates nearby statements for readability.
  **L31 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L32 EN**: Imports namespace `llvm` into this translation unit.
  **L32 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L33 EN**: Separates nearby statements for readability.
  **L33 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L34 EN**: Declares LLVM command-line option `misched-fusion`.
  **L34 CN**: 声明 LLVM 命令行选项 `misched-fusion`。
- **L35 EN**: Declares function or method `desc`.
  **L35 CN**: 声明函数或方法 `desc`。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Begins the definition of `isHazard`.
  **L37 CN**: 开始定义 `isHazard`。
- **L38 EN**: Returns `Dep.getKind() == SDep::Anti || Dep.getKind() == SDep::Output` to the caller.
  **L38 CN**: 向调用者返回 `Dep.getKind() == SDep::Anti || Dep.getKind() == SDep::Output`。
- **L39 EN**: Closes the current scope.
  **L39 CN**: 关闭当前作用域。
- **L40 EN**: Separates nearby statements for readability.
  **L40 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 41-60

````cpp
static SUnit *getPredClusterSU(const SUnit &SU) {
  for (const SDep &SI : SU.Preds)
    if (SI.isCluster())
      return SI.getSUnit();

  return nullptr;
}

bool llvm::hasLessThanNumFused(const SUnit &SU, unsigned FuseLimit) {
  unsigned Num = 1;
  const SUnit *CurrentSU = &SU;
  while ((CurrentSU = getPredClusterSU(*CurrentSU)) && Num < FuseLimit) Num ++;
  return Num < FuseLimit;
}

bool llvm::fuseInstructionPair(ScheduleDAGInstrs &DAG, SUnit &FirstSU,
                               SUnit &SecondSU) {
  // Check that neither instr is already associated with a cluster (either
  // fusion or non-fusion)
  if (FirstSU.isClustered() || SecondSU.isClustered()) {
````
- **L41 EN**: Starts block `static SUnit *getPredClusterSU(const SUnit &SU)`.
  **L41 CN**: 开始代码块 `static SUnit *getPredClusterSU(const SUnit &SU)`。
- **L42 EN**: Starts a loop over a sequence or range.
  **L42 CN**: 开始遍历序列或范围的循环。
- **L43 EN**: Begins a conditional branch.
  **L43 CN**: 开始一个条件分支。
- **L44 EN**: Returns `SI.getSUnit()` to the caller.
  **L44 CN**: 向调用者返回 `SI.getSUnit()`。
- **L45 EN**: Separates nearby statements for readability.
  **L45 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L46 EN**: Returns `nullptr` to the caller.
  **L46 CN**: 向调用者返回 `nullptr`。
- **L47 EN**: Closes the current scope.
  **L47 CN**: 关闭当前作用域。
- **L48 EN**: Separates nearby statements for readability.
  **L48 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L49 EN**: Begins the definition of `hasLessThanNumFused`.
  **L49 CN**: 开始定义 `hasLessThanNumFused`。
- **L50 EN**: Assigns or initializes `unsigned Num`.
  **L50 CN**: 对 `unsigned Num` 进行赋值或初始化。
- **L51 EN**: Assigns or initializes `const SUnit *CurrentSU`.
  **L51 CN**: 对 `const SUnit *CurrentSU` 进行赋值或初始化。
- **L52 EN**: Starts a while loop controlled by a condition.
  **L52 CN**: 开始一个由条件控制的 while 循环。
- **L53 EN**: Returns `Num < FuseLimit` to the caller.
  **L53 CN**: 向调用者返回 `Num < FuseLimit`。
- **L54 EN**: Closes the current scope.
  **L54 CN**: 关闭当前作用域。
- **L55 EN**: Separates nearby statements for readability.
  **L55 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L56 EN**: Provides part of the signature for `fuseInstructionPair`.
  **L56 CN**: 给出 `fuseInstructionPair` 的一部分签名。
- **L57 EN**: Starts block `SUnit &SecondSU)`.
  **L57 CN**: 开始代码块 `SUnit &SecondSU)`。
- **L58 EN**: Comment documents: `Check that neither instr is already associated with a cluster (either`.
  **L58 CN**: 注释说明：`Check that neither instr is already associated with a cluster (either`。
- **L59 EN**: Comment documents: `fusion or non-fusion)`.
  **L59 CN**: 注释说明：`fusion or non-fusion)`。
- **L60 EN**: Begins a conditional branch.
  **L60 CN**: 开始一个条件分支。

### Lines 61-80

````cpp
    ++NumFusionConflicts;
    LLVM_DEBUG({
      dbgs() << "Fusion conflict: cannot fuse SU(" << FirstSU.NodeNum
             << ") and SU(" << SecondSU.NodeNum << ")\n";
      if (FirstSU.isClustered())
        dbgs() << "  SU(" << FirstSU.NodeNum << ") already clustered\n";
      if (SecondSU.isClustered())
        dbgs() << "  SU(" << SecondSU.NodeNum << ") already clustered\n";
    });
    return false;
  }

  // Create a single weak edge between the adjacent instrs. The only effect is
  // to cause bottom-up scheduling to heavily prioritize the clustered instrs.
  if (!DAG.addEdge(&SecondSU, SDep(&FirstSU, SDep::Cluster)))
    return false;

  auto &Clusters = DAG.getClusters();

  unsigned ClusterIdx = Clusters.size();
````
- **L61 EN**: Executes statement `++NumFusionConflicts;`.
  **L61 CN**: 执行语句 `++NumFusionConflicts;`。
- **L62 EN**: Emits debug-only tracing logic.
  **L62 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L63 EN**: Continues logic with `dbgs() << "Fusion conflict: cannot fuse SU(" << FirstSU.NodeNum`.
  **L63 CN**: 继续处理逻辑：`dbgs() << "Fusion conflict: cannot fuse SU(" << FirstSU.NodeNum`。
- **L64 EN**: Executes statement `<< ") and SU(" << SecondSU.NodeNum << ")\n";`.
  **L64 CN**: 执行语句 `<< ") and SU(" << SecondSU.NodeNum << ")\n";`。
- **L65 EN**: Begins a conditional branch.
  **L65 CN**: 开始一个条件分支。
- **L66 EN**: Executes statement `dbgs() << " SU(" << FirstSU.NodeNum << ") already clustered\n";`.
  **L66 CN**: 执行语句 `dbgs() << " SU(" << FirstSU.NodeNum << ") already clustered\n";`。
- **L67 EN**: Begins a conditional branch.
  **L67 CN**: 开始一个条件分支。
- **L68 EN**: Executes statement `dbgs() << " SU(" << SecondSU.NodeNum << ") already clustered\n";`.
  **L68 CN**: 执行语句 `dbgs() << " SU(" << SecondSU.NodeNum << ") already clustered\n";`。
- **L69 EN**: Executes statement `});`.
  **L69 CN**: 执行语句 `});`。
- **L70 EN**: Returns `false` to the caller.
  **L70 CN**: 向调用者返回 `false`。
- **L71 EN**: Closes the current scope.
  **L71 CN**: 关闭当前作用域。
- **L72 EN**: Separates nearby statements for readability.
  **L72 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L73 EN**: Comment documents: `Create a single weak edge between the adjacent instrs. The only effect i…`.
  **L73 CN**: 注释说明：`Create a single weak edge between the adjacent instrs. The only effect i…`。
- **L74 EN**: Comment documents: `to cause bottom-up scheduling to heavily prioritize the clustered instrs…`.
  **L74 CN**: 注释说明：`to cause bottom-up scheduling to heavily prioritize the clustered instrs…`。
- **L75 EN**: Begins a conditional branch.
  **L75 CN**: 开始一个条件分支。
- **L76 EN**: Returns `false` to the caller.
  **L76 CN**: 向调用者返回 `false`。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Assigns or initializes `auto &Clusters`.
  **L78 CN**: 对 `auto &Clusters` 进行赋值或初始化。
- **L79 EN**: Separates nearby statements for readability.
  **L79 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L80 EN**: Assigns or initializes `unsigned ClusterIdx`.
  **L80 CN**: 对 `unsigned ClusterIdx` 进行赋值或初始化。

### Lines 81-100

````cpp
  FirstSU.ParentClusterIdx = ClusterIdx;
  SecondSU.ParentClusterIdx = ClusterIdx;

  SmallPtrSet<SUnit *, 8> Cluster{{&FirstSU, &SecondSU}};
  Clusters.push_back(Cluster);

  // TODO - If we want to chain more than two instructions, we need to create
  // artifical edges to make dependencies from the FirstSU also dependent
  // on other chained instructions, and other chained instructions also
  // dependent on the dependencies of the SecondSU, to prevent them from being
  // scheduled into these chained instructions.
  assert(hasLessThanNumFused(FirstSU, 2) &&
         "Currently we only support chaining together two instructions");

  // Adjust the latency between both instrs.
  for (SDep &SI : FirstSU.Succs)
    if (SI.getSUnit() == &SecondSU)
      SI.setLatency(0);

  for (SDep &SI : SecondSU.Preds)
````
- **L81 EN**: Assigns or initializes `FirstSU.ParentClusterIdx`.
  **L81 CN**: 对 `FirstSU.ParentClusterIdx` 进行赋值或初始化。
- **L82 EN**: Assigns or initializes `SecondSU.ParentClusterIdx`.
  **L82 CN**: 对 `SecondSU.ParentClusterIdx` 进行赋值或初始化。
- **L83 EN**: Separates nearby statements for readability.
  **L83 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L84 EN**: Executes statement `SmallPtrSet<SUnit *, 8> Cluster{{&FirstSU, &SecondSU}};`.
  **L84 CN**: 执行语句 `SmallPtrSet<SUnit *, 8> Cluster{{&FirstSU, &SecondSU}};`。
- **L85 EN**: Executes statement `Clusters.push_back(Cluster);`.
  **L85 CN**: 执行语句 `Clusters.push_back(Cluster);`。
- **L86 EN**: Separates nearby statements for readability.
  **L86 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L87 EN**: Comment documents: `TODO - If we want to chain more than two instructions, we need to create`.
  **L87 CN**: 注释说明：`TODO - If we want to chain more than two instructions, we need to create`。
- **L88 EN**: Comment documents: `artifical edges to make dependencies from the FirstSU also dependent`.
  **L88 CN**: 注释说明：`artifical edges to make dependencies from the FirstSU also dependent`。
- **L89 EN**: Comment documents: `on other chained instructions, and other chained instructions also`.
  **L89 CN**: 注释说明：`on other chained instructions, and other chained instructions also`。
- **L90 EN**: Comment documents: `dependent on the dependencies of the SecondSU, to prevent them from bein…`.
  **L90 CN**: 注释说明：`dependent on the dependencies of the SecondSU, to prevent them from bein…`。
- **L91 EN**: Comment documents: `scheduled into these chained instructions.`.
  **L91 CN**: 注释说明：`scheduled into these chained instructions.`。
- **L92 EN**: Checks an invariant in debug builds.
  **L92 CN**: 在调试构建中检查一个不变量。
- **L93 EN**: Executes statement `"Currently we only support chaining together two instructions");`.
  **L93 CN**: 执行语句 `"Currently we only support chaining together two instructions");`。
- **L94 EN**: Separates nearby statements for readability.
  **L94 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L95 EN**: Comment documents: `Adjust the latency between both instrs.`.
  **L95 CN**: 注释说明：`Adjust the latency between both instrs.`。
- **L96 EN**: Starts a loop over a sequence or range.
  **L96 CN**: 开始遍历序列或范围的循环。
- **L97 EN**: Begins a conditional branch.
  **L97 CN**: 开始一个条件分支。
- **L98 EN**: Executes statement `SI.setLatency(0);`.
  **L98 CN**: 执行语句 `SI.setLatency(0);`。
- **L99 EN**: Separates nearby statements for readability.
  **L99 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L100 EN**: Starts a loop over a sequence or range.
  **L100 CN**: 开始遍历序列或范围的循环。

### Lines 101-120

````cpp
    if (SI.getSUnit() == &FirstSU)
      SI.setLatency(0);

  LLVM_DEBUG(
      dbgs() << "Macro fuse: "; DAG.dumpNodeName(FirstSU); dbgs() << " - ";
      DAG.dumpNodeName(SecondSU); dbgs() << " /  ";
      dbgs() << DAG.TII->getName(FirstSU.getInstr()->getOpcode()) << " - "
             << DAG.TII->getName(SecondSU.getInstr()->getOpcode()) << '\n';);

  // Make data dependencies from the FirstSU also dependent on the SecondSU to
  // prevent them from being scheduled between the FirstSU and the SecondSU.
  if (&SecondSU != &DAG.ExitSU)
    for (const SDep &SI : FirstSU.Succs) {
      SUnit *SU = SI.getSUnit();
      if (SI.isWeak() || isHazard(SI) ||
          SU == &DAG.ExitSU || SU == &SecondSU || SU->isPred(&SecondSU))
        continue;
      LLVM_DEBUG(dbgs() << "  Bind "; DAG.dumpNodeName(SecondSU);
                 dbgs() << " - "; DAG.dumpNodeName(*SU); dbgs() << '\n';);
      DAG.addEdge(SU, SDep(&SecondSU, SDep::Artificial));
````
- **L101 EN**: Begins a conditional branch.
  **L101 CN**: 开始一个条件分支。
- **L102 EN**: Executes statement `SI.setLatency(0);`.
  **L102 CN**: 执行语句 `SI.setLatency(0);`。
- **L103 EN**: Separates nearby statements for readability.
  **L103 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L104 EN**: Emits debug-only tracing logic.
  **L104 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L105 EN**: Executes statement `dbgs() << "Macro fuse: "; DAG.dumpNodeName(FirstSU); dbgs() << " - ";`.
  **L105 CN**: 执行语句 `dbgs() << "Macro fuse: "; DAG.dumpNodeName(FirstSU); dbgs() << " - ";`。
- **L106 EN**: Executes statement `DAG.dumpNodeName(SecondSU); dbgs() << " / ";`.
  **L106 CN**: 执行语句 `DAG.dumpNodeName(SecondSU); dbgs() << " / ";`。
- **L107 EN**: Continues logic with `dbgs() << DAG.TII->getName(FirstSU.getInstr()->getOpcode()) << " - "`.
  **L107 CN**: 继续处理逻辑：`dbgs() << DAG.TII->getName(FirstSU.getInstr()->getOpcode()) << " - "`。
- **L108 EN**: Executes statement `<< DAG.TII->getName(SecondSU.getInstr()->getOpcode()) << '\n';);`.
  **L108 CN**: 执行语句 `<< DAG.TII->getName(SecondSU.getInstr()->getOpcode()) << '\n';);`。
- **L109 EN**: Separates nearby statements for readability.
  **L109 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L110 EN**: Comment documents: `Make data dependencies from the FirstSU also dependent on the SecondSU t…`.
  **L110 CN**: 注释说明：`Make data dependencies from the FirstSU also dependent on the SecondSU t…`。
- **L111 EN**: Comment documents: `prevent them from being scheduled between the FirstSU and the SecondSU.`.
  **L111 CN**: 注释说明：`prevent them from being scheduled between the FirstSU and the SecondSU.`。
- **L112 EN**: Begins a conditional branch.
  **L112 CN**: 开始一个条件分支。
- **L113 EN**: Starts a loop over a sequence or range.
  **L113 CN**: 开始遍历序列或范围的循环。
- **L114 EN**: Assigns or initializes `SUnit *SU`.
  **L114 CN**: 对 `SUnit *SU` 进行赋值或初始化。
- **L115 EN**: Begins a conditional branch.
  **L115 CN**: 开始一个条件分支。
- **L116 EN**: Continues logic with `SU == &DAG.ExitSU || SU == &SecondSU || SU->isPred(&SecondSU))`.
  **L116 CN**: 继续处理逻辑：`SU == &DAG.ExitSU || SU == &SecondSU || SU->isPred(&SecondSU))`。
- **L117 EN**: Skips to the next loop iteration.
  **L117 CN**: 跳到下一次循环迭代。
- **L118 EN**: Emits debug-only tracing logic.
  **L118 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L119 EN**: Executes statement `dbgs() << " - "; DAG.dumpNodeName(*SU); dbgs() << '\n';);`.
  **L119 CN**: 执行语句 `dbgs() << " - "; DAG.dumpNodeName(*SU); dbgs() << '\n';);`。
- **L120 EN**: Executes statement `DAG.addEdge(SU, SDep(&SecondSU, SDep::Artificial));`.
  **L120 CN**: 执行语句 `DAG.addEdge(SU, SDep(&SecondSU, SDep::Artificial));`。

### Lines 121-140

````cpp
    }

  // Make the FirstSU also dependent on the dependencies of the SecondSU to
  // prevent them from being scheduled between the FirstSU and the SecondSU.
  if (&FirstSU != &DAG.EntrySU) {
    for (const SDep &SI : SecondSU.Preds) {
      SUnit *SU = SI.getSUnit();
      if (SI.isWeak() || isHazard(SI) || &FirstSU == SU || FirstSU.isSucc(SU))
        continue;
      LLVM_DEBUG(dbgs() << "  Bind "; DAG.dumpNodeName(*SU); dbgs() << " - ";
                 DAG.dumpNodeName(FirstSU); dbgs() << '\n';);
      DAG.addEdge(&FirstSU, SDep(SU, SDep::Artificial));
    }
    // ExitSU comes last by design, which acts like an implicit dependency
    // between ExitSU and any bottom root in the graph. We should transfer
    // this to FirstSU as well.
    if (&SecondSU == &DAG.ExitSU) {
      for (SUnit &SU : DAG.SUnits) {
        if (SU.Succs.empty())
          DAG.addEdge(&FirstSU, SDep(&SU, SDep::Artificial));
````
- **L121 EN**: Closes the current scope.
  **L121 CN**: 关闭当前作用域。
- **L122 EN**: Separates nearby statements for readability.
  **L122 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L123 EN**: Comment documents: `Make the FirstSU also dependent on the dependencies of the SecondSU to`.
  **L123 CN**: 注释说明：`Make the FirstSU also dependent on the dependencies of the SecondSU to`。
- **L124 EN**: Comment documents: `prevent them from being scheduled between the FirstSU and the SecondSU.`.
  **L124 CN**: 注释说明：`prevent them from being scheduled between the FirstSU and the SecondSU.`。
- **L125 EN**: Begins a conditional branch.
  **L125 CN**: 开始一个条件分支。
- **L126 EN**: Starts a loop over a sequence or range.
  **L126 CN**: 开始遍历序列或范围的循环。
- **L127 EN**: Assigns or initializes `SUnit *SU`.
  **L127 CN**: 对 `SUnit *SU` 进行赋值或初始化。
- **L128 EN**: Begins a conditional branch.
  **L128 CN**: 开始一个条件分支。
- **L129 EN**: Skips to the next loop iteration.
  **L129 CN**: 跳到下一次循环迭代。
- **L130 EN**: Emits debug-only tracing logic.
  **L130 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L131 EN**: Executes statement `DAG.dumpNodeName(FirstSU); dbgs() << '\n';);`.
  **L131 CN**: 执行语句 `DAG.dumpNodeName(FirstSU); dbgs() << '\n';);`。
- **L132 EN**: Executes statement `DAG.addEdge(&FirstSU, SDep(SU, SDep::Artificial));`.
  **L132 CN**: 执行语句 `DAG.addEdge(&FirstSU, SDep(SU, SDep::Artificial));`。
- **L133 EN**: Closes the current scope.
  **L133 CN**: 关闭当前作用域。
- **L134 EN**: Comment documents: `ExitSU comes last by design, which acts like an implicit dependency`.
  **L134 CN**: 注释说明：`ExitSU comes last by design, which acts like an implicit dependency`。
- **L135 EN**: Comment documents: `between ExitSU and any bottom root in the graph. We should transfer`.
  **L135 CN**: 注释说明：`between ExitSU and any bottom root in the graph. We should transfer`。
- **L136 EN**: Comment documents: `this to FirstSU as well.`.
  **L136 CN**: 注释说明：`this to FirstSU as well.`。
- **L137 EN**: Begins a conditional branch.
  **L137 CN**: 开始一个条件分支。
- **L138 EN**: Starts a loop over a sequence or range.
  **L138 CN**: 开始遍历序列或范围的循环。
- **L139 EN**: Begins a conditional branch.
  **L139 CN**: 开始一个条件分支。
- **L140 EN**: Executes statement `DAG.addEdge(&FirstSU, SDep(&SU, SDep::Artificial));`.
  **L140 CN**: 执行语句 `DAG.addEdge(&FirstSU, SDep(&SU, SDep::Artificial));`。

### Lines 141-160

````cpp
      }
    }
  }

  ++NumFused;
  return true;
}

namespace {

/// Post-process the DAG to create cluster edges between instrs that may
/// be fused by the processor into a single operation.
class MacroFusion : public ScheduleDAGMutation {
  std::vector<MacroFusionPredTy> Predicates;
  bool FuseBlock;
  bool scheduleAdjacentImpl(ScheduleDAGInstrs &DAG, SUnit &AnchorSU);

public:
  MacroFusion(ArrayRef<MacroFusionPredTy> Predicates, bool FuseBlock)
      : Predicates(Predicates.begin(), Predicates.end()), FuseBlock(FuseBlock) {
````
- **L141 EN**: Closes the current scope.
  **L141 CN**: 关闭当前作用域。
- **L142 EN**: Closes the current scope.
  **L142 CN**: 关闭当前作用域。
- **L143 EN**: Closes the current scope.
  **L143 CN**: 关闭当前作用域。
- **L144 EN**: Separates nearby statements for readability.
  **L144 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L145 EN**: Executes statement `++NumFused;`.
  **L145 CN**: 执行语句 `++NumFused;`。
- **L146 EN**: Returns `true` to the caller.
  **L146 CN**: 向调用者返回 `true`。
- **L147 EN**: Closes the current scope.
  **L147 CN**: 关闭当前作用域。
- **L148 EN**: Separates nearby statements for readability.
  **L148 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L149 EN**: Opens namespace ``.
  **L149 CN**: 打开命名空间 ``。
- **L150 EN**: Separates nearby statements for readability.
  **L150 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L151 EN**: Comment documents: `Post-process the DAG to create cluster edges between instrs that may`.
  **L151 CN**: 注释说明：`Post-process the DAG to create cluster edges between instrs that may`。
- **L152 EN**: Comment documents: `be fused by the processor into a single operation.`.
  **L152 CN**: 注释说明：`be fused by the processor into a single operation.`。
- **L153 EN**: Starts the declaration of class `MacroFusion`.
  **L153 CN**: 开始声明 class `MacroFusion`。
- **L154 EN**: Executes statement `std::vector<MacroFusionPredTy> Predicates;`.
  **L154 CN**: 执行语句 `std::vector<MacroFusionPredTy> Predicates;`。
- **L155 EN**: Executes statement `bool FuseBlock;`.
  **L155 CN**: 执行语句 `bool FuseBlock;`。
- **L156 EN**: Declares function or method `scheduleAdjacentImpl`.
  **L156 CN**: 声明函数或方法 `scheduleAdjacentImpl`。
- **L157 EN**: Separates nearby statements for readability.
  **L157 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L158 EN**: Continues logic with `public:`.
  **L158 CN**: 继续处理逻辑：`public:`。
- **L159 EN**: Continues logic with `MacroFusion(ArrayRef<MacroFusionPredTy> Predicates, bool FuseBlock)`.
  **L159 CN**: 继续处理逻辑：`MacroFusion(ArrayRef<MacroFusionPredTy> Predicates, bool FuseBlock)`。
- **L160 EN**: Begins the definition of `Predicates`.
  **L160 CN**: 开始定义 `Predicates`。

### Lines 161-180

````cpp
  }

  void apply(ScheduleDAGInstrs *DAGInstrs) override;

  bool shouldScheduleAdjacent(const TargetInstrInfo &TII,
                              const TargetSubtargetInfo &STI,
                              const MachineInstr *FirstMI,
                              const MachineInstr &SecondMI);
};

} // end anonymous namespace

bool MacroFusion::shouldScheduleAdjacent(const TargetInstrInfo &TII,
                                         const TargetSubtargetInfo &STI,
                                         const MachineInstr *FirstMI,
                                         const MachineInstr &SecondMI) {
  return llvm::any_of(Predicates, [&](MacroFusionPredTy Predicate) {
    return Predicate(TII, STI, FirstMI, SecondMI);
  });
}
````
- **L161 EN**: Closes the current scope.
  **L161 CN**: 关闭当前作用域。
- **L162 EN**: Separates nearby statements for readability.
  **L162 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L163 EN**: Declares function or method `apply`.
  **L163 CN**: 声明函数或方法 `apply`。
- **L164 EN**: Separates nearby statements for readability.
  **L164 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L165 EN**: Provides part of the signature for `shouldScheduleAdjacent`.
  **L165 CN**: 给出 `shouldScheduleAdjacent` 的一部分签名。
- **L166 EN**: Continues logic with `const TargetSubtargetInfo &STI,`.
  **L166 CN**: 继续处理逻辑：`const TargetSubtargetInfo &STI,`。
- **L167 EN**: Continues logic with `const MachineInstr *FirstMI,`.
  **L167 CN**: 继续处理逻辑：`const MachineInstr *FirstMI,`。
- **L168 EN**: Executes statement `const MachineInstr &SecondMI);`.
  **L168 CN**: 执行语句 `const MachineInstr &SecondMI);`。
- **L169 EN**: Closes the current scope.
  **L169 CN**: 关闭当前作用域。
- **L170 EN**: Separates nearby statements for readability.
  **L170 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L171 EN**: Continues logic with `} // end anonymous namespace`.
  **L171 CN**: 继续处理逻辑：`} // end anonymous namespace`。
- **L172 EN**: Separates nearby statements for readability.
  **L172 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L173 EN**: Provides part of the signature for `shouldScheduleAdjacent`.
  **L173 CN**: 给出 `shouldScheduleAdjacent` 的一部分签名。
- **L174 EN**: Continues logic with `const TargetSubtargetInfo &STI,`.
  **L174 CN**: 继续处理逻辑：`const TargetSubtargetInfo &STI,`。
- **L175 EN**: Continues logic with `const MachineInstr *FirstMI,`.
  **L175 CN**: 继续处理逻辑：`const MachineInstr *FirstMI,`。
- **L176 EN**: Starts block `const MachineInstr &SecondMI)`.
  **L176 CN**: 开始代码块 `const MachineInstr &SecondMI)`。
- **L177 EN**: Returns `llvm::any_of(Predicates, [&](MacroFusionPredTy Predicate) {` to the caller.
  **L177 CN**: 向调用者返回 `llvm::any_of(Predicates, [&](MacroFusionPredTy Predicate) {`。
- **L178 EN**: Returns `Predicate(TII, STI, FirstMI, SecondMI)` to the caller.
  **L178 CN**: 向调用者返回 `Predicate(TII, STI, FirstMI, SecondMI)`。
- **L179 EN**: Executes statement `});`.
  **L179 CN**: 执行语句 `});`。
- **L180 EN**: Closes the current scope.
  **L180 CN**: 关闭当前作用域。

### Lines 181-200

````cpp

void MacroFusion::apply(ScheduleDAGInstrs *DAG) {
  if (FuseBlock)
    // For each of the SUnits in the scheduling block, try to fuse the instr in
    // it with one in its predecessors.
    for (SUnit &ISU : DAG->SUnits)
        scheduleAdjacentImpl(*DAG, ISU);

  if (DAG->ExitSU.getInstr())
    // Try to fuse the instr in the ExitSU with one in its predecessors.
    scheduleAdjacentImpl(*DAG, DAG->ExitSU);
}

/// Implement the fusion of instr pairs in the scheduling DAG,
/// anchored at the instr in AnchorSU..
bool MacroFusion::scheduleAdjacentImpl(ScheduleDAGInstrs &DAG, SUnit &AnchorSU) {
  const MachineInstr &AnchorMI = *AnchorSU.getInstr();
  const TargetInstrInfo &TII = *DAG.TII;
  const TargetSubtargetInfo &ST = DAG.MF.getSubtarget();

````
- **L181 EN**: Separates nearby statements for readability.
  **L181 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L182 EN**: Begins the definition of `apply`.
  **L182 CN**: 开始定义 `apply`。
- **L183 EN**: Begins a conditional branch.
  **L183 CN**: 开始一个条件分支。
- **L184 EN**: Comment documents: `For each of the SUnits in the scheduling block, try to fuse the instr in`.
  **L184 CN**: 注释说明：`For each of the SUnits in the scheduling block, try to fuse the instr in`。
- **L185 EN**: Comment documents: `it with one in its predecessors.`.
  **L185 CN**: 注释说明：`it with one in its predecessors.`。
- **L186 EN**: Starts a loop over a sequence or range.
  **L186 CN**: 开始遍历序列或范围的循环。
- **L187 EN**: Executes statement `scheduleAdjacentImpl(*DAG, ISU);`.
  **L187 CN**: 执行语句 `scheduleAdjacentImpl(*DAG, ISU);`。
- **L188 EN**: Separates nearby statements for readability.
  **L188 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L189 EN**: Begins a conditional branch.
  **L189 CN**: 开始一个条件分支。
- **L190 EN**: Comment documents: `Try to fuse the instr in the ExitSU with one in its predecessors.`.
  **L190 CN**: 注释说明：`Try to fuse the instr in the ExitSU with one in its predecessors.`。
- **L191 EN**: Executes statement `scheduleAdjacentImpl(*DAG, DAG->ExitSU);`.
  **L191 CN**: 执行语句 `scheduleAdjacentImpl(*DAG, DAG->ExitSU);`。
- **L192 EN**: Closes the current scope.
  **L192 CN**: 关闭当前作用域。
- **L193 EN**: Separates nearby statements for readability.
  **L193 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L194 EN**: Comment documents: `Implement the fusion of instr pairs in the scheduling DAG,`.
  **L194 CN**: 注释说明：`Implement the fusion of instr pairs in the scheduling DAG,`。
- **L195 EN**: Comment documents: `anchored at the instr in AnchorSU..`.
  **L195 CN**: 注释说明：`anchored at the instr in AnchorSU..`。
- **L196 EN**: Begins the definition of `scheduleAdjacentImpl`.
  **L196 CN**: 开始定义 `scheduleAdjacentImpl`。
- **L197 EN**: Assigns or initializes `const MachineInstr &AnchorMI`.
  **L197 CN**: 对 `const MachineInstr &AnchorMI` 进行赋值或初始化。
- **L198 EN**: Assigns or initializes `const TargetInstrInfo &TII`.
  **L198 CN**: 对 `const TargetInstrInfo &TII` 进行赋值或初始化。
- **L199 EN**: Assigns or initializes `const TargetSubtargetInfo &ST`.
  **L199 CN**: 对 `const TargetSubtargetInfo &ST` 进行赋值或初始化。
- **L200 EN**: Separates nearby statements for readability.
  **L200 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 201-220

````cpp
  // Check if the anchor instr may be fused.
  if (!shouldScheduleAdjacent(TII, ST, nullptr, AnchorMI))
    return false;

  // Explorer for fusion candidates among the dependencies of the anchor instr.
  for (SDep &Dep : AnchorSU.Preds) {
    // Ignore dependencies other than data or strong ordering.
    if (Dep.isWeak() || isHazard(Dep))
      continue;

    SUnit &DepSU = *Dep.getSUnit();
    if (DepSU.isBoundaryNode())
      continue;

    // Only chain two instructions together at most.
    const MachineInstr *DepMI = DepSU.getInstr();
    if (!hasLessThanNumFused(DepSU, 2) ||
        !shouldScheduleAdjacent(TII, ST, DepMI, AnchorMI))
      continue;

````
- **L201 EN**: Comment documents: `Check if the anchor instr may be fused.`.
  **L201 CN**: 注释说明：`Check if the anchor instr may be fused.`。
- **L202 EN**: Begins a conditional branch.
  **L202 CN**: 开始一个条件分支。
- **L203 EN**: Returns `false` to the caller.
  **L203 CN**: 向调用者返回 `false`。
- **L204 EN**: Separates nearby statements for readability.
  **L204 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L205 EN**: Comment documents: `Explorer for fusion candidates among the dependencies of the anchor inst…`.
  **L205 CN**: 注释说明：`Explorer for fusion candidates among the dependencies of the anchor inst…`。
- **L206 EN**: Starts a loop over a sequence or range.
  **L206 CN**: 开始遍历序列或范围的循环。
- **L207 EN**: Comment documents: `Ignore dependencies other than data or strong ordering.`.
  **L207 CN**: 注释说明：`Ignore dependencies other than data or strong ordering.`。
- **L208 EN**: Begins a conditional branch.
  **L208 CN**: 开始一个条件分支。
- **L209 EN**: Skips to the next loop iteration.
  **L209 CN**: 跳到下一次循环迭代。
- **L210 EN**: Separates nearby statements for readability.
  **L210 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L211 EN**: Assigns or initializes `SUnit &DepSU`.
  **L211 CN**: 对 `SUnit &DepSU` 进行赋值或初始化。
- **L212 EN**: Begins a conditional branch.
  **L212 CN**: 开始一个条件分支。
- **L213 EN**: Skips to the next loop iteration.
  **L213 CN**: 跳到下一次循环迭代。
- **L214 EN**: Separates nearby statements for readability.
  **L214 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L215 EN**: Comment documents: `Only chain two instructions together at most.`.
  **L215 CN**: 注释说明：`Only chain two instructions together at most.`。
- **L216 EN**: Assigns or initializes `const MachineInstr *DepMI`.
  **L216 CN**: 对 `const MachineInstr *DepMI` 进行赋值或初始化。
- **L217 EN**: Begins a conditional branch.
  **L217 CN**: 开始一个条件分支。
- **L218 EN**: Continues logic with `!shouldScheduleAdjacent(TII, ST, DepMI, AnchorMI))`.
  **L218 CN**: 继续处理逻辑：`!shouldScheduleAdjacent(TII, ST, DepMI, AnchorMI))`。
- **L219 EN**: Skips to the next loop iteration.
  **L219 CN**: 跳到下一次循环迭代。
- **L220 EN**: Separates nearby statements for readability.
  **L220 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 221-234

````cpp
    if (fuseInstructionPair(DAG, DepSU, AnchorSU))
      return true;
  }

  return false;
}

std::unique_ptr<ScheduleDAGMutation>
llvm::createMacroFusionDAGMutation(ArrayRef<MacroFusionPredTy> Predicates,
                                   bool BranchOnly) {
  if (EnableMacroFusion)
    return std::make_unique<MacroFusion>(Predicates, !BranchOnly);
  return nullptr;
}
````
- **L221 EN**: Begins a conditional branch.
  **L221 CN**: 开始一个条件分支。
- **L222 EN**: Returns `true` to the caller.
  **L222 CN**: 向调用者返回 `true`。
- **L223 EN**: Closes the current scope.
  **L223 CN**: 关闭当前作用域。
- **L224 EN**: Separates nearby statements for readability.
  **L224 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L225 EN**: Returns `false` to the caller.
  **L225 CN**: 向调用者返回 `false`。
- **L226 EN**: Closes the current scope.
  **L226 CN**: 关闭当前作用域。
- **L227 EN**: Separates nearby statements for readability.
  **L227 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L228 EN**: Continues logic with `std::unique_ptr<ScheduleDAGMutation>`.
  **L228 CN**: 继续处理逻辑：`std::unique_ptr<ScheduleDAGMutation>`。
- **L229 EN**: Provides part of the signature for `createMacroFusionDAGMutation`.
  **L229 CN**: 给出 `createMacroFusionDAGMutation` 的一部分签名。
- **L230 EN**: Starts block `bool BranchOnly)`.
  **L230 CN**: 开始代码块 `bool BranchOnly)`。
- **L231 EN**: Begins a conditional branch.
  **L231 CN**: 开始一个条件分支。
- **L232 EN**: Returns `std::make_unique<MacroFusion>(Predicates, !BranchOnly)` to the caller.
  **L232 CN**: 向调用者返回 `std::make_unique<MacroFusion>(Predicates, !BranchOnly)`。
- **L233 EN**: Returns `nullptr` to the caller.
  **L233 CN**: 向调用者返回 `nullptr`。
- **L234 EN**: Closes the current scope.
  **L234 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Instruction scheduling** / **指令调度**
- **Machine instruction manipulation** / **机器指令操作**
- **Control-flow updates** / **控制流更新**
- **Hazard recognition** / **冒险识别**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/MacroFusion.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/ScheduleDAG.h`, `llvm/CodeGen/ScheduleDAGInstrs.h`, `llvm/CodeGen/ScheduleDAGMutation.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
