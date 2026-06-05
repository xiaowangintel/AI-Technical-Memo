# AVRExpandPseudoInsts.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRExpandPseudoInsts.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains a pass that expands pseudo instructions into target instructions.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRExpandPseudoInsts.cpp - Expand pseudo instructions -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a pass that expands pseudo instructions into target
  10: // instructions. This pass should be run after register allocation but before
  11: // the post-regalloc scheduling pass.
  12: //
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "AVR.h"
  16: #include "AVRInstrInfo.h"
  17: #include "AVRTargetMachine.h"
  18: #include "MCTargetDesc/AVRMCTargetDesc.h"
  19: 
  20: #include "llvm/CodeGen/MachineFunctionPass.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineInstrBuilder.h"
  22: #include "llvm/CodeGen/MachineRegisterInfo.h"
  23: #include "llvm/CodeGen/TargetRegisterInfo.h"
  24: 
  25: using namespace llvm;
  26: 
  27: #define AVR_EXPAND_PSEUDO_NAME "AVR pseudo instruction expansion pass"
  28: 
  29: namespace {
  30: 
  31: /// Expands "placeholder" instructions marked as pseudo into
  32: /// actual AVR instructions.
  33: class AVRExpandPseudo : public MachineFunctionPass {
  34: public:
  35:   static char ID;
  36: 
  37:   AVRExpandPseudo() : MachineFunctionPass(ID) {}
  38: 
  39:   bool runOnMachineFunction(MachineFunction &MF) override;
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as AVRExpandPseudo, which organize the target-specific behavior exposed by the file.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 AVRExpandPseudo 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41:   StringRef getPassName() const override { return AVR_EXPAND_PSEUDO_NAME; }
  42: 
  43: private:
  44:   typedef MachineBasicBlock Block;
  45:   typedef Block::iterator BlockIt;
  46: 
  47:   const AVRRegisterInfo *TRI;
  48:   const TargetInstrInfo *TII;
  49: 
  50:   bool expandMBB(Block &MBB);
  51:   bool expandMI(Block &MBB, BlockIt MBBI);
  52:   template <unsigned OP> bool expand(Block &MBB, BlockIt MBBI);
  53: 
  54:   MachineInstrBuilder buildMI(Block &MBB, BlockIt MBBI, unsigned Opcode) {
  55:     return BuildMI(MBB, MBBI, MBBI->getDebugLoc(), TII->get(Opcode));
  56:   }
  57: 
  58:   MachineInstrBuilder buildMI(Block &MBB, BlockIt MBBI, unsigned Opcode,
  59:                               Register DstReg) {
  60:     return BuildMI(MBB, MBBI, MBBI->getDebugLoc(), TII->get(Opcode), DstReg);
```

- EN: Function bodies or method definitions such as buildMI contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: buildMI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61:   }
  62: 
  63:   MachineRegisterInfo &getRegInfo(Block &MBB) {
  64:     return MBB.getParent()->getRegInfo();
  65:   }
  66: 
  67:   bool expandArith(unsigned OpLo, unsigned OpHi, Block &MBB, BlockIt MBBI);
  68:   bool expandLogic(unsigned Op, Block &MBB, BlockIt MBBI);
  69:   bool expandLogicImm(unsigned Op, Block &MBB, BlockIt MBBI);
  70:   bool isLogicImmOpRedundant(unsigned Op, unsigned ImmVal) const;
  71:   bool isLogicRegOpUndef(unsigned Op, unsigned ImmVal) const;
  72: 
  73:   template <typename Func> bool expandAtomic(Block &MBB, BlockIt MBBI, Func f);
  74: 
  75:   template <typename Func>
  76:   bool expandAtomicBinaryOp(unsigned Opcode, Block &MBB, BlockIt MBBI, Func f);
  77: 
  78:   bool expandAtomicBinaryOp(unsigned Opcode, Block &MBB, BlockIt MBBI);
  79: 
  80:   /// Specific shift implementation for int8.
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:   bool expandLSLB7Rd(Block &MBB, BlockIt MBBI);
  82:   bool expandLSRB7Rd(Block &MBB, BlockIt MBBI);
  83:   bool expandASRB6Rd(Block &MBB, BlockIt MBBI);
  84:   bool expandASRB7Rd(Block &MBB, BlockIt MBBI);
  85: 
  86:   /// Specific shift implementation for int16.
  87:   bool expandLSLW4Rd(Block &MBB, BlockIt MBBI);
  88:   bool expandLSRW4Rd(Block &MBB, BlockIt MBBI);
  89:   bool expandASRW7Rd(Block &MBB, BlockIt MBBI);
  90:   bool expandLSLW8Rd(Block &MBB, BlockIt MBBI);
  91:   bool expandLSRW8Rd(Block &MBB, BlockIt MBBI);
  92:   bool expandASRW8Rd(Block &MBB, BlockIt MBBI);
  93:   bool expandLSLW12Rd(Block &MBB, BlockIt MBBI);
  94:   bool expandLSRW12Rd(Block &MBB, BlockIt MBBI);
  95:   bool expandASRW14Rd(Block &MBB, BlockIt MBBI);
  96:   bool expandASRW15Rd(Block &MBB, BlockIt MBBI);
  97: 
  98:   // Common implementation of LPMWRdZ and ELPMWRdZ.
  99:   bool expandLPMWELPMW(Block &MBB, BlockIt MBBI, bool IsELPM);
 100:   // Common implementation of LPMBRdZ and ELPMBRdZ.
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 101-120

```cpp
 101:   bool expandLPMBELPMB(Block &MBB, BlockIt MBBI, bool IsELPM);
 102:   // Common implementation of ROLBRdR1 and ROLBRdR17.
 103:   bool expandROLBRd(Block &MBB, BlockIt MBBI);
 104: };
 105: 
 106: char AVRExpandPseudo::ID = 0;
 107: 
 108: bool AVRExpandPseudo::expandMBB(MachineBasicBlock &MBB) {
 109:   bool Modified = false;
 110: 
 111:   BlockIt MBBI = MBB.begin(), E = MBB.end();
 112:   while (MBBI != E) {
 113:     BlockIt NMBBI = std::next(MBBI);
 114:     Modified |= expandMI(MBB, MBBI);
 115:     MBBI = NMBBI;
 116:   }
 117: 
 118:   return Modified;
 119: }
 120: 
```

- EN: Function bodies or method definitions such as expandMBB contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: expandMBB 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 121-140

```cpp
 121: bool AVRExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
 122:   bool Modified = false;
 123: 
 124:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 125:   TRI = STI.getRegisterInfo();
 126:   TII = STI.getInstrInfo();
 127: 
 128:   for (Block &MBB : MF) {
 129:     bool ContinueExpanding = true;
 130:     unsigned ExpandCount = 0;
 131: 
 132:     // Continue expanding the block until all pseudos are expanded.
 133:     do {
 134:       assert(ExpandCount < 10 && "pseudo expand limit reached");
 135:       (void)ExpandCount;
 136: 
 137:       bool BlockModified = expandMBB(MBB);
 138:       Modified |= BlockModified;
 139:       ExpandCount++;
 140: 
```

- EN: Function bodies or method definitions such as runOnMachineFunction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: runOnMachineFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 141-160

```cpp
 141:       ContinueExpanding = BlockModified;
 142:     } while (ContinueExpanding);
 143:   }
 144: 
 145:   return Modified;
 146: }
 147: 
 148: bool AVRExpandPseudo::expandArith(unsigned OpLo, unsigned OpHi, Block &MBB,
 149:                                   BlockIt MBBI) {
 150:   MachineInstr &MI = *MBBI;
 151:   Register SrcLoReg, SrcHiReg, DstLoReg, DstHiReg;
 152:   Register DstReg = MI.getOperand(0).getReg();
 153:   Register SrcReg = MI.getOperand(2).getReg();
 154:   bool DstIsDead = MI.getOperand(0).isDead();
 155:   bool DstIsKill = MI.getOperand(1).isKill();
 156:   bool SrcIsKill = MI.getOperand(2).isKill();
 157:   bool ImpIsDead = MI.getOperand(3).isDead();
 158:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
 159:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 160: 
```

- EN: Function bodies or method definitions such as expandArith contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: expandArith 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 161-180

```cpp
 161:   buildMI(MBB, MBBI, OpLo)
 162:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 163:       .addReg(DstLoReg, getKillRegState(DstIsKill))
 164:       .addReg(SrcLoReg, getKillRegState(SrcIsKill));
 165: 
 166:   auto MIBHI =
 167:       buildMI(MBB, MBBI, OpHi)
 168:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 169:           .addReg(DstHiReg, getKillRegState(DstIsKill))
 170:           .addReg(SrcHiReg, getKillRegState(SrcIsKill));
 171: 
 172:   if (ImpIsDead)
 173:     MIBHI->getOperand(3).setIsDead();
 174: 
 175:   // SREG is always implicitly killed
 176:   MIBHI->getOperand(4).setIsKill();
 177: 
 178:   MI.eraseFromParent();
 179:   return true;
 180: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181: 
 182: bool AVRExpandPseudo::expandLogic(unsigned Op, Block &MBB, BlockIt MBBI) {
 183:   MachineInstr &MI = *MBBI;
 184:   Register SrcLoReg, SrcHiReg, DstLoReg, DstHiReg;
 185:   Register DstReg = MI.getOperand(0).getReg();
 186:   Register SrcReg = MI.getOperand(2).getReg();
 187:   bool DstIsDead = MI.getOperand(0).isDead();
 188:   bool DstIsKill = MI.getOperand(1).isKill();
 189:   bool SrcIsKill = MI.getOperand(2).isKill();
 190:   bool ImpIsDead = MI.getOperand(3).isDead();
 191:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
 192:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 193: 
 194:   auto MIBLO =
 195:       buildMI(MBB, MBBI, Op)
 196:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 197:           .addReg(DstLoReg, getKillRegState(DstIsKill))
 198:           .addReg(SrcLoReg, getKillRegState(SrcIsKill));
 199: 
 200:   // SREG is always implicitly dead
```

- EN: Function bodies or method definitions such as expandLogic contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandLogic 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 201-220

```cpp
 201:   MIBLO->getOperand(3).setIsDead();
 202: 
 203:   auto MIBHI =
 204:       buildMI(MBB, MBBI, Op)
 205:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 206:           .addReg(DstHiReg, getKillRegState(DstIsKill))
 207:           .addReg(SrcHiReg, getKillRegState(SrcIsKill));
 208: 
 209:   if (ImpIsDead)
 210:     MIBHI->getOperand(3).setIsDead();
 211: 
 212:   MI.eraseFromParent();
 213:   return true;
 214: }
 215: 
 216: bool AVRExpandPseudo::isLogicImmOpRedundant(unsigned Op,
 217:                                             unsigned ImmVal) const {
 218: 
 219:   // ANDI Rd, 0xff is redundant.
 220:   if (Op == AVR::ANDIRdK && ImmVal == 0xff)
```

- EN: Function bodies or method definitions such as isLogicImmOpRedundant contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isLogicImmOpRedundant 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221:     return true;
 222: 
 223:   // ORI Rd, 0x0 is redundant.
 224:   if (Op == AVR::ORIRdK && ImmVal == 0x0)
 225:     return true;
 226: 
 227:   return false;
 228: }
 229: 
 230: bool AVRExpandPseudo::isLogicRegOpUndef(unsigned Op, unsigned ImmVal) const {
 231:   // ANDI Rd, 0x00 clears all input bits.
 232:   if (Op == AVR::ANDIRdK && ImmVal == 0x00)
 233:     return true;
 234: 
 235:   // ORI Rd, 0xff sets all input bits.
 236:   if (Op == AVR::ORIRdK && ImmVal == 0xff)
 237:     return true;
 238: 
 239:   return false;
 240: }
```

- EN: Function bodies or method definitions such as isLogicRegOpUndef contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isLogicRegOpUndef 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241: 
 242: bool AVRExpandPseudo::expandLogicImm(unsigned Op, Block &MBB, BlockIt MBBI) {
 243:   MachineInstr &MI = *MBBI;
 244:   Register DstLoReg, DstHiReg;
 245:   Register DstReg = MI.getOperand(0).getReg();
 246:   bool DstIsDead = MI.getOperand(0).isDead();
 247:   bool SrcIsKill = MI.getOperand(1).isKill();
 248:   bool ImpIsDead = MI.getOperand(3).isDead();
 249:   unsigned Imm = MI.getOperand(2).getImm();
 250:   unsigned Lo8 = Imm & 0xff;
 251:   unsigned Hi8 = (Imm >> 8) & 0xff;
 252:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 253: 
 254:   if (!isLogicImmOpRedundant(Op, Lo8)) {
 255:     auto MIBLO =
 256:         buildMI(MBB, MBBI, Op)
 257:             .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 258:             .addReg(DstLoReg, getKillRegState(SrcIsKill))
 259:             .addImm(Lo8);
 260: 
```

- EN: Function bodies or method definitions such as expandLogicImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLogicImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:     // SREG is always implicitly dead
 262:     MIBLO->getOperand(3).setIsDead();
 263: 
 264:     if (isLogicRegOpUndef(Op, Lo8))
 265:       MIBLO->getOperand(1).setIsUndef(true);
 266:   }
 267: 
 268:   if (!isLogicImmOpRedundant(Op, Hi8)) {
 269:     auto MIBHI =
 270:         buildMI(MBB, MBBI, Op)
 271:             .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 272:             .addReg(DstHiReg, getKillRegState(SrcIsKill))
 273:             .addImm(Hi8);
 274: 
 275:     if (ImpIsDead)
 276:       MIBHI->getOperand(3).setIsDead();
 277: 
 278:     if (isLogicRegOpUndef(Op, Hi8))
 279:       MIBHI->getOperand(1).setIsUndef(true);
 280:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281: 
 282:   MI.eraseFromParent();
 283:   return true;
 284: }
 285: 
 286: template <>
 287: bool AVRExpandPseudo::expand<AVR::ADDWRdRr>(Block &MBB, BlockIt MBBI) {
 288:   return expandArith(AVR::ADDRdRr, AVR::ADCRdRr, MBB, MBBI);
 289: }
 290: 
 291: template <>
 292: bool AVRExpandPseudo::expand<AVR::ADCWRdRr>(Block &MBB, BlockIt MBBI) {
 293:   return expandArith(AVR::ADCRdRr, AVR::ADCRdRr, MBB, MBBI);
 294: }
 295: 
 296: template <>
 297: bool AVRExpandPseudo::expand<AVR::SUBWRdRr>(Block &MBB, BlockIt MBBI) {
 298:   return expandArith(AVR::SUBRdRr, AVR::SBCRdRr, MBB, MBBI);
 299: }
 300: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 301-320

```cpp
 301: template <>
 302: bool AVRExpandPseudo::expand<AVR::SUBIWRdK>(Block &MBB, BlockIt MBBI) {
 303:   MachineInstr &MI = *MBBI;
 304:   Register DstLoReg, DstHiReg;
 305:   Register DstReg = MI.getOperand(0).getReg();
 306:   bool DstIsDead = MI.getOperand(0).isDead();
 307:   bool SrcIsKill = MI.getOperand(1).isKill();
 308:   bool ImpIsDead = MI.getOperand(3).isDead();
 309:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 310: 
 311:   auto MIBLO =
 312:       buildMI(MBB, MBBI, AVR::SUBIRdK)
 313:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 314:           .addReg(DstLoReg, getKillRegState(SrcIsKill));
 315: 
 316:   auto MIBHI =
 317:       buildMI(MBB, MBBI, AVR::SBCIRdK)
 318:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 319:           .addReg(DstHiReg, getKillRegState(SrcIsKill));
 320: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 321-340

```cpp
 321:   switch (MI.getOperand(2).getType()) {
 322:   case MachineOperand::MO_GlobalAddress: {
 323:     const GlobalValue *GV = MI.getOperand(2).getGlobal();
 324:     int64_t Offs = MI.getOperand(2).getOffset();
 325:     unsigned TF = MI.getOperand(2).getTargetFlags();
 326:     MIBLO.addGlobalAddress(GV, Offs, TF | AVRII::MO_NEG | AVRII::MO_LO);
 327:     MIBHI.addGlobalAddress(GV, Offs, TF | AVRII::MO_NEG | AVRII::MO_HI);
 328:     break;
 329:   }
 330:   case MachineOperand::MO_Immediate: {
 331:     unsigned Imm = MI.getOperand(2).getImm();
 332:     MIBLO.addImm(Imm & 0xff);
 333:     MIBHI.addImm((Imm >> 8) & 0xff);
 334:     break;
 335:   }
 336:   default:
 337:     llvm_unreachable("Unknown operand type!");
 338:   }
 339: 
 340:   if (ImpIsDead)
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:     MIBHI->getOperand(3).setIsDead();
 342: 
 343:   // SREG is always implicitly killed
 344:   MIBHI->getOperand(4).setIsKill();
 345: 
 346:   MI.eraseFromParent();
 347:   return true;
 348: }
 349: 
 350: template <>
 351: bool AVRExpandPseudo::expand<AVR::SBCWRdRr>(Block &MBB, BlockIt MBBI) {
 352:   return expandArith(AVR::SBCRdRr, AVR::SBCRdRr, MBB, MBBI);
 353: }
 354: 
 355: template <>
 356: bool AVRExpandPseudo::expand<AVR::SBCIWRdK>(Block &MBB, BlockIt MBBI) {
 357:   MachineInstr &MI = *MBBI;
 358:   Register DstLoReg, DstHiReg;
 359:   Register DstReg = MI.getOperand(0).getReg();
 360:   bool DstIsDead = MI.getOperand(0).isDead();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 361-380

```cpp
 361:   bool SrcIsKill = MI.getOperand(1).isKill();
 362:   bool ImpIsDead = MI.getOperand(3).isDead();
 363:   unsigned Imm = MI.getOperand(2).getImm();
 364:   unsigned Lo8 = Imm & 0xff;
 365:   unsigned Hi8 = (Imm >> 8) & 0xff;
 366:   unsigned OpLo = AVR::SBCIRdK;
 367:   unsigned OpHi = AVR::SBCIRdK;
 368:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 369: 
 370:   auto MIBLO =
 371:       buildMI(MBB, MBBI, OpLo)
 372:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 373:           .addReg(DstLoReg, getKillRegState(SrcIsKill))
 374:           .addImm(Lo8);
 375: 
 376:   // SREG is always implicitly killed
 377:   MIBLO->getOperand(4).setIsKill();
 378: 
 379:   auto MIBHI =
 380:       buildMI(MBB, MBBI, OpHi)
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 381-400

```cpp
 381:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 382:           .addReg(DstHiReg, getKillRegState(SrcIsKill))
 383:           .addImm(Hi8);
 384: 
 385:   if (ImpIsDead)
 386:     MIBHI->getOperand(3).setIsDead();
 387: 
 388:   // SREG is always implicitly killed
 389:   MIBHI->getOperand(4).setIsKill();
 390: 
 391:   MI.eraseFromParent();
 392:   return true;
 393: }
 394: 
 395: template <>
 396: bool AVRExpandPseudo::expand<AVR::ANDWRdRr>(Block &MBB, BlockIt MBBI) {
 397:   return expandLogic(AVR::ANDRdRr, MBB, MBBI);
 398: }
 399: 
 400: template <>
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 401-420

```cpp
 401: bool AVRExpandPseudo::expand<AVR::ANDIWRdK>(Block &MBB, BlockIt MBBI) {
 402:   return expandLogicImm(AVR::ANDIRdK, MBB, MBBI);
 403: }
 404: 
 405: template <>
 406: bool AVRExpandPseudo::expand<AVR::ORWRdRr>(Block &MBB, BlockIt MBBI) {
 407:   return expandLogic(AVR::ORRdRr, MBB, MBBI);
 408: }
 409: 
 410: template <>
 411: bool AVRExpandPseudo::expand<AVR::ORIWRdK>(Block &MBB, BlockIt MBBI) {
 412:   return expandLogicImm(AVR::ORIRdK, MBB, MBBI);
 413: }
 414: 
 415: template <>
 416: bool AVRExpandPseudo::expand<AVR::EORWRdRr>(Block &MBB, BlockIt MBBI) {
 417:   return expandLogic(AVR::EORRdRr, MBB, MBBI);
 418: }
 419: 
 420: template <>
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 421-440

```cpp
 421: bool AVRExpandPseudo::expand<AVR::COMWRd>(Block &MBB, BlockIt MBBI) {
 422:   MachineInstr &MI = *MBBI;
 423:   Register DstLoReg, DstHiReg;
 424:   Register DstReg = MI.getOperand(0).getReg();
 425:   bool DstIsDead = MI.getOperand(0).isDead();
 426:   bool DstIsKill = MI.getOperand(1).isKill();
 427:   bool ImpIsDead = MI.getOperand(2).isDead();
 428:   unsigned OpLo = AVR::COMRd;
 429:   unsigned OpHi = AVR::COMRd;
 430:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 431: 
 432:   auto MIBLO =
 433:       buildMI(MBB, MBBI, OpLo)
 434:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 435:           .addReg(DstLoReg, getKillRegState(DstIsKill));
 436: 
 437:   // SREG is always implicitly dead
 438:   MIBLO->getOperand(2).setIsDead();
 439: 
 440:   auto MIBHI =
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 441-460

```cpp
 441:       buildMI(MBB, MBBI, OpHi)
 442:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 443:           .addReg(DstHiReg, getKillRegState(DstIsKill));
 444: 
 445:   if (ImpIsDead)
 446:     MIBHI->getOperand(2).setIsDead();
 447: 
 448:   MI.eraseFromParent();
 449:   return true;
 450: }
 451: 
 452: template <>
 453: bool AVRExpandPseudo::expand<AVR::NEGWRd>(Block &MBB, BlockIt MBBI) {
 454:   MachineInstr &MI = *MBBI;
 455:   Register DstLoReg, DstHiReg;
 456:   Register DstReg = MI.getOperand(0).getReg();
 457:   Register ZeroReg = MI.getOperand(2).getReg();
 458:   bool DstIsDead = MI.getOperand(0).isDead();
 459:   bool DstIsKill = MI.getOperand(1).isKill();
 460:   bool ImpIsDead = MI.getOperand(2).isDead();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 462: 
 463:   // Do NEG on the upper byte.
 464:   auto MIBHI =
 465:       buildMI(MBB, MBBI, AVR::NEGRd)
 466:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 467:           .addReg(DstHiReg, RegState::Kill);
 468:   // SREG is always implicitly dead
 469:   MIBHI->getOperand(2).setIsDead();
 470: 
 471:   // Do NEG on the lower byte.
 472:   buildMI(MBB, MBBI, AVR::NEGRd)
 473:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 474:       .addReg(DstLoReg, getKillRegState(DstIsKill));
 475: 
 476:   // Do an extra SBC.
 477:   auto MISBCI =
 478:       buildMI(MBB, MBBI, AVR::SBCRdRr)
 479:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 480:           .addReg(DstHiReg, getKillRegState(DstIsKill))
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 481-500

```cpp
 481:           .addReg(ZeroReg);
 482:   if (ImpIsDead)
 483:     MISBCI->getOperand(3).setIsDead();
 484:   // SREG is always implicitly killed
 485:   MISBCI->getOperand(4).setIsKill();
 486: 
 487:   MI.eraseFromParent();
 488:   return true;
 489: }
 490: 
 491: template <>
 492: bool AVRExpandPseudo::expand<AVR::CPWRdRr>(Block &MBB, BlockIt MBBI) {
 493:   MachineInstr &MI = *MBBI;
 494:   Register SrcLoReg, SrcHiReg, DstLoReg, DstHiReg;
 495:   Register DstReg = MI.getOperand(0).getReg();
 496:   Register SrcReg = MI.getOperand(1).getReg();
 497:   bool DstIsKill = MI.getOperand(0).isKill();
 498:   bool SrcIsKill = MI.getOperand(1).isKill();
 499:   bool ImpIsDead = MI.getOperand(2).isDead();
 500:   unsigned OpLo = AVR::CPRdRr;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501:   unsigned OpHi = AVR::CPCRdRr;
 502:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
 503:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 504: 
 505:   // Low part
 506:   buildMI(MBB, MBBI, OpLo)
 507:       .addReg(DstLoReg, getKillRegState(DstIsKill))
 508:       .addReg(SrcLoReg, getKillRegState(SrcIsKill));
 509: 
 510:   auto MIBHI = buildMI(MBB, MBBI, OpHi)
 511:                    .addReg(DstHiReg, getKillRegState(DstIsKill))
 512:                    .addReg(SrcHiReg, getKillRegState(SrcIsKill));
 513: 
 514:   if (ImpIsDead)
 515:     MIBHI->getOperand(2).setIsDead();
 516: 
 517:   // SREG is always implicitly killed
 518:   MIBHI->getOperand(3).setIsKill();
 519: 
 520:   MI.eraseFromParent();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 521-540

```cpp
 521:   return true;
 522: }
 523: 
 524: template <>
 525: bool AVRExpandPseudo::expand<AVR::CPCWRdRr>(Block &MBB, BlockIt MBBI) {
 526:   MachineInstr &MI = *MBBI;
 527:   Register SrcLoReg, SrcHiReg, DstLoReg, DstHiReg;
 528:   Register DstReg = MI.getOperand(0).getReg();
 529:   Register SrcReg = MI.getOperand(1).getReg();
 530:   bool DstIsKill = MI.getOperand(0).isKill();
 531:   bool SrcIsKill = MI.getOperand(1).isKill();
 532:   bool ImpIsDead = MI.getOperand(2).isDead();
 533:   unsigned OpLo = AVR::CPCRdRr;
 534:   unsigned OpHi = AVR::CPCRdRr;
 535:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
 536:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 537: 
 538:   auto MIBLO = buildMI(MBB, MBBI, OpLo)
 539:                    .addReg(DstLoReg, getKillRegState(DstIsKill))
 540:                    .addReg(SrcLoReg, getKillRegState(SrcIsKill));
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 541-560

```cpp
 541: 
 542:   // SREG is always implicitly killed
 543:   MIBLO->getOperand(3).setIsKill();
 544: 
 545:   auto MIBHI = buildMI(MBB, MBBI, OpHi)
 546:                    .addReg(DstHiReg, getKillRegState(DstIsKill))
 547:                    .addReg(SrcHiReg, getKillRegState(SrcIsKill));
 548: 
 549:   if (ImpIsDead)
 550:     MIBHI->getOperand(2).setIsDead();
 551: 
 552:   // SREG is always implicitly killed
 553:   MIBHI->getOperand(3).setIsKill();
 554: 
 555:   MI.eraseFromParent();
 556:   return true;
 557: }
 558: 
 559: template <>
 560: bool AVRExpandPseudo::expand<AVR::LDIWRdK>(Block &MBB, BlockIt MBBI) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 561-580

```cpp
 561:   MachineInstr &MI = *MBBI;
 562:   Register DstLoReg, DstHiReg;
 563:   Register DstReg = MI.getOperand(0).getReg();
 564:   bool DstIsDead = MI.getOperand(0).isDead();
 565:   unsigned OpLo = AVR::LDIRdK;
 566:   unsigned OpHi = AVR::LDIRdK;
 567:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 568: 
 569:   auto MIBLO =
 570:       buildMI(MBB, MBBI, OpLo)
 571:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead));
 572: 
 573:   auto MIBHI =
 574:       buildMI(MBB, MBBI, OpHi)
 575:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead));
 576: 
 577:   switch (MI.getOperand(1).getType()) {
 578:   case MachineOperand::MO_GlobalAddress: {
 579:     const GlobalValue *GV = MI.getOperand(1).getGlobal();
 580:     int64_t Offs = MI.getOperand(1).getOffset();
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 581-600

```cpp
 581:     unsigned TF = MI.getOperand(1).getTargetFlags();
 582: 
 583:     MIBLO.addGlobalAddress(GV, Offs, TF | AVRII::MO_LO);
 584:     MIBHI.addGlobalAddress(GV, Offs, TF | AVRII::MO_HI);
 585:     break;
 586:   }
 587:   case MachineOperand::MO_BlockAddress: {
 588:     const BlockAddress *BA = MI.getOperand(1).getBlockAddress();
 589:     unsigned TF = MI.getOperand(1).getTargetFlags();
 590: 
 591:     MIBLO.add(MachineOperand::CreateBA(BA, TF | AVRII::MO_LO));
 592:     MIBHI.add(MachineOperand::CreateBA(BA, TF | AVRII::MO_HI));
 593:     break;
 594:   }
 595:   case MachineOperand::MO_Immediate: {
 596:     unsigned Imm = MI.getOperand(1).getImm();
 597: 
 598:     MIBLO.addImm(Imm & 0xff);
 599:     MIBHI.addImm((Imm >> 8) & 0xff);
 600:     break;
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 601-620

```cpp
 601:   }
 602:   default:
 603:     llvm_unreachable("Unknown operand type!");
 604:   }
 605: 
 606:   MI.eraseFromParent();
 607:   return true;
 608: }
 609: 
 610: template <>
 611: bool AVRExpandPseudo::expand<AVR::LDSWRdK>(Block &MBB, BlockIt MBBI) {
 612:   MachineInstr &MI = *MBBI;
 613:   Register DstLoReg, DstHiReg;
 614:   Register DstReg = MI.getOperand(0).getReg();
 615:   bool DstIsDead = MI.getOperand(0).isDead();
 616:   unsigned OpLo = AVR::LDSRdK;
 617:   unsigned OpHi = AVR::LDSRdK;
 618:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 619: 
 620:   auto MIBLO =
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 621-640

```cpp
 621:       buildMI(MBB, MBBI, OpLo)
 622:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead));
 623: 
 624:   auto MIBHI =
 625:       buildMI(MBB, MBBI, OpHi)
 626:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead));
 627: 
 628:   switch (MI.getOperand(1).getType()) {
 629:   case MachineOperand::MO_GlobalAddress: {
 630:     const GlobalValue *GV = MI.getOperand(1).getGlobal();
 631:     int64_t Offs = MI.getOperand(1).getOffset();
 632:     unsigned TF = MI.getOperand(1).getTargetFlags();
 633: 
 634:     MIBLO.addGlobalAddress(GV, Offs, TF);
 635:     MIBHI.addGlobalAddress(GV, Offs + 1, TF);
 636:     break;
 637:   }
 638:   case MachineOperand::MO_Immediate: {
 639:     unsigned Imm = MI.getOperand(1).getImm();
 640: 
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 641-660

```cpp
 641:     MIBLO.addImm(Imm);
 642:     MIBHI.addImm(Imm + 1);
 643:     break;
 644:   }
 645:   default:
 646:     llvm_unreachable("Unknown operand type!");
 647:   }
 648: 
 649:   MIBLO.setMemRefs(MI.memoperands());
 650:   MIBHI.setMemRefs(MI.memoperands());
 651: 
 652:   MI.eraseFromParent();
 653:   return true;
 654: }
 655: 
 656: template <>
 657: bool AVRExpandPseudo::expand<AVR::LDWRdPtr>(Block &MBB, BlockIt MBBI) {
 658:   MachineInstr &MI = *MBBI;
 659:   Register DstReg = MI.getOperand(0).getReg();
 660:   Register SrcReg = MI.getOperand(1).getReg();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 661-680

```cpp
 661:   bool DstIsKill = MI.getOperand(0).isKill();
 662:   bool SrcIsKill = MI.getOperand(1).isKill();
 663:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
 664: 
 665:   // DstReg has an earlyclobber so the register allocator will allocate them in
 666:   // separate registers.
 667:   assert(DstReg != SrcReg && "Dst and Src registers are the same!");
 668: 
 669:   if (STI.hasTinyEncoding()) {
 670:     // Handle this case in the expansion of LDDWRdPtrQ because it is very
 671:     // similar.
 672:     buildMI(MBB, MBBI, AVR::LDDWRdPtrQ)
 673:         .addDef(DstReg, getKillRegState(DstIsKill))
 674:         .addReg(SrcReg, getKillRegState(SrcIsKill))
 675:         .addImm(0)
 676:         .setMemRefs(MI.memoperands());
 677: 
 678:   } else {
 679:     Register DstLoReg, DstHiReg;
 680:     TRI->splitReg(DstReg, DstLoReg, DstHiReg);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 681-700

```cpp
 681: 
 682:     // Load low byte.
 683:     buildMI(MBB, MBBI, AVR::LDRdPtr)
 684:         .addReg(DstLoReg, RegState::Define)
 685:         .addReg(SrcReg)
 686:         .setMemRefs(MI.memoperands());
 687: 
 688:     // Load high byte.
 689:     buildMI(MBB, MBBI, AVR::LDDRdPtrQ)
 690:         .addReg(DstHiReg, RegState::Define)
 691:         .addReg(SrcReg, getKillRegState(SrcIsKill))
 692:         .addImm(1)
 693:         .setMemRefs(MI.memoperands());
 694:   }
 695: 
 696:   MI.eraseFromParent();
 697:   return true;
 698: }
 699: 
 700: template <>
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 701-720

```cpp
 701: bool AVRExpandPseudo::expand<AVR::LDWRdPtrPi>(Block &MBB, BlockIt MBBI) {
 702:   MachineInstr &MI = *MBBI;
 703:   Register DstLoReg, DstHiReg;
 704:   Register DstReg = MI.getOperand(0).getReg();
 705:   Register SrcReg = MI.getOperand(1).getReg();
 706:   bool DstIsDead = MI.getOperand(0).isDead();
 707:   bool SrcIsDead = MI.getOperand(1).isKill();
 708:   unsigned OpLo = AVR::LDRdPtrPi;
 709:   unsigned OpHi = AVR::LDRdPtrPi;
 710:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 711: 
 712:   assert(DstReg != SrcReg && "SrcReg and DstReg cannot be the same");
 713: 
 714:   auto MIBLO =
 715:       buildMI(MBB, MBBI, OpLo)
 716:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 717:           .addReg(SrcReg, RegState::Define)
 718:           .addReg(SrcReg, RegState::Kill);
 719: 
 720:   auto MIBHI =
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 721-740

```cpp
 721:       buildMI(MBB, MBBI, OpHi)
 722:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 723:           .addReg(SrcReg, RegState::Define | getDeadRegState(SrcIsDead))
 724:           .addReg(SrcReg, RegState::Kill);
 725: 
 726:   MIBLO.setMemRefs(MI.memoperands());
 727:   MIBHI.setMemRefs(MI.memoperands());
 728: 
 729:   MI.eraseFromParent();
 730:   return true;
 731: }
 732: 
 733: template <>
 734: bool AVRExpandPseudo::expand<AVR::LDWRdPtrPd>(Block &MBB, BlockIt MBBI) {
 735:   MachineInstr &MI = *MBBI;
 736:   Register DstLoReg, DstHiReg;
 737:   Register DstReg = MI.getOperand(0).getReg();
 738:   Register SrcReg = MI.getOperand(1).getReg();
 739:   bool DstIsDead = MI.getOperand(0).isDead();
 740:   bool SrcIsDead = MI.getOperand(1).isKill();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 741-760

```cpp
 741:   unsigned OpLo = AVR::LDRdPtrPd;
 742:   unsigned OpHi = AVR::LDRdPtrPd;
 743:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 744: 
 745:   assert(DstReg != SrcReg && "SrcReg and DstReg cannot be the same");
 746: 
 747:   auto MIBHI =
 748:       buildMI(MBB, MBBI, OpHi)
 749:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
 750:           .addReg(SrcReg, RegState::Define)
 751:           .addReg(SrcReg, RegState::Kill);
 752: 
 753:   auto MIBLO =
 754:       buildMI(MBB, MBBI, OpLo)
 755:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
 756:           .addReg(SrcReg, RegState::Define | getDeadRegState(SrcIsDead))
 757:           .addReg(SrcReg, RegState::Kill);
 758: 
 759:   MIBLO.setMemRefs(MI.memoperands());
 760:   MIBHI.setMemRefs(MI.memoperands());
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 761-780

```cpp
 761: 
 762:   MI.eraseFromParent();
 763:   return true;
 764: }
 765: 
 766: template <>
 767: bool AVRExpandPseudo::expand<AVR::LDDWRdPtrQ>(Block &MBB, BlockIt MBBI) {
 768:   MachineInstr &MI = *MBBI;
 769:   Register DstReg = MI.getOperand(0).getReg();
 770:   Register SrcReg = MI.getOperand(1).getReg();
 771:   unsigned Imm = MI.getOperand(2).getImm();
 772:   bool DstIsKill = MI.getOperand(0).isKill();
 773:   bool SrcIsKill = MI.getOperand(1).isKill();
 774:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
 775: 
 776:   // Since we add 1 to the Imm value for the high byte below, and 63 is the
 777:   // highest Imm value allowed for the instruction, 62 is the limit here.
 778:   assert(Imm <= 62 && "Offset is out of range");
 779: 
 780:   // DstReg has an earlyclobber so the register allocator will allocate them in
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 781-800

```cpp
 781:   // separate registers.
 782:   assert(DstReg != SrcReg && "Dst and Src registers are the same!");
 783: 
 784:   if (STI.hasTinyEncoding()) {
 785:     // Reduced tiny cores don't support load/store with displacement. However,
 786:     // they do support postincrement. So we'll simply adjust the pointer before
 787:     // and after and use postincrement to load multiple registers.
 788: 
 789:     // Add offset. The offset can be 0 when expanding this instruction from the
 790:     // more specific LDWRdPtr instruction.
 791:     if (Imm != 0) {
 792:       buildMI(MBB, MBBI, AVR::SUBIWRdK, SrcReg)
 793:           .addReg(SrcReg)
 794:           .addImm(0x10000 - Imm);
 795:     }
 796: 
 797:     // Do a word load with postincrement. This will be lowered to a two byte
 798:     // load.
 799:     buildMI(MBB, MBBI, AVR::LDWRdPtrPi)
 800:         .addDef(DstReg, getKillRegState(DstIsKill))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 801-820

```cpp
 801:         .addReg(SrcReg, getKillRegState(SrcIsKill))
 802:         .addImm(0)
 803:         .setMemRefs(MI.memoperands());
 804: 
 805:     // If the pointer is used after the store instruction, subtract the new
 806:     // offset (with 2 added after the postincrement instructions) so it is the
 807:     // same as before.
 808:     if (!SrcIsKill) {
 809:       buildMI(MBB, MBBI, AVR::SUBIWRdK, SrcReg).addReg(SrcReg).addImm(Imm + 2);
 810:     }
 811:   } else {
 812:     Register DstLoReg, DstHiReg;
 813:     TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 814: 
 815:     // Load low byte.
 816:     buildMI(MBB, MBBI, AVR::LDDRdPtrQ)
 817:         .addReg(DstLoReg, RegState::Define)
 818:         .addReg(SrcReg)
 819:         .addImm(Imm)
 820:         .setMemRefs(MI.memoperands());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 821-840

```cpp
 821: 
 822:     // Load high byte.
 823:     buildMI(MBB, MBBI, AVR::LDDRdPtrQ)
 824:         .addReg(DstHiReg, RegState::Define)
 825:         .addReg(SrcReg, getKillRegState(SrcIsKill))
 826:         .addImm(Imm + 1)
 827:         .setMemRefs(MI.memoperands());
 828:   }
 829: 
 830:   MI.eraseFromParent();
 831:   return true;
 832: }
 833: 
 834: bool AVRExpandPseudo::expandLPMWELPMW(Block &MBB, BlockIt MBBI, bool IsELPM) {
 835:   MachineInstr &MI = *MBBI;
 836:   Register DstLoReg, DstHiReg;
 837:   Register DstReg = MI.getOperand(0).getReg();
 838:   Register SrcReg = MI.getOperand(1).getReg();
 839:   Register SrcLoReg, SrcHiReg;
 840:   bool SrcIsKill = MI.getOperand(1).isKill();
```

- EN: Function bodies or method definitions such as expandLPMWELPMW contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandLPMWELPMW 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 841-860

```cpp
 841:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
 842:   bool IsLPMRn = IsELPM ? STI.hasELPMX() : STI.hasLPMX();
 843: 
 844:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
 845:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
 846: 
 847:   // Set the I/O register RAMPZ for ELPM.
 848:   if (IsELPM) {
 849:     Register Bank = MI.getOperand(2).getReg();
 850:     // out RAMPZ, rtmp
 851:     buildMI(MBB, MBBI, AVR::OUTARr).addImm(STI.getIORegRAMPZ()).addReg(Bank);
 852:   }
 853: 
 854:   // This is enforced by the @earlyclobber constraint.
 855:   assert(DstReg != SrcReg && "SrcReg and DstReg cannot be the same");
 856: 
 857:   if (IsLPMRn) {
 858:     unsigned OpLo = IsELPM ? AVR::ELPMRdZPi : AVR::LPMRdZPi;
 859:     unsigned OpHi = IsELPM ? AVR::ELPMRdZ : AVR::LPMRdZ;
 860:     // Load low byte.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 861-880

```cpp
 861:     auto MIBLO = buildMI(MBB, MBBI, OpLo)
 862:                      .addReg(DstLoReg, RegState::Define)
 863:                      .addReg(SrcReg);
 864:     // Load high byte.
 865:     auto MIBHI = buildMI(MBB, MBBI, OpHi)
 866:                      .addReg(DstHiReg, RegState::Define)
 867:                      .addReg(SrcReg, getKillRegState(SrcIsKill));
 868:     MIBLO.setMemRefs(MI.memoperands());
 869:     MIBHI.setMemRefs(MI.memoperands());
 870:   } else {
 871:     unsigned Opc = IsELPM ? AVR::ELPM : AVR::LPM;
 872:     // Load low byte, and copy to the low destination register.
 873:     auto MIBLO = buildMI(MBB, MBBI, Opc);
 874:     buildMI(MBB, MBBI, AVR::MOVRdRr)
 875:         .addReg(DstLoReg, RegState::Define)
 876:         .addReg(STI.getTmpRegister(), RegState::Kill);
 877:     MIBLO.setMemRefs(MI.memoperands());
 878:     // Increase the Z register by 1.
 879:     if (STI.hasADDSUBIW()) {
 880:       // adiw r31:r30, 1
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 881-900

```cpp
 881:       auto MIINC = buildMI(MBB, MBBI, AVR::ADIWRdK)
 882:                        .addReg(SrcReg, RegState::Define)
 883:                        .addReg(SrcReg, getKillRegState(SrcIsKill))
 884:                        .addImm(1);
 885:       MIINC->getOperand(3).setIsDead();
 886:     } else {
 887:       // subi r30, 255
 888:       // sbci r31, 255
 889:       buildMI(MBB, MBBI, AVR::SUBIRdK)
 890:           .addReg(SrcLoReg, RegState::Define)
 891:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
 892:           .addImm(255);
 893:       auto MIZHI = buildMI(MBB, MBBI, AVR::SBCIRdK)
 894:                        .addReg(SrcHiReg, RegState::Define)
 895:                        .addReg(SrcHiReg, getKillRegState(SrcIsKill))
 896:                        .addImm(255);
 897:       MIZHI->getOperand(3).setIsDead();
 898:       MIZHI->getOperand(4).setIsKill();
 899:     }
 900:     // Load high byte, and copy to the high destination register.
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 901-920

```cpp
 901:     auto MIBHI = buildMI(MBB, MBBI, Opc);
 902:     buildMI(MBB, MBBI, AVR::MOVRdRr)
 903:         .addReg(DstHiReg, RegState::Define)
 904:         .addReg(STI.getTmpRegister(), RegState::Kill);
 905:     MIBHI.setMemRefs(MI.memoperands());
 906:   }
 907: 
 908:   // Restore the Z register if it is not killed.
 909:   if (!SrcIsKill) {
 910:     if (STI.hasADDSUBIW()) {
 911:       // sbiw r31:r30, 1
 912:       auto MIDEC = buildMI(MBB, MBBI, AVR::SBIWRdK)
 913:                        .addReg(SrcReg, RegState::Define)
 914:                        .addReg(SrcReg, getKillRegState(SrcIsKill))
 915:                        .addImm(1);
 916:       MIDEC->getOperand(3).setIsDead();
 917:     } else {
 918:       // subi r30, 1
 919:       // sbci r31, 0
 920:       buildMI(MBB, MBBI, AVR::SUBIRdK)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 921-940

```cpp
 921:           .addReg(SrcLoReg, RegState::Define)
 922:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
 923:           .addImm(1);
 924:       auto MIZHI = buildMI(MBB, MBBI, AVR::SBCIRdK)
 925:                        .addReg(SrcHiReg, RegState::Define)
 926:                        .addReg(SrcHiReg, getKillRegState(SrcIsKill))
 927:                        .addImm(0);
 928:       MIZHI->getOperand(3).setIsDead();
 929:       MIZHI->getOperand(4).setIsKill();
 930:     }
 931:   }
 932: 
 933:   MI.eraseFromParent();
 934:   return true;
 935: }
 936: 
 937: template <>
 938: bool AVRExpandPseudo::expand<AVR::LPMWRdZ>(Block &MBB, BlockIt MBBI) {
 939:   return expandLPMWELPMW(MBB, MBBI, false);
 940: }
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 941-960

```cpp
 941: 
 942: template <>
 943: bool AVRExpandPseudo::expand<AVR::ELPMWRdZ>(Block &MBB, BlockIt MBBI) {
 944:   return expandLPMWELPMW(MBB, MBBI, true);
 945: }
 946: 
 947: bool AVRExpandPseudo::expandLPMBELPMB(Block &MBB, BlockIt MBBI, bool IsELPM) {
 948:   MachineInstr &MI = *MBBI;
 949:   Register DstReg = MI.getOperand(0).getReg();
 950:   Register SrcReg = MI.getOperand(1).getReg();
 951:   bool SrcIsKill = MI.getOperand(1).isKill();
 952:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
 953:   bool IsLPMRn = IsELPM ? STI.hasELPMX() : STI.hasLPMX();
 954: 
 955:   // Set the I/O register RAMPZ for ELPM (out RAMPZ, rtmp).
 956:   if (IsELPM) {
 957:     Register BankReg = MI.getOperand(2).getReg();
 958:     buildMI(MBB, MBBI, AVR::OUTARr).addImm(STI.getIORegRAMPZ()).addReg(BankReg);
 959:   }
 960: 
```

- EN: Function bodies or method definitions such as expandLPMBELPMB contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLPMBELPMB 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 961-980

```cpp
 961:   // Load byte.
 962:   if (IsLPMRn) {
 963:     unsigned Opc = IsELPM ? AVR::ELPMRdZ : AVR::LPMRdZ;
 964:     auto MILB = buildMI(MBB, MBBI, Opc)
 965:                     .addReg(DstReg, RegState::Define)
 966:                     .addReg(SrcReg, getKillRegState(SrcIsKill));
 967:     MILB.setMemRefs(MI.memoperands());
 968:   } else {
 969:     // For the basic ELPM/LPM instruction, its operand[0] is the implicit
 970:     // 'Z' register, and its operand[1] is the implicit 'R0' register.
 971:     unsigned Opc = IsELPM ? AVR::ELPM : AVR::LPM;
 972:     auto MILB = buildMI(MBB, MBBI, Opc);
 973:     buildMI(MBB, MBBI, AVR::MOVRdRr)
 974:         .addReg(DstReg, RegState::Define)
 975:         .addReg(STI.getTmpRegister(), RegState::Kill);
 976:     MILB.setMemRefs(MI.memoperands());
 977:   }
 978: 
 979:   MI.eraseFromParent();
 980:   return true;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 981-1000

```cpp
 981: }
 982: 
 983: template <>
 984: bool AVRExpandPseudo::expand<AVR::ELPMBRdZ>(Block &MBB, BlockIt MBBI) {
 985:   return expandLPMBELPMB(MBB, MBBI, true);
 986: }
 987: 
 988: template <>
 989: bool AVRExpandPseudo::expand<AVR::LPMBRdZ>(Block &MBB, BlockIt MBBI) {
 990:   return expandLPMBELPMB(MBB, MBBI, false);
 991: }
 992: 
 993: template <>
 994: bool AVRExpandPseudo::expand<AVR::LPMWRdZPi>(Block &MBB, BlockIt MBBI) {
 995:   llvm_unreachable("16-bit LPMPi is unimplemented");
 996: }
 997: 
 998: template <>
 999: bool AVRExpandPseudo::expand<AVR::ELPMBRdZPi>(Block &MBB, BlockIt MBBI) {
1000:   llvm_unreachable("8-bit ELPMPi is unimplemented");
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1001-1020

```cpp
1001: }
1002: 
1003: template <>
1004: bool AVRExpandPseudo::expand<AVR::ELPMWRdZPi>(Block &MBB, BlockIt MBBI) {
1005:   llvm_unreachable("16-bit ELPMPi is unimplemented");
1006: }
1007: 
1008: template <typename Func>
1009: bool AVRExpandPseudo::expandAtomic(Block &MBB, BlockIt MBBI, Func f) {
1010:   MachineInstr &MI = *MBBI;
1011:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
1012: 
1013:   // Store the SREG.
1014:   buildMI(MBB, MBBI, AVR::INRdA)
1015:       .addReg(STI.getTmpRegister(), RegState::Define)
1016:       .addImm(STI.getIORegSREG());
1017: 
1018:   // Disable exceptions.
1019:   buildMI(MBB, MBBI, AVR::BCLRs).addImm(7); // CLI
1020: 
```

- EN: Function bodies or method definitions such as expandAtomic contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandAtomic 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1021-1040

```cpp
1021:   f(MI);
1022: 
1023:   // Restore the status reg.
1024:   buildMI(MBB, MBBI, AVR::OUTARr)
1025:       .addImm(STI.getIORegSREG())
1026:       .addReg(STI.getTmpRegister());
1027: 
1028:   MI.eraseFromParent();
1029:   return true;
1030: }
1031: 
1032: template <typename Func>
1033: bool AVRExpandPseudo::expandAtomicBinaryOp(unsigned Opcode, Block &MBB,
1034:                                            BlockIt MBBI, Func f) {
1035:   return expandAtomic(MBB, MBBI, [&](MachineInstr &MI) {
1036:     auto Op1 = MI.getOperand(0);
1037:     auto Op2 = MI.getOperand(1);
1038: 
1039:     MachineInstr &NewInst =
1040:         *buildMI(MBB, MBBI, Opcode).add(Op1).add(Op2).getInstr();
```

- EN: Function bodies or method definitions such as expandAtomicBinaryOp, expandAtomic contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandAtomicBinaryOp, expandAtomic 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1041-1060

```cpp
1041:     f(NewInst);
1042:   });
1043: }
1044: 
1045: bool AVRExpandPseudo::expandAtomicBinaryOp(unsigned Opcode, Block &MBB,
1046:                                            BlockIt MBBI) {
1047:   return expandAtomicBinaryOp(Opcode, MBB, MBBI, [](MachineInstr &MI) {});
1048: }
1049: 
1050: template <>
1051: bool AVRExpandPseudo::expand<AVR::AtomicLoad8>(Block &MBB, BlockIt MBBI) {
1052:   return expandAtomicBinaryOp(AVR::LDRdPtr, MBB, MBBI);
1053: }
1054: 
1055: template <>
1056: bool AVRExpandPseudo::expand<AVR::AtomicLoad16>(Block &MBB, BlockIt MBBI) {
1057:   return expandAtomicBinaryOp(AVR::LDWRdPtr, MBB, MBBI);
1058: }
1059: 
1060: template <>
```

- EN: Function bodies or method definitions such as expandAtomicBinaryOp contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandAtomicBinaryOp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1061-1080

```cpp
1061: bool AVRExpandPseudo::expand<AVR::AtomicStore8>(Block &MBB, BlockIt MBBI) {
1062:   return expandAtomicBinaryOp(AVR::STPtrRr, MBB, MBBI);
1063: }
1064: 
1065: template <>
1066: bool AVRExpandPseudo::expand<AVR::AtomicStore16>(Block &MBB, BlockIt MBBI) {
1067:   return expandAtomicBinaryOp(AVR::STWPtrRr, MBB, MBBI);
1068: }
1069: 
1070: template <>
1071: bool AVRExpandPseudo::expand<AVR::AtomicFence>(Block &MBB, BlockIt MBBI) {
1072:   // On AVR, there is only one core and so atomic fences do nothing.
1073:   MBBI->eraseFromParent();
1074:   return true;
1075: }
1076: 
1077: template <>
1078: bool AVRExpandPseudo::expand<AVR::STSWKRr>(Block &MBB, BlockIt MBBI) {
1079:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
1080:   MachineInstr &MI = *MBBI;
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1081-1100

```cpp
1081:   Register SrcLoReg, SrcHiReg;
1082:   Register SrcReg = MI.getOperand(1).getReg();
1083:   bool SrcIsKill = MI.getOperand(1).isKill();
1084:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1085: 
1086:   auto MIB0 = buildMI(MBB, MBBI, AVR::STSKRr);
1087:   auto MIB1 = buildMI(MBB, MBBI, AVR::STSKRr);
1088: 
1089:   switch (MI.getOperand(0).getType()) {
1090:   case MachineOperand::MO_GlobalAddress: {
1091:     const GlobalValue *GV = MI.getOperand(0).getGlobal();
1092:     int64_t Offs = MI.getOperand(0).getOffset();
1093:     unsigned TF = MI.getOperand(0).getTargetFlags();
1094: 
1095:     if (STI.hasLowByteFirst()) {
1096:       // Write the low byte first for XMEGA devices.
1097:       MIB0.addGlobalAddress(GV, Offs, TF);
1098:       MIB1.addGlobalAddress(GV, Offs + 1, TF);
1099:     } else {
1100:       // Write the high byte first for traditional devices.
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1101-1120

```cpp
1101:       MIB0.addGlobalAddress(GV, Offs + 1, TF);
1102:       MIB1.addGlobalAddress(GV, Offs, TF);
1103:     }
1104: 
1105:     break;
1106:   }
1107:   case MachineOperand::MO_Immediate: {
1108:     unsigned Imm = MI.getOperand(0).getImm();
1109: 
1110:     if (STI.hasLowByteFirst()) {
1111:       // Write the low byte first for XMEGA devices.
1112:       MIB0.addImm(Imm);
1113:       MIB1.addImm(Imm + 1);
1114:     } else {
1115:       // Write the high byte first for traditional devices.
1116:       MIB0.addImm(Imm + 1);
1117:       MIB1.addImm(Imm);
1118:     }
1119: 
1120:     break;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1121-1140

```cpp
1121:   }
1122:   default:
1123:     llvm_unreachable("Unknown operand type!");
1124:   }
1125: 
1126:   if (STI.hasLowByteFirst()) {
1127:     // Write the low byte first for XMEGA devices.
1128:     MIB0.addReg(SrcLoReg, getKillRegState(SrcIsKill))
1129:         .setMemRefs(MI.memoperands());
1130:     MIB1.addReg(SrcHiReg, getKillRegState(SrcIsKill))
1131:         .setMemRefs(MI.memoperands());
1132:   } else {
1133:     // Write the high byte first for traditional devices.
1134:     MIB0.addReg(SrcHiReg, getKillRegState(SrcIsKill))
1135:         .setMemRefs(MI.memoperands());
1136:     MIB1.addReg(SrcLoReg, getKillRegState(SrcIsKill))
1137:         .setMemRefs(MI.memoperands());
1138:   }
1139: 
1140:   MI.eraseFromParent();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1141-1160

```cpp
1141:   return true;
1142: }
1143: 
1144: template <>
1145: bool AVRExpandPseudo::expand<AVR::STWPtrRr>(Block &MBB, BlockIt MBBI) {
1146:   MachineInstr &MI = *MBBI;
1147:   Register DstReg = MI.getOperand(0).getReg();
1148:   Register SrcReg = MI.getOperand(1).getReg();
1149:   bool DstIsKill = MI.getOperand(0).isKill();
1150:   bool DstIsUndef = MI.getOperand(0).isUndef();
1151:   bool SrcIsKill = MI.getOperand(1).isKill();
1152:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
1153: 
1154:   //: TODO: need to reverse this order like inw and stsw?
1155: 
1156:   if (STI.hasTinyEncoding()) {
1157:     // Handle this case in the expansion of STDWPtrQRr because it is very
1158:     // similar.
1159:     buildMI(MBB, MBBI, AVR::STDWPtrQRr)
1160:         .addReg(DstReg,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1161-1180

```cpp
1161:                 getKillRegState(DstIsKill) | getUndefRegState(DstIsUndef))
1162:         .addImm(0)
1163:         .addReg(SrcReg, getKillRegState(SrcIsKill))
1164:         .setMemRefs(MI.memoperands());
1165: 
1166:   } else {
1167:     Register SrcLoReg, SrcHiReg;
1168:     TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1169:     if (STI.hasLowByteFirst()) {
1170:       buildMI(MBB, MBBI, AVR::STPtrRr)
1171:           .addReg(DstReg, getUndefRegState(DstIsUndef))
1172:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1173:           .setMemRefs(MI.memoperands());
1174:       buildMI(MBB, MBBI, AVR::STDPtrQRr)
1175:           .addReg(DstReg, getUndefRegState(DstIsUndef))
1176:           .addImm(1)
1177:           .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1178:           .setMemRefs(MI.memoperands());
1179:     } else {
1180:       buildMI(MBB, MBBI, AVR::STDPtrQRr)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1181-1200

```cpp
1181:           .addReg(DstReg, getUndefRegState(DstIsUndef))
1182:           .addImm(1)
1183:           .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1184:           .setMemRefs(MI.memoperands());
1185:       buildMI(MBB, MBBI, AVR::STPtrRr)
1186:           .addReg(DstReg, getUndefRegState(DstIsUndef))
1187:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1188:           .setMemRefs(MI.memoperands());
1189:     }
1190:   }
1191: 
1192:   MI.eraseFromParent();
1193:   return true;
1194: }
1195: 
1196: template <>
1197: bool AVRExpandPseudo::expand<AVR::STWPtrPiRr>(Block &MBB, BlockIt MBBI) {
1198:   MachineInstr &MI = *MBBI;
1199:   Register SrcLoReg, SrcHiReg;
1200:   Register DstReg = MI.getOperand(0).getReg();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1201-1220

```cpp
1201:   Register SrcReg = MI.getOperand(2).getReg();
1202:   unsigned Imm = MI.getOperand(3).getImm();
1203:   bool DstIsDead = MI.getOperand(0).isDead();
1204:   bool SrcIsKill = MI.getOperand(2).isKill();
1205:   unsigned OpLo = AVR::STPtrPiRr;
1206:   unsigned OpHi = AVR::STPtrPiRr;
1207:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1208: 
1209:   assert(DstReg != SrcReg && "SrcReg and DstReg cannot be the same");
1210: 
1211:   auto MIBLO = buildMI(MBB, MBBI, OpLo)
1212:                    .addReg(DstReg, RegState::Define)
1213:                    .addReg(DstReg, RegState::Kill)
1214:                    .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1215:                    .addImm(Imm);
1216: 
1217:   auto MIBHI =
1218:       buildMI(MBB, MBBI, OpHi)
1219:           .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
1220:           .addReg(DstReg, RegState::Kill)
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1221-1240

```cpp
1221:           .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1222:           .addImm(Imm);
1223: 
1224:   MIBLO.setMemRefs(MI.memoperands());
1225:   MIBHI.setMemRefs(MI.memoperands());
1226: 
1227:   MI.eraseFromParent();
1228:   return true;
1229: }
1230: 
1231: template <>
1232: bool AVRExpandPseudo::expand<AVR::STWPtrPdRr>(Block &MBB, BlockIt MBBI) {
1233:   MachineInstr &MI = *MBBI;
1234:   Register SrcLoReg, SrcHiReg;
1235:   Register DstReg = MI.getOperand(0).getReg();
1236:   Register SrcReg = MI.getOperand(2).getReg();
1237:   unsigned Imm = MI.getOperand(3).getImm();
1238:   bool DstIsDead = MI.getOperand(0).isDead();
1239:   bool SrcIsKill = MI.getOperand(2).isKill();
1240:   unsigned OpLo = AVR::STPtrPdRr;
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1241-1260

```cpp
1241:   unsigned OpHi = AVR::STPtrPdRr;
1242:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1243: 
1244:   assert(DstReg != SrcReg && "SrcReg and DstReg cannot be the same");
1245: 
1246:   auto MIBHI = buildMI(MBB, MBBI, OpHi)
1247:                    .addReg(DstReg, RegState::Define)
1248:                    .addReg(DstReg, RegState::Kill)
1249:                    .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1250:                    .addImm(Imm);
1251: 
1252:   auto MIBLO =
1253:       buildMI(MBB, MBBI, OpLo)
1254:           .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
1255:           .addReg(DstReg, RegState::Kill)
1256:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1257:           .addImm(Imm);
1258: 
1259:   MIBLO.setMemRefs(MI.memoperands());
1260:   MIBHI.setMemRefs(MI.memoperands());
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1261-1280

```cpp
1261: 
1262:   MI.eraseFromParent();
1263:   return true;
1264: }
1265: 
1266: template <>
1267: bool AVRExpandPseudo::expand<AVR::STDWPtrQRr>(Block &MBB, BlockIt MBBI) {
1268:   MachineInstr &MI = *MBBI;
1269:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
1270: 
1271:   Register DstReg = MI.getOperand(0).getReg();
1272:   bool DstIsKill = MI.getOperand(0).isKill();
1273:   unsigned Imm = MI.getOperand(1).getImm();
1274:   Register SrcReg = MI.getOperand(2).getReg();
1275:   bool SrcIsKill = MI.getOperand(2).isKill();
1276: 
1277:   // STD's maximum displacement is 63, so larger stores have to be split into a
1278:   // set of operations.
1279:   // For avrtiny chips, STD is not available at all so we always have to fall
1280:   // back to manual pointer adjustments.
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1281-1300

```cpp
1281:   if (Imm >= 63 || STI.hasTinyEncoding()) {
1282:     // Add offset. The offset can be 0 when expanding this instruction from the
1283:     // more specific STWPtrRr instruction.
1284:     if (Imm != 0) {
1285:       buildMI(MBB, MBBI, AVR::SUBIWRdK, DstReg)
1286:           .addReg(DstReg, RegState::Kill)
1287:           .addImm(0x10000 - Imm);
1288:     }
1289: 
1290:     // Do the store. This is a word store, that will be expanded further.
1291:     buildMI(MBB, MBBI, AVR::STWPtrPiRr, DstReg)
1292:         .addReg(DstReg, getKillRegState(DstIsKill))
1293:         .addReg(SrcReg, getKillRegState(SrcIsKill))
1294:         .addImm(0)
1295:         .setMemRefs(MI.memoperands());
1296: 
1297:     // If the pointer is used after the store instruction, subtract the new
1298:     // offset (with 2 added after the postincrement instructions) so it is the
1299:     // same as before.
1300:     if (!DstIsKill) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1301-1320

```cpp
1301:       buildMI(MBB, MBBI, AVR::SUBIWRdK, DstReg)
1302:           .addReg(DstReg, RegState::Kill)
1303:           .addImm(Imm + 2);
1304:     }
1305:   } else {
1306:     Register SrcLoReg, SrcHiReg;
1307:     TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1308: 
1309:     if (STI.hasLowByteFirst()) {
1310:       buildMI(MBB, MBBI, AVR::STDPtrQRr)
1311:           .addReg(DstReg)
1312:           .addImm(Imm)
1313:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1314:           .setMemRefs(MI.memoperands());
1315:       buildMI(MBB, MBBI, AVR::STDPtrQRr)
1316:           .addReg(DstReg, getKillRegState(DstIsKill))
1317:           .addImm(Imm + 1)
1318:           .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1319:           .setMemRefs(MI.memoperands());
1320:     } else {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1321-1340

```cpp
1321:       buildMI(MBB, MBBI, AVR::STDPtrQRr)
1322:           .addReg(DstReg)
1323:           .addImm(Imm + 1)
1324:           .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1325:           .setMemRefs(MI.memoperands());
1326:       buildMI(MBB, MBBI, AVR::STDPtrQRr)
1327:           .addReg(DstReg, getKillRegState(DstIsKill))
1328:           .addImm(Imm)
1329:           .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1330:           .setMemRefs(MI.memoperands());
1331:     }
1332:   }
1333: 
1334:   MI.eraseFromParent();
1335:   return true;
1336: }
1337: 
1338: template <>
1339: bool AVRExpandPseudo::expand<AVR::STDSPQRr>(Block &MBB, BlockIt MBBI) {
1340:   MachineInstr &MI = *MBBI;
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1341-1360

```cpp
1341:   const MachineFunction &MF = *MBB.getParent();
1342:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
1343: 
1344:   assert(MI.getOperand(0).getReg() == AVR::SP &&
1345:          "SP is expected as base pointer");
1346: 
1347:   assert(STI.getFrameLowering()->hasReservedCallFrame(MF) &&
1348:          "unexpected STDSPQRr pseudo instruction");
1349:   (void)STI;
1350: 
1351:   MI.setDesc(TII->get(AVR::STDPtrQRr));
1352:   MI.getOperand(0).setReg(AVR::R29R28);
1353: 
1354:   return true;
1355: }
1356: 
1357: template <>
1358: bool AVRExpandPseudo::expand<AVR::STDWSPQRr>(Block &MBB, BlockIt MBBI) {
1359:   MachineInstr &MI = *MBBI;
1360:   const MachineFunction &MF = *MBB.getParent();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1361-1380

```cpp
1361:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
1362: 
1363:   assert(MI.getOperand(0).getReg() == AVR::SP &&
1364:          "SP is expected as base pointer");
1365: 
1366:   assert(STI.getFrameLowering()->hasReservedCallFrame(MF) &&
1367:          "unexpected STDWSPQRr pseudo instruction");
1368:   (void)STI;
1369: 
1370:   MI.setDesc(TII->get(AVR::STDWPtrQRr));
1371:   MI.getOperand(0).setReg(AVR::R29R28);
1372: 
1373:   return true;
1374: }
1375: 
1376: template <>
1377: bool AVRExpandPseudo::expand<AVR::INWRdA>(Block &MBB, BlockIt MBBI) {
1378:   MachineInstr &MI = *MBBI;
1379:   Register DstLoReg, DstHiReg;
1380:   unsigned Imm = MI.getOperand(1).getImm();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1381-1400

```cpp
1381:   Register DstReg = MI.getOperand(0).getReg();
1382:   bool DstIsDead = MI.getOperand(0).isDead();
1383:   unsigned OpLo = AVR::INRdA;
1384:   unsigned OpHi = AVR::INRdA;
1385:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1386: 
1387:   // Since we add 1 to the Imm value for the high byte below, and 63 is the
1388:   // highest Imm value allowed for the instruction, 62 is the limit here.
1389:   assert(Imm <= 62 && "Address is out of range");
1390: 
1391:   auto MIBLO =
1392:       buildMI(MBB, MBBI, OpLo)
1393:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1394:           .addImm(Imm);
1395: 
1396:   auto MIBHI =
1397:       buildMI(MBB, MBBI, OpHi)
1398:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1399:           .addImm(Imm + 1);
1400: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1401-1420

```cpp
1401:   MIBLO.setMemRefs(MI.memoperands());
1402:   MIBHI.setMemRefs(MI.memoperands());
1403: 
1404:   MI.eraseFromParent();
1405:   return true;
1406: }
1407: 
1408: template <>
1409: bool AVRExpandPseudo::expand<AVR::OUTWARr>(Block &MBB, BlockIt MBBI) {
1410:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
1411:   MachineInstr &MI = *MBBI;
1412:   Register SrcLoReg, SrcHiReg;
1413:   unsigned Imm = MI.getOperand(0).getImm();
1414:   Register SrcReg = MI.getOperand(1).getReg();
1415:   bool SrcIsKill = MI.getOperand(1).isKill();
1416:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1417: 
1418:   // Since we add 1 to the Imm value for the high byte below, and 63 is the
1419:   // highest Imm value allowed for the instruction, 62 is the limit here.
1420:   assert(Imm <= 62 && "Address is out of range");
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1421-1440

```cpp
1421: 
1422:   // 16 bit I/O writes need the high byte first on normal AVR devices,
1423:   // and in reverse order for the XMEGA/XMEGA3/XMEGAU families.
1424:   auto MIBHI = buildMI(MBB, MBBI, AVR::OUTARr)
1425:                    .addImm(STI.hasLowByteFirst() ? Imm : Imm + 1)
1426:                    .addReg(STI.hasLowByteFirst() ? SrcLoReg : SrcHiReg,
1427:                            getKillRegState(SrcIsKill));
1428:   auto MIBLO = buildMI(MBB, MBBI, AVR::OUTARr)
1429:                    .addImm(STI.hasLowByteFirst() ? Imm + 1 : Imm)
1430:                    .addReg(STI.hasLowByteFirst() ? SrcHiReg : SrcLoReg,
1431:                            getKillRegState(SrcIsKill));
1432: 
1433:   MIBLO.setMemRefs(MI.memoperands());
1434:   MIBHI.setMemRefs(MI.memoperands());
1435: 
1436:   MI.eraseFromParent();
1437:   return true;
1438: }
1439: 
1440: template <>
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1441-1460

```cpp
1441: bool AVRExpandPseudo::expand<AVR::PUSHWRr>(Block &MBB, BlockIt MBBI) {
1442:   MachineInstr &MI = *MBBI;
1443:   Register SrcLoReg, SrcHiReg;
1444:   Register SrcReg = MI.getOperand(0).getReg();
1445:   bool SrcIsKill = MI.getOperand(0).isKill();
1446:   unsigned Flags = MI.getFlags();
1447:   unsigned OpLo = AVR::PUSHRr;
1448:   unsigned OpHi = AVR::PUSHRr;
1449:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
1450: 
1451:   // Low part
1452:   buildMI(MBB, MBBI, OpLo)
1453:       .addReg(SrcLoReg, getKillRegState(SrcIsKill))
1454:       .setMIFlags(Flags);
1455: 
1456:   // High part
1457:   buildMI(MBB, MBBI, OpHi)
1458:       .addReg(SrcHiReg, getKillRegState(SrcIsKill))
1459:       .setMIFlags(Flags);
1460: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1461-1480

```cpp
1461:   MI.eraseFromParent();
1462:   return true;
1463: }
1464: 
1465: template <>
1466: bool AVRExpandPseudo::expand<AVR::POPWRd>(Block &MBB, BlockIt MBBI) {
1467:   MachineInstr &MI = *MBBI;
1468:   Register DstLoReg, DstHiReg;
1469:   Register DstReg = MI.getOperand(0).getReg();
1470:   unsigned Flags = MI.getFlags();
1471:   unsigned OpLo = AVR::POPRd;
1472:   unsigned OpHi = AVR::POPRd;
1473:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1474: 
1475:   buildMI(MBB, MBBI, OpHi, DstHiReg).setMIFlags(Flags); // High
1476:   buildMI(MBB, MBBI, OpLo, DstLoReg).setMIFlags(Flags); // Low
1477: 
1478:   MI.eraseFromParent();
1479:   return true;
1480: }
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1481-1500

```cpp
1481: 
1482: bool AVRExpandPseudo::expandROLBRd(Block &MBB, BlockIt MBBI) {
1483:   // In AVR, the rotate instructions behave quite unintuitively. They rotate
1484:   // bits through the carry bit in SREG, effectively rotating over 9 bits,
1485:   // instead of 8. This is useful when we are dealing with numbers over
1486:   // multiple registers, but when we actually need to rotate stuff, we have
1487:   // to explicitly add the carry bit.
1488: 
1489:   MachineInstr &MI = *MBBI;
1490:   unsigned OpShift, OpCarry;
1491:   Register DstReg = MI.getOperand(0).getReg();
1492:   Register ZeroReg = MI.getOperand(3).getReg();
1493:   bool DstIsDead = MI.getOperand(0).isDead();
1494:   bool DstIsKill = MI.getOperand(1).isKill();
1495:   OpShift = AVR::ADDRdRr;
1496:   OpCarry = AVR::ADCRdRr;
1497: 
1498:   // add r16, r16
1499:   // adc r16, r1
1500: 
```

- EN: Function bodies or method definitions such as expandROLBRd contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandROLBRd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1501-1520

```cpp
1501:   // Shift part
1502:   buildMI(MBB, MBBI, OpShift)
1503:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
1504:       .addReg(DstReg, RegState::Kill)
1505:       .addReg(DstReg, RegState::Kill);
1506: 
1507:   // Add the carry bit
1508:   auto MIB = buildMI(MBB, MBBI, OpCarry)
1509:                  .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
1510:                  .addReg(DstReg, getKillRegState(DstIsKill))
1511:                  .addReg(ZeroReg);
1512: 
1513:   MIB->getOperand(3).setIsDead(); // SREG is always dead
1514:   MIB->getOperand(4).setIsKill(); // SREG is always implicitly killed
1515: 
1516:   MI.eraseFromParent();
1517:   return true;
1518: }
1519: 
1520: template <>
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1521-1540

```cpp
1521: bool AVRExpandPseudo::expand<AVR::ROLBRdR1>(Block &MBB, BlockIt MBBI) {
1522:   return expandROLBRd(MBB, MBBI);
1523: }
1524: 
1525: template <>
1526: bool AVRExpandPseudo::expand<AVR::ROLBRdR17>(Block &MBB, BlockIt MBBI) {
1527:   return expandROLBRd(MBB, MBBI);
1528: }
1529: 
1530: template <>
1531: bool AVRExpandPseudo::expand<AVR::RORBRd>(Block &MBB, BlockIt MBBI) {
1532:   // In AVR, the rotate instructions behave quite unintuitively. They rotate
1533:   // bits through the carry bit in SREG, effectively rotating over 9 bits,
1534:   // instead of 8. This is useful when we are dealing with numbers over
1535:   // multiple registers, but when we actually need to rotate stuff, we have
1536:   // to explicitly add the carry bit.
1537: 
1538:   MachineInstr &MI = *MBBI;
1539:   Register DstReg = MI.getOperand(0).getReg();
1540: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1541-1560

```cpp
1541:   // bst r16, 0
1542:   // ror r16
1543:   // bld r16, 7
1544: 
1545:   // Move the lowest bit from DstReg into the T bit
1546:   buildMI(MBB, MBBI, AVR::BST).addReg(DstReg).addImm(0);
1547: 
1548:   // Rotate to the right
1549:   buildMI(MBB, MBBI, AVR::RORRd, DstReg).addReg(DstReg);
1550: 
1551:   // Move the T bit into the highest bit of DstReg.
1552:   buildMI(MBB, MBBI, AVR::BLD, DstReg).addReg(DstReg).addImm(7);
1553: 
1554:   MI.eraseFromParent();
1555:   return true;
1556: }
1557: 
1558: template <>
1559: bool AVRExpandPseudo::expand<AVR::LSLWRd>(Block &MBB, BlockIt MBBI) {
1560:   MachineInstr &MI = *MBBI;
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1561-1580

```cpp
1561:   Register DstLoReg, DstHiReg;
1562:   Register DstReg = MI.getOperand(0).getReg();
1563:   bool DstIsDead = MI.getOperand(0).isDead();
1564:   bool DstIsKill = MI.getOperand(1).isKill();
1565:   bool ImpIsDead = MI.getOperand(2).isDead();
1566:   unsigned OpLo = AVR::ADDRdRr; // ADD Rd, Rd <==> LSL Rd
1567:   unsigned OpHi = AVR::ADCRdRr; // ADC Rd, Rd <==> ROL Rd
1568:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1569: 
1570:   // Low part
1571:   buildMI(MBB, MBBI, OpLo)
1572:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1573:       .addReg(DstLoReg, getKillRegState(DstIsKill))
1574:       .addReg(DstLoReg, getKillRegState(DstIsKill));
1575: 
1576:   auto MIBHI =
1577:       buildMI(MBB, MBBI, OpHi)
1578:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1579:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1580:           .addReg(DstHiReg, getKillRegState(DstIsKill));
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1581-1600

```cpp
1581: 
1582:   if (ImpIsDead)
1583:     MIBHI->getOperand(3).setIsDead();
1584: 
1585:   // SREG is always implicitly killed
1586:   MIBHI->getOperand(4).setIsKill();
1587: 
1588:   MI.eraseFromParent();
1589:   return true;
1590: }
1591: 
1592: template <>
1593: bool AVRExpandPseudo::expand<AVR::LSLWHiRd>(Block &MBB, BlockIt MBBI) {
1594:   MachineInstr &MI = *MBBI;
1595:   Register DstLoReg, DstHiReg;
1596:   Register DstReg = MI.getOperand(0).getReg();
1597:   bool DstIsDead = MI.getOperand(0).isDead();
1598:   bool DstIsKill = MI.getOperand(1).isKill();
1599:   bool ImpIsDead = MI.getOperand(2).isDead();
1600:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1601-1620

```cpp
1601: 
1602:   // add hireg, hireg <==> lsl hireg
1603:   auto MILSL =
1604:       buildMI(MBB, MBBI, AVR::ADDRdRr)
1605:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1606:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1607:           .addReg(DstHiReg, getKillRegState(DstIsKill));
1608: 
1609:   if (ImpIsDead)
1610:     MILSL->getOperand(3).setIsDead();
1611: 
1612:   MI.eraseFromParent();
1613:   return true;
1614: }
1615: 
1616: bool AVRExpandPseudo::expandLSLW4Rd(Block &MBB, BlockIt MBBI) {
1617:   MachineInstr &MI = *MBBI;
1618:   Register DstLoReg, DstHiReg;
1619:   Register DstReg = MI.getOperand(0).getReg();
1620:   bool DstIsDead = MI.getOperand(0).isDead();
```

- EN: Function bodies or method definitions such as expandLSLW4Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLSLW4Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1621-1640

```cpp
1621:   bool DstIsKill = MI.getOperand(1).isKill();
1622:   bool ImpIsDead = MI.getOperand(3).isDead();
1623:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1624: 
1625:   // swap Rh
1626:   // swap Rl
1627:   buildMI(MBB, MBBI, AVR::SWAPRd)
1628:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1629:       .addReg(DstHiReg, RegState::Kill);
1630:   buildMI(MBB, MBBI, AVR::SWAPRd)
1631:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1632:       .addReg(DstLoReg, RegState::Kill);
1633: 
1634:   // andi Rh, 0xf0
1635:   auto MI0 =
1636:       buildMI(MBB, MBBI, AVR::ANDIRdK)
1637:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1638:           .addReg(DstHiReg, RegState::Kill)
1639:           .addImm(0xf0);
1640:   // SREG is implicitly dead.
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1641-1660

```cpp
1641:   MI0->getOperand(3).setIsDead();
1642: 
1643:   // eor Rh, Rl
1644:   auto MI1 =
1645:       buildMI(MBB, MBBI, AVR::EORRdRr)
1646:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1647:           .addReg(DstHiReg, RegState::Kill)
1648:           .addReg(DstLoReg);
1649:   // SREG is implicitly dead.
1650:   MI1->getOperand(3).setIsDead();
1651: 
1652:   // andi Rl, 0xf0
1653:   auto MI2 =
1654:       buildMI(MBB, MBBI, AVR::ANDIRdK)
1655:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1656:           .addReg(DstLoReg, getKillRegState(DstIsKill))
1657:           .addImm(0xf0);
1658:   // SREG is implicitly dead.
1659:   MI2->getOperand(3).setIsDead();
1660: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1661-1680

```cpp
1661:   // eor Rh, Rl
1662:   auto MI3 =
1663:       buildMI(MBB, MBBI, AVR::EORRdRr)
1664:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1665:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1666:           .addReg(DstLoReg);
1667:   if (ImpIsDead)
1668:     MI3->getOperand(3).setIsDead();
1669: 
1670:   MI.eraseFromParent();
1671:   return true;
1672: }
1673: 
1674: bool AVRExpandPseudo::expandLSLW8Rd(Block &MBB, BlockIt MBBI) {
1675:   MachineInstr &MI = *MBBI;
1676:   Register DstLoReg, DstHiReg;
1677:   Register DstReg = MI.getOperand(0).getReg();
1678:   bool DstIsDead = MI.getOperand(0).isDead();
1679:   bool DstIsKill = MI.getOperand(1).isKill();
1680:   bool ImpIsDead = MI.getOperand(3).isDead();
```

- EN: Function bodies or method definitions such as expandLSLW8Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLSLW8Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1681-1700

```cpp
1681:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1682: 
1683:   // mov Rh, Rl
1684:   buildMI(MBB, MBBI, AVR::MOVRdRr)
1685:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1686:       .addReg(DstLoReg);
1687: 
1688:   // clr Rl
1689:   auto MIBLO =
1690:       buildMI(MBB, MBBI, AVR::EORRdRr)
1691:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1692:           .addReg(DstLoReg, getKillRegState(DstIsKill))
1693:           .addReg(DstLoReg, getKillRegState(DstIsKill));
1694:   if (ImpIsDead)
1695:     MIBLO->getOperand(3).setIsDead();
1696: 
1697:   MI.eraseFromParent();
1698:   return true;
1699: }
1700: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1701-1720

```cpp
1701: bool AVRExpandPseudo::expandLSLW12Rd(Block &MBB, BlockIt MBBI) {
1702:   MachineInstr &MI = *MBBI;
1703:   Register DstLoReg, DstHiReg;
1704:   Register DstReg = MI.getOperand(0).getReg();
1705:   bool DstIsDead = MI.getOperand(0).isDead();
1706:   bool DstIsKill = MI.getOperand(1).isKill();
1707:   bool ImpIsDead = MI.getOperand(3).isDead();
1708:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1709: 
1710:   // mov Rh, Rl
1711:   buildMI(MBB, MBBI, AVR::MOVRdRr)
1712:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1713:       .addReg(DstLoReg);
1714: 
1715:   // swap Rh
1716:   buildMI(MBB, MBBI, AVR::SWAPRd)
1717:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1718:       .addReg(DstHiReg, RegState::Kill);
1719: 
1720:   // andi Rh, 0xf0
```

- EN: Function bodies or method definitions such as expandLSLW12Rd contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandLSLW12Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1721-1740

```cpp
1721:   auto MI0 =
1722:       buildMI(MBB, MBBI, AVR::ANDIRdK)
1723:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1724:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1725:           .addImm(0xf0);
1726:   // SREG is implicitly dead.
1727:   MI0->getOperand(3).setIsDead();
1728: 
1729:   // clr Rl
1730:   auto MI1 =
1731:       buildMI(MBB, MBBI, AVR::EORRdRr)
1732:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1733:           .addReg(DstLoReg, getKillRegState(DstIsKill))
1734:           .addReg(DstLoReg, getKillRegState(DstIsKill));
1735:   if (ImpIsDead)
1736:     MI1->getOperand(3).setIsDead();
1737: 
1738:   MI.eraseFromParent();
1739:   return true;
1740: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1741-1760

```cpp
1741: 
1742: template <>
1743: bool AVRExpandPseudo::expand<AVR::LSLWNRd>(Block &MBB, BlockIt MBBI) {
1744:   MachineInstr &MI = *MBBI;
1745:   unsigned Imm = MI.getOperand(2).getImm();
1746:   switch (Imm) {
1747:   case 4:
1748:     return expandLSLW4Rd(MBB, MBBI);
1749:   case 8:
1750:     return expandLSLW8Rd(MBB, MBBI);
1751:   case 12:
1752:     return expandLSLW12Rd(MBB, MBBI);
1753:   default:
1754:     llvm_unreachable("unimplemented lslwn");
1755:     return false;
1756:   }
1757: }
1758: 
1759: template <>
1760: bool AVRExpandPseudo::expand<AVR::LSRWRd>(Block &MBB, BlockIt MBBI) {
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 1761-1780

```cpp
1761:   MachineInstr &MI = *MBBI;
1762:   Register DstLoReg, DstHiReg;
1763:   Register DstReg = MI.getOperand(0).getReg();
1764:   bool DstIsDead = MI.getOperand(0).isDead();
1765:   bool DstIsKill = MI.getOperand(1).isKill();
1766:   bool ImpIsDead = MI.getOperand(2).isDead();
1767:   unsigned OpLo = AVR::RORRd;
1768:   unsigned OpHi = AVR::LSRRd;
1769:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1770: 
1771:   // High part
1772:   buildMI(MBB, MBBI, OpHi)
1773:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1774:       .addReg(DstHiReg, getKillRegState(DstIsKill));
1775: 
1776:   auto MIBLO =
1777:       buildMI(MBB, MBBI, OpLo)
1778:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1779:           .addReg(DstLoReg, getKillRegState(DstIsKill));
1780: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1781-1800

```cpp
1781:   if (ImpIsDead)
1782:     MIBLO->getOperand(2).setIsDead();
1783: 
1784:   // SREG is always implicitly killed
1785:   MIBLO->getOperand(3).setIsKill();
1786: 
1787:   MI.eraseFromParent();
1788:   return true;
1789: }
1790: 
1791: template <>
1792: bool AVRExpandPseudo::expand<AVR::LSRWLoRd>(Block &MBB, BlockIt MBBI) {
1793:   MachineInstr &MI = *MBBI;
1794:   Register DstLoReg, DstHiReg;
1795:   Register DstReg = MI.getOperand(0).getReg();
1796:   bool DstIsDead = MI.getOperand(0).isDead();
1797:   bool DstIsKill = MI.getOperand(1).isKill();
1798:   bool ImpIsDead = MI.getOperand(2).isDead();
1799:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1800: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1801-1820

```cpp
1801:   // lsr loreg
1802:   auto MILSR =
1803:       buildMI(MBB, MBBI, AVR::LSRRd)
1804:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1805:           .addReg(DstLoReg, getKillRegState(DstIsKill));
1806: 
1807:   if (ImpIsDead)
1808:     MILSR->getOperand(2).setIsDead();
1809: 
1810:   MI.eraseFromParent();
1811:   return true;
1812: }
1813: 
1814: bool AVRExpandPseudo::expandLSRW4Rd(Block &MBB, BlockIt MBBI) {
1815:   MachineInstr &MI = *MBBI;
1816:   Register DstLoReg, DstHiReg;
1817:   Register DstReg = MI.getOperand(0).getReg();
1818:   bool DstIsDead = MI.getOperand(0).isDead();
1819:   bool DstIsKill = MI.getOperand(1).isKill();
1820:   bool ImpIsDead = MI.getOperand(3).isDead();
```

- EN: Function bodies or method definitions such as expandLSRW4Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLSRW4Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1821-1840

```cpp
1821:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1822: 
1823:   // swap Rh
1824:   // swap Rl
1825:   buildMI(MBB, MBBI, AVR::SWAPRd)
1826:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1827:       .addReg(DstHiReg, RegState::Kill);
1828:   buildMI(MBB, MBBI, AVR::SWAPRd)
1829:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1830:       .addReg(DstLoReg, RegState::Kill);
1831: 
1832:   // andi Rl, 0xf
1833:   auto MI0 =
1834:       buildMI(MBB, MBBI, AVR::ANDIRdK)
1835:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1836:           .addReg(DstLoReg, RegState::Kill)
1837:           .addImm(0xf);
1838:   // SREG is implicitly dead.
1839:   MI0->getOperand(3).setIsDead();
1840: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1841-1860

```cpp
1841:   // eor Rl, Rh
1842:   auto MI1 =
1843:       buildMI(MBB, MBBI, AVR::EORRdRr)
1844:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1845:           .addReg(DstLoReg, RegState::Kill)
1846:           .addReg(DstHiReg);
1847:   // SREG is implicitly dead.
1848:   MI1->getOperand(3).setIsDead();
1849: 
1850:   // andi Rh, 0xf
1851:   auto MI2 =
1852:       buildMI(MBB, MBBI, AVR::ANDIRdK)
1853:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1854:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1855:           .addImm(0xf);
1856:   // SREG is implicitly dead.
1857:   MI2->getOperand(3).setIsDead();
1858: 
1859:   // eor Rl, Rh
1860:   auto MI3 =
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1861-1880

```cpp
1861:       buildMI(MBB, MBBI, AVR::EORRdRr)
1862:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1863:           .addReg(DstLoReg, getKillRegState(DstIsKill))
1864:           .addReg(DstHiReg);
1865:   if (ImpIsDead)
1866:     MI3->getOperand(3).setIsDead();
1867: 
1868:   MI.eraseFromParent();
1869:   return true;
1870: }
1871: 
1872: bool AVRExpandPseudo::expandLSRW8Rd(Block &MBB, BlockIt MBBI) {
1873:   MachineInstr &MI = *MBBI;
1874:   Register DstLoReg, DstHiReg;
1875:   Register DstReg = MI.getOperand(0).getReg();
1876:   bool DstIsDead = MI.getOperand(0).isDead();
1877:   bool DstIsKill = MI.getOperand(1).isKill();
1878:   bool ImpIsDead = MI.getOperand(3).isDead();
1879:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1880: 
```

- EN: Function bodies or method definitions such as expandLSRW8Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLSRW8Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1881-1900

```cpp
1881:   // Move upper byte to lower byte.
1882:   buildMI(MBB, MBBI, AVR::MOVRdRr)
1883:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1884:       .addReg(DstHiReg);
1885: 
1886:   // Clear upper byte.
1887:   auto MIBHI =
1888:       buildMI(MBB, MBBI, AVR::EORRdRr)
1889:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1890:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1891:           .addReg(DstHiReg, getKillRegState(DstIsKill));
1892:   if (ImpIsDead)
1893:     MIBHI->getOperand(3).setIsDead();
1894: 
1895:   MI.eraseFromParent();
1896:   return true;
1897: }
1898: 
1899: bool AVRExpandPseudo::expandLSRW12Rd(Block &MBB, BlockIt MBBI) {
1900:   MachineInstr &MI = *MBBI;
```

- EN: Function bodies or method definitions such as expandLSRW12Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandLSRW12Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1901-1920

```cpp
1901:   Register DstLoReg, DstHiReg;
1902:   Register DstReg = MI.getOperand(0).getReg();
1903:   bool DstIsDead = MI.getOperand(0).isDead();
1904:   bool DstIsKill = MI.getOperand(1).isKill();
1905:   bool ImpIsDead = MI.getOperand(3).isDead();
1906:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1907: 
1908:   // Move upper byte to lower byte.
1909:   buildMI(MBB, MBBI, AVR::MOVRdRr)
1910:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1911:       .addReg(DstHiReg);
1912: 
1913:   // swap Rl
1914:   buildMI(MBB, MBBI, AVR::SWAPRd)
1915:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1916:       .addReg(DstLoReg, RegState::Kill);
1917: 
1918:   // andi Rl, 0xf
1919:   auto MI0 =
1920:       buildMI(MBB, MBBI, AVR::ANDIRdK)
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1921-1940

```cpp
1921:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1922:           .addReg(DstLoReg, getKillRegState(DstIsKill))
1923:           .addImm(0xf);
1924:   // SREG is implicitly dead.
1925:   MI0->getOperand(3).setIsDead();
1926: 
1927:   // Clear upper byte.
1928:   auto MIBHI =
1929:       buildMI(MBB, MBBI, AVR::EORRdRr)
1930:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1931:           .addReg(DstHiReg, getKillRegState(DstIsKill))
1932:           .addReg(DstHiReg, getKillRegState(DstIsKill));
1933:   if (ImpIsDead)
1934:     MIBHI->getOperand(3).setIsDead();
1935: 
1936:   MI.eraseFromParent();
1937:   return true;
1938: }
1939: 
1940: template <>
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1941-1960

```cpp
1941: bool AVRExpandPseudo::expand<AVR::LSRWNRd>(Block &MBB, BlockIt MBBI) {
1942:   MachineInstr &MI = *MBBI;
1943:   unsigned Imm = MI.getOperand(2).getImm();
1944:   switch (Imm) {
1945:   case 4:
1946:     return expandLSRW4Rd(MBB, MBBI);
1947:   case 8:
1948:     return expandLSRW8Rd(MBB, MBBI);
1949:   case 12:
1950:     return expandLSRW12Rd(MBB, MBBI);
1951:   default:
1952:     llvm_unreachable("unimplemented lsrwn");
1953:     return false;
1954:   }
1955: }
1956: 
1957: template <>
1958: bool AVRExpandPseudo::expand<AVR::RORWRd>(Block &MBB, BlockIt MBBI) {
1959:   llvm_unreachable("RORW unimplemented");
1960:   return false;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 1961-1980

```cpp
1961: }
1962: 
1963: template <>
1964: bool AVRExpandPseudo::expand<AVR::ROLWRd>(Block &MBB, BlockIt MBBI) {
1965:   llvm_unreachable("ROLW unimplemented");
1966:   return false;
1967: }
1968: 
1969: template <>
1970: bool AVRExpandPseudo::expand<AVR::ASRWRd>(Block &MBB, BlockIt MBBI) {
1971:   MachineInstr &MI = *MBBI;
1972:   Register DstLoReg, DstHiReg;
1973:   Register DstReg = MI.getOperand(0).getReg();
1974:   bool DstIsDead = MI.getOperand(0).isDead();
1975:   bool DstIsKill = MI.getOperand(1).isKill();
1976:   bool ImpIsDead = MI.getOperand(2).isDead();
1977:   unsigned OpLo = AVR::RORRd;
1978:   unsigned OpHi = AVR::ASRRd;
1979:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
1980: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1981-2000

```cpp
1981:   // High part
1982:   buildMI(MBB, MBBI, OpHi)
1983:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
1984:       .addReg(DstHiReg, getKillRegState(DstIsKill));
1985: 
1986:   auto MIBLO =
1987:       buildMI(MBB, MBBI, OpLo)
1988:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
1989:           .addReg(DstLoReg, getKillRegState(DstIsKill));
1990: 
1991:   if (ImpIsDead)
1992:     MIBLO->getOperand(2).setIsDead();
1993: 
1994:   // SREG is always implicitly killed
1995:   MIBLO->getOperand(3).setIsKill();
1996: 
1997:   MI.eraseFromParent();
1998:   return true;
1999: }
2000: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2001-2020

```cpp
2001: template <>
2002: bool AVRExpandPseudo::expand<AVR::ASRWLoRd>(Block &MBB, BlockIt MBBI) {
2003:   MachineInstr &MI = *MBBI;
2004:   Register DstLoReg, DstHiReg;
2005:   Register DstReg = MI.getOperand(0).getReg();
2006:   bool DstIsDead = MI.getOperand(0).isDead();
2007:   bool DstIsKill = MI.getOperand(1).isKill();
2008:   bool ImpIsDead = MI.getOperand(2).isDead();
2009:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2010: 
2011:   // asr loreg
2012:   auto MIASR =
2013:       buildMI(MBB, MBBI, AVR::ASRRd)
2014:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2015:           .addReg(DstLoReg, getKillRegState(DstIsKill));
2016: 
2017:   if (ImpIsDead)
2018:     MIASR->getOperand(2).setIsDead();
2019: 
2020:   MI.eraseFromParent();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2021-2040

```cpp
2021:   return true;
2022: }
2023: 
2024: bool AVRExpandPseudo::expandASRW7Rd(Block &MBB, BlockIt MBBI) {
2025:   MachineInstr &MI = *MBBI;
2026:   Register DstLoReg, DstHiReg;
2027:   Register DstReg = MI.getOperand(0).getReg();
2028:   bool DstIsDead = MI.getOperand(0).isDead();
2029:   bool DstIsKill = MI.getOperand(1).isKill();
2030:   bool ImpIsDead = MI.getOperand(3).isDead();
2031:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2032: 
2033:   // lsl r24
2034:   // mov r24,r25
2035:   // rol r24
2036:   // sbc r25,r25
2037: 
2038:   // lsl r24 <=> add r24, r24
2039:   buildMI(MBB, MBBI, AVR::ADDRdRr)
2040:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
```

- EN: Function bodies or method definitions such as expandASRW7Rd contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandASRW7Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 2041-2060

```cpp
2041:       .addReg(DstLoReg, RegState::Kill)
2042:       .addReg(DstLoReg, RegState::Kill);
2043: 
2044:   // mov r24, r25
2045:   buildMI(MBB, MBBI, AVR::MOVRdRr)
2046:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2047:       .addReg(DstHiReg);
2048: 
2049:   // rol r24 <=> adc r24, r24
2050:   buildMI(MBB, MBBI, AVR::ADCRdRr)
2051:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2052:       .addReg(DstLoReg, getKillRegState(DstIsKill))
2053:       .addReg(DstLoReg, getKillRegState(DstIsKill));
2054: 
2055:   // sbc r25, r25
2056:   auto MISBC =
2057:       buildMI(MBB, MBBI, AVR::SBCRdRr)
2058:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2059:           .addReg(DstHiReg, getKillRegState(DstIsKill))
2060:           .addReg(DstHiReg, getKillRegState(DstIsKill));
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2061-2080

```cpp
2061: 
2062:   if (ImpIsDead)
2063:     MISBC->getOperand(3).setIsDead();
2064:   // SREG is always implicitly killed
2065:   MISBC->getOperand(4).setIsKill();
2066: 
2067:   MI.eraseFromParent();
2068:   return true;
2069: }
2070: 
2071: bool AVRExpandPseudo::expandASRW8Rd(Block &MBB, BlockIt MBBI) {
2072:   MachineInstr &MI = *MBBI;
2073:   Register DstLoReg, DstHiReg;
2074:   Register DstReg = MI.getOperand(0).getReg();
2075:   bool DstIsDead = MI.getOperand(0).isDead();
2076:   bool DstIsKill = MI.getOperand(1).isKill();
2077:   bool ImpIsDead = MI.getOperand(3).isDead();
2078:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2079: 
2080:   // Move upper byte to lower byte.
```

- EN: Function bodies or method definitions such as expandASRW8Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandASRW8Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2081-2100

```cpp
2081:   buildMI(MBB, MBBI, AVR::MOVRdRr)
2082:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2083:       .addReg(DstHiReg);
2084: 
2085:   // Move the sign bit to the C flag.
2086:   buildMI(MBB, MBBI, AVR::ADDRdRr)
2087:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2088:       .addReg(DstHiReg, RegState::Kill)
2089:       .addReg(DstHiReg, RegState::Kill);
2090: 
2091:   // Set upper byte to 0 or -1.
2092:   auto MIBHI =
2093:       buildMI(MBB, MBBI, AVR::SBCRdRr)
2094:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2095:           .addReg(DstHiReg, getKillRegState(DstIsKill))
2096:           .addReg(DstHiReg, getKillRegState(DstIsKill));
2097: 
2098:   if (ImpIsDead)
2099:     MIBHI->getOperand(3).setIsDead();
2100:   // SREG is always implicitly killed
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2101-2120

```cpp
2101:   MIBHI->getOperand(4).setIsKill();
2102: 
2103:   MI.eraseFromParent();
2104:   return true;
2105: }
2106: bool AVRExpandPseudo::expandASRW14Rd(Block &MBB, BlockIt MBBI) {
2107:   MachineInstr &MI = *MBBI;
2108:   Register DstLoReg, DstHiReg;
2109:   Register DstReg = MI.getOperand(0).getReg();
2110:   bool DstIsDead = MI.getOperand(0).isDead();
2111:   bool DstIsKill = MI.getOperand(1).isKill();
2112:   bool ImpIsDead = MI.getOperand(3).isDead();
2113:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2114: 
2115:   // lsl r25
2116:   // sbc r24, r24
2117:   // lsl r25
2118:   // mov r25, r24
2119:   // rol r24
2120: 
```

- EN: Function bodies or method definitions such as expandASRW14Rd contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandASRW14Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 2121-2140

```cpp
2121:   // lsl r25 <=> add r25, r25
2122:   buildMI(MBB, MBBI, AVR::ADDRdRr)
2123:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2124:       .addReg(DstHiReg, RegState::Kill)
2125:       .addReg(DstHiReg, RegState::Kill);
2126: 
2127:   // sbc r24, r24
2128:   buildMI(MBB, MBBI, AVR::SBCRdRr)
2129:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2130:       .addReg(DstLoReg, RegState::Kill)
2131:       .addReg(DstLoReg, RegState::Kill);
2132: 
2133:   // lsl r25 <=> add r25, r25
2134:   buildMI(MBB, MBBI, AVR::ADDRdRr)
2135:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2136:       .addReg(DstHiReg, RegState::Kill)
2137:       .addReg(DstHiReg, RegState::Kill);
2138: 
2139:   // mov r25, r24
2140:   buildMI(MBB, MBBI, AVR::MOVRdRr)
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2141-2160

```cpp
2141:       .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2142:       .addReg(DstLoReg);
2143: 
2144:   // rol r24 <=> adc r24, r24
2145:   auto MIROL =
2146:       buildMI(MBB, MBBI, AVR::ADCRdRr)
2147:           .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2148:           .addReg(DstLoReg, getKillRegState(DstIsKill))
2149:           .addReg(DstLoReg, getKillRegState(DstIsKill));
2150: 
2151:   if (ImpIsDead)
2152:     MIROL->getOperand(3).setIsDead();
2153:   // SREG is always implicitly killed
2154:   MIROL->getOperand(4).setIsKill();
2155: 
2156:   MI.eraseFromParent();
2157:   return false;
2158: }
2159: 
2160: bool AVRExpandPseudo::expandASRW15Rd(Block &MBB, BlockIt MBBI) {
```

- EN: Function bodies or method definitions such as expandASRW15Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: expandASRW15Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2161-2180

```cpp
2161:   MachineInstr &MI = *MBBI;
2162:   Register DstLoReg, DstHiReg;
2163:   Register DstReg = MI.getOperand(0).getReg();
2164:   bool DstIsDead = MI.getOperand(0).isDead();
2165:   bool ImpIsDead = MI.getOperand(3).isDead();
2166:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2167: 
2168:   // lsl r25
2169:   // sbc r25, r25
2170:   // mov r24, r25
2171: 
2172:   // lsl r25 <=> add r25, r25
2173:   buildMI(MBB, MBBI, AVR::ADDRdRr)
2174:       .addReg(DstHiReg, RegState::Define)
2175:       .addReg(DstHiReg, RegState::Kill)
2176:       .addReg(DstHiReg, RegState::Kill);
2177: 
2178:   // sbc r25, r25
2179:   auto MISBC =
2180:       buildMI(MBB, MBBI, AVR::SBCRdRr)
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2181-2200

```cpp
2181:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2182:           .addReg(DstHiReg, RegState::Kill)
2183:           .addReg(DstHiReg, RegState::Kill);
2184:   if (ImpIsDead)
2185:     MISBC->getOperand(3).setIsDead();
2186:   // SREG is always implicitly killed
2187:   MISBC->getOperand(4).setIsKill();
2188: 
2189:   // mov r24, r25
2190:   buildMI(MBB, MBBI, AVR::MOVRdRr)
2191:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2192:       .addReg(DstHiReg);
2193: 
2194:   MI.eraseFromParent();
2195:   return true;
2196: }
2197: 
2198: template <>
2199: bool AVRExpandPseudo::expand<AVR::ASRWNRd>(Block &MBB, BlockIt MBBI) {
2200:   MachineInstr &MI = *MBBI;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2201-2220

```cpp
2201:   unsigned Imm = MI.getOperand(2).getImm();
2202:   switch (Imm) {
2203:   case 7:
2204:     return expandASRW7Rd(MBB, MBBI);
2205:   case 8:
2206:     return expandASRW8Rd(MBB, MBBI);
2207:   case 14:
2208:     return expandASRW14Rd(MBB, MBBI);
2209:   case 15:
2210:     return expandASRW15Rd(MBB, MBBI);
2211:   default:
2212:     llvm_unreachable("unimplemented asrwn");
2213:     return false;
2214:   }
2215: }
2216: 
2217: bool AVRExpandPseudo::expandLSLB7Rd(Block &MBB, BlockIt MBBI) {
2218:   MachineInstr &MI = *MBBI;
2219:   Register DstReg = MI.getOperand(0).getReg();
2220:   bool DstIsDead = MI.getOperand(0).isDead();
```

- EN: Function bodies or method definitions such as expandLSLB7Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: expandLSLB7Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 2221-2240

```cpp
2221:   bool DstIsKill = MI.getOperand(1).isKill();
2222:   bool ImpIsDead = MI.getOperand(3).isDead();
2223: 
2224:   // ror r24
2225:   // clr r24
2226:   // ror r24
2227: 
2228:   buildMI(MBB, MBBI, AVR::RORRd)
2229:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2230:       .addReg(DstReg, RegState::Kill)
2231:       ->getOperand(3)
2232:       .setIsUndef(true);
2233: 
2234:   buildMI(MBB, MBBI, AVR::EORRdRr)
2235:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2236:       .addReg(DstReg, RegState::Kill)
2237:       .addReg(DstReg, RegState::Kill);
2238: 
2239:   auto MIRRC =
2240:       buildMI(MBB, MBBI, AVR::RORRd)
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2241-2260

```cpp
2241:           .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2242:           .addReg(DstReg, getKillRegState(DstIsKill));
2243: 
2244:   if (ImpIsDead)
2245:     MIRRC->getOperand(2).setIsDead();
2246: 
2247:   // SREG is always implicitly killed
2248:   MIRRC->getOperand(3).setIsKill();
2249: 
2250:   MI.eraseFromParent();
2251:   return true;
2252: }
2253: 
2254: template <>
2255: bool AVRExpandPseudo::expand<AVR::LSLBNRd>(Block &MBB, BlockIt MBBI) {
2256:   MachineInstr &MI = *MBBI;
2257:   unsigned Imm = MI.getOperand(2).getImm();
2258:   switch (Imm) {
2259:   case 7:
2260:     return expandLSLB7Rd(MBB, MBBI);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2261-2280

```cpp
2261:   default:
2262:     llvm_unreachable("unimplemented lslbn");
2263:     return false;
2264:   }
2265: }
2266: 
2267: bool AVRExpandPseudo::expandLSRB7Rd(Block &MBB, BlockIt MBBI) {
2268:   MachineInstr &MI = *MBBI;
2269:   Register DstReg = MI.getOperand(0).getReg();
2270:   bool DstIsDead = MI.getOperand(0).isDead();
2271:   bool DstIsKill = MI.getOperand(1).isKill();
2272:   bool ImpIsDead = MI.getOperand(3).isDead();
2273: 
2274:   // rol r24
2275:   // clr r24
2276:   // rol r24
2277: 
2278:   buildMI(MBB, MBBI, AVR::ADCRdRr)
2279:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2280:       .addReg(DstReg, RegState::Kill)
```

- EN: Function bodies or method definitions such as expandLSRB7Rd contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandLSRB7Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 2281-2300

```cpp
2281:       .addReg(DstReg, RegState::Kill)
2282:       ->getOperand(4)
2283:       .setIsUndef(true);
2284: 
2285:   buildMI(MBB, MBBI, AVR::EORRdRr)
2286:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2287:       .addReg(DstReg, RegState::Kill)
2288:       .addReg(DstReg, RegState::Kill);
2289: 
2290:   auto MIRRC =
2291:       buildMI(MBB, MBBI, AVR::ADCRdRr)
2292:           .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2293:           .addReg(DstReg, getKillRegState(DstIsKill))
2294:           .addReg(DstReg, getKillRegState(DstIsKill));
2295: 
2296:   if (ImpIsDead)
2297:     MIRRC->getOperand(3).setIsDead();
2298: 
2299:   // SREG is always implicitly killed
2300:   MIRRC->getOperand(4).setIsKill();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2301-2320

```cpp
2301: 
2302:   MI.eraseFromParent();
2303:   return true;
2304: }
2305: 
2306: template <>
2307: bool AVRExpandPseudo::expand<AVR::LSRBNRd>(Block &MBB, BlockIt MBBI) {
2308:   MachineInstr &MI = *MBBI;
2309:   unsigned Imm = MI.getOperand(2).getImm();
2310:   switch (Imm) {
2311:   case 7:
2312:     return expandLSRB7Rd(MBB, MBBI);
2313:   default:
2314:     llvm_unreachable("unimplemented lsrbn");
2315:     return false;
2316:   }
2317: }
2318: 
2319: bool AVRExpandPseudo::expandASRB6Rd(Block &MBB, BlockIt MBBI) {
2320:   MachineInstr &MI = *MBBI;
```

- EN: Function bodies or method definitions such as expandASRB6Rd contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: expandASRB6Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 2321-2340

```cpp
2321:   Register DstReg = MI.getOperand(0).getReg();
2322:   bool DstIsDead = MI.getOperand(0).isDead();
2323:   bool DstIsKill = MI.getOperand(1).isKill();
2324: 
2325:   // bst r24, 6
2326:   // lsl r24
2327:   // sbc r24, r24
2328:   // bld r24, 0
2329: 
2330:   buildMI(MBB, MBBI, AVR::BST)
2331:       .addReg(DstReg)
2332:       .addImm(6)
2333:       ->getOperand(2)
2334:       .setIsUndef(true);
2335: 
2336:   buildMI(MBB, MBBI, AVR::ADDRdRr) // LSL Rd <==> ADD Rd, Rd
2337:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2338:       .addReg(DstReg, RegState::Kill)
2339:       .addReg(DstReg, RegState::Kill);
2340: 
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2341-2360

```cpp
2341:   buildMI(MBB, MBBI, AVR::SBCRdRr)
2342:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2343:       .addReg(DstReg, RegState::Kill)
2344:       .addReg(DstReg, RegState::Kill);
2345: 
2346:   buildMI(MBB, MBBI, AVR::BLD)
2347:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2348:       .addReg(DstReg, getKillRegState(DstIsKill))
2349:       .addImm(0)
2350:       ->getOperand(3)
2351:       .setIsKill();
2352: 
2353:   MI.eraseFromParent();
2354:   return true;
2355: }
2356: 
2357: bool AVRExpandPseudo::expandASRB7Rd(Block &MBB, BlockIt MBBI) {
2358:   MachineInstr &MI = *MBBI;
2359:   Register DstReg = MI.getOperand(0).getReg();
2360:   bool DstIsDead = MI.getOperand(0).isDead();
```

- EN: Function bodies or method definitions such as expandASRB7Rd contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: expandASRB7Rd 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 2361-2380

```cpp
2361:   bool DstIsKill = MI.getOperand(1).isKill();
2362:   bool ImpIsDead = MI.getOperand(3).isDead();
2363: 
2364:   // lsl r24
2365:   // sbc r24, r24
2366: 
2367:   buildMI(MBB, MBBI, AVR::ADDRdRr)
2368:       .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2369:       .addReg(DstReg, RegState::Kill)
2370:       .addReg(DstReg, RegState::Kill);
2371: 
2372:   auto MIRRC =
2373:       buildMI(MBB, MBBI, AVR::SBCRdRr)
2374:           .addReg(DstReg, RegState::Define | getDeadRegState(DstIsDead))
2375:           .addReg(DstReg, getKillRegState(DstIsKill))
2376:           .addReg(DstReg, getKillRegState(DstIsKill));
2377: 
2378:   if (ImpIsDead)
2379:     MIRRC->getOperand(3).setIsDead();
2380: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2381-2400

```cpp
2381:   // SREG is always implicitly killed
2382:   MIRRC->getOperand(4).setIsKill();
2383: 
2384:   MI.eraseFromParent();
2385:   return true;
2386: }
2387: 
2388: template <>
2389: bool AVRExpandPseudo::expand<AVR::ASRBNRd>(Block &MBB, BlockIt MBBI) {
2390:   MachineInstr &MI = *MBBI;
2391:   unsigned Imm = MI.getOperand(2).getImm();
2392:   switch (Imm) {
2393:   case 6:
2394:     return expandASRB6Rd(MBB, MBBI);
2395:   case 7:
2396:     return expandASRB7Rd(MBB, MBBI);
2397:   default:
2398:     llvm_unreachable("unimplemented asrbn");
2399:     return false;
2400:   }
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 2401-2420

```cpp
2401: }
2402: 
2403: template <> bool AVRExpandPseudo::expand<AVR::SEXT>(Block &MBB, BlockIt MBBI) {
2404:   MachineInstr &MI = *MBBI;
2405:   Register DstLoReg, DstHiReg;
2406:   // sext R17:R16, R17
2407:   // mov     r16, r17
2408:   // lsl     r17
2409:   // sbc     r17, r17
2410:   // sext R17:R16, R13
2411:   // mov     r16, r13
2412:   // mov     r17, r13
2413:   // lsl     r17
2414:   // sbc     r17, r17
2415:   // sext R17:R16, R16
2416:   // mov     r17, r16
2417:   // lsl     r17
2418:   // sbc     r17, r17
2419:   Register DstReg = MI.getOperand(0).getReg();
2420:   Register SrcReg = MI.getOperand(1).getReg();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2421-2440

```cpp
2421:   bool DstIsDead = MI.getOperand(0).isDead();
2422:   bool SrcIsKill = MI.getOperand(1).isKill();
2423:   bool ImpIsDead = MI.getOperand(2).isDead();
2424:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2425: 
2426:   if (SrcReg != DstLoReg)
2427:     buildMI(MBB, MBBI, AVR::MOVRdRr)
2428:         .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2429:         .addReg(SrcReg);
2430: 
2431:   if (SrcReg != DstHiReg) {
2432:     auto MOV = buildMI(MBB, MBBI, AVR::MOVRdRr)
2433:                    .addReg(DstHiReg, RegState::Define)
2434:                    .addReg(SrcReg);
2435:     if (SrcReg != DstLoReg && SrcIsKill)
2436:       MOV->getOperand(1).setIsKill();
2437:   }
2438: 
2439:   buildMI(MBB, MBBI, AVR::ADDRdRr) // LSL Rd <==> ADD Rd, Rr
2440:       .addReg(DstHiReg, RegState::Define)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2441-2460

```cpp
2441:       .addReg(DstHiReg, RegState::Kill)
2442:       .addReg(DstHiReg, RegState::Kill);
2443: 
2444:   auto SBC =
2445:       buildMI(MBB, MBBI, AVR::SBCRdRr)
2446:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2447:           .addReg(DstHiReg, RegState::Kill)
2448:           .addReg(DstHiReg, RegState::Kill);
2449: 
2450:   if (ImpIsDead)
2451:     SBC->getOperand(3).setIsDead();
2452: 
2453:   // SREG is always implicitly killed
2454:   SBC->getOperand(4).setIsKill();
2455: 
2456:   MI.eraseFromParent();
2457:   return true;
2458: }
2459: 
2460: template <> bool AVRExpandPseudo::expand<AVR::ZEXT>(Block &MBB, BlockIt MBBI) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2461-2480

```cpp
2461:   MachineInstr &MI = *MBBI;
2462:   Register DstLoReg, DstHiReg;
2463:   // zext R25:R24, R20
2464:   // mov      R24, R20
2465:   // eor      R25, R25
2466:   // zext R25:R24, R24
2467:   // eor      R25, R25
2468:   // zext R25:R24, R25
2469:   // mov      R24, R25
2470:   // eor      R25, R25
2471:   Register DstReg = MI.getOperand(0).getReg();
2472:   Register SrcReg = MI.getOperand(1).getReg();
2473:   bool DstIsDead = MI.getOperand(0).isDead();
2474:   bool SrcIsKill = MI.getOperand(1).isKill();
2475:   bool ImpIsDead = MI.getOperand(2).isDead();
2476:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2477: 
2478:   if (SrcReg != DstLoReg) {
2479:     buildMI(MBB, MBBI, AVR::MOVRdRr)
2480:         .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2481-2500

```cpp
2481:         .addReg(SrcReg, getKillRegState(SrcIsKill));
2482:   }
2483: 
2484:   auto EOR =
2485:       buildMI(MBB, MBBI, AVR::EORRdRr)
2486:           .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2487:           .addReg(DstHiReg, RegState::Kill | RegState::Undef)
2488:           .addReg(DstHiReg, RegState::Kill | RegState::Undef);
2489: 
2490:   if (ImpIsDead)
2491:     EOR->getOperand(3).setIsDead();
2492: 
2493:   MI.eraseFromParent();
2494:   return true;
2495: }
2496: 
2497: template <>
2498: bool AVRExpandPseudo::expand<AVR::SPREAD>(Block &MBB, BlockIt MBBI) {
2499:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
2500:   MachineInstr &MI = *MBBI;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2501-2520

```cpp
2501:   Register DstLoReg, DstHiReg;
2502:   Register DstReg = MI.getOperand(0).getReg();
2503:   bool DstIsDead = MI.getOperand(0).isDead();
2504:   unsigned Flags = MI.getFlags();
2505:   TRI->splitReg(DstReg, DstLoReg, DstHiReg);
2506: 
2507:   // Low part
2508:   buildMI(MBB, MBBI, AVR::INRdA)
2509:       .addReg(DstLoReg, RegState::Define | getDeadRegState(DstIsDead))
2510:       .addImm(STI.getIORegSPL())
2511:       .setMIFlags(Flags);
2512: 
2513:   // High part
2514:   if (STI.getIORegSPH() != -1) {
2515:     buildMI(MBB, MBBI, AVR::INRdA)
2516:         .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2517:         .addImm(STI.getIORegSPH())
2518:         .setMIFlags(Flags);
2519:   } else {
2520:     // Clear the upper byte if there is no SPH.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2521-2540

```cpp
2521:     auto MI0 =
2522:         buildMI(MBB, MBBI, AVR::EORRdRr)
2523:             .addReg(DstHiReg, RegState::Define | getDeadRegState(DstIsDead))
2524:             .addReg(DstHiReg, RegState::Kill)
2525:             .addReg(DstHiReg);
2526:     // SREG is implicitly dead.
2527:     MI0->getOperand(3).setIsDead();
2528:   }
2529: 
2530:   MI.eraseFromParent();
2531:   return true;
2532: }
2533: 
2534: template <>
2535: bool AVRExpandPseudo::expand<AVR::SPWRITE>(Block &MBB, BlockIt MBBI) {
2536:   const AVRSubtarget &STI = MBB.getParent()->getSubtarget<AVRSubtarget>();
2537:   MachineInstr &MI = *MBBI;
2538:   Register SrcLoReg, SrcHiReg;
2539:   Register SrcReg = MI.getOperand(1).getReg();
2540:   bool SrcIsKill = MI.getOperand(1).isKill();
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2541-2560

```cpp
2541:   unsigned Flags = MI.getFlags();
2542:   TRI->splitReg(SrcReg, SrcLoReg, SrcHiReg);
2543: 
2544:   // From the XMEGA series manual:
2545:   // To prevent corruption when updating the stack pointer from software,
2546:   // a write to SPL will automatically disable interrupts
2547:   // for up to four instructions or until the next I/O memory write.
2548:   if (STI.getELFArch() >= 102) { // An XMEGA device
2549:     buildMI(MBB, MBBI, AVR::OUTARr)
2550:         .addImm(STI.getIORegSPL())
2551:         .addReg(SrcLoReg, getKillRegState(SrcIsKill))
2552:         .setMIFlags(Flags);
2553: 
2554:     buildMI(MBB, MBBI, AVR::OUTARr)
2555:         .addImm(STI.getIORegSPH())
2556:         .addReg(SrcHiReg, getKillRegState(SrcIsKill))
2557:         .setMIFlags(Flags);
2558:   } else { // Disable interrupts for older devices with SPH (3 extra
2559:            // instructions)
2560:     if (STI.getIORegSPH() != -1) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 2561-2580

```cpp
2561:       buildMI(MBB, MBBI, AVR::INRdA)
2562:           .addReg(STI.getTmpRegister(), RegState::Define)
2563:           .addImm(STI.getIORegSREG())
2564:           .setMIFlags(Flags);
2565: 
2566:       buildMI(MBB, MBBI, AVR::BCLRs).addImm(0x07).setMIFlags(Flags);
2567: 
2568:       buildMI(MBB, MBBI, AVR::OUTARr)
2569:           .addImm(STI.getIORegSPH())
2570:           .addReg(SrcHiReg, getKillRegState(SrcIsKill))
2571:           .setMIFlags(Flags);
2572: 
2573:       buildMI(MBB, MBBI, AVR::OUTARr)
2574:           .addImm(STI.getIORegSREG())
2575:           .addReg(STI.getTmpRegister(), RegState::Kill)
2576:           .setMIFlags(Flags);
2577:     }
2578: 
2579:     buildMI(MBB, MBBI, AVR::OUTARr)
2580:         .addImm(STI.getIORegSPL())
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2581-2600

```cpp
2581:         .addReg(SrcLoReg, getKillRegState(SrcIsKill))
2582:         .setMIFlags(Flags);
2583:   }
2584: 
2585:   MI.eraseFromParent();
2586:   return true;
2587: }
2588: 
2589: bool AVRExpandPseudo::expandMI(Block &MBB, BlockIt MBBI) {
2590:   MachineInstr &MI = *MBBI;
2591:   int Opcode = MBBI->getOpcode();
2592: 
2593: #define EXPAND(Op)                                                             \
2594:   case Op:                                                                     \
2595:     return expand<Op>(MBB, MI)
2596: 
2597:   switch (Opcode) {
2598:     EXPAND(AVR::ADDWRdRr);
2599:     EXPAND(AVR::ADCWRdRr);
2600:     EXPAND(AVR::SUBWRdRr);
```

- EN: Function bodies or method definitions such as expandMI contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: expandMI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 2601-2620

```cpp
2601:     EXPAND(AVR::SUBIWRdK);
2602:     EXPAND(AVR::SBCWRdRr);
2603:     EXPAND(AVR::SBCIWRdK);
2604:     EXPAND(AVR::ANDWRdRr);
2605:     EXPAND(AVR::ANDIWRdK);
2606:     EXPAND(AVR::ORWRdRr);
2607:     EXPAND(AVR::ORIWRdK);
2608:     EXPAND(AVR::EORWRdRr);
2609:     EXPAND(AVR::COMWRd);
2610:     EXPAND(AVR::NEGWRd);
2611:     EXPAND(AVR::CPWRdRr);
2612:     EXPAND(AVR::CPCWRdRr);
2613:     EXPAND(AVR::LDIWRdK);
2614:     EXPAND(AVR::LDSWRdK);
2615:     EXPAND(AVR::LDWRdPtr);
2616:     EXPAND(AVR::LDWRdPtrPi);
2617:     EXPAND(AVR::LDWRdPtrPd);
2618:   case AVR::LDDWRdYQ: //: FIXME: remove this once PR13375 gets fixed
2619:     EXPAND(AVR::LDDWRdPtrQ);
2620:     EXPAND(AVR::LPMBRdZ);
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2621-2640

```cpp
2621:     EXPAND(AVR::LPMWRdZ);
2622:     EXPAND(AVR::LPMWRdZPi);
2623:     EXPAND(AVR::ELPMBRdZ);
2624:     EXPAND(AVR::ELPMWRdZ);
2625:     EXPAND(AVR::ELPMBRdZPi);
2626:     EXPAND(AVR::ELPMWRdZPi);
2627:     EXPAND(AVR::AtomicLoad8);
2628:     EXPAND(AVR::AtomicLoad16);
2629:     EXPAND(AVR::AtomicStore8);
2630:     EXPAND(AVR::AtomicStore16);
2631:     EXPAND(AVR::AtomicFence);
2632:     EXPAND(AVR::STSWKRr);
2633:     EXPAND(AVR::STWPtrRr);
2634:     EXPAND(AVR::STWPtrPiRr);
2635:     EXPAND(AVR::STWPtrPdRr);
2636:     EXPAND(AVR::STDWPtrQRr);
2637:     EXPAND(AVR::STDSPQRr);
2638:     EXPAND(AVR::STDWSPQRr);
2639:     EXPAND(AVR::INWRdA);
2640:     EXPAND(AVR::OUTWARr);
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2641-2660

```cpp
2641:     EXPAND(AVR::PUSHWRr);
2642:     EXPAND(AVR::POPWRd);
2643:     EXPAND(AVR::ROLBRdR1);
2644:     EXPAND(AVR::ROLBRdR17);
2645:     EXPAND(AVR::RORBRd);
2646:     EXPAND(AVR::LSLWRd);
2647:     EXPAND(AVR::LSRWRd);
2648:     EXPAND(AVR::RORWRd);
2649:     EXPAND(AVR::ROLWRd);
2650:     EXPAND(AVR::ASRWRd);
2651:     EXPAND(AVR::LSLWHiRd);
2652:     EXPAND(AVR::LSRWLoRd);
2653:     EXPAND(AVR::ASRWLoRd);
2654:     EXPAND(AVR::LSLWNRd);
2655:     EXPAND(AVR::LSRWNRd);
2656:     EXPAND(AVR::ASRWNRd);
2657:     EXPAND(AVR::LSLBNRd);
2658:     EXPAND(AVR::LSRBNRd);
2659:     EXPAND(AVR::ASRBNRd);
2660:     EXPAND(AVR::SEXT);
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 2661-2676

```cpp
2661:     EXPAND(AVR::ZEXT);
2662:     EXPAND(AVR::SPREAD);
2663:     EXPAND(AVR::SPWRITE);
2664:   }
2665: #undef EXPAND
2666:   return false;
2667: }
2668: 
2669: } // end of anonymous namespace
2670: 
2671: INITIALIZE_PASS(AVRExpandPseudo, "avr-expand-pseudo", AVR_EXPAND_PSEUDO_NAME,
2672:                 false, false)
2673: 
2674: FunctionPass *llvm::createAVRExpandPseudoPass() {
2675:   return new AVRExpandPseudo();
2676: }
```

- EN: This range continues the implementation of the backend component described by AVRExpandPseudoInsts.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVR.h`, `AVRInstrInfo.h`, `AVRTargetMachine.h`, `MCTargetDesc/AVRMCTargetDesc.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetRegisterInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
