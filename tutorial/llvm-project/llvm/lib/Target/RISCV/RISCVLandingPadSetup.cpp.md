# RISCVLandingPadSetup.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVLandingPadSetup.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements landing-pad setup and exception-state materialization for RISC-V. / 实现RISC-V 的 landing pad 设置与异常状态物化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```cpp
//===------------ RISCVLandingPadSetup.cpp ---------------------------------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a RISC-V pass to setup landing pad labels for indirect jumps.
// Currently this pass only supports fixed labels.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-20: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVInstrInfo.h"
#include "RISCVMachineFunctionInfo.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 21-29: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-lpad-setup"
#define PASS_NAME "RISC-V Landing Pad Setup"

extern cl::opt<uint32_t> PreferredLandingPadLabel;

namespace {
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 30-37: Type declaration for RISCVLandingPadSetup / RISCVLandingPadSetup 的类型声明
```cpp
class RISCVLandingPadSetup : public MachineFunctionPass {
public:
  static char ID;

  RISCVLandingPadSetup() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &F) override;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 38-45: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
  StringRef getPassName() const override { return PASS_NAME; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 46-54: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // end anonymous namespace

bool RISCVLandingPadSetup::runOnMachineFunction(MachineFunction &MF) {
  const auto &STI = MF.getSubtarget<RISCVSubtarget>();
  const RISCVInstrInfo &TII = *STI.getInstrInfo();

  if (!MF.getInfo<RISCVMachineFunctionInfo>()->hasCFProtectionBranch())
    return false;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 55-62: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  uint32_t Label = 0;
  if (PreferredLandingPadLabel.getNumOccurrences() > 0) {
    if (!isUInt<20>(PreferredLandingPadLabel))
      report_fatal_error("riscv-landing-pad-label=<val>, <val> needs to fit in "
                         "unsigned 20-bits");
    Label = PreferredLandingPadLabel;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 63-78: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Zicfilp does not check X7 if landing pad label is zero.
  if (Label == 0)
    return false;

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF)
    for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
      if (MI.getOpcode() != RISCV::PseudoBRIND &&
          MI.getOpcode() != RISCV::PseudoCALLIndirect &&
          MI.getOpcode() != RISCV::PseudoTAILIndirect)
        continue;
      BuildMI(MBB, MI, MI.getDebugLoc(), TII.get(RISCV::LUI), RISCV::X7)
          .addImm(Label);
      MachineInstrBuilder(MF, &MI).addUse(RISCV::X7, RegState::ImplicitKill);
      Changed = true;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 79-86: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  return Changed;
}

INITIALIZE_PASS(RISCVLandingPadSetup, DEBUG_TYPE, PASS_NAME, false, false)

char RISCVLandingPadSetup::ID = 0;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 87-89: Function implementation: llvm::createRISCVLandingPadSetupPass / 函数实现：llvm::createRISCVLandingPadSetupPass
```cpp
FunctionPass *llvm::createRISCVLandingPadSetupPass() {
  return new RISCVLandingPadSetup();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMachineFunctionInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineInstrBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
