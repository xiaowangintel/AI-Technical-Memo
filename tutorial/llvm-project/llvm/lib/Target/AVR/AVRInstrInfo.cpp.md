# AVRInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRInstrInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the AVR implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRInstrInfo.cpp - AVR Instruction Information --------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the AVR implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "AVRInstrInfo.h"
  14: 
  15: #include "llvm/CodeGen/MachineFrameInfo.h"
  16: #include "llvm/CodeGen/MachineInstrBuilder.h"
  17: #include "llvm/CodeGen/MachineMemOperand.h"
  18: #include "llvm/MC/MCContext.h"
  19: #include "llvm/Support/ErrorHandling.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: #include "AVR.h"
  22: #include "AVRMachineFunctionInfo.h"
  23: #include "AVRRegisterInfo.h"
  24: #include "AVRTargetMachine.h"
  25: #include "MCTargetDesc/AVRMCTargetDesc.h"
  26: 
  27: #define GET_INSTRINFO_CTOR_DTOR
  28: #include "AVRGenInstrInfo.inc"
  29: 
  30: namespace llvm {
  31: 
  32: AVRInstrInfo::AVRInstrInfo(const AVRSubtarget &STI)
  33:     : AVRGenInstrInfo(STI, RI, AVR::ADJCALLSTACKDOWN, AVR::ADJCALLSTACKUP),
  34:       RI(), STI(STI) {}
  35: 
  36: void AVRInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
  37:                                MachineBasicBlock::iterator MI,
  38:                                const DebugLoc &DL, Register DestReg,
  39:                                Register SrcReg, bool KillSrc,
  40:                                bool RenamableDest, bool RenamableSrc) const {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as AVRGenInstrInfo, copyPhysReg contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 AVRGenInstrInfo, copyPhysReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:   const AVRRegisterInfo &TRI = *STI.getRegisterInfo();
  42:   unsigned Opc;
  43: 
  44:   if (AVR::DREGSRegClass.contains(DestReg, SrcReg)) {
  45:     // If our AVR has `movw`, let's emit that; otherwise let's emit two separate
  46:     // `mov`s.
  47:     if (STI.hasMOVW() && AVR::DREGSMOVWRegClass.contains(DestReg, SrcReg)) {
  48:       BuildMI(MBB, MI, DL, get(AVR::MOVWRdRr), DestReg)
  49:           .addReg(SrcReg, getKillRegState(KillSrc));
  50:     } else {
  51:       Register DestLo, DestHi, SrcLo, SrcHi;
  52: 
  53:       TRI.splitReg(DestReg, DestLo, DestHi);
  54:       TRI.splitReg(SrcReg, SrcLo, SrcHi);
  55: 
  56:       // Emit the copies.
  57:       // The original instruction was for a register pair, of which only one
  58:       // register might have been live. Add 'undef' to satisfy the machine
  59:       // verifier, when subreg liveness is enabled.
  60:       // TODO: Eliminate these unnecessary copies.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:       if (DestLo == SrcHi) {
  62:         BuildMI(MBB, MI, DL, get(AVR::MOVRdRr), DestHi)
  63:             .addReg(SrcHi, getKillRegState(KillSrc) | RegState::Undef);
  64:         BuildMI(MBB, MI, DL, get(AVR::MOVRdRr), DestLo)
  65:             .addReg(SrcLo, getKillRegState(KillSrc) | RegState::Undef);
  66:       } else {
  67:         BuildMI(MBB, MI, DL, get(AVR::MOVRdRr), DestLo)
  68:             .addReg(SrcLo, getKillRegState(KillSrc) | RegState::Undef);
  69:         BuildMI(MBB, MI, DL, get(AVR::MOVRdRr), DestHi)
  70:             .addReg(SrcHi, getKillRegState(KillSrc) | RegState::Undef);
  71:       }
  72:     }
  73:   } else {
  74:     if (AVR::GPR8RegClass.contains(DestReg, SrcReg)) {
  75:       Opc = AVR::MOVRdRr;
  76:     } else if (SrcReg == AVR::SP && AVR::DREGSRegClass.contains(DestReg)) {
  77:       Opc = AVR::SPREAD;
  78:     } else if (DestReg == AVR::SP && AVR::DREGSRegClass.contains(SrcReg)) {
  79:       Opc = AVR::SPWRITE;
  80:     } else {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81:       llvm_unreachable("Impossible reg-to-reg copy");
  82:     }
  83: 
  84:     BuildMI(MBB, MI, DL, get(Opc), DestReg)
  85:         .addReg(SrcReg, getKillRegState(KillSrc));
  86:   }
  87: }
  88: 
  89: Register AVRInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
  90:                                            int &FrameIndex) const {
  91:   switch (MI.getOpcode()) {
  92:   case AVR::LDDRdPtrQ:
  93:   case AVR::LDDWRdYQ: { //: FIXME: remove this once PR13375 gets fixed
  94:     if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
  95:         MI.getOperand(2).getImm() == 0) {
  96:       FrameIndex = MI.getOperand(1).getIndex();
  97:       return MI.getOperand(0).getReg();
  98:     }
  99:     break;
 100:   }
```

- EN: Function bodies or method definitions such as isLoadFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isLoadFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-120

```cpp
 101:   default:
 102:     break;
 103:   }
 104: 
 105:   return 0;
 106: }
 107: 
 108: Register AVRInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
 109:                                           int &FrameIndex) const {
 110:   switch (MI.getOpcode()) {
 111:   case AVR::STDPtrQRr:
 112:   case AVR::STDWPtrQRr: {
 113:     if (MI.getOperand(0).isFI() && MI.getOperand(1).isImm() &&
 114:         MI.getOperand(1).getImm() == 0) {
 115:       FrameIndex = MI.getOperand(0).getIndex();
 116:       return MI.getOperand(2).getReg();
 117:     }
 118:     break;
 119:   }
 120:   default:
```

- EN: Function bodies or method definitions such as isStoreToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isStoreToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 121-140

```cpp
 121:     break;
 122:   }
 123: 
 124:   return 0;
 125: }
 126: 
 127: void AVRInstrInfo::storeRegToStackSlot(
 128:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register SrcReg,
 129:     bool isKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
 130:     MachineInstr::MIFlag Flags) const {
 131:   MachineFunction &MF = *MBB.getParent();
 132:   AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
 133: 
 134:   AFI->setHasSpills(true);
 135: 
 136:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 137: 
 138:   MachineMemOperand *MMO = MF.getMachineMemOperand(
 139:       MachinePointerInfo::getFixedStack(MF, FrameIndex),
 140:       MachineMemOperand::MOStore, MFI.getObjectSize(FrameIndex),
```

- EN: Function bodies or method definitions such as storeRegToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: storeRegToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 141-160

```cpp
 141:       MFI.getObjectAlign(FrameIndex));
 142: 
 143:   unsigned Opcode = 0;
 144:   if (RI.isTypeLegalForClass(*RC, MVT::i8)) {
 145:     Opcode = AVR::STDPtrQRr;
 146:   } else if (RI.isTypeLegalForClass(*RC, MVT::i16)) {
 147:     Opcode = AVR::STDWPtrQRr;
 148:   } else {
 149:     llvm_unreachable("Cannot store this register into a stack slot!");
 150:   }
 151: 
 152:   BuildMI(MBB, MI, DebugLoc(), get(Opcode))
 153:       .addFrameIndex(FrameIndex)
 154:       .addImm(0)
 155:       .addReg(SrcReg, getKillRegState(isKill))
 156:       .addMemOperand(MMO);
 157: }
 158: 
 159: void AVRInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
 160:                                         MachineBasicBlock::iterator MI,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 161-180

```cpp
 161:                                         Register DestReg, int FrameIndex,
 162:                                         const TargetRegisterClass *RC,
 163:                                         Register VReg, unsigned SubReg,
 164:                                         MachineInstr::MIFlag Flags) const {
 165:   MachineFunction &MF = *MBB.getParent();
 166:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 167: 
 168:   MachineMemOperand *MMO = MF.getMachineMemOperand(
 169:       MachinePointerInfo::getFixedStack(MF, FrameIndex),
 170:       MachineMemOperand::MOLoad, MFI.getObjectSize(FrameIndex),
 171:       MFI.getObjectAlign(FrameIndex));
 172: 
 173:   unsigned Opcode = 0;
 174:   if (TRI.isTypeLegalForClass(*RC, MVT::i8)) {
 175:     Opcode = AVR::LDDRdPtrQ;
 176:   } else if (TRI.isTypeLegalForClass(*RC, MVT::i16)) {
 177:     // Opcode = AVR::LDDWRdPtrQ;
 178:     //: FIXME: remove this once PR13375 gets fixed
 179:     Opcode = AVR::LDDWRdYQ;
 180:   } else {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 181-200

```cpp
 181:     llvm_unreachable("Cannot load this register from a stack slot!");
 182:   }
 183: 
 184:   BuildMI(MBB, MI, DebugLoc(), get(Opcode), DestReg)
 185:       .addFrameIndex(FrameIndex)
 186:       .addImm(0)
 187:       .addMemOperand(MMO);
 188: }
 189: 
 190: const MCInstrDesc &AVRInstrInfo::getBrCond(AVRCC::CondCodes CC) const {
 191:   switch (CC) {
 192:   default:
 193:     llvm_unreachable("Unknown condition code!");
 194:   case AVRCC::COND_EQ:
 195:     return get(AVR::BREQk);
 196:   case AVRCC::COND_NE:
 197:     return get(AVR::BRNEk);
 198:   case AVRCC::COND_GE:
 199:     return get(AVR::BRGEk);
 200:   case AVRCC::COND_LT:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 201-220

```cpp
 201:     return get(AVR::BRLTk);
 202:   case AVRCC::COND_SH:
 203:     return get(AVR::BRSHk);
 204:   case AVRCC::COND_LO:
 205:     return get(AVR::BRLOk);
 206:   case AVRCC::COND_MI:
 207:     return get(AVR::BRMIk);
 208:   case AVRCC::COND_PL:
 209:     return get(AVR::BRPLk);
 210:   }
 211: }
 212: 
 213: AVRCC::CondCodes AVRInstrInfo::getCondFromBranchOpc(unsigned Opc) const {
 214:   switch (Opc) {
 215:   default:
 216:     return AVRCC::COND_INVALID;
 217:   case AVR::BREQk:
 218:     return AVRCC::COND_EQ;
 219:   case AVR::BRNEk:
 220:     return AVRCC::COND_NE;
```

- EN: Function bodies or method definitions such as getCondFromBranchOpc contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getCondFromBranchOpc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 221-240

```cpp
 221:   case AVR::BRSHk:
 222:     return AVRCC::COND_SH;
 223:   case AVR::BRLOk:
 224:     return AVRCC::COND_LO;
 225:   case AVR::BRMIk:
 226:     return AVRCC::COND_MI;
 227:   case AVR::BRPLk:
 228:     return AVRCC::COND_PL;
 229:   case AVR::BRGEk:
 230:     return AVRCC::COND_GE;
 231:   case AVR::BRLTk:
 232:     return AVRCC::COND_LT;
 233:   }
 234: }
 235: 
 236: AVRCC::CondCodes AVRInstrInfo::getOppositeCondition(AVRCC::CondCodes CC) const {
 237:   switch (CC) {
 238:   default:
 239:     llvm_unreachable("Invalid condition!");
 240:   case AVRCC::COND_EQ:
```

- EN: Function bodies or method definitions such as getOppositeCondition contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getOppositeCondition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 241-260

```cpp
 241:     return AVRCC::COND_NE;
 242:   case AVRCC::COND_NE:
 243:     return AVRCC::COND_EQ;
 244:   case AVRCC::COND_SH:
 245:     return AVRCC::COND_LO;
 246:   case AVRCC::COND_LO:
 247:     return AVRCC::COND_SH;
 248:   case AVRCC::COND_GE:
 249:     return AVRCC::COND_LT;
 250:   case AVRCC::COND_LT:
 251:     return AVRCC::COND_GE;
 252:   case AVRCC::COND_MI:
 253:     return AVRCC::COND_PL;
 254:   case AVRCC::COND_PL:
 255:     return AVRCC::COND_MI;
 256:   }
 257: }
 258: 
 259: bool AVRInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 260:                                  MachineBasicBlock *&TBB,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 261-280

```cpp
 261:                                  MachineBasicBlock *&FBB,
 262:                                  SmallVectorImpl<MachineOperand> &Cond,
 263:                                  bool AllowModify) const {
 264:   // Start from the bottom of the block and work up, examining the
 265:   // terminator instructions.
 266:   MachineBasicBlock::iterator I = MBB.end();
 267:   MachineBasicBlock::iterator UnCondBrIter = MBB.end();
 268: 
 269:   while (I != MBB.begin()) {
 270:     --I;
 271:     if (I->isDebugInstr()) {
 272:       continue;
 273:     }
 274: 
 275:     // Working from the bottom, when we see a non-terminator
 276:     // instruction, we're done.
 277:     if (!isUnpredicatedTerminator(*I)) {
 278:       break;
 279:     }
 280: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 281-300

```cpp
 281:     // A terminator that isn't a branch can't easily be handled
 282:     // by this analysis.
 283:     if (!I->getDesc().isBranch()) {
 284:       return true;
 285:     }
 286: 
 287:     // Handle unconditional branches.
 288:     //: TODO: add here jmp
 289:     if (I->getOpcode() == AVR::RJMPk) {
 290:       UnCondBrIter = I;
 291: 
 292:       if (!AllowModify) {
 293:         TBB = I->getOperand(0).getMBB();
 294:         continue;
 295:       }
 296: 
 297:       // If the block has any instructions after a JMP, delete them.
 298:       MBB.erase(std::next(I), MBB.end());
 299: 
 300:       Cond.clear();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 301-320

```cpp
 301:       FBB = nullptr;
 302: 
 303:       // Delete the JMP if it's equivalent to a fall-through.
 304:       if (MBB.isLayoutSuccessor(I->getOperand(0).getMBB())) {
 305:         TBB = nullptr;
 306:         I->eraseFromParent();
 307:         I = MBB.end();
 308:         UnCondBrIter = MBB.end();
 309:         continue;
 310:       }
 311: 
 312:       // TBB is used to indicate the unconditinal destination.
 313:       TBB = I->getOperand(0).getMBB();
 314:       continue;
 315:     }
 316: 
 317:     // Handle conditional branches.
 318:     AVRCC::CondCodes BranchCode = getCondFromBranchOpc(I->getOpcode());
 319:     if (BranchCode == AVRCC::COND_INVALID) {
 320:       return true; // Can't handle indirect branch.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 321-340

```cpp
 321:     }
 322: 
 323:     // Working from the bottom, handle the first conditional branch.
 324:     if (Cond.empty()) {
 325:       MachineBasicBlock *TargetBB = I->getOperand(0).getMBB();
 326:       if (AllowModify && UnCondBrIter != MBB.end() &&
 327:           MBB.isLayoutSuccessor(TargetBB)) {
 328:         // If we can modify the code and it ends in something like:
 329:         //
 330:         //     jCC L1
 331:         //     jmp L2
 332:         //   L1:
 333:         //     ...
 334:         //   L2:
 335:         //
 336:         // Then we can change this to:
 337:         //
 338:         //     jnCC L2
 339:         //   L1:
 340:         //     ...
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 341-360

```cpp
 341:         //   L2:
 342:         //
 343:         // Which is a bit more efficient.
 344:         // We conditionally jump to the fall-through block.
 345:         BranchCode = getOppositeCondition(BranchCode);
 346:         unsigned JNCC = getBrCond(BranchCode).getOpcode();
 347:         MachineBasicBlock::iterator OldInst = I;
 348: 
 349:         BuildMI(MBB, UnCondBrIter, MBB.findDebugLoc(I), get(JNCC))
 350:             .addMBB(UnCondBrIter->getOperand(0).getMBB());
 351:         BuildMI(MBB, UnCondBrIter, MBB.findDebugLoc(I), get(AVR::RJMPk))
 352:             .addMBB(TargetBB);
 353: 
 354:         OldInst->eraseFromParent();
 355:         UnCondBrIter->eraseFromParent();
 356: 
 357:         // Restart the analysis.
 358:         UnCondBrIter = MBB.end();
 359:         I = MBB.end();
 360:         continue;
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 361-380

```cpp
 361:       }
 362: 
 363:       FBB = TBB;
 364:       TBB = I->getOperand(0).getMBB();
 365:       Cond.push_back(MachineOperand::CreateImm(BranchCode));
 366:       continue;
 367:     }
 368: 
 369:     // Handle subsequent conditional branches. Only handle the case where all
 370:     // conditional branches branch to the same destination.
 371:     assert(Cond.size() == 1);
 372:     assert(TBB);
 373: 
 374:     // Only handle the case where all conditional branches branch to
 375:     // the same destination.
 376:     if (TBB != I->getOperand(0).getMBB()) {
 377:       return true;
 378:     }
 379: 
 380:     AVRCC::CondCodes OldBranchCode = (AVRCC::CondCodes)Cond[0].getImm();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 381-400

```cpp
 381:     // If the conditions are the same, we can leave them alone.
 382:     if (OldBranchCode == BranchCode) {
 383:       continue;
 384:     }
 385: 
 386:     return true;
 387:   }
 388: 
 389:   return false;
 390: }
 391: 
 392: unsigned AVRInstrInfo::insertBranch(MachineBasicBlock &MBB,
 393:                                     MachineBasicBlock *TBB,
 394:                                     MachineBasicBlock *FBB,
 395:                                     ArrayRef<MachineOperand> Cond,
 396:                                     const DebugLoc &DL, int *BytesAdded) const {
 397:   if (BytesAdded)
 398:     *BytesAdded = 0;
 399: 
 400:   // Shouldn't be a fall through.
```

- EN: Function bodies or method definitions such as insertBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: insertBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 401-420

```cpp
 401:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
 402:   assert((Cond.size() == 1 || Cond.size() == 0) &&
 403:          "AVR branch conditions have one component!");
 404: 
 405:   if (Cond.empty()) {
 406:     assert(!FBB && "Unconditional branch with multiple successors!");
 407:     auto &MI = *BuildMI(&MBB, DL, get(AVR::RJMPk)).addMBB(TBB);
 408:     if (BytesAdded)
 409:       *BytesAdded += getInstSizeInBytes(MI);
 410:     return 1;
 411:   }
 412: 
 413:   // Conditional branch.
 414:   unsigned Count = 0;
 415:   AVRCC::CondCodes CC = (AVRCC::CondCodes)Cond[0].getImm();
 416:   auto &CondMI = *BuildMI(&MBB, DL, getBrCond(CC)).addMBB(TBB);
 417: 
 418:   if (BytesAdded)
 419:     *BytesAdded += getInstSizeInBytes(CondMI);
 420:   ++Count;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 421-440

```cpp
 421: 
 422:   if (FBB) {
 423:     // Two-way Conditional branch. Insert the second branch.
 424:     auto &MI = *BuildMI(&MBB, DL, get(AVR::RJMPk)).addMBB(FBB);
 425:     if (BytesAdded)
 426:       *BytesAdded += getInstSizeInBytes(MI);
 427:     ++Count;
 428:   }
 429: 
 430:   return Count;
 431: }
 432: 
 433: unsigned AVRInstrInfo::removeBranch(MachineBasicBlock &MBB,
 434:                                     int *BytesRemoved) const {
 435:   if (BytesRemoved)
 436:     *BytesRemoved = 0;
 437: 
 438:   MachineBasicBlock::iterator I = MBB.end();
 439:   unsigned Count = 0;
 440: 
```

- EN: Function bodies or method definitions such as removeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: removeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 441-460

```cpp
 441:   while (I != MBB.begin()) {
 442:     --I;
 443:     if (I->isDebugInstr()) {
 444:       continue;
 445:     }
 446:     //: TODO: add here the missing jmp instructions once they are implemented
 447:     // like jmp, {e}ijmp, and other cond branches, ...
 448:     if (I->getOpcode() != AVR::RJMPk &&
 449:         getCondFromBranchOpc(I->getOpcode()) == AVRCC::COND_INVALID) {
 450:       break;
 451:     }
 452: 
 453:     // Remove the branch.
 454:     if (BytesRemoved)
 455:       *BytesRemoved += getInstSizeInBytes(*I);
 456:     I->eraseFromParent();
 457:     I = MBB.end();
 458:     ++Count;
 459:   }
 460: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 461-480

```cpp
 461:   return Count;
 462: }
 463: 
 464: bool AVRInstrInfo::reverseBranchCondition(
 465:     SmallVectorImpl<MachineOperand> &Cond) const {
 466:   assert(Cond.size() == 1 && "Invalid AVR branch condition!");
 467: 
 468:   AVRCC::CondCodes CC = static_cast<AVRCC::CondCodes>(Cond[0].getImm());
 469:   Cond[0].setImm(getOppositeCondition(CC));
 470: 
 471:   return false;
 472: }
 473: 
 474: unsigned AVRInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
 475:   unsigned Opcode = MI.getOpcode();
 476: 
 477:   switch (Opcode) {
 478:   // A regular instruction
 479:   default: {
 480:     const MCInstrDesc &Desc = get(Opcode);
```

- EN: Function bodies or method definitions such as reverseBranchCondition, getInstSizeInBytes contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: reverseBranchCondition, getInstSizeInBytes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 481-500

```cpp
 481:     return Desc.getSize();
 482:   }
 483:   case TargetOpcode::EH_LABEL:
 484:   case TargetOpcode::IMPLICIT_DEF:
 485:   case TargetOpcode::KILL:
 486:   case TargetOpcode::DBG_VALUE:
 487:     return 0;
 488:   case TargetOpcode::INLINEASM:
 489:   case TargetOpcode::INLINEASM_BR: {
 490:     const MachineFunction &MF = *MI.getParent()->getParent();
 491:     const AVRTargetMachine &TM =
 492:         static_cast<const AVRTargetMachine &>(MF.getTarget());
 493:     const TargetInstrInfo &TII = *STI.getInstrInfo();
 494:     return TII.getInlineAsmLength(MI.getOperand(0).getSymbolName(),
 495:                                   TM.getMCAsmInfo());
 496:   }
 497:   }
 498: }
 499: 
 500: MachineBasicBlock *
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 501-520

```cpp
 501: AVRInstrInfo::getBranchDestBlock(const MachineInstr &MI) const {
 502:   switch (MI.getOpcode()) {
 503:   default:
 504:     llvm_unreachable("unexpected opcode!");
 505:   case AVR::JMPk:
 506:   case AVR::CALLk:
 507:   case AVR::RCALLk:
 508:   case AVR::RJMPk:
 509:   case AVR::BREQk:
 510:   case AVR::BRNEk:
 511:   case AVR::BRSHk:
 512:   case AVR::BRLOk:
 513:   case AVR::BRMIk:
 514:   case AVR::BRPLk:
 515:   case AVR::BRGEk:
 516:   case AVR::BRLTk:
 517:     return MI.getOperand(0).getMBB();
 518:   case AVR::BRBSsk:
 519:   case AVR::BRBCsk:
 520:     return MI.getOperand(1).getMBB();
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 521-540

```cpp
 521:   case AVR::SBRCRrB:
 522:   case AVR::SBRSRrB:
 523:   case AVR::SBICAb:
 524:   case AVR::SBISAb:
 525:     llvm_unreachable("unimplemented branch instructions");
 526:   }
 527: }
 528: 
 529: bool AVRInstrInfo::isBranchOffsetInRange(unsigned BranchOp,
 530:                                          int64_t BrOffset) const {
 531: 
 532:   switch (BranchOp) {
 533:   default:
 534:     llvm_unreachable("unexpected opcode!");
 535:   case AVR::JMPk:
 536:   case AVR::CALLk:
 537:     return STI.hasJMPCALL();
 538:   case AVR::RCALLk:
 539:   case AVR::RJMPk:
 540:     return isIntN(13, BrOffset);
```

- EN: Function bodies or method definitions such as isBranchOffsetInRange contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isBranchOffsetInRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 541-560

```cpp
 541:   case AVR::BRBSsk:
 542:   case AVR::BRBCsk:
 543:   case AVR::BREQk:
 544:   case AVR::BRNEk:
 545:   case AVR::BRSHk:
 546:   case AVR::BRLOk:
 547:   case AVR::BRMIk:
 548:   case AVR::BRPLk:
 549:   case AVR::BRGEk:
 550:   case AVR::BRLTk:
 551:     return isIntN(7, BrOffset);
 552:   }
 553: }
 554: 
 555: void AVRInstrInfo::insertIndirectBranch(MachineBasicBlock &MBB,
 556:                                         MachineBasicBlock &NewDestBB,
 557:                                         MachineBasicBlock &RestoreBB,
 558:                                         const DebugLoc &DL, int64_t BrOffset,
 559:                                         RegScavenger *RS) const {
 560:   // This method inserts a *direct* branch (JMP), despite its name.
```

- EN: Function bodies or method definitions such as insertIndirectBranch contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: insertIndirectBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 561-574

```cpp
 561:   // LLVM calls this method to fixup unconditional branches; it never calls
 562:   // insertBranch or some hypothetical "insertDirectBranch".
 563:   // See lib/CodeGen/RegisterRelaxation.cpp for details.
 564:   // We end up here when a jump is too long for a RJMP instruction.
 565:   if (STI.hasJMPCALL())
 566:     BuildMI(&MBB, DL, get(AVR::JMPk)).addMBB(&NewDestBB);
 567:   else
 568:     // The RJMP may jump to a far place beyond its legal range. We let the
 569:     // linker to report 'out of range' rather than crash, or silently emit
 570:     // incorrect assembly code.
 571:     BuildMI(&MBB, DL, get(AVR::RJMPk)).addMBB(&NewDestBB);
 572: }
 573: 
 574: } // end of namespace llvm
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRInstrInfo.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`, `llvm/MC/MCContext.h`, `llvm/Support/ErrorHandling.h`, `AVR.h`, `AVRMachineFunctionInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Generated or companion files / 生成或配套文件: `AVRGenInstrInfo.inc`
- Local companions / 本地配套文件: `AVRInstrInfo.h`, `AVRInstrInfo.td`
