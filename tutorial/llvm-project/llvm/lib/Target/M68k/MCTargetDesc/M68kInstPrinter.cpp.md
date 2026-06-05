# M68kInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/MCTargetDesc/M68kInstPrinter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kInstPrinter.cpp - Convert M68k MCInst to asm --------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains definitions for an M68k MCInst printer.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: // TODO Conform with all supported Motorola ASM syntax
  15: // Motorola's assembly has several syntax variants, especially on
  16: // addressing modes.
  17: // For example, you can write pc indirect w/ displacement as
  18: // `x(%pc)`, where `x` is the displacement imm, or `(x,%pc)`.
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```cpp
  19: // Currently we're picking the variant that is different from
  20: // GCC, albeit being recognizable by GNU AS.
  21: // Not sure what is the impact now (e.g. some syntax might
  22: // not be recognized by some old consoles' toolchains, in which
  23: // case we can not use our integrated assembler), but either way,
  24: // it will be great to support all of the variants in the future.
  25: 
  26: #include "M68kInstPrinter.h"
  27: #include "M68kBaseInfo.h"
  28: 
  29: #include "llvm/ADT/StringExtras.h"
  30: #include "llvm/MC/MCAsmInfo.h"
  31: #include "llvm/MC/MCExpr.h"
  32: #include "llvm/MC/MCInst.h"
  33: #include "llvm/MC/MCInstrInfo.h"
  34: #include "llvm/MC/MCSymbol.h"
  35: #include "llvm/Support/ErrorHandling.h"
  36: #include "llvm/Support/raw_ostream.h"
```
- **EN**: It imports dependencies such as `M68kInstPrinter.h`, `M68kBaseInfo.h`, `StringExtras.h`, `MCAsmInfo.h`, `MCExpr.h`, `MCInst.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `M68kInstPrinter.h`, `M68kBaseInfo.h`, `StringExtras.h`, `MCAsmInfo.h`, `MCExpr.h`, `MCInst.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38: using namespace llvm;
  39: 
  40: #define DEBUG_TYPE "asm-printer"
  41: 
  42: #define PRINT_ALIAS_INSTR
  43: #include "M68kGenAsmWriter.inc"
  44: 
  45: void M68kInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  46:   OS << "%" << getRegisterName(Reg);
  47: }
  48: 
  49: void M68kInstPrinter::printInst(const MCInst *MI, uint64_t Address,
  50:                                 StringRef Annot, const MCSubtargetInfo &STI,
  51:                                 raw_ostream &O) {
  52:   if (!printAliasInstr(MI, Address, O))
  53:     printInstruction(MI, Address, O);
  54: 
```
- **EN**: It imports dependencies such as `M68kGenAsmWriter.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `M68kInstPrinter::printRegName`, `M68kInstPrinter::printInst`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `M68kGenAsmWriter.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `M68kInstPrinter::printRegName`, `M68kInstPrinter::printInst` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   printAnnotation(O, Annot);
  56: }
  57: 
  58: void M68kInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
  59:                                    raw_ostream &O) {
  60:   const MCOperand &MO = MI->getOperand(OpNo);
  61:   if (MO.isReg()) {
  62:     printRegName(O, MO.getReg());
  63:     return;
  64:   }
  65: 
  66:   if (MO.isImm()) {
  67:     printImmediate(MI, OpNo, O);
  68:     return;
  69:   }
  70: 
  71:   assert(MO.isExpr() && "Unknown operand kind in printOperand");
  72:   MAI.printExpr(O, *MO.getExpr());
```
- **EN**: The range implements or declares functions including `M68kInstPrinter::printOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstPrinter::printOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73: }
  74: 
  75: void M68kInstPrinter::printImmediate(const MCInst *MI, unsigned opNum,
  76:                                      raw_ostream &O) {
  77:   const MCOperand &MO = MI->getOperand(opNum);
  78:   if (MO.isImm())
  79:     O << '#' << MO.getImm();
  80:   else if (MO.isExpr()) {
  81:     O << '#';
  82:     MAI.printExpr(O, *MO.getExpr());
  83:   } else
  84:     llvm_unreachable("Unknown immediate kind");
  85: }
  86: 
  87: void M68kInstPrinter::printMoveMask(const MCInst *MI, unsigned opNum,
  88:                                     raw_ostream &O) {
  89:   unsigned Mask = MI->getOperand(opNum).getImm();
  90:   assert((Mask & 0xFFFF) == Mask && "Mask is always 16 bits");
```
- **EN**: The range implements or declares functions including `M68kInstPrinter::printImmediate`, `M68kInstPrinter::printMoveMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstPrinter::printImmediate`, `M68kInstPrinter::printMoveMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92:   // A move mask is splitted into two parts:
  93:   // bits 0 ~ 7  correspond to D0 ~ D7 regs
  94:   // bits 8 ~ 15 correspond to A0 ~ A7 regs
  95:   //
  96:   // In the assembly syntax, we want to use a dash to replace
  97:   // a continuous range of registers. For example, if the bit
  98:   // mask is 0b101110, we want to print "D1-D3,D5" instead of
  99:   // "D1,D2,D3,D4,D5".
 100:   //
 101:   // However, we don't want a dash to cross between data registers
 102:   // and address registers (i.e. there shouldn't be a dash crossing
 103:   // bit 7 and 8) since that is not really intuitive. So we simply
 104:   // print the data register part (bit 0~7) and address register part
 105:   // separately.
 106:   uint8_t HalfMask;
 107:   unsigned Reg;
 108:   for (int s = 0; s < 16; s += 8) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     HalfMask = (Mask >> s) & 0xFF;
 110:     // Print separation comma only if
 111:     // both data & register parts have bit(s) set
 112:     if (s != 0 && (Mask & 0xFF) && HalfMask)
 113:       O << '/';
 114: 
 115:     for (int i = 0; HalfMask; ++i) {
 116:       if ((HalfMask >> i) & 0b1) {
 117:         HalfMask ^= 0b1 << i;
 118:         Reg = M68kII::getMaskedSpillRegister(i + s);
 119:         printRegName(O, Reg);
 120: 
 121:         int j = i;
 122:         while ((HalfMask >> (j + 1)) & 0b1)
 123:           HalfMask ^= 0b1 << ++j;
 124: 
 125:         if (j != i) {
 126:           O << '-';
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 127-144 / 第 127-144 行
```cpp
 127:           Reg = M68kII::getMaskedSpillRegister(j + s);
 128:           printRegName(O, Reg);
 129:         }
 130: 
 131:         i = j;
 132: 
 133:         if (HalfMask)
 134:           O << '/';
 135:       }
 136:     }
 137:   }
 138: }
 139: 
 140: void M68kInstPrinter::printDisp(const MCInst *MI, unsigned opNum,
 141:                                 raw_ostream &O) {
 142:   const MCOperand &Op = MI->getOperand(opNum);
 143:   if (Op.isImm()) {
 144:     O << Op.getImm();
```
- **EN**: The range implements or declares functions including `M68kInstPrinter::printDisp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kInstPrinter::printDisp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     return;
 146:   }
 147:   assert(Op.isExpr() && "Unknown operand kind in printOperand");
 148:   MAI.printExpr(O, *Op.getExpr());
 149: }
 150: 
 151: // NOTE forcing (W,L) size available since M68020 only
 152: void M68kInstPrinter::printAbsMem(const MCInst *MI, unsigned opNum,
 153:                                   raw_ostream &O) {
 154:   const MCOperand &MO = MI->getOperand(opNum);
 155: 
 156:   if (MO.isExpr()) {
 157:     MAI.printExpr(O, *MO.getExpr());
 158:     return;
 159:   }
 160: 
 161:   assert(MO.isImm() && "absolute memory addressing needs an immediate");
 162:   O << format("$%0" PRIx64, (uint64_t)MO.getImm());
```
- **EN**: The range implements or declares functions including `M68kInstPrinter::printAbsMem`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstPrinter::printAbsMem` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-163 / 第 163-163 行
```cpp
 163: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68kInstPrinter.h`
- `M68kBaseInfo.h`
- `llvm/ADT/StringExtras.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCSymbol.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/raw_ostream.h`
- `M68kGenAsmWriter.inc`
