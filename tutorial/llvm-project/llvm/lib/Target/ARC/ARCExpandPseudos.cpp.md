# ARCExpandPseudos.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCExpandPseudos.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Expands target pseudo instructions into concrete instruction sequences.
- 目的（中文）: 将目标伪指令展开为实际机器指令序列。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCExpandPseudosPass - ARC expand pseudo loads -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass expands stores with large offsets into an appropriate sequence.
  10: //===----------------------------------------------------------------------===//
  11: 
  12: #include "ARC.h"
  13: #include "ARCInstrInfo.h"
  14: #include "ARCRegisterInfo.h"
  15: #include "ARCSubtarget.h"
  16: #include "MCTargetDesc/ARCInfo.h"
  17: #include "llvm/ADT/Statistic.h"
  18: #include "llvm/CodeGen/MachineFunctionPass.h"
  19: #include "llvm/CodeGen/MachineInstrBuilder.h"
  20: #include "llvm/CodeGen/MachineRegisterInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: 
  22: using namespace llvm;
  23: 
  24: #define DEBUG_TYPE "arc-expand-pseudos"
  25: 
  26: namespace {
  27: 
  28: class ARCExpandPseudos : public MachineFunctionPass {
  29: public:
  30:   static char ID;
  31:   ARCExpandPseudos() : MachineFunctionPass(ID) {}
  32: 
  33:   bool runOnMachineFunction(MachineFunction &Fn) override;
  34: 
  35:   StringRef getPassName() const override { return "ARC Expand Pseudos"; }
  36: 
  37: private:
  38:   void expandStore(MachineFunction &, MachineBasicBlock::iterator);
  39:   void expandCTLZ(MachineFunction &, MachineBasicBlock::iterator);
  40:   void expandCTTZ(MachineFunction &, MachineBasicBlock::iterator);
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as ARCExpandPseudos, which organize the target-specific behavior exposed by the file.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 ARCExpandPseudos 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41: 
  42:   const ARCInstrInfo *TII;
  43: };
  44: 
  45: char ARCExpandPseudos::ID = 0;
  46: 
  47: } // end anonymous namespace
  48: 
  49: static unsigned getMappedOp(unsigned PseudoOp) {
  50:   switch (PseudoOp) {
  51:   case ARC::ST_FAR:
  52:     return ARC::ST_rs9;
  53:   case ARC::STH_FAR:
  54:     return ARC::STH_rs9;
  55:   case ARC::STB_FAR:
  56:     return ARC::STB_rs9;
  57:   default:
  58:     llvm_unreachable("Unhandled pseudo op.");
  59:   }
  60: }
```

- EN: Function bodies or method definitions such as getMappedOp contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: getMappedOp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 61-80

```cpp
  61: 
  62: void ARCExpandPseudos::expandStore(MachineFunction &MF,
  63:                                    MachineBasicBlock::iterator SII) {
  64:   MachineInstr &SI = *SII;
  65:   Register AddrReg = MF.getRegInfo().createVirtualRegister(&ARC::GPR32RegClass);
  66:   Register AddOpc =
  67:       isUInt<6>(SI.getOperand(2).getImm()) ? ARC::ADD_rru6 : ARC::ADD_rrlimm;
  68:   BuildMI(*SI.getParent(), SI, SI.getDebugLoc(), TII->get(AddOpc), AddrReg)
  69:       .addReg(SI.getOperand(1).getReg())
  70:       .addImm(SI.getOperand(2).getImm());
  71:   BuildMI(*SI.getParent(), SI, SI.getDebugLoc(),
  72:           TII->get(getMappedOp(SI.getOpcode())))
  73:       .addReg(SI.getOperand(0).getReg())
  74:       .addReg(AddrReg)
  75:       .addImm(0);
  76:   SI.eraseFromParent();
  77: }
  78: 
  79: void ARCExpandPseudos::expandCTLZ(MachineFunction &MF,
  80:                                   MachineBasicBlock::iterator MII) {
```

- EN: Function bodies or method definitions such as expandStore, expandCTLZ contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandStore, expandCTLZ 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 81-100

```cpp
  81:   // Expand:
  82:   //	%R2<def> = CTLZ %R0, %STATUS<imp-def>
  83:   // To:
  84:   //	%R2<def> = FLS_f_rr %R0, %STATUS<imp-def>
  85:   //	%R2<def,tied1> = MOV_cc_ru6 %R2<tied0>, 32, pred:1, %STATUS<imp-use>
  86:   //	%R2<def,tied1> = RSUB_cc_rru6 %R2<tied0>, 31, pred:2, %STATUS<imp-use>
  87:   MachineInstr &MI = *MII;
  88:   const MachineOperand &Dest = MI.getOperand(0);
  89:   const MachineOperand &Src = MI.getOperand(1);
  90:   Register Ra = MF.getRegInfo().createVirtualRegister(&ARC::GPR32RegClass);
  91:   Register Rb = MF.getRegInfo().createVirtualRegister(&ARC::GPR32RegClass);
  92: 
  93:   BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(ARC::FLS_f_rr), Ra)
  94:       .add(Src);
  95:   BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(ARC::MOV_cc_ru6), Rb)
  96:       .addImm(32)
  97:       .addImm(ARCCC::EQ)
  98:       .addReg(Ra);
  99:   BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(ARC::RSUB_cc_rru6))
 100:       .add(Dest)
```

- EN: This range continues the implementation of the backend component described by ARCExpandPseudos.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 101-120

```cpp
 101:       .addImm(31)
 102:       .addImm(ARCCC::NE)
 103:       .addReg(Rb);
 104: 
 105:   MI.eraseFromParent();
 106: }
 107: 
 108: void ARCExpandPseudos::expandCTTZ(MachineFunction &MF,
 109:                                   MachineBasicBlock::iterator MII) {
 110:   // Expand:
 111:   //	%R0<def> = CTTZ %R0<kill>, %STATUS<imp-def>
 112:   // To:
 113:   //	%R0<def> = FFS_f_rr %R0<kill>, %STATUS<imp-def>
 114:   //	%R0<def,tied1> = MOVcc_ru6 %R0<tied0>, 32, pred:1, %STATUS<imp-use>
 115:   MachineInstr &MI = *MII;
 116:   const MachineOperand &Dest = MI.getOperand(0);
 117:   const MachineOperand &Src = MI.getOperand(1);
 118:   Register R = MF.getRegInfo().createVirtualRegister(&ARC::GPR32RegClass);
 119: 
 120:   BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(ARC::FFS_f_rr), R)
```

- EN: Function bodies or method definitions such as expandCTTZ contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandCTTZ 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-140

```cpp
 121:       .add(Src);
 122:   BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(ARC::MOV_cc_ru6))
 123:       .add(Dest)
 124:       .addImm(32)
 125:       .addImm(ARCCC::EQ)
 126:       .addReg(R);
 127: 
 128:   MI.eraseFromParent();
 129: }
 130: 
 131: bool ARCExpandPseudos::runOnMachineFunction(MachineFunction &MF) {
 132:   const ARCSubtarget *STI = &MF.getSubtarget<ARCSubtarget>();
 133:   TII = STI->getInstrInfo();
 134:   bool Expanded = false;
 135:   for (auto &MBB : MF) {
 136:     MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
 137:     while (MBBI != E) {
 138:       MachineBasicBlock::iterator NMBBI = std::next(MBBI);
 139:       switch (MBBI->getOpcode()) {
 140:       case ARC::ST_FAR:
```

- EN: Function bodies or method definitions such as runOnMachineFunction contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: runOnMachineFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 141-160

```cpp
 141:       case ARC::STH_FAR:
 142:       case ARC::STB_FAR:
 143:         expandStore(MF, MBBI);
 144:         Expanded = true;
 145:         break;
 146:       case ARC::CTLZ:
 147:         expandCTLZ(MF, MBBI);
 148:         Expanded = true;
 149:         break;
 150:       case ARC::CTTZ:
 151:         expandCTTZ(MF, MBBI);
 152:         Expanded = true;
 153:         break;
 154:       default:
 155:         break;
 156:       }
 157:       MBBI = NMBBI;
 158:     }
 159:   }
 160:   return Expanded;
```

- EN: This range continues the implementation of the backend component described by ARCExpandPseudos.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 161-165

```cpp
 161: }
 162: 
 163: FunctionPass *llvm::createARCExpandPseudosPass() {
 164:   return new ARCExpandPseudos();
 165: }
```

- EN: This range continues the implementation of the backend component described by ARCExpandPseudos.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARC.h`, `ARCInstrInfo.h`, `ARCRegisterInfo.h`, `ARCSubtarget.h`, `MCTargetDesc/ARCInfo.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
