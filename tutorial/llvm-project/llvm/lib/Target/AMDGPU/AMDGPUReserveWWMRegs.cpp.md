# AMDGPUReserveWWMRegs.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUReserveWWMRegs.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUReserveWWMRegs for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUReserveWWMRegs 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUReserveWWMRegs.cpp - Add WWM Regs to reserved regs list -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass should be invoked at the end of wwm-regalloc pipeline.
/// It identifies the WWM regs allocated during this pipeline and add
/// them to the list of reserved registers so that they won't be available for
/// per-thread VGPR allocation in the subsequent regalloc pipeline.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUReserveWWMRegs.h"
#include "AMDGPU.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/VirtRegMap.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-47: Preprocessor guards and macros
```cpp
using namespace llvm;

#define DEBUG_TYPE "amdgpu-reserve-wwm-regs"

namespace {

class AMDGPUReserveWWMRegsLegacy : public MachineFunctionPass {
public:
  static char ID;

  AMDGPUReserveWWMRegsLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "AMDGPU Reserve WWM Registers";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPUReserveWWMRegsLegacy`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPUReserveWWMRegsLegacy`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 48-65: Registers LLVM passes
```cpp
class AMDGPUReserveWWMRegs {
public:
  bool run(MachineFunction &MF);
};

} // End anonymous namespace.

INITIALIZE_PASS(AMDGPUReserveWWMRegsLegacy, DEBUG_TYPE,
                "AMDGPU Reserve WWM Registers", false, false)

char AMDGPUReserveWWMRegsLegacy::ID = 0;

char &llvm::AMDGPUReserveWWMRegsLegacyID = AMDGPUReserveWWMRegsLegacy::ID;

bool AMDGPUReserveWWMRegsLegacy::runOnMachineFunction(MachineFunction &MF) {
  return AMDGPUReserveWWMRegs().run(MF);
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPUReserveWWMRegs`, `AMDGPUReserveWWMRegsLegacy::runOnMachineFunction`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPUReserveWWMRegs`, `AMDGPUReserveWWMRegsLegacy::runOnMachineFunction`。

### Lines 66-89: Implements AMDGPUReserveWWMRegsPass::run
```cpp
PreservedAnalyses
AMDGPUReserveWWMRegsPass::run(MachineFunction &MF,
                              MachineFunctionAnalysisManager &) {
  AMDGPUReserveWWMRegs().run(MF);
  // TODO: This should abandon RegisterClassInfo once it is turned into an
  // analysis.
  return PreservedAnalyses::all();
}

bool AMDGPUReserveWWMRegs::run(MachineFunction &MF) {
  SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      unsigned Opc = MI.getOpcode();
      if (Opc != AMDGPU::SI_SPILL_S32_TO_VGPR &&
          Opc != AMDGPU::SI_RESTORE_S32_FROM_VGPR)
        continue;

      Register Reg = Opc == AMDGPU::SI_SPILL_S32_TO_VGPR
                         ? MI.getOperand(0).getReg()
                         : MI.getOperand(1).getReg();

```
**EN:** This section contains concrete logic for AMDGPUReserveWWMRegsPass::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUReserveWWMRegsPass::run`, `PreservedAnalyses::all`, `AMDGPUReserveWWMRegs::run`.
**CN:** 本节包含与 AMDGPUReserveWWMRegsPass::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUReserveWWMRegsPass::run`, `PreservedAnalyses::all`, `AMDGPUReserveWWMRegs::run`。

### Lines 90-111: Declares getRegInfo
```cpp
      assert(Reg.isPhysical() &&
             "All WWM registers should have been allocated by now.");

      MFI->reserveWWMRegister(Reg);
      Changed |= true;
    }
  }

  // The renamable flag can't be set for reserved registers. Reset the flag for
  // MOs involving wwm-regs as they will be reserved during vgpr-regalloc
  // pipeline.
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  for (Register Reg : MFI->getWWMReservedRegs()) {
    for (MachineOperand &MO : MRI.reg_operands(Reg))
      MO.setIsRenamable(false);
  }

  // Now clear the NonWWMRegMask earlier set during wwm-regalloc.
  MFI->clearNonWWMRegAllocMask();

  return Changed;
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUReserveWWMRegsLegacy`, `AMDGPUReserveWWMRegs`, `MachineFunctionPass::getAnalysisUsage`, `AMDGPUReserveWWMRegsLegacy::runOnMachineFunction`, `AMDGPUReserveWWMRegsPass::run`, `PreservedAnalyses::all`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUReserveWWMRegs.h"`
- `"AMDGPU.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/VirtRegMap.h"`
