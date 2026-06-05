# SystemZMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZMCInstLower.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file lowers MachineInstr or LLVM constructs to MCInst form for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将 MachineInstr 或 LLVM 构造降低为 MCInst 形式。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===-- SystemZMCInstLower.cpp - Lower MachineInstr to MCInst -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZMCInstLower.h"
  10: #include "MCTargetDesc/SystemZMCAsmInfo.h"
  11: #include "SystemZAsmPrinter.h"
  12: #include "llvm/IR/Mangler.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCInstLower.h`, `SystemZMCAsmInfo.h`, `SystemZAsmPrinter.h`, `Mangler.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCInstLower.h`, `SystemZMCAsmInfo.h`, `SystemZAsmPrinter.h`, `Mangler.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #include "llvm/MC/MCExpr.h"
  14: #include "llvm/MC/MCInst.h"
  15: #include "llvm/MC/MCStreamer.h"
  16: 
  17: using namespace llvm;
  18: 
  19: // Return the S_* enumeration for MachineOperand target flags Flags.
  20: static SystemZ::Specifier getSpecifierForTFlags(unsigned Flags) {
  21:   switch (Flags & SystemZII::MO_SYMBOL_MODIFIER) {
  22:     case 0:
  23:       return SystemZ::S_None;
  24:     case SystemZII::MO_GOT:
```
- **EN**: It imports dependencies such as `MCExpr.h`, `MCInst.h`, `MCStreamer.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `getSpecifierForTFlags`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 它引入了 `MCExpr.h`, `MCInst.h`, `MCStreamer.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `getSpecifierForTFlags` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 25-36 / 第 25-36 行
```cpp
  25:       return SystemZ::S_GOT;
  26:     case SystemZII::MO_INDNTPOFF:
  27:       return SystemZ::S_INDNTPOFF;
  28:   }
  29:   llvm_unreachable("Unrecognised MO_ACCESS_MODEL");
  30: }
  31: 
  32: SystemZMCInstLower::SystemZMCInstLower(MCContext &ctx,
  33:                                        SystemZAsmPrinter &asmprinter)
  34:   : Ctx(ctx), AsmPrinter(asmprinter) {}
  35: 
  36: const MCExpr *SystemZMCInstLower::getExpr(const MachineOperand &MO,
```
- **EN**: The range implements or declares functions including `Ctx`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `Ctx` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 37-48 / 第 37-48 行
```cpp
  37:                                           SystemZ::Specifier Spec) const {
  38:   const MCSymbol *Symbol;
  39:   bool HasOffset = true;
  40:   switch (MO.getType()) {
  41:   case MachineOperand::MO_MachineBasicBlock:
  42:     Symbol = MO.getMBB()->getSymbol();
  43:     HasOffset = false;
  44:     break;
  45: 
  46:   case MachineOperand::MO_GlobalAddress:
  47:     Symbol = AsmPrinter.getSymbol(MO.getGlobal());
  48:     break;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 49-60 / 第 49-60 行
```cpp
  49: 
  50:   case MachineOperand::MO_ExternalSymbol:
  51:     Symbol = AsmPrinter.GetExternalSymbolSymbol(MO.getSymbolName());
  52:     break;
  53: 
  54:   case MachineOperand::MO_JumpTableIndex:
  55:     Symbol = AsmPrinter.GetJTISymbol(MO.getIndex());
  56:     HasOffset = false;
  57:     break;
  58: 
  59:   case MachineOperand::MO_ConstantPoolIndex:
  60:     Symbol = AsmPrinter.GetCPISymbol(MO.getIndex());
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 61-72 / 第 61-72 行
```cpp
  61:     break;
  62: 
  63:   case MachineOperand::MO_BlockAddress:
  64:     Symbol = AsmPrinter.GetBlockAddressSymbol(MO.getBlockAddress());
  65:     break;
  66: 
  67:   default:
  68:     llvm_unreachable("unknown operand type");
  69:   }
  70:   const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, Spec, Ctx);
  71:   if (HasOffset)
  72:     if (int64_t Offset = MO.getOffset()) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-84 / 第 73-84 行
```cpp
  73:       const MCExpr *OffsetExpr = MCConstantExpr::create(Offset, Ctx);
  74:       Expr = MCBinaryExpr::createAdd(Expr, OffsetExpr, Ctx);
  75:     }
  76:   return Expr;
  77: }
  78: 
  79: MCOperand SystemZMCInstLower::lowerOperand(const MachineOperand &MO) const {
  80:   switch (MO.getType()) {
  81:   case MachineOperand::MO_Register:
  82:     return MCOperand::createReg(MO.getReg());
  83: 
  84:   case MachineOperand::MO_Immediate:
```
- **EN**: The range implements or declares functions including `SystemZMCInstLower::lowerOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZMCInstLower::lowerOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 85-96 / 第 85-96 行
```cpp
  85:     return MCOperand::createImm(MO.getImm());
  86: 
  87:   default: {
  88:     auto Kind = getSpecifierForTFlags(MO.getTargetFlags());
  89:     return MCOperand::createExpr(getExpr(MO, Kind));
  90:   }
  91:   }
  92: }
  93: 
  94: void SystemZMCInstLower::lower(const MachineInstr *MI, MCInst &OutMI) const {
  95:   OutMI.setOpcode(MI->getOpcode());
  96:   for (const MachineOperand &MO : MI->operands())
```
- **EN**: The range implements or declares functions including `SystemZMCInstLower::lower`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZMCInstLower::lower` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 97-100 / 第 97-100 行
```cpp
  97:     // Ignore all implicit register operands.
  98:     if (!MO.isReg() || !MO.isImplicit())
  99:       OutMI.addOperand(lowerOperand(MO));
 100: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `SystemZMCInstLower.h`
- `MCTargetDesc/SystemZMCAsmInfo.h`
- `SystemZAsmPrinter.h`
- `llvm/IR/Mangler.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCStreamer.h`
