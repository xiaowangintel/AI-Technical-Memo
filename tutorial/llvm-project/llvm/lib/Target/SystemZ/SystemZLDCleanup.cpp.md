# SystemZLDCleanup.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZLDCleanup.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZLDCleanup.cpp - Clean up local-dynamic TLS accesses --------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass combines multiple accesses to local-dynamic TLS variables so that
  10: // the TLS base address for the module is only fetched once per execution path
  11: // through the function.
  12: //
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "SystemZMachineFunctionInfo.h"
  16: #include "SystemZTargetMachine.h"
  17: #include "llvm/CodeGen/MachineDominators.h"
  18: #include "llvm/CodeGen/MachineFunctionPass.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMachineFunctionInfo.h`, `SystemZTargetMachine.h`, `MachineDominators.h`, `MachineFunctionPass.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMachineFunctionInfo.h`, `SystemZTargetMachine.h`, `MachineDominators.h`, `MachineFunctionPass.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/CodeGen/MachineInstrBuilder.h"
  20: #include "llvm/CodeGen/MachineRegisterInfo.h"
  21: #include "llvm/CodeGen/TargetInstrInfo.h"
  22: #include "llvm/CodeGen/TargetRegisterInfo.h"
  23: 
  24: using namespace llvm;
  25: 
  26: namespace {
  27: 
  28: class SystemZLDCleanup : public MachineFunctionPass {
  29: public:
  30:   static char ID;
  31:   SystemZLDCleanup() : MachineFunctionPass(ID), TII(nullptr), MF(nullptr) {}
  32: 
  33:   bool runOnMachineFunction(MachineFunction &MF) override;
  34:   void getAnalysisUsage(AnalysisUsage &AU) const override;
  35: 
  36: private:
```
- **EN**: It imports dependencies such as `MachineInstrBuilder.h`, `MachineRegisterInfo.h`, `TargetInstrInfo.h`, `TargetRegisterInfo.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZLDCleanup`. The range implements or declares functions including `SystemZLDCleanup`.
- **CN**: 它引入了 `MachineInstrBuilder.h`, `MachineRegisterInfo.h`, `TargetInstrInfo.h`, `TargetRegisterInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZLDCleanup` 等 TableGen 记录。 这一段实现或声明了 `SystemZLDCleanup` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   bool VisitNode(MachineDomTreeNode *Node, unsigned TLSBaseAddrReg);
  38:   MachineInstr *ReplaceTLSCall(MachineInstr *I, unsigned TLSBaseAddrReg);
  39:   MachineInstr *SetRegister(MachineInstr *I, unsigned *TLSBaseAddrReg);
  40: 
  41:   const SystemZInstrInfo *TII;
  42:   MachineFunction *MF;
  43: };
  44: 
  45: char SystemZLDCleanup::ID = 0;
  46: 
  47: } // end anonymous namespace
  48: 
  49: INITIALIZE_PASS(SystemZLDCleanup, "systemz-ld-cleanup",
  50:                 "SystemZ Local Dynamic TLS Access Clean-up", false, false)
  51: 
  52: FunctionPass *llvm::createSystemZLDCleanupPass(SystemZTargetMachine &TM) {
  53:   return new SystemZLDCleanup();
  54: }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56: void SystemZLDCleanup::getAnalysisUsage(AnalysisUsage &AU) const {
  57:   AU.setPreservesCFG();
  58:   AU.addRequired<MachineDominatorTreeWrapperPass>();
  59:   MachineFunctionPass::getAnalysisUsage(AU);
  60: }
  61: 
  62: bool SystemZLDCleanup::runOnMachineFunction(MachineFunction &F) {
  63:   if (skipFunction(F.getFunction()))
  64:     return false;
  65: 
  66:   TII = F.getSubtarget<SystemZSubtarget>().getInstrInfo();
  67:   MF = &F;
  68: 
  69:   SystemZMachineFunctionInfo* MFI = F.getInfo<SystemZMachineFunctionInfo>();
  70:   if (MFI->getNumLocalDynamicTLSAccesses() < 2) {
  71:     // No point folding accesses if there isn't at least two.
  72:     return false;
```
- **EN**: The range implements or declares functions including `SystemZLDCleanup::getAnalysisUsage`, `SystemZLDCleanup::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZLDCleanup::getAnalysisUsage`, `SystemZLDCleanup::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   }
  74: 
  75:   MachineDominatorTree *DT =
  76:       &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  77:   return VisitNode(DT->getRootNode(), 0);
  78: }
  79: 
  80: // Visit the dominator subtree rooted at Node in pre-order.
  81: // If TLSBaseAddrReg is non-null, then use that to replace any
  82: // TLS_LDCALL instructions. Otherwise, create the register
  83: // when the first such instruction is seen, and then use it
  84: // as we encounter more instructions.
  85: bool SystemZLDCleanup::VisitNode(MachineDomTreeNode *Node,
  86:                                  unsigned TLSBaseAddrReg) {
  87:   MachineBasicBlock *BB = Node->getBlock();
  88:   bool Changed = false;
  89: 
  90:   // Traverse the current block.
```
- **EN**: The range implements or declares functions including `SystemZLDCleanup::VisitNode`.
- **CN**: 这一段实现或声明了 `SystemZLDCleanup::VisitNode` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   for (auto I = BB->begin(), E = BB->end(); I != E; ++I) {
  92:     switch (I->getOpcode()) {
  93:       case SystemZ::TLS_LDCALL:
  94:         if (TLSBaseAddrReg)
  95:           I = ReplaceTLSCall(&*I, TLSBaseAddrReg);
  96:         else
  97:           I = SetRegister(&*I, &TLSBaseAddrReg);
  98:         Changed = true;
  99:         break;
 100:       default:
 101:         break;
 102:     }
 103:   }
 104: 
 105:   // Visit the children of this block in the dominator tree.
 106:   for (MachineDomTreeNode *Child : *Node)
 107:     Changed |= VisitNode(Child, TLSBaseAddrReg);
 108: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   return Changed;
 110: }
 111: 
 112: // Replace the TLS_LDCALL instruction I with a copy from TLSBaseAddrReg,
 113: // returning the new instruction.
 114: MachineInstr *SystemZLDCleanup::ReplaceTLSCall(MachineInstr *I,
 115:                                                unsigned TLSBaseAddrReg) {
 116:   // Insert a Copy from TLSBaseAddrReg to R2.
 117:   MachineInstr *Copy = BuildMI(*I->getParent(), I, I->getDebugLoc(),
 118:                                TII->get(TargetOpcode::COPY), SystemZ::R2D)
 119:                                .addReg(TLSBaseAddrReg);
 120: 
 121:   // Erase the TLS_LDCALL instruction.
 122:   I->eraseFromParent();
 123: 
 124:   return Copy;
 125: }
 126: 
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-143 / 第 127-143 行
```cpp
 127: // Create a virtual register in *TLSBaseAddrReg, and populate it by
 128: // inserting a copy instruction after I. Returns the new instruction.
 129: MachineInstr *SystemZLDCleanup::SetRegister(MachineInstr *I,
 130:                                             unsigned *TLSBaseAddrReg) {
 131:   // Create a virtual register for the TLS base address.
 132:   MachineRegisterInfo &RegInfo = MF->getRegInfo();
 133:   *TLSBaseAddrReg = RegInfo.createVirtualRegister(&SystemZ::GR64BitRegClass);
 134: 
 135:   // Insert a copy from R2 to TLSBaseAddrReg.
 136:   MachineInstr *Next = I->getNextNode();
 137:   MachineInstr *Copy = BuildMI(*I->getParent(), Next, I->getDebugLoc(),
 138:                                TII->get(TargetOpcode::COPY), *TLSBaseAddrReg)
 139:                                .addReg(SystemZ::R2D);
 140: 
 141:   return Copy;
 142: }
 143: 
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
- `SystemZMachineFunctionInfo.h`
- `SystemZTargetMachine.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `llvm/CodeGen/TargetRegisterInfo.h`
