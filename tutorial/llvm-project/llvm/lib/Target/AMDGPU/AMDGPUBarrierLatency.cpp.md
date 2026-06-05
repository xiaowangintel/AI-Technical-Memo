# AMDGPUBarrierLatency.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUBarrierLatency.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUBarrierLatency for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUBarrierLatency 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: File banner, license, and overview
```cpp
//===--- AMDGPUBarrierLatency.cpp - AMDGPU Barrier Latency ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains a DAG scheduling mutation to add latency to:
///       1. Barrier edges between ATOMIC_FENCE instructions and preceding
///          memory accesses potentially affected by the fence.
///          This encourages the scheduling of more instructions before
///          ATOMIC_FENCE instructions.  ATOMIC_FENCE instructions may
///          introduce wait counting or indicate an impending S_BARRIER
///          wait.  Having more instructions in-flight across these
///          constructs improves latency hiding.
///       2. Barrier edges from S_BARRIER_SIGNAL to S_BARRIER_WAIT.
///          This encourages independent work to be scheduled between
///          signal and wait, hiding barrier synchronization latency.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 23-43: Header dependencies and setup
```cpp
#include "AMDGPUBarrierLatency.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIInstrInfo.h"
#include "llvm/CodeGen/ScheduleDAGInstrs.h"
#include "llvm/Support/CommandLine.h"

using namespace llvm;

static cl::opt<unsigned> BarrierSignalWaitLatencyOpt(
    "amdgpu-barrier-signal-wait-latency",
    cl::desc("Synthetic latency between S_BARRIER_SIGNAL and S_BARRIER_WAIT "
             "to encourage scheduling independent work between them"),
    cl::init(16), cl::Hidden);

namespace {

class BarrierLatency : public ScheduleDAGMutation {
private:
  SmallSet<SyncScope::ID, 4> IgnoredScopes;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `BarrierLatency`, `cl::desc`, `cl::init`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`BarrierLatency`, `cl::desc`, `cl::init`。

### Lines 44-60: Defines BarrierLatency
```cpp
public:
  BarrierLatency(MachineFunction *MF) {
    LLVMContext &Context = MF->getFunction().getContext();
    IgnoredScopes.insert(SyncScope::SingleThread);
    IgnoredScopes.insert(Context.getOrInsertSyncScopeID("wavefront"));
    IgnoredScopes.insert(Context.getOrInsertSyncScopeID("wavefront-one-as"));
    IgnoredScopes.insert(Context.getOrInsertSyncScopeID("singlethread-one-as"));

    const GCNSubtarget &ST = MF->getSubtarget<GCNSubtarget>();
    if (!ST.requiresWaitOnWorkgroupReleaseFence()) {
      // Prior to GFX10 workgroup scope does not normally require waitcnts
      IgnoredScopes.insert(Context.getOrInsertSyncScopeID("workgroup"));
    }
  }
  void apply(ScheduleDAGInstrs *DAG) override;
};

```
**EN:** This section contains concrete logic for BarrierLatency. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 BarrierLatency 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 61-84: Defines addLatencyToEdge
```cpp
void addLatencyToEdge(SDep &PredDep, SUnit &SU, unsigned Latency) {
  SUnit *PredSU = PredDep.getSUnit();
  SDep ForwardD = PredDep;
  ForwardD.setSUnit(&SU);
  for (SDep &SuccDep : PredSU->Succs) {
    if (SuccDep == ForwardD) {
      SuccDep.setLatency(SuccDep.getLatency() + Latency);
      break;
    }
  }
  PredDep.setLatency(PredDep.getLatency() + Latency);
  PredSU->setDepthDirty();
  SU.setDepthDirty();
}

void BarrierLatency::apply(ScheduleDAGInstrs *DAG) {
  const SIInstrInfo *TII = static_cast<const SIInstrInfo *>(DAG->TII);
  constexpr unsigned FenceLatency = 2000;
  const unsigned BarrierSignalWaitLatency = BarrierSignalWaitLatencyOpt;

  for (SUnit &SU : DAG->SUnits) {
    const MachineInstr *MI = SU.getInstr();
    unsigned Op = MI->getOpcode();

```
**EN:** This section contains concrete logic for addLatencyToEdge. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `BarrierLatency::apply`.
**CN:** 本节包含与 addLatencyToEdge 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`BarrierLatency::apply`。

### Lines 85-108: Conditional logic and checks
```cpp
    if (Op == AMDGPU::ATOMIC_FENCE) {
      // Update latency on barrier edges of ATOMIC_FENCE.
      // Ignore scopes not expected to have any latency.
      SyncScope::ID SSID =
          static_cast<SyncScope::ID>(MI->getOperand(1).getImm());
      if (IgnoredScopes.contains(SSID))
        continue;

      for (SDep &PredDep : SU.Preds) {
        if (!PredDep.isBarrier())
          continue;
        SUnit *PredSU = PredDep.getSUnit();
        MachineInstr *MI = PredSU->getInstr();
        // Only consider memory loads
        if (!MI->mayLoad() || MI->mayStore())
          continue;
        addLatencyToEdge(PredDep, SU, FenceLatency);
      }
    } else if (Op == AMDGPU::S_BARRIER_WAIT) {
      for (SDep &PredDep : SU.Preds) {
        SUnit *PredSU = PredDep.getSUnit();
        const MachineInstr *PredMI = PredSU->getInstr();
        if (TII->isBarrierStart(PredMI->getOpcode())) {
          addLatencyToEdge(PredDep, SU, BarrierSignalWaitLatency);
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 109-120: Implements llvm::createAMDGPUBarrierLatencyDAGMutation
```cpp
        }
      }
    }
  }
}

} // end namespace

std::unique_ptr<ScheduleDAGMutation>
llvm::createAMDGPUBarrierLatencyDAGMutation(MachineFunction *MF) {
  return std::make_unique<BarrierLatency>(MF);
}
```
**EN:** This section contains concrete logic for llvm::createAMDGPUBarrierLatencyDAGMutation. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPUBarrierLatencyDAGMutation`.
**CN:** 本节包含与 llvm::createAMDGPUBarrierLatencyDAGMutation 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPUBarrierLatencyDAGMutation`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `BarrierLatency`, `cl::desc`, `cl::init`, `BarrierLatency::apply`, `llvm::createAMDGPUBarrierLatencyDAGMutation`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUBarrierLatency.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIInstrInfo.h"`
- `"llvm/CodeGen/ScheduleDAGInstrs.h"`
- `"llvm/Support/CommandLine.h"`
