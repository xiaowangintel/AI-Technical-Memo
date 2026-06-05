# LoongArchExpandPseudoInsts.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchExpandPseudoInsts.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the LoongArch backend.
- **用途 (CN)**: 提供 LoongArch 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- LoongArchExpandPseudoInsts.cpp - Expand pseudo instructions -------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a pass that expands pseudo instructions into target
  10: // instructions.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArch.h"
  15: #include "LoongArchInstrInfo.h"
  16: #include "LoongArchMachineFunctionInfo.h"
  17: #include "MCTargetDesc/LoongArchBaseInfo.h"
  18: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  19: #include "llvm/CodeGen/LivePhysRegs.h"
  20: #include "llvm/CodeGen/MachineFunctionPass.h"
  21: #include "llvm/CodeGen/MachineInstrBuilder.h"
  22: #include "llvm/CodeGen/MachineOperand.h"
  23: #include "llvm/CodeGen/Register.h"
  24: #include "llvm/MC/MCContext.h"
  25: #include "llvm/Support/CodeGen.h"
  26: #include "llvm/Support/ErrorHandling.h"
  27: 
  28: using namespace llvm;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArch.h`, `LoongArchInstrInfo.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchBaseInfo.h`, `LoongArchMCTargetDesc.h`, `LivePhysRegs.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArch.h`, `LoongArchInstrInfo.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchBaseInfo.h`, `LoongArchMCTargetDesc.h`, `LivePhysRegs.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 29-56 / 第 29-56 行
```cpp
  29: 
  30: extern cl::opt<bool> LArchAnnotateTableJump;
  31: 
  32: #define LOONGARCH_PRERA_EXPAND_PSEUDO_NAME                                     \
  33:   "LoongArch Pre-RA pseudo instruction expansion pass"
  34: #define LOONGARCH_EXPAND_PSEUDO_NAME                                           \
  35:   "LoongArch pseudo instruction expansion pass"
  36: 
  37: namespace {
  38: 
  39: class LoongArchPreRAExpandPseudo : public MachineFunctionPass {
  40: public:
  41:   const LoongArchInstrInfo *TII;
  42:   static char ID;
  43: 
  44:   LoongArchPreRAExpandPseudo() : MachineFunctionPass(ID) {}
  45: 
  46:   bool runOnMachineFunction(MachineFunction &MF) override;
  47: 
  48:   void getAnalysisUsage(AnalysisUsage &AU) const override {
  49:     AU.setPreservesCFG();
  50:     MachineFunctionPass::getAnalysisUsage(AU);
  51:   }
  52:   StringRef getPassName() const override {
  53:     return LOONGARCH_PRERA_EXPAND_PSEUDO_NAME;
  54:   }
  55: 
  56: private:
```
- **EN**: This block declares or refines TableGen records such as `LoongArchPreRAExpandPseudo`. The range implements or declares functions including `LoongArchPreRAExpandPseudo`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 该代码块声明或细化了 `LoongArchPreRAExpandPseudo` 等 TableGen 记录。 这一段实现或声明了 `LoongArchPreRAExpandPseudo` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 57-84 / 第 57-84 行
```cpp
  57:   bool expandMBB(MachineBasicBlock &MBB);
  58:   bool expandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  59:                 MachineBasicBlock::iterator &NextMBBI);
  60:   bool expandPcaxxu12iInstPair(MachineBasicBlock &MBB,
  61:                                MachineBasicBlock::iterator MBBI,
  62:                                MachineBasicBlock::iterator &NextMBBI,
  63:                                unsigned OpcodeHi, unsigned OpcodeLo,
  64:                                unsigned FlagsHi, unsigned FlagsLo);
  65:   bool expandLargeAddressLoad(MachineBasicBlock &MBB,
  66:                               MachineBasicBlock::iterator MBBI,
  67:                               MachineBasicBlock::iterator &NextMBBI,
  68:                               unsigned LastOpcode, unsigned IdentifyingMO);
  69:   bool expandLargeAddressLoad(MachineBasicBlock &MBB,
  70:                               MachineBasicBlock::iterator MBBI,
  71:                               MachineBasicBlock::iterator &NextMBBI,
  72:                               unsigned LastOpcode, unsigned IdentifyingMO,
  73:                               const MachineOperand &Symbol, Register DestReg,
  74:                               bool EraseFromParent);
  75:   bool expandLoadAddressPcrel(MachineBasicBlock &MBB,
  76:                               MachineBasicBlock::iterator MBBI,
  77:                               MachineBasicBlock::iterator &NextMBBI,
  78:                               bool Large = false);
  79:   bool expandLoadAddressGot(MachineBasicBlock &MBB,
  80:                             MachineBasicBlock::iterator MBBI,
  81:                             MachineBasicBlock::iterator &NextMBBI,
  82:                             bool Large = false);
  83:   bool expandLoadAddressTLSLE(MachineBasicBlock &MBB,
  84:                               MachineBasicBlock::iterator MBBI,
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 85-112 / 第 85-112 行
```cpp
  85:                               MachineBasicBlock::iterator &NextMBBI);
  86:   bool expandLoadAddressTLSIE(MachineBasicBlock &MBB,
  87:                               MachineBasicBlock::iterator MBBI,
  88:                               MachineBasicBlock::iterator &NextMBBI,
  89:                               bool Large = false);
  90:   bool expandLoadAddressTLSLD(MachineBasicBlock &MBB,
  91:                               MachineBasicBlock::iterator MBBI,
  92:                               MachineBasicBlock::iterator &NextMBBI,
  93:                               bool Large = false);
  94:   bool expandLoadAddressTLSGD(MachineBasicBlock &MBB,
  95:                               MachineBasicBlock::iterator MBBI,
  96:                               MachineBasicBlock::iterator &NextMBBI,
  97:                               bool Large = false);
  98:   bool expandLoadAddressTLSDesc(MachineBasicBlock &MBB,
  99:                                 MachineBasicBlock::iterator MBBI,
 100:                                 MachineBasicBlock::iterator &NextMBBI,
 101:                                 bool Large = false);
 102:   bool expandFunctionCALL(MachineBasicBlock &MBB,
 103:                           MachineBasicBlock::iterator MBBI,
 104:                           MachineBasicBlock::iterator &NextMBBI,
 105:                           bool IsTailCall);
 106:   void annotateTableJump(MachineBasicBlock &MBB,
 107:                          MachineBasicBlock::iterator MBBI);
 108: };
 109: 
 110: char LoongArchPreRAExpandPseudo::ID = 0;
 111: 
 112: bool LoongArchPreRAExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::runOnMachineFunction`.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::runOnMachineFunction` 等函数。

### Lines 113-140 / 第 113-140 行
```cpp
 113:   TII =
 114:       static_cast<const LoongArchInstrInfo *>(MF.getSubtarget().getInstrInfo());
 115:   bool Modified = false;
 116:   for (auto &MBB : MF)
 117:     Modified |= expandMBB(MBB);
 118:   return Modified;
 119: }
 120: 
 121: bool LoongArchPreRAExpandPseudo::expandMBB(MachineBasicBlock &MBB) {
 122:   bool Modified = false;
 123: 
 124:   MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
 125:   while (MBBI != E) {
 126:     MachineBasicBlock::iterator NMBBI = std::next(MBBI);
 127:     Modified |= expandMI(MBB, MBBI, NMBBI);
 128:     MBBI = NMBBI;
 129:   }
 130: 
 131:   return Modified;
 132: }
 133: 
 134: bool LoongArchPreRAExpandPseudo::expandMI(
 135:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 136:     MachineBasicBlock::iterator &NextMBBI) {
 137:   switch (MBBI->getOpcode()) {
 138:   case LoongArch::PseudoLA_PCREL:
 139:     return expandLoadAddressPcrel(MBB, MBBI, NextMBBI);
 140:   case LoongArch::PseudoLA_PCREL_LARGE:
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandMBB`, `LoongArchPreRAExpandPseudo::expandMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandMBB`, `LoongArchPreRAExpandPseudo::expandMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141:     return expandLoadAddressPcrel(MBB, MBBI, NextMBBI, /*Large=*/true);
 142:   case LoongArch::PseudoLA_GOT:
 143:     return expandLoadAddressGot(MBB, MBBI, NextMBBI);
 144:   case LoongArch::PseudoLA_GOT_LARGE:
 145:     return expandLoadAddressGot(MBB, MBBI, NextMBBI, /*Large=*/true);
 146:   case LoongArch::PseudoLA_TLS_LE:
 147:     return expandLoadAddressTLSLE(MBB, MBBI, NextMBBI);
 148:   case LoongArch::PseudoLA_TLS_IE:
 149:     return expandLoadAddressTLSIE(MBB, MBBI, NextMBBI);
 150:   case LoongArch::PseudoLA_TLS_IE_LARGE:
 151:     return expandLoadAddressTLSIE(MBB, MBBI, NextMBBI, /*Large=*/true);
 152:   case LoongArch::PseudoLA_TLS_LD:
 153:     return expandLoadAddressTLSLD(MBB, MBBI, NextMBBI);
 154:   case LoongArch::PseudoLA_TLS_LD_LARGE:
 155:     return expandLoadAddressTLSLD(MBB, MBBI, NextMBBI, /*Large=*/true);
 156:   case LoongArch::PseudoLA_TLS_GD:
 157:     return expandLoadAddressTLSGD(MBB, MBBI, NextMBBI);
 158:   case LoongArch::PseudoLA_TLS_GD_LARGE:
 159:     return expandLoadAddressTLSGD(MBB, MBBI, NextMBBI, /*Large=*/true);
 160:   case LoongArch::PseudoLA_TLS_DESC:
 161:     return expandLoadAddressTLSDesc(MBB, MBBI, NextMBBI);
 162:   case LoongArch::PseudoLA_TLS_DESC_LARGE:
 163:     return expandLoadAddressTLSDesc(MBB, MBBI, NextMBBI, /*Large=*/true);
 164:   case LoongArch::PseudoCALL_SMALL:
 165:   case LoongArch::PseudoCALL_LARGE:
 166:     return expandFunctionCALL(MBB, MBBI, NextMBBI, /*IsTailCall=*/false);
 167:   case LoongArch::PseudoTAIL_SMALL:
 168:   case LoongArch::PseudoTAIL_LARGE:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 169-196 / 第 169-196 行
```cpp
 169:     return expandFunctionCALL(MBB, MBBI, NextMBBI, /*IsTailCall=*/true);
 170:   case LoongArch::PseudoBRIND:
 171:     // If the PseudoBRIND is used to table jump, then emit a label to annotate
 172:     // the `jr` instruction, and save the instructions.
 173:     if (LArchAnnotateTableJump)
 174:       annotateTableJump(MBB, MBBI);
 175:     break;
 176:   }
 177:   return false;
 178: }
 179: 
 180: bool LoongArchPreRAExpandPseudo::expandPcaxxu12iInstPair(
 181:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 182:     MachineBasicBlock::iterator &NextMBBI, unsigned OpcodeHi, unsigned OpcodeLo,
 183:     unsigned FlagsHi, unsigned FlagsLo) {
 184:   MachineFunction *MF = MBB.getParent();
 185:   MachineInstr &MI = *MBBI;
 186:   DebugLoc DL = MI.getDebugLoc();
 187: 
 188:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 189:   bool EnableRelax = STI.hasFeature(LoongArch::FeatureRelax);
 190: 
 191:   Register DestReg = MI.getOperand(0).getReg();
 192:   Register ScratchReg =
 193:       MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 194:   MachineOperand &Symbol = MI.getOperand(1);
 195: 
 196:   MachineInstr *FirstMI =
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandPcaxxu12iInstPair`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandPcaxxu12iInstPair` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197:       BuildMI(MBB, MBBI, DL, TII->get(OpcodeHi), ScratchReg)
 198:           .addDisp(Symbol, 0, LoongArchII::encodeFlags(FlagsHi, EnableRelax));
 199: 
 200:   MachineInstr *SecondMI = nullptr;
 201:   FlagsLo = LoongArchII::encodeFlags(FlagsLo, EnableRelax);
 202: 
 203:   if (OpcodeHi == LoongArch::PCALAU12I) {
 204:     SecondMI = BuildMI(MBB, MBBI, DL, TII->get(OpcodeLo), DestReg)
 205:                    .addReg(ScratchReg)
 206:                    .addDisp(Symbol, 0, FlagsLo);
 207:   } else {
 208:     MCSymbol *PCAddSymbol = MF->getContext().createNamedTempSymbol("pcadd_hi");
 209:     FirstMI->setPreInstrSymbol(*MF, PCAddSymbol);
 210:     SecondMI = BuildMI(MBB, MBBI, DL, TII->get(OpcodeLo), DestReg)
 211:                    .addReg(ScratchReg)
 212:                    .addSym(PCAddSymbol, FlagsLo);
 213:   }
 214: 
 215:   if (MI.hasOneMemOperand())
 216:     SecondMI->addMemOperand(*MF, *MI.memoperands_begin());
 217: 
 218:   MI.eraseFromParent();
 219:   return true;
 220: }
 221: 
 222: bool LoongArchPreRAExpandPseudo::expandLargeAddressLoad(
 223:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 224:     MachineBasicBlock::iterator &NextMBBI, unsigned LastOpcode,
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:     unsigned IdentifyingMO) {
 226:   MachineInstr &MI = *MBBI;
 227:   return expandLargeAddressLoad(MBB, MBBI, NextMBBI, LastOpcode, IdentifyingMO,
 228:                                 MI.getOperand(2), MI.getOperand(0).getReg(),
 229:                                 true);
 230: }
 231: 
 232: bool LoongArchPreRAExpandPseudo::expandLargeAddressLoad(
 233:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 234:     MachineBasicBlock::iterator &NextMBBI, unsigned LastOpcode,
 235:     unsigned IdentifyingMO, const MachineOperand &Symbol, Register DestReg,
 236:     bool EraseFromParent) {
 237:   // Code Sequence:
 238:   //
 239:   // Part1: pcalau12i  $scratch, %MO1(sym)
 240:   // Part0: addi.d     $dest, $zero, %MO0(sym)
 241:   // Part2: lu32i.d    $dest, %MO2(sym)
 242:   // Part3: lu52i.d    $dest, $dest, %MO3(sym)
 243:   // Fin:   LastOpcode $dest, $dest, $scratch
 244: 
 245:   unsigned MO0, MO1, MO2, MO3;
 246:   switch (IdentifyingMO) {
 247:   default:
 248:     llvm_unreachable("unsupported identifying MO");
 249:   case LoongArchII::MO_PCREL_LO:
 250:     MO0 = IdentifyingMO;
 251:     MO1 = LoongArchII::MO_PCREL_HI;
 252:     MO2 = LoongArchII::MO_PCREL64_LO;
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLargeAddressLoad`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLargeAddressLoad` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 253-280 / 第 253-280 行
```cpp
 253:     MO3 = LoongArchII::MO_PCREL64_HI;
 254:     break;
 255:   case LoongArchII::MO_GOT_PC_HI:
 256:   case LoongArchII::MO_LD_PC_HI:
 257:   case LoongArchII::MO_GD_PC_HI:
 258:     // These cases relocate just like the GOT case, except for Part1.
 259:     MO0 = LoongArchII::MO_GOT_PC_LO;
 260:     MO1 = IdentifyingMO;
 261:     MO2 = LoongArchII::MO_GOT_PC64_LO;
 262:     MO3 = LoongArchII::MO_GOT_PC64_HI;
 263:     break;
 264:   case LoongArchII::MO_IE_PC_LO:
 265:     MO0 = IdentifyingMO;
 266:     MO1 = LoongArchII::MO_IE_PC_HI;
 267:     MO2 = LoongArchII::MO_IE_PC64_LO;
 268:     MO3 = LoongArchII::MO_IE_PC64_HI;
 269:     break;
 270:   }
 271: 
 272:   MachineFunction *MF = MBB.getParent();
 273:   MachineInstr &MI = *MBBI;
 274:   DebugLoc DL = MI.getDebugLoc();
 275: 
 276:   assert(MF->getSubtarget<LoongArchSubtarget>().is64Bit() &&
 277:          "Large code model requires LA64");
 278: 
 279:   Register TmpPart1 =
 280:       MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-308 / 第 281-308 行
```cpp
 281:   Register TmpPart0 =
 282:       DestReg.isVirtual()
 283:           ? MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass)
 284:           : DestReg;
 285:   Register TmpParts02 =
 286:       DestReg.isVirtual()
 287:           ? MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass)
 288:           : DestReg;
 289:   Register TmpParts023 =
 290:       DestReg.isVirtual()
 291:           ? MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass)
 292:           : DestReg;
 293: 
 294:   auto Part1 = BuildMI(MBB, MBBI, DL, TII->get(LoongArch::PCALAU12I), TmpPart1);
 295:   auto Part0 = BuildMI(MBB, MBBI, DL, TII->get(LoongArch::ADDI_D), TmpPart0)
 296:                    .addReg(LoongArch::R0);
 297:   auto Part2 = BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU32I_D), TmpParts02)
 298:                    // "rj" is needed due to InstrInfo pattern requirement.
 299:                    .addReg(TmpPart0, RegState::Kill);
 300:   auto Part3 = BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU52I_D), TmpParts023)
 301:                    .addReg(TmpParts02, RegState::Kill);
 302:   BuildMI(MBB, MBBI, DL, TII->get(LastOpcode), DestReg)
 303:       .addReg(TmpParts023)
 304:       .addReg(TmpPart1, RegState::Kill);
 305: 
 306:   if (Symbol.getType() == MachineOperand::MO_ExternalSymbol) {
 307:     const char *SymName = Symbol.getSymbolName();
 308:     Part0.addExternalSymbol(SymName, MO0);
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 309-336 / 第 309-336 行
```cpp
 309:     Part1.addExternalSymbol(SymName, MO1);
 310:     Part2.addExternalSymbol(SymName, MO2);
 311:     Part3.addExternalSymbol(SymName, MO3);
 312:   } else {
 313:     Part0.addDisp(Symbol, 0, MO0);
 314:     Part1.addDisp(Symbol, 0, MO1);
 315:     Part2.addDisp(Symbol, 0, MO2);
 316:     Part3.addDisp(Symbol, 0, MO3);
 317:   }
 318: 
 319:   if (EraseFromParent)
 320:     MI.eraseFromParent();
 321: 
 322:   return true;
 323: }
 324: 
 325: bool LoongArchPreRAExpandPseudo::expandLoadAddressPcrel(
 326:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 327:     MachineBasicBlock::iterator &NextMBBI, bool Large) {
 328:   if (Large)
 329:     // Emit the 5-insn large address load sequence with the `%pc` family of
 330:     // relocs.
 331:     return expandLargeAddressLoad(MBB, MBBI, NextMBBI, LoongArch::ADD_D,
 332:                                   LoongArchII::MO_PCREL_LO);
 333: 
 334:   // Code Sequence:
 335:   //
 336:   // for la32r expands to:
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressPcrel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressPcrel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:   // .Lpcadd_hi:
 338:   //   pcaddu12i $rd, %pcadd_hi20(sym)
 339:   //   addi.w    $rd, $rd, %pcadd_lo12(.Lpcadd_hi)
 340:   //
 341:   // for la32s and la64 expands to:
 342:   //   pcalau12i $rd, %pc_hi20(sym)
 343:   //   addi.w/d  $rd, $rd, %pc_lo12(sym)
 344:   MachineFunction *MF = MBB.getParent();
 345:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 346:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 347:   unsigned OpcodeHi = Has32S ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
 348:   unsigned OpcodeLo = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 349:   unsigned FlagsHi =
 350:       Has32S ? LoongArchII::MO_PCREL_HI : LoongArchII::MO_PCADD_HI;
 351:   unsigned FlagsLo =
 352:       Has32S ? LoongArchII::MO_PCREL_LO : LoongArchII::MO_PCADD_LO;
 353:   return expandPcaxxu12iInstPair(MBB, MBBI, NextMBBI, OpcodeHi, OpcodeLo,
 354:                                  FlagsHi, FlagsLo);
 355: }
 356: 
 357: bool LoongArchPreRAExpandPseudo::expandLoadAddressGot(
 358:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 359:     MachineBasicBlock::iterator &NextMBBI, bool Large) {
 360:   if (Large)
 361:     // Emit the 5-insn large address load sequence with the `%got_pc` family
 362:     // of relocs, loading the result from GOT with `ldx.d` in the end.
 363:     return expandLargeAddressLoad(MBB, MBBI, NextMBBI, LoongArch::LDX_D,
 364:                                   LoongArchII::MO_GOT_PC_HI);
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressGot`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressGot` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365: 
 366:   // Code Sequence:
 367:   //
 368:   // for la32r expands to:
 369:   // .Lpcadd_hi:
 370:   //   pcaddu12i $rd, %got_pcadd_hi20(sym)
 371:   //   ld.w      $rd, $rd, %got_pcadd_lo12(.Lpcadd_hi)
 372:   //
 373:   // for la32s and la64 expands to:
 374:   //   pcalau12i $rd, %got_pc_hi20(sym)
 375:   //   ld.w/d    $rd, $rd, %got_pc_lo12(sym)
 376:   MachineFunction *MF = MBB.getParent();
 377:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 378:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 379:   unsigned OpcodeHi = Has32S ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
 380:   unsigned OpcodeLo = STI.is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
 381:   unsigned FlagsHi =
 382:       Has32S ? LoongArchII::MO_GOT_PC_HI : LoongArchII::MO_GOT_PCADD_HI;
 383:   unsigned FlagsLo =
 384:       Has32S ? LoongArchII::MO_GOT_PC_LO : LoongArchII::MO_GOT_PCADD_LO;
 385:   return expandPcaxxu12iInstPair(MBB, MBBI, NextMBBI, OpcodeHi, OpcodeLo,
 386:                                  FlagsHi, FlagsLo);
 387: }
 388: 
 389: bool LoongArchPreRAExpandPseudo::expandLoadAddressTLSLE(
 390:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 391:     MachineBasicBlock::iterator &NextMBBI) {
 392:   // Code Sequence:
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressTLSLE`.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressTLSLE` 等函数。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   // lu12i.w $rd, %le_hi20_r(sym)
 394:   // add.w/d $rd, $rd, $tp, %le_add_r(sym)
 395:   // addi.w/d $rd, $rd, %le_lo12_r(sym)
 396:   //
 397:   // Code Sequence while using the large code model:
 398:   // lu12i.w $rd, %le_hi20(sym)
 399:   // ori $rd, $rd, %le_lo12(sym)
 400:   // lu32i.d $rd, %le64_lo20(sym)
 401:   // lu52i.d $rd, $rd, %le64_hi12(sym)
 402:   MachineFunction *MF = MBB.getParent();
 403:   MachineInstr &MI = *MBBI;
 404:   DebugLoc DL = MI.getDebugLoc();
 405: 
 406:   bool Large = MF->getTarget().getCodeModel() == CodeModel::Large;
 407:   Register DestReg = MI.getOperand(0).getReg();
 408:   Register Parts01 =
 409:       MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 410:   Register Part1 =
 411:       MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 412:   MachineOperand &Symbol = MI.getOperand(1);
 413: 
 414:   if (!Large) {
 415:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU12I_W), Part1)
 416:         .addDisp(Symbol, 0, LoongArchII::MO_LE_HI_R);
 417: 
 418:     const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 419:     unsigned AddOp = STI.is64Bit() ? LoongArch::PseudoAddTPRel_D
 420:                                    : LoongArch::PseudoAddTPRel_W;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421:     BuildMI(MBB, MBBI, DL, TII->get(AddOp), Parts01)
 422:         .addReg(Part1, RegState::Kill)
 423:         .addReg(LoongArch::R2)
 424:         .addDisp(Symbol, 0, LoongArchII::MO_LE_ADD_R);
 425: 
 426:     unsigned AddiOp = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 427:     BuildMI(MBB, MBBI, DL, TII->get(AddiOp), DestReg)
 428:         .addReg(Parts01, RegState::Kill)
 429:         .addDisp(Symbol, 0, LoongArchII::MO_LE_LO_R);
 430:   } else {
 431:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU12I_W), Part1)
 432:         .addDisp(Symbol, 0, LoongArchII::MO_LE_HI);
 433: 
 434:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::ORI), Parts01)
 435:         .addReg(Part1, RegState::Kill)
 436:         .addDisp(Symbol, 0, LoongArchII::MO_LE_LO);
 437: 
 438:     Register Parts012 =
 439:         MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 440: 
 441:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU32I_D), Parts012)
 442:         // "rj" is needed due to InstrInfo pattern requirement.
 443:         .addReg(Parts01, RegState::Kill)
 444:         .addDisp(Symbol, 0, LoongArchII::MO_LE64_LO);
 445:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU52I_D), DestReg)
 446:         .addReg(Parts012, RegState::Kill)
 447:         .addDisp(Symbol, 0, LoongArchII::MO_LE64_HI);
 448:   }
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 449-476 / 第 449-476 行
```cpp
 449: 
 450:   MI.eraseFromParent();
 451:   return true;
 452: }
 453: 
 454: bool LoongArchPreRAExpandPseudo::expandLoadAddressTLSIE(
 455:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 456:     MachineBasicBlock::iterator &NextMBBI, bool Large) {
 457:   if (Large)
 458:     // Emit the 5-insn large address load sequence with the `%ie_pc` family
 459:     // of relocs, loading the result with `ldx.d` in the end.
 460:     return expandLargeAddressLoad(MBB, MBBI, NextMBBI, LoongArch::LDX_D,
 461:                                   LoongArchII::MO_IE_PC_LO);
 462: 
 463:   // Code Sequence:
 464:   //
 465:   // for la32r expands to:
 466:   // .Lpcadd_hi:
 467:   //   pcaddu12i $rd, %ie_pcadd_hi20(sym)
 468:   //   ld.w      $rd, $rd, %ie_pcadd_lo12(.Lpcadd_hi)
 469:   //
 470:   // for la32s and la64 expands to:
 471:   //   pcalau12i $rd, %ie_pc_hi20(sym)
 472:   //   ld.w/d    $rd, $rd, %ie_pc_lo12(sym)
 473:   MachineFunction *MF = MBB.getParent();
 474:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 475:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 476:   unsigned OpcodeHi = Has32S ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressTLSIE`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressTLSIE` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 477-504 / 第 477-504 行
```cpp
 477:   unsigned OpcodeLo = STI.is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
 478:   unsigned FlagsHi =
 479:       Has32S ? LoongArchII::MO_IE_PC_HI : LoongArchII::MO_IE_PCADD_HI;
 480:   unsigned FlagsLo =
 481:       Has32S ? LoongArchII::MO_IE_PC_LO : LoongArchII::MO_IE_PCADD_LO;
 482:   return expandPcaxxu12iInstPair(MBB, MBBI, NextMBBI, OpcodeHi, OpcodeLo,
 483:                                  FlagsHi, FlagsLo);
 484: }
 485: 
 486: bool LoongArchPreRAExpandPseudo::expandLoadAddressTLSLD(
 487:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 488:     MachineBasicBlock::iterator &NextMBBI, bool Large) {
 489:   if (Large)
 490:     // Emit the 5-insn large address load sequence with the `%got_pc` family
 491:     // of relocs, with the `pcalau12i` insn relocated with `%ld_pc_hi20`.
 492:     return expandLargeAddressLoad(MBB, MBBI, NextMBBI, LoongArch::ADD_D,
 493:                                   LoongArchII::MO_LD_PC_HI);
 494: 
 495:   // Code Sequence:
 496:   //
 497:   // for la32r expands to:
 498:   // .Lpcadd_hi:
 499:   //   pcaddu12i $rd, %ld_pcadd_hi20(sym)
 500:   //   addi.w    $rd, $rd, %ld_pcadd_lo12(.Lpcadd_hi)
 501:   //
 502:   // for la32s and la64 expands to:
 503:   //   pcalau12i $rd, %ld_pc_hi20(sym)
 504:   //   addi.w/d  $rd, $rd, %got_pc_lo12(sym)
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressTLSLD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressTLSLD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-532 / 第 505-532 行
```cpp
 505:   MachineFunction *MF = MBB.getParent();
 506:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 507:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 508:   unsigned OpcodeHi = Has32S ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
 509:   unsigned OpcodeLo = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 510:   unsigned FlagsHi =
 511:       Has32S ? LoongArchII::MO_LD_PC_HI : LoongArchII::MO_LD_PCADD_HI;
 512:   unsigned FlagsLo =
 513:       Has32S ? LoongArchII::MO_GOT_PC_LO : LoongArchII::MO_LD_PCADD_LO;
 514:   return expandPcaxxu12iInstPair(MBB, MBBI, NextMBBI, OpcodeHi, OpcodeLo,
 515:                                  FlagsHi, FlagsLo);
 516: }
 517: 
 518: bool LoongArchPreRAExpandPseudo::expandLoadAddressTLSGD(
 519:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 520:     MachineBasicBlock::iterator &NextMBBI, bool Large) {
 521:   if (Large)
 522:     // Emit the 5-insn large address load sequence with the `%got_pc` family
 523:     // of relocs, with the `pcalau12i` insn relocated with `%gd_pc_hi20`.
 524:     return expandLargeAddressLoad(MBB, MBBI, NextMBBI, LoongArch::ADD_D,
 525:                                   LoongArchII::MO_GD_PC_HI);
 526: 
 527:   // Code Sequence:
 528:   //
 529:   // for la32r expands to:
 530:   // .Lpcadd_hi:
 531:   //   pcaddu12i $rd, %gd_pcadd_hi20(sym)
 532:   //   addi.w    $rd, $rd, %gd_pcadd_lo12(.Lpcadd_hi)
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressTLSGD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressTLSGD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:   //
 534:   // for la32s and la64 expands to:
 535:   //   pcalau12i $rd, %gd_pc_hi20(sym)
 536:   //   addi.w/d  $rd, $rd, %got_pc_lo12(sym)
 537:   MachineFunction *MF = MBB.getParent();
 538:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 539:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 540:   unsigned OpcodeHi = Has32S ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
 541:   unsigned OpcodeLo = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 542:   unsigned FlagsHi =
 543:       Has32S ? LoongArchII::MO_GD_PC_HI : LoongArchII::MO_GD_PCADD_HI;
 544:   unsigned FlagsLo =
 545:       Has32S ? LoongArchII::MO_GOT_PC_LO : LoongArchII::MO_GD_PCADD_LO;
 546:   return expandPcaxxu12iInstPair(MBB, MBBI, NextMBBI, OpcodeHi, OpcodeLo,
 547:                                  FlagsHi, FlagsLo);
 548: }
 549: 
 550: bool LoongArchPreRAExpandPseudo::expandLoadAddressTLSDesc(
 551:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 552:     MachineBasicBlock::iterator &NextMBBI, bool Large) {
 553:   MachineFunction *MF = MBB.getParent();
 554:   MachineInstr &MI = *MBBI;
 555:   DebugLoc DL = MI.getDebugLoc();
 556: 
 557:   const auto &STI = MF->getSubtarget<LoongArchSubtarget>();
 558:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 559:   bool EnableRelax = STI.hasFeature(LoongArch::FeatureRelax);
 560:   unsigned PCA = Has32S ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::expandLoadAddressTLSDesc`.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::expandLoadAddressTLSDesc` 等函数。

### Lines 561-588 / 第 561-588 行
```cpp
 561:   unsigned ADD = STI.is64Bit() ? LoongArch::ADD_D : LoongArch::ADD_W;
 562:   unsigned ADDI = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 563:   unsigned LD = STI.is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
 564:   unsigned MO =
 565:       Has32S ? LoongArchII::MO_DESC_PC_HI : LoongArchII::MO_DESC_PCADD_HI;
 566: 
 567:   Register DestReg = MI.getOperand(0).getReg();
 568:   Register Tmp1Reg =
 569:       MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 570:   MachineOperand &Symbol = MI.getOperand(Large ? 2 : 1);
 571: 
 572:   MachineInstr *PCAMI =
 573:       BuildMI(MBB, MBBI, DL, TII->get(PCA), Tmp1Reg)
 574:           .addDisp(Symbol, 0,
 575:                    LoongArchII::encodeFlags(MO, EnableRelax && !Large));
 576: 
 577:   if (Large) {
 578:     // Code Sequence:
 579:     //
 580:     // pcalau12i  $a0, %desc_pc_hi20(sym)
 581:     // addi.d     $a1, $zero, %desc_pc_lo12(sym)
 582:     // lu32i.d    $a1, %desc64_pc_lo20(sym)
 583:     // lu52i.d    $a1, $a1, %desc64_pc_hi12(sym)
 584:     // add.d      $a0, $a0, $a1
 585:     // ld.d       $ra, $a0, %desc_ld(sym)
 586:     // jirl       $ra, $ra, %desc_call(sym)
 587:     // add.d      $dst, $a0, $tp
 588:     assert(MBB.getParent()->getSubtarget<LoongArchSubtarget>().is64Bit() &&
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 589-616 / 第 589-616 行
```cpp
 589:            "Large code model requires LA64");
 590:     Register Tmp2Reg =
 591:         MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 592:     Register Tmp3Reg =
 593:         MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 594:     Register Tmp4Reg =
 595:         MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass);
 596:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::ADDI_D), Tmp2Reg)
 597:         .addReg(LoongArch::R0)
 598:         .addDisp(Symbol, 0, LoongArchII::MO_DESC_PC_LO);
 599:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU32I_D), Tmp3Reg)
 600:         .addReg(Tmp2Reg, RegState::Kill)
 601:         .addDisp(Symbol, 0, LoongArchII::MO_DESC64_PC_LO);
 602:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::LU52I_D), Tmp4Reg)
 603:         .addReg(Tmp3Reg)
 604:         .addDisp(Symbol, 0, LoongArchII::MO_DESC64_PC_HI);
 605:     BuildMI(MBB, MBBI, DL, TII->get(LoongArch::ADD_D), LoongArch::R4)
 606:         .addReg(Tmp1Reg)
 607:         .addReg(Tmp4Reg);
 608:   } else {
 609:     // Code Sequence:
 610:     //
 611:     // for la32r expands to:
 612:     // .Lpcadd_hi:
 613:     //   pcaddu12i $a0, %desc_pcadd_hi20(sym)
 614:     //   addi.w    $a0, $a0, %desc_pcadd_lo12(.Lpcadd_hi)
 615:     //   ld.w      $ra, $a0, %desc_ld(sym)
 616:     //   jirl      $ra, $ra, %desc_call(sym)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 617-644 / 第 617-644 行
```cpp
 617:     //   add.w     $dst, $a0, $tp
 618:     //
 619:     // for la32s and la64 expands to:
 620:     //   pcalau12i $a0, %desc_pc_hi20(sym)
 621:     //   addi.w/d  $a0, $a0, %desc_pc_lo12(sym)
 622:     //   ld.w/d    $ra, $a0, %desc_ld(sym)
 623:     //   jirl      $ra, $ra, %desc_call(sym)
 624:     //   add.w/d   $dst, $a0, $tp
 625:     if (Has32S) {
 626:       BuildMI(MBB, MBBI, DL, TII->get(ADDI), LoongArch::R4)
 627:           .addReg(Tmp1Reg)
 628:           .addDisp(Symbol, 0,
 629:                    LoongArchII::encodeFlags(LoongArchII::MO_DESC_PC_LO,
 630:                                             EnableRelax));
 631:     } else {
 632:       MCSymbol *PCASymbol = MF->getContext().createNamedTempSymbol("pcadd_hi");
 633:       PCAMI->setPreInstrSymbol(*MF, PCASymbol);
 634:       BuildMI(MBB, MBBI, DL, TII->get(ADDI), LoongArch::R4)
 635:           .addReg(Tmp1Reg)
 636:           .addSym(PCASymbol, LoongArchII::encodeFlags(
 637:                                  LoongArchII::MO_DESC_PCADD_LO, EnableRelax));
 638:     }
 639:   }
 640: 
 641:   BuildMI(MBB, MBBI, DL, TII->get(LD), LoongArch::R1)
 642:       .addReg(LoongArch::R4)
 643:       .addDisp(Symbol, 0,
 644:                LoongArchII::encodeFlags(LoongArchII::MO_DESC_LD,
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```cpp
 645:                                         EnableRelax && !Large));
 646:   BuildMI(MBB, MBBI, DL, TII->get(LoongArch::PseudoDESC_CALL), LoongArch::R1)
 647:       .addReg(LoongArch::R1)
 648:       .addDisp(Symbol, 0,
 649:                LoongArchII::encodeFlags(LoongArchII::MO_DESC_CALL,
 650:                                         EnableRelax && !Large));
 651:   BuildMI(MBB, MBBI, DL, TII->get(ADD), DestReg)
 652:       .addReg(LoongArch::R4)
 653:       .addReg(LoongArch::R2);
 654: 
 655:   MI.eraseFromParent();
 656:   return true;
 657: }
 658: 
 659: bool LoongArchPreRAExpandPseudo::expandFunctionCALL(
 660:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 661:     MachineBasicBlock::iterator &NextMBBI, bool IsTailCall) {
 662:   MachineFunction *MF = MBB.getParent();
 663:   MachineInstr &MI = *MBBI;
 664:   DebugLoc DL = MI.getDebugLoc();
 665:   const MachineOperand &Func = MI.getOperand(0);
 666:   MachineInstrBuilder CALL;
 667:   unsigned Opcode;
 668: 
 669:   switch (MF->getTarget().getCodeModel()) {
 670:   default:
 671:     report_fatal_error("Unexpected code model");
 672:     break;
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `LoongArchPreRAExpandPseudo::expandFunctionCALL`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `LoongArchPreRAExpandPseudo::expandFunctionCALL` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 673-700 / 第 673-700 行
```cpp
 673:   case CodeModel::Small: {
 674:     // CALL:
 675:     // bl func
 676:     // TAIL:
 677:     // b func
 678:     Opcode = IsTailCall ? LoongArch::PseudoB_TAIL : LoongArch::BL;
 679:     CALL = BuildMI(MBB, MBBI, DL, TII->get(Opcode)).add(Func);
 680:     break;
 681:   }
 682:   case CodeModel::Large: {
 683:     // Emit the 5-insn large address load sequence, either directly or
 684:     // indirectly in case of going through the GOT, then JIRL_TAIL or
 685:     // JIRL_CALL to $addr.
 686:     Opcode =
 687:         IsTailCall ? LoongArch::PseudoJIRL_TAIL : LoongArch::PseudoJIRL_CALL;
 688:     Register AddrReg =
 689:         IsTailCall
 690:             ? MF->getRegInfo().createVirtualRegister(&LoongArch::GPRRegClass)
 691:             : LoongArch::R1;
 692: 
 693:     bool UseGOT = Func.getTargetFlags() == LoongArchII::MO_CALL_PLT;
 694:     unsigned MO = UseGOT ? LoongArchII::MO_GOT_PC_HI : LoongArchII::MO_PCREL_LO;
 695:     unsigned LAOpcode = UseGOT ? LoongArch::LDX_D : LoongArch::ADD_D;
 696:     expandLargeAddressLoad(MBB, MBBI, NextMBBI, LAOpcode, MO, Func, AddrReg,
 697:                            false);
 698:     CALL = BuildMI(MBB, MBBI, DL, TII->get(Opcode)).addReg(AddrReg).addImm(0);
 699:     break;
 700:   }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 701-728 / 第 701-728 行
```cpp
 701:   }
 702: 
 703:   // Transfer implicit operands.
 704:   CALL.copyImplicitOps(MI);
 705: 
 706:   // Transfer MI flags.
 707:   CALL.setMIFlags(MI.getFlags());
 708: 
 709:   MI.eraseFromParent();
 710:   return true;
 711: }
 712: 
 713: void LoongArchPreRAExpandPseudo::annotateTableJump(
 714:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI) {
 715:   MachineFunction *MF = MBB.getParent();
 716:   MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
 717: 
 718:   bool IsFound = false;
 719: 
 720:   std::function<void(MachineInstr *, int)> FindJTIMI = [&](MachineInstr *MInst,
 721:                                                            int FindDepth) {
 722:     if (FindDepth < 0)
 723:       return;
 724:     for (auto &MO : MInst->all_uses()) {
 725:       if (IsFound)
 726:         return;
 727:       Register Reg = MO.getReg();
 728:       if (!Reg.isVirtual())
```
- **EN**: The range implements or declares functions including `LoongArchPreRAExpandPseudo::annotateTableJump`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchPreRAExpandPseudo::annotateTableJump` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 729-756 / 第 729-756 行
```cpp
 729:         continue;
 730:       MachineInstr *DefMI = MRI.getVRegDef(Reg);
 731:       if (!DefMI)
 732:         continue;
 733:       for (unsigned Idx = 0; Idx < DefMI->getNumOperands(); ++Idx) {
 734:         MachineOperand &MO = DefMI->getOperand(Idx);
 735:         if (MO.isJTI()) {
 736:           MBBI->setPreInstrSymbol(
 737:               *MF, MF->getContext().createNamedTempSymbol("jrtb_"));
 738:           MF->getInfo<LoongArchMachineFunctionInfo>()->setJumpInfo(
 739:               &*MBBI, MO.getIndex());
 740:           IsFound = true;
 741:           return;
 742:         }
 743:       }
 744:       FindJTIMI(DefMI, --FindDepth);
 745:     }
 746:   };
 747: 
 748:   // FindDepth = 4, probably sufficient.
 749:   FindJTIMI(&*MBBI, /*FindDepth=*/4);
 750: }
 751: 
 752: class LoongArchExpandPseudo : public MachineFunctionPass {
 753: public:
 754:   const LoongArchInstrInfo *TII;
 755:   static char ID;
 756: 
```
- **EN**: This block declares or refines TableGen records such as `LoongArchExpandPseudo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `LoongArchExpandPseudo` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 757-784 / 第 757-784 行
```cpp
 757:   LoongArchExpandPseudo() : MachineFunctionPass(ID) {}
 758: 
 759:   bool runOnMachineFunction(MachineFunction &MF) override;
 760: 
 761:   StringRef getPassName() const override {
 762:     return LOONGARCH_EXPAND_PSEUDO_NAME;
 763:   }
 764: 
 765: private:
 766:   bool expandMBB(MachineBasicBlock &MBB);
 767:   bool expandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 768:                 MachineBasicBlock::iterator &NextMBBI);
 769:   bool expandCopyCFR(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 770:                      MachineBasicBlock::iterator &NextMBBI);
 771:   bool expandFunctionCALL(MachineBasicBlock &MBB,
 772:                           MachineBasicBlock::iterator MBBI,
 773:                           MachineBasicBlock::iterator &NextMBBI,
 774:                           bool IsTailCall);
 775: };
 776: 
 777: char LoongArchExpandPseudo::ID = 0;
 778: 
 779: bool LoongArchExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
 780:   TII =
 781:       static_cast<const LoongArchInstrInfo *>(MF.getSubtarget().getInstrInfo());
 782: 
 783:   bool Modified = false;
 784:   for (auto &MBB : MF)
```
- **EN**: The range implements or declares functions including `LoongArchExpandPseudo`, `LoongArchExpandPseudo::runOnMachineFunction`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchExpandPseudo`, `LoongArchExpandPseudo::runOnMachineFunction` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 785-812 / 第 785-812 行
```cpp
 785:     Modified |= expandMBB(MBB);
 786: 
 787:   return Modified;
 788: }
 789: 
 790: bool LoongArchExpandPseudo::expandMBB(MachineBasicBlock &MBB) {
 791:   bool Modified = false;
 792: 
 793:   MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
 794:   while (MBBI != E) {
 795:     MachineBasicBlock::iterator NMBBI = std::next(MBBI);
 796:     Modified |= expandMI(MBB, MBBI, NMBBI);
 797:     MBBI = NMBBI;
 798:   }
 799: 
 800:   return Modified;
 801: }
 802: 
 803: bool LoongArchExpandPseudo::expandMI(MachineBasicBlock &MBB,
 804:                                      MachineBasicBlock::iterator MBBI,
 805:                                      MachineBasicBlock::iterator &NextMBBI) {
 806:   switch (MBBI->getOpcode()) {
 807:   case LoongArch::PseudoCopyCFR:
 808:     return expandCopyCFR(MBB, MBBI, NextMBBI);
 809:   case LoongArch::PseudoCALL_MEDIUM:
 810:     return expandFunctionCALL(MBB, MBBI, NextMBBI, /*IsTailCall=*/false);
 811:   case LoongArch::PseudoTAIL_MEDIUM:
 812:     return expandFunctionCALL(MBB, MBBI, NextMBBI, /*IsTailCall=*/true);
```
- **EN**: The range implements or declares functions including `LoongArchExpandPseudo::expandMBB`, `LoongArchExpandPseudo::expandMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchExpandPseudo::expandMBB`, `LoongArchExpandPseudo::expandMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 813-840 / 第 813-840 行
```cpp
 813:   }
 814: 
 815:   return false;
 816: }
 817: 
 818: bool LoongArchExpandPseudo::expandCopyCFR(
 819:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 820:     MachineBasicBlock::iterator &NextMBBI) {
 821:   MachineFunction *MF = MBB.getParent();
 822:   MachineInstr &MI = *MBBI;
 823:   DebugLoc DL = MI.getDebugLoc();
 824: 
 825:   // Expand:
 826:   // MBB:
 827:   //    fcmp.caf.s  $dst, $fa0, $fa0 # set $dst 0(false)
 828:   //    bceqz $src, SinkBB
 829:   // FalseBB:
 830:   //    fcmp.cueq.s $dst, $fa0, $fa0 # set $dst 1(true)
 831:   // SinkBB:
 832:   //    fallthrough
 833: 
 834:   const BasicBlock *LLVM_BB = MBB.getBasicBlock();
 835:   auto *FalseBB = MF->CreateMachineBasicBlock(LLVM_BB);
 836:   auto *SinkBB = MF->CreateMachineBasicBlock(LLVM_BB);
 837: 
 838:   MF->insert(++MBB.getIterator(), FalseBB);
 839:   MF->insert(++FalseBB->getIterator(), SinkBB);
 840: 
```
- **EN**: The range implements or declares functions including `LoongArchExpandPseudo::expandCopyCFR`.
- **CN**: 这一段实现或声明了 `LoongArchExpandPseudo::expandCopyCFR` 等函数。

### Lines 841-868 / 第 841-868 行
```cpp
 841:   Register DestReg = MI.getOperand(0).getReg();
 842:   Register SrcReg = MI.getOperand(1).getReg();
 843:   // DestReg = 0
 844:   BuildMI(MBB, MBBI, DL, TII->get(LoongArch::SET_CFR_FALSE), DestReg);
 845:   // Insert branch instruction.
 846:   BuildMI(MBB, MBBI, DL, TII->get(LoongArch::BCEQZ))
 847:       .addReg(SrcReg)
 848:       .addMBB(SinkBB);
 849:   // DestReg = 1
 850:   BuildMI(FalseBB, DL, TII->get(LoongArch::SET_CFR_TRUE), DestReg);
 851: 
 852:   FalseBB->addSuccessor(SinkBB);
 853: 
 854:   SinkBB->splice(SinkBB->end(), &MBB, MI, MBB.end());
 855:   SinkBB->transferSuccessors(&MBB);
 856: 
 857:   MBB.addSuccessor(FalseBB);
 858:   MBB.addSuccessor(SinkBB);
 859: 
 860:   NextMBBI = MBB.end();
 861:   MI.eraseFromParent();
 862: 
 863:   // Make sure live-ins are correctly attached to this new basic block.
 864:   LivePhysRegs LiveRegs;
 865:   computeAndAddLiveIns(LiveRegs, *FalseBB);
 866:   computeAndAddLiveIns(LiveRegs, *SinkBB);
 867: 
 868:   return true;
```
- **EN**: The range implements or declares functions including `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。

### Lines 869-896 / 第 869-896 行
```cpp
 869: }
 870: 
 871: bool LoongArchExpandPseudo::expandFunctionCALL(
 872:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 873:     MachineBasicBlock::iterator &NextMBBI, bool IsTailCall) {
 874:   MachineFunction *MF = MBB.getParent();
 875:   MachineInstr &MI = *MBBI;
 876:   DebugLoc DL = MI.getDebugLoc();
 877:   const MachineOperand &Func = MI.getOperand(0);
 878:   MachineInstrBuilder CALL;
 879:   unsigned Opcode;
 880: 
 881:   switch (MF->getTarget().getCodeModel()) {
 882:   default:
 883:     report_fatal_error("Unexpected code model");
 884:     break;
 885:   case CodeModel::Medium: {
 886:     // for la32 expands to:
 887:     // CALL:
 888:     //   pcaddu12i $ra, %call30(func)
 889:     //   jirl      $ra, $ra, 0
 890:     // TAIL:
 891:     //   pcaddu12i $t8, %call30(func)
 892:     //   jirl      $r0, $t8, 0
 893:     //
 894:     // for la64 expands to:
 895:     // CALL:
 896:     //   pcaddu18i $ra, %call36(func)
```
- **EN**: The range implements or declares functions including `LoongArchExpandPseudo::expandFunctionCALL`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `LoongArchExpandPseudo::expandFunctionCALL` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 897-924 / 第 897-924 行
```cpp
 897:     //   jirl      $ra, $ra, 0
 898:     // TAIL:
 899:     //   pcaddu18i $t8, %call36(func)
 900:     //   jirl      $r0, $t8, 0
 901:     Opcode =
 902:         IsTailCall ? LoongArch::PseudoJIRL_TAIL : LoongArch::PseudoJIRL_CALL;
 903:     Register ScratchReg = IsTailCall ? LoongArch::R20 : LoongArch::R1;
 904:     bool Is64Bit = MF->getSubtarget<LoongArchSubtarget>().is64Bit();
 905:     unsigned PC = Is64Bit ? LoongArch::PCADDU18I : LoongArch::PCADDU12I;
 906:     unsigned MO = Is64Bit ? LoongArchII::MO_CALL36 : LoongArchII::MO_CALL30;
 907:     MachineInstrBuilder MIB = BuildMI(MBB, MBBI, DL, TII->get(PC), ScratchReg);
 908: 
 909:     CALL =
 910:         BuildMI(MBB, MBBI, DL, TII->get(Opcode)).addReg(ScratchReg).addImm(0);
 911: 
 912:     if (Func.isSymbol())
 913:       MIB.addExternalSymbol(Func.getSymbolName(), MO);
 914:     else
 915:       MIB.addDisp(Func, 0, MO);
 916:     break;
 917:   }
 918:   }
 919: 
 920:   // Transfer implicit operands.
 921:   CALL.copyImplicitOps(MI);
 922: 
 923:   // Transfer MI flags.
 924:   CALL.setMIFlags(MI.getFlags());
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 925-947 / 第 925-947 行
```cpp
 925: 
 926:   MI.eraseFromParent();
 927:   return true;
 928: }
 929: 
 930: } // end namespace
 931: 
 932: INITIALIZE_PASS(LoongArchPreRAExpandPseudo, "loongarch-prera-expand-pseudo",
 933:                 LOONGARCH_PRERA_EXPAND_PSEUDO_NAME, false, false)
 934: 
 935: INITIALIZE_PASS(LoongArchExpandPseudo, "loongarch-expand-pseudo",
 936:                 LOONGARCH_EXPAND_PSEUDO_NAME, false, false)
 937: 
 938: namespace llvm {
 939: 
 940: FunctionPass *createLoongArchPreRAExpandPseudoPass() {
 941:   return new LoongArchPreRAExpandPseudo();
 942: }
 943: FunctionPass *createLoongArchExpandPseudoPass() {
 944:   return new LoongArchExpandPseudo();
 945: }
 946: 
 947: } // end namespace llvm
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArch.h`
- `LoongArchInstrInfo.h`
- `LoongArchMachineFunctionInfo.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `llvm/CodeGen/LivePhysRegs.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineOperand.h`
- `llvm/CodeGen/Register.h`
- `llvm/MC/MCContext.h`
- `llvm/Support/CodeGen.h`
- `llvm/Support/ErrorHandling.h`
