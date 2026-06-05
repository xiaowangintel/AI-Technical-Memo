# HexagonELFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonELFObjectWriter.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon ELF relocation and object-writer behavior.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonELFObjectWriter.cpp - Hexagon Target Descriptions ----------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "MCTargetDesc/HexagonFixupKinds.h"
    10: #include "MCTargetDesc/HexagonMCExpr.h"
    11: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    12: #include "llvm/MC/MCAssembler.h"
    13: #include "llvm/MC/MCELFObjectWriter.h"
    14: #include "llvm/MC/MCObjectWriter.h"
    15: #include "llvm/MC/MCValue.h"
    16: 
    17: #define DEBUG_TYPE "hexagon-elf-writer"
    18: 
    19: using namespace llvm;
    20: using namespace Hexagon;
    21: 
    22: namespace {
    23: 
    24: class HexagonELFObjectWriter : public MCELFObjectTargetWriter {
    25: private:
    26:   StringRef CPU;
    27: 
    28: public:
    29:   HexagonELFObjectWriter(uint8_t OSABI, StringRef C);
    30: 
    31:   unsigned getRelocType(const MCFixup &, const MCValue &,
    32:                         bool IsPCRel) const override;
    33: };
    34: }
    35: 
    36: HexagonELFObjectWriter::HexagonELFObjectWriter(uint8_t OSABI, StringRef C)
    37:     : MCELFObjectTargetWriter(/*Is64bit*/ false, OSABI, ELF::EM_HEXAGON,
    38:                               /*HasRelocationAddend*/ true),
    39:       CPU(C) {}
    40: 
    41: unsigned HexagonELFObjectWriter::getRelocType(const MCFixup &Fixup,
    42:                                               const MCValue &Target,
    43:                                               bool IsPCRel) const {
    44:   auto Variant = HexagonMCExpr::VariantKind(Target.getSpecifier());
    45:   switch (Variant) {
    46:   case HexagonMCExpr::VK_GD_GOT:
    47:   case HexagonMCExpr::VK_LD_GOT:
    48:   case HexagonMCExpr::VK_GD_PLT:
    49:   case HexagonMCExpr::VK_LD_PLT:
    50:   case HexagonMCExpr::VK_IE:
```
- EN: It imports headers such as MCTargetDesc/HexagonFixupKinds.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/MC/MCAssembler.h, ... (7 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonELFObjectWriter, which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonFixupKinds.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/MC/MCAssembler.h, ... (7 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonELFObjectWriter 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   case HexagonMCExpr::VK_IE_GOT:
    52:   case HexagonMCExpr::VK_TPREL:
    53:     if (auto *SA = const_cast<MCSymbol *>(Target.getAddSym()))
    54:       static_cast<MCSymbolELF *>(SA)->setType(ELF::STT_TLS);
    55:     break;
    56:   default:
    57:     break;
    58:   }
    59:   switch (Fixup.getKind()) {
    60:   default:
    61:     report_fatal_error("Unrecognized relocation type, fixup kind=" +
    62:                        Twine(Fixup.getKind()));
    63:     break;
    64:   case FK_Data_8:
    65:     // Hexagon is a 32-bit target with no native 64-bit relocation.
    66:     // Handle 8-byte data fixups as 32-bit relocations -- on a
    67:     // little-endian 32-bit platform, addresses occupy the low 4 bytes
    68:     // and the high 4 bytes are zero.
    69:     switch (Variant) {
    70:     case HexagonMCExpr::VK_None:
    71:       return IsPCRel ? ELF::R_HEX_32_PCREL : ELF::R_HEX_32;
    72:     default:
    73:       report_fatal_error("Unrecognized variant type for FK_Data_8");
    74:     };
    75:   case FK_Data_4:
    76:     switch (Variant) {
    77:     case HexagonMCExpr::VK_DTPREL:
    78:       return ELF::R_HEX_DTPREL_32;
    79:     case HexagonMCExpr::VK_GOT:
    80:       return ELF::R_HEX_GOT_32;
    81:     case HexagonMCExpr::VK_GOTREL:
    82:       return ELF::R_HEX_GOTREL_32;
    83:     case HexagonMCExpr::VK_GD_GOT:
    84:       return ELF::R_HEX_GD_GOT_32;
    85:     case HexagonMCExpr::VK_IE:
    86:       return ELF::R_HEX_IE_32;
    87:     case HexagonMCExpr::VK_IE_GOT:
    88:       return ELF::R_HEX_IE_GOT_32;
    89:     case HexagonMCExpr::VK_LD_GOT:
    90:       return ELF::R_HEX_LD_GOT_32;
    91:     case HexagonMCExpr::VK_PCREL:
    92:       return ELF::R_HEX_32_PCREL;
    93:     case HexagonMCExpr::VK_TPREL:
    94:       return ELF::R_HEX_TPREL_32;
    95:     case HexagonMCExpr::VK_None:
    96:       return IsPCRel ? ELF::R_HEX_32_PCREL : ELF::R_HEX_32;
    97:     default:
    98:       report_fatal_error("Unrecognized variant type");
    99:     };
   100:   case FK_Data_2:
```
- EN: It declares or implements routines such as report_fatal_error, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 report_fatal_error 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:     switch(Variant) {
   102:     case HexagonMCExpr::VK_DTPREL:
   103:       return ELF::R_HEX_DTPREL_16;
   104:     case HexagonMCExpr::VK_GOT:
   105:       return ELF::R_HEX_GOT_16;
   106:     case HexagonMCExpr::VK_GD_GOT:
   107:       return ELF::R_HEX_GD_GOT_16;
   108:     case HexagonMCExpr::VK_IE_GOT:
   109:       return ELF::R_HEX_IE_GOT_16;
   110:     case HexagonMCExpr::VK_LD_GOT:
   111:       return ELF::R_HEX_LD_GOT_16;
   112:     case HexagonMCExpr::VK_TPREL:
   113:       return ELF::R_HEX_TPREL_16;
   114:     case HexagonMCExpr::VK_None:
   115:       return ELF::R_HEX_16;
   116:     default:
   117:       report_fatal_error("Unrecognized variant type");
   118:     };
   119:   case FK_Data_1:
   120:     return ELF::R_HEX_8;
   121:   case fixup_Hexagon_B22_PCREL:
   122:     return ELF::R_HEX_B22_PCREL;
   123:   case fixup_Hexagon_B15_PCREL:
   124:     return ELF::R_HEX_B15_PCREL;
   125:   case fixup_Hexagon_B7_PCREL:
   126:     return ELF::R_HEX_B7_PCREL;
   127:   case fixup_Hexagon_LO16:
   128:     return ELF::R_HEX_LO16;
   129:   case fixup_Hexagon_HI16:
   130:     return ELF::R_HEX_HI16;
   131:   case fixup_Hexagon_32:
   132:     return ELF::R_HEX_32;
   133:   case fixup_Hexagon_16:
   134:     return ELF::R_HEX_16;
   135:   case fixup_Hexagon_8:
   136:     return ELF::R_HEX_8;
   137:   case fixup_Hexagon_GPREL16_0:
   138:     return ELF::R_HEX_GPREL16_0;
   139:   case fixup_Hexagon_GPREL16_1:
   140:     return ELF::R_HEX_GPREL16_1;
   141:   case fixup_Hexagon_GPREL16_2:
   142:     return ELF::R_HEX_GPREL16_2;
   143:   case fixup_Hexagon_GPREL16_3:
   144:     return ELF::R_HEX_GPREL16_3;
   145:   case fixup_Hexagon_HL16:
   146:     return ELF::R_HEX_HL16;
   147:   case fixup_Hexagon_B13_PCREL:
   148:     return ELF::R_HEX_B13_PCREL;
   149:   case fixup_Hexagon_B9_PCREL:
   150:     return ELF::R_HEX_B9_PCREL;
```
- EN: It declares or implements routines such as report_fatal_error, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 report_fatal_error 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   case fixup_Hexagon_B32_PCREL_X:
   152:     return ELF::R_HEX_B32_PCREL_X;
   153:   case fixup_Hexagon_32_6_X:
   154:     return ELF::R_HEX_32_6_X;
   155:   case fixup_Hexagon_B22_PCREL_X:
   156:     return ELF::R_HEX_B22_PCREL_X;
   157:   case fixup_Hexagon_B15_PCREL_X:
   158:     return ELF::R_HEX_B15_PCREL_X;
   159:   case fixup_Hexagon_B13_PCREL_X:
   160:     return ELF::R_HEX_B13_PCREL_X;
   161:   case fixup_Hexagon_B9_PCREL_X:
   162:     return ELF::R_HEX_B9_PCREL_X;
   163:   case fixup_Hexagon_B7_PCREL_X:
   164:     return ELF::R_HEX_B7_PCREL_X;
   165:   case fixup_Hexagon_16_X:
   166:     return ELF::R_HEX_16_X;
   167:   case fixup_Hexagon_12_X:
   168:     return ELF::R_HEX_12_X;
   169:   case fixup_Hexagon_11_X:
   170:     return ELF::R_HEX_11_X;
   171:   case fixup_Hexagon_10_X:
   172:     return ELF::R_HEX_10_X;
   173:   case fixup_Hexagon_9_X:
   174:     return ELF::R_HEX_9_X;
   175:   case fixup_Hexagon_8_X:
   176:     return ELF::R_HEX_8_X;
   177:   case fixup_Hexagon_7_X:
   178:     return ELF::R_HEX_7_X;
   179:   case fixup_Hexagon_6_X:
   180:     return ELF::R_HEX_6_X;
   181:   case fixup_Hexagon_32_PCREL:
   182:     return ELF::R_HEX_32_PCREL;
   183:   case fixup_Hexagon_COPY:
   184:     return ELF::R_HEX_COPY;
   185:   case fixup_Hexagon_GLOB_DAT:
   186:     return ELF::R_HEX_GLOB_DAT;
   187:   case fixup_Hexagon_JMP_SLOT:
   188:     return ELF::R_HEX_JMP_SLOT;
   189:   case fixup_Hexagon_RELATIVE:
   190:     return ELF::R_HEX_RELATIVE;
   191:   case fixup_Hexagon_PLT_B22_PCREL:
   192:     return ELF::R_HEX_PLT_B22_PCREL;
   193:   case fixup_Hexagon_GOTREL_LO16:
   194:     return ELF::R_HEX_GOTREL_LO16;
   195:   case fixup_Hexagon_GOTREL_HI16:
   196:     return ELF::R_HEX_GOTREL_HI16;
   197:   case fixup_Hexagon_GOTREL_32:
   198:     return ELF::R_HEX_GOTREL_32;
   199:   case fixup_Hexagon_GOT_LO16:
   200:     return ELF::R_HEX_GOT_LO16;
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   case fixup_Hexagon_GOT_HI16:
   202:     return ELF::R_HEX_GOT_HI16;
   203:   case fixup_Hexagon_GOT_32:
   204:     return ELF::R_HEX_GOT_32;
   205:   case fixup_Hexagon_GOT_16:
   206:     return ELF::R_HEX_GOT_16;
   207:   case fixup_Hexagon_DTPMOD_32:
   208:     return ELF::R_HEX_DTPMOD_32;
   209:   case fixup_Hexagon_DTPREL_LO16:
   210:     return ELF::R_HEX_DTPREL_LO16;
   211:   case fixup_Hexagon_DTPREL_HI16:
   212:     return ELF::R_HEX_DTPREL_HI16;
   213:   case fixup_Hexagon_DTPREL_32:
   214:     return ELF::R_HEX_DTPREL_32;
   215:   case fixup_Hexagon_DTPREL_16:
   216:     return ELF::R_HEX_DTPREL_16;
   217:   case fixup_Hexagon_GD_PLT_B22_PCREL:
   218:     return ELF::R_HEX_GD_PLT_B22_PCREL;
   219:   case fixup_Hexagon_LD_PLT_B22_PCREL:
   220:     return ELF::R_HEX_LD_PLT_B22_PCREL;
   221:   case fixup_Hexagon_GD_GOT_LO16:
   222:     return ELF::R_HEX_GD_GOT_LO16;
   223:   case fixup_Hexagon_GD_GOT_HI16:
   224:     return ELF::R_HEX_GD_GOT_HI16;
   225:   case fixup_Hexagon_GD_GOT_32:
   226:     return ELF::R_HEX_GD_GOT_32;
   227:   case fixup_Hexagon_GD_GOT_16:
   228:     return ELF::R_HEX_GD_GOT_16;
   229:   case fixup_Hexagon_LD_GOT_LO16:
   230:     return ELF::R_HEX_LD_GOT_LO16;
   231:   case fixup_Hexagon_LD_GOT_HI16:
   232:     return ELF::R_HEX_LD_GOT_HI16;
   233:   case fixup_Hexagon_LD_GOT_32:
   234:     return ELF::R_HEX_LD_GOT_32;
   235:   case fixup_Hexagon_LD_GOT_16:
   236:     return ELF::R_HEX_LD_GOT_16;
   237:   case fixup_Hexagon_IE_LO16:
   238:     return ELF::R_HEX_IE_LO16;
   239:   case fixup_Hexagon_IE_HI16:
   240:     return ELF::R_HEX_IE_HI16;
   241:   case fixup_Hexagon_IE_32:
   242:     return ELF::R_HEX_IE_32;
   243:   case fixup_Hexagon_IE_GOT_LO16:
   244:     return ELF::R_HEX_IE_GOT_LO16;
   245:   case fixup_Hexagon_IE_GOT_HI16:
   246:     return ELF::R_HEX_IE_GOT_HI16;
   247:   case fixup_Hexagon_IE_GOT_32:
   248:     return ELF::R_HEX_IE_GOT_32;
   249:   case fixup_Hexagon_IE_GOT_16:
   250:     return ELF::R_HEX_IE_GOT_16;
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   case fixup_Hexagon_TPREL_LO16:
   252:     return ELF::R_HEX_TPREL_LO16;
   253:   case fixup_Hexagon_TPREL_HI16:
   254:     return ELF::R_HEX_TPREL_HI16;
   255:   case fixup_Hexagon_TPREL_32:
   256:     return ELF::R_HEX_TPREL_32;
   257:   case fixup_Hexagon_TPREL_16:
   258:     return ELF::R_HEX_TPREL_16;
   259:   case fixup_Hexagon_6_PCREL_X:
   260:     return ELF::R_HEX_6_PCREL_X;
   261:   case fixup_Hexagon_GOTREL_32_6_X:
   262:     return ELF::R_HEX_GOTREL_32_6_X;
   263:   case fixup_Hexagon_GOTREL_16_X:
   264:     return ELF::R_HEX_GOTREL_16_X;
   265:   case fixup_Hexagon_GOTREL_11_X:
   266:     return ELF::R_HEX_GOTREL_11_X;
   267:   case fixup_Hexagon_GOT_32_6_X:
   268:     return ELF::R_HEX_GOT_32_6_X;
   269:   case fixup_Hexagon_GOT_16_X:
   270:     return ELF::R_HEX_GOT_16_X;
   271:   case fixup_Hexagon_GOT_11_X:
   272:     return ELF::R_HEX_GOT_11_X;
   273:   case fixup_Hexagon_DTPREL_32_6_X:
   274:     return ELF::R_HEX_DTPREL_32_6_X;
   275:   case fixup_Hexagon_DTPREL_16_X:
   276:     return ELF::R_HEX_DTPREL_16_X;
   277:   case fixup_Hexagon_DTPREL_11_X:
   278:     return ELF::R_HEX_DTPREL_11_X;
   279:   case fixup_Hexagon_GD_GOT_32_6_X:
   280:     return ELF::R_HEX_GD_GOT_32_6_X;
   281:   case fixup_Hexagon_GD_GOT_16_X:
   282:     return ELF::R_HEX_GD_GOT_16_X;
   283:   case fixup_Hexagon_GD_GOT_11_X:
   284:     return ELF::R_HEX_GD_GOT_11_X;
   285:   case fixup_Hexagon_LD_GOT_32_6_X:
   286:     return ELF::R_HEX_LD_GOT_32_6_X;
   287:   case fixup_Hexagon_LD_GOT_16_X:
   288:     return ELF::R_HEX_LD_GOT_16_X;
   289:   case fixup_Hexagon_LD_GOT_11_X:
   290:     return ELF::R_HEX_LD_GOT_11_X;
   291:   case fixup_Hexagon_IE_32_6_X:
   292:     return ELF::R_HEX_IE_32_6_X;
   293:   case fixup_Hexagon_IE_16_X:
   294:     return ELF::R_HEX_IE_16_X;
   295:   case fixup_Hexagon_IE_GOT_32_6_X:
   296:     return ELF::R_HEX_IE_GOT_32_6_X;
   297:   case fixup_Hexagon_IE_GOT_16_X:
   298:     return ELF::R_HEX_IE_GOT_16_X;
   299:   case fixup_Hexagon_IE_GOT_11_X:
   300:     return ELF::R_HEX_IE_GOT_11_X;
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-325 / 第 301-325 行

```cpp
   301:   case fixup_Hexagon_TPREL_32_6_X:
   302:     return ELF::R_HEX_TPREL_32_6_X;
   303:   case fixup_Hexagon_TPREL_16_X:
   304:     return ELF::R_HEX_TPREL_16_X;
   305:   case fixup_Hexagon_TPREL_11_X:
   306:     return ELF::R_HEX_TPREL_11_X;
   307:   case fixup_Hexagon_23_REG:
   308:     return ELF::R_HEX_23_REG;
   309:   case fixup_Hexagon_27_REG:
   310:     return ELF::R_HEX_27_REG;
   311:   case fixup_Hexagon_GD_PLT_B22_PCREL_X:
   312:     return ELF::R_HEX_GD_PLT_B22_PCREL_X;
   313:   case fixup_Hexagon_GD_PLT_B32_PCREL_X:
   314:     return ELF::R_HEX_GD_PLT_B32_PCREL_X;
   315:   case fixup_Hexagon_LD_PLT_B22_PCREL_X:
   316:     return ELF::R_HEX_LD_PLT_B22_PCREL_X;
   317:   case fixup_Hexagon_LD_PLT_B32_PCREL_X:
   318:     return ELF::R_HEX_LD_PLT_B32_PCREL_X;
   319:   }
   320: }
   321: 
   322: std::unique_ptr<MCObjectTargetWriter>
   323: llvm::createHexagonELFObjectWriter(uint8_t OSABI, StringRef CPU) {
   324:   return std::make_unique<HexagonELFObjectWriter>(OSABI, CPU);
   325: }
```
- EN: It declares or implements routines such as llvm::createHexagonELFObjectWriter, std::make_unique<HexagonELFObjectWriter>, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonELFObjectWriter, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonELFObjectWriter, std::make_unique<HexagonELFObjectWriter> 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonELFObjectWriter，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonFixupKinds.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/MC/MCAssembler.h, llvm/MC/MCELFObjectWriter.h, llvm/MC/MCObjectWriter.h, llvm/MC/MCValue.h`
- Hexagon symbols / Hexagon 符号: `HexagonELFObjectWriter, HexagonFixupKinds, HexagonMCExpr, HexagonMCTargetDesc`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
