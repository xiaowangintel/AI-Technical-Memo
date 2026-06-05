# LanaiInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiInstrInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the Lanai implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiInstrInfo.cpp - Lanai Instruction Information ------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the Lanai implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LanaiInstrInfo.h"
  14: #include "LanaiAluCode.h"
  15: #include "LanaiCondCode.h"
  16: #include "LanaiSubtarget.h"
  17: #include "MCTargetDesc/LanaiBaseInfo.h"
  18: #include "llvm/ADT/STLExtras.h"
  19: #include "llvm/ADT/SmallVector.h"
  20: #include "llvm/CodeGen/MachineInstrBuilder.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineRegisterInfo.h"
  22: #include "llvm/Support/ErrorHandling.h"
  23: 
  24: using namespace llvm;
  25: 
  26: #define GET_INSTRINFO_CTOR_DTOR
  27: #include "LanaiGenInstrInfo.inc"
  28: 
  29: LanaiInstrInfo::LanaiInstrInfo(const LanaiSubtarget &STI)
  30:     : LanaiGenInstrInfo(STI, RegisterInfo, Lanai::ADJCALLSTACKDOWN,
  31:                         Lanai::ADJCALLSTACKUP),
  32:       RegisterInfo() {}
  33: 
  34: void LanaiInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
  35:                                  MachineBasicBlock::iterator Position,
  36:                                  const DebugLoc &DL,
  37:                                  Register DestinationRegister,
  38:                                  Register SourceRegister, bool KillSource,
  39:                                  bool RenamableDest, bool RenamableSrc) const {
  40:   if (!Lanai::GPRRegClass.contains(DestinationRegister, SourceRegister)) {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as LanaiGenInstrInfo, copyPhysReg contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 LanaiGenInstrInfo, copyPhysReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:     llvm_unreachable("Impossible reg-to-reg copy");
  42:   }
  43: 
  44:   BuildMI(MBB, Position, DL, get(Lanai::OR_I_LO), DestinationRegister)
  45:       .addReg(SourceRegister, getKillRegState(KillSource))
  46:       .addImm(0);
  47: }
  48: 
  49: void LanaiInstrInfo::storeRegToStackSlot(
  50:     MachineBasicBlock &MBB, MachineBasicBlock::iterator Position,
  51:     Register SourceRegister, bool IsKill, int FrameIndex,
  52:     const TargetRegisterClass *RegisterClass, Register /*VReg*/,
  53:     MachineInstr::MIFlag /*Flags*/) const {
  54:   DebugLoc DL;
  55:   if (Position != MBB.end()) {
  56:     DL = Position->getDebugLoc();
  57:   }
  58: 
  59:   if (!Lanai::GPRRegClass.hasSubClassEq(RegisterClass)) {
  60:     llvm_unreachable("Can't store this register to stack slot");
```

- EN: Function bodies or method definitions such as storeRegToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: storeRegToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:   }
  62:   BuildMI(MBB, Position, DL, get(Lanai::SW_RI))
  63:       .addReg(SourceRegister, getKillRegState(IsKill))
  64:       .addFrameIndex(FrameIndex)
  65:       .addImm(0)
  66:       .addImm(LPAC::ADD);
  67: }
  68: 
  69: void LanaiInstrInfo::loadRegFromStackSlot(
  70:     MachineBasicBlock &MBB, MachineBasicBlock::iterator Position,
  71:     Register DestinationRegister, int FrameIndex,
  72:     const TargetRegisterClass *RegisterClass, Register /*VReg*/,
  73:     unsigned /*SubReg*/, MachineInstr::MIFlag /*Flags*/) const {
  74:   DebugLoc DL;
  75:   if (Position != MBB.end()) {
  76:     DL = Position->getDebugLoc();
  77:   }
  78: 
  79:   if (!Lanai::GPRRegClass.hasSubClassEq(RegisterClass)) {
  80:     llvm_unreachable("Can't load this register from stack slot");
```

- EN: Function bodies or method definitions such as loadRegFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: loadRegFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81:   }
  82:   BuildMI(MBB, Position, DL, get(Lanai::LDW_RI), DestinationRegister)
  83:       .addFrameIndex(FrameIndex)
  84:       .addImm(0)
  85:       .addImm(LPAC::ADD);
  86: }
  87: 
  88: bool LanaiInstrInfo::areMemAccessesTriviallyDisjoint(
  89:     const MachineInstr &MIa, const MachineInstr &MIb) const {
  90:   assert(MIa.mayLoadOrStore() && "MIa must be a load or store.");
  91:   assert(MIb.mayLoadOrStore() && "MIb must be a load or store.");
  92: 
  93:   if (MIa.hasUnmodeledSideEffects() || MIb.hasUnmodeledSideEffects() ||
  94:       MIa.hasOrderedMemoryRef() || MIb.hasOrderedMemoryRef())
  95:     return false;
  96: 
  97:   // Retrieve the base register, offset from the base register and width. Width
  98:   // is the size of memory that is being loaded/stored (e.g. 1, 2, 4).  If
  99:   // base registers are identical, and the offset of a lower memory access +
 100:   // the width doesn't overlap the offset of a higher memory access,
```

- EN: Function bodies or method definitions such as areMemAccessesTriviallyDisjoint contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: areMemAccessesTriviallyDisjoint 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-120

```cpp
 101:   // then the memory accesses are different.
 102:   const TargetRegisterInfo *TRI = &getRegisterInfo();
 103:   const MachineOperand *BaseOpA = nullptr, *BaseOpB = nullptr;
 104:   int64_t OffsetA = 0, OffsetB = 0;
 105:   LocationSize WidthA = LocationSize::precise(0),
 106:                WidthB = LocationSize::precise(0);
 107:   if (getMemOperandWithOffsetWidth(MIa, BaseOpA, OffsetA, WidthA, TRI) &&
 108:       getMemOperandWithOffsetWidth(MIb, BaseOpB, OffsetB, WidthB, TRI)) {
 109:     if (BaseOpA->isIdenticalTo(*BaseOpB)) {
 110:       int LowOffset = std::min(OffsetA, OffsetB);
 111:       int HighOffset = std::max(OffsetA, OffsetB);
 112:       LocationSize LowWidth = (LowOffset == OffsetA) ? WidthA : WidthB;
 113:       if (LowWidth.hasValue() &&
 114:           LowOffset + (int)LowWidth.getValue() <= HighOffset)
 115:         return true;
 116:     }
 117:   }
 118:   return false;
 119: }
 120: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 121-140

```cpp
 121: bool LanaiInstrInfo::expandPostRAPseudo(MachineInstr & /*MI*/) const {
 122:   return false;
 123: }
 124: 
 125: static LPCC::CondCode getOppositeCondition(LPCC::CondCode CC) {
 126:   switch (CC) {
 127:   case LPCC::ICC_T: //  true
 128:     return LPCC::ICC_F;
 129:   case LPCC::ICC_F: //  false
 130:     return LPCC::ICC_T;
 131:   case LPCC::ICC_HI: //  high
 132:     return LPCC::ICC_LS;
 133:   case LPCC::ICC_LS: //  low or same
 134:     return LPCC::ICC_HI;
 135:   case LPCC::ICC_CC: //  carry cleared
 136:     return LPCC::ICC_CS;
 137:   case LPCC::ICC_CS: //  carry set
 138:     return LPCC::ICC_CC;
 139:   case LPCC::ICC_NE: //  not equal
 140:     return LPCC::ICC_EQ;
```

- EN: Function bodies or method definitions such as expandPostRAPseudo, getOppositeCondition contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: expandPostRAPseudo, getOppositeCondition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 141-160

```cpp
 141:   case LPCC::ICC_EQ: //  equal
 142:     return LPCC::ICC_NE;
 143:   case LPCC::ICC_VC: //  oVerflow cleared
 144:     return LPCC::ICC_VS;
 145:   case LPCC::ICC_VS: //  oVerflow set
 146:     return LPCC::ICC_VC;
 147:   case LPCC::ICC_PL: //  plus (note: 0 is "minus" too here)
 148:     return LPCC::ICC_MI;
 149:   case LPCC::ICC_MI: //  minus
 150:     return LPCC::ICC_PL;
 151:   case LPCC::ICC_GE: //  greater than or equal
 152:     return LPCC::ICC_LT;
 153:   case LPCC::ICC_LT: //  less than
 154:     return LPCC::ICC_GE;
 155:   case LPCC::ICC_GT: //  greater than
 156:     return LPCC::ICC_LE;
 157:   case LPCC::ICC_LE: //  less than or equal
 158:     return LPCC::ICC_GT;
 159:   default:
 160:     llvm_unreachable("Invalid condtional code");
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 161-180

```cpp
 161:   }
 162: }
 163: 
 164: std::pair<unsigned, unsigned>
 165: LanaiInstrInfo::decomposeMachineOperandsTargetFlags(unsigned TF) const {
 166:   return std::make_pair(TF, 0u);
 167: }
 168: 
 169: ArrayRef<std::pair<unsigned, const char *>>
 170: LanaiInstrInfo::getSerializableDirectMachineOperandTargetFlags() const {
 171:   using namespace LanaiII;
 172:   static const std::pair<unsigned, const char *> TargetFlags[] = {
 173:       {MO_ABS_HI, "lanai-hi"},
 174:       {MO_ABS_LO, "lanai-lo"},
 175:       {MO_NO_FLAG, "lanai-nf"}};
 176:   return ArrayRef(TargetFlags);
 177: }
 178: 
 179: bool LanaiInstrInfo::analyzeCompare(const MachineInstr &MI, Register &SrcReg,
 180:                                     Register &SrcReg2, int64_t &CmpMask,
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 181-200

```cpp
 181:                                     int64_t &CmpValue) const {
 182:   switch (MI.getOpcode()) {
 183:   default:
 184:     break;
 185:   case Lanai::SFSUB_F_RI_LO:
 186:   case Lanai::SFSUB_F_RI_HI:
 187:     SrcReg = MI.getOperand(0).getReg();
 188:     SrcReg2 = Register();
 189:     CmpMask = ~0;
 190:     CmpValue = MI.getOperand(1).getImm();
 191:     return true;
 192:   case Lanai::SFSUB_F_RR:
 193:     SrcReg = MI.getOperand(0).getReg();
 194:     SrcReg2 = MI.getOperand(1).getReg();
 195:     CmpMask = ~0;
 196:     CmpValue = 0;
 197:     return true;
 198:   }
 199: 
 200:   return false;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 201-220

```cpp
 201: }
 202: 
 203: // isRedundantFlagInstr - check whether the first instruction, whose only
 204: // purpose is to update flags, can be made redundant.
 205: // * SFSUB_F_RR can be made redundant by SUB_RI if the operands are the same.
 206: // * SFSUB_F_RI can be made redundant by SUB_I if the operands are the same.
 207: inline static bool isRedundantFlagInstr(MachineInstr *CmpI, unsigned SrcReg,
 208:                                         unsigned SrcReg2, int64_t ImmValue,
 209:                                         MachineInstr *OI) {
 210:   if (CmpI->getOpcode() == Lanai::SFSUB_F_RR &&
 211:       OI->getOpcode() == Lanai::SUB_R &&
 212:       ((OI->getOperand(1).getReg() == SrcReg &&
 213:         OI->getOperand(2).getReg() == SrcReg2) ||
 214:        (OI->getOperand(1).getReg() == SrcReg2 &&
 215:         OI->getOperand(2).getReg() == SrcReg)))
 216:     return true;
 217: 
 218:   if (((CmpI->getOpcode() == Lanai::SFSUB_F_RI_LO &&
 219:         OI->getOpcode() == Lanai::SUB_I_LO) ||
 220:        (CmpI->getOpcode() == Lanai::SFSUB_F_RI_HI &&
```

- EN: Function bodies or method definitions such as isRedundantFlagInstr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isRedundantFlagInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 221-240

```cpp
 221:         OI->getOpcode() == Lanai::SUB_I_HI)) &&
 222:       OI->getOperand(1).getReg() == SrcReg &&
 223:       OI->getOperand(2).getImm() == ImmValue)
 224:     return true;
 225:   return false;
 226: }
 227: 
 228: inline static unsigned flagSettingOpcodeVariant(unsigned OldOpcode) {
 229:   switch (OldOpcode) {
 230:   case Lanai::ADD_I_HI:
 231:     return Lanai::ADD_F_I_HI;
 232:   case Lanai::ADD_I_LO:
 233:     return Lanai::ADD_F_I_LO;
 234:   case Lanai::ADD_R:
 235:     return Lanai::ADD_F_R;
 236:   case Lanai::ADDC_I_HI:
 237:     return Lanai::ADDC_F_I_HI;
 238:   case Lanai::ADDC_I_LO:
 239:     return Lanai::ADDC_F_I_LO;
 240:   case Lanai::ADDC_R:
```

- EN: Function bodies or method definitions such as flagSettingOpcodeVariant contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: flagSettingOpcodeVariant 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 241-260

```cpp
 241:     return Lanai::ADDC_F_R;
 242:   case Lanai::AND_I_HI:
 243:     return Lanai::AND_F_I_HI;
 244:   case Lanai::AND_I_LO:
 245:     return Lanai::AND_F_I_LO;
 246:   case Lanai::AND_R:
 247:     return Lanai::AND_F_R;
 248:   case Lanai::OR_I_HI:
 249:     return Lanai::OR_F_I_HI;
 250:   case Lanai::OR_I_LO:
 251:     return Lanai::OR_F_I_LO;
 252:   case Lanai::OR_R:
 253:     return Lanai::OR_F_R;
 254:   case Lanai::SL_I:
 255:     return Lanai::SL_F_I;
 256:   case Lanai::SRL_R:
 257:     return Lanai::SRL_F_R;
 258:   case Lanai::SA_I:
 259:     return Lanai::SA_F_I;
 260:   case Lanai::SRA_R:
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 261-280

```cpp
 261:     return Lanai::SRA_F_R;
 262:   case Lanai::SUB_I_HI:
 263:     return Lanai::SUB_F_I_HI;
 264:   case Lanai::SUB_I_LO:
 265:     return Lanai::SUB_F_I_LO;
 266:   case Lanai::SUB_R:
 267:     return Lanai::SUB_F_R;
 268:   case Lanai::SUBB_I_HI:
 269:     return Lanai::SUBB_F_I_HI;
 270:   case Lanai::SUBB_I_LO:
 271:     return Lanai::SUBB_F_I_LO;
 272:   case Lanai::SUBB_R:
 273:     return Lanai::SUBB_F_R;
 274:   case Lanai::XOR_I_HI:
 275:     return Lanai::XOR_F_I_HI;
 276:   case Lanai::XOR_I_LO:
 277:     return Lanai::XOR_F_I_LO;
 278:   case Lanai::XOR_R:
 279:     return Lanai::XOR_F_R;
 280:   default:
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 281-300

```cpp
 281:     return Lanai::NOP;
 282:   }
 283: }
 284: 
 285: bool LanaiInstrInfo::optimizeCompareInstr(
 286:     MachineInstr &CmpInstr, Register SrcReg, Register SrcReg2,
 287:     int64_t /*CmpMask*/, int64_t CmpValue,
 288:     const MachineRegisterInfo *MRI) const {
 289:   // Get the unique definition of SrcReg.
 290:   MachineInstr *MI = MRI->getUniqueVRegDef(SrcReg);
 291:   if (!MI)
 292:     return false;
 293: 
 294:   // Get ready to iterate backward from CmpInstr.
 295:   MachineBasicBlock::iterator I = CmpInstr, E = MI,
 296:                               B = CmpInstr.getParent()->begin();
 297: 
 298:   // Early exit if CmpInstr is at the beginning of the BB.
 299:   if (I == B)
 300:     return false;
```

- EN: Function bodies or method definitions such as optimizeCompareInstr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: optimizeCompareInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 301-320

```cpp
 301: 
 302:   // There are two possible candidates which can be changed to set SR:
 303:   // One is MI, the other is a SUB instruction.
 304:   // * For SFSUB_F_RR(r1,r2), we are looking for SUB(r1,r2) or SUB(r2,r1).
 305:   // * For SFSUB_F_RI(r1, CmpValue), we are looking for SUB(r1, CmpValue).
 306:   MachineInstr *Sub = nullptr;
 307:   if (SrcReg2 != 0)
 308:     // MI is not a candidate to transform into a flag setting instruction.
 309:     MI = nullptr;
 310:   else if (MI->getParent() != CmpInstr.getParent() || CmpValue != 0) {
 311:     // Conservatively refuse to convert an instruction which isn't in the same
 312:     // BB as the comparison. Don't return if SFSUB_F_RI and CmpValue != 0 as Sub
 313:     // may still be a candidate.
 314:     if (CmpInstr.getOpcode() == Lanai::SFSUB_F_RI_LO)
 315:       MI = nullptr;
 316:     else
 317:       return false;
 318:   }
 319: 
 320:   // Check that SR isn't set between the comparison instruction and the
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 321-340

```cpp
 321:   // instruction we want to change while searching for Sub.
 322:   const TargetRegisterInfo *TRI = &getRegisterInfo();
 323:   for (--I; I != E; --I) {
 324:     const MachineInstr &Instr = *I;
 325: 
 326:     if (Instr.modifiesRegister(Lanai::SR, TRI) ||
 327:         Instr.readsRegister(Lanai::SR, TRI))
 328:       // This instruction modifies or uses SR after the one we want to change.
 329:       // We can't do this transformation.
 330:       return false;
 331: 
 332:     // Check whether CmpInstr can be made redundant by the current instruction.
 333:     if (isRedundantFlagInstr(&CmpInstr, SrcReg, SrcReg2, CmpValue, &*I)) {
 334:       Sub = &*I;
 335:       break;
 336:     }
 337: 
 338:     // Don't search outside the containing basic block.
 339:     if (I == B)
 340:       return false;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 341-360

```cpp
 341:   }
 342: 
 343:   // Return false if no candidates exist.
 344:   if (!MI && !Sub)
 345:     return false;
 346: 
 347:   // The single candidate is called MI.
 348:   if (!MI)
 349:     MI = Sub;
 350: 
 351:   if (flagSettingOpcodeVariant(MI->getOpcode()) != Lanai::NOP) {
 352:     bool isSafe = false;
 353: 
 354:     SmallVector<std::pair<MachineOperand *, LPCC::CondCode>, 4>
 355:         OperandsToUpdate;
 356:     I = CmpInstr;
 357:     E = CmpInstr.getParent()->end();
 358:     while (!isSafe && ++I != E) {
 359:       const MachineInstr &Instr = *I;
 360:       for (unsigned IO = 0, EO = Instr.getNumOperands(); !isSafe && IO != EO;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 361-380

```cpp
 361:            ++IO) {
 362:         const MachineOperand &MO = Instr.getOperand(IO);
 363:         if (MO.isRegMask() && MO.clobbersPhysReg(Lanai::SR)) {
 364:           isSafe = true;
 365:           break;
 366:         }
 367:         if (!MO.isReg() || MO.getReg() != Lanai::SR)
 368:           continue;
 369:         if (MO.isDef()) {
 370:           isSafe = true;
 371:           break;
 372:         }
 373:         // Condition code is after the operand before SR.
 374:         LPCC::CondCode CC;
 375:         CC = (LPCC::CondCode)Instr.getOperand(IO - 1).getImm();
 376: 
 377:         if (Sub) {
 378:           LPCC::CondCode NewCC = getOppositeCondition(CC);
 379:           if (NewCC == LPCC::ICC_T)
 380:             return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 381-400

```cpp
 381:           // If we have SUB(r1, r2) and CMP(r2, r1), the condition code based on
 382:           // CMP needs to be updated to be based on SUB.  Push the condition
 383:           // code operands to OperandsToUpdate.  If it is safe to remove
 384:           // CmpInstr, the condition code of these operands will be modified.
 385:           if (SrcReg2 != 0 && Sub->getOperand(1).getReg() == SrcReg2 &&
 386:               Sub->getOperand(2).getReg() == SrcReg) {
 387:             OperandsToUpdate.push_back(
 388:                 std::make_pair(&((*I).getOperand(IO - 1)), NewCC));
 389:           }
 390:         } else {
 391:           // No Sub, so this is x = <op> y, z; cmp x, 0.
 392:           switch (CC) {
 393:           case LPCC::ICC_EQ: // Z
 394:           case LPCC::ICC_NE: // Z
 395:           case LPCC::ICC_MI: // N
 396:           case LPCC::ICC_PL: // N
 397:           case LPCC::ICC_F:  // none
 398:           case LPCC::ICC_T:  // none
 399:             // SR can be used multiple times, we should continue.
 400:             break;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 401-420

```cpp
 401:           case LPCC::ICC_CS: // C
 402:           case LPCC::ICC_CC: // C
 403:           case LPCC::ICC_VS: // V
 404:           case LPCC::ICC_VC: // V
 405:           case LPCC::ICC_HI: // C Z
 406:           case LPCC::ICC_LS: // C Z
 407:           case LPCC::ICC_GE: // N V
 408:           case LPCC::ICC_LT: // N V
 409:           case LPCC::ICC_GT: // Z N V
 410:           case LPCC::ICC_LE: // Z N V
 411:             // The instruction uses the V bit or C bit which is not safe.
 412:             return false;
 413:           case LPCC::UNKNOWN:
 414:             return false;
 415:           }
 416:         }
 417:       }
 418:     }
 419: 
 420:     // If SR is not killed nor re-defined, we should check whether it is
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 421-440

```cpp
 421:     // live-out. If it is live-out, do not optimize.
 422:     if (!isSafe) {
 423:       MachineBasicBlock *MBB = CmpInstr.getParent();
 424:       for (const MachineBasicBlock *Succ : MBB->successors())
 425:         if (Succ->isLiveIn(Lanai::SR))
 426:           return false;
 427:     }
 428: 
 429:     // Toggle the optional operand to SR.
 430:     MI->setDesc(get(flagSettingOpcodeVariant(MI->getOpcode())));
 431:     MI->addRegisterDefined(Lanai::SR);
 432:     CmpInstr.eraseFromParent();
 433:     return true;
 434:   }
 435: 
 436:   return false;
 437: }
 438: 
 439: // Identify instructions that can be folded into a SELECT instruction, and
 440: // return the defining instruction.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 441-460

```cpp
 441: static MachineInstr *canFoldIntoSelect(Register Reg,
 442:                                        const MachineRegisterInfo &MRI) {
 443:   if (!Reg.isVirtual())
 444:     return nullptr;
 445:   if (!MRI.hasOneNonDBGUse(Reg))
 446:     return nullptr;
 447:   MachineInstr *MI = MRI.getVRegDef(Reg);
 448:   if (!MI)
 449:     return nullptr;
 450:   // MI is folded into the SELECT by predicating it.
 451:   if (!MI->isPredicable())
 452:     return nullptr;
 453:   // Check if MI has any non-dead defs or physreg uses. This also detects
 454:   // predicated instructions which will be reading SR.
 455:   for (const MachineOperand &MO : llvm::drop_begin(MI->operands(), 1)) {
 456:     // Reject frame index operands.
 457:     if (MO.isFI() || MO.isCPI() || MO.isJTI())
 458:       return nullptr;
 459:     if (!MO.isReg())
 460:       continue;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 461-480

```cpp
 461:     // MI can't have any tied operands, that would conflict with predication.
 462:     if (MO.isTied())
 463:       return nullptr;
 464:     if (MO.getReg().isPhysical())
 465:       return nullptr;
 466:     if (MO.isDef() && !MO.isDead())
 467:       return nullptr;
 468:   }
 469:   bool DontMoveAcrossStores = true;
 470:   if (!MI->isSafeToMove(DontMoveAcrossStores))
 471:     return nullptr;
 472:   return MI;
 473: }
 474: 
 475: MachineInstr *
 476: LanaiInstrInfo::optimizeSelect(MachineInstr &MI,
 477:                                SmallPtrSetImpl<MachineInstr *> &SeenMIs,
 478:                                bool /*PreferFalse*/) const {
 479:   assert(MI.getOpcode() == Lanai::SELECT && "unknown select instruction");
 480:   MachineRegisterInfo &MRI = MI.getParent()->getParent()->getRegInfo();
```

- EN: Function bodies or method definitions such as optimizeSelect contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: optimizeSelect 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 481-500

```cpp
 481:   MachineInstr *DefMI = canFoldIntoSelect(MI.getOperand(1).getReg(), MRI);
 482:   bool Invert = !DefMI;
 483:   if (!DefMI)
 484:     DefMI = canFoldIntoSelect(MI.getOperand(2).getReg(), MRI);
 485:   if (!DefMI)
 486:     return nullptr;
 487: 
 488:   // Find new register class to use.
 489:   MachineOperand FalseReg = MI.getOperand(Invert ? 1 : 2);
 490:   Register DestReg = MI.getOperand(0).getReg();
 491:   const TargetRegisterClass *PreviousClass = MRI.getRegClass(FalseReg.getReg());
 492:   if (!MRI.constrainRegClass(DestReg, PreviousClass))
 493:     return nullptr;
 494: 
 495:   // Create a new predicated version of DefMI.
 496:   MachineInstrBuilder NewMI =
 497:       BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), DefMI->getDesc(), DestReg);
 498: 
 499:   // Copy all the DefMI operands, excluding its (null) predicate.
 500:   const MCInstrDesc &DefDesc = DefMI->getDesc();
```

- EN: This chunk introduces interfaces or data structures such as to, which organize the target-specific behavior exposed by the file. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一段引入了 to 等接口或数据结构，用于组织该文件暴露的目标专用行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 501-520

```cpp
 501:   for (unsigned i = 1, e = DefDesc.getNumOperands();
 502:        i != e && !DefDesc.operands()[i].isPredicate(); ++i)
 503:     NewMI.add(DefMI->getOperand(i));
 504: 
 505:   unsigned CondCode = MI.getOperand(3).getImm();
 506:   if (Invert)
 507:     NewMI.addImm(getOppositeCondition(LPCC::CondCode(CondCode)));
 508:   else
 509:     NewMI.addImm(CondCode);
 510:   NewMI.copyImplicitOps(MI);
 511: 
 512:   // The output register value when the predicate is false is an implicit
 513:   // register operand tied to the first def.  The tie makes the register
 514:   // allocator ensure the FalseReg is allocated the same register as operand 0.
 515:   FalseReg.setImplicit();
 516:   NewMI.add(FalseReg);
 517:   NewMI->tieOperands(0, NewMI->getNumOperands() - 1);
 518: 
 519:   // Update SeenMIs set: register newly created MI and erase removed DefMI.
 520:   SeenMIs.insert(NewMI);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 521-540

```cpp
 521:   SeenMIs.erase(DefMI);
 522: 
 523:   // If MI is inside a loop, and DefMI is outside the loop, then kill flags on
 524:   // DefMI would be invalid when transferred inside the loop.  Checking for a
 525:   // loop is expensive, but at least remove kill flags if they are in different
 526:   // BBs.
 527:   if (DefMI->getParent() != MI.getParent())
 528:     NewMI->clearKillInfo();
 529: 
 530:   // The caller will erase MI, but not DefMI.
 531:   DefMI->eraseFromParent();
 532:   return NewMI;
 533: }
 534: 
 535: // The analyzeBranch function is used to examine conditional instructions and
 536: // remove unnecessary instructions. This method is used by BranchFolder and
 537: // IfConverter machine function passes to improve the CFG.
 538: // - TrueBlock is set to the destination if condition evaluates true (it is the
 539: //   nullptr if the destination is the fall-through branch);
 540: // - FalseBlock is set to the destination if condition evaluates to false (it
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 541-560

```cpp
 541: //   is the nullptr if the branch is unconditional);
 542: // - condition is populated with machine operands needed to generate the branch
 543: //   to insert in insertBranch;
 544: // Returns: false if branch could successfully be analyzed.
 545: bool LanaiInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 546:                                    MachineBasicBlock *&TrueBlock,
 547:                                    MachineBasicBlock *&FalseBlock,
 548:                                    SmallVectorImpl<MachineOperand> &Condition,
 549:                                    bool AllowModify) const {
 550:   // Iterator to current instruction being considered.
 551:   MachineBasicBlock::iterator Instruction = MBB.end();
 552: 
 553:   // Start from the bottom of the block and work up, examining the
 554:   // terminator instructions.
 555:   while (Instruction != MBB.begin()) {
 556:     --Instruction;
 557: 
 558:     // Skip over debug instructions.
 559:     if (Instruction->isDebugInstr())
 560:       continue;
```

- EN: Function bodies or method definitions such as analyzeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: analyzeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 561-580

```cpp
 561: 
 562:     // Working from the bottom, when we see a non-terminator
 563:     // instruction, we're done.
 564:     if (!isUnpredicatedTerminator(*Instruction))
 565:       break;
 566: 
 567:     // A terminator that isn't a branch can't easily be handled
 568:     // by this analysis.
 569:     if (!Instruction->isBranch())
 570:       return true;
 571: 
 572:     // Handle unconditional branches.
 573:     if (Instruction->getOpcode() == Lanai::BT) {
 574:       if (!AllowModify) {
 575:         TrueBlock = Instruction->getOperand(0).getMBB();
 576:         continue;
 577:       }
 578: 
 579:       // If the block has any instructions after a branch, delete them.
 580:       MBB.erase(std::next(Instruction), MBB.end());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 581-600

```cpp
 581: 
 582:       Condition.clear();
 583:       FalseBlock = nullptr;
 584: 
 585:       // Delete the jump if it's equivalent to a fall-through.
 586:       if (MBB.isLayoutSuccessor(Instruction->getOperand(0).getMBB())) {
 587:         TrueBlock = nullptr;
 588:         Instruction->eraseFromParent();
 589:         Instruction = MBB.end();
 590:         continue;
 591:       }
 592: 
 593:       // TrueBlock is used to indicate the unconditional destination.
 594:       TrueBlock = Instruction->getOperand(0).getMBB();
 595:       continue;
 596:     }
 597: 
 598:     // Handle conditional branches
 599:     unsigned Opcode = Instruction->getOpcode();
 600:     if (Opcode != Lanai::BRCC)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 601-620

```cpp
 601:       return true; // Unknown opcode.
 602: 
 603:     // Multiple conditional branches are not handled here so only proceed if
 604:     // there are no conditions enqueued.
 605:     if (Condition.empty()) {
 606:       LPCC::CondCode BranchCond =
 607:           static_cast<LPCC::CondCode>(Instruction->getOperand(1).getImm());
 608: 
 609:       // TrueBlock is the target of the previously seen unconditional branch.
 610:       FalseBlock = TrueBlock;
 611:       TrueBlock = Instruction->getOperand(0).getMBB();
 612:       Condition.push_back(MachineOperand::CreateImm(BranchCond));
 613:       continue;
 614:     }
 615: 
 616:     // Multiple conditional branches are not handled.
 617:     return true;
 618:   }
 619: 
 620:   // Return false indicating branch successfully analyzed.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 621-640

```cpp
 621:   return false;
 622: }
 623: 
 624: // reverseBranchCondition - Reverses the branch condition of the specified
 625: // condition list, returning false on success and true if it cannot be
 626: // reversed.
 627: bool LanaiInstrInfo::reverseBranchCondition(
 628:     SmallVectorImpl<llvm::MachineOperand> &Condition) const {
 629:   assert((Condition.size() == 1) &&
 630:          "Lanai branch conditions should have one component.");
 631: 
 632:   LPCC::CondCode BranchCond =
 633:       static_cast<LPCC::CondCode>(Condition[0].getImm());
 634:   Condition[0].setImm(getOppositeCondition(BranchCond));
 635:   return false;
 636: }
 637: 
 638: // Insert the branch with condition specified in condition and given targets
 639: // (TrueBlock and FalseBlock). This function returns the number of machine
 640: // instructions inserted.
```

- EN: Function bodies or method definitions such as reverseBranchCondition contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: reverseBranchCondition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 641-660

```cpp
 641: unsigned LanaiInstrInfo::insertBranch(MachineBasicBlock &MBB,
 642:                                       MachineBasicBlock *TrueBlock,
 643:                                       MachineBasicBlock *FalseBlock,
 644:                                       ArrayRef<MachineOperand> Condition,
 645:                                       const DebugLoc &DL,
 646:                                       int *BytesAdded) const {
 647:   // Shouldn't be a fall through.
 648:   assert(TrueBlock && "insertBranch must not be told to insert a fallthrough");
 649:   assert(!BytesAdded && "code size not handled");
 650: 
 651:   // If condition is empty then an unconditional branch is being inserted.
 652:   if (Condition.empty()) {
 653:     assert(!FalseBlock && "Unconditional branch with multiple successors!");
 654:     BuildMI(&MBB, DL, get(Lanai::BT)).addMBB(TrueBlock);
 655:     return 1;
 656:   }
 657: 
 658:   // Else a conditional branch is inserted.
 659:   assert((Condition.size() == 1) &&
 660:          "Lanai branch conditions should have one component.");
```

- EN: Function bodies or method definitions such as insertBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: insertBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 661-680

```cpp
 661:   unsigned ConditionalCode = Condition[0].getImm();
 662:   BuildMI(&MBB, DL, get(Lanai::BRCC)).addMBB(TrueBlock).addImm(ConditionalCode);
 663: 
 664:   // If no false block, then false behavior is fall through and no branch needs
 665:   // to be inserted.
 666:   if (!FalseBlock)
 667:     return 1;
 668: 
 669:   BuildMI(&MBB, DL, get(Lanai::BT)).addMBB(FalseBlock);
 670:   return 2;
 671: }
 672: 
 673: unsigned LanaiInstrInfo::removeBranch(MachineBasicBlock &MBB,
 674:                                       int *BytesRemoved) const {
 675:   assert(!BytesRemoved && "code size not handled");
 676: 
 677:   MachineBasicBlock::iterator Instruction = MBB.end();
 678:   unsigned Count = 0;
 679: 
 680:   while (Instruction != MBB.begin()) {
```

- EN: Function bodies or method definitions such as removeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: removeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 681-700

```cpp
 681:     --Instruction;
 682:     if (Instruction->isDebugInstr())
 683:       continue;
 684:     if (Instruction->getOpcode() != Lanai::BT &&
 685:         Instruction->getOpcode() != Lanai::BRCC) {
 686:       break;
 687:     }
 688: 
 689:     // Remove the branch.
 690:     Instruction->eraseFromParent();
 691:     Instruction = MBB.end();
 692:     ++Count;
 693:   }
 694: 
 695:   return Count;
 696: }
 697: 
 698: Register LanaiInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
 699:                                              int &FrameIndex) const {
 700:   if (MI.getOpcode() == Lanai::LDW_RI)
```

- EN: Function bodies or method definitions such as isLoadFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isLoadFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 701-720

```cpp
 701:     if (MI.getOperand(1).isFI() && MI.getOperand(2).isImm() &&
 702:         MI.getOperand(2).getImm() == 0) {
 703:       FrameIndex = MI.getOperand(1).getIndex();
 704:       return MI.getOperand(0).getReg();
 705:     }
 706:   return 0;
 707: }
 708: 
 709: Register LanaiInstrInfo::isLoadFromStackSlotPostFE(const MachineInstr &MI,
 710:                                                    int &FrameIndex) const {
 711:   if (MI.getOpcode() == Lanai::LDW_RI) {
 712:     unsigned Reg;
 713:     if ((Reg = isLoadFromStackSlot(MI, FrameIndex)))
 714:       return Reg;
 715:     // Check for post-frame index elimination operations
 716:     SmallVector<const MachineMemOperand *, 1> Accesses;
 717:     if (hasLoadFromStackSlot(MI, Accesses)){
 718:       FrameIndex =
 719:           cast<FixedStackPseudoSourceValue>(Accesses.front()->getPseudoValue())
 720:               ->getFrameIndex();
```

- EN: Function bodies or method definitions such as isLoadFromStackSlotPostFE contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isLoadFromStackSlotPostFE 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 721-740

```cpp
 721:       return 1;
 722:     }
 723:   }
 724:   return 0;
 725: }
 726: 
 727: Register LanaiInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
 728:                                             int &FrameIndex) const {
 729:   if (MI.getOpcode() == Lanai::SW_RI)
 730:     if (MI.getOperand(0).isFI() && MI.getOperand(1).isImm() &&
 731:         MI.getOperand(1).getImm() == 0) {
 732:       FrameIndex = MI.getOperand(0).getIndex();
 733:       return MI.getOperand(2).getReg();
 734:     }
 735:   return 0;
 736: }
 737: 
 738: bool LanaiInstrInfo::getMemOperandWithOffsetWidth(
 739:     const MachineInstr &LdSt, const MachineOperand *&BaseOp, int64_t &Offset,
 740:     LocationSize &Width, const TargetRegisterInfo * /*TRI*/) const {
```

- EN: Function bodies or method definitions such as isStoreToStackSlot, getMemOperandWithOffsetWidth contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isStoreToStackSlot, getMemOperandWithOffsetWidth 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 741-760

```cpp
 741:   // Handle only loads/stores with base register followed by immediate offset
 742:   // and with add as ALU op.
 743:   if (LdSt.getNumOperands() != 4)
 744:     return false;
 745:   if (!LdSt.getOperand(1).isReg() || !LdSt.getOperand(2).isImm() ||
 746:       !(LdSt.getOperand(3).isImm() && LdSt.getOperand(3).getImm() == LPAC::ADD))
 747:     return false;
 748: 
 749:   switch (LdSt.getOpcode()) {
 750:   default:
 751:     return false;
 752:   case Lanai::LDW_RI:
 753:   case Lanai::LDW_RR:
 754:   case Lanai::SW_RR:
 755:   case Lanai::SW_RI:
 756:     Width = LocationSize::precise(4);
 757:     break;
 758:   case Lanai::LDHs_RI:
 759:   case Lanai::LDHz_RI:
 760:   case Lanai::STH_RI:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 761-780

```cpp
 761:     Width = LocationSize::precise(2);
 762:     break;
 763:   case Lanai::LDBs_RI:
 764:   case Lanai::LDBz_RI:
 765:   case Lanai::STB_RI:
 766:     Width = LocationSize::precise(1);
 767:     break;
 768:   }
 769: 
 770:   BaseOp = &LdSt.getOperand(1);
 771:   Offset = LdSt.getOperand(2).getImm();
 772: 
 773:   if (!BaseOp->isReg())
 774:     return false;
 775: 
 776:   return true;
 777: }
 778: 
 779: bool LanaiInstrInfo::getMemOperandsWithOffsetWidth(
 780:     const MachineInstr &LdSt, SmallVectorImpl<const MachineOperand *> &BaseOps,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 781-800

```cpp
 781:     int64_t &Offset, bool &OffsetIsScalable, LocationSize &Width,
 782:     const TargetRegisterInfo *TRI) const {
 783:   switch (LdSt.getOpcode()) {
 784:   default:
 785:     return false;
 786:   case Lanai::LDW_RI:
 787:   case Lanai::LDW_RR:
 788:   case Lanai::SW_RR:
 789:   case Lanai::SW_RI:
 790:   case Lanai::LDHs_RI:
 791:   case Lanai::LDHz_RI:
 792:   case Lanai::STH_RI:
 793:   case Lanai::LDBs_RI:
 794:   case Lanai::LDBz_RI:
 795:     const MachineOperand *BaseOp;
 796:     OffsetIsScalable = false;
 797:     if (!getMemOperandWithOffsetWidth(LdSt, BaseOp, Offset, Width, TRI))
 798:       return false;
 799:     BaseOps.push_back(BaseOp);
 800:     return true;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 801-802

```cpp
 801:   }
 802: }
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiInstrInfo.h`, `LanaiAluCode.h`, `LanaiCondCode.h`, `LanaiSubtarget.h`, `MCTargetDesc/LanaiBaseInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineInstrBuilder.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Generated or companion files / 生成或配套文件: `LanaiGenInstrInfo.inc`
- Local companions / 本地配套文件: `LanaiInstrInfo.h`, `LanaiInstrInfo.td`
