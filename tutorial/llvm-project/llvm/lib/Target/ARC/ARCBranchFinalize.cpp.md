# ARCBranchFinalize.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCBranchFinalize.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements a late pass that finalizes or relaxes branch instructions after layout decisions.
- 目的（中文）: 实现布局确定后的分支最终化/松弛 Pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCBranchFinalize.cpp - ARC conditional branches ---------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass takes existing conditional branches and expands them into longer
  10: // range conditional branches.
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "ARCInstrInfo.h"
  14: #include "ARCTargetMachine.h"
  15: #include "MCTargetDesc/ARCInfo.h"
  16: #include "llvm/CodeGen/MachineFunctionPass.h"
  17: #include "llvm/CodeGen/MachineInstrBuilder.h"
  18: #include "llvm/CodeGen/MachineRegisterInfo.h"
  19: #include "llvm/CodeGen/Passes.h"
  20: #include "llvm/CodeGen/TargetInstrInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/InitializePasses.h"
  22: #include "llvm/Support/Debug.h"
  23: #include <vector>
  24: 
  25: #define DEBUG_TYPE "arc-branch-finalize"
  26: 
  27: using namespace llvm;
  28: 
  29: namespace llvm {
  30: 
  31: void initializeARCBranchFinalizePass(PassRegistry &Registry);
  32: FunctionPass *createARCBranchFinalizePass();
  33: 
  34: } // end namespace llvm
  35: 
  36: namespace {
  37: 
  38: class ARCBranchFinalize : public MachineFunctionPass {
  39: public:
  40:   static char ID;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as ARCBranchFinalize, which organize the target-specific behavior exposed by the file.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 ARCBranchFinalize 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41: 
  42:   ARCBranchFinalize() : MachineFunctionPass(ID) {
  43:     initializeARCBranchFinalizePass(*PassRegistry::getPassRegistry());
  44:   }
  45: 
  46:   StringRef getPassName() const override {
  47:     return "ARC Branch Finalization Pass";
  48:   }
  49: 
  50:   bool runOnMachineFunction(MachineFunction &MF) override;
  51:   void replaceWithBRcc(MachineInstr *MI) const;
  52:   void replaceWithCmpBcc(MachineInstr *MI) const;
  53: 
  54: private:
  55:   const ARCInstrInfo *TII{nullptr};
  56: };
  57: 
  58: char ARCBranchFinalize::ID = 0;
  59: 
  60: } // end anonymous namespace
```

- EN: This range continues the implementation of the backend component described by ARCBranchFinalize.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61: 
  62: INITIALIZE_PASS_BEGIN(ARCBranchFinalize, "arc-branch-finalize",
  63:                       "ARC finalize branches", false, false)
  64: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
  65: INITIALIZE_PASS_END(ARCBranchFinalize, "arc-branch-finalize",
  66:                     "ARC finalize branches", false, false)
  67: 
  68: // BRcc has 6 supported condition codes, which differ from the 16
  69: // condition codes supported in the predicated instructions:
  70: // EQ -- 000
  71: // NE -- 001
  72: // LT -- 010
  73: // GE -- 011
  74: // LO -- 100
  75: // HS -- 101
  76: static unsigned getCCForBRcc(unsigned CC) {
  77:   switch (CC) {
  78:   case ARCCC::EQ:
  79:     return 0;
  80:   case ARCCC::NE:
```

- EN: Function bodies or method definitions such as getCCForBRcc contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: getCCForBRcc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 81-100

```cpp
  81:     return 1;
  82:   case ARCCC::LT:
  83:     return 2;
  84:   case ARCCC::GE:
  85:     return 3;
  86:   case ARCCC::LO:
  87:     return 4;
  88:   case ARCCC::HS:
  89:     return 5;
  90:   default:
  91:     return -1U;
  92:   }
  93: }
  94: 
  95: static bool isBRccPseudo(MachineInstr *MI) {
  96:   return !(MI->getOpcode() != ARC::BRcc_rr_p &&
  97:            MI->getOpcode() != ARC::BRcc_ru6_p);
  98: }
  99: 
 100: static unsigned getBRccForPseudo(MachineInstr *MI) {
```

- EN: Function bodies or method definitions such as isBRccPseudo, getBRccForPseudo contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: isBRccPseudo, getBRccForPseudo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 101-120

```cpp
 101:   assert(isBRccPseudo(MI) && "Can't get BRcc for wrong instruction.");
 102:   if (MI->getOpcode() == ARC::BRcc_rr_p)
 103:     return ARC::BRcc_rr;
 104:   return ARC::BRcc_ru6;
 105: }
 106: 
 107: static unsigned getCmpForPseudo(MachineInstr *MI) {
 108:   assert(isBRccPseudo(MI) && "Can't get BRcc for wrong instruction.");
 109:   if (MI->getOpcode() == ARC::BRcc_rr_p)
 110:     return ARC::CMP_rr;
 111:   return ARC::CMP_ru6;
 112: }
 113: 
 114: void ARCBranchFinalize::replaceWithBRcc(MachineInstr *MI) const {
 115:   LLVM_DEBUG(dbgs() << "Replacing pseudo branch with BRcc\n");
 116:   unsigned CC = getCCForBRcc(MI->getOperand(3).getImm());
 117:   if (CC != -1U) {
 118:     BuildMI(*MI->getParent(), MI, MI->getDebugLoc(),
 119:             TII->get(getBRccForPseudo(MI)))
 120:         .addMBB(MI->getOperand(0).getMBB())
```

- EN: Function bodies or method definitions such as getCmpForPseudo, replaceWithBRcc contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getCmpForPseudo, replaceWithBRcc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:         .addReg(MI->getOperand(1).getReg())
 122:         .add(MI->getOperand(2))
 123:         .addImm(getCCForBRcc(MI->getOperand(3).getImm()));
 124:     MI->eraseFromParent();
 125:   } else {
 126:     replaceWithCmpBcc(MI);
 127:   }
 128: }
 129: 
 130: void ARCBranchFinalize::replaceWithCmpBcc(MachineInstr *MI) const {
 131:   LLVM_DEBUG(dbgs() << "Branch: " << *MI << "\n");
 132:   LLVM_DEBUG(dbgs() << "Replacing pseudo branch with Cmp + Bcc\n");
 133:   BuildMI(*MI->getParent(), MI, MI->getDebugLoc(),
 134:           TII->get(getCmpForPseudo(MI)))
 135:       .addReg(MI->getOperand(1).getReg())
 136:       .add(MI->getOperand(2));
 137:   BuildMI(*MI->getParent(), MI, MI->getDebugLoc(), TII->get(ARC::Bcc))
 138:       .addMBB(MI->getOperand(0).getMBB())
 139:       .addImm(MI->getOperand(3).getImm());
 140:   MI->eraseFromParent();
```

- EN: Function bodies or method definitions such as replaceWithCmpBcc contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: replaceWithCmpBcc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 141-160

```cpp
 141: }
 142: 
 143: bool ARCBranchFinalize::runOnMachineFunction(MachineFunction &MF) {
 144:   LLVM_DEBUG(dbgs() << "Running ARC Branch Finalize on " << MF.getName()
 145:                     << "\n");
 146:   std::vector<MachineInstr *> Branches;
 147:   bool Changed = false;
 148:   unsigned MaxSize = 0;
 149:   TII = MF.getSubtarget<ARCSubtarget>().getInstrInfo();
 150:   std::map<MachineBasicBlock *, unsigned> BlockToPCMap;
 151:   std::vector<std::pair<MachineInstr *, unsigned>> BranchToPCList;
 152:   unsigned PC = 0;
 153: 
 154:   for (auto &MBB : MF) {
 155:     BlockToPCMap.insert(std::make_pair(&MBB, PC));
 156:     for (auto &MI : MBB) {
 157:       unsigned Size = TII->getInstSizeInBytes(MI);
 158:       if (Size > 8 || Size == 0) {
 159:         LLVM_DEBUG(dbgs() << "Unknown (or size 0) size for: " << MI << "\n");
 160:       } else {
```

- EN: Function bodies or method definitions such as runOnMachineFunction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: runOnMachineFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:         MaxSize += Size;
 162:       }
 163:       if (MI.isBranch()) {
 164:         Branches.push_back(&MI);
 165:         BranchToPCList.emplace_back(&MI, PC);
 166:       }
 167:       PC += Size;
 168:     }
 169:   }
 170:   for (auto P : BranchToPCList) {
 171:     if (isBRccPseudo(P.first))
 172:       isInt<9>(MaxSize) ? replaceWithBRcc(P.first) : replaceWithCmpBcc(P.first);
 173:   }
 174: 
 175:   LLVM_DEBUG(dbgs() << "Estimated function size for " << MF.getName() << ": "
 176:                     << MaxSize << "\n");
 177: 
 178:   return Changed;
 179: }
 180: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-183

```cpp
 181: FunctionPass *llvm::createARCBranchFinalizePass() {
 182:   return new ARCBranchFinalize();
 183: }
```

- EN: This range continues the implementation of the backend component described by ARCBranchFinalize.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCInstrInfo.h`, `ARCTargetMachine.h`, `MCTargetDesc/ARCInfo.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetInstrInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
