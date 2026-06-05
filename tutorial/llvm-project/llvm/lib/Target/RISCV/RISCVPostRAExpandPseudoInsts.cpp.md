# RISCVPostRAExpandPseudoInsts.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVPostRAExpandPseudoInsts.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements expansion of generic RISC-V pseudo instructions into concrete machine instructions. / 实现将通用 RISC-V 伪指令展开为具体机器指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVPostRAExpandPseudoInsts.cpp - Expand pseudo instrs ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that expands the pseudo instruction pseudolisimm32
// into target instructions. This pass should be run during the post-regalloc
// passes, before post RA scheduling.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 14-24: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVInstrInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"

using namespace llvm;

#define RISCV_POST_RA_EXPAND_PSEUDO_NAME                                       \
  "RISC-V post-regalloc pseudo instruction expansion pass"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 25-35: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
namespace {

class RISCVPostRAExpandPseudo : public MachineFunctionPass {
public:
  const RISCVInstrInfo *TII;
  static char ID;

  RISCVPostRAExpandPseudo() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 36-47: Function implementation: getPassName / 函数实现：getPassName
```cpp
  StringRef getPassName() const override {
    return RISCV_POST_RA_EXPAND_PSEUDO_NAME;
  }

private:
  bool expandMBB(MachineBasicBlock &MBB);
  bool expandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                MachineBasicBlock::iterator &NextMBBI);
  bool expandMovImm(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI);
  bool expandMovAddr(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI);
  bool expandMERGE(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI);
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 48-57: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

char RISCVPostRAExpandPseudo::ID = 0;

bool RISCVPostRAExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
  TII = static_cast<const RISCVInstrInfo *>(MF.getSubtarget().getInstrInfo());
  bool Modified = false;
  for (auto &MBB : MF)
    Modified |= expandMBB(MBB);
  return Modified;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 58-67: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVPostRAExpandPseudo::expandMBB(MachineBasicBlock &MBB) {
  bool Modified = false;

  MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  while (MBBI != E) {
    MachineBasicBlock::iterator NMBBI = std::next(MBBI);
    Modified |= expandMI(MBB, MBBI, NMBBI);
    MBBI = NMBBI;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 68-84: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return Modified;
}

bool RISCVPostRAExpandPseudo::expandMI(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator MBBI,
                                       MachineBasicBlock::iterator &NextMBBI) {
  switch (MBBI->getOpcode()) {
  case RISCV::PseudoMovImm:
    return expandMovImm(MBB, MBBI);
  case RISCV::PseudoMovAddr:
    return expandMovAddr(MBB, MBBI);
  case RISCV::PseudoMERGE:
    return expandMERGE(MBB, MBBI);
  default:
    return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 85-96: Function implementation: RISCVPostRAExpandPseudo::expandMovImm / 函数实现：RISCVPostRAExpandPseudo::expandMovImm
```cpp
}

bool RISCVPostRAExpandPseudo::expandMovImm(MachineBasicBlock &MBB,
                                           MachineBasicBlock::iterator MBBI) {
  DebugLoc DL = MBBI->getDebugLoc();

  int64_t Val = MBBI->getOperand(1).getImm();

  Register DstReg = MBBI->getOperand(0).getReg();
  bool DstIsDead = MBBI->getOperand(0).isDead();
  bool Renamable = MBBI->getOperand(0).isRenamable();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 97-107: Function implementation: RISCVPostRAExpandPseudo::expandMovAddr / 函数实现：RISCVPostRAExpandPseudo::expandMovAddr
```cpp
  TII->movImm(MBB, MBBI, DL, DstReg, Val, MachineInstr::NoFlags, Renamable,
              DstIsDead);

  MBBI->eraseFromParent();
  return true;
}

bool RISCVPostRAExpandPseudo::expandMovAddr(MachineBasicBlock &MBB,
                                            MachineBasicBlock::iterator MBBI) {
  DebugLoc DL = MBBI->getDebugLoc();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 108-122: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  Register DstReg = MBBI->getOperand(0).getReg();
  bool DstIsDead = MBBI->getOperand(0).isDead();
  bool Renamable = MBBI->getOperand(0).isRenamable();

  BuildMI(MBB, MBBI, DL, TII->get(RISCV::LUI))
      .addReg(DstReg, RegState::Define | getRenamableRegState(Renamable))
      .add(MBBI->getOperand(1));
  BuildMI(MBB, MBBI, DL, TII->get(RISCV::ADDI))
      .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead) |
                          getRenamableRegState(Renamable))
      .addReg(DstReg, RegState::Kill | getRenamableRegState(Renamable))
      .add(MBBI->getOperand(2));
  MBBI->eraseFromParent();
  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 123-132: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

/// Transfer implicit operands on the pseudo instruction to the
/// instructions created from the expansion.
static void transferImpOps(MachineInstr &OldMI, MachineInstrBuilder &MI) {
  const MCInstrDesc &Desc = OldMI.getDesc();
  for (const MachineOperand &MO :
       llvm::drop_begin(OldMI.operands(), Desc.getNumOperands())) {
    assert(MO.isReg() && MO.getReg());
    MI.add(MO);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 133-152: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

// Expand PseudoMERGE to MERGE, MVM, or MVMN.
bool RISCVPostRAExpandPseudo::expandMERGE(MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator MBBI) {
  MachineInstr &MI = *MBBI;
  DebugLoc DL = MI.getDebugLoc();

  Register DstReg = MI.getOperand(0).getReg();
  if (DstReg == MI.getOperand(3).getReg()) {
    // Expand to MVMN
    auto I = BuildMI(MBB, MBBI, DL, TII->get(RISCV::MVMN))
                 .add(MI.getOperand(0))
                 .add(MI.getOperand(3))
                 .add(MI.getOperand(2))
                 .add(MI.getOperand(1));
    transferImpOps(*MBBI, I);
  } else if (DstReg == MBBI->getOperand(2).getReg()) {
    // Expand to MVM
    auto I = BuildMI(MBB, MBBI, DL, TII->get(RISCV::MVM))
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 153-172: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
                 .add(MI.getOperand(0))
                 .add(MI.getOperand(2))
                 .add(MI.getOperand(3))
                 .add(MI.getOperand(1));
    transferImpOps(*MBBI, I);
  } else if (DstReg == MI.getOperand(1).getReg()) {
    // Expand to MERGE
    auto I = BuildMI(MBB, MBBI, DL, TII->get(RISCV::MERGE))
                 .add(MI.getOperand(0))
                 .add(MI.getOperand(1))
                 .add(MI.getOperand(2))
                 .add(MI.getOperand(3));
    transferImpOps(*MBBI, I);
  } else {
    // Use an additional move.
    RegState RegState =
        getRenamableRegState(MI.getOperand(1).isRenamable()) |
        getKillRegState(MI.getOperand(1).isKill() &&
                        MI.getOperand(1).getReg() !=
                            MI.getOperand(2).getReg() &&
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 173-186: Definitions and supporting logic / 定义与支撑逻辑
```cpp
                        MI.getOperand(1).getReg() != MI.getOperand(3).getReg());
    BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(RISCV::ADDI))
        .addDef(DstReg, getRenamableRegState(MI.getOperand(0).isRenamable()))
        .addReg(MI.getOperand(1).getReg(), RegState)
        .addImm(0);
    auto I = BuildMI(MBB, MBBI, MI.getDebugLoc(), TII->get(RISCV::MERGE))
                 .add(MI.getOperand(0))
                 .addReg(DstReg,
                         RegState::Kill | getRenamableRegState(
                                              MI.getOperand(0).isRenamable()))
                 .add(MI.getOperand(2))
                 .add(MI.getOperand(3));
    transferImpOps(*MBBI, I);
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 187-196: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  MI.eraseFromParent();
  return true;
}

} // end of anonymous namespace

INITIALIZE_PASS(RISCVPostRAExpandPseudo, "riscv-post-ra-expand-pseudo",
                RISCV_POST_RA_EXPAND_PSEUDO_NAME, false, false)
namespace llvm {
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 197-201: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
FunctionPass *createRISCVPostRAExpandPseudoPass() {
  return new RISCVPostRAExpandPseudo();
}

} // end of namespace llvm
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

## Key Concepts / 关键概念
- **Pseudo-instruction expansion** / **伪指令展开**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineInstrBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
