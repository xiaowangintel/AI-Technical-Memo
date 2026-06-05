# ELFAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/ELFAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements ELF Assembly Parser.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- ELFAsmParser.cpp - ELF Assembly Parser -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/ScopeExit.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDirectives.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParser.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolELF.h"
#include "llvm/MC/SectionKind.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/ScopeExit.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/ScopeExit.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`。

### Lines 25-38
```cpp
#include "llvm/Support/SMLoc.h"
#include <cassert>
#include <cstdint>

using namespace llvm;

namespace {

class ELFAsmParser : public MCAsmParserExtension {
  template<bool (ELFAsmParser::*HandlerMethod)(StringRef, SMLoc)>
  void addDirectiveHandler(StringRef Directive) {
    MCAsmParser::ExtensionDirectiveHandler Handler = std::make_pair(
        this, HandleDirective<ELFAsmParser, HandlerMethod>);

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/SMLoc.h`, `cassert`, `cstdint`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/SMLoc.h`, `cassert`, `cstdint`。

### Lines 39-51
```cpp
    getParser().addDirectiveHandler(Directive, Handler);
  }

  bool parseSectionSwitch(StringRef Section, unsigned Type, unsigned Flags,
                          SectionKind Kind);

public:
  ELFAsmParser() { BracketExpressionsSupported = true; }

  void Initialize(MCAsmParser &Parser) override {
    // Call the base implementation.
    this->MCAsmParserExtension::Initialize(Parser);

```
- **EN**: Implements logic around `getParser`, `parseSectionSwitch`, `ELFAsmParser`, `Initialize`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getParser`, `parseSectionSwitch`, `ELFAsmParser`, `Initialize` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 52-75
```cpp
    addDirectiveHandler<&ELFAsmParser::parseSectionDirectiveData>(".data");
    addDirectiveHandler<&ELFAsmParser::parseSectionDirectiveText>(".text");
    addDirectiveHandler<&ELFAsmParser::parseSectionDirectiveBSS>(".bss");
    addDirectiveHandler<&ELFAsmParser::parseSectionDirectiveRoData>(".rodata");
    addDirectiveHandler<&ELFAsmParser::parseSectionDirectiveTData>(".tdata");
    addDirectiveHandler<&ELFAsmParser::parseSectionDirectiveTBSS>(".tbss");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveSection>(".section");
    addDirectiveHandler<
      &ELFAsmParser::parseDirectivePushSection>(".pushsection");
    addDirectiveHandler<&ELFAsmParser::parseDirectivePopSection>(".popsection");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveSize>(".size");
    addDirectiveHandler<&ELFAsmParser::parseDirectivePrevious>(".previous");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveType>(".type");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveIdent>(".ident");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveSymver>(".symver");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveVersion>(".version");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveWeakref>(".weakref");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveSymbolAttribute>(".weak");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveSymbolAttribute>(".local");
    addDirectiveHandler<
      &ELFAsmParser::parseDirectiveSymbolAttribute>(".protected");
    addDirectiveHandler<
      &ELFAsmParser::parseDirectiveSymbolAttribute>(".internal");
    addDirectiveHandler<
```
- **EN**: Implements logic around `parseSectionDirectiveData>`, `parseSectionDirectiveText>`, `parseSectionDirectiveBSS>`, `parseSectionDirectiveRoData>`, and 13 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `parseSectionDirectiveData>`, `parseSectionDirectiveText>`, `parseSectionDirectiveBSS>`, `parseSectionDirectiveRoData>`, and 13 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 76-99
```cpp
      &ELFAsmParser::parseDirectiveSymbolAttribute>(".hidden");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveSubsection>(".subsection");
    addDirectiveHandler<&ELFAsmParser::parseDirectiveCGProfile>(".cg_profile");
  }

  // FIXME: Part of this logic is duplicated in the MCELFStreamer. What is
  // the best way for us to get access to it?
  bool parseSectionDirectiveData(StringRef, SMLoc) {
    return parseSectionSwitch(".data", ELF::SHT_PROGBITS,
                              ELF::SHF_WRITE | ELF::SHF_ALLOC,
                              SectionKind::getData());
  }
  bool parseSectionDirectiveText(StringRef, SMLoc) {
    return parseSectionSwitch(".text", ELF::SHT_PROGBITS,
                              ELF::SHF_EXECINSTR |
                              ELF::SHF_ALLOC, SectionKind::getText());
  }
  bool parseSectionDirectiveBSS(StringRef, SMLoc) {
    return parseSectionSwitch(".bss", ELF::SHT_NOBITS,
                              ELF::SHF_WRITE |
                              ELF::SHF_ALLOC, SectionKind::getBSS());
  }
  bool parseSectionDirectiveRoData(StringRef, SMLoc) {
    return parseSectionSwitch(".rodata", ELF::SHT_PROGBITS,
```
- **EN**: Implements logic around `parseDirectiveSymbolAttribute>`, `parseDirectiveSubsection>`, `parseDirectiveCGProfile>`, `parseSectionDirectiveData`, and 7 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSymbolAttribute>`, `parseDirectiveSubsection>`, `parseDirectiveCGProfile>`, `parseSectionDirectiveData`, and 7 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 100-123
```cpp
                              ELF::SHF_ALLOC,
                              SectionKind::getReadOnly());
  }
  bool parseSectionDirectiveTData(StringRef, SMLoc) {
    return parseSectionSwitch(".tdata", ELF::SHT_PROGBITS,
                              ELF::SHF_ALLOC |
                              ELF::SHF_TLS | ELF::SHF_WRITE,
                              SectionKind::getThreadData());
  }
  bool parseSectionDirectiveTBSS(StringRef, SMLoc) {
    return parseSectionSwitch(".tbss", ELF::SHT_NOBITS,
                              ELF::SHF_ALLOC |
                              ELF::SHF_TLS | ELF::SHF_WRITE,
                              SectionKind::getThreadBSS());
  }
  bool parseDirectivePushSection(StringRef, SMLoc);
  bool parseDirectivePopSection(StringRef, SMLoc);
  bool parseDirectiveSection(StringRef, SMLoc);
  bool parseDirectiveSize(StringRef, SMLoc);
  bool parseDirectivePrevious(StringRef, SMLoc);
  bool parseDirectiveType(StringRef, SMLoc);
  bool parseDirectiveIdent(StringRef, SMLoc);
  bool parseDirectiveSymver(StringRef, SMLoc);
  bool parseDirectiveVersion(StringRef, SMLoc);
```
- **EN**: Implements logic around `getReadOnly`, `parseSectionDirectiveTData`, `parseSectionSwitch`, `getThreadData`, and 11 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getReadOnly`, `parseSectionDirectiveTData`, `parseSectionSwitch`, `getThreadData`, and 11 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 124-138
```cpp
  bool parseDirectiveWeakref(StringRef, SMLoc);
  bool parseDirectiveSymbolAttribute(StringRef, SMLoc);
  bool parseDirectiveSubsection(StringRef, SMLoc);
  bool parseDirectiveCGProfile(StringRef, SMLoc);

private:
  bool parseSectionName(StringRef &SectionName);
  bool parseSectionArguments(bool IsPush, SMLoc loc);
  unsigned parseSunStyleSectionFlags();
  bool maybeParseSectionType(StringRef &TypeName);
  bool parseMergeSize(int64_t &Size);
  bool parseGroup(StringRef &GroupName, bool &IsComdat);
  bool parseLinkedToSym(MCSymbolELF *&LinkedToSym);
};

```
- **EN**: Implements logic around `parseDirectiveWeakref`, `parseDirectiveSymbolAttribute`, `parseDirectiveSubsection`, `parseDirectiveCGProfile`, and 7 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `parseDirectiveWeakref`, `parseDirectiveSymbolAttribute`, `parseDirectiveSubsection`, `parseDirectiveCGProfile`, and 7 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 139-155
```cpp
} // end anonymous namespace

/// parseDirectiveSymbolAttribute
///  ::= { ".local", ".weak", ... } [ identifier ( , identifier )* ]
bool ELFAsmParser::parseDirectiveSymbolAttribute(StringRef Directive, SMLoc) {
  MCSymbolAttr Attr = StringSwitch<MCSymbolAttr>(Directive)
    .Case(".weak", MCSA_Weak)
    .Case(".local", MCSA_Local)
    .Case(".hidden", MCSA_Hidden)
    .Case(".internal", MCSA_Internal)
    .Case(".protected", MCSA_Protected)
    .Default(MCSA_Invalid);
  assert(Attr != MCSA_Invalid && "unexpected symbol attribute directive!");
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    while (true) {
      StringRef Name;

```
- **EN**: Implements logic around `parseDirectiveSymbolAttribute`, `StringSwitch<MCSymbolAttr>`, `Case`, `Default`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `parseDirectiveSymbolAttribute`, `StringSwitch<MCSymbolAttr>`, `Case`, `Default`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 156-168
```cpp
      if (getParser().parseIdentifier(Name))
        return TokError("expected identifier");

      if (getParser().discardLTOSymbol(Name)) {
        if (getLexer().is(AsmToken::EndOfStatement))
          break;
        continue;
      }

      MCSymbol *Sym = getContext().parseSymbol(Name);

      getStreamer().emitSymbolAttribute(Sym, Attr);

```
- **EN**: Implements logic around `TokError`, `getContext`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `getContext`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 169-181
```cpp
      if (getLexer().is(AsmToken::EndOfStatement))
        break;

      if (getLexer().isNot(AsmToken::Comma))
        return TokError("expected comma");
      Lex();
    }
  }

  Lex();
  return false;
}

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 182-193
```cpp
bool ELFAsmParser::parseSectionSwitch(StringRef Section, unsigned Type,
                                      unsigned Flags, SectionKind Kind) {
  const MCExpr *Subsection = nullptr;
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    if (getParser().parseExpression(Subsection))
      return true;
  }
  Lex();

  getStreamer().switchSection(getContext().getELFSection(Section, Type, Flags),
                              Subsection);

```
- **EN**: Implements logic around `parseSectionSwitch`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionSwitch`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 194-205
```cpp
  return false;
}

bool ELFAsmParser::parseDirectiveSize(StringRef, SMLoc) {
  MCSymbol *Sym;
  if (getParser().parseSymbol(Sym))
    return TokError("expected identifier");

  if (getLexer().isNot(AsmToken::Comma))
    return TokError("expected comma");
  Lex();

```
- **EN**: Implements logic around `parseDirectiveSize`, `TokError`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSize`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 206-217
```cpp
  const MCExpr *Expr;
  if (getParser().parseExpression(Expr))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token");
  Lex();

  getStreamer().emitELFSize(Sym, Expr);
  return false;
}

```
- **EN**: Implements logic around `TokError`, `Lex`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 218-229
```cpp
bool ELFAsmParser::parseSectionName(StringRef &SectionName) {
  // A section name can contain -, so we cannot just use
  // parseIdentifier.
  SMLoc FirstLoc = getLexer().getLoc();
  unsigned Size = 0;

  if (getLexer().is(AsmToken::String)) {
    SectionName = getTok().getIdentifier();
    Lex();
    return false;
  }

```
- **EN**: Implements logic around `parseSectionName`, `getLexer`, `getTok`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionName`, `getLexer`, `getTok`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 230-249
```cpp
  while (!getParser().hasPendingError()) {
    SMLoc PrevLoc = getLexer().getLoc();
    if (getLexer().is(AsmToken::Comma) ||
      getLexer().is(AsmToken::EndOfStatement))
      break;

    unsigned CurSize;
    if (getLexer().is(AsmToken::String)) {
      CurSize = getTok().getIdentifier().size() + 2;
      Lex();
    } else if (getLexer().is(AsmToken::Identifier)) {
      CurSize = getTok().getIdentifier().size();
      Lex();
    } else {
      CurSize = getTok().getString().size();
      Lex();
    }
    Size += CurSize;
    SectionName = StringRef(FirstLoc.getPointer(), Size);

```
- **EN**: Implements logic around `getLexer`, `getTok`, `Lex`, `StringRef`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `getLexer`, `getTok`, `Lex`, `StringRef` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 250-263
```cpp
    // Make sure the following token is adjacent.
    if (PrevLoc.getPointer() + CurSize != getTok().getLoc().getPointer())
      break;
  }
  if (Size == 0)
    return true;

  return false;
}

static unsigned parseSectionFlags(const Triple &TT, StringRef flagsStr,
                                  bool *UseLastGroup) {
  unsigned flags = 0;

```
- **EN**: Implements logic around `parseSectionFlags`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionFlags` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 264-287
```cpp
  // If a valid numerical value is set for the section flag, use it verbatim
  if (!flagsStr.getAsInteger(0, flags))
    return flags;

  for (char i : flagsStr) {
    switch (i) {
    case 'a':
      flags |= ELF::SHF_ALLOC;
      break;
    case 'e':
      flags |= ELF::SHF_EXCLUDE;
      break;
    case 'x':
      flags |= ELF::SHF_EXECINSTR;
      break;
    case 'w':
      flags |= ELF::SHF_WRITE;
      break;
    case 'o':
      flags |= ELF::SHF_LINK_ORDER;
      break;
    case 'M':
      flags |= ELF::SHF_MERGE;
      break;
```
- **EN**: Implements subsystem-specific case analysis using a `switch` over kinds, opcodes, or states.
- **CN**: 通过对 kind、opcode 或状态执行 `switch` 分析来实现子系统相关逻辑。

### Lines 288-311
```cpp
    case 'S':
      flags |= ELF::SHF_STRINGS;
      break;
    case 'T':
      flags |= ELF::SHF_TLS;
      break;
    case 'c':
      if (TT.getArch() != Triple::xcore)
        return -1U;
      flags |= ELF::XCORE_SHF_CP_SECTION;
      break;
    case 'd':
      if (TT.getArch() != Triple::xcore)
        return -1U;
      flags |= ELF::XCORE_SHF_DP_SECTION;
      break;
    case 'y':
      if (TT.isARM() || TT.isThumb())
        flags |= ELF::SHF_ARM_PURECODE;
      else if (TT.isAArch64())
        flags |= ELF::SHF_AARCH64_PURECODE;
      else
        return -1U;
      break;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 312-335
```cpp
    case 's':
      if (TT.getArch() != Triple::hexagon)
        return -1U;
      flags |= ELF::SHF_HEX_GPREL;
      break;
    case 'G':
      flags |= ELF::SHF_GROUP;
      break;
    case 'l':
      if (TT.getArch() != Triple::x86_64)
        return -1U;
      flags |= ELF::SHF_X86_64_LARGE;
      break;
    case 'R':
      if (TT.isOSSolaris())
        flags |= ELF::SHF_SUNW_NODISCARD;
      else
        flags |= ELF::SHF_GNU_RETAIN;
      break;
    case '?':
      *UseLastGroup = true;
      break;
    default:
      return -1U;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 336-349
```cpp
    }
  }

  return flags;
}

unsigned ELFAsmParser::parseSunStyleSectionFlags() {
  unsigned flags = 0;
  while (getLexer().is(AsmToken::Hash)) {
    Lex(); // Eat the #.

    if (!getLexer().is(AsmToken::Identifier))
      return -1U;

```
- **EN**: Implements logic around `parseSunStyleSectionFlags`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSunStyleSectionFlags`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 350-361
```cpp
    StringRef flagId = getTok().getIdentifier();
    if (flagId == "alloc")
      flags |= ELF::SHF_ALLOC;
    else if (flagId == "execinstr")
      flags |= ELF::SHF_EXECINSTR;
    else if (flagId == "write")
      flags |= ELF::SHF_WRITE;
    else if (flagId == "tls")
      flags |= ELF::SHF_TLS;
    else
      return -1U;

```
- **EN**: Implements logic around `getTok`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 362-374
```cpp
    Lex(); // Eat the flag.

    if (!getLexer().is(AsmToken::Comma))
        break;
    Lex(); // Eat the comma.
  }
  return flags;
}


bool ELFAsmParser::parseDirectivePushSection(StringRef s, SMLoc loc) {
  getStreamer().pushSection();

```
- **EN**: Implements logic around `Lex`, `parseDirectivePushSection`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `parseDirectivePushSection`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 375-388
```cpp
  if (parseSectionArguments(/*IsPush=*/true, loc)) {
    getStreamer().popSection();
    return true;
  }

  return false;
}

bool ELFAsmParser::parseDirectivePopSection(StringRef, SMLoc) {
  if (!getStreamer().popSection())
    return TokError(".popsection without corresponding .pushsection");
  return false;
}

```
- **EN**: Implements logic around `getStreamer`, `parseDirectivePopSection`, `TokError`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getStreamer`, `parseDirectivePopSection`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 389-412
```cpp
bool ELFAsmParser::parseDirectiveSection(StringRef, SMLoc loc) {
  return parseSectionArguments(/*IsPush=*/false, loc);
}

bool ELFAsmParser::maybeParseSectionType(StringRef &TypeName) {
  AsmLexer &L = getLexer();
  if (L.isNot(AsmToken::Comma))
    return false;
  Lex();
  if (L.isNot(AsmToken::At) && L.isNot(AsmToken::Percent) &&
      L.isNot(AsmToken::String)) {
    if (getContext().getAsmInfo().getCommentString().starts_with('@'))
      return TokError("expected '%<type>' or \"<type>\"");
    else
      return TokError("expected '@<type>', '%<type>' or \"<type>\"");
  }
  if (!L.is(AsmToken::String))
    Lex();
  if (L.is(AsmToken::Integer)) {
    TypeName = getTok().getString();
    Lex();
  } else if (getParser().parseIdentifier(TypeName))
    return TokError("expected identifier");
  return false;
```
- **EN**: Implements logic around `parseDirectiveSection`, `parseSectionArguments`, `maybeParseSectionType`, `getLexer`, and 4 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSection`, `parseSectionArguments`, `maybeParseSectionType`, `getLexer`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 413-425
```cpp
}

bool ELFAsmParser::parseMergeSize(int64_t &Size) {
  if (getLexer().isNot(AsmToken::Comma))
    return TokError("expected the entry size");
  Lex();
  if (getParser().parseAbsoluteExpression(Size))
    return true;
  if (Size <= 0)
    return TokError("entry size must be positive");
  return false;
}

```
- **EN**: Implements logic around `parseMergeSize`, `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMergeSize`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 426-449
```cpp
bool ELFAsmParser::parseGroup(StringRef &GroupName, bool &IsComdat) {
  AsmLexer &L = getLexer();
  if (L.isNot(AsmToken::Comma))
    return TokError("expected group name");
  Lex();
  if (L.is(AsmToken::Integer)) {
    GroupName = getTok().getString();
    Lex();
  } else if (getParser().parseIdentifier(GroupName)) {
    return TokError("invalid group name");
  }
  if (L.is(AsmToken::Comma)) {
    Lex();
    StringRef Linkage;
    if (getParser().parseIdentifier(Linkage))
      return TokError("invalid linkage");
    if (Linkage != "comdat")
      return TokError("Linkage must be 'comdat'");
    IsComdat = true;
  } else {
    IsComdat = false;
  }
  return false;
}
```
- **EN**: Implements logic around `parseGroup`, `getLexer`, `TokError`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseGroup`, `getLexer`, `TokError`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 450-471
```cpp

bool ELFAsmParser::parseLinkedToSym(MCSymbolELF *&LinkedToSym) {
  AsmLexer &L = getLexer();
  if (L.isNot(AsmToken::Comma))
    return TokError("expected linked-to symbol");
  Lex();
  StringRef Name;
  SMLoc StartLoc = L.getLoc();
  if (getParser().parseIdentifier(Name)) {
    if (getParser().getTok().getString() == "0") {
      getParser().Lex();
      LinkedToSym = nullptr;
      return false;
    }
    return TokError("invalid linked-to symbol");
  }
  LinkedToSym = static_cast<MCSymbolELF *>(getContext().lookupSymbol(Name));
  if (!LinkedToSym || !LinkedToSym->isInSection())
    return Error(StartLoc, "linked-to symbol is not in a section: " + Name);
  return false;
}

```
- **EN**: Implements logic around `parseLinkedToSym`, `getLexer`, `TokError`, `Lex`, and 4 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseLinkedToSym`, `getLexer`, `TokError`, `Lex`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 472-493
```cpp
static bool hasPrefix(StringRef SectionName, StringRef Prefix) {
  return SectionName.consume_front(Prefix) &&
         (SectionName.empty() || SectionName[0] == '.');
}

static bool allowSectionTypeMismatch(const Triple &TT, StringRef SectionName,
                                     unsigned Type) {
  if (TT.getArch() == Triple::x86_64) {
    // x86-64 psABI names SHT_X86_64_UNWIND as the canonical type for .eh_frame,
    // but GNU as emits SHT_PROGBITS .eh_frame for .cfi_* directives. Don't
    // error for SHT_PROGBITS .eh_frame
    return SectionName == ".eh_frame" && Type == ELF::SHT_PROGBITS;
  }
  if (TT.isMIPS()) {
    // MIPS .debug_* sections should have SHT_MIPS_DWARF section type to
    // distinguish among sections contain DWARF and ECOFF debug formats,
    // but in assembly files these sections have SHT_PROGBITS type.
    return SectionName.starts_with(".debug_") && Type == ELF::SHT_PROGBITS;
  }
  return false;
}

```
- **EN**: Implements logic around `hasPrefix`, `consume_front`, `empty`, `allowSectionTypeMismatch`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `hasPrefix`, `consume_front`, `empty`, `allowSectionTypeMismatch`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 494-510
```cpp
bool ELFAsmParser::parseSectionArguments(bool IsPush, SMLoc loc) {
  StringRef SectionName;

  if (parseSectionName(SectionName))
    return TokError("expected identifier");

  StringRef TypeName;
  int64_t Size = 0;
  StringRef GroupName;
  bool IsComdat = false;
  unsigned Flags = 0;
  unsigned extraFlags = 0;
  const MCExpr *Subsection = nullptr;
  bool UseLastGroup = false;
  MCSymbolELF *LinkedToSym = nullptr;
  int64_t UniqueID = ~0;

```
- **EN**: Implements logic around `parseSectionArguments`, `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSectionArguments`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 511-525
```cpp
  // Set the defaults first.
  if (hasPrefix(SectionName, ".rodata") || SectionName == ".rodata1")
    Flags |= ELF::SHF_ALLOC;
  else if (SectionName == ".fini" || SectionName == ".init" ||
           hasPrefix(SectionName, ".text"))
    Flags |= ELF::SHF_ALLOC | ELF::SHF_EXECINSTR;
  else if (hasPrefix(SectionName, ".data") || SectionName == ".data1" ||
           hasPrefix(SectionName, ".bss") ||
           hasPrefix(SectionName, ".init_array") ||
           hasPrefix(SectionName, ".fini_array") ||
           hasPrefix(SectionName, ".preinit_array"))
    Flags |= ELF::SHF_ALLOC | ELF::SHF_WRITE;
  else if (hasPrefix(SectionName, ".tdata") || hasPrefix(SectionName, ".tbss"))
    Flags |= ELF::SHF_ALLOC | ELF::SHF_WRITE | ELF::SHF_TLS;

```
- **EN**: Implements logic around `hasPrefix`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `hasPrefix` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 526-547
```cpp
  if (getLexer().is(AsmToken::Comma)) {
    Lex();

    if (IsPush && getLexer().isNot(AsmToken::String)) {
      if (getParser().parseExpression(Subsection))
        return true;
      if (getLexer().isNot(AsmToken::Comma))
        goto EndStmt;
      Lex();
    }

    if (getLexer().isNot(AsmToken::String)) {
      if (getLexer().isNot(AsmToken::Hash))
        return TokError("expected string");
      extraFlags = parseSunStyleSectionFlags();
    } else {
      StringRef FlagsStr = getTok().getStringContents();
      Lex();
      extraFlags = parseSectionFlags(getContext().getTargetTriple(), FlagsStr,
                                     &UseLastGroup);
    }

```
- **EN**: Implements logic around `Lex`, `TokError`, `parseSunStyleSectionFlags`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `TokError`, `parseSunStyleSectionFlags`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 548-560
```cpp
    if (extraFlags == -1U)
      return TokError("unknown flag");
    Flags |= extraFlags;

    bool Mergeable = Flags & ELF::SHF_MERGE;
    bool Group = Flags & ELF::SHF_GROUP;
    if (Group && UseLastGroup)
      return TokError("Section cannot specifiy a group name while also acting "
                      "as a member of the last group");

    if (maybeParseSectionType(TypeName))
      return true;

```
- **EN**: Implements logic around `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 561-583
```cpp
    AsmLexer &L = getLexer();
    if (TypeName.empty()) {
      if (Mergeable)
        return TokError("Mergeable section must specify the type");
      if (Group)
        return TokError("Group section must specify the type");
      if (L.isNot(AsmToken::EndOfStatement))
        return TokError("expected end of directive");
    }

    if (Mergeable || TypeName == "llvm_cfi_jump_table")
      if (parseMergeSize(Size))
        return true;
    if (Flags & ELF::SHF_LINK_ORDER)
      if (parseLinkedToSym(LinkedToSym))
        return true;
    if (Group)
      if (parseGroup(GroupName, IsComdat))
        return true;
    if (maybeParseUniqueID(UniqueID))
      return true;
  }

```
- **EN**: Implements logic around `getLexer`, `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getLexer`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 584-607
```cpp
EndStmt:
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("expected end of directive");
  Lex();

  unsigned Type = ELF::SHT_PROGBITS;

  if (TypeName.empty()) {
    if (SectionName.starts_with(".note"))
      Type = ELF::SHT_NOTE;
    else if (hasPrefix(SectionName, ".init_array"))
      Type = ELF::SHT_INIT_ARRAY;
    else if (hasPrefix(SectionName, ".bss"))
      Type = ELF::SHT_NOBITS;
    else if (hasPrefix(SectionName, ".tbss"))
      Type = ELF::SHT_NOBITS;
    else if (hasPrefix(SectionName, ".fini_array"))
      Type = ELF::SHT_FINI_ARRAY;
    else if (hasPrefix(SectionName, ".preinit_array"))
      Type = ELF::SHT_PREINIT_ARRAY;
  } else {
    if (TypeName == "init_array")
      Type = ELF::SHT_INIT_ARRAY;
    else if (TypeName == "fini_array")
```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 608-631
```cpp
      Type = ELF::SHT_FINI_ARRAY;
    else if (TypeName == "preinit_array")
      Type = ELF::SHT_PREINIT_ARRAY;
    else if (TypeName == "nobits")
      Type = ELF::SHT_NOBITS;
    else if (TypeName == "progbits")
      Type = ELF::SHT_PROGBITS;
    else if (TypeName == "note")
      Type = ELF::SHT_NOTE;
    else if (TypeName == "unwind")
      Type = ELF::SHT_X86_64_UNWIND;
    else if (TypeName == "llvm_odrtab")
      Type = ELF::SHT_LLVM_ODRTAB;
    else if (TypeName == "llvm_linker_options")
      Type = ELF::SHT_LLVM_LINKER_OPTIONS;
    else if (TypeName == "llvm_call_graph_profile")
      Type = ELF::SHT_LLVM_CALL_GRAPH_PROFILE;
    else if (TypeName == "llvm_dependent_libraries")
      Type = ELF::SHT_LLVM_DEPENDENT_LIBRARIES;
    else if (TypeName == "llvm_sympart")
      Type = ELF::SHT_LLVM_SYMPART;
    else if (TypeName == "llvm_bb_addr_map")
      Type = ELF::SHT_LLVM_BB_ADDR_MAP;
    else if (TypeName == "llvm_offloading")
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 632-644
```cpp
      Type = ELF::SHT_LLVM_OFFLOADING;
    else if (TypeName == "llvm_lto")
      Type = ELF::SHT_LLVM_LTO;
    else if (TypeName == "llvm_jt_sizes")
      Type = ELF::SHT_LLVM_JT_SIZES;
    else if (TypeName == "llvm_cfi_jump_table")
      Type = ELF::SHT_LLVM_CFI_JUMP_TABLE;
    else if (TypeName == "llvm_call_graph")
      Type = ELF::SHT_LLVM_CALL_GRAPH;
    else if (TypeName.getAsInteger(0, Type))
      return TokError("unknown section type");
  }

```
- **EN**: Implements logic around `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 645-668
```cpp
  if (UseLastGroup) {
    if (auto *Section = static_cast<const MCSectionELF *>(
            getStreamer().getCurrentSectionOnly()))
      if (const MCSymbol *Group = Section->getGroup()) {
        GroupName = Group->getName();
        IsComdat = Section->isComdat();
        Flags |= ELF::SHF_GROUP;
      }
  }

  MCSectionELF *Section =
      getContext().getELFSection(SectionName, Type, Flags, Size, GroupName,
                                 IsComdat, UniqueID, LinkedToSym);
  getStreamer().switchSection(Section, Subsection);
  // Check that flags are used consistently. However, the GNU assembler permits
  // to leave out in subsequent uses of the same sections; for compatibility,
  // do likewise.
  if (!TypeName.empty() && Section->getType() != Type &&
      !allowSectionTypeMismatch(getContext().getTargetTriple(), SectionName,
                                Type))
    Error(loc, "changed section type for " + SectionName + ", expected: 0x" +
                   utohexstr(Section->getType()));
  if ((extraFlags || Size || !TypeName.empty()) && Section->getFlags() != Flags)
    Error(loc, "changed section flags for " + SectionName + ", expected: 0x" +
```
- **EN**: Implements logic around `getStreamer`, `getName`, `isComdat`, `getContext`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getStreamer`, `getName`, `isComdat`, `getContext`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 669-682
```cpp
                   utohexstr(Section->getFlags()));
  if ((extraFlags || Size || !TypeName.empty()) &&
      Section->getEntrySize() != Size)
    Error(loc, "changed section entsize for " + SectionName +
                   ", expected: " + Twine(Section->getEntrySize()));

  if (getContext().getGenDwarfForAssembly() &&
      (Section->getFlags() & ELF::SHF_ALLOC) &&
      (Section->getFlags() & ELF::SHF_EXECINSTR)) {
    bool InsertResult = getContext().addGenDwarfSection(Section);
    if (InsertResult && getContext().getDwarfVersion() <= 2)
      Warning(loc, "DWARF2 only supports one section per compilation unit");
  }

```
- **EN**: Implements logic around `utohexstr`, `getEntrySize`, `Error`, `Twine`, and 3 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `utohexstr`, `getEntrySize`, `Error`, `Twine`, and 3 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 683-694
```cpp
  return false;
}

bool ELFAsmParser::parseDirectivePrevious(StringRef DirName, SMLoc) {
  MCSectionSubPair PreviousSection = getStreamer().getPreviousSection();
  if (PreviousSection.first == nullptr)
      return TokError(".previous without corresponding .section");
  getStreamer().switchSection(PreviousSection.first, PreviousSection.second);

  return false;
}

```
- **EN**: Implements logic around `parseDirectivePrevious`, `getStreamer`, `TokError`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectivePrevious`, `getStreamer`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 695-707
```cpp
static MCSymbolAttr MCAttrForString(StringRef Type) {
  return StringSwitch<MCSymbolAttr>(Type)
      .Cases({"STT_FUNC", "function"}, MCSA_ELF_TypeFunction)
      .Cases({"STT_OBJECT", "object"}, MCSA_ELF_TypeObject)
      .Cases({"STT_TLS", "tls_object"}, MCSA_ELF_TypeTLS)
      .Cases({"STT_COMMON", "common"}, MCSA_ELF_TypeCommon)
      .Cases({"STT_NOTYPE", "notype"}, MCSA_ELF_TypeNoType)
      .Cases({"STT_GNU_IFUNC", "gnu_indirect_function"},
             MCSA_ELF_TypeIndFunction)
      .Case("gnu_unique_object", MCSA_ELF_TypeGnuUniqueObject)
      .Default(MCSA_Invalid);
}

```
- **EN**: Implements logic around `MCAttrForString`, `StringSwitch<MCSymbolAttr>`, `Cases`, `Case`, and 1 more symbols; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `MCAttrForString`, `StringSwitch<MCSymbolAttr>`, `Cases`, `Case`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 708-724
```cpp
/// parseDirectiveELFType
///  ::= .type identifier , STT_<TYPE_IN_UPPER_CASE>
///  ::= .type identifier , #attribute
///  ::= .type identifier , @attribute
///  ::= .type identifier , %attribute
///  ::= .type identifier , "attribute"
bool ELFAsmParser::parseDirectiveType(StringRef, SMLoc) {
  MCSymbol *Sym;
  if (getParser().parseSymbol(Sym))
    return TokError("expected identifier");

  bool AllowAt = getLexer().getAllowAtInIdentifier();
  if (!AllowAt &&
      !getContext().getAsmInfo().getCommentString().starts_with("@"))
    getLexer().setAllowAtInIdentifier(true);
  llvm::scope_exit _([&]() { getLexer().setAllowAtInIdentifier(AllowAt); });

```
- **EN**: Implements logic around `parseDirectiveType`, `TokError`, `getLexer`, `getContext`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveType`, `TokError`, `getLexer`, `getContext`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 725-744
```cpp
  // NOTE the comma is optional in all cases.  It is only documented as being
  // optional in the first case, however, GAS will silently treat the comma as
  // optional in all cases.  Furthermore, although the documentation states that
  // the first form only accepts STT_<TYPE_IN_UPPER_CASE>, in reality, GAS
  // accepts both the upper case name as well as the lower case aliases.
  if (getLexer().is(AsmToken::Comma))
    Lex();

  if (getLexer().isNot(AsmToken::Identifier) &&
      getLexer().isNot(AsmToken::Hash) &&
      getLexer().isNot(AsmToken::Percent) &&
      getLexer().isNot(AsmToken::String)) {
    if (!getLexer().getAllowAtInIdentifier())
      return TokError("expected STT_<TYPE_IN_UPPER_CASE>, '#<type>', "
                      "'%<type>' or \"<type>\"");
    else if (getLexer().isNot(AsmToken::At))
      return TokError("expected STT_<TYPE_IN_UPPER_CASE>, '#<type>', '@<type>', "
                      "'%<type>' or \"<type>\"");
  }

```
- **EN**: Implements logic around `Lex`, `getLexer`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `getLexer`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 745-758
```cpp
  if (getLexer().isNot(AsmToken::String) &&
      getLexer().isNot(AsmToken::Identifier))
    Lex();

  SMLoc TypeLoc = getLexer().getLoc();

  StringRef Type;
  if (getParser().parseIdentifier(Type))
    return TokError("expected symbol type");

  MCSymbolAttr Attr = MCAttrForString(Type);
  if (Attr == MCSA_Invalid)
    return Error(TypeLoc, "unsupported attribute");

```
- **EN**: Implements logic around `getLexer`, `Lex`, `TokError`, `MCAttrForString`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getLexer`, `Lex`, `TokError`, `MCAttrForString`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 759-773
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("expected end of directive");
  Lex();

  getStreamer().emitSymbolAttribute(Sym, Attr);

  return false;
}

/// parseDirectiveIdent
///  ::= .ident string
bool ELFAsmParser::parseDirectiveIdent(StringRef, SMLoc) {
  if (getLexer().isNot(AsmToken::String))
    return TokError("expected string");

```
- **EN**: Implements logic around `TokError`, `Lex`, `getStreamer`, `parseDirectiveIdent`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `getStreamer`, `parseDirectiveIdent` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 774-785
```cpp
  StringRef Data = getTok().getIdentifier();

  Lex();

  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("expected end of directive");
  Lex();

  getStreamer().emitIdent(Data);
  return false;
}

```
- **EN**: Implements logic around `getTok`, `Lex`, `TokError`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `Lex`, `TokError`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 786-805
```cpp
/// parseDirectiveSymver
///  ::= .symver foo, bar2@zed
bool ELFAsmParser::parseDirectiveSymver(StringRef, SMLoc) {
  MCSymbol *OriginalSym;
  StringRef Name, Action;
  if (getParser().parseSymbol(OriginalSym))
    return TokError("expected identifier");

  if (getLexer().isNot(AsmToken::Comma))
    return TokError("expected a comma");

  // ARM assembly uses @ for a comment...
  // except when parsing the second parameter of the .symver directive.
  // Force the next symbol to allow @ in the identifier, which is
  // required for this directive and then reset it to its initial state.
  const bool AllowAtInIdentifier = getLexer().getAllowAtInIdentifier();
  getLexer().setAllowAtInIdentifier(true);
  Lex();
  getLexer().setAllowAtInIdentifier(AllowAtInIdentifier);

```
- **EN**: Implements logic around `parseDirectiveSymver`, `TokError`, `getLexer`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveSymver`, `TokError`, `getLexer`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 806-818
```cpp
  if (getParser().parseIdentifier(Name))
    return TokError("expected identifier");

  if (!Name.contains('@'))
    return TokError("expected a '@' in the name");
  bool KeepOriginalSym = !Name.contains("@@@");
  if (parseOptionalToken(AsmToken::Comma)) {
    if (getParser().parseIdentifier(Action) || Action != "remove")
      return TokError("expected 'remove'");
    KeepOriginalSym = false;
  }
  (void)parseOptionalToken(AsmToken::EndOfStatement);

```
- **EN**: Implements logic around `TokError`, `contains`, `parseOptionalToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `contains`, `parseOptionalToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 819-830
```cpp
  getStreamer().emitELFSymverDirective(OriginalSym, Name, KeepOriginalSym);
  return false;
}

/// parseDirectiveVersion
///  ::= .version string
bool ELFAsmParser::parseDirectiveVersion(StringRef, SMLoc) {
  if (getLexer().isNot(AsmToken::String))
    return TokError("expected string");

  StringRef Data = getTok().getIdentifier();

```
- **EN**: Implements logic around `getStreamer`, `parseDirectiveVersion`, `TokError`, `getTok`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getStreamer`, `parseDirectiveVersion`, `TokError`, `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 831-846
```cpp
  Lex();

  MCSection *Note = getContext().getELFSection(".note", ELF::SHT_NOTE, 0);

  getStreamer().pushSection();
  getStreamer().switchSection(Note);
  getStreamer().emitInt32(Data.size() + 1); // namesz
  getStreamer().emitInt32(0);               // descsz = 0 (no description).
  getStreamer().emitInt32(1);               // type = NT_VERSION
  getStreamer().emitBytes(Data);            // name
  getStreamer().emitInt8(0);                // NUL
  getStreamer().emitValueToAlignment(Align(4));
  getStreamer().popSection();
  return false;
}

```
- **EN**: Implements logic around `Lex`, `getContext`, `getStreamer`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `getContext`, `getStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 847-858
```cpp
/// parseDirectiveWeakref
///  ::= .weakref foo, bar
bool ELFAsmParser::parseDirectiveWeakref(StringRef, SMLoc) {
  // FIXME: Share code with the other alias building directives.

  MCSymbol *Alias;
  if (getParser().parseSymbol(Alias))
    return TokError("expected identifier");

  if (getLexer().isNot(AsmToken::Comma))
    return TokError("expected a comma");

```
- **EN**: Implements logic around `parseDirectiveWeakref`, `TokError`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveWeakref`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 859-875
```cpp
  Lex();

  MCSymbol *Sym;
  if (getParser().parseSymbol(Sym))
    return TokError("expected identifier");

  getStreamer().emitWeakReference(Alias, Sym);
  return false;
}

bool ELFAsmParser::parseDirectiveSubsection(StringRef, SMLoc) {
  const MCExpr *Subsection = MCConstantExpr::create(0, getContext());
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    if (getParser().parseExpression(Subsection))
     return true;
  }

```
- **EN**: Implements logic around `Lex`, `TokError`, `getStreamer`, `parseDirectiveSubsection`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `TokError`, `getStreamer`, `parseDirectiveSubsection`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 876-888
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("expected end of directive");

  Lex();

  return getStreamer().switchSection(getStreamer().getCurrentSectionOnly(),
                                     Subsection);
}

bool ELFAsmParser::parseDirectiveCGProfile(StringRef S, SMLoc Loc) {
  return MCAsmParserExtension::parseDirectiveCGProfile(S, Loc);
}

```
- **EN**: Implements logic around `TokError`, `Lex`, `getStreamer`, `parseDirectiveCGProfile`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `getStreamer`, `parseDirectiveCGProfile` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 889-889
```cpp
MCAsmParserExtension *llvm::createELFAsmParser() { return new ELFAsmParser; }
```
- **EN**: Implements logic around `createELFAsmParser`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `createELFAsmParser` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/ScopeExit.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDirectives.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCAsmParser.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`, `llvm/MC/MCSectionELF.h` ... (+7 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat
