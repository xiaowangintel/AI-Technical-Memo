# RISCVExpandAtomicPseudoInsts.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVExpandAtomicPseudoInsts.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements expansion of RISC-V atomic pseudo instructions into concrete machine sequences. / 实现将 RISC-V 原子伪指令展开为具体机器指令序列。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVExpandAtomicPseudoInsts.cpp - Expand atomic pseudo instrs. ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that expands atomic pseudo instructions into
// target instructions. This pass should be run at the last possible moment,
// avoiding the possibility for other passes to break the requirements for
// forward progress in the LR/SC block.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-28: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVInstrInfo.h"
#include "RISCVTargetMachine.h"

#include "llvm/CodeGen/LivePhysRegs.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"

using namespace llvm;

#define RISCV_EXPAND_ATOMIC_PSEUDO_NAME                                        \
  "RISC-V atomic pseudo instruction expansion pass"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 29-43: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
namespace {

class RISCVExpandAtomicPseudo : public MachineFunctionPass {
public:
  const RISCVSubtarget *STI;
  const RISCVInstrInfo *TII;
  static char ID;

  RISCVExpandAtomicPseudo() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return RISCV_EXPAND_ATOMIC_PSEUDO_NAME;
  }
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 44-67: Header guard and interface framing / 头文件保护与接口框架
```cpp

private:
  bool expandMBB(MachineBasicBlock &MBB);
  bool expandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                MachineBasicBlock::iterator &NextMBBI);
  bool expandAtomicBinOp(MachineBasicBlock &MBB,
                         MachineBasicBlock::iterator MBBI, AtomicRMWInst::BinOp,
                         bool IsMasked, int Width,
                         MachineBasicBlock::iterator &NextMBBI);
  bool expandAtomicMinMaxOp(MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator MBBI,
                            AtomicRMWInst::BinOp, bool IsMasked, int Width,
                            MachineBasicBlock::iterator &NextMBBI);
  bool expandAtomicCmpXchg(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator MBBI, bool IsMasked,
                           int Width, MachineBasicBlock::iterator &NextMBBI);
#ifndef NDEBUG
  unsigned getInstSizeInBytes(const MachineFunction &MF) const {
    unsigned Size = 0;
    for (auto &MBB : MF)
      for (auto &MI : MBB)
        Size += TII->getInstSizeInBytes(MI);
    return Size;
  }
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 68-84: Header guard and interface framing / 头文件保护与接口框架
```cpp
#endif
};

char RISCVExpandAtomicPseudo::ID = 0;

bool RISCVExpandAtomicPseudo::runOnMachineFunction(MachineFunction &MF) {
  STI = &MF.getSubtarget<RISCVSubtarget>();
  TII = STI->getInstrInfo();

#ifndef NDEBUG
  const unsigned OldSize = getInstSizeInBytes(MF);
#endif

  bool Modified = false;
  for (auto &MBB : MF)
    Modified |= expandMBB(MBB);
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 85-100: Header guard and interface framing / 头文件保护与接口框架
```cpp
#ifndef NDEBUG
  const unsigned NewSize = getInstSizeInBytes(MF);
  assert(OldSize >= NewSize);
#endif
  return Modified;
}

bool RISCVExpandAtomicPseudo::expandMBB(MachineBasicBlock &MBB) {
  bool Modified = false;

  MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  while (MBBI != E) {
    MachineBasicBlock::iterator NMBBI = std::next(MBBI);
    Modified |= expandMI(MBB, MBBI, NMBBI);
    MBBI = NMBBI;
  }
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 101-128: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return Modified;
}

bool RISCVExpandAtomicPseudo::expandMI(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator MBBI,
                                       MachineBasicBlock::iterator &NextMBBI) {
  // RISCVInstrInfo::getInstSizeInBytes expects that the total size of the
  // expanded instructions for each pseudo is correct in the Size field of the
  // tablegen definition for the pseudo.
  switch (MBBI->getOpcode()) {
  case RISCV::PseudoAtomicSwap32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xchg, false, 32,
                             NextMBBI);
  case RISCV::PseudoAtomicSwap64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xchg, false, 64,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadAdd32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Add, false, 32,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadAdd64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Add, false, 64,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadSub32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Sub, false, 32,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadSub64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Sub, false, 64,
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 129-156: Definitions and supporting logic / 定义与支撑逻辑
```cpp
                             NextMBBI);
  case RISCV::PseudoAtomicLoadAnd32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::And, false, 32,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadAnd64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::And, false, 64,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadOr32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Or, false, 32, NextMBBI);
  case RISCV::PseudoAtomicLoadOr64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Or, false, 64, NextMBBI);
  case RISCV::PseudoAtomicLoadXor32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xor, false, 32,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadXor64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xor, false, 64,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadNand32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Nand, false, 32,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadNand64:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Nand, false, 64,
                             NextMBBI);
  case RISCV::PseudoAtomicLoadMin32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Min, false, 32,
                                NextMBBI);
  case RISCV::PseudoAtomicLoadMin64:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Min, false, 64,
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 157-184: Definitions and supporting logic / 定义与支撑逻辑
```cpp
                                NextMBBI);
  case RISCV::PseudoAtomicLoadMax32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Max, false, 32,
                                NextMBBI);
  case RISCV::PseudoAtomicLoadMax64:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Max, false, 64,
                                NextMBBI);
  case RISCV::PseudoAtomicLoadUMin32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMin, false, 32,
                                NextMBBI);
  case RISCV::PseudoAtomicLoadUMin64:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMin, false, 64,
                                NextMBBI);
  case RISCV::PseudoAtomicLoadUMax32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMax, false, 32,
                                NextMBBI);
  case RISCV::PseudoAtomicLoadUMax64:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMax, false, 64,
                                NextMBBI);
  case RISCV::PseudoMaskedAtomicSwap32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xchg, true, 32,
                             NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadAdd32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Add, true, 32, NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadSub32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Sub, true, 32, NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadNand32:
    return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Nand, true, 32,
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 185-204: Definitions and supporting logic / 定义与支撑逻辑
```cpp
                             NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadMax32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Max, true, 32,
                                NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadMin32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Min, true, 32,
                                NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadUMax32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMax, true, 32,
                                NextMBBI);
  case RISCV::PseudoMaskedAtomicLoadUMin32:
    return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMin, true, 32,
                                NextMBBI);
  case RISCV::PseudoCmpXchg32:
    return expandAtomicCmpXchg(MBB, MBBI, false, 32, NextMBBI);
  case RISCV::PseudoCmpXchg64:
    return expandAtomicCmpXchg(MBB, MBBI, false, 64, NextMBBI);
  case RISCV::PseudoMaskedCmpXchg32:
    return expandAtomicCmpXchg(MBB, MBBI, true, 32, NextMBBI);
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 205-228: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return false;
}

static unsigned getLRForRMW32(AtomicOrdering Ordering,
                              const RISCVSubtarget *Subtarget) {
  switch (Ordering) {
  default:
    llvm_unreachable("Unexpected AtomicOrdering");
  case AtomicOrdering::Monotonic:
    return RISCV::LR_W;
  case AtomicOrdering::Acquire:
    if (Subtarget->hasStdExtZtso())
      return RISCV::LR_W;
    return RISCV::LR_W_AQ;
  case AtomicOrdering::Release:
    return RISCV::LR_W;
  case AtomicOrdering::AcquireRelease:
    if (Subtarget->hasStdExtZtso())
      return RISCV::LR_W;
    return RISCV::LR_W_AQ;
  case AtomicOrdering::SequentiallyConsistent:
    return RISCV::LR_W_AQRL;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 229-250: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

static unsigned getSCForRMW32(AtomicOrdering Ordering,
                              const RISCVSubtarget *Subtarget) {
  switch (Ordering) {
  default:
    llvm_unreachable("Unexpected AtomicOrdering");
  case AtomicOrdering::Monotonic:
    return RISCV::SC_W;
  case AtomicOrdering::Acquire:
    return RISCV::SC_W;
  case AtomicOrdering::Release:
    if (Subtarget->hasStdExtZtso())
      return RISCV::SC_W;
    return RISCV::SC_W_RL;
  case AtomicOrdering::AcquireRelease:
    if (Subtarget->hasStdExtZtso())
      return RISCV::SC_W;
    return RISCV::SC_W_RL;
  case AtomicOrdering::SequentiallyConsistent:
    return RISCV::SC_W_RL;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 251-272: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

static unsigned getLRForRMW64(AtomicOrdering Ordering,
                              const RISCVSubtarget *Subtarget) {
  switch (Ordering) {
  default:
    llvm_unreachable("Unexpected AtomicOrdering");
  case AtomicOrdering::Monotonic:
    return RISCV::LR_D;
  case AtomicOrdering::Acquire:
    if (Subtarget->hasStdExtZtso())
      return RISCV::LR_D;
    return RISCV::LR_D_AQ;
  case AtomicOrdering::Release:
    return RISCV::LR_D;
  case AtomicOrdering::AcquireRelease:
    if (Subtarget->hasStdExtZtso())
      return RISCV::LR_D;
    return RISCV::LR_D_AQ;
  case AtomicOrdering::SequentiallyConsistent:
    return RISCV::LR_D_AQRL;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 273-294: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

static unsigned getSCForRMW64(AtomicOrdering Ordering,
                              const RISCVSubtarget *Subtarget) {
  switch (Ordering) {
  default:
    llvm_unreachable("Unexpected AtomicOrdering");
  case AtomicOrdering::Monotonic:
    return RISCV::SC_D;
  case AtomicOrdering::Acquire:
    return RISCV::SC_D;
  case AtomicOrdering::Release:
    if (Subtarget->hasStdExtZtso())
      return RISCV::SC_D;
    return RISCV::SC_D_RL;
  case AtomicOrdering::AcquireRelease:
    if (Subtarget->hasStdExtZtso())
      return RISCV::SC_D;
    return RISCV::SC_D_RL;
  case AtomicOrdering::SequentiallyConsistent:
    return RISCV::SC_D_RL;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 295-313: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

static unsigned getLRForRMW(AtomicOrdering Ordering, int Width,
                            const RISCVSubtarget *Subtarget) {
  if (Width == 32)
    return getLRForRMW32(Ordering, Subtarget);
  if (Width == 64)
    return getLRForRMW64(Ordering, Subtarget);
  llvm_unreachable("Unexpected LR width\n");
}

static unsigned getSCForRMW(AtomicOrdering Ordering, int Width,
                            const RISCVSubtarget *Subtarget) {
  if (Width == 32)
    return getSCForRMW32(Ordering, Subtarget);
  if (Width == 64)
    return getSCForRMW64(Ordering, Subtarget);
  llvm_unreachable("Unexpected SC width\n");
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 314-327: Function implementation: doAtomicBinOpExpansion / 函数实现：doAtomicBinOpExpansion
```cpp

static void doAtomicBinOpExpansion(const RISCVInstrInfo *TII, MachineInstr &MI,
                                   DebugLoc DL, MachineBasicBlock *ThisMBB,
                                   MachineBasicBlock *LoopMBB,
                                   MachineBasicBlock *DoneMBB,
                                   AtomicRMWInst::BinOp BinOp, int Width,
                                   const RISCVSubtarget *STI) {
  Register DestReg = MI.getOperand(0).getReg();
  Register ScratchReg = MI.getOperand(1).getReg();
  Register AddrReg = MI.getOperand(2).getReg();
  Register IncrReg = MI.getOperand(3).getReg();
  AtomicOrdering Ordering =
      static_cast<AtomicOrdering>(MI.getOperand(4).getImm());
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 328-355: Commentary and design intent / 注释与设计意图
```cpp
  // .loop:
  //   lr.[w|d] dest, (addr)
  //   binop scratch, dest, val
  //   sc.[w|d] scratch, scratch, (addr)
  //   bnez scratch, loop
  BuildMI(LoopMBB, DL, TII->get(getLRForRMW(Ordering, Width, STI)), DestReg)
      .addReg(AddrReg);
  switch (BinOp) {
  default:
    llvm_unreachable("Unexpected AtomicRMW BinOp");
  case AtomicRMWInst::Xchg:
    BuildMI(LoopMBB, DL, TII->get(RISCV::ADDI), ScratchReg)
        .addReg(IncrReg)
        .addImm(0);
    break;
  case AtomicRMWInst::Add:
    BuildMI(LoopMBB, DL, TII->get(RISCV::ADD), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::Sub:
    BuildMI(LoopMBB, DL, TII->get(RISCV::SUB), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::And:
    BuildMI(LoopMBB, DL, TII->get(RISCV::AND), ScratchReg)
        .addReg(DestReg)
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 356-383: Definitions and supporting logic / 定义与支撑逻辑
```cpp
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::Or:
    BuildMI(LoopMBB, DL, TII->get(RISCV::OR), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::Xor:
    BuildMI(LoopMBB, DL, TII->get(RISCV::XOR), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::Nand:
    BuildMI(LoopMBB, DL, TII->get(RISCV::AND), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    BuildMI(LoopMBB, DL, TII->get(RISCV::XORI), ScratchReg)
        .addReg(ScratchReg)
        .addImm(-1);
    break;
  case AtomicRMWInst::Max:
    BuildMI(LoopMBB, DL, TII->get(RISCV::MAX), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::Min:
    BuildMI(LoopMBB, DL, TII->get(RISCV::MIN), ScratchReg)
        .addReg(DestReg)
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 384-404: Definitions and supporting logic / 定义与支撑逻辑
```cpp
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::UMax:
    BuildMI(LoopMBB, DL, TII->get(RISCV::MAXU), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::UMin:
    BuildMI(LoopMBB, DL, TII->get(RISCV::MINU), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  }
  BuildMI(LoopMBB, DL, TII->get(getSCForRMW(Ordering, Width, STI)), ScratchReg)
      .addReg(ScratchReg)
      .addReg(AddrReg);
  BuildMI(LoopMBB, DL, TII->get(RISCV::BNE))
      .addReg(ScratchReg)
      .addReg(RISCV::X0)
      .addMBB(LoopMBB);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 405-426: Function implementation: insertMaskedMerge / 函数实现：insertMaskedMerge
```cpp

static void insertMaskedMerge(const RISCVInstrInfo *TII, DebugLoc DL,
                              MachineBasicBlock *MBB, Register DestReg,
                              Register OldValReg, Register NewValReg,
                              Register MaskReg, Register ScratchReg) {
  assert(OldValReg != ScratchReg && "OldValReg and ScratchReg must be unique");
  assert(OldValReg != MaskReg && "OldValReg and MaskReg must be unique");
  assert(ScratchReg != MaskReg && "ScratchReg and MaskReg must be unique");

  // We select bits from newval and oldval using:
  // https://graphics.stanford.edu/~seander/bithacks.html#MaskedMerge
  // r = oldval ^ ((oldval ^ newval) & masktargetdata);
  BuildMI(MBB, DL, TII->get(RISCV::XOR), ScratchReg)
      .addReg(OldValReg)
      .addReg(NewValReg);
  BuildMI(MBB, DL, TII->get(RISCV::AND), ScratchReg)
      .addReg(ScratchReg)
      .addReg(MaskReg);
  BuildMI(MBB, DL, TII->get(RISCV::XOR), DestReg)
      .addReg(OldValReg)
      .addReg(ScratchReg);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 427-443: Function implementation: doMaskedAtomicBinOpExpansion / 函数实现：doMaskedAtomicBinOpExpansion
```cpp

static void doMaskedAtomicBinOpExpansion(const RISCVInstrInfo *TII,
                                         MachineInstr &MI, DebugLoc DL,
                                         MachineBasicBlock *ThisMBB,
                                         MachineBasicBlock *LoopMBB,
                                         MachineBasicBlock *DoneMBB,
                                         AtomicRMWInst::BinOp BinOp, int Width,
                                         const RISCVSubtarget *STI) {
  assert(Width == 32 && "Should never need to expand masked 64-bit operations");
  Register DestReg = MI.getOperand(0).getReg();
  Register ScratchReg = MI.getOperand(1).getReg();
  Register AddrReg = MI.getOperand(2).getReg();
  Register IncrReg = MI.getOperand(3).getReg();
  Register MaskReg = MI.getOperand(4).getReg();
  AtomicOrdering Ordering =
      static_cast<AtomicOrdering>(MI.getOperand(5).getImm());
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 444-471: Commentary and design intent / 注释与设计意图
```cpp
  // .loop:
  //   lr.w destreg, (alignedaddr)
  //   binop scratch, destreg, incr
  //   xor scratch, destreg, scratch
  //   and scratch, scratch, masktargetdata
  //   xor scratch, destreg, scratch
  //   sc.w scratch, scratch, (alignedaddr)
  //   bnez scratch, loop
  BuildMI(LoopMBB, DL, TII->get(getLRForRMW32(Ordering, STI)), DestReg)
      .addReg(AddrReg);
  switch (BinOp) {
  default:
    llvm_unreachable("Unexpected AtomicRMW BinOp");
  case AtomicRMWInst::Xchg:
    BuildMI(LoopMBB, DL, TII->get(RISCV::ADDI), ScratchReg)
        .addReg(IncrReg)
        .addImm(0);
    break;
  case AtomicRMWInst::Add:
    BuildMI(LoopMBB, DL, TII->get(RISCV::ADD), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
  case AtomicRMWInst::Sub:
    BuildMI(LoopMBB, DL, TII->get(RISCV::SUB), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    break;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 472-492: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case AtomicRMWInst::Nand:
    BuildMI(LoopMBB, DL, TII->get(RISCV::AND), ScratchReg)
        .addReg(DestReg)
        .addReg(IncrReg);
    BuildMI(LoopMBB, DL, TII->get(RISCV::XORI), ScratchReg)
        .addReg(ScratchReg)
        .addImm(-1);
    break;
  }

  insertMaskedMerge(TII, DL, LoopMBB, ScratchReg, DestReg, ScratchReg, MaskReg,
                    ScratchReg);

  BuildMI(LoopMBB, DL, TII->get(getSCForRMW32(Ordering, STI)), ScratchReg)
      .addReg(ScratchReg)
      .addReg(AddrReg);
  BuildMI(LoopMBB, DL, TII->get(RISCV::BNE))
      .addReg(ScratchReg)
      .addReg(RISCV::X0)
      .addMBB(LoopMBB);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 493-508: Function implementation: RISCVExpandAtomicPseudo::expandAtomicBinOp / 函数实现：RISCVExpandAtomicPseudo::expandAtomicBinOp
```cpp

bool RISCVExpandAtomicPseudo::expandAtomicBinOp(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
    AtomicRMWInst::BinOp BinOp, bool IsMasked, int Width,
    MachineBasicBlock::iterator &NextMBBI) {
  MachineInstr &MI = *MBBI;
  DebugLoc DL = MI.getDebugLoc();

  MachineFunction *MF = MBB.getParent();
  auto LoopMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());

  // Insert new MBBs.
  MF->insert(++MBB.getIterator(), LoopMBB);
  MF->insert(++LoopMBB->getIterator(), DoneMBB);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 509-522: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Set up successors and transfer remaining instructions to DoneMBB.
  LoopMBB->addSuccessor(LoopMBB);
  LoopMBB->addSuccessor(DoneMBB);
  DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
  DoneMBB->transferSuccessors(&MBB);
  MBB.addSuccessor(LoopMBB);

  if (!IsMasked)
    doAtomicBinOpExpansion(TII, MI, DL, &MBB, LoopMBB, DoneMBB, BinOp, Width,
                           STI);
  else
    doMaskedAtomicBinOpExpansion(TII, MI, DL, &MBB, LoopMBB, DoneMBB, BinOp,
                                 Width, STI);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 523-542: Function implementation: insertSext / 函数实现：insertSext
```cpp
  NextMBBI = MBB.end();
  MI.eraseFromParent();

  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *LoopMBB);
  computeAndAddLiveIns(LiveRegs, *DoneMBB);

  return true;
}

static void insertSext(const RISCVInstrInfo *TII, DebugLoc DL,
                       MachineBasicBlock *MBB, Register ValReg,
                       Register ShamtReg) {
  BuildMI(MBB, DL, TII->get(RISCV::SLL), ValReg)
      .addReg(ValReg)
      .addReg(ShamtReg);
  BuildMI(MBB, DL, TII->get(RISCV::SRA), ValReg)
      .addReg(ValReg)
      .addReg(ShamtReg);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 543-556: Function implementation: doAtomicMinMaxOpExpansion / 函数实现：doAtomicMinMaxOpExpansion
```cpp

static void doAtomicMinMaxOpExpansion(
    const RISCVInstrInfo *TII, MachineInstr &MI, DebugLoc DL,
    MachineBasicBlock *ThisMBB, MachineBasicBlock *LoopHeadMBB,
    MachineBasicBlock *LoopIfBodyMBB, MachineBasicBlock *LoopTailMBB,
    MachineBasicBlock *DoneMBB, AtomicRMWInst::BinOp BinOp, int Width,
    const RISCVSubtarget *STI) {
  Register DestReg = MI.getOperand(0).getReg();
  Register ScratchReg = MI.getOperand(1).getReg();
  Register AddrReg = MI.getOperand(2).getReg();
  Register IncrReg = MI.getOperand(3).getReg();
  AtomicOrdering Ordering =
      static_cast<AtomicOrdering>(MI.getOperand(4).getImm());
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 557-575: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // .loophead:
  //   lr.[w|d] dest, (addr)
  //   mv scratch, dest
  //   ifnochangeneeded scratch, incr, .looptail
  BuildMI(LoopHeadMBB, DL, TII->get(getLRForRMW(Ordering, Width, STI)), DestReg)
      .addReg(AddrReg);
  BuildMI(LoopHeadMBB, DL, TII->get(RISCV::ADDI), ScratchReg)
      .addReg(DestReg)
      .addImm(0);
  switch (BinOp) {
  default:
    llvm_unreachable("Unexpected AtomicRMW BinOp");
  case AtomicRMWInst::Max: {
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGE))
        .addReg(ScratchReg)
        .addReg(IncrReg)
        .addMBB(LoopTailMBB);
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 576-595: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  case AtomicRMWInst::Min: {
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGE))
        .addReg(IncrReg)
        .addReg(ScratchReg)
        .addMBB(LoopTailMBB);
    break;
  }
  case AtomicRMWInst::UMax:
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGEU))
        .addReg(ScratchReg)
        .addReg(IncrReg)
        .addMBB(LoopTailMBB);
    break;
  case AtomicRMWInst::UMin:
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGEU))
        .addReg(IncrReg)
        .addReg(ScratchReg)
        .addMBB(LoopTailMBB);
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 596-614: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // .loopifbody:
  //   mv scratch, incr
  BuildMI(LoopIfBodyMBB, DL, TII->get(RISCV::ADDI), ScratchReg)
      .addReg(IncrReg)
      .addImm(0);

  // .looptail:
  //   sc.[w|d] scratch, scratch, (addr)
  //   bnez scratch, loop
  BuildMI(LoopTailMBB, DL, TII->get(getSCForRMW(Ordering, Width, STI)),
          ScratchReg)
      .addReg(ScratchReg)
      .addReg(AddrReg);
  BuildMI(LoopTailMBB, DL, TII->get(RISCV::BNE))
      .addReg(ScratchReg)
      .addReg(RISCV::X0)
      .addMBB(LoopHeadMBB);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 615-632: Function implementation: doMaskedAtomicMinMaxOpExpansion / 函数实现：doMaskedAtomicMinMaxOpExpansion
```cpp

static void doMaskedAtomicMinMaxOpExpansion(
    const RISCVInstrInfo *TII, MachineInstr &MI, DebugLoc DL,
    MachineBasicBlock *ThisMBB, MachineBasicBlock *LoopHeadMBB,
    MachineBasicBlock *LoopIfBodyMBB, MachineBasicBlock *LoopTailMBB,
    MachineBasicBlock *DoneMBB, AtomicRMWInst::BinOp BinOp, int Width,
    const RISCVSubtarget *STI) {
  assert(Width == 32 && "Should never need to expand masked 64-bit operations");
  Register DestReg = MI.getOperand(0).getReg();
  Register Scratch1Reg = MI.getOperand(1).getReg();
  Register Scratch2Reg = MI.getOperand(2).getReg();
  Register AddrReg = MI.getOperand(3).getReg();
  Register IncrReg = MI.getOperand(4).getReg();
  Register MaskReg = MI.getOperand(5).getReg();
  bool IsSigned = BinOp == AtomicRMWInst::Min || BinOp == AtomicRMWInst::Max;
  AtomicOrdering Ordering =
      static_cast<AtomicOrdering>(MI.getOperand(IsSigned ? 7 : 6).getImm());
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 633-648: Commentary and design intent / 注释与设计意图
```cpp
  //
  // .loophead:
  //   lr.w destreg, (alignedaddr)
  //   and scratch2, destreg, mask
  //   mv scratch1, destreg
  //   [sext scratch2 if signed min/max]
  //   ifnochangeneeded scratch2, incr, .looptail
  BuildMI(LoopHeadMBB, DL, TII->get(getLRForRMW32(Ordering, STI)), DestReg)
      .addReg(AddrReg);
  BuildMI(LoopHeadMBB, DL, TII->get(RISCV::AND), Scratch2Reg)
      .addReg(DestReg)
      .addReg(MaskReg);
  BuildMI(LoopHeadMBB, DL, TII->get(RISCV::ADDI), Scratch1Reg)
      .addReg(DestReg)
      .addImm(0);
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 649-667: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  switch (BinOp) {
  default:
    llvm_unreachable("Unexpected AtomicRMW BinOp");
  case AtomicRMWInst::Max: {
    insertSext(TII, DL, LoopHeadMBB, Scratch2Reg, MI.getOperand(6).getReg());
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGE))
        .addReg(Scratch2Reg)
        .addReg(IncrReg)
        .addMBB(LoopTailMBB);
    break;
  }
  case AtomicRMWInst::Min: {
    insertSext(TII, DL, LoopHeadMBB, Scratch2Reg, MI.getOperand(6).getReg());
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGE))
        .addReg(IncrReg)
        .addReg(Scratch2Reg)
        .addMBB(LoopTailMBB);
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 668-681: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case AtomicRMWInst::UMax:
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGEU))
        .addReg(Scratch2Reg)
        .addReg(IncrReg)
        .addMBB(LoopTailMBB);
    break;
  case AtomicRMWInst::UMin:
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BGEU))
        .addReg(IncrReg)
        .addReg(Scratch2Reg)
        .addMBB(LoopTailMBB);
    break;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 682-699: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // .loopifbody:
  //   xor scratch1, destreg, incr
  //   and scratch1, scratch1, mask
  //   xor scratch1, destreg, scratch1
  insertMaskedMerge(TII, DL, LoopIfBodyMBB, Scratch1Reg, DestReg, IncrReg,
                    MaskReg, Scratch1Reg);

  // .looptail:
  //   sc.w scratch1, scratch1, (addr)
  //   bnez scratch1, loop
  BuildMI(LoopTailMBB, DL, TII->get(getSCForRMW32(Ordering, STI)), Scratch1Reg)
      .addReg(Scratch1Reg)
      .addReg(AddrReg);
  BuildMI(LoopTailMBB, DL, TII->get(RISCV::BNE))
      .addReg(Scratch1Reg)
      .addReg(RISCV::X0)
      .addMBB(LoopHeadMBB);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 700-716: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVExpandAtomicPseudo::expandAtomicMinMaxOp(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
    AtomicRMWInst::BinOp BinOp, bool IsMasked, int Width,
    MachineBasicBlock::iterator &NextMBBI) {
  // Using MIN(U)/MAX(U) is preferrable if permitted
  if (STI->hasPermissiveZalrsc() && STI->hasStdExtZbb() && !IsMasked)
    return expandAtomicBinOp(MBB, MBBI, BinOp, IsMasked, Width, NextMBBI);

  MachineInstr &MI = *MBBI;
  DebugLoc DL = MI.getDebugLoc();
  MachineFunction *MF = MBB.getParent();
  auto LoopHeadMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto LoopIfBodyMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto LoopTailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 717-732: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Insert new MBBs.
  MF->insert(++MBB.getIterator(), LoopHeadMBB);
  MF->insert(++LoopHeadMBB->getIterator(), LoopIfBodyMBB);
  MF->insert(++LoopIfBodyMBB->getIterator(), LoopTailMBB);
  MF->insert(++LoopTailMBB->getIterator(), DoneMBB);

  // Set up successors and transfer remaining instructions to DoneMBB.
  LoopHeadMBB->addSuccessor(LoopIfBodyMBB);
  LoopHeadMBB->addSuccessor(LoopTailMBB);
  LoopIfBodyMBB->addSuccessor(LoopTailMBB);
  LoopTailMBB->addSuccessor(LoopHeadMBB);
  LoopTailMBB->addSuccessor(DoneMBB);
  DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
  DoneMBB->transferSuccessors(&MBB);
  MBB.addSuccessor(LoopHeadMBB);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 733-749: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (!IsMasked)
    doAtomicMinMaxOpExpansion(TII, MI, DL, &MBB, LoopHeadMBB, LoopIfBodyMBB,
                              LoopTailMBB, DoneMBB, BinOp, Width, STI);
  else
    doMaskedAtomicMinMaxOpExpansion(TII, MI, DL, &MBB, LoopHeadMBB,
                                    LoopIfBodyMBB, LoopTailMBB, DoneMBB, BinOp,
                                    Width, STI);

  NextMBBI = MBB.end();
  MI.eraseFromParent();

  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *LoopHeadMBB);
  computeAndAddLiveIns(LiveRegs, *LoopIfBodyMBB);
  computeAndAddLiveIns(LiveRegs, *LoopTailMBB);
  computeAndAddLiveIns(LiveRegs, *DoneMBB);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 750-774: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return true;
}

// If a BNE on the cmpxchg comparison result immediately follows the cmpxchg
// operation, it can be folded into the cmpxchg expansion by
// modifying the branch within 'LoopHead' (which performs the same
// comparison). This is a valid transformation because after altering the
// LoopHead's BNE destination, the BNE following the cmpxchg becomes
// redundant and and be deleted. In the case of a masked cmpxchg, an
// appropriate AND and BNE must be matched.
//
// On success, returns true and deletes the matching BNE or AND+BNE, sets the
// LoopHeadBNETarget argument to the target that should be used within the
// loop head, and removes that block as a successor to MBB.
bool tryToFoldBNEOnCmpXchgResult(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MBBI,
                                 Register DestReg, Register CmpValReg,
                                 Register MaskReg,
                                 MachineBasicBlock *&LoopHeadBNETarget) {
  SmallVector<MachineInstr *> ToErase;
  auto E = MBB.end();
  if (MBBI == E)
    return false;
  MBBI = skipDebugInstructionsForward(MBBI, E);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 775-788: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If we have a masked cmpxchg, match AND dst, DestReg, MaskReg.
  if (MaskReg.isValid()) {
    if (MBBI == E || MBBI->getOpcode() != RISCV::AND)
      return false;
    Register ANDOp1 = MBBI->getOperand(1).getReg();
    Register ANDOp2 = MBBI->getOperand(2).getReg();
    if (!(ANDOp1 == DestReg && ANDOp2 == MaskReg) &&
        !(ANDOp1 == MaskReg && ANDOp2 == DestReg))
      return false;
    // We now expect the BNE to use the result of the AND as an operand.
    DestReg = MBBI->getOperand(0).getReg();
    ToErase.push_back(&*MBBI);
    MBBI = skipDebugInstructionsForward(std::next(MBBI), E);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 789-805: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Match BNE DestReg, MaskReg.
  if (MBBI == E || MBBI->getOpcode() != RISCV::BNE)
    return false;
  Register BNEOp0 = MBBI->getOperand(0).getReg();
  Register BNEOp1 = MBBI->getOperand(1).getReg();
  if (!(BNEOp0 == DestReg && BNEOp1 == CmpValReg) &&
      !(BNEOp0 == CmpValReg && BNEOp1 == DestReg))
    return false;

  // Make sure the branch is the only user of the AND.
  if (MaskReg.isValid()) {
    if (BNEOp0 == DestReg && !MBBI->getOperand(0).isKill())
      return false;
    if (BNEOp1 == DestReg && !MBBI->getOperand(1).isKill())
      return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 806-828: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  ToErase.push_back(&*MBBI);
  LoopHeadBNETarget = MBBI->getOperand(2).getMBB();
  MBBI = skipDebugInstructionsForward(std::next(MBBI), E);
  if (MBBI != E)
    return false;

  MBB.removeSuccessor(LoopHeadBNETarget);
  for (auto *MI : ToErase)
    MI->eraseFromParent();
  return true;
}

bool RISCVExpandAtomicPseudo::expandAtomicCmpXchg(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, bool IsMasked,
    int Width, MachineBasicBlock::iterator &NextMBBI) {
  MachineInstr &MI = *MBBI;
  DebugLoc DL = MI.getDebugLoc();
  MachineFunction *MF = MBB.getParent();
  auto LoopHeadMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto LoopTailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
  auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 829-844: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  Register DestReg = MI.getOperand(0).getReg();
  Register ScratchReg = MI.getOperand(1).getReg();
  Register AddrReg = MI.getOperand(2).getReg();
  Register CmpValReg = MI.getOperand(3).getReg();
  Register NewValReg = MI.getOperand(4).getReg();
  Register MaskReg = IsMasked ? MI.getOperand(5).getReg() : Register();

  MachineBasicBlock *LoopHeadBNETarget = DoneMBB;
  tryToFoldBNEOnCmpXchgResult(MBB, std::next(MBBI), DestReg, CmpValReg, MaskReg,
                              LoopHeadBNETarget);

  // Insert new MBBs.
  MF->insert(++MBB.getIterator(), LoopHeadMBB);
  MF->insert(++LoopHeadMBB->getIterator(), LoopTailMBB);
  MF->insert(++LoopTailMBB->getIterator(), DoneMBB);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 845-872: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Set up successors and transfer remaining instructions to DoneMBB.
  LoopHeadMBB->addSuccessor(LoopTailMBB);
  LoopHeadMBB->addSuccessor(LoopHeadBNETarget);
  LoopTailMBB->addSuccessor(DoneMBB);
  LoopTailMBB->addSuccessor(LoopHeadMBB);
  DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
  DoneMBB->transferSuccessors(&MBB);
  MBB.addSuccessor(LoopHeadMBB);

  AtomicOrdering Ordering =
      static_cast<AtomicOrdering>(MI.getOperand(IsMasked ? 6 : 5).getImm());

  if (!IsMasked) {
    // .loophead:
    //   lr.[w|d] dest, (addr)
    //   bne dest, cmpval, done
    BuildMI(LoopHeadMBB, DL, TII->get(getLRForRMW(Ordering, Width, STI)),
            DestReg)
        .addReg(AddrReg);
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BNE))
        .addReg(DestReg)
        .addReg(CmpValReg)
        .addMBB(LoopHeadBNETarget);
    // .looptail:
    //   sc.[w|d] scratch, newval, (addr)
    //   bnez scratch, loophead
    BuildMI(LoopTailMBB, DL, TII->get(getSCForRMW(Ordering, Width, STI)),
            ScratchReg)
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 873-895: Function implementation / 函数实现
```cpp
        .addReg(NewValReg)
        .addReg(AddrReg);
    BuildMI(LoopTailMBB, DL, TII->get(RISCV::BNE))
        .addReg(ScratchReg)
        .addReg(RISCV::X0)
        .addMBB(LoopHeadMBB);
  } else {
    // .loophead:
    //   lr.w dest, (addr)
    //   and scratch, dest, mask
    //   bne scratch, cmpval, done
    Register MaskReg = MI.getOperand(5).getReg();
    BuildMI(LoopHeadMBB, DL, TII->get(getLRForRMW(Ordering, Width, STI)),
            DestReg)
        .addReg(AddrReg);
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::AND), ScratchReg)
        .addReg(DestReg)
        .addReg(MaskReg);
    BuildMI(LoopHeadMBB, DL, TII->get(RISCV::BNE))
        .addReg(ScratchReg)
        .addReg(CmpValReg)
        .addMBB(LoopHeadBNETarget);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 896-912: Commentary and design intent / 注释与设计意图
```cpp
    // .looptail:
    //   xor scratch, dest, newval
    //   and scratch, scratch, mask
    //   xor scratch, dest, scratch
    //   sc.w scratch, scratch, (adrr)
    //   bnez scratch, loophead
    insertMaskedMerge(TII, DL, LoopTailMBB, ScratchReg, DestReg, NewValReg,
                      MaskReg, ScratchReg);
    BuildMI(LoopTailMBB, DL, TII->get(getSCForRMW(Ordering, Width, STI)),
            ScratchReg)
        .addReg(ScratchReg)
        .addReg(AddrReg);
    BuildMI(LoopTailMBB, DL, TII->get(RISCV::BNE))
        .addReg(ScratchReg)
        .addReg(RISCV::X0)
        .addMBB(LoopHeadMBB);
  }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 913-926: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

  NextMBBI = MBB.end();
  MI.eraseFromParent();

  LivePhysRegs LiveRegs;
  computeAndAddLiveIns(LiveRegs, *LoopHeadMBB);
  computeAndAddLiveIns(LiveRegs, *LoopTailMBB);
  computeAndAddLiveIns(LiveRegs, *DoneMBB);

  return true;
}

} // end of anonymous namespace
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 927-932: Function implementation: llvm::createRISCVExpandAtomicPseudoPass / 函数实现：llvm::createRISCVExpandAtomicPseudoPass
```cpp
INITIALIZE_PASS(RISCVExpandAtomicPseudo, "riscv-expand-atomic-pseudo",
                RISCV_EXPAND_ATOMIC_PSEUDO_NAME, false, false)

FunctionPass *llvm::createRISCVExpandAtomicPseudoPass() {
  return new RISCVExpandAtomicPseudo();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **Pseudo-instruction expansion** / **伪指令展开**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/LivePhysRegs.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineInstrBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
