# AMDGPUHazardLatency.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUHazardLatency.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUHazardLatency for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUHazardLatency 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File banner, license, and overview
```cpp
//===--- AMDGPUHazardLatency.cpp - AMDGPU Hazard Latency Adjustment -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains a DAG scheduling mutation to adjust the
///       latency of data edges between instructions which use registers
///       potentially subject to additional hazard waits not accounted
///       for in the normal scheduling model.
///       While the scheduling model is typically still accurate in these
///       scenarios, adjusting latency of relevant edges can improve wait
///       merging and reduce pipeline impact of any required waits.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 19-34: Header dependencies and setup
```cpp
#include "AMDGPUHazardLatency.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIInstrInfo.h"
#include "llvm/CodeGen/ScheduleDAGInstrs.h"

using namespace llvm;

namespace {

class HazardLatency : public ScheduleDAGMutation {
private:
  const GCNSubtarget &ST;
  const SIRegisterInfo &TRI;
  const MachineRegisterInfo &MRI;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `HazardLatency`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`HazardLatency`。

### Lines 35-48: Defines HazardLatency
```cpp
public:
  HazardLatency(MachineFunction *MF)
      : ST(MF->getSubtarget<GCNSubtarget>()), TRI(*ST.getRegisterInfo()),
        MRI(MF->getRegInfo()) {}
  void apply(ScheduleDAGInstrs *DAG) override;
};

void HazardLatency::apply(ScheduleDAGInstrs *DAG) {
  constexpr unsigned MaskLatencyBoost = 3;

  // Hazard only manifests in Wave64
  if (!ST.hasVALUMaskWriteHazard() || !ST.isWave64())
    return;

```
**EN:** This section contains concrete logic for HazardLatency. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `HazardLatency::apply`.
**CN:** 本节包含与 HazardLatency 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`HazardLatency::apply`。

### Lines 49-66: Conditional logic and checks
```cpp
  for (SUnit &SU : DAG->SUnits) {
    const MachineInstr *MI = SU.getInstr();
    if (!SIInstrInfo::isVALU(*MI))
      continue;
    if (MI->getOpcode() == AMDGPU::V_READLANE_B32 ||
        MI->getOpcode() == AMDGPU::V_READFIRSTLANE_B32)
      continue;
    for (SDep &SuccDep : SU.Succs) {
      if (SuccDep.isCtrl())
        continue;
      // Boost latency on VALU writes to SGPRs used by VALUs.
      // Reduce risk of premature VALU pipeline stall on associated reads.
      MachineInstr *DestMI = SuccDep.getSUnit()->getInstr();
      if (!SIInstrInfo::isVALU(*DestMI))
        continue;
      Register Reg = SuccDep.getReg();
      if (!TRI.isSGPRReg(MRI, Reg))
        continue;
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::isVALU`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::isVALU`。

### Lines 67-77: Implements llvm::createAMDGPUHazardLatencyDAGMutation
```cpp
      SuccDep.setLatency(SuccDep.getLatency() * MaskLatencyBoost);
    }
  }
}

} // end namespace

std::unique_ptr<ScheduleDAGMutation>
llvm::createAMDGPUHazardLatencyDAGMutation(MachineFunction *MF) {
  return std::make_unique<HazardLatency>(MF);
}
```
**EN:** This section contains concrete logic for llvm::createAMDGPUHazardLatencyDAGMutation. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPUHazardLatencyDAGMutation`.
**CN:** 本节包含与 llvm::createAMDGPUHazardLatencyDAGMutation 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPUHazardLatencyDAGMutation`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `HazardLatency`, `HazardLatency::apply`, `SIInstrInfo::isVALU`, `llvm::createAMDGPUHazardLatencyDAGMutation`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUHazardLatency.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIInstrInfo.h"`
- `"llvm/CodeGen/ScheduleDAGInstrs.h"`
