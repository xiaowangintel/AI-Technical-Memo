# SystemZHLASMAsmStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZHLASMAsmStreamer.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===- SystemZHLASMAsmStreamer.cpp - HLASM Assembly Text Output -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZHLASMAsmStreamer.h"
  10: #include "llvm/ADT/StringExtras.h"
  11: #include "llvm/BinaryFormat/GOFF.h"
  12: #include "llvm/MC/MCExpr.h"
  13: #include "llvm/MC/MCGOFFAttributes.h"
  14: #include "llvm/MC/MCGOFFStreamer.h"
  15: #include "llvm/MC/MCSymbolGOFF.h"
  16: #include "llvm/Support/Casting.h"
  17: #include "llvm/Support/Signals.h"
  18: #include <sstream>
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZHLASMAsmStreamer.h`, `StringExtras.h`, `GOFF.h`, `MCExpr.h`, `MCGOFFAttributes.h`, `MCGOFFStreamer.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZHLASMAsmStreamer.h`, `StringExtras.h`, `GOFF.h`, `MCExpr.h`, `MCGOFFAttributes.h`, `MCGOFFStreamer.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: using namespace llvm;
  21: 
  22: void SystemZHLASMAsmStreamer::visitUsedSymbol(const MCSymbol &Sym) {
  23:   Assembler->registerSymbol(Sym);
  24: }
  25: 
  26: void SystemZHLASMAsmStreamer::EmitEOL() {
  27:   // Comments are emitted on a new line before the instruction.
  28:   if (IsVerboseAsm)
  29:     EmitComment();
  30: 
  31:   std::istringstream Stream(Str);
  32:   SmallVector<std::string> Lines;
  33:   std::string Line;
  34:   while (std::getline(Stream, Line, '\n'))
  35:     Lines.push_back(Line);
  36: 
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::visitUsedSymbol`, `SystemZHLASMAsmStreamer::EmitEOL`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::visitUsedSymbol`, `SystemZHLASMAsmStreamer::EmitEOL` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   for (auto S : Lines) {
  38:     if (LLVM_LIKELY(S.length() < ContIndicatorColumn)) {
  39:       FOS << S;
  40:       // Each line in HLASM must fill the full 80 characters.
  41:       FOS.PadToColumn(InstLimit);
  42:       FOS << "\n";
  43:     } else {
  44:       // If last character before end of the line is not a space
  45:       // we must insert an additional non-space character that
  46:       // is not part of the statement coding. We just reuse
  47:       // the existing character by making the new substring start
  48:       // 1 character sooner, thus "duplicating" that character
  49:       // If The last character is a space. We insert an X instead.
  50:       std::string TmpSubStr = S.substr(0, ContIndicatorColumn);
  51:       if (!TmpSubStr.compare(ContIndicatorColumn - 1, 1, " "))
  52:         TmpSubStr.replace(ContIndicatorColumn - 1, 1, "X");
  53: 
  54:       FOS << TmpSubStr;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 55-72 / 第 55-72 行
```cpp
  55:       FOS.PadToColumn(InstLimit);
  56:       FOS << "\n";
  57: 
  58:       size_t Emitted = ContIndicatorColumn - 1;
  59: 
  60:       while (Emitted < S.length()) {
  61:         if ((S.length() - Emitted) < ContLen)
  62:           TmpSubStr = S.substr(Emitted, S.length());
  63:         else {
  64:           TmpSubStr = S.substr(Emitted, ContLen);
  65:           if (!TmpSubStr.compare(ContLen - 1, 1, " "))
  66:             TmpSubStr.replace(ContLen - 1, 1, "X");
  67:         }
  68:         FOS.PadToColumn(ContStartColumn);
  69:         FOS << TmpSubStr;
  70:         FOS.PadToColumn(InstLimit);
  71:         FOS << "\n";
  72:         Emitted += ContLen - 1;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 73-90 / 第 73-90 行
```cpp
  73:       }
  74:     }
  75:   }
  76:   Str.clear();
  77: }
  78: 
  79: void SystemZHLASMAsmStreamer::changeSection(MCSection *Section,
  80:                                             uint32_t Subsection) {
  81:   MAI->printSwitchToSection(*Section, Subsection,
  82:                             getContext().getTargetTriple(), OS);
  83:   MCStreamer::changeSection(Section, Subsection);
  84:   EmitEOL();
  85: }
  86: 
  87: void SystemZHLASMAsmStreamer::emitAlignmentDS(uint64_t ByteAlignment,
  88:                                               std::optional<int64_t> Value,
  89:                                               unsigned ValueSize,
  90:                                               unsigned MaxBytesToEmit) {
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::changeSection`, `SystemZHLASMAsmStreamer::emitAlignmentDS`.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::changeSection`, `SystemZHLASMAsmStreamer::emitAlignmentDS` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   if (!isPowerOf2_64(ByteAlignment))
  92:     report_fatal_error("Only power-of-two alignments are supported ");
  93: 
  94:   OS << " DS 0";
  95:   switch (ValueSize) {
  96:   default:
  97:     llvm_unreachable("Invalid size for machine code value!");
  98:   case 1:
  99:     OS << "B";
 100:     break;
 101:   case 2:
 102:     OS << "H";
 103:     break;
 104:   case 4:
 105:     OS << "F";
 106:     break;
 107:   case 8:
 108:     OS << "D";
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     break;
 110:   case 16:
 111:     OS << "Q";
 112:     break;
 113:   }
 114: 
 115:   EmitEOL();
 116: }
 117: 
 118: void SystemZHLASMAsmStreamer::AddComment(const Twine &T, bool EOL) {
 119:   if (!IsVerboseAsm)
 120:     return;
 121: 
 122:   T.toVector(CommentToEmit);
 123: 
 124:   if (EOL)
 125:     CommentToEmit.push_back('\n'); // Place comment in a new line.
 126: }
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::AddComment`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::AddComment` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127: 
 128: void SystemZHLASMAsmStreamer::EmitComment() {
 129:   if (CommentToEmit.empty() && CommentStream.GetNumBytesInBuffer() == 0)
 130:     return;
 131: 
 132:   StringRef Comments = CommentToEmit;
 133: 
 134:   assert(Comments.back() == '\n' && "Comment array not newline terminated");
 135:   do {
 136:     // Emit a line of comments, but not exceeding 80 characters.
 137:     size_t Position = std::min(InstLimit - 2, Comments.find('\n'));
 138:     FOS << MAI->getCommentString() << ' ' << Comments.substr(0, Position)
 139:         << '\n';
 140: 
 141:     if (Comments[Position] == '\n')
 142:       Position++;
 143:     Comments = Comments.substr(Position);
 144:   } while (!Comments.empty());
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::EmitComment`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::EmitComment` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 145-162 / 第 145-162 行
```cpp
 145: 
 146:   CommentToEmit.clear();
 147: }
 148: 
 149: void SystemZHLASMAsmStreamer::emitValueToAlignment(Align Alignment,
 150:                                                    int64_t Fill,
 151:                                                    uint8_t FillLen,
 152:                                                    unsigned MaxBytesToEmit) {
 153:   emitAlignmentDS(Alignment.value(), Fill, FillLen, MaxBytesToEmit);
 154: }
 155: 
 156: void SystemZHLASMAsmStreamer::emitCodeAlignment(Align Alignment,
 157:                                                 const MCSubtargetInfo *STI,
 158:                                                 unsigned MaxBytesToEmit) {
 159:   // Emit with a text fill value.
 160:   if (MAI->getTextAlignFillValue())
 161:     emitAlignmentDS(Alignment.value(), MAI->getTextAlignFillValue(), 1,
 162:                     MaxBytesToEmit);
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::emitValueToAlignment`, `SystemZHLASMAsmStreamer::emitCodeAlignment`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::emitValueToAlignment`, `SystemZHLASMAsmStreamer::emitCodeAlignment` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   else
 164:     emitAlignmentDS(Alignment.value(), std::nullopt, 1, MaxBytesToEmit);
 165: }
 166: 
 167: void SystemZHLASMAsmStreamer::emitBytes(StringRef Data) {
 168:   assert(getCurrentSectionOnly() &&
 169:          "Cannot emit contents before setting section!");
 170:   if (Data.empty())
 171:     return;
 172: 
 173:   OS << " DC ";
 174:   size_t Len = Data.size();
 175:   SmallVector<uint8_t> Chars;
 176:   Chars.resize(Len);
 177:   OS << "XL" << Len;
 178:   uint32_t Index = 0;
 179:   for (uint8_t C : Data) {
 180:     Chars[Index] = C;
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::emitBytes`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::emitBytes` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     Index++;
 182:   }
 183: 
 184:   OS << '\'' << toHex(Chars) << '\'';
 185: 
 186:   EmitEOL();
 187: }
 188: 
 189: void SystemZHLASMAsmStreamer::emitInstruction(const MCInst &Inst,
 190:                                               const MCSubtargetInfo &STI) {
 191: 
 192:   InstPrinter->printInst(&Inst, 0, "", STI, OS);
 193:   EmitEOL();
 194: }
 195: 
 196: static void emitXATTR(raw_ostream &OS, StringRef Name, MCSectionGOFF *ADA,
 197:                       bool IsIndirectReference, GOFF::ESDLinkageType Linkage,
 198:                       GOFF::ESDExecutable Executable,
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::emitInstruction`.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::emitInstruction` 等函数。

### Lines 199-216 / 第 199-216 行
```cpp
 199:                       GOFF::ESDBindingScope BindingScope) {
 200:   llvm::ListSeparator Sep(",");
 201:   OS << Name << " XATTR ";
 202:   OS << Sep << "LINKAGE(" << (Linkage == GOFF::ESD_LT_OS ? "OS" : "XPLINK")
 203:      << ")";
 204: 
 205:   const bool NotUnspecified = (Executable != GOFF::ESD_EXE_Unspecified);
 206:   if (NotUnspecified || IsIndirectReference) {
 207:     OS << Sep << "REFERENCE(";
 208:     llvm::ListSeparator SepRef(",");
 209: 
 210:     if (NotUnspecified)
 211:       OS << SepRef << (Executable == GOFF::ESD_EXE_CODE ? "CODE" : "DATA");
 212: 
 213:     if (IsIndirectReference)
 214:       OS << SepRef << "INDIRECT";
 215: 
 216:     OS << ")";
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   }
 218:   // Emit PSECT only for code symbols.
 219:   if (ADA && Executable != GOFF::ESD_EXE_DATA)
 220:     OS << Sep << "PSECT(" << ADA->getName() << ")";
 221:   if (BindingScope != GOFF::ESD_BSC_Unspecified) {
 222:     OS << Sep << "SCOPE(";
 223:     switch (BindingScope) {
 224:     case GOFF::ESD_BSC_Section:
 225:       OS << "SECTION";
 226:       break;
 227:     case GOFF::ESD_BSC_Module:
 228:       OS << "MODULE";
 229:       break;
 230:     case GOFF::ESD_BSC_Library:
 231:       OS << "LIBRARY";
 232:       break;
 233:     case GOFF::ESD_BSC_ImportExport:
 234:       OS << "EXPORT";
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:       break;
 236:     default:
 237:       break;
 238:     }
 239:     OS << ')';
 240:   }
 241: }
 242: 
 243: void SystemZHLASMAsmStreamer::emitLabel(MCSymbol *Symbol, SMLoc Loc) {
 244:   MCSymbolGOFF *Sym = static_cast<MCSymbolGOFF *>(Symbol);
 245: 
 246:   MCStreamer::emitLabel(Sym, Loc);
 247: 
 248:   // Emit label and ENTRY statement only if not implied by CSECT. Do not emit a
 249:   // label if the symbol is on a PR section.
 250:   bool EmitLabelAndEntry =
 251:       !static_cast<MCSectionGOFF *>(getCurrentSectionOnly())->isPR();
 252:   if (!Sym->isTemporary() && Sym->isInEDSection()) {
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::emitLabel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::emitLabel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253:     EmitLabelAndEntry =
 254:         Sym->getName() !=
 255:         static_cast<MCSectionGOFF &>(Sym->getSection()).getParent()->getName();
 256:     if (EmitLabelAndEntry) {
 257:       OS << " ENTRY " << Sym->getName();
 258:       EmitEOL();
 259:     }
 260: 
 261:     emitXATTR(OS, Sym->getName(), Sym->getADA(), Sym->isIndirect(),
 262:               Sym->getLinkage(), Sym->getCodeData(), Sym->getBindingScope());
 263:     EmitEOL();
 264:     if (Sym->hasExternalName())
 265:       OS << Sym->getName() << " ALIAS C'" << Sym->getExternalName() << "'\n";
 266:   }
 267: 
 268:   if (EmitLabelAndEntry) {
 269:     OS << Sym->getName() << " DS 0H";
 270:     EmitEOL();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 271-288 / 第 271-288 行
```cpp
 271:   }
 272: }
 273: 
 274: bool SystemZHLASMAsmStreamer::emitSymbolAttribute(MCSymbol *Sym,
 275:                                                   MCSymbolAttr Attribute) {
 276:   return static_cast<MCSymbolGOFF *>(Sym)->setSymbolAttribute(Attribute);
 277: }
 278: 
 279: void SystemZHLASMAsmStreamer::emitRawTextImpl(StringRef String) {
 280:   String.consume_back("\n");
 281:   OS << String;
 282:   EmitEOL();
 283: }
 284: 
 285: // Slight duplicate of MCExpr::print due to HLASM only recognizing limited
 286: // arithmetic operators (+-*/).
 287: void SystemZHLASMAsmStreamer::emitHLASMValueImpl(const MCExpr *Value,
 288:                                                  unsigned Size, bool Parens) {
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::emitSymbolAttribute`, `SystemZHLASMAsmStreamer::emitRawTextImpl`, `SystemZHLASMAsmStreamer::emitHLASMValueImpl`.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::emitSymbolAttribute`, `SystemZHLASMAsmStreamer::emitRawTextImpl`, `SystemZHLASMAsmStreamer::emitHLASMValueImpl` 等函数。

### Lines 289-306 / 第 289-306 行
```cpp
 289:   switch (Value->getKind()) {
 290:   case MCExpr::Constant: {
 291:     OS << "XL" << Size << '\'';
 292:     MAI->printExpr(OS, *Value);
 293:     OS << '\'';
 294:     return;
 295:   }
 296:   case MCExpr::Binary: {
 297:     const MCBinaryExpr &BE = cast<MCBinaryExpr>(*Value);
 298:     int64_t Const;
 299:     // Or is handled differently.
 300:     if (BE.getOpcode() == MCBinaryExpr::Or) {
 301:       emitHLASMValueImpl(BE.getLHS(), Size, true);
 302:       OS << ',';
 303:       emitHLASMValueImpl(BE.getRHS(), Size, true);
 304:       return;
 305:     }
 306: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 307-324 / 第 307-324 行
```cpp
 307:     if (Parens)
 308:       OS << "AD(";
 309:     emitHLASMValueImpl(BE.getLHS(), Size);
 310: 
 311:     switch (BE.getOpcode()) {
 312:     case MCBinaryExpr::LShr: {
 313:       Const = cast<MCConstantExpr>(BE.getRHS())->getValue();
 314:       OS << '/' << (1 << Const);
 315:       if (Parens)
 316:         OS << ')';
 317:       return;
 318:     }
 319:     case MCBinaryExpr::Add:
 320:       OS << '+';
 321:       break;
 322:     case MCBinaryExpr::Div:
 323:       OS << '/';
 324:       break;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 325-342 / 第 325-342 行
```cpp
 325:     case MCBinaryExpr::Mul:
 326:       OS << '*';
 327:       break;
 328:     case MCBinaryExpr::Sub:
 329:       OS << '-';
 330:       break;
 331:     default:
 332:       getContext().reportError(SMLoc(),
 333:                                "Unrecognized HLASM arithmetic expression!");
 334:     }
 335:     emitHLASMValueImpl(BE.getRHS(), Size);
 336:     if (Parens)
 337:       OS << ')';
 338:     return;
 339:   }
 340:   case MCExpr::Target:
 341:     MAI->printExpr(OS, *Value);
 342:     return;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 343-360 / 第 343-360 行
```cpp
 343:   default:
 344:     Parens &= isa<MCSymbolRefExpr>(Value);
 345:     if (Parens)
 346:       OS << "AD(";
 347:     MAI->printExpr(OS, *Value);
 348:     if (Parens)
 349:       OS << ')';
 350:     return;
 351:   }
 352: }
 353: 
 354: void SystemZHLASMAsmStreamer::emitValueImpl(const MCExpr *Value, unsigned Size,
 355:                                             SMLoc Loc) {
 356:   assert(Size <= 8 && "Invalid size");
 357:   assert(getCurrentSectionOnly() &&
 358:          "Cannot emit contents before setting section!");
 359: 
 360:   MCStreamer::emitValueImpl(Value, Size, Loc);
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::emitValueImpl`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::emitValueImpl` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 361-378 / 第 361-378 行
```cpp
 361:   OS << " DC ";
 362:   emitHLASMValueImpl(Value, Size, true);
 363:   EmitEOL();
 364: }
 365: 
 366: void SystemZHLASMAsmStreamer::finishImpl() {
 367:   for (auto &Symbol : getAssembler().symbols()) {
 368:     if (Symbol.isTemporary() || !Symbol.isRegistered() || Symbol.isDefined())
 369:       continue;
 370:     auto &Sym = static_cast<MCSymbolGOFF &>(const_cast<MCSymbol &>(Symbol));
 371:     if (Sym.getCodeData() == GOFF::ESD_EXE_DATA) {
 372:       OS << Sym.getADA()->getParent()->getExternalName() << " CATTR PART("
 373:          << Sym.getName() << ")";
 374:       EmitEOL();
 375:     } else {
 376:       OS << " " << (Sym.isWeak() ? "WXTRN" : "EXTRN") << " " << Sym.getName();
 377:       EmitEOL();
 378:     }
```
- **EN**: The range implements or declares functions including `SystemZHLASMAsmStreamer::finishImpl`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHLASMAsmStreamer::finishImpl` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 379-389 / 第 379-389 行
```cpp
 379:     emitXATTR(OS, Sym.getName(), Sym.getADA(), Sym.isIndirect(),
 380:               Sym.getLinkage(), Sym.getCodeData(), Sym.getBindingScope());
 381:     EmitEOL();
 382:     if (Sym.hasExternalName())
 383:       OS << Sym.getName() << " ALIAS C'" << Sym.getExternalName() << "'\n";
 384:   }
 385: 
 386:   // Finish the assembly output.
 387:   OS << " END";
 388:   EmitEOL();
 389: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZHLASMAsmStreamer.h`
- `llvm/ADT/StringExtras.h`
- `llvm/BinaryFormat/GOFF.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCGOFFAttributes.h`
- `llvm/MC/MCGOFFStreamer.h`
- `llvm/MC/MCSymbolGOFF.h`
- `llvm/Support/Casting.h`
- `llvm/Support/Signals.h`
- `sstream`
