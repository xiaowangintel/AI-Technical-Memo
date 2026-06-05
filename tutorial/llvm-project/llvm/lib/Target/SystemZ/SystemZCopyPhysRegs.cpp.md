# SystemZCopyPhysRegs.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZCopyPhysRegs.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===---------- SystemZPhysRegCopy.cpp - Handle phys reg copies -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass makes sure that a COPY of a physical register will be
  10: // implementable after register allocation in copyPhysReg() (this could be
  11: // done in EmitInstrWithCustomInserter() instead if COPY instructions would
  12: // be passed to it).
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: #include "SystemZTargetMachine.h"
  17: #include "llvm/CodeGen/MachineDominators.h"
  18: #include "llvm/CodeGen/MachineFunctionPass.h"
  19: #include "llvm/CodeGen/MachineInstrBuilder.h"
  20: #include "llvm/CodeGen/MachineRegisterInfo.h"
  21: #include "llvm/CodeGen/TargetInstrInfo.h"
  22: #include "llvm/CodeGen/TargetRegisterInfo.h"
  23: 
  24: using namespace llvm;
```
- **EN**: It imports dependencies such as `SystemZTargetMachine.h`, `MachineDominators.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h`, `MachineRegisterInfo.h`, `TargetInstrInfo.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `SystemZTargetMachine.h`, `MachineDominators.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h`, `MachineRegisterInfo.h`, `TargetInstrInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 25-36 / 第 25-36 行
```cpp
  25: 
  26: namespace {
  27: 
  28: class SystemZCopyPhysRegs : public MachineFunctionPass {
  29: public:
  30:   static char ID;
  31:   SystemZCopyPhysRegs() : MachineFunctionPass(ID), TII(nullptr), MRI(nullptr) {}
  32: 
  33:   bool runOnMachineFunction(MachineFunction &MF) override;
  34:   void getAnalysisUsage(AnalysisUsage &AU) const override;
  35: 
  36: private:
```
- **EN**: This block declares or refines TableGen records such as `SystemZCopyPhysRegs`. The range implements or declares functions including `SystemZCopyPhysRegs`.
- **CN**: 该代码块声明或细化了 `SystemZCopyPhysRegs` 等 TableGen 记录。 这一段实现或声明了 `SystemZCopyPhysRegs` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37: 
  38:   bool visitMBB(MachineBasicBlock &MBB);
  39: 
  40:   const SystemZInstrInfo *TII;
  41:   MachineRegisterInfo *MRI;
  42: };
  43: 
  44: char SystemZCopyPhysRegs::ID = 0;
  45: 
  46: } // end anonymous namespace
  47: 
  48: INITIALIZE_PASS(SystemZCopyPhysRegs, "systemz-copy-physregs",
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 49-60 / 第 49-60 行
```cpp
  49:                 "SystemZ Copy Physregs", false, false)
  50: 
  51: FunctionPass *llvm::createSystemZCopyPhysRegsPass(SystemZTargetMachine &TM) {
  52:   return new SystemZCopyPhysRegs();
  53: }
  54: 
  55: void SystemZCopyPhysRegs::getAnalysisUsage(AnalysisUsage &AU) const {
  56:   AU.setPreservesCFG();
  57:   MachineFunctionPass::getAnalysisUsage(AU);
  58: }
  59: 
  60: bool SystemZCopyPhysRegs::visitMBB(MachineBasicBlock &MBB) {
```
- **EN**: The range implements or declares functions including `SystemZCopyPhysRegs::getAnalysisUsage`, `SystemZCopyPhysRegs::visitMBB`.
- **CN**: 这一段实现或声明了 `SystemZCopyPhysRegs::getAnalysisUsage`, `SystemZCopyPhysRegs::visitMBB` 等函数。

### Lines 61-72 / 第 61-72 行
```cpp
  61:   bool Modified = false;
  62: 
  63:   // Certain special registers can only be copied from a subset of the
  64:   // default register class of the type. It is therefore necessary to create
  65:   // the target copy instructions before regalloc instead of in copyPhysReg().
  66:   for (MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  67:        MBBI != E; ) {
  68:     MachineInstr *MI = &*MBBI++;
  69:     if (!MI->isCopy())
  70:       continue;
  71: 
  72:     DebugLoc DL = MI->getDebugLoc();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 73-84 / 第 73-84 行
```cpp
  73:     Register SrcReg = MI->getOperand(1).getReg();
  74:     Register DstReg = MI->getOperand(0).getReg();
  75: 
  76:     if (DstReg.isVirtual() &&
  77:         (SrcReg == SystemZ::CC || SystemZ::AR32BitRegClass.contains(SrcReg))) {
  78:       Register Tmp = MRI->createVirtualRegister(&SystemZ::GR32BitRegClass);
  79:       if (SrcReg == SystemZ::CC)
  80:         BuildMI(MBB, MI, DL, TII->get(SystemZ::IPM), Tmp);
  81:       else
  82:         BuildMI(MBB, MI, DL, TII->get(SystemZ::EAR), Tmp).addReg(SrcReg);
  83:       MI->getOperand(1).setReg(Tmp);
  84:       Modified = true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-96 / 第 85-96 行
```cpp
  85:     }
  86:     else if (SrcReg.isVirtual() &&
  87:              SystemZ::AR32BitRegClass.contains(DstReg)) {
  88:       Register Tmp = MRI->createVirtualRegister(&SystemZ::GR32BitRegClass);
  89:       MI->getOperand(0).setReg(Tmp);
  90:       MachineInstr *NMI =
  91:           BuildMI(MBB, MBBI, DL, TII->get(SystemZ::SAR), DstReg).addReg(Tmp);
  92:       // SAR now writes the final value to DstReg, so update debug values.
  93:       MBB.getParent()->substituteDebugValuesForInst(*MI, *NMI);
  94:       Modified = true;
  95:     }
  96:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 97-108 / 第 97-108 行
```cpp
  97: 
  98:   return Modified;
  99: }
 100: 
 101: bool SystemZCopyPhysRegs::runOnMachineFunction(MachineFunction &F) {
 102:   TII = F.getSubtarget<SystemZSubtarget>().getInstrInfo();
 103:   MRI = &F.getRegInfo();
 104: 
 105:   bool Modified = false;
 106:   for (auto &MBB : F)
 107:     Modified |= visitMBB(MBB);
 108: 
```
- **EN**: The range implements or declares functions including `SystemZCopyPhysRegs::runOnMachineFunction`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZCopyPhysRegs::runOnMachineFunction` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-111 / 第 109-111 行
```cpp
 109:   return Modified;
 110: }
 111: 
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZTargetMachine.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `llvm/CodeGen/TargetRegisterInfo.h`
