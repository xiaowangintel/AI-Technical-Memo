# AMDGPUExportClustering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUExportClustering.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUExportClustering for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUExportClustering 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: File banner, includes, and setup
```cpp
//===--- AMDGPUExportClusting.cpp - AMDGPU Export Clustering  -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains a DAG scheduling mutation to cluster shader
///       exports.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUExportClustering.h"
#include "SIInstrInfo.h"
#include "llvm/CodeGen/ScheduleDAGInstrs.h"

using namespace llvm;

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 22-42: Declares class ExportClustering
```cpp
class ExportClustering : public ScheduleDAGMutation {
public:
  ExportClustering() = default;
  void apply(ScheduleDAGInstrs *DAG) override;
};

static bool isExport(const SUnit &SU) {
  return SIInstrInfo::isEXP(*SU.getInstr());
}

static bool isPositionExport(const SIInstrInfo *TII, SUnit *SU) {
  const MachineInstr *MI = SU->getInstr();
  unsigned Imm = TII->getNamedOperand(*MI, AMDGPU::OpName::tgt)->getImm();
  return Imm >= AMDGPU::Exp::ET_POS0 && Imm <= AMDGPU::Exp::ET_POS_LAST;
}

static void sortChain(const SIInstrInfo *TII, SmallVector<SUnit *, 8> &Chain,
                      unsigned PosCount) {
  if (!PosCount || PosCount == Chain.size())
    return;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `ExportClustering`, `SIInstrInfo::isEXP`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`ExportClustering`, `SIInstrInfo::isEXP`。

### Lines 43-65: Declares Copy
```cpp
  // Position exports should occur as soon as possible in the shader
  // for optimal performance.  This moves position exports before
  // other exports while preserving the order within different export
  // types (pos or other).
  SmallVector<SUnit *, 8> Copy(Chain);
  unsigned PosIdx = 0;
  unsigned OtherIdx = PosCount;
  for (SUnit *SU : Copy) {
    if (isPositionExport(TII, SU))
      Chain[PosIdx++] = SU;
    else
      Chain[OtherIdx++] = SU;
  }
}

static void buildCluster(ArrayRef<SUnit *> Exports, ScheduleDAGInstrs *DAG) {
  SUnit *ChainHead = Exports.front();

  // Now construct cluster from chain by adding new edges.
  for (unsigned Idx = 0, End = Exports.size() - 1; Idx < End; ++Idx) {
    SUnit *SUa = Exports[Idx];
    SUnit *SUb = Exports[Idx + 1];

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 66-83: Conditional logic and checks
```cpp
    // Copy all dependencies to the head of the chain to avoid any
    // computation being inserted into the chain.
    for (const SDep &Pred : SUb->Preds) {
      SUnit *PredSU = Pred.getSUnit();
      if (!isExport(*PredSU) && !Pred.isWeak())
        DAG->addEdge(ChainHead, SDep(PredSU, SDep::Artificial));
    }

    // New barrier edge ordering exports
    DAG->addEdge(SUb, SDep(SUa, SDep::Barrier));
    // Also add cluster edge
    DAG->addEdge(SUb, SDep(SUa, SDep::Cluster));
  }
}

static void removeExportDependencies(ScheduleDAGInstrs *DAG, SUnit &SU) {
  SmallVector<SDep, 2> ToAdd, ToRemove;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 84-106: Conditional logic and checks
```cpp
  for (const SDep &Pred : SU.Preds) {
    SUnit *PredSU = Pred.getSUnit();
    if (Pred.isBarrier() && isExport(*PredSU)) {
      ToRemove.push_back(Pred);
      if (isExport(SU))
        continue;

      // If we remove a barrier we need to copy dependencies
      // from the predecessor to maintain order.
      for (const SDep &ExportPred : PredSU->Preds) {
        SUnit *ExportPredSU = ExportPred.getSUnit();
        if (ExportPred.isBarrier() && !isExport(*ExportPredSU))
          ToAdd.push_back(SDep(ExportPredSU, SDep::Barrier));
      }
    }
  }

  for (SDep Pred : ToRemove)
    SU.removePred(Pred);
  for (SDep Pred : ToAdd)
    DAG->addEdge(&SU, Pred);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 107-126: Implements ExportClustering::apply
```cpp
void ExportClustering::apply(ScheduleDAGInstrs *DAG) {
  const SIInstrInfo *TII = static_cast<const SIInstrInfo *>(DAG->TII);

  SmallVector<SUnit *, 8> Chain;

  // Pass through DAG gathering a list of exports and removing barrier edges
  // creating dependencies on exports. Freeing exports of successor edges
  // allows more scheduling freedom, and nothing should be order dependent
  // on exports.  Edges will be added later to order the exports.
  unsigned PosCount = 0;
  for (SUnit &SU : DAG->SUnits) {
    if (!isExport(SU))
      continue;

    Chain.push_back(&SU);
    if (isPositionExport(TII, &SU))
      PosCount++;

    removeExportDependencies(DAG, SU);

```
**EN:** This section contains concrete logic for ExportClustering::apply. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ExportClustering::apply`.
**CN:** 本节包含与 ExportClustering::apply 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ExportClustering::apply`。

### Lines 127-144: Conditional logic and checks
```cpp
    SmallVector<SDep, 4> Succs(SU.Succs);
    for (SDep Succ : Succs)
      removeExportDependencies(DAG, *Succ.getSUnit());
  }

  // Apply clustering if there are multiple exports
  if (Chain.size() > 1) {
    sortChain(TII, Chain, PosCount);
    buildCluster(Chain, DAG);
  }
}

} // end namespace

std::unique_ptr<ScheduleDAGMutation>
llvm::createAMDGPUExportClusteringDAGMutation() {
  return std::make_unique<ExportClustering>();
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::createAMDGPUExportClusteringDAGMutation`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::createAMDGPUExportClusteringDAGMutation`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `ExportClustering`, `SIInstrInfo::isEXP`, `ExportClustering::apply`, `llvm::createAMDGPUExportClusteringDAGMutation`
- **Main themes / 核心主题**: instruction semantics / 指令语义; scheduling / 调度; LLVM pass integration / LLVM Pass 集成; SelectionDAG processing / SelectionDAG 处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUExportClustering.h"`
- `"SIInstrInfo.h"`
- `"llvm/CodeGen/ScheduleDAGInstrs.h"`
