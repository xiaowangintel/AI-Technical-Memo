# ARCInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/MCTargetDesc/ARCInstPrinter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCInstPrinter.cpp - ARC MCInst to assembly syntax -------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This class prints an ARC MCInst to a .s file.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "ARCInstPrinter.h"
  14: #include "MCTargetDesc/ARCInfo.h"
  15: #include "llvm/ADT/StringExtras.h"
  16: #include "llvm/MC/MCExpr.h"
  17: #include "llvm/MC/MCInst.h"
  18: #include "llvm/MC/MCInstrInfo.h"
  19: #include "llvm/MC/MCSymbol.h"
  20: #include "llvm/Support/Casting.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. This chunk introduces interfaces or data structures such as prints, which organize the target-specific behavior exposed by the file.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 这一段引入了 prints 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 21-40

```cpp
  21: #include "llvm/Support/Debug.h"
  22: #include "llvm/Support/raw_ostream.h"
  23: 
  24: using namespace llvm;
  25: 
  26: #define DEBUG_TYPE "asm-printer"
  27: 
  28: #include "ARCGenAsmWriter.inc"
  29: 
  30: template <class T>
  31: static const char *BadConditionCode(T cc) {
  32:   LLVM_DEBUG(dbgs() << "Unknown condition code passed: " << cc << "\n");
  33:   return "{unknown-cc}";
  34: }
  35: 
  36: static const char *ARCBRCondCodeToString(ARCCC::BRCondCode BRCC) {
  37:   switch (BRCC) {
  38:   case ARCCC::BREQ:
  39:     return "eq";
  40:   case ARCCC::BRNE:
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as T, which organize the target-specific behavior exposed by the file. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 T 等接口或数据结构，用于组织该文件暴露的目标专用行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 41-60

```cpp
  41:     return "ne";
  42:   case ARCCC::BRLT:
  43:     return "lt";
  44:   case ARCCC::BRGE:
  45:     return "ge";
  46:   case ARCCC::BRLO:
  47:     return "lo";
  48:   case ARCCC::BRHS:
  49:     return "hs";
  50:   }
  51:   return BadConditionCode(BRCC);
  52: }
  53: 
  54: static const char *ARCCondCodeToString(ARCCC::CondCode CC) {
  55:   switch (CC) {
  56:   case ARCCC::EQ:
  57:     return "eq";
  58:   case ARCCC::NE:
  59:     return "ne";
  60:   case ARCCC::P:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 61-80

```cpp
  61:     return "p";
  62:   case ARCCC::N:
  63:     return "n";
  64:   case ARCCC::HS:
  65:     return "hs";
  66:   case ARCCC::LO:
  67:     return "lo";
  68:   case ARCCC::GT:
  69:     return "gt";
  70:   case ARCCC::GE:
  71:     return "ge";
  72:   case ARCCC::VS:
  73:     return "vs";
  74:   case ARCCC::VC:
  75:     return "vc";
  76:   case ARCCC::LT:
  77:     return "lt";
  78:   case ARCCC::LE:
  79:     return "le";
  80:   case ARCCC::HI:
```

- EN: This range continues the implementation of the backend component described by ARCInstPrinter.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:     return "hi";
  82:   case ARCCC::LS:
  83:     return "ls";
  84:   case ARCCC::PNZ:
  85:     return "pnz";
  86:   case ARCCC::AL:
  87:     return "al";
  88:   case ARCCC::NZ:
  89:     return "nz";
  90:   case ARCCC::Z:
  91:     return "z";
  92:   }
  93:   return BadConditionCode(CC);
  94: }
  95: 
  96: void ARCInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  97:   OS << StringRef(getRegisterName(Reg)).lower();
  98: }
  99: 
 100: void ARCInstPrinter::printInst(const MCInst *MI, uint64_t Address,
```

- EN: Function bodies or method definitions such as printRegName contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: printRegName 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 101-120

```cpp
 101:                                StringRef Annot, const MCSubtargetInfo &STI,
 102:                                raw_ostream &O) {
 103:   printInstruction(MI, Address, O);
 104:   printAnnotation(O, Annot);
 105: }
 106: 
 107: static void printExpr(const MCExpr *Expr, const MCAsmInfo *MAI,
 108:                       raw_ostream &OS) {
 109:   int Offset = 0;
 110:   const MCSymbolRefExpr *SRE;
 111: 
 112:   if (const auto *CE = dyn_cast<MCConstantExpr>(Expr)) {
 113:     OS << "0x";
 114:     OS.write_hex(CE->getValue());
 115:     return;
 116:   }
 117: 
 118:   if (const auto *BE = dyn_cast<MCBinaryExpr>(Expr)) {
 119:     SRE = dyn_cast<MCSymbolRefExpr>(BE->getLHS());
 120:     const auto *CE = dyn_cast<MCConstantExpr>(BE->getRHS());
```

- EN: Function bodies or method definitions such as printExpr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printExpr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:     assert(SRE && CE && "Binary expression must be sym+const.");
 122:     Offset = CE->getValue();
 123:   } else {
 124:     SRE = dyn_cast<MCSymbolRefExpr>(Expr);
 125:     assert(SRE && "Unexpected MCExpr type.");
 126:   }
 127:   assert(SRE->getSpecifier() == 0);
 128: 
 129:   // Symbols are prefixed with '@'
 130:   OS << '@';
 131:   SRE->getSymbol().print(OS, MAI);
 132: 
 133:   if (Offset) {
 134:     if (Offset > 0)
 135:       OS << '+';
 136:     OS << Offset;
 137:   }
 138: }
 139: 
 140: void ARCInstPrinter::printOperand(const MCInst *MI, unsigned OpNum,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:                                   raw_ostream &O) {
 142:   const MCOperand &Op = MI->getOperand(OpNum);
 143:   if (Op.isReg()) {
 144:     printRegName(O, Op.getReg());
 145:     return;
 146:   }
 147: 
 148:   if (Op.isImm()) {
 149:     O << Op.getImm();
 150:     return;
 151:   }
 152: 
 153:   assert(Op.isExpr() && "unknown operand kind in printOperand");
 154:   printExpr(Op.getExpr(), &MAI, O);
 155: }
 156: 
 157: void ARCInstPrinter::printMemOperandRI(const MCInst *MI, unsigned OpNum,
 158:                                        raw_ostream &O) {
 159:   const MCOperand &base = MI->getOperand(OpNum);
 160:   const MCOperand &offset = MI->getOperand(OpNum + 1);
```

- EN: Function bodies or method definitions such as printMemOperandRI contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printMemOperandRI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:   assert(base.isReg() && "Base should be register.");
 162:   assert(offset.isImm() && "Offset should be immediate.");
 163:   printRegName(O, base.getReg());
 164:   O << "," << offset.getImm();
 165: }
 166: 
 167: void ARCInstPrinter::printPredicateOperand(const MCInst *MI, unsigned OpNum,
 168:                                            raw_ostream &O) {
 169: 
 170:   const MCOperand &Op = MI->getOperand(OpNum);
 171:   assert(Op.isImm() && "Predicate operand is immediate.");
 172:   O << ARCCondCodeToString((ARCCC::CondCode)Op.getImm());
 173: }
 174: 
 175: void ARCInstPrinter::printBRCCPredicateOperand(const MCInst *MI, unsigned OpNum,
 176:                                                raw_ostream &O) {
 177:   const MCOperand &Op = MI->getOperand(OpNum);
 178:   assert(Op.isImm() && "Predicate operand is immediate.");
 179:   O << ARCBRCondCodeToString((ARCCC::BRCondCode)Op.getImm());
 180: }
```

- EN: Function bodies or method definitions such as printPredicateOperand, printBRCCPredicateOperand contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: printPredicateOperand, printBRCCPredicateOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 181-200

```cpp
 181: 
 182: void ARCInstPrinter::printCCOperand(const MCInst *MI, int OpNum,
 183:                                     raw_ostream &O) {
 184:   O << ARCCondCodeToString((ARCCC::CondCode)MI->getOperand(OpNum).getImm());
 185: }
 186: 
 187: void ARCInstPrinter::printU6ShiftedBy(unsigned ShiftBy, const MCInst *MI,
 188:                                       int OpNum, raw_ostream &O) {
 189:   const MCOperand &MO = MI->getOperand(OpNum);
 190:   if (MO.isImm()) {
 191:     unsigned Value = MO.getImm();
 192:     unsigned Value2 = Value >> ShiftBy;
 193:     if (Value2 > 0x3F || (Value2 << ShiftBy != Value)) {
 194:       errs() << "!!! Instruction has out-of-range U6 immediate operand:\n"
 195:              << "    Opcode is " << MI->getOpcode() << "; operand value is "
 196:              << Value;
 197:       if (ShiftBy)
 198:         errs() << " scaled by " << (1 << ShiftBy) << "\n";
 199:       assert(false && "instruction has wrong format");
 200:     }
```

- EN: Function bodies or method definitions such as printCCOperand, printU6ShiftedBy contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: printCCOperand, printU6ShiftedBy 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-207

```cpp
 201:   }
 202:   printOperand(MI, OpNum, O);
 203: }
 204: 
 205: void ARCInstPrinter::printU6(const MCInst *MI, int OpNum, raw_ostream &O) {
 206:   printU6ShiftedBy(0, MI, OpNum, O);
 207: }
```

- EN: Function bodies or method definitions such as printU6 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: printU6 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- MC instruction representation / MC 指令表示
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCInstPrinter.h`, `MCTargetDesc/ARCInfo.h`, `llvm/ADT/StringExtras.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/Casting.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `ARCGenAsmWriter.inc`
- Local companions / 本地配套文件: `ARCInstPrinter.h`
