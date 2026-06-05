# LoongArchMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchMCInstLower.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将 MachineInstr 或 LLVM 构造降低为 MCInst 形式。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- LoongArchMCInstLower.cpp - Convert LoongArch MachineInstr to an MCInst -=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains code to lower LoongArch MachineInstrs to their
  10: // corresponding MCInst records.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArch.h"
  15: #include "MCTargetDesc/LoongArchBaseInfo.h"
  16: #include "MCTargetDesc/LoongArchMCAsmInfo.h"
  17: #include "llvm/BinaryFormat/ELF.h"
  18: #include "llvm/CodeGen/AsmPrinter.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArch.h`, `LoongArchBaseInfo.h`, `LoongArchMCAsmInfo.h`, `ELF.h`, `AsmPrinter.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArch.h`, `LoongArchBaseInfo.h`, `LoongArchMCAsmInfo.h`, `ELF.h`, `AsmPrinter.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/CodeGen/MachineBasicBlock.h"
  20: #include "llvm/CodeGen/MachineInstr.h"
  21: #include "llvm/MC/MCAsmInfo.h"
  22: #include "llvm/MC/MCContext.h"
  23: 
  24: using namespace llvm;
  25: 
  26: static MCOperand lowerSymbolOperand(const MachineOperand &MO, MCSymbol *Sym,
  27:                                     const AsmPrinter &AP) {
  28:   MCContext &Ctx = AP.OutContext;
  29:   uint16_t Kind = 0;
  30: 
  31:   switch (LoongArchII::getDirectFlags(MO)) {
  32:   default:
  33:     llvm_unreachable("Unknown target flag on GV operand");
  34:   case LoongArchII::MO_None:
  35:     Kind = LoongArchMCExpr::VK_None;
  36:     break;
```
- **EN**: It imports dependencies such as `MachineBasicBlock.h`, `MachineInstr.h`, `MCAsmInfo.h`, `MCContext.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `lowerSymbolOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 它引入了 `MachineBasicBlock.h`, `MachineInstr.h`, `MCAsmInfo.h`, `MCContext.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `lowerSymbolOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   case LoongArchII::MO_CALL:
  38:   case LoongArchII::MO_CALL_PLT:
  39:     Kind = ELF::R_LARCH_B26;
  40:     break;
  41:   case LoongArchII::MO_PCREL_HI:
  42:     Kind = ELF::R_LARCH_PCALA_HI20;
  43:     break;
  44:   case LoongArchII::MO_PCREL_LO:
  45:     Kind = ELF::R_LARCH_PCALA_LO12;
  46:     break;
  47:   case LoongArchII::MO_PCREL64_LO:
  48:     Kind = ELF::R_LARCH_PCALA64_LO20;
  49:     break;
  50:   case LoongArchII::MO_PCREL64_HI:
  51:     Kind = ELF::R_LARCH_PCALA64_HI12;
  52:     break;
  53:   case LoongArchII::MO_GOT_PC_HI:
  54:     Kind = ELF::R_LARCH_GOT_PC_HI20;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:     break;
  56:   case LoongArchII::MO_GOT_PC_LO:
  57:     Kind = ELF::R_LARCH_GOT_PC_LO12;
  58:     break;
  59:   case LoongArchII::MO_GOT_PC64_LO:
  60:     Kind = ELF::R_LARCH_GOT64_PC_LO20;
  61:     break;
  62:   case LoongArchII::MO_GOT_PC64_HI:
  63:     Kind = ELF::R_LARCH_GOT64_PC_HI12;
  64:     break;
  65:   case LoongArchII::MO_LE_HI:
  66:     Kind = ELF::R_LARCH_TLS_LE_HI20;
  67:     break;
  68:   case LoongArchII::MO_LE_LO:
  69:     Kind = ELF::R_LARCH_TLS_LE_LO12;
  70:     break;
  71:   case LoongArchII::MO_LE64_LO:
  72:     Kind = ELF::R_LARCH_TLS_LE64_LO20;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     break;
  74:   case LoongArchII::MO_LE64_HI:
  75:     Kind = ELF::R_LARCH_TLS_LE64_HI12;
  76:     break;
  77:   case LoongArchII::MO_IE_PC_HI:
  78:     Kind = ELF::R_LARCH_TLS_IE_PC_HI20;
  79:     break;
  80:   case LoongArchII::MO_IE_PC_LO:
  81:     Kind = ELF::R_LARCH_TLS_IE_PC_LO12;
  82:     break;
  83:   case LoongArchII::MO_IE_PC64_LO:
  84:     Kind = ELF::R_LARCH_TLS_IE64_PC_LO20;
  85:     break;
  86:   case LoongArchII::MO_IE_PC64_HI:
  87:     Kind = ELF::R_LARCH_TLS_IE64_PC_HI12;
  88:     break;
  89:   case LoongArchII::MO_LD_PC_HI:
  90:     Kind = ELF::R_LARCH_TLS_LD_PC_HI20;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     break;
  92:   case LoongArchII::MO_GD_PC_HI:
  93:     Kind = ELF::R_LARCH_TLS_GD_PC_HI20;
  94:     break;
  95:   case LoongArchII::MO_CALL30:
  96:     Kind = ELF::R_LARCH_CALL30;
  97:     break;
  98:   case LoongArchII::MO_CALL36:
  99:     Kind = ELF::R_LARCH_CALL36;
 100:     break;
 101:   case LoongArchII::MO_DESC_PC_HI:
 102:     Kind = ELF::R_LARCH_TLS_DESC_PC_HI20;
 103:     break;
 104:   case LoongArchII::MO_DESC_PC_LO:
 105:     Kind = ELF::R_LARCH_TLS_DESC_PC_LO12;
 106:     break;
 107:   case LoongArchII::MO_DESC64_PC_LO:
 108:     Kind = ELF::R_LARCH_TLS_DESC64_PC_LO20;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     break;
 110:   case LoongArchII::MO_DESC64_PC_HI:
 111:     Kind = ELF::R_LARCH_TLS_DESC64_PC_HI12;
 112:     break;
 113:   case LoongArchII::MO_DESC_LD:
 114:     Kind = ELF::R_LARCH_TLS_DESC_LD;
 115:     break;
 116:   case LoongArchII::MO_DESC_CALL:
 117:     Kind = ELF::R_LARCH_TLS_DESC_CALL;
 118:     break;
 119:   case LoongArchII::MO_LE_HI_R:
 120:     Kind = ELF::R_LARCH_TLS_LE_HI20_R;
 121:     break;
 122:   case LoongArchII::MO_LE_ADD_R:
 123:     Kind = ELF::R_LARCH_TLS_LE_ADD_R;
 124:     break;
 125:   case LoongArchII::MO_LE_LO_R:
 126:     Kind = ELF::R_LARCH_TLS_LE_LO12_R;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     break;
 128:   case LoongArchII::MO_PCADD_HI:
 129:     Kind = ELF::R_LARCH_PCADD_HI20;
 130:     break;
 131:   case LoongArchII::MO_PCADD_LO:
 132:     Kind = ELF::R_LARCH_PCADD_LO12;
 133:     break;
 134:   case LoongArchII::MO_GOT_PCADD_HI:
 135:     Kind = ELF::R_LARCH_GOT_PCADD_HI20;
 136:     break;
 137:   case LoongArchII::MO_GOT_PCADD_LO:
 138:     Kind = ELF::R_LARCH_GOT_PCADD_LO12;
 139:     break;
 140:   case LoongArchII::MO_IE_PCADD_HI:
 141:     Kind = ELF::R_LARCH_TLS_IE_PCADD_HI20;
 142:     break;
 143:   case LoongArchII::MO_IE_PCADD_LO:
 144:     Kind = ELF::R_LARCH_TLS_IE_PCADD_LO12;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     break;
 146:   case LoongArchII::MO_LD_PCADD_HI:
 147:     Kind = ELF::R_LARCH_TLS_LD_PCADD_HI20;
 148:     break;
 149:   case LoongArchII::MO_LD_PCADD_LO:
 150:     Kind = ELF::R_LARCH_TLS_LD_PCADD_LO12;
 151:     break;
 152:   case LoongArchII::MO_GD_PCADD_HI:
 153:     Kind = ELF::R_LARCH_TLS_GD_PCADD_HI20;
 154:     break;
 155:   case LoongArchII::MO_GD_PCADD_LO:
 156:     Kind = ELF::R_LARCH_TLS_GD_PCADD_LO12;
 157:     break;
 158:   case LoongArchII::MO_DESC_PCADD_HI:
 159:     Kind = ELF::R_LARCH_TLS_DESC_PCADD_HI20;
 160:     break;
 161:   case LoongArchII::MO_DESC_PCADD_LO:
 162:     Kind = ELF::R_LARCH_TLS_DESC_PCADD_LO12;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     break;
 164:     // TODO: Handle more target-flags.
 165:   }
 166: 
 167:   const MCExpr *ME = MCSymbolRefExpr::create(Sym, Ctx);
 168: 
 169:   if (!MO.isJTI() && !MO.isMBB() && MO.getOffset())
 170:     ME = MCBinaryExpr::createAdd(
 171:         ME, MCConstantExpr::create(MO.getOffset(), Ctx), Ctx);
 172: 
 173:   if (Kind != LoongArchMCExpr::VK_None)
 174:     ME = LoongArchMCExpr::create(ME, Kind, Ctx, LoongArchII::hasRelaxFlag(MO));
 175:   return MCOperand::createExpr(ME);
 176: }
 177: 
 178: bool llvm::lowerLoongArchMachineOperandToMCOperand(const MachineOperand &MO,
 179:                                                    MCOperand &MCOp,
 180:                                                    const AsmPrinter &AP) {
```
- **EN**: The range implements or declares functions including `llvm::lowerLoongArchMachineOperandToMCOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `llvm::lowerLoongArchMachineOperandToMCOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   switch (MO.getType()) {
 182:   default:
 183:     report_fatal_error(
 184:         "lowerLoongArchMachineOperandToMCOperand: unknown operand type");
 185:   case MachineOperand::MO_Register:
 186:     // Ignore all implicit register operands.
 187:     if (MO.isImplicit())
 188:       return false;
 189:     MCOp = MCOperand::createReg(MO.getReg());
 190:     break;
 191:   case MachineOperand::MO_RegisterMask:
 192:     // Regmasks are like implicit defs.
 193:     return false;
 194:   case MachineOperand::MO_Immediate:
 195:     MCOp = MCOperand::createImm(MO.getImm());
 196:     break;
 197:   case MachineOperand::MO_ConstantPoolIndex:
 198:     MCOp = lowerSymbolOperand(MO, AP.GetCPISymbol(MO.getIndex()), AP);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:     break;
 200:   case MachineOperand::MO_GlobalAddress:
 201:     MCOp = lowerSymbolOperand(MO, AP.getSymbolPreferLocal(*MO.getGlobal()), AP);
 202:     break;
 203:   case MachineOperand::MO_MachineBasicBlock:
 204:     MCOp = lowerSymbolOperand(MO, MO.getMBB()->getSymbol(), AP);
 205:     break;
 206:   case MachineOperand::MO_ExternalSymbol:
 207:     MCOp = lowerSymbolOperand(
 208:         MO, AP.GetExternalSymbolSymbol(MO.getSymbolName()), AP);
 209:     break;
 210:   case MachineOperand::MO_BlockAddress:
 211:     MCOp = lowerSymbolOperand(
 212:         MO, AP.GetBlockAddressSymbol(MO.getBlockAddress()), AP);
 213:     break;
 214:   case MachineOperand::MO_JumpTableIndex:
 215:     MCOp = lowerSymbolOperand(MO, AP.GetJTISymbol(MO.getIndex()), AP);
 216:     break;
```
- **EN**: This span continues the file's main responsibility: this file lowers MachineInstr or LLVM constructs to MCInst form for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   case MachineOperand::MO_MCSymbol:
 218:     MCOp = lowerSymbolOperand(MO, MO.getMCSymbol(), AP);
 219:     break;
 220:   }
 221:   return true;
 222: }
 223: 
 224: bool llvm::lowerLoongArchMachineInstrToMCInst(const MachineInstr *MI,
 225:                                               MCInst &OutMI, AsmPrinter &AP) {
 226:   OutMI.setOpcode(MI->getOpcode());
 227: 
 228:   for (const MachineOperand &MO : MI->operands()) {
 229:     MCOperand MCOp;
 230:     if (lowerLoongArchMachineOperandToMCOperand(MO, MCOp, AP))
 231:       OutMI.addOperand(MCOp);
 232:   }
 233:   return false;
 234: }
```
- **EN**: The range implements or declares functions including `llvm::lowerLoongArchMachineInstrToMCInst`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `llvm::lowerLoongArchMachineInstrToMCInst` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `LoongArch.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `MCTargetDesc/LoongArchMCAsmInfo.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCContext.h`
