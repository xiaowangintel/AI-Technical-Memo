# RISCVRedundantCopyElimination.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVRedundantCopyElimination.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements elimination of redundant register copies in RISC-V machine code. / 实现消除 RISC-V 机器码中的冗余寄存器拷贝。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Commentary and design intent / 注释与设计意图
```cpp
//=- RISCVRedundantCopyElimination.cpp - Remove useless copy for RISC-V -----=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass removes unnecessary zero copies in BBs that are targets of
// beqz/bnez instructions. For instance, the copy instruction in the code below
// can be removed because the beqz jumps to BB#2 when a0 is zero.
//  BB#1:
//    beqz %a0, <BB#2>
//  BB#2:
//    %a0 = COPY %x0
//
// This pass also recognizes Xqcibi branch-immediate forms when compared
// against non-zero immediates.
//
// This pass should be run after register allocation and is based on the
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 21-34: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
// earliest versions of AArch64RedundantCopyElimination.
//
// FIXME: Support compare with non-zero immediates where the immediate is stored
// in a register.
//
//===----------------------------------------------------------------------===//

#include "RISCV.h"
#include "RISCVInstrInfo.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Support/Debug.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 35-46: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-copyelim"

STATISTIC(NumCopiesRemoved, "Number of copies removed.");

namespace {
class RISCVRedundantCopyElimination : public MachineFunctionPass {
  const MachineRegisterInfo *MRI;
  const TargetRegisterInfo *TRI;
  const TargetInstrInfo *TII;
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 47-58: Function implementation: RISCVRedundantCopyElimination / 函数实现：RISCVRedundantCopyElimination
```cpp
public:
  static char ID;
  RISCVRedundantCopyElimination() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;
  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoVRegs();
  }

  StringRef getPassName() const override {
    return "RISC-V Redundant Copy Elimination";
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 59-70: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

private:
  bool optimizeBlock(MachineBasicBlock &MBB);
};

} // end anonymous namespace

char RISCVRedundantCopyElimination::ID = 0;

INITIALIZE_PASS(RISCVRedundantCopyElimination, "riscv-copyelim",
                "RISC-V Redundant Copy Elimination", false, false)
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 71-85: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
static bool
guaranteesZeroRegInBlock(MachineBasicBlock &MBB,
                         const SmallVectorImpl<MachineOperand> &Cond,
                         MachineBasicBlock *TBB) {
  assert(Cond.size() == 3 && "Unexpected number of operands");
  assert(TBB != nullptr && "Expected branch target basic block");
  auto Opc = Cond[0].getImm();
  if (Opc == RISCV::BEQ && Cond[2].isReg() && Cond[2].getReg() == RISCV::X0 &&
      TBB == &MBB)
    return true;
  if (Opc == RISCV::BNE && Cond[2].isReg() && Cond[2].getReg() == RISCV::X0 &&
      TBB != &MBB)
    return true;
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 86-103: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static bool
guaranteesRegEqualsImmInBlock(MachineBasicBlock &MBB,
                              const SmallVectorImpl<MachineOperand> &Cond,
                              MachineBasicBlock *TBB) {
  assert(Cond.size() == 3 && "Unexpected number of operands");
  assert(TBB != nullptr && "Expected branch target basic block");
  auto Opc = Cond[0].getImm();
  if ((Opc == RISCV::QC_BEQI || Opc == RISCV::QC_E_BEQI ||
       Opc == RISCV::NDS_BEQC || Opc == RISCV::BEQI) &&
      Cond[2].isImm() && Cond[2].getImm() != 0 && TBB == &MBB)
    return true;
  if ((Opc == RISCV::QC_BNEI || Opc == RISCV::QC_E_BNEI ||
       Opc == RISCV::NDS_BNEC || Opc == RISCV::BNEI) &&
      Cond[2].isImm() && Cond[2].getImm() != 0 && TBB != &MBB)
    return true;
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 104-115: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVRedundantCopyElimination::optimizeBlock(MachineBasicBlock &MBB) {
  // Check if the current basic block has a single predecessor.
  if (MBB.pred_size() != 1)
    return false;

  // Check if the predecessor has two successors, implying the block ends in a
  // conditional branch.
  MachineBasicBlock *PredMBB = *MBB.pred_begin();
  if (PredMBB->succ_size() != 2)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 116-126: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  MachineBasicBlock *TBB = nullptr, *FBB = nullptr;
  SmallVector<MachineOperand, 3> Cond;
  if (TII->analyzeBranch(*PredMBB, TBB, FBB, Cond, /*AllowModify*/ false) ||
      Cond.empty())
    return false;

  Register TargetReg = Cond[1].getReg();

  if (!TargetReg)
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 127-144: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  bool IsZeroCopy = guaranteesZeroRegInBlock(MBB, Cond, TBB);

  if (!IsZeroCopy && !guaranteesRegEqualsImmInBlock(MBB, Cond, TBB))
    return false;

  bool Changed = false;
  MachineBasicBlock::iterator LastChange = MBB.begin();
  // Remove redundant Copy instructions unless TargetReg is modified.
  for (MachineBasicBlock::iterator I = MBB.begin(), E = MBB.end(); I != E;) {
    MachineInstr *MI = &*I;
    ++I;
    bool RemoveMI = false;
    if (IsZeroCopy) {
      if (MI->isCopy() && MI->getOperand(0).isReg() &&
          MI->getOperand(1).isReg()) {
        Register DefReg = MI->getOperand(0).getReg();
        Register SrcReg = MI->getOperand(1).getReg();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 145-164: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
        if (SrcReg == RISCV::X0 && !MRI->isReserved(DefReg) &&
            TargetReg == DefReg)
          RemoveMI = true;
      }
    } else {
      // Xqcibi, XAndesPref and Zibi compare with non-zero immediate:
      // remove redundant addi rd,x0,imm or qc.li rd,imm as applicable.
      if (MI->getOpcode() == RISCV::ADDI && MI->getOperand(0).isReg() &&
          MI->getOperand(1).isReg() && MI->getOperand(2).isImm()) {
        Register DefReg = MI->getOperand(0).getReg();
        Register SrcReg = MI->getOperand(1).getReg();
        int64_t Imm = MI->getOperand(2).getImm();
        if (SrcReg == RISCV::X0 && !MRI->isReserved(DefReg) &&
            TargetReg == DefReg && Imm == Cond[2].getImm())
          RemoveMI = true;
      } else if (MI->getOpcode() == RISCV::QC_LI && MI->getOperand(0).isReg() &&
                 MI->getOperand(1).isImm()) {
        Register DefReg = MI->getOperand(0).getReg();
        int64_t Imm = MI->getOperand(1).getImm();
        if (!MRI->isReserved(DefReg) && TargetReg == DefReg &&
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 165-179: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
            Imm == Cond[2].getImm())
          RemoveMI = true;
      }
    }

    if (RemoveMI) {
      LLVM_DEBUG(dbgs() << "Remove redundant Copy: ");
      LLVM_DEBUG(MI->print(dbgs()));

      MI->eraseFromParent();
      Changed = true;
      LastChange = I;
      ++NumCopiesRemoved;
      continue;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 180-199: Definitions and supporting logic / 定义与支撑逻辑
```cpp

    if (MI->modifiesRegister(TargetReg, TRI))
      break;
  }

  if (!Changed)
    return false;

  MachineBasicBlock::iterator CondBr = PredMBB->getFirstTerminator();
  assert((CondBr->getOpcode() == RISCV::BEQ ||
          CondBr->getOpcode() == RISCV::BNE ||
          CondBr->getOpcode() == RISCV::BEQI ||
          CondBr->getOpcode() == RISCV::BNEI ||
          CondBr->getOpcode() == RISCV::QC_BEQI ||
          CondBr->getOpcode() == RISCV::QC_BNEI ||
          CondBr->getOpcode() == RISCV::QC_E_BEQI ||
          CondBr->getOpcode() == RISCV::QC_E_BNEI ||
          CondBr->getOpcode() == RISCV::NDS_BEQC ||
          CondBr->getOpcode() == RISCV::NDS_BNEC) &&
         "Unexpected opcode");
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 200-209: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  assert(CondBr->getOperand(0).getReg() == TargetReg && "Unexpected register");

  // Otherwise, we have to fixup the use-def chain, starting with the
  // BEQ(I)/BNE(I). Conservatively mark as much as we can live.
  CondBr->clearRegisterKills(TargetReg, TRI);

  // Add newly used reg to the block's live-in list if it isn't there already.
  if (!MBB.isLiveIn(TargetReg))
    MBB.addLiveIn(TargetReg);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 210-220: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Clear any kills of TargetReg between CondBr and the last removed COPY.
  for (MachineInstr &MMI : make_range(MBB.begin(), LastChange))
    MMI.clearRegisterKills(TargetReg, TRI);

  return true;
}

bool RISCVRedundantCopyElimination::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 221-230: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  TII = MF.getSubtarget().getInstrInfo();
  TRI = MF.getSubtarget().getRegisterInfo();
  MRI = &MF.getRegInfo();

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF)
    Changed |= optimizeBlock(MBB);

  return Changed;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 231-234: Function implementation: llvm::createRISCVRedundantCopyEliminationPass / 函数实现：llvm::createRISCVRedundantCopyEliminationPass
```cpp

FunctionPass *llvm::createRISCVRedundantCopyEliminationPass() {
  return new RISCVRedundantCopyElimination();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/Statistic.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/Debug.h` — Directly referenced by this file. / 该文件直接引用的依赖。
