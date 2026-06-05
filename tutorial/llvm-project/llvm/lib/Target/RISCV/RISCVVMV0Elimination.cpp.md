# RISCVVMV0Elimination.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVVMV0Elimination.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements elimination or rewriting of special vmv0 vector move patterns. / 实现消除或重写特殊 vmv0 向量移动模式。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Commentary and design intent / 注释与设计意图
```cpp
//===- RISCVVMV0Elimination.cpp - VMV0 Elimination -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===---------------------------------------------------------------------===//
//
// Mask operands in vector pseudos have to be in v0. We select them as a virtual
// register in the singleton vmv0 register class instead of copying them to $v0
// straight away, to make optimizing masks easier.
//
// However register coalescing may end up coleascing copies into vmv0, resulting
// in instructions with multiple uses of vmv0 that the register allocator can't
// allocate:
//
// %x:vrnov0 = PseudoVADD_VV_M1_MASK %0:vrnov0, %1:vr, %2:vmv0, %3:vmv0, ...
//
// To avoid this, this pass replaces any uses* of vmv0 with copies to $v0 before
// register coalescing and allocation:
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 21-30: Commentary and design intent / 注释与设计意图
```cpp
//
// %x:vrnov0 = PseudoVADD_VV_M1_MASK %0:vrnov0, %1:vr, %2:vr, %3:vmv0, ...
// ->
// $v0 = COPY %3:vr
// %x:vrnov0 = PseudoVADD_VV_M1_MASK %0:vrnov0, %1:vr, %2:vr, $0, ...
//
// * The only uses of vmv0 left behind are when used for inline asm with the vm
// constraint.
//
//===---------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 31-40: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVSubtarget.h"
#ifndef NDEBUG
#include "llvm/ADT/PostOrderIterator.h"
#endif
#include "llvm/CodeGen/MachineFunctionPass.h"

using namespace llvm;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 41-51: Header guard and interface framing / 头文件保护与接口框架
```cpp
#define DEBUG_TYPE "riscv-vmv0-elimination"

namespace {

class RISCVVMV0Elimination : public MachineFunctionPass {
public:
  static char ID;
  RISCVVMV0Elimination() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 52-62: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  MachineFunctionProperties getRequiredProperties() const override {
    // TODO: We could move this closer to regalloc, out of SSA, which would
    // allow scheduling past mask operands. We would need to preserve live
    // intervals.
    return MachineFunctionProperties().setIsSSA();
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 63-74: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
};

} // namespace

char RISCVVMV0Elimination::ID = 0;

INITIALIZE_PASS(RISCVVMV0Elimination, DEBUG_TYPE, "RISC-V VMV0 Elimination",
                false, false)

FunctionPass *llvm::createRISCVVMV0EliminationPass() {
  return new RISCVVMV0Elimination();
}
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 75-85: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static bool isVMV0(const MCOperandInfo &MCOI) {
  return MCOI.RegClass == RISCV::VMV0RegClassID;
}

bool RISCVVMV0Elimination::runOnMachineFunction(MachineFunction &MF) {
  // Skip if the vector extension is not enabled.
  const RISCVSubtarget *ST = &MF.getSubtarget<RISCVSubtarget>();
  if (!ST->hasVInstructions())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 86-101: Header guard and interface framing / 头文件保护与接口框架
```cpp
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const TargetInstrInfo *TII = ST->getInstrInfo();

#ifndef NDEBUG
  // Assert that we won't clobber any existing reads of v0 where we need to
  // insert copies.
  const TargetRegisterInfo *TRI = MRI.getTargetRegisterInfo();
  ReversePostOrderTraversal<MachineBasicBlock *> RPOT(&*MF.begin());
  for (MachineBasicBlock *MBB : RPOT) {
    bool V0Clobbered = false;
    for (MachineInstr &MI : *MBB) {
      assert(!(MI.readsRegister(RISCV::V0, TRI) && V0Clobbered) &&
             "Inserting a copy to v0 would clobber a read");
      if (MI.modifiesRegister(RISCV::V0, TRI))
        V0Clobbered = false;
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 102-111: Header guard and interface framing / 头文件保护与接口框架
```cpp
      if (any_of(MI.getDesc().operands(), isVMV0))
        V0Clobbered = true;
    }

    assert(!(V0Clobbered &&
             any_of(MBB->successors(),
                    [](auto *Succ) { return Succ->isLiveIn(RISCV::V0); })) &&
           "Clobbered a v0 used in a successor");
  }
#endif
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 112-121: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool MadeChange = false;
  SmallVector<MachineInstr *> DeadCopies;

  // For any instruction with a vmv0 operand, replace it with a copy to v0.
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      assert(count_if(MI.getDesc().operands(), isVMV0) < 2 &&
             "Expected only one or zero vmv0 operands");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 122-137: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      for (auto [OpNo, MCOI] : enumerate(MI.getDesc().operands())) {
        if (isVMV0(MCOI)) {
          MachineOperand &MO = MI.getOperand(OpNo);
          Register Src = MO.getReg();
          assert(MO.isUse() && MO.getSubReg() == RISCV::NoSubRegister &&
                 Src.isVirtual() && "vmv0 use in unexpected form");

          // Peek through a single copy to match what isel does.
          if (MachineInstr *SrcMI = MRI.getVRegDef(Src);
              SrcMI->isCopy() && SrcMI->getOperand(1).getReg().isVirtual() &&
              SrcMI->getOperand(1).getSubReg() == RISCV::NoSubRegister) {
            // Delete any dead copys to vmv0 to avoid allocating them.
            if (MRI.hasOneNonDBGUse(Src))
              DeadCopies.push_back(SrcMI);
            Src = SrcMI->getOperand(1).getReg();
          }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 138-147: Definitions and supporting logic / 定义与支撑逻辑
```cpp

          BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(RISCV::COPY), RISCV::V0)
              .addReg(Src);

          MO.setReg(RISCV::V0);
          MadeChange = true;
          break;
        }
      }
    }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 148-167: Type declaration for for / for 的类型声明
```cpp
  }

  for (MachineInstr *MI : DeadCopies)
    MI->eraseFromParent();

  if (!MadeChange)
    return false;

  // Now that any constraints requiring vmv0 are gone, eliminate any uses of
  // vmv0 by recomputing the reg class.
  // The only remaining uses should be around inline asm.
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      for (MachineOperand &MO : MI.uses()) {
        if (MO.isReg() && MO.getReg().isVirtual() &&
            MRI.getRegClass(MO.getReg()) == &RISCV::VMV0RegClass) {
          MRI.recomputeRegClass(MO.getReg());
          assert((MRI.getRegClass(MO.getReg()) != &RISCV::VMV0RegClass ||
                  MI.isInlineAsm() ||
                  MRI.getVRegDef(MO.getReg())->isInlineAsm()) &&
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 168-175: Definitions and supporting logic / 定义与支撑逻辑
```cpp
                 "Non-inline-asm use of vmv0 left behind");
        }
      }
    }
  }

  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/PostOrderIterator.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
