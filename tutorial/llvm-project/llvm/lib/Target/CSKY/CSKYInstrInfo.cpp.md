# CSKYInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the CSKY implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYInstrInfo.h - CSKY Instruction Information --------*- C++ -*---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the CSKY implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYInstrInfo.h"
  14: #include "CSKYConstantPoolValue.h"
  15: #include "CSKYMachineFunctionInfo.h"
  16: #include "CSKYTargetMachine.h"
  17: #include "llvm/CodeGen/MachineFrameInfo.h"
  18: #include "llvm/MC/MCContext.h"
  19: 
  20: #define DEBUG_TYPE "csky-instr-info"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: 
  22: using namespace llvm;
  23: 
  24: #define GET_INSTRINFO_CTOR_DTOR
  25: #include "CSKYGenInstrInfo.inc"
  26: 
  27: CSKYInstrInfo::CSKYInstrInfo(const CSKYSubtarget &STI,
  28:                              const CSKYRegisterInfo &TRI)
  29:     : CSKYGenInstrInfo(STI, TRI, CSKY::ADJCALLSTACKDOWN, CSKY::ADJCALLSTACKUP),
  30:       STI(STI) {
  31:   v2sf = STI.hasFPUv2SingleFloat();
  32:   v2df = STI.hasFPUv2DoubleFloat();
  33:   v3sf = STI.hasFPUv3SingleFloat();
  34:   v3df = STI.hasFPUv3DoubleFloat();
  35: }
  36: 
  37: static void parseCondBranch(MachineInstr &LastInst, MachineBasicBlock *&Target,
  38:                             SmallVectorImpl<MachineOperand> &Cond) {
  39:   // Block ends with fall-through condbranch.
  40:   assert(LastInst.getDesc().isConditionalBranch() &&
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as CSKYGenInstrInfo, parseCondBranch contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 CSKYGenInstrInfo, parseCondBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:          "Unknown conditional branch");
  42:   Target = LastInst.getOperand(1).getMBB();
  43:   Cond.push_back(MachineOperand::CreateImm(LastInst.getOpcode()));
  44:   Cond.push_back(LastInst.getOperand(0));
  45: }
  46: 
  47: bool CSKYInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
  48:                                   MachineBasicBlock *&TBB,
  49:                                   MachineBasicBlock *&FBB,
  50:                                   SmallVectorImpl<MachineOperand> &Cond,
  51:                                   bool AllowModify) const {
  52:   TBB = FBB = nullptr;
  53:   Cond.clear();
  54: 
  55:   // If the block has no terminators, it just falls into the block after it.
  56:   MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
  57:   if (I == MBB.end() || !isUnpredicatedTerminator(*I))
  58:     return false;
  59: 
  60:   // Count the number of terminators and find the first unconditional or
```

- EN: Function bodies or method definitions such as analyzeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: analyzeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:   // indirect branch.
  62:   MachineBasicBlock::iterator FirstUncondOrIndirectBr = MBB.end();
  63:   int NumTerminators = 0;
  64:   for (auto J = I.getReverse(); J != MBB.rend() && isUnpredicatedTerminator(*J);
  65:        J++) {
  66:     NumTerminators++;
  67:     if (J->getDesc().isUnconditionalBranch() ||
  68:         J->getDesc().isIndirectBranch()) {
  69:       FirstUncondOrIndirectBr = J.getReverse();
  70:     }
  71:   }
  72: 
  73:   // If AllowModify is true, we can erase any terminators after
  74:   // FirstUncondOrIndirectBR.
  75:   if (AllowModify && FirstUncondOrIndirectBr != MBB.end()) {
  76:     while (std::next(FirstUncondOrIndirectBr) != MBB.end()) {
  77:       std::next(FirstUncondOrIndirectBr)->eraseFromParent();
  78:       NumTerminators--;
  79:     }
  80:     I = FirstUncondOrIndirectBr;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81:   }
  82: 
  83:   // We can't handle blocks that end in an indirect branch.
  84:   if (I->getDesc().isIndirectBranch())
  85:     return true;
  86: 
  87:   // We can't handle blocks with more than 2 terminators.
  88:   if (NumTerminators > 2)
  89:     return true;
  90: 
  91:   // Handle a single unconditional branch.
  92:   if (NumTerminators == 1 && I->getDesc().isUnconditionalBranch()) {
  93:     TBB = getBranchDestBlock(*I);
  94:     return false;
  95:   }
  96: 
  97:   // Handle a single conditional branch.
  98:   if (NumTerminators == 1 && I->getDesc().isConditionalBranch()) {
  99:     parseCondBranch(*I, TBB, Cond);
 100:     return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-120

```cpp
 101:   }
 102: 
 103:   // Handle a conditional branch followed by an unconditional branch.
 104:   if (NumTerminators == 2 && std::prev(I)->getDesc().isConditionalBranch() &&
 105:       I->getDesc().isUnconditionalBranch()) {
 106:     parseCondBranch(*std::prev(I), TBB, Cond);
 107:     FBB = getBranchDestBlock(*I);
 108:     return false;
 109:   }
 110: 
 111:   // Otherwise, we can't handle this.
 112:   return true;
 113: }
 114: 
 115: unsigned CSKYInstrInfo::removeBranch(MachineBasicBlock &MBB,
 116:                                      int *BytesRemoved) const {
 117:   if (BytesRemoved)
 118:     *BytesRemoved = 0;
 119:   MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
 120:   if (I == MBB.end())
```

- EN: Function bodies or method definitions such as removeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: removeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 121-140

```cpp
 121:     return 0;
 122: 
 123:   if (!I->getDesc().isUnconditionalBranch() &&
 124:       !I->getDesc().isConditionalBranch())
 125:     return 0;
 126: 
 127:   // Remove the branch.
 128:   if (BytesRemoved)
 129:     *BytesRemoved += getInstSizeInBytes(*I);
 130:   I->eraseFromParent();
 131: 
 132:   I = MBB.end();
 133: 
 134:   if (I == MBB.begin())
 135:     return 1;
 136:   --I;
 137:   if (!I->getDesc().isConditionalBranch())
 138:     return 1;
 139: 
 140:   // Remove the branch.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 141-160

```cpp
 141:   if (BytesRemoved)
 142:     *BytesRemoved += getInstSizeInBytes(*I);
 143:   I->eraseFromParent();
 144:   return 2;
 145: }
 146: 
 147: MachineBasicBlock *
 148: CSKYInstrInfo::getBranchDestBlock(const MachineInstr &MI) const {
 149:   assert(MI.getDesc().isBranch() && "Unexpected opcode!");
 150:   // The branch target is always the last operand.
 151:   int NumOp = MI.getNumExplicitOperands();
 152:   assert(MI.getOperand(NumOp - 1).isMBB() && "Expected MBB!");
 153:   return MI.getOperand(NumOp - 1).getMBB();
 154: }
 155: 
 156: unsigned CSKYInstrInfo::insertBranch(
 157:     MachineBasicBlock &MBB, MachineBasicBlock *TBB, MachineBasicBlock *FBB,
 158:     ArrayRef<MachineOperand> Cond, const DebugLoc &DL, int *BytesAdded) const {
 159:   if (BytesAdded)
 160:     *BytesAdded = 0;
```

- EN: Function bodies or method definitions such as getBranchDestBlock, insertBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getBranchDestBlock, insertBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 161-180

```cpp
 161: 
 162:   // Shouldn't be a fall through.
 163:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
 164:   assert((Cond.size() == 2 || Cond.size() == 0) &&
 165:          "CSKY branch conditions have two components!");
 166: 
 167:   // Unconditional branch.
 168:   if (Cond.empty()) {
 169:     MachineInstr &MI = *BuildMI(&MBB, DL, get(CSKY::BR32)).addMBB(TBB);
 170:     if (BytesAdded)
 171:       *BytesAdded += getInstSizeInBytes(MI);
 172:     return 1;
 173:   }
 174: 
 175:   // Either a one or two-way conditional branch.
 176:   unsigned Opc = Cond[0].getImm();
 177:   MachineInstr &CondMI = *BuildMI(&MBB, DL, get(Opc)).add(Cond[1]).addMBB(TBB);
 178:   if (BytesAdded)
 179:     *BytesAdded += getInstSizeInBytes(CondMI);
 180: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 181-200

```cpp
 181:   // One-way conditional branch.
 182:   if (!FBB)
 183:     return 1;
 184: 
 185:   // Two-way conditional branch.
 186:   MachineInstr &MI = *BuildMI(&MBB, DL, get(CSKY::BR32)).addMBB(FBB);
 187:   if (BytesAdded)
 188:     *BytesAdded += getInstSizeInBytes(MI);
 189:   return 2;
 190: }
 191: 
 192: static unsigned getOppositeBranchOpc(unsigned Opcode) {
 193:   switch (Opcode) {
 194:   default:
 195:     llvm_unreachable("Unknown conditional branch!");
 196:   case CSKY::BT32:
 197:     return CSKY::BF32;
 198:   case CSKY::BT16:
 199:     return CSKY::BF16;
 200:   case CSKY::BF32:
```

- EN: Function bodies or method definitions such as getOppositeBranchOpc contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getOppositeBranchOpc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 201-220

```cpp
 201:     return CSKY::BT32;
 202:   case CSKY::BF16:
 203:     return CSKY::BT16;
 204:   case CSKY::BHZ32:
 205:     return CSKY::BLSZ32;
 206:   case CSKY::BHSZ32:
 207:     return CSKY::BLZ32;
 208:   case CSKY::BLZ32:
 209:     return CSKY::BHSZ32;
 210:   case CSKY::BLSZ32:
 211:     return CSKY::BHZ32;
 212:   case CSKY::BNEZ32:
 213:     return CSKY::BEZ32;
 214:   case CSKY::BEZ32:
 215:     return CSKY::BNEZ32;
 216:   }
 217: }
 218: 
 219: bool CSKYInstrInfo::reverseBranchCondition(
 220:     SmallVectorImpl<MachineOperand> &Cond) const {
```

- EN: Function bodies or method definitions such as reverseBranchCondition contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: reverseBranchCondition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 221-240

```cpp
 221:   assert((Cond.size() == 2) && "Invalid branch condition!");
 222:   Cond[0].setImm(getOppositeBranchOpc(Cond[0].getImm()));
 223:   return false;
 224: }
 225: 
 226: Register CSKYInstrInfo::movImm(MachineBasicBlock &MBB,
 227:                                MachineBasicBlock::iterator MBBI,
 228:                                const DebugLoc &DL, uint64_t Val,
 229:                                MachineInstr::MIFlag Flag) const {
 230:   if (!isInt<32>(Val))
 231:     report_fatal_error("Should only materialize 32-bit constants.");
 232: 
 233:   MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
 234: 
 235:   Register DstReg;
 236:   if (STI.hasE2()) {
 237:     DstReg = MRI.createVirtualRegister(&CSKY::GPRRegClass);
 238: 
 239:     if (isUInt<16>(Val)) {
 240:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVI32), DstReg)
```

- EN: Function bodies or method definitions such as movImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: movImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 241-260

```cpp
 241:           .addImm(Val & 0xFFFF)
 242:           .setMIFlags(Flag);
 243:     } else if (isShiftedUInt<16, 16>(Val)) {
 244:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVIH32), DstReg)
 245:           .addImm((Val >> 16) & 0xFFFF)
 246:           .setMIFlags(Flag);
 247:     } else {
 248:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVIH32), DstReg)
 249:           .addImm((Val >> 16) & 0xFFFF)
 250:           .setMIFlags(Flag);
 251:       BuildMI(MBB, MBBI, DL, get(CSKY::ORI32), DstReg)
 252:           .addReg(DstReg)
 253:           .addImm(Val & 0xFFFF)
 254:           .setMIFlags(Flag);
 255:     }
 256: 
 257:   } else {
 258:     DstReg = MRI.createVirtualRegister(&CSKY::mGPRRegClass);
 259:     if (isUInt<8>(Val)) {
 260:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVI16), DstReg)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 261-280

```cpp
 261:           .addImm(Val & 0xFF)
 262:           .setMIFlags(Flag);
 263:     } else if (isUInt<16>(Val)) {
 264:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVI16), DstReg)
 265:           .addImm((Val >> 8) & 0xFF)
 266:           .setMIFlags(Flag);
 267:       BuildMI(MBB, MBBI, DL, get(CSKY::LSLI16), DstReg)
 268:           .addReg(DstReg)
 269:           .addImm(8)
 270:           .setMIFlags(Flag);
 271:       if ((Val & 0xFF) != 0)
 272:         BuildMI(MBB, MBBI, DL, get(CSKY::ADDI16), DstReg)
 273:             .addReg(DstReg)
 274:             .addImm(Val & 0xFF)
 275:             .setMIFlags(Flag);
 276:     } else if (isUInt<24>(Val)) {
 277:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVI16), DstReg)
 278:           .addImm((Val >> 16) & 0xFF)
 279:           .setMIFlags(Flag);
 280:       BuildMI(MBB, MBBI, DL, get(CSKY::LSLI16), DstReg)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 281-300

```cpp
 281:           .addReg(DstReg)
 282:           .addImm(8)
 283:           .setMIFlags(Flag);
 284:       if (((Val >> 8) & 0xFF) != 0)
 285:         BuildMI(MBB, MBBI, DL, get(CSKY::ADDI16), DstReg)
 286:             .addReg(DstReg)
 287:             .addImm((Val >> 8) & 0xFF)
 288:             .setMIFlags(Flag);
 289:       BuildMI(MBB, MBBI, DL, get(CSKY::LSLI16), DstReg)
 290:           .addReg(DstReg)
 291:           .addImm(8)
 292:           .setMIFlags(Flag);
 293:       if ((Val & 0xFF) != 0)
 294:         BuildMI(MBB, MBBI, DL, get(CSKY::ADDI16), DstReg)
 295:             .addReg(DstReg)
 296:             .addImm(Val & 0xFF)
 297:             .setMIFlags(Flag);
 298:     } else {
 299:       BuildMI(MBB, MBBI, DL, get(CSKY::MOVI16), DstReg)
 300:           .addImm((Val >> 24) & 0xFF)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 301-320

```cpp
 301:           .setMIFlags(Flag);
 302:       BuildMI(MBB, MBBI, DL, get(CSKY::LSLI16), DstReg)
 303:           .addReg(DstReg)
 304:           .addImm(8)
 305:           .setMIFlags(Flag);
 306:       if (((Val >> 16) & 0xFF) != 0)
 307:         BuildMI(MBB, MBBI, DL, get(CSKY::ADDI16), DstReg)
 308:             .addReg(DstReg)
 309:             .addImm((Val >> 16) & 0xFF)
 310:             .setMIFlags(Flag);
 311:       BuildMI(MBB, MBBI, DL, get(CSKY::LSLI16), DstReg)
 312:           .addReg(DstReg)
 313:           .addImm(8)
 314:           .setMIFlags(Flag);
 315:       if (((Val >> 8) & 0xFF) != 0)
 316:         BuildMI(MBB, MBBI, DL, get(CSKY::ADDI16), DstReg)
 317:             .addReg(DstReg)
 318:             .addImm((Val >> 8) & 0xFF)
 319:             .setMIFlags(Flag);
 320:       BuildMI(MBB, MBBI, DL, get(CSKY::LSLI16), DstReg)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 321-340

```cpp
 321:           .addReg(DstReg)
 322:           .addImm(8)
 323:           .setMIFlags(Flag);
 324:       if ((Val & 0xFF) != 0)
 325:         BuildMI(MBB, MBBI, DL, get(CSKY::ADDI16), DstReg)
 326:             .addReg(DstReg)
 327:             .addImm(Val & 0xFF)
 328:             .setMIFlags(Flag);
 329:     }
 330:   }
 331: 
 332:   return DstReg;
 333: }
 334: 
 335: Register CSKYInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
 336:                                             int &FrameIndex) const {
 337:   switch (MI.getOpcode()) {
 338:   default:
 339:     return 0;
 340:   case CSKY::LD16B:
```

- EN: Function bodies or method definitions such as isLoadFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isLoadFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 341-360

```cpp
 341:   case CSKY::LD16H:
 342:   case CSKY::LD16W:
 343:   case CSKY::LD32B:
 344:   case CSKY::LD32BS:
 345:   case CSKY::LD32H:
 346:   case CSKY::LD32HS:
 347:   case CSKY::LD32W:
 348:   case CSKY::FLD_S:
 349:   case CSKY::FLD_D:
 350:   case CSKY::f2FLD_S:
 351:   case CSKY::f2FLD_D:
 352:   case CSKY::RESTORE_CARRY:
 353:     break;
 354:   }
 355: 
 356:   if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
 357:       MI.getOperand(2).getImm() == 0) {
 358:     FrameIndex = MI.getOperand(1).getIndex();
 359:     return MI.getOperand(0).getReg();
 360:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 361-380

```cpp
 361: 
 362:   return 0;
 363: }
 364: 
 365: Register CSKYInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
 366:                                            int &FrameIndex) const {
 367:   switch (MI.getOpcode()) {
 368:   default:
 369:     return 0;
 370:   case CSKY::ST16B:
 371:   case CSKY::ST16H:
 372:   case CSKY::ST16W:
 373:   case CSKY::ST32B:
 374:   case CSKY::ST32H:
 375:   case CSKY::ST32W:
 376:   case CSKY::FST_S:
 377:   case CSKY::FST_D:
 378:   case CSKY::f2FST_S:
 379:   case CSKY::f2FST_D:
 380:   case CSKY::SPILL_CARRY:
```

- EN: Function bodies or method definitions such as isStoreToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isStoreToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 381-400

```cpp
 381:     break;
 382:   }
 383: 
 384:   if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
 385:       MI.getOperand(2).getImm() == 0) {
 386:     FrameIndex = MI.getOperand(1).getIndex();
 387:     return MI.getOperand(0).getReg();
 388:   }
 389: 
 390:   return 0;
 391: }
 392: 
 393: void CSKYInstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
 394:                                         MachineBasicBlock::iterator I,
 395:                                         Register SrcReg, bool IsKill, int FI,
 396:                                         const TargetRegisterClass *RC,
 397:                                         Register VReg,
 398:                                         MachineInstr::MIFlag Flags) const {
 399:   DebugLoc DL;
 400:   if (I != MBB.end())
```

- EN: Function bodies or method definitions such as storeRegToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: storeRegToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 401-420

```cpp
 401:     DL = I->getDebugLoc();
 402: 
 403:   MachineFunction &MF = *MBB.getParent();
 404:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
 405:   MachineFrameInfo &MFI = MF.getFrameInfo();
 406: 
 407:   unsigned Opcode = 0;
 408: 
 409:   if (CSKY::GPRRegClass.hasSubClassEq(RC)) {
 410:     Opcode = CSKY::ST32W; // Optimize for 16bit
 411:   } else if (CSKY::CARRYRegClass.hasSubClassEq(RC)) {
 412:     Opcode = CSKY::SPILL_CARRY;
 413:     CFI->setSpillsCR();
 414:   } else if (v2sf && CSKY::sFPR32RegClass.hasSubClassEq(RC))
 415:     Opcode = CSKY::FST_S;
 416:   else if (v2df && CSKY::sFPR64RegClass.hasSubClassEq(RC))
 417:     Opcode = CSKY::FST_D;
 418:   else if (v3sf && CSKY::FPR32RegClass.hasSubClassEq(RC))
 419:     Opcode = CSKY::f2FST_S;
 420:   else if (v3df && CSKY::FPR64RegClass.hasSubClassEq(RC))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 421-440

```cpp
 421:     Opcode = CSKY::f2FST_D;
 422:   else {
 423:     llvm_unreachable("Unknown RegisterClass");
 424:   }
 425: 
 426:   MachineMemOperand *MMO = MF.getMachineMemOperand(
 427:       MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOStore,
 428:       MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
 429: 
 430:   BuildMI(MBB, I, DL, get(Opcode))
 431:       .addReg(SrcReg, getKillRegState(IsKill))
 432:       .addFrameIndex(FI)
 433:       .addImm(0)
 434:       .addMemOperand(MMO);
 435: }
 436: 
 437: void CSKYInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
 438:                                          MachineBasicBlock::iterator I,
 439:                                          Register DestReg, int FI,
 440:                                          const TargetRegisterClass *RC,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 441-460

```cpp
 441:                                          Register VReg, unsigned SubReg,
 442:                                          MachineInstr::MIFlag Flags) const {
 443:   DebugLoc DL;
 444:   if (I != MBB.end())
 445:     DL = I->getDebugLoc();
 446: 
 447:   MachineFunction &MF = *MBB.getParent();
 448:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
 449:   MachineFrameInfo &MFI = MF.getFrameInfo();
 450: 
 451:   unsigned Opcode = 0;
 452: 
 453:   if (CSKY::GPRRegClass.hasSubClassEq(RC)) {
 454:     Opcode = CSKY::LD32W;
 455:   } else if (CSKY::CARRYRegClass.hasSubClassEq(RC)) {
 456:     Opcode = CSKY::RESTORE_CARRY;
 457:     CFI->setSpillsCR();
 458:   } else if (v2sf && CSKY::sFPR32RegClass.hasSubClassEq(RC))
 459:     Opcode = CSKY::FLD_S;
 460:   else if (v2df && CSKY::sFPR64RegClass.hasSubClassEq(RC))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 461-480

```cpp
 461:     Opcode = CSKY::FLD_D;
 462:   else if (v3sf && CSKY::FPR32RegClass.hasSubClassEq(RC))
 463:     Opcode = CSKY::f2FLD_S;
 464:   else if (v3df && CSKY::FPR64RegClass.hasSubClassEq(RC))
 465:     Opcode = CSKY::f2FLD_D;
 466:   else {
 467:     llvm_unreachable("Unknown RegisterClass");
 468:   }
 469: 
 470:   MachineMemOperand *MMO = MF.getMachineMemOperand(
 471:       MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOLoad,
 472:       MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
 473: 
 474:   BuildMI(MBB, I, DL, get(Opcode), DestReg)
 475:       .addFrameIndex(FI)
 476:       .addImm(0)
 477:       .addMemOperand(MMO);
 478: }
 479: 
 480: void CSKYInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 481-500

```cpp
 481:                                 MachineBasicBlock::iterator I,
 482:                                 const DebugLoc &DL, Register DestReg,
 483:                                 Register SrcReg, bool KillSrc,
 484:                                 bool RenamableDest, bool RenamableSrc) const {
 485:   if (CSKY::GPRRegClass.contains(SrcReg) &&
 486:       CSKY::CARRYRegClass.contains(DestReg)) {
 487:     if (STI.hasE2()) {
 488:       BuildMI(MBB, I, DL, get(CSKY::BTSTI32), DestReg)
 489:           .addReg(SrcReg, getKillRegState(KillSrc))
 490:           .addImm(0);
 491:     } else {
 492:       assert(SrcReg < CSKY::R8);
 493:       BuildMI(MBB, I, DL, get(CSKY::BTSTI16), DestReg)
 494:           .addReg(SrcReg, getKillRegState(KillSrc))
 495:           .addImm(0);
 496:     }
 497:     return;
 498:   }
 499: 
 500:   if (CSKY::CARRYRegClass.contains(SrcReg) &&
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 501-520

```cpp
 501:       CSKY::GPRRegClass.contains(DestReg)) {
 502: 
 503:     if (STI.hasE2()) {
 504:       BuildMI(MBB, I, DL, get(CSKY::MVC32), DestReg)
 505:           .addReg(SrcReg, getKillRegState(KillSrc));
 506:     } else {
 507:       assert(DestReg < CSKY::R16);
 508:       assert(DestReg < CSKY::R8);
 509:       BuildMI(MBB, I, DL, get(CSKY::MOVI16), DestReg).addImm(0);
 510:       BuildMI(MBB, I, DL, get(CSKY::ADDC16))
 511:           .addReg(DestReg, RegState::Define)
 512:           .addReg(SrcReg, RegState::Define)
 513:           .addReg(DestReg, getKillRegState(true))
 514:           .addReg(DestReg, getKillRegState(true))
 515:           .addReg(SrcReg, getKillRegState(true));
 516:       BuildMI(MBB, I, DL, get(CSKY::BTSTI16))
 517:           .addReg(SrcReg, RegState::Define | getDeadRegState(KillSrc))
 518:           .addReg(DestReg)
 519:           .addImm(0);
 520:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 521-540

```cpp
 521:     return;
 522:   }
 523: 
 524:   unsigned Opcode = 0;
 525:   if (CSKY::GPRRegClass.contains(DestReg, SrcReg))
 526:     Opcode = STI.hasE2() ? CSKY::MOV32 : CSKY::MOV16;
 527:   else if (v2sf && CSKY::sFPR32RegClass.contains(DestReg, SrcReg))
 528:     Opcode = CSKY::FMOV_S;
 529:   else if (v3sf && CSKY::FPR32RegClass.contains(DestReg, SrcReg))
 530:     Opcode = CSKY::f2FMOV_S;
 531:   else if (v2df && CSKY::sFPR64RegClass.contains(DestReg, SrcReg))
 532:     Opcode = CSKY::FMOV_D;
 533:   else if (v3df && CSKY::FPR64RegClass.contains(DestReg, SrcReg))
 534:     Opcode = CSKY::f2FMOV_D;
 535:   else if (v2sf && CSKY::sFPR32RegClass.contains(SrcReg) &&
 536:            CSKY::GPRRegClass.contains(DestReg))
 537:     Opcode = CSKY::FMFVRL;
 538:   else if (v3sf && CSKY::FPR32RegClass.contains(SrcReg) &&
 539:            CSKY::GPRRegClass.contains(DestReg))
 540:     Opcode = CSKY::f2FMFVRL;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 541-560

```cpp
 541:   else if (v2df && CSKY::sFPR64RegClass.contains(SrcReg) &&
 542:            CSKY::GPRRegClass.contains(DestReg))
 543:     Opcode = CSKY::FMFVRL_D;
 544:   else if (v3df && CSKY::FPR64RegClass.contains(SrcReg) &&
 545:            CSKY::GPRRegClass.contains(DestReg))
 546:     Opcode = CSKY::f2FMFVRL_D;
 547:   else if (v2sf && CSKY::GPRRegClass.contains(SrcReg) &&
 548:            CSKY::sFPR32RegClass.contains(DestReg))
 549:     Opcode = CSKY::FMTVRL;
 550:   else if (v3sf && CSKY::GPRRegClass.contains(SrcReg) &&
 551:            CSKY::FPR32RegClass.contains(DestReg))
 552:     Opcode = CSKY::f2FMTVRL;
 553:   else if (v2df && CSKY::GPRRegClass.contains(SrcReg) &&
 554:            CSKY::sFPR64RegClass.contains(DestReg))
 555:     Opcode = CSKY::FMTVRL_D;
 556:   else if (v3df && CSKY::GPRRegClass.contains(SrcReg) &&
 557:            CSKY::FPR64RegClass.contains(DestReg))
 558:     Opcode = CSKY::f2FMTVRL_D;
 559:   else {
 560:     LLVM_DEBUG(dbgs() << "src = " << SrcReg << ", dst = " << DestReg);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 561-580

```cpp
 561:     LLVM_DEBUG(I->dump());
 562:     llvm_unreachable("Unknown RegisterClass");
 563:   }
 564: 
 565:   BuildMI(MBB, I, DL, get(Opcode), DestReg)
 566:       .addReg(SrcReg, getKillRegState(KillSrc));
 567: }
 568: 
 569: Register CSKYInstrInfo::getGlobalBaseReg(MachineFunction &MF) const {
 570:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
 571:   MachineConstantPool *MCP = MF.getConstantPool();
 572:   MachineRegisterInfo &MRI = MF.getRegInfo();
 573: 
 574:   Register GlobalBaseReg = CFI->getGlobalBaseReg();
 575:   if (GlobalBaseReg != 0)
 576:     return GlobalBaseReg;
 577: 
 578:   // Insert a pseudo instruction to set the GlobalBaseReg into the first
 579:   // MBB of the function
 580:   MachineBasicBlock &FirstMBB = MF.front();
```

- EN: Function bodies or method definitions such as getGlobalBaseReg contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getGlobalBaseReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 581-600

```cpp
 581:   MachineBasicBlock::iterator MBBI = FirstMBB.begin();
 582:   DebugLoc DL;
 583: 
 584:   CSKYConstantPoolValue *CPV = CSKYConstantPoolSymbol::Create(
 585:       Type::getInt32Ty(MF.getFunction().getContext()), "_GLOBAL_OFFSET_TABLE_",
 586:       0, CSKYCP::ADDR);
 587: 
 588:   unsigned CPI = MCP->getConstantPoolIndex(CPV, Align(4));
 589: 
 590:   MachineMemOperand *MO =
 591:       MF.getMachineMemOperand(MachinePointerInfo::getConstantPool(MF),
 592:                               MachineMemOperand::MOLoad, 4, Align(4));
 593:   BuildMI(FirstMBB, MBBI, DL, get(CSKY::LRW32), CSKY::R28)
 594:       .addConstantPoolIndex(CPI)
 595:       .addMemOperand(MO);
 596: 
 597:   GlobalBaseReg = MRI.createVirtualRegister(&CSKY::GPRRegClass);
 598:   BuildMI(FirstMBB, MBBI, DL, get(TargetOpcode::COPY), GlobalBaseReg)
 599:       .addReg(CSKY::R28);
 600: 
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 601-620

```cpp
 601:   CFI->setGlobalBaseReg(GlobalBaseReg);
 602:   return GlobalBaseReg;
 603: }
 604: 
 605: unsigned CSKYInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
 606:   switch (MI.getOpcode()) {
 607:   default:
 608:     return MI.getDesc().getSize();
 609:   case CSKY::CONSTPOOL_ENTRY:
 610:     return MI.getOperand(2).getImm();
 611:   case CSKY::SPILL_CARRY:
 612:   case CSKY::RESTORE_CARRY:
 613:   case CSKY::PseudoTLSLA32:
 614:     return 8;
 615:   case TargetOpcode::INLINEASM_BR:
 616:   case TargetOpcode::INLINEASM: {
 617:     const MachineFunction *MF = MI.getParent()->getParent();
 618:     const char *AsmStr = MI.getOperand(0).getSymbolName();
 619:     return getInlineAsmLength(AsmStr, MF->getTarget().getMCAsmInfo());
 620:   }
```

- EN: Function bodies or method definitions such as getInstSizeInBytes contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getInstSizeInBytes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 621-624

```cpp
 621:   case TargetOpcode::BUNDLE:
 622:     return getInstBundleSize(MI);
 623:   }
 624: }
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYInstrInfo.h`, `CSKYConstantPoolValue.h`, `CSKYMachineFunctionInfo.h`, `CSKYTargetMachine.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/MC/MCContext.h`, `CSKYGenInstrInfo.inc`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `CSKYGenInstrInfo.inc`
- Local companions / 本地配套文件: `CSKYInstrInfo.h`, `CSKYInstrInfo.td`
