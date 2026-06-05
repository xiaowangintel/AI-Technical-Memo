# RISCVLateBranchOpt.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVLateBranchOpt.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements late branch optimization and branch-layout cleanup for RISC-V. / 实现RISC-V 的后期分支优化与分支布局清理。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVLateBranchOpt.cpp - Late Stage Branch Optimization -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// This file provides RISC-V specific target optimizations, currently it's
/// limited to convert conditional branches into unconditional branches when
/// the condition can be statically evaluated.
///
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 14-21: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVInstrInfo.h"
#include "RISCVSubtarget.h"

using namespace llvm;

#define RISCV_LATE_BRANCH_OPT_NAME "RISC-V Late Branch Optimisation Pass"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 22-30: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
namespace {

struct RISCVLateBranchOpt : public MachineFunctionPass {
  static char ID;

  RISCVLateBranchOpt() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return RISCV_LATE_BRANCH_OPT_NAME; }
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 31-39: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &Fn) override;

private:
  bool runOnBasicBlock(MachineBasicBlock &MBB) const;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 40-47: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  const RISCVInstrInfo *RII = nullptr;
};
} // namespace

char RISCVLateBranchOpt::ID = 0;
INITIALIZE_PASS(RISCVLateBranchOpt, "riscv-late-branch-opt",
                RISCV_LATE_BRANCH_OPT_NAME, false, false)
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 48-56: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
bool RISCVLateBranchOpt::runOnBasicBlock(MachineBasicBlock &MBB) const {
  MachineBasicBlock *TBB, *FBB;
  SmallVector<MachineOperand, 4> Cond;
  if (RII->analyzeBranch(MBB, TBB, FBB, Cond, /*AllowModify=*/false))
    return false;

  if (!TBB || Cond.size() != 3)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 57-68: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  RISCVCC::CondCode CC = RISCVInstrInfo::getCondFromBranchOpc(Cond[0].getImm());
  assert(CC != RISCVCC::COND_INVALID);

  MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();

  // Try and convert a conditional branch that can be evaluated statically
  // into an unconditional branch.
  int64_t C0, C1;
  if (!RISCVInstrInfo::isFromLoadImm(MRI, Cond[1], C0) ||
      !RISCVInstrInfo::isFromLoadImm(MRI, Cond[2], C1))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 69-79: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MachineBasicBlock *Folded =
      RISCVInstrInfo::evaluateCondBranch(CC, C0, C1) ? TBB : FBB;

  // At this point, its legal to optimize.
  RII->removeBranch(MBB);

  // Only need to insert a branch if we're not falling through.
  if (Folded) {
    DebugLoc DL = MBB.findBranchDebugLoc();
    RII->insertBranch(MBB, Folded, nullptr, {}, DL);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 80-92: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Update the successors. Remove them all and add back the correct one.
  while (!MBB.succ_empty())
    MBB.removeSuccessor(MBB.succ_end() - 1);

  // If it's a fallthrough, we need to figure out where MBB is going.
  if (!Folded) {
    MachineFunction::iterator Fallthrough = ++MBB.getIterator();
    if (Fallthrough != MBB.getParent()->end())
      MBB.addSuccessor(&*Fallthrough);
  } else
    MBB.addSuccessor(Folded);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 93-102: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return true;
}

bool RISCVLateBranchOpt::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;

  auto &ST = Fn.getSubtarget<RISCVSubtarget>();
  RII = ST.getInstrInfo();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 103-111: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  bool Changed = false;
  for (MachineBasicBlock &MBB : Fn)
    Changed |= runOnBasicBlock(MBB);
  return Changed;
}

FunctionPass *llvm::createRISCVLateBranchOptPass() {
  return new RISCVLateBranchOpt();
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
