# RISCVInsertReadWriteCSR.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInsertReadWriteCSR.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements insertion of CSR read/write instructions required by RISC-V semantics. / 实现插入 RISC-V 语义所需的 CSR 读写指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVInsertReadWriteCSR.cpp - Insert Read/Write of RISC-V CSR -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This file implements the machine function pass to insert read/write of CSR-s
// of the RISC-V instructions.
//
// Currently the pass implements:
// -Writing and saving frm before an RVV floating-point instruction with a
//  static rounding mode and restores the value after.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-25: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "MCTargetDesc/RISCVBaseInfo.h"
#include "RISCV.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
using namespace llvm;

#define DEBUG_TYPE "riscv-insert-read-write-csr"
#define RISCV_INSERT_READ_WRITE_CSR_NAME "RISC-V Insert Read/Write CSR Pass"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 26-35: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
static cl::opt<bool>
    DisableFRMInsertOpt("riscv-disable-frm-insert-opt", cl::init(false),
                        cl::Hidden,
                        cl::desc("Disable optimized frm insertion."));

namespace {

class RISCVInsertReadWriteCSR : public MachineFunctionPass {
  const TargetInstrInfo *TII;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 36-46: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
public:
  static char ID;

  RISCVInsertReadWriteCSR() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 47-56: Function implementation: getPassName / 函数实现：getPassName
```cpp

  StringRef getPassName() const override {
    return RISCV_INSERT_READ_WRITE_CSR_NAME;
  }

private:
  bool emitWriteRoundingMode(MachineBasicBlock &MBB);
  bool emitWriteRoundingModeOpt(MachineBasicBlock &MBB);
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 57-70: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // end anonymous namespace

char RISCVInsertReadWriteCSR::ID = 0;

INITIALIZE_PASS(RISCVInsertReadWriteCSR, DEBUG_TYPE,
                RISCV_INSERT_READ_WRITE_CSR_NAME, false, false)

// TODO: Use more accurate rounding mode at the start of MBB.
bool RISCVInsertReadWriteCSR::emitWriteRoundingModeOpt(MachineBasicBlock &MBB) {
  bool Changed = false;
  MachineInstr *LastFRMChanger = nullptr;
  unsigned CurrentRM = RISCVFPRndMode::DYN;
  Register SavedFRM;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 71-83: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (MachineInstr &MI : MBB) {
    if (MI.getOpcode() == RISCV::SwapFRMImm ||
        MI.getOpcode() == RISCV::WriteFRMImm) {
      CurrentRM = MI.getOperand(0).getImm();
      SavedFRM = Register();
      continue;
    }

    if (MI.getOpcode() == RISCV::WriteFRM) {
      CurrentRM = RISCVFPRndMode::DYN;
      SavedFRM = Register();
      continue;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 84-94: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    if (MI.isCall() || MI.isInlineAsm() ||
        MI.readsRegister(RISCV::FRM, /*TRI=*/nullptr)) {
      // Restore FRM before unknown operations.
      if (SavedFRM.isValid())
        BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(RISCV::WriteFRM))
            .addReg(SavedFRM);
      CurrentRM = RISCVFPRndMode::DYN;
      SavedFRM = Register();
      continue;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 95-105: Definitions and supporting logic / 定义与支撑逻辑
```cpp

    assert(!MI.modifiesRegister(RISCV::FRM, /*TRI=*/nullptr) &&
           "Expected that MI could not modify FRM.");

    int FRMIdx = RISCVII::getFRMOpNum(MI.getDesc());
    if (FRMIdx < 0)
      continue;
    unsigned InstrRM = MI.getOperand(FRMIdx).getImm();

    LastFRMChanger = &MI;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 106-125: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Make MI implicit use FRM.
    MI.addOperand(MachineOperand::CreateReg(RISCV::FRM, /*IsDef*/ false,
                                            /*IsImp*/ true));
    Changed = true;

    // Skip if MI uses same rounding mode as FRM.
    if (InstrRM == CurrentRM)
      continue;

    if (!SavedFRM.isValid()) {
      // Save current FRM value to SavedFRM.
      MachineRegisterInfo *MRI = &MBB.getParent()->getRegInfo();
      SavedFRM = MRI->createVirtualRegister(&RISCV::GPRRegClass);
      BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(RISCV::SwapFRMImm), SavedFRM)
          .addImm(InstrRM);
    } else {
      // Don't need to save current FRM when SavedFRM having value.
      BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(RISCV::WriteFRMImm))
          .addImm(InstrRM);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 126-136: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    CurrentRM = InstrRM;
  }

  // Restore FRM if needed.
  if (SavedFRM.isValid()) {
    assert(LastFRMChanger && "Expected valid pointer.");
    MachineInstrBuilder MIB =
        BuildMI(*MBB.getParent(), {}, TII->get(RISCV::WriteFRM))
            .addReg(SavedFRM);
    MBB.insertAfter(LastFRMChanger, MIB);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 137-149: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return Changed;
}

// This function also swaps frm and restores it when encountering an RVV
// floating point instruction with a static rounding mode.
bool RISCVInsertReadWriteCSR::emitWriteRoundingMode(MachineBasicBlock &MBB) {
  bool Changed = false;
  for (MachineInstr &MI : MBB) {
    int FRMIdx = RISCVII::getFRMOpNum(MI.getDesc());
    if (FRMIdx < 0)
      continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 150-169: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    unsigned FRMImm = MI.getOperand(FRMIdx).getImm();

    // The value is a hint to this pass to not alter the frm value.
    if (FRMImm == RISCVFPRndMode::DYN)
      continue;

    Changed = true;

    // Save
    MachineRegisterInfo *MRI = &MBB.getParent()->getRegInfo();
    Register SavedFRM = MRI->createVirtualRegister(&RISCV::GPRRegClass);
    BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(RISCV::SwapFRMImm),
            SavedFRM)
        .addImm(FRMImm);
    MI.addOperand(MachineOperand::CreateReg(RISCV::FRM, /*IsDef*/ false,
                                            /*IsImp*/ true));
    // Restore
    MachineInstrBuilder MIB =
        BuildMI(*MBB.getParent(), {}, TII->get(RISCV::WriteFRM))
            .addReg(SavedFRM);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 170-180: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    MBB.insertAfter(MI, MIB);
  }
  return Changed;
}

bool RISCVInsertReadWriteCSR::runOnMachineFunction(MachineFunction &MF) {
  // Skip if the vector extension is not enabled.
  const RISCVSubtarget &ST = MF.getSubtarget<RISCVSubtarget>();
  if (!ST.hasVInstructions())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 181-190: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  TII = ST.getInstrInfo();

  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {
    if (DisableFRMInsertOpt)
      Changed |= emitWriteRoundingMode(MBB);
    else
      Changed |= emitWriteRoundingModeOpt(MBB);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 191-197: Function implementation: llvm::createRISCVInsertReadWriteCSRPass / 函数实现：llvm::createRISCVInsertReadWriteCSRPass
```cpp

  return Changed;
}

FunctionPass *llvm::createRISCVInsertReadWriteCSRPass() {
  return new RISCVInsertReadWriteCSR();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `MCTargetDesc/RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
