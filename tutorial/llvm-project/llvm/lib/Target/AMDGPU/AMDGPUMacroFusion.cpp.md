# AMDGPUMacroFusion.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMacroFusion.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMacroFusion for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUMacroFusion 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File banner, includes, and setup
```cpp
//===--- AMDGPUMacroFusion.cpp - AMDGPU Macro Fusion ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains the AMDGPU implementation of the DAG scheduling
///  mutation to pair instructions back to back.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUMacroFusion.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIInstrInfo.h"
#include "llvm/CodeGen/MacroFusion.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 19-31: Defines shouldScheduleAdjacent
```cpp
using namespace llvm;

namespace {

/// Check if the instr pair, FirstMI and SecondMI, should be fused
/// together. Given SecondMI, when FirstMI is unspecified, then check if
/// SecondMI may be part of a fused pair at all.
static bool shouldScheduleAdjacent(const TargetInstrInfo &TII_,
                                   const TargetSubtargetInfo &TSI,
                                   const MachineInstr *FirstMI,
                                   const MachineInstr &SecondMI) {
  const SIInstrInfo &TII = static_cast<const SIInstrInfo&>(TII_);

```
**EN:** This section contains concrete logic for shouldScheduleAdjacent. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 shouldScheduleAdjacent 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 32-42: Switch-based control flow
```cpp
  switch (SecondMI.getOpcode()) {
  case AMDGPU::V_ADDC_U32_e64:
  case AMDGPU::V_SUBB_U32_e64:
  case AMDGPU::V_SUBBREV_U32_e64:
  case AMDGPU::V_CNDMASK_B32_e64: {
    // Try to cluster defs of condition registers to their uses. This improves
    // the chance VCC will be available which will allow shrinking to VOP2
    // encodings.
    if (!FirstMI)
      return true;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 43-59: Result computation and returns
```cpp
    const MachineBasicBlock &MBB = *FirstMI->getParent();
    const MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
    const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
    const MachineOperand *Src2 = TII.getNamedOperand(SecondMI,
                                                     AMDGPU::OpName::src2);
    return FirstMI->definesRegister(Src2->getReg(), TRI);
  }
  default:
    return false;
  }

  return false;
}

} // end namespace


```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

### Lines 60-66: Defines createAMDGPUMacroFusionDAGMutation
```cpp
namespace llvm {

std::unique_ptr<ScheduleDAGMutation> createAMDGPUMacroFusionDAGMutation() {
  return createMacroFusionDAGMutation(shouldScheduleAdjacent);
}

} // end namespace llvm
```
**EN:** This section contains concrete logic for createAMDGPUMacroFusionDAGMutation. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 createAMDGPUMacroFusionDAGMutation 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMacroFusion.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIInstrInfo.h"`
- `"llvm/CodeGen/MacroFusion.h"`
