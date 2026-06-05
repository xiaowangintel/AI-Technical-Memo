# LanaiDelaySlotFiller.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiDelaySlotFiller.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiDelaySlotFiller.cpp - Lanai delay slot filler ----------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Simple pass to fill delay slots with useful instructions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "Lanai.h"
  14: #include "LanaiTargetMachine.h"
  15: #include "llvm/ADT/SmallSet.h"
  16: #include "llvm/ADT/Statistic.h"
  17: #include "llvm/CodeGen/MachineFunctionPass.h"
  18: #include "llvm/CodeGen/MachineInstrBuilder.h"
  19: #include "llvm/CodeGen/TargetInstrInfo.h"
  20: #include "llvm/Support/CommandLine.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: 
  22: using namespace llvm;
  23: 
  24: #define DEBUG_TYPE "delay-slot-filler"
  25: 
  26: STATISTIC(FilledSlots, "Number of delay slots filled");
  27: 
  28: static cl::opt<bool>
  29:     NopDelaySlotFiller("lanai-nop-delay-filler", cl::init(false),
  30:                        cl::desc("Fill Lanai delay slots with NOPs."),
  31:                        cl::Hidden);
  32: 
  33: namespace {
  34: struct Filler : public MachineFunctionPass {
  35:   // Target machine description which we query for reg. names, data
  36:   // layout, etc.
  37:   const TargetInstrInfo *TII;
  38:   const TargetRegisterInfo *TRI;
  39:   MachineBasicBlock::instr_iterator LastFiller;
  40: 
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as Filler, which organize the target-specific behavior exposed by the file.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 Filler 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41:   static char ID;
  42:   explicit Filler() : MachineFunctionPass(ID) {}
  43: 
  44:   StringRef getPassName() const override { return "Lanai Delay Slot Filler"; }
  45: 
  46:   bool runOnMachineBasicBlock(MachineBasicBlock &MBB);
  47: 
  48:   bool runOnMachineFunction(MachineFunction &MF) override {
  49:     const LanaiSubtarget &Subtarget = MF.getSubtarget<LanaiSubtarget>();
  50:     TII = Subtarget.getInstrInfo();
  51:     TRI = Subtarget.getRegisterInfo();
  52: 
  53:     bool Changed = false;
  54:     for (MachineBasicBlock &MBB : MF)
  55:       Changed |= runOnMachineBasicBlock(MBB);
  56:     return Changed;
  57:   }
  58: 
  59:   MachineFunctionProperties getRequiredProperties() const override {
  60:     return MachineFunctionProperties().setNoVRegs();
```

- EN: Function bodies or method definitions such as Filler contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: Filler 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 61-80

```cpp
  61:   }
  62: 
  63:   void insertDefsUses(MachineBasicBlock::instr_iterator MI,
  64:                       SmallSet<unsigned, 32> &RegDefs,
  65:                       SmallSet<unsigned, 32> &RegUses);
  66: 
  67:   bool isRegInSet(SmallSet<unsigned, 32> &RegSet, unsigned Reg);
  68: 
  69:   bool delayHasHazard(MachineBasicBlock::instr_iterator MI, bool &SawLoad,
  70:                       bool &SawStore, SmallSet<unsigned, 32> &RegDefs,
  71:                       SmallSet<unsigned, 32> &RegUses);
  72: 
  73:   bool findDelayInstr(MachineBasicBlock &MBB,
  74:                       MachineBasicBlock::instr_iterator Slot,
  75:                       MachineBasicBlock::instr_iterator &Filler);
  76: };
  77: char Filler::ID = 0;
  78: } // end of anonymous namespace
  79: 
  80: // createLanaiDelaySlotFillerPass - Returns a pass that fills in delay
```

- EN: This range continues the implementation of the backend component described by LanaiDelaySlotFiller.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81: // slots in Lanai MachineFunctions
  82: FunctionPass *
  83: llvm::createLanaiDelaySlotFillerPass(const LanaiTargetMachine & /*tm*/) {
  84:   return new Filler();
  85: }
  86: 
  87: // runOnMachineBasicBlock - Fill in delay slots for the given basic block.
  88: // There is one or two delay slot per delayed instruction.
  89: bool Filler::runOnMachineBasicBlock(MachineBasicBlock &MBB) {
  90:   bool Changed = false;
  91:   LastFiller = MBB.instr_end();
  92: 
  93:   for (MachineBasicBlock::instr_iterator I = MBB.instr_begin();
  94:        I != MBB.instr_end(); ++I) {
  95:     if (I->getDesc().hasDelaySlot()) {
  96:       MachineBasicBlock::instr_iterator InstrWithSlot = I;
  97:       MachineBasicBlock::instr_iterator J = I;
  98: 
  99:       // Treat RET specially as it is only instruction with 2 delay slots
 100:       // generated while all others generated have 1 delay slot.
```

- EN: Function bodies or method definitions such as createLanaiDelaySlotFillerPass, runOnMachineBasicBlock contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: createLanaiDelaySlotFillerPass, runOnMachineBasicBlock 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:       if (I->getOpcode() == Lanai::RET) {
 102:         // RET is generated as part of epilogue generation and hence we know
 103:         // what the two instructions preceding it are and that it is safe to
 104:         // insert RET above them.
 105:         MachineBasicBlock::reverse_instr_iterator RI = ++I.getReverse();
 106:         assert(RI->getOpcode() == Lanai::LDW_RI && RI->getOperand(0).isReg() &&
 107:                RI->getOperand(0).getReg() == Lanai::FP &&
 108:                RI->getOperand(1).isReg() &&
 109:                RI->getOperand(1).getReg() == Lanai::FP &&
 110:                RI->getOperand(2).isImm() && RI->getOperand(2).getImm() == -8);
 111:         ++RI;
 112:         assert(RI->getOpcode() == Lanai::ADD_I_LO &&
 113:                RI->getOperand(0).isReg() &&
 114:                RI->getOperand(0).getReg() == Lanai::SP &&
 115:                RI->getOperand(1).isReg() &&
 116:                RI->getOperand(1).getReg() == Lanai::FP);
 117:         MachineBasicBlock::instr_iterator FI = RI.getReverse();
 118:         MBB.splice(std::next(I), &MBB, FI, I);
 119:         FilledSlots += 2;
 120:       } else {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:         if (!NopDelaySlotFiller && findDelayInstr(MBB, I, J)) {
 122:           MBB.splice(std::next(I), &MBB, J);
 123:         } else {
 124:           BuildMI(MBB, std::next(I), DebugLoc(), TII->get(Lanai::NOP));
 125:         }
 126:         ++FilledSlots;
 127:       }
 128: 
 129:       Changed = true;
 130:       // Record the filler instruction that filled the delay slot.
 131:       // The instruction after it will be visited in the next iteration.
 132:       LastFiller = ++I;
 133: 
 134:       // Bundle the delay slot filler to InstrWithSlot so that the machine
 135:       // verifier doesn't expect this instruction to be a terminator.
 136:       MIBundleBuilder(MBB, InstrWithSlot, std::next(LastFiller));
 137:     }
 138:   }
 139:   return Changed;
 140: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141: 
 142: bool Filler::findDelayInstr(MachineBasicBlock &MBB,
 143:                             MachineBasicBlock::instr_iterator Slot,
 144:                             MachineBasicBlock::instr_iterator &Filler) {
 145:   SmallSet<unsigned, 32> RegDefs;
 146:   SmallSet<unsigned, 32> RegUses;
 147: 
 148:   insertDefsUses(Slot, RegDefs, RegUses);
 149: 
 150:   bool SawLoad = false;
 151:   bool SawStore = false;
 152: 
 153:   for (MachineBasicBlock::reverse_instr_iterator I = ++Slot.getReverse();
 154:        I != MBB.instr_rend(); ++I) {
 155:     // skip debug value
 156:     if (I->isDebugInstr())
 157:       continue;
 158: 
 159:     // Convert to forward iterator.
 160:     MachineBasicBlock::instr_iterator FI = I.getReverse();
```

- EN: Function bodies or method definitions such as findDelayInstr contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: findDelayInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161: 
 162:     if (I->hasUnmodeledSideEffects() || I->isInlineAsm() || I->isLabel() ||
 163:         FI == LastFiller || I->isPseudo())
 164:       break;
 165: 
 166:     if (delayHasHazard(FI, SawLoad, SawStore, RegDefs, RegUses)) {
 167:       insertDefsUses(FI, RegDefs, RegUses);
 168:       continue;
 169:     }
 170:     Filler = FI;
 171:     return true;
 172:   }
 173:   return false;
 174: }
 175: 
 176: bool Filler::delayHasHazard(MachineBasicBlock::instr_iterator MI, bool &SawLoad,
 177:                             bool &SawStore, SmallSet<unsigned, 32> &RegDefs,
 178:                             SmallSet<unsigned, 32> &RegUses) {
 179:   if (MI->isImplicitDef() || MI->isKill())
 180:     return true;
```

- EN: Function bodies or method definitions such as delayHasHazard contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: delayHasHazard 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181: 
 182:   // Loads or stores cannot be moved past a store to the delay slot
 183:   // and stores cannot be moved past a load.
 184:   if (MI->mayLoad()) {
 185:     if (SawStore)
 186:       return true;
 187:     SawLoad = true;
 188:   }
 189: 
 190:   if (MI->mayStore()) {
 191:     if (SawStore)
 192:       return true;
 193:     SawStore = true;
 194:     if (SawLoad)
 195:       return true;
 196:   }
 197: 
 198:   assert((!MI->isCall() && !MI->isReturn()) &&
 199:          "Cannot put calls or returns in delay slot.");
 200: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201:   for (const MachineOperand &MO : MI->operands()) {
 202:     unsigned Reg;
 203: 
 204:     if (!MO.isReg() || !(Reg = MO.getReg()))
 205:       continue; // skip
 206: 
 207:     if (MO.isDef()) {
 208:       // check whether Reg is defined or used before delay slot.
 209:       if (isRegInSet(RegDefs, Reg) || isRegInSet(RegUses, Reg))
 210:         return true;
 211:     }
 212:     if (MO.isUse()) {
 213:       // check whether Reg is defined before delay slot.
 214:       if (isRegInSet(RegDefs, Reg))
 215:         return true;
 216:     }
 217:   }
 218:   return false;
 219: }
 220: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221: // Insert Defs and Uses of MI into the sets RegDefs and RegUses.
 222: void Filler::insertDefsUses(MachineBasicBlock::instr_iterator MI,
 223:                             SmallSet<unsigned, 32> &RegDefs,
 224:                             SmallSet<unsigned, 32> &RegUses) {
 225:   // If MI is a call or return, just examine the explicit non-variadic operands.
 226:   const MCInstrDesc &MCID = MI->getDesc();
 227:   unsigned E = MI->isCall() || MI->isReturn() ? MCID.getNumOperands()
 228:                                               : MI->getNumOperands();
 229:   for (unsigned I = 0; I != E; ++I) {
 230:     const MachineOperand &MO = MI->getOperand(I);
 231:     unsigned Reg;
 232: 
 233:     if (!MO.isReg() || !(Reg = MO.getReg()))
 234:       continue;
 235: 
 236:     if (MO.isDef())
 237:       RegDefs.insert(Reg);
 238:     else if (MO.isUse())
 239:       RegUses.insert(Reg);
 240:   }
```

- EN: Function bodies or method definitions such as insertDefsUses contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: insertDefsUses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-258

```cpp
 241: 
 242:   // Call & return instructions defines SP implicitly. Implicit defines are not
 243:   // included in the RegDefs set of calls but instructions modifying SP cannot
 244:   // be inserted in the delay slot of a call/return as these instructions are
 245:   // expanded to multiple instructions with SP modified before the branch that
 246:   // has the delay slot.
 247:   if (MI->isCall() || MI->isReturn())
 248:     RegDefs.insert(Lanai::SP);
 249: }
 250: 
 251: // Returns true if the Reg or its alias is in the RegSet.
 252: bool Filler::isRegInSet(SmallSet<unsigned, 32> &RegSet, unsigned Reg) {
 253:   // Check Reg and all aliased Registers.
 254:   for (MCRegAliasIterator AI(Reg, TRI, true); AI.isValid(); ++AI)
 255:     if (RegSet.count(*AI))
 256:       return true;
 257:   return false;
 258: }
```

- EN: Function bodies or method definitions such as isRegInSet contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isRegInSet 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Lanai.h`, `LanaiTargetMachine.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/Support/CommandLine.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
