# CSKYELFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYELFObjectWriter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements target-specific object writer logic and relocation record handling.
- 目的（中文）: 实现目标专用的目标文件写出逻辑与重定位记录处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYELFObjectWriter.cpp - CSKY ELF Writer -------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "CSKYFixupKinds.h"
  10: #include "CSKYMCAsmInfo.h"
  11: #include "CSKYMCTargetDesc.h"
  12: #include "MCTargetDesc/CSKYMCAsmInfo.h"
  13: #include "llvm/MC/MCContext.h"
  14: #include "llvm/MC/MCELFObjectWriter.h"
  15: #include "llvm/MC/MCObjectWriter.h"
  16: #include "llvm/MC/MCSymbolELF.h"
  17: 
  18: #define DEBUG_TYPE "csky-elf-object-writer"
  19: 
  20: using namespace llvm;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 21-40

```cpp
  21: 
  22: namespace {
  23: 
  24: class CSKYELFObjectWriter : public MCELFObjectTargetWriter {
  25: public:
  26:   CSKYELFObjectWriter(uint8_t OSABI = 0)
  27:       : MCELFObjectTargetWriter(false, OSABI, ELF::EM_CSKY, true){};
  28:   ~CSKYELFObjectWriter() {}
  29: 
  30:   unsigned getRelocType(const MCFixup &, const MCValue &,
  31:                         bool IsPCRel) const override;
  32:   bool needsRelocateWithSymbol(const MCValue &, unsigned Type) const override;
  33: };
  34: 
  35: } // namespace
  36: 
  37: unsigned CSKYELFObjectWriter::getRelocType(const MCFixup &Fixup,
  38:                                            const MCValue &Target,
  39:                                            bool IsPCRel) const {
  40:   const MCExpr *Expr = Fixup.getValue();
```

- EN: This chunk introduces interfaces or data structures such as CSKYELFObjectWriter, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CSKYELFObjectWriter, getRelocType contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 CSKYELFObjectWriter 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CSKYELFObjectWriter, getRelocType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 41-60

```cpp
  41:   // Determine the type of the relocation
  42:   auto Kind = Fixup.getKind();
  43:   uint8_t Modifier = Target.getSpecifier();
  44: 
  45:   switch (Target.getSpecifier()) {
  46:   case CSKY::S_TLSIE:
  47:   case CSKY::S_TLSLE:
  48:   case CSKY::S_TLSGD:
  49:   case CSKY::S_TLSLDM:
  50:   case CSKY::S_TLSLDO:
  51:     if (auto *SA = const_cast<MCSymbol *>(Target.getAddSym()))
  52:       static_cast<MCSymbolELF *>(SA)->setType(ELF::STT_TLS);
  53:     break;
  54:   default:
  55:     break;
  56:   }
  57: 
  58:   if (IsPCRel) {
  59:     switch (Kind) {
  60:     default:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-80

```cpp
  61:       LLVM_DEBUG(dbgs() << "Unknown Kind1  = " << Kind);
  62:       reportError(Fixup.getLoc(), "Unsupported relocation type");
  63:       return ELF::R_CKCORE_NONE;
  64:     case FK_Data_4:
  65:       return ELF::R_CKCORE_PCREL32;
  66:     case CSKY::fixup_csky_pcrel_uimm16_scale4:
  67:       return ELF::R_CKCORE_PCREL_IMM16_4;
  68:     case CSKY::fixup_csky_pcrel_uimm8_scale4:
  69:       return ELF::R_CKCORE_PCREL_IMM8_4;
  70:     case CSKY::fixup_csky_pcrel_imm26_scale2:
  71:       return ELF::R_CKCORE_PCREL_IMM26_2;
  72:     case CSKY::fixup_csky_pcrel_imm18_scale2:
  73:       return ELF::R_CKCORE_PCREL_IMM18_2;
  74:     case CSKY::fixup_csky_pcrel_imm16_scale2:
  75:       return ELF::R_CKCORE_PCREL_IMM16_2;
  76:     case CSKY::fixup_csky_pcrel_imm10_scale2:
  77:       return ELF::R_CKCORE_PCREL_IMM10_2;
  78:     case CSKY::fixup_csky_pcrel_uimm7_scale4:
  79:       return ELF::R_CKCORE_PCREL_IMM7_4;
  80:     }
```

- EN: This range continues the implementation of the backend component described by CSKYELFObjectWriter.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:   }
  82: 
  83:   switch (Kind) {
  84:   default:
  85:     LLVM_DEBUG(dbgs() << "Unknown Kind2  = " << Kind);
  86:     reportError(Fixup.getLoc(), "Unsupported relocation type");
  87:     return ELF::R_CKCORE_NONE;
  88:   case FK_Data_1:
  89:     reportError(Fixup.getLoc(), "1-byte data relocations not supported");
  90:     return ELF::R_CKCORE_NONE;
  91:   case FK_Data_2:
  92:     reportError(Fixup.getLoc(), "2-byte data relocations not supported");
  93:     return ELF::R_CKCORE_NONE;
  94:   case FK_Data_4:
  95:     if (Expr->getKind() == MCExpr::Specifier) {
  96:       auto TK = cast<MCSpecifierExpr>(Expr)->getSpecifier();
  97:       if (TK == CSKY::S_ADDR)
  98:         return ELF::R_CKCORE_ADDR32;
  99:       if (TK == CSKY::S_GOT)
 100:         return ELF::R_CKCORE_GOT32;
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:       if (TK == CSKY::S_GOTOFF)
 102:         return ELF::R_CKCORE_GOTOFF;
 103:       if (TK == CSKY::S_PLT)
 104:         return ELF::R_CKCORE_PLT32;
 105:       if (TK == CSKY::S_TLSIE)
 106:         return ELF::R_CKCORE_TLS_IE32;
 107:       if (TK == CSKY::S_TLSLE)
 108:         return ELF::R_CKCORE_TLS_LE32;
 109:       if (TK == CSKY::S_TLSGD)
 110:         return ELF::R_CKCORE_TLS_GD32;
 111:       if (TK == CSKY::S_TLSLDM)
 112:         return ELF::R_CKCORE_TLS_LDM32;
 113:       if (TK == CSKY::S_TLSLDO)
 114:         return ELF::R_CKCORE_TLS_LDO32;
 115:       if (TK == CSKY::S_GOTPC)
 116:         return ELF::R_CKCORE_GOTPC;
 117:       if (TK == CSKY::S_None)
 118:         return ELF::R_CKCORE_ADDR32;
 119: 
 120:       LLVM_DEBUG(dbgs() << "Unknown FK_Data_4 TK  = " << TK);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:       reportError(Fixup.getLoc(), "unknown target FK_Data_4");
 122:     } else {
 123:       switch (Modifier) {
 124:       default:
 125:         reportError(Fixup.getLoc(), "invalid fixup for 4-byte data relocation");
 126:         return ELF::R_CKCORE_NONE;
 127:       case CSKY::S_GOT:
 128:         return ELF::R_CKCORE_GOT32;
 129:       case CSKY::S_GOTOFF:
 130:         return ELF::R_CKCORE_GOTOFF;
 131:       case CSKY::S_PLT:
 132:         return ELF::R_CKCORE_PLT32;
 133:       case CSKY::S_TLSGD:
 134:         return ELF::R_CKCORE_TLS_GD32;
 135:       case CSKY::S_TLSLDM:
 136:         return ELF::R_CKCORE_TLS_LDM32;
 137:       case CSKY::S_TPOFF:
 138:         return ELF::R_CKCORE_TLS_LE32;
 139:       case CSKY::S_None:
 140:         return ELF::R_CKCORE_ADDR32;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 141-160

```cpp
 141:       }
 142:     }
 143:     return ELF::R_CKCORE_NONE;
 144:   case FK_Data_8:
 145:     reportError(Fixup.getLoc(), "8-byte data relocations not supported");
 146:     return ELF::R_CKCORE_NONE;
 147:   case CSKY::fixup_csky_addr32:
 148:     return ELF::R_CKCORE_ADDR32;
 149:   case CSKY::fixup_csky_addr_hi16:
 150:     return ELF::R_CKCORE_ADDR_HI16;
 151:   case CSKY::fixup_csky_addr_lo16:
 152:     return ELF::R_CKCORE_ADDR_LO16;
 153:   case CSKY::fixup_csky_doffset_imm18:
 154:     return ELF::R_CKCORE_DOFFSET_IMM18;
 155:   case CSKY::fixup_csky_doffset_imm18_scale2:
 156:     return ELF::R_CKCORE_DOFFSET_IMM18_2;
 157:   case CSKY::fixup_csky_doffset_imm18_scale4:
 158:     return ELF::R_CKCORE_DOFFSET_IMM18_4;
 159:   case CSKY::fixup_csky_got_imm18_scale4:
 160:     return ELF::R_CKCORE_GOT_IMM18_4;
```

- EN: This range continues the implementation of the backend component described by CSKYELFObjectWriter.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 161-179

```cpp
 161:   case CSKY::fixup_csky_plt_imm18_scale4:
 162:     return ELF::R_CKCORE_PLT_IMM18_4;
 163:   }
 164: }
 165: 
 166: bool CSKYELFObjectWriter::needsRelocateWithSymbol(const MCValue &V,
 167:                                                   unsigned Type) const {
 168:   switch (V.getSpecifier()) {
 169:   case CSKY::S_PLT:
 170:   case CSKY::S_GOT:
 171:     return true;
 172:   default:
 173:     return false;
 174:   }
 175: }
 176: 
 177: std::unique_ptr<MCObjectTargetWriter> llvm::createCSKYELFObjectWriter() {
 178:   return std::make_unique<CSKYELFObjectWriter>();
 179: }
```

- EN: Function bodies or method definitions such as needsRelocateWithSymbol, createCSKYELFObjectWriter contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: needsRelocateWithSymbol, createCSKYELFObjectWriter 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

## Key Concepts / 关键概念

- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYFixupKinds.h`, `CSKYMCAsmInfo.h`, `CSKYMCTargetDesc.h`, `MCTargetDesc/CSKYMCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSymbolELF.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
