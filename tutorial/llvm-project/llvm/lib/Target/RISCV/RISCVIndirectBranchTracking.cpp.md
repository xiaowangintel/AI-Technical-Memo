# RISCVIndirectBranchTracking.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVIndirectBranchTracking.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V indirect-branch tracking and related control-flow hardening support. / 实现RISC-V 间接分支跟踪及相关控制流加固支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Commentary and design intent / 注释与设计意图
```cpp
//===------ RISCVIndirectBranchTracking.cpp - Enables lpad mechanism ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The pass adds LPAD (AUIPC with rd = X0) machine instructions at the
// beginning of each basic block or function that is referenced by an indirect
// jump/call instruction.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 14-24: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVInstrInfo.h"
#include "RISCVMachineFunctionInfo.h"
#include "RISCVSubtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 25-39: Header guard and interface framing / 头文件保护与接口框架
```cpp
#define DEBUG_TYPE "riscv-indirect-branch-tracking"
#define PASS_NAME "RISC-V Indirect Branch Tracking"

using namespace llvm;

cl::opt<uint32_t> PreferredLandingPadLabel(
    "riscv-landing-pad-label", cl::ReallyHidden,
    cl::desc("Use preferred fixed label for all labels"));

namespace {
class RISCVIndirectBranchTracking : public MachineFunctionPass {
public:
  static char ID;
  RISCVIndirectBranchTracking() : MachineFunctionPass(ID) {}
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 40-49: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  StringRef getPassName() const override { return PASS_NAME; }

  bool runOnMachineFunction(MachineFunction &MF) override;

private:
  const Align LpadAlign = Align(4);
};

} // end anonymous namespace
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 50-66: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
INITIALIZE_PASS(RISCVIndirectBranchTracking, DEBUG_TYPE, PASS_NAME, false,
                false)

char RISCVIndirectBranchTracking::ID = 0;

FunctionPass *llvm::createRISCVIndirectBranchTrackingPass() {
  return new RISCVIndirectBranchTracking();
}

static void
emitLpad(MachineBasicBlock &MBB, const RISCVInstrInfo *TII, uint32_t Label,
         MachineBasicBlock::iterator I = MachineBasicBlock::iterator{}) {
  if (!I.isValid())
    I = MBB.begin();
  BuildMI(MBB, I, MBB.findDebugLoc(I), TII->get(RISCV::AUIPC), RISCV::X0)
      .addImm(Label);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 67-76: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static bool isCallReturnTwice(const MachineOperand &MOp) {
  if (!MOp.isGlobal())
    return false;
  auto *CalleeFn = dyn_cast<Function>(MOp.getGlobal());
  if (!CalleeFn)
    return false;
  AttributeList Attrs = CalleeFn->getAttributes();
  return Attrs.hasFnAttr(Attribute::ReturnsTwice);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 77-91: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVIndirectBranchTracking::runOnMachineFunction(MachineFunction &MF) {
  const auto &Subtarget = MF.getSubtarget<RISCVSubtarget>();
  const RISCVInstrInfo *TII = Subtarget.getInstrInfo();

  if (!MF.getInfo<RISCVMachineFunctionInfo>()->hasCFProtectionBranch())
    return false;

  uint32_t FixedLabel = 0;
  if (PreferredLandingPadLabel.getNumOccurrences() > 0) {
    if (!isUInt<20>(PreferredLandingPadLabel))
      report_fatal_error("riscv-landing-pad-label=<val>, <val> needs to fit in "
                         "unsigned 20-bits");
    FixedLabel = PreferredLandingPadLabel;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 92-106: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF) {
    if (&MBB == &MF.front()) {
      Function &F = MF.getFunction();
      // When trap is taken, landing pad is not needed.
      if (F.hasFnAttribute("interrupt"))
        continue;

      if (F.hasAddressTaken() || !F.hasLocalLinkage()) {
        emitLpad(MBB, TII, FixedLabel);
        if (MF.getAlignment() < LpadAlign)
          MF.setAlignment(LpadAlign);
        Changed = true;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 107-116: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      continue;
    }

    if (MBB.hasAddressTaken()) {
      emitLpad(MBB, TII, FixedLabel);
      if (MBB.getAlignment() < LpadAlign)
        MBB.setAlignment(LpadAlign);
      Changed = true;
    }
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 117-127: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Check for calls to functions with ReturnsTwice attribute and insert
  // LPAD after such calls
  for (MachineBasicBlock &MBB : MF) {
    for (MachineBasicBlock::iterator I = MBB.begin(); I != MBB.end(); ++I) {
      if (I->isCall() && I->getNumOperands() > 0 &&
          isCallReturnTwice(I->getOperand(0))) {
        auto NextI = std::next(I);
        emitLpad(MBB, TII, FixedLabel, NextI);
        Changed = true;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 128-132: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    }
  }

  return Changed;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMachineFunctionInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/Statistic.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineBasicBlock.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineInstrBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineModuleInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
