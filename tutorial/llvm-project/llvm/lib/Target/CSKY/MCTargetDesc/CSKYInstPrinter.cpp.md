# CSKYInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYInstPrinter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYInstPrinter.cpp - Convert CSKY MCInst to asm syntax ---------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This class prints an CSKY MCInst to a .s file.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: #include "CSKYInstPrinter.h"
  13: #include "MCTargetDesc/CSKYBaseInfo.h"
  14: #include "MCTargetDesc/CSKYMCAsmInfo.h"
  15: #include "llvm/ADT/STLExtras.h"
  16: #include "llvm/ADT/StringExtras.h"
  17: #include "llvm/MC/MCAsmInfo.h"
  18: #include "llvm/MC/MCExpr.h"
  19: #include "llvm/MC/MCInst.h"
  20: #include "llvm/MC/MCInstrInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. This chunk introduces interfaces or data structures such as prints, which organize the target-specific behavior exposed by the file.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 这一段引入了 prints 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCRegisterInfo.h"
  22: #include "llvm/MC/MCSection.h"
  23: #include "llvm/MC/MCSubtargetInfo.h"
  24: #include "llvm/MC/MCSymbol.h"
  25: #include "llvm/Support/CommandLine.h"
  26: #include "llvm/Support/Debug.h"
  27: #include "llvm/Support/ErrorHandling.h"
  28: #include "llvm/Support/FormattedStream.h"
  29: 
  30: using namespace llvm;
  31: 
  32: #define DEBUG_TYPE "csky-asm-printer"
  33: 
  34: // Include the auto-generated portion of the assembly writer.
  35: #define PRINT_ALIAS_INSTR
  36: #include "CSKYGenAsmWriter.inc"
  37: 
  38: static cl::opt<bool>
  39:     NoAliases("csky-no-aliases",
  40:               cl::desc("Disable the emission of assembler pseudo instructions"),
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41:               cl::init(false), cl::Hidden);
  42: 
  43: static cl::opt<bool>
  44:     ArchRegNames("csky-arch-reg-names",
  45:                  cl::desc("Print architectural register names rather than the "
  46:                           "ABI names (such as r14 instead of sp)"),
  47:                  cl::init(false), cl::Hidden);
  48: 
  49: // The command-line flags above are used by llvm-mc and llc. They can be used by
  50: // `llvm-objdump`, but we override their values here to handle options passed to
  51: // `llvm-objdump` with `-M` (which matches GNU objdump). There did not seem to
  52: // be an easier way to allow these options in all these tools, without doing it
  53: // this way.
  54: bool CSKYInstPrinter::applyTargetSpecificCLOption(StringRef Opt) {
  55:   if (Opt == "no-aliases") {
  56:     NoAliases = true;
  57:     return true;
  58:   }
  59:   if (Opt == "numeric") {
  60:     ArchRegNames = true;
```

- EN: Function bodies or method definitions such as applyTargetSpecificCLOption contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: applyTargetSpecificCLOption 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-80

```cpp
  61:     return true;
  62:   }
  63:   if (Opt == "debug") {
  64:     DebugFlag = true;
  65:     return true;
  66:   }
  67:   if (Opt == "abi-names") {
  68:     ABIRegNames = true;
  69:     return true;
  70:   }
  71: 
  72:   return false;
  73: }
  74: 
  75: void CSKYInstPrinter::printInst(const MCInst *MI, uint64_t Address,
  76:                                 StringRef Annot, const MCSubtargetInfo &STI,
  77:                                 raw_ostream &O) {
  78:   const MCInst *NewMI = MI;
  79: 
  80:   if (NoAliases || !printAliasInstr(NewMI, Address, STI, O))
```

- EN: Function bodies or method definitions such as printInst contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printInst 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:     printInstruction(NewMI, Address, STI, O);
  82:   printAnnotation(O, Annot);
  83: }
  84: 
  85: void CSKYInstPrinter::printRegName(raw_ostream &O, MCRegister Reg) {
  86:   if (PrintBranchImmAsAddress)
  87:     O << getRegisterName(Reg, ABIRegNames ? CSKY::ABIRegAltName
  88:                                           : CSKY::NoRegAltName);
  89:   else
  90:     O << getRegisterName(Reg);
  91: }
  92: 
  93: void CSKYInstPrinter::printFPRRegName(raw_ostream &O, unsigned RegNo) const {
  94:   if (PrintBranchImmAsAddress)
  95:     O << getRegisterName(RegNo, CSKY::NoRegAltName);
  96:   else
  97:     O << getRegisterName(RegNo);
  98: }
  99: 
 100: void CSKYInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
```

- EN: Function bodies or method definitions such as printRegName, printFPRRegName contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printRegName, printFPRRegName 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:                                    const MCSubtargetInfo &STI, raw_ostream &O) {
 102:   const MCOperand &MO = MI->getOperand(OpNo);
 103: 
 104:   if (MO.isReg()) {
 105:     unsigned Reg = MO.getReg();
 106:     bool useABIName = false;
 107:     if (PrintBranchImmAsAddress)
 108:       useABIName = ABIRegNames;
 109:     else
 110:       useABIName = !ArchRegNames;
 111: 
 112:     if (Reg == CSKY::C)
 113:       O << "";
 114:     else if (STI.hasFeature(CSKY::FeatureJAVA)) {
 115:       if (Reg == CSKY::R23)
 116:         O << (useABIName ? "fp" : "r23");
 117:       else if (Reg == CSKY::R24)
 118:         O << (useABIName ? "top" : "r24");
 119:       else if (Reg == CSKY::R25)
 120:         O << (useABIName ? "bsp" : "r25");
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:       else
 122:         printRegName(O, Reg);
 123:     } else
 124:       printRegName(O, Reg);
 125: 
 126:     return;
 127:   }
 128: 
 129:   if (MO.isImm()) {
 130:     uint64_t TSFlags = MII.get(MI->getOpcode()).TSFlags;
 131: 
 132:     if (((TSFlags & CSKYII::AddrModeMask) != CSKYII::AddrModeNone) &&
 133:         PrintBranchImmAsAddress)
 134:       O << formatHex(MO.getImm());
 135:     else
 136:       O << MO.getImm();
 137:     return;
 138:   }
 139: 
 140:   assert(MO.isExpr() && "Unknown operand kind in printOperand");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:   MAI.printExpr(O, *MO.getExpr());
 142: }
 143: 
 144: void CSKYInstPrinter::printDataSymbol(const MCInst *MI, unsigned OpNo,
 145:                                       const MCSubtargetInfo &STI,
 146:                                       raw_ostream &O) {
 147:   const MCOperand &MO = MI->getOperand(OpNo);
 148: 
 149:   O << "[";
 150:   if (MO.isImm())
 151:     O << MO.getImm();
 152:   else
 153:     MAI.printExpr(O, *MO.getExpr());
 154:   O << "]";
 155: }
 156: 
 157: void CSKYInstPrinter::printConstpool(const MCInst *MI, uint64_t Address,
 158:                                      unsigned OpNo, const MCSubtargetInfo &STI,
 159:                                      raw_ostream &O) {
 160:   const MCOperand &MO = MI->getOperand(OpNo);
```

- EN: Function bodies or method definitions such as printDataSymbol, printConstpool contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printDataSymbol, printConstpool 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161: 
 162:   if (MO.isImm()) {
 163:     if (PrintBranchImmAsAddress) {
 164:       uint64_t Target = Address + MO.getImm();
 165:       Target &= 0xfffffffc;
 166:       O << formatHex(Target);
 167:     } else {
 168:       O << MO.getImm();
 169:     }
 170:     return;
 171:   }
 172: 
 173:   assert(MO.isExpr() && "Unknown operand kind in printConstpool");
 174: 
 175:   O << "[";
 176:   MAI.printExpr(O, *MO.getExpr());
 177:   O << "]";
 178: }
 179: 
 180: void CSKYInstPrinter::printCSKYSymbolOperand(const MCInst *MI, uint64_t Address,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181:                                              unsigned OpNo,
 182:                                              const MCSubtargetInfo &STI,
 183:                                              raw_ostream &O) {
 184:   const MCOperand &MO = MI->getOperand(OpNo);
 185:   if (!MO.isImm()) {
 186:     return printOperand(MI, OpNo, STI, O);
 187:   }
 188: 
 189:   if (PrintBranchImmAsAddress) {
 190:     uint64_t Target = Address + MO.getImm();
 191:     Target &= 0xffffffff;
 192:     O << formatHex(Target);
 193:   } else {
 194:     O << MO.getImm();
 195:   }
 196: }
 197: 
 198: void CSKYInstPrinter::printPSRFlag(const MCInst *MI, unsigned OpNo,
 199:                                    const MCSubtargetInfo &STI, raw_ostream &O) {
 200:   auto V = MI->getOperand(OpNo).getImm();
```

- EN: Function bodies or method definitions such as printPSRFlag contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printPSRFlag 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201: 
 202:   ListSeparator LS;
 203: 
 204:   if ((V >> 3) & 0x1)
 205:     O << LS << "ee";
 206:   if ((V >> 2) & 0x1)
 207:     O << LS << "ie";
 208:   if ((V >> 1) & 0x1)
 209:     O << LS << "fe";
 210:   if ((V >> 0) & 0x1)
 211:     O << LS << "af";
 212: }
 213: 
 214: void CSKYInstPrinter::printRegisterSeq(const MCInst *MI, unsigned OpNum,
 215:                                        const MCSubtargetInfo &STI,
 216:                                        raw_ostream &O) {
 217:   printRegName(O, MI->getOperand(OpNum).getReg());
 218:   O << "-";
 219:   printRegName(O, MI->getOperand(OpNum + 1).getReg());
 220: }
```

- EN: Function bodies or method definitions such as printRegisterSeq contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printRegisterSeq 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221: 
 222: void CSKYInstPrinter::printRegisterList(const MCInst *MI, unsigned OpNum,
 223:                                         const MCSubtargetInfo &STI,
 224:                                         raw_ostream &O) {
 225:   auto V = MI->getOperand(OpNum).getImm();
 226:   ListSeparator LS;
 227: 
 228:   if (V & 0xf) {
 229:     O << LS;
 230:     printRegName(O, CSKY::R4);
 231:     auto Offset = (V & 0xf) - 1;
 232:     if (Offset) {
 233:       O << "-";
 234:       printRegName(O, CSKY::R4 + Offset);
 235:     }
 236:   }
 237: 
 238:   if ((V >> 4) & 0x1) {
 239:     O << LS;
 240:     printRegName(O, CSKY::R15);
```

- EN: Function bodies or method definitions such as printRegisterList contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printRegisterList 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:   }
 242: 
 243:   if ((V >> 5) & 0x7) {
 244:     O << LS;
 245:     printRegName(O, CSKY::R16);
 246: 
 247:     auto Offset = ((V >> 5) & 0x7) - 1;
 248: 
 249:     if (Offset) {
 250:       O << "-";
 251:       printRegName(O, CSKY::R16 + Offset);
 252:     }
 253:   }
 254: 
 255:   if ((V >> 8) & 0x1) {
 256:     O << LS;
 257:     printRegName(O, CSKY::R28);
 258:   }
 259: }
 260: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-272

```cpp
 261: const char *CSKYInstPrinter::getRegisterName(MCRegister Reg) {
 262:   return getRegisterName(Reg, ArchRegNames ? CSKY::NoRegAltName
 263:                                            : CSKY::ABIRegAltName);
 264: }
 265: 
 266: void CSKYInstPrinter::printFPR(const MCInst *MI, unsigned OpNo,
 267:                                const MCSubtargetInfo &STI, raw_ostream &O) {
 268:   const MCOperand &MO = MI->getOperand(OpNo);
 269:   assert(MO.isReg());
 270: 
 271:   printFPRRegName(O, MO.getReg());
 272: }
```

- EN: Function bodies or method definitions such as printFPR contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: printFPR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- MC instruction representation / MC 指令表示
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYInstPrinter.h`, `MCTargetDesc/CSKYBaseInfo.h`, `MCTargetDesc/CSKYMCAsmInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringExtras.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `CSKYGenAsmWriter.inc`
- Local companions / 本地配套文件: `CSKYInstPrinter.h`
