# RISCVPushPopOptimizer.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVPushPopOptimizer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements optimization of push/pop style save-restore sequences for RISC-V. / 实现优化 RISC-V 中 push/pop 风格的保存恢复序列。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```cpp
//===------- RISCVPushPopOptimizer.cpp - RISC-V Push/Pop opt. pass --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a pass that replaces Zcmp POP instructions with
// POPRET[Z] where possible.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-25: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVInstrInfo.h"
#include "RISCVMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineInstr.h"

using namespace llvm;

#define RISCV_PUSH_POP_OPT_NAME "RISC-V Zcmp Push/Pop optimization pass"

namespace {
struct RISCVPushPopOpt : public MachineFunctionPass {
  static char ID;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 26-38: Function implementation: RISCVPushPopOpt / 函数实现：RISCVPushPopOpt
```cpp
  RISCVPushPopOpt() : MachineFunctionPass(ID) {}

  const RISCVInstrInfo *TII;
  const TargetRegisterInfo *TRI;

  // Track which register units have been modified and used.
  LiveRegUnits ModifiedRegUnits, UsedRegUnits;

  bool usePopRet(MachineBasicBlock::iterator &MBBI,
                 MachineBasicBlock::iterator &NextI, bool IsReturnZero);
  bool adjustRetVal(MachineBasicBlock::iterator &MBBI);
  bool runOnMachineFunction(MachineFunction &Fn) override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 39-48: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  StringRef getPassName() const override { return RISCV_PUSH_POP_OPT_NAME; }
};

char RISCVPushPopOpt::ID = 0;

} // end of anonymous namespace

INITIALIZE_PASS(RISCVPushPopOpt, "riscv-push-pop-opt", RISCV_PUSH_POP_OPT_NAME,
                false, false)
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 49-58: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
static bool isPop(unsigned Opcode) {
  switch (Opcode) {
  case RISCV::CM_POP:
  case RISCV::QC_CM_POP:
    return true;
  default:
    return false;
  }
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 59-69: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
static unsigned getPopRetOpcode(unsigned PopOpcode, bool IsReturnZero) {
  assert(isPop(PopOpcode) && "Unexpected Pop Opcode");

  switch (PopOpcode) {
  case RISCV::CM_POP:
    return IsReturnZero ? RISCV::CM_POPRETZ : RISCV::CM_POPRET;
  case RISCV::QC_CM_POP:
    return IsReturnZero ? RISCV::QC_CM_POPRETZ : RISCV::QC_CM_POPRET;
  default:
    llvm_unreachable("Unhandled Pop Opcode");
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 70-84: Function implementation: RISCVPushPopOpt::usePopRet / 函数实现：RISCVPushPopOpt::usePopRet
```cpp
}

bool RISCVPushPopOpt::usePopRet(MachineBasicBlock::iterator &MBBI,
                                MachineBasicBlock::iterator &NextI,
                                bool IsReturnZero) {
  // Since Pseudo instruction lowering happen later in the pipeline,
  // this will detect all ret instruction.
  DebugLoc DL = NextI->getDebugLoc();
  unsigned Opc = getPopRetOpcode(MBBI->getOpcode(), IsReturnZero);
  MachineInstrBuilder PopRetBuilder =
      BuildMI(*NextI->getParent(), NextI, DL, TII->get(Opc))
          .add(MBBI->getOperand(0))
          .add(MBBI->getOperand(1))
          .setMIFlag(MachineInstr::FrameDestroy);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 85-97: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Copy over the variable implicit uses and defs from the CM_POP. They depend
  // on what register list has been picked during frame lowering.
  const MCInstrDesc &PopDesc = MBBI->getDesc();
  unsigned FirstNonDeclaredOp = PopDesc.getNumOperands() +
                                PopDesc.NumImplicitUses +
                                PopDesc.NumImplicitDefs;
  for (unsigned i = FirstNonDeclaredOp; i < MBBI->getNumOperands(); ++i)
    PopRetBuilder.add(MBBI->getOperand(i));

  MBBI->eraseFromParent();
  NextI->eraseFromParent();
  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 98-117: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Search for last assignment to a0 and if possible use ret_val slot of POP to
// store return value.
bool RISCVPushPopOpt::adjustRetVal(MachineBasicBlock::iterator &MBBI) {
  MachineBasicBlock::reverse_iterator RE = MBBI->getParent()->rend();
  // Track which register units have been modified and used between the POP
  // insn and the last assignment to register a0.
  ModifiedRegUnits.clear();
  UsedRegUnits.clear();
  // Since POP instruction is in Epilogue no normal instructions will follow
  // after it. Therefore search only previous ones to find the return value.
  for (MachineBasicBlock::reverse_iterator I =
           next_nodbg(MBBI.getReverse(), RE);
       I != RE; I = next_nodbg(I, RE)) {
    MachineInstr &MI = *I;
    if (auto OperandPair = TII->isCopyInstrImpl(MI)) {
      Register DestReg = OperandPair->Destination->getReg();
      Register Source = OperandPair->Source->getReg();
      if (DestReg == RISCV::X10 && Source == RISCV::X0) {
        MI.removeFromParent();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 118-128: Definitions and supporting logic / 定义与支撑逻辑
```cpp
        return true;
      }
    }
    // Update modified / used register units.
    LiveRegUnits::accumulateUsedDefed(MI, ModifiedRegUnits, UsedRegUnits, TRI);
    // If a0 was modified or used, there is no possibility
    // of using ret_val slot of POP instruction.
    if (!ModifiedRegUnits.available(RISCV::X10) ||
        !UsedRegUnits.available(RISCV::X10))
      return false;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 129-140: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return false;
}

bool RISCVPushPopOpt::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;

  // If Zcmp extension is not supported, abort.
  const RISCVSubtarget *Subtarget = &Fn.getSubtarget<RISCVSubtarget>();
  if (!Subtarget->hasStdExtZcmp() && !Subtarget->hasVendorXqccmp())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 141-157: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  TII = Subtarget->getInstrInfo();
  TRI = Subtarget->getRegisterInfo();

  // Resize the modified and used register unit trackers.  We do this once
  // per function and then clear the register units each time we determine
  // correct return value for the POP.
  ModifiedRegUnits.init(*TRI);
  UsedRegUnits.init(*TRI);

  bool Modified = false;
  for (auto &MBB : Fn) {
    // RET should be the only terminator.
    auto RetMBBI = MBB.getFirstTerminator();
    if (RetMBBI == MBB.end() || RetMBBI->getOpcode() != RISCV::PseudoRET ||
        RetMBBI == MBB.begin())
      continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 158-167: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    // The previous instruction should be a POP.
    auto PopMBBI = prev_nodbg(RetMBBI, MBB.begin());
    if (isPop(PopMBBI->getOpcode()) &&
        PopMBBI->getFlag(MachineInstr::FrameDestroy))
      Modified |= usePopRet(PopMBBI, RetMBBI, adjustRetVal(PopMBBI));
  }

  return Modified;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 168-172: Function implementation: llvm::createRISCVPushPopOptimizationPass / 函数实现：llvm::createRISCVPushPopOptimizationPass
```cpp
/// createRISCVPushPopOptimizationPass - returns an instance of the
/// Push/Pop optimization pass.
FunctionPass *llvm::createRISCVPushPopOptimizationPass() {
  return new RISCVPushPopOpt();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMachineFunctionInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineInstr.h` — Directly referenced by this file. / 该文件直接引用的依赖。
