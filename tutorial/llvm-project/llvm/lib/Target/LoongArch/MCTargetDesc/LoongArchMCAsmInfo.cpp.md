# LoongArchMCAsmInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchMCAsmInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- LoongArchMCAsmInfo.cpp - LoongArch Asm properties ------*- C++ -*--===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the declarations of the LoongArchMCAsmInfo properties.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchMCAsmInfo.h"
  14: #include "llvm/BinaryFormat/Dwarf.h"
  15: #include "llvm/BinaryFormat/ELF.h"
  16: #include "llvm/MC/MCContext.h"
  17: #include "llvm/MC/MCStreamer.h"
  18: #include "llvm/TargetParser/Triple.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchMCAsmInfo.h`, `Dwarf.h`, `ELF.h`, `MCContext.h`, `MCStreamer.h`, `Triple.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchMCAsmInfo.h`, `Dwarf.h`, `ELF.h`, `MCContext.h`, `MCStreamer.h`, `Triple.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: using namespace llvm;
  21: 
  22: const LoongArchMCExpr *LoongArchMCExpr::create(const MCExpr *Expr, uint16_t S,
  23:                                                MCContext &Ctx, bool Hint) {
  24:   return new (Ctx) LoongArchMCExpr(Expr, S, Hint);
  25: }
  26: 
  27: static StringRef getLoongArchSpecifierName(uint16_t S) {
  28:   switch (S) {
  29:   default:
  30:     llvm_unreachable("Invalid ELF symbol kind");
  31:   case ELF::R_LARCH_B16:
  32:     return "b16";
  33:   case ELF::R_LARCH_B21:
  34:     return "b21";
  35:   case ELF::R_LARCH_MARK_LA:
  36:   case ELF::R_LARCH_ABS_HI20:
```
- **EN**: The range implements or declares functions including `getLoongArchSpecifierName`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getLoongArchSpecifierName` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     return "abs_hi20";
  38:   case ELF::R_LARCH_ABS_LO12:
  39:     return "abs_lo12";
  40:   case ELF::R_LARCH_ABS64_LO20:
  41:     return "abs64_lo20";
  42:   case ELF::R_LARCH_ABS64_HI12:
  43:     return "abs64_hi12";
  44:   case ELF::R_LARCH_PCALA_HI20:
  45:     return "pc_hi20";
  46:   case ELF::R_LARCH_PCALA_LO12:
  47:     return "pc_lo12";
  48:   case ELF::R_LARCH_PCALA64_LO20:
  49:     return "pc64_lo20";
  50:   case ELF::R_LARCH_PCALA64_HI12:
  51:     return "pc64_hi12";
  52:   case ELF::R_LARCH_GOT_PC_HI20:
  53:     return "got_pc_hi20";
  54:   case ELF::R_LARCH_GOT_PC_LO12:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:     return "got_pc_lo12";
  56:   case ELF::R_LARCH_GOT64_PC_LO20:
  57:     return "got64_pc_lo20";
  58:   case ELF::R_LARCH_GOT64_PC_HI12:
  59:     return "got64_pc_hi12";
  60:   case ELF::R_LARCH_GOT_HI20:
  61:     return "got_hi20";
  62:   case ELF::R_LARCH_GOT_LO12:
  63:     return "got_lo12";
  64:   case ELF::R_LARCH_GOT64_LO20:
  65:     return "got64_lo20";
  66:   case ELF::R_LARCH_GOT64_HI12:
  67:     return "got64_hi12";
  68:   case ELF::R_LARCH_TLS_LE_HI20:
  69:     return "le_hi20";
  70:   case ELF::R_LARCH_TLS_LE_LO12:
  71:     return "le_lo12";
  72:   case ELF::R_LARCH_TLS_LE64_LO20:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     return "le64_lo20";
  74:   case ELF::R_LARCH_TLS_LE64_HI12:
  75:     return "le64_hi12";
  76:   case ELF::R_LARCH_TLS_IE_PC_HI20:
  77:     return "ie_pc_hi20";
  78:   case ELF::R_LARCH_TLS_IE_PC_LO12:
  79:     return "ie_pc_lo12";
  80:   case ELF::R_LARCH_TLS_IE64_PC_LO20:
  81:     return "ie64_pc_lo20";
  82:   case ELF::R_LARCH_TLS_IE64_PC_HI12:
  83:     return "ie64_pc_hi12";
  84:   case ELF::R_LARCH_TLS_IE_HI20:
  85:     return "ie_hi20";
  86:   case ELF::R_LARCH_TLS_IE_LO12:
  87:     return "ie_lo12";
  88:   case ELF::R_LARCH_TLS_IE64_LO20:
  89:     return "ie64_lo20";
  90:   case ELF::R_LARCH_TLS_IE64_HI12:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     return "ie64_hi12";
  92:   case ELF::R_LARCH_TLS_LD_PC_HI20:
  93:     return "ld_pc_hi20";
  94:   case ELF::R_LARCH_TLS_LD_HI20:
  95:     return "ld_hi20";
  96:   case ELF::R_LARCH_TLS_GD_PC_HI20:
  97:     return "gd_pc_hi20";
  98:   case ELF::R_LARCH_TLS_GD_HI20:
  99:     return "gd_hi20";
 100:   case ELF::R_LARCH_CALL30:
 101:     return "call30";
 102:   case ELF::R_LARCH_CALL36:
 103:     return "call36";
 104:   case ELF::R_LARCH_TLS_DESC_PC_HI20:
 105:     return "desc_pc_hi20";
 106:   case ELF::R_LARCH_TLS_DESC_PC_LO12:
 107:     return "desc_pc_lo12";
 108:   case ELF::R_LARCH_TLS_DESC64_PC_LO20:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     return "desc64_pc_lo20";
 110:   case ELF::R_LARCH_TLS_DESC64_PC_HI12:
 111:     return "desc64_pc_hi12";
 112:   case ELF::R_LARCH_TLS_DESC_HI20:
 113:     return "desc_hi20";
 114:   case ELF::R_LARCH_TLS_DESC_LO12:
 115:     return "desc_lo12";
 116:   case ELF::R_LARCH_TLS_DESC64_LO20:
 117:     return "desc64_lo20";
 118:   case ELF::R_LARCH_TLS_DESC64_HI12:
 119:     return "desc64_hi12";
 120:   case ELF::R_LARCH_TLS_DESC_LD:
 121:     return "desc_ld";
 122:   case ELF::R_LARCH_TLS_DESC_CALL:
 123:     return "desc_call";
 124:   case ELF::R_LARCH_TLS_LE_HI20_R:
 125:     return "le_hi20_r";
 126:   case ELF::R_LARCH_TLS_LE_ADD_R:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     return "le_add_r";
 128:   case ELF::R_LARCH_TLS_LE_LO12_R:
 129:     return "le_lo12_r";
 130:   case ELF::R_LARCH_PCREL20_S2:
 131:     return "pcrel_20";
 132:   case ELF::R_LARCH_TLS_LD_PCREL20_S2:
 133:     return "ld_pcrel_20";
 134:   case ELF::R_LARCH_TLS_GD_PCREL20_S2:
 135:     return "gd_pcrel_20";
 136:   case ELF::R_LARCH_TLS_DESC_PCREL20_S2:
 137:     return "desc_pcrel_20";
 138:   case ELF::R_LARCH_PCADD_HI20:
 139:     return "pcadd_hi20";
 140:   case ELF::R_LARCH_PCADD_LO12:
 141:     return "pcadd_lo12";
 142:   case ELF::R_LARCH_GOT_PCADD_HI20:
 143:     return "got_pcadd_hi20";
 144:   case ELF::R_LARCH_GOT_PCADD_LO12:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     return "got_pcadd_lo12";
 146:   case ELF::R_LARCH_TLS_IE_PCADD_HI20:
 147:     return "ie_pcadd_hi20";
 148:   case ELF::R_LARCH_TLS_IE_PCADD_LO12:
 149:     return "ie_pcadd_lo12";
 150:   case ELF::R_LARCH_TLS_LD_PCADD_HI20:
 151:     return "ld_pcadd_hi20";
 152:   case ELF::R_LARCH_TLS_LD_PCADD_LO12:
 153:     return "ld_pcadd_lo12";
 154:   case ELF::R_LARCH_TLS_GD_PCADD_HI20:
 155:     return "gd_pcadd_hi20";
 156:   case ELF::R_LARCH_TLS_GD_PCADD_LO12:
 157:     return "gd_pcadd_lo12";
 158:   case ELF::R_LARCH_TLS_DESC_PCADD_HI20:
 159:     return "desc_pcadd_hi20";
 160:   case ELF::R_LARCH_TLS_DESC_PCADD_LO12:
 161:     return "desc_pcadd_lo12";
 162:   }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163: }
 164: 
 165: LoongArchMCExpr::Specifier LoongArch::parseSpecifier(StringRef name) {
 166:   return StringSwitch<LoongArchMCExpr::Specifier>(name)
 167:       .Case("plt", ELF::R_LARCH_B26)
 168:       .Case("b16", ELF::R_LARCH_B16)
 169:       .Case("b21", ELF::R_LARCH_B21)
 170:       .Case("b26", ELF::R_LARCH_B26)
 171:       .Case("abs_hi20", ELF::R_LARCH_ABS_HI20)
 172:       .Case("abs_lo12", ELF::R_LARCH_ABS_LO12)
 173:       .Case("abs64_lo20", ELF::R_LARCH_ABS64_LO20)
 174:       .Case("abs64_hi12", ELF::R_LARCH_ABS64_HI12)
 175:       .Case("pc_hi20", ELF::R_LARCH_PCALA_HI20)
 176:       .Case("pc_lo12", ELF::R_LARCH_PCALA_LO12)
 177:       .Case("pc64_lo20", ELF::R_LARCH_PCALA64_LO20)
 178:       .Case("pc64_hi12", ELF::R_LARCH_PCALA64_HI12)
 179:       .Case("got_pc_hi20", ELF::R_LARCH_GOT_PC_HI20)
 180:       .Case("got_pc_lo12", ELF::R_LARCH_GOT_PC_LO12)
```
- **EN**: The range implements or declares functions including `LoongArch::parseSpecifier`.
- **CN**: 这一段实现或声明了 `LoongArch::parseSpecifier` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:       .Case("got64_pc_lo20", ELF::R_LARCH_GOT64_PC_LO20)
 182:       .Case("got64_pc_hi12", ELF::R_LARCH_GOT64_PC_HI12)
 183:       .Case("got_hi20", ELF::R_LARCH_GOT_HI20)
 184:       .Case("got_lo12", ELF::R_LARCH_GOT_LO12)
 185:       .Case("got64_lo20", ELF::R_LARCH_GOT64_LO20)
 186:       .Case("got64_hi12", ELF::R_LARCH_GOT64_HI12)
 187:       .Case("le_hi20", ELF::R_LARCH_TLS_LE_HI20)
 188:       .Case("le_lo12", ELF::R_LARCH_TLS_LE_LO12)
 189:       .Case("le64_lo20", ELF::R_LARCH_TLS_LE64_LO20)
 190:       .Case("le64_hi12", ELF::R_LARCH_TLS_LE64_HI12)
 191:       .Case("ie_pc_hi20", ELF::R_LARCH_TLS_IE_PC_HI20)
 192:       .Case("ie_pc_lo12", ELF::R_LARCH_TLS_IE_PC_LO12)
 193:       .Case("ie64_pc_lo20", ELF::R_LARCH_TLS_IE64_PC_LO20)
 194:       .Case("ie64_pc_hi12", ELF::R_LARCH_TLS_IE64_PC_HI12)
 195:       .Case("ie_hi20", ELF::R_LARCH_TLS_IE_HI20)
 196:       .Case("ie_lo12", ELF::R_LARCH_TLS_IE_LO12)
 197:       .Case("ie64_lo20", ELF::R_LARCH_TLS_IE64_LO20)
 198:       .Case("ie64_hi12", ELF::R_LARCH_TLS_IE64_HI12)
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```cpp
 199:       .Case("ld_pc_hi20", ELF::R_LARCH_TLS_LD_PC_HI20)
 200:       .Case("ld_hi20", ELF::R_LARCH_TLS_LD_HI20)
 201:       .Case("gd_pc_hi20", ELF::R_LARCH_TLS_GD_PC_HI20)
 202:       .Case("gd_hi20", ELF::R_LARCH_TLS_GD_HI20)
 203:       .Case("call30", ELF::R_LARCH_CALL30)
 204:       .Case("call36", ELF::R_LARCH_CALL36)
 205:       .Case("desc_pc_hi20", ELF::R_LARCH_TLS_DESC_PC_HI20)
 206:       .Case("desc_pc_lo12", ELF::R_LARCH_TLS_DESC_PC_LO12)
 207:       .Case("desc64_pc_lo20", ELF::R_LARCH_TLS_DESC64_PC_LO20)
 208:       .Case("desc64_pc_hi12", ELF::R_LARCH_TLS_DESC64_PC_HI12)
 209:       .Case("desc_hi20", ELF::R_LARCH_TLS_DESC_HI20)
 210:       .Case("desc_lo12", ELF::R_LARCH_TLS_DESC_LO12)
 211:       .Case("desc64_lo20", ELF::R_LARCH_TLS_DESC64_LO20)
 212:       .Case("desc64_hi12", ELF::R_LARCH_TLS_DESC64_HI12)
 213:       .Case("desc_ld", ELF::R_LARCH_TLS_DESC_LD)
 214:       .Case("desc_call", ELF::R_LARCH_TLS_DESC_CALL)
 215:       .Case("le_hi20_r", ELF::R_LARCH_TLS_LE_HI20_R)
 216:       .Case("le_add_r", ELF::R_LARCH_TLS_LE_ADD_R)
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-234 / 第 217-234 行
```cpp
 217:       .Case("le_lo12_r", ELF::R_LARCH_TLS_LE_LO12_R)
 218:       .Case("pcrel_20", ELF::R_LARCH_PCREL20_S2)
 219:       .Case("ld_pcrel_20", ELF::R_LARCH_TLS_LD_PCREL20_S2)
 220:       .Case("gd_pcrel_20", ELF::R_LARCH_TLS_GD_PCREL20_S2)
 221:       .Case("desc_pcrel_20", ELF::R_LARCH_TLS_DESC_PCREL20_S2)
 222:       .Case("pcadd_hi20", ELF::R_LARCH_PCADD_HI20)
 223:       .Case("pcadd_lo12", ELF::R_LARCH_PCADD_LO12)
 224:       .Case("got_pcadd_hi20", ELF::R_LARCH_GOT_PCADD_HI20)
 225:       .Case("got_pcadd_lo12", ELF::R_LARCH_GOT_PCADD_LO12)
 226:       .Case("ie_pcadd_hi20", ELF::R_LARCH_TLS_IE_PCADD_HI20)
 227:       .Case("ie_pcadd_lo12", ELF::R_LARCH_TLS_IE_PCADD_LO12)
 228:       .Case("ld_pcadd_hi20", ELF::R_LARCH_TLS_LD_PCADD_HI20)
 229:       .Case("ld_pcadd_lo12", ELF::R_LARCH_TLS_LD_PCADD_LO12)
 230:       .Case("gd_pcadd_hi20", ELF::R_LARCH_TLS_GD_PCADD_HI20)
 231:       .Case("gd_pcadd_lo12", ELF::R_LARCH_TLS_GD_PCADD_LO12)
 232:       .Case("desc_pcadd_hi20", ELF::R_LARCH_TLS_DESC_PCADD_HI20)
 233:       .Case("desc_pcadd_lo12", ELF::R_LARCH_TLS_DESC_PCADD_LO12)
 234:       .Default(0);
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```cpp
 235: }
 236: 
 237: void LoongArchMCAsmInfo::anchor() {}
 238: 
 239: LoongArchMCAsmInfo::LoongArchMCAsmInfo(const Triple &TT,
 240:                                        const MCTargetOptions &Options)
 241:     : MCAsmInfoELF(Options) {
 242:   CodePointerSize = CalleeSaveStackSlotSize = TT.isArch64Bit() ? 8 : 4;
 243:   AlignmentIsInBytes = false;
 244:   Data8bitsDirective = "\t.byte\t";
 245:   Data16bitsDirective = "\t.half\t";
 246:   Data32bitsDirective = "\t.word\t";
 247:   Data64bitsDirective = "\t.dword\t";
 248:   ZeroDirective = "\t.space\t";
 249:   CommentString = "#";
 250:   SupportsDebugInformation = true;
 251:   DwarfRegNumForCFI = true;
 252:   ExceptionsType = ExceptionHandling::DwarfCFI;
```
- **EN**: The range implements or declares functions including `LoongArchMCAsmInfo::anchor`.
- **CN**: 这一段实现或声明了 `LoongArchMCAsmInfo::anchor` 等函数。

### Lines 253-264 / 第 253-264 行
```cpp
 253: }
 254: 
 255: void LoongArchMCAsmInfo::printSpecifierExpr(raw_ostream &OS,
 256:                                             const MCSpecifierExpr &Expr) const {
 257:   auto S = Expr.getSpecifier();
 258:   bool HasSpecifier = S != 0 && S != ELF::R_LARCH_B26;
 259:   if (HasSpecifier)
 260:     OS << '%' << getLoongArchSpecifierName(S) << '(';
 261:   printExpr(OS, *Expr.getSubExpr());
 262:   if (HasSpecifier)
 263:     OS << ')';
 264: }
```
- **EN**: The range implements or declares functions including `LoongArchMCAsmInfo::printSpecifierExpr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchMCAsmInfo::printSpecifierExpr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。

## Dependencies / 依赖关系
- `LoongArchMCAsmInfo.h`
- `llvm/BinaryFormat/Dwarf.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCStreamer.h`
- `llvm/TargetParser/Triple.h`
