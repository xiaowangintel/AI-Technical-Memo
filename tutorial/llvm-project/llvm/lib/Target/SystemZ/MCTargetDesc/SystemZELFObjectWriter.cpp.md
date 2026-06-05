# SystemZELFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZELFObjectWriter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZELFObjectWriter.cpp - SystemZ ELF writer -------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/SystemZMCAsmInfo.h"
  10: #include "MCTargetDesc/SystemZMCFixups.h"
  11: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  12: #include "llvm/BinaryFormat/ELF.h"
  13: #include "llvm/MC/MCContext.h"
  14: #include "llvm/MC/MCELFObjectWriter.h"
  15: #include "llvm/MC/MCExpr.h"
  16: #include "llvm/MC/MCFixup.h"
  17: #include "llvm/MC/MCObjectWriter.h"
  18: #include "llvm/MC/MCValue.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCAsmInfo.h`, `SystemZMCFixups.h`, `SystemZMCTargetDesc.h`, `ELF.h`, `MCContext.h`, `MCELFObjectWriter.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCAsmInfo.h`, `SystemZMCFixups.h`, `SystemZMCTargetDesc.h`, `ELF.h`, `MCContext.h`, `MCELFObjectWriter.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/Support/ErrorHandling.h"
  20: #include <cassert>
  21: #include <cstdint>
  22: #include <memory>
  23: 
  24: using namespace llvm;
  25: 
  26: namespace {
  27: 
  28: class SystemZELFObjectWriter : public MCELFObjectTargetWriter {
  29: public:
  30:   SystemZELFObjectWriter(uint8_t OSABI);
  31:   ~SystemZELFObjectWriter() override = default;
  32: 
  33: protected:
  34:   // Override MCELFObjectTargetWriter.
  35:   unsigned getRelocType(const MCFixup &, const MCValue &,
  36:                         bool IsPCRel) const override;
```
- **EN**: It imports dependencies such as `ErrorHandling.h`, `cassert`, `cstdint`, `memory` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZELFObjectWriter`.
- **CN**: 它引入了 `ErrorHandling.h`, `cassert`, `cstdint`, `memory` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZELFObjectWriter` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   bool needsRelocateWithSymbol(const MCValue &, unsigned Type) const override;
  38:   unsigned getAbsoluteReloc(SMLoc Loc, unsigned Kind) const;
  39:   unsigned getPCRelReloc(SMLoc Loc, unsigned Kind) const;
  40: };
  41: 
  42: } // end anonymous namespace
  43: 
  44: SystemZELFObjectWriter::SystemZELFObjectWriter(uint8_t OSABI)
  45:     : MCELFObjectTargetWriter(/*Is64Bit_=*/true, OSABI, ELF::EM_S390,
  46:                               /*HasRelocationAddend_=*/true) {}
  47: 
  48: // Return the relocation type for an absolute value of MCFixupKind Kind.
  49: unsigned SystemZELFObjectWriter::getAbsoluteReloc(SMLoc Loc,
  50:                                                   unsigned Kind) const {
  51:   switch (Kind) {
  52:   case FK_Data_1:
  53:   case SystemZ::FK_390_U8Imm:
  54:   case SystemZ::FK_390_S8Imm:
```
- **EN**: The range implements or declares functions including `MCELFObjectTargetWriter`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `MCELFObjectTargetWriter` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 55-72 / 第 55-72 行
```cpp
  55:     return ELF::R_390_8;
  56:   case SystemZ::FK_390_U12Imm:
  57:     return ELF::R_390_12;
  58:   case FK_Data_2:
  59:   case SystemZ::FK_390_U16Imm:
  60:   case SystemZ::FK_390_S16Imm:
  61:     return ELF::R_390_16;
  62:   case SystemZ::FK_390_S20Imm:
  63:     return ELF::R_390_20;
  64:   case FK_Data_4:
  65:   case SystemZ::FK_390_U32Imm:
  66:   case SystemZ::FK_390_S32Imm:
  67:     return ELF::R_390_32;
  68:   case FK_Data_8:
  69:     return ELF::R_390_64;
  70:   }
  71:   reportError(Loc, "Unsupported absolute address");
  72:   return 0;
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: }
  74: 
  75: // Return the relocation type for a PC-relative value of MCFixupKind Kind.
  76: unsigned SystemZELFObjectWriter::getPCRelReloc(SMLoc Loc, unsigned Kind) const {
  77:   switch (Kind) {
  78:   case FK_Data_2:
  79:   case SystemZ::FK_390_U16Imm:
  80:   case SystemZ::FK_390_S16Imm:
  81:     return ELF::R_390_PC16;
  82:   case FK_Data_4:
  83:   case SystemZ::FK_390_U32Imm:
  84:   case SystemZ::FK_390_S32Imm:
  85:     return ELF::R_390_PC32;
  86:   case FK_Data_8:
  87:     return ELF::R_390_PC64;
  88:   case SystemZ::FK_390_PC12DBL:
  89:     return ELF::R_390_PC12DBL;
  90:   case SystemZ::FK_390_PC16DBL:
```
- **EN**: The range implements or declares functions including `SystemZELFObjectWriter::getPCRelReloc`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZELFObjectWriter::getPCRelReloc` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     return ELF::R_390_PC16DBL;
  92:   case SystemZ::FK_390_PC24DBL:
  93:     return ELF::R_390_PC24DBL;
  94:   case SystemZ::FK_390_PC32DBL:
  95:     return ELF::R_390_PC32DBL;
  96:   }
  97:   reportError(Loc, "Unsupported PC-relative address");
  98:   return 0;
  99: }
 100: 
 101: unsigned SystemZELFObjectWriter::getRelocType(const MCFixup &Fixup,
 102:                                               const MCValue &Target,
 103:                                               bool IsPCRel) const {
 104:   SMLoc Loc = Fixup.getLoc();
 105:   unsigned Kind = Fixup.getKind();
 106:   auto Specifier = SystemZ::Specifier(Target.getSpecifier());
 107:   switch (Specifier) {
 108:   case SystemZ::S_INDNTPOFF:
```
- **EN**: The range implements or declares functions including `SystemZELFObjectWriter::getRelocType`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZELFObjectWriter::getRelocType` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   case SystemZ::S_NTPOFF:
 110:   case SystemZ::S_TLSGD:
 111:   case SystemZ::S_TLSLD:
 112:   case SystemZ::S_TLSLDM:
 113:   case SystemZ::S_DTPOFF:
 114:     if (auto *SA = const_cast<MCSymbol *>(Target.getAddSym()))
 115:       static_cast<MCSymbolELF *>(SA)->setType(ELF::STT_TLS);
 116:     break;
 117:   default:
 118:     break;
 119:   }
 120: 
 121:   switch (Specifier) {
 122:   case SystemZ::S_None:
 123:     if (IsPCRel)
 124:       return getPCRelReloc(Loc, Kind);
 125:     return getAbsoluteReloc(Loc, Kind);
 126: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   case SystemZ::S_NTPOFF:
 128:     assert(!IsPCRel && "NTPOFF shouldn't be PC-relative");
 129:     switch (Kind) {
 130:     case FK_Data_4:
 131:       return ELF::R_390_TLS_LE32;
 132:     case FK_Data_8:
 133:       return ELF::R_390_TLS_LE64;
 134:     }
 135:     reportError(Loc, "Unsupported thread-local address (local-exec)");
 136:     return 0;
 137: 
 138:   case SystemZ::S_INDNTPOFF:
 139:     if (IsPCRel && Kind == SystemZ::FK_390_PC32DBL)
 140:       return ELF::R_390_TLS_IEENT;
 141:     reportError(Loc,
 142:                 "Only PC-relative INDNTPOFF accesses are supported for now");
 143:     return 0;
 144: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   case SystemZ::S_DTPOFF:
 146:     assert(!IsPCRel && "DTPOFF shouldn't be PC-relative");
 147:     switch (Kind) {
 148:     case FK_Data_4:
 149:       return ELF::R_390_TLS_LDO32;
 150:     case FK_Data_8:
 151:       return ELF::R_390_TLS_LDO64;
 152:     }
 153:     reportError(Loc, "Unsupported thread-local address (local-dynamic)");
 154:     return 0;
 155: 
 156:   case SystemZ::S_TLSLDM:
 157:     assert(!IsPCRel && "TLSLDM shouldn't be PC-relative");
 158:     switch (Kind) {
 159:     case FK_Data_4:
 160:       return ELF::R_390_TLS_LDM32;
 161:     case FK_Data_8:
 162:       return ELF::R_390_TLS_LDM64;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     case SystemZ::FK_390_TLS_CALL:
 164:       return ELF::R_390_TLS_LDCALL;
 165:     }
 166:     reportError(Loc, "Unsupported thread-local address (local-dynamic)");
 167:     return 0;
 168: 
 169:   case SystemZ::S_TLSGD:
 170:     assert(!IsPCRel && "TLSGD shouldn't be PC-relative");
 171:     switch (Kind) {
 172:     case FK_Data_4:
 173:       return ELF::R_390_TLS_GD32;
 174:     case FK_Data_8:
 175:       return ELF::R_390_TLS_GD64;
 176:     case SystemZ::FK_390_TLS_CALL:
 177:       return ELF::R_390_TLS_GDCALL;
 178:     }
 179:     reportError(Loc, "Unsupported thread-local address (general-dynamic)");
 180:     return 0;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 181-198 / 第 181-198 行
```cpp
 181: 
 182:   case SystemZ::S_GOT:
 183:   case SystemZ::S_GOTENT:
 184:     if (IsPCRel && Kind == SystemZ::FK_390_PC32DBL)
 185:       return ELF::R_390_GOTENT;
 186:     reportError(Loc, "Only PC-relative GOT accesses are supported for now");
 187:     return 0;
 188: 
 189:   case SystemZ::S_PLT:
 190:     assert(IsPCRel && "@PLT shouldn't be PC-relative");
 191:     switch (Kind) {
 192:     case SystemZ::FK_390_PC12DBL:
 193:       return ELF::R_390_PLT12DBL;
 194:     case SystemZ::FK_390_PC16DBL:
 195:       return ELF::R_390_PLT16DBL;
 196:     case SystemZ::FK_390_PC24DBL:
 197:       return ELF::R_390_PLT24DBL;
 198:     case SystemZ::FK_390_PC32DBL:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 199-216 / 第 199-216 行
```cpp
 199:       return ELF::R_390_PLT32DBL;
 200:     }
 201:     reportError(Loc, "Unsupported PC-relative PLT address");
 202:     return 0;
 203: 
 204:   default:
 205:     llvm_unreachable("Modifier not supported");
 206:   }
 207: }
 208: 
 209: bool SystemZELFObjectWriter::needsRelocateWithSymbol(const MCValue &V,
 210:                                                      unsigned Type) const {
 211:   switch (V.getSpecifier()) {
 212:   case SystemZ::S_GOT:
 213:   case SystemZ::S_PLT:
 214:     return true;
 215:   default:
 216:     return false;
```
- **EN**: The range implements or declares functions including `SystemZELFObjectWriter::needsRelocateWithSymbol`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZELFObjectWriter::needsRelocateWithSymbol` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 217-223 / 第 217-223 行
```cpp
 217:   }
 218: }
 219: 
 220: std::unique_ptr<MCObjectTargetWriter>
 221: llvm::createSystemZELFObjectWriter(uint8_t OSABI) {
 222:   return std::make_unique<SystemZELFObjectWriter>(OSABI);
 223: }
```
- **EN**: The range implements or declares functions including `llvm::createSystemZELFObjectWriter`.
- **CN**: 这一段实现或声明了 `llvm::createSystemZELFObjectWriter` 等函数。

## Key Concepts / 关键概念
- **Target backend structure**: Shows how LLVM splits a backend into MC, CodeGen, and target-specific layers. / 展示 LLVM 如何将后端拆分为 MC、CodeGen 和目标相关层。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZMCAsmInfo.h`
- `MCTargetDesc/SystemZMCFixups.h`
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCELFObjectWriter.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCFixup.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCValue.h`
- `llvm/Support/ErrorHandling.h`
- `cassert`
- `cstdint`
- `memory`
