# SystemZInstPrinterCommon.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZInstPrinterCommon.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- SystemZInstPrinterCommon.cpp - Common SystemZ MCInst to assembly funcs -=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZInstPrinterCommon.h"
  10: #include "MCTargetDesc/SystemZMCAsmInfo.h"
  11: #include "llvm/MC/MCExpr.h"
  12: #include "llvm/MC/MCInst.h"
  13: #include "llvm/MC/MCRegister.h"
  14: #include "llvm/MC/MCSymbol.h"
  15: #include "llvm/Support/Casting.h"
  16: #include "llvm/Support/ErrorHandling.h"
  17: #include "llvm/Support/MathExtras.h"
  18: #include "llvm/Support/raw_ostream.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZInstPrinterCommon.h`, `SystemZMCAsmInfo.h`, `MCExpr.h`, `MCInst.h`, `MCRegister.h`, `MCSymbol.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZInstPrinterCommon.h`, `SystemZMCAsmInfo.h`, `MCExpr.h`, `MCInst.h`, `MCRegister.h`, `MCSymbol.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include <cassert>
  20: #include <cstdint>
  21: 
  22: using namespace llvm;
  23: 
  24: #define DEBUG_TYPE "asm-printer"
  25: 
  26: void SystemZInstPrinterCommon::printAddress(const MCAsmInfo *MAI,
  27:                                             MCRegister Base,
  28:                                             const MCOperand &DispMO,
  29:                                             MCRegister Index, raw_ostream &O) {
  30:   printOperand(DispMO, MAI, O);
  31:   if (Base || Index) {
  32:     O << '(';
  33:     if (Index) {
  34:       printRegName(O, Index);
  35:       O << ',';
  36:     }
```
- **EN**: It imports dependencies such as `cassert`, `cstdint` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `SystemZInstPrinterCommon::printAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `cassert`, `cstdint` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `SystemZInstPrinterCommon::printAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     if (Base)
  38:       printRegName(O, Base);
  39:     else
  40:       O << '0';
  41:     O << ')';
  42:   }
  43: }
  44: 
  45: void SystemZInstPrinterCommon::printOperand(const MCOperand &MO,
  46:                                             const MCAsmInfo *MAI,
  47:                                             raw_ostream &O) {
  48:   if (MO.isReg()) {
  49:     if (!MO.getReg())
  50:       O << '0';
  51:     else
  52:       printRegName(O, MO.getReg());
  53:   } else if (MO.isImm())
  54:     markup(O, Markup::Immediate) << MO.getImm();
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   else if (MO.isExpr())
  56:     MAI->printExpr(O, *MO.getExpr());
  57:   else
  58:     llvm_unreachable("Invalid operand");
  59: }
  60: 
  61: void SystemZInstPrinterCommon::printRegName(raw_ostream &O, MCRegister Reg) {
  62:   printFormattedRegName(&MAI, Reg, O);
  63: }
  64: 
  65: template <unsigned N>
  66: void SystemZInstPrinterCommon::printUImmOperand(const MCInst *MI, int OpNum,
  67:                                                 raw_ostream &O) {
  68:   const MCOperand &MO = MI->getOperand(OpNum);
  69:   if (MO.isExpr()) {
  70:     MAI.printExpr(O, *MO.getExpr());
  71:     return;
  72:   }
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printRegName`, `SystemZInstPrinterCommon::printUImmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printRegName`, `SystemZInstPrinterCommon::printUImmOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   uint64_t Value = static_cast<uint64_t>(MO.getImm());
  74:   assert(isUInt<N>(Value) && "Invalid uimm argument");
  75:   markup(O, Markup::Immediate) << Value;
  76: }
  77: 
  78: template <unsigned N>
  79: void SystemZInstPrinterCommon::printSImmOperand(const MCInst *MI, int OpNum,
  80:                                                 raw_ostream &O) {
  81:   const MCOperand &MO = MI->getOperand(OpNum);
  82:   if (MO.isExpr()) {
  83:     MAI.printExpr(O, *MO.getExpr());
  84:     return;
  85:   }
  86:   int64_t Value = MI->getOperand(OpNum).getImm();
  87:   assert(isInt<N>(Value) && "Invalid simm argument");
  88:   markup(O, Markup::Immediate) << Value;
  89: }
  90: 
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printSImmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printSImmOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91: void SystemZInstPrinterCommon::printU1ImmOperand(const MCInst *MI, int OpNum,
  92:                                                  raw_ostream &O) {
  93:   printUImmOperand<1>(MI, OpNum, O);
  94: }
  95: 
  96: void SystemZInstPrinterCommon::printU2ImmOperand(const MCInst *MI, int OpNum,
  97:                                                  raw_ostream &O) {
  98:   printUImmOperand<2>(MI, OpNum, O);
  99: }
 100: 
 101: void SystemZInstPrinterCommon::printU3ImmOperand(const MCInst *MI, int OpNum,
 102:                                                  raw_ostream &O) {
 103:   printUImmOperand<3>(MI, OpNum, O);
 104: }
 105: 
 106: void SystemZInstPrinterCommon::printU4ImmOperand(const MCInst *MI, int OpNum,
 107:                                                  raw_ostream &O) {
 108:   printUImmOperand<4>(MI, OpNum, O);
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printU1ImmOperand`, `SystemZInstPrinterCommon::printU2ImmOperand`, `SystemZInstPrinterCommon::printU3ImmOperand`, `SystemZInstPrinterCommon::printU4ImmOperand`.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printU1ImmOperand`, `SystemZInstPrinterCommon::printU2ImmOperand`, `SystemZInstPrinterCommon::printU3ImmOperand`, `SystemZInstPrinterCommon::printU4ImmOperand` 等函数。

### Lines 109-126 / 第 109-126 行
```cpp
 109: }
 110: 
 111: void SystemZInstPrinterCommon::printS8ImmOperand(const MCInst *MI, int OpNum,
 112:                                                  raw_ostream &O) {
 113:   printSImmOperand<8>(MI, OpNum, O);
 114: }
 115: 
 116: void SystemZInstPrinterCommon::printU8ImmOperand(const MCInst *MI, int OpNum,
 117:                                                  raw_ostream &O) {
 118:   printUImmOperand<8>(MI, OpNum, O);
 119: }
 120: 
 121: void SystemZInstPrinterCommon::printU12ImmOperand(const MCInst *MI, int OpNum,
 122:                                                   raw_ostream &O) {
 123:   printUImmOperand<12>(MI, OpNum, O);
 124: }
 125: 
 126: void SystemZInstPrinterCommon::printS16ImmOperand(const MCInst *MI, int OpNum,
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printS8ImmOperand`, `SystemZInstPrinterCommon::printU8ImmOperand`, `SystemZInstPrinterCommon::printU12ImmOperand`.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printS8ImmOperand`, `SystemZInstPrinterCommon::printU8ImmOperand`, `SystemZInstPrinterCommon::printU12ImmOperand` 等函数。

### Lines 127-144 / 第 127-144 行
```cpp
 127:                                                   raw_ostream &O) {
 128:   printSImmOperand<16>(MI, OpNum, O);
 129: }
 130: 
 131: void SystemZInstPrinterCommon::printU16ImmOperand(const MCInst *MI, int OpNum,
 132:                                                   raw_ostream &O) {
 133:   printUImmOperand<16>(MI, OpNum, O);
 134: }
 135: 
 136: void SystemZInstPrinterCommon::printS32ImmOperand(const MCInst *MI, int OpNum,
 137:                                                   raw_ostream &O) {
 138:   printSImmOperand<32>(MI, OpNum, O);
 139: }
 140: 
 141: void SystemZInstPrinterCommon::printU32ImmOperand(const MCInst *MI, int OpNum,
 142:                                                   raw_ostream &O) {
 143:   printUImmOperand<32>(MI, OpNum, O);
 144: }
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printU16ImmOperand`, `SystemZInstPrinterCommon::printS32ImmOperand`, `SystemZInstPrinterCommon::printU32ImmOperand`.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printU16ImmOperand`, `SystemZInstPrinterCommon::printS32ImmOperand`, `SystemZInstPrinterCommon::printU32ImmOperand` 等函数。

### Lines 145-162 / 第 145-162 行
```cpp
 145: 
 146: void SystemZInstPrinterCommon::printU48ImmOperand(const MCInst *MI, int OpNum,
 147:                                                   raw_ostream &O) {
 148:   printUImmOperand<48>(MI, OpNum, O);
 149: }
 150: 
 151: void SystemZInstPrinterCommon::printPCRelOperand(const MCInst *MI,
 152:                                                  uint64_t Address, int OpNum,
 153:                                                  raw_ostream &O) {
 154:   const MCOperand &MO = MI->getOperand(OpNum);
 155: 
 156:   // If the label has already been resolved to an immediate offset (say, when
 157:   // we're running the disassembler), just print the immediate.
 158:   if (MO.isImm()) {
 159:     int64_t Offset = MO.getImm();
 160:     if (PrintBranchImmAsAddress)
 161:       markup(O, Markup::Target) << formatHex(Address + Offset);
 162:     else
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printU48ImmOperand`, `SystemZInstPrinterCommon::printPCRelOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printU48ImmOperand`, `SystemZInstPrinterCommon::printPCRelOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:       markup(O, Markup::Immediate) << formatImm(Offset);
 164:     return;
 165:   }
 166: 
 167:   // If the branch target is simply an address then print it in hex.
 168:   const MCConstantExpr *BranchTarget = dyn_cast<MCConstantExpr>(MO.getExpr());
 169:   int64_t TargetAddress;
 170:   if (BranchTarget && BranchTarget->evaluateAsAbsolute(TargetAddress)) {
 171:     markup(O, Markup::Target) << formatHex((uint64_t)TargetAddress);
 172:   } else {
 173:     // Otherwise, just print the expression.
 174:     MAI.printExpr(O, *MO.getExpr());
 175:   }
 176: }
 177: 
 178: void SystemZInstPrinterCommon::printPCRelTLSOperand(const MCInst *MI,
 179:                                                     uint64_t Address, int OpNum,
 180:                                                     raw_ostream &O) {
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printPCRelTLSOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printPCRelTLSOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   // Output the PC-relative operand.
 182:   printPCRelOperand(MI, Address, OpNum, O);
 183: 
 184:   // Output the TLS marker if present.
 185:   if ((unsigned)OpNum + 1 < MI->getNumOperands()) {
 186:     const MCOperand &MO = MI->getOperand(OpNum + 1);
 187:     const MCSymbolRefExpr &RefExp = cast<MCSymbolRefExpr>(*MO.getExpr());
 188:     switch (RefExp.getSpecifier()) {
 189:     case SystemZ::S_TLSGD:
 190:       O << ":tls_gdcall:";
 191:       break;
 192:     case SystemZ::S_TLSLDM:
 193:       O << ":tls_ldcall:";
 194:       break;
 195:     default:
 196:       llvm_unreachable("Unexpected symbol kind");
 197:     }
 198:     O << RefExp.getSymbol().getName();
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   }
 200: }
 201: 
 202: void SystemZInstPrinterCommon::printOperand(const MCInst *MI, int OpNum,
 203:                                             raw_ostream &O) {
 204:   printOperand(MI->getOperand(OpNum), &MAI, O);
 205: }
 206: 
 207: void SystemZInstPrinterCommon::printBDAddrOperand(const MCInst *MI, int OpNum,
 208:                                                   raw_ostream &O) {
 209:   printAddress(&MAI, MI->getOperand(OpNum).getReg(), MI->getOperand(OpNum + 1),
 210:                0, O);
 211: }
 212: 
 213: void SystemZInstPrinterCommon::printBDXAddrOperand(const MCInst *MI, int OpNum,
 214:                                                    raw_ostream &O) {
 215:   printAddress(&MAI, MI->getOperand(OpNum).getReg(), MI->getOperand(OpNum + 1),
 216:                MI->getOperand(OpNum + 2).getReg(), O);
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printOperand`, `SystemZInstPrinterCommon::printBDAddrOperand`, `SystemZInstPrinterCommon::printBDXAddrOperand`.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printOperand`, `SystemZInstPrinterCommon::printBDAddrOperand`, `SystemZInstPrinterCommon::printBDXAddrOperand` 等函数。

### Lines 217-234 / 第 217-234 行
```cpp
 217: }
 218: 
 219: void SystemZInstPrinterCommon::printBDLAddrOperand(const MCInst *MI, int OpNum,
 220:                                                    raw_ostream &O) {
 221:   MCRegister Base = MI->getOperand(OpNum).getReg();
 222:   const MCOperand &DispMO = MI->getOperand(OpNum + 1);
 223:   uint64_t Length = MI->getOperand(OpNum + 2).getImm();
 224:   printOperand(DispMO, &MAI, O);
 225:   O << '(' << Length;
 226:   if (Base) {
 227:     O << ",";
 228:     printRegName(O, Base);
 229:   }
 230:   O << ')';
 231: }
 232: 
 233: void SystemZInstPrinterCommon::printBDRAddrOperand(const MCInst *MI, int OpNum,
 234:                                                    raw_ostream &O) {
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printBDLAddrOperand`, `SystemZInstPrinterCommon::printBDRAddrOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printBDLAddrOperand`, `SystemZInstPrinterCommon::printBDRAddrOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   MCRegister Base = MI->getOperand(OpNum).getReg();
 236:   const MCOperand &DispMO = MI->getOperand(OpNum + 1);
 237:   MCRegister Length = MI->getOperand(OpNum + 2).getReg();
 238:   printOperand(DispMO, &MAI, O);
 239:   O << "(";
 240:   printRegName(O, Length);
 241:   if (Base) {
 242:     O << ",";
 243:     printRegName(O, Base);
 244:   }
 245:   O << ')';
 246: }
 247: 
 248: void SystemZInstPrinterCommon::printBDVAddrOperand(const MCInst *MI, int OpNum,
 249:                                                    raw_ostream &O) {
 250:   printAddress(&MAI, MI->getOperand(OpNum).getReg(), MI->getOperand(OpNum + 1),
 251:                MI->getOperand(OpNum + 2).getReg(), O);
 252: }
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printBDVAddrOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printBDVAddrOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-268 / 第 253-268 行
```cpp
 253: 
 254: void SystemZInstPrinterCommon::printLXAAddrOperand(const MCInst *MI, int OpNum,
 255:                                              raw_ostream &O) {
 256:   printAddress(&MAI, MI->getOperand(OpNum).getReg(), MI->getOperand(OpNum + 1),
 257:                MI->getOperand(OpNum + 2).getReg(), O);
 258: }
 259: 
 260: void SystemZInstPrinterCommon::printCond4Operand(const MCInst *MI, int OpNum,
 261:                                                  raw_ostream &O) {
 262:   static const char *const CondNames[] = {"o",  "h",  "nle", "l",   "nhe",
 263:                                           "lh", "ne", "e",   "nlh", "he",
 264:                                           "nl", "le", "nh",  "no"};
 265:   uint64_t Imm = MI->getOperand(OpNum).getImm();
 266:   assert(Imm > 0 && Imm < 15 && "Invalid condition");
 267:   O << CondNames[Imm - 1];
 268: }
```
- **EN**: The range implements or declares functions including `SystemZInstPrinterCommon::printLXAAddrOperand`, `SystemZInstPrinterCommon::printCond4Operand`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstPrinterCommon::printLXAAddrOperand`, `SystemZInstPrinterCommon::printCond4Operand` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `SystemZInstPrinterCommon.h`
- `MCTargetDesc/SystemZMCAsmInfo.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCRegister.h`
- `llvm/MC/MCSymbol.h`
- `llvm/Support/Casting.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/MathExtras.h`
- `llvm/Support/raw_ostream.h`
- `cassert`
- `cstdint`
